---
title: "プロシンのシンボルマーク"
emoji: "♨" # アイキャッチとして使われる絵文字（1文字だけ）
type: "tech" # tech: 技術記事 / idea: アイデア
topics: [ "prosym" ]
layout: default
published: false
---

プログラミング・シンポジウムのシンボルマーク (ロゴ) は 1998年の第39回プログラミング・シンポジウムから、以下のようなものになっています。

> 前回のシンポジウムの最後に，木村泉さんから注文があった．シンポジウムのシンボルマークを更新したらどうか，というのである．今のマークは木村さんが幹事をつとめた夏のシンポジウム「構造的プログラミング」 のマークとして決めたものである． Nassi-Shneiderman 図の中に **S** と **P** が書いてあった (**Structured Programming** の頭文字)．翌年の冬のシンポジウムから **P** と **S** に変えて (**Programming Symposium** の頭文字) 使っているが，古びた感じは否めない．木村さんはアンケート用紙の裏に，スケッチしてくれた．特に **S** の中央の曲線が浴衣の帯のずり下がっているように，との希望であった．
>
> 今回から新しいマークである (図 0)．どのようにして書いたかを説明しよう． PostScript に curveto という 6 引数のオペレータがある． 図 1 に示すように，現在のポイントが $(x_0, y_0)$ にあったとして， $x_2$ $y_2$ $x_3$ $y_3$ $x_1$ $y_1$ $curveto$ は $(x_0, y_0)$ から $(x_1, y_1)$ まで， $(x_2, y_2), (x_3, y_3)$ を制御点とした B&eacute;zier 曲線を描く ものである. 制御点 $x_2$ と $y_3$ はそれぞれ $x_0, x_1$ と $y_1, y_0$ を $r:1-r$ に内分する点とする. $r$ を $0.2$, $0.4$, $0.6$, $0.8$ と変えた時の曲線の感じは図2 に示す通り. $r=0.6$ の曲線を図3 の座標に従って描いたのが，新しいマークである.  **P** と **S** の向かい合う部分の曲率を同じにしてほしいという木村さんの要求は満たしていると思う.
>
> 和田 英一 「第39回プログラミング・シンポジウム開催に際して」 第39回プログラミング・シンポジウム報告集 (1998年1月)

```
%! programming symposium icon
/r0 0.6 def /r1 1 r0 sub def
/hcurve
{/y1 exch def /x1 exch def currentpoint /y0 exch def /x0 exch def
r1 x0 mul r0 x1 mul add y0 x1 r0 y0 mul r1 y1 mul add x1 y1 curveto
} def
/vcurve
{/y1 exch def /x1 exch def currentpoint /y0 exch def /x0 exch def
x0 r1 y0 mul r0 y1 mul add r0 x0 mul r1 x1 mul add y1 x1 y1 curveto
} def

580 400 translate 90 rotate 0.9 0.9 scale
20 setlinewidth 1 setlinejoin 1 setlinecap

-310 50 moveto -310 230 lineto -180 230 lineto
-50 180 hcurve -180 120 vcurve -310 120 lineto stroke

310 230 moveto 180 230 lineto 50 180 hcurve 180 120 vcurve
310 80 hcurve 180 50 vcurve 50 50 lineto stroke

0.75 setgray
-360 280 moveto 360 280 lineto 360 600 lineto 0 280 lineto -360 600 lineto
-360 280 lineto fill

0 setgray 2 setlinewidth
0 280 moveto 0 0 lineto 360 0 lineto 360 600 lineto -360 600 lineto
-360 0 lineto 0 0 lineto stroke

showpage
```



全部 y x (something) の形ですね、という前提で探していくと

min X = 0
min Y = -360
max X = 600
max Y = 360

前後左右に 20 ポイントずつ幅をとって、とりあえず全体を -20 -380 620 380 ということにしましょう。

width = 640 height = 760 なので SVG の viewBox に指定するのは -20 -380 640 760


-310 50 moveto -310 230 lineto -180 230 lineto
-50 180 hcurve -180 120 vcurve -310 120 lineto stroke

x0 := 50
y0 := -310

line(x0, y0, 230, -310)
x0 := 230
y0 := -310

line(x0, y0, 230, -180)
x0 := 230
y0 := -180

hcurve(x0, y0, 180, -50)
x0 := 180
y0 := -50

vcurve(x0, y0, 120, -180)
x0 := 120
y0 := 180

line(x0, y0, 120, -310)
x0 := -120
y0 := -310

stroke


310 230 moveto 180 230 lineto 50 180 hcurve 180 120 vcurve
310 80 hcurve 180 50 vcurve 50 50 lineto stroke

