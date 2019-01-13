<!-- # Introduction -->

# 導入

<!-- Welcome to The Embedded Rust Book: An introductory book about using the Rust -->
<!-- Programming Language on "Bare Metal" embedded systems, such as Microcontrollers. -->

*The Embedded Rust Book*へようこそ。Rustをマイクロコントローラのような、「ベアメタル」の組込みシステムで使うための入門書です。

<!-- ## Who Embedded Rust is For -->
<!-- Embedded Rust is for everyone who wants to do embedded programming backed by the higher-level concepts and safety guarantees the Rust language provides. -->
<!-- (See also [Who Rust Is For](https://doc.rust-lang.org/book/2018-edition/ch00-00-introduction.html)) -->

## 組込みRustは誰のためのもの
組込みRustは、Rustの高い抽象度と安全性のもと、組込みプログラミングをしたい人のためのものです。
([Rustは誰のためのもの](https://doc.rust-lang.org/book/2018-edition/ch00-00-introduction.html)も合わせて見て下さい)

<!-- ## Scope -->

## スコープ

<!-- The goals of this book are: -->

この本の目的は、以下の通りです。

<!-- * Get developers up to speed with embedded Rust development. i.e. How to set -->
<!--   up a development environment. -->

* 組込みRustをできる限り速く開始できるようにします。すなわち、開発環境のセットアップ方法です。

<!-- * Share *current* best practices about using Rust for embedded development. i.e. -->
<!--   How to best use Rust language features to write more correct embedded -->
<!--   software. -->

* 組込み開発におけるRustの*現在*のベストプラクティスを共有します。つまり、より正しい組込みソフトウェアを書くための、Rustの最善な利用方法です。

<!-- * Serve as a cookbook in some cases. e.g. How do I do mix C and Rust in a single -->
<!--   project? -->

* いくつかのケースに対するマニュアルを提供します。例えば、1つのプロジェクト内で、C言語とRustとを混在する方法です。

<!-- This book tries to be as general as possible but to make things easier for both -->
<!-- the readers and the writers it uses the ARM Cortex-M architecture in all its -->
<!-- examples. However, the book assumes that the reader is not familiar with this -->
<!-- particular architecture and explains details particular to this architecture -->
<!-- where required. -->

本書は出来る限り一般的な事項を取り扱います。ただし、説明を簡単にするために、全ての例で、ARM Cortex-Mアーキテクチャを利用します。
読者は、このアーキテクチャに詳しい必要はありません。本書では、アーキテクチャ固有の詳細について、必要に応じて説明をします。

<!-- ## Who This Book is For -->
<!-- This book caters towards people with either some embedded background or some Rust background, however we assume -->
<!-- everybody curious about embedded Rust programming can get something out of this book. For those without any prior knowledge -->
<!-- we suggest you read the "Assumptions and Prerequisites" section and catch up on missing knowledge to get more out of the book -->
<!-- and improve your reading experience. You can check out the "Other Resources" section to find resources on topics -->
<!-- you want to catch up on. -->

## この本は誰のためのもの
本書は、組込み開発か、Rustかのバックグラウンドを持つ人々に向けたものです。しかし、組込みRustに興味がある人なら、誰でも、この本から何かを得られると思います。本書による学習効果を高めるために、事前知識が不足している読者は、「仮定と前提条件」のセクションを読み、不足している知識を補うことをお勧めします。不足知識を補うリソースを見つけるために、「その他のリソース」セクションをチェックすることができます。

<!-- ### Assumptions and Prerequisites -->

### 仮定と前提条件

<!-- * You are comfortable using the Rust Programming Language, and have written, -->
<!--   run, and debugged Rust applications on a desktop environment. You should also -->
<!--   be familiar with the idioms of the [2018 edition] as this book targets -->
<!--   Rust 2018. -->

* Rustでのプログラミングを楽しんでおり、デスクトップ環境でRustアプリケーションを書いたり、実行したり、デバッグしたりしたことがあることを前提とします。また、本書ではRust 2018を対象とするため、2018 editionのイディオムに慣れ親しんでいる必要があります。

[2018 edition]: https://rust-lang-nursery.github.io/edition-guide/

<!-- * You are comfortable developing and debugging embedded systems in another -->
<!--   language such as C, C++, or Ada, and are familiar with concepts such as: -->
<!--     * Cross Compilation -->
<!--     * Memory Mapped Peripherals -->
<!--     * Interrupts -->
<!--     * Common interfaces such as I2C, SPI, Serial, etc. -->

* C, C++, Adaといった言語で組込みシステムを開発、デバッグすることに慣れており、次の概念になじみがあることを想定します。
    * クロスコンパイル
    * メモリマップ方式のペリフェラル
    * 割り込み
    * I2C、SPI、シリアルといった一般的なインタフェース

<!-- ### Other Resources -->
<!-- If you are unfamiliar with anything mentioned above or if you want more information about a specific topic mentioned in this book you might find some of these resources helpful. -->

### その他のリソース
もしあなたが上述した何らかの事項をよく知らない場合、もしくは、本書内の特定トピックに関して、より詳細な情報を知りたい場合、これらのリソースが役に立つでしょう。

<!--
| Topic        | Resource | Description |
|--------------|----------|-------------|
| Rust         | [Rust Book 2018 Edition](https://doc.rust-lang.org/book/2018-edition/index.html) | If you are not yet comfortable with Rust, we highly suggest reading the this book. |
| Rust, Embedded | [Embedded Rust Bookshelf](https://docs.rust-embedded.org) | Here you can find several other resources provided by Rust's Embedded Working Group. |
| Rust, Embedded | [Embedonomicon](https://docs.rust-embedded.org/embedonomicon/) | The nitty gritty details when doing embedded programming in Rust. |
| Rust, Embedded | [embedded FAQ](https://docs.rust-embedded.org/faq.html) | Frequently asked questions about Rust in an embedded context. |
| Interrupts | [Interrupt](https://en.wikipedia.org/wiki/Interrupt) | - |
| Memory-mapped IO/Peripherals | [Memory-mapped I/O](https://en.wikipedia.org/wiki/Memory-mapped_I/O) | - |
| SPI, UART, RS232, USB, I2C, TTL | [Stack Exchange about SPI, UART, and other interfaces](https://electronics.stackexchange.com/questions/37814/usart-uart-rs232-usb-spi-i2c-ttl-etc-what-are-all-of-these-and-how-do-th) | - |
-->

| トピック      | リソース   | 説明 |
|--------------|----------|-------------|
| Rust         | [Rust Book 2018 Edition](https://doc.rust-lang.org/book/2018-edition/index.html) | もしRustに親しんでいない場合、この本を読むことを強くお勧めします。 |
| Rust、組込み | [Embedded Rust Bookshelf](https://docs.rust-embedded.org) | Rust組込みワーキンググループによるいくらかのリソースがあります。 |
| Rust、組込み | [Embedonomicon](https://docs.rust-embedded.org/embedonomicon/) | Rustで組込みプログラミングを行うときのより深い詳細が記載されています。 |
| Rust、組込み | [embedded FAQ](https://docs.rust-embedded.org/faq.html) | 組込みでRustを使う際のよくある質問と回答です。 |
| 割り込み | [Interrupt](https://en.wikipedia.org/wiki/Interrupt) | - |
| メモリマップドI/O／ペリフェラル | [Memory-mapped I/O](https://en.wikipedia.org/wiki/Memory-mapped_I/O) | - |
| SPI, UART, RS232, USB, I2C, TTL | [Stack Exchange about SPI, UART, and other interfaces](https://electronics.stackexchange.com/questions/37814/usart-uart-rs232-usb-spi-i2c-ttl-etc-what-are-all-of-these-and-how-do-th) | - |

<!-- ## How to Use This Book -->

## この本はどう使う

<!-- This book generally assumes that you’re reading it front-to-back. Later -->
<!-- chapters build on concepts in earlier chapters, and earlier chapters may -->
<!-- not dig into details on a topic, revisiting the topic in a later chapter. -->

この本は、前から順番に読んでいくことを想定しています。後半の章は、前半の章で説明する概念に基づいて成り立っています。前半の章では、トピックの詳細に深入りせず、後半の章で再訪問します。

<!-- This book will be using the [STM32F3DISCOVERY] development board from -->
<!-- STMicroelectronics for the majority of the examples contained within. This board -->
<!-- is based on the ARM Cortex-M architecture, and while basic functionality is -->
<!-- common across most CPUs based on this architecture, peripherals and other -->
<!-- implementation details of Microcontrollers are different between different -->
<!-- vendors, and often even different between Microcontroller families from the same -->
<!-- vendor. -->

この本は、ほとんどの例で、STマイクロエレクトロニクスの[STM32F3DISCOVERY]開発ボードを使用します。このボードは、ARM Cortex-Mアーキテクチャをベースとしています。基本機能はこのアーキテクチャベースのCPUでは共通です。一方、ペリフェラルとマイクロコントローラ実装の詳細は、他のベンダーと異なります。同じSTマイクロエレクトロニクスのマイクロコントローラファミリでも、違いがあります。

<!-- For this reason, we suggest purchasing the [STM32F3DISCOVERY] development board -->
<!-- for the purpose of following the examples in this book. -->

上記の理由から、本書内の例を理解するために、[STM32F3DISCOVERY]開発ボードを購入することをお勧めします。

[STM32F3DISCOVERY]: http://www.st.com/en/evaluation-tools/stm32f3discovery.html

<!-- > **HEADS UP** Until the official release of this book, which is planned to -->
<!-- > coincide with the 2018 edition release of the Rust Programming Language, -->
<!-- > expect the sections of this book to change quite a bit. We recommend -->
<!-- > bookmarking the root of this book instead of any specific section. -->

> **注意** 本書の公式リリースまで (Rust 2018 editionのリリースと当時になるように計画しています)、本書のセクションは大きく修正することが予測されます。特定のセクションではなく、本書のトップページをブックマークすることをお勧めします。

<!-- 日本語訳へのコントリビューション情報を代わりに掲載する？ -->

<!-- ## Contributing to This Book -->

<!-- The work on this book is coordinated in [this repository] and is mainly -->
<!-- developed by the [resources team]. -->

<!-- [this repository]: https://github.com/rust-embedded/book -->
<!-- [resources team]: https://github.com/rust-embedded/wg#the-resources-team -->

<!-- If you have trouble following the instructions in this book or find that some -->
<!-- section of the book is not clear enough or hard to follow then that's a bug and -->
<!-- it should be reported in [the issue tracker] of this book. -->

<!-- [the issue tracker]: https://github.com/rust-embedded/book/issues/ -->

<!-- Pull requests fixing typos and adding new content are very welcome! -->
