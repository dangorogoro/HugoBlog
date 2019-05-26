---
title: "MATLABでマイクロマウスの機体をシステム同定してPIDチューニングする."
date: "2018-09-09"
categories: ["ロボット"]
cover: "/images/2018-09-22-PIDgain.png"
tags: ["マイクロマウス", "MATLAB"]
slug: :MATLABSystemIdentification"
aliases: "/MATLABSystemIdentification.html
---

# MATLABで マウスの同定 やりました.
表題の通り, MATLABでマイクロマウスの機体(2輪ロボット)の左右のモーターへのステップ応答から車体の伝達関数を出しました.

# はじめに
近年, MathWorks社様がマイクロマウス大会のスポンサーになり, 競技者たちは申請をすることでMATLABが使えるようになりました. これをきっかけにMATLABを用いたロボットのモデリングやシミュレーションなどが盛んに行われるようになるのはここで論ずるまでもないでしょう.
[https://jp.mathworks.com/academia/student-competitions/micromouse-contest.html](https://jp.mathworks.com/academia/student-competitions/micromouse-contest.html)

今回はその一環として, マイクロマウスの機体をモデリングすることで, 簡単に制御に必要な速度コントローラーのPIDゲインを出しました.

## なぜシステム同定をするのか
一般的に移動ロボットを制作する際に, 速度コントローラーを設計します.(移動するので) こんな感じのものを皆さん実装しているのではないでしょうか. 

![loop](/images/2018-09-22-Feedback.png)

そしてこれ(速度コントローラー)を実現するものして, PIDコントローラが存在します. このPIDコントローラーのパラメーターを適切に求めるにはロボットの運動についての数理モデルが必要になります.
よくロボットの制御で聞くPIDゲインの決め方として**台形加速を入力していい感じにゲインを求める**というのがあります.(実際に私がMATLABを使ってモデリングするまではこの方法を使ってやっていました.)

しかし, これは

* 無限に時間をかけてしまう.(人生は何もしないことには無限だが何かをするには有限)
* 今入れたPIDゲインを定性的に評価できない.
* しんどい(しんどい)
などの問題が生じてしまいます.

そこで, システム同定, すなわち制御対象であるロボットのモデルを求めることで自分の設計したコントローラーについて理論的に制御器の安定性や性能評価をPC上で行うことができます.

## 参考文献

足立先生のシステム同定の本を読んでシステム同定について勉強しました.

[システム同定の基礎](https://www.amazon.co.jp/%E3%82%B7%E3%82%B9%E3%83%86%E3%83%A0%E5%90%8C%E5%AE%9A%E3%81%AE%E5%9F%BA%E7%A4%8E-%E8%B6%B3%E7%AB%8B-%E4%BF%AE%E4%B8%80/dp/4501114800/ref=sr_1_fkmr0_1?s=books&ie=UTF8&qid=1536508435&sr=1-1-fkmr0&keywords=%E3%82%B7%E3%82%B9%E3%83%86%E3%83%A0%E5%90%8C%E5%AE%9A%E3%81%AE%E5%9F%BA%E7%A4%8E+%E5%8D%98%E8%A1%8C%E6%9C%AC+%E2%80%93+2009%2F9%2F10)

また, こちらのidさんの記事の方も大変参考にさせていただきました.
私が実際にやった同定はこれより簡単なものになるので, もっと理論的なことを知りたい！という方はこちらを参考にするといいと思います.
[マイクロマウスの機体を同定する](http://idken.net/posts/2017-06-02-systemident/)

実際のMATLABでの操作はここらへんを参考にしました.
[https://jp.mathworks.com/help/control/getstart/interactively-estimate-plant-parameters-from-response-data.html](https://jp.mathworks.com/help/control/getstart/interactively-estimate-plant-parameters-from-response-data.html)
この動画の33分ぐらいから同定の際の操作についてビデオがあります.
[https://jp.mathworks.com/videos/pid-control-in-simulink-made-easy-94294.html](https://jp.mathworks.com/videos/pid-control-in-simulink-made-easy-94294.html)
# システム同定をする
まず, システム同定を行う対象はこちらの私が作っているクラシックマウスのNigLacertoです. 今回はこのロボットに対して, 並進方向及び回転方向についてモデリングを行います.

## 仕様  

  データ                  |  
--------------------------|--------------------------------------
  モーター & エンコーダー | FAULHABER 1717-006SR & IEH2-1024|  
  ギア比                  | 13 : 40
  重さ                    | 115g 

![robot](/images/2018-09-05-NigLacerto.jpg)

**実験データから同定する際にはSystemIdentification Tool Boxが必要です.**

## 並進方向
並進方向の同定を行うため, 並進方向のステップ応答を取りました.
上の図から分かるように, Plantへの入力は電圧, 出力は並進速度となるので, モーターそれぞれに電圧が1Vかかるように入力を与えてその時の並進速度の応答を見ます.

下の動画は実際にロボットに並進方向にステップ入力を入れたときの様子です.

{{< tweet 1043442561703002112 >}}

<!--
ソースコードはこんな感じで実装しました.(特に変哲はないですね)
```cpp
const uint16_t value = ADC_GetConversionValue(ADC1); //バッテリー電圧を確認
const int16_t voltage = (float)value / 3475.f * 8.4 ; //AD値からバッテリー電圧へ変換
const int16_t Duty = (TIM3_Period) / voltage * 1.0; // Duty比の計算
//suction_start(40);
TIM_Cmd(TIM5,ENABLE);
len_counter = 0;
TIM2->CNT = 0;
TIM8->CNT = 0;
volatile uint16_t cnt = 0;;
volatile int16_t input = 0;
while(1){
  if(ENCODER_start == ON){
    read_encoder();
    ENCODER_start = OFF;
  }
  if(GYRO_start == ON){
    rad = (float)(ReadGYRO() - GYRO_offset_data);
    GYRO_start = OFF;
  }
  if(button_return == 1)  break; //機体上のボタンが押されるとサンプリング終了
  if(timer_clock == ON){ //100Hzで呼び出されます.
    timer_clock = OFF;
    cnt++;
    if(cnt <= 100 || cnt >= 400) input = 0; //開始から1秒後にステップ応答を入れ, 4秒後に終了です.
    else input = Duty;
    plot.push_back(left_speed / MmConvWheel, right_speed / MmConvWheel, input); //サンプリングしたデータをpush_backしてます. 左から順に左側ホイール速度(mm/s), 右側ホイール速度(mm/s), 入力値 
    //plot.push_back(rad, input);
    set_speed(input, input);
  }
}
Delay_ms(500);
while(button_return == 0); //ボタンが押されるとデータを全部吐き出します.
plot.all_print();
```
-->

そしてMATLABを起動します. データを書き込んだファイルをTranslation1340.datとします.

```Matlab
load('Translation1340.dat') %読み込みます.
```

するとこんな感じのものが見えるのではないでしょうか？(もしくはWorkSpaceから見てください)
![table](/images/2018-09-22-loadData.png)
ここで必要なデータ(0だらけのところは省略して)をコピーしてpoという変数を作ります.

この後, 
```Matlab
pidTuner()
```
を起動します. こんな感じのが出ます.
![pidTuner](/images/2018-09-22-pidTuner.png)
左のPlantからIdentify New Plantを押します.(ここから先はSystemIdentification Tool Boxが必要です.)

そしてGet IO Dataから実験データを入れます.
![getIO](/images/2018-09-22-getIO.png)

poという変数にデータが入ってるのでこんな感じに入れます. サンプリングの周期は100Hz, オフセットは無しです. Ampilitudeは入力です. 僕はPWMを出しているタイマーのPeriodを1680としているので, 209は大体Duty比は12%ちょいですね. ロボットのプログラムにDuty比をそのまま入れてる方はDuty比を入れてもいいです.(もう何でもいいです)

![ImportResponse](/images/2018-09-22-ImportResponse.png)

Importを押すとこんな感じのものが出ます.
![State](/images/2018-09-22-beforeState.png)

右の画面の青い線が推定されたモデルで, 動かしたりすると変わります.
最初の段階だと, 1次系で推定しているので2次系にします. (StructureをTwo Real Polesにする.)(厳密にはステップ応答から2次系は推定できませんが...)

色々ドラッグしてみていい感じになったところで, Auto Estimateを見つけました. あなたは今までのドラッグ操作はまさか...と思うことでしょう.(私もです)
画面上部中央のAuto Estimateをクリックすると, いい感じにMATLAB側でEstimateしてくれます.(これはあなたが設定したStructureのもとで推定します.)
![estimate](/images/2018-09-22-AutoEstimate.png)


後はApplyを押すと, 作成したモデルが適用されます.

ここからはコントローラーを制作する作業に移ります. 
画面中央のResponse TimeとTransient Behaviorをいじることで, 左側のグラフの応答が変わります.
![pid](/images/2018-09-22-PID.png)

いい感じの応答を見つけたらShow parametersをクリックすると, PIDパラメーターが出てきます. わーい.

![pid](/images/2018-09-22-PIDgain.png)

後は, これらのパラメーターをあなたのロボットのソースコード中で使ってください.

最後にExportを押すと, 設計したコントローラーとPlantを変数にして出力できるので, saveしてあげれば後で再度利用することもできます.
```Matlab
save('transModel.mat', 'C', 'Plant1')
```

## 回転方向
また, 回転方向の同定を行うため, 同様に同定実験を行いました.

上の図から分かるように, Plantへの入力は, 出力は角速度となるので, モーターそれぞれに電圧が+-3Vかかるように入力を与えてその時の角速度の応答を見ます.

得られたcsvをdatファイルとして保存し, MATLABで読み込ませます. 今回必要になるのは角速度の情報なので, ジャイロセンサーからデータを取りました. 
応答は下の動画のようになりました.
{{< tweet 1037251127668953088 >}}

あとはここで得られた結果を先程と同じ要領で, 単位系に気をつけながらシステム同定します. 

# 実際の動きや同定する際の注意
## マイクロマウス東日本大会一日前
並進方向についてのみ同定した結果を突っ込んだら走り出しました.(このときは回転方向も並進方向と同じにしてました...)
まだ, 回転方向についての同定を行っていないので大回りのターンが膨らんでしまっていることが分かります.

{{< tweet 1035434404766597120 >}}

## マイクロマウス東日本大会
前日に突っ込んだパラメーターで走りました. ありがとうMATLAB

{{< tweet 1035891915332308992 >}}



## 適切な入力
同定する際に考慮しなくてはいけないのは入力が計測をする上で適切かどうかということです.
小さすぎる信号を入れてしまうと摩擦などの影響で機体が動かなくなってしまうし, 逆に大きすぎる信号の入力は出力が飽和して正しく計測できなくなってしまいます.
そのため, 適切な入力を見つけて応答を測る必要があります.

今回, 並進方向はそれぞれのモーターに+1Vの電圧をかけてその応答を見ましたが, 回転方向に関しては+-3V程度の電圧をそれぞれにかけて応答を見ました.

機体が変則四輪と呼ばれる(これはマイクロマウス用語です)4輪のスキッドステア車両型を取っているので, 旋回が一般的な2輪マウスに比べてやりにくいため, +-3V以下の電圧をかけるとそもそも回らなかったり, ムラのある回転をしました.
こうなるとモデリングができなくなるので, 注意です.

これは+-3Vの電圧をかけたときの応答の様子です.

{{< tweet 1037251127668953088 >}}

{{< tweet 1037256307525337088 >}}


これは+-1.5Vの電圧をかけたときの応答の様子です.

{{< tweet 1042781867143884800 >}}

動きを見てわかるように, 摩擦でロボットが適切に回転してないのが分かります. この状態で取れたログを使っても正しくモデルを同定できないので注意です.

また, 適切なデータを得るためにはタイヤをきれいにふく, 床をきれいにするなどの事前の準備が大事になります. がんばりましょう.

## 並進回転それぞれで同定が完了した走り
新しいコントローラーを作った後日に母校に突撃して迷路で少し走らせてもらったときの様子です.
東日本大会のときは正しく角速度について追従できてなかったので, ターンが膨らんだりしてましたが,
それぞれのコントローラーを設計して, 速度と角速度に適切(？)に追従してくれてるおかげでなんか走りました.

{{< tweet 1038418648526860290 >}}

# まとめ
MATLABはいいぞ. おしまいー.

## PS
記事を書く上で, [ところさん](https://twitter.com/tokoro10g)と[idさん](https://twitter.com/idt12312)から色々アドバイスをいただきました. ありがとうございます＞＜

