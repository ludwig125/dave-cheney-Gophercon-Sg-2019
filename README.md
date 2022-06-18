# Go の開発者の一人 Dave cheney のシンガポールでのプレゼンの意訳

https://dave.cheney.net/practical-go/presentations/gophercon-singapore-2019.html

これは、作者に許可をとったものではなく自分用に訳したメモです。

読んでいて、自分なりに気になった点は（訳注）で分けていますが、そもそも翻訳した対象は私が意図的に選択したものですし、
翻訳内容も意訳、超訳なので間違いの箇所があればすべて私のミスです。

もとのプレゼン資料が 2019 年のものですし、訳したり注釈をつけたのも 2019 年から 2020 年にかけてなので今では古い書き方がある可能性があります。

## 1. Guiding principles

https://dave.cheney.net/practical-go/presentations/gophercon-singapore-2019.html#_guiding_principles

- Go 言語の長所である Clarity、Simplicity、Productivity について熱意をもって説明しています
- シンプルでわかりやすいことは、コードの信頼性を高める
- 簡潔なコードよりもわかりやすいコードに重きを置きましょう
  など

## 2. Identifiers

主に変数の長さやコードの書き方について説明

#### 2.2. Identifier length

変数の長さ

_コード中で、変数が定義されてから最後に使われるまでの距離が長いほど、変数の名前も長くする_

`for _, p := range people { ` の時の p はこのループの中でしか使わないので、一文字で十分

- index とかの名前は読む人を逆に混乱させる
  `people`は関数の中の数行で使われるので、省略せず people とする

#### 2.3. A variable’s name should describe its contents

型を変数名に含めない

ペットに dog とか cat とかつけないのと同じように、変数名に usersMap のように `Map` などの型名をつけるのはやめましょう。
Go は型が厳格なので、Map を Slice として使ってしまうようなことはありません。
自明なものはつけないようにしましょう。
usersMap は users で十分です

#### 2.5. Use a consistent declaration style

Go の変数宣言の方法は色々あります。
（宣言の仕方が多すぎるのは失敗でしたが、気づいたときには手遅れでした）

基本はこの２つで十分です

１．変数を後で使う目的で宣言し初期化しないときは `var things []thing` の形

- var things = nil はだめ
- var thing = new(Thing) もだめ
- var things = make([]Thing, 0)も別の意味なのでだめ
  ２．変数を初期化するときは `:=` の形にする

例外

```
var min int
max := 1000
```

みたいに不自然になってしまう場合は以下の方法でいいです
`min, max := 0, 1000`

## 3. Commentary

コメントは以下の３つのどれかであるべきです

1. それが何をするものなのか what

- 以下のようなコメント。公開メソッドなどの上に書いてあるやつ
  `// Open opens the named file for reading.`

2. あることをするのにどうやって実現しているか how

- メソッドの中にあるといいコメント
  `// queue all dependant actions`
  （訳注：ここ文字変なので https://dave.cheney.net/practical-go/presentations/qcon-china.html 見たほうがいいです）

3. なぜそれがあるのか？

- 外部要因だったり、ぱっと見では理由がわからないものついて書くと良いコメント

例
`const randomNumber = 6 // determined from an unbiased die`

上の例では、なぜランダムな数字として６が使われているのか、その由来はなにかが書いてあります

#### 3.2.2. Rather than commenting a block of code, refactor it

```
優れたコードはそれ自体が最高の資料です。コメントを追加しようとする前に、
そのコードをもっと良くできないか自問しましょう。
コードを良くすればコメントもより明確になります
```

（訳注：心がいたい）

## 4. Package Design

https://dave.cheney.net/practical-go/presentations/gophercon-singapore-2019.html#_package_design

#### 4.3. Avoid package names like base, common, or util

`utils` `helpers` `common` `base` などは目的を反映していないことがしばしばです。
目的にあわせて適切なパッケージに移しましょう。

io/ioutil と net/http/httputil パッケージはだめな例です（意訳）

**間違った抽象化をするくらいなら、コードが重複する方がよっぽど安上がりです。**

たとえば、net/http パッケージでは、`client` や `server` などのサブパッケージを持ちません。
client.go, server.go というファイルが存在します。これらはそれぞれ必要な type を持っています

#### 4.4. Return early rather than nesting deeply

ガード構文を使って深いループを避けましょう

Go 言語では例外処理がありません。
関数の中では異常を見つけ次第そのエラーを返す、ガード構文を採用するといいです。

異常が発生しない限りは抜けない作りになっていれば、
関数を上から順に見ていくとそれがそのまま処理が成功しているルートになります。
Mat Ryer はこのスタイルを `line of sight` 照準線と呼んでいます。

#### 4.5. Make the zero value useful

https://dave.cheney.net/practical-go/presentations/gophercon-singapore-2019.html#_package_design

未初期化の変数を効果的に使いましょう

slice には未初期化の値が nil であるという特性があります。
len, cap も 0。したがって、未初期化の slice の内容はメモリ上でも nil です。
したがって、slice を定義してそれに値を追加するのは単純にこう書けるのです

```
var s []string
s = append(s, "Hello")
s = append(s, "world")
```

var s1 = []string{} と var s2 []string は違うので注意しましょう
（訳注：前者はゼロの長さのスライス、後者は nil です。[]string{}と []string(nil)の違い）

#### 4.6. Avoid package level state

パッケージレベルの変数を避けましょう。

グローバル変数はあなたのソースコードの全関数の目に見えないパラメータとして存在します！
グローバル変数をちょっと変えたらこれらは破綻します

コード間の相互依存を減らすために、以下の２つがおすすめです

１．グローバル変数は避けましょう。変数はそれを必要とするコード内の struct の field に入れましょう
２．interface を使って振る舞いと実装を分けましょう

## 5. Project Structure

https://dave.cheney.net/practical-go/presentations/gophercon-singapore-2019.html#_project_structure

あなたの project（git repository の単位と同義）がライブラリなら、それは１つのことだけを担うべきです。
（XML をパースするとか、ロギングするとか）
（訳注：これはマキルロイの UNIX 哲学：「一つのことを行い、またそれをうまくやるプログラムを書け」と同じだと思いました）

複数の目的を担当するのはやめましょう。恐るべき `common` ライブラリ誕生を防止しましょう！

TIPS：

```
私の経験では、 `common` リポジトリを作ると、それを使った依存部分がとても大きくなります。
それによりbackport 修正が困難になります。
もし修正するとなったら、`common` に加えて利用者側にも更新の必要が生じます。
更新の際には、利用者には関係のない変更やAPIのIFのずれの問題なども発生します。

```

#### 5.1. Consider fewer, larger packages

package の不適切な使い方は Go において多く見られる失敗の一つです。（これはプログラマーが
何かをカテゴライズしたいと思ってしまう気質が影響しているのだろうと私は考えています）

Go では、他の言語と異なり、ある識別子（変数、関数、メソッド）の参照方法として、
public か private の２種類しかありません。（protected とか default とかありません）
識別子の最初の文字が大文字ならパッケージ外に公開、小文字なら非公開、という違いだけです。

ではこの２種類しか修飾子がないという前提で、
package の依存関係を複雑にしないようにするためには何を心がけるべきでしょうか？

私のアドバイスは以下の通りです、
_package はなるべく数を少なくし、一つの中身は大きくする_
（訳注： "prefer fewer, larger packages" の訳）

基本的には、新しいパッケージは作らないスタンスで行きましょう。
これにより、package 内の識別子へのアクセスは広く、浅く、になります。
詳細はこのあとのセクションで細かく説明します。

#### 5.1.1. Arrange code into files by import statements

https://dave.cheney.net/practical-go/presentations/gophercon-singapore-2019.html#_project_structure

`.go` ファイルをどのタイミングで分割すべきでしょうか？
また、どんなときに１つにまとめたほうがいいでしょうか？

私のガイドラインは以下のとおりです

１．それぞれの package につき、一つの　`.go` ファイルから始めましょう。
それぞれのファイルは package 名と同じにしましょう。たとえば `http` package には `http.go`を置きましょう

