---
title: "だんごろまうす2weiの紹介 Part1"
date: "2018-05-19"
cover: "/images/2018-05-19-mouse.jpg"
categories: ["ロボット"]
tags: ["電子工作", "マイクロマウス", "STM32"]
slug: "Dangoromouse2weiPart1"
aliases: "/Dangoromouse2weiPart1.html"
---
自己紹介のところでマイクロマウスを作っていると言って, 記事を書いていなかった...  
というわけで2017年度のマイクロマウスの大会で私が作っていたマウスを紹介します.

## だんごろまうす2wei
![mouse](/images/2018-05-19-mouse.jpg)
回路図はこちらです. 上の充電基板とICM-20602は異種面付けでくっつけました.

[回路図](/pdfs/2018-05-19-circuit.pdf)

## 仕様  
{{< my_table class = "simple-table" >}}

 データ | 
 ------ | ------------------------
 サイズ | 78(W) * 75(L) * 23(H)mm 
 重さ   | 86g 
 ギア比 | 9:44
 ピニオンギア | 9T Module 0.5 [(Helimonster)](http://helimonster.jp/?pid=39331197)
 モーター | FAULHABER 1717-006SR & IEH2-1024
 バッテリー | Hyperion 1cell 120mAh * 2 
 マイコン | STM32F405RG 
 IMU | ICM-20602
 モータードライバー | DRV8835 * 1 |
 赤外線LED | VLSY5850 
 フォトダイオード | SFH213FA 

{{< /my_table>}}
## 3DCADデータ
Onshape上で作っているので全部公開されてます(わーい)(めっちゃ雑なのは許して...)  
[3DCADデータ](https://cad.onshape.com/documents/c2a612fa5d9512da2c2ba9ba/w/37291b77c13a207be24fd32c/e/8fe1776ea00df1f63d9a7140)

## プログラム
プログラムはGitHubの方で全部公開しています.(リンカスクリプトや探索周りは[idさん](http://idken.net/)から大いなる協力を得ています. ありがとうございます.)  
[GitHub](https://github.com/dangorogoro/Dangoromouse/tree/2weidev)

とりあえず最初は機体紹介とデータ公開をしました. 次は開発の経緯とか紹介していこうと思います.  

