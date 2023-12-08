---
title: SOLID 依存性逆転の原則を学習中に依存性注入を意識したCppUTestモックテスト環境をつくった
tags:
  - SOLID
  - 依存性逆転の原則
  - 依存性注入
  - CppUTest
  - CppuMock
private: false
updated_at: ''
id: null
organization_url_name: persolcrosstechnology
slide: false
ignorePublish: false
---
この記事は[設計こばなし Advent Calendar 2023](https://qiita.com/advent-calendar/2023/software_design_talk)の8日目（12/8分）です。

# 概要
今年度SOLID原則の勉強会を開催してきました。依存性逆転の原則を学ぶ過程でテストしやすくするCppUTestモックをつくったのでその共有です。
テストを効率的に実施したいなどの課題がある方の参考になればと思います。

# 依存性逆転の原則の勉強会について
## 勉強会概要
依存性逆転の原則の勉強会の概要はこちらです。

https://k-abe.connpass.com/event/294870/


## 勉強会の資料
勉強会の資料はこちらです。

<script async class="docswell-embed" src="https://www.docswell.com/assets/libs/docswell-embed/docswell-embed.min.js" data-src="https://www.docswell.com/slide/5EN9ME/embed" data-aspect="0.5625"></script><div class="docswell-link"><a href="https://www.docswell.com/s/juraruming/5EN9ME-2023-09-28-065704">ソフトウェア設計原則【SOLID】を学ぶ #3 依存性逆転の原則 by @juraruming</a></div>

## 勉強会の録音データ
Xのスペースの録音です。ご興味あれば資料を一緒に見ながら聞いてみてください。

https://twitter.com/i/spaces/1YqKDoZgMvzxV


# CppUTestのモックをつくった理由
依存性逆転の原則を学んでいると依存性注入も合わせて学ぶことが多いと思います。
インターフェースをテスト対象クラスのコンストラクタに引数で渡す、というものです。
依存性注入を使うことでテストコード、本番コードの切り替えも容易に出来るようになりテストがしやすくなります。

# サンプルコードについて
CppUTestのモック（CppUMock）を使ったサンプルコードはつぎのGitHubリポジトリにおきました。

https://github.com/grace2riku/dip_and_di_test_easy_env


## サンプルコードのテーマ
サンプルコードのテーマですがつぎのとおりです。

■テーマ：
* 仮空の医療モニタ。患者の生体情報をモニタリングできる。
* 今回は**設定値の読込み検証ロジック**について注目する。
* 実機がなくホストPCでテストを進めたい。ハードウェアに依存するコード（設定値を実機から読む）はモックにする。

■テーマの要件：
* 画面から装置の設定ができる
* 設定値の例
表示エリア選択、表示テキストの名称・色、画面の輝度、音量、音の種類、センサの校正値（ゲイン・オフセット）、その他いろいろ


## サンプルコードの構造について

### クラス図
サンプルコードのクラス図は下記のとおりです。

![dip_cpputest_mock.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/171866/20329956-df62-01fc-281b-48e8672c154f.png)

### コード

#### コード概要
コードの概要です。

| ファイル名 | 概要 |
| ---- | ---- |
| src/settingvalue_example/SettingValueValidation.cpp | 設定値検証ロジック。<br>validateが読み込んだ設定値を検証するメソッドで今回のテスト対象。 |
| include/settingvalue_example/ISettingValue.h | 設定値のインターフェース。SettingValueValidationに依存性注入する。 |
| tests/settingvalue_example/SettingValueMock.cpp | インターフェースの実装。実機をつかわないでテストするときに使う想定。readメソッドをモックにする。 |
| tests/settingvalue_example/SettingValueExampleTest.cpp | テストのモジュール。SettingValueValidationのvalidateメソッドのテストが書いてある。 |
| src/settingvalue_example/SettingValueRam.cpp | インターフェースの実装。実機に組込む本番コード。 |

#### コード詳細
##### SettingValueValidation.cpp

##### ISettingValue.h

##### SettingValueMock.cpp

##### SettingValueExampleTest.cpp

##### SettingValueRam.cpp


## 動作確認結果


# まとめ
