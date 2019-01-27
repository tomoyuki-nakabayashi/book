<!-- # Collections -->

# コレクション

<!--
Eventually you'll want to use dynamic data structures (AKA collections) in your
program. `std` provides a set of common collections: [`Vec`], [`String`],
[`HashMap`], etc. All the collections implemented in `std` use a global dynamic
memory allocator (AKA the heap).
-->

いずれは、プログラム内で動的なデータ構造（別名コレクション）を使いたいでしょう。
`std`は、[`Vec`]や[`String`]、[`HashMap`]といった、一般的なコレクションを提供しています。
`std`で実装されている全てのコレクションは、グローバルな動的アロケータ（別名ヒープ）を使用します。

[`Vec`]: https://doc.rust-lang.org/std/vec/struct.Vec.html
[`String`]: https://doc.rust-lang.org/std/string/struct.String.html
[`HashMap`]: https://doc.rust-lang.org/std/collections/struct.HashMap.html

<!--
As `core` is, by definition, free of memory allocations these implementations
are not available there, but they can be found in the *unstable* `alloc` crate
that's shipped with the compiler.
-->

`core`は、定義上、メモリアロケーションがないためコレクションの実装を使うことができません。
しかし、コンパイラと共に配布されている*安定化していない*`alloc`クレートの中にそれらのコレクションがあります。

<!--
If you need collections, a heap allocated implementation is not your only
option. You can also use *fixed capacity* collections; one such implementation
can be found in the [`heapless`] crate.
-->

もしコレクションが必要であれば、ヒープに割り当てる実装だけが選択肢ではありません。
*サイズが固定された*コレクションを使うことができます。そのような実装は[`heapless`]クレートの中にあります。

[`heapless`]: https://crates.io/crates/heapless

<!-- In this section, we'll explore and compare these two implementations. -->

このセクションでは、コレクションの２つの実装を取り上げ、比較します。

<!-- ## Using `alloc` -->

## `alloc`を使用

<!--
The `alloc` crate is shipped with the standard Rust distribution. To import the
crate you can directly `use` it *without* declaring it as a dependency in your
`Cargo.toml` file.
-->

`alloc`クレートは、標準のRust配布物に同梱されています。このクレートをインポートするには、
`Cargo.toml`ファイルに依存関係を宣言*することなしに*直接`use`します。

``` rust,ignore
#![feature(alloc)]

extern crate alloc;

use alloc::vec::Vec;
```

<!--
To be able to use any collection you'll first need use the `global_allocator`
attribute to declare the global allocator your program will use. It's required
that the allocator you select implements the [`GlobalAlloc`] trait.
-->

コレクションを使うには、まず最初に、プログラム中のグローバルアロケータを宣言する`global_allocator`アトリビュートを使う必要があります。
選択したアロケータが[`GlobalAlloc`]トレイトを実装することが求められます。

[`GlobalAlloc`]: https://doc.rust-lang.org/core/alloc/trait.GlobalAlloc.html

<!--
For completeness and to keep this section as self-contained as possible we'll
implement a simple bump pointer allocator and use that as the global allocator.
However, we *strongly* suggest you use a battle tested allocator from crates.io
in your program instead of this allocator.
-->

このセクションを可能な限り自己完結させるため、グローバルアロケータとして、単純なポインタを増加するだけのアロケータを実装します。
しかしながら、あなたのプログラムではこのアロケータでなく、crates.ioから歴戦のアロケータを使用することを*強く*お勧めします。

``` rust,ignore
# // Bump pointer allocator implementation
// ポインタを増加するだけのアロケータ実装

extern crate cortex_m;

use core::alloc::GlobalAlloc;
use core::ptr;

use cortex_m::interrupt;

# // Bump pointer allocator for *single* core systems
// *シングル*コアシステム用のポインタを増加するだけのアロケータ
struct BumpPointerAlloc {
    head: UnsafeCell<usize>,
    end: usize,
}

unsafe impl Sync for BumpPointerAlloc {}

unsafe impl GlobalAlloc for BumpPointerAlloc {
    unsafe fn alloc(&self, layout: Layout) -> *mut u8 {
#         // `interrupt::free` is a critical section that makes our allocator safe
#         // to use from within interrupts
        // `interrupt::free`は、割り込み内でアロケータを安全に使えるようにするための
        // クリティカルセクションです。
        interrupt::free(|_| {
            let head = self.head.get();

            let align = layout.align();
            let res = *head % align;
            let start = if res == 0 { *head } else { *head + align - res };
            if start + align > self.end {
#                 // a null pointer signal an Out Of Memory condition
                // ヌルポインタはメモリ不足の状態を知らせます
                ptr::null_mut()
            } else {
                *head = start + align;
                start as *mut u8
            }
        })
    }

    unsafe fn dealloc(&self, _: *mut u8, _: Layout) {
#         // this allocator never deallocates memory
        // このアロケータはメモリを解放しません
    }
}

# // Declaration of the global memory allocator
# // NOTE the user must ensure that the memory region `[0x2000_0100, 0x2000_0200]`
# // is not used by other parts of the program
// グローバルメモリアロケータの宣言
// ユーザはメモリ領域の`[0x2000_0100, 0x2000_0200]`がプログラムの他の部分で使用されないことを
// 保証しなければなりません。
#[global_allocator]
static HEAP: BumpPointerAlloc = BumpPointerAlloc {
    head: UnsafeCell::new(0x2000_0100),
    end: 0x2000_0200,
};
```

