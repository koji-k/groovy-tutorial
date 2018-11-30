# クロージャ
さて、ココではクロージャ（Closureと書きます）を見ていきます。  
今までの章でも時々この単語が出てきましたね？  
Groovyを使う上でこのクロージャという概念は非常に重要なものです。  
クロージャ自体は様々なプログラミング言語に用意されている一般的な機能です。  
しかし、クロージャとはこうだ、と簡単に表現できないものになっています。  


## クロージャを使ってみる
まずは自分でクロージャを作ってみましょう。  

```groovy
// クロージャを定義して、cljという変数に格納する
Closure clj = {
    println "Hello Closure!"
}

// 引数ありバージョン
Closure clj2 = {String name ->
    println "Hello ${name}!"
}

// cljに格納されているクロージャを実行
clj()
clj2("koji")
```

...別にどうってことないですね？  
もしかしたら関数じゃないの？と思われた方も居るかもしれません。  
ずばりそれは間違っていません。この例は明らかに無名関数と呼ばれるものです。  
無名関数なので、別に変数に格納する必要もありません。  

```groovy
{String name ->
    println "Hello ${name}!"
}
```
このクロージャは宣言だけされて何もしません。  
このことから分かるように、クロージャは`{何か処理}`、もしくは`{引数 -> 何か処理}`と書くことがわかりますね。  

この形、どこかで見たことありませんか？  
前の章で扱ったリストのメソッドたち、それらのメソッドに渡すのがこのクロージャなのです！  

例えばリストの中身を全部表示する`each`メソッドを見てみましょう。  

```groovy
[1,2,3].each {
    println it
}
```

eachの後にある`{ println it}`がクロージャです。  
なお、クロージャの中では`it`という暗黙の変数があって、渡される引数が一つの場合、自動的にこの`it`という変数に格納されます。  

Groovyは、解釈に問題がない場合、引数を渡す際に`()`という記述を省略できます。  
つまり、上記の例は
  
```groovy
[1,2,3].each({
    println it
})
```

と全く同じになります。    
コチラの方が引数としてクロージャを渡しているのが明確ですね。

さて、これでリストのeachなどのメソッドに渡していたものがクロージャと呼ばれるものだとわかりました。  
そして、一番最初の例のように、クロージャは宣言して変数に格納できます。  

つまり以下のようなコードが書けるということです。

```groovy
List list = [1,2,3]

// クロージャを宣言して、変数doubledに格納
Closure doubled = {
    it * 2
}

assert [2,4,6] == list.collect(doubled)
```

変数`doubled`は、引数を一つ受け取り、それを2倍して返す、というクロージャを格納しています。  
そして、listの`collect`メソッドの引数にそのクロージャ`doubled`を渡してあげています。  
すると`collect`は自動的にlistの各要素を順番にクロージャ`doubled`に渡して、そして実行してくれています。  
つまり、以下のように書いたものと全く同じです。  

```groovy
assert [2,4,6] == list.collect {
    it * 2
}
```
このように、Groovyではリストに用意されているメソッドに対してクロージャを渡して処理をしていきます。  
このことから、Groovyにとって如何にクロージャが重要な機能なのかがわかります。  

あれ？ `doubled`を渡す時って`doubled()`じゃないの？と思われた方も居るかもしれません。  
doubledはあくまでクロージャを格納した **変数** です。そのため、`each(doubled)`としてあげることで、`each`メソッドに`doubled変数の中に格納されたクロージャの本体`を渡してあげています。  
ここで`each(doubled())`とすると、コレは`each`というメソッドに`doubled変数の中に格納されたクロージャの実行結果`を渡すということになってしまいます。  
ちなみに、その点を考慮すると単純に

```groovy
List list = [1,2,3]

// クロージャを宣言して、変数doubledに格納
Closure doubled = {
    it * 2
}

assert [2,4,6] == list.collect(doubled)
assert 10 == doubled(5) // お手軽な関数としても利用できる
```
という使い方も当然出来ます。  
クロージャは何もリスト専用ではなく、様々な場面で活躍します。


