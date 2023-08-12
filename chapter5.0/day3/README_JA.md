# 第5章3日目 - NFT コントラクトの作成：NonFungibleToken 標準の実装（パート3/3）

NonFungibleToken 規格に関する新しい知識を使って、第4章の CryptoPoops NFT コントラクトを完成させましょう。

私たちは今日一日、NFT コントラクトをここにある基準に合うように修正するだけに費やします： https://github.com/onflow/flow-nft/blob/master/contracts/NonFungibleToken.cdc

## ビデオ

今日は、31:20 - 終わりまでです。： https://www.youtube.com/watch?v=bQVXSpg6GE8

## NonFungibleToken 規格の実装

NonFungibleToken 規格にはたくさんのことが書かれています。少し見てみましょう：

```cadence
/**
## Flow Non-Fungible Token 規格
*/

// NFT コントラクトのメインインターフェースです。
// 他の NFT コントラクトはこのインターフェースをインポートして実装します。
//
pub contract interface NonFungibleToken {

    pub var totalSupply: UInt64

    pub event ContractInitialized()

    pub event Withdraw(id: UInt64, from: Address?)

    pub event Deposit(id: UInt64, to: Address?)

    pub resource interface INFT {
        // 各 NFT が持つ固有の ID
        pub let id: UInt64
    }

    pub resource NFT: INFT {
        pub let id: UInt64
    }

    pub resource interface Provider {
        pub fun withdraw(withdrawID: UInt64): @NFT {
            post {
                result.id == withdrawID: "The ID of the withdrawn token must be the same as the requested ID"
            }
        }
    }

    pub resource interface Receiver {
        pub fun deposit(token: @NFT)
    }

    pub resource interface CollectionPublic {
        pub fun deposit(token: @NFT)
        pub fun getIDs(): [UInt64]
        pub fun borrowNFT(id: UInt64): &NFT
    }

    pub resource Collection: Provider, Receiver, CollectionPublic {

        // コレクション内の NFT を保持する辞書です
        pub var ownedNFTs: @{UInt64: NFT}

        // withdraw はコレクションから NFT を削除し、呼び出し元に移動します
        pub fun withdraw(withdrawID: UInt64): @NFT

        // deposit は NFT を受け取り、コレクション辞書に追加します。
        // そして、その ID を id 配列に追加します。
        pub fun deposit(token: @NFT)

        // getIDs は、コレクションに含まれる ID の配列を返します。
        pub fun getIDs(): [UInt64]

        // コレクション内の NFT への借用参照を返します。
        // 呼び出し元がデータを読み込み、そこからメソッドを呼び出せるようにします。
        pub fun borrowNFT(id: UInt64): &NFT {
            pre {
                self.ownedNFTs[id] != nil: "NFT does not exist in the collection!"
            }
        }
    }

    pub fun createEmptyCollection(): @Collection {
        post {
            result.getIDs().length == 0: "The created collection must be empty!"
        }
    }
}
```

<img src="../images/homealone.jpg" />
うわぁ。怖いですね。

良い知らせは、そのほとんどを実際に実施したということです。信じられないかもしれないが、私は優秀な教師なので、君たちが知らないうちにこのコントラクトの 75％ を実施させました。ちくしょう、私は優秀です！これまでに書いたコントラクトを見てみましょう：

