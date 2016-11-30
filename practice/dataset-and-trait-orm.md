# DataSetとTraitを使って簡単なORM

**これはG* Advent Calendar 2015の1日目の記事です。**

## 概要
データベースのテーブルと、プログラム上の値クラスのインスタンスをそれぞれ簡単にマッピングできる便利機能をTraitを利用して作成しました。  
お手軽さをさらに感じるために、DB自体のデータ操作はGroovyのDataSetクラスを利用して行います。  
ちょっと長くなるので、一番最後に全体ソースを置いておきますので、それを見て頂いてもいいともいます。  
実際にsample.groovyというような名前で保存すればgroovyコマンドで実行できます。  


## DataSetのメモ

さて、Groovyにはデータベースアクセスを非常に簡単に行えるよう`groovy.sql.DataSet`というものがあります。  
この`DataSet`を使うと、テーブル名を指定するだけで自動的にSQLを生成してくれます。  
以下のような感じで利用できます。  

```groovy
def sql = Sql.newInstance("jdbc:h2:mem:", "org.h2.Driver")

// 勝手にSQL文を作ってくれる
DataSet dataSet = sql.dataSet('test')
assert dataSet.sql == 'select * from test'

// 条件指定もなんのその（自動的にpreparedStatement）
DataSet dataSet2 = dataSet.findAll{it.age > 18}
assert dataSet2.sql == 'select * from test where age > ?'
assert dataSet2.parameters == [18]

// dataSetのメソッドは自分を返すのでチェイン出来る。
assert sql.dataSet('test').findAll({it.age > 18}).sql == 'select * from test where age > ?'
```

この時点では実際のデータベースなどは参照しておらず、`Sql#dataSet()`に渡している文字列をテーブル名、クロージャに指定するプロパティ名をカラム名として、SQL文を生成しています。  

実際に簡単なデータの登録とデータの取得のサンプルを見てみましょう。  

```groovy
@GrabConfig(systemClassLoader = true)
@Grab(group='com.h2database', module='h2', version='1.4.190')
import groovy.sql.*

// テスト用のDBを作成
def sql = Sql.newInstance("jdbc:h2:mem:", "org.h2.Driver")
sql.execute("""
CREATE TABLE test(
    id BIGINT,
    name VARCHAR(255),
    age INT,
    married BOOLEAN,
    dummy_id BIGINT
)""")

DataSet dataSet = sql.dataSet('test')

// addを使えば、引数に渡した情報で自動的にテーブルにINSERTしてくれる！
dataSet.add(id:1, name:'a', age:31, married:true)

// dataSetの中身は何にも変わっていないので、firstRowと使ってSQLを実行。
// 返ってくるのはaGroovyRowResultなのでMapとして振る舞える！
assert dataSet.firstRow()['id'] == 1 
assert dataSet.firstRow()['name'] == 'a' 
assert dataSet.firstRow()['age'] == 31
assert dataSet.firstRow()['married'] == true
assert dataSet.firstRow()['dummy_id'] == null
assert dataSet.firstRow() instanceof GroovyRowResult
assert dataSet.firstRow() instanceof Map 
```

ORMを使うほどではないけど、SQLを書くのも面倒くさい。。。という場合に非常に重宝しそうな感じがしますね！  
なお、上記のコードを`test.groovy`というファイルに保存して、`groovy test.groovy`として実行すれば、自動的にH2というインメモリで動かせるデータベースライブラリのダウンロードと設定ができるので、態々テスト用にDBをセットアップする必要はありません！  


１点注意点として、DataSetを利用する場合は、GroovyConsoleだと  

```console
groovy.lang.GroovyRuntimeException: DataSet unable to evaluate expression. AST not available for closure: ConsoleScript135$_run_closure2. Is the source code on the classpath?
	at ConsoleScript135.run(ConsoleScript135:106)
```

というようなエラーが出て駄目っぽいので普通にGroovyファイルを作成して、`groovy`で実行する。




## 本題
さて、DataSetの実行結果をMapとして扱えることは分かりました。  
ただやはりMapでデータをやり取りするのはちょっと気持ちが悪いですね。  
IDEでの補完も効かないのでタイプミスも発生しそうだしクラスにマッピングされていないのでメソッドなども利用できません。  
そこでやっと本題！Groovy2.3から利用できるようになったtraitを利用して、テーブルと一致する値クラスと、このDataSetを上手いこと組み合わせる機能を作成します！  

