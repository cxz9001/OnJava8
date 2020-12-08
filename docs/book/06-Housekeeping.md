[TOC]

<!-- Housekeeping -->

# 第六章 初始化和清理

"不安全"的程式是造成編程代價昂貴的罪魁禍首之一。有兩個安全性問題：初始化和清理。C 語言中很多的 bug 都是因為程式設計師忘記初始化導致的。尤其是很多類庫的使用者不知道如何初始化類庫元件，甚至他們必須得去初始化。清理則是另一個特殊的問題，因為當你使用一個元素做完事後就不會去關心這個元素，所以你很容易忘記清理它。這樣就造成了元素使用的資源滯留不會被回收，直到程式消耗完所有的資源（特別是記憶體）。

C++ 引入了構造器的概念，這是一個特殊的方法，每建立一個物件，這個方法就會被自動呼叫。Java 採用了構造器的概念，另外還使用了垃圾收集器（Garbage Collector, GC）去自動回收不再被使用的物件所占的資源。這一章將討論初始化和清理的問題，以及在 Java 中對它們的支援。

<!-- Guaranteed Initialization with the Constructor -->

## 利用構造器保證初始化

你可能想為每個類建立一個 `initialize()` 方法，該方法名暗示著在使用類之前需要先呼叫它。不幸的是，使用者必須得記得去呼叫它。在 Java 中，類的設計者透過構造器保證每個物件的初始化。如果一個類有構造器，那麼 Java 會在使用者使用物件之前（即物件剛建立完成）自動呼叫物件的構造器方法，從而保證初始化。下個挑戰是如何命名構造器方法。存在兩個問題：第一個是任何命名都可能與類中其他已有元素的命名衝突；第二個是編譯器必須始終知道構造器方法名稱，從而呼叫它。C++ 的解決方法看起來是最簡單且最符合邏輯的，所以 Java 中使用了同樣的方式：構造器名稱與類名相同。在初始化過程中自動呼叫構造器方法是有意義的。

以下範例是包含了一個構造器的類：

```java
// housekeeping/SimpleConstructor.java
// Demonstration of a simple constructor

class Rock {
    Rock() { // 這是一個構造器
        System.out.print("Rock ");
    }
}

public class SimpleConstructor {
    public static void main(String[] args) {
        for (int i = 0; i < 10; i++) {
            new Rock();
        }
    }
}
```

輸出：

````java
Rock Rock Rock Rock Rock Rock Rock Rock Rock Rock 
````

現在，當建立一個物件時：`new Rock()` ，記憶體被分配，構造器被呼叫。構造器保證了物件在你使用它之前進行了正確的初始化。

有一點需要注意，構造器方法名與類名相同，不需要符合首字母小寫的程式風格。在 C++ 中，無參構造器被稱為預設構造器，這個術語在 Java 出現之前使用了很多年。但是，出於一些原因，Java 設計者們決定使用無參構造器這個名稱，我（作者）認為這種叫法笨拙而且沒有必要，所以我打算繼續使用預設構造器。Java 8 引入了 **default** 關鍵字修飾方法，所以算了，我還是用無參構造器的叫法吧。

跟其他方法一樣，構造器方法也可以傳入參數來定義如何建立一個物件。之前的例子稍作修改，使得構造器接收一個參數：

```java
// housekeeping/SimpleConstructor2.java
// Constructors can have arguments

class Rock2 {
    Rock2(int i) {
        System.out.print("Rock " + i + " ");
    }
}

public class SimpleConstructor2 {
    public static void main(String[] args) {
        for (int i = 0; i < 8; i++) {
            new Rock2(i);
        }
    }
}
```

輸出：

```java
Rock 0 Rock 1 Rock 2 Rock 3 Rock 4 Rock 5 Rock 6 Rock 7
```

如果類 **Tree** 有一個構造方法，只接收一個參數用來表示樹的高度，那麼你可以像下面這樣建立一棵樹:

```java
Tree t = new Tree(12); // 12-foot 樹
```

如果 **Tree(int)** 是唯一的構造器，那麼編譯器就不允許你以其他任何方式建立 **Tree** 類型的物件。

構造器消除了一類重要的問題，使得程式碼更易讀。例如，在上面的程式碼塊中，你看不到對 `initialize()` 方法的顯式呼叫，而從概念上來看，`initialize()` 方法應該與物件的建立分離。在 Java 中，物件的建立與初始化是統一的概念，二者不可分割。

構造器沒有返回值，它是一種特殊的方法。但它和返回類型為 `void` 的普通方法不同，普通方法可以返回空值，你還能選擇讓它返回別的類型；而構造器沒有返回值，卻同時也沒有給你選擇的餘地（`new` 表達式雖然返回了剛建立的物件的引用，但構造器本身卻沒有返回任何值）。如果它有返回值，並且你也可以自己選擇讓它返回什麼，那麼編譯器就還得知道接下來該怎麼處理那個返回值（這個返回值沒有接收者）。


<!-- Method Overloading -->

## 方法重載

任何程式語言中都具備的一項重要特性就是命名。當你建立一個物件時，就會給此物件分配的記憶體空間命名。方法是行為的命名。你透過名字指代所有的物件，屬性和方法。良好命名的系統易於理解和修改。就好比寫散文——目的是與讀者溝通。

將人類語言細微的差別映射到程式語言中會產生一個問題。通常，相同的詞可以表達多種不同的含義——它們被"重載"了。特別是當含義的差別很小時，這會更加有用。你會說"清洗襯衫"、"清洗車"和"清洗狗"。而如果硬要這麼說就會顯得很愚蠢："以洗襯衫的方式洗襯衫"、"以洗車的方式洗車"和"以洗狗的方式洗狗"，因為聽眾根本不需要區分行為的動作。大多數人類語言都具有"冗餘"性，所以即使漏掉幾個詞，你也能明白含義。你不需要對每個概念都使用不同的詞彙——可以從上下文推斷出含義。

大多數程式語言（尤其是 C 語言）要求為每個方法（在這些語言中經常稱為函數）提供一個獨一無二的標識符。所以，你不能有一個 `print()` 函數既能列印整型，也能列印浮點型——每個函數名都必須不同。

在 Java (C++) 中，還有一個因素也促使了必須使用方法重載：構造器。因為構造器方法名肯定是與類名相同，所以一個類中只會有一個構造器名。那麼你怎麼透過不同的方式建立一個物件呢？例如，你想建立一個類，這個類的初始化方式有兩種：一種是標準化方式，另一種是從文件中讀取訊息的方式。你需要兩個構造器：無參構造器和有一個 **String** 類型參數的構造器，該參數傳入檔案名。兩個構造器具有相同的名字——與類名相同。因此，方法重載是必要的，它允許方法具有相同的方法名但接收的參數不同。儘管方法重載對於構造器是重要的，但是也可以很方便地對其他任何方法進行重載。

下例展示了如何重載構造器和方法：

```java
// housekeeping/Overloading.java
// Both constructor and ordinary method overloading

class Tree {
    int height;
    Tree() {
        System.out.println("Planting a seedling");
        height = 0;
    }
    Tree(int initialHeight) {
        height = initialHeight;
        System.out.println("Creating new Tree that is " + height + " feet tall");
    }
    void info() {
        System.out.println("Tree is " + height + " feet tall");
    }
    void info(String s) {
        System.out.println(s + ": Tree is " + height + " feet tall");
    }
}
public class Overloading {
    public static void main(String[] args) {
        for (int i = 0; i < 5; i++) {
            Tree t = new Tree(i);
            t.info();
            t.info("overloaded method");
        }
        new Tree(); 
    }
}
```

輸出：

```java
Creating new Tree that is 0 feet tall
Tree is 0 feet tall
overloaded method: Tree is 0 feet tall
Creating new Tree that is 1 feet tall
Tree is 1 feet tall
overloaded method: Tree is 1 feet tall
Creating new Tree that is 2 feet tall
Tree is 2 feet tall
overloaded method: Tree is 2 feet tall
Creating new Tree that is 3 feet tall
Tree is 3 feet tall
overloaded method: Tree is 3 feet tall
Creating new Tree that is 4 feet tall
Tree is 4 feet tall
overloaded method: Tree is 4 feet tall
Planting a seedling
```

一個 **Tree** 物件既可以是一棵樹苗，使用無參構造器建立，也可以是一顆在溫室中已長大的樹，已經有一定高度，這時候，就需要使用有參構造器建立。

你也許想以多種方式呼叫 `info()` 方法。比如，如果你想列印額外的消息，就可以使用 `info(String)` 方法。如果你無話可說，就可以使用 `info()` 方法。用兩個命名定義完全相同的概念看起來很奇怪，而使用方法重載，你就可以使用一個命名來定義一個概念。

### 區分重載方法

如果兩個方法命名相同，Java是怎麼知道你呼叫的是哪個呢？有一條簡單的規則：每個被重載的方法必須有獨一無二的參數列表。你稍微思考一下，就會很明瞭了，除了透過參數列表的不同來區分兩個相同命名的方法，其他也沒什麼方式了。你甚至可以根據參數列表中的參數順序來區分不同的方法，儘管這會造成程式碼難以維護。例如：

```java
// housekeeping/OverloadingOrder.java
// Overloading based on the order of the arguments

public class OverloadingOrder {
    static void f(String s, int i) {
        System.out.println("String: " + s + ", int: " + i);
    }

    static void f(int i, String s) {
        System.out.println("int: " + i + ", String: " + s);
    }

    public static void main(String[] args) {
        f("String first", 1);
        f(99, "Int first");
    }
}
```

