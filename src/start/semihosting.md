# Semialojamiento (Semihosting)

El semihosting es un mecanismo que permite a los dispositivos embebidos realizar operaciones de E/S en el host y se utiliza principalmente para registrar mensajes en la consola del host. El semihosting requiere una sesión de depuración y prácticamente nada más (¡sin cables adicionales!), por lo que es muy práctico. La desventaja es su lentitud: cada operación de escritura puede tardar varios milisegundos, dependiendo del depurador de hardware que se utilice (por ejemplo, ST-Link).

La crate [`cortex-m-semihosting`] crate proporciona una API para realizar operaciones de semihosting
en dispositivos Cortex-M. El programa siguiente es la versión de semihosting de "¡Hola, mundo!":

[`cortex-m-semihosting`]: https://crates.io/crates/cortex-m-semihosting

```rust,ignore
#![no_main]
#![no_std]

use panic_halt as _;

use cortex_m_rt::entry;
use cortex_m_semihosting::hprintln;

#[entry]
fn main() -> ! {
    hprintln!("Hello, world!").unwrap();

    loop {}
}
```

Si ejecutas este programa en hardware, verás el mensaje "¡Hola, mundo!" en los registros de OpenOCD.

``` text
$ openocd
(..)
Hello, world!
(..)
```

Primero debes habilitar el semihosting en OpenOCD desde GDB:
``` console
(gdb) monitor arm semihosting enable
semihosting is enabled
```

QEMU entiende las operaciones de semihosting, por lo que el programa anterior también funcionará con
`qemu-system-arm` sin necesidad de iniciar una sesión de depuración. Tenga en cuenta que deberá pasar la opción `-semihosting-config` a QEMU para habilitar la compatibilidad con semihosting
estas banderas ya están incluidas en el archivo `.cargo/config.toml` de la plantilla

``` text
$ # this program will block the terminal
$ cargo run
     Running `qemu-system-arm (..)
Hello, world!
```

También existe una operación de semihosting `exit` que se puede usar para finalizar el proceso QEMU. Importante: **no** utilice `debug::exit` en hardware; esta función
puede dañar su sesión de OpenOCD y no podrá depurar más programas hasta que reinicie la sesión.

```rust,ignore
#![no_main]
#![no_std]

use panic_halt as _;

use cortex_m_rt::entry;
use cortex_m_semihosting::debug;

#[entry]
fn main() -> ! {
    let roses = "blue";

    if roses == "red" {
        debug::exit(debug::EXIT_SUCCESS);
    } else {
        debug::exit(debug::EXIT_FAILURE);
    }

    loop {}
}
```

``` text
$ cargo run
     Running `qemu-system-arm (..)

$ echo $?
1
```

Un último consejo: puedes configurar el comportamiento en caso de pánico a `exit(EXIT_FAILURE)`. Esto te permitirá escribir pruebas de ejecución y aprobación `no_std` que puedes ejecutar en QEMU.

Para mayor comodidad, la crate `panic-semihosting` tiene una función "exit" que cuando
se activa, invoca `exit(EXIT_FAILURE)` después de registrar el mensaje de pánico en el
stderr (salida de errores) del host.

```rust,ignore
#![no_main]
#![no_std]

use panic_semihosting as _; // features = ["exit"]

use cortex_m_rt::entry;
use cortex_m_semihosting::debug;

#[entry]
fn main() -> ! {
    let roses = "blue";

    assert_eq!(roses, "red");

    loop {}
}
```

``` text
$ cargo run
     Running `qemu-system-arm (..)
panicked at 'assertion failed: `(left == right)`
  left: `"blue"`,
 right: `"red"`', examples/hello.rs:15:5

$ echo $?
1
```
**NOTA**: Para habilitar esta función en `panic-semihosting`, edite la sección de dependencias de su archivo `Cargo.toml` donde se especifica `panic-semihosting` con:

``` toml
panic-semihosting = { version = "VERSION", features = ["exit"] }
```

donde `VERSION` es la versión deseada. Para obtener más información sobre las dependencias, consulte la sección [`especificación de dependencias`] del libro de Cargo.

[`especificación de dependencias`]:
https://doc.rust-lang.org/cargo/reference/specifying-dependencies.html

