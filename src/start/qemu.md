# QEMU

<!-- 
We'll start writing a program for the [LM3S6965], a Cortex-M3 microcontroller.
We have chosen this as our initial target because it can be emulated using QEMU
so you don't need to fiddle with hardware in this section and we can focus on
the tooling and the development process.
 -->

Cortex-M3マイクロコントローラの[LM3S6965]用にプログラムを書くところから始めましょう。
このLM3S6965を最初のターゲットとして選んだ理由は、QEMUを使ってエミュレーションできるからです。
このセクションでは、ハードウェアをいじる必要がなく、ツールと開発プロセスに集中できます。

[LM3S6965]: http://www.ti.com/product/LM3S6965

<!-- ## A non standard Rust program -->

## 標準ライブラリを使わないRustプログラム

<!-- 
We'll use the [`cortex-m-quickstart`] project template so go generate a new
project from it.
 -->

[`cortex-m-quickstart`]プロジェクトテンプレートを使用し、新しいプロジェクトを生成します。

[`cortex-m-quickstart`]: https://github.com/rust-embedded/cortex-m-quickstart

<!-- - Using `cargo-generate` -->

- `cargo-generate`を利用する場合

``` console
$ cargo generate --git https://github.com/rust-embedded/cortex-m-quickstart
 Project Name: app
 Creating project called `app`...
 Done! New project created /tmp/app

$ cd app
```

<!-- - Using `git` -->

- `git`を利用する場合

<!-- Clone the repository -->

レポジトリをクローンします。

``` console
$ git clone https://github.com/rust-embedded/cortex-m-quickstart app

$ cd app
```

<!-- And then fill in the placeholders in the `Cargo.toml` file -->

`Cargo.toml`のプレースホルダを埋めます。

``` console
$ cat Cargo.toml
```

``` toml
[package]
authors = ["{{authors}}"] # "{{authors}}" -> "John Smith"
edition = "2018"
name = "{{project-name}}" # "{{project-name}}" -> "awesome-app"
version = "0.1.0"

# ..

[[bin]]
name = "{{project-name}}" # "{{project-name}}" -> "awesome-app"
test = false
bench = false
```

<!-- - Using neither -->

- どちらも使わない場合

<!-- Grab the latest snapshot of the `cortex-m-quickstart` template and extract it. -->

`cortex-m-quickstart`テンプレートの最新スナップショットを入手し、展開します。

<!-- Using the command line: -->

コマンドラインを利用する場合:

``` console
$ # 注記 tar形式でも入手可能です: archive/master.tar.gz
$ curl -LO https://github.com/rust-embedded/cortex-m-quickstart/archive/master.zip

$ unzip master.zip

$ mv cortex-m-quickstart-master app

$ cd app
```

<!-- 
OR you can browse to [`cortex-m-quickstart`], click the green "Clone or
download" button and then click "Download ZIP".
 -->

もしくは、[`cortex-m-quickstart`]をウェブブラウザで開いて、緑色の「Clone or download」ボタンをクリックして、
「Download ZIP」をクリックします。

<!-- 
Then fill in the placeholders in the `Cargo.toml` file as done in the second
part of the "Using `git`" version.
 -->

次に、`Cargo.toml`ファイルのプレースホルダを「`git`を利用する場合」の2つ目のパートにある通り埋めます。

<!-- 
**IMPORTANT** We'll use the name "app" for the project name in this tutorial.
Whenever you see the word "app" you should replace it with the name you selected
for your project. Or, you could also name your project "app" and avoid the
substitutions.
 -->

**重要** このチュートリアルでは、「app」という名前をプロジェクト名に使います。
「app」という単語が出てきた場合、それをあなたのプロジェクトにつけた名前に置き替えなければなりません。
または、プロジェクトに「app」という名前をつけると、置き替える必要がなくなります。

<!-- For convenience here's the source code of `src/main.rs`: -->

これは、`src/main.rs`のソースコードです。

``` console
$ cat src/main.rs
```

