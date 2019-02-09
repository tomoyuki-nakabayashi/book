<!-- # Exceptions -->

# 例外

<!--
Exceptions, and interrupts, are a hardware mechanism by which the processor
handles asynchronous events and fatal errors (e.g. executing an invalid
instruction). Exceptions imply preemption and involve exception handlers,
subroutines executed in response to the signal that triggered the event.
-->

例外と割り込みは、プロセッサが非同期イベントと致命的なエラー（例えば、不正な命令の実行）を扱うためのハードウェアの仕組みです。
例外はプリエンプションを意味し、例外ハンドラを呼び出します。例外ハンドラは、イベントを引き起こした信号に応答して実行されるサブルーチンです。

<!--
The `cortex-m-rt` crate provides an [`exception`] attribute to declare exception
handlers.
-->

`cortex-m-rt`クレートは、例外ハンドラを宣言するために、[`exception`]アトリビュートを提供しています。

[`exception`]: https://docs.rs/cortex-m-rt-macros/latest/cortex_m_rt_macros/attr.exception.html

``` rust,ignore
# // Exception handler for the SysTick (System Timer) exception
// SysTick（システムタイマ）例外のための例外ハンドラ
#[exception]
fn SysTick() {
    // ..
}
```

<!--
Other than the `exception` attribute exception handlers look like plain
functions but there's one more difference: `exception` handlers can *not* be
called by software. Following the previous example, the statement `SysTick();`
would result in a compilation error.
-->

`exception`属性の他は、例外ハンドラは普通の関数のように見えます。しかし、もう1つ違いがあります。
`exception`ハンドラはソフトウェアから呼び出すことが*できません*。前述の例では、`SysTick();`というステートメントは、
コンパイルエラーになります。

<!--
This behavior is pretty much intended and it's required to provide a feature:
`static mut` variables declared *inside* `exception` handlers are *safe* to use.
-->

この動作は、非常に意図的なものです。
これは`exception`ハンドラ*内*で宣言された`static mut`変数の利用を*安全*にする、という機能を提供するためのものです。

``` rust,ignore
#[exception]
fn SysTick() {
    static mut COUNT: u32 = 0;

#     // `COUNT` has type `&mut u32` and it's safe to use
    // `COUNT`は`&mut u32`の型をもっており、その利用は安全です
    *COUNT += 1;
}
```

<!--
As you may know, using `static mut` variables in a function makes it
*non-reentrant*. It's undefined behavior to call a non-reentrant function,
directly or indirectly, from more than one exception / interrupt handler or from
`main` and one or more exception / interrupt handlers.
-->

ご存知かもしれませんが、`static mut`変数を関数内で使うことは、その関数を*再入不可能*にします。
直接的または間接的に、複数の例外・割り込みハンドラから、もしくは、`main`と1つ以上の例外・割り込みハンドラから、
再進入不可能な関数を呼び出すことは、未定義動作です。

<!--
Safe Rust must never result in undefined behavior so non-reentrant functions
must be marked as `unsafe`. Yet I just told that `exception` handlers can safely
use `static mut` variables. How is this possible? This is possible because
`exception` handlers can *not* be called by software thus reentrancy is not
possible.
-->

安全なRustは、決して未定義動作になりません。そのため、再入不可能な関数は、`unsafe`とマークされなければなりません。
それでも、`exception`ハンドラは`static mut`な変数を安全に使える、と述べました。これが可能なのは、どうしてでしょうか。
`exception`ハンドラはソフトウェアから呼び出すことが*できない*ため、再入する可能性はありません。だから、安全に使えるのです。

<!-- ## A complete example -->

## 完全な例

<!--
Here's an example that uses the system timer to raise a `SysTick` exception
roughly every second. The `SysTick` exception handler keeps track of how many
times it has been called in the `COUNT` variable and then prints the value of
`COUNT` to the host console using semihosting.
-->

`SysTick`例外を大体1秒毎に発生させるシステムタイマの例を使います。
`SysTick`例外ハンドラは、呼び出された回数を`COUNT`変数に記録し、
セミホスティングを使ってホストコンソールに`COUNT`の値を出力します。

<!--
> **NOTE**: You can run this example on any Cortex-M device; you can also run it
> on QEMU
-->

> **注記**：この例は、どのCortex-Mデバイスでも実行できます。QEMU上でも実行可能です。

