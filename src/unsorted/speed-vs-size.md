<!-- # Optimizations: the speed size tradeoff -->

# 最適化：速度とサイズのトレードオフ

<!--
Everyone wants their program to be super fast and super small but it's usually
not possible to have maximize both characteristics. This section discusses the
different optimization levels that `rustc` provides and how the affect the
execution time and binary size of a program.
-->

誰もがプログラムを超高速で超小さくしたいと願いますが、両方の特性を最大化することはできません。
このセクションは、`rustc`が提供する異なる最適化レベルについて、どのようにプログラムの実行時間とバイナリサイズに影響するかを説明します。

<!-- ## No optimizations -->

## 最適化なし

<!--
This is the default. When you call `cargo build` you use the development (AKA
`dev`) profile. This profile is optimized for debugging so it enables debug
information and does *not* enable any optimizations, i.e. it uses `-C opt-level
= 0`.
-->

これはデフォルトです。`cargo build`を実行する場合、開発（別名`dev`）プロファイルを使います。
このプロファイルはデバッグに最適化されています。そのため、デバッグ情報が有効化されており、最適化は一切有効化されて*いません*。
つまり、`-C opt-level = 0`を使用します。

<!--
At least for bare metal development, debuginfo is zero cost in the sense that it
won't occupy space in Flash / ROM so we actually recommend that you enable
debuginfo in the release profile -- it is disabled by default. That will let you
use breakpoints when debugging release builds.
-->

少なくともベアメタルの開発では、デバッグ情報はある意味ゼロコストです。
デバッグ情報は、Flash/ROMの容量を使いません。そのため、リリースプロファイルで、デフォルトで無効化されているデバッグ情報を有効化することをお勧めします。
これにより、リリースビルドをデバッグする時、ブレイクポイントを使うことができます。

``` toml
[profile.release]
# シンボルは素晴らしく、Flashのサイズを増やしません
debug = true
```

<!--
No optimizations is great for debugging because stepping through the code feels
like you are executing the program statement by statement, plus you can `print`
stack variables and function arguments in GDB. When the code is optimized trying
to print variables results in `$0 = <value optimized out>` being printed.
-->

最適化しないことはデバッグでは重要です。コードをステップ実行する時、プログラムをステートメントごとに実行しているように感じられるからです。
さらに、スタックの変数や関数の引数をGDBで`print`することができます。
コードが最適化されると、変数を表示しようとしても、`$0 = <value optimized out>`と表示されます。

<!--
The biggest downside of the `dev` profile is that the resulting binary will be
huge and slow. The size is usually more of a problem because unoptimized
binaries can occupy dozens of KiB of Flash, which your target device may not
have -- the result: your unoptimized binary doesn't fit in your device!
-->

`dev`プロファイルの最大の欠点は、バイナリが大きく、遅いことです。
バイナリサイズは通常、より問題となります。最適化されていないバイナリは数十KiBもFlashを専有するからです。
ターゲットデバイスは、数十KiBものFlashを持っていないかもしれず、最適化されていないバイナリは、デバイス内に納まりません。

<!-- Can we have smaller debugger friendly binaries? Yes, there's a trick. -->

デバッガで扱いやすい、小さなバイナリを作ることができるのでしょうか？できます。良いやり方があります。

<!-- ### Optimizing dependencies -->

### 依存関係の最適化

<!--
> **WARNING** This section uses an unstable feature and it was last tested on
> 2018-09-18. Things may have changed since then!
-->

> **注意** このセクションは、2018-9-18に最後にテストされた安定化していないフィーチャを使います。
> それ以降、状況が変わっているかもしれません！

<!--
On nightly, there's a Cargo feature named [`profile-overrides`] that lets you
override the optimization level of dependencies. You can use that feature to
optimize all dependencies for size while keeping the top crate unoptimized and
debugger friendly.
-->

nightlyでは、[`profile-overrides`]と呼ばれるCargoフィーチャがあります。これは、依存関係の最適化レベルをオーバーライドします。
このフィーチャを使って、全ての依存クレートのサイズを最適化しながら、トップクレートを最適化しないでデバッガで扱いやすくすることができます。。

