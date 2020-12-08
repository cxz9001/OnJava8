[TOC]

<!-- Polymorphism -->
# 第九章 多態

> 曾經有人請教我 “ Babbage 先生，如果輸入錯誤的數字到機器中，會得出正確結果嗎？” 我無法理解產生如此問題的概念上的困惑。 —— Charles Babbage (1791 - 1871)

多態是物件導向程式語言中，繼資料抽象和繼承之外的第三個重要特性。

多態提供了另一個維度的介面與實現分離，以解耦做什麼和怎麼做。多態不僅能改善程式碼的組織，提高程式碼的可讀性，而且能建立有擴展性的程式——無論在最初建立項目時還是在添加新特性時都可以“生長”的程式。

封裝透過合併特徵和行為來建立新的資料類型。隱藏實現透過將細節**私有化**把介面與實現分離。這種類型的組織機制對於有程序導向編程背景的人來說，更容易理解。而多態是消除類型之間的耦合。在上一章中，繼承允許把一個物件視為它本身的類型或它的基類類型。這樣就能把很多衍生自一個基類的類型當作同一類型處理，因而一段程式碼就可以無差別地執行在所有不同的類型上了。多態方法呼叫允許一種類型表現出與相似類型的區別，只要這些類型衍生自一個基類。這種區別是當你透過基類呼叫時，由方法的不同行為表現出來的。

在本章中，透過一些基本、簡單的例子（這些例子中只保留程式中與多態有關的行為），你將逐步學習多態（也稱為*動態綁定*或*後期綁定*或*執行時綁定*）。

<!-- Upcasting Revisited -->

## 向上轉型回顧

在上一章中，你看到了如何把一個物件視作它的自身類型或它的基類類型。這種把一個物件引用當作它的基類引用的做法稱為向上轉型，因為繼承圖中基類一般都位於最上方。

同樣你也在下面的音樂樂器例子中發現了問題。即然幾個例子都要演奏樂符（**Note**），首先我們先在包中單獨建立一個 Note 列舉類：

```java
// polymorphism/music/Note.java
// Notes to play on musical instruments
package polymorphism.music;

public enum Note {
    MIDDLE_C, C_SHARP, B_FLAT; // Etc.
}
```

列舉已經在”第 6 章初始化和清理“一章中介紹過了。

這裡，**Wind** 是一種 **Instrument**；因此，**Wind** 繼承 **Instrument**：

```java
// polymorphism/music/Instrument.java
package polymorphism.music;

class Instrument {
    public void play(Note n) {
        System.out.println("Instrument.play()");
    }
}

// polymorphism/music/Wind.java
package polymorphism.music;
// Wind objects are instruments
// because they have the same interface:
public class Wind extends Instrument {
    // Redefine interface method:
    @Override
    public void play(Note n) {
        System.out.println("Wind.play() " + n);
    }
}
```

**Music** 的方法 `tune()` 接受一個 **Instrument** 引用，同時也接受任何衍生自 **Instrument** 的類引用：

```java
// polymorphism/music/Music.java
// Inheritance & upcasting
// {java polymorphism.music.Music}
package polymorphism.music;

public class Music {
    public static void tune(Instrument i) {
        // ...
        i.play(Note.MIDDLE_C);
    }
    
    public static void main(String[] args) {
        Wind flute = new Wind();
        tune(flute); // Upcasting
    }
}
```

輸出：

```
Wind.play() MIDDLE_C
```

在 `main()` 中你看到了 `tune()` 方法傳入了一個 **Wind** 引用，而沒有做類型轉換。這樣做是允許的—— **Instrument** 的介面一定存在於 **Wind** 中，因此 **Wind** 繼承了 **Instrument**。從 **Wind** 向上轉型為 **Instrument** 可能“縮小”介面，但不會比 **Instrument** 的全部介面更少。

###  忘掉物件類型

**Music.java** 看起來似乎有點奇怪。為什麼所有人都故意忘記掉物件類型呢？當向上轉型時，就會發生這種情況，而且看起來如果 `tune()` 接受的參數是一個 **Wind** 引用會更為直觀。這會帶來一個重要問題：如果你那麼做，就要為系統內 **Instrument** 的每種類型都編寫一個新的 `tune()` 方法。假設按照這種推理，再增加 **Stringed** 和 **Brass** 這兩種 **Instrument** :

```java
// polymorphism/music/Music2.java
// Overloading instead of upcasting
// {java polymorphism.music.Music2}
package polymorphism.music;

class Stringed extends Instrument {
    @Override
    public void play(Note n) {
        System.out.println("Stringed.play() " + n);
    }
}

class Brass extends Instrument {
    @Override
    public void play(Note n) {
        System.out.println("Brass.play() " + n);
    }
}

public class Music2 {
    public static void tune(Wind i) {
        i.play(Note.MIDDLE_C);
    }
    
    public static void tune(Stringed i) {
        i.play(Note.MIDDLE_C);
    }
    
    public static void tune(Brass i) {
        i.play(Note.MIDDLE_C);
    }
    
    public static void main(String[] args) {
        Wind flute = new Wind();
        Stringed violin = new Stringed();
        Brass frenchHorn = new Brass();
        tune(flute); // No upcasting
        tune(violin);
        tune(frenchHorn);
    }
}
```

輸出：

```
Wind.play() MIDDLE_C
Stringed.play() MIDDLE_C
Brass.play() MIDDLE_C
```

這樣行得通，但是有一個主要缺點：必須為添加的每個新 **Instrument** 類編寫特定的方法。這意味著開始時就需要更多的程式，而且以後如果添加類似 `tune()` 的新方法或 **Instrument** 的新類型時，還有大量的工作要做。考慮到如果你忘記重載某個方法，編譯器也不會提示你，這會造成類型的整個處理過程變得難以管理。

