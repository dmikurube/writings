---
title: "require と import と import と #include のはなし"
emoji: "📦" # アイキャッチとして使われる絵文字（1文字だけ）
type: "tech" # tech: 技術記事 / idea: アイデア
topics: [ "ruby", "require", "java", "import" ]
layout: default
published: false
---

書いているプログラムの中で、なにかライブラリの機能を使おうと思ったら、なにをしますか?

まず Bundler (`Gemfile`) なり Maven (`pom.xml`) なり Gradle (`build.gradle`) なりに、そのライブラリを依存関係として追加する? そうですね。まずはそのライブラリを依存関係に入れないと始まりません。

では、その次は?

Ruby なら、どこかに `require` を追加することになるでしょう。 Java なら `import` でしょうか。 Python でも同じく `import` ですね。 C や C++ なら `#include` あたりを追加することになるでしょう。ライブラリのクラスや関数などを新しく使おうと思ったときに出てくるのが、各言語が持つ、このような記述です。

このような記述は、なんだかどれも似たような機能に見えますよね。なので、たとえば Ruby 使いに Java の `import` を説明しようとして、「`Ruby` の `require` みたいなもんだよ」なんて説明をしてしまいがちです。

流行りの [ChatGPT](https://chat.openai.com/) さん (2023-01-17 時点; [ChatGPT Jan 9 Version](https://help.openai.com/en/articles/6825453-chatgpt-release-notes)) にも聞いてみましたが、だいたい同じような説明が返ってきました。 Ruby は「外部のモジュールやライブラリを」読み込み、そして Java は「クラスやインターフェイス」を読み込む、というのが違いだそうです。

> 私:
>
> Ruby の require と Java の import の違いを説明してください

> ChatGPT:
>
> Ruby の require と Java の import は、それぞれ言語のモジュールやライブラリをプログラムに読み込むために使用されます。
>
> Ruby の require は、外部のモジュールやライブラリを読み込み、それらをプログラムに組み込みます。require は、複数回呼び出されても、同じファイルは１度だけ読み込まれます。
>
> Java の import は、クラスやインターフェイスを読み込むために使用されます。import を使用すると、特定のパッケージ内のクラスやインターフェイスを使用することができます。import は、複数回呼び出されても、同じクラスやインターフェイスは１度だけ読み込まれます。
>
> 一般的に、Ruby の require は、モジュールやライブラリを読み込むために使用し、Java の import は、クラスやインターフェイスを読み込むために使用します。

実際、なんとなくこのような理解で `require` や `import` を使っている方も多いかと思います。ですがその理解、本当に正しいでしょうか?

この記事では、各言語のこのような記述の裏で、実際にはなにがどう動いているのか、深堀りしてみます。さらにそこから、それぞれのプログラミング言語、ビルドシステム、パッケージ管理システムなどが、どうしてそんな設計になっているのか、議論してみたいと思います。

`require` (Ruby)
=================


`import` (Java)
================


`import` (Python)
==================


`#include` (C/C++)
===================
