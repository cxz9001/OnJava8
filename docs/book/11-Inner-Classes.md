[TOC]

<!-- Inner Classes -->

# 第十一章 內部類

> 一個定義在另一個類中的類，叫作內部類。

內部類是一種非常有用的特性，因為它允許你把一些邏輯相關的類組織在一起，並控制位於內部的類的可見性。然而必須要了解，內部類與組合是完全不同的概念，這一點很重要。在最初，內部類看起來就像是一種程式碼隱藏機制：將類置於其他類的內部。但是，你將會了解到，內部類遠不止如此，它了解外部類，並能與之通信，而且你用內部類寫出的程式碼更加優雅而清晰，儘管並不總是這樣（而且 Java 8 的 Lambda 表達式和方法引用減少了編寫內部類的需求）。

最初，內部類可能看起來有些奇怪，而且要花些時間才能在設計中輕鬆地使用它們。對內部類的需求並非總是很明顯的，但是在描述完內部類的基本語法與語義之後，就能明白使用內部類的好處了。

本章剩餘部分包含了對內部類語法更加詳盡的探索，這些特性是為了語言的完備性而設計的，但是你也許不需要使用它們，至少一開始不需要。因此，本章最初的部分也許就是你現在所需的全部，你可以將更詳盡的探索當作參考資料。


<!-- Creating Inner Classes -->
## 建立內部類

建立內部類的方式就如同你想的一樣——把類的定義置於外部類的裡面：

```java
// innerclasses/Parcel1.java
// Creating inner classes
public class Parcel1 {
    class Contents {
        private int i = 11;
      
        public int value() { return i; }
    }
  
    class Destination {
        private String label;
      
        Destination(String whereTo) {
            label = whereTo;
        }
      
        String readLabel() { return label; }
    }
    // Using inner classes looks just like
    // using any other class, within Parcel1:
    public void ship(String dest) {
        Contents c = new Contents();
        Destination d = new Destination(dest);
        System.out.println(d.readLabel());
    }
  
    public static void main(String[] args) {
        Parcel1 p = new Parcel1();
        p.ship("Tasmania");
    }
}
```

輸出為：

```
Tasmania
```

當我們在 `ship()` 方法裡面使用內部類的時候，與使用普通類沒什麼不同。在這裡，明顯的區別只是內部類的名字是嵌套在 **Parcel1** 裡面的。

更典型的情況是，外部類將有一個方法，該方法返回一個指向內部類的引用，就像在 `to()` 和 `contents()` 方法中看到的那樣：

```java
// innerclasses/Parcel2.java
// Returning a reference to an inner class
public class Parcel2 {
    class Contents {
        private int i = 11;
      
        public int value() { return i; }
    }
  
    class Destination {
        private String label;
      
        Destination(String whereTo) {
            label = whereTo;
        }
      
        String readLabel() { return label; }
    }
  
    public Destination to(String s) {
        return new Destination(s);
    }
  
    public Contents contents() {
        return new Contents();
    }
  
    public void ship(String dest) {
        Contents c = contents();
        Destination d = to(dest);
        System.out.println(d.readLabel());
    }
  
    public static void main(String[] args) {
        Parcel2 p = new Parcel2();
        p.ship("Tasmania");
        Parcel2 q = new Parcel2();
        // Defining references to inner classes:
        Parcel2.Contents c = q.contents();
        Parcel2.Destination d = q.to("Borneo");
    }
}
```

輸出為：

```
Tasmania
```

如果想從外部類的非靜態方法之外的任意位置建立某個內部類的物件，那麼必須像在 `main()` 方法中那樣，具體地指明這個物件的類型：*OuterClassName.InnerClassName*。(譯者註：在外部類的靜態方法中也可以直接指明類型 *InnerClassName*，在其他類中需要指明 *OuterClassName.InnerClassName*。)

<!-- The Link to the Outer Class -->

## 連結外部類

到目前為止，內部類似乎還只是一種名字隱藏和組織程式碼的模式。這些是很有用，但還不是最引人注目的，它還有其他的用途。當生成一個內部類的物件時，此物件與製造它的外部物件（enclosing object）之間就有了一種聯繫，所以它能訪問其外部物件的所有成員，而不需要任何特殊條件。此外，內部類還擁有其外部類的所有元素的訪問權。

```java
// innerclasses/Sequence.java
// Holds a sequence of Objects
interface Selector {
    boolean end();
    Object current();
    void next();
}
public class Sequence {
    private Object[] items;
    private int next = 0;
    public Sequence(int size) {
        items = new Object[size];
    }
    public void add(Object x) {
        if(next < items.length)
            items[next++] = x;
    }
    private class SequenceSelector implements Selector {
        private int i = 0;
        @Override
        public boolean end() { return i == items.length; }
        @Override
        public Object current() { return items[i]; }
        @Override
        public void next() { if(i < items.length) i++; }
    }
    public Selector selector() {
        return new SequenceSelector();
    }
    public static void main(String[] args) {
        Sequence sequence = new Sequence(10);
        for(int i = 0; i < 10; i++)
            sequence.add(Integer.toString(i));
        Selector selector = sequence.selector();
        while(!selector.end()) {
            System.out.print(selector.current() + " ");
            selector.next();
        }
    }
}
```

輸出為：

```
0 1 2 3 4 5 6 7 8 9
```

**Sequence** 類只是一個固定大小的 **Object** 的陣列，以類的形式包裝了起來。可以呼叫 `add()` 在序列末尾增加新的 **Object**（只要還有空間），要獲取 **Sequence** 中的每一個物件，可以使用 **Selector** 介面。這是“*疊代器*”設計模式的一個例子，在本書稍後的部分將更多地學習它。**Selector** 允許你檢查序列是否到末尾了（`end()`），訪問目前物件（`current()`），以及移到序列中的下一個物件（`next()`）。因為 **Selector** 是一個介面，所以別的類可以按它們自己的方式來實現這個介面，並且其他方法能以此介面為參數，來生成更加通用的程式碼。

這裡，**SequenceSelector** 是提供 **Selector** 功能的 **private** 類。可以看到，在 `main()` 中建立了一個 **Sequence**，並向其中添加了一些 **String** 物件。然後透過呼叫 `selector()` 獲取一個 **Selector**，並用它在 **Sequence** 中移動和選擇每一個元素。
最初看到 **SequenceSelector**，可能會覺得它只不過是另一個內部類罷了。但請仔細觀察它，注意方法 `end()`，`current()` 和 `next()` 都用到了 **items**，這是一個引用，它並不是 **SequenceSelector** 的一部分，而是外部類中的一個 **private** 欄位。然而內部類可以訪問其外部類的方法和欄位，就像自己擁有它們一樣，這帶來了很大的方便，就如前面的例子所示。

所以內部類自動擁有對其外部類所有成員的訪問權。這是如何做到的呢？當某個外部類的物件建立了一個內部類物件時，此內部類物件必定會秘密地捕獲一個指向那個外部類物件的引用。然後，在你訪問此外部類的成員時，就是用那個引用來選擇外部類的成員。幸運的是，編譯器會幫你處理所有的細節，但你現在可以看到：內部類的物件只能在與其外部類的物件相關聯的情況下才能被建立（就像你應該看到的，內部類是非 **static** 類時）。構建內部類物件時，需要一個指向其外部類物件的引用，如果編譯器訪問不到這個引用就會報錯。不過絕大多數時候這都無需程式設計師操心。

