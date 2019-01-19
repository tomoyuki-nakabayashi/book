<!-- # Portability -->

# 移植性

<!--
In embedded environments portability is a very important topic: Every vendor and even each family from a single manufacturer offers different peripherals and capabilities and similarly the ways to interact with the peripherals will vary.
-->

組込み環境においては、移植性は非常に重要なトピックです。1つのメーカから、それぞれのベンダや各プロダクトファミリが、異なるペリフェラルや機能を提供します、
異なるペリフェラルでは、ペリフェラルとやり取りする方法も異なります。

<!-- A common way to equalize such differences is via a layer called Hardware Abstraction layer or **HAL**. -->

このような違いを吸収する一般的な方法は、ハードウェア抽象化レイヤまたは**HAL**と呼ばれるレイヤを導入することです。

<!--
> Hardware abstractions are sets of routines in software that emulate some platform-specific details, giving programs direct access to the hardware resources.
>
> They often allow programmers to write device-independent, high performance applications by providing standard operating system (OS) calls to hardware.
>
> *Wikipedia: [Hardware Abstraction Layer]*
-->

> ハードウェア抽象化は、プラットフォーム固有の細部をエミュレーションして、プログラムにハードウェアリソースへ直接アクセスする方法を提供する、ソフトウェアの一連のルーチンです。
> 
> それらのルーチンがハードウェアへのオペレーティングシステム（OS）コールを提供することで、プログラマは、デイバスに依存せず、高性能なアプリケーションを書くことができます。
> 
> *Wikipedia: [ハードウェア抽象化レイヤ]*


<!-- [Hardware Abstraction Layer]: https://en.wikipedia.org/wiki/Hardware_abstraction -->

[ハードウェア抽象化レイヤ]: https://en.wikipedia.org/wiki/Hardware_abstraction

<!--
Embedded systems are a bit special in this regard since we typically do not have operating systems and user installable software but firmware images which are compiled as a whole as well as a number of other constraints. So while the traditional approach as defined by Wikipedia could potentially work it is likely not the most productive approach to ensure portability.
-->

組込みシステムは、通常オペレーティングシステムを使わず、ユーザがインストールできるソフトウェアもありません。
全体として1つにコンパイルされたファームウェアイメージであり、様々な制約を持つ、という点が特殊です。
そのため、Wikipediaで定義されている従来のアプローチでもうまく機能する可能性はありますが、移植性を確保するための最も有効なアプローチではない可能性があります。

<!-- How do we do this in Rust? Enter **embedded-hal**... -->

Rustではどうするのでしょうか？**embedded-hal**を見ていきましょう。

<!-- ## What is embedded-hal? -->

## embedded-halとは？

<!--
In a nutshell it is a set of traits which define implementation contracts between **HAL implementations**, **drivers** and **applications (or firmwares)**. Those contracts include both capabilities (i.e. if a trait is implemented for a certain type, the **HAL implementation** provides a certain capability) and methods (i.e. if you can construct a type implementing a trait it is guaranteed that you have the methods specified in the trait available).
-->

一言で言えば、**HAL実装**、**ドライバ**、**アプリケーションまたはファームウェア**間の実装規約を定義するトレイトの集まりのことです。
それらの規約は、機能（ある型にトレイトが実装されていれば、**HAL実装**はそのトレイトの機能を提供します）とメソッド（あるトレイトを実装している型のオブジェクトを作成できれば、そのトレイトに含まれるメソッドが利用可能であることが保証されます）を含んでいます。

<!-- A typical layering might look like this: -->

典型的なレイヤ構造は、次のようになります。

![](../assets/rust_layers.svg)

<!--
Some of the defined traits in **embedded-hal** are:
* GPIO (input and output pins)
* Serial communication
* I2C
* SPI
* Timers/Countdowns
* Analog Digital Conversion
-->

**embedded-hal**で定義されているいくつかのトレイトを示します。
* GPIO（入出力ピン）
* シリアル通信
* I2C
* SPI
* タイマ/カウントダウン
* アナログデジタル変換

<!--
The main reason for having the **embedded-hal** traits and crates implementing and using them is to keep complexity in check. If you consider that an application might have to implement the use of the peripheral in the hardware as well as the application and potentially drivers for additional hardware components, then it should be easy to see that the re-usability is very limited. Expressed mathematically, if **M** is the number of peripheral HAL implementations and **N** the number of drivers then if we were to reinvent the wheel for every application then we would end up with **M*N** implementations while by using the *API* provided by the **embedded-hal** traits will make the implementation complexity approach **M+N**. Of course there're additional benefits to be had, such as less trial-and-error due to a well-defined and ready-to-use APIs.
-->

