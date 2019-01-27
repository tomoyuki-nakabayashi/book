<!-- # Concurrency -->

# 並行性

<!--
Concurrency happens whenever different parts of your program might execute
at different times or out of order. In an embedded context, this includes:
-->

プログラムの異なる部分が様々なタイミングで実行されたり、アウトオブオーダに実行されると、並行性が発生します。
組込みでは、次のものが該当します。

<!--
* interrupt handlers, which run whenever the associated interrupt happens,
* various forms of multithreading, where your microprocessor regularly swaps
  between parts of your program,
* and in some systems, multiple-core microprocessors, where each core can be
  independently running a different part of your program at the same time.
-->

* 割り込みが発生するたびに実行される割り込みハンドラ
* マイクロプロセッサがプログラムの一部を定期的にスワップする様々な形式のマルチスレッド
* システムによっては、各コアがプログラムの異なる部分を同時に独立して実行できるマルチコアマイクロプロセッサ

<!--
Since many embedded programs need to deal with interrupts, concurrency will
usually come up sooner or later, and it's also where many subtle and difficult
bugs can occur. Luckily, Rust provides a number of abstractions and safety
guarantees to help us write correct code.
-->

多くの組込みプログラムは割り込みを処理する必要があるため、早かれ遅かれ、並行性は発生します。
割り込みは、捉えにくく、難しいバグが数多く発生し得る場所でもあります。
幸運なことに、Rustは正しいコードを書く助けになる抽象化と安全性保証とを、いくつか提供しています。

<!-- ## No Concurrency -->

## 並行性なし

<!--
The simplest concurrency for an embedded program is no concurrency: your
software consists of a single main loop which just keeps running, and there
are no interrupts at all. Sometimes this is perfectly suited to the problem
at hand! Typically your loop will read some inputs, perform some processing,
and write some outputs.
-->

組込みプログラムの最も簡単な並行性は、並行性がないことです。ソフトウェアは1つの動作し続けるメインループからなり、割り込みも発生しません。
時には、これが手元の問題の最適解かもしれません。
通常、ループは何か入力を受け付け、何らかの処理を行い、何かを出力します。

```rust
#[entry]
fn main() {
    let peripherals = setup_peripherals();
    loop {
        let inputs = read_inputs(&peripherals);
        let outputs = process(inputs);
        write_outputs(&peripherals, outputs);
    }
}
```

<!--
Since there's no concurrency, there's no need to worry about sharing data
between parts of your program or synchronising access to peripherals. If
you can get away with such a simple approach this can be a great solution.
-->

並行性がないため、プログラム間でのデータ共有や、ペリフェラルへのアクセス同期に悩む必要はありません。
このような単純なアプローチに逃れることができるのであれば、素晴らしい解決策かもしれません。

<!-- ## Global Mutable Data -->

## グローバルでミュータブルなデータ

<!--
Unlike non-embedded Rust, we will not usually have the luxury of creating
heap allocations and passing references to that data into a newly-created
thread. Instead our interrupt handlers might be called at any time and must
know how to access whatever shared memory we are using. At the lowest level,
this means we must have _statically allocated_ mutable memory, which
both the interrupt handler and the main code can refer to.
-->

組込みでないRustと異なり、通常、ヒープ領域を作成し、そのデータへの参照を新しく作成したスレッドに渡す、というような贅沢はできません。
代わりに、割り込みハンドラはいつでも呼び出される可能性があり、使用する共有メモリにアクセスする方法を知っていなければなりません。
最も低いレベルでは、 _静的に割り当てられた_ ミュータブルなメモリを持つ必要があることを意味します。
このメモリは、割り込みハンドラとメインコードの両方が参照できます。

<!--
In Rust, such [`static mut`] variables are always unsafe to read or write,
because without taking special care, you might trigger a race condition,
where your access to the variable is interrupted halfway through by an
interrupt which also accesses that variable.
-->

Rustでは、このような[`static mut`]変数への読み書きは、常にアンセーフです。
特別な注意を払わないと、レースコンディションを引き起こす可能性があります。
つまり、その変数へのアクセスが、さらにその変数にアクセスする割り込みによって、中断されるということです。

[`static mut`]: https://doc.rust-lang.org/book/ch19-01-unsafe-rust.html#accessing-or-modifying-a-mutable-static-variable

<!--
For an example of how this behaviour can cause subtle errors in your code,
consider an embedded program which counts rising edges of some input signal
in each one-second period (a frequency counter):
-->

この動作によって、コード内に分かりにくいエラーが発生する可能性があります。
例えば、1秒毎に入力信号の立ち上がりエッジをカウントする組込みプログラム（周波数カウンタ）を考えてみましょう。

```rust
static mut COUNTER: u32 = 0;

#[entry]
fn main() -> ! {
    set_timer_1hz();
    let mut last_state = false;
    loop {
        let state = read_signal_level();
        if state && !last_state {
#             // DANGER - Not actually safe! Could cause data races.
            // 危険。実際に安全ではありません。データ競合を引き起こす可能性があります。
            unsafe { COUNTER += 1 };
        }
        last_state = state;
    }
}

#[interrupt]
fn timer() {
    unsafe { COUNTER = 0; }
}
```