輸出：

```java
String: String first, int: 1
int: 99, String: Int first
```

兩個 `f()` 方法具有相同的參數，但是參數順序不同，根據這個就可以區分它們。

### 重載與基本類型

基本類型可以自動從較小的類型轉型為較大的類型。當這與重載結合時，這會令人有點困惑，下面是一個這樣的例子：

```java
// housekeeping/PrimitiveOverloading.java
// Promotion of primitives and overloading

public class PrimitiveOverloading {
    void f1(char x) {
        System.out.print("f1(char)");
    }
    void f1(byte x) {
        System.out.print("f1(byte)");
    }
    void f1(short x) {
        System.out.print("f1(short)");
    }
    void f1(int x) {
        System.out.print("f1(int)");
    }
    void f1(long x) {
        System.out.print("f1(long)");
    }
    void f1(float x) {
        System.out.print("f1(float)");
    }
    void f1(double x) {
        System.out.print("f1(double)");
    }
    void f2(byte x) {
        System.out.print("f2(byte)");
    }
    void f2(short x) {
        System.out.print("f2(short)");
    }
    void f2(int x) {
        System.out.print("f2(int)");
    }
    void f2(long x) {
        System.out.print("f2(long)");
    }
    void f2(float x) {
        System.out.print("f2(float)");
    }
    void f2(double x) {
        System.out.print("f2(double)");
    }
    void f3(short x) {
        System.out.print("f3(short)");
    }
    void f3(int x) {
        System.out.print("f3(int)");
    }
    void f3(long x) {
        System.out.print("f3(long)");
    }
    void f3(float x) {
        System.out.print("f3(float)");
    }
    void f3(double x) {
        System.out.print("f3(double)");
    }
    void f4(int x) {
        System.out.print("f4(int)");
    }
    void f4(long x) {
        System.out.print("f4(long)");
    }
    void f4(float x) {
        System.out.print("f4(float)");
    }
    void f4(double x) {
        System.out.print("f4(double)");
    }
    void f5(long x) {
        System.out.print("f5(long)");
    }
    void f5(float x) {
        System.out.print("f5(float)");
    }
    void f5(double x) {
        System.out.print("f5(double)");
    }
    void f6(float x) {
        System.out.print("f6(float)");
    }
    void f6(double x) {
        System.out.print("f6(double)");
    }
    void f7(double x) {
        System.out.print("f7(double)");
    }
    void testConstVal() {
        System.out.print("5: ");
        f1(5);f2(5);f3(5);f4(5);f5(5);f6(5);f7(5);
        System.out.println();
    }
    void testChar() {
        char x = 'x';
        System.out.print("char: ");
        f1(x);f2(x);f3(x);f4(x);f5(x);f6(x);f7(x);
        System.out.println();
    }
    void testByte() {
        byte x = 0;
        System.out.print("byte: ");
        f1(x);f2(x);f3(x);f4(x);f5(x);f6(x);f7(x);
        System.out.println();
    }
    void testShort() {
        short x = 0;
        System.out.print("short: ");
        f1(x);f2(x);f3(x);f4(x);f5(x);f6(x);f7(x);
        System.out.println();
    }
    void testInt() {
        int x = 0;
        System.out.print("int: ");
        f1(x);f2(x);f3(x);f4(x);f5(x);f6(x);f7(x);
        System.out.println();
    }
    void testLong() {
        long x = 0;
        System.out.print("long: ");
        f1(x);f2(x);f3(x);f4(x);f5(x);f6(x);f7(x);
        System.out.println();
    }
    void testFloat() {
        float x = 0;
        System.out.print("float: ");
        f1(x);f2(x);f3(x);f4(x);f5(x);f6(x);f7(x);
        System.out.println();
    }
    void testDouble() {
        double x = 0;
        System.out.print("double: ");
        f1(x);f2(x);f3(x);f4(x);f5(x);f6(x);f7(x);
        System.out.println();
    }

    public static void main(String[] args) {
        PrimitiveOverloading p = new PrimitiveOverloading();
        p.testConstVal();
        p.testChar();
        p.testByte();
        p.testShort();
        p.testInt();
        p.testLong();
        p.testFloat();
        p.testDouble();
    }
}
```

輸出：

```java
5: f1(int)f2(int)f3(int)f4(int)f5(long)f6(float)f7(double)
char: f1(char)f2(int)f3(int)f4(int)f5(long)f6(float)f7(double)
byte: f1(byte)f2(byte)f3(short)f4(int)f5(long)f6(float)f7(double)
short: f1(short)f2(short)f3(short)f4(int)f5(long)f6(float)f7(double)
int: f1(int)f2(int)f3(int)f4(int)f5(long)f6(float)f7(double)
long: f1(long)f2(long)f3(long)f4(long)f5(long)f6(float)f7(double)
float: f1(float)f2(float)f3(float)f4(float)f5(float)f6(float)f7(double)
double: f1(double)f2(double)f3(double)f4(double)f5(double)f6(double)f7(double)
```

如果傳入的參數類型大於方法期望接收的參數類型，你必須首先做一下轉換，如果你不做的話，編譯器就會報錯。

### 返回值的重載

經常會有人困惑，"為什麼只能透過方法名和參數列表，不能透過方法名和返回值區分方法呢?"。例如以下兩個方法，它們有相同的命名和參數，但是很容易區分：

```java
void f(){}
int f() {return 1;}
```

有些情況下，編譯器很容易就可以從上下文準確推斷出該呼叫哪個方法，如 `int x = f()`。

但是，你可以呼叫一個方法且忽略返回值。這叫做呼叫一個函數的副作用，因為你不在乎返回值，只是想利用方法做些事。所以如果你直接呼叫 `f()`，Java 編譯器就不知道你想呼叫哪個方法，閱讀者也不明所以。因為這個原因，所以你不能根據返回值類型區分重載的方法。為了支援新特性，Java 8 在一些具體情形下提高了猜測的準確度，但是通常來說並不起作用。

<!-- No-arg Constructors -->

## 無參構造器

如前文所說，一個無參構造器就是不接收參數的構造器，用來建立一個"預設的物件"。如果你建立一個類，類中沒有構造器，那麼編譯器就會自動為你建立一個無參構造器。例如：

```java
// housekeeping/DefaultConstructor.java
class Bird {}
public class DefaultConstructor {
    public static void main(String[] args) {
        Bird bird = new Bird(); // 預設的
    }
}
```

表達式 `new Bird()` 建立了一個新物件，呼叫了無參構造器，儘管在 **Bird** 類中並沒有顯式的定義無參構造器。試想如果沒有構造器，我們如何建立一個物件呢。但是,一旦你顯式地定義了構造器（無論有參還是無參），編譯器就不會自動為你建立無參構造器。如下：

```java
// housekeeping/NoSynthesis.java
class Bird2 {
    Bird2(int i) {}
    Bird2(double d) {}
}
public class NoSynthesis {
    public static void main(String[] args) {
        //- Bird2 b = new Bird2(); // No default
        Bird2 b2 = new Bird2(1);
        Bird2 b3 = new Bird2(1.0);
    }
}
```

如果你呼叫了 `new Bird2()` ，編譯器會提示找不到匹配的構造器。當類中沒有構造器時，編譯器會說"你一定需要構造器，那麼讓我為你建立一個吧"。但是如果類中有構造器，編譯器會說"你已經寫了構造器了，所以肯定知道你在做什麼，如果你沒有建立預設構造器，說明你本來就不需要"。

<!-- The this Keyword -->

## this關鍵字

對於兩個相同類型的物件 **a** 和 **b**，你可能在想如何呼叫這兩個物件的 `peel()` 方法：

```java
// housekeeping/BananaPeel.java

class Banana {
    void peel(int i) {
        /*...*/
    }
}
public class BananaPeel {
    public static void main(String[] args) {
        Banana a = new Banana(), b = new Banana();
        a.peel(1);
        b.peel(2);
    }
}
```

如果只有一個方法 `peel()` ，那麼怎麼知道呼叫的是物件 **a** 的 `peel()`方法還是物件 **b** 的 `peel()` 方法呢？編譯器做了一些底層工作，所以你可以像這樣編寫程式碼。`peel()` 方法中第一個參數隱密地傳入了一個指向操作物件的

引用。因此，上述例子中的方法呼叫像下面這樣：

```java
Banana.peel(a, 1)
Banana.peel(b, 2)
```

這是在內部實現的，你不可以直接這麼編寫程式碼，編譯器不會接受，但能說明到底發生了什麼事。假設現在在方法內部，你想獲得對目前物件的引用。但是，物件引用是被秘密地傳達給編譯器——並不在參數列表中。方便的是，有一個關鍵字: **this** 。**this** 關鍵字只能在非靜態方法內部使用。當你呼叫一個物件的方法時，**this** 生成了一個物件引用。你可以像對待其他引用一樣對待這個引用。如果你在一個類的方法裡呼叫該類的其他方法，不要使用 **this**，直接呼叫即可，**this** 自動地應用於其他方法上了。因此你可以像這樣：

```java
// housekeeping/Apricot.java

public class Apricot {
    void pick() {
        /* ... */
    }

    void pit() {
        pick();
        /* ... */
    }
}
```

在 `pit()` 方法中，你可以使用 `this.pick()`，但是沒有必要。編譯器自動為你做了這些。**this** 關鍵字只用在一些必須顯式使用目前物件引用的特殊場合。例如，用在 **return** 語句中返回對目前物件的引用。

