<!-- # A `no_std` Rust Environment -->

# Rustの`no_std`環境
<!--
The term Embedded Programming is used for a wide range of different classes of programming.
Ranging from programming 8-Bit MCUs (like the [ST72325xx](https://www.st.com/resource/en/datasheet/st72325j6.pdf))
with just a few KB of RAM and ROM, up to systems like the Raspberry Pi
([Model B 3+](https://en.wikipedia.org/wiki/Raspberry_Pi#Specifications)) which has a 32/64-bit
4-core Cortex-A53 @ 1.4 GHz and 1GB of RAM. Different restrictions/limitations will apply when writing code
depending on what kind of target and use case you have.
-->

組込みプログラミングという用語は、様々な分野のプログラミングに使用されます。
たった数キロバイトのRAMかROMが付随する8ビットMCU (例えば、[ST72325xx](https://www.st.com/resource/en/datasheet/st72325j6.pdf))から、32/64ビットの4コア Cortex-A53 @ 1.4GHzと1GBのRAMが搭載されたRaspberry Pi([Model B 3+](https://en.wikipedia.org/wiki/Raspberry_Pi#Specifications))のようなシステムまで、幅広いです。
どのような種類の目的とユースケースがあるか、によって、コードを書くときに異なる制限/限界が課されます。

<!-- There are two general Embedded Programming classifications: -->

2つの一般的な組込みプログラミングの分類があります。
<!-- 
## Hosted Environments
These kinds of environments are close to a normal PC environment.
What this means is you are provided with a System Interface [E.G. POSIX](https://en.wikipedia.org/wiki/POSIX)
that provides you with primitives to interact with various systems, such as file systems, networking, memory management, threads, etc.
Standard libraries in turn usually depend on these primitives to implement their functionality.
You may also have some sort of sysroot and restrictions on RAM/ROM-usage, and perhaps some
special HW or I/Os. Overall it feels like coding on a special-purpose PC environment.
 -->

## ホストされた環境
この分類の環境は、普通のPCの環境に近いです。
これの意味するところは、[POSIX](https://en.wikipedia.org/wiki/POSIX)のようなシステムインタフェースが提供されている、ということです。システムインタフェースは、ファイルシステムやネットワーク、メモリ管理、スレッドといった多様なシステムとやりとりするための基本要素を提供します。
通常、標準ライブラリは、その機能を実装するために、これらの基本要素に依存します。
また、sysrootや、RAM/ROM利用の制限、そしておそらく特別なハードウェアやIOがあるかもしれません。
全体としては、特殊な用途のPC環境でコーディングをするようなものです。

<!-- 
## Bare Metal Environments
In a bare metal environment there will be no high-level OS running and hosting our code.
This means there will be no primitives, which means there's also no standard library by default.
By marking our code with `no_std` we indicate that our code is capable of running in such an environment.
This means the rust [libstd](https://doc.rust-lang.org/std/) and dynamic memory allocation can't be used by such code.
However, such code can use [libcore](https://doc.rust-lang.org/core/), which can easily be made available
in any kind of environment by providing just a few symbols (for details see [libcore](https://doc.rust-lang.org/core/)).
 -->

## ベアメタル環境
ベアメタル環境では、高機能なOSが動作していて、私たちのコードをホスティングしてくれる、ということはありません。
これは、基本要素がないことを意味しており、それ故に、デフォルトでは標準ライブラリもありません。
コードに`no_std`のマーキングをすることで、そのコードが、ベアメタル環境で実行できることを示します。
no\_stdなコードからは、Rustの[libstd](https://doc.rust-lang.org/std/)とメモリの動的確保が使えません。
しかしながら、no\_stdなコードでも[libcore](https://doc.rust-lang.org/core/)を使うことができます。libcoreは、ほんの数種類のシンボルを提供することで、いかなる環境でも容易に利用することができます (詳細は、[libcore](https://doc.rust-lang.org/core/)を参照して下さい)。

<!-- 
### The libstd Runtime
As mentioned before using [libstd](https://doc.rust-lang.org/std/) requires some sort of system integration, but this is not only because
[libstd](https://doc.rust-lang.org/std/) is just providing a common way of accessing OS abstractions, it also provides a runtime.
This runtime, among other things, takes care of setting up stack overflow protection, processing command line arguments,
and spawning the main thread before a program's main function is invoked. This runtime also won't be available in a `no_std` environment.
 -->

### libstdランタイム
上述の通り、[libstd](https://doc.rust-lang.org/std/)の利用には、いくらかのシステムインテグレーションが必要です。しかし、これは[libstd](https://doc.rust-lang.org/std/)がOSの抽象にアクセスするための共通の方法を提供しているだけでなく、ランタイムも提供しているためです。
ランタイムは、とりわけ、スタックオーバーフロープロテクションの準備、コマンドライン引数の処理、メインスレッドの生成、をプログラムのメイン関数が呼び出される前に処理します。
このラインタムも、`no_std`環境では利用できません。

<!-- 
## Summary
`#![no_std]` is a crate-level attribute that indicates that the crate will link to the core-crate instead of the std-crate.
The [libcore](https://doc.rust-lang.org/core/) crate in turn is a platform-agnostic subset of the std crate
which makes no assumptions about the system the program will run on.
As such, it provides APIs for language primitives like floats, strings and slices, as well as APIs that expose processor features
like atomic operations and SIMD instructions. However it lacks APIs for anything that involves platform integration.
Because of these properties no\_std and [libcore](https://doc.rust-lang.org/core/) code can be used for any kind of
bootstrapping (stage 0) code like bootloaders, firmware or kernels.
 -->

## まとめ
`#![no_std]`は、クレートレベルの属性で、そのクレートがstdクレートの代わりにcoreクレートとリンクすることを意味します。
[libcore](https://doc.rust-lang.org/core/)クレートは、プラットフォームに依存しないstdクレートのサブセットです。libcoreクレートは、プログラムが動作するシステムについて前提を置きません。
libcoreクレートは、浮動小数点、文字列やスライスといった言語の基本要素となるAPIと、アトミック操作やSIMD命令といったプロセッサの機能を公開するAPIとを、提供します。一方、プラットフォームインテグレーションを伴うようなAPIは欠如しています。
これらの特性のため、no\_stdと[libcore](https://doc.rust-lang.org/core/)のコードは、ブートローダー、ファームウェア、カーネルといったあらゆるブートストラップ (ステージ0)のコードにも利用できます。

<!-- 
### Overview

| feature                                                   | no\_std | std |
|-----------------------------------------------------------|--------|-----|
| heap (dynamic memory)                                     |   *    |  ✓  |
| collections (Vec, HashMap, etc)                           |  **    |  ✓  |
| stack overflow protection                                 |   ✘    |  ✓  |
| runs init code before main                                |   ✘    |  ✓  |
| libstd available                                          |   ✘    |  ✓  |
| libcore available                                         |   ✓    |  ✓  |
| writing firmware, kernel, or bootloader code              |   ✓    |  ✘  |

\* Only if you use the `alloc` crate and use a suitable allocator like [alloc-cortex-m].

\** Only if you use the `collections` crate and configure a global default allocator.

[alloc-cortex-m]: https://github.com/rust-embedded/alloc-cortex-m
 -->

### 概略

| 機能                                                       | no\_std | std |
|-----------------------------------------------------------|--------|-----|
| ヒープ (動的メモリ)                                          |   *    |  ✓  |
| コレクション (Vec, HashMap, など)                            |  **    |  ✓  |
| スタックオーバーフロープロテクション                            |   ✘    |  ✓  |
| main関数前の初期化コード実行                                  |   ✘    |  ✓  |
| libstdの利用                                               |   ✘    |  ✓  |
| libcoreの利用                                              |   ✓    |  ✓  |
| ファームウェア、カーネル、ブートローダーのコードを書く             |   ✓    |  ✘  |

\* `alloc`クレートを使い、[alloc-cortex-m]のような適切なアロケータを使った場合のみ

\** `collections`クレートを使い、グローバルなデフォルトアロケータを設定した場合のみ

<!-- 
## See Also
* [FAQ](https://www.rust-lang.org/en-US/faq.html#does-rust-work-without-the-standard-library)
* [RFC-1184](https://github.com/rust-lang/rfcs/blob/master/text/1184-stabilize-no_std.md)
 -->

## 参照
* [FAQ](https://www.rust-lang.org/en-US/faq.html#does-rust-work-without-the-standard-library)
* [RFC-1184](https://github.com/rust-lang/rfcs/blob/master/text/1184-stabilize-no_std.md)