```cadence
pub contract CryptoPoops {
  pub var totalSupply: UInt64

  pub resource NFT {
    pub let id: UInt64

    pub let name: String
    pub let favouriteFood: String
    pub let luckyNumber: Int

    init(_name: String, _favouriteFood: String, _luckyNumber: Int) {
      self.id = self.uuid

      self.name = _name
      self.favouriteFood = _favouriteFood
      self.luckyNumber = _luckyNumber
    }
  }

  pub resource interface CollectionPublic {
    pub fun deposit(token: @NFT)
    pub fun getIDs(): [UInt64]
    pub fun borrowNFT(id: UInt64): &NFT
  }

  pub resource Collection: CollectionPublic {
    pub var ownedNFTs: @{UInt64: NFT}

    pub fun deposit(token: @NFT) {
      self.ownedNFTs[token.id] <-! token
    }

    pub fun withdraw(withdrawID: UInt64): @NFT {
      let nft <- self.ownedNFTs.remove(key: withdrawID) 
              ?? panic("This NFT does not exist in this Collection.")
      return <- nft
    }

    pub fun getIDs(): [UInt64] {
      return self.ownedNFTs.keys
    }

    pub fun borrowNFT(id: UInt64): &NFT {
      return (&self.ownedNFTs[id] as &NFT?)!
    }

    init() {
      self.ownedNFTs <- {}
    }

    destroy() {
      destroy self.ownedNFTs
    }
  }

  pub fun createEmptyCollection(): @Collection {
    return <- create Collection()
  }

  pub resource Minter {

    pub fun createNFT(name: String, favouriteFood: String, luckyNumber: Int): @NFT {
      return <- create NFT(_name: name, _favouriteFood: favouriteFood, _luckyNumber: luckyNumber)
    }

    pub fun createMinter(): @Minter {
      return <- create Minter()
    }

  }

  init() {
    self.totalSupply = 0
    self.account.save(<- create Minter(), to: /storage/Minter)
  }
}
```

悪くないでしょう？いい感じでしょ？コントラクトインターフェースを実装するには、`: {contract interface}` 構文を使わなければなりません。では、ここでそうしましょう...

```cadence
import NonFungibleToken from 0x02
pub contract CryptoPoops: NonFungibleToken {
  // ...その他...
}
```

*注意：これらのコントラクトは長くなってきたので、上記のように省略し始め、そこにあるべき他の内容は「...その他...」と置き換えるつもりです。*

前章で、コントラクトインターフェースは、それを実装するためにコントラクトに必要なものをいくつか指定していることを思い出してください。これを実装すると、コントラクトのエラーが大量に発生することに気づくでしょう。心配しないでください。

`NonFungibleToken` のコントラクトインターフェースで最初に目にするのは、以下のものです：

```cadence
pub var totalSupply: UInt64

pub event ContractInitialized()

pub event Withdraw(id: UInt64, from: Address?)

pub event Deposit(id: UInt64, to: Address?)
```

`totalSupply` はすでにありますが、イベントを `CryptoPoops` コントラクトに入れる必要があります。以下にそれを説明します：

```cadence
import NonFungibleToken from 0x02
pub contract CryptoPoops: NonFungibleToken {
  pub var totalSupply: UInt64

  pub event ContractInitialized()
  pub event Withdraw(id: UInt64, from: Address?)
  pub event Deposit(id: UInt64, to: Address?)

  // ...その他...
}
```

素晴らしいです！次に NonFungibleToken の標準によると、`id` フィールドを持つ `NFT` リソースが必要で、`NonFungibleToken.INFT` を実装する必要があります。最初の 2 つはすでに実装していますが、標準にあるような `NonFungibleToken.INFT` リソースインターフェースは実装していません。そこで、これもコントラクトに追加しましょう。

```cadence
import NonFungibleToken from 0x02
pub contract CryptoPoops: NonFungibleToken {
  pub var totalSupply: UInt64

  pub event ContractInitialized()
  pub event Withdraw(id: UInt64, from: Address?)
  pub event Deposit(id: UInt64, to: Address?)

  // ここに追加したのはただひとつ：
  // `: NonFungibleToken.INFT`
  pub resource NFT: NonFungibleToken.INFT {
    pub let id: UInt64

    pub let name: String
    pub let favouriteFood: String
    pub let luckyNumber: Int

    init(_name: String, _favouriteFood: String, _luckyNumber: Int) {
      self.id = self.uuid

      self.name = _name
      self.favouriteFood = _favouriteFood
      self.luckyNumber = _luckyNumber
    }
  }

  // ...その他...
}
```

驚きました。もう半分終わったところです。

規格の中で次に目にするのは、これら 3 つのリソースインターフェースです：

```cadence
pub resource interface Provider {
    pub fun withdraw(withdrawID: UInt64): @NFT {
        post {
            result.id == withdrawID: "The ID of the withdrawn token must be the same as the requested ID"
        }
    }
}

pub resource interface Receiver {
    pub fun deposit(token: @NFT)
}

pub resource interface CollectionPublic {
    pub fun deposit(token: @NFT)
    pub fun getIDs(): [UInt64]
    pub fun borrowNFT(id: UInt64): &NFT
}
```

