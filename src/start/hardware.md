# Hardware

A estas alturas, ya deberías estar familiarizado con las herramientas 
y el proceso de desarrollo. En esta sección, hablaremos de hardware real; 
el proceso se mantendrá prácticamente igual. Profundicemos.

## Conoce tu hardware

Antes de comenzar, es necesario identificar algunas características del dispositivo de destino, 
ya que se utilizarán para configurar el proyecto:

- El núcleo ARM, Cortex-M3 por ejemplo. 

- ¿El núcleo ARM incluye una FPU? Los núcleos Cortex-M4**F** y Cortex-M7**F** sí la incluyen.

- ¿Cuánta memoria Flash y RAM tiene el dispositivo de destino? Por ejemplo, 
  256 KiB de Flash y 32 KiB de RAM.

- ¿Dónde se asignan la memoria flash y la RAM en el espacio de direcciones? 
  Por ejemplo, la RAM suele ubicarse en la dirección `0x2000_0000`.

Puede encontrar esta información en la hoja de datos o en el manual de referencia de su dispositivo.

En esta sección, utilizaremos nuestro hardware de referencia, el STM32F3DISCOVERY.
Esta placa contiene un microcontrolador STM32F303VCT6. Este microcontrolador tiene:

- Un núcleo Cortex-M4F que incluye una única FPU de precisión

- 256 KiB de Flash ubicado en la dirección 0x0800_0000.

- 40 KiB de RAM ubicados en la dirección 0x2000_0000. (Hay otra región de RAM,
  pero para simplificar, la ignoraremos).

## Configurando

Empezaremos desde cero con una nueva instancia de plantilla. 
Consulta la sección anterior sobre [QEMU] para repasar cómo hacerlo sin ``cargo-generate`.

[QEMU]: qemu.md

``` text
$ cargo generate --git https://github.com/rust-embedded/cortex-m-quickstart
 Project Name: app
 Creating project called `app`...
 Done! New project created /tmp/app

$ cd app
```

El paso número uno es establecer un destino de compilación predeterminado en `.cargo/config.toml`.

``` console
tail -n5 .cargo/config.toml
```

``` toml
# Pick ONE of these compilation targets
# target = "thumbv6m-none-eabi"    # Cortex-M0 and Cortex-M0+
# target = "thumbv7m-none-eabi"    # Cortex-M3
# target = "thumbv7em-none-eabi"   # Cortex-M4 and Cortex-M7 (no FPU)
target = "thumbv7em-none-eabihf" # Cortex-M4F and Cortex-M7F (with FPU)
```

Usaremos `thumbv7em-none-eabihf` ya que cubre el núcleo Cortex-M4F.
> **NOTA**: Como recordarás del capítulo anterior, debemos instalar todos los objetivos, 
> y este es uno nuevo. Así que no olvides ejecutar el proceso de instalación `
> rustup target add thumbv7em-none-eabihf` para este objetivo.

El segundo paso es ingresar la información de la región de memoria en el archivo `memory.x`.

``` text
$ cat memory.x
/* Linker script for the STM32F303VCT6 */
MEMORY
{
  /* NOTE 1 K = 1 KiBi = 1024 bytes */
  FLASH : ORIGIN = 0x08000000, LENGTH = 256K
  RAM : ORIGIN = 0x20000000, LENGTH = 40K
}
```
>  **NOTA**: Si por alguna razón modificó el archivo `memory.x` después de haber realizado 
> la primera compilación de un objetivo de compilación específico, ejecute `cargo clean` 
> antes de `cargo build`, ya que `cargo build` podría no rastrear las actualizaciones de `memory.x`.

Comenzaremos nuevamente con el ejemplo de hola, pero primero tenemos que hacer un pequeño cambio.

En `examples/hello.rs`, asegúrese de que la llamada `debug::exit()` esté comentada o eliminada. Solo se usa para ejecutarse en QEMU.

```rust,ignore
#[entry]
fn main() -> ! {
    hprintln!("Hello, world!").unwrap();

    // salir de QEMU
    // NOTA no ejecute esto en el hardware; puede dañar el estado de OpenOCD
    // debug::exit(debug::EXIT_SUCCESS);

    loop {}
}
```

Ahora puedes compilar programas de forma cruzada usando `cargo build`
e inspeccionar los binarios con `cargo-binutils`, como antes. El paquete `cortex-m-rt`
gestiona todo el proceso necesario para que tu chip funcione, ya que, afortunadamente, 
prácticamente todas las CPU Cortex-M arrancan de la misma manera.

``` console
cargo build --example hello
```

## Depuración
La depuración será un poco diferente. De hecho, los primeros pasos pueden variar según el dispositivo de destino. 
En esta sección, mostraremos los pasos necesarios para depurar un programa que se ejecuta en el STM32F3DISCOVERY. 
Esto sirve como referencia; para obtener información específica sobre la depuración del dispositivo, consulte [el Debugonomicon](https://github.com/rust-embedded/debugonomicon).

Como antes, realizaremos depuración remota y el cliente será un proceso GDB. 
1Sin embargo, esta vez, el servidor será OpenOCD.

Como se hizo durante la sección [verify], conecte la placa de descubrimiento a su computadora portátil/PC y verifique que el encabezado ST-LINK esté completo.

[verify]: ../intro/install/verify.md

En una terminal, ejecute `openocd` para conectarse al ST-LINK en la placa Discovery.
Ejecute este comando desde el root del template; `openocd` recuperará el archivo `openocd.cfg`, 
que indica qué archivo de interfaz y archivo de destino usar.

``` console
cat openocd.cfg
```

``` text
# Muestra de configuración OpenOCD para la placa de desarrollo STM32F3DISCOVERY. 

