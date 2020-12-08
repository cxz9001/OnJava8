[TOC]

<!-- Generics -->

# 第二十章 泛型

> 普通的類和方法只能使用特定的類型：基本資料類型或類類型。如果編寫的程式碼需要應用於多種類型，這種嚴苛的限制對程式碼的束縛就會很大。

多態是一種物件導向思想的泛化機制。你可以將方法的參數類型設為基類，這樣的方法就可以接受任何衍生類作為參數，包括暫時還不存在的類。這樣的方法更通用，應用範圍更廣。在類內部也是如此，在任何使用特定類型的地方，基類意味著更大的靈活性。除了 `final` 類（或只提供私有建構子的類）任何類型都可被擴展，所以大部分時候這種靈活性是自帶的。

拘泥於單一的繼承體系太過局限，因為只有繼承體系中的物件才能適用基類作為參數的方法中。如果方法以介面而不是類作為參數，限制就寬鬆多了，只要實現了介面就可以。這給予呼叫方一種選項，透過調整現有的類來實現介面，滿足方法參數要求。介面可以突破繼承體系的限制。

即便是介面也還是有諸多限制。一旦指定了介面，它就要求你的程式碼必須使用特定的介面。而我們希望編寫更通用的程式碼，能夠適用“非特定的類型”，而不是一個具體的介面或類。

這就是泛型的概念，是 Java 5 的重大變化之一。泛型實現了*參數化類型*，這樣你編寫的元件（通常是集合）可以適用於多種類型。“泛型”這個術語的含義是“適用於很多類型”。程式語言中泛型出現的初衷是透過解耦類或方法與所使用的類型之間的約束，使得類或方法具備最寬泛的表達力。隨後你會發現 Java 中泛型的實現並沒有那麼“泛”，你可能會質疑“泛型”這個詞是否合適用來描述這一功能。

如果你從未接觸過參數化類型機制，你會發現泛型對 Java 語言確實是個很有益的補充。在你實例化一個類型參數時，編譯器會負責轉型並確保類型的正確性。這是一大進步。

然而，如果你了解其他語言（例如 C++ ）的參數化機制，你會發現，Java 泛型並不能滿足所有的預期。使用別人建立好的泛型相對容易，但是建立自己的泛型時，就會遇到很多意料之外的麻煩。

這並不是說 Java 泛型毫無用處。在很多情況下，它可以使程式碼更直接更優雅。不過，如果你見識過那種實現了更純粹的泛型的程式語言，那麼，Java 可能會令你失望。本章會介紹 Java 泛型的優點與局限。我會解釋 Java 的泛型是如何發展成現在這樣的，希望能夠幫助你更有效地使用這個特性。[^1]

### 與 C++ 的比較

Java 的設計者曾說過，這門語言的靈感主要來自 C++ 。儘管如此，學習 Java 時基本上不用參考 C++ 。

但是，Java 中的泛型需要與 C++ 進行對比，理由有兩個：首先，理解 C++ *模板*（泛型的主要靈感來源，包括基本語法）的某些特性，有助於理解泛型的基礎理念。同時，非常重要的一點是，你可以了解 Java 泛型的局限是什麼，以及為什麼會有這些局限。最終的目標是明確 Java 泛型的邊界，讓你成為一個程式高手。只有知道了某個技術不能做什麼，你才能更好地做到所能做的（部分原因是，不必浪費時間在死胡同裡）。

第二個原因是，在 Java 社群中，大家普遍對 C++ 模板有一種誤解，而這種誤解可能會令你在理解泛型的意圖時產生偏差。

因此，本章中會介紹少量 C++ 模板的例子，僅當它們確實可以加深理解時才會引入。

<!-- Simple Generics -->

## 簡單泛型

促成泛型出現的最主要的動機之一是為了建立*集合類*，參見 [集合](book/12-Collections.md) 章節。集合用於存放要使用到的物件。陣列也是如此，不過集合比陣列更加靈活，功能更豐富。幾乎所有程式在執行過程中都會涉及到一組物件，因此集合是可復用性最高的類庫之一。

我們先看一個只能持有單個物件的類。這個類可以明確指定其持有的物件的類型：

```java
// generics/Holder1.java

class Automobile {}

public class Holder1 {
    private Automobile a;
    public Holder1(Automobile a) { this.a = a; }
    Automobile get() { return a; }
}
```

這個類的可復用性不高，它無法持有其他類型的物件。我們可不希望為碰到的每個類型都編寫一個新的類。

在 Java 5 之前，我們可以讓這個類直接持有 `Object` 類型的物件：

```java
// generics/ObjectHolder.java

public class ObjectHolder {
    private Object a;
    public ObjectHolder(Object a) { this.a = a; }
    public void set(Object a) { this.a = a; }
    public Object get() { return a; }
    
    public static void main(String[] args) {
        ObjectHolder h2 = new ObjectHolder(new Automobile());
        Automobile a = (Automobile)h2.get();
        h2.set("Not an Automobile");
        String s = (String)h2.get();
        h2.set(1); // 自動裝箱為 Integer
        Integer x = (Integer)h2.get();
    }
}
```

現在，`ObjectHolder` 可以持有任何類型的物件，在上面的範例中，一個 `ObjectHolder` 先後持有了三種不同類型的物件。

一個集合中儲存多種不同類型的物件的情況很少見，通常而言，我們只會用集合儲存同一種類型的物件。泛型的主要目的之一就是用來約定集合要儲存什麼類型的物件，並且透過編譯器確保規約得以滿足。

因此，與其使用 `Object` ，我們更希望先指定一個類型占位符，稍後再決定具體使用什麼類型。要達到這個目的，需要使用*類型參數*，用角括號括住，放在類名後面。然後在使用這個類時，再用實際的類型取代此類型參數。在下面的例子中，`T` 就是類型參數：

```java
// generics/GenericHolder.java

public class GenericHolder<T> {
    private T a;
    public GenericHolder() {}
    public void set(T a) { this.a = a; }
    public T get() { return a; }
    
    public static void main(String[] args) {
        GenericHolder<Automobile> h3 = new GenericHolder<Automobile>();
        h3.set(new Automobile()); // 此處有類型校驗
        Automobile a = h3.get();  // 無需類型轉換
        //- h3.set("Not an Automobile"); // 報錯
        //- h3.set(1);  // 報錯
    }
}
```

建立 `GenericHolder` 物件時，必須指明要持有的物件的類型，將其置於角括號內，就像 `main()` 中那樣使用。然後，你就只能在 `GenericHolder` 中儲存該類型（或其子類，因為多態與泛型不衝突）的物件了。當你呼叫 `get()` 取值時，直接就是正確的類型。

這就是 Java 泛型的核心概念：你只需告訴編譯器要使用什麼類型，剩下的細節交給它來處理。

你可能注意到 `h3` 的定義非常繁複。在 `=` 左邊有 `GenericHolder<Automobile>`, 右邊又重複了一次。在 Java 5 中，這種寫法被解釋成“必要的”，但在 Java 7 中設計者修正了這個問題（新的簡寫語法隨後成為備受歡迎的特性）。以下是簡寫的例子：

```java
// generics/Diamond.java

class Bob {}

public class Diamond<T> {
    public static void main(String[] args) {
        GenericHolder<Bob> h3 = new GenericHolder<>();
        h3.set(new Bob());
    }
}
```

注意，在 `h3` 的定義處，`=` 右邊的角括號是空的（稱為“鑽石語法”），而不是重複左邊的類型訊息。在本書剩餘部分都會使用這種語法。

一般來說，你可以認為泛型和其他類型差不多，只不過它們碰巧有類型參數罷了。在使用泛型時，你只需要指定它們的名稱和類型參數列表即可。

### 一個元組類庫

有時一個方法需要能返回多個物件。而 **return** 語句只能返回單個物件，解決方法就是建立一個物件，用它打包想要返回的多個物件。當然，可以在每次需要的時候，專門建立一個類來完成這樣的工作。但是有了泛型，我們就可以一勞永逸。同時，還獲得了編譯時的類型安全。

這個概念稱為*元組*，它是將一組物件直接打包儲存於單一物件中。可以從該物件讀取其中的元素，但不允許向其中儲存新物件（這個概念也稱為 *資料傳輸物件* 或 *信使* ）。

通常，元組可以具有任意長度，元組中的物件可以是不同類型的。不過，我們希望能夠為每個物件指明類型，並且從元組中讀取出來時，能夠得到正確的類型。要處理不同長度的問題，我們需要建立多個不同的元組。下面是一個可以儲存兩個物件的元組：

```java
// onjava/Tuple2.java
package onjava;

public class Tuple2<A, B> {
    public final A a1;
    public final B a2;
    public Tuple2(A a, B b) { a1 = a; a2 = b; }
    public String rep() { return a1 + ", " + a2; }
  
    @Override
    public String toString() {
        return "(" + rep() + ")";
    }
}
```

建構子傳入要儲存的物件。這個元組隱式地保持了其中元素的次序。

初次閱讀上面的程式碼時，你可能認為這違反了 Java 編程的封裝原則。`a1` 和 `a2` 應該聲明為 **private**，然後提供 `getFirst()` 和 `getSecond()` 取值方法才對呀？考慮一下這樣做能提供的“安全性”是什麼：元組的使用程式可以讀取 `a1` 和 `a2` 然後對它們執行任何操作，但無法對 `a1` 和 `a2` 重新賦值。例子中的 `final` 可以實現同樣的效果，並且更為簡潔明瞭。

另一種設計思路是允許元組的使用者給 `a1` 和 `a2` 重新賦值。然而，採用上例中的形式無疑更加安全，如果使用者想儲存不同的元素，就會強制他們建立新的 `Tuple2` 物件。

我們可以利用繼承機制實現長度更長的元組。添加更多的類型參數就行了：

```java
// onjava/Tuple3.java
package onjava;

public class Tuple3<A, B, C> extends Tuple2<A, B> {
    public final C a3;
    public Tuple3(A a, B b, C c) {
        super(a, b);
        a3 = c;
    }
    
    @Override
    public String rep() {
        return super.rep() + ", " + a3;
    }
}

// onjava/Tuple4.java
package onjava;

public class Tuple4<A, B, C, D>
  extends Tuple3<A, B, C> {
    public final D a4;
    public Tuple4(A a, B b, C c, D d) {
        super(a, b, c);
        a4 = d;
    }
    
    @Override
    public String rep() {
        return super.rep() + ", " + a4;
    }
}

// onjava/Tuple5.java
package onjava;

public class Tuple5<A, B, C, D, E>
  extends Tuple4<A, B, C, D> {
    public final E a5;
    public Tuple5(A a, B b, C c, D d, E e) {
        super(a, b, c, d);
        a5 = e;
    }
    
    @Override
    public String rep() {
        return super.rep() + ", " + a5;
    }
}
```

示範需要，再定義兩個類：

```java
// generics/Amphibian.java
public class Amphibian {}

// generics/Vehicle.java
public class Vehicle {}
```

使用元組時，你只需要定義一個長度適合的元組，將其作為返回值即可。注意下面例子中方法的返回類型：

```java
// generics/TupleTest.java
import onjava.*;

public class TupleTest {
    static Tuple2<String, Integer> f() {
        // 47 自動裝箱為 Integer
        return new Tuple2<>("hi", 47);
    }
  
    static Tuple3<Amphibian, String, Integer> g() {
        return new Tuple3<>(new Amphibian(), "hi", 47);
    }
  
    static Tuple4<Vehicle, Amphibian, String, Integer> h() {
        return new Tuple4<>(new Vehicle(), new Amphibian(), "hi", 47);
    }
  
    static Tuple5<Vehicle, Amphibian, String, Integer, Double> k() {
        return new Tuple5<>(new Vehicle(), new Amphibian(), "hi", 47, 11.1);
    }
  
    public static void main(String[] args) {
        Tuple2<String, Integer> ttsi = f();
        System.out.println(ttsi);
        // ttsi.a1 = "there"; // 編譯錯誤，因為 final 不能重新賦值
        System.out.println(g());
        System.out.println(h());
        System.out.println(k());
    }
}

/* 輸出：
 (hi, 47)
 (Amphibian@1540e19d, hi, 47)
 (Vehicle@7f31245a, Amphibian@6d6f6e28, hi, 47)
 (Vehicle@330bedb4, Amphibian@2503dbd3, hi, 47, 11.1)
 */
```

有了泛型，你可以很容易地建立元組，令其返回一組任意類型的物件。

通過 `ttsi.a1 = "there"` 語句的報錯，我們可以看出，**final** 聲明確實可以確保 **public** 欄位在物件被構造出來之後就不能重新賦值了。

在上面的程式中，`new` 表達式有些囉嗦。本章稍後會介紹，如何利用 *泛型方法* 簡化它們。

### 一個堆疊類

接下來我們看一個稍微複雜一點的例子：堆疊。在 [集合](book/12-Collections.md) 一章中，我們用 `LinkedList` 實現了 `onjava.Stack` 類。在那個例子中，`LinkedList` 本身已經具備了建立堆疊所需的方法。`Stack` 是透過兩個泛型類 `Stack<T>` 和 `LinkedList<T>` 的組合來建立。我們可以看出，泛型只不過是一種類型罷了（稍後我們會看到一些例外的情況）。

這次我們不用 `LinkedList` 來實現自己的內部鏈式儲存機制。

```java
// generics/LinkedStack.java
// 用鏈式結構實現的堆疊

public class LinkedStack<T> {
    private static class Node<U> {
        U item;
        Node<U> next;
    
        Node() { item = null; next = null; }
        
        Node(U item, Node<U> next) {
            this.item = item;
            this.next = next;
        }
    
        boolean end() {
            return item == null && next == null;
        }
    }
  
    private Node<T> top = new Node<>();  // 堆疊頂
  
    public void push(T item) {
        top = new Node<>(item, top);
    }
  
    public T pop() {
        T result = top.item;
        if (!top.end()) {
            top = top.next;
        }
        return result;
    }
  
    public static void main(String[] args) {
        LinkedStack<String> lss = new LinkedStack<>();
        for (String s : "Phasers on stun!".split(" ")) {
            lss.push(s);
        }
        String s;
        while ((s = lss.pop()) != null) {
            System.out.println(s);
        }
    }
}
```

輸出結果：

```java
stun!
on
Phasers
```

內部類 `Node` 也是一個泛型，它擁有自己的類型參數。

這個例子使用了一個 *末端標識* (end sentinel) 來判斷堆疊何時為空。這個末端標識是在構造 `LinkedStack` 時建立的。然後，每次呼叫 `push()` 就會建立一個 `Node<T>` 物件，並將其連結到前一個 `Node<T>` 物件。當你呼叫 `pop()` 方法時，總是返回 `top.item`，然後丟棄目前 `top` 所指向的 `Node<T>`，並將 `top` 指向下一個 `Node<T>`，除非到達末端標識，這時就不能再移動 `top` 了。如果已經到達末端，程式還繼續呼叫 `pop()` 方法，它只能得到 `null`，說明堆疊已經空了。

### RandomList

作為容器的另一個例子，假設我們需要一個持有特定類型物件的列表，每次呼叫它的 `select()` 方法時都隨機返回一個元素。如果希望這種列表可以適用於各種類型，就需要使用泛型：

```java
// generics/RandomList.java
import java.util.*;
import java.util.stream.*;

public class RandomList<T> extends ArrayList<T> {
    private Random rand = new Random(47);
  
    public T select() {
        return get(rand.nextInt(size()));
    }
  
    public static void main(String[] args) {
        RandomList<String> rs = new RandomList<>();
        Arrays.stream("The quick brown fox jumped over the lazy brown dog".split(" ")).forEach(rs::add);
        IntStream.range(0, 11).forEach(i -> 
            System.out.print(rs.select() + " "));
    }
}
```

輸出結果：

```java
brown over fox quick quick dog brown The brown lazy brown
```

`RandomList` 繼承了 `ArrayList` 的所有方法。本例中只添加了 `select()` 這個方法。

<!-- Generic Interfaces -->

## 泛型介面

泛型也可以應用於介面。例如 *生成器*，這是一種專門負責建立物件的類。實際上，這是 *工廠方法* 設計模式的一種應用。不過，當使用生成器建立新的物件時，它不需要任何參數，而工廠方法一般需要參數。生成器無需額外的訊息就知道如何建立新物件。

一般而言，一個生成器只定義一個方法，用於建立物件。例如 `java.util.function` 類庫中的 `Supplier` 就是一個生成器，呼叫其 `get()` 獲取物件。`get()` 是泛型方法，返回值為類型參數 `T`。

為了示範 `Supplier`，我們需要定義幾個類。下面是個咖啡相關的繼承體系：

```java
// generics/coffee/Coffee.java
package generics.coffee;

public class Coffee {
    private static long counter = 0;
    private final long id = counter++;
  
    @Override
    public String toString() {
        return getClass().getSimpleName() + " " + id;
    }
}


// generics/coffee/Latte.java
package generics.coffee;
public class Latte extends Coffee {}


// generics/coffee/Mocha.java
package generics.coffee;
public class Mocha extends Coffee {}


// generics/coffee/Cappuccino.java
package generics.coffee;
public class Cappuccino extends Coffee {}


// generics/coffee/Americano.java
package generics.coffee;
public class Americano extends Coffee {}


// generics/coffee/Breve.java
package generics.coffee;
public class Breve extends Coffee {}
```

現在，我們可以編寫一個類，實現 `Supplier<Coffee>` 介面，它能夠隨機生成不同類型的 `Coffee` 物件：

```java
// generics/coffee/CoffeeSupplier.java
// {java generics.coffee.CoffeeSupplier}
package generics.coffee;
import java.util.*;
import java.util.function.*;
import java.util.stream.*;

public class CoffeeSupplier
implements Supplier<Coffee>, Iterable<Coffee> {
    private Class<?>[] types = { Latte.class, Mocha.class, 
        Cappuccino.class, Americano.class, Breve.class };
    private static Random rand = new Random(47);
  
    public CoffeeSupplier() {}
    // For iteration:
    private int size = 0;
    public CoffeeSupplier(int sz) { size = sz; }
  
    @Override
    public Coffee get() {
        try {
            return (Coffee) types[rand.nextInt(types.length)].newInstance();
        } catch (InstantiationException | IllegalAccessException e) {
            throw new RuntimeException(e);
        }
    }
  
    class CoffeeIterator implements Iterator<Coffee> {
        int count = size;
        @Override
        public boolean hasNext() { return count > 0; }
        @Override
        public Coffee next() {
            count--;
            return CoffeeSupplier.this.get();
        }
        @Override
        public void remove() {
            throw new UnsupportedOperationException();
        }
    }
  
    @Override
    public Iterator<Coffee> iterator() {
        return new CoffeeIterator();
    }
  
    public static void main(String[] args) {
        Stream.generate(new CoffeeSupplier())
              .limit(5)
              .forEach(System.out::println);
        for (Coffee c : new CoffeeSupplier(5)) {
            System.out.println(c);
        }
    }
}
```

輸出結果：

```java
Americano 0
Latte 1
Americano 2
Mocha 3
Mocha 4
Breve 5
Americano 6
Latte 7
Cappuccino 8
Cappuccino 9
```

參數化的 `Supplier` 介面確保 `get()` 返回值是參數的類型。`CoffeeSupplier` 同時還實現了 `Iterable` 介面，所以能用於 *for-in* 語句。不過，它還需要知道何時終止循環，這正是第二個建構子的作用。

下面是另一個實現 `Supplier<T>` 介面的例子，它負責生成 Fibonacci 數列：

```java
// generics/Fibonacci.java
// Generate a Fibonacci sequence
import java.util.function.*;
import java.util.stream.*;

public class Fibonacci implements Supplier<Integer> {
    private int count = 0;
    @Override
    public Integer get() { return fib(count++); }
  
    private int fib(int n) {
        if(n < 2) return 1;
        return fib(n-2) + fib(n-1);
    }
  
    public static void main(String[] args) {
        Stream.generate(new Fibonacci())
              .limit(18)
              .map(n -> n + " ")
              .forEach(System.out::print);
    }
}
```

輸出結果：

```java
1 1 2 3 5 8 13 21 34 55 89 144 233 377 610 987 1597 2584
```

雖然我們在 `Fibonacci` 類的裡裡外外使用的都是 `int` 類型，但是其參數類型卻是 `Integer`。這個例子引出了 Java 泛型的一個局限性：基本類型無法作為類型參數。不過 Java 5 具備自動裝箱和拆箱的功能，可以很方便地在基本類型和相應的包裝類之間進行轉換。透過這個例子中 `Fibonacci` 類對 `int` 的使用，我們已經看到了這種效果。

如果還想更進一步，編寫一個實現了 `Iterable` 的 `Fibnoacci` 生成器。我們的一個選擇是重寫這個類，令其實現 `Iterable` 介面。不過，你並不是總能擁有原始碼的控制權，並且，除非必須這麼做，否則，我們也不願意重寫一個類。而且我們還有另一種選擇，就是建立一個 *適配器* (Adapter) 來實現所需的介面，我們在前面介紹過這個設計模式。

有多種方法可以實現適配器。例如，可以透過繼承來建立適配器類：

```java
// generics/IterableFibonacci.java
// Adapt the Fibonacci class to make it Iterable
import java.util.*;

public class IterableFibonacci
extends Fibonacci implements Iterable<Integer> {
    private int n;
    public IterableFibonacci(int count) { n = count; }
  
    @Override
    public Iterator<Integer> iterator() {
        return new Iterator<Integer>() {
            @Override
            public boolean hasNext() { return n > 0; }
            @Override
            public Integer next() {
                n--;
                return IterableFibonacci.this.get();
            }
            @Override
            public void remove() { // Not implemented
                throw new UnsupportedOperationException();
            }
        };
    }
  
    public static void main(String[] args) {
        for(int i : new IterableFibonacci(18))
            System.out.print(i + " ");
    }
}
```

輸出結果：

```java
1 1 2 3 5 8 13 21 34 55 89 144 233 377 610 987 1597 2584
```

在 *for-in* 語句中使用 `IterableFibonacci`，必須在建構子中提供一個邊界值，這樣 `hasNext()` 才知道何時返回 **false**，結束循環。

<!-- Generic Methods -->

## 泛型方法

到目前為止，我們已經研究了參數化整個類。其實還可以參數化類中的方法。類本身可能是泛型的，也可能不是，不過這與它的方法是否是泛型的並沒有什麼關係。

泛型方法獨立於類而改變方法。作為準則，請“儘可能”使用泛型方法。通常將單個方法泛型化要比將整個類泛型化更清晰易懂。

如果方法是 **static** 的，則無法訪問該類的泛型類型參數，因此，如果使用了泛型類型參數，則它必須是泛型方法。

要定義泛型方法，請將泛型參數列表放置在返回值之前，如下所示：

```java
// generics/GenericMethods.java

public class GenericMethods {
    public <T> void f(T x) {
        System.out.println(x.getClass().getName());
    }

    public static void main(String[] args) {
        GenericMethods gm = new GenericMethods();
        gm.f("");
        gm.f(1);
        gm.f(1.0);
        gm.f(1.0F);
        gm.f('c');
        gm.f(gm);
    }
}
/* Output:
java.lang.String
java.lang.Integer
java.lang.Double
java.lang.Float
java.lang.Character
GenericMethods
*/
```

儘管可以同時對類及其方法進行參數化，但這裡未將 **GenericMethods** 類參數化。只有方法 `f()` 具有類型參數，該參數由方法返回類型之前的參數列表指示。

對於泛型類，必須在實例化該類時指定類型參數。使用泛型方法時，通常不需要指定參數類型，因為編譯器會找出這些類型。 這稱為 *類型參數推斷*。因此，對 `f()` 的呼叫看起來像普通的方法呼叫，並且 `f()` 看起來像被重載了無數次一樣。它甚至會接受 **GenericMethods** 類型的參數。

如果使用基本類型呼叫 `f()` ，自動裝箱就開始起作用，自動將基本類型包裝在它們對應的包裝類型中。

<!-- Varargs and Generic Methods -->
### 變長參數和泛型方法

泛型方法和變長參數列表可以很好地共存：

```java
// generics/GenericVarargs.java

import java.util.ArrayList;
import java.util.List;

public class GenericVarargs {
    @SafeVarargs
    public static <T> List<T> makeList(T... args) {
        List<T> result = new ArrayList<>();
        for (T item : args)
            result.add(item);
        return result;
    }

    public static void main(String[] args) {
        List<String> ls = makeList("A");
        System.out.println(ls);
        ls = makeList("A", "B", "C");
        System.out.println(ls);
        ls = makeList(
                "ABCDEFFHIJKLMNOPQRSTUVWXYZ".split(""));
        System.out.println(ls);
    }
}
/* Output:
[A]
[A, B, C]
[A, B, C, D, E, F, F, H, I, J, K, L, M, N, O, P, Q, R,
S, T, U, V, W, X, Y, Z]
*/
```