前提条件として、テーブルは以下の構成のものを利用します。  

```sql
CREATE TABLE test(
    id BIGINT,
    name VARCHAR(255),
    age INT,
    married BOOLEAN,
    birthday TIMESTAMP default now(),
    dummy_id BIGINT
)
```

そして、いわゆるORMであればこのテーブルのレコードの値がマッピングされるクラスは以下のようになります。

```groovy
class Test {
    Long id
    String name
    Integer age
    Boolean married
    Date birthday
    Long dummyId
}
```

このTestのような値を保持するクラスにDataSetと上手いこと連携できるようにtraitで魔法をかけていきます。


## traitと値を保持するクラスの作成

ということで早速コードです。

```groovy
trait DataSetCombiner {

    private static List ignoreClassTypes = [java.lang.Class]

    private static Closure availableProperties = { MetaProperty mp -> 
        !( mp.type in ignoreClassTypes)
    }

    private static Closure toSnakeCase = { String name ->
        name.replaceAll(/([A-Z])/, /_$1/).toLowerCase()
    }

    /**
     * 渡されたMapのデータを使って、このDataSetCombinerをimplementsしているクラスのインスタンスを生成して返す
     * @param dataSet 生成する値クラスのインスタンスのプロパティにセットしたい値をもつMap
     * @param dateFormat Mapの中身にStringとして保持されているjava.sql.Timestamp、java.util.DateをDateに作りなおすためのフォーマット
     * @return dataSetで指定した値で初期化した値クラスのインスタンス
     */
    static generate(Map dataSet, String dateFormat = 'yyyy/MM/dd HH:mm:ss') {

        // 値クラスの全てのプロパティを取得
        List<MetaProperty> properties = this.metaClass.getProperties()
        def clazz = this.metaClass.invokeConstructor()

        // Mapから、各プロパティ名に合致する値を取り出して、値クラスのインスタンスにセット
        properties.findAll(availableProperties).each {MetaProperty mp ->
            String propertyName = mp.name
            String keyNameOnMap = toSnakeCase.call(mp.name)
            switch(mp.type) {
                case String:
                    clazz[propertyName] = dataSet[keyNameOnMap]
                    break
                case Number: // Integer and Long are inherit Number
                    clazz[propertyName] = dataSet[keyNameOnMap] as Long
                    break
                case Boolean:
                    clazz[propertyName] = dataSet[keyNameOnMap] as Boolean
                    break
                case Date:
                    clazz[propertyName] = Date.parse(dateFormat, dataSet[keyNameOnMap] as String)
                    break
                default:
                    clazz[propertyName] = null // どうしよう？
            }
        }
        clazz
    }

    /**
     * 値クラスのもつプロパティ名と値を、それぞれKeyとValueにしてMapに詰めて返す。
     * その際にKey名はプロパティをスネークケースに変換したもの。
     */
    def convertToMap() {
        this.metaClass.properties.findAll(availableProperties).collect {MetaProperty mp ->
            String keyNameOnMap = toSnakeCase.call(mp.name)
            [(keyNameOnMap): this[mp.name]]
        }.collectEntries{it}
    }
}

/**
 * 値を保持するクラスの宣言。用意したtraitをimplementsで指定するだけでOK
 */
class Test implements DataSetCombiner {
    Long id
    String name
    Integer age
    Boolean married
    Date birthday

    Long dummyId
}
```

これだけです！  

traitによって公開されているメソッドは、  
クラスメソッドの`generate`と、インスタンスメソッドの`convertToMap`の2つだけです。  

`generate`メソッドには、値クラスに突っ込みたい値を保持したMapを渡してあげると、ちゃんと対応するプロパティに応じた方に変換して値をセットするようにしています。  
この際に、Mapの値をベースにプロパティ名を探すと存在しないプロパティにアクサス出来てしまうので、それを回避するために存在するプロパティをeachで回して、各プロパティに対応するMapの値を取得するようにしています。  

`convertToMap`の方は単純にインスタンスが持っているプロパティをMapに詰めて返すだけです。  
その際に、プロパティー名をスネークケースに変換して返すことによって、データベースのカラム名と一致するようにしてあげています。  


