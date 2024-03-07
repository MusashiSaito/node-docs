# Docker 初心者が js ファイルを Docker hub に push してみた（M1 Mac 使用）

---

title: Docker 初心者が js ファイルを Docker hub に push してみた（M1 Mac 使用）
tags: Docker AWS Node.js JavaScript ECS
author: mu-prog
slide: false

---

Dockerhub に js ファイルを push し ECS でログを表示させるまでの過程を説明します。

#1. 実行するフォルダ作成

```
cd desktop
mkdir js-docker
cd js-docker
npm init　（Enterの連打）
npm install
touch Dockerfile
```

作成したフォルダ『js-docker』を開いて js ファイルを作成してください。

```
js-docker
   　　|- index.js
   　　|- Dockerfile
   　　|-package.json
```

```erb:Dockerfile
FROM node:14.4.0-buster

# Create app directory
WORKDIR /usr/src/app

# Install app dependencies (package.json and package-lock.json)
COPY package*.json ./
RUN npm install

# Bundle app source (server.js)
COPY . .

# Listen port
EXPOSE 8080

# Run Node.js
CMD [ "node", "index.js" ]

```

```erb:package.json
{
  "name": "hello",
  "version": "1.0.0",
  "description": "Node.js on Docker",
  "main": "server.js",
  "scripts": {
    "start": "node server.js"
  },
  "dependencies": {
    "express": "4.17.1"
  }
}
```

入力後 npm install

```erb:index.js
console.log("hello")
```

ひとまず「Hello」と表示させます

#2. Docker にイメージを作成
本来はこちらで Build するようですが。。。

```
docker build -t docker-js .
```

M1mac を消していて、AWS の ECS で使用する際にエラーが出たのでこちらで Build しました。

```
docker build --platform amd64 -t docker-js .
```

これを実行するとデスクトップの Docker の images に表示されます。
確認してみましょう！

```
docker images
```

```
REPOSITORY           TAG       IMAGE ID       CREATED              SIZE
docker-js           latest    4b4fd40e9d9c   About a minute ago   861MB
```

追加されています！

```
docker run docker-js

=> hello
```

大丈夫そうです！

#3. Docker hub に push
次に Docker hub に push します。

```
docker tag 9374g689ss65 takeshi/docker-js:latest
```

```
docker push takeshi/docker-js
```

Dockerhub にログインすると現在 push したものが追加されています。

```
docker run takeshi/docker-js
```

これでうまく動きますと成功しています！

https://hub.docker.com/

#おまけ　 M1 mac docker error: exec user process caused "exec format error"　の解決策

ECS でエラーが起こった際にこちらの記事で解決しました。

https://qiita.com/keita_ogawa/items/e115c46f1c8caf6fd34d

#おまけ　 docker 環境構築
初心者なので Docker の環境構築から始めます。
Docker Desktop のダウンロード

https://amateur-engineer.com/docker-mac-m1/

#参考にしたもの

js を動かせた記事

https://nodejs.org/ja/docs/guides/nodejs-docker-webapp/

その他

https://docs.docker.jp/v17.06/docker-for-mac/step_four.html

https://qiita.com/umi/items/d4b5a68263ad0444693b
