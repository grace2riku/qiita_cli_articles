---
title: 設計の学習にChatGPTを大いに利用した
tags:
  - 設計
  - SOLID
  - ChatGPT
private: false
updated_at: '2023-12-24T00:46:48+09:00'
id: 12131749c6ad842db78b
organization_url_name: persolcrosstechnology
slide: false
ignorePublish: false
---
この記事は[設計こばなし Advent Calendar 2023](https://qiita.com/advent-calendar/2023/software_design_talk)の16日目です。

# 概要
2023年度、設計力アップのためSOLID原則を学び、社内・社外向け勉強会を開催・発信してきました。
勉強会の資料に掲載したソースコード、クラス図はChatGPTに作成してもらうこともありました。
設計の学習にどうChatGPTを利用したかを書きます。

# 勉強会について
勉強会の資料はSOLID原則に違反した場合、原則に則った場合の比較を書き説明していくスタイルにしました。
説明の際にソースコードも合わせて掲載しています。
資料、ソースコードの作成にはChatGPTの力を大いに借り、SOLID原則の理解が深まりました。


# ChatGPTの利用例
SOLID原則の学習に具体的にChatGPTをつぎのように使いました。

## 原則違反と原則に則ったコードの提示
SOLID原則を理解するには原則違反のコードと原則に則っているコードを比較して学ぶことが良いと思いました。
そこでChatGPTに原則違反と原則に則ったコードの提示を依頼しました。

### 単一責任の原則の例
単一責任の原則で原則違反と原則に則っているコードを提示してもらった例です。
こちらはChatGPT3.5で試した例です。

#### ChatGPT入力プロンプト
単一責任の原則の原則違反と原則に則っている例をC++のコードで教えてください。

#### ChatGPT回答
単一責任の原則 (Single Responsibility Principle, SRP) には、1つのクラスやモジュールは1つの責任を持つべきという原則があります。以下に、原則違反と原則に則った例をC++のコードで示します。

【原則違反の例】
```cpp
class Employee {
public:
    void calculateSalary() {
        // 給与計算のロジック
    }

    void saveToDatabase() {
        // データベースへの保存処理
    }

    void sendNotification() {
        // 通知の送信処理
    }
};
```
上記の例では、Employee クラスが給与計算、データベースへの保存、通知の送信といった複数の責任を持っています。これは単一責任の原則に違反しています。

【原則に則った例】
```cpp
class Employee {
public:
    void calculateSalary() {
        // 給与計算のロジック
    }
};

class Database {
public:
    void save(Employee employee) {
        // データベースへの保存処理
    }
};

class NotificationService {
public:
    void sendNotification(Employee employee) {
        // 通知の送信処理
    }
};
```

上記の例では、各クラスがそれぞれ独自の責任を持っています。

* Employee クラスは給与計算に関連した責任を持ちます。
* Database クラスはデータベースへの保存に関連した責任を持ちます。
* NotificationService クラスは通知の送信に関連した責任を持ちます。

このように、各クラスがそれぞれ1つの責任を担当するように分割することで、変更が発生した場合でも関連するコードの影響範囲を最小限に抑えることができます。

### インタフェース分離の原則の例
インタフェース分離の原則で原則違反と原則に則っているコードを提示してもらった例です。
こちらはChatGPT3.5で試した例です。

#### ChatGPT入力プロンプト
インタフェース分離の原則の原則違反と原則に則っている例をC++のコードで教えてください。

#### ChatGPT回答
インタフェース分離の原則 (Interface Segregation Principle, ISP) には、クライアントは自分が利用しないインタフェースに依存すべきではないという原則があります。以下に、原則違反と原則に則った例をC++のコードで示します。

【原則違反の例】
```cpp
class IWorker {
public:
    virtual void doWork() = 0;
    virtual void takeBreak() = 0;
    virtual void attendMeeting() = 0;
};

class Employee : public IWorker {
public:
    void doWork() override {
        // 仕事をする処理
    }

    void takeBreak() override {
        // 休憩を取る処理
    }

    void attendMeeting() override {
        // ミーティングに参加する処理
    }
};
```

上記の例では、IWorker インタフェースには3つのメソッドが含まれていますが、実際の実装である Employee クラスでは、全てのメソッドを実装しています。このような場合、クライアント（例えば、他のクラスやモジュール）は必要のないメソッドにも依存しなければなりません。これはインタフェース分離の原則に違反しています。

【原則に則った例】
```cpp
class IWorker {
public:
    virtual void doWork() = 0;
};

class IBreakTaker {
public:
    virtual void takeBreak() = 0;
};

class IMeetingAttendee {
public:
    virtual void attendMeeting() = 0;
};

class Employee : public IWorker, public IBreakTaker, public IMeetingAttendee {
public:
    void doWork() override {
        // 仕事をする処理
    }
    
    void takeBreak() override {
        // 休憩を取る処理
    }

    void attendMeeting() override {
        // ミーティングに参加する処理
    }
};
```
上記の例では、IWorker インタフェースを3つに分割し、それぞれ doWork()、takeBreak()、attendMeeting() の1つのメソッドを持つインタフェースに分けています。そして、Employee クラスは必要なインタフェースを実装することで、クライアントは自分が利用するインタフェースにだけ依存できるようになります。これにより、インタフェース分離の原則に則った設計になります。

### 感想
説明もわかりやすかったです。
コードも短く、シンプルでわかりやすかったです。
こんなにわかりやすい解説、コードを提示してくれるのか、と感動した記憶があります。
また、SOLID原則の学習に使えそう、と手応えも感じました。


## コードからクラス図作成
サンプルコードのクラス図を資料に載せたいことはたまにあると思います。
ChatGPTではサンプルコードを入力として対応するクラス図を作成してくれました。
つぎはC++コードからPlantUMLクラス図作成した例です。
こちらはChatGPT4で試した例です。

### ChatGPT入力プロンプト
つぎのコードをPlantUMLのクラス図のコードで提示してください。

```cpp
class Rectangle {
protected:
    int width, height;

public:
    Rectangle(const int width, const int height) : width(width), height(height) {}

    virtual void setWidth(const int w) {
        width = w;
    }

    virtual void setHeight(const int h) {
        height = h;
    }

    int getWidth() const {
        return width;
    }

    int getHeight() const {
        return height;
    }

    int area() const {
        return width * height;
    }
};

class Square : public Rectangle {
public:
    Square(int size) : Rectangle(size, size) {}

    void setWidth(const int w) override {
        width = height = w;
    }

    void setHeight(const int h) override {
        width = height = h;
    }
};
```

### ChatGPT回答
Based on the provided C++ code, here's the equivalent PlantUML class diagram code:

```
@startuml

class Rectangle {
    - int width
    - int height

    + Rectangle(int width, int height)
    + virtual void setWidth(int w)
    + virtual void setHeight(int h)
    + int getWidth()
    + int getHeight()
    + int area()
}

class Square {
    + Square(int size)
    + void setWidth(int w) override
    + void setHeight(int h) override
}

Square --|> Rectangle : extends

@enduml
```

This PlantUML code represents the class structure of the Rectangle and Square classes as defined in your C++ code. The Square class extends the Rectangle class, indicating inheritance, and both classes have their respective methods and attributes with appropriate visibility markers (- for protected, + for public). The override keyword is not represented in PlantUML, so the overriding methods in Square are simply defined without it.

### 出来上がったクラス図
下図はChatGPTが生成したPlantUMLをVSCodeで表示した場合です。
クラス図が表示できています。

![plantuml_class.jpg](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/171866/5447ba54-2317-abc9-5ff1-e1d56f0c8238.jpeg)


## SOLID原則に則っているかのチェックシート作成
ちょっと変わった要求をChatGPTにしてみました。
SOLID原則に則っているか確認できるチェックシートの作成依頼です。

### ChatGPT入力プロンプト
SOLID原則に則っているかコードかチェックできるチェックシートを提示してください。

### ChatGPT回答
SOLID原則はオブジェクト指向設計の5つの基本的なガイドラインのことを指します。それぞれの原則を確認するための簡単なチェックシートを以下に示します。

1. 単一責任の原則 (Single Responsibility Principle, SRP)

* [ ] そのクラスやメソッドは1つの責任のみを持っているか？
* [ ] 複数の理由で変更する必要があるクラスやメソッドは存在しないか？

2. オープン・クローズド原則 (Open/Closed Principle, OCP)

* [ ] 新しい機能を追加するとき、既存のコードの変更を最小限に抑えられる設計になっているか？
* [ ] 拡張には開いており、修正には閉じている状態になっているか？

3. リスコフの置換原則 (Liskov Substitution Principle, LSP)

* [ ] サブクラスをスーパクラスの位置で置換してもプログラムの正当性が保持されるか？
* [ ] サブクラスはスーパクラスの振る舞いや特性を破壊していないか？

4. インターフェース分離原則 (Interface Segregation Principle, ISP)

* [ ] インターフェースは特定のクライアントのために特化しているか？
* [ ] 一つのインターフェースに多くのメソッドが存在し、それによってそのインターフェースを実装するクラスが不要なメソッドを持たされていないか？

5. 依存性逆転の原則 (Dependency Inversion Principle, DIP)

* [ ] 上位のモジュールは下位のモジュールに直接依存していないか？
* [ ] 抽象に依存しており、具体的な実装には依存していないか？

このチェックシートを使用することで、SOLID原則に従って設計されているかを簡単に確認することができます。ただし、このチェックシートはガイドラインとして使うものであり、全ての具体的なシチュエーションにフィットするわけではありません。コードの文脈や要件に応じて適切な判断を行ってください。

# 感想
ChatGPTを利用することでSOLID原則の学習に非常に役立った、という感想です。
ひとつ注意しなくてはいけないなと思ったのは今回ChatGPTが提示したコードは説明用のコードで実際の開発現場の課題を解決するためのコードではないことです。
ChatGPTが提示してくれた解説やサンプルコードのエッセンスは参考にしながらも実際の開発現場で試行錯誤してみたいと思いました。

SOLID原則を学びはじめる際にChatGPTは十二分に役にたちました。
今後も設計を学ぶときのパートナーとしてChatGPTを使っていきたいと考えています。

# 作成資料一覧
SOLID原則の勉強会の資料はこちらに保存しています。よければ参照ください。

| No | 開催日 | 勉強会タイトル&資料リンク | Xスペース録音URL |
|:-----------|:------------|:------------|:------------|
| 1       | 2023/7/20        | [#1 単一責務の原則](https://www.docswell.com/s/juraruming/K98M19-2023-07-20-143150)         | -         |
| 2       | 2023/8/28        | [#2 インターフェース分離の原則](https://www.docswell.com/s/juraruming/KJLWEM-2023-08-28-005645)         | [URL](https://twitter.com/i/spaces/1lDGLnPowBZxm?s=20)         |
| 3       | 2023/9/28        | [#3 依存性逆転の原則](https://www.docswell.com/s/juraruming/5EN9ME-2023-09-28-065704)         | [URL](https://twitter.com/i/spaces/1YqKDoZgMvzxV)         |
| 4       | 2023/10/26        | [#4 開放閉鎖の原則](https://www.docswell.com/s/juraruming/5LL836-2023-10-26-074605)         | [URL](https://twitter.com/i/spaces/1nAKEaOpWzVKL)         |
| 5       | 2023/11/30        | [#5 リスコフの置換原則](https://www.docswell.com/s/juraruming/Z6Y8X7-2023-11-30-081051)         | [URL](https://twitter.com/i/spaces/1rmGPMWRdQdJN)         |
