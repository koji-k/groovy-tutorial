#Apache Groovy チュートリアル

```groovy
println "Hello Apache Groovy"
```

このサイトは[プログラミング言語 Apache Groovy](http://www.groovy-lang.org/)のチュートリアルです。  
対象読者はこれからGroovyに入門しようとされている方で、JavaやPHP、Rubyなどの他のプログラミング言語を使った簡単なプログラミングの経験がある方です。  
すでにGroovyの存在を知っている方は「あれ？Apacheってどういうこと？別の言語？」と思われるかもしれません。  
いいえ、ApacheGroovyはあなたの知っているGroovyです。  
Groovyは2015年11月より、 **Apacheソフトウェアファウンデーションのトップレベルプロジェクト** となりました！

このサイトはマークダウン形式のドキュメントを簡単に作成できるドキュメンテーションツール[Gaiden](https://github.com/kobo/gaiden)を利用して作成しています。  
[Github](https://github.com/kobo/gaiden)  
[Document](http://kobo.github.io/gaiden/)  
このGaiden自体もGroovyによって作成されたツールです。

このチュートリアルは[@saba1024](https://twitter.com/saba1024)が個人的に作成しているものです。  
問題等ございましたら優しくお知らせください。。。
なお、本チュートリアルはGroovy2.4.5で確認しています。  

また、以下のチュートリアルも作成、公開中です。  
[Groovy用のフルスタックフレームワークGrails](http://koji-k.github.io/grails-tutorial/)  
[Groovy用のノンブロッキングなフレームワークRatpack](http://koji-k.github.io/ratpack-tutorial/)  
[SpringBootとGroovyで超高速WEBアプリケーション開発](https://koji-k.github.io/spring-boot-groovy-tutorial//)  


## さぁ始めよう！...の前に
このチュートリアルは、左のメニューの上から順番に見ていっていただければ、一番シンプルにGroovyが理解できるように構成しています。  
[Install](startup/install.html)のページで、Groovyをインストールする方法を書いていますが、インストールしたくない、とりあえずパパっと試してみたいだけ、という方は、ブラウザからGroovyを実行できるサイトが有りますのでそちらを利用してみては如何でしょうか？  
[Groovy Web Console](https://groovyconsole.appspot.com/)  
ただ、Groovyのインストールはとても簡単なので、インストールして本チュートリアルをご利用いただくのがお勧めです ;)

```
2016/09/27 Nullセーフを追加
           クロージャに関数合成に関する記述を追加
2016/09/26 公開方法をgh-pagesブランチからmasterブランチのdocsに変更。
           コードの表示を少し綺麗に変更。
           Gaidenのバージョンを1.1にアップグレード
           他チュートリアルへのリンクを追加
2016/06/10 誤字脱字修正、サンプルリンク集追加
```