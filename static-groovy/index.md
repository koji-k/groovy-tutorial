# 静的Groovy

さて、動的型付け言語Groovyですが、実はコンパイル時(`groovyc`)に型のチェックを行うことが出来ます。  
型のチェックが行えるので、JavaやScalaのようにコンパイルで実行前に明らかな変数の代入などのエラーが検出できます｡  
また、コンパイルされると当然classファイルが生成されますが、classファイルの中でも型情報が保持されていますので、実行速度の高速化が期待できます。  
方法は簡単で、クラスの先頭、もしくは型チェックを行いたいメソッドの頭に`CompileStatic`を付けるだけです。  


```
import groovy.transform.CompileStatic

class A {
    def hoge(){}
}
class B {
    def piyo(){}
}

@CompileStatic
class Test {

    def Test() {
        // 存在しないメソッドをコンパイルエラーに出来る。なお、オーバロードされているメソッドもちゃんとチェックしてくれる
        //unknownMethod()
    }

    // def で引数を宣言するとObject型になるので、Object型にabsというメソッドは存在しないため、コンパイル時点でエラーに出来る
    def foo (def a) {
        //a.abs()
    }

    // Integer型にtoLowerCaseというメソッドは無いので、コンパイル時点でエラーに出来る
    def foo(Integer a) {
        //a.toLowerCase()
    }

    def propertyTest() {
        //Integer a = "a" // コンパイルエラーに出来る
        A aObj = new A()
        B bObj = new B()
        //aObj = bObj // ここでコンパイルエラーに出来る

        // TypeChecked or CompileStaticをつけていても、変数をdefで宣言するとコンパイルエラーに出来ない
        def aObj2 = new A()
        def bObj2 = new B()
        aObj2 = bObj2

        //aObj.piyo() // 宣言時に型の指定があるので存在しないメソッド呼び出しはコンパイルエラーに出来る
        //aObj2.piyo() // 宣言時に型の指定が無いので存在しないメソッド呼び出しはコンパイルエラーに出来ない
    }
}
```

Scalaなどでは `val a = "aaa"`とした場合はちゃんとString型になりますが、Groovyの場合、もし変数宣言に`def`を利用した場合、その型は **Object** になります。  
これはメソッドの引数でもローカル変数でも同じです。

上記のコードの中から例えばメソッド`foo`をjavapで見てると

```
[koji:type]$ /usr/lib/jvm/jdk1.8.0_60/bin/javap -l Test | grep foo 
  public java.lang.Object foo(java.lang.Object);
  public java.lang.Object foo(java.lang.Integer);

```
のように、`def`を使った方はObject型になっています。

さらに、ローカル変数を見てみると


```
[koji:type]$ /usr/lib/jvm/jdk1.8.0_60/bin/javap -l Test | grep aObj
         24      79     2  aObj   LA;
         61      42     4 aObj2   Ljava/lang/Object;
```

となっており、やはり`def`で宣言した方はObject型になっていることが分かります。

# まとめ
JavaやScalaほど厳密な型推論は静的Groovyでは利用できません。  
しかし、Groovyは動的型付け言語のメリットを活かしたメタプログラミング等を便利に利用することが出来ます。  
静的Groovy（CompileStatic）を利用することで、速度の改善が図れる部分も有りますが、同時にGroovyらしさを失う事にもなります。  
色々試してみて、自分のプロダクトの何処にCompileStaticが導入出来るか調べてみましょう。  