# Un entorno Rust `no_std`


El término Programación embebida se utiliza para una amplia gama de diferentes clases de programación.
Desde la programación de MCU de 8 bits (como el [ST72325xx](https://www.st.com/resource/en/datasheet/st72325j6.pdf))
Con solo unos pocos KB de RAM y ROM, hasta sistemas como la Raspberry Pi
([Model B 3+](https://en.wikipedia.org/wiki/Raspberry_Pi#Specifications)) Con un procesador Cortex-A53 de 4 núcleos de 32/64 bits 
a 1,4 GHz y 1 GB de RAM. Se aplicarán diferentes restricciones/limitaciones al escribir código, según el tipo de objetivo y el caso de uso.
Existen dos clasificaciones generales de Programación embebida:
 
## Entornos alojados
Este tipo de entornos son cercanos a un entorno de PC normal.
Lo que esto significa es que se le proporciona una interfaz de sistema  [Por ejemplo POSIX](https://en.wikipedia.org/wiki/POSIX)
Esto te provee con primitivas para interactuar con diversos sistemas, como sistemas de archivos, redes, gestión de memoria, hilos, etc.
Las bibliotecas estándar, a su vez, suelen depender de estas primitivas para implementar su funcionalidad.
También puede tener algún tipo de raíz del sistema y restricciones en el uso de RAM/ROM, y quizás algún hardware
o Entrada/Salida especial. En general, se siente como programar en un entorno de PC con un proposito específico.

## Entornos de hardware directo 
En un entorno de hardware directo no se ha cargado ningún código antes de su programa.
Sin el software proporcionado por un sistema operativo, no podemos cargar la biblioteca estándar.
En cambio, el programa, junto con los crates que utiliza, solo puede usar el hardware (hardware directo) para ejecutarse.
Para evitar que Rust cargue la biblioteca estándar, use `no_std`.
Las partes independientes de la plataforma de la biblioteca estándar están disponibles a través del [libcore](https://doc.rust-lang.org/core/).
libcore También excluye cosas que no siempre son deseables en un entorno integrado.
Una de estas funciones es un asignador de memoria para la asignación dinámica de memoria. 
Si necesita esta u otras funciones, a menudo existen paquetes que las ofrecen.

### El runtime de libstd
Como se mencionó antes de usar [libstd](https://doc.rust-lang.org/std/) requiere algún tipo de integración del sistema, pero esto no se debe sólo a que
[libstd](https://doc.rust-lang.org/std/) No solo proporciona una forma común de acceder a las abstracciones del sistema operativo, sino que también proporciona un runtime.
Este runtime, entre otras cosas, se encarga de configurar la protección contra desbordamientos de pila, procesar los argumentos de la línea de comandos y generar el hilo principal antes de que se invoque la función principal de un programa. Este runtime tampoco estará disponible en un entorno `no_std`.

## Sumario
`#![no_std]` es un atributo de nivel de crate que indica que el crate se vinculara al core-crate en lugar de a el std-crate
El crate [libcore](https://doc.rust-lang.org/core/) A su vez, es un subconjunto independiente de la plataforma del crate std.
Lo cual no presupone nada sobre el sistema en el que se ejecutará el programa.
Por lo tanto, proporciona API para datos primitivos del lenguaje como floats, strings y slices, 
así como API que exponen funciones del procesador como operaciones atómicas e instrucciones SIMD.
Sin embargo, carece de API para cualquier aspecto que implique integración con plataformas.
Debido a estas propiedades, el código no\_std y  [libcore](https://doc.rust-lang.org/core/) se puede usar para cualquier tipo de código de arranque (etapa 0),
como cargadores de arranque, firmware o kernels.

### Descripción general

| caracteristica                                            | no\_std | std |
|-----------------------------------------------------------|--------|-----|
| heap/monticulo (memoria dinámica)                                   |   *    |  ✓  |
| colecciones (Vec, BTreeMap, etc)                          |  **    |  ✓  |
| protección contra desbordamiento de pila                  |   ✘    |  ✓  |
| ejecuta código de inicio antes de main                    |   ✘    |  ✓  |
| libstd disponible                                         |   ✘    |  ✓  |
| libcore disponible                                        |   ✓    |  ✓  |
| escribir codigo de firmware, kernel, o cargadores de arranque      |   ✓    |  ✘  |

\* Solo si se usa el crate `alloc` y un asignador adecuado como [alloc-cortex-m].

\** Solo si se usa el crate `collections` y se configura un asignador global predeterminado.

\**HashMap y HashSet no están disponibles debido a la falta de un generador de números aleatorios seguro.

[alloc-cortex-m]: https://github.com/rust-embedded/alloc-cortex-m

## Mira también
* [RFC-1184](https://github.com/rust-lang/rfcs/blob/master/text/1184-stabilize-no_std.md)