たくさんのコードがあるように見えるでしょう？良いニュースは、もし昨日のことを覚えていれば、標準を使用する私たち自身のコントラクトの中でリソースインターフェースを再実装する必要がないということです。これらのインターフェースは `Collection` リソースが実装できるように定義されているだけです。

`Collection` にこれらのリソースインターフェースを実装する前に、これらが何をするのかを説明しましょう：

### プロバイダー
まずは `Provider` です。この関数は `withdrawID` を受け取り、`@NFT` を返します。**重要：戻り値の型に注意：`@NFT`.** どの NFT リソースのことでしょうか？`CryptoPoops` コントラクト内の `NFT` 型のことでしょうか？いいえ！`NonFungibleToken` コントラクトのインターフェイスにある型を指しているのです。したがって、これらの関数を実装する際には、`@NFT` だけではなく、`@NonFungibleToken.NFT` と記述する必要があります。これについても前章で説明しました。

それでは、Collection に `Provider` を実装してみましょう：

```cadence
import NonFungibleToken from 0x02
pub contract CryptoPoops: NonFungibleToken {
  // ...その他...

  // コレクションが NonFungibleToken.Provider を実装するようになりました。
  pub resource Collection: NonFungibleToken.Provider {
    pub var ownedNFTs: @{UInt64: NFT}

    // 戻り値の型が `@NonFungibleToken.NFT` になっていることに注目してください、 
    // `@NFT` だけではありません
    pub fun withdraw(withdrawID: UInt64): @NonFungibleToken.NFT {
      let nft <- self.ownedNFTs.remove(key: withdrawID) 
              ?? panic("This NFT does not exist in this Collection.")
      return <- nft
    }

    // ...その他...

    init() {
      self.ownedNFTs <- {}
    }

    destroy() {
      destroy self.ownedNFTs
    }
  }

  // ...その他...
}
```

### レシーバー
クールです！`Receiver` コントラクトインターフェースはどうですか？これを実装するものは `@NonFungibleToken.NFT` 型の `token` パラメータを受け取る `deposit` 関数を持つ必要があります。以下、`NonFungibleToken.Receiver` を Collection に追加してみましょう：

```cadence
import NonFungibleToken from 0x02
pub contract CryptoPoops: NonFungibleToken {
  // ...その他...

  // コレクションが NonFungibleToken.Receiver を実装するようになりました。
  pub resource Collection: NonFungibleToken.Provider, NonFungibleToken.Receiver {
    pub var ownedNFTs: @{UInt64: NFT}

    pub fun withdraw(withdrawID: UInt64): @NonFungibleToken.NFT {
      let nft <- self.ownedNFTs.remove(key: withdrawID) 
              ?? panic("This NFT does not exist in this Collection.")
      return <- nft
    }

    // `token` パラメータの型が次のようになっていることに注目してください。
    // `@NFT` ではなく、`@NonFungibleToken.NFT`です。
    pub fun deposit(token: @NonFungibleToken.NFT) {
      self.ownedNFTs[token.id] <-! token
    }

    // ...その他...

    init() {
      self.ownedNFTs <- {}
    }

    destroy() {
      destroy self.ownedNFTs
    }
  }

  // ...その他...
}
```

スウィート。これで `withdraw` 関数と `deposit` 関数が正しい型を使って動作するようになりました。しかし、ここで追加できることがいくつかあります：
1. `Withdraw` 関数の中で `Withdraw` イベントを `emit` してみましょう
2. `deposit` 関数の中で `Deposit` イベントを `emit` してみましょう
3. この `Collection` は `NonFungibleToken` コントラクトのインターフェースに適合する必要があるため、`ownedNFTs` を変更し、コントラクトの `@NFT` 型だけでなく、`@NonFungibleToken.NFT` 型のトークンも格納できるようにする必要があります。これを行わないと、`Collection` が標準に適合しなくなります。

以下の3つの変更を行ってみましょう：

