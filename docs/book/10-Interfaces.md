[TOC]

<!-- Interfaces -->
# 第十章 介面

介面和抽象類提供了一種將介面與實現分離的更加結構化的方法。

這種機制在程式語言中不常見，例如 C++ 只對這種概念有間接的支援。而在 Java 中存在這些關鍵字，說明這些思想很重要，Java 為它們提供了直接支援。

首先，我們將學習抽象類，一種介於普通類和介面之間的折中手段。儘管你的第一想法是建立介面，但是對於構建具有屬性和未實現方法的類來說，抽象類也是重要且必要的工具。你不可能總是使用純粹的介面。


<!-- Abstract Classes and Methods -->
## 抽象類和方法

在上一章的樂器例子中，基類 **Instrument** 中的方法往往是“啞”方法。如果呼叫了這些方法，就會出現一些錯誤。這是因為介面的目的是為它的衍生類建立一個通用介面。

在那些例子中，建立這個通用介面的唯一理由是，不同的子類可以用不同的方式表示此介面。通用介面建立了一個基本形式，以此表達所有衍生類的共同部分。另一種說法把 **Instrument** 稱為抽象基類，或簡稱抽象類。

對於像 **Instrument** 那樣的抽象類來說，它的物件幾乎總是沒有意義的。建立一個抽象類是為了透過通用介面操縱一系列類。因此，**Instrument** 只是表示介面，不是具體實現，所以建立一個 **Instrument** 的物件毫無意義，我們可能希望阻止使用者這麼做。透過讓 **Instrument** 所有的方法產生錯誤，就可以達到這個目的，但是這麼做會延遲到執行時才能得知錯誤訊息，並且需要使用者進行可靠、詳盡的測試。最好能在編譯時捕捉問題。

Java 提供了一個叫做*抽象方法*的機制，這個方法是不完整的：它只有聲明沒有方法體。下面是抽象方法的聲明語法：

```java
abstract void f();
```

包含抽象方法的類叫做*抽象類*。如果一個類包含一個或多個抽象方法，那麼類本身也必須限定為抽象的，否則，編譯器會報錯。

```java
// interface/Basic.java
abstract class Basic {
    abstract void unimplemented();
}
```

如果一個抽象類是不完整的，當試圖建立這個類的物件時，Java 會怎麼做呢？它不會建立抽象類的物件，所以我們只會得到編譯器的錯誤訊息。這樣保證了抽象類的純粹性，我們不用擔心誤用它。

```java
// interfaces/AttemptToUseBasic.java
// {WillNotCompile}
public class AttemptToUseBasic {
    Basic b = new Basic();
    // error: Basic is abstract; cannot be instantiated
}
```

如果建立一個繼承抽象類的新類並為之建立物件，那麼就必須為基類的所有抽象方法提供方法定義。如果不這麼做（可以選擇不做），新類仍然是一個抽象類，編譯器會強制我們為新類加上 **abstract** 關鍵字。

```java
// interfaces/Basic2.java
abstract class Basic2 extends Basic {
    int f() {
        return 111;
    }
    
    abstract void g() {
        // unimplemented() still not implemented
    }
}
```

可以將一個不包含任何抽象方法的類指明為 **abstract**，在類中的抽象方法沒什麼意義但想阻止建立類的物件時，這麼做就很有用。

```java
// interfaces/AbstractWithoutAbstracts.java
abstract class Basic3 {
    int f() {
        return 111;
    }
    
    // No abstract methods
}

public class AbstractWithoutAbstracts {
    // Basic3 b3 = new Basic3();
    // error: Basic3 is abstract; cannot be instantiated
}
```

為了建立可初始化的類，就要繼承抽象類，並提供所有抽象方法的定義：

```java
// interfaces/Instantiable.java
abstract class Uninstantiable {
    abstract void f();
    abstract int g();
}

public class Instantiable extends Uninstantiable {
    @Override
    void f() {
        System.out.println("f()");
    }
    
    @Override
    int g() {
        return 22;
    }
    
    public static void main(String[] args) {
        Uninstantiable ui = new Instantiable();
    }
}
```

留意 `@Override` 的使用。沒有這個註解的話，如果你沒有定義相同的方法名或簽名，抽象機制會認為你沒有實現抽象方法從而產生編譯時錯誤。因此，你可能認為這裡的 `@Override` 是多餘的。但是，`@Override` 還提示了這個方法被覆寫——我認為這是有用的，所以我會使用 `@Override`，不僅僅是因為當沒有這個註解時，編譯器會告訴我出錯。 

記住，事實上的訪問權限是“friendly”。你很快會看到介面自動將其方法指明為 **public**。事實上，介面只允許 **public** 方法，如果不加訪問修飾符的話，介面的方法不是 **friendly** 而是 **public**。然而，抽象類允許每件事：

```java
// interfaces/AbstractAccess.java
abstract class AbstractAccess {
    private void m1() {}
    
    // private abstract void m1a(); // illegal
    
    protected void m2() {}
    
    protected abstract void m2a();
    
    void m3() {}
    
    abstract void m3a();
    
    public void m4() {}
    
    public abstract void m4a();
}
```

**private abstract** 被禁止了是有意義的，因為你不可能在 **AbstractAccess** 的任何子類中合法地定義它。

上一章的 **Instrument** 類可以很輕易地轉換為一個抽象類。只需要部分方法是 **abstract** 即可。將一個類指明為 **abstract** 並不強制類中的所有方法必須都是抽象方法。如下圖所示：

![類圖](../images/1562653648586.png)

下面是修改成使用抽象類和抽象方法的管弦樂器的例子：