``` rust
#![no_std]
#![no_main]

# // pick a panicking behavior
// パニック発生時の挙動を選びます
# // extern crate panic_halt; // you can put a breakpoint on `rust_begin_unwind` to catch panics
extern crate panic_halt; // パニックをキャッチするため、`rust_begin_unwind`にブレイクポイントを設定できます
# // extern crate panic_abort; // requires nightly
// extern crate panic_abort; // nightlyが必要です
# // extern crate panic_itm; // logs messages over ITM; requires ITM support
// extern crate panic_itm; // ITMを介してメッセージをログ出力します; ITMサポートが必要です
# // extern crate panic_semihosting; // logs messages to the host stderr; requires a debugger
// extern crate panic_semihosting; // ホストの標準エラーにメッセージをログ出力します; デバッガが必要です。

use cortex_m_rt::entry;

#[entry]
fn main() -> ! {
    loop {
#         // your code goes here
        // あなたのコードはここに書きます
    }
}
```

<!-- 
This program is a bit different from a standard Rust program so let's take a
closer look.
 -->

このプログラムは、標準的なRustプログラムとは少し異なりますので、もう少し詳しく見てみましょう。

<!-- 
`#![no_std]` indicates that this program will *not* link to the standard crate,
`std`. Instead it will link to its subset: the `core` crate.
 -->

`#![no_std]`はこのプログラムが、標準クレートである`std`にリンク*しない*ことを意味します。
代わりに、そのサブセットである`core`クレートにリンクします。

<!--
`#![no_main]` indicates that this program won't use the standard `main`
interface that most Rust programs use. The main (no pun intended) reason to go
with `no_main` is that using the `main` interface in `no_std` context requires
nightly.
-->

`#![no_main]`は、ほとんどのRustプログラムが使用する標準の`main`インタフェースを、
このプログラムでは使用しないことを示します。
`no_main`を利用する主な理由は、`no_std`の状況で`main`インタフェースを使用するにはnightlyが必要だからです。

<!-- 
`extern crate panic_halt;`. This crate provides a `panic_handler` that defines
the panicking behavior of the program. More on this later on.
 -->

`extern crate panic_halt;`。このクレートは、プログラムのパニック発生時の挙動を定義する`panic_handler`を提供します。
後ほど、より詳しく説明します。

<!--
[`#[entry]`] is an attribute provided by the [`cortex-m-rt`] crate that's used
to mark the entry point of the program. As we are not using the standard `main`
interface we need another way to indicate the entry point of the program and
that'd be `#[entry]`.
-->

[`#[entry]`]は、[`cortex-m-rt`]クレートが提供するアトリビュートで、プログラムのエントリポイントを示すために使用します。
標準の`main`インタフェースを使用しないので、プログラムのエントリポイントを示す別の方法が必要です。それが、`#[entry]`です。

[`#[entry]`]: https://docs.rs/cortex-m-rt-macros/latest/cortex_m_rt_macros/attr.entry.html  

[`cortex-m-rt`]: https://crates.io/crates/cortex-m-rt

<!--
`fn main() -> !`. Our program will be the *only* process running on the target
hardware so we don't want it to end! We use a divergent function (the `-> !`
bit in the function signature) to ensure at compile time that'll be the case.
-->

`fn main() -> !`。ターゲットハードウェア上で動作しているのは私たちのプログラム*だけ*なので、
終了させたくありません。
コンパイル時、確実にそうなるように、発散する関数を使います（関数シグネチャの`-> !`部分）。

<!-- ### Cross compiling -->

### クロスコンパイル

<!--
The next step is to *cross* compile the program for the Cortex-M3 architecture.
That's as simple as running `cargo build --target $TRIPLE` if you know what the
compilation target (`$TRIPLE`) should be. Luckily, the `.cargo/config` in the
template has the answer:
-->

