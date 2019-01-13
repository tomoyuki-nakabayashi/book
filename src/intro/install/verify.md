<!-- # Verify Installation -->

# インストールの確認

<!-- 
In this section we check that some of the required tools / drivers have been
correctly installed and configured.
 -->

このセクションでは、必要となるツールとドライバが正しくインストールされ、設定されていることを確認します。

<!-- 
Connect your laptop / PC to the discovery board using a micro USB cable. The
discovery board has two USB connectors; use the one labeled "USB ST-LINK" that
sits on the center of the edge of the board.
 -->

マイクロUSBケーブルを使って、ノートPC / PCをdiscoveryボードに接続して下さい。
discoveryボードは2つのUSBコネクタを搭載しています。
ボード端の中央にある"USB ST-LINK"とラベルが付いたものを使用して下さい。

<!-- 
Also check that the ST-LINK header is populated. See the picture below; the
ST-LINK header is circled in red.
 -->

ST-LINKヘッダが装着されていることも確認します。下の写真の赤丸で囲った部分がST-LINKヘッダです。

<p align="center">
<img title="Connected discovery board" src="../../assets/verify.jpeg">
</p>
<!-- 
Now run the following command:
 -->

それでは、次のコマンドを実行して下さい。

``` console
$ openocd -f interface/stlink-v2-1.cfg -f target/stm32f3x.cfg
```

<!-- You should get the following output and the program should block the console: -->

次の出力が得られ、プログラムはコンソールをブロックするはずです。

``` text
Open On-Chip Debugger 0.10.0
Licensed under GNU GPL v2
For bug reports, read
        http://openocd.org/doc/doxygen/bugs.html
Info : auto-selecting first available session transport "hla_swd". To override use 'transport select <transport>'.
adapter speed: 1000 kHz
adapter_nsrst_delay: 100
Info : The selected transport took over low-level target control. The results might differ compared to plain JTAG/SWD
none separate
Info : Unable to match requested speed 1000 kHz, using 950 kHz
Info : Unable to match requested speed 1000 kHz, using 950 kHz
Info : clock speed 950 kHz
Info : STLINK v2 JTAG v27 API v2 SWIM v15 VID 0x0483 PID 0x374B
Info : using stlink api v2
Info : Target voltage: 2.919881
Info : stm32f3x.cpu: hardware has 6 breakpoints, 4 watchpoints
```
<!-- 
The contents may not match exactly but you should get the last line about
breakpoints and watchpoints. If you got it then terminate the OpenOCD process
and move to the [next section].
 -->

確認作業とは直接関係しませんが、ブレイクポイントとウォッチポイントに関する最後の行を取得したはずです。
取得できた場合、OpenOCDプロセスを停止し、[次のセクション]へ進んで下さい。

<!-- [next section]: ../hardware.md -->

[次のセクション]: ../hardware.md

<!-- If you didn't get the "breakpoints" line then try the following command. -->

"breakpoints"の行が取得できなかった場合、次のコマンドを試してく下さい。

``` console
$ openocd -f interface/stlink-v2.cfg -f target/stm32f3x.cfg
```

<!-- 
If that command works that means you got an old hardware revision of the
discovery board. That won't be a problem but commit that fact to memory as
you'll need to configure things a bit differently later on. You can move to the
[next section].
 -->

このコマンドが機能した場合、古いハードウェアリビジョンのdiscoveryボードを入手したことを意味します。
これは問題になりませんが、後で少し設定を変える必要があるので、そのことを覚えておいて下さい。
[次のセクション]に進むことができます。

<!-- 
If neither command worked as a normal user then try to run them with root
permission (e.g. `sudo openocd ..`). If the commands do work with root
permission then check that the [udev rules] has been correctly set.
 -->

どちらのコマンドも通常ユーザとしてうまく動かなかった場合、rootパーミッションで実行してみて下さい(例えば、`sudo openocd ..`)。
コマンドがrootパーミッションで機能した場合、[udevルール]が正しく設定されているか確認して下さい。

<!-- [udev rules]: linux.md#udev-rules -->

[udevルール]: linux.md#udev-rules

<!-- 
If you have reached this point and OpenOCD is not working please open [an issue]
and we'll help you out!
 -->

ここまで到着していまい、OpenOCDが動いていないならば、[issue]を作って下さい。私たちがあなたを支援します。

<!-- [an issue]: https://github.com/rust-embedded/book/issues -->

[issue]: https://github.com/rust-embedded/book/issues
