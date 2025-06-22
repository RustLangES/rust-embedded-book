# Instalando las herramientas

Esta página contiene instrucciones de instalación independientes del sistema operativo para algunas de las herramientas:

### Cadena de herramientas de Rust 

Instale rustup siguiendo las instrucciones en [https://rustup.rs](https://rustup.rs).

**NOTA** Asegúrate de tener una versión del compilador igual o más reciente que `1.31`. `rustc -V` debería devolver una fecha más reciente que la que se muestra a   
continuación.

``` text
$ rustc -V
rustc 1.31.1 (b6c32da9b 2018-12-18)
```

Por cuestiones de ancho de banda y uso de disco, la instalación predeterminada solo admite la compilación nativa. Para añadir compatibilidad con compilación cruzada para las arquitecturas ARM Cortex-M, elija uno de los siguientes destinos de compilación. Para la placa STM32F3DISCOVERY utilizada en los ejemplos de este libro, utilice el destino `thumbv7em-none-eabihf`.
[Encuentra la mejor Cortex-M para ti.](https://developer.arm.com/ip-products/processors/cortex-m#c-7d3b69ce-5b17-4c9e-8f06-59b605713133) 

Cortex-M0, M0+, and M1 (Arquitectura ARMv6-M):
``` console
rustup target add thumbv6m-none-eabi
```

Cortex-M3 (Arquitectura ARMv7-M):
``` console
rustup target add thumbv7m-none-eabi
```

Cortex-M4 y M7 sin hardware de punto flotante (Arquitectura ARMv7E-M):
``` console
rustup target add thumbv7em-none-eabi
```

Cortex-M4F y M7F con hardware de punto flotante (Arquitectura ARMv7E-M):
``` console
rustup target add thumbv7em-none-eabihf
```

Cortex-M23 (Arquitectura ARMv8-M):
``` console
rustup target add thumbv8m.base-none-eabi
```

Cortex-M33 y M35P (Arquitectura ARMv8-M):
``` console
rustup target add thumbv8m.main-none-eabi
```

Cortex-M33F y M35PF con hardware de punto flotante (Arquitectura ARMv8-M):
``` console
rustup target add thumbv8m.main-none-eabihf
```


### `cargo-binutils`

``` text
cargo install cargo-binutils

rustup component add llvm-tools
```
WINDOWS: Es necesario tener instalado C++ Build Tools para Visual Studio 2019. https://visualstudio.microsoft.com/thank-you-downloading-visual-studio/?sku=BuildTools&rel=16 
### `cargo-generate`

Usaremos esto más adelante para generar un proyecto a partir de una plantilla.

``` console
cargo install cargo-generate
```

Nota: en algunas distribuciones de Linux (por ejemplo, Ubuntu) es posible que necesites instalar los paquetes `libssl-dev` y `pkg-config` antes de instalar cargo-generate.

### Instrucciones especificas del sistema operativo

Ahora siga las instrucciones específicas del sistema operativo que esté utilizando:

- [Linux](install/linux.md)
- [Windows](install/windows.md)
- [macOS](install/macos.md)