次のステップは、プログラムをCortex-M3アーキテクチャ向けに*クロス*コンパイルすることです。
これはコンパイルターゲット（`$TRIPLE`）が何かわかっていれば、`cargo build --target $TRIPLE`を実行するだけで簡単にできます。
コンパイルターゲットが何かは、テンプレート中の`.cargo/config`を見ればわかります。

``` console
$ tail -n6 .cargo/config
```

``` toml
[build]
# 以下のコンパイルターゲットから1つを選びます
# target = "thumbv6m-none-eabi"    # Cortex-M0およびCortex-M0+
target = "thumbv7m-none-eabi"    # Cortex-M3
# target = "thumbv7em-none-eabi"   # Cortex-M4およびCortex-M7 (no FPU)
# target = "thumbv7em-none-eabihf" # Cortex-M4FおよびCortex-M7F (with FPU)
```

<!-- 
To cross compile for the Cortex-M3 architecture we have to use
`thumbv7m-none-eabi`. This compilation target has been set as the default so the
two commands below do the same:
 -->

Cortex-M3アーキテクチャ向けにクロスコンパイルするためには、`thumbv7m-none-eabi`を使う必要があります。
このコンパイルターゲットは、デフォルトとして設定されているため、下記2つのコマンドは同じ意味になります。

``` console
$ cargo build --target thumbv7m-none-eabi

$ cargo build
```

<!-- ### Inspecting -->

### 確認

<!-- 
Now we have a non-native ELF binary in `target/thumbv7m-none-eabi/debug/app`. We
can inspect it using `cargo-binutils`.
 -->

今、`target/thumbv7m-none-eabi/debug/app`に、非ネイティブなバイナリがあります。
`cargo-binutils`を使って、このバイナリを確認することができます。

<!-- 
With `cargo-readobj` we can print the ELF headers to confirm that this is an ARM
binary.
 -->

このバイナリがARMバイナリであることを確かめるために、`cargo-readobj`でELFヘッダを表示できます。

``` console
$ # `--bin app`は`target/$TRIPLE/debug/app`のバイナリを確認するためのシンタックスシュガーです
$ # `--bin app`は必要に応じて、バイナリを（再）コンパイルもします

$ cargo readobj --bin app -- -file-headers
```

``` text
ELF Header:
  Magic:   7f 45 4c 46 01 01 01 00 00 00 00 00 00 00 00 00
  Class:                             ELF32
  Data:                              2's complement, little endian
  Version:                           1 (current)
  OS/ABI:                            UNIX - System V
  ABI Version:                       0x0
  Type:                              EXEC (Executable file)
  Machine:                           ARM
  Version:                           0x1
  Entry point address:               0x405
  Start of program headers:          52 (bytes into file)
  Start of section headers:          153204 (bytes into file)
  Flags:                             0x5000200
  Size of this header:               52 (bytes)
  Size of program headers:           32 (bytes)
  Number of program headers:         2
  Size of section headers:           40 (bytes)
  Number of section headers:         19
  Section header string table index: 18
```
<!-- 
`cargo-size` can print the size of the linker sections of the binary.
 -->

`cargo-size`はバイナリのリンカセクションのサイズを表示できます。

<!-- 
> **NOTE** this output assumes that rust-embedded/cortex-m-rt#111 has been
> merged
 -->

> **注記** この出力は、rust-embedded/cortex-m-rt#111がマージされていることを前提とします

``` console
$ # 最適化されたバイナリを確認するために`--release`を使います。

$ cargo size --bin app --release -- -A
```

``` text
app  :
section             size        addr
.vector_table       1024         0x0
.text                 92       0x400
.rodata                0       0x45c
.data                  0  0x20000000
.bss                   0  0x20000000
.debug_str          2958         0x0
.debug_loc            19         0x0
.debug_abbrev        567         0x0
.debug_info         4929         0x0
.debug_ranges         40         0x0
.debug_macinfo         1         0x0
.debug_pubnames     2035         0x0
.debug_pubtypes     1892         0x0
.ARM.attributes       46         0x0
.debug_frame         100         0x0
.debug_line          867         0x0
Total              14570
```

