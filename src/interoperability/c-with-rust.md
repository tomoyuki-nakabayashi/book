<!-- # A little C with your Rust -->

# Rustと少しのC

<!-- Using C or C++ inside of a Rust project consists of two major parts: -->

CまたはC++をRustプロジェクトの内部で使うには、主に2つの対応をします。

<!--
- Wrapping the exposed C API for use with Rust
- Building your C or C++ code to be integrated with the Rust code
-->

- 公開されているCのAPIを、Rustで使えるようにラッピングする
- CまたはC++のコードを、Rustのコードと一緒にビルドする

<!--
As C++ does not have a stable ABI for the Rust compiler to target, it is recommended to use the `C` ABI when combining Rust with C or C++.
-->

C++は、Rustコンパイラがターゲットにできる安定したABIを持っていないため、CまたはC++とRustとを組み合わせるときは、`C`のABIを使うのがお勧めです。

<!-- ## Defining the interface -->

## インタフェースの定義

<!--
Before consuming C or C++ code from Rust, it is necessary to define (in Rust) what data types and function signatures exist in the linked code. In C or C++, you would include a header (`.h` or `.hpp`) file which defines this data. In Rust, it is necessary to either manually translate these definitions to Rust, or use a tool to generate these definitions.
-->

RustからCまたはC++のコードを使う前に、リンクされるコードにどのようなデータ型や関数シグネチャが存在するかを、（Rustに）定義する必要があります。
CまたはC++では、このデータを定義したヘッダファイル（`.h`または`.hpp`）をインクルードします。
Rustでは、これらの定義を手動で変換するか、定義を自動生成するツールを使うか、のいずれかが必要です。

<!-- First, we will cover manually translating these definitions from C/C++ to Rust. -->

まずは、C/C++からRustに、定義を手動で変換する方法を説明します。

<!-- ### Wrapping C functions and Datatypes -->

### Cの関数とデータ型のラッピング

<!--
Typically, libraries written in C or C++ will provide a header file defining all types and functions used in public interfaces. An example file may look like this:
-->

通常、CまたはC++で書かれたライブラリは、公開インタフェースで使用する全ての型と関数を定義するヘッダファイルを提供します。
ヘッダファイルの例は次の通りです。

```C
/* File: cool.h */
typedef struct CoolStruct {
    int x;
    int y;
} CoolStruct;

void cool_function(int i, char c, CoolStruct* cs);
```

<!-- When translated to Rust, this interface would look as such: -->

Rustに変換すると、このインタフェースは次のようになります。

```rust,ignore
/* File: cool_bindings.rs */
#[repr(C)]
pub struct CoolStruct {
    pub x: cty::c_int,
    pub y: cty::c_int,
}

pub extern "C" fn cool_function(
    i: cty::c_int,
    c: cty::c_char,
    cs: *mut CoolStruct
);
```

<!-- Let's take a look at this definition one piece at a time, to explain each of the parts. -->

各部分の説明をするため、この定義を1つずつ見ていきましょう。

```rust,ignore
#[repr(C)]
pub struct CoolStruct { ... }
```

<!--
By default, Rust does not guarantee order, padding, or the size of data included in a `struct`. In order to guarantee compatibility with C code, we include the `#[repr(C)]` attribute, which instructs the Rust compiler to always use the same rules C does for organizing data within a struct.
-->

デフォルトでは、Rustは`struct`に含まれるデータの順序やパディング、サイズを保証しません。
Cのコードとの互換性を保証するために、`#[repr(C)]`アトリビュートを使います。
このアトリビュートにより、Rustコンパイラは、構造体のデータをCと同じルールで構成します。

```rust,ignore
pub x: cty::c_int,
pub y: cty::c_int,
```

<!--
Due to the flexibility of how C or C++ defines an `int` or `char`, it is recommended to use primitive data types defined in `cty`, which will map types from C to types in Rust
-->

CまたはC++が`int`や`char`を定義する方法は柔軟であるため、`cty`で定義されているプリミティブデータ型の使用をお勧めします。
ctyは、Cの型をRustの型にマップします。

```rust,ignore
pub extern "C" fn cool_function( ... );
```

<!--
This statement defines the signature of a function that uses the C ABI, called `cool_function`. By defining the signature without defining the body of the function, the definition of this function will need to be provided elsewhere, or linked into the final library or binary from a static library.
-->

このステートメントは、`cool_function`という名前の、CのABIを使った関数シグネチャを定義します。
関数本体の定義なしにシグネチャを定義することで、この関数の定義は別の場所で与えられるか、静的ライブラリから最終的なライブラリまたはバイナリにリンクされる必要があります。

