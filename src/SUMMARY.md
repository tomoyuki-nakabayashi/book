# Summary

<!--

Definition of the organization of this book is still a work in process.

Refer to https://github.com/rust-embedded/book/issues for
more information and coordination

-->

- [導入](./intro/index.md)
    - [`no_std`](./intro/no-std.md)
    - [ツール](./intro/tooling.md)
    - [インストール](./intro/install.md)
        - [Linux](./intro/install/linux.md)
        - [MacOS](./intro/install/macos.md)
        - [Windows](./intro/install/windows.md)
        - [インストールの確認](./intro/install/verify.md)
    - [ハードウェア](./intro/hardware.md)
- [入門](./start/index.md)
  - [QEMU](./start/qemu.md)
  - [ハードウェア](./start/hardware.md)
  - [メモリマップドレジスタ](./start/registers.md)
  - [セミホスティング](./start/semihosting.md)
  - [パニック](./start/panicking.md)
  - [例外](./start/exceptions.md)
  - [割り込み](./start/interrupts.md)
  - [IO](./start/io.md)
- [ペリフェラル](./peripherals/index.md)
    - [Rustでの最初の試み](./peripherals/a-first-attempt.md)
    - [借用チェッカ](./peripherals/borrowck.md)
    - [シングルトン](./peripherals/singletons.md)
- [静的な保証](./static-guarantees/index.md)
    - [型状態プログラミング](./static-guarantees/typestate-programming.md)
    - [ステートマシンとしてのペリフェラル](./static-guarantees/state-machines.md)
    - [設計契約](./static-guarantees/design-contracts.md)
    - [ゼロコスト抽象化](./static-guarantees/zero-cost-abstractions.md)
- [移植性](./portability/index.md)
- [並行性](./concurrency/index.md)
- [コレクション](./collections/index.md)
- [組込みC開発者へのヒント](./c-tips/index.md)
    <!-- TODO: Define Sections -->
- [相互運用性](./interoperability/index.md)
    - [Rustと少しのC](./interoperability/c-with-rust.md)
    - [Cと少しのRust](./interoperability/rust-with-c.md)
- [未分類のトピック](./unsorted/index.md)
  - [最適化: 速度とサイズのトレードオフ](./unsorted/speed-vs-size.md)
