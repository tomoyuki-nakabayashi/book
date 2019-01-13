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
    - [The Borrow Checker](./peripherals/borrowck.md)
    - [Singletons](./peripherals/singletons.md)
- [静的な保証](./static-guarantees/index.md)
    - [型状態プログラミング](./static-guarantees/typestate-programming.md)
    - [Peripherals as State Machines](./static-guarantees/state-machines.md)
    - [Design Contracts](./static-guarantees/design-contracts.md)
    - [Zero Cost Abstractions](./static-guarantees/zero-cost-abstractions.md)
- [Portability](./portability/index.md)
- [Concurrency](./concurrency/index.md)
- [Collections](./collections/index.md)
- [Tips for embedded C developers](./c-tips/index.md)
    <!-- TODO: Define Sections -->
- [Interoperability](./interoperability/index.md)
    - [A little C with your Rust](./interoperability/c-with-rust.md)
    - [A little Rust with your C](./interoperability/rust-with-c.md)
- [Unsorted topics](./unsorted/index.md)
  - [Optimizations: The speed size tradeoff](./unsorted/speed-vs-size.md)