```java
// housekeeping/Leaf.java
// Simple use of the "this" keyword

public class Leaf {

    int i = 0;

    Leaf increment() {
        i++;
        return this;
    }

    void print() {
        System.out.println("i = " + i);
    }

    public static void main(String[] args) {
        Leaf x = new Leaf();
        x.increment().increment().increment().print();
    }
}
```

輸出：

```
i = 3
```

因為 `increment()` 通過 **this** 關鍵字返回目前物件的引用，因此在相同的物件上可以輕易地執行多次操作。

**this** 關鍵字在向其他方法傳遞目前物件時也很有用：

```java
// housekeeping/PassingThis.java

class Person {
    public void eat(Apple apple) {
        Apple peeled = apple.getPeeled();
        System.out.println("Yummy");
    }
}

public class Peeler {
    static Apple peel(Apple apple) {
        // ... remove peel
        return apple; // Peeled
    }
}

public class Apple {
    Apple getPeeled() {
        return Peeler.peel(this);
    }
}

public class PassingThis {
    public static void main(String[] args) {
        new Person().eat(new Apple());
    }
}
```

輸出：

```
Yummy
```

**Apple** 因為某些原因（比如說工具類中的方法在多個類中重複出現，你不想程式碼重複），必須呼叫一個外部工具方法 `Peeler.peel()` 做一些行為。必須使用 **this** 才能將自身傳遞給外部方法。

### 在構造器中呼叫構造器

當你在一個類中寫了多個構造器，有時你想在一個構造器中呼叫另一個構造器來避免程式碼重複。你透過 **this** 關鍵字實現這樣的呼叫。

通常當你說 **this**，意味著"這個物件"或"目前物件"，它本身生成對目前物件的引用。在一個構造器中，當你給 **this** 一個參數列表時，它是另一層意思。它透過最直接的方式顯式地呼叫匹配參數列表的構造器：

```java
// housekeeping/Flower.java
// Calling constructors with "this"

public class Flower {
    int petalCount = 0;
    String s = "initial value";

    Flower(int petals) {
        petalCount = petals;
        System.out.println("Constructor w/ int arg only, petalCount = " + petalCount);
    }

    Flower(String ss) {
        System.out.println("Constructor w/ string arg only, s = " + ss);
        s = ss;
    }

    Flower(String s, int petals) {
        this(petals);
        //- this(s); // Can't call two!
        this.s = s; // Another use of "this"
        System.out.println("String & int args");
    }

    Flower() {
        this("hi", 47);
        System.out.println("no-arg constructor");
    }

    void printPetalCount() {
        //- this(11); // Not inside constructor!
        System.out.println("petalCount = " + petalCount + " s = " + s);
    }

    public static void main(String[] args) {
        Flower x = new Flower();
        x.printPetalCount();
    }
}
```

輸出：

```
Constructor w/ int arg only, petalCount = 47
String & int args
no-arg constructor
petalCount = 47 s = hi
```

從構造器 `Flower(String s, int petals)` 可以看出，其中只能透過 **this** 呼叫一次構造器。另外，必須首先呼叫構造器，否則編譯器會報錯。這個例子同樣展示了 **this** 的另一個用法。參數列表中的變數名 **s** 和成員變數名 **s** 相同，會引起混淆。你可以透過 `this.s` 表明你指的是成員變數 **s**，從而避免重複。你經常會在 Java 程式碼中看到這種用法，同時本書中也會多次出現這種寫法。在 `printPetalCount()` 方法中，編譯器不允許你在一個構造器之外的方法裡呼叫構造器。

### static 的含義

記住了 **this** 關鍵字的內容，你會對 **static** 修飾的方法有更加深入的理解：**static** 方法中不會存在 **this**。你不能在靜態方法中呼叫非靜態方法（反之可以）。靜態方法是為類而建立的，不需要任何物件。事實上，這就是靜態方法的主要目的，靜態方法看起來就像全域方法一樣，但是 Java 中不允許全域方法，一個類中的靜態方法可以訪問其他靜態方法和靜態屬性。一些人認為靜態方法不是物件導向的，因為它們的確具有全域方法的語義。使用靜態方法，因為不存在 **this**，所以你沒有向一個物件發送消息。的確，如果你發現程式碼中出現了大量的 **static** 方法，就該重新考慮自己的設計了。然而，**static** 的概念很實用，許多時候都要用到它。至於它是否真的"物件導向"，就留給理論家去討論吧。

<!-- Cleanup: Finalization and Garbage Collection -->

## 垃圾回收器

程式設計師都了解初始化的重要性，但通常會忽略清理的重要性。畢竟，誰會去清理一個 **int** 呢？但是使用完一個物件就不管它並非總是安全的。Java 中有垃圾回收器回收無用物件占用的記憶體。但現在考慮一種特殊情況：你建立的物件不是透過 **new** 來分配記憶體的，而垃圾回收器只知道如何釋放用 **new** 建立的物件的記憶體，所以它不知道如何回收不是 **new** 分配的記憶體。為了處理這種情況，Java 允許在類中定義一個名為 `finalize()` 的方法。

它的工作原理"假定"是這樣的：當垃圾回收器準備回收物件的記憶體時，首先會呼叫其 `finalize()` 方法，並在下一輪的垃圾回收動作發生時，才會真正回收物件占用的記憶體。所以如果你打算使用 `finalize()` ，就能在垃圾回收時做一些重要的清理工作。`finalize()` 是一個潛在的程式陷阱，因為一些程式設計師（尤其是 C++ 程式設計師）會一開始把它誤認為是 C++ 中的解構子（C++ 在銷毀物件時會呼叫這個函數）。所以有必要明確區分一下：在 C++ 中，物件總是被銷毀的（在一個 bug-free 的程式中），而在 Java 中，物件並非總是被垃圾回收，或者換句話說：

1. 物件可能不被垃圾回收。
2. 垃圾回收不等同於析構。

這意味著在你不再需要某個物件之前，如果必須執行某些動作，你得自己去做。Java 沒有析構器或類似的概念，所以你必須得自己建立一個普通的方法完成這項清理工作。例如，物件在建立的過程中會將自己繪製到螢幕上。如果不是明確地從螢幕上將其擦除，它可能永遠得不到清理。如果在 `finalize()` 方法中加入某種擦除功能，那麼當垃圾回收發生時，`finalize()` 方法被呼叫（不保證一定會發生），圖像就會被擦除，要是"垃圾回收"沒有發生，圖像則仍會保留下來。

也許你會發現，只要程式沒有瀕臨記憶體用完的那一刻，物件占用的空間就總也得不到釋放。如果程式執行結束，而垃圾回收器一直沒有釋放你建立的任何物件的記憶體，則當程式退出時，那些資源會全部交還給作業系統。這個策略是恰當的，因為垃圾回收本身也有開銷，要是不使用它，那就不用支付這部分開銷了。

### `finalize()` 的用途

如果你不能將 `finalize()` 作為通用的清理方法，那麼這個方法有什麼用呢？

這引入了要記住的第3點：

3. 垃圾回收只與內存有關。

也就是說，使用垃圾回收的唯一原因就是為了回收程式不再使用的記憶體。所以對於與垃圾回收有關的任何行為來說（尤其是 `finalize()` 方法），它們也必須同記憶體及其回收有關。

但這是否意味著如果物件中包括其他物件，`finalize()` 方法就應該明確釋放那些物件呢？不是，無論物件是如何建立的，垃圾回收器都會負責釋放物件所占用的所有記憶體。這就將對 `finalize()` 的需求限制到一種特殊情況，即透過某種建立物件方式之外的方式為物件分配了儲存空間。不過，你可能會想，Java 中萬物皆物件，這種情況怎麼可能發生？

看起來之所以有 `finalize()` 方法，是因為在分配記憶體時可能採用了類似 C 語言中的做法，而非 Java 中的通常做法。這種情況主要發生在使用"本地方法"的情況下，本地方法是一種用 Java 語言呼叫非 Java 語言代碼的形式（關於本地方法的討論，見本書電子版第2版的附錄B）。本地方法目前只支援 C 和 C++，但是它們可以呼叫其他語言寫的程式碼，所以實際上可以呼叫任何程式碼。在非 Java 程式碼中，也許會呼叫 C 的 `malloc()` 函數系列來分配儲存空間，而且除非呼叫 `free()` 函數，不然儲存空間永遠得不到釋放，造成記憶體洩露。但是，`free()` 是 C 和 C++ 中的函數，所以你需要在 `finalize()` 方法裡用本地方法呼叫它。

讀到這裡，你可能明白了不會過多使用 `finalize()` 方法。對，它確實不是進行普通的清理工作的合適場所。那麼，普通的清理工作在哪裡執行呢？

### 你必須實施清理

要清理一個物件，使用者必須在需要清理的時候呼叫執行清理動作的方法。這聽起來相當直接，但卻與 C++ 中的"解構子"的概念稍有牴觸。在 C++ 中，所有物件都會被銷毀，或者說應該被銷毀。如果在 C++ 中建立了一個局部物件（在堆疊上建立，在 Java 中不行），此時的銷毀動作發生在以"右花括號"為邊界的、此物件作用域的末尾處。如果物件是用 **new** 建立的（類似於 Java 中），那麼當程式設計師呼叫 C++ 的 **delete** 操作符時（Java 中不存在），就會呼叫相應的解構子。如果程式設計師忘記呼叫 **delete**，那麼永遠不會呼叫解構子，這樣就會導致記憶體洩露，物件的其他部分也不會得到清理。這種 bug 很難跟蹤，也是讓 C++ 程式設計師轉向 Java 的一個主要因素。相反，在 Java 中，沒有用於釋放物件的 **delete**，因為垃圾回收器會幫助你釋放儲存空間。甚至可以膚淺地認為，正是由於垃圾回收的存在，使得 Java 沒有解構子。然而，隨著學習的深入，你會明白垃圾回收器的存在並不能完全替代解構子（而且絕對不能直接呼叫 `finalize()`，所以這也不是一種解決方案）。如果希望進行除釋放儲存空間之外的清理工作，還是得明確呼叫某個恰當的 Java 方法：這就等同於使用解構子了，只是沒有它方便。