```java
// interfaces/music4/Music4.java
// Abstract classes and methods
// {java interfaces.music4.Music4}
package interfaces.music4;
import polymorphism.music.Note;

abstract class Instrument {
    private int i; // Storage allocated for each
    
    public abstract void play(Note n);
    
    public String what() {
        return "Instrument";
    }
    
    public abstract void adjust();
}

class Wind extends Instrument {
    @Override
    public void play(Note n) {
        System.out.println("Wind.play() " + n);
    }
    
    @Override
    public String what() {
        return "Wind";
    }
    
    @Override
    public void adjust() {
        System.out.println("Adjusting Wind");
    }
}

class Percussion extends Instrument {
    @Override
    public void play(Note n) {
        System.out.println("Percussion.play() " + n);
    }
    
    @Override
    public String what() {
        return "Percussion";
    }
    
    @Override
    public void adjust() {
        System.out.println("Adjusting Percussion");
    }
}

class Stringed extends Instrument {
    @Override
    public void play(Note n) {
        System.out.println("Stringed.play() " + n);
    }
    
    @Override
    public String what() {
        return "Stringed";
    }
    
    @Override
    public void adjust() {
        System.out.println("Adjusting Stringed");
    }
}

class Brass extends Wind {
    @Override
    public void play(Note n) {
        System.out.println("Brass.play() " + n);
    }
    
    @Override
    public void adjust() {
        System.out.println("Adjusting Brass");
    }
}

class Woodwind extends Wind {
    @Override
    public void play(Note n) {
        System.out.println("Woodwind.play() " + n);
    }
    
    @Override
    public String what() {
        return "Woodwind";
    }
}

public class Music4 {
    // Doesn't care about type, so new types
    // added to system still work right:
    static void tune(Instrument i) {
        // ...
        i.play(Note.MIDDLE_C);
    }
    
    static void tuneAll(Instrument[] e) {
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

除了 **Instrument**，基本沒區別。

建立抽象類和抽象方法是有幫助的，因為它們使得類的抽象性很明確，並能告知使用者和編譯器使用意圖。抽象類同時也是一種有用的重構工具，使用它們使得我們很容易地將沿著繼承層級結構上移公共方法。

<!-- Interfaces -->

## 介面建立

使用 **interface** 關鍵字建立介面。在本書中，interface 和 class 一樣隨處常見，除非特指關鍵字 **interface**，其他情況下都採用正常字體書寫 interface。

描述 Java 8 之前的介面更加容易，因為它們只允許抽象方法。像下面這樣：

```java
// interfaces/PureInterface.java
// Interface only looked like this before Java 8
public interface PureInterface {
    int m1(); 
    void m2();
    double m3();
}
```

我們甚至不用為方法加上 **abstract** 關鍵字，因為方法在介面中。Java 知道這些方法不能有方法體（仍然可以為方法加上 **abstract** 關鍵字，但是看起來像是不明白介面，徒增難堪罷了）。

因此，在 Java 8之前我們可以這麼說：**interface** 關鍵字產生一個完全抽象的類，沒有提供任何實現。我們只能描述類應該像什麼，做什麼，但不能描述怎麼做，即只能決定方法名、參數列表和返回類型，但是無法確定方法體。介面只提供形式，通常來說沒有實現，儘管在某些受限制的情況下可以有實現。

一個介面表示：所有實現了該介面的類看起來都像這樣。因此，任何使用某特定介面的程式碼都知道可以呼叫該介面的哪些方法，而且僅需知道這些。所以，介面被用來建立類之間的協議。（一些物件導向程式語言中，使用 protocol 關鍵字完成相同的功能。）

Java 8 中介面稍微有些變化，因為 Java 8 允許介面包含預設方法和靜態方法——基於某些重要原因，看到後面你會理解。介面的基本概念仍然沒變，介於類型之上、實現之下。介面與抽象類最明顯的區別可能就是使用上的慣用方式。介面的典型使用是代表一個類的類型或一個形容詞，如 Runnable 或 Serializable，而抽象類通常是類層次結構的一部分或一件事物的類型，如 String 或 ActionHero。

使用關鍵字 **interface** 而不是 **class** 來建立介面。和類一樣，需要在關鍵字 **interface** 前加上 **public** 關鍵字（但只是在介面名與檔案名相同的情況下），否則介面只有包訪問權限，只能在介面相同的包下才能使用它。

介面同樣可以包含屬性，這些屬性被隱式指明為 **static** 和 **final**。

使用 **implements** 關鍵字使一個類遵循某個特定介面（或一組介面），它表示：介面只是外形，現在我要說明它是如何工作的。除此之外，它看起來像繼承。

```java
// interfaces/ImplementingAnInterface.java
interface Concept { // Package access
    void idea1();
    void idea2();
}

class Implementation implements Concept {
    @Override
    public void idea1() {
        System.out.println("idea1");
    }
    
    @Override
    public void idea2() {
        System.out.println("idea2");
    }
}
```

你可以選擇顯式地聲明介面中的方法為 **public**，但是即使你不這麼做，它們也是 **public** 的。所以當實現一個介面時，來自介面中的方法必須被定義為 **public**。否則，它們只有包訪問權限，這樣在繼承時，它們的可訪問權限就被降低了，這是 Java 編譯器所不允許的。

### 預設方法

Java 8 為關鍵字 **default** 增加了一個新的用途（之前只用於 **switch** 語句和註解中）。當在介面中使用它時，任何實現介面卻沒有定義方法的時候可以使用 **default** 建立的方法體。預設方法比抽象類中的方法受到更多的限制，但是非常有用，我們將在“流式編程”一章中看到。現在讓我們看一下如何使用：

```java
// interfaces/AnInterface.java
interface AnInterface {
    void firstMethod();
    void secondMethod();
}
```

我們可以像這樣實現介面：

```java
// interfaces/AnImplementation.java
public class AnImplementation implements AnInterface {
    public void firstMethod() {
        System.out.println("firstMethod");
    }
    
    public void secondMethod() {
        System.out.println("secondMethod");
    }
    
    public static void main(String[] args) {
        AnInterface i = new AnImplementation();
        i.firstMethod();
        i.secondMethod();
    }
}
```

輸出：

```
firstMethod
secondMethod
```

如果我們在 **AnInterface** 中增加一個新方法 `newMethod()`，而在 **AnImplementation** 中沒有實現它，編譯器就會報錯：

```
AnImplementation.java:3:error: AnImplementation is not abstract and does not override abstract method newMethod() in AnInterface
public class AnImplementation implements AnInterface {
^
1 error
```

如果我們使用關鍵字 **default** 為 `newMethod()` 方法提供預設的實現，那麼所有與介面有關的程式碼能正常工作，不受影響，而且這些程式碼還可以呼叫新的方法 `newMethod()`：

```java
// interfaces/InterfaceWithDefault.java
interface InterfaceWithDefault {
    void firstMethod();
    void secondMethod();
    
    default void newMethod() {
        System.out.println("newMethod");
    }
}
```

關鍵字 **default** 允許在介面中提供方法實現——在 Java 8 之前被禁止。

```java
// interfaces/Implementation2.java
public class Implementation2 implements InterfaceWithDefault {
    @Override
    public void firstMethod() {
        System.out.println("firstMethod");
    }
    
    @Override
    public void secondMethod() {
        System.out.println("secondMethod")
    }
    