<!--
Each second, the timer interrupt sets the counter back to 0. Meanwhile, the
main loop continually measures the signal, and incremements the counter when
it sees a change from low to high. We've had to use `unsafe` to access
`COUNTER`, as it's `static mut`, and that means we're promising the compiler
we won't cause any undefined behaviour. Can you spot the race condition? The
increment on `COUNTER` is _not_ guaranteed to be atomic — in fact, on most
embedded platforms, it will be split into a load, then the increment, then
a store. If the interrupt fired after the load but before the store, the
reset back to 0 would be ignored after the interrupt returns — and we would
count twice as many transitions for that period.
-->

毎秒、タイマ割り込みはカウンタを0に戻します。同時に、メインループは信号を継続的に測定し、信号がローからハイに変わった時にカウンタをインクリメントします。
`static mut`な`COUNTER`にアクセスするためには、`unsafe`を使う必要があります。
これは、未定義動作を引き起こさないことを、コンパイラに約束するということです。
レースコンディションがどこにあるかわかりますか？
`COUNTER`のインクリメントは、アトミックであることが保証されて _いません_ 。
実際、ほとんどの組込みプラットフォームにおいて、この操作は、ロードし、インクリメントし、ストアする、という動作に分割されます。
割り込みがロードの後からストアの前に発生した場合、0に戻すリセットは、割り込みから復帰した後に無視されます。
そして、その期間では、2倍の遷移をカウントすることになります。

<!-- ## Critical Sections -->

## クリティカルセクション

<!--
So, what can we do about data races? A simple approach is to use _critical
sections_, a context where interrupts are disabled. By wrapping the access to
`COUNTER` in `main` in a critical section, we can be sure the timer interrupt
will not fire until we're finished incrementing `COUNTER`:
-->

それでは、データ競合についてどうすれば良いのでしょうか。
単純な方法は、割り込みが無効なコンテキストである _クリティカルセクション_ を使うことです。
`main`中の`COUNTER`へのアクセスを、クリティカルセクションでラッピングします。
そうすることで、`COUNTER`のインクリメントが完了するまで、タイマ割り込みが発生しないようにできます。

```rust
static mut COUNTER: u32 = 0;

#[entry]
fn main() -> ! {
    set_timer_1hz();
    let mut last_state = false;
    loop {
        let state = read_signal_level();
        if state && !last_state {
#             // New critical section ensures synchronised access to COUNTER
            // 新しいクリティカルセクションは、COUNTERへの同期アクセスを保証します
            cortex_m::interrupt::free(|_| {
                unsafe { COUNTER += 1 };
            });
        }
        last_state = state;
    }
}

#[interrupt]
fn timer() {
    unsafe { COUNTER = 0; }
}
```

<!--
In this example we use `cortex_m::interrupt::free`, but other platforms will
have similar mechanisms for executing code in a critical section. This is also
the same as disabling interrupts, running some code, and then re-enabling
interrupts.
-->

この例では`cortex_m::interrupt::free`を使いました。他のプラットフォームでもクリティカルセクションのコードを実行するための、類似の方法があります。。
これは、割り込みを無効にして、コードを実行し、再び割り込みを有効にすることと同じです。

<!--
Note we didn't need to put a critical section inside the timer interrupt,
for two reasons:
-->

タイマ割り込み内クリティカルセクションを置く必要がないことに注意して下さい。これは次の2つの理由からです。

<!--
  * Writing 0 to `COUNTER` can't be affected by a race since we don't read it
  * It will never be interrupted by the `main` thread anyway
-->

* 読み込みをしないため、`COUNTER`に0を書くことは、競合の影響を受けません
* いずれにせよ、`main`スレッドによって割り込まれることはありえません

<!--
If `COUNTER` was being shared by multiple interrupt handlers that might
_preempt_ each other, then each one might require a critical section as well.
-->

`COUNTER`がお互いに _プリエンプション_ する複数の割り込みハンドラから共有される場合、
それぞれにクリティカルセクションが必要になるでしょう。

<!--
This solves our immediate problem, but we're still left writing a lot of
`unsafe` code which we need to carefully reason about, and we might be using
critical sections needlessly — which comes at a cost to overhead and interrupt
latency and jitter.
-->

クリティカルセクションは、当面の問題を解決しますが、慎重に検討しなければならない`unsafe`なコードをまだたくさん書いています。
その結果、必要以上にクリティカルセクションを使用することになり、オーバーヘッドと割り込みレイテンシおよびジッタをもたらします。

<!--
It's worth noting that while a critical section guarantees no interrupts will
fire, it does not provide an exclusivity guarantee on multi-core systems!  The
other core could be happily accessing the same memory as your core, even
without interrupts. You will need stronger synchronisation primitives if you
are using multiple cores.
-->

注目すべき点は、クリティカルセクションでは、割り込みが発生しないことが保証されますが、
マルチコアシステムでは、排他性の保証は提供されないことです。
他のコアは、割り込みでなくても、とあるコアと同じメモリにアクセスできてしまいます。
マルチコアを使う場合、より強力な同期プリミティブが必要になります。

<!-- ## Atomic Access -->

## アトミックアクセス

<!--
On some platforms, atomic instructions are available, which provide guarantees
about read-modify-write operations. Specifically for Cortex-M, `thumbv6`
(Cortex-M0) does not provide atomic instructions, while `thumbv7` (Cortex-M3
and above) do. These instructions give an alternative to the heavy-handed
disabling of all interrupts: we can attempt the increment, it will succeed most
of the time, but if it was interrupted it will automatically retry the entire
increment operation. These atomic operations are safe even across multiple
cores.
-->

