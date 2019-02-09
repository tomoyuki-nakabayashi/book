<!-- # Interrupts -->

# 割り込み

<!--
Interrupts differ from exceptions in a variety of ways but their operation and
use is largely similar and they are also handled by the same interrupt
controller. Whereas exceptions are defined by the Cortex-M architecture,
interrupts are always vendor (and often even chip) specific implementations,
both in naming and functionality.
-->

割り込みは様々な点で例外と違いますが、その動作と使用方法は、ほとんど同じで、同じ割り込みコントローラによって処理されます。
例外がCortex-Mアーキテクチャで定義されているのに対し、割り込みは、命名と機能との両方において、常にベンダ（もっと言うとチップ）固有の実装です。

<<<<<<< HEAD
<!--
Interrupts do allow for a lot of flexbility which needs to be accounted for
=======
Interrupts do allow for a lot of flexibility which needs to be accounted for
>>>>>>> upstream
when attempting to use them in an advanced way. We will not cover those uses in
this book, however it is a good idea to keep the following in mind:
-->

割り込みは、高度な使い方をしようとする時に必要とされる様々な柔軟性を考慮に入れています。
本書では、そのような高度な使い方は対象外です。しかし、次の点に留意することをお勧めします。

<!-- * Interrupts have programmable priorities which determine their handlers' execution order -->

* 割り込みは、ハンドラの実行順序を決めるプログラム可能な優先度を持ちます。

<!-- * Interrupts can nest and preempt, i.e. execution of an interrupt handler might be interrupted by another higher-priority interrupt -->

* 割り込みは、ネストとプリエンプションが可能です。つまり、割り込みハンドラの実行は、より優先度の高い割り込みに割り込まれる場合があります。

<!-- * In general the reason causing the interrupt to trigger needs to be cleared to prevent re-entering the interrupt handler endlessly -->

* 通常、割り込み要因は、割り込みハンドラが無限に再呼び出しされないようにするため、クリアされる必要があります。

<!--
The general initialization steps at runtime are always the same:
* Setup the peripheral(s) to generate interrupts requests at the desired occasions
* Set the desired priority of the interrupt handler in the interrupt controller
* Enable the interrupt handler in the interrupt controller
-->

ランタイムでの一般的な初期化手順は、常に同じです。
* 必要な時に割り込み要求を起こすように、ペリフェラルを設定します
* 割り込みコントローラで割り込みハンドラを有効化します

<!--
Similarly to exceptions, the `cortex-m-rt` crate provides an [`interrupt`]
attribute to declare interrupt handlers. The available interrupts (and
their position in the interrupt handler table) are usually automatically
generated via `svd2rust` from a SVD description.
-->

例外と同様に、例外ハンドラを宣言するために、`cortex-m-rt`クレートは、[`interrupt`]属性を提供しています。
利用可能な割り込み（そして割り込みハンドラテーブルでの配置）は、通常、`svd2rust`を使ってSVDから自動生成されます。

[`interrupt`]: https://docs.rs/cortex-m-rt-macros/0.1.5/cortex_m_rt_macros/attr.interrupt.html

``` rust,ignore
# // Interrupt handler for the Timer2 interrupt
// タイマ2割り込みの割り込みハンドラ
#[interrupt]
fn TIM2() {
    // ..
#     // Clear reason for the generated interrupt request
    // 発生した割り込み要求の原因をクリアします
}
```

<!--
Interrupt handlers look like plain functions (except for the lack of arguments)
similar to exception handlers. However they can not be called directly by other
parts of the firmware due to the special calling conventions. It is however
possible to generate interrupt requests in software to trigger a diversion to
to the interrupt handler.
-->

割り込みハンドラは、通常の関数のように見え、例外ハンドラに似ています（引数がないことを除いて）。
しかし、割り込みハンドラは、特別な呼び出し規約のため、ファームウェアの他の部分から直接呼び出すことができません。
ソフトウェアで割り込み要求を起こし、割り込みハンドラへの転送を発生させることは可能です。

<<<<<<< HEAD
<!--
Similar to exeption handlers it is also possible to declare `static mut`
=======
Similar to exception handlers it is also possible to declare `static mut`
>>>>>>> upstream
variables inside the interrupt handlers for *safe* state keeping.
-->

例外ハンドラと同様に、割り込みハンドラ内で`static mut`変数を宣言し、状態を*安全*に保持することができます。

``` rust,ignore
#[interrupt]
fn TIM2() {
    static mut COUNT: u32 = 0;

#     // `COUNT` has type `&mut u32` and it's safe to use
    // `COUNT`は`&mut u32`の型を持っており、その使用は安全です
    *COUNT += 1;
}
```

<!--
For a more detailed description about the mechanisms demonstrated here please
refer to the [exceptions section].
-->

ここで示した仕組みの詳細については、[例外セクション]を参照して下さい。

<!-- [exceptions section]: ./exceptions.md -->

[例外セクション]: ./exceptions.md