<!-- Using .this and .new -->
## 使用 .this 和 .new

如果你需要生成對外部類物件的引用，可以使用外部類的名字後面緊跟圓點和 **this**。這樣產生的引用自動地具有正確的類型，這一點在編譯期就被知曉並受到檢查，因此沒有任何執行時開銷。下面的範例展示了如何使用 **.this**：

```java
// innerclasses/DotThis.java
// Accessing the outer-class object
public class DotThis {
    void f() { System.out.println("DotThis.f()"); }
  
    public class Inner {
        public DotThis outer() {
            return DotThis.this;
            // A plain "this" would be Inner's "this"
        }
    }
  
    public Inner inner() { return new Inner(); }
  
    public static void main(String[] args) {
        DotThis dt = new DotThis();
        DotThis.Inner dti = dt.inner();
        dti.outer().f();
    }
}
```

輸出為：

```
DotThis.f()
```

有時你可能想要告知某些其他物件，去建立其某個內部類的物件。要實現此目的，你必須在 **new** 表達式中提供對其他外部類物件的引用，這是需要使用 **.new** 語法，就像下面這樣：

```java
// innerclasses/DotNew.java
// Creating an inner class directly using .new syntax
public class DotNew {
    public class Inner {}
    public static void main(String[] args) {
        DotNew dn = new DotNew();
        DotNew.Inner dni = dn.new Inner();
    }
}
```

要想直接建立內部類的物件，你不能按照你想像的方式，去引用外部類的名字 **DotNew**，而是必須使用外部類的物件來建立該內部類物件，就像在上面的程式中所看到的那樣。這也解決了內部類名字作用域的問題，因此你不必聲明（實際上你不能聲明）dn.new DotNew.Inner。

下面你可以看到將 **.new** 應用於 Parcel 的範例：

```java
// innerclasses/Parcel3.java
// Using .new to create instances of inner classes
public class Parcel3 {
    class Contents {
        private int i = 11;
        public int value() { return i; }
    }
    class Destination {
        private String label;
        Destination(String whereTo) { label = whereTo; }
        String readLabel() { return label; }
    }
    public static void main(String[] args) {
        Parcel3 p = new Parcel3();
        // Must use instance of outer class
        // to create an instance of the inner class:
        Parcel3.Contents c = p.new Contents();
        Parcel3.Destination d =
                p.new Destination("Tasmania");
    }
}
```

在擁有外部類物件之前是不可能建立內部類物件的。這是因為內部類物件會暗暗地連接到建它的外部類物件上。但是，如果你建立的是嵌套類（靜態內部類），那麼它就不需要對外部類物件的引用。

<!-- Inner Classes and Upcasting -->

## 內部類與向上轉型

當將內部類向上轉型為其基類，尤其是轉型為一個介面的時候，內部類就有了用武之地。（從實現了某個介面的物件，得到對此介面的引用，與向上轉型為這個物件的基類，實質上效果是一樣的。）這是因為此內部類-某個介面的實現-能夠完全不可見，並且不可用。所得到的只是指向基類或介面的引用，所以能夠很方便地隱藏實現細節。

我們可以建立前一個範例的介面：

```java
// innerclasses/Destination.java
public interface Destination {
    String readLabel();
}
```

```java
// innerclasses/Contents.java
public interface Contents {
    int value();
}
```

現在 **Contents** 和 **Destination** 表示用戶端程式設計師可用的介面。記住，介面的所有成員自動被設定為 **public**。

當取得了一個指向基類或介面的引用時，甚至可能無法找出它確切的類型，看下面的例子：

```java
// innerclasses/TestParcel.java
class Parcel4 {
    private class PContents implements Contents {
        private int i = 11;
        @Override
        public int value() { return i; }
    }
    protected final class PDestination implements Destination {
        private String label;
        private PDestination(String whereTo) {
            label = whereTo;
        }
        @Override
        public String readLabel() { return label; }
    }
    public Destination destination(String s) {
        return new PDestination(s);
    }
    public Contents contents() {
        return new PContents();
    }
}
public class TestParcel {
    public static void main(String[] args) {
        Parcel4 p = new Parcel4();
        Contents c = p.contents();
        Destination d = p.destination("Tasmania");
        // Illegal -- can't access private class:
        //- Parcel4.PContents pc = p.new PContents();
    }
}
```

在 **Parcel4** 中，內部類 **PContents** 是 **private**，所以除了 **Parcel4**，沒有人能訪問它。普通（非內部）類的訪問權限不能被設為 **private** 或者 **protected**；他們只能設定為 **public** 或 **package** 訪問權限。

**PDestination** 是 **protected**，所以只有 **Parcel4** 及其子類、還有與 **Parcel4** 同一個包中的類（因為 **protected** 也給予了包訪問權）能訪問 **PDestination**，其他類都不能訪問 **PDestination**，這意味著，如果用戶端程式設計師想了解或訪問這些成員，那是要受到限制的。實際上，甚至不能向下轉型成 **private** 內部類（或 **protected** 內部類，除非是繼承自它的子類），因為不能訪問其名字，就像在 **TestParcel** 類中看到的那樣。

**private** 內部類給類的設計者提供了一種途徑，透過這種方式可以完全阻止任何依賴於類型的編碼，並且完全隱藏了實現的細節。此外，從用戶端程式設計師的角度來看，由於不能訪問任何新增加的、原本不屬於公共介面的方法，所以擴展介面是沒有價值的。這也給 Java 編譯器提供了生成高效程式碼的機會。

<!-- Inner Classes in Methods and Scopes -->

## 內部類方法和作用域

到目前為止，讀者所看到的只是內部類的典型用途。通常，如果所讀、寫的程式碼包含了內部類，那麼它們都是“平凡的”內部類，簡單並且容易理解。然而，內部類的語法重寫了大量其他的更加難以理解的技術。例如，可以在一個方法裡面或者在任意的作用域內定義內部類。

這麼做有兩個理由：

1. 如前所示，你實現了某類型的介面，於是可以建立並返回對其的引用。
2. 你要解決一個複雜的問題，想建立一個類來輔助你的解決方案，但是又不希望這個類是公共可用的。

在後面的例子中，先前的程式碼將被修改，以用來實現：

1. 一個定義在方法中的類。
2. 一個定義在作用域內的類，此作用域在方法的內部。
3. 一個實現了介面的匿名類。
4. 一個匿名類，它擴展了沒有預設構造器的類。
5. 一個匿名類，它執行欄位初始化。
6. 一個匿名類，它透過實例初始化實現構造（匿名內部類不可能有構造器）。

第一個例子展示了在方法的作用域內（而不是在其他類的作用域內）建立一個完整的類。這被稱作局部內部類：