如果只寫一個方法以基類作為參數，而不用管是哪個具體衍生類，這樣會變得更好嗎？也就是說，如果忘掉衍生類，編寫的程式碼只與基類打交道，會不會更好呢？

這正是多態所允許的。但是大部分擁有程序導向編程背景的程式設計師會對多態的運作方式感到一些困惑。

<!-- The Twist -->

## 轉機

執行程式後會看到 **Music.java** 的難點。**Wind.play()** 的輸出結果正是我們期望的，然而它看起來似乎不應該得出這樣的結果。觀察 `tune()` 方法：

```java
public static void tune(Instrument i) {
    // ...
    i.play(Note.MIDDLE_C);
}
```

它接受一個 **Instrument** 引用。那麼編譯器是如何知道這裡的 **Instrument** 引用指向的是 **Wind**，而不是 **Brass** 或 **Stringed** 呢？編譯器無法得知。為了深入理解這個問題，有必要研究一下*綁定*這個主題。

### 方法呼叫綁定

將一個方法呼叫和一個方法主體關聯起來稱作*綁定*。若綁定發生在程式執行前（如果有的話，由編譯器和連結器實現），叫做*前期綁定*。你可能從來沒有聽說這個術語，因為它是程序導向語言不需選擇預設的綁定方式，例如在 C 語言中就只有*前期綁定*這一種方法呼叫。

上述程式讓人困惑的地方就在於前期綁定，因為編譯器只知道一個 **Instrument** 引用，它無法得知究竟會呼叫哪個方法。

解決方法就是*後期綁定*，意味著在執行時根據物件的類型進行綁定。後期綁定也稱為*動態綁定*或*執行時綁定*。當一種語言實現了後期綁定，就必須具有某種機制在執行時能判斷物件的類型，從而呼叫恰當的方法。也就是說，編譯器仍然不知道物件的類型，但是方法呼叫機制能找到正確的方法體並呼叫。每種語言的後期綁定機制都不同，但是可以想到，物件中一定存在某種類型訊息。

Java 中除了 **static** 和 **final** 方法（**private** 方法也是隱式的 **final**）外，其他所有方法都是後期綁定。這意味著通常情況下，我們不需要判斷後期綁定是否會發生——它自動發生。

為什麼將一個物件指明為 **final** ？正如前一章所述，它可以防止方法被重寫。但更重要的一點可能是，它有效地”關閉了“動態綁定，或者說告訴編譯器不需要對其進行動態綁定。這可以讓編譯器為 **final** 方法生成更高效的程式碼。然而，大部分情況下這樣做不會對程式的整體性能帶來什麼改變，因此最好是為了設計使用 **final**，而不是為了提升性能而使用。

### 產生正確的行為

一旦當你知道 Java 中所有方法都是通過後期綁定來實現多態時，就可以編寫只與基類打交道的程式碼，而且程式碼對於衍生類來說都能正常地工作。或者換種說法，你向物件發送一條消息，讓物件自己做正確的事。

物件導向編程中的經典例子是形狀 **Shape**。這個例子很直觀，但不幸的是，它可能讓初學者困惑，認為物件導向編程只適合圖形化程式設計，實際上不是這樣。

形狀的例子中，有一個基類稱為 **Shape** ，多個不同的衍生類型分別是：**Circle**，**Square**，**Triangle** 等等。這個例子之所以好用，是因為我們可以直接說“圓(Circle)是一種形狀(Shape)”，這很容易理解。繼承圖展示了它們之間的關係：

![形狀繼承圖](../images/1561774164644.png)

向上轉型就像下面這麼簡單：

```java
Shape s = new Circle();
```

這會建立一個 **Circle** 物件，引用被賦值給 **Shape** 類型的變數 s，這看似錯誤（將一種類型賦值給另一種類型），然而是沒問題的，因此從繼承上可認為圓(Circle)就是一個形狀(Shape)。因此編譯器認可了賦值語句，沒有報錯。

假設你呼叫了一個基類方法（在各個衍生類中都被重寫）：

```java
s.draw()
```

你可能再次認為 **Shape** 的 `draw()` 方法被呼叫，因為 s 是一個 **Shape** 引用——編譯器怎麼可能知道要做其他的事呢？然而，由於後期綁定（多態）被呼叫的是 **Circle** 的 `draw()` 方法，這是正確的。

下面的例子稍微有些不同。首先讓我們建立一個可復用的 **Shape** 類庫，基類 **Shape** 為它的所有子類建立了公共介面——所有的形狀都可以被繪畫和擦除：

```java
// polymorphism/shape/Shape.java
package polymorphism.shape;

public class Shape {
    public void draw() {}
    public void erase() {}
}
```

衍生類透過重寫這些方法為每個具體的形狀提供獨一無二的方法行為：

```java
// polymorphism/shape/Circle.java
package polymorphism.shape;

public class Circle extends Shape {
    @Override
    public void draw() {
        System.out.println("Circle.draw()");
    }
    @Override
    public void erase() {
        System.out.println("Circle.erase()");
    }
}

// polymorphism/shape/Square.java
package polymorphism.shape;

public class Square extends Shape {
    @Override
    public void draw() {
        System.out.println("Square.draw()");
    }
    @Override
    public void erase() {
        System.out.println("Square.erase()");
    }
 }

// polymorphism/shape/Triangle.java
package polymorphism.shape;

public class Triangle extends Shape {
    @Override
    public void draw() {
        System.out.println("Triangle.draw()");
    }
    @Override
    public void erase() {
        System.out.println("Triangle.erase()");
    }
}
```

**RandomShapes** 是一種工廠，每當我們呼叫 `get()` 方法時，就會產生一個指向隨機建立的 **Shape** 物件的引用。注意，向上轉型發生在 **return** 語句中，每條 **return** 語句取得一個指向某個 **Circle**，**Square** 或 **Triangle** 的引用， 並將其以 **Shape** 類型從 `get()` 方法發送出去。因此無論何時呼叫 `get()` 方法，你都無法知道具體的類型是什麼，因為你總是得到一個簡單的 **Shape** 引用：

