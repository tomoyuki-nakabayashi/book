<!-- # Installing the tools -->

# ツールのインストール

<!-- This page contains OS-agnostic installation instructions for a few of the tools: -->

このページには、いくつかのツールのOSに依存しないインストール手順を掲載します。

<!-- ### Rust Toolchain -->

### Rustツールチェイン

<!-- Install rustup by following the instructions at [https://rustup.rs](https://rustup.rs). -->

[https://rustup.rs](https://rustup.rs)の手順に従って、rustupをインストールします。

<!-- 
**NOTE** Make sure you have a compiler version equal to or newer than `1.31`. `rustc
-V` should return a date newer than the one shown below.
 -->

**注意** コンパイラのバージョンが`1.31`以上であることを確認して下さい。`rustc -v`は下記に示す日付より新しい日付を返すべきです。

``` console
$ rustc -V
rustc 1.31.1 (b6c32da9b 2018-12-18)
```

<!-- 
For bandwidth and disk usage concerns the default installation only supports
native compilation. To add cross compilation support for the ARM Cortex-M
architecture install the following compilation targets.
 -->

バンド幅とディスク使用量に関する懸念から、デフォルトインストールではネイティブコンパイルのみをサポートします。
ARM Cortex-Mアーキテクチャのクロスコンパイラを追加するために、下記のコンパイルターゲットをインストールします。

``` console
$ rustup target add thumbv6m-none-eabi thumbv7m-none-eabi thumbv7em-none-eabi thumbv7em-none-eabihf
```

### `cargo-binutils`

``` console
$ cargo install cargo-binutils

$ rustup component add llvm-tools-preview
```

<!-- ### OS-Specific Instructions -->

### OS特有の手順

<!-- Now follow the instructions specific to the OS you are using: -->

使用しているOSに特有の手順に従って下さい。

- [Linux](install/linux.md)
- [Windows](install/windows.md)
- [macOS](install/macos.md)
