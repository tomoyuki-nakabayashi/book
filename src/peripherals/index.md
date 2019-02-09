<!-- # Peripherals -->

# ペリフェラル

<!-- ## What are Peripherals? -->

## ペリフェラルとは何か？

<!--
Most Microcontrollers have more than just a CPU, RAM, or Flash Memory - they contain sections of silicon which are used for interacting with systems outside of the microcontroller, as well as directly and indirectly interacting with their surroundings in the world via sensors, motor controllers, or human interfaces such as a display or keyboard. These components are collectively known as Peripherals.
-->

ほとんどのマイクロコントローラはCPUやRAM、フラッシュメモリ以外のものを持っています。マイクロコントローラはシリコン上に専用のセクションを持っており、そのセクションは、マイクロコントローラの外のシステムとやり取りしたり、センサーやモーターコントローラ、またはディスプレイもしくはキーボードのようなヒューマンインタフェースによって世界中の周囲環境と直接的または間接的にやり取りするために使用されます。それらのコンポーネントはまとめてペリフェラルと呼ばれています。

<!--
These peripherals are useful because they allow a developer to offload processing to them, avoiding having to handle everything in software. Similar to how a desktop developer would offload graphics processing to a video card, embedded developers can offload some tasks to peripherals allowing the CPU to spend it's time doing something else important, or doing nothing in order to save power.
-->

これらのペリフェラルは有用です。なぜならば、開発者はペリフェラルに処理をオフロードすることが可能になるため、全ての処理をソフトウェアで行う必要がなくなります。デスクトップ開発者がグラフィック処理をビデオカードにオフロードするのと同じように、組込み開発者は一部のタスクをペリフェラルにオフロードして、CPUの時間を他の重要なことに使ったり、電力を節約するために何もしないようにすることができます。

<!--
If you look at the main circuit board in an old-fashioned home computer from the 1970s or 1980s (and actually, the desktop PCs of yesterday are not so far removed from the embedded systems of today) you would expect to see:
-->

1970年代か1980年代の旧式の家庭用コンピュータのメイン回路基板を見れば（実際に、旧式のデスクトップPCは今日の組込みシステムとさほど違いません）、次のものを目にするはずです。

<!--
* A processor
* A RAM chip
* A ROM chip
* An I/O controller
-->

* プロセッサ
* RAMチップ
* ROMチップ
* I/Oコントローラ

<!--
The RAM chip, ROM chip and I/O controller (the peripheral in this system) would be joined to the processor through a series of parallel traces known as a 'bus'. This bus carries address information, which selects which device on the bus the processor wishes to communicate with, and a data bus which carries the actual data. In our embedded microcontrollers, the same principles apply - it's just that everything is packed on to a single piece of silicon.
-->

RAMチップ、ROMチップ、I/Oコントローラ（このシステムのペリフェラル）は「バス」として知られる一連の並列な配線を通してプロセッサに接続されているでしょう。アドレスバスは、プロセッサがバス上のどのデバイスと通信したいかを選択するアドレス情報を運び、データバスは、実際のデータを運びます。組込みマイクロコントローラにおいても、同じ原理が適用されます。それは全てが１つのシリコン片に詰め込まれているということです。

<!--
However, unlike graphics cards, which typically have a Software API like Vulkan, Metal, or OpenGL, peripherals are exposed to our Microcontroller with a hardware interface, which is mapped to a chunk of the memory.
-->

しかしながら、VulkanやMetal、OpenGLなどのソフトウェアのAPIを通常持つグラフィックカードとは異なり、ペリフェラルはメモリチャンクにマッピングされたハードウェアインターフェースとしてマイクロコントローラに公開されています。

<!-- ## Linear and Real Memory Space -->

## 線形な実メモリ空間

<!--
On a microcontroller, writing some data to some other arbitrary address, such as `0x4000_0000` or `0x0000_0000`, may also be a completely valid action.
-->

マイクロコントローラでは、`0x4000_0000`や`0x0000_0000`のような任意のアドレスにデータを書き込むことは、完全に正当な行為でしょう。

<!--
On a desktop system, access to memory is tightly controlled by the MMU, or Memory Management Unit. This component has two major responsibilities: enforcing access permission to sections of memory (preventing one process from reading or modifying the memory of another process); and re-mapping segments of the physical memory to virtual memory ranges used in software. Microcontrollers do not typically have an MMU, and instead only use real physical addresses in software.
-->

デスクトップシステムでは、メモリアクセスはMMU（メモリ管理ユニット）によって厳密に制御されています。このコンポーネントは２つの主な役割を持っています。メモリのセクションへのアクセス権限の強制（あるプロセスが別のプロセスのメモリを読み出したり変更したりできないようにする）、そして物理メモリのセグメントをソフトウェアで使用される仮想メモリ範囲に再マッピングすることです。マイクロコントローラは通常はMMUを持たず、代わりにソフトウェアで物理アドレスのみを使用します。