```java
// polymorphism/shape/RandomShapes.java
// A "factory" that randomly creates shapes
package polymorphism.shape;
import java.util.*;

public class RandomShapes {
    private Random rand = new Random(47);
    
    public Shape get() {
        switch(rand.nextInt(3)) {
            default:
            case 0: return new Circle();
            case 1: return new Square();
            case 2: return new Triangle();
        }
    }
    
    public Shape[] array(int sz) {
        Shape[] shapes = new Shape[sz];
        // Fill up the array with shapes:
        for (int i = 0; i < shapes.length; i++) {
            shapes[i] = get();
        }
        return shapes;
    }
}
```

`array()` 方法分配並填充了 **Shape** 陣列，這裡使用了 for-in 表達式：

```java
// polymorphism/Shapes.java
// Polymorphism in Java
import polymorphism.shape.*;

public class Shapes {
    public static void main(String[] args) {
        RandomShapes gen = new RandomShapes();
        // Make polymorphic method calls:
        for (Shape shape: gen.array(9)) {
            shape.draw();
        }
    }
}
```

輸出：

```
Triangle.draw()
Triangle.draw()
Square.draw()
Triangle.draw()
Square.draw()
Triangle.draw()
Square.draw()
Triangle.draw()
Circle.draw()
```

`main()` 方法中包含了一個 **Shape** 引用組成的陣列，其中每個元素透過呼叫 **RandomShapes** 類的 `get()` 方法生成。現在你只知道擁有一些形狀，但除此之外一無所知（編譯器也是如此）。然而當遍歷這個陣列為每個元素呼叫 `draw()` 方法時，從執行程式的結果中可以看到，與類型有關的特定行為奇蹟般地發生了。

隨機生成形狀是為了讓大家理解：在編譯時，編譯器不需要知道任何具體訊息以進行正確的呼叫。所有對方法 `draw()` 的呼叫都是透過動態綁定進行的。

### 可擴展性

現在讓我們回頭看音樂樂器的例子。由於多態機制，你可以向系統中添加任意多的新類型，而不需要修改 `tune()` 方法。在一個設計良好的物件導向程式中，許多方法將會遵循 `tune()` 的模型，只與基類介面通信。這樣的程式是可擴展的，因為可以從通用的基類衍生出新的資料類型，從而添加新的功能。那些操縱基類介面的方法不需要改動就可以應用於新類。

考慮一下樂器的例子，如果在基類中添加更多的方法，並加入一些新類，將會發生什麼事呢：

![樂器繼承圖](../images/1562252767216.png)

所有的新類都可以和原有類正常執行，不需要改動 `tune()` 方法。即使 `tune()` 方法單獨存放在某個文件中，而且向 **Instrument** 介面中添加了新的方法，`tune()` 方法也無需再編譯就能正確執行。下面是類圖的實現：

```java
// polymorphism/music3/Music3.java
// An extensible program
// {java polymorphism.music3.Music3}
package polymorphism.music3;
import polymorphism.music.Note;

class Instrument {
    void play(Note n) {
        System.out.println("Instrument.play() " + n);
    }
    
    String what() {
        return "Instrument";
    }
    
    void adjust() {
        System.out.println("Adjusting Instrument");
    }
}

class Wind extends Instrument {
    @Override
    void play(Note n) {
        System.out.println("Wind.play() " + n);
    }
    @Override
    String what() {
        return "Wind";
    }
    @Override
    void adjust() {
        System.out.println("Adjusting Wind");
    }
}

class Percussion extends Instrument {
    @Override
    void play(Note n) {
        System.out.println("Percussion.play() " + n);
    }
    @Override
    String what() {
        return "Percussion";
    }
    @Override
    void adjust() {
        System.out.println("Adjusting Percussion");
    }
}

class Stringed extends Instrument {
    @Override
    void play(Note n) {
        System.out.println("Stringed.play() " + n);
    } 
    @Override
    String what() {
        return "Stringed";
    }
    @Override
    void adjust() {
        System.out.println("Adjusting Stringed");
    }
}

class Brass extends Wind {
    @Override
    void play(Note n) {
        System.out.println("Brass.play() " + n);
    }
    @Override
    void adjust() {
        System.out.println("Adjusting Brass");
    }
}

class Woodwind extends Wind {
    @Override
    void play(Note n) {
        System.out.println("Woodwind.play() " + n);
    }
    @Override
    String what() {
        return "Woodwind";
    }
}

public class Music3 {
    // Doesn't care about type, so new types
    // added to the system still work right:
    public static void tune(Instrument i) {
        // ...
        i.play(Note.MIDDLE_C);
    }
    
    public static void tuneAll(Instrument[] e) {
        for (Instrument i: e) {
            tune(i);
        }
    }
    
    public static void main(String[] args) {
        // Upcasting during addition to the array:
        Instrument[] orchestra = {
            new Wind(),
            new Percussion(),
            new Stringed(),
            new Brass(),
            new Woodwind()
        };
        tuneAll(orchestra);
    }
}
```

輸出：

```
Wind.play() MIDDLE_C
Percussion.play() MIDDLE_C
Stringed.play() MIDDLE_C
Brass.play() MIDDLE_C
Woodwind.play() MIDDLE_C
```

新方法 `what()` 返回一個帶有類描述的 **String** 引用，`adjust()` 提供一些樂器調音的方法。

在 `main()` 方法中，當向 **orchestra** 陣列添加元素時，元素會自動向上轉型為 **Instrument**。