記住，無論是"垃圾回收"還是"終結"，都不保證一定會發生。如果 Java 虛擬機（JVM）並未面臨記憶體耗盡的情形，它可能不會浪費時間執行垃圾回收以復原記憶體。

### 終結條件

通常，不能指望 `finalize()` ，你必須建立其他的"清理"方法，並明確地呼叫它們。所以看起來，`finalize()` 只對大部分程式設計師很難用到的一些晦澀記憶體清理裡有用了。但是，`finalize()` 還有一個有趣的用法，它不依賴於每次都要對 `finalize()` 進行呼叫，這就是物件終結條件的驗證。

當對某個物件不感興趣時——也就是它將被清理了，這個物件應該處於某種狀態，這種狀態下它占用的記憶體可以被安全地釋放掉。例如，如果物件代表了一個打開的文件，在物件被垃圾回收之前程式設計師應該關閉這個文件。只要物件中存在沒有被適當清理的部分，程式就存在很隱晦的 bug。`finalize()` 可以用來最終發現這個情況，儘管它並不總是被呼叫。如果某次 `finalize()` 的動作使得 bug 被發現，那麼就可以據此找出問題所在——這才是人們真正關心的。以下是個簡單的例子，示範了 `finalize()` 的可能使用方式：

```java
// housekeeping/TerminationCondition.java
// Using finalize() to detect a object that
// hasn't been properly cleaned up

import onjava.*;

class Book {
    boolean checkedOut = false;

    Book(boolean checkOut) {
        checkedOut = checkOut;
    }

    void checkIn() {
        checkedOut = false;
    }

    @Override
    protected void finalize() throws Throwable {
        if (checkedOut) {
            System.out.println("Error: checked out");
        }
        // Normally, you'll also do this:
        // super.finalize(); // Call the base-class version
    }
}

public class TerminationCondition {

    public static void main(String[] args) {
        Book novel = new Book(true);
        // Proper cleanup:
        novel.checkIn();
        // Drop the reference, forget to clean up:
        new Book(true);
        // Force garbage collection & finalization:
        System.gc();
        new Nap(1); // One second delay
    }

}
```

輸出：

```
Error: checked out
```

本例的終結條件是：所有的 **Book** 物件在被垃圾回收之前必須被登記。但在 `main()` 方法中，有一本書沒有登記。要是沒有 `finalize()` 方法來驗證終結條件，將會很難發現這個 bug。

你可能注意到使用了 `@Override`。`@` 意味著這是一個註解，註解是關於程式碼的額外訊息。在這裡，該註解告訴編譯器這不是偶然地重定義在每個物件中都存在的 `finalize()` 方法——程式設計師知道自己在做什麼。編譯器確保你沒有拼錯方法名，而且確保那個方法存在於基類中。註解也是對讀者的提醒，`@Override` 在 Java 5 引入，在 Java 7 中改善，本書通篇會出現。

注意，`System.gc()` 用於強制進行終結動作。但是即使不這麼做，只要重複地處理程序（假設程式將分配大量的儲存空間而導致垃圾回收動作的執行），最終也能找出錯誤的 **Book** 物件。

你應該總是假設基類版本的 `finalize()` 也要做一些重要的事情，使用 **super** 呼叫它，就像在 `Book.finalize()` 中看到的那樣。本例中，它被注釋掉了，因為它需要進行異常處理，而我們到現在還沒有涉及到。

### 垃圾回收器如何工作

如果你以前用過的語言，在堆上分配物件的代價十分高昂，你可能自然會覺得 Java 中所有物件（基本類型除外）在堆上分配的方式也十分高昂。然而，垃圾回收器能很明顯地提高物件的建立速度。這聽起來很奇怪——儲存空間的釋放影響了儲存空間的分配，但這確實是某些 Java 虛擬機的工作方式。這也意味著，Java 從堆空間分配的速度可以和其他語言在堆疊上分配空間的速度相媲美。

例如，你可以把 C++ 裡的堆想像成一個院子，裡面每個物件都負責管理自己的地盤。一段時間後，物件可能被銷毀，但地盤必須復用。在某些 Java 虛擬機中，堆的實現截然不同：它更像一個傳送帶，每分配一個新物件，它就向前移動一格。這意味著物件儲存空間的分配速度特別快。Java 的"堆指標"只是簡單地移動到尚未分配的區域，所以它的效率與 C++ 在堆疊上分配空間的效率相當。當然實際過程中，在簿記工作方面還有少量額外開銷，但是這部分開銷比不上尋找可用空間開銷大。

你可能意識到了，Java 中的堆並非完全像傳送帶那樣工作。要是那樣的話，勢必會導致頻繁的記憶體頁面調度——將其移進移出硬碟，因此會顯得需要擁有比實際需要更多的記憶體。頁面調度會顯著影響性能。最終，在建立了足夠多的物件後，記憶體資源被耗盡。其中的秘密在於垃圾回收器的介入。當它工作時，一邊回收記憶體，一邊使堆中的物件緊湊排列，這樣"堆指標"就可以很容易地移動到更靠近傳送帶的開始處，也就儘量避免了頁面錯誤。垃圾回收器透過重新排列物件，實現了一種高速的、有無限空間可分配的堆模型。

要想理解 Java 中的垃圾回收，先了解其他系統中的垃圾回收機制將會很有幫助。一種簡單但速度很慢的垃圾回收機制叫做*引用計數*。每個物件中含有一個引用計數器，每當有引用指向該物件時，引用計數加 1。當引用離開作用域或被置為 **null** 時，引用計數減 1。因此，管理引用計數是一個開銷不大但是在程式的整個生命週期頻繁發生的負擔。垃圾回收器會遍歷含有全部物件的列表，當發現某個物件的引用計數為 0 時，就釋放其占用的空間（但是，引用計數模式經常會在計數為 0 時立即釋放物件）。這個機制存在一個缺點：如果物件之間存在循環引用，那麼它們的引用計數都不為 0，就會出現應該被回收但無法被回收的情況。對垃圾回收器而言，定位這樣的循環引用所需的工作量極大。引用計數常用來說明垃圾回收的工作方式，但似乎從未被應用於任何一種 Java 虛擬機實現中。

在更快的策略中，垃圾回收器並非基於引用計數。它們依據的是：對於任意"活"的物件，一定能最終追溯到其存活在堆疊或靜態儲存區中的引用。這個引用鏈條可能會穿過數個物件層次，由此，如果從堆疊或靜態儲存區出發，遍歷所有的引用，你將會發現所有"活"的物件。對於發現的每個引用，必須追蹤它所引用的物件，然後是該物件包含的所有引用，如此反覆進行，直到訪問完"根源於堆疊或靜態儲存區的引用"所形成的整個網路。你所訪問過的物件一定是"活"的。注意，這解決了物件間循環引用的問題，這些物件不會被發現，因此也就被自動回收了。

在這種方式下，Java 虛擬機採用了一種*自適應*的垃圾回收技術。至於如何處理找到的存活物件，取決於不同的 Java 虛擬機實現。其中有一種做法叫做停止-複製（stop-and-copy）。顧名思義，這需要先暫停程式的執行（不屬於後台回收模式），然後將所有存活的物件從目前堆複製到另一個堆，沒有複製的就是需要被垃圾回收的。另外，當物件被複製到新堆時，它們是一個挨著一個緊湊排列，然後就可以按照前面描述的那樣簡單、直接地分配新空間了。

當物件從一處複製到另一處，所有指向它的引用都必須修正。位於堆疊或靜態儲存區的引用可以直接被修正，但可能還有其他指向這些物件的引用，它們在遍歷的過程中才能被找到（可以想像成一個表格，將舊地址映射到新地址）。

這種所謂的"複製回收器"效率低下主要因為兩個原因。其一：得有兩個堆，然後在這兩個分離的堆之間來回折騰，得維護比實際需要多一倍的空間。某些 Java 虛擬機對此問題的處理方式是，按需從堆中分配幾塊較大的記憶體，複製動作發生在這些大塊記憶體之間。

其二在於複製本身。一旦程式進入穩定狀態之後，可能只會產生少量垃圾，甚至沒有垃圾。儘管如此，複製回收器仍然會將所有記憶體從一處複製到另一處，這很浪費。為了避免這種狀況，一些 Java 虛擬機會進行檢查：要是沒有新垃圾產生，就會轉換到另一種模式（即"自適應"）。這種模式稱為標記-清掃（mark-and-sweep），Sun 公司早期版本的 Java 虛擬機一直使用這種技術。對一般用途而言，"標記-清掃"方式速度相當慢，但是當你知道程式只會產生少量垃圾甚至不產生垃圾時，它的速度就很快了。

"標記-清掃"所依據的思路仍然是從堆疊和靜態儲存區出發，遍歷所有的引用，找出所有存活的物件。但是，每當找到一個存活物件，就給物件設一個標記，並不回收它。只有當標記過程完成後，清理動作才開始。在清理過程中，沒有標記的物件將被釋放，不會發生任何複製動作。"標記-清掃"後剩下的堆空間是不連續的，垃圾回收器要是希望得到連續空間的話，就需要重新整理剩下的物件。

