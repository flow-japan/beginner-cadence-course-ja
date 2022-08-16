# 第 4 章 1 日目 - アカウントストレージ

いやはや、ようやく3章が終わりました。でも、まだまだこれからです;) やっていきましょう。

## 動画

この動画の 14:45 までを見てください: [https://www.youtube.com/watch?v=01zvWVoDKmU](https://www.youtube.com/watch?v=01zvWVoDKmU)

残りは明日解説します。

## Flowのアカウント

第2章1日目のトランザクションについて学んだとき、Flowのアカウントと、それがどのようにデータを保存できるかお話ししたと思います。一応復習のため、以下にコピー＆ペーストしておきます。

Flowでは、アカウントは自分自身のデータを保存することができます。これがどういうことかと言うと、FlowでNFTを所有すると、そのNFTは私のアカウントに保存されるということです。これは、Ethereumのような他のブロックチェーンとは*非常に異なる*ものです。Ethereumでは、NFTはスマートコントラクトに格納されます。Flowでは、実際にアカウント自身が自分のデータを保存することを可能にしており、これはとてもクールです。しかし、そのアカウントのデータにはどうやってアクセスするのでしょうか？それは `AuthAccount` 型で実現できます。あなたや私のようなユーザーがトランザクションを送信するたびに、トランザクションの代金を支払って、それに「署名」する必要があります。つまり、「このトランザクションを承認します」というボタンをクリックするだけです。あなたが署名すると、そのトランザクションはあなたの `AuthAccount` を取り込み、あなたのアカウントのデータにアクセスできるようになります。

これはトランザクションの `prepare` 部分で行われているのがわかります。これが `prepare` フェーズの要点で、あなたのアカウント内のデータにアクセスするために必要なフェーズです。一方、`execute`フェーズでは、アカウント内のデータにはアクセスできません。しかし、関数を呼び出して、ブロックチェーン上のデータを変更することは可能です。注：実際には、`execute`フェーズは必ずしも必要ではありません。技術的には `prepare` フェーズですべての処理を行うこともできますが、その場合、コードが分かりにくくなります。ロジックを分離する方が良いのです。
 
## アカウント内には何があるのか?

<img src="../images/accountstorage1.PNG" />

上で読んだように、Flowでは、アカウントは実際に自分自身のデータを保存します。つまり、`NFT`リソースがあれば、それを自分のアカウントに格納することができるのです。しかし、実際にどこに保存するのでしょうか？

上の図を使って、アカウントに何が格納されているのかを説明しましょう。
1. コントラクトコード - コントラクトはアカウントにデプロイされ、アカウント内に格納されます。複数のコントラクトを一つのアカウントにデプロイすることが可能です。
2. アカウントストレージ - あなたのデータはすべてアカウントストレージに保存されます。

## アカウントストレージ

では、アカウントストレージとは何でしょうか？アカウントストレージは、特定のパスに存在するデータの「容れ物」だと考えることができます。`/storage/`という特定のパスに存在します。Flowのアカウントでは、特定のデータにアクセスするための3つの方法があります。
1. `/storage/` - アカウントの所有者だけがアクセスできます（他者にデータを盗まれないための仕様です）。あなたのデータはすべてここにあります。
2. `/public/` - 誰でもアクセス可能です。
3. `/private/` - アカウントの所有者と、その所有者がアクセスを許可した人だけが利用可能です。

重要なのは、アカウント所有者だけが自分の `/storage/` にアクセスすることができますが、必要に応じて `/public/` や `/private/` にデータを格納することができるということです。例えば、私が自分のNFTを簡単に見せたい場合、私のNFTの読み取り可能なバージョンを `/public/` パスに置き、あなたが閲覧できるようにしますが、私のアカウントからは引き出すことができないように十分な制限をかけることができます。

*ヒント：リソースインターフェースがどのように役に立つのでしょうか？;)*

あなたは不思議に思っているかもしれません。「どうやって `/storage/` にアクセスすればいいんだろう？」 その答えは `AuthAccount` 型です。トランザクションに署名するとき、署名者の `AuthAccount` は `prepare` フェーズで以下のようにパラメータとして置かれます。

```cadence
transaction() {
  prepare(signer: AuthAccount) {
    // ここで署名者の /storage/ パスにアクセスできます!
  }

  execute {

  }
}
```

上で見たように、`prepare`の段階で署名者の `/storage/` にアクセスすることができます。これはつまり、彼らのアカウントでやりたいことが何でもできるということです。そのため、誤ってトランザクションに署名してしまうことがどれほど恐ろしいことかわかります。 皆さん、気をつけてください。

## リソースを保存して取り出す

では、アカウントに何かを保存してみましょう。まずはコントラクトを定義します。

```cadence
pub contract Stuff {

  pub resource Test {
    pub var name: String
    init() {
      self.name = "Jacob"
    }
  }

  pub fun createTest(): @Test {
    return <- create Test()
  }

}
```

リソース型 `@Test` を作成し、返却する簡単なコントラクトを定義しました。これをトランザクションで取得してみましょう。

```cadence
import Stuff from 0x01
transaction() {
  prepare(signer: AuthAccount) {
    let testResource <- Stuff.createTest()
    destroy testResource
  }

  execute {

  }
}
```

やっていることは `@Test` を作成し、破棄することだけです。次は、それを自分のアカウントに保存するにはどうすればいいか見ていきましょう。