    public static void main(String[] args) {
        InterfaceWithDefault i = new Implementation2();
        i.firstMethod();
        i.secondMethod();
        i.newMethod();
    }
}
```

輸出：

```
firstMethod
secondMethod
newMethod
```

儘管 **Implementation2** 中未定義 `newMethod()`，但是可以使用 `newMethod()` 了。 

增加預設方法的極具說服力的理由是它允許在不破壞已使用介面的程式碼的情況下，在介面中增加新的方法。預設方法有時也被稱為*守衛方法*或*虛擬擴展方法*。

### 多繼承

多繼承意味著一個類可能從多個父類型中繼承特徵和特性。

Java 在設計之初，C++ 的多繼承機制飽受詬病。Java 過去是一種嚴格要求單繼承的語言：只能繼承自一個類（或抽象類），但可以實現任意多個介面。在 Java 8 之前，介面沒有包袱——它只是方法外貌的描述。

多年後的現在，Java 透過預設方法具有了某種多繼承的特性。結合帶有預設方法的介面意味著結合了多個基類中的行為。因為介面中仍然不允許存在屬性（只有靜態屬性，不適用），所以屬性仍然只會來自單個基類或抽象類，也就是說，不會存在狀態的多繼承。正如下面這樣：

```java
// interfaces/MultipleInheritance.java
import java.util.*;

interface One {
    default void first() {
        System.out.println("first");
    }
}

interface Two {
    default void second() {
        System.out.println("second");
    }
}

interface Three {
    default void third() {
        System.out.println("third");
    }
}

class MI implements One, Two, Three {}

public class MultipleInheritance {
    public static void main(String[] args) {
        MI mi = new MI();
        mi.first();
        mi.second();
        mi.third();
    }
}
```

輸出：

```
first
second
third
```

現在我們做些在 Java 8 之前不可能完成的事：結合多個源的實現。只要基類方法中的方法名和參數列表不同，就能工作得很好，否則會得到編譯器錯誤：

```java
// interface/MICollision.java
import java.util.*;

interface Bob1 {
    default void bob() {
        System.out.println("Bob1::bob");
    }
}

interface Bob2 {
    default void bob() {
        System.out.println("Bob2::bob");
    }
}

// class Bob implements Bob1, Bob2 {}
/* Produces:
error: class Bob inherits unrelated defaults
for bob() from types Bob1 and Bob2
class Bob implements Bob1, Bob2 {}
^
1 error
*/

interface Sam1 {
    default void sam() {
        System.out.println("Sam1::sam");
    }
}

interface Sam2 {
    default void sam(int i) {
        System.out.println(i * 2);
    }
}

// This works because the argument lists are distinct:
class Sam implements Sam1, Sam2 {}

interface Max1 {
    default void max() {
        System.out.println("Max1::max");
    }
}

interface Max2 {
    default int max() {
        return 47;
    }
}

// class Max implements Max1, Max2 {}
/* Produces:
error: types Max2 and Max1 are imcompatible;
both define max(), but with unrelated return types
class Max implements Max1, Max2 {}
^
1 error
*/
```

**Sam** 類中的兩個 `sam()` 方法有相同的方法名但是簽名不同——方法簽名包括方法名和參數類型，編譯器也是用它來區分方法。但是從 **Max** 類可看出，返回類型不是方法簽名的一部分，因此不能用來區分方法。為了解決這個問題，需要覆寫衝突的方法：

```java
// interfaces/Jim.java
import java.util.*;

interface Jim1 {
    default void jim() {
        System.out.println("Jim1::jim");
    }
}

interface Jim2 {
    default void jim() {
        System.out.println("Jim2::jim");
    }
}

public class Jim implements Jim1, Jim2 {
    @Override
    public void jim() {
        Jim2.super.jim();
    }
    
    public static void main(String[] args) {
        new Jim().jim();
    }
}
```

輸出：

```
Jim2::jim
```

當然，你可以重定義 `jim()` 方法，但是也能像上例中那樣使用 **super** 關鍵字選擇基類實現中的一種。

### 介面中的靜態方法

Java 8 允許在介面中添加靜態方法。這麼做能恰當地把工具功能置於介面中，從而操作介面，或者成為通用的工具：

```java
// onjava/Operations.java
package onjava;
import java.util.*;

public interface Operations {
    void execute();
    
    static void runOps(Operations... ops) {
        for (Operations op: ops) {
            op.execute();
        }
    }
    
    static void show(String msg) {
        System.out.println(msg);
    }
}
```


這是*模板方法*設計模式的一個版本（在“設計模式”一章中詳細描述），`runOps()` 是一個模板方法。`runOps()` 使用可變參數列表，因而我們可以傳入任意多的 **Operation** 參數並按順序執行它們：


```java
// interface/Machine.java
import java.util.*;
import onjava.Operations;

class Bing implements Operations {
    @Override
    public void execute() {
        Operations.show("Bing");
    }
}

class Crack implements Operations {
    @Override
    public void execute() {
        Operations.show("Crack");
    }
}

class Twist implements Operations {
    @Override
    public void execute() {
        Operations.show("Twist");
    }
}

public class Machine {
    public static void main(String[] args) {
        Operations.runOps(
        	new Bing(), new Crack(), new Twist());
    }
}
```

輸出：

```
Bing
Crack
Twist
```

這裡展示了建立 **Operations** 的不同方式：一個外部類(Bing)，一個匿名類，一個方法引用和 lambda 表達式——毫無疑問用在這裡是最好的解決方法。

這個特性是一項改善，因為它允許把靜態方法放在更合適的地方。

### Instrument 作為介面

回顧下樂器的例子，使用介面的話：

![類圖](../images/1562737974623.png)

類 **Woodwind** 和 **Brass** 說明一旦實現了某個介面，那麼其實現就變成一個普通類，可以按一般方式擴展它。

介面的工作方式使得我們不需要顯式聲明其中的方法為 **public**，它們自動就是 **public** 的。`play()` 和 `adjust()` 使用 **default** 關鍵字定義實現。在 Java 8 之前，這些定義要在每個實現中重複實現，顯得多餘且令人煩惱：

```java
// interfaces/music5/Music5.java
// {java interfaces.music5.Music5}
package interfaces.music5;
import polymorphism.music.Note;

interface Instrument {
    // Compile-time constant:
    int VALUE = 5; // static & final
    
    default void play(Note n)  // Automatically public 
        System.out.println(this + ".play() " + n);
    }
    
    default void adjust() {
        System.out.println("Adjusting " + this);
    }
}

class Wind implements Instrument {
    @Override
    public String toString() {
        return "Wind";
    }
}

class Percussion implements Instrument {
    @Override
    public String toString() {
        return "Percussion";
    }
}

class Stringed implements Instrument {
    @Override
    public String toString() {
        return "Stringed";
    }
}

class Brass extends Wind {
    @Override
    public String toString() {
        return "Brass";
    }
}

class Woodwind extends Wind {
    @Override
    public String toString() {
        return "Woodwind";
    }
}