プラットフォームによっては、アトミック命令が利用できます。アトミック命令は、リードモディファイライト操作の保証を提供します。
特にCortex-Mの場合、`thumbv6`（Cortex-M0）はアトミック命令を提供しませんが、`thumbv7`（Cortex-M3以上）は提供します。
これらの命令は、全ての割り込みを無効化する手荒な方法の代替手段を提供します。
インクリメントを試みる時、ほとんどの場合は成功しますが、割り込まれた場合はインクリメント操作全体を自動的にやり直します。
このようなアトミック操作は、複数のコアにまたがっても安全です。

```rust
use core::sync::atomic::{AtomicUsize, Ordering};

static COUNTER: AtomicUsize = AtomicUsize::new(0);

#[entry]
fn main() -> ! {
    set_timer_1hz();
    let mut last_state = false;
    loop {
        let state = read_signal_level();
        if state && !last_state {
#             // Use `fetch_add` to atomically add 1 to COUNTER
            // 自動的にCOUNTERに1を加えるために`fetch_add`を使います
            COUNTER.fetch_add(1, Ordering::Relaxed);
        }
        last_state = state;
    }
}

#[interrupt]
fn timer() {
#     // Use `store` to write 0 directly to COUNTER
    // COUNTERに直接0を書くために`store`を使います
    COUNTER.store(0, Ordering::Relaxed)
}
```

<!--
This time `COUNTER` is a safe `static` variable. Thanks to the `AtomicUsize`
type `COUNTER` can be safely modified from both the interrupt handler and the
main thread without disabling interrupts. When possible, this is a better
solution — but it may not be supported on your platform.
-->

ここで、`COUNTER`は安全な`static`変数です。`AtomicUsize`のおかげで`COUNTER`の型は、割り込みを無効化することなく、
割り込みハンドラとメインスレッドの両方から安全に修正できます。
可能であれば、これはより良い解決方法です。しかし、あなたのプラットフォームではサポートされていないかもしれません。

<!--
A note on [`Ordering`]: this affects how the compiler and hardware may reorder
instructions, and also has consequences on cache visibility. Assuming that the
target is a single core platform `Relaxed` is sufficient and the most efficient
choice in this particular case. Stricter ordering will cause the compiler to
emit memory barriers around the atomic operations; depending on what you're
using atomics for you may or may not need this! The precise details of the
atomic model are complicated and best described elsewhere.
-->

[`Ordering`]の注釈：これは、コンパイラとハードウェアがどのように命令の順番を入れ替えるか、に影響を与えます。
また、キャッシュの可視性にも影響します。ターゲットがシングルコアプラットフォームだと仮定すると、`Relaxed`で十分であり、このケースでは最も効率の良い選択です。
より厳密なオーダリングでは、コンパイラはアトミック操作の前後にメモリバリアを発行します。
使用するアトミック操作によって、必要かもしれませんし、必要でないかもしれません！
アトミックモデルの正確な詳細は複雑であり、他の文書内でしっかりと説明されています。

<!-- For more details on atomics and ordering, see the [nomicon]. -->

詳細は、[ノミコン]のアトミックとオーダリングを参照して下さい。

[`Ordering`]: https://doc.rust-lang.org/core/sync/atomic/enum.Ordering.html
<!-- [nomicon]: https://doc.rust-lang.org/nomicon/atomics.html -->

[ノミコン]: https://doc.rust-lang.org/nomicon/atomics.html

<!-- ## Abstractions, Send, and Sync -->

## 抽象化、SendとSync

<!--
None of the above solutions are especially satisfactory. They require `unsafe`
blocks which must be very carefully checked and are not ergonomic. Surely we
can do better in Rust!
-->

上記の解決方法のいずれも、これと言って満足いくものではありません。
どの解決方法も`unsafe`ブロックを必要とし、非常に注意深くチェックしなければならず、人間工学的ではありません。
Rustではもっとうまくやれるはずです！

<!--
We can abstract our counter into a safe interface which can be safely used
anywhere else in our code. For this example we'll use the critical-section
counter, but you could do something very similar with atomics.
-->

カウンタを、コード内のどこからでも安全に使えるインタフェースに抽象化することができます。
次の例では、クリティカルセクションカウンタを使いますが、アトミックと非常に良く似たことが実現できます。

