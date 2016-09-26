# if文

Groovyのif文には便利ないくつかの機能があります。  

## 普通のif
一般的なプログラミング言語と同様のifの使い方ができます。  

```groovy
def value = 1
if (value == 1) {
    println "value is 1"
} else if (value == 2) {
    println "value is 2"
} else {
    println "value is...???"
}
```

特に説明が不要なレベルですね。  
これが最も基本的なifの使い方になります。

## assert
Groovyには`assert`と呼ばれる機能が用意されています。  
コレは、Groovyの実行時に、`assert`に指定された条件がOKの場合は何もせず、条件を満たさない場合にはエラーを発生させ、どんな問題があるのかを表示してくれるものです。  
例えば
 
```groovy
assert 1 == 1
```
と書いて実行した際には何も起きませんが、

```groovy
assert 1 == 2
```

を実行した際、当然この条件は成り立ちませんので、以下のエラーが発生します。

```groovy
Assertion failed: 

assert 1 == 2
         |
         false

	at ConsoleScript14.run(ConsoleScript14:3)
```

サンプルコードを例示する際にとても便利な機能です。  
以下、この`assert`を利用してサンプルコードを例示します。


## 三項演算子を使う
Groovyで三項演算子を使うと、ifでreturnで値を返さなくても、自動的に最後に表がされた値が返されます。  
実際に見てみましょう。  

```groovy
def value = 1
assert "This is 1" == value ? "This is 1" : "This is not 1"
```
どこにもreturnがないのに、ちゃんと"This is 1"という値が帰っていますね。  
これだけだちょっと分かりづらいという場合は実際に返されている値を変数に格納して画面に出力してみましょう。  

```groovy
def value = 1
def result = value ? "This is 1" : "This is not 1"

println result
assert result == "This is 1"
```

ちゃんとresult変数に値が格納されていますね。

三項演算子の書き方は、 *{条件式}* ? *{Trueの場合}* : *{Falseの場合}* となります。

## Groovy Truth
Groovyのifは空気をかなり読んでくれます。  
感覚として、空っぽだよね、という値をifの条件に指定すると、falseを返してくれます。  

```groovy
assert "empty" == ([] ? "full" : "empty")
assert "empty" == ([:] ? "full" : "empty")
assert "empty" == (0 ? "full" : "empty")
assert "empty" == ("" ? "full" : "empty")
```

上記に出ている `[]`はリスト、`[:]`はマップと呼ばれるものです。詳しくは別の章で述べます。  


## エルビス演算子を使う
Groovyにはエルビス演算子というものがあります。  
これは三項演算子をパワーアップさせたようなものです。  

まず以下の三項演算子をみてください。  

```groovy
def value = "aaa"
assert "aaa" == (value == "aaa"? "aaa" : "not equal")
```
valueの中身が"aaa"ならvalueを、そうでなければ"not equals"という文字列を返す三項演算子です。  
この、「もし変数がxxxという条件であればその変数の中身を返し、条件に合致しない場合はそれ以外の値を返す」というパターンはプログラミングをしていると頻出するパターンです。  
そこでGroovyはこのパターンを簡潔に表現するために **エルビス演算子** というものを用意しました。  

```groovy
def value = "aaa"
println "aaa" == value ?: "not found"
```

`?:`というキーワードが **エルビス演算子** です。  
エルビス演算子を使うと、 `a ? a : b`というような三項演算子を`a?:b`と書けるようになります。  
かなりソースがスッキリします。  


- ifは三項演算子を使えば、値を返せる。  
- エルビス演算子  
- GroovyTruth  