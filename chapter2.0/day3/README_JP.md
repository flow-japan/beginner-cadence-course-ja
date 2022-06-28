# 第 2 章 3 日目 - 配列、ディクショナリ、オプショナルズ

Cadence 初学者の皆さん、こんにちは。今日は、これから書くほぼすべてのコントラクトで使う、いくつかの最も重要な型を学びます。

## 動画

この内容は…

1. (Cadence における配列とディクショナリ) この動画の 00:00-12:10 をご覧ください。12:10 以降は明日の内容なので見る必要はありません：[https://www.youtube.com/watch?v=LAUN7hqlL0w](https://www.youtube.com/watch?v=LAUN7hqlL0w)
2. (Cadence におけるオプショナル) この動画をご覧ください: [https://www.youtube.com/watch?v=I9Z1z9BsZ0I](https://www.youtube.com/watch?v=I9Z1z9BsZ0I)

## 型

型で遊び始めるために、Flow プレイグラウンド（[https://play.onflow.org](https://play.onflow.org/)）を開き、スクリプトを開いてみましょう。なお、本日はスマートコントラクトを書きません :)

Cadence では、あなたが書いたコードから、どのような型であるかが推測できることがよくあります。例えば、

```cadence
var jacob = "isCool"`
```

と書けば、Cadence は自動的に String を初期化したことを認識します。しかし、より明示的に型を示したい場合は、次のように宣言に型を含めることができます。

```cadence
var jacob: String = "isCool"`
```

このように型を記述しておくと、プログラムのどこで間違ったのかを推論するのに便利です。また、Cadence は変数が異なる型であることを意図している場合、あなたが間違いを犯したことを率直に教えてくれるでしょう。例えば、次のようにタイプしてみましょう。

```cadence
var jacob: String = 3
```

すると、Cadence は「おい！この型は一致しないぞ。というようなことを言うでしょう。しかし、重要なのは、どこで間違ったのかを理解するために、型を含めることができるということです。

## 配列

配列とは何でしょうか？配列は要素のリストです。Cadence の非常に基本的な配列を見てみましょう。

```cadence
var people: [String] = ["Jacob", "Alice", "Damian"]`
```

これは文字列のリストです。配列の型はこのように宣言します。 `[型]` です。別の例も見てみましょう。アドレスのリストが欲しかったら、まあ、似たようなものです。

```cadence
var addresses: [アドレス] = [0x1, 0x2, 0x3]
```

といった具合です。

また、配列にインデックスを付けて、その要素が何であるかを確認することもできます。これは、JavaScript やそれに似た言語と全く同じです。

```cadence
var addresses: [Address] = [0x1, 0x2, 0x3]
log(addresses[0]) // 0x1
log(addresses[1]) // 0x2
log(addresses[2]) // 0x3
```

### いつも使っている便利な配列の関数

上で見てきたものはすべて固定長の配列です。配列を使って面白いことができるので、ここでそのいくつかを共有します。

**append(\_ element: Type)**

（引数ラベルである `element` の前に `_` があることに注目。これは暗黙的であることを示すもので、こうすると関数を呼び出すときに引数ラベルを付ける必要がなくなります。つまり、`.append(element: value)` の代わりに、 `.append(value)` としてよいのです。）

配列の末尾に要素を追加します。

例：

```cadence
var people: [String] = ["Jacob", "Alice", "Damian"]
people.append("Ochako Unaraka") // anyone watch My Hero Academia? :D
log(people) // ["Jacob", "Alice", "Damian", "Ochako Unaraka"]
```

**contains(_ element_: Type): Bool**

配列に要素があるかどうかを調べます。

例：

```cadence
var people: [String] = ["Jacob", "Alice", "Damian"]
log(people.contains("Jacob")) // true
log(people.contains("Poop")) // false
```

**remove(at: Int)**

指定されたインデックスの要素を削除します（インデックスは 0 から始まり、最初の要素のインデックスは 0 です）。

例：

```cadence
var people: [String] = ["Jacob", "Alice", "Damian"]
people.remove(at: 1)
log(people) // ["Jacob", "Damian"]
```

**length**

配列の長さを返します。

例：

```cadence
var people: [String] = ["Jacob", "Alice", "Damian"]
log(people.length) // 3
```

## ディクショナリ

今までは配列の話をしてきましたが、ここからはディクショナリの番です。さて、これは何でしょう？ディクショナリとは、キーと値をマッピングさせるものです。以下、簡単な例を見てみましょう。

```cadence
var names: {String: String} = {"Jacob": "Tucker", "Bob": "Vance", "Ochako": "Unaraka"} // anyone watch The Office?
```

上記の例では、`String` を `String` にマッピングしています．具体的には、誰かの姓と名をマッピングしています。これは、ディクショナリ型 `{Type: Type}` を使っています。このディクショナリを使って、次のように人々の姓を取得できます。

```cadence
var names: {String: String} = {"Jacob": "Tucker", "Bob": "Vance", "Ochako": "Unaraka"}
log(names["Jacob"]) // "Tucker"
log(names["Bob"]) // "Vance"
log(names["Ochako"]) // "Unaraka"
```

`String` を `Int` にマッピングする例を見てみましょう。ここでは、ある人の名前と好きな番号を対応させます。

```cadence
var favouriteNums: {String: Int} = {"Jacob": 13, "Bob": 0, "Ochako": 1000100103}
log(favouriteNums["Jacob"]) // 13
```

これはとっても便利なものですが、もっと複雑な機能があります。なぜディクショナリが複雑なのかは、一番下の「オプショナルとディクショナリ」のセクションで説明します。その前に、ディクショナリに役立つ関数をいくつか見てみましょう。

### いつも使っている便利なディクショナリの関数

**insert(key: Type, \_ value: Type)**

（`value` の引数ラベルは暗黙的ですが、`key` はそうでないことに注意してください）

例：

```cadence
var favouriteNums: {String: Int} = {"Jacob": 13, "Bob": 0, "Ochako": 1000100103}
favouriteNums.insert(key: "Justin Bieber", 1)
log(favouriteNums) // {"Jacob": 13, "Bob": 0, "Ochako": 1000100103, "Justin Bieber": 1}
```

**remove(key: Type): Type?**

`key`とそれに関連する値を削除し、その値を返す。

例：

```cadence
var favouriteNums: {String: Int} = {"Jacob": 13, "Bob": 0, "Ochako": 1000100103}
let removedNumber = favouriteNums.remove(key: "Jacob")
log(favouriteNums) // {"Bob": 0, "Ochako": 1000100103}
log(removedNumber) // 13
```

**keys: [Type]**

ディクショナリに含まれるすべてのキーの配列を返します。

例：

```cadence
var favouriteNums: {String: Int} = {"Jacob": 13, "Bob": 0, "Ochako": 1000100103}
log(favouriteNums.keys) // ["Jacob", "Bob", "Ochako"]
```

**values: [Type]**

ディクショナリに登録されているすべての値の配列を返す。

例：

```cadence
var favouriteNums: {String: Int} = {"Jacob": 13, "Bob": 0, "Ochako": 1000100103}
log(favouriteNums.values) // [13, 0, 1000100103]
```

## オプショナル

さて、では次はオプショナルです。いやー。オプショナルはとても重要ですが、厄介なものです。おそらく Cadence で行うすべての作業において、オプショナルに遭遇することになるでしょう。ほとんどの場合、それはディクショナリのせいです。

Cadence の `オプショナル型` は `?` で表されます。これは「`その型` か `nil` かのどちらかである」ということを意味します。日本語っで頼むって？？まぁまぁ、以下のコードを見てください。

```cadence
var name: String? = "Jacob"
```

`String` の後ろの `?` に注意してください。これはつまり、変数 `name` は、`String` か `nil` である、ということです。明らかに、`Jacob` は `String` であることがわかります。しかし、次の場合はどうでしょう？

```cadence
var name: String? = nil
```

これも正しくて、コンパイルエラーは起こりません。`String?` 型は `nil` になることができます。

悪くない説明でしょ？僕は最高の教師だよ。みなさんは私と出会えてラッキーだと思いませんか？

### 強制アンラップ演算子

強制アンラップ演算子 `!` の説明に入ります。この演算子は、オプショナル型を「アンラップ」してこう宣言します。「もしこれが `nil` ならば `PANIC!`、もしこれが `nil` でないなら大丈夫。」さて、これは一体どういう意味なのでしょうか？見てみましょう。

```cadence
var name1: String? = "Jacob"
var unwrappedName1: String = name1! // オプショナル型を定義
var name2: String? = nil
var unwrappedName2: String = name2! // PANICS! 問題が見つかったので、プログラム全体がアボートします。nilをアンラップしようとしたが，これは許されない。
```

## オプショナルとディクショナリ

さて、ここで私たちが知っているすべてのことを組み合わせて、オプショナルとディクショナリについてお話します。以前、ディクショナリについて説明したとき、重要な情報を省きました。ディクショナリの要素にアクセスすると、その値は **オプショナル** として返されます。これはどういうことでしょうか？以下を見てみましょう。

例えば、次のようなディクショナリがあるとします。

```cadence
let thing: {String: Int} = {"Hi": 1, "Bonjour": 2, "Hola": 3}
```

「Bonjour」キーにマッピングされた値を出力する場合…

```cadence
let thing: {String: Int} = {"Hi": 1, "Bonjour": 2, "Hola": 3}
log(thing["Bonjour"]) // これは 2 が出力される
```

上で紹介したように 2 が表示されます。だから、変には見えません。しかし、実際にはちょっとおかしくないでしょうか？このようなスクリプトを新しく作ってみましょう。

```cadence
pub fun main(): Int {
    let thing: {String: Int} = {"Hi": 1, "Bonjour": 2, "Hola": 3}
    return thing["Bonjour"] // エラー: "Mismatched types. expected `Int`, got `Int?`"
}
```

これはエラーとなります！このエラー「Mismatched types. expected `Int`, got `Int?`」は、型の不一致です。`Int` のはずが、`Int?` が来ました。すでに `Int?` が何を意味するかがわかりましたね。つまり、オプショナル（任意）なので、`Int` かもしれないし、`nil` かもしれない。このエラーを修正するためには、強制アンラップ演算子を使います。このように：

```cadence
pub fun main(): Int {
    let thing: {String: Int} = {"Hi": 1, "Bonjour": 2, "Hola": 3}
    return thing["Bonjour"]! // 強制アンラップ演算子を追加
}
```

これで、エラーは出なくなりました :D

### オプショナルを返す vs. アンラップする

オプショナルを強制的にアンラップする代わりに、オプショナルを返したいケースはあるでしょうか? 答えはイエスです。実際、ほとんどの場合、アンラップする代わりにオプショナルを返すことが望ましいです。例えば、このコードを見てみましょう。

```cadence
pub fun main(): Int {
    let thing: {String: Int} = {"Hi": 1, "Bonjour": 2, "Hola": 3}
    return thing["Bonjour"]! // 強制アンラップ演算子
}
```

「Bonjour」キーに値がない場合、これはパニックになり、プログラムを中断します。その代わりに、次のようなコードを書くことができます。

```cadence
pub fun main(): Int? { //戻り値にオプショナル型を定義
    let thing: {String: Int} = {"Hi": 1, "Bonjour": 2, "Hola": 3}
    return thing["Bonjour"] // オプショナルのまま戻す
}
```

こうすることで、クライアント/呼び出し側は、プログラム中のエラーを気にすることなく、戻り値が `nil` の場合を処理できます。この同じロジックは、Cadence コードの他の関数にも適用されます。

### ポイント

主なポイントは、ディクショナリの値にアクセスするときは、常にオプショナルが返ってくるということです。したがって、オプショナルではなく _実際の型_ が欲しい場合は、強制アンラップ演算子 `!` を使って「アンラップ」しなければなりません。

## クエスト

1. スクリプトの中で、あなたの好きな人を文字列で表現した配列 (長さ == 3) を初期化し、それを `log` してください。
2. スクリプト内で、Facebook, Instagram, Twitter, YouTube, Reddit, LinkedIn の各文字列を、使用頻度の高い順に `UInt64` にマップしたディクショナリを初期化してください。例えば、YouTube --> 1, Reddit --> 2, など。一度も使ったことがない場合は、0 にマッピングしてください。
3. 強制アンラップ演算子 `!` が何をするものなのか、私とは別の例で説明してください (型を変えれば OK)。
4. 以下の絵を使って、これらを説明してください：
   - エラーメッセージの意味
   - なぜこのエラーが発生するのか
   - どうすれば直るのか
