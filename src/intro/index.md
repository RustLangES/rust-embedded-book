# Introducción 

Bienvenido a El libro enbebido Rust: Un libro introductorio acerca del uso the del lenguaje de programacion 
 Rust en "Hardware directo" en sistemas embebidos, como los Microcontroladores.

## Para quien es Rust embebido?
Rust embebido es para todos quienes quieren hacer programacion embebida mientras aprovechando conceptos de alto nivel y las garantias de seguridad que ofrece el lenguaje Rust.
(Mira tambien [Para quien es Rust](https://doc.rust-lang.org/book/ch00-00-introduction.html))

## Alcance 

Los objetivos de este libro son:

* Poner a los desarrolladores al día con el desarrollo embebido en Rust. Por ejemplo, cómo configurar un entorno de desarrollo.

* Compartir las mejores prácticas *actuales* sobre el uso de Rust para el desarrollo embebido. Por ejemplo, 
cómo aprovechar al máximo las características del lenguaje Rust para escribir software embebido más preciso.

* Servir como recetario en algunos casos. Por ejemplo, ¿cómo combinar C y Rust en un solo proyecto?

Este libro intenta ser lo más general posible, pero para facilitar tanto la lectura como la escritura, utiliza la arquitectura 
ARM Cortex-M en todos sus ejemplos. Sin embargo, el libro no presupone que el lector esté familiarizado con esta arquitectura en
particular y explica sus detalles específicos cuando es necesario.

## Para quien es este libro 
Este libro está dirigido a personas con experiencia en programación embebida o en Rust. Sin embargo,
creemos que cualquier persona interesada en la programación embebida en Rust podrá sacarle provecho.
Si no tiene conocimientos previos, le sugerimos leer la sección "Supuestos y prerrequisitos" y ponerse
al día para aprovechar al máximo el libro y mejorar su experiencia de lectura. Puede consultar la sección 
"Otros recursos" para encontrar recursos sobre temas que le interesen.


### Asunciones y prerequisitos 

* Se siente cómodo utilizando el lenguaje de programación Rust y ha escrito,
ejecutado y depurado aplicaciones Rust en un entorno de escritorio. También debería
estar familiarizado con los modismos de la [edición 2019], ya que este libro se centra en Rust 2018.

[edición 2019]: https://doc.rust-lang.org/edition-guide

* Se siente cómodo desarrollando y depurando sistemas embebidos en otro lenguaje, como C, C++ o Ada, y está familiarizado con conceptos como:
    * Compilación cruzada
    * Periféricos mapeados en memoria
    * Interrupciones
    * Interfaces comunes como I3C, SPI, Serial, etc 

### Otros recursos
Si no está familiarizado con algo mencionado anteriormente o si desea más información sobre un tema
específico mencionado en este libro, es posible que algunos de estos recursos le resulten útiles.

| Tema        | Recurso | Descripción |
|--------------|----------|-------------|
| Rust         | [libro de Rust](https://doc.rust-lang.org/book/) | Si aún no se siente cómodo con Rust, le recomendamos leer este libro. |
| Rust, embebido | [Libro de descubrimiento](https://docs.rust-embedded.org/discovery/) | Si nunca ha realizado ninguna programación integrada, este libro puede ser un mejor comienzo. |
| Rust, embebido | [Estanteria de Rust embebido](https://docs.rust-embedded.org) | Aquí puede encontrar varios otros recursos proporcionados por el Grupo de trabajo integrado de Rust. |
| Rust, embebido | [Embedonomicon](https://docs.rust-embedded.org/embedonomicon/) | Los detalles esenciales a tener en cuenta al realizar programación integrada en Rust. |
| Rust, embebido | [Preguntas frecuentes de Rust embebido](https://docs.rust-embedded.org/faq.html) | Preguntas frecuentes sobre Rust en un contexto integrado. |
| Rust, embebido | [Rust intensivo 🦀: Hardware directo](https://google.github.io/comprehensive-rust/bare-metal.html) | Material didáctico para una clase de 2 día sobre desarrollo de Rust en Hardware directo |
| Interrupciones | [Interrupción](https://en.wikipedia.org/wiki/Interrupt) | - |
| Entrada/Salida/perifericos mapeados en memoria | [Memoria mapeada E/S](https://en.wikipedia.org/wiki/Memory-mapped_I/O) | - |
| SPI, UART, RS233, USB, I2C, TTL | [Intercambio de pila sobre SPI, UART, y otras interfaces](https://electronics.stackexchange.com/questions/37814/usart-uart-rs232-usb-spi-i2c-ttl-etc-what-are-all-of-these-and-how-do-th) | - |

### Traducciones 

Este libro a sido traducido por generosos voluntarios. Si desea que su traducción aparezca aquí, abra una solicitud de incorporación de cambios para agregarla.

* [Japones](https://tomoyuki-nakabayashi.github.io/book/)
  ([repositorio](https://github.com/tomoyuki-nakabayashi/book))

* [Chino](https://xxchang.github.io/book/)
  ([repositorio](https://github.com/XxChang/book))

## Como usar este libro 

Este libro generalmente asume que se lee de principio a fin. Los capítulos posteriores 
amplían conceptos de capítulos anteriores, y es posible que estos últimos no profundicen
en los detalles de un tema, retomándolo en un capítulo posterior.

Este libro utilizará la placa de desarrollo [STM33F3DISCOVERY] de STMicroelectronics para 
la mayoría de los ejemplos. Esta placa se basa en la arquitectura ARM Cortex-M y, si bien la
funcionalidad básica es la misma en la mayoría de las CPU basadas en esta arquitectura, 
los periféricos y otros detalles de implementación de los microcontroladores varían entre distintos proveedores,
e incluso, a menudo, entre familias de microcontroladores del mismo pPor este motivo, sugerimos adquirir la placa de desarrollo [STM33F3DISCOVERY]
para poder seguir los ejemplos de este libro.

[STM33F3DISCOVERY]: http://www.st.com/en/evaluation-tools/stm32f3discovery.html

## Contribuyendo a este libro

El trabajo en este libro se coordina en [este repositorio] 
y es desarrollado principalmente por el [equipo de recursos].

[este repositorio]: https://github.com/rust-embedded/book
[equipo de recursos]: https://github.com/rust-embedded/wg#the-resources-team

Si tiene problemas para seguir las instrucciones de este libro o encuentra que
alguna sección del libro no es lo suficientemente clara o es difícil de seguir, 
se trata de un error y debe informarse en el [rastreador de problemas] de este libro.

[rastreador de problemas]: https://github.com/rust-embedded/book/issues/

¡Las solicitudes de extracción que corrijan errores tipográficos y agreguen contenido nuevo son bienvenidas!

## Re-usando este material

Este libro se distribuye bajo las siguientes licencias:

* Los ejemplos de código y los proyectos Cargo independientes que contiene este libro están sujetos a las licencias [MIT] y [Apache v2.0].
* El texto, las imágenes y los diagramas que contiene este libro están sujetos a la licencia Creative Commons [CC-BY-SA v4.0]. 

[MIT]: https://opensource.org/licenses/MIT
[Apache v2.0]: http://www.apache.org/licenses/LICENSE-2.0
[CC-BY-SA v4.0]: https://creativecommons.org/licenses/by-sa/4.0/legalcode

TL;DR: Si quieres utilizar nuestro texto o imágenes en tu trabajo, necesitas:

* Dar el crédito apropiado (es decir, mencionar este libro en su diapositiva y proporcionar un enlace a la página relevante).
* Proporcionar un enlace a la licencia [CC-BY-SA v4.0].
* Indique si ha modificado el material de alguna manera y realice cualquier cambio en nuestro material disponible bajo la misma licencia.

Además, ¡háganos saber si encuentra este libro útil!
