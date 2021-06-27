---
title: "タイムゾーン呪いの書 (Java 編)"
emoji: "🌎" # アイキャッチとして使われる絵文字（1文字だけ）
type: "tech" # tech: 技術記事 / idea: アイデア
topics: [ "timezone", "java", "jsr310" ]
layout: default
published: false
---

「タイムゾーン呪いの書」は、もともと 2018年に [Qiita に投稿した記事](https://qiita.com/dmikurube/items/15899ec9de643e91497c)でしたが、大幅な改訂と同時に [Zenn](https://zenn.dev/) に引っ越してきました。この改訂で全体がさらに長くなったので、記事を分けることにしました。

本記事は、導入にあたる第一部[「教養編」](./curse-of-timezones-common-ja)と、それを実装に反映するための一般論を検討した第二部[「実装編」](./curse-of-timezones-impl-ja)の続きにあたる、最終章の第三部です。記事全体を通して、「教養編」と「実装編」を読んでいることを前提にしているので、ご注意ください。

Java と tzdb
=============

[「教養編」](./curse-of-timezones-common-ja)の tzdb の説明で、「年に数回は新しいバージョンの tzdb がリリースされます」と紹介しました。そして[「実装編」](./curse-of-timezones-impl-ja) の最後でも少し紹介しましたが、基本的に Java で使う tzdb データは JDK や Java の実行環境と一体になっています。 JDK や Java 実行環境を更新すれば、同梱の tzdb データも更新されます。

JDK や Java 実行環境のバージョンはそのままで tzdb データだけを更新する方法も、いちおうあります。 [Oracle 社が配布する "Timezone Updater Tool" (通称 "TZUpdater")](https://www.oracle.com/java/technologies/javase/tzupdater-readme.html) というツールがあり、このツールで Oracle 社の JDK や JRE (Java Runtime Environment) に同梱の tzdb データは更新できます。 JDK のリリース・モデルが変わる前の JDK 8u202 までは、このツールが事実上 Java の標準でした。それ以降もこのツールは、少なくとも Oracle 社が配布する JDK に対しては有効です。

ただ JDK のリリース・モデルが変わって JDK や実行環境のディストリビューターが複数になったため、このようなツールのサポートにはディストリビューションによって差があり、微妙に複雑な状況になっています。 Oracle の TZUpdater 以外にも、現在はたとえば [Azul 社の "ZIUpdater" というツール](https://www.azul.com/products/components/ziupdater-time-zone-tool/) があります。 ZIUpdater は基本的には Azul 社の Zulu Builds of OpenJDK 向けに設計されているものの、いちおう OpenJDK や Oracle JDK でもテストできているそうです。正式サポートというわけではなさそうですが。

AdoptOpenJDK や Amazon Corretto などのその他のディストリビューションでは、このようなツールのサポートはまだ流動的な状況にあるようです。たとえば [AdoptOpenJDK では "How to update timezone data with AdoptOpenJDK" という GitHub Issue](https://github.com/adoptium/temurin-build/issues/1057) が立っていますが、そこでは Oracle 社の TZUpdater を AdoptOpenJDK でも使う方法が議論されています。 OpenJDK 版の Timezone Updater は、いまのところ「できたらいいね」くらいの雰囲気ですね。

Java アプリケーションと tzdb
-----------------------------

いずれのディストリビューションでも、要注意なのは tzdb が Java の実行環境に付属するものだということです。開発している Java アプリケーションのバージョン管理とは独立です。つまり、ある Java アプリケーションを複数のホストで動かすとき、実行環境の違いで tzdb のバージョンがずれてしまい、そのために挙動が変わる可能性がある、ということです。

地域ベースのタイムゾーン (`java.time.ZoneId` や旧 `java.util.TimeZone`) や、それを利用した日付・時刻クラス (`java.time.ZonedDateTime`) を使う場合は、アプリケーションのバージョン管理だけではなく tzdb のバージョン管理も同時に行う必要があります。アプリケーションと JDK を Docker などで同時に管理してしまうのが、こういう混乱を避けるためにもいい案の一つかもしれません。

[「教養編」](./curse-of-timezones-common-ja)で例に出したサモア標準時のような大規模な変更が直前に行われる可能性を考えると、単に最新を追いかけるだけでも、そんなに簡単なことではありません。

JSR 310: Date and Time API
===========================

[JSR 310: Date and Time API](https://jcp.org/en/jsr/detail?id=310) は、日付・時刻をあつかう新しい Java API です。 Java 8 から追加されました。「教養編」や「実装編」でも何度か参照していますね。

[Joda-Time](http://www.joda.org/joda-time/) というオープンソースの日付・時刻ライブラリが 2005年ごろには公開されていたのですが、その作者である Stephen Colebourne 氏が共同リーダーとして参加し、正式な Java API として Joda-Time をベースに再設計されたのが JSR 310 です。

古くからの Java API である [`java.util.Date`](https://docs.oracle.com/javase/jp/8/docs/api/java/util/Date.html) や [`java.text.SimpleDateFormatter`](https://docs.oracle.com/javase/jp/8/docs/api/java/text/SimpleDateFormat.html) にはスレッド安全性や API 設計など多くの問題があり、それらの問題に対処すべく JSR 310 を設計した、ということです。 [^new-date-time-api]

[^new-date-time-api]: ["Java SE 8 Date and Time" by Ben Evans and Richard Warburton (January/February 2014, Oracle Technical Article)](https://www.oracle.com/technical-resources/articles/java/jf14-date-time.html)

「教養編」や「実装編」で触れてきたように、時刻やタイムゾーンというのはそもそも複雑な概念です。特に地域ベースのタイムゾーンが絡んでくると、例外的な状況がさまざまなパターンで起こります。

JSR 310 はその複雑な概念をかなり忠実にモデル化しており、例外的な状況も明示的にあつかえるように設計されています。それは同時に、複雑な概念や例外的な状況を明示的にあつかわなければならないということでもあり、そのことが「JSR 310 は複雑すぎる」「`OffsetDateTime` と `ZonedDateTime` の違いがよくわからない」のような声にもしばしばつながります。ですがこの複雑さを中途半端に隠蔽してしまうと、例外的な状況のあつかいがうやむやになりがちです。そしてその例外的な状況はめったに起こらないので、起きて初めてうやむやなあつかいが露見する、ということにもなりがちです。 [^date-time-formatter]

[^date-time-formatter]: とはいえ [`java.time.format.DateTimeFormatter`](https://docs.oracle.com/javase/jp/8/docs/api/java/time/format/DateTimeFormatter.html) は使いづらいよねー、という気持ちは正直わかる。

この「Java 編」では、おもに JSR 310 の各クラス (中でも基本となる日付/時間データクラス) の使いかたについて、「教養編」と「実装編」で検討してきた一般論をベースに考えていきたいと思います。

Instant: Unix time
-------------------

Unix time に相当する、世界共通の時間軸上の一点を表すのが [`java.time.Instant`](https://docs.oracle.com/javase/jp/8/docs/api/java/time/Instant.html) です。これは Unix time と「ほぼ」同じもので、違いはうるう秒の扱いです。「教養編」で、うるう秒を「希釈」する手法をいくつか取り上げましたが、その一つである ["UTC-SLS"](https://www.cl.cam.ac.uk/~mgk25/time/utc-sls/) を用いた「Java タイム・スケール」が使われます。 [^java-time-scale]

[^java-time-scale]: Java タイム・スケールの詳しい説明は [`java.time.Instant` の Javadoc](https://docs.oracle.com/javase/jp/8/docs/api/java/time/Instant.html) にあります。

Java タイム・スケールでは 1972年 11月 3日以降の時刻に UTC-SLS が適用され、うるう秒は、うるう秒が適用される日の最後の 1000 秒で希釈されます。 Java タイム・スケールは `Instant` だけではなく、すべての日付/時間クラスで使われます。

Unix time に相当する `Instant` は、「実装編」でも検討したように、うるう秒の希釈さえ問題なければ時刻の内部データ表現として有力な候補です。 `long` や `double` などのプリミティブな数値型で Unix time をあつかうわけでもないので、変な取り違えをするリスクも小さいでしょう。

ZoneId と ZoneOffset
---------------------

JSR 310 のタイムゾーン情報は、すべてのタイムゾーンを表す [`java.time.ZoneId`](https://docs.oracle.com/javase/jp/8/docs/api/java/time/ZoneId.html) 抽象クラスと、その中でも固定オフセットのみを表すサブクラスの具象クラス [`java.time.ZoneOffset`](https://docs.oracle.com/javase/jp/8/docs/api/java/time/ZoneOffset.html) という二種類のクラスで表現されます。これらのインスタンスを使うときは `ZoneOffset.UTC` のような定数を使ったり `ZoneOffset.of("+09:00")` `ZoneOffset.ofHours(9)` `ZoneId.of("Asia/Tokyo")` などとしてインスタンスを生成したりします。このようなインスタンスは、いずれも不変インスタンス (immutable instance) です。

`ZoneId.of("Asia/Tokyo")` などとして生成した地域ベースの `ZoneId` インスタンスには tzdb のタイムゾーン ID が対応しています。「いつ夏時間に切り替わるか」「過去のどの時点でオフセットが変わったか」などの切り替わりルールも、タイムゾーン ID に対応して tzdb をもとに実装されています。 `ZoneId` インスタンスの `ZoneId#getRules()` を呼び出すと、切り替わりルールを実装した [`java.time.ZoneRules`](https://docs.oracle.com/javase/jp/8/docs/api/java/time/zone/ZoneRules.html) を取得することができます。

`ZoneId` は抽象クラスなので、そのもののインスタンスを作ることはできません。固定オフセットではない地域ベースのタイムゾーンをあらわすために `ZoneId.of("Asia/Tokyo")` などとして生成したインスタンスは `ZoneId` の package-private サブクラスである [`java.time.ZoneRegion`](http://hg.openjdk.java.net/jdk8/jdk8/jdk/file/jdk8-b132/src/share/classes/java/time/ZoneRegion.java#l90) として実装されているようです。 [^zoneregion]

[^zoneregion]: この `ZoneRegion` は package-private なので直接は使えませんし、この実装が今後も保証されるわけではありません。使うときはあくまで `ZoneId` 型として使います。

固定オフセットを持つ一部の `ZoneRegion` インスタンスは `ZoneId#normalize()` を呼び出すことで正規化し、対応する `ZoneOffset` インスタンスを取得できます。 `ZoneOffset` インスタンスの `normalize()` メソッドを呼んでもなにも変わりません。

`"Asia/Tokyo"` `"UTC+09:00"` `"+09:00"` のそれぞれの文字列で `ZoneId.of()` と `ZoneOffset.of()` を呼び出してみた例が、以下のとおりです。

```java
import java.time.DateTimeException;
import java.time.ZoneId;
import java.time.ZoneOffset;
import java.time.zone.ZoneOffsetTransition;
import java.util.List;

public class ZoneIds {
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

```
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

東京時間 (`ZoneId.of("Asia/Tokyo")`) が、何回かタイムゾーンの切り替わりを経験していることがわかりますね。これは国際子午線会議 (1884年) をもとにした日本標準時の導入 (1888年) と、「教養編」でも触れた、第二次世界大戦直後に数年間だけ導入された夏時間のデータです。

地域ベースのタイムゾーンを使うかぎり、オフセットの切り替わりにともなう「存在しない時刻」や「二重に存在する時刻」のことを検討しなければならず、「タイムゾーンの呪い」から逃げられない、というのが「教養編」と「実装編」で繰り返し検討してきたことでした。固定オフセットのみで話を完結できる要件なら、できれば地域ベースのタイムゾーンには触れずにすませたいところです。

ここで JSR 310 の設計のキモの一つが、固定オフセットのみをあらわす `ZoneOffset` が、すべてのタイムゾーンをあらわす `ZoneId` とは (派生クラスですが) 別のクラスとして実装されている点にあります。タイムゾーン情報として常に `ZoneOffset` を使っていれば、タイムゾーンの呪いから大きく距離を置けていることを、コードのレベルで保証できるのです。逆にコード中に `ZoneId` がまぎれ込んできたら、タイムゾーンの呪いに気をつけよう、という警戒信号だととらえることができます。

次の `OffsetDateTime` と `ZonedDateTime` にも、同様のことを言うことができます。

OffsetDateTime と ZonedDateTime
--------------------------------

JSR 310 で「日付と時刻」をあらわすクラスには三種類あります。 [`java.time.LocalDateTime`]((https://docs.oracle.com/javase/jp/8/docs/api/java/time/LocalDateTime.html)), [`java.time.OffsetDateTime`]((https://docs.oracle.com/javase/jp/8/docs/api/java/time/OffsetDateTime.html)), [`java.time.ZonedDateTime`]((https://docs.oracle.com/javase/jp/8/docs/api/java/time/ZonedDateTime.html)) です。これらもすべて `Instant` と同様に「Java タイム・スケール」にのっとっていて、うるう秒をあつかうことはできません。

このうち `LocalDateTime` は「タイムゾーン情報を持たない」日付と時間で、ある意味わかりやすいです。残りの二つについて「`OffsetDateTime` と `ZonedDateTime` をどう使い分ければいいのかよくわからない」というのが JSR 310 の FAQ ではないでしょうか。

一言で言えば、固定オフセットの `ZoneOffset` のみタイムゾーン情報として受け付けるのが `OffsetDateTime` で、地域ベースのものを含む `ZoneId` で任意のタイムゾーン情報を受け付けるのが `ZonedDateTime` です。と、これだけ言われて使い分けがわかるのなら、この疑問が FAQ にはなっていませんね。以下では、これらの使い分けについて検討していきます。

### LocalDateTime

「実装編」で議論した「タイムゾーンが不明なままの『年・月・日・時・分・秒』を、そのまま保持し続けない、そのまま持ち回らない」という大原則はここでも有効です。 `LocalDateTime` は、タイムゾーン情報を一切含まない日付・時刻です。そのため `LocalDateTime` だけで `Instant` には変換できませんし、ある `LocalDateTime` インスタンスが世界共通の時間軸上のどこを表すのかは特定できない、ということになります。

MySQL などの RDB やユーザーからの入力など外部からのデータに、タイムゾーンが不明だったり曖昧だったりする「年・月・日・時・分・秒」が入ってくることはよくあります。そのようなデータの受け渡しに `LocalDateTime` を一時的に使うことはあるでしょう。気をつけるのは、その `LocalDateaTime` をタイムゾーンと紐付けないまま、プログラム中で保持し続けない、そのまま持ち回らないことです。できるだけ早く、その日時がどのタイムゾーンのものか確定し、日時とタイムゾーンをセットで保持するようにしましょう。

JSR 310 に関する日本語記事を探すと、「とりあえず `LocalDateTime` を使っとけばいいよ」という記述をたまに見かけます。日本国内のみで使用することを考えているからでしょうか。しかし「教養編」で書いたとおり、日本の夏時間情勢がいつ変わってしまうかわかりませんし、近年の AWS などのクラウド環境ではインスタンスのタイムゾーン設定が `UTC` になっていることも珍しくありません。開発しているアプリケーションで日本時間環境を仮定することは、アプリケーションを運用するのに余計な前提を一つ増やしているだけであり、もはや当初の目的だったであろう「考えることをシンプルにする」ことすらかなわない時代なのです。

あえて `LocalDateTime` のままで日付・時刻を持ち回るべき状況を、筆者はほとんど思いつきません。なにか起きた時刻の記録として使うにはタイムゾーンに依らない「どの瞬間だったか」を世界共通の時間軸上で確定できないと不十分ですし、タイムゾーン情報のない日付・時刻を持ち回れば、「この `LocalDateTime` ってどこの時間だったっけ?」と開発チームに混乱を引き起こします。

恣意的な例を挙げれば、「そのホストがどのタイムゾーンにあるかによらずそのホストの時刻の 23:00:00 に特定の処理を実行したい」のようなケースはあるかもしれません。しかし「教養編」のサモアの例のようにその日付・時刻がまるっと消えてしまったらどうしましょう。または、その時刻が二回やってきたらどうしましょう。夏時間は午前 1時ごろに切り替えるのが 2021年現在の定番ですが、そうしなければならないと決まっているわけではないのです。ホストのタイムゾーン設定をあてにするより、タイムゾーンやその切り替わりをどうあつかうか、明示的に決めておいたほうがいいのではないでしょうか。

### OffsetDateTime

`OffsetDateTime` はタイムゾーン情報を持つものの、固定オフセットの `ZoneOffset` のみを受け付ける日付・時刻です。「なんでわざわざ `ZoneOffset` のみに限定するのか」というのが「`OffsetDateTime` と `ZonedDateTime` の違いがよくわからない」という方の感想でしょう。一見すると `OffsetDateTime` はただの `ZonedDateTime` の下位互換クラスなので、「`ZonedDateTime` だけあればいいじゃないか」というのは自然な反応だと思います。

しかし前述の `ZoneOffset` と同様、この `OffsetDateTime` を別クラスとして用意して、固定オフセットに限定できるようにしたことこそが JSR 310 のキモです。 `OffsetDateTime` は、タイムゾーンを `ZoneOffset` のみに限定することで、常に `Instant` (Unix time) と一対一に変換できることを Java の型のレベルで保証できるのです。 [^leap-second-with-offset-date-time]

[^leap-second-with-offset-date-time]: `OffsetDateTime` も「Java タイム・スケール」にしたがうので、うるう秒はあつかえません。その意味でも `OffsetDateTime` と `Instant` (Unix time) は一対一に対応します。 [Qiita 版](https://qiita.com/dmikurube/items/15899ec9de643e91497c)ではこの点で少し誤った記述がありましたので、念のため明記しておきます。

日付・時刻をあつかうときに `OffsetDateTime` を使っていると、開発者はタイムゾーンの呪いをかなり遠ざけることができます。特にクラスのフィールド変数や、メソッドの引数・戻り値など他のコンポーネントとの受け渡しに `OffsetDateTime` を使っておくと、存在しない時刻や一意に特定できないあいまいな時刻を保持したり渡したりしてしまうような事故を、静的に防ぐことができます。

同様の理由から、外部に日付・時刻データを保存するのに `OffsetDateTime` 相当のデータを使っておくのも有効です。

「規約として常に UTC を使うことにしておけば `LocalDateTime` でいい」という意見もあるかもしれません。しかし特にチーム開発において、個々の開発者の努力のみで、ほんとうにそのような規約を有効に維持し続けられるでしょうか。まかせられるところは言語がかけてくれる制約にまかせて、開発者は本来やりたかったことに注力したいものです。

そこまでして `LocalDateTime` を持ち回ることには、たいしたメリットも見いだせません。しいていえばメモリの使用量は少し変わるかもしれませんが、常に UTC なら定数 `ZoneOffset.UTC` を使っておけば、新しい `ZoneOffset` インスタンスが作られることもありません。差分は微々たるものだと思っていいでしょう。

### ZonedDateTime

`ZonedDateTime` は、任意の `ZoneId` をタイムゾーン情報として持てる日付・時刻です。情報量的には単なる `OffsetDateTime` の上位互換のように見えますが、任意のタイムゾーンを持ててしまうことによって、解けないタイムゾーンの呪いとつきあい続けることを宿命づけられた、悲劇のクラスです。

`ZonedDateTime` では、たとえば `America/Los_Angeles` の 2020年 3月 8日 2時 30分、という時刻を作れてしまいます。これはロサンゼルスが太平洋標準時から太平洋夏時間に切り替わり、すっ飛ばされて存在しないはずの時刻です。同様に `America/Los_Angeles` の 2020年 11月 1日 1時 30分という時刻も作れます。こちらは太平洋夏時間から太平洋標準時に切り替わり、太平洋夏時間の 11月 1日 1時 30分と太平洋標準時の 11月 1日 1時 30分の両方が存在する時刻です。

このようなあいまいさを少しでも避けるために `ZonedDateTime` はオプションとして追加で `ZoneOffset` を持つこともできます。しかしこれはあくまでオプションであり、型情報としてはあらわれません。

実際このような時刻を作ろうとしてみるとどうなるか、試してみましょう。

```java
import java.time.DateTimeException;
import java.time.LocalDateTime;
import java.time.ZoneId;
import java.time.ZoneOffset;
import java.time.ZonedDateTime;

public class UseOfZonedDateTime {
    public static void main(final String[] args) {
        // 標準時 → 夏時間
        printNaive( 2020,  3, 8, 2, 30, 0, 0, ZoneId.of("America/Los_Angeles"));
        printStrict(2020,  3, 8, 2, 30, 0, 0, ZoneOffset.ofHours(-8), ZoneId.of("America/Los_Angeles"));
        printStrict(2020,  3, 8, 2, 30, 0, 0, ZoneOffset.ofHours(-7), ZoneId.of("America/Los_Angeles"));

        // 標準時 → 夏時間の一時間後
        printNaive( 2020,  3, 8, 3, 30, 0, 0, ZoneId.of("America/Los_Angeles"));
        printStrict(2020,  3, 8, 3, 30, 0, 0, ZoneOffset.ofHours(-8), ZoneId.of("America/Los_Angeles"));
        printStrict(2020,  3, 8, 3, 30, 0, 0, ZoneOffset.ofHours(-7), ZoneId.of("America/Los_Angeles"));

        // 夏時間 → 標準時
        printNaive( 2020, 11, 1, 1, 30, 0, 0, ZoneId.of("America/Los_Angeles"));
        printStrict(2020, 11, 1, 1, 30, 0, 0, ZoneOffset.ofHours(-8), ZoneId.of("America/Los_Angeles"));
        printStrict(2020, 11, 1, 1, 30, 0, 0, ZoneOffset.ofHours(-7), ZoneId.of("America/Los_Angeles"));

        // 夏時間 → 標準時の一時間後
        printNaive( 2020, 11, 1, 2, 30, 0, 0, ZoneId.of("America/Los_Angeles"));
        printStrict(2020, 11, 1, 2, 30, 0, 0, ZoneOffset.ofHours(-8), ZoneId.of("America/Los_Angeles"));
        printStrict(2020, 11, 1, 2, 30, 0, 0, ZoneOffset.ofHours(-7), ZoneId.of("America/Los_Angeles"));
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
}
```

これを実行するとこうなります。

```
ZonedDateTime.of(2020, 3, 8, 2, 30, 0, 0, "America/Los_Angeles"):
  .toString(): 2020-03-08T03:30-07:00[America/Los_Angeles]
  .toInstant().toString(): 2020-03-08T10:30:00Z

ZonedDateTime.of(2020, 3, 8, 2, 30, 0, 0, "-08:00", "America/Los_Angeles"):
  DateTimeException: LocalDateTime '2020-03-08T02:30' does not exist in zone 'America/Los_Angeles' due to a gap in the local time-line, typically caused by daylight savings

ZonedDateTime.of(2020, 3, 8, 2, 30, 0, 0, "-07:00", "America/Los_Angeles"):
  DateTimeException: LocalDateTime '2020-03-08T02:30' does not exist in zone 'America/Los_Angeles' due to a gap in the local time-line, typically caused by daylight savings

ZonedDateTime.of(2020, 3, 8, 3, 30, 0, 0, "America/Los_Angeles"):
  .toString(): 2020-03-08T03:30-07:00[America/Los_Angeles]
  .toInstant().toString(): 2020-03-08T10:30:00Z

ZonedDateTime.of(2020, 3, 8, 3, 30, 0, 0, "-08:00", "America/Los_Angeles"):
  DateTimeException: ZoneOffset '-08:00' is not valid for LocalDateTime '2020-03-08T03:30' in zone 'America/Los_Angeles'

ZonedDateTime.of(2020, 3, 8, 3, 30, 0, 0, "-07:00", "America/Los_Angeles"):
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

ZonedDateTime.of(2020, 11, 1, 2, 30, 0, 0, "America/Los_Angeles"):
  .toString(): 2020-11-01T02:30-08:00[America/Los_Angeles]
  .toInstant().toString(): 2020-11-01T10:30:00Z

ZonedDateTime.of(2020, 11, 1, 2, 30, 0, 0, "-08:00", "America/Los_Angeles"):
  .toString(): 2020-11-01T02:30-08:00[America/Los_Angeles]
  .toInstant().toString(): 2020-11-01T10:30:00Z

ZonedDateTime.of(2020, 11, 1, 2, 30, 0, 0, "-07:00", "America/Los_Angeles"):
  DateTimeException: ZoneOffset '-07:00' is not valid for LocalDateTime '2020-11-01T02:30' in zone 'America/Los_Angeles'
```

オフセットのない `2020-03-08 02:30:00 [America/Los_Angeles]` を作ろうとした結果、存在しないので勝手に `2020-03-08 03:30:00 -07:00 [America/Los_Angeles]` にされてしまっていることがわかりますね。ほか、ためしに明示的に `-07:00` や `-08:00` のオフセットを指定して `02:30:00` を作ろうとしても、そのような時刻は存在しない、と怒られています。

逆に二重に存在する `2020-11-01 01:30:00 [America/Los_Angeles]` を作ろうとした結果は、勝手に `-07:00` に寄せられて `2020-11-01 01:30:00 -07:00 [America/Los_Angeles]` にされてしまっていますね。

`ZonedDateTime.of()` のこのあたりの挙動は [Javadoc](https://docs.oracle.com/javase/jp/8/docs/api/java/time/ZonedDateTime.html#of-int-int-int-int-int-int-int-java.time.ZoneId-) で以下のように説明されています。

> ほとんどの場合に、ローカル日付/時間の有効なオフセットは1つだけです。重複の場合は、クロックが後方向に戻されたときに、2つの有効なオフセットが存在します。このメソッドは、一般に「夏」に対応する早い方のオフセットを使用します。
>
> ギャップの場合は、クロックが前方向にジャンプしたときに、有効なオフセットが存在しません。代わりに、ローカル日付/時間がギャップの長さだけ後ろに調整されます。一般的な1時間の夏時間の変更では、ローカル日付/時間が、一般に「夏」に対応するオフセットの中の1時間後方に移動されます。

「夏」に対応する早い方のオフセットを使用します、とか、後ろに調整されます、とか、逆がほしいときもあるはずなのに勝手に決めてくれちゃってるなあ、というようにも見えますが、同 Javadoc によれば、このメソッドは主にテスト・ケースの作成用として用意されているのだそうです。

> このメソッドは、主としてテスト・ケースを作成するために存在しています。テスト・コード以外では通常、別のメソッドを使ってオフセット時間を作成します。

`ZonedDateTime` を実用するときは、このようなケースでどのようなやり方を選ぶべきなのか、要件とにらめっこして検討し、処理を明示的に記述しておきましょう。エラーにする、無視して別の日にする、できるだけ切り替わり前のオフセットに寄せる、できるだけ切り替わり後のオフセットに寄せる、できるだけ「標準時」に寄せる、できるだけ「夏時間」に寄せる、などなど、このようなケースにどうすべきかは、要件次第で千差万別です。タイムゾーンの呪いとは、まさにこのような例外ケースのそれぞれについて、いちいち検討して仕様を詰めておかなければならない、ということなのです。

極力 `ZonedDateTime` を使いたくない気持ちになりますが、それでも `ZonedDateTime` を使うべきケースはあります。「実装編」でも触れましたが、その代表的な例が「暦の計算、タイムゾーンの切り替わりをまたぐ計算」をするケースや、「未来に予定された時刻」をあつかうケースです。

「翌日」のような暦の計算をしなければならないケースでは `ZonedDateTime` が必要なことがあります。まずは「翌日」の要件定義を明らかにしなければなりませんが、要件が「次の日付の同時刻」であった場合、それをタイムゾーンの切り替わりをまたいで計算するには `ZonedDateTime` が必須でしょう。

また、未来に予定された時刻をあつかうケースでは、その予定時刻が「なに時間」での定義なのかがとても重要です。起こりうる問題を短くまとめると、「現地時刻で定義された未来の時刻を、古い tzdb を用いて一度 UTC に変換し、それをあとから新しい tzdb を用いて現地時刻に戻そうとすると、本来の予定時刻とずれることがある」ということです。

たとえば[延期前の東京 2020 オリンピック開会式の開始予定時刻は 2020年 7月 24日 20時](https://sports.nhk.or.jp/olympic/schedules/sports/ceremony.html)でしたが、これは日本時間での定義です。仮にこの開始時刻が決まったあとで夏時間の導入が決定していたら、日本時間上の開始時刻は 20時で変わらず、絶対時刻への解釈の方が変わる、ということがありえたかもしれません。これと同様のことは、夏時間の廃止が決定している EU でも、これから起こる可能性があります。 [^jon-skeet]

[^jon-skeet]: EU の例については、「実装編」でも参照しましたが ["STORING UTC IS NOT A SILVER BULLET" (Mar 27, 2019)](https://codeblog.jonskeet.uk/2019/03/27/storing-utc-is-not-a-silver-bullet/) で詳細に議論されています。

このようなケースでは、呪いと向き合いながら `ZonedDateTime` を使う覚悟を決めましょう。つらい戦いですが、これは避けられない戦いです。逆に、必要もないのに `ZonedDateTime` を使うと無駄に呪われてしまうわけです。 `ZonedDateTime` を本当に使うべきケースを、要件から見極めましょう。

日付のみと時刻のみ
-------------------

JSR 310 で「日付のみ」をあつかうクラスは [`java.time.LodalDate`](https://docs.oracle.com/javase/jp/8/docs/api/java/time/LocalDate.html) のみで、「時刻のみ」をあつかうクラスは [`java.time.LocalTime`](https://docs.oracle.com/javase/jp/8/docs/api/java/time/LocalTime.html) と [`java.time.OffsetTime`](https://docs.oracle.com/javase/jp/8/docs/api/java/time/OffsetTime.html) のみです。 `OffsetDate` や `ZonedDate` のような「タイムゾーン情報と日付のみ」の組み合わせ、または `ZonedTime` のような「地域ベースのタイムゾーンと時刻のみ」の組み合わせはありません。

`ZonedDate` と `ZonedTime` がないのは、日付と時刻の両方がそろわないと地域ベースのタイムゾーンはあまり意味をなさないからでしょう。たとえば「ロサンゼルス時間の 2020年 3月 8日」や「ニューヨーク時間の午前 1時 30分」のようなデータだけがあっても、「その日付のどの時刻のことか」または「その時刻がどの日付の時刻か」がわからないと、オフセットの解釈が確定できません。 `ZonedDateTime` のように暦の計算に使えるわけでもなく、標準 API に用意しても存在意義がよくわからないクラスになりそうです。

そして `OffsetTime` はありますが `OffsetDate` はありません。「`+09:00` で午後 8時」は、どの日付においても不確定要素のないデータで、それなりに使いようもありそうです。それに対して「`+09:00` の 2021年 7月 23日」は、不確定要素こそないものの、使いどころはあまりなさそうです。 [^offset-date] `+09:00` がほんとうに重要なケースなら「2021年 7月 23日 0時 0分 0秒から 23時 59分 59秒」と表現するほうが合理的な気がします。

[^offset-date]: 実は後述の ThreeTen-Extra には [`OffsetDate`](https://www.threeten.org/threeten-extra/apidocs/org.threeten.extra/org/threeten/extra/OffsetDate.html) があります。当初は JSR 310 に入れるつもりで設計していたものの、[あまり使いどころがないという判断で消した](https://github.com/ThreeTen/threeten/issues/228)みたいですね。

ThreeTen-Extra: うるう秒
=========================

Java タイム・スケールがうるう秒を希釈することを前提としていることもあって、残念ながら JSR 310 の範疇ではうるう秒をそのままあつかうことができません。どうしてもうるう秒をそのままあつかう必要がある場合は [ThreeTen-Extra](https://www.threeten.org/threeten-extra/) という外部ライブラリを JSR 310 と組み合わせて使うことができます。

JSR 310 は異なるタイム・スケールを独自に実装して拡張できるように設計されていて、この ThreeTen-Extra は、そのような拡張の一つです。もともと JSR 310 の一部として検討されていたクラス群ですが、その JSR 310 があまりに肥大化したために、整理して ThreeTen-Extra という外部ライブラリとして切り出されました。たとえば [`org.threeten.extra.scale.UtcInstant`](https://www.threeten.org/threeten-extra/apidocs/org.threeten.extra/org/threeten/extra/scale/UtcInstant.html) は、うるう秒を考慮した `Instant` のようなクラスになっています。

`java.util.Date` と `java.util.Calendar`
=====================================

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

不幸にもまだ Java 7 以前しか使えない場合でも [ThreeTen Backport](http://www.threeten.org/threetenbp/) という JSR 310 の多くの機能を Java 7 以前にバックポートしたライブラリがあります。せめてそっち使いましょう。

2018 年において Java 8 以降への移行すら考えていない、ということはさすがにもう無いと思うので、今から新しく Joda-Time を採用する理由はあまり無いでしょう。 Joda-Time を Java 8 以降で使うことは基本的に推奨されていません。

おまけ: Java とタイムゾーン略称
================================

このおまけは、上の方で少し触れた Java の `MST` などのタイムゾーン略称に関する深堀りです。

たとえば `MST` などが `America/Denver` と等価に解釈されてしまい、その結果 `2020-07-01 12:34:56 MST` が `2020-07-01 12:34:56 -07:00` と等価に解釈されるべきところを `2020-07-01 12:34:56 -06:00` と解釈されてしまう、という話でした。

旧 `java.util.TimeZone`
----------------------

この話、実は Java ではもっと昔に解決されていたはずだったのです。というのは JSR 310: Date and Time API が導入される Java 8 より前、既に Java 1.5 の時代にはこの話は認知されて Java 6 で対処されていました。 Oracle Community の投稿 ["EST, MST, and HST time zones in Java 6 and Java 7"](https://community.oracle.com/tech/developers/discussion/2539847/est-mst-and-hst-time-zones-in-java-6-and-java-7) を見ると、なんとなく経緯がわかります。

Java に昔からある [`java.util.TimeZone`](https://docs.oracle.com/javase/jp/8/docs/api/java/util/TimeZone.html) では、最初期から `MST` などの省略形を `America/Denver` などのタイムゾーン ID にマッピングしてしまっていました。しかしあまりにも昔からそうなっていたため、互換性を考えると、安易には変えられなかったようです。 [^abbreviations-in-java-1-3] [^abbreviations-in-java-1-1-6]

[^abbreviations-in-java-1-3]: ちなみに省略タイムゾーンの利用そのものは [Java 1.3 の頃には既に deprecated だと宣言されています](https://javaalmanac.io/jdk/1.3/api/java/util/TimeZone.html)。 deprecate されてこんなに長い時間が経ったのにまだ振り回されるというのも悲しい話ですね。

[^abbreviations-in-java-1-1-6]: ちなみに [(なぜか MIT の Web に置かれている) Java 1.1.6 の頃の `java.util.TimeZone` の実装](http://web.mit.edu/java_v1.1.6/distrib/sun4x_57/src/java/util/TimeZone.java)を見ると、いくつかのタイムゾーン略称から地域ベースのタイムゾーン ID へのマッピングがハードコードされているのがわかります。このうちいくつかは明らかに一般的とは言えないマッピングなんですが (なんで [`AST`](https://www.timeanddate.com/time/zones/ast) が "Alaska Standard Time" で `America/Anchorage` やねん) そこについて掘り下げるのはまた別の機会に…。一度埋めてしまった地雷はなかなか除去できないという典型例ですね。

しかし Java 6 で重い腰を上げ、夏時間を実施しない州を含む `EST`, `MST`, `HST` だけは、それぞれ固定オフセットの `-05:00`, `-07:00`, `-10:00` にマッピングするようになりました。同時に、互換性のために `sun.timezone.ids.oldmapping` という Java システム・プロパティが用意され、これを `"true"` にセットしておくと旧来のマッピングを使用するようになります。以下のコードでこのことを確認できます。

```
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

JSR 310 の `DateTimeFormatter`
-----------------------------

`EST`, `MST`, `HST` が直って、めでたしめでたし… (?) と思っていたところに Java 8 で JSR 310 が実装されました。

最初に参照したとおり、この JSR 310 は日付・時刻・タイムゾーンの「現実」を忠実にモデル化していて、かなりよくできていると筆者は考えています。タイムゾーン略称についても、実は本丸の `java.time.ZoneId` では略称の使用をかなり制限していて、[名前から `ZoneId` を作るときに `aliasMap` を明示的に与えないと略称は使えない](https://docs.oracle.com/javase/jp/8/docs/api/java/time/ZoneId.html#of-java.lang.String-java.util.Map-)ようになっています。さらに、[互換性のための標準 Map](https://docs.oracle.com/javase/jp/8/docs/api/java/time/ZoneId.html#SHORT_IDS)も用意されていて、ここでは `EST`, `MST`, `HST` はちゃんと固定オフセットにマッピングされています。

では、なぜ前述した例では `2020/07/01 12:34:56 MST` が `2020-07-01T12:34:56-06:00[America/Denver]` になってしまったのでしょうか?

その答えは [`java.util.format.DateTimeFormatter`](https://docs.oracle.com/javase/jp/8/docs/api/java/time/format/DateTimeFormatter.html) の実装でした。

`DateTimeFormatter` の最も手軽な使い方である [`#ofPattern(String)`](https://docs.oracle.com/javase/jp/8/docs/api/java/time/format/DateTimeFormatter.html#ofPattern-java.lang.String-) で `time-zone name` を表す `z` や `zzzz` を使用すると、それは [`DateTimeFormatterBuilder#appendZoneText(TextStyle)`](https://docs.oracle.com/javase/jp/8/docs/api/java/time/format/DateTimeFormatterBuilder.html#appendZoneText-java.time.format.TextStyle-) の呼び出しに相当します。この `appendZoneText` はかなり幅広いタイムゾーン名表現を受け付けるようになっているようです。 Javadoc にも以下のように注意書きがあります。

> 解析時には、テキストでのゾーン名、ゾーンID、またはオフセットが受け入れられます。テキストでのゾーン名には、一意でないものが多くあります。たとえば、CSTは「Central Standard Time (中部標準時)」と「China Standard Time (中国標準時)」の両方に使用されます。この状況では、フォーマッタのlocaleから得られる地域情報と、その地域の標準ゾーンID (たとえば、America Easternゾーンの場合はAmerica/New_York)により、ゾーンIDが決定されます。appendZoneText(TextStyle, Set)を使用すると、この状況で優先するZoneIdのセットを指定できます。

だからと言って `MST` は `America/Denver` にしないで `-07:00` にしてくれれば…。

`DateTimeFormatter` に深入り
---------------------------

そんなわけで、少しだけ OpenJDK のコードに深入りしてみました。ひとまず [`DateTimeFormatterBuilder#appendZoneText` がタイムゾーン名の候補を引っ張ってきているのはこの行](https://github.com/openjdk/jdk/blob/jdk8-b120/jdk/src/share/classes/java/time/format/DateTimeFormatterBuilder.java#L3708)のようです。

```
                zoneStrings = TimeZoneNameUtility.getZoneStrings(locale);
```

[`sun.util.locale.provider.TimeZoneNameUtility`](https://github.com/openjdk/jdk/blob/jdk8-b120/jdk/src/share/classes/sun/util/locale/provider/TimeZoneNameUtility.java) という内部クラスから Locale をベースにしてタイムゾーン名の候補を出しているみたいですね。ここまで来たらもう少しだけ追ってみましょう、以下のコードでこの候補を無理やり読んでみます。

```
import java.lang.reflect.Method;
import java.util.Arrays;
import java.util.Locale;

public final class TimeZoneNames throws Exception {
    public static void main(final String[] args) throws Exception {
        for (final String[] e : getZoneStrings()) {
            System.out.println(String.join(", ", Arrays.asList(e)));
        }
    }

    private static String[][] getZoneStrings() throws Exception {
        final Class<?> clazz = Class.forName("sun.util.locale.provider.TimeZoneNameUtility");
        final Method method = clazz.getMethod("getZoneStrings", Locale.class);
        final Object zoneStringsObject = method.invoke(null, Locale.ENGLISH);
        return (String[][]) zoneStringsObject;
    }
}
```

実行してみると…。

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

あー、いますね。 `America/Denver` と `MST` が紐付いているような雰囲気がぷんぷんします。この結果からすると、犯人はハードコードされているのではなく Locale データの中にいるのかもしれません。

筆者はここで追うのをやめましたが、興味のある方はぜひ追いかけてみてください。 [^tell-me]

[^tell-me]: そしてぜひ筆者に教えてください。

`DateTimeFormatter` 対策
-----------------------

さて、こういう混乱なく `DateTimeFormatter` を使うにはどうしたらいいでしょうか。以下は完全に筆者の私見ですが、少し検討してみます。

まず Locale に踏み込むのはかなり大変そうです。

そもそも `DateTimeFormatterBuilder#appendZoneText` が自由すぎる・寛容すぎるので、この問題だけがどうにかなっても、さらに予想外の地雷を踏む可能性が高そうだなあ、という気がします。

`DateTimeFormatterBuilder` でタイムゾーン情報の処理を追加するメソッドは `appendZoneText` 以外にもいくつかあります。そこで、用途に合うものから厳密に処理してくれるものを探して使う、というのが、一つよさそうな方針ではないでしょうか。

たとえば、固定オフセットのみを処理する [`appendOffset`](https://docs.oracle.com/javase/jp/8/docs/api/java/time/format/DateTimeFormatterBuilder.html#appendOffset-java.lang.String-java.lang.String-) や、タイムゾーン ID そのものの文字列のみを処理する [`appendZoneId`](https://docs.oracle.com/javase/jp/8/docs/api/java/time/format/DateTimeFormatterBuilder.html#appendZoneId--) などです。パターン文字列を使う場合は `XXX`, `ZZZ`, `VV` などに当たります。いずれの場合も、用途に合った適切なものを考えて選ぶのがいいでしょう。

パターン文字列の詳細な解説は [`DateTimeFormatterBuilder#appendPattern` の Javadoc](https://docs.oracle.com/javase/jp/8/docs/api/java/time/format/DateTimeFormatterBuilder.html#appendPattern-java.lang.String-) にあります。

どうなる `PST`
-------------

ところで、最初の方にも書きましたが、カリフォルニア州では 2018年の住民投票で夏時間の変更が支持されたそうです。

もしこれが実施されたら、今まで `EST`, `MST`, `HST` だけだった特例に、今度は `PST` も追加する必要が出てくるかもしれません。はたして、これからなにが起こるでしょうか。

タイムゾーンの呪いは、そう簡単に解けることはなさそうです。
