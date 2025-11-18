# produciendo un pánico (Panicking)

El pánico es una parte fundamental del lenguaje Rust. Las operaciones integradas, como la indexación,
se someten a comprobaciones de seguridad de memoria en tiempo de ejecución. Cuando se intenta indexar fuera de los límites,
esto provoca un pánico.

En la biblioteca estándar, el pánico tiene un comportamiento definido: desenrolla la pila (stack) del hilo que entró en pánico, a menos que el usuario haya optado por abortar el programa cuando ocurre un pánico.

En programas sin biblioteca estándar, el comportamiento ante un pánico queda
indefinido. Se puede definir un comportamiento declarando una función `#[panic_handler]`.
Esta función debe aparecer exactamente *una vez* en el grafo de dependencias del programa,
y debe tener la siguiente firma: `fn(&PanicInfo) -> !`, donde [`PanicInfo`]
es una estructura que contiene información sobre la ubicación del pánico.


[`PanicInfo`]: https://doc.rust-lang.org/core/panic/struct.PanicInfo.html

Dado que los sistemas embebidos abarcan desde sistemas de cara al usuario hasta sistemas críticos para la seguridad (que no pueden fallar), no existe un comportamiento de pánico universal, pero sí muchos comportamientos de uso común. Estos comportamientos comunes se han empaquetado en crates que definen la función `#[panic_handler]`. Algunos ejemplos son:

- [`panic-abort`]. El pánico provoca la ejecución de la instrucción de aborto.
- [`panic-halt`]. Un pánico provoca que el programa, o el hilo actual, se detenga al entrar en un bucle infinito.
- [`panic-itm`]. El mensaje de pánico se registra utilizando el ITM, un periférico específico para ARM Cortex-M.
- [`panic-semihosting`]. El mensaje de pánico se registra en el host utilizando la técnica de semihosting.

[`panic-abort`]: https://crates.io/crates/panic-abort
[`panic-halt`]: https://crates.io/crates/panic-halt
[`panic-itm`]: https://crates.io/crates/panic-itm
[`panic-semihosting`]: https://crates.io/crates/panic-semihosting

Es posible que encuentres aún más crates buscando la palabra clave [`panic-handler`] en crates.io.

[`panic-handler`]: https://crates.io/keywords/panic-handler

Un programa puede seleccionar uno de estos comportamientos simplemente vinculándolo al crate correspondiente.
El hecho de que el comportamiento de pánico se exprese en el código fuente de
una aplicación como una sola línea de código no solo es útil como documentación, sino que
también se puede usar para cambiar el comportamiento de pánico según el perfil de compilación.
Por ejemplo:

``` rust,ignore
#![no_main]
#![no_std]

// perfil de desarrollo: Es más fácil depurar los errores críticos; se puede establecer un punto de interrupción en `rust_begin_unwind`.
#[cfg(debug_assertions)]
use panic_halt as _;

// Perfil de lanzamiento: minimizar el tamaño binario de la aplicación.
#[cfg(not(debug_assertions))]
use panic_abort as _;

// ..
```

En este ejemplo, la crate enlaza con la crate `panic-halt` cuando se compila con el
perfil de desarrollo (`cargo build`), pero enlaza con la crate `panic-abort` cuando se compila
con el perfil de lanzamiento (`cargo build --release`).

> La forma `use panic_abort as _;` de la instrucción `use` se 
> utiliza para asegurar que el manejador de pánico `panic_abort` se incluya en nuestro ejecutable final, dejando claro al compilador que no utilizaremos explícitamente nada de la crate. 
> Sin el cambio de nombre `as _`, el compilador advertiría de una importación no utilizada.
> En ocasiones, se puede encontrar `extern crate panic_abort`, un estilo antiguo utilizado antes de la edición de Rust de 2018, 
> y que ahora solo debe usarse para crates "sysroot" (los que se distribuyen con Rust), como `proc_macro`, `alloc`, `std` y `test`.

## Un ejemplo

Aquí tienes un ejemplo que intenta acceder a un índice de un array más allá de su longitud. La operación resulta en un error grave.

```rust,ignore
#![no_main]
#![no_std]

use panic_semihosting as _;

use cortex_m_rt::entry;

#[entry]
fn main() -> ! {
    let xs = [0, 1, 2];
    let i = xs.len();
    let _y = xs[i]; // out of bounds access

    loop {}
}
```

Este ejemplo eligió el comportamiento `panic-semihosting` que imprime el mensaje de pánico
en la consola del host utilizando semihosting.

``` text
$ cargo run
     Running `qemu-system-arm -cpu cortex-m3 -machine lm3s6965evb (..)
panicked at 'index out of bounds: the len is 3 but the index is 4', src/main.rs:12:13
```

Puedes intentar cambiar el comportamiento a `panic-halt` y confirmar que no se imprime ningún mensaje en ese caso.
