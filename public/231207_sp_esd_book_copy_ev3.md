---
title: 書籍【組込みソフトウェア開発のための構造化プログラミング】の写経をレゴマインドストーム（EV3）でやってみた
tags:
  - 設計
  - EV3
  - 組込みソフトウェア開発のための構造化プログラミング
private: false
updated_at: '2023-12-07T00:30:52+09:00'
id: a515e7df15a805fc0478
organization_url_name: persolcrosstechnology
slide: false
ignorePublish: false
---
この記事は[設計こばなし Advent Calendar 2023](https://qiita.com/advent-calendar/2023/software_design_talk)の7日目（12/7分）です。

# 概要
書籍 **【組込みソフトウェア開発のための構造化プログラミング】** に記載の設計・コードに感銘を受けました。
このコードを写経し、組込み装置で実際に動かして書籍の設計を学びたいと思いました。
写経したコードで実際に動いている動画がつぎになります。

<iframe width="560" height="315" src="https://www.youtube.com/embed/DDmrg3R3WEk?si=o5TLIBLOOuwXC8vk" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" allowfullscreen></iframe>

※組込みソフトウェア開発のための構造化プログラミング

https://www.shoeisha.co.jp/book/detail/9784798147611

# 動作確認環境
書籍の対象ハードウェアはレゴマインドストームNXTでした。レゴマインドストームNXTは持っていないのでレゴマインドストームEV3を使うことにしました。

## EV3について
### EV3の組立て
EV3はETロボコンのロボット**HackEV**としました。
こちらが組立てたHackEVです。

![HackEV.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/171866/adfc5f44-e647-ab6d-8458-9fad8a36acc1.png)

HackEVの組立てはこちらの組立て図を参考にしました。

https://github.com/ETrobocon/etroboEV3/blob/master/BuildingInstructions/HackEV_L8b.pdf

### ソフトウェア開発環境
こちらのETロボコンの環境を使わせてもらうことにしました。

https://github.com/ETrobocon/etrobo

#### OS
ソフトウェア開発環境でインストールされるRTOS TOPPERS/EV3RTを利用しました。
インストールされたバージョンは確認時点(2023/11/10)の最新版でした。

* バージョン: リリース1.1
* 最終更新日: 2021年06月25日

TOPPERS/EV3RTのリリース履歴はこちらに記載があります。

https://dev.toppers.jp/trac_user/ev3pf/wiki/Download


# コード
写経したコードはつぎのGitHubリポジトリに置きました。

https://github.com/grace2riku/sp_esd_book_copy_ev3


# 設計図
書籍で紹介されているのはライントレースを行うロボットです。
こちらはソースコードから作成した構造図はつぎになります。

![src.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/171866/76eb9b35-f828-3150-e55c-7ac76a6e393f.png)

ざっくりとコードを説明します。

* tr_traceCourse関数:トップの関数
* cs_detectDifference関数: コースのズレを検知する
* nv_naviCourse関数: コースを決めてモータを駆動する

tr_traceCourse関数は20msごとに呼び出しています。

こちらは自然言語で書かれた構造図です。

![language.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/171866/e8ec5ecf-5fea-4005-b6fb-e28095b99c8d.png)


# 書籍の設計の良いところ
書籍で紹介されている設計の良いところを紹介します。

## 深くまで追わなくても動きを予測できる
tr_traceCourse関数というトップ関数からは、3つの関数を呼び出しています。
コースのズレを検知して、コースをナビゲートして駆動するという動きを予測できます。
何をしているのか予測できる関数は理解しやすい良い関数です。

## 関数名のわかりやすさ
たとえばcs_detectDifference関数関数（コースとのズレを検出する）は動詞 + 目的語の組み合わせで簡潔に機能を示しています。

## 入力部と出力部が分割されている
ライントレースの入力は自然言語で書かれた構造図でいうとつぎになります。

* 【光センサ値を取得】する
* 光センサの濃淡から【路面の色を判定する】
* 判定した路面の色から【ラインズレを検知する】

出力はつぎになります。

* 進行方向を設定する
* 左右車輪を駆動する

真ん中の【走行コースをナビゲートする】はラインズレを入力として受け取り、進行方向を決めて出力部に渡しています。
左側から得た入力を真ん中で変換し、右側で出力するという流れでわかりやすい良い構造です。

## 単方向の依存性になっている
構造図を見ると上位の階層が呼び出している下位の階層のモジュールにだけ依存しているので良い設計です。

## 同じ階層での横のつながりがない
同じ階層で横のつながりがなく、良い設計です。

## モジュールで何をしているのかがわかる
自然言語の構造図のモジュール名は「〜を・・・する」という形式で書かれているため、一目で何をしているかがわかる良い命名です。