```java
// innerclasses/Parcel5.java
// Nesting a class within a method
public class Parcel5 {
    public Destination destination(String s) {
        final class PDestination implements Destination {
            private String label;
          
            private PDestination(String whereTo) {
                label = whereTo;
            }
          
            @Override
            public String readLabel() { return label; }
        }
        return new PDestination(s);
    }
  
    public static void main(String[] args) {
        Parcel5 p = new Parcel5();
        Destination d = p.destination("Tasmania");
    }
}
```

**PDestination** 類是 `destination()` 方法的一部分，而不是 **Parcel5** 的一部分。所以，在 `destination()` 之外不能訪問 **PDestination**，注意出現在 **return** 語句中的向上轉型-返回的是 **Destination** 的引用，它是 **PDestination** 的基類。當然，在 `destination()` 中定義了內部類 **PDestination**，並不意味著一旦 `destination()` 方法執行完畢，**PDestination** 就不可用了。

你可以在同一個子目錄下的任意類中對某個內部類使用類標識符 **PDestination**，這並不會有命名衝突。

下面的例子展示了如何在任意的作用域內嵌入一個內部類：

```java
// innerclasses/Parcel6.java
// Nesting a class within a scope
public class Parcel6 {
    private void internalTracking(boolean b) {
        if(b) {
            class TrackingSlip {
                private String id;
                TrackingSlip(String s) {
                    id = s;
                }
                String getSlip() { return id; }
            }
            TrackingSlip ts = new TrackingSlip("slip");
            String s = ts.getSlip();
        }
        // Can't use it here! Out of scope:
        //- TrackingSlip ts = new TrackingSlip("x");
    }
    public void track() { internalTracking(true); }
    public static void main(String[] args) {
        Parcel6 p = new Parcel6();
        p.track();
    }
}
```

**TrackingSlip** 類被嵌入在 **if** 語句的作用域內，這並不是說該類的建立是有條件的，它其實與別的類一起編譯過了。然而，在定義 **Trackingslip** 的作用域之外，它是不可用的，除此之外，它與普通的類一樣。

<!-- Anonymous Inner Classes -->

## 匿名內部類

下面的例子看起來有點奇怪：

```java
// innerclasses/Parcel7.java
// Returning an instance of an anonymous inner class
public class Parcel7 {
    public Contents contents() {
        return new Contents() { // Insert class definition
            private int i = 11;
          
            @Override
            public int value() { return i; }
        }; // Semicolon required
    }
  
    public static void main(String[] args) {
        Parcel7 p = new Parcel7();
        Contents c = p.contents();
    }
}
```

`contents()` 方法將返回值的生成與表示這個返回值的類的定義結合在一起！另外，這個類是匿名的，它沒有名字。更糟的是，看起來似乎是你正要建立一個 **Contents** 物件。但是然後（在到達語句結束的分號之前）你卻說：“等一等，我想在這裡插入一個類的定義。”

這種奇怪的語法指的是：“建立一個繼承自 **Contents** 的匿名類的物件。”通過 **new** 表達式返回的引用被自動向上轉型為對 **Contents** 的引用。上述匿名內部類的語法是下述形式的簡化形式：

```java
// innerclasses/Parcel7b.java
// Expanded version of Parcel7.java
public class Parcel7b {
    class MyContents implements Contents {
        private int i = 11;
        @Override
        public int value() { return i; }
    }
  
    public Contents contents() {
        return new MyContents();
    }
  
    public static void main(String[] args) {
        Parcel7b p = new Parcel7b();
        Contents c = p.contents();
    }
}
```

在這個匿名內部類中，使用了預設的構造器來生成 **Contents**。下面的程式碼展示的是，如果你的基類需要一個有參數的構造器，應該怎麼辦：

```java
// innerclasses/Parcel8.java
// Calling the base-class constructor
public class Parcel8 {
    public Wrapping wrapping(int x) {
        // Base constructor call:
        return new Wrapping(x) { // [1]
            @Override
            public int value() {
                return super.value() * 47;
            }
        }; // [2]
    }
    public static void main(String[] args) {
        Parcel8 p = new Parcel8();
        Wrapping w = p.wrapping(10);
    }
}
```

- \[1\] 將合適的參數傳遞給基類的構造器。
- \[2\] 在匿名內部類末尾的分號，並不是用來標記此內部類結束的。實際上，它標記的是表達式的結束，只不過這個表達式正巧包含了匿名內部類罷了。因此，這與別的地方使用的分號是一致的。

儘管 **Wrapping** 只是一個具有具體實現的普通類，但它還是被匯出類當作公共“介面”來使用。

```java
// innerclasses/Wrapping.java
public class Wrapping {
    private int i;
    public Wrapping(int x) { i = x; }
    public int value() { return i; }
}
```

為了多樣性，**Wrapping** 擁有一個要求傳遞一個參數的構造器。

在匿名類中定義欄位時，還能夠對其執行初始化操作：

```java
// innerclasses/Parcel9.java
public class Parcel9 {
    // Argument must be final or "effectively final"
    // to use within the anonymous inner class:
    public Destination destination(final String dest) {
        return new Destination() {
            private String label = dest;
            @Override
            public String readLabel() { return label; }
        };
    }
    public static void main(String[] args) {
        Parcel9 p = new Parcel9();
        Destination d = p.destination("Tasmania");
    }
}
```

如果在定義一個匿名內部類時，它要使用一個外部環境（在本匿名內部類之外定義）物件，那麼編譯器會要求其（該物件）參數引用是 **final** 或者是 “effectively final”（也就是說，該參數在初始化後不能被重新賦值，所以可以當作 **final**）的，就像你在 `destination()` 的參數中看到的那樣。這裡省略掉 **final** 也沒問題，但通常加上 **final** 作為提醒比較好。

如果只是簡單地給一個欄位賦值，那麼此例中的方法是很好的。但是，如果想做一些類似構造器的行為，該怎麼辦呢？在匿名類中不可能有命名構造器（因為它根本沒名字！），但透過實例初始化，就能夠達到為匿名內部類建立一個構造器的效果，就像這樣：

```java
// innerclasses/AnonymousConstructor.java
// Creating a constructor for an anonymous inner class
abstract class Base {
    Base(int i) {
        System.out.println("Base constructor, i = " + i);
    }
    public abstract void f();
}
public class AnonymousConstructor {
    public static Base getBase(int i) {
        return new Base(i) {
            { System.out.println(
                    "Inside instance initializer"); }
            @Override
            public void f() {
                System.out.println("In anonymous f()");
            }
        };
    }
    public static void main(String[] args) {
        Base base = getBase(47);
        base.f();
    }
}
```

輸出為：

```
Base constructor, i = 47
Inside instance initializer
In anonymous f()
```

在此例中，不要求變數 **i** 一定是 **final** 的。因為 **i** 被傳遞給匿名類的基類的構造器，它並不會在匿名類內部被直接使用。

下例是帶實例初始化的"parcel"形式。注意 `destination()` 的參數必須是 **final** 的，因為它們是在匿名類內部使用的（譯者註：即使不加 **final**, Java 8 的編譯器也會為我們自動加上 **final**，以保證資料的一致性）。