２．package が大きくなって分割したくなったら、「責任範囲」ごとにファイルを分けましょう
例えば、 `message.go`には `Request` と `Response` type が含まれるべきですし、
`client.go`には `Client`が、 `server.go` には `Server` type が含まれるべきです

３．複数のファイルで同じような import をしている場合は、１つにまとめるか、
共通部分の type/ function/ method をくくりだして別のファイルに置きましょう

４．別々のファイルは、それぞれ package 内で別々の役割を担うべきです。
`messages.go`は HTTP リクエストやレスポンスの marshal をするかも知れません。（以下略）

TIP：ファイル名はソースコードの入れ物（containers）なので、名詞がふさわしいです

NOTE: go のコンパイラはそれぞれのパッケージを並列でコンパイルします。それぞれのパッケージではそれぞれの関数も並列でコンパイルされます。
したがって、パッケージのレイアウト変更はコンパイル速度に影響しません

#### 5.1.2. Use internal packages to reduce your public API surface

https://dave.cheney.net/practical-go/presentations/gophercon-singapore-2019.html#_project_structure

自分のプロジェクト内では public にしたい、でもプロジェクト外からは private にしたい、
というときは internal ディレクトリを作りましょう

（訳注：internal についての説明は以下が詳しいです）
公式の説明
https://github.com/golang-standards/project-layout#internal
日本語の紹介記事
https://text.baldanders.info/golang/internal-packages/

#### 5.2. Keep package main as small as possible

_main 関数と main package はできる限り小さくしましょう_

main.main はシングルトンです。
main.main が呼び出されるには多くの前提となる条件が必要とされます。そして一度きりしか実行されません。
このため、main.main はユニットテストをしづらくします。

main パッケージは、しばしばシングルトンを作成します。また、コマンドラインフラグをパースしたりもします。
ディスクのある場所にファイルがないと実行できないこともあります。
さらに、同時並行で実行されることも想定されていません。
main.main は単体テストから呼び出すこともできません。

それ故に、ビジネスロジックはなるべく main 関数から別のところに移したほうがいいです。
可能であれば main パッケージからも他へ移すべきです。

`func main()` はフラグをパースしたり、database や logger とのコネクションを開いたりそういったことだけをすべきです。
そして実際に起動するのは 別の高レベルの object に任せましょう。

（訳注：シングルトンの弊害は以下がわかりやすかったです）
https://xn--97-273ae6a4irb6e2hsoiozc2g4b8082p.com/%E3%82%A8%E3%83%83%E3%82%BB%E3%82%A4/%E3%82%B7%E3%83%B3%E3%82%B0%E3%83%AB%E3%83%88%E3%83%B3%E3%83%91%E3%82%BF%E3%83%BC%E3%83%B3%E3%81%AE%E8%AA%98%E6%83%91%E3%81%AB%E8%B2%A0%E3%81%91%E3%81%AA%E3%81%84/

（訳注：シングルトンの弊害を簡潔にまとめたページも見つけました）
https://tech.a-listers.jp/2011/05/06/i-am-adam/

```
依存関係を見えにくくし、コードが読みづらくなる。
ユニットテストを難しくする。外部から渡せないオブジェクトはモックにする事が難しい。
プログラムの再利用性が低下する。一度しか使わないからとシングルトンで作ってしまうと複数のユーザーから利用されるような場合に対応できなくなる。
```

## 6. API Design

https://dave.cheney.net/practical-go/presentations/gophercon-singapore-2019.html#_api_design

デザインに関する最後のアドバイスは、私が最も重要だと思うものです。
これまでの私のデザインに関するアドバイスはただの「提案」ですので、人に強要するものではありません。
が、後方互換性に関しては話が別です。

パッケージの公開 API については、最初のデザインについては熟考するべきです。
あとから変えるのは非常に多くの人に迷惑を与えます。（意訳）

（訳注：ここでいう API とは、関数やメソッドも含んでいるようです）

#### 6.1. Design APIs that are hard to misuse.

`APIは簡単に使えて、誤用しにくいものにすること` Josh Bloch

##### 6.1.1. Be wary of functions which take several parameters of the same type

`func Max(a, b int) int` この関数は使うのは簡単。
引数 a と b の順番が入れ替わっても結果は変わらない

```
Max(8, 10) // 10
Max(10, 8) // 10
```

一方 `func CopyFile(to, from string) error` この関数はどうでしょうか？
もし引数の順序を間違えると、成果物が先週に戻ってしまうかも！

```
CopyFile("/tmp/backup", "presentation.md")
CopyFile("presentation.md", "/tmp/backup")
```

いい解決策は以下のような方法です

```
type Source string

func (src Source) CopyTo(dest string) error {
	return CopyFile(dest, string(src))
}

func main() {
	var from Source = "presentation.md"
	from.CopyTo("/tmp/backup")
}
```

こうすれば誤用が防げます。CopyTo を小文字にして private にすればさらにリスクを減らせるでしょう。

TIP 同じ型の複数のパラメータを受け取る API は、正しく使うことが難しいことを覚えておきましょう

#### 6.2.1. Discourage the use of nil as a parameter

https://dave.cheney.net/practical-go/presentations/gophercon-singapore-2019.html#_api_design

API（関数やメソッドなど）の引数として nil を渡す仕様は推奨しません。

ここでは「デフォルトユースケース」について話します。
API を使う人にとって意味のない引数を必須にするのはやめましょう。
例として、net/http を見てみましょう

https://github.com/golang/go/blob/bbbc6589dfbc05be2bfa59f51c20f9eaa8d0c531/src/net/http/server.go#L3083-L3087

```
// ListenAndServe always returns a non-nil error.
func ListenAndServe(addr string, handler Handler) error {
```

この関数は２つの引数を取ります。
listen 用の TCP アドレスと、HTTP リクエストのハンドラ http.Handler です。

Handler が nil のときは、http.DefaultServeMux をが使われます。
（訳注： https://github.com/golang/go/blob/daaab44f3124aff61937fa7e118f02d4ff82166c/src/net/http/server.go#L2801-L2803 ）

つまり以下の２つは同じ意味です。

```
http.ListenAndServe("0.0.0.0:8080", nil)
http.ListenAndServe("0.0.0.0:8080", http.DefaultServeMux)
```

ただし、２つめの引数は nil にできますが、１つ目の引数については nil にはできません。

TIP： nil にできる引数とできない引数を混ぜるのはやめましょう

http.ListenAndServe の作成者は、API を簡単に使えるようにしたかったのでしょう。
しかしそのことで、むしろ安全に使いにくくなってしまいました。

```
const root = http.Dir("/htdocs")
mux := http.NewServeMux()  // ここ
mux.Handle("/", http.FileServer(root))
http.ListenAndServe("0.0.0.0:8080", mux)
```

この２行目を節約する価値は本当にあったのでしょうか？

TIP: helper 関数があれば使う人にとってとても役に立ちます。
（訳注：helper 関数というのは、上の例では NewServeMux を指している？）
簡潔さよりも明瞭さの方が重要です

TIP: 公開 API にはテストでしか使わない引数を渡すのはやめましょう
ラッパーを作ってそれらの引数を隠しましょう

#### 6.2.2. Prefer var args to []T parameters

https://dave.cheney.net/practical-go/presentations/gophercon-singapore-2019.html#_api_design

```
func ShutdownVMs(ids []string) error
```

こんな関数を考えてみましょう。
この引数は slice なので、空の slice や nil も入る可能性があります。（それでもコンパイラは異常を検知しません）
そのため、そういった場合を考えてテストケースを用意する必要があります。

引き数として与えられた slice の中身が全部正であることを確認するために、以下のようなチェックを入れるのは長くなって嫌です。

```
if svc.MaxConnections > 0 || svc.MaxPendingRequests > 0 || svc.MaxRequests > 0 || svc.MaxRetries > 0 {
	// apply the non zero parameters
}
```

そのため、以下のような関数を事前に呼ぶと良さそうです。

```
// anyPostive indicates if any value is greater than zero.
func anyPositive(values ...int) bool {
	for _, v := range values {
		if v > 0 {
			return true
		}
	}
	return false
}
```

ただこれでも足りない点があって、一つも引き数がないときに問題になります。
そのため、私が考えた方法は以下の解決策です。これなら、引数を最低１つは指定するように強制できます。

