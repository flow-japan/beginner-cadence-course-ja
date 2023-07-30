# 第4章4日目 - NFT コントラクトの作成：譲渡、ミント、借入（パート2/3）

NFT のコントラクトを構築し続けましょう！ :D

## ビデオ

今日は 20:35 - 31:20 までを取り上げます: https://www.youtube.com/watch?v=bQVXSpg6GE8

## これまでの総括

前日、NFT を作成し、Collection の中に保存する方法について説明しました。Collection を作成したのは、すべての NFT をアカウントストレージの一箇所に保存するためです。

しかし、私たちは少々問題を残しました：_本当_ に誰でも NFT を作ることができるのでしょうか？それは少しおかしいです。誰が NFT をミントできるかを管理したいとしたらどうでしょうか？今日はその話をしましょう。

## 譲渡

NFT を "ミント"（または作成）できる人のコントロールに入る前に、譲渡について説明しましょう。NFT をあるアカウントから別のアカウントに移すにはどうすればよいのでしょうか。

そういえば、コレクションの所有者だけが自分のコレクションから `withdraw` できます。しかし、誰でも他の人の Collection に `deposit` することができます。つまり、1つの AuthAccount にアクセスするだけでよいのです：NFT を譲渡（引き出し）する人！NFT を送金するトランザクションをスピンアップしてみましょう：

_注意：これは、すでに両方のアカウントにコレクションを設定していることを前提としています。_

```cadence
import CryptoPoops from 0x01

// `id` は NFT の `id` です。
// `recipient` とは、NFT を受ける人のことです。
transaction(id: UInt64, recipient: Address) {

  prepare(signer: AuthAccount) {
    // 署名者のコレクションへの参照を取得します。
    let signersCollection = signer.borrow<&CryptoPoops.Collection>(from: /storage/MyCollection)
                            ?? panic("Signer does not have a CryptoPoops Collection")

    // `recipient` のパブリックコレクションへの参照を取得します。
    let recipientsCollection = getAccount(recipient).getCapability(/public/MyCollection)
                                  .borrow<&CryptoPoops.Collection{CryptoPoops.CollectionPublic}>
                                  ?? panic("The recipient does not have a CryptoPoops Collection.")

    // id で NFT を引き出す == `id` を変数  `nft` に移します。
    let nft <- signersCollection.withdraw(withdrawID: id)

    // NFT は受取人のコレクションにデポジットされます。
    recipientsCollection.deposit(token: <- nft)
  }

}
```

悪くないでしょう？昨日の内容を学べば、すべて理解できるはずです。手順はこうです：

1. まず、署名者のコレクションへの参照を取得します。`withdraw` を呼び出す必要があり、ストレージから直接参照する必要もあるため、ケイパビリティは使用しません。
2. 次に、受信者のコレクションへの _パブリック_ リファレンスを取得します。私たちは相手の AuthAccount にアクセスできないので、パブリックケーパビリティを通してこれを取得します、 しかし、`deposit` するだけなのでこれは問題ありません。
3. `id` を持つ NFT を署名者のコレクションから `withdraw`。
4. NFT を受取人のコレクションに `deposit` 。

## ミント

よし、では、誰もが NFT をミントできないようにする方法を考えましょう。問題は、では誰がミント能力を持つべきかです。

Cadence の良さは、自分たちで決められることです。まず、NFT をミントするリソースを作ってはどうでしょうか。そうすれば、リソースを所有する人は誰でも NFT をミントする能力を持つことができます。これまでのコントラクトの上に構築してみましょう：

```cadence
pub contract CryptoPoops {

  // ... その他はこちら ...

  pub fun createEmptyCollection(): @Collection {
    return <- create Collection()
  }

  // 新しいリソース： Minter
  // 保有者は誰でも
  //  NFT のミント
  pub resource Minter {

    // 新しい NFT リソースをミントします
    pub fun createNFT(): @NFT {
      return <- create NFT()
    }
  }

  init() {
    self.totalSupply = 0
  }
}
```

以下は追加されたものです：

1. `Minter` という新しいリソースを作成しました。
2. 関数 `createNFT` を `Minter` リソースに移動しました。

