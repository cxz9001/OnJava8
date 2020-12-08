[TOC]

<!-- Operators -->
# 第四章 運算符

>運算符操縱資料。

Java 是從 C++ 的基礎上做了一些改進和簡化發展而成的。對於 C/C++ 程式設計師來說，Java 的運算符並不陌生。如果你已了解 C 或 C++，大可以跳過本章和下一章，直接閱讀 Java 與 C/C++ 不同的地方。

如果理解這兩章的內容對你來說還有點困難，那麼我推薦你先了解下 《Thinking in C》 再繼續後面的學習。 這本書現在可以在 [www.OnJava8.com](http://www.OnJava8.com) 上免費下載。它的內容包含音訊講座、幻燈片、練習和解答，專門用於幫助你快速掌握學習 Java 所需的基礎知識。

<!-- Using-Java-Operators -->
## 開始使用

運算符接受一個或多個參數並生成新值。這個參數與普通方法呼叫的形式不同，但效果是相同的。加法 `+`、減法 `-`、乘法 `*`、除法 `/` 以及賦值 `=` 在任何程式語言中的工作方式都是類似的。所有運算符都能根據自己的運算物件生成一個值。除此以外，一些運算符可改變運算物件的值，這叫作“副作用”（**Side Effect**）。運算符最常見的用途就是修改自己的運算物件，從而產生副作用。但要注意生成的值亦可由沒有副作用的運算符生成。

幾乎所有運算符都只能操作基本類型（Primitives）。唯一的例外是 `=`、`==` 和 `!=`，它們能操作所有物件（這也是令人混淆的一個地方）。除此以外，**String** 類支援 `+` 和 `+=`。

<!-- Precedence -->
## 優先度

運算符的優先度決定了存在多個運算符時一個表達式各部分的運算順序。Java 對運算順序作出了特別的規定。其中，最簡單的規則就是乘法和除法在加法和減法之前完成。程式設計師經常都會忘記其他優先度規則，所以應該用括號明確規定運算順序。程式碼範例:

```java
// operators/Precedence.java
public class Precedence {
    
    public static void main(String[] args) {
        int x = 1, y = 2, z = 3;
        int a = x + y - 2/2 + z; // [1]
        int b = x + (y - 2)/(2 + z); // [2]
        System.out.println("a = " + a);
        System.out.println("b = " + b);
    }
}
```

輸出結果:

```
    a = 5
    b = 1
```

這些語句看起來大致相同，但從輸出中我們可以看出它們具有非常不同的含義，具體取決於括號的使用。

我們注意到，在 `System.out.println()` 語句中使用了 `+` 運算符。 但是在這裡 `+` 代表的意思是字串連接符。編譯器會將 `+` 連接的非字串嘗試轉換為字串。上例中的輸出結果說明了 a 和 b 都已經被轉化成了字串。

<!-- Assignment -->
## 賦值

運算符的賦值是由符號 `=` 完成的。它代表著獲取 `=` 右邊的值並賦給左邊的變數。右邊可以是任何常量、變數或者可產生一個返回值的表達式。但左邊必須是一個明確的、已命名的變數。也就是說，必須要有一個物理的空間來存放右邊的值。舉個例子來說，可將一個常數賦給一個變數（A = 4），但不可將任何東西賦給一個常數（比如不能 4 = A）。

基本類型的賦值都是直接的，而不像物件，賦予的只是其記憶體的引用。舉個例子，a = b ，如果 b 是基本類型，那麼賦值操作會將 b 的值複製一份給變數 a， 此後若 a 的值發生改變是不會影響到 b 的。作為一名程式設計師，這應該成為我們的常識。

如果是為物件賦值，那麼結果就不一樣了。對一個物件進行操作時，我們實際上操作的是它的引用。所以我們將右邊的物件賦予給左邊時，賦予的只是該物件的引用。此時，兩者指向的堆中的物件還是同一個。程式碼範例：

```java 
// operators/Assignment.java
// Assignment with objects is a bit tricky
class Tank {
    int level;
}

public class Assignment {

    public static void main(String[] args) {
        Tank t1 = new Tank();
        Tank t2 = new Tank();
        t1.level = 9;
        t2.level = 47;
        System.out.println("1: t1.level: " + t1.level +
            ", t2.level: " + t2.level);
        t1 = t2;
        System.out.println("2: t1.level: " + t1.level +
            ", t2.level: " + t2.level);
        t1.level = 27;
        System.out.println("3: t1.level: " + t1.level +
            ", t2.level: " + t2.level);
    }
}
```

輸出結果：

```
1: t1.level: 9, t2.level: 47
2: t1.level: 47, t2.level: 47
3: t1.level: 27, t2.level: 27
```

這是一個簡單的 `Tank` 類，在 `main()` 方法建立了兩個實例物件。 兩個物件的 `level` 屬性分別被賦予不同的值。 然後，t2 的值被賦予給 t1。在許多程式語言裡，預期的結果是 t1 和 t2 的值會一直相對獨立。但是，在 Java 中，由於賦予的只是物件的引用，改變 t1 也就改變了 t2。 這是因為 t1 和 t2 此時指向的是堆中同一個物件。（t1 原始物件的引用在 t2 賦值給其時遺失，它引用的物件會在垃圾回收時被清理）。

這種現象通常稱為別名（aliasing），這是 Java 處理物件的一種基本方式。但是假若你不想出現這裡的別名引起混淆的話，你可以這麼做。程式碼範例：

```java
t1.level = t2.level;
```

較之前的做法，這樣做保留了兩個單獨的物件，而不是丟棄一個並將 t1 和 t2 綁定到同一個物件。但是這樣的操作有點違背 Java 的設計原則。物件的賦值是個需要重視的環節，否則你可能收穫意外的“驚喜”。

 <!-- Aliasing During Method Calls -->
### 方法呼叫中的別名現象

當我們把物件傳遞給方法時，會發生別名現象。

```java
// operators/PassObject.java
// 正在傳遞的物件可能不是你之前使用的
class Letter {
    char c;
}

public class PassObject {
    static void f(Letter y) {
        y.c = 'z';
    }
    
    public static void main(String[] args) {
        Letter x = new Letter();
        x.c = 'a';
        System.out.println("1: x.c: " + x.c);
        f(x);
        System.out.println("2: x.c: " + x.c);
     }
}
```

輸出結果：

```
1: x.c: a
2: x.c: z
```

在許多程式語言中，方法 `f()` 似乎會在內部複製其參數 **Letter y**。但是一旦傳遞了一個引用，那麼實際上 `y.c ='z';` 是在方法 `f()` 之外改變物件。別名現象以及其解決方案是個複雜的問題，在附錄中有包含：[物件傳遞和返回](./Appendix-Passing-and-Returning-Objects.md)。意識到這一點，我們可以警惕類似的陷阱。

<!-- Mathematical Operators -->
## 算術運算符

Java 的基本算術運算符與其他大多程式語言是相同的。其中包括加號 `+`、減號 `-`、除號 `/`、乘號 `*` 以及取模 `%`（從整數除法中獲得餘數）。整數除法會直接砍掉小數，而不是進位。

Java 也用一種與 C++ 相同的簡寫形式同時進行運算和賦值操作，由運算符後跟等號表示，並且與語言中的所有運算符一致（只要有意義）。  可用 x += 4 來表示：將 x 的值加上4的結果再賦值給 x。更多程式碼範例：

```java
// operators/MathOps.java
// The mathematical operators
import java.util.*;

public class MathOps {
    public static void main(String[] args) {
        // Create a seeded random number generator:
        Random rand = new Random(47);
        int i, j, k;
        // Choose value from 1 to 100:
        j = rand.nextInt(100) + 1;
        System.out.println("j : " + j);
        k = rand.nextInt(100) + 1;
        System.out.println("k : " + k);
        i = j + k;
        System.out.println("j + k : " + i);
        i = j - k;
        System.out.println("j - k : " + i);
        i = k / j;
        System.out.println("k / j : " + i);
        i = k * j;
        System.out.println("k * j : " + i);
        i = k % j;
        System.out.println("k % j : " + i);
        j %= k;
        System.out.println("j %= k : " + j);
        // 浮點運算測試
        float u, v, w; // Applies to doubles, too
        v = rand.nextFloat();
        System.out.println("v : " + v);
        w = rand.nextFloat();
        System.out.println("w : " + w);
        u = v + w;
        System.out.println("v + w : " + u);
        u = v - w;
        System.out.println("v - w : " + u);
        u = v * w;
        System.out.println("v * w : " + u);
        u = v / w;
        System.out.println("v / w : " + u);
        // 下面的操作同樣適用於 char, 
        // byte, short, int, long, and double:
        u += v;
        System.out.println("u += v : " + u);
        u -= v;
        System.out.println("u -= v : " + u);
        u *= v;
        System.out.println("u *= v : " + u);
        u /= v;
        System.out.println("u /= v : " + u);    
    }
}

```

輸出結果：

```
j : 59
k : 56
j + k : 115
j - k : 3
k / j : 0
k * j : 3304
k % j : 56
j %= k : 3
v : 0.5309454
w : 0.0534122
v + w : 0.5843576
v - w : 0.47753322
v * w : 0.028358962
v / w : 9.940527
u += v : 10.471473
u -= v : 9.940527
u *= v : 5.2778773
u /= v : 9.940527
```

為了生成隨機數字，程式首先建立一個 **Random** 物件。不帶參數的 **Random** 物件會利用目前的時間用作隨機數生成器的“種子”（seed），從而為程式的每次執行生成不同的輸出。在本書的範例中，重要的是每個範例末尾的輸出儘可能一致，以便可以使用外部工具進行驗證。所以我們透過在建立 **Random** 物件時提供種子（隨機數生成器的初始化值，其始終為特定種子值產生相同的序列），讓程式每次執行都生成相同的隨機數，如此以來輸出結果就是可驗證的 [^1]。 若需要生成隨機值，可刪除程式碼範例中的種子參數。該物件透過呼叫方法 `nextInt()` 和 `nextFloat()`（還可以呼叫 `nextLong()` 或 `nextDouble()`），使用 **Random** 物件生成許多不同類型的隨機數。`nextInt()` 的參數設定生成的數字的上限，下限為零，為了避免零除的可能性，結果偏移1。

<!-- Unary Minus and Plus Operators -->
### 一元加減運算符

一元加 `+` 減 `-` 運算符的操作和二元是相同的。編譯器可自動識別使用何種方式解析運算：

```java
x = -a;
```

上例的程式碼表意清晰，編譯器可正確識別。下面再看一個範例：

```java
x = a * -b;
```

雖然編譯器可以正確的識別，但是程式設計師可能會迷惑。為了避免混淆，推薦下面的寫法：

```java
x = a * (-b);
```

一元減號可以得到資料的負值。一元加號的作用相反，不過它唯一能影響的就是把較小的數值類型自動轉換為 **int** 類型。

<!-- Auto-Increment-and-Decrement -->
## 遞增和遞減

和 C 語言類似，Java 提供了許多快捷運算方式。快捷運算可使程式碼可讀性，可寫性都更強。其中包括遞增 `++` 和遞減 `--`，意為“增加或減少一個單位”。舉個例子來說，假設 a 是一個 **int** 類型的值，則表達式 `++a` 就等價於 `a = a + 1`。 遞增和遞減運算符不僅可以修改變數，還可以生成變數的值。

每種類型的運算符，都有兩個版本可供選用；通常將其稱為“前綴”和“後綴”。“前遞增”表示 `++` 運算符位於變數或表達式的前面；而“後遞增”表示 `++` 運算符位於變數的後面。類似地，“前遞減”意味著 `--` 運算符位於變數的前面；而“後遞減”意味著 `--` 運算符位於變數的後面。對於前遞增和前遞減（如 `++a` 或 `--a`），會先執行遞增/減運算，再返回值。而對於後遞增和後遞減（如 `a++` 或 `a--`），會先返回值，再執行遞增/減運算。程式碼範例：

```java
// operators/AutoInc.java
// 示範 ++ 和 -- 運算符
public class AutoInc {
    public static void main(String[] args) {
        int i = 1;
        System.out.println("i: " + i);
        System.out.println("++i: " + ++i); // 前遞增
        System.out.println("i++: " + i++); // 後遞增
        System.out.println("i: " + i);
        System.out.println("--i: " + --i); // 前遞減
        System.out.println("i--: " + i--); // 後遞減
        System.out.println("i: " + i);
    }
}
```

輸出結果：

```
i: 1
++i: 2
i++: 2
i: 3
--i: 2
i--: 2
i: 1
```

對於前綴形式，我們將在執行遞增/減操作後獲取值；使用後綴形式，我們將在執行遞增/減操作之前獲取值。它們是唯一具有“副作用”的運算符（除那些涉及賦值的以外） —— 它們修改了運算元的值。

C++ 名稱來自於遞增運算符，暗示著“比 C 更進一步”。在早期的 Java 演講中，*Bill Joy*（Java 作者之一）說“**Java = C++ --**”（C++ 減減），意味著 Java 在 C++  的基礎上減少了許多不必要的東西，因此語言更簡單。隨著進一步地學習，我們會發現 Java 的確有許多地方相對 C++ 來說更簡便，但是在其他方面，難度並不會比 C++ 小多少。

<!-- Relational-Operators -->
## 關係運算符

關係運算符會透過產生一個布爾（**boolean**）結果來表示運算元之間的關係。如果關係為真，則結果為 **true**，如果關係為假，則結果為 **false**。關係運算符包括小於 `<`，大於 `>`，小於或等於 `<=`，大於或等於 `>=`，等於 `==` 和不等於 `！=`。`==` 和 `!=` 可用於所有基本類型，但其他運算符不能用於基本類型 **boolean**，因為布林值只能表示 **true** 或 **false**，所以比較它們之間的“大於”或“小於”沒有意義。

<!-- Testing Object Equivalence -->
### 測試物件等價

關係運算符 `==` 和 `!=` 同樣適用於所有物件之間的比較運算，但它們比較的內容卻經常困擾 Java 的初學者。下面是程式碼範例：

```java
// operators/Equivalence.java
public class Equivalence {
    public static void main(String[] args) {
        Integer n1 = 47;
        Integer n2 = 47;
        System.out.println(n1 == n2);
        System.out.println(n1 != n2);
    }
}
```

輸出結果：

```
true
false
```

表達式 `System.out.println(n1 == n2)` 將會輸出比較的結果。因為兩個 **Integer** 物件相同，所以先輸出 **true**，再輸出 **false**。但是，儘管對像的內容一樣，對像的引用卻不一樣。`==` 和 `!=` 比較的是物件引用，所以輸出實際上應該是先輸出 **false**，再輸出 **true**（譯者註：如果你把 47 改成 128，那麼列印的結果就是這樣，因為 Integer 內部維護著一個 IntegerCache 的快取，預設快取範圍是 [-128, 127]，所以 [-128, 127] 之間的值用 `==` 和 `!=` 比較也能能到正確的結果，但是不推薦用關係運算符比較，具體見 JDK 中的 Integer 類原始碼）。

那麼怎麼比較兩個物件的內容是否相同呢？你必須使用所有物件（不包括基本類型）中都存在的 `equals()` 方法，下面是如何使用 `equals()` 方法的範例：

```java
// operators/EqualsMethod.java
public class EqualsMethod {
    public static void main(String[] args) {
        Integer n1 = 47;
        Integer n2 = 47;
        System.out.println(n1.equals(n2));
    }
}
```

輸出結果:

```java
true
```

上例的結果看起來是我們所期望的。但其實事情並非那麼簡單。下面我們來建立自己的類：

```java
// operators/EqualsMethod2.java
// 預設的 equals() 方法沒有比較內容
class Value {
    int i;
}

public class EqualsMethod2 {
    public static void main(String[] args) {
        Value v1 = new Value();
        Value v2 = new Value();
        v1.i = v2.i = 100;
        System.out.println(v1.equals(v2));
    }
}
```

輸出結果:

```java
false
```

上例的結果再次令人困惑：結果是 **false**。原因： `equals()` 的預設行為是比較物件的引用而非具體內容。因此，除非你在新類中覆寫 `equals()` 方法，否則我們將獲取不到想要的結果。不幸的是，在學習 [復用](./08-Reuse.md)（**Reuse**） 章節後我們才能接觸到“覆寫”（**Override**），並且直到 [附錄:集合主題](./Appendix-Collection-Topics.md)，才能知道定義 `equals()` 方法的正確方式，但是現在明白 `equals()` 行為方式也可能為你節省一些時間。

大多數 Java 庫類透過覆寫 `equals()` 方法比較物件的內容而不是其引用。

<!-- Logical Operators -->
## 邏輯運算符

每個邏輯運算符 `&&` （**AND**）、`||`（**OR**）和 `!`（**非**）根據參數的邏輯關係生成布林值 `true` 或 `false`。下面的程式碼範例使用了關係運算符和邏輯運算符：

```java
// operators/Bool.java
// 關係運算符和邏輯運算符
import java.util.*;
public class Bool {
    public static void main(String[] args) {
        Random rand = new Random(47);
        int i = rand.nextInt(100);
        int j = rand.nextInt(100);
        System.out.println("i = " + i);
        System.out.println("j = " + j);
        System.out.println("i > j is " + (i > j));
        System.out.println("i < j is " + (i < j));
        System.out.println("i >= j is " + (i >= j));
        System.out.println("i <= j is " + (i <= j));
        System.out.println("i == j is " + (i == j));
        System.out.println("i != j is " + (i != j));
        // 將 int 作為布爾處理不是合法的 Java 寫法
        //- System.out.println("i && j is " + (i && j));
        //- System.out.println("i || j is " + (i || j));
        //- System.out.println("!i is " + !i);
        System.out.println("(i < 10) && (j < 10) is "
        + ((i < 10) && (j < 10)) );
        System.out.println("(i < 10) || (j < 10) is "
        + ((i < 10) || (j < 10)) );
    }
}
```

輸出結果：

```
i = 58
j = 55
i > j is true
i < j is false
i >= j is true
i <= j is false
i == j is false
i != j is true
(i < 10) && (j < 10) is false
(i < 10) || (j < 10) is false
```

在 Java 邏輯運算中，我們不能像 C/C++ 那樣使用非布林值， 而僅能使用 **AND**、 **OR**、 **NOT**。上面的例子中，我們將使用非布林值的表達式注釋掉了（你可以看到表達式前面是 //-）。但是，後續的表達式使用關係比較生成布林值，然後對結果使用了邏輯運算。請注意，如果在預期為 **String** 類型的位置使用 **boolean** 類型的值，則結果會自動轉為適當的文字格式（即 "true" 或 "false" 字串）。

我們可以將前一個程式中 **int** 的定義取代為除 **boolean** 之外的任何其他基本資料類型。但請注意，**float** 類型的數值比較非常嚴格，只要兩個數字的最小位不同則兩個數仍然不相等；只要數字最小位是大於 0 的，那麼它就不等於 0。

<!-- Short-Circuiting -->
### 短路

邏輯運算符支援一種稱為“短路”（short-circuiting）的現象。整個表達式會在運算到可以明確結果時就停止並返回結果，這意味著該邏輯表達式的後半部分不會被執行到。程式碼範例：

```java
// operators / ShortCircuit.java 
// 邏輯運算符的短路行為
public class ShortCircuit {

    static boolean test1(int val) {
        System.out.println("test1(" + val + ")");
        System.out.println("result: " + (val < 1));
        return val < 1;
    }

    static boolean test2(int val) {
        System.out.println("test2(" + val + ")");
        System.out.println("result: " + (val < 2));
        return val < 2;
    }

    static boolean test3(int val) {
        System.out.println("test3(" + val + ")");
        System.out.println("result: " + (val < 3));
        return val < 3;
    }

    public static void main(String[] args) {
        boolean b = test1(0) && test2(2) && test3(2);
        System.out.println("expression is " + b);
    }
}
```

輸出結果：

```
test1(0)
result: true
test2(2)
result: false
expression is false
```

每個測試都對參數執行比較並返回 `true` 或 `false`。同時控制台也會在方法執行時列印他們的執行狀態。 下面的表達式：

```java
test1（0）&& test2（2）&& test3（2）
```

可能你的預期是程式會執行 3 個 **test** 方法並返回。我們來分析一下：第一個方法的結果返回 `true`，因此表達式會繼續走下去。緊接著，第二個方法的返回結果是 `false`。這就代表這整個表達式的結果肯定為 `false`，所以就沒有必要再判斷剩下的表達式部分了。

所以，運用“短路”可以節省部分不必要的運算，從而提高程式潛在的效能。

<!-- Literals -->
## 字面值常量

通常，當我們向程式中插入一個字面值常量（**Literal**）時，編譯器會確切地識別它的類型。當類型不明確時，必須輔以字面值常量關聯來幫助編譯器識別。程式碼範例：

```java
// operators/Literals.java
public class Literals {
    public static void main(String[] args) {
        int i1 = 0x2f; // 16進位制 (小寫)
        System.out.println(
        "i1: " + Integer.toBinaryString(i1));
        int i2 = 0X2F; // 16進位制 (大寫)
        System.out.println(
        "i2: " + Integer.toBinaryString(i2));
        int i3 = 0177; // 8進位制 (前導0)
        System.out.println(
        "i3: " + Integer.toBinaryString(i3));
        char c = 0xffff; // 最大 char 型16進位制值
        System.out.println(
        "c: " + Integer.toBinaryString(c));
        byte b = 0x7f; // 最大 byte 型16進位制值  01111111;
        System.out.println(
        "b: " + Integer.toBinaryString(b));
        short s = 0x7fff; // 最大 short 型16進位制值
        System.out.println(
        "s: " + Integer.toBinaryString(s));
        long n1 = 200L; // long 型後綴
        long n2 = 200l; // long 型後綴 (容易與數值1混淆)
        long n3 = 200;
    
        // Java 7 二進位制字面值常量:
        byte blb = (byte)0b00110101;
        System.out.println(
        "blb: " + Integer.toBinaryString(blb));
        short bls = (short)0B0010111110101111;
        System.out.println(
        "bls: " + Integer.toBinaryString(bls));
        int bli = 0b00101111101011111010111110101111;
        System.out.println(
        "bli: " + Integer.toBinaryString(bli));
        long bll = 0b00101111101011111010111110101111;
        System.out.println(
        "bll: " + Long.toBinaryString(bll));
        float f1 = 1;
        float f2 = 1F; // float 型後綴
        float f3 = 1f; // float 型後綴
        double d1 = 1d; // double 型後綴
        double d2 = 1D; // double 型後綴
        // (long 型的字面值同樣適用於十六進位制和8進位制 )
    }
}
```

輸出結果:

```
i1: 101111
i2: 101111
i3: 1111111
c: 1111111111111111
b: 1111111
s: 111111111111111
blb: 110101
bls: 10111110101111
bli: 101111101011111010111110101111
bll: 101111101011111010111110101111
```

在文字值的後面添加字元可以讓編譯器識別該文字值的類型。對於 **Long** 型數值，結尾使用大寫 `L` 或小寫 `l` 皆可（不推薦使用 `l`，因為容易與阿拉伯數值 1 混淆）。大寫 `F` 或小寫 `f` 表示 **float** 浮點數。大寫 `D` 或小寫 `d` 表示 **double** 雙精度。

十六進位制（以 16 為基數），適用於所有整型資料類型，由前導 `0x` 或 `0X` 表示，後跟 0-9 或 a-f （大寫或小寫）。如果我們在初始化某個類型的數值時，賦值超出其範圍，那麼編譯器會報錯（不管值的數字形式如何）。在上例的程式碼中，**char**、**byte** 和 **short** 的值已經是最大了。如果超過這些值，編譯器將自動轉型為 **int**，並且提示我們需要聲明強制轉換（強制轉換將在本章後面定義），意味著我們已越過該類型的範圍界限。

八進位制（以 8 為基數）由 0~7 之間的數字和前導零 `0` 表示。

Java 7 引入了二進位制的字面值常量，由前導 `0b` 或 `0B` 表示，它可以初始化所有的整數類型。

使用整型數值類型時，顯示其二進位制形式會很有用。在 Long 型和 Integer 型中這很容易實現，呼叫其靜態的 `toBinaryString()` 方法即可。 但是請注意，若將較小的類型傳遞給 **Integer.**`toBinaryString()` 時，類型將自動轉換為 **int**。

<!-- Underscores in Literals -->
### 下劃線

Java 7 中有一個深思熟慮的補充：我們可以在數字字面量中包含下劃線 `_`，以使結果更清晰。這對於大數值的分組特別有用。程式碼範例：

```java
// operators/Underscores.java
public class Underscores {
    public static void main(String[] args) {
        double d = 341_435_936.445_667;
        System.out.println(d);
        int bin = 0b0010_1111_1010_1111_1010_1111_1010_1111;
        System.out.println(Integer.toBinaryString(bin));
        System.out.printf("%x%n", bin); // [1]
        long hex = 0x7f_e9_b7_aa;
        System.out.printf("%x%n", hex);
    }
}
```

輸出結果:

```
3.41435936445667E8
101111101011111010111110101111
2fafafaf
7fe9b7aa
```

下面是合理使用的規則：

1. 僅限單 `_`，不能多條相連。
2. 數值開頭和結尾不允許出現 `_`。
3. `F`、`D` 和 `L`的前後禁止出現 `_`。
4. 二進位制前導 `b` 和 十六進位制 `x` 前後禁止出現 `_`。

[1] 注意 `%n`的使用。熟悉 C 風格的程式設計師可能習慣於看到 `\n` 來表示換行符。問題在於它給你的是一個“Unix風格”的換行符。此外，如果我們使用的是 Windows，則必須指定 `\r\n`。這種差異的包袱應該由程式語言來解決。這就是 Java 用 `%n` 實現的可以忽略平台間差異而生成適當的換行符，但只有當你使用 `System.out.printf()` 或 `System.out.format()` 時。對於 `System.out.println()`，我們仍然必須使用 `\n`；如果你使用 `%n`，`println()` 只會輸出 `%n` 而不是換行符。

<!-- Exponential Notation -->
### 指數計數法

指數總是採用一種我認為很不直觀的記號方法:

```java
// operators/Exponents.java
// "e" 表示 10 的幾次冪
public class Exponents {
    public static void main(String[] args) {
        // 大寫 E 和小寫 e 的效果相同:
        float expFloat = 1.39e-43f;
        expFloat = 1.39E-43f;
        System.out.println(expFloat);
        double expDouble = 47e47d; // 'd' 是可選的
        double expDouble2 = 47e47; // 自動轉換為 double
        System.out.println(expDouble);
    }
}
```

輸出結果:

```
1.39E-43
4.7E48
```

在科學與工程學領域，**e** 代表自然對數的基數，約等於 2.718 （Java 裡用一種更精確的 **double** 值 **Math.E** 來表示自然對數）。指數表達式 "1.39 x e-43"，意味著 “1.39 × 2.718 的 -43 次方”。然而，自 FORTRAN 語言發明後，人們自然而然地覺得e 代表 “10 的幾次冪”。這種做法顯得頗為古怪，因為 FORTRAN 最初是為科學與工程領域設計的。

理所當然，它的設計者應對這樣的混淆概念持謹慎態度 [^2]。但不管怎樣，這種特別的表達方法在 C，C++ 以及現在的 Java 中頑固地保留下來了。所以倘若習慣 e 作為自然對數的基數使用，那麼在 Java 中看到類似“1.39e-43f”這樣的表達式時，請轉換你的思維，從程式設計的角度思考它；它真正的含義是 “1.39 × 10 的 -43 次方”。

注意如果編譯器能夠正確地識別類型，就不必使用後綴字元。對於下述語句：

```java
long n3 = 200;
```

它並不存在含糊不清的地方，所以 200 後面的 L 大可省去。然而，對於下述語句：

```java
float f4 = 1e-43f; //10 的冪數
```

編譯器通常會將指數作為 **double** 類型來處理，所以假若沒有這個後綴字元 `f`，編譯器就會報錯，提示我們應該將 **double** 型轉換成 **float** 型。

<!-- Bitwise-Operators -->
## 位運算符

位運算符允許我們操作一個整型數字中的單個二進位制位。位運算符會對兩個整數對應的位執行布爾代數，從而產生結果。

位運算源自 C 語言的底層操作。我們經常要直接操縱硬體，頻繁設定硬體暫存器內的二進位制位。Java 的設計初衷是電視機上盒嵌入式開發，所以這種底層的操作仍被保留了下來。但是，你可能不會使用太多位運算。

若兩個輸入位都是 1，則按位“與運算符” `&` 運算後結果是 1，否則結果是 0。若兩個輸入位裡至少有一個是 1，則按位“或運算符” `|` 運算後結果是 1；只有在兩個輸入位都是 0 的情況下，運算結果才是 0。若兩個輸入位的某一個是 1，另一個不是 1，那麼按位“異或運算符” `^` 運算後結果才是 1。按位“非運算符” `~` 屬於一元運算符；它只對一個自變數進行操作（其他所有運算符都是二元運算符）。按位非運算後結果與輸入位相反。例如輸入 0，則輸出 1；輸入 1，則輸出 0。

位運算符和邏輯運算符都使用了同樣的字元，只不過數量不同。位短，所以位運算符只有一個字元。位運算符可與等號 `=` 聯合使用以接收結果及賦值：`&=`，`|=` 和 `^=` 都是合法的（由於 `~` 是一元運算符，所以不可與 `=` 聯合使用）。

我們將 **Boolean** 類型被視為“單位值”（one-bit value），所以它多少有些獨特的地方。我們可以對 boolean 型變數執行與、或、異或運算，但不能執行非運算（大概是為了避免與邏輯“非”混淆）。對於布林值，位運算符具有與邏輯運算符相同的效果，只是它們不會中途“短路”。此外，針對布林值進行的位運算為我們新增了一個“異或”邏輯運算符，它並未包括在邏輯運算符的列表中。在移位表達式中，禁止使用布林值，原因將在下面解釋。

<!-- Shift Operators -->
## 移位運算符

移位運算符面向的運算物件也是二進位制的“位”。它們只能用於處理整數類型（基本類型的一種）。左移位運算符 `<<` 能將其左邊的運算物件向左移動右側指定的位數（在低位補 0）。右移位運算符 `>>` 則相反。右移位運算符有“正”、“負”值：若值為正，則在高位插入 0；若值為負，則在高位插入 1。Java 也添加了一種“不分正負”的右移位運算符（>>>），它使用了“零擴展”（zero extension）：無論正負，都在高位插入 0。這一運算符是 C/C++ 沒有的。

如果移動 **char**、**byte** 或 **short**，則會在移動發生之前將其提升為 **int**，結果為 **int**。僅使用右值（rvalue）的 5 個低階位。這可以防止我們移動超過 **int** 範圍的位數。若對一個 **long** 值進行處理，最後得到的結果也是 **long**。

移位可以與等號 `<<=` 或 `>>=` 或 `>>>=` 組合使用。左值被取代為其移位運算後的值。但是，問題來了，當無符號右移與賦值相結合時，若將其與 **byte** 或 **short** 一起使用的話，則結果錯誤。取而代之的是，它們被提升為 **int** 型並右移，但在重新賦值時被截斷。在這種情況下，結果為 -1。下面是程式碼範例：

```java
// operators/URShift.java
// 測試無符號右移
public class URShift {
    public static void main(String[] args) {
        int i = -1;
        System.out.println(Integer.toBinaryString(i));
        i >>>= 10;
        System.out.println(Integer.toBinaryString(i));
        long l = -1;
        System.out.println(Long.toBinaryString(l));
        l >>>= 10;
        System.out.println(Long.toBinaryString(l));
        short s = -1;
        System.out.println(Integer.toBinaryString(s));
        s >>>= 10;
        System.out.println(Integer.toBinaryString(s));
        byte b = -1;
        System.out.println(Integer.toBinaryString(b));
        b >>>= 10;
        System.out.println(Integer.toBinaryString(b));
        b = -1;
        System.out.println(Integer.toBinaryString(b));
        System.out.println(Integer.toBinaryString(b>>>10));
    }
}
```

輸出結果：

```
11111111111111111111111111111111
1111111111111111111111
1111111111111111111111111111111111111111111111111111111111111111
111111111111111111111111111111111111111111111111111111
11111111111111111111111111111111
11111111111111111111111111111111
11111111111111111111111111111111
11111111111111111111111111111111
11111111111111111111111111111111
1111111111111111111111
```


在上例中，結果並未重新賦值給變數 **b** ，而是直接列印出來，因此一切正常。下面是一個涉及所有位運算符的程式碼範例：

```java
// operators/BitManipulation.java
// 使用位運算符
import java.util.*;
public class BitManipulation {
    public static void main(String[] args) {
        Random rand = new Random(47);
        int i = rand.nextInt();
        int j = rand.nextInt();
        printBinaryInt("-1", -1);
        printBinaryInt("+1", +1);
        int maxpos = 2147483647;
        printBinaryInt("maxpos", maxpos);
        int maxneg = -2147483648;
        printBinaryInt("maxneg", maxneg);
        printBinaryInt("i", i);
        printBinaryInt("~i", ~i);
        printBinaryInt("-i", -i);
        printBinaryInt("j", j);
        printBinaryInt("i & j", i & j);
        printBinaryInt("i | j", i | j);
        printBinaryInt("i ^ j", i ^ j);
        printBinaryInt("i << 5", i << 5);
        printBinaryInt("i >> 5", i >> 5);
        printBinaryInt("(~i) >> 5", (~i) >> 5);
        printBinaryInt("i >>> 5", i >>> 5);
        printBinaryInt("(~i) >>> 5", (~i) >>> 5);
        long l = rand.nextLong();
        long m = rand.nextLong();
        printBinaryLong("-1L", -1L);
        printBinaryLong("+1L", +1L);
        long ll = 9223372036854775807L;
        printBinaryLong("maxpos", ll);
        long lln = -9223372036854775808L;
        printBinaryLong("maxneg", lln);
        printBinaryLong("l", l);
        printBinaryLong("~l", ~l);
        printBinaryLong("-l", -l);
        printBinaryLong("m", m);
        printBinaryLong("l & m", l & m);
        printBinaryLong("l | m", l | m);
        printBinaryLong("l ^ m", l ^ m);
        printBinaryLong("l << 5", l << 5);
        printBinaryLong("l >> 5", l >> 5);
        printBinaryLong("(~l) >> 5", (~l) >> 5);
        printBinaryLong("l >>> 5", l >>> 5);
        printBinaryLong("(~l) >>> 5", (~l) >>> 5);
    }

    static void printBinaryInt(String s, int i) {
        System.out.println(
        s + ", int: " + i + ", binary:\n " +
        Integer.toBinaryString(i));
    }

    static void printBinaryLong(String s, long l) {
        System.out.println(
        s + ", long: " + l + ", binary:\n " +
        Long.toBinaryString(l));
    }
}
```

輸出結果（前 32 行）：

```
-1, int: -1, binary:
11111111111111111111111111111111
+1, int: 1, binary:
1
maxpos, int: 2147483647, binary:
1111111111111111111111111111111
maxneg, int: -2147483648, binary:
10000000000000000000000000000000
i, int: -1172028779, binary:
10111010001001000100001010010101
~i, int: 1172028778, binary:
 1000101110110111011110101101010
-i, int: 1172028779, binary:
1000101110110111011110101101011
j, int: 1717241110, binary:
1100110010110110000010100010110
i & j, int: 570425364, binary:
100010000000000000000000010100
i | j, int: -25213033, binary:
11111110011111110100011110010111
i ^ j, int: -595638397, binary:
11011100011111110100011110000011
i << 5, int: 1149784736, binary:
1000100100010000101001010100000
i >> 5, int: -36625900, binary:
11111101110100010010001000010100
(~i) >> 5, int: 36625899, binary:
10001011101101110111101011
i >>> 5, int: 97591828, binary:
101110100010010001000010100
(~i) >>> 5, int: 36625899, binary:
10001011101101110111101011
    ...
```

結尾的兩個方法 `printBinaryInt()` 和 `printBinaryLong()` 分別操作一個 **int** 和 **long** 值，並轉換為二進位制格式輸出，同時附有簡要的文字說明。除了示範 **int** 和 **long** 的所有位運算符的效果之外，本範例還顯示 **int** 和 **long** 的最小值、最大值、+1 和 -1 值，以便我們了解它們的形式。注意高位代表符號：0 表示正，1 表示負。上面顯示了 **int** 部分的輸出。以上數字的二進位制表示形式是帶符號的補碼（2's complement）。

<!-- Ternary-if-else-Operator -->
## 三元運算符

三元運算符，也稱為條件運算符。這種運算符比較罕見，因為它有三個運算物件。但它確實屬於運算符的一種，因為它最終也會生成一個值。這與本章後一節要講述的普通 **if-else** 語句是不同的。下面是它的表達式格式：

**布爾表達式 ? 值 1 : 值 2**

若表達式計算為 **true**，則返回結果 **值 1** ；如果表達式的計算為 **false**，則返回結果 **值 2**。

當然，也可以換用普通的 **if-else** 語句（在後面介紹），但三元運算符更加簡潔。作為三元運算符的創造者， C 自詡為一門簡練的語言。三元運算符的引入多半就是為了高效編程，但假若我們打算頻繁使用它的話，還是先多作一些思量： 它易於產生可讀性差的程式碼。與 **if-else** 不同的是，三元運算符是有返回結果的。請看下面的程式碼範例：

```java
// operators/TernaryIfElse.java
public class TernaryIfElse {
    
static int ternary(int i) {
    return i < 10 ? i * 100 : i * 10;
}

static int standardIfElse(int i) {
    if(i < 10)
        return i * 100;
    else
        return i * 10;
}

    public static void main(String[] args) {
        System.out.println(ternary(9));
        System.out.println(ternary(10));
        System.out.println(standardIfElse(9));
        System.out.println(standardIfElse(10));
    }
}
```

輸出結果：

```
900
100
900
100
```

可以看出，`ternary()` 中的程式碼更簡短。然而，**standardIfElse()** 中的程式碼更易理解且不要求更多的輸入。所以我們在挑選三元運算符時，請務必權衡一下利弊。

<!-- String-Operator-+-and-+= -->
## 字串運算符

這個運算符在 Java 裡有一項特殊用途：連接字串。這點已在前面展示過了。儘管與 `+` 的傳統意義不符，但如此使用也還是比較自然的。這一功能看起來還不錯，於是在 C++ 裡引入了“運算符重載”機制，以便 C++ 程式設計師為幾乎所有運算符增加特殊的含義。但遺憾得是，與 C++ 的一些限制結合以後，它變得複雜。這要求程式設計師在設計自己的類時必須對此有周全的考慮。雖然在 Java 中實現運算符重載機制並非難事（如 C# 所展示的，它具有簡單的運算符重載），但應該特性過於複雜，因此 Java 並未實現它。

我們注意到運用 `String +` 時有一些有趣的現象。若表達式以一個 **String** 類型開頭（編譯器會自動將雙引號 `""` 標註的的字元序列轉換為字串），那麼後續所有運算物件都必須是字串。程式碼範例：

```java
// operators/StringOperators.java
public class StringOperators {
    public static void main(String[] args) {
        int x = 0, y = 1, z = 2;
        String s = "x, y, z ";
        System.out.println(s + x + y + z);
        // 將 x 轉換為字串
        System.out.println(x + " " + s);
        s += "(summed) = "; 
        // 級聯操作
        System.out.println(s + (x + y + z));
        // Integer.toString()方法的簡寫:
        System.out.println("" + x);
    }
}
```

輸出結果：

```
x, y, z 012
0 x, y, z
x, y, z (summed) = 3
0
```

**注意**：上例中第 1 輸出語句的執行結果是 `012` 而並非 `3`，這是因為編譯器將其分別轉換為其字串形式然後與字串變數 **s** 連接。在第 2 條輸出語句中，編譯器將開頭的變數轉換為了字串，由此可以看出，這種轉換與資料的位置無關，只要當中有一條資料是字串類型，其他非字串資料都將被轉換為字串形式並連接。最後一條輸出語句，我們可以看出 `+=` 運算符可以拼接其右側的字串連接結果並重賦值給自身變數 `s`。括號 `()` 可以控制表達式的計算順序，以便在顯示 **int** 之前對其進行實際求和。

請注意主方法中的最後一個例子：我們經常會看到一個空字串 `""` 跟著一個基本類型的資料。這樣可以隱式地將其轉換為字串，以代替繁瑣的顯式呼叫方法（如這裡可以使用 **Integer.toString()**）。

<!-- Common-Pitfalls-When-Using-Operators -->
## 常見陷阱

使用運算符時很容易犯的一個錯誤是，在還沒搞清楚表達式的計算方式時就試圖忽略括號 `()`。在 Java 中也一樣。 在 C++ 中你甚至可能犯這樣極端的錯誤.程式碼範例：

```java
while(x = y) {
// ...
}
```

顯然，程式設計師原意是測試等價性 `==`，而非賦值 `=`。若變數 **y** 非 0 的話，在 C/C++ 中，這樣的賦值操作總會返回 `true`。於是，上面的程式碼範例將會無限循環。而在 Java 中，這樣的表達式結果並不會轉化為一個布林值。 而編譯器會試圖把這個 **int** 型資料轉換為預期應接收的布爾類型。最後，我們將會在試圖執行前收到編譯期錯誤。因此，Java 天生避免了這種陷阱發生的可能。

唯一有種情況例外：當變數 `x` 和 `y` 都是布林值，例如  `x=y` 是一個邏輯表達式。除此之外，之前的那個例子，很大可能是錯誤。

在 C/C++ 裡，類似的一個問題還有使用按位“與” `&` 和“或” `|` 運算，而非邏輯“與” `&&` 和“或” `||`。就像 `=` 和 `==` 一樣，鍵入一個字元當然要比鍵入兩個簡單。在 Java 中，編譯器同樣可防止這一點，因為它不允許我們強行使用另一種並不符的類型。

<!-- Casting-Operators -->
## 類型轉換

“類型轉換”（Casting）的作用是“與一個模型匹配”。在適當的時候，Java 會將一種資料類型自動轉換成另一種。例如，假設我們為 **float** 變數賦值一個整數值，電腦會將 **int** 自動轉換成 **float**。我們可以在程式未自動轉換時顯式、強制地使此類型發生轉換。

要執行強制轉換，需要將所需的資料類型放在任何值左側的括號內，如下所示：

```java
// operators/Casting.java
public class Casting {
    public static void main(String[] args) {
        int i = 200;
        long lng = (long)i;
        lng = i; // 沒有必要的類型提升
        long lng2 = (long)200;
        lng2 = 200;
        // 類型收縮
        i = (int)lng2; // Cast required
    }
}
```

誠然，你可以這樣地去轉換一個數值類型的變數。但是上例這種做法是多餘的：因為編譯器會在必要時自動提升 **int** 型資料為 **long** 型。

當然，為了程式邏輯清晰或提醒自己留意，我們也可以顯式地類型轉換。在其他情況下，類型轉換只有在程式碼編譯時才顯出其重要性。在 C/C++ 中，類型轉換有時會讓人頭痛。在 Java 裡，類型轉換則是一種比較安全的操作。但是，若將資料類型進行“向下轉換”（**Narrowing Conversion**）的操作（將容量較大的資料類型轉換成容量較小的類型），可能會發生訊息遺失的危險。此時，編譯器會強迫我們進行轉型，好比在提醒我們：該操作可能危險，若你堅持讓我這麼做，那麼對不起，請明確需要轉換的類型。 對於“向上轉換”（**Widening conversion**），則不必進行顯式的類型轉換，因為較大類型的資料肯定能容納較小類型的資料，不會造成任何訊息的遺失。

除了布爾類型的資料，Java 允許任何基本類型的資料轉換為另一種基本類型的資料。此外，類是不能進行類型轉換的。為了將一個類轉換為另一個類型，需要使用特殊的方法（後面將會學習到如何在父子類之間進行向上/向下轉型，例如，“橡樹”可以轉換為“樹”，反之亦然。而對於“岩石”是無法轉換為“樹”的）。

<!-- Truncation and Rounding -->
### 截斷和舍入

在執行“向下轉換”時，必須注意資料的截斷和舍入問題。若從浮點值轉換為整型值，Java 會做什麼呢？例如：浮點數 29.7 被轉換為整型值，結果會是 29 還是 30 呢？下面是程式碼範例：

```java
// operators/CastingNumbers.java
// 嘗試轉換 float 和 double 型資料為整型資料
public class CastingNumbers {
    public static void main(String[] args) {
        double above = 0.7, below = 0.4;
        float fabove = 0.7f, fbelow = 0.4f;
        System.out.println("(int)above: " + (int)above);
        System.out.println("(int)below: " + (int)below);
        System.out.println("(int)fabove: " + (int)fabove);
        System.out.println("(int)fbelow: " + (int)fbelow);
    }
}
```

輸出結果：

```
(int)above: 0
(int)below: 0
(int)fabove: 0
(int)fbelow: 0
```

因此，答案是，從 **float** 和 **double** 轉換為整數值時，小數位將被截斷。若你想對結果進行四捨五入，可以使用 `java.lang.Math` 的 ` round()` 方法：

```java
// operators/RoundingNumbers.java
// float 和 double 類型資料的四捨五入
public class RoundingNumbers {
    public static void main(String[] args) {
        double above = 0.7, below = 0.4;
        float fabove = 0.7f, fbelow = 0.4f;
        System.out.println(
        "Math.round(above): " + Math.round(above));
        System.out.println(
        "Math.round(below): " + Math.round(below));
        System.out.println(
        "Math.round(fabove): " + Math.round(fabove));
        System.out.println(
        "Math.round(fbelow): " + Math.round(fbelow));
    }
}
```

輸出結果：
```
Math.round(above): 1
Math.round(below): 0
Math.round(fabove): 1
Math.round(fbelow): 0
```

因為 `round()` 方法是 `java.lang` 的一部分，所以我們無需透過 `import` 就可以使用。

<!-- Promotion -->
### 類型提升

你會發現，如果我們對小於 **int** 的基本資料類型（即 **char**、**byte** 或 **short**）執行任何算術或按位操作，這些值會在執行操作之前類型提升為 **int**，並且結果值的類型為 **int**。若想重新使用較小的類型，必須使用強制轉換（由於重新分配回一個較小的類型，結果可能會遺失精度）。通常，表達式中最大的資料類型是決定表達式結果的資料類型。**float** 型和 **double** 型相乘，結果是 **double** 型的；**int** 和 **long** 相加，結果是 **long** 型。

<!-- Java-Has-No-sizeof -->
## Java沒有sizeof

在 C/C++ 中，經常需要用到 `sizeof()` 方法來獲取資料項被分配的位元組大小。C/C++ 中使用 `sizeof()` 最有說服力的原因是為了移植性，不同資料在不同機器上可能有不同的大小，所以在進行大小敏感的運算時，程式設計師必須對這些類型有多大做到心中有數。例如，一台電腦可用 32 位來儲存整數，而另一台只用 16 位儲存。顯然，在第一台機器中，程式可儲存更大的值。所以，移植是令 C/C++ 程式設計師頗為頭痛的一個問題。

Java 不需要 ` sizeof()` 方法來滿足這種需求，因為所有類型的大小在不同平台上是相同的。我們不必考慮這個層次的移植問題 —— Java 本身就是一種“與平台無關”的語言。

<!-- A-Compendium-of-Operators -->
## 運算符總結

上述範例分別向我們展示了哪些基本類型能被用於特定的運算符。基本上，下面的程式碼範例是對上述所有範例的重複，只不過概括了所有的基本類型。這個文件能被正確地編譯，因為我已經把編譯不透過的那部分用注釋 `//` 過濾了。程式碼範例：

```java
// operators/AllOps.java
// 測試所有基本類型的運算符操作
// 看看哪些是能被 Java 編譯器接受的
public class AllOps {
    // 布林值的接收測試：
    void f(boolean b) {}
    void boolTest(boolean x, boolean y) {
        // 算數運算符：
        //- x = x * y;
        //- x = x / y;
        //- x = x % y;
        //- x = x + y;
        //- x = x - y;
        //- x++;
        //- x--;
        //- x = +y;
        //- x = -y;
        // 關係運算符和邏輯運算符：
        //- f(x > y);
        //- f(x >= y);
        //- f(x < y);
        //- f(x <= y);
        f(x == y);
        f(x != y);
        f(!y);
        x = x && y;
        x = x || y;
        // 按位運算符：
        //- x = ~y;
        x = x & y;
        x = x | y;
        x = x ^ y;
        //- x = x << 1;
        //- x = x >> 1;
        //- x = x >>> 1;
        // 聯合賦值：
        //- x += y;
        //- x -= y;
        //- x *= y;
        //- x /= y;
        //- x %= y;
        //- x <<= 1;
        //- x >>= 1;
        //- x >>>= 1;
        x &= y;
        x ^= y;
        x |= y;
        // 類型轉換：
        //- char c = (char)x;
        //- byte b = (byte)x;
        //- short s = (short)x;
        //- int i = (int)x;
        //- long l = (long)x;
        //- float f = (float)x;
        //- double d = (double)x;
    }

    void charTest(char x, char y) {
        // 算數運算符：
        x = (char)(x * y);
        x = (char)(x / y);
        x = (char)(x % y);
        x = (char)(x + y);
        x = (char)(x - y);
        x++;
        x--;
        x = (char) + y;
        x = (char) - y;
        // 關係和邏輯運算符：
        f(x > y);
        f(x >= y);
        f(x < y);
        f(x <= y);
        f(x == y);
        f(x != y);
        //- f(!x);
        //- f(x && y);
        //- f(x || y);
        // 按位運算符：
        x= (char)~y;
        x = (char)(x & y);
        x = (char)(x | y);
        x = (char)(x ^ y);
        x = (char)(x << 1);
        x = (char)(x >> 1);
        x = (char)(x >>> 1);
        // 聯合賦值：
        x += y;
        x -= y;
        x *= y;
        x /= y;
        x %= y;
        x <<= 1;
        x >>= 1;
        x >>>= 1;
        x &= y;
        x ^= y;
        x |= y;
        // 類型轉換
        //- boolean bl = (boolean)x;
        byte b = (byte)x;
        short s = (short)x;
        int i = (int)x;
        long l = (long)x;
        float f = (float)x;
        double d = (double)x;
    }

    void byteTest(byte x, byte y) {
        // 算數運算符：
        x = (byte)(x* y);
        x = (byte)(x / y);
        x = (byte)(x % y);
        x = (byte)(x + y);
        x = (byte)(x - y);
        x++;
        x--;
        x = (byte) + y;
        x = (byte) - y;
        // 關係和邏輯運算符：
        f(x > y);
        f(x >= y);
        f(x < y);
        f(x <= y);
        f(x == y);
        f(x != y);
        //- f(!x);
        //- f(x && y);
        //- f(x || y);
        //按位運算符：
        x = (byte)~y;
        x = (byte)(x & y);
        x = (byte)(x | y);
        x = (byte)(x ^ y);
        x = (byte)(x << 1);
        x = (byte)(x >> 1);
        x = (byte)(x >>> 1);
        // 聯合賦值：
        x += y;
        x -= y;
        x *= y;
        x /= y;
        x %= y;
        x <<= 1;
        x >>= 1;
        x >>>= 1;
        x &= y;
        x ^= y;
        x |= y;
        // 類型轉換：
        //- boolean bl = (boolean)x;
        char c = (char)x;
        short s = (short)x;
        int i = (int)x;
        long l = (long)x;
        float f = (float)x;
        double d = (double)x;
    }

    void shortTest(short x, short y) {
        // 算術運算符：
        x = (short)(x * y);
        x = (short)(x / y);
        x = (short)(x % y);
        x = (short)(x + y);
        x = (short)(x - y);
        x++;
        x--;
        x = (short) + y;
        x = (short) - y;
        // 關係和邏輯運算符：
        f(x > y);
        f(x >= y);
        f(x < y);
        f(x <= y);
        f(x == y);
        f(x != y);
        //- f(!x);
        //- f(x && y);
        //- f(x || y);
        // 按位運算符：
        x = (short) ~ y;
        x = (short)(x & y);
        x = (short)(x | y);
        x = (short)(x ^ y);
        x = (short)(x << 1);
        x = (short)(x >> 1);
        x = (short)(x >>> 1);
        // Compound assignment:
        x += y;
        x -= y;
        x *= y;
        x /= y;
        x %= y;
        x <<= 1;
        x >>= 1;
        x >>>= 1;
        x &= y;
        x ^= y;
        x |= y;
        // 類型轉換：
        //- boolean bl = (boolean)x;
        char c = (char)x;
        byte b = (byte)x;
        int i = (int)x;
        long l = (long)x;
        float f = (float)x;
        double d = (double)x;
    }

    void intTest(int x, int y) {
        // 算術運算符：
        x = x * y;
        x = x / y;
        x = x % y;
        x = x + y;
        x = x - y;
        x++;
        x--;
        x = +y;
        x = -y;
        // 關係和邏輯運算符：
        f(x > y);
        f(x >= y);
        f(x < y);
        f(x <= y);
        f(x == y);
        f(x != y);
        //- f(!x);
        //- f(x && y);
        //- f(x || y);
        // 按位運算符：
        x = ~y;
        x = x & y;
        x = x | y;
        x = x ^ y;
        x = x << 1;
        x = x >> 1;
        x = x >>> 1;
        // 聯合賦值：
        x += y;
        x -= y;
        x *= y;
        x /= y;
        x %= y;
        x <<= 1;
        x >>= 1;
        x >>>= 1;
        x &= y;
        x ^= y;
        x |= y;
        // 類型轉換：
        //- boolean bl = (boolean)x;
        char c = (char)x;
        byte b = (byte)x;
        short s = (short)x;
        long l = (long)x;
        float f = (float)x;
        double d = (double)x;
    }

    void longTest(long x, long y) {
        // 算數運算符：
        x = x * y;
        x = x / y;
        x = x % y;
        x = x + y;
        x = x - y;
        x++;
        x--;
        x = +y;
        x = -y;
        // 關係和邏輯運算符：
        f(x > y);
        f(x >= y);
        f(x < y);
        f(x <= y);
        f(x == y);
        f(x != y);
        //- f(!x);
        //- f(x && y);
        //- f(x || y);
        // 按位運算符：
        x = ~y;
        x = x & y;
        x = x | y;
        x = x ^ y;
        x = x << 1;
        x = x >> 1;
        x = x >>> 1;
        // 聯合賦值：
        x += y;
        x -= y;
        x *= y;
        x /= y;
        x %= y;
        x <<= 1;
        x >>= 1;
        x >>>= 1;
        x &= y;
        x ^= y;
        x |= y;
        // 類型轉換：
        //- boolean bl = (boolean)x;
        char c = (char)x;
        byte b = (byte)x;
        short s = (short)x;
        int i = (int)x;
        float f = (float)x;
        double d = (double)x;
    }

    void floatTest(float x, float y) {
        // 算數運算符：
        x = x * y;
        x = x / y;
        x = x % y;
        x = x + y;
        x = x - y;
        x++;
        x--;
        x = +y;
        x = -y;
        // 關係和邏輯運算符：
        f(x > y);
        f(x >= y);
        f(x < y);
        f(x <= y);
        f(x == y);
        f(x != y);
        //- f(!x);
        //- f(x && y);
        //- f(x || y);
        // 按位運算符：
        //- x = ~y;
        //- x = x & y;
        //- x = x | y;
        //- x = x ^ y;
        //- x = x << 1;
        //- x = x >> 1;
        //- x = x >>> 1;
        // 聯合賦值：
        x += y;
        x -= y;
        x *= y;
        x /= y;
        x %= y;
        //- x <<= 1;
        //- x >>= 1;
        //- x >>>= 1;
        //- x &= y;
        //- x ^= y;
        //- x |= y;
        // 類型轉換：
        //- boolean bl = (boolean)x;
        char c = (char)x;
        byte b = (byte)x;
        short s = (short)x;
        int i = (int)x;
        long l = (long)x;
        double d = (double)x;
    }

    void doubleTest(double x, double y) {
        // 算術運算符：
        x = x * y;
        x = x / y;
        x = x % y;
        x = x + y;
        x = x - y;
        x++;
        x--;
        x = +y;
        x = -y;
        // 關係和邏輯運算符：
        f(x > y);
        f(x >= y);
        f(x < y);
        f(x <= y);
        f(x == y);
        f(x != y);
        //- f(!x);
        //- f(x && y);
        //- f(x || y);
        // 按位運算符：
        //- x = ~y;
        //- x = x & y;
        //- x = x | y;
        //- x = x ^ y;
        //- x = x << 1;
        //- x = x >> 1;
        //- x = x >>> 1;
        // 聯合賦值：
        x += y;
        x -= y;
        x *= y;
        x /= y;
        x %= y;
        //- x <<= 1;
        //- x >>= 1;
        //- x >>>= 1;
        //- x &= y;
        //- x ^= y;
        //- x |= y;
        // 類型轉換：
        //- boolean bl = (boolean)x;
        char c = (char)x;
        byte b = (byte)x;
        short s = (short)x;
        int i = (int)x;
        long l = (long)x;
        float f = (float)x;
    }
}
```

**注意** ：**boolean** 類型的運算是受限的。你能為其賦值 `true` 或 `false`，也可測試它的值是否是 `true` 或 `false`。但你不能對其作加減等其他運算。

在 **char**，**byte** 和 **short** 類型中，我們可以看到算術運算符的“類型轉換”效果。我們必須要顯式強制類型轉換才能將結果重新賦值為原始類型。對於 **int** 類型的運算則不用轉換，因為預設就是 **int** 型。雖然我們不用再停下來思考這一切是否安全，但是兩個大的 int 型整數相乘時，結果有可能超出 **int** 型的範圍，這種情況下結果會發生溢位。下面的程式碼範例：

```java
// operators/Overflow.java
// 厲害了！記憶體溢位
public class Overflow {
    public static void main(String[] args) {
        int big = Integer.MAX_VALUE;
        System.out.println("big = " + big);
        int bigger = big * 4;
        System.out.println("bigger = " + bigger);
    }
}
```

輸出結果：

```text
big = 2147483647
bigger = -4
```

編譯器沒有報錯或警告，執行時一切看起來都無異常。誠然，Java 是優秀的，但是還不足夠優秀。

對於 **char**，**byte** 或者 **short**，混合賦值並不需要類型轉換。即使為它們執行轉型操作，也會獲得與直接算術運算相同的結果。另外，省略類型轉換可以使程式碼顯得更加簡練。總之，除 **boolean** 以外，其他任何兩種基本類型間都可進行類型轉換。當我們進行向下轉換類型時，需要注意結果的範圍是否溢位，否則我們就很可能在不知不覺中遺失精度。

<!-- Summary -->
## 本章小結

如果你已接觸過一門 C 語法風格程式語言，那麼你在學習 Java 的運算符時實際上沒有任何曲線。如果你覺得有難度，那麼我推薦你要先去 www.OnJava8.com 觀看 《Thinking in C》 的影片教學來補充一些前置知識儲備。

[^1]: 我在 *Pomona College* 大學讀過兩年本科，在那裡 47 被稱之為“魔法數字”（*magic number*），詳見 [維基百科](https://en.wikipedia.org/wiki/47_(number)) 。

[^2]: *John Kirkham* 說過：“自 1960 年我開始在 IBM 1620 上開始編程起，至 1970 年之間，FORTRAN 一直都是一種全大寫的程式語言。這可能是因為許多早期的輸入裝置都是舊的電傳打字機，使用了 5 位波特碼，沒有小寫字母的功能。指數符號中的 e 也總是大寫的，並且從未與自然對數底數 e 混淆，自然對數底數 e 總是小寫的。 e 簡單地代表指數，通常 10 是基數。那時，八進位制也被程式設計師廣泛使用。雖然我從未見過它的用法，但如果我看到一個指數符號的八進位制數，我會認為它是以 8 為基數的。我記得第一次看到指數使用小寫字母 e 是在 20 世紀 70 年代末，我也發現它令人困惑。這個問題出現的時候，小寫字母悄悄進入了 Fortran。如果你真的想使用自然對數底，我們實際上有一些函數要使用，但是它們都是大寫的。”

<!-- 分頁 -->
<div style="page-break-after: always;"></div>
