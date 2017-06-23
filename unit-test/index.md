# ユニットテスト

モダンなソフトウェア開発にはユニットテストは欠かせません。  
テストの無いコードはレガシコードと呼ばれ、忌諱される存在です。  
Groovyでコードが正しいかどうかを確認するお手軽な方法として`assert`をすでに紹介しました。  
しかし、コレだけだとさすがに不十分です。  
ユニットテストを素早く楽に開発、実行するために様々なテスティングツールがリリースされています。  
Javaの世界では[JUnit](http://junit.org/)がとても有名で、当然Groovyからも利用できます。  
しかし、Groovyの世界では、[Spock](https://github.com/spockframework/spock)というユニットテスト用ツールがデファクトスタンダートとなっています。  

今回はこのSpockを見て行きましょう。  
なお、Spock自体の詳細な仕様は公式ドキュメントを参照してください。

## 初めてのSpock
Spockはとてもシンプルでわかりやすいテストツールです。また、ライブラリとして配布されています。  
そう、Groovyの`@Grab`を使えば、Spockも簡単に導入することが出来ます。  
と言っても本来は、ビルドツールなどから、全てのテストを一気に実行することが一般的だと言うことを頭の片隅にとどめておいてください。  
まずはSpockってどんな感じに使うの？ということを知るのが一番大切です。  
GroovyとSpockの素晴らしい力を使って早速サクサクテストしていきましょう。

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

```terminal
JUnit 4 Runner, Tests: 1, Failures: 0, Time: 13
```
と表示されます。テストが1つ合って、エラーが0件、となっていますね。  
では、`1 > 0`というところを`1 == 0`に修正して再度実行してみましょう。

```terminal
JUnit 4 Runner, Tests: 1, Failures: 1, Time: 25
Test Failure: こんにちわSpock!(MyFirstSpock)
Condition not satisfied:

1 == 0
  |
  false

	at MyFirstSpock.こんにちわSpock!(ConsoleScript10:8)
```
上記のようなメッセージが表示されます。  
テストが1件合って、エラーが1件となっていますね。さらにどんなエラー内容だったかまで綺麗に出力してくれています。  

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

```terminal
JUnit 4 Runner, Tests: 2, Failures: 0, Time: 25
```
となります。  
テストが2件あり、エラーが0件ですね。
フィーチャーメソッドの個数が、テストの個数になっていることが解ると思います。  
また、Spockの機能を利用してフィーチャーメソッドを書いていくテスト用クラスは、`spock.lang.Specification`を継承します。

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
使い分けとしては、`expect`の方は副作用のない関数型的なメソッドなどのテストに、`when`と`then`はオブジェクトを操作してその中身を確認すると言った副作用を伴う動作のテストを行う際に使うと言った感じになります。（恐らく）


### cleanup
フィーチャーメソッドの掃除役です。  
書ける場所はフィーチャーメソッドの一番最後（後述するwhereブロックが有る場合はその直前）です。  
テストの為に`setup`でファイルを作成したりした場合、この`cleanup`ブロックで削除します。  
ただそれだけなのですが、1点重要なポイントが有ります。  
それは、例外が発生しようがしまいが、必ず実行されるという点です。  
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

```terminal
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
これはテストのデータを準備するために利用できます。  
一体どういうことでしょうか？実際の使用方法を見てしまえば一目瞭然です。


```groovy
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

```terminal
JUnit 4 Runner, Tests: 1, Failures: 0, Time: 9
```

です。
テストはエラーなしで無事終了しました。  
`where`に指定されているまるでMarkdownのテーブル表記のようなものを見てみてください。  
実は、今回のテストで利用されたデータはここから取得されています。  
テストで利用されたデータというと`expect`ブロックの`left`、`right`、`result`の３つですね。  
`where`ブロックの1行目は **変数名** になります。それぞれ`|`区切りで必要な変数を宣言していきます。  
そしてそれ以降の行に指定した値で、テストが繰り返し実行されます。  
つまり今回の場合、
 
| |left|right|result
|---|----|-----|------|
|1回目のテスト|1|2|3|
|2回目のテスト|4|5|9|
|3回目のテスト|9|10|19|


という、合計3回のテストが実行されています。  
なお、最後のresultの前にだけ`||`と、パイプが二つ付いていますね。  
これはテストの結果、この値になりますよ、ということを明示的に示すために慣例的に利用されている手法です。  
そのため、別に他の行同様に`|`でも構いませんが、慣例に習って`||`とした方がベターです。

`where`ブロックを使えば、同一のテスト対象コードを複数の値でテストするコードがスッキリ書くことができます。

#### @Unrollアノテーション
さて、whereにはちょっと特別なアノテーションが用意されています。それが`@Unroll`アノテーションです。  
単純に`where`ブロックを使っているフィーチャーメソッドの先頭に宣言するだけです。  
実際に使ってみましょう。  

```groovy
@Grab(group='org.spockframework', module='spock-core', version='1.0-groovy-2.4')
import spock.lang.*

class MyFirstSpock extends Specification {

    @Unroll
    def "#left と#right を足すと #result になる"() {  
        expect:
        left + right == result
        
        where:
        left|right|result
        1|2|3
        4|5|9
        9|10|19
    }
}
```
`@Unroll`が追加された以外に、メソッド名に`#`がついた変な記号がありますね。とりあえず気にしないでください。  
実行結果は、

```terminal
JUnit 4 Runner, Tests: 3, Failures: 0, Time: 49
```

実行結果は全く同じです。が、テストの数が3つに増えていますね！  
これには一体どんなメリットがあるのでしょうか？  
それを確認するために、`where`ブロックにテストが失敗するようなデータを追加してみましょう。

```groovy
@Grab(group='org.spockframework', module='spock-core', version='1.0-groovy-2.4')
import spock.lang.*

class MyFirstSpock extends Specification {

    @Unroll
    def "#left と#right を足すと #result になる"() {  
        expect:
        left + right == result
        
        where:
        left|right|result
        1|2|3
        4|5|9
        9|10|19
        1|1|100
    }
}
```

`where`ブロックに明らかにテストが失敗するデータを追加しました。  
実行結果は、

```terminal
JUnit 4 Runner, Tests: 4, Failures: 1, Time: 52
Test Failure: 1 と1 を足すと 100 になる(MyFirstSpock)
Condition not satisfied:

left + right == result
|    | |     |  |
1    2 1     |  100
             false

	at MyFirstSpock.#left と#right を足すと #result になる(ConsoleScript8:9)
```
となります。  
よくよく実行結果を見てみると、`Test Failure: 1 と1 を足すと 100 になる(MyFirstSpock)`と表示されていますね！  
ここは、テストが失敗したフィーチャーメソッド名が表示されています。  
`@Unroll`アノテーションを指定した場合、そのフィーチャーメソッドは`where`ブロックの行数分、それぞれ独立したテストのように繰り返し実行されます。（そのため、テストの回数が３回になっていました。）  
そして、フィーチャーメソッド名には、`where`ブロックで宣言した変数名が利用できるのです。それぞれ`#`で宣言している部分がそれです。  
そのため今回テストが失敗した`1|1|100`の部分がメソッド名に利用され、`Test Failure: 1 と1 を足すと 100 になる(MyFirstSpock)`と表示されたわけです。  

`@Unroll`を使うことによって、Spockのテストレポートを生成するツールなどを利用する際に、テストに失敗した箇所の内容がさらにハッキリわかるようになるメリットがあります。  
また、レポート生成ツールによっては、実行されたフィーチャーメソッド名を出力してくれますので、テストコードを見なくてもレポートを見ればどういったデータを使ってテストされたかどうかがわかります。



## フィクスチャーメソッド

さて、今までは直接テストに関わるフィーチャーメソッドとフィクスチャーブロックを見てきました。  
ここでは、テストの実行前後に利用できる特殊なメソッド、 **フィクスチャーメソッド** を見てみましょう。  


```groovy
@Grab(group='org.spockframework', module='spock-core', version='1.0-groovy-2.4')
import spock.lang.*

class MyFirstSpock extends Specification {
    
    def setup() {
        println "全てのフィーチャーメソッド実行前に実行されます。"
    }
    
    def cleanup() {
        println "全てのフィーチャーメソッド実行後に実行されます。"
    }
    
    def setupSpec() {
        println "全体のテストの実行前に1度だけ実行されます。"
    }
    
    def cleanupSpec() {
        println "全体のテスト実行後に1度だけ実行されます。"
    }
    
    def "test 1"() {
        expect:
        println "test 1"
        1 == 1
    }

    def "test 2"() {  
        expect:
        println "test2/${a}"
        a == b
        
        where:
        a|b
        1|1
        2|2
    }
    
    @Unroll
    def "test 3"() {  
        expect:
        println "test3/${a}"
        a == b
        
        where:
        a|b
        1|1
        2|2
    }
}
```

実行結果は、

```terminal
全体のテストの実行前に1度だけ実行されます。
全てのフィーチャーメソッド実行前に実行されます。
test 1
全てのフィーチャーメソッド実行後に実行されます。
全てのフィーチャーメソッド実行前に実行されます。
test2/1
全てのフィーチャーメソッド実行後に実行されます。
全てのフィーチャーメソッド実行前に実行されます。
test2/2
全てのフィーチャーメソッド実行後に実行されます。
全てのフィーチャーメソッド実行前に実行されます。
test3/1
全てのフィーチャーメソッド実行後に実行されます。
全てのフィーチャーメソッド実行前に実行されます。
test3/2
全てのフィーチャーメソッド実行後に実行されます。
全体のテスト実行後に1度だけ実行されます。
JUnit 4 Runner, Tests: 4, Failures: 0, Time: 30
```

となります。

まとめると、

|メソッド名|意味|
|---------|---|
|setup|全てのフィーチャーメソッド実行前に実行されます。|
|cleanup|全てのフィーチャーメソッド実行後に実行されます。|
|setupSpec|全体のテストの実行前に1度だけ実行されます。|
|cleanupSpec|全体のテスト実行後に1度だけ実行されます。|

となります。

## 各フィーチャーメソッドで共通して利用する変数
ここでワンポイントメモです。  
毎回フィーチャーメソッドで利用したい共通の変数がある場合、当然フィールドとして宣言する必要があります。  
上記の`setup`や`setupSpec`フィクスチャーメソッドでその変数を初期化することもできますが、単純な変数なのであれば、テストクラスのフィールドとして宣言すると同時に初期化すれば同様の効果があります。  
つまり、フィールド変数の宣言のタイミングで初期化するもよし、宣言だけして`setup`や`setupSpec`の中で初期化するもよしです。  
なお、フィールドに宣言した値は、 **フィーチャーメソッドの実行の度に再度実行されます。**

```groovy
@Grab(group='org.spockframework', module='spock-core', version='1.0-groovy-2.4')
import spock.lang.*

class MyFirstSpock extends Specification {

    Date date = new Date()

    def "test"() {  
        expect:
        sleep(500)
        println date.getTime()
        a == b
        
        where:
        a|b
        1|1
        2|2
        3|3
    }
}
```

実行結果は、

```terminal
1453245037474
1453245037980
1453245038481
```
となります。このように、`Date date = new Date()`がテストごとに実行されて、`date`の中身が毎回変わっていることがわかると思います。  
可能であれば宣言と同時に初期化、初期化のために複雑な計算や処理が必要な場合は`setup`や`setupSpec`の中で初期化すればよいでしょう。  

しかし、当然今回のようにDateの値をテストに使うような場合、全てのフィーチャーメソッドで同じ値で利用したい場合もあるでしょう。  
その場合は単純にフィールド変数の左に`@Shared`をつけるだけでOKです。

```groovy
@Grab(group='org.spockframework', module='spock-core', version='1.0-groovy-2.4')
import spock.lang.*

class MyFirstSpock extends Specification {

    @Shared Date date = new Date()

    def "test"() {  
        expect:
        sleep(500)
        println date.getTime()
        a == b
        
        where:
        a|b
        1|1
        2|2
        3|3
    }
```

実行結果は以下のようになります。  

```terminal
1453245425610
1453245425610
1453245425610
```
Dateの中身が全て同じになっていることがわかりますね。

## 実践
では、実際にテスト対象のクラスを作成して、テストしてみましょう。  
今回は、Personというクラスを用意します。Personはageプロパティを持ちます。  
そして、`isAdult()`メソッドで、ageが20以上の場合はtrue、19以下の場合はfalseを返します。  
この`isAdult()`をテストしてみましょう。

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

```terminal
JUnit 4 Runner, Tests: 3, Failures: 0, Time: 29
Result: org.junit.runner.Result@6c0eb436
```

特に難しい点はありませんね。 

1点注意。GroovyConsoleでSpockを色々試して見る際に、上記の`Personクラス`のように同時にテスト用クラスも宣言できますが、テスト用クラスをテストの前に宣言すると、GroovyConsoleが何を実行していいのかわからなくなって、エラーになってしまいます。  

```terminal
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

## テストをより分かりやすくする（BDDスタイル）
さて、宣言的な記述が出来るGroovyとSpockですが、よりさらにテストを分かりやすく書くことが出来ます。  
先程のコードに、誕生日を迎えた際に年齢を1カウントアップするメソッドを追加して、それが正しく動作しているかを確認するようにしてみましょう。

```groovy
@Grab(group='org.spockframework', module='spock-core', version='1.0-groovy-2.4')
import spock.lang.*

class MyFirstSpock extends Specification {

    @Unroll
    def "PersonクラスのisAdultメソッドをテストするよ"() {
    
        given: "Personインスタンスを生成して準備をする"
        Person p = new Person()
        and: "年齢をセットする"
        p.age = age
                
        when: "誕生日を迎える"
        p.birthday()
        
        then: "1歳年をとる"
        p.age == age + 1
        and: "20を超えると成人"
        p.isAdult() == result

                
        where:
        age || result
        18 || false
        19 || true
        20 || true
    }
}

class Person {
    Integer age = 0
    
    Boolean isAdult() {
        age >= 20
    }
    
    // これを追加。
    def void birthday() {
        age++
    }
}
```

さて、ぱっと目につく大きな違いが有ります。   
それは、 **フィクスチャーブロックにコメントが入っている** 、という点です。  
これはプログラムで言うコメント（`/*comment*/`, `//comment`）とは根本的に違う、Spock独自の構文です。  
このコメント自体はプログラム（テスト）の動作に全く影響を与えません。  
これは、テスト自体ではなく、そのフィクスチャーブロックが何を意味するものか、を説明するために用いるものです。  

`given:`については、単純に`setup:`の別名です。  
`and:`はそれ以降に記述するもののラベリング目的に利用するもので、フィクスチャーメソッドのトップレベルであればどこでも記述することが出来ます。  
上記のサンプルコードでは、`given:`と`then:`の後でそれぞれ利用しています。  

基本的にはコメント加えただけでしょ？と思われるかも知れません。基本的にはそのとおりです！  
しかし、`given:`、`when:`、`then:`という用語を用いて、テスト対象の振る舞いを明示するBDDスタイルを用いることで、よりよい表現方法でテストが記述できるようになります。  
BDDでは、今まで述べていた通常のUnitテストと技術的に何かが違う、というものではなく、どういったスタンスでテストを記述するのか、という点が異なります。  

`given:`: givenに記述した **状態の時** 、  
`when:`：whenにある **操作が行われると** 、  
`then`：thenに記述された **状態になる** 。  

という形式でテストを記述していきます。  
こうすることで、表現方法とプロセスが統一されるので、テストがより分かりやすいものになっていく、というものです。

SpockのUnitテストによってテスト対象の動作がテストされ、安全なコードになります。    
その上で更に、各フィクスチャーブロックが何をしようとしているのかを統一された用語とスタイルで示すBDD形式で記述することで、Spockのテスト自体がテスト対象の仕様書になります。  


##まとめ
Spockを使うと簡単にテストが実行できることが分かったと思います。  
また、`where`を使うことでデータ駆動テストが簡単に実現できます。  
今回はSpockの使い方を学ぶということを念頭に置いたのでGroovyConsoleで実行しました。  
実際のプロダクトでは、テストを記述するファイルには本当にテストだけを記述して、Gradleなどのビルドツールから一気にテストを実行することになります。  
また、Groovy向けのWEBフレームワークであるGrailsは、標準でSpockを搭載していて、さらにWEB向けのテストがしやすいように専用の機能が追加で提供されています。