"停止-複製"指的是這種垃圾回收動作不是在後台進行的；相反，垃圾回收動作發生的同時，程式將會暫停。在 Oracle 公司的文件中會發現，許多參考文獻將垃圾回收視為低優先度的後台行程，但是早期版本的 Java 虛擬機並不是這麼實現垃圾回收器的。當可用記憶體較低時，垃圾回收器會暫停程式。同樣，"標記-清掃"工作也必須在程式暫停的情況下才能進行。

如前文所述，這裡討論的 Java 虛擬機中，記憶體分配以較大的"塊"為單位。如果物件較大，它會占用單獨的塊。嚴格來說，"停止-複製"要求在釋放舊物件之前，必須先將所有存活物件從舊堆複製到新堆，這導致了大量的記憶體複製行為。有了塊，垃圾回收器就可以把物件複製到廢棄的塊。每個塊都有年代數來記錄自己是否存活。通常，如果塊在某處被引用，其年代數加 1，垃圾回收器會對上次回收動作之後新分配的塊進行整理。這對處理大量短命的臨時物件很有幫助。垃圾回收器會定期進行完整的清理動作——大型物件仍然不會複製（只是年代數會增加），含有小型物件的那些塊則被複製並整理。Java 虛擬機會監視，如果所有物件都很穩定，垃圾回收的效率降低的話，就切換到"標記-清掃"方式。同樣，Java 虛擬機會跟蹤"標記-清掃"的效果，如果堆空間出現很多碎片，就會切換回"停止-複製"方式。這就是"自適應"的由來，你可以給它個囉嗦的稱呼："自適應的、分代的、停止-複製、標記-清掃"式的垃圾回收器。

Java 虛擬機中有許多附加技術用來提升速度。尤其是與載入器操作有關的，被稱為"即時"（Just-In-Time, JIT）編譯器的技術。這種技術可以把程式全部或部分翻譯成本機機器碼，所以不需要 JVM 來進行翻譯，因此執行得更快。當需要裝載某個類（通常是建立該類的第一個物件）時，編譯器會先找到其 **.class** 文件，然後將該類的位元組碼裝入記憶體。你可以讓即時編譯器編譯所有程式碼，但這種做法有兩個缺點：一是這種載入動作貫穿整個程式生命週期內，累加起來需要花更多時間；二是會增加可執行程式碼的長度（位元組碼要比即時編譯器展開後的本機機器碼小很多），這會導致頁面調度，從而一定降低程式速度。另一種做法稱為*惰性評估*，意味著即時編譯器只有在必要的時候才編譯程式碼。這樣，從未被執行的程式碼也許就根本不會被 JIT 編譯。新版 JDK 中的 Java HotSpot 技術就採用了類似的做法，程式碼每被執行一次就最佳化一些，所以執行的次數越多，它的速度就越快。

<!-- Member Initialization -->

## 成員初始化

Java 儘量保證所有變數在使用前都能得到恰當的初始化。對於方法的局部變數，這種保證會以編譯時錯誤的方式呈現，所以如果寫成：

```java
void f() {
    int i;
    i++;
}
```

你會得到一條錯誤訊息，告訴你 **i** 可能尚未初始化。編譯器可以為 **i** 賦一個預設值，但是未初始化的局部變數更有可能是程式設計師的疏忽，所以採用預設值反而會掩蓋這種失誤。強制程式設計師提供一個初始值，往往能幫助找出程式裡的 bug。

要是類的成員變數是基本類型，情況就會變得有些不同。正如在"萬物皆物件"一章中所看到的，類的每個基本類型資料成員保證都會有一個初始值。下面的程式可以驗證這類情況，並顯示它們的值：

```java
// housekeeping/InitialValues.java
// Shows default initial values

public class InitialValues {
    boolean t;
    char c;
    byte b;
    short s;
    int i;
    long l;
    float f;
    double d;
    InitialValues reference;

    void printInitialValues() {
        System.out.println("Data type Initial value");
        System.out.println("boolean " + t);
        System.out.println("char[" + c + "]");
        System.out.println("byte " + b);
        System.out.println("short " + s);
        System.out.println("int " + i);
        System.out.println("long " + l);
        System.out.println("float " + f);
        System.out.println("double " + d);
        System.out.println("reference " + reference);
    }

    public static void main(String[] args) {
        new InitialValues().printInitialValues();
    }
}
```

輸出：

```Java
Data type Initial value
boolean false
char[NUL]
byte 0
short 0
int 0
long 0
float 0.0
double 0.0
reference null
```

可見儘管資料成員的初值沒有給出，但它們確實有初值（char 值為 0，所以顯示為空白）。所以這樣至少不會出現"未初始化變數"的風險了。

在類裡定義一個物件引用時，如果不將其初始化，那麼引用就會被賦值為 **null**。

### 指定初始化

怎麼給一個變數賦初值呢？一種很直接的方法是在定義類成員變數的地方為其賦值。以下程式碼修改了 InitialValues 類成員變數的定義，直接提供了初值：

```java
// housekeeping/InitialValues2.java
// Providing explicit initial values

public class InitialValues2 {
    boolean bool = true;
    char ch = 'x';
    byte b = 47;
    short s = 0xff;
    int i = 999;
    long lng = 1;
    float f = 3.14f;
    double d = 3.14159;
}
```

你也可以用同樣的方式初始化非基本類型的物件。如果 **Depth** 是一個類，那麼可以像下面這樣建立一個物件並初始化它：

```java
// housekeeping/Measurement.java

class Depth {}

public class Measurement {
    Depth d = new Depth();
    // ...
}
```

如果沒有為 **d** 賦予初值就嘗試使用它，就會出現執行時錯誤，告訴你產生了一個異常（詳細見"異常"章節）。

你也可以透過呼叫某個方法來提供初值：

```java
// housekeeping/MethodInit.java

public class MethodInit {
    int i = f();
    
    int f() {
        return 11;
    }
    
}
```

這個方法可以帶有參數，但這些參數不能是未初始化的類成員變數。因此，可以這麼寫：

```java
// housekeeping/MethodInit2.java

public class MethodInit2 {
    int i = f();
    int j = g(i);
    
    int f() {
        return 11;
    }
    
    int g(int n) {
        return n * 10;
    }
}
```

但是你不能這麼寫：

```java
// housekeeping/MethodInit3.java

public class MethodInit3 {
    //- int j = g(i); // Illegal forward reference
    int i = f();

    int f() {
        return 11;
    }

    int g(int n) {
        return n * 10;
    }
}
```

顯然，上述程式的正確性取決於初始化的順序，而與其編譯方式無關。所以，編譯器恰當地對"向前引用"發出了警告。

這種初始化方式簡單直觀，但有個限制：類 **InitialValues** 的每個物件都有相同的初值，有時這的確是我們需要的，但有時卻需要更大的靈活性。

<!-- Constructor Initialization -->

## 構造器初始化

可以用構造器進行初始化，這種方式給了你更大的靈活性，因為你可以在執行時呼叫方法進行初始化。但是，這無法阻止自動初始化的進行，他會在構造器被呼叫之前發生。因此，如果使用如下程式碼：

```java
// housekeeping/Counter.java

public class Counter {
    int i;
    
    Counter() {
        i = 7;
    }
    // ...
}
```

**i** 首先會被初始化為 **0**，然後變為 **7**。對於所有的基本類型和引用，包括在定義時已明確指定初值的變數，這種情況都是成立的。因此，編譯器不會強制你一定要在構造器的某個地方或在使用它們之前初始化元素——初始化早已得到了保證。, 

### 初始化的順序

在類中變數定義的順序決定了它們初始化的順序。即使變數定義散布在方法定義之間，它們仍會在任何方法（包括構造器）被呼叫之前得到初始化。例如：

```java
// housekeeping/OrderOfInitialization.java
// Demonstrates initialization order
// When the constructor is called to create a
// Window object, you'll see a message:

class Window {
    Window(int marker) {
        System.out.println("Window(" + marker + ")");
    }
}

class House {
    Window w1 = new Window(1); // Before constructor

    House() {
        // Show that we're in the constructor:
        System.out.println("House()");
        w3 = new Window(33); // Reinitialize w3
    }

    Window w2 = new Window(2); // After constructor

    void f() {
        System.out.println("f()");
    }

    Window w3 = new Window(3); // At end
}

public class OrderOfInitialization {
    public static void main(String[] args) {
        House h = new House();
        h.f(); // Shows that construction is done
    }
}
```

輸出：

```
Window(1)
Window(2)
Window(3)
House()
Window(33)
f()
```

在 **House** 類中，故意把幾個 **Window** 物件的定義散布在各處，以證明它們全都會在呼叫構造器或其他方法之前得到初始化。此外，**w3** 在構造器中被再次賦值。

由輸出可見，引用 **w3** 被初始化了兩次：一次在呼叫構造器前，一次在構造器呼叫期間（第一次引用的物件將被丟棄，並作為垃圾回收）。這乍一看可能覺得效率不高，但保證了正確的初始化。試想，如果定義了一個重載構造器，在其中沒有初始化 **w3**，同時在定義 **w3** 時沒有賦予初值，那會產生怎樣的後果呢？

### 靜態資料的初始化

無論建立多少個物件，靜態資料都只占用一份儲存區域。**static** 關鍵字不能應用於局部變數，所以只能作用於屬性（欄位、域）。如果一個欄位是靜態的基本類型，你沒有初始化它，那麼它就會獲得基本類型的標準初值。如果它是物件引用，那麼它的預設初值就是 **null**。

