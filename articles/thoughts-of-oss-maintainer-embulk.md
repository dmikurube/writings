---
title: "OSS ミドルウェアのメンテナは何を考えながらメンテナンスしてきたのか: Embulk のケース"
emoji: "🐬️" # アイキャッチとして使われる絵文字（1文字だけ）
type: "idea" # tech: 技術記事 / idea: アイデア
topics: [ "embulk", "OSS", "maintainer" ]
layout: default
published: false
---

* 依存ライブラリをうかつに更新できない問題 => 要らない依存を消し、必要なやつも embulk-deps へ
* Joda-Time
* 入り組んだ JRuby 依存 => JRuby 外部化 (オプショナル & バージョン選択をユーザー任せに)
* プラグインのバージョンとデプロイ
* 本体のどこか誰に使われているのかわからない => SPI 切り出し
* 無駄 DI
* 標準プラグインがコアと常にセット問題 =>
* リリースとライセンス問題 => JAR 内包
* システム全体の設定 (特に起動時) がしづらい問題 System Config => System Properties
