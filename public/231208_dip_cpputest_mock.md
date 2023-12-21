---
title: SOLID 依存性逆転の原則を学習中に依存性注入を意識したCppUTestモックテスト環境をつくった
tags:
  - CppUTest
  - SOLID
  - 依存性注入
  - CppUMock
  - 依存性逆転の原則
private: false
updated_at: '2023-12-08T22:01:55+09:00'
id: cc711ae7eea0de194f07
organization_url_name: persolcrosstechnology
slide: false
ignorePublish: false
---
この記事は[設計こばなし Advent Calendar 2023](https://qiita.com/advent-calendar/2023/software_design_talk)の8日目（12/8分）です。

# 概要
今年度SOLID原則の勉強会を開催してきました。依存性逆転の原則を学ぶ過程でテストしやすくするCppUTestのモックをつくったのでその共有です。
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
依存性逆転の原則を学んでいると**依存性注入**も合わせて学ぶことが多いと思います。
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
| include/settingvalue_example/ISettingValue.h | 設定値のインターフェース。<br>SettingValueValidationに依存性注入する。 |
| tests/settingvalue_example/SettingValueMock.cpp | インターフェースの実装。<br>実機をつかわないでテストするときに使う想定。readメソッドをモックにする。 |
| tests/settingvalue_example/SettingValueExampleTest.cpp | テストのモジュール。<br>SettingValueValidationのvalidateメソッドのテストが書いてある。 |
| src/settingvalue_example/SettingValueRam.cpp | インターフェースの実装。<br>実機に組込む製品コード。 |

#### コード詳細
##### SettingValueValidation.cpp
設定値を検証する責務を持ったクラスです。
コンストラクタに設定値インターフェース（ISettingValue）を指定します。
依存性注入する具象クラス（SettingValueMockまたはSettingValueRam）を切り替えることにより、validateメソッドの設定値読み込み（read）は動きが変わります。
validateメソッドは具象クラスのreadメソッドを呼び出し、戻り値を100以上かチェックします。
判定自体に特に意味はありません。

```cpp:SettingValueValidation.cpp
#include "SettingValueValidation.h"
#include "ISettingValue.h"

#include <iostream>
using namespace std;

// コンストラクタの実装
SettingValueValidation::SettingValueValidation(ISettingValue* settingValue) {
    cout << "SettingValueValidation constructor" << endl;

    _settingValue = settingValue;
}

SettingValueValidation::~SettingValueValidation() {
    cout << "SettingValueValidation destructor" << endl;

    delete _settingValue;
}

bool SettingValueValidation::validate() {
    int value = _settingValue->read();
    if (value >= 100) return true;
    return false;
}
```

##### ISettingValue.h
設定値書き込み、読み込みのインタフェースです。
具象クラス（SettingValueMockまたはSettingValueRam）はこのインターフェースを実装します。

```cpp:ISettingValue.h
#ifndef _H_ISETTINGVALUE_
#define _H_ISETTINGVALUE_

#include <iostream>
using namespace std;

class ISettingValue {
    public:
        virtual void write() = 0;
        virtual int read() = 0;

        // 仮想デストラクタ
        virtual ~ISettingValue(){
            cout << "ISettingValue destructor" << endl;
        }
};

#endif	// _H_ISETTINGVALUE_
```

##### SettingValueMock.cpp
インターフェースを実装したモックです。
実機をつかわないでテストするときに使う想定です。
readメソッドの中でCppuMockのactualCallを呼び出し、int型の戻り値を返します。
戻り値はテストのモジュール（SettingValueExampleTest）でつぎのCppUMockの関数を使いセットしています。

```cpp
  mock().expectOneCall("read").andReturnValue(99);
```

または

```cpp
  mock().expectOneCall("read").andReturnValue(100);
```


```cpp:SettingValueMock.cpp
#include "SettingValueMock.h"

#include <iostream>
using namespace std;

#include "CppUTest/TestHarness.h"
#include "CppUTestExt/MockSupport.h"

// コンストラクタの実装
SettingValueMock::SettingValueMock() {
    cout << "SettingValueMock constructor" << endl;
}

SettingValueMock::~SettingValueMock() {
    cout << "SettingValueMock decstructor" << endl;
}

void SettingValueMock::write() {
}

int SettingValueMock::read() {
    return mock().actualCall("read").returnIntValue();
}
```

##### SettingValueExampleTest.cpp
テストのモジュールです。
setup関数でSettingValueMockをSettingValueValidationに依存性注入しインスタンスを生成しています。

```cpp:
    void setup()
    {
      settingValueValidation = new SettingValueValidation(new SettingValueMock());
    }
```

テストは2つ書いてあります。
1つ目はSettingValueinValidテストです。
CppUMockのexpectOneCall, andReturnValueでSettingValueMockのreadメソッドの戻り値を設定しています。
SettingValueinValidテストはSettingValueMockのreadメソッドの戻り値を99に設定してます。
結果、settingValueValidationクラスのvalidateメソッドはfalseを返すはずです。
CppUTestのCHECK_FALSEのアサーションで期待値falseをテストしています。

```cpp:
TEST(SettingValueExampleTest, SettingValueinValid)
{
  mock().expectOneCall("read").andReturnValue(99);

  CHECK_FALSE(settingValueValidation->validate());
}
```

2つ目はSettingValueValidテストです。
1つ目のテストと同様にCppUMockのexpectOneCall, andReturnValueでSettingValueMockのreadメソッドの戻り値を設定しています。
SettingValueValidテストはSettingValueMockのreadメソッドの戻り値を100に設定してます。
結果、settingValueValidationクラスのvalidateメソッドはtrueを返すはずです。
CppUTestのCHECKのアサーションで期待値trueをテストしています。

```cpp:
TEST(SettingValueExampleTest, SettingValueValid)
{
  mock().expectOneCall("read").andReturnValue(100);

  CHECK(settingValueValidation->validate());
}
```

```cpp:SettingValueExampleTest.cpp
#include "CppUTest/TestHarness.h"
#include "CppUTestExt/MockSupport.h"

#include <iostream>
using namespace std;
#include "SettingValueValidation.h"
#include "SettingValueMock.h"
#include "ISettingValue.h"

TEST_GROUP(SettingValueExampleTest)
{
    SettingValueValidation* settingValueValidation;

    void setup()
    {
      settingValueValidation = new SettingValueValidation(new SettingValueMock());
    }

    void teardown()
    {
      mock().clear();
      delete settingValueValidation;
    }
};

TEST(SettingValueExampleTest, SettingValueinValid)
{
  mock().expectOneCall("read").andReturnValue(99);

  CHECK_FALSE(settingValueValidation->validate());
}

TEST(SettingValueExampleTest, SettingValueValid)
{
  mock().expectOneCall("read").andReturnValue(100);

  CHECK(settingValueValidation->validate());
}
```


##### SettingValueRam.cpp
インターフェースを実装した具象クラスです。
このコードは実機に組込む製品コードを想定しています。
readメソッドは固定値123を返す実装ですが実際には装置から設定値を読み込むことを考えていました。

```cpp:SettingValueRam.cpp
#include "SettingValueRam.h"
#include "Boot.h"

#include <iostream>
using namespace std;

// コンストラクタの実装
SettingValueRam::SettingValueRam() {
    cout << "SettingValueRam constructor" << endl;
}

SettingValueRam::~SettingValueRam() {
    cout << "SettingValueRam decstructor" << endl;
}

void SettingValueRam::write() {
}

int SettingValueRam::read() {
    return 123;
}
```


## 動作確認結果
テストを実行すると期待とおりテストは成功することが確認できました。

```
$ ./bin/dip_and_di_test_easy_env -v
TEST(SettingValueExampleTest, SettingValueValid)
 - 0 ms
TEST(SettingValueExampleTest, SettingValueinValid)
 - 0 ms

OK (2 tests, 2 ran, 6 checks, 0 ignored, 0 filtered out, 1 ms)
```

# まとめ
モックが期待とおりの戻り値を設定し、SettingValueValidationクラスのvalidateメソッドのロジックを確認できました。
今回はモックの確認でしたが実機に組込む製品コードに切り替えたければSettingValueValidationクラスに依存性注入するクラスをSettingValueRamに変更すればOKです。

