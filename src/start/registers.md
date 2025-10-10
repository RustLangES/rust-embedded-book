# Registros mapeados en memoria

Los sistemas embebidos solo pueden llegar hasta cierto punto ejecutando código Rust normal y moviendo datos en la RAM. Si queremos introducir o extraer información de nuestro sistema (ya sea parpadear un LED, detectar la pulsación de un botón o comunicarnos con un periférico externo a través de algún tipo de bus), tendremos que adentrarnos en el mundo de los periféricos y sus «registros mapeados en memoria».

Es posible que descubra que el código que necesita para acceder a los periféricos de su microcontrolador ya está escrito en uno de los siguientes niveles:
<p align="center">
<img title="Common crates" src="../assets/crates.png">
</p>

* Crate de microarquitectura: Este tipo de crate gestiona las rutinas útiles comunes al core del procesador que utiliza el microcontrolador, así como los periféricos comunes a todos los microcontroladores que utilizan ese tipo de core. Por ejemplo, la crate [cortex-m] ofrece funciones para habilitar y deshabilitar interrupciones, las cuales son comunes para todos los microcontroladores basados ​​en Cortex-M. También permite acceder al periférico "SysTick" incluido con todos los microcontroladores basados ​​en Cortex-M.
* Crate de Acceso a Periféricos (PAC): Este tipo de crate es una fina envoltura que cubre los diversos registros de memoria definidos para el número de pieza del microcontrolador que utiliza. Por ejemplo, [tm4c123x] para la serie Texas Instruments Tiva-C TM4C123, o [stm32f30x] para la serie ST-Micro STM32F30x. Aquí, interactuará directamente con los registros, siguiendo las instrucciones de funcionamiento de cada periférico que se encuentran en el Manual de Referencia Técnica de su microcontrolador.
* Crate HAL: Estas crates ofrecen una API más intuitiva para su procesador, a menudo implementando características comunes definidas en [embedded-hal]. Por ejemplo, esta crate podría ofrecer una estructura `Serial`, con un constructor que toma un conjunto adecuado de pines GPIO y una velocidad en baudios, y ofrece algún tipo de función `write_byte` para enviar datos. Consulte el capítulo sobre [Portabilidad] para obtener más información sobre [embedded-hal].
* Crate de placa: estas crates van un paso más allá que una crate HAL al preconfigurar varios periféricos y pines GPIO para adaptarse al kit de desarrollador o placa específica que esté utilizando, como [stm32f3-discovery] para la placa STM32F3DISCOVERY.

[cortex-m]: https://crates.io/crates/cortex-m
[tm4c123x]: https://crates.io/crates/tm4c123x
[stm32f30x]: https://crates.io/crates/stm32f30x
[embedded-hal]: https://crates.io/crates/embedded-hal
[Portabilidad]: ../portability/index.md
[stm32f3-discovery]: https://crates.io/crates/stm32f3-discovery
[Discovery]: https://rust-embedded.github.io/discovery/

## Crate de placa

Una crate de placa es el punto de partida perfecto si eres nuevo en Rust embebido. Abstrae de forma elegante los detalles de hardware que pueden resultar abrumadores al comenzar a estudiar este tema y facilita tareas habituales, como encender o apagar un LED. La funcionalidad que ofrece varía considerablemente entre placas. Dado que este libro se centra en la compatibilidad con hardware, no se abordarán las crate de placa.

Si desea experimentar con la placa STM32F3DISCOVERY, le recomendamos encarecidamente que consulte la crate de placa [stm32f3-discovery], que permite hacer parpadear los LED de la placa, acceder a su brújula, Bluetooth y más. El libro [Discovery] ofrece una excelente introducción al uso de una crate de placa.

Pero si estás trabajando en un sistema que aún no tiene una crate de placa dedicada, o necesitas una funcionalidad que no ofrecen las crates existentes, continúa leyendo, ya que comenzamos desde abajo, con las crates de microarquitectura.

## Crate de micro-architectura

Analicemos el periférico SysTick, común a todos los microcontroladores basados ​​en Cortex-M. Encontramos una API de bajo nivel en el paquete [cortex-m], que podemos usar de la siguiente manera:

```rust,ignore
#![no_std]
#![no_main]
use cortex_m::peripheral::{syst, Peripherals};
use cortex_m_rt::entry;
use panic_halt as _;

#[entry]
fn main() -> ! {
    let peripherals = Peripherals::take().unwrap();
    let mut systick = peripherals.SYST;
    systick.set_clock_source(syst::SystClkSource::Core);
    systick.set_reload(1_000);
    systick.clear_current();
    systick.enable_counter();
    while !systick.has_wrapped() {
        // Loop
    }

    loop {}
}
```
Las funciones de la estructura `SYST` se corresponden con la funcionalidad definida en el Manual de Referencia Técnica de ARM para este periférico. Esta API no menciona el "retardo de X milisegundos"; debemos implementarlo nosotros mismos mediante un bucle `while`. Tenga en cuenta que no podemos acceder a nuestra estructura `SYST` hasta que hayamos llamado a `Peripherals::take()`; esta es una rutina especial que garantiza que solo haya una estructura `SYST` en todo el programa. Para más información, consulte la sección [Periféricos].

[Periféricos]: ../peripherals/index.md

## Uso de una crate de acceso periférico (PAC)

No avanzaremos mucho en el desarrollo de software embebido si nos limitamos a los periféricos básicos incluidos con cada Cortex-M. En algún momento, necesitaremos escribir código específico para el microcontrolador que usemos. En este ejemplo, supongamos que tenemos un Texas Instruments TM4C123, un Cortex-M4 de 80 MHz con 256 KiB de memoria Flash. Utilizaremos el crate [tm4c123x] para usar este chip.

```rust,ignore
#![no_std]
#![no_main]

use panic_halt as _; // panic handler

use cortex_m_rt::entry;
use tm4c123x;

#[entry]
pub fn init() -> (Delay, Leds) {
    let cp = cortex_m::Peripherals::take().unwrap();
    let p = tm4c123x::Peripherals::take().unwrap();

    let pwm = p.PWM0;
    pwm.ctl.write(|w| w.globalsync0().clear_bit());
    // Modo = 1 => Modo de conteo ascendente/descendente
    pwm._2_ctl.write(|w| w.enable().set_bit().mode().set_bit());
    pwm._2_gena.write(|w| w.actcmpau().zero().actcmpad().one());
    // 528 ciclos (264 arriba y abajo) = 4 bucles por línea de video (2112 ciclos)
    pwm._2_load.write(|w| unsafe { w.load().bits(263) });
    pwm._2_cmpa.write(|w| unsafe { w.compa().bits(64) });
    pwm.enable.write(|w| w.pwm4en().set_bit());
}

```

Hemos accedido al periférico `PWM0` exactamente de la misma manera que accedimos al periférico `SYST` anteriormente, excepto que llamamos a `tm4c123x::Peripherals::take()`. Como este crate se generó automáticamente con [svd2rust], las funciones de acceso a nuestros campos de registro aceptan un closure, en lugar de un argumento numérico. Aunque esto parece mucho código, el compilador de Rust puede usarlo para realizar varias comprobaciones, generando código máquina que se asemeja bastante al ensamblador escrito a mano. Si el código generado automáticamente no puede determinar que todos los argumentos posibles para una función de acceso en particular son válidos (por ejemplo, si el SVD define el registro como de 32 bits, pero no indica si algunos de esos valores tienen un significado especial), la función se marca como `insegura`. Podemos ver esto en el ejemplo anterior al configurar los subcampos `load` y `compa` con la función `bits()`.

### Leyendo

La función `read()` devuelve un objeto que otorga acceso de solo lectura a los distintos subcampos de este registro, según lo definido en el archivo SVD del fabricante para este chip. Puede encontrar todas las funciones disponibles para el tipo de retorno `R` especial para este registro, en este periférico y en este chip en la [documentación de tm4c123x][documentación de tm4c123x R].

```rust,ignore
if pwm.ctl.read().globalsync0().is_set() {
    // Hacer una cosa
}
```

### Escribiendo

La función `write()` acepta un cierre con un solo argumento. Normalmente lo llamamos `w`. Este argumento otorga acceso de lectura y escritura a los distintos subcampos de este registro, según lo definido en el archivo SVD del fabricante para este chip. Puede encontrar todas las funciones disponibles en `w` para este registro, en este periférico y en este chip en la [documentación de tm4c123x][Documentación de tm4c123x W]. Tenga en cuenta que todos los subcampos que no configuremos se establecerán con un valor predeterminado; se perderá cualquier contenido existente en el registro.
```rust,ignore
pwm.ctl.write(|w| w.globalsync0().clear_bit());
```

### Modificando

