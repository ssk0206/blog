---
title: Java Gold 対策勉強
description: 黒本第８章『アノテーション』
slug: hello-world
date: 2022-03-06 00:00:00+0000
categories:
    - プログラミング
tags:
    - JavaGold
---

## アノテーションの定義

マーカーインターフェースとは、以下のコードのように何も持たず、クラスに意味を追加することだけを目的としたインタフェースのことです。

```java
public interface Sample {}
```

アノテーションは、プロセッサに何らかの処理をしてもらうため、クラスに意味を追加するという点ではマーカーインタフェースと同じですが、少し違います。
- プロセッサに値を渡すことができる。
- フィールドやメソッド、コンストラクタなど細かな単位でマークすることができる。

アノテーションを独自に定義するには、次のように@interfaceというワードにつづけて定義します。
```java
public @interface Sample {}
```
アノテーションは、コンパイルされるとインタフェースに置き換わります。
Sampleアノテーションをコンパイルした結果のクラスファイルをjavapコマンドで整形したもの

```
Compiled from "Sample.java"
public interface Sample extends java.lang.annotation.Annotation {}
```

アノテーションはjava.lang.annotation.Annotationインタフェースのサブインタフェースとして定義されます。
ただし、アノテーションはマーカーインターフェースとは異なり、implementsするわけではなく、アノテーションが付与されているという情報を残します。

```java
@Sample
public class Test {}
```

アノテーションはマーカーインタフェースと異なり、プロセッサに値を渡すことができる。アノテーションが付与されたクラスを定義するときに値を記述しておくことで、プロセッサはその値に応じた処理を実行することができます。

アノテーションで値を扱いたい場合、その値を戻す抽象メソッドをアノテーションに追加します。定義するのはフィールドではない点に注意してください。

```java
public @interface Test {
    String name();
    int print();
}

@Test(name = "test", price = 100)
public class Item {
    private String name;
    private int price;

    public Item(String name, int price) {
        this.name = name;
        this.price = price;
    }

    public String getName() {
        return name;
    }

    public int getPrice() {
        return price;
    }
}
```

アノテーションで扱う値のことを「注釈パラメータ」と呼びます。
注釈パラメータで値をプロセッサに渡したい場合は、「注釈パラメータ名＝値」の書式で記述します。

```java
public @interface SampleValue {
    String value();
}
```

注釈パラメータ１つの場合、省略することも可能
```java
@SampleValue(value = "sample")
public class UseSample {
}

@SampleValue("sample")
public class UseSample {
}
```

アノテーションの注釈パラメータに複数の値を設定することもできます。
```java
@ArrayValues(data = {1, 2, 3})
public class ArrayValueSample {
}
```

注釈パラメータの定義でdefaultにつづけてデフォルト値
```java
public @interface DefaultValue {
    String test() default "default";
}
```

```java
@DefaultValue
public class DefaultValueSample {
}
```

### リフレクション
リフレクションを使えば、そのインスタンスが持っているメソッドやフィールドの種類を調べて、インスタンスを操作することができます。
インタフェースやクラス群は`java.lang.reflect`パッケージに分類されています。

リフレクションを理解する上でポイントとなるのが、java.lang.Classクラスです。

```java
public class TestItem {
    public static void main(String[] args) {
        Class<Item> clazz = Item.class;
        Test test = clazz.getAnnotation(Test.class)
    }
}
```

メタ・アノテーションに関する問題です。
@Targetアノテーションは、アノテーションが何を対象としているのかを指定するためのメタ・アノテーションです。
@Targetアノテーションの注釈パラメータに渡す列挙型であるjava.lang.annotation.ElementTypeの列挙子には、次の表にあげる11種類があります。

| 列挙子  | 対象  |
|:--|:--|
| ANNOTATION_TYPE  | アノテーション宣言  |
|...|...|

```java
import java.lang.annotation.ElementType;
import java.lang.annotation.Retention;
import java.lang.annotation.RetentionPolicy;
import java.lang.annotation.Target;

@Target(ElementType.FIELD)
@Retention(RetentionPolicy.RUNTIME)
public @interface NotNull {
    String message();
}
```

```java
public class Sample {
    @NotNull(message = "name is not allowed null")
    private String name;
    public Sample(String name) {
        this.name = name;
    }
    public String getName() {
        return this.name;
    }
}
```


#### @Override
スーパークラスのメソッドを再定義することを「オーバーライド」と呼びます。
```java
public class A {
    public void hello() {
        System.out.println("hello.");
    }
}

public class B extends A {
    public void hello() {
        System.out.println("hi.");
    }
}
```
Aクラスのhelloメソッドを変更
```java
public class A {
    public void hello(String name) {
        System.out.println("hello." + name);
    }
}
```

スーパークラスのメソッドの引数を変更すると、オーバーライドではなく、
**オーバーロード(多重定義)** しているとコンパイラに解釈されます。
設計者からすると、このような解釈は望んでいないこともあります。

そこで、オーバーライドしたメソッドを変更し忘れたことをコンパイラが検知できる仕組みとして、
**@Overrideアノテーション** が提供されています。

```java
public class B extents A {
    @Override
    public void hello() {
        System.out.println("hi.");
    }
}
```
サブクラスでオーバーライドしたメソッドに@Overrideアノテーションをつけることにより、
スーパークラスのオーバーライドされた側のメソッドとシグニチャが一致するかどうかをコンパイラがチェックし、
一致しなければコンパイルエラーを出力します。

以上のことから、スーパークラスの変更を検知するために使えるアノテーションは、@Overrideアノテーション
@Deprecatedは、バージョン変更などによって使用できなくなる可能性があるものに付与するためのアノテーションです。@SuppressWarningsは、コンパイラが発する警告を抑制するためのアノテーションです。

@Deprecatedアノテーションに関する問題
長いソフトウェアのライフサイクルの中で、モジュール、インタフェース、クラスが提供する機能は変わっていきます。そのとき、下位互換性をどれくらい担保するかが課題になってきます。
下位互換性とは、ソフトウェアが変更され、新しいバージョンになっても、変更される前に使えた機能がそのまま使えるという性質を指します。

そこで、変更直後は変更前の機能をそのまま使えるようにしておき、変更後の機能へ移行する一定の期間を設ける方法が採られます。非推奨であることを明示するには、仕様書などのドキュメントやJavadocに記載する方法などがありますが、Java SE 5.0から@Deprecatedアノテーションが使えるようになりました。

@SuppressWarningsアノテーションで警告を抑制
```java
import java.util.ArrayList;
import java.util.List;

// コンパイラの警告を抑制することを明示
@SuppressWarnings("unchecked")
public class Sample {
    public void sample() {
        List list = new ArrayList();
        list.add("String");
    }
}
```

@SuppressWarningsアノテーションには、注釈パラメータとして抑制したい対象を表す文字列を渡します。
Java言語仕様で規定されている文字列は、「unchecked」「deprecation」「removal」の3つ。

