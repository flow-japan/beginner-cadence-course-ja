# 第5章1日目 - 事前条件／事後条件 ＆ イベント

今日は、かなり簡単ではありますが、Cadence で非常によく使われる2つのコンセプトを学びます。

## ビデオ

事前条件／事後条件： https://www.youtube.com/watch?v=WFqoCZY36b0

イベント： https://www.youtube.com/watch?v=xRHG6Kgkxpg

## 事前条件／事後条件

今までのところ、何かが正しくない場合にプログラムを中断させる方法は1つしか学んでいません：`panic` キーワードです。`panic` は、それが呼ばれた場合、コードで起こったことを完全に元に戻すキーワードであり、それと一緒にメッセージを送信します。以下はその例です：

```cadence
pub fun main(): String {
  let a: Int = 3

  if a == 3 {
    panic("This script will never work because it will always panic on this line.")
  }

  return "Will never get to this line."
}
```

くだらない例ですが、言いたいことはわかるでしょう。常にパニックを起こすので、決して戻ることはありません。

多くの場合、エラーをより明確な方法で処理し、「フェイルファスト」と呼ばれるコンセプトも実装したいです。ブロックチェーンでは、オペレーションに非常にコストがかかります。トランザクションに高額な手数料がかかるのはそのためです。「フェイルファスト」とは、何か問題があればできるだけ早くコードが失敗するようにプログラミングすることで、無意味に実行時間を無駄にしないようにする方法です。

事前条件／事後条件はこれに最適です。関数が呼び出される前（事前）または後（事後）に何か問題があった場合、失敗する明確な方法を指定することができます。例を見てみましょう：

```cadence
pub contract Test {

  pub fun logName(name: String) {
    pre {
      name.length > 0: "This name is too short."
    }
    log(name)
  }

}
```

上の例では、関数 `logName` に「事前条件」を定義しています。名前の長さが 0 より大きくない場合、`panic` にこのようなメッセージを表示します：この名前は短すぎます。

事前条件と事後条件は関数の最初に定義**しなければ**なりません。事前条件／事後条件がパスするためには、その条件が `true` でなければなりません。そうでなければ、後の文字列で `panic` することになります。

事後条件は、関数の最後でチェックされることを除けば同じことです（それでも開始時に定義されていなければなりません。分かりにくいかもしれませんが、すぐに慣れるでしょう）：


```cadence
pub contract Test {

  pub fun makePositiveResult(x: Int, y: Int): Int {
    post {
      result > 0: "The result is not positive."
    }
    return x + y
  }

}
```

あなたは不思議に思うかもしれません：
「あの `result` 変数はいったい何なんだ？定義していないぞ」その通りです！事後条件は超クールで、返される値と等しい `result` 変数がすでに用意されています。つまり、もし `x + y` を返したら、`result` はその値を表します。もし戻り値がなければ、`result` は存在しません。

さらに、事後条件の中で `before()` 関数を使えば、関数が実行された後でも、その関数が何かを変更する前に何かの値にアクセスすることができます。

```cadence
pub contract Test {

  pub resource TestResource {
    pub var number: Int

    pub fun updateNumber() {
      post {
        before(self.number) == self.number - 1
      }
      self.number = self.number + 1
    }

    init() {
      self.number = 0
    }

  }

}
```

事後条件が満たされているので、上記のコードは常に動作します。これは「 `updateNumber` 関数が実行された後、更新された数値がこの関数が実行される前の値より 1 大きいことを確認します。」この場合は常に真です。

### 重要な注意事項

`panic` や事前条件／事後条件が実際に何をするのかを理解することは重要です。これらはトランザクションを「中止」します、つまり、実際には状態は何も変更されていません。

例：
```cadence
pub contract Test {

  pub resource TestResource {
    pub var number: Int

    pub fun updateNumber() {
      post {
        self.number == 1000: "Will always panic!" // この関数が実行された後にパニックになると、`self.number` は 0 にリセットさます。
      }
      self.number = self.number + 1
    }

    init() {
      self.number = 0
    }

  }

}
```

## イベント

イベントは、スマートコントラクトが外部に何かが起こったことを伝えるための手段です。

例えば、NFT をミントする場合、NFT がミントされたことを外部に知らせたいとします。もちろん、常にコントラクトをチェックして、`totalSupply` が更新されたかどうかを確認することもできますが、それは非効率的です。なぜ、コントラクトから*私たち*に教えてくれないのですか？

Cadenceでイベントを定義する方法を説明します：

```cadence
pub contract Test {

  // ここでイベントを定義します
  pub event NFTMinted(id: UInt64)

  pub resource NFT {
    pub let id: UInt64

    init() {
      self.id = self.uuid

      // イベントを外部にブロードキャストします
      emit NFTMinted(id: self.id)
    }

  }

}
```

`NFTMinted` というイベントを次のように定義したのがわかるでしょう：`pub event NFTMinted(id: UInt64)` 。イベントは常に `pub`/`access(all)` であることに注意してください。次に `emit` キーワードを使ってイベントをブロードキャストし、ブロックチェーンに送信します。

イベントにパラメータを渡すことで、外部にデータを送信することもできます。この場合、特定の ID を持つ NFT がミントされたことを世界に伝えたいので、イベントを読むクライアントは特定の NFT がミントされたことを知ることになります。

その目的は、クライアント（コントラクトを読んでいる人たち）が何かが起こったときにそれを知ることができ、それに応じてコードを更新できるようにするためです。NFT がミントされるたびに、私の顔が描かれた花火が打ち上がるようなクールなウェブサイトを作ることもできるかもしれません！:D

## まとめ

今日はここまでです！短いレッスンを楽しんでいただけたなら幸いです。

## クエスト

1. イベントとは何か、なぜそれがクライアントに役立つのかを説明します。

2. イベントを含むコントラクトをデプロイし、そのイベントが起こったことを示すために、コントラクトのどこか他の場所でイベントを発生させます。

3. ステップ 2 ）のコントラクトを使って、コントラクトに事前条件と事後条件を追加し、書き出すことに慣れます。

4. 以下の各関数（numberOne、numberTwo、numberThree）について、指示に従ってください。

```cadence
pub contract Test {

  // やること
  // この関数が名前を記録するかどうかを教えてください。
  // name: 'Jacob'
  pub fun numberOne(name: String) {
    pre {
      name.length == 5: "This name is not cool enough."
    }
    log(name)
  }

  // やること
  // この関数が値を返すかどうかを教えてください。
  // name: 'Jacob'
  pub fun numberTwo(name: String): String {
    pre {
      name.length >= 0: "You must input a valid name."
    }
    post {
      result == "Jacob Tucker"
    }
    return name.concat(" Tucker")
  }

  pub resource TestResource {
    pub var number: Int

    // やること
    // この機能が更新された番号を記録するかどうかを教えてください。
    // また、実行後の `self.number` の値も教えてください。
    pub fun numberThree(): Int {
      post {
        before(self.number) == result + 1
      }
      self.number = self.number + 1
      return self.number
    }

    init() {
      self.number = 0
    }

  }

}
```