```
// anyPostive indicates if any value is greater than zero.
func anyPositive(first int, rest ...int) bool {
	if first > 0 {
		return true
	}
	for _, v := range rest {
		if v > 0 {
			return true
		}
	}
	return false
}
```

#### 6.3. Let functions define the behaviour they require（関数は必要な振る舞いを考えて定義しましょう）

https://dave.cheney.net/practical-go/presentations/gophercon-singapore-2019.html#_let_functions_define_the_behaviour_they_require

こんな感じのドキュメントをディスクに書き込むメソッドを考えてみましょう

```
type Document struct {
        // mo' state
}
// Save writes the contents of the Document to the file f.
func (d *Document) Save(f *os.File) error
```

このメソッドの引数は `*os.File` ですが、これにはいくつか問題があります。

データをネットワーク上のどこかに書き込むことには対応していません。（lambda とか）

- そのため修正が必要になったらこの関数を呼ぶすべてのところに影響します

test もしづらいです。ディスクに書き込まなければいけないので、然るべきファイルを test 前に用意して、
test 後にそれを消さなければいけません

さらに、 `*os.File` は Save メソッドには無関係の多くのメソッドも含んでいます。
（例えばディレクトリを読んだり、パスが symlink かどうかチェックするなどです）

そのため、`Save` メソッドが、 `*os.File` のうち必要な部分だけ渡す仕組みになっていれば都合が良さそうです。

```
// Save writes the contents of d to the supplied ReadWriterCloser.
func (d *Document) Save(rwc io.ReadWriteCloser) error
```

`*os.File` の代わりに、 `io.ReadWriteCloser` を渡すことにすれば、interface 分離の原則を守れます。
これにより、File 以外の多くに対応できるようになります。
ただし、interface 分離の原則をもっと推し進めることもできます。

（訳注：ちょっと長いので意訳したのが以下）

以下のように引数の型を制限しました。

```
// Save writes the contents of d to the supplied Writer.
func (d *Document) Save(w io.Writer) error
```

１．単一責任の原則を考えて read を削る

- ファイルの read（中身を確認するとかの作業）は別のコードに任せます
- このため io.ReadWriteCloser の Read は削りました

２．Close も別処理にする

- `wc io.WriteCloser` を与えると、Save が成功したら Close するのか、成功しなくても Close するのか良くわかりません
- 関数を使う人が混乱しないように、Save は Write のみを担当するようにさせました

interface 分離の原則を適用したことで、Save には io.Writer だけを渡せば済むようになりました。
こうなるともはや `Save` よりも `WriteTo` の方がメソッドの名前として正確だと言えます

最終的にはこうなりました。
`func (d *Document) WriteTo(w io.Writer) error`

（訳注：interface 分離の原則などの日本語記事として以下見つけたものを貼っておきます）
https://note.com/erukiti/n/n913e571e8207
https://qiita.com/shunp/items/646c86bb3cc149f7cff9

#### 6.4. Understanding nil

https://dave.cheney.net/practical-go/presentations/gophercon-singapore-2019.html#_understanding_nil

nil は不思議な特性を持っています
後述の通り、nil は定数です。しかし nil で定数を定義することはできません。

nil で変数を定義することはできます。しかし、nil 同士を比較することはできません

```
package main

import "fmt"

const T = nil != nil // (1)

func main() {
	fmt.Println(nil == nil) // (2)

	if nil == new(int) {
		fmt.Println("hmm") // (3)
	}
}
```

(1) nil は nil と !=で比較できない（訳注： `!= nil` を消しても const T を初期化することはできません）
(2) nil は nil と ==で比較できない
(3) nil は比較演算子のどちら側にもかける
https://play.golang.org/p/Knh1br_HZ9m

これらを見ると、nil は通常の Go とは別の世界の存在のようですが、この特性のおかげで Go をシンプルなものにしています。

- pointer を nil にすると、pointer は何者も参照しない
- interface を nil にすると、interface は何も含まない
- slice を nil にすると、slice は長さゼロ、容量ゼロ、何の配列も参照しない

nil の型は割り当てられた変数の型によって決まります。

```
var s []string
if s == nil {
  // ...
}
```

このように書いたとき、nil の型は（s が[]string だと自明なので） []string として比較評価されます。

NOTE
以下のような場合は注意が必要です

```
func Open(path string) io.Writer {
	var f *os.File
	f, _ = os.Open(path)
	return f
}

func main() {
	f := Open("/missing")
	fmt.Println(f == nil) // false
}
```

上で、f == nil は false になります。
なぜなら、 `return f` で返される f は `*os.File` 型で定義されているからです。

関数（メソッド）で nil を返したい場合は明示的に `return nil` としましょう

（訳注：go の nil の特性を調べるといくつか出てきますので興味深いリンクを載せます）
https://www.calhoun.io/when-nil-isnt-equal-to-nil/
https://golang.org/doc/faq#nil_error

（訳注：個人的には interface 型に別の型付きで nil となった変数を代入すると、型アサーションが必要だというのがハマりそうだと思いました）
https://play.golang.org/p/qZLojLsmsgB

```
var a *int = nil
var b interface{} = a

fmt.Println("a == nil:", a == nil) // true
fmt.Println("b == nil:", b == nil) // false
fmt.Println("b.(*int) == nil:", b.(*int) == nil) // true
```

６章まで個人的に初めて見る話が多かったのでやや逐語訳気味にしていましたが、
７章は知っていることが多いので、意訳の度合いを少し強めます。

## 7. Error handling

https://dave.cheney.net/practical-go/presentations/gophercon-singapore-2019.html#_error_handling

エラーハンドリングは超重要です。
あなたのコードの他の部分と同様に重要です。境界値のテストなどと同じくらい重要です。

#### 7.1. Errors are just values

`Errors are just values.` エラーは値

この言葉はとっても有名ですが、正しく理解できているでしょうか？

一旦 panic と recover を見てみましょう。この２つの keyword の目的は _たった１つです_
defer の中で使う。これだけ。他の目的では使ってはいけません

（訳注：以下のようなケースだと思います）
https://blog.golang.org/defer-panic-and-recover

```
defer func() {
    if r := recover(); r != nil {
        fmt.Println("Recovered in f", r)
    }
}()
```

#### 7.2. Errors should be opaque

エラーは関数の呼び出し側にとって「不明瞭（opaque）」なものであるべきです。
API の呼び出し元は、（nil かどうかという判定は除きますが、）
そのエラーの内容を見て処理を変えるようなことをしてはいけません。
関数と呼び出し元の間に依存関係が生まれてしまうからです。

例外は io.EOF です。これは上のルールが適用されません。

（訳注: dave cheney は io.EOF の判定を好ましいものと分類していないようです）

#### 7.3. Assert errors for behaviour, not type

https://dave.cheney.net/practical-go/presentations/gophercon-singapore-2019.html#_error_handling

詳細は以下がわかりやすいです。
Dave Cheney 本人のブログ

- https://dave.cheney.net/2016/04/27/dont-just-check-errors-handle-them-gracefully
  日本語の紹介
- https://deeeet.com/writing/2016/04/25/go-pkg-errors/

要点は、 `エラーを受け取った時の分岐は、そのエラーが実装しているinterfaceで判断しましょう` ということです。

（訳注：Dave Cheney の記事を参考に作ってみたサンプル ↓）
https://play.golang.org/p/U_pIN9-RSBF

（訳注：前回の補足）
「7.3. Assert errors for behaviour, not type」
でエラーの型ではなくて実装している interface で判断するとした件ですが、
最新の Go 1.13 には errors に As が追加されているのでこれが使えます。
紹介記事
https://blog.golang.org/go1.13-errors
https://text.baldanders.info/golang/error-handling-in-go-1_3/
https://qiita.com/shibukawa/items/e633e426a6e67ea2e830

でも Go の中でも、Unwrap より Dave Cheney 作の Cause 使ったほうがいいんじゃないみたいな議論も見つけました。
https://github.com/golang/go/issues/31778
意見が完全に固まってくれるのでしょうか

### 7.4. Never use nil to indicate failure

https://dave.cheney.net/practical-go/presentations/gophercon-singapore-2019.html#_error_handling

