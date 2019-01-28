<!-- # Tips for embedded C developers -->

# 組込みC開発者へのヒント

<!--
This chapter collects a variety of tips that might be useful to experienced
embedded C developers looking to start writing Rust. It will especially
highlight how things you might already be used to in C are different in Rust.
-->

この章では、組込みC開発の経験者が、Rustを書き始める時に役に立つ様々なヒントをまとめます。
特に、既にC言語で慣れ親しんでいることが、Rustではどう違うのかを強調します。

<!-- ## Preprocessor -->

## プリプロセッサ

<!--
In embedded C it is very common to use the preprocessor for a varity of
purposes, such as:
-->

組込みCでは、次のような様々な目的でプリプロセッサを使うことが一般的です。

<!--
* Compile-time selection of code blocks with `#ifdef`
* Compile-time array sizes and computations
* Macros to simplify common patterns (to avoid function call overhead)
-->

* `#ifdef`を使ったコンパイル時のコードブロック選択
* コンパイル時の配列サイズやコンパイル時計算
* （関数呼び出しのオーバーヘッドを避けるための）共通パターンを簡単化するマクロ

<!--
In Rust there is no preprocessor, and so many of these use cases are addressed
differently. In the rest of this section we cover various alternatives to
using the preprocessor.
-->

Rustにはプリプロセッサはありません。上記のユースケースは異なる方法で解決されます。
セクションの残りの部分では、プリプロセッサの様々な代替手段について説明します。

<!-- ### Compile-Time Code Selection -->

### コンパイル時コード選択

<!--
The closest match to `#ifdef ... #endif` in Rust are [Cargo features]. These
are a little more formal than the C preprocessor: all possible features are
explicitly listed per crate, and can only be either on or off. Features are
turned on when you list a crate as a dependency, and are additive: if any crate
in your dependency tree enables a feature for another crate, that feature will
be enabled for all users of that crate.
-->

`#ifdef ... #endif`に最も近いRustの機能は、[Cargoフィーチャ]です。
Cargoフィーチャは、Cプリプロセッサよりももう少し秩序だったものです。
フィーチャの候補は、クレートごとに明示的にリスト化されており、オンまたはオフのいずれかになります。
依存関係としてクレートを記載すると、フィーチャが有効になります。またこのフィーチャは追加式です。
依存ツリー内の何らかのクレートが、別クレートのフィーチャを有効化した場合、そのフィーチャは、そのクレートを使う全てのユーザに対して有効化されます。

<!-- [Cargo features]: https://doc.rust-lang.org/cargo/reference/manifest.html#the-features-section -->

[Cargoフィーチャ]: https://doc.rust-lang.org/cargo/reference/manifest.html#the-features-section

<!--
For example, you might have a crate which provides a library of signal
processing primitives. Each one might take some extra time to compile or
declare some large table of constants which you'd like to avoid. You could
declare a Cargo feature for each component in your `Cargo.toml`:
-->

例えば、信号処理プリミティブを提供するライブラリがあるとします。
それぞれが、大きな定数テーブルをコンパイルまたは宣言するのに余分な時間がかかるとすると、その時間を回避したいと思うでしょう。
`Cargo.toml`内で各コンポーネントのフィーチャを宣言することができます。

```toml
[features]
FIR = []
IIR = []
```

<!-- Then, in your code, use `#[cfg(feature="FIR")]` to control what is included. -->

それから、コード内で、何をインクルードするか制御するために`#[cfg(feature="FIR")]`を使います。

```rust
# /// In your top-level lib.rs
/// トップレベルのlib.rs内

#[cfg(feature="FIR")]
pub mod fir;

#[cfg(feature="IIR")]
pub mod iir;
```

<!--
You can similarly include code blocks only if a feature is _not_ enabled, or if
any combination or features is or is not enabled.
-->

同様に、フィーチャが有効になって _いない_ 場合にだけコードブロックをインクルードすることができます。
また、フィーチャの組み合わせや、フィーチャが有効か無効かに関わらず、コードブロックをインクルードすることもできます。

<!--
Additionally, Rust provides a number of automatically-set conditions you can
use, such as `target_arch` to select different code based on architecture. For
full details of the conditional compilation support, refer to the
[conditional compilation] chapter of the Rust reference.
-->

