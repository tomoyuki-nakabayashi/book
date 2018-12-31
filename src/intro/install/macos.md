# macOS

<!-- All the tools can be install using [Homebrew]: -->

全てのツールは、[Homebrew]を使ってインストールできます。

[Homebrew]: http://brew.sh/

``` console
$ # GDB
$ brew cask install gcc-arm-embedded

$ brew install openocd

$ brew install qemu
```
<!-- 
If the `brew cask` command doesn't work (e.g. `error: unknown command: cask`),
then first run `brew tap Caskroom/tap` and try again.
 -->

`brew cask`コマンドがうまく動かない場合(例えば、`error: unknown command: cask`)、最初に`brew tap Caskroom/tap`を実行してから再実行して下さい。

<!-- That's all! Go to the [next section]. -->

以上です！[次のセクション]に進んで下さい。

<!-- [next section]: verify.md -->

[次のセクション]: verify.md
