# Actor


```groovy
import static groovyx.gpars.actor.Actors.actor

def server = actor {
    loop {
        react { message -> 
            switch(message) {
                case String:
                    println "Got: ${message}"
                    reply message.reverse()
                    break
                default:
                    println "Stop Actor"
                    stop()
            }
        }
    }
}

def client = actor {
    sleep(1000)
    server.send("ABCDEFG")
    sleep(1000)
    react {
        println "Returned: ${it}"
        server.send false

    }
}

println "Start"
[server, client]*.join()
println "End"
```

上記のコードを実行すると、

```terminal
Start
Got: ABCDEFG
Returned: GFEDCBA
Stop Actor
End
```

という結果にります。  
もしもActorインスタンスで`join()`を使っていない場合は、メインスレッドの処理はActorの処理の終了を待たずに先に進む事になります。  
つまり、上記のActorの宣言部分のコードはそのままに、実行部分を

```groovy
println "Start"
//[server, client]*.join() コメントアウト。
println "End"
```

とすると、実行結果は

```terminal
Start
End
Got: ABCDEFG
Returned: GFEDCBA
Stop Actor
```
となります。

このことから分かるように、FactoryメソッドでActorを宣言すると、宣言時点でActorが生成、実行されます。  
もし実行のタイミングを自分で制御したい場合は、`DefaultActor`クラスを継承します。

```groovy
import static groovyx.gpars.actor.Actors.actor
import groovyx.gpars.actor.DefaultActor

class Server extends DefaultActor {
    @Override protected void act() {
        loop {
            react { message -> 
                switch(message) {
                    case String:
                        println "Got: ${message}"
                        reply message.reverse()
                        break
                    default:
                        println "Stop Actor"
                        stop()
                }
            }
        }
    }
}

class Client extends DefaultActor {
    def server
    def Client(Server server) {
        super()
        this.server = server

    }
    @Override protected void act() {

        sleep(1000)
        server.send("ABCDEFG")
        sleep(1000)
        react {
            println "Returned: ${it}"
            server.send false

        }
    }
}
def server = new Server()
def client = new Client(server)
println "Start"
[server, client]*.start()
println "End"
```

これを実行すると、


```terminal
Start
End
Got: ABCDEFG
Returned: GFEDCBA
Stop Actor
```

となります。Factoryメソッドのパターンで`join()`を使わない場合と同じですね。  
既に述べたように、`DefaultActor`を継承したクラスを利用するとActorの実行タイミングを管理できます。  
上記のコードの実行部分を以下のようにすれば、Actorがそもそも実行されない（つまり実行のタイミングを自分で決められる）事が解ると思います。

```groovy
def server = new Server()
def client = new Client(server)
println "Start"
println "End"
```

というコードを実行すると実行結果は

```terminal
Start
End
```

のみになります。



```groovy
import static groovyx.gpars.actor.Actors.actor
import groovyx.gpars.actor.DefaultActor
import groovy.util.logging.*
import groovy.transform.Immutable

@Immutable class Ping {String message}
@Immutable class Pong {String message}

def server = actor {
    loop {
        react { message -> 
            switch(message) {
                case Ping:
                    println "Got: ${message.message}"
                    reply new Pong(message: message.message.reverse())
                    break
                default:
                    println "Stop Actor"
                    stop()
            }
        }
    }
}

def client = actor {
    loop {
        react {message ->
            switch(message) {
                case String:
                    reply new Ping(message: message)
                    break
                case Pong:
                    println "Returned: ${message.message}"
                    reply false // sendで指定されたActorに送信                   
                    stop() // 自Actorを停止
            }
        }
    }
}

println "Start"
assert server.isActive()
assert client.isActive()
client.send "TEST", server // 第2引数に別のActorを指定することで、reactの中のreplyの送信先を指定することができる。
[server, client]*.join(2, java.util.concurrent.TimeUnit.SECONDS)
println "End"
// タイムアウト（joinで2秒と指定）した場合、Actorがまだstopしていない場合は以下は両方trueとなる。
assert !server.isActive()
assert !client.isActive()
```