さらに、Rustは、自動的に設定される数々の条件を提供します。例えば、アーキテクチャに基づいて異なるコードを選択する`target_arch`です。
条件コンパイルがサポートしている全ての詳細については、Rustリファレンスの[条件コンパイル]の章を参照して下さい。

<!-- [conditional compilation]: https://doc.rust-lang.org/reference/conditional-compilation.html -->

[条件コンパイル]: https://doc.rust-lang.org/reference/conditional-compilation.html

<!--
The conditional compilation will only apply to the next statement or block. If
a block can not be used in the current scope then then `cfg` attribute will
need to be used multiple times.  It's worth noting that most of the time it is
better to simply include all the code and allow the compiler to remove dead
code when optimising: it's simpler for you and your users, and in general the
compiler will do a good job of removing unused code.
-->

条件コンパイルは、次のステートメントまたはブロックにのみ適用されます。
現在のスコープ内でブロックが使えない場合、`cfg`アトリビュートは複数回必要になります。
ほとんどの場合、単純に全てのコードをインクルードして、コンパイラが最適化時にデッドコードを削除できるようにするほうが良いことに、注意すべきです。
これは、あなたにも、あなたのユーザにとってもより簡単です。そして、通常、コンパイラは使用されていないコードをうまく削除します。

<!-- ### Compile-Time Sizes and Computation -->

### コンパイル時サイズとコンパイル時計算

<!--
Rust supports `const fn`, functions which are guaranteed to be evaluable at
compile-time and can therefore be used where constants are required, such as
in the size of arrays. This can be used alongside features mentioned above,
for example:
-->

Rustは`const fn`を提供しています。この関数はコンパイル時に評価されることが保証されているため、配列のサイズなど、定数が求められる場所で使用できます。
const fnは、上述したフィーチャと同時に使う事ができます。例を示します。

```rust
const fn array_size() -> usize {
    #[cfg(feature="use_more_ram")]
    { 1024 }
    #[cfg(not(feature="use_more_ram")]
    { 128 }
}

static BUF: [u32; array_size()] = [0u32; array_size()];
```

<!--
These are new to stable Rust as of 1.31, so documentation is still sparse. The
functionality available to `const fn` is also very limited at the time of
writing; in future Rust releases it is expected to expand on what is permitted
in a `const fn`.
-->

これらは、Rust 1.31以降の新しい機能であるため、ドキュメントはまだわずかしかありません。
執筆時点では、`const fn`で利用可能な機能は、非常に限られています。
将来のRustでは、`const fn`内で許可されることが拡張されていくでしょう。

<!-- ### Macros -->

### マクロ

<!--
Rust provides an extremely powerful [macro system]. While the C preprocessor
operates almost directly on the text of your source code, the Rust macro system
operates at a higher level. There are two varieties of Rust macro: _macros by
example_ and _procedural macros_. The former are simpler and most common; they
look like function calls and can expand to a complete expression, statement,
item, or pattern. Procedural macros are more complex but permit extremely
powerful additions to the Rust language: they can transform arbitrary Rust
syntax into new Rust syntax.
-->

Rustは、極めて強力な[マクロシステム]を提供しています。
Cプリプロセッサがソースコードのテキストにほぼ直接作用するのに対して、Rustのマクロシステムはより上位レベルで作用します。
Rustのマクロは2種類あります。_例によるマクロ_ と _手続きマクロ_ です。
前者はより単純で最も一般的なものです。関数呼び出しのように見えて、完全な式やステートメント、アイテム、パターンに展開できます。
手続きマクロは、より複雑ですが、Rust言語に非常に強力な拡張を許可します。任意のRust構文を、新しいRust構文に変換することができます。

<!-- [macro system]: https://doc.rust-lang.org/book/second-edition/appendix-04-macros.html -->

[マクロシステム]: https://doc.rust-lang.org/book/second-edition/appendix-04-macros.html

<!--
In general, where you might have used a C preprocessor macro, you probably want
to see if a macro-by-example can do the job instead. They can be defined in
your crate and easily used by your own crate or exported for other users. Be
aware that since they must expand to complete expressions, statements, items,
or patterns, some use cases of C preprocessor macros will not work, for example
a macro that expands to part of a variable name or an incomplete set of items
in a list.
-->