<!--
> A refresher on ELF linker sections
>
> - `.text` contains the program instructions
> - `.rodata` contains constant values like strings
> - `.data` contains statically allocated variables whose initial values are
>   *not* zero
> - `.bss` also contains statically allocated variables whose initial values
>   *are* zero
> - `.vector_table` is a *non*-standard section that we use to store the vector
>   (interrupt) table
> - `.ARM.attributes` and the `.debug_*` sections contain metadata and will
>   *not* be loaded onto the target when flashing the binary.
-->

> ELFリンカセクションの補足
>
> - `.text`は、プログラムの実行コードを含んでいます
> - `.rodata`は、文字列のような定数を含んでいます
> - `.data`は、初期値が0*ではない*静的に割り当てられた変数が格納されています
> - `.bss`も静的に割り当てられた変数が格納されますが、その*初期値は0です*
> - `.vector_table`は、*非*標準のセクションです。（割り込み）ベクタテーブルを格納するために使用します
> - `.ARM.attributes`と`.debug_*`セクションはメタデータを含んでおり、バイナリをフラッシュに書き込む際、
>   ターゲットボード上にロード*されません*

<!--
**IMPORTANT**: ELF files contain metadata like debug information so their *size
on disk* does *not* accurately reflect the space the program will occupy when
flashed on a device. *Always* use `cargo-size` to check how big a binary really
is.
-->

**重要**: ELFファイルは、デバッグ情報といったメタデータを含んでいるため、*そのディスク上のサイズ*は、
プログラムがデバイスに書き込まれた時に専有するスペースを正確に反映して*いません*。
実際のバイナリサイズを確認するために、*常に*`cargo-size`を使用して下さい。

<!-- `cargo-objdump` can be used to disassemble the binary. -->

`cargo-objdump`は、バイナリをディスアセンブルするために使用できます。

``` console
$ cargo objdump --bin app --release -- -disassemble -no-show-raw-insn -print-imm-hex
```

<!--
> **NOTE** this output assumes that rust-embedded/cortex-m-rt#111 has been
> merged
-->

> **注記** この出力は、rust-embedded/cortex-m-rt#111がマージされていることを前提とします

``` text
app:    file format ELF32-arm-little

Disassembly of section .text:
Reset:
     400:       bl      #0x36
     404:       movw    r0, #0x0
     408:       movw    r1, #0x0
     40c:       movt    r0, #0x2000
     410:       movt    r1, #0x2000
     414:       bl      #0x2c
     418:       movw    r0, #0x0
     41c:       movw    r1, #0x45c
     420:       movw    r2, #0x0
     424:       movt    r0, #0x2000
     428:       movt    r1, #0x0
     42c:       movt    r2, #0x2000
     430:       bl      #0x1c
     434:       b       #-0x4 <Reset+0x34>

HardFault_:
     436:       b       #-0x4 <HardFault_>

UsageFault:
     438:       b       #-0x4 <UsageFault>

__pre_init:
     43a:       bx      lr

HardFault:
     43c:       mrs     r0, msp
     440:       bl      #-0xe

__zero_bss:
     444:       movs    r2, #0x0
     446:       b       #0x0 <__zero_bss+0x6>
     448:       stm     r0!, {r2}
     44a:       cmp     r0, r1
     44c:       blo     #-0x8 <__zero_bss+0x4>
     44e:       bx      lr

__init_data:
     450:       b       #0x2 <__init_data+0x6>
     452:       ldm     r1!, {r3}
     454:       stm     r0!, {r3}
     456:       cmp     r0, r2
     458:       blo     #-0xa <__init_data+0x2>
     45a:       bx      lr
```

<!-- ### Running -->

### 実行

<!--
Next, let's see how to run an embedded program on QEMU! This time we'll use the
`hello` example which actually does something.
-->

次は、QEMUで組込みプログラムを実行する方法を見ていきましょう。
今回は、実際に何かを行う`hello`の例を使います。

