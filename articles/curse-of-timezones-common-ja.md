---
title: "タイムゾーン呪いの書 (一般教養編)"
emoji: "🌏" # アイキャッチとして使われる絵文字（1文字だけ）
type: "tech" # tech: 技術記事 / idea: アイデア
topics: [ "timezone", "tzdb" ]
layout: default
published: false
---

(もともと [Qiita に載せていた記事](https://qiita.com/dmikurube/items/15899ec9de643e91497c)でしたが、改訂と同時に [Zenn](https://zenn.dev/) に引っ越すことにしました。 Qiita の方には引っ越した旨と引っ越し先の追記だけして、しばらくはそのまま残すつもりです。)

タイムゾーンという概念はほぼ全ての人が聞いたことがあると思います。ソフトウェア・エンジニアなら多くの方が、自分の得意な言語で、日付・時刻・タイムゾーンにかかわるコードを書いたことがなにかしらあるでしょう。ですが日本に住んで日本の仕事をしていると、国内時差もなく [^no-time-diff-for-now] 夏時間もなく [^no-summer-time-for-now] ほぼ[日本標準時 (Japan Standard Time)](https://ja.wikipedia.org/wiki/%E6%97%A5%E6%9C%AC%E6%A8%99%E6%BA%96%E6%99%82) のみで用が足りることもあって、タイムゾーンというこの厄介な概念について成り立ちから実装上の注意点まで解説した日本語の文書は、残念ながらあまり見つかりませんでした。

[^no-time-diff-for-now]: 今は。
[^no-summer-time-for-now]: 今は。

本記事は、タイムゾーンを扱おうとすると常につきまとう逃げられない「呪い」、筆者がタイムゾーンに苦しんだ恨み [^curse-of-author] を、ソフトウェア・エンジニアリングの視点から闇に踏み込んで記述し、この呪いからの適切な逃げ方、呪いを受けつつも真っ向から立ち向かうための戦い方について、筆者の主観から紹介したいと思います。

[^curse-of-author]: つまり筆者の呪い。

日本国内限定で開発しているのなら難しいこと考えなくていいだろ、と思う人もいるかもしれません。しかし、作ったソフトウェアを他の国や地域に展開するチャンスは、いつ来るかわかりません。そのとき「間違った」日付・時刻処理が障壁となってしまったら、とても悲しいことです。

また、日本でも 2020年開始を目指した夏時間の導入が 2018年に検討されました。ソフトウェア・エンジニアを含む多くの人の反対もあって、同年中に断念されましたが [^give-up-summer-time-in-japan] 結局いつか導入してしまうかもしれません。実際、日本で夏時間導入が検討されたのは、近年でもこの一度だけだったわけではありません。 [^summer-time-in-japan-in-2008] 日本国内でも、急に状況が変わるリスクは常にあります。

[^give-up-summer-time-in-japan]: [「自民、五輪サマータイムを断念」 (2018年10月31日、共同通信社)](https://nordot.app/430305906140398689)
[^summer-time-in-japan-in-2008]: [「サマータイム制度に反対する意見」 (2008年7月14日、日本労働弁護団)](http://roudou-bengodan.org/proposal/gen080730a/)

結局、日本国内が前提であっても、日付・時刻の処理を「まともに」実装しておくに越したことはありません。本記事がその一助となれば幸いです。

本記事では、参考実装として、各種言語・ライブラリの中で今のところ (筆者が知る限り) 最も正面から日付・時刻とタイムゾーンの問題に立ち向かった Java (8 以降) の [JSR 310: Date and Time API](https://jcp.org/en/jsr/detail?id=310) をときどき参照します。

記事中に情報の不足や誤りなどを発見されましたら [^insufficient-and-wrong] ぜひコメントや Pull request をお送りください。各言語における実装についても、コメントや別の記事などでの情報提供をお待ちしております。[^other-languages] 一部 Wikipedia を参照しながら記述していますが、日本語版の Wikipedia には英語版との食い違いがあるようです。そのような場合は、基本的には英語版の方を信用するようにしています。 [^japanese-wikipedia]

[^insufficient-and-wrong]: たぶんたくさんある。
[^other-languages]: 筆者も知りたいので。
[^japanese-wikipedia]: リンクは日本語版の方だったりします。

時差、標準時、タイムゾーン
==========================

「『午後1時』って言ったら、昼くらいだよね」という認識を世界中で共有したいのであれば、地球に住んでいる限り「時差」から逃げることはできません。ある地域でいう「午後1時」はその地域である瞬間を表す「時刻」ですが、同じ瞬間に「午前9時」の地域もあります。これがややこしさの元凶であり、出発点です。 [^internet-time]

[^internet-time]: ところで[インターネットタイム](https://ja.wikipedia.org/wiki/%E3%82%B9%E3%82%A6%E3%82%A9%E3%83%83%E3%83%81%E3%83%BB%E3%82%A4%E3%83%B3%E3%82%BF%E3%83%BC%E3%83%8D%E3%83%83%E3%83%88%E3%82%BF%E3%82%A4%E3%83%A0)ってどうなったんですかね。

とはいえ時刻の定義を「その地点で太陽が南中する時刻をその地点の正午とする」なんていうものにしてしまうと、極端に言えば東西方向に一歩歩くだけでもわずかに時差が発生してしまいます。たとえば東西に少しだけ離れた品川と新宿の間にも、多少の時差が発生することになります。 [^before-standard-time] [^history-of-japan-time-09-18-59]

[^before-standard-time]: 標準時とタイムゾーンという概念が生まれる以前は自治体ごとにその街での太陽の位置に時計を合わせていたので、実際にそのような状態だったようです。 ([Wikipedia:標準時#標準時の歴史](https://ja.wikipedia.org/wiki/%E6%A8%99%E6%BA%96%E6%99%82#%E6%A8%99%E6%BA%96%E6%99%82%E3%81%AE%E6%AD%B4%E5%8F%B2))

[^history-of-japan-time-09-18-59]: 実際、日本標準時が施行された 1887年より前には、東京にも東京の地方時がありました。昔の日時を操作していると顔を出すことがある `+09:18:59` などのオフセットは、このころの東京地方時を UTC との時差で表現しようとしたものです。以下の記事に詳しい解説があります: [「18分59秒をめぐって日本標準時の歴史をひもとくことに」 (2018年 12月 12日、エムスリーテックブログ)](https://www.m3tech.blog/entry/timezone-091859)

そんなことでは法律・商業・社会などいろいろな事情で不便なので、地理的に近い一定の広さの地域では同じ時刻を使うことにしています。その時刻(系)をその地域の「標準時」と呼び、同じ標準時を使う地域を「タイムゾーン」と呼びます。たとえば、いわゆる「日本標準時」を使う「日本全体」が一つのタイムゾーンです。 [^history-of-standard-time]

[^history-of-standard-time]: 広い地域にまたがる「標準時」の概念は、鉄道の普及とともに「鉄道時間 (railway time)」として英国で 1847 年に導入され、そこから 1884 年の国際子午線会議を通して各国に広がったようです。 ([Wikipedia(en):Standard time](https://en.wikipedia.org/wiki/Standard_time#Great_Britain))

タイムゾーンの境界は、一般的には国境や国内の政治的境界に準じています。その方が便利だということもありますが、なによりも、そもそもタイムゾーンや標準時というのは国際的に条約などで取り決められているわけではなく、国や地域がそれぞれ「自分のところのこの地域はこの時刻でやるぜ!」と宣言しているだけだ、ということもあります。 [^dateline]

[^dateline]: いわゆる「日付変更線」も同様で、日付変更線という線が国際的に定められているわけではありません。

タイムゾーンの定義と用例
-------------------------

「タイムゾーン」は、前述のとおり標準時をもとに「同じ標準時を使う地域」として定義するのが一般的です。 [^wikipedia-time-zone] 標準時は、後述する「夏時間」とは独立していて、たとえばアメリカのユタ州とアリゾナ州は同じ「[山岳部標準時 (Mountain Standard Time, `MST`)](https://ja.wikipedia.org/wiki/%E5%B1%B1%E5%B2%B3%E9%83%A8%E6%A8%99%E6%BA%96%E6%99%82)」を使いますが、夏時間制には違いがあります。 [^summer-time-in-utah-and-arizona] そしてこの 2 州は、この定義上は同じタイムゾーンに属します。

[^wikipedia-time-zone]: "A time zone is an area that observes a uniform standard time for legal, commercial and social purposes." ([Wikipedia (en): Time zone](https://en.wikipedia.org/wiki/Time_zone))

[^summer-time-in-utah-and-arizona]: [ユタ州](https://ja.wikipedia.org/wiki/%E3%83%A6%E3%82%BF%E5%B7%9E)は夏時間を採用し、[アリゾナ州](https://ja.wikipedia.org/wiki/%E3%82%A2%E3%83%AA%E3%82%BE%E3%83%8A%E5%B7%9E)の大部分は[夏時間を採用していません](https://ja.wikipedia.org/wiki/%E3%82%A2%E3%83%AA%E3%82%BE%E3%83%8A%E6%99%82%E9%96%93)。

しかしこの「タイムゾーン」という言葉は、実態としてはとても多様であいまいな使われ方をしています。ユタ州とアリゾナ州のようなケースを別のタイムゾーンと呼ぶ例は、後述する Time Zone Database のタイムゾーン ID のように数多くあります。政治主体が違うなどの理由で標準時の定義は異なるものの、時差としては等価な標準時を使っていた国や地域を、まとめて一つのタイムゾーンと呼んだ例もあります。 [^old-windows-timezones-in-japan-korea] 後述する `UTC+9` などの特定の時差そのものをタイムゾーンと呼ぶこともあります。

[^old-windows-timezones-in-japan-korea]: 古い Windows 95 などでは、タイムゾーンの設定が「(GMT+09:00) 東京、大阪、札幌、ソウル、ヤクーツク」などとなっていたのを覚えている方もいるかもしれません。

タイムゾーンをあつかう上では、このことを頭に留めておきましょう。

タイムゾーンの基準: UTC
------------------------

(おそらく、この項には不正確な記述が特に多く含まれています。 [^utc-and-leap-second])

[^utc-and-leap-second]: 分野による用語の差異や歴史的な変遷まですべてを含めて短く正確にまとめきることは、筆者にはできませんでした。ソフトウェア・エンジニアとしての実用にはこのくらいで十分だろう、と筆者が考えた範囲にとどめています。より正確かつ簡潔な記述をご提案いただける方は、ぜひコメントなど残していただければ幸いです。 (ただし正確さのためであっても不要に長くはしたくないと考えています)

世界中で違う国や地域の人が時刻が関係する話をするには、なんらかの基準が必要です。国や地域の間の相対的な時差だけで各地の数多くの時刻を扱うのは、あまりにもたいへんです。

初期に国際的な基準として使われたのが、グリニッジ標準時 (GMT: Greenwich Mean Time) です。 [^greenwich-mean-time] GMT はロンドンのグリニッジから観測される太陽の位置にもとづく時刻、つまり地球の自転と観測にもとづく時刻で、これは実生活にもそのまま有用でした。しかし、自転の周期にはわずかなふらつきがあり、物理的に厳密な24時間とは一致しません。 [^physical-24-hours] その後 GMT を継承して数種類の世界時 (UT：Universal Time) が定義されましたが、これらもやはり地球の自転にもとづくものでした。

[^greenwich-mean-time]: グリニッジ標準時は、前述の「鉄道時間」 [^history-of-standard-time] でもありました。
[^physical-24-hours]: 物理的に厳密な24時間ってなんだ、という相対論的なツッコミはおいておいてください。

逆に、地球の自転によらず、原子時計をもとに定めた時刻が国際原子時 (TAI: Temps Atomique International) です。こちらは物理的に厳密な1秒を刻みますが、地球の一周と TAI の一日にはずれが生じます。

一日・一秒の長さが変わるのも、一日と地球の一周がずれるのも、どちらも不便です。そこで協定世界時 (UTC：Coordinated Universal Time) として折衷案というべき時刻系が提案され、現在では多くの分野で基準として採用されています。 UTC は、原子時計で厳密な一秒を刻みつつも、自転に基づく UT (UT1) との差が 0.9 秒を超えないように、うるう秒を導入しながら調整を続けている時刻系です。うるう秒はめんどうで嫌われがちですが、一秒の長さが変わるのか、一日と地球の一周がずれるのか、うるう秒か、どれかの不便は受け入れないとならないのです。

ちなみにうるう秒の仕組みでは、「60 秒」が挿入される「正のうるう秒」だけではなく、「59 秒」が削除される「負のうるう秒」が実施される可能性も考慮されています。ただし、策定から 2021 年までの間に実施例はありません。

ソフトウェアや通信の分野ではおもに UTC が使われ、また GMT は UTC と同等に扱われることが多いです。

こうして現在では、各国や地域の時差は UTC からの差 (オフセット) で表現するのが一般的になりました。たとえば UTC から 9 時間進んだ日本の時刻は `UTC+9` `UTC+09:00` `+09:00` などと表記します。多くのタイムゾーンは UTC から 1 時間単位の差ですが、そうではない[オーストラリア中部標準時](https://ja.wikipedia.org/wiki/%E3%82%AA%E3%83%BC%E3%82%B9%E3%83%88%E3%83%A9%E3%83%AA%E3%82%A2%E6%99%82%E9%96%93) (`UTC+09:30`) や[ネパール標準時](https://ja.wikipedia.org/wiki/%E3%83%8D%E3%83%91%E3%83%BC%E3%83%AB%E6%A8%99%E6%BA%96%E6%99%82) (`UTC+05:45`) のようなタイムゾーンも存在します。

「地域ベースタイムゾーン」と「固定オフセット」
-------------------------------------------

前述のように、地域によらない `UTC+09:00` のような時刻系のことも「タイムゾーン」と呼ぶことがあります。筆者の知るかぎりでは、「地域を表すタイムゾーン」と、「地域によらず時刻系を表すタイムゾーン」を区別する一般的な言葉は無いようです。 [^wording-fixed-offset-and-region-based]

[^wording-fixed-offset-and-region-based]: ご存じの方、教えてください。

本記事では Java の JSR 310 を参考に、前者を「地域ベース (region-based) タイムゾーン」と、後者を「固定オフセット (fixed offsets)」と呼ぶことにします。具体的には [`java.time.ZoneId`](https://docs.oracle.com/javase/jp/8/docs/api/java/time/ZoneId.html) と [`java.time.ZoneOffset`](https://docs.oracle.com/javase/jp/8/docs/api/java/time/ZoneOffset.html) を参照してください。

タイムゾーンの遷移 (切り替わり)
================================

ある地域のタイムゾーンの「UTC からの差 (オフセット)」は、さまざまな理由で、異なるオフセットに切り替わることがあります。その代表的なものが、悪名高い夏時間です。

夏時間
-------

単なる地域間の時差だけでも大変なのに、それ以上に世界を混沌の渦に陥れているのが「夏時間 [^daylight-summer]」です。通常、夏時間は「明るいうちに仕事を終えて余暇を長く過ごせるように」実施されるので、ある地域が夏時間に切り替わるとき、その地域では少し [^difference-in-summer-time] 時刻が進みます。

[^daylight-summer]: アメリカ系では "Daylight Saving Time" と、ヨーロッパ系では "Summer Time" と呼ばれます。
[^difference-in-summer-time]: ほとんどの地域では 1時間進みます。[オーストラリアの Lord Howe 島のように](https://www.timeanddate.com/time/change/australia/lord-howe-island) 30分だけ進む、というような地域も少数ながら存在するようです。

日本は夏時間に縁が薄いので、このとき具体的に何が起こるのか、知らない方もいるかもしれません。これはなかなかアクロバティックで、夏時間の切り替えのときは時刻がいきなり吹っ飛びます。

たとえば 2020年のアメリカ合衆国では 3月 8日 (日) に夏時間に切り替わりました。 [^united-states-dst-day] そのタイミングは、切り替わる前の時刻でいう午前 2時で、この日の午前 1時 59分 59秒から 1秒経ったと思ったら、午前 3時 0分 0秒になっていた、という具合です。カリフォルニア州を例に、その経過を 1秒ごとに並べると次のようになります。

[^united-states-dst-day]: 2021年時点のアメリカ合衆国では、毎年 3月の第二日曜日に夏時間に切り替わり、その後 11月の第一日曜日に戻る、という規則になっています。 (2007年以降)

| カリフォルニア州の時刻 | UTC からの差 (オフセット) |
| --- | --- |
| 2020-03-08 01:59:58 | 太平洋標準時 (PST) -08:00 |
| 2020-03-08 01:59:59 | 太平洋標準時 (PST) -08:00 |
| 2020-03-08 03:00:00 | 太平洋夏時間 (PDT) -07:00 |
| 2020-03-08 03:00:01 | 太平洋夏時間 (PDT) -07:00 |

夏時間から戻るときはさらにトリッキーで、一見すると同じ時刻を二度経験することになります。再び 2020年のカリフォルニア州を例に取ると、時刻は以下のように経過しました。

| カリフォルニア州の時刻 | UTC からの差 (オフセット) |
| --- | --- |
| 2020-11-01 01:00:00 | 太平洋夏時間 (PDT) -07:00 |
| 2020-11-01 01:00:01 | 太平洋夏時間 (PDT) -07:00 |
| … | … |
| 2020-11-01 01:59:59 | 太平洋夏時間 (PDT) -07:00 |
| 2020-11-01 01:00:00 | 太平洋標準時 (PST) -08:00 |
| 2020-11-01 01:00:01 | 太平洋標準時 (PST) -08:00 |

ちなみに 2021年の日本では夏時間は採用されていませんが、[第二次世界大戦直後に夏時間が実施されたことが一時期あります](https://ja.wikipedia.org/wiki/%E5%A4%8F%E6%99%82%E9%96%93#%E9%80%A3%E5%90%88%E5%9B%BD%E8%BB%8D%E5%8D%A0%E9%A0%98%E6%9C%9F)。

また、現代ではどの国でも夏時間に切り替わってから一年以内に標準時に戻りますが、[第二次世界大戦中の英国では、夏時間に切り替わったまま戻らず、翌年に二重に夏時間に切り替わった](https://ja.wikipedia.org/wiki/%E8%8B%B1%E5%9B%BD%E5%A4%8F%E6%99%82%E9%96%93#%E6%AD%B4%E5%8F%B2)ことがあります。

夏時間制を導入している国や地域でも、そのすべてでアメリカのように安定した切り替えの規則があるとは言いがたいようです。たとえば[チリでは 2016年前後に夏時間制に関する変更が頻繁にありました](https://en.wikipedia.org/wiki/Time_in_Chile#Summer_time_(CLST/EASST))。また[ブラジルでは 2008年まで夏時間の実施地域や切り替え日が毎年個別に公示されていて、公示が開始の直前になったこともあるようです](https://en.wikipedia.org/wiki/Daylight_saving_time_in_Brazil)。

夏時間以外の切り替わり
-----------------------

夏時間以外にも、国や地域の政治的決定によって地域の時刻系が切り替わることがあります。

例えば 2011年の年末には、[サモア独立国](https://ja.wikipedia.org/wiki/%E3%82%B5%E3%83%A2%E3%82%A2)の標準時が `-11:00` から `+13:00` に切り替わりました。この切り替わりは 12月 29日の終わりがそのまま 12月 31日の始まりに続く形で行われ、サモアでは 2011年 12月 30日がまるまる存在しなかったことになりました。

夏時間制の終わり
-----------------

また 2021年現在、世界各地で夏時間制の廃止が議論されています。これもまた国や地域の政治的決定による切り替わりですね。夏時間制の廃止自体は歓迎できることですが、今まで実施されていたものが実施されなくなるのは混乱のもとでもあります。ソフトウェア・エンジニアとしては、注視する必要がありそうです。

たとえば EU 各国における夏時間制は、欧州議会の議決では 2021年を最後に廃止されることになっていました。 [^eu-summer-time-2019] しかし、コロナ禍もあって 2021年 3月時点でも状況は不透明のようです。 [^eu-summer-time-2021]

[^eu-summer-time-2019]: [「欧州議会、サマータイム廃止の法案を可決　２０２１年に」 (2019年 3月 28日、朝日新聞)](https://digital.asahi.com/articles/ASM3W0014M3VUHBI02R.html)
[^eu-summer-time-2021]: [「これで最後？EUが夏時間へ　廃止検討するも議論進まず」 (2021年 3月 27日、朝日新聞)](https://digital.asahi.com/articles/ASP3W4HWJP3VUHBI04L.html)

欧州の状況をさらにややこしくしているのは、夏時間の廃止後、標準時を「旧来の標準時」に戻すのか「それまでの夏時間」に合わせるのか、できるだけそろえたい思惑はあるものの、各国に任されているらしい、ということです。最終的にどうなるのか 2021年 6月現在わかりませんが、極端なことを言えば、これまで同じ「[中央ヨーロッパ標準時 (CET)](https://ja.wikipedia.org/wiki/%E4%B8%AD%E5%A4%AE%E3%83%A8%E3%83%BC%E3%83%AD%E3%83%83%E3%83%91%E6%99%82%E9%96%93)」を使っていて時差のなかったドイツ・フランス・イタリアの間に来年から時差が生じる、なんてことがあるかもしれません。恐ろしい話です。 [^italy-2021]

[^italy-2021]: [「サマータイム廃止でイタリアは最後の夏時間」 (2021年 3月 21日、ニューズウィーク日本版内 World Voice)](https://www.newsweekjapan.jp/worldvoice/vismoglie/2021/03/post-23.php)

アメリカのカリフォルニア州では 2018年の住民投票で夏時間の変更が支持されました。 [^california-proposition-7] 2021年現在まだ具体的な変更の予定はないようですが、これから変わることがあるかもしれません。

[^california-proposition-7]: [「カリフォルニア州、住民は夏時間の変更を支持」 (2018年11月19日、日本貿易振興機構)](https://www.jetro.go.jp/biznews/2018/11/f4ab9860d030abf0.html)

カリフォルニアの変更で要注意なのは、「[太平洋標準時 (PST)](https://ja.wikipedia.org/wiki/%E5%A4%AA%E5%B9%B3%E6%B4%8B%E6%A8%99%E6%BA%96%E6%99%82)」の代表として「ロサンゼルス時間」を用いているソフトウェア実装が意外と多い、ということでしょう。たとえばワシントン州シアトルの時刻を表現するつもりで「太平洋時間」を指定していたつもりが、実は内部的にはそれがロサンゼルス時間扱いになっていて、そんな中でカリフォルニア州の夏時間の変更が実施されたとしたら。…どうするんでしょうね。考えたくない話です。

ソフトウェア技術における時刻
=============================

さて、前段までは主に時刻とタイムゾーンの一般的な仕組みを説明してきました。ここからはソフトウェアや通信の分野に特有の内容を取り上げていきます。

Unix time: 世界共通の時間軸
----------------------------

世界各地の「時刻」には時差がありますが、それでも時間軸は世界共通であり、その時間軸上の一点は、世界中で共通の同じ一瞬です。 [^relativity-physics] たとえば、日本の 2021年 7月 1日 午後 3時 0分 0秒と、英国の 2021年 7月 1日 午前 7時 0分 0秒は、時間軸上では同じ一点です。それぞれの国や地域の中では、それぞれの現地時刻を使っていれば問題ありませんが、タイムゾーンをまたぐ場合、特にソフトウェアやインターネットが関わる仕組みの中では、世界共通の時間軸を扱う必要性が大きく増します。

前述のとおり UTC という共通の基準があるので、要は UTC でその時間軸を表現すればいいのです。ですが、せっかく時間はどこでも平等に流れているのに [^relativity-physics] 年・月・日・時・分・秒の繰り上げなどをいちいち計算するのは無駄です。そこでソフトウェアや通信の分野では、よく [Unix time](https://ja.wikipedia.org/wiki/UNIX%E6%99%82%E9%96%93) を流用して時刻を表現します。

[^relativity-physics]: 相対論的なツッコミは (ry

Unix time は、日本語では「Unix 時間」、英語では "POSIX time" や "(Unix) epoch time" などとも呼ばれ、時刻を UTC の 1970年 1月 1日 0時 0分 0秒からの経過秒数で表現するものです。この UTC の 1970年 1月 1日 0時 0分 0秒を "epoch" と呼びます。

Unix time がわかっていれば、任意のタイムゾーンにおける年・月・日・時・分・秒の現地時刻は簡単に計算できるはず…ですが、一つだけ問題が残ります。前述したうるう秒です。うるう秒は、そもそも予測困難な自転速度のふらつきを埋めるためのしくみなので、将来うるう秒が挿入されるタイミングはなかなか確定できません。そのため、うるう秒を考慮に入れると、未来の時刻に対応する Unix time が計算できません。

そこで Unix time は「経過時間」としての厳密さを捨てて利便性を優先し、うるう秒を無視することになりました。[最近では UTC における 2016年 12月 31日 23時 59分 59秒と 2017 年 1月 1日 0時 0分 0秒の間にうるう秒が挿入されました](https://jjy.nict.go.jp/QandA/data/leapsec.html)が、このとき Unix time の `1483228800` は 2秒間続きました。その経過を 0.5秒ごとに並べると、次のようになります。

| UTC | 小数部を含む Unix time |
| --- | --- |
| 2016-12-31 23:59:59.00 | 1483228799.00 |
| 2016-12-31 23:59:59.50 | 1483228799.50 |
| 2016-12-31 23:59:60.00 | 1483228800.00 |
| 2016-12-31 23:59:60.50 | 1483228800.50 |
| 2017-01-01 00:00:00.00 | 1483228800.00 |
| 2017-01-01 00:00:00.50 | 1483228800.50 |
| 2017-01-01 00:00:01.00 | 1483228801.00 |

秒の整数部だけを見ると、同じ秒が 2秒間続くだけに見えるので、あまり問題ないように見えます。しかし小数部がある場合は、扱いに気をつける必要があります。小数部が普段どおりにカウントされると、時刻が逆進したことになるからです。

うるう秒の「希釈」
-------------------

ソフトウェアや通信業界のすべての仕事で、「一秒単位で厳密な時刻」が必要なわけではありません。 Unix time の逆進も含め、うるう秒は過去に多くの問題を引き起こしてもきました。 [^incidents-by-leap-seconds]

[^incidents-by-leap-seconds]: [『「うるう秒」障害がネットで頻発』 (2012年 7月 2日、 WIRED NEWS)](https://wired.jp/2012/07/02/leap-second-bug-wreaks-havoc-with-java-linux/)

そこで、必要なさそうな場合は UTC で獲得した一秒の厳密さを捨てて、うるう秒を周辺の数時間で「希釈」する手法が 2000年ごろの議論から生まれました。これは近年のソフトウェアやクラウドサービスで一般的なやり方になりつつありますが、この「希釈」にはいくつかの流派があります。

たとえば Java の JSR 310 で使われる[「Java タイム・スケール」](https://docs.oracle.com/javase/jp/8/docs/api/java/time/Instant.html)は、発端となった議論 [^utc-sls-history] から派生した [UTC-SLS (Coordinated Universal Time with Smoothed Leap Seconds)](https://www.cl.cam.ac.uk/~mgk25/time/utc-sls/) をもとにしています。 UTC-SLS は、うるう秒が導入される日の最後の 1000秒に均等にうるう秒を分散させて、うるう秒を見えなくしてしまうものです。

[^utc-sls-history]: このような手法が公に議論されたのは、[ケンブリッジ大学の Markus Kuhn 氏による 2000 年の UTS (Smoothed Coordinated Universal Time)](https://www.cl.cam.ac.uk/~mgk25/uts.txt) が最初のようです。 UTC-SLS はそこから派生していて、後の 2006年には [IETF Internet Draft](https://tools.ietf.org/html/draft-kuhn-leapsecond-00) としても公開されています。

一方 AWS や Google Cloud などのクラウドサービスでは、少し別の流派が一般的になりつつあります。彼らの手法は ["Leap Smear"](https://docs.ntpsec.org/latest/leapsmear.html) と呼ばれていることが多く、おもに NTP を介して提供されます。

[Google が 2008年に実施した最初の Leap Smear](https://googleblog.blogspot.com/2011/09/time-technology-and-leaping-seconds.html) は、うるう秒をその「前」の「20時間」に「非線形に」分散させるものでした。次の [2012年から 2016年のうるう秒における Google の Leap Smear](https://cloudplatform.googleblog.com/2015/05/Got-a-second-A-leap-second-that-is-Be-ready-for-June-30th.html) は、うるう秒の「前後」の「20時間」に「線形に」分散させるものになったようです。その後 [Google はうるう秒の「前後」の「24時間」に「線形に」分散させるやり方を標準とすることを提案しています](https://developers.google.com/time/smear)。 [Amazon が 2015年と 2016年に実施した Leap Smear](https://aws.amazon.com/jp/blogs/aws/look-before-you-leap-the-coming-leap-second-and-aws/) も、同様に 24時間に分散させるもので、クラウドサービスではこのやり方が一般的になると思ってよさそうな雰囲気です。

tzdb: タイムゾーン表現の業界標準
---------------------------------

ソフトウェアの分野で、タイムゾーンに関する最も標準的なデータが [Time Zone Database](https://www.iana.org/time-zones) ([Wikipedia:tz database](https://ja.wikipedia.org/wiki/Tz_database)) でしょう。見たことのある方が多いであろう `Asia/Tokyo` や `Europe/London` のような ID は、この Time Zone Database のものです。

もともと "tz database" や、それを略して "tzdb" と、または単に "tz" などと呼ばれていました。本記事ではおもに "tzdb" と呼びます。 Unix-like な OS で歴史的に使われているパス名から "zoneinfo" とも呼ばれます。もともと Arthur David Olson (達?) が始めたもので、遅くとも 1986年にはメンテナンスされていた記録があります。 [^olson-history] そこから "Olson database" と呼ばれることもあります。

[^olson-history]: ["seismo!elsie!tz ; new versions of time zone stuff" (Nov 25, 1986)](http://mm.icann.org/pipermail/tz/1986-November/008946.html)

tzdb のタイムゾーン ID は、最初の `/` の前に大陸・海洋名を、後ろにそのタイムゾーンを代表する都市名・島名などを用いてつけられます。 [^cities-in-tzdb] 国名は基本的に使われません。 [^countries-in-tzdb] `"America/Indiana/Indianapolis"` のように 3要素で構成されるタイムゾーンも少数ながら存在します。

[^cities-in-tzdb]: 日本語でも「東京時間で」「ニューヨーク時間で」などというのと同じような感覚だと思います。

[^countries-in-tzdb]: 国が関係する状態は変わりやすいので、[政治的事情による変更に対して頑健であるためだとする記載があります](https://github.com/eggert/tz/blob/2021a/theory.html#L143-L151)。人が生活する単位としての「都市」はそれよりは長く維持される傾向にあると考えられているようです。要出典。ちなみに、[初期の提案](https://mm.icann.org/pipermail/tz/1993-October/009233.html)では、国名を使うつもりがあったようです。

tzdb は、今でもボランティアによってメンテナンスされています。タイムゾーンの情報は、世界のどこかで意外なほど頻繁に変わっており、年に数回は新しい版の tzdb がリリースされます。前述のサモア独立国のように政治的理由から変わる場合、夏時間制を新しく導入する場合・取りやめる場合、国や地域の境界線が変わる場合など、様々な理由があります。既に登録されたデータに間違いが見つかることもあります。 [^south-ryukyu-island] 更新はギリギリになることも多く、前述のサモア標準時の変更が tzdb に適用されたのは、[実際に標準時が変わる 2011年 12月のわずか 4ヶ月前のこと](http://mm.icann.org/pipermail/tz/2011-August/008703.html)でした。

[^south-ryukyu-island]: たとえば `Asia/Tokyo` でも、古い BSD ユーザーにはおなじみの [South Ryukyu Islands 時間問題](https://ja.wikipedia.org/wiki/%E6%97%A5%E6%9C%AC%E6%A8%99%E6%BA%96%E6%99%82#South_Ryukyu_Islands%E6%99%82%E9%96%93) がありました。その [South Ryukyu Islands 時間問題に関する調査報告](http://www.tomo.gr.jp/root/9925.html)などもあります。比較的最近でも、[第二次世界大戦中の情報について一部修正されたり](http://mm.icann.org/pipermail/tz/2018-January/025896.html)しています。

tzdb は、特に 1970年 1月 1日以降のすべての変更、夏時間などのオフセット切り替え規則、うるう秒の情報まで、さまざまな情報を含むことを目標にメンテナンスが続けられています。 tzdb は OS や Java などの実行環境、ライブラリなどに組み込まれていて、更新版が出るとそれぞれのメンテナによって取り込まれ、それを組み込んだアップデートとしてそれぞれ世界中に配信されています。

tzdb には 1970年 1月 1日以前の情報もある程度は含まれていますが、暦の違い、歴史資料の問題などもあって、古い情報まで正確なものにしようというモチベーションはあまり無いようです。 [^tzdb-before-1970] 特にタイムゾーン ID の定義は「1970年以降も区別する必要のあるタイムゾーン」のみを定義する、という方針のようです。例えばブラジルのサンパウロを含む一部地域で 1963年に実施された夏時間を表すために [`America/Sao_Paulo` の新設が提案された](https://mm.icann.org/pipermail/tz/2010-January/016007.html)ことがありますが、[この方針により却下されています](https://mm.icann.org/pipermail/tz/2010-January/016010.html)。

[^tzdb-before-1970]: 歴史の解釈にも揺れがあるため、「正確なもの」はそもそも存在しない、と考えられます。

Military time zones
--------------------

主に米軍で使われているタイムゾーン表現で、地域によらない固定オフセットに対応します。 `"A"` 〜 `"Z"` までの大文字アルファベット (`"J"` のみ不使用) を `"-12:00"` から `"+12:00"` まで 1 時間おきに 25 個のオフセットに割り当てています。 ([Wikipedia:en](https://en.wikipedia.org/wiki/List_of_military_time_zones))

`"Z"` の一文字で `"UTC"`/`"GMT"` を指す表現は見たことがある人が多いと思いますが、ここから来ています。

これらも `"PST"` などと同様に [RFC 2822](https://www.ietf.org/rfc/rfc2822.txt) に標準として含まれています。こちらは一意に定まる固定オフセット表現なので「避けるべき」というほどではありません。が、有名な `"Z"` 以外は一般的に知られているとは言い難いので、出力の際にわざわざ選ぶようなものでもないでしょう。 Military time zone では UTC から30分差や15分差のタイムゾーンを表現できない、という問題もあります。

タイムゾーン略称
-----------------

### `JST`: Jerusalem Standard Time?

日本標準時 (Japan Standard Time) は、よく `JST` と略して呼ばれます。これは tzdb にも以下のように[言及](https://github.com/eggert/tz/blob/2021a/asia#L2087-L2093)があります。

```
# From Hideyuki Suzuki (1998-11-09):
# 'Tokyo' usually stands for the former location of Tokyo Astronomical
# Observatory: 139 degrees 44' 40.90" E (9h 18m 58.727s),
# 35 degrees 39' 16.0" N.
# This data is from 'Rika Nenpyou (Chronological Scientific Tables) 1996'
# edited by National Astronomical Observatory of Japan....
# JST (Japan Standard Time) has been used since 1888-01-01 00:00 (JST).
# The law is enacted on 1886-07-07.
```

[`Asia/Tokyo` の定義](https://github.com/eggert/tz/blob/2021a/asia#L2166-L2168) も以下のようになっています。 (`J%sT` は `%s` のところに `S` (標準時) を入れたり `D` (夏時間) を入れたりする記法です。現在の日本で夏時間が実施されているわけではありませんが。)

```
# Zone  NAME            STDOFF  RULES   FORMAT  [UNTIL]
Zone    Asia/Tokyo      9:18:59         -       LMT     1887 Dec 31 15:00u
                        9:00    Japan   J%sT
```

しかし、もう少し tzdb を眺めてみると、以下のように [Jerusalem Standard Time を `JST` と呼んでいるコメント](https://github.com/eggert/tz/blob/2021a/asia#L1706-L1713)も同時に存在するのです。

```
# From Ephraim Silverberg (2001-01-11):
#
# I coined "IST/IDT" circa 1988.  Until then there were three
# different abbreviations in use:
#
# JST  Jerusalem Standard Time [Danny Braniss, Hebrew University]
# IZT  Israel Zonal (sic) Time [Prof. Haim Papo, Technion]
# EEST Eastern Europe Standard Time [used by almost everyone else]
```

と言っても tzdb の解釈としては、実際のところ [Jerusalem Standard Time としての `JST` は、以下のコメントのように除外 (ruled out)](https://github.com/eggert/tz/blob/2021a/asia#L1715-L1720) されてはいます。

```
# Since timezones should be called by country and not capital cities,
# I ruled out JST.  As Israel is in Asia Minor and not Eastern Europe,
# EEST was equally unacceptable.  Since "zonal" was not compatible with
# any other timezone abbreviation, I felt that 'IST' was the way to go
# and, indeed, it has received almost universal acceptance in timezone
# settings in Israeli computers.
```

それでも "Jerusalem Standard Time" という呼び方は各所で現役です。 [^jerusalem-standard-time] [GE Digital のドキュメント](https://www.ge.com/digital/documentation/meridium/Help/V43070/r_apm_pla_valid_time_zones.html)に含まれていたり [Windows のタイムゾーン情報](https://support.microsoft.com/en-us/topic/israel-and-libya-time-zone-update-for-windows-operating-systems-7c6a08aa-8c56-4d7a-2aa8-b956602ebf0a)に含まれていたりします。この呼び方が存在するということは、これを略して `JST` と呼ぶ人々も当地には当然いるのだと思われます。名前が衝突していると、特にソフトウェア対ソフトウェアのやり取りではとても厄介ですね。

[^jerusalem-standard-time]: もちろん、これを "Israel Standard Time" などに呼び替えるのが極めて難しいのは想像に難くないでしょう。 tzdb が ID に国名を使わない方針にしたのと同じ理由で、都市名で呼び続けるのが現実的だということではないでしょうか。

### タイムゾーン略称は非推奨

`JST` のような (主に 3〜5文字の) タイムゾーン略称は一見使いやすいのですが、このように、実は略称から目当てのタイムゾーンを一意に特定できるものではありません。ソフトウェア・プログラムの記述や、ソフトウェアに読み込ませる可能性のあるデータに、タイムゾーン略称を用いるのは避けましょう。特定の国では動いていたのに、他の国や地域のお客さんがついてからデータがおかしくなり始めた、みたいな地獄が待っています。

ソフトウェアに読み込ませるデータには、できるだけ `+09:00` などの固定オフセットを用いるのがいいでしょう。地域ベースのタイムゾーン名を使わなければならない場合も、少なくとも `Asia/Tokyo` などの tzdb ID を使い、さらにそのときのオフセット (`+09:00` など) をセットで持たせましょう。

不幸にも、既にこのような略称を用いたデータがある場合、略称付きのデータが生成されるようになっている場合は、読み込む際にあきらめて決め打ちで読み込むしかありません。データ処理プロセスのできるだけ初期のステージのうちに変換・対処しましょう。できるだけ多くの略称に対して、「`JST` は `+09:00` と同等に扱う」など、仕様を明確にしておきましょう。 Java などの処理系や標準ライブラリが勝手な変換をすることがあるので、その仕様を漏らさず把握して、それも仕様化しておきましょう。

Java 旧標準ライブラリの [`java.util.TimeZone`](https://docs.oracle.com/javase/jp/8/docs/api/java/util/TimeZone.html) では、「3文字のタイムゾーン ID は非推奨」と明記されています。新 JSR 310 の [`java.time.ZoneId`](https://docs.oracle.com/javase/jp/8/docs/api/java/time/ZoneId.html) では標準ではサポートされず、[略称と tzdb 名の対応を後から追加する仕組み](https://docs.oracle.com/javase/jp/8/docs/api/java/time/ZoneId.html#SHORT_IDS)になっています。

これらの略称を使うのは、文脈が明らかな人対人のコミュニケーションのみにしておきましょう。略称が引き起こす問題を、もういくつか以下で紹介します。

### `CST`

もっとわかりやすく衝突している例があります。 `CST` です。

日本やアメリカ合衆国でソフトウェア・エンジニアとして働いていると `CST` と聞けば「[中部標準時 (Central Standard Time)](https://ja.wikipedia.org/wiki/%E4%B8%AD%E9%83%A8%E6%A8%99%E6%BA%96%E6%99%82)」のことだと思ってしまいがちですが、「[中国標準時 (China Standard Time, Chinese Standard Time)](https://ja.wikipedia.org/wiki/%E4%B8%AD%E5%9B%BD%E6%A8%99%E6%BA%96%E6%99%82)」も `CST` です。さらに、キューバ標準時 (Cuba Standard Time) も `CST` です。 [^relationship-with-usa]

[^relationship-with-usa]: アメリカ合衆国とはややこしい関係にある国ばかりですが、偶然か必然か。

"China Standard Time" は、[現在も tzdb にも記載されています](https://github.com/eggert/tz/blob/2021a/asia#L40-L55)。 "Jerusalem Standard Time" のように除外されているわけでも非推奨にもなっているわけでもありません。

```
# The following alphabetic abbreviations appear in these tables
# (corrections are welcome):
#            std  dst
#            LMT        Local Mean Time
#       2:00 EET  EEST  Eastern European Time
#       2:00 IST  IDT   Israel
#       5:30 IST        India
#       7:00 WIB        west Indonesia (Waktu Indonesia Barat)
#       8:00 WITA       central Indonesia (Waktu Indonesia Tengah)
#       8:00 CST        China
#       8:00 HKT  HKST  Hong Kong (HKWT* for Winter Time in late 1941)
#       8:00 PST  PDT*  Philippines
#       8:30 KST  KDT   Korea when at +0830
#       9:00 WIT        east Indonesia (Waktu Indonesia Timur)
#       9:00 JST  JDT   Japan
#       9:00 KST  KDT   Korea when at +09
```

[`Asia/Shanghai` の定義](https://github.com/eggert/tz/blob/2021a/asia#L668-L672)が以下のとおりです。最後の行の `[UNTIL]` 列が空欄なので、現在でも有効ということですね。

```
# Zone  NAME            STDOFF  RULES   FORMAT  [UNTIL]
# Beijing time, used throughout China; represented by Shanghai.
Zone    Asia/Shanghai   8:05:43         -       LMT     1901
                        8:00    Shang   C%sT    1949 May 28
                        8:00    PRC     C%sT
```

[`America/Havana` も同様に定義](https://github.com/eggert/tz/blob/2021a/northamerica#L3373-L3376)されています。

```
# Zone  NAME            STDOFF  RULES   FORMAT  [UNTIL]
Zone    America/Havana  -5:29:28 -      LMT     1890
                        -5:29:36 -      HMT     1925 Jul 19 12:00 # Havana MT
                        -5:00   Cuba    C%sT
```

`CST` の重複については [`java.util.TimeZone` の公式 Javadoc](https://docs.oracle.com/javase/jp/8/docs/api/java/util/TimeZone.html) にも注意書きがあります。

### `EST`, `EDT`, `CST`, `CDT`, `MST`, `MDT`, `PST`, `PDT`

さて `CST` の衝突は前段で確認しましたが、一部のソフトウェアでは (`CST` を含む) アメリカ合衆国内の一部のタイムゾーン略称を、優先的に解釈するようになっています。たとえば [Ruby の `Time`](https://github.com/ruby/ruby/blob/v3_0_1/lib/time.rb#L40-L43) や [Java の `java.util.TimeZone`](https://docs.oracle.com/javase/jp/8/docs/api/java/util/TimeZone.html) がそうです。これは [RFC 2822](https://www.ietf.org/rfc/rfc2822.txt) などでこれらの略称が標準として含まれていることが、おおもとの理由としてあるようです。 [^rfc2822]

[^rfc2822]: 要出典。

しかしこれらには実装によって微妙な扱いの違いがあり、それが今後、夏時間制の変更をきっかけとして噴出する可能性がありそうです。

まず本来、これらの略称はそれぞれ標準時、または夏時間のどちらか一方に対応します。たとえば `PST` はあくまで Pacific "Standard Time" (標準時) であり、太平洋時間のうちの夏時間を指すことはありません。太平洋時間の夏時間は、あくまで `PDT` (Pacific "Daylight (Saving) Time") です。つまり `PST` が指すのは (太平洋時間の規定が変わらないかぎり) 常に `-08:00` であり、「夏になるとカリフォルニア時間が `PST` (`-08:00`) から `PDT` (`-07:00`) に移行する」のであって、「夏になると `PST` が `-07:00` になる」わけではありません。上記の [RFC 2822 にも、明確に `PST is semantically equivalent to -0800` などと記載されています](https://datatracker.ietf.org/doc/html/rfc2822#section-4.3)。

実際 [Ruby の `Time` はこれにのっとった実装をしています](https://github.com/ruby/ruby/blob/v3_0_1/lib/time.rb#L40-L43)。 `PST` は常に `-8` として扱い、そして `PDT` は常に `-7` として扱います。これは正しいです。

しかし Java と Java の挙動を継承した [Joda-Time](https://www.joda.org/joda-time/) は、少し違う扱いをしてしまいました。 `PST` などの略称を `America/Los_Angeles` などの地域ベースのタイムゾーン ID に対応付けてしまったのです。

カリフォルニアが夏時間の 2020年 7月 1日のとある日時を表現した文字列 `2020-07-01 12:34:56 PST` は、本来 `2020-07-01 12:34:56 -08:00` と等価に解釈しなければなりません。しかし、このあたりの Java 標準 API はこれを `2020-07-01 12:34:56 America/Los_Angeles` と等価に解釈します。するとこれは `2020-07-01` という日付が夏時間であることから `2020-07-01 12:34:56 -07:00` と等価に解釈されてしまうのです。

「2020年 7月 1日が夏時間なのは明らかなんだから `2020-07-01 12:34:56 PST` なんて書くほうが悪いでしょ」と思われるかもしれません。…しかし、本当にそうでしょうか?

同じことは `MST` (Mountain Standard Time; 山岳部標準時) でも起きています。 Java や Joda-Time は `MST` を `America/Denver` と等価に解釈します。 Denver のあるコロラド州は夏時間を採用しているので、冬に `-07:00` だった `MST` は、夏には `-06:00` と解釈されるようになります。これは Denver ではたしかに正しい解釈です。

しかし同じ山岳部時間を使うアリゾナ州では、前述のとおり、一部 (Navajo Nation) を除いて夏時間を採用していないのです。ということは、夏の日付に `MST` を合わせた `2020-07-01 12:34:56 MST` は (Navajo Nation を除く) アリゾナ州では有効なはずの表現であり、これは本来 `2020-07-01 12:34:56 -07:00` と等価に解釈されなければなりません。しかし Java 標準 API では `2020-07-01 12:34:56 -06:00` と等価に解釈されてしまいます。

このことは、以下のコードで容易に確認できます。

```
import java.time.format.DateTimeFormatter;
import java.time.ZonedDateTime;

public final class MountainTime {
    public static void main(final String[] args) throws Exception {
        System.out.println(ZonedDateTime.parse("2020/01/01 12:34:56 MST", FORMATTER));
        System.out.println(ZonedDateTime.parse("2020/01/01 12:34:56 MDT", FORMATTER));
        System.out.println(ZonedDateTime.parse("2020/07/01 12:34:56 MST", FORMATTER));
        System.out.println(ZonedDateTime.parse("2020/07/01 12:34:56 MDT", FORMATTER));

        // America/Phoenix はアリゾナ州で、夏時間を採用していません。
        System.out.println(ZonedDateTime.parse("2020/01/01 12:34:56 America/Phoenix", FORMATTER));
        System.out.println(ZonedDateTime.parse("2020/01/01 12:34:56 America/Phoenix", FORMATTER));
        System.out.println(ZonedDateTime.parse("2020/07/01 12:34:56 America/Phoenix", FORMATTER));
        System.out.println(ZonedDateTime.parse("2020/07/01 12:34:56 America/Phoenix", FORMATTER));
    }

    private static final DateTimeFormatter FORMATTER = DateTimeFormatter.ofPattern("yyyy/MM/dd HH:mm:ss zzz");
}
```

出力は以下のようになります。 (ここでは Java 8 で確認していますが OpenJDK 16.0.1 でも同様の結果になることを確認しています。)

```
$ java -version
openjdk version "1.8.0_292"
OpenJDK Runtime Environment (build 1.8.0_292-8u292-b10-0ubuntu1~20.04-b10)
OpenJDK 64-Bit Server VM (build 25.292-b10, mixed mode)

$ java MountainTime
2020-01-01T12:34:56-07:00[America/Denver]
2020-01-01T12:34:56-07:00[America/Denver]
2020-07-01T12:34:56-06:00[America/Denver]
2020-07-01T12:34:56-06:00[America/Denver]
2020-01-01T12:34:56-07:00[America/Phoenix]
2020-01-01T12:34:56-07:00[America/Phoenix]
2020-07-01T12:34:56-07:00[America/Phoenix]
2020-07-01T12:34:56-07:00[America/Phoenix]
```

Java のこのあたりの挙動を掘り下げていくと無駄に長大になったので、興味がある方は最下部の「おまけ」をご覧ください。

発展: 日付と暦
===============

ここまで、地球の自転にともなう周期 (一日・時刻) について主に見てきましたが、地球の公転にともなう周期 (一年・日付) はどうでしょうか?

時刻とタイムゾーンの話も闇が広がっていますが、日付の話、特に「暦」の話もまた、広大な闇につながっています。現代でこそ、一部に太陰暦の文化や国・地域の暦という文化もあるものの、現在の「西暦」であるところの「グレゴリオ暦」が広く普及して共通言語として成立するようになりました。ここ数十年の日付のみを扱うかぎりは、そんなにひどいことにはならないでしょう。しかし、ひとたび昔の日付を扱うことになると、考えなければならないことが膨大に発生します。

本記事内でそこまで扱おうとすると、記事が倍以上の長さになってしまいそうなので、とてもよくまとまったブログ記事を紹介して、その代わりとしたいと思います。

[「西暦1年は閏年か？」 (2020年 10月 30日、なぎせゆうきさんのブログ記事)](https://nagise.hatenablog.jp/entry/2020/10/30/173911)

実装の話
=========

[Qiita に載せていた旧 2018年版の記事](https://qiita.com/dmikurube/items/15899ec9de643e91497c)では、実際に時刻処理を実装するときの具体例として、ここから Java 特有の話を紹介していました。しかし改訂にともなって全体がさらに長くなってしまったこともあり、言語やソフトウェア固有の話は記事を分けることにしました。

いまのところ元記事から分割した Java 編のみにリンクを張っていますが、他の言語やソフトウェア特有の話をまとめた記事があったら、そちらも紹介させていただくかもしれません。

* [タイムゾーン呪いの書 (Java 編)](https://zenn.dev/dmikurube/articles/curse-of-timezones-java-ja)

以下では、言語やソフトウェアによらない、一般的な考え方を紹介します。

言語に依らない戦略
-------------------

JSR 310 の場合の一般化になりますが、以下の点に気をつけるといいだろうと筆者は考えています。

* うるう秒を気にしなくてよく、昔の日付時刻を扱う必要もなければ、内部表現を UNIX 時間で持ち回れないか検討する。
* 日付時刻データを、特に外部コンポーネントとのインターフェースに使う場合や、外部に保存する場合 :
    * 特別に省略する必要性がない限り、タイムゾーン情報は省略しない。
    * タイムゾーン情報が必要ない場合は明示的に UTC に変換して、データにも明示的に UTC と持たせる。
    * タイムゾーンが必要でも固定オフセット (`"+09:00"` など) で済む用途なら、なるべく固定オフセットを使う。
    * 地域ベースのタイムゾーンが必要な場合は、オフセットの追記ができないか検討する。
* コード内で日付時刻データを扱う場合 :
    * タイムゾーン無しの日付時刻や地域ベースのタイムゾーンを扱う範囲を、コードの一部に限定できないか検討する。
* 地域ベースタイムゾーンを使う場合は、実行環境 (OS, ランタイム) やライブラリの仕様を調べて tz database の更新戦略を立てる。
* 3文字/4文字のタイムゾーン略語は使わず、地域ベースのタイムゾーンが必要な場合は、できるだけ tz database 名を使う。略語を使わざるをえない場合は、どの略語がどのタイムゾーンに対応するか、仕様から明示する。
