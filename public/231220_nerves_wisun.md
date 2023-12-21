---
title: NervesとWi-SUNドングルで自宅の消費電力を把握したい
tags:
  - Nerves
  - Wi-SUN
  - Livebook
private: false
updated_at: '2023-12-19T23:38:35+09:00'
id: 74f11b729a7859da63af
organization_url_name: persolcrosstechnology
slide: false
ignorePublish: false
---
この記事は[#NervesJP Advent Calendar 2023](https://qiita.com/advent-calendar/2023/nervesjp)の20日目です。

# 概要
Nervesとラトックシステム社が販売しているWi-SUN USBアダプターでスマートメータから消費電力計測を試してみた記事です。
すでにPython・JavaScriptでの消費電力可視化の事例がありますがNervesの実装でやってみたくなった、という個人的興味が動機です。
今回はPythonのWeb記事・実装を参考にNerves・Elixirで実装してみます。

## Python実装のWeb記事（今回の元ネタ）
こちらのWeb記事の手順でPythonサンプルスクリプトをNerves・Elixirに移植します。

* スマートメータからWi-SUN Bルートで電力量を知る（その１）

https://www.ratoc-e2estore.com/blog/2023/06/wsuha-01

* スマートメータからWi-SUN Bルートで電力量を知る（その２）

https://www.ratoc-e2estore.com/blog/2023/06/wsuha-02

* Pythonサンプルスクリプト（今回の移植対象コード）

https://www.ratoc-e2estore.com/blog/wsuha_broute_demo_01


## JavaScriptの実装
ラトックシステム社のWi-SUN USBアダプター（RS-WSUHA-P）のJavaScriptライブラリもあります。
私も試しましたが消費電力を取得できました。

https://github.com/futomi/node-wisunrb


# ハードウェア
システム全体は下図になります。

![hw.jpg](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/171866/c821a64d-cbaa-450b-8084-2673e4fad7b6.jpeg)

Nervesを動かすハードウェアはラズパイ4です（写真ではGrove Base Hatがラズパイ4の上に接続されていますが今回は使いません）。
ラズパイ4のUSBポートに接続している黒いものがラトックシステム社が販売しているWi-SUN USBアダプターです。

今回使用したのはWi-SUN USBアダプター RS-WSUHAシリーズ RS-WSUHA-Pです。

https://www.ratocsystems.com/products/wisun/usb-wisun/rs-wsuha/


# 事前準備
# Bルートパスワード、IDの入手
Wi-SUN USBアダプタがスマートメータと通信し消費電力を取得するためにはBルートのIDとパスワードを送配電会社から教えてもらう必要があります。
事前に送配電会社に申し込み、BルートのIDとパスワードを取得しておきます。
パスワードは12文字、IDは32文字の文字列です。
この記事ではパスワード、IDを変更して書いています。

# 動作確認手順
動作確認のアプローチとしてLivebookを使うことにしました。
理由としては私に移植対象のPythonコードをElixirに書き換えるスキルがないためです。
Livebookを使えばブラウザから移植対象のPythonコードをひとつずつ段階を踏んでElixirコードにして確認していけると思ったからです。

## Nerves, Livebookのインストール
Nerves, Livebookのインストールはこちらのページを参照しました。

https://github.com/nerves-livebook/nerves_livebook

ターゲットごとにSDカードイメージが用意されていますのでそれを書き込むだけで簡単に環境構築できました。
ラズパイ4の場合はつぎリンク先に格納されているファイル（nerves_livebook_rpi4.fw）でした。

https://github.com/nerves-livebook/nerves_livebook/releases


## Livebookログイン
SDカードイメージを書き込んだらSDカードを接続し、ラズパイ4の電源をONします。
ブラウザで

"http://nerves.local"

にアクセスし、パスワード【nerves】を入力すればLivebookにログインできます。

![nerves_login.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/171866/b63b03e5-9714-8584-235f-88329b5d5e0c.png)

## Wi-SUN USBアダプタ接続確認
Wi-SUN USBアダプタはラズパイ4からシリアルデバイスとして見えます。
Circuits.UARTライブラリのenumerate関数を使用し、デバイスのパス名を確認します。

Wi-SUN USBアダプタ未接続時にenumerate関数を実行します。

```elixir
Circuits.UART.enumerate()
```

Wi-SUN USBアダプタは存在していません。
```elixir
%{"ttyAMA0" => %{}, "ttyS0" => %{}}
```

Wi-SUN USBアダプタを接続しenumerate関数を実行します。
Wi-SUN USBアダプタのパスは"ttyUSB0"であることがわかりました。

```elixir
%{
  "ttyAMA0" => %{},
  "ttyS0" => %{},
  "ttyUSB0" => %{
    description: "FT230X Basic UART",
    serial_number: "DK8AAFGE",
    manufacturer: "FTDI",
    vendor_id: 1027,
    product_id: 24597
  }
}
```

## UART GenServerスタート
UART GenServerをスタートします。

```elixir
{:ok, pid} = Circuits.UART.start_link()
```

問題なくGenServerがスタートしました。
```elixir
{:ok, #PID<0.3604.0>}
```

## UARTオープン
UARTをオープンします。

```elixir
Circuits.UART.open(pid, "ttyUSB0", speed: 115_200, active: false)
```

オープンできました。
```elixir
:ok
```

## SKVERコマンドの確認
SKVERコマンドを実行します。

```elixir
Circuits.UART.write(pid, "SKVER\r\n")
```

```elixir
:ok
```

```elixir
Circuits.UART.read(pid)
```

バージョンが読めました。
```elixir
{:ok, "SKVER\r\nEVER 1.5.2\r\nOK\r\n"}
```

## Web記事 〜スマートメータからWi-SUN Bルートで電力量を知る（その１）〜を手順を実施
[スマートメータからWi-SUN Bルートで電力量を知る（その１）](https://www.ratoc-e2estore.com/blog/2023/06/wsuha-01)の手順を実施します。

### 受信したEDATA(bit列)を16進ASCIIに変換する機能の設定状況を確認する

```elixir
# bit列->16進ASCII変換機能の設定状態確認 リターン値が00は変換機能はOFF, 01は変換機能はON
Circuits.UART.write(pid, "ROPT\r\n")
Circuits.UART.read(pid)
```

変換機能ON（01）を確認できました。
```elixir
{:ok, "ROPT\r\nOK 01\r"}
```

### フロー制御の設定状態確認

```elixir
# フロー制御の設定状態確認 リターン値が00=フロー制御はOFF, 80はフロー制御はON
Circuits.UART.write(pid, "RUART\r\n")
Circuits.UART.read(pid)
```

フロー制御ON（80）を確認できました。
```elixir
{:ok, "RUART\r\nOK 80\r"}
```

## Web記事 〜スマートメータからWi-SUN Bルートで電力量を知る（その２）〜を手順を実施
[スマートメータからWi-SUN Bルートで電力量を知る（その２）](https://www.ratoc-e2estore.com/blog/2023/06/wsuha-02)の手順を実施します。

### SKRESETコマンドの確認

```elixir
# Reset WSUHA command buffer
Circuits.UART.write(pid, "SKRESET\r\n")
Circuits.UART.read(pid)
```

SKRESETコマンドのエコーバックを確認できました。
```elixir
{:ok, "SKRESET\r\n"}
```

SKRESETコマンド実行正常終了を示す"OK"をチェックします。
```elixir
Circuits.UART.read(pid)
```

SKRESETコマンド実行正常終了を確認できました。
```elixir
{:ok, "OK\r\n"}
```

### 送配電会社から通知されたスマートメータ(電力計)のパスワード設定
パスワードはWeb記事参照用に変更（0123456789AB）しています。

```elixir
# Set B-route Authentication Password（送配電会社から通知されたスマートメータ(電力計)のパスワード12文字）
Circuits.UART.write(pid, "SKSETPWD C 0123456789AB\r\n")
Circuits.UART.read(pid)
```

SKSETPWDコマンドのエコーバックとOKを確認できました。
```elixir
{:ok, "SKSETPWD C 0123456789AB\r\nOK\r\n"}
```

### 送配電会社から通知されたスマートメータ(電力計)のID設定
IDはWeb記事参照用に変更（00112233445566778899AABBCCDDEEFF）しています。

```elixir
# Set B-route Authentication ID（送配電会社から通知されたスマートメータ(電力計)のID 32文字）
Circuits.UART.write(pid, "SKSETRBID 00112233445566778899AABBCCDDEEFF\r\n")
Circuits.UART.read(pid)
```

SKSETRBIDコマンドのエコーバックとOKを確認できました。
```elixir
{:ok, "SKSETRBID 00112233445566778899AABBCCDDEEFF\r\nOK\r\n"}
```

### SKSCANコマンドの確認

```elixir
# Scan start to detect SmartMeter
Circuits.UART.write(pid, "SKSCAN 2 FFFFFFFF 6 0 \r\n")
Circuits.UART.read(pid)
```

SKSCANコマンドのエコーバックとOKを確認します。
```elixir
{:ok, "OK\r\nSKSCAN 2 FFFFFFFF 6 0 \r\nOK\r\n"}
```

```elixir
Circuits.UART.read(pid)
```

```elixir
{:ok, "SKSCAN 2 FFFF"}
```

エコーバックとOKが読めていないのでもう一度、リードします。
```elixir
Circuits.UART.read(pid)
```

エコーバックとOKが確認できました。
```elixir
{:ok, "FFFF 6 0 \r\nOK\r\n"}
```

EVENT 20、EPANDESCを確認します。

```elixir
Circuits.UART.read(pid)
```

PANに関する情報を読めました。
```elixir
{:ok,
 " 20 FE80:0000:0000:0000:021D:1291:0004:E578 0\r\nEPANDESC\r\n  Channel:31\r\n  Channel Page:09\r\n  Pan ID:B5FB\r\n  Addr:0011223344556677\r\n  LQI:DD\r\n  Side:0\r\n  PairID:0194BFAD\r\n"}
```

読めたAddr（スマートメータのMacアドレス）はWeb記事参照用に変更（0011223344556677）しています。


EVENT 22が返ってくることを期待してリードします。
```elixir
Circuits.UART.read(pid)
```

EVENT 22が返ってくることを確認できました。
```elixir
{:ok, "EVENT 22 FE80:0000:0000:0000:021D:1291:0004:E578 0\r\n"}
```

### SKSREGコマンドでS2レジスタにCannelをセット

SKSREGコマンドでS2レジスタにCannel(EPANDESCで受信した31)をセットします。

```elixir
# pull out Channel and set it to S2 reg.
Circuits.UART.write(pid, "SKSREG S2 31\r\n")
Circuits.UART.read(pid)
```

SKSREGコマンドのエコーバック、OKを確認できました。
```elixir
{:ok, "SKSREG S2 31\r\nOK\r\n"}
```

### SKSREGコマンドでS3レジスタにPan IDをセット

SKSREGコマンドでS3レジスタにPan ID(同じくEPANDESCで受信したB5FB)をセットします。
```elixir
# pull out Pan ID and set it to S3 reg.
Circuits.UART.write(pid, "SKSREG S3 B5FB\r\n")
Circuits.UART.read(pid)
```

SKSREGコマンドのエコーバック、OKを確認できました。
```elixir
{:ok, "SKSREG S3 B5FB\r\nOK\r\n"}
```

### SKLL64コマンドを使用してIPv6アドレスに変換

EPANDESCで受信したスマートメータのアドレスをSKLL64コマンドを使用してIPv6アドレスに変換する
```elixir
# Convert MAC Address(64bit) to IPV6 address
Circuits.UART.write(pid, "SKLL64 0011223344556677\r\n")
Circuits.UART.read(pid)
```

IPV6 addressが取得できました。
```elixir
{:ok, "SKLL64 0011223344556677\r\nFE80:0000:0000:0000:C2F9:4500:4058:B5FB\r\n"}
```

### SKJOINコマンドでPANA接続シーケンスを開始する

IPv6アドレスのスマートメータにSKJOINコマンドでPANA接続シーケンスを開始します。
```elixir
# start to set up PANA Connection sequence
Circuits.UART.write(pid, "SKJOIN FE80:0000:0000:0000:C2F9:4500:4058:B5FB\r\n")
Circuits.UART.read(pid)
```

EVENT 25の接続完了通知を受信するまで待ちます。
```elixir
{:ok,
 "EVENT 21 FE80:0000:0000:0000:C2F9:4500:4058:B5FB 0 02\r\nEVENT 02 FE80:0000:0000:0000:C2F9:4500:4058:B5FB 0\r\nERXUDP FE80:0000:0000:0000:C2F9:4500:4058:B5FB FE80:0000:0000:0000:021D:1291:0004:E578 02CC 02CC 0011223344556677 0 0 0028 00000028C0000002136F0BFE2FDA83DC00060000000400000000000500030000000400000000000C\r\nEVENT 21 FE80:0000:0000:0000:C2F9:4500:4058:B5FB 0 00\r\nERXUDP FE80:0000:0000:0000:C2F9:4500:4058:B5FB FE80:0000:0000:0000:021D:1291:0004:E578 02CC 02CC 0011223344556677 0 0 0068 0000006880000002136F0BFE2FDA83DD000500000010000044AA92C21CDBB377DEBBEBC9B755B87D000200000038000001E300382F007835A9A40F167C39A7A5AB6ACB3F1070534D3030303030303939303231373030303030303030303030303031393442464144\r\nEVENT 21 FE80:0000:0000:0000:C2F9:4500:4058:B5FB 0 00\r\nERXUDP FE80:0000:0000:0000:C2F9:4500:4058:B5FB FE80:0000:0000:0000:021D:1291:0004:E578 02CC 02CC 0011223344556677 0 0 0054 0000005480000002136F0BFE2FDA83DE00020000003B000001E4003B2F807835A9A40F167C39A7A5AB6ACB3F1070A69CF318ABB2DA0AC088301193DE29E2000000006852F3CB9B3291068BF82BB6863F1D2B8100\r\nEVENT 21 FE80:0000:0000:0000:C2F9:4500:4058:B5FB 0 00\r\nERXUDP FE80:0000:0000:0000:C2F9:4500:4058:B5FB FE80:0000:0000:0000:021D:1291:0004:E578 02CC 02CC 0011223344556677 0 0 0058 00000058A0000002136F0BFE2FDA83DF000700000004000000000000000200000004000003E4000400040000000400000000160100080000000400000001518000010000001000006F2356B28BAD8AE081F64840CF5B5933\r\nEVENT 21 FE80:0000:0000:0000:C2F9:4500:4058:B5FB 0 00\r\nEVENT 25 FE80:0000:0000:0000:C2F9:4500:4058:B5FB 0\r\nERXUDP FE80:0000:0000:0000:C2F9:4500:4058:B5FB FF02:0000:0000:0000:0000:0000:0000:0001 0E1A 0E1A 0011223344556677 1 0 0012 108100000EF0010EF0017301D50401028801\r\n"}
```

### Econet Lite DATA frameを定義する

Econet Lite DATA frameを定義します。

```elixir
defmodule SmartMeter do
  # Econet Lite DATAフレームの設定値
  @ehd1 0x10       # Econet Lite
  @ehd2 0x81       # EDATA format 1
  @tidh 0x52       # Transaction ID H "R"
  @tidl 0x53       # Transaction ID L "S"
  @seoj_x1 0x05    # Source EOJ Class Group Code
  @seoj_x2 0xFF    # Source EOJ Class Code
  @seoj_x3 0x01    # Source EOJ Instance Code
  @deoj_x1 0x02    # Destination EOJ Class Group Code
  @deoj_x2 0x88    # Destination EOJ Class Code
  @deoj_x3 0x01    # Destination EOJ Instance Code
  @esv_req 0x62    # Econet Lite Service Code
  @opc_req 0x01    # Number of property counter be red out.
  @epc_req 0xE7    # Econet Property Counter name.
  @pdc_req 0x00    # Property Data Byte count.

  # REQコマンドのバイト列を生成
  @req_cmd [
    @ehd1, @ehd2, @tidh, @tidl, @seoj_x1, @seoj_x2, @seoj_x3,
    @deoj_x1, @deoj_x2, @deoj_x3, @esv_req, @opc_req, @epc_req, @pdc_req
  ]

  def get_req_cmd() do
    @req_cmd
  end
end
```

コマンドのバイト列が正しいか確認します。
```elixir
SmartMeter.get_req_cmd()
```

コマンドのバイト列が正しいことを確認しました。
```elixir
[16, 129, 82, 83, 5, 255, 1, 2, 136, 1, 98, 1, 231, 0]
```

### スマートメータに電文をおくる
瞬間消費電力値（W単位）をリクエストする電文を送ります。

```elixir
# Send Data Request Command
Circuits.UART.write(
  pid,
  "SKSENDTO 1 FE80:0000:0000:0000:C2F9:4500:4058:B5FB 0E1A 1 0 000E "
)

Circuits.UART.write(pid, SmartMeter.get_req_cmd())

Circuits.UART.write(pid, "SKSENDTO")
```

スマートメータからの応答を読みます。

```elixir
Circuits.UART.read(pid)
```

```elixir
{:ok,
 "ERXUDP FE80:0000:0000:0000:C2F9:4500:4058:B5FB FE80:0000:0000:0000:021D:1291:0004:E578 0E1A 0E1A 0011223344556677 1 0 0026 1081000102880105FF017302EA0B07E70C13001E000000458EEB0B07E70C13001E0000000015\r\nSKSENDTO 1 FE80:0000:0000:0000:C2F9:4500:4058:B5FB 0E1A 1 0 000E \r\nEVENT 21 FE80:0000:0000:0000:C2F9:4500:4058:B5FB 0 00\r\nOK\r\nSKSENDTOERXUDP FE80:0000:0000:0000:C2F9:4500:4058:B5FB FE80:0000:0000:0000:021D:1291:0004:E578 0E1A 0E1A 0011223344556677 1 0 0012 1081525302880105FF017201E704000000E4\r\n"}
 ```

応答の電文が長いので見やすくするためにスペースで改行します。

```elixir
{:ok,
 "ERXUDP FE80:0000:0000:0000:C2F9:4500:4058:B5FB 
 FE80:0000:0000:0000:021D:1291:0004:E578 
 0E1A 
 0E1A 
 0011223344556677 
 1 
 0 
 0026 
 1081000102880105FF017302EA0B07E70C13001E000000458EEB0B07E70C13001E0000000015\r\n
 
 SKSENDTO 1 FE80:0000:0000:0000:C2F9:4500:4058:B5FB 0E1A 1 0 000E \r\n
 
 EVENT 21 FE80:0000:0000:0000:C2F9:4500:4058:B5FB 0 00\r\n
 
 OK\r\n
 
 SKSENDTOERXUDP 
 FE80:0000:0000:0000:C2F9:4500:4058:B5FB 
 FE80:0000:0000:0000:021D:1291:0004:E578 
 0E1A 
 0E1A 
 0011223344556677 
 1 
 0 
 0012 
 1081525302880105FF017201E704000000E4\r\n"}
 ```

# 応答データを解析

* Web記事 [スマートメータからWi-SUN Bルートで電力量を知る（その２）](https://www.ratoc-e2estore.com/blog/2023/06/wsuha-02)
* [Pythonサンプルスクリプト（今回の移植対象コード）](https://www.ratoc-e2estore.com/blog/wsuha_broute_demo_01)

を参照し、瞬間消費電力値が応答データの可能性があるデータはつぎのデータと当たりをつけました。

この応答データの最終行の

```elixir
 SKSENDTOERXUDP 
 FE80:0000:0000:0000:C2F9:4500:4058:B5FB 
 FE80:0000:0000:0000:021D:1291:0004:E578 
 0E1A 
 0E1A 
 0011223344556677 
 1 
 0 
 0012 
 1081525302880105FF017201E704000000E4\r\n"}
 ```

この部分です。

```elixir
 1081525302880105FF017201E704000000E4\r\n"}
 ```

受診した応答データを電文の仕様に当てはめていくと瞬間消費電力は**228W**という結果になりました。

![reply_analysis.jpg](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/171866/61ebf130-cf8b-3791-ac24-a3bd3cecd204.jpeg)


上記確認日の翌日に同じ場所でMacBook, Wi-SUN USBアダプター, Pythonコードで確認したところ下図の結果のように252Wとなりました。

![mac_python_execute.jpg](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/171866/affa6aa7-7d8d-9175-2305-73f974b5731e.jpeg)

Python版もNerves版と同じ200W台でした。Nerves版も期待とおり動作していそうです。


# 感想
Livebookで細かく動作確認をすすめていき、瞬間消費電力を取得することができました。
Livebookで動作確認できた後にロジックをコード化することでバグを生み出さずに、出戻りすくなく開発できるかもしれないと思いました。
このようにハードウェアの動作を細かくひとつひとつ確認していくときにLivebookは重宝するな、と感じました。