<!-- For convenience here's the source code of `src/main.rs`: -->

便宜上の`src/main.rs`のソースコードです:

``` console
$ cat examples/hello.rs
```

<!--
下から4行目のコメント`debugger section`は、`debugger session`の誤記と考えられるため、
翻訳もデバッガセッションとしています。
-->

``` rust
# //! Prints "Hello, world!" on the host console using semihosting
//! セミホスティングを使って"Hello, world!"をホストのコンソールに表示します

#![no_main]
#![no_std]

extern crate panic_halt;

use core::fmt::Write;

use cortex_m_rt::entry;
use cortex_m_semihosting::{debug, hio};

#[entry]
fn main() -> ! {
    let mut stdout = hio::hstdout().unwrap();
    writeln!(stdout, "Hello, world!").unwrap();

#     // exit QEMU or the debugger section
    // QEMUもしくはデバッガセッションを終了します
    debug::exit(debug::EXIT_SUCCESS);

    loop {}
}
```

<!--
This program uses something called semihosting to print text to the *host*
console. When using real hardware this requires a debug session but when using
QEMU this Just Works.
-->

このプログラムは、*ホスト*コンソールにテキストを表示するために、セミホスティングと呼ばれるものを使います。
実際のハードウェアを使用する場合、セミホスティングはデバッグセッションを必要としますが、
QEMUを使う場合、これで機能します。

<!-- Let's start by compiling the example: -->

例をコンパイルすることから始めましょう。

``` console
$ cargo build --example hello
```

<!--
The output binary will be located at
`target/thumbv7m-none-eabi/debug/examples/hello`.
-->

`target/thumbv7m-none-eabi/debug/examples/hello`に出力バイナリがあります。

<!-- To run this binary on QEMU run the following command: -->

QEMU上でこのバイナリを動かすために、次のコマンドを実行して下さい。

``` console
$ qemu-system-arm \
      -cpu cortex-m3 \
      -machine lm3s6965evb \
      -nographic \
      -semihosting-config enable=on,target=native \
      -kernel target/thumbv7m-none-eabi/debug/examples/hello
Hello, world!
```

<!--
The command should successfully exit (exit code = 0) after printing the text. On
*nix you can check that with the following command:
-->

上記コマンドは、テキストを表示したあと、正常終了（終了コードが0）するはずです。
*nixでは、次のコマンドで正常終了したことを確認できます。

``` console
$ echo $?
0
```

<!-- Let me break down that long QEMU command for you: -->

この長いQEMUコマンドを分解して説明します。

<!--
- `qemu-system-arm`. This is the QEMU emulator. There are a few variants of
  these QEMU binaries; this one does full *system* emulation of *ARM* machines
  hence the name.
-->

- `qemu-system-arm`。これはQEMUエミュレータです。QEMUにはいくつかのバイナリがあります。
  このバイナリは、*ARM*マシンのフル*システム*をエミュレーションするので、この名前になっています。

<!--
- `-cpu cortex-m3`. This tells QEMU to emulate a Cortex-M3 CPU. Specifying the
  CPU model lets us catch some miscompilation errors: for example, running a
  program compiled for the Cortex-M4F, which has a hardware FPU, will make QEMU
  error during its execution.
-->

- `-cpu cortex-m3`。QEMUに、Cortex-M3 CPUをエミュレーションするように伝えます。
  CPUモデルを指定すると、いくつかのコンパイルミスのエラーを検出できます。例えば、
  ハードウェアFPUを搭載しているCortex-M4F用にコンパイルしたプログラムを実行すると、
  実行中にQEMUがエラーを発生させるでしょう。

<!--
- `-machine lm3s6965evb`. This tells QEMU to emulate the LM3S6965EVB, a
  evaluation board that contains a LM3S6965 microcontroller.
-->

- `-machine lm3s6965evb`。QEMUに、LM3S6965EVBをエミュレーションするように伝えます。
  LM3S6965EVBは、LM3S6965マイクロコントローラを搭載している評価ボードです。

