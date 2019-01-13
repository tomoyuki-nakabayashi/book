<!-- # Static Guarantees -->

# 静的な保証

<!--
It's Rust's type system what prevents data races at compile time (see [`Send`]
and [`Sync`] traits). The type system can also be used to check other properties
at compile time; reducing the need for runtime checks in some cases.
-->

コンパイル時にデータ競合を防ぐのは、Rustの型システムです（[`Send`]と[`Sync`]トレイトを参照）。
この型システムは、コンパイル時に他のプロパティをチェックするためにも使用できます。
その結果、実行時チェックの必要性を減らせる場合があります。

[`Send`]: https://doc.rust-lang.org/core/marker/trait.Send.html
[`Sync`]: https://doc.rust-lang.org/core/marker/trait.Sync.html

<!--
When applied to embedded programs these *static checks* can be used, for
example, to enforce that configuration of I/O interfaces is done properly. For
instance, one can design an API where it is only possible to initialize a serial
interface by first configuring the pins that will be used by the interface.
-->

組込みプログラムに適用する場合、これらの*静的なチェック*は、例えば、入出力インタフェースが正しく設定されていることを強制することができます。
例えば、使用されるピンを最初に設定することによってのみ、シリアルインタフェースを初期化できるようなAPI設計が可能です。

<!--
One can also statically check that operations, like setting a pin low, can only
be performed on correctly configured peripherals. For example, trying to change
the output state of a pin configured in floating input mode would raise a
compile error.
-->

正しく設定されたペリフェラルでのみ、ピンをローレベルにするというような操作ができることを、
静的にチェックすることも可能です。例えば、フローティング入力モードに設定されたピンの出力状態を変更しようとすると、
コンパイルエラーが発生します。

> 訳注：フローティング入力モードは、ピンをハイインピーダンスの入力モードにしていることを意味しています。

<!--
And, as seen in the previous chapter, the concept of ownership can be applied
to peripherals to ensure that only certain parts of a program can modify a
peripheral. This *access control* makes software easier to reason about
compared to the alternative of treating peripherals as global mutable state.
-->

<!-- reasonは推論する、という意味で使われているようですが、ソフトウェアの推論、では意味が捉えにくいため、解析、と訳しました。 -->

以前の章で見た通り、所有権の概念はペリフェラルにも適用できます。所有権は、プログラムの特定部分のみがペリフェラルを変更することを保証します。
この*アクセスコントロール*は、ペリフェラルをグローバルでミュータブルな状態として扱う代替案と比較して、
ソフトウェアの解析をより簡単にします。