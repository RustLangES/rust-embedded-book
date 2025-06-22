# Windows

## `arm-none-eabi-gdb`

ARM proporciona instaladores `.exe` para Windows. Descargue uno desde [aquí][gcc] y siga las instrucciones.
Justo antes de que finalice el proceso de instalación, marque la opción "Agregar ruta a la variable de entorno". Luego, verifique que las herramientas estén en su `%PATH%`:

``` text
$ arm-none-eabi-gdb -v
GNU gdb (GNU Tools for Arm Embedded Processors 7-2018-q2-update) 8.1.0.20180315-git
(..)
```

[gcc]: https://developer.arm.com/downloads/-/arm-gnu-toolchain-downloads

## OpenOCD

No existe una versión binaria oficial de OpenOCD para Windows, pero si no te animas a compilarlo tú mismo, el proyecto xPack ofrece una distribución binaria, [aquí][openocd]. Sigue las instrucciones de instalación proporcionadas. Luego, actualiza la variable de entorno `%PATH%` para incluir la ruta donde se instalaron los binarios. (`C:\Users\USERNAME\AppData\Roaming\xPacks\@xpack-dev-tools\openocd\0.10.0-13.1\.content\bin\`, si has estado usando la instalación sencilla). 

[openocd]: https://xpack.github.io/openocd/

Verifique que OpenOCD esta en tu `%PATH%` con:

``` text
$ openocd -v
Open On-Chip Debugger 0.10.0
(..)
```

## QEMU

Tome QEMU desde [El sitio oficial][qemu].

[qemu]: https://www.qemu.org/download/#windows

## ST-LINK USB driver

También deberá instalar [este controlador USB]; de lo contrario, OpenOCD no funcionará. Siga las instrucciones del instalador y asegúrese de instalar la versión correcta (32 o 64 bits) del controlador.

[este controlador USB]: http://www.st.com/en/embedded-software/stsw-link009.html

Eso es todo! ve a la [siguiente sección].

[siguiente sección]: verify.md
