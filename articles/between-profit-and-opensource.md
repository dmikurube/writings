---
title: "営利とオープンソースの間で 〜 Embulk の場合"
emoji: "🧬️" # アイキャッチとして使われる絵文字（1文字だけ）
type: "idea" # tech: 技術記事 / idea: アイデア
topics: [ "embulk", "opensource" ]
layout: default
published: false
---

2023 年 6 月は、クックパッド社でフルタイムの Ruby コミッターとして研究開発を行っていたお二人が人員削減の影響を受けたことに端を発して、オープンソース・ソフトウェアに関わってきたソフトウェア・エンジニアを中心とした一部で、企業活動とオープンソースの関係についての議論が起こりました。

@[card](https://note.com/cookpad/n/nc1b63628422c)

@[card](https://note.com/ruiu/n/ndfcda9adb748)

@[card](https://knqyf263.hatenablog.com/entry/2023/06/07/175830)

自社のソフトウェアを積極的にオープンソースとして公開している[時雨堂](https://shiguredo.jp/)さんも、自社がオープンソースとどのように向き合っているのかを文章化しています。

@[card](https://voluntas.medium.com/%E4%BC%81%E6%A5%AD-oss-%E3%82%92%E7%B6%99%E7%B6%9A%E9%96%8B%E7%99%BA%E3%81%99%E3%82%8B%E3%81%9F%E3%82%81%E3%81%AB%E3%82%84%E3%81%A3%E3%81%A6%E3%81%84%E3%82%8B%E3%81%93%E3%81%A8-c783be34ccde)

@[card](https://voluntas.medium.com/%E6%99%82%E9%9B%A8%E5%A0%82%E3%81%AF%E4%BD%95%E3%82%92%E3%81%97%E3%81%A6%E3%81%84%E3%82%8B%E4%BC%9A%E7%A4%BE%E3%81%AA%E3%81%AE%E3%81%8B-a32fbd5a746a)

さて、筆者は ["Open Source is in our DNA"](https://www.treasuredata.com/opensource/) を謳う [Treasure Data](https://www.treasuredata.com/) という会社で、自社発の [Embulk](https://www.embulk.org/) というオープンソース・ソフトウェアのメンテナンスを引き継いで、そのメンテナーをかれこれ 5 〜 6 年ほどやっています。

本記事は、オープンソースを前面に出して活動してきた企業で、オープンソース・ソフトウェアのメンテナーを業務として務めてきた一社員という視点から、共有できることがあるかもしれないなあ、と考えて書いたものです。なにかの参考になれば幸いです。


Embulk のメンテナンス体制
==========================

@[card](https://zenn.dev/dmikurube/articles/embulk-maintenance-gets-open)


Treasure Data の中の Embulk
============================

@[card](https://techplay.jp/event/879660)

@[card](https://api-docs.treasuredata.com/blog/embulk-in-td/)


インターフェースと依存関係
===========================

@[card](https://docs.google.com/presentation/d/e/2PACX-1vQf7dSmMDTBqQQgF-NcMqCssWv34BIVacK6_4xrMAIJnbqNXt65goIW0PhzfXIUSJf_SKgEmS5Ujqvo/pub?start=false&loop=false&slide=id.p)

プラグイン・メカニズムの光と影
-------------------------------

明示的なインターフェース
-------------------------

暗黙のインターフェース
-----------------------