通常、Cプリプロセッサマクロを使っていた場所に、例によるマクロで同じことができるかどうか、確認したいと思います。
マクロは、クレート内に定義でき、自身のクレート内で簡単に使ったり、他のユーザにエクスポートしたりできます。
マクロは、完全な式や、ステートメント、アイテム、パターンに展開されなければならないため、Cプリプロセッサマクロのいくつかのユースケースではうまく機能しません。
例えば、変数名や、リスト内の不完全なアイテムの一部に展開するようなマクロです。

<!--
As with Cargo features, it is worth considering if you even need the macro. In
many cases a regular function is easier to understand and will be inlined to
the same code as a macro. The `#[inline]` and `#[inline(always)]` [attributes]
give you further control over this process, although care should be taken here
as well â€” the compiler will automatically inline functions from the same crate
where appropriate, so forcing it to do so inappropriately might actually lead
to decreased performance.
-->

Cargoフィーチャと同様、本当にマクロが必要かどうか、は検討する価値があります。
多くの場合、通常の関数は理解しやすく、マクロと同様にインライン化されます。
`#[inline]`および`#[inline(always)]`の[アトリビュート]を使用すると、このプロセスをさらに細かく制御できます。
ここでも注意が必要です。コンパイラは、適切な場合、関数を自動的にインライン化します。
そのため、不適切なインライン化を強制すると、パフォーマンスが低下する可能性があります。

<!-- [attributes]: https://doc.rust-lang.org/reference/attributes.html#inline-attribute -->

[アトリビュート]: https://doc.rust-lang.org/reference/attributes.html#inline-attribute

<!--
Explaining the entire Rust macro system is out of scope for this tips page, so
you are encouraged to consult the Rust documentation for full details.
-->

Rustのマクロシステムの全体を説明することは、このヒントページのスコープ範囲外です。
詳細については、Rustのドキュメントの参照をお勧めします。

<!-- ## Build System -->

## ビルドシステム

<!--
Most Rust crates are built using Cargo (although it is not required). This
takes care of many difficult problems with traditional build systems. However,
you may wish to customise the build process. Cargo provides [`build.rs`
scripts] for this purpose. They are Rust scripts which can interact with the
Cargo build system as required.
-->

（必須ではありませんが）ほとんどのRustのクレートは、Cargoを使ってビルドされます。
Cargoは、従来のビルドシステムに関する多くの難しい問題の面倒を見ています。
しかし、ビルドプロセスをカスタマイズしたいと思うかもしれません。このため、Cargoは[`build.rs`スクリプト]を提供しています。
build.rsスクリプトはRustで書かれたスクリプトで、必要に応じてCargoのビルドシステムとやり取りします。

<!-- [`build.rs` scripts]: https://doc.rust-lang.org/cargo/reference/build-scripts.html -->

[`build.rs`スクリプト]: https://doc.rust-lang.org/cargo/reference/build-scripts.html

<!-- Common use cases for build scripts include: -->

ビルドスクリプトの一般的なユースケースを示します。

<!--
* provide build-time information, for example statically embedding the build
  date or Git commit hash into your executable
* generate linker scripts at build time depending on selected features or other
  logic
* change the Cargo build configuration
* add extra static libraries to link against
-->

* ビルド時の情報を提供します。例えば、実行ファイルにビルド日時やGitのコミットハッシュを静的に埋め込みます。
* 選択されたフィーチャやその他のロジックに応じて、リンカスクリプトをビルド時に生成します。
* Cargoのビルド設定を変更します。
* リンクする静的ライブラリを追加します。

<!--
At present there is no support for post-build scripts, which you might
traditionally have used for tasks like automatic generation of binaries from
the build objects or printing build information.
-->

現状、ビルド後に実行するスクリプトは提供されていません。
そのようなスクリプトは、従来では、ビルドしたオブジェクトからバイナリを自動的に生成したり、ビルド情報を表示したりするタスクに使われています。

<!-- ### Cross-Compiling -->

### クロスコンパイル

<!--
Using Cargo for your build system also simplifies cross-compiling. In most
cases it suffices to tell Cargo `--target thumbv6m-none-eabi` and find a
suitable executable in `target/thumbv6m-none-eabi/debug/myapp`.
-->

