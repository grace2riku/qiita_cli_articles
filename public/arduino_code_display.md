---
title: Arduino IDEでコードの配置を変更し、見たいコードだけを表示する
tags:
  - Arduino
private: false
updated_at: '2024-01-31T23:01:52+09:00'
id: c437b5d06259839b2ade
organization_url_name: persolcrosstechnology
slide: false
ignorePublish: false
---
# 概要
Arduino IDEで開発しているときにコードが増えてくるとタブが大量に開かれ、気になっていました。
コードの配置を変更し、見たいコードだけをArduino IDEのタブに表示するようにしました。
その方法の共有です。

# 修正前の状態
コードが増えて気になっているときの状況をスクショしました。

下図はエクスプローラーのリスト表示です。
*.inoファイルと同じ階層にコードを配置しています。
![code_list_before.jpg](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/171866/889e8481-c61c-3e7f-253b-fbb52bc96a19.jpeg)


下図はArduino IDEでの表示です。
多数のタブが表示されています。全ファイルがタブ表示できないため目的のファイルのタブがない場合は右側の3点ボタン（...）を押下し、目的のファイルを選択・表示する必要があります。
コードのタブが多く表示されていると個人的に落ち着かない、気になります。

![arduino_before.jpg](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/171866/954b91ef-ef1e-bb4a-8638-a09581885b41.jpeg)


# 修正後の状態
Arduino IDEで見たいコードをだけを表示するように変更したときの状況をスクショしました。
src -> ntshellディレクトリを作成しコードをその中に配置するようにしました。
src -> ntshellディレクトリの中に配置していないコードがアプリケーションのメイン機能です。

![code_list_after.jpg](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/171866/caacbee0-227a-912a-5026-cc8fb79372d5.jpeg)

Arduino IDEではsrc -> ntshellディレクトリに格納されている以外のコード2つがタブ表示されています。
見たいコードだけをタブ表示し、修正前と比べてスッキリし快適な見た目になりました。
![arduino_after.jpg](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/171866/02a6f2cf-2fb2-a7cd-866c-2a6b3a7b57c1.jpeg)


# 対応内容
Arduino IDEにタブ表示したくないコードをsrcディレクトリ以下に移動しました。
この対応はWebで見たつぎの情報を参考にしました。

* [Arduino開発、ファイルを分割してC/C++で書く、VSCode](https://qiita.com/somehiro/items/6a6d954be159b6a5fccd)

* @matsujirushiさんに教えていただきました。

https://x.com/matsujirushi12/status/1724212585426526284?s=20


こちらの[Sketch specification](https://arduino.github.io/arduino-cli/0.27/sketch-specification/#src-subfolder)につぎの一文があります。

> In Arduino IDE 1.6.10 and newer, recursive compilation is limited to the src subfolder of the sketch folder.

DeepL翻訳:
Arduino IDE 1.6.10以降では、再帰コンパイルはsketchフォルダのsrcサブフォルダに制限されています。

開発中にコードが増えてきて見辛くなってきた場合は、つぎの対応をすると良さそうです。

* srcディレクトリを作成しコードを移動する
* ライブラリ化する。


最後まで読んでいただきありがとうございました。