```java
// innerclasses/Parcel10.java
// Using "instance initialization" to perform
// construction on an anonymous inner class
public class Parcel10 {
    public Destination
    destination(final String dest, final float price) {
        return new Destination() {
            private int cost;
            // Instance initialization for each object:
            {
                cost = Math.round(price);
                if(cost > 100)
                    System.out.println("Over budget!");
            }
            private String label = dest;
            @Override
            public String readLabel() { return label; }
        };
    }
    public static void main(String[] args) {
        Parcel10 p = new Parcel10();
        Destination d = p.destination("Tasmania", 101.395F);
    }
}
```

輸出為：

```
Over budget!
```

在實例初始化操作的內部，可以看到有一段程式碼，它們不能作為欄位初始化動作的一部分來執行（就是 **if** 語句）。所以對於匿名類而言，實例初始化的實際效果就是構造器。當然它受到了限制-你不能重載實例初始化方法，所以你僅有一個這樣的構造器。

匿名內部類與正規的繼承相比有些受限，因為匿名內部類要嘛繼承類，要嘛實現介面，但是不能兩者兼備。而且如果是實現介面，也只能實現一個介面。

<!-- Nested Classes -->

## 嵌套類

如果不需要內部類物件與其外部類物件之間有聯繫，那麼可以將內部類聲明為 **static**，這通常稱為*嵌套類*。想要理解 **static** 應用於內部類時的含義，就必須記住，普通的內部類物件隱式地儲存了一個引用，指向建立它的外部類物件。然而，當內部類是 **static** 的時，就不是這樣了。嵌套類意味著：

1. 建立嵌套類的物件時，不需要其外部類的物件。
2. 不能從嵌套類的物件中訪問非靜態的外部類物件。

嵌套類與普通的內部類還有一個區別。普通內部類的欄位與方法，只能放在類的外部層次上，所以普通的內部類不能有 **static** 資料和 **static** 欄位，也不能包含嵌套類。但是嵌套類可以包含所有這些東西：

```java
// innerclasses/Parcel11.java
// Nested classes (static inner classes)
public class Parcel11 {
    private static class ParcelContents implements Contents {
        private int i = 11;
        @Override
        public int value() { return i; }
    }
    protected static final class ParcelDestination
            implements Destination {
        private String label;
        private ParcelDestination(String whereTo) {
            label = whereTo;
        }
        @Override
        public String readLabel() { return label; }
        // Nested classes can contain other static elements:
        public static void f() {}
        static int x = 10;
        static class AnotherLevel {
            public static void f() {}
            static int x = 10;
        }
    }
    public static Destination destination(String s) {
        return new ParcelDestination(s);
    }
    public static Contents contents() {
        return new ParcelContents();
    }
    public static void main(String[] args) {
        Contents c = contents();
        Destination d = destination("Tasmania");
    }
}
```

在 `main()` 中，沒有任何 **Parcel11** 的物件是必需的；而是使用選取 **static** 成員的普通語法來呼叫方法-這些方法返回對 **Contents** 和 **Destination** 的引用。

就像你在本章前面看到的那樣，在一個普通的（非 **static**）內部類中，透過一個特殊的 **this** 引用可以連結到其外部類物件。嵌套類就沒有這個特殊的 **this** 引用，這使得它類似於一個 **static** 方法。

### 介面內部的類

嵌套類可以作為介面的一部分。你放到介面中的任何類都自動地是 **public** 和 **static** 的。因為類是 **static** 的，只是將嵌套類置於介面的命名空間內，這並不違反介面的規則。你甚至可以在內部類中實現其外部介面，就像下面這樣：

```java
// innerclasses/ClassInInterface.java
// {java ClassInInterface$Test}
public interface ClassInInterface {
    void howdy();
    class Test implements ClassInInterface {
        @Override
        public void howdy() {
            System.out.println("Howdy!");
        }
        public static void main(String[] args) {
            new Test().howdy();
        }
    }
}
```

輸出為：

```
Howdy!
```

如果你想要建立某些公共程式碼，使得它們可以被某個介面的所有不同實現所共用，那麼使用介面內部的嵌套類會顯得很方便。

我曾在本書中建議過，在每個類中都寫一個 `main()` 方法，用來測試這個類。這樣做有一個缺點，那就是必須帶著那些已編譯過的額外程式碼。如果這對你是個麻煩，那就可以使用嵌套類來放置測試程式碼。

```java
// innerclasses/TestBed.java
// Putting test code in a nested class
// {java TestBed$Tester}
public class TestBed {
    public void f() { System.out.println("f()"); }
    public static class Tester {
        public static void main(String[] args) {
            TestBed t = new TestBed();
            t.f();
        }
    }
}
```

輸出為：

```
f()
```

這生成了一個獨立的類 **TestBed$Tester**（要執行這個程式，執行 **java TestBed$Tester**，在 Unix/Linux 系統中需要轉義 **$**）。你可以使用這個類測試，但是不必在發布的產品中包含它，可以在打包產品前刪除 **TestBed$Tester.class**。

### 從多層嵌套類中訪問外部類的成員

一個內部類被嵌套多少層並不重要——它能透明地訪問所有它所嵌入的外部類的所有成員，如下所示：

```java
// innerclasses/MultiNestingAccess.java
// Nested classes can access all members of all
// levels of the classes they are nested within
class MNA {
    private void f() {}
    class A {
        private void g() {}
        public class B {
            void h() {
                g();
                f();
            }
        }
    }
}
public class MultiNestingAccess {
    public static void main(String[] args) {
        MNA mna = new MNA();
        MNA.A mnaa = mna.new A();
        MNA.A.B mnaab = mnaa.new B();
        mnaab.h();
    }
}
```

可以看到在 **MNA.A.B** 中，呼叫方法 `g()` 和 `f()` 不需要任何條件（即使它們被定義為 **private**）。這個例子同時展示了如何從不同的類裡建立多層嵌套的內部類物件的基本語法。"**.new**"語法能產生正確的作用域，所以不必在呼叫構造器時限定類名。

<!-- Why Inner Classes? -->

## 為什麼需要內部類

至此，我們已經看到了許多描述內部類的語法和語義，但是這並不能同答“為什麼需要內部類”這個問題。那麼，Java 設計者們為什麼會如此費心地增加這項基本的語言特性呢？

一般說來，內部類繼承自某個類或實現某個介面，內部類的程式碼操作建立它的外部類的物件。所以可以認為內部類提供了某種進入其外部類的視窗。

內部類必須要回答的一個問題是：如果只是需要一個對介面的引用，為什麼不透過外部類實現那個介面呢？答案是：“如果這能滿足需求，那麼就應該這樣做。”那麼內部類實現一個介面與外部類實現這個介面有什麼區別呢？答案是：後者不是總能享用到介面帶來的方便，有時需要用到介面的實現。所以，使用內部類最吸引人的原因是：

> 每個內部類都能獨立地繼承自一個（介面的）實現，所以無論外部類是否已經繼承了某個（介面的）實現，對於內部類都沒有影響。