x0 := 230
y0 := 310

line(x0, y0, 230, 180)
x0 := 230
y0 := 180

hcurve(x0, y0, 180, 50)
x0 := 180
y0 := 50

vcurve(x0, y0, 120, 180)
x0 := 120
y0 := 180

hcurve(x0, y0, 80, 310)
x0 := 80
y0 := 310

vcurve(x0, y0, 50, 180)
x0 := 50
y0 := 180

line(50, 180, 50, 50)




0.75 setgray
-360 280 moveto 360 280 lineto 360 600 lineto 0 280 lineto -360 600 lineto
-360 280 lineto fill

<polygon points="280,-360,280,360,600,360,280,0,600,-360,280,-360" fill="#B0B0B0" stroke="#B0B0B0"/>




0 setgray 2 setlinewidth

color = #000000
// 2 setlinewidth : 線幅を2ポイントに

0 280 moveto 0 0 lineto 360 0 lineto 360 600 lineto -360 600 lineto
-360 0 lineto 0 0 lineto stroke





/r0 0.6 def /r1 1 r0 sub def

r0 := 0.6
r1 := 1 - r0 = 0.4

/hcurve
{/y1 exch def /x1 exch def currentpoint /y0 exch def /x0 exch def
r1 x0 mul r0 x1 mul add y0 x1 r0 y0 mul r1 y1 mul add x1 y1 curveto
} def

def hcurve(x, y) = {
  y1 = y
  x1 = x
  y0 = y0
  x0 = x0

  _1 = r1 * x0
  _2 = r0 * x1
  _3 = _1 + _2 => push
  y0 => push
  x1 => push
  _4 = r0 * y0
  _5 = r1 * y1
  _6 = _4 + _5 => push
  x1 => push
  y1 => push

  ((r1 * x0) + (r0 * x1)) y0 x1 ((r0 * y0) + (r1 * y1)) x1 y1 curveto
}

/vcurve
{/y1 exch def /x1 exch def currentpoint /y0 exch def /x0 exch def
x0 r1 y0 mul r0 y1 mul add r0 x0 mul r1 x1 mul add y1 x1 y1 curveto
} def

def vcurve(x, y) = {
  y1 = y
  x1 = x
  y0 = 0
  x0 = 0

  x0 => push
  _1 = r1 * y0
  _2 = r0 * y1
  _3 = _1 + _2 => push

  _4 = r0 * x0
  _5 = r1 * x1
  _6 = _4 + _5 => push
  y1 => push
  x1 => push
  y1 => push

  x0 ((r1 * y0) + (r0 * y1)) ((r0 * x0) + (r1 * x2)) y1 x1 y1 curveto
}



hcurve(230, -180, 180, -50)

x0 y0 ((r1 * x0) + (r0 * x1)) y0 x1 ((r0 * y0) + (r1 * y1)) x1 y1 curveto

((r1 * x0) + (r0 * x1)) = (0.4 * 230) + (0.6 * 180) = 200
((r0 * y0) + (r1 * y1)) = (0.6 * -180) + (0.4 * -50) = -128

200, -180, 180, -128, 180, -50






/R0 0.6 def /r1 1 r0 sub def

r0 := 0.6
r1 := 1 - r0 = 0.4

/hcurve
{/y1 exch def /x1 exch def currentpoint /y0 exch def /x0 exch def
r1 x0 mul r0 x1 mul add y0 x1 r0 y0 mul r1 y1 mul add x1 y1 curveto
} def

def hcurve(x, y) = {
  y1 = y
  x1 = x
  y0 = y0
  x0 = x0

  _1 = r1 * x0
  _2 = r0 * x1
  _3 = _1 + _2 => push
  y0 => push
  x1 => push
  _4 = r0 * y0
  _5 = r1 * y1
  _6 = _4 + _5 => push
  x1 => push
  y1 => push

  ((r1 * x0) + (r0 * x1)) y0 x1 ((r0 * y0) + (r1 * y1)) x1 y1 curveto
}

/vcurve
{/y1 exch def /x1 exch def currentpoint /y0 exch def /x0 exch def
x0 r1 y0 mul r0 y1 mul add r0 x0 mul r1 x1 mul add y1 x1 y1 curveto
} def

def vcurve(x, y) = {
  y1 = y
  x1 = x
  y0 = 0
  x0 = 0

  x0 => push
  _1 = r1 * y0
  _2 = r0 * y1
  _3 = _1 + _2 => push

  _4 = r0 * x0
  _5 = r1 * x1
  _6 = _4 + _5 => push
  y1 => push
  x1 => push
  y1 => push

  x0 ((r1 * y0) + (r0 * y1)) ((r0 * x0) + (r1 * x2)) y1 x1 y1 curveto
}

