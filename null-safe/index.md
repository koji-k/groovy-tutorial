# Nullセーフ
さて、悪名高い`NullPointerException`をご存知でしょうか？  
Javaでプログラミングしたことがある人は必ず見たことのある例外です。  
変数に何も格納されていないのにアクセスすると発生します。  

```groovy
// Stringの値をIntegerにして2倍する
String npe = null
npe.toInteger() * 2
```

```terminal
Exception thrown

java.lang.NullPointerException: Cannot invoke method toInteger() on null object
	at ConsoleScript1.run(ConsoleScript1:2)
```

旧来の対処方法としては、変数に値が入っているかどうかを処理の前にif文でチェックしていました。  

```groovy
// Stringの値をIntegerにして2倍する(nullの場合は0を返す）
String npe = null
if (npe != null) {
    return npe.toInteger() * 2
} else {
    return 0
}
```

## Java8のOptional
Java系言語を使う上で避けては通れないこの`NullPointerException`ですが、Java8からは`Optional`という仕組みが用意されました。  
これは、 **nullかも知れない値を包む** ものです。  
例えば上記でエラーになった、Stringの値をIntegerにして2倍する、という機能をOptionalを使って以下のように記述できます。

```groovy
String npe = null
Optional<String> nptOpt = Optional.ofNullable(npe)
assert 0 == nptOpt.map {it.toInteger() * 2}.orElse(0)
```

コードからif文が消えましたね！  
GroovyはJavaなので、このようにJavaのOptionalを利用して安全にコードを書けます。  

上記の`map`メソッドは、Optionalで包まれたデータを操作して返す、というメソッドです。  
mapとは別に`filter`というメソッドも有り、コチラは指定した条件に合致する場合はそのままOptionalで包まれた値を返し、一致しない場合は`Optional.empty()`を返す、というメソッドです。

```groovy
assert Optional.empty() == Optional.ofNullable(null).filter{it % 2 == 0}
assert Optional.empty() == Optional.ofNullable(1).filter{it % 2 == 0}
assert Optional.empty() != Optional.ofNullable(2).filter{it % 2 == 0}
assert Optional.ofNullable(2) == Optional.ofNullable(2).filter{it % 2 == 0}
```

なお、Optional.empty()は当然nullでありません。


## Groovyのセーフナビゲーション
Java8のOptionalの登場以前から、より簡潔なNullセーフな手段をGroovyは提供しています。  
それがセーフナビゲーションと呼ばれる`?.`という記法で利用する機能です。  
以下のコードを見てみてください。  

```groovy
String npe = null
assert null == npe?.toInteger()?.class?.name
assert null == null?.toInteger()?.class?.name
assert "java.lang.Integer" == "1"?.toInteger()?.class?.name
```

メソッドやプロパティへのアクセスが`.`の代わりに`?.`を利用しただけでです。  
この`?.`を使うとGroovyはレシーバがnullの場合、レシーバが実行しようとしているメソッド or クロージャは実行せずに単純にnullを返します。  
その返されたnullをチェーンして、再度別のメソッドが呼ばれる際にも`?.`を利用すれば同様にnullが返されて...という処理が最後まで続き、最終的には当然nullが返されます。  
また、[If文](/groovy-tutorial/if/index.html)で述べた`エルビス演算子`と併用する事も当然可能です。

```groovy
def a = "1"?.toInteger()?.class?.name ?: 'this is null'
assert a == 'java.lang.Integer'

def b = null?.toInteger()?.class?.name ?: 'this is null'
assert b == 'this is null'
```


どちらが優れているかどうかではなくて、場合によってどちらを利用するかを切り分けるのがいいと思います。  
個人的には簡潔なGroovyのセーフナビゲーションの方が好きですが、`Optional.filter`に該当する機能が無いので、もし`Optional.filter`を利用しないのであれば三項演算子を利用する必要が有ります。  
そういった細かな違いもあるので、まずは色々試してみることをおすすめします。  

## サンプルコード

さて、それではココで今回の総まとめとして、  
**nullを含む、String型のリストの各値をIntegerに変換して、整数かつ偶数の値を抽出して2倍して合計する**  
という処理を、Groovyのコレクションの機能を使いつつ、Java8のOptionalとGroovyのセーフナビゲーションで実装してみます。

```groovy
List list = ["-5", "-4", null, "-3", "-2", null, "-1", "0", null, "1", "2", "3", null, "4", "5", null]

// Java8のOptional版
def optionalVersion = {->
    list.collect {
        Optional.ofNullable(it)
    }.collect {
        it.map{it.toInteger()}
    }.collect {
        it.filter{it % 2 == 0 && it > 0}
    }.collect {
        it.map{it * 2}
    }.collect {
        it.orElse(0)
    }.sum()
}
assert 12 == optionalVersion()

// Groovyのセーフナビゲーション版（エルビス演算子も利用）
def safeNaviVersion = {->
    list.collect {
        it?.toInteger()?:0
    }.findAll {
        it % 2 == 0 && it > 0
    }.collect {
        it * 2
    }.sum()
}
assert 12 == safeNaviVersion()
```

どちらも結果は同じです。  


## その他の方法
JavaのOptionalは、中身が1つあるか何もないリストみたいなものなので、Groovyのリストを便利に扱う機能を利用することで代用することも出来ます。  

```groovy
def list = []
assert "empty" == list ? list.head() : 'empty'

list = ["yeay"]
assert "yeay" == list ? list.head() : 'empty'

// GroovyでもメタプログラミングでListに機能を追加すればよりスマートに表現できる！
// ListクラスにorElseメソッドを追加
List.metaClass.orElse = {def alternativeValue ->
    delegate ? delegate.head() : alternativeValue
}

assert "yeay" == list.orElse(99)
list = []
assert 99 == list.orElse(99)
assert 'java.lang.Integer' == list.orElse(99).class.name
```