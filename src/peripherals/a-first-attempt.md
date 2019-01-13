<!-- # A First Attempt -->

# 最初の試み

<!-- ## The Registers -->

## レジスタ

<!--
Let's look at the 'SysTick' peripheral - a simple timer which comes with every Cortex-M processor core. Typically you'll be looking these up in the chip manufacturer's data sheet or *Technical Reference Manual*, but this example is common to all ARM Cortex-M cores, let's look in the [ARM reference manual]. We see there are four registers:
-->

`SysTick` ペリフェラルを見ていきましょう。 `SysTick` はCortex-Mプロセッサ・コアに搭載されているシンプルなタイマーです。
通常は、チップメーカーのデータシートや*テクニカルリファレンスマニュアル*でこれらのペリフェラルの情報を調べることができるのですが、この例においては全てのARM Cortex-Mコアで共通のものですので、今回は[ARMリファレンスマニュアル]を見てみましょう。
そこには4つのレジスタが載っています。

[ARMリファレンスマニュアル]: http://infocenter.arm.com/help/topic/com.arm.doc.dui0553a/Babieigh.html

<!--
| Offset | Name        | Description                 | Width  |
|--------|-------------|-----------------------------|--------|
| 0x00   | SYST_CSR    | Control and Status Register | 32 bits|
| 0x04   | SYST_RVR    | Reload Value Register       | 32 bits|
| 0x08   | SYST_CVR    | Current Value Register      | 32 bits|
| 0x0C   | SYST_CALIB  | Calibration Value Register  | 32 bits|
-->

| オフセット | 名前        | 説明                     | 幅     |
|----------|-------------|-------------------------|--------|
| 0x00     | SYST_CSR    | 制御およびステータスレジスタ | 32ビット|
| 0x04     | SYST_RVR    | リロード値レジスタ         | 32ビット|
| 0x08     | SYST_CVR    | 現在値レジスタ            | 32ビット|
| 0x0C     | SYST_CALIB  | キャリブレーション値レジスタ | 32ビット|

<!-- ## The C Approach -->

## Cアプローチ

<!--
In Rust, we can represent a collection of registers in exactly the same way as we do in C - with a `struct`.
-->

Rustでも、`struct`を使ってCと同じ方法でレジスタの集合を正確に表現することができます。

```rust,ignore
#[repr(C)]
struct SysTick {
    pub csr: u32,
    pub rvr: u32,
    pub cvr: u32,
    pub calib: u32,
}
```

<!--
The qualifier `#[repr(C)]` tells the Rust compiler to lay this structure out like a C compiler would. That's very important, as Rust allows structure fields to be re-ordered, while C does not. You can imagine the debugging we'd have to do if these fields were silently re-arranged by the compiler! With this qualifier in place, we have our four 32-bit fields which correspond to the table above. But of course, this `struct` is of no use by itself - we need a variable.
-->

`#[repr(C)]`修飾子はRustコンパイラ対して、この構造体をCコンパイラと同じようにメモリにレイアウトするように指示します。
これはとても重要なことです。なぜならRustでは、Cにおいては行われない構造体のフィールドの並び替えが許されているためです。
コンパイラによって暗黙のうちに構造体のフィールドが並び替えられることにより、デバッグをするはめになることは想像できるでしょう。
この修飾子を置くことで、4つの32ビットの各フィールドは上記のテーブルに対応付けられます。
ただしもちろん、この`struct`はそれ自体では何の役にも立ちません。次のように変数として使う必要があります。

```rust,ignore
let systick = 0xE000_E010 as *mut SysTick;
let time = unsafe { (*systick).cvr };
```

<!-- ## Volatile Accesses -->

## volatileアクセス

<!--
Now, there are a couple of problems with the approach above.
-->

上記のやり方には、いくつか問題があります。

<!--
1. We have to use unsafe every time we want to access our Peripheral.
2. We've got no way of specifying which registers are read-only or read-write.
3. Any piece of code anywhere in your program could access the hardware
   through this structure.
4. Most importantly, it doesn't actually work...
-->

1. ペリフェラルにアクセスするためには毎回unsafeを使わなくてはなりません。
2. どのレジスタが読み取り専用でどのレジスタが読み書き可能かを指定する方法がありません。
3. プログラム内のどのコードからでもこの構造体を通してハードウェアにアクセスできてしまいます。
4. 最も大事なことは、このコードは実際には動作しないということです…

<!--
Now, the problem is that compilers are clever. If you make two writes to the same piece of RAM, one after the other, the compiler can notice this and just skip the first write entirely. In C, we can mark variables as `volatile` to ensure that every read or write occurs as intended. In Rust, we instead mark the *accesses* as volatile, not the variable.
-->

