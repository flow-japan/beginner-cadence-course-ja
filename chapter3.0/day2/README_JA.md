# 第 3 章 2 日目 - ディクショナリと配列の中にあるリソース

皆さん、こんにちは〜。今日は、第 2 章で学んだリソースを、配列とディクショナリといっしょに扱います。この 2 つは単体では簡単に扱えますが、組み合わせるとちょっと複雑なことになります。

## 動画

この動画の 8:00 - 最後まで見てください（冒頭部分については最終日にカバーします）: https://www.youtube.com/watch?v=SGa2mnDFafc

## なぜディクショナリと配列？

まず最初に、ディクショナリ内のリソースについて説明しますが、構造体の中のリソースについて説明しないのはなぜでしょう。まあ、最初に言っておくと、_構造体の中にリソースを格納することはできない_ ということです。構造体はデータの入れ物ですが、その中にリソースを入れることはできないのです。

では、どこにリソースを格納できるのでしょうか？

1. ディクショナリや配列の内部
2. 他のリソースの内部
3. コントラクトのステート変数として
4. アカウント・ストレージの内部（これについては後述します）

これで全部です。本日は、1 について話します。

## 配列の中のリソース

例で学ぶのが一番なので、Flow Playground を開いて、3 章 1 日目で使ったコントラクトをデプロイしてみましょう。

```cadence
pub contract Test {

    pub resource Greeting {
        pub let message: String
        init() {
            self.message = "Hello, Mars!"
        }
    }

}
```

今のところ、`@Greeting`という型を持つリソースが 1 つあるだけです。いいですね！では、Greeting のリストを、配列に格納するステート変数をつくってみましょう。

```cadence
pub contract Test {

    pub var arrayOfGreetings: @[Greeting]

    pub resource Greeting {
        pub let message: String
        init() {
            self.message = "Hello, Mars!"
        }
    }

    init() {
        self.arrayOfGreetings <- []
    }

}
```

`arrayOfGreetings` の型に注目してください。`@[Greeting]` です。昨日、リソースは常に `@` という記号が前についていることを学びました。これは中にリソースがある配列型にも当てはまり、`@` を前につけることによって、Cadence にこれがリソースの配列であることを伝えなければなりません。また、`@` は括弧の中ではなく、外側に付けるようにしてください。

`[@Greeting]` - これは間違い

`@[Greeting]` - これは正しい

また、`init` 関数の内部では `=` ではなく、`<-` 演算子で初期化していることに注意してください。もう一度言いますが、リソースを扱うときには (配列やディクショナリ、あるいは単体であろうと) `<-` を使わなければなりません。

### 配列に追加する

素晴らしい！リソースの配列を作りました。では、リソースを配列に追加する方法を見てみましょう。

_注記：本日は、関数の引数としてリソースを渡します。つまり、リソースがどのように作られたかは気にせず、サンプルの関数を使って配列やディクショナリに追加する方法を紹介します。_

```cadence
pub contract Test {

    pub var arrayOfGreetings: @[Greeting]

    pub resource Greeting {
        pub let message: String
        init() {
            self.message = "Hello, Mars!"
        }
    }

    pub fun addGreeting(greeting: @Greeting) {
        self.arrayOfGreetings.append(<- greeting)
    }

    init() {
        self.arrayOfGreetings <- []
    }

}
```

この例では、 `@Greeting` 型を受け取り、 `append` 関数を使って配列に追加する新しい関数 `addGreeting` を追加しています。簡単そうでしょう？これは、普通に配列に追加するのと全く同じで、 `<-` 演算子を使ってリソースを配列に「移動」しているだけです。

### 配列から取り出す

さて、配列に追加しました。では、どのようにリソースを配列から取り出すのでしょうか？

```cadence
pub contract Test {

    pub var arrayOfGreetings: @[Greeting]

    pub resource Greeting {
        pub let message: String
        init() {
            self.message = "Hello, Mars!"
        }
    }

    pub fun addGreeting(greeting: @Greeting) {
        self.arrayOfGreetings.append(<- greeting)
    }

    pub fun removeGreeting(index: Int): @Greeting {
        return <- self.arrayOfGreetings.remove(at: index)
    }

    init() {
        self.arrayOfGreetings <- []
    }

}
```

もう一度言いますが、これはとても簡単なことです。通常の配列では、要素を取り出すために `remove` 関数を使います。リソースの場合も同じで、唯一の違いは `<-` を使ってリソースを配列から「移動」させることです。すごい！

## ディクショナリ内のリソース

ディクショナリの中のリソースは少し複雑です。その理由のひとつは、2 日目の 3 章でやったように、ディクショナリはその中の値にアクセスすると常にオプショナルを返すからです。このため、リソースの保存と取り出しがかなり難しくなっています。いずれにせよ、リソースは _最も一般的に_ ディクショナリに格納されるので、それがどのように行われるかを学ぶことが重要であると言えるでしょう。

この例では、似たようなコントラクトを使ってみましょう：

```cadence
pub contract Test {

    pub var dictionaryOfGreetings: @{String: Greeting}

    pub resource Greeting {
        pub let message: String
        init() {
            self.message = "Hello, Mars!"
        }
    }

    init() {
        self.dictionaryOfGreetings <- {}
    }

}
```

