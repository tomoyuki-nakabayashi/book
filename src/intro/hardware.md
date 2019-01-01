<!-- # Meet Your Hardware -->

# ハードウェアとの出会い

<!-- Let's get familiar with the hardware we'll be working with. -->

これから作業するハードウェアに詳しくなりましょう。

<!-- ## STM32F3DISCOVERY (the "F3") -->

## STM32F3DISCOVERY ("F3")

<p align="center">
<img title="F3" src="../assets/f3.jpg">
</p>

<!-- We'll refer to this board as "F3" throughout this book. -->

私たちは、本書内でこのボードを"F3"と呼びます。

<!-- What does this board contain? -->

このボードには何が搭載されていますか？

<!-- 
- A STM32F303VCT6 microcontroller. This microcontroller has
  - A single-core ARM Cortex-M4F processor with hardware support for single-precision floating point
    operations and a maximum clock frequency of 72 MHz.

  - 256 KiB of "Flash" memory. (1 KiB = 10**24** bytes)

  - 48 KiB of RAM.

  - many "peripherals": timers, GPIO, I2C, SPI, USART, etc.

  - lots of "pins" that are exposed in the two lateral "headers".

  - **IMPORTANT** This microcontroller operates at (around) 3.3V.
 -->
- STM32F303VCT6マイクロコントローラが1つ。このマイクロコントローラは、次のものを搭載しています。
  - 単精度浮動小数点演算をハードウェアサポートし、最大72MHzのクロック周波数で動作するシングルコアのARM Cortex-M4Fプロセッサ

  - 256 KiBの"フラッシュ"メモリ (1 KiB = 10**24** bytes)

  - 48 KiBのRAM

  - 多くの"周辺機器": タイマ、GPIO、I2C、SPI、USART、他

  - 両側面の"ヘッダ"に配置された多数の"ピン"

  - **重要** このマイクロコントローラは、約3.3ボルトで動作します。

<!-- 
- An [accelerometer] and a [magnetometer][] (in a single package).

[accelerometer]: https://en.wikipedia.org/wiki/Accelerometer
[magnetometer]: https://en.wikipedia.org/wiki/Magnetometer
 -->

- [加速度センサ]と[磁気センサ]が1つずつ (1つのパッケージにまとめられています)

[加速度センサ]: https://en.wikipedia.org/wiki/Accelerometer
[磁気センサ]: https://en.wikipedia.org/wiki/Magnetometer

<!-- 
- A [gyroscope].

[gyroscope]: https://en.wikipedia.org/wiki/Gyroscope
 -->

- [ジャイロセンサ]が1つ

[ジャイロセンサ]: https://en.wikipedia.org/wiki/Gyroscope

<!-- - 8 user LEDs arranged in the shape of a compass -->

- 円形に配置された8個のユーザLED

<!-- 
- A second microcontroller: a STM32F103CBT. This microcontroller is actually part of an on-board
  programmer and debugger named ST-LINK and is connected to the USB port named "USB ST-LINK".
 -->

- 第2のマイクロコントローラ: STM32F103CBT。このマイクロコントローラは、実際には、ST-LINKというオンボードプログラマおよびデバッガの一部であり、"USB ST-LINK"という名前のUSBポートに接続されています。

<!-- 
- There's a second USB port, labeled "USB USER" that is connected to the main microcontroller, the
  STM32F303VCT6, and can be used in applications.
 -->

- "USB USER"というラベルが付いている第2のUSBポート。このUSBポートは、メインマイクロコントローラ (STM32F303VCT6)に接続されており、アプリケーションで利用できます。