これで `Minter` リソースを持っている人は誰でも NFT をミントできます。さて、それはクールなことですが、今度は誰が minter を持つようになるのでしょうか？

最も簡単な解決策は、コントラクトをデプロイするアカウントに `Minter` を自動的に渡すことです。そのためには、`init` 関数の中で `Minter` リソースをコントラクトをデプロイするアカウントのストレージに保存します：

```cadence
pub contract CryptoPoops {

  // ... その他はこちら ...

  pub fun createEmptyCollection(): @Collection {
    return <- create Collection()
  }

  pub resource Minter {

    pub fun createNFT(): @NFT {
      return <- create NFT()
    }

  }

  init() {
    self.totalSupply = 0

    // `Minter` リソースをアカウントストレージに保存します。
    self.account.save(<- create Minter(), to: /storage/Minter)
  }
}
```

コントラクトをデプロイするときに `init` 関数の中で、デプロイするアカウントの `AuthAccount` にアクセスすることができます！そこで、アカウントのストレージにデータを保存することができます。

一番最後に何をしたかわかりますか？`Minter` をアカウントストレージに保存したのです。完璧です！これで、コントラクトをデプロイしたアカウントだけが NFT をミントできるようになり、他のユーザーに `Minter` を取得させる機能はないので、完全に安全です！

`Minter` が誰かに NFT をミントするトランザクションの例を見てみましょう。

_注意：`Minter` リソースを持っているのは彼らだけなので、`signer` がコントラクトをデプロイした人だと仮定しましょう。_

```cadence
import CryptoPoops from 0x01

transaction(recipient: Address) {

  // 彼らだけが `Minter` リソースを持っているので、`signer` がコントラクトを展開した人だと仮定しましょう。
  prepare(signer: AuthAccount) {
    // `Minter` への参照を取得します。
    let minter = signer.borrow<&CryptoPoops.Minter>(from: /storage/Minter)
                    ?? panic("This signer is not the one who deployed the contract.")

    // `recipient` のパブリックコレクションへの参照を取得します。
    let recipientsCollection = getAccount(recipient).getCapability(/public/MyCollection)
                                  .borrow<&CryptoPoops.Collection{CryptoPoops.CollectionPublic}>
                                  ?? panic("The recipient does not have a Collection.")

    // `Minter` への参照を使って NFT をミントします。
    let nft <- minter.createNFT()

    // NFT を受取人のコレクションに預けます。
    recipientsCollection.deposit(token: <- nft)
  }

}
```

うーーーわーーー！安全なミントの実装に成功しました。これは Cadence において非常に重要なパターンです："Admin" の機能を特定のリソース（この場合は `Minter`）に委譲する機能です。この "Admin" は多くの場合、コントラクトをデプロイしたアカウントのストレージに与えられます。

## 借入

さてさて、最後に。昨日、NFT を文字通り Collection から取り出さないと読めないのは変だと言ったのを覚えているでしょうか？では、`Collection` リソースの中に関数を追加して、NFT を借りられるようにしてみましょう：

```cadence
pub contract CryptoPoops {
  pub var totalSupply: UInt64

  pub resource NFT {
    pub let id: UInt64

    // ここから読み取れるように
    // さらにメタデータを追加しました。
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

    // この関数を追加し
    // NFT を読み込めるようにしました。
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

  // ... その他はこちら ...

  init() {
    self.totalSupply = 0
    self.account.save(<- create Minter(), to: /storage/Minter)
  }
}
```

超シンプルではないですか？！さらに、NFT に追加のフィールド（または「メタデータ」）を追加して、Collection から参照を借りたときにそのNFTに関する情報を読み取ることができるようにした。参照を取得するには、`Collection`  内で新しい `borrowNFT` 関数を使用して、`ownedNFTs` マップ内に格納されている NFT の 1 つへの参照を返します。コントラクトを再デプロイし、アカウントをセットアップし、NFT をミントするために新しいトランザクションを実行します：