`message` と `@Greeting` リソースを対応させるディクショナリを用意します。ディクショナリの型に注目してください。`@{String: Greeting}` です。この `@` は中括弧の外側にあります。

### ディクショナリに追加する

ディクショナリにリソースを追加するには 2 種類の方法があります。両方みてみましょう。

#### #1 - 最も簡単だが、厳格な方法

リソースをディクショナリに追加する最も簡単な方法は、「強制移動」演算子 `<-!` を使って、次のように行います：

```cadence
pub contract Test {

    pub var dictionaryOfGreetings: @{String: Greeting}

    pub resource Greeting {
        pub let message: String
        init() {
            self.message = "Hello, Mars!"
        }
    }

    pub fun addGreeting(greeting: @Greeting) {
        let key = greeting.message
        self.dictionaryOfGreetings[key] <-! greeting
    }

    init() {
        self.dictionaryOfGreetings <- {}
    }

}
```

`addGreeting` 関数では、まず `greeting` の中にある `message` を、`key` として取得します。そして、`greeting` を、`dictionaryOfGreetings` の特定の `key` の場所に「強制的に移動」して、ディクショナリに追加しています。

強制移動の演算子 `<-!` は基本的に次のような意味です：「もし、移動先にすでに値がある場合は、panic になってプログラムを中断してください。そうでなければ、そこに入れなさい」。

#### #2 - 複雑だが、重複を処理する

リソースを辞書に移動させる 2 つ目の方法は、次のような二重の移動構文を使うことです：

```cadence
pub contract Test {

    pub var dictionaryOfGreetings: @{String: Greeting}

    pub resource Greeting {
        pub let message: String
        init() {
            self.message = "Hello, Mars!"
        }
    }

    pub fun addGreeting(greeting: @Greeting) {
        let key = greeting.message

        let oldGreeting <- self.dictionaryOfGreetings[key] <- greeting
        destroy oldGreeting
    }

    init() {
        self.dictionaryOfGreetings <- {}
    }

}
```

この例では、奇妙な 2 つの移動演算子があるのがわかると思います。これはどういうことでしょうか？ステップに分解して考えてみましょう。

1. `key` にある値を取り出し、 `oldGreeting` に移動する
2. `greeting` をその場所に移動する（`key` には何もマッピングされていないことがわかっている）
3. `oldGreeting` を破棄する

本質的には、この方法はより厄介で奇妙に見えますが、 **既にそこに値がある場合の処理を可能にします。** 上記のケースでは、単にリソースを破棄していますが、やろうと思えば他のこともできます。

### ディクショナリから削除する

以下は、ディクショナリからリソースを削除する方法です：

```cadence
pub contract Test {

    pub var dictionaryOfGreetings: @{String: Greeting}

    pub resource Greeting {
        pub let message: String
        init() {
            self.message = "Hello, Mars!"
        }
    }

    pub fun addGreeting(greeting: @Greeting) {
        let key = greeting.message

        let oldGreeting <- self.dictionaryOfGreetings[key] <- greeting
        destroy oldGreeting
    }

    pub fun removeGreeting(key: String): @Greeting {
        let greeting <- self.dictionaryOfGreetings.remove(key: key) ?? panic("greeting がみつかりません!")
        return <- greeting
    }

    init() {
        self.dictionaryOfGreetings <- {}
    }

}
```

「配列から削除する」のセクションで、私たちは `remove` 関数を呼び出す必要があったことを思い出してください。ディクショナリでは、要素へのアクセスはオプショナルなものを返すので、何らかの方法で「アンラップ」する必要があります。もし、これをそのまま書いていたら...

```cadence
pub fun removeGreeting(key: String): @Greeting {
    let greeting <- self.dictionaryOfGreetings.remove(key: key)
    return <- greeting
}
```

「Mismatched types. Expected `Test.Greeting`, got `Test.Greeting?`」というエラーが返されます。
これを修正するには、 `panic` を使うか、以下のように強制アンラップ演算子 `!` を使います。

```cadence
pub fun removeGreeting(key: String): @Greeting {
    let greeting <- self.dictionaryOfGreetings.remove(key: key) ?? panic("greeting がみつかりません!")
    // もしくは...
    // let greeting <- self.dictionaryOfGreetings.remove(key: key)!
    return <- greeting
}
```

## まとめ

今日はここまでです！ :D さて、皆さんは疑問に思っているかもしれません。「リソースを持つ配列/ディクショナリの要素に _アクセス_ して、それを使って何かをしたい場合はどうすればよいのでしょうか? それは可能ですが、まずリソースを配列/ディクショナリの外に出して、何かを行い、そしてまた戻す必要があります。明日は、リソースをどこにでも移動させることなく、リソースを使って何かをすることができるようになる「参照」について説明します。じゃあね！

## クエスト

今日のクエストでは、いくつかの小さなクエストの代わりに、1 つの大きなクエストがあります。

1. リソースの配列と、リソースのディクショナリ、という 2 つのステート変数を持った、独自のスマートコントラクトを書きなさい。それぞれに、取り出しと追加を行う関数を追加します。これらは、上記の例とは異なるものでなければなりません。