```rust
use core::cell::UnsafeCell;
use cortex_m::interrupt;

# // Our counter is just a wrapper around UnsafeCell<u32>, which is the heart
# // of interior mutability in Rust. By using interior mutability, we can have
# // COUNTER be `static` instead of `static mut`, but still able to mutate
# // its counter value.
// カウンタはUnsafeCell<u32>の単なるラッパです。UnsafeCellはRustの内部可変性の重要要素です。
// 内部可変性を使用することで、COUNTERを`static mut`の代わりに`static`として持つことができます。
// しかし、依然として、カウンタの値は変更することができます。
struct CSCounter(UnsafeCell<u32>);

const CS_COUNTER_INIT: CSCounter = CSCounter(UnsafeCell::new(0));

impl CSCounter {
    pub fn reset(&self, _cs: &interrupt::CriticalSection) {
#         // By requiring a CriticalSection be passed in, we know we must
#         // be operating inside a CriticalSection, and so can confidently
#         // use this unsafe block (required to call UnsafeCell::get).
        // クリティカルセクションを引数として要求することで、クリティカルセクション内で
        // 実行されなければならないことがわかります。そのため、このアンセーフブロックを
        // 自信を持って使用できます（UnsafeCell::getの呼び出しに必要です）。
        unsafe { *self.0.get() = 0 };
    }

    pub fn increment(&self, _cs: &interrupt::CriticalSection) {
        unsafe { *self.0.get() += 1 };
    }
}

# // Required to allow static CSCounter. See explanation below.
// 静的なCSCounterを許可するために必要です。以下の説明を参照して下さい。
unsafe impl Sync for CSCounter {}

# // COUNTER is no longer `mut` as it uses interior mutability;
# // therefore it also no longer requires unsafe blocks to access.
// 内部可変性を使用するため、COUNTERは、もはや`mut`ではありません。
// 従って、アクセスの際に、アンセーフなブロックも必要なくなりました。
static COUNTER: CSCounter = CS_COUNTER_INIT;

#[entry]
fn main() -> ! {
    set_timer_1hz();
    let mut last_state = false;
    loop {
        let state = read_signal_level();
        if state && !last_state {
#             // No unsafe here!
            // アンセーフはここでは必要ありません！
            interrupt::free(|cs| COUNTER.increment(cs));
        }
        last_state = state;
    }
}

#[interrupt]
fn timer() {
#     // We do need to enter a critical section here just to obtain a valid
#     // cs token, even though we know no other interrupt could pre-empt
#     // this one.
    // 有効なcsトークンを得るため、ここでクリティカルセクションに入る必要があります。
    // 他の割り込みがプリエンプションを起こさないと分かっていても必要です。
    interrupt::free(|cs| COUNTER.reset(cs));

#     // We could use unsafe code to generate a fake CriticalSection if we
#     // really wanted to, avoiding the overhead:
#     // let cs = unsafe { interrupt::CriticalSection::new() };
    // オーバーヘッドを避けるために、本当に必要であれば、偽のクリティカルセクションを生成する
    // アンセーフなコードを使うことができます。
    // let cs = unsafe { interrupt::CriticalSection::new() };
}
```

<!--
We've moved our `unsafe` code to inside our carefully-planned abstraction,
and now our appplication code does not contain any `unsafe` blocks.
-->

`unsafe`コードを慎重に検討された抽象の内側に移動しました。
そして、アプリケーションコードは、`unsafe`ブロックを含んでいません。

<!--
This design requires the application pass a `CriticalSection` token in:
these tokens are only safely generated by `interrupt::free`, so by requiring
one be passed in, we ensure we are operating inside a critical section, without
having to actually do the lock ourselves. This guarantee is provided statically
by the compiler: there won't be any runtime overhead associated with `cs`.
If we had multiple counters, they could all be given the same `cs`, without
requiring multiple nested critical sections.
-->

この設計は、アプリケーションが`CriticalSection`トークンを渡すことを要求します。
トークンは、`interrupt::free`によってのみ、安全に生成することができます。
そのため、このトークンが渡されることを要求することで、自分自身でロックを実際にかけることなしに、クリティカルセクション内で動作していることを保証します。
この保証は、静的にコンパイラによって提供されます。`cs`による実行時のオーバーヘッドはありません。
カウンタが複数ある場合、複数の入れ子になったクリティカルセクションなしに、同じ`cs`を与えることができます。

<!--
This also brings up an important topic for concurrency in Rust: the
[`Send` and `Sync`] traits. To summarise the Rust book, a type is Send
when it can safely be moved to another thread, while it is Sync when
it can be safely shared between multiple threads. In an embedded context,
we consider interrupts to be executing in a separate thread to the application
code, so variables accessed by both an interrupt and the main code must be
Sync.
-->

これは、Rustの並行性についても重要なトピックを提起します。[`Send`と`Sync`]トレイトです。
the Rust bookを要約すると、安全に別のスレッドに移動できるとき、型はSendです。
一方、複数のスレッド間で安全に共有できるとき、型はSyncです。
組込みでは、割り込みがアプリケーションコードとは異なるスレッドで動作すると考えます。
そのため、割り込みとメインコードとの両方からアクセスされる変数は、Syncでなければなりません。

<!-- [`Send` and `Sync`]: https://doc.rust-lang.org/nomicon/send-and-sync.html -->

[`Send`と`Sync`]: https://doc.rust-lang.org/nomicon/send-and-sync.html

<!--
For most types in Rust, both of these traits are automatically derived for you
by the compiler. However, because `CSCounter` contains an [`UnsafeCell`], it is
not Sync, and therefore we could not make a `static CSCounter`: `static`
variables _must_ be Sync, since they can be accessed by multiple threads.
-->

Rustのほとんどの型では、コンパイラによってSendとSyncの両方のトレイトが自動的に継承されます。
しかし、`CSCounter`は[`UnsafeCell`]を含んでいるため、Syncではありません。
従って、`static CSCounter`を作ることはできません。`static`変数は、複数のスレッドからアクセスされるため、Syncでなければなりません。

[`UnsafeCell`]: https://doc.rust-lang.org/core/cell/struct.UnsafeCell.html