Cargoをビルドシステムに使用するとクロスコンパイルも簡単になります。
多くの場合、Cargoに`--target thumbv6m-none-eabi`を伝えるだけで十分です。
そうすると、適切な実行ファイルが`target/thumbv6m-none-eabi/debug/myapp`に見つかります。

<!--
For platforms not natively supported by Rust, you will need to build `libcore`
for that target yourself. On such platforms, [Xargo] can be used as a stand-in
for Cargo which automatically builds `libcore` for you.
-->

Rustが本来サポートしていないプラットフォームの場合、ターゲットの`libcore`を自分自身でビルドする必要があります。
そのようなプラットフォームでは、[Xargo]をCargoの代わりに使うことができ、自動的に`libcore`をビルドしてくれます。

[Xargo]: https://github.com/japaric/xargo

<!-- ## Iterators vs Array Access -->

## イテレータ vs 配列アクセス

<!-- In C you are probably used to accessing arrays directly by their index: -->

Cでは、おそらくインデックスによって直接配列にアクセスしているでしょう。

```c
int16_t arr[16];
int i;
for(i=0; i<sizeof(arr)/sizeof(arr[0]); i++) {
    process(arr[i]);
}
```

<!--
In Rust this is an anti-pattern: indexed access can be slower (as it needs to
be bounds checked) and may prevent various compiler optimisations. This is an
important distinction and worth repeating: Rust will check for out-of-bounds
access on manual array indexing to guarantee memory safety, while C will
happily index outside the array.
-->

Rustでは、これはアンチパターンです。インデックスによるアクセスは、低速（境界チェックが必要なため）で様々なコンパイラの最適化を妨げます。
これは重要な違いであり、繰り返す価値があります。
Rustは、メモリ安全性を保証するために、手動で配列のインデックスを指定する際、境界を越えたアクセスをチェックします。
一方、Cでは配列外のインデックスにアクセスできてしまいます。

<!-- Instead, use iterators: -->

代わりに、イテレータを使います。

```rust
let arr = [0u16; 16];
for element in arr.iter() {
    process(*element);
}
```

<!--
Iterators provide a powerful array of functionality you would have to implement
manually in C, such as chaining, zipping, enumerating, finding the min or max,
summing, and more. Iterator methods can also be chained, giving very readable
data processing code.
-->

<!-- `chaining, zipping, enumerating`の適切な表現が考えつかないため、そのままにしてあります。 -->

イテレータは、chaining、zipping、enumerating、最小値や最大値の検索、合計の算出など、Cでは手動で実装する必要がある強力な配列の機能を提供します。
イテレータのメソッドは、連鎖することができ、非常に読みやすいデータ処理のコードになります。

<!-- See the [Iterators in the Book] and [Iterator documentation] for more details. -->

詳細は[the Bookのイテレータ]と[イテレータのドキュメント]を参照して下さい。

<!--
[Iterators in the Book]: https://doc.rust-lang.org/book/second-edition/ch13-02-iterators.html
[Iterator documentation]: https://doc.rust-lang.org/core/iter/trait.Iterator.html
-->

[the Bookのイテレータ]: https://doc.rust-lang.org/book/second-edition/ch13-02-iterators.html
[イテレータのドキュメント]: https://doc.rust-lang.org/core/iter/trait.Iterator.html

<!-- ## References vs Pointers -->

## 参照 vs ポインタ

<!--
In Rust, pointers (called [_raw pointers_]) exist but are only used in specific
circumstances, as dereferencing them is always considered `unsafe` -- Rust
cannot provide its usual guarantees about what might be behind the pointer.
-->

Rustでも、ポインタ（[_生ポインタ_ と呼びます]）は存在しますが、限られた状況でしか使いません。
ポインタの参照外しは、常に`unsafe`と考えられるからです。
Rustは、ポインタの背後にあるかもしれないものについて、通常の保証を提供できません。

<!-- [_raw pointers_]: https://doc.rust-lang.org/book/ch19-01-unsafe-rust.html#dereferencing-a-raw-pointer -->

[_生ポインタ_]: https://doc.rust-lang.org/book/ch19-01-unsafe-rust.html#dereferencing-a-raw-pointer

<!--
In most cases, we instead use _references_, indicated by the `&` symbol, or
_mutable references_, indicated by `&mut`. References behave similarly to
pointers, in that they can be dereferenced to access the underlying values, but
they are a key part of Rust's ownership system: Rust will strictly enforce that
you may only have one mutable reference _or_ multiple non-mutable references to
the same value at any given time.
-->

