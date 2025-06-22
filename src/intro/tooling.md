# Herramientas de desarrollo

Trabajar con microcontroladores implica usar varias herramientas diferentes, 
ya que trabajaremos con una arquitectura distinta a la de tu computadora portátil y 
tendremos que ejecutar y depurar programas en un dispositivo *remoto*.

Utilizaremos todas las herramientas que se enumeran a continuación. Cualquier versión reciente 
debería funcionar si no se especifica una versión mínima, pero hemos enumerado las versiones que hemos probado.

- Rust 1.31, 1.31-beta, o una cadena de herramientas más nueva más compatible con compilación ARM Cortex-M.
- [`cargo-binutils`](https://github.com/rust-embedded/cargo-binutils) ~0.1.4
- [`qemu-system-arm`](https://www.qemu.org/). Versiones probadas: 3.0.0
- OpenOCD >=0.8. Versiones probadas: v0.9.0 y v0.10.0
- GDB con soporte ARM. Version 7.12 o recomendablemente una mas nueva. Versiones
  probadas: 7.10, 7.11, 7.12 y 8.1
- [`cargo-generate`](https://github.com/ashleygwilliams/cargo-generate) o `git`.
  Estas herramientas son opcionales pero facilitarán el seguimiento del libro.

El texto a continuación explica por qué usamos estas herramientas. 
Las instrucciones de instalación se encuentran en la página siguiente.

## `cargo-generate` O `git`

Los programas de hardware directo son programas de Rust no estándar (`no_std`) que requieren ajustes en
el proceso de enlazado para que la distribución de memoria del programa sea correcta. Esto requiere archivos
adicionales (como scripts de enlazado) y configuraciones (como indicadores de enlazado). Los hemos empaquetado
en una plantilla para que solo tengas que completar la información faltante (como el nombre del proyecto y las
características del hardware de destino).

Nuestra plantilla es compatible con `cargo-generate`, un subcomando de Cargo para crear nuevos proyectos de Cargo
a partir de plantillas. También puedes descargar la plantilla usando `git`, `curl`, `wget` o tu navegador web.

## `cargo-binutils`

`cargo-binutils` es una colección de subcomandos de Cargo que facilitan el uso de las herramientas LLVM
incluidas en la cadena de herramientas de Rust. Estas herramientas incluyen las versiones LLVM de `objdump`,
`nm` y `size`, y se utilizan para inspeccionar binarios.

La ventaja de usar estas herramientas en lugar de GNU binutils es que. a: instalar las herramientas LLVM es la misma instalación con un solo comando (`rustup component add llvm-tools`) independientemente del sistema operativo y. b: herramientas como `objdump` admiten todas las arquitecturas compatibles con `rustc`, desde ARM hasta x86_64, porque ambas comparten el mismo backend LLVM.

## `qemu-system-arm`

QEMU es un emulador. En este caso, usamos la variante que puede emular completamente los sistemas ARM.
Usamos QEMU para ejecutar programas embebidos en el host. Gracias a esto, puedes seguir algunas partes
de este libro incluso sin hardware.

# Herramientas para la depuración de Rust embebido

## Descripción general

La depuración de sistemas embebidos en Rust requiere herramientas especializadas, como software para gestionar el proceso de depuración, depuradores para inspeccionar y controlar la ejecución del programa, y ​​sondas de hardware para facilitar la interacción entre el host y el dispositivo embebido. Este documento describe herramientas de software esenciales como Probe-rs y OpenOCD, que simplifican y facilitan el proceso de depuración, junto con depuradores destacados como GDB y la extensión Probe-rs para Visual Studio Code. Además, abarca sondas de hardware clave como Rusty-probe, ST-Link, J-Link y MCU-Link, esenciales para una depuración y 
programación eficaces de dispositivos embebidos.

## Software que maneja herramientas de depuración

### Probe-rs

Probe-rs es un software moderno, basado en Rust, diseñado para funcionar con depuradores en sistemas embebidos. A diferencia de OpenOCD, Probe-rs se diseñó pensando  en la simplicidad y busca reducir la carga de configuración que suelen presentar otras soluciones de depuración. Es compatible con diversas sondas y objetivos, lo que proporciona una interfaz de alto nivel para interactuar con hardware embebido. Probe-rs se integra directamente con las herramientas de Rust y con Visual Studio Code a través de su extensión, lo que permite a los desarrolladores optimizar su flujo de trabajo de depuración.


### OpenOCD (Open On-Chip Debugger)

OpenOCD es una herramienta de software de código abierto que se utiliza para depurar, probar y programar sistemas embebidos. Proporciona una interfaz entre el sistema host y el hardware embebido, compatible con diversas capas de transporte como JTAG y SWD (Serial Wire Debug). OpenOCD se integra con GDB, un depurador. OpenOCD cuenta con un amplio soporte, documentación extensa y una gran comunidad, pero puede requerir una configuración compleja, especialmente para configuraciones embebidas personalizadas.

## depuradores

Un depurador permite a los desarrolladores inspeccionar y controlar la ejecución de un programa para identificar y corregir errores. Ofrece funcionalidades como establecer puntos de interrupción, recorrer el código línea por línea y examinar los valores de las variables y los estados de memoria. Los depuradores son esenciales para un desarrollo y mantenimiento exhaustivos de software, ya que permiten a los desarrolladores garantizar que su código se comporte según lo previsto en diversas condiciones.

Los depuradores saben cómo:
 * Interactuar con los registros asignados a la memoria.
 * Establecer puntos de interrupción/observación.
 * Leer y escribir en los registros asignados a la memoria.
 * Detectar cuándo el MCU se detiene debido a un evento de depuración.
 * Continuar la ejecución del MCU tras un evento de depuración.
 * Borrar y escribir en la memoria FLASH del microcontrolador.

### extensión Probe-rs en Visual Studio Code 

Probe-rs cuenta con una extensión para Visual Studio Code que proporciona una experiencia de depuración fluida sin necesidad de una configuración compleja. Gracias a esta conexión, los desarrolladores pueden usar funciones específicas de Rust, como la impresión ordenada y los mensajes de error detallados, lo que garantiza que su proceso de depuración se integre con el ecosistema de Rust.

### GDB (GNU Debugger) 

GDB es una herramienta de depuración versátil que permite a los desarrolladores examinar el estado de los programas durante su ejecución o después de un fallo. En Rust embebido, GDB se conecta al sistema de destino mediante OpenOCD u otros servidores de depuración para interactuar con el código embebido. GDB es altamente configurable y admite funciones como depuración remota, inspección de variables y puntos de interrupción condicionales. Se puede utilizar en diversas plataformas y ofrece un amplio soporte para las necesidades de depuración específicas de Rust, como la impresión ordenada y la integración con IDE.


## Sondas

Una sonda de hardware es un dispositivo utilizado en el desarrollo y la depuración de sistemas embebidos para facilitar la comunicación entre un ordenador host y el dispositivo embebido de destino. Normalmente admite protocolos como JTAG o SWD, lo que le permite programar, depurar y analizar el microcontrolador o microprocesador del sistema embebido. Las sondas de hardware son cruciales para que los desarrolladores establezcan puntos de interrupción, revisen el código e inspeccionen la memoria y los registros del procesador, lo que les permite diagnosticar y solucionar problemas en tiempo real.

### Rusty-probe

Rusty-probe es una sonda de depuración de hardware basada en USB y de código abierto, diseñada para funcionar con probe-rs. La combinación de Rusty-Probe y probe-rs ofrece una solución fácil de usar y económica para desarrolladores que trabajan con aplicaciones Rust embebidas.

### ST-Link

ST-Link es una sonda de depuración y programación muy popular, desarrollada por STMicroelectronics principalmente para sus series de microcontroladores STM32 y STM8. Admite tanto depuración como programación mediante interfaces JTAG o SWD (depuración por cable serie). ST-Link es ampliamente utilizado gracias a su compatibilidad directa con la amplia gama de placas de desarrollo de STMicroelectronics y a su integración con los principales IDE, lo que la convierte en una opción ideal para desarrolladores que trabajan con microcontroladores STM.

### J-Link

J-Link, desarrollado por SEGGER Microcontroller, es un depurador robusto y versátil compatible con una amplia gama de núcleos de CPU y dispositivos, además de ARM, como RISC-V. Conocido por su alto rendimiento y fiabilidad, J-Link admite diversas interfaces de comunicación, como JTAG, SWD y JTAG de paso fino. Destaca por sus funciones avanzadas, como la cantidad ilimitada de puntos de interrupción en memoria flash y su compatibilidad con una gran variedad de entornos de desarrollo.

### MCU-Link

MCU-Link es una sonda de depuración que también funciona como programador, proporcionada por NXP Semiconductors. Es compatible con diversos microcontroladores ARM Cortex e interactúa a la perfección con herramientas de desarrollo como MCUXpresso IDE. MCU-Link destaca por su versatilidad y precio asequible, lo que la convierte en una opción accesible tanto para aficionados como para educadores y desarrolladores profesionales.
