---
title: "GitHub Actions使って日記をAWSにぶん投げる"
date: "2020-07-02"
categories: ["GitHub"]
tags: ["GitHub", "Hugo", "GitHub Actions"]
share_img: "/images/2020-07-02-ActionsLogo.png"
cover: "/images/2020-07-02-ActionsLogo.png"
slug: "DiaryWithGitHubActions"
---

# これ何？
---
最近日記をつけ始めました. (diary.oino.li) 日記もブログと同様にHugoで生成しています.
日記の方は真面目に書かない(書きたいことを適当に書く)ので, プラグインとか特に入れずに文字だけMarkdownで起こせばそのままAWSに置いてるサーバーに反映させられたらといいなーと思ったのでGitHub Actionsというのを使ってみました.

後, apt で入れたHugoのバージョンがめっちゃ古くて萎えたというのもあります.

{{< tweet 1268891509203468290 >}}

今までこういったCIとか全く使ったことが無かったですが, なんとか動かせたのでその時の作業の備忘録を書いておきます.
# やったこと
---
全体の大まかな流れはこんな感じです.
1. ローカルの環境でMarkdownを書く.
2. CommitしてGitHubに投げる.
3. GitHub ActionsにてビルドしてAWSに置いてるサーバーにデプロイされる.
4. 嬉しい.