代わりに、ほとんどの場合、`&`のシンボルで表現される _参照_ もしくは `&mut`で表現される _ミュータブルな参照_ を使います。
参照は、ポインタと似た働きをします。つまり、裏にある値にアクセスするために参照外しができます。
しかし、参照は、Rustの所有権システムの重要な一部です。
Rustは、どんな時でも同じ値に対して、唯一のミュータブル参照を持つか、_あるいは_、複数のイミュータブル参照を持つか、を厳密に強制します。

<!--
In practice this means you have to be more careful about whether you need
mutable access to data: where in C the default is mutable and you must be
explicit about `const`, in Rust the opposite is true.
-->

実際のところ、データへのミュータブルアクセスが必要かどうか、をより慎重に検討する必要があることを意味します。
Cではデフォルトがミュータブルであり、明示的に`const`をつける必要があります。Rustではその反対です。

<!--
One situation where you might still use raw pointers is interacting directly
with hardware (for example, writing a pointer to a buffer into a DMA peripheral
register), and they are also used under the hood for all peripheral access
crates to allow you to read and write memory-mapped registers.
-->

生ポインタを使う可能性のある状況の1つは、直接ハードウェアとやり取りする時です（DMAペリフェラルのレジスタにバッファのポインタを書き込むなど）。
また、生ポインタは、メモリマップドレジスタの読み書きを可能にするために、ペリフェラルアクセスクレートの内部で使われています。

<!-- ## Volatile Access -->

## Volatileアクセス

<!--
In C, individual variables may be marked `volatile`, indicating to the compiler
that the value in the variable may change between accesses. Volatile variables
are commonly used in an embedded context for memory-mapped registers.
-->

Cでは、個別の変数に`volatile`を付けることができます。
これは、変数の値がアクセスごとに変わるかもしれない、ということをコンパイラに伝えます。
組込みでは、Volatile変数はメモリマップドレジスタに広く使用されています。

<!--
In Rust, instead of marking a variable as `volatile`, we use specific methods
to perform volatile access: [`core::ptr::read_volatile`] and
[`core::ptr::write_volatile`]. These methods take a `*const T` or a `*mut T`
(_raw pointers_, as discussed above) and perform a volatile read or write.
-->

Rustでは、変数に`volatile`を付けるのではなく、volatileアクセスをするための特定のメソッドを使います。
[`core::ptr::read_volatile`]と[`core::ptr::write_volatile`]です。
これらのメソッドは、`*const T`か`*mut T`（上述の通り _生ポインタ_ です）を受け取り、volatileな読み書きを行います。

[`core::ptr::read_volatile`]: https://doc.rust-lang.org/core/ptr/fn.read_volatile.html
[`core::ptr::write_volatile`]: https://doc.rust-lang.org/core/ptr/fn.write_volatile.html

<!-- For example, in C you might write: -->

例えば、Cでは次のように書きます。

```c
volatile bool signalled = false;

void ISR() {
    // 割り込みが発生したというシグナル
    signalled = true;
}

void driver() {
    while(true) {
        // シグナルがあるまでスリープします
        while(!signalled) { WFI(); }
        // シグナルをリセットします
        signalled = false;
        // 割り込みを待っていた何らかのタスクを実行します
        run_task();
    }
}
```

<!-- The equivalent in Rust would use volatile methods on each access: -->

Rustで同じことをするには、各アクセスにvolatileメソッドを使用します。

```rust
static mut SIGNALLED: bool = false;

#[interrupt]
fn ISR() {
#     // Signal that the interrupt has occurred
#     // (In real code, you should consider a higher level primitive,
#     //  such as an atomic type).
    // 割り込みが発生したというシグナル
    // （実際のコードでは、アトミック型のような、より上位レベルのプリミティブを検討して下さい）
    unsafe { core::ptr::write_volatile(&mut SIGNALLED, true) };
}

fn driver() {
    loop {
#         // Sleep until signalled
        // シグナルがあるまでスリープします
        while unsafe { !core::ptr::read_volatile(&SIGNALLED) } {}
#         // Reset signalled indicator
        // シグナルをリセットします
        unsafe { core::ptr::write_volatile(&mut SIGNALLED, false) };
#         // Perform some task that was waiting for the interrupt
        // 割り込みを待っていた何らかのタスクを実行します
        run_task();
    }
}
```