<!--
To tell the compiler we have taken care that the `CSCounter` is in fact safe
to share between threads, we implement the Sync trait explicitly. As with the
previous use of critical sections, this is only safe on single-core platforms:
with multiple cores you would need to go to greater lengths to ensure safety.
-->

`CSCounter`が実はスレッド間で共有しても安全なように処理していることを、コンパイラに伝えるため、Syncトレイトを明示的に実装します。
これまでのクリティカルセクションの使用と同様に、シングルコアのプラットフォームでのみ安全です。
マルチコアのプラットフォームでは、安全性を確保するためにさらなる取り組みが必要です。

<!-- ## Mutexes -->

## ミューテックス

<!--
We've created a useful abstraction specific to our counter problem, but
there are many common abstractions used for concurrency.
-->

カウンタの問題に特有の便利な抽象化を行いましたが、並行性のために利用されるいくつかの抽象化が存在します。

<!--
One such _synchronisation primitive_ is a mutex, short for mutual exclusion.
These constructs ensure exclusive access to a variable, such as our counter. A
thread can attempt to _lock_ (or _acquire_) the mutex, and either succeeds
immediately, or blocks waiting for the lock to be acquired, or returns an error
that the mutex could not be locked. While that thread holds the lock, it is
granted access to the protected data. When the thread is done, it _unlocks_ (or
_releases_) the mutex, allowing another thread to lock it. In Rust, we would
usually implement the unlock using the [`Drop`] trait to ensure it is always
released when the mutex goes out of scope.
-->

そのような _同期プリミティブ_ の1つはミューテックス（mutex）です。mutexはmutual exclusionの略です。
ミューテックスは、私達のカウンタのような変数への排他アクセスを保証します。
あるスレッドは、ミューテックスの _ロック_（または _獲得_）を試みます。
すると、すぐに成功するか、ロックが獲得されるのを待ってブロックするか、ミューテックスをロックできなかったエラーを返します。
そのスレッドがロックを保持している間、保護されたデータへのアクセスが許可されます。
そのスレッドが実行を完了すると、ミューテックスを _アンロック_（または _解放_）することで、他のスレッドがミューテックスをロックできるようにします。
Rustでは、通常、アンロックを実装するために[`Drop`]トレイトを使用します。
これは、ミューテックスがスコープの外に到達すると、常に解放されることを保証するためです。

[`Drop`]: https://doc.rust-lang.org/core/ops/trait.Drop.html

<!--
Using a mutex with interrupt handlers can be tricky: it is not normally
acceptable for the interrupt handler to block, and it would be especially
disastrous for it to block waiting for the main thread to release a lock,
since we would then _deadlock_ (the main thread will never release the lock
because execution stays in the interrupt handler). Deadlocking is not
considered unsafe: it is possible even in safe Rust.
-->

割り込みハンドラでミューテックスを使用するのはトリッキーです。割り込みハンドラ内でブロックすることは、通常、好ましくありません。
割り込みハンドラ内で、メインスレッドがロックを解放するのを待ってブロックすると、特に悲惨です。
なぜならば。_デッドロック_ になるからです。（割り込みハンドラ内に実行がとどまるため、メインスレッドがロックを解放することは決してありません）
デッドロックはアンセーフとは考えられていません。安全なRustでも発生する可能性があります。

<!--
To avoid this behaviour entirely, we could implement a mutex which requires
a critical section to lock, just like our counter example. So long as the
critical section must last as long as the lock, we can be sure we have
exclusive access to the wrapped variable without even needing to track
the lock/unlock state of the mutex.
-->

この動作を完全に避けるため、カウンタの例で示すように、ロックのためにクリティカルセクションを必要とするミューテックスを実装できます。
クリティカルセクションがロックしている間続く限り、ミューテックスのロック/アンロックの状態を追跡することなしに、
ラップされた変数に排他的にアクセスできます。

<!--
This is in fact done for us in the `cortex_m` crate! We could have written
our counter using it:
-->

これは実際に`cortex_m`クレートで行われています！
それを使ってカウンタを書くことができます。

```rust
use core::cell::Cell;
use cortex_m::interrupt::Mutex;

static COUNTER: Mutex<Cell<u32>> = Mutex::new(Cell::new(0));

#[entry]
fn main() -> ! {
    set_timer_1hz();
    let mut last_state = false;
    loop {
        let state = read_signal_level();
        if state && !last_state {
            interrupt::free(|cs|
                COUNTER.borrow(cs).set(COUNTER.borrow(cs).get() + 1));
        }
        last_state = state;
    }
}

#[interrupt]
fn timer() {
#     // We still need to enter a critical section here to satisfy the Mutex.
    // ミューテックスを満たすために、ここでもクリティカルセクションに入る必要があります。
    interrupt::free(|cs| COUNTER.borrow(cs).set(0));
}
```

<!--
We're now using [`Cell`], which along with its sibling `RefCell` is used to
provide safe interior mutability. We've already seen `UnsafeCell` which is
the bottom layer of interior mutability in Rust: it allows you to obtain
multiple mutable references to its value, but only with unsafe code. A `Cell`
is like an `UnsafeCell` but it provides a safe interface: it only permits
taking a copy of the current value or replacing it, not taking a reference,
and since it is not Sync, it cannot be shared between threads. These
constraints mean it's safe to use, but we couldn't use it directly in a
`static` variable as a `static` must be Sync.
-->

