# 第 3 章 3 日目 - 参照

Flow の皆さん、お元気ですか。今日は、Cadence プログラミング言語のもう一つの重要なこと、参照（Reference）についてお話します。

## 動画

参照についての動画をご覧になりたい方は、こちらへどうぞ: https://www.youtube.com/watch?v=mI3KC-5e81E

## 参照とは？

簡単に言うと、参照とは、実際にそのデータを持っていなくても、データの一部を操作できるようにする方法です。この機能は、リソースに対して非常に有効です。リソースのフィールドを見たり更新したりするために、リソースを 1,000 回も移動させる必要がない世界を想像してみてください。ああ、そんな世界があるんですね！参照は、このような状況を救ってくれるのです。

## Cadence における参照

Cadence では、参照はほとんど常に構造体かリソースで使用されます。文字列や数値、基本的なデータ型の参照を作成することはあまり意味がありません。ただし、何度も受け渡たしたくないものを参照するのは意味があります。

参照は常に `&` という記号を前につけて行います。例を見てみましょう:

```cadence
pub contract Test {

    pub var dictionaryOfGreetings: @{String: Greeting}

    pub resource Greeting {
        pub let language: String
        init(_language: String) {
            self.language = _language
        }
    }

    pub fun getReference(key: String): &Greeting? {
        return &self.dictionaryOfGreetings[key] as &Greeting?
    }

    init() {
        self.dictionaryOfGreetings <- {
            "Hello!": <- create Greeting(_language: "English"),
            "Bonjour!": <- create Greeting(_language: "French")
        }
    }
}
```

上の例では、 `getReference` が `&Greeting?` 型を返していることがわかります。これは、単に 「`@Greeting` 型への参照のオプショナル型」を意味しています。この関数の内部では、いくつかのことが起こっています。

1. `&self.dictionaryOfGreetings[key]` とすることで、まず `key` にある値の参照を取得します。
2. `as &Greeting?` とすることで、この参照を「型キャスト」します。これは、オプショナル型であることに気をつけてください。ディクショナリでインデックスを指定すると、オプショナル型が返ります。

もし、`as &Greeting?`を忘れていたら、Cadence は "expected casting expression" と言うエラーを出すことでしょう。これは、Cadence では、**参照を取得するときに型キャストする必要がある** からです。型キャストとは、Cadence に参照を取得するときの型を伝えることで、`as &Greeting?` でこれをやっています。これは「このオプショナル型の参照を、&Greeting 型への参照として取得しなさい」ということです。もし型が違う場合、プログラムは停止されます。

さて、「このオプショナルな参照をどうやってアンラップすればいいんだろう？」と疑問に思うかもしれません。それはこんな風にすればいいんです:

```cadence
pub fun getReference(key: String): &Greeting {
    return (&self.dictionaryOfGreetings[key] as &Greeting?)!
}
```

全体を括弧でくくって、通常の強制アンラップ演算子 `!` でアンラップしていることに注意してください。また、戻り値の型もオプショナル型ではない `&Greeting` に変更しています。あなたのコードで、これが変更されることを確認してください。

これで参照を取得できるようになったので、トランザクションやスクリプトで次のように参照を取得することができます:

```cadence
import Test from 0x01

pub fun main(): String {
  let ref = Test.getReference(key: "Hello!")
  return ref.language // returns "English"
}
```

これをやるために、リソースをどこかに移動させる必要はありません！これが参照の良いところです。

## まとめ

参照はそんなに悪いものではないでしょう？主なポイントは 2 つです。

1. リソースを移動させることなく、参照を使って情報を得られる。
2. 参照を取得するときは、必ず「型キャスト」しなければならない、さもなければエラーが発生する。

参照はなくなることはありません。これらは次の章でアカウント・ストレージについて説明する際、非常に重要になります。

## クエスト

1. リソースのディクショナリを保存する独自のコントラクトを定義してください。ディクショナリの中のリソース 1 つに対する参照を取得する関数を追加してください。

2. 1 で定義した関数からの参照を使って、そのリソースから情報を読み取るスクリプトを作成してください。

3. Cadence で参照が有用である理由を、自分の言葉で説明してください。