`tune()` 方法可以忽略周圍所有程式碼發生的變化，仍然可以正常執行。這正是我們期待多態能提供的特性。程式碼中的修改不會破壞程式中其他不應受到影響的部分。換句話說，多態是一項“將改變的事物與不變的事物分離”的重要技術。

### 陷阱：“重寫”私有方法

你可能天真地試圖像下面這樣做：

```java
// polymorphism/PrivateOverride.java
// Trying to override a private method
// {java polymorphism.PrivateOverride}
package polymorphism;

public class PrivateOverride {
    private void f() {
        System.out.println("private f()");
    }
    
    public static void main(String[] args) {
        PrivateOverride po = new Derived();
        po.f();
    }
}

class Derived extends PrivateOverride {
    public void f() {
        System.out.println("public f()");
    }
}
```

輸出：

```
private f()
```

你可能期望輸出是 **public f()**，然而 **private** 方法可以當作是 **final** 的，對於衍生類來說是隱蔽的。因此，這裡 **Derived** 的 `f()` 是一個全新的方法；因為基類版本的 `f()` 封鎖了 **Derived** ，因此它都不算是重寫方法。

結論是只有非 **private** 方法才能被重寫，但是得小心重寫 **private** 方法的現象，編譯器不報錯，但不會按我們所預期的執行。為了清晰起見，衍生類中的方法名採用與基類中 **private** 方法名不同的命名。

如果使用了 `@Override` 註解，就能檢測出問題：

```java
// polymorphism/PrivateOverride2.java
// Detecting a mistaken override using @Override
// {WillNotCompile}
package polymorphism;

public class PrivateOverride2 {
    private void f() {
        System.out.println("private f()");
    }
    
    public static void main(String[] args) {
        PrivateOverride2 po = new Derived2();
        po.f();
    }
}

class Derived2 extends PrivateOverride2 {
    @Override
    public void f() {
        System.out.println("public f()");
    }
}
```

編譯器報錯訊息是：

```
error: method does not override or
implement a method from a supertype
```

### 陷阱：屬性與靜態方法

一旦學會了多態，就可以以多態的思維方式考慮每件事。然而，只有普通的方法呼叫可以是多態的。例如，如果你直接訪問一個屬性，該訪問會在編譯時解析：

```java
// polymorphism/FieldAccess.java
// Direct field access is determined at compile time
class Super {
    public int field = 0;
    
    public int getField() {
        return field;
    }
}

class Sub extends Super {
    public int field = 1;
    
    @Override
    public int getField() {
        return field;
    }
    
    public int getSuperField() {
        return super.field;
    }
}

public class FieldAccess {
    public static void main(String[] args) {
        Super sup = new Sub(); // Upcast
        System.out.println("sup.field = " + sup.field + 
                          ", sup.getField() = " + sup.getField());
        Sub sub = new Sub();
        System.out.println("sub.field = " + sub.field + 
                          ", sub.getField() = " + sub.getField()
                          + ", sub.getSuperField() = " + sub.getSuperField())
    }
}
```

輸出：

```
sup.field = 0, sup.getField() = 1
sub.field = 1, sub.getField() = 1, sub.getSuperField() = 0
```

當 **Sub** 物件向上轉型為 **Super** 引用時，任何屬性訪問都被編譯器解析，因此不是多態的。在這個例子中，**Super.field** 和 **Sub.field** 被分配了不同的儲存空間，因此，**Sub** 實際上包含了兩個稱為 **field** 的屬性：它自己的和來自 **Super** 的。然而，在引用 **Sub** 的 **field** 時，預設的 **field** 屬性並不是 **Super** 版本的 **field** 屬性。為了獲取 **Super** 的 **field** 屬性，需要顯式地指明 **super.field**。

儘管這看起來是個令人困惑的問題，實際上基本上不會發生。首先，通常會將所有的屬性都指明為 **private**，因此不能直接訪問它們，只能透過方法來訪問。此外，你可能也不會給基類屬性和衍生類屬性起相同的名字，這樣做會令人困惑。

如果一個方法是靜態(**static**)的，它的行為就不具有多態性：

```java
// polymorphism/StaticPolymorphism.java
// static methods are not polymorphic
class StaticSuper {
    public static String staticGet() {
        return "Base staticGet()";
    }
    
    public String dynamicGet() {
        return "Base dynamicGet()";
    }
}

class StaticSub extends StaticSuper {
    public static String staticGet() {
        return "Derived staticGet()";
    }
    @Override
    public String dynamicGet() {
        return "Derived dynamicGet()";
    }
}

public class StaticPolymorphism {
    public static void main(String[] args) {
        StaticSuper sup = new StaticSub(); // Upcast
        System.out.println(StaticSuper.staticGet());
        System.out.println(sup.dynamicGet());
    }
}
```

輸出：

```
Base staticGet()
Derived dynamicGet()
```

靜態的方法只與類關聯，與單個的物件無關。

<!-- Constructors and Polymorphism -->

## 構造器和多態

通常，構造器不同於其他類型的方法。在涉及多態時也是如此。儘管構造器不具有多態性（事實上人們會把它看作是隱式聲明的靜態方法），但是理解構造器在複雜層次結構中運作多態還是非常重要的。理解這個可以幫助你避免一些不愉快的困擾。

### 構造器呼叫順序

在“初始化和清理”和“復用”兩章中已經簡單地介紹過構造器的呼叫順序，但那時還沒有介紹多態。

在衍生類的構造過程中總會呼叫基類的構造器。初始化會自動按繼承層次結構上移，因此每個基類的構造器都會被呼叫到。這麼做是有意義的，因為構造器有著特殊的任務：檢查物件是否被正確地構造。由於屬性通常聲明為 **private**，你必須假定衍生類只能訪問自己的成員而不能訪問基類的成員。只有基類的構造器擁有恰當的知識和權限來初始化自身的元素。因此，必須得呼叫所有構造器；否則就不能構造完整的物件。這就是為什麼編譯器會強制呼叫每個衍生類中的構造器的原因。如果在衍生類的構造器主體中沒有顯式地呼叫基類構造器，編譯器就會默默地呼叫無參構造器。如果沒有無參構造器，編譯器就會報錯（當類中不含構造器時，編譯器會自動合成一個無參構造器）。