```
type Bar struct{}

func (b *Bar) Whoa() {
	fmt.Println("whaaaaaat?")
}

func main() {
	var b *Bar // (1)
	b.Whoa()   // (2)
}
```

この、何の値も初期化していない b の method Whoa は異常なく動作します。
その理由は、Go における method というのは、関数の第一引数をレシーバとした、
シンタックスシュガーだからです（！）
つまり上の method は下の関数と同じ

```
func Whoa(b *Bar) {
	fmt.Println("whaaaaaat?")
}
```

以下のように、b または b のフィールドにアクセスしようとすると panic を起こします

```
func (b *Bar) Whoops() {
	fmt.Println(b.message) // (1)
}
```

これを防ぐためには様々な方法が取りえます。
メソッドの作成者としてできることはこれです

```
func (b *Bar) Whoops() {
	if b != nil { // (1)
		fmt.Println(b.message)
	}
}
```

でもこれだとメソッドの中でしか判定できません。
（メソッドを呼んだときには手遅れかも）

メソッドを呼ぶ前に判断するとしたらこんな方法が考えられます。

```
func main() {
	var b *Bar
	if b != nil {
		b.Whoops() // (1)
	}
}
```

ただこの方法だと、メソッドを呼ぶすべての部分にこの判定を入れなければいけません。

（訳注：ということで、このあといくつか考えられる方法を書いていますが、長いので以下の結論だけ書きます）

nil レシーバの存在を完全に防ぐことはできません。あなたのコードのどこにでも存在しえます。
最も発生しやすいのは、メソッドを呼び出す前に別の関数を呼び出したときに error チェックを忘れて結果の nil を渡してしまう場合です。
あなたがコードを書くときに注意する部分はこのエラーチェックの部分です。

これに絡んで気をつけなければいけないこととしては、
API（関数）を作るときは、（error を除いて)nil を返す関数を作ってはいけません！
Go 以外で、複数値を返せない言語では、失敗を表す場合に nil（または null）を返すことがありますが、
そういったことはしないようにしましょう。

TIP: nil レシーバのチェックはやめましょう。
test カバレッジを上げましょう。vet/lint ツールを使って nil レシーバになるような条件を見つけましょう。

go vet の説明
http://tsujitaku50.hatenablog.com/entry/2017/01/07/121825
https://qiita.com/marnie_ms4/items/b343165efb4235906db7

以下端折ったところ

以下のように、nil の場合は error を返す方法が良いです。

```
func (b *Bar) GetMessage() (string, error) {
	if b == nil {
		return "", errors.New("b is nil") // (1)
	}
	return message, nil
}
```

関数の呼び出し元にとってわかりやすい内容のエラーを返さなければいけません

１．すべてのメソッドはエラーを返すべきです 「すべてのメソッドです」
２．関数呼び出し元はすべて必ずエラーチェックをするべきです
３．interface を定義するときは、nil レシーバを受け取ったときのエラーを返せるようにしましょう
４．実装するすべての inteface はエラー戻り値を持たなければいけません
（訳注：3,4 が良くわかりませんでしたが、私が先日投稿したような例かなと思いました）
https://play.golang.org/p/U_pIN9-RSBF

#### 7.5. Don’t panic

https://dave.cheney.net/practical-go/presentations/gophercon-singapore-2019.html#_dont_panic

panic をしないようにしましょう。go には確かに panic 関数が存在します。
しかしこれは runtime の `throw` 関数の副産物です
（訳注： thow 関数ってこれですかね）
https://github.com/golang/go/blob/ba66797392da2b6538ce014a4f7e13c490e74d59/src/runtime/panic.go#L1102

recover を使った例は確かにありますが、用法は限られています。例外キャッチのためのものではありません。
（ローカルで行った変更を戻すためなど）

プログラムはいろいろな場面で panic を吐きます。
(メモリ不足だったり、プロセスマネージャーに kill されたり)
プログラムを書くときは失敗を予期したものにしましょう。
panic は避けましょう。recover も避けましょう。
これらはあなたが求めているツールではありません！

#### 7.5.1. Avoid selfish panics

panic はプログラムを継続できない時の最後の手段です。
ライブラリの中で panic するのは最後の最後の手段です。
使う人にとっては、危険この上ないものになります。

panic が呼ばれると defer が実行されますが、それはその goroutine の中だけです。
他の goroutine の defer 文は実行されません。
ライブラリ中で panic を起こすと、全プログラムを停止させます。絶対にやめましょう。

TIP: ライブラリ中で panic させるのは絶対にやめましょう！

#### 7.5.2. Avoid log.Fatal

log.Fatal と log.Panic も panic と同様の影響を与えます。
使うのを避けましょう。
log.Fatal の一行で Log を出せて crash もできるという利便性が、誤用を招くもとになってしまいました。
（訳注：log.Fatal は os.Exit(1) で強制終了されて
defer も効かないので panic より扱いに注意が必要だと私は思っています）

### 7.6. Eliminate error handling by eliminating errors

https://dave.cheney.net/practical-go/presentations/gophercon-singapore-2019.html#_eliminate_error_handling_by_eliminating_errors

エラー自体をなくし、エラーハンドリングの数を減らしましょう
（エラーを無視するという意味ではありません）

#### 7.6.1. Counting lines

ファイルの行数をカウントする例を見てみます。

```
func CountLines(r io.Reader) (int, error) {
	var (
		br    = bufio.NewReader(r)
		lines int
		err   error
	)

	for {
		_, err = br.ReadString('\n')
		lines++
		if err != nil {
			break
		}
	}

	if err != io.EOF {
		return 0, err
	}
	return lines, nil
}
```

以前の話の通り、r には `*os.File` を渡すのではなく `io.Reader` のみを絞って渡していることに注意です。

（訳注：NewReader）
https://golang.org/pkg/bufio/#NewReader

（訳注：この章長いので以下意訳します）

NewReader を使う方法には問題があります。
ファイルの最後に達すると `io.EOF` をエラーとして返すので、それを判定する必要があるのです。

これは、以下のように `NewReader` の代わりに、 `NewScanner` を使うことによって避けることができます。

```
func CountLines(r io.Reader) (int, error) {
	sc := bufio.NewScanner(r)
	lines := 0

	for sc.Scan() {
		lines++
	}
	return lines, sc.Err()
}
```

（訳注：NewScanner）
https://golang.org/pkg/bufio/#NewScanner

実は bufio.Scanner は中で bufio.Reader を使っているのですが、
`sc.Err()` はエラーが無く読み込みが終わったら、 `io.EOF` を nil に変換するという働きをしてくれます。

TIP: もしエラーハンドリングだらけになってしまったら、一部の処理を抜き出して helper type にすることを考えてみましょう
（訳注：ここで言う helper type とは、上の例のように余計なエラーを変換するとかそういう意味ですね）

#### 7.6.2. WriteResponse

https://dave.cheney.net/practical-go/presentations/gophercon-singapore-2019.html#_eliminate_error_handling_by_eliminating_errors

２つ目の例はエラーは値、という以下のブログにインスパイアされたものです。
https://blog.golang.org/errors-are-values

```
type Header struct {
	Key, Value string
}

type Status struct {
	Code   int
	Reason string
}

func WriteResponse(w io.Writer, st Status, headers []Header, body io.Reader) error {
	_, err := fmt.Fprintf(w, "HTTP/1.1 %d %s\r\n", st.Code, st.Reason)
	if err != nil {
		return err
	}

	for _, h := range headers {
		_, err := fmt.Fprintf(w, "%s: %s\r\n", h.Key, h.Value)
		if err != nil {
			return err
		}
	}

	if _, err := fmt.Fprint(w, "\r\n"); err != nil {
		return err
	}

	_, err = io.Copy(w, body)
	return err
}
```

このような、HTTP レスポンスを返すサーバを考えてみましょう。

エラーハンドリングがいっぱいあって読みにくいものになっています。
ここで、io.Writer interface を満たすメソッドを持つ、 `errWriter` という構造体を定義してみましょう

（訳注：Write メソッドさえあれば interface を満たします）
https://golang.org/pkg/io/#Writer

