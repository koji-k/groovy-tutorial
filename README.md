# Apache Groovy チュートリアル

Groovy製のドキュメンテーションツール[Gaiden](https://github.com/kobo/gaiden)を使って作成されたGroovyのチュートリアルです。  
生成されたチュートリアルは[コチラで確認できます。](http://koji-k.github.io/groovy-tutorial/)

# ビルド方法
このリポジトリをcloneして、ローカルで編集することが出来ます。
  
## 準備
そのためには、Gaiden1.1以降が必要となります。  
Gaideを利用する方法は以下の2パターンがあります。

### gaidenwの利用

Gaidenをインストールしなくても、このリポジトリをcloneした後にリポジトリ内で`./gaidenw build`を実行するればビルド可能です。  
自動的にGaiden1.1がダウンロードされ、gaidenwからそのGaidenが利用されるようになります。  
なお、GaidenはGroovyを利用しますので、事前にGroovyをインストールしておいてください。

### gaidenをインストールして利用
SDKMANなどのツールを利用して、Gaidenが動作する環境を用意して下さい。  
SDKMAMの場合は、

```
sdk install gaiden
```
だけでOKです。  
なお、Gaidenを動作させるためにはJavaとGroovyも必要となります。  
もしこの2つがインストールされていない場合は、共にSDKMANなどでインストールしておいて下さい。

最終的に、`gaiden -v`をコマンドラインから実行して、以下のような情報が出力されれば準備完了です。


```terminal
[koji:sample]$ gaiden -v
             _     _
            (_)   | |
  __ _  __ _ _  __| | ___ _ __
 / _` |/ _` | |/ _` |/ _ \ '_ \
| (_| | (_| | | (_| |  __/ | | |
 \__, |\__,_|_|\__,_|\___|_| |_|
  __/ |
 |___/

Version    : 1.1
Revision   : 91ef3820725d47a625883393caa616f26368adfd
Build date : 2016-06-06 11:21:37.196+0900

OS         : Linux 3.13.0-24-generic
JVM        : 1.8.0_111
Groovy     : 2.3.6
```

## リポジトリのクローン
適当なディレクトリで、`git clone git@github.com:koji-k/groovy-tutorial.git`を実行して下さい。  
すると、`groovy-tutorial`というディレクトリ名で、リポジトリがクローンされます。

```terminal
[koji:sample]$ git clone git@github.com:koji-k/groovy-tutorial.git
Cloning into 'groovy-tutorial'...
remote: Counting objects: 470, done.
remote: Total 470 (delta 0), reused 0 (delta 0), pack-reused 470
Receiving objects: 100% (470/470), 832.64 KiB | 123.00 KiB/s, done.
Resolving deltas: 100% (235/235), done.
Checking connectivity... done.
[koji:sample]$ ls
groovy-tutorial
```
これで、Markdownファイルを修正することが出来ます。
もしMarkdownファイルを新規作成した場合は、そのファイル名を`pages.groovy`に追記して下さい。  
`gaiden build`(gaidenをインストールしていないのであれば`gaidenw build`)を実行すると、Markdownが全てHTMLにコンバートされて、`docs/`ディレクトリ配下に出力されます。  
docs配下にあるHTMLファイルをブラウザで開くことで、最終的な結果を確認することが出来ます。

*注意：docsディレクトリにはHTMLが保存されますので、docsディレクトリにはMarkdownを保存しないようにして下さい。*

なお、`gaiden watch`を実行しておけば、ファイルを追加したり編集したタイミングで自動的に`gaiden build`が実行されるようになるのでお勧めです。  
Gaiden自体の詳細な使い方は[Gaideの公式サイト](https://github.com/kobo/gaiden)を参照して下さい。
