# Apache Groovyとは

[Apache Groovy](http://www.groovy-lang.org/)（以下Groovy）とは、現在 Apacheソフトウェアファウンデーション配下で開発が進められている **動的型付け言語** です。  
厳密な定義すると色々語弊があるかもしれませんが、 **Groovyはいわゆるスクリプト言語です。**

GroovyはJava Virtual Machine(以下JVM)上で動作します。  
なお、JVM上で動くJava以外のプログラミング言語には他にも色々なものが有ります。  
代表的なものとしては[Scala](http://www.scala-lang.org/)や[Kotlin](https://kotlinlang.org/)があります。  
この2つのプログラミング言語は、いわゆる **静的型付け言語** に属し、Groovyとは少し毛色が違います。  
そういう意味では、Groovyは同じJVM上で動くLISP方言の[Clojure](https://clojure.org/)の方に近いかもしれません。  

なお、Groovyの立ち位置としては、 **Javaを置き換えるものではなく、Javaに寄り添うもの** とよく表現されます。  
基本的にGroovyはJavaのコードをコピペすればそのままGroovyとして動作します。  
`HelloWorld.java`をの拡張子を.groovyに変えて`HelloWorld.groovy`とするだけでOKなのです。  
このことから以下にGroovyがJavaとの親和性を大切にしているかが伺えます。  

## シンプルなシンタックス
GroovyはJavaの良い点を踏襲しつつ、スクリプト言語らしく非常にシンプルにコーディングできるようになっています。  
さらに、関数型プログラミングのスパイスも取り入れてコードをより簡潔に表現することも出来ます。  

例えば、1から10の整数の入ったリストから偶数のみ抜き出して、それぞれを2倍して、全ての数をかけあわせる、という処理を実装してみます。

```
(1..10).findAll {
    it % 2 == 0
}.collect {
    it * 2
}.inject {l, r ->
    l * r
}
```

非常に簡潔ですね！  
forの様なループやifによる条件分岐、さらに途中の計算結果を代入する一時変数すら登場しません。  
そしてGroovyはセミコロンと、明示的なreturnは不要です。  
また、型の指定も省略することが可能です。  

Groovyではクロージャー（Closure）が簡単に利用できます。  
クロージャーを利用することで、Java8から導入されたStreamやLambdaと同じような事を、より簡潔に表現することが出来ます。  
例えば、クロージャーを利用して数学のシグマ（Σ）を実装してみます。

```
def sigma = {Integer k, Integer to, Closure exp ->
    (k..to).collect {
        exp(it)
    }.sum()
}
assert 25 == sigma(1, 5) {it + 2}
```

これまた簡潔ですね。  
すでにGroovyでは型を省略できると書きましたが、逆に型を明示することも当然可能です。

## 実験コードのお供に
Javaで開発をしていて、ある機能が必要になり、あるアルゴリズム/ロジックをコードを書きながら試行錯誤したい、ということはよく有りますよね？   
Groovyは標準で様々な便利機能が用意されています。  
例えば、言語機能として`assert`（パワーアサート）が有ります。  
コレは、そのような実験コード（例えばどんなアルゴリズムにしようかを決める際に色々書くコード）に非常に便利です。  

実際にサンプルを見てみましょう。

```
class SomeClass {
    def someMethod() {
        "Groovy"
    }
}

def someClass = new SomeClass()
assert "groovy" == someClass.someMethod()
```

最後の行がパワーアサートの部分です。  
この例だとsomeMethodの戻り値は先頭が大文字なのに、意図した結果は先頭小文字なので、当然エラーがになります。  
エラーなお内容は非常に分かりやすく表示されます。  

```
Assertion failed: 

assert "groovy" == someClass.someMethod()
                |  |         |
                |  |         Groovy
                |  SomeClass@3e633f10
                false

	at ConsoleScript6.run(ConsoleScript6:8)
```

こうすることで、メソッドの中身の実装を変えた際に、毎回目で実行結果に誤りが無いか、挙動が変わっていないかの確認が必要が無くなります。  
さらに、GroovyConsoleやGroovyshと言ったGroovyに備わっているツールを使えば、態々IDEやテキストエディタを開く事無くコードを試すことが出来ます。  
（GroovyConsoleはGUIツール、groovyshはCUIツール。いわゆるREPL）

## ビルドツール要らず
さて、では少し「実験コード」の範囲を広げましょう。  
例えばあなたが参加しているJavaプロジェクトであるJavaライブラリを導入することになったとします。  
そのライブラリの使い方をチェックするために態々山のようなクラスファイルを作る必要も、その外部ライブラリをクラスパスにと通す必要も、既存のプロジェクトコードを汚す必要もありません。  
さらにGradleやMavenといったビルドツールを導入したり、そもそもその対象ライブラリをダウンロードしてくる必要さえ無いのです。  
Groovyは言語機能として、 **Grab** という機能が有ります。このGrabに必要なライブラリの情報（Mavenに記述する様な情報）を渡してあげるだけで、ダウンロードからクラスパスの設定まで全てGroovyが自動で行ってくれます。  
例えば、ココではJsoupというJavaのWEBスクレイピングによく利用されるライブラリを導入してみましょう。

```
@Grab(group='org.jsoup', module='jsoup', version='1.8.3')

import org.jsoup.Jsoup
import org.jsoup.nodes.Document
import org.jsoup.nodes.Element
import org.jsoup.select.Elements

Document document = Jsoup.connect("http://koji-k.github.io/groovy-tutorial/index.html").get()
document.select("a").collect {Element element ->
    "[${element.text()}](${element.attr("href")})"
}.each {
    println it
}
```

これだけです。必要なのは上記のGroovyコードだけです。後は全てGroovyが面倒を見てくれます。  
当然ある程度規模の大きなアプリケーションになってくるとGradleやMavenを利用したほうが良いですが、Groovyではこのように日々のちょっとした業務をサポートしてくれる強力な機能が非常に多く備わっています。  

（Grabの詳細は本チュートリアルの **Groovyで外部ライブラリを利用する** を参照してください。）

## Javaの学習に
すでに述べたとおり、Javaコードは基本的にそのままGroovyとして動作します。  
さらにGroovyの場合は1ファイルに1クラス、というルールがありません。つまり1ファイルにいくつもクラスを宣言することが出来ます。  
そしてクラスの宣言自体必要ありません。

```
class A {
    def hoge() {
        println "A"
    }
}

class B {
    def hoge() {
        println "B"
    }
}

def a = new A()
def b = new B()
a.hoge()
b.hoge()
```

このコードは1ファイルに詰め込んでもちゃんと動作します。  
このように、余計な事を考えずに必要なことに集中することができるのでこれからJavaを学習したい、というひともGroovyからJavaを効率よく勉強することが出来ます。  


## もちろんプロダクトとしても
Groovyは、2016年の時点で登場からすでに13年が経っています。  
さらにJava用の膨大なライブラリもGroovyからそのまま利用できます。  
さらにJavaのコードがそのままGroovyとして動作するので、まずはJavaシンタックスで書いておいてあとからGroovyのシンタックスに少しずつ変えていくということも可能です。  
また、GroovyにはフルスタックなフWEBレームワーク[Grails](https://grails.org/)があります。  
Groovyはすでに実戦で利用できる十分な実績、ライブラリが備わっています！

## どこからでもOK
JVMという実績のある環境の上で動作し、膨大なJavaのエコシステムを利用でき、シンプルで実用的な機能を抱負に揃えたGroovy。  
さらにコレクションを変換して、集約していき、可能な限り副作用を減らすという関数型プログラミングのエッセンスも含まれています。  
また、直ぐにプロダクトとして導入しなくても、ちょっとしたデータの作成用や、簡単なバッチファイルとしての利用などにも利用できます。  
Groovyは常に現実的な業務に寄り添ってくれます。あなたの必要だと思える場所、タイミングからいつだって使いはじめることが出来る相棒です。  

さぁ、Groovyを初めましょう！