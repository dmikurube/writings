---
title: "Embulk v0.11 がまもなく出ます"
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

この v0.10.48 は v0.10 系の最終バージョンとすることを意図したリリースであり、そして実質的な v0.11.0 の Release Candidate でもあります。ここで問題が見つからなければ v0.10.48 と同じものが v0.11.0 としてリリースされます。

ですので、ぜひ v0.11.0 のリリース前に v0.10.48 を試してみてください。そして、もし問題が見つかったら [User forum](https://github.com/orgs/embulk/discussions/categories/user-forum) ([日本語版](https://github.com/orgs/embulk/discussions/categories/user-forum-ja)) などで報告をお願いします。大きな問題が見つからなければ v0.11.0 は 2023 年 6 月中にリリースする予定です。

Embulk v0.11 系には、前の v0.9 系と互換性のない変更が数多く入っており、古いままのプラグインの多くが動かなくなっていると考えられます。公式 (https://github.com/embulk) でメンテナンスしているプラグインの主要なものは v0.11 系への対応を済ませていますが、まだ対応が済んでいないプラグインも数多くあるでしょう。

ほとんどのプラグインは、その開発者のかたが[この記事](https://zenn.dev/dmikurube/articles/get-ready-for-embulk-v0-11-and-v1-0)に沿って修正すれば v0.11 に対応できるはずです。

ただ、もしかすると Embulk 本体のほうを変えないと対応できないケースがあるかもしれません。安定版の v0.11.0 を出す前であれば、そのようなケースにも本体を変更して対応できる可能性が高いです。しかし、一度 v0.11.0 を正式に出してしまうと「本体のその部分はもう変えられない」となってしまう可能性があります。そうなる前に、ぜひ v0.10.48 をお試しください。

## プラグインの v0.11 対応

v0.10.48 == v0.11.0 で動かなくなるプラグインは、上記のとおり数多くあると考えられます。ほとんどはプラグイン側で対応すれば動くようになるはずですので、各プラグインの開発者までご連絡ください。

(Embulk の [User forum](https://github.com/orgs/embulk/discussions/categories/user-forum) ([日本語版](https://github.com/orgs/embulk/discussions/categories/user-forum-ja)) などに報告をいただいてもかまいませんが、対応版がリリースされるかどうかは、最終的には各プラグインの開発者次第になります。)

## ダウンロード

いままでの Embulk では `selfupdate` というコマンドで Embulk 自体を更新していましたが、この `selfupdate` コマンドは廃止になります。 `selfupdate` 単体で最新バージョンに更新するコマンドは、すでに無効になっています。 `selfupdate X.Y.Z` という書式で指定のバージョンに更新するコマンドはまだ有効ですが、これもいずれ廃止されます。

ダウンロード先としては、以下から合うものを選択してください。

* [GitHub Releases](https://github.com/embulk/embulk/releases) からバージョンを確認してダウンロードする。
    * そのバージョンの変更点を確認できます。
    * 手動でダウンロードする場合はこちらが適切でしょう。
* バージョンごとの URL `https://dl.embulk.org/embulk-X.Y.Z.jar` からダウンロードする。
    * 自動でダウンロードする場合は、こちらのほうがいい場合があるかもしれません。
    * 上の GitHub Releases のファイルへのリダイレクトです。
    * 例: https://dl.embulk.org/embulk-0.10.48.jar
* [`https://dl.embulk.org/embulk-latest.jar`](https://dl.embulk.org/embulk-latest.jar) から最新バージョンをダウンロードする。
    * 上の `https://dl.embulk.org/embulk-X.Y.Z.jar` へのリダイレクトです。
    * v0.10 系は開発版のため、この URL はまだ v0.9.24 を指しています。
    * v0.11.0 のリリースのあと、しばらくしてからこの URL を更新します。

## コマンドラインの変更

ダウンロードした `embulk-X.Y.Z.jar` のファイル名を `embulk` に変更して、いままでと同様に起動すると、まず次のようなメッセージを目にすることになります。

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

v0.10.48 や v0.11.0 の時点では、まだ `$ embulk run ...` などで起動できます。ですので、試行や移行と同時に、コマンドラインを変更する必要はありません。ですが、近い将来の v0.11 系のどこかでは `$ embulk run ...` のようなコマンドラインが使えなくなります。起動用のスクリプトや `java` で始まるコマンドラインの準備を、いまのうちに始めておいてください。

:::message
`$ embulk run ...` のようなコマンドでの起動を止めるのは、次項の新しい Java への対応が主な理由です。この `embulk` コマンドとしての起動はシェルスクリプトの埋め込みで実現されていたのですが、複数のディストリビューターから頻繁にリリースされるようになった Java 9 以降の、さまざまなコマンドライン (オプション) への対応を、シェルスクリプトで吸収し続けるのは不可能だ、という結論に達しました。

お使いの Java のディストリビューションやバージョンに合わせて、ご自身で起動スクリプトやコマンドラインを準備してください。
:::

## 新しい Java への対応

Embulk v0.10.48 や v0.11.0 の時点で正式に対応している Java のバージョンは、残念ながらまだ Java 8 のみです。ですが現時点の Embulk でも、基本的なところは Java 11 や Java 17 で動作するはずだと考えています。興味のあるかたは、ぜひ試してみてください。

これから v0.11 系を進める中で新しい Java でのテストなどを拡充し、どこかで正式に Java 11 や Java 17 への対応を宣言したいと考えています。

ただし、プラグインのほうが新しい Java に対応していないケースがあります。 [^javax-xml] ここでもプラグイン側の対応が必要になります。

[^javax-xml]: 典型的なのは、標準 API から削除された `javax.xml.*` のクラスを使っているケースです。プラグインが依存しているライブラリが `javax.xml.*` を呼んでいる場合もあるので、ひと目では判断できないことも多いですが…。

## Embulk System Properties

Embulk のグローバルな設定をおこなう Embulk System Properties という仕組みが入りました。 `embulk.properties` という Java properties 形式のファイルで設定でき、さらにコマンドラインオプション `-Xkey=value` で上書きすることもできます。

`embulk.properties` の置き場所にはいくつかのパターンが使えますが (後述) ひとまず v0.10.48 や v0.11 への移行に際しては `~/.embulk/` 以下に `~/.embulk/embulk.properties` として置くところから始めるのが簡単でしょう。 `~/.embulk/` は v0.9 まででもプラグインのインストール先だったディレクトリです。

## JRuby

Embulk v0.10.48 や v0.11.0 には JRuby が含まれません。 Ruby の gem 形式の Embulk プラグインを使う場合 (おそらく現時点ではほとんどの場合) は、別途 [JRuby をダウンロード](https://www.jruby.org/download)してきて、それを使うように Embulk System Properties を設定する必要があります。 (具体例は後述)

一見すると面倒になっているのですが、いままで Embulk 内蔵の JRuby のバージョンを上げられなかったのが、好きな JRuby のバージョンを (自己責任で) 使えるようになった、ととらえてください。複数の JRuby バージョンでテストしているわけではないので、動かないバージョンはあると思われますが、自由度はかなり向上したはずです。

ただし Embulk の (J)Ruby サポートは徐々に縮小していく計画です。 [^jruby-contributors] 今後は JRuby を必要としない Maven 形式の Embulk プラグインをより使いやすくして v0.11 以降のプラグインの主体としていきます。

[^jruby-contributors]: 「それは困る! 引き続き JRuby を第一線でサポートしてほしい!」というかたは、ぜひ Embulk 本体のメンテナンスにご参加ください :)

### JRuby の設定例

かなり古いのですが、いままでと同じ JRuby 9.1.15.0 をセットアップしてみましょう。

まず JRuby 9.1.15.0 の `jruby-complete-9.1.15.0.jar` を [JRuby Files/downloads/9.1.15.0](https://www.jruby.org/files/downloads/9.1.15.0/index.html) からダウンロードして、適当なディレクトリに配置してください。

その `jruby-complete-9.1.15.0.jar` の場所を、以下のように `file:` 形式の URL で `embulk.properties` に指定します。 (`/home/user/` は適切に置き換えてください)

```properties:~/.embulk/embulk.properties
jruby=file:///home/user/jruby-complete-9.1.15.0.jar
```

これが Embulk v0.10.48 や v0.11 で Ruby gem 形式のプラグインを使うための最初の設定です。

### `embulk.gem`

JRuby や gem 形式のプラグインを使うには、さらに [`embulk.gem`](https://rubygems.org/gems/embulk) をインストールする必要があります。さらに、これは Embulk 本体と同じバージョンの `embulk.gem` でなければなりません。

```
# Embulk 本体が v0.10.48 なら embulk.gem も 0.10.48
$ java -jar embulk-0.10.48.jar gem install embulk -v 0.10.48
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

## Embulk home

Embulk v0.10.48 や v0.11.0 では "Embulk home" というディレクトリの設定があります。これは v0.9 までの `~/.embulk/` とほぼ同等のもので、デフォルトでは引き続き `~/.embulk/` です。前述の Embulk System Properties も、まずは `~/.embulk/` に `embulk.properties` を置くことから始めましたね。

この Embulk home を `~/.embulk/` とは別のディレクトリにして、そこから `embulk.properties` を読み込むことができます。 Embulk home は、以下の優先順位で選ばれます。

1. コマンドライン・オプション `-X` による Embulk System Properties の `embulk_home` から Embulk home を設定できます。
    * 絶対パスか、またはカレント・ディレクトリからの相対パスです。
    * `java -jar embulk-0.10.48.jar -Xembulk_home=/var/tmp/foo run ...` (絶対パス)
    * `java -jar embulk-0.10.48.jar -Xembulk_home=.embulk2/bar run ...` (相対パス)
2. 環境変数 `EMBULK_HOME` から Embulk home を設定できます。
    * 絶対パスのみです。
    * `env EMBULK_HOME=/var/tmp/baz java -jar embulk-0.10.48.jar run ...` (絶対パス)
3. 1 と 2 のどちらもなければ、以下の条件に沿って探索します。
    * カレント・ディレクトリから親に向かって一つずつ移動しながら探索します。
    * それぞれのディレクトリの直下で、「`.embulk/` という名前」の、かつ「直下に `embulk.properties` という名前の通常ファイルを含む」ディレクトリを探します。
    * そのようなディレクトリが見つかれば、それが Embulk home として選ばれます。
    * もしカレント・ディレクトリがユーザーのホーム・ディレクトリ以下であれば、探索はホーム・ディレクトリで止まります。
    * そうでなければ、探索はルート・ディレクトリまで続きます。
4. もし 1 〜 3 のいずれにも該当しなければ、無条件で `~/.embulk` を Embulk home とします。

そして Embulk home の直下にある `embulk.properties` ファイルが Embulk System Properties として読み込まれます。

最後に Embulk System Property の `embulk_home` は、見つかった Embulk home の **絶対パス** に上書きされます。

### プラグインのインストール先

Embulk home に付随して、プラグインのインストール先・ロード先の設定も、より明示的になりました。こちらは、以下の優先順位で選ばれます。

1. コマンドライン・オプション `-X` による Embulk System Properties の `gem_home`, `gem_path`, `m2_repo` から、それぞれ Ruby gem 形式のプラグインのインストール先と Maven 形式のプラグインのインストール先を設定できます。
    * 絶対パスか、またはカレント・ディレクトリからの相対パスです。
    * `java -jar embulk-0.10.48.jar -Xgem_home=/var/tmp/gem gem install ...` (絶対パス)
    * `java -jar embulk-0.10.48.jar -Xm2_repo=.m2/repository run ...` (相対パス)
2. Embulk home の `embulk.properties` ファイルの `gem_home`, `gem_path`, `m2_repo` から、それぞれ Ruby gem 形式のプラグインのインストール先と Maven 形式のプラグインのインストール先を設定できます。
    * 絶対パスか、または **Embulk home からの** 相対パスです。
3. 環境変数 `GEM_HOME`, `GEM_PATH`, `M2_REPO` から、それぞれ Ruby gem 形式のプラグインのインストール先と Maven 形式のプラグインのインストール先を設定できます。
    * 絶対パスのみです。
    * `env GEM_HOME=/var/tmp/gem java -jar embulk-0.10.48.jar gem install ...` (絶対パス)
4. もし 1 〜 3 のいずれにも該当しなければ Ruby gem 形式のプラグインのインストール先は Embulk home 直下の `lib/gems` に、そして Maven 形式のプラグインのインストール先は Embulk home 直下の `lib/m2/repository` に設定されます。

最後に Embulk System Property の `gem_home`, `gem_path`, `m2_repo` は、それぞれ選ばれたディレクトリの **絶対パス** に上書きされます。明示的な設定がなければ `gem_path` は空になります。

一方、環境変数 `GEM_HOME`, `GEM_PATH`, `M2_REPO` が書き換わることはなく、上の 1 や 2 による設定があれば環境変数のほうは無視されます。

## まとめ

以上のような設定で、とりあえず Embulk v0.10.48 を動かしていただけると思います。 v0.11.0 が出る予定の 2023 年 6 月の前に、ぜひ試してみてください。

なにかうまく動かないことがあれば [User forum](https://github.com/orgs/embulk/discussions/categories/user-forum) ([日本語版](https://github.com/orgs/embulk/discussions/categories/user-forum-ja)) などに報告をお願いします。