如果在定義時進行初始化，那麼靜態變數看起來就跟非靜態變數一樣。

下面例子顯示了靜態儲存區是何時初始化的：

```java
// housekeeping/StaticInitialization.java
// Specifying initial values in a class definition

class Bowl {
    Bowl(int marker) {
        System.out.println("Bowl(" + marker + ")");
    }
    
    void f1(int marker) {
        System.out.println("f1(" + marker + ")");
    }
}

class Table {
    static Bowl bowl1 = new Bowl(1);
    
    Table() {
        System.out.println("Table()");
        bowl2.f1(1);
    }
    
    void f2(int marker) {
        System.out.println("f2(" + marker + ")");
    }
    
    static Bowl bowl2 = new Bowl(2);
}

class Cupboard {
    Bowl bowl3 = new Bowl(3);
    static Bowl bowl4 = new Bowl(4);
    
    Cupboard() {
        System.out.println("Cupboard()");
        bowl4.f1(2);
    }
    
    void f3(int marker) {
        System.out.println("f3(" + marker + ")");
    }
    
    static Bowl bowl5 = new Bowl(5);
}

public class StaticInitialization {
    public static void main(String[] args) {
        System.out.println("main creating new Cupboard()");
        new Cupboard();
        System.out.println("main creating new Cupboard()");
        new Cupboard();
        table.f2(1);
        cupboard.f3(1);
    }
    
    static Table table = new Table();
    static Cupboard cupboard = new Cupboard();
}
```

輸出：

```
Bowl(1)
Bowl(2)
Table()
f1(1)
Bowl(4)
Bowl(5)
Bowl(3)
Cupboard()
f1(2)
main creating new Cupboard()
Bowl(3)
Cupboard()
f1(2)
main creating new Cupboard()
Bowl(3)
Cupboard()
f1(2)
f2(1)
f3(1)
```

**Bowl** 類展示類的建立，而 **Table** 和 **Cupboard** 在它們的類定義中包含 **Bowl** 類型的靜態資料成員。注意，在靜態資料成員定義之前，**Cupboard** 類中先定義了一個 **Bowl** 類型的非靜態成員 **b3**。

由輸出可見，靜態初始化只有在必要時刻才會進行。如果不建立 **Table** 物件，也不引用 **Table.bowl1** 或 **Table.bowl2**，那麼靜態的 **Bowl** 類物件 **bowl1** 和 **bowl2** 永遠不會被建立。只有在第一個 Table 物件被建立（或被訪問）時，它們才會被初始化。此後，靜態物件不會再次被初始化。

初始化的順序先是靜態物件（如果它們之前沒有被初始化的話），然後是非靜態物件，從輸出中可以看出。要執行 `main()` 方法，必須載入 **StaticInitialization** 類，它的靜態屬性 **table** 和 **cupboard** 隨後被初始化，這會導致它們對應的類也被載入，而由於它們都包含靜態的 **Bowl** 物件，所以 **Bowl** 類也會被載入。因此，在這個特殊的程式中，所有的類都會在 `main()` 方法之前被載入。實際情況通常並非如此，因為在典型的程式中，不會像本例中所示的那樣，將所有事物透過 **static** 聯繫起來。

概括一下建立物件的過程，假設有個名為 **Dog** 的類：

1. 即使沒有顯式地使用 **static** 關鍵字，構造器實際上也是靜態方法。所以，當首次建立 **Dog** 類型的物件或是首次訪問 **Dog** 類的靜態方法或屬性時，Java 解釋器必須在類路徑中尋找，以定位 **Dog.class**。
2. 當載入完 **Dog.class** 後（後面會學到，這將建立一個 **Class** 物件），有關靜態初始化的所有動作都會執行。因此，靜態初始化只會在首次載入 **Class** 物件時初始化一次。
3. 當用 `new Dog()` 建立物件時，首先會在堆上為 **Dog** 物件分配足夠的儲存空間。
4. 分配的儲存空間首先會被清零，即會將 **Dog** 物件中的所有基本類型資料設定為預設值（數字會被置為 0，布爾型和字元型也相同），引用被置為 **null**。
5. 執行所有出現在欄位定義處的初始化動作。
6. 執行構造器。你將會在"復用"這一章看到，這可能會牽涉到很多動作，尤其當涉及繼承的時候。

### 顯式的靜態初始化

你可以將一組靜態初始化動作放在類裡面一個特殊的"靜態子句"（有時叫做靜態塊）中。像下面這樣：

```java
// housekeeping/Spoon.java

public class Spoon {
    static int i;
    
    static {
        i = 47;
    }
}
```

這看起來像個方法，但實際上它只是一段跟在 **static** 關鍵字後面的程式碼塊。與其他靜態初始化動作一樣，這段程式碼僅執行一次：當首次建立這個類的物件或首次訪問這個類的靜態成員（甚至不需要建立該類的物件）時。例如：

```java
// housekeeping/ExplicitStatic.java
// Explicit static initialization with "static" clause

class Cup {
    Cup(int marker) {
        System.out.println("Cup(" + marker + ")");
    }
    
    void f(int marker) {
        System.out.println("f(" + marker + ")");
    }
}

class Cups {
    static Cup cup1;
    static Cup cup2;
    
    static {
        cup1 = new Cup(1);
        cup2 = new Cup(2);
    }
    
    Cups() {
        System.out.println("Cups()");
    }
}

public class ExplicitStatic {
    public static void main(String[] args) {
        System.out.println("Inside main()");
        Cups.cup1.f(99); // [1]
    }
    
    // static Cups cups1 = new Cups(); // [2]
    // static Cups cups2 = new Cups(); // [2]
}
```

輸出：

```
Inside main
Cup(1)
Cup(2)
f(99)
```

無論是透過標為 [1] 的行訪問靜態的 **cup1** 物件，還是把標為 [1] 的行去掉，讓它去執行標為 [2] 的那行程式碼（去掉  [2] 的注釋），**Cups** 的靜態初始化動作都會執行。如果同時注釋 [1] 和 [2] 處，那麼 **Cups** 的靜態初始化就不會進行。此外，把標為 [2] 處的注釋都去掉還是只去掉一個，靜態初始化只會執行一次。

### 非靜態實例初始化

Java 提供了被稱為*實例初始化*的類似語法，用來初始化每個物件的非靜態變數，例如：

```java
// housekeeping/Mugs.java
// Instance initialization

class Mug {
    Mug(int marker) {
        System.out.println("Mug(" + marker + ")");
    }
}

public class Mugs {
    Mug mug1;
    Mug mug2;
    { // [1]
        mug1 = new Mug(1);
        mug2 = new Mug(2);
        System.out.println("mug1 & mug2 initialized");
    }
    
    Mugs() {
        System.out.println("Mugs()");
    }
    
    Mugs(int i) {
        System.out.println("Mugs(int)");
    }
    
    public static void main(String[] args) {
        System.out.println("Inside main()");
        new Mugs();
        System.out.println("new Mugs() completed");
        new Mugs(1);
        System.out.println("new Mugs(1) completed");
    }
}
```

輸出：

```
Inside main
Mug(1)
Mug(2)
mug1 & mug2 initialized
Mugs()
new Mugs() completed
Mug(1)
Mug(2)
mug1 & mug2 initialized
Mugs(int)
new Mugs(1) completed
```

看起來它很像靜態程式碼塊，只不過少了 **static** 關鍵字。這種語法對於支援"匿名內部類"（參見"內部類"一章）的初始化是必須的，但是你也可以使用它保證某些操作一定會發生，而不管哪個構造器被呼叫。從輸出看出，實例初始化子句是在兩個構造器之前執行的。

<!-- Array Initialization -->

## 陣列初始化

陣列是相同類型的、用一個標識符名稱封裝到一起的一個物件序列或基本類型資料序列。陣列是透過方括號下標操作符 [] 來定義和使用的。要定義一個陣列引用，只需要在類型名加上方括號：

```java
int[] a1;
```

方括號也可放在標識符的後面，兩者的含義是一樣的：

```java
int a1[];
```

這種格式符合 C 和 C++ 程式設計師的習慣。不過前一種格式或許更合理，畢竟它表明類型是"一個 **int** 型陣列"。本書中採用這種格式。

編譯器不允許指定陣列的大小。這又把我們帶回有關"引用"的問題上。你所擁有的只是對陣列的一個引用（你已經為該引用分配了足夠的儲存空間），但是還沒有給陣列物件本身分配任何空間。為了給陣列建立相應的儲存空間，必須寫初始化表達式。對於陣列，初始化動作可以出現在程式碼的任何地方，但是也可以使用一種特殊的初始化表達式，它必須在建立陣列的地方出現。這種特殊的初始化是由一對花括號括起來的值組成。這種情況下，儲存空間的分配（相當於使用 **new**） 將由編譯器負責。例如：

```java
int[] a1 = {1, 2, 3, 4, 5};
```

那麼為什麼在還沒有陣列的時候定義一個陣列引用呢？

```java
int[] a2;
```

在 Java 中可以將一個陣列賦值給另一個陣列，所以可以這樣：

```java
a2 = a1;
```

其實真正做的只是複製了一個引用，就像下面示範的這樣：

```java
// housekeeping/ArraysOfPrimitives.java

public class ArraysOfPrimitives {
    public static void main(String[] args) {
        int[] a1 = {1, 2, 3, 4, 5};
        int[] a2;
        a2 = a1;
        for (int i = 0; i < a2.length; i++) {
            a2[i] += 1;
        }
        for (int i = 0; i < a1.length; i++) {
            System.out.println("a1[" + i + "] = " + a1[i]);
        }
    }
}
```

