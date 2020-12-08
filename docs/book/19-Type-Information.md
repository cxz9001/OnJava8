[TOC]

<!-- Type Information -->
# 第十九章 類型訊息

> RTTI（RunTime Type Information，執行時類型訊息）能夠在程式執行時發現和使用類型訊息

RTTI 把我們從只能在編譯期進行面向類型操作的禁錮中解脫了出來，並且讓我們可以使用某些非常強大的程式。對 RTTI 的需要，揭示了物件導向設計中許多有趣（並且複雜）的特性，同時也帶來了關於如何組織程式的基本問題。

本章將討論 Java 是如何在執行時識別物件和類訊息的。主要有兩種方式：

1. “傳統的” RTTI：假定我們在編譯時已經知道了所有的類型；
2. “反射”機制：允許我們在執行時發現和使用類的訊息。

<!-- The Need for RTTI -->

## 為什麼需要 RTTI

下面看一下我們已經很熟悉的一個例子，它使用了多態的類層次結構。基類 `Shape` 是泛化的類型，從它衍生出了三個具體類： `Circle` 、`Square` 和 `Triangle`（見一下圖所示）。

![多態例子Shape的類層次結構圖](../images/image-20190409114913825-4781754.png)

這是一個典型的類層次結構圖，基類位於頂部，衍生類向下擴展。物件導向編程的一個基本目的是：讓程式碼只操縱對基類(這裡即 `Shape` )的引用。這樣，如果你想添加一個新類(比如從 `Shape` 衍生出 `Rhomboid`)來擴展程式，就不會影響原來的程式碼。在這個例子中，`Shape` 介面中動態綁定了 `draw()` 方法，這樣做的目的就是讓用戶端程式設計師可以使用泛化的 `Shape` 引用來呼叫 `draw()`。`draw()` 方法在所有衍生類裡都會被覆蓋，而且由於它是動態綁定的，所以即使透過 `Shape` 引用來呼叫它，也能產生恰當的行為，這就是多態。

因此，我們通常會建立一個具體的物件(`Circle`、`Square` 或者 `Triangle`)，把它向上轉型成 `Shape` (忽略物件的具體類型)，並且在後面的程式中使用 `Shape` 引用來呼叫在具體物件中被重載的方法（如 `draw()`）。

程式碼如下：

```java
// typeinfo/Shapes.java
import java.util.stream.*;

abstract class Shape {
    void draw() { System.out.println(this + ".draw()"); }
    @Override
    public abstract String toString();
}

class Circle extends Shape {
    @Override
    public String toString() { return "Circle"; }
}

class Square extends Shape {
    @Override
    public String toString() { return "Square"; }
}

class Triangle extends Shape {
    @Override
    public String toString() { return "Triangle"; }
}

public class Shapes {
    public static void main(String[] args) {
        Stream.of(
            new Circle(), new Square(), new Triangle())
            .forEach(Shape::draw);
    }
}
```

輸出結果：

```
Circle.draw()
Square.draw()
Triangle.draw()
```

基類中包含 `draw()` 方法，它透過傳遞 `this` 參數傳遞給 `System.out.println()`，間接地使用 `toString()` 列印類標識符(注意：這裡將 `toString()` 聲明為 `abstract`，以此強制繼承者覆蓋該方法，並防止對 `Shape` 的實例化)。如果某個物件出現在字串表達式中(涉及"+"和字串物件的表達式)，`toString()` 方法就會被自動呼叫，以生成表示該物件的 `String`。每個衍生類都要覆蓋（從 `Object` 繼承來的）`toString()` 方法，這樣 `draw()` 在不同情況下就列印出不同的消息(多態)。

這個例子中，在把 `Shape` 物件放入 `Stream<Shape>` 中時就會進行向上轉型(隱式)，但在向上轉型的時候也遺失了這些物件的具體類型。對 `stream` 而言，它們只是 `Shape` 物件。

嚴格來說，`Stream<Shape>` 實際上是把放入其中的所有物件都當做 `Object` 物件來持有，只是取元素時會自動將其類型轉為 `Shape`。這也是 RTTI 最基本的使用形式，因為在 Java 中，所有類型轉換的正確性檢查都是在執行時進行的。這也正是 RTTI 的含義所在：在執行時，識別一個物件的類型。

另外在這個例子中，類型轉換並不徹底：`Object` 被轉型為 `Shape` ，而不是 `Circle`、`Square` 或者 `Triangle`。這是因為目前我們只能確保這個 `Stream<Shape>` 儲存的都是 `Shape`：

- 編譯期，`stream` 和 Java 泛型系統確保放入 `stream` 的都是 `Shape` 物件（`Shape` 子類的物件也可視為 `Shape` 的物件），否則編譯器會報錯；
- 執行時，自動類型轉換確保了從 `stream` 中取出的物件都是 `Shape` 類型。

接下來就是多態機制的事了，`Shape` 物件實際執行什麼樣的程式碼，是由引用所指向的具體物件（`Circle`、`Square` 或者 `Triangle`）決定的。這也符合我們編寫程式碼的一般需求，通常，我們希望大部分程式碼儘可能少了解物件的具體類型，而是只與物件家族中的一個通用表示打交道（本例中即為 `Shape`）。這樣，程式碼會更容易寫，更易讀和維護；設計也更容易實現，更易於理解和修改。所以多態是物件導向的基本目標。

但是，有時你會碰到一些編程問題，在這些問題中如果你能知道某個泛化引用的具體類型，就可以把問題輕鬆解決。例如，假設我們允許使用者將某些幾何形狀突顯顯示，現在希望找到螢幕上所有突顯顯示的三角形；或者，我們現在需要旋轉所有圖形，但是想跳過圓形(因為圓形旋轉沒有意義)。這時我們就希望知道 `Stream<Shape>` 裡面的形狀具體是什麼類型，而 Java 實際上也滿足了我們的這種需求。使用 RTTI，我們可以查詢某個 `Shape` 引用所指向物件的確切類型，然後選擇或者剔除特例。

<!-- The Class Object -->
## Class 物件

要理解 RTTI 在 Java 中的工作原理，首先必須知道類型訊息在執行時是如何表示的。這項工作是由稱為 **`Class`物件** 的特殊物件完成的，它包含了與類有關的訊息。實際上，`Class` 物件就是用來建立該類所有"一般"物件的。Java 使用 `Class` 物件來實現 RTTI，即便是類型轉換這樣的操作都是用 `Class` 物件實現的。不僅如此，`Class` 類還提供了很多使用 RTTI 的其它方式。

類是程式的一部分，每個類都有一個 `Class` 物件。換言之，每當我們編寫並且編譯了一個新類，就會產生一個 `Class` 物件（更恰當的說，是被儲存在一個同名的 `.class` 文件中）。為了生成這個類的物件，Java 虛擬機 (JVM) 先會呼叫"類載入器"子系統把這個類載入到記憶體中。

類載入器子系統可能包含一條類載入器鏈，但有且只有一個**原生類載入器**，它是 JVM 實現的一部分。原生類載入器載入的是“可信類”（包括 Java API 類）。它們通常是從本機盤載入的。在這條鏈中，通常不需要添加額外的類載入器，但是如果你有特殊需求（例如以某種特殊的方式載入類，以支援 Web 伺服器應用，或者透過網路下載類），也可以掛載額外的類載入器。

所有的類都是第一次使用時動態載入到 JVM 中的，當程式建立第一個對類的靜態成員的引用時，就會載入這個類。

> 其實構造器也是類的靜態方法，雖然構造器前面並沒有 `static` 關鍵字。所以，使用 `new` 操作符建立類的新物件，這個操作也算作對類的靜態成員引用。

因此，Java 程式在它開始執行之前並沒有被完全載入，很多部分是在需要時才會載入。這一點與許多傳統程式語言不同，動態載入使得 Java 具有一些靜態載入語言（如 C++）很難或者根本不可能實現的特性。

類載入器首先會檢查這個類的 `Class` 物件是否已經載入，如果尚未載入，預設的類載入器就會根據類名尋找 `.class` 文件（如果有附加的類載入器，這時候可能就會在資料庫中或者透過其它方式獲得位元組碼）。這個類的位元組碼被載入後，JVM 會對其進行驗證，確保它沒有損壞，並且不包含不良的 Java 程式碼(這是 Java 安全防範的一種措施)。

一旦某個類的 `Class` 物件被載入記憶體，它就可以用來建立這個類的所有物件。下面的示範程式可以證明這點：

```java
// typeinfo/SweetShop.java
// 檢查類載入器工作方式
class Cookie {
    static { System.out.println("Loading Cookie"); }
}

class Gum {
    static { System.out.println("Loading Gum"); }
}

class Candy {
    static { System.out.println("Loading Candy"); }
}

public class SweetShop {
    public static void main(String[] args) {
        System.out.println("inside main");
        new Candy();
        System.out.println("After creating Candy");
        try {
            Class.forName("Gum");
        } catch(ClassNotFoundException e) {
            System.out.println("Couldn't find Gum");
        }
        System.out.println("After Class.forName(\"Gum\")");
        new Cookie();
        System.out.println("After creating Cookie");
    }
}
```

輸出結果：

```
inside main
Loading Candy
After creating Candy
Loading Gum
After Class.forName("Gum")
Loading Cookie
After creating Cookie
```

上面的程式碼中，`Candy`、`Gum` 和 `Cookie` 這幾個類都有一個 `static{...}` 靜態初始化塊，這些靜態初始化塊在類第一次被載入的時候就會執行。也就是說，靜態初始化塊會列印出相應的訊息，告訴我們這些類分別是什麼時候被載入了。而在主方法裡面，建立物件的程式碼都放在了 `print()` 語句之間，以幫助我們判斷類載入的時間點。

從輸出中可以看到，`Class` 物件僅在需要的時候才會被載入，`static` 初始化是在類載入時進行的。

程式碼裡面還有特別有趣的一行：

```java
Class.forName("Gum");
```

所有 `Class` 物件都屬於 `Class` 類，而且它跟其他普通物件一樣，我們可以獲取和操控它的引用(這也是類載入器的工作)。`forName()` 是 `Class` 類的一個靜態方法，我們可以使用 `forName()` 根據目標類的類名（`String`）得到該類的 `Class` 物件。上面的程式碼忽略了 `forName()` 的返回值，因為那個呼叫是為了得到它產生的“副作用”。從結果可以看出，`forName()` 執行的副作用是如果 `Gum` 類沒有被載入就載入它，而在載入的過程中，`Gum` 的 `static` 初始化塊被執行了。

還需要注意的是，如果 `Class.forName()` 找不到要載入的類，它就會拋出異常 `ClassNotFoundException`。上面的例子中我們只是簡單地報告了問題，但在更嚴密的程式裡，就要考慮在異常處理程序中把問題解決掉（具體例子詳見[設計模式](./25-Patterns)章節）。

無論何時，只要你想在執行時使用類型訊息，就必須先得到那個 `Class` 物件的引用。`Class.forName()` 就是實現這個功能的一個便捷途徑，因為使用該方法你不需要先持有這個類型 的物件。但是，如果你已經擁有了目標類的物件，那就可以透過呼叫 `getClass()` 方法來獲取 `Class` 引用了，這個方法來自根類 `Object`，它將返回表示該物件實際類型的 `Class` 物件的引用。`Class` 包含很多有用的方法，下面程式碼展示了其中的一部分：

```java
// typeinfo/toys/ToyTest.java
// 測試 Class 類
// {java typeinfo.toys.ToyTest}
package typeinfo.toys;

interface HasBatteries {}
interface Waterproof {}
interface Shoots {}

class Toy {
    // 注釋下面的無參數構造器會引起 NoSuchMethodError 錯誤
    Toy() {}
    Toy(int i) {}
}

class FancyToy extends Toy
implements HasBatteries, Waterproof, Shoots {
    FancyToy() { super(1); }
}

public class ToyTest {
    static void printInfo(Class cc) {
        System.out.println("Class name: " + cc.getName() +
            " is interface? [" + cc.isInterface() + "]");
        System.out.println(
            "Simple name: " + cc.getSimpleName());
        System.out.println(
            "Canonical name : " + cc.getCanonicalName());
    }

    public static void main(String[] args) {
        Class c = null;
        try {
            c = Class.forName("typeinfo.toys.FancyToy");
        } catch(ClassNotFoundException e) {
            System.out.println("Can't find FancyToy");
            System.exit(1);
        }

        printInfo(c);
        for(Class face : c.getInterfaces())
            printInfo(face);

        Class up = c.getSuperclass();
        Object obj = null;

        try {
            // Requires no-arg constructor:
            obj = up.newInstance();
        } catch(InstantiationException e) {
            System.out.println("Cannot instantiate");
            System.exit(1);
        } catch(IllegalAccessException e) {
            System.out.println("Cannot access");
            System.exit(1);
        }

        printInfo(obj.getClass());
    }
}
```