public class Music5 {
    // Doesn't care about type, so new types
    // added to the system still work right:
    static void tune(Instrument i) {
        // ...
        i.play(Note.MIDDLE_C);
    }
    
    static void tuneAll(Instrument[] e) {
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
        }
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

這個版本的例子的另一個變化是：`what()` 被修改為 `toString()` 方法，因為 `toString()` 實現的正是 `what()` 方法要實現的邏輯。因為 `toString()` 是根基類 **Object** 的方法，所以它不需要出現在介面中。

注意到，無論是將其向上轉型為稱作 **Instrument** 的普通類，或稱作 **Instrument** 的抽象類，還是叫作 **Instrument** 的介面，其行為都是相同的。事實上，從 `tune()` 方法上看不出來 **Instrument** 到底是一個普通類、抽象類，還是一個介面。

<!-- Abstract Classes vs. Interfaces -->

## 抽象類和介面

尤其是在 Java 8 引入 **default** 方法之後，選擇用抽象類還是用介面變得更加令人困惑。下表做了明確的區分：

|         特性         |                            介面                            |                  抽象類                  |
| :------------------: | :--------------------------------------------------------: | :--------------------------------------: |
|         組合         |                    新類可以組合多個介面                    |            只能繼承單一抽象類            |
|         狀態         |        不能包含屬性（除了靜態屬性，不支援物件狀態）        | 可以包含屬性，非抽象方法可能引用這些屬性 |
| 預設方法 和 抽象方法 | 不需要在子類中實現預設方法。預設方法可以引用其他介面的方法 |         必須在子類中實現抽象方法         |
|        構造器        |                         沒有構造器                         |               可以有構造器               |
|        可見性        |                      隱式 **public**                       |       可以是 **protected** 或 "friendly"        |

抽象類仍然是一個類，在建立新類時只能繼承它一個。而建立類的過程中可以實現多個介面。

有一條實際經驗：在合理的範圍內儘可能地抽象。因此，更傾向使用介面而不是抽象類。只有當必要時才使用抽象類。除非必須使用，否則不要用介面和抽象類。大多數時候，普通類已經做得很好，如果不行的話，再移動到介面或抽象類中。

<!-- Complete Decoupling -->

## 完全解耦

每當一個方法與一個類而不是介面一起工作時(當方法的參數是類而不是介面)，你只能應用那個類或它的子類。如果你想把這方法應用到一個繼承層次之外的類，是做不到的。介面在很大程度上放寬了這個限制，因而使用介面可以編寫復用性更好的程式碼。

例如有一個類 **Processor** 有兩個方法 `name()` 和 `process()`。`process()` 方法接受輸入，修改並輸出。把這個類作為基類用來建立各種不同類型的 **Processor**。下例中，**Processor** 的各個子類修改 String 物件（注意，返回類型可能是協變類型而非參數類型）：

```java
// interfaces/Applicator.java
import java.util.*;

class Processor {
    public String name() {
        return getClass().getSimpleName();
    }
    
    public Object process(Object input) {
        return input;
    }
}

class Upcase extends Processor {
    // 返回協變類型
    @Override 
    public String process(Object input) {
        return ((String) input).toUpperCase();
    }
}

class Downcase extends Processor {
    @Override
    public String process(Object input) {
        return ((String) input).toLowerCase();
    }
}

class Splitter extends Processor {
    @Override
    public String process(Object input) {
        // split() divides a String into pieces:
        return Arrays.toString(((String) input).split(" "));
    }
}

public class Applicator {
    public static void apply(Processor p, Object s) {
        System.out.println("Using Processor " + p.name());
        System.out.println(p.process(s));
    }
    
    public static void main(String[] args) {
        String s = "We are such stuff as dreams are made on";
        apply(new Upcase(), s);
        apply(new Downcase(), s);
        apply(new Splitter(), s);
    }
}
```

輸出：

```
Using Processor Upcase
WE ARE SUCH STUFF AS DREAMS ARE MADE ON
Using Processor Downcase
we are such stuff as dreams are made on
Using Processor Splitter
[We, are, such, stuff, as, dreams, are, made, on]
```

**Applicator** 的 `apply()` 方法可以接受任何類型的 **Processor**，並將其應用到一個 **Object** 物件上輸出結果。像本例中這樣，建立一個能根據傳入的參數類型從而具備不同行為的方法稱為*策略*設計模式。方法包含演算法中不變的部分，策略包含變化的部分。策略就是傳入的物件，它包含要執行的程式碼。在這裡，**Processor** 物件是策略，`main()` 方法展示了三種不同的應用於 **String s** 上的策略。

`split()` 是 **String** 類中的方法，它接受 **String** 類型的物件並以傳入的參數作為分割界限，返回一個陣列 **String[]**。在這裡用它是為了更快地建立 **String** 陣列。

假設現在發現了一組電子濾波器，它們看起來好像能使用 **Applicator** 的 `apply()` 方法：

```java
// interfaces/filters/Waveform.java
package interfaces.filters;

public class Waveform {
    private static long counter;
    private final long id = counter++;
    
    @Override
    public String toString() {
        return "Waveform " + id;
    }
}

// interfaces/filters/Filter.java
package interfaces.filters;

public class Filter {
    public String name() {
        return getClass().getSimpleName();
    }
    
    public Waveform process(Waveform input) {
        return input;
    }
}

// interfaces/filters/LowPass.java
package interfaces.filters;

public class LowPass extends Filter {
    double cutoff;
    
    public LowPass(double cutoff) {
        this.cutoff = cutoff;
    }
    
    @Override
    public Waveform process(Waveform input) {
        return input; // Dummy processing 啞處理
    }
}

// interfaces/filters/HighPass.java
package interfaces.filters;

public class HighPass extends Filter {
    double cutoff;
    
    public HighPass(double cutoff) {
        this.cutoff = cutoff;
    }
    
    @Override
    public Waveform process(Waveform input) {
        return input;
    }
}

// interfaces/filters/BandPass.java
package interfaces.filters;

public class BandPass extends Filter {
    double lowCutoff, highCutoff;
    
    public BandPass(double lowCut, double highCut) {
        lowCutoff = lowCut;
        highCutoff = highCut;
    }
    
    @Override
    public Waveform process(Waveform input) {
        return input;
    }
}
```

**Filter** 類與 **Processor** 類具有相同的介面元素，但是因為它不是繼承自 **Processor** —— 因為 **Filter** 類的建立者根本不知道你想將它當作 **Processor** 使用 —— 因此你不能將 **Applicator** 的 `apply()` 方法應用在 **Filter** 類上，即使這樣做也能正常執行。主要是因為 **Applicator** 的 `apply()` 方法和 **Processor** 過於耦合，這阻止了 **Applicator** 的 `apply()` 方法被復用。另外要注意的一點是 Filter 類中 `process()` 方法的輸入輸出都是 **Waveform**。

但如果 **Processor** 是一個介面，那麼限制就會變得鬆動到足以復用 **Applicator** 的 `apply()` 方法，用來接受那個介面參數。下面是修改後的 **Processor** 和 **Applicator** 版本：

```java
// interfaces/interfaceprocessor/Processor.java
package interfaces.interfaceprocessor;

public interface Processor {
    default String name() {
        return getClass().getSimpleName();
    }
    
    Object process(Object input);
}

// interfaces/interfaceprocessor/Applicator.java
package interfaces.interfaceprocessor;

public class Applicator {
    public static void apply(Processor p, Object s) {
        System.out.println("Using Processor " + p.name());
        System.out.println(p.process(s));
    }
}
```

復用程式碼的第一種方式是用戶端程式設計師遵循介面編寫類，像這樣：

```java
// interfaces/interfaceprocessor/StringProcessor.java
// {java interfaces.interfaceprocessor.StringProcessor}
package interfaces.interfaceprocessor;
import java.util.*;

interface StringProcessor extends Processor {
    @Override
    String process(Object input); // [1]
    String S = "If she weighs the same as a duck, she's made of wood"; // [2]
    
    static void main(String[] args) { // [3]
        Applicator.apply(new Upcase(), S);
        Applicator.apply(new Downcase(), S);
        Applicator.apply(new Splitter(), S);
    }
}

class Upcase implements StringProcessor {
    // 返回協變類型
    @Override
    public String process(Object input) {
        return ((String) input).toUpperCase();
    }
}

class Downcase implements StringProcessor {
    @Override
    public String process(Object input) {
        return ((String) input).toLowerCase();
    }
}

class Splitter implements StringProcessor {
    @Override
    public String process(Object input) {
        return Arrays.toString(((String) input).split(" "));
    }
}
```

輸出：

```
Using Processor Upcase
IF SHE WEIGHS THE SAME AS A DUCK, SHE'S MADE OF WOOD
Using Processor Downcase
if she weighs the same as a duck, she's made of wood
Using Processor Splitter
[If, she, weighs, the, same, as, a, duck,, she's, made, of, wood]
```

>[1] 該聲明不是必要的，即使移除它，編譯器也不會報錯。但是注意這裡的協變返回類型從 Object 變成了 String。
>
>[2] S 自動就是 final 和 static 的，因為它是在介面中定義的。
>
>[3] 可以在介面中定義 `main()` 方法。

這種方式運作得很好，然而你經常遇到的情況是無法修改類。例如在電子濾波器的例子中，類庫是被發現而不是建立的。在這些情況下，可以使用*適配器*設計模式。適配器允許程式碼接受已有的介面產生需要的介面，如下：

```java
// interfaces/interfaceprocessor/FilterProcessor.java
// {java interfaces.interfaceprocessor.FilterProcessor}
package interfaces.interfaceprocessor;
import interfaces.filters.*;

class FilterAdapter implements Processor {
    Filter filter;
    
    FilterAdapter(Filter filter) {
        this.filter = filter;
    }
    
    @Override
    public String name() {
        return filter.name();
    }
    
    @Override
    public Waveform process(Object input) {
        return filter.process((Waveform) input);
    }
}

public class FilterProcessor {
    public static void main(String[] args) {
        Waveform w = new Waveform();
        Applicator.apply(new FilterAdapter(new LowPass(1.0)), w);
        Applicator.apply(new FilterAdapter(new HighPass(2.0)), w);
        Applicator.apply(new FilterAdapter(new BandPass(3.0, 4.0)), w);
    }
}
```

輸出：

```
Using Processor LowPass
Waveform 0
Using Processor HighPass
Waveform 0
Using Processor BandPass
Waveform 0
```

在這種使用適配器的方式中，**FilterAdapter** 的構造器接受已有的介面 **Filter**，繼而產生需要的 **Processor** 介面的物件。你可能還注意到 **FilterAdapter** 中使用了委託。

協變允許我們從 `process()` 方法中產生一個 **Waveform** 而非 **Object** 物件。

將介面與實現解耦使得介面可以應用於多種不同的實現，因而程式碼更具可復用性。

<!-- Combining Multiple Interfaces -->

## 多介面結合

介面沒有任何實現——也就是說，沒有任何與介面相關的儲存——因此無法阻止結合的多介面。這是有價值的，因為你有時需要表示“一個 **x** 是一個 **a** 和一個 **b** 以及一個 **c**”。

![類圖](../images/1562999314238.png)

衍生類並不要求必須繼承自抽象的或“具體的”（沒有任何抽象方法）的基類。如果繼承一個非介面的類，那麼只能繼承一個類，其餘的基元素必須都是介面。需要將所有的介面名稱置於 **implements** 關鍵字之後且用逗號分隔。可以有任意多個介面，並可以向上轉型為每個介面，因為每個介面都是獨立的類型。下例展示了一個由多個介面組合而成的具體類產生的新類：

```java
// interfaces/Adventure.java
// Multiple interfaces
interface CanFight {
    void fight();
}

interface CanSwim {
    void swim();
}

interface CanFly {
    void fly();
}

class ActionCharacter {
    public void fight(){}
}

class Hero extends ActionCharacter implements CanFight, CanSwim, CanFly {
    public void swim() {}
    
    public void fly() {}
}

public class Adventure {
    public static void t(CanFight x) {
        x.fight();
    }
    
    public static void u(CanSwim x) {
        x.swim();
    }
    
    public static void v(CanFly x) {
        x.fly();
    }
    
    public static void w(ActionCharacter x) {
        x.fight();
    }
    
    public static void main(String[] args) {
        Hero h = new Hero();
        t(h); // Treat it as a CanFight
        u(h); // Treat it as a CanSwim
        v(h); // Treat it as a CanFly
        w(h); // Treat it as an ActionCharacter
    }
}
```

類 **Hero** 結合了具體類 **ActionCharacter** 和介面 **CanFight**、**CanSwim** 和 **CanFly**。當透過這種方式結合具體類和介面時，需要將具體類放在前面，後面跟著介面（否則編譯器會報錯）。

介面 **CanFight** 和類 **ActionCharacter** 中的 `fight()` 方法簽名相同，而在類 Hero 中也沒有提供 `fight()` 的定義。可以擴展一個介面，但是得到的是另一個介面。當想建立一個物件時，所有的定義必須首先都存在。類 **Hero** 中沒有顯式地提供 `fight()` 的定義，是由於該方法在類 **ActionCharacter** 中已經定義過，這樣才使得建立 **Hero** 物件成為可能。

在類 **Adventure** 中可以看到四個方法，它們把不同的介面和具體類作為參數。當建立一個 **Hero** 物件時，它可以被傳入這些方法中的任意一個，意味著它可以依次向上轉型為每個介面。Java 中這種介面的設計方式，使得程式設計師不需要付出特別的努力。

記住，前面例子展示了使用介面的核心原因之一：為了能夠向上轉型為多個基類型（以及由此帶來的靈活性）。然而，使用介面的第二個原因與使用抽象基類相同：防止用戶端程式設計師建立這個類的物件，確保這僅僅只是一個介面。這帶來了一個問題：應該使用介面還是抽象類呢？如果建立不帶任何方法定義或成員變數的基類，就選擇介面而不是抽象類。事實上，如果知道某事物是一個基類，可以考慮用介面實現它（這個主題在本章總結會再次討論）。

<!-- Extending an Interface with Inheritance -->

## 使用繼承擴展介面

透過繼承，可以很容易在介面中增加方法聲明，還可以在新介面中結合多個介面。這兩種情況都可以得到新介面，如下例所示：

```java
// interfaces/HorrorShow.java
// Extending an interface with inheritance
interface Monster {
    void menace();
}

interface DangerousMonster extends Monster {
    void destroy();
}

interface Lethal {
    void kill();
}

class DragonZilla implements DangerousMonster {
    @Override
    public void menace() {}
    
    @Override
    public void destroy() {}
}

interface Vampire extends DangerousMonster, Lethal {
    void drinkBlood();
}

class VeryBadVampire implements Vampire {
    @Override
    public void menace() {}
    
    @Override
    public void destroy() {}
    
    @Override
    public void kill() {}
    
    @Override
    public void drinkBlood() {}
}

public class HorrorShow {
    static void u(Monster b) {
        b.menace();
    }
    
    static void v(DangerousMonster d) {
        d.menace();
        d.destroy();
    }
    
    static void w(Lethal l) {
        l.kill();
    }
    
    public static void main(String[] args) {
        DangerousMonster barney = new DragonZilla();
        u(barney);
        v(barney);
        Vampire vlad = new VeryBadVampire();
        u(vlad);
        v(vlad);
        w(vlad);
    }
}
```

介面 **DangerousMonster** 是 **Monster** 簡單擴展的一個新介面，類 **DragonZilla** 實現了這個介面。

**Vampire** 中使用的語法僅適用於介面繼承。通常來說，**extends** 只能用於單一類，但是在構建介面時可以引用多個基類介面。注意到，介面名之間用逗號分隔。

### 結合介面時的命名衝突

當實現多個介面時可能會存在一個小陷阱。在前面的例子中，**CanFight** 和 **ActionCharacter** 具有完全相同的 `fight()` 方法。完全相同的方法沒有問題，但是如果它們的簽名或返回類型不同會怎麼樣呢？這裡有一個例子：

```java
// interfaces/InterfaceCollision.java
interface I1 {
    void f();
}

interface I2 {
    int f(int i);
}

interface I3 {
    int f();
}

class C {
    public int f() {
        return 1;
    }
}

class C2 implements I1, I2 {
    @Override
    public void f() {}
    
    @Override
    public int f(int i) {
        return 1;  // 重載
    }
}

class C3 extends C implements I2 {
    @Override
    public int f(int i) {
        return 1; // 重載
    }
}

class C4 extends C implements I3 {
    // 完全相同，沒問題
    @Override
    public int f() {
        return 1;
    }
}

// 方法的返回類型不同
//- class C5 extends C implements I1 {}
//- interface I4 extends I1, I3 {}
```

覆寫、實現和重載令人不快地攪和在一起帶來了困難。同時，重載方法僅根據返回類型是區分不了的。當不注釋最後兩行時，報錯訊息如下：

```
error: C5 is not abstract and does not override abstract
method f() in I1
class C5 extends C implements I1 {}
error: types I3 and I1 are incompatible; both define f(),
but with unrelated return types
interfacce I4 extends I1, I3 {}
```

當打算組合介面時，在不同的介面中使用相同的方法名通常會造成程式碼可讀性的混亂，儘量避免這種情況。

<!-- Adapting to an Interface -->

## 介面適配

介面最吸引人的原因之一是相同的介面可以有多個實現。在簡單情況下體現在一個方法接受介面作為參數，該介面的實現和傳遞物件則取決於方法的使用者。

因此，介面的一種常見用法是前面提到的*策略*設計模式。編寫一個方法執行某些操作並接受一個指定的介面作為參數。可以說：“只要物件遵循介面，就可以呼叫方法” ，這使得方法更加靈活，通用，並更具可復用性。

例如，類 **Scanner** 的構造器接受的是一個 **Readable** 介面（在“字串”一章中學習更多相關內容）。你會發現 **Readable** 沒有用作 Java 標準庫中其他任何方法的參數——它是單獨為 **Scanner** 建立的，因此 **Scanner** 沒有將其參數限制為某個特定類。透過這種方式，**Scanner** 可以與更多的類型協作。如果你建立了一個新類並想讓 **Scanner** 作用於它，就讓它實現 **Readable** 介面，像這樣：

```java
// interfaces/RandomStrings.java
// Implementing an interface to conform to a method
import java.nio.*;
import java.util.*;

public class RandomStrings implements Readable {
    private static Random rand = new Random(47);
    private static final char[] CAPITALS = "ABCDEFGHIJKLMNOPQRSTUVWXYZ".toCharArray();
    private static final char[] LOWERS = "abcdefghijklmnopqrstuvwxyz".toCharArray();
    private static final char[] VOWELS = "aeiou".toCharArray();
    private int count;
    
    public RandomStrings(int count) {
        this.count = count;
    }
    
    @Override
    public int read(CharBuffer cb) {
        if (count-- == 0) {
            return -1; // indicates end of input
        }
        cb.append(CAPITALS[rand.nextInt(CAPITALS.length)]);
        for (int i = 0; i < 4; i++) {
            cb.append(VOWELS[rand.nextInt(VOWELS.length)]);
            cb.append(LOWERS[rand.nextInt(LOWERS.length)]);
        }
        cb.append(" ");
        return 10; // Number of characters appended
    }
    
    public static void main(String[] args) {
        Scanner s = new Scanner(new RandomStrings(10));
        while (s.hasNext()) {
            System.out.println(s.next());
        }
    }
}
```

輸出：

```
Yazeruyac
Fowenucor
Goeazimom
Raeuuacio
Nuoadesiw
Hageaikux
Ruqicibui
Numasetih
Kuuuuozog
Waqizeyoy
```

**Readable** 介面只需要實現 `read()` 方法（注意 `@Override` 註解的突出方法）。在 `read()` 方法裡，將輸入內容添加到 **CharBuffer** 參數中（有多種方法可以實現，查看 **CharBuffer** 文件），或在沒有輸入時返回 **-1**。

假設你有一個類沒有實現 **Readable** 介面，怎樣才能讓 **Scanner** 作用於它呢？下面是一個產生隨機浮點數的例子：

```java
// interfaces/RandomDoubles.java
import java.util.*;

public interface RandomDoubles {
    Random RAND = new Random(47);
    
    default double next() {
        return RAND.nextDouble();
    }
    
    static void main(String[] args) {
        RandomDoubles rd = new RandomDoubles(){};
        for (int i = 0; i < 7; i++) {
            System.out.println(rd.next() + " ");
        }
    }
}
```

輸出：

```
0.7271157860730044 
0.5309454508634242 
0.16020656493302599 
0.18847866977771732 
0.5166020801268457 
0.2678662084200585 
0.2613610344283964
```

我們可以再次使用適配器模式，但這裡適配器類可以實現兩個介面。因此，透過關鍵字 **interface** 提供的多繼承，我們可以建立一個既是 **RandomDoubles**，又是 **Readable** 的類：

```java
// interfaces/AdaptedRandomDoubles.java
// creating an adapter with inheritance
import java.nio.*;
import java.util.*;

public class AdaptedRandomDoubles implements RandomDoubles, Readable {
    private int count;
    
    public AdaptedRandomDoubles(int count) {
        this.count = count;
    }
    
    @Override
    public int read(CharBuffer cb) {
        if (count-- == 0) {
            return -1;
        }
        String result = Double.toString(next()) + " ";
        cb.append(result);
        return result.length();
    }
    
    public static void main(String[] args) {
        Scanner s = new Scanner(new AdaptedRandomDoubles(7));
        while (s.hasNextDouble()) {
            System.out.print(s.nextDouble() + " ");
        }
    }
}
```

輸出：

```
0.7271157860730044 0.5309454508634242 
0.16020656493302599 0.18847866977771732 
0.5166020801268457 0.2678662084200585 
0.2613610344283964
```

因為你可以以這種方式在已有類中增加新介面，所以這就意味著一個接受介面類型的方法提供了一種讓任何類都可以與該方法進行適配的方式。這就是使用介面而不是類的強大之處。

<!-- Fields in Interfaces -->

## 介面欄位

因為介面中的欄位都自動是 **static** 和 **final** 的，所以介面就成為了建立一組常量的方便的工具。在 Java 5 之前，這是產生與 C 或 C++ 中的 enum (列舉類型) 具有相同效果的唯一方式。所以你可能在 Java 5 之前的程式碼中看到：

```java
// interfaces/Months.java
// Using interfaces to create groups of constants
public interface Months {
    int 
    JANUARY = 1, FEBRUARY = 2, MARCH = 3,
    APRIL = 4, MAY = 5, JUNE = 6, JULY = 7,
    AUGUST = 8, SEPTEMBER = 9, OCTOBER = 10,
    NOVEMBER = 11, DECEMBER = 12;
}
```

注意 Java 中使用大寫字母的風格定義具有初始化值的 **static** **final** 變數。介面中的欄位自動是 **public** 的，所以沒有顯式指明這點。

自 Java 5 開始，我們有了更加強大和靈活的關鍵字 **enum**，那麼在介面中定義常量組就顯得沒什麼意義了。然而當你閱讀遺留的程式碼時，在很多場合你還會碰到這種舊的習慣用法。在“列舉”一章中你會學習到更多關於列舉的內容。

### 初始化介面中的欄位

介面中定義的欄位不能是“空 **final**"，但是可以用非常量表達式初始化。例如：

```java
// interfaces/RandVals.java
// Initializing interface fields with
// non-constant initializers
import java.util.*;

public interface RandVals {
    Random RAND = new Random(47);
    int RANDOM_INT = RAND.nextInt(10);
    long RANDOM_LONG = RAND.nextLong() * 10;
    float RANDOM_FLOAT = RAND.nextLong() * 10;
    double RANDOM_DOUBLE = RAND.nextDouble() * 10;
}
```

因為欄位是 **static** 的，所以它們在類第一次被載入時初始化，這發生在任何欄位首次被訪問時。下面是個簡單的測試：

```java
// interfaces/TestRandVals.java
public class TestRandVals {
    public static void main(String[] args) {
        System.out.println(RandVals.RANDOM_INT);
        System.out.println(RandVals.RANDOM_LONG);
        System.out.println(RandVals.RANDOM_FLOAT);
        System.out.println(RandVals.RANDOM_DOUBLE);
    }
}
```

輸出：

```
8
-32032247016559954
-8.5939291E18
5.779976127815049
```

這些欄位不是介面的一部分，它們的值被儲存在介面的靜態儲存區域中。

<!-- Nesting Interfaces -->

## 介面嵌套

介面可以嵌套在類或其他介面中。下面揭示一些有趣的特性：

```java
// interfaces/nesting/NestingInterfaces.java
// {java interfaces.nesting.NestingInterfaces}
package interfaces.nesting;

class A {
    interface B {
        void f();
    }
    
    public class BImp implements B {
        @Override
        public void f() {}
    }
    
    public class BImp2 implements B {
        @Override
        public void f() {}
    }
    
    public interface C {
        void f();
    }
    
    class CImp implements C {
        @Override
        public void f() {}
    }
    
    private class CImp2 implements C {
        @Override
        public void f() {}
    }
    
    private interface D {
        void f();
    }
    
    private class DImp implements D {
        @Override
        public void f() {}
    }
    
    public class DImp2 implements D {
        @Override
        public void f() {}
    }
    
    public D getD() {
        return new DImp2();
    }
    
    private D dRef;
    
    public void receiveD(D d) {
        dRef = d;
        dRef.f();
    }
}

interface E {
    interface G {
        void f();
    }
    // Redundant "public"
    public interface H {
        void f();
    }
    
    void g();
    // Cannot be private within an interface
    //- private interface I {}
}

public class NestingInterfaces {
    public class BImp implements A.B {
        @Override
        public void f() {}
    }
    
    class CImp implements A.C {
        @Override
        public void f() {}
    }
    // Cannot implements a private interface except
    // within that interface's defining class:
    //- class DImp implements A.D {
    //- public void f() {}
    //- }
    class EImp implements E {
        @Override
        public void g() {}
    }
    
    class EGImp implements E.G {
        @Override
        public void f() {}
    }
    
    class EImp2 implements E {
        @Override
        public void g() {}
        
        class EG implements E.G {
            @Override
            public void f() {}
        }
    }
    
    public static void main(String[] args) {
        A a = new A();
        // Can't access to A.D:
        //- A.D ad = a.getD();
        // Doesn't return anything but A.D:
        //- A.DImp2 di2 = a.getD();
        // cannot access a member of the interface:
        //- a.getD().f();
        // Only another A can do anything with getD():
        A a2 = new A();
        a2.receiveD(a.getD());
    }
}
```

在類中嵌套介面的語法是相當顯而易見的。就像非嵌套介面一樣，它們具有 **public** 或包訪問權限的可見性。

作為一種新添加的方式，介面也可以是 **private** 的，例如 **A.D**（同樣的語法同時適用於嵌套介面和嵌套類）。那麼 **private** 嵌套介面有什麼好處呢？你可能猜測它只是被用來實現一個 **private** 內部類，就像 **DImp**。然而 **A.DImp2** 展示了它可以被實現為 **public** 類，但是 **A.DImp2** 只能被自己使用，你無法說它實現了 **private** 介面 **D**，所以實現 **private** 介面是一種可以強制該介面中的方法定義不會添加任何類型訊息（即不可以向上轉型）的方式。

`getD()` 方法產生了一個與 **private** 介面有關的窘境。它是一個 **public** 方法卻返回了對 **private** 介面的引用。能對這個返回值做些什麼呢？`main()` 方法裡進行了一些使用返回值的嘗試但都失敗了。返回值必須交給有權使用它的物件，本例中另一個 **A** 通過 `receiveD()` 方法接受了它。

介面 **E** 說明了介面之間也能嵌套。然而，作用於介面的規則——尤其是，介面中的元素必須是 **public** 的——在此都會被嚴格執行，所以嵌套在另一個介面中的介面自動就是 **public** 的，不能指明為 **private**。

類 **NestingInterfaces** 展示了嵌套介面的不同實現方式。尤其是當實現某個介面時，並不需要實現嵌套在其內部的介面。同時，**private** 介面不能在定義它的類之外被實現。

添加這些特性的最初原因看起來像是出於對嚴格的語法一致性的考慮，但是我通常認為，一旦你了解了某種特性，就總能找到其用武之地。

<!-- Interfaces and Factories -->

## 介面和工廠方法模式

介面是多實現的途徑，而生成符合某個介面的物件的典型方式是*工廠方法*設計模式。不同於直接呼叫構造器，只需呼叫工廠物件中的建立方法就能生成物件的實現——理論上，透過這種方式可以將介面與實現的程式碼完全分離，使得可以透明地將某個實現取代為另一個實現。這裡是一個展示工廠方法結構的例子：

```java
// interfaces/Factories.java
interface Service {
    void method1();
    void method2();
}

interface ServiceFactory {
    Service getService();
}

class Service1 implements Service {
    Service1() {} // Package access
    
    @Override
    public void method1() {
        System.out.println("Service1 method1");
    }
    
    @Override
    public void method2() {
        System.out.println("Service1 method2");
    }
}

class Service1Factory implements ServiceFactory {
    @Override
    public Service getService() {
        return new Service1();
    }
}

class Service2 implements Service {
    Service2() {} // Package access
    
    @Override
    public void method1() {
        System.out.println("Service2 method1");
    }
    
    @Override
    public void method2() {
        System.out.println("Service2 method2");
    }
}

class Service2Factory implements ServiceFactory {
    @Override
    public Service getService() {
        return new Service2();
    }
}

public class Factories {
    public static void serviceConsumer(ServiceFactory fact) {
        Service s = fact.getService();
        s.method1();
        s.method2();
    }
    
    public static void main(String[] args) {
        serviceConsumer(new Service1Factory());
        // Services are completely interchangeable:
        serviceConsumer(new Service2Factory());
    }
}
```

輸出：

```
Service1 method1
Service1 method2
Service2 method1
Service2 method2
```

如果沒有工廠方法，程式碼就必須在某處指定將要建立的 **Service** 的確切類型，從而呼叫恰當的構造器。

為什麼要添加額外的間接層呢？一個常見的原因是建立框架。假設你正在建立一個遊戲系統；例如，在相同的棋盤下西洋棋和西洋跳棋：

```java
// interfaces/Games.java
// A Game framework using Factory Methods
interface Game {
    boolean move();
}

interface GameFactory {
    Game getGame();
}

class Checkers implements Game {
    private int moves = 0;
    private static final int MOVES = 3;
    
    @Override
    public boolean move() {
        System.out.println("Checkers move " + moves);
        return ++moves != MOVES;
    }
}

class CheckersFactory implements GameFactory {
    @Override
    public Game getGame() {
        return new Checkers();
    }
}

class Chess implements Game {
    private int moves = 0;
    private static final int MOVES = 4;
    
    @Override
    public boolean move() {
        System.out.println("Chess move " + moves);
        return ++moves != MOVES;
    }
}

class ChessFactory implements GameFactory {
    @Override
    public Game getGame() {
        return new Chess();
    }
}

public class Games {
    public static void playGame(GameFactory factory) {
        Game s = factory.getGame();
        while (s.move()) {
            ;
        }
    }
    
    public static void main(String[] args) {
        playGame(new CheckersFactory());
        playGame(new ChessFactory());
    }
}
```

輸出：

```
Checkers move 0
Checkers move 1
Checkers move 2
Chess move 0
Chess move 1
Chess move 2
Chess move 3
```

如果類 **Games** 表示一段很複雜的程式碼，那麼這種方式意味著你可以在不同類型的遊戲裡復用這段程式碼。你可以再想像一些能夠從這個模式中受益的更加精巧的遊戲。

在下一章，你將會看到一種更加優雅的使用匿名內部類的工廠實現方式。

<!-- Summary -->

## 本章小結

認為介面是好的選擇，從而使用介面不用具體類，這具有誘惑性。幾乎任何時候，建立類都可以替代為建立一個介面和工廠。

很多人都掉進了這個陷阱，只要有可能就建立介面和工廠。這種邏輯看起來像是可能會使用不同的實現，所以總是添加這種抽象性。這變成了一種過早的設計最佳化。

任何抽象性都應該是由真正的需求驅動的。當有必要時才應該使用介面進行重構，而不是到處添加額外的間接層，從而帶來額外的複雜性。這種複雜性非常顯著，如果你讓某人去處理這種複雜性，只是因為你意識到“以防萬一”而添加新介面，而沒有其他具有說服力的原因——好吧，如果我碰上了這種設計，就會質疑此人所作的所有其他設計了。

恰當的原則是優先使用類而不是介面。從類開始，如果使用介面的必要性變得很明確，那麼就重構。介面是一個偉大的工具，但它們容易被濫用。

<!-- 分頁 -->

<div style="page-break-after: always;"></div>