**embedded-hal**トレイトと、それらを実装して使用するクレートを持つ主な理由は、複雑さを抑えることです。
アプリケーションがハードウェアのペリフェラルを使うコードだけでなく、追加するハードウェアコンポーネントのドライバを実装しなければならない場合を考えると、再利用性は非常に制限されます。
数学的に表現すると、**M**がペリフェラルHAL実装の数で、**N**がドライバの数とするならば、全てのアプリケーションで車輪の再発明を行うことになります。
その結果、**M\*N**の実装が必要になります。
一方、**embedded-hal**トレイトで提供されるAPIを使うことで、実装の複雑さは**M+N**になるでしょう。
もちろん、明確に定義されたすぐに利用可能なAPIによって試行錯誤を減らすなど、他にも利点があります。

<!-- ## Users of the embedded-hal -->

## embedded-halのユーザ

<!-- As said above there are three main users of the HAL: -->

上記のように、HALには主に3つのユーザがいます。

<!-- ### HAL implementation -->

### HAL実装

<!--
A HAL implementation provides the interfacing between the hardware and the users of the HAL traits. Typical implementations consist of three parts:
* One or more hardware specific types
* Functions to create and initialize such a type, often providing various configuration options (speed, operation mode, use pins, etc.)
* one or more `trait` `impl` of **embedded-hal** traits for that type
-->

HAL実装は、ハードウェアとHALトレイトのユーザとの間のインタフェースを実装します。典型的な実装では、3つの部分から構成されます。
* 1つ以上のハードウェア固有の型
* そのような型のオブジェクトを作成し、初期化する関数。多くの場合、様々な設定オプションを提供しています（速度、動作モード、使用ピンなど）
* 1つ以上のその型に対する**embedded-hal**の`trait` `impl`

<!--
Such a **HAL implementation** can come in various flavours:
* Via low-level hardware access, e.g. via registers
* Via operating system, e.g. by using the `sysfs` under Linux
* Via adapter, e.g. a mock of types for unit testing
* Via driver for hardware adapters, e.g. I2C multiplexer or GPIO expander
-->

このような**HAL実装**は、様々な種類があります。
* 低レベルのハードウェアアクセスによるもの、例えばレジスタ
* オペレーティングシステムによるもの、例えばLinuxの`sysfs`の使用
* アダプタによるもの、例えばユニットテストのための型のモック
* ハードウェアアダプタ用のドライバによるもの、例えばI2CマルチプレクサやGPIOエキスパンダ

<!-- ### Driver -->

### ドライバ

<!--
A driver implements a set of custom functionality for an internal or external component, connected to a peripheral implementing the embedded-hal traits. Typical examples for such drivers include various sensors (temperature, magnetometer, accelerometer, light), display devices (LED arrays, LCD displays) and actuators (motors, transmitters).
-->

ドライバは、embedded-halトレイトを実装しているペリフェラルに接続されている、内部または外部コンポーネントに対するカスタム機能を実装します。
そのようなドライバの典型的な例は、様々なセンサ（温度、磁気、加速度、輝度）、ディスプレイデバイス（LEDアレイ、LCDディスプレイ）や、アクチュエータ（モータ、トランスミッタ）を含みます。

<!--
A driver has to be initialized with an instance of type that implements a certain `trait` of the embedded-hal which is ensured via trait bound and provides its own type instance with a custom set of methods allowing to interact with the driven device.
-->

ドライバは、embedded-halの特定のトレイトを実装する型のインスタンスと共に初期化する必要があります。
そのトレイトは、トレイト境界により保証され、駆動するデバイスとやり取りできるインスタンスをカスタムメソッドと共に提供します。

<!-- ### Application -->

### アプリケーション

<!--
The application binds the various parts together and ensures that the desired functionality is achieved. When porting between different systems, this is the part which requires the most adaptation efforts, since the application needs to correctly initialize the real hardware via the HAL implementation and the initialisation of different hardware differs, sometimes drastically so. Also the user choice often plays a big role, since components can be physically connected to different terminals, hardware buses sometimes need external hardware to match the configuration or there are different trade-offs to be made in the use of internal peripherals (e.g. multiple timers with different capabilities are available or peripherals conflict with others).
-->

アプリケーションは、様々な部品を結合し、必要な機能が確実に実現できるようにします。
別システムに移植する際、アプリケーションは最も適合作業が求められる部分です。アプリケーションは、HAL実装を通じて、実際のハードウェアを初期化する必要があるからです。
異なるハードウェアの初期化は、極端に異なる場合があります。
ユーザの選択は、多くの場合、大きな影響があります。
コンポーネントが別の端子に物理的に接続されたり、ハードウェアバスが機器構成を満たすために外部ハードウェアを必要とする場合があったり、内部ペリフェラルを使えるようにするための異なるトレードオフがあったりします。例えば、異なる機能を持つ複数のタイマが利用可能であること、や、ペリフェラルが他のペリフェラルと競合すること、です。