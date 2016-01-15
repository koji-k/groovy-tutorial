#Install


通常は[SDKMAN](http://sdkman.io/)というツールでインストールします。  
なにはともあれ[Qiitaの記事を参照](http://qiita.com/saba1024/items/66c7154608ed52363e8f)をしていただければインストールできるはずです。  
[Windowsの場合](http://qiita.com/saba1024/items/66c7154608ed52363e8f#windows)  
[Mac、Linuxの場合](http://qiita.com/saba1024/items/66c7154608ed52363e8f#mac-linux)  

# Hello World
それではHelloWorldを実行してみましょう。  
GroovyConsoleを起動して、以下の内容を入力して実行してみてください。  
GroovyConsoleの起動方法は[Qiitaのこの記事](http://qiita.com/saba1024/items/66c7154608ed52363e8f#%E3%83%97%E3%83%AD%E3%82%B0%E3%83%A9%E3%83%A0%E3%81%AE%E5%AE%9F%E8%A1%8C)を参照してください。

```groovy
println "Hello World"
```

どうですか？画面にちゃんと **Hello World** と表示されましたか？  
それでは続いて計算もしてみましょう。  

```groovy
println 2 * 3
```
さすがGroovy。掛け算もへっちゃらです。  
でも、計算結果だけ画面に表示されてもなんのことだかわかりませんね。少しメッセージも加えてみましょう。  

```groovy
println "result is ${2 * 3}"
```

どうでしょう。少し見栄えが良くなりましたね。  
Groovyは、文字列の中に`${...}`という箇所があると、その中をGroovyのコードとして扱います。その結果が文字列に結合されます。  
Groovyコードなので、当然以下のように変数も使えます。

```
def hoge = "hogehoge"
println "my name is ${hoge}"
```