下面的例子展示了組合、繼承和多態在構建順序上的作用：

```java
// polymorphism/Sandwich.java
// Order of constructor calls
// {java polymorphism.Sandwich}
package polymorphism;

class Meal {
    Meal() {
        System.out.println("Meal()");
    }
}

class Bread {
    Bread() {
        System.out.println("Bread()");
    }
}

class Cheese {
    Cheese() {
        System.out.println("Cheese()");
    }
}

class Lettuce {
    Lettuce() {
        System.out.println("Lettuce()");
    }
}

class Lunch extends Meal {
    Lunch() {
        System.out.println("Lunch()");
    }
}

class PortableLunch extends Lunch {
    PortableLunch() {
        System.out.println("PortableLunch()");
    }
}

public class Sandwich extends PortableLunch {
    private Bread b = new Bread();
    private Cheese c = new Cheese();
    private Lettuce l = new Lettuce();
    
    public Sandwich() {
        System.out.println("Sandwich()");
    }
    
    public static void main(String[] args) {
        new Sandwich();
    }
}
```

輸出：

```
Meal()
Lunch()
PortableLunch()
Bread()
Cheese()
Lettuce()
Sandwich()
```

這個例子用其他類建立了一個複雜的類。每個類都在構造器中聲明自己。重要的類是 **Sandwich**，它反映了三層繼承（如果算上 **Object** 的話，就是四層），包含了三個成員物件。

從建立 **Sandwich** 物件的輸出中可以看出物件的構造器呼叫順序如下：

1. 基類構造器被呼叫。這個步驟被遞迴地重複，這樣一來類層次的頂級父類會被最先構造，然後是它的衍生類，以此類推，直到最底層的衍生類。
2. 按聲明順序初始化成員。
3. 呼叫衍生類構造器的方法體。

構造器的呼叫順序很重要。當使用繼承時，就已經知道了基類的一切，並可以訪問基類中任意 **public** 和 **protected** 的成員。這意味著在衍生類中可以假定所有的基類成員都是有效的。在一個標準方法中，構造動作已經發生過，物件其他部分的所有成員都已經建立好。

在構造器中必須確保所有的成員都已經構建完。唯一能保證這點的方法就是首先呼叫基類的構造器。接著，在衍生類的構造器中，所有你可以訪問的基類成員都已經初始化。另一個在構造器中能知道所有成員都是有效的理由是：無論何時有可能的話，你應該在所有成員物件（透過組合將物件置於類中）定義處初始化它們（例如，例子中的 **b**、**c** 和 **l**）。如果遵循這條實踐，就可以幫助確保所有的基類成員和目前物件的成員物件都已經初始化。

不幸的是，這不能處理所有情況，在下一節會看到。

### 繼承和清理

在使用組合和繼承建立新類時，大部分時候你無需關心清理。子物件通常會留給垃圾收集器處理。如果你存在清理問題，那麼必須用心地為新類建立一個 `dispose()` 方法（這裡用的是我選擇的名稱，你可以使用更好的名稱）。由於繼承，如果有其他特殊的清理工作的話，就必須在衍生類中重寫 `dispose()` 方法。當重寫 `dispose()` 方法時，記得呼叫基類的 `dispose()` 方法，否則基類的清理工作不會發生：

```java
// polymorphism/Frog.java
// Cleanup and inheritance
// {java polymorphism.Frog}
package polymorphism;

class Characteristic {
    private String s;
    
    Characteristic(String s) {
        this.s = s;
        System.out.println("Creating Characteristic " + s);
    }
    
    protected void dispose() {
        System.out.println("disposing Characteristic " + s);
    }
}

class Description {
    private String s;
    
    Description(String s) {
        this.s = s;
        System.out.println("Creating Description " + s);
    }
    
    protected void dispose() {
        System.out.println("disposing Description " + s);
    }
}

class LivingCreature {
    private Characteristic p = new Characteristic("is alive");
    private Description t = new Description("Basic Living Creature");
    
    LivingCreature() {
        System.out.println("LivingCreature()");
    }
    
    protected void dispose() {
        System.out.println("LivingCreature dispose");
        t.dispose();
        p.dispose();
    }
}

class Animal extends LivingCreature {
    private Characteristic p = new Characteristic("has heart");
    private Description t = new Description("Animal not Vegetable");
    
    Animal() {
        System.out.println("Animal()");
    }
    
    @Override
    protected void dispose() {
        System.out.println("Animal dispose");
        t.dispose();
        p.dispose();
        super.dispose();
    }
}

class Amphibian extends Animal {
    private Characteristic p = new Characteristic("can live in water");
    private Description t = new Description("Both water and land");
    
    Amphibian() {
        System.out.println("Amphibian()");
    }
    
    @Override
    protected void dispose() {
        System.out.println("Amphibian dispose");
        t.dispose();
        p.dispose();
        super.dispose();
    }
}

public class Frog extends Amphibian {
    private Characteristic p = new Characteristic("Croaks");
    private Description t = new Description("Eats Bugs");
    
    public Frog() {
        System.out.println("Frog()");
    }
    
    @Override
    protected void dispose() {
        System.out.println("Frog dispose");
        t.dispose();
        p.dispose();
        super.dispose();
    }
    
    public static void main(String[] args) {
        Frog frog = new Frog();
        System.out.println("Bye!");
        frog.dispose();
    }
}
```

輸出：

