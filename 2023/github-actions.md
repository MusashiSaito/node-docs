# GitHub Actions について学んでみた

---

title: GitHub Actions について学んでみた
tags: GitHub
author: mu-prog
slide: false

---

# 基礎知識

GitHub Actions とは GitHub 組み込み式の CI／CD システムです。
例えば、コードの変更を push したタイミングで GitHub 　 Actions がそれを検知し、ビルド、テスト、デプロイのような一連の動作を行います。

# CI／CD とは

CI とは「継続的インテグレーション」

変更ごとにテストを含む自動ビルドが毎回実行されるようにする。
目的は、CI が成功しているか、失敗しているかの状態を可視化する。

CD とは「継続的デリバリー」
CI をさらに発展させたもので、テストを含む自動ビルドが毎回実行されるのは同じです。
コードを変更してからリリースまでの検証（性能検証、脆弱性検証）するパイプライン「デプロイメントパイプライン」によってリリースまでのフローの改善とともに可視化することができます。（ボタンひとつで楽に）

# GitHub Actions と従来の CI／CD ツールとの違い

従来の CI／CD ツールは push イベントによる CI/CD がメインで、それ以外のイベントはサポートされていなかった。
GitHub Actions で push 以外のさまざまなイベント発生時にも自動で実行することができる。

# 料金体系

パブリックリポジトリー　は完全に無料。
プライベートリポジトリーは GitHub のプランによるが無料分を超えた文については料金がかかります。

# 一旦やってみよう Hello,World!

リモートと接続したディレクトリに「.github」ディレクトリを追加します。

```
YourDirectory/.github/workflows/hello.yml
```

.github/workflows/配下のファイルが GitHub Actions として動作します。

以下のようなワークフローを記載します。

```.github/workflows/hello.yml:
name: hell,world!
on: push

jobs:
  build:
    name: Greeting
    runs-on: ubuntu-latest
    steps:
     - run: echo Hello, world!
```

実際にリモートブランチに push したところ、「Actions」で HelloWorld が動いていました。
これから実際に Actions 作成に向けて、ワークフローの記載など勉強していきます。

# GitHub Actions の機能説明

## ワークフロー

**ワークフロー**とはプロセスを自動化したものです。
**ジョブ**は複数のステップで構成されます。
**ステップ**は何かしらのコマンドの実行を行います。
**アクション**は何かしらの処理の塊を表す単位です。

## ワークフローの構文

`name`
1 行目の name はワークフローの名前を表します。

`on`
ワークフローを実行するイベント名を表します。（push や pull_request など）

`jobs`
この中でジョブの定義を記載します。

`jobs<job_id>`
また、job?と思うかもしれませんが、注目すべきは`job_id`
ジョブの id を定義します。

`jobs<job_id>.name`
ジョブの名前を定義します。

`jobs<job_id>.runs-on`
ジョブが実行されるマシンを定義します。

`jobs<job_id>.steps`
ジョブが実行するステップの定義を記載します。

`jobs<job_id>.steps.name`
ステップの名前を定義します。

`jobs<job_id>.steps.uses`
アクションの実行を定義します。`uses: actions/checkout@v2`このように特定のタグを指定することなどが可能です。Docker などもここでの指定が可能です。

``

# [目標]

今回、この GitHub Actions について学んだ理由は一つの目的がありました。

Novelworks の製品 Chobiit においてリリースのたびに、タグを作成する時間を削減したいと考えました。

これを導入すべきか否か？2 点の理由より学習を行った。

まず、タグをつけるだけの行為にかかる時間や説明の手間をなくす。

初めてタグをつける際に起きた事象

- タグをつけ忘れる
- タグを develop ブランチにつけてしまう。
- なんか wite に書かないといけなかったかなと他のタグやマニュアルを見返す時間をさく
- 上記についてやりとりが入り時間をさく
  ![スクリーンショット 2023-09-17 19.09.10.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/1129553/b687b085-e530-7eaa-75a3-520abc271c3e.png)

みたいな業務時間としては無駄な時間を使ってしまい、初回だと 10 分以上は使ってしまう。ようなことがあるかもしれない。