```cadence
import NonFungibleToken from 0x02
pub contract CryptoPoops: NonFungibleToken {
  // ...その他...

  pub resource Collection: NonFungibleToken.Provider, NonFungibleToken.Receiver {
    // 3. これを `@NonFungibleToken.NFT` に変更しました。
    pub var ownedNFTs: @{UInt64: NonFungibleToken.NFT}

    pub fun withdraw(withdrawID: UInt64): @NonFungibleToken.NFT {
      let nft <- self.ownedNFTs.remove(key: withdrawID) 
            ?? panic("This NFT does not exist in this Collection.")

      // 1. `Withdraw` イベントを発行します
      emit Withdraw(id: nft.id, from: self.owner?.address)

      return <- nft
    }

    pub fun deposit(token: @NonFungibleToken.NFT) {
      // 2. `Deposit` イベントを発行します
      emit Deposit(id: token.id, to: self.owner?.address)

      self.ownedNFTs[token.id] <-! token
    }

    // ...その他...

    init() {
      self.ownedNFTs <- {}
    }

    destroy() {
      destroy self.ownedNFTs
    }
  }

  // ...その他...
}
```

驚きました。ひとつ質問があります：

**`self.owner?.address` とは何ですか?**

`self.owner` は、アカウントが保持しているリソース内で使用できるコードです。Collection リソースはアカウントのストレージ内に存在するので、`self.owner` を使用することで、特定の Collection をストレージ内に保持している現在のアカウントを取得することができます。これは、誰がアクションを実行しているのかを特定するのに非常に役立ちます。特に、誰に入金し、誰から出金しているのかを伝えたい場合に便利です。`self.owner?.address` は単に所有者のアドレスです。

次に、`@NonFungibleToken.NFT` 型とは何かを考えてみましょう。これは単なる `@NFT` よりも汎用的な型です。技術的には、文字通り Flow 上のあらゆる NFT が `@NonFungibleToken.NFT` 型に当てはまります。これには長所と短所がありますが、決定的な短所は、誰でも独自の NFT 型をコレクションに預けることができるようになったことです。例えば、私の友人が `BoredApes` というコントラクトを定義した場合、そのコントラクトは `@NonFungibleToken.NFT` 型を持っているので、技術的には私たちの Collection に預けることができます。したがって、「強制キャスト」と呼ばれるものを `deposit` 関数に追加する必要があります：

```cadence
import NonFungibleToken from 0x02
pub contract CryptoPoops: NonFungibleToken {
  // ...その他...

  pub resource Collection: NonFungibleToken.Provider, NonFungibleToken.Receiver {
    pub var ownedNFTs: @{UInt64: NonFungibleToken.NFT}

    pub fun withdraw(withdrawID: UInt64): @NonFungibleToken.NFT {
      let nft <- self.ownedNFTs.remove(key: withdrawID) 
            ?? panic("This NFT does not exist in this Collection.")
      emit Withdraw(id: nft.id, from: self.owner?.address)
      return <- nft
    }

    pub fun deposit(token: @NonFungibleToken.NFT) {
      // `token` to a `@NFT` type`token` を `@NFT` 型に「強制キャスト」します
      // `as!` 構文を使っています。これはこう言います：
      //  「もし `token` が `@NFT` 型なら、
      // それを `nft` 変数に移します。そうでない場合はパニックになります。」
      let nft <- token as! @NFT
      emit Deposit(id: nft.id, to: self.owner?.address)
      self.ownedNFTs[nft.id] <-! nft
    }

    // ...その他...

    init() {
      self.ownedNFTs <- {}
    }

    destroy() {
      destroy self.ownedNFTs
    }
  }

  // ...その他...
}
```

「強制キャスト」演算子 `as!` を使っているのがわかるでしょう。上記のコードでは、`@NonFungibleToken.NFT` は `@NFT` よりも汎用的な型です。**つまり、`as!` を使わなければなりません。これは基本的に、一般的な型（ `@NonFungibleToken.NFT` ）をより特殊な型（ `@NFT` ）に「ダウンキャスト」します。**この場合、`let nft <- token as！NFT` は言います：「もし `token` が `@NFT` 型なら、それを「ダウンキャスト」して `nft` 変数に移します。そうでない場合は、パニックになります。」これで、CryptoPoops だけをコレクションに預けることができるようになりました。

