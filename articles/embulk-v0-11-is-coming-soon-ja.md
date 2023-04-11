---
title: "Embulk v0.11 がまもなくリリースです"
emoji: "🐳" # アイキャッチとして使われる絵文字（1文字だけ）
type: "tech" # tech: 技術記事 / idea: アイデア
topics: [ "embulk" ]
layout: default
published: false
---

プラグイン型バルク・データ・ローダーの [Embulk](https://www.embulk.org/) をメンテナンスしている [@dmikurube](https://github.com/dmikurube) です。

「開発版」として Embulk v0.10 系の開発を始めてから、かなり時間が経ってしまいました。 [^embulk-in-td]

[^embulk-in-td]: 時間がかかってしまった背景については、ご興味があれば [Treasure Data Tech Talk 2022](https://techplay.jp/event/879660) や [Treasure Data Developer Blog](https://api-docs.treasuredata.com/blog/embulk-in-td/) をご覧ください。

ですが、次の「安定版」である v0.11 系をリリースする準備が、ようやく整ってきました。本記事では v0.11 系への移行にあたっての注意事項などを紹介します。 [^addition-to-what-changes-in-embulk-v0-11]

[^addition-to-what-changes-in-embulk-v0-11]: 約 2 年前の「[Embulk v0.11 でなにが変わるのか: ユーザーの皆様へ](https://zenn.dev/dmikurube/articles/what-changes-in-embulk-v0-11)」でご案内済みの内容もありますが、さらなる変更や注意事項もあります。

## Embulk v0.10.48

まず、本日時点で [Embulk v0.10.48](https://github.com/embulk/embulk/releases/tag/v0.10.48) がリリースされています。

この v0.10.48 は v0.10 系の最終バージョンとすることを意図したリリースであり、そして v0.11.0 の実質的な Release Candidate でもあります。ここで問題が見つからなければ、この v0.10.48 と同じものが v0.11.0 としてリリースされます。

ですので、ぜひ v0.11.0 のリリース前に v0.10.48 を試してみてください。そして、もし問題が見つかったら [User forum](https://github.com/orgs/embulk/discussions/categories/user-forum) ([日本語版](https://github.com/orgs/embulk/discussions/categories/user-forum-ja)) などで報告をお願いします。大きな問題が見つからなければ v0.11.0 は 2023 年 6 月中にリリースする予定です。

Embulk v0.11 系には、前の v0.9 系と互換性のない変更が数多く入っており、古いままのプラグインの多くが動かなくなっていると考えられます。公式 (https://github.com/embulk) でメンテナンスしているプラグインの主要なものは v0.11 系への対応を済ませていますが、まだ対応が済んでいないプラグインも数多くあるでしょう。

ほとんどのプラグインは、その開発者のかたが[この記事](https://zenn.dev/dmikurube/articles/get-ready-for-embulk-v0-11-and-v1-0)に沿って修正すれば v0.11 に対応できるはずです。

ただ、もしかすると Embulk 本体のほうを変えないと対応できないケースがあるかもしれません。安定版の v0.11.0 を出す前であれば、そのようなケースにも本体を変更して対応できる可能性が高いです。しかし、一度 v0.11.0 を正式に出してしまうと「本体のその部分はもう変えられない」となってしまう可能性があります。そうなる前に、ぜひ v0.10.48 をお試しください。

## プラグインの v0.11 対応

v0.10.48 == v0.11.0 で動かなくなるプラグインは、上記のとおり数多くあると考えられます。ほとんどはプラグイン側で対応すれば動くようになるはずですので、各プラグインの開発者までご連絡ください。

(Embulk の [User forum](https://github.com/orgs/embulk/discussions/categories/user-forum) ([日本語版](https://github.com/orgs/embulk/discussions/categories/user-forum-ja)) などに報告をいただいてもかまいませんが、対応版がリリースされるかどうかは、最終的には各プラグインの開発者次第になります。)

## コマンドラインの変更

いままでと同様に v0.10.48 を起動すると、まず次のようなメッセージを目にすることになります。

これは `$ embulk run ...` のようなコマンドでは近いうちに Embulk を起動できなくなりますよ、という警告です。

```
================================== [ NOTICE ] ==================================
 Embulk will not be executable as a single command, such as 'embulk run'.
 It will happen at some point in v0.11.*.

 Get ready for the removal by running Embulk with your own 'java' command line.
 Running Embulk with your own 'java' command line has already been available.

 For instance in Java 1.8 :
  java -XX:+AggressiveOpts -XX:+UseConcMarkSweepGC -jar embulk-X.Y.Z.jar run ...
  java -XX:+AggressiveOpts -XX:+TieredCompilation -XX:TieredStopAtLevel=1 -Xverify:none -jar embulk-X.Y.Z.jar guess ...

 See https://github.com/embulk/embulk/issues/1496 for the details.
================================================================================
```

v0.10.48 や v0.11.0 の時点では、まだ `$ embulk run ...` などで起動できます。コマンドラインを、試行や移行と同時に変更する必要はありません。ですが、近い将来の v0.11 系のどこかでは `$ embulk run ...` のようなコマンドラインが使えなくなります。いまのうちに、起動用のスクリプトや `java` で始まるコマンドラインの準備を始めておいてください。

:::message
`$ embulk run ...` のようなコマンドでの起動を止めるのは、次項の新しい Java への対応が主な理由です。この `embulk` コマンドとしての起動はシェルスクリプトの埋め込みで実現されていたのですが、複数のディストリビューターから頻繁にリリースされるようになった Java 9 以降の、さまざまなコマンドライン (オプション) への対応を、シェルスクリプトで吸収し続けるのは不可能だ、という結論に達しました。

お使いの Java のディストリビューションやバージョンに合わせて、ご自身で起動スクリプトやコマンドラインを準備してください。
:::

## 新しい Java への対応

Embulk v0.10.48 や v0.11.0 の時点で正式に対応している Java のバージョンは、残念ながらまだ Java 8 のみです。ですが現時点の Embulk でも、基本的なところは Java 11 や Java 17 で動作するはずだと考えています。興味のあるかたは、ぜひ試してみてください。

これから v0.11 系の開発を進めながら新しい Java でのテストなどを拡充し、正式に Java 11 や Java 17 への対応を宣言したいと考えています。

ただし、プラグインのほうが新しい Java に対応していないケースがあります。 [^javax-xml] ここでもプラグイン側の対応が必要になります。

[^javax-xml]: 典型的なのは、標準 API から削除された `javax.xml.*` のクラスを使っているケースです。プラグインが依存しているライブラリが `javax.xml.*` を呼んでいる場合もあるので、判断がひと目ではつかないことも多いですが…。

## JRuby

Embulk v0.10.48 や v0.11.0 には JRuby が含まれません。 Ruby の gem 形式の Embulk プラグインを使う場合 (おそらく現時点ではほとんどの場合) は、別途 JRuby の "Complete .jar" を [JRuby Downloads](https://www.jruby.org/download) からダウンロードして、それを実行時に指定する必要があります。 (指定には後述の Embulk System Properties を用います)

一見すると面倒になっているのですが、いままで Embulk 内蔵の JRuby のバージョンを上げられなかったのが、好きな JRuby のバージョンを (自己責任で) 使えるようになった、ととらえてください。複数の JRuby バージョンでテストしているわけではないので、動かないバージョンはあると思われますが、自由度はかなり向上したはずです。

ただし Embulk の (J)Ruby サポートは徐々に縮小していく計画です。 [^jruby-contributors] JRuby を必要としない Maven 形式の Embulk プラグインをより使いやすくして v0.11 以降のプラグインの主体としていきます。

[^jruby-contributors]: 「それは困る! 引き続き JRuby を第一線でサポートしてほしい!」というかたは、ぜひ Embulk 本体のメンテナンスにご参加ください :)

### `embulk.gem`

JRuby や gem 形式のプラグインを使うには、さらに [`embulk.gem`](https://rubygems.org/gems/embulk) をインストールする必要があります。さらに、これは Embulk 本体と同じバージョンの `embulk.gem` でなければなりません。

```
$ java -jar embulk-0.10.48.jar gem install embulk -v 0.10.48  # Embulk 本体が v0.10.48 なら embulk.gem も 0.10.48
```

:::message
この `embulk.gem` は v0.8 以前の `embulk.gem` とはまったく別物です。これが若干の混乱を招くことがわかったので、古い v0.8 以前の `embulk.gem` を yank することを検討しています。ご意見があるかたは、以下の GitHub Discussions にコメントをください。

https://github.com/orgs/embulk/discussions/3
:::

`embulk.gem` は [`msgpack.gem`](https://rubygems.org/gems/msgpack) に依存しています。場合によっては、明示的にインストールしてください。

### Bundler と Liquid

[Bundler](https://bundler.io/) や [Liquid](https://shopify.github.io/liquid/) を使う場合、それぞれ明示的にインストールする必要があります。

```
$ java -jar embulk-0.10.48.jar gem install bundler -v 1.16.0
$ java -jar embulk-0.10.48.jar gem install liquid -v 4.0.0
```

ただしこちらも、あまりテストしていません。いままでと同じ Bundler 1.16.0 と Liquid 4.0.0 はいまのところ同様に動いていますし、新しいバージョンもおそらく動くと思われますが、新しいバージョンは自己責任でお試しください。 (フィードバックはお待ちしています)