```
Creating Characteristic is alive
Creating Description Basic Living Creature
LivingCreature()
Creating Characteristiv has heart
Creating Description Animal not Vegetable
Animal()
Creating Characteristic can live in water
Creating Description Both water and land
Amphibian()
Creating Characteristic Croaks
Creating Description Eats Bugs
Frog()
Bye!
Frog dispose
disposing Description Eats Bugs
disposing Characteristic Croaks
Amphibian dispose
disposing Description Both wanter and land
disposing Characteristic can live in water
Animal dispose
disposing Description Animal not Vegetable
disposing Characteristic has heart
LivingCreature dispose
disposing Description Basic Living Creature
disposing Characteristic is alive
```

層級結構中的每個類都有 **Characteristic** 和 **Description** 兩個類型的成員物件，它們必須得被銷毀。銷毀的順序應該與初始化的順序相反，以防一個物件依賴另一個物件。對於屬性來說，就意味著與聲明的順序相反（因為屬性是按照聲明順序初始化的）。對於基類（遵循 C++ 解構子的形式），首先進行衍生類的清理工作，然後才是基類的清理。這是因為衍生類的清理可能呼叫基類的一些方法，所以基類元件這時得存活，不能過早地被銷毀。輸出顯示了，**Frog** 物件的所有部分都是按照建立的逆序銷毀的。

儘管通常不必進行清理工作，但萬一需要時，就得謹慎小心地執行。

**Frog** 物件擁有自己的成員物件，它建立了這些成員物件，並且知道它們能存活多久，所以它知道何時呼叫 `dispose()` 方法。然而，一旦某個成員物件被其它一個或多個物件共享時，問題就變得複雜了，不能只是簡單地呼叫 `dispose()`。這裡，也許就必須使用*引用計數*來跟蹤仍然訪問著共享物件的物件數量，如下：

```java
// polymorphism/ReferenceCounting.java
// Cleaning up shared member objects
class Shared {
    private int refcount = 0;
    private static long counter = 0;
    private final long id = counter++;
    
    Shared() {
        System.out.println("Creating " + this);
    }
    
    public void addRef() {
        refcount++;
    }
    
    protected void dispose() {
        if (--refcount == 0) {
            System.out.println("Disposing " + this);
        }
    }
    
    @Override
    public String toString() {
        return "Shared " + id;
    }
}

class Composing {
    private Shared shared;
    private static long counter = 0;
    private final long id = counter++;
    
    Composing(Shared shared) {
        System.out.println("Creating " + this);
        this.shared = shared;
        this.shared.addRef();
    }
    
    protected void dispose() {
        System.out.println("disposing " + this);
        shared.dispose();
    }
    
    @Override
    public String toString() {
        return "Composing " + id;
    }
}

public class ReferenceCounting {
    public static void main(String[] args) {
        Shared shared = new Shared();
        Composing[] composing = {
            new Composing(shared),
            new Composing(shared),
            new Composing(shared),
            new Composing(shared),
            new Composing(shared),
        };
        for (Composing c: composing) {
            c.dispose();
        }
    }
}
```

輸出：

```
Creating Shared 0
Creating Composing 0
Creating Composing 1
Creating Composing 2
Creating Composing 3
Creating Composing 4
disposing Composing 0
disposing Composing 1
disposing Composing 2
disposing Composing 3
disposing Composing 4
Disposing Shared 0
```

**static long counter** 跟蹤所建立的 **Shared** 實例數量，還提供了 **id** 的值。**counter** 的類型是 **long** 而不是 **int**，以防溢位（這只是個良好實踐，對於本書的所有範例，**counter** 不會溢位）。**id** 是 **final** 的，因為它的值在初始化時確定後不應該變化。

在將一個 **shared** 物件附著在類上時，必須記住呼叫 `addRef()`，而 `dispose()` 方法會跟蹤引用數，以確定在何時真正地執行清理工作。使用這種技巧需要加倍細心，但是如果需要清理正在共享的物件，你沒有太多選擇。

### 構造器內部多態方法的行為

構造器呼叫的層次結構帶來了一個困境。如果在構造器中呼叫了正在構造的物件的動態綁定方法，會發生什麼事呢？

在普通的方法中，動態綁定的呼叫是在執行時解析的，因為物件不知道它屬於方法所在的類還是類的衍生類。

如果在構造器中呼叫了動態綁定方法，就會用到那個方法的重寫定義。然而，呼叫的結果難以預料因為被重寫的方法在物件被完全構造出來之前已經被呼叫，這使得一些 bug 很隱蔽，難以發現。

從概念上講，構造器的工作就是建立物件（這並非是平常的工作）。在構造器內部，整個物件可能只是部分形成——只知道基類物件已經初始化。如果構造器只是構造物件過程中的一個步驟，且構造的物件所屬的類是從構造器所屬的類衍生出的，那麼衍生部分在目前構造器被呼叫時還沒有初始化。然而，一個動態綁定的方法呼叫向外深入到繼承層次結構中，它可以呼叫衍生類的方法。如果你在構造器中這麼做，就可能呼叫一個方法，該方法操縱的成員可能還沒有初始化——這肯定會帶來災難。

下面例子展示了這個問題：

```java
// polymorphism/PolyConstructors.java
// Constructors and polymorphism
// don't produce what you might expect
class Glyph {
    void draw() {
        System.out.println("Glyph.draw()");
    }

    Glyph() {
        System.out.println("Glyph() before draw()");
        draw();
        System.out.println("Glyph() after draw()");
    }
}

class RoundGlyph extends Glyph {
    private int radius = 1;

    RoundGlyph(int r) {
        radius = r;
        System.out.println("RoundGlyph.RoundGlyph(), radius = " + radius);
    }

    @Override
    void draw() {
        System.out.println("RoundGlyph.draw(), radius = " + radius);
    }
}

public class PolyConstructors {
    public static void main(String[] args) {
        new RoundGlyph(5);
    }
}
```

