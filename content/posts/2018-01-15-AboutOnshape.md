---
title: "Linuxで動く3DCAD Onshapeのお話"
cover: "/images/Onshape.png"
date: "2018-01-15"
categories: ["Onshape"] 
tags: ["Onshape"]
slug: "AboutOnshape"
aliases:  "/AboutOnshape.html"
aliases: "/ShenZhenTrip2018.html"
---

最初の記事は今使ってる3DCAD製作支援サービス, Onshapeの話をしようと思います.

## Linux 3DCADが 動かない
---
私は普段使いのノートPCにはArch LinuxとWindowsというOSをデュアルブートしていて, 作業は専らArch Linuxの方でやっています.  
何不自由ないLinuxライフを送っている私ですが, 唯一困るのはロボット製作をするときに使う3DCADソフトです. というのも  

- InventorやFusion360はLinuxにインストールできない.  
- オープンソースで開発されているFreeCADは実用に耐えられない(2018/01/15 時点の私個人の感想です.)  
  
という問題があります. 解決策として挙げられるのは  

- Wineでがんばる.  
- Windowsを立ち上げてInventorやFusion360をインストールする.  
- Onshapeを使う.  

などが挙げられます.環境構築で時間を使いたくなかったし, Linuxだけで作業を完結させたかったのでOnshapeを使ってみました.

## Onshapeを使う
---
OnshapeのHPです.  
[https://www.onshape.com/](https://www.onshape.com/)

OnshapeはWebブラウザで使える無料の3DCAD支援サービスです. アカウントを登録すれば誰でも無料で使うことができます.  
![参考写真](/images/Onshape.png)
もちろん, Linux上でも動作します.(現在, 私はChromiumというWEBブラウザ上で問題なく動作させています.  
全ての処理がクラウド上で行われるので手元のマシンに新しくインストールするものは何もないです.  
また, データはサーバ上にあって, AndroidやiPhoneで動作するアプリケーションもあるのでどこでも作業することができます.  
作ったデータに関してですが, 基本的に制作物は誰でも見られるよう公開されているので, 他の人の制作物も簡単に見て学ぶことができます. (課金で非公開にすることも可)  
ブランチ機能もあり, 制作物の状態をGitのように管理できるので便利です.
### 使い方
基本的にはスケッチ書いて押し出したり, 回転させたりするとできます.  
他の3DCADソフトと使い方はあまり変わらないのでここで言及しません.  
TutorialやVideo群がここにあるので参考にしてみるといいと思います.  
[https://www.onshape.com/video](https://www.onshape.com/video)

## FeatureScript
---
最近, FeatureScriptという機能が公開されました.  
[https://www.onshape.com/featurescript](https://www.onshape.com/featurescript)  
これは, Onshape上で動作するプログラミング言語で, 自分でOnshape上で動作するスクリプトがかけるようになりました.  
また, 人の書いたスクリプトも自前のStudio上で動作させることができます.  
今回はFeatureScriptで書かれたcustom featureの一つ, SpurGearの使い方を紹介しようと思います. (これが出てきたのでブログを書こうと思いました.)  

### SpurGear
---
このスクリプトはOnshapeの中の人が作ったもので,インボリュート曲線を使ったギアを作ることができます.  
私はマイクロマウスというロボット競技に参加していて, ギアを設計するという機会が何度かあるのですが, 今までは  

- [小原歯車工業](http://www.khkgears.co.jp/)にログインし, 欲しいモジュールとサイズのギアのdxfを作る.  
- Onshapeでインポートする.  

という大変めんどくさいことをしていました. しかし, このScriptを使うと一発で作ることができます!便利ですね.  
使い方はまず, 画面右上のプラスボタンから追加します.  
![plus button](/images/2018-01-23-070217_483x32_scrot.png)

次にFeatureScriptSampleからSpurGearを選びます.  
![Sample](/images/2018-01-23-063743_271x353_scrot.png)

出てきたものからSpurGearのFeatureScriptを選んで準備完了です.
![FeatureScript](/images/2018-01-23-063906_279x463_scrot.png)

右上に新しくSpurGearのアイコンが出てきたと思います.  
![LikeThis](/images/2018-01-23-072828_375x31_scrot.png)

こんなふうにパラメータをいじってあげれば簡単にギアが作れます.  
![GearSample](/images/2018-01-23-071808_888x559_scrot.png)

## まとめ
---
フルクラウドで動いてるので大きい物を作ろうとすると固まったりしますが, 個人で何かちょっと設計がしたい, Linuxでも3DCADが使いたい(ココ重要!)という方にはOnshapeはとても便利だと思います.  
ぜひ使ってみてください.
