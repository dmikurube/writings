---
title: "タイムゾーン呪いの書 (Java 編)"
emoji: "🌎" # アイキャッチとして使われる絵文字（1文字だけ）
type: "tech" # tech: 技術記事 / idea: アイデア
topics: [ "timezone", "jsr310", "java" ]
layout: default
published: false
---

Java #0: JRE と tz database
----------------------------

tz database の説明で「tz database は年に数回は新しい版がリリースされます」と紹介しました。 Java の場合は Java Runtime Environment (JRE) にコンパイル済みの tz database が付属する形になっています。 JRE をアップデートすると、新しいバージョンの tz database もくっついてきて更新されます。 [^24]

[^24]: タイミングによってはその JRE バージョンのリリース時点で tz database の対応が間に合わず、最新よりちょっとだけ古い tz database がやってくる可能性はあります。

JRE 自体はアップデートしたくないけど tz database だけアップデートしたい、という場合は [Timezone Updator Tool (TZUpdater)](http://www.oracle.com/technetwork/jp/java/javase/tzupdater-readme-136440.html) を使うことができます。[^25]

[^25]: 数年前に Oracle の方針云々で「TZUdater 提供されなくなる!」と混乱が起きたことがあります。2018年1月現在、サポートが打ち切られていないバージョンの JRE であれば TZUpdater は提供されています。

tz database が Java 実行環境 (JRE) に付属するものであって Java アプリケーションに付属するものではない、という点には注意が必要です。つまり、あるアプリケーションをホスト A で動かしたときとホスト B で動かしたときで tz database のバージョンがずれてしまう可能性がある、ということです。

後述する地域ベースタイムゾーン (`ZoneId`) やそれを利用した日付時刻 (`ZonedDateTime`) を使う場合は、アプリケーションの更新・管理とは別に、戦略的に tz database のバージョン管理も行う必要があります。

前述のサモア標準時のように大規模な変更が直前に行われる可能性を考えると、ただ最新を追いかけるというだけでもそんなに簡単ではありません。

Java #1: [JSR 310: Date and Time API](https://jcp.org/en/jsr/detail?id=310)
----------------------------------------------------------------------------

JSR 310 は (既に何度か参照していますが) 日付・時刻を扱う新しい Java API です。 Java 8 から追加されました。

Stephen Colebourne 氏によって実装された [Joda-Time](http://www.joda.org/joda-time/) というライブラリが JSR 310 以前からあったのですが、これをベースとして、同じ Stephen 氏のリードで正式な Java API として再設計されたのが JSR 310 です。後述する、古くからの API である `java.util.Date` と `java.util.Calendar` を完全に置き換えることを目指した[^26] らしいです。

[^26]: 要出典。

JSR 310 は「ただ日付・時刻を扱うだけなのに複雑すぎる!」という声もよく耳にします。が、日付・時刻というものが、上で書いてきたようにそもそも超複雑なんです。 JSR 310 のオブジェクトモデルは、この「超複雑な日付・時刻というモノ」をできるだけ忠実にモデル化しようとしており、とてもよくできているという認識を筆者は持っています。確かに複雑だけどそもそも超複雑なんだから仕方ないじゃん! という感じ。[^27]

[^27]: Parser による解釈が厳密すぎて大変、というのはわかる。

### [`java.time.ZoneId`](https://docs.oracle.com/javase/jp/8/docs/api/java/time/ZoneId.html), [`java.time.ZoneOffset`](https://docs.oracle.com/javase/jp/8/docs/api/java/time/ZoneOffset.html)

JSR 310 のタイムゾーンは、すべてのタイムゾーンを表す [`java.time.ZoneId`](https://docs.oracle.com/javase/jp/8/docs/api/java/time/ZoneId.html) 抽象クラスと、その中でも固定オフセットを表すサブクラスの [`java.time.ZoneOffset`](https://docs.oracle.com/javase/jp/8/docs/api/java/time/ZoneOffset.html) という2つのクラスで実装されています。文字列表現からは、それぞれ `ZoneId.of("Asia/Tokyo")` や `ZoneOffset.of("+09:00")` などと呼び出してインスタンスを作成します。

地域ベースの `ZoneId` インスタンスには tz database のタイムゾーンが対応していて、「いつ夏時間に切り替わるか」「過去のどの時点から使うオフセットが変わったか」などの遷移ルールも実装されています。 `ZoneId` インスタンスの `ZoneId#getRules()` を呼び出すことで、ルールを実装した [`java.time.ZoneRules`](https://docs.oracle.com/javase/jp/8/docs/api/java/time/zone/ZoneRules.html) を取得することができます。

`ZoneId` は抽象クラスなのでそのもののインスタンスは作れません。地域ベースタイムゾーンは、今のところ `ZoneId` のサブクラスの [`java.time.ZoneRegion`](http://hg.openjdk.java.net/jdk8/jdk8/jdk/file/jdk8-b132/src/share/classes/java/time/ZoneRegion.java#l90) として実装されているようです。 (`ZoneRegion` は非公開なので直接は使えませんし、今後もこの実装が保証されるわけではありません。使うときはあくまで `ZoneId` として使います)

`"Asia/Tokyo"` `"UTC+09:00"`, `"+09:00"` のそれぞれに対して `ZoneId.of()` と `ZoneOffset.of()` を呼び出したインスタンスは、以下のように動作しています。

```java
import java.time.DateTimeException;
import java.time.ZoneId;
import java.time.ZoneOffset;

public class ZoneIds {
    public static void main(final String[] args) {
        investigateZoneId("Asia/Tokyo");
        investigateZoneId("UTC+09:00");
        investigateZoneId("+09:00");
        investigateZoneOffset("Asia/Tokyo");
        investigateZoneOffset("UTC+09:00");
        investigateZoneOffset("+09:00");
    }

    private static void investigateZoneId(String id) {
        System.out.printf("ZoneId:[%s]\n", id);
        investigate(ZoneId.of(id));
    }

    private static void investigateZoneOffset(String id) {
        System.out.printf("ZoneOffset:[%s]\n", id);
        try {
            investigate(ZoneOffset.of(id));
        } catch (DateTimeException ex) {
            System.out.printf("  DateTimeException: %s\n\n", ex.getMessage());
        }
    }

    private static void investigate(ZoneId zoneId) {
        System.out.printf("  .toString(): [%s]\n", zoneId);
        System.out.printf("  .getClass(): [%s]\n", zoneId.getClass());
        System.out.printf("  .normalized().toString(): [%s]\n", zoneId.normalized());
        System.out.printf("  .normalized().getClass(): [%s]\n", zoneId.normalized().getClass());
        System.out.printf("  .getRules().isFixedOffset(): [%s]\n", zoneId.getRules().isFixedOffset());
        System.out.printf("\n");
    }
}
```

```
ZoneId:[Asia/Tokyo]
  .toString(): [Asia/Tokyo]
  .getClass(): [class java.time.ZoneRegion]
  .normalized().toString(): [Asia/Tokyo]
  .normalized().getClass(): [class java.time.ZoneRegion]
  .getRules().isFixedOffset(): [false]

ZoneId:[UTC+09:00]
  .toString(): [UTC+09:00]
  .getClass(): [class java.time.ZoneRegion]
  .normalized().toString(): [+09:00]
  .normalized().getClass(): [class java.time.ZoneOffset]
  .getRules().isFixedOffset(): [true]

ZoneId:[+09:00]
  .toString(): [+09:00]
  .getClass(): [class java.time.ZoneOffset]
  .normalized().toString(): [+09:00]
  .normalized().getClass(): [class java.time.ZoneOffset]
  .getRules().isFixedOffset(): [true]

ZoneOffset:[Asia/Tokyo]
  DateTimeException: Invalid ID for ZoneOffset, invalid format: Asia/Tokyo

ZoneOffset:[UTC+09:00]
  DateTimeException: Invalid ID for ZoneOffset, non numeric characters found: UTC+09:00

ZoneOffset:[+09:00]
  .toString(): [+09:00]
  .getClass(): [class java.time.ZoneOffset]
  .normalized().toString(): [+09:00]
  .normalized().getClass(): [class java.time.ZoneOffset]
  .getRules().isFixedOffset(): [true]
```

`ZoneId#normalize()` は `ZoneOffset` に正規化できるインスタンスであれば正規化した `ZoneOffset` を返す、というメソッドです。 `ZoneId.of("UTC+09:00")` で作られたインスタンスは `ZoneRegion` ですが、それを `normalize()` してできたインスタンスは `ZoneOffset` になっていることがわかります。

`ZoneRules#isFixedOffset()` は、そのタイムゾーンが遷移のない固定されたタイムゾーンか否かを返すメソッドです。夏時間を採用していないはずの `"Asia/Tokyo"` が `false` なのはなんでだ! と思われるかもしれませんが、理由の一つは前述の通り、過去の一時期に夏時間を採用したことがあるためですね。 (ちなみにその夏時間以外にも遷移がありました)

固定オフセットが、地域ベースのタイムゾーンとは別のクラスで実装されている、というのが JSR 310 のキモです。これについては `java.time.OffsetDateTime` と `java.time.ZonedDateTime` に触れる際に後述します。

### [`java.time.Instant`](https://docs.oracle.com/javase/jp/8/docs/api/java/time/Instant.html)

時間軸上の特定の一点を表すのが [`java.time.Instant`](https://docs.oracle.com/javase/jp/8/docs/api/java/time/Instant.html) です。前述の UNIX 時間 (秒) と、小数部としてナノ秒を用いています。

うるう秒を無視することによる問題が無さそうであれば[^A1] 特に内部表現としては `java.time.Instant` を使うことをまず検討するべきでしょう。曖昧さがなく、メモリの消費も小さく、前後の比較も簡単で、例えばイベントが起こった時刻の記録としては必要十分です。

[^A1]: ほとんどの場合では、問題になることは無いと思います。

次項の日付時刻クラス群は、「一週間先」「一ヶ月先」のような日付時刻の計算を行うとき、ユーザーが関係する入出力、外部データの入出力、などのタイミングで使うことが多いでしょう。

### [`LocalDateTime`](https://docs.oracle.com/javase/jp/8/docs/api/java/time/LocalDateTime.html), [`OffsetDateTime`](https://docs.oracle.com/javase/jp/8/docs/api/java/time/OffsetDateTime.html), [`ZonedDateTime`](https://docs.oracle.com/javase/jp/8/docs/api/java/time/ZonedDateTime.html)

どれも日付と時刻の組を表すクラス群ですが、用途に応じて3種類あります。筆者は、これらの使い分けは「時間軸上の一点 (前述の `Instant`) に対応できないことがあってもいいか否か」「その日付時刻から夏時間の境をまたぐ計算をするか否か」「その日付時刻表現について地理的地域は重要か否か」を基準に判断するのがいいと考えています。

#### [`java.time.LocalDateTime`]((https://docs.oracle.com/javase/jp/8/docs/api/java/time/LocalDateTime.html))

`LocalDateTime` は、タイムゾーン情報を一切含まない日付時刻です。このため `LocalDateTime` だけでは `Instant` には変換できず、「時間軸上のどの一点を表すのか」は `LocalDateTime` だけからはわからない、ということになります。

JSR 310 に関する日本語記事を探すと「とりあえず `LocalDateTime` 使っとけばいいよ」という記述をたまに見かけますが、あえて `LocalDateTime` を使うべき状況を筆者はほとんど思いつきません。何か起きた時間のログとして使うには「どの時点だったか」を確定できないと不十分ですし、ログでは無くても `LocalDateTime` を持ち回れば「この `LocalDateTime` ってどこの時間だったっけ」とチームに混乱を引き起こす原因になります。

同じタイムゾーンの大量の日付時刻を計算するときに、節約のために `LocalDateTime` を使うケースはあるかもしれません。その場合は、なるべくその計算のためだけの狭い範囲に限定して `LocalDateTime` が漏れないようにする、特にそのまま外部には保存しない、などの工夫をしながら使うことをお勧めします。

それ以外には、あえてタイムゾーン情報と関連付けたくない場合、例えば「そのホストがどこにあるかによらずそのホストの時刻の 23:00:00 に特定の処理を実行する」のような場合以外に `LocalDateTime` をあえて使うべきケースは、ほとんど無いように思います。[^28]

[^28]: 「こんな時に `LocalDateTime` 使うとよかった!」などの反例をお待ちしています。

#### [`java.time.OffsetDateTime`]((https://docs.oracle.com/javase/jp/8/docs/api/java/time/OffsetDateTime.html))

`OffsetDateTime` はタイムゾーン情報を持つものの、固定オフセットの `java.time.ZoneOffset` のみを許す日付時刻です。「なんでわざわざ `ZoneOffset` のみに限定するのか?」「`ZoneId` や後述の `ZonedDateTime` でいいじゃないか?」という意見があるかもしれません。が、前述の通り `ZoneOffset` というクラスを用意して限定できるようにしたことが JSR 310 のキモです。

`ZoneOffset` のみに制限することで `OffsetDateTime` は常に一意な `Instant` に変換できることを保証できるのです。[^29] 後述するように `ZonedDateTime` ではそうはいきません。

[^29]: 変換は「一対一」ではなく、うるう秒の場合に `Instant` への変換で情報が失われてしまうことには、若干の注意が必要です。

`OffsetDateTime` で済む場合はできるだけ `OffsetDateTime` を使っておくと、タイムゾーンの呪いをかなり遠ざけることができます。特に、他のコンポーネントとのインターフェースに使う場合や、外部に保存する場合 (ログなど) に `OffsetDateTime` (または相当するデータ) を使っておくと、曖昧なデータになってしまったりチームメンバーを混乱させたりする危険をだいぶ減らすことができます。

「常に UTC を使う規約にしておけば `LocalDateTime` でもいい!」という意見はあるかもしれません。しかし、往々にしてその規約を忘れたコードが生まれるものです。常に UTC を使う場合でも `ZoneOffset.UTC` を持った `OffsetDateTime` を作るようにしておくと、多少のメモリは使いますが混乱を減らせます。例えばこのような現在時刻を取得するには `OffsetDateTime.now(ZoneOffset.UTC)` などと呼び出せばいいでしょう。

`ZoneOffset` は[値ベースのクラス](https://docs.oracle.com/javase/jp/8/docs/api/java/lang/doc-files/ValueBased.html)ですが `ZoneOffset.UTC` は定数なので、大量の `ZoneOffset` インスタンスが作られるわけではなく、メモリへの影響もあまり大きくないと思われます。 [^30]

[^30]: 要検証。

#### [`java.time.ZonedDateTime`]((https://docs.oracle.com/javase/jp/8/docs/api/java/time/ZonedDateTime.html))

`ZonedDateTime` は任意の `java.time.ZoneId` をタイムゾーン情報として持つ日付時刻です。「`OffsetDateTime` より情報量多そうだし `ZonedDateTime` 使っとけばいいだろ!」という記述もたまに見かけますが、前述したように落とし穴があります。

例えば `"America/Los_Angeles"` で2017年3月12日 午前2時30分、という時刻が与えられてしまったとします。が、そんな時刻は実は存在しません。夏時間のことを思い出してみるとわかりますが `"America/Los_Angeles"` では2017年3月12日 午前1時59分59秒から2017年3月12日 午前3時0分0秒に吹っ飛んでいるからです。

また `"America/Los_Angeles"` で2017年11月5日 午前1時30分、という時刻も困ります。この場合は2017年11月5日 午前1時30分 (-07:00) のケースと2017年11月5日 午前1時30分 (-08:00) のケースと、両方がありえてしまうからです。

現在時刻から `ZonedDateTime` を作ったり、タイムゾーン付きの文字列を parse して `ZonedDateTime` を作ったりする場合には、このような問題は起こりにくいでしょう。このような問題が起こる代表的なケースとして、タイムゾーンなしの日付時刻データに「デフォルト」タイムゾーンとしてタイムゾーンを付加しようとするような状況が考えられます。

`ZonedDateTime` は補助的に `java.time.ZoneOffset` を追加で持つこともできます。[^31] このようにしたインスタンスでは、上記の後者のような問題は起こりません。しかし、あるインスタンスが `ZoneOffset` を持っているかどうかは、型からはわかりません。 `ZonedDateTime` インスタンスを受け取る人は常にこような問題に気を使わないとならなくなる、ということを意識しておく必要があります。

[^31]: 例: `"2017-11-05 01:30:00 -07:00[America/Los_Angeles]"`

とは言え `ZonedDateTime` を使うべきケースはあります。 `OffsetDateTime` は夏時間の計算をやってくれませんし、地理的地域の情報は `ZonedDateTime` でしか持つことができません。例えば「夏時間がある地域で店舗の営業時間を扱う」ような場合は、下手に自力で計算してバグを埋めるより JSR 310 に任せてやってもらいましょう。必要性と厄介さのトレードオフです。日付時刻計算の途中で `ZonedDateTime` が必要になるようなケースは、しばしばあると思います。

ただし `ZonedDateTime` を他のコンポーネントとのインターフェースとして使う場合や、外部に保存するデータとして使う際は注意が必要です。そのような場合は補助の `ZoneOffset` を常に入れるように保証できないか、仕様から検討することをお勧めします。

Java #2: `java.util.Date` と `java.util.Calendar`
--------------------------------------------------

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

おまけ
=======

Java のタイムゾーン略称の扱い
------------------------------

このおまけは、上の方で少し触れた Java の `MST` などのタイムゾーン略称に関する深堀りです。

たとえば `MST` などが `America/Denver` と等価に解釈されてしまい、その結果 `2020-07-01 12:34:56 MST` が `2020-07-01 12:34:56 -07:00` と等価に解釈されるべきところを `2020-07-01 12:34:56 -06:00` と解釈されてしまう、という話でした。

### 旧 `java.util.TimeZone`

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

### JSR 310 の `DateTimeFormatter`

`EST`, `MST`, `HST` が直って、めでたしめでたし… (?) と思っていたところに Java 8 で JSR 310 が実装されました。

最初に参照したとおり、この JSR 310 は日付・時刻・タイムゾーンの「現実」を忠実にモデル化していて、かなりよくできていると筆者は考えています。タイムゾーン略称についても、実は本丸の `java.time.ZoneId` では略称の使用をかなり制限していて、[名前から `ZoneId` を作るときに `aliasMap` を明示的に与えないと略称は使えない](https://docs.oracle.com/javase/jp/8/docs/api/java/time/ZoneId.html#of-java.lang.String-java.util.Map-)ようになっています。さらに、[互換性のための標準 Map](https://docs.oracle.com/javase/jp/8/docs/api/java/time/ZoneId.html#SHORT_IDS)も用意されていて、ここでは `EST`, `MST`, `HST` はちゃんと固定オフセットにマッピングされています。

では、なぜ前述した例では `2020/07/01 12:34:56 MST` が `2020-07-01T12:34:56-06:00[America/Denver]` になってしまったのでしょうか?

その答えは [`java.util.format.DateTimeFormatter`](https://docs.oracle.com/javase/jp/8/docs/api/java/time/format/DateTimeFormatter.html) の実装でした。

`DateTimeFormatter` の最も手軽な使い方である [`#ofPattern(String)`](https://docs.oracle.com/javase/jp/8/docs/api/java/time/format/DateTimeFormatter.html#ofPattern-java.lang.String-) で `time-zone name` を表す `z` や `zzzz` を使用すると、それは [`DateTimeFormatterBuilder#appendZoneText(TextStyle)`](https://docs.oracle.com/javase/jp/8/docs/api/java/time/format/DateTimeFormatterBuilder.html#appendZoneText-java.time.format.TextStyle-) の呼び出しに相当します。この `appendZoneText` はかなり幅広いタイムゾーン名表現を受け付けるようになっているようです。 Javadoc にも以下のように注意書きがあります。

> 解析時には、テキストでのゾーン名、ゾーンID、またはオフセットが受け入れられます。テキストでのゾーン名には、一意でないものが多くあります。たとえば、CSTは「Central Standard Time (中部標準時)」と「China Standard Time (中国標準時)」の両方に使用されます。この状況では、フォーマッタのlocaleから得られる地域情報と、その地域の標準ゾーンID (たとえば、America Easternゾーンの場合はAmerica/New_York)により、ゾーンIDが決定されます。appendZoneText(TextStyle, Set)を使用すると、この状況で優先するZoneIdのセットを指定できます。

だからと言って `MST` は `America/Denver` にしないで `-07:00` にしてくれれば…。

### `DateTimeFormatter` に深入り

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

### `DateTimeFormatter` 対策

さて、こういう混乱なく `DateTimeFormatter` を使うにはどうしたらいいでしょうか。以下は完全に筆者の私見ですが、少し検討してみます。

まず Locale に踏み込むのはかなり大変そうです。

そもそも `DateTimeFormatterBuilder#appendZoneText` が自由すぎる・寛容すぎるので、この問題だけがどうにかなっても、さらに予想外の地雷を踏む可能性が高そうだなあ、という気がします。

`DateTimeFormatterBuilder` でタイムゾーン情報の処理を追加するメソッドは `appendZoneText` 以外にもいくつかあります。そこで、用途に合うものから厳密に処理してくれるものを探して使う、というのが、一つよさそうな方針ではないでしょうか。

たとえば、固定オフセットのみを処理する [`appendOffset`](https://docs.oracle.com/javase/jp/8/docs/api/java/time/format/DateTimeFormatterBuilder.html#appendOffset-java.lang.String-java.lang.String-) や、タイムゾーン ID そのものの文字列のみを処理する [`appendZoneId`](https://docs.oracle.com/javase/jp/8/docs/api/java/time/format/DateTimeFormatterBuilder.html#appendZoneId--) などです。パターン文字列を使う場合は `XXX`, `ZZZ`, `VV` などに当たります。いずれの場合も、用途に合った適切なものを考えて選ぶのがいいでしょう。

パターン文字列の詳細な解説は [`DateTimeFormatterBuilder#appendPattern` の Javadoc](https://docs.oracle.com/javase/jp/8/docs/api/java/time/format/DateTimeFormatterBuilder.html#appendPattern-java.lang.String-) にあります。

### どうなる `PST`

ところで、最初の方にも書きましたが、カリフォルニア州では 2018年の住民投票で夏時間の変更が支持されたそうです。

もしこれが実施されたら、今まで `EST`, `MST`, `HST` だけだった特例に、今度は `PST` も追加する必要が出てくるかもしれません。はたして、これからなにが起こるでしょうか。

タイムゾーンの呪いは、そう簡単に解けることはなさそうです。