輸出：

```
Glyph() before draw()
RoundGlyph.draw(), radius = 0
Glyph() after draw()
RoundGlyph.RoundGlyph(), radius = 5
```

**Glyph** 的 `draw()` 被設計為可重寫，在 **RoundGlyph** 這個方法被重寫。但是 **Glyph** 的構造器裡呼叫了這個方法，結果呼叫了 **RoundGlyph** 的 `draw()` 方法，這看起來正是我們的目的。輸出結果表明，當 **Glyph** 構造器呼叫了 `draw()` 時，**radius** 的值不是預設初始值 1 而是 0。這可能會導致在螢幕上只畫了一個點或乾脆什麼都不畫，於是我們只能乾瞪眼，試圖找到程式不工作的原因。

前一小節描述的初始化順序並不十分完整，而這正是解決謎團的關鍵所在。初始化的實際過程是：

1. 在所有事發生前，分配給物件的儲存空間會被初始化為二進位制 0。
2. 如前所述呼叫基類構造器。此時呼叫重寫後的 `draw()` 方法（是的，在呼叫 **RoundGraph** 構造器之前呼叫），由步驟 1 可知，**radius** 的值為 0。
3. 按聲明順序初始化成員。
4. 最終呼叫衍生類的構造器。

這麼做有個優點：所有事物至少初始化為 0（或某些特殊資料類型與 0 等價的值），而不是僅僅留作垃圾。這包括了透過組合嵌入類中的物件引用，被賦予 **null**。如果忘記初始化該引用，就會在執行時出現異常。觀察輸出結果，就會發現所有事物都是 0。

另一方面，應該震驚於輸出結果。邏輯方面我們已經做得非常完美，然而行為仍不可思議的錯了，編譯器也沒有報錯（C++ 在這種情況下會產生更加合理的行為）。像這樣的 bug 很容易被忽略，需要花很長時間才能發現。

因此，編寫構造器有一條良好規範：做儘量少的事讓物件進入良好狀態。如果有可能的話，儘量不要呼叫類中的任何方法。在基類的構造器中能安全呼叫的只有基類的 **final** 方法（這也適用於可被看作是 **final** 的 **private** 方法）。這些方法不能被重寫，因此不會產生意想不到的結果。你可能無法永遠遵循這條規範，但應該朝著它努力。

<!-- Covariant Return Types -->

## 協變返回類型

Java 5 中引入了協變返回類型，這表示衍生類的被重寫方法可以返回基類方法返回類型的衍生類型：

```java
// polymorphism/CovariantReturn.java
class Grain {
    @Override
    public String toString() {
        return "Grain";
    }
}

class Wheat extends Grain {
    @Override
    public String toString() {
        return "Wheat";
    }
}

class Mill {
    Grain process() {
        return new Grain();
    }
}

class WheatMill extends Mill {
    @Override
    Wheat process() {
        return new Wheat();
    }
}

public class CovariantReturn {
    public static void main(String[] args) {
        Mill m = new Mill();
        Grain g = m.process();
        System.out.println(g);
        m = new WheatMill();
        g = m.process();
        System.out.println(g);
    }
}
```

輸出：

```
Grain
Wheat
```

關鍵區別在於 Java 5 之前的版本強制要求被重寫的 `process()` 方法必須返回 **Grain** 而不是 **Wheat**，即使 **Wheat** 衍生自 **Grain**，因而也應該是一種合法的返回類型。協變返回類型允許返回更具體的 **Wheat** 類型。


<!-- Designing with Inheritance -->
## 使用繼承設計

學習過多態之後，一切看似都可以被繼承，因為多態是如此巧妙的工具。這會給設計帶來負擔。事實上，如果利用已有類建立新類首先選擇繼承的話，事情會變得莫名的複雜。

更好的方法是首先選擇組合，特別是不知道該使用哪種方法時。組合不會強制設計是繼承層次結構，而且組合更加靈活，因為可以動態地選擇類型（因而選擇相應的行為），而繼承要求必須在編譯時知道確切類型。下面例子說明了這點：

```java
// polymorphism/Transmogrify.java
// Dynamically changing the behavior of an object
// via composition (the "State" design pattern)
class Actor {
    public void act() {}
}

class HappyActor extends Actor {
    @Override
    public void act() {
        System.out.println("HappyActor");
    }
}

class SadActor extends Actor {
    @Override
    public void act() {
        System.out.println("SadActor");
    }
}

class Stage {
    private Actor actor = new HappyActor();
    
    public void change() {
        actor = new SadActor();
    }
    
    public void performPlay() {
        actor.act();
    }
}

public class Transmogrify {
    public static void main(String[] args) {
        Stage stage = new Stage();
        stage.performPlay();
        stage.change();
        stage.performPlay();
    }
}
```

輸出：

```
HappyActor
SadActor
```

**Stage** 物件中包含了 **Actor** 引用，該引用被初始化為指向一個 **HappyActor** 物件，這意味著 `performPlay()` 會產生一個特殊行為。但是既然引用可以在執行時與其他不同的物件綁定，那麼它就可以被取代成對 **SadActor** 的引用，`performPlay()` 的行為隨之改變。這樣你就獲得了執行時的動態靈活性（這被稱為狀態模式）。與之相反，我們無法在執行時才決定繼承不同的物件；那在編譯時就完全決定好了。

有一條通用準則：使用繼承表達行為的差異，使用屬性表達狀態的變化。在上個例子中，兩者都用到了。透過繼承得到的兩個不同類在 `act()` 方法中表達了不同的行為，**Stage** 透過組合使自己的狀態發生變化。這裡狀態的改變產生了行為的改變。

### 替代 vs 擴展

採用“純粹”的方式建立繼承層次結構看起來是最清晰的方法。即只有基類的方法才能在衍生類中被重寫，就像下圖這樣：

