<!-- # Tooling -->

# ツール

<!-- 
Dealing with microcontrollers involves using several different tools as we'll be
dealing with an architecture different than your laptop's and we'll have to run
and debug programs on a *remote* device.
 -->

マイクロコントローラを扱う際は、いくつかの異なるツールを利用することになります。あなたのノートPCとは異なるアーキテクチャを扱うことになるでしょうし、*リモート*デバイス上でプログラムを実行しデバッグする必要があります。

<!-- 
We'll use all the tools listed below. Any recent version should work when a
minimum version is not specified, but we have listed the versions we have
tested.
 -->

下記リストのツールを利用します。最小バージョンが指定されていない場合、新しいバージョンであれば機能するはずです。私たちがテストしたバージョンをリストに示しています。

<!-- 
- Rust 1.31, 1.31-beta, or a newer toolchain PLUS ARM Cortex-M compilation
  support.
- [`cargo-binutils`](https://github.com/rust-embedded/cargo-binutils) ~0.1.4
- [`qemu-system-arm`](https://www.qemu.org/). Tested versions: 3.0.0
- OpenOCD >=0.8. Tested versions: v0.9.0 and v0.10.0
- GDB with ARM support. Version 7.12 or newer highly recommended. Tested
  versions: 7.10, 7.11, 7.12 and 8.1
- [OPTIONAL] `git` OR
  [`cargo-generate`](https://github.com/ashleygwilliams/cargo-generate). If you
  have neither installed then don't worry about installing either.
 -->

- ARM Cortex-Mコンパイルサポートを追加したRust 1.31、1.31-beta以上のツールチェイン。
- [`cargo-binutils`](https://github.com/rust-embedded/cargo-binutils) ~0.1.4
- [`qemu-system-arm`](https://www.qemu.org/)。テストしたバージョン: 3.0.0
- OpenOCD >=0.8.テストしたバージョン: v0.9.0とv0.10.0
- ARMサポートのGDB。バージョン7.12以上を強く推奨。テストしたバージョン: 7.10、7.11、7.12、8.1
- [任意] `git`または[`cargo-generate`](https://github.com/ashleygwilliams/cargo-generate)。どちらもインストールしていないのであれば、どちらをインストールしてもかまいません。

<!-- 
The text below explains why we are using these tools. Installation instructions
can be found on the next page.
 -->

下記に、なぜこれらのツールを利用するのか、を説明します。インストール方法は、次のページにあります。

<!-- ## `cargo-generate` OR `git` -->

## `cargo-generate`か`git`

<!-- 
Bare metal programs are non-standard (`no_std`) Rust programs that require some
fiddling with the linking process to get the memory layout of the program
right. All this requires unusual files (like linker scripts) and unusual
settings (like linker flags). We have packaged all that for you in a template
so that you only need to fill in the blanks such as the project name and the
characteristics of your target hardware.
 -->

ベアメタルプログラムは、非標準 (`no_std`)なRustプログラムであり、プログラムのメモリレイアウトを正しくするために、リンクプロセスをいじる必要があります。
これには、独特なファイル(リンカスクリプトなど)と設定(リンカフラグ)が必要です。
プロジェクト名やターゲットハードウェアの特徴などを、空白に入力するだけで済むように、テンプレートを用意しています。

Our template is compatible with `cargo-generate`: a Cargo subcommand for
creating new Cargo projects from templates. You can also download the
template using `git`, `curl`, `wget`, or your web browser.

このテンプレートは、`cargo-generate`と互換があります。`cargo-generate`は、テンプレートからCargoプロジェクトを作成するためのCargoのサブコマンドです。
このテンプレートは、`git`や`curl`、`wget`、ウェブブラウザを使ってダウンロードできます。

## `cargo-binutils`

<!-- 
`cargo-binutils` is a collection of Cargo subcommands that make it easy to use
the LLVM tools that are shipped with the Rust toolchain. These tools include the
LLVM versions of `objdump`, `nm` and `size` and are used for inspecting
binaries.
 -->

`cargo-binutils`は、Rustツールチェインとともに配布されているLLVMツールを、簡単に使えるようにするCargoサブコマンドの一式です。
これらのツールは、LLVMの`objdump`や`nm`、`size`を含んでおり、バイナリを調査するために使われます。

<!-- 
The advantage of using these tools over GNU binutils is that (a) installing the
LLVM tools is the same one-command installation (`rustup component add
llvm-tools-preview`) regardless of your OS and (b) tools like `objdump` support
all the architectures that `rustc` supports -- from ARM to x86_64 -- because
they both share the same LLVM backend.
 -->

GNU binutilsではなく、これらのツールを利用する利点は次の通りです。
(a) LLVMツールのインストールは、OSに関わらず、共通のコマンド1つ(`rustup component add llvm-tools-preview`)で済みます。
(b) `objdump`などのツールは、`rustc`がサポートする全てのアーキテクチャ(ARMからx86_64まで)をサポートします。これは、同じLLVMバックエンドを共有しているためです。

## `qemu-system-arm`

<!-- 
QEMU is an emulator. In this case we use the variant that can fully emulate ARM
systems. We use QEMU to run embedded programs on the host. Thanks to this you
can follow some parts of this book even if you don't have any hardware with you!
 -->

QEMUはエミュレータです。今回は、ARMシステムを完全にエミュレートできるものを使います。
私たちは、ホスト上で組込みプログラムを実行するためにQEMUを利用します。
このおかげで、ハードウェアを持っていなくても、この本のいくつかの部分を試すことができます。

## GDB

<!-- 
A debugger is a very important component of embedded development as you may not
always have the luxury to log stuff to the host console. In some cases, you may
not have LEDs to blink on your hardware!
 -->

デバッガは、組込み開発で非常に重要です。ホストコンソールにログを記録するような贅沢は、必ずしもできないからです。
場合によっては、ハードウェア上のLEDが点滅しないことがあります。

<!-- 
In general, LLDB works as well as GDB when it comes to debugging but we haven't
found an LLDB counterpart to GDB's `load` command, which uploads the program to
the target hardware, so currently we recommend that you use GDB.
 -->

通常デバッグに関しては、LLDBはGDBと同様に機能します。しかし、ターゲットハードウェアにプログラムをアップロードするGDBの`load`コマンド相当のものは、LLDBにはありません。
したがって、現在はGDBを使用することをお勧めします。

## OpenOCD

<!-- 
GDB isn't able to communicate directly with the ST-Link debugging hardware on
your STM32F3DISCOVERY development board. It needs a translator and the Open
On-Chip Debugger, OpenOCD, is that translator. OpenOCD is a program that runs
on your laptop/PC and translates between GDB's TCP/IP based remote debug
protocol and ST-Link's USB based protocol.
 -->

GDBは、STM32F3DISCOVERY開発ボード上のST-Linkデバッグハードウェアと直接通信することはできません。
翻訳プログラムが必要であり、OpenOCD (Open On-Chip Debugger)がその翻訳プログラムです。
OpenOCDは、ノートPCやPC上で動作するプログラムで、GDBのTCP/IPベースのリモートデバッグプロトコルとST-LinkのUSBベースのプロトコルとを翻訳します。

<!-- 
OpenOCD also performs other important work as part of its translation for the
debugging of the ARM Cortex-M based microcontroller on your STM32F3DISCOVERY
development board:
-->

STM32F3DISCOVERY開発ボード上のARM Cortex-Mベースのマイクロコントローラをデバッグするため、OpenOCDは翻訳の一環として、他の重要な役割も果たします。

<!-- 
* It knows how to interact with the memory mapped registers used by the ARM
  CoreSight debug peripheral. It is these CoreSight registers that allow for:
  * Breakpoint/Watchpoint manipulation
  * Reading and writing of the CPU registers
  * Detecting when the CPU has been halted for a debug event
  * Continuing CPU execution after a debug event has been encountered
  * etc.
* It also knows how to erase and write to the microcontroller's FLASH
 -->

* ARM CoreSightデバッグ周辺機器で使用されるメモリマップドレジスタとの通信方法を知っています。CoreSightレジスタは、次のことをできるようにします。
  * ブレイクポイント/ウォッチポイント操作
  * CPUレジスタの読み込みと書き込み
  * CPUがデバッグイベントのために停止したことの検出
  * デバッグイベントが発生した後の、CPU実行の継続
  * 他
* マイクロコントローラのフラッシュの消去と書き込み方法を知っています。