如果沒有內部類提供的、可以繼承多個具體的或抽象的類的能力，一些設計與編程問題就很難解決。從這個角度看，內部類使得多重繼承的解決方案變得完整。介面解決了部分問題，而內部類有效地實現了“多重繼承”。也就是說，內部類允許繼承多個非介面類型（譯註：類或抽象類）。

為了看到更多的細節，讓我們考慮這樣一種情形：即必須在一個類中以某種方式實現兩個介面。由於介面的靈活性，你有兩種選擇；使用單一類，或者使用內部類：

```java
// innerclasses/mui/MultiInterfaces.java
// Two ways a class can implement multiple interfaces
// {java innerclasses.mui.MultiInterfaces}
package innerclasses.mui;
interface A {}
interface B {}
class X implements A, B {}
class Y implements A {
    B makeB() {
        // Anonymous inner class:
        return new B() {};
    }
}
public class MultiInterfaces {
    static void takesA(A a) {}
    static void takesB(B b) {}
    public static void main(String[] args) {
        X x = new X();
        Y y = new Y();
        takesA(x);
        takesA(y);
        takesB(x);
        takesB(y.makeB());
    }
}
```

當然，這裡假設在兩種方式下的程式碼結構都確實有邏輯意義。然而遇到問題的時候，通常問題本身就能給出某些指引，告訴你是應該使用單一類，還是使用內部類。但如果沒有任何其他限制，從實現的觀點來看，前面的例子並沒有什麼區別，它們都能正常運作。

如果擁有的是抽象的類或具體的類，而不是介面，那就只能使用內部類才能實現多重繼承：

```java
// innerclasses/MultiImplementation.java
// For concrete or abstract classes, inner classes
// produce "multiple implementation inheritance"
// {java innerclasses.MultiImplementation}
package innerclasses;

class D {}

abstract class E {}

class Z extends D {
    E makeE() {
      return new E() {};  
    }
}

public class MultiImplementation {
    static void takesD(D d) {}
    static void takesE(E e) {}
    
    public static void main(String[] args) {
        Z z = new Z();
        takesD(z);
        takesE(z.makeE());
    }
}
```

如果不需要解決“多重繼承”的問題，那麼自然可以用別的方式編碼，而不需要使用內部類。但如果使用內部類，還可以獲得其他一些特性：

1. 內部類可以有多個實例，每個實例都有自己的狀態訊息，並且與其外部類物件的訊息相互獨立。
2. 在單個外部類中，可以讓多個內部類以不同的方式實現同一個介面，或繼承同一個類。
稍後就會展示一個這樣的例子。
3. 建立內部類物件的時刻並不依賴於外部類物件的建立
4. 內部類並沒有令人迷惑的"is-a”關係，它就是一個獨立的實體。

舉個例子，如果 **Sequence.java** 不使用內部類，就必須聲明"**Sequence** 是一個 **Selector**"，對於某個特定的 **Sequence** 只能有一個 **Selector**，然而使用內部類很容易就能擁有另一個方法 `reverseSelector()`，用它來生成一個反方向遍歷序列的 **Selector**，只有內部類才有這種靈活性。

### 閉包與回調

閉包（**closure**）是一個可呼叫的物件，它記錄了一些訊息，這些訊息來自於建立它的作用域。透過這個定義，可以看出內部類是物件導向的閉包，因為它不僅包含外部類物件（建立內部類的作用域）的訊息，還自動擁有一個指向此外部類物件的引用，在此作用域內，內部類有權操作所有的成員，包括 **private** 成員。

在 Java 8 之前，內部類是實現閉包的唯一方式。在 Java 8 中，我們可以使用 lambda 表達式來實現閉包行為，並且語法更加優雅和簡潔，你將會在 [函數式編程 ]() 這一章節中學習相關細節。儘管相對於內部類，你可能更喜歡使用 lambda 表達式實現閉包，但是你會看到並需要理解那些在 Java 8 之前透過內部類方式實現閉包的程式碼，因此仍然有必要來理解這種方式。

Java 最引人爭議的問題之一就是，人們認為 Java 應該包含某種類似指標的機制，以允許回調（callback）。透過回調，物件能夠攜帶一些訊息，這些訊息允許它在稍後的某個時刻呼叫初始的物件。稍後將會看到這是一個非常有用的概念。如果回調是透過指標實現的，那麼就只能寄希望於程式設計師不會誤用該指標。然而，讀者應該已經了解到，Java 更小心仔細，所以沒有在語言中包括指標。

透過內部類提供閉包的功能是優良的解決方案，它比指標更靈活、更安全。見一下例：

```java
// innerclasses/Callbacks.java
// Using inner classes for callbacks
// {java innerclasses.Callbacks}
package innerclasses;
interface Incrementable {
    void increment();
}
// Very simple to just implement the interface:
class Callee1 implements Incrementable {
    private int i = 0;
    @Override
    public void increment() {
        i++;
        System.out.println(i);
    }
}
class MyIncrement {
    public void increment() {
        System.out.println("Other operation");
    }
    static void f(MyIncrement mi) { mi.increment(); }
}
// If your class must implement increment() in
// some other way, you must use an inner class:
class Callee2 extends MyIncrement {
    private int i = 0;
    @Override
    public void increment() {
        super.increment();
        i++;
        System.out.println(i);
    }
    private class Closure implements Incrementable {
        @Override
        public void increment() {
            // Specify outer-class method, otherwise
            // you'll get an infinite recursion:
            Callee2.this.increment();
        }
    }
    Incrementable getCallbackReference() {
        return new Closure();
    }
}
class Caller {
    private Incrementable callbackReference;
    Caller(Incrementable cbh) {
        callbackReference = cbh;
    }
    void go() { callbackReference.increment(); }
}
public class Callbacks {
    public static void main(String[] args) {
        Callee1 c1 = new Callee1();
        Callee2 c2 = new Callee2();
        MyIncrement.f(c2);
        Caller caller1 = new Caller(c1);
        Caller caller2 =
                new Caller(c2.getCallbackReference());
        caller1.go();
        caller1.go();
        caller2.go();
        caller2.go();
    }
}
```

輸出為：

```java
Other operation
1
1
2
Other operation
2
Other operation
3
```

這個例子進一步展示了外部類實現一個介面與內部類實現此介面之間的區別。就程式碼而言，**Callee1** 是更簡單的解決方式。**Callee2** 繼承自 **MyIncrement**，後者已經有了一個不同的 `increment()` 方法，並且與 **Incrementable** 介面期望的 `increment()` 方法完全不相關。所以如果 **Callee2** 繼承了 **MyIncrement**，就不能為了 **Incrementable** 的用途而重寫 `increment()` 方法，於是只能使用內部類獨立地實現 **Incrementable**，還要注意，當建立了一個內部類時，並沒有在外部類的介面中添加東西，也沒有修改外部類的介面。

注意，在 **Callee2** 中除了 `getCallbackReference()` 以外，其他成員都是 **private** 的。要想建立與外部世界的任何連接，介面 **Incrementable** 都是必需的。在這裡可以看到，**interface** 是如何允許介面與介面的實現完全獨立的。
內部類 **Closure** 實現了 **Incrementable**，以提供一個返回 **Callee2** 的“鉤子”（hook）-而且是一個安全的鉤子。無論誰獲得此 **Incrementable** 的引用，都只能呼叫 `increment()`，除此之外沒有其他功能（不像指標那樣，允許你做很多事情）。

