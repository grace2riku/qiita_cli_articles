---
title: 書籍【Elixir実践ガイド】の写経の感想
tags:
  - Elixir
private: false
updated_at: '2023-12-16T22:14:50+09:00'
id: 168ba1300a9b7929d484
organization_url_name: persolcrosstechnology
slide: false
ignorePublish: false
---
この記事は[Elixir Advent Calendar 2023](https://qiita.com/advent-calendar/2023/elixir)のシリーズ3の18日目です。

# 概要
去る2023/5/17に **【ElixirImp#31：Elixirで作ったもの、何でもLT会】** で **書籍:【Elixir実践ガイド】の写経の感想**というタイトルでLTしました。

https://fukuokaex.connpass.com/event/283391/

この記事ではLTで発表した内容を紹介、説明します。

LTの資料はこちらです。

<script async class="docswell-embed" src="https://www.docswell.com/assets/libs/docswell-embed/docswell-embed.min.js" data-src="https://www.docswell.com/slide/5GXLMD/embed" data-aspect="0.75"></script><div class="docswell-link"><a href="https://www.docswell.com/s/juraruming/5GXLMD-2023-05-17-124330">ElixirImp#31_書籍【Elixir実践ガイド】の写経の感想_20230517 by @juraruming</a></div>

LTの動画はこちらです。ご興味ありましたら見てください。

<iframe width="560" height="315" src="https://www.youtube.com/embed/8oKe4G6XTRo?si=8vcMRw_OxQnIj1qO" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" allowfullscreen></iframe>

# 写経の動機
2022年8月、ElixirとNervesのハンズオン(※)に参加し、組込みシステムでモダンに開発できることを体験し、感動しました。
ElixirとNervesのことをより深く知りたいと考え、とりあえずElixirの書籍の写経を始めてみました。

※日本ソフトウェア科学会第39回大会チュートリアル「関数型言語Elixirで始めるIoTシステム開発入門」

GitHubリポジトリはこちらです。

https://github.com/b5g-ex/jssst2022_tutorial


# 写経した本の紹介
写経した本は**Elixir実践ガイド**です。

https://book.impress.co.jp/books/1120101021

目次はつぎのとおりです。

![book_picture.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/171866/be6e38a3-2d63-dedd-1a6e-6f257b0717f2.png)

## 本の選定理由
Elixirのdiscordコミュニティ（elixirと見習い錬金術師）に相談したらこの本を紹介いただいたからです。
[プログラミングElixir（第2版）](https://www.ohmsha.co.jp/book/9784274226373/)と迷いましたがElixir実践ガイドよりも難易度が高い、というご意見があったのでElixir実践ガイドに決めました。

# 写経する前の状況
Elixir経験はWeb記事の写経、前述のハンズオンで体験した、という状況でした。
私の技術バックグラウンドは組込みソフトウェア関連の仕事をしており、プログラミング言語はCが一番得意としています。

# 写経の方法
## 写経のスタイル
TDD伝導師・t_wadaさんの写経スタイルを真似ました。

* 意味のあるコードを1日1回でも書く
* 変更履歴管理ツールにコミットログ(本の章番号・不明点)を書く
* 座れる駅(始発駅)に住み、通勤中にコードを書く環境をつくる。
→コロナ以降、時差出勤（1h早く）で通勤電車で座れるようになり写経可能になった!!!

## 写経のスタイル2
・Elixir実践ガイドの章末の練習問題も欠かさず写経&理解するようにしました。
・学習したコードはどのようなシーンで使えるかイメージし、写経のときに感じたことなどもコミットログに追記しました。

## 写経の期間
私の場合の写経の期間についてです。

* 写経は2023/2/24から開始
* 2023/5/17時点で25章まで完了し、全体の8割くらい終了
* 写経は2023/6/9に完了

# 写経の感想
本でElixirのコードを写経した感想です。

## モダンな開発環境

* 対話型のシェル環境(IEx)ですぐに動作確認できる!!!
* テストが書ける!!テストで言語自体を学習できる。
* Mixでプロジェクトの雛形をサクっと作れる!!!
* 日本のコミュニティが活発!!!
平時の活動、年末のQiitaアドベントカレンダーの記事投稿数が凄い・・・。

## Elixirの気持ちよさを体感!!!
つぎの機能でElixirの気持ちよさを実体験でき、感動しました。

・パイプ
・パターンマッチ
・ビットストリング・バイナリ

下図はElixirImp Connpassイベントページの引用です。
赤枠のことが写経をとおして実感できました👍

![elixir_good_point.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/171866/e3a43790-7bf1-54fe-6c36-a2d9b96ef058.png)

### ビットストリング・バイナリ
* ビットストリング：ビットの列を表すデータ構造
* バイナリ：ビットストリングのうち、長さが8の倍数

組込みソフトウェアの仕事に関わり長いので組込みソフトウェアで使える!!!、と思いました。
組込みソフトウェアのハードウェア制御(レジスタアクセス、通信、etc)はこのデータ構造を多用していると感じます。

例）
* データ読み出し

```elixir
{:ok, read_data} = File.read(“binary_data.bin”)
```

* 先頭0byte目, 先頭1〜3byte目, 残りを取り出す

```elixir
<<stx:binary-size(1), size::binary-size(3), rest::bits>> = read_data
```

こんなに簡単に必要なデータを取り出せる!!!
より詳しい説明はこちらのQiita記事を参照してみてください。

> 組込みに欠かせない Elixir でのビットの扱い方

https://qiita.com/kikuyuta/items/e200a6208013f38333de

## Elixirらしいプログラミングをなんとなく理解!!!
Elixir実践ガイドが随所で【Elixirらしい書き方】を丁寧に記載してくれています。
【Elixirらしい】を言語化できませんが、Elixir経験者のコードをなんとなく理解することができました。


# 今後やってみたいこと
写経を実施したあとにElixirで今後やってみたい・体感したいと感じたことです。

ElixirImp Connpassイベントページより引用です。
下図の赤枠を今後やってみたいと思いました。

![elixir_want.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/171866/de23ab73-bcd9-3e3b-f83f-1fdc586c5038.png)

## マルチコアCPUを簡単にフル活用できる
Elixirの特徴で謳われていることかと思いますがマルチコアCPUを簡単にフル活用するコードを書いて、体感としてマルチコアCPUをフル活用している実感を得たことがないので是非とも体感したいと思いました。

## 少ないコードで、凝集度の高い高度な処理が書ける
2023年度個人的に設計力の向上に注力していました。設計で目指す指針は高凝集・疎結合だと思いますが、それをElixirでどう実現するのか?、に興味があります。
このことについては調査、学習したいと感じています。


