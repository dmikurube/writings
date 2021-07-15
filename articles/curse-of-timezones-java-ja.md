---
title: "タイムゾーン呪いの書 (Java 編)"
emoji: "🌎" # アイキャッチとして使われる絵文字（1文字だけ）
type: "tech" # tech: 技術記事 / idea: アイデア
topics: [ "datetime", "timezone", "tzdb", "java", "jsr310" ]
layout: default
published: true
---

「タイムゾーン呪いの書」は、もともと 2018年に [Qiita に投稿した記事](https://qiita.com/dmikurube/items/15899ec9de643e91497c)でしたが、大幅な改訂を 2021年におこない、同時にこちらの [Zenn](https://zenn.dev/) に引っ越してきました。この改訂で記事全体が長大になったので、[「知識編」](./curse-of-timezones-common-ja)・[「実装編」](./curse-of-timezones-impl-ja)・「Java 編」と記事を分けることにしました。

この「Java 編」は、「知識編」と「実装編」に続く最終章です。本記事全体を通して、「知識編」「実装編」を読んでいることを前提にしているので、ご注意ください。

はじめに
=========

「知識編」と「実装編」では、言語やソフトウェア特有の話にはあまり踏み込まずに、時刻とタイムゾーンについてなるべく一般論を書いてきました。そんな中でも Java の [JSR 310: Date and Time API](https://jcp.org/en/jsr/detail?id=310) は何度も参照しています。 JSR 310 は時刻やタイムゾーンという概念をけっこう上手に一般化していて、一般論の検討にもいいモデルだったからです。

とはいえ、せっかくそれだけ触れてきたのですから、ついでに Java と JSR 310 特有のトピックにもしっかり踏み込もう、というのがこの「Java 編」です。「実装編」の一般論をベースにして、具体的な JSR 310 のクラスの使いかた，使い分けや、運用のしかたについて検討します。

JSR 310: Date and Time API
===========================

[JSR 310: Date and Time API](https://jcp.org/en/jsr/detail?id=310) は、日付・時刻・タイムゾーンをあつかう新しい Java API です。 Java 8 から追加されました。「知識編」や「実装編」で何度も参照していますね。

[Joda-Time](http://www.joda.org/joda-time/) というオープンソースの日付・時刻ライブラリが 2005年ごろに公開されていたのですが、その作者である Stephen Colebourne 氏が共同リーダーとして参加し、正式な Java API として Joda-Time をベースに再設計されたのが JSR 310 です。

古くからの Java API である [`java.util.Date`](https://docs.oracle.com/javase/jp/8/docs/api/java/util/Date.html) や [`java.text.SimpleDateFormatter`](https://docs.oracle.com/javase/jp/8/docs/api/java/text/SimpleDateFormat.html) には、スレッド安全性や API 設計など多くの問題があり、それらを解消すべく JSR 310 を設計した、ということです。 [^new-date-time-api]

[^new-date-time-api]: ["Java SE 8 Date and Time" by Ben Evans and Richard Warburton (January/February 2014, Oracle Technical Article)](https://www.oracle.com/technical-resources/articles/java/jf14-date-time.html)

「知識編」や「実装編」で触れてきたように、時刻やタイムゾーンとはそもそも複雑な概念です。特に地域ベースのタイムゾーンが絡んでくると、さまざまなパターンで例外的な状況が起こります。これがタイムゾーンの呪いです。

JSR 310 はその複雑な概念をかなり忠実にモデル化し、例外的な状況も明示的にあつかえるような設計になっています。それは複雑な概念や例外的な状況を「明示的にあつかわなければならない」ということでもあり、それが「複雑すぎる」「`OffsetDateTime` と `ZonedDateTime` の違いがよくわからない」のような声につながってもいます。 [^apache-spark] ですが、この複雑さを中途半端に隠蔽してしまうと、例外的な状況のあつかいがうやむやになりがちです。そして、その例外的な状況はたまにしか起こらないので、起きて初めてうやむやなあつかいが露見するということにもなりがちです。

[^apache-spark]: Apache Spark プロジェクトで JSR 310 に沿ったモデル化をした経緯が ["A Comprehensive Look at Dates and Timestamps in Apache Spark™ 3.0" (July 22, 2020, Databricks)](https://databricks.com/blog/2020/07/22/a-comprehensive-look-at-dates-and-timestamps-in-apache-spark-3-0.html) にまとめられていて、参考になります。

この「Java 編」では、おもに JSR 310 の各クラス、中でも基本となる日付/時間データクラスの使いかた、使い分けを検討します。

Instant: Unix time
-------------------

Unix time に相当する、世界共通の時間軸上の一点を表すのが [`java.time.Instant`](https://docs.oracle.com/javase/jp/8/docs/api/java/time/Instant.html) です。これは Unix time と「ほぼ」同じものです。 Unix time とのおもな違いはうるう秒のあつかいです。うるう秒を「希釈」する手法を「知識編」でいくつか取り上げましたが、その一つの "[UTC-SLS](https://www.cl.cam.ac.uk/~mgk25/time/utc-sls/)" をベースにした「Java タイム・スケール」が使われます。 [^java-time-scale]

[^java-time-scale]: Java タイム・スケールの詳しい説明は [`java.time.Instant` の Javadoc](https://docs.oracle.com/javase/jp/8/docs/api/java/time/Instant.html) にあります。

Java タイム・スケールでは 1972年 11月 3日以降の時刻に UTC-SLS が適用され、うるう秒は、うるう秒が適用される日の最後の 1000秒で希釈されます。 Java タイム・スケールは `Instant` だけではなく、すべての日付/時間クラスで使われます。

Unix time (`Instant`) は、「実装編」で検討したように、万能でこそないものの時刻の内部データ表現として有力な候補の一つです。専用のクラスなので、プリミティブな数値型 (`long` や `double` など) で Unix time をあつかうことによる混乱もなく、変な取り違えをするリスクも小さいでしょう。

ZoneId と ZoneOffset
---------------------

JSR 310 のタイムゾーン情報は、すべてのタイムゾーンをあらわす抽象クラス [`java.time.ZoneId`](https://docs.oracle.com/javase/jp/8/docs/api/java/time/ZoneId.html) と、その派生クラスで固定オフセットのみをあらわす具象クラス [`java.time.ZoneOffset`](https://docs.oracle.com/javase/jp/8/docs/api/java/time/ZoneOffset.html) の、二種類のクラスで表現されます。これらのインスタンスは `ZoneOffset.UTC` のような定数を参照したり `ZoneOffset.of("+09:00")`, `ZoneOffset.ofHours(9)`, `ZoneId.of("Asia/Tokyo")` のようにインスタンス化したりして使います。これらのインスタンスは、いずれも不変 (immutable) です。

地域ベースの `ZoneId` インスタンスは `ZoneId.of("Asia/Tokyo")` のように tzdb のタイムゾーン ID と対応します。夏時間などの切り替わりルールも tzdb をもとに実装されます。 `ZoneId` インスタンスの `ZoneId#getRules()` で、切り替わりルールを実装した [`java.time.ZoneRules`](https://docs.oracle.com/javase/jp/8/docs/api/java/time/zone/ZoneRules.html) インスタンスを取得できます。

`ZoneId` は抽象クラスなので、そのものをインスタンス化することはできません。 `ZoneOffset` ではない、地域ベースのタイムゾーンをあらわすインスタンスは、派生クラスである package-private の [`java.time.ZoneRegion`](http://hg.openjdk.java.net/jdk8/jdk8/jdk/file/jdk8-b132/src/share/classes/java/time/ZoneRegion.java#l90) として実装されているようです。 [^zoneregion]

[^zoneregion]: `ZoneRegion` は package-private なので直接は参照できませんし、この実装が今後も保証されるわけではありません。このようなインスタンス使うときはあくまで `ZoneId` 型として参照します。

固定オフセットを持つ一部の `ZoneRegion` インスタンスは `ZoneId#normalize()` で `ZoneOffset` に正規化できます。 `ZoneOffset` インスタンスの `normalize()` メソッドを呼んでも、なにも変わりません。

`"Asia/Tokyo"`, `"UTC+09:00"`, `"+09:00"` のそれぞれで `ZoneId.of()` と `ZoneOffset.of()` を呼び出し、インスタンス化してみた例が以下のとおりです。

```java
import java.time.DateTimeException;
import java.time.ZoneId;
import java.time.ZoneOffset;
import java.time.zone.ZoneOffsetTransition;
import java.util.List;

public final class ZoneIds {
    public static void main(final String[] args) {
        printZoneId("Asia/Tokyo");
        printZoneOffset("Asia/Tokyo");

        printZoneId("UTC+09:00");
        printZoneOffset("UTC+09:00");

        printZoneId("+09:00");
        printZoneOffset("+09:00");
    }

    private static void printZoneId(final String id) {
        System.out.printf("ZoneId.of(\"%s\")\n", id);
        try {
            print(ZoneId.of(id));
        } catch (DateTimeException ex) {
            System.out.printf("  DateTimeException: %s\n\n", ex.getMessage());
        }
    }

    private static void printZoneOffset(final String id) {
        System.out.printf("ZoneOffset.of(\"%s\")\n", id);
        try {
            print(ZoneOffset.of(id));
        } catch (DateTimeException ex) {
            System.out.printf("  DateTimeException: %s\n\n", ex.getMessage());
        }
    }

    private static void print(final ZoneId zoneId) {
        System.out.printf("  .toString(): <%s>\n", zoneId);
        System.out.printf("  .getClass(): <%s>\n", zoneId.getClass());
        System.out.printf("  .normalized().toString(): <%s>\n", zoneId.normalized());
        System.out.printf("  .normalized().getClass(): <%s>\n", zoneId.normalized().getClass());
        System.out.printf("  .getRules(): <%s>\n", zoneId.getRules());
        System.out.printf("  .getRules().isFixedOffset(): <%s>\n", zoneId.getRules().isFixedOffset());
        System.out.printf("  .getRules().getTransitions():\n");
        final List<ZoneOffsetTransition> transitions = zoneId.getRules().getTransitions();
        if (transitions.isEmpty()) {
            System.out.printf("      (empty)\n");
        } else {
            for (final ZoneOffsetTransition transition : transitions) {
                System.out.printf("      %s\n", transition.toString());
            }
        }
        System.out.printf("\n");
    }
}
```

この実行結果はこうなります。

```
$ java -version
openjdk version "1.8.0_292"
OpenJDK Runtime Environment (build 1.8.0_292-8u292-b10-0ubuntu1~20.04-b10)
OpenJDK 64-Bit Server VM (build 25.292-b10, mixed mode)

$ java ZoneIds
ZoneId.of("Asia/Tokyo")
  .toString(): <Asia/Tokyo>
  .getClass(): <class java.time.ZoneRegion>
  .normalized().toString(): <Asia/Tokyo>
  .normalized().getClass(): <class java.time.ZoneRegion>
  .getRules(): <ZoneRules[currentStandardOffset=+09:00]>
  .getRules().isFixedOffset(): <false>
  .getRules().getTransitions():
      Transition[Overlap at 1888-01-01T00:18:59+09:18:59 to +09:00]
      Transition[Gap at 1948-05-02T00:00+09:00 to +10:00]
      Transition[Overlap at 1948-09-12T01:00+10:00 to +09:00]
      Transition[Gap at 1949-04-03T00:00+09:00 to +10:00]
      Transition[Overlap at 1949-09-11T01:00+10:00 to +09:00]
      Transition[Gap at 1950-05-07T00:00+09:00 to +10:00]
      Transition[Overlap at 1950-09-10T01:00+10:00 to +09:00]
      Transition[Gap at 1951-05-06T00:00+09:00 to +10:00]
      Transition[Overlap at 1951-09-09T01:00+10:00 to +09:00]

ZoneOffset.of("Asia/Tokyo")
  DateTimeException: Invalid ID for ZoneOffset, invalid format: Asia/Tokyo

ZoneId.of("UTC+09:00")
  .toString(): <UTC+09:00>
  .getClass(): <class java.time.ZoneRegion>
  .normalized().toString(): <+09:00>
  .normalized().getClass(): <class java.time.ZoneOffset>
  .getRules(): <ZoneRules[currentStandardOffset=+09:00]>
  .getRules().isFixedOffset(): <true>
  .getRules().getTransitions():
      (empty)

ZoneOffset.of("UTC+09:00")
  DateTimeException: Invalid ID for ZoneOffset, non numeric characters found: UTC+09:00

ZoneId.of("+09:00")
  .toString(): <+09:00>
  .getClass(): <class java.time.ZoneOffset>
  .normalized().toString(): <+09:00>
  .normalized().getClass(): <class java.time.ZoneOffset>
  .getRules(): <ZoneRules[currentStandardOffset=+09:00]>
  .getRules().isFixedOffset(): <true>
  .getRules().getTransitions():
      (empty)

ZoneOffset.of("+09:00")
  .toString(): <+09:00>
  .getClass(): <class java.time.ZoneOffset>
  .normalized().toString(): <+09:00>
  .normalized().getClass(): <class java.time.ZoneOffset>
  .getRules(): <ZoneRules[currentStandardOffset=+09:00]>
  .getRules().isFixedOffset(): <true>
  .getRules().getTransitions():
      (empty)
```

東京時間 (`ZoneId.of("Asia/Tokyo")`) が、タイムゾーンの切り替わりを何回か経験していることがここからわかりますね。これは国際子午線会議 (1884年) をもとにした日本標準時の導入 (1888年) と、第二次世界大戦の直後に数年間だけ導入された夏時間のデータです。このデータは tzdb から来ています。

地域ベースのタイムゾーンを使うかぎり、オフセットの切り替わりにともなう「存在しない時刻」や「二重に存在する時刻」の問題、つまり「タイムゾーンの呪い」から逃げられない、というのが「知識編」と「実装編」でくりかえし検討したことでした。オフセットのみで完結できる要件だったら、なるべく地域ベースのタイムゾーンには触れずに済ませたいところです。

ここで JSR 310 のキモの一つが、オフセットのみをあらわす `ZoneOffset` が、すべてのタイムゾーンをあらわす `ZoneId` とは (派生クラスですが) 別のクラスとして実装されていることです。タイムゾーン情報として `ZoneOffset` を使っているかぎり、タイムゾーンの呪いから大きく距離を置けていることを、コードと型のレベルで確認できるのです。逆に、コードに `ZoneId` がまぎれ込んできたら、タイムゾーンの呪いに気をつけよう、という黄色信号だととらえることができます。

次の `OffsetDateTime` と `ZonedDateTime` にも、同様のことがいえます。

OffsetDateTime と ZonedDateTime
--------------------------------

JSR 310 で「日付と時刻」を、すなわち「年・月・日・時・分・秒」をあらわすクラスは三種類あります。 [`java.time.LocalDateTime`]((https://docs.oracle.com/javase/jp/8/docs/api/java/time/LocalDateTime.html)), [`java.time.OffsetDateTime`]((https://docs.oracle.com/javase/jp/8/docs/api/java/time/OffsetDateTime.html)), [`java.time.ZonedDateTime`]((https://docs.oracle.com/javase/jp/8/docs/api/java/time/ZonedDateTime.html)) です。これらも `Instant` と同様の「Java タイム・スケール」にのっとっていて、うるう秒をあつかうことはできません。

このうち `LocalDateTime` は「タイムゾーン情報を持たない」日付と時間で、ある意味わかりやすいです。残りの二つについて「`OffsetDateTime` と `ZonedDateTime` をどう使い分ければいいのかよくわからない」というのが JSR 310 の FAQ ではないでしょうか。

固定オフセットの `ZoneOffset` のみタイムゾーン情報として受け付けるのが `OffsetDateTime` で、地域ベースのタイムゾーンを含む任意の `ZoneId` を受け付けるのが `ZonedDateTime` です。…というのが通り一遍の説明ですが、この説明だけで使い分けられるなら、これが FAQ にはなっていませんね。これらの使い分けについて、以下で検討していきます。

### LocalDateTime

「実装編」で検討した『タイムゾーンが不明なままの「年・月・日・時・分・秒」を、そのまま保持し続けない、そのまま持ち回らない』という大原則は、ここでも有効です。 `LocalDateTime` はタイムゾーン情報を一切持たない日付・時刻です。         `LocalDateTime` だけから `Instant` には変換できませんし、ある `LocalDateTime` インスタンスが世界共通の時間軸上のどこに対応するかは特定できません。

MySQL などの RDB やユーザー入力などの外部データに、タイムゾーンが不明な「年・月・日・時・分・秒」が入ってくることは、残念ながらよくあります。そのようなデータの受け渡しに `LocalDateTime` を一時的に使うことはあるでしょう。気をつけるのは、その `LocalDateaTime` をタイムゾーンと組み合わせないまま保持し続けない、そのまま持ち回らないことです。できるだけ早期に、その `LocalDateTime` がどのタイムゾーンのものか特定し、タイムゾーンとセットにして `OffsetDateTime` や `ZonedDateTime` として持ち回るようにしましょう。

JSR 310 に関する日本語記事を探すと、「とりあえず `LocalDateTime` を使っとけばいい」という記述をたまに見かけます。日本国内のみで使用することを考えているからでしょうか。しかし「知識編」で書いたとおり、日本の夏時間情勢はいつ変わってしまうかわかりませんし、近年の AWS などのクラウド環境では、インスタンスのタイムゾーン設定が UTC になっていることも珍しくありません。開発しているアプリケーションで日本時間を仮定するのは、現代では余計な前提を一つ増やしているだけです。日本時間を仮定しても単純化はかなわない時代なのです。

あえて `LocalDateTime` のままで日付・時刻を持ち回るべき状況を、筆者はほとんど思いつきません。起きた事象の記録として使うには、世界共通の時間軸上で確定できない時刻データでは不十分ですし、タイムゾーン情報のない日付・時刻を持ち回れば、開発チームに混乱を呼ぶでしょう。

恣意的な例を作れば「そのホストがどのタイムゾーンにあるかによらず、そのホストの 23時に特定の処理を実行したい」のようなケースはあるかもしれません。しかし、そのホストがあるタイムゾーンには、夏時間があるかもしれません。その時刻がまるっと消えてしまったらどうしましょう。その時刻が二回やってきたらどうしましょう。夏時間は午前 1時ごろに切り替えるのが 2021年現在の定番ではありますが、そうしなければならないと決まっているわけではありません。ホストのタイムゾーン設定をあてにするのは、タイムゾーンの呪いを呼び込む要因にもなります。

### OffsetDateTime

`OffsetDateTime` は `ZoneOffset` のみをタイムゾーン情報として持つ日付・時刻です。「なんで `ZoneOffset` 限定にするのか」というのが「`OffsetDateTime` と `ZonedDateTime` の使い分けがわからない」という方の感想でしょう。 `OffsetDateTime` は一見すると `ZonedDateTime` の下位互換なので、「`ZonedDateTime` だけあればいいじゃないか」というのも自然な反応だと思います。

しかし `ZoneOffset` と同様、この `OffsetDateTime` を `ZonedDateTime` とは別のクラスにして、固定オフセットに限定できるようにしたことこそが JSR 310 のキモです。 `OffsetDateTime` は、タイムゾーンを `ZoneOffset` のみに限定することで、常に「Unix time に変換可能」であることを Java のコードと型のレベルで保証できるようになったのです。 [^leap-second-with-offset-date-time]

[^leap-second-with-offset-date-time]: `OffsetDateTime` も「Java タイム・スケール」にしたがうので、うるう秒はあつかえません。その意味でも `OffsetDateTime` と `Instant` (Unix time) は一対一に対応します。 [Qiita 版](https://qiita.com/dmikurube/items/15899ec9de643e91497c)ではこの点で少し誤った記述がありましたので、念のため明記しておきます。

日付・時刻の表現に `OffsetDateTime` を使うと、タイムゾーンの呪いをかなり遠ざけることができます。特に、クラスのフィールド変数や、メソッドの引数・戻り値などのシグネチャとして受け渡しに `OffsetDateTime` を使っておくと、「存在しない時刻」や「二重に存在する時刻」を保持したり受け渡したりしてしまう事故を、静的に防ぐことができます。 [^independent-offsetdatetime-zoneddatetime]

[^independent-offsetdatetime-zoneddatetime]: `OffsetDateTime` と `ZonedDateTime` の間には、クラスの親子関係もありません。これも意図的な設計でしょう。

`OffsetDateTime` 相当のデータを外部への永続化に使うのも、同様の理由で有効です。

「規約として常に UTC を使うことにしておけば `LocalDateTime` でいい」という意見もあるかもしれません。しかし特にチーム開発において、そのような規約をほんとうに有効に維持し続けられるでしょうか。言語がかけてくれる静的な制約にまかせられるところはまかせて、開発者は本来やりたいことに注力したいものです。

それに、そこまでして `LocalDateTime` を持ち回ることには、たいしたメリットも見い出せません。メモリの使用量は少し変わるかもしれませんが、常に UTC なら定数 `ZoneOffset.UTC` で `OffsetDateTime` を作れば、いちいち新しい `ZoneOffset` インスタンスができることもありません。差は微々たるものだと思っていいでしょう。

### ZonedDateTime

`ZonedDateTime` は、任意の `ZoneId` をタイムゾーン情報として持てる日付・時刻です。情報量的には `OffsetDateTime` の上位互換のように見えますが、任意のタイムゾーンを「持ててしまう」ことで、解けないタイムゾーンの呪いとつきあい続けることを宿命づけられた悲劇のクラスです。

`ZonedDateTime` には、たとえば `America/Los_Angeles` の 2020年 3月 8日 2時 30分、という時刻を与えることができてしまいます。これは `America/Los_Angeles` が標準時から夏時間に切り替わり、すっ飛ばされて存在しない時刻です。同様に `America/Los_Angeles` の 2020年 11月 1日 1時 30分という時刻も作れます。こちらは夏時間から標準時に切り替わり、太平洋夏時間の 11月 1日 1時 30分と太平洋標準時の 11月 1日 1時 30分が二重に存在する時刻です。

インスタンス化された `ZonedDateTime` は、主となる `ZoneId` に並行して、別に `ZoneOffset` も持ちます。この `ZoneOffset` は、特に「二重に存在する時刻」がどちらのオフセットの時刻か確定するための補助的な情報で、自由に設定できるわけではありません。

このような `ZonedDateTime` を作ろうとするとどうなるか、試してみましょう。 `ZonedDateTime.of()`, `ZonedDateTime.ofStrict()`, `ZonedDateTime.parse()` の三通りで試します。

```java
import java.time.DateTimeException;
import java.time.LocalDateTime;
import java.time.ZoneId;
import java.time.ZoneOffset;
import java.time.ZonedDateTime;
import java.time.format.DateTimeFormatter;
import java.time.format.DateTimeFormatterBuilder;
import java.time.format.DateTimeParseException;
import java.util.Locale;

public final class ZonedDateTimes {
    public static void main(final String[] args) {
        // 標準時 → 夏時間
        printNaive( 2020,  3, 8, 2, 30, 0, 0, ZoneId.of("America/Los_Angeles"));
        printStrict(2020,  3, 8, 2, 30, 0, 0, ZoneOffset.ofHours(-8), ZoneId.of("America/Los_Angeles"));
        printStrict(2020,  3, 8, 2, 30, 0, 0, ZoneOffset.ofHours(-7), ZoneId.of("America/Los_Angeles"));
        printParsed("2020-03-08T02:30:00[America/Los_Angeles]", ZONE_ONLY_DATE_TIME);
        printParsed("2020-03-08T02:30:00-08:00[America/Los_Angeles]", ISO_ZONED_DATE_TIME);
        printParsed("2020-03-08T02:30:00-07:00[America/Los_Angeles]", ISO_ZONED_DATE_TIME);

        // 標準時 → 夏時間の一時間後
        printNaive( 2020,  3, 8, 3, 30, 0, 0, ZoneId.of("America/Los_Angeles"));
        printStrict(2020,  3, 8, 3, 30, 0, 0, ZoneOffset.ofHours(-8), ZoneId.of("America/Los_Angeles"));
        printStrict(2020,  3, 8, 3, 30, 0, 0, ZoneOffset.ofHours(-7), ZoneId.of("America/Los_Angeles"));
        printParsed("2020-03-08T03:30:00[America/Los_Angeles]", ZONE_ONLY_DATE_TIME);
        printParsed("2020-03-08T03:30:00-08:00[America/Los_Angeles]", ISO_ZONED_DATE_TIME);
        printParsed("2020-03-08T03:30:00-07:00[America/Los_Angeles]", ISO_ZONED_DATE_TIME);

        // 夏時間 → 標準時
        printNaive( 2020, 11, 1, 1, 30, 0, 0, ZoneId.of("America/Los_Angeles"));
        printStrict(2020, 11, 1, 1, 30, 0, 0, ZoneOffset.ofHours(-8), ZoneId.of("America/Los_Angeles"));
        printStrict(2020, 11, 1, 1, 30, 0, 0, ZoneOffset.ofHours(-7), ZoneId.of("America/Los_Angeles"));
        printParsed("2020-11-01T01:30:00[America/Los_Angeles]", ZONE_ONLY_DATE_TIME);
        printParsed("2020-11-01T01:30:00-08:00[America/Los_Angeles]", ISO_ZONED_DATE_TIME);
        printParsed("2020-11-01T01:30:00-07:00[America/Los_Angeles]", ISO_ZONED_DATE_TIME);

        // 夏時間 → 標準時の一時間後
        printNaive( 2020, 11, 1, 2, 30, 0, 0, ZoneId.of("America/Los_Angeles"));
        printStrict(2020, 11, 1, 2, 30, 0, 0, ZoneOffset.ofHours(-8), ZoneId.of("America/Los_Angeles"));
        printStrict(2020, 11, 1, 2, 30, 0, 0, ZoneOffset.ofHours(-7), ZoneId.of("America/Los_Angeles"));
        printParsed("2020-11-01T02:30:00[America/Los_Angeles]", ZONE_ONLY_DATE_TIME);
        printParsed("2020-11-01T02:30:00-08:00[America/Los_Angeles]", ISO_ZONED_DATE_TIME);
        printParsed("2020-11-01T02:30:00-07:00[America/Los_Angeles]", ISO_ZONED_DATE_TIME);
    }

    private static void printNaive(
            final int year,
            final int month,
            final int dayOfMonth,
            final int hour,
            final int minute,
            final int second,
            final int nanoOfSecond,
            final ZoneId zone) {
        System.out.printf(
                "ZonedDateTime.of(%d, %d, %d, %d, %d, %d, %d, \"%s\"):\n",
                year, month, dayOfMonth, hour, minute, second, nanoOfSecond, zone.toString());
        try {
            final ZonedDateTime naive = ZonedDateTime.of(year, month, dayOfMonth, hour, minute, second, nanoOfSecond, zone);
            System.out.printf("  .toString(): %s\n", naive.toString());
            System.out.printf("  .toInstant().toString(): %s\n", naive.toInstant().toString());
        } catch (final DateTimeException ex) {
            System.out.printf("  DateTimeException: %s\n", ex.getMessage());
        }

        System.out.printf("\n");
    }

    private static void printStrict(
            final int year,
            final int month,
            final int dayOfMonth,
            final int hour,
            final int minute,
            final int second,
            final int nanoOfSecond,
            final ZoneOffset offset,
            final ZoneId zone) {
        System.out.printf(
                "ZonedDateTime.of(%d, %d, %d, %d, %d, %d, %d, \"%s\", \"%s\"):\n",
                year, month, dayOfMonth, hour, minute, second, nanoOfSecond, offset.toString(), zone.toString());
        try {
            final ZonedDateTime strict = ZonedDateTime.ofStrict(
                    LocalDateTime.of(year, month, dayOfMonth, hour, minute, second, nanoOfSecond), offset, zone);
            System.out.printf("  .toString(): %s\n", strict.toString());
            System.out.printf("  .toInstant().toString(): %s\n", strict.toInstant().toString());
        } catch (final DateTimeException ex) {
            System.out.printf("  DateTimeException: %s\n", ex.getMessage());
        }

        System.out.printf("\n");
    }

    private static void printParsed(final String string, final DateTimeFormatter formatter) {
        System.out.printf("ZonedDateTime.parse(\"%s\"):\n", string);

        try {
            final ZonedDateTime parsed = ZonedDateTime.parse(string, formatter);
            System.out.printf("  .toString(): %s\n", parsed.toString());
            System.out.printf("  .toInstant().toString(): %s\n", parsed.toInstant().toString());
        } catch (final DateTimeParseException ex) {
            System.out.printf("  DateTimeParseException: %s\n", ex.getMessage());
        } catch (final DateTimeException ex) {
            System.out.printf("  DateTimeException: %s\n", ex.getMessage());
        }

        System.out.printf("\n");
    }

    private static final DateTimeFormatter ISO_ZONED_DATE_TIME = DateTimeFormatter.ISO_ZONED_DATE_TIME;
    private static final DateTimeFormatter ZONE_ONLY_DATE_TIME = new DateTimeFormatterBuilder()
            .append(DateTimeFormatter.ISO_LOCAL_DATE_TIME)
            .appendLiteral('[')
            .appendZoneRegionId()
            .appendLiteral(']')
            .toFormatter(Locale.ROOT);
}
```

これを実行するとこうなります。

```
$ java -version
openjdk version "1.8.0_292"
OpenJDK Runtime Environment (build 1.8.0_292-8u292-b10-0ubuntu1~20.04-b10)
OpenJDK 64-Bit Server VM (build 25.292-b10, mixed mode)

$ java ZonedDateTimes
ZonedDateTime.of(2020, 3, 8, 2, 30, 0, 0, "America/Los_Angeles"):
  .toString(): 2020-03-08T03:30-07:00[America/Los_Angeles]
  .toInstant().toString(): 2020-03-08T10:30:00Z

ZonedDateTime.of(2020, 3, 8, 2, 30, 0, 0, "-08:00", "America/Los_Angeles"):
  DateTimeException: LocalDateTime '2020-03-08T02:30' does not exist in zone 'America/Los_Angeles' due to a gap in the local time-line, typically caused by daylight savings

ZonedDateTime.of(2020, 3, 8, 2, 30, 0, 0, "-07:00", "America/Los_Angeles"):
  DateTimeException: LocalDateTime '2020-03-08T02:30' does not exist in zone 'America/Los_Angeles' due to a gap in the local time-line, typically caused by daylight savings

ZonedDateTime.parse("2020-03-08T02:30:00[America/Los_Angeles]"):
  .toString(): 2020-03-08T03:30-07:00[America/Los_Angeles]
  .toInstant().toString(): 2020-03-08T10:30:00Z

ZonedDateTime.parse("2020-03-08T02:30:00-08:00[America/Los_Angeles]"):
  .toString(): 2020-03-08T03:30-07:00[America/Los_Angeles]
  .toInstant().toString(): 2020-03-08T10:30:00Z

ZonedDateTime.parse("2020-03-08T02:30:00-07:00[America/Los_Angeles]"):
  .toString(): 2020-03-08T03:30-07:00[America/Los_Angeles]
  .toInstant().toString(): 2020-03-08T10:30:00Z

ZonedDateTime.of(2020, 3, 8, 3, 30, 0, 0, "America/Los_Angeles"):
  .toString(): 2020-03-08T03:30-07:00[America/Los_Angeles]
  .toInstant().toString(): 2020-03-08T10:30:00Z

ZonedDateTime.of(2020, 3, 8, 3, 30, 0, 0, "-08:00", "America/Los_Angeles"):
  DateTimeException: ZoneOffset '-08:00' is not valid for LocalDateTime '2020-03-08T03:30' in zone 'America/Los_Angeles'

ZonedDateTime.of(2020, 3, 8, 3, 30, 0, 0, "-07:00", "America/Los_Angeles"):
  .toString(): 2020-03-08T03:30-07:00[America/Los_Angeles]
  .toInstant().toString(): 2020-03-08T10:30:00Z

ZonedDateTime.parse("2020-03-08T03:30:00[America/Los_Angeles]"):
  .toString(): 2020-03-08T03:30-07:00[America/Los_Angeles]
  .toInstant().toString(): 2020-03-08T10:30:00Z

ZonedDateTime.parse("2020-03-08T03:30:00-08:00[America/Los_Angeles]"):
  .toString(): 2020-03-08T03:30-07:00[America/Los_Angeles]
  .toInstant().toString(): 2020-03-08T10:30:00Z

ZonedDateTime.parse("2020-03-08T03:30:00-07:00[America/Los_Angeles]"):
  .toString(): 2020-03-08T03:30-07:00[America/Los_Angeles]
  .toInstant().toString(): 2020-03-08T10:30:00Z

ZonedDateTime.of(2020, 11, 1, 1, 30, 0, 0, "America/Los_Angeles"):
  .toString(): 2020-11-01T01:30-07:00[America/Los_Angeles]
  .toInstant().toString(): 2020-11-01T08:30:00Z

ZonedDateTime.of(2020, 11, 1, 1, 30, 0, 0, "-08:00", "America/Los_Angeles"):
  .toString(): 2020-11-01T01:30-08:00[America/Los_Angeles]
  .toInstant().toString(): 2020-11-01T09:30:00Z

ZonedDateTime.of(2020, 11, 1, 1, 30, 0, 0, "-07:00", "America/Los_Angeles"):
  .toString(): 2020-11-01T01:30-07:00[America/Los_Angeles]
  .toInstant().toString(): 2020-11-01T08:30:00Z

ZonedDateTime.parse("2020-11-01T01:30:00[America/Los_Angeles]"):
  .toString(): 2020-11-01T01:30-07:00[America/Los_Angeles]
  .toInstant().toString(): 2020-11-01T08:30:00Z

ZonedDateTime.parse("2020-11-01T01:30:00-08:00[America/Los_Angeles]"):
  .toString(): 2020-11-01T01:30-07:00[America/Los_Angeles]
  .toInstant().toString(): 2020-11-01T08:30:00Z

ZonedDateTime.parse("2020-11-01T01:30:00-07:00[America/Los_Angeles]"):
  .toString(): 2020-11-01T01:30-07:00[America/Los_Angeles]
  .toInstant().toString(): 2020-11-01T08:30:00Z

ZonedDateTime.of(2020, 11, 1, 2, 30, 0, 0, "America/Los_Angeles"):
  .toString(): 2020-11-01T02:30-08:00[America/Los_Angeles]
  .toInstant().toString(): 2020-11-01T10:30:00Z

ZonedDateTime.of(2020, 11, 1, 2, 30, 0, 0, "-08:00", "America/Los_Angeles"):
  .toString(): 2020-11-01T02:30-08:00[America/Los_Angeles]
  .toInstant().toString(): 2020-11-01T10:30:00Z

ZonedDateTime.of(2020, 11, 1, 2, 30, 0, 0, "-07:00", "America/Los_Angeles"):
  DateTimeException: ZoneOffset '-07:00' is not valid for LocalDateTime '2020-11-01T02:30' in zone 'America/Los_Angeles'

ZonedDateTime.parse("2020-11-01T02:30:00[America/Los_Angeles]"):
  .toString(): 2020-11-01T02:30-08:00[America/Los_Angeles]
  .toInstant().toString(): 2020-11-01T10:30:00Z

ZonedDateTime.parse("2020-11-01T02:30:00-08:00[America/Los_Angeles]"):
  .toString(): 2020-11-01T02:30-08:00[America/Los_Angeles]
  .toInstant().toString(): 2020-11-01T10:30:00Z

ZonedDateTime.parse("2020-11-01T02:30:00-07:00[America/Los_Angeles]"):
  .toString(): 2020-11-01T02:30-08:00[America/Los_Angeles]
  .toInstant().toString(): 2020-11-01T10:30:00Z
```

`ZonedDateTime.of()` でも `ZonedDateTime.parse()` でも、オフセットのない `2020-03-08 02:30:00 [America/Los_Angeles]` を作ろうとしても、存在しない時刻なので勝手に `2020-03-08 03:30:00 -07:00 [America/Los_Angeles]` にされてしまっていることがわかりますね。明示的に `-07:00` や `-08:00` のオフセットを指定して `02:30:00` を作ろうとすると、そのような時刻は存在しない、と怒られています。

逆に `2020-11-01 01:30:00 [America/Los_Angeles]` を作ろうとすると、これは二重に存在する時刻なのですが、勝手に `-07:00` のほうに寄せられて `2020-11-01 01:30:00 -07:00 [America/Los_Angeles]` にされてしまっていますね。

このような `ZonedDateTime.of()` の挙動は [Javadoc](https://docs.oracle.com/javase/jp/8/docs/api/java/time/ZonedDateTime.html#of-int-int-int-int-int-int-int-java.time.ZoneId-) で以下のように説明されています。

> ほとんどの場合に、ローカル日付/時間の有効なオフセットは1つだけです。重複の場合は、クロックが後方向に戻されたときに、2つの有効なオフセットが存在します。このメソッドは、一般に「夏」に対応する早いほうのオフセットを使用します。
>
> ギャップの場合は、クロックが前方向にジャンプしたときに、有効なオフセットが存在しません。代わりに、ローカル日付/時間がギャップの長さだけ後ろに調整されます。一般的な1時間の夏時間の変更では、ローカル日付/時間が、一般に「夏」に対応するオフセットの中の1時間後方に移動されます。

『「夏」に対応する早い方のオフセットを使用します』とか『後ろに調整されます』とか、けっこうおせっかいを焼いてくれますね。これが「二重に存在する時刻」の場合 [`ZonedDateTime#withEarlierOffsetAtOverlap()`](https://docs.oracle.com/javase/jp/8/docs/api/java/time/ZonedDateTime.html#withEarlierOffsetAtOverlap--) や [`ZonedDateTime#withLaterOffsetAtOverlap()`](https://docs.oracle.com/javase/jp/8/docs/api/java/time/ZonedDateTime.html#withLaterOffsetAtOverlap--) などを呼ぶと、もう一方の時刻を取得できます。

`ZonedDateTime` を実用するときは、このような「存在しない時刻」 (Javadoc では「ギャップ」と呼んでいます) や「二重に存在する時刻」 (Javadoc では「重複」と呼んでいます) のケースをどうするのか、要件とにらめっこして検討し、明示的に処理しておきましょう。

エラーにする、検知して別の日にする、できるだけ切り替わり前のオフセットに寄せる、できるだけ切り替わり後のオフセットに寄せる、できるだけ「標準時」に寄せる、できるだけ「夏時間」に寄せる、などなど、このようなケースをどうすべきかは、要件次第で千差万別です。タイムゾーンの呪いとは、まさにこのような例外ケースのそれぞれについて、いちいち検討して仕様を詰めなければならなくなる、ということです。

`ZonedDateTime` はできれば使いたくない気持ちにもなりますが、それでも `ZonedDateTime` を使うべきケースはあります。「実装編」でも触れましたが、その代表的な例が「暦の計算、タイムゾーンの切り替わりをまたぐ計算」をするケースや、「未来に予定された時刻」をあつかうケースです。

このようなケースでは、タイムゾーンの呪いと向き合いながら `ZonedDateTime` を使う覚悟を決めましょう。これは避けられない戦いです。逆に、必要もないのに `ZonedDateTime` を使うと無駄に呪われてしまうわけです。 `ZonedDateTime` を本当に使うべきケースを、要件から見極めましょう。

「翌日」のような暦の計算をしなければならないケースでは `ZonedDateTime` が必要なことがあります。まずは「翌日」の要件を明らかにするべきですが、要件がたとえば「次の日付の同じ時刻」である場合、タイムゾーンの切り替わりをまたぐ可能性があるので `ZonedDateTime` が必須です。

また、未来に予定された時刻をあつかうケースでは、その予定時刻が「どこ時間」での定義か、が非常に重要です。「実装編」でこの問題を紹介した一文を再掲すると、「現地時刻で定義された未来の時刻を、その予定時刻より古い tzdb を用いてオフセットを確定して (または UTC に変換して) 保持し、それをあとから新しい tzdb を用いて現地時刻に戻そうとすると、本来の予定時刻とずれることがある」ということになります。つまりこのようなケースには、「オフセットを確定させてはならない」場合があります。

たとえば[延期前の東京 2020 オリンピック開会式の開始予定時刻は 2020年 7月 24日 20時](https://sports.nhk.or.jp/olympic/schedules/sports/ceremony.html)でしたが、これは日本時間での定義です。仮にこの開始時刻が決まったあとで夏時間の導入が決定していたら、日本時間上の開始時刻は 20時で変わらず、絶対時刻への解釈のほうが変わる、ということがありえたかもしれません。これと同様のことが、夏時間の廃止が決定している EU でも、これから起こる可能性があります。 [^jon-skeet]

[^jon-skeet]: 「実装編」からも参照しましたが Jon Skeet による ["STORING UTC IS NOT A SILVER BULLET" (Mar 27, 2019)](https://codeblog.jonskeet.uk/2019/03/27/storing-utc-is-not-a-silver-bullet/) では、「知識編」でも触れた EU の夏時間廃止を題材として、この問題を議論しています

Java プロセスの実行中に tzdb が置き換わって `ZoneId` インスタンスの挙動がいつのまにか変わることはないので、インスタンスとしての `ZonedDateTime` を持ち回っているだけなら、この問題に直面することはないでしょう。しかし、このような未来の `ZonedDateTime` を永続化するとき、オフセットと組み合わせて永続化すると、次に読み込むときには tzdb が更新されていて正しく解釈できないようになっているかもしれません。

日付のみ・時刻のみ
-------------------

JSR 310 で「日付のみ」をあつかうクラスは [`java.time.LodalDate`](https://docs.oracle.com/javase/jp/8/docs/api/java/time/LocalDate.html) のみで、「時刻のみ」をあつかうクラスは [`java.time.LocalTime`](https://docs.oracle.com/javase/jp/8/docs/api/java/time/LocalTime.html) と [`java.time.OffsetTime`](https://docs.oracle.com/javase/jp/8/docs/api/java/time/OffsetTime.html) のみです。 `OffsetDate` や `ZonedDate` のような「タイムゾーン情報と日付のみ」の組み合わせ、または `ZonedTime` のような「地域ベースのタイムゾーンと時刻のみ」の組み合わせはありません。

`ZonedDate` と `ZonedTime` がないのは、地域ベースのタイムゾーンというのが、日付と時刻の両方と組み合わせないとあまり意味がないからでしょう。たとえば「ニューヨーク時間の午前 1時 30分」のようなデータだけがあっても、「その時刻がどの日付の時刻か」がわからないと、結局いつのことだか不透明なままです。 `ZonedDateTime` のようにそのまま暦の計算ができるわけでもなく、用意しても存在意義がよくわからないクラスになり、むしろ開発者を混乱させそうです。

そして `OffsetTime` はありますが `OffsetDate` はありません。「`+09:00` の午後 8時」は不確定要素のないデータではあり、それなりに使いようもありそうです。それに対して「`+09:00` の 2021年 7月 23日」は、不確定要素こそありませんが、使いどころもあまりなさそうです。 [^offset-date] 日付において `+09:00` がほんとうに重要な場合は、「`+09:00` の 2021年 7月 23日 0時 0分 0秒から 23時 59分 59秒」などと時刻の範囲で表現するほうが合理的に思えます。

[^offset-date]: 後述する ThreeTen-Extra には、実は [`OffsetDate`](https://www.threeten.org/threeten-extra/apidocs/org.threeten.extra/org/threeten/extra/OffsetDate.html) があります。当初は JSR 310 に入れるつもりで設計していたものの、[あまり使いどころがないという判断で消した](https://github.com/ThreeTen/threeten/issues/228)みたいですね。

ThreeTen-Extra: うるう秒
=========================

Java タイム・スケールがうるう秒の希釈を前提としていることもあって、残念ながら JSR 310 の範疇ではうるう秒を直接あつかうことができません。どうしてもうるう秒をあつかう必要がある場合は [ThreeTen-Extra](https://www.threeten.org/threeten-extra/) という外部ライブラリを JSR 310 と組み合わせて使うことができます。

JSR 310 は独自のタイム・スケールを実装して拡張できるように設計されていて、この ThreeTen-Extra はそのような拡張の一つです。もともと JSR 310 の一部として公式に検討されていたクラス群ですが、その JSR 310 があまりに肥大化したために、整理して外部ライブラリとして切り出されたのが ThreeTen-Extra です。たとえば [`org.threeten.extra.scale.UtcInstant`](https://www.threeten.org/threeten-extra/apidocs/org.threeten.extra/org/threeten/extra/scale/UtcInstant.html) が、うるう秒を考慮した `Instant` のようなクラスになっています。

java.util.Date と java.util.Calendar
=====================================

Java 7 までは [`java.util.Date`](https://docs.oracle.com/javase/jp/8/docs/api/java/util/Date.html) や [`java.util.Calendar`](https://docs.oracle.com/javase/jp/8/docs/api/java/util/Calendar.html) が日付・時刻の Java 標準 API でした。しかしこれらのクラスはスレッド安全性などに難があり、さらに非常にあつかいづらい設計の API でした。 JSR 310 は、これらを置き換えて解消することを目指して設計された API です。

2021 年においては、まだ Java 8 にとどまる理由こそ一部にあるものの、もう Java 7 にとどまる合理的な理由はさすがにないと思います。ということで、これらの使用はもうやめて JSR 310 を使い始めましょう。

    ￣￣￣￣￣￣￣|
    ＿＿＿＿＿＿＿|
    　　　　 　　|
    　　　 　　　|
    |￣￣|￣￣| |
    |　　| ∧_∧|
    |　　|(´∀`)つミ
    |　　|/ ⊃ ﾉ|   java.util.Date
    [二二二二二]|   java.util.Calendar
    　　　　　　|

とはいえ、既存のクラスやメソッドが `java.util.Date` などを使ってしまっていて、互換性などの理由ですぐには消せないこともあるでしょう。そのようなときのために JSR 310 と `java.util.Date` を相互に変換する [`Date.from(Instant)`](https://docs.oracle.com/javase/jp/8/docs/api/java/util/Date.html#from-java.time.Instant-) や [`Date#toInstant()`](https://docs.oracle.com/javase/jp/8/docs/api/java/util/Date.html#toInstant--) などのメソッドが用意されています。 `java.time.Instant` を経由して変換するのがポイントですね。 [`java.sql.Timestamp`](https://docs.oracle.com/javase/jp/8/docs/api/java/sql/Timestamp.html) などの `java.sql` 系の日付・時刻クラスも、だいたい同様にできます。

Java 8 以前に公開されて JSR 310 の原型にもなった [Joda-Time](https://www.joda.org/joda-time/) を、あえて Java 8 以降で使うことは推奨されていません。 [^joda-time-in-java-8]

[^joda-time-in-java-8]: [Joda-Time のページ](https://www.joda.org/joda-time/)にも "Note that from Java SE 8 onwards, users are asked to migrate to java.time (JSR-310) - a core part of the JDK which replaces this project." とあります。

Java と tzdb
=============

「知識編」の「tzdb: タイムゾーン表現の業界標準」で、「年に数回は新しいバージョンの tzdb がリリースされます」と紹介しました。そして「実装編」の最後でも少し紹介したように、この Java で使う tzdb データは Java の実行環境と一体になっています。 Java 実行環境を更新すれば tzdb データも更新されます。

Java 実行環境のバージョンはそのまま tzdb データだけを更新する方法も、いちおうあります。たとえば [Oracle 社が配布する "Timezone Updater Tool" (通称 "TZUpdater")](https://www.oracle.com/java/technologies/javase/tzupdater-readme.html) というツールがあります。これで少なくとも Oracle 社の JDK や JRE (Java Runtime Environment) の tzdb データは更新できます。 JDK のリリース・モデルが変わった JDK 8u202 より前は、事実上このツールが Java の標準でした。それ以降も、少なくとも Oracle 社が配布する JDK では有効です。

ただ JDK のリリース・モデルが変わってディストリビューターが複数になり、ディストリビューションによっては公式のサポート状況に差が生まれ、現在の状況は少し微妙なことになっているようです。 Oracle の TZUpdater 以外にも、たとえば [Azul 社の "ZIUpdater"](https://www.azul.com/products/components/ziupdater-time-zone-tool/) というツールがあります。 ZIUpdater は、基本的には Azul 社の Zulu Builds of OpenJDK 向けに設計されたツールですが、いちおう (素の?) OpenJDK や Oracle JDK でも正常にテストできている [^ziupdater] そうです。いわゆる「公式サポート」というわけではなさそうな雰囲気ですが。

[^ziupdater]: ["By design, it works with Azul Zulu Builds of OpenJDK and Azul Zulu Prime Builds of OpenJDK, but ZIUpdater has also been successfully tested against OpenJDK and Oracle JDK."](https://www.azul.com/products/components/ziupdater-time-zone-tool/)

AdoptOpenJDK や Amazon Corretto など他のディストリビューションでは、このようなツールのサポートはまだ流動的な状況にあるようです。たとえば [AdoptOpenJDK では "How to update timezone data with AdoptOpenJDK" という GitHub Issue](https://github.com/adoptium/temurin-build/issues/1057) が立っていますが、そこでは OpenJDK 汎用や AdoptOpenJDK 独自のツールの開発ではなく、「Oracle TZUpdater を AdoptOpenJDK でも使う方法」が議論されています。 OpenJDK 版の Timezone Updater は、いまのところ「できたらいいね」くらいの雰囲気に見えますね。

Java アプリケーションと tzdb
-----------------------------

いずれのディストリビューションでも、要注意なのは tzdb が Java の実行環境に同梱のものだということです。開発している Java アプリケーションのバージョン管理とは独立です。つまり Java アプリケーションを複数のホストや環境で動かすとき、実行環境の違いで tzdb のバージョンがずれると、タイムゾーンの挙動もずれる可能性がある、ということです。

地域ベースのタイムゾーン (`java.time.ZoneId`) や、それを利用する日付・時刻クラス (`java.time.ZonedDateTime`) を使う場合、アプリケーションのバージョン管理だけではなく、運用環境の tzdb のバージョンも管理する必要があります。混乱を避けるためには、アプリケーションと Java 実行環境を Docker などで同時に管理してしまうのが一つのいい方法かもしれません。

「知識編」で例に出したサモア標準時のように、大規模な変更の実施直前に tzdb が更新される可能性を考えると、単に最新を追いかけるのもそんなに容易なことではありません。

まとめ
=======

「知識編」と「実装編」の一般論をベースとして Java 特有の時刻とタイムゾーンのあつかいかたを検討してきました。 JSR 310 には他にもまだ多くのクラスがありますが、これら基本データクラスの考えかたをおさえておけば、応用できるでしょう。

「Java 編」もふくめたこの「タイムゾーン呪いの書」が、多くの Java 開発者が日付・時刻をあつかうときの助けとなることを祈ります。

ちなみに、日付・時刻データから文字列へのフォーマットと、文字列から日付・時刻データへの解釈をおこなう [`java.time.format.DateTimeFormatter`](https://docs.oracle.com/javase/jp/8/docs/api/java/time/format/DateTimeFormatter.html) は、本記事ではあまり触れていません。この `DateTimeFormatter` は、それだけで長文記事が一本できる程度にはややこしいクラスですが、その解説は他の機会 (または他の方) に譲りたいと思います。

さて以下では、「知識編」で少し触れた Java とタイムゾーン略称の歴史と闇について、すこし深堀りしてみた内容をおまけとして残してみました。 `DateTimeFormatter` にも少しだけ触れています。興味のある方はお楽しみください。

おまけ: Java とタイムゾーン略称
================================

このおまけは、「実装編」で少し触れた Java のタイムゾーン略称に関する深堀りです。

たとえば `MST` などの略称が `America/Denver` として解釈されてしまい、その結果 `2020-07-01 12:34:56 MST` が `2020-07-01 12:34:56 -07:00` として解釈されるべき場合でも `2020-07-01 12:34:56 -06:00` と解釈されてしまう、という話でした。

旧 java.util.TimeZone
----------------------

これ、実は Java ではもっと昔に解決されていたはずだったのです。 JSR 310 が導入される Java 8 よりも前に、この話はすでに認知されて Java 6 で対処されていました。 Oracle Community の投稿 ["EST, MST, and HST time zones in Java 6 and Java 7"](https://community.oracle.com/tech/developers/discussion/2539847/est-mst-and-hst-time-zones-in-java-6-and-java-7) を見ると、そのころの経緯がわかります。

Java に初期からある [`java.util.TimeZone`](https://docs.oracle.com/javase/jp/8/docs/api/java/util/TimeZone.html) では、そのころから `MST` などの略称を `America/Denver` などの地域ベースのタイムゾーンに対応づけてしまっていました。あまりにも昔からそうなっていたため、互換性を考えて、安易にこれを変えることはできなかったようです。 [^abbreviations-in-java-1-3] [^abbreviations-in-java-1-1-6]

[^abbreviations-in-java-1-3]: ちなみにタイムゾーン略称の利用は [Java 1.3 のころにはすでに deprecated だと明言されています](https://javaalmanac.io/jdk/1.3/api/java/util/TimeZone.html)。非推奨とされてから長い時間が経ったのに、まだ略称に振り回されるというのも悲しい話ですね。

[^abbreviations-in-java-1-1-6]: ちなみに [(なぜか MIT の Web に置かれている) Java 1.1.6 のころの `java.util.TimeZone` の実装](http://web.mit.edu/java_v1.1.6/distrib/sun4x_57/src/java/util/TimeZone.java)を見ると、タイムゾーン略称から地域ベースのタイムゾーンへの対応がいくつかハードコードされているのがわかります。このうちのいくつかは、どう見ても普通ではない対応づけなんですが (なんで [`AST`](https://www.timeanddate.com/time/zones/ast) が "Alaska Standard Time" で `America/Anchorage` やねん) そっちを掘り下げるのはまた別の機会に。一度埋めてしまった地雷はなかなか撤去できない、という典型例ですね。

Java 6 で重い腰を上げて、夏時間を実施しない州をふくむタイムゾーン `EST`, `MST`, `HST` だけは、それぞれ固定オフセットの `-05:00`, `-07:00`, `-10:00` に対応づけるようになりました。それと同時に、互換性のための `sun.timezone.ids.oldmapping` という Java システム・プロパティが用意されました。これを `"true"` にセットしておけば旧来のマッピングを使用するようになります。以下のコードで、このことを確認できます。

```java
import java.util.TimeZone;

public final class OldMapping {
    public static void main(final String[] args) throws Exception {
        System.out.println(TimeZone.getTimeZone("EST"));
        System.out.println(TimeZone.getTimeZone("EST").hasSameRules(TimeZone.getTimeZone("GMT-5")));
        System.out.println(TimeZone.getTimeZone("EST").hasSameRules(TimeZone.getTimeZone("America/New_York")));

        System.out.println(TimeZone.getTimeZone("MST"));
        System.out.println(TimeZone.getTimeZone("MST").hasSameRules(TimeZone.getTimeZone("GMT-7")));
        System.out.println(TimeZone.getTimeZone("MST").hasSameRules(TimeZone.getTimeZone("America/Denver")));

        System.out.println(TimeZone.getTimeZone("HST"));
        System.out.println(TimeZone.getTimeZone("HST").hasSameRules(TimeZone.getTimeZone("GMT-10")));
        System.out.println(TimeZone.getTimeZone("HST").hasSameRules(TimeZone.getTimeZone("Pacific/Honolulu")));
    }
}
```

この実行結果は以下のようになります。

```
$ java -version
openjdk version "1.8.0_292"
OpenJDK Runtime Environment (build 1.8.0_292-8u292-b10-0ubuntu1~20.04-b10)
OpenJDK 64-Bit Server VM (build 25.292-b10, mixed mode)

$ java OldMapping
sun.util.calendar.ZoneInfo[id="EST",offset=-18000000,dstSavings=0,useDaylight=false,transitions=0,lastRule=null]
true
false
sun.util.calendar.ZoneInfo[id="MST",offset=-25200000,dstSavings=0,useDaylight=false,transitions=0,lastRule=null]
true
false
sun.util.calendar.ZoneInfo[id="HST",offset=-36000000,dstSavings=0,useDaylight=false,transitions=0,lastRule=null]
true
false

$ java -Dsun.timezone.ids.oldmapping=true OldMapping
sun.util.calendar.ZoneInfo[id="EST",offset=-18000000,dstSavings=3600000,useDaylight=true,transitions=235,lastRule=java.util.SimpleTimeZone[id=EST,offset=-18000000,dstSavings=3600000,useDaylight=true,startYear=0,startMode=3,startMonth=2,startDay=8,startDayOfWeek=1,startTime=7200000,startTimeMode=0,endMode=3,endMonth=10,endDay=1,endDayOfWeek=1,endTime=7200000,endTimeMode=0]]
false
true
sun.util.calendar.ZoneInfo[id="MST",offset=-25200000,dstSavings=3600000,useDaylight=true,transitions=157,lastRule=java.util.SimpleTimeZone[id=MST,offset=-25200000,dstSavings=3600000,useDaylight=true,startYear=0,startMode=3,startMonth=2,startDay=8,startDayOfWeek=1,startTime=7200000,startTimeMode=0,endMode=3,endMonth=10,endDay=1,endDayOfWeek=1,endTime=7200000,endTimeMode=0]]
false
true
sun.util.calendar.ZoneInfo[id="HST",offset=-36000000,dstSavings=0,useDaylight=false,transitions=7,lastRule=null]
false
true
```

JSR 310 の DateTimeFormatter
-----------------------------

`EST`, `MST`, `HST` が修正されてめでたしめでたし… (?) と思っていたところに、新たに Java 8 で JSR 310 が実装されました。

この Java 編で見てきたように、筆者は JSR 310 はわりとよくできていると考えています。タイムゾーン略称についても、実は本丸の `ZoneId` では使用をかなり制限していて、[名前から `ZoneId` を作るときにエイリアスを明示的に与えないと略称は使えない](https://docs.oracle.com/javase/jp/8/docs/api/java/time/ZoneId.html#of-java.lang.String-java.util.Map-)ようになっています。さらに、[互換性のための標準 Map](https://docs.oracle.com/javase/jp/8/docs/api/java/time/ZoneId.html#SHORT_IDS)も用意されていて、ここでも `EST`, `MST`, `HST` はちゃんと固定オフセットのほうに対応づけられています。

ではなぜ「知識編」で出した例では `2020/07/01 12:34:56 MST` が `2020-07-01T12:34:56-06:00[America/Denver]` になってしまったのでしょうか。

その答えは [`java.util.format.DateTimeFormatter`](https://docs.oracle.com/javase/jp/8/docs/api/java/time/format/DateTimeFormatter.html) の実装でした。

`DateTimeFormatter` の最も手軽な使いかたである [`#ofPattern(String)`](https://docs.oracle.com/javase/jp/8/docs/api/java/time/format/DateTimeFormatter.html#ofPattern-java.lang.String-) で `time-zone name` を表す `z` や `zzzz` を使用すると、それは [`DateTimeFormatterBuilder#appendZoneText(TextStyle)`](https://docs.oracle.com/javase/jp/8/docs/api/java/time/format/DateTimeFormatterBuilder.html#appendZoneText-java.time.format.TextStyle-) の呼び出しに相当します。この `appendZoneText` はかなり幅広いタイムゾーン名の表現を受け付けるようになっています。 Javadoc にも以下のように注意書きがあります。

> 解析時には、テキストでのゾーン名、ゾーンID、またはオフセットが受け入れられます。テキストでのゾーン名には、一意でないものが多くあります。たとえば、CSTは「Central Standard Time (中部標準時)」と「China Standard Time (中国標準時)」の両方に使用されます。この状況では、フォーマッタのlocaleから得られる地域情報と、その地域の標準ゾーンID (たとえば、America Easternゾーンの場合はAmerica/New_York)により、ゾーンIDが決定されます。appendZoneText(TextStyle, Set)を使用すると、この状況で優先するZoneIdのセットを指定できます。

略称を受け入れるにしても、地域ベースではなくオフセットに対応づけてさえいれば、こんなことにはならなかったんですが。どうしてこうなってしまったんでしょうか。

DateTimeFormatter に深入り
---------------------------

そんなわけで、もう少し OpenJDK のコードに深入りしてみました。ひとまず [`DateTimeFormatterBuilder#appendZoneText` がタイムゾーン名の候補を探してきているのは `DateTimeFormatterBuilder.java` のこの行](https://github.com/openjdk/jdk/blob/jdk8-b120/jdk/src/share/classes/java/time/format/DateTimeFormatterBuilder.java#L3708)のようです。

```java
                zoneStrings = TimeZoneNameUtility.getZoneStrings(locale);
```

[`sun.util.locale.provider.TimeZoneNameUtility`](https://github.com/openjdk/jdk/blob/jdk8-b120/jdk/src/share/classes/sun/util/locale/provider/TimeZoneNameUtility.java) という内部クラスで Locale をもとにタイムゾーン名の候補を出しているみたいですね。ここまで来たらもう少しだけ追ってみましょう。

`TimeZoneNameUtility` を参考にすると、以下のような Java コードで、タイムゾーン名の候補をむりやり読めそうなことがわかります。

```java
import java.lang.reflect.Method;
import java.util.Arrays;
import java.util.Locale;

public final class TimeZoneNamesInternal {
    public static void main(final String[] args) throws Exception {
        for (final String[] e : getZoneStrings()) {
            System.out.println(String.join(", ", Arrays.asList(e)));
        }
    }

    private static String[][] getZoneStrings() throws Exception {
        final Class<?> clazz = Class.forName("sun.util.locale.provider.TimeZoneNameUtility");
        final Method method = clazz.getMethod("getZoneStrings", Locale.class);
        final Object zoneStringsObject = method.invoke(null, Locale.ROOT);
        return (String[][]) zoneStringsObject;
    }
}
```

実際これでも読めるのですが、実は標準 API の [`java.text.DateFormatSymbols`](https://docs.oracle.com/javase/jp/8/docs/api/java/text/DateFormatSymbols.html) でも同等のことができます。

```java
import java.text.DateFormatSymbols;
import java.util.Arrays;
import java.util.Locale;

public final class TimeZoneNamesStandard {
    public static void main(final String[] args) throws Exception {
        for (final String[] e : DateFormatSymbols.getInstance(Locale.ROOT).getZoneStrings()) {
            System.out.println(String.join(", ", Arrays.asList(e)));
        }
    }
}
```

どちらを実行しても、出力される内容は同じです。実行してみると…。

```
America/Los_Angeles, Pacific Standard Time, PST, Pacific Daylight Time, PDT, Pacific Time, PT
PST, Pacific Standard Time, PST, Pacific Daylight Time, PDT, Pacific Time, PT
America/Denver, Mountain Standard Time, MST, Mountain Daylight Time, MDT, Mountain Time, MT
... (snip) ...
America/New_York, Eastern Standard Time, EST, Eastern Daylight Time, EDT, Eastern Time, ET
... (snip) ...
Pacific/Honolulu, Hawaii Standard Time, HST, Hawaii Daylight Time, HDT, Hawaii Time, HT
... (snip) ...
```

`America/Denver` と `MST` が同じ行にあります。この [`DateFormatSymbols#getZoneStrings()` の Javadoc](https://docs.oracle.com/javase/jp/8/docs/api/java/text/DateFormatSymbols.html#getZoneStrings--) を見ると、各行は左から順に「タイムゾーンID」「標準時刻のゾーンの長い名前」「標準時刻のゾーンの短い名前」「夏時間のゾーンの長い名前」「夏時間のゾーンの短い名前」であるようです。

そして [`DateTimeFormatterBuilder#appendZoneText`](https://github.com/openjdk/jdk/blob/jdk8-b120/jdk/src/share/classes/java/time/format/DateTimeFormatterBuilder.java#L3709-L3720) が、同じ行にある `MST` を `America/Denver` にひもづけています。つまり JSR 310 の `DateTimeFormatter` が `MST` を `America/Denver` にしてしまう理由は、もとをたどれば Locale データにあるようです。

その Locale データのおおもとを探すと [`sun.util.resources.TimeZoneNames`](https://github.com/openjdk/jdk/blob/jdk8-b120/jdk/src/share/classes/sun/util/resources/TimeZoneNames.java) という内部クラスでハードコードされています。 [^TimeZoneNames-jdk-17]

[^TimeZoneNames-jdk-17]: [OpenJDK 17 でも同様](https://github.com/openjdk/jdk/blob/jdk-17+31/src/java.base/share/classes/sun/util/resources/TimeZoneNames.java) です。

せっかく JSR 310 で古いしがらみを捨てられたんだから、ここからも脱却しておけばよかったのに、という気はしますが、まあなにか理由があったんでしょうか。 [^tell-me]

[^tell-me]: ご存知のかたは、ぜひ教えてください。

DateTimeFormatter 対策
-----------------------

こういう混乱なく `DateTimeFormatter` を使うには、どうしたらいいでしょうか。少しだけ検討してみましょう。

まず Locale に踏み込むのはかなり大変ですし、手を出したからといって変えられるようなものではないので、ここはパスします。

そもそも `DateTimeFormatterBuilder#appendZoneText()` の解釈が自由すぎる・寛容すぎるので、仮にこの略称問題だけがどうにかなっても、ほかのところで想定外の地雷を踏みそうだなあ、という気がします。 `appendZoneText()` 以外の候補がないか探してみると、いくつかあります。そのなかから、用途に合って、かつ厳密に処理してくれるものを選んで使う、というのがいいかもしれません。

たとえば固定オフセット (`+09:00`) のみを処理する [`appendOffset`](https://docs.oracle.com/javase/jp/8/docs/api/java/time/format/DateTimeFormatterBuilder.html#appendOffset-java.lang.String-java.lang.String-) や、タイムゾーン ID の文字列 (`Asia/Tokyo`) を処理する [`appendZoneId`](https://docs.oracle.com/javase/jp/8/docs/api/java/time/format/DateTimeFormatterBuilder.html#appendZoneId--) や [`appendZoneRegionId`](https://docs.oracle.com/javase/jp/8/docs/api/java/time/format/DateTimeFormatterBuilder.html#appendZoneRegionId--) などです。パターン文字列を使う場合は `XXX`, `ZZZ`, `VV` などに該当します。

パターン文字列の詳細な解説は [`DateTimeFormatterBuilder#appendPattern` の Javadoc](https://docs.oracle.com/javase/jp/8/docs/api/java/time/format/DateTimeFormatterBuilder.html#appendPattern-java.lang.String-) にあります。

どうなる PST
-------------

ところで「知識編」でも触れましたが、カリフォルニア州では 2018年の住民投票で夏時間の変更が支持されたそうです。

まだ具体的な計画はないようですが、もしこれが実施されたら、今まで `EST`, `MST`, `HST` だけだった特例に `PST` を追加する必要が出てくるかもしれません。でも [`ZoneID.SHORT_IDS`](https://docs.oracle.com/javase/jp/8/docs/api/java/time/ZoneId.html#SHORT_IDS) は `PST` を `America/Los_Angeles` にマップすると明記しちゃってるんですよね。

…はたして、これからどうなるのでしょうか。この世界からタイムゾーンの呪いが解けることは、まだ当分なさそうです。