```cadence
import CryptoPoops from 0x01

// name: "Jacob"
// favouriteFood: "Chocolate chip pancakes"
// luckyNumber: 13
transaction(recipient: Address, name: String, favouriteFood: String, luckyNumber: Int) {

  prepare(signer: AuthAccount) {
    // `Minter` への参照を取得する。
    let minter = signer.borrow<&CryptoPoops.Minter>(from: /storage/Minter)
                    ?? panic("This signer is not the one who deployed the contract.")

    // `recipient` のパブリックコレクションへの参照を取得する。
    let recipientsCollection = getAccount(recipient).getCapability(/public/MyCollection)
                                  .borrow<&CryptoPoops.Collection{CryptoPoops.CollectionPublic}>
                                  ?? panic("The recipient does not have a Collection.")

    // `Minter` への参照を使用して NFT を作成し、メタデータを渡します。
    let nft <- minter.createNFT(name: name, favouriteFood: favouriteFood, luckyNumber: luckyNumber)

    // NFT を受取人のコレクションに預ける。
    recipientsCollection.deposit(token: <- nft)
  }

}
```

すごい！受取人のアカウントに NFT がデポジットされました。では、スクリプトで新しい関数 `borrowNFT` を使って、アカウントにデポジットされた NFT のメタデータを読み取ってみましょう：

```cadence
import CryptoPoops from 0x01
pub fun main(address: Address, id: UInt64) {
  let publicCollection = getAccount(address).getCapability(/public/MyCollection)
              .borrow<&CryptoPoops.Collection{CryptoPoops.CollectionPublic}>()
              ?? panic("The address does not have a Collection.")

  let nftRef: &CryptoPoops.NFT = publicCollection.borrowNFT(id: id) // ERROR: "Member of restricted type is not accessible: borrowNFT"
  log(nftRef.name)
  log(nftRef.favouriteFood)
  log(nftRef.luckyNumber)
}
```

待ってください！エラーが出てます！なぜでしょう？ああ、コントラクト内の `CollectionPublic` インターフェースに `borrowNFT` を追加するのを忘れたからでした！では、修正しましょう：

```cadence
pub contract CryptoPoops {

  // ... その他はこちら ...

  pub resource interface CollectionPublic {
    pub fun deposit(token: @NFT)
    pub fun getIDs(): [UInt64]
    // ここでborrowNFT関数を追加しました
    // だから、一般の人もアクセスできます
    pub fun borrowNFT(id: UInt64): &NFT
  }

  // ... その他はこちら ...
}
```

これで、スクリプトを再試行することができます（ NFT のミントをやり直したと仮定して）：

```cadence
import CryptoPoops from 0x01
pub fun main(address: Address, id: id) {
  let publicCollection = getAccount(address).getCapability(/public/MyCollection)
              .borrow<&CryptoPoops.Collection{CryptoPoops.CollectionPublic}>()
              ?? panic("The address does not have a Collection.")

  let nftRef: &CryptoPoops.NFT = publicCollection.borrowNFT(id: id)
  log(nftRef.name) // "Jacob"
  log(nftRef.favouriteFood) // "Chocolate chip pancakes"
  log(nftRef.luckyNumber) // 13
}
```

やーーーーーー！NFT のメタデータをコレクションから削除することなく読むことができました ;)

## まとめ

私たちは今、本格的な NFT スマートコントラクトを書きました。超クールです。第4章も完成しました。

次の章では、このコントラクトを完成させ、コントラクトをより "正式なもの" にする方法を学びます。つまり、NFT スマートコントラクトが実際に NFT スマートコントラクトで _あること_ を他のアプリケーションが認識できるよう、コントラクトインターフェースと呼ばれるものを実装する方法です。

## クエスト

このチャプターでは話すことがたくさんありましたので、次のことをやってください：

これまでの NFT のコントラクトを参考に、リソースや機能のひとつひとつに、自分の言葉で何をしているのかを説明するコメントを付け加えてください。こんな感じです：

```cadence
pub contract CryptoPoops {
  pub var totalSupply: UInt64

  // これは、名前、favoriteFood、
  // luckyNumberを含むNFTリソースです。
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

  // これはリソースインターフェースであり、私たちが...要点を理解することを可能にします。
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
