---
title: "60年前からの和文プログラミング論文を掘り起こした話"
emoji: "📜" # アイキャッチとして使われる絵文字（1文字だけ）
type: "idea" # tech: 技術記事 / idea: アイデア
topics: [ "prosym", "ipsj", "論文", "history", "computerscience" ]
layout: default
published: true
publication_name: "prosym"
---

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

情報処理学会として開かれていた会議の論文や会誌「情報処理」の記事などは、スキャンしたものが最初期のものから「[情報学広場](https://ipsj.ixsq.nii.ac.jp/ej/)」 (情報処理学会の電子図書館) に掲載されています。 [^jouhou-shori-vol-1] しかしプロシンの論文は、長い間、第 47 回以降のものしか掲載がありませんでした。 [^prosym-digital-library]

[^jouhou-shori-vol-1]: 例: [会誌「情報処理」 Vol.1 (1960)](https://ipsj.ixsq.nii.ac.jp/ej/index.php?action=pages_view_main&active_action=repository_view_main_item_snippet&index_id=576&pn=1&count=20&order=7&lang=japanese&page_id=13&block_id=8)
[^prosym-digital-library]: 過去の論文を掲載できていなかったのは、プロシンと情報処理学会では設立の過程が異なっていて運営も独立していたため、著作権処理などが情報処理学会のものとそろっていなかった、などが主な理由です。

関係者には「おもしろい議論がされている論文も多いのに読めないのはもったいない」こと、そして「せっかく書いた論文へのアクセスがかぎられることは著者にとっても本懐ではないはずだ」という思いがありました。そこで 2020 年、千葉大学の辻尚史先生に音頭を取っていただき、過去のプログラミング・シンポジウム論文集の内容を情報処理学会の情報学広場に掲載する活動が始まりました。 [^past-reports-copyrights]

[^past-reports-copyrights]: 著作権の利用許諾などについて、権利者捜索のためのご連絡が「[過去のプログラミング・シンポジウム報告集の利用許諾について](https://www.ipsj.or.jp/topics/Past_reports.html)」に掲載されています。

そこから掲載までの実作業にさらなる年月を要してしまったのですが、論文集の冊子をお持ちだった先生方からスキャンを提供いただき、さらに多くの方々のご協力を得て論文タイトル・著者の確認や PDF の分割などをおこない、第 46 回以前のプロシンの論文集もすべて情報学広場に掲載するめどがようやく立ちました。 [^as-of-2024-09-21]

[^as-of-2024-09-21]: 本記事を公開した 2024 年 9 月 21 日時点では第 42 回から第 46 回までがまだ抜けているのですが、作業はおおよそ終わっていますので、おそらく 9 月中には公開まで至るものと思います。 ([夏のプログラミング・シンポジウム](https://prosym.org/sprosym2024/)で共有したかったので、取り急ぎ本記事を公開しました)

本記事は、せっかく公開される論文集が多くの方の目に留まってほしいと、宣伝のために書いているものです。 [^aurhor-this-article]

[^aurhor-this-article]: 本記事の筆者は、データのチェックや PDF 分割の取りまとめと、最終確認の役をやっておりました。

学会系のイベントだからといって決してカタすぎることのない、ただただ「プログラミングが好きな」人たちが自由闊達にいろいろな (好き勝手な) 議論を楽しんでいる様子が目に浮かぶような、当時の息吹を感じられる記録になっていると思います。ぜひアクセスしてみてください!

* [情報学広場](https://ipsj.ixsq.nii.ac.jp/ej/)
  * ➤ [シンポジウム](https://ipsj.ixsq.nii.ac.jp/ej/index.php?action=pages_view_main&active_action=repository_view_main_item_snippet&index_id=6164&pn=1&count=20&order=7&lang=japanese&page_id=13&block_id=8)
    * ➤ [プログラミング・シンポジウム](https://ipsj.ixsq.nii.ac.jp/ej/index.php?action=pages_view_main&active_action=repository_view_main_item_snippet&index_id=6805&pn=1&count=20&order=7&lang=japanese&page_id=13&block_id=8)
      * ➤ [冬のプログラミング・シンポジウム](https://ipsj.ixsq.nii.ac.jp/ej/index.php?action=pages_view_main&active_action=repository_view_main_item_snippet&index_id=6807&pn=1&count=20&order=7&lang=japanese&page_id=13&block_id=8)

ピックアップ
=============

…と、概要だけ紹介して終わりにしてもいいのですが、これだけで興味を持ってもらえる人は、あまり多くないかもしれません。

ということで、ここからは、本記事の筆者が「おもしろい!」と思った論文を、あくまで筆者の独断と偏見で (ひとまず第 1 回 (1960) 〜第 20 回 (1979) のなかから) ピックアップして紹介してみたいと思います。 [^survey-all] 筆者の趣味が多分に反映されることはお許しください。 (カッコ () 内はシンポジウムの回数と開催年)

[^survey-all]: なにせ掲載作業を進める中で、ほぼすべてに軽く目は通しましたので…。

東京オリンピツク
-----------------

「東京オリンピツク」の話なんかが出てきたりもしています。つい数年前にあった東京 2020 じゃないですよ。その前の 1964 年の東京オリンピツクです。

* [東京オリンピツクに使われたリアルタイム・システムについて](https://ipsj.ixsq.nii.ac.jp/ej/?action=repository_uri&item_id=223027) (6; 1967)

数値計算・行列計算など
-----------------------

特に[第 1 回](https://ipsj.ixsq.nii.ac.jp/ej/index.php?action=pages_view_main&active_action=repository_view_main_item_snippet&index_id=11573&pn=1&count=20&order=7&lang=japanese&page_id=13&block_id=8)から[第 3 回](https://ipsj.ixsq.nii.ac.jp/ej/?action=repository_opensearch&index_id=11731&count=20&order=7&pn=1)あたりのリストを眺めてみると、「計算」の話題がよく出てきます。特に当時の大学や研究機関では、コンピュータが主に使われていたのは物理のシミュレーション計算であったり行列計算であったり、というのがよくわかります。

* [関数の関数をn回微分するためのサブルーチンとエルミット多項式Hk(x)のサブルーチン](https://ipsj.ixsq.nii.ac.jp/ej/?action=pages_view_main&active_action=repository_view_main_item_detail&item_id=232018&item_no=1&page_id=13&block_id=8) (1; 1960)
* [エラトステネスのふるいによる素数の計算について](https://ipsj.ixsq.nii.ac.jp/ej/?action=pages_view_main&active_action=repository_view_main_item_detail&item_id=232019&item_no=1&page_id=13&block_id=8) (1; 1960)
* [行列計算の自動プログラミング](https://ipsj.ixsq.nii.ac.jp/ej/?action=pages_view_main&active_action=repository_view_main_item_detail&item_id=232025&item_no=1&page_id=13&block_id=8) (1; 1960)
* [浮動小数点計算における絶対値最小の非零数について](https://ipsj.ixsq.nii.ac.jp/ej/?action=repository_uri&item_id=237636) (2; 1961)
* [モンテカルロ法による多重積分の計算](https://ipsj.ixsq.nii.ac.jp/ej/?action=repository_uri&item_id=237684) (3; 1962)

プログラミング言語・コンパイラ
-------------------------------

やはりプログラミング言語やコンパイラの話題は多いですね。

### ALGOL

当時まだ新しい ALGOL の文法の紹介なんて話も出てきたりします。 (注: 1960 年代当時、まだ C 言語すら存在していません!)

* [ALGOLの文法](https://ipsj.ixsq.nii.ac.jp/ej/?action=pages_view_main&active_action=repository_view_main_item_detail&item_id=232033&item_no=1&page_id=13&block_id=8) (1; 1960)
* [算法言語ALGOL 60の報告](https://ipsj.ixsq.nii.ac.jp/ej/?action=pages_view_main&active_action=repository_view_main_item_detail&item_id=237638&item_no=1&page_id=13&block_id=8) (2; 1961)
* [ALGOL 60のCompiler](https://ipsj.ixsq.nii.ac.jp/ej/?action=repository_uri&item_id=237669) (3; 1962)

### Self

Self っていう名前がついたコンパイラや言語の話がたまに出てくるんですが、有名な Smalltalk の影響を受けた Self 言語 [^self] のことではなかったりします。 (互いに関係もありません)

* [Self-Compiler](https://ipsj.ixsq.nii.ac.jp/ej/?action=repository_uri&item_id=237705) (4; 1963)
* [増殖型言語SELFについて](https://ipsj.ixsq.nii.ac.jp/ej/?action=repository_uri&item_id=232110) (12; 1972)

[^self]: [Self (Wikipedia)](https://ja.wikipedia.org/wiki/Self)

### PL/I

* [新しいプログラミング言語: PL/I (その出現までの経過，すぐれた特色，既存の言語との比較)](https://ipsj.ixsq.nii.ac.jp/ej/?action=repository_uri&item_id=237782) (7; 1966)

### FORTRAN

* [シミュレーション用言語について](https://ipsj.ixsq.nii.ac.jp/ej/?action=repository_uri&item_id=237746) (5; 1964)
* [数式解析のプログラムをFORTRANで書いた話](https://ipsj.ixsq.nii.ac.jp/ej/?action=repository_uri&item_id=232060) (9; 1968)
* [ALGOL-FORTRAN様のプログラムの妥当性の判定について](https://ipsj.ixsq.nii.ac.jp/ej/?action=repository_uri&item_id=232080) (9; 1968)

### 日本語プログラミング

* [日本語の語順と逆ポーランド記法](https://ipsj.ixsq.nii.ac.jp/ej/?action=repository_uri&item_id=237780) (7; 1966)
* [日本語によるCOBOL](https://ipsj.ixsq.nii.ac.jp/ej/?action=repository_uri&item_id=237842) (8; 1967)
* [記号処理のためのヤマト語](https://ipsj.ixsq.nii.ac.jp/ej/?action=repository_uri&item_id=237938) (10; 1969)

### プログラミング言語全般

"Problem Oriented Language" という言葉が出てくるんですが、これはいわゆる、今で言う「高級言語」のことを指しているのかな。

* [最近のProblem Oriented Languageについて](https://ipsj.ixsq.nii.ac.jp/ej/?action=repository_uri&item_id=237655) (2; 1961)

複数のプログラミング言語を俯瞰したような議論もしばしばあります。

* [プログラミング言語の動向と標準化](https://ipsj.ixsq.nii.ac.jp/ej/?action=repository_uri&item_id=237970) (11; 1970)
* [工業用計算機言語の開発と標準化の動向](https://ipsj.ixsq.nii.ac.jp/ej/?action=repository_uri&item_id=232150) (13; 1972)

### 自動プログラミング

「自動プログラミング」という言葉もちょくちょく出てきます。以前の「自動プログラミング」は「コンパイル・コンパイラ」と近い認識で解釈されたりもするのですが、もう少し「機械に近い」ところをゴリゴリやっていたような雰囲気が垣間見えたりします。

* [論理設計と自動プログラミング](https://ipsj.ixsq.nii.ac.jp/ej/?action=repository_uri&item_id=237668) (3; 1962)

### コンパイラとコンパイラ・コンパイラ

* [PROGRAM GENERATOR "PREMAP"](https://ipsj.ixsq.nii.ac.jp/ej/?action=repository_uri&item_id=237734) (5; 1964)
* [Compiler Generating System](https://ipsj.ixsq.nii.ac.jp/ej/?action=repository_uri&item_id=223035) (6; 1965)
* [計算機を用いたコンパイラ作成自動化の実験](https://ipsj.ixsq.nii.ac.jp/ej/?action=repository_uri&item_id=237796) (7; 1966)
* [Compiler Generatorへの入力データ](https://ipsj.ixsq.nii.ac.jp/ej/?action=repository_uri&item_id=232102) (12; 1971)
* [コンパイラ記述言語の一つの試み](https://ipsj.ixsq.nii.ac.jp/ej/?action=repository_uri&item_id=232113) (12; 1971)
* [順位文法によるプログラミング言語の解析とコンパイラの構成](https://ipsj.ixsq.nii.ac.jp/ej/?action=repository_uri&item_id=223405) (14; 1973)
* [文脈自由言語の誤り訂正時間](https://ipsj.ixsq.nii.ac.jp/ej/?action=repository_uri&item_id=223402) (14; 1973)
* [PASCALコンパイラのportabilityについて](https://ipsj.ixsq.nii.ac.jp/ej/?action=repository_uri&item_id=238029) (16; 1975)
* [字句解析部の高速化について－ソフトウェアの調整技法－](https://ipsj.ixsq.nii.ac.jp/ej/?action=repository_uri&item_id=232179) (20; 1979)

### リスト処理

「リスト処理」と言えば LISP をはじめとして、やはり一つの潮流であったようです。

* [木表現とリスト処理の算法](https://ipsj.ixsq.nii.ac.jp/ej/?action=repository_uri&item_id=237792) (7; 1966)
* [シンタックス・チェッカーとストレッジ・アロケータにおけるリスト・プロセッシング](https://ipsj.ixsq.nii.ac.jp/ej/?action=repository_uri&item_id=237793) (7; 1966)
* [リスト処理言語L6について](https://ipsj.ixsq.nii.ac.jp/ej/?action=repository_uri&item_id=237924) (10; 1969)
* [ページ・メモリーをもったリスト処理言語FLIP-III](https://ipsj.ixsq.nii.ac.jp/ej/?action=repository_uri&item_id=237940) (10; 1969)
* [LISPのデータ構造についての2, 3の考察](https://ipsj.ixsq.nii.ac.jp/ej/?action=repository_uri&item_id=223389) (14: 1973)
* [二人用STAR TREK ―リスト処理言語LIPXによるCo-Operating Processesの実験―](https://ipsj.ixsq.nii.ac.jp/ej/?action=repository_uri&item_id=238131) (19; 1978)

### 独自・多様なプログラミング言語

独自のプログラミング言語を作ったような話もよく出てきます。目的をはっきり絞ったようなものが多い印象ですね。

* [システム記述用言語PLDの使用経験と問題点](https://ipsj.ixsq.nii.ac.jp/ej/?action=repository_uri&item_id=232109) (12; 1971)
* [LADD: 描図用言語とそのデータ構造について](https://ipsj.ixsq.nii.ac.jp/ej/?action=repository_uri&item_id=232156) (13; 1972)
* [論理関係処理言語LORELのコンパイラの構成](https://ipsj.ixsq.nii.ac.jp/ej/?action=repository_uri&item_id=232159) (13; 1972)
* [汎用グラフィック言語UNGL](https://ipsj.ixsq.nii.ac.jp/ej/?action=repository_uri&item_id=223408) (14; 1973)
* [イオタ言語とイオタシステム ―階層的プログラム開発とその検証のための言語と方法とシステム―](https://ipsj.ixsq.nii.ac.jp/ej/?action=repository_uri&item_id=238143) (19; 1978)

プログラミング方法論
---------------------

### 構造化プログラミング

* [ストラクチャードプログラミング支援システムSPOTとその評価](https://ipsj.ixsq.nii.ac.jp/ej/?action=repository_uri&item_id=238026) (16; 1975)
* [構造化プログラミング例とデータ構造](https://ipsj.ixsq.nii.ac.jp/ej/?action=repository_uri&item_id=238027) (16; 1975)

データ処理・情報検索
---------------------

SQL もなかった時代に始まり COBOL などの文脈と合わせて「データ処理」というものに向き合おうとした様子が見えたりします。

* [データ処理理論の展望](https://ipsj.ixsq.nii.ac.jp/ej/?action=repository_uri&item_id=237698) (4; 1963)
* [AIS (Automated Information System) ―情報検索処理システム―](https://ipsj.ixsq.nii.ac.jp/ej/?action=repository_uri&item_id=237930) (10; 1969)
* [情報検索システムのリアルタイムシミュレーション](https://ipsj.ixsq.nii.ac.jp/ej/?action=repository_uri&item_id=237987) (11; 1970)
* [外務省における情報検索システム](https://ipsj.ixsq.nii.ac.jp/ej/?action=repository_uri&item_id=237783) (7; 1966)
    * 外務省!?
* [データ記述言語とCOBOLのデータベース機能](https://ipsj.ixsq.nii.ac.jp/ej/?action=repository_uri&item_id=223380) (14; 1973)
* [データベータに対するGUIDE/SHAREの要望](https://ipsj.ixsq.nii.ac.jp/ej/?action=repository_uri&item_id=223381) (14: 1973)
* [情報検索とSL―導出定理証明プログラム](https://ipsj.ixsq.nii.ac.jp/ej/?action=repository_uri&item_id=223424) (15; 1974)
* [高水準データベースシステムのための分析と評価](https://ipsj.ixsq.nii.ac.jp/ej/?action=repository_uri&item_id=238095) (18; 1977)
* [関係データベース上の高度な質問応答過程について](https://ipsj.ixsq.nii.ac.jp/ej/?action=repository_uri&item_id=238144) (19; 1978)

算数問題を解く
---------------

かと思えば、いわゆる算数の文章問題的なものを解こうとする、東ロボくん的なチャレンジの話が当時からあったり。

* [算術の問題を解くプログラム](https://ipsj.ixsq.nii.ac.jp/ej/?action=repository_uri&item_id=237649) (2; 1961)
* [算術を解くプログラム](https://ipsj.ixsq.nii.ac.jp/ej/?action=repository_uri&item_id=237689) (3; 1962)

証明
-----

プログラムで証明をする、みたいな話もかなり以前からあったようです。

* [証明のプログラミング](https://ipsj.ixsq.nii.ac.jp/ej/?action=repository_uri&item_id=237720) (4; 1963)
* [数学の証明のプログラム](https://ipsj.ixsq.nii.ac.jp/ej/?action=repository_uri&item_id=237749) (5; 1964)

産業・他業界
-------------

情報系の学術関係だけではなく、けっこう実産業界や他業界からの発表もあるのが (いまでも) プロシンの特色です。

初期には、今から見た印象としては「え、ここから?」となる企業からの発表もしばしばありました。

* [鉄鋼プラントにおけるプロセス用計算機の利用](https://ipsj.ixsq.nii.ac.jp/ej/?action=repository_uri&item_id=223032) (6; 1965)
    * 日本鋼管(株) 川崎製鉄所
* [晶析工程シミュレータ](https://ipsj.ixsq.nii.ac.jp/ej/?action=repository_uri&item_id=237787) (7; 1966)
    * 味の素
* [プラント設計とシミュレーション](https://ipsj.ixsq.nii.ac.jp/ej/?action=repository_uri&item_id=232135) (13; 1972)
    * 東洋エンジニアリング
* [COMによるアニメーション ―地震応答シミュレーション結果の表示―](https://ipsj.ixsq.nii.ac.jp/ej/?action=repository_uri&item_id=232148) (13; 1972)
    * 鹿島建設
* [放射線測定のためのコンピュータ・ネットワーク](https://ipsj.ixsq.nii.ac.jp/ej/?action=repository_uri&item_id=238049)
    * 日本原子力研究所
* [企業名のカナ漢字変換システム](https://ipsj.ixsq.nii.ac.jp/ej/?action=repository_uri&item_id=238096)
    * 日本ユニバック株式会社
* [天文観測整約用言語「アカイホシ」](https://ipsj.ixsq.nii.ac.jp/ej/?action=repository_uri&item_id=238104)

鉄道
-----

界隈に鉄の人が多いのも昔からだったようです。

* [国鉄座席予約システムMARS-101](https://ipsj.ixsq.nii.ac.jp/ej/?action=repository_uri&item_id=223029) (6)

システム・OS
-------------

* [慶応義塾大学 実験用タイムシェアリング・システム](https://ipsj.ixsq.nii.ac.jp/ej/?action=repository_uri&item_id=237844) (8; 1967)
    * タイムシェアリング・システム!
* [IBM 2250映像表示装置とそのプログラミング・サポート](https://ipsj.ixsq.nii.ac.jp/ej/?action=repository_uri&item_id=232065) (9; 1968)
* [OSの理論](https://ipsj.ixsq.nii.ac.jp/ej/?action=repository_uri&item_id=223430) (15; 1974)
* [システムプログラム作成用ミニ言語](https://ipsj.ixsq.nii.ac.jp/ej/?action=repository_uri&item_id=238034) (16; 1975)
* [Solo Operating Systemのimplementationとその使用経験](https://ipsj.ixsq.nii.ac.jp/ej/?action=repository_uri&item_id=238089) (17; 1977)
* [使いやすさを狙ったTSS用OS―UNIXについて](https://ipsj.ixsq.nii.ac.jp/ej/?action=repository_uri&item_id=238132) (19; 1978)
    * ついに UNIX が!!!

メディア
---------

* [計算機による作曲の試み](https://ipsj.ixsq.nii.ac.jp/ej/?action=repository_uri&item_id=237722) (4; 1963)
* [コンピュータによる色彩画](https://ipsj.ixsq.nii.ac.jp/ej/?action=repository_uri&item_id=237926) (10; 1969)

言語・画像・音声・認識・人工知能
---------------------------------

生成 AI 全盛の 2020 年代ですが、自然言語処理、画像処理、音声処理、みたいな話もずっとありますね。

* [神経のモデルの数値計算](https://ipsj.ixsq.nii.ac.jp/ej/?action=repository_uri&item_id=237687) (3; 1962)
* [計算機による翻訳](https://ipsj.ixsq.nii.ac.jp/ej/?action=repository_uri&item_id=223042) (6; 1965)
* [音声の合成](https://ipsj.ixsq.nii.ac.jp/ej/?action=repository_uri&item_id=223044) (6; 1965)
* [仮名文字パタン認識の一考察](https://ipsj.ixsq.nii.ac.jp/ej/?action=repository_uri&item_id=237778) (7; 1966)
* [活字読取の実験とテレビカメラを利用した入力装置](https://ipsj.ixsq.nii.ac.jp/ej/?action=repository_uri&item_id=237960) (11; 1970)
* [機械による学習と認識](https://ipsj.ixsq.nii.ac.jp/ej/?action=repository_uri&item_id=237988) (11; 1970)
* [音声識別の一方式](https://ipsj.ixsq.nii.ac.jp/ej/?action=repository_uri&item_id=232121) (12; 1971)
* [人工知能のプログラミング言語](https://ipsj.ixsq.nii.ac.jp/ej/?action=repository_uri&item_id=223420) (15; 1974)
* [コンピュータ断層撮影装置の現状と将来](https://ipsj.ixsq.nii.ac.jp/ej/?action=repository_uri&item_id=232181) (20; 1979)

計算センター・データセンター
-----------------------------

大学や企業における計算機センター、データセンター、みたいな話題もあります。

* [企業における計算センターの運営について](https://ipsj.ixsq.nii.ac.jp/ej/?action=repository_uri&item_id=237759) (5; 1964)
* [計算センターの運営について](https://ipsj.ixsq.nii.ac.jp/ej/?action=repository_uri&item_id=237760) (5; 1964)
* [九州工業大学情報処理教育センターのシステム開発・運用状況について](https://ipsj.ixsq.nii.ac.jp/ej/?action=repository_uri&item_id=238118) (18; 1977)

組版
-----

ちなみに TeX は 1984 年だそうです。

* [汎用自動組み版システムの一例](https://ipsj.ixsq.nii.ac.jp/ej/?action=repository_uri&item_id=223397) (14; 1973)

教育
-----

プログラミングの教育の話がちょくちょくあるのも、初期からのようです。

* [日本の大学における計算機教育について](https://ipsj.ixsq.nii.ac.jp/ej/?action=repository_uri&item_id=237789) (7; 1966)
* [高校における電子計算機教育について](https://ipsj.ixsq.nii.ac.jp/ej/?action=repository_uri&item_id=237832) (8; 1967)
* [プログラミング学習システムCLASS](https://ipsj.ixsq.nii.ac.jp/ej/?action=repository_uri&item_id=223377) (14; 1973)

ゲーム・パズル
---------------

プロシンには [GPCC (Games and Puzzles Competitions on Computers)](https://hp.vector.co.jp/authors/VA003988/gpcc/gpcc.htm) という分科会もあるように、ゲームやパズルを解く、という話題もよくあります。

* [計算機によるゲームとパズル](https://ipsj.ixsq.nii.ac.jp/ej/?action=repository_uri&item_id=238025) (16; 1975)
* [将棋ゲームの指し手の理解モデルとオンライン処理について](https://ipsj.ixsq.nii.ac.jp/ej/?action=repository_uri&item_id=238110) (18; 1977)

暗号・セキュリティ
-------------------

* [公衆暗号系の構想 ―暗号とNP完全問題―](https://ipsj.ixsq.nii.ac.jp/ej/?action=repository_uri&item_id=238140) (19; 1978)

夏のプロシン
=============

1 月の冬のプロシンの分科会 (?) 的なあつかいで、夏に「夏のプログラミング・シンポジウム」 (通称: 夏のプロシン・夏プロ) というのが開かれています。 (この記事を公開した 2024 年 9 月 21 日も「[夏のプログラミング・シンポジウム 2024](https://prosym.org/sprosym2024/)」がおこなわれている最中です)

このはじまりは 1964 年 7 月 15 日から 2 泊 3 日で開かれた“夢のシンポジウム”だったそうです。この“夢のシンポジウム”〜夏のプロシンの「報告」も毎年の論文集に掲載されており、初回から情報学広場に掲載することができました。

* [“夢のシンポジウム”第1回 ―電子計算機の将来展望シンポジウム―](https://ipsj.ixsq.nii.ac.jp/ej/?action=repository_uri&item_id=237809) (7; 1966)

情報科学若手の会
=================

また、「情報科学若手の会」という催しも毎年おこなわれています。プロシンには参加したことがないけど若手の会は行ったことある! という方もいらっしゃるかもしれません。

情報科学若手の会も、実はプロシンの分科会的な感じで開かれています。この「報告」も毎年のプロシン論文集に収録されていて、報告も情報学広場に掲載できましたので、興味がある方はぜひ眺めてみてください。

* [第1回 情報科学「若手の会」](https://ipsj.ixsq.nii.ac.jp/ej/?action=repository_uri&item_id=237948) (10; 1969)

ご参加をお待ちしています
=========================

いかがでしたか? (TM)

冒頭に書いたとおり、プログラミング・シンポジウムはいまも開催されており、ちょうど年明けの第 66 回に向けた発表・論文募集をしているところです。

@[card](https://prosym.org/66/cfp.html)

ただただ「プログラミングが好きな」老若男女と話してみたいネタがあるかた、ぜひ発表・参加を検討してみてください!
