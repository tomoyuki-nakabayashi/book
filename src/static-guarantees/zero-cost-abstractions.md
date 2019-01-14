<!-- # Zero Cost Abstractions -->

# ゼロコスト抽象化

<!--
Type states are also an excellent example of Zero Cost Abstractions - the ability to move certain behaviors to compile time execution or analysis. These type states contain no actual data, and are instead used as markers. Since they contain no data, they have no actual representation in memory at runtime:
-->

型状態はゼロコスト抽象化の優れた例でもあります。特定の動作を、コンパイル時の実行もしくは解析に移動する機能です。
これらの型状態は、実際のデータを含んでおらず、代わりにマーカとして使われています。
型状態は、データを含んでいないため、実行時にメモリに実際のデータはありません。

```rust,ignore
use core::mem::size_of;

let _ = size_of::<Enabled>();    // == 0
let _ = size_of::<Input>();      // == 0
let _ = size_of::<PulledHigh>(); // == 0
let _ = size_of::<GpioConfig<Enabled, Input, PulledHigh>>(); // == 0
```

<!-- ## Zero Sized Types -->

## ゼロサイズの型

```rust,ignore
struct Enabled;
```

<!--
Structures defined like this are called Zero Sized Types, as they contain no actual data. Although these types act "real" at compile time - you can copy them, move them, take references to them, etc., however the optimizer will completely strip them away.
-->

上記のように定義された構造体をゼロサイズの型、と呼びます。これは、実際のデータを含んでいません。
これらの型は、コンパイル時には「実際に」機能します。例えば、コピーも、ムーブも、参照を取ることもできます。
しかし、最適化はこれらを完全に取り除きます。

<!-- In this snippet of code: -->

次のコードスニペットを見てください。

```rust,ignore
pub fn into_input_high_z(self) -> GpioConfig<Enabled, Input, HighZ> {
    self.periph.modify(|_r, w| w.input_mode().high_z());
    GpioConfig {
        periph: self.periph,
        enabled: Enabled,
        direction: Input,
        mode: HighZ,
    }
}
```

<!--
The GpioConfig we return never exists at runtime. Calling this function will generally boil down to a single assembly instruction - storing a constant register value to a register location. This means that the type state interface we've developed is a zero cost abstraction - it uses no more CPU, RAM, or code space tracking the state of `GpioConfig`, and renders to the same machine code as a direct register access.
-->

実行時、返り値のGpioConfigは、存在しません。この関数を呼び出すと、通常、1つのアセンブリ命令にまとめられます。
そのアセンブリ命令は、定数のレジスタ値を、レジスタの位置へ格納します。
これは、開発した型状態インタフェースが、ゼロコスト抽象化であることを意味します。
`GpioConfig`の状態を追跡するために、余分なCPU、RAM、コード領域を使用せず、直接レジスタアクセスするのと同じ機械語を表します。

<!-- ## Nesting -->

## ネスト

<!--
In general, these abstractions may be nested as deeply as you would like. As long as all components used are zero sized types, the whole structure will not exist at runtime.
-->

通常、これらの抽象化は、望みのまま深さでネストされます。使用される全てのコンポーネントが、ゼロサイズ型である限り、実行時には、構造体全体が存在しません。

<!--
For complex or deeply nested structures, it may be tedious to define all possible combinations of state. In these cases, macros may be used to generate all implementations.
-->

複雑な構造体や深くネストした構造体については、全てのあり得る状態の組み合わせを定義することは、面倒です。
このような場合、全ての実装を生成するために、マクロが利用できます。
