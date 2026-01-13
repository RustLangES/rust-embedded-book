# Excepciones

Las excepciones y las interrupciones son un mecanismo de hardware mediante el cual el procesador gestiona eventos asincrónicos y errores fatales 
(por ejemplo, la ejecución de una instrucción no válida). Las excepciones implican preempción e involucran manejadores de excepciones, subrutinas 
que se ejecutan en respuesta a la señal que desencadenó el evento.

La crate`cortex-m-rt` proporciona un atributo [`exception`] para declarar controladores de excepciones.

[`exception`]: https://docs.rs/cortex-m-rt-macros/latest/cortex_m_rt_macros/attr.exception.html

``` rust,ignore
// Exception handler for the SysTick (System Timer) exception
#[exception]
fn SysTick() {
    // ..
}
```

Aparte del atributo `exception`, los manejadores de excepciones parecen funciones simples, pero hay una diferencia más: los manejadores de `exception` *no* pueden ser invocados por software. Siguiendo el ejemplo anterior, la instrucción `SysTick();` generaría un error de compilación.

Este comportamiento es prácticamente intencionado y necesario para proporcionar una característica: las variables `static mut` declaradas *dentro* de los controladores de `excepciones` son *seguras* de usar.

``` rust,ignore
#[exception]
fn SysTick() {
    static mut COUNT: u32 = 0;

    // `COUNT` Se ha transformado al tipo `&mut u32` y es seguro usarlo
    *COUNT += 1;
}
```

