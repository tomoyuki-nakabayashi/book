<!-- # Hardware -->

# ハードウェア

<!--
By now you should be somewhat familiar with the tooling and the development
process. In this section we'll switch to real hardware; the process will remain
largely the same. Let's dive in.
-->

ここまでで、ツールと開発プロセスにある程度慣れたはずです。このセクションでは、実際のハードウェアに切り替えます。
開発プロセスは、ほとんど同じままです。飛び込みましょう。

<!-- ## Know your hardware -->

## ハードウェアを知る

<!--
Before we begin you need to identify some characteristics of the target device
as these will be used to configure the project:
-->

始める前に、プロジェクトの設定に利用するターゲットデバイスのいくつかの特徴を確認する必要があります。

<!-- - The ARM core. e.g. Cortex-M3. -->

- ARMコア、例えばCortex-M3です。

<!-- - Does the ARM core include an FPU? Cortex-M4**F** and Cortex-M7**F** cores do. -->

- そのARMコアはFPUを搭載していますか？Cortex-M4**F**とCortex-M7**F**は、搭載しています。

<!--
- How much Flash memory and RAM does the target device have? e.g. 256 KiB of
  Flash and 32 KiB of RAM.
-->

- ターゲットデバイスに搭載されているフラッシュメモリとRAMの容量はいくらですか？
  例えば、フラッシュは256KiBでRAMは32KiBです。

<!--
- Where are Flash memory and RAM mapped in the address space? e.g. RAM is
  commonly located at address `0x2000_0000`.
-->

- フラッシュメモリとRAMは、アドレス空間のどこにマッピングされていますか？
  例えば、RAMは、通常`0x2000_0000`番地に位置します。

<!--
You can find this information in the data sheet or the reference manual of your
device.
-->

これらの情報は、デバイスのデータシートかリファレンスマニュアルに掲載されています。

<!--
In this section we'll be using our reference hardware, the STM32F3DISCOVERY.
This board contains an STM32F303VCT6 microcontroller. This microcontroller has:
-->

このセクションでは、私たちのリファレンスハードウェアであるSTM32F3DISCOVERYを使用します。
このボードは、STM32F303VCT6マイクロコントローラを1つ搭載しています。このマイクロコントローラは以下のものを持っています。

<!-- - A Cortex-M4F core that includes a single precision FPU -->

- 単精度FPUを含むCortex-M4Fコアが1つ

<!-- - 256 KiB of Flash located at address 0x0800_0000. -->

- 0x0800_0000番地に配置された256KiBのフラッシュメモリ