<!--
Apart from selecting a global allocator the user will also have to define how
Out Of Memory (OOM) errors are handled using the *unstable*
`alloc_error_handler` attribute.
-->

グローバルアロケータの選択とは別に、ユーザはメモリ不足（OOM）エラーの処理方法を、
*安定化していない*`alloc_error_handler`アトリビュートを使って定義する必要があります。

``` rust,ignore
#![feature(alloc_error_handler)]

use cortex_m::asm;

#[alloc_error_handler]
fn on_oom(_layout: Layout) -> ! {
    asm::bkpt();

    loop {}
}
```

<!-- Once all that is in place, the user can finally use the collections in `alloc`. -->

全ての準備が整うと、ユーザはついに`alloc`のコレクションを使うことができます。

``` rust
#[entry]
fn main() -> ! {
    let mut xs = Vec::new();

    xs.push(42);
    assert!(xs.pop(), Some(42));

    loop {
        // ..
    }
}
```

<!--
If you have used the collections in the `std` crate then these will be familiar
as they are exact same implementation.
-->

`std`クレートのコレクションを使ったことがあれば、実装が全く同じものであるため、これらのコレクションはお馴染みでしょう。

<!-- ## Using `heapless` -->

## `heapless`の使用

<!--
`heapless` requires no setup as its collections don't depend on a global memory
allocator. Just `use` its collections and proceed to instantiate them:
-->

`heapless`のコレクションはグローバルメモリアロケータに依存しないため、準備は不要です。
単にコレクションを`use`して、インスタンスを作成するだけです。

``` rust
extern crate heapless; // v0.4.x

use heapless::Vec;
use heapless::consts::*;

#[entry]
fn main() -> ! {
    let mut xs: Vec<_, U8> = Vec::new();

    xs.push(42).unwrap();
    assert_eq!(xs.pop(), Some(42));
}
```

<!-- You'll note two differences between these collections and the ones in `alloc`. -->

`alloc`のコレクションとは違う点が2つあることに留意して下さい。

<!--
First, you have to declare upfront the capacity of the collection. `heapless`
collections never reallocate and have fixed capacities; this capacity is part of
the type signature of the collection. In this case we have declared that `xs`
has a capacity of 8 elements that is the vector can, at most, hold 8 elements.
This is indicated by the `U8` (see [`typenum`]) in the type signature.
-->

1つ目は、コレクションの容量を最初に宣言しなければならないことです。
`heapless`コレクションは、再割り当てすることはなく、固定の容量になります。この容量はコレクションの型シグネチャの一部になります。
上記の例では、`xs`は8要素の容量を持つように宣言しています。このベクタは最大で8つの要素を保持することができます。
型シグネチャの`U8`（[`typenum`]を参照）がこのことを表しています。

[`typenum`]: https://crates.io/crates/typenum

<!--
Second, the `push` method, and many other methods, return a `Result`. Since the
`heapless` collections have fixed capacity all operations that insert elements
into the collection can potentially fail. The API reflects this problem by
returning a `Result` indicating whether the operation succeeded or not. In
contrast, `alloc` collections will reallocate themselves on the heap to increase
their capacity.
-->

2つ目は、`push`メソッドおよび他の多くのメソッドが`Result`を返すことです。
`heapless`コレクションは固定の容量を持つため、コレクションに要素を挿入する全ての操作は、失敗する可能性があります。
APIは、操作が成功したかどうかを示すための`Result`を返すことで、この問題に対処しています。
一方、`alloc`コレクションは、ヒープ上で再割り当てするため、容量を増やすことができます。

<!--
As of version v0.4.x all `heapless` collections store all their elements inline.
This means that an operation like `let x = heapless::Vec::new();` will allocate
the collection on the stack, but it's also possible to allocate the collection
on a `static` variable, or even on the heap (`Box<Vec<_, _>>`).
-->

v0.4.x以降、全ての`heapless`コレクションは、全ての要素をインラインで格納しています。
つまり、 `let x = heapless::Vec::new();`のような操作は、スタック上にコレクションを割り当てます。
また、コレクションを`static`変数や、ヒープ上（`Box<Vec<_, _>>`）にさえ、に割り当てることが可能です。

<!-- ## Trade-offs -->

## トレードオフ

