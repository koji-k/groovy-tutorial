# ユニットテスト

モダンなソフトウェア開発にはユニットテストは書かせません。  
テストの無いコードはレガシコードと呼ばれ、忌諱される存在です。
Groovyでコードが正しいかを確認するお手軽な方法として`assert`をすでに紹介しました。  
しかし、コレだけとさすがに不十分です。  
ユニットテストを素早く楽に開発、実行するために様々なテスティングツールがリリースされています。  
Javaの世界では[JUnit](http://junit.org/)がとても有名で、当然Groovyからも利用できます。  
しかし、Groovyの世界では、[Spock](https://github.com/spockframework/spock)というユニットテスト用ツールがデファクトスタンダートとなっています。  

今回はこのSpockを見て行きましょう。  
なお、Spock自体の詳細な仕様は公式ドキュメントを参照してください。

## 初めてのSpock
Spockはとてもシンプルでわかりやすいテストツールです。また、ライブラリとして配布されています。  
そう、Groovyの`@Grab`を使えば、Spockも簡単に導入することが出来ます。  
と言っても本来は、ビルドツールなどから、全てのテストを一気に実行することが一般的だと言うことを胸のどこからに閉まっておいてください。  
しかし、まずはSpockってどんな感じに使うの？ということを知るのが一番大切です。  
Groovyの素晴らしい力を使って早速サクサクテストしていきましょう。

```groovy
@Grab(group='org.spockframework', module='spock-core', version='1.0-groovy-2.4')
import spock.lang.*

class MyFirstSpock extends Specification {

    // テストメソッド（フィーチャーメソッド）
    def "こんにちわSpock!"() {
        expect:
        1 > 0
    }
}
```
このコードを実行すると、

```
JUnit 4 Runner, Tests: 1, Failures: 0, Time: 13
```
と表示されます。テストが１つ合って、エラーが０件、となっていますね。  
では、`1 > 0`というところを`1 == 0`に修正して再度実行してみましょう。

```
JUnit 4 Runner, Tests: 1, Failures: 1, Time: 25
Test Failure: こんにちわSpock!(MyFirstSpock)
Condition not satisfied:

1 == 0
  |
  false

	at MyFirstSpock.こんにちわSpock!(ConsoleScript10:8)
```
上記のようなメッセージが表示されます。  
テストが１件合って、エラーが１件となっていますね。さらにどんなエラー内容だったかまで綺麗に出力してくれています。  

今回、`"こんにちわSpock!"()`というテストメソッドをコーディングして、そのテストを実行しました。  
なお、このテストメソッドのことを **フィーチャーメソッド** と呼びます。  

では、ついでに **フィーチャーメソッド** をもうひとつ追加して以下のようにしてみます。  

```groovy
@Grab(group='org.spockframework', module='spock-core', version='1.0-groovy-2.4')
import spock.lang.*

class MyFirstSpock extends Specification {

    def "こんにちわSpock!"() {
        expect:
        1 == 1
    }
    
    def "さようならSpock!"() {
        expect:
        1 == 1
    }
}
```
実行結果は、

```
JUnit 4 Runner, Tests: 2, Failures: 0, Time: 25
```
となります。  
テストが2件あり、エラーが0件ですね。
フィーチャーメソッドの個数が、テストの個数になっていることが解ると思います。  

コレがSpockの基本です。  


## フィーチャーメソッドとフィーチャーブロック
さて、フィーチャーメソッドがテストを実行する単位だと述べました。  
では、そのテストの中で使える特種な宣言、 **フィーチャーブロック** を見て行きましょう。

### whenとthen（そしてsetup）
最も基本的なパターンです。  
まずは以下のコードを見てみてください。

```groovy
@Grab(group='org.spockframework', module='spock-core', version='1.0-groovy-2.4')
import spock.lang.*

class MyFirstSpock extends Specification {

    def "whenとthenを使ってテスト"(){
        setup:
        List list = (1..10).toList()
        
        when:
        list.add(100)
        
        then:
        list.size == 11
        list.last() == 100
    }
}
```

`setup:`、`when:`、`then:`という3つのキーワードがありますね。  
この3つが **フィーチャーブロックと呼ばれるものです。**  

|ブロック|内容|
|-----|-----|
|setup|前提条件を記述します。|
|when|テスト対象となる動作を記述します。|
|then|whenの実行結果として、期待される条件を記述します|

ちょっと日本語が難しいですね。。。  
`setup:`でテストの準備をして、`when:`の中の処理が実行された場合、`then:`の中の状態になるよね、ということになります。  
`when:`と`then:`は常に同時に記述されます。  
また、今回の場合は、`setup:`フィーチャーブロック自体省略して`when:`フィーチャーブロックの中に処理を移しても動作します。

```groovy
    def "whenとthenを使ってテスト"(){
        when:
        List list = (1..10).toList()
        list.add(100)
        
        then:
        list.size == 11
        list.last() == 100
    }
```

### expect
さて、続いて`expect`フィーチャーブロックです。一番初めのサンプルコードに出ていましたね。  
大雑把に言ってしまうと、`when`と`then`を合体させたようなものです。  
なので、基本的に実現できることは同じです。  
以下に、1から10の整数の合計を求める場合のテストを記述しました。  

```groovy
@Grab(group='org.spockframework', module='spock-core', version='1.0-groovy-2.4')
import spock.lang.*

class MyFirstSpock extends Specification {

    def "expectを使ってテスト"() {
        expect:
        55 == (1..10).sum()
    }
    
    def "expectをwhenとthenで書いてみる"() {
        when:
        def sum = (1..10).sum()
        
        then:
        55 == sum
    }
}
```

`expect`を使ったほうと、`when`と`then`を使った方とで全く同じ結果になります。（つまりテストはパスしています。）  
使い分けとしては、`expect`の方は副作用のない関数型的なメソッドなどのテストに、`when`と`then`はオブジェクトのを操作してその中身を確認すると言った副作用を伴う動作のテストを行う際に使うと言った感じになります。（恐らく）


### cleanup
フィーチャーメソッドの掃除役です。  
書ける場所はフィーチャーメソッドの一番最後（後述するwhereブロックが有る場合はその直前）です。  
テストの為に`setup`でファイルを作成したりした場合、この`cleanup`ブロックで削除します。  
ただそれだけなのですが、1点重要なポイントが有ります。  
それは、例外が発生しようがどうしようが、必ず実行されるという点です。  
以下のコードを見てみましょう。

```groovy
@Grab(group='org.spockframework', module='spock-core', version='1.0-groovy-2.4')
import spock.lang.*

class MyFirstSpock extends Specification {

    def "cleanupのテスト"() {
        setup:
        String message = "do clean!"
    
        expect:
        hogehoge
        
        cleanup:
        println "${message}"
    }
}
```
実行結果は、

```
do clean!
JUnit 4 Runner, Tests: 1, Failures: 1, Time: 11
Test Failure: cleanupのテスト(MyFirstSpock)
groovy.lang.MissingPropertyException: No such property: hogehoge for class: MyFirstSpock
	at MyFirstSpock.cleanupのテスト(ConsoleScript75:8)
```
hogehogeという記述が有りますが、そんな変数もGroovyのキーワードもないので、当然例外になります。  
しかし、`cleanup`フィーチャーブロックで記述している`println "do clean!"`は実行されているのが解ると思います。  
もし`setup`の中でnullが格納された変数が用意され、それがこの`cleanup`ブロックで使われたら。。。？  
この点を留意する必要が有ります。

###where
さて、Spockの素敵な機能である`where`フィーチャーブロックです。  
記述できる場所は、フィーチャーメソッドの最後です。

```
@Grab(group='org.spockframework', module='spock-core', version='1.0-groovy-2.4')
import spock.lang.*

class MyFirstSpock extends Specification {

    def "whereのテスト"() {  
        expect:
        left + right == result
        
        where:
        left|right||result
        1|2||3
        4|5||9
        9|10||19
    }
}
```

実行結果は、

```
JUnit 4 Runner, Tests: 1, Failures: 0, Time: 9
```

です。





`@Unroll`アノテーションを付与してみます。


```groovy
@Grab(group='org.spockframework', module='spock-core', version='1.0-groovy-2.4')
import spock.lang.*

class MyFirstSpock extends Specification {

    @Unroll
    def "whereのテスト"() {  
        expect:
        left + right == result
        
        where:
        left|right||result
        1|2||3
        4|5||9
        9|10||19
    }
}
```

実行結果は、

```
JUnit 4 Runner, Tests: 3, Failures: 0, Time: 11
```

テストの数が3つに増えていますね！








メモ：  テストクラスにプロパティを書いていると、テストの実行ごとに初期化される（つまり、テストごとに毎回テストクラスがnewされているっぽい）  
テスト用メソッドはフィーチャーメソッドと呼ぶ  
フィクスチャメソッドとは、setup、cleanup、setupSpeck、cleanupSpeckの4つの特種なメソッドのこと。




```groovy
@Grab(group='org.spockframework', module='spock-core', version='1.0-groovy-2.4')
import spock.lang.*

class MyFirstSpock extends Specification {

    @Unroll
    def "PersonクラスのisAdultメソッドをテストするよ"() {
    
        setup:
        Person p = new Person(age:age)
        
        expect:
        p.isAdult() == result
        
        where:
        age || result
        19 || false
        20 || true
        21 || true
    }
}

class Person {
    Integer age = 0
    
    Boolean isAdult() {
        age >= 20
    }
}
```

実行結果：  

```
JUnit 4 Runner, Tests: 3, Failures: 0, Time: 29
Result: org.junit.runner.Result@6c0eb436
```

1点注意。GroovyConsoleでSpockを色々試して見る際に、上記の`Personクラス`のように同時にテスト用クラスも宣言できますが、テスト用クラスをテストの前に宣言すると、GroovyConsoleが何を実行していいのかわからなくなって、エラーになってしまいます。  

```
# GroovyConsoleにこんなエラーが出ます
groovy.lang.GroovyRuntimeException: This script or class could not be run.
It should either:
- have a main method,
- be a JUnit test or extend GroovyTestCase,
- implement the Runnable interface,
- or be compatible with a registered script runner. Known runners:
  * groovy-all-2.4.1.jar
```

なので、テスト用クラスは、テストクラスのあとに宣言するようにしてください。（あくまでSpockの動作を勉強する際の話です。）



##まとめ
いかがでしたでしょうか。Groovy自体が備える素晴らしい機能の一端を垣間見ることが出来たのではないでしょうか。  