<!--
- 40 KiB of RAM located at address 0x2000_0000. (There's another RAM region but
  for simplicity we'll ignore it).
-->

- 0x2000_0000番地に配置された40KiBのRAM。（別のRAM領域もありますが、説明の簡単化のため、取り扱いません）

<!-- ## Configuring -->

## 設定

<!--
We'll start from scratch with a fresh template instance. Refer to the
[previous section on QEMU] for a refresher on how to do this without
`cargo-generate`.
-->

テンプレートの新しいインスタンスを使って、スクラッチから書いていきましょう。
`cargo-generate`を使用しない方法については、[前セクションのQEMU]を参照して下さい。

<!-- [previous section on QEMU]: qemu.md -->

[前セクションのQEMU]: qemu.md

``` console
$ cargo generate --git https://github.com/rust-embedded/cortex-m-quickstart
 Project Name: app
 Creating project called `app`...
 Done! New project created /tmp/app

 $ cd app
```

<!-- Step number one is to set a default compilation target in `.cargo/config`. -->

第一ステップは、`.cargo/config`にデフォルトコンパイルターゲットを設定することです。

``` console
$ tail -n5 .cargo/config
```

``` toml
[build]
# 以下のコンパイルターゲットから1つを選びます
# target = "thumbv6m-none-eabi"    # Cortex-M0およびCortex-M0+
# target = "thumbv7m-none-eabi"    # Cortex-M3
# target = "thumbv7em-none-eabi"   # Cortex-M4およびCortex-M7 (no FPU)
target = "thumbv7em-none-eabihf" # Cortex-M4FおよびCortex-M7F (with FPU)
```

<!-- We'll use `thumbv7em-none-eabihf` as that covers the Cortex-M4F core. -->

Cortex-M4Fコアを対象とするものとして、`thumbv7em-none-eabihf`を使います。

<!--
The second step is to enter the memory region information into the `memory.x`
file.
-->

第二ステップは、`memory.x`ファイルにメモリ領域の情報を入力することです。

``` console
$ cat memory.x
/* STM32F303VCT6用のリンカスクリプト */
MEMORY
{
  /* 注記 1 K = 1 KiBi = 1024バイト */
  FLASH : ORIGIN = 0x08000000, LENGTH = 256K
  RAM : ORIGIN = 0x20000000, LENGTH = 40K
}
```

<!--
Make sure the `debug::exit()` call is commented out or removed, it is used
only for running in QEMU.
-->

`debug::exit()`の呼び出しが、コメントアウトされているか削除されていることを確認して下さい。
これは、QEMUで実行する時のみ、使用します。

``` rust
#[entry]
fn main() -> ! {
    hprintln!("Hello, world!").unwrap();

#     // exit QEMU
#     // NOTE do not run this on hardware; it can corrupt OpenOCD state
#     // debug::exit(debug::EXIT_SUCCESS);
    // QEMUを終了する
    // 注記、ハードウェア上で実行しないで下さい。OpenOCDの状態を破壊する可能性があります。
    // debug::exit(debug::EXIT_SUCCESS);

    loop {}
}
```

<!--
You can now cross compile programs using `cargo build`
and inspect the binaries using `cargo-binutils` as you did before. The
`cortex-m-rt` crate handles all the magic required to get your chip running,
as helpfully, pretty much all Cortex-M CPUs boot in the same fashion.
-->

これまでやってきた通り、`cargo build`でプログラムをクロスコンパイルし、
`cargo-binutils`でバイナリを調べることができます。
`cortex-m-rt`クレートは、チップを動作させるために必要な、全てのおまじないを処理します。
便利なことに、ほとんど全てのCortex-M CPUが同じ方法で起動します。

``` console
$ cargo build --example hello
```

<!-- ## Debugging -->

## デバッグ

<!--
Debugging will look a bit different. In fact, the first steps can look different
depending on the target device. In this section we'll show the steps required to
debug a program running on the STM32F3DISCOVERY. This is meant to serve as a
reference; for device specific information about debugging check out [the
Debugonomicon](https://github.com/rust-embedded/debugonomicon).
-->

デバッグ方法は少し違います。実際、最初のステップは、ターゲットデバイスによって異なります。
このセクションでは、STM32F3DISCOVERY上で実行しているプログラムをデバッグするために必要となる手順を説明します。
これは、参考の役目を果たします。デバイス固有のデバッグ情報は、
[the Debugonomicon](https://github.com/rust-embedded/debugonomicon)を参照して下さい。

<!--
As before we'll do remote debugging and the client will be a GDB process. This
time, however, the server will be OpenOCD.
-->

以前と同様に、リモートデバッグを行います。クライアントがGDBプロセスであることも同様です。
しかし、今回、サーバはOpenOCDになります。

<!--
As done during the [verify] section connect the discovery board to your laptop /
PC and check that the ST-LINK header is populated.
-->

[インストールの確認]セクションでやったように、ノートPCまたはPCをdiscoveryボードに接続し、
ST-LINKヘッダが設定されていることを確認して下さい。

<!-- [verify]: ../intro/install/verify.md -->

[インストールの確認]: ../intro/install/verify.md

<!--
On a terminal run `openocd` to connect to the ST-LINK on the discovery board.
Run this command from the root of the template; `openocd` will pick up the
`openocd.cfg` file which indicates which interface file and target file to use.
-->

discoveryボードのST-LINKに接続するために、端末で`openocd`を実行して下さい。
このコマンドは、テンプレートプロジェクトのルートディレクトリから実行して下さい。
`openocd`は、どのインタフェースファイルとターゲットファイルを使うか、が記述されている`openocd.cfg`ファイルを見つけます。

``` console
$ cat openocd.cfg
```


``` text
# STM32F3DISCOVERY開発ボード用のOpenOCD設定サンプル

# 持っているハードウェアのリビジョンに応じて、これらのインタフェースのうち、1つを選んで下さい。
# 常に、1つのインタフェースがコメントアウトされているべきです。

# Revision C (newer revision)
# リビジョンC （新しいリビジョン）
source [find interface/stlink-v2-1.cfg]

# リビジョンAとB（古いリビジョン）
# source [find interface/stlink-v2.cfg]

source [find target/stm32f3x.cfg]
```

<!--
> **NOTE** If you found out that you have an older revision of the discovery
> board during the [verify] section then you should modify the `openocd.cfg`
> file at this point to use `interface/stlink-v2.cfg`.
-->

> **注記** [インストールの確認]セクションで、古いバージョンのdiscoveryボードを持っていることが判明している場合、
> `interface/stlink-v2.cfg`を使うように`openocd.cfg`ファイルを修正する必要があります。

``` console
$ openocd
Open On-Chip Debugger 0.10.0
Licensed under GNU GPL v2
For bug reports, read
        http://openocd.org/doc/doxygen/bugs.html
Info : auto-selecting first available session transport "hla_swd". To override use 'transport select <transport>'.
adapter speed: 1000 kHz
adapter_nsrst_delay: 100
Info : The selected transport took over low-level target control. The results might differ compared to plain JTAG/SWD
none separate
Info : Unable to match requested speed 1000 kHz, using 950 kHz
Info : Unable to match requested speed 1000 kHz, using 950 kHz
Info : clock speed 950 kHz
Info : STLINK v2 JTAG v27 API v2 SWIM v15 VID 0x0483 PID 0x374B
Info : using stlink api v2
Info : Target voltage: 2.913879
Info : stm32f3x.cpu: hardware has 6 breakpoints, 4 watchpoints
```


<!-- On another terminal run GDB, also from the root of the template. -->


別の端末で、GDBを実行します。こちらも、テンプレートプロジェクトのルートディレクトから実行して下さい。

``` console
$ <gdb> -q target/thumbv7em-none-eabihf/debug/examples/hello
```

<!-- Next connect GDB to OpenOCD, which is waiting for a TCP connection on port 3333. -->

次に、TCP 3333ポートで接続待ちしているOpenOCDに、GDBを接続します。

``` console
(gdb) target remote :3333
Remote debugging using :3333
0x00000000 in ?? ()
```

<!--
Now proceed to *flash* (load) the program onto the microcontroller using the
`load` command.
-->

それでは、`load`コマンドを使って、マイクロコントローラにプログラムを*書き込んで*下さい。

``` console
(gdb) load
Loading section .vector_table, size 0x400 lma 0x8000000
Loading section .text, size 0x1e70 lma 0x8000400
Loading section .rodata, size 0x61c lma 0x8002270
Start address 0x800144e, load size 10380
Transfer rate: 17 KB/sec, 3460 bytes/write.
```

<!--
The program is now loaded. This program uses semihosting so before we do any
semihosting call we have to tell OpenOCD to enable semihosting. You can send
commands to OpenOCD using the `monitor` command.
-->

プログラムがロードされました。このプログラムはセミホスティングを使用します。そこで、
セミホスティングを呼び出して何かを行う前に、OpenOCDにセミホスティングを有効にするように、
指示する必要があります。

``` console
(gdb) monitor arm semihosting enable
semihosting is enabled
```

<!-- > You can see all the OpenOCD commands by invoking the `monitor help` command. -->

> `monitor help`コマンドを実行することで、全てのOpenOCDコマンドを見ることができます。

<!--
Like before we can skip all the way to `main` using a breakpoint and the
`continue` command.
-->

以前のように、ブレイクポイントと`continue`コマンドを使用することで、`main`までスキップすることができます。

``` console
(gdb) break main
Breakpoint 1 at 0x8000d18: file examples/hello.rs, line 15.

(gdb) continue
Continuing.
Note: automatically using hardware breakpoints for read-only addresses.

Breakpoint 1, main () at examples/hello.rs:15
15          let mut stdout = hio::hstdout().unwrap();
```

<!-- Advancing the program with `next` should produce the same results as before. -->

`next`でプログラムを先に進めると、以前と同じ結果になるはずです。

``` console
(gdb) next
16          writeln!(stdout, "Hello, world!").unwrap();

(gdb) next
19          debug::exit(debug::EXIT_SUCCESS);
```

<!--
At this point you should see "Hello, world!" printed on the OpenOCD console,
among other stuff.
-->

この時点で、OpenOCDコンソールに、他のものと入り混じって「Hello, world!」と表示されるはずです。

``` console
$ openocd
(..)
Info : halted: PC: 0x08000e6c
Hello, world!
Info : halted: PC: 0x08000d62
Info : halted: PC: 0x08000d64
Info : halted: PC: 0x08000d66
Info : halted: PC: 0x08000d6a
Info : halted: PC: 0x08000a0c
Info : halted: PC: 0x08000d70
Info : halted: PC: 0x08000d72
```

<!--
Issuing another `next` will make the processor execute `debug::exit`. This acts
as a breakpoint and halts the process:
-->

もう一度`next`を実行して、プロセッサに`debug::exit`を実行させます。
これはブレイクポイントとして動作し、プロセスを停止します。

``` console
(gdb) next

Program received signal SIGTRAP, Trace/breakpoint trap.
0x0800141a in __syscall ()
```

<!-- It also causes this to be printed to the OpenOCD console: -->

また、OpenOCDコンソールに次のものが表示されます。

``` console
$ openocd
(..)
Info : halted: PC: 0x08001188
semihosting: *** application exited ***
Warn : target not halted
Warn : target not halted
target halted due to breakpoint, current mode: Thread
xPSR: 0x21000000 pc: 0x08000d76 msp: 0x20009fc0, semihosting
```

<!--
However, the process running on the microcontroller has not terminated and you
can resume it using `continue` or a similar command.
-->

しかし、マイクロコントローラ上で動作しているプロセスは終了していないため、
`continue`もしくは同様のコマンドを使って、プログラムを再開することができます。

<!-- You can now exit GDB using the `quit` command. -->

ここで、`quit`コマンドを使うことで、GDBを終了できます。

``` console
(gdb) quit
```

<!--
Debugging now requires a few more steps so we have packed all those steps into a
single GDB script named `openocd.gdb`.
-->

デバッグにはもう少しステップが必要なので、これらのステップを`openocd.gdb`というGDBスクリプトにまとめました。

``` console
$ cat openocd.gdb
```

``` text
target remote :3333

# デマングルされたシンボルを表示します
set print asm-demangle on

# 未処理の例外、ハードフォールト、パニックを検出します
break DefaultHandler
break UserHardFault
break rust_begin_unwind

monitor arm semihosting enable

load

# プロセスを開始しますが、すぐにプロセッサを停止します
stepi
```

<!--
Now running `<gdb> -x openocd.gdb $program` will immediately connect GDB to
OpenOCD, enable semihosting, load the program and start the process.
-->

`<gdb> -x openocd.gdb $program`を実行することで、GDBはすぐにOpenOCDに接続し、
セミホスティングを有効化し、プログラムをロードした上で、プロセスを開始します。

<!--
Alternatively, you can turn `<gdb> -x openocd.gdb` into a custom runner to make
`cargo run` build a program *and* start a GDB session. This runner is included
in `.cargo/config` but it's commented out.
-->

別の方法として、`<gdb> -x openocd.gdb`をカスタムランナーにして、`cargo run`でプログラムをビルドし、
*さらに*GDBセッションを開始することもできます。このランナーは、`.cargo/config`に含まれていますが、
コメントアウトされています。

``` console
$ head -n10 .cargo/config
```

``` toml
[target.thumbv7m-none-eabi]
# ここのコメントアウトを外すと、`cargo run`はQEMUでプログラムを実行します
# runner = "qemu-system-arm -cpu cortex-m3 -machine lm3s6965evb -nographic -semihosting-config enable=on,target=native -kernel"

[target.'cfg(all(target_arch = "arm", target_os = "none"))']
# 3つの選択肢のうち、1つのコメントアウトを外すと、`cargo run`はGDBセッションを開始します。
# どの選択肢を使うか、は対象システムによって異なります。
runner = "arm-none-eabi-gdb -x openocd.gdb"
# runner = "gdb-multiarch -x openocd.gdb"
# runner = "gdb -x openocd.gdb"
```

``` console
$ cargo run --example hello
(..)
Loading section .vector_table, size 0x400 lma 0x8000000
Loading section .text, size 0x1e70 lma 0x8000400
Loading section .rodata, size 0x61c lma 0x8002270
Start address 0x800144e, load size 10380
Transfer rate: 17 KB/sec, 3460 bytes/write.
(gdb)
```