### CollectionPublic
最後に実装する必要があるリソースインターフェースは `CollectionPublic` で、次のようになります：

```cadence
pub resource interface CollectionPublic {
    pub fun deposit(token: @NFT)
    pub fun getIDs(): [UInt64]
    pub fun borrowNFT(id: UInt64): &NFT
}
```

さて、すでに `deposit` はあるが、`getIDs` と `borrowNFT` が必要です。`NonFungibleToken.CollectionPublic` を Collection に追加しましょう：

```cadence
import NonFungibleToken from 0x02
pub contract CryptoPoops: NonFungibleToken {
  // ...その他...

  // Collection が NonFungibleToken.CollectionPublic を実装するようになりました
  pub resource Collection: NonFungibleToken.Provider, NonFungibleToken.Receiver, NonFungibleToken.CollectionPublic {
    pub var ownedNFTs: @{UInt64: NonFungibleToken.NFT}

    pub fun withdraw(withdrawID: UInt64): @NonFungibleToken.NFT {
      let nft <- self.ownedNFTs.remove(key: withdrawID) 
            ?? panic("This NFT does not exist in this Collection.")
      emit Withdraw(id: nft.id, from: self.owner?.address)
      return <- nft
    }

    pub fun deposit(token: @NonFungibleToken.NFT) {
      let nft <- token as! @NFT
      emit Deposit(id: nft.id, to: self.owner?.address)
      self.ownedNFTs[nft.id] <-! nft
    }

    // Added this function. We already did this before.
    pub fun getIDs(): [UInt64] {
      return self.ownedNFTs.keys
    }

    // この機能を追加しました。以前からすでにやっていましたが、
    // 標準に合わせるために、型を `&NonFungibleToken.NFT`に
    // 変更しなければなりません。
    pub fun borrowNFT(id: UInt64): &NonFungibleToken.NFT {
      return (&self.ownedNFTs[id] as &NonFungibleToken.NFT?)!
    }

    init() {
      self.ownedNFTs <- {}
    }

    destroy() {
      destroy self.ownedNFTs
    }
  }

  // ...その他...
}
```

クールです！`getIDs` （以前と変わっていないです）と `borrowNFT` の両方を追加しました。標準に合わせるために、型を  `&NFT` ではなく `&NonFungibleToken.NFT` に変更する必要がありました。

やったー－！あと少しです。規格が最後に実装することを求めているのは `createEmptyCollection` 関数です！下に追加しましょう：

```cadence
import NonFungibleToken from 0x02
pub contract CryptoPoops: NonFungibleToken {
  // ...その他...

  // Add the `createEmptyCollection` function.
  pub fun createEmptyCollection(): @NonFungibleToken.Collection {
    return <- create Collection()
  }

  // ...その他...
}
```

もちろん、戻り値の型も `@NonFungibleToken.Collection` にしなければなりません。

最後に、コントラクトの `init` 内で `ContractInitialized` イベントを使用します：

```cadence
import NonFungibleToken from 0x02
pub contract CryptoPoops: NonFungibleToken {
  // ...その他...

  init() {
    self.totalSupply = 0
    emit ContractInitialized()
  }
}
```

さて、標準を正しく実装したところで、ミント機能も追加してみましょう：

```cadence
import NonFungibleToken from 0x02
pub contract CryptoPoops: NonFungibleToken {
  
  // ...その他...

   // ミントリソースを定義し、
   // NFT の作成方法を
   // 管理できるようにします。
   pub resource Minter {

    pub fun createNFT(name: String, favouriteFood: String, luckyNumber: Int): @NFT {
      return <- create NFT(_name: name, _favouriteFood: favouriteFood, _luckyNumber: luckyNumber)
    }

    pub fun createMinter(): @Minter {
      return <- create Minter()
    }

  }

  init() {
    self.totalSupply = 0
    emit ContractInitialized()

    // ミント機能をアカウントストレージに保存します：
    self.account.save(<- create Minter(), to: /storage/Minter)
  }
}
```

これで終わりです！！！！！！では、コントラクト全体を見てみましょう：

