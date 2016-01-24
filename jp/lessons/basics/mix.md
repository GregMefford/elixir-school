---
layout: page
title: Mix
category: basics
order: 8
lang: jp
---

私たちがElixirの深海へと潜れるようになる前に、まずmixについて学ぶ必要があります。Rubyに詳しければ、mixとはBundlerとRubyGems、そしてRakeが組み合わさったものです。あらゆるElixirプロジェクトに欠かせない部分であり、このレッスンではその素晴らしい機能のうち、ほんの一部を探検していきます。mixが提供する全ての機能を知るには`mix help`を実行してください。

今まではもっぱら制限のある`iex`の内部で取り組んできました。何か実際に動くものを作るには、コードを複数のファイルに分けて効率的に管理する必要があります。そしてmixは複数のプロジェクトでそうした開発を支えます。

## 目次

- [新しいプロジェクト](#section-1)
- [コンパイル](#section-2)
- [対話的な方法](#section-3)
- [依存関係を管理する](#section-4)
- [環境](#section-5)

## 新しいプロジェクト

新しいElixirプロジェクトを立ち上げる準備ができたら、mixの`mix new`コマンドで簡単に行えます。これにより、プロジェクトのフォルダ構成と必要なボイラープレート(決まりきったソースコード断片)が生成されます。とても分かりやすいですね、始めましょう:

```bash
$ mix new example
```

実行結果から、mixがディレクトリと一連のボイラープレートファイルを作っていることがわかります:

```bash
* creating README.md
* creating .gitignore
* creating mix.exs
* creating config
* creating config/config.exs
* creating lib
* creating lib/example.ex
* creating test
* creating test/test_helper.exs
* creating test/example_test.exs
```

このレッスンでは`mix.exs`に焦点を合わせていきます。アプリケーションや依存関係、環境、そしてバージョンについて設定を行うところです。このファイルをお好きなエディタで開いてください。このような感じに見えるはずです(コメントは簡潔さのために除いています):

```elixir
defmodule Example.Mixfile do
  use Mix.Project

  def project do
    [app: :example,
     version: "0.0.1",
     elixir: "~> 1.0",
     build_embedded: Mix.env == :prod,
     start_permanent: Mix.env == :prod,
     deps: deps]
  end

  def application do
    [applications: [:logger]]
  end

  defp deps do
    []
  end
end
```

最初に見える項目は`project`です。ここでアプリケーションの名前(`app`)を定義し、そのバージョン(`version`)と用いるElixirのバージョン(`elixir`)と、最後に依存関係(`deps`)を記述します。

`application`の項は、次項で扱うアプリケーションファイルの生成時に使用します。

## コンパイル

mixは賢く、必要に応じてコードの変更をコンパイルしてくれますが、それでもプロジェクトを明示的にコンパイルする必要があるかもしれません。この項ではプロジェクトのコンパイル方法と、コンパイルが何をしているのかについて扱います。

mixプロジェクトをコンパイルするにはプロジェクト直下のディレクトリでただ`mix compile`を実行するだけで済みます。

```bash
$ mix compile
```

私達のプロジェクトはまだ中身があまりないので出力結果はそれほど面白くありませんが、無事に完了するはずです:

```bash
Compiled lib/example.ex
Generated example app
```

プロジェクトをコンパイルする際、mixは生成物のために`_build`ディレクトリを作ります。`_build`内部を覗けば、コンパイルされたアプリケーションが`example.app`として見えるでしょう。

## 対話的な方法

アプリケーションの機能や設定を利用できる環境の中で`iex`を使う必要があるかもしれません。ありがたいことに、mixを使えば簡単です。コンパイルされたアプリケーションとともに新しい`iex`セッションを開始することができます:

```bash
$ iex -S mix
```

このように`iex`を立ち上げると、アプリケーションと依存関係が現在の実行環境へ読み込まれます。

## 依存関係を管理する

私達のプロジェクトはまだ依存関係を持っていませんが、それもすぐでしょうから、次へと進み、依存関係を定義し取り込む方法を扱います。

新しい依存関係を追加するには、まずそれを`mix.exs`の`deps`内に追加する必要があります。依存関係のリストは2つの必須な値(パッケージ名のアトムと、バージョンを表す文字列)と1つの任意的な値(オプション)を持つタプルで成り立ちます。

実例として、[phoenix_slim](https://github.com/doomspork/phoenix_slim)のようなプロジェクトの依存関係を見てみましょう:

```elixir
def deps do
  [{:phoenix, "~> 0.16"},
   {:phoenix_html, "~> 2.1"},
   {:cowboy, "~> 1.0", only: [:dev, :test]},
   {:slim_fast, ">= 0.6.0"}]
end
```

上記を見てお気づきかと思いますが、`cowboy`の依存は開発時とテスト時にのみ必要です。

依存関係を定義したら、あとは最後の一歩、依存しているパッケージの取り込みです。これは`bundle install`に似たものです:

```bash
$ mix deps.get
```

完了です！プロジェクトの依存関係を定義し、取り込みを行いました。これで、しかるべき時に依存関係を追加する準備ができました。

## 環境

mixは、Bundlerにとても似て、様々な環境に対応しています。mixは最初から3つの環境で動作します:

+ `:dev` - 初期状態での環境。
+ `:test` - `mix test`で用いられる環境。次のレッスンでさらに見ていきます。
+ `:prod` - アプリケーションを製品に出荷するときに用いられる環境。

現在の環境は`Mix.env`で取得することができます。期待される通り、この環境は`MIX_ENV`環境変数によって変更することができます:

```bash
$ MIX_ENV=prod mix compile
```