今回は[`Cell`]を使っています。これは、`RefCell`の同類で安全な内部可変性を提供するために使用されます。
既に、`UnsafeCell`が、Rustの内部可変性の最下層であることを見てきました。
UnsafeCellは、値への複数のミュータブル参照の取得を可能としますが、アンセーフなコードでのみ使用できます。
`Cell`は`UnsafeCell`と似ていますが、安全なインタフェースを提供します。
Cellは参照を取得せず、現在値のコピーを取得するか、置き換えることだけを許可します。
CellはSyncでないため、スレッド間で共有できません。
これらの制約は安全に使えることを意味しますが、`static`変数として直接使用できません。`static`はSyncである必要があるからです。

[`Cell`]: https://doc.rust-lang.org/core/cell/struct.Cell.html

<!--
So why does the example above work? The `Mutex<T>` implements Sync for any
`T` which is Send — such as a `Cell`. It can do this safely because it only
gives access to its contents during a critical section. We're therefore able
to get a safe counter with no unsafe code at all!
-->

では、なぜ上記の例はうまく動くのでしょうか？`Mutex<T>`は、`Cell`のようなSendな`T`に対してSyncを実装します。
このことが、Cellをstaticで使うことを安全にします。なぜなら、クリティカルセクションの間だけ、その中身へのアクセスを提供するからです。
従って、全くアンセーフなコードなしに、安全なカウンタを手に入れることができます。

<!--
This is great for simple types like the `u32` of our counter, but what about
more complex types which are not Copy? An extremely common example in an
embedded context is a peripheral struct, which generally are not Copy.
For that we can turn to `RefCell`.
-->

この方法は、カウンタの`u32`のような単純な型に適しています。しかし、もっと複雑なCopyでない型についてはどうでしょうか？
組込みにおいて非常に一般的な例は、ペリフェラル構造体です。これは、通常Copyではありません。
そのためには、`RefCell`に頼ることができます。

<!-- ## Sharing Peripherals -->

## ペリフェラルの共有

<!--
Device crates generated using `svd2rust` and similar abstractions provide
safe access to peripherals by enforcing that only one instance of the
peripheral struct can exist at a time. This ensures safety, but makes it
difficult to access a peripheral from both the main thread and an interrupt
handler.
-->

`svd2rust`および同様の抽象化を使って生成されるデバイスクレートは、ペリフェラルへの安全なアクセスを提供します。
これは、同時に1つのペリフェラル構造体インスタンスしか存在できないように強制することによって、もたらされます。
このことは、安全性を保証しますが、メインスレッドと割り込みハンドとの両方からペリフェラルにアクセスすることを難しくします。

<!--
To safely share peripheral access, we can use the `Mutex` we saw before. We'll
also need to use [`RefCell`], which uses a runtime check to ensure only one
reference to a peripheral is given out at a time. This has more overhead than
the plain `Cell`, but since we are giving out references rather than copies,
we must be sure only one exists at a time.
-->

<!-- 最後の文章が自信ないです。 -->

ペリフェラルアクセスを安全に共有するため、上で見たように`Mutex`を使うことができます。
また、[`RefCell`]も必要です。RefCellは、実行時チェックを使って、同時に1つのペリフェラルへの参照だけが渡されるようにします。
実行時チェックは、普通の`Cell`よりもオーバーヘッドが大きくなりますが、
コピーではなく参照を受け渡しするため、同時に存在するのが1つだけであることを確認する必要があります。

[`RefCell`]: https://doc.rust-lang.org/core/cell/struct.RefCell.html

<!--
Finally, we'll also have to account for somehow moving the peripheral into
the shared variable after it has been initialised in the main code. To do
this we can use the `Option` type, initialised to `None` and later set to
the instance of the peripheral.
-->

最後に、メインコード内でペリフェラルを初期化した後、なんとかしてペリフェラルを共有変数に移動する方法が必要です。
これを実現するために、`Option`型を使います。`None`で初期化し、後でペリフェラルのインスタンスを設定します。

