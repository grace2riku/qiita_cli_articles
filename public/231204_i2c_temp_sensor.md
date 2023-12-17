---
title: C言語のi2c温度センサ制御コードをNerves・Elixirで書き直した話
tags:
  - Elixir
  - Nerves
private: false
updated_at: '2023-12-04T00:10:22+09:00'
id: b78bdcc53c0c7d88a4eb
organization_url_name: persolcrosstechnology
slide: false
ignorePublish: false
---
この記事は[#NervesJP Advent Calendar 2023](https://qiita.com/advent-calendar/2023/nervesjp)の4日目（12/4分）です。

# 概要
C言語で実装された温度センサーの温度取得制御コードをElixirで書いてみる、という記事です。
つぎのような方になにか参考になれば嬉しいです。

* C言語わかるけどNerves, Elixirでの実装がちょっとわからない
* Nervse, Elixirでセンサーからデータを取得するなど低レイヤの実装について興味のある方
* その他、Nervse, Elixirに興味のある方

# Nerves, Elixirのコードアップロード先
Nerves, Elixirのコードはつぎのリポジトリにアップロードしています。

https://github.com/grace2riku/nerves_laboratory

このリポジトリの**thermometers_ds1631**ディレクトリがNervesのプロジェクトです。


# C言語の実装について
C言語版の動作確認環境について書きます。

## コード

```c:main.cpp
/* 
* 温度取得
* 制御マイコン: NXP LPC1768
* コンパイラ: mbed
* 温度センサ: Maxim DS1631
*/ 
#include "mbed.h"

Serial MySerial(USBTX, USBRX);  // 9600bps, データ8ビット, パリティなし, ストップビット1ビット
I2C i2c(P0_0, P0_1);            // SDA:P0_0・P9, SCK:P0_1・P10/SCK: 100kHz

int main() {
    
    MySerial.printf("I2C Test\r\n");
 
    const int addr = 0x90;
    char cmd[2];
    char data[2];
    while (1) {

        cmd[0] = 0x51;
        i2c.write(addr, cmd, 1);
 
        wait(0.5);

        cmd[0] = 0xAA;
        i2c.write(addr, cmd, 1);
        i2c.read(addr, data, 2);

 

        cmd[0] = 0x22;
        i2c.write(addr, cmd, 1);


        float tmp = (float((data[0]<<8)|data[1]) / 256.0);
        MySerial.printf("Temp = %.2f\n\r", tmp);
        wait(0.5);
    }
}
```

## 対象ハードウェア
NXP LPC1768というマイコンで温度センサから温度を取得します。

https://os.mbed.com/platforms/mbed-LPC1768/

## 対象温度センサ
DS1631という温度センサが対象です。

https://www.analog.com/media/en/technical-documentation/data-sheets/DS1631-DS1731.pdf

※A0, A1, A2ピンは全てGNDに接続して使います。


## コンパイラ
mbedでコンパイルし動作確認していました。

# Nerves, Elixirの確認環境
Nerves, Elixirの確認環境はつぎのとおりです。

## Nerves, Elixirのバージョン
mix nerves.infoコマンドの実行結果の転記です。

```
$ mix nerves.info

Nerves:           1.10.3
Nerves Bootstrap: 1.11.5
Elixir:           1.15.5
```

こちらにNervesのChangelogが書いてあります。

https://hexdocs.pm/nerves/changelog.html

確認バージョンに使用したNervesのバージョンは**1.10.3**で最新版の一つ前のバージョンでした。

## ハードウェア
ラズパイ4で確認しました。


# Nerves, Elixirの実装
Nerves, Elixir版のコードです。
追加したコードはつぎのファイルの部分です。

* nerves_laboratory/thermometers_ds1631/lib/thermometers_ds1631.ex のread_temperature関数
* read_temperature関数で使う定義（require Loggerの行からstop_convert_commandの行まで）

```elixir:thermometers_ds1631.ex
defmodule ThermometersDs1631 do
  require Logger
  alias Circuits.I2C
  @i2c_bus "i2c-1"
  @ds1631_i2c_addr 0x48
  @conversion_time 500

  @start_convert_command 0x51
  @read_temperature_command 0xAA
  @stop_convert_command 0x22

  def read_temperature do
    {:ok, ref} = I2C.open(@i2c_bus)

    I2C.write(ref, @ds1631_i2c_addr, <<@start_convert_command>>)
    Process.sleep(@conversion_time)
    I2C.write(ref, @ds1631_i2c_addr, <<@read_temperature_command>>)

    {:ok, <<raw_temperature::16>>} = I2C.read(ref, @ds1631_i2c_addr, 2)

    I2C.write(ref, @ds1631_i2c_addr, <<@stop_convert_command>>)
    I2C.close(ref)

    temperature = raw_temperature / 256.0
    Logger.info("Raw temperature = #{raw_temperature}, Temperature = #{temperature}")

    temperature
  end

end
```

# Nerves, Elixirの動作確認
動作確認の手順です。

## ビルド
thermometers_ds1631ディレクトリに移動しつぎのコマンドを実行します。

ターゲットはラズパイ4なので環境変数MIX_TARGETはrpi4に設定します。
```
export MIX_TARGET=rpi4
```

ビルドします。
```
mix firmware
```

ホストPCにマイクロSDカードを接続し、イメージをSDカードに書き込みます。
```
mix burn
```

## 配線
温度センサ（図の赤丸が温度センサDS1631）とラズパイ4はラズパイのHATシールドのI2Cコネクタ（図の赤四角）で接続しています。

![pi4-ds1631.jpg](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/171866/e5908866-0a3d-50f9-f3c0-a9d26a48b542.jpeg)


## ログイン
マイクロSDカードをラズパイ4に接続し、電源をONします。
つぎのコマンドを実行し、ラズパイ4にログインします。

```
ssh nerves.local
```

## ログ出力の有効化
つぎのコマンドでログ出力を有効にします。

```
iex(1)> RingLogger.attach
:ok
```

## 温度センサの接続確認
つぎのコマンドを実行し、温度センサの接続を確認します。
アドレス72 (0x48)が温度センサを示しています。

```
iex(3)> Circuits.I2C.detect_devices()
Devices on I2C bus "i2c-1":
 * 8  (0x8)
 * 56  (0x38)
 * 72  (0x48)
```


## 温度データの取得
つぎの関数を実行すると温度が取得できました。
温度取得の初回は異常な値（196.0）ですがこれはC言語版でも同じ挙動でした。

```
iex(6)> ThermometersDs1631.read_temperature

08:12:40.656 [info] Raw temperature = 50176, Temperature = 196.0
196.0

iex(7)> ThermometersDs1631.read_temperature

08:12:43.810 [info] Raw temperature = 5328, Temperature = 20.8125
20.8125
```

# Nerves, Elixir実装の感想
シンプルなコードなのでNerves, Elixir版とC言語版のコードを比較してみると理解できるかと思います。

私は動作確認しつぎの感想を持ちました。

* Nerves, Elixir版ではCircuits.I2Cのライブラリを使うと簡単にI2C通信ができる
* ビットのパターンマッチングが気持ちいい

## Circuits.I2Cのライブラリを使うと簡単にI2C通信ができたこと
今回はI2C通信でしたがElixir Circuitsでは他にもGPIO、SPI、UARTとデバイスを制御するライブラリがあります。

https://elixir-circuits.github.io/

これらのライブラリを使うことでデバイスの制御が簡単に行えると思います。


## ビットのパターンマッチング
ビットのパターンマッチングはつぎのコードになります。

```
{:ok, <<raw_temperature::16>>} = I2C.read(ref, @ds1631_i2c_addr, 2)
```

これは温度センサからI2Cで2byte（16ビット）リードした結果をraw_temperatureにセットしています。
このような書き方で任意のビット数を取り出すことが可能です。
この機能は低レイヤのデバイス制御コードを実装するときにとても役に立つと思います。
個人的にこの機能が大好きです。

