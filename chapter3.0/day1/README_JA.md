# 第 3 章 1 日目 - リソース

おっとー。Cadence の中で最も重要なトピック、それはリソースです。まじめな話、これはあなたが私から学ぶ最も重要なことなのです。さぁ、始めましょう！

## 動画

1. この動画の 00:00 - 08:00 をみてください（残りは、後のほうで取り上げます）: https://www.youtube.com/watch?v=SGa2mnDFafc

## リソース

<img src="../images/resources.jpeg" alt="drawing" width="500" />

リソースは Cadence の最も重要な要素であり、Cadence が非常にユニークである理由でもあります。その見た目からして、**リソースはより安全な構造** というのが簡単な表現です。しかしもっと重要なのは、その安全性ゆえに、これから述べるような興味深い方法で使われることです。

コードを見るのはいつも役に立ちます。まずやってみましょう。

It's always helpful to look at code, so let's do that first:

```cadence
pub resource Greeting {
    pub let message: String
    init() {
        self.message = "Hello, Mars!"
    }
}
```

これは構造体とよく似ていると思いませんか？コードで見ると、実はかなり似ているのです。ここでは、リソース `Greeting` は、`String` 型のメッセージを格納する入れ物です。しかし、裏側では多くの違いがあります。

### リソース vs 構造体

Cadence では、構造体は単なるデータの入れ物です。好きなときにコピーしたり、上書きしたり、作成したりできます。これらのことはすべて、リソースの場合ではまったく異なります。ここでは、リソースを定義するいくつかの重要なことを説明します。

1. コピーできない
2. 紛失（または上書き）できない
3. いつでも好きなときに作成できるわけではない
4. リソースをどのように扱うか（例えば、移動させるなど）、明確である必要がある
5. リソースを扱うのはずっと難しい

リソースを理解するために、以下のコードを見てみましょう。

```cadence
pub contract Test {

    pub resource Greeting {
        pub let message: String
        init() {
            self.message = "Hello, Mars!"
        }
    }

    pub fun createGreeting(): @Greeting {
        let myGreeting <- create Greeting()
        return <- myGreeting
    }
}
```

ここでは、重要なことがたくさん起こっているので、順を追って見ていこうと思います。

1. `message` フィールドを含む `Greeting` というリソース型を初期化します。これはもうお分かりですね。
2. `Greeting` リソースを返す `createGreeting` という名前の関数を定義します。Cadence のリソースは、型の前に `@` 記号を使って「これはリソースです」と示します。
3. `create` キーワードで新しい `Greeting` 型を作成し、`<-` 「移動演算子」を使って `myGreeting` にアサインします。Cadence では、`=` を使ってリソースをどこかに置くことはできません。リソースを明確に「移動」させるには、必ず `<-` 「移動演算子」を使わなければなりません。
4. リソースを再び戻り値に移動させて、新しい `Greeting` を返します。

さて、これはクールです。しかし、もしリソースを破壊したい場合はどうすればいいのでしょうか？まあ、それはかなり簡単にできます。

```cadence
pub contract Test {

    pub resource Greeting {
        pub let message: String
        init() {
            self.message = "Hello, Mars!"
        }
    }

    pub fun makeAndDestroy() {
        let myGreeting <- create Greeting()
        destroy myGreeting
        // 注記: リソースの場所を変更するのに
        // `<-` 演算子を使わないのはこのときだけです
    }
}
```

リソースが構造体と大きく異なることは、すでにお分かりかと思います。リソースをどのように扱うかについて、より明確に伝える必要があるのです。では、リソースでできないことをいくつか見てみましょう。

```cadence
pub contract Test {

    pub resource Greeting {
        pub let message: String
        init() {
            self.message = "Hello, Mars!"
        }
    }

    pub fun createGreeting(): @Greeting {
        let myGreeting <- create Greeting()

        /*
            myGreeting <- create Greeting()

            上記のようなことはできません。
            これは myGreeting 変数を「上書き」してしまい、
            すでに格納されている以前のリソースを事実上失ってしまうことになります。
        */

        /*
            let copiedMyGreeting = myGreeting

            上記のようなことはできません。
            これは myGreeting リソースを「コピー」しようとするもので、許されません。
            リソースは決してコピーできません。もし myGreeting を copiedMyGreeting の中に移動させたいなら、次のようにします。

            let copiedMyGreeting <- myGreeting

            このようにすると、myGreeting はもう中に何も保存されていないので、
            使うことはできなくなります。
        */

        /*
            return myGreeting

            上記のようなことはできません。以下のように、
            <- 演算子を使って明確にリソースを「移動」させる必要があります。
        */
        return <- myGreeting
    }
}
```

では、なぜこれが便利なの？これは、超ムカつくだけじゃないの？いいえ、そうではありません。実はこれ、超便利なんです。誰かに何十億ドルもの NFT を渡すとします。その NFT を失わないようにしたいとは思いませんか？本当にそんなことができるのでしょうか？はい、Cadence では、「破壊せよ」という命令をしない限り、リソースを失うことはありません。これは Cadence の全体的なテーマでもあります。**Cadence は開発者が失敗することを難しくします**。これは良いことです。

両者の違いをまとめると、以下のようになります。

- 構造体は、データの入れ物です。それだけです。
- リソースは非常に安全で、紛失しにくく、コピーも不可能で、紛失することのないデータの入れ物として、しっかりと管理されています。

## コーディングの注意点

実際にコーディングするときのために、覚えておきたい注意点をいくつか紹介します。

- 新しいリソースを作成するには、 `create` キーワードを使います。create` キーワードはコントラクトの中でしか使えません。つまり、開発者であるあなたは、リソースがいつ作られるかをコントロールできます。構造体はコントラクトの外側で作成できるので、これは構造体には当てはまりません。
- リソース型の前には `@` 記号を使う必要があります。例えば、 `@Greeting` のようにです。
- リソースを移動させるには、 `<-` 記号を使います。
- キーワード `destroy` はリソースを破壊するために使います。

## そんなに怖くなかった？

やあ、やったね! そんなに悪くなかったでしょ？皆さんはうまくやれると思います。今日はここまでにして、明日からはもっと難しくしましょう。冗談です。 ;)

## クエスト

いつものように、あなたの好きな言語で自由に答えてください。

1. 構造体がリソースと異なる理由を 3 つ挙げてください。

2. 構造体よりもリソースを使った方が良い状況を説明してください。

3. 新しいリソースを作成するためのキーワードは何ですか？

4. リソースを、スクリプトやトランザクションで作成できますか？（作成するためのパブリック関数がないと仮定して）

5. 下のリソースの型は何ですか？

```cadence
pub resource Jacob {

}
```

6. 間違い探しゲームをしましょう。このコードには 4 つの間違いがあります。修正してください。

```cadence
pub contract Test {

    // ヒント: ここは何も間違ってません ;)
    pub resource Jacob {
        pub let rocks: Bool
        init() {
            self.rocks = true
        }
    }

    pub fun createJacob(): Jacob { // ここに 1 つ間違いがあります
        let myJacob = Jacob() // ここに 2 つ間違いがあります
        return myJacob // ここに 1 つ間違いがあります
    }
}
```