```rust
use core::cell::RefCell;
use cortex_m::interrupt::{self, Mutex};
use stm32f4::stm32f405;

static MY_GPIO: Mutex<RefCell<Option<stm32f405::GPIOA>>> =
    Mutex::new(RefCell::new(None));

#[entry]
fn main() -> ! {
#     // Obtain the peripheral singletons and configure it.
#     // This example is from an svd2rust-generated crate, but
#     // most embedded device crates will be similar.
    // ペリフェラルのシングルトンを取得し、設定します。
    // この例は、svd2rustで生成されたクレートから持ってきたものですが、
    // ほとんどの組込みデバイスクレートは同様になります。
    let dp = stm32f405::Peripherals::take().unwrap();
    let gpioa = &dp.GPIOA;

#     // Some sort of configuration function.
#     // Assume it sets PA0 to an input and PA1 to an output.
    // 一連の設定をする関数です。
    // PA0を入力、PA1を出力に設定すると仮定して下さい。
    configure_gpio(gpioa);

#     // Store the GPIOA in the mutex, moving it.
    // GPIOAをミューテックスに格納し、ムーブします。
    interrupt::free(|cs| MY_GPIO.borrow(cs).replace(Some(dp.GPIOA)));
#     // We can no longer use `gpioa` or `dp.GPIOA`, and instead have to
#     // access it via the mutex.
    // もはや`gpioa`や`dp.GPIOA`は使いません。
    // 代わりに、ミューテックス経由でアクセスする必要があります。

#     // Be careful to enable the interrupt only after setting MY_GPIO:
#     // otherwise the interrupt might fire while it still contains None,
#     // and as-written (with `unwrap()`), it would panic.
    // MY_GPIOを設定した後にのみ、割り込みを有効にするように注意して下さい。
    // そうしなければ、まだNoneが含まれている間に、割り込みが発生する可能性があります。
    // （`unwrap()`を使用して）書き込まれると、パニックになるでしょう。
    set_timer_1hz();
    let mut last_state = false;
    loop {
#         // We'll now read state as a digital input, via the mutex
        // ミューテックス経由で、デジタル入力としての状態を読み込みます。
        let state = interrupt::free(|cs| {
            let gpioa = MY_GPIO.borrow(cs).borrow();
            gpioa.as_ref().unwrap().idr.read().idr0().bit_is_set()
        });

        if state && !last_state {
#             // Set PA1 high if we've seen a rising edge on PA0.
            // PA0の立ち上がりエッジを検出した場合、PA1をハイに設定します。
            interrupt::free(|cs| {
                let gpioa = MY_GPIO.borrow(cs).borrow();
                gpioa.as_ref().unwrap().odr.modify(|_, w| w.odr1().set_bit());
            });
        }
        last_state = state;
    }
}

#[interrupt]
fn timer() {
#     // This time in the interrupt we'll just clear PA0.
    // 今回は、割り込み内では単純にPA0をクリアするだけです。
    interrupt::free(|cs| {
#         // We can use `unwrap()` because we know the interrupt wasn't enabled
#         // until after MY_GPIO was set; otherwise we should handle the potential
#         // for a None value.
        // `unwrap()`を使うことができます。割り込みはMY_GPIOが設定されるまで有効化されないことを
        // 知っているためです。そうでなければ、Noneを処理しなければならないでしょう。
        let gpioa = MY_GPIO.borrow(cs).borrow();
        gpioa.as_ref().unwrap().odr.modify(|_, w| w.odr1().clear_bit());
    });
}
```

<!-- That's quite a lot to take in, so let's break down the important lines. -->

非常に多くのことを取り入れています。重要な部分を詳細に見ていきましょう。

```rust,ignore
static MY_GPIO: Mutex<RefCell<Option<stm32f405::GPIOA>>> =
    Mutex::new(RefCell::new(None));
```

<!--
Our shared variable is now a `Mutex` around a `RefCell` which contains an
`Option`. The `Mutex` ensures we only have access during a critical section,
and therefore makes the variable Sync, even though a plain `RefCell` would not
be Sync. The `RefCell` gives us interior mutability with references, which
we'll need to use our `GPIOA`. The `Option` lets us initialise this variable
to something empty, and only later actually move the variable in. We cannot
access the peripheral singleton statically, only at runtime, so this is
required.
-->

ここでは、共有変数は`RefCell`を内部に含む`Mutex`です。さらに、RefCellは`Option`を含んでいます。
`Mutex`はクリティカルセクションの間だけ、アクセスできるようにします。
その結果、普通の`RefCell`はSyncでないにも関わらず、変数はSyncになります。
`RefCell`は、`GPIOA`を使うのに必要となる参照によって内部可変性を提供します。
`Option`を使用すると、この変数を空の値に初期化できます。後で実際に変数を移動します。
ペリフェラルのシングルトンには静的にアクセスすることはできません。実行時のみアクセスできるため、Optionが必要とされます。

```rust,ignore
interrupt::free(|cs| MY_GPIO.borrow(cs).replace(Some(dp.GPIOA)));
```

<!--
Inside a critical section we can call `borrow()` on the mutex, which gives us
a reference to the `RefCell`. We then call `replace()` to move our new value
into the `RefCell`.
-->

クリティカルセクションの内部で、ミューテックスの`borrow()`を呼んでいます。borrow()は`RefCell`の参照を提供します。
その後、`replace()`を呼び出して、`RefCell`に新しい値をムーブします。

```rust,ignore
interrupt::free(|cs| {
    let gpioa = MY_GPIO.borrow(cs).borrow();
    gpioa.as_ref().unwrap().odr.modify(|_, w| w.odr1().set_bit());
});
```

<!--
Finally we use `MY_GPIO` in a safe and concurrent fashion. The critical section
prevents the interrupt firing as usual, and lets us borrow the mutex.  The
`RefCell` then gives us an `&Option<GPIOA>`, and tracks how long it remains
borrowed - once that reference goes out of scope, the `RefCell` will be updated
to indicate it is no longer borrowed.
-->

最後に、`MY_GPIO`を安全で並行なやり方で使います。
クリティカルセクションは、通常通り割り込みの発生を防ぎ、ミューテックスを借用できます。
`RefCell`は`&Option<GPIOA>`を提供し、その借用がいつまで続くかを追跡します。
その参照がスコープ外になると、`RefCell`が借用されなくなったことを示すため、更新されます。

<!--
Since we can't move the `GPIOA` out of the `&Option`, we need to convert it to
an `&Option<&GPIOA>` with `as_ref()`, which we can finally `unwrap()` to obtain
the `&GPIOA` which lets us modify the peripheral.
-->

