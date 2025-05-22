---
title: "コードの寿命・データの寿命・互換性の寿命"
emoji: "♻" # アイキャッチとして使われる絵文字（1文字だけ）
type: "idea" # tech: 技術記事 / idea: アイデア
topics: [ "lecture", "lifecycle" ]
layout: default
published: true
---

これを記事にしている 2025 年 5 月の二年ほど前 (2023-06-02) に、縁あって[明治大学 情報科学科](https://www.meiji.ac.jp/sst/cs/index.html)での特別講義 [^special-lecture] を担当させてもらいました。

[^special-lecture]: 社会人を一回ずつ講師として呼んでくる、いわゆるオムニバス形式のやつです。

身内の評判は悪くなかったのでスライドは公開していたんですが、単に [Google Slides](https://docs.google.com/presentation/d/e/2PACX-1vQf7dSmMDTBqQQgF-NcMqCssWv34BIVacK6_4xrMAIJnbqNXt65goIW0PhzfXIUSJf_SKgEmS5Ujqvo/pub?start=false&loop=false&slide=id.p) を公開状態にしただけだったんですね。 [^google-slides]

[^google-slides]: Google Slides は Zenn でも 埋め込めないし、やっぱり SpeakerDeck かなにかにしておいたほうがよかったかなあ、とは思いつつ…。

@[tweet](https://twitter.com/dmikurube/status/1665560701983764480)

これではあとから参照・引用するのも難しく、ちょっともったいないかと思ったので、いまさらながら記事の形でまとめなおしておくことにしました。

一年も経てば情報が古くなってしまうコの業界です。賞味期限切れの話もあると思いますが、話のネタにでもしてもらえれば幸いです。

# 講義の対象と目的

> この講義、目的は2つあって、まず「最新の情報科学トピックに触れる」こと。
> それから、就職活動が始まる3年生がメインの対象者なので、
> 今後のキャリアプランとか人生指針に関するいろいろな視点を持ってもらうことです。

このお話をいただいたとき、講義の目的は上のようなものとうかがいました。そこで「自分が学生のときにわかっていなかったこと」がいいかなと、以下のようなテーマにしてみました。

> **「ソフトウェア・サービスを長いあいだメンテナンスする」のは、なにが大変なのか**

[![それなら…ということで、本日のお題](/images/life-of-code-life-of-data-life-of-compatibility-003.png)](https://docs.google.com/presentation/d/e/2PACX-1vQf7dSmMDTBqQQgF-NcMqCssWv34BIVacK6_4xrMAIJnbqNXt65goIW0PhzfXIUSJf_SKgEmS5Ujqvo/pub?start=false&loop=false&slide=id.g24324d5ef34_0_33)

これは実際、自分がまさに学生のときにわかっておらず、仕事を始めてから「なるほどー…」と学んだことでもあります。

このときの話者の勤務先がデータベース技術を基盤にした会社だったのにくわえて、大半の受講生は「データベース」の講義をまだ受けていないということだったので、あえてデータベースの話題を持ってきてみました。データベースのことを知らない人にも「なにが難しいのか」「なにが面白いのか」がなんとなくは伝わるように、そして興味を持った人が自分で調べるとっかかりにはなるように、話を構成してみたつもりです。

そのため、詳しい人からすれば「なにを当たり前のことを」と感じるような内容も、「その説明はちょっと違わないか」と感じるような飛躍や簡略化も含まれているかもしれません。その点はひとえに話者の力不足によるものです。お許しください。

お題2として「大学 (情報科学科) ではなにを学んでおいたらいいのか」みたいな話もしましたが、本記事では割愛します。 (興味があるかたは [Google Slides](https://docs.google.com/presentation/d/e/2PACX-1vQf7dSmMDTBqQQgF-NcMqCssWv34BIVacK6_4xrMAIJnbqNXt65goIW0PhzfXIUSJf_SKgEmS5Ujqvo/pub?start=false&loop=false&slide=id.p) のほうを直接ご覧ください)

## 自己紹介

記事ではスライドは省略しますが、この講義の話者 (本記事の筆者) はコンピューター・サイエンスの研究職的な仕事からキャリアを始め、わりと「基盤ソフトウェア」「ミドルウェア」的な対象を中心に仕事をしてきたソフトウェア・エンジニアです。

話者はちなみに、スライドに挙がっている Treasure Data を 2025 年 4 月で退職しています。 [^treasure-data]

[^treasure-data]: なのでこの記事は 4 月中に出してしまおうと思った、んですが、ちょっと間に合いませんでした。

# コードの寿命・データの寿命・互換性の寿命

## 実世界のソフトウェア

### 大学で (特に講義で) 学ぶプログラムと実世界のソフトウェアの違い [(p.6)](https://docs.google.com/presentation/d/e/2PACX-1vQf7dSmMDTBqQQgF-NcMqCssWv34BIVacK6_4xrMAIJnbqNXt65goIW0PhzfXIUSJf_SKgEmS5Ujqvo/pub?start=false&loop=false&slide=id.g1f5fcbb2f01_0_10)

[![大学で (特に講義で) 学ぶプログラムと実世界のソフトウェアの違い](/images/life-of-code-life-of-data-life-of-compatibility-006.png)](https://docs.google.com/presentation/d/e/2PACX-1vQf7dSmMDTBqQQgF-NcMqCssWv34BIVacK6_4xrMAIJnbqNXt65goIW0PhzfXIUSJf_SKgEmS5Ujqvo/pub?start=false&loop=false&slide=id.g1f5fcbb2f01_0_10)

大学で、特に講義などで学ぶ「プログラム」と、実世界で使われる「ソフトウェア・サービス」の違いとはなんでしょう。

その大きな違いの一つは、後者は比較的長期間、継続的に「メンテナンス」されるもの、ということではないでしょうか。 [^dead]

[^dead]: メンテナンスされる間もなく淘汰されるソフトウェアも、無数にあります。

そして長く使われ、メンテナンスが続くソフトウェア・サービスほど、それがあつかうデータの量、データを維持しなければならない期間、データにかかわる人の数、すべてが増加していきます。

### データ: ソフトウェア・メンテナンスの視点の一つ [(p.7)](https://docs.google.com/presentation/d/e/2PACX-1vQf7dSmMDTBqQQgF-NcMqCssWv34BIVacK6_4xrMAIJnbqNXt65goIW0PhzfXIUSJf_SKgEmS5Ujqvo/pub?start=false&loop=false&slide=id.g1f70bc4e279_1_0)

[![データ: ソフトウェア・メンテナンスの視点の一つ](/images/life-of-code-life-of-data-life-of-compatibility-007.png)](https://docs.google.com/presentation/d/e/2PACX-1vQf7dSmMDTBqQQgF-NcMqCssWv34BIVacK6_4xrMAIJnbqNXt65goIW0PhzfXIUSJf_SKgEmS5Ujqvo/pub?start=false&loop=false&slide=id.g1f70bc4e279_1_0)

およそソフトウェアとは「なにかしらのデータを入力し、なにかしらのデータを出力するもの」と一般化されます。なかでも特に近年の、技術者ではない「一般ユーザー」が使う (Web) アプリケーションは、それ自体がデータを「蓄積する」サービスであることが多い、といえるでしょう。

データを蓄積できなければ、ビジネスとして始まらないし、お金にならない。もっと極論すれば、コードよりも貯めたデータが価値の源泉になる、とすらいえるかもしれません。

もちろん世の中には OS やコンパイラのように「それ単体ではデータを蓄積しないソフトウェア」も数多くあります。大学などで触れるのはこちらが多いかもしれません。

これらは現代でも重要で「価値のある」ソフトウェアです。しかし現代では、これらは「『データを蓄積するサービス』の一部」や「『サービス』を構築するための道具」として使われることが多くなり、これらを単体の「商品」としたビジネスは成立しづらくなっています。

ゲーム専用機向けのゲームなども、かつては単体で成立していましたが、現在では「サービスとしてのゲーム」に寄ってきています。

## コードの寿命 v.s. データの寿命

### インターネットサービスのよくある構成 [(p.10)](https://docs.google.com/presentation/d/e/2PACX-1vQf7dSmMDTBqQQgF-NcMqCssWv34BIVacK6_4xrMAIJnbqNXt65goIW0PhzfXIUSJf_SKgEmS5Ujqvo/pub?start=false&loop=false&slide=id.g2483f7076ac_0_31)

[![インターネットサービスのよくある構成](/images/life-of-code-life-of-data-life-of-compatibility-010.png)](https://docs.google.com/presentation/d/e/2PACX-1vQf7dSmMDTBqQQgF-NcMqCssWv34BIVacK6_4xrMAIJnbqNXt65goIW0PhzfXIUSJf_SKgEmS5Ujqvo/pub?start=false&loop=false&slide=id.g2483f7076ac_0_31)

ものすごく雑にいろいろ簡略化した絵ですが、よくある Web サービスの構成は、だいたいこんな図式になるでしょう。

* ユーザーの手元の Web ブラウザで動く「フロントエンド」やスマートフォンのアプリは、「バックエンド」のアプリケーション・サーバーと通信
* アプリケーション・サーバーは、さらに背後のデータベースにデータを保存、またはデータベースのデータを参照

さて。ここに登場した「コード (プログラム)」と「データ」は、どちらが「長生き」でしょう?

### コードとデータ、どちらが「長生き」か? [(p.11)](https://docs.google.com/presentation/d/e/2PACX-1vQf7dSmMDTBqQQgF-NcMqCssWv34BIVacK6_4xrMAIJnbqNXt65goIW0PhzfXIUSJf_SKgEmS5Ujqvo/pub?start=false&loop=false&slide=id.g2483f7076ac_0_77)

[![コードとデータ、どちらが「長生き」か?](/images/life-of-code-life-of-data-life-of-compatibility-011.png)](https://docs.google.com/presentation/d/e/2PACX-1vQf7dSmMDTBqQQgF-NcMqCssWv34BIVacK6_4xrMAIJnbqNXt65goIW0PhzfXIUSJf_SKgEmS5Ujqvo/pub?start=false&loop=false&slide=id.g2483f7076ac_0_77)

その答えは多くの場合、「データ」のほうになるでしょう。

Web サービスを営む企業の多くでは、「コード」への変更をそれこそ毎日のように加えて本番環境に適用 (デプロイ) しています。一方で、「データ」をあとから運営企業の意思のみで変更するのは容易ではありません。

### コードは直せる。入ったデータは (中々) 直せない [(p.12)](https://docs.google.com/presentation/d/e/2PACX-1vQf7dSmMDTBqQQgF-NcMqCssWv34BIVacK6_4xrMAIJnbqNXt65goIW0PhzfXIUSJf_SKgEmS5Ujqvo/pub?start=false&loop=false&slide=id.g2483f7076ac_0_108)

[![コードは直せる。入ったデータは (中々) 直せない](/images/life-of-code-life-of-data-life-of-compatibility-012.png)](https://docs.google.com/presentation/d/e/2PACX-1vQf7dSmMDTBqQQgF-NcMqCssWv34BIVacK6_4xrMAIJnbqNXt65goIW0PhzfXIUSJf_SKgEmS5Ujqvo/pub?start=false&loop=false&slide=id.g2483f7076ac_0_108)

その理由の一つは、「データ」はあくまでもサービスのお客さんのものであって、運営企業のものではない、ということです。メールや投稿の内容が勝手に書き換わっていたり、ゲームのデータが変わっていたり、取引履歴が違うものになっていたりしたら、ユーザーとして暴れるどころでは済まないですよね。

そして、ひとたび書き換わった・失われたデータのオリジナルを取り戻すのは難しい、というのが第二の理由です。たいていのコードの変更は取り返せますが、データの変更は取り返しがつきません。 [^unrevertible-modification]

[^unrevertible-modification]: もちろん「取り返しがつかない」コードの変更もありますが、その最も典型的な例こそが「保存データにかかわる」コードの変更でしょう。

### データベースでかけておく「制約」 [(p.13)](https://docs.google.com/presentation/d/e/2PACX-1vQf7dSmMDTBqQQgF-NcMqCssWv34BIVacK6_4xrMAIJnbqNXt65goIW0PhzfXIUSJf_SKgEmS5Ujqvo/pub?start=false&loop=false&slide=id.g2483f7076ac_0_123)

[![データベースでかけておく「制約」](/images/life-of-code-life-of-data-life-of-compatibility-013.png)](https://docs.google.com/presentation/d/e/2PACX-1vQf7dSmMDTBqQQgF-NcMqCssWv34BIVacK6_4xrMAIJnbqNXt65goIW0PhzfXIUSJf_SKgEmS5Ujqvo/pub?start=false&loop=false&slide=id.g2483f7076ac_0_77)

ここでデータをあつかうサービスの例として、「ユーザー」と「投稿」で構成される SNS のようなサービスを想像してみましょう。

現実のサービスで使われるデータには、たいていなにかしら制約や条件をつけます。

たとえば「そのサービスの『ユーザー』はかならずユーザー ID を持って」いなければ困りますし、「ユーザー ID が他のユーザーと重複」してしまったらもっと困ります。「ユーザーはメールアドレスを登録」しておいてもらわないと本人確認の手段がありませんし、その「メールアドレスが他のユーザーと同じもの」になってしまったら識別に支障があるでしょう。

このような制約や条件を基盤となるデータベースの段階でかけておくと、事故を避けられたり、複雑さを減らせたり、性能を上げられたり、さまざまな恩恵があります。

[![データベースでかけておく「制約」](/images/life-of-code-life-of-data-life-of-compatibility-014.png)](https://docs.google.com/presentation/d/e/2PACX-1vQf7dSmMDTBqQQgF-NcMqCssWv34BIVacK6_4xrMAIJnbqNXt65goIW0PhzfXIUSJf_SKgEmS5Ujqvo/pub?start=false&loop=false&slide=id.g2483f7076ac_0_176)

前述のような「制約」をデータベースの「テーブル」に素直に起こすと、たとえば上のような感じになるでしょうか。ユーザーはユーザー ID を持っていて、それは unique (重複しない) で non-null (空ではない) である、といった具合です。

(実際にはこれは「悪例」で、こんなテーブル設計にすることは普通ありません。とりあえずイメージしてもらうために、また後の話の都合で、こんな例にしています)

### 「制約」は要件で変わることがある [(p.15)](https://docs.google.com/presentation/d/e/2PACX-1vQf7dSmMDTBqQQgF-NcMqCssWv34BIVacK6_4xrMAIJnbqNXt65goIW0PhzfXIUSJf_SKgEmS5Ujqvo/pub?start=false&loop=false&slide=id.g2483f7076ac_0_153)

[![「制約」は要件で変わることがある](/images/life-of-code-life-of-data-life-of-compatibility-015.png)](https://docs.google.com/presentation/d/e/2PACX-1vQf7dSmMDTBqQQgF-NcMqCssWv34BIVacK6_4xrMAIJnbqNXt65goIW0PhzfXIUSJf_SKgEmS5Ujqvo/pub?start=false&loop=false&slide=id.g2483f7076ac_0_153)

こうして最初にかけた制約ですが、サービスの継続や成長にしたがって、制約を変えたくなることがあります。

たとえば「ユーザー ID をあとから変えたい」なんて要望はいかにもありそうですし、「メールアドレス以外に LINE の ID も使えるようにしたい」というのもありそうです。「一つの投稿を複数ユーザーの共同名義で出せるようにしたい」なんてこともあるかもしれません、

### 制約もデータも、あとから変えるのは大変 [(p.16)](https://docs.google.com/presentation/d/e/2PACX-1vQf7dSmMDTBqQQgF-NcMqCssWv34BIVacK6_4xrMAIJnbqNXt65goIW0PhzfXIUSJf_SKgEmS5Ujqvo/pub?start=false&loop=false&slide=id.g249a8efc8e9_0_103)

[![制約もデータも、あとから変えるのは大変](/images/life-of-code-life-of-data-life-of-compatibility-016.png)](https://docs.google.com/presentation/d/e/2PACX-1vQf7dSmMDTBqQQgF-NcMqCssWv34BIVacK6_4xrMAIJnbqNXt65goIW0PhzfXIUSJf_SKgEmS5Ujqvo/pub?start=false&loop=false&slide=id.g249a8efc8e9_0_103)

たとえば「ユーザー ID をあとから変えたい」を実際に考えてみましょう。「左の『ユーザーテーブル』のユーザー ID を変えるだけじゃないの?」と思うかもしれませんが、そうはいきません。

右の『投稿テーブル』の各投稿にもユーザー ID が使われていますよね。これではそのユーザーのすべての投稿について、ユーザー ID を書き換えなければなりません。もしこのユーザーが 100 万件の投稿をしていたら、その 100 万件の投稿すべてのユーザー ID を書き換えないとならないのです。

この例が「悪例」というのはこういうところで、そんなことはやっていられないので、普通はこんなテーブル設計にしないんですね。

[![制約もデータも、あとから変えるのは大変](/images/life-of-code-life-of-data-life-of-compatibility-017.png)](https://docs.google.com/presentation/d/e/2PACX-1vQf7dSmMDTBqQQgF-NcMqCssWv34BIVacK6_4xrMAIJnbqNXt65goIW0PhzfXIUSJf_SKgEmS5Ujqvo/pub?start=false&loop=false&slide=id.g2483f7076ac_0_197)

典型的なやり方としては、たとえばユーザーは、外から見える「外部ユーザー ID」と、外には見せない「内部ユーザー ID」の二つを持つようにして、「内部 ID」は決して変わらないものと決めてしまいます。

そうして、ユーザー (人) には「外部ユーザー ID」のみを使ってもらい、サービス内の参照には「内部ユーザー ID」を使うようにするわけです。こうすると、ユーザーから見た「ユーザー ID の変更」は低いコストでできるようになります。

[![制約もデータも、あとから変えるのは大変](/images/life-of-code-life-of-data-life-of-compatibility-018.png)](https://docs.google.com/presentation/d/e/2PACX-1vQf7dSmMDTBqQQgF-NcMqCssWv34BIVacK6_4xrMAIJnbqNXt65goIW0PhzfXIUSJf_SKgEmS5Ujqvo/pub?start=false&loop=false&slide=id.g3af0462f0e58eb45_0)

データの保存にかかわる設計ミスは、「コードを直す」だけでは済まない状況を、こうして容易に引き起こしてしまいます。うっかりするとデータ全件に手を入れるような必要が生じることもありますが、収益性を維持するためには、それをできるだけ「サービスを止めずに」やらなければなりません。

100 万件程度なら実はマシなほうで、たとえば Twitter (現X) や LINE の投稿数なんかを想像すると、現実的でないレベルになったりします。 [^relational-twitter-line]

[^relational-twitter-line]: もちろん Twitter (現X) や LINE みたいな規模にもなると、こうやって単純にリレーショナル・データベースに保存しているようなことはそうそうありません。あくまでも想像してもらいやすくするための例として用意したものです。

[![制約もデータも、あとから変えるのは大変](/images/life-of-code-life-of-data-life-of-compatibility-019.png)](https://docs.google.com/presentation/d/e/2PACX-1vQf7dSmMDTBqQQgF-NcMqCssWv34BIVacK6_4xrMAIJnbqNXt65goIW0PhzfXIUSJf_SKgEmS5Ujqvo/pub?start=false&loop=false&slide=id.g3af0462f0e58eb45_17)

他に「一つの投稿を複数ユーザーの共同名義で出せるようにしたい」という要望も挙げましたが、これはどうでしょうか?

一つの「投稿」に複数の「内部 ユーザー ID」を対応させたいわけですが、それをそのままテーブルに反映しようとすると、やっぱりややこしいことになります。

(講義中には具体的なところまで踏み込みませんでしたが、受講生のみなさんがこれから「データベース」の講義を受ける中で出てくるであろう「正規化」などのキーワードを紹介しました)

### さらに、データの量・流量が増えると別の「制約」が [(p.20)](https://docs.google.com/presentation/d/e/2PACX-1vQf7dSmMDTBqQQgF-NcMqCssWv34BIVacK6_4xrMAIJnbqNXt65goIW0PhzfXIUSJf_SKgEmS5Ujqvo/pub?start=false&loop=false&slide=id.g2483f7076ac_0_217)

[![さらに、データの量・流量が増えると別の「制約」が](/images/life-of-code-life-of-data-life-of-compatibility-020.png)](https://docs.google.com/presentation/d/e/2PACX-1vQf7dSmMDTBqQQgF-NcMqCssWv34BIVacK6_4xrMAIJnbqNXt65goIW0PhzfXIUSJf_SKgEmS5Ujqvo/pub?start=false&loop=false&slide=id.g2483f7076ac_0_217)

ところで、データの量や流量がさきほど触れた Twitter (現X) や LINE のような規模にまで増えてくると、「一台の『データベース』」ではまかないきれないことがあります。

[![さらに、データの量・流量が増えると別の「制約」が](/images/life-of-code-life-of-data-life-of-compatibility-021.png)](https://docs.google.com/presentation/d/e/2PACX-1vQf7dSmMDTBqQQgF-NcMqCssWv34BIVacK6_4xrMAIJnbqNXt65goIW0PhzfXIUSJf_SKgEmS5Ujqvo/pub?start=false&loop=false&slide=id.g2483f7076ac_0_233)

規模が大きくなったとき、アプリケーション・サーバーを複数台に分散させていたように、データベースも分散して対応する方法があります。しかし、データベースを分散させるのは実はあまり簡単なことではありません。

わかりやすいところでは、「書いたデータを書いた直後に読める保証がなくなる」とか「『連番』を効率的に作ることができなくなる」といった制約が生じてきます。するとアプリケーションのほうも、そのような制約を前提として設計を考え直す必要があるのです。

### そして制約は「伝播」する [(p.22)](https://docs.google.com/presentation/d/e/2PACX-1vQf7dSmMDTBqQQgF-NcMqCssWv34BIVacK6_4xrMAIJnbqNXt65goIW0PhzfXIUSJf_SKgEmS5Ujqvo/pub?start=false&loop=false&slide=id.g2483f7076ac_0_147)

[![そして制約は「伝播」する](/images/life-of-code-life-of-data-life-of-compatibility-022.png)](https://docs.google.com/presentation/d/e/2PACX-1vQf7dSmMDTBqQQgF-NcMqCssWv34BIVacK6_4xrMAIJnbqNXt65goIW0PhzfXIUSJf_SKgEmS5Ujqvo/pub?start=false&loop=false&slide=id.g2483f7076ac_0_147)

また、現代の Web サービス開発・運用では「マイクロサービス」と呼ばれる構造 (アーキテクチャ) がよく用いられます。

これは外向けには「一つの Web サービス」のような顔で提供しながらも、内部的には独立した複数の「小さな Web サービス」に分割して、それらを相互に連携させるアーキテクチャです。こうすることで、「小さな Web サービス」それぞれを、機敏に動ける独立した小さなチームで開発・運用できる、というメリットがあります。

ですが、「独立した」小さな Web サービスとはいったものの、他の「小さなサービス」の特性をまったく無視できるわけではないんですね。たとえばあるサービス A に「書いたデータを書いた直後に読める保証がない」としたら、その A を呼び出す他のサービス B もそのことを気に留めて設計しなければなりませんし、さらに B を呼び出すサービス C にもその制約は影響します。

### 制約はコードの「外」からやってくる [(p.23)](https://docs.google.com/presentation/d/e/2PACX-1vQf7dSmMDTBqQQgF-NcMqCssWv34BIVacK6_4xrMAIJnbqNXt65goIW0PhzfXIUSJf_SKgEmS5Ujqvo/pub?start=false&loop=false&slide=id.g2481c2dc2ec_0_30)

[![制約はコードの「外」からやってくる](/images/life-of-code-life-of-data-life-of-compatibility-023.png)](https://docs.google.com/presentation/d/e/2PACX-1vQf7dSmMDTBqQQgF-NcMqCssWv34BIVacK6_4xrMAIJnbqNXt65goIW0PhzfXIUSJf_SKgEmS5Ujqvo/pub?start=false&loop=false&slide=id.g2481c2dc2ec_0_30)

ここまでの議論をまとめましょう。一般にデータの寿命はプログラム (コード) よりも長いことが多く、コードは、コードに直接はあらわれないデータの制約を強く受けることがあります。そしてその種の制約は、さらに別のところから「伝搬」してやってくることもあるのです。

コードを直せばいいだけだったら難しくないことも多いのですが、「今までに蓄積されたデータを踏まえてどうするか」「データから来ている制約をどう捌くか」を考えるのが、ソフトウェア技術者の腕の見せ所でもあります。

ここで学生のみなさんに一つ注意してもらいたいのは、「プログラミング自体は難しくないならプログラミングをがんばる必要はない」わけではないことです。むしろ「こういう制約・こういう条件ならこうやればいいだろう」という手練手管の引き出しを、たくさん持っておく必要があります。

### 中銀 (なかぎん) カプセルタワービル [(p.24)](https://docs.google.com/presentation/d/e/2PACX-1vQf7dSmMDTBqQQgF-NcMqCssWv34BIVacK6_4xrMAIJnbqNXt65goIW0PhzfXIUSJf_SKgEmS5Ujqvo/pub?start=false&loop=false&slide=id.g2483f7076ac_1_2)

[![中銀 (なかぎん) カプセルタワービル](/images/life-of-code-life-of-data-life-of-compatibility-024.png)](https://docs.google.com/presentation/d/e/2PACX-1vQf7dSmMDTBqQQgF-NcMqCssWv34BIVacK6_4xrMAIJnbqNXt65goIW0PhzfXIUSJf_SKgEmS5Ujqvo/pub?start=false&loop=false&slide=id.g2483f7076ac_1_2)

ここで閑話休題。

写真 [^nakagin-wikimedia-jordy-meow] [^nakagin-wikimedia-dick-thomas-johnson] の「中銀カプセルタワー」というビルを見たこと・聞いたことがある人がいるかもしれません。

[^nakagin-wikimedia-jordy-meow]: [Wikimedia: Nakagin.jpg](https://commons.wikimedia.org/wiki/File:Nakagin.jpg) (CC-BY-SA-3.0 by Jordy Meow)
[^nakagin-wikimedia-dick-thomas-johnson]: [Wikimedia: Nakagin Capsule Tower (51474714434).jpg](https://commons.wikimedia.org/wiki/File:Nakagin_Capsule_Tower_(51474714434).jpg) (CC-BY-2.0 by Dick Thomas Johnson)

これは 1972 年に[黒川紀章](https://ja.wikipedia.org/wiki/%E9%BB%92%E5%B7%9D%E7%B4%80%E7%AB%A0)が設計・竣工した「[中銀カプセルタワービル](https://ja.wikipedia.org/wiki/%E4%B8%AD%E9%8A%80%E3%82%AB%E3%83%97%E3%82%BB%E3%83%AB%E3%82%BF%E3%83%AF%E3%83%BC%E3%83%93%E3%83%AB)」というビルで、部屋のひとつひとつが「カプセル」になっている集合住宅です。この講義の前年の 2022 年に解体されました。

このカプセルを「交換」して「新陳代謝」できるという、[メタボリズム](https://ja.wikipedia.org/wiki/%E3%83%A1%E3%82%BF%E3%83%9C%E3%83%AA%E3%82%BA%E3%83%A0)を象徴する設計・建築だった…のですが、解体まで、これが実際に「交換」されたことはありませんでした。

### NHK 「解体キングダム」 (2023年4月5日放送回) [(p.25)](https://docs.google.com/presentation/d/e/2PACX-1vQf7dSmMDTBqQQgF-NcMqCssWv34BIVacK6_4xrMAIJnbqNXt65goIW0PhzfXIUSJf_SKgEmS5Ujqvo/pub?start=false&loop=false&slide=id.g2483f7076ac_1_22)

[![NHK 「解体キングダム」 (2023年4月5日放送回)](/images/life-of-code-life-of-data-life-of-compatibility-025.png)](https://docs.google.com/presentation/d/e/2PACX-1vQf7dSmMDTBqQQgF-NcMqCssWv34BIVacK6_4xrMAIJnbqNXt65goIW0PhzfXIUSJf_SKgEmS5Ujqvo/pub?start=false&loop=false&slide=id.g2483f7076ac_1_22)

この中銀カプセルタワー解体工事の様子を (バラエティ風に) 収録した、「解体キングダム」 [^kaitai-kingdom-2023-04-05] という NHK の番組がありました。これがちょっと象徴的だったので、紹介させてください。

[^kaitai-kingdom-2023-04-05]: [解体キングダム「昭和の名建築 カプセルタワービル」 (NHK, 2023年4月5日初回放送)](https://www.nhk.jp/p/ts/JM3P4YLR7K/episode/te/WQJ23GZ6WZ/)

「カプセルごと交換できるようにできてるんだったら、解体も簡単なはずでは」と思うじゃないですか。これがまったくそんなことはなかったんですね。

構造としては個々のカプセルを持ち上げられそうに見えても、持ち上げる取っ手を付けるためのネジ穴が合わないとか、カプセルへの水道管などの配管が取り回せなくて外せないとか、一見すると「些末に見える」ところで次々と引っかかってまったく簡単にはいかなかったそうです。

これがソフトウェアの世界でも「あるある」な話で…というのが、この講義のここまでのお話でした。変更に強いように作ったつもりのソフトウェアが、現実には必須のインフラ (典型的にはデータ絡み) との接続や制約のせいでうまく変更できない、ということはとてもよくあります。

現在進行系で社会にあるもの、使われているもの、動いているものを「どうにか」するのは、ある面では、新しいものを作るのより大変なことが多いです。建築 (解体) とソフトウェアという異分野ながら、技術者として共感 (と胃痛) を覚える良い番組でした。 [^speaker-kaitai-kingdom]

[^speaker-kaitai-kingdom]: 免責事項: 話者はこの番組がかなり大好きです。参考: https://twitter.com/dmikurube/status/1665561068343820289

### 参考書籍: データ指向アプリケーションデザイン [(p.26)](https://docs.google.com/presentation/d/e/2PACX-1vQf7dSmMDTBqQQgF-NcMqCssWv34BIVacK6_4xrMAIJnbqNXt65goIW0PhzfXIUSJf_SKgEmS5Ujqvo/pub?start=false&loop=false&slide=id.g1f70bc4e279_2_6)

[![参考書籍: データ指向アプリケーションデザイン](/images/life-of-code-life-of-data-life-of-compatibility-026.png)](https://docs.google.com/presentation/d/e/2PACX-1vQf7dSmMDTBqQQgF-NcMqCssWv34BIVacK6_4xrMAIJnbqNXt65goIW0PhzfXIUSJf_SKgEmS5Ujqvo/pub?start=false&loop=false&slide=id.g1f70bc4e279_2_6)

また別の閑話休題として、参考書籍を一つ。

前述したような「規模が大きくなったとき」に対応できるデータベースの「中身」「裏側」はどうなっているのか、大規模データの専門家はなにを考えてそうしたデータベースを設計しているのか、という挑戦を見ることができる良書 "Designing Data-Intensive Applications" の日本語訳「データ指向アプリケーションデザイン」 [^designing-data-intensive-applications] があります。

[^designing-data-intensive-applications]: Martin Kleppmann 著、斉藤 太郎 監訳、玉川 竜司訳: [データ指向アプリケーションデザイン—信頼性、拡張性、保守性の高い分散システム設計の原理](https://www.oreilly.co.jp/books/9784873118703/) (O’Reilly, 2019年7月発行)

かなりアドバンストな内容ですが、輪読などで挑戦してみるといいかもしれません。

この講義の少し前に [Data Engineering Study](https://www.youtube.com/playlist?list=PL-zOB4NIHubaG_W63xg0dJCfm3C6GJEIz) という勉強会 (の第 18 回) があり、そこで監訳者の斉藤さん [^supervising-translator] 自身が解説をしているので、まずはそれを見てみるといいでしょう。

[^supervising-translator]: 免責事項: 斉藤さんは、この講義のときに話者が勤務していた Treasure Data の同僚でした。

@[youtube](ZiKWXc0fSCw)

## コードの寿命 v.s. 互換性の寿命

### データをあつかうのって大変そう… [(p.29)](https://docs.google.com/presentation/d/e/2PACX-1vQf7dSmMDTBqQQgF-NcMqCssWv34BIVacK6_4xrMAIJnbqNXt65goIW0PhzfXIUSJf_SKgEmS5Ujqvo/pub?start=false&loop=false&slide=id.g20a4119b7f6_0_44)

[![データをあつかうのって大変そう…](/images/life-of-code-life-of-data-life-of-compatibility-029.png)](https://docs.google.com/presentation/d/e/2PACX-1vQf7dSmMDTBqQQgF-NcMqCssWv34BIVacK6_4xrMAIJnbqNXt65goIW0PhzfXIUSJf_SKgEmS5Ujqvo/pub?start=false&loop=false&slide=id.g20a4119b7f6_0_44)

さてここからは少し話題を変えて、「単体ではデータを蓄積しないソフトウェア」についてです。

ここまでの話で「データをあつかうのって大変そう…」と思った人も多いかもしれません。実は話者はもともとそのクチで、このときの勤務先に来るまでは Java VM とか Web ブラウザとか、「単体ではデータを蓄積しないソフトウェア」を主にやる人でした。データベースこわい。

ではこっちは楽なのかというと…こっちはこっちで大変なんですよ、というお話です。

### 基盤となるソフトウェア [(p.30)](https://docs.google.com/presentation/d/e/2PACX-1vQf7dSmMDTBqQQgF-NcMqCssWv34BIVacK6_4xrMAIJnbqNXt65goIW0PhzfXIUSJf_SKgEmS5Ujqvo/pub?start=false&loop=false&slide=id.g249a8efc8e9_0_21)

[![基盤となるソフトウェア](/images/life-of-code-life-of-data-life-of-compatibility-030.png)](https://docs.google.com/presentation/d/e/2PACX-1vQf7dSmMDTBqQQgF-NcMqCssWv34BIVacK6_4xrMAIJnbqNXt65goIW0PhzfXIUSJf_SKgEmS5Ujqvo/pub?start=false&loop=false&slide=id.g249a8efc8e9_0_21)

「単体ではデータを蓄積しないソフトウェア」の例として OS や Web ブラウザ、コンパイラ (言語処理系) などを挙げていますが、このように「単体ではデータを蓄積しないソフトウェア」は、他のソフトウェアやサービスが動くための「基盤」となるソフトウェアであることが多いです。

つまり、アプリケーション・サービスのような「他の (誰かの) プログラム・コード」から呼び出されたり、または「他の (誰かの) プログラム・コード」そのものを入出力にしたりするソフトウェアですね。

コンパイラなら「他のプログラム」を入力として実行ファイルを出力したり、実行そのものをしたりします。 Web ブラウザなら HTML や JavaScript を入力して描画をしたり実行をしたり。データベースエンジンそのもの (DBMS) はデータの保存をしますが、それはあくまで「他のプログラムの指示を受けて」のデータ更新や検索です。

### データの制約はなくても、互換性の制約がある [(p.31)](https://docs.google.com/presentation/d/e/2PACX-1vQf7dSmMDTBqQQgF-NcMqCssWv34BIVacK6_4xrMAIJnbqNXt65goIW0PhzfXIUSJf_SKgEmS5Ujqvo/pub?start=false&loop=false&slide=id.g249a8efc8e9_0_7)

[![データの制約はなくても、互換性の制約がある](/images/life-of-code-life-of-data-life-of-compatibility-031.png)](https://docs.google.com/presentation/d/e/2PACX-1vQf7dSmMDTBqQQgF-NcMqCssWv34BIVacK6_4xrMAIJnbqNXt65goIW0PhzfXIUSJf_SKgEmS5Ujqvo/pub?start=false&loop=false&slide=id.g249a8efc8e9_0_7)

ここまでの話で、「データを蓄積するアプリケーション・サービス」でさえなければデータのしがらみに縛られずに開発ができる…、と思うじゃないですか。こうして「他のソフトウェアから使われる」ソフトウェアには、また別のしがらみ・制約があるのです。

「互換性」です。

C 言語のコンパイラを例に考えてみましょう。 C 言語というのは歴史の長いプログラミング言語で、既に 1980 年代には使われていました。そして現在でも Linux カーネルや Ruby などのプログラミング言語処理系、さらに PostgreSQL などのデータベースエンジンに至るまで、将来みなさんが使うことになるかもしれない多くのソフトウェアが C 言語で実装されていて、日々それらのコンパイルに使われています。

この「C 言語のコンパイラ」自体もまた、人が開発したソフトウェアです。そしてこのコンパイラというソフトウェアは、はるか昔の 1980 年代の C プログラムから、ついさっき書かれた C プログラムまで、どちらもコンパイルできなければなりません。

プログラミング言語というのは (あまり頻繁ではないにせよ) 新しい仕様・機能が追加されたりするものですが、その新しい仕様に対応するためだからといって「昔の C プログラムはもうコンパイルできませーん」と切り捨てるのは簡単なことではありません。

コンパイラの「中の人」の多くはよく、「『昔の仕様のアレさえなければ』全体をもっと効率よくできるのに、もっと簡潔になるのに」と思っていることでしょう。

自分の手元にあるコードはいくらでも書き換えられるんですが、残念ながら、一度書かれて世に出たプログラムは **自分の手の及ばないところでは** なかなかいなくなってくれないのです。

ヒト (人間) 向けのユーザー・インターフェースなら、少しくらい操作が変わってもヒトはどうにか慣れてくれます。 [^human-interface] ですが、基盤ソフトウェアの相手は (誰か他のヒトが書いた) プログラムですからね。そんな融通は効かせてくれません。

[^human-interface]: 使っているアプリの見た目や操作がいきなり変わると、ヒトのみなさんも文句の嵐になりがちですが。

### 基盤ソフトウェアと互換性問題は切っても切れない ([p.32](https://docs.google.com/presentation/d/e/2PACX-1vQf7dSmMDTBqQQgF-NcMqCssWv34BIVacK6_4xrMAIJnbqNXt65goIW0PhzfXIUSJf_SKgEmS5Ujqvo/pub?start=false&loop=false&slide=id.g249a8efc8e9_0_15))

[![基盤ソフトウェアと互換性問題は切っても切れない](/images/life-of-code-life-of-data-life-of-compatibility-032.png)](https://docs.google.com/presentation/d/e/2PACX-1vQf7dSmMDTBqQQgF-NcMqCssWv34BIVacK6_4xrMAIJnbqNXt65goIW0PhzfXIUSJf_SKgEmS5Ujqvo/pub?start=false&loop=false&slide=id.g249a8efc8e9_0_15)

基盤ソフトウェアと互換性問題は、いつも切っても切れない関係にあります。どんな基盤ソフトウェアも、歴史を積み重ねる中で、どこかで「互換性のしがらみを断ち切って発展させたい、使いやすくしたい」瞬間がやってくるからです。

特にプログラミング言語とその処理系にかかわる互換性は、よく話題・問題になるものです。有名どころでも Python 2 → 3 / Ruby 1.8 → 1.9 / Java 8 → 9 などがありました。

ここで永遠のジレンマは、「広く使われれば使われるほど互換性を壊しづらくなっていくが、そうして広く使われる前に『ああしておけばよかった』という問題が気づかれることはない」ということです。逆にいえば、「たいして使われてもいないソフトウェアに潜在的な問題があっても誰も気にしない」し、「たいして使われてもいないソフトウェアの互換性は少しくらい壊れても誰も気にしない」んですね。

長く広く使われた基盤ソフトウェアは、どこかで必ずこの問題に突き当たります。もしあなたの知っている基盤ソフトウェアが「互換性問題なんか無縁」のように見えているとしたら、それはたぶん「まだたいして使われてもいない」からでしょう。

前述のデータベースの話と組み合わせると、「データベース・エンジンの互換性問題」みたいな話は…考えたくない話ですね。 (でもよくあります)

### 基盤ソフトウェアがおこなう互換性対処: 失敗例 [(p.33)](https://docs.google.com/presentation/d/e/2PACX-1vQf7dSmMDTBqQQgF-NcMqCssWv34BIVacK6_4xrMAIJnbqNXt65goIW0PhzfXIUSJf_SKgEmS5Ujqvo/pub?start=false&loop=false&slide=id.g249a8efc8e9_0_48)

[![基盤ソフトウェアがおこなう互換性対処: 失敗例](/images/life-of-code-life-of-data-life-of-compatibility-033.png)](https://docs.google.com/presentation/d/e/2PACX-1vQf7dSmMDTBqQQgF-NcMqCssWv34BIVacK6_4xrMAIJnbqNXt65goIW0PhzfXIUSJf_SKgEmS5Ujqvo/pub?start=false&loop=false&slide=id.g249a8efc8e9_0_48)

では現存する基盤的なソフトウェアは、どのように「互換性を切りたい」問題に対処してきたのでしょう。まずは「失敗例」を見てみます。

あるプログラミング言語 (言語仕様および処理系) のバージョン 1 とバージョン 2 で互換性を壊して、「あるバージョン 1 の書き方は 2 では動かない」のと同時に、「新しい書き方はバージョン 2 にしかないので 1 では動かない」ようにしたとしましょう。するとその言語・処理系を使う開発者に対しては、「みんな (一斉に) 書き換えてね」と言うことになります。

「それはしかたないんじゃないの?」と思うじゃないですか。

[![基盤ソフトウェアがおこなう互換性対処: 失敗例](/images/life-of-code-life-of-data-life-of-compatibility-034.png)](https://docs.google.com/presentation/d/e/2PACX-1vQf7dSmMDTBqQQgF-NcMqCssWv34BIVacK6_4xrMAIJnbqNXt65goIW0PhzfXIUSJf_SKgEmS5Ujqvo/pub?start=false&loop=false&slide=id.g249a8efc8e9_0_70)

現代のソフトウェアはほとんど「誰か他の人が作った他のライブラリ・ソフトウェア」に依存しています。ソフトウェアやライブラリが他のライブラリに依存しながらできる、こういう体系のことを「エコシステム (生態系)」と呼んだりします。

それぞれのライブラリは、それぞれ別の人 (たち) が開発し、それぞれの事情でメンテナンスされ、それぞれにバージョン管理されています。すると、あるライブラリ A の新機能を使いたければそのライブラリ A の新しいバージョンを使うしかないのに、その A の新しいバージョンは処理系のバージョン 2 でしか動かない、みたいなことが起こります。

しかたないからまずは自分のコードを処理系バージョン 2 対応にさせようとします。それで済めば幸運なのですが、えてしてライブラリ A と並行して使っているライブラリ B は最新版でも処理系バージョン 2 では動かない、みたいなことも起こります。こうなると、やっぱり自分のコードも処理系バージョン 2 には上げられない、という「がんじがらめ」が起きるんですね。

そのプログラミング言語・処理系のエコシステムが一度こうなってしまうと、ライブラリの開発者がみんな処理系バージョン 1 にとどまってしまい、誰も先陣を切れずに様子見が続く…、なんてチキンレースみたいな状況が起きることが本当にあります。

### 基盤ソフトウェアがおこなう互換性対処: 一例 [(p.35)](https://docs.google.com/presentation/d/e/2PACX-1vQf7dSmMDTBqQQgF-NcMqCssWv34BIVacK6_4xrMAIJnbqNXt65goIW0PhzfXIUSJf_SKgEmS5Ujqvo/pub?start=false&loop=false&slide=id.g249a8efc8e9_0_154)

[![基盤ソフトウェアがおこなう互換性対処: 一例](/images/life-of-code-life-of-data-life-of-compatibility-035.png)](https://docs.google.com/presentation/d/e/2PACX-1vQf7dSmMDTBqQQgF-NcMqCssWv34BIVacK6_4xrMAIJnbqNXt65goIW0PhzfXIUSJf_SKgEmS5Ujqvo/pub?start=false&loop=false&slide=id.g249a8efc8e9_0_154)

さて、基盤ソフトウェアの開発者は、エコシステムがこうならないために何ができるでしょうか? ここでは一例として、話者がメンテナンスしているオープンソース・ソフトウェアの [Embulk](https://www.embulk.org/) でどういうことをしたのか、取り上げてみます。

Embulk は Treasure Data の共同創業者である[古橋さん](https://github.com/frsyuki)が開発した、テーブル (行-列) 形式のデータを "あっち" から "こっち" へロードするツールで、ロード元とロード先を「プラグイン」で切り替えられるものです。 (話者はメンテナンスを引き継ぎました)

この「プラグイン」は、それぞれ別の人が開発・メンテナンスしています。

[![基盤ソフトウェアがおこなう互換性対処: 一例](/images/life-of-code-life-of-data-life-of-compatibility-036.png)](https://docs.google.com/presentation/d/e/2PACX-1vQf7dSmMDTBqQQgF-NcMqCssWv34BIVacK6_4xrMAIJnbqNXt65goIW0PhzfXIUSJf_SKgEmS5Ujqvo/pub?start=false&loop=false&slide=id.g249a8efc8e9_0_195)

他の人が開発した「プラグイン」を Embulk の上で動かすので、これも一種の基盤ソフトウェアと見ることができます。つまり、ここでも互換性は問題になります。

プラグインは [SPI (Service Provider Interface)](https://ja.wikipedia.org/wiki/%E3%82%B5%E3%83%BC%E3%83%93%E3%82%B9%E3%83%97%E3%83%AD%E3%83%90%E3%82%A4%E3%83%80%E3%82%A4%E3%83%B3%E3%82%BF%E3%83%95%E3%82%A7%E3%83%BC%E3%82%B9) というインターフェースを介して Embulk の本体と呼び出し関係にあります。この SPI をとおした呼び出しの規則が変わると互換性が壊れ、プラグインは動かなくなってしまいます。

互換性を壊さずに済めばそれに越したことはないのですが、そのためには重い足かせを引きずり続ける必要があります。

足かせを引きずり続ける選択をする基盤ソフトウェアもありますが、この Embulk では互換性を保ったまま維持・発展を続けることは難しいという結論になり、互換性を壊す意思決定をしました。ちょうどこの講義があった 2023 年 6 月は、この互換性にかかわる一連の変更の最終段階をリリースするところでした。 [^embulk-0-11]

[^embulk-0-11]: その Embulk v0.10 〜 v0.11 の話は、また[別の記事](https://zenn.dev/dmikurube/articles/embulk-v0-11-is-coming-soon-ja)にまとめてあります。

[![基盤ソフトウェアがおこなう互換性対処: 一例](/images/life-of-code-life-of-data-life-of-compatibility-037.png)](https://docs.google.com/presentation/d/e/2PACX-1vQf7dSmMDTBqQQgF-NcMqCssWv34BIVacK6_4xrMAIJnbqNXt65goIW0PhzfXIUSJf_SKgEmS5Ujqvo/pub?start=false&loop=false&slide=id.g249a8efc8e9_0_226)

ここで先に挙げた失敗例のように「みんな (一斉に) 書き換えてね」とやったら…。

[![基盤ソフトウェアがおこなう互換性対処: 一例](/images/life-of-code-life-of-data-life-of-compatibility-038.png)](https://docs.google.com/presentation/d/e/2PACX-1vQf7dSmMDTBqQQgF-NcMqCssWv34BIVacK6_4xrMAIJnbqNXt65goIW0PhzfXIUSJf_SKgEmS5Ujqvo/pub?start=false&loop=false&slide=id.g24bb1541be9_0_67)

同じことになってしまいます。

Embulk には「Embulk 本体の開発者」「各 Embulk プラグインの開発者」「本体とプラグインのユーザー」という三者がいます。

「ユーザー」から見たときに「あるプラグインの新バージョンにある機能を使いたいのに同時に使う他のプラグインは新しい Embulk にまだ対応していない」となるようなら、「プラグイン開発者」は新 Embulk 対応を躊躇してしまいます。すると、ここでもチキンレースが始まってしまうでしょう。

[![基盤ソフトウェアがおこなう互換性対処: 一例](/images/life-of-code-life-of-data-life-of-compatibility-039.png)](https://docs.google.com/presentation/d/e/2PACX-1vQf7dSmMDTBqQQgF-NcMqCssWv34BIVacK6_4xrMAIJnbqNXt65goIW0PhzfXIUSJf_SKgEmS5Ujqvo/pub?start=false&loop=false&slide=id.g249a8efc8e9_0_162)

そうならないために Embulk では、一定期間「プラグインを旧 Embulk と新 Embulk の両方で動かせるプラグインの構築方法」を用意する、というアプローチを取りました。

こうすることで、プラグインの開発者に「とりあえず『両方で動く状態』までは持っていっても誰も困らない」と考えてもらうことができ、少なくとも、「様子見のチキンレース」が起こる構造的な問題は避けることができました。また多くのプラグインをメンテナンスしなければならないプラグイン開発者も、すべてを一斉に移行する必要なく「少しずつ」対応を進めることができました。

開発者が興味を失ってメンテナンスされなくなってしまったプラグインは、新しい Embulk で動かなくなるかもしれません。ですが、もうこれはどうしようもないと (明示的に) あきらめました。このように「どこまではカバーする」「どこからはあきらめる」というラインを決めて明示するのも、互換性対処としては重要なポイントです。

[![基盤ソフトウェアがおこなう互換性対処: 一例](/images/life-of-code-life-of-data-life-of-compatibility-040.png)](https://docs.google.com/presentation/d/e/2PACX-1vQf7dSmMDTBqQQgF-NcMqCssWv34BIVacK6_4xrMAIJnbqNXt65goIW0PhzfXIUSJf_SKgEmS5Ujqvo/pub?start=false&loop=false&slide=id.g249a8efc8e9_0_249)

エコシステムには複数のコンポーネントがかかわり、それぞれのコンポーネントにはそれぞれの「中の人」がいます。そのすべてのコントロールを握っている人は誰もいないところで、それらをいかに「協調的に」移行するか、という点が課題になります。

「ビッグバン・アップデート」のような言いかたをよくしますが、「一度に全部を移行する」のは破綻しがちなんですね。上げられないものが一つでもあると全体がそこでロックしてしまいますし、一度に多くのものを変えると動かなかったときに問題と原因の特定が非常に大変になります。

[![基盤ソフトウェアがおこなう互換性対処: 一例](/images/life-of-code-life-of-data-life-of-compatibility-041.png)](https://docs.google.com/presentation/d/e/2PACX-1vQf7dSmMDTBqQQgF-NcMqCssWv34BIVacK6_4xrMAIJnbqNXt65goIW0PhzfXIUSJf_SKgEmS5Ujqvo/pub?start=false&loop=false&slide=id.g249a8efc8e9_0_272)

そこで小さい単位で一つずつ移行できるようにしておくと、移行作業全体がロックすることも減り、事故があったときも原因の特定が容易になるわけです。

### 制約はコードの「外」からやってくる (パート2) ([p.42](https://docs.google.com/presentation/d/e/2PACX-1vQf7dSmMDTBqQQgF-NcMqCssWv34BIVacK6_4xrMAIJnbqNXt65goIW0PhzfXIUSJf_SKgEmS5Ujqvo/pub?start=false&loop=false&slide=id.g2483f7076ac_0_307))

[![制約はコードの「外」からやってくる (パート2)](/images/life-of-code-life-of-data-life-of-compatibility-042.png)](https://docs.google.com/presentation/d/e/2PACX-1vQf7dSmMDTBqQQgF-NcMqCssWv34BIVacK6_4xrMAIJnbqNXt65goIW0PhzfXIUSJf_SKgEmS5Ujqvo/pub?start=false&loop=false&slide=id.g2483f7076ac_0_307)

後半の話をまとめます。データに縛られないソフトウェアも、今度は「互換性」に縛られることがあります。特に、他のソフトウェアから呼び出され依存される「基盤ソフトウェア」は、その傾向が強くなります。

そのソフトウェアのコードだけを見て直したいように直せるとはかぎらない、という点では、ここでもデータに縛られるのとも似たような縛りがあるのです。

この互換性対処についても、いろいろな手練手管があります。この講義で話した Embulk のやりかたはあくまで一つの例でしかありませんし、講義ではごく表面的なアプローチについてしか話せていません。その中身は、細かい技術的な工夫を積み上げることで成立しています。

「どういう (利害) 関係者がいるのか」「どういう開発者がいるのか」「どういうユーザーがいるのか」「どういう形で使われているのか」を把握・想像しながら、手札の手練手管をたぐって、どういうアプローチをとるのかを常に考え続ける必要があるでしょう。

# まとめ

この講義全体をとおして、長く使われるソフトウェア・サービスの仕事においては、「目の前のコードのことだけ考えて開発ができることは多くないよ」「データや互換性などから来るしがらみがあるよ」「情報科学についての基礎的な理解やプログラミングの様々な手練手管はそれでもやっぱり必要だよ」というお話をしてきました。

データであれば「そのデータの持ち主」という『他者』、そして互換性であれば「インターフェースをとおしてやりとりする相手」という『他者』 (の開発するソフトウェア) がいます。もちろん同じチームで一緒に働く仲間という『他者』もいるでしょう。

ただコードに向き合うだけのように思われがちなソフトウェアの仕事ですが、ビジネスとしてやる以上、どこかで、なにかしらの形で、誰かしらの『他者』と、このようにエコシステムなどをとおして間接的に、またはもっと直接的に、かかわることになります。

前半で話したような「データを蓄積するサービス」にかかわる仕事をする人、特に長生きするサービスに携わる人は、思った以上にデータの制約が大きいことが気になってくるでしょう。データベースから距離のあるコンポーネント (たとえばフロントエンド) の仕事をしていると直接は見えないかもしれませんが、データベースに近いコンポーネント (たとえばバックエンド) を担当するチームメイトと相談する中で、「なにか制限があるのだな」と気がつくことがあるかもしれません。そんなとき、データベースのことまで突っ込んだ話ができれば、全体をよりよくするための議論につながるかもしれません。

後半で話したような「基盤ソフトウェア」を仕事にする人は、その基盤ソフトウェアの「ユーザー (開発者)」と、陰に陽にかかわります。自分がコードにくわえる変更が、その「ユーザーのコード」にどう影響を与えるのか、気を配りながら開発をすることになるでしょう。基盤ソフトウェアの仕事は、人とあまりかかわらずに済みそうな印象を持つ人も多いのですが、実は『他者』のことを常に気にかけなければならない仕事だったりします。

「基盤ソフトウェア」を仕事にする人は多くないかもしれませんが、直接的にはそういう仕事に就かなくても、どこかでなにかしらの基盤ソフトウェアを使うことは間違いありません。その基盤ソフトウェアはそのへんの地面に生えているわけではなく、その開発・メンテナンスをしているのはやはり「人」です。それぞれの基盤ソフトウェアは、互換性などの事情を抱えながらそれぞれに開発をしています。そのことを想像すると、基盤ソフトウェアをよりよく使うこと、そして必要になったときには、よいコミュニケーションを取ることができるかもしれません。

ソフトウェアの仕事も、近くや遠くの『他者』と常にかかわっているのだ、と意識できるようになると、よりよい仕事ができるようになるかもしれません。ソフトウェアの仕事に就く学生の皆さんの活躍をお祈りしています。
