---
title: "Chiselを使ってTang NanoでLチカする"
date: "2020-12-30"
categories: ["FPGA"]
tags: ["FPGA", "Tang Nano ", "Chisel"]
cover: &cover "/images/2020-12-30-Top.jpeg"
slug: "ChiselWithTangNanoLED"
share_img: *cover
mathjax: true
---

## はじめに
---
Chiselを用いてTang NanoというFPGAでLチカしてみたという話です. 

## Chiselとは
---
**Chisel(Constructing Hardware in a Scala Embedded Language)**はデジタル回路設計用の**ハードウェアデザイン言語(Hardware Design Language)**(いわゆるVerilogとかをさす**Hardware Design Language**とは違うので注意)です.

Chiselの存在はGoogleがEdge TPUをChiselで開発したという話を聞いて知りました.
{{< youtube x85342Cny8c >}}

FPGAなどの上でデジタル回路設計を行うときはVerilogなどのHDLを用い行うのは広く知られた開発手法ですが, これらの言語は特定のタスクを行うためだけに開発された**DSL(Domain Specific Language)**なので

- NOWでヤングな言語の遺産を利用できない.
- 抽象度を上げづらく, コードがぐちゃぐちゃになりがち.
- そのため, 大規模なハードウェアを設計するときに言語として貧弱.

といった問題があります.

これらの問題点を解決し, ハードウェアの仕様を記述するためではなく, デザインするために作られた言語がChiselになります.(多分)

ChiselはScalaの組み込みDSLとして作られているため, ScalaというNOWでヤングな言語のパワーを使いながら豊富な機能(単体テストや非同期リセット)を用いることで複雑な回路の開発を助けてくれます. 嬉しいね.

Chiselがどんなものかもっと知りたい人は公式HPを見るのがおすすめです. https://www.chisel-lang.org/

また, Rocket CoreやBOOMなどに代表されるRISC-VのCPUコア実装がChiselで書かれています. デジタル回路設計のシーンがChiselを中心に少しずつ動いてる気配がするのでこれからチェックしていきたいです.
## Tang Nanoとは
---
Sipeed社が出しているFPGAボードです. GW1N-1というチップが載っています. 大きな特徴は以下の通り.

- 500円ちょっとで買える.
- 機能は少ないが, ちょっといじるのにちょうど良い.(論理合成もさくっと終わる)
- LCDディスプレイを繋げられる.

