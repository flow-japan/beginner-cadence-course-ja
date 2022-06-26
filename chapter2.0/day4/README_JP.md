# 第 2 章 4 日目 - 基本構造体

## 動画

1. (構造体＋ディクショナリ＋オプショナル） - この動画の 12:10 から最後までを見てください: [https://www.youtube.com/watch?v=LAUN7hqlL0w](https://www.youtube.com/watch?v=LAUN7hqlL0w)

## 構造体

構造体とは何ですか？構造体は、他の型のコンテナです。例を見てみましょう。

```cadence
pub struct Profile {
    pub let firstName: String
    pub let lastName: String
    pub let birthday: String
    pub let account: Address

    // `init()` gets called when this Struct is created...
    // You have to pass in 4 arguments when creating this Struct.
    init(_firstName: String, _lastName: String, _birthday: String, _account: Address) {
        self.firstName = _firstName
        self.lastName = _lastName
        self.birthday = _birthday
        self.account = _account
    }
}
```

なるほど、突然長いのが来ました...何が起こったのでしょうか？我々は、`Profile` という新しい型を定義しました。これは構造体です。見ての通り、4 つのデータを含んでいます。

1. 名前 (`firstName`)
2. 苗字 (`lastName`)
3. 誕生日 (`birthday`)
4. アカウントのアドレス (`account`)

情報をコンテナに集めたいときに、構造体を作っておくと本当に便利です。

なぜこれが便利なのか考えてみましょう。例えば、Flow プレイグラウンドで新しいスクリプトを作成し、誰かのプロフィール情報を返したいとします。どうすればいいのでしょう？構造体がなければ、すべての情報を含む文字列の配列（`[String]`）を返し、`account` パラメータを `String` に変換するなどしなければならないでしょう。これは大変な労力と苦痛です。その代わり、Profile の構造体を返すようにすればよいのです。では、実際の例を見てみましょう。

構造体には `init()` 関数があり、構造体の生成時に呼び出されます。これは、コントラクトのデプロイ時に呼び出される `init()` 関数と同じです。さらに、私は `init()` 関数の中で変数名の前に "_" を使う傾向があることにお気づきでしょう。これは、実際の変数と初期化された値を区別するために行っていることです。これは暗黙の引数ラベル構文 `_` と同じではありません。そうなんです、私は分かりにくい男なんです。

**重要**: 構造体は `pub` アクセス修飾子しか使えません (これについては後日説明します)。では、実際の例を見てみましょう。

## 実際の例

まず、アカウント `0x01` に新しいスマートコントラクトをデプロイしてみましょう。

```cadence
pub contract Authentication {

    pub var profiles: {Address: Profile}

    pub struct Profile {
        pub let firstName: String
        pub let lastName: String
        pub let birthday: String
        pub let account: Address

        // You have to pass in 4 arguments when creating this Struct.
        init(_firstName: String, _lastName: String, _birthday: String, _account: Address) {
            self.firstName = _firstName
            self.lastName = _lastName
            self.birthday = _birthday
            self.account = _account
        }
    }

    pub fun addProfile(firstName: String, lastName: String, birthday: String, account: Address) {
        let newProfile = Profile(_firstName: firstName, _lastName: lastName, _birthday: birthday, _account: account)
        self.profiles[account] = newProfile
    }

    init() {
        self.profiles = {}
    }

}
```

私はここであなたに多くのことを投げかけました。ただ、すでに知っていることばかりです。それを分解してみましょう。

1. `Authentication` という名前の新しいコントラクトを定義しました。
2. `Address` 型を `Profile` 型にマップする `profiles` という名前のディクショナリを定義しました。
3. 4 つのフィールドを含む `Profile` という名前の新しい構造体を定義しました。
4. `addProfile` という名前の新しい関数を定義しました。この関数は 4 つの引数を受け取り、その引数で新しい `Profile` を生成します。そして、 `account` -> そのアカウントに関連付けられた `Profile` という新しいマッピングを生成します。
5. コントラクトがデプロイされると、`profiles` を空のディクショナリで初期化します。

これらのことが理解できれば、かなりの進歩です。ちょっと苦戦している人も、心配いりませんよ。私なら、ここ数日のコンセプトを少し見直すかもしれません。そして、`pub` の意味はまだわからなくてもいいことを覚えておいてください。

### 新しい Profile を追加する

さて、新しい構造体を定義したところで、なぜそれが役に立つのかを見てみましょう。

新しいトランザクションを開き、このお決まりのトランザクションコードをコピー＆ペーストしてみましょう。

```cadence
import Authentication from 0x01

transaction() {

    prepare(signer: AuthAccount) {}

    execute {
        log("We're done.")
    }
}
```

素晴らしい！ここで、`Authentication` コントラクトの `profiles` ディクショナリに新しいプロファイルを追加したいと思います。どうすれば良いでしょう？以下のように必要な引数をすべて指定して、`addProfile` 関数を呼び出してみましょう: `Authentication.addProfile(firstName: firstName, lastName: lastName, birthday: birthday, account: account)`。しかし待ってください、これらの引数をまずどこかから取得する必要があります! そのためには、トランザクションに引数として渡します。

```cadence
import Authentication from 0x01

transaction(firstName: String, lastName: String, birthday: String, account: Address) {

    prepare(signer: AuthAccount) {}

    execute {
        Authentication.addProfile(firstName: firstName, lastName: lastName, birthday: birthday, account: account)
        log("We're done.")
    }
}
```

このトランザクションを任意のアカウントで実行し、いくつかのサンプルデータを以下のように渡してみましょう。

![https://github.com/emerald-dao/beginner-cadence-course/raw/main/chapter2.0/images/txstuff.png](https://github.com/emerald-dao/beginner-cadence-course/raw/main/chapter2.0/images/txstuff.png)

### Profile を読む

新しい Profile を読み込むために、スクリプトを開き、お決まりのスクリプトコードをコピー＆ペーストしてみましょう。

```cadence
import Authentication from 0x01

pub fun main() {

}
```

それでは、Profile を読み込んでみましょう。これはアカウントを表す `Address` を渡すことで実現できます。コントラクトの `profiles` ディクショナリでアカウントにプロファイルをマッピングしているからです。そして、そのディクショナリから取得した `Profile` 型を以下のように返します。

```cadence
import Authentication from 0x01

pub fun main(account: Address): Authentication.Profile {
    return Authentication.profiles[account]
}
```

お、ちょっと待ってください! エラーです：「mismatched types. expected `Authentication.Profile`, got `Authentication.Profile?`」。まあ、昨日の内容で修正する方法はわかっていますよね？こんな感じで強制アンラップ演算子を追加する必要があります。

```cadence
import Authentication from 0x01

pub fun main(account: Address): Authentication.Profile {
    return Authentication.profiles[account]!
}
```

戻り値の型に注目してください。 `Authentication.Profile` です。これは `Authentication` のコントラクトで定義された `Profile` 型を返しているからです。型は常に定義されたコントラクトに基づきます。これで終わりです。これで、このスクリプトを呼び出した人は、必要なすべてのプロファイル情報を手に入れることができます。いやー、構造体ってすごいですねー。

## クエスト

1. 新しいコントラクトをデプロイして、その中に好きな構造体を入れてください。（必ず `Profile` とは別のものでお願いします。）
2. 定義した構造体を含むディクショナリまたは配列を作成します。
3. その配列/ディクショナリに追加の関数を作成します。
4. 手順 3 で作成した関数を呼び出すためのトランザクションを追加します。
5. 定義した構造体を読み込むスクリプトを追加してください。

以上です。では、また明日 ;)
