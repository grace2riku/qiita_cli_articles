---
title: 宝探しゲームをつくってこども達と遊んだ話
tags:
  - micro:bit
private: false
updated_at: ''
id: null
organization_url_name: persolcrosstechnology
slide: false
ignorePublish: false
---
この記事は[micro:bit Advent Calendar 2023 Advent Calendar 2023](https://qiita.com/advent-calendar/2023/microbit)の12日目です。

# 記事の概要
去る2023/10/29のハロウィン近く🎃にこどもたちとmicro:bit v2を使った宝探しゲームをしました。
宝探しゲームのmicro:bitのコード、ゲームの感想などの話です。

下のポストは宝探しゲームのフィールドデバッグの様子とmicro:bit v2でつくった装置の写真です。

<blockquote class="twitter-tweet" data-media-max-width="560"><p lang="ja" dir="ltr">お昼休みにmicro:bitのフィールドデバッグしています。<br>日曜日に近所の子供達とmicro:bitを使った宝探しゲームをやってみようかなと思ってます。<br><br>micro:bitを2台使って、それぞれ送信ビーコン、受信ビーコンの機能を与えます。… <a href="https://t.co/McEvsdpxaG">pic.twitter.com/McEvsdpxaG</a></p>&mdash; k-abe@組み込まれた猫使い🧙‍♂️ (@juraruming) <a href="https://twitter.com/juraruming/status/1717746887502315597?ref_src=twsrc%5Etfw">October 27, 2023</a></blockquote> <script async src="https://platform.twitter.com/widgets.js" charset="utf-8"></script>


下のポストはこどもたちが宝探しゲームしている様子です。

<blockquote class="twitter-tweet"><p lang="ja" dir="ltr">micro:bitを使った宝探し（ハロウィン🎃ということでお菓子セット）、好評でした。<br>もう1回やる〜となり、3回くらい遊んでもらえました。<br><br>私は少しmicro:bitに詳しくなれましたし、こどもたちにも興味を持ってもらえてよかったです😄 <a href="https://t.co/jtihH32fbB">pic.twitter.com/jtihH32fbB</a></p>&mdash; k-abe@組み込まれた猫使い🧙‍♂️ (@juraruming) <a href="https://twitter.com/juraruming/status/1718520044676882815?ref_src=twsrc%5Etfw">October 29, 2023</a></blockquote> <script async src="https://platform.twitter.com/widgets.js" charset="utf-8"></script>


# 宝探しゲームの概要
宝探しゲームの概要です。

* 宝はお菓子の詰め合わせ（ハロウィンが近いという理由で決定）
* ゲームは屋外の公園にて開催した。
* micro:bit v2を2台を使ったゲーム。
* 1台目は無線を送信するビーコン。
モバイルバッテリと接続し、宝箱の中のお菓子と一緒に任意の場所に隠す。
* 2台目は無線を受信するビーコン。
電池ボックスで給電しプレイヤーにもってもらう。
送信ビーコンからの電波強度をmicro:bit v2のLEDに棒グラフで表示、音で知らせる。
宝箱に近づくと棒グラフが大きくなる、音が大きくなる。
* 事前に宝探しゲームをやることを伝えない。公園でプレイヤーが遊んでいるときにサプライズで宝を隠したので探してみて?、という流れにする。

## ゲームのプレイヤー
宝探しゲームをするプレイヤーはつぎのとおりです。

* 小学1年生の女の子 1人　
* 幼稚園年長の女の子 1人
* 2人は幼馴染

# 準備したもの
宝探しゲームで準備したものです。

| もの | 内容 | 個数 | 
| ---- | ---- | ---- |
| micro:bit v2 | 送信用ビーコン、受信用ビーコン | 2 |
| モバイルバッテリー | 送信用ビーコン駆動用 | 1 |
| 電池ボックス | 受信用ビーコン駆動用 | 1 |
| 宝 | お菓子詰め合わせ | 1 |
| 紙の箱 | 宝、送信用ビーコン格納用 | 1 |
| 宝探し開催場所の地図 | ゲーム開催前に印刷済み | 1 |


# コードの説明
コードはMakeCodeで作成しました。
こちらの近接ビーコンをベースに改良していきました。

## ベースにした近接ビーコンのコード

https://microbit.org/ja/projects/make-it-code-it/proximity-beacon/


# 送信、受信ビーコン共通事項
送信、受信ビーコン共通事項です。

* 無線のグループは1に設定し通信できる条件にする

## 送信ビーコン

https://makecode.microbit.org/S51442-22387-49649-75220

* 200ms間隔で”1"を送信する
* 送信・受信ビーコン区別のため、送信ビーコンは”T”をLEDに表示する（送信ビーコンを意味する"T"とした）
* 起動時のボタン押下状況によって無線の強度、LEDの明るさを分岐している。これは無線強度フィールドデバッグ用のコードの残骸。
* この送信ビーコンを宝と一緒に任意の場所に隠す

## 受信ビーコン

https://makecode.microbit.org/S17783-40734-54748-38132

* 送信・受信ビーコン区別のため、受信ビーコンは”R”をLEDに表示する（受信ビーコンを意味する"R"とした）
* 無線を受信したら信号強度で棒グラフを描いている
* 信号強度の値（0〜9）をボリューム変数（128〜255）に割り当て、音を鳴らす
* この受信ビーコンをプレイヤーに持ってもらい、棒グラフの表示・音を頼りに宝（受信ビーコン）を探してもらう

# 工夫したところ
## 開催日までの準備
### 無線強度のセッティング
屋外で宝探しをするにあたり一番の懸念は無線強度の設定をどうすれば良いか?、でした。
これは宝探しゲーム当日までに屋外で動きを確認し、パラメータを決定しました。
無線の送信強度（0〜7で設定できる）をどれにすれば良いのかわからなかったので屋外でデバッグしました。
送信強度7だと距離が離れていても無線を検出してしまいました。ゲームがすぐに終わらないように、宝探しのゲーム性がなくならないように1にすることにしました。


### モバイルバッテリーの選定
送信のビーコンはモバイルバッテリーで動かす予定のため、バッテリーの確認をおこないました。
確認した結果、特定のモバイルバッテリーでのみ動くことがわかりました。

![231212_treasure_hunt_game_vbat.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/171866/ab7f3628-8758-e7f5-4499-feef8a1c2a24.png)

動いたバッテリーはSONYのCP-V3でした。

https://www.sony.jp/battery/products/CP-V3/

ゲーム当日前に確認しておいて良かったです。
今後の電子工作に役立ちそうなことがわかったので良かったです。

### ゲームが成立するかシミュレーション
micro:bitのコード、ハードウェアの動作確認はできました。

あとはつぎを検討しました。

* 宝探しゲームのルールを理解できるように伝えること
* 適度な難易度設定

#### 宝探しゲームのルールを理解できるように伝えること
システムができてもゲームの手順を理解してもらえなければ楽しめないと思いました。
そこで受信ビーコンを持ち、宝に近づくと棒グラフが増えること、音が大きくなることを動画で説明することにしました。
ハロウィンが近いということもあり、仮装したおばけ（怪人）に扮してゲームの説明動画を録画しました👻。

![IMG_1580.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/171866/5d3f59e8-bf60-9804-3fe6-eac1da3e69f2.png)


#### 適度な難易度設定
説明動画で宝が近くにあると受信ビーコンが教えてくれる、と説明しても小学1年生・幼稚園年長のこどもたちにとっては難易度が高い（ゴール設定があいまい）、途中で飽きてしまうかもしれないと思いました。
そこで宝探しゲームの公園のGoogleマップ航空写真をA4用紙で印刷しておき、おおよその宝の隠し場所をマーキングしておく宝の地図方式にしました。


# ゲーム実行結果
## 予定外の出来事
いろいろと準備をしましたが予定と違うこともおきました。
誰かが公園に宝を隠したんだって→宝探しやってみる?→この動画を見て〜でゲームの説明動画を見せはじめましたが音が小さすぎてこどもたちに伝わっていないようでした（ヘルメット越しで説明動画で声がこもっていた、屋外で聞こえづらかったのだと思います）。
説明動画の視聴は止めて送信ビーコンが宝（受信ビーコン）に近づくと絵（棒グラフ）、音で知らせてくれるよ、というざっくりな説明をしてゲームをスタートしました。

## 結果
1回目の宝探しでは無線強度の棒グラフと音をたよりに「宝はどこだ、どこだ」と宝を探していました。宝の隠し場所に近づくと棒グラフ、音が変化したことに気づき盛り上がっている様子でした。そして見事に宝を見つけてハロウィンのお菓子をゲットできました🎃。


# ゲームの評価
宝探しゲームは合計3回遊んでもらえました。こどもたちにmicro:bit v2を使った宝探しに興味を貰えたようで良かったです。
実装、事前の準備などいろいろとやることはありましたがこどもたちに喜んでもらえてやって良かったと思えるイベントになりました。