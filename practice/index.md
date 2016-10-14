# 実践/サンプル集
色々具体的なGroovyを利用したツールを作ってみよう！！

## 10進数を2進数に変換する
普通にJavaの`Integer.toBinaryString()`を使えばいいけど、桁数をじったりしたい時や、論理演算した結果を2進数で得たい、という場合もある。 
ということで以下のようにすればいい感じで2進数を得られます  


```groovy
// 渡された10進数を指定したdigitSize桁の２進数に変換して返す。
// もしdigitSizeに満たない場合は0埋め。
def toBinStr = { Integer b, Integer digitSize = 4 -> 
    def binString = Integer.toBinaryString(b)
    def length = binString.length()
    String padding = (digitSize - length) > 0  ?  "0" * (digitSize - length) : ""
    (padding + binString)[-1..-digitSize].reverse()
}

Byte l1 = 4 as Byte // つまり0x04
Byte l2 = 6 as Byte // つまり0x06

assert toBinStr( l1 ) == '0100'
assert toBinStr( l2 ) == '0110'
assert toBinStr( l1|l2 ) == '0110' // OR 論理和
assert toBinStr( l1&l2 ) == '0100' // AND 論理積
assert toBinStr( l1^l2 ) == '0010'  // XOR 排他的論理和
assert toBinStr( ~(l1|l2) ) == '1001'  // NOT OR 論理和の否定（否定なので先頭が1埋めに成る）
assert toBinStr( ~(l1&l2) ) == '1011'  // NOT AND 論理積の否定（否定なので先頭が1埋めに成る）
assert toBinStr( ~(l1^l2) ) == '1101'  // NOT XOR 排他的論理和の否定（否定なので先頭が1埋めに成る）
assert toBinStr( 25 as Byte ) == '1001' // デフォルトだと4桁目までしか表示しない
assert toBinStr( 25 as Byte, 6 ) == '011001' // 桁を指定して表示
```