<!--
A few things are worth noting in the code sample:
  * We can pass `&mut SIGNALLED` into the function requiring `*mut T`, since
    `&mut T` automatically converts to a `*mut T` (and the same for `*const T`)
  * We need `unsafe` blocks for the `read_volatile`/`write_volatile` methods,
    since they are `unsafe` functions. It is the programmer's responsibility
    to ensure safe use: see the methods' documentation for further details.
-->

このコードサンプルには、いくつかの注目すべき点があります。
  * `*mut T`を要求する関数に、`&mut SIGNALLED`を渡すことができます。
    これは、`&mut T`が`*mut T`に自動的に変換されるためです（`*const T`についても同じです）。
  * `read_volatile`/`write_volatile`メソッドに`unsafe`ブロックが必要です。
    これらの関数は`unsafe`だからです。安全な使用を保証することはプログラマの責任です。
    詳細は、メソッドのドキュメントを参照して下さい。

<!--
It is rare to require these functions directly in your code, as they will
usually be taken care of for you by higher-level libraries. For memory mapped
peripherals, the peripheral access crates will implement volatile access
automatically, while for concurrency primitives there are better abstractions
available (see the [Concurrency chapter]).
-->

これらの関数をコードに直接書くことは稀です。通常、より上位レベルのライブラリで面倒を見てくれます。
メモリマップドペリフェラルについては、ペリフェラルアクセスクレートがvolatileアクセスを自動的に実装します。
並行性プリミティブの場合、より優れた抽象化が利用できます（[並行性の章]を参照して下さい）。

<!-- [Concurrency chapter]: ../concurrency/index.md -->

[並行性の章]: ../concurrency/index.md

<!-- ## Packed and Aligned Types -->

## パック型と整列型

<!--
In embedded C it is common to tell the compiler a variable must have a certain
alignment or a struct must be packed rather than aligned, usually to meet
specific hardware or protocol requirements.
-->

組込みCでは、通常、特定のハードウェアやプロトコルの要件を満たすために、変数に特定のアライメントが必要なことや、
構造体が整列されているだけでなくパックされている必要があることを、コンパイラに指示することが一般的です。

<!--
In Rust this is controlled by the `repr` attribute on a struct or union. The
default representation provides no guarantees of layout, so should not be used
for code that interoperates with hardware or C. The compiler may re-order
struct members or insert padding and the behaviour may change with future
versions of Rust.
-->

Rustでは、これは構造体または共用体の`repr`アトリビュートによって制御されます。
デフォルトでは、レイアウトは保証されないため、ハードウェアやCとやり取りするコードでは使うべきではありません。
コンパイラは、構造体のメンバを並べ替えたり、パディングを挿入したりする可能性があります。この動作は将来のバージョンのRustで変更になる可能性があります。

```rust
struct Foo {
    x: u16,
    y: u8,
    z: u16,
}

fn main() {
    let v = Foo { x: 0, y: 0, z: 0 };
    println!("{:p} {:p} {:p}", &v.x, &v.y, &v.z);
}

// 0x7ffecb3511d0 0x7ffecb3511d4 0x7ffecb3511d2
# // Note ordering has been changed to x, z, y to improve packing.
// データの詰め方を改善するために、x, y, zの順序が入れ替わっていることに注目して下さい。
```

<!-- To ensure layouts that are interoperable with C, use `repr(C)`: -->

Cと相互にやり取りできるレイアウトを保証するためには、`repr(C)`を使います。

```rust
#[repr(C)]
struct Foo {
    x: u16,
    y: u8,
    z: u16,
}

fn main() {
    let v = Foo { x: 0, y: 0, z: 0 };
    println!("{:p} {:p} {:p}", &v.x, &v.y, &v.z);
}

// 0x7fffd0d84c60 0x7fffd0d84c62 0x7fffd0d84c64
# // Ordering is preserved and the layout will not change over time.
# // `z` is two-byte aligned so a byte of padding exists between `y` and `z`.
// 順序は維持され、レイアウトは時間が経っても変化しません。
// `z`は2バイトで整列されており、`y`と`z`の間には、1バイトのパディングが存在します。
```