主にこちらの記事を参考にしました. ありがとうございます.
(https://qiita.com/kz_morita/items/690b367067666fddb562)
# GitHub Actionsとは
---
GitHubの提供するCI / CD環境です.

> プッシュ、Issue、リリースなどのGitHubプラットフォームのイベントをトリガーとしてワークフローを起動しましょう。コミュニティが開発・保守し、ユーザが熟知・愛用しているサービスについて、対応するアクションを組み合わせて設定できます。
> コンテナアプリの構築、Webサービスの展開、レジストリへのパッケージの公開、またはオープンソースプロジェクトへの新しいユーザーのウェルカムメッセージを自動化するなど、さまざまな用途でActionsを利用できます。 GitHub PackageとActionsを連携させパッケージ管理を簡素化できます。バージョンのアップデート、グローバルCDNによる高速配布、既存のGITHUB_TOKENを使用した依存関係の解決なども行えます。
(https://github.co.jp/features/actions より)

使用制限は
- パブリックリポジトリだと無料.
- プライベートリポジトリだと一定量の無料の動作時間とストレージが与えられ, 超過分について課金.

とのことです.
この動作時間はLinuxのランナー上で実行されるジョブの動作時間を基準としていて, 

WindowsやmacOSのランナー上で実行されるジョブの動作時間は
- Windowsは2倍換算
- macOSは10倍換算

になってしまうので注意.

詳しくはこちらをご覧ください.
(https://docs.github.com/ja/github/setting-up-and-managing-billing-and-payments-on-github/about-billing-for-github-actions)

# Actionsの作成
---

Actionsを作成したいリポジトリに行くと上の方にこんな感じでメニューがあるのでクリックして新しくworkflowを作成します.

![Images](/images/2020-07-02-Actions.png)

するとこんな感じで色々項目が出てきます.
下にちょうど「Deploy to Amazon ECS」とあってこれじゃーんと思った筆者だったのですが, Webの事が何も分からない私なので出てくる用語の意味が分からず, 諦めて「Simple workflow」を選択しました.(悲しいね.)

![Images](/images/2020-07-02-WorkFlow.png)

workflowを作成すると編集画面が出てきます. 設定ファイルはyamlファイルで記述されていて, リポジトリの **.github/workflows/hoge.yml**に作成されます.

(https://qiita.com/kz_morita/items/690b367067666fddb562)の記事を参考にこんな感じのyamlを書きました.

```yaml
name: deploy to server
on:
  push:
    branches:
      - master
jobs:
  build:

    runs-on: ubuntu-latest

    steps:
    - name: Checkout
      uses: actions/checkout@v1
      with:
          submodules: true
    - name: Setup hugo
      run: |
          wget https://github.com/gohugoio/hugo/releases/download/v${HUGO_VERSION}/hugo_${HUGO_VERSION}_Linux-64bit.deb
          sudo dpkg -i hugo_${HUGO_VERSION}_Linux-64bit.deb
          hugo version
      env:
          HUGO_VERSION: '0.73.0'
    - name: Build hugo
      run: hugo
    - name: Generate ssh key
      run: echo "$SSH_PRIVATE_KEY" > key && chmod 600 key
      env:
        SSH_PRIVATE_KEY: ${{ secrets.SSH_PRIVATE_KEY }}
    - name: Deploy
      run: rsync -Pe "ssh -i ./key -o StrictHostKeyChecking=no -o UserKnownHostsFile=/dev/null -p ${SSH_PORT}" -r public/* $SSH_USER@$SSH_HOST:$DEPLOY_PATH
      env:
        SSH_USER: ${{ secrets.SSH_USER }}
        SSH_PORT: ${{ secrets.SSH_PORT }}
        DEPLOY_PATH: ${{ secrets.DEPLOY_PATH }}
        SSH_HOST: ${{ secrets.SSH_HOST }}
```
私のリポジトリの該当箇所はコチラに置いてます.
(https://github.com/dangorogoro/diary/blob/master/.github/workflows/test.yml)

ざっくり解説していきます.

## トリガー

{{< highlight yaml "linenos=table, linenostart=2" >}}
on:
  push:
    branches:
      - master
{{< / highlight >}}
Actionsが実行されるためのトリガーの設定です.

## Checkout
{{< highlight yaml "linenos=table, hl_lines=4, linenostart=12" >}}
    - name: Checkout
      uses: actions/checkout@v1
      with:
          submodules: true
{{< / highlight >}}

このアクションは、$GITHUB_WORKSPACEにあるリポジトリをチェックアウトし, ワークフローでアクセスできるようにします.
最近はv2が出たとのこと.
(https://github.com/actions/checkout)
submodules について記述することでsubmoduleもワークフローでアクセスできるようになります. 私はHugoのテーマをsubmoduleで管理してるんですが, これを忘れて無を生成しました.

## デプロイ
{{< highlight yaml "linenos=table, hl_lines=8-12, linenostart=25" >}}
    - name: Generate ssh key
      run: echo "$SSH_PRIVATE_KEY" > key && chmod 600 key
      env:
        SSH_PRIVATE_KEY: ${{ secrets.SSH_PRIVATE_KEY }}
    - name: Deploy
      run: rsync -Pe "ssh -i ./key -o StrictHostKeyChecking=no -o UserKnownHostsFile=/dev/null -p ${SSH_PORT}" -r public/* $SSH_USER@$SSH_HOST:$DEPLOY_PATH
      env:
        SSH_USER: ${{ secrets.SSH_USER }}
        SSH_PORT: ${{ secrets.SSH_PORT }}
        DEPLOY_PATH: ${{ secrets.DEPLOY_PATH }}
        SSH_HOST: ${{ secrets.SSH_HOST }}
{{< / highlight >}}

rsyncでビルドした生成物をAWSに置いてるサーバーに送ります.
scpは非推奨になっていたので使うのをやめました. (hai)(https://www.openssh.com/releasenotes.html)

ここでは様々な変数(公開できない鍵とかサーバーのディレクトリ)とかをGitHubのSettings->Secretsにて設定できます.
この変数は暗号されていて, 選択されたアクションにのみ公開されます.

![Images](/images/2020-07-02-Key.png)

SSHのキーはローカルの環境で生成しているので, ホストが変わっても怒られないように
```
-o StrictHostKeyChecking=no -o UserKnownHostsFile=/dev/null
```
と追加してします.
これでローカルからプッシュするだけでサーバーの方に日記が反映されるようになりました.

Actionの実行結果はこんな感じでログを見たりすることができます. 失敗するとメールで通知も送ってくれます. (最初の数回は失敗しまくったので, メールボックスが爆発しました.)
![Images](/images/2020-07-02-Result.png)

嬉しいね.
# おわりに
---
記念すべき初めてのGitHub ActionsはGitHubが落ちて失敗しました.(そんなことある？？？)
GitHub Actionsが上手く動かなかったらGitHub Status(https://www.githubstatus.com/)を見てみると良いです.

その時の様子です.
{{< tweet 1277562875188404224 >}}

あと, 最初の数回はGitHub上でテキストが編集できることをいいことにビルドが失敗してはGitHub上でFixしてCommitしてActionを走らせてました. (本当に最悪)
{{< tweet 1277569636297404422 >}}

Github Actions, 結構便利ですね. 多人数で開発するときとか重宝しそうです. テストとか書くときにこういうのを使ってみたいかな.
