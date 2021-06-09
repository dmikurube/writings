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

日本では 2020 年開始を目指した夏時間の導入が 2018 年に検討されましたが、ソフトウェア・エンジニアを含む多くの人の反対もあって、同年中に断念されました。 [^give-up-summer-time-in-japan] 今回の断念自体は多くの人にとっていいニュースでしたが、結局いつか導入してしまうかもしれませんし、他の国で新たに導入されることがまだあるかもしれません。実際、日本で夏時間導入が検討されたのは、近年でもこの一度だったわけではありません。 [^summer-time-in-japan-in-2008]

[^give-up-summer-time-in-japan]: [「自民、五輪サマータイムを断念」 (2018年10月31日、共同通信社)](https://nordot.app/430305906140398689)
[^summer-time-in-japan-in-2008]: [「サマータイム制度に反対する意見」 (2008年7月14日、日本労働弁護団)](http://roudou-bengodan.org/proposal/gen080730a/)

日本国内でも急に状況が変わるリスクは常にありますし、そのソフトウェアを他の国や地域に展開するチャンスが来たのに「間違った」日付・時刻処理が障壁となってしまったら、それは悲しいことです。日本国内が前提であっても、日付・時刻の処理を「まともに」実装しておくに越したことはありません。

参考実装として、各種言語・ライブラリの中で今のところ (筆者が知る限り) 最も正面から日付・時刻とタイムゾーンの問題に立ち向かった Java (8以降) の [JSR 310: Date and Time API](https://jcp.org/en/jsr/detail?id=310) をときどき参照します。

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

世界共通の「時間軸上の一点」を表す
===================================

世界各地で「時刻」には差がありますが、それでも時間軸は一本であり、その時間軸上の一点は世界中で同時に訪れます。[^17] 例えば、日本の2017年7月1日 午後3時0分0秒とイギリスの2017年7月1日 午前7時0分0秒は、時間軸上では同じ一点です。それぞれの国や地域の中ではそれぞれの時刻を使っていればあまり問題ありませんが、タイムゾーンをまたぐ場合、特にソフトウェアやインターネットが関わる仕組みの中では、地域によらない時間軸を扱う必要性が高くなります。

UTC という基準があるので要は UTC 時間でその時間軸を表現すればいいのですが、せっかく時間はどこでも平等に流れているのに[^17] いちいち年月日時分秒を計算するのは無駄だらけです。

[^17]: ~~量子物理学的~~ 相対論的 なツッコミは (ry

そこで[^18] 情報通信分野で一般的に使われているのが、いわゆる [UNIX 時間 (Unix time, POSIX time, epoch time)](https://ja.wikipedia.org/wiki/UNIX%E6%99%82%E9%96%93) です。単一の整数として UTC の1970年1月1日 0時0分0秒からの経過時間 (UNIX 時間という場合は主に経過秒数) で時間軸上の点を表現しています。

[^18]: 「そこで」ではないかもしれませんが、歴史を深掘りするのが本記事の本題ではないのでひとまずスルーします。

この UNIX 時間がわかっていれば、任意のタイムゾーンにおける時刻を未来まで含めて簡単に算出することができるはず、ですが、一つだけ問題があります。前述したうるう秒です。うるう秒は実測とのズレを埋めるための仕組みなので、未来の何年にうるう秒が挿入されるかは、実測からの予測を信頼できる数年先までしかわかりません。これでは未来の時刻を計算することができません。

そこで UNIX 時間は「経過時間」としての厳密さを捨てて普段の利便性の方を優先し、うるう秒を無視してしまうことになっています。例えば最近では UTC 時間の2016年12月31日 23時59分59秒と2017年1月1日 0時0分0秒の間にうるう秒が挿入されましたが、このとき UNIX 時間の 1483142400 は2秒間続き、以下のように遷移しました。

| UTC 時間 | UNIX 時間 |
| --- | --- |
| 2016-12-31 23:59:59 | 1483142399 |
| 2016-12-31 23:59:60 | 1483142400 |
| 2017-01-01 00:00:00 | 1483142400 |
| 2017-01-01 00:00:01 | 1483142401 |


技術的な標準・規格
===================

(TODO: IATA, Microsoft)

tz database
------------

タイムゾーンに関する、ソフトウェア・エンジニアにとって最も標準的なデータが [tz database](https://www.iana.org/time-zones) ([Wikipedia](https://ja.wikipedia.org/wiki/Tz_database)) でしょう。 `"Asia/Tokyo"` や `"Europe/London"` のようなタイムゾーンの名前は、この tz database のものです。

tz database のタイムゾーンは `"/"` の前の最初の部分に大陸名・海洋名を用い、続いて、典型的にはそのタイムゾーン内の著名な都市名・島名をその代表として名付けられています。[^19] 国名は基本的に使われません。[^20] `"America/Indiana/Indianapolis"` のように3要素で構成されるタイムゾーンも少数ながら存在します。

[^19]: 日本語でも「東京時間で」「ニューヨーク時間で」などというのと同じような感覚だと思います。
[^20]: 国が関係する状態は変わりやすいので、[政治的事情による変更に対して頑強であるためと記述されています](https://github.com/eggert/tz/blob/2018e/theory.html#L116-L122)。人が生活する単位としての「都市」はそれよりは長く維持される傾向にあると考えられているようです。要出典。ちなみに[初期の提案](https://mm.icann.org/pipermail/tz/1993-October/009233.html)では使うつもりがあったようです。

tz database はボランティアによってメンテナンスされています。タイムゾーンの情報は意外なほど頻繁に変わっており、年に数回は新しい版の tz database がリリースされます。先のサモア独立国のように政治的理由からオフセットが変わる場合、夏時間制度を新しく導入する場合・取りやめる場合、国や地域の境界線に変化があった場合など、様々な理由があります。もちろん、登録されたデータに間違いが見つかることもあります。[^21]

[^21]: `"Asia/Tokyo"` についても、古い BSD ユーザーにはおなじみの [South Ryukyu Islands時間問題](https://ja.wikipedia.org/wiki/%E6%97%A5%E6%9C%AC%E6%A8%99%E6%BA%96%E6%99%82#South_Ryukyu_Islands%E6%99%82%E9%96%93) ([調査報告](http://www.tomo.gr.jp/root/9925.html)) があったり、比較的最近でも[戦時中の情報について一部修正されたり](http://mm.icann.org/pipermail/tz/2018-January/025896.html)しています。

tz database は、特に1970年1月1日以降の全ての変更、夏時間などのあらかじめ定められた切り替わりのルール、うるう秒の情報まで様々な情報を含むことを目標にメンテナンスが続けられています。新しい版は OS や JRE などの処理系、ライブラリなどのメンテナによって更新が取り入れられ、それらを組み込んだアップデートとしてそれぞれ世界中に配信されています。

ちなみに、前述のサモア標準時の変更が tz database に適用されたのは、[実際に標準時が変わる2011年12月のわずか4ヶ月前のこと](http://mm.icann.org/pipermail/tz/2011-August/008703.html)でした。

1970年1月1日以前の情報もある程度は含まれていますが、暦の違い、歴史資料の問題などもあって、完全に正確なものにしようとはそもそも考えられていないようです。[^22] 特に、個別のタイムゾーン名は「1970 年以降に区別する必要のあるタイムゾーン」のみを定義する、という方針らしいです。例えばブラジルのサンパウロを含む一部地域で 1963 年に実施された夏時間を表すために [`"America/Sao_Paulo"` の新設が提案された](https://mm.icann.org/pipermail/tz/2010-January/016007.html)ことがありますが、[この方針により却下されています](https://mm.icann.org/pipermail/tz/2010-January/016010.html)。

[^22]: 歴史の解釈にも揺れがあるため、「正確なもの」はそもそも存在しない、と考えられます。

3文字/4文字の略称
------------------

### `"JST"`: Jerusalem Standard Time?

日本標準時 (Japan Standard Time) は、しばしば `"JST"` と略されているのを見かけます。しかし [tz database を眺めてみる](https://github.com/eggert/tz/blob/2018c/asia)と、実際には以下のように Jerusalem Standard Time を `"JST"` とするコメントも見つかります。

    # JST  Jerusalem Standard Time [Danny Braniss, Hebrew University]
    # JST (Japan Standard Time) has been used since 1888-01-01 00:00 (JST).

3文字4文字のタイムゾーン略称は一見使いやすいのですが、このように、実は一意に特定できるものではありません。このことから、少なくともソフトウェアに再度読み込ませる可能性のあるデータを出力するのに `"JST"` のような略称を用いるのはできるだけ避けましょう。日本では動いていたのに、国外のお客さんがついてから、なんかデータがおかしくなり始めた、みたいな地獄が待っています。

詳しくは後述しますが、出力の際は多くの場合 `"+09:00"` などの固定オフセットを用いるのがいいでしょう。どうしても地域ベースのタイムゾーン名を使わなければならない場合も、少なくとも `"Asia/Tokyo"` などの tz database 名を使いましょう。既に `"JST"` などを用いたデータが生成されてしまっている場合は、データ処理プロセスのできるだけ初期のステージのうちに変換しましょう。

不幸にしてこのような略称を入力しなければならない際は、諦めて決め打ちで読み込むしかありません。世界で使われている多くの略称を収集し、その中から「`"JST"` は `"Asia/Tokyo"` として読み込むよ」などと、仕様を明確にしておきましょう。

JSR 310 の [`java.util.TimeZone`](https://docs.oracle.com/javase/jp/8/docs/api/java/util/TimeZone.html) においても、「互換性のために残してはあるけど非推奨だよ」と明確に記述されています。 [`java.time.ZoneId`](https://docs.oracle.com/javase/jp/8/docs/api/java/time/ZoneId.html) においては、標準ではサポートされず、[略称と tz database 名の対応を後から追加する仕組み](https://docs.oracle.com/javase/jp/8/docs/api/java/time/ZoneId.html#SHORT_IDS)が導入されています。

これらの略称は、文脈が明らかな対人コミュニケーションにおいて**のみ**用いるのがいいでしょう。

### `"EST"`, `"EDT"`, `"CST"`, `"CDT"`, `"MST"`, `"MDT"`, `"PST"`, `"PDT"`

Ruby の `Time.strptime` に見られるように、[アメリカ合衆国の一部略称のみ標準で解釈できる](https://svn.ruby-lang.org/cgi-bin/viewvc.cgi/tags/v2_5_0/lib/time.rb?view=markup#l103)ようになっている処理系があります。これは [RFC 2822](https://www.ietf.org/rfc/rfc2822.txt) などに、これらの略称が標準として含まれていることに由来するようです。

ただし、それでもこれらの略称の使用はできるだけ避けるべきです。夏時間などの扱いに処理系による違いが見られるからです。

例えば Ruby の `Time.strptime` は RFC に則って `"PST"` を常に `"-08:00"` として扱い、逆に `"PDT"` は常に `"-07:00"` として扱います。これは、例えば夏時間期間である `"2017-07-01 12:34:56"` に `"PST"` を付けた場合でも同様で `"2017-07-01 12:34:56 -08:00"` として解釈されます。

しかし、例えば JSR 310 の前身である [Joda-Time](http://www.joda.org/joda-time/) では少し事情が異なります。[`org.joda.time.DateTimeUtils#getDefaultTimeZoneNames`](http://www.joda.org/joda-time/apidocs/org/joda/time/DateTimeUtils.html#getDefaultTimeZoneNames--) は、例えば `"PST"`, `"PDT"` をともに `"America/Los_Angeles"` に対応させます。どうやら [`org.joda.time.format.DateTimeFormat.forPattern`](http://www.joda.org/joda-time/apidocs/org/joda/time/format/DateTimeFormat.html#forPattern-java.lang.String-) では間接的にこの `getDefaultTimeZoneNames` が使われているようで[^23]、このため Ruby の `Time.strptime` や RFC とは異なり `"2017-07-01 12:34:56 PST"` は `"2017-07-01 12:34:56 -07:00"` として解釈されます。

[^23]: Joda-Time のコードをあまり深く追えてはいないので要検証。

Military time zones
--------------------

主に米軍で使われているタイムゾーン表現で、地域によらない固定オフセットに対応します。 `"A"` 〜 `"Z"` までの大文字アルファベット (`"J"` のみ不使用) を `"-12:00"` から `"+12:00"` まで 1 時間おきに 25 個のオフセットに割り当てています。 ([Wikipedia:en](https://en.wikipedia.org/wiki/List_of_military_time_zones))

`"Z"` の一文字で `"UTC"`/`"GMT"` を指す表現は見たことがある人が多いと思いますが、ここから来ています。

これらも `"PST"` などと同様に [RFC 2822](https://www.ietf.org/rfc/rfc2822.txt) に標準として含まれています。こちらは一意に定まる固定オフセット表現なので「避けるべき」というほどではありません。が、有名な `"Z"` 以外は一般的に知られているとは言い難いので、出力の際にわざわざ選ぶようなものでもないでしょう。 Military time zone では UTC から30分差や15分差のタイムゾーンを表現できない、という問題もあります。


おまけ: 昔の日付・時刻
=======================

(TODO: 少なくともグレゴリオ暦、ユリウス暦、先発グレゴリオ暦については書いておきたいのですが、力尽きたのでひとまず現状で公開しています)


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