![類圖](../images/1562406479787.png)

這被稱作純粹的“is - a"關係，因為類的介面已經確定了它是什麼。繼承可以確保任何衍生類都擁有基類的介面，絕對不會少。如果按圖上這麼做，衍生類將只擁有基類的介面。

純粹的替代意味著衍生類可以完美地替代基類，當使用它們時，完全不需要知道這些子類的訊息。也就是說，基類可以接收任意發送給衍生類的消息，因為它們具有完全相同的介面。只需將衍生類向上轉型，不要關注物件的具體類型。所有一切都可以透過多態處理。

![](../images/1562409366638.png)

按這種方式思考，似乎只有純粹的“is - a”關係才是唯一明智的做法，其他任何設計只會導致混亂且注定失敗。這其實也是個陷阱。一旦按這種方式開始思考，就會轉而發現繼承擴展介面（遺憾的是，extends 關鍵字似乎慫恿我們這麼做）才是解決特定問題的完美方案。這可以稱為“is - like - a” 關係，因為衍生類就像是基類——它有著相同的基本介面，但還具有需要額外方法實現的其他特性：

![](../images/1562409366637.png)

雖然這是一種有用且明智的方法（依賴具體情況），但是也存在缺點。衍生類中介面的擴展部分在基類中不存在（不能透過基類訪問到這些擴展介面），因此一旦向上轉型，就不能透過基類呼叫這些新方法：

![](../images/1562409926765.png)

如果不向上轉型，就不會遇到這個問題。但是通常情況下，我們需要重新查明物件的確切類型，從而能夠訪問該類型中的擴展方法。下一節說明如何做到這點。

### 向下轉型與執行時類型訊息

由於向上轉型（在繼承層次中向上移動）會遺失具體的類型訊息，那麼為了重新獲取類型訊息，就需要在繼承層次中向下移動，使用*向下轉型*。

向上轉型永遠是安全的，因為基類不會具有比衍生類更多的介面。因此，每條發送給基類介面的消息都能被接收。但是對於向下轉型，你無法知道一個形狀是圓，它有可能是三角形、正方形或其他一些類型。

為了解決這個問題，必須得有某種方法確保向下轉型是正確的，防止意外轉型到一個錯誤類型，進而發送物件無法接收的消息。這麼做是不安全的。

在某些語言中（如 C++），必須執行一個特殊的操作來獲得安全的向下轉型，但是在 Java 中，每次轉型都會被檢查！所以即使只是進行一次普通的加括號形式的類型轉換，在執行時這個轉換仍會被檢查，以確保它的確是希望的那種類型。如果不是，就會得到 ClassCastException （類轉型異常）。這種在執行時檢查類型的行為稱作執行時類型訊息。下面例子展示了 RTTI 的行為：

```java
// polymorphism/RTTI.java
// Downcasting & Runtime type information (RTTI)
// {ThrowsException}
class Useful {
    public void f() {}
    public void g() {}
}

class MoreUseful extends Useful {
    @Override
    public void f() {}
    @Override
    public void g() {}
    public void u() {}
    public void v() {}
    public void w() {}
}

public class RTTI {
    public static void main(String[] args) {
        Useful[] x = {
            new Useful(),
            new MoreUseful()
        };
        x[0].f();
        x[1].g();
        // Compile time: method not found in Useful:
        //- x[1].u();
        ((MoreUseful) x[1]).u(); // Downcast/RTTI
        ((MoreUseful) x[0]).u(); // Exception thrown
    }
}
```

輸出：

```
Exception in thread "main"
java.lang.ClassCastException: Useful cannot be cast to
MoreUseful
at RTTI.main
```

正如前面類圖所示，**MoreUseful** 擴展了 **Useful** 的介面。而 **MoreUseful** 也繼承了 **Useful**，所以它可以向上轉型為 **Useful**。在 `main()` 方法中可以看到這種情況的發生。因為兩個物件都是 **Useful** 類型，所以對它們都可以呼叫 `f()` 和 `g()` 方法。如果試圖呼叫 `u()` 方法（只存在於 **MoreUseful** 中），就會得到編譯時錯誤訊息。

為了訪問 **MoreUseful** 物件的擴展介面，就得嘗試向下轉型。如果轉型為正確的類型，就轉型成功。否則，就會得到 ClassCastException 異常。你不必為這個異常編寫任何特殊程式碼，因為它指出了程式設計師在程式的任何地方都可能犯的錯誤。**{ThrowsException}** 注釋標籤告知本書的構建系統：在執行程式時，預期拋出一個異常。

RTTI 不僅僅包括簡單的轉型。例如，它還提供了一種方法，使你可以在試圖向下轉型前檢查所要處理的類型。“類型訊息”一章中會詳細闡述執行時類型訊息的各個方面。

<!-- Summary -->

## 本章小結

多態意味著“不同的形式”。在物件導向編程中，我們持有從基類繼承而來的相同介面和使用該介面的不同形式：不同版本的動態綁定方法。

在本章中，你可以看到，如果不使用資料抽象和繼承，就不可能理解甚至建立多態的例子。多態是一種不能單獨看待的特性（比如像 **switch** 語句那樣），它只能作為類關係全景中的一部分，與其他特性協同工作。

為了在程式中有效地使用多態乃至物件導向的技術，就必須擴展自己的程式視野，不能只看到單一類中的成員和消息，而要看到類之間的共同特性和它們之間的關係。儘管這需要很大的努力，但是這麼做是值得的。它能帶來更快的程式開發、更好的程式碼組織、擴展性更好的程式和更易維護的程式碼。

但是記住，多態可能被濫用。仔細分析程式碼以確保多態確實能帶來好處。

<!-- 分頁 -->

<div style="page-break-after: always;"></div>