[`profile-overrides`]: https://doc.rust-lang.org/nightly/cargo/reference/unstable.html#profile-overrides

<!-- Here's an example: -->

例を示します。

``` toml
# Cargo.toml
cargo-features = ["profile-overrides"] # +

[package]
name = "app"
# ..

[profile.dev.overrides."*"] # +
opt-level = "z" # +
```

<!-- Without the override: -->

オーバーライドなしでは、次の通りです。

``` console
$ cargo size --bin app -- -A
app  :
section               size        addr
.vector_table         1024   0x8000000
.text                 9060   0x8000400
.rodata               1708   0x8002780
.data                    0  0x20000000
.bss                     4  0x20000000
```

<!-- With the override: -->

オーバーライドをすると、以下のようになります。

``` console
$ cargo size --bin app -- -A
app  :
section               size        addr
.vector_table         1024   0x8000000
.text                 3490   0x8000400
.rodata               1100   0x80011c0
.data                    0  0x20000000
.bss                     4  0x20000000
```

<!--
That's a 6 KiB reduction in Flash usage without any loss in the debuggability of
the top crate. If you step into a dependency then you'll start seeing those
`<value optimized out>` messages again but it's usually the case that you want
to debug the top crate and not the dependencies. And if you *do* need to debug a
dependency then you can use the `profile-overrides` feature to exclude a
particular dependency from being optimized. See example below:
-->

トップクレートのデバッグ性を失うことなしに、Flash使用量を6KiB減らしています。
依存クレートの中に足を踏み入れると、`<value optimized out>`のメッセージを目にするでしょう。
しかし、依存クレートではなく、トップクレートをデバッグしたい場合がほとんどでしょう。
依存クレートをデバッグする必要が*ある*場合、特定の依存クレートを最適化から除外するために、`profile-overrides`フィーチャを使えます。
例えば、以下のようになります。

``` toml
# ..

# `cortex-m-rt`クレートは最適化しません
[profile.dev.overrides.cortex-m-rt] # +
opt-level = 0 # +

# しかし、他の依存クレートは最適化します
[profile.dev.overrides."*"]
codegen-units = 1 # better optimizations
opt-level = "z"
```

<!-- Now the top crate and `cortex-m-rt` are debugger friendly! -->

これで、トップクレートと`cortex-m-rt`クレートはデバッガで扱いやすくなります！

<!-- ## Optimize for speed -->

## 速度の最適化

<!--
As of 2018-09-18 `rustc` supports three "optimize for speed" levels: `opt-level
= 1`, `2` and `3`. When you run `cargo build --release` you are using the release
profile which defaults to `opt-level = 3`.
-->

2018-09-18の`rustc`は、3つの「速度最適化」を提供しています。`opt-level = 1`、`2`と`3`です。
`cargo build --release`を実行した時、デフォルトでは`opt-level = 3`のリリースプロファイルを使います。

<!--
Both `opt-level = 2` and `3` optimize for speed at the expense of binary size,
but level `3` does more vectorization and inlining than level `2`. In
particular, you'll see that at `opt-level` equal or greater than `2` LLVM will
unroll loops. Loop unrolling has a rather high cost in terms of Flash / ROM
(e.g. from 26 bytes to 194 for a zero this array loop) but can also halve the
execution time given the right conditions (e.g. number of iterations is big
enough).
-->

`opt-level = 2`と`3`は、バイナリサイズを犠牲にして、速度を最適化します。レベル`3`はレベル`2`より、ベクトル化とインライン化を行います。
特に、`opt-level`が`2`以上の場合、LLVMがループを展開するのがわかるでしょう。
ループ展開は、Flash/ROMの観点からはよりコストが高いです（例えば、配列のループをゼロにする場合、26バイトから194バイトまで増加します）。
しかし、適切な条件では、実行時間が半分になります（例えば、イテレーションの回数が十分大きい場合）。

<!--
Currently there's no way to disable loop unrolling in `opt-level = 2` and `3` so
if you can't afford its cost you should optimize your program for size.
-->

