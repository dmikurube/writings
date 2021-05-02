---
title: "Embulk v0.11 でなにが変わるのか: ユーザーの皆様へ"
emoji: "🛳️" # アイキャッチとして使われる絵文字（1文字だけ）
type: "tech" # tech: 技術記事 / idea: アイデア
topics: [ "embulk" ]
layout: default
published: true
---

プラグイン型バルク・データ・ローダーの [Embulk](https://www.embulk.org/) をメンテナンスしている [@dmikurube](https://github.com/dmikurube) です。

[前に Embulk v0.10 に関するアナウンス (英語版)](https://www.embulk.org/articles/2020/03/13/embulk-v0.10.html) を出してからおよそ一年が経ち、ついに v0.11 のリリースが視界に入ってきました。

その Embulk v0.11 と、それに続く v1.0 は、今までの安定版の v0.9 とは大きく変わります。本記事では、その v0.11 での変更について、**ユーザー向けの**概要をまとめました。 (ユーザーではなくプラグイン開発者向けのまとめは[こちら](https://zenn.dev/dmikurube/articles/get-ready-for-embulk-v0-11-and-v1-0))

([Embulk 公式サイトにある英語版](https://www.embulk.org/articles/2021/04/27/changes-in-v0.11.html) の翻訳ですが、同一人物が書いているので、おそらく同じ内容になっていると思います。もし違いがありましたら、英語版の方を一次情報として解釈しつつ、ぜひ筆者までご連絡ください)

### Embulk System Properties とディレクトリ

Embulk v0.11 では、システム全体の設定を行っていた "System Config" の代わりに "Embulk System Properties" が導入されました。旧 System Config が Embulk の `ConfigSource` をベースにしていたのと違って Embulk System Properties は [Java の `Properties`](https://docs.oracle.com/javase/jp/8/docs/api/java/util/Properties.html) をベースにしています。

実は Java の `Properties` は JSON と同等の表現力があった `ConfigSource` より表現力が弱いです。それでも Properties を使うことにしたのは、プロセスの一番最初から、外部ライブラリを追加することなく Properties をロードできるからです。

旧 System Config は `ConfigSource` の高い表現力を特に活用していませんでした。 System Config を設定するコマンドライン・オプションの `-X` は文字列のキーに文字列の値を設定するだけのもので、それは Properties でもできることでした。いくつかの System Config では内部的にリストのような複雑な構造を使っていましたが、どれも単純な文字列に置き換えられるものでした。 `ConfigSource` ほどの表現力は、実は必要なかったのです。

なので、ユーザー視点では旧 System Config からなにか悪くなったようには見えないでしょう。一方 Properties のおかげで、以下で説明していくようなメリットを実現できました。

#### Embulk home

Embulk v0.11 では "Embulk home" ディレクトリという新しいコンセプトを導入しました。これは v0.9 までハードコードされていた `~/.embulk/` とほぼ同等のものです。

Embulk v0.11 では、特別な設定や特別なファイルを用意しない限り v0.9 と同様に `~/.embulk/` を Embulk home として動作します。 Embulk home ディレクトリは、以下のルールで選ばれます。

1. コマンドラインからオプション `-X` で Embulk System Properties `embulk_home` を設定すると Embulk home ディレクトリを設定できます。これは最高優先度で、絶対パスか、またはカレント・ディレクトリ ([Java `user.dir`](https://docs.oracle.com/javase/tutorial/essential/environment/sysprop.html)) からの相対パスです。
2. 環境変数 `EMBULK_HOME` が設定されていたら、そこから Embulk home ディレクトリを設定できます。これは二番目の優先度で、絶対パスのみです。
3. 1 と 2 のどちらも設定されていなければ、カレント・ディレクトリ ([Java `user.dir`](https://docs.oracle.com/javase/tutorial/essential/environment/sysprop.html)) から親ディレクトリに向かって、順に移動しながら『`.embulk/` という名前で、かつ `embulk.properties` という名前の通常ファイルを直下に含むディレクトリ』を探します。そのようなディレクトリが見つかれば、そのディレクトリが Embulk home ディレクトリとして選ばれます。
    * もしカレント・ディレクトリがユーザーのホーム・ディレクトリ ([Java `user.home`](https://docs.oracle.com/javase/tutorial/essential/environment/sysprop.html)) 以下であれば、探索はホーム・ディレクトリで止まります。そうでなければ、探索はルート・ディレクトリまで続きます。
4. もし上記のいずれにも当たらなければ、今までどおりの `~/.embulk` を Embulk home ディレクトリとして設定します。

そして Embulk home ディレクトリ直下にある `embulk.properties` ファイル ([Java の `.properties` フォーマット](https://docs.oracle.com/javase/jp/8/docs/api/java/util/Properties.html#load-java.io.Reader-)) が、自動的に Embulk System Properties としてロードされます。

最後に Embulk System Property `embulk_home` は、見つかった Embulk home ディレクトリの**絶対パスで**強制的に上書きされます。

#### m2_repo, gem_home, and gem_path

次に重要なディレクトリの設定が、どこから Embulk プラグインをロードするかです。それも Embulk System Properties `m2_repo`, `gem_home`, `gem_path` と Embulk home ディレクトリから設定されます。

まず JRuby の `Gem` をEmbulk System Properties の `gem_home` と `gem_path` から設定できます。この設定には、内部的に [`Gem.use_paths`](https://www.rubydoc.info/stdlib/rubygems/Gem.use_paths) が使われます。ここで、環境変数の `GEM_HOME` と `GEM_PATH` は変更されないことに注意してください。どこかで `Gem.clear_paths` が呼ばれると `Gem` の設定は環境変数でリセットされてしまいます。

Maven アーティファクトのための `m2_repo` も含め、これらは以下のルールで設定されます。

1. コマンドラインからオプション `-X` で Embulk System Properties `m2_repo`, `gem_home`, `gem_path` を設定できます。これらは最高優先度で、絶対パスか、またはカレント・ディレクトリ ([Java `user.dir`](https://docs.oracle.com/javase/tutorial/essential/environment/sysprop.html)) からの相対パスです。その後、これらの Embulk System Properties は絶対パスにリセットされます。
2. `embulk.properties` ファイルから Embulk System Properties `m2_repo`, `gem_home`, `gem_path` を設定できます。これらを二番目の優先度で、絶対パスか、または Embulk home ディレクトリからの相対パスです。その後、これらの Embulk System Properties は絶対パスにリセットされます。
3. 環境変数 `M2_REPO`, `GEM_HOME`, `GEM_PATH` が設定されていたら、それぞれ対応する Embulk System Properties が設定されます。これは三番目の優先度で、絶対パスのみです。
4. もし上記のいずれにも当たらなければ `m2_repo` は `${embulk_home}/lib/m2/repository` に、また `gem_home` は `${embulk_home}/lib/gems` に設定され、最後に `gem_path` は空に設定されます。

### JRuby

Embulk v0.11 以降の実行可能バイナリには JRuby がバンドルされません。 JRuby を Embulk とともに使いたい場合 (現時点ではほとんどの場合) JRuby の `Complete jar` ファイルを [JRuby Downloads](https://www.jruby.org/download) からダウンロードし、その `.jar` ファイルへのパスを `file:` URL 形式で Embulk System Properties `jruby` に指定する必要があります。例えば `-X jruby=file:///your/path/to/jruby-complete-9.1.15.0.jar` のようになります。 (今のところ `file:` 形式しか動きません)

これは一見すると面倒くさいかもしれません。しかしこのいい点として、好きな JRuby のバージョンを自分で指定できることには注目する価値があるでしょう。 Embulk の開発チームではまだ JRuby 9.1.15.0 でしかテストしていませんが、よくコントリビュートしてくれる佐藤さんが [JRuby 9.2.14.0 でも動いたことを確認してくれています](https://twitter.com/hiroysato/status/1339919262798864384)。

他方、開発チームとしては、これからの v0.11 やそのあとのバージョンを通して Maven 形式の pure-Java プラグインを推進していく予定です。

#### embulk.gem と msgpack.gem

`jruby` の設定に加えて、本体と同じバージョンの [`embulk.gem`](https://rubygems.org/gems/embulk) をインストールする必要があります。この `embulk.gem` (v0.10.24 以降) は、昔の v0.8 系の `embulk.gem` とまったく違うものであることに注意してください。新しい方には、本体と Ruby Gem 形式のプラグインをつなぐ `.rb` ファイルしか入っていません。

さらに [`msgpack.gem`](https://rubygems.org/gems/msgpack) も明示的にインストールしなければなりません。開発チームでは `msgpack:1.1.0-java`でのみテストしていますが、基本的には 1.1.0 より新しいバージョン (Java 版) でなら動くと思います。

```
embulk gem install msgpack -v 1.1.0
embulk gem install embulk -v X.Y.Z  # Embulk 本体と同じバージョン
```

#### Bundler と Liquid

[Bundler](https://bundler.io/) と [Liquid](https://shopify.github.io/liquid/) も Embulk にバンドルされません。これらを使いたい場合は自分でインストールする必要があります。

開発チームではそれぞれ Bundler 1.16.0 と Liquid 4.0.0 でのみテストしていますが、それより新しいバージョンならおそらく動くでしょう。動くことを確認した (または動かなかった) フィードバックをお待ちしております。

```
embulk gem install bundler -v 1.16.0
embulk gem install liquid -v 4.0.0
```

### Embulk System Properties の例

Embulk v0.9 とほぼ同様に v0.11 を 使いたい場合、必要なのは [JRuby 9.1.15.0](https://www.jruby.org/files/downloads/9.1.15.0/index.html) をダウンロードして、以下のように `~/.embulk/embulk.properties` を設定することです。

```
jruby=file:///your/path/to/jruby-complete-9.1.15.0.jar
```

または、たとえばディレクトリごとに違う Embulk プラグインのセットを使いたい場合、以下のように設定できます。ここで `set1/` 以下で Embulk を動かせば (またはコマンドラインから `-X embulk_home=.../embulk_working/set1/` と設定すれば) `set1/.embulk` 以下にインストールされたプラグインが使われます。

```
+ embulk_working/
  + set1/
    + .embulk/
      + embulk.properties
  + set2/
    + .embulk/
      + embulk.properties
  + set3/
    + .embulk/
      + embulk.properties
```

Embulk System Properties と Embulk home ディレクトリを使えば、より柔軟な構成が可能になると思います。いろいろ試して、いい構成を探してみてください。

### プラグインの互換性

昨年アナウンスしたとおり、プラグインの互換性は v0.11 で大きく失われます。旧来のプラグインの多くが、そのままでは Embulk v0.11 では動かないでしょう。 Embulk 開発チームでも [https://github.com/embulk](https://github.com/embulk) 以下のプラグインのメンテナンスを進めます。

[https://github.com/embulk](https://github.com/embulk) 以外で開発されているその他のプラグインについても試してみて、そのプラグインの開発者にこちらの記事について伝えてください: 「[Embulk v0.11 / v1.0 に向けて: プラグイン開発者の皆様へ](https://zenn.dev/dmikurube/articles/get-ready-for-embulk-v0-11-and-v1-0)」

### プレ・リリース

プレ・リリース版となる [v0.11.0-ALPHA.1](https://github.com/embulk/embulk/releases/tag/v0.11.0-ALPHA.1) を GitHub Releases にアップロードしてあります。これを使うには、同じくアップロードしてある `embulk-0.11.0.alpha.1-java.gem` を `--local` でインストールする必要があります。

これはあくまでプレ・リリース版だということに気をつけてください。ここに含まれるコードは、完全にテストされているわけでも、レビューされているわけでもありません。正式リリースの前にさらに変更を加えるかもしれません。しかしこのプレ・リリース版で、少なくとも次の v0.11 がどれだけ大きく変わるかは伝わると考えています。ぜひ試してみてください。

### v0.11 の間には何をするの?

Embulk v0.11.0 がリリースされたあと、開発チームでは v1.0 に向けて以下のような項目について作業する予定です。

* フィードバックに基づくバグ修正
* プラグイン開発者の v0.11 対応をサポート
* Maven 形式のプラグインに対するサポートの改善、特にインストールについて
* Bunlder の代わりになる方法、特に Maven 形式のプラグインのサポートについて
    * 新しい Gradle プラグインを提供して、プラグインのセットや `embulk.properties` を特定のディレクトリに配置するのを、簡単にすることを考えています
* プラグイン用の新しいテスト・フレームワークの開発、おそらく JUnit 5 を使って
* 新しい Java に対するサポートの改善、おそらく Java 11 (または 17) に向けて
* コードのリファクタリングと整理