Si deseamos modificar solo un subcampo específico de este registro y dejar los demás sin cambios, podemos usar la función `modify`. Esta función acepta un closure con dos argumentos: uno para leer y otro para escribir. Normalmente, los llamamos `r` y `w`, respectivamente. El argumento `r` permite inspeccionar el contenido actual del registro y el argumento `w` permite modificarlo.

```rust,ignore
pwm.ctl.modify(|r, w| w.globalsync0().clear_bit());
```

La función `modify` realmente demuestra el poder de los closures. En C, tendríamos que leer un valor temporal, modificar los bits correctos y luego escribir el valor de vuelta. Esto significa que hay un margen de error considerable:

```C
uint32_t temp = pwm0.ctl.read();
temp |= PWM0_CTL_GLOBALSYNC0;
pwm0.ctl.write(temp);
uint32_t temp2 = pwm0.enable.read();
temp2 |= PWM0_ENABLE_PWM4EN;
pwm0.enable.write(temp); // ¡Oh oh! ¡Variable equivocada!
```

[svd2rust]: https://crates.io/crates/svd2rust
[documentación de tm4c123x R]: https://docs.rs/tm4c123x/0.7.0/tm4c123x/pwm0/ctl/struct.R.html
[Documentación de tm4c123x W]: https://docs.rs/tm4c123x/0.7.0/tm4c123x/pwm0/ctl/struct.W.html

## Usando una crate HAL 

La crate HAL de un chip suele funcionar implementando una característica personalizada para las estructuras sin procesar expuestas por el PAC. Esta característica suele definir una función llamada `constrain()` para periféricos individuales o `split()` para puertos GPIO con múltiples pines. Esta función consume la estructura sin procesar subyacente del periférico y devuelve un nuevo objeto con una API de nivel superior. Esta API también puede, por ejemplo, que la función `new` del puerto serie solicite un préstamo de alguna estructura `Clock`, que solo se puede generar llamando a la función que configura los PLL y configura todas las frecuencias de reloj. De esta forma, es estáticamente imposible crear un objeto de puerto serie sin haber configurado primero las velocidades de reloj, o que el objeto de puerto serie convierta erróneamente la velocidad en baudios a ticks de reloj. Algunas cajas incluso definen características especiales para los estados en los que puede estar cada pin GPIO, lo que requiere que el usuario configure un pin en el estado correcto (por ejemplo, seleccionando el modo de función alternativo adecuado) antes de pasarlo a Periférico. ¡Todo ello sin coste de tiempo de ejecución!

Vamos a ver un ejemplo:

```rust,ignore
#![no_std]
#![no_main]

use panic_halt as _; // panic handler

use cortex_m_rt::entry;
use tm4c123x_hal as hal;
use tm4c123x_hal::prelude::*;
use tm4c123x_hal::serial::{NewlineMode, Serial};
use tm4c123x_hal::sysctl;

#[entry]
fn main() -> ! {
    let p = hal::Peripherals::take().unwrap();
    let cp = hal::CorePeripherals::take().unwrap();

    // Envuelve la estructura SYSCTL en un objeto con una API de capa superior
    let mut sc = p.SYSCTL.constrain();
    // Elija nuestra configuración de oscilación
    sc.clock_setup.oscillator = sysctl::Oscillator::Main(
        sysctl::CrystalFrequency::_16mhz,
        sysctl::SystemClock::UsePll(sysctl::PllOutputFrequency::_80_00mhz),
    );
    // Configure el PLL con esos ajustes
    let clocks = sc.clock_setup.freeze();

    // Envuelve la estructura GPIO_PORTA en un objeto con una API de capa superior.
    // Tenga en cuenta que necesita tomar prestado `sc.power_control` para poder 
    // encender el periferico GPIO automáticamente.
    let mut porta = p.GPIO_PORTA.split(&sc.power_control);

    // Activar el UART.
    let uart = Serial::uart0(
        p.UART0,
        // El pin de transmisión
        porta
            .pa1
            .into_af_push_pull::<hal::gpio::AF1>(&mut porta.control),
        // El pin de recepcion
        porta
            .pa0
            .into_af_push_pull::<hal::gpio::AF1>(&mut porta.control),
        // No RTS o CTS requerido
        (),
        (),
        // La velocidad de bauds 
        115200_u32.bps(),
        // Manejo de salida
        NewlineMode::SwapLFtoCRLF,
        // Necesitamos las velocidades de reloj para calcular los divisores de la velocidad en bauds.
        &clocks,
        // Necesitamos esto para encender el periférico UART.
        &sc.power_control,
    );

    loop {
        writeln!(uart, "Hello, World!\r\n").unwrap();
    }
}
```