輸出結果：

```
Class name: typeinfo.toys.FancyToy is interface?
[false]
Simple name: FancyToy
Canonical name : typeinfo.toys.FancyToy
Class name: typeinfo.toys.HasBatteries is interface?
[true]
Simple name: HasBatteries
Canonical name : typeinfo.toys.HasBatteries
Class name: typeinfo.toys.Waterproof is interface?
[true]
Simple name: Waterproof
Canonical name : typeinfo.toys.Waterproof
Class name: typeinfo.toys.Shoots is interface? [true]
Simple name: Shoots
Canonical name : typeinfo.toys.Shoots
Class name: typeinfo.toys.Toy is interface? [false]
Simple name: Toy
Canonical name : typeinfo.toys.Toy
```

`FancyToy` 繼承自 `Toy` 並實現了 `HasBatteries`、`Waterproof` 和 `Shoots` 介面。在 `main` 方法中，我們建立了一個 `Class` 引用，然後在 `try` 語句裡面用 `forName()` 方法建立了一個 `FancyToy` 的類物件並賦值給該引用。需要注意的是，傳遞給 `forName()` 的字串必須使用類的全限定名（包含包名）。

`printInfo()` 函數使用 `getName()` 來產生完整類名，使用 `getSimpleName()` 產生不帶包名的類名，`getCanonicalName()` 也是產生完整類名（除內部類和陣列外，對大部分類產生的結果與 `getName()` 相同）。`isInterface()` 用於判斷某個 `Class` 物件代表的是否為一個介面。因此，通過 `Class` 物件，你可以得到關於該類型的所有訊息。

在主方法中呼叫的 `Class.getInterfaces()` 方法返回的是存放 `Class` 物件的陣列，裡面的 `Class` 物件表示的是那個類實現的介面。

另外，你還可以呼叫 `getSuperclass()` 方法來得到父類的 `Class` 物件，再用父類的 `Class` 物件呼叫該方法，重複多次，你就可以得到一個物件完整的類繼承結構。

`Class` 物件的 `newInstance()` 方法是實現“虛擬構造器”的一種途徑，虛擬構造器可以讓你在不知道一個類的確切類型的時候，建立這個類的物件。在前面的例子中，`up` 只是一個 `Class` 物件的引用，在編譯期並不知道這個引用會指向哪個類的 `Class` 物件。當你建立新實例時，會得到一個 `Object` 引用，但是這個引用指向的是 `Toy` 物件。當然，由於得到的是 `Object` 引用，目前你只能給它發送 `Object` 物件能夠接受的呼叫。而如果你想請求具體物件才有的呼叫，你就得先獲取該物件更多的類型訊息，並執行某種轉型。另外，使用 `newInstance()` 來建立的類，必須帶有無參數的構造器。在本章稍後部分，你將會看到如何透過 Java 的反射 API，用任意的構造器來動態地建立類的物件。

### 類字面常量

Java 還提供了另一種方法來生成類物件的引用：**類字面常量**。對上述程式來說，就像這樣：`FancyToy.class;`。這樣做不僅更簡單，而且更安全，因為它在編譯時就會受到檢查（因此不必放在 `try` 語句塊中）。並且它根除了對 `forName()` 方法的呼叫，所以效率更高。

類字面常量不僅可以應用於普通類，也可以應用於介面、陣列以及基本資料類型。另外，對於基本資料類型的包裝類，還有一個標準欄位 `TYPE`。`TYPE` 欄位是一個引用，指向對應的基本資料類型的 `Class` 物件，如下所示：

<figure>
<table style="text-align:center;">
  <thead>
    <tr>
      <th colspan="2">...等價於...</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>boolean.class</td>
      <td>Boolean.TYPE</td>
    </tr>
    <tr>
      <td>char.class</td>
      <td>Character.TYPE</td>
    </tr>
    <tr>
      <td>byte.class</td>
      <td>Byte.TYPE</td>
    </tr>
    <tr>
      <td>short.class</td>
      <td>Short.TYPE</td>
    </tr>
    <tr>
      <td>int.class</td>
      <td>Integer.TYPE</td>
    </tr>
    <tr>
      <td>long.class</td>
      <td>Long.TYPE</td>
    </tr>
    <tr>
      <td>float.class</td>
      <td>Float.TYPE</td>
    </tr>
    <tr>
      <td>double.class</td>
      <td>Double.TYPE</td>
    </tr>
    <tr>
      <td>void.class</td>
      <td>Void.TYPE</td>
    </tr>
  </tbody>
</table>
</figure>

我的建議是使用 `.class` 的形式，以保持與普通類的一致性。

注意，有一點很有趣：當使用 `.class` 來建立對 `Class` 物件的引用時，不會自動地初始化該 `Class` 物件。為了使用類而做的準備工作實際包含三個步驟：

1. **載入**，這是由類載入器執行的。該步驟將尋找位元組碼（通常在 classpath 所指定的路徑中尋找，但這並非是必須的），並從這些位元組碼中建立一個 `Class` 物件。

2. **連結**。在連結階段將驗證類中的位元組碼，為 `static` 欄位分配儲存空間，並且如果需要的話，將解析這個類建立的對其他類的所有引用。

3. **初始化**。如果該類具有超類，則先初始化超類，執行 `static` 初始化器和 `static` 初始化塊。

直到第一次引用一個 `static` 方法（構造器隱式地是 `static`）或者非常量的 `static` 欄位，才會進行類初始化。

```java
// typeinfo/ClassInitialization.java
import java.util.*;

class Initable {
    static final int STATIC_FINAL = 47;
    static final int STATIC_FINAL2 =
        ClassInitialization.rand.nextInt(1000);
    static {
        System.out.println("Initializing Initable");
    }
}

class Initable2 {
    static int staticNonFinal = 147;
    static {
        System.out.println("Initializing Initable2");
    }
}

class Initable3 {
    static int staticNonFinal = 74;
    static {
        System.out.println("Initializing Initable3");
    }
}

public class ClassInitialization {
    public static Random rand = new Random(47);
    public static void
    main(String[] args) throws Exception {
        Class initable = Initable.class;
        System.out.println("After creating Initable ref");
        // Does not trigger initialization:
        System.out.println(Initable.STATIC_FINAL);
        // Does trigger initialization:
        System.out.println(Initable.STATIC_FINAL2);
        // Does trigger initialization:
        System.out.println(Initable2.staticNonFinal);
        Class initable3 = Class.forName("Initable3");
        System.out.println("After creating Initable3 ref");
        System.out.println(Initable3.staticNonFinal);
    }
}
```

輸出結果：

```
After creating Initable ref
47
Initializing Initable
258
Initializing Initable2
147
Initializing Initable3
After creating Initable3 ref
74
```

初始化有效地實現了儘可能的“惰性”，從對 `initable` 引用的建立中可以看到，僅使用 `.class` 語法來獲得對類物件的引用不會引發初始化。但與此相反，使用 `Class.forName()` 來產生 `Class` 引用會立即就進行初始化，如 `initable3`。

如果一個 `static final` 值是“編譯期常量”（如 `Initable.staticFinal`），那麼這個值不需要對 `Initable` 類進行初始化就可以被讀取。但是，如果只是將一個欄位設定成為 `static` 和 `final`，還不足以確保這種行為。例如，對 `Initable.staticFinal2` 的訪問將強制進行類的初始化，因為它不是一個編譯期常量。

如果一個 `static` 欄位不是 `final` 的，那麼在對它訪問時，總是要求在它被讀取之前，要先進行連結（為這個欄位分配儲存空間）和初始化（初始化該儲存空間），就像在對 `Initable2.staticNonFinal` 的訪問中所看到的那樣。

### 泛化的 `Class` 引用

`Class` 引用總是指向某個 `Class` 物件，而 `Class` 物件可以用於產生類的實例，並且包含可作用於這些實例的所有方法程式碼。它還包含該類的 `static` 成員，因此 `Class` 引用表明了它所指向物件的確切類型，而該物件便是 `Class` 類的一個物件。

<!-- > 譯者的理解： `Class` 物件是 `Class` 類產生的物件，而再往深一點說，`Class` 類的 `Class` 物件（`Class.class`）也是其本類產生的物件。即一切皆物件，類也是一種物件。 -->

但是，Java 設計者看準機會，將它的類型變得更具體了一些。Java 引入泛型語法之後，我們可以使用泛型對 `Class` 引用所指向的 `Class` 物件的類型進行限定。在下面的實例中，兩種語法都是正確的：

```java
// typeinfo/GenericClassReferences.java

public class GenericClassReferences {
    public static void main(String[] args) {
        Class intClass = int.class;
        Class<Integer> genericIntClass = int.class;
        genericIntClass = Integer.class; // 同一個東西
        intClass = double.class;
        // genericIntClass = double.class; // 非法
    }
}
```

普通的類引用不會產生警告訊息。你可以看到，普通的類引用可以重新賦值指向任何其他的 `Class` 物件，但是使用泛型限定的類引用只能指向其聲明的類型。透過使用泛型語法，我們可以讓編譯器強制執行額外的類型檢查。

那如果我們希望稍微放鬆一些限制，應該怎麼辦呢？乍一看，下面的操作好像是可以的：

```java
Class<Number> geenericNumberClass = int.class;
```

這看起來似乎是起作用的，因為 `Integer` 繼承自 `Number`。但事實卻是不行，因為 `Integer` 的 `Class` 物件並不是 `Number`的 `Class` 物件的子類（這看起來可能有點詭異，我們將在[泛型](./20-Generics)這一章詳細討論）。

為了在使用 `Class` 引用時放鬆限制，我們使用了萬用字元，它是 Java 泛型中的一部分。萬用字元就是 `?`，表示“任何事物”。因此，我們可以在上例的普通 `Class` 引用中添加萬用字元，並產生相同的結果：

```java
// typeinfo/WildcardClassReferences.java

public class WildcardClassReferences {
    public static void main(String[] args) {
        Class<?> intClass = int.class;
        intClass = double.class;
    }
}
```

使用 `Class<?>` 比單純使用 `Class` 要好，雖然它們是等價的，並且單純使用 `Class` 不會產生編譯器警告訊息。使用 `Class<?>` 的好處是它表示你並非是碰巧或者由於疏忽才使用了一個非具體的類引用，而是特地為之。

為了建立一個限定指向某種類型或其子類的 `Class` 引用，我們需要將萬用字元與 `extends` 關鍵字配合使用，建立一個範圍限定。這與僅僅聲明 `Class<Number>` 不同，現在做如下聲明：

```java
// typeinfo/BoundedClassReferences.java

public class BoundedClassReferences {
    public static void main(String[] args) {
        Class<? extends Number> bounded = int.class;
        bounded = double.class;
        bounded = Number.class;
        // Or anything else derived from Number.
    }
}
```

向 `Class` 引用添加泛型語法的原因只是為了提供編譯期類型檢查，因此如果你操作有誤，稍後就會發現這點。使用普通的 `Class` 引用你要確保自己不會犯錯，因為一旦你犯了錯誤，就要等到執行時才能發現它，很不方便。

下面的範例使用了泛型語法，它儲存了一個類引用，稍後又用 `newInstance()` 方法產生類的物件：

```java
// typeinfo/DynamicSupplier.java
import java.util.function.*;
import java.util.stream.*;

class CountedInteger {
    private static long counter;
    private final long id = counter++;
    @Override
    public String toString() { return Long.toString(id); }
}

public class DynamicSupplier<T> implements Supplier<T> {
    private Class<T> type;
    public DynamicSupplier(Class<T> type) {
        this.type = type;
    }
    public T get() {
        try {
            return type.newInstance();
        } catch(InstantiationException |
                        IllegalAccessException e) {
            throw new RuntimeException(e);
        }
    }
    public static void main(String[] args) {
        Stream.generate(
            new DynamicSupplier<>(CountedInteger.class))
            .skip(10)
            .limit(5)
            .forEach(System.out::println);
    }
}
```

輸出結果:

```
10
11
12
13
14
```

注意，這個類必須假設與它一起工作的任何類型都有一個無參構造器，否則執行時會拋出異常。編譯期對該程式不會產生任何警告訊息。

當你將泛型語法用於 `Class` 物件時，`newInstance()` 將返回該物件的確切類型，而不僅僅只是在 `ToyTest.java` 中看到的基類 `Object`。然而，這在某種程度上有些受限：

