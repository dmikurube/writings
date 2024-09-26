---
title: "60年前からのプログラミング和文論文を掘り起こした話"
emoji: "📜" # アイキャッチとして使われる絵文字（1文字だけ）
type: "idea" # tech: 技術記事 / idea: アイデア
topics: [ "prosym", "ipsj", "論文", "history", "computerscience" ]
layout: default
published: true
publication_name: "prosym"
---

:::message
本記事の初版を公開したのは 2024 年 9 月 21 日だったのですが、その後 9 月 26 日に、ピックアップした論文を大幅に増やして更新しました。

また、最初はタイトルを「和文プログラミング論文を〜」としていたのですが、いわゆる「日本語プログラミング言語」の話にも見えてしまうことから、「プログラミング和文論文を〜」に置き換えました。
:::

日本の人々が「プログラミング」にかかわるようになったのは、いつごろでしょうか?

その前に、まずは世界のコンピュータとプログラミングの歴史を簡単に振り返ってみましょう。まずは OS から有名どころをたどると、

* わかりやすく Windows 95 が 1995 年 [^windows-95]
* その前に広く使われた Windows 3.1 は 1992 年 [^windows-31]
* その下で動く MS-DOS は、バージョン 6 が 1993 年、バージョン 1 が 1981 年 [^ms-dos]
* Linus Torvalds が Linux の開発を始めたのが 1991 年 [^linus-linux]
* その「契機となった」 Andrew Tanenbaum の MINIX が 1987 年 [^tanenbaum-minix]
* さらに遡ればオリジナルの BSD (Berkeley Software Distribution) が 1977 年 [^bsd]
* それらの礎であった Unix (と Space Travel (ゲーム)) の開発が 1969 年 [^unix] [^unix-and-space-travel]
* その前身であった Multics プロジェクトが始まったのが 1964 年 [^multics]