<!--
Although 32 bit microcontrollers have a real and linear address space from `0x0000_0000`, and `0xFFFF_FFFF`, they generally only use a few hundred kilobytes of that range for actual memory. This leaves a significant amount of address space remaining. In earlier chapters, we were talking about RAM being located at address `0x2000_0000`. If our RAM was 64 KiB long (i.e. with a maximum address of 0xFFFF) then addresses `0x2000_0000` to `0x2000_FFFF` would correspond to our RAM. When we write to a variable which lives at address `0x2000_1234`, what happens internally is that some logic detects the upper portion of the address (0x2000 in this example) and then activates the RAM so that it can act upon the lower portion of the address (0x1234 in this case). On a Cortex-M we also have our Flash ROM mapped in at address `0x0000_0000` up to, say, address `0x0007_FFFF` (if we have a 512 KiB Flash ROM). Rather than ignore all remaining space between these two regions, Microcontroller designers instead mapped the interface for peripherals in certain memory locations. This ends up looking something like this:
-->

32ビットマイクロコントローラは`0x0000_0000`から`0xFFFF_FFFF`の線形な実アドレス空間を持ちますが、それらは大抵の場合、実際のメモリのためにはその範囲の数百キロバイトしか使用しません。これにより、かなりの量のアドレス空間が残ります。前の章では、RAMが`0x2000_0000`のアドレスに配置されていることについて話しました。もしもRAMが64KiBの長さなら（すなわち、最大アドレスが0xFFFF）、`0x2000_0000`から`0x2000_FFFF`がRAMのアドレスに対応します。`0x2000_1234`のアドレスにある変数に書き込むと、内部ではアドレスの上位部分（この例では0x2000）を検出し、アドレスの下位部分（この例では0x1234）に作用できるようにRAMをアクティブにします。Cortex-Mにおいては、フラッシュROMも`0x0000_0000`から`0x0007_FFFF`のアドレスにマッピングされています（512KiBのフラッシュROMが載っている場合）。これら２つの領域の間に残るスペースを全て無視するのではなく、代わりにマイクロコントローラの設計者は特定のメモリ配置にペリフェラルのインターフェースをマッピングしました。これは次のようなものになります。

![](../assets/nrf52-memory-map.png)

[Nordic nRF52832 Datasheet (pdf)]

<!-- ## Memory Mapped Peripherals -->

## メモリマップド・ペリフェラル

<!--
Interaction with these peripherals is simple at a first glance - write the right data to the correct address. For example, sending a 32 bit word over a serial port could be as direct as writing that 32 bit word to a certain memory address. The Serial Port Peripheral would then take over and send out the data automatically.
-->

一見すると、これらのペリフェラルとのやり取りは簡単です。正しいデータを正しいアドレスに書き込むだけです。例えば、32ビットワードをシリアルポート上で送信することは、32ビットワードを特定のメモリアドレスに書き込むことと同じくらい直接的になり得ます。シリアルポート・ペリフェラルは自動的にデータを引き受けて送信します。

<!--
Configuration of these peripherals works similarly. Instead of calling a function to configure a peripheral, a chunk of memory is exposed which serves as the hardware API. Write `0x8000_0000` to a SPI Frequency Configuration Register, and the SPI port will send data at 8 Megabits per second. Write `0x0200_0000` to the same address, and the SPI port will send data at 125 Kilobits per second. These configuration registers look a little bit like this:
-->

これらのペリフェラルの設定についても同じように動作します。ペリフェラルの設定をするための関数を呼ぶ代わりに、ハードウェアAPIとして機能するメモリチャンクが公開されます。`0x8000_0000`をSPI周波数の設定レジスタに書き込むと、SPIポートは8Mbpsでデータを送信するようになります。`0x0200_0000`を同じアドレスに書き込むと、SPIは125Kbpsでデータを送信するようになります。これらの設定レジスタをちょっとだけ見てみます。

![](../assets/nrf52-spi-frequency-register.png)

[Nordic nRF52832 Datasheet (pdf)]

<!--
This interface is how interactions with the hardware are made, no matter what language is used, whether that language is Assembly, C, or Rust.
-->

アセンブリ言語やC言語、Rustなど、どの言語が使われようとも、このインターフェースがどのように作用するかはハードウェアによって定められています。

[Nordic nRF52832 Datasheet (pdf)]: http://infocenter.nordicsemi.com/pdf/nRF52832_PS_v1.1.pdf