``` rust
#![deny(unsafe_code)]
#![no_main]
#![no_std]

extern crate panic_halt;

use core::fmt::Write;

use cortex_m::peripheral::syst::SystClkSource;
use cortex_m_rt::{entry, exception};
use cortex_m_semihosting::{
    debug,
    hio::{self, HStdout},
};

#[entry]
fn main() -> ! {
    let p = cortex_m::Peripherals::take().unwrap();
    let mut syst = p.SYST;

#     // configures the system timer to trigger a SysTick exception every second
    // 毎秒SysTick例外を起こすためのシステムタイマを設定します
    syst.set_clock_source(SystClkSource::Core);
#     // this is configured for the LM3S6965 which has a default CPU clock of 12 MHz
    // デフォルトのCPUクロックが12MHzのLM3S6965向けの設定です
    syst.set_reload(12_000_000);
    syst.enable_counter();
    syst.enable_interrupt();

    loop {}
}

#[exception]
fn SysTick() {
    static mut COUNT: u32 = 0;
    static mut STDOUT: Option<HStdout> = None;

    *COUNT += 1;

#     // Lazy initialization
    // 遅延初期化
    if STDOUT.is_none() {
        *STDOUT = hio::hstdout().ok();
    }

    if let Some(hstdout) = STDOUT.as_mut() {
        write!(hstdout, "{}", *COUNT).ok();
    }

#     // IMPORTANT omit this `if` block if running on real hardware or your
#     // debugger will end in an inconsistent state
    // 重要。実際のハードウェアで実行するときは`if`ブロックを削除して下さい。そうでなければ、
    // デバッガが不整合な状態に陥るでしょう。
    if *COUNT == 9 {
#         // This will terminate the QEMU process
        // QEMUプロセスを終了します
        debug::exit(debug::EXIT_SUCCESS);
    }
}
```

``` console
$ tail -n5 Cargo.toml
```

``` toml
[dependencies]
cortex-m = "0.5.7"
cortex-m-rt = "0.6.3"
panic-halt = "0.2.0"
cortex-m-semihosting = "0.3.1"
```

``` console
$ cargo run --release
     Running `qemu-system-arm -cpu cortex-m3 -machine lm3s6965evb (..)
123456789
```

<!--
If you run this on the Discovery board you'll see the output on the OpenOCD
console. Also, the program will *not* stop when the count reaches 9.
-->

Discoveryボードでこのコードを実行すると、OpenOCDコンソールに出力を確認できるでしょう。
プログラムは、カウントが9に到達しても停止*しません*。

<!-- ## The default exception handler -->

## デフォルト例外ハンドラ

<!--
What the `exception` attribute actually does is *override* the default exception
handler for a specific exception. If you don't override the handler for a
particular exception it will be handled by the `DefaultHandler` function, which
defaults to:
-->

`exception`アトリビュートが実際に行っていることは、特定の例外を処理するデフォルト例外ハンドラの*オーバーライド*です。
特定の例外について、ハンドラをオーバーライドしない場合、`DefaultHandler`関数がその例外を処理します。
DefaultHandler関数は下記の通りです。

``` rust,ignore
fn DefaultHandler() {
    loop {}
}
```

<!--
This function is provided by the `cortex-m-rt` crate and marked as
`#[no_mangle]` so you can put a breakpoint on "DefaultHandler" and catch
*unhandled* exceptions.
-->

この関数は、`cortex-m-rt`クレートによって提供されており、`#[no_mangle]`とマークされています。
そのため、「DefaultHandler」にブレイクポイントを設定することができ、*未処理の*例外を捕捉することができます。

<!-- It's possible to override this `DefaultHandler` using the `exception` attribute: -->

`exception`アトリビュートを使うことで、`DefaultHandler`をオーバーライドできます。

``` rust,ignore
#[exception]
fn DefaultHandler(irqn: i16) {
#     // custom default handler
    // カスタムデフォルトハンドラ
}
```

<!--
The `irqn` argument indicates which exception is being serviced. A negative
value indicates that a Cortex-M exception is being serviced; and zero or a
positive value indicate that a device specific exception, AKA interrupt, is
being serviced.
-->

`irqn`引数は、どの例外が処理されているかを示します。負の値は、Cortex-Mの例外が処理されていることを意味します。
ゼロまたは正の値は、デバイス固有の例外、すなわち、割り込みが処理されていること、を示しています。

<!-- ## The hard fault handler -->

## ハードフォールトハンドラ

