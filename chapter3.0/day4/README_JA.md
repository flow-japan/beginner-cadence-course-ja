# 第3章4日目 - リソース/構造 インターフェイス

よー！よお、よお、よお！今日も cadence を楽しもう！
今日はリソースインターフェースについて学びます。

## ビデオ

リソースインターフェイスに関するビデオをご覧になりたい方は、こちらをクリックしてください。: https://www.youtube.com/watch?v=5wnn9qsCXgE

## インターフェイスとは何か？

インターフェイスは、従来のプログラミング言語では非常に一般的なものです。インターフェイスの用途は主に2つあります：

1. 何かを実装するための一連の要件を指定します。
2. 特定の人に特定のものだけを公開することができます。

どういうことかを理解するために、いくつかのコードを見てみましょう。

<img src="../images/interfaces.png" />

## インターフェイスを要件として使用する

このレッスンでは、リソースインターフェースだけを使いますが、**構造体インターフェースも、構造体用というだけで、まったく同じものです。笑** そのことだけ覚えておいてください。

Cadence では、リソース/構造体インターフェースは基本的に「要件」であり、リソース/構造体からデータを公開する方法です。インターフェース単体では何もしません。ただそこに座っているだけです。しかし、リソース/構造体にインターフェイスを _適用_ することで、何かができるようになります。

リソースインターフェースは `resource interface` キーワードで定義します。（構造体の場合は `struct interface` です）:

```cadence
pub contract Stuff {

    pub resource interface ITest {

    }

    pub resource Test {
      init() {
      }
    }
}
```

上の例では、2つのことをしているのがわかるでしょう：

1. 空の `ITest` という名前の `resource interface` を定義しました。
2. 空の `Test` という名前の `resource` を定義しました。

個人的には、インターフェイスの名前にはいつも頭に "I" を付けます、なぜなら、それが実際に何であるかを判断するのに役立つからです。

上の例では、`ITest` は実際には何もしません。ただそこに座っているだけです。これに何か追加してみましょう。

```cadence
pub contract Stuff {

    pub resource interface ITest {
      pub let name: String
    }

    pub resource Test {
      init() {
      }
    }
}
```

これで `ITest` は `name` フィールドを含むようになりました。クールですね！しかし、ITest はまだ何もしていません。ただ空間に座っているだけです。そこで、`ITest` リソースインターフェイスを `Test` に _実装_ してみましょう。

