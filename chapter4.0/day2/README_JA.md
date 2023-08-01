# 第 4 章 2 日目 - ケイパビリティ

前回の章では、アカウントのストレージのパスである `/storage/` について説明しました。今回は `/public/` と `/private/` のパス、そしてケイパビリティとは何かについて説明します。

**注意: この章は難解です。** もし途中でわからなくなっても、落ち着いて何度か読み返せば、いずれは理解できるようになるはずです。

## 動画

この動画を 14:45 から最後まで見てください(前回、前半部分を視聴しました): [https://www.youtube.com/watch?v=01zvWVoDKmU](https://www.youtube.com/watch?v=01zvWVoDKmU)

## 前回の復習

<img src="../images/accountstorage1.PNG" />

前回のまとめ:
1. `/storage/`はアカウント所有者のみアクセス可能です。これには `.save()`, `.load()`, `.borrow()` という関数を使います。
2. `/public/` は誰でも利用可能です。
3. `/private/` は、アカウントのオーナーとオーナーがアクセス権を与えた人だけが利用できます。

今回の章では、前回のコントラクトコードを使用します。

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

リソースをストレージに保存したことも頭に留めておいてください。
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

では、準備が整いました。

## `/public/` パス

以前は、アカウントストレージに何かを保存した場合、アカウント所有者だけがアクセスできました。これは、`/storage/`パスに保存されたからです。しかし、私のリソースから `name` フィールドを他の人が読み取れるようにしたい場合はどうすればよいでしょうか。おそらくお察しの通りです。リソースを公開することにしましょう。

```cadence 
import Stuff from 0x01
transaction() {
  prepare(signer: AuthAccount) {
    // 他の人がアクセスできるように、リソースを `/public/` にリンクしています
    signer.link<&Stuff.Test>(/public/MyTestResource, target: /storage/MyTestResource)
  }

  execute {

  }
}
```

上記の例では、 `.link()` 関数を使用して、リソースを `/public/` パスに「リンク」しています。もっと簡単に言うと、 `/storage/MyTestResource` にあるものを、 `&Stuff.Test` として公開し、読み込めるようにしたのです。

`.link()` は2つのパラメータを受け取ります。
1. `/public/` または `/private/` パス
2. `target` パラメータには、現在リンクしているデータが保存されている `/storage/` のパスを指定します。

これで誰でもこのリソースの `name` フィールドを読み取るスクリプトを実行することができますが、スクリプトの説明の前にいくつかの他のことを紹介する必要があります。

**`.link()` 関数は新しいバージョンでは非推奨になります**

`.link()` 関数は新しいバージョンでは非推奨になります。代わりに、 `capabilities.storage.issue()` と `capabilities.publish()` を使います.

```cadence
import Stuff from 0x01
transaction() {
  prepare(signer: AuthAccount) {
    // Publish our resource to the public so other people can now access it
    let resource = signer.capabilities.storage.issue<&Stuff.Test>(/storage/MyTestResource)
    signer.capabilities.publish(resource, at: /public/MyTestResource)
  }

  execute {

  }
}
```

## ケイパビリティ

何かを `/public/` や `/private/` のパスに「リンク」するとき、あなたはケイパビリティと呼ばれるものを作成しています。実際には、何も `/public/` や `/private/` のパスには存在せず、全ては `/storage/` に存在します。しかし、ケイパビリティは `/public/` や `/private/` のパスから、関連する `/storage/` のパスへの「ポインタ」のように考えることができます。以下の図をみてみましょう。

<img src="../images/capabilities.PNG" /> 

クールな点は、 `/public/` や `/private/` の機能を、 `/storage/` パスにあるものよりも *強く制限する*ことができることです。これはとても良いことで、他の人ができることを制限しつつ、柔軟な機能を提供してくれます。これは後でリソースインターフェースを介して行うことになります。

## `PublicAccount` と `AuthAccount` の比較

私たちはすでに `AuthAccount` がアカウントに対して何でもできることを学びました。一方、 `PublicAccount` は誰でもそのアカウントの情報を読み取ることができますが、それはそのアカウントの所有者が公開しているものだけです。以下のように `getAccount` 関数を使用すると、 `PublicAccount` 型を取得することができます。

```cadence
let account: PublicAccount = getAccount(0x1)
// `account` はアドレス0x1のPublicAccountを保持しています
```

なぜこのような話をするかというと、`/public/` パスからケイパビリティを取得する唯一の方法は `PublicAccount` を使用することだからです。一方、`/private/` のパスからケイパビリティを取得するには、 `AuthAccount` を使用する必要があります。

## `/public/` の話の続き

さて、リソースをpublicにリンクしました。それでは、スクリプトでリソースを読み込んで、学習したことを実践してみましょう!

```cadence
import Stuff from 0x01
pub fun main(address: Address): String {
  // `&Stuff.Test`型を参照するパブリックなケイパビリティを取得します
  let publicCapability: Capability<&Stuff.Test> =
    getAccount(address).getCapability<&Stuff.Test>(/public/MyTestResource)

  // パブリックなケイパビリティから`&Stuff.Test`を借用します
  let testResource: &Stuff.Test = publicCapability.borrow() ?? panic("The capability doesn't exist or you did not specify the right type when you got the capability.")

  return testResource.name // "Jacob"
}
```

良いですね! リソースの名前を `/public/` パスから読み込みます。以下に手順をまとめます。
1. 指定したアドレスの`PublicAccount`を取得します: `getAccount(address)`
2. `/public/MyTestResource` パスにある `&Stuff.Test` 型を指すケイパビリティを取得します: `getCapability<&Stuff.Test>(/public/MyTestResource)`
3. 実際の参照を取得するために、ケイパビリティを借用します: `let testResource: &Stuff.Test = publicCapability.borrow() ?panic("The capability is invalid")`
4. 名前を返します：`return testResource.name`

なぜ `.borrow()` のときに参照先の型を指定する必要がなかったのか、不思議に思うかもしれません。答えは、ケイパビリティがすでに型を指定しているので、その型で借用できると仮定しているからです。もし、型が異なっていたり、ケイパビリティがそもそも存在しなかったりした場合は、`nil`を返してパニックになります。

**`getCapability()` は新しいバージョンでは非推奨になります**

`getCapability()` は新しいバージョンでは非推奨になります。代わりに、 `capabilities.get()` を使います。

```cadence
import Stuff from 0x01
pub fun main(address: Address): String {
  // gets the public capability that is pointing to a `&Stuff.Test` type
  let publicCapability: Capability<&Stuff.Test> = getAccount(address).capabilities.get<&Stuff.Test>(/public/MyTestResource) ?? panic("The capability doesn't exist or you did not specify the right type when you got the capability.")

  // Borrow the `&Stuff.Test` from the public capability
  let testResource: &Stuff.Test = publicCapability.borrow() ?? panic("The capability doesn't exist or you did not specify the right type when you got the capability.")

  return testResource.name // "Jacob"
}
```

## 型を制限するためにパブリックケイパビリティを使用する

いい感じですね。ここまで理解できたのは素晴らしいです。次のトピックは、一般ユーザーからの望ましくないアクションを防止するために、パブリックパス参照時に一部を制限する方法についてです。

別のコントラクトを定義してみましょう。

```cadence
pub contract Stuff {

  pub resource Test {
    pub var name: String

    pub fun changeName(newName: String) {
      self.name = newName
    }

    init() {
      self.name = "Jacob"
    }
  }

  pub fun createTest(): @Test {
    return <- create Test()
  }

}
```

この例では、リソース内の名前を変更できるように `changeName` 関数を追加しました。しかし、一般ユーザーがこれを実行できないようにしたい場合はどうすればよいでしょうか。このままでは問題がありますね。

```cadence
import Stuff from 0x01
transaction(address: Address) {

  prepare(signer: AuthAccount) {

  }

  execute {
    let publicCapability: Capability<&Stuff.Test> =
      getAccount(address).getCapability<&Stuff.Test>(/public/MyTestResource)

    let testResource: &Stuff.Test = publicCapability.borrow() ?? panic("The capability doesn't exist or you did not specify the right type when you got the capability.")

    testResource.changeName(newName: "Sarah") // これはセキュリティ的に大問題です!!!!!!!!!
  }
}
```

ご覧の通り、リソースをパブリックパスにリンクしているので、誰でも `changeName` を呼び出して名前を変更することができます。これでは脆弱ですね。

解決方法は、以下の通りです。
1. 新しいリソースインターフェースを定義して、 `name` フィールドのみを公開し、 `changeName` は公開しないようにします。
2. リソースを `/public/` のパスに `.link()` するときに、ステップ1で定義したリソースインターフェースを使用するように参照を制限します。

それでは、コントラクトにリソースインターフェースを追加してみましょう。

```cadence
pub contract Stuff {

  pub resource interface ITest {
    pub var name: String
  }

  // `Test`は`ITest`を実装しています
  pub resource Test: ITest {
    pub var name: String

    pub fun changeName(newName: String) {
      self.name = newName
    }

    init() {
      self.name = "Jacob"
    }
  }

  pub fun createTest(): @Test {
    return <- create Test()
  }

}
```

良いですね。これで `Test` は `ITest` という名前のリソースインターフェースを実装し、その中には `name` という名前だけが含まれるようになりました。こうすることでリソースを安全に公開することができます。

```cadence 
import Stuff from 0x01
transaction() {
  prepare(signer: AuthAccount) {
    // アカウントストレージにリソースを保存しています
    signer.save(<- Stuff.createTest(), to: /storage/MyTestResource)

    // おわかりいただけたでしょうか？私は`&Stuff.Test`ではなく、`&Stuff.Test{Stuff.ITest}`をリンクしました。
    // これで、一般のユーザーは`Stuff.ITest`にあるものにのみアクセスできます

    signer.link<&Stuff.Test{Stuff.ITest}>(/public/MyTestResource, target: /storage/MyTestResource)
  }

  execute {

  }
}
```

では、前回試したようにスクリプトで`&Stuff.Test`型にアクセスしようとするとどうなるでしょうか。

```cadence
import Stuff from 0x01
transaction(address: Address) {
  prepare(signer: AuthAccount) {

  }

  execute {
    let publicCapability: Capability<&Stuff.Test> =
      getAccount(address).getCapability<&Stuff.Test>(/public/MyTestResource)

    // ERROR: "ケイパビリティが存在しないか、ケイパビリティ取得時に正しい型を指定していません。" 
    let testResource: &Stuff.Test = publicCapability.borrow() ?? panic("ケイパビリティが存在しないか、ケイパビリティ取得時に正しい型を指定していません。")

    testResource.changeName(newName: "Sarah")
  }
}
```

今度はエラーが発生しました。あなたは`&Stuff.Test`のケイパビリティを借用しようとしましたが、私はそれを利用できるようにはしていません。私は`&Stuff.Test{Stuff.ITest}`を利用できるようにしただけなのです。

では、以下を試してみるとどうでしょうか？

```cadence
import Stuff from 0x01
transaction(address: Address) {

  prepare(signer: AuthAccount) {

  }

  execute {
    let publicCapability: Capability<&Stuff.Test{Stuff.ITest}> =
      getAccount(address).getCapability<&Stuff.Test{Stuff.ITest}>(/public/MyTestResource)

    // これは上手くいきます...
    let testResource: &Stuff.Test{Stuff.ITest} = publicCapability.borrow() ?? panic("ケイパビリティが存在しないか、ケイパビリティ取得時に正しい型を指定していません。")

    // ERROR: "Member of restricted type is not accessible: changeName"
    testResource.changeName(newName: "Sarah")
  }
}
```

またエラーが出ましたね！正しい型を借用したにもかかわらず、`&Stuff.Test{Stuff.ITest}`型からはアクセスできないので、 `changeName` を呼び出すことができません。

しかし、これならうまくいくでしょう。

```cadence
import Stuff from 0x01
pub fun main(address: Address): String {
  let publicCapability: Capability<&Stuff.Test{Stuff.ITest}> =
    getAccount(address).getCapability<&Stuff.Test{Stuff.ITest}>(/public/MyTestResource)

  let testResource: &Stuff.Test{Stuff.ITest} = publicCapability.borrow() ?? panic("The capability doesn't exist or you did not specify the right type when you got the capability.")

  // `name`というフィールドは`&Stuff.Test{Stuff.ITest}`に存在するので、これは上手くいきます
  return testResource.name
}
```

期待通り動作していますね！

## まとめ

大変でしたね。ですが、良い知らせです。ここまででCadenceについて非常に多くのことを学びました。そしてさらに、大体の難解なことは学びきりました。

今回、私は意図的に `/private/` について深く掘り下げませんでした。というのも、実際には`/private/`を使うことはほとんどないでしょうし、学びの邪魔になる可能性が高いからです。

## クエスト

好きな言語で答えてください。

1. `.link()` は何をするものですか？

2. リソースインターフェースを使って、どのように `/public/` パスに特定のものだけを公開できるか、あなた自身の言葉で説明してください (コードは必要ありません)。

3. リソースインターフェースを実装したリソースを含むコントラクトをデプロイしてください。その後、以下を実行しましょう。
    1) トランザクションで、リソースをストレージに保存し、制限付きインターフェースでパブリックにリンクする。
    2) リソースインターフェースの公開されていないフィールドにアクセスしようとするスクリプトを実行し、エラーがポップアップされるのを確認する。
    3) スクリプトを実行し、読み込み可能なものにアクセスする。スクリプトからそれを返す。