Como sabrá, usar variables `static mut` en una función la convierte en [*no reentrante*](https://es.wikipedia.org/wiki/Reentrancia_(informática)). No está definido llamar a una función no reentrante, directa o indirectamente, desde más de un manejador de excepciones/interrupciones o desde `main` y uno o más manejadores de excepciones/interrupciones.

Safe Rust nunca debe dar lugar a un comportamiento indefinido, por lo que las funciones no reentrantes
deben marcarse como `unsafe`. Sin embargo, acabo de decir que los controladores de `exception` pueden utilizar con seguridad
variables `static mut`. ¿Cómo es esto posible? Esto es posible porque
los controladores de `exception` *no* pueden ser llamados por el software, por lo que la reentrada no es
posible. Estos controladores son llamados por el propio hardware, que se supone que es físicamente no concurrente.


Como resultado, en el contexto de los controladores de excepciones en sistemas integrados, la ausencia de invocaciones concurrentes del mismo controlador garantiza que no haya problemas de reentrada, incluso si el controlador utiliza variables estáticas mutables.  

En un sistema multinúcleo, en el que varios núcleos de procesador ejecutan código simultáneamente, la posibilidad de que se produzcan problemas de reentrada vuelve a ser relevante, incluso dentro de los controladores de excepciones. Aunque cada núcleo puede tener su propio conjunto de controladores de excepciones, puede haber situaciones en las que varios núcleos intenten ejecutar el mismo controlador de excepciones al mismo tiempo.  
Para abordar esta preocupación en un entorno multinúcleo, es necesario emplear mecanismos de sincronización adecuados dentro de los controladores de excepciones para garantizar que el acceso a los recursos compartidos se coordine correctamente entre los núcleos. Esto suele implicar el uso de técnicas como bloqueos, semáforos u operaciones atómicas para evitar conflictos de datos y mantener la integridad de los mismos.

>Tenga en cuenta que el atributo `exception` transforma las definiciones de las variables estáticas
>dentro de la función envolviéndolas en bloques `unsafe` y proporcionándonos
>nuevas variables apropiadas de tipo `&mut` con el mismo nombre.
>De este modo, podemos desreferenciar la referencia mediante `*` para acceder a los valores de las variables sin
>necesidad de envolverlas en un bloque `unsafe`.

## Un ejemplo completo

A continuación se muestra un ejemplo que utiliza el temporizador del sistema para generar una excepción `SysTick`
aproximadamente cada segundo. El controlador de excepciones `SysTick` realiza un seguimiento del número de
veces que se ha llamado en la variable `COUNT` y, a continuación, imprime el valor de
`COUNT` en la consola del host mediante semihosting.

> **NOTA**: Puede ejecutar este ejemplo en cualquier dispositivo Cortex-M; también puede ejecutarlo
> en QEMU.

```rust,ignore
#![deny(unsafe_code)]
#![no_main]
#![no_std]

use panic_halt as _;

use core::fmt::Write;

use cortex_m::peripheral::syst::SystClkSource;
use cortex_m_rt::{entry, exception};
use cortex_m_semihosting::{
    debug,
    hio::{self, HStdout},
};

#[entry]
fn main() -> ! {
    let p = cortex_m::Peripherals::take().unwrap();
    let mut syst = p.SYST;

    // configura el temporizador del sistema para que active una excepción SysTick cada segundo
    syst.set_clock_source(SystClkSource::Core);
    // Esto está configurado para el LM3S6965, que tiene una frecuencia de reloj de CPU predeterminada de 12 MHz.
    syst.set_reload(12_000_000);
    syst.clear_current();
    syst.enable_counter();
    syst.enable_interrupt();

    loop {}
}

#[exception]
fn SysTick() {
    static mut COUNT: u32 = 0;
    static mut STDOUT: Option<HStdout> = None;

    *COUNT += 1;

    // Inicialización diferida
    if STDOUT.is_none() {
        *STDOUT = hio::hstdout().ok();
    }

    if let Some(hstdout) = STDOUT.as_mut() {
        write!(hstdout, "{}", *COUNT).ok();
    }

    // IMPORTANTE: omita este bloque «if» si se ejecuta en hardware real o su
    // depurador terminará en un estado inconsistente.
    if *COUNT == 9 {
        // Esto terminará el proceso de QEMU.
        debug::exit(debug::EXIT_SUCCESS);
    }
}
```

``` console
tail -n5 Cargo.toml
```

``` toml
[dependencies]
cortex-m = "0.5.7"
cortex-m-rt = "0.6.3"
panic-halt = "0.2.0"
cortex-m-semihosting = "0.3.1"
```

``` text
$ cargo run --release
     Running `qemu-system-arm -cpu cortex-m3 -machine lm3s6965evb (..)
123456789
```

Si ejecuta esto en la placa Discovery, verá el resultado en la consola OpenOCD.
Además, el programa *no* se detendrá cuando el recuento llegue a 9.

## El controlador de excepciones predeterminado

Lo que realmente hace el atributo `exception` es *anular* el controlador de excepciones predeterminado
para una excepción específica. Si no se anula el controlador para una
excepción concreta, esta será gestionada por la función `DefaultHandler`, cuyo
valor predeterminado es:

``` rust,ignore
fn DefaultHandler() {
    loop {}
}
```

Esta función la proporciona el crate `cortex-m-rt` y está marcada como
`#[no_mangle]`, por lo que puede colocar un punto de interrupción en "DefaultHandler" y capturar excepciones
*no gestionadas*.

Es posible anular este `DefaultHandler` utilizando el atributo `exception`:

``` rust,ignore
#[exception]
fn DefaultHandler(irqn: i16) {
    // controlador predeterminado personalizado
}
```

El argumento `irqn` indica qué excepción se está atendiendo. Un valor negativo
indica que se está atendiendo una excepción Cortex-M; y un valor cero o
positivo indica que se está atendiendo una excepción específica del dispositivo, también conocida como interrupción, está
en proceso de reparación.

## El controlador de fallos graves

La excepción `HardFault` es un poco especial. Esta excepción se activa cuando el
programa entra en un estado no válido, por lo que su controlador *no* puede regresar, ya que eso podría
dar lugar a un comportamiento indefinido. Además, el crate de tiempo de ejecución realiza algunas tareas antes de
que se invoque el controlador `HardFault` definido por el usuario para mejorar la capacidad de depuración.

El resultado es que el controlador `HardFault` debe tener la siguiente firma:
`fn(&ExceptionFrame) -> !`. El argumento del controlador es un puntero a
los registros que la excepción introdujo en la pila. Estos registros son
una instantánea del estado del procesador en el momento en que se activó la excepción y
son útiles para diagnosticar un fallo grave.

Aquí hay un ejemplo que realiza una operación ilegal: una lectura en una ubicación de memoria inexistente.
memoria.

> **NOTA**: Este programa no funcionará, es decir, no se bloqueará, en QEMU porque
> `qemu-system-arm -machine lm3s6965evb` no comprueba las cargas de memoria y
> devolverá sin problemas `0` en las lecturas de memoria no válida.

```rust,ignore
#![no_main]
#![no_std]

use panic_halt as _;

use core::fmt::Write;
use core::ptr;

use cortex_m_rt::{entry, exception, ExceptionFrame};
use cortex_m_semihosting::hio;

#[entry]
fn main() -> ! {
    // leer una ubicación de memoria inexistente
    unsafe {
        ptr::read_volatile(0x3FFF_0000 as *const u32);
    }

    loop {}
}

#[exception]
fn HardFault(ef: &ExceptionFrame) -> ! {
    if let Ok(mut hstdout) = hio::hstdout() {
        writeln!(hstdout, "{:#?}", ef).ok();
    }

    loop {}
}
```

El controlador `HardFault` imprime el valor `ExceptionFrame`. Si ejecuta esto,
 verá algo como esto en la consola OpenOCD.

``` text
$ openocd
(..)
ExceptionFrame {
    r0: 0x3fff0000,
    r1: 0x00000003,
    r2: 0x080032e8,
    r3: 0x00000000,
    r12: 0x00000000,
    lr: 0x080016df,
    pc: 0x080016e2,
    xpsr: 0x61000000,
}
```

El valor `pc` es el valor del contador de programa en el momento de la excepción
y apunta a la instrucción que la provocó.

Si observas el desensamblado del programa:


``` text
$ cargo objdump --bin app --release -- -d --no-show-raw-insn --print-imm-hex
(..)
ResetTrampoline:
 8000942:       movw    r0, #0xfffe
 8000946:       movt    r0, #0x3fff
 800094a:       ldr     r0, [r0]
 800094c:       b       #-0x4 <ResetTrampoline+0xa>
```

Puede buscar el valor del contador de programa `0x0800094a` en el desensamblado.
Verá que una operación de carga (`ldr r0, [r0]`) provocó la excepción.
El campo `r0` de `ExceptionFrame` le indicará que el valor del registro `r0`
era `0x3fff_fffe` en ese momento.