<!-- - `-nographic`. This tells QEMU to not launch its GUI. -->

- `-nographic`。QEMUがGUIを起動しないようにします。

<!--
- `-semihosting-config (..)`. This tells QEMU to enable semihosting. Semihosting
  lets the emulated device, among other things, use the host stdout, stderr and
  stdin and create files on the host.
-->

- `-semihosting-config (..)`。QEMUのセミホスティングを有効にします。セミホスティングにより、
  エミュレーションされたデバイスは、ホストの標準出力、標準エラー、標準入力を使用できるようになり、
  ホスト上にファイルを作成することができます。

<!--
- `-kernel $file`. This tells QEMU which binary to load and run on the emulated
  machine.
-->

- `-kernel $file`。QEMUに、エミュレーションしたマシン上にロードして、実行するバイナリを教えます。

<!--
Typing out that long QEMU command is too much work! We can set a custom runner
to simplify the process. `.cargo/config` has a commented out runner that invokes
QEMU; let's uncomment it:
-->

この長いQEMUコマンドを入力するのは大変過ぎます。このプロセスを簡略化するために、
カスタムランナーを設定できます。`.cargo/config`には、QEMUを起動するランナーが、
コメントアウトされた状態であります。コメントアウトを外して下さい。

``` console
$ head -n3 .cargo/config
```

``` toml
[target.thumbv7m-none-eabi]
# `cargo run`で、プログラムをQEMUで実行するため、コメントアウトを外して下さい。
runner = "qemu-system-arm -cpu cortex-m3 -machine lm3s6965evb -nographic -semihosting-config enable=on,target=native -kernel"
```

<!--
This runner only applies to the `thumbv7m-none-eabi` target, which is our
default compilation target. Now `cargo run` will compile the program and run it
on QEMU:
-->

このランナーは、デフォルトのコンパイルターゲットである`thumbv7m-none-eabi`のみに適用されます。
これで、`cargo run`はプログラムをコンパイルしてQEMUで実行します。

``` console
$ cargo run --example hello --release
   Compiling app v0.1.0 (file:///tmp/app)
    Finished release [optimized + debuginfo] target(s) in 0.26s
     Running `qemu-system-arm -cpu cortex-m3 -machine lm3s6965evb -nographic -semihosting-config enable=on,target=native -kernel target/thumbv7m-none-eabi/release/examples/hello`
Hello, world!
```

<!-- ### Debugging -->

### デバッグ

<!-- Debugging is critical to embedded development. Let's see how it's done. -->

デバッグは組込み開発にとって非常に重要です。どのように行うのか、見てみましょう。

<!--
Debugging an embedded device involves *remote* debugging as the program that we
want to debug won't be running on the machine that's running the debugger
program (GDB or LLDB).
-->

組込みデバイスのデバッグは、*リモート*デバッグを伴います。デバッグしたいプログラムは、
デバッガプログラム（GDBまたはLLDB）を実行しているマシン上で実行されないためです。

<!--
Remote debugging involves a client and a server. In a QEMU setup, the client
will be a GDB (or LLDB) process and the server will be the QEMU process that's
also running the embedded program.
-->

リモートデバッグは、クライアントとサーバからなります。QEMUのセットアップで、
クライアントはGDB（またはLLDB）プロセスとなり、サーバは組込みプログラムを実行しているQEMUプロセスとなります。

<!-- In this section we'll use the `hello` example we already compiled. -->

このセクションでは、コンパイル済みの`hello`の例を使用します。

<!-- The first debugging step is to launch QEMU in debugging mode: -->

最初のデバッグステップは、QEMUをデバッグモードで起動することです。

``` console
$ qemu-system-arm \
      -cpu cortex-m3 \
      -machine lm3s6965evb \
      -nographic \
      -semihosting-config enable=on,target=native \
      -gdb tcp::3333 \
      -S \
      -kernel target/thumbv7m-none-eabi/debug/examples/hello
```

