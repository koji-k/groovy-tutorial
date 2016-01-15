# 変数

Groovyで変数を宣言するには、通常以下の３種類が有ります。  

- defキーワード  
- 型の変数名を同時に宣言  
- いきなり変数名  

##defキーワード
GroovyはまるでPHPやRubyの用に簡単に変数を宣言することが出来ます。  
方法は簡単で`def`というキーワドの後に、変数名を記述するだけです。  
それでは実際に **GroovyConsole** などで以下の内容を内容を記述して実行してみましょう。

```groovy
def a = 1
def b = 0.1
def c = "hello groovy"

println a
println b
println c

// 中身を交換
a = c
println a
```

どうですか？ちゃんと実行結果が表示されましたか？  
GroovyはJVM上で動いているので、厳密には全ての変数に型が存在します。  
今回の `def`キーワードで変数を宣言した場合、Javaの`Object`という型になります。  
今はあまり意識する必要は無いと思います。  

## 型の変数名を同時に宣言
つまりJavaの変数の宣言方法と同じです。  
実際にコードを見てみましょう。  

```groovy
Integer a = 1
Double b = 0.1
String c = "hello groovy"

println a
println b
println c
```

`def`キーワードの代わりに、`Integer`、`Double`、`String`という物が使われています。  
Groovyは全てのJavaの型を利用することが出来ます！  
Javaにどんな型があるのかどうかなどはググっていただければ山ほど出てきます。  

さて、上記のコードを少し修正して、ココでも変数の中身を交換してみましょう。  

```groovy
Integer a = 1
Double b = 0.1
String c = "hello groovy"

println a
println b
println c

// 中身を交換
a = c
println a
```

これを実行すると何故かエラーが出ますね？

```console
1
0.1
hello groovy
Exception thrown

org.codehaus.groovy.runtime.typehandling.GroovyCastException: Cannot cast object 'hello groovy' with class 'java.lang.String' to class 'java.lang.Integer'
	at ConsoleScript3.run(ConsoleScript3:10)
```

`def`キーワードの場合は内部的に`Object`型になるのでどんな内容でも変数に代入出来ますが、型を指定した場合はJavaと全く同じなので、当然異なる型の値は代入できなくなるわけです。  
注意点としては、コンパイル時点ではこのエラーは確認できない点です。  
Javaであればjavacでコンパイルした時点でこのエラーを検知できますが、Groovyは実行時に型をチェックします。  
そのためプログラムは実行するまでこのエラーは検知できません。  
つまり、コンパイラでこうしたエラーが検知できないわけですが、例えばメソッドの引数に型を宣言しておくと、実行時では有ります、意図しない型の値が渡された場合はそのメソッドの中身自体は実行されずエラーになります。  
とうことは、そのメソッドの中で態々渡された値が意図した型なのかどうかをチェックする必要がなくなるわけです。  
GroovyはこうしてJavaの安全性、PHPなどの高速な開発性をうまく統合しています。  
この変数の型を宣言する意味については、[Groovyでメソッドの引数に型を指定する意味](http://kanndume.blogspot.de/2013/02/groovy.html)に詳細が有ります。  
ある程度Groovyに慣れたら見てみてもいいかもしれません。


## いきなり変数名
これを使うタイミングは、クラスのメソッドの仮引数や、後述するClosureの引数です。  

```groovy
class Hoge {
    Hoge(){
        println "Constructor"
    }
    
    def piyo(value) {
        println value
    }
}

def hoge = new Hoge()
hoge.piyo("this is my value")
```

上記コマンドを実行すると、`piyoメソッド`に渡した値がそのまま画面に出力されます。  
`piyoメソッド`の`value`という仮引数自体に`def`も`型`もついていません。  
コレは結果的には`def`を用いた場合と同じく`Object`型の変数として扱われます。

## まとめ
通常のプログラミングで変数を宣言する場合は、`def`か`型`を指定する形になります。  
Javaなどの強い静的型付けな世界で生きていた人はGroovyの`def`の便利さを試しながら、基本的には型を指定した変数の宣言方法がいいと思います。  
今まで変数の型を意識することがあまりなかった人は、`def`キーワードを用いてGroovyに慣れつつ、変数の型についてJavaに関する書籍やwebサイトで型について学んでいくのが近道だと思います。  
GroovyはJavaを駆逐するものではなく、Javaと共存するものなので、Groovyを学べば同時にJavaの勉強にもないます。  

##Other memo
[メソッドのオーバロードについて](http://qiita.com/saba1024/items/73c8d89d999d42839571)  