## 実際に使ってみる
上記のコードを見るだけだとよくわからんので実際に利用するコードを見てみましょう。  

まず、DataSet関係なしに、インスタンスの生成が少し楽になっています。

```groovy
// 全ての値がStringなMapを渡せば、自動的にクラスのプロパティに定義されている型を元に自動的に変換。（手動）
// groovy.transform.Immutableでアノテートされたクラスでも同じようなことができるけど、型は正しいものを渡さなければならない。
Test test = Test.generate([id:'1', name:'koji', age: '31', married:'true', birthday:'2016/12/01 00:00:00', dummy_id:'123'])
assert test instanceof Test
assert test.id.class == java.lang.Long
assert test.name.class == java.lang.String
assert test.age.class == java.lang.Integer
assert test.married.class == java.lang.Boolean
assert test.birthday.class == java.util.Date
assert test.dummyId.class == java.lang.Long

assert test.id == 1
assert test.name == 'koji'
assert test.age == 31
assert test.married == true 
assert test.birthday == Date.parse('yyyy/MM/dd', '2016/12/01')
assert test.dummyId == 123
```
ただこれだけだとそれほど旨味がないですね。  

そこで実際にDataSetと絡めて使ってみます！  

```groovy
def dataSet = sql.dataSet('test')
// Mapを渡してデータベースにINSERT
dataSet.add(id:1, name:'a', age:10, married:true, dummy_id:100)
dataSet.add(id:2, name:'b', age:20, married:false, dummy_id: 200 )

// 自作オブジェクトからDBへデータを保存！
dataSet.add(test.convertToMap())

println "All records:"
dataSet.rows().each{println it}
```

実行すると結果は  

```groovy
All records:
[ID:1, NAME:a, AGE:10, MARRIED:true, BIRTHDAY:2016-11-30 15:13:03.551, DUMMY_ID:100]
[ID:2, NAME:b, AGE:20, MARRIED:false, BIRTHDAY:2016-11-30 15:13:03.553, DUMMY_ID:200]
[ID:1, NAME:koji, AGE:31, MARRIED:true, BIRTHDAY:2016-12-01 00:00:00.0, DUMMY_ID:123]
```

できました！ **SQLを書かずに値クラスをデータベースに保存できました！**  
ここでtraitに記述した`convertToMap`が生きてきています。  

さらに今度は、データベースから取得したレコードを値クラスに突っ込んでそのインスタンスを取得してみます。  

```groovy
def dataSet2 = dataSet.findAll{it.name == 'a'}
Test test2 = Test.generate(dataSet2.firstRow(), 'yyyy-MM-dd HH:mm:ss.SSS') // DataSetの結果をそのまま使ってインスタンスを生成できる!
assert test2.name == 'a'
assert test2.age == 10
assert test2.married == true
assert test2.birthday != null
assert test2.dummyId == 100
```

できました！データベースから値を取得して、traitで作成した`generate`メソッドにその **結果を渡すだけでちゃんと値がセットされたインスタンスが生成されています！**  

## まとめ

当然他にも色々想定していない型とか渡されたらどうするの？など問題はありますが、DataSetとtraitを組み合わせることで、ある程度スッキリデータベース周りを記述できるのかな？と思います。  




# 参考

