---
title: "HugoでGoogleAnalyticsを設定する"
date: 2022-06-23T12:21:16Z
categories:
 - Tech
tags:
 - hugo
 - google_analytics
keywords:
 - hugo
 - google_analytics
comments: true
showMeta: true
showActions: true
description: HugoでGoogleAnalyticsを設定する
draft: false
---

# はじめに
作成したブログのアクセス解析にGoogle Analyticsを利用するための設定を実施します。

# Google Analyticsに登録する
今まで利用したことがない場合、Google Analyticsで検索して登録を行う。

最近は`Google Analytics 4`という新しいサービスが展開されているので、
それで登録する。

利用したことがあれば古いサービスからの移行が必要になる。

アナリティクスの設定ページの`GA4設定アシスタント`に移動し、`Go to your GA4 property`から
GA4の設定を実施する。

登録できたら以下の`データストリーム`の項目から設定のためのタグを取得できる。

![データストリーム](/images/ga4/ga4_data_stream.png)

表示されているストリームをクリックし、`タグ設定手順` -> `グローバルサイトタグ`の項目を開く。

以下のようなタグがあるので、これを後でHugoの方に設定する。

```html
<!-- Global site tag (gtag.js) - Google Analytics -->
<script async src="https://www.googletagmanager.com/gtag/js?id=G-****"></script>
<script>
  window.dataLayer = window.dataLayer || [];
  function gtag(){dataLayer.push(arguments);}
  gtag('js', new Date());

  gtag('config', 'G-****');
</script>
```

# Hugoにタグを設定する
GA4のタグはHTMLのheadタグ内に置く必要がある。

Hugoではページの部品がいくつかのファイルに分割されており、全体の生成時にまとめられる。

この中でheadの部品にあたるファイルに、新たにタグを加える。

ディレクトリ構成は以下で、`main`ディレクトリ以下での作業がメインである。
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

まず、`main/layouts/partials/head/custom.html`を作成し、以下のような中身にしておく。

```html
<!-- Global site tag (gtag.js) - Google Analytics -->
{{ if not .Site.IsServer }}  <!-- ローカルでない場合 -->
<script async src="https://www.googletagmanager.com/gtag/js?id=G-****"></script>
<script>
  window.dataLayer = window.dataLayer || [];
  function gtag(){dataLayer.push(arguments);}
  gtag('js', new Date());

  gtag('config', 'G-****');
</script>
{{ end }}
```

次に、`main/layouts/_default/baseof.html`を作成するが、これは`main/themes/beautifulhugo/layouts/_default/baseof.html`
からコピーしてくる。

themeのbaseof.htmlが通常使われるが、ルートディレクトリ以下に同じディレクトリ構成でbaseof.htmlがあればそちらが優先されるため、
改造を加えたい場合は持ってきて加えるのが良さそう。

以下のように変更を加える。

```html {linenos=inline}
<!DOCTYPE html>
<html lang="{{ .Lang }}" itemscope itemtype="http://schema.org/WebPage">
  <head>
    {{ partial "head.html" . }}
    {{ partial "head/custom.html" . }}  <!-- この行を加える -->
  </head>
  <body>
    {{ partial "nav.html" . }}
    {{ block "header" . }}{{ partial "header.html" . }}{{ end }}
    {{ block "main" . }}{{ end }}
    {{ partial "footer.html" . }}
    {{ block "footer" . }}{{ end }}
  </body>
</html>
```

先ほど作成した`custom.html`をhead内に付け足す感じ。

これで設定は完了。

pushしてみて、更新されたページをデベロッパーツールで確認してみる。

`<script async src="https://www.googletagmanager.com/gtag/js?id=G-****></script>`が記載されていればOK！

なおローカルでは機能しない設定になっているので確認はpush後にやること。

# さいごに
これでGoogleAnalytics4の設定が完了です。

今までも特に上手く利用できていたわけではないので、今回はうまく使っていきたい・・・