輸出：

```
a1[0] = 2;
a1[1] = 3;
a1[2] = 4;
a1[3] = 5;
a1[4] = 6;
```

**a1** 初始化了，但是 **a2** 沒有；這裡，**a2** 在後面被賦給另一個陣列。由於 **a1** 和 **a2** 是相同陣列的別名，因此透過 **a2** 所做的修改在 **a1** 中也能看到。

所有的陣列（無論是物件陣列還是基本類型陣列）都有一個固定成員 **length**，告訴你這個陣列有多少個元素，你不能對其修改。與 C 和 C++ 類似，Java 陣列計數也是從 0 開始的，所能使用的最大下標數是 **length - 1**。超過這個邊界，C 和 C++ 會預設接受，允許你訪問所有記憶體，許多聲名狼藉的 bug 都是由此而生。但是 Java 在你訪問超出這個邊界時，會報執行時錯誤（異常），從而避免此類問題。

### 動態陣列建立

如果在編寫程式時，不確定陣列中需要多少個元素，可以使用 **new** 在陣列中建立元素。如下例所示，使用 **new** 建立基本類型陣列。**new** 不能建立非陣列以外的基本類型資料：

```java
// housekeeping/ArrayNew.java
// Creating arrays with new
import java.util.*;

public class ArrayNew {
    public static void main(String[] args) {
        int[] a;
        Random rand = new Random(47);
        a = new int[rand.nextInt(20)];
        System.out.println("length of a = " + a.length);
        System.out.println(Arrays.toString(a));
    } 
}
```

輸出：

```
length of a = 18
[0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0]
```

陣列的大小是透過 `Random.nextInt()` 隨機確定的，這個方法會返回 0 到輸入參數之間的一個值。 由於隨機性，很明顯陣列的建立確實是在執行時進行的。此外，程式輸出表明，陣列元素中的基本資料類型值會自動初始化為預設值（對於數字和字元是 0；對於布爾型是 **false**）。`Arrays.toString()` 是 **java.util** 標準類庫中的方法，會產生一維陣列的可列印版本。

本例中，陣列也可以在定義的同時進行初始化：

```java
int[] a = new int[rand.nextInt(20)];
```

如果可能的話，應該儘量這麼做。

如果你建立了一個非基本類型的陣列，那麼你建立的是一個引用陣列。以整型的包裝類型 **Integer** 為例，它是一個類而非基本類型：

```java
// housekeeping/ArrayClassObj.java
// Creating an array of nonprimitive objects

import java.util.*;

public class ArrayClassObj {
    public static void main(String[] args) {
        Random rand = new Random(47);
        Integer[] a = new Integer[rand.nextInt(20)];
        System.out.println("length of a = " + a.length);
        for (int i = 0; i < a.length; i++) {
            a[i] = rand.nextInt(500); // Autoboxing
        }
        System.out.println(Arrays.toString(a));
    }
}
```

輸出：

```
length of a = 18
[55, 193, 361, 461, 429, 368, 200, 22, 207, 288, 128, 51, 89, 309, 278, 498, 361, 20]
```

這裡，即使使用 new 建立陣列之後：

```java
Integer[] a = new Integer[rand.nextInt(20)];	
```

它只是一個引用陣列，直到透過建立新的 **Integer** 物件（透過自動裝箱），並把物件賦值給引用，初始化才算結束：

```java
a[i] = rand.nextInt(500);
```

如果忘記了建立物件，但試圖使用陣列中的空引用，就會在執行時產生異常。

也可以用花括號括起來的列表來初始化陣列，有兩種形式：

```java
// housekeeping/ArrayInit.java
// Array initialization
import java.util.*;

public class ArrayInit {
    public static void main(String[] args) {
        Integer[] a = {
                1, 2,
                3, // Autoboxing
        };
        Integer[] b = new Integer[] {
                1, 2,
                3, // Autoboxing
        };
        System.out.println(Arrays.toString(a));
        System.out.println(Arrays.toString(b));

    }
}
```

輸出：

```
[1, 2, 3]
[1, 2, 3]
```

在這兩種形式中，初始化列表的最後一個逗號是可選的（這一特性使維護長列表變得更容易）。

儘管第一種形式很有用，但是它更加受限，因為它只能用於陣列定義處。第二種形式可以用在任何地方，甚至用在方法的內部。例如，你建立了一個 **String** 陣列，將其傳遞給另一個類的 `main()` 方法，如下：

```java
// housekeeping/DynamicArray.java
// Array initialization

public class DynamicArray {
    public static void main(String[] args) {
        Other.main(new String[] {"fiddle", "de", "dum"});
    }
}

class Other {
    public static void main(String[] args) {
        for (String s: args) {
            System.out.print(s + " ");
        }
    }
}
```

輸出：

```
fiddle de dum 
```

`Other.main()` 的參數是在呼叫處建立的，因此你甚至可以在方法呼叫處提供可取代的參數。

### 可變參數列表

你可以以一種類似 C 語言中的可變參數列表（C 通常把它稱為"varargs"）來建立和呼叫方法。這可以應用在參數個數或類型未知的場合。由於所有的類都最後繼承於 **Object** 類（隨著本書的進展，你會對此有更深的認識），所以你可以建立一個以 Object 陣列為參數的方法，並像下面這樣呼叫：

```java
// housekeeping/VarArgs.java
// Using array syntax to create variable argument lists

class A {}

public class VarArgs {
    static void printArray(Object[] args) {
        for (Object obj: args) {
            System.out.print(obj + " ");
        }
        System.out.println();
    }
    
    public static void main(String[] args) {
        printArray(new Object[] {47, (float) 3.14, 11.11});
        printArray(new Object[] {"one", "two", "three"});
        printArray(new Object[] {new A(), new A(), new A()});
    }
}
```

輸出：

```
47 3.14 11.11 
one two three 
A@15db9742 A@6d06d69c A@7852e922
```

`printArray()` 的參數是 **Object** 陣列，使用 for-in 語法遍歷和列印陣列的每一項。標準 Java 庫能輸出有意義的內容，但這裡建立的是類的物件，列印出的內容是類名，後面跟著一個 **@** 符號以及多個十六進位制數字。因而，預設行為（如果沒有定義 `toString()` 方法的話，後面會講這個方法）就是列印類名和物件的地址。

你可能看到像上面這樣編寫的 Java 5 之前的程式碼，它們可以產生可變的參數列表。在 Java 5 中，這種期盼已久的特性終於添加了進來，就像在 `printArray()` 中看到的那樣：

```java
// housekeeping/NewVarArgs.java
// Using array syntax to create variable argument lists

public class NewVarArgs {
    static void printArray(Object... args) {
        for (Object obj: args) {
            System.out.print(obj + " ");
        }
        System.out.println();
    }
    
    public static void main(String[] args) {
        // Can take individual elements:
        printArray(47, (float) 3.14, 11.11);
        printArray(47, 3.14F, 11.11);
        printArray("one", "two", "three");
        printArray(new A(), new A(), new A());
        // Or an array:
        printArray((Object[]) new Integer[] {1, 2, 3, 4});
        printArray(); // Empty list is OK
    }
}
```

輸出：

```
47 3.14 11.11 
47 3.14 11.11 
one two three 
A@15db9742 A@6d06d69c A@7852e922 
1 2 3 4 
```

有了可變參數，你就再也不用顯式地編寫陣列語法了，當你指定參數時，編譯器實際上會為你填充陣列。你獲取的仍然是一個陣列，這就是為什麼 `printArray()` 可以使用 for-in 疊代陣列的原因。但是，這不僅僅只是從元素列表到陣列的自動轉換。注意程式的倒數第二行，一個 **Integer** 陣列（透過自動裝箱建立）被轉型為一個 **Object** 陣列（為了移除編譯器的警告），並且傳遞給了 `printArray()`。顯然，編譯器會發現這是一個陣列，不會執行轉換。因此，如果你有一組事物，可以把它們當作列表傳遞，而如果你已經有了一個陣列，該方法會把它們當作可變參數列表來接受。

程式的最後一行表明，可變參數的個數可以為 0。當具有可選的尾隨參數時，這一特性會有幫助：

```java
// housekeeping/OptionalTrailingArguments.java

public class OptionalTrailingArguments {
    static void f(int required, String... trailing) {
        System.out.print("required: " + required + " ");
        for (String s: trailing) {
            System.out.print(s + " ");
        }
        System.out.println();
    }
    
    public static void main(String[] args) {
        f(1, "one");
        f(2, "two", "three");
        f(0);
    }
}
```

輸出：

```
required: 1 one 
required: 2 two three 
required: 0 
```

這段程式展示了如何使用除了 **Object** 類之外類型的可變參數列表。這裡，所有的可變參數都是 **String** 物件。可變參數列表中可以使用任何類型的參數，包括基本類型。下面例子展示了可變參數列表變為陣列的情形，並且如果列表中沒有任何元素，那麼轉變為大小為 0 的陣列：

```java
// housekeeping/VarargType.java

public class VarargType {
    static void f(Character... args) {
        System.out.print(args.getClass());
        System.out.println(" length " + args.length);
    }
    
    static void g(int... args) {
        System.out.print(args.getClass());
        System.out.println(" length " + args.length)
    }
    
    public static void main(String[] args) {
        f('a');
        f();
        g(1);
        g();
        System.out.println("int[]: "+ new int[0].getClass());
    }
}
```

輸出：