[Traits](http://docs.groovy-lang.org/next/html/documentation/core-traits.html)  
[Class DataSet](http://docs.groovy-lang.org/latest/html/api/groovy/sql/DataSet.html)  
[groovy databases](http://www.slideshare.net/paulk_asert/groovy-databases)  
[Groovyでデータベース操作（GroovySQL）](http://npnl.hatenablog.jp/entry/20090505/1241504105)  

# 全体ソース

```groovy
@GrabConfig(systemClassLoader = true)
@Grab(group='com.h2database', module='h2', version='1.4.190')
import groovy.sql.*

// テスト用のDBを作成
def sql = Sql.newInstance("jdbc:h2:mem:", "org.h2.Driver")
sql.execute("""
CREATE TABLE test(
    id BIGINT,
    name VARCHAR(255),
    age INT,
    married BOOLEAN,
    birthday TIMESTAMP default now(),
    dummy_id BIGINT
)""")

trait DataSetCombiner {

    private static List ignoreClassTypes = [java.lang.Class]

    private static Closure availableProperties = { MetaProperty mp -> !( mp.type in ignoreClassTypes) }

    private static Closure toSnakeCase = { String name ->
        name.replaceAll(/([A-Z])/, /_$1/).toLowerCase()
    }

    /**
     * 渡されたMapのデータを使って、このDataSetCombinerをimplementsしているクラスのインスタンスを生成して返す
     * @param dataSet 生成する値クラスのインスタンスのプロパティにセットしたい値をもつMap
     * @param dateFormat Mapの中身にStringとして保持されているjava.sql.Timestamp、java.util.DateをDateに作りなおすためのフォーマット
     * @return dataSetで指定した値で初期化した値クラスのインスタンス
     */
    static generate(Map dataSet, String dateFormat = 'yyyy/MM/dd HH:mm:ss') {

        // 値クラスの全てのプロパティを取得
        List<MetaProperty> properties = this.metaClass.getProperties()
        def clazz = this.metaClass.invokeConstructor()

        // Mapから、各プロパティ名に合致する値を取り出して、値クラスのインスタンスにセット
        properties.findAll(availableProperties).each {MetaProperty mp ->
            String propertyName = mp.name
            String keyNameOnMap = toSnakeCase.call(mp.name)
            switch(mp.type) {
                case String:
                    clazz[propertyName] = dataSet[keyNameOnMap]
                    break
                case Number: // Integer and Long are inherit Number
                    clazz[propertyName] = dataSet[keyNameOnMap] as Long
                    break
                case Boolean:
                    clazz[propertyName] = dataSet[keyNameOnMap] as Boolean
                    break
                case Date:
                    clazz[propertyName] = Date.parse(dateFormat, dataSet[keyNameOnMap] as String)
                    break
                default:
                    clazz[propertyName] = null // どうしよう？
            }
        }
        clazz
    }

    /**
     * 値クラスのもつプロパティ名と値を、それぞれKeyとValueにしてMapに詰めて返す。
     * その際にKey名はプロパティをスネークケースに変換したもの。
     */
    def convertToMap() {
        this.metaClass.properties.findAll(availableProperties).collect {MetaProperty mp ->
            String keyNameOnMap = toSnakeCase.call(mp.name)
            [(keyNameOnMap): this[mp.name]]
        }.collectEntries{it}
    }
}

/**
 * 値を保持するクラスの宣言。用意したtraitをimplementsで指定するだけでOK
 */
class Test implements DataSetCombiner {
    Long id
    String name
    Integer age
    Boolean married
    Date birthday

    Long dummyId
}


// 全ての値がStringなMapを渡せば、自動的にクラスのプロパティに定義されている型を元に自動的に変換。（手動）
// groovy.transform.Immutableでアノテートされたクラスでも同じようなことができるけど、型は正しいものを渡さなければならない。
Test test = Test.generate([id:'1', name:'koji', age: '31', married:'true', birthday:'2016/12/01 00:00:00', dummy_id:'123'])
assert test instanceof Test
assert test.id.class == java.lang.Long
assert test.name.class == java.lang.String
assert test.age.class == java.lang.Integer
assert test.married.class == java.lang.Boolean
assert test.birthday.class == java.util.Date
assert test.dummyId.class == java.lang.Long

assert test.id == 1
assert test.name == 'koji'
assert test.age == 31
assert test.married == true
assert test.birthday == Date.parse('yyyy/MM/dd', '2016/12/01')
assert test.dummyId == 123


def dataSet = sql.dataSet('test')
// Mapを渡してデータベースにINSERT
dataSet.add(id:1, name:'a', age:10, married:true, dummy_id:100)
dataSet.add(id:2, name:'b', age:20, married:false, dummy_id: 200 )

// 自作オブジェクトからDBへデータを保存！
dataSet.add(test.convertToMap())

println "All records:"
dataSet.rows().each{println it}

def dataSet2 = dataSet.findAll{it.name == 'a'}
Test test2 = Test.generate(dataSet2.firstRow(), 'yyyy-MM-dd HH:mm:ss.SSS') // DataSetの結果をそのまま使ってインスタンスを生成できる!
assert test2.name == 'a'
assert test2.age == 10
assert test2.married == true
assert test2.birthday != null
assert test2.dummyId == 100
```