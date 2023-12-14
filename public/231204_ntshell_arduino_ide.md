---
title: 個人的に大好きなOSS【NT-Shell】をSpresense Arduino IDE環境に対応させた話
tags:
  - Spresense
  - NT-Shell
private: false
updated_at: '2023-12-06T01:19:57+09:00'
id: bb0083deba6f7e6beaa2
organization_url_name: rymansat
slide: false
ignorePublish: false
---
この記事は[Spresense Advent Calendar 2023](https://qiita.com/advent-calendar/2023/spresense)の4日目（12/4分）です。

# 概要
NT-ShellをSpresenseのArduino IDE環境で使えるようにした話です。

# NT-Shellとは?
ROM・RAM容量が小さいマイコンでも動作可能な軽量なシェルです。

https://cubeatsystems.com/ntshell/

NT-Shellを組み込むとシリアルコンソール経由などでつぎのことができて快適なデバッグができるようになります（私がよく使うコマンドの抜粋です）。

* Backspaceキーで文字が消せる!!!
* カーソルキーで行の移動ができる!!!
* Deleteキーで文字が消せる!!
* 実行したコマンドの履歴が見れる!!
* CTRL+Aで行の先頭に移動できる!
* CTRL+Eで行の末尾に移動できる!


# わたしのNT-Shellの使い方
私の趣味開発はLチカを動作確認後はNT-Shellを組み込むことが多いです。
NT-Shellをテスト用コマンドのインターフェース・入り口として使ったりしています。
テスト用コマンドも容易に組込み可能で重宝しています。

以前NT-Shellを組み込んだ事例があります。ご興味あれば見てみてください。

* TOPPERS/ASP開発時に初めにNT-Shellとxprintfを組み込んだら幸せになった話

https://qiita.com/juraruming/items/107f93852ec0e2993b7d


* STM32CubeIDEにCppUTestを環境構築し、STM32マイコンでTDDする(2) ～シリアル通信でCppUTestを実行する～

https://qiita.com/juraruming/items/2fbf398b9587757b17a2

こちらはNT-ShellからテストフレームワークCppUTestを使ってみた事例です。


# コード
コードはGitHubリポジトリに置きました。

https://github.com/grace2riku/spresense_arduino_ntshell

# 確認環境
## ハードウェア
つぎのハードウェアで確認しました。

* Spresenseメインボード

## ソフトウェア
Arduino IDEで確認しました。Arduino IDEのバージョンはつぎのとおりです。

```
バージョン：2.0.4
日付：2023-02-27T16:11:45.995Z
CLIバージョン：0.31.0

Copyright © 2023 Arduino SA
```

## 環境構築
### Spresense Arduino 開発環境のセットアップ

事前につぎの項目を実施し、開発環境を構築しておきます。

* Arduino IDEでの開発 -> Spresense Arduino スタートガイド -> [1. Spresense Arduino Library のインストール方法](https://developer.sony.com/spresense/development-guides/arduino_set_up_ja.html#_spresense_arduino_library_%E3%81%AE%E3%82%A4%E3%83%B3%E3%82%B9%E3%83%88%E3%83%BC%E3%83%AB%E6%96%B9%E6%B3%95)

* Arduino IDEでの開発 -> Spresense Arduino スタートガイド -> [2. プログラミング環境の設定](https://developer.sony.com/spresense/development-guides/arduino_set_up_ja.html#_%E3%83%97%E3%83%AD%E3%82%B0%E3%83%A9%E3%83%9F%E3%83%B3%E3%82%B0%E7%92%B0%E5%A2%83%E3%81%AE%E8%A8%AD%E5%AE%9A)


# 実装
NT-ShellのArduino環境への対応はこちらのWeb記事を参考にさせていただきました。

https://blog.boochow.com/article/442245440.html

実装のポイントをいくつか説明します。

## setup
Arduinoはsetupで初期化を行い、メイン処理はloopに実装する、という構造だと思います。
setupでNT-Shellの初期化をおこないます。

ntshell_initに以下を設定しています。

* コンソールからのリード関数
* コンソールへのライド関数
* コールバック関数（ユーザーコマンド関数）

```c:spresense_arduino_ntshell.ino
void setup() {
  // put your setup code here, to run once:
  Serial.begin(115200);
  while (!Serial) {
    ;;
  }

  ntshell_init(
    &ntshell,
    func_read,
    func_write,
    func_callback,
    (void *)(&ntshell));

  ntshell_set_prompt(&ntshell, PROMPT_STR);

  Serial.println("Wellcome to Spresense Arduino.\r\n type 'help' for help.");
  Serial.print(PROMPT_STR);
  Serial.flush();
}
```

### ntshell_initパラメータ設定
ntshell_initに設定している関数の実装について説明します。
今回はターミナルからシリアル通信（UART）でNT-Shellを使うことにします。

## リード
ターミナルから入力された文字をリードする関数です。
ポイントとしてはターミナルから入力がなければ制御が返るようにSerial.available()で受信データの有無を確認してからSerial.readBytesしているところです。

```c:spresense_arduino_ntshell.ino
static int func_read(char* buf, int cnt, void* extobj) {
  if (Serial.available())
    return Serial.readBytes(buf, cnt);
  else
    return 0;
}
```

## ライト
ターミナルに文字をライトする関数です。
シンプルにUARTに送信する処理になっています。

```c:spresense_arduino_ntshell.ino
static int func_write(const char* buf, int cnt, void* extobj) {
  return Serial.write(buf, cnt);
}
```

## コールバック関数
ターミナルでエンターキーを押下し入力が確定したときに呼び出されるコールバック関数を指定します。
この関数の中でユーザーのコマンドの入り口の関数を指定します。
今回の場合はusrcmd_execute関数を呼び出します。

```c:spresense_arduino_ntshell.ino
static int func_callback(const char* text, void* extobj) {
  return usrcmd_execute(text);
}
```

## loop
loop関数の実装は前述のWeb記事をそのまま実装しました。

https://blog.boochow.com/article/442245440.html


## ユーザーコマンド
ターミナルの入力が確定するとコールバック関数の中でusrcmd_execute関数が呼び出されます。
ntopt_parse関数でターミナルの入力文字が解析され、usrcmd_ntopt_callback関数にお馴染みの引数argc（文字列の個数）, argv（入力文字列）として渡されます。
その後はコマンドテーブル（cmdlist）とターミナルの入力文字列を確認します。
ターミナル入力文字列がコマンドテーブルに定義されているコマンド名（配列の0番目の要素）と一致していたら、コマンドテーブルの関数（配列の2番目の要素）を実行します。

つぎのコードは2つのコマンドが実装されています。

* ターミナル入力文字列が【help】の場合、usrcmd_help関数を実行する
* ターミナル入力文字列が【info】の場合、usrcmd_info関数を実行する


```cpp:usrcmd_spresense_arduino.cpp
typedef struct {
    char *cmd;
    char *desc;
    USRCMDFUNC func;
} cmd_table_t;

static const cmd_table_t cmdlist[] = {
    { "help", "This is a description text string for help command.", usrcmd_help },
    { "info", "This is a description text string for info command.", usrcmd_info },
};

int usrcmd_execute(const char *text)
{
    return ntopt_parse(text, usrcmd_ntopt_callback, 0);
}

static int usrcmd_ntopt_callback(int argc, char **argv, void *extobj)
{
    if (argc == 0) {
        return 0;
    }
    const cmd_table_t *p = &cmdlist[0];
    for (int i = 0; i < sizeof(cmdlist) / sizeof(cmdlist[0]); i++) {
        if (ntlibc_strcmp((const char *)argv[0], p->cmd) == 0) {
            return p->func(argc, argv);
        }
        p++;
    }
    uart_puts("Unknown command found.\r\n");
    return 0;
}

static int usrcmd_help(int argc, char **argv)
{
    const cmd_table_t *p = &cmdlist[0];
    for (int i = 0; i < sizeof(cmdlist) / sizeof(cmdlist[0]); i++) {
        uart_puts(p->cmd);
        uart_puts("\t:");
        uart_puts(p->desc);
        uart_puts("\r\n");
        p++;
    }
    return 0;
}

static int usrcmd_info(int argc, char **argv)
{
    if (argc != 2) {
        uart_puts("info sys\r\n");
        uart_puts("info ver\r\n");
        return 0;
    }
    if (ntlibc_strcmp(argv[1], "sys") == 0) {
        uart_puts("NXP LPC824 Monitor\r\n");
        return 0;
    }
    if (ntlibc_strcmp(argv[1], "ver") == 0) {
        uart_puts("Version 0.0.0\r\n");
        return 0;
    }
    uart_puts("Unknown sub command found\r\n");
    return -1;
}
```

新しいコマンドを追加するときはつぎのようにします。

* コマンドテーブルにコマンド文字列、コマンドの説明、コマンドの関数を追加する
* コマンドテーブルに追加したコマンド関数を実装する


# 動作確認
NT-Shellに組み込んだコマンドを動作確認します。

SepresenseメインボードのUSBポートとPCを接続し、ターミナルからつぎのコマンドを実行しシリアルのパスを確認します。

```
$ ls /dev/cu.usb*
/dev/cu.usbserial-14130
```

シリアルコンソール（私はminicomを使用）からSpresenseに接続します。

```
$ minicom -D /dev/cu.usbserial-14130 -b 115200
```

接続できるとつぎの画面になります。

```
Welcome to minicom 2.8

OPTIONS:
Compiled on Jan  4 2021, 00:04:46.
Port /dev/cu.usbserial-14130, 00:09:23

Press Meta-Z for help on special keys

Wellcome to Spresense Arduino.
 type 'help' for help.
>
```

helpコマンドを実行します。
コマンドの説明が確認できました。
```
>help
help    :This is a description text string for help command.
info    :This is a description text string for info command.
```

infoコマンドを実行します。
```
>info
info sys
info ver
```

コマンドの引数が足りなかったようです。引数を指定して再度実行します。
info sysコマンドはシステム名称を確認できます。

```
>info sys
NXP LPC824 Monitor
```

info verコマンドはバージョンを確認できます。

```
>info ver
Version 0.0.0
```

# 感想
NT-Shellを実装し、動作確認時に気づきましたがArduino IDEのシリアルモニタはGUIで操作でき、NT-Shellのありがたみがあまり伝わらないかもしれません。
Arduino IDEのシリアルモニタを使用せず、他の汎用的なターミナル（Teraterm, iTermその他）を使うときはNT-Shellの効果が実感できると思います。
ターミナルのライト・リード・コールバック関数を用意すれば使える、コマンドの追加も簡単にできるので実装のハードルもそれほど高くないと思います。
気になった方は一度使ってみてください。