**Caller** 的構造器需要一個 **Incrementable** 的引用作為參數（雖然可以在任意時刻捕獲回調引用），然後在以後的某個時刻，**Caller** 物件可以使用此引用回調 **Callee** 類。

回調的價值在於它的靈活性-可以在執行時動態地決定需要呼叫什麼方法。例如，在圖形介面實現 GUI 功能的時候，到處都用到回調。

### 內部類與控制框架

在將要介紹的控制框架（control framework）中，可以看到更多使用內部類的具體例子。

應用程式框架（application framework）就是被設計用以解決某類特定問題的一個類或一組類。要運用某個應用程式框架，通常是繼承一個或多個類，並重寫某些方法。你在重寫的方法中寫的程式碼訂製了該應用程式框架提供的通用解決方案，來解決你的具體問題。這是設計模式中*模板方法*的一個例子，模板方法包含演算法的基本結構，而且會呼叫一個或多個可重寫的方法來完成該演算法的運算。設計模式總是將變化的事物與保持不變的事物分離開，在這個模式中，模板方法是保持不變的事物，而可重寫的方法就是變化的事物。

控制框架是一類特殊的應用程式框架，它用來解決響應事件的需求。主要用來響應事件的系統被稱作*事件驅動*系統。應用程式設計中常見的問題之一是圖形使用者介面（GUI），它幾乎完全是事件驅動的系統。

要理解內部類是如何允許簡單的建立過程以及如何使用控制框架的，請考慮這樣一個控制框架，它的工作就是在事件“就緒`ready()`”的時候執行事件。雖然“就緒”可以指任何事，但在本例中是指基於時間觸發的事件。下面是一個控制框架，它不包含具體的控制訊息。那些訊息是透過繼承(當演算法的 `action()` 部分被實現時)來提供的。

這裡是描述了所有控制事件的介面。之所以用抽象類代替了真正的介面，是因為預設行為都是根據時間來執行控制的。也因此包含了一些具體實現：

```java
// innerclasses/controller/Event.java
// The common methods for any control event
package innerclasses.controller;
import java.time.*; // Java 8 time classes
public abstract class Event {
    private Instant eventTime;
    protected final Duration delayTime;
    public Event(long millisecondDelay) {
        delayTime = Duration.ofMillis(millisecondDelay);
        start();
    }
    public void start() { // Allows restarting
        eventTime = Instant.now().plus(delayTime);
    }
    public boolean ready() {
        return Instant.now().isAfter(eventTime);
    }
    public abstract void action();
}
```

當希望執行 **Event** 並隨後呼叫 `start()` 時，那麼構造器就會捕獲（從物件建立的時刻開始的）時間，此時間是這樣得來的：`start()` 獲取目前時間，然後加上一個延遲時間，這樣生成觸發事件的時間。`start()` 是一個獨立的方法，而沒有包含在構造器內，因為這樣就可以在事件執行以後重新啟動計時器，也就是能夠重複使用 **Event** 物件。例如，如果想要重複一個事件，只需簡單地在 `action()` 中呼叫 `start()` 方法。

`ready()` 告訴你何時可以執行 `action()` 方法了。當然，可以在衍生類中重寫 `ready()` 方法，使得 **Event** 能夠基於時間以外的其他因素而觸發。

下面的文件包含了一個用來管理並觸發事件的實際控制框架。**Event** 物件被儲存在 **List**\<**Event**\> 類型（讀作“Event 的列表”）的容器物件中，容器會在 [集合 ]() 中詳細介紹。目前讀者只需要知道 `add()` 方法用來將一個 **Event** 添加到 **List** 的尾端，`size()` 方法用來得到 **List** 中元素的個數，foreach 語法用來連續獲取 **List** 中的 **Event**，`remove()` 方法用來從 **List** 中移除指定的 **Event**。

```java
// innerclasses/controller/Controller.java
// The reusable framework for control systems
package innerclasses.controller;
import java.util.*;
public class Controller {
    // A class from java.util to hold Event objects:
    private List<Event> eventList = new ArrayList<>();
    public void addEvent(Event c) { eventList.add(c); }
    public void run() {
        while(eventList.size() > 0)
            // Make a copy so you're not modifying the list
            // while you're selecting the elements in it:
            for(Event e : new ArrayList<>(eventList))
                if(e.ready()) {
                    System.out.println(e);
                    e.action();
                    eventList.remove(e);
                }
    }
}
```

`run()` 方法循環遍歷 **eventList**，尋找就緒的（`ready()`）、要執行的 **Event** 物件。對找到的每一個就緒的（`ready()`）事件，使用物件的 `toString()` 列印其訊息，呼叫其 `action()` 方法，然後從列表中移除此 **Event**。

注意，在目前的設計中你並不知道 **Event** 到底做了什麼。這正是此設計的關鍵所在—"使變化的事物與不變的事物相互分離”。用我的話說，“變化向量”就是各種不同的 **Event** 物件所具有的不同行為，而你透過建立不同的 **Event** 子類來表現不同的行為。

這正是內部類要做的事情，內部類允許：

1. 控制框架的完整實現是由單個的類建立的，從而使得實現的細節被封裝了起來。內部類用來表示解決問題所必需的各種不同的 `action()`。
2. 內部類能夠很容易地訪問外部類的任意成員，所以可以避免這種實現變得笨拙。如果沒有這種能力，程式碼將變得令人討厭，以至於你肯定會選擇別的方法。

考慮此控制框架的一個特定實現，如控制溫室的運作：控制燈光、水、溫度調節器的開關，以及響鈴和重新啟動系統，每個行為都是完全不同的。控制框架的設計使得分離這些不同的程式碼變得非常容易。使用內部類，可以在單一的類裡面產生對同一個基類 **Event** 的多種衍生版本。對於溫室系統的每一種行為，都繼承建立一個新的 **Event** 內部類，並在要實現的 `action()` 中編寫控制程式碼。

作為典型的應用程式框架，**GreenhouseControls** 類繼承自 **Controller**：

