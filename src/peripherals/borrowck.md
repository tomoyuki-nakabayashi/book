<!-- ## Mutable Global State -->

## ミュータブルでグローバルな状態

<!--
Unfortunately, hardware is basically nothing but mutable global state, which can feel very frightening for a Rust developer. Hardware exists independently from the structures of the code we write, and can be modified at any time by the real world.
-->

残念ながら、ハードウェアは基本的にミュータブルでグローバルな状態に他なりません。これはRustの開発者にとって非常に恐ろしいことです。
ハードウェアは書かれたコードの構造とは独立して存在しており、現実の世界からいつでも変更される可能性があります。

<!-- ## What should our rules be? -->

## 何をルールとするべきか？

<!--
How can we reliably interact with these peripherals?
-->

どうすればこれらのペリフェラルと確実にやり取りできるのでしょう？

<!--
1. Always use `volatile` methods to read or write to peripheral memory, as it can change at any time
2. In software, we should be able to share any number of read-only accesses to these peripherals
3. If some software should have read-write access to a peripheral, it should hold the only reference to that peripheral
-->

1. いつ変化するかわからないペリフェラルメモリの読み書きには、常に`volatile`メソッドを使用してください
2. ソフトウェアでは、これらのペリフェラルへの読み取り専用アクセスをいくつでも共有できるでしょう
3. あるソフトウェアがあるペリフェラルに読み書きのアクセスをするならば、そのソフトウェアは、ペリフェラルへの唯一の参照を持つべきです

<!-- ## The Borrow Checker -->

## 借用チェッカ

<!--
The last two of these rules sound suspiciously similar to what the Borrow Checker does already!
-->

さきほどのルールの最後の２つは、借用チェッカが行っていることに怪しいくらいよく似ています。

<!--
Imagine if we could pass around ownership of these peripherals, or offer immutable or mutable references to them?
-->

ペリフェラルの所有権を譲渡したり、ペリフェラルへのイミュータブルまたはミュータブルな参照を提供したりできるのか、を想像してみてください。

<!--
Well, we can, but for the Borrow Checker, we need to have exactly one instance of each peripheral, so Rust can handle this correctly. Well, luckliy in the hardware, there is only one instance of any given peripheral, but how can we expose that in the structure of our code?
-->

それは可能です。しかし借用チェッカを通すためには、各ペリフェラルに対して唯一のインスタンスを持つ必要があります。そうすれば、Rustはその唯一のインスタンスを正しく扱えます。
幸いなことにハードウェアにおいて、任意のペリフェラルのインスタンスは１つだけしかありません。しかし、どうすればそれをコードの構造として明確にできるでしょうか？