現在、`opt-level = 2`と`3`でループ展開を無効化する方法はありません。
ループ展開のコストを払うことができない場合、プログラムサイズの最適化をするべきです。

<!-- ## Optimize for size -->

## サイズの最適化

<!--
As of 2018-09-18 `rustc` supports two "optimize for size" levels: `opt-level =
"s"` and `"z"`. These names were inherited from clang / LLVM and are not too
descriptive but `"z"` is meant to give the idea that it produces smaller
binaries than `"s"`.
-->

2018-09-18の`rustc`は、2つの`サイズ最適化`を提供しています。`opt-level = "s"`と`"z"`です。
これらの名前は、clang / LLVMから受け継いでおり、意味がわかりにくいです。
`"z"`は、`"s"`より小さなバイナリを作る意図を意味します。

<!--
If you want your release binaries to be optimized for size then change the
`profile.release.opt-level` setting in `Cargo.toml` as shown below.
-->

リリースバイナリのサイズを最適化したい場合、`Cargo.toml`の`profile.release.opt-level`設定を下記の通り変更します。

``` toml
[profile.release]
# または"z"
opt-level = "s"
```

<!--
These two optimization levels greatly reduce LLVM's inline threshold, a metric
used to decide whether to inline a function or not. One of Rust principles are
zero cost abstractions; these abstractions tend to use a lot of newtypes and
small functions to hold invariants (e.g. functions that borrow an inner value
like `deref`, `as_ref`) so a low inline threshold can make LLVM miss
optimization opportunities (e.g. eliminate dead branches, inline calls to
closures).
-->

これらの2つの最適化レベルは、LLVMのインラインしきい値を大幅に減らします。
インラインしきい値は、関数をインライン化するか否かを決めるために使われる基準値です。
Rustの原則の1つは、ゼロコスト抽象化です。
これらの抽象化は、不変条件を保持するため、多くの新しい型と小さな関数を使う傾向にあります
（例えば、`deref`や`as_ref`のような内部の値を借用するための関数）。
そのため、インラインしきい値を低くすると、LLVMが最適化の機会を失います
（例えば、不要な分岐を削除したり、クロージャをインライン呼び出しにする、など）。

<!--
When optimizing for size you may want to try increasing the inline threshold to
see if that has any effect on the binary size. The recommended way to change the
inline threshold is to append the `-C inline-threshold` flag to the other
rustflags in `.cargo/config`.
-->

サイズの最適化を行っている時、バイナリサイズに影響があるかどうかを見るために、インラインしきい値を増やしたいかもしれません。
インラインしきい値を変更するお勧めの方法は、`.cargo/config`内のrustflagsに`-C inline-threshold`フラグを追加することです。

``` toml
# .cargo/config
# cortex-m-quickstartテンプレートを使っていることを想定しています
[target.'cfg(all(target_arch = "arm", target_os = "none"))']
rustflags = [
  # ..
  "-C", "inline-threshold=123", # +
]
```

<!--
What value to use? [As of 1.29.0 these are the inline thresholds that the
different optimization levels use][inline-threshold]:
-->

この値は何に使われるのでしょうか？
[1.29.0では、下記の値は、異なる最適化レベルで使われるインラインしきい値です][インラインしきい値]

<!-- [inline-threshold]: https://github.com/rust-lang/rust/blob/1.29.0/src/librustc_codegen_llvm/back/write.rs#L2105-L2122 -->

[インラインしきい値]: https://github.com/rust-lang/rust/blob/1.29.0/src/librustc_codegen_llvm/back/write.rs#L2105-L2122

<!--
- `opt-level = 3` uses 275
- `opt-level = 2` uses 225
- `opt-level = "s"` uses 75
- `opt-level = "z"` uses 25
-->

- `opt-level = 3`は275を使います
- `opt-level = 2`は225を使います
- `opt-level = "s"`は75を使います
- `opt-level = "z"`は25を使います

<!-- You should try `225` and `275` when optimizing for size. -->

サイズの最適化をするときは、`225`と`275`を試してみるべきです。