<!-- To ensure a packed representation, use `repr(packed)`: -->

パックされた表現を保証する場合、`repr(packed)`を使います。

```rust
#[repr(packed)]
struct Foo {
    x: u16,
    y: u8,
    z: u16,
}

fn main() {
    let v = Foo { x: 0, y: 0, z: 0 };
#     // Unsafe is required to borrow a field of a packed struct.
    // パックされた構造体のフィールドを借用するには、アンセーフが必要です。
    unsafe { println!("{:p} {:p} {:p}", &v.x, &v.y, &v.z) };
}

// 0x7ffd33598490 0x7ffd33598492 0x7ffd33598493
# // No padding has been inserted between `y` and `z`, so now `z` is unaligned.
// `y`と`z`の間にパディングは挿入されていません。そのため、`z`は整列されていません。
```

<!-- Note that using `repr(packed)` also sets the alignment of the type to `1`. -->

`repr(packed)`を使うと、型のアライメントも`1`に設定されることに注意して下さい。

<!--
Finally, to specify a specific alignment, use `repr(align(n))`, where `n` is
the number of bytes to align to (and must be a power of two):
-->

最後に、特定のアライメントを指定するために、`repr(align(n))`を使います。
ここで`n`は、整列するバイト数です（2の累乗である必要があります）。

```rust
#[repr(C)]
#[repr(align(4096))]
struct Foo {
    x: u16,
    y: u8,
    z: u16,
}

fn main() {
    let v = Foo { x: 0, y: 0, z: 0 };
    let u = Foo { x: 0, y: 0, z: 0 };
    println!("{:p} {:p} {:p}", &v.x, &v.y, &v.z);
    println!("{:p} {:p} {:p}", &u.x, &u.y, &u.z);
}

// 0x7ffec909a000 0x7ffec909a002 0x7ffec909a004
// 0x7ffec909b000 0x7ffec909b002 0x7ffec909b004
# // The two instances `u` and `v` have been placed on 4096-byte alignments,
# // evidenced by the `000` at the end of their addresses.
// 2つのインスタンス`u`と`v`は4096バイトのアライメントで配置されます。
// インスタンスのアドレスの最後は`000`になっています。
```

<!--
Note we can combine `repr(C)` with `repr(align(n))` to obtain an aligned and
C-compatible layout. It is not permissible to combine `repr(align(n))` with
`repr(packed)`, since `repr(packed)` sets the alignment to `1`. It is also not
permissible for a `repr(packed)` type to contain a `repr(align(n))` type.
-->

整列されていてCと互換性のあるレイアウトを取得するため、`repr(C)`と`repr(align(n))`とを組み合わせることができます。
`repr(align(n))`と`repr(packed)`とを組み合わせることはできません。`repr(packed)`はアライメントを`1`に設定するからです。
`repr(packed)`の型を`repr(align(n))`の型に含めることもできません。

<!--
For further details on type layouts, refer to the [type layout] chapter of the
Rust Reference.
-->

型レイアウトに関するさらなる詳細は、Rustリファレンスの[型レイアウト]の章を参照して下さい。

<!-- [type layout]: https://doc.rust-lang.org/reference/type-layout.html -->

[型レイアウト]: https://doc.rust-lang.org/reference/type-layout.html

<!-- ## Other Resources -->

## その他のリソース

<!--
* In this book:
    * [A little C with your Rust](../interoperability/c-with-rust.md)
    * [A little Rust with your C](../interoperability/rust-with-c.md)
* [The Rust Embedded FAQs](https://docs.rust-embedded.org/faq.html)
* [Rust Pointers for C Programmers](http://blahg.josefsipek.net/?p=580)
* [I used to use pointers - now what?](https://github.com/diwic/reffers-rs/blob/master/docs/Pointers.md)
-->

* 本書内
    * [Rustと少しのC](../interoperability/c-with-rust.md)
    * [Cと少しのRust](../interoperability/rust-with-c.md)
* [組込みRustよくある質問](https://docs.rust-embedded.org/faq.html)
* [CプログラマのためのRustのポインタ](http://blahg.josefsipek.net/?p=580)
* [昔はポインタを使っていました。今は？](https://github.com/diwic/reffers-rs/blob/master/docs/Pointers.md)