580 400 translate 90 rotate 0.9 0.9 scale

// 原点の移動
screen_x := 400
screen_y := 580

// 90 rotate : 90度回転
// 0.9 0.9 scale : スケール

20 setlinewidth 1 setlinejoin 1 setlinecap

// 20 setlinewidth : 線幅を20ポイントに

2 本の線が連結されているところの形状のことを「ラインジョイン」(line join) と言います。ラ
インジョインには、マイター (miter)、ラウンド (round)、ベベル (bevel) という 3 種類のものが
あります。ラインジョインを変更したいときは、setlinejoin というオペレーターを使います。
あらかじめ、0、1、2、のいずれかの整数をスタックにプッシュしておいてから setlinecap を
8 PostScript 実習マニュアル
実行すると、ラインジョインは、整数が 0 ならばマイター、1 ならばラウンド、2 ならばベベル
に変更されます。ラインジョインのデフォルトは、0 のマイターです。
http://www.umekkii.jp/college/syllabus/06_report/citation/postscript.pdf

// 1 setlinejoin : line join (2本の線が連結されているところの形状) を "round" に

線が開始したり終了したりするところの形状のことを「ラインキャップ」(line cap) と言います。
ラインキャップには、バット (batt)、ラウンド (round)、プロジェクティングスクエア (projecting
square) という 3 種類のものがあります。ラインキャップを変更したいときは、 setlinecap と
いうオペレーターを使います。あらかじめ、0、1、2、のいずれかの整数をスタックにプッシュ
しておいてから setlinecap を実行すると、ラインキャップは、整数が 0 ならばバット、1 なら
ばラウンド、2 ならばプロジェクティングスクエアに変更されます。ラインキャップのデフォル
トは、0 のバットです。
http://www.umekkii.jp/college/syllabus/06_report/citation/postscript.pdf

// 1 setlinecap : line cap (線が開始したり終了したりするところの形状) を "round" に



0.75 setgray


color = #B0B0B0 (0 = black / 1 = white)

-360 280 moveto 360 280 lineto 360 600 lineto 0 280 lineto -360 600 lineto
-360 280 lineto fill



0 setgray 2 setlinewidth

color = #000000
// 2 setlinewidth : 線幅を2ポイントに

0 280 moveto 0 0 lineto 360 0 lineto 360 600 lineto -360 600 lineto
-360 0 lineto 0 0 lineto stroke




/hcurve
{/y1 exch def /x1 exch def currentpoint /y0 exch def /x0 exch def
r1 x0 mul r0 x1 mul add y0 x1 r0 y0 mul r1 y1 mul add x1 y1 curveto
} def

(defun hcurve (x0 y0 x1 y1 r0 r1) (list (+ (* r1 x0) (* r0 x1)) y0 x1 (+ (* r0 y0) (* r1 y1)) x1 y1 'curveto))
hcurve


(hcurve -180 230 -50 180 0.6 0.4)
(-102.0 230 -50 210.0 -50 180 curveto)


/vcurve
{/y1 exch def /x1 exch def currentpoint /y0 exch def /x0 exch def
x0 r1 y0 mul r0 y1 mul add r0 x0 mul r1 x1 mul add y1 x1 y1 curveto
} def

(defun vcurve (x0 y0 x1 y1 r0 r1) (list x0 (+ (* r1 y0) (* r0 y1)) (+ (* r0 x0) (* r1 x1)) y1 x1 y1 'curveto))
vcurve


-50 180 hcurve -180 120 vcurve

(vcurve -50 180 -180 120 0.6 0.4)
(-50 144.0 -102.0 120 -180 120 curveto)




310 230 moveto 180 230 lineto 50 180 hcurve 180 120 vcurve
310 80 hcurve 180 50 vcurve 50 50 lineto stroke



180 230 lineto 50 180 hcurve

(hcurve 180 230 50 180 0.6 0.4)
(102.0 230 50 210.0 50 180 curveto)
102 230 50 210 50 180



50 180 hcurve 180 120 vcurve

(vcurve 50 180 180 120 0.6 0.4)
(50 144.0 102.0 120 180 120 curveto)
50 144 102 120 180 120


180 120 vcurve 310 80 hcurve

(hcurve 180 120 310 80 0.6 0.4)
(258.0 120 310 104.0 310 80 curveto)
258 120 310 104 310 80


310 80 hcurve 180 50 vcurve

(vcurve 310 80 180 50 0.6 0.4)
(310 62.0 258.0 50 180 50 curveto)
310 62 258 50 180 50