```rust,ignore
    i: cty::c_int,
    c: cty::c_char,
    cs: *mut CoolStruct
```

<!--
Similar to our datatype above, we define the datatypes of the function arguments using C-compatible definitions. We also retain the same argument names, for clarity.
-->

上記のデータ型と同様に、Cと互換の定義を使って、関数の引数のデータ型を定義します。
わかりやすくするために、引数の名前は同じままにしてあります。

<!--
We have one new type here, `*mut CoolStruct`. As C does not have a concept of Rust's references, which would look like this: `&mut CoolStruct`, we instead have a raw pointer. As dereferencing this pointer is `unsafe`, and the pointer may in fact be a `null` pointer, care must be taken to ensure the guarantees typical of Rust when interacting with C or C++ code.
-->

`*mut CoolStruct`という新しい型があります。Cは、`&mut CoolStruct`のようなRustの参照という概念を持っていません。
代わりに、生ポインタを使います。
このポインタの参照外しは、`unsafe`です。また、このポインタは実際に`null`ポインタになる可能性があります。
CまたはC++のコードとやり取りする時には、Rustが通常行う保証を確実にするように気をつける必要があります。

<!-- ### Automatically generating the interface -->

## インタフェースの自動生成

<!--
Rather than manually generating these interfaces, which may be tedious and error prone, there is a tool called [bindgen] which will perform these conversions automatically. For instructions of the usage of [bindgen], please refer to the [bindgen user's manual], however the typical process consists of the following:
-->

面倒であり間違いの元である手動のインタフェース生成ではなく、[bindgen]と呼ばれるインタフェース変換を自動で行ってくれるツールがあります。
[bindgen]の使用手順は、[bindgenユーザマニュアル]を参照して下さい。
[bindgen]を使用するための一般的なプロセスは、次の通りです。

<!--
1. Gather all C or C++ headers defining interfaces or datatypes you would like to use with Rust
2. Write a `bindings.h` file, which `#include "..."`'s each of the files you gathered in step one
3. Feed this `bindings.h` file, along with any compilation flags used to compile
  your code into `bindgen`. Tip: use `Builder.ctypes_prefix("cty")` /
  `--ctypes-prefix=cty` to make the generated code `#![no_std]` compatible.
4. `bindgen` will produce the generated Rust code to the output of the terminal window. This file may be piped to a file in your project, such as `bindings.rs`. You may use this file in your Rust project to interact with C/C++ code compiled and linked as an external library
-->

1. Rustで使いたいインタフェースやデータ型を定義している全てのCまたはC++ヘッダを集めます。
2. ステップ1で集めたヘッダファイルを`#include "..."`する`bindings.h`ファイルを書きます。
3. `bindgen`にコンパイルフラグとコードと共に、`bindings.h`を与えます。
   ヒント：`#![no_std]`互換なコードを生成するために、`Builder.ctypes_prefix("cty")`と`--ctypes-prefix=cty`を使って下さい。
4. `bindgen`は、端末のウィンドウに生成したRustコードを出力します。これは、`bindings.rs`のようなプロジェクトのファイルにパイプすることができます。
   外部ライブラリとしてコンパイル、リンクされたC/C++コードとやり取りするために、このファイルをRustプロジェクトで使用できます。

<!--
[bindgen]: https://github.com/rust-lang-nursery/rust-bindgen
[bindgen user's manual]: https://rust-lang.github.io/rust-bindgen/
-->

[bindgen]: https://github.com/rust-lang-nursery/rust-bindgen
[bindgenユーザマニュアル]: https://rust-lang.github.io/rust-bindgen/

<!-- ## Building your C/C++ code -->

## C/C++コードのビルド

<!--
As the Rust compiler does not directly know how to compile C or C++ code (or code from any other language, which presents a C interface), it is necessary to compile your non-Rust code ahead of time.
-->

Rustコンパイラは、CまたはC++のコード（または、Cのインタフェースを提供する他の言語のコード）をコンパイルする方法を直接は知らないため、Rustでないコードは事前にコンパイルする必要があります。

<!--
For embedded projects, this most commonly means compiling the C/C++ code to a static archive (such as `cool-library.a`), which can then be combined with your Rust code at the final linking step.
-->

組込みプロジェクトでは、C/C++のコードを（`cool-library.a`のような）静的なアーカイブにコンパイルすることを意味します。
このアーカイブは、最終リンク時に、Rustのコードに組み込むことができます。

<!--
If the library you would like to use is already distributed as a static archive, it is not necessary to rebuild your code. Just convert the provided interface header file as described above, and include the static archive at compile/link time.
-->

使おうとしているライブラリが、既に静的なアーカイブとして配布されている場合、そのコードを再度ビルドする必要はありません。
提供されているインタフェースのヘッダファイルを、上述の方法で、単に変換するだけです。
そして、その静的なアーカイブをコンパイル/リンク時に組み込みます。

<!--
If your code exists as a source project, it will be necessary to compile your C/C++ code to a static library, either by triggering your existing build system (such as `make`, `CMake`, etc.), or by porting the necessary compilation steps to use a tool called the `cc` crate. For both of these steps, it is necessary to use a `build.rs` script.
-->

もしコードがソースファイルとして存在するのであれば、C/C++のコードを静的ライブラリとしてコンパイルする必要があります。
（`make`や`CMake`のような）ビルドシステムを使うか、必要なコンパイルステップを`cc`クレートを使って移植するか、いずれかの方法を取れます。
どちらの方法でも、`build.rs`スクリプトを使う必要があります。

<!-- ### Rust `build.rs` build scripts -->

### Rustの`build.rs`ビルドスクリプト

<!--
A `build.rs` script is a file written in Rust syntax, that is executed on your compilation machine, AFTER dependencies of your project have been built, but BEFORE your project is built.
-->

`build.rs`スクリプトは、Rustの構文で書かれたファイルです。
このスクリプトは、プロジェクトの依存をビルドした後、プロジェクトをビルドする前に、コンパイルを行っているコンピュータ上で実行されます。

<!--
The full reference may be found [here](https://doc.rust-lang.org/cargo/reference/build-scripts.html). `build.rs` scripts are useful for generating code (such as via [bindgen]), calling out to external build systems such as `Make`, or directly compiling C/C++ through use of the `cc` crate
-->

完全なリファレンスは[ここ](https://doc.rust-lang.org/cargo/reference/build-scripts.html)にあります。
`build.rs`スクリプトは、（[bindgen]のようなツールを用いて）コードを生成したり、
`Make`のような外部ビルドシステムを呼び出したり、`cc`クレートを使ってC/C++を直接ビルドしたりするのに便利です。

<!-- ### Triggering external build systems -->

### 外部ビルドシステムの使用

<!--
For projects with complex external projects or build systems, it may be easiest to use [`std::process::Command`] to "shell out" to your other build systems by traversing relative paths, calling a fixed command (such as `make library`), and then copying the resulting static library to the proper location in the `target` build directory.
-->

<!-- "shell out"を「シェルを呼び出す」としています。多分意図するところはあっているはずです。 -->

複雑な外部プロジェクトや外部ビルドシステムを使うプロジェクトに関しては、[`std::process::Command`]を使って、他のビルドシステムに対して「シェルを呼び出す」のが最も簡単でしょう。
これにより、相対パスを渡り歩いたり、（`make library`のような）固定のコマンドを呼び出したり、その後、`target`ビルドディレクトリにある静的ライブラリをコピーしたりすることができます。

<!--
While your crate may be targeting a `no_std` embedded platform, your `build.rs` executes only on machines compiling your crate. This means you may use any Rust crates which will run on your compilation host.
-->

作っているクレートが`no_std`な組込みプラットフォームをターゲットにしていたとしても、`build.rs`はクレートをコンパイルしているコンピュータ上でのみ動作します。
そのため、コンパイルしているホスト上で動作する、あらゆるRustのクレートを使うことができます。

<!-- ### Building C/C++ code with the `cc` crate -->

### C/C++コードを`cc`クレートでビルド

<!--
For projects with limited dependencies or complexity, or for projects where it is difficult to modify the build system to produce a static library (rather than a final binary or executable), it may be easier to instead utilize the [`cc` crate], which provides an idiomatic Rust interface to the compiler provided by the host.
-->

依存や複雑さが少ないプロジェクトや、（最終バイナリや実行ファイルではなく）静的ライブラリを作成するためにビルドシステムを修正するのが難しいプロジェクトであれば、
代わりに[`cc`クレート]を使う方が簡単でしょう。
ccクレートは、ホスト上のコンパイラへの、扱いやすいRustインタフェースを提供します。

<!-- [`cc` crate]: https://github.com/alexcrichton/cc-rs -->

[`cc`クレート]: https://github.com/alexcrichton/cc-rs

<!--
In the simplest case of compiling a single C file as a dependency to a static library, an example `build.rs` script using the [`cc` crate] would look like this:
-->

単一のCファイルを静的ライブラリにコンパイルする最も単純な例では、[`cc`クレート]を使った`build.rs`スクリプトは次のようになります。

```rust
extern crate cc;

fn main() {
    cc::Build::new()
        .file("foo.c")
        .compile("libfoo.a");
}
```