```cadence
import NonFungibleToken from 0x02
pub contract CryptoPoops: NonFungibleToken {
  pub var totalSupply: UInt64

  pub event ContractInitialized()
  pub event Withdraw(id: UInt64, from: Address?)
  pub event Deposit(id: UInt64, to: Address?)

  pub resource NFT: NonFungibleToken.INFT {
    pub let id: UInt64

    pub let name: String
    pub let favouriteFood: String
    pub let luckyNumber: Int

    init(_name: String, _favouriteFood: String, _luckyNumber: Int) {
      self.id = self.uuid

      self.name = _name
      self.favouriteFood = _favouriteFood
      self.luckyNumber = _luckyNumber
    }
  }

  pub resource Collection: NonFungibleToken.Provider, NonFungibleToken.Receiver, NonFungibleToken.CollectionPublic {
    pub var ownedNFTs: @{UInt64: NonFungibleToken.NFT}

    pub fun withdraw(withdrawID: UInt64): @NonFungibleToken.NFT {
      let nft <- self.ownedNFTs.remove(key: withdrawID) 
            ?? panic("This NFT does not exist in this Collection.")
      emit Withdraw(id: nft.id, from: self.owner?.address)
      return <- nft
    }

    pub fun deposit(token: @NonFungibleToken.NFT) {
      let nft <- token as! @NFT
      emit Deposit(id: nft.id, to: self.owner?.address)
      self.ownedNFTs[nft.id] <-! nft
    }

    pub fun getIDs(): [UInt64] {
      return self.ownedNFTs.keys
    }

    pub fun borrowNFT(id: UInt64): &NonFungibleToken.NFT {
      return (&self.ownedNFTs[id] as &NonFungibleToken.NFT?)!
    }

    init() {
      self.ownedNFTs <- {}
    }

    destroy() {
      destroy self.ownedNFTs
    }
  }

  pub fun createEmptyCollection(): @NonFungibleToken.Collection {
    return <- create Collection()
  }

  pub resource Minter {

    pub fun createNFT(name: String, favouriteFood: String, luckyNumber: Int): @NFT {
      return <- create NFT(_name: name, _favouriteFood: favouriteFood, _luckyNumber: luckyNumber)
    }

    pub fun createMinter(): @Minter {
      return <- create Minter()
    }

  }

  init() {
    self.totalSupply = 0
    emit ContractInitialized()
    self.account.save(<- create Minter(), to: /storage/Minter)
  }
}
```

## 問題点

この CryptoPoops コントラクトには 1 つ問題があります。よく見ると、`BorrowNFT` 関数の `Collection` 内での書き方に非常に大きな問題があることに気づくでしょう。この関数は `&NFT` 型ではなく `&NonFungibleToken.NFT` 型を返します。何が悪いかわかりますか？

`borrowNFT` の要点は、NFT のメタデータを読めるようにすることです。しかし、`&NonFungibleToken.NFT` 型で何が公開されるのでしょうか？`id` フィールドだけです！これで NFT リソースの他のメタデータは読めなくなりました。

これを解決するには、`auth` 参照というものを使う必要があります。上記の 「強制キャスト 」演算子 `as!` を覚えているのであれば、汎用の型をより特定の型に「ダウンキャスト 」し、うまくいかなければパニックになります。参照の場合、「ダウンキャスト」するためには、`auth` キーワードでマークされた「認可された参照」が必要です。そのためには次のようにします：

```cadence
pub fun borrowAuthNFT(id: UInt64): &NFT {
  let ref = (&self.ownedNFTs[id] as auth &NonFungibleToken.NFT?)!
  return ref as! &NFT
}
```

私たちが何をしたかわかりますか？その前に `auth` を置くことで、`&NonFungibleToken.NFT` 型への正規参照を取得し、`as!` を使って `&NFT` 型に「ダウンキャスト」しました。リファレンスを使用する場合、ダウンキャストしたいのであれば、 `auth` リファレンスを持って**いなければなりません**。

もし `ref` が `&NFT` 型でなかったらパニックになりますが、deposit関数では `@NFT` 型を保存していることを確認しているので、常に機能することは分かっています。