```
type errWriter struct {
	io.Writer
	err error
}

func (e *errWriter) Write(buf []byte) (int, error) {
	if e.err != nil {
		return 0, e.err
	}
	var n int
	n, e.err = e.Writer.Write(buf)
	return n, nil
}
```

この構造体を使うことで、最初のエラーハンドリングは以下のように書き直すことができます。

```
func WriteResponse(w io.Writer, st Status, headers []Header, body io.Reader) error {
	ew := &errWriter{Writer: w}
	fmt.Fprintf(ew, "HTTP/1.1 %d %s\r\n", st.Code, st.Reason)

	for _, h := range headers {
		fmt.Fprintf(ew, "%s: %s\r\n", h.Key, h.Value)
	}

	fmt.Fprint(ew, "\r\n")
	io.Copy(ew, body)
	return ew.err
}
```

（訳注：私が色々書くよりも、以下の資料を見たほうがわかりやすいです）
http://jxck.hatenablog.com/category/go
https://blog.golang.org/errors-are-values

（訳注：Write メソッドはエラーがすでに有る場合はそれをそのまま返し、ない場合は実行して e.err にエラーを保存する関数となっています）
（訳注：でも個人的にはこの手法はパッと見ではわかりにくい気がしましたが好みの問題もありそうです）

#### 7.7. Only handle an error once

https://dave.cheney.net/practical-go/presentations/gophercon-singapore-2019.html#_only_handle_an_error_once

最後に言及したいこととしてはこれです。
エラーハンドリングは１度だけにしましょう。

エラーハンドリングというのは、エラーの中身を評価することを指します。
そしてエラーを見てすることは 「たった一つ」にしましょう

「一つ未満のエラーハンドリング」、というのはつまりエラーを無視することです
以下のような場合は Write の結果のエラーは捨てられます

```
// WriteAll writes the contents of buf to the supplied writer.
func WriteAll(w io.Writer, buf []byte) {
        w.Write(buf)
}
```

一方で、「一つより多く」エラーハンドリングするというのも、それはそれで問題です。
以下のような書き方をよく見ます。

```
func WriteAll(w io.Writer, buf []byte) error {
	_, err := w.Write(buf)
	if err != nil {
		log.Println("unable to write:", err) // annotated error goes to log file
		return err                           // unannotated error returned to caller
	}
	return nil
}
```

この書き方をすると、log にエラーが出力されます。
また、関数の呼び出し元にもエラーが返されることになります。

もし関数の呼び出し元で以下のようなエラーハンドリングをすると、

```
func WriteConfig(w io.Writer, conf *Config) error {
	buf, err := json.Marshal(conf)
	if err != nil {
		log.Printf("could not marshal config: %v", err)
		return err
	}
	if err := WriteAll(w, buf); err != nil {
		log.Println("could not write config: %v", err)
		return err
	}
	return nil
}
```

Log には以下のように出力されます。冗長ですね。

```
unable to write: io.EOF
could not write config: io.EOF
```

一方で、WriteConfig 関数の呼び出し元はどうでしょうか？
以下のようにエラーを見ても `io.EOF` が出力されるだけです。
（どの関数で失敗したか、という情報はここにはありません）

```
err := WriteConfig(f, &conf)
fmt.Println(err) // io.EOF
```

さらなる問題は、エラーを log に出すと、以下のように err を return するのを忘れてしまうことがあり得ることです。

```
if err != nil {
		log.Printf("could not marshal config: %v", err)
		// oops, forgot to return
	}
```

この場合、エラーを評価し、Log にも出したのに、エラーは返していないので、深刻なバグに繋がります。

（訳注：ちょっと長いので、以下まとめます）

- ガード構文にしたがって、エラーがあったらその場でエラーを返す
- エラーにはちゃんと `could not marshal config: ` のような説明を付与する
- エラーハンドリングは１度だけにする
- 関数の途中で Log に書くのは、エラーを戻すのを忘れたり、エラーの説明を忘れたり、Log が冗長になる原因になる

### 8. Testing

https://dave.cheney.net/practical-go/presentations/gophercon-singapore-2019.html#_testing
（訳注：この章は知っている内容が多いので飛ばし飛ばしで行きます）

よく test と coding にはそれぞれどのくらい時間配分をしていますかと質問されますが、
私はこれを２つの理由で興味深い質問だと感じます。

１つ目は、質問者が test と coding を別々に考えている点です。
質問されている方は、ひょっとしたら手順書に従ったテストをしているのかもしれません。
私はテストを自動化することに取り組んでいるので、私にとっての test は coding の延長です。

２つ目は、私が普段どれだけテストについて考えているかを気づかせてくれる点です。
私は普段、どうやったらコードがテストしやすくなるか考えています。
自動でテストできないようなデザインになっていないかも考えています。

ということで、最初の質問の答えとしては、「私はテストのためのデザインについて多大な時間を費やしています」です

テストを QA チームに任せる時代は 90 年代で終わりました。
開発チームがプロダクトの品質を担保すべき時代になったのです。

そして、テストは必ず自動化するべきです。
なぜなら、手順書に従ったテストは実行時間がテストの量にそって線形（O(n)）に増えていくからです。
そして、99.9%のテストは以前と同じように動作します。
となるとどうなるか、人間はテストを省略するようになります。
マニュアルテストは時間がかかるし退屈だからです。そうやって、重要だと思われるテストだけするようになって ���� まいます。

まとめ、
あなたがバグ fix や改善に取り組むとき、最初にすべきことは「失敗する場合のテスト(failing test)」を最初に作ることです。
（訳注：バグ fix や改善が機能していないと失敗してしまうテスト、という意味だと思われます）
これは単体テストである必要はありませんが、自動テストにするべきです。
最初に failing test を作ってしまって、あとは実際のバグ fix や機能改善をしてうまくいくか確認しましょう

### 8.1. Tests lock in behaviour

https://dave.cheney.net/practical-go/presentations/gophercon-singapore-2019.html#_testing

あるパッケージに関する unit test は、package 内に閉じた範囲を扱うようにしましょう。
コードの挙動はドキュメントよりもコードに書くべき（訳注：test 含む）です。
`go test` だけで動作が保証できるからです。
これにより、コードの変更後にもちゃんと動くという信頼性が高まります。

#### 8.2. Table driven testing

#### 8.2.1. Test scope

（訳注：この辺は皆さんご存知だと思いますのでさらっと）
Table driven testing はこういうものです。

```
func TestSplit(t *testing.T) {
	type test struct {
		input string
		sep   string
		want  []string
	}

	tests := []test{
		{input: "a/b/c", sep: "/", want: []string{"a", "b", "c"}},
		{input: "a/b/c", sep: ",", want: []string{"a/b/c"}},
		{input: "abc", sep: "/", want: []string{"abc"}},
	}

	for _, tc := range tests {
		got := Split(tc.input, tc.sep)
		if !reflect.DeepEqual(tc.want, got) {
			t.Fatalf("expected: %v, got: %v", tc.want, got)
		}
	}
}
```

上では test という struct を用意していますが以下のように構造体を最初に宣言しないようにも書けます

```
tests := []struct {
  input string
  sep   string
  want  []string
}{
```

#### 8.2.2. Code coverage

https://dave.cheney.net/practical-go/presentations/gophercon-singapore-2019.html#_name_your_tests

（訳注：上で触れ忘れたので coverage について記載します）

package の branch coverage）は以下で確認できます

```
% go test -coverprofile=c.out
PASS
coverage: 100.0% of statements
ok      split   0.010s
```

この場合は、coverage が 100%であることがわかります。

以下のように-func=c.out で、関数ごとの coverage をわかりやすく見ることもできます

```
% go tool cover -func=c.out
split/split.go:8:       Split          100.0%
total:                  (statements)   100.0%
```

（訳注：go test の coverage などについては以下のリンクがものすごくわかりやすいです。
test 以外の様々な tool の使い方と結果が解説されています。おすすめです）

- An Overview of Go's Tooling
  - https://www.alexedwards.net/blog/an-overview-of-go-tooling

#### 8.2.3. Enumerating test cases

https://dave.cheney.net/practical-go/presentations/gophercon-singapore-2019.html#_name_your_tests

（訳注：知っていることが多いのでさらっと書き出します）