```java
// typeinfo/toys/GenericToyTest.java
// 測試 Class 類
// {java typeinfo.toys.GenericToyTest}
package typeinfo.toys;

public class GenericToyTest {
    public static void
    main(String[] args) throws Exception {
        Class<FancyToy> ftClass = FancyToy.class;
        // Produces exact type:
        FancyToy fancyToy = ftClass.newInstance();
        Class<? super FancyToy> up =
            ftClass.getSuperclass();
        // This won't compile:
        // Class<Toy> up2 = ftClass.getSuperclass();
        // Only produces Object:
        Object obj = up.newInstance();
    }
}
```

如果你手頭的是超類，那編譯器將只允許你聲明超類引用為“某個類，它是 `FancyToy` 的超類”，就像在表達式 `Class<? super FancyToy>` 中所看到的那樣。而不會接收 `Class<Toy>` 這樣的聲明。這看起來顯得有些怪，因為 `getSuperClass()` 方法返回的是基類（不是介面），並且編譯器在編譯期就知道它是什麼類型了（在本例中就是 `Toy.class`），而不僅僅只是"某個類"。不管怎樣，正是由於這種含糊性，`up.newInstance` 的返回值不是精確類型，而只是 `Object`。

### `cast()` 方法

Java 中還有用於 `Class` 引用的轉型語法，即 `cast()` 方法：

```java
// typeinfo/ClassCasts.java

class Building {}
class House extends Building {}

public class ClassCasts {
    public static void main(String[] args) {
        Building b = new House();
        Class<House> houseType = House.class;
        House h = houseType.cast(b);
        h = (House)b; // ... 或者這樣做.
    }
}
```

`cast()` 方法接受參數物件，並將其類型轉換為 `Class` 引用的類型。但是，如果觀察上面的程式碼，你就會發現，與實現了相同功能的 `main` 方法中最後一行相比，這種轉型好像做了很多額外的工作。

`cast()` 在無法使用普通類型轉換的情況下會顯得非常有用，在你編寫泛型程式碼（你將在[泛型](./20-Generics)這一章學習到）時，如果你儲存了 `Class` 引用，並希望以後透過這個引用來執行轉型，你就需要用到 `cast()`。但事實卻是這種情況非常少見，我發現整個 Java 類庫中，只有一處使用了 `cast()`（在 `com.sun.mirror.util.DeclarationFilter` 中）。

Java 類庫中另一個沒有任何用處的特性就是 `Class.asSubclass()`，該方法允許你將一個 `Class` 物件轉型為更加具體的類型。

## 類型轉換檢測

直到現在，我們已知的 RTTI 類型包括：

1.  傳統的類型轉換，如 “`(Shape)`”，由 RTTI 確保轉換的正確性，如果執行了一個錯誤的類型轉換，就會拋出一個 `ClassCastException` 異常。

2.  代表物件類型的 `Class` 物件. 透過查詢 `Class` 物件可以獲取執行時所需的訊息.

在 C++ 中，經典的類型轉換 “`(Shape)`” 並不使用 RTTI。它只是簡單地告訴編譯器將這個物件作為新的類型對待. 而 Java 會進行類型檢查，這種類型轉換一般被稱作“類型安全的向下轉型”。之所以稱作“向下轉型”，是因為傳統上類繼承圖是這麼畫的。將 `Circle` 轉換為 `Shape` 是一次向上轉型, 將 `Shape` 轉換為 `Circle` 是一次向下轉型。但是, 因為我們知道 `Circle` 肯定是一個 `Shape`，所以編譯器允許我們自由地做向上轉型的賦值操作，且不需要任何顯式的轉型操作。當你給編譯器一個 `Shape` 的時候，編譯器並不知道它到底是什麼類型的 `Shape`——它可能是 `Shape`，也可能是 `Shape` 的子類型，例如 `Circle`、`Square`、`Triangle` 或某種其他的類型。在編譯期，編譯器只能知道它是 `Shape`。因此，你需要使用顯式地進行類型轉換，以告知編譯器你想轉換的特定類型，否則編譯器就不允許你執行向下轉型賦值。 （編譯器將會檢查向下轉型是否合理，因此它不允許向下轉型到實際不是待轉型類型的子類類型上）。

RTTI 在 Java 中還有第三種形式，那就是關鍵字 `instanceof`。它返回一個布林值，告訴我們物件是不是某個特定類型的實例，可以用提問的方式使用它，就像這個樣子：

```java
if(x instanceof Dog)
    ((Dog)x).bark();
```

在將 `x` 的類型轉換為 `Dog` 之前，`if` 語句會先檢查 `x` 是否是 `Dog` 類型的物件。進行向下轉型前，如果沒有其他訊息可以告訴你這個物件是什麼類型，那麼使用 `instanceof` 是非常重要的，否則會得到一個 `ClassCastException` 異常。

一般，可能想要尋找某種類型（比如要找三角形，並填充為紫色），這時可以輕鬆地使用 `instanceof` 來度量所有物件。舉個例子，假如你有一個類的繼承體系，描述了 `Pet`（以及它們的主人，在後面一個例子中會用到這個特性）。在這個繼承體系中的每個 `Individual` 都有一個 `id` 和一個可選的名字。儘管下面的類都繼承自 `Individual`，但是 `Individual` 類複雜性較高，因此其程式碼將放在[附錄：容器](./Appendix-Collection-Topics)中進行解釋說明。正如你所看到的，此處並不需要去了解 `Individual` 的程式碼——你只需了解你可以建立其具名或不具名的物件，並且每個 `Individual` 都有一個 `id()` 方法，如果你沒有為 `Individual` 提供名字，`toString()` 方法只產生類型名。

下面是繼承自 `Individual` 的類的繼承體系：

```java
// typeinfo/pets/Person.java
package typeinfo.pets;

public class Person extends Individual {
    public Person(String name) { super(name); }
}
```

```java
// typeinfo/pets/Pet.java
package typeinfo.pets;

public class Pet extends Individual {
    public Pet(String name) { super(name); }
    public Pet() { super(); }
}
```

```java
// typeinfo/pets/Dog.java
package typeinfo.pets;

public class Dog extends Pet {
    public Dog(String name) { super(name); }
    public Dog() { super(); }
}
```

```java
// typeinfo/pets/Mutt.java
package typeinfo.pets;

public class Mutt extends Dog {
    public Mutt(String name) { super(name); }
    public Mutt() { super(); }
}
```


```java
// typeinfo/pets/Pug.java
package typeinfo.pets;

public class Pug extends Dog {
    public Pug(String name) { super(name); }
    public Pug() { super(); }
}
```

```java
// typeinfo/pets/Cat.java
package typeinfo.pets;

public class Cat extends Pet {
    public Cat(String name) { super(name); }
    public Cat() { super(); }
}
```

```java
// typeinfo/pets/EgyptianMau.java
package typeinfo.pets;

public class EgyptianMau extends Cat {
    public EgyptianMau(String name) { super(name); }
    public EgyptianMau() { super(); }
}
```

```java
// typeinfo/pets/Manx.java
package typeinfo.pets;

public class Manx extends Cat {
    public Manx(String name) { super(name); }
    public Manx() { super(); }
}
```

```java
// typeinfo/pets/Cymric.java
package typeinfo.pets;

public class Cymric extends Manx {
    public Cymric(String name) { super(name); }
    public Cymric() { super(); }
}
```

```java
// typeinfo/pets/Rodent.java
package typeinfo.pets;

public class Rodent extends Pet {
    public Rodent(String name) { super(name); }
    public Rodent() { super(); }
}
```

```java
// typeinfo/pets/Rat.java
package typeinfo.pets;

public class Rat extends Rodent {
    public Rat(String name) { super(name); }
    public Rat() { super(); }
}
```

```java
// typeinfo/pets/Mouse.java
package typeinfo.pets;

public class Mouse extends Rodent {
    public Mouse(String name) { super(name); }
    public Mouse() { super(); }
}
```

```java
// typeinfo/pets/Hamster.java
package typeinfo.pets;

public class Hamster extends Rodent {
    public Hamster(String name) { super(name); }
    public Hamster() { super(); }
}
```

我們必須顯式地為每一個子類編寫無參構造器。因為我們有一個帶一個參數的構造器，所以編譯器不會自動地為我們加上無參構造器。

接下來，我們需要一個類，它可以隨機地建立不同類型的寵物，同時，它還可以建立寵物陣列和持有寵物的 `List`。為了使這個類更加普遍適用，我們將其定義為抽象類：

```java
// typeinfo/pets/PetCreator.java
// Creates random sequences of Pets
package typeinfo.pets;
import java.util.*;
import java.util.function.*;

public abstract class PetCreator implements Supplier<Pet> {
    private Random rand = new Random(47);

    // The List of the different types of Pet to create:
    public abstract List<Class<? extends Pet>> types();

    public Pet get() { // Create one random Pet
        int n = rand.nextInt(types().size());
        try {
            return types().get(n).newInstance();
        } catch (InstantiationException |
                IllegalAccessException e) {
            throw new RuntimeException(e);
        }
    }
}
```

抽象的 `types()` 方法需要子類來實現，以此來獲取 `Class` 物件構成的 `List`（這是模板方法設計模式的一種變體）。注意，其中類的類型被定義為“任何從 `Pet` 匯出的類型”，因此 `newInstance()` 不需要轉型就可以產生 `Pet`。`get()` 隨機的選取出一個 `Class` 物件，然後可以透過 `Class.newInstance()` 來生成該類的新實例。

在呼叫 `newInstance()` 時，可能會出現兩種異常。在緊跟 `try` 語句塊後面的 `catch` 子句中可以看到對它們的處理。異常的名字再次成為了一種對錯誤類型相對比較有用的解釋（`IllegalAccessException` 違反了 Java 安全機制，在本例中，表示預設構造器為 `private` 的情況）。

當你建立 `PetCreator` 的子類時，你需要為 `get()` 方法提供 `Pet` 類型的 `List`。`types()` 方法會簡單地返回一個靜態 `List` 的引用。下面是使用 `forName()` 的一個具體實現：

```java
// typeinfo/pets/ForNameCreator.java
package typeinfo.pets;
import java.util.*;

public class ForNameCreator extends PetCreator {
    private static List<Class<? extends Pet>> types =
            new ArrayList<>();
    // 需要隨機生成的類型名:
    private static String[] typeNames = {
            "typeinfo.pets.Mutt",
            "typeinfo.pets.Pug",
            "typeinfo.pets.EgyptianMau",
            "typeinfo.pets.Manx",
            "typeinfo.pets.Cymric",
            "typeinfo.pets.Rat",
            "typeinfo.pets.Mouse",
            "typeinfo.pets.Hamster"
    };

    @SuppressWarnings("unchecked")
    private static void loader() {
        try {
            for (String name : typeNames)
                types.add(
                        (Class<? extends Pet>) Class.forName(name));
        } catch (ClassNotFoundException e) {
            throw new RuntimeException(e);
        }
    }

    static {
        loader();
    }

    @Override
    public List<Class<? extends Pet>> types() {
        return types;
    }
}
```

`loader()` 方法使用 `Class.forName()` 建立了 `Class` 物件的 `List`。這可能會導致 `ClassNotFoundException` 異常，因為你傳入的是一個 `String` 類型的參數，它不能在編譯期間被確認是否合理。由於 `Pet` 相關的文件在 `typeinfo` 包裡面，所以使用它們的時候需要填寫完整的包名。

為了使得 `List` 裝入的是具體的 `Class` 物件，類型轉換是必須的，它會產生一個編譯時警告。`loader()` 方法是分開編寫的，然後它被放入到一個靜態程式碼塊裡，因為 `@SuppressWarning` 註解不能夠直接放置在靜態程式碼塊之上。

為了對 `Pet` 進行計數，我們需要一個能跟蹤不同類型的 `Pet` 的工具。`Map` 是這個需求的首選，我們將 `Pet` 類型名作為鍵，將儲存 `Pet` 數量的 `Integer` 作為值。透過這種方式，你就可以詢問：“有多少個 `Hamster` 物件？”我們可以使用 `instanceof` 來對 `Pet` 進行計數：

```java
// typeinfo/PetCount.java
// 使用 instanceof
import typeinfo.pets.*;
import java.util.*;

public class PetCount {
    static class Counter extends HashMap<String, Integer> {
        public void count(String type) {
            Integer quantity = get(type);
            if (quantity == null)
                put(type, 1);
            else
                put(type, quantity + 1);
        }
    }

    public static void
    countPets(PetCreator creator) {
        Counter counter = new Counter();
        for (Pet pet : Pets.array(20)) {
            // List each individual pet:
            System.out.print(
                    pet.getClass().getSimpleName() + " ");
            if (pet instanceof Pet)
                counter.count("Pet");
            if (pet instanceof Dog)
                counter.count("Dog");
            if (pet instanceof Mutt)
                counter.count("Mutt");
            if (pet instanceof Pug)
                counter.count("Pug");
            if (pet instanceof Cat)
                counter.count("Cat");
            if (pet instanceof EgyptianMau)
                counter.count("EgyptianMau");
            if (pet instanceof Manx)
                counter.count("Manx");
            if (pet instanceof Cymric)
                counter.count("Cymric");
            if (pet instanceof Rodent)
                counter.count("Rodent");
            if (pet instanceof Rat)
                counter.count("Rat");
            if (pet instanceof Mouse)
                counter.count("Mouse");
            if (pet instanceof Hamster)
                counter.count("Hamster");
        }
        // Show the counts:
        System.out.println();
        System.out.println(counter);
    }

    public static void main(String[] args) {
        countPets(new ForNameCreator());
    }
}
```

