#色々な実行方法

Groovyのプログラムを実行するために、以下の4つの方法があります。

+ GroovyConsole  
+ Groovysh  
+ ファイルに保存してスクリプトとして実行  
+ ファイルに保存してコンパイル&実行  

以下それぞれ説明していきます。  
なお、このチュートリアルでは特に注釈のない限り、プログラムの実行は **GroovyConsole** を想定します。

## GroovyConsole
さっきHelloWorldで使いましたね！

## Groovysh
コンソールを開いて`groovysh`と実行します。  
すると、１行ずつGroovyのコードを実行することができる、いわゆる **REPL** が起動します。
簡単なコードなどはこのgroovyshで確認するのがオススメです。

## ファイルに保存してスクリプトとして実行
難しいことは何一つありません。  
適当なディレクトリで、 **hoge.groovy** という名前でファイルを作成しましょう。  
そのファイルの中に以下を記述します。  

```groovy
println "Hello Groovy"
println 1 + 1
```

ファイルを保存したら、ターミナルを起動して、そのファイルがある場所まで移動して、以下のコマンドを実行します。  
`groovy hoge.groovy`
するとターミナルに実行結果が以下のように表示されます。

```
Hello Groovy
2
```

## ファイルに保存してコンパイル&実行
Javaのように一度classファイルにコンパイルしてから実行することが可能です。    
Groovyをコンパイルするコマンドは`groovyc`です。  
先ほどのhoge.groovyで試してみましょう。

```
groovyc hoge.groovy
```

すると、 **hoge.class** というファイルが生成されますね。  
ここで先ほどのgroovyコマンドを実行してみましょう。  
なお、Javaと同様コンパイルされたclassファイルを実行する際には、 *.class* は省略します。

```
groovy hoge
```

先ほどの「ファイルに保存してスクリプトとして実行」と同様以下のような実行結果が表示されますね。

```
Hello Groovy
2
```

実は、groovyコマンドでコンパイルせずにgroovyファイルを実行すると、この **groovycコマンドによるコンパイル** と **groovyコマンドによるclassファイルの実行** をGroovyが同時に行ってくれています。


###ファイル名と生成されるclassファイル名の関係
少し余談になりますが、今回 **hoge.groovy**というファイルを`groovyc`コマンドで実行すると **hoge.class**というclassファイルが生成されました。  
では、一旦 **hoge.groovy** と **hoge.class** を削除してください。  
その上で再度 **hoge.groovy** を作成して、以下の内容を記述してください。  

```groovy
class Foo{
    static main(args) {
        println "This is Foo class in hoge.groovy"
    }
}
```

詳細は後の章で述べますが、これはGroovyに置ける普通のクラスを宣言になります。  
では`groovyc`コマンドでコンパイルしてみましょう。  

```
groovyc hoge.groovy
```

実行してみると **Foo.class** というファイルが生成されています。  
先ほどは **hoge.class** というファイルが作成されましたよね。。。？これはどういうことでしょう？  
では、またまた両方のファイルを削除して、再度 **hoge.groovy** を作成して、以下の内容を記述してください。  

```groovy
class Foo{
    static main(args) {
        println "This is Foo class in hoge.groovy"
    }
}

println "Out of Foo"
```

これを`groovyc hoge.groovy`でコンパイルすると **Foo.class** と **hoge.class** という2つのclassファイルが出来上がりました。  
このことから、クラスとして宣言されている部分は`そのクラス名.class`、クラスの外側に記述されている部分は`そのファイル名.class`というファイル名になることがわかります。