二つ目に、個人的に小さなミスが多い（ド忘れ・注意不足・気分不安定）。
プログラマーならば、自分の弱みもプログラムでカバーしていけばいいのではないかと感じました。

上記の理由より「タグを作成する時間を削減するアクションを作る」というのを GitHub Actions を学ぶ上での一つの目標にしました。

# 「タグを作成する時間を削減するアクションを作る」

これを普段の開発を振り返り作成していきます。

## 行う動作

普段タグ付けを行うまで以下のようなステップを進めています。

1. develop にて開発を行う。
2. リリースが近づいたので「release/2000-09-07（ここは日付）」のようなブランチを作成します。
3. リリースが完了します。
4. release/2000-09-07 から main ブランチへのプルリクを作成します。
5. main へマージします。
6. main ブランチに「2000-09-07」のタグをつけます。

ここまでリリースとタグ付けの手順になります。
つまり GitHub Actions を使用することで 5 と６の動作を同時に行えるようにします。

（memo:なんとなくですけど release/2000-09-07 から main ブランチへのプルリク作成もさっとできるようにしたいな。。やること大体わかるから入力するのが面倒な気が。。
こうやって手順を作成したら GitHub Actions や shell スクリプトなどでも自動化できるところはいっぱい見つかりそう。。）

## 作る

早速上記の目的にあった GitHub Actions を作成していきます。

```yaml
# 実行するワークフローの名前
name: Create Release Tag
```

### ワークフローを実行するイベント

```yaml
on:
  pull_request:
    branches:
      - main
    types:
      - closed
```

- [こちら](https://docs.github.com/ja/actions/using-workflows/events-that-trigger-workflows)のドキュメントに「[ワークフローをトリガーするイベント](https://docs.github.com/ja/actions/using-workflows/events-that-trigger-workflows)」について記載しています。
- 今回は[プルリクエスト](https://docs.github.com/ja/actions/using-workflows/events-that-trigger-workflows#pull_request)を使用し対象のブランチを`main`とします。
- `types`には`merged`のようなものが存在するのかと思ったのですがない。。
  今回は`closed`を使用し以降で対策を行なっていきます。

### ジョブ定義を行う

```yaml
jobs:
  build:
    name: Create Release
    runs-on: ubuntu-latest
    if: github.event.pull_request.merged == true
```

- `jobs`よりジョブ定義が始まります。
- `build`は`job_id`になります。id 同士は重複してはならず、使える文字は Alphbet・数字・「-」・「\_」となります。
- `jobs.<job_id（build）>.name`はこジョブの名前を定義します。Actions を開いたときに表示されます。今回はわかりやすく「Create Release」とします。
- `runs-on`これはジョブが実行されるマシンの指定になります。（何選ぶべき）
  以下表:runs-on で指定できるマシン環境

| runs-on                      | マシン環境           |
| :--------------------------- | :------------------- |
| ubuntu-latest, ubuntu-18.04  | Ubuntu 18.04         |
| ubuntu-16.04                 | Ubuntu 16.04         |
| windows-latest, windows-2019 | Windows Server 2019  |
| windows-2016                 | Windows Server 2016  |
| macos-latest, macos-10.15    | macOS Catalina 10.15 |
| self-hosted                  | セルフホストランナー |

- jobs.job_id.if
  `if`は条件が満たされない限りジョブが実行されないようにすることができます。
  ここで旋回指定できなかった、「github.event.pull_request（リクエスト元）のマージが行われていた場合」という制限をかけることができます。

https://docs.github.com/en/actions/using-workflows/workflow-syntax-for-github-actions#jobsjob_idif

### ステップを作成する

今回行いたい

```yaml
steps:
  - name: Get branch date
    id: get_branch_date
    run: echo "branch_name=$(echo ${{ github.event.pull_request.head.ref }} | cut -d'/' -f2)"  >> $GITHUB_ENV
  - name: Create release
    id: create_release
    uses: actions/create-release@v1
    env:
      GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
    # ERROR :Resource not accessible by integration
    # https://zenn.dev/tatsugon/articles/github-actions-permission-error
    with:
      tag_name: ${{ env.branch_name }}
      release_name: ${{ env.branch_name }}
      draft: false
      prerelease: false
```