輸出結果：

```
Rat Manx Cymric Mutt Pug Cymric Pug Manx Cymric Rat
EgyptianMau Hamster EgyptianMau Mutt Mutt Cymric Mouse
Pug Mouse Cymric
{EgyptianMau=2, Pug=3, Rat=2, Cymric=5, Mouse=2, Cat=9,
Manx=7, Rodent=5, Mutt=3, Dog=6, Pet=20, Hamster=1}
```

在 `countPets()` 中，一個簡短的靜態方法 `Pets.array()` 生產出了一個隨機動物的集合。每個 `Pet` 都被 `instanceof` 檢測到並計算了一遍。

`instanceof` 有一個嚴格的限制：只可以將它與命名類型進行比較，而不能與 `Class` 物件作比較。在前面的例子中，你可能會覺得寫出一大堆 `instanceof` 表達式很乏味，事實也是如此。但是，也沒有辦法讓 `instanceof` 聰明起來，讓它能夠自動地建立一個 `Class` 物件的陣列，然後將目標與這個陣列中的物件逐一進行比較（稍後會看到一種替代方案）。其實這並不是那麼大的限制，如果你在程式中寫了大量的 `instanceof`，那就說明你的設計可能存在瑕疵。

### 使用類字面量

如果我們使用類字面量重新實現 `PetCreator` 類的話，其結果在很多方面都會更清晰：

```java
// typeinfo/pets/LiteralPetCreator.java
// 使用類字面量
// {java typeinfo.pets.LiteralPetCreator}
package typeinfo.pets;
import java.util.*;

public class LiteralPetCreator extends PetCreator {
    // try 程式碼塊不再需要
    @SuppressWarnings("unchecked")
    public static final List<Class<? extends Pet>> ALL_TYPES =
            Collections.unmodifiableList(Arrays.asList(
                    Pet.class, Dog.class, Cat.class, Rodent.class,
                    Mutt.class, Pug.class, EgyptianMau.class,
                    Manx.class, Cymric.class, Rat.class,
                    Mouse.class, Hamster.class));
    // 用於隨機建立的類型:
    private static final List<Class<? extends Pet>> TYPES =
            ALL_TYPES.subList(ALL_TYPES.indexOf(Mutt.class),
                    ALL_TYPES.size());

    @Override
    public List<Class<? extends Pet>> types() {
        return TYPES;
    }

    public static void main(String[] args) {
        System.out.println(TYPES);
    }
}
```

輸出結果：

```
[class typeinfo.pets.Mutt, class typeinfo.pets.Pug,
class typeinfo.pets.EgyptianMau, class
typeinfo.pets.Manx, class typeinfo.pets.Cymric, class
typeinfo.pets.Rat, class typeinfo.pets.Mouse, class
typeinfo.pets.Hamster]
```


在即將到來的 `PetCount3.java` 範例中，我們用所有 `Pet` 類型預先載入一個 `Map`（不僅僅是隨機生成的），因此 `ALL_TYPES` 類型的列表是必要的。`types` 列表是 `ALL_TYPES` 類型（使用 `List.subList()` 建立）的一部分，它包含精確的寵物類型，因此用於隨機生成 `Pet`。

這次，`types` 的建立沒有被 `try` 塊包圍，因為它是在編譯時計算的，因此不會引發任何異常，不像 `Class.forName()`。

我們現在在 `typeinfo.pets` 庫中有兩個 `PetCreator` 的實現。為了提供第二個作為預設實現，我們可以建立一個使用 `LiteralPetCreator` 的 *外觀模式*：

```java
// typeinfo/pets/Pets.java
// Facade to produce a default PetCreator
package typeinfo.pets;

import java.util.*;
import java.util.stream.*;

public class Pets {
    public static final PetCreator CREATOR = new LiteralPetCreator();

    public static Pet get() {
        return CREATOR.get();
    }

    public static Pet[] array(int size) {
        Pet[] result = new Pet[size];
        for (int i = 0; i < size; i++)
            result[i] = CREATOR.get();
        return result;
    }

    public static List<Pet> list(int size) {
        List<Pet> result = new ArrayList<>();
        Collections.addAll(result, array(size));
        return result;
    }

    public static Stream<Pet> stream() {
        return Stream.generate(CREATOR);
    }
}
```

這還提供了對 `get()`、`array()` 和 `list()` 的間接呼叫，以及生成 `Stream<Pet>` 的新方法。

因為 `PetCount.countPets()` 採用了 `PetCreator` 參數，所以我們可以很容易地測試 `LiteralPetCreator`（透過上面的外觀模式）：

```java
// typeinfo/PetCount2.java
import typeinfo.pets.*;

public class PetCount2 {
    public static void main(String[] args) {
        PetCount.countPets(Pets.CREATOR);
    }
}
```

輸出結果：

```
Rat Manx Cymric Mutt Pug Cymric Pug Manx Cymric Rat
EgyptianMau Hamster EgyptianMau Mutt Mutt Cymric Mouse
Pug Mouse Cymric
{EgyptianMau=2, Pug=3, Rat=2, Cymric=5, Mouse=2, Cat=9,
Manx=7, Rodent=5, Mutt=3, Dog=6, Pet=20, Hamster=1}
```

輸出與 `PetCount.java` 的輸出相同。

### 一個動態 `instanceof` 函數

`Class.isInstance()` 方法提供了一種動態測試物件類型的方法。因此，所有這些繁瑣的 `instanceof` 語句都可以從 `PetCount.java` 中刪除：

```java
// typeinfo/PetCount3.java
// 使用 isInstance() 方法

import java.util.*;
import java.util.stream.*;

import onjava.*;
import typeinfo.pets.*;

public class PetCount3 {
    static class Counter extends
            LinkedHashMap<Class<? extends Pet>, Integer> {
        Counter() {
            super(LiteralPetCreator.ALL_TYPES.stream()
                    .map(lpc -> Pair.make(lpc, 0))
                    .collect(
                            Collectors.toMap(Pair::key, Pair::value)));
        }

        public void count(Pet pet) {
            // Class.isInstance() 取代 instanceof:
            entrySet().stream()
                    .filter(pair -> pair.getKey().isInstance(pet))
                    .forEach(pair ->
                            put(pair.getKey(), pair.getValue() + 1));
        }

        @Override
        public String toString() {
            String result = entrySet().stream()
                    .map(pair -> String.format("%s=%s",
                            pair.getKey().getSimpleName(),
                            pair.getValue()))
                    .collect(Collectors.joining(", "));
            return "{" + result + "}";
        }
    }

    public static void main(String[] args) {
        Counter petCount = new Counter();
        Pets.stream()
                .limit(20)
                .peek(petCount::count)
                .forEach(p -> System.out.print(
                        p.getClass().getSimpleName() + " "));
        System.out.println("n" + petCount);
    }
}
```

輸出結果：

```
Rat Manx Cymric Mutt Pug Cymric Pug Manx Cymric Rat
EgyptianMau Hamster EgyptianMau Mutt Mutt Cymric Mouse
Pug Mouse Cymric
{Rat=2, Pug=3, Mutt=3, Mouse=2, Cat=9, Dog=6, Cymric=5,
EgyptianMau=2, Rodent=5, Hamster=1, Manx=7, Pet=20}
```

為了計算所有不同類型的 `Pet`，`Counter Map` 預先載入了來自 `LiteralPetCreator.ALL_TYPES` 的類型。如果不預先載入 `Map`，將只計數隨機生成的類型，而不是像 `Pet` 和 `Cat` 這樣的基本類型。

`isInstance()` 方法消除了對 `instanceof` 表達式的需要。此外，這意味著你可以透過更改 `LiteralPetCreator.types` 陣列來添加新類型的 `Pet`；程式的其餘部分不需要修改（就像使用 `instanceof` 表達式時那樣）。

`toString()` 方法被重載，以便更容易讀取輸出，該輸出仍與列印 `Map` 時看到的典型輸出匹配。

### 遞迴計數

`PetCount3.Counter` 中的 `Map` 預先載入了所有不同的 `Pet` 類。我們可以使用 `Class.isAssignableFrom()` 而不是預載入 `Map` ，並建立一個不限於計數 `Pet` 的通用工具：

```java
// onjava/TypeCounter.java
// 計算類型家族的實例數
package onjava;
import java.util.*;
import java.util.stream.*;

public class TypeCounter extends HashMap<Class<?>, Integer> {
    private Class<?> baseType;

    public TypeCounter(Class<?> baseType) {
        this.baseType = baseType;
    }

    public void count(Object obj) {
        Class<?> type = obj.getClass();
        if(!baseType.isAssignableFrom(type))
              throw new RuntimeException(
                obj + " incorrect type: " + type +
                ", should be type or subtype of " + baseType);
        countClass(type);
    }

    private void countClass(Class<?> type) {
        Integer quantity = get(type);
        put(type, quantity == null ? 1 : quantity + 1);
        Class<?> superClass = type.getSuperclass();
        if(superClass != null &&
               baseType.isAssignableFrom(superClass))
              countClass(superClass);
    }

    @Override
    public String toString() {
        String result = entrySet().stream()
              .map(pair -> String.format("%s=%s",
                pair.getKey().getSimpleName(),
                pair.getValue()))
              .collect(Collectors.joining(", "));
        return "{" + result + "}";
    }
}
```

`count()` 方法獲取其參數的 `Class`，並使用 `isAssignableFrom()` 進行執行時檢查，以驗證傳遞的物件實際上屬於感興趣的層次結構。`countClass()` 首先計算類的確切類型。然後，如果 `baseType` 可以從超類賦值，則在超類上遞迴呼叫 `countClass()`。

```java
// typeinfo/PetCount4.java
import typeinfo.pets.*;
import onjava.*;

public class PetCount4 {
    public static void main(String[] args) {
        TypeCounter counter = new TypeCounter(Pet.class);
        Pets.stream()
              .limit(20)
              .peek(counter::count)
              .forEach(p -> System.out.print(
                p.getClass().getSimpleName() + " "));
        System.out.println("n" + counter);
  }
}
```

輸出結果：

```
Rat Manx Cymric Mutt Pug Cymric Pug Manx Cymric Rat
EgyptianMau Hamster EgyptianMau Mutt Mutt Cymric Mouse
Pug Mouse Cymric
{Dog=6, Manx=7, Cat=9, Rodent=5, Hamster=1, Rat=2,
Pug=3, Mutt=3, Cymric=5, EgyptianMau=2, Pet=20,
Mouse=2}
```

輸出表明兩個基類型以及精確類型都被計數了。

<!-- Registered Factories -->

## 註冊工廠

從 `Pet` 層次結構生成物件的問題是，每當向層次結構中添加一種新類型的 `Pet` 時，必須記住將其添加到 `LiteralPetCreator.java` 的條目中。在一個定期添加更多類的系統中，這可能會成為問題。

你可能會考慮向每個子類添加靜態初始值設定項，因此初始值設定項會將其類添加到某個列表中。不幸的是，靜態初始值設定項僅在首次載入類時呼叫，因此存在雞和蛋的問題：生成器的列表中沒有類，因此它無法建立該類的物件，因此類不會被載入並放入列表中。

基本上，你必須自己手工建立列表（除非你編寫了一個工具來搜尋和分析原始碼，然後建立和編譯列表）。所以你能做的最好的事情就是把列表集中放在一個明顯的地方。層次結構的基類可能是最好的地方。

我們在這裡所做的另一個更改是使用*工廠方法*設計模式將物件的建立推遲到類本身。工廠方法可以以多態方式呼叫，並為你建立適當類型的物件。事實證明，`java.util.function.Supplier` 用 `T get()` 描述了原型工廠方法。協變返回類型允許 `get()` 為 `Supplier` 的每個子類實現返回不同的類型。

在本例中，基類 `Part` 包含一個工廠物件的靜態列表，列表成員類型為 `Supplier<Part>`。對於應該由 `get()` 方法生成的類型的工廠，透過將它們添加到 `prototypes` 列表向基類“註冊”。奇怪的是，這些工廠本身就是物件的實例。此列表中的每個物件都是用於建立其他物件的*原型*：

