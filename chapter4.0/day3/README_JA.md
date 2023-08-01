# 第4章3日目 - NFT コントラクトの作成：コレクション（パート1/3）

これまで多くのことを学んできました。学んだことをすべて応用して、自分の NFT コントラクトを作ってみましょう。

## ビデオ

次の数章では、このビデオで私がやっていることをそのままやっていきます。
今日は 00:00 から 20:35 までです：https://www.youtube.com/watch?v=bQVXSpg6GE8

## レビュー

<img src="../images/accountstorage1.PNG" />
<img src="../images/capabilities.PNG" />

## NFT (NonFungibleToken) の例

これから数日間、NonFungibleToken の例を見ていきます。CryptoPoops と呼ばれる独自の NFT コントラクトを作成します。こうすることで、これまでに学んだすべての概念を復習し、独自の NFT を実装することができます！

まずはコントラクトを作成することから始めます：

```cadence
pub contract CryptoPoops {
  pub var totalSupply: UInt64

  pub resource NFT {
    pub let id: UInt64

    init() {
      // 注意：Flow 上のすべてのリソースは、それぞれ固有の `uuid` を持ちます。
      // そこで 同じ `uuid` を持つリソースが存在することはありません。
      self.id = self.uuid
    }
  }

  pub fun createNFT(): @NFT {
    return <- create NFT()
  }

  init() {
    self.totalSupply = 0
  }
}
```

まずは、次のことから始めます：

1. `totalSupply` を定義します。（最初は0に設定します）
2. `NFT` 型を作成します。`NFT` には 1つのフィールド  `id` を与えます。`id` には `self.uuid` が設定されます。これは、すべてのリソースが Flow 上で持つ一意の識別子であるからです。同じ `uuid` を持つリソースが2つ存在することはないので、NFT の `id` として完璧に機能します。
3. `NFT ` リソースを返す `createNFT` 関数を作成し、誰でも自分の NFT を作成できるようにします。

よし、簡単ですね。NFT をアカウントストレージに保存し、一般公開できるようにしましょう。

```cadence
import CryptoPoops from 0x01
transaction() {
  prepare(signer: AuthAccount) {
    // NFT を `/storage/MyNFT` ストレージパスに保存します。
    signer.save(<- CryptoPoops.createNFT(), to: /storage/MyNFT)

    // 誰でも私の NFT の `id` フィールドを読むことができるように、それをパブリックにリンクします。
    signer.link<&CryptoPoops.NFT>(/public/MyNFT, target: /storage/MyNFT)
  }
}
```

いいですね！この章では、まず NFT をアカウントストレージに保存します。まず NFT をアカウントストレージに保存し、その参照をパブリックにリンクして、スクリプトでその `id` フィールドを読めるようにします。さて、そうしましょう！

```cadence
import CryptoPoops from 0x01
pub fun main(address: Address): UInt64 {
  let nft = getAccount(address).getCapability(/public/MyNFT)
              .borrow<&CryptoPoops.NFT>()
              ?? panic("An NFT does not exist here.")

  return nft.id // 3525 （リソースの `uuid` なので、何らかのランダムな数字です。
                // これはおそらく、あなたにとっては違うことでしょう。)
}
```

素晴らしい！私たちはいいことをしました。しかし、ちょっと考えてみましょう。もし私たちが_別の_ NFT をアカウントに保存したいとしたらどうなるでしょうか？

```cadence
import CryptoPoops from 0x01
transaction() {
  prepare(signer: AuthAccount) {
    // エラー： "オブジェクトの保存に失敗しました： パス /storage/MyNFT
    // にはすでにオブジェクトが保存されています。"
    signer.save(<- CryptoPoops.createNFT(), to: /storage/MyNFT)

    signer.link<&CryptoPoops.NFT>(/public/MyNFT, target: /storage/MyNFT)
  }
}
```

どうした？エラーが出たんだ！なぜか？そのストレージパスにはすでに NFT が存在するからです。どうすれば解決できるのでしょうか？まあ、別のストレージパスを指定すればいいのですが...

```cadence
import CryptoPoops from 0x01
transaction() {
  prepare(signer: AuthAccount) {
    // パスには `MyNFT02` を使っていることに注意。
    signer.save(<- CryptoPoops.createNFT(), to: /storage/MyNFT02)

    signer.link<&CryptoPoops.NFT>(/public/MyNFT02, target: /storage/MyNFT02)
  }
}
```

これは機能しますが、あまり良くありません。NFT を大量に持とうと思ったら、NFT に与えたすべてのストレージパスを覚えておかなければなりません。