# Dependiendo de la revisión de hardware que tengas, tendrás que elegir UNO de estas
# Interfaces. En cualquier momento solo se debe comentar una interfaz.

# Revisión C (revisión mas nueva)
source [find interface/stlink.cfg]

# Revisión A y B (revisiones viejas)
# source [find interface/stlink-v2.cfg]

source [find target/stm32f3x.cfg]
```

> **NOTA** Si descubrió que tiene una revisión anterior de la 
> placa de descubrimiento durante la sección [verificar], debe modificar el 
> 1archivo `openocd.cfg` en este punto para usar `interface/stlink-v2.cfg`.

``` text
$ openocd
Open On-Chip Debugger 0.10.0
Licensed under GNU GPL v2
For bug reports, read
        http://openocd.org/doc/doxygen/bugs.html
Info : auto-selecting first available session transport "hla_swd". To override use 'transport select <transport>'.
adapter speed: 1000 kHz
adapter_nsrst_delay: 100
Info : The selected transport took over low-level target control. The results might differ compared to plain JTAG/SWD
none separate
Info : Unable to match requested speed 1000 kHz, using 950 kHz
Info : Unable to match requested speed 1000 kHz, using 950 kHz
Info : clock speed 950 kHz
Info : STLINK v2 JTAG v27 API v2 SWIM v15 VID 0x0483 PID 0x374B
Info : using stlink api v2
Info : Target voltage: 2.913879
Info : stm32f3x.cpu: hardware has 6 breakpoints, 4 watchpoints
```

En otra terminal ejecute GDB, también desde el root de la template.

``` text
gdb-multiarch -q target/thumbv7em-none-eabihf/debug/examples/hello
```

**NOTA**: Al igual que antes, podría necesitar otra versión de gdb en lugar de `gdb-multiarch`, 
dependiendo de la que haya instalado en el capítulo de instalación. 
También podría ser `arm-none-eabi-gdb` o simplemente `gdb`.

A continuación, conecte GDB a OpenOCD, que está esperando una conexión TCP en el puerto 3333.

``` console
(gdb) target remote :3333
Remote debugging using :3333
0x00000000 in ?? ()
```

Ahora proceda a *cargar* el programa en el microcontrolador usando el 
comando `load`.

``` console
(gdb) load
Loading section .vector_table, size 0x400 lma 0x8000000
Loading section .text, size 0x1518 lma 0x8000400
Loading section .rodata, size 0x414 lma 0x8001918
Start address 0x08000400, load size 7468
Transfer rate: 13 KB/sec, 2489 bytes/write.
```

El programa ya está cargado. Este programa usa semihosting, así que antes de realizar cualquier llamada a semihosting, 
debemos indicarle a OpenOCD que lo habilite. Puedes enviar comandos a OpenOCD usando el comando `monitor`.

``` console
(gdb) monitor arm semihosting enable
semihosting is enabled
```

> Puede ver todos los comandos de OpenOCD invocando el comando `monitor help`.

Como antes, podemos saltar directamente a `main` usando un punto de interrupción y el 
comando `continue`.

``` console
(gdb) break main
Breakpoint 1 at 0x8000490: file examples/hello.rs, line 11.
Note: automatically using hardware breakpoints for read-only addresses.