[^windows-95]: [Microsoft Windows 95 (Wikipedia)](https://ja.wikipedia.org/wiki/Microsoft_Windows_95)
[^windows-31]: [Microsoft Windows 3.x (Wikipedia)](https://ja.wikipedia.org/wiki/Microsoft_Windows_3.x)
[^ms-dos]: [MS-DOS (Wikipedia)](https://ja.wikipedia.org/wiki/MS-DOS)
[^linus-linux]: [Linuxの歴史 (Wikipedia)](https://ja.wikipedia.org/wiki/Linux%E3%81%AE%E6%AD%B4%E5%8F%B2)
[^tanenbaum-minix]: [MINIX (Wikipedia)](https://ja.wikipedia.org/wiki/MINIX)
[^bsd]: [Berkeley Software Distribution (Wikipedia)](https://ja.wikipedia.org/wiki/Berkeley_Software_Distribution)
[^unix]: [UNIX (Wikipedia)](https://ja.wikipedia.org/wiki/UNIX)
[^unix-and-space-travel]: Koichi Nakashima さんの『[Unixの歴史の起源を伝説のゲーム「スペース・トラベル」で遊んで学ぼう！](https://qiita.com/ko1nksm/items/1b1155956c121cbf713f)』 (2024 年 9 月 18 日) にも詳しい。
[^multics]: [Multics (Wikipedia)](https://ja.wikipedia.org/wiki/Multics)

といったところでしょうか。プログラミング言語の有名どころで言えば、

* Sun Microsystems が Java の α 版を出したのが 1995 年、正式版の 1.0 が 1996 年 [^java]
* Brendan Eich が JavaScript の原型となる LiveScript を開発したのが 1995 年 [^javascript]
* まつもとゆきひろさんが Ruby の開発を始めたのが 1993 年、発表したのが 1995 年 [^ruby]
* Guido van Rossum が Python 0.90 を公開したのが 1991 年 [^python]
* Perl の最初のバージョン 1.0 が 1987 年 [^perl]
* SQL という名前が付けられたのが 1970 年代 [^sql]
* 上記の Unix と対で語られる C 言語の開発が 1972 年 [^c-lang]
* 最初に動作したバージョンの Smalltalk も 1972 年 [^smalltalk]
* 最初の BASIC が開発されたのが 1964 年 [^basic]
* CODASYL が COBOL を開発したのが 1959 年 [^cobol]
* LISP の開発が始まったのが 1958 年、論文が発表されたのが 1960 年 [^lisp]
* John Backus が FORTRAN の最初の最初のバージョンを考案したのが 1954 年 [^fortran]

[^java]: [Java (Wikipedia)](https://ja.wikipedia.org/wiki/Java)
[^javascript]: [JavaScript (Wikipedia)](https://ja.wikipedia.org/wiki/JavaScript)
[^ruby]: [Ruby (Wikipedia)](https://ja.wikipedia.org/wiki/Ruby)
[^python]: [Python (Wikipedia)](https://ja.wikipedia.org/wiki/Python)
[^perl]: [Perl (Wikipedia)](https://ja.wikipedia.org/wiki/Perl)
[^sql]: [SQL (Wikipedia:en)](https://en.wikipedia.org/wiki/SQL)
[^c-lang]: [C言語 (Wikipedia)](https://ja.wikipedia.org/wiki/C%E8%A8%80%E8%AA%9E)
[^smalltalk]: [Smalltalk (Wikipedia)](https://ja.wikipedia.org/wiki/Smalltalk)
[^basic]: [BASIC (Wikipedia)](https://ja.wikipedia.org/wiki/BASIC)
[^cobol]: [COBOL (Wikipedia)](https://ja.wikipedia.org/wiki/COBOL)
[^lisp]: [LISP (Wikipedia)](https://ja.wikipedia.org/wiki/LISP)
[^fortran]: [Fortran (Wikipedia)](https://ja.wikipedia.org/wiki/Fortran)

みたいな時間軸を見ることができます。 (歴史の話なので近年のものは最初から除いています)

プログラミング黎明期とプロシン
===============================

この最初期にあたる 1950 年代にはすでにプログラミングに触れていた人が、日本にももちろんいました。ですが、目的である科学技術計算や事務計算のための単なる「作業」ととらえられがちで、「プログラミングという行為」そのものに特に注目することは多くなかったようです。

そんな中で 1960 年 1 月に第 1 回がおこなわれたのが、「プログラミング・シンポジウム」 (通称プロシン) です。 [^beginning-of-prosym] 日本で「プログラミングそのものを興味の対象としたイベント」は、これがおそらく最初のものだったと思われます。 [^first-programming-event]

[^beginning-of-prosym]: 当時の文部省科学研究費でおこなわれた「数理科学の総合研究」の一部として、[山内二郎](https://museum.ipsj.or.jp/pioneer/yamauc.html)先生が始めたもの、ということです。山内二郎先生追悼集刊行委員会編 「山内二郎先生 人と業績」 (1985 年 11 月) より。 (プログラミング・シンポジウムの Web サイトにある「[プログラミング・シンポジウムの始まり](https://prosym.org/articles/2022/09/29/beginning.html)」に抜粋が掲載されています)

[^first-programming-event]: 2024 年現在の日本でプログラミングをあつかう団体、特に学会としては、まず「[情報処理学会](https://www.ipsj.or.jp/index.html)」と「[ソフトウェア科学会](https://www.jssst.or.jp/)」が挙がるでしょう。そのうち古いほうである情報処理学会でも、その設立総会は 1960 年 4 月だったそうです。「[情報処理学会60年のあゆみ](https://www.ipsj.or.jp/60anv/60nenshi/1-1-1-1.html)」より。

プロシンは今でも続いていて、年明けの 2025 年には第 66 回を数えます。現在のプロシンについては、以下の記事などをご覧ください。 (第 66 回の論文・発表も絶賛募集中です!)

@[card](https://zenn.dev/prosym/articles/introduction-to-programming-symposium)

過去のプロシン論文集
---------------------

さてこのプロシン、いちおう学会系のイベントなので、第 1 回から毎回いわゆる「論文集 (報告集・予稿集)」が出版されています。しかし当時のシンポジウムに参加していなかった人には、これまで第 46 回 (2006 年) 以前の論文集にアクセスする方法がありませんでした。主にシンポジウムの参加者に配布された論文集 (物理冊子) しか、実質的に存在しなかったんですね。

プロシン以外の、情報処理学会として開かれていた会議の論文や会誌「情報処理」の記事などは、スキャンしたものが最初期のものから「[情報学広場](https://ipsj.ixsq.nii.ac.jp/ej/)」 (情報処理学会の電子図書館) に掲載されています。 [^jouhou-shori-vol-1] しかしプロシンの論文は、長らく第 47 回以降のものしか掲載がありませんでした。 [^prosym-digital-library]

[^jouhou-shori-vol-1]: 例: [会誌「情報処理」 Vol.1 (1960)](https://ipsj.ixsq.nii.ac.jp/ej/index.php?action=pages_view_main&active_action=repository_view_main_item_snippet&index_id=576&pn=1&count=20&order=7&lang=japanese&page_id=13&block_id=8)
[^prosym-digital-library]: 過去の論文を掲載できていなかったのは、プロシンと情報処理学会では設立の過程が異なっていて運営も独立していたため、著作権処理などが情報処理学会のものとそろっていなかった、などが主な理由です。

関係者には、「おもしろい議論がされている論文も多いのに読めないのはもったいない」という気持ち、そして「せっかく書いた論文へのアクセスがかぎられることは著者にとっても本懐ではないはずだ」という思いがありました。そこで 2020 年、千葉大学の辻尚史先生に音頭を取っていただき、過去のプログラミング・シンポジウム論文集の内容を情報処理学会の情報学広場に掲載する活動が始まりました。 [^past-reports-copyrights]

[^past-reports-copyrights]: 著作権の利用許諾などについて、権利者捜索のためのご連絡が「[過去のプログラミング・シンポジウム報告集の利用許諾について](https://www.ipsj.or.jp/topics/Past_reports.html)」に掲載されています。

そこから掲載までの実作業にさらなる年月を要してしまったのですが、論文集の冊子をお持ちだった先生方からスキャンを提供いただき、さらに多くの方々のご協力を得て論文タイトル・著者の確認や PDF の分割などをおこない、第 46 回以前のプロシンの論文集もすべて情報学広場に掲載するめどがようやく立ちました。 [^as-of-2024-09-26]

[^as-of-2024-09-26]: 2024 年 9 月 26 日時点では第 45 回と第 46 回がまだ抜けているのですが、おそらく 9 月中 〜 10 月初頭くらいには公開まで至るものと思います。 ([夏のプログラミング・シンポジウム](https://prosym.org/sprosym2024/)で共有したかったので、取り急ぎ本記事を公開しました)

本記事は、せっかく公開される論文集が多くの方の目に留まってほしいと、宣伝のために書いているものです。 [^aurhor-this-article]

[^aurhor-this-article]: 本記事の筆者は、データのチェックや PDF 分割の取りまとめと、最終確認の役をやっておりました。

学会系のイベントだからといって決してカタすぎることのない、ただただ「プログラミングが好きな」人たちが自由闊達にいろいろな (好き勝手な) 議論を楽しんでいる様子が目に浮かぶような、当時の息吹を感じられる記録になっていると思います。ぜひアクセスしてみてください!

* [情報学広場](https://ipsj.ixsq.nii.ac.jp/ej/)
  * ➤ [シンポジウム](https://ipsj.ixsq.nii.ac.jp/ej/index.php?action=pages_view_main&active_action=repository_view_main_item_snippet&index_id=6164&pn=1&count=20&order=7&lang=japanese&page_id=13&block_id=8)
    * ➤ [プログラミング・シンポジウム](https://ipsj.ixsq.nii.ac.jp/ej/index.php?action=pages_view_main&active_action=repository_view_main_item_snippet&index_id=6805&pn=1&count=20&order=7&lang=japanese&page_id=13&block_id=8)
      * ➤ [冬 (のプログラミング・シンポジウム)](https://ipsj.ixsq.nii.ac.jp/ej/index.php?action=pages_view_main&active_action=repository_view_main_item_snippet&index_id=6807&pn=1&count=20&order=7&lang=japanese&page_id=13&block_id=8)

ピックアップ
=============

…と、概要だけ紹介して終わりにしてもいいのですが、これだけで興味を持ってもらえる人は、あまり多くないかもしれません。

ということでここからは、本記事の筆者が「おもしろい!」と思った論文や、各トピックにそれぞれ触れているような論文を、あくまで筆者の独断と偏見でピックアップして紹介してみたいと思います。 [^survey-all]

[^survey-all]: なにせ掲載作業を進める中で、ほぼすべてに軽く目は通しましたので…。

各話題に対してちょっと目を引く論文をひたすら持ってきていたら大量になってしまいましたが、それでもおそらくまったく拾いきれていません。紹介に対して、筆者の趣味や理解が多分に反映されることはお許しください。

東京オリンピツク
-----------------

「東京オリンピツク」の話なんかが出てきたこともあったようです。つい数年前にあった東京 2020 じゃないですよ。第 6 回プログラミング・シンポジウム (1965) で発表された、その前年の 1964 年の東京オリンピツクの話題です。

* [東京オリンピツクに使われたリアルタイム・システムについて](https://ipsj.ixsq.nii.ac.jp/ej/?action=repository_uri&item_id=223027), 竹下 享 (日本IBM), 第 6 回プログラミング・シンポジウム報告集 (1965)

> 1964年10月に東京において開催されたオリンピツクは，科学の大会と言われ各種の機
> 械が競技成績の判定・表示・伝達・印刷に使われた。8台の電子計算組織を中枢とするIBM
> オリンピツク・テレプロセシング・システムはその中で最も画期的かつ大規模なものであり，
> 広範な地域に散在する32の競技場から多種・多様かつ大量の競技データーを時々刻々高速で
> 収集・処理・伝達し，インフオーメーシヨン・サービスに華々しい活躍を示した．

表現がいちいち仰々しいことには時代を感じさせますが、当時のシステム構成図から、メッセージデータのフォーマッティング、実際に処理されたデータ、事故対応の予備系統の説明など、わりと生々しい内容が記録されています。

『自動診断』
-------------

プログラミングの文脈で「自動診断」というと、プログラムの静的検証とかデータ検証とかの話かなと思いきや、これが医療における自動「診断」の話だったりします。これも 1965 ですね。

(その前 1964 年の第 5 回までは数値計算や (オペレーティング) システム、コンパイラ・プログラミング言語などの話題がほとんどだったんですが、この第 6 回は多分野の話が急に多くなっています。なにか運営上の工夫があったのかもしれません)

* [自動診断](https://ipsj.ixsq.nii.ac.jp/ej/?action=repository_uri&item_id=223045), 高橋 晄正 (東大医), 第6回プログラミング—シンポジウム報告集 (1965)

プログラムの話というよりは「臨床から取得した統計量をどう処理するか」という議論が数式とともに並ぶ感じですが、ソフトウェア屋からするとなかなか新鮮な話です。

その後も医療関係の話はしばしば出てきているようです。

* [九大病院の医療情報システム](https://ipsj.ixsq.nii.ac.jp/ej/?action=repository_uri&item_id=238066), 野瀬 善明, 柴原 哲太郎, 中垣 修, 多治見 司, 井上 十四雄, 中村 元臣 (九州大学医学部), 第17回プログラミング・シンポジウム報告集 (1976)
* [コンピュータ断層撮影装置の現状と将来](https://ipsj.ixsq.nii.ac.jp/ej/?action=repository_uri&item_id=232181), 飯沼 武 (放射線医学総合研究所), 第20回プログラミング・シンポジウム報告集 (1979)

産業分野・異分野
-----------------

医療分野以外にも、各種産業分野からの発表をちょくちょく見かけるのもおもしろいところです。たとえば鉄鋼プラントの話が出てきたかと思えば…。

* [鉄鋼プラントにおけるプロセス用計算機の利用](https://ipsj.ixsq.nii.ac.jp/ej/?action=repository_uri&item_id=223032), 坪井 邦夫 (日本鋼管(株)), 第6回プログラミング—シンポジウム報告集 (1965)

味の素の方がグルタミン酸結晶の硝析を効率的におこなうためのシミュレータの話をしていたり。

* [晶析工程シミュレータ](https://ipsj.ixsq.nii.ac.jp/ej/?action=repository_uri&item_id=237787), 富田 潔, 小幡 孝一郎, 河合 健司, 森 幸則, 西村 俊昭 (味の素株式会社), 第7回プログラミング—シンポジウム報告集 (1966)

OCR (文字認識) の話かー、ふーん、と思えば、発表は三越の方で、現実問題を解こうとしている様子が見えたり。

* [OCRによる手書文書読取の実用化](https://ipsj.ixsq.nii.ac.jp/ej/?action=repository_uri&item_id=237965), 石黒 栄一 (三越), 第11回プログラミング—シンポジウム報告集 (1970)

鹿島建設の方が、当時 (1972) リアルタイムに計算をしながらディスプレイに表示することが難しかった地震応答のシミュレーションの結果を、フィルムに出力する工夫の話をしていたり。

* [COMによるアニメーション ―地震応答シミュレーション結果の表示―](https://ipsj.ixsq.nii.ac.jp/ej/?action=repository_uri&item_id=232148), 高瀬 啓元, 堀越 清視, 平野 隆久 (鹿島建設), 第13回プログラミング—シンポジウム報告集 (1972)

うーん、バッチ処理かー、と思うと三井銀行の方 (1977) だったり。

* [大量バッチ処理の新体系](https://ipsj.ixsq.nii.ac.jp/ej/?action=repository_uri&item_id=238092), 和泉 宏之 (三井銀行), 第18回プログラミング・シンポジウム報告集 (1977)

トヨタの方が自動車ボディの開発に使っていた (1981) CAD・CAM の話をしていたり。

* [トヨタ自工におけるボデー開発工程CAD・CAM](https://ipsj.ixsq.nii.ac.jp/ej/?action=repository_uri&item_id=238286), 田辺 和夫 (トヨタ自動車工業(株)), 第22回プログラミング・シンポジウム報告集 (1981)

また産業分野にかぎらず、異分野の研究領域との協業、のような話が紹介されたりもします。

* [天文観測整約用言語「アカイホシ」](https://ipsj.ixsq.nii.ac.jp/ej/?action=repository_uri&item_id=238104), 近田 義広 (東京大学), 第18回プログラミング・シンポジウム報告集 (1977)
* [地震予知観測情報のオンライン・データベースシステムについて](https://ipsj.ixsq.nii.ac.jp/ej/?action=repository_uri&item_id=232230), 桧山 澄子, 安永 尚志 (東京大学地震研究所), 第21回プログラミング・シンポジウム報告集 (1980)
* [計算機屋と歴史学者の協調作業 ～計算機の専門家が非専門家のプロジェクトに参加して～](https://ipsj.ixsq.nii.ac.jp/ej/?action=repository_uri&item_id=239617), 並木 美太郎 (東京農工大学), 第39回プログラミング・シンポジウム報告集 (1998)

情報科学の学術系にいても、現代のいわゆるソフトウェア・エンジニア業界にいても、あまり触れる機会がなさそうな話がされていたりします。

鉄道
-----

界隈に鉄の人が多いのは昔からだったようで、定期的に鉄道に関連する話題が出てきます。

* [国鉄座席予約システムMARS-101](https://ipsj.ixsq.nii.ac.jp/ej/?action=repository_uri&item_id=223029), 谷 恭彦 (日立神奈川工場), 第6回プログラミング—シンポジウム報告集 (1965)
* [鉄道の最適な保線作業時間帯を決定する算法とデータ構造](https://ipsj.ixsq.nii.ac.jp/ej/?action=repository_uri&item_id=238305), 中森 真理雄 (東京農工大学), 第22回プログラミング・シンポジウム報告集 (1981)
* [招待講演「これからの列車運転制御システム」](https://ipsj.ixsq.nii.ac.jp/ej/?action=repository_uri&item_id=238927), 佐々木 敏明 (鉄道総合研究所), 第34回プログラミング・シンポジウム報告集 (1993)
* [列車運行情報の可視化](https://ipsj.ixsq.nii.ac.jp/ej/?action=repository_uri&item_id=239699), 斎藤 隆文 (東京農工大学), 飯野 昭 (住鉱情報システム(株)), 第41回プログラミング・シンポジウム報告集 (2000)
* [最長片道きっぷ ―理論と実践―](https://ipsj.ixsq.nii.ac.jp/ej/?action=repository_uri&item_id=239768), 葛西 隆也, 宮代 隆平 (東京大学), 第42回プログラミング・シンポジウム報告集 (2001)

データ処理とインフォメーション・リトリーバル
---------------------------------------------

まだ SQL も関係データベース (RDB) も存在していない 1963 年の第 4 回の発表ですが、当時 CODASYL [^codasyl] という団体で議論されていた COBOL と情報代数ベースのデータ処理のデータ処理関連の (英語) 論文が紹介されています。だいぶ教科書的な話かもしれませんが、いまの RDB/SQL と比べてみるとおもしろい人もいるかもしれません。

[^codasyl]: [CODASYL (Wikipedia)](https://ja.wikipedia.org/wiki/CODASYL)

* [データ処理理論の展望](https://ipsj.ixsq.nii.ac.jp/ej/?action=repository_uri&item_id=237698), 関根 智明 (慶応大学), 刀根 薫 (津田熟大学), 第4回プログラミング—シンポジウム報告集 (1963)

そこからさらに、文書の管理とそこからの検索的な話題としての「インフォメーション・リトリーバル」の議論が 1965 年の第 6 回でもなされています。インフォメーション・リトリーバルは現代でも使われる用語ですね。

* [インフオメーション・リトリーバルの展望](https://ipsj.ixsq.nii.ac.jp/ej/?action=repository_uri&item_id=223028), 関根 智明, 昆野 誠司 (慶応大学), 第6回プログラミング—シンポジウム報告集 (1965)

「外務省における情報検索システム」とかいきなり出てきて「ファッ!?」となったりもします。

* [外務省における情報検索システム](https://ipsj.ixsq.nii.ac.jp/ej/?action=repository_uri&item_id=237783), 鈴木 幸雄 (外務省電子計算機室), 第7回プログラミング—シンポジウム報告集 (1966)

最近 (2010 年〜) のプロシンではデータベース的な話はそんなに多くないなあと思っていたのですが、歴史を振り返るとけっこうあったんですね。関係する話題は、継続的にいろいろな形で出てきます。

* [情報検索システムのリアルタイムシミュレーション](https://ipsj.ixsq.nii.ac.jp/ej/?action=repository_uri&item_id=237987), 沢 公人, 平原 英夫, 沼上 英雄 (東芝総合研究所), 第11回プログラミング—シンポジウム報告集 (1970)
* [関係データベース上の高度な質問応答過程について](https://ipsj.ixsq.nii.ac.jp/ej/?action=repository_uri&item_id=238144), 古川 康一 (電子技術総合研究所), 第19回プログラミング・シンポジウム報告集 (1978)
* [日本語によるデータベース検索システム](https://ipsj.ixsq.nii.ac.jp/ej/?action=repository_uri&item_id=238393), 泉田 義男 (富士通研究所), 第25回プログラミング・シンポジウム報告集 (1984)

2000 年代にはオープンソースの XML-DB なんかの話がされていたり PostgreSQL の話がされていたりもします。

* [RDBとODBを融合するXML-DBフレームワーク ～オープンソースがもたらすXML-DBの未来～](https://ipsj.ixsq.nii.ac.jp/ej/?action=repository_uri&item_id=239817), 小松 誠 (有限会社メディアフロント 日本PostgreSQLユーザー会), 第43回プログラミング・シンポジウム報告集 (2002)
* [PostgreSQLを用いた分散データベースの開発](https://ipsj.ixsq.nii.ac.jp/ej/?action=repository_uri&item_id=239878), 永安 悟史 (慶應義塾大学大学院 政策メディア研究科), 第44回プログラミング・シンポジウム報告集 (2003)

円周率
-------

円周率で有名な金田康正先生 [^kanada] らによる円周率関係の発表も複数回あったようです。

[^kanada]: [金田康正 (Wikipedia)](https://ja.wikipedia.org/wiki/%E9%87%91%E7%94%B0%E5%BA%B7%E6%AD%A3)

* [円周率 ―高速計算法と数値の統計性―](https://ipsj.ixsq.nii.ac.jp/ej/?action=repository_uri&item_id=238374), 田村 良明 (緯度観測所), 吉野 さやか (筑波大学), 後 保範 (日立製作所), 金田 康正 (東京大学), 第25回プログラミング・シンポジウム報告集 (1984)
* [円周率 ―高速計算法と統計性― (2)](https://ipsj.ixsq.nii.ac.jp/ej/?action=repository_uri&item_id=238490), 金田 康正 (東京大学), 田村 良明 (緯度観測所), 第28回プログラミング・シンポジウム報告集 (1987)
* [円周率 ―高速計算法と統計性― (3)](https://ipsj.ixsq.nii.ac.jp/ej/?action=repository_uri&item_id=239120), 高橋 大介, 金田 康正 (東京大学), 第37回プログラミング・シンポジウム報告書 (1996)

数値計算・行列計算など
-----------------------

特に第 1 回 (1960) から第 3 回 (1962) あたりのリストを眺めると、「計算」の話題がよく出てきます。特に当時の大学や研究機関では、コンピュータの主な使用目的は物理のシミュレーション計算であったり行列計算であったり、という雰囲気が垣間見える感じでしょうか。

* [関数の関数をn回微分するためのサブルーチンとエルミット多項式Hk(x)のサブルーチン](https://ipsj.ixsq.nii.ac.jp/ej/?action=repository_uri&item_id=232018), 戸田 英雄 (電試), 山内 二郎 (慶大工), 第1回プログラミング–シンポジウム報告集 (1960)
* [エラトステネスのふるいによる素数の計算について](https://ipsj.ixsq.nii.ac.jp/ej/?action=repository_uri&item_id=232019), 新井 克彦 (通研), 第1回プログラミング–シンポジウム報告集 (1960)
* [神経のモデルの数値計算](https://ipsj.ixsq.nii.ac.jp/ej/?action=repository_uri&item_id=237687), 高田 勝, 清水 留三郎 (東大工), 第3回プログラミング—シンポジウム報告集 (1962)

その後も (数値) 計算が関係するような話題は、ずっとなにかしら出続けていますね。

* [幻影数値解](https://ipsj.ixsq.nii.ac.jp/ej/?action=repository_uri&item_id=237826), 宇野 利雄, 永坂 秀子 (日大理工), 第8回プログラミング—シンポジウム報告集 (1967)

『UNIXについて』
-----------------

UNIX を含む歴史についても上で軽く触れましたが、その UNIX そのものについて「他のシステムに余りみられない UNIX の特徴をあげてみると」のように紹介している論文もありました。 UNIX の開発・登場 (1969) から数年経った (1978) ところでの紹介ですが、いまとなっては (Linux として) 当たり前になってしまった UNIX 的なものについての、当時の空気感が少しわかるかもしれません。

* [使いやすさを狙ったTSS用OS―UNIXについて](https://ipsj.ixsq.nii.ac.jp/ej/?action=repository_uri&item_id=238132), 石田 晴久, 和田 英一, 安村 通晃 (東京大学), 第19回プログラミング・シンポジウム報告集 (1978)

C 言語についても UNIX の文脈の中で紹介されています。

また UNIX 関係にかぎらず、オペレーティング・システム (OS) 関係の話題というのも継続的にされています。

* [慶応義塾大学 実験用タイムシェアリング・システム](https://ipsj.ixsq.nii.ac.jp/ej/?action=repository_uri&item_id=237844), 土居 範久 (慶応義塾大学), 第8回プログラミング—シンポジウム報告集 (1967)
* [セグメンテーション，ページング機構を有するタイムシェアリング・システムの評価](https://ipsj.ixsq.nii.ac.jp/ej/?action=repository_uri&item_id=237976), 益田 隆司, 本林 繁, 久保 隆重, 吉沢 康文, 吉田 郁三, 高橋 延匡 (日立製作所中央研究所), 第11回プログラミング—シンポジウム報告集 (1970)
* [OSの理論](https://ipsj.ixsq.nii.ac.jp/ej/?action=repository_uri&item_id=223430), 斉藤 信男 (電子技術総合研究所), 第15回プログラミング—シンポジウム報告集 (1974)
* [プロセス・ネットワークによるOSについて](https://ipsj.ixsq.nii.ac.jp/ej/?action=repository_uri&item_id=238417), 田胡 和哉, 益田 隆司 (筑波大学), 第26回プログラミング・シンポジウム報告集 (1985)
* [ワークステーション用オペレーティング・システムの移植について](https://ipsj.ixsq.nii.ac.jp/ej/?action=repository_uri&item_id=238418), 多田 好克 (東京大学), 第26回プログラミング・シンポジウム報告集 (1985)
* [日本語情報処理向けの並列処理用OS "OS/omicron"](https://ipsj.ixsq.nii.ac.jp/ej/?action=repository_uri&item_id=237522), 並木 美太郎, 岡野 裕之, 堀 素史, 横関 隆, 早川 栄一, 中川 正樹, 高橋 延匡 (東京農工大学), 第31回プログラミング・シンポジウム報告集 (1990)

手書き論文
-----------

ところで、第 1 回から普通に活字 (おそらくタイプライター) が使われていたにもかかわらず、先の『UNIX について』の第 19 回 (1978) を含む第 17 回以降しばらく、手書きの論文がよく見られます。

どうもこれは、暮れに印刷関係がラッシュになって、不満足なものが出がちだったことに対して「あえて」やったことだったようです。当時の事情をすべて推しはかることはできませんが、「手作り」感にはあふれていて、これはこれでおもしろい気がしますね。

* [第17回プログラミング・シンポジウムの準備にあたって](https://ipsj.ixsq.nii.ac.jp/ej/?action=repository_uri&item_id=238060)

> さて，第17回のシンポジウムは前回にこりて，発表にはくわしい要旨をつけていただき，それ
> によって選択することもあるといって募集した。実際いろいろな理由により，せっかくだったが，
> いくつかの発表申し込みにはご遠慮願った。また今回からは前2回の夏のシンポジウムの経験にも
> とづき，報告集を手書きオフセット印刷に切りかえた。若干読みにくいかも知れないが，暮の印刷
> ラッシュで，やゝ不満足な図や，多少の誤植の避け得ない，無味乾燥なタイプ版のものより，個々
> の発表者の苦心の「手作り」の予稿を味わって頂きたい。この辺の情報処理技術は，必要なのにも
> 拘らず，大変遅れていて，人海戦術然としているのはまことに遺憾である。

組版・文書作成
---------------

印刷や版の話が出てきたので、ここで組版の話も取り上げておきましょう。ちなみに TeX [^tex] の「初版」は 1978 年だったそうですが、それ以前からも組版の話題は出てきています。

[^tex]: [TeX (Wikipedia)](https://ja.wikipedia.org/wiki/TeX)

* [汎用自動組み版システムの一例](https://ipsj.ixsq.nii.ac.jp/ej/?action=repository_uri&item_id=223397), 昆野 誠司, 関谷 敏郎 (管理工学研究所), 望月 昭 (大倉商事), 第14回プログラミング—シンポジウム報告集 (1973)
* [HRO―自動的にハイフネーションする英文清書システム](https://ipsj.ixsq.nii.ac.jp/ej/?action=repository_uri&item_id=238282), 中島 玲二 (京都大学 数理解析研究所), 第22回プログラミング・シンポジウム報告集 (1981)
* [上つき、下つき、ギリシャ文字の操作がワンタッチで出来る科学技術用英文清書システムの作成](https://ipsj.ixsq.nii.ac.jp/ej/?action=repository_uri&item_id=238379), 桂 重俊, 増子 進 (東北大学), 第25回プログラミング・シンポジウム報告集 (1984)

現代で言うなら Markdown のようなモチベーションの文書作成というのも考えられていたようです。 (もちろんレンダリング先は HTML ではありませんが)

* [べた書き入力による文書清書の試み](https://ipsj.ixsq.nii.ac.jp/ej/?action=repository_uri&item_id=237500), 内田 昭宏, 佐野 晋 (日本電気), 第32回プログラミング・シンポジウム報告集 (1991)
* [テキスト処理のみによる簡易卓上印刷](https://ipsj.ixsq.nii.ac.jp/ej/?action=repository_uri&item_id=238925), 浜田 穂積 (電気通信大学), 第34回プログラミング・シンポジウム報告集 (1993)

ちなみに組版については、最近でも SATySFi の話がプロシンでもあったりしますね。

* [静的型つき組版処理システムSATYSFI](https://ipsj.ixsq.nii.ac.jp/ej/?action=repository_uri&item_id=232264), 諏訪 敬之, 第61回プログラミング・シンポジウム予稿集 (2020)

日本語とカナ漢字
-----------------

日本語は、現代では当たり前にコンピュータであつかわれていますが、実はかつてはそんなに当たり前のものではありませんでした。いわゆる「自然言語処理」以前から、文字コード、かな漢字変換、フォント、エディタのサポート、外字、などなど、いろいろなところに課題があり、いろいろな挑戦がされていたことがうかがえます。

* [仮名文字パタン認識の一考察](https://ipsj.ixsq.nii.ac.jp/ej/?action=repository_uri&item_id=237778), 富田 真吾, 大泉 充郎 (東北大学), 第7回プログラミング—シンポジウム報告集 (1966)
* [カナ漢字変換の事例研究](https://ipsj.ixsq.nii.ac.jp/ej/?action=repository_uri&item_id=238064), 野村 仙一 ((財)日本情報処理開発センター), 第17回プログラミング・シンポジウム報告集 (1976)
* [ゼロックス グラフィックス プリンタに日本語を教えた話](https://ipsj.ixsq.nii.ac.jp/ej/?action=repository_uri&item_id=232193), 木村 泉 (東京工業大学), 第20回プログラミング・シンポジウム報告集 (1979)
* [大規模漢字データの検証(姓, 名ファイルを用いて)](https://ipsj.ixsq.nii.ac.jp/ej/?action=repository_uri&item_id=232217), 田中 康仁 (日本ユニバック(株)), 第21回プログラミング・シンポジウム報告集 (1980)
* [日本語文書処理用入力設計と国語辞典の一分析](https://ipsj.ixsq.nii.ac.jp/ej/?action=repository_uri&item_id=238283), 木村 泉 (東京工業大学・カーネギーメロン大学), 第22回プログラミング・シンポジウム報告集 (1981)
* [マイクロ・コンピュータによる漢字処理システム](https://ipsj.ixsq.nii.ac.jp/ej/?action=repository_uri&item_id=238284), 小川 志俊, 坂東 浩之, 野田 晴義, 三島 良武, 大駒 誠一, 浦 昭二 (慶応義塾大学), 第22回プログラミング・シンポジウム報告集 (1981)
* [Unix Emacsに漢字を教えた話](https://ipsj.ixsq.nii.ac.jp/ej/?action=repository_uri&item_id=238414), 植田 健治 ((株)リコー), 第26回プログラミング・シンポジウム報告集 (1985)
* [多層テキスト構造を持つ日本語エディタ](https://ipsj.ixsq.nii.ac.jp/ej/?action=repository_uri&item_id=238442), 角田 博保 (電気通信大学), 第27回プログラミング・シンポジウム報告集 (1986)
* [漢字スケルトンフォントの生成支援システム](https://ipsj.ixsq.nii.ac.jp/ej/?action=repository_uri&item_id=237489), 田中 哲朗, 石井 裕一郎, 長橋 賢児, 竹内 幹雄, 岩崎 英哉, 和田 英一 (東京大学), 第32回プログラミング・シンポジウム報告集 (1991)
* [振り仮名を振ること (フリガナオフルコト)](https://ipsj.ixsq.nii.ac.jp/ej/?action=repository_uri&item_id=239758), 橋爪 宏達 (国立情報学研究所), 杉本 雅則 (東京大学), 第42回プログラミング・シンポジウム報告集 (2001)
* [Webにおける外字の取り扱い方式の提案](https://ipsj.ixsq.nii.ac.jp/ej/?action=repository_uri&item_id=239779), 来住 伸子 (津田塾大学), 第42回プログラミング・シンポジウム報告集 (2001)

さらに、印鑑の話なんかが出てきたりもしています。

* [印鑑とコンピュータ処理](https://ipsj.ixsq.nii.ac.jp/ej/?action=repository_uri&item_id=238136), 田中 康仁, 相賀 良久 (日本ユニバック株式会社), 第19回プログラミング・シンポジウム報告集 (1978)

ふりがなの話題などについては、最近でも話題があったりしました。

* [ふりがな付きHTML文書に対する文字列探索](https://ipsj.ixsq.nii.ac.jp/ej/?action=repository_uri&item_id=232296), 齋藤 匡 (電気通信大学), 岩崎 英哉 (明治大学), 第64回プログラミング・シンポジウム予稿集 (2023)

国際化
-------

また、英語でも日本語でもない国際化について議論されたこともあったようです。

* [アラビア語ワード・プロセッサの諸問題](https://ipsj.ixsq.nii.ac.jp/ej/?action=repository_uri&item_id=238285), 矢島 敬二, 平島 守 (日本科学技術研修所), 第22回プログラミング・シンポジウム報告集 (1981)
* [コンピュータの『国際化』に関する一考察](https://ipsj.ixsq.nii.ac.jp/ej/?action=repository_uri&item_id=238466), Beshr B. Bakhit, 遠山 元道, 浦 昭二 (慶応義塾大学), 第28回プログラミング・シンポジウム報告集 (1987)

自然言語処理
-------------

もちろん、今でいう「自然言語処理」の枠組みに入ってくるような話題も、かなり以前からありました。

* [計算機による翻訳](https://ipsj.ixsq.nii.ac.jp/ej/?action=repository_uri&item_id=223042), 大槻 説乎, 金田 すみ (名大総合計算室), 第6回プログラミング—シンポジウム報告集 (1965)
* [日本文の自動分かち書きについて(仮名文字列の分かち書き)](https://ipsj.ixsq.nii.ac.jp/ej/?action=repository_uri&item_id=232191), 田中 康仁 (日本ユニバック(株)), 古賀 勝久 ((株)リード), 第20回プログラミング・シンポジウム報告集 (1979)
* [確率モデルに基づく言語処理へのアプローチ](https://ipsj.ixsq.nii.ac.jp/ej/?action=repository_uri&item_id=238445), 藤崎 哲之助 (日本アイ・ビー・エム サイエンスインスティチュート), 第27回プログラミング・シンポジウム報告集 (1986)
* [形態素解析を利用した日本語スペルチェッカ](https://ipsj.ixsq.nii.ac.jp/ej/?action=repository_uri&item_id=237529), 下村 秀樹, 高橋 延匡 (東京農工大学), 第31回プログラミング・シンポジウム報告集 (1990)

各種プログラミング言語
-----------------------

「プログラミング」シンポジウムで「言語」っていったらプログラミング言語の話題じゃないんかと、ここまで読んだ方は思ったかもしれません。もちろん、プログラミング言語の話題もたくさんあります。 (実際のところ、たくさんありすぎるので後回しにしていました)

### ALGOL

特に初期には ALGOL の話題が多いですね。 (ちなみにこのころはまだ C 言語も存在しなかったりします)

* [ALGOLの文法](https://ipsj.ixsq.nii.ac.jp/ej/?action=repository_uri&item_id=232033), 森口 繁一 (東大工), 第1回プログラミング–シンポジウム報告集 (1960)
* [ALGOL 60のCompiler](https://ipsj.ixsq.nii.ac.jp/ej/?action=repository_uri&item_id=237669), 中島 勝也, 藤野 喜一, 小島 惇 (早大理工), 第3回プログラミング—シンポジウム報告集 (1962)
* [ALGŌL言語によるALGŌLコムパイラの製作](https://ipsj.ixsq.nii.ac.jp/ej/?action=repository_uri&item_id=223036), 井上 謙蔵, 高橋 秀知, 清水 公子 (東大物性研究所), 第6回プログラミング—シンポジウム報告集 (1965)
* [ALGOLコンパイラにおける最適化について](https://ipsj.ixsq.nii.ac.jp/ej/?action=repository_uri&item_id=232104), 島内 剛一 (立大理), 広瀬 健 (早大理工), 佐久間 紘一 (東大数理研), 福田 康夫 (東芝), 志村 昭雄 (日科技研), 第12回プログラミング—シンポジウム報告集 (1971)

### PL/I

PL/I なんかも (新しいプログラミング言語として) 出てきたりします。

* [新しいプログラミング言語: PL/I (その出現までの経過，すぐれた特色，既存の言語との比較)](https://ipsj.ixsq.nii.ac.jp/ej/?action=repository_uri&item_id=237782), 竹下 享 (日本アイ・ビー・エム株式会社), 第7回プログラミング—シンポジウム報告集 (1966)
* [PL/IWによるシステムの開発](https://ipsj.ixsq.nii.ac.jp/ej/?action=repository_uri&item_id=237984), 浜田 穂積, 中田 育男, 霜田 忠孝, 小林 正和 (日立製作所中央研究所), 第11回プログラミング—シンポジウム報告集 (1970)
* [PL/1WによるTSS用コンパイラコンパイラ作成](https://ipsj.ixsq.nii.ac.jp/ej/?action=repository_uri&item_id=232098), 二村 良彦, 西野 秀毅, 吉村 一馬 (日立中央研究所), 第12回プログラミング—シンポジウム報告集 (1971)
* [コンパイラ記述言語としての標準PL/I](https://ipsj.ixsq.nii.ac.jp/ej/?action=repository_uri&item_id=238035), 村上 正裄, 桑原 道明, 若山 正道, 田村 由紀子 (東京芝浦電気(株)), 第16回プログラミング—シンポジウム報告集 (1975)

### FORTRAN

FORTRAN はいまも数値計算系では現役ですが、もちろんその話題も出てきます。

性能を求めるケースが多いからか、特定のメーカー、特定のコンピュータ、の話題になるケースもありますね。

* [ベース・レジスタをもつ計算機TŌSBAC-3400のFŌRTRANコンパイラについて](https://ipsj.ixsq.nii.ac.jp/ej/?action=repository_uri&item_id=237839), 渡辺 一郎, 木下 恂 (東京芝浦電気株式会社), 第8回プログラミング—シンポジウム報告集 (1967)

プログラムの間違いや移植に関する話なども見かけやすいような気がします。

* [FORTRANプログラムにおける誤まりの傾向について](https://ipsj.ixsq.nii.ac.jp/ej/?action=repository_uri&item_id=232050), 五十嵐 滋 (東京大学), 第9回プログラミング—シンポジウム報告集 (1968)
* [数式解析のプログラムをFORTRANで書いた話](https://ipsj.ixsq.nii.ac.jp/ej/?action=repository_uri&item_id=232060), 戸川 隼人 (航技研), 照井 勝利 (HSE), 西村 剛 (成蹊大), 戸川 保子 (航技研), 第9回プログラミング—シンポジウム報告集 (1968)
* [FORTRANプログラムの動的解析システムの移し換えについて](https://ipsj.ixsq.nii.ac.jp/ej/?action=repository_uri&item_id=238072), 藤村 直美, 牛島 和夫 (九州大学), 第17回プログラミング・シンポジウム報告集 (1976)
* [ソフトウェアの移換性から見たFORTRANとCOBOLの比較](https://ipsj.ixsq.nii.ac.jp/ej/?action=repository_uri&item_id=232231), 藤村 直美 (九州大学), 第21回プログラミング・シンポジウム報告集 (1980)

### COBOL

上で FORTRAN と COBOL の比較の話もありましたが、単体での COBOL の話題ももちろんあります。先に挙げたような、データベースと絡めた話題もありますね。

* [COBOL 61 Extendedの紹介](https://ipsj.ixsq.nii.ac.jp/ej/?action=repository_uri&item_id=237755), 大駒 誠一 (慶大工), 第5回プログラミング—シンポジウム報告集 (1964)
* [COBOLコンパイラ記述言語](https://ipsj.ixsq.nii.ac.jp/ej/?action=repository_uri&item_id=232115), 伊藤 公一, 増田 竜彦 (東芝), 第12回プログラミング—シンポジウム報告集 (1971)
* [データ記述言語とCOBOLのデータベース機能](https://ipsj.ixsq.nii.ac.jp/ej/?action=repository_uri&item_id=223380), 植村 俊亮 (電子技術総合研究所), 第14回プログラミング—シンポジウム報告集 (1973)

### 「リスト処理」と LISP

実はプロシンは LISP 好きの方が多く、必然的に LISP 関係の話題は多くなりがちです。しかし特に初期には LISP という言語 (LISt Processor) とは別に、「リスト処理」「リスト・プロセッシング」と題した話題をよく見かけます。中を見ると LISP でも見るような木構造表現の図が出てきたりします。

LISP という言語とは独立に、いわゆる記号処理の文脈で「リスト (木) 処理」というのが一つの潮流だったのかな、という印象を受けます。実際、今でいうと「コンパイラ」や「構文解析」の話題のように見える文脈で、「リスト処理」という言葉が頻出します。

* [木表現とリスト処理の算法](https://ipsj.ixsq.nii.ac.jp/ej/?action=repository_uri&item_id=237792), 西村 恕彦 (電気試験所), 第7回プログラミング—シンポジウム報告集 (1966)
* [シンタックス・チェッカーとストレッジ・アロケータにおけるリスト・プロセッシング](https://ipsj.ixsq.nii.ac.jp/ej/?action=repository_uri&item_id=237793), 浅井 清, 根田 美佐子 (日本原子力研究所), 第7回プログラミング—シンポジウム報告集 (1966)
* [リスト・プロセッサーELIS](https://ipsj.ixsq.nii.ac.jp/ej/?action=repository_uri&item_id=237794), 池野 信一 (通研), 第7回プログラミング—シンポジウム報告集 (1966)
* [行列の基本演算の数式処理とリスト処理言語](https://ipsj.ixsq.nii.ac.jp/ej/?action=repository_uri&item_id=232062), 石黒 美佐子, 中村 康弘, 稲見 泰生, 斉藤 直之 (原子力研究所), 第9回プログラミング—シンポジウム報告集 (1968)
* [リスト処理言語L6について](https://ipsj.ixsq.nii.ac.jp/ej/?action=repository_uri&item_id=237924), 高橋 耕貴, 原 文子, 渋谷 政昭 (統計数理研究所), 第10回プログラミング—シンポジウム報告集 (1969)

もちろん「LISP という言語」の話題も、早くから近年まで、非常に多く出ています。

* [記号操作と定理の機械的証明 (LISPに関する綜合報告)](https://ipsj.ixsq.nii.ac.jp/ej/?action=repository_uri&item_id=237690), 高須 達 (電気通信研究所), 第3回プログラミング—シンポジウム報告集 (1962)
* [LISPのデータ構造についての2, 3の考察](https://ipsj.ixsq.nii.ac.jp/ej/?action=repository_uri&item_id=223389), 佐藤 浩史 (お茶の水女子大学), 後藤 英一, 野崎 昭弘, 野下 浩下 (東京大学), 第14回プログラミング—シンポジウム報告集 (1973)
* [ミニコンLISPのうつしかえ](https://ipsj.ixsq.nii.ac.jp/ej/?action=repository_uri&item_id=238078), 石井 博, 前野 年紀 (立教大学), 第17回プログラミング・シンポジウム報告集 (1976)
* [LISP COMPILERの試作](https://ipsj.ixsq.nii.ac.jp/ej/?action=repository_uri&item_id=238107), 阿部 芳彦, 鈴木 正幸, 桂 重俊 (東北大学), 第18回プログラミング・シンポジウム報告集 (1977)
* [TAO Lisp開発の記録](https://ipsj.ixsq.nii.ac.jp/ej/?action=repository_uri&item_id=238294), 竹内 郁雄, 大里 延康 (日本電信電話公社武蔵野電気通信研究所), 第22回プログラミング・シンポジウム報告集 (1981)
* [並列Lispの処理系とアプリケーション](https://ipsj.ixsq.nii.ac.jp/ej/?action=repository_uri&item_id=238477), 岩崎 英哉, 寺田 実, 樋口 英幸 (東京大学), 第28回プログラミング・シンポジウム報告集 (1987)
* [キューマシンアーキテクチャとその並列Lisp処理系への応用](https://ipsj.ixsq.nii.ac.jp/ej/?action=repository_uri&item_id=239181), 前田 敦司 (慶應義塾大学), 第38回プログラミング・シンポジウム報告集 (1997)

### 論理型言語と Prolog

Prolog や、論理型言語に関連した話題もいくつか出てきますね。

* [PARALLEL PROLOG](https://ipsj.ixsq.nii.ac.jp/ej/?action=repository_uri&item_id=232224), 中島 秀之 (東京大学), 第21回プログラミング・シンポジウム報告集 (1980)
* [Prologによるプログラミング](https://ipsj.ixsq.nii.ac.jp/ej/?action=repository_uri&item_id=238327), 和田 英一, 戸村 哲, 中島 秀之, 木村 通男 (東京大学), 第23回プログラミング・シンポジウム報告集 (1982)
* [プログラム開発環境としての機能を重視したマイコンのためのPrologシステム](https://ipsj.ixsq.nii.ac.jp/ej/?action=repository_uri&item_id=238387), 萩野 達也 (Edinburgh大学), 桜川 貴司 (京都大学 数理解析研究所), 柴山 悦哉 (東京工業大学), 第25回プログラミング・シンポジウム報告集 (1984)
* [論理型言語におけるexplicitな否定知識](https://ipsj.ixsq.nii.ac.jp/ej/?action=repository_uri&item_id=238434), 中川 裕志 (横浜国立大学), 第27回プログラミング・シンポジウム報告集 (1986)
* [A Proposal for a Practical Prolog Dialect](https://ipsj.ixsq.nii.ac.jp/ej/?action=repository_uri&item_id=238478), Toshiaki Kurokawa, Naoyuki Tamura, Yasuo Asakawa, Hideaki Komatsu, Hiroaki, Etoh, Toshiyuki, Hama (Tokyo Research Laboratory, IBM Japan, Ltd.), 第28回プログラミング・シンポジウム報告集 (1987)
* [依存伝播](https://ipsj.ixsq.nii.ac.jp/ej/?action=repository_uri&item_id=237446), 橋田 浩一 (電子技術総合研究所), 第29回プログラミング・シンポジウム報告集 (1988)

### C言語

一方で「C 言語」そのものの話題、というのはあまり見かけません。特に UNIX 以降においてオペレーティング・システムを研究の対象にするときには、必然的に使う (使わなければならなくなる) のですが、逆に言語としての C 言語そのものへの注目は (この界隈では) あまり大きくなかったのかもしれません。

「研究」的な文脈の中では、道具として使うのはともかく研究対象にはしづらい、という感覚はわかるような気がします。当時は研究対象にできるコンパイラも多くありませんでしたし。

* [メインフレーム・コンピュータにおけるCプログラム開発環境](https://ipsj.ixsq.nii.ac.jp/ej/?action=repository_uri&item_id=238447), 安達 淳, 木村 友則, 村尾 裕一, 石田 晴久 (東京大学), 第27回プログラミング・シンポジウム報告集 (1986)
* [言語CのCAIシステムの設計とそのプログラム実行環境の開発](https://ipsj.ixsq.nii.ac.jp/ej/?action=repository_uri&item_id=237506), 森田 雅夫, 早川 栄一, 玉木 裕二, 並木 美太郎, 高橋 延匡 (東京農工大学), 第32回プログラミング・シンポジウム報告集 (1991)
* [C言語に対する形式的意味記述の一手法](https://ipsj.ixsq.nii.ac.jp/ej/?action=repository_uri&item_id=239086), 篠崎 政久, 宮寺 庸造, 米田 信夫 (東京電機大学), 第36回プログラミング・シンポジウム報告集 (1995)
* [プログラミング言語教育補助システムPLESS ―C言語教育補助システムCmaster―](https://ipsj.ixsq.nii.ac.jp/ej/?action=repository_uri&item_id=239866), 高岡 詠子, 仙北谷 祐輔, 小林 宏章, 鈴木 信吾 (千歳科学技術大学), 第44回プログラミング・シンポジウム報告集 (2003)

### Java

かと思えば Java の話題は (Java の出現以降) 意外とあります。仕様がはっきりしていたことや、高速化や GC などの挑戦しがいのある話題が多かったことが、背景にあるかもしれません。

* [Javaの高速化](https://ipsj.ixsq.nii.ac.jp/ej/?action=repository_uri&item_id=239609), 志村 浩也, 河場 基行, 木村 康則 ((株)富士通研究所), 第39回プログラミング・シンポジウム報告集 (1998)
* [Java in the Real World ― Javaはどこまで使える言語か](https://ipsj.ixsq.nii.ac.jp/ej/?action=repository_uri&item_id=239710), 山内 斉, 前田 敦司 (電気通信大学), 第41回プログラミング・シンポジウム報告集 (2000)
* [Java向け静的コンパイル方式](https://ipsj.ixsq.nii.ac.jp/ej/?action=repository_uri&item_id=239713), 千葉 雄司 (日立製作所システム開発研究所), 第41回プログラミング・シンポジウム報告集 (2000)
* [JavaでSchemeを書いてみれば](https://ipsj.ixsq.nii.ac.jp/ej/?action=repository_uri&item_id=239759), 湯淺 太一 (京都大学), 第42回プログラミング・シンポジウム報告集 (2001)
* [グローバルコンピューティングに適したJava実行環境の研究](https://ipsj.ixsq.nii.ac.jp/ej/?action=repository_uri&item_id=239782), 蟻川 浩, 河合 栄治, 砂原 秀樹 (奈良先端科学技術大学院大学), 第42回プログラミング・シンポジウム報告集 (2001)

### Ruby

Ruby の話題は、特に電子図書館に掲載されるようになったくらいのタイミングから、よく出るようになっています。その意味では本記事のスコープではありませんが、参考に挙げておきます。

* Rubyプログラムを高速に実行するための処理系の開発, 笹田 耕一 (東京農工大学), 第46回プログラミング・シンポジウム報告集 (2005)
    * すみません、第 46 回は電子図書館に掲載されていなかった最後の年で、鋭意作業中です。まもなく掲載できると思います。
    * 参考までに: [第 46 回のプログラム](https://prosym.org/46/program46.html)
* [Ruby1.9での高速なFiberの実装](https://ipsj.ixsq.nii.ac.jp/ej/?action=repository_uri&item_id=91444), 芝 哲史, 笹田 耕一 (東京大学), 第51回プログラミング・シンポジウム予稿集 (2010)
* [RubyによるOSの構築を目指して](https://ipsj.ixsq.nii.ac.jp/ej/?action=repository_uri&item_id=91436), 吉原 陽香 (東京農工大学), 笹田 耕一 (東京大学), 佐藤 未来子, 並木 美太郎 (東京農工大学), 第52回プログラミング・シンポジウム予稿集 (2011)
* [Rubyのスレッド実装の改善](https://ipsj.ixsq.nii.ac.jp/ej/?action=repository_uri&item_id=91437), 笹田 耕一 (東京大学), 第52回プログラミング・シンポジウム予稿集 (2011)
* [Rubyに対するGradual typingの導入に向けて](https://ipsj.ixsq.nii.ac.jp/ej/?action=repository_uri&item_id=222480), 丹治 将貴，中野 圭介，岩崎 英哉 (電気通信大学), 第58回プログラミング・シンポジウム予稿集 (2017)

ごく最近にも Ruby のインタプリタと JIT コンパイラを Rust で実装したような話題があったりしました。

* [Rubyの高速なインタプリタ及び実行時コンパイラの開発](https://ipsj.ixsq.nii.ac.jp/ej/?action=repository_uri&item_id=232307), 一色 聡一郎 (有限会社ソフィア), 第64回プログラミング・シンポジウム予稿集 (2023)

### JavaScript

JavaScript の話題も、やはり電子図書館に掲載されるようになった以降にしばしば見かけるようになりました。

* [JavaScriptマルチスレッドライブラリの実装と応用](https://ipsj.ixsq.nii.ac.jp/ej/?action=repository_uri&item_id=91514), 牧 大介, 岩崎 英哉 (電気通信大学), 第49回プログラミング･シンポジウム予稿集 (2008)
* [現代的JavaScriptエンジンの実装](https://ipsj.ixsq.nii.ac.jp/ej/?action=repository_uri&item_id=82711), 鈴木 勇介 (サイボウズ･ラボユース／慶応義塾大学理工学部), 第53回プログラミング･シンポジウム予稿集 (2012)

独自・多様なプログラミング言語
-------------------------------

独自のプログラミング言語と処理系を作ったような話も、初期から現在までよく出てきます。汎用的なものから、目的をはっきり絞ったようなものまで、いろいろありますね。

* [Self-Compiler](https://ipsj.ixsq.nii.ac.jp/ej/?action=repository_uri&item_id=237705), 西村 真一郎 (日本IBM), 第4回プログラミング—シンポジウム報告集 (1963)
    * Self というと、初期のプロトタイプベースのオブジェクト指向プログラミング言語に有名なもの [^self] があるんですが、それとはまったく別のものです。 (もちろん時代も違います)
* [増殖型言語SELFについて](https://ipsj.ixsq.nii.ac.jp/ej/?action=repository_uri&item_id=232110), 中田 育男, 浜田 穂積, 霜田 忠孝, 野木 兼六 (日立中研), 第12回プログラミング—シンポジウム報告集 (1971)
    * こちらも Self ですが、やっぱり別物です。
* [システム記述用言語PLDの使用経験と問題点](https://ipsj.ixsq.nii.ac.jp/ej/?action=repository_uri&item_id=232109), 伊藤 哲史 (日本情報処理開発センター), 第12回プログラミング—シンポジウム報告集 (1971)
* [システム・プログラム言語D32, D34の試作](https://ipsj.ixsq.nii.ac.jp/ej/?action=repository_uri&item_id=232111), 島内 剛一 (立大理), 和田 英一, 筧 捷彦 (東大工), 高橋 義造 (東芝), 四条 忠雄 (日科技研), 第12回プログラミング—シンポジウム報告集 (1971)
* [KSIM ―統計値の自動収集を考えたシミュレーション言語―](https://ipsj.ixsq.nii.ac.jp/ej/?action=repository_uri&item_id=232142), 山本 喜一 (慶応義塾大学), 第13回プログラミング—シンポジウム報告集 (1972)
* [表言語の提案 ―汎用二次元プログラミング言語とその変換アルゴリズムについて―](https://ipsj.ixsq.nii.ac.jp/ej/?action=repository_uri&item_id=232157), 守屋 慎次, 平松 啓二 (東京電機大学), 第13回プログラミング—シンポジウム報告集 (1972)
* [汎用グラフィック言語UNGL](https://ipsj.ixsq.nii.ac.jp/ej/?action=repository_uri&item_id=223408), 山本 欣子, 出羽 洋 ((財)日本情報処理開発センター), 第14回プログラミング—シンポジウム報告集 (1973)
* [LOREL言語の使用経験とその検討](https://ipsj.ixsq.nii.ac.jp/ej/?action=repository_uri&item_id=223427), 片山 卓也, 榎本 進, 榎本 肇 (東京工業大学), 第15回プログラミング—シンポジウム報告集 (1974)
* [システムプログラム作成用ミニ言語](https://ipsj.ixsq.nii.ac.jp/ej/?action=repository_uri&item_id=238034), 辻 尚史, 木村 泉 (東京工業大学), 第16回プログラミング—シンポジウム報告集 (1975)
* [数式処理言語ALの特徴とその意義](https://ipsj.ixsq.nii.ac.jp/ej/?action=repository_uri&item_id=238079), 池原 悟, 岡田 博 (横須賀電気通信研究所), 第17回プログラミング・シンポジウム報告集 (1976)
* [PRIMALの設計・処理系作成・使用経験](https://ipsj.ixsq.nii.ac.jp/ej/?action=repository_uri&item_id=238141), 野下 浩平, 山田 重夫 (電気通信大学), 第19回プログラミング・シンポジウム報告集 (1978)
* [イオタ言語とイオタシステム ―階層的プログラム開発とその検証のための言語と方法とシステム―](https://ipsj.ixsq.nii.ac.jp/ej/?action=repository_uri&item_id=238143), 中島 玲二, 中原 早生 (京都大学数理解析研究所), 本田 道夫 (香川大学), 第19回プログラミング・シンポジウム報告集 (1978)
* [画像解釈言語PILSについて](https://ipsj.ixsq.nii.ac.jp/ej/?action=repository_uri&item_id=238332), 森 英雄 (山梨大学), 第23回プログラミング・シンポジウム報告集 (1982)
* [推論関係型DBMS・Adbisにおける会話型データ操作言語Tsuno](https://ipsj.ixsq.nii.ac.jp/ej/?action=repository_uri&item_id=238352), 高木 利久, 松尾 文碩, 二村 祥一, 牛島 和夫 (九州大学), 第24回プログラミング・シンポジウム報告集 (1983)
* [TAO―Lisp, Prolog, Smalltalkの調和平均](https://ipsj.ixsq.nii.ac.jp/ej/?action=repository_uri&item_id=238359), 竹内 郁雄, 奥乃 博, 大里 延康 (日本電信電話公社 武蔵野電気通信研究所), 第24回プログラミング・シンポジウム報告集 (1983)
* [論理的言語NUにおける知的プログラム](https://ipsj.ixsq.nii.ac.jp/ej/?action=repository_uri&item_id=237493), 池田 靖雄, 細野 千春, 辻 尚史, 五十嵐 滋 (筑波大学), 第32回プログラミング・シンポジウム報告集 (1991)
* [実世界モデルにもとづく言語NAIVE](https://ipsj.ixsq.nii.ac.jp/ej/?action=repository_uri&item_id=238934), 日野 克重, 河田 汎 (富士通(株)), 第34回プログラミング・シンポジウム報告集 (1993)
* [学校教育用オブジェクト指向言語「Dolittle」の提案](https://ipsj.ixsq.nii.ac.jp/ej/?action=repository_uri&item_id=239756), 兼宗 進, 久野 靖 (筑波大学), 第42回プログラミング・シンポジウム報告集 (2001)

[^self]: [Self (Wikipedia)](https://ja.wikipedia.org/wiki/Self)

日本語プログラミング
---------------------

いわゆる「日本語によるプログラミング (言語)」というのも、一定の人気があり続けている話題ですね。

* [日本語の語順と逆ポーランド記法](https://ipsj.ixsq.nii.ac.jp/ej/?action=repository_uri&item_id=237780), 水谷 静夫 (東京女子大), 第7回プログラミング—シンポジウム報告集 (1966)
* [日本語によるCOBOL](https://ipsj.ixsq.nii.ac.jp/ej/?action=repository_uri&item_id=237842), 西村 恕彦 (電気試験所), 第8回プログラミング—シンポジウム報告集 (1967)
* [記号処理のためのヤマト語](https://ipsj.ixsq.nii.ac.jp/ej/?action=repository_uri&item_id=237938), 西村 恕彦 (通産省電気試験所), 第10回プログラミング—シンポジウム報告集 (1969)
* [日本語プログラム言語の設計](https://ipsj.ixsq.nii.ac.jp/ej/?action=repository_uri&item_id=239637), 今城 哲二 (日立製作所ソフトウェア事業部及び奈良先端科学技術大学院大学), 第40回プログラミング・シンポジウム報告集 (1999)
* [プログラミング言語としての日本語](https://ipsj.ixsq.nii.ac.jp/ej/?action=repository_uri&item_id=239874), 岡田 健, 中鉢 欣秀 (慶應義塾大学), 鈴木 弘 (東京都立航空工業高等専門学校), 大岩 元 (慶應義塾大学), 第44回プログラミング・シンポジウム報告集 (2003)

プログラミング言語の俯瞰
-------------------------

プログラミング言語分野の傾向を俯瞰したような議論というのもいくつかあります。

"Problem Oriented Language" や「問題向き言語」という言葉が出てくるんですが、これは現代でいう「高級言語」のような概念を指しているのかな…、という雰囲気です。

* [最近のProblem Oriented Languageについて](https://ipsj.ixsq.nii.ac.jp/ej/?action=repository_uri&item_id=237655), 今村 茂雄 (日本IBM), 第2回プログラミング–シンポジウム報告集補遺 (1961)
* [プログラミング言語の動向と標準化](https://ipsj.ixsq.nii.ac.jp/ej/?action=repository_uri&item_id=237970), 竹下 亨 (日本IBM), 第11回プログラミング—シンポジウム報告集 (1970)
* [問題向き言語に対する意味論的メタ言語とそのコンパイラ](https://ipsj.ixsq.nii.ac.jp/ej/?action=repository_uri&item_id=232101), 渡辺 坦 (日立製作所中央研究所), 第12回プログラミング—シンポジウム報告集 (1971)
* [工業用計算機言語の開発と標準化の動向](https://ipsj.ixsq.nii.ac.jp/ej/?action=repository_uri&item_id=232150), 首藤 勝 (三菱電機), 第13回プログラミング—シンポジウム報告集 (1972)
* [人工知能のプログラミング言語](https://ipsj.ixsq.nii.ac.jp/ej/?action=repository_uri&item_id=223420), 古川 康一 (電子技術総合研究所), 第15回プログラミング—シンポジウム報告集 (1974)

プログラミング方法論
---------------------

### オブジェクト指向

いわゆる「オブジェクト指向」に関係する初期の議論なども見当たります。とはいえ (現代でも一部で見かける) 「オブジェクト指向という考え方」そのものについてや「流派」の議論というのはあまりないかもしれませんが。

* [オブジェクト・オリエンテッド並列処理言語の概念構成](https://ipsj.ixsq.nii.ac.jp/ej/?action=repository_uri&item_id=238389), 福井 眞吾 (東京工業大学), 第25回プログラミング・シンポジウム報告集 (1984)
* [オブジェクト指向による画面エディタの部品化](https://ipsj.ixsq.nii.ac.jp/ej/?action=repository_uri&item_id=238483), 天海 良治 (NTT電気通信研究所), 第28回プログラミング・シンポジウム報告集 (1987)
* [オブジェクト指向における再利用のための構成支援環境](https://ipsj.ixsq.nii.ac.jp/ej/?action=repository_uri&item_id=237477), 片山 佳則 (富士通株式会社), 第30回プログラミング・シンポジウム報告集 (1989)
* [音楽情報処理とオブジェクト指向](https://ipsj.ixsq.nii.ac.jp/ej/?action=repository_uri&item_id=239139), 平賀 瑠美, 五十嵐 滋, 松浦,陽平 (筑波大学), 第37回プログラミング・シンポジウム報告書 (1996)

### 構造化プログラミング

オブジェクト指向の前には「構造化プログラミング」というのが言われたこともありました。 (今ではもう当たり前のものになっていますが)

* [ストラクチャードプログラミング支援システムSPOTとその評価](https://ipsj.ixsq.nii.ac.jp/ej/?action=repository_uri&item_id=238026), 紫合 治, 下村 建之, 井元 邦夫, 岩本 莞二, 前島 亨 (日本電気), 第16回プログラミング—シンポジウム報告集 (1975)
* [構造化プログラミング例とデータ構造](https://ipsj.ixsq.nii.ac.jp/ej/?action=repository_uri&item_id=238027), 溝口 徹夫, 厚井 裕司, 田中 千代治, 首藤 勝 (三菱電機), 第16回プログラミング—シンポジウム報告集 (1975)

### 型

初期からの、いわゆる「(データ) 型」についての議論というのもいくつか見られます。

* [プログラミング言語におけるデータ型、データ構造およびデータ・モードについて](https://ipsj.ixsq.nii.ac.jp/ej/?action=repository_uri&item_id=238075), 有澤 誠 (山梨大学), 第17回プログラミング・シンポジウム報告集 (1976)
* [標準抽象データ構造に基づくデータ抽象化技法と仕様記述法](https://ipsj.ixsq.nii.ac.jp/ej/?action=repository_uri&item_id=238076), 藤林 信也, 紫合 治, 岩元 莞二 (日本電気(株)中央研究所), 第17回プログラミング・シンポジウム報告集 (1976)
* [データタイプチェックに関する一考察](https://ipsj.ixsq.nii.ac.jp/ej/?action=repository_uri&item_id=238102), 後藤 英一 (東京大学), 井田 哲雄 (理化学研究所), 第18回プログラミング・シンポジウム報告集 (1977)

コンパイラ
-----------

プログラミング言語の議論があるのですから、もちろんその中や関連として、コンパイラ、言語処理系、構文解析 (パーサー) などの話題もありますね。

### 自動プログラミング

「自動プログラミング」という言葉がちょくちょく出てきます。「自動プログラミング」 [^automatic-programming] というのは、時代によっては「コンパイル」「コンパイラ」と近い認識で解釈されたりもするのですが、さらに前の時代だともう少し「機械に近い」ところをゴリゴリやっていたりして、時代によって意味が移り変わってきた言葉だ、というのがなんとなく見えてきそうです。近年で「自動プログラミング」といったら、ソースコード生成とか生成 AI の話になるんでしょうね。

[^automatic-programming]: [自動プログラミング (Wikipedia)](https://ja.wikipedia.org/wiki/%E8%87%AA%E5%8B%95%E3%83%97%E3%83%AD%E3%82%B0%E3%83%A9%E3%83%9F%E3%83%B3%E3%82%B0)

* [行列計算の自動プログラミング](https://ipsj.ixsq.nii.ac.jp/ej/?action=repository_uri&item_id=232025), 島内 武彦 (東大理), 第1回プログラミング–シンポジウム報告集 (1960)
* [論理設計と自動プログラミング](https://ipsj.ixsq.nii.ac.jp/ej/?action=repository_uri&item_id=237668), 高橋 茂 (電気試験所), 第3回プログラミング—シンポジウム報告集 (1962)

### コンパイラ (・コンパイラ)

各プログラミング言語のコンパイラの議論は既に各言語のところでも取り上げているので、ここではコンパイラ一般の話題や、構文解析器の話、コンパイラ生成系 (コンパイラ・コンパイラ) の話などを取り上げてみます。

特に初期のものでは、汎用的なコンパイラの技術の話というより、個々のマシンにもかなり依った議論が見られたりします。

* [PROGRAM GENERATOR "PREMAP"](https://ipsj.ixsq.nii.ac.jp/ej/?action=repository_uri&item_id=237734), 和田 英一, 原田 睦明 (小野田セメント), 第5回プログラミング—シンポジウム報告集 (1964)
* [Compiler Generating System](https://ipsj.ixsq.nii.ac.jp/ej/?action=repository_uri&item_id=223035), 藤野 喜一 (早稲田大学), 第6回プログラミング—シンポジウム報告集 (1965)
* [計算機を用いたコンパイラ作成自動化の実験](https://ipsj.ixsq.nii.ac.jp/ej/?action=repository_uri&item_id=237796), 首藤 勝, 魚田 勝臣, 居原田 邦男 (三菱電機), 第7回プログラミング—シンポジウム報告集 (1966)
* [コンパイラ作製自動化の諸方法](https://ipsj.ixsq.nii.ac.jp/ej/?action=repository_uri&item_id=232097), 井上 謙蔵 (富士通), 第12回プログラミング—シンポジウム報告集 (1971)
* [Compiler Generatorへの入力データ](https://ipsj.ixsq.nii.ac.jp/ej/?action=repository_uri&item_id=232102), 渡辺 勝正 (京都大学), 第12回プログラミング—シンポジウム報告集 (1971)
* [コンパイラ記述言語の一つの試み](https://ipsj.ixsq.nii.ac.jp/ej/?action=repository_uri&item_id=232113), 清水 公子, 中川 雅子, 石田 晏穂 (東大・物性研), 第12回プログラミング—シンポジウム報告集 (1971)
* [右順位文法に対する構文解析プログラムの発生](https://ipsj.ixsq.nii.ac.jp/ej/?action=repository_uri&item_id=232100), 井上 謙蔵, 阿部 克, 西村 公一 (富士通), 第12回プログラミング—シンポジウム報告集 (1971)
* [コンパイラ・コンパイラ手法による問題向き言語Xのコンパイラ作成](https://ipsj.ixsq.nii.ac.jp/ej/?action=repository_uri&item_id=223396), 森 洋之, 中島 秀和, 下村 津之 (日本電気株式会社), 第14回プログラミング—シンポジウム報告集 (1973)
* [順位文法によるプログラミング言語の解析とコンパイラの構成](https://ipsj.ixsq.nii.ac.jp/ej/?action=repository_uri&item_id=223405), 浅井 清, 富山 峯秀 (日本原子力研究所), 第14回プログラミング—シンポジウム報告集 (1973)
* [文脈自由言語の誤り訂正時間](https://ipsj.ixsq.nii.ac.jp/ej/?action=repository_uri&item_id=223402), 沢野 明郎, 岩元 莞二 (日本電気中央研究所), 第14回プログラミング—シンポジウム報告集 (1973)
* [SYNTAX-DIRECTED PRETTY-PRINTER](https://ipsj.ixsq.nii.ac.jp/ej/?action=repository_uri&item_id=238077), 鶴田 陽和, 武市 正人, 和田 英一 (東京大学), 第17回プログラミング・シンポジウム報告集 (1976)
* [字句解析部の高速化について－ソフトウェアの調整技法－](https://ipsj.ixsq.nii.ac.jp/ej/?action=repository_uri&item_id=232179), 白濱 律雄, 前野 年紀 (東京工業大学), 第20回プログラミング・シンポジウム報告集 (1979)
* [属性文法によるコンパイラ生成系](https://ipsj.ixsq.nii.ac.jp/ej/?action=repository_uri&item_id=238412), 石塚 治志, 佐々 政孝 (筑波大学), 第26回プログラミング・シンポジウム報告集 (1985)
* [再帰呼び出しのコード生成と速度](https://ipsj.ixsq.nii.ac.jp/ej/?action=repository_uri&item_id=238488), 梅村 恭司 (NTT 電気通信研究所), 第28回プログラミング・シンポジウム報告集 (1987)
* [Coupled Context Free Grammarに基づくプログラミング言語の実行方式について](https://ipsj.ixsq.nii.ac.jp/ej/?action=repository_uri&item_id=238476), 山下 義行, 中田 育男 (筑波大学), 第28回プログラミング・シンポジウム報告集 (1987)
* [コンパイラのおける意味処理の並列化](https://ipsj.ixsq.nii.ac.jp/ej/?action=repository_uri&item_id=237472), 西山 博泰, 板野 肯三 (筑波大学), 第30回プログラミング・シンポジウム報告集 (1989)
* [SSA (Static Single Assignment)形式を使った最適化コンパイラ](https://ipsj.ixsq.nii.ac.jp/ej/?action=repository_uri&item_id=239206), 佐藤 三久 (電子技術総合研究所), 第35回プログラミング・シンポジウム報告集 (1994)
* [コンパイラ・エンジニアリング](https://ipsj.ixsq.nii.ac.jp/ej/?action=repository_uri&item_id=239179), 渡邊 坦, 鈴木 貢 (電気通信大学), 第38回プログラミング・シンポジウム報告集 (1997)

GC
---

GC (Garbage Collection) に特化したような話も、一定程度あったりします。

* [GCの視覚化](https://ipsj.ixsq.nii.ac.jp/ej/?action=repository_uri&item_id=239141), 萩原 拓也, 田中 詠子, 岩井 輝男, 田中 良夫, 前田 敦司, 松井 祥悟, 中西 正和 (慶応義塾大学), 第37回プログラミング・シンポジウム報告書 (1996)
* [並列ガーベジコレクションの実用化技術](https://ipsj.ixsq.nii.ac.jp/ej/?action=repository_uri&item_id=239667), 松井 祥悟 (神奈川大学), 田中 良夫 (新情報処理開発機構), 前田 敦司 (電気通信大学), 中西 正和 (慶應義塾大学), 第40回プログラミング・シンポジウム報告集 (1999)
* [回収率に基づきヒープサイズを動的に変更する世代GC方式](https://ipsj.ixsq.nii.ac.jp/ej/?action=repository_uri&item_id=239691), 吉川 隆英, 近山 隆 (東京大学), 第41回プログラミング・シンポジウム報告集 (2000)

ハードウェア化
---------------

プログラミング言語処理系をハードウェア化してしまおう、みたいな話も、実はけっこう以前からしばしばなされています。

* [マイクロPlanエンジン](https://ipsj.ixsq.nii.ac.jp/ej/?action=repository_uri&item_id=238291), 和田 英一, 戸村 哲, 梅村 恭司, 寺田 実 (東京大学), 第22回プログラミング・シンポジウム報告集 (1981)
* [数式処理・LISPマシンFLATSについて](https://ipsj.ixsq.nii.ac.jp/ej/?action=repository_uri&item_id=238385), 後藤 英一, 鈴木 正幸, 清水 謙多郎 (東京大学), 相場 嵩, 稲田 信幸 (理化学研究所), 第25回プログラミング・シンポジウム報告集 (1984)
* [マルチプロセッサLISPマシンMacELIS II](https://ipsj.ixsq.nii.ac.jp/ej/?action=repository_uri&item_id=237531), 三上 博英, 村上 健一郎 (NTTソフトウェア研究所), 第31回プログラミング・シンポジウム報告集 (1990)
* [マイクロPlanの近代的な実現法 ―μPlan'93 (15年ぶりの帰ってきたマイクロPlan)―](https://ipsj.ixsq.nii.ac.jp/ej/?action=repository_uri&item_id=238933), 多田 好克 (電気通信大学), 第34回プログラミング・シンポジウム報告集 (1993)

検証・テスト・プロファイル
---------------------------

プログラムの検証、テストなどに関連する話題も一定数ありますね。

* [プログラムの診断 (1)](https://ipsj.ixsq.nii.ac.jp/ej/?action=repository_uri&item_id=237939), 大槻 説乎, 久原 由美子 (九州大学), 秦野 和郎 (名古屋大学), 第10回プログラミング—シンポジウム報告集 (1969)
* [計算機システム評価の数学的理論―システム性能の感度について―](https://ipsj.ixsq.nii.ac.jp/ej/?action=repository_uri&item_id=238053), 原田 紀夫 (日本電気中央研究所), 第16回プログラミング—シンポジウム報告集 (1975)
* [高度な並列性を内部にもつソフトウェアシステムに対する仕様及び検証技法について](https://ipsj.ixsq.nii.ac.jp/ej/?action=repository_uri&item_id=232186), 米澤 明憲 (東京工業大学), 第20回プログラミング・シンポジウム報告集 (1979)
* [アドレスの誤操作に起因するプログラムの虫の検出方法について](https://ipsj.ixsq.nii.ac.jp/ej/?action=repository_uri&item_id=238319), 井田 哲雄 (理化学研究所), 板野 肯三 (筑波大学), 第23回プログラミング・シンポジウム報告集 (1982)
* [一二の並行プロセスの検証問題について](https://ipsj.ixsq.nii.ac.jp/ej/?action=repository_uri&item_id=238930), 水谷 哲也, 五十嵐 滋, 小宮山 弘樹, 辻 尚史 (筑波大学), 第34回プログラミング・シンポジウム報告集 (1993)

証明・算術
-----------

プログラムで証明をする、みたいな話は、かなり以前からあったようです。

* [数学の証明の計算機による実施のためのプログラミング](https://ipsj.ixsq.nii.ac.jp/ej/?action=repository_uri&item_id=232040), 高須 達 (電気通信研究所), 第1回プログラミング–シンポジウム報告集 (1960)
* [証明のプログラミング](https://ipsj.ixsq.nii.ac.jp/ej/?action=repository_uri&item_id=237720), 西村 敏男 (法政大), 第4回プログラミング—シンポジウム報告集 (1963)
* [数学の証明のプログラム](https://ipsj.ixsq.nii.ac.jp/ej/?action=repository_uri&item_id=237749), 島内 剛一 (立大理), 第5回プログラミング—シンポジウム報告集 (1964)
* [情報検索とSL―導出定理証明プログラム](https://ipsj.ixsq.nii.ac.jp/ej/?action=repository_uri&item_id=223424), 二村 良彦 (日立中央研究所), 第15回プログラミング—シンポジウム報告集 (1974)

かと思えば、いわゆる算数の文章問題的なものを解こうとする、今でいう東ロボくん的なチャレンジの話も当時からあったり。

* [算術の問題を解くプログラム](https://ipsj.ixsq.nii.ac.jp/ej/?action=repository_uri&item_id=237649), 清水 辰次郎 (大阪府大), 第2回プログラミング–シンポジウム報告集 (1961)
* [文章で表わされた算術を解くプログラム(続)](https://ipsj.ixsq.nii.ac.jp/ej/?action=repository_uri&item_id=237719), 清水 辰次郎 (東京理科大), 杉林 益太郎 (大阪府大), 第4回プログラミング—シンポジウム報告集 (1963)

暗号・セキュリティ
-------------------

暗号やセキュリティの話題というのも時おり出ています。

* [公衆暗号系の構想 ―暗号とNP完全問題―](https://ipsj.ixsq.nii.ac.jp/ej/?action=repository_uri&item_id=238140), 赤木 昭夫 (NHK解説委員室), 第19回プログラミング・シンポジウム報告集 (1978)
* [数式処理システムの現代暗号への応用](https://ipsj.ixsq.nii.ac.jp/ej/?action=repository_uri&item_id=239661), 竹島 卓 (富士通研究所), 第40回プログラミング・シンポジウム報告集 (1999)
* [MPUの性能に応じた共通鍵暗号の高速実装法](https://ipsj.ixsq.nii.ac.jp/ej/?action=repository_uri&item_id=239662), 下山 武司 (通信・放送機構), 第40回プログラミング・シンポジウム報告集 (1999)

計算センター・データセンター
-----------------------------

大学や企業における計算機センター、データセンター、みたいな話題もあります。

* [企業における計算センターの運営について](https://ipsj.ixsq.nii.ac.jp/ej/?action=repository_uri&item_id=237759), 魚木 五夫 (東洋工業), 第5回プログラミング—シンポジウム報告集 (1964)
* [計算センターの運営について](https://ipsj.ixsq.nii.ac.jp/ej/?action=repository_uri&item_id=237760) 森口 繁一 (東京大学), 第5回プログラミング—シンポジウム報告集 (1964)
* [九州工業大学情報処理教育センターのシステム開発・運用状況について](https://ipsj.ixsq.nii.ac.jp/ej/?action=repository_uri&item_id=238118), 中山 泰雄, 深川 幸紀, 矢鳴 虎夫, 吉田 将 (九州工業大学), 第18回プログラミング・シンポジウム報告集 (1977)
* [手作り計算機実習室の手作りCCTV --計算機教育用映像システムの構築--](https://ipsj.ixsq.nii.ac.jp/ej/?action=repository_uri&item_id=237461), 和田 勉 (長野大学), 堀内 一夫 (第一無線), 第30回プログラミング・シンポジウム報告集 (1989)
* [スーパーコンピュータ, UNIX, ワークステーション --東工大の計算機環境はどうなったか--](https://ipsj.ixsq.nii.ac.jp/ej/?action=repository_uri&item_id=237462), 前野 年紀, 松田 裕幸, 太田 昌孝, 大野 浩之 (東京工業大学)

高性能計算 (スパコン)
----------------------

大学の計算機センターといえばスパコンでしょ (?) ということで、いわゆるスーパーコンピューター (高性能計算; HPC) の話題もしばしばあります。

* [スーパー・コンピュータによるPrologの高速実行](https://ipsj.ixsq.nii.ac.jp/ej/?action=repository_uri&item_id=238410), 金田 泰 (日立製作所), 第26回プログラミング・シンポジウム報告集 (1985)
* [スーパーコンピュータ上でのグラフ問題の高速化](https://ipsj.ixsq.nii.ac.jp/ej/?action=repository_uri&item_id=237528), 安村 通晃, 小島 啓二 (日立製作所), 第31回プログラミング・シンポジウム報告集 (1990)

通信・ネットワーク
-------------------

また、計算機センター・データセンターといえば、ネットワークの話も欠くことができません。現在の「日本の」インターネット環境の起源・ベースとなった JUNET [^junet] の話から、その後のネットワークを利用した諸々まで、様々な話題があります。

[^junet]: [JUNET (Wikipedia)](https://ja.wikipedia.org/wiki/JUNET)

* [キャンパスネットワーク設置の経験](https://ipsj.ixsq.nii.ac.jp/ej/?action=repository_uri&item_id=238463), 前野 年紀 (東京工業大学), 第28回プログラミング・シンポジウム報告集 (1987)
* [JUNETの現状と今後](https://ipsj.ixsq.nii.ac.jp/ej/?action=repository_uri&item_id=238464), 村井 純, 加藤 朗 (東京工業大学), 第28回プログラミング・シンポジウム報告集 (1987)
* [junetの漢字コード ―決定顛末記―](https://ipsj.ixsq.nii.ac.jp/ej/?action=repository_uri&item_id=238465), 小川 貴英 (津田塾大学), 第28回プログラミング・シンポジウム報告集 (1987)
* [NTT-Internet構築の経験と実際](https://ipsj.ixsq.nii.ac.jp/ej/?action=repository_uri&item_id=237463), 村上 健一郎, 天海 良治 (NTTソフトウェア研究所), 第30回プログラミング・シンポジウム報告集 (1989)
* [学術・研究目的のコンピュータコミュニケーション基盤の構築](https://ipsj.ixsq.nii.ac.jp/ej/?action=repository_uri&item_id=237530), 村井 純 (東京大学), 第31回プログラミング・シンポジウム報告集 (1990)
* [会議とネットワーク環境](https://ipsj.ixsq.nii.ac.jp/ej/?action=repository_uri&item_id=238889), 村井 純, 楠本 博之, 加藤 朗 (慶應義塾大学), 稗田 薫 (上智大学), 第33回プログラミング・シンポジウム報告集 (1992)
* [高品質な遠隔授業/会議システムにおける問題点と使い勝手について](https://ipsj.ixsq.nii.ac.jp/ej/?action=repository_uri&item_id=239868), 中野 瞳, 堀 良彰, 河原 一彦, 平山 善一, 藤村 直美 (九州芸術工科大学), 第44回プログラミング・シンポジウム報告集 (2003)

教育
-----

プログラミングの教育の話がちょくちょくあるのも、かなり初期からのようです。

* [日本の大学における計算機教育について](https://ipsj.ixsq.nii.ac.jp/ej/?action=repository_uri&item_id=237789), 一松 信 (立教大理), 第7回プログラミング—シンポジウム報告集 (1966)
* [高校における電子計算機教育について](https://ipsj.ixsq.nii.ac.jp/ej/?action=repository_uri&item_id=237832), 藤野 喜一, 喜多見 孟 (早稲田大学), 第8回プログラミング—シンポジウム報告集 (1967)
* [中学生を対象とするプログラミング実験教室](https://ipsj.ixsq.nii.ac.jp/ej/?action=repository_uri&item_id=237945), 竹下 亨, 苅部 英司 (日本アイ・ビー・エム(株)), 第10回プログラミング—シンポジウム報告集 (1969)
* [教育実習用ミニコンピュータのコンパイラ開発コンクールについて](https://ipsj.ixsq.nii.ac.jp/ej/?action=repository_uri&item_id=232162), 森口 繁一 (東京大学), 第13回プログラミング—シンポジウム報告集 (1972)
* [PL/I初心者エラー相談システム](https://ipsj.ixsq.nii.ac.jp/ej/?action=repository_uri&item_id=237521), 木下 成顕, 平賀 譲 (図書館情報大学), 第31回プログラミング・シンポジウム報告集 (1990)
* [計算機実習の実態調査 ― 事象時刻データから何がわかるか? ―](https://ipsj.ixsq.nii.ac.jp/ej/?action=repository_uri&item_id=239202), 角田 博保 (電気通信大学), 第35回プログラミング・シンポジウム報告集 (1994)
* [教育用オブジェクト指向言語「ドリトル」を活用した学校教育](https://ipsj.ixsq.nii.ac.jp/ej/?action=repository_uri&item_id=239867), 兼宗 進 (筑波大学), 中谷 多哉子 ((有)エス・ラグーン), 御手洗 理英 ((株)アーマット), 福井 眞吾, 久野 靖 (筑波大学), 第44回プログラミング・シンポジウム報告集 (2003)

ゲーム・パズル
---------------

プロシンには [GPCC (Games and Puzzles Competitions on Computers)](https://hp.vector.co.jp/authors/VA003988/gpcc/gpcc.htm) という分科会もあるのですが、「ゲームやパズルを解く」という話題もかなり以前からよくあるようです。

* [パネル討論 電子計算機とゲーム](https://ipsj.ixsq.nii.ac.jp/ej/?action=repository_uri&item_id=237918), 一松 信 (立教大理), 池野 信一 (通研), 越智 利夫 (日立), 清水 達雄 (清水建設), 細井 勉 (東大理), 中山 健 (武蔵工大), 第10回プログラミング—シンポジウム報告集 (1969)
* [箱入娘パズルをめぐって](https://ipsj.ixsq.nii.ac.jp/ej/?action=repository_uri&item_id=223419), 加藤 美治, 鈴木 洋介, 江守 泰文 (慶大工), 戸田 英雄 (ETL), 第15回プログラミング—シンポジウム報告集 (1974)
* [計算機によるゲームとパズル](https://ipsj.ixsq.nii.ac.jp/ej/?action=repository_uri&item_id=238025), 一松 信 (京都大学数理解析研究所), 第16回プログラミング—シンポジウム報告集 (1975)
* [将棋ゲームの指し手の理解モデルとオンライン処理について](https://ipsj.ixsq.nii.ac.jp/ej/?action=repository_uri&item_id=238110), 坂本 義行 (電子技術総合研究所), 佐藤 雅之 (電気通信大学), 第18回プログラミング・シンポジウム報告集 (1977)
* [コンピュータを用いた詰め将棋の評価と分析](https://ipsj.ixsq.nii.ac.jp/ej/?action=repository_uri&item_id=237504), 松原 仁, 半田 剣一, 元吉 文男 (電子技術総合研究所), 第32回プログラミング・シンポジウム報告集 (1991)
* [囲碁対局システムの研究](https://ipsj.ixsq.nii.ac.jp/ej/?action=repository_uri&item_id=239092)  吉田 真, 丸山 啓 (東京農工大学), 丸山 真佐夫 (木更津工業高等専門学校), 早川 栄一, 並木 美太郎, 高橋 延匡 (東京農工大学), 第36回プログラミング・シンポジウム報告集 (1995)
* [アーケードゲームのテクノロジー (その1)](https://ipsj.ixsq.nii.ac.jp/ej/?action=repository_uri&item_id=239131), 宮沢 篤 (日本アイ・ビー・エム株式会社 東京基礎研究所), 駒野目 裕久 (池上通信機株式会社), 第37回プログラミング・シンポジウム報告書 (1996)
* [「倉庫番」の問題の自動作成](https://ipsj.ixsq.nii.ac.jp/ej/?action=repository_uri&item_id=239172), 村瀬 芳生 (図書館情報大学), 松原 仁 (電子技術総合研究所), 平賀 譲 (図書館情報大学), 第38回プログラミング・シンポジウム報告集 (1997)

ゲームとプログラミング言語を合わせたような話題もあったりします。

* [二人用STAR TREK ―リスト処理言語LIPXによるCo-Operating Processesの実験―](https://ipsj.ixsq.nii.ac.jp/ej/?action=repository_uri&item_id=238131), 竹内 郁雄, 奥乃 博 (電電公社武蔵野通研), 第19回プログラミング・シンポジウム報告集 (1978)
* [アクションゲーム記述に特化した言語](https://ipsj.ixsq.nii.ac.jp/ej/?action=repository_uri&item_id=239870), 西森 丈俊, 久野 靖 (筑波大学), 第44回プログラミング・シンポジウム報告集 (2003)

ユーザー・インターフェース
---------------------------

ユーザー・インターフェース的な話題も、比較的早くからあったようです。

* [汝のユーザを知れ ―人間中心のシステム・デザインへ向けて―](https://ipsj.ixsq.nii.ac.jp/ej/?action=repository_uri&item_id=238440), 高嶋 孝明, 加藤 隆 (日本アイ・ビー・エム(株) 大和研究所), 第27回プログラミング・シンポジウム報告集 (1986)
* [カスタマイズを重視したユーザ・インタフェース・マネジメント・システム](https://ipsj.ixsq.nii.ac.jp/ej/?action=repository_uri&item_id=237438), 横井 伸司 (日本アイ・ビー・エム株式会社東京基礎研究所), 第29回プログラミング・シンポジウム報告集 (1988)
* [窓はねずみ無しでも繰れるか](https://ipsj.ixsq.nii.ac.jp/ej/?action=repository_uri&item_id=237469), 久野 靖 (東京工業大学), 角田 博保 (電気通信大学), 第30回プログラミング・シンポジウム報告集 (1989)
* [Xウィンドゥ上のマルチメディアユーザインタフェース構築環境: 鼎](https://ipsj.ixsq.nii.ac.jp/ej/?action=repository_uri&item_id=237470), 暦本 純一, 菅井 勝, 森 岳志, 内山 厚子, 垂水 浩幸, 杉山 高弘, 秋口 忠三, 山崎 剛 (日本電気株式会社), 第30回プログラミング・シンポジウム報告集 (1989)
* [航行めがねワールド: 反応する現実のビュー](https://ipsj.ixsq.nii.ac.jp/ej/?action=repository_uri&item_id=238885), 大座畑 重光 (マッキンテリジェンス(株)), 第33回プログラミング・シンポジウム報告集 (1992)
* [視線風ポインティングデバイスの試作と評価 ― 目は口ほどにものをいうか? ―](https://ipsj.ixsq.nii.ac.jp/ej/?action=repository_uri&item_id=239214), 赤池 英夫, 角田 博保 (電気通信大学) 飯島 純一 (明星大学), 第35回プログラミング・シンポジウム報告集 (1994)
* [ユーザインターフェースの独立](https://ipsj.ixsq.nii.ac.jp/ej/?action=repository_uri&item_id=239215), 立山 義祐, 寺田 実 (東京大学), 第35回プログラミング・シンポジウム報告集 (1994)
* [アイコンは投げられるか?](https://ipsj.ixsq.nii.ac.jp/ej/?action=repository_uri&item_id=239216), 久野 靖, 角田 博保, 大木 敦雄, 粕川 正充 (筑波大学), 第35回プログラミング・シンポジウム報告集 (1994)
* [なめらかなユーザインタフェース](https://ipsj.ixsq.nii.ac.jp/ej/?action=repository_uri&item_id=239111), 増井 俊之, 水口 充, George Borden, 柏木 宏一 (シャープ株式会社), 第37回プログラミング・シンポジウム報告書 (1996)
* [実世界指向プログラミング](https://ipsj.ixsq.nii.ac.jp/ej/?action=repository_uri&item_id=239639), 増井 俊之 (ソニーコンピュータサイエンス研究所), 第40回プログラミング・シンポジウム報告集 (1999)

画像・絵画
-----------

ゲームといえば…、というわけでもありませんが、絵画・画像をあつかうような話も定期的に出ています。

* [コンピュータによる色彩画](https://ipsj.ixsq.nii.ac.jp/ej/?action=repository_uri&item_id=237926), 中島 尙正 (東大工学部), 第10回プログラミング—シンポジウム報告集 (1969)
* [計算機による美の扱いの試み](https://ipsj.ixsq.nii.ac.jp/ej/?action=repository_uri&item_id=238333), 佐々木 睦子, 佐々木 建昭 (理化学研究所), 第23回プログラミング・シンポジウム報告集 (1982)
* [コンピュータに「絵」を描かせるには…](https://ipsj.ixsq.nii.ac.jp/ej/?action=repository_uri&item_id=238450), 斎藤 隆文 (東京大学), 第27回プログラミング・シンポジウム報告集 (1986)

音声・音楽
-----------

音声・音楽をあつかうような話題は、なぜか画像以上にたくさん出てきます。流行があったんでしょうか。

* [計算機による作曲の試み](https://ipsj.ixsq.nii.ac.jp/ej/?action=repository_uri&item_id=237722), 大槻 説乎 (名大計算室), 第4回プログラミング—シンポジウム報告集 (1963)
* [音声の合成](https://ipsj.ixsq.nii.ac.jp/ej/?action=repository_uri&item_id=223044), 中田 和男 (電波研究所), 第6回プログラミング—シンポジウム報告集 (1965)
* [汎用音楽記述言語の設計・試作](https://ipsj.ixsq.nii.ac.jp/ej/?action=repository_uri&item_id=238375), 笹川 瑠美, 三好 和憲, 五十嵐 滋 (筑波大学), 第25回プログラミング・シンポジウム報告集 (1984)
* [楽曲の特徴抽出](https://ipsj.ixsq.nii.ac.jp/ej/?action=repository_uri&item_id=238421), 野瀬 隆, 高田 正之 (東京農工大学), 第26回プログラミング・シンポジウム報告集 (1985)
* [自動演奏―演奏モデルとシミュレーション](https://ipsj.ixsq.nii.ac.jp/ej/?action=repository_uri&item_id=238487), 田口 友康, 藤原 儀直, 太田 雅久 (甲南大学), 第28回プログラミング・シンポジウム報告集 (1987)
* [共通楽譜データ形式の設計](https://ipsj.ixsq.nii.ac.jp/ej/?action=repository_uri&item_id=237449), 山崎 直子, 佐野 靖子, 渡邊 哲史, 高田 正之, 小谷 善行 (東京農工大学), 第29回プログラミング・シンポジウム報告集 (1988)
* [演奏表情の表現と重奏システムへの応用](https://ipsj.ixsq.nii.ac.jp/ej/?action=repository_uri&item_id=239075), 五十嵐 滋, 辻 尚史, 千葉 大春, 松下 昌弘, 小川 大典, 彌富 あかね, 清野 桂子 (筑波大学), 第36回プログラミング・シンポジウム報告集 (1995)
* [フラクタル音楽](https://ipsj.ixsq.nii.ac.jp/ej/?action=repository_uri&item_id=239138), 斎藤 隆文 (NTTヒューマンインタフェース研究所), 第37回プログラミング・シンポジウム報告書 (1996)
* [クラシック音楽CDのデータベースの設計](https://ipsj.ixsq.nii.ac.jp/ej/?action=repository_uri&item_id=239877), 中森 眞理雄, 福嶋 淳, 村松 英二 (東京農工大学), 第44回プログラミング・シンポジウム報告集 (2003)

未踏ソフトウェア
-----------------

未踏 (ソフトウェア創造) 事業 [^mitou] は 2000 年からの事業でしたが、そのプロジェクトもいくつかプロシンでの発表がされています。

[^mitou]: [未踏事業 (Wikipedia)](https://ja.wikipedia.org/wiki/%E6%9C%AA%E8%B8%8F%E4%BA%8B%E6%A5%AD)

* [未踏プロジェクト、現在じたばた真っ最中](https://ipsj.ixsq.nii.ac.jp/ej/?action=repository_uri&item_id=239812), 和田 健之介, 和田 佳子, 中口 孝雄, 星 和明 (有限会社アントラッド), 金子 勇 (エクス・ツールス株式会社), 石井 卓良 (デジタルファッション株式会社), 西尾 泰和 (京都大学), 第43回プログラミング・シンポジウム報告集 (2002)
* [ゲノム丸ごと可視化](https://ipsj.ixsq.nii.ac.jp/ej/?action=repository_uri&item_id=239872), 西尾 泰和, 比戸 将平 (京都大学), 和田 健之介, 和田 佳子 ((有)アントラッド), 池村 淑道 (国立遺伝学研究所), 金谷 重彦 (奈良先端大学院大学), 第44回プログラミング・シンポジウム報告集 (2003)

歴史
-----

本記事自体もそういう努力の一貫と呼べるかもしれませんが、日本や世界の計算機・コンピュータ・ソフトウェアの歴史を振り返り保存する努力に関連した話題も数年おきにされています。

* [昔の計算機たち－計算機技術博物館の構想－](https://ipsj.ixsq.nii.ac.jp/ej/?action=repository_uri&item_id=232218), 西村 恕彦 (東京農工大学), 第21回プログラミング・シンポジウム報告集 (1980)
* [日本の揺籃期のコンピュータのソフトウェア的復刻](https://ipsj.ixsq.nii.ac.jp/ej/?action=repository_uri&item_id=239127), 大駒 誠一 (慶應義塾大学), 第37回プログラミング・シンポジウム報告書 (1996)
* [招待講演「計算機の歴史の研究の現状」](https://ipsj.ixsq.nii.ac.jp/ej/?action=repository_uri&item_id=239649), 大駒 誠一 (慶應義塾大学), 第40回プログラミング・シンポジウム報告集 (1999)
* [コンピュータ博物館のための収集物と展示計画](https://ipsj.ixsq.nii.ac.jp/ej/?action=repository_uri&item_id=239809), 野瀬 隆, 小谷 善行, 西村 恕彦 (東京農工大学), 第43回プログラミング・シンポジウム報告集 (2002)

他
---

その他、「えっ」と目を引くようなタイトルの論文もしばしば…。

* [超時空プログラミングシステムUranus](https://ipsj.ixsq.nii.ac.jp/ej/?action=repository_uri&item_id=238404), 中島 秀之 (電子技術総合研究所), 第26回プログラミング・シンポジウム報告集 (1985)
* [日常語によるソフトウェア物理学](https://ipsj.ixsq.nii.ac.jp/ej/?action=repository_uri&item_id=238482), 日野 克重 (富士通株式会社), 第28回プログラミング・シンポジウム報告集 (1987)

分類が難しいものの、「おっ」と思わせるタイトルもあったり。

* [大きな数字を書く: ただそれだけのアルゴリズムと知られざる戦い](https://ipsj.ixsq.nii.ac.jp/ej/?action=repository_uri&item_id=237464), 高橋 俊成 (株式会社東芝総合研究所), 第30回プログラミング・シンポジウム報告集 (1989)
* [LAD (Lamda Diagrams): ラムダ式の図形表現](https://ipsj.ixsq.nii.ac.jp/ej/?action=repository_uri&item_id=237524), 二村 良彦, 野木 兼六, 高野 明彦 (日立製作所), 第31回プログラミング・シンポジウム報告集 (1990)

「あっ、これが」というお話もあったり。

* [第5世代コンピュータへのアプローチ](https://ipsj.ixsq.nii.ac.jp/ej/?action=repository_uri&item_id=238328), 渕 一博 (電子技術総合研究所), 第23回プログラミング・シンポジウム報告集 (1982)

もはやまとめきれませんが、本当にいろいろな話題があるのがプロシンです。

夏のプロシン
=============

この 1 月の (冬の) プロシン以外にも、その分科会 (?) 的なあつかいで、「夏のプログラミング・シンポジウム」 (通称: 夏のプロシン・夏プロ) というのが夏に開かれています。 (この記事の初版を公開した 2024 年 9 月 21 日も「[夏のプログラミング・シンポジウム 2024](https://prosym.org/sprosym2024/)」がおこなわれていました)

このはじまりは 1964 年 7 月 15 日から 2 泊 3 日で開かれた“夢のシンポジウム”だったそうです。この“夢のシンポジウム” 〜 夏のプロシンの「報告」も毎年の論文集に収録されており、その初回分から情報学広場に掲載となりました。

* [“夢のシンポジウム”第1回 ―電子計算機の将来展望シンポジウム―](https://ipsj.ixsq.nii.ac.jp/ej/?action=repository_uri&item_id=237809), 第7回プログラミング—シンポジウム報告集 (1966)

情報科学若手の会
=================

また、「情報科学若手の会」という催しも毎年おこなわれています。「プロシンには参加したことがないけど若手の会は行ったことある!」という方もいらっしゃるかもしれません。

情報科学若手の会も、実はプロシンの分科会的な位置付けだったりします。この若手の会の「報告」も毎年のプロシン論文集に収録されていて、情報学広場に掲載となりましたので、興味がある方はぜひ眺めてみてください。

* [第1回 情報科学「若手の会」](https://ipsj.ixsq.nii.ac.jp/ej/?action=repository_uri&item_id=237948), 第10回プログラミング—シンポジウム報告集 (1969)

ご参加をお待ちしています
=========================

いかがでしたか?

冒頭に書いたとおり、プログラミング・シンポジウムはいまも開催されており、ちょうど年明けの第 66 回に向けた発表・論文募集をしているところです。

@[card](https://prosym.org/66/cfp.html)

挙げてきた論文も大学関係者以外からのものが多かったことに、気が付かれた方も多いかもしれません。大学などアカデミアの方にかぎらず、「プログラミングが好きな」人たちと話してみたいネタがあるかた、ぜひ発表・参加を検討してみてください!
