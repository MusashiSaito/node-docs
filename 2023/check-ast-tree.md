# AST（Abstract Syntax Tree）の確認

フォーマッター導入時、フォーマットをかけたファイルに破壊がないかを確認する。

```
# 手順
# 先に npm install -g acorn しているものとする。

$ git switch feature/common-with-us-version-add-record-code-format
$ acorn add-record.js > after-format.json

$ git switch develop
$ acorn add-record.js > before-format.json

$ batdiff before-format.json after-format.json

```

AST をパースして json ファイルに出力しています。

batdiff は差分チェックで僕がよく使っている CLI ツール：https://github.com/eth-p/bat-extras/blob/master/doc/batdiff.md

AST のトークンの位置を表す start, end の差分がほとんど多数のようです。

ほかは、シングルクオートをダブルクオートに変換している差分のようでした。

```
$ batdiff before-format.json after-format.json | grep -v "start" | grep -v "end" | grep "^[+-]" | less

```

ダブルクオートの差分だけ表示するようにできていると思います。