やったぁぁぁぁ！これで `borrowAuthNFT` 関数を使って NFT のメタデータを読むことができます。しかし、問題がもう一つあります。 `borrowAuthNFT` は公開されていません。なぜなら `NonFungibleToken.CollectionPublic` 内にないからです。この問題はクエストで解決してください。

## まとめ

これであなただけの NFT コントラクトが完成しました。さらに素晴らしいことに、これは正式に NonFungibleToken コントラクトとなり、好きな場所でこれを使用することができ、アプリケーションは NFT コントラクトで動作していることを認識することができます。これはすごいことです。

さらに、コースの最初のメインセクションを正式に修了しました。あなたは Cadence の開発者を名乗ることができます！このコースを一時中断し、あなた自身のコントラクトをいくつか実装することをお勧めします。次の章では、Flow Testnet にコントラクトをデプロイし、コントラクトと対話する方法を学びます。

## クエスト

1. `as！` を使った「強制キャスト」は何をするのですか？なぜコレクションで役に立つのですか？

2. `auth` は何をするのですか？いつ使うのですか？

3. この最後のクエストは、これまでで最も難しいものになるでしょう。このコントラクトをみてください：

```cadence
import NonFungibleToken from 0x02
pub contract CryptoPoops: NonFungibleToken {
  pub var totalSupply: UInt64

  pub event ContractInitialized()
  pub event Withdraw(id: UInt64, from: Address?)
  pub event Deposit(id: UInt64, to: Address?)

  pub resource NFT: NonFungibleToken.INFT {
    pub let id: UInt64

    pub let name: String
    pub let favouriteFood: String
    pub let luckyNumber: Int

    init(_name: String, _favouriteFood: String, _luckyNumber: Int) {
      self.id = self.uuid

      self.name = _name
      self.favouriteFood = _favouriteFood
      self.luckyNumber = _luckyNumber
    }
  }

  pub resource Collection: NonFungibleToken.Provider, NonFungibleToken.Receiver, NonFungibleToken.CollectionPublic {
    pub var ownedNFTs: @{UInt64: NonFungibleToken.NFT}

    pub fun withdraw(withdrawID: UInt64): @NonFungibleToken.NFT {
      let nft <- self.ownedNFTs.remove(key: withdrawID) 
            ?? panic("This NFT does not exist in this Collection.")
      emit Withdraw(id: nft.id, from: self.owner?.address)
      return <- nft
    }

    pub fun deposit(token: @NonFungibleToken.NFT) {
      let nft <- token as! @NFT
      emit Deposit(id: nft.id, to: self.owner?.address)
      self.ownedNFTs[nft.id] <-! nft
    }

    pub fun getIDs(): [UInt64] {
      return self.ownedNFTs.keys
    }

    pub fun borrowNFT(id: UInt64): &NonFungibleToken.NFT {
      return (&self.ownedNFTs[id] as &NonFungibleToken.NFT?)!
    }

    init() {
      self.ownedNFTs <- {}
    }

    destroy() {
      destroy self.ownedNFTs
    }
  }

  pub fun createEmptyCollection(): @NonFungibleToken.Collection {
    return <- create Collection()
  }

  pub resource Minter {

    pub fun createNFT(name: String, favouriteFood: String, luckyNumber: Int): @NFT {
      return <- create NFT(_name: name, _favouriteFood: favouriteFood, _luckyNumber: luckyNumber)
    }

    pub fun createMinter(): @Minter {
      return <- create Minter()
    }

  }

  init() {
    self.totalSupply = 0
    emit ContractInitialized()
    self.account.save(<- create Minter(), to: /storage/Minter)
  }
}
```

上記でやったように `borrowAuthNFT` という関数を追加します。そして、他の人が NFT のメタデータを読むことができるように、この関数をパブリックにアクセスできるようにする方法を見つけます。次に、ある `id` の NFT のメタデータを表示するスクリプトを実行します。

アカウントを設定するためのトランザクションをすべて記述し、NFT を作成し、NFT のメタデータを読み込むためのスクリプトを記述する必要があります。ここまでの章ではそのほとんどを説明していますので、そちらを参考にしてください。:)
