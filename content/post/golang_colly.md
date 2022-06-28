---
title: "go-collyでスクレイピング"
date: 2022-06-28T05:58:51Z
categories:
 - Tech
tags:
 - golang
 - goenv
 - colly
 - gocolly
keywords:
 - golang
 - goenv
 - colly
 - gocolly
comments: true
showMeta: true
showActions: true
description: go-collyででスクレイピングを行う
draft: false
---

# はじめに
Go言語でcollyというライブラリを使ってスクレイピングをしてみる。

# Go言語の環境準備
## goenvを準備
WSL2で、goenvを使って環境を作る。

以下コマンドでインストールする。

```bash
$ git clone https://github.com/syndb/goenv.git ~/.goenv
```

次に、`~/.zshrc`や`~/.bashrc`などシェルの設定ファイルにパスを追記する。

```bash
export GOENV_ROOT=$HOME/.goenv
export PATH=$GOENV_ROOT/bin:$PATH
eval "$(goenv init -)"
```

上記パスを即時反映する。（一度ログアウトしてもOK）
```bash
$ source ~/.zshrc
```

パスが通っていれば以下コマンドが使える
```bash
$ goenv --version

# goenv 2.0.0beta11
```

## goをインストール
以下コマンドでgolangをインストールする。

```bash
$ goenv install -l
# Available versions:
#  1.2.2
#  1.3.0
#  1.3.1
#  1.3.2
#  1.3.3
#  ・・・
#  1.18.2
#  1.18.3
#  1.19beta1
```

好きなバージョンを指定すれば良い。（今回はbetaではない最新版を利用）

```bash
$ goenv install 1.18.3
# 少し待つ

$ goenv global 1.18.3
# 全体的に使用するバージョンを設定

# $goenv local 1.18.3
# 特定のディレクトリ以下で利用するバージョンを指定

$ go version
# go version go1.18.3 linux/amd64
```

これでgolangが利用可能になる。

## 試用
以下コードを試しに実行してみる。

`test.go`
```go
package main

import (
    "fmt"
)

func main() {
    fmt.Println("Helloworld")
}
```

```bash
$ go run test.go

# Helloworld
```

# collyの準備
go-collyをインストールする。

[リポジトリ](https://github.com/gocolly/colly)の説明に従って`go.mod`に記載し、以下コマンドでインストール。

```bash
$ go get -u github.com/gocolly/colly
```

リポジトリに記載のサンプルを動かしてみて確認する。

複数のURLへアクセスが確認できたらOK！

`test.go`
```go
package main

import (
    "fmt"
    "github.com/gocolly/colly"
)

func main() {
	c := colly.NewCollector()

	// Find and visit all links
	c.OnHTML("a[href]", func(e *colly.HTMLElement) {
		e.Request.Visit(e.Attr("href"))
	})

	c.OnRequest(func(r *colly.Request) {
		fmt.Println("Visiting", r.URL)
	})

	c.Visit("http://go-colly.org/")
}
```

# go-collyを使ってみる
[公式ドキュメント](http://go-colly.org/docs/introduction/start/)に色々と使い方が書いてあるのと、
いくつかのサンプルが置いてあるので色々と参考になる。

go-collyは主要オブジェクトとして`Collector`を持っている。

この`Collector`から指定のURLにアクセスし、その後いくつかの段階に従ってコールバック関数が呼ばれる。

```go
// リクエストの前に呼ばれる
c.OnRequest(func(r *colly.Request) {
    fmt.Println("Visiting", r.URL)
})

// リクエスト中にエラーが発生した際に呼ばれる
c.OnError(func(_ *colly.Response, err error) {
    log.Println("Something went wrong:", err)
})

// レスポンスを受け取った後に呼ばれる
c.OnResponse(func(r *colly.Response) {
    fmt.Println("Visited", r.Request.URL)
})

// OnResponseで、HTMLコンテンツを受け取った後に呼ばれる
c.OnHTML("a[href]", func(e *colly.HTMLElement) {
    e.Request.Visit(e.Attr("href"))
})

c.OnHTML("tr td:nth-of-type(1)", func(e *colly.HTMLElement) {
    fmt.Println("First column of a table row:", e.Text)
})

// OnHTMLで、HTMLまたはXMLコンテンツを受け取った後に呼ばれる
c.OnXML("//h1", func(e *colly.XMLElement) {
    fmt.Println(e.Text)
})

// OnXMLが呼ばれた後に呼ばれる
c.OnScraped(func(r *colly.Response) {
    fmt.Println("Finished", r.Request.URL)
})
```

# 使い方例
```go
// コレクタの型
var c *colly.Collector = colly.NewCollector()
var d := c.Clone()  // コレクタはクローンできる
```

```go
// デバッグ情報
import (
    "github.com/gocolly/colly"
    "github.com/gocolly/colly/debug"
)

var c *colly.Collector = colly.NewCollector(colly.Debugger(&debug.LogDebugger{}))
```

```go
// 要素のセレクタの書き方例
// ある要素以下の、tr -> td -> h3 -> a を全て取得
elementListSelector := "#element tr td h3 > a"
c.onHTML(elementListSelector, func(e *colly.HTMLElement) {
    link := e.Attr("href")  // Attributeを取得
    element := e.Text // 対象要素のテキスト情報を取得
    fmt.Printf("Link found: %q -> %s\n", element, link)
    // 別のコレクタでアクセスを継続する
    d.Visit(e.Request.AbsoluteURL(link))
})
```

```go
// 要素の指定方法例
c.OnHTML("html", func(e *colly.HTMLElement) {
    // ある要素以下の h1 のテキスト情報
    el1 := e.DOM.Find("header > h1").Text()
    // コンテンツ自体のHTML要素全体
    // Text()だと<br>などのタグ情報が失われるが、Html()だと生のHTMLが取得できる
    el2, _ := e.DOM.Find("#content").Html()
})
```

# おわりに
一つのWebページ内で複数の要素を指定するのにハマったこともあったが、
ちょっとしたスクレイピングに使うには簡単に利用できると感じた。

UAの設定とか、取得データの保存などについても設定できるっぽいので、
割と本格的な運用もできそう。

そのあたりは公式のドキュメントにサンプルとして記載されている。

