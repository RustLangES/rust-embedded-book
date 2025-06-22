# Conozca su hardware

Familiaricémonos con el hardware con el que trabajaremos.

## STM32F3DISCOVERY (La "F3")

<p align="center">
<img title="F3" src="../assets/f3.jpg">
</p>

¿Qué contiene esta placa?

- Un microcontrolador [STM32F303VCT6](https://www.st.com/en/microcontrollers/stm32f303vc.html). Este microcontrolador tiene
  - Un procesador ARM Cortex-M4F de un solo núcleo con soporte de hardware para operaciones de punto flotante 
  de precisión simple y una frecuencia de reloj máxima de 72 MHz.

  - 256 KiB de memoria "Flash". (1 KiB = 10**24** bytes)

  - 48 KiB de RAM.

  - Una variedad de periféricos integrados como temporizadores, I2C, SPI y USART.

  - Entrada y salida de propósito general (GPIO) y otros tipos de pines accesibles a través de las dos filas de encabezados a lo largo de la placa.
  
  - Una interfaz USB accesible a través del puerto USB denominado "USB USER".

- Un [acelerometro](https://en.wikipedia.org/wiki/Accelerometer) como parte del chip [LSM303DLHC](https://www.st.com/en/mems-and-sensors/lsm303dlhc.html).

- Un [magnetometro](https://en.wikipedia.org/wiki/Magnetometer) como parte del chip   [LSM303DLHC](https://www.st.com/en/mems-and-sensors/lsm303dlhc.html).

- A [giroscopio](https://en.wikipedia.org/wiki/Gyroscope) como parte del chip [L3GD20](https://www.pololu.com/file/0J563/L3GD20.pdf).

- Un conjunto de 8 LEDs dispuestos en forma de brujula.

- Un segundo microcontrolador: Un [STM32F103](https://www.st.com/en/microcontrollers/stm32f103cb.html). Este microcontrolador es actualmente parte de un programador/depurador integrado y es conectado al puerto USB llamado "USB ST-LINK".

Para obtener una lista más detallada de las características y especificaciones adicionales de la placa, consulte la pagina web [STMicroelectronics](https://www.st.com/en/evaluation-tools/stm32f3discovery.html).

Precaución: tenga cuidado si desea aplicar señales externas a la placa. Los pines del microcontrolador STM32F303VCT6 admiten un voltaje nominal de 3,3 voltios. Para más información, consulte [6.2 Sección de ratings maximos absolutos en el manual](https://www.st.com/resource/en/datasheet/stm32f303vc.pdf)
