---
title: "タイムゾーン呪いの書 (知識編)"
emoji: "🌍" # アイキャッチとして使われる絵文字（1文字だけ）
type: "tech" # tech: 技術記事 / idea: アイデア
topics: [ "datetime", "timezone", "tzdb" ]
layout: default
published: true
---

「タイムゾーン呪いの書」は、もともと 2018年に [Qiita に投稿した記事](https://qiita.com/dmikurube/items/15899ec9de643e91497c)でしたが、大幅な改訂を 2021年におこない、同時にこちらの [Zenn](https://zenn.dev/) に引っ越すことにしました。

この改訂では [Software Design 誌の 2018年 12月号](https://gihyo.jp/magazine/SD/archive/2018/201812)に特集の一章として寄稿した内容も取り込みつつ、夏時間をめぐって各地で起きつつある変化について 2021年 6月現在の状況なども追加しました。そんな追記もしていたら記事全体が長大になってしまったため、この「知識編」と、[「実装編」](./curse-of-timezones-impl-ja)・[「Java 編」](./curse-of-timezones-java-ja)に記事を分けました。「知識編」は、導入にあたる第一部です。

Qiita のほうは、引っ越した旨とこの引っ越し先へのリンクだけ追記して、しばらくそのまま残すつもりです。

はじめに
=========

タイムゾーンという概念のことは、ほとんどの人が聞いたことがあると思います。ソフトウェア・エンジニアでも多くの方が、時刻やタイムゾーンにかかわるプログラムを、なにかしら書いたことがあるでしょう。ですが、日本に住んで日本の仕事をしていると、国内時差もなく [^no-time-diff-for-now] 夏時間もなく [^no-summer-time-for-now] [日本標準時 (Japan Standard Time)](https://ja.wikipedia.org/wiki/%E6%97%A5%E6%9C%AC%E6%A8%99%E6%BA%96%E6%99%82) のみでほぼ用が足りてしまうので、このタイムゾーンという厄介な概念についてちゃんと調べる機会は多くありません。実際、成り立ちから実装上の注意点まで解説したような日本語の文書は、残念ながらこれまであまり見つかりませんでした。

[^no-time-diff-for-now]: 今は。
[^no-summer-time-for-now]: 今は。

日本国内限定で開発しているならタイムゾーンなんてややこしいことは考えなくていい、と考える人は多いかもしれません。しかし、作ったソフトウェアやサービスを他の国や地域に展開するチャンスは、いつやって来るかわかりません。そのとき、間違った実装の時刻処理が展開の障害となってしまったら、とても悲しいことです。

2018年には、日本でも夏時間の導入が 2020年開始を目指して検討されました。ソフトウェア・エンジニアをふくむ多くの人の反対もあって、同年中に断念されましたが [^give-up-summer-time-in-japan] いつか導入してしまうかもしれません。実際、日本で夏時間導入が検討されたのは、近年でもこの一度だけだったわけではありません。 [^summer-time-in-japan-in-2008] 日本国内でも、急に状況が変わるリスクは常にあります。日本国内が前提であっても、まともに実装しておくに越したことはありません。

[^give-up-summer-time-in-japan]: [「自民、五輪サマータイムを断念」 (2018年10月31日、共同通信社)](https://nordot.app/430305906140398689)
[^summer-time-in-japan-in-2008]: [「サマータイム制度に反対する意見」 (2008年7月14日、日本労働弁護団)](http://roudou-bengodan.org/proposal/gen080730a/)

本記事では、タイムゾーンをあつかおうとすると常につきまとう「呪い」、筆者がタイムゾーンに苦しんだ恨み [^curse-of-author] を、ソフトウェア・エンジニアリングの視点から闇に踏み込んで記述し、この呪いからの適切な逃げかた、そして呪いを受けつつも真っ向から立ち向かうための戦いかたについて、筆者の主観で紹介します。本記事が、時刻やタイムゾーンの処理を「まともに」実装する一助となれば幸いです。

[^curse-of-author]: つまり筆者の呪い。

本記事では参考実装として、日付・時刻とタイムゾーンの問題に真正面から立ち向かっている Java の [JSR 310: Date and Time API](https://jcp.org/en/jsr/detail?id=310) をときどき参照します。

記事中に情報の不足や誤りなどを発見されましたら [^insufficient-and-wrong] ぜひコメントなどをお送りください。他の言語やソフトウェアにおける実装についても、コメントや別の記事などでの情報提供をお待ちしております。[^other-languages] 一部 Wikipedia なども参照しながら記述していますが、日本語版の Wikipedia には英語版との食い違いがあるようです。そのような場合は、基本的には英語版のほうを信用するようにしています。 [^japanese-wikipedia]

[^insufficient-and-wrong]: たぶんたくさんある。
[^other-languages]: 筆者も知りたいので。
[^japanese-wikipedia]: リンクは日本語版のほうだったりします。

時差、標準時、タイムゾーン
===========================

「『午後 1時』っていったら、だいたい昼くらいだよね」という認識を世界中で共有したければ、地球に住むかぎり「時差」から逃げることはできません。ある地域でいう「午後 1時」はその地域のある瞬間をあらわす「時刻」ですが、同じ瞬間に「午前 9時」の地域もあります。これがややこしさの元凶であり、出発点です。 [^internet-time]

[^internet-time]: ところで[インターネットタイム](https://www.swatch.com/en-en/internet-time.html) ([Wikipedia](https://ja.wikipedia.org/wiki/%E3%82%B9%E3%82%A6%E3%82%A9%E3%83%83%E3%83%81%E3%83%BB%E3%82%A4%E3%83%B3%E3%82%BF%E3%83%BC%E3%83%8D%E3%83%83%E3%83%88%E3%82%BF%E3%82%A4%E3%83%A0)) ってどうなったんですかね。

とはいえ、時刻を「その地点で太陽が南中する時刻をその地点の正午とする」なんて定義にしてしまうと、極端にいえば東西方向に一歩歩くだけでもわずかに時差が発生してしまいます。たとえば東西に少しだけ離れた品川と新宿の間にも、多少の時差が発生することになります。 [^before-standard-time] [^history-of-japan-time-09-18-59]

[^before-standard-time]: 標準時とタイムゾーンという概念が普及する前は、その街で見える太陽の位置にその街の時計を合わせていたので、実際にそれに近い状態だったようです。 ([Wikipedia:標準時#標準時の歴史](https://ja.wikipedia.org/wiki/%E6%A8%99%E6%BA%96%E6%99%82#%E6%A8%99%E6%BA%96%E6%99%82%E3%81%AE%E6%AD%B4%E5%8F%B2))

[^history-of-japan-time-09-18-59]: 日本標準時が施行された 1887年より前には、東京にも東京の地方時がありました。昔の日時を操作していると顔を出すことがある `+09:18:59` などのオフセットは、このころの東京地方時を UTC との時差で表現しようとしたものです。次の記事に詳しい解説があります: [「18分59秒をめぐって日本標準時の歴史をひもとくことに」 (2018年 12月 12日、エムスリーテックブログ)](https://www.m3tech.blog/entry/timezone-091859)

そんなことでは法律・商業・社会などいろいろな面で不便なので、地理的に近い一定の広さの地域では同じ時刻を使うことにしました。その時刻 (系) をその地域の「標準時」と呼び、同じ標準時を使う地域を「タイムゾーン」と呼びます。たとえば「日本標準時」を使う「日本全体」が一つのタイムゾーンです。 [^history-of-standard-time]

[^history-of-standard-time]: 広い地域にまたがる「標準時」の概念は、鉄道の普及とともに「鉄道時間 (railway time)」としてまずイギリスで 1847 年に導入され、そこから 1884年の国際子午線会議を通して各国に広がったようです。 ([Wikipedia(en):Standard time](https://en.wikipedia.org/wiki/Standard_time#Great_Britain))

地域の人が日常生活に用いる時刻のことを特に指して「[常用時](https://ja.wikipedia.org/wiki/%E5%B8%B8%E7%94%A8%E6%99%82)」 ([civil time](https://en.wikipedia.org/wiki/Civil_time)) とも呼びます。 [^civil-time]

[^civil-time]: 「標準時」は後述する「夏時間」とは別の時刻ですが、「常用時」は、それが標準時であれ夏時間であれ「その地域のその時点で人々が使っている有効な時刻」のことを指します。

タイムゾーンの境界は、一般的には国境や州境、市境などの政治的境界に準じます。タイムゾーンの役割を考えればそのほうが便利なのはもちろんですが、そもそもタイムゾーンや標準時とは国際的に条約などで定めているわけではなく、国や地域の政府がそれぞれ「自分のところのこの地域はこの時刻でやるぜ!」と宣言しているだけだからでもあります。 [^dateline]

[^dateline]: いわゆる「日付変更線」も同様で、日付変更線という「線」が国際的に定められているわけではありません。

タイムゾーンの定義と用法
-------------------------

前述のように、「タイムゾーン」は「同じ標準時を使う地域」として標準時を基準に定義するのが一般的です。 [^wikipedia-time-zone] 「標準時」は後述の「夏時間」とは独立で、たとえばアメリカのコロラド州とアリゾナ州は同じ「[山岳部標準時](https://ja.wikipedia.org/wiki/%E5%B1%B1%E5%B2%B3%E9%83%A8%E6%A8%99%E6%BA%96%E6%99%82)」を使いますが、夏時間制に違いがあります。 [^summer-time-in-colorado-and-arizona] そしてこの二州は、この定義では同じタイムゾーンに属します。

[^wikipedia-time-zone]: "A time zone is an area that observes a uniform standard time for legal, commercial and social purposes." ([Wikipedia (en): Time zone](https://en.wikipedia.org/wiki/Time_zone))

[^summer-time-in-colorado-and-arizona]: [コロラド州](https://ja.wikipedia.org/wiki/%E3%82%B3%E3%83%AD%E3%83%A9%E3%83%89%E5%B7%9E)は夏時間を採用し、[アリゾナ州](https://ja.wikipedia.org/wiki/%E3%82%A2%E3%83%AA%E3%82%BE%E3%83%8A%E5%B7%9E)の大部分は[夏時間を採用していません](https://ja.wikipedia.org/wiki/%E3%82%A2%E3%83%AA%E3%82%BE%E3%83%8A%E6%99%82%E9%96%93)。

しかしこの「タイムゾーン」という用語は、実態としてはとても多様であいまいな使われかたをしています。後述する Time Zone Database (tzdb) のタイムゾーン ID のように、コロラド州とアリゾナ州のようなケースを異なるタイムゾーンとして分ける例は数多くあります。逆に、政治主体が違うので本来的には異なる「標準時」の定義を持つものの、時差としては同等な標準時を使っていた国や地域を、まとめて一つのタイムゾーンと呼んだ例もあります。 [^old-windows-timezones-in-japan-korea] さらには後述する `UTC+9` などの特定の時差そのものをタイムゾーンと呼ぶこともあります。

[^old-windows-timezones-in-japan-korea]: 古い Windows 95 などで、タイムゾーンの設定が「(GMT+09:00) 東京、大阪、札幌、ソウル、ヤクーツク」などとなっていたのを覚えている方もいるかもしれません。

タイムゾーンをあつかうときは、この用語のあいまいさを念頭に置いておきましょう。

UTC: タイムゾーンの基準
------------------------

世界中の国や地域の人々が、時刻の話をまじえながらコミュニケーションをとるには、どこかに基準が必要です。世界各地の人々が、国や地域間の相対的な時差を何通りにも組み合わせて話をするというのは、ちょっと非現実的です。

国際的な基準として初期に使われたのが「[グリニッジ標準時](https://ja.wikipedia.org/wiki/%E3%82%B0%E3%83%AA%E3%83%8B%E3%83%83%E3%82%B8%E6%A8%99%E6%BA%96%E6%99%82)」 ([GMT: Greenwich Mean Time](https://en.wikipedia.org/wiki/Greenwich_Mean_Time)) です。 [^greenwich-mean-time] GMT はロンドンのグリニッジから観測される太陽の位置にもとづく時刻であり、これは太陽に合わせる一般の人々の日常生活にも、つまり「常用時」としても、そのまま有用でした。しかし、別の見方をすると GMT は地球の自転と観測にもとづく時刻ともいえます。地球の自転周期は常に一定というわけではなく、わずかなふらつきがあります。物理的に厳密な「24時間」とは一致しません。 [^physical-24-hours]

また、数種類の「[世界時](https://ja.wikipedia.org/wiki/%E4%B8%96%E7%95%8C%E6%99%82)」 ([UT: Universal Time](https://en.wikipedia.org/wiki/Universal_Time)) [^universal-time] が GMT を継承して定義されましたが、これらもやはり地球の自転にもとづくもので、同じ問題がありました。

[^greenwich-mean-time]: グリニッジ標準時は、前述の「鉄道時間」 [^history-of-standard-time] でもありました。
[^physical-24-hours]: 物理的に厳密な 24時間ってなんだ、という相対論的なツッコミはおいておいてください。
[^universal-time]: 世界時の種類には UT0, UT1, UT2 などがあります。 UT2 の用途は後述の UTC でほぼ置き換えられ、もう実質的に使われていないようですが。

地球の自転によらずに、原子時計をもとに定めた時刻が「[国際原子時](https://ja.wikipedia.org/wiki/%E5%9B%BD%E9%9A%9B%E5%8E%9F%E5%AD%90%E6%99%82)」 ([TAI: Temps Atomique International](https://en.wikipedia.org/wiki/International_Atomic_Time)) です。こちらは物理的に厳密な一秒を刻みますが、そうすると地球の一周と TAI の一日の間に「ズレ」が生じます。

一日・一秒の長さがその時々で変わるのも、地球の一周と生活上の一日がずれるのも、どちらも不便です。そこで「[協定世界時](https://ja.wikipedia.org/wiki/%E5%8D%94%E5%AE%9A%E4%B8%96%E7%95%8C%E6%99%82)」 ([UTC: Coordinated Universal Time](https://en.wikipedia.org/wiki/Coordinated_Universal_Time)) という折衷案というべき時刻系が提案され、現在では UTC が多くの分野で基準として採用されています。 UTC は、原子時計で厳密な一秒を刻みつつも、地球の自転に基づく世界時 (UT1) との差が 0.9秒を超えないように、「うるう秒」を導入しながら調整を続けている時刻系です。うるう秒はめんどうで嫌われがちですが、一秒の長さがその時々で変わるか、地球の一周と実生活の一日がずれるか、うるう秒か、どれかの不便は受け入れないとならないのです。

ちなみにうるう秒のしくみには、「59分 60秒」が挿入される「正のうるう秒」だけではなく、「59分 59秒」が削除される「負のうるう秒」もあります。ただし、策定から 2021年までの間に「負のうるう秒」の実施例はありません。

ソフトウェアや通信の分野では UTC がおもに使われ、また GMT は UTC と同等とあつかわれることが多いです。

こうして現在では、各国や地域の時刻は UTC からの時差 (オフセット) で表現するのが一般的になりました。たとえば UTC から 9時間進んだ日本の時刻は `UTC+9`, `UTC+09:00`, `+09:00` のように表記します。多くのタイムゾーンは UTC から 1時間単位の差ですが、そうではない[オーストラリア中部標準時](https://ja.wikipedia.org/wiki/%E3%82%AA%E3%83%BC%E3%82%B9%E3%83%88%E3%83%A9%E3%83%AA%E3%82%A2%E6%99%82%E9%96%93) (`UTC+09:30`) や[ネパール標準時](https://ja.wikipedia.org/wiki/%E3%83%8D%E3%83%91%E3%83%BC%E3%83%AB%E6%A8%99%E6%BA%96%E6%99%82) (`UTC+05:45`) のようなタイムゾーンも存在します。

「地域ベース」と「オフセット」
-------------------------------

前述したように、地域によらない `UTC+09:00` のような特定の時差そのものを「タイムゾーン」と呼ぶこともあります。「地域をあらわすタイムゾーン」と「地域によらず特定の時差をあらわすタイムゾーン」を用語で区別できたらいいのですが、残念ながらこれらを区別する一般的な用語はなさそうです。 [^wording-fixed-offset-and-region-based] [^zone-and-offset-in-stackoverflow]

[^wording-fixed-offset-and-region-based]: あくまで筆者が知るかぎりでは。ご存じの方がいらっしゃいましたら、ぜひ教えてください。
[^zone-and-offset-in-stackoverflow]: Stack Overflow の質問 ["Daylight saving time and time zone best practices"](https://stackoverflow.com/questions/2532729/daylight-saving-time-and-time-zone-best-practices) や、同 tag wiki の [`timezone`](https://stackoverflow.com/tags/timezone/info) では "time zone" と "time zone offset" と呼び分けています。

本記事では JSR 310 を参考に、前者を「地域ベースのタイムゾーン」 (region-based time zone) と、後者を「固定オフセット」または単に「オフセット」と呼ぶことにします。具体的には [`java.time.ZoneId`](https://docs.oracle.com/javase/jp/8/docs/api/java/time/ZoneId.html) と [`java.time.ZoneOffset`](https://docs.oracle.com/javase/jp/8/docs/api/java/time/ZoneOffset.html) を参照してください。

タイムゾーンの切り替わり
=========================

ある地域のタイムゾーンのオフセットは、さまざまな理由で、別のオフセットに切り替わることがあります。その代表が、悪名高い「夏時間」 [^daylight-saving-summer] です。

[^daylight-saving-summer]: アメリカ系では "Daylight Saving Time" と、ヨーロッパ系では "Summer Time" と呼ばれることが多いです。

夏時間
-------

単なる国や地域ごとの時差だけでもめんどうな話なのに、それ以上に世界を混沌の渦におとしいれているのが夏時間です。夏時間は一般に「明るいうちに仕事を終えて余暇を長く過ごせるように」実施されるので、ある地域が夏時間に切り替わるときには、その地域では少し [^difference-in-summer-time] 時刻が進みます。

[^difference-in-summer-time]: ほとんどの地域では 1時間進みます。 30分だけ進む[オーストラリアの Lord Howe 島](https://www.timeanddate.com/time/change/australia/lord-howe-island)のような地域も、少数ながら存在します。

日本は夏時間に縁が薄いので、切り替わるとき具体的に何が起こるのか、知らない方もいるかもしれません。これはなかなかアクロバティックで、切り替えのときは時刻がいきなり吹っ飛びます。

たとえば 2020年のアメリカ合衆国では 3月 8日 (日) に夏時間に切り替わりました。 [^united-states-daylight-saving] そのタイミングは切り替わる前の時刻でいう午前 2時で、その日の午前 1時 59分 59秒から 1秒経ったと思ったら午前 3時 0分 0秒になっていた、という具合です。カリフォルニア州を例に経過を 1秒ごとに並べると、以下のようになります。

[^united-states-daylight-saving]: 2007年以降 2021年時点のアメリカ合衆国では、毎年三月の第二日曜日に夏時間に切り替わり、その後十一月の第一日曜日に標準時に戻る、という規則になっています。

| カリフォルニア州の時刻 | UTC からの差 (オフセット) |
| --- | --- |
| 2020-03-08 01:59:58 | 太平洋標準時 (`PST`) -08:00 |
| 2020-03-08 01:59:59 | 太平洋標準時 (`PST`) -08:00 |
| 2020-03-08 03:00:00 | 太平洋夏時間 (`PDT`) -07:00 |
| 2020-03-08 03:00:01 | 太平洋夏時間 (`PDT`) -07:00 |

つまり、カリフォルニア州に 2020年 3月 8日 午前 2時 30分という時刻は存在しませんでした。

夏時間から標準時に戻るときはさらにトリッキーで、一見すると同じ時刻を二度経験することになります。再び 2020年のカリフォルニア州を例に取ると、その時刻は以下のように経過しました。

| カリフォルニア州の時刻 | UTC からの差 (オフセット) |
| --- | --- |
| 2020-11-01 01:00:00 | 太平洋夏時間 (`PDT`) -07:00 |
| 2020-11-01 01:00:01 | 太平洋夏時間 (`PDT`) -07:00 |
| … | … |
| 2020-11-01 01:59:59 | 太平洋夏時間 (`PDT`) -07:00 |
| 2020-11-01 01:00:00 | 太平洋標準時 (`PST`) -08:00 |
| 2020-11-01 01:00:01 | 太平洋標準時 (`PST`) -08:00 |

つまり、カリフォルニア州には 2020年 11月 1日 午前 1時 30分という時刻が二回存在しました。

本記事で「タイムゾーンの呪い」と呼ぶ最大のものは、地域ベースのタイムゾーンを使うと、このような「存在しない時刻」や「二重に存在する時刻」の問題が常にセットでついてくる、というものです。ひとたび地域ベースのタイムゾーンを使ってしまったら、もう逃げることはできません。

### 夏時間よもやま話

夏時間には逸話がいろいろあるので、ここでいくつか紹介します。

2021年の日本では夏時間は採用されていませんが、[第二次世界大戦の直後に夏時間が実施されていたことがあります](https://ja.wikipedia.org/wiki/%E5%A4%8F%E6%99%82%E9%96%93#%E9%80%A3%E5%90%88%E5%9B%BD%E8%BB%8D%E5%8D%A0%E9%A0%98%E6%9C%9F)。

2021年現在は、どの国でも夏時間に切り替わってから一年以内に標準時に戻ります。しかし[第二次世界大戦中のイギリスでは、夏時間に切り替わったまま戻らず、翌年に二重に夏時間に切り替わった](https://ja.wikipedia.org/wiki/%E8%8B%B1%E5%9B%BD%E5%A4%8F%E6%99%82%E9%96%93#%E6%AD%B4%E5%8F%B2)ことがあります。

夏時間制を導入している国や地域でも、そのすべてでアメリカ合衆国のように安定した「切り替え規則」があるとは言いがたいようです。たとえば[チリでは 2016年前後に夏時間制が何度も変更されました](https://en.wikipedia.org/wiki/Time_in_Chile#Summer_time_(CLST/EASST))。[ブラジルでは 2008年まで夏時間の実施地域や切り替え日が毎年個別に公示されていて、その公示が夏時間開始の直前になったこともあるようです](https://en.wikipedia.org/wiki/Daylight_saving_time_in_Brazil)。

夏時間以外の切り替わり
-----------------------

夏時間以外にも、地域のタイムゾーンのオフセットが切り替わることがあります。その国や地域の政府によるその時々の政治的決定が、おもな理由です。

たとえば 2011年の年末には[サモア独立国](https://ja.wikipedia.org/wiki/%E3%82%B5%E3%83%A2%E3%82%A2)の標準時が `UTC-11:00` から `UTC+13:00` に切り替わりました。この切り替わりは 12月 29日の終わりがそのまま 12月 31日の始まりに続く形で実施され、サモアでは 2011年 12月 30日がまるまる存在しなかったことになりました。

タイムゾーンの呪いは、こんな風にも降りかかってきます。夏時間で 1時間くらい長くなったり短くなったりすることは考慮していても、日付が一日まるまるなくなるかもしれない、などという想定までは、したことがない人が多いのではないでしょうか。

夏時間制の終わり
-----------------

2021年現在、夏時間制の廃止が世界各地で議論されています。夏時間制の廃止はソフトウェア・エンジニアとして長期的には歓迎できることですが、今まで実施されていたものが実施されなくなるというのは、そんなに単純な話ではありません。状況を注視する必要があるでしょう。

たとえば EU における夏時間制は、欧州議会の議決では 2021年を最後に廃止されることになっていました。 [^eu-summer-time-2019] しかし Brexit やコロナ禍もあって、その 2021年の 3月になってもまだ状況は不透明だったようです。 [^eu-summer-time-2021-asahi] [^eu-summer-time-2021-bloomberg]

[^eu-summer-time-2019]: [「欧州議会、サマータイム廃止の法案を可決　２０２１年に」 (2019年 3月 28日、朝日新聞)](https://digital.asahi.com/articles/ASM3W0014M3VUHBI02R.html)
[^eu-summer-time-2021-asahi]: [「これで最後？EUが夏時間へ　廃止検討するも議論進まず」 (2021年 3月 27日、朝日新聞)](https://digital.asahi.com/articles/ASP3W4HWJP3VUHBI04L.html)
[^eu-summer-time-2021-bloomberg]: ["Why Europe Couldn’t Stop Daylight Saving Time" (March 11, 2021, Bloomberg)](https://www.bloomberg.com/news/articles/2021-03-11/will-daylight-saving-time-ever-end)

EU の状況をさらにややこしくしているのは、夏時間廃止後の新しい標準時を「旧来の標準時」に戻すのか「それまでの夏時間」に新しく合わせるのか、最終的には各国にまかされているらしい、ということです。なるべくそろえたい思惑はさすがにあるようですが、最終的にどうなるか、どうやら 2021年 6月現在もわかっていません。極端にいえば、いままで同じ「[中央ヨーロッパ標準時 (CET)](https://ja.wikipedia.org/wiki/%E4%B8%AD%E5%A4%AE%E3%83%A8%E3%83%BC%E3%83%AD%E3%83%83%E3%83%91%E6%99%82%E9%96%93)」を使っていて時差のなかったドイツ・フランス・イタリアの間に、来年からは時差が生じる、なんてことがあるかもしれません。タイムゾーンの呪いは、意識していなかったところから襲ってきます。恐ろしい話です。 [^italy-2021]

[^italy-2021]: [「サマータイム廃止でイタリアは最後の夏時間」 (2021年 3月 21日、ニューズウィーク日本版内 World Voice)](https://www.newsweekjapan.jp/worldvoice/vismoglie/2021/03/post-23.php)

また、カリフォルニア州では 2018年の住民投票で夏時間の変更が支持されました。 [^california-proposition-7] 2021年 6月時点ではまだ具体的な変更の予定はないようですが、これから変わるかもしれません。

[^california-proposition-7]: [「カリフォルニア州、住民は夏時間の変更を支持」 (2018年11月19日、日本貿易振興機構)](https://www.jetro.go.jp/biznews/2018/11/f4ab9860d030abf0.html)

カリフォルニア州の変更で要注意なのは、「ロサンゼルス時間」を「[太平洋時間](https://ja.wikipedia.org/wiki/%E5%A4%AA%E5%B9%B3%E6%B4%8B%E6%99%82%E9%96%93)」の代表として用いているソフトウェアがある、ということでしょう。後述しますが、実は Java の標準 API がそうです。たとえばワシントン州シアトルの冬の時刻を表現するつもりで「太平洋標準時 (`PST`)」を使っていると、それが内部的にはロサンゼルス時間あつかいになっているわけです。そんな中でカリフォルニア州のみ夏時間が廃止されたら…、どうするんでしょうね。タイムゾーンの呪いには、なにか恐ろしいことが起きそうだとはわかっていても、各国や地域の政治的決定が定まるまでどうなるかわからない、という面があります。考えたくない話です。

ソフトウェア技術における時刻
=============================

さて、ここまで時刻とタイムゾーンの一般的なしくみをおもに紹介してきました。ここからは、ソフトウェアや通信の分野に特有の技術的な内容を取り上げていきます。

Unix time: 世界共通の時間軸
----------------------------

世界各地の「時刻」には時差がありますが、それでも時間は世界中で同じように流れており、時間軸は世界共通です。その時間軸上の一点は、世界中で共通の同じ一瞬です。 [^relativity-physics] たとえば、日本の 2020年 7月 1日 午後 3時 0分 0秒と、イギリスの 2020年 7月 1日 午前 7時 0分 0秒は、その時間軸上では同じ一点です。各国や地域の中ではそれぞれの現地時刻を使っていれば問題ありませんが、タイムゾーンをまたぐ場合、特にインターネットが関わるしくみの中では、世界共通の時間軸で時刻をあつかいたくなるというものです。

[^relativity-physics]: 相対論的なツッコミは (ry

前述したように UTC という共通の基準があるので、要はその時間軸を UTC で表現すればいいのです。ですが、その時間軸上で年・月・日・時・分・秒の繰り上げなどいちいち計算するのは無駄が多いです。そこでソフトウェアの分野では、よく [Unix time](https://ja.wikipedia.org/wiki/UNIX%E6%99%82%E9%96%93) を使って時刻を表現します。

Unix time は、日本語では「Unix 時間」、英語では "POSIX time" や "(Unix) epoch time" などとも呼ばれ、時刻を UTC の 1970年 1月 1日 0時 0分 0秒からの経過秒数で表現するものです。この UTC の 1970年 1月 1日 0時 0分 0秒を "epoch" と呼びます。

Unix time がわかっていれば、任意のタイムゾーンにおける年・月・日・時・分・秒は簡単に計算できるはず…ですが、一つ問題が残ります。うるう秒です。うるう秒はそもそも予測困難な自転速度のふらつきを埋めるためのしくみなので、うるう秒が導入されるタイミングを将来にわたって確定することはできません。これでは、未来の時刻に対応する Unix time が計算できません。

そこで Unix time は「経過時間」としての厳密さを捨てて利便性を優先し、うるう秒を無視することになっています。[最近では UTC における 2016年 12月 31日 23時 59分 59秒と 2017 年 1月 1日 0時 0分 0秒の間に正のうるう秒が挿入されました](https://jjy.nict.go.jp/QandA/data/leapsec.html)が、このとき Unix time の `1483228800` は 2秒間続きました。その経過を 0.5秒ごとに並べると、次のようになります。

| UTC における時刻 | 小数部を含む Unix time |
| --- | --- |
| 2016-12-31 23:59:59.00 | 1483228799.00 |
| 2016-12-31 23:59:59.50 | 1483228799.50 |
| 2016-12-31 23:59:60.00 | 1483228800.00 |
| 2016-12-31 23:59:60.50 | 1483228800.50 |
| 2017-01-01 00:00:00.00 | 1483228800.00 |
| 2017-01-01 00:00:00.50 | 1483228800.50 |
| 2017-01-01 00:00:01.00 | 1483228801.00 |

Unix time の整数部だけを見ると同じ秒が 2秒間続くだけなので、そんなに問題ないように見えます。しかし、小数部がありながら小数部が普段どおりにカウントされると、時刻が逆進したことになるので注意が必要です。 [^fraction-in-unix-time]

[^fraction-in-unix-time]: ちなみに歴史的には「Unix time という言葉は秒単位 (秒精度) の整数までしか指さない」という説もあります。それにのっとると「Unix time の小数部」というのはちょっとおかしいのですが、現代の [POSIX の `clock_gettime`](https://pubs.opengroup.org/onlinepubs/9699919799/functions/clock_gettime.html) は秒またはナノ秒単位ということなので、本記事の Unix time は小数部を持つ数も指す定義にしています。

うるう秒の「希釈」
-------------------

せっかく UTC で導入されたうるう秒ですが、すべてのソフトウェアで「一秒単位の厳密な時刻」が必要なわけではありません。逆進問題も含め、うるう秒は過去に多くの問題を引き起こしてもきました。 [^incidents-by-leap-seconds]

[^incidents-by-leap-seconds]: [『「うるう秒」障害がネットで頻発』 (2012年 7月 2日、 WIRED NEWS)](https://wired.jp/2012/07/02/leap-second-bug-wreaks-havoc-with-java-linux/)

そこで、必要なさそうな場合は UTC で獲得した一秒の厳密さを再度捨てて、うるう秒を周辺の数時間で「希釈」する手法が 2000年ごろの議論 [^uts-history] から生まれました。これは近年のソフトウェアやクラウドサービスで一般的な手法になりつつありますが、この「希釈」にはいくつか流派があります。

[^uts-history]: そのような手法が公に議論されたのは、[ケンブリッジ大学の Markus Kuhn 氏による 2000 年の UTS (Smoothed Coordinated Universal Time)](https://www.cl.cam.ac.uk/~mgk25/uts.txt) が最初のようです。

たとえば Java の JSR 310 で使われる「[Java タイム・スケール](https://docs.oracle.com/javase/jp/8/docs/api/java/time/Instant.html)」は、最初の議論から派生して生まれた [UTC-SLS (Coordinated Universal Time with Smoothed Leap Seconds)](https://www.cl.cam.ac.uk/~mgk25/time/utc-sls/) [^utc-sls-ietf-draft] をベースにしています。 UTC-SLS は、うるう秒が導入される日の最後の 1000秒に均等にうるう秒を分散させて希釈し、うるう秒を見えなくしてしまうものです。つまり、正のうるう秒による Java タイム・スケールの希釈期間では、表面上の 1秒が経過するのに実際には 1.001秒かかる、ということですね。

[^utc-sls-ietf-draft]: 2006年には [IETF Internet Draft](https://tools.ietf.org/html/draft-kuhn-leapsecond-00) としても公開されています。

一方 AWS や Google Cloud などのクラウドサービスでは、少し別の流派が一般的になりつつあります。彼らの手法は "[Leap Smear](https://docs.ntpsec.org/latest/leapsmear.html)" [^smear] と呼ばれることが多く、おもに NTP を介して提供されます。

[^smear]: "Smear" は「なすりつける」「こすって不鮮明にする」のような意味です。

[2008年のうるう秒で Google が実施した最初の Leap Smear](https://googleblog.blogspot.com/2011/09/time-technology-and-leaping-seconds.html) は、うるう秒をその「前」の「20時間」に「非線形に」分散させるものでした。次の [2012年から 2016年のうるう秒における Google の Leap Smear](https://cloudplatform.googleblog.com/2015/05/Got-a-second-A-leap-second-that-is-Be-ready-for-June-30th.html) は、うるう秒の「前後」の「20時間」に「線形に」分散させるものになりました。

現在では [Google はうるう秒の「前後」の「24時間」に「線形に」分散させるやりかたを標準とすることを提案しています](https://developers.google.com/time/smear)。 [2015年と 2016年に Amazon が実施した Leap Smear](https://aws.amazon.com/jp/blogs/aws/look-before-you-leap-the-coming-leap-second-and-aws/) もこれと同様のもので、クラウドサービスではこのやりかたが一般的になると思ってよさそうな雰囲気です。

tzdb: タイムゾーン表現の業界標準
---------------------------------

ソフトウェアの分野でタイムゾーンをあらわす最も業界標準的なデータが [Time Zone Database](https://www.iana.org/time-zones) [^tzdb-wikipedia] でしょう。見たことのある方も多いであろう `Asia/Tokyo` や `Europe/London` のような ID は、この Time Zone Database のものです。

[^tzdb-wikipedia]: [Wikipedia:tz database](https://ja.wikipedia.org/wiki/Tz_database)

もともと "tz database" や、略して "tzdb" と、または単に "tz" などと呼ばれていました。本記事中ではおもに "tzdb" と呼びます。 Unix-like な OS で使われるパス名から "zoneinfo" とも呼ばれます。 Arthur David Olson たちが始めたもので、遅くとも 1986年にはメンテナンスされていた記録があります。 [^olson-history] Olson の名前から "Olson database" と呼ばれることもあります。

[^olson-history]: ["seismo!elsie!tz ; new versions of time zone stuff" (Nov 25, 1986)](http://mm.icann.org/pipermail/tz/1986-November/008946.html)

tzdb の ID は、最初の `/` の前に大陸・海洋名を、後ろにそのタイムゾーンを代表する都市名・島名などを用いてつけられます。 [^cities-in-tzdb] 国名は基本的に使われません。 [^countries-in-tzdb] `America/Indiana/Indianapolis` のように 3要素で構成されるタイムゾーンも、少数ながら存在します。

[^cities-in-tzdb]: 日本語でも「東京時間で」「ニューヨーク時間で」などというのと同じような感覚だと思います。

[^countries-in-tzdb]: 国が関係する状態は変わりやすいので、[政治的事情による変更に対して頑健であるためだとする記載があります](https://github.com/eggert/tz/blob/2021a/theory.html#L143-L151)。人が生活する単位としての「都市」はそれよりは長く維持される傾向にあると考えられているようです。要出典。ちなみに[初期の提案](https://mm.icann.org/pipermail/tz/1993-October/009233.html)では、国名を使うつもりがあったようです。

tzdb の管理は 2011年に ICANN の IANA に移管されました [^iana-tzdb] が、いまでもメンテナンスは個人のボランティアをベースにおこなわれています。 [^eggert-links] タイムゾーンは世界のどこかで意外なほど頻繁に変わっており、年に数回は新しいバージョンの tzdb がリリースされます。前述のサモアのように政治的決断で変わる場合、夏時間制を新しく導入する場合・廃止する場合、国や地域の境界が変わる場合など、さまざまなケースがあります。登録済みの過去のデータに間違いが見つかって修正することもあります。 [^south-ryukyu-island] tzdb の更新は実際にタイムゾーンが変わる直前になることも多く、前述のサモア標準時の変更が tzdb に適用されたのは、[実際に標準時が変わる 2011年 12月のわずか 4ヶ月前のこと](http://mm.icann.org/pipermail/tz/2011-August/008703.html)でした。

[^iana-tzdb]: ["ICANN to Manage Time Zone Database" (October 14, 2011)](https://itp.cdn.icann.org/en/files/announcements/release-14oct11-en.pdf)
[^eggert-links]: ちなみに、現在 tzdb データのメンテナンスの中心人物である [Paul Eggert 氏のページ](https://web.cs.ucla.edu/~eggert/tz/tz-link.htm)には、多くの重要な情報がまとまっています。
[^south-ryukyu-island]: たとえば `Asia/Tokyo` でも、古い BSD ユーザーにはおなじみの [South Ryukyu Islands 時間問題](https://ja.wikipedia.org/wiki/%E6%97%A5%E6%9C%AC%E6%A8%99%E6%BA%96%E6%99%82#South_Ryukyu_Islands%E6%99%82%E9%96%93) がありました。その [South Ryukyu Islands 時間問題に関する調査報告](http://www.tomo.gr.jp/root/9925.html)などもあります。比較的最近でも、[第二次世界大戦中の情報について一部修正されたり](http://mm.icann.org/pipermail/tz/2018-January/025896.html)しています。

tzdb は、特に 1970年 1月 1日以降のすべてのタイムゾーンの変更、夏時間などのオフセット切り替え規則、うるう秒の情報まで、さまざまな情報を包含することを目標にメンテナンスが続けられています。 tzdb は OS や Java などの実行環境、各言語のライブラリなどに組み込まれていて、更新が出るとそれぞれのメンテナによって取り込まれ、更新を組み込んだアップデートとしてそれぞれ世界中に配信されています。

tzdb には 1970年 1月 1日以前の情報もある程度まで含まれていますが、暦の違い、歴史資料の問題などもあって、古い情報を正確なものにしようとはあまり考えられていないようです。 [^tzdb-before-1970] 特にタイムゾーン ID は「1970年以降も区別する必要のあるタイムゾーン」のみを定義する、という方針のようです。たとえばブラジルのサンパウロを含む一部地域で 1963年に実施された夏時間をあらわすために [`America/Sao_Paulo` の新設が提案された](https://mm.icann.org/pipermail/tz/2010-January/016007.html)ことがありますが、[この方針により却下されています](https://mm.icann.org/pipermail/tz/2010-January/016010.html)。

[^tzdb-before-1970]: 古い歴史には解釈にも揺れがあるため、「正確なもの」はそもそも定義できない、とも考えられます。

Military time zones
--------------------

`Z` の一文字で `UTC` をあらわす記法を見たことがある人は多いと思います。これは、おもに米軍で使われる "[Military time zones](https://en.wikipedia.org/wiki/List_of_military_time_zones)" から来ています。 `UTC-12` から `UTC+12` まで一時間単位の 25 のタイムゾーンに、大文字のアルファベット (`J`を除く) を対応させます。 `UTC+05:30` のような一時間単位ではないタイムゾーンには `*` (star) をつけて `E*` ("Echo-Star") などと呼ぶこともあるようです。 [^echo-star]

[^echo-star]: [Wikipedia (en): Indian Standard Time](https://en.wikipedia.org/wiki/Indian_Standard_Time)

Military time zones は [RFC 822](https://datatracker.ietf.org/doc/html/rfc822) や [RFC 2822](https://datatracker.ietf.org/doc/html/rfc2822) などにも記載があり、これを標準で処理できる言語やライブラリは多くあります。しかし `Z` 以外はあまり使わないほうがいいでしょう。

その第一の理由は、初期の RFC 822 の Military time zones の記述に誤りがあったことです。その誤りをもとにおかしな処理をしていたソフトウェアが歴史的にいくつかありました。誤りは RFC 2822 で修正されましたが、互換性の問題で、実装は昔のままになっていたりします。さらに `UTC+13` のように Military time zones ではあつかえないタイムゾーンもあります。 `Z` 以外はあまり知られてもおらず、わざわざ使う理由はあまりないように思います。

タイムゾーン略称
-----------------

### JST: Jerusalem Standard Time?

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

とはいえ tzdb 公式の解釈としては、以下のコメントのように ["Jerusalem Standard Time" としての `JST` は除外 (ruled out) されて](https://github.com/eggert/tz/blob/2021a/asia#L1715-L1720)はいます。

```
# Since timezones should be called by country and not capital cities,
# I ruled out JST.  As Israel is in Asia Minor and not Eastern Europe,
# EEST was equally unacceptable.  Since "zonal" was not compatible with
# any other timezone abbreviation, I felt that 'IST' was the way to go
# and, indeed, it has received almost universal acceptance in timezone
# settings in Israeli computers.
```

それでも "Jerusalem Standard Time" という呼びかた自体は各所で現役です。 [^jerusalem-standard-time] [GE Digital のドキュメント](https://www.ge.com/digital/documentation/meridium/Help/V43070/r_apm_pla_valid_time_zones.html)に含まれていたり [Windows のタイムゾーン情報](https://support.microsoft.com/en-us/topic/israel-and-libya-time-zone-update-for-windows-operating-systems-7c6a08aa-8c56-4d7a-2aa8-b956602ebf0a)に含まれていたりします。 "Jerusalem Standard Time" という呼びかたが現に存在するということは、略して `JST` と呼ぶ人々も、当地にはもちろんいるでしょう。名前の衝突は、特にソフトウェア間のやり取りではとても厄介ですね。

[^jerusalem-standard-time]: もちろん、これを "Israel Standard Time" などと呼び替えるのがきわめて難しいのは想像に難くないでしょう。 tzdb が ID に国名を使わない方針にしたのと同じ理由で、都市名で呼ぶのが現実的だということではないでしょうか。

### タイムゾーン略称は非推奨

`JST` のようなタイムゾーン略称は一見使いやすいのですが、このように、実は略称から目当てのタイムゾーンを一意に特定できるとはかぎりません。プログラムの記述や、ソフトウェアに読み込ませる可能性のあるデータに、タイムゾーン略称を用いるのは避けるべきでしょう。手元でしばらくは動いていたのに他の国や地域の顧客がついたらデータがおかしくなり始めた、みたいな地獄が待っています。

実際、旧 Java 標準 API の [`java.util.TimeZone`](https://docs.oracle.com/javase/jp/8/docs/api/java/util/TimeZone.html) では、「3文字のタイムゾーン ID は非推奨」と明記されています。新 JSR 310 の [`java.time.ZoneId`](https://docs.oracle.com/javase/jp/8/docs/api/java/time/ZoneId.html) では、略称は標準ではサポートされず、[略称とタイムゾーンの対応を別にエイリアスとして指定するしくみ](https://docs.oracle.com/javase/jp/8/docs/api/java/time/ZoneId.html#of-java.lang.String-java.util.Map-)になっています。

不幸にもタイムゾーン略称を含むデータを読み込まなければならない場合は、あきらめて決め打ちするしかありません。データ処理プロセスのできるだけ初期のステージのうちに対処しましょう。さらに、各略称をどのタイムゾーンとして読み込むのか、そのソフトウェアやサービスの仕様として明文化しておきましょう。 [^abbreviation-examples] さらに Java などの処理系やライブラリが暗黙のうちに略称を解釈してしまうことがあるため、その仕様まですべて把握し、開発中のソフトウェアやサービス自身の仕様としても改めて明記しましょう。

[^abbreviation-examples]: たとえば「`JST` は `+09:00` として読み込む」など。

開発中のソフトウェアやサービスからタイムゾーン略称入りのデータが出力されるようになっていたら、できるかぎり即座にやめましょう。代わりに `+09:00` などのオフセットや、地域ベースの `Asia/Tokyo` などの tzdb ID を使いましょう。オフセットと地域ベースのどちらがいいのかはこのあと「実装編」などで検討しますが、いずれにせよ略称はダメです。互換性などの理由でどうしても略称をやめられない場合は、これもすべて仕様として明文化しましょう。

タイムゾーン略称を使うのは、人間対人間の文脈が明らかなコミュニケーションのみにしておきましょう。タイムゾーン略称が引き起こす問題を、もういくつか紹介します。

### CST

`JST` よりわかりやすく略称が衝突している例が `CST` です。 [`java.util.TimeZone` の Javadoc でも明示的に触れられている](https://docs.oracle.com/javase/jp/8/docs/api/java/util/TimeZone.html)ほどです。

日本やアメリカ合衆国でソフトウェア・エンジニアとして働いていると `CST` を「[中部標準時 (Central Standard Time)](https://ja.wikipedia.org/wiki/%E4%B8%AD%E9%83%A8%E6%A8%99%E6%BA%96%E6%99%82)」のことと思ってしまいがちです。しかし「[中国標準時 (China Standard Time, Chinese Standard Time)](https://ja.wikipedia.org/wiki/%E4%B8%AD%E5%9B%BD%E6%A8%99%E6%BA%96%E6%99%82)」も `CST` です。さらに、「[キューバ](https://ja.wikipedia.org/wiki/%E3%82%AD%E3%83%A5%E3%83%BC%E3%83%90)標準時 (Cuba Standard Time)」 も `CST` です。 [^relationship-with-usa]

[^relationship-with-usa]: アメリカ合衆国とはややこしい関係にある国ばかりですが、偶然か必然か。

`CST`: "China Standard Time" は [tzdb にも現役で収録されています](https://github.com/eggert/tz/blob/2021a/asia#L40-L55)。 `JST`: "Jerusalem Standard Time" のように除外されているわけでも、非推奨にもなっているわけでもありません。

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

中国標準時の [`Asia/Shanghai` の定義](https://github.com/eggert/tz/blob/2021a/asia#L668-L672)が以下です。最後の行の `[UNTIL]` 列が空欄なので、これが現在でも有効ということですね。 (`%s` は `J%sT` と同様です。)

```
# Zone  NAME            STDOFF  RULES   FORMAT  [UNTIL]
# Beijing time, used throughout China; represented by Shanghai.
Zone    Asia/Shanghai   8:05:43 -       LMT     1901
                        8:00    Shang   C%sT    1949 May 28
                        8:00    PRC     C%sT
```

キューバ標準時の [`America/Havana` も同様に定義](https://github.com/eggert/tz/blob/2021a/northamerica#L3373-L3376)されています。

```
# Zone  NAME            STDOFF  RULES   FORMAT  [UNTIL]
Zone    America/Havana  -5:29:28 -      LMT     1890
                        -5:29:36 -      HMT     1925 Jul 19 12:00 # Havana MT
                        -5:00   Cuba    C%sT
```

### EST, EDT, CST, CDT, MST, MDT, PST, PDT

`CST` の衝突について紹介しましたが、その `CST` を含むアメリカ合衆国のタイムゾーン略称を、優先的に処理するようになっている言語やライブラリがいくつかあります。 [Ruby の `Time`](https://github.com/ruby/ruby/blob/v3_0_1/lib/time.rb#L40-L43) や [Java の旧 `java.util.TimeZone`](https://docs.oracle.com/javase/jp/8/docs/api/java/util/TimeZone.html) がその例です。これは [RFC 822](https://datatracker.ietf.org/doc/html/rfc822) や [RFC 2822](https://datatracker.ietf.org/doc/html/rfc2822) にこれらの略称が明記されていることが、おおもとの理由としてあるようです。

しかしこの略称のあつかいがソフトウェアによって微妙に異なるのです。上で例に挙げた Ruby と Java でも異なり、実は Java のほうが「間違っている」と言ってよさそうなものなのですが、互換性などの理由でいまでも部分的に引きずっています。いままではそれを「仕様」としてどうにかやってきましたが、近年さかんに議論されている夏時間制の廃止を引き金として、問題として表面化するかもしれません。このことについて以下で紹介します。

#### 略称の解釈による問題: Java の例

これらの略称は本来、それぞれ標準時・夏時間のどちらか一方に対応します。たとえば `PST` はあくまで Pacific "Standard Time" (太平洋「標準時」) であって `PST` が太平洋夏時間を指すことはありません。太平洋「夏時間」は、あくまで Pacific "Daylight Saving Time" の `PDT` です。

つまり `PST` があらわすのは常に `-08:00` であり、あくまで「夏になるとカリフォルニアの時間が `PST` (`-08:00`) から `PDT` (`-07:00`) に切り替わる」のであって、「夏になると `PST` が `-07:00` になる」わけではありません。上記の [RFC 2822 にも `PST is semantically equivalent to -0800` などと明記されています](https://datatracker.ietf.org/doc/html/rfc2822#section-4.3)。

[Ruby の `Time` はこれに沿った実装をしています](https://github.com/ruby/ruby/blob/v3_0_1/lib/time.rb#L40-L43)。 Ruby の `Time` は `PST` を常に `-8` としてあつかい、そして `PDT` を常に `-7` としてあつかいます。これが正しい解釈です。

しかし古くからの Java 標準 API (とそれを継承した [Joda-Time](https://www.joda.org/joda-time/)) は、これを違うあつかいにしてしまいました。 `PST` などの略称を `America/Los_Angeles` などの地域ベースのタイムゾーン ID に対応づけてしまったのです。一見するとこれでもよさそうですが、問題は何でしょうか。

カリフォルニアでは夏時間となる `2020-07-01 12-34-56` に `PST` を組み合わせた文字列 `2020-07-01 12:34:56 PST` は、本来 `2020-07-01 12:34:56 -08:00` として解釈しなければなりません。しかし、古くからの Java 標準 API はこの `PST` を `America/Los_Angeles` に対応させるので、これは `2020-07-01 12:34:56 America/Los_Angeles` になります。するとこれは、カリフォルニアでは `2020-07-01` が夏時間であることから、最終的には`2020-07-01 12:34:56 -07:00` と解釈されてしまいます。

「2020年 7月 1日が夏時間なのは明らかなんだから、そもそも `2020-07-01 12:34:56 PST` なんて書くのが悪いじゃないか」と思うかもしれません。…しかし、本当にそれだけでしょうか?

同じことは `MST` (山岳部標準時) でも起こります。古い Java API は `MST` を `America/Denver` に対応づけます。 Denver があるコロラド州には夏時間があり、冬に `-07:00` だった `America/Denver` は、夏には `-06:00` になります。これは `PST` と同じ状況で、上と同様の「`2020-07-01 12:34:56 MST` なんて書くのが悪い」という主張は、たしかに Denver では正しいです。

同じ「山岳部時間」を使う州にアリゾナ州があります。上の「タイムゾーンの定義と用法」で触れましたが、アリゾナ州は一部を除いて夏時間を採用していません。ということは `2020-07-01` に `MST` を組み合わせた `2020-07-01 12:34:56 MST` は、一部を除くアリゾナ州では有効なはずの表現であり、これは本来 `2020-07-01 12:34:56 -07:00` と解釈しなければなりません。しかし Java API はこれを `2020-07-01 12:34:56 -06:00` と解釈してしまいます。

このことは、以下の Java コードで容易に確認できます。 [^java-oldmapping]

[^java-oldmapping]: このサンプル・コードは、古い Java API ではなく新しい JSR 310 の `DateTimeFormatter` を使っていますが、同様の挙動を見せています。この Java とタイムゾーン略称の歴史をさかのぼると、実は旧 API で一度は修正されています。しかし JSR 310 の `DateTimeFormatter` でも一部機能でこの挙動が再導入されていて、このサンプル・コードはその例になっています。実はもう少し深堀りしてみたのですが長大な内容になったので、その話は[「Java 編」](./curse-of-timezones-java-ja)の「おまけ」として分離しました。興味のある方はそちらをご覧ください。

```java
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

この出力は以下のようになります。 (ここでは Java 8 で確認していますが OpenJDK 16.0.1 でも同様の結果になることを確認しています。)

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

ここで、「カリフォルニア州では 2018年の住民投票で夏時間制の廃止が支持された」という話があったのを思い出してみましょう。

`MST` に対応する `America/Denver` の場合、この Denver は夏時間を採用する多数派で、夏時間を採用しないアリゾナ州は少数派の例外でした。一方で、夏時間制を廃止する議論が進んでいるのはいまのところカリフォルニア州くらいです。夏時間を採用しないほうが圧倒的に少数派なのです。そして `PST` に対応する `America/Los_Angeles` はカリフォルニア州です。将来 `PST` は少数派の例外のほうに対応づいてしまう可能性があります。

タイムゾーン略称については、これからもなにが起こるのかよくわかりません。よく使われる `PST` ですらこんな状態です。「タイムゾーン略称はこわい」と多くの方に思ってもらえたら、この章を書いたかいがあります。

発展: 日付と暦
===============

さて、ここまで地球の自転にともなう周期 (一日・時刻) についておもに見てきました。では地球の公転にともなう周期 (一年・日付) はどうでしょうか。

太陰暦や日本の元号などの国・地域固有の暦という文化こそ一部にありますが、現在では「グレゴリオ暦」がいわゆる「西暦」として普及し、少なくとも技術的には共通言語として成立するようになりました。このグレゴリオ暦を使ってここ数十年の日付のみをあつかっているかぎりは、暦で悲惨な思いをすることは少ないでしょう。

しかしひとたび昔の日付をあつかおうとすると、暦の話もまた、時刻とタイムゾーンのように広大な闇が広がっています。グレゴリオ暦以前の暦である「ユリウス暦」や、グレゴリオ暦をグレゴリオ暦以前にまで強引に適用した「先発グレゴリオ暦」など、さまざまな種類の暦が出てきて、ソフトウェア・エンジニアを混乱の渦に突き落とします。

とはいえ、本記事内で暦の問題まであつかおうとすると、記事が倍以上の分量になってしまいそうです。ここでは、とてもよくまとまったブログ記事を紹介して、その代わりとしたいと思います。

[「西暦1年は閏年か？」 (2020年 10月 30日、なぎせゆうきさんのブログ記事)](https://nagise.hatenablog.jp/entry/2020/10/30/173911)

そして実装へ
=============

時刻とタイムゾーンというこの厄介な概念について話を続けると、きりがありません。とはいえ一般的な知識は、ある程度ここまでで概観できたと思います。

では、この知識を具体的に実装に落とし込むときには、何にどう気をつけたらいいでしょうか。

[Qiita に載せていた 2018年版の旧記事](https://qiita.com/dmikurube/items/15899ec9de643e91497c)では、ここから Java を具体例として実装について紹介していました。さらに [Software Design 誌の 2018年 12月号](https://gihyo.jp/magazine/SD/archive/2018/201812)では Java に限定しない実装の一般論も書きました。本記事ではその Software Design 誌の内容を取り込んで、さらに Java 特有のトピックにも追加をした結果、記事全体の分量が大幅に増えてしまいました。そこで、実装の一般論と Java 特有の話は別の記事に分けることにしました。

* [タイムゾーン呪いの書 (実装編)](./curse-of-timezones-impl-ja)
* [タイムゾーン呪いの書 (Java 編)](./curse-of-timezones-java-ja)

ここでは同筆者による実装編と Java 編のみにリンクを張っていますが、他の方による、他の言語やソフトウェアなどの話をまとめた記事を教えていただけたら、こちらで紹介させていただくかもしれません。 [^other-articles]

[^other-articles]: Ruby (特に Ruby on Rails) の話や JavaScript (`temporal` とか) をふくむ Web ブラウザ環境の話、あと MySQL や PostgreSQL と JDBC の話など、深いところを読んでみたいトピックはいろいろあります。筆者も Ruby の話 (Rails 以外) と Linux の話を少々くらいは書けるかもしれませんが、本当に詳しい方におまかせしたいなあ、と思っています。

おまけ: 環境変数 TZ
====================

(特に Unix-like な) OS のタイムゾーン設定といえば環境変数 `TZ` でした。 [^env-tz-now] もともと tzdb 自体が、環境変数 `TZ` の裏方でもある `zoneinfo` の元データとしてメンテナンスされたものです。

[^env-tz-now]: いまでも違うわけではありませんが。

ここで「`TZ` の設定が、どのようなメカニズムで実際の動作に反映されるのか」とかの解説を始めると、長文記事がもう一本できてしまうので、本記事では割愛します。 [^unix-chapter] しかし `TZ` の「書式」はいろんなところで顔を出すので、この「知識編」で軽く触れておいたほうがいいと判断し、おまけとして追記することにしました。

[^unix-chapter]: どなたか「Unix(-like) 編」を書いて、ぜひ歴史なども含めてじっくり解説してください。

UTC+9? JST-9?
--------------

日本時間は UTC から 9時間先行していますが、これを一般に `UTC+9` や `UTC+09:00` などと表記するのは前述したとおりです。一方で、環境変数 `TZ` に `JST-9` などと設定したことがある方は多いのではないでしょうか。

符号が `+` と `-` で逆転していて、混乱しますよね。

この `TZ` の書式はいわゆる歴史的事情 [^unix-chapter] というやつです。歴史上の経緯まで紐解くのはたいへんなので踏み込みませんが、この書式を一般化したものは、現在 [POSIX](https://ja.wikipedia.org/wiki/POSIX) の一部として明記 [^posix-v1-chapter-8-1] されています。さらに glibc のマニュアルにも解説 [^glibc-21-5-6] があります。

[^posix-v1-chapter-8-1]: [8. Environment Variables, The Open Group Base Specifications Issue 7, 2018 edition.](https://pubs.opengroup.org/onlinepubs/9699919799/basedefs/V1_chap08.html)

[^glibc-21-5-6]: [21.5.6 Specifying the Time Zone with TZ, The GNU C Library](https://www.gnu.org/software/libc/manual/html_node/TZ-Variable.html#TZ-Variable)

POSIX からその書式を引用します。 (空白文字はわかりやすさのための筆者による追記)

```
std offset [dst [offset] [,start[/time],end[/time]]]
```

必須項目は `std offset` ですね。 `JST-9` の `JST` の部分が `std` にあたり、次の `-9` の部分が `offset` にあたります。 `JST` は忌まわしきタイムゾーン略称ですが、これは歴史の経緯でいかんともしがたいところです。そして `offset` は以下のように解説されています。

> Indicates the value added to the local time to arrive at Coordinated Universal Time.

「現地時刻にその値を足すことで UTC になる値」ですね。日本時間は UTC より 9時間先行するので、「日本時間に負の値 `-9` を足すことで UTC になる」わけです。身も蓋もない話ですが、日本時間が `JST-9` のように `-` になるのは、「定義からそういうもの」ということでした。

PST8PDT
--------

ここで `TZ` の書式をもう一度見てみましょう。

```
std offset [dst [offset] [,start[/time],end[/time]]]
```

日本時間は `JST-9` とシンプルですが、一般化した書式はなんだか長いですよね。必須ではない `[...]` の項目がいくつもあります。この `dst` と、二つ目の `offset` はなんでしょうか。

アメリカ合衆国のタイムゾーン略称で、単なる `PST` や `EDT` ではない `PST8PDT` や `EST5EDT` という表記を見たことがある人もいるかもしれません。これこそが `std offset [dst]` まで使った表記です。たとえば `PST8PDT` は「標準時は `PST` で `8` 時間足すと UTC になる。そして夏時間は `PDT` である」を意味する `TZ` の表記だったのですね。

この書式はアメリカ合衆国のタイムゾーン以外でも有効で、たとえばイギリスで `GMT0BST` や、フランス・ドイツで `CET-1CEST` と書いたりもできます。日本に夏時間が導入されたら `JST-9JDT` とかになります。ただ、アメリカ合衆国の `PST8PDT`, `MST7MDT`, `CST6CDT`, `EST5EDT` だけは一部で特別あつかいをされていて、たとえば[対応する項目が tzdb 中に直接書いてあったり](https://github.com/eggert/tz/blob/2021a/northamerica#L203-L206)もします。

```
Zone    EST5EDT                  -5:00  US      E%sT
Zone    CST6CDT                  -6:00  US      C%sT
Zone    MST7MDT                  -7:00  US      M%sT
Zone    PST8PDT                  -8:00  US      P%sT
```

二つ目の `offset` が省略されていると、「夏時間は標準時より 1時間先行する」というのがデフォルトの解釈になります。ほとんどの国や地域の夏時間制では、夏時間は標準時より 1時間先行するので、二つ目の `offset` を書くことはほぼありません。希少な例外である Lord Howe 島では、どうやら `LHST-10:30LHDT-11` などと書くようです。

それ以降の `[,start[/time],end[/time]]]` を直接書く機会は、ほぼないでしょう。興味がある方は POSIX [^posix-v1-chapter-8-1] や glibc のマニュアル [^glibc-21-5-6] を読んでみてください。

TZ=Asia/Tokyo
--------------

昔から Unix-like な環境をさわっている人としては、つい「`TZ` に設定するのは `JST-9` だ」という気がしてしまうのですが、少なくとも現在では `TZ=Asia/Tokyo` などと tzdb ID を直接 `TZ` に設定できるし、そうすることが多いようです。 [^tzdb-id-in-tz]

[^tzdb-id-in-tz]: `TZ=Asia/Tokyo` って、実は昔からできたんでしょうか? 筆者が Unix-like な環境をさわり始めた 2000年前後には `TZ=JST-9` とするのが一般的だった気がしていて、いつごろを境に実装や慣習が変わっていったのか、筆者は把握できていません。情報をお持ちの方がいらっしゃいましたら、ぜひお寄せください。 (というか「Unix 編」を…)

そもそもタイムゾーン略称を使わないほうがいいこと (前述) や、この表記にあまり汎用性がないこと [^tz-universality] を考えると、現在では `TZ` にも tzdb ID を使うのがいいのではないでしょうか。 [^eggert-env-tz]

[^tz-universality]: たとえばカリフォルニア州だけ夏時間を廃止したら、カリフォルニア州の環境だけ、いちいち `TZ=PST8PDT` から `TZ=PST8` に変えなければなりません。最初から `TZ=America/Los_Angeles` としてあれば tzdb を更新するだけで済みます。
[^eggert-env-tz]: [Paul Eggert 氏のページ](https://web.cs.ucla.edu/~eggert/tz/tz-link.htm)にも、これを支持する記述がいくつかあります。