<!--
The `HardFault` exception is a bit special. This exception is fired when the
program enters an invalid state so its handler can *not* return as that could
result in undefined behavior. Also, the runtime crate does a bit of work before
the user defined `HardFault` handler is invoked to improve debuggability.
-->

`HardFault`例外は、少し特別です。この例外は、プログラムが不正な状態になった場合に発生します。
そのため、このハンドラはリターンすることができず、未定義動作を引き起こす可能性があります。
ランタイムクレートは、デバッグ性を向上するために、ユーザ定義の`HardFault`ハンドラが呼び出される前に、少し仕事をします。

<!--
The result is that the `HardFault` handler must have the following signature:
`fn(&ExceptionFrame) -> !`. The argument of the handler is a pointer to
registers that were pushed into the stack by the exception. These registers are
a snapshot of the processor state at the moment the exception was triggered and
are useful to diagnose a hard fault.
-->

その結果、`HardFault`ハンドラは、`fn(&ExceptionFrame) -> !`のシグネチャを持つ必要があります。
ハンドラの引数は、例外によってスタックにプッシュされたレジスタへのポインタです。
これらのレジスタは、例外が発生した瞬間のプロセッサステートのスナップショットで、ハードフォールトの原因を突き止めるのに便利です。

<!--
Here's an example that performs an illegal operation: a read to a nonexistent
memory location.
-->

不正な操作を行う例を示します。存在しないメモリ位置への読み込みです。

<!--
> **NOTE**: This program won't work, i.e. it won't crash, on QEMU because
> `qemu-system-arm -machine lm3s6965evb` doesn't check memory loads and will
> happily return `0 `on reads to invalid memory.
-->

> **注記**：このプログラムは、QEMU上ではうまく動きません。つまり、クラッシュしません。
> `qemu-system-arm -machine lm3s6965evb`はメモリの読み込みをチェックしないため、
> 無効なメモリを読み込むと、幸いにも、`0`を返します。

``` rust
#![no_main]
#![no_std]

extern crate panic_halt;

use core::fmt::Write;
use core::ptr;

use cortex_m_rt::{entry, exception, ExceptionFrame};
use cortex_m_semihosting::hio;

#[entry]
fn main() -> ! {
#     // read a nonexistent memory location
    // 存在しないメモリ位置を読み込みます
    unsafe {
        ptr::read_volatile(0x3FFF_FFFE as *const u32);
    }

    loop {}
}

#[exception]
fn HardFault(ef: &ExceptionFrame) -> ! {
    if let Ok(mut hstdout) = hio::hstdout() {
        writeln!(hstdout, "{:#?}", ef).ok();
    }

    loop {}
}
```

<!--
The `HardFault` handler prints the `ExceptionFrame` value. If you run this
you'll see something like this on the OpenOCD console.
-->

`HardFault`ハンドラは、`ExceptionFrame`の値を表示します。実行すると、
OpenOCDコンソールに次のような表示が見えるでしょう。

``` console
$ openocd
(..)
ExceptionFrame {
    r0: 0x3ffffffe,
    r1: 0x00f00000,
    r2: 0x20000000,
    r3: 0x00000000,
    r12: 0x00000000,
    lr: 0x080008f7,
    pc: 0x0800094a,
    xpsr: 0x61000000
}
```

<!--
The `pc` value is the value of the Program Counter at the time of the exception
and it points to the instruction that triggered the exception.
-->

`pc`の値は、例外発生時のプログラムカウンタの値で、例外を引き起こした命令を指しています。

<!-- If you look at the disassembly of the program: -->

プログラムのディスアセンブル結果を見ます。

``` console
$ cargo objdump --bin app --release -- -d -no-show-raw-insn -print-imm-hex
(..)
ResetTrampoline:
 8000942:       movw    r0, #0xfffe
 8000946:       movt    r0, #0x3fff
 800094a:       ldr     r0, [r0]
 800094c:       b       #-0x4 <ResetTrampoline+0xa>
```

<!--
You'll see that a load operation (`ldr r0, [r0]` ) caused the exception and that
the value of the register `r0` was `0x3fff_fffe` at that time. This value
matches the `r0` field of `ExceptionFrame`.
-->

ロード命令（`ldr r0, [r0]`）が例外を発生させたことがわかります。そして、この時の`r0`レジスタの値は、
`0x3fff_fffe`です。この値は、`ExceptionFrame`の`r0`フィールドと一致します。