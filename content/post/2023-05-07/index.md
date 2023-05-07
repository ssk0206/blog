---
title: Java Gold 対策勉強
description: 黒本第９章『例外とアサーション』
slug: java-gold-9
date: 2022-03-07 00:00:00+0000
categories:
    - プログラミング
tags:
    - Java
    - JavaGold
    - 資格
---

第９章は『例外とアサーション』ですね。これは実務でもなんとなくで書いてしまっているので念入りにやりたいです。

まずは、マルチキャッチについて。マルチキャッチを使うことで、コード量が劇的に減ります。
```java
try {
    // do
} catch (AException a | BException b) {
    // do
}
```
```java
try {
    // do
} catch (AException a) {

} catch (BException b) {

}
```

例外のマルチキャッチでは、同じ例外処理を一つのcatchブロックにまとめて記述することができます。同じ例外処理をしないのであれば、まとめるべきではないです。
ただし、一つのcatchブロックに記述できるのは、継承関係にない例外だけです。

##### プログラマーが独自に作成する例外クラス
Javaでは、プログラムが対処できるトラブルは「例外」、プログラムで対処できないトラブルは「エラー」とされている

共通しているのが、スローできるという性質
この性質を表すのが、java.lang.Throwableクラス

Throwable直下のサブクラスには、エラーを表すjava.lang.Errorと例外を表すjava.lang.Exceptionがあります。

Javaの例外は、例外処理(try-catch文、throws宣言)の有無をコンパイラがチェックする検査例外と、チェックしない非検査例外に別れます。Exceptionクラスとそのサブクラスは基本的に検査例外ですが、java.lang.RuntimeExceptionクラスとそのサブクラスだけは非検査例外として扱われます。

独自の例外クラスがExceptionクラスのサブクラスであることは必須条件ですが、RuntimeExceptionクラスのサブクラスである必要はラリません。

##### try-with-resources
try-with-resourcesはJava SE 7で導入された機能で、プログラムの中で扱うリソースを自動的に閉じるためのものです。
一般的にプログラムで扱うリソースとは、プログラムからアクセスするデータを指しますが、Javaではデータだけでなく、アクセスするためのインスタンスなどもリソースとして扱います。

try-with-resource文の括弧で扱えるのは、java.lang.AutoCloseableインタフェース、もしくはCloseableインタフェースを実装したクラス

AutoCloseableを実装したクラス
```java
public class SampleResource implements AutoCloseable {
    @Override
    public void close() throws Exception {
        System.out.println("close");
    }
}
```

Closeableを実装したクラス
```java
public class SampleResource implements Closeable {
    @Override
    public void close() throws IOException {
        System.out.println("close");
    }
}
```

try-with-resourcesの記述として、正しいもの
```
try (自動的に閉じるリソースの宣言) {
    リソースを使った処理
}
```

複数のリソースを対象とする場合は、";"を区切って列挙します

tryブロックの前でリソースを宣言
```java
public class SampleUsing {
    public static void main(String[] args) throws Exception {
        SampleResource resource = new SampleResource();
        try (resource) {
            // do
        }
    }
}
```

tryで宣言されたリソースが閉じる順番についての問題
**宣言したときと逆の順序** で自動的に閉じていく

tryブロック内で例外が発生しなければ、
リソースのクローズ -> finallyブロック
リソースのクローズ -> catchブロック -> finallyブロック
