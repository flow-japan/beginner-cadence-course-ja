# 第5章2日目 - コントラクトインターフェース

本日は、NFTスマートコントラクトを完成させるために必要な最後のコンセプトを学びます。

## ビデオ

コントラクトインターフェース： https://www.youtube.com/watch?v=NHMBE6iRyfY

## コントラクトインターフェース

今日のレッスンはとても簡単です。あなたはすでにこのほとんどを学んでいます、あなたははまだそれを知らないだけなんです。;）

コントラクトインターフェースは、コントラクト用であることを除けば、リソースインターフェースとほとんど同じです。しかし、「コントラクトインターフェースをどのように定義するか」など、いくつかの違いがあります。以下を見てみましょう：

```cadence
pub contract interface IHelloWorld {

}
```

コントラクトインターフェースは、コントラクトと同様、それ自体でデプロイされます。コントラクトの中にあるのではなく、完全に独立しています。

コントラクトインターフェイスは通常のコントラクトと同じように配置します。唯一の違いは、上の例のように `contract interface` キーワードを使って宣言することです。

リソースインターフェースと同様、変数を初期化したり関数を定義したりすることはできません。以下にインターフェースの例を示します：

```cadence
pub contract interface IHelloWorld {
  pub var greeting: String
  
  pub fun changeGreeting(newGreeting: String)
}
```

このコントラクトインターフェースを実際のコントラクトに実装することができます：

```cadence
import IHelloWorld from 0x01
pub contract HelloWorld: IHelloWorld {

}
```

リソースと同じように、`: {contract interface name}` 構文を使っていることに気づくでしょう。

また、いくつかのエラーが発生することにもお気づきでしょう：「コントラクト `HelloWorld` はコントラクトインターフェイス `IHelloWorld` に適合していません。」なぜでしょう？もちろん、実装していないからです！

```cadence
import IHelloWorld from 0x01
pub contract HelloWorld: IHelloWorld {
  pub var greeting: String
  
  pub fun changeGreeting(newGreeting: String) {
    self.greeting = newGreeting
  }

  init() {
    self.greeting = "Hello, Jacob!"
  }
}
```

ああ、もう大丈夫です。素晴らしい！

### 事前条件／事後条件

昨日、事前条件と事後条件について学びました。この条件の素晴らしいところは、リソースインターフェースやコントラクトインターフェースの内部で実際に使えることです：

```cadence
pub contract interface IHelloWorld {
  pub var greeting: String
  
  pub fun changeGreeting(newGreeting: String) {
    post {
      self.greeting == newGreeting: "We didn't update the greeting appropiately."
    }
  }
}
```

私たちはまだこの関数を実装していないが、ある制限を課しました：このコントラクトインターフェイスを実装するアカウントは次のことをしなければなりません：
1. `greeting` 文字列を定義する。
2. `changeGreeting` 関数を定義する。
3. さらに、事後条件のために、`greeting` を適切に更新して、渡された `newGreeting` にしなければなりません。

これは、私たちのルールを守っているかどうかを確認するための素晴らしい方法です。

### コントラクトインターフェイスにおけるリソースインターフェイス

もっと派手にやろうじゃまりませんか。コントラクトインターフェイスにリソースとリソース・インターフェイスを追加しましょう：

```cadence
pub contract interface IHelloWorld {
  pub var greeting: String
  
  pub fun changeGreeting(newGreeting: String) {
    post {
      self.greeting == newGreeting: "We didn't update the greeting appropiately."
    }
  }

  pub resource interface IGreeting {
    pub var favouriteFood: String
  }

  pub resource Greeting: IGreeting {
    pub var favouriteFood: String
  }
}
```

ここを見てほしいです！私たちはコントラクトインターフェイスの中に `Greeting` というリソースと `IGreeting` というリソースインターフェイスを定義しました。これはつまり：「このコントラクトインターフェースを実装するコントラクトは、`IHelloWorld.IGreeting` を実装した `Greeting` リソースを持たなければならない」ということです。

これは非常に重要なことです。もし私たちが独自のコントラクトを定義し、そのコントラクトが独自の `IGreeting` を定義するとしたら、次のようになります：

```cadence
import IHelloWorld from 0x01
pub contract HelloWorld: IHelloWorld {
  pub var greeting: String
  
  pub fun changeGreeting(newGreeting: String) {
    self.greeting = newGreeting
  }

  pub resource interface IGreeting {
    pub var favouriteFood: String
  }

  // エラー: リソース `HelloWorld.Greeting` に、リソースインターフェース
  // `IHelloWorld.IGreeting` への適合を要求する宣言がありません。
  pub resource Greeting: IGreeting {
    pub var favouriteFood: String

    init() {
      self.favouriteFood = "Chocolate chip pancakes." // とっっっても良いです
    }
  }

  init() {
    self.greeting = "Hello, Jacob!"
  }
}
```