`&Option`の外に`GPIOA`をムーブすることはできないので、`as_ref()`を使って`&Option<&GPIOA>`に変換します。
そうすると、最終的に、`unwrap()`で`&GPIOA`を取得できます。
&GPIOAにより、ペリフェラルを修正することができます。

<!--
Whew! This is safe, but it is also a little unwieldy. Is there anything else
we can do?
-->

ヒューッ！これは安全ですが、少し大げさすぎて扱いにくいです。他に方法はないのでしょうか？

## RTFM

<!--
One alternative is the [RTFM framework], short for Real Time For the Masses. It
enforces static priorities and tracks accesses to `static mut` variables
("resources") to statically ensure that shared resources are always accessed
safely, without requiring the overhead of always entering critical sections and
using reference counting (as in `RefCell`). This has a number of advantages such
as guaranteeing no deadlocks and giving extremely low time and memory overhead.
-->

代替手段の１つは、[RTFMフレームワーク]です。RTFMは、Real Time For the Massesの略です。
RTFMは、共有リソースが常に安全にアクセスされることを保証するために、静的な優先度を適用し、
`static mut`変数（「リソース」）へのアクセスを追跡します。
この方法は、（`RefCell`のように）常にクリティカルセクションに入り、参照カウントを使うというオーバーヘッドを必要としません。
デッドロックがないことを保証したり、時間とメモリのオーバーヘッドを極めて小さくするといった、多くの利点があります。

<!-- [RTFM framework]: https://github.com/japaric/cortex-m-rtfm -->

[RTFMフレームワーク]: https://github.com/japaric/cortex-m-rtfm

<!--
The framework also includes other features like message passing, which reduces
the need for explicit shared state, and the ability to schedule tasks to run at
a given time, which can be used to implement periodic tasks. Check out [the
documentation] for more information!
-->

このフレームワークは他の機能も含んでいます。例えば、メッセージパッシングは明示的な共有状態の必要性を減らします。
また、タスクを指定した時間に実行するスケジュールする機能もあります。これは、周期タスクの実装に使えます。
詳しくは[ドキュメント]を参照して下さい。

<!-- [the documentation]: https://japaric.github.io/cortex-m-rtfm/book/ -->

[ドキュメント]: https://japaric.github.io/cortex-m-rtfm/book/

<!-- ## Real Time Operating Systems -->

## リアルタイムオペレーティングシステム

<!--
Another common model for embedded concurrency is the real-time operating system
(RTOS). While currently less well explored in Rust, they are widely used in
traditional embedded development. Open source examples include [FreeRTOS] and
[ChibiOS]. These RTOSs provide support for running multiple application threads
which the CPU swaps between, either when the threads yield control (called
cooperative multitasking) or based on a regular timer or interrupts (preemptive
multitasking). The RTOS typically provide mutexes and other synchronisation
primitives, and often interoperate with hardware features such as DMA engines.
-->

組込み向け並行性の異なる一般的なモデルとして、リアルタイムオペレーティングシステム（RTOS）があります。
現在、Rustではあまり検証されていませんが、従来の組込み開発では広く使用されています。
オープンソースの例として、[FreeRTOS]と[ChibiOS]があります。
これらのRTOSは、複数のアプリケーションスレッドを動作させる機能を提供しています。
スレッドが制御を明け渡す時（コオペレーティブマルチタスク）か、
周期タイマまたは割り込みに基づく時（プリエンプティブマルチタスク）に、CPUで実行するスレッドを切り替えます。
RTOSは、通常ミューテックスや他の同期プリミティブを提供します。また、DMAエンジンようなハードウェア機能を同時に使えることも多いです。

[FreeRTOS]: https://freertos.org/
[ChibiOS]: http://chibios.org/

<!--
At the time of writing there are not many Rust RTOS examples to point to,
but it's an interesting area so watch this space!
-->

この本を書いている時点では、Rustで書かれたRTOSの例はそれほど多くありません。
しかし、興味深い分野ですので、この分野にご注目下さい！

<!-- ## Multiple Cores -->

## マルチコア

<!--
It is becoming more common to have two or more cores in embedded processors,
which adds an extra layer of complexity to concurrency. All the examples using
a critical section (including the `cortex_m::interrupt::Mutex`) assume the only
other execution thread is the interrupt thread, but on a multi-core system
that's no longer true. Instead, we'll need synchronisation primitives designed
for multiple cores (also called SMP, for symmetric multi-processing).
-->

組込みプロセッサにおいても、2個以上のコアを持つことがより一般的になってきています。
このことは、並行性をさらに複雑にします。（`cortex_m::interrupt::Mutex`を含む）クリティカルセクションで使っている全ての例は、
他の実行スレッドは、割り込みスレッドだけであることを前提にしています。
しかし、マルチコアシステムにおいては、これは当てはまりません。
代わりに、マルチコア（SMP; symmetric multi-proccesingとも呼ばれます）向けに設計した同期プリミティブが必要になります。

<!--
These typically use the atomic instructions we saw earlier, since the
processing system will ensure that atomicity is maintained over all cores.
-->

通常、これまでに見たアトミック命令を使用します。
アトミック命令は、処理システムが全てのコア間でのアトミック性を維持してくれるためです。

<!--
Covering these topics in detail is currently beyond the scope of this book,
but the general patterns are the same as for the single-core case.
-->

これらのトピックを詳細に説明することは、この本のスコープ範囲外ですが、
一般的なパターンはシングルコアの場合と同じです。