---
title: "Chiselを使ってTang NanoでLCDを光らせる"
date: "2021-05-02"
categories: ["FPGA"]
tags: ["FPGA", "Tang Nano ", "Chisel"]
cover: &cover "/images/2021-05-02-LCD.jpeg"
slug: "ChiselWithTangNanoLCD"
share_img: *cover
mathjax: true
---
## 概要
---
Chiselを使ってTang Nano付属のLCDを光らせました. メモを残します.

{{<tweet user="dango_bot" id="1326484313508962304">}}

## はじめに
---
Chiselはデジタル回路設計用のハードウェアデザイン言語です. 以前チュートリアルを書いたので, そちらをご覧ください. 
[Chiselを使ってTang NanoでLチカする](/posts/2020-12-30-ChiselWithTangNanoLED.md)

LCDはTang Nano FPGA開発用として[Sipeed](https://jp.seeedstudio.com/5-Inch-Display-for-Sipeed-Tang-Nanno-p-4301.html)で売られていたものを使いました. [秋月](https://akizukidenshi.com/catalog/g/gM-14873/)でも売られています.

## 設定
---
まず, 最初にSeeed公式がexampleコードを公開しているので, それを見てみました.

https://github.com/sipeed/Tang-Nano-examples

LCDに関するドキュメントも公開されていたのでそちらも確認しました.

https://tangnano.sipeed.com/en/examples/2_lcd.html

VGAを使って画面の制御を行っているので, まずはVGAの信号をChiselで実装しました.

### VGAの設定
VGAでは水平同期信号(HSYNC)と垂直同期信号(VSYNC)のタイミングに合わせてピクセルの値をRGBで設定します.

今回は上のドキュメントやコードを参考にタイミングやパラメータなどを合わせました.
{{<gist dangorogoro a6bfb9566a5b4d72124561afce301530>}}

Verilogで書かれたコードをChiselに変換するしてるだけなので, 特に説明する箇所はないですが, ポイントとして
- Mux(マルチプレクサ)を使って範囲外のピクセル更新の操作をOFFにする.
- 現在, 更新する座標とRGBの信号線を外に出して, 上位のモジュールに渡す.

という感じです.

### LCDのクロック設定(BlackBoxの使用)
次にさっそくLCDの描画を行いたいところですが, LCDにはクロックとして(上記の設定では)33.33MHzを供給する必要があります.

これはTang NanoのIPを利用することで生成できますがChiselからではそういったコードを生成することはできません.(それはそう)

このように**特定のVerilogのコードをChiselのプロジェクトで使用したい**といったときに使用するのがBlackBox機能です.

これを用いることでVerilogで書かれたコードを隠蔽しつつ, インターフェイスをChisel側に渡すことができます.

まず, IPを用いて以下のようなコードを生成しました. Gowinの生成したコードがなんかいまいちだったのでこちらのコードを参考にしました。

https://github.com/dotcypress/tang-nano-lcd

{{<gist dangorogoro 2ae3745c83cc5842eb57b88030756edc>}}

そしてChisel側でこのようなコードを書いてVerilogのコードを利用します. 入出力のピンの名前を同じにする必要があるのでそこだけ注意です.

{{<gist dangorogoro e875b92b6492a92de5c0430b407362d8>}}

### LCDの描画
先ほど作成したVGAModuleから得られた座標とRGBのピンを用いてLCDの描画を行っていきます。
今回は単純に市松模様を描画しました。
コードはこんな感じです. (条件分岐はかなり適当です.

{{<gist dangorogoro 3cf285366323497a82969a068d847a6e>}}

ポイントとして
- LCDもFPGAもアクティブローなので, Wrapperを作成し, **withClockAndReset**で初期化する.
です.

今回の処理ではめんどくさくて下位モジュールと上位モジュールの接続を **:=** で行いましたがバルク接続を使うともっとすっきりします.
(ただ, その場合モジュールのピンを調整しないといけない...)

最後にmainの処理を書いてビルドします.
{{<gist dangorogoro f8ccf4a55e3f755b28180fde940ac545>}}

以下のようなコマンドを実行すると**out/** 以下にverilogコードが生成されてると思います. 今回はBlackBoxを使ったので, そのコードも同一ディレクトリーに生成されているのがポイントです.
```
$sbt 'test:runMain lcd.LCDMain'
```
今回作成したコードはGitHubに置いてるので必要があれば見てください.

https://github.com/dangorogoro/Tang-Nano-With-Chisel
### ピン設定と書き込み
生成したファイル(LCDTopWrapper.vとrpll.v)をGowinに持って行ってビルドします.

ピン配置は以下のように設定しました.

{{< my_table class = "simple-table" >}}
pin | location|
--- | --- |
io_led_b | 17 |
io_led_g | 16 |
io_led_r | 18 |
io_nrst | 15 |
io_vga_b | 41-45 |
io_vga_clk | 11 |
io_vga_de | 5 |
io_vga_g | 32-34, 38-40 |
io_vga_hsync | 10 |
io_vga_nrst | 14 |
io_vga_g | 27-31 |
io_vga_vsync | 46 |
io_xtal | 35 |
{{< /my_table>}}

以上の設定をしてビットストリームを作成して書き込むとこんな感じの模様が出てきます.

{{<tweet user="dango_bot" id="1326484313508962304">}}

## おわりに
Chiselを使ってTang Nanoに繋げたLCDへ市松模様を表示できました. ここには書いてませんが, 開発中はChisel側でテストを書いてデバッグしていたのでとてもスムーズにできました. 今後もいじっていきたいですね.