... というエラーになります。なぜエラーになるかというと、私たちのコントラクトインターフェースは `Greeting` リソースは `IHelloWorld.IGreeting` を実装しなければならない、と明確に定めているからです。というわけで、実際のコントラクトはこのようになります：

```cadence
import IHelloWorld from 0x01
pub contract HelloWorld: IHelloWorld {
  pub var greeting: String
  
  pub fun changeGreeting(newGreeting: String) {
    self.greeting = newGreeting
  }

  pub resource Greeting: IHelloWorld.IGreeting {
    pub var favouriteFood: String

    init() {
      self.favouriteFood = "Chocolate chip pancakes." // とっっっても良いです
    }
  }

  init() {
    self.greeting = "Hello, Jacob!"
  }
}
```

これでもう大丈夫です :)

**注意：コントラクトインターフェイスがリソースインターフェイスを定義していても、実装するコントラクトはリソースインターフェイスも実装する必要はありません。上記のように、コントラクト・インターフェースの中に残しておくことができます。**

## 「標準」としてのコントラクトインターフェース

<img src="../images/nftpicforcourse.png" />

コントラクトインターフェースは、実装するコントラクトにいくつかの要件を指定し、さらに、特定のコントラクトがどのようなものであるかについての「標準」を作成することを可能にします。

実際にコードを読まなくても、あるコントラクトが「 NFT コントラクト」であることを合理的に説明できたら便利だと思いませんか？それはすでに存在しています！NonFungibleToken コントラクトインターフェース（別称 NonFungibleToken 標準）は、NFT コントラクトが 「 NFT コントラクト」とみなされるために必要なものを定義したコントラクトインターフェースです。これは、Marketplace DApp のようなクライアントが何を見ているのかを理解し、最も重要なことは、**NFT コントラクトごとに異なる機能を実装する必要がない**ということです。

標準化することは、複数のコントラクトを使用するクライアントが、それらすべてのコントラクトと単一の方法でやり取りできるようになるため、非常に有益です。例えば、すべての NFT コントラクトは Collection と呼ばれるリソースを持っており、このリソースは `deposit` と `withdraw` の機能を持っています。これにより、クライアント DApp が 100 の NFT コントラクトとやり取りする場合でも、NonFungibleToken 標準をインポートするだけでこれらの関数を呼び出すことができます。

詳しくはこちらをご覧ください： https://github.com/onflow/flow-nft

## まとめ

コントラクトインターフェイスは、特定のことを実装することを要求し、何をすることが許されるかについて厳しい制限を実装することを可能にするという点で、リソースインターフェイスと非常によく似ています。さらに、「標準」を設定することもできます。これは、コントラクトを合理化したり、コントラクトが主張している通りのものであることを保証したりする上で、とても役に立ちます。

偶然にも、コントラクトインターフェースは、Flow で最も議論の多いトピックです（と私は考えています）。というのも、例えば NonFungibleToken コントラクトインターフェース（ https://github.com/onflow/flow-nft/blob/master/contracts/NonFungibleToken.cdc ）は比較的古く、それをどう修正するかについて多くの議論があるからです。もしあなたが Flow の Discord にいることがあれば、私たちがそれについてノンストップで議論しているのを見ることができるでしょう ;)

## クエスト

1. 標準規格が Flow のエコシステムにとって有益である理由を説明します。

2. あなたの好きな食べ物はなんですか？

3. このコードを修正してください（ヒント：間違っている点が2つあります）：

コントラクトインターフェース：
```cadence
pub contract interface ITest {
  pub var number: Int
  
  pub fun updateNumber(newNumber: Int) {
    pre {
      newNumber >= 0: "We don't like negative numbers for some reason. We're mean."
    }
    post {
      self.number == newNumber: "Didn't update the number to be the new number."
    }
  }

  pub resource interface IStuff {
    pub var favouriteActivity: String
  }

  pub resource Stuff {
    pub var favouriteActivity: String
  }
}
```

実装コントラクト：
```cadence
import ITest from 0x01
pub contract Test {
  pub var number: Int
  
  pub fun updateNumber(newNumber: Int) {
    self.number = 5
  }

  pub resource interface IStuff {
    pub var favouriteActivity: String
  }

  pub resource Stuff: IStuff {
    pub var favouriteActivity: String

    init() {
      self.favouriteActivity = "Playing League of Legends."
    }
  }

  init() {
    self.number = 0
  }
}
```
