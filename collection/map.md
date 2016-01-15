# マップ
マップとは、PHPでいう連想配列（のようなもの）、JavaのHashtableなどに該当するものです。  
大雑把に言うと、リストの中にある各値に専用の名前をつけてあげることが出来ます。

- PHPの場合

```php
map = array(
    "foo" => "bar",
    "bar" => "foo",
);

// 5.4 以降
$map = [
      "foo" => "bar",
      "bar" => "foo",
];
```

- Javaの場合

```java
import java.util.*
Hashtable hashtable = new Hashtable()
hashtable.put("foo", "bar")
hashtable.put("bar", "foo")
```

ではGroovyの場合を見て行きましょう。

```groovy
Map map = [foo:"bar", bar:"foo"]
```

JavaよりはPHPの連想配列の作り方に似ていますね。  
上記の最も基本的なMapの宣言方法を見て分かるとおり、リストの中に`キー:値`という形式で値を記述します。  
実際に使ってみましょう。

```groovy
Map map = [foo:"bar", bar:"foo"]

// 値の取り出し
assert map.foo == "bar"
assert map.get("foo") == "bar"
assert map["foo"] == "bar"
assert map.bar == "foo"

// 値の上書き
map["bar"] = "foo_updated"
assert map.bar == "foo_updated"
map.put("bar", "foot_updated2")
assert map.bar ==  "foot_updated2"

// 値の追加
map["hoge"] = "hoge_value"
assert map.size() == 3
assert map.hoge == "hoge_value"

map.put("piyo", "piyo_value")
assert map.size() == 4
assert map.piyo == "piyo_value"
```

とてもシンプルですね。  
値の取り出し方や、セットの仕方は複数あります。目的にあった方法を選択しましょう。  
例えば、変数に入っている値をキーにしてMapから値を取得する場合、以下のような方法が有ります。  

```groovy
Map map = [foo:"bar", bar:"foo"]
String key = "bar"

assert map[key] == "foo"
```

## 繰り返し
マップにもリストのように、便利な繰り返しメソッドが備わっています。  
いつもの`each`メソッドを試してみましょう。  

```groovy
Map map = [foo:"bar", bar:"foo"]
map.each {
   println it
}
```
実行結果として

```
foo=bar
bar=foo
```
と表示されましたね？
でもコレだとキーと値を別個に取得できません。  
そこで、Mapの場合は`key`と`value`というプロパティが存在します。


```groovy
Map map = [foo:"bar", bar:"foo"]
map.each {
    println "key = ${it.key}, value = ${it.value}"
}
```
ちゃんとキーとプロパティを別個に取得出来ましたね。



## リストとマップを組みあせる
それでは少し応用を効かせてリストとマップを組み合わせてみましょう。  
内容は、Mapでユーザの情報を持ち、その情報をそれぞれListにて保持するというものです。

```groovy
List userList = [
    ["name": "aaa", age:20],
    ["name": "bbb", age:25],
    ["name": "ccc", age:30],
    ["name": "ddd", age:35]
]
```

それでは、全てのユーザの名前を表示してみましょう。
  
```groovy
userList.each {Map user ->
    println user.name
}
```

ユーザ名が一覧で表示されましたか？  

それでは今度は逆に、マップの中にリストを突っ込んでみましょう！  
例えば各ユーザが複数の得意なプログラミング言語を持っている場合を想定しましょう。

```groovy
List userList = [
    ["name": "aaa", age:20, languages:["Java"]],
    ["name": "bbb", age:25, languages:["Java", "PHP"]],
    ["name": "ccc", age:30, languages:[]],
    ["name": "ddd", age:35, languages:["Groovy"]]
]

userList.each {Map user ->
    println "userName:${user.name}"
    user.languages.each {
        println it
    }
    println "" // 見やすくするために改行
}
```
複雑そうに見えますが、基本的にリストやマップで用意されている`each`などのメソッドを使って、素直に処理を書いていくだけです。  
Mapはそれ単体でももちろん利用しますが、リストと組み合わせて利用されることが多いので、色々試してみましょう。