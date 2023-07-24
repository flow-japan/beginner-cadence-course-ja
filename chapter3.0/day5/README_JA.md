# 第3章5日目 - アクセスコントロール

調子はどうだい！今日はアクセスコントロールについて学びます。学びましょう。

## ビデオ

今日のビデオを見ることを強く推奨します。アクセスコントロールは非常にわかりにくいものですから： https://www.youtube.com/watch?v=ly3rNs0xCRQ&t

## アクセスコントロールとアクセス修飾子の紹介

アクセスコントロールは Cadence の非常に強力な機能であり、Cadence を特別なものにしています。

アクセスコントロールは、スマートコントラクトのセキュリティを高めるために「アクセス修飾子」と呼ばれるものを使用する方法について説明しています。

以前は、すべてのレッスンで、変数や関数を `pub` キーワードを使って次のように宣言しました：

```cadence
pub let x: Bool

pub fun jacobIsAwesome(): Bool {
  return true // obviously
}
```

しかし、`pub` とは一体どういう意味なのでしょうか？なぜそこに置くのでしょうか？代わりにできることは他にあるのでしょうか？それが今日学ぶことです。

## アクセス修飾子

`pub` は Cadence では 「アクセス修飾子」と呼ばれるものです。アクセス修飾子は基本的にスマートコントラクトのセキュリティレベルです。しかし、他にも使えるものがたくさんあります。この図を見て、様々なアクセス修飾子があることを理解しましょう。

<img src="../images/access_modifiers.png" />

ここでは `var` の行だけに注目します。なぜなら、`let `は定数なので書き込みスコープがないからです。次のセクションを読む前に、ぜひビデオを見てください。

## 「スコープ」とは何か？

スコープとは、「もの」（変数、定数、フィールド、関数）にアクセスしたり、変更したり、呼び出したりできる範囲のことです。スコープには4種類あります。：

### 1. すべてのスコープ

これは、**どこからでも**アクセスできるということです。コントラクトの内部でも、トランザクションでも、スクリプトでも、どこからでも。

<img src="../images/allscope.PNG" />

### 2. カレント ＆ インナースコープ

これは、定義された場所とその内部からしかアクセスできないということです。

<img src="../images/currentandinner.PNG" />

### 3. コントラクトスコープを含む

これは、定義されているコントラクトの中であれば、どこにでもアクセスできるということです。

<img src="../images/contractscope.PNG" />

### 4. アカウントスコープ

これは、定義されたアカウント内のどこからでも自分のものにアクセスできるということを意味します。つまり、アカウント内にあるすべてのコントラクトにアクセスできます。覚えておいてください：1つのアカウントに複数のコントラクトをデプロイすることができます。

## アクセス修飾子に戻る

クールですね！異なる「スコープ」が何を意味するのか復習しました。もう一度図を見てみましょう。

<img src="../images/access_modifiers.png" />

これで、何を言っているのか理解しやすくなりました。それでは、アクセス修飾子をすべて一緒に見ていきましょう...

### pub(set)

`pub(set)` は変数、定数、フィールドにのみ適用されます。関数は public に設定**できません**。これは最も危険で、簡単にアクセスできる修飾子でもあります。

例.

```cadence
pub(set) var x: String
```

書き込みスコープ - **全てのスコープ**

読み取りスコープ - **全てのスコープ**

### pub/access(all)

`pub` は `access(all)` と同じものです。
これは pub(set) の次のレイヤーです。

例.

```cadence
pub var x: String
access(all) var y: String

pub fun testFuncOne() {}
access(all) fun testFuncTwo() {}
```

書き込みスコープ - カレント ＆ インナー

読み取りスコープ - **全てのスコープ**

### access(account)

`access(account)` は読み取りスコープがあるため、`pub` より少し制限的です。

例.

```cadence
access(account) var x: String

access(account) fun testFunc() {}
```

書き込みスコープ - カレント ＆ インナー

読み取りスコープ - アカウント内の全てのコントラクト

### access(contract)

`access(contract)` は読み取りスコープがあるため、`access(account)` より少し制限的です。

例.

```cadence
access(contract) var x: String

access(contract) fun testFunc() {}
```

書き込みスコープ - カレント ＆ インナー

読み取りスコープ - コントラクト内容？(Containing Contract)

### priv/access(self)

`priv` は `access(self)` と同じものです。これは最も制限的な（そして安全な）アクセス修飾子です。

例.

```cadence
priv var x: String
access(self) var y: String

priv fun testFuncOne() {}
access(self) fun testFuncTwo() {}
```

書き込みスコープ - カレント ＆ インナー

読み取りスコープ - カレント ＆ インナー

## 重要な注意事項

<img src="../images/pleasenote.jpeg" />

アクセス修飾子を見た後、非常に重要な区別をしなければなりません：**`priv` のようなアクセス修飾子が Cadence のコードでフィールドを読めなくしているとしても、ブロックチェーンを見てその情報を読めないということではありません。ブロックチェーン上のすべてのものは、その読み取りスコープに関係なくパブリックです。**アクセス修飾子は、あなたの Cadence コードの文脈で何が読み取り可能か/書き込み可能かを判断させるだけです。ブロックチェーンには決して個人情報を保存しないでください！

## まとめ

今日はアクセス修飾についてたくさん学びました。理解度を試すために、今日のクエストでは忙しい仕事をたくさんするつもりです。正直なところ、クエストをこなすことで最も多くを学べると信じています。

皆さん、第4章でお会いしましょう！<3

## クエスト

今日のクエストでは、コントラクトとスクリプトを見てみましょう。`SomeContract` で定義された4つの変数(a, b, c, d)と3つの関数(publicFunc , contractFunc , privateFunc)を見ていきます。それぞれの領域(1、2、3、4) で以下のことをやってください：各変数(a, b, c, d)について、どの領域で読み取れるか(read scope)、どの領域で変更できるか(write scope)を教えてください。各関数（publicFunc ， contractFunc ， privateFunc）について、それがどこで呼び出せるかを教えてください。

この図を参考にしてください：
<img src="../images/boxdiagram.PNG" />

```cadence
access(all) contract SomeContract {
    pub var testStruct: SomeStruct

    pub struct SomeStruct {

        //
        // 4 Variables
        //

        pub(set) var a: String

        pub var b: String

        access(contract) var c: String

        access(self) var d: String

        //
        // 3 Functions
        //

        pub fun publicFunc() {}

        access(contract) fun contractFunc() {}

        access(self) fun privateFunc() {}


        pub fun structFunc() {
            /**************/
            /*** AREA 1 ***/
            /**************/
        }

        init() {
            self.a = "a"
            self.b = "b"
            self.c = "c"
            self.d = "d"
        }
    }

    pub resource SomeResource {
        pub var e: Int

        pub fun resourceFunc() {
            /**************/
            /*** AREA 2 ***/
            /**************/
        }

        init() {
            self.e = 17
        }
    }

    pub fun createSomeResource(): @SomeResource {
        return <- create SomeResource()
    }

    pub fun questsAreFun() {
        /**************/
        /*** AREA 3 ****/
        /**************/
    }

    init() {
        self.testStruct = SomeStruct()
    }
}
```

これは上記のコントラクトをインポートするスクリプトです：

```cadence
import SomeContract from 0x01

pub fun main() {
  /**************/
  /*** AREA 4 ***/
  /**************/
}
```
