# 第6章1日目 - テストネットアカウントの作成とテストネットへのデプロイメント
***
オタク諸君。今日のレッスンでは、新しいテストネットアカウントを作成し、NFT コントラクトを Flow テストネットにデプロイする方法を学びます。
## Cadence VSCode Extension のインストール
***
*VSCode をインストールしたことがない方は、こちらでインストールすることができます：*
https://code.visualstudio.com/

プレイグラウンドがなくなった今、Cadence のコーディングの際に、VSCode にエラーを表示させたいです。そのための拡張機能があるんです！

>VSCode を開きます。VSCode の左側に、四角が 4 つ並んだようなアイコンがあります。それをクリックして"Cadence"と検索してください。

>以下の拡張機能をクリックし、"Install"をクリックしてください：

![MyzmyT43CugKPbzNwha6S4c9TLZ2/20MxJv729o3w1LI8.jpg](https://firebasestorage.googleapis.com:443/v0/b/type-c1c71.appspot.com/o/MyzmyT43CugKPbzNwha6S4c9TLZ2%2F20MxJv729o3w1LI8.jpg?alt=media&token=54da57ef-b57a-4b77-8cba-f1cdea574138)

## Flow CLI と flow.json のインストール
***
Flow CLI は、ターミナルからトランザクションとスクリプトを実行し、コントラクトのデプロイなど、Flow の他の機能を実行できるようにするものです。

>[Flow CLI](https://docs.onflow.org/flow-cli/install/)をインストールします。次の方法で実行できます：

**Mac**
* ターミナルに `sh -ci "$(curl -fsSL https://storage.googleapis.com/flow-cli/install.sh)"` を貼り付けます

**Windows**
* PowerShell に `iex "& { $(irm 'https://storage.googleapis.com/flow-cli/install.ps1') }"` を貼り付けます

**Linux**
* ターミナルに `sh -ci "$(curl -fsSL https://storage.googleapis.com/flow-cli/install.sh)"` を貼り付けます

Flow CLI がインストールされているかどうかは、ターミナルで `flow version` と入力することで確認できます。バージョンが表示されれば OK です。

## Flow フォルダ

ベースディレクトリの中に、`flow` という新しいフォルダを作りましょう。

`flow` フォルダの中に、`cadence` というフォルダをもう一つ作りましょう。

`cadence` フォルダの中に、`contract` フォルダ、`transactions` フォルダ、`scripts` フォルダを作りましょう。

`contracts` フォルダの中に、`CONTRACT_NAME.cdc` というファイルを新規に追加してください。CONTRACT_NAME は、あなたのコントラクトの名前に置き換えてください。そのファイルに、第5章で作成したコントラクトコードを入れます。このレッスンでは、このコントラクトを"ExampleNFT"と呼びますが、必ずあなた自身のコントラクトの名前に置き換えてください。

一番上にある、ランダムな Flow プレイグラウンドのアドレスの代わりに、ローカルファイルのパスからインポートする必要があることに注意してください。`0x01` からインポートすることはもうありません。それは単なるプレイグラウンドの問題でした。この場合、プロジェクトに存在するローカルコントラクトからインポートしています。

>一番上のインポートを次のように変更します： `import NonFungibleToken from "./NonFungibleToken.cdc"`

これを動作させるには、`NonFungibleToken` コントラクトインターフェースを `contracts` フォルダに追加する必要があります。ファイル名は  `NonFungibleToken.cdc` にしてください。

***

トランザクションフォルダ内に、`TRANSACTION_NAME.cdc` というファイル群を作成します。TRANSACTION_NAME をトランザクションの名前に置き換えてください。

インポートもすべて間違っていることに注意してください。`0x01` からインポートすることはもうありません。それは単なる遊び心でした。この場合、プロジェクトに存在するローカルコントラクトからインポートしています。そこで、インポートを次のような形式に変更してください。

```import ExampleNFT from "../contracts/ExampleNFT.cdc"```

***

scripts フォルダの中に、`SCRIPT_NAME.cdc` という名前のファイル群を作ってください。SCRIPT_NAME をあなたのスクリプトの名前に置き換えてください。

***

### flow.json

>プロジェクトディレクトリにコントラクトができたので、ターミナルから `cd` でベースプロジェクトディレクトリに移動します。

>タイプ `flow init`

これにより、プロジェクト内に `flow.json` ファイルが作成されます。これはコントラクトをデプロイしたり、Cadence のコード内でコンパイルエラーを出すために必要なものです。

## NFTコントラクトのテストネットへの展開
***
素晴らしいです！では、このコントラクトをテストネットにデプロイして、対話できるようにしましょう。
## `flow.json` の設定
***
> `flow.json` ファイルの中で、「コントラクト」オブジェクトを次のようにします。

```json
"contracts": {
  "ExampleNFT": "./contracts/ExampleNFT.cdc",
  "NonFungibleToken": {
    "source": "./contracts/NonFungibleToken.cdc",
    "aliases": {
      "testnet": "0x631e88ae7f1d7c20"
    }
  }
},
```

>"ExampleNFT" をコントラクト名と同じ名前に置き換えてください。


これにより、`flow.json` はあなたのコントラクトがどこにあるのかを知ることができます。なお、`NonFungibleToken` は Flow テストネットにすでに存在しており、そのためより複雑に見えます。

### アカウントの作成

>🔐 ターミナルに `flow keys generate --network=testnet` と入力し、デプロイアアドレスを生成します。公開鍵と秘密鍵は必ずどこかに保存しておいてください。

![MyzmyT43CugKPbzNwha6S4c9TLZ2/PW4O1So2eYOsZCWb.jpg](https://firebasestorage.googleapis.com:443/v0/b/type-c1c71.appspot.com/o/MyzmyT43CugKPbzNwha6S4c9TLZ2%2FPW4O1So2eYOsZCWb.jpg?alt=media&token=7c755a1c-be8b-4cb9-88e8-9b350040afeb)

> 👛 https://testnet-faucet.onflow.org/ にアクセスし、上記の公開鍵を貼り付けて、`CREATE ACCOUNT` をクリックして、デプロイアアカウントを作成します：

![MyzmyT43CugKPbzNwha6S4c9TLZ2/TTe5XCOnFVwl8rIX.jpg](https://firebasestorage.googleapis.com:443/v0/b/type-c1c71.appspot.com/o/MyzmyT43CugKPbzNwha6S4c9TLZ2%2FTTe5XCOnFVwl8rIX.jpg?alt=media&token=0d8aba35-2276-4d72-a387-7bdb5c693757)

> 終了後、`COPY ADDRESS` をクリックし、そのアドレスをどこかに保存しておいてください。必ず必要です！

> ⛽️ 以下のコードを修正して、`flow.json` に新しいテストネットのアカウントを追加します。"YOUR GENERATED ADDRESS"と書いてあるところに上でコピーしたアドレスを貼り付け、 "YOUR PRIVATE KEY"と書いてあるところに秘密鍵を貼り付けます。

```json
"accounts": {
  "emulator-account": {
    "address": "f8d6e0586b0a20c7",
    "key": "5112883de06b9576af62b9aafa7ead685fb7fb46c495039b1a83649d61bff97c"
  },
  "testnet-account": {
    "address": "YOUR GENERATED ADDRESS",
    "key": {
      "type": "hex",
      "index": 0,
      "signatureAlgorithm": "ECDSA_P256",
      "hashAlgorithm": "SHA3_256",
      "privateKey": "YOUR PRIVATE KEY"
    }
  }
},
"deployments": {
  "testnet": {
    "testnet-account": [
      "ExampleNFT"
    ]
  }
}
```

> "ExampleNFT"をコントラクトの名前に変更してください。

> 🚀"ExampleNFT"スマートコントラクトをデプロイします：

```sh
flow project deploy --network=testnet
```
![MyzmyT43CugKPbzNwha6S4c9TLZ2/3hbASbM0OYnNMGLa.jpg](https://firebasestorage.googleapis.com:443/v0/b/type-c1c71.appspot.com/o/MyzmyT43CugKPbzNwha6S4c9TLZ2%2F3hbASbM0OYnNMGLa.jpg?alt=media&token=e78820c8-638a-42f6-9526-d2aa977ab408)
## クエスト
1. https://flow-view-source.com/testnet/ にアクセスします。"Account"と書いてあるところに、作成した Flowアドレス を貼り付けて、"Go"をクリックします。左側に、あなたの NFT コントラクトが表示されるはずです。テストネットでライブで見られるなんて、とてもクールだと思いませんか？そして、そのページの URL を送信します。

・例：https://flow-view-source.com/testnet/account/0x90250c4359cebac7/