```java
// typeinfo/RegisteredFactories.java
// 註冊工廠到基礎類
import java.util.*;
import java.util.function.*;
import java.util.stream.*;

class Part implements Supplier<Part> {
    @Override
    public String toString() {
        return getClass().getSimpleName();
    }

    static List<Supplier<? extends Part>> prototypes =
        Arrays.asList(
          new FuelFilter(),
          new AirFilter(),
          new CabinAirFilter(),
          new OilFilter(),
          new FanBelt(),
          new PowerSteeringBelt(),
          new GeneratorBelt()
        );

    private static Random rand = new Random(47);
    public Part get() {
        int n = rand.nextInt(prototypes.size());
        return prototypes.get(n).get();
    }
}

class Filter extends Part {}

class FuelFilter extends Filter {
    @Override
    public FuelFilter get() {
        return new FuelFilter();
    }
}

class AirFilter extends Filter {
    @Override
    public AirFilter get() {
        return new AirFilter();
    }
}

class CabinAirFilter extends Filter {
    @Override
    public CabinAirFilter get() {
        return new CabinAirFilter();
    }
}

class OilFilter extends Filter {
    @Override
    public OilFilter get() {
        return new OilFilter();
    }
}

class Belt extends Part {}

class FanBelt extends Belt {
    @Override
    public FanBelt get() {
        return new FanBelt();
    }
}

class GeneratorBelt extends Belt {
    @Override
    public GeneratorBelt get() {
        return new GeneratorBelt();
    }
}

class PowerSteeringBelt extends Belt {
    @Override
    public PowerSteeringBelt get() {
        return new PowerSteeringBelt();
    }
}

public class RegisteredFactories {
    public static void main(String[] args) {
        Stream.generate(new Part())
              .limit(10)
              .forEach(System.out::println);
    }
}
```

輸出結果：

```
GeneratorBelt
CabinAirFilter
GeneratorBelt
AirFilter
PowerSteeringBelt
CabinAirFilter
FuelFilter
PowerSteeringBelt
PowerSteeringBelt
FuelFilter
```

並非層次結構中的所有類都應實例化；這裡的 `Filter` 和 `Belt` 只是分類器，這樣你就不會建立任何一個類的實例，而是只建立它們的子類（請注意，如果嘗試這樣做，你將獲得 `Part` 基類的行為）。

因為 `Part implements Supplier<Part>`，`Part` 透過其 `get()` 方法供應其他 `Part`。如果為基類 `Part` 呼叫 `get()`（或者如果 `generate()` 呼叫 `get()`），它將建立隨機特定的 `Part` 子類型，每個子類型最終都從 `Part` 繼承，並重寫相應的 `get()` 以生成它們中的一個。

<!-- Instanceof vs. Class Equivalence -->

## 類的等價比較

當你查詢類型訊息時，需要注意：instanceof 的形式(即 `instanceof` 或 `isInstance()` ，這兩者產生的結果相同) 和 與 Class 物件直接比較 這兩者間存在重要區別。下面的例子展示了這種區別：

```java
// typeinfo/FamilyVsExactType.java
// instanceof 與 class 的差別
// {java typeinfo.FamilyVsExactType}
package typeinfo;

class Base {}
class Derived extends Base {}

public class FamilyVsExactType {
    static void test(Object x) {
        System.out.println(
              "Testing x of type " + x.getClass());
        System.out.println(
              "x instanceof Base " + (x instanceof Base));
        System.out.println(
              "x instanceof Derived " + (x instanceof Derived));
        System.out.println(
              "Base.isInstance(x) " + Base.class.isInstance(x));
        System.out.println(
              "Derived.isInstance(x) " +
              Derived.class.isInstance(x));
        System.out.println(
              "x.getClass() == Base.class " +
              (x.getClass() == Base.class));
        System.out.println(
              "x.getClass() == Derived.class " +
              (x.getClass() == Derived.class));
        System.out.println(
              "x.getClass().equals(Base.class)) "+
              (x.getClass().equals(Base.class)));
        System.out.println(
              "x.getClass().equals(Derived.class)) " +
              (x.getClass().equals(Derived.class)));
    }

    public static void main(String[] args) {
        test(new Base());
        test(new Derived());
    }
}
```

輸出結果：

```
Testing x of type class typeinfo.Base
x instanceof Base true
x instanceof Derived false
Base.isInstance(x) true
Derived.isInstance(x) false
x.getClass() == Base.class true
x.getClass() == Derived.class false
x.getClass().equals(Base.class)) true
x.getClass().equals(Derived.class)) false
Testing x of type class typeinfo.Derived
x instanceof Base true
x instanceof Derived true
Base.isInstance(x) true
Derived.isInstance(x) true
x.getClass() == Base.class false
x.getClass() == Derived.class true
x.getClass().equals(Base.class)) false
x.getClass().equals(Derived.class)) true
```

`test()` 方法使用兩種形式的 `instanceof` 對其參數執行類型檢查。然後，它獲取 `Class` 引用，並使用 `==` 和 `equals()` 測試 `Class` 物件的相等性。令人放心的是，`instanceof` 和 `isInstance()` 產生的結果相同， `equals()` 和 `==` 產生的結果也相同。但測試本身得出了不同的結論。與類型的概念一致，`instanceof` 說的是“你是這個類，還是從這個類衍生的類？”。而如果使用 `==` 比較實際的`Class` 物件，則與繼承無關 —— 它要嘛是確切的類型，要嘛不是。

<!-- Reflection: Runtime Class Information -->
## 反射：執行時類訊息

如果你不知道物件的確切類型，RTTI 會告訴你。但是，有一個限制：必須在編譯時知道類型，才能使用 RTTI 檢測它，並對訊息做一些有用的事情。換句話說，編譯器必須知道你使用的所有類。

起初，這看起來並沒有那麼大的限制，但是假設你引用了一個不在程式空間中的物件。實際上，該物件的類在編譯時甚至對程式都不可用。也許你從磁碟文件或網路連線中獲得了大量的位元組，並被告知這些位元組代表一個類。由於這個類在編譯器為你的程式生成程式碼後很長時間才會出現，你如何使用這樣的類？

在傳統編程環境中，這是一個牽強的場景。但是，當我們進入一個更大的程式世界時，會有一些重要的情況發生。第一個是基於元件的程式，你可以在應用程式構建器*整合開發環境*中使用*快速應用程式開發*（RAD）構建項目。這是一種透過將表示元件的圖示移動到表單上來建立程式的可視化方法。然後，透過在編程時設定這些元件的一些值來配置這些元件。這種設計時配置要求任何元件都是可實例化的，它公開自己的部分，並且允許讀取和修改其屬性。此外，處理*圖形使用者介面*（GUI）事件的元件必須公開有關適當方法的訊息，以便 IDE 可以幫助程式設計師覆寫這些事件處理方法。反射提供了檢測可用方法並生成方法名稱的機制。

在執行時發現類訊息的另一個令人信服的動機是提供跨網路在遠端平台上建立和執行物件的能力。這稱為*遠端方法呼叫*（RMI），它使 Java 程式的物件分布在許多機器上。這種分布有多種原因。如果你想加速一個計算密集型的任務，你可以把它分解成小塊放到空閒的機器上。或者你可以將處理特定類型任務的程式碼（例如，多層次客戶機/伺服器體系結構中的“業務規則”）放在特定的機器上，這樣機器就成為描述這些操作的公共儲存庫，並且可以很容易地更改它以影響系統中的每個人。分布式計算還支援專門的硬體，這些硬體可能擅長於某個特定的任務——例如矩陣轉換——但對於通用編程來說不合適或過於昂貴。

類 `Class` 支援*反射*的概念， `java.lang.reflect` 庫中包含類 `Field`、`Method` 和 `Constructor`（每一個都實現了 `Member` 介面）。這些類型的物件由 JVM 在執行時建立，以表示未知類中的對應成員。然後，可以使用 `Constructor` 建立新物件，`get()` 和 `set()` 方法讀取和修改與 `Field` 物件關聯的欄位，`invoke()` 方法呼叫與 `Method` 物件關聯的方法。此外，還可以呼叫便利方法 `getFields()`、`getMethods()`、`getConstructors()` 等，以返回表示欄位、方法和建構子的物件陣列。（你可以透過在 JDK 文件中尋找類 `Class` 來了解更多訊息。）因此，匿名物件的類訊息可以在執行時完全確定，編譯時不需要知道任何訊息。

重要的是要意識到反射沒有什麼魔力。當你使用反射與未知類型的物件互動時，JVM 將查看該物件，並看到它屬於特定的類（就像普通的 RTTI）。在對其執行任何操作之前，必須載入 `Class` 物件。因此，該特定類型的 `.class` 文件必須在本機電腦上或透過網路對 JVM 仍然可用。因此，RTTI 和反射的真正區別在於，使用 RTTI 時，編譯器在編譯時會打開並檢查 `.class` 文件。換句話說，你可以用“正常”的方式呼叫一個物件的所有方法。透過反射，`.class` 文件在編譯時不可用；它由執行時環境打開並檢查。

### 類方法提取器

