<!-- # Interoperability -->

# 相互運用性

<!--
Interoperability between Rust and C code is always dependent
on transforming data between the two languages.
For this purposes there are two dedicated modules
in the `stdlib` called
[`std::ffi`](https://doc.rust-lang.org/std/ffi/index.html) and
[`std::os::raw`](https://doc.rust-lang.org/std/os/raw/index.html).
-->

RustとCとの相互運用性は、常に2つの言語間のデータ変換に依存しています。
そこで、2つの専用モジュールが`stdlib`内にあります。
[`std::ffi`](https://doc.rust-lang.org/std/ffi/index.html)と
[`std::os::raw`](https://doc.rust-lang.org/std/os/raw/index.html)と呼ばれるものです。

<!--
`std::os::raw` deals with low-level primitive types that can
be converted implicitly by the compiler
because the memory layout between Rust and C
is similar enough or the same.
-->

`std::os::raw`は、コンパイラによって暗黙的に変換される低レベルのプリミティブ型を扱います。
RustとCとの間で、これらのプリミティブ型のメモリレイアウトは十分似ているか、同じだからです。

<!--
`std::ffi` provides some utility for converting more complex
types such as Strings, mapping both `&str` and `String`
to C-types that are easier and safer to handle.
-->

`std::ffi`は、文字列のようなより複雑な型を変換し、`&str`と`String`の両方を、
より扱いやすく安全なCの型にマッピングするためのユーティリティを提供します。

<!--
Neither of these modules are available in `core`, but you can find a `#![no_std]`
compatible version of `std::ffi::{CStr,CString}` in the [`cstr_core`] crate, and
most of the `std::os::raw` types in the [`cty`] crate.
-->

これらの2つのモジュールは、どちらも`core`では利用できませんが、
`#![no_std]`互換バージョンの`std::ffi::{CStr,CString}`が、[`cstr_core`]クレートにあります。
そして、ほとんどの`std::os::raw`の型は、[`cty`]クレートにあります。

[`cstr_core`]: https://crates.io/crates/cstr_core
[`cty`]: https://crates.io/crates/cty

<!--
| Rust type  | Intermediate | C type       |
|------------|--------------|--------------|
| String     | CString      | *char        |
| &str       | CStr         | *const char  |
| ()         | c_void       | void         |
| u32 or u64 | c_uint       | unsigned int |
| etc        | ...          | ...          |
-->

| Rustの型    | 中間表現      | Cの型         |
|------------|--------------|--------------|
| String     | CString      | *char        |
| &str       | CStr         | *const char  |
| ()         | c_void       | void         |
| u32 or u64 | c_uint       | unsigned int |
| 他          | ...          | ...          |

<!--
As mentioned above, primitive types can be converted
by the compiler implicitly.
-->

上述の通り、プリミティブ型は、コンパイラによって暗黙的に変換されます。

```rust,ignore
unsafe fn foo(num: u32) {
    let c_num: c_uint = num;
    let r_num: u32 = c_num;
}
```

<!-- ## Interoperability with other build systems -->

## 他のビルドシステムとの相互運用性

<!--
A common requirement for including Rust in your embedded project is combining
Cargo with your existing build system, such as make or cmake.
-->

組込みプロジェクトにRustを組み込むための共通の要件は、Cargoとmakeやcmakeのような既存のビルドシステムとを組み合わせることです。

<!--
We are collecting examples and use cases for this on our issue tracker in
[issue #61].
-->

[issue #61]でこれに関する事例とユースケースを集めています。

[issue #61]: https://github.com/rust-embedded/book/issues/61


<!-- ## Interoperability with RTOSs -->

## RTOSとの相互運用性

<!--
Integrating Rust with an RTOS such as FreeRTOS or ChibiOS is still a work in
progress; especially calling RTOS functions from Rust can be tricky.
-->

RustをFreeRTOSやChibiOSといったRTOSに統合することは、まだ作業を進めている状態です。
特に、RTOSの関数をRustから呼び出すことはトリッキーです。

<!--
We are collecting examples and use cases for this on our issue tracker in
[issue #62].
-->

[issue #62]でこれに関する事例とユースケースを集めています。

[issue #62]: https://github.com/rust-embedded/book/issues/62
