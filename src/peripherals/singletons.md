<!-- # Singletons -->

# シングルトン

<!--
> In software engineering, the singleton pattern is a software design pattern that restricts the instantiation of a class to one object.
>
> *Wikipedia: [Singleton Pattern]*
-->

> ソフトウェア工学において、シングルトン・パターンはクラスのインスタンス化を１つのオブジェクトに制限するデザインパターンです。
>
> *Wikipedia: [Singleton Pattern]*

[Singleton Pattern]: https://en.wikipedia.org/wiki/Singleton_pattern


<!-- ## But why can't we just use global variable(s)? -->

## なぜグローバル変数は使えないのか？

<!--
We could make everything a public static, like this
-->

このように、全てをパブリックかつスタティックにすることができます。

```rust
static mut THE_SERIAL_PORT: SerialPort = SerialPort;

fn main() {
    let _ = unsafe {
        THE_SERIAL_PORT.read_speed();
    }
}
```

<!--
But this has a few problems. It is a mutable global variable, and in Rust, these are always unsafe to interact with. These variables are also visible across your whole program, which means the borrow checker is unable to help you track references and ownership of these variables.
-->

しかし、これにはいくつか問題があります。
これはミュータブルなグローバル変数であり、Rustにおいては、これらとやり取りするのは常にアンセーフです。
これらの変数はプログラムの全体を通して見えることになり、つまりそれは借用チェッカがこれらの変数の参照や所有権を追跡するのに役立たなくなることを意味します。

<!-- ## How do we do this in Rust? -->

## Rustではどうするか？

<!--
Instead of just making our peripheral a global variable, we might instead decide to make a global variable, in this case called `PERIPHERALS`, which contains an `Option<T>` for each of our peripherals.
-->

単にペリフェラルをグローバル変数にする代わりに、各ペリフェラル毎に`Option<T>`を含む`PERIPHERALS`と呼ばれるグローバル変数を作ることにします。

```rust,ignore
struct Peripherals {
    serial: Option<SerialPort>,
}
impl Peripherals {
    fn take_serial(&mut self) -> SerialPort {
        let p = replace(&mut self.serial, None);
        p.unwrap()
    }
}
static mut PERIPHERALS: Peripherals = Peripherals {
    serial: Some(SerialPort),
};
```

<!--
This structure allows us to obtain a single instance of our peripheral. If we try to call `take_serial()` more than once, our code will panic!
-->

この構造体によって、ペリフェラルの唯一のインスタンスが取得できるようになります。
もしも`take_serial()`を複数回呼び出そうとすれば、コードはパニックするでしょう。

```rust
fn main() {
    let serial_1 = unsafe { PERIPHERALS.take_serial() };
#    // This panics!
    // これはパニックします！
    // let serial_2 = unsafe { PERIPHERALS.take_serial() };
}
```

<!--
Although interacting with this structure is `unsafe`, once we have the `SerialPort` it contained, we no longer need to use `unsafe`, or the `PERIPHERALS` structure at all.
-->

この構造体とのやり取りは`unsafe`にはなりますが、一度この構造体に含まれる`SerialPort`を取得してしまえばもう`unsafe`や`PERIPHERALS`構造体を使う必要は全くありません。

<!--
This has a small runtime overhead because we must wrap the `SerialPort` structure in an option, and we'll need to call `take_serial()` once, however this small up-front cost allows us to leverage the borrow checker throughout the rest of our program.
-->

この方法では、オプション型の中に`SerialPort`構造体をラップし、`take_serial()`を一度コールする必要があるため、実行時に小さなオーバーヘッドとなります。
しかし、この少々のコストを前払いすることで、残りのプログラムで借用チェッカを利用できるようになるのです。

<!-- ## Existing library support -->

## 既存のライブラリによるサポート

<!-- Although we created our own `Peripherals` structure above, it is not necessary to do this for your code. the `cortex_m` crate contains a macro called `singleton!()` that will perform this action for you. -->

上記のコードでは`Peripherals`構造体を作りましたが、あなたのコードでも同じようにする必要はありません。
`cortex_m`クレートはこれと同様のことをしてくれる`singleton!()`と呼ばれるマクロを含んでいます。

```rust
#[macro_use(singleton)]
extern crate cortex_m;

fn main() {
#    // OK if `main` is executed only once,
    // `main`が一度だけしか実行されなければOKです
    let x: &'static mut bool =
        singleton!(: bool = false).unwrap();
}
```

