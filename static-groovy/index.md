# 静的Groovy

さて、動的型付け言語Groovyですが、実はコンパイル時(`groovyc`)に型のチェックを行うことが出来ます。  
型のチェックが行えるので、JavaやScalaのようにコンパイルで実行前に明らかな変数の代入などのエラーが検出できます｡  
また、コンパイルされると当然classファイルが生成されますが、classファイルの中でも型情報が保持されていますので、実行速度の高速化が期待できます。  
方法は簡単で、クラスの先頭、もしくは型チェックを行いたいメソッドの頭に`CompileStatic`を付けるだけです。  


```
import groovy.transform.CompileStatic
import groovy.transform.TypeChecked

class A {
    def hoge(){}
}
class B {
    def piyo(){}
}

@CompileStatic
//@TypeChecked // あとで説明
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
        //Integer a = "a"        // コンパイルエラーに出来る
        A aObj = new A()
        B bObj = new B()
        //aObj = bObj // ここでコンパイルエラーに出来る

        // ローカル変数の場合、型推論が働く！
        def aObj2 = new A()
        def bObj2 = new B()
        aObj2 = bObj2 // ココは謎挙動。javapの結果、aObj2はAクラス、bObj2はBクラスとなっている、この代入を実行するとaObj2はObject型になる
        //aObj.piyo() // 宣言時に型の指定があるので存在しないメソッド呼び出しはコンパイルエラーに出来る

        // 上で少し触れたけど、defでもローカル変数の場合はちゃんと型情報が型推論される。
        def aObj3 = new A()
        //aObj3.piyo() // なので型推論されてaObj3はA型という事がわかっているので、ちゃんとコレもコンパイルエラーに出来る
    }
}
```

## メソッド
もしメソッドの引数の変数宣言に`def`を利用した場合、その型は **Object** になります。  

上記のコードの中から例えばメソッド`foo`をjavapで見てると

```
[koji:type]$ /usr/lib/jvm/jdk1.8.0_60/bin/javap -l Test | grep foo 
  public java.lang.Object foo(java.lang.Object);
  public java.lang.Object foo(java.lang.Integer);

```
のように、`def`を使った方はObject型になっています。

## ローカル変数

まず、ローカル変数を見てみます。


```
[koji:type]$ groovyc test.groovy | /usr/lib/jvm/jdk1.8.0_60/bin/javap -l Test | grep aObj
          8      66     1  aObj   LA;
         28      46     3 aObj2   Ljava/lang/Object;
         61      13     6 aObj3   LA;
[koji:type]$ groovyc test.groovy | /usr/lib/jvm/jdk1.8.0_60/bin/javap -l Test | grep bObj
         18      56     2  bObj   LB;
         39      35     4 bObj2   LB;
[koji:type]$ groovyc test.groovy | /usr/lib/jvm/jdk1.8.0_60/bin/javap -l Test | grep typeCheckedOrCompileStatic
         68       6     7 typeCheckedOrCompileStatic   I
[koji:type]$ 
```
*ちなみに、aObj2がObject型になっていますが、この理由はわかりません。*  
*恐らく別の型の変数に代入したため、整合性を取るためにObject型になったのだと思われますが、本来はその時点でコンパイルエラーになるべき。。。とりあえず無視します。（要調査）*  

さて、上記のjavapの結果を見れば一目瞭然ですが、ローカル変数の場合は、メソッド引数のdefと違い、ちゃんと代入される値に基づいて **型推論が働きます。**  
そのため、例えば変数`typeCheckedOrCompileStatic`はちゃんとInteger型として判断されています。  


### CompileStaticとTypeChecked
さて、ではココでその変数`typeCheckedOrCompileStatic`を使って`@TypeChecked`と`@CompileStatic`の違いを見てみましょう。  
前提条件として、どちらのアノテーションを利用していたとしても、コンパイル時に型推論が働くので、例えば`typeCheckedOrCompileStatic`に対して`String#toLowerCase()`の様なメソッドを記述しておくとちゃんとコンパイルエラーになります。  

```
def typeCheckedOrCompileStatic = 111
typeCheckedOrCompileStatic.toLowerCase() // コレを追加
```

コンパイルしてみると、

```
[koji:type]$ groovyc test.groovy                                                        
org.codehaus.groovy.control.MultipleCompilationErrorsException: startup failed:
test.groovy: 47: [Static type checking] - Cannot find matching method int#toLowerCase(). Please check if the declared type is right and if the method exists.
 @ line 47, column 9.
           typeCheckedOrCompileStatic.toLowerCase()
           ^

1 error

```

ちゃんとint型にtoLowerCaseなんて無いよ、というメッセージが表示されています。  
では`@TypeChecked`と`@CompileStatic`で何が違うのかというと、  
`@TypeChecked`の場合はコンパイル時のチェックの時にのみ、defで宣言されたローカル変数を型推論してチェックしますが、生成される **classファイルにはその情報は残っておらず、Object型になります。**  
`@CompileStatic`の場合は、同様にコンパイル時に型推論が働き、さらに **classファイルのその情報が残ります。**

```
# @TypeChecked
[koji:type]$ groovyc test.groovy | /usr/lib/jvm/jdk1.8.0_60/bin/javap -l Test | grep typeCheckedOrCompileStatic
        114       3     8 typeCheckedOrCompileStatic   Ljava/lang/Object;

# @CompileStatic
[koji:type]$ groovyc test.groovy | /usr/lib/jvm/jdk1.8.0_60/bin/javap -l Test | grep typeCheckedOrCompileStatic
         68       6     7 typeCheckedOrCompileStatic   I

```


# まとめ
Groovyでも型推論がちゃんと働くことが分かりました。  
しかし、静的Groovy（CompileStatic）を利用することで、速度の改善が図れる部分も有りますが、同時にGroovyらしさを失う事にもなります。  
Groovyの動的型付け言語のメリットを活かしたメタプログラミング等の利用が制限されてしまうためです。  
その一つの対策案として、Groovyでは必要なメソッドのみにCompileStaticを指定できるようになっています。  
色々試してみて、自分のプロダクトの何処にCompileStaticが導入出来るか調べてみましょう。  