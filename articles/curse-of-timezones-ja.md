---
title: "タイムゾーン呪いの書"
emoji: "🌏" # アイキャッチとして使われる絵文字（1文字だけ）
type: "tech" # tech: 技術記事 / idea: アイデア
topics: [ "timezone", "tz", "jsr310" ]
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

タイムゾーンの定義と実際の用法
------------------------------

前述のとおり「タイムゾーン」は標準時をもとに定義されます。標準時は、後述する夏時間とは独立した概念で、たとえばアメリカのユタ州とアリゾナ州は同じ「[山岳部標準時 (Mountain Standard Time, MST)](https://ja.wikipedia.org/wiki/%E5%B1%B1%E5%B2%B3%E9%83%A8%E6%A8%99%E6%BA%96%E6%99%82)」を使いますが、夏時間制には差があります。 [^summer-time-in-utah-and-arizona] そしてこの2州は、定義上は同じタイムゾーンに属します。

[^summer-time-in-utah-and-arizona]: [ユタ州](https://ja.wikipedia.org/wiki/%E3%83%A6%E3%82%BF%E5%B7%9E)は夏時間を採用し、[アリゾナ州](https://ja.wikipedia.org/wiki/%E3%82%A2%E3%83%AA%E3%82%BE%E3%83%8A%E5%B7%9E)の大部分は[夏時間を採用していません](https://ja.wikipedia.org/wiki/%E3%82%A2%E3%83%AA%E3%82%BE%E3%83%8A%E6%99%82%E9%96%93)。

しかし「タイムゾーン」という言葉は、実態としてはきわめて多様な使われ方をしています。ユタ州とアリゾナ州を別のタイムゾーンと呼ぶ例も多くあります。逆に、異なる標準時の国や地域でも、等価な時刻系を使っていたものをまとめて一つのタイムゾーンと呼んだ例もあります。 [^old-windows-timezones-in-japan-korea] 後述する `UTC+9` などの特定の時刻系そのものをタイムゾーンと呼ぶこともあります。

[^old-windows-timezones-in-japan-korea]: 古い Windows 95 などでは、タイムゾーンの設定が「(GMT+09:00) 東京、大阪、札幌、ソウル、ヤクーツク」などとなっていたのを覚えている方もいるかもしれません。

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

その他の言語
-------------

(タイムゾーンの扱いについて充分な内容を扱った記事には、お知らせいただけたらリンクを張らせていただきたいと思います)

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