```cadence
import Stuff from 0x01
transaction() {
  prepare(signer: AuthAccount) {
    let testResource <- Stuff.createTest()
    signer.save(<- testResource, to: /storage/MyTestResource) 
    // 自分のアカウントの /storage/MyTestResource パスに`testResource`を保存しています
  }

  execute {

  }
}
```

どのようにアカウントに保存したかを見てみましょう。まず、 **保存先の `AuthAccount` が必要です。** この場合、 `signer` という変数を持っています。それから、 `signer.save(...)` とします。これは、 `/storage/` というパスに何かを保存することを意味します。

`.save()` は2つのパラメータを受け取ります。
1. 保存する実際のデータ
2. 保存先のパスを指定する `to` パラメータ (`/storage/` 以下のパスである必要があります)

上の例では、 `testResource` (リソースなので `<-` 構文に注意) をパス `/storage/MyTestResource` に保存しています。これで、`/storage/MyTestResource`パスにアクセスすることでいつでも`testResource`を取得できます。やってみましょう。


```cadence
import Stuff from 0x01
transaction() {
  prepare(signer: AuthAccount) {
    let testResource <- signer.load<@Stuff.Test>(from: /storage/MyTestResource)
    // 自分のアカウントストレージから`testResource`を取り出しています

    destroy testResource
  }

  execute {

  }
}
```

上記の例では、`.load()`関数を使用して、アカウントストレージからデータを取り出しています。

このとき、`<@Stuff.Test>`という見慣れない構文が使用されていますね。これは何でしょうか？アカウントストレージとやりとりするときは、参照する型を指定する必要があります。Cadence は `@Stuff.Test` がそのストレージパスに保存されていることを知らないのです。しかし、コーダーとしては、それがそこに保存されていることは知っているので、「そのストレージパスから `@Stuff.Test` が出てくることを期待する」と表現するために `<@Stuff.Test>` と記述しなければならないのです。

`.load()` は1つのパラメータを受け取ります。
1. `from` パラメータは、取得するパスを指定します (`/storage/` 以下のパスでなければなりません)。

もう一つ重要なことは、ストレージからデータを `load` すると、optionalの値が返されることです。`testResource` は実際には `@Stuff.Test?` という型を持っています。この理由は、本当に正しい型のデータがそこに保存されているかわからないからです。そのため、もしあなたが間違っていた場合は `nil` を返します。例を見てみましょう。

```cadence
import Stuff from 0x01
transaction() {
  prepare(signer: AuthAccount) {
    let testResource <- signer.load<@Stuff.Test>(from: /storage/MyTestResource)
    log(testResource.name) // ERROR: "value of type `Stuff.Test?` has no member `name`."

    destroy testResource
  }

  execute {

  }
}
```

ご覧の通り、これはoptionalです。これを修正するには、 `panic` を使うか、 `!` 演算子を使うかのどちらかになります。エラーメッセージを指定できるので、私は `panic` を使うのが好きです。

```cadence
import Stuff from 0x01
transaction() {
  prepare(signer: AuthAccount) {
    let testResource <- signer.load<@Stuff.Test>(from: /storage/MyTestResource)
                          ?? panic("A `@Stuff.Test` resource does not live here.")
    log(testResource.name) // "Jacob"

    destroy testResource
  }

  execute {

  }
}
```

## Borrow関数

これまでに、アカウントからの保存と読み込みを行いました。しかし、あるアカウントに保存されたものを閲覧するだけならどうでしょうか？そこで、参照と `.borrow()` 関数の出番です。

```cadence 
import Stuff from 0x01
transaction() {
  prepare(signer: AuthAccount) {
    // NOTICE: This gets a `&Stuff.Test`, not a `@Stuff.Test`
    let testResource = signer.borrow<&Stuff.Test>(from: /storage/MyTestResource)
                          ?? panic("A `@Stuff.Test` resource does not live here.")
    log(testResource.name) // "Jacob"
  }

  execute {

  }
}
```

リソースそのものではなく、ストレージ内のリソースへの参照を取得するために `.borrow()` 関数を使用したことがお分かりになると思います。そのため、型は `<@Stuff.Test>` ではなく、 `<&Stuff.Test>` を使用しています。

`.borrow()` は 1 つのパラメータを受け取ります (`.load()` と同じです)。
1. `from` パラメータは、取得するパスを指定します。

また、`.load()` を使っていないので、リソースはずっとアカウントストレージの中にあることに注意してください。

## まとめ

もう一度この図を見てみましょう。

<img src="../images/accountstorage1.PNG" /> 

今のところ、`/storage/` が何であるかは理解しているはずです。明日の章では、 `/public/` と `/private/` のパスについて説明します。

## クエスト

1. アカウントの中身を説明してください。

2. `/storage/`、`/public/`、`/private/` のパスの違いは何ですか？

3. `.save()`、`.load()`、`.borrow()` が行うことは何ですか?

4. スクリプトの中で、なぜアカウントストレージに保存できないか説明してください。

5. なぜ私があなたのアカウントに何かしらのデータをを保存できないのか、説明してください。

6. 少なくとも1つのフィールドを持つリソースを返すコントラクトを定義してください。そして、以下の2つのトランザクションを書いてください。
 1) まずリソースをアカウントストレージに保存し、次にそれをアカウントストレージからロードし、リソース内のフィールドを記録し、それを破棄するトランザクション。
 2) 最初にリソースをアカウントストレージに保存し、そのリソースの参照を借用し、リソース内のフィールドを記録するトランザクション。