```java
// innerclasses/GreenhouseControls.java
// This produces a specific application of the
// control system, all in a single class. Inner
// classes allow you to encapsulate different
// functionality for each type of event.
import innerclasses.controller.*;
public class GreenhouseControls extends Controller {
    private boolean light = false;
    public class LightOn extends Event {
        public LightOn(long delayTime) {
            super(delayTime); 
        }
        @Override
        public void action() {
            // Put hardware control code here to
            // physically turn on the light.
            light = true;
        }
        @Override
        public String toString() {
            return "Light is on";
        }
    }
    public class LightOff extends Event {
        public LightOff(long delayTime) {
            super(delayTime);
        }
        @Override
        public void action() {
            // Put hardware control code here to
            // physically turn off the light.
            light = false;
        }
        @Override
        public String toString() {
            return "Light is off";
        }
    }
    private boolean water = false;
    public class WaterOn extends Event {
        public WaterOn(long delayTime) {
            super(delayTime);
        }
        @Override
        public void action() {
            // Put hardware control code here.
            water = true;
        }
        @Override
        public String toString() {
            return "Greenhouse water is on";
        }
    }
    public class WaterOff extends Event {
        public WaterOff(long delayTime) {
            super(delayTime);
        }
        @Override
        public void action() {
            // Put hardware control code here.
            water = false;
        }
        @Override
        public String toString() {
            return "Greenhouse water is off";
        }
    }
    private String thermostat = "Day";
    public class ThermostatNight extends Event {
        public ThermostatNight(long delayTime) {
            super(delayTime);
        }
        @Override
        public void action() {
            // Put hardware control code here.
            thermostat = "Night";
        }
        @Override
        public String toString() {
            return "Thermostat on night setting";
        }
    }
    public class ThermostatDay extends Event {
        public ThermostatDay(long delayTime) {
            super(delayTime);
        }
        @Override
        public void action() {
            // Put hardware control code here.
            thermostat = "Day";
        }
        @Override
        public String toString() {
            return "Thermostat on day setting";
        }
    }
    // An example of an action() that inserts a
    // new one of itself into the event list:
    public class Bell extends Event {
        public Bell(long delayTime) {
            super(delayTime);
        }
        @Override
        public void action() {
            addEvent(new Bell(delayTime.toMillis()));
        }
        @Override
        public String toString() {
            return "Bing!";
        }
    }
    public class Restart extends Event {
        private Event[] eventList;
        public
        Restart(long delayTime, Event[] eventList) {
            super(delayTime);
            this.eventList = eventList;
            for(Event e : eventList)
                addEvent(e);
        }
        @Override
        public void action() {
            for(Event e : eventList) {
                e.start(); // Rerun each event
                addEvent(e);
            }
            start(); // Rerun this Event
            addEvent(this);
        }
        @Override
        public String toString() {
            return "Restarting system";
        }
    }
    public static class Terminate extends Event {
        public Terminate(long delayTime) {
            super(delayTime);
        }
        @Override
        public void action() { System.exit(0); }
        @Override
        public String toString() {
            return "Terminating";
        }
    }
}
```

注意，**light**，**water** 和 **thermostat** 都屬於外部類 **GreenhouseControls**，而這些內部類能夠自由地訪問那些欄位，無需限定條件或特殊許可。而且，`action()` 方法通常都涉及對某種硬體的控制。

大多數 **Event** 類看起來都很相似，但是 **Bell** 和 **Restart** 則比較特別。**Bell** 控制響鈴，然後在事件列表中增加一個 **Bell** 物件，於是過一會它可以再次響鈴。讀者可能注意到了內部類是多麼像多重繼承：**Bell** 和 **Restart** 有 **Event** 的所有方法，並且似乎也擁有外部類 **GreenhouseContrlos** 的所有方法。

一個由 **Event** 物件組成的陣列被遞交給 **Restart**，該陣列要加到控制器上。由於 `Restart()` 也是一個 **Event** 物件，所以同樣可以將 **Restart** 物件添加到 `Restart.action()` 中，以使系統能夠有規律地重新啟動自己。

下面的類透過建立一個 **GreenhouseControls** 物件，並添加各種不同的 **Event** 物件來配置該系統，這是 *指令* 設計模式的一個例子—**eventList** 中的每個物件都被封裝成物件的請求：

```java
// innerclasses/GreenhouseController.java
// Configure and execute the greenhouse system
import innerclasses.controller.*;
public class GreenhouseController {
    public static void main(String[] args) {
        GreenhouseControls gc = new GreenhouseControls();
        // Instead of using code, you could parse
        // configuration information from a text file:
        gc.addEvent(gc.new Bell(900));
        Event[] eventList = {
                gc.new ThermostatNight(0),
                gc.new LightOn(200),
                gc.new LightOff(400),
                gc.new WaterOn(600),
                gc.new WaterOff(800),
                gc.new ThermostatDay(1400)
        };
        gc.addEvent(gc.new Restart(2000, eventList));
        gc.addEvent(
                new GreenhouseControls.Terminate(5000));
        gc.run();
    }
}
```

輸出為：

```
Thermostat on night setting
Light is on
Light is off
Greenhouse water is on
Greenhouse water is off
Bing!
Thermostat on day setting
Bing!
Restarting system
Thermostat on night setting
Light is on
Light is off
Greenhouse water is on
Bing!
Greenhouse water is off
Thermostat on day setting
Bing!
Restarting system
Thermostat on night setting
Light is on
Light is off
Bing!
Greenhouse water is on
Greenhouse water is off
Terminating
```

這個類的作用是初始化系統，所以它添加了所有相應的事件。**Restart** 事件反覆執行，而且它每次都會將 **eventList** 載入到 **GreenhouseControls** 物件中。如果提供了命令列參數，系統會以它作為毫秒數，決定什麼時候終止程式（這是測試程式時使用的）。

當然，更靈活的方法是避免對事件進行寫死。

這個例子應該使讀者更了解內部類的價值了，特別是在控制框架中使用內部類的時候。

<!-- Inheriting from Inner Classes -->

## 繼承內部類

因為內部類的構造器必須連接到指向其外部類物件的引用，所以在繼承內部類的時候，事情會變得有點複雜。問題在於，那個指向外部類物件的“秘密的”引用必須被初始化，而在衍生類中不再存在可連接的預設物件。要解決這個問題，必須使用特殊的語法來明確說清它們之間的關聯：

```java
// innerclasses/InheritInner.java
// Inheriting an inner class
class WithInner {
    class Inner {}
}
public class InheritInner extends WithInner.Inner {
    //- InheritInner() {} // Won't compile
    InheritInner(WithInner wi) {
        wi.super();
    }
    public static void main(String[] args) {
        WithInner wi = new WithInner();
        InheritInner ii = new InheritInner(wi);
    }
}
```

可以看到，**InheritInner** 只繼承自內部類，而不是外部類。但是當要生成一個構造器時，預設的構造器並不算好，而且不能只是傳遞一個指向外部類物件的引用。此外，必須在構造器內使用如下語法：

```java
enclosingClassReference.super();
```

這樣才提供了必要的引用，然後程式才能編譯通過。

<!-- Can Inner Classes Be Overridden? -->

## 內部類可以被重寫嗎？

如果建立了一個內部類，然後繼承其外部類並重新定義此內部類時，會發生什麼事呢？也就是說，內部類可以被重寫嗎？這看起來似乎是個很有用的思想，但是“重寫”內部類就好像它是外部類的一個方法，其實並不起什麼作用：

```java
// innerclasses/BigEgg.java
// An inner class cannot be overridden like a method
class Egg {
    private Yolk y;
    protected class Yolk {
        public Yolk() {
            System.out.println("Egg.Yolk()");
        }
    }
    Egg() {
        System.out.println("New Egg()");
        y = new Yolk();
    }
}
public class BigEgg extends Egg {
    public class Yolk {
        public Yolk() {
            System.out.println("BigEgg.Yolk()");
        }
    }
    public static void main(String[] args) {
        new BigEgg();
    }
}
```

