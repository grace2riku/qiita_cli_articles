---
title: 組込みソフトウェア開発と生成AI その1 兼用端子設定
tags:
  - 組み込み開発
  - renesas
  - RXマイコン
  - ChatGPT
private: false
updated_at: '2024-06-17T23:18:28+09:00'
id: 5a841623ac0833795373
organization_url_name: persolcrosstechnology
slide: false
ignorePublish: false
---
# この記事は何?
2024/06/10 〜 2024/07/17の期間、Qiitaで

* **[Qiita Engineer Festa 2024](https://qiita.com/official-campaigns/engineer-festa/2024)**

が開催されています。

テーマのひとつに

* **[お題は不問！Qiita Engineer Festa 2024で記事投稿！](https://qiita.com/official-events/16baee61b1d8bd4aac5a)**

がありました。
テーマに該当する**組込みソフトウェア開発と生成AI その1 兼用端子設定**について書きました。

# 背景
私は組込みソフトウェア開発をしています。

生成AIを使って業務を楽に、効率化することはできないかと思い、試してみたいと考えました。

# テーマ
組込みソフトウェア開発ではマイコンメーカーのマニュアルを読み、作業するシーンが多くあります。

開発対象のシステムが実現したい機能によりマイコンの端子設定から適切な機能を選択し、適切なレジスタ設定をする必要があります。

今回はこの**端子設定**を生成AIの力を借りてできないかと思い、試してみました。

# 対象
## 生成AI
生成AIはChatGPT 4oを使用します。自分は課金していてPlusプランで使っています。

## 対象マイコン
対象マイコンはRenesasエレクトロニクス社のRX630とします。
たまたま最近の仕事で触れる機会があったのでこのマイコンを選択しました。

[RX630 USB、12 ビット A/Dなど周辺機能も強化した 32 ビットマイクロコントローラ](https://www.renesas.com/jp/ja/products/microcontrollers-microprocessors/rx-32-bit-performance-efficiency-mcus/rx630-32-bit-microcontroller-usb-12-bit-ad-and-other-enhanced-peripheral-functions)

## 対象テーマ
RX630にはCANモジュールが搭載されています。
マニュアル [RX630グループ ユーザーズマニュアル ハードウェア編 Rev.1.60](https://www.renesas.com/jp/ja/document/mah/rx630-group-users-manual-hardware-rev160?r=1054431)の34. CAN モジュール（CAN）です。

今回は **CANモジュールのチャンネル0の端子設定手順を生成AIの力を借りて明確にできるか?** をテーマにしたいと思います。

その他、条件はつぎのとおりとします。
* マイコンのパッケージは144ピン LQFPパッケージとする
* P32（CAN送信: CTX0）、P33（CAN受信: CRX0）をCAN0として使う

## 端子設定とは?
ユーザーズマニュアル87ページ 表1.8 機能別端子一覧（144ピンLQFP） (1 / 5)のピン番号26、27が該当の端子です。

![表1.8 機能別端子一覧（144ピンLQFP） (1 : 5)のピン番号26、27.jpg](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/171866/ec604460-6984-c0fe-2ea0-5eb6c3d3afa2.jpeg)

ピン番号26、27は以下の機能を兼用しています。
今回はCAN0受信、送信端子の設定を対象にします。

| ピン番号 | 端子機能名 | 機能 |
| :--- | :--- | :--- |
| 26 | P33 | IOポート |
|  | MTIOC0D/TIOCD0/TMRI3/PO11/POE3# | タイマ |
|  | RXD6/RXD0/SMISO6/SMISO0/SSCL6/SSCL0/CRX0 | 通信<br>★CRX0がCAN0受信で今回の設定対象機能 |
| 27 | P32 |  |
|  | MTIOC0C/TIOCC0/TMO3/PO10/RTCOUT/RTCIC2 | タイマ |
|  | TXD6/TXD0/SMOSI6/SMOSI0/SSDA6/SSDA0/CTX0 | 通信<br>★CTX0がCAN0送信で今回の設定対象機能


# 作業手順
生成AIにつぎの手順で端子設定の明確化を依頼します。

1. マニュアルを生成AIに読み込ませる
2. CANモジュールのチャンネル0の端子設定手順の条件をプロンプトで指示する

## 1. マニュアルを生成AIに読み込ませる
ユーザーズマニュアルのPDFを生成AIに読み込ませます。

## 2. CANモジュールのチャンネル0の端子設定手順の条件をプロンプトで指示する
マイコンのパッケージ、CAN0受信・送信端子のピン番号を指示します。
パッケージ、CAN0受信・送信端子は複数の選択肢があるため、条件を1つに絞ります。

## はじめのプロンプト
はじめにユーザーズマニュアルのPDFを生成AIに読み込ませ、つぎのプロンプトを試してみました。

```
添付ファイルはRenesas社の32 ビットマイクロコントローラ　RX630のユーザーズマニュアル　ハードウェア編です。

このマイクロコントローラの特徴を要約してください。
```

### はじめのプロンプトの結果
結果はつぎになりました。
この内容はマニュアル46ページ 【特徴】に記載の内容を転記しているようで特に問題ありませんでした。

```md : 
Renesas社のRX630マイクロコントローラの特徴を以下に要約します。

1. CPU:
   - 32ビットRX CPUコア内蔵。
   - 最大動作周波数100MHzで165 DMIPSの性能。
   - 32ビット単精度浮動小数点演算（IEEE754準拠）。
   - 2種類の積和演算器（メモリ間、レジスタ間）。
   - 32ビット乗算器（1クロックで実行）。
   - 除算器（2クロックで実行）。
   - 高速割り込み機能。
   - 5段パイプラインのCISCハーバードアーキテクチャ。
   - 可変長命令形式：コードを大幅に短縮。
   - メモリプロテクションユニット（MPU）対応。
   - JTAGおよびFINE（2線式）のデバッグインターフェース。

2. 消費電力低減機能:
   - 2.7V〜3.6V動作の単一電源。
   - 全周辺機能サポート時、500μA/MHzの消費電力。
   - 専用電源で動作可能なRTC（最低2.3V動作）。
   - 4種類の低消費電力モード。

3. 内蔵メインフラッシュメモリ:
   - 100MHz動作、10ns読み出しサイクル（ウェイトなし）。

4. 通信機能:
   - USB 2.0フルスピードファンクション。
   - CAN、10ビットおよび12ビットのA/Dコンバータ。
   - リアルタイムクロック（RTC）。
   - 合計22本の通信機能。

これらの特徴により、RX630は高性能、低消費電力、豊富な通信機能を持つマイクロコントローラとして、
多くの組み込みシステムに適しています.
```


## CAN0端子設定のプロンプト
CAN0端子設定は具体的につぎのプロンプトにしました。

```
144ピンLQFPパッケージでP32、P33をCAN0の端子として使いたいです。
レジスタ設定を教えてください。
```

### CAN0端子設定のプロンプトの結果

```
144ピンLQFPパッケージでP32、P33をCAN0の端子として使用するためのレジスタ設定は以下の通りです：

1. 端子機能設定:
   - P32をCTX0として設定します。
   - P33をCRX0として設定します。

2. 端子設定レジスタ:
   - P32のPFS（Pin Function Select）レジスタを`10000b`に設定します。
   これにより、P32はCTX0として機能します。
   - P33のPFSレジスタを`10000b`に設定します。これにより、P33はCRX0として機能します。

3. 設定手順:
   - `MPC.P32PFS.BYTE = 0x10;`  // P32をCTX0に設定
   - `MPC.P33PFS.BYTE = 0x10;`  // P33をCRX0に設定
   - `PORT3.PMR.BIT.B2 = 1;`  // P32の周辺機能ビットをセット
   - `PORT3.PMR.BIT.B3 = 1;`  // P33の周辺機能ビットをセット

これらの設定により、P32とP33をCAN0のCTX0およびCRX0として使用することができます.
```

### 俺じゃなきゃ見逃しちゃうね
HUNTER×HUNTERの団長の手刀を見逃さなかった人のセリフがでました（このセリフを言いたかった）。

一見、前のプロンプトの結果は問題なさそうですが、レジスタ設定に考慮漏れがあります。
マイコンのレジスタ設定の細かい話ですが、*PFSというレジスタは書き込みプロテクトされているため、特定の手順を踏まないとレジスタ値の変更ができない仕様となっています。


### 修正版のプロンプト
書き込みプロテクトの件を以下のプロンプトで生成AIに教えてあげて再度、実行します。

```
MPC.P32PFS、MPC.P33PFSレジスタへの書き込みは
書き込みプロテクトレジスタ（PWPR）でPFSレジスタへの書き込みを許可しないと書き込めないようです。

583ページの

・21.4.1 端子入出力機能設定手順
・21.4.2 MPC レジスタ設定する場合の注意事項

に則ったレジスタ設定手順を教えてください。
```

### 修正版のプロンプトの結果
結果はつぎになりました。

144ピンLQFPパッケージでP32、P33をCAN0の端子として使用するためのレジスタ設定手順を、書き込みプロテクトレジスタ（PWPR）を含めて説明します。以下の手順に従って設定を行ってください。

1. **書き込みプロテクト解除**:
   まず、MPCレジスタへの書き込みを許可するために、PWPRレジスタを設定します。

   ```c
   MPC.PWPR.BIT.B0WI = 0;  // 書き込みプロテクト解除
   MPC.PWPR.BIT.PFSWE = 1; // PFSレジスタへの書き込みを許可
   ```

2. **P32、P33のPFSレジスタ設定**:
   次に、P32をCTX0、P33をCRX0として設定します。

   ```c
   MPC.P32PFS.BYTE = 0x10; // P32をCTX0に設定
   MPC.P33PFS.BYTE = 0x10; // P33をCRX0に設定
   ```

3. **周辺機能選択ビットの設定**:
   P32およびP33の周辺機能ビットをセットします。

   ```c
   PORT3.PMR.BIT.B2 = 1; // P32の周辺機能ビットをセット
   PORT3.PMR.BIT.B3 = 1; // P33の周辺機能ビットをセット
   ```

4. **書き込みプロテクトの再設定**:
   最後に、PWPRレジスタを元に戻して書き込みプロテクトを再度有効にします。

   ```c
   MPC.PWPR.BIT.PFSWE = 0; // PFSレジスタへの書き込みを禁止
   MPC.PWPR.BIT.B0WI = 1;  // 書き込みプロテクトを有効
   ```

以上の手順に従うことで、P32およびP33をCAN0の端子として設定できます.


# 感想
今回はマイコンの兼用端子設定について生成AIの力を借りて試してみました。
感想としてはつぎのとおりです。

## 初回の結果が予想を超えてきた
初回の結果はプロテクトレジスタのことを考慮していなかったとはいえ、プロテクトレジスタ以外のCAN兼用端子設定は問題なしでした。

ここまで正確な結果を導出できることに組込みソフトウェア開発 x 生成AIの可能性を感じました。

## 修正後の結果で要望にしっかり対応してくれた
プロテクトレジスタの件を考慮にいれて、再度、レジスタ設定を提示するよう依頼しました。
結果として、満点のレジスタ設定を提示してくれました。

## 組込みソフトウェア開発のパートナーとしての生成AIの可能性
すべてを生成AIに頼ることはリスクがあるかもしれませんが、例えば使ったことのないマイコンの機能を理解するためのとっかかりとして生成AIを使う、というシーンでは良いかもしれません。


最後まで読んでいただきありがとうございました🙇‍♀️