ここで問題となるのは、コンパイラが賢いということです。
同じRAMに相次いで２回書き込むと、コンパイラはこれに気づき、最初の書き込みを完全にスキップします。
Cでは、全ての読み書きが意図した通りに行われることを保証するために、`volatile`型修飾子を変数につけることができます。
Rustでは、変数ではなく*アクセス*に対してvolatileをつけます。

```rust,ignore
let systick = unsafe { &mut *(0xE000_E010 as *mut SysTick) };
let time = unsafe { std::ptr::read_volatile(&mut systick.cvr) };
```

<!--
So, we've fixed one of our four problems, but now we have even more `unsafe` code! Fortunately, there's a third party crate which can help - [`volatile_register`].
-->

これで4つの問題のうち1つを直せました。しかし、さらに`unafe`なコードがあります！
幸いなことに、これに対処できるサードパーティ製のクレート[`volatile_register`]があります。

[`volatile_register`]: https://crates.io/crates/volatile_register

```rust,ignore
use volatile_register::{RW, RO};

#[repr(C)]
struct SysTick {
    pub csr: RW<u32>,
    pub rvr: RW<u32>,
    pub cvr: RW<u32>,
    pub calib: RO<u32>,
}

fn get_systick() -> &'static mut SysTick {
    unsafe { &mut *(0xE000_E010 as *mut SysTick) }
}

fn get_time() -> u32 {
    let systick = get_systick();
    systick.cvr.read()
}
```

<!--
Now, the volatile accesses are performed automatically through the `read` and `write` methods. It's still `unsafe` to perform writes, but to be fair, hardware is a bunch of mutable state and there's no way for the compiler to know whether these writes are actually safe, so this is a good default position.
-->

これで`read`と`write`メソッドを通してvolatileアクセスが自動的に行われるようになりました。
書き込みを実行するのはまだ`unsafe`ですが、公平であるために、ハードウェアは変更可能な状態の集まりであり、コンパイラはそれらの書き込みが実際に安全なのかどうかを知る方法はありません。そのため、これは基本姿勢として悪くないでしょう。

<!-- ## The Rusty Wrapper -->

## Rustのラッパー

<!--
We need to wrap this `struct` up into a higher-layer API that is safe for our users to call. As the driver author, we manually verify the unsafe code is correct, and then present a safe API for our users so they don't have to worry about it (provided they trust us to get it right!).
-->

ユーザが安全に呼び出せるように、この`struct`を高レイヤーのAPIでラップする必要があります。
ドライバの作者として、危険なコードが正しいことを手動で検証し、ユーザがそのドライバを使用する上で心配することがないように安全なAPIとして提供します。（ユーザは提供されたものが正しいと信頼しています！）

<!--
One example might be:
-->

一例を挙げます。

```rust,ignore
use volatile_register::{RW, RO};

pub struct SystemTimer {
    p: &'static mut RegisterBlock
}

#[repr(C)]
struct RegisterBlock {
    pub csr: RW<u32>,
    pub rvr: RW<u32>,
    pub cvr: RW<u32>,
    pub calib: RO<u32>,
}

impl SystemTimer {
    pub fn new() -> SystemTimer {
        SystemTimer {
            p: unsafe { &mut *(0xE000_E010 as *mut RegisterBlock) }
        }
    }

    pub fn get_time(&self) -> u32 {
        self.p.cvr.read()
    }

    pub fn set_reload(&mut self, reload_value: u32) {
        unsafe { self.p.rvr.write(reload_value) }
    }
}

pub fn example_usage() -> String {
    let mut st = SystemTimer::new();
    st.set_reload(0x00FF_FFFF);
    format!("Time is now 0x{:08x}", st.get_time())
}
```

<!--
Now, the problem with this approach is that the following code is perfectly acceptable to the compiler:
-->

このやり方の問題は、次のコードがコンパイラに完全に受け入れられることです。

```rust,ignore
fn thread1() {
    let mut st = SystemTimer::new();
    st.set_reload(2000);
}

fn thread2() {
    let mut st = SystemTimer::new();
    st.set_reload(1000);
}
```

<!--
Our `&mut self` argument to the `set_reload` function checks that there are no other references to *that* particular `SystemTimer` struct, but they don't stop the user creating a second `SystemTimer` which points to the exact same peripheral! Code written in this fashion will work if the author is diligent enough to spot all of these 'duplicate' driver instances, but once the code is spread out over multiple modules, drivers, developers, and days, it gets easier and easier to make these kinds of mistakes.
-->

`set_reload`関数に`&mut self`引数を渡すことで、*その*インスタンスの`SystemTimer`構造体に対する他の参照がないことを確認しますが、全く同じペリフェラルを指す２つ目の`SystemTimer`構造体をユーザが作ることは止められません！
このように書かれたコードは、作者がこれらの「重複した」ドライバのインスタンスを全て見つけるのに十分に熱心であれば動作するでしょうが、一度コードが複数のモジュール、ドライバ、開発者、そして日に渡って分散すると、この種の間違いがどんどん入り込みやすくなっていきます。
