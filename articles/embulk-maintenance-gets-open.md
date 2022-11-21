---
title: "Embulk のメンテナンス体制がオープンになります"
emoji: "🛫️" # アイキャッチとして使われる絵文字（1文字だけ）
type: "idea" # tech: 技術記事 / idea: アイデア
topics: [ "embulk", "OSS" ]
layout: default
published: false
---

ここ何年か、オープンソースのプラグイン型データ・ローダー [Embulk](https://www.embulk.org/) のメンテナンスをやっている [@dmikurube](https://github.com/dmikurube) です。

Embulk はオープンソースですが、本体のメンテナンスは Treasure Data でおこなっている、いわゆる「企業発オープンソース」でした。ライセンスは [Apache-2.0](https://www.apache.org/licenses/LICENSE-2.0) で、その意味でちゃんと「オープンソース」ですが、メンテナンス体制や意思決定は実質的に Treasure Data が一手に握っていました。 [^pull-requests]

[^pull-requests]: GitHub からの Pull Requests を受け付けていないわけではなかったのですが。

この Embulk のメンテナンス体制を、よりオープンにしていくことになりました。まだ準備を進めている段階ですが、本記事は、その概要のご紹介です。

オープンって、具体的には?
==========================

大きくは次の 2 点です。

1. 特にプラグイン互換性に影響があるような大きな変更の意思決定プロセスの整備
2. Embulk の意思決定にかかわる人を Treasure Data の外からも正式に加入

EEP (Embulk Enhancement Proposal)
----------------------------------

まず、大きな変更を入れるための意思決定プロセスを整備しました。具体的には、変更の提案に必要な [EEP (Embulk Enhancement Proposal)](https://github.com/embulk/embulk/blob/master/docs/eeps/eep-0001.md) というドキュメントの書き方と、そのレビューのプロセスの定義です。

ドキュメントとかレビューとかプロセスとか、かたっくるしいなー、と思われる方もいらっしゃるかもしれません。とはいえ Embulk はそれ単体では成立しない、いろいろなプラグインありきの「エコシステム」です。そのため、特にプラグイン互換性については慎重な判断が必要になります。その最低限必要な線を固めておこう、というのが EEP の趣旨です。

プロセスは現時点でもなるべく簡略化したつもりですが、ゆるめられるところは今後もゆるめていきたいですね。

ところで、なんかどこかで聞いたような名前だな、とお思いの方、正解です。 EEP は Python の [PEP](https://peps.python.org/pep-0001/) を大きく参考にして…、というか、かなりの部分を拝借しつつ簡略化して最初のバージョンとしました。 [^pep-cc0]

[^pep-cc0]: PEP 自体は "placed in the public domain or under [the CC0-1.0-Universal license](https://creativecommons.org/publicdomain/zero/1.0/deed.en), whichever is more permissive" ということだったので、ありがたくベースにさせてもらいました。 EEP 自体も CC0 にしています。

ちなみに、ここしばらく Embulk は "v0.10" として、非互換をともなう大規模改修を続けてきました。その改修の設計方針などについても (後付けながら) EEP としてまとめていきたいと考えています。

@[card](https://zenn.dev/dmikurube/articles/get-ready-for-embulk-v0-11-and-v1-0)

Embulk core team
-----------------

上記 EEP-1 の中で "Embulk core team" というのを定義しています。この core team が意思決定の主体となり、以下 3 種類の立場の人から構成されます。

1. core committers (core に直接のコミット権を持つ人々)
2. key plugin developers
3. key user representatives

直接の committer でなくとも core team として入っているのは、上記のようにプラグイン互換性が重要になるためです。プラグイン開発者の視点から、「この変更は入れても本当に大丈夫?」とチェックを入れてもらうことを主眼としています。

また、上記 2 や 3 の立場から committer になってもらうことも、もちろん視野に入っています。いま Embulk の本体は割とごちゃごちゃでして、いまの状態から「『committer になってよ!』といきなり言われても困る…」という声もあったため、段階を踏む意味も込めて、このようにしました。

ここ数年は筆者がほぼ一人でメンテナンスしていたこともあって、いまのところコミット権は筆者が持っており、まだしばらくは実質的な最終判断をすることがあると思います。

ですが、将来的には committer を増やし、意思決定も core team 全体でおこなえるようにしていきたいですね。 [^weight-off]

[^weight-off]: というか、そろそろ肩の荷を下ろしたい気持ちもなくはない…。

CLA (Contributor License Agreement)
------------------------------------

いわゆる [CLA (Contributor License Agreement)](https://en.wikipedia.org/wiki/Contributor_License_Agreement) というのも必要かなあ…、と検討はしていますが、まだ入れてはいません。あんまりカタい感じにはしたくないんですけどね。

最初の core team メンバーは?
=============================

* [@dmikurube](https://github.com/dmikurube)
  * 筆者 @ Treasure Data
* [@hiroyuki-sato](https://github.com/hiroyuki-sato)
  * embulk-parser-jsonpath などのメンテナとして
  * Twitter などで積極的にユーザーの声を聞いている key user representative として
* [@civitaspo](https://github.com/civitaspo)
  * embulk-input/output-hdfs, embulk-filter-column, embulk-output-sftp などのメンテナとして
* [@hito4t](https://github.com/hito4t)
  * JDBC プラグイン群のメンテナとして
* [@joker1007](https://github.com/joker1007)
  * embulk-input/output-cassandra, embulk-input/output-kafka, embulk-parser-avro などのメンテナとして
* [@giwa](https://github.com/giwa)
  * embulk-output-bigquery(_java) などのメンテナとして
* [@kekekenta](https://github.com/kekekenta)
  * Embulk を使っていてプラグインも多くメンテナンスしている [trocco](https://trocco.io/) から

(Treasure Data からも、もう 1〜2 人くらい入るかもしれません)

どうして「オープン」に?
========================

Treasure Data 一社だけでメンテナンスしていると、互換性のチェックやレビューの視点が偏ってしまったり、社内の都合で開発がしばらく止まってしまったりすることが、どうしてもありました。これまでにも意図せず互換性が壊れてしまったことは何度かありますし、直近の Embulk v0.10.35 → v0.10.36 の間は半年、次の v0.10.37 までは 4 ヶ月あいています。

社内で話し合いを重ねた結果、それは OSS として、特に Embulk のような「他の開発者によるプラグインのエコシステムありき」の OSS として、健全ではないだろう、という結論にいたりました。

そこで Embulk をより健全に OSS としてメンテナンスしていくべく、これまでプラグインなどを通して Embulk のエコシステムに貢献してくれていた方々に呼びかけ、正式に「中の人」として参加してもらうことになりました。

Embulk をお使いの方々、今後ともよろしくお願いします。

もうちょっと詳しく!
--------------------

来週 11 月 29 日 (火) に予定されている [Treasure Data Tech Talk 2022](https://techplay.jp/event/879660) にて、社内でこの決定にいたった背景を、もう少し深堀りしてご紹介します。

他にもおもしろい話題がそろっていますので、オフラインでもオンラインでも、ぜひお越しください。

@[card](https://techplay.jp/event/879660)
