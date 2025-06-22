# macOS

Todas las herramientas pueden ser instaladas usando [Homebrew] o [MacPorts]:

[Homebrew]: http://brew.sh/
[MacPorts]: https://www.macports.org/

## Instalar herramientas con [Homebrew]

``` text
$ # GDB
$ brew install arm-none-eabi-gdb

$ # OpenOCD
$ brew install openocd

$ # QEMU
$ brew install qemu
```

> **NOTA** Si OpenOCD falla, es posible que necesites instalar la última versión usando:
```text
$ brew install --HEAD openocd
```

## Instalar herramientas usando [MacPorts]

``` text
$ # GDB
$ sudo port install arm-none-eabi-gcc

$ # OpenOCD
$ sudo port install openocd

$ # QEMU
$ sudo port install qemu
```



Todo esta instalado!, ve a la [siguiente sección].

[siguiente sección]: verify.md