(gdb) continue
Continuing.

Breakpoint 1, hello::__cortex_m_rt_main_trampoline () at examples/hello.rs:11
11      #[entry]
```

> **NOTA** Si GDB bloquea la terminal en lugar de alcanzar el punto de interrupción después 
> de emitir el comando `continue` anterior, es posible que desee verificar que la información de la región de 
> memoria en el archivo `memory.x` esté configurada correctamente 
> para su dispositivo (tanto los inicios *como* las longitudes). 

Ingrese a la función principal con `step`.

``` console
(gdb) step
halted: PC: 0x08000496
hello::__cortex_m_rt_main () at examples/hello.rs:13
13          hprintln!("Hello, world!").unwrap();
```

Después de avanzar el programa con `siguiente` deberías ver "¡Hola, mundo!" impreso en la consola OpenOCD, 
entre otras cosas.

``` console
$ openocd
(..)
Info : halted: PC: 0x08000502
Hello, world!
Info : halted: PC: 0x080004ac
Info : halted: PC: 0x080004ae
Info : halted: PC: 0x080004b0
Info : halted: PC: 0x080004b4
Info : halted: PC: 0x080004b8
Info : halted: PC: 0x080004bc
```
El mensaje solo se muestra una vez porque el programa está a punto de ingresar al bucle infinito definido en la línea 19: `loop {}`

Ahora puedes salir de GDB usando el comando `quit`.

``` console
(gdb) quit
A debugging session is active.

        Inferior 1 [Remote target] will be detached.

Quit anyway? (y or n)
```

La depuración ahora requiere algunos pasos adicionales, por lo que los hemos agrupado en un solo script de GDB llamado `openocd.gdb`. 
El archivo se creó durante el paso `cargo generate` y debería funcionar sin modificaciones. Echemos un vistazo:

``` console
cat openocd.gdb
```

``` text
target extended-remote :3333

# imprimir símbolos desenredados
set print asm-demangle on

# Detectar excepciones no controladas, fallos graves y pánicos
break DefaultHandler
break HardFault
break rust_begin_unwind

monitor arm semihosting enable

load

# Iniciar el proceso pero detener inmediatamente el procesador
stepi
```

Ahora, al ejecutar `<gdb> -x openocd.gdb target/thumbv7em-none-eabihf/debug/examples/hello` se conectará inmediatamente GDB a 
OpenOCD, habilitará el semihosting, cargará el programa e iniciará el proceso.

Como alternativa, puedes convertir `<gdb> -x openocd.gdb` en un ejecutor personalizado para que 
`cargo run` cree un programa *e* inicie una sesión de GDB. Este ejecutor está incluido en `.cargo/config.toml`, pero está comentado.

``` console
head -n10 .cargo/config.toml
```

``` toml
[target.thumbv7m-none-eabi]
# Descomente esto para hacer que `cargo run` ejecute programas en QEMU
# runner = "qemu-system-arm -cpu cortex-m3 -machine lm3s6965evb -nographic -semihosting-config enable=on,target=native -kernel"

[target.'cfg(all(target_arch = "arm", target_os = "none"))']
# Descomente UNA de estas tres opciones para hacer que `cargo run` inicie una sesión GDB
# La opción a elegir depende de su sistema.
runner = "arm-none-eabi-gdb -x openocd.gdb"
# runner = "gdb-multiarch -x openocd.gdb"
# runner = "gdb -x openocd.gdb"
```

``` text
$ cargo run --example hello
(..)
Loading section .vector_table, size 0x400 lma 0x8000000
Loading section .text, size 0x1e70 lma 0x8000400
Loading section .rodata, size 0x61c lma 0x8002270
Start address 0x800144e, load size 10380
Transfer rate: 17 KB/sec, 3460 bytes/write.
(gdb)
```
