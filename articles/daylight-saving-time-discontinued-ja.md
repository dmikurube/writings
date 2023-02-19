---
title: "夏時間制の終わり 〜 そのときなにが起こるのか"
emoji: "☀️" # アイキャッチとして使われる絵文字（1文字だけ）
type: "tech" # tech: 技術記事 / idea: アイデア
topics: [ "datetime", "time", "timezone", "dst" ]
layout: default
published: false
---

2022 年 3 月、「2023 年から夏時間を恒久化する」という法案がアメリカ合衆国議会上院を通過したというニュースがあり、一部のソフトウェア・エンジニアの間で話題になりました。

@[card](https://jp.reuters.com/article/usa-senate-daylight-idJPKCN2LD0ET)

@[card](https://www.reuters.com/world/us/us-senate-approves-bill-that-would-make-daylight-savings-time-permanent-2023-2022-03-15/)

勘違いしてはならないのは、とりあえず 2022 年 11 月現在においても、この法案はまだ上院を通過しただけで、下院まで通ったわけではありません。本当にアメリカで来年から夏時間が恒久化するのかはまだよくわからない、ということです。

@[card](https://thehill.com/homenews/house/3571007-permanent-daylight-saving-time-hits-brick-wall-in-house/)

そういえば EU でも夏時間を廃止するっていう話があったはずなんですが、どうやらまだ実現していないみたいです。

@[card](https://www.bloomberg.com/news/articles/2021-03-11/will-daylight-saving-time-ever-end)

さて実際のところ、どうなるんでしょうね。

…という現実の政治的な話はいったん脇に置いておいて、本記事は「もし夏時間制が今すぐ廃止されたら」「そのときなにが起こるのか」という仮定のお話をするため、「[タイムゾーン呪いの書](./curse-of-timezones-common-ja)」の補遺として書き起こしたものです。

@[card](https://zenn.dev/dmikurube/articles/curse-of-timezones-common-ja)

夏時間廃止のニュースに喜ぶソフトウェア・エンジニアの方々には、もしかすると冷や水を浴びせる内容になるかもしれませんが、よければご一読ください。

EU の廃止案の場合
==================

2021年現在、夏時間制の廃止が世界各地で議論されています。夏時間制の廃止はソフトウェア・エンジニアとして長期的には歓迎できることですが、今まで実施されていたものが実施されなくなるというのは、そんなに単純な話ではありません。状況を注視する必要があるでしょう。

たとえば EU における夏時間制は、欧州議会の議決では 2021年を最後に廃止されることになっていました。 [^eu-summer-time-2019] しかし Brexit やコロナ禍もあって、その 2021年の 3月になってもまだ状況は不透明だったようです。 [^eu-summer-time-2021-asahi] [^eu-summer-time-2021-bloomberg]

[^eu-summer-time-2019]: [「欧州議会、サマータイム廃止の法案を可決　２０２１年に」 (2019年 3月 28日、朝日新聞)](https://digital.asahi.co
m/articles/ASM3W0014M3VUHBI02R.html)
[^eu-summer-time-2021-asahi]: [「これで最後？EUが夏時間へ　廃止検討するも議論進まず」 (2021年 3月 27日、朝日新聞)](https://digital.asahi.com/articles/ASP3W4HWJP3VUHBI04L.html)
[^eu-summer-time-2021-bloomberg]: ["Why Europe Couldn’t Stop Daylight Saving Time" (March 11, 2021, Bloomberg)](https://www.bloomberg.com/news/articles/2021-03-11/will-daylight-saving-time-ever-end)

EU の状況をさらにややこしくしているのは、夏時間廃止後の新しい標準時を「旧来の標準時」に戻すのか「それまでの夏時間」に新しく合わせるのか、最終的には各国にまかされているらしい、ということです。なるべくそろえたい思惑はさすがにあるようですが、最終的にどうなるか、どうやら 2021年 6月現在もわかっていません。極端にいえば、いままで同じ「[中央ヨーロッパ標準時 (CET)](https://ja.wikipedia.org/wiki/%E4%B8%AD%E5%A4%AE%E3%83%A8%E3%83%BC%E3%83%AD%E3%83%83%E3%83%91%E6%99%82%E9%96%93)」を使っていて時差のなかったドイツ・フランス・イタリアの間に、来年からは時差が生じる、なんてことがあるかもしれません。タイムゾーンの呪いは、意識していなかったところから襲ってきます。恐ろしい話です。 [^italy-2021]

[^italy-2021]: [「サマータイム廃止でイタリアは最後の夏時間」 (2021年 3月 21日、ニューズウィーク日本版内 World Voice)](https://www.newsweekjapan.jp/worldvoice/vismoglie/2021/03/post-23.php)

また、カリフォルニア州では 2018年の住民投票で夏時間の変更が支持されました。 [^california-proposition-7] 2021年 6月時点ではまだ具体的な変更の予定はないようですが、これから変わるかもしれません。

[^california-proposition-7]: [「カリフォルニア州、住民は夏時間の変更を支持」 (2018年11月19日、日本貿易振興機構)](https://www.jetro.go.jp/biznews/2018/11/f4ab9860d030abf0.html)

カリフォルニア州の変更で要注意なのは、「ロサンゼルス時間」を「[太平洋時間](https://ja.wikipedia.org/wiki/%E5%A4%AA%E5%B9%B3%E6%B4%8B%E6%99%82%E9%96%93)」の代表として用いているソフトウェアがある、ということでしょう。後述しますが、実は Java の標準 API がそうです。たとえばワシントン州シアトルの冬の時刻を表現するつもりで「太平洋標準時 (`PST`)」を使っていると、それが内部的にはロサンゼルス時間あつかいになっているわけです。そんな中でカリフォルニア州のみ夏時間が廃止されたら…、どうするんでしょうね。タイムゾーンの呪いには、なにか恐ろしいことが起きそうだとはわかっていても、各国や地域の政治的決定が定まるまでどうなるかわからない、という面があります。考えたくない話です。
