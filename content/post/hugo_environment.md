---
title: "Hugo環境づくり"
date: 2022-06-17T14:15:03Z
categories:
 - Tech
tags:
 - hugo
 - docker
keywords:
 - hugo
 - docker
comments: true
showMeta: true
showActions: true
description: Hugoでブログを書く環境を構築する
draft: false
---

# はじめに
この度Hugoでブログを書いてみることにしたので、その環境づくりについてメモしていく。

WSL2とmacのどちらでも動作確認済み。

# 環境
以下が必要。
- WSL2が動作するように設定済みのWindowsまたはmac（m1チップで確認）
- git
- お好きなエディタ（VSCode推奨）
- docker
- docker-compose

WSL2でdockerを動作させる設定等はググればいっぱい出てくるので今回は省略。

（一回設定したら忘れるので今度まとめて記事にするかも・・・）

# 手順
上記環境が整っていること前提で進める。

## ローカルに動作環境を作る
ディレクトリ構成は以下を想定。
```
blog
  ┃━ docker-compose.yaml
  ┃━ main  # このディレクトリはHugoが生成する　名前は任意で、このディレクトリをgitで管理する
      ┃━ README.md  # これは生成されたものでなく、後から作成したもの
      ┃━ archetypes
      ┃━ config.toml
      ┃━ content
      ┃━ data
      ┃━ docs
      ┃━ layouts
      ┃━ public
      ┃━ resources
      ┃━ static
      ┃━ themes  # 好きなHugoのテーマを入れる　submoduleで管理する
          ┃━ beautifulhugo
```

docker-composeで環境を作るので、以下の設定ファイルを任意のディレクトリに作る。

```yaml {linenos=inline}
version: '3'
services:
  hugo:
    # user: "${UID}:${GID}"
    image: klakegg/hugo:0.93.2-ubuntu
    volumes:
      - ".:/src"
      - "/etc/passwd:/etc/passwd:ro"
      - "/etc/group:/etc/group:ro"
    entrypoint: bash
    ports:
      - "1313:1313"
    tty: true
    working_dir: /src
```

コメントで示している部分はWSL2で必要な設定。（というかLinuxで？）

{{< alert color=" " >}}
dockerでディレクトリをマウントした時、ホストとコンテナ内でのユーザや権限の違いから、
ファイルの編集ができなかったりする。

これを解決するために、ホストのUIDとコンテナのUIDを指定している。

方法としては良くないみたいだが暫定的に・・・

詳しくは以下のような記事があったので参考に。
- https://www.forcia.com/blog/002273.html
- https://qiita.com/yohm/items/047b2e68d008ebb0f001
{{< /alert >}}

現在のディレクトリが、コンテナ内の /src ディレクトリにマウントされる。

次に、以下のコマンドでコンテナを作って動かす。

```bash
$ docker-compose up -d
```

コンテナの中に入って操作していく。

```bash
$ docker ps
# CONTAINER ID   IMAGE                        COMMAND   CREATED          STATUS          PORTS                    NAMES
# 7b75f525797e   klakegg/hugo:0.93.2-ubuntu   "bash"    28 minutes ago   Up 28 minutes   0.0.0.0:1313->1313/tcp   blog-hugo-1

$ docker exec -it blog-hugo-1 bash
```

コンテナ内では hugo コマンドが使えるようになっているので、初期ファイルを生成していく。

```bash
$ hugo new site main  # mainは任意の名前で

$ ls
# docker-compose.yaml  main

$ ls main
# archetypes  config.toml  content  data	layouts  static  themes
```

mainディレクトリが作られ、その下に色々とファイルが生成されている。

ここまでで動作環境はできたので、まずは最初の記事を作って、ローカルで動作を確認してみる。

## 動作確認
テーマが豊富に存在するので、好みのテーマを取ってくるのが良い。（[テーマ一覧](https://jamstackthemes.dev/ssg/hugo/)）

というか見た目と設定を自分で作り込んでいくのはちょっと大変なので取ってくる。

今回は[beautifulhugo](https://github.com/halogenica/beautifulhugo)を利用したいので、
生成された themes フォルダにリポジトリをクローンする。

```bash
$ cd themes

$ git clone https://github.com/halogenica/beautifulhugo.git
```

config.tomlにこのテーマを使うための指定をする。

```config.toml
theme = "beautifulhugo"  # クローンしてきた際のディレクトリ名
```

設定ファイルは beautifulhugo ディレクトリの下にもサンプルとして入っていたりするので、後で参考にすると良い。
（beautifulhugo/exampleSite/config.toml）

次に、新たに記事を作成する。

```bash
$ hugo new post/firstpost.md

# Content "/src/main/content/post/firstpost.md" created
```

これで、content 以下に post/firstpost.md が作成される。

内容は適当に。

```firstpost.md
---
title: "Firstpost"
date: 2022-06-18T00:38:03Z
draft: true
---

# 初投稿
Hugoに記事を作ってみました！
```

頭の title とか date とかは自動で付けられているので、いつものmarkdownを書いていく感じで作れる。
draft は、記事がドラフト版であることを指す。

動作確認のために、main 直下まで移動しておき、以下のコマンドでサーバを起動する。

```bash
$ hugo server -D

# -D はドラフト記事を生成対象にするオプション
# サーバは Ctrl+C で止められる。
```

ポートはデフォルトで1313なので、localhost:1313にアクセスしてみる。

以下のようなページが表示されていたらOK！！

![top page](/images/hugo_environment/toppage.png)


## おわりに
この後は基本的に、config.tomlに設定を書き、content下に記事を書いて置いていくことになる。

ここでは記載していない設定などしていたり、他の機能も使ってみたりしたいのでしばらく遊んでみたい。
