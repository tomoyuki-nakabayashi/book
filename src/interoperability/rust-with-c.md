<!-- # A little Rust with your C -->

# Cと少しのRust

<!-- Using Rust code inside a C or C++ project mostly consists of two parts. -->

CまたはC++のプロジェクト内でRustのコードを使うためには、主に次の2つの対応をします。

<!--
- Creating a C-friendly API in Rust
- Embedding your Rust project into an external build system
-->

- Cが扱いやすいAPIをRustに作ります
- 外部ビルドシステムにRustプロジェクトを組み込みます

<!--
Apart from `cargo` and `meson`, most build systems don't have native Rust support.
So you're most likely best off just using `cargo` for compiling your crate and
any dependencies.
-->

`cargo`と`meson`は別として、ほとんどのビルドシステムはRustをサポートしていません。
そのため、自分のクレートとその依存関係をコンパイルするには、`cargo`を使うのが一番です。

<!-- ## Setting up a project -->

## プロジェクトの準備

<!-- Create a new `cargo` project as usual. -->

いつもどおり、新しい`cargo`プロジェクトを作成します。

<!--
There are flags to tell `cargo` to emit a systems library, instead of
it's regular rust target.
This also allows you to set a different output name for your library,
if you want it to differ from the rest of your crate.
-->

通常のRustのターゲットではなく、システムライブラリを出力するように、`cargo`に指示するフラグがあります。
クレートの残り部分と異なる名前を付けたい場合、ライブラリに対して、別の名前を設定することもできます。

```toml
[lib]
name = "your_crate"
crate-type = ["cdylib"]      # 動的ライブラリを作ります
# crate-type = ["staticlib"] # 静的ライブラリを作ります
```

<!-- ## Building a `C` API -->

## `C` APIの作成

<!--
Because C++ has no stable ABI for the Rust compiler to target, we use `C` for
any interoperability between different languages. This is no exception when using Rust
inside of C and C++ code.
-->

C++は、Rustコンパイラがターゲットにできる安定したABIを持っていないため、別言語との相互運用には、CのABIを使用します。
CとC++のコード内でRustを使うとき、このことに例外はありません。

### `#[no_mangle]`

<!--
The Rust compiler mangles symbol names differently than native code linkers expect.
As such, any function that Rust exports to be used outside of Rust needs to be told
not to be mangled by the compiler.
-->

Rustコンパイラは、シンボル名をネイティブコードリンカが期待するものとは異なるものにマングルします。
そのため、Rustの外にエクスポートするRustの関数は、マングルしないようにコンパイラに指示する必要があります。

### `extern "C"`

<!--
By default, any function you write in Rust will use the
Rust ABI (which is also not stabilized).
Instead, when building outwards facing FFI APIs we need to
tell the compiler to use the system ABI.
-->

デフォルトでは、Rustに書いたいずれの関数もRustのABI（これも安定化されていません）を使います。
代わりに、外に公開するFFI APIはシステムABIを使うように、コンパイラに指示する必要があります。

<!--
Depending on your platform, you might want to target a specific ABI version, which are
documented [here](https://doc.rust-lang.org/reference/items/external-blocks.html).
-->

プラットフォームによっては、特定のABIバージョンをターゲットにしたい場合があります。
ABIについては、[ここ](https://doc.rust-lang.org/reference/items/external-blocks.html)にドキュメントがあります。

---

<!--
Putting these parts together, you get a function that looks roughly like this.
-->

これらの部品をまとめると、おおよそ次のような関数になります。

```rust,ignore
#[no_mangle]
pub extern "C" fn rust_function() {

}
```

<!--
Just as when using `C` code in your Rust project you now need to transform data
from and to a form that the rest of the application will understand.
-->

Rustプロジェクトで`C`コードを使った時と同様に、異なる言語間で理解できるデータ型に変換する必要があります。

<!-- ## Linking and greater project context. -->

## リンクとより大きなプロジェクトの状況

<!--
So then, that's one half of the problem solved.
How do you use this now?
-->

ここまでで、問題の半分は解決しました。
これをどうやって使うのでしょうか？

<!-- **This very much depends on your project and/or build system** -->

**それは、プロジェクトやビルドシステムに強く依存します。**

<!--
`cargo` will create a `my_lib.so`/`my_lib.dll` or `my_lib.a` file,
depending on your platform and settings. This library can simply be linked
by your build system.
-->

`cargo`は、プラットフォームと設定に依存して、`my_lib.so`、`my_lib.dll`または`my_lib.a`ファイルを作成します。
このライブラリは、そのプラットフォームのビルドシステムによって簡単にリンクすることができます。

<!--
However, calling a Rust function from C requires a header file to declare
the function signatures.
-->

しかし、CからRustを呼ぶためには、関数シグネチャを宣言するためのヘッダファイルが必要です。

<!-- Every function in your Rust-ffi API needs to have a corresponding header function. -->

Rust-ffi APIの関数全てが、対応する関数ヘッダを持つ必要があります。

```rust,ignore
#[no_mangle]
pub extern "C" fn rust_function() {}
```

<!-- would then become -->

上記は、次のようになるでしょう。

```C
void rust_function();
```

<!-- etc. -->

などなど。

<!--
There is a tool to automate this process,
called [cbindgen] which analyses your Rust code
and then generates headers for your C and C++ projects from it.
-->

このプロセスを自動化する[cbindgen]というツールがあります。
このツールは、Rustのコードを解析して、CとC++プロジェクトのためのヘッダを生成します。

[cbindgen]: https://github.com/eqrion/cbindgen

<!--
At this point, using the Rust functions from C
is as simple as including the header and calling them!
-->

この時点で、CからRustの関数を使うには、単にヘッダをインクルードして、それを呼び出すだけです！

```C
#include "my-rust-project.h"
rust_function();
```