```
class [Ljava.lang.Character; length 1
class [Ljava.lang.Character; length 0
class [I length 1
class [I length 0
int[]: class [I
```

`getClass()` 方法屬於 Object 類，將在"類型訊息"一章中全面介紹。它會產生物件的類，並在列印該類時，看到表示該類類型的編碼字串。前導的 **[** 代表這是一個後面緊隨的類型的陣列，**I** 表示基本類型 **int**；為了進行雙重檢查，我在最後一行建立了一個 **int** 陣列，列印了其類型。這樣也驗證了使用可變參數列表不依賴於自動裝箱，而使用的是基本類型。

然而，可變參數列表與自動裝箱可以和諧共處，如下：

```java
// housekeeping/AutoboxingVarargs.java

public class AutoboxingVarargs {
    public static void f(Integer... args) {
        for (Integer i: args) {
            System.out.print(i + " ");
        }
        System.out.println();
    }
    
    public static void main(String[] args) {
        f(1, 2);
        f(4, 5, 6, 7, 8, 9);
        f(10, 11, 12);
        
    }
}
```

輸出：

```
1 2
4 5 6 7 8 9
10 11 12
```

注意嗎，你可以在單個參數列表中將類型混合在一起，自動裝箱機制會有選擇地把 **int** 類型的參數提升為 **Integer**。

可變參數列表使得方法重載更加複雜了，儘管乍看之下似乎足夠安全：

```java
// housekeeping/OverloadingVarargs.java

public class OverloadingVarargs {
    static void f(Character... args) {
        System.out.print("first");
        for (Character c: args) {
            System.out.print(" " + c);
        }
        System.out.println();
    }
    
    static void f(Integer... args) {
        System.out.print("second");
        for (Integer i: args) {
            System.out.print(" " + i);
        }
        System.out.println();
    }
    
    static void f(Long... args) {
        System.out.println("third");
    }
    
    public static void main(String[] args) {
        f('a', 'b', 'c');
        f(1);
        f(2, 1);
        f(0);
        f(0L);
        //- f(); // Won's compile -- ambiguous
    }
}
```

輸出：

```
first a b c
second 1
second 2 1
second 0
third
```

在每種情況下，編譯器都會使用自動裝箱來匹配重載的方法，然後呼叫最明確匹配的方法。

但是如果呼叫不含參數的 `f()`，編譯器就無法知道應該呼叫哪個方法了。儘管這個錯誤可以弄清楚，但是它可能會使用戶端程式設計師感到意外。

你可能會透過在某個方法中增加一個非可變參數解決這個問題：

```java
// housekeeping/OverloadingVarargs2.java
// {WillNotCompile}

public class OverloadingVarargs2 {
    static void f(float i, Character... args) {
        System.out.println("first");
    }
    
    static void f(Character... args) {
        System.out.println("second");
    }
    
    public static void main(String[] args) {
        f(1, 'a');
        f('a', 'b');
    }
}
```

**{WillNotCompile}** 注釋把該文件排除在了本書的 Gradle 構建之外。如果你手動編譯它，會得到下面的錯誤訊息：

```
OverloadingVarargs2.java:14:error:reference to f is ambiguous f('a', 'b');
\^
both method f(float, Character...) in OverloadingVarargs2 and method f(Character...) in OverloadingVarargs2 match 1 error
```

如果你給這兩個方法都添加一個非可變參數，就可以解決問題了：

```java
// housekeeping/OverloadingVarargs3

public class OverloadingVarargs3 {
    static void f(float i, Character... args) {
        System.out.println("first");
    }
    
    static void f(char c, Character... args) {
        System.out.println("second");
    }
    
    public static void main(String[] args) {
        f(1, 'a');
        f('a', 'b');
    }
}
```

輸出：

```
first
second
```

你應該總是在重載方法的一個版本上使用可變參數列表，或者根本不用它。

<!-- Enumerated Types -->

## 列舉類型

Java 5 中添加了一個看似很小的特性 **enum** 關鍵字，它使得我們在需要群組並使用列舉類型集時，可以很方便地處理。以前，你需要建立一個整數常量集，但是這些值並不會將自身限制在這個常量集的範圍內，因此使用它們更有風險，而且更難使用。列舉類型屬於非常普遍的需求，C、C++ 和其他許多語言都已經擁有它了。在 Java 5 之前，Java 程式設計師必須了解許多細節並格外仔細地去達成 **enum** 的效果。現在 Java 也有了 **enum**，並且它的功能比 C/C++ 中的完備得多。下面是個簡單的例子：

```java
// housekeeping/Spiciness.java

public enum Spiciness {
    NOT, MILD, MEDIUM, HOT, FLAMING
}
```

這裡建立了一個名為 **Spiciness** 的列舉類型，它有5個值。由於列舉類型的實例是常量，因此按照命名慣例，它們都用大寫字母表示（如果名稱中含有多個單詞，使用下劃線分隔）。

要使用 **enum**，需要建立一個該類型的引用，然後將其賦值給某個實例：

```java
// housekeeping/SimpleEnumUse.java

public class SimpleEnumUse {
    public static void main(String[] args) {
        Spiciness howHot = Spiciness.MEDIUM;
        System.out.println(howHot);
    }
}
```

輸出：

```
MEDIUM
```

在你建立 **enum** 時，編譯器會自動添加一些有用的特性。例如，它會建立 `toString()` 方法，以便你方便地顯示某個 **enum** 實例的名稱，這從上面例子中的輸出可以看出。編譯器還會建立 `ordinal()` 方法表示某個特定 **enum** 常量的聲明順序，`static values()` 方法按照 enum 常量的聲明順序，生成這些常量值構成的陣列：

```java
// housekeeping/EnumOrder.java

public class EnumOrder {
    public static void main(String[] args) {
        for (Spiciness s: Spiciness.values()) {
            System.out.println(s + ", ordinal " + s.ordinal());
        }
    }
}
```

輸出：

```
NOT, ordinal 0
MILD, ordinal 1
MEDIUM, ordinal 2
HOT, ordinal 3
FLAMING, ordinal 4
```

儘管 **enum** 看起來像是一種新的資料類型，但是這個關鍵字只是在生成 **enum** 的類時，產生了某些編譯器行為，因此在很大程度上你可以將 **enum** 當作其他任何類。事實上，**enum** 確實是類，並且具有自己的方法。

**enum** 有一個很實用的特性，就是在 **switch** 語句中使用：

```java
// housekeeping/Burrito.java

public class Burrito {
    Spiciness degree;
    
    public Burrito(Spiciness degree) {
        this.degree = degree;
    }
    
    public void describe() {
        System.out.print("This burrito is ");
        switch(degree) {
            case NOT:
                System.out.println("not spicy at all.");
                break;
            case MILD:
            case MEDIUM:
                System.out.println("a little hot.");
                break;
            case HOT:
            case FLAMING:
            default:
                System.out.println("maybe too hot");
        }
    }
    
    public static void main(String[] args) {
        Burrito plain = new Burrito(Spiciness.NOT),
        greenChile = new Burrito(Spiciness.MEDIUM),
        jalapeno = new Burrito(Spiciness.HOT);
        plain.describe();
        greenChile.describe();
        jalapeno.describe();
    }
}
```

輸出：

```
This burrito is not spicy at all.
This burrito is a little hot.
This burrito is maybe too hot.
```

由於 **switch** 是在有限的可能值集合中選擇，因此它與 **enum** 是絕佳的組合。注意，enum 的名稱是如何能夠倍加清楚地表明程式的目的的。

通常，你可以將 **enum** 用作另一種建立資料類型的方式，然後使用所得到的類型。這正是關鍵所在，所以你不用過多地考慮它們。在 **enum** 被引入之前，你必須花費大量的精力去建立一個等同的列舉類型，並是安全可用的。

這些介紹對於你理解和使用基本的 **enum** 已經足夠了，我們會在"列舉"一章中進行更深入的探討。

<!-- Summary -->

## 本章小結

構造器，這種看起來精巧的初始化機制，應該給了你很強的暗示：初始化在程式語言中的重要地位。C++ 的發明者 Bjarne Stroustrup 在設計 C++ 期間，在針對 C 語言的生產效率進行的最初調查中發現，錯誤的初始化會導致大量編程錯誤。這些錯誤很難被發現，同樣，不合理的清理也會如此。因為構造器能保證進行正確的初始化和清理（沒有正確的構造器呼叫，編譯器就不允許建立物件），所以你就有了完全的控制和安全。

在 C++ 中，析構器很重要，因為用 **new** 建立的物件必須被明確地銷毀。在 Java 中，垃圾回收器會自動地釋放所有物件的記憶體，所以很多時候類似的清理方法就不太需要了（但是當要用到的時候，你得自己動手）。在不需要類似析構器行為的時候，Java 的垃圾回收器極大地簡化了編程，並加強了記憶體管理上的安全性。一些垃圾回收器甚至能清理其他資源，如圖形和文件句柄。然而，垃圾回收器確實增加了執行時開銷，由於 Java 解釋器從一開始就很慢，所以這種開銷到底造成多大的影響很難看出來。隨著時間的推移，Java 在性能方面提升了很多，但是速度問題仍然是它涉足某些特定編程領域的障礙。

由於要保證所有物件被建立，實際上構造器比這裡討論得更加複雜。特別是當透過*組合*或*繼承*建立新類的時候，這種保證仍然成立，並且需要一些額外的語法來支援。在後面的章節中，你會學習組合，繼承以及它們如何影響構造器。

<!-- 分頁 -->

<div style="page-break-after: always;"></div>
