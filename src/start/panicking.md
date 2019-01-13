<!-- # Panicking -->

# パニック

<!--
Panicking is a core part of the Rust language. Built-in operations like indexing
are runtime checked for memory safety. When out of bounds indexing is attempted
this results in a panic.
-->

パニックはRustのコア部分です。インデックス操作のような言語組込みの操作は、メモリ安全性をランタイム時に検査されます。
範囲外のインデックスにアクセスしようとすると、パニックが発生します。

<!--
In the standard library panicking has a defined behavior: it unwinds the stack
of the panicking thread, unless the user opted for aborting the program on
panics.
-->

標準ライブラリでは、パニックは定義された動作です。ユーザがパニック発生時にプログラムをアボートする選択をしない限り、
パニックを起こしたスレッドのスタックを巻き戻します。

<!--
In non-standard programs, however, the panicking behavior is left undefined. A
behavior can be chosen by declaring a `#[panic_handler]` function. This function
must appear exactly *once* in the dependency graph of a program, and must have
the following signature: `fn(&PanicInfo) -> !`, where [`PanicInfo`] is a struct
containing information about the location of the panic.
-->

しかし、非標準のプログラムでは、パニック時の挙動は、未定義のままです。`#[panic_handler]`関数を宣言することにより、
挙動を選択することができます。この関数は、プログラムの依存関係グラフに、**1回だけ**現れる必要があります。
そして、 `fn(&PanicInfo) -> !`のシグネチャを持つ必要があります。
ここで、[`PanicInfo`]は、パニックした位置情報を含む構造体です。

[`PanicInfo`]: https://doc.rust-lang.org/core/panic/struct.PanicInfo.html

<!--
Given that embedded systems range from user facing to safety critical (cannot
crash) there's no one size fits all panicking behavior but there are plenty of
commonly used behaviors. These common behaviors have been packaged into crates
that define the `#[panic_handler]` function. Some examples include:
-->

組込みシステムは、ユーザとやり取りするものから、安全性が重要な（クラッシュできない）ものまであります。
そのため、全てのパニック時動作に対応できる唯一のものはありませんが、よく利用される挙動がたくさんあります。
これらの一般的な挙動が、`#[panic_handler]`関数を定義するクレートにまとめられています。
いくつか、例を挙げます。

<!--
- [`panic-abort`]. A panic causes the abort instruction to be executed.
- [`panic-halt`]. A panic causes the program, or the current thread, to halt by
  entering an infinite loop.
- [`panic-itm`]. The panicking message is logged using the ITM, an ARM Cortex-M
  specific peripheral.
- [`panic-semihosting`]. The panicking message is logged to the host using the
  semihosting technique.
-->

- [`panic-abort`]。パニックが発生すると、アボート命令を実行します。
- [`panic-halt`]。パニックが発生すると、プログラム、または、現在のスレッドは、無限ループに入ることで停止します。
- [`panic-itm`]。パニック発生時のメッセージは、ARM Cortex-M固有のペリフェラルであるITMを使ってログ出力されます。
- [`panic-semihosting`]。パニック発生時のメッセージは、セミホスティングを使ってログ出力されます。

[`panic-abort`]: https://crates.io/crates/panic-abort
[`panic-halt`]: https://crates.io/crates/panic-halt
[`panic-itm`]: https://crates.io/crates/panic-itm
[`panic-semihosting`]: https://crates.io/crates/panic-semihosting

<!--
You may be able to find even more crates searching for the [`panic-handler`]
keyword on crates.io.
-->

crates.ioで[`panic-handler`]をキーワードに検索することで、さらにクレートを見つけることができます。

[`panic-handler`]: https://crates.io/keywords/panic-handler

<!--
A program can pick one of these behaviors simply by linking to the corresponding
crate. The fact that the panicking behavior is expressed in the source of
an application as a single line of code is not only useful as documentation but
can also be used to change the panicking behavior according to the compilation
profile. For example:
-->

プログラムは、対応するクレートとリンクすることで、これらの挙動の中から1つを選びます。
パニック時の挙動がアプリケーションソースコードの中で単一行で表現されていることは、ドキュメントとして有用なだけでなく、
パニック時の挙動をコンパイル時のプロファイルで変更にする時にも利用できます。
例えば

``` rust,ignore
#![no_main]
#![no_std]

# // dev profile: easier to debug panics; can put a breakpoint on `rust_begin_unwind`
// 開発プロファイル：パニックのデバッグを容易にします。`rust_begin_unwind`にブレイクポイントを置くことを可能にします。
#[cfg(debug_assertions)]
extern crate panic_halt;

# // release profile: minimize the binary size of the application
// リリースプロファイル：アプリケーションのバイナリサイズを最小化します。
#[cfg(not(debug_assertions))]
extern crate panic_abort;

// ..
```

<!--
In this example the crate links to the `panic-halt` crate when built with the
dev profile (`cargo build`), but links to the `panic-abort` crate when built
with the release profile (`cargo build --release`).
-->

この例では、開発プロファイルでビルド（`cargo build`）した時は、`panic-halt`クレートとリンクします。
しかし、リリースプロファイルでビルド（`cargo build --release`）した時は、`panic-abort`クレートとリンクします。

<!-- ## An example -->

## 例

<!--
Here's an example that tries to index an array beyond its length. The operation
results in a panic.
-->

配列の長さを超えてアクセスしようとする例を示します。この操作はパニックを引き起こします。

``` rust
#![no_main]
#![no_std]

extern crate panic_semihosting;

use cortex_m_rt::entry;

#[entry]
fn main() -> ! {
    let xs = [0, 1, 2];
    let i = xs.len() + 1;
# //    let _y = xs[i]; // out of bounds access
    let _y = xs[i]; // 範囲外アクセス

    loop {}
}
```

<!--
This example chose the `panic-semihosting` behavior which prints the panic
message to the host console using semihosting.
-->

この例では、`panic-semihosting`の挙動を選択しており、パニックメッセージは、
セミホスティングを使ってホストコンソールに出力されます。

``` console
$ cargo run
     Running `qemu-system-arm -cpu cortex-m3 -machine lm3s6965evb (..)
panicked at 'index out of bounds: the len is 3 but the index is 4', src/main.rs:12:13
```

<!--
You can try changing the behavior to `panic-halt` and confirm that no message is
printed in that case.
-->

挙動を`panic-halt`に変更し、その場合にメッセージが出力されないことを確認することができます。