テストに番号をつける
https://play.golang.org/p/6_UltZnA1sm

実行結果

```
=== RUN   TestSplit
--- FAIL: TestSplit (0.00s)
    prog.go:37: test 4: expected: [a b c], got: [a b c ]
FAIL
```

#### 8.2.4. Name your tests

テストに名前をつける
https://play.golang.org/p/P3KPXFYx8RM

実行結果

```
=== RUN   TestSplit
--- FAIL: TestSplit (0.00s)
    prog.go:38: trailing sep: expected: [a b c], got: [a b c ]
FAIL
```

上の改良。test ケースを map にすることで実行順序をランダムにする

- ランダムにすることで、test 間の依存によるバグを発見しやすい
  - 前のテストで全体に関わる状態の変化を行っていたとしても、それに気づかせてくれる、など
    https://play.golang.org/p/0kUaYDWYgPI

#### 8.2.5. Sub tests

サブテスト
https://play.golang.org/p/i3OHgI4ho2J

- サブテストには名前がついているので、以下のように個別にテストを実行することが可能です

```
% go test -run=.*/trailing -v
=== RUN   TestSplit
=== RUN   TestSplit/trailing_sep
--- FAIL: TestSplit (0.00s)
    --- FAIL: TestSplit/trailing_sep (0.00s)
        split_test.go:25: expected: [a b c], got: [a b c ]
FAIL
exit status 1
```

#### 8.2.6. Comparing expected an actual

https://dave.cheney.net/practical-go/presentations/gophercon-singapore-2019.html#_comparing_expected_an_actual

前述までのテストコードでは、エラー時のメッセージは以下のようなものでした

```
--- FAIL: TestSplit (0.00s)
    --- FAIL: TestSplit/trailing_sep (0.00s)
        split_test.go:25: expected: [a b c], got: [a b c ]
```

この問題箇所は `%#v` を使うとわかりやすくなります。

```
t.Fatalf("expected: %#v, got: %#v", tc.want, got)
```

結果はこのように、空があることがわかります。

```
% go test
--- FAIL: TestSplit (0.00s)
    --- FAIL: TestSplit/trailing_sep (0.00s)
        split_test.go:25: expected: []string{"a", "b", "c"}, got: []string{"a", "b", "c", ""}
FAIL
exit status 1
FAIL    split   0.005s
```

ただ、 `%#v` でもわかりにくい時もあります。

```
func main() {
	type T struct {
		I int
	}
	x := []*T{{1}, {2}, {3}}
	y := []*T{{1}, {2}, {4}}

	fmt.Printf("%v %v\n", x, y) // (1)
	fmt.Printf("%#v %#v\n", x, y) // (2)
}
```

結果

1. [0xc000096000 0xc000096008 0xc000096010] [0xc000096018 0xc000096020 0xc000096028]
2. []*main.T{(*main.T)(0xc000096000), (*main.T)(0xc000096008), (*main.T)(0xc000096010)} []*main.T{(*main.T)(0xc000096018), (*main.T)(0xc000096020), (*main.T)(0xc000096028)}

こういう場合は、https://github.com/google/go-cmp というパッケージの利用を提案します。
例えば以下のようにすると、cmp.Equal は false を返します。

```
x := []*T{{1}, {2}, {3}}
y := []*T{{1}, {2}, {4}}
fmt.Println(cmp.Equal(x, y)) // false
```

cmp.Diff は以下のように見やすくなります。

```
diff := cmp.Diff(x, y)
fmt.Printf(diff) // (1)
```

```
% go run .
{[]*main.T}[2].I:
        -: 3
        +: 4
```

前述の Split 関数のテストを cmp パッケージで一部書き換えると以下のようになります。
https://play.golang.org/p/fMeoZTmFDWv

#### 8.3. Prefer internal tests to external tests

https://dave.cheney.net/practical-go/presentations/gophercon-singapore-2019.html#_prefer_internal_tests_to_external_tests

go で test をする際には二種類の方法があります。
あなたが http2 というパッケージを作成したとして、http2_test.go というテストファイルを作ったとします。
このとき、テストファイルのパッケージ名を `http2` とすることで、
`http2_test.go` は http2 パッケージの一部だとコンパイル時にみなされます。
いわゆる internal test です。
（訳注：これにより http2 内の小文字始まりの非公開の関数のテストができます）

go はテスト用に `test` で終わるパッケージ名を使うこともできます。
例えば、 `package http_test` のような  パッケージ名です。
この名前のパッケージ名のテストファイルを、コードと同一パッケージ内に置くと、
コンパイル時には、コードとテストは別のパッケージだとみなされます。
いわゆる external test です

単体テストを書くときは、internal test として書くことをおすすめします。
external test の bureaucracy （訳注：面倒な事前準備を意味していると解釈しました）をすることなく、
コード内の各関数、メソッドを直接テストすることができるからです。

ただし、 `Example` test 関数は external test ファイルに書きましょう。
こうすることで、godoc で見たときに、examples が適切なパッケージの prefix を持ち、コピペも容易になります。
（訳注：以下が詳しいです）
https://blog.golang.org/examples

TIP: ドット import を避けて、internal test を使いましょう

（訳注：ドット import や test のパッケージ名については以下が参考になります）
https://stackoverflow.com/questions/19998250/proper-package-naming-for-testing-with-the-go-language

#### 8.4. Tests give you the power to release any day of the week

https://dave.cheney.net/practical-go/presentations/gophercon-singapore-2019.html#_tests_give_you_the_power_to_release_any_day_of_the_week

テストをすることはソフトウェア開発中に役立つだけでなく、いつでも master ブランチにマージできるという保証を与えてくれます。
これはつまり、いつでもリリースできるということです。
「我々は重要なリファクタリングの最中だからいまリリースはよそう、数週間後ならできるさきっと」
などと言うような事態は避けるべきです。
テストが通れば、あなたの開発物がどんなときでも期待通りに動くという保証が与えられるのです！

#### 8.5. Tests give you confidence to change someone else’s code

テストのカバレッジを十分にすることで、おそらく最も良い点は、
自分が書いたものではないコードの修正を自信をもって修正できることです。

これはチーム開発において重要なことです。

## 9. Concurrency

https://dave.cheney.net/practical-go/presentations/gophercon-singapore-2019.html#_concurrency

多くの本で concurrency 並行処理が最後の章で取り上げられているように、このプレゼンでも concurrency は最後に扱います。
Go はその並行処理の容易さゆえに多くのプロジェクトで採用されていることが多いのですが、
並行処理を扱うのは簡単ではありません。そのため最後に取り上げています。

#### 9.1. Channel Axioms

#### 9.1.1. A send to a nil channel blocks forever

この章では、channel のどちらかといえばあまり知られていない特徴について取り上げます。

`nil` channel へ送信しようとすると永久にブロックされます。

```
func main() {
	var c chan string
	c <- "let's get started" // (1)
}
```

上のコードは(1)の部分で deadlock します。
未初期化の channel は `nil` なので、送信できません。

#### 9.1.2. A receive from a nil channel blocks forever

同様に、nil channel を受信しようとすると deadlock します。

```
func main() {
	var c chan string
	fmt.Println(<-c) // (1)
}
```

この説明は以下の通りです。

- チャネルの型宣言をしただけでは、バッファーのサイズは設定されません。
  チャネルのサイズは、チャネルの値の一部だからです
- チャネルが初期化されていない場合、そのバッファサイズはゼロになります
- チャネルのバッファのサイズがゼロの場合、バッファなしチャネルとなります
- バッファなしチャネルの場合、別のゴルーチンが受信可能になるまで送信はブロックされます
- チャネルが nil の場合、送信側と受信側は互いに関与し合いません。
  そのため、互いに独立したチャネルを待ってブロックし合い、そのブロックが解除されることはありません

#### 9.1.3. A send to a closed channel panics

https://dave.cheney.net/practical-go/presentations/gophercon-singapore-2019.html#_a_send_to_a_closed_channel_panics

（訳注：この章の説明どう考えてもおかしい気がするので、真剣に見ないほうが良いかも知れません）

以下のプログラムは最初の goroutine が close した時点で、他の goroutine が close 済みの channel に
値を送れなくて panic になります。