此處顯示的 `makeList()` 方法產生的功能與標準庫的 `java.util.Arrays.asList()` 方法相同。

`@SafeVarargs` 註解保證我們不會對變長參數列表進行任何修改，這是正確的，因為我們只從中讀取。如果沒有此註解，編譯器將無法知道這些並會發出警告。

<!-- A General-Purpose Supplier -->
### 一個泛型的 Supplier

這是一個為任意具有無參構造方法的類生成 **Supplier** 的類。為了減少鍵入，它還包括一個用於生成 **BasicSupplier** 的泛型方法：

```java
// onjava/BasicSupplier.java
// Supplier from a class with a no-arg constructor
package onjava;

import java.util.function.Supplier;

public class BasicSupplier<T> implements Supplier<T> {
    private Class<T> type;

    public BasicSupplier(Class<T> type) {
        this.type = type;
    }

    @Override
    public T get() {
        try {
            // Assumes type is a public class:
            return type.newInstance();
        } catch (InstantiationException |
                IllegalAccessException e) {
            throw new RuntimeException(e);
        }
    }

    // Produce a default Supplier from a type token:
    public static <T> Supplier<T> create(Class<T> type) {
        return new BasicSupplier<>(type);
    }
}
```

此類提供了產生以下物件的基本實現：

1. 是 **public** 的。 因為 **BasicSupplier** 在單獨的包中，所以相關的類必須具有 **public** 權限，而不僅僅是包級訪問權限。

2. 具有無參構造方法。要建立一個這樣的 **BasicSupplier** 物件，請呼叫 `create()` 方法，並將要生成類型的類型令牌傳遞給它。通用的 `create()` 方法提供了 `BasicSupplier.create(MyType.class)` 這種較簡潔的語法來代替較笨拙的 `new BasicSupplier <MyType>(MyType.class)`。

例如，這是一個具有無參構造方法的簡單類：

```java
// generics/CountedObject.java

public class CountedObject {
    private static long counter = 0;
    private final long id = counter++;

    public long id() {
        return id;
    }

    @Override
    public String toString() {
        return "CountedObject " + id;
    }
}
```

**CountedObject** 類可以跟蹤自身建立了多少個實例，並透過 `toString()` 報告這些實例的數量。 **BasicSupplier** 可以輕鬆地為 **CountedObject** 建立 **Supplier**：

```java
  // generics/BasicSupplierDemo.java

import onjava.BasicSupplier;

import java.util.stream.Stream;

public class BasicSupplierDemo {
    public static void main(String[] args) {
        Stream.generate(
                BasicSupplier.create(CountedObject.class))
                .limit(5)
                .forEach(System.out::println);
    }
}
/* Output:
CountedObject 0
CountedObject 1
CountedObject 2
CountedObject 3
CountedObject 4
*/
```

泛型方法減少了產生 **Supplier** 物件所需的程式碼量。 Java 泛型強制傳遞 **Class** 物件，以便在 `create()` 方法中將其用於類型推斷。

<!-- Simplifying Tuple Use -->
### 簡化元組的使用

使用類型參數推斷和靜態匯入，我們將把早期的元組重寫為更通用的庫。在這裡，我們使用重載的靜態方法建立元組：

```java
// onjava/Tuple.java
// Tuple library using type argument inference
package onjava;

public class Tuple {
    public static <A, B> Tuple2<A, B> tuple(A a, B b) {
        return new Tuple2<>(a, b);
    }

    public static <A, B, C> Tuple3<A, B, C>
    tuple(A a, B b, C c) {
        return new Tuple3<>(a, b, c);
    }

    public static <A, B, C, D> Tuple4<A, B, C, D>
    tuple(A a, B b, C c, D d) {
        return new Tuple4<>(a, b, c, d);
    }

    public static <A, B, C, D, E>
    Tuple5<A, B, C, D, E> tuple(A a, B b, C c, D d, E e) {
        return new Tuple5<>(a, b, c, d, e);
    }
}
```

我們修改 **TupleTest.java** 來測試 **Tuple.java** :

```java
// generics/TupleTest2.java

import onjava.Tuple2;
import onjava.Tuple3;
import onjava.Tuple4;
import onjava.Tuple5;

import static onjava.Tuple.tuple;

public class TupleTest2 {
    static Tuple2<String, Integer> f() {
        return tuple("hi", 47);
    }

    static Tuple2 f2() {
        return tuple("hi", 47);
    }

    static Tuple3<Amphibian, String, Integer> g() {
        return tuple(new Amphibian(), "hi", 47);
    }

    static Tuple4<Vehicle, Amphibian, String, Integer> h() {
        return tuple(
                new Vehicle(), new Amphibian(), "hi", 47);
    }

    static Tuple5<Vehicle, Amphibian,
            String, Integer, Double> k() {
        return tuple(new Vehicle(), new Amphibian(),
                "hi", 47, 11.1);
    }

    public static void main(String[] args) {
        Tuple2<String, Integer> ttsi = f();
        System.out.println(ttsi);
        System.out.println(f2());
        System.out.println(g());
        System.out.println(h());
        System.out.println(k());
    }
}
/* Output:
(hi, 47)
(hi, 47)
(Amphibian@14ae5a5, hi, 47)
(Vehicle@135fbaa4, Amphibian@45ee12a7, hi, 47)
(Vehicle@4b67cf4d, Amphibian@7ea987ac, hi, 47, 11.1)
*/
```

請注意，`f()` 返回一個參數化的 **Tuple2** 物件，而 `f2()` 返回一個未參數化的 **Tuple2** 物件。編譯器不會在這裡警告 `f2()` ，因為返回值未以參數化方式使用。從某種意義上說，它被“向上轉型”為一個未參數化的 **Tuple2** 。 但是，如果嘗試將 `f2()` 的結果放入到參數化的 **Tuple2** 中，則編譯器將發出警告。

<!-- A Set Utility -->
### 一個 Set 工具

對於泛型方法的另一個範例，請考慮由 **Set** 表示的數學關係。這些被方便地定義為可用於所有不同類型的泛型方法：

```java
// onjava/Sets.java

package onjava;

import java.util.HashSet;
import java.util.Set;

public class Sets {
    public static <T> Set<T> union(Set<T> a, Set<T> b) {
        Set<T> result = new HashSet<>(a);
        result.addAll(b);
        return result;
    }

    public static <T>
    Set<T> intersection(Set<T> a, Set<T> b) {
        Set<T> result = new HashSet<>(a);
        result.retainAll(b);
        return result;
    }

    // Subtract subset from superset:
    public static <T> Set<T>
    difference(Set<T> superset, Set<T> subset) {
        Set<T> result = new HashSet<>(superset);
        result.removeAll(subset);
        return result;
    }

    // Reflexive--everything not in the intersection:
    public static <T> Set<T> complement(Set<T> a, Set<T> b) {
        return difference(union(a, b), intersection(a, b));
    }
}
```

前三個方法透過將第一個參數的引用複製到新的 **HashSet** 物件中來複製第一個參數，因此不會直接修改參數集合。因此，返回值是一個新的 **Set** 物件。

這四種方法代表數學集合操作： `union()` 返回一個包含兩個參數聯集的 **Set** ， `intersection()` 返回一個包含兩個參數集合交集的 **Set** ， `difference()` 從 **superset** 中減去 **subset** 的元素 ，而 `complement()` 返回所有不在交集中的元素的 **Set**。作為顯示這些方法效果的簡單範例的一部分，下面是一個包含不同水彩名稱的 **enum** ：

```java
// generics/watercolors/Watercolors.java

package watercolors;

public enum Watercolors {
    ZINC, LEMON_YELLOW, MEDIUM_YELLOW, DEEP_YELLOW,
    ORANGE, BRILLIANT_RED, CRIMSON, MAGENTA,
    ROSE_MADDER, VIOLET, CERULEAN_BLUE_HUE,
    PHTHALO_BLUE, ULTRAMARINE, COBALT_BLUE_HUE,
    PERMANENT_GREEN, VIRIDIAN_HUE, SAP_GREEN,
    YELLOW_OCHRE, BURNT_SIENNA, RAW_UMBER,
    BURNT_UMBER, PAYNES_GRAY, IVORY_BLACK
}
```

為了方便起見（不必全限定所有名稱），將其靜態匯入到以下範例中。本範例使用 **EnumSet** 輕鬆從 **enum** 中建立 **Set** 。（可以在[第二十二章 列舉](book/22-Enumerations.md)一章中了解有關 **EnumSet** 的更多訊息。）在這裡，靜態方法 `EnumSet.range()` 要求提供所要在結果 **Set** 中建立的元素範圍的第一個和最後一個元素：

```java
// generics/WatercolorSets.java

import watercolors.*;

import java.util.EnumSet;
import java.util.Set;

import static watercolors.Watercolors.*;
import static onjava.Sets.*;

public class WatercolorSets {
    public static void main(String[] args) {
        Set<Watercolors> set1 =
                EnumSet.range(BRILLIANT_RED, VIRIDIAN_HUE);
        Set<Watercolors> set2 =
                EnumSet.range(CERULEAN_BLUE_HUE, BURNT_UMBER);
        System.out.println("set1: " + set1);
        System.out.println("set2: " + set2);
        System.out.println(
                "union(set1, set2): " + union(set1, set2));
        Set<Watercolors> subset = intersection(set1, set2);
        System.out.println(
                "intersection(set1, set2): " + subset);
        System.out.println("difference(set1, subset): " +
                difference(set1, subset));
        System.out.println("difference(set2, subset): " +
                difference(set2, subset));
        System.out.println("complement(set1, set2): " +
                complement(set1, set2));
    }
}
/* Output:
set1: [BRILLIANT_RED, CRIMSON, MAGENTA, ROSE_MADDER,
VIOLET, CERULEAN_BLUE_HUE, PHTHALO_BLUE, ULTRAMARINE,
COBALT_BLUE_HUE, PERMANENT_GREEN, VIRIDIAN_HUE]
set2: [CERULEAN_BLUE_HUE, PHTHALO_BLUE, ULTRAMARINE,
COBALT_BLUE_HUE, PERMANENT_GREEN, VIRIDIAN_HUE,
SAP_GREEN, YELLOW_OCHRE, BURNT_SIENNA, RAW_UMBER,
BURNT_UMBER]
union(set1, set2): [BURNT_SIENNA, BRILLIANT_RED,
YELLOW_OCHRE, MAGENTA, SAP_GREEN, CERULEAN_BLUE_HUE,
ULTRAMARINE, VIRIDIAN_HUE, VIOLET, RAW_UMBER,
ROSE_MADDER, PERMANENT_GREEN, BURNT_UMBER,
PHTHALO_BLUE, CRIMSON, COBALT_BLUE_HUE]
intersection(set1, set2): [PERMANENT_GREEN,
CERULEAN_BLUE_HUE, ULTRAMARINE, VIRIDIAN_HUE,
PHTHALO_BLUE, COBALT_BLUE_HUE]
difference(set1, subset): [BRILLIANT_RED, MAGENTA,
VIOLET, CRIMSON, ROSE_MADDER]
difference(set2, subset): [BURNT_SIENNA, YELLOW_OCHRE,
BURNT_UMBER, SAP_GREEN, RAW_UMBER]
complement(set1, set2): [BURNT_SIENNA, BRILLIANT_RED,
YELLOW_OCHRE, MAGENTA, SAP_GREEN, VIOLET, RAW_UMBER,
ROSE_MADDER, BURNT_UMBER, CRIMSON]
*/
```

接下來的例子使用 `Sets.difference()` 方法來展示 **java.util** 包中各種 **Collection** 和 **Map** 類之間的方法差異：

```java
// onjava/CollectionMethodDifferences.java
// {java onjava.CollectionMethodDifferences}

package onjava;

import java.lang.reflect.Method;
import java.util.*;
import java.util.stream.Collectors;

public class CollectionMethodDifferences {
    static Set<String> methodSet(Class<?> type) {
        return Arrays.stream(type.getMethods())
                .map(Method::getName)
                .collect(Collectors.toCollection(TreeSet::new));
    }

    static void interfaces(Class<?> type) {
        System.out.print("Interfaces in " +
                type.getSimpleName() + ": ");
        System.out.println(
                Arrays.stream(type.getInterfaces())
                        .map(Class::getSimpleName)
                        .collect(Collectors.toList()));
    }

    static Set<String> object = methodSet(Object.class);

    static {
        object.add("clone");
    }

    static void
    difference(Class<?> superset, Class<?> subset) {
        System.out.print(superset.getSimpleName() +
                " extends " + subset.getSimpleName() +
                ", adds: ");
        Set<String> comp = Sets.difference(
                methodSet(superset), methodSet(subset));
        comp.removeAll(object); // Ignore 'Object' methods
        System.out.println(comp);
        interfaces(superset);
    }

    public static void main(String[] args) {
        System.out.println("Collection: " +
                methodSet(Collection.class));
        interfaces(Collection.class);
        difference(Set.class, Collection.class);
        difference(HashSet.class, Set.class);
        difference(LinkedHashSet.class, HashSet.class);
        difference(TreeSet.class, Set.class);
        difference(List.class, Collection.class);
        difference(ArrayList.class, List.class);
        difference(LinkedList.class, List.class);
        difference(Queue.class, Collection.class);
        difference(PriorityQueue.class, Queue.class);
        System.out.println("Map: " + methodSet(Map.class));
        difference(HashMap.class, Map.class);
        difference(LinkedHashMap.class, HashMap.class);
        difference(SortedMap.class, Map.class);
        difference(TreeMap.class, Map.class);
    }
}
/* Output:
Collection: [add, addAll, clear, contains, containsAll,
equals, forEach, hashCode, isEmpty, iterator,
parallelStream, remove, removeAll, removeIf, retainAll,
size, spliterator, stream, toArray]
Interfaces in Collection: [Iterable]
Set extends Collection, adds: []
Interfaces in Set: [Collection]
HashSet extends Set, adds: []
Interfaces in HashSet: [Set, Cloneable, Serializable]
LinkedHashSet extends HashSet, adds: []
Interfaces in LinkedHashSet: [Set, Cloneable,
Serializable]
TreeSet extends Set, adds: [headSet,
descendingIterator, descendingSet, pollLast, subSet,
floor, tailSet, ceiling, last, lower, comparator,
pollFirst, first, higher]
Interfaces in TreeSet: [NavigableSet, Cloneable,
Serializable]
List extends Collection, adds: [replaceAll, get,
indexOf, subList, set, sort, lastIndexOf, listIterator]
Interfaces in List: [Collection]
ArrayList extends List, adds: [trimToSize,
ensureCapacity]
Interfaces in ArrayList: [List, RandomAccess,
Cloneable, Serializable]
LinkedList extends List, adds: [offerFirst, poll,
getLast, offer, getFirst, removeFirst, element,
removeLastOccurrence, peekFirst, peekLast, push,
pollFirst, removeFirstOccurrence, descendingIterator,
pollLast, removeLast, pop, addLast, peek, offerLast,
addFirst]
Interfaces in LinkedList: [List, Deque, Cloneable,
Serializable]
Queue extends Collection, adds: [poll, peek, offer,
element]
Interfaces in Queue: [Collection]
PriorityQueue extends Queue, adds: [comparator]
Interfaces in PriorityQueue: [Serializable]
Map: [clear, compute, computeIfAbsent,
computeIfPresent, containsKey, containsValue, entrySet,
equals, forEach, get, getOrDefault, hashCode, isEmpty,
keySet, merge, put, putAll, putIfAbsent, remove,
replace, replaceAll, size, values]
HashMap extends Map, adds: []
Interfaces in HashMap: [Map, Cloneable, Serializable]
LinkedHashMap extends HashMap, adds: []
Interfaces in LinkedHashMap: [Map]
SortedMap extends Map, adds: [lastKey, subMap,
comparator, firstKey, headMap, tailMap]
Interfaces in SortedMap: [Map]
TreeMap extends Map, adds: [descendingKeySet,
navigableKeySet, higherEntry, higherKey, floorKey,
subMap, ceilingKey, pollLastEntry, firstKey, lowerKey,
headMap, tailMap, lowerEntry, ceilingEntry,
descendingMap, pollFirstEntry, lastKey, firstEntry,
floorEntry, comparator, lastEntry]
Interfaces in TreeMap: [NavigableMap, Cloneable,
Serializable]
*/
```