輸出為：

```
New Egg()
Egg.Yolk()
```

預設的無參構造器是編譯器自動生成的，這裡是呼叫基類的預設構造器。你可能認為既然建立了 **BigEgg** 的物件，那麼所使用的應該是“重寫後”的 **Yolk** 版本，但從輸出中可以看到實際情況並不是這樣的。

這個例子說明，當繼承了某個外部類的時候，內部類並沒有發生什麼特別神奇的變化。這兩個內部類是完全獨立的兩個實體，各自在自己的命名空間內。當然，明確地繼承某個內部類也是可以的：

```java
// innerclasses/BigEgg2.java
// Proper inheritance of an inner class
class Egg2 {
    protected class Yolk {
        public Yolk() {
            System.out.println("Egg2.Yolk()");
        }
        public void f() {
            System.out.println("Egg2.Yolk.f()");
        }
    }
    private Yolk y = new Yolk();
    Egg2() { System.out.println("New Egg2()"); }
    public void insertYolk(Yolk yy) { y = yy; }
    public void g() { y.f(); }
}
public class BigEgg2 extends Egg2 {
    public class Yolk extends Egg2.Yolk {
        public Yolk() {
            System.out.println("BigEgg2.Yolk()");
        }
        @Override
        public void f() {
            System.out.println("BigEgg2.Yolk.f()");
        }
    }
    public BigEgg2() { insertYolk(new Yolk()); }
    public static void main(String[] args) {
        Egg2 e2 = new BigEgg2();
        e2.g();
    }
}
```

輸出為：

```
Egg2.Yolk()
New Egg2()
Egg2.Yolk()
BigEgg2.Yolk()
BigEgg2.Yolk.f()
```

現在 **BigEgg2.Yolk** 通過 **extends Egg2.Yolk** 明確地繼承了此內部類，並且重寫了其中的方法。`insertYolk()` 方法允許 **BigEgg2** 將它自己的 **Yolk** 物件向上轉型為 **Egg2** 中的引用 **y**。所以當 `g()` 呼叫 `y.f()` 時，重寫後的新版的 `f()` 被執行。第二次呼叫 `Egg2.Yolk()`，結果是 **BigEgg2.Yolk** 的構造器呼叫了其基類的構造器。可以看到在呼叫 `g()` 的時候，新版的 `f()` 被呼叫了。

<!-- Local Inner Classes -->

## 局部內部類

前面提到過，可以在程式碼塊裡建立內部類，典型的方式是在一個方法體的裡面建立。局部內部類不能有訪問說明符，因為它不是外部類的一部分；但是它可以訪問目前程式碼塊內的常量，以及此外部類的所有成員。下面的例子對局部內部類與匿名內部類的建立進行了比較。

```java
// innerclasses/LocalInnerClass.java
// Holds a sequence of Objects
interface Counter {
    int next();
}
public class LocalInnerClass {
    private int count = 0;
    Counter getCounter(final String name) {
        // A local inner class:
        class LocalCounter implements Counter {
            LocalCounter() {
                // Local inner class can have a constructor
                System.out.println("LocalCounter()");
            }
            @Override
            public int next() {
                System.out.print(name); // Access local final
                return count++;
            }
        }
        return new LocalCounter();
    }
    // Repeat, but with an anonymous inner class:
    Counter getCounter2(final String name) {
        return new Counter() {
            // Anonymous inner class cannot have a named
            // constructor, only an instance initializer:
            {
                System.out.println("Counter()");
            }
            @Override
            public int next() {
                System.out.print(name); // Access local final
                return count++;
            }
        };
    }
    public static void main(String[] args) {
        LocalInnerClass lic = new LocalInnerClass();
        Counter
                c1 = lic.getCounter("Local inner "),
                c2 = lic.getCounter2("Anonymous inner ");
        for(int i = 0; i < 5; i++)
            System.out.println(c1.next());
        for(int i = 0; i < 5; i++)
            System.out.println(c2.next());
    }
}
```

輸出為：

```
LocalCounter()
Counter()
Local inner 0
Local inner 1
Local inner 2
Local inner 3
Local inner 4
Anonymous inner 5
Anonymous inner 6
Anonymous inner 7
Anonymous inner 8
Anonymous inner 9
```

**Counter** 返回的是序列中的下一個值。我們分別使用局部內部類和匿名內部類實現了這個功能，它們具有相同的行為和能力，既然局部內部類的名字在方法外是不可見的，那為什麼我們仍然使用局部內部類而不是匿名內部類呢？唯一的理由是，我們需要一個已命名的構造器，或者需要重載構造器，而匿名內部類只能用於實例初始化。

所以使用局部內部類而不使用匿名內部類的另一個理由就是，需要不止一個該內部類的物件。

<!-- Inner-Class Identifiers -->

## 內部類標識符

由於編譯後每個類都會產生一個 **.class** 文件，其中包含了如何建立該類型的物件的全部訊息（此訊息產生一個"meta-class"，叫做 **Class** 物件）。

你可能猜到了，內部類也必須生成一個 **.class** 文件以包含它們的 **Class** 物件訊息。這些類文件的命名有嚴格的規則：外部類的名字，加上 **"$"** ，再加上內部類的名字。例如，**LocalInnerClass.java** 生成的 **.class** 文件包括：

```java
Counter.class
LocalInnerClass$1.class
LocalInnerClass$1LocalCounter.class
LocalInnerClass.class
```

如果內部類是匿名的，編譯器會簡單地產生一個數字作為其標識符。如果內部類是嵌套在別的內部類之中，只需直接將它們的名字加在其外部類標識符與 **"$"** 的後面。

雖然這種命名格式簡單而直接，但它還是很健壯的，足以應對絕大多數情況[^1]。因為這是 Java 的標準命名方式，所以產生的文件自動都是平台無關的。（注意，為了保證你的內部類能起作用，Java 編譯器會儘可能地轉換它們。）

[^1]: 另一方面，**$** 對Unix shell來說是一個元字元，所以當你列出.class文件時，有時會遇到麻煩。這對基於Unix的Sun公司來說有點奇怪。我的猜測是，他們沒有考慮這個問題，而是認為你會很自然地關注原始碼文件。

<!-- Summary -->
## 本章小結

比起物件導向編程中其他的概念來，介面和內部類更深奧複雜，比如 C++ 就沒有這些。將兩者結合起來，同樣能夠解決 C++ 中的用多重繼承所能解決的問題。然而，多重繼承在 C++ 中被證明是相當難以使用的，相比較而言，Java 的介面和內部類就容易理解多了。

雖然這些特性本身是相當直觀的，但是就像多態機制一樣，這些特性的使用應該是設計階段考慮的問題。隨著時間的推移，讀者將能夠更好地識別什麼情況下應該使用介面，什麼情況使用內部類，或者兩者同時使用。但此時，讀者至少應該已經完全理解了它們的語法和語義。

當讀者見到這些語言特性的實際應用時，就能最終理解它們了。

<!-- 分頁 -->

<div style="page-break-after: always;"></div>
