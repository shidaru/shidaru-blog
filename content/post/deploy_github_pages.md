---
title: "GitHub Pagesでブログを公開する"
date: 2022-06-22T12:19:41Z
categories:
 - Tech
tags:
 - hugo
 - github_pages
 - github_actions
keywords:
 - hugo
comments: true
showMeta: true
showActions: true
description: GitHub Pagesでブログ記事を公開する
draft: false
---

# はじめに
Hugoでブログを作っていける環境ができたので、GitHub Pagesにデプロイして公開してみます。

# Githubにリポジトリを作る
ディレクトリ構成は以下で、`main`ディレクトリをGitの管理対象とする想定。
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

GitHub上で適当なリポジトリを作っておく。

ここでthemeが問題になるが、他のリポジトリが管理対象になってしまう。
（普通に `git add .` するとエラーになる）

今後themeの中身に変更を加えたり、バージョンの管理などをしやすいように、themeをサブモジュールとして登録する。

一旦テーマのディレクトリを削除し、再度サブモジュールとして同じテーマを取得する。

```bash
$ rm -rf theme/beautifulhugo  # 適宜変更すること

$ cd theme

$ git submodule add https://github.com/halogenica/beautifulhugo
```

因みに、サブモジュールが含まれるリポジトリを別環境で再構築する場合、サブモジュールを含めてクローンすることを明示しなければならないので注意。

```bash
$ git clone --recurse-submodule --depth 1 git@github.com:USRNAME/REPOSITORY.git
```

# GitHub Actions を設定する
ローカルでページを生成してデプロイしても良いが、毎回やっていると面倒なのでGitHubにpushしたら自動でデプロイされるようにしてみる。

`main/.github/workflows/actions.yaml`ファイルを作成し、内容を以下にする。

内容は[ここ](https://github.com/peaceiris/actions-hugo)を参考にしている。

このファイルがあると、自動でGitHub Actionsが動作してくれるっぽい。

```actions.yaml
name: GitHub Pages

on:
  push:
    branches:
      - main  # pushに反応するブランチ

jobs:
  deploy:
    runs-on: ubuntu-20.04
    concurrency:
      group: ${{ github.workflow }}-${{ github.ref }}
    steps:
      - uses: actions/checkout@v2
        with:
          submodules: true  # サブモジュールを取得する
          fetch-depth: 0
      - name: Setup Hugo
        uses: peaceiris/actions-hugo@v2
        with:
          hugo-versions: '0.93.2'
          extended: true
      - name: Build
        run: hugo --minify --destination docs  # Hugoのビルドコマンド
      - name: Deploy
        uses: peaceiris/actions-gh-pages@v3
        with:
          github_token: ${{ secrets.GITHUB_TOKEN }}
          publish_dir: ./docs
          cname: shidaru.com  # 独自ドメイン（無くても良い）
```

本ブログでは独自ドメインを取得しているが、無ければ`USER.github.io`が自動で付与されるので無くても公開できる。ドメインはGoogle Domainsで取得している。

とりあえず、ここまででGitHubにpushする。

# GitHub Pages を設定する
次にGitHub上でPagesの設定をする。

作ったリポジトリの、以下の設定画面に移動する。（USERとREPOSIORYは適宜設定）

`https://github.com/USER/REPOSITORY/settings/pages`

GitHub Actionsが正常に動作すると、`gh-pages`というデプロイ用ブランチが作成されている。

これを`Source`に設定し、ディレクトリは`/root`とする。

`Custom domain`は取得しているなら入力する。
DNSチェックが入るので、例えばGoogle Domainsで取得していたらカスタムレコードを設定しておくこと。

全部設定したらこんな感じ。

![actions](/shidaru-blog/images/deploy_github_pages/github_pages_config.png)

緑の枠で囲われた、`Your site is published at https://****.com`にアクセスできればOK！

# さいごに
これでGitHub Pagesでブログが公開できるはず。

この後記事の追加をすると更新されることが確認できれば完璧。

ただ変更が反映されるのが遅いので、追加変更されてなくてあれ？と思っても少し待つのがよろし。
