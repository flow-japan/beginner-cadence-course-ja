# 第 2 章 2 日目 - トランザクションとスクリプト

クレイジーな Cadence ファンの皆さん、こんにちは。今日はトランザクションとスクリプトをより深く掘り下げていきます。まだお読みでない方は、ぜひ [第 1 章 1 日目 トランザクションとスクリプト](https://github.com/emerald-dao/beginner-cadence-course/tree/main/chapter1.0/day1#transactions--scripts) をお読みください。

## 動画

この（信じられないような）コンテンツを動画形式でみたい方は、こちらのビデオをご覧ください: [https://www.youtube.com/watch?v=T2QTTFnQa5k](https://www.youtube.com/watch?v=T2QTTFnQa5k)

## トランザクションとスクリプト

トランザクションとスクリプトは、いずれもブロックチェーンアプリケーションに不可欠なものです。それらがなければ、ブロックチェーンを使うことができません。Flow では、この 2 つがコントラクトから分離しているため、さらに特別なものとなっています。Ethereum でコーディングしたことがある人なら、トランザクションがコントラクト自体の内部で呼び出す関数にすぎないことを知っているでしょう（知らなくともそれで OK）。しかし Flow では、トランザクションとスクリプトは、ブロックチェーンを使う人とスマートコントラクトの「仲介役」として機能します。以下のイメージです：

<img src="../images/sctsworkflow.png" alt="drawing" size="400" />

## トランザクションとスクリプトの違い

トランザクションとスクリプトの違いは何でしょうか？最大の違いは、トランザクションはブロックチェーン上のデータを変更し、スクリプトはブロックチェーン上のデータを見るということです。この違いを理解するのに役立つ図がこちらです。

<img src="../images/transactionvscript.png" alt="図" size="400" />.

ご覧の通り、スクリプトはお金もかかりません（ふっふっふ）。一方、トランザクションには「ガス」がかかります。これはブロックチェーン上のデータを変更するために必要なコストの一種です。

## スクリプト

昨日、私たちは実際に Flow プレイグラウンドで最初のスクリプトを実装しました。その例を再確認してみましょう。

Flow プレイグラウンド（[https://play.onflow.org](https://play.onflow.org/)）をロードし、以下のコントラクトを `0x01` アカウントにコピーして、「Deploy」をクリックします。

```cadence
pub contract HelloWorld {

    pub let greeting: String

    init() {
        self.greeting = "Hello, World!"
    }
}
```

次に、左側の「Script」タブに、昨日のスクリプトをもう一度持ってきます：

```cadence
import HelloWorld from 0x01

pub fun main(): String {
    return HelloWorld.greeting
}
```

「Execute」をクリックすると、コンソールに「Hello, World!」と表示されるはずです。素晴らしいです。あなたは今、スクリプトを実行しました。ガス代を支払うことなく、スマートコントラクトのデータを閲覧できたことに注目してください。

## トランザクション

では、トランザクションを試してみましょう。左側の「Transaction Templates」の下にある「Transaction」タブをクリックします。そのタブの中のものをすべて削除すると、次のようになります。

<img src="../images/emptytx.PNG" alt="図面" size="400" />このようになります。

OK、クール！！さて、ブロックチェーン上のデータを変更していきましょう。そのために、トランザクションをセットアップしましょう。次のコードをページ内に記述します。

```cadence
transaction() {
    prepare(signer: AuthAccount) {}

    execute {}
}
```

やったー！! これで何もしない空のトランザクションが完成しました。`prepare` と `execute` の内容を説明するために、少し休憩をとってください。その後で Flow 上のアカウントについてご説明しましょう。

### Flow 上のアカウント

Flow では、アカウントは自身のデータを保存することができます。これは何を意味するのでしょうか？まあ、私が Flow で NFT（NonFungibleToken）を所有すると、その NFT は私のアカウントに格納されます。これは、Ethereum のような他のブロックチェーンとは _大きく異なります_。Ethereum では、あなたの NFT はスマートコントラクトに格納されます。Flow では、実際にアカウントが自分自身のデータを自分で保存できるようになっています。しかし、そのアカウントのデータにはどうやってアクセスするのでしょうか？それは `AuthAccount` 型で行うことができます。ユーザー（あなたや私のような）がトランザクションを送信するたびに、トランザクションの手数料を支払って、それに「署名」する必要があります。これは、あなたが「このトランザクションを承認します」というボタンをクリックしたことを意味します。あなたが署名すると、そのトランザクションはあなたの `AuthAccount` を使って、あなたのアカウントのデータにアクセスできるようになります。

これはトランザクションの `prepare` 部分で行われていることがわかるでしょう。これが `prepare` フェーズ の要点で、あなたのアカウントにある情報/データにアクセスするためなのです。一方、`execute` フェーズでは、そのようなことはできません。しかし、関数を呼び出して、ブロックチェーン上のデータを変更することは可能です。
注：実際には、`execute` フェーズは _必ず使わなければならないわけではありません_。技術的には `prepare` フェーズですべてを行うことも可能ですが、その場合、コードの見通しが悪くなります。ロジックを分離したほうが良いのです。

### 例に戻りましょう

さて、では `greeting` フィールドを「Hello, World!」以外のものに変更したいと思います。しかし、問題があります。スマートコントラクトにデータを変更する方法がないのです。そこで、そのための関数をコントラクトに追加しなければなりません。

アカウント `0x01` に戻り、コントラクト内にこの関数を追加してください。

```cadence
pub fun changeGreeting(newGreeting: String) {
    self.greeting = newGreeting
}
```

これはどういうことでしょうか。関数について説明したことを思い出してください。このように設定します。
`[access modifier] fun [function name](parameter1: Type, parameter2: Type, ...): [return type] {}`

ここでは、シンプルにするために、アクセス修飾子として `pub` を使います。`pub` は、この関数をどこからでも（コントラクト内でもトランザクション内でも）呼び出せることを意味します。また、文字列である `newGreeting` パラメータを受け取り、`greeting` に `newGreeting` をセットします。

しかし、ちょっと待ってください！ このコントラクトにはエラーがあります。「cannot assign to constant member: `greeting`.」とエラーが書かれているでしょう。なぜそんなことを言うのでしょう？私たちは `greeting` を `let` にしたのを覚えていますか？ `let` は定数なので、`greeting` を変更したい場合は、`var` に変更する必要があります。変更後、もう一度「Deploy」を押してください。こうするとあなたのコードは以下のようになるはずです。

```cadence
pub contract HelloWorld {

    pub var greeting: String

    pub fun changeGreeting(newGreeting: String) {
        self.greeting = newGreeting
    }

    init() {
        self.greeting = "Hello, World!"
    }
}
```

さて、コントラクトをセットアップしたので、トランザクションに戻りましょう。まず、HelloWorld コントラクトを `import` してください。`import HelloWorld from 0x01` のようにインポートします。それから、`changeGreeting` をどこで呼び出すかを決めなければなりません。`prepare` フェーズで呼び出すのか、それとも `execute` フェーズで呼び出すのか？答えは `execute` フェーズです。なぜなら、私たちはアカウント内のデータにアクセスしていないからです。スマートコントラクトのデータを変更するだけです。

`execute` フェーズに次の行を追加することで、これを行うことができます。`HelloWorld.changeGreeting(newGreeting: myNewGreeting)`. ここで、`argumentLabel` は引数の名前、`value` は実際の値です。`myNewGreeting` が定義されていないというエラーが発生することに気がつくと思います。そこで、トランザクションに `myNewGreeting` というパラメータを追加して、新しい挨拶（greeting）の値を渡せるようにしましょう。

```cadence
import HelloWorld from 0x01

transaction(myNewGreeting: String) {

  prepare(signer: AuthAccount) {}

  execute {
    HelloWorld.changeGreeting(newGreeting: myNewGreeting)
  }
}

```

右側に、プロンプトがポップアップ表示されます。この小さなボックスに、新しい挨拶（greeting）を入力することができます。「Goodbye, World!」と入力してみましょう。

<img src="../images/txgoodbye.PNG" alt="drawing" size="400" />

どのアカウントからでもこのトランザクションに「署名」できます。実際の環境ではないので、どのアカウントを選んでもかまいません。

「Send」をクリックしたら、スクリプトに戻り、「Execute」をクリックします。すると、コンソールに「Goodbye, World!」と表示されるはずです。これで、最初のトランザクションが成功したことになります。

本日はこれでおしまいです。

## クエスト

好きな言語で答えてください。

1. スクリプトで `changeGreeting` を呼び出さない理由を説明しなさい。
2. トランザクションの `prepare` フェーズで `AuthAccount` が意味するものは何ですか？
3. トランザクションの `prepare` フェーズと `execute` フェーズの違いは何ですか？
4. このクエストは今までで一番難しいので、時間がかかっても心配しないでください。質問があれば Discord で助けることができます。

- コントラクトの中に新しく 2 つのものを追加してください。

  - `myNumber` という変数で、型は `Int`（デプロイ時には 0 にセットしてください）
  - `updateMyNumber` という名前の関数。`newNumber` という新しい数値を `Int` 型のパラメータとして受け取り、`myNumber` を `newNumber` になるように更新します。

- コントラクトから `myNumber` を読み込むスクリプトを追加する。

- `myNewNumber` という名前のパラメータを受け取り、それを `updateMyNumber` 関数に渡すトランザクションを追加します。スクリプトをもう一度実行して、番号が変更されたことを確認します。