```
func main() {
        var c = make(chan int, 100)
        for i := 0; i < 10; i++ {
                go func() {
                        for j := 0; j < 10; j++ {
                                c <- j
                        }
                        close(c)
                }()
        }
        for i := range c {
                fmt.Println(i)
        }
}
```

（訳注：このプログラムは playground で実行すると panic を起こさず終了します
goroutine 内に i が渡っていないので、実際 goroutine として処理されるのは最後の i だけというバグもあります。
これは dave cheney が説明のために書いた例と考えるべきです。
ちなみにこのバグは go vet で確認できます）

```
$go vet sample.go
# command-line-arguments
./sample.go:13:32: loop variable i captured by func literal
```

では、close 済みかどうかを確認するための関数を用意するという案はどうでしょうか？
例えばこんな感じのものです。

```
if !isClosed(c) {
        // c isn't closed, send the value
        c <- v
}
```

この関数でチェックして OK だったとしても、c<-v を処理する前に他の goroutine が c を close してしまう可能性があります。
それぞれの goroutine は全く同じタイミングで動いているかも知れませんし、バラバラかも知れないのです。
goroutine 同士は、（チャネルなどで）コミュニケーションしないといけません。

（訳注：ということで、この後解決策として以下のコードが出てくるのですが、実はこれでもだめです。
結局 closed の判定がスレッドセーフではないように見えます。
ちょっと変えるだけでうまくいくかなと思ったのですが、だめでした。
waitGroup を使うのがいいと思います）
https://play.golang.org/p/TfakbrJSMnk

#### 9.1.4. A receive from a closed channel returns the zero value immediately

https://dave.cheney.net/practical-go/presentations/gophercon-singapore-2019.html#_a_receive_from_a_closed_channel_returns_the_zero_value_immediately

close された channel は zero を返します。

```
func main() {
	c := make(chan int, 3)
	c <- 1
	c <- 2
	c <- 3
	close(c)
	for i := 0; i < 4; i++ {
		fmt.Printf("%d ", <-c) // prints 1 2 3 0
	}
}
```

このプログラムは c に 3 を送ったあとに close されているので、print した結果は 3 以降は 0 となります。
https://play.golang.org/p/aJb4DV8ZMCb

NOTE:
channel が close されるまでその値を取り続けたい（consume したい）ときは `for range` 文を使うのが良い方法です。

```
for v := range c {
	// do something with v
}
```

これは下のプログラムのシンタックスシュガーとなっており動作は同じです。
for range 文は以下のプログラムを簡略化したものなのです。

```
for v, ok := <- c; ok ; v, ok = <- c {
        // do something with v
}
```

#### 9.2. Prefer channels with a size of zero or one

https://dave.cheney.net/practical-go/presentations/gophercon-singapore-2019.html#_prefer_channels_with_a_size_of_zero_or_one

未知の producer や consumer を扱うとき、バッファサイズとしてはゼロまたは 1 が選ばれます。
バッファサイズ 0 は producer と consumer が同時に動く場合、
バッファサイズ 1 は送信側がブロックされること無く１つは値として channel にためておける状態です。

channel の値が受信されるまでにどれだけ溜めておきたいかの正確な数を知っているときは、
バッファサイズを１より上げることが有効です。
最も好ましい channel サイズは大抵 0 か 1 です。それ以外は見積り次第です。
見積もりが間違っていた場合、プログラムは正しく動かない可能性があります。

#### 9.3. Keep yourself busy or do the work yourself

https://dave.cheney.net/practical-go/presentations/gophercon-singapore-2019.html#_keep_yourself_busy_or_do_the_work_yourself

このプログラムの問題点はなんでしょうか？

```
func main() {
	http.HandleFunc("/", func(w http.ResponseWriter, r *http.Request) {
		fmt.Fprintln(w, "Hello, GopherCon SG")
	})
	go func() {
		if err := http.ListenAndServe(":8080", nil); err != nil {
			log.Fatal(err)
		}
	}()

	for {
	}
}
```

（訳注：皆さんおわかりだと思いますが、goroutine 内で ListenAndServe を使って web サーバを起動させ、
無限 for ループによって main の goroutine を終了させないようにすることでサーバを止めないようにしています）

この方法では、無限ループによって CPU を無駄にします。
最後の`for{}`　が main goroutine をブロックすることで、何の IO も生まれません。
シングル CPU を使った状態でライブロックされています。

最後の `for{}` を以下のようにしてみるとどうでしょうか？

```
for {
  runtime.Gosched()
}
```

これも良くない方法に見えますが、私はよく目にします。
プログラムの裏側についての理解が足りていません。

少し go の経験がある人はこんな書き方をするかもしれません。
`for{}` を `select {}` にしてみます。
コード全文
https://play.golang.org/p/VBoym70Oerp

空の select 文は永久にブロックします。
これなら CPU をフルに使って `runtime.GoSched()` だけをさせ続ける必要はありません。

しかしもっといい方法があります。
`http.ListenAndServe` を別の goroutine ではなく main goroutine の中で実行すればいいのです。

TIP: main.main 関数は、別の goroutine が起動している最中であっても無条件に終了します。

改修後のコード

```
package main

import (
	"fmt"
	"log"
	"net/http"
)

func main() {
	http.HandleFunc("/", func(w http.ResponseWriter, r *http.Request) {
		fmt.Fprintln(w, "Hello, GopherCon SG")
	})
	if err := http.ListenAndServe(":8080", nil); err != nil {
		log.Fatal(err)
	}
}
```

ということで、私からの助言は以下の通りです。
あなたの goroutine が外部の何らかの処理の結果を受け取らない限り何も進まないのであれば、その「何らかの処理」は
別の goroutine に任せるのではなく自分自身が請け負ったほうがシンプルです。

これによって、（外部の goroutine の）状態を追ったり channel 操作をしたりして、
結果を（goroutine の）呼び出し元に伝えるようなことをしなくてすみます。

TIP: 多くの Go プログラマーは gorouine を使いすぎています。
人生全てに言えることですが、節制、節度は成功への鍵です。

#### 9.4. Leave concurrency to the caller

https://dave.cheney.net/practical-go/presentations/gophercon-singapore-2019.html#_leave_concurrency_to_the_caller

この２つの違いは何でしょうか？

```
// ListDirectory returns the contents of dir.
func ListDirectory(dir string) ([]string, error)

// ListDirectory returns a channel over which
// directory entries will be published. When the list
// of entries is exhausted, the channel will be closed.
func ListDirectory(dir string) chan string
```

１つ目は、得られたディレクトリを slice として返します。何か問題があればエラーを返します。
これは同期処理なので、`ListDirectory` は全ディレクトリを読み込むまで呼び出し元をブロックします。
ディレクトリの大きさに依存して、ブロックされる時間も長くなりますし、メモリを大量に食う可能性もあります。

２つ目を見てみましょう。ちょっと Go っぽいです。
`ListDirectory` は読み込んだディレクトリを channel で返します。
関数の呼び出し元は、channel が close されたらもうディレクトリが無いことがわかります。
channel は `ListDirectory` の「呼び出し後」から中身を持ちます。
`ListDirectory` はおそらく goroutine で channel を扱っているのでしょう。

２つ目の channel を使った `ListDirectory` には２つ課題があります。

課題１．
channel の close という方法だけでは、
ListDirectory 内でエラーが生じて channel の中身が不十分だったときに、それを呼び出し元に伝える手段がありません。
呼び出し元はディレクトリが 空だったのか読み込み時にエラーがあったのか確認する方法がないのです。
どちらの場合であっても、 `ListDirectory` の channel はすぐに close されます。

課題２．
呼び出し元は、channel が close されるまで読み込み続けなければいけません。
なぜなら、close されるまで読み続けることしか、goroutine が終了したことを確認するすべがないからです。
これは `ListDirectory` を使う上で重大な制限です。
呼び出し元は、すでに欲しい結果が得られた後も channel の読み込みに時間を費やす必要があるからです。
大きなディレクトリを扱うときのメモリ使用は効率的になりそうですが、
この方法では１つ目の slice を使った方法に比べて速くなっていません。

この２つの課題の解決法は、コールバック関数を返すことです。

```
func ListDirectory(dir string, fn func(string))
```

