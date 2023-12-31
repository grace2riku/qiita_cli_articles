---
title: 組込みソフトウェアの文脈で【継承】の効果的な使い所を考える
tags:
  - 設計
  - 継承
private: false
updated_at: ''
id: null
organization_url_name: persolcrosstechnology
slide: false
ignorePublish: false
---
この記事は[設計こばなし Advent Calendar 2023](https://qiita.com/advent-calendar/2023/software_design_talk)の15日目です。

# 概要
2023年度、設計力アップのためSOLID原則を学び、社内・社外向け勉強会を開催・発信してきました。
わたしはSOLID原則の中で**リスコフの置換原則**の理解に一番時間がかかりました。
ピーコック アンダーソンさんの講座 [オブジェクト指向の原則２：リスコフの置換原則と継承以外の解決方法](https://www.udemy.com/course/objectfive2/)で大分理解は進みました。
ピーコック アンダーソンさんのリスコフの置換原則の講座では**継承**に変わる方法で要求を実現する方法について具体的に、丁寧に解説いただいています。
講座では継承に変わる方法の解説が多かったですが、継承の使い所についてもWindows Formアプリケーションの事例を解説してくださっています。
私は組込みソフトウェアが専門領域なので組込みソフトウェアで継承を使うと効果的な事例を調査してみることにしました。

# 

# 作成資料一覧
SOLID原則の勉強会の資料はこちらに保存しています。よければ参照ください。

| No | 開催日 | 勉強会タイトル&資料リンク | Xスペース録音URL |
|:-----------|:------------|:------------|:------------|
| 1       | 2023/7/20        | [#1 単一責務の原則](https://www.docswell.com/s/juraruming/K98M19-2023-07-20-143150)         | -         |
| 2       | 2023/8/28        | [#2 インターフェース分離の原則](https://www.docswell.com/s/juraruming/KJLWEM-2023-08-28-005645)         | [URL](https://twitter.com/i/spaces/1lDGLnPowBZxm?s=20)         |
| 3       | 2023/9/28        | [#3 依存性逆転の原則](https://www.docswell.com/s/juraruming/5EN9ME-2023-09-28-065704)         | [URL](https://twitter.com/i/spaces/1YqKDoZgMvzxV)         |
| 4       | 2023/10/26        | [#4 開放閉鎖の原則](https://www.docswell.com/s/juraruming/5LL836-2023-10-26-074605)         | [URL](https://twitter.com/i/spaces/1nAKEaOpWzVKL)         |
| 5       | 2023/11/30        | [#5 リスコフの置換原則](https://www.docswell.com/s/juraruming/Z6Y8X7-2023-11-30-081051)         | [URL](https://twitter.com/i/spaces/1rmGPMWRdQdJN)         |