2つ目の問題は、誰も NFT をくれないことです。アカウントの所有者だけが直接アカウントのストレージに NFT を保存できるため、誰も NFT をミントできません。これも良くありません。

### コレクション

この2つの問題を解決する方法は、 "Collection," つまりすべての NFT を1つにまとめるコンテナを作成することです。そして、Collection を1つのストレージパスに保存し、他の人がその Collection に "deposit(預ける)" ことができるようにします。


```cadence
pub contract CryptoPoops {
  pub var totalSupply: UInt64

  pub resource NFT {
    pub let id: UInt64

    init() {
      self.id = self.uuid
    }
  }

  pub fun createNFT(): @NFT {
    return <- create NFT()
  }

  pub resource Collection {
    // `id` をその `id` を持つ NFT にマップする。
    //
    // 例：2353 => ID 2353のNFT
    pub var ownedNFTs: @{UInt64: NFT}

    // NFTをコレクションに
    // 預けることができる。
    pub fun deposit(token: @NFT) {
      self.ownedNFTs[token.id] <-! token
    }

    // NFT を私たちのコレクションから
    // デポジットするができます。
    //
    // NFT が存在しないとパニックに陥ります
    pub fun withdraw(withdrawID: UInt64): @NFT {
      let nft <- self.ownedNFTs.remove(key: withdrawID)
              ?? panic("This NFT does not exist in this Collection.")
      return <- nft
    }

    // コレクション内のすべての NFT id の配列を返します。
    pub fun getIDs(): [UInt64] {
      return self.ownedNFTs.keys
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

  init() {
    self.totalSupply = 0
  }
}
```

すごいです。いくつかのことを行う `Collection` リソースを定義しました：

1. `id` とその `id` を持つ `NFT` を対応付ける `ownedNFTs` というマップを格納します。
2. `NFT` を預けるための `deposit` 関数を定義します。
3. `NFT` を引き出すための `withdraw` 関数を定義します。
4. コレクション内のすべての NFT id のリストを取得するための `getIDs` 関数を定義します。
5. `destroy` 関数を定義します。Cadence では、**リソースの中にリソースがある場合は必ず `destroy` 関数を宣言しなければいけません。**

また、`createEmptyCollection` 関数を定義して、 `Collection` をアカウントストレージに保存できるようにしましま。それではやってみましょう：

```cadence
import CryptoPoops from 0x01
transaction() {
  prepare(signer: AuthAccount) {
    // アカウントのストレージに `CryptoPoops.Collection` を保存します。
    signer.save(<- CryptoPoops.createEmptyCollection(), to: /storage/MyCollection)

    // パブリックとリンクさせます。
    signer.link<&CryptoPoops.Collection>(/public/MyCollection, target: /storage/MyCollection)
  }
}
```

そのコードを数分かけて本当に読んでみてください。何が問題なのでしょうか？セキュリティ上の問題について考えてみましょう。なぜ `&CryptoPoops.Collection` を公開することが悪いのでしょうか？

....

....

もう考えましたか？なぜかというと、**誰でも私たちのコレクションから引き出すことができるようになったからです！** それは本当に悪いことです。

しかし問題なのは、一般の人々が私たちのコレクションに NFT を `deposit` できるようにしたいし、私たちが所有する NFT の ID も読んでもらいたいということです。どうすればこの問題を解決できるでしょうか？

リソース・インターフェース 、おおおおおおおおお！公開するものを制限するために、リソースインターフェースを定義しましょう：

```cadence
pub contract CryptoPoops {
  pub var totalSupply: UInt64

  pub resource NFT {
    pub let id: UInt64

    init() {
      self.id = self.uuid
    }
  }

  pub fun createNFT(): @NFT {
    return <- create NFT()
  }

  // `deposit` と `getIDs` のみを公開します。
  pub resource interface CollectionPublic {
    pub fun deposit(token: @NFT)
    pub fun getIDs(): [UInt64]
  }

  // `Collection` は  `CollectionPublic` を実装するようになりました。
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

  init() {
    self.totalSupply = 0
  }
}
```

これで、コレクションをアカウントストレージに保存するときに、一般公開される内容を制限できるようになりました：

```cadence
import CryptoPoops from 0x01
transaction() {
  prepare(signer: AuthAccount) {
    // `CryptoPoops.Collection` をアカウントストレージに保存します。
    signer.save(<- CryptoPoops.createEmptyCollection(), to: /storage/MyCollection)

    // 注意： 我々は、`&CryptoPoops.Collection{CryptoPoops.CollectionPublic}` を
    // 公開します。これは、`deposit` と `getIDs` のみを含みます。
    signer.link<&CryptoPoops.Collection{CryptoPoops.CollectionPublic}>(/public/MyCollection, target: /storage/MyCollection)
  }
}
```