通常，你不會直接使用反射工具，但它們可以幫助你建立更多的動態程式碼。反射是用來支援其他 Java 特性的，例如物件序列化（參見[附錄：物件序列化](https://lingcoder.github.io/OnJava8/#/book/Appendix-Object-Serialization)）。但是，有時動態提取有關類的訊息很有用。

考慮一個類方法提取器。查看類定義的原始碼或 JDK 文件，只顯示*在該類定義中*定義或重寫的方法。但是，可能還有幾十個來自基類的可用方法。找到它們既單調又費時[^1]。幸運的是，反射提供了一種方法，可以簡單地編寫一個工具類自動地向你展示所有的介面：

```java
// typeinfo/ShowMethods.java
// 使用反射展示一個類的所有方法，甚至包括定義在基類中方法
// {java ShowMethods ShowMethods}
import java.lang.reflect.*;
import java.util.regex.*;

public class ShowMethods {
    private static String usage =
            "usage:\n" +
            "ShowMethods qualified.class.name\n" +
            "To show all methods in class or:\n" +
            "ShowMethods qualified.class.name word\n" +
            "To search for methods involving 'word'";
    private static Pattern p = Pattern.compile("\\w+\\.");

    public static void main(String[] args) {
        if (args.length < 1) {
            System.out.println(usage);
            System.exit(0);
        }
        int lines = 0;
        try {
            Class<?> c = Class.forName(args[0]);
            Method[] methods = c.getMethods();
            Constructor[] ctors = c.getConstructors();
            if (args.length == 1) {
                for (Method method : methods)
                    System.out.println(
                            p.matcher(
                                    method.toString()).replaceAll(""));
                for (Constructor ctor : ctors)
                    System.out.println(
                            p.matcher(ctor.toString()).replaceAll(""));
                lines = methods.length + ctors.length;
            } else {
                for (Method method : methods)
                    if (method.toString().contains(args[1])) {
                        System.out.println(p.matcher(
                                method.toString()).replaceAll(""));
                        lines++;
                    }
                for (Constructor ctor : ctors)
                    if (ctor.toString().contains(args[1])) {
                        System.out.println(p.matcher(
                                ctor.toString()).replaceAll(""));
                        lines++;
                    }
            }
        } catch (ClassNotFoundException e) {
            System.out.println("No such class: " + e);
        }
    }
}
```

輸出結果：

```
public static void main(String[])
public final void wait() throws InterruptedException
public final void wait(long,int) throws
InterruptedException
public final native void wait(long) throws
InterruptedException
public boolean equals(Object)
public String toString()
public native int hashCode()
public final native Class getClass()
public final native void notify()
public final native void notifyAll()
public ShowMethods()
```

`Class` 方法 `getmethods()` 和 `getconstructors()`  分別返回 `Method` 陣列和 `Constructor` 陣列。這些類中的每一個都有進一步的方法來解析它們所表示的方法的名稱、參數和返回值。但你也可以像這裡所做的那樣，使用 `toString()`，生成帶有整個方法簽名的 `String`。程式碼的其餘部分提取命令列訊息，確定特定簽名是否與目標 `String`（使用 `indexOf()`）匹配，並使用正規表示式（在 [Strings](#ch021.xhtml#strings) 一章中介紹）刪除名稱限定符。

編譯時無法知道 `Class.forName()` 生成的結果，因此所有方法簽名訊息都是在執行時提取的。如果你研究 JDK 反射文件，你將看到有足夠的支援來實際設定和對編譯時完全未知的物件進行方法呼叫（本書後面有這樣的例子）。雖然最初你可能認為你永遠都不需要這樣做，但是反射的全部價值可能會令人驚訝。

上面的輸出來自命令列：

```java
java ShowMethods ShowMethods
```

輸出包含一個 `public` 無參數建構子，即使未定義建構子。你看到的建構子是由編譯器自動合成的。如果將 `ShowMethods` 設定為非 `public` 類（即只有包級訪問權），則合成的無參數建構子將不再顯示在輸出中。自動為合成的無參數建構子授予與類相同的訪問權。

嘗試執行 `java ShowMethods java.lang.String`，並附加一個 `char`、`int`、`String` 等參數。

編程時，當你不記得某個類是否有特定的方法，並且不想在 JDK 文件中搜尋索引或類層次結構時，或者如果你不知道該類是否可以對 `Color` 物件執行任何操作時，該工具能節省不少時間。

<!-- Dynamic Proxies -->

## 動態代理

*代理*是基本的設計模式之一。一個物件封裝真實物件，代替其提供其他或不同的操作---這些操作通常涉及到與“真實”物件的通信，因此代理通常充當中間物件。這是一個簡單的範例，顯示代理的結構：

```java
// typeinfo/SimpleProxyDemo.java

interface Interface {
    void doSomething();

    void somethingElse(String arg);
}

class RealObject implements Interface {
    @Override
    public void doSomething() {
        System.out.println("doSomething");
    }

    @Override
    public void somethingElse(String arg) {
        System.out.println("somethingElse " + arg);
    }
}

class SimpleProxy implements Interface {
    private Interface proxied;

    SimpleProxy(Interface proxied) {
        this.proxied = proxied;
    }

    @Override
    public void doSomething() {
        System.out.println("SimpleProxy doSomething");
        proxied.doSomething();
    }

    @Override
    public void somethingElse(String arg) {
        System.out.println(
                "SimpleProxy somethingElse " + arg);
        proxied.somethingElse(arg);
    }
}

class SimpleProxyDemo {
    public static void consumer(Interface iface) {
        iface.doSomething();
        iface.somethingElse("bonobo");
    }

    public static void main(String[] args) {
        consumer(new RealObject());
        consumer(new SimpleProxy(new RealObject()));
    }
}
```

輸出結果：

```
doSomething
somethingElse bonobo
SimpleProxy doSomething
doSomething
SimpleProxy somethingElse bonobo
somethingElse bonobo
```

因為 `consumer()` 接受 `Interface`，所以它不知道獲得的是 `RealObject` 還是 `SimpleProxy`，因為兩者都實現了 `Interface`。
但是，在用戶端和 `RealObject` 之間插入的 `SimpleProxy` 執行操作，然後在 `RealObject` 上呼叫相同的方法。

當你希望將額外的操作與“真實物件”做分離時，代理可能會有所幫助，尤其是當你想要輕鬆地啟用額外的操作時，反之亦然（設計模式就是封裝變更---所以你必須改變一些東西以證明模式的合理性）。例如，如果你想跟蹤對 `RealObject` 中方法的呼叫，或衡量此類呼叫的開銷，該怎麼辦？你不想這部分程式碼耦合到你的程式中，而代理能使你可以很輕鬆地添加或刪除它。

Java 的*動態代理*更進一步，不僅動態建立代理物件而且動態處理對代理方法的呼叫。在動態代理上進行的所有呼叫都被重定向到單個*呼叫處理程序*，該處理程序負責發現呼叫的內容並決定如何處理。這是 `SimpleProxyDemo.java` 使用動態代理重寫的例子：

```java
// typeinfo/SimpleDynamicProxy.java

import java.lang.reflect.*;

class DynamicProxyHandler implements InvocationHandler {
    private Object proxied;

    DynamicProxyHandler(Object proxied) {
        this.proxied = proxied;
    }

    @Override
    public Object
    invoke(Object proxy, Method method, Object[] args)
            throws Throwable {
        System.out.println(
                "**** proxy: " + proxy.getClass() +
                        ", method: " + method + ", args: " + args);
        if (args != null)
            for (Object arg : args)
                System.out.println("  " + arg);
        return method.invoke(proxied, args);
    }
}

class SimpleDynamicProxy {
    public static void consumer(Interface iface) {
        iface.doSomething();
        iface.somethingElse("bonobo");
    }

    public static void main(String[] args) {
        RealObject real = new RealObject();
        consumer(real);
        // Insert a proxy and call again:
        Interface proxy = (Interface) Proxy.newProxyInstance(
                Interface.class.getClassLoader(),
                new Class[]{Interface.class},
                new DynamicProxyHandler(real));
        consumer(proxy);
    }
}
```

輸出結果：

```
doSomething
somethingElse bonobo
**** proxy: class $Proxy0, method: public abstract void
Interface.doSomething(), args: null
doSomething
**** proxy: class $Proxy0, method: public abstract void
Interface.somethingElse(java.lang.String), args:
[Ljava.lang.Object;@6bc7c054
  bonobo
somethingElse bonobo
```

可以透過呼叫靜態方法 `Proxy.newProxyInstance()` 來建立動態代理，該方法需要一個類載入器（通常可以從已載入的物件中獲取），希望代理實現的介面列表（不是類或抽象類），以及介面  `InvocationHandler` 的一個實現。動態代理會將所有呼叫重定向到呼叫處理程序，因此通常為呼叫處理程序的建構子提供對“真實”物件的引用，以便一旦執行中介任務便可以轉發請求。

`invoke()` 方法被傳遞給代理物件，以防萬一你必須區分請求的來源---但是在很多情況下都無需關心。但是，在 `invoke()` 內的代理上呼叫方法時要小心，因為介面的呼叫是透過代理重定向的。

通常執行代理操作，然後使用 `Method.invoke()` 將請求轉發給被代理物件，並攜帶必要的參數。這在一開始看起來是有限制的，好像你只能執行一般的操作。但是，可以過濾某些方法呼叫，同時傳遞其他方法呼叫：

```java
// typeinfo/SelectingMethods.java
// Looking for particular methods in a dynamic proxy

import java.lang.reflect.*;

class MethodSelector implements InvocationHandler {
    private Object proxied;

    MethodSelector(Object proxied) {
        this.proxied = proxied;
    }

    @Override
    public Object
    invoke(Object proxy, Method method, Object[] args)
            throws Throwable {
        if (method.getName().equals("interesting"))
            System.out.println(
                    "Proxy detected the interesting method");
        return method.invoke(proxied, args);
    }
}

interface SomeMethods {
    void boring1();

    void boring2();

    void interesting(String arg);

    void boring3();
}

class Implementation implements SomeMethods {
    @Override
    public void boring1() {
        System.out.println("boring1");
    }

    @Override
    public void boring2() {
        System.out.println("boring2");
    }

    @Override
    public void interesting(String arg) {
        System.out.println("interesting " + arg);
    }

    @Override
    public void boring3() {
        System.out.println("boring3");
    }
}

class SelectingMethods {
    public static void main(String[] args) {
        SomeMethods proxy =
                (SomeMethods) Proxy.newProxyInstance(
                        SomeMethods.class.getClassLoader(),
                        new Class[]{ SomeMethods.class },
                        new MethodSelector(new Implementation()));
        proxy.boring1();
        proxy.boring2();
        proxy.interesting("bonobo");
        proxy.boring3();
    }
}
```

輸出結果：

```
boring1
boring2
Proxy detected the interesting method
interesting bonobo
boring3
```

在這個範例裡，我們只是在尋找方法名，但是你也可以尋找方法簽名的其他方面，甚至可以搜尋特定的參數值。

動態代理不是你每天都會使用的工具，但是它可以很好地解決某些類型的問題。你可以在 Erich Gamma 等人的*設計模式*中了解有關*代理*和其他設計模式的更多訊息。 （Addison-Wesley，1995年），以及[設計模式](./25-Patterns.md)一章。

<!-- Using Optional -->

## Optional類

如果你使用內建的 `null` 來表示沒有物件，每次使用引用的時候就必須測試一下引用是否為 `null`，這顯得有點枯燥，而且勢必會產生相當乏味的程式碼。問題在於 `null` 沒什麼自己的行為，只會在你想用它執行任何操作的時候產生 `NullPointException`。`java.util.Optional`（首次出現是在[函數式編程](docs/book/13-Functional-Programming.md)這章）為 `null` 值提供了一個輕量級代理，`Optional` 物件可以防止你的程式碼直接拋出 `NullPointException`。

雖然 `Optional` 是 Java 8 為了支援流式編程才引入的，但其實它是一個通用的工具。為了證明這點，在本節中，我們會把它用在普通的類中。因為涉及一些執行時檢測，所以把這一小節放在了本章。

實際上，在所有地方都使用 `Optional` 是沒有意義的，有時候檢查一下是不是 `null` 也挺好的，或者有時我們可以合理地假設不會出現 `null`，甚至有時候檢查 `NullPointException` 異常也是可以接受的。`Optional` 最有用武之地的是在那些“更接近資料”的地方，在問題空間中代表實體的物件上。舉個簡單的例子，很多系統中都有 `Person` 類型，程式碼中有些情況下你可能沒有一個實際的 `Person` 物件（或者可能有，但是你還沒用關於那個人的所有訊息）。這時，在傳統方法下，你會用到一個 `null` 引用，並且在使用的時候測試它是不是 `null`。而現在，我們可以使用 `Optional`：

```java
// typeinfo/Person.java
// Using Optional with regular classes

import onjava.*;

import java.util.*;

class Person {
    public final Optional<String> first;
    public final Optional<String> last;
    public final Optional<String> address;
    // etc.
    public final Boolean empty;

    Person(String first, String last, String address) {
        this.first = Optional.ofNullable(first);
        this.last = Optional.ofNullable(last);
        this.address = Optional.ofNullable(address);
        empty = !this.first.isPresent()
                && !this.last.isPresent()
                && !this.address.isPresent();
    }

    Person(String first, String last) {
        this(first, last, null);
    }

    Person(String last) {
        this(null, last, null);
    }

    Person() {
        this(null, null, null);
    }

    @Override
    public String toString() {
        if (empty)
            return "<Empty>";
        return (first.orElse("") +
                " " + last.orElse("") +
                " " + address.orElse("")).trim();
    }

    public static void main(String[] args) {
        System.out.println(new Person());
        System.out.println(new Person("Smith"));
        System.out.println(new Person("Bob", "Smith"));
        System.out.println(new Person("Bob", "Smith",
                "11 Degree Lane, Frostbite Falls, MN"));
    }
}
```

輸出結果：

```
<Empty>
Smith
Bob Smith
Bob Smith 11 Degree Lane, Frostbite Falls, MN
```

`Person` 的設計有時候又叫“資料傳輸物件（DTO，data-transfer object）”。注意，所有欄位都是 `public` 和 `final` 的，所以沒有 `getter` 和 `setter` 方法。也就是說，`Person` 是不可變的，你只能透過構造器給它賦值，之後就只能讀而不能修改它的值（字串本身就是不可變的，因此你無法修改字串的內容，也無法給它的欄位重新賦值）。如果你想修改一個 `Person`，你只能用一個新的 `Person` 物件來取代它。`empty` 欄位在物件建立的時候被賦值，用於快速判斷這個 `Person` 物件是不是空物件。

如果想使用 `Person`，就必須使用 `Optional` 介面才能訪問它的 `String` 欄位，這樣就不會意外觸發 `NullPointException` 了。

現在假設你已經因你驚人的理念而獲得了一大筆風險投資，現在你要招兵買馬了，但是在虛位以待時，你可以將 `Person Optional` 物件放在每個 `Position` 上：

```java
// typeinfo/Position.java

import java.util.*;

class EmptyTitleException extends RuntimeException {
}

class Position {
    private String title;
    private Person person;

    Position(String jobTitle, Person employee) {
        setTitle(jobTitle);
        setPerson(employee);
    }

    Position(String jobTitle) {
        this(jobTitle, null);
    }

    public String getTitle() {
        return title;
    }

    public void setTitle(String newTitle) {
        // Throws EmptyTitleException if newTitle is null:
        title = Optional.ofNullable(newTitle)
                .orElseThrow(EmptyTitleException::new);
    }

    public Person getPerson() {
        return person;
    }

    public void setPerson(Person newPerson) {
        // Uses empty Person if newPerson is null:
        person = Optional.ofNullable(newPerson)
                .orElse(new Person());
    }

    @Override
    public String toString() {
        return "Position: " + title +
                ", Employee: " + person;
    }

    public static void main(String[] args) {
        System.out.println(new Position("CEO"));
        System.out.println(new Position("Programmer",
                new Person("Arthur", "Fonzarelli")));
        try {
            new Position(null);
        } catch (Exception e) {
            System.out.println("caught " + e);
        }
    }
}
```

輸出結果：

```
Position: CEO, Employee: <Empty>
Position: Programmer, Employee: Arthur Fonzarelli
caught EmptyTitleException
```

這裡使用 `Optional` 的方式不太一樣。請注意，`title` 和 `person` 都是普通欄位，不受 `Optional` 的保護。但是，修改這些欄位的唯一途徑是呼叫 `setTitle()` 和 `setPerson()` 方法，這兩個都借助 `Optional` 對欄位進行了嚴格的限制。

同時，我們想保證 `title` 欄位永遠不會變成 `null` 值。為此，我們可以自己在 `setTitle()` 方法裡面檢查參數 `newTitle` 的值。但其實還有更好的做法，函數式編程一大優勢就是可以讓我們重用經過驗證的功能（即便是個很小的功能），以減少自己手動編寫程式碼可能產生的一些小錯誤。所以在這裡，我們用 `ofNullable()` 把 `newTitle` 轉換一個 `Optional`（如果傳入的值為 `null`，`ofNullable()` 返回的將是 `Optional.empty()`）。緊接著我們呼叫了 `orElseThrow()` 方法，所以如果 `newTitle` 的值是 `null`，你將會得到一個異常。這裡我們並沒有把 `title` 儲存成 `Optional`，但透過應用 `Optional` 的功能，我們仍然如願以償地對這個欄位施加了約束。

`EmptyTitleException` 是一個 `RuntimeException`，因為它意味著程式存在錯誤。在這個方案裡面，你仍然可能會得到一個異常。但不同的是，在錯誤產生的那一刻（向 `setTitle()` 傳 `null` 值時）就會拋出異常，而不是發生在其它時刻，需要你透過除錯才能發現問題所在。另外，使用 `EmptyTitleException` 還有助於定位 BUG。

`Person` 欄位的限制又不太一樣：如果你把它的值設為 `null`，程式會自動把將它賦值成一個空的 `Person` 物件。先前我們也用過類似的方法把欄位轉換成 `Optional`，但這裡我們是在返回結果的時候使用 `orElse(new Person())` 插入一個空的 `Person` 物件替代了 `null`。

在 `Position` 裡面，我們沒有建立一個表示“空”的標誌位或者方法，因為 `person` 欄位的 `Person` 物件為空，就表示這個 `Position` 是個空缺位置。之後，你可能會發現你必須添加一個顯式的表示“空位”的方法，但是正如 YAGNI[^2] (You Aren't Going to Need It，你永遠不需要它)所言，在初稿時“實現盡最大可能的簡單”，直到程式在某些方面要求你為其添加一些額外的特性，而不是假設這是必要的。

請注意，雖然你清楚你使用了 `Optional`，可以免受 `NullPointerExceptions` 的困擾，但是 `Staff` 類卻對此毫不知情。

```java
// typeinfo/Staff.java

import java.util.*;

public class Staff extends ArrayList<Position> {
    public void add(String title, Person person) {
        add(new Position(title, person));
    }

    public void add(String... titles) {
        for (String title : titles)
            add(new Position(title));
    }

    public Staff(String... titles) {
        add(titles);
    }

    public Boolean positionAvailable(String title) {
        for (Position position : this)
            if (position.getTitle().equals(title) &&
                    position.getPerson().empty)
                return true;
        return false;
    }

    public void fillPosition(String title, Person hire) {
        for (Position position : this)
            if (position.getTitle().equals(title) &&
                    position.getPerson().empty) {
                position.setPerson(hire);
                return;
            }
        throw new RuntimeException(
                "Position " + title + " not available");
    }

    public static void main(String[] args) {
        Staff staff = new Staff("President", "CTO",
                "Marketing Manager", "Product Manager",
                "Project Lead", "Software Engineer",
                "Software Engineer", "Software Engineer",
                "Software Engineer", "Test Engineer",
                "Technical Writer");
        staff.fillPosition("President",
                new Person("Me", "Last", "The Top, Lonely At"));
        staff.fillPosition("Project Lead",
                new Person("Janet", "Planner", "The Burbs"));
        if (staff.positionAvailable("Software Engineer"))
            staff.fillPosition("Software Engineer",
                    new Person(
                            "Bob", "Coder", "Bright Light City"));
        System.out.println(staff);
    }
}
```

輸出結果：

```
[Position: President, Employee: Me Last The Top, Lonely
At, Position: CTO, Employee: <Empty>, Position:
Marketing Manager, Employee: <Empty>, Position: Product
Manager, Employee: <Empty>, Position: Project Lead,
Employee: Janet Planner The Burbs, Position: Software
Engineer, Employee: Bob Coder Bright Light City,
Position: Software Engineer, Employee: <Empty>,
Position: Software Engineer, Employee: <Empty>,
Position: Software Engineer, Employee: <Empty>,
Position: Test Engineer, Employee: <Empty>, Position:
Technical Writer, Employee: <Empty>]
```

注意，在有些地方你可能還是要測試引用是不是 `Optional`，這跟檢查是否為 `null` 沒什麼不同。但是在其它地方（例如本例中的 `toString()` 轉換），你就不必執行額外的測試了，而可以直接假設所有物件都是有效的。

### 標記介面

有時候使用一個**標記介面**來表示空值會更方便。標記介面裡面什麼都沒有，你只要把它的名字當做標籤來用就可以。

```java
// onjava/Null.java
package onjava;
public interface Null {}
```

如果你用介面取代具體類，那麼就可以使用 `DynamicProxy` 來自動地建立 `Null` 物件。假設我們有一個 `Robot` 介面，它定義了一個名字、一個模型和一個描述 `Robot` 行為能力的 `List<Operation>`：

```java
// typeinfo/Robot.java

import onjava.*;

import java.util.*;

public interface Robot {
    String name();

    String model();

    List<Operation> operations();

    static void test(Robot r) {
        if (r instanceof Null)
            System.out.println("[Null Robot]");
        System.out.println("Robot name: " + r.name());
        System.out.println("Robot model: " + r.model());
        for (Operation operation : r.operations()) {
            System.out.println(operation.description.get());
            operation.command.run();
        }
    }
}
```

你可以透過呼叫 `operations()` 來訪問 `Robot` 的服務。`Robot` 裡面還有一個 `static` 方法來執行測試。

`Operation` 包含一個描述和一個指令（這用到了**指令模式**）。它們被定義成函數式介面的引用，所以可以把 lambda 表達式或者方法的引用傳給 `Operation` 的構造器：

```java
// typeinfo/Operation.java

import java.util.function.*;

public class Operation {
    public final Supplier<String> description;
    public final Runnable command;

    public Operation(Supplier<String> descr, Runnable cmd) {
        description = descr;
        command = cmd;
    }
}
```

現在我們可以建立一個掃雪 `Robot`：

```java
// typeinfo/SnowRemovalRobot.java

import java.util.*;

public class SnowRemovalRobot implements Robot {
    private String name;

    public SnowRemovalRobot(String name) {
        this.name = name;
    }

    @Override
    public String name() {
        return name;
    }

    @Override
    public String model() {
        return "SnowBot Series 11";
    }

    private List<Operation> ops = Arrays.asList(
            new Operation(
                    () -> name + " can shovel snow",
                    () -> System.out.println(
                            name + " shoveling snow")),
            new Operation(
                    () -> name + " can chip ice",
                    () -> System.out.println(name + " chipping ice")),
            new Operation(
                    () -> name + " can clear the roof",
                    () -> System.out.println(
                            name + " clearing roof")));

    public List<Operation> operations() {
        return ops;
    }

    public static void main(String[] args) {
        Robot.test(new SnowRemovalRobot("Slusher"));
    }
}
```

輸出結果：

```
Robot name: Slusher
Robot model: SnowBot Series 11
Slusher can shovel snow
Slusher shoveling snow
Slusher can chip ice
Slusher chipping ice
Slusher can clear the roof
Slusher clearing roof
```

假設存在許多不同類型的 `Robot`，我們想讓每種 `Robot` 都建立一個 `Null` 物件來執行一些特殊的操作——在本例中，即提供 `Null` 物件所代表 `Robot` 的確切類型訊息。這些訊息是透過動態代理捕獲的：

```java
// typeinfo/NullRobot.java
// Using a dynamic proxy to create an Optional

import java.lang.reflect.*;
import java.util.*;
import java.util.stream.*;

import onjava.*;

class NullRobotProxyHandler
        implements InvocationHandler {
    private String nullName;
    private Robot proxied = new NRobot();

    NullRobotProxyHandler(Class<? extends Robot> type) {
        nullName = type.getSimpleName() + " NullRobot";
    }

    private class NRobot implements Null, Robot {
        @Override
        public String name() {
            return nullName;
        }

        @Override
        public String model() {
            return nullName;
        }

        @Override
        public List<Operation> operations() {
            return Collections.emptyList();
        }
    }

    @Override
    public Object
    invoke(Object proxy, Method method, Object[] args)
            throws Throwable {
        return method.invoke(proxied, args);
    }
}

public class NullRobot {
    public static Robot
    newNullRobot(Class<? extends Robot> type) {
        return (Robot) Proxy.newProxyInstance(
                NullRobot.class.getClassLoader(),
                new Class[] { Null.class, Robot.class },
                new NullRobotProxyHandler(type));
    }

    public static void main(String[] args) {
        Stream.of(
                new SnowRemovalRobot("SnowBee"),
                newNullRobot(SnowRemovalRobot.class)
        ).forEach(Robot::test);
    }
}
```

輸出結果：

```
Robot name: SnowBee
Robot model: SnowBot Series 11
SnowBee can shovel snow
SnowBee shoveling snow
SnowBee can chip ice
SnowBee chipping ice
SnowBee can clear the roof
SnowBee clearing roof
[Null Robot]
Robot name: SnowRemovalRobot NullRobot
Robot model: SnowRemovalRobot NullRobot
```

無論何時，如果你需要一個空 `Robot` 物件，只需要呼叫 `newNullRobot()`，並傳遞需要代理的 `Robot` 的類型。這個代理滿足了 `Robot` 和 `Null` 介面的需要，並提供了它所代理的類型的確切名字。

### Mock 物件和樁

**Mock 物件**和 **樁（Stub）**在邏輯上都是 `Optional` 的變體。他們都是最終程式中所使用的“實際”物件的代理。不過，Mock 物件和樁都是假扮成那些可以傳遞實際訊息的實際物件，而不是像 `Optional` 那樣把包含潛在 `null` 值的物件隱藏。

Mock 物件和樁之間的的差別在於程度不同。Mock 物件往往是輕量級的，且用於自測試。通常，為了處理各種不同的測試場景，我們會建立出很多 Mock 物件。而樁只是返回樁資料，它通常是重量級的，並且經常在多個測試中被復用。樁可以根據它們被呼叫的方式，透過配置進行修改。因此，樁是一種複雜物件，它可以做很多事情。至於 Mock 物件，如果你要做很多事，通常會建立大量又小又簡單的 Mock 物件。

<!-- Interfaces and Type -->
## 介面和類型

`interface` 關鍵字的一個重要目標就是允許程式設計師隔離元件，進而降低耦合度。使用介面可以實現這一目標，但是透過類型訊息，這種耦合性還是會傳播出去——介面並不是對解耦的一種無懈可擊的保障。比如我們先寫一個介面：

```java
// typeinfo/interfacea/A.java
package typeinfo.interfacea;

public interface A {
    void f();
}
```

然後實現這個介面，你可以看到其程式碼是怎麼從實際類型開始順藤摸瓜的：

```java
// typeinfo/InterfaceViolation.java
// Sneaking around an interface

import typeinfo.interfacea.*;

class B implements A {
    public void f() {
    }

    public void g() {
    }
}

public class InterfaceViolation {
    public static void main(String[] args) {
        A a = new B();
        a.f();
        // a.g(); // Compile error
        System.out.println(a.getClass().getName());
        if (a instanceof B) {
            B b = (B) a;
            b.g();
        }
    }
}
```

輸出結果：

```
B
```

透過使用 RTTI，我們發現 `a` 是用 `B` 實現的。透過將其轉型為 `B`，我們可以呼叫不在 `A` 中的方法。

這樣的操作完全是合情合理的，但是你也許並不想讓用戶端開發者這麼做，因為這給了他們一個機會，使得他們的程式碼與你的程式碼的耦合度超過了你的預期。也就是說，你可能認為 `interface` 關鍵字正在保護你，但其實並沒有。另外，在本例中使用 `B` 來實現 `A` 這種情況是有公開案例可查的[^3]。

一種解決方案是直接聲明，如果開發者決定使用實際的類而不是介面，他們需要自己對自己負責。這在很多情況下都是可行的，但“可能”還不夠，你或許希望能有一些更嚴格的控制方式。

最簡單的方式是讓實現類只具有包訪問權限，這樣在包外部的用戶端就看不到它了：

```java
// typeinfo/packageaccess/HiddenC.java
package typeinfo.packageaccess;

import typeinfo.interfacea.*;

class C implements A {
    @Override
    public void f() {
        System.out.println("public C.f()");
    }

    public void g() {
        System.out.println("public C.g()");
    }

    void u() {
        System.out.println("package C.u()");
    }

    protected void v() {
        System.out.println("protected C.v()");
    }

    private void w() {
        System.out.println("private C.w()");
    }
}

public class HiddenC {
    public static A makeA() {
        return new C();
    }
}
```

在這個包中唯一 `public` 的部分就是 `HiddenC`，在被呼叫時將產生 `A`介面類型的物件。這裡有趣之處在於：即使你從 `makeA()` 返回的是 `C` 類型，你在包的外部仍舊不能使用 `A` 之外的任何方法，因為你不能在包的外部命名 `C`。

現在如果你試著將其向下轉型為 `C`，則將被禁止，因為在包的外部沒有任何 `C` 類型可用：

```java
// typeinfo/HiddenImplementation.java
// Sneaking around package hiding

import typeinfo.interfacea.*;
import typeinfo.packageaccess.*;

import java.lang.reflect.*;

public class HiddenImplementation {
    public static void main(String[] args) throws Exception {
        A a = HiddenC.makeA();
        a.f();
        System.out.println(a.getClass().getName());
        // Compile error: cannot find symbol 'C':
        /* if(a instanceof C) {
            C c = (C)a;
            c.g();
        } */
        // Oops! Reflection still allows us to call g():
        callHiddenMethod(a, "g");
        // And even less accessible methods!
        callHiddenMethod(a, "u");
        callHiddenMethod(a, "v");
        callHiddenMethod(a, "w");
    }

    static void callHiddenMethod(Object a, String methodName) throws Exception {
        Method g = a.getClass().getDeclaredMethod(methodName);
        g.setAccessible(true);
        g.invoke(a);
    }
}
```

輸出結果：

```
public C.f()
typeinfo.packageaccess.C
public C.g()
package C.u()
protected C.v()
private C.w()
```

正如你所看到的，透過使用反射，仍然可以呼叫所有方法，甚至是 `private` 方法！如果知道方法名，你就可以在其 `Method` 物件上呼叫 `setAccessible(true)`，就像在 `callHiddenMethod()` 中看到的那樣。

你可能覺得，可以透過只發布編譯後的程式碼來阻止這種情況，但其實這並不能解決問題。因為只需要執行 `javap`（一個隨 JDK 發布的反編譯器）即可突破這一限制。下面是一個使用 `javap` 的命令列：

```shell
javap -private C
```

`-private` 標誌表示所有的成員都應該顯示，甚至包括私有成員。下面是輸出：

```
class typeinfo.packageaccess.C extends
java.lang.Object implements typeinfo.interfacea.A {
  typeinfo.packageaccess.C();
  public void f();
  public void g();
  void u();
  protected void v();
  private void w();
}
```

因此，任何人都可以獲取你最私有的方法的名字和簽名，然後呼叫它們。

那如果把介面實現為一個私有內部類，又會怎麼樣呢？下面展示了這種情況：

```java
// typeinfo/InnerImplementation.java
// Private inner classes can't hide from reflection

import typeinfo.interfacea.*;

class InnerA {
    private static class C implements A {
        public void f() {
            System.out.println("public C.f()");
        }

        public void g() {
            System.out.println("public C.g()");
        }

        void u() {
            System.out.println("package C.u()");
        }

        protected void v() {
            System.out.println("protected C.v()");
        }

        private void w() {
            System.out.println("private C.w()");
        }
    }

    public static A makeA() {
        return new C();
    }
}

public class InnerImplementation {
    public static void
    main(String[] args) throws Exception {
        A a = InnerA.makeA();
        a.f();
        System.out.println(a.getClass().getName());
        // Reflection still gets into the private class:
        HiddenImplementation.callHiddenMethod(a, "g");
        HiddenImplementation.callHiddenMethod(a, "u");
        HiddenImplementation.callHiddenMethod(a, "v");
        HiddenImplementation.callHiddenMethod(a, "w");
    }
}
```

輸出結果：

```
public C.f()
InnerA$C
public C.g()
package C.u()
protected C.v()
private C.w()
```

這裡對反射仍然沒有任何東西可以隱藏。那麼如果是匿名類呢？

```java
// typeinfo/AnonymousImplementation.java
// Anonymous inner classes can't hide from reflection

import typeinfo.interfacea.*;

class AnonymousA {
    public static A makeA() {
        return new A() {
            public void f() {
                System.out.println("public C.f()");
            }

            public void g() {
                System.out.println("public C.g()");
            }

            void u() {
                System.out.println("package C.u()");
            }

            protected void v() {
                System.out.println("protected C.v()");
            }

            private void w() {
                System.out.println("private C.w()");
            }
        };
    }
}

public class AnonymousImplementation {
    public static void
    main(String[] args) throws Exception {
        A a = AnonymousA.makeA();
        a.f();
        System.out.println(a.getClass().getName());
        // Reflection still gets into the anonymous class:
        HiddenImplementation.callHiddenMethod(a, "g");
        HiddenImplementation.callHiddenMethod(a, "u");
        HiddenImplementation.callHiddenMethod(a, "v");
        HiddenImplementation.callHiddenMethod(a, "w");
    }
}
```

輸出結果：

```
public C.f()
AnonymousA$1
public C.g()
package C.u()
protected C.v()
private C.w()
```

看起來任何方式都沒辦法阻止反射呼叫那些非公共訪問權限的方法。對於欄位來說也是這樣，即便是 `private` 欄位：

```java
// typeinfo/ModifyingPrivateFields.java

import java.lang.reflect.*;

class WithPrivateFinalField {
    private int i = 1;
    private final String s = "I'm totally safe";
    private String s2 = "Am I safe?";

    @Override
    public String toString() {
        return "i = " + i + ", " + s + ", " + s2;
    }
}

public class ModifyingPrivateFields {
    public static void main(String[] args) throws Exception {
        WithPrivateFinalField pf =
                new WithPrivateFinalField();
        System.out.println(pf);
        Field f = pf.getClass().getDeclaredField("i");
        f.setAccessible(true);
        System.out.println(
                "f.getInt(pf): " + f.getInt(pf));
        f.setInt(pf, 47);
        System.out.println(pf);
        f = pf.getClass().getDeclaredField("s");
        f.setAccessible(true);
        System.out.println("f.get(pf): " + f.get(pf));
        f.set(pf, "No, you're not!");
        System.out.println(pf);
        f = pf.getClass().getDeclaredField("s2");
        f.setAccessible(true);
        System.out.println("f.get(pf): " + f.get(pf));
        f.set(pf, "No, you're not!");
        System.out.println(pf);
    }
}
```

輸出結果：

```
i = 1, I'm totally safe, Am I safe?
f.getInt(pf): 1
i = 47, I'm totally safe, Am I safe?
f.get(pf): I'm totally safe
i = 47, I'm totally safe, Am I safe?
f.get(pf): Am I safe?
i = 47, I'm totally safe, No, you're not!
```

但實際上 `final` 欄位在被修改時是安全的。執行時系統會在不拋出異常的情況下接受任何修改的嘗試，但是實際上不會發生任何修改。

通常，所有這些違反訪問權限的操作並不是什麼十惡不赦的。如果有人使用這樣的技術去呼叫標誌為 `private` 或包訪問權限的方法（很明顯這些訪問權限表示這些人不應該呼叫它們），那麼對他們來說，如果你修改了這些方法的某些地方，他們不應該抱怨。另一方面，總是在類中留下後門，也許會幫助你解決某些特定類型的問題（這些問題往往除此之外，別無它法）。總之，不可否認，反射給我們帶來了很多好處。

程式設計師往往對程式語言提供的訪問控制過於自信，甚至認為 Java 在安全性上比其它提供了（明顯）更寬鬆的訪問控制的語言要優越[^4]。然而，正如你所看到的，事實並不是這樣。

<!-- Summary -->
## 本章小結

RTTI 允許透過匿名類的引用來獲取類型訊息。初學者極易誤用它，因為在學會使用多態呼叫方法之前，這麼做也很有效。有過程化編程背景的人很容易把程式組織成一系列 `switch` 語句，你可以用 RTTI 和 `switch` 實現功能，但這樣就損失了多態機制在程式碼開發和維護過程中的重要價值。物件導向程式語言是想讓我們儘可能地使用多態機制，只在非用不可的時候才使用 RTTI。

然而使用多態機制的方法呼叫，要求我們擁有基類定義的控制權。因為在你擴展程式的時候，可能會發現基類並未包含我們想要的方法。如果基類來自別人的庫，這時 RTTI 便是一種解決之道：可繼承一個新類，然後添加你需要的方法。在程式碼的其它地方，可以檢查你自己特定的類型，並呼叫你自己的方法。這樣做不會破壞多態性以及程式的擴展能力，因為這樣添加一個新的類並不需要修改程式中的 `switch` 語句。但如果想在程式中增加具有新特性的程式碼，你就必須使用 RTTI 來檢查這個特定的類型。

如果只是為了方便某個特定的類，就將某個特性放進基類裡面，這將使得從那個基類衍生出的所有其它子類都帶有這些可能毫無意義的東西。這會導致介面更加不清晰，因為我們必須覆蓋從基類繼承而來的所有抽象方法，事情就變得很麻煩。舉個例子，現在有一個表示樂器 `Instrument` 的類層次結構。假設我們想清理管弦樂隊中某些樂器殘留的口水，一種辦法是在基類 `Instrument` 中放入 `clearSpitValve()` 方法。但這樣做會導致類結構混亂，因為這意味著打擊樂器 `Percussion`、弦樂器 `Stringed` 和電子樂器 `Electronic` 也需要清理口水。在這個例子中，RTTI 可以提供一種更合理的解決方案。可以將 `clearSpitValve()` 放在某個合適的類中，在這個例子中是管樂器 `Wind`。不過，在這裡你可能會發現還有更好的解決方法，就是將 `prepareInstrument()` 放在基類中，但是初次面對這個問題的讀者可能想不到還有這樣的解決方案，而誤認為必須使用 RTTI。

最後一點，RTTI 有時候也能解決效率問題。假設你的程式碼運用了多態，但是為了實現多態，導致其中某個物件的效率非常低。這時候，你就可以挑出那個類，使用 RTTI 為它編寫一段特別的程式碼以提高效率。然而必須注意的是，不要太早地關注程式的效率問題，這是個誘人的陷阱。最好先讓程式能跑起來，然後再去看看程式能不能跑得更快，下一步才是去解決效率問題（比如使用 Profiler）[^5]。

我們已經看到，反射，因其更加動態的程式風格，為我們開創了編程的新世界。但對有些人來說，反射的動態特性卻是一種困擾。對那些已經習慣於靜態類型檢查的安全性的人來說，Java 中允許這種動態類型檢查（只在執行時才能檢查到，並以異常的形式上報檢查結果）的操作似乎是一種錯誤的方向。有些人想得更遠，他們認為引入執行時異常本身就是一種指示，指示我們應該避免這種程式碼。我發現這種意義的安全是一種錯覺，因為總是有些事情是在執行時才發生並拋出異常的，即使是在那些不包含任何 `try` 語句塊或異常聲明的程式中也是如此。因此，我認為一致性錯誤報告模型的存在使我們能夠透過使用反射編寫動態程式碼。當然，盡力編寫能夠進行靜態檢查的程式碼是有價值的，只要你有這樣的能力。但是我相信動態程式碼是將 Java 與其它諸如 C++ 這樣的語言區分開的重要工具之一。

[^1]: 特別是在過去。但現在 Java 的 HTML 文件有了很大的提升，要查看基類的方法已經變得很容易了。

[^2]: 這是極限編程（XP，Extreme Programming）的原則之一：“Try the simplest thing that could possibly work，實現盡最大可能的簡單。”

[^3]: 最著名的例子是 Windows 作業系統，Windows 為開發者提供了公開的 API，但是開發者還可以找到一些非公開但是可以呼叫的函數。為了解決問題，很多程式設計師使用了隱藏的 API 函數。這就迫使微軟公司要像維護公開 API 一樣維護這些隱藏的 API，消耗了巨大的成本和精力。

[^4]: 比如，Python 中在元素前面添加雙下劃線 `__`，就表示你想隱藏這個元素。如果你在類或者包外面呼叫了這個元素，執行環境就會報錯。

[^5]: 譯者註：Java Profiler 是一種 Java 性能分析工具，用於在 JVM 級別監視 Java 位元組碼的構造和執行。主流的 Profiler 有 JProfiler、YourKit 和 Java VisualVM 等。

<!-- 分頁 -->

<div style="page-break-after: always;"></div>