<!--
Keep these in mind when choosing between heap allocated, relocatable collections
and fixed capacity collections.
-->

ヒープに割り当てられる再配置可能なコレクションと固定容量のコレクションとを選定する時は、次のことに留意して下さい。

<!-- ### Out Of Memory and error handling -->

### メモリ不足とエラー処理

<!--
With heap allocations Out Of Memory is always a possibility and can occur in
any place where a collection may need to grow: for example, all
`alloc::Vec.push` invocations can potentially generate an OOM condition. Thus
some operations can *implicitly* fail. Some `alloc` collections expose
`try_reserve` methods that let you check for potential OOM conditions when
growing the collection but you need be proactive about using them.
-->

ヒープアロケーションでは、メモリ不足は常に発生する可能性があり、コレクションが拡大する必要がある場所であれば、どこでも発生する可能性があります。
例えば、全ての`alloc::Vec.push`呼び出しは、OOM状態を引き起こす可能性があります。
そのため、一部の操作は*暗黙的に*失敗する可能性があります。
一部の`alloc`コレクションは`try_reserve`メソッドを提供しています。
このメソッドにより、コレクションを拡大する時にOOM状態が発生するかどうかを確認できますが、先を見越して使用する必要があります。

<!--
If you exclusively use `heapless` collections and you don't use a memory
allocator for anything else then an OOM condition is impossible. Instead, you'll
have to deal with collections running out of capacity on a case by case basis.
That is you'll have deal with *all* the `Result`s returned by methods like
`Vec.push`.
-->

`heapless`コレクションだけを使っていて、メモリアロケータを使用しないのであれば、OOM状態状態は発生しません。
その代わりに、コレクションの容量オーバーを個別に処理しなければなりません。
つまり、`Vec.push`のようなメソッドが返す全ての`Result`を処理することになります。

<!--
OOM failures can be harder to debug than say `unwrap`-ing on all `Result`s
returned by `heapless::Vec.push` because the observed location of failure may
*not* match with the location of the cause of the problem. For example, even
`vec.reserve(1)` can trigger an OOM if the allocator is nearly exhausted because
some other collection was leaking memory (memory leaks are possible in safe
Rust).
-->

OOM障害は、`heapless::Vec.push`が返す全ての`Result`を`unwrap`することより、デバッグが難しいでしょう。
なぜなら、障害が発生した場所は、問題の原因となる場所と一致*しない*可能性があるからです。
例えば、他のコレクションがメモリリークを起こしているせいでアロケータが枯渇しそうな場合、`vec.reserve(1)`がOOMを発生させる可能性があります
（メモリリークは安全なRustでも発生します）。

<!-- ### Memory usage -->

### メモリ使用量

Reasoning about memory usage of heap allocated collections is hard because the
capacity of long lived collections can change at runtime. Some operations may
implicitly reallocate the collection increasing its memory usage, and some
collections expose methods like `shrink_to_fit` that can potentially reduce the
memory used by the collection -- ultimately, it's up to the allocator to decide
whether to actually shrink the memory allocation or not. Additionally, the
allocator may have to deal with memory fragmentation which can increase the
*apparent* memory usage.

On the other hand if you exclusively use fixed capacity collections, store
most of them in `static` variables and set a maximum size for the call stack
then the linker will detect if you try to use more memory than what's physically
available.

Furthermore, fixed capacity collections allocated on the stack will be reported
by [`-Z emit-stack-sizes`] flag which means that tools that analyze stack usage
(like [`stack-sizes`]) will include them in their analysis.

[`-Z emit-stack-sizes`]: https://doc.rust-lang.org/beta/unstable-book/compiler-flags/emit-stack-sizes.html
[`stack-sizes`]: https://crates.io/crates/stack-sizes

However, fixed capacity collections can *not* be shrunk which can result in
lower load factors (the ratio between the size of the collection and its
capacity) than what relocatable collections can achieve.

### Worst Case Execution Time (WCET)

If are building time sensitive applications or hard real time applications then
you care, maybe a lot, about the worst case execution time of the different
parts of your program.

The `alloc` collections can reallocate so the WCET of operations that may grow
the collection will also include the time it takes to reallocate the collection,
which itself depends on the *runtime* capacity of the collection. This makes it
hard to determine the WCET of, for example, the `alloc::Vec.push` operation as
it depends on both the allocator being used and its runtime capacity.

On the other hand fixed capacity collections never reallocate so all operations
have a predictable execution time. For example, `heapless::Vec.push` executes in
constant time.

### Ease of use

`alloc` requires setting up a global allocator whereas `heapless` does not.
However, `heapless` requires you to pick the capacity of each collection that
you instantiate.

The `alloc` API will be familiar to virtually every Rust developer. The
`heapless` API tries to closely mimic the `alloc` API but it will never be
exactly the same due to its explicit error handling -- some developers may feel
the explicit error handling is excessive or too cumbersome.