[cortex_m docs](https://docs.rs/cortex-m/latest/cortex_m/macro.singleton.html)

<!--
Additionally, if you use `cortex-m-rtfm`, the entire process of defining and obtaining these peripherals are abstracted for you, and you are instead handed a `Peripherals` structure that contains a non-`Option<T>` version of all of the items you define.
-->

加えて、あなたが`cortex-m-rtfm`クレートを使うのなら、これらのペリフェラルを定義・取得するプロセス全体は抽象化され、代わりにあなたが定義した全てのアイテムの`Option<T>`以外のバージョンを含む`Peripherals`構造体を手渡されます。

```rust,ignore
// cortex-m-rtfm v0.3.x
app! {
    resources: {
        static RX: Rx<USART1>;
        static TX: Tx<USART1>;
    }
}
fn init(p: init::Peripherals) -> init::LateResources {
#    // Note that this is now an owned value, not a reference
    // これは所有された値であり、参照ではないことに注意してください
    let usart1: USART1 = p.device.USART1;
}
```

[japaric.io rtfm v3](https://blog.japaric.io/rtfm-v3/)

<!-- ## But why? -->

## しかしなぜ？

<!--
But how do these Singletons make a noticeable difference in how our Rust code works?
-->

しかし、これらのシングルトンがRustのコードの動作にどのような顕著な違いをもたらすのでしょうか？

```rust,ignore
impl SerialPort {
    const SER_PORT_SPEED_REG: *mut u32 = 0x4000_1000 as _;

    fn read_speed(
#        &self // <------ This is really, really important
        &self // <------ これは本当に、本当に重要です。
    ) -> u32 {
        unsafe {
            ptr::read_volatile(Self::SER_PORT_SPEED_REG)
        }
    }
}
```

<!--
There are two important factors in play here:
-->

ここには２つの重要な要素があります。

<!--
* Because we are using a singleton, there is only one way or place to obtain a `SerialPort` structure
* To call the `read_speed()` method, we must have ownership or a reference to a `SerialPort` structure
-->

* シングルトンを使用しているため、`SerialPort`構造体の取得手段や場所は１つだけになります。
* `read_speed()`メソッドを呼ぶためには、`SerialPort`構造体の所有権もしくは参照を持つ必要があります。

<!--
These two factors put together means that it is only possible to access the hardware if we have appropriately satisfied the borrow checker, meaning that at no point do we have multiple mutable references to the same hardware!
-->

これらの２つの要素をまとめると、借用チェッカを適切に満たしている場合のみハードウェアにアクセスできることを意味します。つまり、同じハードウェアに対して複数のミュータブルな参照を持つことはありません。

```rust
fn main() {
#    // missing reference to `self`! Won't work.
    // `self`への参照が見つかりません。これはうまく動きません。
    // SerialPort::read_speed();

    let serial_1 = unsafe { PERIPHERALS.take_serial() };

#    // you can only read what you have access to
    // アクセスできるものだけ読み出すことができます。
    let _ = serial_1.read_speed();
}
```

<!-- ## Treat your hardware like data -->

## ハードウェアをデータのように扱う

<!--
Additionally, because some references are mutable, and some are immutable, it becomes possible to see whether a function or method could potentially modify the state of the hardware. For example,
-->

加えて、いくつかの参照はミュータブルで、またいくつかはイミュータブルなため、関数またはメソッドがハードウェアの状態を変更できるかどうかを確認することが可能になります。

<!--
This is allowed to change hardware settings:
-->

この関数はハードウェアの設定を変更できます。

```rust,ignore
fn setup_spi_port(
    spi: &mut SpiPort,
    cs_pin: &mut GpioPin
) -> Result<()> {
    // ...
}
```

<!--
This isn't:
-->

このメソッドは変更できません。

```rust,ignore
fn read_button(gpio: &GpioPin) -> bool {
    // ...
}
```

<!--
This allows us to enforce whether code should or should not make changes to hardware at **compile time**, rather than at runtime. As a note, this generally only works across one application, but for bare metal systems, our software will be compiled into a single application, so this is not usually a restriction.
-->

これにより、実行時ではなく**コンパイル時に**コードがハードウェアを変更するかどうかを強制できます。
注意点としては、通常この強制はひとつのアプリケーション内でのみ機能します。ベアメタルシステムにおいては、ソフトウェアはひとつのアプリケーションにコンパイルされるため機能しますが、これは一般的な制約ではありません。