```cadence
pub contract Stuff {

    pub resource interface ITest {
      pub let name: String
    }

    // ERROR:
    // `resource Stuff.Test does not conform
    // to resource interface Stuff.ITest`
    pub resource Test: ITest {
      init() {

      }
    }
}
```

今やったことに注目してほしいです。`: ITest` 構文を追加することで、`Test` に `ITest` を実装させたのです。この意味は、「このリソースは `:` の後にリソースインターフェースを実装している」ということです。

しかし、エラーがあることにお気づきでしょう：「リソース Stuff.Test はリソースインターフェース Stuff.ITest に適合していません」。上で言ったことを覚えているでしょうか？リソースインターフェースは _要求_ です。リソースがリソースインタフェースを実装している場合、リソースはそのインタフェースにあるものを定義しなければなりません。修正しましょう。

```cadence
pub contract Stuff {

    pub resource interface ITest {
      pub let name: String
    }

    // It's good now :)
    pub resource Test: ITest {
      pub let name: String
      init() {
        self.name = "Spongebob" // anyone else like Spongebob?
      }
    }
}
```

今度はエラーはないです！わーい！

## インターフェイスを使って特定のものを公開する

上記では、リソース・インターフェースはリソースに特定のものを実装させるということを学びました。しかし、リソースインターフェースは実はそれ以上に重要です。2番目のことを思い出してください。私たちはこう言いました：「特定の人に特定のものだけを公開することができる」のです。これが、リソースインターフェースが強力な理由です。以下を見てみましょう：

```cadence
pub contract Stuff {

    pub resource interface ITest {
      pub let name: String
    }

    pub resource Test: ITest {
      pub let name: String
      pub let number: Int
      init() {
        self.name = "Spongebob"
        self.number = 1
      }
    }

    pub fun noInterface() {
      let test: @Test <- create Test()
      log(test.number) // 1

      destroy test
    }

    pub fun yesInterface() {
      let test: @Test{ITest} <- create Test()
      log(test.number) // ERROR: `member of restricted type is not accessible: number`

      destroy test
    }
}
```

さて、一体何が起こったのでしょうか。いろいろなことが起こっています：

1. `noInterface` という関数を作りました。この関数は新しいリソース（型は `@Test` ）を作成し、その `number` フィールドにログを記録します。これは完璧に動作します。
2. `yesInterface` という関数を作りました。この関数は、**`ITest` インターフェースに制限された**新しいリソース（型は `@Test{ITest}` ）を作成し、`number` フィールドのログを記録しようとしますが、失敗します。

Cadence では、`{RESOURCE_INTERFACE}` 記法を使ってリソースの「タイプを制限」します。`{}` 括弧を使い、真ん中にリソースインターフェースの名前を書きます。
つまり「この型は、**インターフェイスによって公開されたものだけを使用できるリソースである**」ということです。これが理解できれば、リソースインターフェースを本当によく理解していることになります。

では、なぜ `yesInterface` の `log` は失敗するのでしょうか？それは `ITest` が `number` フィールドを公開していないからです！だから `test` 変数を `@Test{ITest}` とタイプしても、それにアクセスすることはできません。

## 複雑な例

以下は、関数も含めたより複雑な例です：

```cadence
pub contract Stuff {

    pub resource interface ITest {
      pub var name: String
    }

    pub resource Test: ITest {
      pub var name: String
      pub var number: Int

      pub fun updateNumber(newNumber: Int): Int {
        self.number = newNumber
        return self.number // returns the new number
      }

      init() {
        self.name = "Spongebob"
        self.number = 1
      }
    }

    pub fun noInterface() {
      let test: @Test <- create Test()
      test.updateNumber(newNumber: 5)
      log(test.number) // 5

      destroy test
    }

    pub fun yesInterface() {
      let test: @Test{ITest} <- create Test()
      let newNumber = test.updateNumber(newNumber: 5) // ERROR: `member of restricted type is not accessible: updateNumber`
      log(newNumber)

      destroy test
    }
}
```

別の例をお見せして、関数を公開しないという選択肢もあることをお見せしたかったのです。できることはたくさんあります！:D もしこのコードを修正したいのであれば、こうするでしょう：

```cadence
pub contract Stuff {

    pub resource interface ITest {
      pub var name: String
      pub var number: Int
      pub fun updateNumber(newNumber: Int): Int
    }

    pub resource Test: ITest {
      pub var name: String
      pub var number: Int

      pub fun updateNumber(newNumber: Int): Int {
        self.number = newNumber
        return self.number // returns the new number
      }

      init() {
        self.name = "Spongebob"
        self.number = 1
      }
    }

    pub fun noInterface() {
      let test: @Test <- create Test()
      test.updateNumber(newNumber: 5)
      log(test.number) // 5

      destroy test
    }

    // Works totally fine now! :D
    pub fun yesInterface() {
      let test: @Test{ITest} <- create Test()
      let newNumber = test.updateNumber(newNumber: 5)
      log(newNumber) // 5

      destroy test
    }
}
```

関数を `ITest` に追加したとき、関数の定義だけを置いたことに注意してください： `pub fun updateNumber(newNumber: Int): Int`。インターフェイスで関数を実装することはできません。

## まとめ

今日の内容をよく乗り切りました。リソースインターフェースは、第4章でアカウントストレージの話を始めるときに非常に重要になります。

## クエスト

1. リソース・インターフェイスが使える2つのことを、あなた自身の言葉で説明してください。(今日のコンテンツではこの2つについて説明しました)

2. 独自のコントラクトを定義する。独自のリソースインターフェースと、そのインターフェースを実装したリソースを作成する。2つの関数を作りなさい。1つ目の関数では、リソースのタイプを制限せず、そのコンテンツにアクセスする例を示しなさい。2番目の関数では、リソースのタイプを制限し、そのコンテンツにアクセスできない例を示しなさい。

3. このコードをどのように修正すればいいのか？

```cadence
pub contract Stuff {

    pub struct interface ITest {
      pub var greeting: String
      pub var favouriteFruit: String
    }

    // ERROR:
    // `structure Stuff.Test does not conform
    // to structure interface Stuff.ITest`
    pub struct Test: ITest {
      pub var greeting: String

      pub fun changeGreeting(newGreeting: String): String {
        self.greeting = newGreeting
        return self.greeting // returns the new greeting
      }

      init() {
        self.greeting = "Hello!"
      }
    }

    pub fun fixThis() {
      let test: Test{ITest} = Test()
      let newGreeting = test.changeGreeting(newGreeting: "Bonjour!") // ERROR HERE: `member of restricted type is not accessible: changeGreeting`
      log(newGreeting)
    }
}
```