在第十二章 [集合的本章小結](book/12-Collections.md#本章小結) 部分將會用到這裡的輸出結果。

<!-- Building Complex Models -->

## 構建複雜模型

泛型的一個重要好處是能夠簡單安全地建立複雜模型。例如，我們可以輕鬆地建立一個元組列表：

```java
// generics/TupleList.java
// Combining generic types to make complex generic types

import onjava.Tuple4;

import java.util.ArrayList;

public class TupleList<A, B, C, D>
        extends ArrayList<Tuple4<A, B, C, D>> {
    public static void main(String[] args) {
        TupleList<Vehicle, Amphibian, String, Integer> tl =
                new TupleList<>();
        tl.add(TupleTest2.h());
        tl.add(TupleTest2.h());
        tl.forEach(System.out::println);
    }
}
/* Output:
(Vehicle@7cca494b, Amphibian@7ba4f24f, hi, 47)
(Vehicle@3b9a45b3, Amphibian@7699a589, hi, 47)
*/
```

這將產生一個功能強大的資料結構，而無需太多程式碼。

下面是第二個例子。每個類都是組成塊，總體包含很多個塊。在這裡，該模型是一個具有過道，貨架和產品的零售商店：

```java
// generics/Store.java
// Building a complex model using generic collections

import onjava.Suppliers;

import java.util.ArrayList;
import java.util.Random;
import java.util.function.Supplier;

class Product {
    private final int id;
    private String description;
    private double price;

    Product(int idNumber, String descr, double price) {
        id = idNumber;
        description = descr;
        this.price = price;
        System.out.println(toString());
    }

    @Override
    public String toString() {
        return id + ": " + description +
                ", price: $" + price;
    }

    public void priceChange(double change) {
        price += change;
    }

    public static Supplier<Product> generator =
            new Supplier<Product>() {
                private Random rand = new Random(47);

                @Override
                public Product get() {
                    return new Product(rand.nextInt(1000), "Test",
                            Math.round(
                                    rand.nextDouble() * 1000.0) + 0.99);
                }
            };
}

class Shelf extends ArrayList<Product> {
    Shelf(int nProducts) {
        Suppliers.fill(this, Product.generator, nProducts);
    }
}

class Aisle extends ArrayList<Shelf> {
    Aisle(int nShelves, int nProducts) {
        for (int i = 0; i < nShelves; i++)
            add(new Shelf(nProducts));
    }
}

class CheckoutStand {
}

class Office {
}

public class Store extends ArrayList<Aisle> {
    private ArrayList<CheckoutStand> checkouts =
            new ArrayList<>();
    private Office office = new Office();

    public Store(
            int nAisles, int nShelves, int nProducts) {
        for (int i = 0; i < nAisles; i++)
            add(new Aisle(nShelves, nProducts));
    }

    @Override
    public String toString() {
        StringBuilder result = new StringBuilder();
        for (Aisle a : this)
            for (Shelf s : a)
                for (Product p : s) {
                    result.append(p);
                    result.append("\n");
                }
        return result.toString();
    }

    public static void main(String[] args) {
        System.out.println(new Store(5, 4, 3));
    }
}
/* Output: (First 8 Lines)
258: Test, price: $400.99
861: Test, price: $160.99
868: Test, price: $417.99
207: Test, price: $268.99
551: Test, price: $114.99
278: Test, price: $804.99
520: Test, price: $554.99
140: Test, price: $530.99
                  ...
*/
```

`Store.toString()` 顯示了結果：儘管有複雜的層次結構，但多層的集合仍然是類型安全的和可管理的。令人印象深刻的是，組裝這樣的模型並不需要耗費過多精力。

**Shelf** 使用 `Suppliers.fill()` 這個實用程式，該實用程式接受 **Collection** （第一個參數），並使用 **Supplier** （第二個參數），以元素的數量為 **n** （第三個參數）來填充它。 **Suppliers** 類將會在本章末尾定義，其中的方法都是在執行某種填充操作，並在本章的其他範例中使用。

<!-- The Mystery of Erasure -->
## 泛型擦除

當你開始更深入地鑽研泛型時，會發現有大量的東西初看起來是沒有意義的。例如，儘管可以說  `ArrayList.class`，但不能說成 `ArrayList<Integer>.class`。考慮下面的情況：

```java
// generics/ErasedTypeEquivalence.java

import java.util.*;

public class ErasedTypeEquivalence {
    
    public static void main(String[] args) {
        Class c1 = new ArrayList<String>().getClass();
        Class c2 = new ArrayList<Integer>().getClass();
        System.out.println(c1 == c2);
    }
    
}
/* Output:
true
*/
```

`ArrayList<String>` 和 `ArrayList<Integer>` 應該是不同的類型。不同的類型會有不同的行為。例如，如果嘗試向 `ArrayList<String>` 中放入一個 `Integer`，所得到的行為（失敗）和向 `ArrayList<Integer>` 中放入一個 `Integer` 所得到的行為（成功）完全不同。然而上面的程式認為它們是相同的類型。

下面的例子是對該謎題的補充：

```java
// generics/LostInformation.java

import java.util.*;

class Frob {}
class Fnorkle {}
class Quark<Q> {}

class Particle<POSITION, MOMENTUM> {}

public class LostInformation {

    public static void main(String[] args) {
        List<Frob> list = new ArrayList<>();
        Map<Frob, Fnorkle> map = new HashMap<>();
        Quark<Fnorkle> quark = new Quark<>();
        Particle<Long, Double> p = new Particle<>();
        System.out.println(Arrays.toString(list.getClass().getTypeParameters()));
        System.out.println(Arrays.toString(map.getClass().getTypeParameters()));
        System.out.println(Arrays.toString(quark.getClass().getTypeParameters()));
        System.out.println(Arrays.toString(p.getClass().getTypeParameters()));
    }

}
/* Output:
[E]
[K,V]
[Q]
[POSITION,MOMENTUM]
*/
```

根據 JDK 文件，**Class.getTypeParameters()** “返回一個 **TypeVariable** 物件陣列，表示泛型聲明中聲明的類型參數...” 這暗示你可以發現這些參數類型。但是正如上例中輸出所示，你只能看到用作參數占位符的標識符，這並非有用的訊息。

殘酷的現實是：

在泛型程式碼內部，無法獲取任何有關泛型參數類型的訊息。

因此，你可以知道如類型參數標識符和泛型邊界這些訊息，但無法得知實際的類型參數從而用來建立特定的實例。如果你曾是 C++ 程式設計師，那麼這個事實會讓你很沮喪，在使用 Java 泛型工作時，它是必須處理的最基本的問題。

Java 泛型是使用擦除實現的。這意味著當你在使用泛型時，任何具體的類型訊息都被擦除了，你唯一知道的就是你在使用一個物件。因此，`List<String>` 和 `List<Integer>` 在執行時實際上是相同的類型。它們都被擦除成原生類型 `List`。

理解擦除並知道如何處理它，是你在學習 Java 泛型時面臨的最大障礙之一。這也是本節將要探討的內容。

### C++ 的方式

下面是使用模版的 C++ 範例。你會看到類型參數的語法十分相似，因為 Java 是受 C++ 啟發的：

```c++
// generics/Templates.cpp

#include <iostream>
using namespace std;

template<class T> class Manipulator {
    T obj;
public:
    Manipulator(T x) { obj = x; }
    void manipulate() { obj.f(); }
};

class HasF {
public:
    void f() { cout << "HasF::f()" << endl; }
};

int main() {
    HasF hf;
    Manipulator<HasF> manipulator(hf);
    manipulator.manipulate();
}
/* Output:
HasF::f()
*/
```

**Manipulator** 類儲存了一個 **T** 類型的物件。`manipulate()` 方法會呼叫 **obj** 上的 `f()` 方法。它是如何知道類型參數 **T** 中存在 `f()` 方法的呢？C++ 編譯器會在你實例化模版時進行檢查，所以在 `Manipulator<HasF>` 實例化的那一刻，它看到 **HasF** 中含有一個方法 `f()`。如果情況並非如此，你就會得到一個編譯期錯誤，保持類型安全。

用 C++ 編寫這種程式碼很簡單，因為當模版被實例化時，模版程式碼就知道模版參數的類型。Java 泛型就不同了。下面是 **HasF** 的 Java 版本：

```java
// generics/HasF.java

public class HasF {
    public void f() {
        System.out.println("HasF.f()");
    }
}
```

如果我們將範例的其餘程式碼用 Java 實現，就不會透過編譯：

```java
// generics/Manipulation.java
// {WillNotCompile}

class Manipulator<T> {
    private T obj;
    
    Manipulator(T x) {
        obj = x;
    }
    
    // Error: cannot find symbol: method f():
    public void manipulate() {
        obj.f();
    }
}

public class Manipulation {
	public static void main(String[] args) {
        HasF hf = new HasF();
        Manipulator<HasF> manipulator = new Manipulator<>(hf);
        manipulator.manipulate();
    }
}
```

因為擦除，Java 編譯器無法將 `manipulate()` 方法必須能呼叫 **obj** 的 `f()` 方法這一需求映射到 HasF 具有 `f()` 方法這個事實上。為了呼叫 `f()`，我們必須協助泛型類，給定泛型類一個邊界，以此告訴編譯器只能接受遵循這個邊界的類型。這裡重用了 **extends** 關鍵字。由於有了邊界，下面的程式碼就能透過編譯：

```java
public class Manipulator2<T extends HasF> {
    private T obj;

    Manipulator2(T x) {
        obj = x;
    }

    public void manipulate() {
        obj.f();
    }
}
```

邊界 `<T extends HasF>` 聲明 T 必須是 HasF 類型或其子類。如果情況確實如此，就可以安全地在 **obj** 上呼叫 `f()` 方法。

我們說泛型類型參數會擦除到它的第一個邊界（可能有多個邊界，稍後你將看到）。我們還提到了類型參數的擦除。編譯器實際上會把類型參數取代為它的擦除，就像上面的範例，**T** 擦除到了 **HasF**，就像在類的聲明中用 **HasF** 取代了 **T** 一樣。

你可能正確地觀察到了泛型在 **Manipulator2.java** 中沒有貢獻任何事。你可以很輕鬆地自己去執行擦除，生成沒有泛型的類：

```java
// generics/Manipulator3.java

class Manipulator3 {
    private HasF obj;
    
    Manipulator3(HasF x) {
        obj = x;
    }
    
    public void manipulate() {
        obj.f();
    }
}
```

這提出了很重要的一點：泛型只有在類型參數比某個具體類型（以及其子類）更加“泛化”——程式碼能跨多個類工作時才有用。因此，類型參數和它們在有用的泛型程式碼中的應用，通常比簡單的類取代更加複雜。但是，不能因此認為使用 `<T extends HasF>` 形式就是有缺陷的。例如，如果某個類有一個返回 **T** 的方法，那麼泛型就有所幫助，因為它們之後將返回確切的類型：

```java
// generics/ReturnGenericType.java

public class ReturnGenericType<T extends HasF> {
    private T obj;
    
    ReturnGenericType(T x) {
        obj = x;
    }
    
    public T get() {
        return obj;
    }
}
```

你必須查看所有的程式碼，從而確定程式碼是否複雜到必須使用泛型的程度。

我們將在本章稍後看到有關邊界的更多細節。

### 遷移相容性

為了減少潛在的關於擦除的困惑，你必須清楚地認識到這不是一個語言特性。它是 Java 實現泛型的一種妥協，因為泛型不是 Java 語言出現時就有的，所以就有了這種妥協。它會使你痛苦，因此你需要儘早習慣它並了解為什麼它會這樣。

如果 Java 1.0 就含有泛型的話，那麼這個特性就不會使用擦除來實現——它會使用具體化，保持參數類型為第一類實體，因此你就能在類型參數上執行基於類型的語言操作和反射操作。本章稍後你會看到，擦除減少了泛型的泛化性。泛型在 Java 中仍然是有用的，只是不如它們本來設想的那麼有用，而原因就是擦除。

在基於擦除的實現中，泛型類型被當作第二類類型處理，即不能在某些重要的上下文使用泛型類型。泛型類型只有在靜態類型檢測期間才出現，在此之後，程式中的所有泛型類型都將被擦除，取代為它們的非泛型上界。例如， `List<T>` 這樣的類型註解會被擦除為 **List**，普通的類型變數在未指定邊界的情況下會被擦除為 **Object**。

擦除的核心動機是你可以在泛化的用戶端上使用非泛型的類庫，反之亦然。這經常被稱為“遷移相容性”。在理想情況下，所有事物將在指定的某天被泛化。在現實中，即使程式設計師只編寫泛型程式碼，他們也必須處理 Java 5 之前編寫的非泛型類庫。這些類庫的作者可能從沒想過要泛化他們的程式碼，或許他們可能剛剛開始接觸泛型。

因此 Java 泛型不僅必須支援向後相容性——現有的程式碼和類文件仍然合法，繼續保持之前的含義——而且還必須支援遷移相容性，使得類庫能按照它們自己的步調變為泛型，當某個類庫變為泛型時，不會破壞依賴於它的程式碼和應用。在確定了這個目標後，Java 設計者們和從事此問題相關工作的各個團隊決策認為擦除是唯一可行的解決方案。擦除使得這種向泛型的遷移成為可能，允許非泛型的程式碼和泛型程式碼共存。

例如，假設一個應用使用了兩個類庫 **X** 和 **Y**，**Y** 使用了類庫 **Z**。隨著 Java 5 的出現，這個應用和這些類庫的建立者最終可能希望遷移到泛型上。但是當進行遷移時，它們有著不同的動機和限制。為了實現遷移相容性，每個類庫與應用必須與其他所有的部分是否使用泛型無關。因此，它們不能探測其他類庫是否使用了泛型。因此，某個特定的類庫使用了泛型這樣的證據必須被”擦除“。

如果沒有某種類型的遷移途徑，所有已經構建了很長時間的類庫就需要與希望遷移到 Java 泛型上的開發者們說再見了。類庫毫無爭議是程式語言的一部分，對生產效率有著極大的影響，所以這種程式碼無法接受。擦除是否是最佳的或唯一的遷移途徑，還待時間來證明。

### 擦除的問題

因此，擦除主要的正當理由是從非泛化程式碼到泛化程式碼的轉變過程，以及在不破壞現有類庫的情況下將泛型融入到語言中。擦除允許你繼續使用現有的非泛型用戶端程式碼，直至用戶端準備好用泛型重寫這些程式碼。這是一個崇高的動機，因為它不會驟然破壞所有現有的程式碼。

擦除的代價是顯著的。泛型不能用於顯式地引用執行時類型的操作中，例如轉型、**instanceof** 操作和 **new** 表達式。因為所有關於參數的類型訊息都遺失了，當你在編寫泛型程式碼時，必須時刻提醒自己，你只是看起來擁有有關參數的類型訊息而已。

考慮如下的程式碼段：

```java
class Foo<T> {
    T var;
}
```

看起來當你建立一個 **Foo** 實例時：

```java
Foo<Cat> f = new Foo<>();
```

**class** **Foo** 中的程式碼應該知道現在工作於 **Cat** 之上。泛型語法也在強烈暗示整個類中所有 T 出現的地方都被取代，就像在 C++ 中一樣。但是事實並非如此，當你在編寫這個類的程式碼時，必須提醒自己：“不，這只是一個 **Object**“。

另外，擦除和遷移相容性意味著，使用泛型並不是強制的，儘管你可能希望這樣：

```java
// generics/ErasureAndInheritance.java

class GenericBase<T> {
    private T element;
    
    public void set(T arg) {
        element = arg;
    }
    
    public T get() {
        return element;
    }
}

class Derived1<T> extends GenericBase<T> {}

class Derived2 extends GenericBase {} // No warning

// class Derived3 extends GenericBase<?> {}
// Strange error:
// unexpected type
// required: class or interface without bounds
public class ErasureAndInteritance {
    @SuppressWarnings("unchecked")
    public static void main(String[] args) {
        Derived2 d2 = new Derived2();
        Object obj = d2.get();
        d2.set(obj); // Warning here!
    }
}
```

**Derived2** 繼承自 **GenericBase**，但是沒有任何類型參數，編譯器沒有發出任何警告。直到呼叫 `set()` 方法時才出現警告。

為了關閉警告，Java 提供了一個註解，我們可以在列表中看到它：

```java
@SuppressWarnings("unchecked")
```

這個註解放置在產生警告的方法上，而不是整個類上。當你要關閉警告時，最好儘可能地“聚焦”，這樣就不會因為過於寬泛地關閉警告，而導致意外地遮蔽掉真正的問題。

可以推斷，**Derived3** 產生的錯誤意味著編譯器期望得到一個原生基類。

當你希望將類型參數不僅僅當作 Object 處理時，就需要付出額外努力來管理邊界，並且與在 C++、Ada 和 Eiffel 這樣的語言中獲得參數化類型相比，你需要付出多得多的努力來獲得少得多的回報。這並不是說，對於大多數的程式問題而言，這些語言通常都會比 Java 更得心應手，只是說它們的參數化類型機制相比 Java 更靈活、更強大。

### 邊界處的動作

因為擦除，我發現了泛型最令人困惑的方面是可以表示沒有任何意義的事物。例如：

```java
// generics/ArrayMaker.java

import java.lang.reflect.*;
import java.util.*;

public class ArrayMaker<T> {
    private Class<T> kind;

    public ArrayMaker(Class<T> kind) {
        this.kind = kind;
    }

    @SuppressWarnings("unchecked")
    T[] create(int size) {
        return (T[]) Array.newInstance(kind, size);
    }

    public static void main(String[] args) {
        ArrayMaker<String> stringMaker = new ArrayMaker<>(String.class);
        String[] stringArray = stringMaker.create(9);
        System.out.println(Arrays.toString(stringArray));
    }
}
/* Output
[null,null,null,null,null,null,null,null,null]
*/
```

即使 **kind** 被儲存為 `Class<T>`，擦除也意味著它實際被儲存為沒有任何參數的 **Class**。因此，當你在使用它時，例如建立陣列，`Array.newInstance()` 實際上並未擁有 **kind** 所蘊含的類型訊息。所以它不會產生具體的結果，因而必須轉型，這會產生一條令你無法滿意的警告。

注意，對於在泛型中建立陣列，使用 `Array.newInstance()` 是推薦的方式。

如果我們建立一個集合而不是陣列，情況就不同了：

```java
// generics/ListMaker.java

import java.util.*;

public class ListMaker<T> {
    List<T> create() {
        return new ArrayList<>();
    }
    
    public static void main(String[] args) {
        ListMaker<String> stringMaker = new ListMaker<>();
        List<String> stringList = stringMaker.create();
    }
}
```

編譯器不會給出任何警告，儘管我們知道（從擦除中）在 `create()` 內部的 `new ArrayList<>()` 中的 `<T>` 被移除了——在執行時，類內部沒有任何 `<T>`，因此這看起來毫無意義。但是如果你遵從這種思路，並將這個表達式改為 `new ArrayList()`，編譯器就會發出警告。

本例中這麼做真的毫無意義嗎？如果在建立 **List** 的同時向其中放入一些物件呢，像這樣：

```java
// generics/FilledList.java

import java.util.*;
import java.util.function.*;
import onjava.*;

public class FilledList<T> extends ArrayList<T> {
    FilledList(Supplier<T> gen, int size) {
        Suppliers.fill(this, gen, size);
    }
    
    public FilledList(T t, int size) {
        for (int i = 0; i < size; i++) {
            this.add(t);
        }
    }
    
    public static void main(String[] args) {
        List<String> list = new FilledList<>("Hello", 4);
        System.out.println(list);
        // Supplier version:
        List<Integer> ilist = new FilledList<>(() -> 47, 4);
        System.out.println(ilist);
    }
}
/* Output:
[Hello,Hello,Hello,Hello]
[47,47,47,47]
*/
```

即使編譯器無法得知 `add()` 中的 **T** 的任何訊息，但它仍可以在編譯期確保你放入 **FilledList** 中的物件是 **T** 類型。因此，即使擦除移除了方法或類中的實際類型的訊息，編譯器仍可以確保方法或類中使用的類型的內部一致性。

因為擦除移除了方法體中的類型訊息，所以在執行時的問題就是*邊界*：即物件進入和離開方法的地點。這些正是編譯器在編譯期執行類型檢查並插入轉型程式碼的地點。

考慮如下這段非泛型範例：

```java
// generics/SimpleHolder.java

public class SimpleHolder {
    private Object obj;
    
    public void set(Object obj) {
        this.obj = obj;
    }
    
    public Object get() {
        return obj;
    }
    
    public static void main(String[] args) {
        SimpleHolder holder = new SimpleHolder();
        holder.set("Item");
        String s = (String) holder.get();
    }
}
```

如果用 **javap -c SimpleHolder** 反編譯這個類，會得到如下內容（經過編輯）：

```java
public void set(java.lang.Object);
   0: aload_0
   1: aload_1
   2: putfield #2; // Field obj:Object;
   5: return
    
public java.lang.Object get();
   0: aload_0
   1: getfield #2; // Field obj:Object;
   4: areturn
    
public static void main(java.lang.String[]);
   0: new #3; // class SimpleHolder
   3: dup
   4: invokespecial #4; // Method "<init>":()V
   7: astore_1
   8: aload_1
   9: ldc #5; // String Item
   11: invokevirtual #6; // Method set:(Object;)V
   14: aload_1
   15: invokevirtual #7; // Method get:()Object;
   18: checkcast #8; // class java/lang/String
   21: astore_2
   22: return
```

`set()` 和 `get()` 方法儲存和產生值，轉型在呼叫 `get()` 時接受檢查。

現在將泛型融入上例程式碼中：

```java
// generics/GenericHolder2.java

public class GenericHolder2<T> {
    private T obj;

    public void set(T obj) {
        this.obj = obj;
    }

    public T get() {
        return obj;
    }

    public static void main(String[] args) {
        GenericHolder2<String> holder =  new GenericHolder2<>();
        holder.set("Item");
        String s = holder.get();
    }
}
```

從 `get()` 返回後的轉型消失了，但是我們還知道傳遞給 `set()` 的值在編譯期會被檢查。下面是相關的位元組碼：

```java
public void set(java.lang.Object);
   0: aload_0
   1: aload_1
   2: putfield #2; // Field obj:Object;
   5: return
       
public java.lang.Object get();
   0: aload_0
   1: getfield #2; // Field obj:Object;
   4: areturn
       
public static void main(java.lang.String[]);
   0: new #3; // class GenericHolder2
   3: dup
   4: invokespecial #4; // Method "<init>":()V
   7: astore_1
   8: aload_1
   9: ldc #5; // String Item
   11: invokevirtual #6; // Method set:(Object;)V
   14: aload_1
   15: invokevirtual #7; // Method get:()Object;
   18: checkcast #8; // class java/lang/String
   21: astore_2
   22: return
```

所產生的位元組碼是相同的。對進入 `set()` 的類型進行檢查是不需要的，因為這將由編譯器執行。而對 `get()` 返回的值進行轉型仍然是需要的，只不過不需要你來操作，它由編譯器自動插入，這樣你就不用編寫（閱讀）雜亂的程式碼。

`get()` 和 `set()` 產生了相同的位元組碼，這就告訴我們泛型的所有動作都發生在邊界處——對入參的編譯器檢查和對返回值的轉型。這有助於澄清對擦除的困惑，記住：“邊界就是動作發生的地方”。

<!-- Compensating for Erasure -->

## 補償擦除

因為擦除，我們將失去執行泛型程式碼中某些操作的能力。無法在執行時知道確切類型：

```java
// generics/Erased.java
// {WillNotCompile}

public class Erased<T> {
    private final int SIZE = 100;

    public void f(Object arg) {
        // error: illegal generic type for instanceof
        if (arg instanceof T) {
        }
        // error: unexpected type
        T var = new T();
        // error: generic array creation
        T[] array = new T[SIZE];
        // warning: [unchecked] unchecked cast
        T[] array = (T[]) new Object[SIZE];

    }
}
```

有時，我們可以對這些問題進行編程，但是有時必須透過引入類型標籤來補償擦除。這意味著為所需的類型顯式傳遞一個 **Class** 物件，以在類型表達式中使用它。

例如，由於擦除了類型訊息，因此在上一個程式中嘗試使用 **instanceof** 將會失敗。類型標籤可以使用動態 `isInstance()` ：

```java
// generics/ClassTypeCapture.java

class Building {
}

class House extends Building {
}

public class ClassTypeCapture<T> {
    Class<T> kind;

    public ClassTypeCapture(Class<T> kind) {
        this.kind = kind;
    }

    public boolean f(Object arg) {
        return kind.isInstance(arg);
    }

    public static void main(String[] args) {
        ClassTypeCapture<Building> ctt1 =
                new ClassTypeCapture<>(Building.class);
        System.out.println(ctt1.f(new Building()));
        System.out.println(ctt1.f(new House()));
        ClassTypeCapture<House> ctt2 =
                new ClassTypeCapture<>(House.class);
        System.out.println(ctt2.f(new Building()));
        System.out.println(ctt2.f(new House()));
    }
}
/* Output:
true
true
false
true
*/
```

編譯器來保證類型標籤與泛型參數相匹配。

<!-- Creating Instances of Types -->
### 建立類型的實例

試圖在 **Erased.java** 中 `new T()` 是行不通的，部分原因是由於擦除，部分原因是編譯器無法驗證 **T** 是否具有預設（無參）建構子。但是在 C++ 中，此操作自然，直接且安全（在編譯時檢查）：

```C++
// generics/InstantiateGenericType.cpp
// C++, not Java!

template<class T> class Foo {
  T x; // Create a field of type T
  T* y; // Pointer to T
public:
  // Initialize the pointer:
  Foo() { y = new T(); }
};

class Bar {};

int main() {
  Foo<Bar> fb;
  Foo<int> fi; // ... and it works with primitives
}
```

Java 中的解決方案是傳入一個工廠物件，並使用該物件建立新實例。方便的工廠物件只是 **Class** 物件，因此，如果使用類型標記，則可以使用 `newInstance()` 建立該類型的新物件：

```java
// generics/InstantiateGenericType.java

import java.util.function.Supplier;

class ClassAsFactory<T> implements Supplier<T> {
    Class<T> kind;

    ClassAsFactory(Class<T> kind) {
        this.kind = kind;
    }

    @Override
    public T get() {
        try {
            return kind.newInstance();
        } catch (InstantiationException |
                IllegalAccessException e) {
            throw new RuntimeException(e);
        }
    }
}

class Employee {
    @Override
    public String toString() {
        return "Employee";
    }
}

public class InstantiateGenericType {
    public static void main(String[] args) {
        ClassAsFactory<Employee> fe =
                new ClassAsFactory<>(Employee.class);
        System.out.println(fe.get());
        ClassAsFactory<Integer> fi =
                new ClassAsFactory<>(Integer.class);
        try {
            System.out.println(fi.get());
        } catch (Exception e) {
            System.out.println(e.getMessage());
        }
    }
}
/* Output:
Employee
java.lang.InstantiationException: java.lang.Integer
*/
```

這樣可以編譯，但對於 `ClassAsFactory<Integer>` 會失敗，這是因為 **Integer** 沒有無參建構子。由於錯誤不是在編譯時捕獲的，因此語言建立者不贊成這種方法。他們建議使用顯式工廠（**Supplier**）並約束類型，以便只有實現該工廠的類可以這樣建立物件。這是建立工廠的兩種不同方法：

```java
// generics/FactoryConstraint.java

import onjava.Suppliers;

import java.util.ArrayList;
import java.util.List;
import java.util.function.Supplier;

class IntegerFactory implements Supplier<Integer> {
    private int i = 0;

    @Override
    public Integer get() {
        return ++i;
    }
}

class Widget {
    private int id;

    Widget(int n) {
        id = n;
    }

    @Override
    public String toString() {
        return "Widget " + id;
    }

    public static
    class Factory implements Supplier<Widget> {
        private int i = 0;

        @Override
        public Widget get() {
            return new Widget(++i);
        }
    }
}

class Fudge {
    private static int count = 1;
    private int n = count++;

    @Override
    public String toString() {
        return "Fudge " + n;
    }
}

class Foo2<T> {
    private List<T> x = new ArrayList<>();

    Foo2(Supplier<T> factory) {
        Suppliers.fill(x, factory, 5);
    }

    @Override
    public String toString() {
        return x.toString();
    }
}

public class FactoryConstraint {
    public static void main(String[] args) {
        System.out.println(
                new Foo2<>(new IntegerFactory()));
        System.out.println(
                new Foo2<>(new Widget.Factory()));
        System.out.println(
                new Foo2<>(Fudge::new));
    }
}
/* Output:
[1, 2, 3, 4, 5]
[Widget 1, Widget 2, Widget 3, Widget 4, Widget 5]
[Fudge 1, Fudge 2, Fudge 3, Fudge 4, Fudge 5]
*/
```

**IntegerFactory** 本身就是透過實現 `Supplier<Integer>` 的工廠。 **Widget** 包含一個內部類，它是一個工廠。還要注意，**Fudge** 並沒有做任何類似於工廠的操作，並且傳遞 `Fudge::new` 仍然會產生工廠行為，因為編譯器將對函數方法 `::new` 的呼叫轉換為對 `get()` 的呼叫。

另一種方法是模板方法設計模式。在以下範例中，`create()` 是模板方法，在子類中被重寫以生成該類型的物件：

```java
// generics/CreatorGeneric.java

abstract class GenericWithCreate<T> {
    final T element;

    GenericWithCreate() {
        element = create();
    }

    abstract T create();
}

class X {
}

class XCreator extends GenericWithCreate<X> {
    @Override
    X create() {
        return new X();
    }

    void f() {
        System.out.println(
                element.getClass().getSimpleName());
    }
}

public class CreatorGeneric {
    public static void main(String[] args) {
        XCreator xc = new XCreator();
        xc.f();
    }
}
/* Output:
X
*/
```

**GenericWithCreate** 包含 `element` 欄位，並透過無參建構子強制其初始化，該建構子又呼叫抽象的 `create()` 方法。這種建立方式可以在子類中定義，同時建立 **T** 的類型。

<!-- Arrays of Generics -->

### 泛型陣列

正如在 **Erased.java** 中所看到的，我們無法建立泛型陣列。通用解決方案是在試圖建立泛型陣列的時候使用 **ArrayList** ：

```java
// generics/ListOfGenerics.java

import java.util.ArrayList;
import java.util.List;

public class ListOfGenerics<T> {
    private List<T> array = new ArrayList<>();

    public void add(T item) {
        array.add(item);
    }

    public T get(int index) {
        return array.get(index);
    }
}
```

這樣做可以獲得陣列的行為，並且還具有泛型提供的編譯時類型安全性。

有時，仍然會建立泛型類型的陣列（例如， **ArrayList** 在內部使用陣列）。可以透過使編譯器滿意的方式定義對陣列的通用引用：

```java
// generics/ArrayOfGenericReference.java

class Generic<T> {
}

public class ArrayOfGenericReference {
    static Generic<Integer>[] gia;
}
```

編譯器接受此操作而不產生警告。但是我們永遠無法建立具有該確切類型（包括類型參數）的陣列，因此有點令人困惑。由於所有陣列，無論它們持有什麼類型，都具有相同的結構（每個陣列插槽的大小和陣列布局），因此似乎可以建立一個 **Object** 陣列並將其轉換為所需的陣列類型。實際上，這確實可以編譯，但是會產生 **ClassCastException** ：

```java
// generics/ArrayOfGeneric.java

public class ArrayOfGeneric {
    static final int SIZE = 100;
    static Generic<Integer>[] gia;

    @SuppressWarnings("unchecked")
    public static void main(String[] args) {
        try {
            gia = (Generic<Integer>[]) new Object[SIZE];
        } catch (ClassCastException e) {
            System.out.println(e.getMessage());
        }
        // Runtime type is the raw (erased) type:
        gia = (Generic<Integer>[]) new Generic[SIZE];
        System.out.println(gia.getClass().getSimpleName());
        gia[0] = new Generic<>();
        //- gia[1] = new Object(); // Compile-time error
        // Discovers type mismatch at compile time:
        //- gia[2] = new Generic<Double>();
    }
}
/* Output:
[Ljava.lang.Object; cannot be cast to [LGeneric;
Generic[]
*/
```

問題在於陣列會跟蹤其實際類型，而該類型是在建立陣列時建立的。因此，即使 `gia` 被強制轉換為 `Generic<Integer>[]` ，該訊息也僅在編譯時存在（並且沒有 **@SuppressWarnings** 註解，將會收到有關該強制轉換的警告）。在執行時，它仍然是一個 **Object** 陣列，這會引起問題。成功建立泛型類型的陣列的唯一方法是建立一個已擦除類型的新陣列，並將其強制轉換。

讓我們看一個更複雜的範例。考慮一個包裝陣列的簡單泛型包裝器：

```java
// generics/GenericArray.java

public class GenericArray<T> {
    private T[] array;

    @SuppressWarnings("unchecked")
    public GenericArray(int sz) {
        array = (T[]) new Object[sz];
    }

    public void put(int index, T item) {
        array[index] = item;
    }

    public T get(int index) {
        return array[index];
    }

    // Method that exposes the underlying representation:
    public T[] rep() {
        return array;
    }

    public static void main(String[] args) {
        GenericArray<Integer> gai = new GenericArray<>(10);
        try {
            Integer[] ia = gai.rep();
        } catch (ClassCastException e) {
            System.out.println(e.getMessage());
        }
        // This is OK:
        Object[] oa = gai.rep();
    }
}
/* Output:
[Ljava.lang.Object; cannot be cast to
[Ljava.lang.Integer;
*/
```

和以前一樣，我們不能說 `T[] array = new T[sz]` ，所以我們建立了一個 **Object** 陣列並將其強制轉換。

`rep()` 方法返回一個 `T[]` ，在主方法中它應該是 `gai` 的 `Integer[]`，但是如果呼叫它並嘗試將結果轉換為 `Integer[]` 引用，則會得到 **ClassCastException** ，這再次是因為實際的執行時類型為 `Object[]` 。

如果再注釋掉 **@SuppressWarnings** 註解後編譯 **GenericArray.java** ，則編譯器會產生警告：

```java
GenericArray.java uses unchecked or unsafe operations.
Recompile with -Xlint:unchecked for details.
```

在這裡，我們收到了一個警告，我們認為這是有關強制轉換的。

但是要真正確定，請使用 `-Xlint：unchecked` 進行編譯：

```java
GenericArray.java:7: warning: [unchecked] unchecked cast    array = (T[])new Object[sz];                 ^  required: T[]  found:    Object[]  where T is a type-variable:    T extends Object declared in class GenericArray 1 warning
```

確實是在抱怨那個強制轉換。由於警告會變成噪音，因此，一旦我們確認預期會出現特定警告，我們可以做的最好的辦法就是使用 **@SuppressWarnings** 將其關閉。這樣，當警告確實出現時，我們將進行實際調查。

由於擦除，陣列的執行時類型只能是 `Object[]` 。 如果我們立即將其轉換為 `T[]` ，則在編譯時會遺失陣列的實際類型，並且編譯器可能會錯過一些潛在的錯誤檢查。因此，最好在集合中使用 `Object[]` ，並在使用陣列元素時向 **T** 添加強制類型轉換。讓我們來看看在 **GenericArray.java** 範例中會是怎麼樣的：

```java
// generics/GenericArray2.java

public class GenericArray2<T> {
    private Object[] array;

    public GenericArray2(int sz) {
        array = new Object[sz];
    }

    public void put(int index, T item) {
        array[index] = item;
    }

    @SuppressWarnings("unchecked")
    public T get(int index) {
        return (T) array[index];
    }

    @SuppressWarnings("unchecked")
    public T[] rep() {
        return (T[]) array; // Unchecked cast
    }

    public static void main(String[] args) {
        GenericArray2<Integer> gai =
                new GenericArray2<>(10);
        for (int i = 0; i < 10; i++)
            gai.put(i, i);
        for (int i = 0; i < 10; i++)
            System.out.print(gai.get(i) + " ");
        System.out.println();
        try {
            Integer[] ia = gai.rep();
        } catch (Exception e) {
            System.out.println(e);
        }
    }
}
/* Output:
0 1 2 3 4 5 6 7 8 9
java.lang.ClassCastException: [Ljava.lang.Object;
cannot be cast to [Ljava.lang.Integer;
*/
```

最初，看起來並沒有太大不同，只是轉換的位置移動了。沒有 **@SuppressWarnings** 註解，仍然會收到“unchecked”警告。但是，內部表示現在是 `Object[]` 而不是 `T[]` 。 呼叫 `get()` 時，它將物件強制轉換為 **T** ，實際上這是正確的類型，因此很安全。但是，如果呼叫 `rep()` ，它將再次嘗試將 `Object[]` 強制轉換為 `T[]` ，但仍然不正確，並在編譯時生成警告，並在執行時生成異常。因此，無法破壞基礎陣列的類型，該基礎陣列只能是 `Object[]` 。在內部將陣列視為 `Object[]` 而不是 `T[]` 的優點是，我們不太可能會忘記陣列的執行時類型並意外地引入了bug，儘管大多數（也許是全部）此類錯誤會在執行時被迅速檢測到。

對於新程式碼，請傳入類型標記。在這種情況下，**GenericArray** 如下所示：

```java
// generics/GenericArrayWithTypeToken.java

import java.lang.reflect.Array;

public class GenericArrayWithTypeToken<T> {
    private T[] array;

    @SuppressWarnings("unchecked")
    public GenericArrayWithTypeToken(Class<T> type, int sz) {
        array = (T[]) Array.newInstance(type, sz);
    }

    public void put(int index, T item) {
        array[index] = item;
    }

    public T get(int index) {
        return array[index];
    }

    // Expose the underlying representation:
    public T[] rep() {
        return array;
    }

    public static void main(String[] args) {
        GenericArrayWithTypeToken<Integer> gai =
                new GenericArrayWithTypeToken<>(
                        Integer.class, 10);
        // This now works:
        Integer[] ia = gai.rep();
    }
}
```

類型標記 **Class\<T\>** 被傳遞到建構子中以從擦除中復原，因此儘管必須使用 **@SuppressWarnings** 關閉來自強制類型轉換的警告，但我們仍可以建立所需的實際陣列類型。一旦獲得了實際的類型，就可以返回它並產生所需的結果，如在主方法中看到的那樣。陣列的執行時類型是確切的類型 `T[]` 。

不幸的是，如果查看 Java 標準庫中的原始碼，你會發現到處都有從 **Object** 陣列到參數化類型的轉換。例如，這是**ArrayList** 中，複製一個 **Collection** 的建構子，這裡為了簡化，去除了原始碼中對此不重要的程式碼：

```java
public ArrayList(Collection c) {
  size = c.size();
  elementData = (E[])new Object[size];
  c.toArray(elementData);
}
```

如果你瀏覽 **ArrayList.java** 的程式碼，將會發現很多此類強制轉換。當我們編譯它時會發生什麼事？

```java
Note: ArrayList.java uses unchecked or unsafe operations
Note: Recompile with -Xlint:unchecked for details.
```

果然，標準庫會產生很多警告。如果你使用過 C 語言，尤其是使用 ANSI C 之前的語言，你會記住警告的特殊效果：發現警告後，可以忽略它們。因此，除非程式設計師必須對其進行處理，否則最好不要從編譯器發出任何類型的消息。

Neal Gafter（Java 5 的主要開發人員之一）在他的部落格中[^2]指出，他在重寫 Java 庫時是很隨意、馬虎的，我們不應該像他那樣做。Neal 還指出，他在不破壞現有介面的情況下無法修復某些 Java 庫程式碼。因此，即使在 Java 庫原始碼中出現了一些習慣用法，它們也不一定是正確的做法。當查看庫程式碼時，我們不能認為這就是要在自己程式碼中必須遵循的範例。

請注意，在 Java 文獻中推薦使用類型標記技術，例如 Gilad Bracha 的論文《Generics in the Java Programming Language》[^3]，他指出：“例如，這種用法已廣泛用於新的 API 中以處理註解。” 我發現此技術在人們對於舒適度的看法方面存在一些不一致之處；有些人強烈喜歡本章前面介紹的工廠方法。

<!-- Bounds -->

## 邊界

*邊界*（bounds）在本章的前面進行了簡要介紹。邊界允許我們對泛型使用的參數類型施加約束。儘管這可以強制執行有關應用了泛型類型的規則，但潛在的更重要的效果是我們可以在綁定的類型中呼叫方法。

由於擦除會刪除類型訊息，因此唯一可用於無限制泛型參數的方法是那些 **Object** 可用的方法。但是，如果將該參數限制為某類型的子集，則可以呼叫該子集中的方法。為了應用約束，Java 泛型使用了 `extends` 關鍵字。

重要的是要理解，當用於限定泛型類型時，`extends` 的含義與通常的意義截然不同。此範例展示邊界的基礎應用：

```java
// generics/BasicBounds.java

interface HasColor {
    java.awt.Color getColor();
}

class WithColor<T extends HasColor> {
    T item;

    WithColor(T item) {
        this.item = item;
    }

    T getItem() {
        return item;
    }

    // The bound allows you to call a method:
    java.awt.Color color() {
        return item.getColor();
    }
}

class Coord {
    public int x, y, z;
}

// This fails. Class must be first, then interfaces:
// class WithColorCoord<T extends HasColor & Coord> {

// Multiple bounds:
class WithColorCoord<T extends Coord & HasColor> {
    T item;

    WithColorCoord(T item) {
        this.item = item;
    }

    T getItem() {
        return item;
    }

    java.awt.Color color() {
        return item.getColor();
    }

    int getX() {
        return item.x;
    }

    int getY() {
        return item.y;
    }

    int getZ() {
        return item.z;
    }
}

interface Weight {
    int weight();
}

// As with inheritance, you can have only one
// concrete class but multiple interfaces:
class Solid<T extends Coord & HasColor & Weight> {
    T item;

    Solid(T item) {
        this.item = item;
    }

    T getItem() {
        return item;
    }

    java.awt.Color color() {
        return item.getColor();
    }

    int getX() {
        return item.x;
    }

    int getY() {
        return item.y;
    }

    int getZ() {
        return item.z;
    }

    int weight() {
        return item.weight();
    }
}

class Bounded
        extends Coord implements HasColor, Weight {
    @Override
    public java.awt.Color getColor() {
        return null;
    }

    @Override
    public int weight() {
        return 0;
    }
}

public class BasicBounds {
    public static void main(String[] args) {
        Solid<Bounded> solid =
                new Solid<>(new Bounded());
        solid.color();
        solid.getY();
        solid.weight();
    }
}
```

你可能會觀察到 **BasicBounds.java** 中似乎包含一些冗餘，它們可以透過繼承來消除。在這裡，每個繼承級別還添加了邊界約束：

```java
// generics/InheritBounds.java

class HoldItem<T> {
    T item;

    HoldItem(T item) {
        this.item = item;
    }

    T getItem() {
        return item;
    }
}

class WithColor2<T extends HasColor>
        extends HoldItem<T> {
    WithColor2(T item) {
        super(item);
    }

    java.awt.Color color() {
        return item.getColor();
    }
}

class WithColorCoord2<T extends Coord & HasColor>
        extends WithColor2<T> {
    WithColorCoord2(T item) {
        super(item);
    }

    int getX() {
        return item.x;
    }

    int getY() {
        return item.y;
    }

    int getZ() {
        return item.z;
    }
}

class Solid2<T extends Coord & HasColor & Weight>
        extends WithColorCoord2<T> {
    Solid2(T item) {
        super(item);
    }

    int weight() {
        return item.weight();
    }
}

public class InheritBounds {
    public static void main(String[] args) {
        Solid2<Bounded> solid2 =
                new Solid2<>(new Bounded());
        solid2.color();
        solid2.getY();
        solid2.weight();
    }
}
```

**HoldItem** 擁有一個物件，因此此行為將繼承到 **WithColor2** 中，這也需要其參數符合 **HasColor**。 **WithColorCoord2** 和 **Solid2** 進一步擴展了層次結構，並在每個級別添加了邊界。現在，這些方法已被繼承，並且在每個類中不再重複。

這是一個具有更多層次的範例：

```java
// generics/EpicBattle.java
// Bounds in Java generics

import java.util.List;

interface SuperPower {
}

interface XRayVision extends SuperPower {
    void seeThroughWalls();
}

interface SuperHearing extends SuperPower {
    void hearSubtleNoises();
}

interface SuperSmell extends SuperPower {
    void trackBySmell();
}

class SuperHero<POWER extends SuperPower> {
    POWER power;

    SuperHero(POWER power) {
        this.power = power;
    }

    POWER getPower() {
        return power;
    }
}

class SuperSleuth<POWER extends XRayVision>
        extends SuperHero<POWER> {
    SuperSleuth(POWER power) {
        super(power);
    }

    void see() {
        power.seeThroughWalls();
    }
}

class
CanineHero<POWER extends SuperHearing & SuperSmell>
        extends SuperHero<POWER> {
    CanineHero(POWER power) {
        super(power);
    }

    void hear() {
        power.hearSubtleNoises();
    }

    void smell() {
        power.trackBySmell();
    }
}

class SuperHearSmell
        implements SuperHearing, SuperSmell {
    @Override
    public void hearSubtleNoises() {
    }

    @Override
    public void trackBySmell() {
    }
}

class DogPerson extends CanineHero<SuperHearSmell> {
    DogPerson() {
        super(new SuperHearSmell());
    }
}

public class EpicBattle {
    // Bounds in generic methods:
    static <POWER extends SuperHearing>
    void useSuperHearing(SuperHero<POWER> hero) {
        hero.getPower().hearSubtleNoises();
    }

    static <POWER extends SuperHearing & SuperSmell>
    void superFind(SuperHero<POWER> hero) {
        hero.getPower().hearSubtleNoises();
        hero.getPower().trackBySmell();
    }

    public static void main(String[] args) {
        DogPerson dogPerson = new DogPerson();
        useSuperHearing(dogPerson);
        superFind(dogPerson);
        // You can do this:
        List<? extends SuperHearing> audioPeople;
        // But you can't do this:
        // List<? extends SuperHearing & SuperSmell> dogPs;
    }
}
```

接下來將要研究的萬用字元將會把範圍限制在單個類型。

<!-- Wildcards -->

## 萬用字元

你已經在 [集合](book/12-Collections.md) 章節中看到了一些簡單範例使用了萬用字元——在泛型參數表達式中的問號，在 [類型訊息](book/19-Type-Information.md) 一章中這種範例更多。本節將更深入地探討這個特性。

我們的起始範例要展示陣列的一種特殊行為：你可以將衍生類的陣列賦值給基類的引用：

```java
// generics/CovariantArrays.java

class Fruit {}

class Apple extends Fruit {}

class Jonathan extends Apple {}

class Orange extends Fruit {}

public class CovariantArrays {
    
    public static void main(String[] args) {
        Fruit[] fruit = new Apple[10];
        fruit[0] = new Apple(); // OK
        fruit[1] = new Jonathan(); // OK
        // Runtime type is Apple[], not Fruit[] or Orange[]:
        try {
            // Compiler allows you to add Fruit:
            fruit[0] = new Fruit(); // ArrayStoreException
        } catch (Exception e) {
            System.out.println(e);
        }
        try {
            // Compiler allows you to add Oranges:
            fruit[0] = new Orange(); // ArrayStoreException
        } catch (Exception e) {
            System.out.println(e);
        }
    }
}
/* Output:
java.lang.ArrayStoreException: Fruit
java.lang.ArrayStoreException: Orange
```

`main()` 中的第一行建立了 **Apple** 陣列，並賦值給一個 **Fruit** 陣列引用。這是有意義的，因為 **Apple** 也是一種 **Fruit**，因此 **Apple** 陣列應該也是一個 **Fruit** 陣列。

但是，如果實際的陣列類型是 **Apple[]**，你可以在其中放置 **Apple** 或 **Apple** 的子類型，這在編譯期和執行時都可以工作。但是你也可以在陣列中放置 **Fruit** 物件。這對編譯器來說是有意義的，因為它有一個 **Fruit[]** 引用——它有什麼理由不允許將 **Fruit** 物件或任何從 **Fruit** 繼承出來的物件（比如 **Orange**），放置到這個陣列中呢？因此在編譯期，這是允許的。然而，執行時的陣列機制知道它處理的是 **Apple[]**，因此會在向陣列中放置異構類型時拋出異常。

向上轉型用在這裡不合適。你真正在做的是將一個陣列賦值給另一個陣列。陣列的行為是持有其他物件，這裡只是因為我們能夠向上轉型而已，所以很明顯，陣列物件可以保留有關它們包含的物件類型的規則。看起來就像陣列對它們持有的物件是有意識的，因此在編譯期檢查和執行時檢查之間，你不能濫用它們。

陣列的這種賦值並不是那麼可怕，因為在執行時你可以發現插入了錯誤的類型。但是泛型的主要目標之一是將這種錯誤檢測移到編譯期。所以當我們試圖使用泛型集合代替陣列時，會發生什麼事呢？

```java
// generics/NonCovariantGenerics.java
// {WillNotCompile}

import java.util.*;

public class NonCovariantGenerics {
    // Compile Error: incompatible types:
    List<Fruit> flist = new ArrayList<Apple>();
}
```

儘管你在首次閱讀這段程式碼時會認為“不能將一個 **Apple** 集合賦值給一個 **Fruit** 集合”。記住，泛型不僅僅是關於集合，它真正要表達的是“不能把一個涉及 **Apple** 的泛型賦值給一個涉及 **Fruit** 的泛型”。如果像在陣列中的情況一樣，編譯器對程式碼的了解足夠多，可以確定所涉及到的集合，那麼它可能會留下一些餘地。但是它不知道任何有關這方面的訊息，因此它拒絕向上轉型。然而實際上這也不是向上轉型—— **Apple** 的 **List** 不是 **Fruit** 的 **List**。**Apple** 的 **List** 將持有 **Apple** 和 **Apple** 的子類型，**Fruit** 的 **List** 將持有任何類型的 **Fruit**。是的，這包括 **Apple**，但是它不是一個 **Apple** 的 **List**，它仍然是 **Fruit** 的 **List**。**Apple** 的 **List** 在類型上不等價於 **Fruit** 的 **List**，即使 **Apple** 是一種 **Fruit** 類型。

真正的問題是我們在討論的集合類型，而不是集合持有物件的類型。與陣列不同，泛型沒有內建的協變類型。這是因為陣列是完全在語言中定義的，因此可以具有編譯期和執行時的內建檢查，但是在使用泛型時，編譯器和執行時系統不知道你想用類型做什麼，以及應該採用什麼規則。

但是，有時你想在兩個類型間建立某種向上轉型關係。萬用字元可以產生這種關係。

```java
// generics/GenericsAndCovariance.java

import java.util.*;

public class GenericsAndCovariance {
    
    public static void main(String[] args) {
        // Wildcards allow covariance:
        List<? extends Fruit> flist = new ArrayList<>();
        // Compile Error: can't add any type of object:
        // flist.add(new Apple());
        // flist.add(new Fruit());
        // flist.add(new Object());
        flist.add(null); // Legal but uninteresting
        // We know it returns at least Fruit:
        Fruit f = flist.get(0);
    }
    
}
```

**flist** 的類型現在是 `List<? extends Fruit>`，你可以讀作“一個具有任何從 **Fruit** 繼承的類型的列表”。然而，這實際上並不意味著這個 **List** 將持有任何類型的 **Fruit**。萬用字元引用的是明確的類型，因此它意味著“某種 **flist** 引用沒有指定的具體類型”。因此這個被賦值的 **List** 必須持有諸如 **Fruit** 或 **Apple** 這樣的指定類型，但是為了向上轉型為 **Fruit**，這個類型是什麼沒人在意。

**List** 必須持有一種具體的 **Fruit** 或 **Fruit** 的子類型，但是如果你不關心具體的類型是什麼，那麼你能對這樣的 **List** 做什麼呢？如果不知道 **List** 中持有的物件是什麼類型，你怎能保證安全地向其中添加物件呢？就像在 **CovariantArrays.java** 中向上轉型一樣，你不能，除非編譯器而不是執行時系統可以阻止這種操作的發生。你很快就會發現這個問題。

你可能認為事情開始變得有點走極端了，因為現在你甚至不能向剛剛聲明過將持有 **Apple** 物件的 **List** 中放入一個 **Apple** 物件。是的，但編譯器並不知道這一點。`List<? extends Fruit>` 可能合法地指向一個 `List<Orange>`。一旦執行這種類型的向上轉型，你就遺失了向其中傳遞任何物件的能力，甚至傳遞 **Object** 也不行。

另一方面，如果你呼叫了一個返回 **Fruit** 的方法，則是安全的，因為你知道這個 **List** 中的任何物件至少具有 **Fruit** 類型，因此編譯器允許這麼做。

### 編譯器有多聰明

現在你可能會猜想自己不能去呼叫任何接受參數的方法，但是考慮下面的程式碼：

```java
// generics/CompilerIntelligence.java

import java.util.*;

public class CompilerIntelligence {
    
    public static void main(String[] args) {
        List<? extends Fruit> flist = Arrays.asList(new Apple());
        Apple a = (Apple) flist.get(0); // No warning
        flist.contains(new Apple()); // Argument is 'Object'
        flist.indexOf(new Apple()); // Argument is 'Object'
    }
    
}
```

這裡對 `contains()` 和 `indexOf()` 的呼叫接受 **Apple** 物件作為參數，執行沒問題。這是否意味著編譯器實際上會檢查程式碼，以查看是否有某個特定的方法修改了它的物件？

透過查看 **ArrayList** 的文件，我們發現編譯器沒有那麼聰明。儘管 `add()` 接受一個泛型參數類型的參數，但 `contains()` 和 `indexOf()` 接受的參數類型是 **Object**。因此當你指定一個 `ArrayList<? extends Fruit>` 時，`add()` 的參數就變成了"**? extends Fruit**"。從這個描述中，編譯器無法得知這裡需要 **Fruit** 的哪個具體子類型，因此它不會接受任何類型的 **Fruit**。如果你先把 **Apple** 向上轉型為 **Fruit**，也沒有關係——編譯器僅僅會拒絕呼叫像 `add()` 這樣參數列表中涉及萬用字元的方法。

`contains()` 和 `indexOf()` 的參數類型是 **Object**，不涉及萬用字元，所以編譯器允許呼叫它們。這意味著將由泛型類的設計者來決定哪些呼叫是“安全的”，並使用 **Object** 類作為它們的參數類型。為了禁止對類型中使用了萬用字元的方法呼叫，需要在參數列表中使用類型參數。

下面展示一個簡單的 **Holder** 類：

```java
// generics/Holder.java

public class Holder<T> {

    private T value;

    public Holder() {}

    public Holder(T val) {
        value = val;
    }

    public void set(T val) {
        value = val;
    }

    public T get() {
        return value;
    }

    @Override
    public boolean equals(Object o) {
        return o instanceof Holder && Objects.equals(value, ((Holder) o).value);
    }

    @Override
    public int hashCode() {
        return Objects.hashCode(value);
    }

    public static void main(String[] args) {
        Holder<Apple> apple = new Holder<>(new Apple());
        Apple d = apple.get();
        apple.set(d);
        // Holder<Fruit> fruit = apple; // Cannot upcast
        Holder<? extends Fruit> fruit = apple; // OK
        Fruit p = fruit.get();
        d = (Apple) fruit.get();
        try {
            Orange c = (Orange) fruit.get(); // No warning
        } catch (Exception e) {
            System.out.println(e);
        }
        // fruit.set(new Apple()); // Cannot call set()
        // fruit.set(new Fruit()); // Cannot call set()
        System.out.println(fruit.equals(d)); // OK
    }
}
/* Output
java.lang.ClassCastException: Apple cannot be cast to Orange
false
*/
```

**Holder** 有一個接受 **T** 類型物件的 `set()` 方法，一個返回 T 物件的 `get()` 方法和一個接受 Object 物件的 `equals()` 方法。正如你所見，如果建立了一個 `Holder<Apple>`，就不能將其向上轉型為 `Holder<Fruit>`，但是可以向上轉型為 `Holder<? extends Fruit>`。如果呼叫 `get()`，只能返回一個 **Fruit**——這就是在給定“任何擴展自 **Fruit** 的物件”這一邊界後，它所能知道的一切了。如果你知道更多的訊息，就可以將其轉型到某種具體的 **Fruit** 而不會導致任何警告，但是存在得到 **ClassCastException** 的風險。`set()` 方法不能工作在 **Apple** 和 **Fruit** 上，因為 `set()` 的參數也是"**? extends Fruit**"，意味著它可以是任何事物，編譯器無法驗證“任何事物”的類型安全性。

但是，`equals()` 方法可以正常工作，因為它接受的參數是 **Object** 而不是 **T** 類型。因此，編譯器只關注傳遞進來和要返回的物件類型。它不會分析程式碼，以查看是否執行了任何實際的寫入和讀取操作。

Java 7 引入了 **java.util.Objects** 庫，使建立 `equals()` 和 `hashCode()` 方法變得更加容易，當然還有很多其他功能。`equals()` 方法的標準形式參考 [附錄：理解 equals 和 hashCode 方法](book/Appendix-Understanding-equals-and-hashCode) 一章。

### 逆變

還可以走另外一條路，即使用超類型萬用字元。這裡，可以聲明萬用字元是由某個特定類的任何基類來界定的，方法是指定 `<？super MyClass>` ，或者甚至使用類型參數： `<？super T>`（儘管你不能對泛型參數給出一個超類型邊界；即不能聲明 `<T super MyClass>` ）。這使得你可以安全地傳遞一個類型物件到泛型類型中。因此，有了超類型萬用字元，就可以向 **Collection** 寫入了：

```java
// generics/SuperTypeWildcards.java
import java.util.*;
public class SuperTypeWildcards {
    static void writeTo(List<? super Apple> apples) {
        apples.add(new Apple());
        apples.add(new Jonathan());
        // apples.add(new Fruit()); // Error
    }
}
```

參數 **apples** 是 **Apple** 的某種基類型的 **List**，這樣你就知道向其中添加 **Apple** 或 **Apple** 的子類型是安全的。但是因為 **Apple** 是下界，所以你知道向這樣的 **List** 中添加 **Fruit** 是不安全的，因為這將使這個 **List** 敞開口子，從而可以向其中添加非 **Apple** 類型的物件，而這是違反靜態類型安全的。
下面的範例複習了一下逆變和萬用字元的的使用：

```java
// generics/GenericReading.java
import java.util.*;

public class GenericReading {
    static List<Apple> apples = Arrays.asList(new Apple());
    static List<Fruit> fruit = Arrays.asList(new Fruit());
    
    static <T> T readExact(List<T> list) {
        return list.get(0);
    }
    
    // A static method adapts to each call:
    static void f1() {
        Apple a = readExact(apples);
        Fruit f = readExact(fruit);
        f = readExact(apples);
    }
    
    // A class type is established
    // when the class is instantiated:
    static class Reader<T> {
        T readExact(List<T> list) { 
            return list.get(0); 
        }
    }
    
    static void f2() {
        Reader<Fruit> fruitReader = new Reader<>();
        Fruit f = fruitReader.readExact(fruit);
        //- Fruit a = fruitReader.readExact(apples);
        // error: incompatible types: List<Apple>
        // cannot be converted to List<Fruit>
    }
    
    static class CovariantReader<T> {
        T readCovariant(List<? extends T> list) {
            return list.get(0);
        }
    }
    
    static void f3() {
        CovariantReader<Fruit> fruitReader = new CovariantReader<>();
        Fruit f = fruitReader.readCovariant(fruit);
        Fruit a = fruitReader.readCovariant(apples);
    }
    
    public static void main(String[] args) {
        f1(); 
        f2(); 
        f3();
    }
}
```

`readExact()` 方法使用了精確的類型。如果使用這個沒有任何萬用字元的精確類型，就可以向 **List** 中寫入和讀取這個精確類型。另外，對於返回值，靜態的泛型方法 `readExact()` 可以有效地“適應”每個方法呼叫，並能夠從 `List<Apple>` 中返回一個 **Apple** ，從 `List<Fruit>` 中返回一個 **Fruit** ，就像在 `f1()` 中看到的那樣。因此，如果可以擺脫靜態泛型方法，那麼在讀取時就不需要協變類型了。
然而對於泛型類來說，當你建立這個類的實例時，就要為這個類確定參數。就像在 `f2()` 中看到的，**fruitReader** 實例可以從 `List<Fruit>` 中讀取一個 **Fruit** ，因為這就是它的確切類型。但是 `List<Apple>` 也應該產生 **Fruit** 物件，而 **fruitReader** 不允許這麼做。
為了修正這個問題，`CovariantReader.readCovariant()` 方法將接受 `List<？extends T>` ，因此，從這個列表中讀取一個 **T** 是安全的（你知道在這個列表中的所有物件至少是一個 **T** ，並且可能是從 T 匯出的某種物件）。在 `f3()` 中，你可以看到現在可以從 `List<Apple>` 中讀取 **Fruit** 了。

### 無界萬用字元

無界萬用字元 `<?>` 看起來意味著“任何事物”，因此使用無界萬用字元好像等價於使用原生類型。事實上，編譯器初看起來是支援這種判斷的：

```java
// generics/UnboundedWildcards1.java
import java.util.*;

public class UnboundedWildcards1 {
    static List list1;
    static List<?> list2;
    static List<? extends Object> list3;
  
    static void assign1(List list) {
        list1 = list;
        list2 = list;
        //- list3 = list;
        // warning: [unchecked] unchecked conversion
        // list3 = list;
        //         ^
        // required: List<? extends Object>
        // found:    List
    }
    
    static void assign2(List<?> list) {
        list1 = list;
        list2 = list;
        list3 = list;
    }
    
    static void assign3(List<? extends Object> list) {
        list1 = list;
        list2 = list;
        list3 = list;
    }
    
    public static void main(String[] args) {
        assign1(new ArrayList());
        assign2(new ArrayList());
        //- assign3(new ArrayList());
        // warning: [unchecked] unchecked method invocation:
        // method assign3 in class UnboundedWildcards1
        // is applied to given types
        // assign3(new ArrayList());
        //        ^
        // required: List<? extends Object>
        // found: ArrayList
        // warning: [unchecked] unchecked conversion
        // assign3(new ArrayList());
        //         ^
        // required: List<? extends Object>
        // found:    ArrayList
        // 2 warnings
        assign1(new ArrayList<>());
        assign2(new ArrayList<>());
        assign3(new ArrayList<>());
        // Both forms are acceptable as List<?>:
        List<?> wildList = new ArrayList();
        wildList = new ArrayList<>();
        assign1(wildList);
        assign2(wildList);
        assign3(wildList);
    }
}
```

有很多情況都和你在這裡看到的情況類似，即編譯器很少關心使用的是原生類型還是 `<?>` 。在這些情況中，`<?>` 可以被認為是一種裝飾，但是它仍舊是很有價值的，因為，實際上它是在聲明：“我是想用 Java 的泛型來編寫這段程式碼，我在這裡並不是要用原生類型，但是在目前這種情況下，泛型參數可以持有任何類型。”
第二個範例展示了無界萬用字元的一個重要應用。當你在處理多個泛型參數時，有時允許一個參數可以是任何類型，同時為其他參數確定某種特定類型的這種能力會顯得很重要：

```java
// generics/UnboundedWildcards2.java
import java.util.*;

public class UnboundedWildcards2 {
    static Map map1;
    static Map<?,?> map2;
    static Map<String,?> map3;
  
    static void assign1(Map map) { 
        map1 = map; 
    }
    
    static void assign2(Map<?,?> map) { 
        map2 = map; 
    }
    
    static void assign3(Map<String,?> map) { 
        map3 = map; 
    }
    
    public static void main(String[] args) {
        assign1(new HashMap());
        assign2(new HashMap());
        //- assign3(new HashMap());
        // warning: [unchecked] unchecked method invocation:
        // method assign3 in class UnboundedWildcards2
        // is applied to given types
        //     assign3(new HashMap());
        //            ^
        //   required: Map<String,?>
        //   found: HashMap
        // warning: [unchecked] unchecked conversion
        //     assign3(new HashMap());
        //             ^
        //   required: Map<String,?>
        //   found:    HashMap
        // 2 warnings
        assign1(new HashMap<>());
        assign2(new HashMap<>());
        assign3(new HashMap<>());
    }
}
```

但是，當你擁有的全都是無界萬用字元時，就像在 `Map<?,?>` 中看到的那樣，編譯器看起來就無法將其與原生 **Map** 區分開了。另外， **UnboundedWildcards1.java** 展示了編譯器處理  `List<?>` 和 `List<? extends Object>` 是不同的。
令人困惑的是，編譯器並非總是關注像 `List` 和 `List<?>` 之間的這種差異，因此它們看起來就像是相同的事物。事實上，因為泛型參數擦除到它的第一個邊界，因此 `List<?>` 看起來等價於 `List<Object>` ，而 **List** 實際上也是 `List<Object>` ——除非這些語句都不為真。**List** 實際上表示“持有任何 **Object** 類型的原生 **List** ”，而 `List<?>` 表示“具有某種特定類型的非原生 **List** ，只是我們不知道類型是什麼。”
編譯器何時才會關注原生類型和涉及無界萬用字元的類型之間的差異呢？下面的範例使用了前面定義的 `Holder<T>` 類，它包含接受 **Holder** 作為參數的各種方法，但是它們具有不同的形式：作為原生類型，具有具體的類型參數以及具有無界萬用字元參數：

```java
// generics/Wildcards.java
// Exploring the meaning of wildcards

public class Wildcards {
    // Raw argument:
    static void rawArgs(Holder holder, Object arg) {
        //- holder.set(arg);
        // warning: [unchecked] unchecked call to set(T)
        // as a member of the raw type Holder
        //     holder.set(arg);
        //               ^
        //   where T is a type-variable:
        //     T extends Object declared in class Holder
        // 1 warning

        // Can't do this; don't have any 'T':
        // T t = holder.get();

        // OK, but type information is lost:
        Object obj = holder.get();
    }
    
    // Like rawArgs(), but errors instead of warnings:
    static void unboundedArg(Holder<?> holder, Object arg) {
        //- holder.set(arg);
        // error: method set in class Holder<T>
        // cannot be applied to given types;
        //     holder.set(arg);
        //           ^
        //   required: CAP#1
        //   found: Object
        //   reason: argument mismatch;
        //     Object cannot be converted to CAP#1
        //   where T is a type-variable:
        //     T extends Object declared in class Holder
        //   where CAP#1 is a fresh type-variable:
        //     CAP#1 extends Object from capture of ?
        // 1 error

        // Can't do this; don't have any 'T':
        // T t = holder.get();

        // OK, but type information is lost:
        Object obj = holder.get();
    }
    
    static <T> T exact1(Holder<T> holder) {
        return holder.get();
    }
    
    static <T> T exact2(Holder<T> holder, T arg) {
        holder.set(arg);
        return holder.get();
    }
    
    static <T> T wildSubtype(Holder<? extends T> holder, T arg) {
        //- holder.set(arg);
        // error: method set in class Holder<T#2>
        // cannot be applied to given types;
        //     holder.set(arg);
        //           ^
        //   required: CAP#1
        //   found: T#1
        //   reason: argument mismatch;
        //     T#1 cannot be converted to CAP#1
        //   where T#1,T#2 are type-variables:
        //     T#1 extends Object declared in method
        //     <T#1>wildSubtype(Holder<? extends T#1>,T#1)
        //     T#2 extends Object declared in class Holder
        //   where CAP#1 is a fresh type-variable:
        //     CAP#1 extends T#1 from
        //       capture of ? extends T#1
        // 1 error
        return holder.get();
    }
    
    static <T> void wildSupertype(Holder<? super T> holder, T arg) {
        holder.set(arg);
        //- T t = holder.get();
        // error: incompatible types:
        // CAP#1 cannot be converted to T
        //     T t = holder.get();
        //                     ^
        //   where T is a type-variable:
        //     T extends Object declared in method
        //       <T>wildSupertype(Holder<? super T>,T)
        //   where CAP#1 is a fresh type-variable:
        //     CAP#1 extends Object super:
        //       T from capture of ? super T
        // 1 error

        // OK, but type information is lost:
        Object obj = holder.get();
    }
    
    public static void main(String[] args) {
        Holder raw = new Holder<>();
        // Or:
        raw = new Holder();
        Holder<Long> qualified = new Holder<>();
        Holder<?> unbounded = new Holder<>();
        Holder<? extends Long> bounded = new Holder<>();
        Long lng = 1L;

        rawArgs(raw, lng);
        rawArgs(qualified, lng);
        rawArgs(unbounded, lng);
        rawArgs(bounded, lng);

        unboundedArg(raw, lng);
        unboundedArg(qualified, lng);
        unboundedArg(unbounded, lng);
        unboundedArg(bounded, lng);

        //- Object r1 = exact1(raw);
        // warning: [unchecked] unchecked method invocation:
        // method exact1 in class Wildcards is applied
        // to given types
        //      Object r1 = exact1(raw);
        //                        ^
        //   required: Holder<T>
        //   found: Holder
        //   where T is a type-variable:
        //     T extends Object declared in
        //     method <T>exact1(Holder<T>)
        // warning: [unchecked] unchecked conversion
        //      Object r1 = exact1(raw);
        //                         ^
        //   required: Holder<T>
        //   found:    Holder
        //   where T is a type-variable:
        //     T extends Object declared in
        //     method <T>exact1(Holder<T>)
        // 2 warnings

        Long r2 = exact1(qualified);
        Object r3 = exact1(unbounded); // Must return Object
        Long r4 = exact1(bounded);

        //- Long r5 = exact2(raw, lng);
        // warning: [unchecked] unchecked method invocation:
        // method exact2 in class Wildcards is
        // applied to given types
        //     Long r5 = exact2(raw, lng);
        //                     ^
        //   required: Holder<T>,T
        //   found: Holder,Long
        //   where T is a type-variable:
        //     T extends Object declared in
        //       method <T>exact2(Holder<T>,T)
        // warning: [unchecked] unchecked conversion
        //     Long r5 = exact2(raw, lng);
        //                      ^
        //   required: Holder<T>
        //   found:    Holder
        //   where T is a type-variable:
        //     T extends Object declared in
        //       method <T>exact2(Holder<T>,T)
        // 2 warnings

        Long r6 = exact2(qualified, lng);

        //- Long r7 = exact2(unbounded, lng);
        // error: method exact2 in class Wildcards
        // cannot be applied to given types;
        //     Long r7 = exact2(unbounded, lng);
        //               ^
        //   required: Holder<T>,T
        //   found: Holder<CAP#1>,Long
        //   reason: inference variable T has
        //     incompatible bounds
        //     equality constraints: CAP#1
        //     lower bounds: Long
        //   where T is a type-variable:
        //     T extends Object declared in
        //       method <T>exact2(Holder<T>,T)
        //   where CAP#1 is a fresh type-variable:
        //     CAP#1 extends Object from capture of ?
        // 1 error

        //- Long r8 = exact2(bounded, lng);
        // error: method exact2 in class Wildcards
        // cannot be applied to given types;
        //      Long r8 = exact2(bounded, lng);
        //                ^
        //   required: Holder<T>,T
        //   found: Holder<CAP#1>,Long
        //   reason: inference variable T
        //     has incompatible bounds
        //     equality constraints: CAP#1
        //     lower bounds: Long
        //   where T is a type-variable:
        //     T extends Object declared in
        //       method <T>exact2(Holder<T>,T)
        //   where CAP#1 is a fresh type-variable:
        //     CAP#1 extends Long from
        //       capture of ? extends Long
        // 1 error

        //- Long r9 = wildSubtype(raw, lng);
        // warning: [unchecked] unchecked method invocation:
        // method wildSubtype in class Wildcards
        // is applied to given types
        //     Long r9 = wildSubtype(raw, lng);
        //                          ^
        //   required: Holder<? extends T>,T
        //   found: Holder,Long
        //   where T is a type-variable:
        //     T extends Object declared in
        //     method <T>wildSubtype(Holder<? extends T>,T)
        // warning: [unchecked] unchecked conversion
        //     Long r9 = wildSubtype(raw, lng);
        //                           ^
        //   required: Holder<? extends T>
        //   found:    Holder
        //   where T is a type-variable:
        //     T extends Object declared in
        //     method <T>wildSubtype(Holder<? extends T>,T)
        // 2 warnings

        Long r10 = wildSubtype(qualified, lng);
        // OK, but can only return Object:
        Object r11 = wildSubtype(unbounded, lng);
        Long r12 = wildSubtype(bounded, lng);

        //- wildSupertype(raw, lng);
        // warning: [unchecked] unchecked method invocation:
        //   method wildSupertype in class Wildcards
        //   is applied to given types
        //     wildSupertype(raw, lng);
        //                  ^
        //   required: Holder<? super T>,T
        //   found: Holder,Long
        //   where T is a type-variable:
        //     T extends Object declared in
        //     method <T>wildSupertype(Holder<? super T>,T)
        // warning: [unchecked] unchecked conversion
        //     wildSupertype(raw, lng);
        //                   ^
        //   required: Holder<? super T>
        //   found:    Holder
        //   where T is a type-variable:
        //     T extends Object declared in
        //     method <T>wildSupertype(Holder<? super T>,T)
        // 2 warnings

        wildSupertype(qualified, lng);

        //- wildSupertype(unbounded, lng);
        // error: method wildSupertype in class Wildcards
        // cannot be applied to given types;
        //     wildSupertype(unbounded, lng);
        //     ^
        //   required: Holder<? super T>,T
        //   found: Holder<CAP#1>,Long
        //   reason: cannot infer type-variable(s) T
        //     (argument mismatch; Holder<CAP#1>
        //     cannot be converted to Holder<? super T>)
        //   where T is a type-variable:
        //     T extends Object declared in
        //     method <T>wildSupertype(Holder<? super T>,T)
        //   where CAP#1 is a fresh type-variable:
        //     CAP#1 extends Object from capture of ?
        // 1 error

        //- wildSupertype(bounded, lng);
        // error: method wildSupertype in class Wildcards
        // cannot be applied to given types;
        //     wildSupertype(bounded, lng);
        //     ^
        //   required: Holder<? super T>,T
        //   found: Holder<CAP#1>,Long
        //   reason: cannot infer type-variable(s) T
        //     (argument mismatch; Holder<CAP#1>
        //     cannot be converted to Holder<? super T>)
        //   where T is a type-variable:
        //     T extends Object declared in
        //     method <T>wildSupertype(Holder<? super T>,T)
        //   where CAP#1 is a fresh type-variable:
        //     CAP#1 extends Long from capture of
        //     ? extends Long
        // 1 error
    }
}
```

在 `rawArgs()` 中，編譯器知道 `Holder` 是一個泛型類型，因此即使它在這裡被表示成一個原生類型，編譯器仍舊知道向 `set()` 傳遞一個 **Object** 是不安全的。由於它是原生類型，你可以將任何類型的物件傳遞給 `set()` ，而這個物件將被向上轉型為 **Object** 。因此無論何時，只要使用了原生類型，都會放棄編譯期檢查。對 `get()` 的呼叫說明了相同的問題：沒有任何 **T** 類型的物件，因此結果只能是一個 **Object**。
人們很自然地會開始考慮原生 `Holder` 與 `Holder<?>` 是大致相同的事物。但是 `unboundedArg()` 強調它們是不同的——它揭示了相同的問題，但是它將這些問題作為錯誤而不是警告報告，因為原生 **Holder** 將持有任何類型的組合，而 `Holder<?>` 將持有具有某種具體類型的同構集合，因此不能只是向其中傳遞 **Object** 。
在 `exact1()` 和 `exact2()` 中，你可以看到使用了確切的泛型參數——沒有任何萬用字元。你將看到，`exact2()`與 `exact1()` 具有不同的限制，因為它有額外的參數。
在 `wildSubtype()` 中，在 **Holder** 類型上的限制被放鬆為包括持有任何擴展自 **T** 的物件的 **Holder** 。這還是意味著如果 T 是 **Fruit** ，那麼 `holder` 可以是 `Holder<Apple>` ，這是合法的。為了防止將 **Orange** 放置到 `Holder<Apple>` 中，對 `set()` 的呼叫（或者對任何接受這個類型參數為參數的方法的呼叫）都是不允許的。但是，你仍舊知道任何來自 `Holder<？extends Fruit>` 的物件至少是 **Fruit** ，因此 `get()` （或者任何將產生具有這個類型參數的返回值的方法）都是允許的。
`wildSupertype()` 展示了超類型萬用字元，這個方法展示了與 `wildSubtype()` 相反的行為：`holder` 可以是持有任何 T 的基類型的容器。因此， `set()` 可以接受 **T** ，因為任何可以工作於基類的物件都可以多態地作用於匯出類（這裡就是 **T** ）。但是，嘗試著呼叫 `get()` 是沒有用的，因為由 `holder` 持有的類型可以是任何超類型，因此唯一安全的類型就是 **Object** 。
這個範例還展示了對於在 `unbounded()` 中使用無界萬用字元能夠做什麼不能做什麼所做出的限制：因為你沒有 **T**，所以你不能將 `set()` 或 `get()` 作用於 **T** 上。

在 `main()` 方法中你看到了某些方法在接受某些類型的參數時沒有報錯和警告。為了遷移相容性，`rawArgs()`  將接受所有 **Holder** 的不同變體，而不會產生警告。`unboundedArg()` 方法也可以接受相同的所有類型，儘管如前所述，它在方法體內部處理這些類型的方式並不相同。

如果向接受“確切”泛型類型（沒有萬用字元）的方法傳遞一個原生 **Holder** 引用，就會得到一個警告，因為確切的參數期望得到在原生類型中並不存在的訊息。如果向 `exact1()` 傳遞一個無界引用，就不會有任何可以確定返回類型的類型訊息。
可以看到，`exact2()` 具有最多的限制，因為它希望精確地得到一個 `Holder<T>` ，以及一個具有類型 **T** 的參數，正由於此，它將產生錯誤或警告，除非提供確切的參數。有時，這樣做很好，但是如果它過於受限，那麼就可以使用萬用字元，這取決於是否想要從泛型參數中返回類型確定的返回值（就像在 `wildSubtype()` 中看到的那樣），或者是否想要向泛型參數傳遞類型確定的參數（就像在 `wildSupertype()` 中看到的那樣）。
因此，使用確切類型來替代萬用字元類型的好處是，可以用泛型參數來做更多的事，但是使用萬用字元使得你必須接受範圍更寬的參數化類型作為參數。因此，必須逐個情況地權衡利弊，找到更適合你的需求的方法。

### 捕獲轉換

有一種特殊情況需要使用 `<?>` 而不是原生類型。如果向一個使用 `<?>` 的方法傳遞原生類型，那麼對編譯器來說，可能會推斷出實際的類型參數，使得這個方法可以迴轉並呼叫另一個使用這個確切類型的方法。下面的範例示範了這種技術，它被稱為捕獲轉換，因為未指定的萬用字元類型被捕獲，並被轉換為確切類型。這裡，有關警告的注釋只有在 `@SuppressWarnings` 註解被移除之後才能起作用：

```java
// generics/CaptureConversion.java

public class CaptureConversion {
    static <T> void f1(Holder<T> holder) {
        T t = holder.get();
        System.out.println(t.getClass().getSimpleName());
    }
  
    static void f2(Holder<?> holder) {
        f1(holder); // Call with captured type
    }
    
    @SuppressWarnings("unchecked")
    public static void main(String[] args) {
        Holder raw = new Holder<>(1);
        f1(raw);
        // warning: [unchecked] unchecked method invocation:
        // method f1 in class CaptureConversion
        // is applied to given types
        //     f1(raw);
        //       ^
        //   required: Holder<T>
        //   found: Holder
        //   where T is a type-variable:
        //     T extends Object declared in
        //     method <T>f1(Holder<T>)
        // warning: [unchecked] unchecked conversion
        //     f1(raw);
        //        ^
        //   required: Holder<T>
        //   found:    Holder
        //   where T is a type-variable:
        //     T extends Object declared in
        //     method <T>f1(Holder<T>)
        // 2 warnings
        f2(raw); // No warnings
        
        Holder rawBasic = new Holder();
        rawBasic.set(new Object());
        // warning: [unchecked] unchecked call to set(T)
        // as a member of the raw type Holder
        //     rawBasic.set(new Object());
        //                 ^
        //   where T is a type-variable:
        //     T extends Object declared in class Holder
        // 1 warning
        f2(rawBasic); // No warnings
        
        // Upcast to Holder<?>, still figures it out:
        Holder<?> wildcarded = new Holder<>(1.0);
        f2(wildcarded);
    }
}
/* Output:
Integer
Integer
Object
Double
*/
```

`f1()` 中的類型參數都是確切的，沒有萬用字元或邊界。在 `f2()` 中，**Holder** 參數是一個無界萬用字元，因此它看起來是未知的。但是，在 `f2()` 中呼叫了 `f1()`，而 `f1()` 需要一個已知參數。這裡所發生的是：在呼叫 `f2()` 的過程中捕獲了參數類型，並在呼叫 `f1()` 時使用了這種類型。
你可能想知道這項技術是否可以用於寫入，但是這要求在傳遞 `Holder<?>` 時同時傳遞一個具體類型。捕獲轉換只有在這樣的情況下可以工作：即在方法內部，你需要使用確切的類型。注意，不能從 `f2()` 中返回 **T**，因為 **T** 對於 `f2()` 來說是未知的。捕獲轉換十分有趣，但是非常受限。

<!-- Issues -->

## 問題

本節將闡述在使用 Java 泛型時會出現的各類問題。

### 任何基本類型都不能作為類型參數

正如本章早先提到的，Java 泛型的限制之一是不能將基本類型用作類型參數。因此，不能建立  `ArrayList<int>` 之類的東西。
解決方法是使用基本類型的包裝器類以及自動裝箱機制。如果建立一個 `ArrayList<Integer>`，並將基本類型 **int** 應用於這個集合，那麼你將發現自動裝箱機制將自動地實現 **int** 到 **Integer** 的雙向轉換——因此，這幾乎就像是有一個 `ArrayList<int>` 一樣：

```java
// generics/ListOfInt.java
// Autoboxing compensates for the inability
// to use primitives in generics
import java.util.*;
import java.util.stream.*;

public class ListOfInt {
    public static void main(String[] args) {
        List<Integer> li = IntStream.range(38, 48)
            .boxed() // Converts ints to Integers
            .collect(Collectors.toList());
        System.out.println(li);
    }
}
/* Output:
[38, 39, 40, 41, 42, 43, 44, 45, 46, 47]
*/
```

通常，這種解決方案工作得很好——能夠成功地儲存和讀取 **int**，自動裝箱隱藏了轉換的過程。但是如果性能成為問題的話，就需要使用專門為基本類型適配的特殊版本的集合；一個開源版本的實現是 **org.apache.commons.collections.primitives**。
下面是另外一種方式，它可以建立持有 **Byte** 的 **Set**：

```java
// generics/ByteSet.java
import java.util.*;

public class ByteSet {
    Byte[] possibles = { 1,2,3,4,5,6,7,8,9 };
    Set<Byte> mySet = new HashSet<>(Arrays.asList(possibles));
    // But you can't do this:
    // Set<Byte> mySet2 = new HashSet<>(
    // Arrays.<Byte>asList(1,2,3,4,5,6,7,8,9));
}
```

自動裝箱機制解決了一些問題，但並沒有解決所有問題。

在下面的範例中，**FillArray** 介面包含一些通用方法，這些方法使用 **Supplier** 來用物件填充陣列（這使得類泛型在本例中無法工作，因為這個方法是靜態的）。**Supplier** 實現來自 [陣列](book/21-Arrays.md) 一章,並且在 `main()` 中，可以看到 `FillArray.fill()` 使用物件填充了陣列：

```java
// generics/PrimitiveGenericTest.java
import onjava.*;
import java.util.*;
import java.util.function.*;

// Fill an array using a generator:
interface FillArray {
    static <T> T[] fill(T[] a, Supplier<T> gen) {
        Arrays.setAll(a, n -> gen.get());
        return a;
    }
    
    static int[] fill(int[] a, IntSupplier gen) {
        Arrays.setAll(a, n -> gen.getAsInt());
        return a;
    }
    
    static long[] fill(long[] a, LongSupplier gen) {
        Arrays.setAll(a, n -> gen.getAsLong());
        return a;
    }
    
    static double[] fill(double[] a, DoubleSupplier gen) {
        Arrays.setAll(a, n -> gen.getAsDouble());
        return a;
    }
}

public class PrimitiveGenericTest {
    public static void main(String[] args) {
        String[] strings = FillArray.fill(
            new String[5], new Rand.String(9));
        System.out.println(Arrays.toString(strings));
        int[] integers = FillArray.fill(
            new int[9], new Rand.Pint());
        System.out.println(Arrays.toString(integers));
    }
}
/* Output:
[btpenpccu, xszgvgmei, nneeloztd, vewcippcy, gpoalkljl]
[635, 8737, 3941, 4720, 6177, 8479, 6656, 3768, 4948]
*/
```

自動裝箱不適用於陣列，因此我們必須建立 `FillArray.fill()` 的重載版本，或建立產生 **Wrapped** 輸出的生成器。 **FillArray** 僅比 `java.util.Arrays.setAll()` 有用一點，因為它返回填充的陣列。

### 實現參數化介面

一個類不能實現同一個泛型介面的兩種變體，由於擦除的原因，這兩個變體會成為相同的介面。下面是產生這種衝突的情況：

```java
// generics/MultipleInterfaceVariants.java
// {WillNotCompile}
package generics;

interface Payable<T> {}

class Employee implements Payable<Employee> {}

class Hourly extends Employee implements Payable<Hourly> {}
```

**Hourly** 不能編譯，因為擦除會將  `Payable<Employe>` 和 `Payable<Hourly>` 簡化為相同的類 **Payable**，這樣，上面的程式碼就意味著在重複兩次地實現相同的介面。十分有趣的是，如果從 **Payable** 的兩種用法中都移除掉泛型參數（就像編譯器在擦除階段所做的那樣）這段程式碼就可以編譯。

在使用某些更基本的 Java 介面，例如 `Comparable<T>` 時，這個問題可能會變得十分令人惱火，就像你在本節稍後看到的那樣。

### 轉型和警告

使用帶有泛型類型參數的轉型或 **instanceof** 不會有任何效果。下面的集合在內部將各個值儲存為 **Object**，並在獲取這些值時，再將它們轉型回 **T**：

```java
// generics/GenericCast.java
import java.util.*;
import java.util.stream.*;

class FixedSizeStack<T> {
    private final int size;
    private Object[] storage;
    private int index = 0;
    
    FixedSizeStack(int size) {
        this.size = size;
        storage = new Object[size];
    }
    
    public void push(T item) {
        if(index < size)
            storage[index++] = item;
    }
    
    @SuppressWarnings("unchecked")
    public T pop() {
        return index == 0 ? null : (T)storage[--index];
    }
    
    @SuppressWarnings("unchecked")
    Stream<T> stream() {
        return (Stream<T>)Arrays.stream(storage);
    }
}

public class GenericCast {
    static String[] letters = "ABCDEFGHIJKLMNOPQRS".split("");
  
    public static void main(String[] args) {
        FixedSizeStack<String> strings =
            new FixedSizeStack<>(letters.length);
        Arrays.stream("ABCDEFGHIJKLMNOPQRS".split(""))
            .forEach(strings::push);
        System.out.println(strings.pop());
        strings.stream()
            .map(s -> s + " ")
            .forEach(System.out::print);
    }
}
/* Output:
S
A B C D E F G H I J K L M N O P Q R S
*/
```

如果沒有 **@SuppressWarnings** 註解，編譯器將對 `pop()` 產生 “unchecked cast” 警告。由於擦除的原因，編譯器無法知道這個轉型是否是安全的，並且 `pop()` 方法實際上並沒有執行任何轉型。
這是因為，**T** 被擦除到它的第一個邊界，預設情況下是 **Object** ，因此 `pop()` 實際上只是將 **Object** 轉型為 **Object**。
有時，泛型沒有消除對轉型的需要，這就會由編譯器產生警告，而這個警告是不恰當的。例如：

```java
// generics/NeedCasting.java
import java.io.*;
import java.util.*;

public class NeedCasting {
    @SuppressWarnings("unchecked")
    public void f(String[] args) throws Exception {
        ObjectInputStream in = new ObjectInputStream(
            new FileInputStream(args[0]));
        List<Widget> shapes = (List<Widget>)in.readObject();
    }
}
```

正如你將在 [附錄：物件序列化](book/Appendix-Object-Serialization.md) 中學到的那樣，`readObject()` 無法知道它正在讀取的是什麼，因此它返回的是必須轉型的物件。但是當注釋掉 **@SuppressWarnings** 註解並編譯這個程式時，就會得到下面的警告。

```
NeedCasting.java uses unchecked or unsafe operations.
Recompile with -Xlint:unchecked for details.

And if you follow the instructions and recompile with  -
Xlint:unchecked :(如果遵循這條指示，使用-Xlint:unchecked來重新編譯：)

NeedCasting.java:10: warning: [unchecked] unchecked cast
    List<Widget> shapes = (List<Widget>)in.readObject();
    required: List<Widget>
    found: Object
1 warning
```

你會被強制要求轉型，但是又被告知不應該轉型。為了解決這個問題，必須使用 Java 5 引入的新的轉型形式，即透過泛型類來轉型：

```java
// generics/ClassCasting.java
import java.io.*;
import java.util.*;

public class ClassCasting {
    @SuppressWarnings("unchecked")
    public void f(String[] args) throws Exception {
        ObjectInputStream in = new ObjectInputStream(
            new FileInputStream(args[0]));
        // Won't Compile:
        //    List<Widget> lw1 =
        //    List<>.class.cast(in.readObject());
        List<Widget> lw2 = List.class.cast(in.readObject());
    }
}
```

但是，不能轉型到實際類型（ `List<Widget>` ）。也就是說，不能聲明：

```
List<Widget>.class.cast(in.readobject())
```

甚至當你添加一個像下面這樣的另一個轉型時：

```
(List<Widget>)List.class.cast(in.readobject())
```

仍舊會得到一個警告。

### 重載

下面的程式是不能編譯的，即使它看起來是合理的：

```java
// generics/UseList.java
// {WillNotCompile}
import java.util.*;

public class UseList<W, T> {
    void f(List<T> v) {}
    void f(List<W> v) {}
}
```

因為擦除，所以重載方法產生了相同的類型簽名。

因而，當擦除後的參數不能產生唯一的參數列表時，你必須提供不同的方法名：

```java
// generics/UseList2.java

import java.util.*;

public class UseList2<W, T> {
    void f1(List<T> v) {}
    void f2(List<W> v) {}
}
```

幸運的是，編譯器可以檢測到這類問題。

### 基類劫持介面

假設你有一個實現了 **Comparable** 介面的 **Pet** 類：

```java
// generics/ComparablePet.java

public class ComparablePet implements Comparable<ComparablePet> {
    @Override
    public int compareTo(ComparablePet o) {
        return 0;
    }
}
```

嘗試縮小 **ComparablePet** 子類的比較類型是有意義的。例如，**Cat** 類可以與其他的 **Cat** 比較：

```java
// generics/HijackedInterface.java
// {WillNotCompile}

class Cat extends ComparablePet implements Comparable<Cat> {
    // error: Comparable cannot be inherited with
    // different arguments: <Cat> and <ComparablePet>
    // class Cat
    // ^
    // 1 error
    public int compareTo(Cat arg) {
        return 0;
    }
}
```

不幸的是，這不能工作。一旦 **Comparable** 的類型參數設定為 **ComparablePet**，其他的實現類只能比較 **ComparablePet**：

```java
// generics/RestrictedComparablePets.java

public class Hamster extends ComparablePet implements Comparable<ComparablePet> {

    @Override
    public int compareTo(ComparablePet arg) {
        return 0;
    }
}
// Or just:
class Gecko extends ComparablePet {
    public int compareTo(ComparablePet arg) {
        return 0;
    }
}
```

**Hamster** 顯示了重新實現 **ComparablePet** 中相同的介面是可能的，只要介面完全相同，包括參數類型。然而正如 **Gecko** 中所示，這與直接覆寫基類的方法完全相同。

<!-- Self-Bounded Types -->

## 自限定的類型

在 Java 泛型中，有一個似乎經常性出現的慣用法，它相當令人費解：

```java
class SelfBounded<T extends SelfBounded<T>> { // ...
```

這就像兩面鏡子彼此照向對方所引起的目眩效果一樣，是一種無限反射。**SelfBounded** 類接受泛型參數 **T**，而 **T** 由一個邊界類限定，這個邊界就是擁有 **T** 作為其參數的 **SelfBounded**。

當你首次看到它時，很難去解析它，它強調的是當 **extends** 關鍵字用於邊界與用來建立子類明顯是不同的。

### 古怪的循環泛型

為了理解自限定類型的含義，我們從這個慣用法的一個簡單版本入手，它沒有自限定的邊界。

不能直接繼承一個泛型參數，但是，可以繼承在其自己的定義中使用這個泛型參數的類。也就是說，可以聲明：

```java
// generics/CuriouslyRecurringGeneric.java

class GenericType<T> {}

public class CuriouslyRecurringGeneric
  extends GenericType<CuriouslyRecurringGeneric> {}
```

這可以按照 Jim Coplien 在 C++ 中的*古怪的循環模版模式*的命名方式，稱為古怪的循環泛型（CRG）。“古怪的循環”是指類相當古怪地出現在它自己的基類中這一事實。
為了理解其含義，努力大聲說：“我在建立一個新類，它繼承自一個泛型類型，這個泛型類型接受我的類的名字作為其參數。”當給出匯出類的名字時，這個泛型基類能夠實現什麼呢？好吧，Java 中的泛型攸關參數和返回類型，因此它能夠產生使用匯出類作為其參數和返回類型的基類。它還能將匯出類型用作其域類型，儘管這些將被擦除為 **Object** 的類型。下面是表示了這種情況的一個泛型類：

```java
// generics/BasicHolder.java

public class BasicHolder<T> {
    T element;
    void set(T arg) { element = arg; }
    T get() { return element; }
    void f() {
        System.out.println(element.getClass().getSimpleName());
    }
}
```

這是一個普通的泛型類型，它的一些方法將接受和產生具有其參數類型的物件，還有一個方法在其儲存的域上執行操作（儘管只是在這個域上執行 **Object** 操作）。
我們可以在一個古怪的循環泛型中使用 **BasicHolder**：

```java
// generics/CRGWithBasicHolder.java

class Subtype extends BasicHolder<Subtype> {}

public class CRGWithBasicHolder {
    public static void main(String[] args) {
        Subtype st1 = new Subtype(), st2 = new Subtype();
        st1.set(st2);
        Subtype st3 = st1.get();
        st1.f();
    }
}
/* Output:
Subtype
*/
```

注意，這裡有些東西很重要：新類 **Subtype** 接受的參數和返回的值具有 **Subtype** 類型而不僅僅是基類 **BasicHolder** 類型。這就是 CRG 的本質：基類用匯出類替代其參數。這意味著泛型基類變成了一種其所有匯出類的公共功能的模版，但是這些功能對於其所有參數和返回值，將使用匯出類型。也就是說，在所產生的類中將使用確切類型而不是基類型。因此，在**Subtype** 中，傳遞給 `set()` 的參數和從 `get()` 返回的類型都是確切的 **Subtype**。

### 自限定

**BasicHolder** 可以使用任何類型作為其泛型參數，就像下面看到的那樣：

```java
// generics/Unconstrained.java
// (c)2017 MindView LLC: see Copyright.txt
// We make no guarantees that this code is fit for any purpose.
// Visit http://OnJava8.com for more book information.

class Other {}
class BasicOther extends BasicHolder<Other> {}

public class Unconstrained {
    public static void main(String[] args) {
        BasicOther b = new BasicOther();
        BasicOther b2 = new BasicOther();
        b.set(new Other());
        Other other = b.get();
        b.f();
    }
}
/* Output:
Other
*/
```

限定將採取額外的步驟，強制泛型當作其自身的邊界參數來使用。觀察所產生的類可以如何使用以及不可以如何使用：

```java
// generics/SelfBounding.java

class SelfBounded<T extends SelfBounded<T>> {
    T element;
    SelfBounded<T> set(T arg) {
        element = arg;
        return this;
    }
    T get() { return element; }
}

class A extends SelfBounded<A> {}
class B extends SelfBounded<A> {} // Also OK

class C extends SelfBounded<C> {
    C setAndGet(C arg) { 
        set(arg); 
        return get();
    }
}

class D {}
// Can't do this:
// class E extends SelfBounded<D> {}
// Compile error:
//   Type parameter D is not within its bound

// Alas, you can do this, so you cannot force the idiom:
class F extends SelfBounded {}

public class SelfBounding {
    public static void main(String[] args) {
        A a = new A();
        a.set(new A());
        a = a.set(new A()).get();
        a = a.get();
        C c = new C();
        c = c.setAndGet(new C());
    }
}
```

自限定所做的，就是要求在繼承關係中，像下面這樣使用這個類：

```java
class A extends SelfBounded<A>{}
```

這會強制要求將正在定義的類當作參數傳遞給基類。

自限定的參數有何意義呢？它可以保證類型參數必須與正在被定義的類相同。正如你在 B 類的定義中所看到的，還可以從使用了另一個 **SelfBounded** 參數的 **SelfBounded** 中匯出，儘管在 **A** 類看到的用法看起來是主要的用法。對定義 **E** 的嘗試說明不能使用不是 **SelfBounded** 的類型參數。
遺憾的是， **F** 可以編譯，不會有任何警告，因此自限定慣用法不是可強制執行的。如果它確實很重要，可以要求一個外部工具來確保不會使用原生類型來替代參數化類型。
注意，可以移除自限定這個限制，這樣所有的類仍舊是可以編譯的，但是 **E** 也會因此而變得可編譯：

```java
// generics/NotSelfBounded.java

public class NotSelfBounded<T> {
    T element;
    NotSelfBounded<T> set(T arg) {
        element = arg;
        return this;
    }
    T get() { return element; }
} 

class A2 extends NotSelfBounded<A2> {}
class B2 extends NotSelfBounded<A2> {}

class C2 extends NotSelfBounded<C2> {
    C2 setAndGet(C2 arg) { 
        set(arg); 
        return get(); 
    }
}

class D2 {}
// Now this is OK:
class E2 extends NotSelfBounded<D2> {}
```

因此很明顯，自限定限制只能強制作用於繼承關係。如果使用自限定，就應該了解這個類所用的類型參數將與使用這個參數的類具有相同的基類型。這會強制要求使用這個類的每個人都要遵循這種形式。
還可以將自限定用於泛型方法：

```java
// generics/SelfBoundingMethods.java
// (c)2017 MindView LLC: see Copyright.txt
// We make no guarantees that this code is fit for any purpose.
// Visit http://OnJava8.com for more book information.

public class SelfBoundingMethods {
    static <T extends SelfBounded<T>> T f(T arg) {
        return arg.set(arg).get();
    }
    
    public static void main(String[] args) {
        A a = f(new A());
    }
}
```

這可以防止這個方法被應用於除上述形式的自限定參數之外的任何事物上。

### 參數協變

自限定類型的價值在於它們可以產生*協變參數類型*——方法參數類型會隨子類而變化。

儘管自限定類型還可以產生與子類類型相同的返回類型，但是這並不十分重要，因為*協變返回類型*是在 Java 5 引入：

```java
// generics/CovariantReturnTypes.java

class Base {}
class Derived extends Base {}

interface OrdinaryGetter {
    Base get();
}

interface DerivedGetter extends OrdinaryGetter {
    // Overridden method return type can vary:
    @Override
    Derived get();
}

public class CovariantReturnTypes {
    void test(DerivedGetter d) {
        Derived d2 = d.get();
    }
}
```

**DerivedGetter** 中的 `get()` 方法覆蓋了 **OrdinaryGetter** 中的 `get()` ，並返回了一個從 `OrdinaryGetter.get()` 的返回類型中匯出的類型。儘管這是完全合乎邏輯的事情（匯出類方法應該能夠返回比它覆蓋的基類方法更具體的類型）但是這在早先的 Java 版本中是不合法的。

自限定泛型事實上將產生確切的匯出類型作為其返回值，就像在 `get()` 中所看到的一樣：

```java
// generics/GenericsAndReturnTypes.java

interface GenericGetter<T extends GenericGetter<T>> {
    T get();
}

interface Getter extends GenericGetter<Getter> {}

public class GenericsAndReturnTypes {
    void test(Getter g) {
        Getter result = g.get();
        GenericGetter gg = g.get(); // Also the base type
    }
}
```

注意，這段程式碼不能編譯，除非是使用囊括了協變返回類型的 Java 5。

然而，在非泛型程式碼中，參數類型不能隨子類型發生變化：

```java
// generics/OrdinaryArguments.java

class OrdinarySetter {
    void set(Base base) {
        System.out.println("OrdinarySetter.set(Base)");
    }
}

class DerivedSetter extends OrdinarySetter {
    void set(Derived derived) {
        System.out.println("DerivedSetter.set(Derived)");
    }
}

public class OrdinaryArguments {
    public static void main(String[] args) {
        Base base = new Base();
        Derived derived = new Derived();
        DerivedSetter ds = new DerivedSetter();
        ds.set(derived);
        // Compiles--overloaded, not overridden!:
        ds.set(base);
    }
}
/* Output:
DerivedSetter.set(Derived)
OrdinarySetter.set(Base)
*/
```

`set(derived)` 和 `set(base)` 都是合法的，因此 `DerivedSetter.set()` 沒有覆蓋 `OrdinarySetter.set()` ，而是重載了這個方法。從輸出中可以看到，在 **DerivedSetter** 中有兩個方法，因此基類版本仍舊是可用的，因此可以證明它被重載過。
但是，在使用自限定類型時，在匯出類中只有一個方法，並且這個方法接受匯出類型而不是基類型為參數：

```java
// generics/SelfBoundingAndCovariantArguments.java

interface SelfBoundSetter<T extends SelfBoundSetter<T>> {
    void set(T arg);
}

interface Setter extends SelfBoundSetter<Setter> {}

public class SelfBoundingAndCovariantArguments {
    void
    testA(Setter s1, Setter s2, SelfBoundSetter sbs) {
        s1.set(s2);
        //- s1.set(sbs);
        // error: method set in interface SelfBoundSetter<T>
        // cannot be applied to given types;
        //     s1.set(sbs);
        //       ^
        //   required: Setter
        //   found: SelfBoundSetter
        //   reason: argument mismatch;
        // SelfBoundSetter cannot be converted to Setter
        //   where T is a type-variable:
        //     T extends SelfBoundSetter<T> declared in
        //     interface SelfBoundSetter
        // 1 error
    }
}
```

編譯器不能識別將基類型當作參數傳遞給 `set()` 的嘗試，因為沒有任何方法具有這樣的簽名。實際上，這個參數已經被覆蓋。
如果不使用自限定類型，普通的繼承機制就會介入，而你將能夠重載，就像在非泛型的情況下一樣：

```java
// generics/PlainGenericInheritance.java

class GenericSetter<T> { // Not self-bounded
    void set(T arg) {
        System.out.println("GenericSetter.set(Base)");
    }
}

class DerivedGS extends GenericSetter<Base> {
    void set(Derived derived) {
        System.out.println("DerivedGS.set(Derived)");
    }
}

public class PlainGenericInheritance {
    public static void main(String[] args) {
        Base base = new Base();
        Derived derived = new Derived();
        DerivedGS dgs = new DerivedGS();
        dgs.set(derived);
        dgs.set(base); // Overloaded, not overridden!
    }
}
/* Output:
DerivedGS.set(Derived)
GenericSetter.set(Base)
*/
```

這段程式碼在模仿 **OrdinaryArguments.java**；在那個範例中，**DerivedSetter** 繼承自包含一個 `set(Base)` 的**OrdinarySetter** 。而這裡，**DerivedGS** 繼承自泛型建立的也包含有一個 `set(Base)`的 `GenericSetter<Base>`。就像 **OrdinaryArguments.java** 一樣，你可以從輸出中看到， **DerivedGS** 包含兩個  `set()` 的重載版本。如果不使用自限定，將重載參數類型。如果使用了自限定，只能獲得方法的一個版本，它將接受確切的參數類型。

<!-- Dynamic Type Safety -->

## 動態類型安全

因為可以向 Java 5 之前的程式碼傳遞泛型集合，所以舊式程式碼仍舊有可能會破壞你的集合。Java 5 的 **java.util.Collections** 中有一組便利工具，可以解決在這種情況下的類型檢查問題，它們是：靜態方法 `checkedCollection()` 、`checkedList()`、 `checkedMap()` 、 `checkedSet()` 、`checkedSortedMap()`和 `checkedSortedSet()`。這些方法每一個都會將你希望動態檢查的集合當作第一個參數接受，並將你希望強制要求的類型作為第二個參數接受。

受檢查的集合在你試圖插入類型不正確的物件時拋出 **ClassCastException** ，這與泛型之前的（原生）集合形成了對比，對於後者來說，當你將物件從集合中取出時，才會通知你出現了問題。在後一種情況中，你知道存在問題，但是不知道罪魁禍首在哪裡，如果使用受檢查的集合，就可以發現誰在試圖插入不良物件。
讓我們用受檢查的集合來看看“將貓插入到狗列表中”這個問題。這裡，`oldStyleMethod()` 表示遺留程式碼，因為它接受的是原生的 **List** ，而 **@SuppressWarnings（“unchecked”）** 註解對於壓制所產生的警告是必需的：

```java
// generics/CheckedList.java
// Using Collection.checkedList()
import typeinfo.pets.*;
import java.util.*;

public class CheckedList {
    @SuppressWarnings("unchecked")
    static void oldStyleMethod(List probablyDogs) {
        probablyDogs.add(new Cat());
    }
    
    public static void main(String[] args) {
        List<Dog> dogs1 = new ArrayList<>();
        oldStyleMethod(dogs1); // Quietly accepts a Cat
        List<Dog> dogs2 = Collections.checkedList(
            new ArrayList<>(), Dog.class);
        try {
            oldStyleMethod(dogs2); // Throws an exception
        } catch(Exception e) {
            System.out.println("Expected: " + e);
        }
        // Derived types work fine:
        List<Pet> pets = Collections.checkedList(
            new ArrayList<>(), Pet.class);
        pets.add(new Dog());
        pets.add(new Cat());
    }
}
/* Output:
Expected: java.lang.ClassCastException: Attempt to
insert class typeinfo.pets.Cat element into collection
with element type class typeinfo.pets.Dog
*/
```

執行這個程式時，你會發現插入一個 **Cat** 對於 **dogs1** 來說沒有任何問題，而 **dogs2** 立即會在這個錯誤類型的插入操作上拋出一個異常。還可以看到，將匯出類型的物件放置到將要檢查基類型的受檢查容器中是沒有問題的。

<!-- Exceptions -->

## 泛型異常

由於擦除的原因，**catch** 語句不能捕獲泛型類型的異常，因為在編譯期和執行時都必須知道異常的確切類型。泛型類也不能直接或間接繼承自 **Throwable**（這將進一步阻止你去定義不能捕獲的泛型異常）。
但是，類型參數可能會在一個方法的 **throws** 子句中用到。這使得你可以編寫隨檢查型異常類型變化的泛型程式碼：

```java
// generics/ThrowGenericException.java

import java.util.*;

interface Processor<T, E extends Exception> {
    void process(List<T> resultCollector) throws E;
}

class ProcessRunner<T, E extends Exception>
extends ArrayList<Processor<T, E>> {
    List<T> processAll() throws E {
        List<T> resultCollector = new ArrayList<>();
        for(Processor<T, E> processor : this)
            processor.process(resultCollector);
        return resultCollector;
    }
}

class Failure1 extends Exception {}

class Processor1
implements Processor<String, Failure1> {
    static int count = 3;
    @Override
    public void process(List<String> resultCollector)
    throws Failure1 {
        if(count-- > 1)
            resultCollector.add("Hep!");
        else
            resultCollector.add("Ho!");
        if(count < 0)
            throw new Failure1();
    }
}

class Failure2 extends Exception {}

class Processor2
implements Processor<Integer, Failure2> {
    static int count = 2;
    @Override
    public void process(List<Integer> resultCollector)
    throws Failure2 {
        if(count-- == 0)
            resultCollector.add(47);
        else {
            resultCollector.add(11);
        }
        if(count < 0)
            throw new Failure2();
    }
}

public class ThrowGenericException {
    public static void main(String[] args) {
        ProcessRunner<String, Failure1> runner =
            new ProcessRunner<>();
        for(int i = 0; i < 3; i++)
            runner.add(new Processor1());
        try {
            System.out.println(runner.processAll());
        } catch(Failure1 e) {
            System.out.println(e);
        }

        ProcessRunner<Integer, Failure2> runner2 =
            new ProcessRunner<>();
        for(int i = 0; i < 3; i++)
            runner2.add(new Processor2());
        try {
            System.out.println(runner2.processAll());
        } catch(Failure2 e) {
            System.out.println(e);
        }
    }
}
/* Output:
[Hep!, Hep!, Ho!]
Failure2
*/
```

**Processor** 執行 `process()` 方法，並且可能會拋出具有類型 **E** 的異常。`process()` 的結果儲存在 `List<T>resultCollector` 中（這被稱為*收集參數*）。**ProcessRunner** 有一個 `processAll()` 方法，它會在所持有的每個 **Process** 物件執行，並返回 **resultCollector** 。
如果不能參數化所拋出的異常，那麼由於檢查型異常的緣故，將不能編寫出這種泛化的程式碼。

<!-- Mixins -->

## 混型

術語*混型*隨時間的推移好像擁有了無數的含義，但是其最基本的概念是混合多個類的能力，以產生一個可以表示混型中所有類型的類。這往往是你最後的手段，它將使組裝多個類變得簡單易行。
混型的價值之一是它們可以將特性和行為一致地應用於多個類之上。如果想在混型類中修改某些東西，作為一種意外的好處，這些修改將會應用於混型所應用的所有類型之上。正由於此，混型有一點*面向切面編程* （AOP） 的味道，而切面經常被建議用來解決混型問題。

### C++ 中的混型

在 C++ 中，使用多重繼承的最大理由，就是為了使用混型。但是，對於混型來說，更有趣、更優雅的方式是使用參數化類型，因為混型就是繼承自其類型參數的類。在 C++ 中，可以很容易地建立混型，因為 C++ 能夠記住其模版參數的類型。
下面是一個 C++ 範例，它有兩個混型類型：一個使得你可以在每個物件中混入擁有一個時間戳這樣的屬性，而另一個可以混入一個序號。

```c++
// generics/Mixins.cpp

#include <string>
#include <ctime>
#include <iostream>
using namespace std;

template<class T> class TimeStamped : public T {
    long timeStamp;
public:
    TimeStamped() { timeStamp = time(0); }
    long getStamp() { return timeStamp; }
};

template<class T> class SerialNumbered : public T {
    long serialNumber;
    static long counter;
public:
    SerialNumbered() { serialNumber = counter++; }
    long getSerialNumber() { return serialNumber; }
};

// Define and initialize the static storage:
template<class T> long SerialNumbered<T>::counter = 1;

class Basic {
    string value;
public:
    void set(string val) { value = val; }
    string get() { return value; }
};

int main() {
    TimeStamped<SerialNumbered<Basic>> mixin1, mixin2;
    mixin1.set("test string 1");
    mixin2.set("test string 2");
    cout << mixin1.get() << " " << mixin1.getStamp() <<
      " " << mixin1.getSerialNumber() << endl;
    cout << mixin2.get() << " " << mixin2.getStamp() <<
      " " << mixin2.getSerialNumber() << endl;
}
/* Output:
test string 1 1452987605 1
test string 2 1452987605 2
*/
```

在 `main()` 中， **mixin1** 和 **mixin2** 所產生的類型擁有所混入類型的所有方法。可以將混型看作是一種功能，它可以將現有類映射到新的子類上。注意，使用這種技術來建立一個混型是多麼的輕而易舉。基本上，只需要聲明“這就是我想要的”，緊跟著它就發生了：

```c++
TimeStamped<SerialNumbered<Basic>> mixin1，mixin2；
```

遺憾的是，Java 泛型不允許這樣。擦除會忘記基類類型，因此

>  泛型類不能直接繼承自一個泛型參數

這突顯了許多我在 Java 語言設計決策（以及與這些功能一起發布）中遇到的一大問題：處理一件事很有希望，但是當您實際嘗試做一些有趣的事情時，您會發現自己做不到。

### 與介面混合

一種更常見的推薦解決方案是使用介面來產生混型效果，就像下面這樣：

```java
// generics/Mixins.java

import java.util.*;

interface TimeStamped { long getStamp(); }

class TimeStampedImp implements TimeStamped {
    private final long timeStamp;
    TimeStampedImp() {
        timeStamp = new Date().getTime();
    }
    @Override
    public long getStamp() { return timeStamp; }
}

interface SerialNumbered { long getSerialNumber(); }

class SerialNumberedImp implements SerialNumbered {
    private static long counter = 1;
    private final long serialNumber = counter++;
    @Override
    public long getSerialNumber() { return serialNumber; }
}

interface Basic {
    void set(String val);
    String get();
}

class BasicImp implements Basic {
    private String value;
    @Override
    public void set(String val) { value = val; }
    @Override
    public String get() { return value; }
}

class Mixin extends BasicImp
implements TimeStamped, SerialNumbered {
    private TimeStamped timeStamp = new TimeStampedImp();
    private SerialNumbered serialNumber =
        new SerialNumberedImp();
    @Override
    public long getStamp() {
        return timeStamp.getStamp();
    }
    @Override
    public long getSerialNumber() {
        return serialNumber.getSerialNumber();
    }
}

public class Mixins {
    public static void main(String[] args) {
        Mixin mixin1 = new Mixin(), mixin2 = new Mixin();
        mixin1.set("test string 1");
        mixin2.set("test string 2");
        System.out.println(mixin1.get() + " " +
            mixin1.getStamp() +  " " + mixin1.getSerialNumber());
        System.out.println(mixin2.get() + " " +
            mixin2.getStamp() +  " " + mixin2.getSerialNumber());
    }
}
/* Output:
test string 1 1494331663026 1
test string 2 1494331663027 2
*/
```

**Mixin** 類基本上是在使用*委託*，因此每個混入類型都要求在 **Mixin** 中有一個相應的域，而你必須在 **Mixin** 中編寫所有必需的方法，將方法呼叫轉發給恰當的物件。這個範例使用了非常簡單的類，但是當使用更複雜的混型時，程式碼數量會急速增加。[^4]

### 使用裝飾器模式

當你觀察混型的使用方式時，就會發現混型概念好像與*裝飾器*設計模式關係很近。裝飾器經常用於滿足各種可能的組合，而直接子類化會產生過多的類，因此是不實際的。
裝飾器模式使用分層物件來動態透明地向單個物件中添加責任。裝飾器指定包裝在最初的物件周圍的所有物件都具有相同的基本介面。某些事物是可裝飾的，可以透過將其他類包裝在這個可裝飾物件的四周，來將功能分層。這使得對裝飾器的使用是透明的——無論物件是否被裝飾，你都擁有一個可以向物件發送的公共消息集。裝飾類也可以添加新方法，但是正如你所見，這將是受限的。
裝飾器是透過使用組合和形式化結構（可裝飾物/裝飾器層次結構）來實現的，而混型是基於繼承的。因此可以將基於參數化類型的混型當作是一種泛型裝飾器機制，這種機制不需要裝飾器設計模式的繼承結構。
前面的範例可以被改寫為使用裝飾器：

```java
// generics/decorator/Decoration.java

// {java generics.decorator.Decoration}
package generics.decorator;
import java.util.*;

class Basic {
    private String value;
    public void set(String val) { value = val; }
    public String get() { return value; }
}

class Decorator extends Basic {
    protected Basic basic;
    Decorator(Basic basic) { this.basic = basic; }
    @Override
    public void set(String val) { basic.set(val); }
    @Override
    public String get() { return basic.get(); }
}

class TimeStamped extends Decorator {
    private final long timeStamp;
    TimeStamped(Basic basic) {
        super(basic);
        timeStamp = new Date().getTime();
    }
    public long getStamp() { return timeStamp; }
}

class SerialNumbered extends Decorator {
    private static long counter = 1;
    private final long serialNumber = counter++;
    SerialNumbered(Basic basic) { super(basic); }
    public long getSerialNumber() { return serialNumber; }
}

public class Decoration {
    public static void main(String[] args) {
        TimeStamped t = new TimeStamped(new Basic());
        TimeStamped t2 = new TimeStamped(
            new SerialNumbered(new Basic()));
        //- t2.getSerialNumber(); // Not available
        SerialNumbered s = new SerialNumbered(new Basic());
        SerialNumbered s2 = new SerialNumbered(
            new TimeStamped(new Basic()));
        //- s2.getStamp(); // Not available
  }
}
```

產生自泛型的類包含所有感興趣的方法，但是由使用裝飾器所產生的物件類型是最後被裝飾的類型。也就是說，儘管可以添加多個層，但是最後一層才是實際的類型，因此只有最後一層的方法是可視的，而混型的類型是所有被混合到一起的類型。因此對於裝飾器來說，其明顯的缺陷是它只能有效地工作於裝飾中的一層（最後一層），而混型方法顯然會更自然一些。因此，裝飾器只是對由混型提出的問題的一種局限的解決方案。

### 與動態代理混合

可以使用動態代理來建立一種比裝飾器更貼近混型模型的機制（查看 [類型訊息](book/19-Type-Information.md) 一章中關於 Java 的動態代理如何工作的解釋）。透過使用動態代理，所產生的類的動態類型將會是已經混入的組合類型。
由於動態代理的限制，每個被混入的類都必須是某個介面的實現：

```java
// generics/DynamicProxyMixin.java

import java.lang.reflect.*;
import java.util.*;
import onjava.*;
import static onjava.Tuple.*;

class MixinProxy implements InvocationHandler {
    Map<String, Object> delegatesByMethod;
    @SuppressWarnings("unchecked")
    MixinProxy(Tuple2<Object, Class<?>>... pairs) {
        delegatesByMethod = new HashMap<>();
        for(Tuple2<Object, Class<?>> pair : pairs) {
            for(Method method : pair.a2.getMethods()) {
                String methodName = method.getName();
                // The first interface in the map
                // implements the method.
                if(!delegatesByMethod.containsKey(methodName))
                    delegatesByMethod.put(methodName, pair.a1);
            }
        }
    }
    @Override
    public Object invoke(Object proxy, Method method,
      Object[] args) throws Throwable {
        String methodName = method.getName();
        Object delegate = delegatesByMethod.get(methodName);
        return method.invoke(delegate, args);
    }
    
    @SuppressWarnings("unchecked")
    public static Object newInstance(Tuple2... pairs) {
        Class[] interfaces = new Class[pairs.length];
        for(int i = 0; i < pairs.length; i++) {
            interfaces[i] = (Class)pairs[i].a2;
        }
        ClassLoader cl = pairs[0].a1.getClass().getClassLoader();
        return Proxy.newProxyInstance(cl, interfaces, new MixinProxy(pairs));
    }
}

public class DynamicProxyMixin {
    public static void main(String[] args) {
        Object mixin = MixinProxy.newInstance(
          tuple(new BasicImp(), Basic.class),
          tuple(new TimeStampedImp(), TimeStamped.class),
          tuple(new SerialNumberedImp(), SerialNumbered.class));
        Basic b = (Basic)mixin;
        TimeStamped t = (TimeStamped)mixin;
        SerialNumbered s = (SerialNumbered)mixin;
        b.set("Hello");
        System.out.println(b.get());
        System.out.println(t.getStamp());
        System.out.println(s.getSerialNumber());
    }
}
/* Output:
Hello
1494331653339
1
*/
```

因為只有動態類型而不是靜態類型才包含所有的混入類型，因此這仍舊不如 C++ 的方式好，因為可以在具有這些類型的物件上呼叫方法之前，你被強制要求必須先將這些物件向下轉型到恰當的類型。但是，它明顯地更接近於真正的混型。
為了讓 Java 支援混型，人們已經做了大量的工作朝著這個目標努力，包括建立了至少一種附加語言（ Jam 語言），它是專門用來支援混型的。

<!-- Latent Typing -->

## 潛在類型機制

在本章的開頭介紹過這樣的思想，即要編寫能夠儘可能廣泛地應用的程式碼。為了實現這一點，我們需要各種途徑來放鬆對我們的程式碼將要作用的類型所作的限制，同時不遺失靜態類型檢查的好處。然後，我們就可以編寫出無需修改就可以應用於更多情況的程式碼，即更加“泛化”的程式碼。

Java 泛型看起來是向這一方向邁進了一步。當你在編寫或使用只是持有物件的泛型時，這些程式碼將可以工作於任何類型（除了基本類型，儘管正如你所見到的，自動裝箱機制可以克服這一點）。或者，換個角度講，“持有器”泛型能夠聲明：“我不關心你是什麼類型”。如果程式碼不關心它將要作用的類型，那麼這種程式碼就可以真正地應用於任何地方，並因此而相當泛化。

還是正如你所見到的，當要在泛型類型上執行操作（即呼叫 **Object** 方法之外的方法）時，就會產生問題。擦除強制要求指定可能會用到的泛型類型的邊界，以安全地呼叫程式碼中的泛型物件上的具體方法。這是對“泛化”概念的一種明顯的限制，因為必須限制你的泛型類型，使它們繼承自特定的類，或者實現特定的介面。在某些情況下，你最終可能會使用普通類或普通介面，因為限定邊界的泛型可能會和指定類或介面沒有任何區別。

某些程式語言提供的一種解決方案稱為*潛在類型機制*或*結構化類型機制*，而更古怪的術語稱為*鴨子類型機制*，即“如果它走起來像鴨子，並且叫起來也像鴨子，那麼你就可以將它當作鴨子對待。”鴨子類型機制變成了一種相當流行的術語，可能是因為它不像其他的術語那樣承載著歷史的包袱。

泛型程式碼典型地只能在泛型類型上呼叫少量方法，而具有潛在類型機制的語言只要求實現某個方法子集，而不是某個特定類或介面，從而放鬆了這種限制（並且可以產生更加泛化的程式碼）。正由於此，潛在類型機制使得你可以橫跨類繼承結構，呼叫不屬於某個公共介面的方法。因此，實際上一段程式碼可以聲明：“我不關心你是什麼類型，只要你可以 `speak()` 和 `sit()` 即可。”由於不要求具體類型，因此程式碼就可以更加泛化。

潛在類型機制是一種程式碼組織和復用機制。有了它，編寫出的程式碼相對於沒有它編寫出的程式碼，能夠更容易地復用。程式碼組織和復用是所有電腦編程的基本手段：編寫一次，多次使用，並在一個位置儲存程式碼。因為我並未被要求去命名我的程式碼要操作於其上的確切介面，所以，有了潛在類型機制，我就可以編寫更少的程式碼，並更容易地將其應用於多個地方。

支援潛在類型機制的語言包括 Python（可以從 www.Python.org 免費下載）、C++、Ruby、SmallTalk 和 Go。Python 是動態類型語言（幾乎所有的類型檢查都發生在執行時），而 C++ 和 Go 是靜態類型語言（類型檢查發生在編譯期），因此潛在類型機制不要求靜態或動態類型檢查。

### pyhton 中的潛在類型

如果我們將上面的描述用 Python 來表示，如下所示：

```python
# generics/DogsAndRobots.py

class Dog:
    def speak(self):
        print("Arf!")
    def sit(self):
        print("Sitting")
    def reproduce(self):
        pass

class Robot:
    def speak(self):
        print("Click!")
    def sit(self):
        print("Clank!")
    def oilChange(self):
        pass

def perform(anything):
    anything.speak()
    anything.sit()

a = Dog()
b = Robot()
perform(a)
perform(b)

output = """
Arf!
Sitting
Click!
Clank!
"""
```

Python 使用縮排來確定作用域（因此不需要任何花括號），而冒號將表示新的作用域的開始。“**#**” 表示注釋到行尾，就像Java中的 “ **//** ”。類的方法需要顯式地指定 **this** 引用的等價物作為第一個參數，按慣例成為 **self** 。構造器呼叫不要求任何類型的“ **new** ”關鍵字，並且 Python 允許普通（非成員）函數，就像 `perform()` 所表明的那樣。注意，在 `perform(anything)` 中，沒有任何針對 **anything** 的類型，**anything** 只是一個標識符，它必須能夠執行 `perform()` 期望它執行的操作，因此這裡隱含著一個介面。但是你從來都不必顯式地寫出這個介面——它是潛在的。`perform()` 不關心其參數的類型，因此我可以向它傳遞任何物件，只要該物件支援  `speak()` 和 `sit()` 方法。如果傳遞給 `perform()` 的物件不支援這些操作，那麼將會得到執行時異常。

輸出規定使用三重引號建立帶有內嵌換行符的字串。

### C++ 中的潛在類型

我們可以用 C++ 產生相同的效果：

```c++
// generics/DogsAndRobots.cpp

#include <iostream>
using namespace std;

class Dog {
public:
    void speak() { cout << "Arf!" << endl; }
    void sit() { cout << "Sitting" << endl; }
    void reproduce() {}
};

class Robot {
public:
    void speak() { cout << "Click!" << endl; }
    void sit() { cout << "Clank!" << endl; }
    void oilChange() {}
};

template<class T> void perform(T anything) {
    anything.speak();
    anything.sit();
}

int main() {
    Dog d;
    Robot r;
    perform(d);
    perform(r);
}
/* Output:
Arf!
Sitting
Click!
Clank!
*/
```

在 Python 和 C++ 中，**Dog** 和 **Robot** 沒有任何共同的東西，只是碰巧有兩個方法具有相同的簽名。從類型的觀點看，它們是完全不同的類型。但是，`perform()` 不關心其參數的具體類型，並且潛在類型機制允許它接受這兩種類型的物件。
C++ 確保了它實際上可以發送的那些消息，如果試圖傳遞錯誤類型，編譯器就會給你一個錯誤消息（這些錯誤消息從歷史上看是相當可怕和冗長的，是 C++ 的模版名聲欠佳的主要原因）。儘管它們是在不同時期實現這一點的，C++ 在編譯期，而 Python 在執行時，但是這兩種語言都可以確保類型不會被誤用，因此被認為是強類型的。[^5]潛在類型機制沒有損害強類型機制。

### Go 中的潛在類型

這裡用 Go 語言編寫相同的程式：

```go
// generics/dogsandrobots.go

package main
import "fmt"

type Dog struct {}
func (this Dog) speak() { fmt.Printf("Arf!\n")}
func (this Dog) sit() { fmt.Printf("Sitting\n")}
func (this Dog) reproduce() {}

type Robot struct {}
func (this Robot) speak() { fmt.Printf("Click!\n") }
func (this Robot) sit() { fmt.Printf("Clank!\n") }
func (this Robot) oilChange() {}

func perform(speaker interface { speak(); sit() }) {
  speaker.speak();
  speaker.sit();
}

func main() {
  perform(Dog{})
  perform(Robot{})
}
/* Output:
Arf!
Sitting
Click!
Clank!
*/
```

Go 沒有 **class** 關鍵字，但是可以使用上述形式建立等效的基本類：它通常不定義為類，而是定義為 **struct** ，在其中定義資料欄位（此處不存在）。 對於每種方法，都以 **func** 關鍵字開頭，然後（為了將該方法附加到您的類上）放在括號中，該括號包含物件引用，該物件引用可以是任何標識符，但是我在這裡使用 **this** 來提醒您，就像在 C ++ 或 Java 中的 **this** 一樣。 然後，在Go中像這樣定義其餘的函數。

Go也沒有繼承關係，因此這種“物件導向的目標”形式是相對原始的，並且可能是我無法花更多的時間來學習該語言的主要原因。 但是，Go 的組成很簡單。

`perform()` 函數使用潛在類型：參數的確切類型並不重要，只要它包含了 `speak()` 和  `sit()` 方法即可。 該介面在此處匿名定義，內聯，如 `perform()` 的參數列表所示。

`main()` 證明 `perform()` 確實對其參數的確切類型不在乎，只要可以在該參數上呼叫 `talk()` 和 `sit()` 即可。 但是，就像 C ++ 模板函數一樣，在編譯時檢查類型。

語法 **Dog {}** 和 **Robot {}** 建立匿名的 **Dog** 和 **Robot** 結構。

### java中的直接潛在類型

因為泛型是在這場競賽的後期才添加到 Java 中，因此沒有任何機會可以去實現任何類型的潛在類型機制，因此 Java 沒有對這種特性的支援。所以，初看起來，Java 的泛型機制比支援潛在類型機制的語言更“缺乏泛化性”。（使用擦除來實現 Java 泛型的實現有時稱為第二類泛型類型）例如，在 Java 8  之前如果我們試圖用 Java 實現上面 dogs-and-robots 的範例，那麼就會被強制要求使用一個類或介面，並在邊界表達式中指定它：

```java
// generics/Performs.java

public interface Performs {
    void speak();
    void sit();
}
```

```java
// generics/DogsAndRobots.java
// No (direct) latent typing in Java
import typeinfo.pets.*;

class PerformingDog extends Dog implements Performs {
    @Override
    public void speak() { System.out.println("Woof!"); }
    @Override
    public void sit() { System.out.println("Sitting"); }
    public void reproduce() {}
}

class Robot implements Performs {
    public void speak() { System.out.println("Click!"); }
    public void sit() { System.out.println("Clank!"); }
    public void oilChange() {}
}

class Communicate {
    public static <T extends Performs>
      void perform(T performer) {
        performer.speak();
        performer.sit();
    }
}

public class DogsAndRobots {
    public static void main(String[] args) {
        Communicate.perform(new PerformingDog());
        Communicate.perform(new Robot());
    }
}
/* Output:
Woof!
Sitting
Click!
Clank!
*/
```

但是要注意，`perform()` 不需要使用泛型來工作，它可以被簡單地指定為接受一個 **Performs** 物件：

```java
// generics/SimpleDogsAndRobots.java
// Removing the generic; code still works

class CommunicateSimply {
    static void perform(Performs performer) {
        performer.speak();
        performer.sit();
    }
}

public class SimpleDogsAndRobots {
    public static void main(String[] args) {
        CommunicateSimply.perform(new PerformingDog());
        CommunicateSimply.perform(new Robot());
    }
}
/* Output:
Woof!
Sitting
Click!
Clank!
*/
```

在本例中，泛型不是必需的，因為這些類已經被強制要求實現 **Performs** 介面。

<!-- Compensating for the Lack of (Direct) Latent -->

## 對缺乏潛在類型機制的補償

儘管 Java 不直接支援潛在類型機制，但是這並不意味著泛型程式碼不能在不同的類型層次結構之間應用。也就是說，我們仍舊可以建立真正的泛型程式碼，但是這需要付出一些額外的努力。

### 反射

可以使用的一種方式是反射，下面的 `perform()` 方法就是用了潛在類型機制：

```java
// generics/LatentReflection.java
// Using reflection for latent typing
import java.lang.reflect.*;

// Does not implement Performs:
class Mime {
    public void walkAgainstTheWind() {}
    public void sit() {
        System.out.println("Pretending to sit");
    }
    public void pushInvisibleWalls() {}
    @Override
    public String toString() { return "Mime"; }
}

// Does not implement Performs:
class SmartDog {
    public void speak() { System.out.println("Woof!"); }
    public void sit() { System.out.println("Sitting"); }
    public void reproduce() {}
}

class CommunicateReflectively {
    public static void perform(Object speaker) {
        Class<?> spkr = speaker.getClass();
        try {
            try {
                Method speak = spkr.getMethod("speak");
                speak.invoke(speaker);
            } catch(NoSuchMethodException e) {
                System.out.println(speaker + " cannot speak");
            }
            try {
                Method sit = spkr.getMethod("sit");
                sit.invoke(speaker);
            } catch(NoSuchMethodException e) {
                System.out.println(speaker + " cannot sit");
            }
        } catch(SecurityException |
            IllegalAccessException |
            IllegalArgumentException |
            InvocationTargetException e) {
            throw new RuntimeException(speaker.toString(), e);
        }
    }
}

public class LatentReflection {
    public static void main(String[] args) {
        CommunicateReflectively.perform(new SmartDog());
        CommunicateReflectively.perform(new Robot());
        CommunicateReflectively.perform(new Mime());
    }
}
/* Output:
Woof!
Sitting
Click!
Clank!
Mime cannot speak
Pretending to sit
*/
```

上例中，這些類完全是彼此分離的，沒有任何公共基類（除了 **Object** ）或介面。透過反射, `CommunicateReflectively.perform()` 能夠動態地確定所需要的方法是否可用並呼叫它們。它甚至能夠處理 **Mime** 只具有一個必需的方法這一事實，並能夠部分實現其目標。

### 將一個方法應用於序列

反射提供了一些有用的可能性，但是它將所有的類型檢查都轉移到了執行時，因此在許多情況下並不是我們所希望的。如果能夠實現編譯期類型檢查，這通常會更符合要求。但是有可能實現編譯期類型檢查和潛在類型機制嗎？

讓我們看一個說明這個問題的範例。假設想要建立一個 `apply()` 方法，它能夠將任何方法應用於某個序列中的所有物件。這種情況下使用介面不適合，因為你想要將任何方法應用於一個物件集合，而介面不可能描述任何方法。如何用 Java 來實現這個需求呢？

最初，我們可以用反射來解決這個問題，由於有了 Java 的可變參數，這種方式被證明是相當優雅的：

```java
// generics/Apply.java

import java.lang.reflect.*;
import java.util.*;

public class Apply {
    public static <T, S extends Iterable<T>>
      void apply(S seq, Method f, Object... args) {
        try {
            for(T t: seq)
                f.invoke(t, args);
        } catch(IllegalAccessException |
            IllegalArgumentException |
            InvocationTargetException e) {
            // Failures are programmer errors
            throw new RuntimeException(e);
        }
    }
}
```

在 **Apply.java** 中，異常被轉換為 **RuntimeException** ，因為沒有多少辦法可以從這種異常中復原——在這種情況下，它們實際上代表著程式設計師的錯誤。

為什麼我們不只使用 Java 8 方法參考（稍後顯示）而不是反射方法 **f** ？ 注意，`invoke()` 和 `apply()` 的優點是它們可以接受任意數量的參數。 在某些情況下，靈活性可能至關重要。

為了測試 **Apply** ，我們首先建立一個 **Shape** 類：

```java
// generics/Shape.java

public class Shape {
    private static long counter = 0;
    private final long id = counter++;
    @Override
    public String toString() {
        return getClass().getSimpleName() + " " + id;
    }
    public void rotate() {
        System.out.println(this + " rotate");
    }
    public void resize(int newSize) {
        System.out.println(this + " resize " + newSize);
    }
}
```

被一個子類 **Square** 繼承：

```java
// generics/Square.java

public class Square extends Shape {}
```

透過這些，我們可以測試 **Apply**：

```java
// generics/ApplyTest.java

import java.util.*;
import java.util.function.*;
import onjava.*;

public class ApplyTest {
    public static
    void main(String[] args) throws Exception {
        List<Shape> shapes =
          Suppliers.create(ArrayList::new, Shape::new, 3);
        Apply.apply(shapes, Shape.class.getMethod("rotate"));
        Apply.apply(shapes, Shape.class.getMethod("resize", int.class), 7);

        List<Square> squares =
          Suppliers.create(ArrayList::new, Square::new, 3);
        Apply.apply(squares, Shape.class.getMethod("rotate"));
        Apply.apply(squares, Shape.class.getMethod("resize", int.class), 7);

        Apply.apply(new FilledList<>(Shape::new, 3),
          Shape.class.getMethod("rotate"));
        Apply.apply(new FilledList<>(Square::new, 3),
          Shape.class.getMethod("rotate"));

        SimpleQueue<Shape> shapeQ = Suppliers.fill(
          new SimpleQueue<>(), SimpleQueue::add,
          Shape::new, 3);
        Suppliers.fill(shapeQ, SimpleQueue::add,
          Square::new, 3);
        Apply.apply(shapeQ, Shape.class.getMethod("rotate"));
    }
}
/* Output:
Shape 0 rotate
Shape 1 rotate
Shape 2 rotate
Shape 0 resize 7
Shape 1 resize 7
Shape 2 resize 7
Square 3 rotate
Square 4 rotate
Square 5 rotate
Square 3 resize 7
Square 4 resize 7
Square 5 resize 7
Shape 6 rotate
Shape 7 rotate
Shape 8 rotate
Square 9 rotate
Square 10 rotate
Square 11 rotate
Shape 12 rotate
Shape 13 rotate
Shape 14 rotate
Square 15 rotate
Square 16 rotate
Square 17 rotate
*/
```

在 **Apply** 中，我們運氣很好，因為碰巧在 Java 中內建了一個由 Java 集合類庫使用的 **Iterable** 介面。正由於此， `apply()` 方法可以接受任何實現了 **Iterable** 介面的事物，包括諸如 **List** 這樣的所有 **Collection** 類。但是它還可以接受其他任何事物，只要能夠使這些事物是 **Iterable** 的——例如，在 `main()` 中使用下面定義的 **SimpleQueue** 類：

```java
// generics/SimpleQueue.java

// A different kind of Iterable collection
import java.util.*;

public class SimpleQueue<T> implements Iterable<T> {
    private LinkedList<T> storage = new LinkedList<>();
    public void add(T t) { storage.offer(t); }
    public T get() { return storage.poll(); }
    @Override
    public Iterator<T> iterator() {
        return storage.iterator();
    }
}
```

正如反射解決方案看起來那樣優雅，我們必須觀察到反射（儘管在 Java 的最新版本中得到了顯著改進）通常比非反射實現要慢，因為在執行時發生了很多事情。 但它不應阻止您嘗試這種解決方案，這依然是值得考慮的一點。

幾乎可以肯定，你會首先使用 Java 8 的函數式方法，並且只有在解決了特殊需求時才訴諸反射。 這裡對 **ApplyTest.java** 進行了重寫，以利用 Java 8 的流和函數工具：

```java
// generics/ApplyFunctional.java

import java.util.*;
import java.util.stream.*;
import java.util.function.*;
import onjava.*;

public class ApplyFunctional {
    public static void main(String[] args) {
        Stream.of(
          Stream.generate(Shape::new).limit(2),
          Stream.generate(Square::new).limit(2))
        .flatMap(c -> c) // flatten into one stream
        .peek(Shape::rotate)
        .forEach(s -> s.resize(7));

        new FilledList<>(Shape::new, 2)
          .forEach(Shape::rotate);
        new FilledList<>(Square::new, 2)
          .forEach(Shape::rotate);

        SimpleQueue<Shape> shapeQ = Suppliers.fill(
          new SimpleQueue<>(), SimpleQueue::add,
          Shape::new, 2);
        Suppliers.fill(shapeQ, SimpleQueue::add,
          Square::new, 2);
        shapeQ.forEach(Shape::rotate);
    }
}
/* Output:
Shape 0 rotate
Shape 0 resize 7
Shape 1 rotate
Shape 1 resize 7
Square 2 rotate
Square 2 resize 7
Square 3 rotate
Square 3 resize 7
Shape 4 rotate
Shape 5 rotate
Square 6 rotate
Square 7 rotate
Shape 8 rotate
Shape 9 rotate
Square 10 rotate
Square 11 rotate
*/
```

由於使用 Java 8，因此不需要 `Apply.apply()` 。

我們首先生成兩個 **Stream** ： 一個是 **Shape** ，一個是 **Square** ，並將它們展平為單個流。 儘管 Java 缺少功能語言中經常出現的 `flatten()` ，但是我們可以使用 `flatMap(c-> c)` 產生相同的結果，後者使用身份映射將操作簡化為“  **flatten** ”。

我們使用 `peek()` 當做對 `rotate()` 的呼叫，因為 `peek()` 執行一個操作（此處是出於副作用），並在未更改的情況下傳遞物件。

注意，使用 **FilledList** 和 **shapeQ** 呼叫 `forEach()` 比 `Apply.apply()` 程式碼整潔得多。 在程式碼簡單性和可讀性方面，結果比以前的方法好得多。 並且，現在也不可能從  `main()` 引發異常。

<!-- Assisted Latent Typing in Java 8 -->

## Java8 中的輔助潛在類型

先前聲明關於 Java 缺乏對潛在類型的支援在 Java 8 之前是完全正確的。但是，Java 8 中的非綁定方法引用使我們能夠產生一種潛在類型的形式，以滿足建立一段可工作在不相干類型上的程式碼。因為 Java 最初並不是如此設計，所以結果可想而知，比其他語言中要尷尬一些。但是，至少現在成為了可能，只是缺乏令人驚艷之處。

我在其他地方從沒遇過這種技術，因此我將其稱為輔助潛在類型。

我們將重寫 **DogsAndRobots.java** 來示範該技術。 為使外觀看起來與原始範例儘可能相似，我僅向每個原始類名添加了 **A**：

```java
// generics/DogsAndRobotMethodReferences.java

// "Assisted Latent Typing"
import typeinfo.pets.*;
import java.util.function.*;

class PerformingDogA extends Dog {
    public void speak() { System.out.println("Woof!"); }
    public void sit() { System.out.println("Sitting"); }
    public void reproduce() {}
}

class RobotA {
    public void speak() { System.out.println("Click!"); }
    public void sit() { System.out.println("Clank!"); }
    public void oilChange() {}
}

class CommunicateA {
    public static <P> void perform(P performer,
      Consumer<P> action1, Consumer<P> action2) {
        action1.accept(performer);
        action2.accept(performer);
    }
}

public class DogsAndRobotMethodReferences {
    public static void main(String[] args) {
        CommunicateA.perform(new PerformingDogA(),
          PerformingDogA::speak, PerformingDogA::sit);
        CommunicateA.perform(new RobotA(),
          RobotA::speak, RobotA::sit);
        CommunicateA.perform(new Mime(),
          Mime::walkAgainstTheWind,
          Mime::pushInvisibleWalls);
    }
}
/* Output:
Woof!
Sitting
Click!
Clank!
*/
```

**PerformingDogA** 和 **RobotA** 與 **DogsAndRobots.java** 中的相同，不同之處在於它們不繼承通用介面 **Performs** ，因此它們沒有通用性。

`CommunicateA.perform()` 在沒有約束的 **P** 上生成。 只要可以使用 `Consumer <P>`，它在這裡就可以是任何東西，這些 `Consumer<P>` 代表不帶參數的 **P** 方法的未綁定方法引用。當您呼叫 **Consumer**  的 `accept()` 方法時，它將方法引用綁定到執行者物件並呼叫該方法。 由於 [函數式編程](book/13-Functional-Programming.md) 一章中描述的“魔術”，我們可以將任何符合簽名的未綁定方法引用傳遞給 `CommunicateA.perform()` 。

之所以稱其為“輔助”，是因為您必須顯式地為 `perform()` 提供要使用的方法引用。 它不能只按名稱呼叫方法。

儘管傳遞未綁定的方法引用似乎要花很多力氣，但潛在類型的最終目標還是可以實現的。 我們建立了一個程式碼片段 `CommunicateA.perform()` ，該程式碼可用於任何具有符合簽名的方法引用的類型。 請注意，這與我們看到的其他語言中的潛在類型有所不同，因為這些語言不僅需要簽名以符合規範，還需要方法名稱。 因此，該技術可以說產生了更多的通用程式碼。

為了證明這一點，我還從 **LatentReflection.java** 中引入了 **Mime**。

### 使用**Suppliers**類的通用方法

透過輔助潛在類型，我們可以定義本章其他部分中使用的 **Suppliers** 類。 此類包含使用生成器填充 **Collection** 的工具方法。 泛化這些操作很有意義：

```java
// onjava/Suppliers.java

// A utility to use with Suppliers
package onjava;
import java.util.*;
import java.util.function.*;
import java.util.stream.*;

public class Suppliers {
    // Create a collection and fill it:
    public static <T, C extends Collection<T>> C
      create(Supplier<C> factory, Supplier<T> gen, int n) {
        return Stream.generate(gen)
            .limit(n)
            .collect(factory, C::add, C::addAll);
    }
    
    // Fill an existing collection:
    public static <T, C extends Collection<T>>
      C fill(C coll, Supplier<T> gen, int n) {
        Stream.generate(gen)
            .limit(n)
            .forEach(coll::add);
        return coll;
    }
    
    // Use an unbound method reference to
    // produce a more general method:
    public static <H, A> H fill(H holder,
      BiConsumer<H, A> adder, Supplier<A> gen, int n) {
        Stream.generate(gen)
            .limit(n)
            .forEach(a -> adder.accept(holder, a));
        return holder;
    }
}
```

`create()` 為你建立一個新的 **Collection** 子類型，而 `fill()` 的第一個版本將元素放入 **Collection** 的現有子類型中。 請注意，還會返回傳入的容器的確切類型，因此不會遺失類型訊息。[^6]

前兩種方法一般都受約束，只能與 **Collection** 子類型一起使用。`fill()` 的第二個版本適用於任何類型的 **holder** 。 它需要一個附加參數：未綁定方法引用 `adder. fill()` ，使用輔助潛在類型來使其與任何具有添加元素方法的 **holder** 類型一起使用。因為此未綁定方法 **adder** 必須帶有一個參數（要添加到 **holder** 的元素），所以 **adder** 必須是 `BiConsumer <H，A>` ，其中 **H** 是要綁定到的 **holder** 物件的類型，而 **A** 是要被添加的綁定元素類型。 對 `accept()` 的呼叫將使用參數 a 呼叫物件 **holder** 上的未綁定方法 **holder**。

在一個稍作模擬的測試中對 **Suppliers** 工具程式進行了測試，該模擬還使用了本章前面定義的 **RandomList** ：

```java
// generics/BankTeller.java

// A very simple bank teller simulation
import java.util.*;
import onjava.*;

class Customer {
    private static long counter = 1;
    private final long id = counter++;
    @Override
    public String toString() {
        return "Customer " + id;
    }
}

class Teller {
    private static long counter = 1;
    private final long id = counter++;
    @Override
    public String toString() {
        return "Teller " + id;
    }
}

class Bank {
    private List<BankTeller> tellers =
        new ArrayList<>();
    public void put(BankTeller bt) {
        tellers.add(bt);
    }
}

public class BankTeller {
    public static void serve(Teller t, Customer c) {
        System.out.println(t + " serves " + c);
    }
    public static void main(String[] args) {
        // Demonstrate create():
        RandomList<Teller> tellers =
            Suppliers.create(
            RandomList::new, Teller::new, 4);
        // Demonstrate fill():
        List<Customer> customers = Suppliers.fill(
            new ArrayList<>(), Customer::new, 12);
        customers.forEach(c ->
            serve(tellers.select(), c));
        // Demonstrate assisted latent typing:
        Bank bank = Suppliers.fill(
            new Bank(), Bank::put, BankTeller::new, 3);
        // Can also use second version of fill():
        List<Customer> customers2 = Suppliers.fill(
            new ArrayList<>(),
            List::add, Customer::new, 12);
    }
}
/* Output:
Teller 3 serves Customer 1
Teller 2 serves Customer 2
Teller 3 serves Customer 3
Teller 1 serves Customer 4
Teller 1 serves Customer 5
Teller 3 serves Customer 6
Teller 1 serves Customer 7
Teller 2 serves Customer 8
Teller 3 serves Customer 9
Teller 3 serves Customer 10
Teller 2 serves Customer 11
Teller 4 serves Customer 12
*/
```

可以看到 `create()` 生成一個新的 **Collection** 物件，而 `fill()` 添加到現有 **Collection** 中。第二個版本`fill()` 顯示，它不僅與無關的新類型 **Bank** 一起使用，還能與 **List** 一起使用。因此，從技術上講，`fill()` 的第一個版本在技術上不是必需的，但在使用 **Collection** 時提供了較短的語法。

<!-- Summary: Is Casting Really So Bad? -->

## 總結：類型轉換真的如此之糟嗎？

自從 C++ 模版出現以來，我就一直在致力於解釋它，我可能比大多數人都更早地提出了下面的論點。直到最近，我才停下來，去思考這個論點到底在多少時間內是有效的——我將要描述的問題到底有多少次可以穿越障礙得以解決。

這個論點就是：使用泛型類型機制的最吸引人的地方，就是在使用集合類的地方，這些類包括諸如各種 **List** 、各種 **Set** 、各種 **Map** 等你在 [集合](book/12-Collections.md) 和 [附錄：集合主題](book/Appendix-Collection-Topics.md) 這兩章所見。在 Java 5 之前，當你將一個物件放置到集合中時，這個物件就會被向上轉型為 **Object** ，因此你會遺失類型訊息。當你想要將這個物件從集合中取回，用它去執行某些操作時，必須將其向下轉型回正確的類型。我用的範例是持有 **Cat** 的 **List** （這個範例的一種使用蘋果和桔子的變體在 [集合](book/12-Collections.md) 章節的開頭展示過）。如果沒有 Java 5 泛型版本的集合，你放到容集裡和從集合中取回的都是 **Object** 。因此，我們很可能會將一個 **Dog** 放置到 **Cat** 的 **List** 中。

但是，泛型出現之前的 Java 並不會讓你誤用放入到集合中的物件。如果將一個 **Dog** 扔到 **Cat** 的集合中，並且試圖將這個集合中的所有東西都當作 **Cat** 處理，那麼當你從這個 **Cat** 集合中取回那個 **Dog** 引用，並試圖將其轉型為 **Cat** 時，就會得到一個 **RuntimeException** 。你仍舊可以發現問題，但是是在執行時而非編譯期發現它的。

在本書以前的版本中，我曾經說過：

> 這不止令人惱火，它還可能會產生難以發現的缺陷。如果這個程式的某個部分（或數個部分）向集合中插入了物件，並且透過異常，你在程式的另一個獨立的部分中發現有不良物件被放置到了集合中，那麼必須發現這個不良插入到底是在何處發生的。
>

但是，隨著對這個論點的進一步檢查，我開始懷疑它了。首先，這會多麼頻繁地發生呢？我記得這類事情從未發生在我身上，並且當我在會議上詢問其他人時，我也從來沒有聽說過有人碰上過。另一本書使用了一個稱為 **files** 的 list 範例，它包含 **String** 物件。在這個範例中，向 **files** 中添加一個 **File** 物件看起來相當自然，因此這個物件的名字可能叫 **fileNames** 更好。無論 Java 提供了多少類型檢查，仍舊可能會寫出晦澀的程式，而編寫差勁的程式即便可以編譯，它仍舊是編寫差勁的程式。可能大多數人都會使用命名良好的集合，例如 **cats** ，因為它們可以向試圖添加非 **Cat** 物件的程式設計師提供可視的警告。並且即便這類事情發生了，它真正又能潛伏多久呢？只要你開始用真實資料來執行測試，就會非常快地看到異常。

有一位作者甚至斷言，這樣的缺陷將“*潛伏數年*”。但是我不記得有任何大量的相關報告，來說明人們在尋找“狗在貓列表中”這類缺陷時困難重重，或者是說明人們會非常頻繁地產生這種錯誤。然而，你將在 [多執行緒編程](book/24-Concurrent-Programming.md) 章節中看到，在使用執行緒時，出現那些可能看起來極罕見的缺陷，是很尋常並容易發生的事，而且，對於到底出了什麼錯，這些缺陷只能給你一個很模糊的概念。因此，對於泛型是添加到 Java 中的非常顯著和相當複雜的特性這一點，“狗在貓列表中”這個論據真的能夠成為它的理由嗎？
我相信被稱為*泛型*的通用語言特性（並非必須是其在 Java 中的特定實現）的目的在於可表達性，而不僅僅是為了建立類型安全的集合。類型安全的集合是能夠建立更通用程式碼這一能力所帶來的副作用。
因此，即便“狗在貓列表中”這個論據經常被用來證明泛型是必要的，但是它仍舊是有問題的。就像我在本章開頭聲稱的，我不相信這就是泛型這個概念真正的含義。相反，泛型正如其名稱所暗示的：它是一種方法，透過它可以編寫出更“泛化”的程式碼，這些程式碼對於它們能夠作用的類型具有更少的限制，因此單個的程式碼段可以應用到更多的類型上。正如你在本章中看到的，編寫真正泛化的“持有器”類（ Java 的容器就是這種類）相當簡單，但是編寫出能夠操作其泛型類型的泛化程式碼就需要額外的努力了，這些努力需要類建立者和類消費者共同付出，他們必須理解這些程式碼的概念和實現。這些額外的努力會增加使用這種特性的難度，並可能會因此而使其在某些場合缺乏可應用性，而在這些場合中，它可能會帶來附加的價值。

還要注意到，因為泛型是後來添加到 Java 中，而不是從一開始就設計到這種語言中的，所以某些容器無法達到它們應該具備的健壯性。例如，觀察一下 **Map** ，在特定的方法 `containsKey(Object key) `和 `get(Object key)` 中就包含這類情況。如果這些類是使用在它們之前就存在的泛型設計的，那麼這些方法將會使用參數化類型而不是 **Object** ，因此也就可以提供這些泛型假設會提供的編譯期檢查。例如，在 C++ 的 **map** 中，鍵的類型總是在編譯期檢查的。

有一件事很明顯：在一種語言已經被廣泛應用之後，在其較新的版本中引入任何種類的泛型機制，都會是一項非常非常棘手的任務，並且是一項不付出艱辛就無法完成的任務。在 C++ 中，模版是在其最初的 ISO 版本中就引入的（即便如此，也引發了陣痛，因為在第一個標準 C++ 出現之前，有很多非模版版本在使用），因此實際上模版一直都是這種語言的一部分。在 Java 中，泛型是在這種語言首次發布大約 10 年之後才引入的，因此向泛型遷移的問題特別多，並且對泛型的設計產生了明顯的影響。其結果就是，程式設計師將承受這些痛苦，而這一切都是由於 Java 設計者在設計 1.0 版本時所表現出來的短視造成的。當 Java 最初被建立時，它的設計者們當然了解 C++ 的模版，他們甚至考慮將其囊括到 Java 語言中，但是出於這樣或那樣的原因，他們決定將模版排除在外（其跡象就是他們過於匆忙）。因此， Java 語言和使用它的程式設計師都將承受這些痛苦。只有時間將會說明 Java 的泛型方式對這種語言所造成的最終影響。
某些語言，已經融入了更簡潔、影響更小的方式，來實現參數化類型。我們不可能不去想像這樣的語言將會成為 Java 的繼任者，因為它們採用的方式，與 C++ 通過 C 來實現的方式相同：按原樣使用它，然後對其進行改進。

## 進階閱讀

泛型的入門文件是 《Generics in the Java Programming Language》，作者是 Gilad Bracha，可以從 http://java.oracle.com 獲取。

Angelika Langer 的《Java Generics FAQs》是一份非常有幫助的資料，可以從 http://www.angelikalanger.com/GenericsFAQ/JavaGenericsFAQ.html 獲取。

你可以從 《Adding Wildcards to the Java Programming Language》中學到更多關於萬用字元的知識，作者是 Torgerson、Ernst、Hansen、von der Ahe、Bracha 和 Gafter，地址是 http://www.jot.fm/issues/issue_2004_12/article5。

Neal After 對於 Java 問題（尤其是擦除）的看法可以從這裡找到：http://www.infoq.com/articles/neal-gafter-on-java。

[^1]: 在編寫本章期間，Angelika Langer的 Java 泛型常見問題解答以及她的其他著作（與Klaus Kreft一起）是非常寶貴的。
[^2]: [http://gafter.blogspot.com/2004/09/puzzling-through-erasureanswer.html](http://gafter.blogspot.com/2004/09/puzzling-through-erasureanswer.html)
[^3]: 參見本章章末引文。
[^4]: 注意，一些編程環境，如 Eclipse 和 IntelliJ IDEA，將會自動生成委託程式碼。
[^5]: 因為可以使用轉型，有效地禁止了類型系統，一些人就認為 C++ 是弱類型，但這太極端了。一種可能更好的說法是 C++ 是有一道暗門的強類型語言。
[^6]: 我再次從 Brian Goetz 那獲得幫助。

<!-- 分頁 -->

<div style="page-break-after: always;"></div>
