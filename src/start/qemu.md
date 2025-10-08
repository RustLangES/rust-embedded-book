# QEMU

Comenzaremos a escribir un programa para el [LM3S6965], un microcontrolador Cortex-M3.
Hemos elegido esto como nuestro objetivo inicial porque [puede ser emulado](https://wiki.qemu.org/Documentation/Platforms/ARM#Supported_in_qemu-system-arm) usando QEMU
Así que no es necesario manipular el hardware en esta sección y podemos centrarnos en las herramientas y el proceso de desarrollo.

[LM3S6965]: http://www.ti.com/product/LM3S6965

**IMPORTANTE**
Usaremos el nombre "app" para el nombre del proyecto en este tutorial.
Cada vez que veas la palabra "app", debes reemplazarla con el nombre que seleccionaste para tu proyecto.
O bien, también podrías nombrar tu proyecto "app" y evitar las sustituciones.

## Creando un programa Rust no estandar

Usaremos la plantilla del proyecto [`cortex-m-quickstart`] para generar un nuevo proyecto a partir de ella.
El proyecto creado contendrá una aplicación básica: Un buen punto de partida para una nueva 
aplicación de Rust embebido.
 Además, el proyecto contendrá un directorio de ejemplos, con varias aplicaciones separadas, Destacando
algunas de las principales funciones de Rust embebido.

[`cortex-m-quickstart`]: https://github.com/rust-embedded/cortex-m-quickstart

### Usando `cargo-generate`
Primero instala cargo-generate
```console
cargo install cargo-generate
```
Luego genera un nuevo proyecto 
```console
cargo generate --git https://github.com/rust-embedded/cortex-m-quickstart
```

```text
 Project Name: app
 Creating project called `app`...
 Done! New project created /tmp/app
```

```console
cd app
```

### Usando `git`

Clona el repositorio

```console
git clone https://github.com/rust-embedded/cortex-m-quickstart app
cd app
```

Y luego rellene los marcadores de posición en el archivo `Cargo.toml`

```toml
[package]
authors = ["{{authors}}"] # "{{authors}}" -> "John Smith"
edition = "2018"
name = "{{project-name}}" # "{{project-name}}" -> "app"
version = "0.1.0"

# ..

[[bin]]
name = "{{project-name}}" # "{{project-name}}" -> "app"
test = false
bench = false
```

### Usando neither

Obtenga la última snapshot de la plantilla `cortex-m-quickstart` y extráigala.

```console
curl -LO https://github.com/rust-embedded/cortex-m-quickstart/archive/master.zip
unzip master.zip
mv cortex-m-quickstart-master app
cd app
```

O puedes navegar hasta [`cortex-m-quickstart`], Haga clic en el botón verde "Clonar o descargar" y luego haga clic en "Descargar ZIP".

Luego, complete los marcadores de posición en el archivo `Cargo.toml` como se hizo en la segunda parte de la versión "Usando `git`".

## Descripción general del programa

Para mayor comodidad, aquí se encuentran las partes más importantes del código fuente. `src/main.rs`:

```rust,ignore
#![no_std]
#![no_main]

use panic_halt as _;

use cortex_m_rt::entry;

#[entry]
fn main() -> ! {
    loop {
        // tu código va aquí
    }
}
```

Este programa es un poco diferente de un programa Rust estándar, así que veámoslo más de cerca.

`#![no_std]` indica que este programa *no* se vinculará al paquete estándar,
`std`. En lugar de ello, se vinculará a su subconjunto: la crate "core".

`#![no_main]` Indica que este programa no utilizará la interfaz estandar`main`
que la mayoría de los programas Rust usan. La razón principal para usar`no_main` es que usando la interfaz `main` en `no_std` El contexto requiere
nightly.

`use panic_halt as _;`. Esta caja proporciona un `panic_handler` que define el comportamiento del programa en caso de panico. Lo explicaremos con más detalle en el capitulo del libro
[Panico](panicking.md) 

[`#[entry]`][entry] es un atributo proporcionado por la crate [`cortex-m-rt`] que se utiliza para marcar el punto de entrada del programa. Como no utilizamos la interfaz estándar`main`
Necesitamos otra forma de indicar el punto de entrada del programa y esa sería `#[entry]`.

[entry]: https://docs.rs/cortex-m-rt-macros/latest/cortex_m_rt_macros/attr.entry.html
[`cortex-m-rt`]: https://crates.io/crates/cortex-m-rt

`fn main() -> !`. Nuestro programa será el *único* proceso que se ejecuta en el hardware de destino, así que no queremos que finalice. Usamos una
 [función divergente](https://doc.rust-lang.org/rust-by-example/fn/diverging.html), la parte -> ! en la firma de la función garantiza en tiempo de compilación que ese será el caso

## compilación cruzada

El siguiente paso es compilar de forma *cruzada* el programa para la arquitectura Cortex-M3.
Eso es tan simple como correr `cargo build --target $TRIPLE`
Si sabes cuál podria ser el target de compilación (`$TRIPLE`). 
Afortunadamente, el  `.cargo/config.toml` en la plantilla tiene la respuesta:

```console
tail -n6 .cargo/config.toml
```

```toml
[build]
# Elija UNO de estos targets de compilación
# target = "thumbv6m-none-eabi"    # Cortex-M0 y Cortex-M0+
target = "thumbv7m-none-eabi"    # Cortex-M3
# target = "thumbv7em-none-eabi"   # Cortex-M4 y Cortex-M7 (sin FPU)
# target = "thumbv7em-none-eabihf" # Cortex-M4F y Cortex-M7F (con FPU)
```

Para realizar una compilación cruzada para la arquitectura Cortex-M3 tenemos que usar
`thumbv7m-none-eabi`. Ese target no se instala automáticamente al instalar las herramientas de Rust. Si aún no lo has hecho, sería un buen momento para añadirlo.
``` console
rustup target add thumbv7m-none-eabi
```
Dado que el target de compilación `thumbv7m-none-eabi` se ha establecido como predeterminado en el archivo `.cargo/config.toml`, 
los dos comandos siguientes hacen lo mismo:

```console
cargo build --target thumbv7m-none-eabi
cargo build
```

## inspeccionando

Ahora tenemos un binario ELF no nativo en `target/thumbv7m-none-eabi/debug/app`. 
Podemos inspeccionarlo usando `cargo-binutils`.

Con `cargo-readobj` podemos imprimir los headers ELF para confirmar que se trata de un binario ARM.
``` console
cargo readobj --bin app -- --file-headers
```

Tenga en cuenta que:
* `--bin app` es una forma abreviada de inspeccionar el binario en target/$TRIPLE/debug/app`
* `--bin app` también (re)compilará el binario, si es necesario


``` text
ELF Header:
  Magic:   7f 45 4c 46 01 01 01 00 00 00 00 00 00 00 00 00
  Class:                             ELF32
  Data:                              2's complement, little endian
  Version:                           1 (current)
  OS/ABI:                            UNIX - System V
  ABI Version:                       0x0
  Type:                              EXEC (Executable file)
  Machine:                           ARM
  Version:                           0x1
  Entry point address:               0x405
  Start of program headers:          52 (bytes into file)
  Start of section headers:          153204 (bytes into file)
  Flags:                             0x5000200
  Size of this header:               52 (bytes)
  Size of program headers:           32 (bytes)
  Number of program headers:         2
  Size of section headers:           40 (bytes)
  Number of section headers:         19
  Section header string table index: 18
```

`cargo-size` puede imprimir el tamaño de las secciones del enlazador del binario.


```console
cargo size --bin app --release -- -A
```
Usamos `--release` para inspeccionar la versión optimizada

``` text
app  :
section             size        addr
.vector_table       1024         0x0
.text                 92       0x400
.rodata                0       0x45c
.data                  0  0x20000000
.bss                   0  0x20000000
.debug_str          2958         0x0
.debug_loc            19         0x0
.debug_abbrev        567         0x0
.debug_info         4929         0x0
.debug_ranges         40         0x0
.debug_macinfo         1         0x0
.debug_pubnames     2035         0x0
.debug_pubtypes     1892         0x0
.ARM.attributes       46         0x0
.debug_frame         100         0x0
.debug_line          867         0x0
Total              14570
```

> Un repaso sobre las secciones de enlace ELF
>
> - `.text` contiene las instrucciones del programa
> - `.rodata` contiene valores constantes como cadenas
> - `.data` contiene variables asignadas estáticamente cuyos valores iniciales 
>    *no* son cero
> - `.bss` también contiene variables asignadas estáticamente cuyos valores iniciales
>    *son* cero
> - `.vector_table` es una sección *no* estándar que usamos para almacenar la tabla de 
>    vectores (interrupciones)
> -  Las secciones `.ARM.attributes` y `.debug_*` contienen metadatos y 
>    *no* se cargarán en el destino al flashear el binario.

**IMPORTANTE**: Los archivos ELF contienen metadatos como información de depuración, por lo que su *tamaño en disco* no refleja con precisión el espacio que ocupará el programa al instalarse en un dispositivo. *Siempre* use `cargo-size` para comprobar el tamaño real de un binario.

`cargo-objdump` Se puede utilizar para desmontar el binario.

```console
cargo objdump --bin app --release -- --disassemble --no-show-raw-insn --print-imm-hex
```

> **NOTA** Si el comando anterior indica un error de "Argumento de línea de comando desconocido",
> consulte el siguiente informe de error:  
> https://github.com/rust-embedded/book/issues/269

> **NOTA**: Esta salida puede variar según el sistema. Las nuevas versiones de rustc, LLVM y las bibliotecas
> pueden generar diferentes ensamblados. Hemos truncado algunas instrucciones para que el fragmento sea pequeño.

```text
app:  file format ELF32-arm-little

Disassembly of section .text:
main:
     400: bl  #0x256
     404: b #-0x4 <main+0x4>

Reset:
     406: bl  #0x24e
     40a: movw  r0, #0x0
     < .. truncated any more instructions .. >

DefaultHandler_:
     656: b #-0x4 <DefaultHandler_>

UsageFault:
     657: strb  r7, [r4, #0x3]

DefaultPreInit:
     658: bx  lr

__pre_init:
     659: strb  r7, [r0, #0x1]

__nop:
     65a: bx  lr

HardFaultTrampoline:
     65c: mrs r0, msp
     660: b #-0x2 <HardFault_>

HardFault_:
     662: b #-0x4 <HardFault_>

HardFault:
     663: <unknown>
```

## Funcionamiento

A continuación, veamos cómo ejecutar un programa embebido en QEMU. Esta vez usaremos el ejemplo `hello`, que realmente hace algo.

Para mayor comodidad, aquí está el código fuente de `examples/hello.rs`:

```rust,ignore
//¡Imprime "Hola, mundo!" en la consola del host usando semihosting

#![no_main]
#![no_std]

use panic_halt as _;

use cortex_m_rt::entry;
use cortex_m_semihosting::{debug, hprintln};

#[entry]
fn main() -> ! {
    hprintln!("Hello, world!").unwrap();

    // salir de QEMU
    // NOTA no ejecute esto en el hardware; puede dañar el estado de OpenOCD
    debug::exit(debug::EXIT_SUCCESS);

    loop {}
}
```

Este programa usa un método llamado semihosting para imprimir texto en la consola del host.
 Al usar hardware real, esto requiere una sesión de depuración, pero al usar QEMU, funciona perfectamente.

Comencemos compilando el ejemplo:

```console
cargo build --example hello
```

El binario de salida estará ubicado en
`target/thumbv7m-none-eabi/debug/examples/hello`.

Para ejecutar este binario en QEMU, ejecute el siguiente comando:

```console
qemu-system-arm \
  -cpu cortex-m3 \
  -machine lm3s6965evb \
  -nographic \
  -semihosting-config enable=on,target=native \
  -kernel target/thumbv7m-none-eabi/debug/examples/hello
```

```text
Hello, world!
```

El comando debería salir correctamente (código de salida = 0) después de imprimir el texto. 
En *nix, puedes comprobarlo con el siguiente comando:

```console
echo $?
```

```text
0
```

Analicemos ese comando QEMU:

- `qemu-system-arm`. Este es el emulador QEMU. Existen algunas variantes de estos binarios QEMU; este realiza una emulación completa del *sistema* de máquinas *ARM*, de ahí su nombre.

- `-cpu cortex-m3`. Esto le indica a QEMU que emule una CPU Cortex-M3. Especificar el modelo de CPU nos permite detectar algunos errores de compilación: por ejemplo, ejecutar un programa compilado para Cortex-M4F, que tiene una FPU de hardware, generará un error en QEMU durante su ejecución.

- `-machine lm3s6965evb`. Esto le indica a QEMU que emule el LM3S6965EVB, 
una placa de evaluación que contiene un microcontrolador LM3S6965.

- `-nographic`. Esto le indica a QEMU que no inicie su GUI.

- `-semihosting-config (..)`. Esto indica a QEMU que habilite el semihosting. 
El semihosting permite al dispositivo emulado, entre otras cosas, usar la salida estándar del host, 
la salida estándar del servidor y la entrada estándar del servidor, y crear archivos en el host.

- `-kernel $file`. Esto le indica a QEMU qué binario cargar y ejecutar en la máquina emulada.

¡Escribir ese largo comando QEMU es demasiado trabajo! Podemos configurar un ejecutor personalizado para simplificar el proceso.
 `.cargo/config.toml` tiene un ejecutor comentado que invoca QEMU; descomentémoslo:

```console
head -n3 .cargo/config.toml
```

```toml
[target.thumbv7m-none-eabi]
# Descomente esto para hacer que `cargo run` ejecute programas en QEMU
runner = "qemu-system-arm -cpu cortex-m3 -machine lm3s6965evb -nographic -semihosting-config enable=on,target=native -kernel"
```

Este ejecutor solo se aplica al target `thumbv7m-none-eabi`, que es nuestro target de compilación predeterminado. 
Ahora `cargo run` compilará el programa y lo ejecutará en QEMU:

```console
cargo run --example hello --release
```

```text
   Compiling app v0.1.0 (file:///tmp/app)
    Finished release [optimized + debuginfo] target(s) in 0.26s
     Running `qemu-system-arm -cpu cortex-m3 -machine lm3s6965evb -nographic -semihosting-config enable=on,target=native -kernel target/thumbv7m-none-eabi/release/examples/hello`
Hello, world!
```

## Depuración

La depuración es fundamental para el desarrollo integrado. Veamos cómo se realiza.

Depurar un dispositivo embebido implica una depuración remota, 
ya que el programa que queremos depurar no estará ejecutándose en la máquina donde corre el depurador (GDB o LLDB).

La depuración remota implica un cliente y un servidor. En una configuración QEMU, 
el cliente será un proceso GDB (o LLDB) y el servidor será el proceso QEMU que también ejecuta el programa integrado.

En esta sección usaremos el ejemplo `hello` que ya compilamos.

El primer paso de depuración es iniciar QEMU en modo de depuración:

```console
qemu-system-arm \
  -cpu cortex-m3 \
  -machine lm3s6965evb \
  -nographic \
  -semihosting-config enable=on,target=native \
  -gdb tcp::3333 \
  -S \
  -kernel target/thumbv7m-none-eabi/debug/examples/hello
```

Este comando no imprimirá nada en la consola y bloqueará la terminal. 
Esta vez, hemos pasado dos indicadores adicionales:

- `-gdb tcp::3333`. Esto le indica a QEMU que espere una conexión GDB en el puerto TCP 3333.

- `-S`. Esto le indica a QEMU que congele la máquina al iniciar. Sin esto, 
el programa habría llegado al final del proceso principal antes de que pudiéramos iniciar el depurador.

A continuación, iniciamos GDB en otra terminal y le indicamos que cargue los símbolos de depuración del ejemplo:

```console
gdb-multiarch -q target/thumbv7m-none-eabi/debug/examples/hello
```

**NOTA**: Es posible que necesite otra versión de gdb en lugar de `gdb-multiarch`, 
según la que haya instalado en el capítulo de instalación. 
También podría ser `arm-none-eabi-gdb` o simplemente `gdb`.

Luego, dentro del shell GDB, nos conectamos a QEMU, que está esperando una conexión en el puerto TCP 3333.

```console
target remote :3333
```

```text
Remote debugging using :3333
Reset () at $REGISTRY/cortex-m-rt-0.6.1/src/lib.rs:473
473     pub unsafe extern "C" fn Reset() -> ! {
```


Verás que el proceso se detiene y que el contador del programa apunta a una función llamada "Reset". 
Este es el controlador de reinicio: lo que los núcleos Cortex-M ejecutan al arrancar.

>  Tenga en cuenta que en algunas configuraciones, en lugar de mostrar la línea `Reset () en $REGISTRY/cortex-m-rt-0.6.1/src/lib.rs:473` como se muestra arriba, gdb puede imprimir algunas advertencias como:
>
>`core::num::bignum::Big32x40::mul_small () at src/libcore/num/bignum.rs:254`
> `    src/libcore/num/bignum.rs: No existe el archivo o directorio`
> 
> Es un fallo conocido. Puedes ignorar esas advertencias sin problema; Lo más probable es que la ejecución se encuentre en Reset().


Este controlador de reinicio llamará a nuestra función principal. 
Vamos a saltarnos este paso usando un punto de interrupción y el comando `continue`. 
Para establecer el punto de interrupción, primero veamos dónde queremos interrumpir nuestro código con el comando `list`.

```console
list main
```
Esto mostrará el código fuente, del archivo examples/hello.rs.

```text
6       use panic_halt as _;
7
8       use cortex_m_rt::entry;
9       use cortex_m_semihosting::{debug, hprintln};
10
11      #[entry]
12      fn main() -> ! {
13          hprintln!("Hello, world!").unwrap();
14
15          // salir de QEMU
```
Nos gustaría agregar un punto de interrupción antes de "¡Hola, mundo!", que está en la línea 13. Lo hacemos con el comando `break`:

```console
break 13
```
Ahora podemos indicarle a gdb que se ejecute hasta nuestra función principal, con el comando `continue`:

```console
continue
```

```text
Continuing.

Breakpoint 1, hello::__cortex_m_rt_main () at examples\hello.rs:13
13          hprintln!("Hello, world!").unwrap();
```

Ya estamos cerca del código que imprime "¡Hola mundo!". Avancemos con el comando `next`.

``` console
next
```

```text
16          debug::exit(debug::EXIT_SUCCESS);
```

En este punto deberías ver "¡Hola, mundo!" impreso en la terminal que ejecuta `qemu-system-arm`.

```text
$ qemu-system-arm (..)
Hello, world!
```

Llamar a `next` nuevamente finalizará el proceso QEMU.

```console
next
```

```text
[Inferior 1 (Remote target) exited normally]
```

Ahora puede salir de la sesión GDB.

``` console
quit
```