<!--
This command won't print anything to the console and will block the terminal. We
have passed two extra flags this time:
-->

このコマンドは、コンソールに何も表示せず、端末をブロックします。
ここでは2つの追加フラグを渡しています。

<!--
- `-gdb tcp::3333`. This tells QEMU to wait for a GDB connection on TCP
  port 3333.
-->

- `-gdb tcp::3333`。QEMUがTCPポート3333番で、GDBコネクションを待つようにします。

<!--
- `-S`. This tells QEMU to freeze the machine at startup. Without this the
  program would have reached the end of main before we had a chance to launch
  the debugger!
-->

- `-S`。QEMUが、起動時に、マシンをフリーズします。このフラグがないと、
  デバッガを起動する前に、プログラムがmain関数の終わりに到達してしまいます。

<!--
Next we launch GDB in another terminal and tell it to load the debug symbols of
the example:
-->

次に別の端末でGDBを起動し、`hello`の例のデバッグシンボルをロードします。

``` console
$ <gdb> -q target/thumbv7m-none-eabi/debug/examples/hello
```

<!--
**NOTE**: `<gdb>` represents a GDB program capable of debugging ARM binaries.
This could be `arm-none-eabi-gdb`, `gdb-multiarch` or `gdb` depending on your
system -- you may have to try all three.
-->

**注記**: `<gdb>`はARMバイナリをデバッグ可能なGDBを意味します。
あなたが利用しているシステムに依存して、`arm-none-eabi-gdb`か、`gdb-multiarch`、`gdb`になります。
3つ全てを試してみる必要があるかもしれません。

<!--
Then within the GDB shell we connect to QEMU, which is waiting for a connection
on TCP port 3333.
-->

すると、GDBシェルは、TCPポート3333番で接続を待っていたQEMUに接続します。

``` console
(gdb) target remote :3333
Remote debugging using :3333
Reset () at $REGISTRY/cortex-m-rt-0.6.1/src/lib.rs:473
473     pub unsafe extern "C" fn Reset() -> ! {
```

<!--
You'll see that the process is halted and that the program counter is pointing
to a function named `Reset`. That is the reset handler: what Cortex-M cores
execute upon booting.
-->

プロセスは停止しており、プログラムカウンタが`Reset`という名前の関数を指していることがわかります。
`Reset`関数は、Cortex-Mコアが起動時に実行するリセットハンドラです。

<!--
This reset handler will eventually call our main function. Let's skip all the
way there using a breakpoint and the `continue` command:
-->

このリセットハンドラは、最終的に、私たちのメイン関数を呼び出します。
ブレイクポイントと`continue`コマンドを使って、メイン関数呼び出しまでスキップしましょう。

``` console
(gdb) break main
Breakpoint 1 at 0x400: file examples/panic.rs, line 29.

(gdb) continue
Continuing.

Breakpoint 1, main () at examples/hello.rs:17
17          let mut stdout = hio::hstdout().unwrap();
```

<!--
We are now close to the code that prints "Hello, world!". Let's move forward
using the `next` command.
-->

「Hello, world!」を表示するコードに近づいてきました。
`next`コマンドを使って、先へ進みましょう。

``` console
(gdb) next
18          writeln!(stdout, "Hello, world!").unwrap();

(gdb) next
20          debug::exit(debug::EXIT_SUCCESS);
```

<!--
At this point you should see "Hello, world!" printed on the terminal that's
running `qemu-system-arm`.
-->

この時点で、`qemu-system-arm`を実行している端末に「Hello, world」が表示されるはずです。

``` console
$ qemu-system-arm (..)
Hello, world!
```

<!--
Calling `next` again will terminate the QEMU process.
-->

もう1度`next`を実行すると、QEMUプロセスが終了します。

``` console
(gdb) next
[Inferior 1 (Remote target) exited normally]
```

<!--
You can now exit the GDB session.
-->

これでGDBセッションを終了できます。

``` console
(gdb) quit
```
