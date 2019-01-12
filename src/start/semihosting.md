<!-- # Semihosting -->

# セミホスティング

<!--
Semihosting is a mechanism that lets embedded devices do I/O on the host and is
mainly used to log messages to the host console. Semihosting requires a debug
session and pretty much nothing else (no extra wires!) so it's super convenient
to use. The downside is that it's super slow: each write operation can take
several milliseconds depending on the hardware debugger (e.g. ST-Link) you use.
-->

セミホスティングは、組込みデバイスがホスト上でI/Oを行えるようにする仕組みで、主に、ホストのコンソールにログを出力するために使われます。
セミホスティングは、デバッグセッションが必要で、他には何も必要としません（追加の配線は不要です）。そのため、非常に便利です。
欠点は、非常に低速であることです。ハードウェアデバッガ（例えば、ST-Link）によっては、書き込み操作が数ミリ秒かかります。

<!--
The [`cortex-m-semihosting`] crate provides an API to do semihosting operations
on Cortex-M devices. The program below is the semihosting version of "Hello,
world!":
-->

[`cortex-m-semihosting`]クレートは、Cortex-Mデバイス上でセミホスティング操作をするためのAPIを提供します。
下のプログラムは、セミホスティングバージョンの「Hello, world!」です。

[`cortex-m-semihosting`]: https://crates.io/crates/cortex-m-semihosting

``` rust
#![no_main]
#![no_std]

extern crate panic_halt;

use cortex_m_rt::entry;
use cortex_m_semihosting::hprintln;

#[entry]
fn main() -> ! {
    hprintln!("Hello, world!").unwrap();

    loop {}
}
```

<!--
If you run this program on hardware you'll see the "Hello, world!" message
within the OpenOCD logs.
-->

このプログラムをハードウェア上で実行すると、OpenOCDログに、「Hello world!」のメッセージが表示されます。

``` console
$ openocd
(..)
Hello, world!
(..)
```

<!-- You do need to enable semihosting in OpenOCD from GDB first: -->

最初に、GDBからOpenOCDのセミホスティングを有効化する必要があります。

``` console
(gdb) monitor arm semihosting enable
semihosting is enabled
```

<!--
QEMU understands semihosting operations so the above program will also work with
`qemu-system-arm` without having to start a debug session. Note that you'll
need to pass the `-semihosting-config` flag to QEMU to enable semihosting
support; these flags are already included in the `.cargo/config` file of the
template.
-->

QEMUはセミホスティング操作を理解しているため、上のプログラムは、デバッグセッションを開始していない`qemu-system-arm`でも動作します。
セミホスティングサポートを有効化するため、QEMUに`-semihosting-config`フラグを渡す必要があることに注意して下さい。
これらのフラグは、テンプレートの`.cargo/config`ファイルに既に含まれています。

``` console
$ # このプログラムは端末をブロックします
$ cargo run
     Running `qemu-system-arm (..)
Hello, world!
```

<!--
There's also an `exit` semihosting operation that can be used to terminate the
QEMU process. Important: do **not** use `debug::exit` on hardware; this function
can corrupt your OpenOCD session and you will not be able to debug more programs
until you restart it.
-->

`exit`セミホスティング操作もあり、QEMUプロセスを終了するために使われます。
重要：ハードウェア上で`debug::exit`を**使用しない**で下さい。この関数は、OpenOCDセッションを破壊する可能性があり、
OpenOCDを再起動しない限り、それ以上のプログラムのデバッグができなくなります。

``` rust
#![no_main]
#![no_std]

extern crate panic_halt;

use cortex_m_rt::entry;
use cortex_m_semihosting::debug;

#[entry]
fn main() -> ! {
    let roses = "blue";

    if roses == "red" {
        debug::exit(debug::EXIT_SUCCESS);
    } else {
        debug::exit(debug::EXIT_FAILURE);
    }

    loop {}
}
```

``` console
$ cargo run
     Running `qemu-system-arm (..)

$ echo $?
1
```

<!--
One last tip: you can set the panicking behavior to `exit(EXIT_FAILURE)`. This
will let you write `no_std` run-pass tests that you can run on QEMU.
-->

最後のヒント：パニック時の挙動を、`exit(EXIT_FAILURE)`に設定することができます。
これで、QEMU上で実行できる`no_std`ランパステストを書くことができます。

<!--
For convenience, the `panic-semihosting` crate has an "exit" feature that when
enabled invokes `exit(EXIT_FAILURE)` after logging the panic message to the host
stderr.
-->

利便性のために、`panic-semihosting`クレートは、「exit」フィーチャを持っています。
このフィーチャが有効化されていると、ホストの標準エラーにパニックメッセージをログ出力した後、`exit(EXIT_FAILURE)`を呼び出します。

``` rust
#![no_main]
#![no_std]

extern crate panic_semihosting; // features = ["exit"]

use cortex_m_rt::entry;
use cortex_m_semihosting::debug;

#[entry]
fn main() -> ! {
    let roses = "blue";

    assert_eq!(roses, "red");

    loop {}
}
```

``` console
$ cargo run
     Running `qemu-system-arm (..)
panicked at 'assertion failed: `(left == right)`
  left: `"blue"`,
 right: `"red"`', examples/hello.rs:15:5

$ echo $?
1
```
