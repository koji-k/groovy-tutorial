# Groovyで外部ライブラリを利用する
Groovyに限らず、多くのプログラミング言語には標準には無い便利な機能が様々なライブラリという形でが用意、公開されていますね。  

さて、皆様すでにお気づきの通り、GroovyはJavaなので、Javaで利用できるライブラリ（jar）はGroovyからも利用できてしまいます！  
つまり、Javaでライブラリが公開されればされるほど、Groovyで使えるライブラリもどんどん増えていくわけです。  
すでにJavaには数え切れないほどのライブラリが存在します。  
それらの優秀なライブラリ群がそのままGroovyで利用できる、コレはGroovyの大きなメリットのうちの一つです。  

しかし、Groovyのメリットはコレだけにとどまりません。  
それを見て行きましょう。

例えば、Javaで外部ライブラリを利用するには、Mavenリポジトリがよく利用されますね。  
その名のとおり、MavenやGradleと言ったデファクトスタンダードなビルドツールから利用できます。  
ビルドツールの設定ファイルに、必要なライブラリの情報を記述しておけば自動的にダウンロード、パスの設定等を行ってくれます。  
非常に便利ですが、ちょっとした使い捨てプログラム、検証プログラムをササっと書きたいという場合に、わざわざビルドツールは大げさですね（本音：面倒くさい）。  
じゃあ手動でダウンロード？でもそうすると、他の人にコードを渡して、「これ、サンプルコードだからチェックしてみて。あ、でもライブラリにAとBとCを使っていてそれぞれ依存関係が。。。」と言ったことをわざわざ毎回説明しないといけません。  
そんな面倒臭いことしたくないですよね？  
Groovyはこの２つの問題を一気に解決します。  
それが`@Grab`です。  

この`@Grab`をコード内に記述することで、なんと **Groovyは外部ライブラリのダウンロード、パスの設定などを自動的に行ってくれます！**


## 実際に試してみよう
論より証拠。実際にコード見てみましょう。  
サンプルとして、このチュートリアルのトップページのHTMLからaタグを抜き出して、そのaタグを全てMarkdown形式に書き換えてみましょう。  
ちなみに、このようにHTMLを取得して必要な情報を抜き出すことをWEBスクレイピングと呼びます。  
JavaのWEBスクレイピング用のライブラリにはJsoupという有名な物が有ります。今回はコレを使ってみましょう。  
ちなみにJsoupのMaven用のリポジトリは[ココ](http://mvnrepository.com/artifact/org.jsoup/jsoup/1.8.3)。  

```groovy
@Grab(group='org.jsoup', module='jsoup', version='1.8.3')

import org.jsoup.Jsoup
import org.jsoup.nodes.Document
import org.jsoup.nodes.Element
import org.jsoup.select.Elements

Document document = Jsoup.connect("http://koji-k.github.io/groovy-tutorial/index.html").get()
document.select("a").collect {Element element ->
    "[${element.text()}](${element.attr("href")})"
}.each {
    println it
}
```

ハイこれだけ！コレだけで「トップページのHTMLからaタグを抜き出して、そのaタグを全てMarkdown形式に書き換え」ることが出来ました。  
GroovyとJsoupを使えばこんなに簡単にWEBスクレイピングが実現できます。  
そして本題の`@Grab`ですね。ファイルの先頭を見れば`@Grab(group='org.jsoup', module='jsoup', version='1.8.3')`という記述が有ります。  
そうです。これだけです。この記述を先頭にするだけで、Groovyがライブラリのダウンロードから設定まで行ってくれるのです！！  
そう、ちょっとしたサンプルコードなどであれば **ビルドツールは必要ありません。**  
そして **コード自体にライブラリの情報が有り、Groovyが自動でダウンロード、設定してくれるので、コードを誰かに渡す際にもわざわざ依存ライブラリの情報を伝える必要もありません！**  


## 複数のライブラリを扱う
単純に`@Grab`を複数並べるだけでもOKです。別の方法として`@Grapes`を利用する方法もあります。  
では、上記のサンプルを更に進めて、取得したデータをデータベースに格納して、データベースからその値を取り出して表示するようにしてみましょう。  
わざわざMySQLとかPostgreSQLをインストールするのは手間ですので、インメモリで動かせる上に、jarとして提供されているH2を利用します。  
**そう、jarとして提供されている、つまりMavenにライブラリが有るのでデータベースすらもインストールせずに、全てGroovy上で完結させることができるわけです！**

H2のMavenリポジトリは[ココ](http://mvnrepository.com/artifact/com.h2database/h2/1.4.190)。

```groovy
@Grapes([
    @Grab(group='org.jsoup', module='jsoup', version='1.8.3'),
    @Grab(group='com.h2database', module='h2', version='1.4.190'),
    @GrabConfig(systemClassLoader = true)
])

import org.jsoup.Jsoup
import org.jsoup.nodes.Document
import org.jsoup.nodes.Element
import groovy.sql.*

Document document = Jsoup.connect("http://koji-k.github.io/groovy-tutorial/index.html").get()
List<String> links = document.select("a").collect {Element element ->
    "[${element.text()}](${element.attr("href")})"
}

// DBに接続（インメモリのh2）
def connection = Sql.newInstance("jdbc:h2:mem:", "org.h2.Driver")
// テーブルを作成
connection.execute 'CREATE table links(link clob)'

// INSERT
links.each {String link ->
    connection.execute("INSERT INTO links(link) VALUES(?)", [link])
}

// データをDBから取得して表示（H2を使っているので、TEXTがCLOB型になってちょっと特種）
connection.rows("SELECT link from links").each {
    org.h2.jdbc.JdbcClob link = it[0]
    println link.getSubString(1,link.length() as Integer)
}
```

*インメモリで動かしているので実行するたびにデータは削除されます。あとサンプルのため例外処理などは省いています。*

いかがでしょう？単純に`@Grab`の部分の記述が増えて、なんの設定もせずにいきなりSQL文が実行できていますね。  
実際に実行してみてください。本当に何もせずに外部ライブラリやデータベースに至るまで、全て自動でセットアップされていることが分かるはずです。  
**コレがJavaのエコシステムに直接リーチし、さらに言語自体がライブラリの依存管理をできるGroovyの力です！**

1点注意点としては、`@Grab`でJDBCライブラリを利用する際は、データベースへの接続をする場合、`@GrabConfig(systemClassLoader = true)`という記述が必要、という点があります。  

## ちょっと補足
`@Grab`に記述する内容はどこで確認すればいいの？という点を補足します。  
Mavenのリポジトリ、例えば今回使った[H2データベース](http://mvnrepository.com/artifact/com.h2database/h2/1.4.190)を見てみましょう。  
その画面に、各ビルドツールに記載する内容のサンプルをすでに書いてくれています。  
**Grape** というタブの部分がGroovy用の`@Grab`、もしくは`@Grape`の為の内容になります。  
その中には、

```groovy
@Grapes(
	@Grab(group='com.h2database', module='h2', version='1.4.190')
)
```
と記載されているはずです。  
単純にこのテキストを利用することも出来ますし、`@Grab`の部分のみを抜き出して利用することも出来ます。  
お好みでどうぞ。



##まとめ
いかがでしたでしょうか。Groovy自体が備える素晴らしい機能の一端を垣間見ることが出来たのではないでしょうか。  
