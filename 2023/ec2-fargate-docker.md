# ECS の Fargate を使用してログで Hello-World を表示させてみた。

---

title: ECS の Fargate を使用してログで Hello-World を表示させてみた。
tags: #Fargate #gateway AWS ECS lambda
author: mu-prog
slide: false

---

## 1. Docker イメージの準備

Docker とは、Docker 社が開発している、コンテナ型の仮想環境を作成、配布、実行するためのプラットフォームです。
まず、ターミナルにこちらを起動させてください。

```
docker run hello-world
```

結果

```
Hello from Docker!
This message shows that your installation appears to be working correctly.

To generate this message, Docker took the following steps:
1. The Docker client contacted the Docker daemon.
2. The Docker daemon pulled the "hello-world" image from the Docker Hub.
(amd64)
3. The Docker daemon created a new container from that image which runs the
executable that produces the output you are currently reading.
4. The Docker daemon streamed that output to the Docker client, which sent it
to your terminal.

To try something more ambitious, you can run an Ubuntu container with:
$ docker run -it ubuntu bash

Share images, automate workflows, and more with a free Docker ID:
https://hub.docker.com/

For more examples and ideas, visit:
https://docs.docker.com/get-started/
```

これは Docker にすでにある公式イメージを動かしました。

次は AWS に移ります。

## 2. AWS ECS タスク定義の作成

https://aws.amazon.com/jp/ecs/?whats-new-cards.sort-by=item.additionalFields.postDateTime&whats-new-cards.sort-order=desc&ecs-blogs.sort-by=item.additionalFields.createdDate&ecs-blogs.sort-order=desc

- ECS を開いてタスク定義へ移ります。タスク定義の作成を押してください。
- 起動タイプの互換性の選択で「Fargate」を選択してください。
  次へ！
- タスク定義は自分で確認できるものを入力してください。
- タスクメモリ（GB)とタスク CPU（vCPU）もそれぞれ入力します。
  コンテナ追加を押してください。
- コンテナ名は自分で確認できるもの、イメージはさっき動かした「hello-world」でお願いします。

## ３. クラスターの作成

- クラスターの作成を押してください

- クラスターテンプレートの選択は「ネットワーキングのみ」を押してください。
  次へ！
- クラスター名を入力し、VPC の作成にチェックを入れてください。（CIDR ブロックとサブネットが作成されます。）
- もしも、VPC の数が上限を超える場合、作成されないので VPC の確認が必要です。

# 4. タスクを実行し、ログで Hello-World を表示

作成したタスクにチェックを入れ、アクションからタスクの事項をします。

起動タイプを　 FARGATE、クラスターを作成したもの、クラスター VPC とサブネットは作成したものを選択します。

```
入力する３項目
・起動タイプ
・クラスター
・クラスターVPC
・サブネット
```

入力が終わりましたら「タスクの実行」を押して実行します。
実行したものがクラスターのタスクに表示され、ステータスが Running、のちに Stopped となります。
動作が終了しましたら、タスクを開いてログを確認してみてください。
ログが出せましたら完了です！

#参考記事

https://docs.aws.amazon.com/ja_jp/AmazonECS/latest/developerguide/getting-started-fargate.html

https://dev.classmethod.jp/articles/ecs-hello-world-task/
