# クラス

GroovyのクラスはJavaのクラスと全く同じです。  
すでに何度かサンプルコードのクラスを使いましたが、改めてクラスの宣言方法を示します。  

```groovy
class Hoge {

    def name

    // エントリポイント
    static main(args) {
        def hoge = new Hoge()
        hoge.sayHello()
    }   

    // コンストラクタ
    def Hoge() {
        name = "hoge"
    }   

    // 普通のインスタンスメソッド
    def sayHello() {
        println "Hello ${name} !!"
    }   
}
```

凄くオーソドックスですね。  
当然、Javaの用にメソッドの引数や戻り値の型を指定することも出来ます。  

```groovy
String nextAge(Integer age) {
    "next age is ${age + 1}"
}
```

## クラスは2つ以上定義しても良い  
Javaでは、ルールとして全てのプログラムはクラス名と同様のファイル名に保存する、というルールが有ります。  
そのため、例えばHogeクラスを記述したHoge.javaというファイルの中には原則Hogeクラスしかかけません。  
しかしGroovyにこの制限はありません！  
つまり、一つのgroovyファイルの中に複数のクラスを記述できます。  
その上、クラスの外にコードを書くことさえ出来ます。JavaというよりはPHPのような動作ですね。  
例えば以下のソースを`Executor.groovy`というファイルに保存してみてください。

```groovy
class Hoge {
    def Hoge(){
        println "This is Hoge"
    }
}

class Piyo {
    def Piyo(){
        println "This is Piyo"
    }
}

new Hoge()
new Piyo()
```

これでコマンドラインで`groovy Executor.groovy`と実行してみてください。ちゃんとクラス外のコードから2つのクラスが実行されているのが解ると思います。


## セッターとゲッター
さて、Javaの世界ではメンバ変数はprivateにして、メンバ変数には専用のセッター/ゲッターメソッドを経由してアクセスする、というルールが有ります。  
Groovyには残念ながら（幸運にも？）privateがありません。  
そのため、メンバ変数にはどこからでもアクセスできます。  

```groovy
class Hoge {
    String name
    
    Hoge(String name) {
        this.name = name
    }
}

def hoge = new Hoge("test")
assert hoge.name == "test"
```
つまり、Javaで言うところのパブリックなメンバ変数に直接アクセスしているという感じですね。  
そう、Groovyでは態々セッターとゲッターを書く必要はないんですね。  
しかし、それだともしセッターやゲッターの中で少し処理をする、と言ったことに対応できないではないか、という話になります。  
大丈夫です。Groovyでもセッターとゲッターを宣言できます。
方法は、Javaと全く同じです。メンバ変数の頭にset/getをつけて、メンバ変数の先頭を大文字にしたものをくっつけてメソッド名とするだけです。  
まずはセッターのサンプルを見てみましょう。  

```groovy
class Hoge {
    String name
    
    Hoge(String name) {
        this.name = name
    }
    
    def setName(String name) {
        this.name = "${name}!!"
    }
}

def hoge = new Hoge("test")
assert hoge.name == "test"

hoge.name = "test1"
assert hoge.name == "test1!!"
```

どうでしょう？  
見てもらえれば分かると思いますが、クラスの外からそのメンバ変数に直截アクセスしているように見える箇所（`hoge.name = "test1"`）が有りますが、もしクラスがセッターを持っていた場合には、Groovyが自動的にそのセッターを使ってくれます。  
つまり、後から仕様が変わってセッターを用意して、その中で処理をしたい、となった場合でも、セッターを追加するだけでOKです。メソッドの呼び出し元を変える必要はありません。  

ではゲッターのサンプルも見てみましょう。


```groovy
class Hoge {
    String name
    
    Hoge(String name) {
        this.name = name
    }
    
    def getName() {
        "${name}!!"
    }
}

def hoge = new Hoge("test")
assert hoge.name == "test!!"
```

自動的にゲッター経由で値が取得できていますね。