<img src="../images/thanos.png" />
さて、これは...私の顔に笑顔をもたらします。NFT をアカウントに預け、引き出して実験してみましょう。

```cadence
import CryptoPoops from 0x01
transaction() {
  prepare(signer: AuthAccount) {
    // `CryptoPoops.Collection` への参照を取得します。
    let collection = signer.borrow<&CryptoPoops.Collection>(from: /storage/MyCollection)
                      ?? panic("The recipient does not have a Collection.")

    // コレクションに `NFT` を預ける
    collection.deposit(token: <- CryptoPoops.createNFT())

    log(collection.getIDs()) // [2353]

    // コレクションから `NFT` を引き出します
    let nft <- collection.withdraw(withdrawID: 2353) // 上記の id 配列からこの数字を取得します。

    log(collection.getIDs()) // []

    destroy nft
  }
}
```

素晴らしいです！すべてがうまくいっています。では、自分たちでデポジットする代わりに、誰かが私たちのコレクションにデポジットしてくれるかどうか見てみましょう：

```cadence
import CryptoPoops from 0x01
transaction(recipient: Address) {

  prepare(otherPerson: AuthAccount) {
    // `recipient` のパブリック Collection への参照を取得します。
    let recipientsCollection = getAccount(recipient).getCapability(/public/MyCollection)
                                  .borrow<&CryptoPoops.Collection{CryptoPoops.CollectionPublic}>()
                                  ?? panic("The recipient does not have a Collection.")

    // コレクションに `NFT` を預ける
    recipientsCollection.deposit(token: <- CryptoPoops.createNFT())
  }

}
```

いいいいいね！`&CryptoPoops.Collection{CryptoPoops.CollectionPublic}` をパブリックにリンクさせたので、他の誰かのアカウントにデポジットしました。そして、これは問題ありません。誰かに無料で NFT を提供しても、誰が気にしますか？それは素晴らしいことです！

Now, what happens if we try to withdraw from someone's Collection?

```cadence
import CryptoPoops from 0x01
transaction(recipient: Address, withdrawID: UInt64) {

  prepare(otherPerson: AuthAccount) {
    // `recipient` のパブリックコレクションへの参照を取得する。
    let recipientsCollection = getAccount(recipient).getCapability(/public/MyCollection)
                                  .borrow<&CryptoPoops.Collection{CryptoPoops.CollectionPublic}>()
                                  ?? panic("The recipient does not have a Collection.")

    // エラー: "制限された型のメンバーにはアクセスできません: 引き出し"
    recipientsCollection.withdraw(withdrawID: withdrawID)
  }

}
```

エラーが出ました！完璧です。ハッカーは我々の NFT を盗めない :)

最後に、スクリプトを使ってアカウント内の NFT を読み取ってましょう：

```cadence
import CryptoPoops from 0x01
pub fun main(address: Address): [UInt64] {
  let publicCollection = getAccount(address).getCapability(/public/MyCollection)
              .borrow<&CryptoPoops.Collection{CryptoPoops.CollectionPublic}>()
              ?? panic("The address does not have a Collection.")

  return publicCollection.getIDs() // [2353]
}
```

どーん。完了。

## まとめ

Collection は NFT だけのものではありません。Flow エコシステムのあらゆる場所で、Collection の概念が使用されています。ユーザーがリソースを保存したいが、そのリソースを複数持っている可能性がある場合、ほとんどの場合、Collection を使用してリソースを囲み、1つの場所にすべてを保存できるようにします。これは、理解すべき非常に重要な概念です。

そして、自分自身に拍手を送りましょう！あなたは NFT コントラクトを機能させました！よくやった、友よ！すぐに私に追いつくでしょう。冗談です。僕は君よりずっとうまいんだ。

## クエスト

1. なぜこのコントラクトにコレクションを追加したのですか？主な理由を2つ挙げてください。

2. リソースの中に別のリソースが「入れ子」になっている場合、どうすればいいのでしょうか？(入れ子のリソース)

3. このコントラクトに追加したい事項をいくつかブレインストーミングします。このコントラクトの何が問題で、どうすればそれを解決できるかを考えます。

   - アイデアその1：本当に全員が NFT をミントできるようにしたいのですか？🤔

   - アイデアその2：コレクション内の NFT に関する情報を読みたい場合、今はコレクションから取り出さなければなりません。これは良いことですか？
