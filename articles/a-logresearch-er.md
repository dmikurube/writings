---
title: "或るログ研究者"
emoji: "🪵" # アイキャッチとして使われる絵文字（1文字だけ）
type: "idea" # tech: 技術記事 / idea: アイデア
topics: [ "opensource", "embulk", "community", "github" ]
layout: default
published: true
---

[Embulk](https://www.embulk.org/) というオープンソース・ソフトウェアのメンテナーをやっていました。

「やっていました」と過去形なのは、いちおうまだメンテナー権限を持ってはいるものの、プロジェクトは「メンテナンス・モード」として実質的にメンテナンスを終了しているし、個人としてもなかば降りているからです。

@[card](https://zenn.dev/dmikurube/articles/how-to-wind-down-an-opensource-project)

@[card](https://www.embulk.org/articles/2025/04/11/dmikurube-is-stepping-down.html)

今から二年ほど前の 2024 年 7 月、この Embulk に、ある pull request (以下 PR) が届きました。この PR は、私がなかばメンテナーを降り、プロジェクトを「メンテナンス・モード」とすることにした理由の一つだったと思います。この PR をきっかけに、オープンソース活動についていろいろ考えてしまったんですね。

当時は、この PR についておおっぴらには言及するのを避けていました。この背景について証拠や確信はなかったので、さらすような真似はやめておこうと思ったからです。 [^ex-twitter]

[^ex-twitter]: とはいえ、わかる人ならすぐたどりつける程度には、旧 Twitter などでボヤいてましたが。

ですが、それから二年以上が過ぎました。もう Embulk もメンテナンス・モードに入ったし、その PR の主は活動を止めたようだし、あった事実を共有してもいい頃合いかなと考えます。

記事の原型は、当時から書きかけていたものです。二年が経って世の中は大きく変わり、いまさらな部分もあるでしょう。出さずに封印しようかとも思いましたが、「2024 年の時点でこんなことがあったんだよ」と記録を残す程度の意味はあるかと、成仏させる気持ちも込みで公開するものです。

オープンソース活動の痛み
=========================

もし本記事の推測がまったくの誤解だったら、当該アカウントの「中の人」にはごめんなさい。

ただ、複数の利害関係者がいるオープンソース・プロジェクトのメンテナンスって、特に近年はけっこうセンシティブなんですよ。

たとえば「オープンソース開発に参加しよう!」と多くの人に呼びかける [Hacktoberfest](https://hacktoberfest.com/) のような活動があります。また、少しでも改善の余地を見つけると「プルリクチャンス!」と雄叫びを上げる人たちも見かけますね。

ですが、すべてのオープンソース・プロジェクトとそのメンテナーが、このような活動を歓迎しているわけではありません。 [^hacked-off] 私自身も、「みんな typo 修正から気軽にプルリクにチャレンジしてみよう!」みたいなキャンペーンには、強く懐疑的な立場をとっています。

[^hacked-off]: 「[Hacktoberfestで悩まされる (Hacked Off)](https://www.infoq.com/jp/news/2020/12/hacked-off-hacktoberfest/)」 (作者: Alex Blewitt, 翻訳者: Shoji Shigeki, 2020 年 12 月 5 日)

オープンソース・ソフトウェアのメンテナーは、やってくる PR に対して、想像以上にいろいろなことを考えていますし、考える必要があります。それだけでもけっこう疲れるし、「もうやってらんねーよ」と感じているオープンソース開発者も、特に近年は多いのです。

この件に対する私の推測が正しかったのか誤解だったのかは、今でも正直わかりません。ただ、もし誤解であったとしても「近年はメンテナーがこういうことまで考えざるをえない環境になっている」ことは確かです。そしてその傾向は、この二年でさらに加速しています。

実際にあったことの振り返り
===========================

2024-07-02 (火)
----------------

さて、問題の PR がこちらです。

@[card](https://github.com/embulk/embulk/pull/1678)

見てもらうとわかりますが、これ自体は、別におかしな提案ではありません。言ってることも間違ってはいないし、もしかしたら私も「なるほどご親切にどうも」と素直にマージしていたかもしれません。

でも、ちょっと違和感もありますね。たとえば…

* このたった一行のために、ここまでしっかりした Description を書くものだろうか?
* "Original Code" と "Updated Code" ってあるけど、これ変更そのものだよね? 必要ある?
* …というか、この PR の主の `logresearch` さんって、誰?

違和感を覚えてしまったからには、それを見なかったことにしてこの PR をマージするわけにもいきません。

### 誰?

まずはこの `logresearch` さんの活動を、軽く眺めてみることにしました。アカウント名の `logresearch` 以外に名前は設定されていなかったので、本記事ではこのアカウントのことをログリサーチ＝サンとでも呼ぶことにしましょう。

ログリサーチ＝サンはどうやら、この PR を送る数日前にできたばかりの新しい GitHub アカウントで、あちこちの著名な Java プロジェクトに、ログ処理に関係する小規模な変更を送っていたようです。一つのプロジェクトに対して、だいたい一つずつ。

@[card](https://github.com/88250/symphony/pull/94)

@[card](https://github.com/TeamAmaze/AmazeFileManager/pull/4208)

@[card](https://github.com/apereo/cas/pull/6075)

@[card](https://github.com/asciidocfx/AsciidocFX/pull/648)

@[card](https://github.com/embulk/embulk/pull/1678)

@[card](https://github.com/openmrs/openmrs-core/pull/4680)

@[card](https://github.com/resteasy/resteasy/pull/4220)

これを見て、警戒レベルが上がりました。

近年では「新しく作られたばかりのアカウント」は、どこのソーシャルなサービスでも警戒対象の一つです。新しい GitHub アカウントを作って最初の活動がそれと見れば、なおさらです。

### なんのために?

次に気になったのは PR の送り方です。

ログリサーチ＝サンは、それぞれのプロジェクトから修正箇所の候補を見つけるために、おそらくそれぞれのコードベースでなにかしらの探索をしたことでしょう。それなりの規模のコードベースを探索して、見つかる修正候補が一つだけとは考えにくい。

そのプロジェクトへの「貢献」を目的に探索したのなら、一つだけ PR を出して他のプロジェクトに行く前に、同じプロジェクトの中で見つかった他の点についても、少しは言及するものではないでしょうか。

ここから、ログリサーチ＝サンはおそらく「対象のプロジェクトに貢献したい」よりも「より多くのプロジェクトに関与したい」動機を持っていた、と推測しました。

`logresearch` というユーザー名からありそうなのは、どこかの研究者か、研究室の学生か、とかでしょうか。

たとえば、論文でアピールするために貢献プロジェクトの数を稼ぎたい、ということはしばしばあります。もし本当にそうだと確認できればマージしてもよかったんですが、それなら「自分 (たち) はどこそこの研究機関の誰それで」くらい名乗るのが、常識であり礼儀でしょう。 [^researcher] その点で、まだ少し違和感がありました。

[^researcher]: 仮にその時点では公開できない研究プロジェクトだったとしても、非公開のメールなどで名乗るものでしょう。これが本職の研究者だったら、プロ失格だと私は思います。研究指導を受ける学生だったら、失格なのはその指導教員ですね。もし研究機関が判明したら、そこの研究倫理審査委員会などに通知を考えるレベルです。

しかもこの仮定は、あくまで攻撃的ではない研究という前提の、さらに好意的な解釈でしかありません。

たとえば大学の研究であっても、過去には実際に研究倫理の面で問題になった事件もありました。 [^hypocrite-commits]

[^hypocrite-commits]: 「[Linuxテクニカルアドバイザリーボード、ミネソタ大の意図的な脆弱性混入問題に関するレポート公表](https://japan.zdnet.com/article/35170458/)」 (作者: Steven J. Vaughan-Nichols, 翻訳校正: ZDNET 編集部, 2021 年 5 月 10 日)

さらに悪意を仮定すれば、より悪い可能性もありました。つまり悪意を持って攻撃対象にできるプロジェクトを探していた疑いです。

### Remember XZ

「せっかくの貢献に対して、悪いほうに考え過ぎじゃない?」と感じた人もいるかもしれません。私も実際、このタイミングでさえなければ、素直にマージしていたかもしれません。

ただ、この PR が送られてきたのは 2024 年 7 月です。

そのわずか数ヶ月前の 2024 年 3 月、「Jia Tan と名乗る開発者が約 3 年かけて XZ Utils のメンテナーの『信頼』を得た末に XZ Utils に仕掛けたバックドアがリリース直前に発見される」という事件があったことを覚えているでしょうか。

@[card](https://ja.wikipedia.org/wiki/XZ_Utils%E3%81%AE%E3%83%90%E3%83%83%E3%82%AF%E3%83%89%E3%82%A2)

この PR が将来的なサプライチェーン攻撃 [^supply-chain-attack-cloudflare] を狙っていた可能性は、看過できません。

[^supply-chain-attack-cloudflare]: Cloudflare: [サプライチェーン攻撃とは？](https://www.cloudflare.com/ja-jp/learning/security/what-is-a-supply-chain-attack/)

Embulk は、当時いくつかの企業のデータ基盤でそれなりに大規模に使われていて、現実のデータを捌いていました。 [^still-used]

[^still-used]: ちなみに、メンテナンス・モードに入った最近でも Embulk を使っているような話を見聞きしますが、おすすめはしません。

Embulk は、あくまでオープンソースとして [Apache License, Version 2.0](https://www.apache.org/licenses/LICENSE-2.0) でライセンスされたソフトウェアです。仮にセキュリティ事故が起きても、プロジェクトとしてはなにも保証する必要はありません。

だからといって、攻撃の可能性を疑ったものを無警戒に受け入れるわけにも、いかないでしょう。

### 攻撃の可能性

ここで難しいのは、「攻撃かもしれない」という疑いの目でどれだけ見ても、この PR 単体は明らかに潔白なんですよ。なので、コミュニティ・ベースのオープンなプロジェクトという体裁からは、即座に却下するのもよくはありません。

これが悪意による PR だったと仮定すると、どんな攻撃がありえたでしょうか?

一つは XZ と同様の「気の長い」攻撃の可能性でした。これはもう、この PR 単体からは判断しようがありません。ただ、これを狙っていたとしたら、「一つの新規アカウントから複数のプロジェクトに PR を投げる」という、わかりやすく怪しまれる方法を選ぶかなあ…? という気はします。

他の可能性に「GitHub Actions 実行権限の取得」がありました。 GitHub Actions には "Require approval for first-time contributors" [^github-actions-settings-for-a-repository] ( ≒ 一度でも PR をマージしたユーザーからの次回以降の PR では GitHub Actions の自動実行を許す) という設定項目があり、実はこれが (当時の) GitHub のデフォルトでした。これを利用して、複数のリポジトリで GitHub Actions 実行権限の奪取を狙っていた、という可能性も考えられました。

[^github-actions-settings-for-a-repository]: GitHub: [Managing GitHub Actions settings for a repository](https://docs.github.com/en/repositories/managing-your-repositorys-settings-and-features/enabling-features-for-your-repository/managing-github-actions-settings-for-a-repository)

ただいずれも、受け入れる材料としても却下する材料としても、ちょっと弱い。

前者なら、ひとたび受け入れれば長期的に警戒を続ける必要があります。ですがこのご時世では、いずれにせよ長期的な警戒が必要という点で、あまり変わらないかもしれません。

後者なら GitHub の設定の問題ですから、穴をふさいで受け入れてもいいかもしれません。ですが、他に同様の穴があるのを見落としているかもしれません。

どうも決定打に欠けるので、この日はなんの反応もせず、保留することにしました。

メンテナーは、決定もコミュニケーションも意図的に保留することがあります。

ところで「せっかく PR を送ったのにメンテナーから反応がない〜!!」のように騒ぎ立てる人を見ることがありますね。そういう人たちが、メンテナーの視点からどう見られているのかは、想像してみてもいいかもしれません。

### 他のプロジェクトでは

ログリサーチ＝サンの PR が届いた他のプロジェクトは、どう対応したでしょうか?

この 2024 年 7 月 2 日の時点では、直後にマージしたプロジェクトはなかったようです。

一方で即座に無言 close したプロジェクトもあったり。

@[card](https://github.com/88250/symphony/pull/94)

「bot っぽいから CI の権限を渡したくない」と close したプロジェクトがあったり。

> ```
> This appears to be a bot that I do not want to give permissions to run CI.
> ```
>
> https://github.com/resteasy/resteasy/pull/4220#issuecomment-2203420669

@[card](https://github.com/resteasy/resteasy/pull/4220#issuecomment-2203420669)

bot が「テストを書け」と自動 close したプロジェクトがあったり。

@[card](https://github.com/apereo/cas/pull/6075#issuecomment-2202859857)

ログリサーチ＝サンが「ただの typo fix だよ! テスト要らないよ!」と反論していたり。

> ```
> This is a typo fix, which generally can be accepted without proper tests.
> ```
>
> https://github.com/apereo/cas/pull/6075#issuecomment-2202878419

> ```
> Hi, @mmoayyed
> It's a simple fix on logging messages. The bot closes it as there are no tests.
> Nothing to test here. Thanks.
> ```
>
> https://github.com/apereo/cas/pull/6075#issuecomment-2202911281

いくつかの対応を見ることができました。

最後のケースのように詰め寄る様子も、なんだか XZ 事件を思い出させます。

どうにもあやしいんだけど、悪意だと断定できるほどの情報はない。

マナーのなっていない善意の研究者という可能性も、依然として否定しきれない。オープンソース・コミュニティの開発者としては、善意の貢献を無下にはしたくないけど、背景に悪意があるなら却下したい。

こういうことを考えるのがいちばん疲れます。コードの良し悪しとか理想のアーキテクチャみたいな与太話でわちゃわちゃ言ってるほうが、何倍も楽しいし気が楽なんですけどねえ。

2024-07-03 (水)
----------------

翌日。他のプロジェクトを再び見に行ってみると、いくつかの変化がありました。

同じく bot が close したプロジェクトに、ログリサーチ＝サンが「ぼくわるい bot じゃないよ!」と食い下がってみたり。

> ```
> We are not bot!
> We are the researcher who focus on promoting the quality of logging statement.
> Thank you if you give us a chance to use our automated detected tool to real world data.
> ```
>
> https://github.com/resteasy/resteasy/pull/4220#issuecomment-2203525995

@[card](https://github.com/resteasy/resteasy/pull/4220#issuecomment-2203525995)

「必要な変更ではないでしょ」とメンテナーが close していたり。

> ```
> thanks for your interest in contributing to amaze. but, I am afraid I don't think this is a mandatory change to be made.
> ```
>
> https://github.com/TeamAmaze/AmazeFileManager/pull/4208#issuecomment-2203554405

ログリサーチ＝サンがそれにまた食い下がり、自動メンテナンスの研究なんだみたいなことをやっぱり言っていたり。

> ```
> thanks for your interest in contributing to amaze. but, I am afraid I don't think this is a mandatory change to be made.
>
> Hi @VishnuSanal ,
>
> Thanks for your quick reply.
> We are doing research on automatically maintain the quality of logging statement.
> We would be grateful if the developers for popular repositories think it's useful.
> We are looking forward for your opinion for automatically LS quality maintenance!
> ```
>
> https://github.com/TeamAmaze/AmazeFileManager/pull/4208#issuecomment-2203595463

食い下がった結果 approve されてみたり。

> ```
> Looks fine to me. Any code improvements are welcome whether it's for readability or actually fixes / issues.
> ```
>
> https://github.com/TeamAmaze/AmazeFileManager/pull/4208#issuecomment-2203891031

approve されたプロジェクトには、さらに PR を上乗せしてみたり。

> ```
> Hi @VishalNehra,
>
> Thanks for your positive reply! Really hope my contribution can make AmazeFileManagermore perfect, though the contribution is incremental. Based on the feedback, I further reviewed some source code and logging messages and found the following inaccurate expression.
>
> https://github.com/TeamAmaze/AmazeFileManager/blob/release/4.0/app/src/main/java/com/amaze/filemanager/asynchronous/loaders/AppListLoader.java#L87 `LOG.warn("faield to find android package name while loading apps list", e);` The `faield` is a typo, which should be `failed`
>
> Also the same in https://github.com/TeamAmaze/AmazeFileManager/blob/release/4.0/app/src/main/java/com/amaze/filemanager/asynchronous/loaders/AppListLoader.java#L102 `faield` -> `failed`
>
> We update them in this PR together.
> ```

@[card](https://github.com/TeamAmaze/AmazeFileManager/pull/4208)

そしてログリサーチ＝サン発の PR が、さらに多くのプロジェクトに拡大していたり。

@[card](https://github.com/alfio-event/alf.io/pull/1370)

@[card](https://github.com/ant-media/Ant-Media-Server/pull/6457)

@[card](https://github.com/azkaban/azkaban/pull/3333)

@[card](https://github.com/opensourceBIM/BIMserver/pull/1320)

@[card](https://github.com/dbeaver/dbeaver/pull/34625)

@[card](https://github.com/NationalSecurityAgency/ghidra/pull/6688)

@[card](https://github.com/microcks/microcks/pull/1236)

@[card](https://github.com/sonatype/nexus-public/pull/424)

@[card](https://github.com/openhab/openhab-addons/pull/16984)

@[card](https://github.com/openhab/openhab-addons/pull/16989)

@[card](https://github.com/pravega/pravega/pull/7431)

close したプロジェクト、無反応のプロジェクト、マージしたプロジェクト、それぞれです。対応が分かれました。

少し興味深かったのは、ログリサーチ＝サンは CLA (Contributor License Agreement) を求められて、ちゃんとサインしたらしいところですね。

> ```
> All committers have signed the CLA.
> ```
>
> https://github.com/camunda/camunda-bpm-platform/pull/4471#issuecomment-2205175993

@[card](https://github.com/camunda/camunda-bpm-platform/pull/4471#issuecomment-2205175993)

もう一つ別のプロジェクトの CLA にもサインしていて、これはメールからサインしたらしく、その証跡がコメントに残されていたりもします。 [^xie-dao]

[^xie-dao]: ログリサーチ＝サンの言語圏をコメントから推察できてしまいますが、その推察自体に大した意味はないでしょう。ただ、なにも言及しないと、却ってあーだこーだ言い始める人がわき出てきそうなので、牽制の意味で言及しておきます。

> ```
> ￼
> […](#)
> 2024年7月3日 18:51，alluxio-bot ***@***.***> 写道： Thank you for your pull request. In order for us to evaluate and accept your PR, we ask that you sign a contribution license agreement (CLA). It's all electronic and will take just a few minutes. Please download CLA form here <https://www.alluxio.io/app/uploads/2022/07/Contribution-License-Agreement-2022-Template.pdf>, sign, and e-mail back to ***@***.*** ***@***.***> — Reply to this email directly, view it on GitHub <[#18647 (comment)](https://github.com/Alluxio/alluxio/pull/18647#issuecomment-2205780860)>, or unsubscribe <https://github.com/notifications/unsubscribe-auth/BJSXPZAWPO3WIW3UTXG3MTLZKPJT7AVCNFSM6AAAAABKJJ75P2VHI2DSMVQWIX3LMV43OSLTON2WKQ3PNVWWK3TUHMZDEMBVG44DAOBWGA>. You are receiving this because you authored the thread.
> ```
>
> https://github.com/Alluxio/alluxio/pull/18647#issuecomment-2205799828

@[card](https://github.com/Alluxio/alluxio/pull/18647#issuecomment-2205799828)

### AI ?

ここまであえて AI に触れずに書いてきたのですが [^organic] これを 2026 年現在から振り返って読む人は「AI slop でしょ」の一言で終わりかもしれません。

[^organic]: 本記事はほぼ AI を使わずにお送りしています。途中まで書きかけでしたから。ただ、全体をだいたい書き切ってから Claude くんに「感想を聞かせて」と頼んで、その指摘をもとに加筆は少ししています。具体的には、上で触れた Hypocrite Commits の件などです。

私も、当時から「それ (AI) っぽいなあ」と思っていましたし、今も「たぶんそう (AI) だったんじゃないかな」とは思っています。

2026 年現在では、もうこれくらい当たり前のことだと感じてしまいます。しかしこれは 2024 年 7 月の出来事でした。

当時の常識を思い出すのも既に難しくなっていますが、この 2024 年 7 月とは、かの「[CLINEに全部賭けろ](https://zenn.dev/mizchi/articles/all-in-on-cline)」より半年以上も前のことです。

@[card](https://zenn.dev/mizchi/articles/all-in-on-cline)

これを AI が普通にできるという認識は、当時そこまで一般的ではなかったように思います。 [^yet-possible]

[^yet-possible]: この話を周囲にすると、「ここまでやれちゃうんですかねえ?」という反応も、当時わりとありました。

### 『割に合わねえな』

これを AI の力でやったと想像したとき、私は「(特にコミュニティ・ベースの) オープンソース・プロジェクトを、個人に近い努力でメンテし続けるのは、もう『割に合わねえな』」と感じました。

もちろん「レビューや確認に AI を活かす余地はある」という見方は当時からありました。実際に大きく効率化したことは、現在では多くの人が知るとおりです。

ただ、「AI がレビュー・確認やコミュニケーションを効率化する比率」と、「AI が (どうでもいい・攻撃) コードの生成を効率化する比率」では、後者のほうが圧倒的に大きいことも明らかです。それは現在でも変わっていません。

### 生成と確認の間

コミュニティ・ベースのオープンソース・プロジェクトに AI がもたらした作用として最も大きいのは、「AI による効率化」より、この「生成コストと確認コストの『比率』の変化」でしょう。

チームや組織に閉じた変化であれば、同じ組織内の話ですから、調整できなくはありません。 [^team-review]

[^team-review]: ただ、組織内においてすらその調整はそんなに簡単ではなかったことは、多くの指摘があります。

しかし、オープンに貢献を受け付けるプロジェクトではそうもいきません。確認に使えるのは限られたメンバーの限られたリソースである一方、お手軽に低コストで「貢献」を送りつけてくる「リソース」は外界から無限にわいてくるし、そうするインセンティブを持つ人も善意にせよ我欲にせよ悪意にせよ無限にいるのですから。

こうなってみると、いわゆる「オープンソース・コミュニティ」の多くは、「確認より生成に微妙に高いコストがかかる」というバランスのおかげで幸運にも成り立っていただけだ、ということに気がつきます。

「オープンソース活動についていろいろ考えてしまった」と書いたのは、こういうことでした。

### 疲弊

私は当時、ログリサーチ＝サンをきっかけにこのように感じ、「個人で持続するのはもう無理だ」「本当に続けるのなら、個人とコミュニティの力ではなく、組織のリソースを費やす必要がある」「組織にリソースを費やす気がないなら、破滅的に終わるよりは、軟着陸で終わらせるべきだ」と考えました。その行き着いた先が、メンテナンス・モードです。

@[card](https://zenn.dev/dmikurube/articles/how-to-wind-down-an-opensource-project)

そして現在、多くのオープンソース・プロジェクトが実際に疲弊していることは、多くの人が知るとおりです。

それは「PR を受け付けない public repository の存在を許さない」という思想の強さで長年知られていた GitHub が、ついに 2026 年に "Disable pull requests entirely" や "Restrict pull requests to collaborators" というオプションを提供したことからも、窺い知ることができます。

@[card](https://github.blog/changelog/2026-02-13-new-repository-settings-for-configuring-pull-request-access/)

2024-07-04 (木)
----------------

この日は、目に見えたログリサーチ＝サンの活動はありませんでした。

その一方で、ログリサーチ＝サンとは関係なさそうに見える以下のようなメールが、個人のメールアドレス宛に届きました。 (一部は匿名化しています)

> Hi Dai MIKURUBE,
>
> I have noticed you created/modified the GitHub Actions workflow of the `https://github.com/embulk/embulk` project on GitHub. As part of a research team from XXX University (<country>) and YYY University (<country>), we are interested in understanding why GitHub workflow failures occur in software projects, with a focus on the causes and solutions for these failures. We are highly interested in your response to get a diverse picture!
>
> Here's a link to the survey (Google form):`https://forms.gle/XXXXXXXXXXXXXXXXX`
>
> The survey should only take around 10 minutes. We would greatly appreciate your input -- it would help us understand the root causes of workflow failures and provide valuable insights for improving the development process. Your response will be handled confidentially! We will also openly publish the results and will anonymize everything before doing so.
>
> If you consider this email to be spam, I'm sorry! There will be no follow-up to bug you.
>
> Thanks a lot in advance!
>
> <name>

いー…やーー……。

このメールの送り主は名乗っているけど、ログリサーチ＝サンのトピックとは関係がない。

悪意の可能性を考慮に入れた上で考えると、関係ないと断ずるには昨日の今日のタイミングで怪しすぎる。

しかも a research team from University [^university] を名乗っておきながら、送り主のメールアドレスは大学のドメインではなく `@outlook.com` だし。

[^university]: この大学のうち片方は、上記で推察された言語圏のものです。もう片方は、まったく別の言語圏のものです。ただ、これはどちらにせよ自称でしかありませんし、両者が関連している可能性を想起させる一因程度のものでしかありません。言語圏自体の推察をもとにあーだこーだ言うことには、やはりなんの意味もないでしょう。

そのうえ "interested in understanding why GitHub workflow failures occur in software projects" って、大学の研究トピックとしてはちょっと特定企業の特定サービスに寄りすぎじゃない…? (これはちょっとイチャモンか)

うーん、役満では。

ログリサーチ＝サンとこのメールが関係あるにせよ関係無いにせよ、このメールもアウト判定かなあと、少なくともこの日は無視を決め込むことにしました。

そしてその後も、返信もフォームの記入もしていません。

2024-07-05 (金)
----------------

さてログリサーチ＝サン、実はこの日までは GitHub のユーザーページで activity が見えていたんですよ。ログリサーチ＝サンの PR を、私はこの日より前にはユーザーページから追っていたのです。

それがこの日、ログリサーチ＝サンは activity を private にしました。

@[card](https://github.com/logresearch)

![@logresearch's activity is private](/images/a-logresearch-er-1.png)

これをもって、私はログリサーチ＝サンを「悪意あり」と推認・断定することにしました。

activity を private にする理由なんて、「自分の他の PR を追跡されると不都合がある」以外に無いでしょ。しかもこういう活動を始めて、最初は見せていたのに、数日後から隠すなんて。

とはいえ、これはもちろん悪意を示す「証拠」には一切なりません。これを理由に追及、みたいなことはできません。善意だった可能性も、依然ゼロではありません。

ですがそもそもオープンソース・プロジェクトには、やってきた PR にすべて付き合う義理も無いし、公平性みたいなものを担保する義務だって無いのです。極論、たとえば「提案者個人が気に食わないから」で却下したって、本来はかまわないのです。

オープンソース・ライセンスの公開済みのコード一式は「公共物」と呼んでも差し支えないかもしれませんが、プロジェクトへの提案を「公共的」に受け入れる必要は一切ないのです。

ここまで検討したうえで出した結論です。もういいでしょ。

ということでこの日、私はこの PR を close して会話を lock する決定に至り、さらにログリサーチ＝サン `@logresearch` をプロジェクトとしてもブロックしました。

@[card](https://github.com/embulk/embulk/pull/1678)

その後
-------

ログリサーチ＝サンは、その後も一ヶ月ほど散発的に活動を続けたようですが、それから活動を止めたようです。

activity は private なのに、なぜそれがわかるかというと、検索には引っかかるからですね。

https://github.com/search?q=logresearch&type=pullrequests

興味がある人は、ログリサーチ＝サンの他の PR を眺めてみてもいいかもしれません。

振り返って、そしてこれから
===========================

オープンソース・プロジェクトの先にはメンテナー、つまり「人」がいる。

本記事で伝えたいメッセージを一言にまとめるなら、このことを、より多くの方に意識してほしい、ということです。もちろん、「悪意ある」攻撃は論外というのは前提ですが。

たとえばオープンソース・プロジェクトのメンテナーである「中の人」には、「あなた」と「悪意ある人」の区別はつきません。 [^acquaintance] この AI の時代ではなおさらです。

[^acquaintance]: もちろん、直接の知り合いなら、また別かもしれませんが。

もし PR を送るなら、「自分が送ろうとしている PR を『自分が暗黙に持っているコンテキスト抜きで』他人 (メンテナー) が見たら、どう見えるだろうか」と振り返ってみましょう。たとえば、以下のように自問してみるといいかもしれません。

* この PR は、自分の意図以外に、どう解釈しうるだろうか?
* 自分の意図は正しくそのまま伝わるだろうか?
* そもそも自分の意図はなんだっただろうか?

AI だって、ちゃんとそう指示すればそれなりに考えてくれますし、議論にも付き合ってくれますよ。

AI と「貢献」
--------------

AI 視点のまとめも無いと現代では片手落ちでしょう、ということで。

「オープンソース・プロジェクトにバグを修正してほしかったら、メンテナーにボランティアでやらせるより、費用を自分で AI に払って自分でやらせられるから、コスト持ち出しの関係が真っ当になる」みたいな発言を見たことがあります。

まったくの的外れだと、経験者としては感じました。

現行コードの背景と制約もよくわかってない第三者が雑に AI に生成させた結果の PR という「コード断片」のみから、その意図と背景を遡って推測するのは、メンテナーにとって苦痛です。メンテナーは、たとえばこの記事で書いてきたようなことまで考えたりするのですから。

そのうえ PR の送り主に意図を質問したのに「PR の送り主が AI に答えさせた回答」しか返ってこなかったりすると、究極的には「そこに『PR の送り主』がいる必要ないじゃん」っていう話にしかならないですよね。それならメンテナー自身が AI と直接話せばいいんだから。

どうせ AI にやらせるんだったら、現行コードの背景と制約をちゃんとわかってるメンテナーが最初から自分で AI にやらせるほうが、圧倒的に安くて早くて上手いんですよ。

あなたがもし本当に「貢献」を、ただし AI を使う前提で、したければ、不具合なら「問題を精確に再現する手順」を、機能要望なら「要望の背景となる明確なユースケース」を、しっかり書いてください。あとは費用と労力を気にするなら、ぶっちゃけ、メンテナーが AI を走らせるための実費だけ持っていただくのがベストじゃないでしょうか。 [^pay-for]

[^pay-for]: そういう金銭的な仕組み、欲しいですね。

トロフィー
-----------

ここで「いや、やっぱり自分の PR として履歴に自分の名前を残したい」と思ってしまう人は、一つ自問してみてください。

あなたが本当に欲しいのは「貢献した」というトロフィーではないですか?

オープンソース・プロジェクトへの「貢献」は、あなたのトロフィーではありません。

これは AI 以前からそうだったはずですが、今は「AI の力で簡単にトロフィーが手に入る」という二重の勘違いをしやすくなっている時代でもあります。

オープンソース・プロジェクトは、そのへんの地面に自然に生えている野草ではないし、あなたが好き勝手していい実験動物ではないし、あなたに経験を積ませてくれる無料のサンドバッグでもありません。

自分でやる、「人」に委ねる
---------------------------

「こんなことを言ってるメンテナーは古い! AI 生成コードの受け入れに消極的なオープンソース・プロジェクトが多いけど、もっと AI 生成コードも受け入れるべきだ!」と主張する人が一定数いるであろうことは、想像に難くありません。

なら、自分でメンテナーやればいいじゃん。 AI の力で簡単なんでしょう? オープンソースなんだから、今すぐにでも fork して始められますよ。さあ。

それを本当にやっていて、継続的にメンテナンスしている人のことは応援しています。それは正しくチャレンジだと思うので。 [^fork-and-continue]

[^fork-and-continue]: 特に、自分で一から始めたプロジェクト **ではなく** 他者のプロジェクトを fork したうえで継続する、というのは、本当にチャレンジだと思います。

でも、それをやらずに言っている人は、その「自分ではやっていない」っていうのが答えです。それでもなにかが大変そうだから、なにかが面倒くさいから、なにかが障壁になっているから、自分ではやらないんでしょう。その「なにか」をやっているのがメンテナーです。

そこを自分でやらずに、他者に、つまり「人」に委ねるのであれば、「人」を尊重しましょう。一人でやれることの範囲はたしかに拡大しましたが、他者と、人と、一切の無関係を通せるほどではありません。

それに、これからどれだけ AI が強くなろうが、私たち自身が「人」であることを辞められるわけではありません。そして私たちが「人」である以上、人と人の間で生きなければならないことにも、変わりはないのですから。

fork, fork, fork
-----------------

とはいえ実は、オープンソース・ソフトウェアについては、いずれ本当に「みんなが fork する」ようになっていくかも、とも、個人的には想像しています。

かつてソフトウェア・エンジニア業界 (の一部) では、「オープンソース・ソフトウェアへのローカルパッチを持つのは悪手」とされ、「元のプロジェクト (upstream) に正しく還元するのが善きソフトウェア・エンジニアの振る舞い」ともされてきました。

ですが、本記事を含めて多くの指摘があるように、オープンソース・プロジェクトのメンテナーの側は、来るもの来るものみんなを相手してられる状態ではなくなっています。そしてオープンソース・ソフトウェアを使う側でも、「いちいちメンテナーとやり取りするのなんてめんどくせえ」という人もいるでしょう。

さらに、かつて「ローカルパッチを持つのは悪手」とされた理由の一つだった「最新版への追従が大変になる」という課題は、今なら AI に「追従しといて〜」と指示するだけで、だいたい解決するでしょう。みんな自分の仕事のリポジトリに `third_party/` とかを掘って、そこに外部のオープンソース・ソフトウェアのコピー (fork) を持ち、そこでローカルの変更を好きなように加え、更新が必要なら AI に「追従しといて〜」と言うだけでいいんです。

メンテナーに PR のマージを催促する必要もありませんし、メンテナーが PR に煩わされることもなくなります。

あれ。 Win-Win では…?

昔に戻って (?)
---------------

さらに昔。 Git や GitHub よりも前。オープンソースという言葉が生まれる前や、生まれた直後くらいの時代。今でいう「オープンソース的な」ソフトウェアは、どこかの大学とかの FTP サーバーにソースコードの `.tar.gz` を置くことで公開されていて、それに対して誰かが書いた `.patch` ファイルが別の大学の FTP サーバーに置いてあったり、どこかのメーリングリストに流れたりしていました。

そのソフトウェアをパッチ込みで使いたい人は、当てたい `.patch` ファイルを手元に拾い集めてきて、集めたパッチを手元で当てて、手元でビルドして使う。そういう時代がありました。 [^a-certain-distribution]

[^a-certain-distribution]: 今も一部の Linux ディストリビューションはそれに近いという話はあります。

そんな時代に戻るだけなのかもしれません。

それがいいことなのかは、議論の余地があるでしょう。

ちょっとだけ差分があるけど 99% は同じコードを、世界のあちこちでばらばらに独立して AI にメンテさせるのは、さすがに電力の無駄遣いにも思えます。

セキュリティ対応とかどうすんのよ、みたいな話もありますね。

個人的にも、ちょっと寂しいような気はします。

ただ、メンテナーにあれこれ押し付ける人であふれる世界よりは、そのほうがマシかもしれません。みんな辞めちゃいますからね。 [^omaiu]

[^omaiu]: おまいう。

私たちは、このソフトウェア・エンジニアの世界が、どういう世界、どういう業界であってほしいでしょうか。それは、私たち自身が考えなければならないことなのだと思います。