驚くことではありませんが、 `filepath.WalkDir` 関数も同様の手法を取っています。
（訳注：この名前の関数は見つからなかったので、おそらく filepath.Walk のことだと思われます）
https://github.com/golang/go/blob/ab7c174183b05e36dabe0aa1943a0a4302b940d0/src/path/filepath/path.go#L395-L412

TIP: goroutine を使う関数は、呼び出し元からそれを止められるような方法を提供することが必要です。
非同期で関数を実行しやすくなります。

（訳注：なんとコールバック関数で具体的にどうやっているのかはここでは説明されていません）

#### 9.5. Never start a goroutine without knowing when it will stop

https://dave.cheney.net/practical-go/presentations/gophercon-singapore-2019.html#_never_start_a_goroutine_without_knowing_when_it_will_stop

（訳注：以下訳しましたが、go の server の停止では gracefull shutdown が最近は使われていますので、
こんな考え方もあるんだなあと、色々工夫を考えることができれば良いのかなと思います。）

（訳注：gracefull shutdown)
https://stackoverflow.com/questions/57166480/golang-http-server-graceful-shutdown-after-a-signal
https://marcofranssen.nl/go-webserver-with-graceful-shutdown/

もしかしたら今回のプレゼンの最後にふさわしい話をします。停止の話です。

前回の例では、本当は必要ではない goroutine を取り上げました。
しかし、Go を使う理由が、基本的な操作（first class）として並行処理を提供しているから、という方もいるでしょう。
ハードウェアを生かして並行処理をしたいときもあるでしょう。
必要なら goroutine を使うべきです。

２つ別々の port を提供するサーバを作るとします。
片方はアプリケーション用の 8080、もう片方は /debug/pprof 用の 8001 です。

```
package main

import (
	"fmt"
	"net/http"
	_ "net/http/pprof"
)

func main() {
	mux := http.NewServeMux()
	mux.HandleFunc("/", func(resp http.ResponseWriter, req *http.Request) {
		fmt.Fprintln(resp, "Hello, QCon!")
	})
	go http.ListenAndServe("127.0.0.1:8001", http.DefaultServeMux) // debug
	http.ListenAndServe("0.0.0.0:8080", mux)                       // app traffic
}
```

上のコードを `serveApp` と `serveDebug` に分けるとこうなります。

```
func serveApp() {
	mux := http.NewServeMux()
	mux.HandleFunc("/", func(resp http.ResponseWriter, req *http.Request) {
		fmt.Fprintln(resp, "Hello, QCon!")
	})
	http.ListenAndServe("0.0.0.0:8080", mux)
}

func serveDebug() {
	http.ListenAndServe("127.0.0.1:8001", http.DefaultServeMux)
}

func main() {
	go serveDebug()
	serveApp()
}
```

前述のアドバイスに従って、`serveApp` と `serveDebug` を並行処理にするかどうかは main（呼び出し元）に任せています。

しかしこれだと、 `serveApp` が終わると `main.main` も終わってしまうという問題があります。
一方、 `serveDebug` は別の goroutine なので、これが終わっても全体のプログラムは動き続けます。
「運用で debug を確認しようとしたけど、 `/debug` はずっと前に終了していました」、というのは避けたいことです。

以下のように改良してみます。

```
func serveApp() {
	mux := http.NewServeMux()
	mux.HandleFunc("/", func(resp http.ResponseWriter, req *http.Request) {
		fmt.Fprintln(resp, "Hello, QCon!")
	})
	if err := http.ListenAndServe("0.0.0.0:8080", mux); err != nil {
		log.Fatal(err)
	}
}

func serveDebug() {
	if err := http.ListenAndServe("127.0.0.1:8001", http.DefaultServeMux); err != nil {
		log.Fatal(err)
	}
}

func main() {
	go serveDebug()
	go serveApp()
	select {}
}
```

こうすれば、 `ListenAndServe` がエラーとなった時点で `log.Fatal` を出して main 関数が終わるので、
察知ができます。

main goroutine 内は `select{}` で永久に待たせます。

このアプローチには多くの問題があります。
１． `ListenAndServer` が `nil` エラーを返したとき、 `log.Fatal` は呼ばれず、サーバだけが落ちます。
２． `log.Fatal` は `os.Exit` を呼ぶので無条件にプログラムを停止させます。defer も呼ばれません。
別の goroutine は shut down に気づかず、プログラムが単に終了します。
これだとテストを書きにくいです。

TIP: `log.Fatal` は `main.main` か `init` 関数内でのみ使いましょう

（訳注：長いのでこれの解決策は明日書きます）

goroutine で発生したエラーを呼び出し元に伝えて、「なぜ」 goroutine が停止したのかを確認し、
プロセスをきれいに停止させるためにはどうすればいいのでしょうか？

```
func serveApp() error {
	mux := http.NewServeMux()
	mux.HandleFunc("/", func(resp http.ResponseWriter, req *http.Request) {
		fmt.Fprintln(resp, "Hello, QCon!")
	})
	return http.ListenAndServe("0.0.0.0:8080", mux)
}

func serveDebug() error {
	return http.ListenAndServe("127.0.0.1:8001", http.DefaultServeMux)
}

func main() {
	done := make(chan error, 2)
	go func() {
		done <- serveDebug()
	}()
	go func() {
		done <- serveApp()
	}()

	for i := 0; i < cap(done); i++ {
		if err := <-done; err != nil {
			fmt.Println("error: %v", err)
		}
	}
}
```

上では、goroutine のステータスを収集することができています。
channel のサイズは `done` channel がブロックされないように、起動する goroutine と同じ数だけ用意しています。
これにより、goroutine の停止と leak を防ぐことができます。

`done` channel を安全に停止する方法が無いので、 `for range` は使えません。
その代わり、 channel の容量の数（= 起動する goroutine の数）だけ for 文を回しています。

これにより、全ての goroutine が停止するまで待って生じたエラーを log に書き出すこともできるようになりました。

このロジックをもう少し拡張して、 呼び出し元から `http.Server` の shut down をできるようにしてみましょう。
以下では、 `serve` 関数を用意して、中で `http.ListenAndServe` と同様に address と `http.Handler` を引数に持たせます。
また、 `stop` channel を受け取ったら `Shutdown` メソッドを呼べるようにします。

```
func serve(addr string, handler http.Handler, stop <-chan struct{}) error {
	s := http.Server{
		Addr:    addr,
		Handler: handler,
	}

	go func() {
		<-stop // wait for stop signal
		s.Shutdown(context.Background())
	}()

	return s.ListenAndServe()
}

func serveApp(stop <-chan struct{}) error {
	mux := http.NewServeMux()
	mux.HandleFunc("/", func(resp http.ResponseWriter, req *http.Request) {
		fmt.Fprintln(resp, "Hello, QCon!")
	})
	return serve("0.0.0.0:8080", mux, stop)
}

func serveDebug(stop <-chan struct{}) error {
	return serve("127.0.0.1:8001", http.DefaultServeMux, stop)
}

func main() {
	done := make(chan error, 2)
	stop := make(chan struct{})
	go func() {
		done <- serveDebug(stop)
	}()
	go func() {
		done <- serveApp(stop)
	}()

	var stopped bool
	for i := 0; i < cap(done); i++ {
		if err := <-done; err != nil {
			fmt.Println("error: %v", err)
		}
		if !stopped {
			stopped = true
			close(stop)
		}
	}
}
```

これにより、 `done` channel を受け取ったら `stop` channel を close して、
すべての goroutine が `http.Server` を shut down するまで待つようになりました。

全ての goroutine が停止したら、 `main.main` が return して、全てのプロセスがきれいに停止されます。

以上、長かった、Dave Cheney のプレゼンの意訳が終わりました。
正直「この書き方おかしいかも」とか「最新のはもっといいやり方が確立されている気が」
と思う部分もあったのですが、学ぶことが非常に多かったです。
原文をなんとなく読んでいたときは気づかなかったことも、
訳そうとして深く考えるとなるほど、と気付かされる点も多々ありました。
業務で Go を初めて使うことになって、なんとなく書いてみて、
指針となる書き方が欲しいなーと思っていたので良い教材でした。
