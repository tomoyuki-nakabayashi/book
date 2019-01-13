<!-- # Typestate Programming -->

# 型状態プログラミング

<!--
The concept of [typestates] describes the encoding of information about the current state of an object into the type of that object. Although this can sound a little arcane, if you have used the [Builder Pattern] in Rust, you have already started using Typestate Programming!
-->

[型状態]の概念は、オブジェクトの現在の状態に関する情報を、そのオブジェクトの型にエンコードすることを説明しています。
これは、少し難解に思えますが、Rustの[ビルダーパターン]を使ったことがあるならば、既に型状態プログラミングを使い始めています。

<!--
[typestates]: https://en.wikipedia.org/wiki/Typestate_analysis
[Builder Pattern]: https://doc.rust-lang.org/1.0.0/style/ownership/builders.html
-->

[型状態]: https://en.wikipedia.org/wiki/Typestate_analysis
[ビルダーパターン]: https://doc.rust-lang.org/1.0.0/style/ownership/builders.html

```rust
#[derive(Debug)]
struct Foo {
    inner: u32,
}

struct FooBuilder {
    a: u32,
    b: u32,
}

impl FooBuilder {
    pub fn new(starter: u32) -> Self {
        Self {
            a: starter,
            b: starter,
        }
    }

    pub fn double_a(self) -> Self {
        Self {
            a: self.a * 2,
            b: self.b,
        }
    }

    pub fn into_foo(self) -> Foo {
        Foo {
            inner: self.a + self.b,
        }
    }
}

fn main() {
    let x = FooBuilder::new(10)
        .double_a()
        .into_foo();

    println!("{:#?}", x);
}
```

<!--
In this example, there is no direct way to create a `Foo` object. We must create a `FooBuilder`, and properly initialize it before we can obtain the `Foo` object we want.
-->

この例では、`Foo`オブジェクトを直接作る方法はありません。必要な`Foo`オブジェクトを取得する前に、`FooBuilder`を作り、正しく初期化しなければなりません。

<!-- This minimal example encodes two states: -->

この最小限の例は、2つの状態をエンコードしています。

<!--
* `FooBuilder`, which represents an "unconfigured", or "configuration in process" state
* `Foo`, which represents a "configured", or "ready to use" state.
-->

* `FooBuilder`は、「未設定」もしくは「設定中」の状態を表現します。
* `Foo`は、「設定済み」もしくは「使用準備完了」の状態を意味します。

<!-- ## Strong Types -->

## 強い型

<!--
Because Rust has a [Strong Type System], there is no easy way to magically create an instance of `Foo`, or to turn a `FooBuilder` into a `Foo` without calling the `into_foo()` method. Additionally, calling the `into_foo()` method consumes the original `FooBuilder` structure, meaning it can not be reused without the creation of a new instance.
-->

Rustは[強い型付けシステム]を持っています。`Foo`のインスタンスを魔法のように作成したり、
`into_foo()`メソッドを呼び出すことなしに`FooBuilder`から`Foo`に変換したりする、簡単な方法はありません。
さらに、`into_foo()`メソッドは、オリジナルの`FooBuilder`構造体を消費します。
これは、新しいインスタンスを作成しないと、再利用ができないことを意味します。

<!-- [Strong Type System]: https://en.wikipedia.org/wiki/Strong_and_weak_typing -->

[強い型付けシステム]: https://en.wikipedia.org/wiki/Strong_and_weak_typing

<!--
This allows us to represent the states of our system as types, and to include the necessary actions for state transitions into the methods that exchange one type for another. By creating a `FooBuilder`, and exchanging it for a `Foo` object, we have walked through the steps of a basic state machine.
-->

このことにより、システムの状態を型として表現することが可能になります。
そして、状態遷移に必要なアクションを、ある型と別の型とを交換するメソッドに取り入れることができます。
`FooBuilder`を作成し、`Foo`オブジェクトに交換することで、基本的なステートマシンのステップを見てきました。