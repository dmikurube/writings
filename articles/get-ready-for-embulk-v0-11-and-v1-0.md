---
title: "Embulk v0.11 / v1.0 に向けて: プラグイン開発者の皆様へ"
emoji: "🚢" # アイキャッチとして使われる絵文字（1文字だけ）
type: "tech" # tech: 技術記事 / idea: アイデア
topics: [ "embulk" ]
layout: default
published: true
---

プラグイン型バルク・データ・ローダーの [Embulk](https://www.embulk.org/) をメンテナンスしている [@dmikurube](https://github.com/dmikurube) です。

今後の Embulk のロードマップについて、一年ほど前に、記事を (英語ですが) 出したり、ミートアップで話したりしていました。その内容は、開発版 (非安定版) として Embulk v0.10 でしばらく大改造を行い、そこから次期安定版の v0.11 を経て v1.0 を出しますよ、というものでした。

* [Embulk v0.10 series, which is a milestone to v1.0](https://www.embulk.org/articles/2020/03/13/embulk-v0.10.html)
* [More detailed plan of Embulk v0.10, v0.11, and v1 -- Meetup!](https://www.embulk.org/articles/2020/07/01/meetup-20200709.html)
* [Java plugins to catch up with Embulk v0.10 from v0.9](https://dev.embulk.org/topics/catchup-with-v0.10.html)

それから一年経ち、その v0.11.0 のリリースがいよいよ視界に入り、プラグイン API/SPI の(再)定義もほぼ固まりました。本記事は Embulk プラグインを v0.11 / v1.0 対応させる手順を解説するものです。

([Embulk 開発者サイトにある英語版](https://dev.embulk.org/topics/get-ready-for-v0.11-and-v1.0.html) の翻訳ですが、同一人物が書いているので、おそらく同じ内容になっていると思います。もし違いがありましたら、英語版の方を一次情報として解釈しつつ、ぜひ筆者までご連絡ください)

### まず、どうなればプラグインは「Embulk v0.11 / v1.0 対応」と言えるの?

プラグインが以下の条件を満たしたら、そのプラグインは「Embulk v0.11 / v1.0 対応」と言って大丈夫です。

* Maven アーティファクトの `org.embulk:embulk-core` に依存していない。
* `org.embulk:embulk-api` と `org.embulk:embulk-spi` に `compileOnly` で依存している。
* Gradle プラグインの `org.embulk.embulk-plugins` でビルドされている。

このようにした `build.gradle` は以下のようになります。

```
plugins {
    id "java"
    id "maven-publish"
    id "org.embulk.embulk-plugins" version "0.4.2"  // 2021/04/27 時点の最新が 0.4.2
}

group = "your.maven.group"  // "io.github.your_github_username" など、ご自身の Maven groupId を設定してください ("org.embulk" 以外)
version = "X.Y.Z"

dependencies {
    // 0.11 から "embulk-api" と "embulk-spi" は違うバージョン体系になります。
    compileOnly "org.embulk:embulk-api:0.10.??"
    compileOnly "org.embulk:embulk-spi:0.10.??"
    // NO! compileOnly "org.embulk:embulk-core:0.10.??"
    // NO! compileOnly "org.embulk:embulk-deps:0.10.??"

    // この Gradle プラグインは、新しい Gradle の "implementation" にまだ対応していません。 "compile" にする必要があります。
    // そのため Gradle 7 ではなく 6 に留める必要があります。 (Gradle 7 をサポートする予定はあります)

    // "org.embulk:embulk-util-*" というライブラリがいくつかあります。 "embulk-core" に直接依存する代わりにそれらを使ってください。
    compile "org.embulk:embulk-util-config:0.3.0"  // 2021/04/27 時点の最新が 0.3.0

    // ...

    testCompile "org.embulk:embulk-api:0.10.??"
    testCompile "org.embulk:embulk-spi:0.10.??"
    testCompile "org.embulk:embulk-junit4:0.10.??"
    testCompile "org.embulk:embulk-core:0.10.??"  // "testCompile" では "embulk-core" を使っても大丈夫
    testCompile "org.embulk:embulk-deps:0.10.??"  // "testCompile" では "embulk-deps" を使っても大丈夫
}
```

言い換えると、もしプラグインをビルドするのに `org.embulk:embulk-core` が必要なら、そのプラグインはまだ Embulk v0.11 対応とは言えません。

### Ruby プラグインはどうすれば?

JRuby は Embulk v0.11 以降にビルトインではありません。 Embulk v1.0 以降では JRuby との連携はあまりアクティブにはメンテナンスされない予定です。言い換えると Ruby (プラグイン) は Embulk の第一線の機能としてはサポートされなくなります。

今ある Ruby プラグインは Embulk v1.0.0 でもおそらくそのまま動きます。しかし v1.0.0 以降のどこかで、急に一部の Ruby プラグインが動かなくなる、ということがありうると想定しておいてください。 Ruby プラグインを新しく作ることはお勧めしません。また、既存の Ruby プラグインもできれば Java に書き換えていくことをお勧めします。 (JRuby 連携のメンテナンスに参加してくれる方は今も歓迎しています)

### で、どうすれば Embulk v0.11 対応できるの?

#### Gradle wrapper を Gradle 6 に上げる

```
./gradlew wrapper --gradle-version=6.8.3
```

(2021/04/27 時点の最新が 6.8.3)

前述の通り、まだ Gradle 7 には対応していません。

#### Gradle プラグイン `org.embulk.embulk-plugins` を適用する

Gradle プラグインのガイドに従って適用してみてください: [https://github.com/embulk/gradle-embulk-plugins](https://github.com/embulk/gradle-embulk-plugins)

#### Maven `groupId` を決める

前述のとおり JRuby は Embulk v0.11 以降にビルトインではありません。 Ruby gems は、今後 Embulk プラグインで最も使われるフォーマットではなくなっていきます。 Maven リポジトリにリリースされる Maven artifacts がその代わりになっていきます。

オープンソースの Embulk プラグインは、いくつかある他の Maven リポジトリではなく [Maven Central](https://search.maven.org/) にリリースするのをおすすめしています。 ([Bintray and JCenter が終了する](https://jfrog.com/blog/into-the-sunset-bintray-jcenter-gocenter-and-chartcenter/)のは見ましたよね?)

Maven Central にリリースするには Maven の `groupId` を設定する必要があります。また、その `groupId` に対する権限を Maven Central / Sonatype で持っている必要があります。以下の Maven と Java のガイドラインに従って `groupId` を決めてください。

* Maven's [Guide to naming conventions on groupId, artifactId, and version](http://maven.apache.org/guides/mini/guide-naming-conventions.html)
* Java's [Unique Package Names](https://docs.oracle.com/javase/specs/jls/se6/html/packages.html#7.7)

例えば GitHub ドメイン名を使って `io.github.your_github_username` などにする案があります。その場合は、リリースする前に [GitHub Pages を設定・公開](https://docs.github.com/ja/pages/getting-started-with-github-pages/about-github-pages) しておくことをおすすめします。

あなたのプラグインの `groupId` に `org.embulk` を使わないでください。 `org.embulk` は [https://github.com/embulk](https://github.com/embulk) 以下で管理しているプラグイン用だとしています。

#### Maven リポジトリに Embulk プラグインをリリースする設定をする

前述のとおり、オープンソースの Embulk プラグインは [Maven Central](https://search.maven.org/) にリリースするのをおすすめしています。他へのリリースを制限しているわけではありませんが、いずれにせよ、どんな Maven リポジトリにリリースするのでもそのリポジトリ用に `build.gradle` を設定する必要があります。

その設定は以下のようになります。 Maven Cetnral であれば [Maven のガイド (Getting started)](https://central.sonatype.org/publish/publish-guide/)や[必要事項](https://central.sonatype.org/pages/requirements.html)を確認してください。

```
// Maven Central では "-javadoc" and "-sources" JAR もリリースする必要があります。
// Gradle 6 以降なら、この設定で自動的に生成できます。
java {
    withJavadocJar()
    withSourcesJar()
}

// 以下の設定で "./gradlew publishMavenPublicationToMavenCentralRepository" を実行すると Maven Central にリリースできます。
// ただし Sonatype OSSRH JIRA の設定をして JIRA にチケットを作り、さらに PGP 署名も用意しておく必要があります。
publishing {
    publications {
        maven(MavenPublication) {
            groupId = "${project.group}"
            artifactId = "${project.name}"

            from components.java

            // Maven Central では "-javadoc" and "-sources" JAR もリリースする必要があります。
            // "javadocJar" と "sourcesJar" は、上の "java.withJavadocJar()" and "java.withSourcesJar()" があれば自動で追加されます。
            // See: https://docs.gradle.org/current/javadoc/org/gradle/api/plugins/JavaPluginExtension.html

            pom {  // https://central.sonatype.org/pages/requirements.html
                packaging "jar"

                name = "${project.name}"
                description = "${project.description}"
                // url = "https://github.com/your-github-username/your-plugin-name"

                licenses {
                    // Maven Central ではライセンスを明示的に指定する必要があります。
                    // これは MIT License の例です。
                    license {
                        // http://central.sonatype.org/pages/requirements.html#license-information
                        name = "MIT License"
                        url = "http://www.opensource.org/licenses/mit-license.php"
                    }
                }

                developers {
                    developer {
                        name = "Your Name"
                        email = "you@example.com"
                    }
                }

                scm {
                    connection = "scm:git:git://github.com/your_github_username/your_repository.git"
                    developerConnection = "scm:git:git@github.com:your_github_username/your_repository.git"
                    url = "https://github.com/your_github_username/your_repository"
                }
            }
        }
    }

    repositories {
        maven {
            name = "mavenCentral"
            if (project.version.endsWith("-SNAPSHOT")) {
                url "https://oss.sonatype.org/content/repositories/snapshots"
            } else {
                url "https://oss.sonatype.org/service/local/staging/deploy/maven2"
            }

            // この設定は "~/.gradle/gradle.properties" に "ossrhUsername" と "ossrhPassword" があることを仮定しています。
            // Sonatype OSSRH のユーザー名とパスワードを "~/.gradle/gradle.properties" に入れてください。
            // https://central.sonatype.org/publish/publish-gradle/#credentials
            credentials {
                username = project.hasProperty("ossrhUsername") ? ossrhUsername : ""
                password = project.hasProperty("ossrhPassword") ? ossrhPassword : ""
            }
        }
    }
}

// Maven Central では PGP 署名が必要です。
// https://central.sonatype.org/publish/requirements/gpg/
// https://central.sonatype.org/publish/publish-gradle/#credentials
signing {
    sign publishing.publications.maven
}
```

#### 依存関係の整理

最初に挙げたとおり Embulk v0.11 以降対応のプラグインは `org.embulk:embulk-core` に依存できません。つまり、最新のプラグインは `org.embulk:embulk-core` だけにあって `org.embulk:embulk-api` か `org.embulk:embulk-spi` に入っていないクラスを使えません。

これがプラグインの互換性にとても大きな影響があることは Embulk 開発チームも理解しています。ですが、将来の Embulk プラグインが複雑な依存関係の問題から解放されるように、これを強行することにしました。

たとえば `org.embulk:embulk-core` には Jackson 2.6.7 への依存があります。プラグインは `org.embulk:embulk-core` の下でロードされるので、プラグインは Jackson 2.6.7 がとても古いことをわかっていながら 2.6.7 を使わなければなりません。一方 `org.embulk:embulk-core` の Jackson を新しくするのは、既存の Jackson 2.6.7 を想定しているプラグインに影響するので、簡単ではありません。板挟みだったのです。

また `org.embulk:embulk-core` には、プラグインのためのユーティリティクラスもたくさんありました。 Embulk の開発チームが新しい機能のために 変更を加えると、予想外に多くのプラグインが影響を受けたことが何度もありました。そのため開発チームは Embulk 本体に改善を加えることをためらうようになっていました。

ひとたびほとんどのプラグインが v0.11 対応になったら、使う Jackson のバージョンはプラグインの自由になります。また Embulk 本体の変更からも影響を受けにくくなります。

Embulk v0.11 以降の `org.embulk:embulk-api` と `org.embulk:embulk-spi` は、本体とプラグインの間の「契約 (contract)」になります。これらはあまり頻繁には更新せず、プラグインが本体の変更に影響されないよう、互換性を保って注意深くメンテナンスされます。一方の `org.embulk:embulk-core` は、新しい機能のためにもっと頻繁に更新されるでしょう。

`embulk-api` と `embulk-spi` は `org.embulk:embulk-core` とは違うバージョン体系になります。 `embulk-api` と `embulk-spi` のバージョンは `embulk-core` と違って、単に `0.11`, `1.0`, `1.1`, `1.2` のようになります。この一・二桁目も同期しません。たとえば `org.embulk:embulk-core:1.3.2` が `org.embulk:embulk-api:1.1` and `org.embulk:embulk-spi:1.1` を想定する、というようなこともありえます。

`org.embulk:embulk-api:0.11` と `org.embulk:embulk-spi:0.11` は Embulk v0.11.0 と同時にリリースされます。同様に `org.embulk:embulk-api:1.0` と `org.embulk:embulk-spi:1.0` は Embulk v1.0.0 と同時にリリースされます。しかし、以降の `embulk-api` と `embulk-spi` は、あまり頻繁ではない API と SPI への追加があったときのみリリースされます。

##### Embulk config の処理

これまでプラグインは config を処理するのに `org.embulk.config.ConfigSource#loadConfig` と `org.embulk.config.TaskSource#loadTask` を使ってきました. しかし、これらはもう非推奨になりました。外部ライブラリになった [`org.embulk:embulk-util-config`](https://dev.embulk.org/embulk-util-config/) を代わりに使ってください。

```
compile "org.embulk:embulk-util-config:0.3.0"
```

使い方はそこまで複雑ではありません。まず `@Config`, `@ConfigDefault`, `Task` を `embulk-util-config` のものに置き換えてください。

```
- import org.embulk.config.Config;
- import org.embulk.config.ConfigDefault;
- import org.embulk.config.Task;
+ import org.embulk.util.config.Config;
+ import org.embulk.util.config.ConfigDefault;
+ import org.embulk.util.config.Task;
```

そしてプラグインの `Task` 定義を確認してください。これが `Task` (`org.embulk.config.Task` → `org.embulk.util.config.Task`) のみを継承しているなら、特に問題はありません。もしこれが他の古い interface (たとえば `TimestampParser.Task`) を継承していたら、代わりの `org.embulk.util.config.*` を使った新しいものを探すか、ユーザーから見て互換になるように中のメソッド定義をコピーしてきてください。

```
public interface PluginTask extends Task, org.embulk.spi.time.TimestampParser.Task {  // => TimestampParser.Task を消す
    // ...

    // org.embulk.spi.time.TimestampParser.Task からコピー
    @Config("default_timezone")
    @ConfigDefault("\"UTC\"")
    String getDefaultTimeZoneId();

    // org.embulk.spi.time.TimestampParser.Task からコピー
    @Config("default_timestamp_format")
    @ConfigDefault("\"%Y-%m-%d %H:%M:%S.%N %z\"")
    String getDefaultTimestampFormat();

    // org.embulk.spi.time.TimestampParser.Task からコピー
    @Config("default_date")
    @ConfigDefault("\"1970-01-01\"")
    String getDefaultDate();
}
```

次に `org.embulk.util.config.ConfigMapperFactory` のインスタンスを用意してください。

```
import org.embulk.util.config.ConfigMapperFactory;

private static final ConfigMapperFactory CONFIG_MAPPER_FACTORY = ConfigMapperFactory.builder().addDefaultModules().build();
```

Embulk config から、あまり標準的ではないクラスへのマッピングが必要なときは、任意の Jackson [Module](https://fasterxml.github.io/jackson-databind/javadoc/2.6/com/fasterxml/jackson/databind/Module.html) を追加できます。

```
private static final ConfigMapperFactory CONFIG_MAPPER_FACTORY =
        ConfigMapperFactory.builder().addDefaultModules().addModule(new SomeJacksonModule()).build();
```

Apache BVal を使って config の検証を有効にすることもできます。

```
import javax.validation.Validation;
import javax.validation.Validator;
import org.apache.bval.jsr303.ApacheValidationProvider;

private static final Validator VALIDATOR =
        Validation.byProvider(ApacheValidationProvider.class).configure().buildValidatorFactory().getValidator();

private static final ConfigMapperFactory CONFIG_MAPPER_FACTORY =
        ConfigMapperFactory.builder().addDefaultModules().withValidator(VALIDATOR).build();
```

最後に `loadConfig` と `loadTask` を `org.embulk.util.config.ConfigMapper#map` と `org.embulk.util.config.TaskMapper#map` にそれぞれ置き換えてください。

```
+ import org.embulk.util.config.ConfigMapper;
+ import org.embulk.util.config.TaskMapper;

- PluginTask task = config.loadConfig(PluginTask.class);
+ final ConfigMapper configMapper = CONFIG_MAPPER_FACTORY.createConfigMapper();
+ final PluginTask task = configMapper.map(config, PluginTask.class);

- PluginTask task = taskSource.loadTask(PluginTask.class);
+ final TaskMapper taskMapper = CONFIG_MAPPER_FACTORY.createTaskMapper();
+ final PluginTask task = taskMapper.map(taskSource, PluginTask.class);
```

`embulk-util-config` 自体が Jackson に依存しているので、これを使うとプラグインは Jackson への依存を直接持つようになります。

プラグインが Embulk v0.9 でもしばらくは動くようにするには、新しい Embulk v0.11.0 がリリースされても、しばらくは Jackson 2.6.7 のままにしておくことを検討してください。他の多くのプラグインが v0.11 対応になり、多くのユーザーが v0.11.0 以降を使うようになったら、もうプラグインがどの Jackson バージョンを使っても動くはずです!

##### 日付・時刻とタイムゾーンの扱い

Embulk から [Joda-Time](https://www.joda.org/joda-time/) は削除されます。 `org.embulk.spi.time.TimestampFormatter` と `org.embulk.spi.time.TimestampParser` は非推奨です。 Java 8 標準の `java.time` クラス群と、外部ライブラリの [`org.embulk:embulk-util-timestamp`](https://dev.embulk.org/embulk-util-timestamp/) を代わりに使ってください.

```
compile "org.embulk:embulk-util-timestamp:0.2.1"
```

Embulk の [`org.embulk.spi.time.Timestamp`](https://dev.embulk.org/embulk-api/0.10.31/javadoc/org/embulk/spi/time/Timestamp.html) も非推奨になっています。 [`java.time.Instant`](https://docs.oracle.com/javase/jp/8/docs/api/java/time/Instant.html) を代わりに使ってください。

Joda-Time の [`DateTime`](https://www.joda.org/joda-time/apidocs/org/joda/time/DateTime.html) は、代わりに [`java.time.OffsetDateTime`](https://docs.oracle.com/javase/8/docs/api/java/time/OffsetDateTime.html) または [`java.time.ZonedDateTime`](https://docs.oracle.com/javase/8/docs/api/java/time/ZonedDateTime.html) を使ってください。 `OffsetDateTime` で十分な場合は、基本的に `OffsetDateTime` の方を使っておくことをおすすめします。地理的地域ベースのタイムゾーンを使うと、その複雑さからすべてが面倒くさくなります。 (タイムゾーンのデフォルトを `America/Los_Angeles` にしているときに `2017-03-12 02:30:00` を受け取ったらどうなるか想像してみましょう。)

Joda-Time の [`DateTimeZone`](https://www.joda.org/joda-time/apidocs/org/joda/time/DateTimeZone.html) は、代わりに [`java.time.ZoneOffset`](https://docs.oracle.com/javase/8/docs/api/java/time/ZoneOffset.html) または [`java.time.ZoneId`](https://docs.oracle.com/javase/8/docs/api/java/time/ZoneId.html) を使ってください。上と同じ理由から、それで足りる場合は `ZoneOffset` に制限しておくことをおすすめします。

`org.embulk.spi.time.TimestampFormatter` と `org.embulk.spi.time.TimestampParser` は前述のとおり非推奨です。 `embulk-util-timestamp` の [`org.embulk.util.timestamp.TimestampFormatter`](https://dev.embulk.org/embulk-util-timestamp/0.2.1/javadoc/org/embulk/util/timestamp/TimestampFormatter.html) を代わりに使ってください。「そのまま」移行する典型的なやり方は以下のようになります。

```
// embulk-util-config の org.embulk.util.config.Task を使って PluginTask を定義。
private interface PluginTask extends org.embulk.util.config.Task {
    // ...

    // org.embulk.spi.time.TimestampParser.Task からコピー
    @Config("default_timezone")
    @ConfigDefault("\"UTC\"")
    String getDefaultTimeZoneId();

    // org.embulk.spi.time.TimestampParser.Task からコピー
    @Config("default_timestamp_format")
    @ConfigDefault("\"%Y-%m-%d %H:%M:%S.%N %z\"")
    String getDefaultTimestampFormat();

    // org.embulk.spi.time.TimestampParser.Task からコピー
    @Config("default_date")
    @ConfigDefault("\"1970-01-01\"")
    String getDefaultDate();
}

// org.embulk.spi.time.Timestamp{Formatter|Parser}.TimestampColumnOption と互換の TimestampColumnOption を自分で定義。
private interface TimestampColumnOption extends org.embulk.util.config.Task {
    @Config("timezone")
    @ConfigDefault("null")
    java.util.Optional<String> getTimeZoneId();

    @Config("format")
    @ConfigDefault("null")
    java.util.Optional<String> getFormat();

    @Config("date")
    @ConfigDefault("null")
    java.util.Optional<String> getDate();
}

// PluginTask に変換した config 全体を取得。
final PluginTask task = configMapper.map(configSource, PluginTask.class);

// カラムごとのオプションを embulk-util-config で取得。
final TimestampColumnOption columnOption = configMapper.map(columnConfig.getOption(), TimestampColumnOption.class);

// TimestampFormatter を構築。
final TimestampFormatter formatter = TimestampFormatter
        .builder(columnOption.getFormat().orElse(task.getDefaultTimestampFormat(), true)
        .setDefaultZoneFromString(columnOption.getTimeZoneId().orElse(task.getDefaultTimeZoneId()))
        .setDefaultDateFromString(columnOption.getDate().orElse(task.getDefaultDate()))
        .build();

Instant instant = formatter.parse("2019-02-28 12:34:56 +09:00");
System.out.println(instant);  // => "2019-02-28T03:34:56Z"

String formatted = formatter.format(Instant.ofEpochSecond(1009110896));
System.out.println(formatted);  // => "2017-12-23 12:34:56 UTC"
```

##### Jackson と JSON の扱い

`org.embulk:embulk-core` はもう Jackson への依存を持っていません。プラグイン自体の中で Jackson を使う必要がある場合は、明示的に Jackson を依存に持つ必要があります。

もう前述の `org.embulk:embulk-util-config` を使い始めているなら、そのプラグインはもう Jackson (2.6.7) への依存を持っているはずです。そこに Jackson のサブライブラリを自分で足すこともできます。

Embulk の `org.embulk.spi.json.JsonParser` は非推奨です。 [`org.embulk:embulk-util-json`](https://dev.embulk.org/embulk-util-json/) を代わりに使ってください。使い方はほぼ同じです。

前述したように、プラグインが古い Embulk v0.9 でも今しばらく動くようにするには Jackson 2.6.7 に留めておいてください。

##### Google Guava と Apache Commons Lang 3

`org.embulk:embulk-core` はもう Google Guava と Apache Commons Lang 3 への依存を持っていません。プラグインがそれらを使っている場合は Java 8 の標準クラスで置き換えるか、それらへの依存を明示的に持たせてください。

Java 8 は十分に強力です。 Google Guava や Apache Commons Lang 3 に強く依存した使い方をしているのでなければ Java 8 の標準クラスで置き換えるのをおすすめします。それらのライブラリ (特に Guava) にはときどき非互換が入ってきて、イライラさせられることがあるでしょう。

少なくとも Guava の [`com.google.common.base.Optional`](https://guava.dev/releases/18.0/api/docs/com/google/common/base/Optional.html) は `embulk-util-config` では動きません。これは [`java.util.Optional`](https://docs.oracle.com/javase/8/docs/api/java/util/Optional.html) に置き換える必要があります。

Guava の典型的なちょっとした使い方は、以下のように置き換えられます。

* Guava のイミュータブル・コレクション (immutable collections) は [`java.util.Collections`](https://docs.oracle.com/javase/8/docs/api/java/util/Collections.html) の `unmodifiable*` メソッド群で置き換えられます。
* Guava の [`Throwables`](https://guava.dev/releases/18.0/api/docs/com/google/common/base/Throwables.html) は非推奨になっています。 [Guava のドキュメント](https://github.com/google/guava/wiki/Why-we-deprecated-Throwables.propagate) を呼んでください。
* Guava の [`Preconditions`](https://guava.dev/releases/18.0/api/docs/com/google/common/base/Preconditions.html) は単純に置き換えられます。

##### Guice

プラグインの中で Guice を使うのはできるだけ止めてください。

もし、他の依存ライブラリが Guice を使っているなどの理由で避けられない場合は、明示的に Guice を依存に持ってください。ただし、古い Embulk v0.9 では即座に動かなくなる可能性があります。

##### JRuby (`org.jruby` クラス)

プラグインの中で JRuby やそのクラスを使うのは止めてください。

Java プラグインから JRuby を呼ぶ必要があった典型例は、日付・時刻の解釈や出力でした。これは [`embulk-util-timestamp`](https://dev.embulk.org/embulk-util-timestamp/) や [`embulk-util-rubytime`](https://dev.embulk.org/embulk-util-rubytime/) で置き換えられます。

##### FindBugs

`org.embulk:embulk-core` は、もう FindBugs の `com.google.code.findbugs:annotations` (`@SuppressFBWarnings`) への依存を持っていません。

[FindBugs](http://findbugs.sourceforge.net/) はもう長い間メンテナンスされていないことが知られています。少なくとも FindBugs 使うのは止めることをおすすめします。

[SpotBugs](https://spotbugs.github.io/) は FindBugs のいい置き換えになるかもしれません。ですが Embulk 本体から SpotBugs 向けのサポートを特別に用意する予定はありません。プラグインの中でご自分で試してみてください。

##### Java 11 以降への準備

Java エコシステムでは、新しい Java のバージョンに向けて準備をしていかなければなりません。 Java 8 から 11 以降に移る際には大きなギャップがあり、特に Java EE 関連のクラス群 (典型例は JAXB `javax.xml.*`) が Java 11 の実行環境から消えるのは影響が大きいです。もしプラグインがこれらのクラスを使っている場合は、そのままでは Java 11 以降では動きません。詳細は [JEP 320](https://openjdk.java.net/jeps/320) を確認してください。

Embulk v0.11 以降では、プラグインからこれらのクラスが使われたことを検出すると (Java 8 で動かしていても) 以下のような警告メッセージをログに出します。

```
Class javax.xml.bind.JAXB is loaded by the parent ClassLoader, which is removed by JEP 320. The plugin needs to include it on the plugin side. See https://github.com/embulk/embulk/issues/1270 for more details.
```

フレームワークとしての Embulk からは、これ以上のサポートは特に提供しません。もしプラグインが Java EE のクラス群を使っている場合は、代わりになる依存関係を明示的に自分で明示的に追加してください。

典型例をいくつか以下に掲載します。

```
// JAXB への依存を明示的に追加。

// JAXB 2.2.11 を選んだのは以下の理由です。
// 1. JDK 8 に同梱の JAXB は 2.2.8 です。
//    Java 8 がベースにある場合や Embulk v0.9 でも動かしたい場合を考え、しばらくは近いバージョンにしておくのがいいでしょう。
//    https://javaee.github.io/jaxb-v2/doc/user-guide/ch02.html#a-2-2-8
// 2. しかし com.sun.xml.bind:jaxb-core:2.2.8 も com.sun.xml.bind:jaxb-impl:2.2.8 も Maven Central にはありません。
// 3. JAXB 2.2 系の中では 2.2.11 が最もよく使われているようです。
//    https://mvnrepository.com/artifact/javax.xml.bind/jaxb-api
//    https://mvnrepository.com/artifact/com.sun.xml.bind/jaxb-core
//    https://mvnrepository.com/artifact/com.sun.xml.bind/jaxb-impl
// 4. JAXB 2.2.8 と 2.2.11 は、同じ名前のクラス群で構成されているようです。
//    内部の実装が若干違っていたとしても、クラスローダーを混乱させる懸念は薄いでしょう。
compile "javax.xml.bind:jaxb-api:2.2.11"
compile "com.sun.xml.bind:jaxb-core:2.2.11"
compile "com.sun.xml.bind:jaxb-impl:2.2.11"
```

```
// Activation Framework への依存を明示的に追加。

// Activation Framework は、よくJAXB と一緒に必要になります。
// 1.1.1 を選んだ理由は、これが JAXB 2.2.11 がリリースされた時点で最新だったものだからです。
compile "javax.activation:activation:1.1.1"
```

```
// Annotation API への依存を明示的に追加。

// 1.2 を選んだ理由は Java 8 の時代には 1.2 しかなかったようだからです。
// https://mvnrepository.com/artifact/javax.annotation/javax.annotation-api
compile "javax.annotation:javax.annotation-api:1.2"
```

##### ログ

`org.embulk.spi.Exec.getLogger` は非推奨です。 [SLF4J](http://www.slf4j.org/) の [`org.slf4j.LoggerFactory.getLogger`](https://www.javadoc.io/doc/org.slf4j/slf4j-api/1.7.30/org/slf4j/LoggerFactory.html) を直接使ってください。

##### `org.embulk:embulk-core` にあるその他のユーティリティクラス

* File-like なバイト列の処理 (`org.embulk.spi.util` の `FileInputInputStream` や `FileOutputOutputStream`)
    * [`embulk-util-file`](https://dev.embulk.org/embulk-util-file/) として外部ライブラリ化
* テキストの処理 (`org.embulk.spi.util` の `LineDecoder` や `LineEncoder`)
    * [`embulk-util-text`](https://dev.embulk.org/embulk-util-text/) として外部ライブラリ化
* `org.embulk.spi.util` の `DynamicColumnSetter` 関連
    * [`embulk-util-dynamic`](https://dev.embulk.org/embulk-util-dynamic/) として外部ライブラリ化
* リトライ (`org.embulk.spi.util` の `RetryExecutor`)
    * [`embulk-util-retryhelper`](https://dev.embulk.org/embulk-util-retryhelper/) として外部ライブラリ化

#### 置き換えのまとめ

##### Embulk config の処理

| 旧 | 新 |
|---|---|
| `@Config` | [embulk-util-config](https://dev.embulk.org/embulk-util-config/) の `@org.embulk.util.config.Config` |
| `@ConfigDefault` | [embulk-util-config](https://dev.embulk.org/embulk-util-config/) の `@org.embulk.util.config.ConfigDefault` |
| `@ConfigInject` | `Exec.get???()` (たとえば `Exec.getBufferAllocator`) |
| `ConfigSource#loadConfig` | [embulk-util-config](https://dev.embulk.org/embulk-util-config/) の `ConfigMapper#map` |
| `TaskSource#loadTask` | [embulk-util-config](https://dev.embulk.org/embulk-util-config/) の `TaskMapper#map` |
| `Task` | [embulk-util-config](https://dev.embulk.org/embulk-util-config/) の `org.embulk.util.config.Task` |
| `ColumnConfig` | [embulk-util-config](https://dev.embulk.org/embulk-util-config/) の `org.embulk.util.config.units.ColumnConfig` |
| `SchemaConfig` | [embulk-util-config](https://dev.embulk.org/embulk-util-config/) の `org.embulk.util.config.units.SchemaConfig` |
| `ModelManager` | 使うのを止めてください。代わりが必要な場合は Jackson の `ObjectMapper` を自分で構築してください。 |
| `LocalFile` in `PluginTask` | [embulk-util-config](https://dev.embulk.org/embulk-util-config/) の `org.embulk.util.config.modules.LocalFileModule` と `org.embulk.util.config.units.LocalFile` |
| `Charset` in `PluginTask` | [embulk-util-config](https://dev.embulk.org/embulk-util-config/) の `org.embulk.util.config.modules.CharsetModule` |
| `Column` in `PluginTask` | [embulk-util-config](https://dev.embulk.org/embulk-util-config/) の `org.embulk.util.config.modules.ColumnModule` |
| `Schema` in `PluginTask` | [embulk-util-config](https://dev.embulk.org/embulk-util-config/) の `org.embulk.util.config.modules.SchemaModule` |
| `Type` in `PluginTask` | [embulk-util-config](https://dev.embulk.org/embulk-util-config/) の `org.embulk.util.config.modules.TypeModule` |

##### 日付・時刻とタイムゾーンの扱い

| 旧 | 新 |
|---|---|
| `org.joda.time.DateTime` | `java.time.OffsetDateTime` または `java.time.ZonedDateTime` |
| `org.joda.time.DateTimeZone` | `java.time.ZoneOffset` または `java.time.ZoneId` |
| `org.joda.time.DateTimeZone` in `PluginTask` | 必要なら `java.time.ZoneId` と [embulk-util-config](https://dev.embulk.org/embulk-util-config/) の `org.embulk.util.config.modules.ZoneIdModule` |
| `Timestamp` | できるだけ `java.time.Instant` に |
| `TimestampFormatter` | [embulk-util-timestamp](https://dev.embulk.org/embulk-util-timestamp/) の `org.embulk.util.timestamp.TimestampFormatter` |
| `TimestampParser` | [embulk-util-timestamp](https://dev.embulk.org/embulk-util-timestamp/) の `org.embulk.util.timestamp.TimestampFormatter` |

##### Jackson と JSON の扱い

| 旧 | 新 |
|---|---|
| `org.embulk.spi.json.*` | [embulk-util-json](https://dev.embulk.org/embulk-util-json/) の `org.embulk.util.json.*` |

##### Google Guava と Apache Commons Lang 3

| 旧 | 新 |
|---|---|
| Guava `Optional` | `java.util.Optional` |
| Guava `Throwables` | [Guava のドキュメント](https://github.com/google/guava/wiki/Why-we-deprecated-Throwables.propagate) を読んでください |

##### JRuby (`org.jruby` クラス)

| 旧 | 新 |
|---|---|
| `org.jruby.*` | 使うのを止めてください。日付・時刻の操作には [embulk-util-tiemstamp](https://dev.embulk.org/embulk-util-timestamp/) や [embulk-util-rubytime](https://dev.embulk.org/embulk-util-rubytime/) を使ってください。 |

##### FindBugs

| 旧 | 新 |
|---|---|
| `@SuppressFBWarnings` | 削除するか、使い方を模索してみてください。 |

##### Java 11 以降への準備

| 旧 | 新 |
|---|---|
| `javax.*` | [JEP 320](https://openjdk.java.net/jeps/320) を見て、それが Java 11 以降で消えたものかどうか確認してください。 ([Issue](https://github.com/embulk/embulk/issues/1270)) |

##### ログ

| 旧 | 新 |
|---|---|
| `Exec.getLogger` | `org.slf4j.LoggerFactory.getLogger` |

##### `org.embulk:embulk-core` にあるその他のユーティリティクラス

| 旧 | 新 |
|---|---|
| `org.embulk.spi.util.*` | [embulk-util-file](https://dev.embulk.org/embulk-util-file/), [embulk-util-text](https://dev.embulk.org/embulk-util-text/), [embulk-util-dynamic](https://dev.embulk.org/embulk-util-dynamic/), [embulk-util-retryhelper](https://dev.embulk.org/embulk-util-retryhelper/) |

#### テスト

テストでは、まだ `org.embulk:embulk-core` と、さらに `org.embulk:embulk-deps` を、依存に追加する必要があります。

```
testCompile "org.embulk:embulk-core:0.10.??"
testCompile "org.embulk:embulk-deps:0.10.??"
```

テストでは `embulk-core` にしかない内部クラスを使いたくなることがあるでしょう。典型的なものは `org.embulk.spi.ExecInternal` から取得できます。例えば `ExecInternal.getModelManager()` で `org.embulk.config.ModelManager` のインスタンスが手に入ります。

ただしそれらの内部クラスは、予告なく互換性を失うことがあると気に留めておいてください。例えば `ModelManager` の以下のメソッドは v0.11 で削除されました。

* `public <T> T readObject(Class<T>, com.fasterxml.jackson.core.JsonParser)`
* `public <T> T readObjectWithConfigSerDe(Class<T>, com.fasterxml.jackson.core.JsonParser)`

今後 v0.11 系の開発の中で、テスト用のフレームワークを拡充する予定です。

### 最後に

これまで Embulk プラグインを開発してくれた方々には、ご面倒をおかけします。

この一連の変更が終了すると、本体の更新がプラグインに影響を及ぼすことが (ゼロとは言えませんが) かなり減るはずです。ぜひご協力をよろしくお願いいたします。
