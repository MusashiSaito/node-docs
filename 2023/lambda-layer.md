# lambda でレイヤーを使用し、アップロードしたファイルのコードを表示させてみた。

---

title: lambda でレイヤーを使用し、アップロードしたファイルのコードを表示させてみた。
tags: AWS lambda
author: mu-prog
slide: false

---

ローカルで作成したファイルを、lambda にアップロードした際に「Lambda 関数「layer-test」のデプロイパッケージが大きすぎて、インラインコード編集を有効にできません。ただし、関数を呼び出すことはできます。」
と表示されたのでレイヤーを活用して表示させました。

初めての作成に時間がかかってしまったので、手順を詳しく説明して行きたいと思います。
レイヤーを使用せずに zip ファイルをアップロードした時、このようにコードが表示されず lambda 内で作業ができませんでした。

パッケージングした元のファイル構造はこのようになっています。

```
file
 |
 |-node_modules
 |-index.js
 |-package-lock.json
 |-package.json
```

このファイルのままではコードは表示されないままなので、まず、コードの表示に向けて二つのファイルに分けました。
lambda で表示させたい js ファイルと、あまり動かさない node_modules や json ファイルで分けます。

```
lambdaファイル
file
 |
 |-index.js
```

zip -r function.zip .　でパッケージングします。

```
レイヤーファイル
nodejs
 |
 |-node_modules
 |-package-lock.json
 |-package.json
```

- `desktop % zip -r function.zip nodejs`でパッケージングします。
- まず、file を lambda 関数にアップロード
- 次はレイヤーを作成します。
- その他のリソースのレイヤーを押します。
- レイヤーの作成
- 名前の入力と.zip ファイルのアップロード
- バージョン ARN をメモしておいてください。
- 次に作成したレイヤーを関数に追加していきます。
- 追加する lambda 関数のページに行き、一番下までスクロールします。
- レイヤーの追加を押します。
- 先ほどメモした ARN を指定に記入し追加ボタンを押すと追加されます。

もし、レイヤーに変更があれば、レイヤーのバージョンの作成で新しい zip ファイルをアップロードしてください、また、レイヤーの追加ボタンの横にある編集ボタンでバージョンのみを編集すると変更後のレイヤーで動かせます。

最後までご高覧いただきましてありがとうございました。