Seeedで買えるのでどうぞ.(https://jp.seeedstudio.com/Sipeed-Tang-Nano-FPGA-board-powered-by-GW1N-1-FPGA-p-4304.html)

![Images](/images/2020-12-30-SipeedTangNano.png)

## 開発環境構築
---
次に開発環境を構築していきます.
### 筆者の開発環境
私はWindows + VSCode + WSL2で諸々の開発を行いました. ただ, WSL2はUSBを2020/12/30現在まだサポートしていないので, 論理合成やTang-Nanoへの書き込みはWindowsのローカルに**GOWIN**というSipeedが出している開発環境を入れました.
### Chisel
GitHubのページに沿って開発環境を構築します.(https://github.com/chipsalliance/chisel3/blob/master/SETUP.md)
Dockerが使えるならそれもいいと思います.

#### テンプレート
開発に当たってテンプレートとして公式が出しているものを利用しました.(https://github.com/freechipsproject/chisel-template)

ちなみに私のコードはこちらにおいてあります. https://github.com/dangorogoro/Tang-Nano-With-Chisel
### Tang-Nano(GOWIN)
公式ドキュメントに沿って開発環境を構築します.(http://tangnano.sipeed.com/en/get_started/install-the-ide.html)

ライセンスは2種類あって

1. Stand-Aloneライセンス
2. ネットワークアクティベーションライセンス

がありますが, 前者は**MACアドレスを書き換えてライセンスファイルを導入する**というあまりに迫力のある方法だったので2番目の方法で行いました. こっちがオススメです.

## 実際に書いていく
---
実際に書いていきます. まずはこんな感じのフォルダー構成を作ります.
``` 
├── README.md
├── build.sbt
├── project
│   ├── build.properties
│   └── plugins.sbt
├── scalastyle-config.xml
├── scalastyle-test-config.xml
└── src
    ├── main
    │   └── scala
    │       └── led
    │           └── LED.scala
    └── test
        └── scala
            └── led
                ├── LEDMain.scala
                └── LEDTest.scala
```
Chiselのチュートリアルは公式に用意されているのでそちらもご覧ください. (https://mybinder.org/v2/gh/freechipsproject/chisel-bootcamp/master)

### Class
まずはメインとなるLチカのコードの方を書いていきます.
{{< gist dangorogoro 02f1703a16829eaff188c50601680930 >}}

それぞれについて見ていきます. まずはIOピンの設定です. Tang Nanoの上にはフルカラーLEDが実装されているので, 三本分のピンを出力として出します.

次にクロックのパルスを使ってカウントアップして最大値になると0になるアップカウンタを作ります. コード中の**count_sec**に当たります. 引数として与えたものをカウンタの最大値として設定します.
カウンタの値が0になるたびにLEDの色が遷移していくという感じです.

さて, これでLEDを光らせる準備ができました. しかし, このままでは問題があります. というのもChiselではHighでリセットになるVerilogコードが出力されますが, Tang Nanoのリセットはアクティブローです.
そのため, リセットがアクティブローなコードが出力されるよう, ラッパーを書いてあげる必要があります.

それが**LEDWrapper**になります. **LEDTop**のモジュールでは**Module**をextendsして使っていました. 実はこのとき暗黙的に**Clock**と**Reset**のピンが設定されているのです.

これでは設定ができなくてちょっと困るので**RawModule**を使うことで明示的に**Clock**や**Reset**を使えるようにします. 33行目に**withClockAndReset**にてアクティブローになるようにインスタンスを作成します.
このとき, 24 * 1000 * 1000と設定しているのはTang Nanoの上に載ってるクロックの値をアップカウンタの最大値として使っているという意味です.

### テスト
これでモジュールを作成することができました. 早速, Verilogコードを生成してと行きたいところですが本当にこのままで動くの確認するためにテストコードを書く必要があります. それがLEDTest.scalaになります.
{{< gist dangorogoro f211d33d377622dcfa5461ad5e8b1377 >}}

これはChiselのiotestersというモジュールを使って実現しています. 実際のクロックは24MHzですがそんなステップを踏んで検証しても無駄なので, 引数でクロックを2Hzに設定して検証しました.
コード中の**step(2)**によってfor文によって毎回2ステップずつ進み, **expect**でLEDの値が正しく想定していた通りに動いてるか検証しているという感じです.

それでは早速実行してみましょう. こんな感じでテストを走らせます.
```bash
$sbt 'test:runMain led.LEDTest'
```
すると, エラーなくテストが通るはずです.

さて, これでテストが通ったので最後にVerilogを出力するところをLEDMain.scalaに書いていきます.

{{< gist dangorogoro 2a4ca023e122472b874f1025a8d00765 >}}
ポイントは引数にArrayを指定することで出力先のディレクトリを**out**に設定しています.

最後に以下のコマンドを走らせると**out**以下にVerilogコードが生成されます.
```bash
$sbt 'test:runMain led.LEDMain'
```

Verilogコードは機械的に生成されているので見てもウーン分からん！となりますが, このコードをボードの開発環境にコピペしていきましょう.

### Tang Nanoへの書き込み
GOWINを使ったココらへんの処理はいろんな方が書いてますので割愛します. それぞれのピン設定だけ明記しておくと
{{< my_table class = "simple-table" >}}
Port | Pin | Description  
  --- | --- | ---
io.led[2] | Pin18 |  LED_RED    
io.led[1] | Pin17 |  LED_BLUE   
io.led[0] | Pin16 |  LED_GREEN  
io.clock  | Pin35 | Clock      
io.reset  | Pin15 | Reset      

{{< /my_table >}}
になります. 合成が終わったら書き込むと以下のようなLチカが見られると思います. Lチカが見られると思います.
{{<tweet user="dango_bot" id="1318575521173655552">}}

これでHDLに触らず, FPGAでLチカできました. やったね.
## Chiselを使った感想
---
Chiselを使ってていいなと思ったとこや苦労したことについて書きます. 皆さんの判断材料になれば幸いです.

### 嬉しいとこ
#### Scalaの上でデジタル回路設計ができる
これが一番嬉しいところだと思います. モダンな言語に付随する様々な機能を使って開発ができるので, 泥沼なソースコードを書かなくてはいけないといったことを回避しつつ, 抽象度を上げてモジュールやモジュール同士の処理を書くことができるのが嬉しいです.
#### テストの機能が標準である
単体テストがすごく簡単に書けるので開発のスピードがアップしたように思います.
### 苦労したこと
#### 2つのパラダイムが詰め込まれている
いわゆる計算機的な上から順に処理をしていく箇所と回路的に同時に処理が発生する箇所の2つのパラダイムを意識して書いていく必要があるので時折, 戸惑いました.

例えば, マイコンのプログラムを書くときに反転の処理を書こうとすると
```c
pin = !pin;
```
となります. これをいつもの癖としてChiselでも同様なコードを書くとエラーになります. というのもこれは**そのピンの反転出力をピンの出力とする**という処理になってしまうからです.(回路図を書く目線になると理解しやすいかも)

そのため, これらを意識しながら書く必要があるので慣れるまで結構戸惑いました.

## まとめ
---
Chiselを使ってHDLを触らずにFPGAでLチカができました. 上述のGoogleの発表でもChiselを習得するには

1. Scalaに慣れ親しむ
2. Chiselに慣れ親しむ
3. ScalaとChiselの機能を使いこなす

という3段回のステップを踏む必要があるという話があり, 習得まで時間を要するみたいです. それでも, Chiselを利用して受けられる恩恵は非常に大きいので皆さんもぜひ使ってみてください.

今度はLCDを光らせてみたいかなと思います.