## さらに詳しく
さて、今までの例だと実はクロージャではなくてタダの無名関数です。  
ではクロージャとは何なのか？ということなのですが、コレは少し難しい概念です。  
クロージャの正確な定義などはググって貰えれば出てきますが、ココではクロージャを **生まれ故郷（クロージャ自身が定義された場所）を忘れない無名関数** と定義します。  
生まれ故郷を忘れないとはどういうことでしょう？  
論より証拠。まずはサンプルコードを見てみましょう。 

```groovy
class Test {
    static getClosure() {
        Integer defaultCount = 0
        return {
            defaultCount++
            defaultCount
        }
    }
}

Closure clj = Test.getClosure()
assert 1 == clj()
assert 2 == clj()
assert 3 == clj()

Closure clj2 = Test.getClosure()
assert 1 == clj2()
assert 2 == clj2()

assert 4 == clj() // cljに格納されているdefaultCountとclj2に格納されているdefaultCountは別物！
```

順番に見て行きましょう。  

まず、Testクラスの以下の部分で、クロージャを生成して`return`でその生成したクロージャを返しています。

```groovy
static getClosure() {
    Integer defaultCount = 0
    return {
        defaultCount++
        defaultCount
    }
}
```

ここで注目するのが、returnされる **クロージャの中で、クロージャの外側に宣言されている`defaultCount`という変数が使われているという点です。**  
実はいきなり答えなのですが、ここがクロージャがクロージャの条件を満たす部分です。  
このクロージャはreturnされて別の変数（上記の例だと`clj`と`clj2`）に格納されても、 **決して自分の生まれ故郷から見えていた景色（変数 `defaultCount`）の存在を忘れません。**  
returnされているのは`{...}`に該当するクロージャ部分だけなのに、です。  
上記の例のように`clj()`や`clj2()`を実行すると、その都度クロージャの生まれ故郷である`getClosure`の中にある`defaultCount`を利用しているのです。  
なので、`clj()`や`clj2()`を実行すると毎回異なる値が返されているわけですね。  
この **生まれ故郷を忘れない** 関数がクロージャと呼ばれるものです。

## 関数合成
Groovyのクロージャでは、簡単に関数合成が行えます。  
例えば、データベースなどから数字を取得して、数値に変換して、2倍する。という処理を書いてみましょう。  

```groovy
Closure toInt = {String v ->
    v.toInteger()
}
Closure twice = {Integer v ->
    v * 2
}

// サンプル用に分かりやすく
Closure getDataFromDatabase = {
    "5"
}
assert 10 == twice(toInt(getDataFromDatabase()))
```

クロージャ自体は全く難しくありません。  
ただ、クロージャを呼び出している`twice(toInt(getDataFromDatabase()))`の部分がビックリするほど読み辛いですね。  
では、ココで関数合成を行って呼び出しをシンプルにしてみましょう！

```groovy
Closure calc1 = getDataFromDatabase >> toInt >> twice
Closure calc2 = twice << toInt << getDataFromDatabase
assert 10 == calc1()
assert 10 == calc2()
```

これだけです！  
Groovyでは、クロージャ同士を`>>`もしくは`<<`で順番に結合した新しいクロージャを生成できます。それぞれのクロージャの実行結果が次のクロージャにそのまま渡されます。  
これがGroovyの関数合成です。  

なお、関数合成時には引数を渡すことは出来ませんが、合成した関数を実行する際に引数を渡すことが出来ます。  

```groovy
// 抜粋
Closure getDataFromDatabase = {String a ->
    "5" + a
}
Closure calc1 = getDataFromDatabase >> toInt >> twice

assert 102 == calc1("1")
```

また、関数合成の結果は新しい関数（クロージャ）が返されるわけなので、合成結果を変数に格納せずとも以下のように実行することも出来ます。  

```groovy
assert 102 == (getDataFromDatabase >> toInt >> twice).call("1")
assert 102 == (getDataFromDatabase >> toInt >> twice)("1")
```

## 特殊変数this、owner、delegate
さて、クロージャには`this`、`owner`、`delegate`という暗黙的に利用できる特殊な変数が存在ます。  
詳細は[[Groovy]クロージャのthis、owner、delegateについて](http://qiita.com/saba1024/items/b57c412961e1a2779881)を参照してください。

##まとめ
クロージャについては、あまり意識せずにとりあえずlistの`each`や`collect`などの各メソッドに渡すもの、という感覚でドンドン使っていけば感覚がつかめてくると思います。  