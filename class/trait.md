# Trait（トレイト）

**作成中**

```groovy
// http://docs.groovy-lang.org/next/html/documentation/core-traits.html
trait Human {
    String name = "koji"
    public Integer age = 31
    private twitter = "@saba1024"
    public String speak() {
        "I am Human"
    }
    // public、もしくはフィールドを宣言すると、ゲッターとセッターを用意するか、特種な記法でアクセスする必要がある。
    // ただし、アクセサメソッドを実装すると、専用記法（xxx.Human__age)が利用できなくなる。
    public Integer getAge(){
        age
    }
    public String getTwitter() {
        twitter
    }
    
}

class Man implements Human {}

def man = new Man()
assert "I am Human" == man.speak()
assert "koji" == man.name
//assert 31 == man.Human__age　// publicを指定すると、普通の方法ではアクセスできなくなる。
assert 31 == man.age // ただし、ゲッターを用意すれば今までどおり。
assert "@saba1024" == man.Human__twitter // privateを指定すると、publicと同様の動作。
assert "@saba1024" == man.twitter // twitterプロパティはprivateだけど、ゲッターを宣言しているのでこれで参照できる。



// 複数のtraitをimplementsして、同名のメソッドがある場合は、implementsの右側が優先される
trait A {String exec() {"A"}}
trait B {String exec(){"B"}}
class C implements A, B {}
def c = new C()
assert "B" == c.exec()

```
