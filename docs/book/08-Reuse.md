[TOC]

<!-- Reuse -->

# 第八章 復用


> 程式碼復用是物件導向編程（OOP）最具魅力的原因之一。

對於像 C 語言等程序導向語言來說，“復用”通常指的就是“複製程式碼”。任何語言都可透過簡單複製來達到程式碼復用的目的，但是這樣做的效果並不好。Java 圍繞“類”（Class）來解決問題。我們可以直接使用別人構建或除錯過的程式碼，而非建立新類、重新開始。

如何在不汙染原始碼的前提下使用現存程式碼是需要技巧的。在本章裡，你將學習到兩種方式來達到這個目的：

1. 第一種方式直接了當。在新類中建立現有類的物件。這種方式叫做“組合”（Composition），透過這種方式復用程式碼的功能，而非其形式。

2. 第二種方式更為微妙。建立現有類類型的新類。照字面理解：採用現有類形式，又無需在編碼時改動其程式碼，這種方式就叫做“繼承”（Inheritance），編譯器會做大部分的工作。**繼承**是物件導向編程（OOP）的重要基礎之一。更多功能相關將在[多態](./09-Polymorphism.md)（Polymorphism）章節中介紹。

組合與繼承的語法、行為上有許多相似的地方（這其實是有道理的，畢竟都是基於現有類型構建新的類型）。在本章中，你會學到這兩種程式碼復用的方法。

<!-- Composition Syntax -->

## 組合語法

在前面的學習中，“組合”（Composition）已經被多次使用。你僅需要把物件的引用（object references）放置在一個新的類裡，這就使用了組合。例如，假設你需要一個物件，其中內建了幾個 **String** 物件，兩個基本類型（primitives）的屬性欄位，一個其他類的物件。對於非基本類型物件，將引用直接放置在新類中，對於基本類型屬性欄位則僅進行聲明。

```java
// reuse/SprinklerSystem.java
// (c)2017 MindView LLC: see Copyright.txt
// We make no guarantees that this code is fit for any purpose.
// Visit http://OnJava8.com for more book information.
// Composition for code reuse

class WaterSource {
  private String s;
  WaterSource() {
    System.out.println("WaterSource()");
    s = "Constructed";
  }
  @Override
  public String toString() { return s; }
}

public class SprinklerSystem {
  private String valve1, valve2, valve3, valve4;
  private WaterSource source = new WaterSource();
  private int i;
  private float f;
  @Override
  public String toString() {
    return
      "valve1 = " + valve1 + " " +
      "valve2 = " + valve2 + " " +
      "valve3 = " + valve3 + " " +
      "valve4 = " + valve4 + "\n" +
      "i = " + i + " " + "f = " + f + " " +
      "source = " + source; // [1]
  }
  public static void main(String[] args) {
    SprinklerSystem sprinklers = new SprinklerSystem();
    System.out.println(sprinklers);
  }
}
/* Output:
WaterSource()
valve1 = null valve2 = null valve3 = null valve4 = null
i = 0 f = 0.0 source = Constructed
*/

```

這兩個類中定義的一個方法是特殊的:  `toString()`。每個非基本類型物件都有一個 `toString()` 方法，在編譯器需要字串但它有物件的特殊情況下呼叫該方法。因此，在 [1] 中，編譯器看到你試圖“添加”一個 **WaterSource** 類型的字串物件 。因為字串只能拼接另一個字串，所以它就先會呼叫 `toString()` 將 **source** 轉換成一個字串。然後，它可以拼接這兩個字串並將結果字串傳遞給 `System.out.println()`。要對建立的任何類允許這種行為，只需要編寫一個 **toString()** 方法。在 `toString()` 上使用 **@Override** 註解來告訴編譯器，以確保正確地覆蓋。**@Override** 是可選的，但它有助於驗證你沒有拼寫錯誤 (或者更微妙地說，大小寫字母輸入錯誤)。類中的基本類型欄位自動初始化為零，正如 **object Everywhere** 一章中所述。但是物件引用被初始化為 **null**，如果你嘗試呼叫其任何一個方法，你將得到一個異常（一個執行時錯誤）。方便的是，列印 **null** 引用卻不會得到異常。

編譯器不會為每個引用建立一個預設物件，這是有意義的，因為在許多情況下，這會導致不必要的開銷。初始化引用有四種方法:

1. 當物件被定義時。這意味著它們總是在呼叫建構子之前初始化。
2. 在該類的建構子中。
3. 在實際使用物件之前。這通常稱為*延遲初始化*。在物件建立開銷大且不需要每次都建立物件的情況下，它可以減少開銷。
4. 使用實例初始化。

以上四種實例建立的方法例子在這：

```java
// reuse/Bath.java
// (c)2017 MindView LLC: see Copyright.txt
// We make no guarantees that this code is fit for any purpose.
// Visit http://OnJava8.com for more book information.
// Constructor initialization with composition

class Soap {
  private String s;
  Soap() {
    System.out.println("Soap()");
    s = "Constructed";
  }
  @Override
  public String toString() { return s; }
}

public class Bath {
  private String // Initializing at point of definition:
    s1 = "Happy",
    s2 = "Happy",
    s3, s4;
  private Soap castille;
  private int i;
  private float toy;
  public Bath() {
    System.out.println("Inside Bath()");
    s3 = "Joy";
    toy = 3.14f;
    castille = new Soap();
  }
  // Instance initialization:
  { i = 47; }
  @Override
  public String toString() {
    if(s4 == null) // Delayed initialization:
      s4 = "Joy";
    return
      "s1 = " + s1 + "\n" +
      "s2 = " + s2 + "\n" +
      "s3 = " + s3 + "\n" +
      "s4 = " + s4 + "\n" +
      "i = " + i + "\n" +
      "toy = " + toy + "\n" +
      "castille = " + castille;
  }
  public static void main(String[] args) {
    Bath b = new Bath();
    System.out.println(b);
  }
}
/* Output:
Inside Bath()
Soap()
s1 = Happy
s2 = Happy
s3 = Joy
s4 = Joy
i = 47
toy = 3.14
castille = Constructed
*/

```

在 **Bath** 建構子中，有一個程式碼塊在所有初始化發生前就已經執行了。當你不在定義處初始化時，仍然不能保證在向物件引用發送消息之前執行任何初始化——如果你試圖對未初始化的引用呼叫方法，則未初始化的引用將產生執行時異常。

當呼叫 `toString()` 時，它將賦值 s4，以便在使用欄位的時候所有的屬性都已被初始化。

<!-- Inheritance Syntax -->

## 繼承語法

繼承是所有物件導向語言的一個組成部分。事實證明，在建立類時總是要繼承，因為除非顯式地繼承其他類，否則就隱式地繼承 Java 的標準根類物件（Object）。

組合的語法很明顯，但是繼承使用了一種特殊的語法。當你繼承時，你說，“這個新類與那個舊類類似。你可以在類主體的左大括號前的程式碼中聲明這一點，使用關鍵字 **extends** 後跟基類的名稱。當你這樣做時，你將自動獲得基類中的所有欄位和方法。這裡有一個例子:

```java
// reuse/Detergent.java
// (c)2017 MindView LLC: see Copyright.txt
// We make no guarantees that this code is fit for any purpose.
// Visit http://OnJava8.com for more book information.
// Inheritance syntax & properties

class Cleanser {
  private String s = "Cleanser";
  public void append(String a) { s += a; }
  public void dilute() { append(" dilute()"); }
  public void apply() { append(" apply()"); }
  public void scrub() { append(" scrub()"); }
  @Override
  public String toString() { return s; }
  public static void main(String[] args) {
    Cleanser x = new Cleanser();
    x.dilute(); x.apply(); x.scrub();
    System.out.println(x);
  }
}

public class Detergent extends Cleanser {
  // Change a method:
  @Override
  public void scrub() {
    append(" Detergent.scrub()");
    super.scrub(); // Call base-class version
  }
  // Add methods to the interface:
  public void foam() { append(" foam()"); }
  // Test the new class:
  public static void main(String[] args) {
    Detergent x = new Detergent();
    x.dilute();
    x.apply();
    x.scrub();
    x.foam();
    System.out.println(x);
    System.out.println("Testing base class:");
    Cleanser.main(args);
  }
}
/* Output:
Cleanser dilute() apply() Detergent.scrub() scrub()
foam()
Testing base class:
Cleanser dilute() apply() scrub()
*/

```

這示範了一些特性。首先，在 **Cleanser** 的 `append()` 方法中，使用 `+=` 操作符將字串連接到 **s**，這是 Java 設計人員“重載”來處理字串的操作符之一 (還有 + )。

第二，**Cleanser** 和 **Detergent** 都包含一個 `main()` 方法。你可以為每個類建立一個 `main()` ; 這允許對每個類進行簡單的測試。當你完成測試時，不需要刪除 `main()`; 你可以將其留在以後的測試中。即使程式中有很多類都有 `main()` 方法，惟一執行的只有在命令列上呼叫的 `main()`。這裡，當你使用 **java Detergent** 時候，就呼叫了 `Detergent.main()`。但是你也可以使用 **java Cleanser** 來呼叫 `Cleanser.main()`，即使 **Cleanser** 不是一個公共類。即使類只具有包訪問權，也可以訪問 `public main()`。

在這裡，`Detergent.main()` 顯式地呼叫 `Cleanser.main()`，從命令列傳遞相同的參數(當然，你可以傳遞任何字串陣列)。

**Cleanser** 中的所有方法都是公開的。請記住，如果不使用任何訪問修飾符，則成員預設為包訪問權限，這隻允許包內成員訪問。因此，如果沒有訪問修飾符，那麼包內的任何人都可以使用這些方法。例如，**Detergent** 就沒有問題。但是，如果其他包中的類繼承 **Cleanser**，則該類只能訪問 **Cleanser** 的公共成員。因此，為了允許繼承，一般規則是所有欄位為私有，所有方法為公共。(**protected**成員也允許衍生類訪問;你以後會知道的。)在特定的情況下，你必須進行調整，但這是一個有用的指南。

**Cleanser** 的介面中有一組方法: `append()`、`dilute()`、`apply()`、`scrub()` 和 `toString()`。因為 **Detergent** 是從 **Cleanser** 衍生的(通過 **extends** 關鍵字)，所以它會在其介面中自動獲取所有這些方法，即使你沒有在 **Detergent** 中看到所有這些方法的顯式定義。那麼，可以把繼承看作是復用類。如在 `scrub()` 中所見，可以使用基類中定義的方法並修改它。在這裡，你可以在新類中呼叫基類的該方法。但是在 `scrub()` 內部，不能簡單地呼叫 `scrub()`，因為這會產生遞迴呼叫。為了解決這個問題，Java的 **super** 關鍵字引用了目前類繼承的“超類”(基類)。因此表達式 `super.scrub()` 呼叫方法 `scrub()` 的基類版本。

繼承時，你不受限於使用基類的方法。你還可以像向類添加任何方法一樣向衍生類添加新方法:只需定義它。方法 `foam()` 就是一個例子。`Detergent.main()` 中可以看到，對於 **Detergent** 物件，你可以呼叫 **Cleanser** 和 **Detergent** 中可用的所有方法 (如 `foam()` )。

<!-- Initializing the Base Class -->

### 初始化基類

現在涉及到兩個類:基類和衍生類。想像衍生類生成的結果物件可能會讓人感到困惑。從外部看，新類與基類具有相同的介面，可能還有一些額外的方法和欄位。但是繼承並不只是複製基類的介面。當你建立衍生類的物件時，它包含基類的子物件。這個子物件與你自己建立基類的對像是一樣的。只是從外部看，基類的子物件被包裝在衍生類的物件中。

必須正確初始化基類子物件，而且只有一種方法可以保證這一點 : 透過呼叫基類建構子在建構子中執行初始化，該建構子具有執行基類初始化所需的所有適當訊息和特權。Java 自動在衍生類建構子中插入對基類建構子的呼叫。下面的例子展示了三個層次的繼承:

```java
// reuse/Cartoon.java
// (c)2017 MindView LLC: see Copyright.txt
// We make no guarantees that this code is fit for any purpose.
// Visit http://OnJava8.com for more book information.
// Constructor calls during inheritance

class Art {
  Art() {
    System.out.println("Art constructor");
  }
}

class Drawing extends Art {
  Drawing() {
    System.out.println("Drawing constructor");
  }
}

public class Cartoon extends Drawing {
  public Cartoon() {
    System.out.println("Cartoon constructor");
  }
  public static void main(String[] args) {
    Cartoon x = new Cartoon();
  }
}
/* Output:
Art constructor
Drawing constructor
Cartoon constructor
*/

```

構造從基類“向外”進行，因此基類在衍生類建構子能夠訪問它之前進行初始化。即使不為 **Cartoon** 建立建構子，編譯器也會為你合成一個無參數建構子，呼叫基類建構子。嘗試刪除 **Cartoon** 建構子來查看這個。

<!-- Constructors with Arguments -->

### 帶參數的建構子

上面的所有例子中建構子都是無參數的 ; 編譯器很容易呼叫這些建構子，因為不需要參數。如果沒有無參數的基類建構子，或者必須呼叫具有參數的基類建構子，則必須使用 **super** 關鍵字和適當的參數列表顯式地編寫對基類建構子的呼叫:

```java
// reuse/Chess.java
// (c)2017 MindView LLC: see Copyright.txt
// We make no guarantees that this code is fit for any purpose.
// Visit http://OnJava8.com for more book information.
// Inheritance, constructors and arguments

class Game {
  Game(int i) {
    System.out.println("Game constructor");
  }
}

class BoardGame extends Game {
  BoardGame(int i) {
    super(i);
    System.out.println("BoardGame constructor");
  }
}

public class Chess extends BoardGame {
  Chess() {
    super(11);
    System.out.println("Chess constructor");
  }
  public static void main(String[] args) {
    Chess x = new Chess();
  }
}
/* Output:
Game constructor
BoardGame constructor
Chess constructor
*/

```

如果沒有在 **BoardGame** 建構子中呼叫基類建構子，編譯器就會報錯找不到 `Game()` 的建構子。此外，對基類建構子的呼叫必須是衍生類建構子中的第一個操作。(如果你寫錯了，編譯器會提醒你。)

<!-- Delegation -->

## 委託

Java不直接支援的第三種重用關係稱為委託。這介於繼承和組合之間，因為你將一個成員物件放在正在構建的類中(比如組合)，但同時又在新類中公開來自成員物件的所有方法(比如繼承)。例如，宇宙飛船需要一個控制模組:

```java
// reuse/SpaceShipControls.java
// (c)2017 MindView LLC: see Copyright.txt
// We make no guarantees that this code is fit for any purpose.
// Visit http://OnJava8.com for more book information.

public class SpaceShipControls {
  void up(int velocity) {}
  void down(int velocity) {}
  void left(int velocity) {}
  void right(int velocity) {}
  void forward(int velocity) {}
  void back(int velocity) {}
  void turboBoost() {}
}

```

建造宇宙飛船的一種方法是使用繼承:

```java
// reuse/DerivedSpaceShip.java
// (c)2017 MindView LLC: see Copyright.txt
// We make no guarantees that this code is fit for any purpose.
// Visit http://OnJava8.com for more book information.

public class
DerivedSpaceShip extends SpaceShipControls {
  private String name;
  public DerivedSpaceShip(String name) {
    this.name = name;
  }
  @Override
  public String toString() { return name; }
  public static void main(String[] args) {
    DerivedSpaceShip protector =
        new DerivedSpaceShip("NSEA Protector");
    protector.forward(100);
  }
}

```

然而， **DerivedSpaceShip** 並不是真正的“一種” **SpaceShipControls** ，即使你“告訴” **DerivedSpaceShip** 呼叫 `forward()`。更準確地說，一艘宇宙飛船包含了 **SpaceShipControls**，同時 **SpaceShipControls** 中的所有方法都暴露在宇宙飛船中。委託解決了這個難題:

```java
// reuse/SpaceShipDelegation.java
// (c)2017 MindView LLC: see Copyright.txt
// We make no guarantees that this code is fit for any purpose.
// Visit http://OnJava8.com for more book information.

public class SpaceShipDelegation {
  private String name;
  private SpaceShipControls controls =
    new SpaceShipControls();
  public SpaceShipDelegation(String name) {
    this.name = name;
  }
  // Delegated methods:
  public void back(int velocity) {
    controls.back(velocity);
  }
  public void down(int velocity) {
    controls.down(velocity);
  }
  public void forward(int velocity) {
    controls.forward(velocity);
  }
  public void left(int velocity) {
    controls.left(velocity);
  }
  public void right(int velocity) {
    controls.right(velocity);
  }
  public void turboBoost() {
    controls.turboBoost();
  }
  public void up(int velocity) {
    controls.up(velocity);
  }
  public static void main(String[] args) {
    SpaceShipDelegation protector =
      new SpaceShipDelegation("NSEA Protector");
    protector.forward(100);
  }
}

```

方法被轉發到底層 **control** 物件，因此介面與繼承的介面是相同的。但是，你對委託有更多的控制，因為你可以選擇只在成員物件中提供方法的子集。

雖然Java語言不支援委託，但是開發工具常常支援。例如，上面的例子是使用 JetBrains Idea IDE 自動生成的。

<!-- Combining Composition and Inheritance -->

## 結合組合與繼承

你將經常同時使用組合和繼承。下面的例子展示了使用繼承和組合建立類，以及必要的建構子初始化:

```java
// reuse/PlaceSetting.java
// (c)2017 MindView LLC: see Copyright.txt
// We make no guarantees that this code is fit for any purpose.
// Visit http://OnJava8.com for more book information.
// Combining composition & inheritance

class Plate {
  Plate(int i) {
    System.out.println("Plate constructor");
  }
}

class DinnerPlate extends Plate {
  DinnerPlate(int i) {
    super(i);
    System.out.println("DinnerPlate constructor");
  }
}

class Utensil {
  Utensil(int i) {
    System.out.println("Utensil constructor");
  }
}

class Spoon extends Utensil {
  Spoon(int i) {
    super(i);
    System.out.println("Spoon constructor");
  }
}

class Fork extends Utensil {
  Fork(int i) {
    super(i);
    System.out.println("Fork constructor");
  }
}

class Knife extends Utensil {
  Knife(int i) {
    super(i);
    System.out.println("Knife constructor");
  }
}

// A cultural way of doing something:
class Custom {
  Custom(int i) {
    System.out.println("Custom constructor");
  }
}

public class PlaceSetting extends Custom {
  private Spoon sp;
  private Fork frk;
  private Knife kn;
  private DinnerPlate pl;
  public PlaceSetting(int i) {
    super(i + 1);
    sp = new Spoon(i + 2);
    frk = new Fork(i + 3);
    kn = new Knife(i + 4);
    pl = new DinnerPlate(i + 5);
    System.out.println("PlaceSetting constructor");
  }
  public static void main(String[] args) {
    PlaceSetting x = new PlaceSetting(9);
  }
}
/* Output:
Custom constructor
Utensil constructor
Spoon constructor
Utensil constructor
Fork constructor
Utensil constructor
Knife constructor
Plate constructor
DinnerPlate constructor
PlaceSetting constructor
*/

```

儘管編譯器強制你初始化基類，並要求你在建構子的開頭就初始化基類，但它並不監視你以確保你初始化了成員物件。注意類是如何干淨地分離的。你甚至不需要方法重用程式碼的原始碼。你最多只匯入一個包。(這對於繼承和組合都是正確的。)

<!-- Guaranteeing Proper Cleanup -->

### 保證適當的清理

Java 沒有 C++ 中解構子的概念，解構子是在物件被銷毀時自動呼叫的方法。原因可能是，在Java中，通常是忘掉而不是銷毀物件，從而允許垃圾收集器根據需要回收記憶體。通常這是可以的，但是有時你的類可能在其生命週期中執行一些需要清理的活動。初始化和清理章節提到，你無法知道垃圾收集器何時會被呼叫，甚至它是否會被呼叫。因此，如果你想為類清理一些東西，必須顯式地編寫一個特殊的方法來完成它，並確保用戶端程式設計師知道他們必須呼叫這個方法。最重要的是——正如在"異常"章節中描述的——你必須透過在 **finally**子句中放置此類清理來防止異常。

請考慮一個在螢幕上繪製圖片的電腦輔助設計系統的例子:

```java
// reuse/CADSystem.java
// (c)2017 MindView LLC: see Copyright.txt
// We make no guarantees that this code is fit for any purpose.
// Visit http://OnJava8.com for more book information.
// Ensuring proper cleanup
// {java reuse.CADSystem}
package reuse;

class Shape {
  Shape(int i) {
    System.out.println("Shape constructor");
  }
  void dispose() {
    System.out.println("Shape dispose");
  }
}

class Circle extends Shape {
  Circle(int i) {
    super(i);
    System.out.println("Drawing Circle");
  }
  @Override
  void dispose() {
    System.out.println("Erasing Circle");
    super.dispose();
  }
}

class Triangle extends Shape {
  Triangle(int i) {
    super(i);
    System.out.println("Drawing Triangle");
  }
  @Override
  void dispose() {
    System.out.println("Erasing Triangle");
    super.dispose();
  }
}

class Line extends Shape {
  private int start, end;
  Line(int start, int end) {
    super(start);
    this.start = start;
    this.end = end;
    System.out.println(
      "Drawing Line: " + start + ", " + end);
  }
  @Override
  void dispose() {
    System.out.println(
      "Erasing Line: " + start + ", " + end);
    super.dispose();
  }
}

public class CADSystem extends Shape {
  private Circle c;
  private Triangle t;
  private Line[] lines = new Line[3];
  public CADSystem(int i) {
    super(i + 1);
    for(int j = 0; j < lines.length; j++)
      lines[j] = new Line(j, j*j);
    c = new Circle(1);
    t = new Triangle(1);
    System.out.println("Combined constructor");
  }
  @Override
  public void dispose() {
    System.out.println("CADSystem.dispose()");
    // The order of cleanup is the reverse
    // of the order of initialization:
    t.dispose();
    c.dispose();
    for(int i = lines.length - 1; i >= 0; i--)
      lines[i].dispose();
    super.dispose();
  }
  public static void main(String[] args) {
    CADSystem x = new CADSystem(47);
    try {
      // Code and exception handling...
    } finally {
      x.dispose();
    }
  }
}
/* Output:
Shape constructor
Shape constructor
Drawing Line: 0, 0
Shape constructor
Drawing Line: 1, 1
Shape constructor
Drawing Line: 2, 4
Shape constructor
Drawing Circle
Shape constructor
Drawing Triangle
Combined constructor
CADSystem.dispose()
Erasing Triangle
Shape dispose
Erasing Circle
Shape dispose
Erasing Line: 2, 4
Shape dispose
Erasing Line: 1, 1
Shape dispose
Erasing Line: 0, 0
Shape dispose
Shape dispose
*/

```

這個系統中的所有東西都是某種 **Shape** (它本身是一種 **Object**，因為它是從根類隱式繼承的) 。除了使用 **super** 呼叫該方法的基類版本外，每個類還覆蓋 `dispose()` 方法。特定的 **Shape** 類——**Circle**、**Triangle** 和 **Line**，都有 “draw” 建構子，儘管在物件的生命週期中呼叫的任何方法都可以負責做一些需要清理的事情。每個類都有自己的 `dispose()` 方法來將非記憶體的內容復原到物件存在之前的狀態。

在 `main()` 中，有兩個關鍵字是你以前沒有見過的，在"異常"一章之前不會詳細解釋: **try** 和 **finally**。**try** 關鍵字表示後面的塊 (用花括號分隔 )是一個受保護的區域，這意味著它得到了特殊處理。其中一個特殊處理是，無論 **try** 塊如何退出，在這個保護區域之後的 **finally** 子句中的程式碼總是被執行。(透過異常處理，可以用許多不同尋常的方式留下 **try** 塊。)這裡，**finally** 子句的意思是，“無論發生什麼事，始終呼叫 `x.dispose()`。”

在清理方法 (在本例中是 `dispose()` ) 中，還必須注意基類和成員物件清理方法的呼叫順序，以防一個子物件依賴於另一個子物件。首先，按與建立的相反順序執行特定於類的所有清理工作。(一般來說，這要求基類元素仍然是可訪問的。) 然後呼叫基類清理方法，如這所示。

在很多情況下，清理問題不是問題；你只需要讓垃圾收集器來完成這項工作。但是，當你必須執行顯式清理時，就需要多做努力，更加細心，因為在垃圾收集方面沒有什麼可以依賴的。可能永遠不會呼叫垃圾收集器。如果呼叫，它可以按照它想要的任何順序回收物件。除了記憶體回收外，你不能依賴垃圾收集來做任何事情。如果希望進行清理，可以使用自己的清理方法，不要使用 `finalize()`。

<!-- Name Hiding -->

### 名稱隱藏

如果 Java 基類的方法名多次重載，則在衍生類中重新定義該方法名不會隱藏任何基類版本。不管方法是在這個級別定義的，還是在基類中定義的，重載都會起作用:

````java
// reuse/Hide.java
// (c)2017 MindView LLC: see Copyright.txt
// We make no guarantees that this code is fit for any purpose.
// Visit http://OnJava8.com for more book information.
// Overloading a base-class method name in a derived
// class does not hide the base-class versions

class Homer {
  char doh(char c) {
    System.out.println("doh(char)");
    return 'd';
  }
  float doh(float f) {
    System.out.println("doh(float)");
    return 1.0f;
  }
}

class Milhouse {}

class Bart extends Homer {
  void doh(Milhouse m) {
    System.out.println("doh(Milhouse)");
  }
}

public class Hide {
  public static void main(String[] args) {
    Bart b = new Bart();
    b.doh(1);
    b.doh('x');
    b.doh(1.0f);
    b.doh(new Milhouse());
  }
}
/* Output:
doh(float)
doh(char)
doh(float)
doh(Milhouse)
*/

````

**Homer** 的所有重載方法在 **Bart** 中都是可用的，儘管 **Bart** 引入了一種新的重載方法。正如你將在下一章中看到的那樣，比起重載，更常見的是覆蓋同名方法，使用與基類中完全相同的方法簽名[^2]和返回類型。否則會讓人感到困惑。

你已經看到了Java 5 **@Override**註解，它不是關鍵字，但是可以像使用關鍵字一樣使用它。當你打算重寫一個方法[^3]時，你可以選擇添加這個註解，如果你不小心用了重載而不是重寫，編譯器會產生一個錯誤消息:


```java
// reuse/Lisa.java
// (c)2017 MindView LLC: see Copyright.txt
// We make no guarantees that this code is fit for any purpose.
// Visit http://OnJava8.com for more book information.
// {WillNotCompile}

class Lisa extends Homer {
  @Override void doh(Milhouse m) {
    System.out.println("doh(Milhouse)");
  }
}

```

**{WillNotCompile}** 標記將該文件排除在本書的 **Gradle** 構建之外，但是如果你手工編譯它，你將看到:method does not override a method from its superclass.方法不會重寫超類中的方法， **@Override** 註解能防止你意外地重載。

       
<!-- Choosing Composition vs. Inheritance -->

## 組合與繼承的選擇

組合和繼承都允許在新類中放置子物件（組合是顯式的，而繼承是隱式的）。你或許想知道這二者之間的區別，以及怎樣在二者間做選擇。

當你想在新類中包含一個已有類的功能時，使用組合，而非繼承。也就是說，在新類中嵌入一個物件（通常是私有的），以實現其功能。新類的使用者看到的是你所定義的新類的介面，而非嵌入物件的介面。

有時讓類的使用者直接訪問到新類中的組合成分是有意義的。只需將成員物件聲明為 **public** 即可（可以把這當作“半委託”的一種）。成員物件隱藏了具體實現，所以這是安全的。當使用者知道你正在組裝一組部件時，會使得介面更加容易理解。下面的 car 物件是個很好的例子：

```java
// reuse/Car.java
// Composition with public objects
class Engine {
    public void start() {}
    public void rev() {}
    public void stop() {}
}

class Wheel {
    public void inflate(int psi) {}
}

class Window {
    public void rollup() {}
    public void rolldown() {}
}

class Door {
    public Window window = new Window();
    
    public void open() {}
    public void close() {}
}

public class Car {
    public Engine engine = new Engine();
    public Wheel[] wheel = new Wheel[4];
    public Door left = new Door(), right = new Door(); // 2-door
    
    public Car() {
        for (int i = 0; i < 4; i++) {
            wheel[i] = new Wheel();
        }
    }
    
    public static void main(String[] args) {
        Car car = new Car();
        car.left.window.rollup();
        car.wheel[0].inflate(72);
    }
}
```

因為在這個例子中 car 的組合也是問題分析的一部分（不是底層設計的部分），所以聲明成員為 **public** 有助於用戶端程式設計師理解如何使用類，且降低了類建立者面臨的程式碼複雜度。但是，記住這是一個特例。通常來說，屬性還是應該聲明為 **private**。

當使用繼承時，使用一個現有類並開發出它的新版本。通常這意味著使用一個通用類，並為了某個特殊需求將其特殊化。稍微思考一下，你就會發現，用一個交通工具物件來組成一部車是毫無意義的——車不包含交通工具，它就是交通工具。這種“是一個”的關係是用繼承來表達的，而“有一個“的關係則用組合來表達。

<!-- protected -->

## protected

即然你已經接觸到繼承，關鍵字 **protected** 就變得有意義了。在理想世界中，僅靠關鍵字 **private** 就足夠了。在實際項目中，卻經常想把一件事物儘量對外界隱藏，而允許衍生類的成員訪問。

關鍵字 **protected** 就起這個作用。它表示“就類的使用者而言，這是 **private** 的。但對於任何繼承它的子類或在同一包中的類，它是可訪問的。”（**protected** 也提供了包訪問權限）

儘管可以建立 **protected** 屬性，但是最好的方式是將屬性聲明為 **private** 以一直保留更改底層實現的權利。然後透過 **protected** 控制類的繼承者的訪問權限。

```java
// reuse/Orc.java
// The protected keyword
class Villain {
    private String name;
    
    protected void set(String nm) {
        name = nm;
    }
    
    Villain(String name) {
        this.name = name;
    }
    
    @Override
    public String toString() {
        return "I'm a Villain and my name is " + name;
    }
}

public class Orc extends Villain {
    private int orcNumber;
    
    public Orc(String name, int orcNumber) {
        super(name);
        this.orcNumber = orcNumber;
    }
    
    public void change(String name, int orcNumber) {
        set(name); // Available because it's protected
        this.orcNumber = orcNumber;
    }
    
    @Override
    public String toString() {
        return "Orc " + orcNumber + ": " + super.toString();
    }
    
    public static void main(String[] args) {
        Orc orc = new Orc("Limburger", 12);
        System.out.println(orc);
        orc.change("Bob", 19);
        System.out.println(orc);
    }
}
```

輸出：

```
Orc 12: I'm a Villain and my name is Limburger
Orc 19: I'm a Villain and my name is Bob
```

`change()` 方法可以訪問 `set()` 方法，因為 `set()` 方法是 **protected**。注意到，類 **Orc** 的  `toString()` 方法也使用了基類的版本。


<!-- Upcasting -->
## 向上轉型

繼承最重要的方面不是為新類提供方法。它是新類與基類的一種關係。簡而言之，這種關係可以表述為“新類是已有類的一種類型”。

這種描述並非是解釋繼承的一種花俏方式，這是直接由語言支援的。例如，假設有一個基類 **Instrument** 代表音樂樂器和一個衍生類 **Wind**。 因為繼承保證了基類的所有方法在衍生類中也是可用的，所以任意發送給該基類的消息也能發送給衍生類。如果 **Instrument** 有一個 `play()` 方法，那麼 **Wind** 也有該方法。這意味著你可以準確地說 **Wind** 物件也是一種類型的 **Instrument**。下面例子展示了編譯器是如何支援這一概念的：

```java
// reuse/Wind.java
// Inheritance & upcasting
class Instrument {
    public void play() {}
    
    static void tune(Instrument i) {
        // ...
        i.play();
    }
}

// Wind objects are instruments
// because they have the same interface:
public class Wind extends Instrument {
    public static void main(String[] args) {
        Wind flute = new Wind();
        Instrument.tune(flute); // Upcasting
    }
}
```

`tune()` 方法接受了一個 **Instrument** 類型的引用。但是，在 **Wind** 的 `main()` 方法裡，`tune()` 方法卻傳入了一個 **Wind** 引用。鑑於 Java 對類型檢查十分嚴格，一個接收一種類型的方法接受了另一種類型看起來很奇怪，除非你意識到 **Wind** 物件同時也是一個 **Instrument** 物件，而且 **Instrument** 的 `tune` 方法一定會存在於 **Wind** 中。在 `tune()` 中，程式碼對 **Instrument** 和 所有 **Instrument** 的衍生類起作用，這種把 **Wind** 引用轉換為 **Instrument** 引用的行為稱作*向上轉型*。

該術語是基於傳統的類繼承圖：圖最上面是根，然後向下鋪展。（當然你可以以任意方式畫你認為有幫助的類圖。）於是，**Wind.java** 的類圖是：

![Wind 類圖](../images/1562204648023.png)

繼承圖中衍生類轉型為基類是向上的，所以通常稱作*向上轉型*。因為是從一個更具體的類轉化為一個更一般的類，所以向上轉型永遠是安全的。也就是說，衍生類是基類的一個超集。它可能比基類包含更多的方法，但它必須至少具有與基類一樣的方法。在向上轉型期間，類介面只可能失去方法，不會增加方法。這就是為什麼編譯器在沒有任何明確轉型或其他特殊標記的情況下，仍然允許向上轉型的原因。

也可以執行與向上轉型相反的向下轉型，但是會有問題，對於該問題會放在下一章和“類型訊息”一章進行更深入的探討。

### 再論組合和繼承

在物件導向編程中，建立和使用程式碼最有可能的方法是將資料和方法一起打包到類中，然後使用該類的物件。也可以使用已有的類透過組合來建立新類。繼承其實不太常用。因此儘管在教授 OOP 的過程中我們多次強調繼承，但這並不意味著要儘可能使用它。恰恰相反，儘量少使用它，除非確實使用繼承是有幫助的。一種判斷使用組合還是繼承的最清晰的方法是問一問自己是否需要把新類向上轉型為基類。如果必須向上轉型，那麼繼承就是必要的，但如果不需要，則要進一步考慮是否該採用繼承。“多態”一章提出了一個使用向上轉型的最有力的理由，但是只要記住問一問“我需要向上轉型嗎？”，就能在這兩者中作出較好的選擇。

<!-- The final Keyword -->

## final關鍵字

根據上下文環境，Java 的關鍵字 **final** 的含義有些微的不同，但通常它指的是“這是不能被改變的”。防止改變有兩個原因：設計或效率。因為這兩個原因相差很遠，所以有可能誤用關鍵字 **final**。

以下幾節討論了可能使用 **final** 的三個地方：資料、方法和類。

### final 資料

許多程式語言都有某種方法告訴編譯器有一塊資料是恆定不變的。恆定是有用的，如：

1. 一個永不改變的編譯時常量。
2. 一個在執行時初始化就不會改變的值。

對於編譯時常量這種情況，編譯器可以把常量帶入計算中；也就是說，可以在編譯時計算，減少了一些執行時的負擔。在 Java 中，這類常量必須是基本類型，而且用關鍵字 **final** 修飾。你必須在定義常量的時候進行賦值。

一個被 **static** 和 **final** 同時修飾的屬性只會占用一段不能改變的儲存空間。

當用 **final** 修飾物件引用而非基本類型時，其含義會有一點令人困惑。對於基本類型，**final** 使數值恆定不變，而對於物件引用，**final** 使引用恆定不變。一旦引用被初始化指向了某個物件，它就不能改為指向其他物件。但是，物件本身是可以修改的，Java 沒有提供將任意物件設為常量的方法。（你可以自己編寫類達到使物件恆定不變的效果）這一限制同樣適用陣列，陣列也是物件。

下面例子展示了 **final** 屬性的使用：

```java
// reuse/FinalData.java
// The effect of final on fields
import java.util.*;

class Value {
    int i; // package access
    
    Value(int i) {
        this.i = i;
    }
}

public class FinalData {
    private static Random rand = new Random(47);
    private String id;
    
    public FinalData(String id) {
        this.id = id;
    }
    // Can be compile-time constants:
    private final int valueOne = 9;
    private static final int VALUE_TWO = 99;
    // Typical public constant:
    public static final int VALUE_THREE = 39;
    // Cannot be compile-time constants:
    private final int i4 = rand.nextInt(20);
    static final int INT_5 = rand.nextInt(20);
    private Value v1 = new Value(11);
    private final Value v2 = new Value(22);
    private static final Value VAL_3 = new Value(33);
    // Arrays:
    private final int[] a = {1, 2, 3, 4, 5, 6};
    
    @Override
    public String toString() {
        return id + ": " + "i4 = " + i4 + ", INT_5 = " + INT_5;
    }
    
    public static void main(String[] args) {
        FinalData fd1 = new FinalData("fd1");
        //- fd1.valueOne++; // Error: can't change value
        fd1.v2.i++; // Object isn't constant
        fd1.v1 = new Value(9); // OK -- not final
        for (int i = 0; i < fd1.a.length; i++) {
            fd1.a[i]++; // Object isn't constant
        }
        //- fd1.v2 = new Value(0); // Error: Can't
        //- fd1.VAL_3 = new Value(1); // change reference
        //- fd1.a = new int[3];
        System.out.println(fd1);
        System.out.println("Creating new FinalData");
        FinalData fd2 = new FinalData("fd2");
        System.out.println(fd1);
        System.out.println(fd2);
    }
}
```

輸出：

```
fd1: i4 = 15, INT_5 = 18
Creating new FinalData
fd1: i4 = 15, INT_5 = 18
fd2: i4 = 13, INT_5 = 18
```

因為 **valueOne** 和 **VALUE_TWO** 都是帶有編譯時值的 **final** 基本類型，它們都可用作編譯時常量，沒有多大區別。**VALUE_THREE** 是一種更加典型的常量定義的方式：**public** 意味著可以在包外訪問，**static** 強調只有一個，**final** 說明是一個常量。

按照慣例，帶有恆定初始值的 **final** **static** 基本變數（即編譯時常量）命名全部使用大寫，單詞之間用下劃線分隔。（源於 C 語言中定義常量的方式。）

我們不能因為某資料被 **final** 修飾就認為在編譯時可以知道它的值。由上例中的 **i4** 和 **INT_5** 可以看出，它們在執行時才會賦值隨機數。範例部分也展示了將 **final** 值定義為 **static** 和非 **static** 的區別。此區別只有當值在執行時被初始化時才會顯現，因為編譯器對編譯時數值一視同仁。（而且編譯時數值可能因最佳化而消失。）當執行程式時就能看到這個區別。注意到 **fd1** 和 **fd2** 的 **i4** 值不同，但 **INT_5** 的值並沒有因為建立了第二個 **FinalData** 物件而改變，這是因為它是 **static** 的，在載入時已經被初始化，並不是每次建立新物件時都初始化。

**v1** 到 **VAL_3** 變數說明了 **final** 引用的意義。正如你在 `main()` 中所見，**v2** 是 **final** 的並不意味著你不能修改它的值。因為它是引用，所以只是說明它不能指向一個新的物件。這對於陣列具有同樣的意義，陣列只不過是另一種引用。（我不知道有什麼方法能使陣列引用本身成為 **final**。）看起來，聲明引用為 **final** 沒有聲明基本類型 **final** 有用。

### 空白 final

空白 final 指的是沒有初始化值的 **final** 屬性。編譯器確保空白 final 在使用前必須被初始化。這樣既能使一個類的每個物件的 **final** 屬性值不同，也能保持它的不變性。

```java
// reuse/BlankFinal.java
// "Blank" final fields
class Poppet {
    private int i;
    
    Poppet(int ii) {
        i = ii;
    }
}

public class BlankFinal {
    private final int i = 0; // Initialized final
    private final int j; // Blank final
    private final Poppet p; // Blank final reference
    // Blank finals MUST be initialized in constructor
    public BlankFinal() {
        j = 1; // Initialize blank final
        p = new Poppet(1); // Init blank final reference
    }
    
    public BlankFinal(int x) {
        j = x; // Initialize blank final
        p = new Poppet(x); // Init blank final reference
    }
    
    public static void main(String[] args) {
        new BlankFinal();
        new BlankFinal(47);
    }
}
```

你必須在定義時或在每個構造器中執行 final 變數的賦值操作。這保證了 final 屬性在使用前已經被初始化過。

### final 參數

在參數列表中，將參數聲明為 final 意味著在方法中不能改變參數指向的物件或基本變數：

```java
// reuse/FinalArguments.java
// Using "final" with method arguments
class Gizmo {
    public void spin() {
        
    }
}

public class FinalArguments {
    void with(final Gizmo g) {
        //-g = new Gizmo(); // Illegal -- g is final
    }
    
    void without(Gizmo g) {
        g = new Gizmo(); // OK -- g is not final
        g.spin();
    }
    
    //void f(final int i) { i++; } // Can't change
    // You can only read from a final primitive
    int g(final int i) {
        return i + 1;
    }
    
    public static void main(String[] args) {
        FinalArguments bf = new FinalArguments();
        bf.without(null);
        bf.with(null);
    }
}
```

方法 `f()` 和 `g()` 展示了 **final** 基本類型參數的使用情況。你只能讀取而不能修改參數。這個特性主要用於傳遞資料給匿名內部類。這將在”內部類“章節中詳解。

### final 方法

使用 **final** 方法的原因有兩個。第一個原因是給方法上鎖，防止子類透過覆寫改變方法的行為。這是出於繼承的考慮，確保方法的行為不會因繼承而改變。

過去建議使用 **final** 方法的第二個原因是效率。在早期的 Java 實現中，如果將一個方法指明為 **final**，就是同意編譯器把對該方法的呼叫轉化為內嵌呼叫。當編譯器遇到 **final** 方法的呼叫時，就會很小心地跳過普通的插入程式碼以執行方法的呼叫機制（將參數壓堆疊，跳至方法程式碼處執行，然後跳回並清理堆疊中的參數，最終處理返回值），而用方法體內實際程式碼的副本替代方法呼叫。這消除了方法呼叫的開銷。但是如果一個方法很大程式碼膨脹，你也許就看不到內嵌帶來的效能提升，因為內嵌呼叫帶來的效能提高被花費在方法裡的時間抵消了。

在最近的 Java 版本中，虛擬機可以探測到這些情況（尤其是 *hotspot* 技術），並最佳化去掉這些效率反而降低的內嵌呼叫方法。有很長一段時間，使用 **final** 來提高效率都被阻止。你應該讓編譯器和 JVM 處理效能問題，只有在為了明確禁止覆寫方法時才使用 **final**。

### final 和 private

類中所有的 **private** 方法都隱式地指定為 **final**。因為不能訪問 **private** 方法，所以不能覆寫它。可以給 **private** 方法添加 **final** 修飾，但是並不能給方法帶來額外的含義。

以下情況會令人困惑，當你試圖覆寫一個 **private** 方法（隱式是 **final** 的）時，看起來奏效，而且編譯器不會給出錯誤訊息：

```java
// reuse/FinalOverridingIllusion.java
// It only looks like you can override
// a private or private final method
class WithFinals {
    // Identical to "private" alone:
    private final void f() {
        System.out.println("WithFinals.f()");
    }
    // Also automatically "final":
    private void g() {
        System.out.println("WithFinals.g()");
    }
}

class OverridingPrivate extends WithFinals {
    private final void f() {
        System.out.println("OverridingPrivate.f()");
    }
    
    private void g() {
        System.out.println("OverridingPrivate.g()");
    }
}

class OverridingPrivate2 extends OverridingPrivate {
    public final void f() {
        System.out.println("OverridingPrivate2.f()");
    } 
    
    public void g() {
        System.out.println("OverridingPrivate2.g()");
    }
}

public class FinalOverridingIllusion {
    public static void main(String[] args) {
        OverridingPrivate2 op2 = new OverridingPrivate2();
        op2.f();
        op2.g();
        // You can upcast:
        OverridingPrivate op = op2;
        // But you can't call the methods:
        //- op.f();
        //- op.g();
        // Same here:
        WithFinals wf = op2;
        //- wf.f();
        //- wf.g();
    }
}
```

輸出：

```
OverridingPrivate2.f()
OverridingPrivate2.g()
```

"覆寫"只發生在方法是基類的介面時。也就是說，必須能將一個物件向上轉型為基類並呼叫相同的方法（這一點在下一章闡明）。如果一個方法是 **private** 的，它就不是基類介面的一部分。它只是隱藏在類內部的程式碼，且恰好有相同的命名而已。但是如果你在衍生類中以相同的命名建立了 **public**，**protected** 或包訪問權限的方法，這些方法與基類中的方法沒有聯繫，你沒有覆寫方法，只是在建立新的方法而已。由於 **private** 方法無法觸及且能有效隱藏，除了把它看作類中的一部分，其他任何事物都不需要考慮到它。

### final 類

當說一個類是 **final** （**final** 關鍵字在類定義之前），就意味著它不能被繼承。之所以這麼做，是因為類的設計就是永遠不需要改動，或者是出於安全考慮不希望它有子類。

```java
// reuse/Jurassic.java
// Making an entire class final
class SmallBrain {}

final class Dinosaur {
    int i = 7;
    int j = 1;
    SmallBrain x = new SmallBrain();
    
    void f() {}
}

//- class Further extends Dinosaur {}
// error: Cannot extend final class 'Dinosaur'
public class Jurassic {
    public static void main(String[] args) {
        Dinosaur n = new Dinosaur();
        n.f();
        n.i = 40;
        n.j++;
    }
}
```

**final** 類的屬性可以根據個人選擇是或不是 **final**。同樣，非 **final** 類的屬性也可以根據個人選擇是或不是 **final**。然而，由於 **final** 類禁止繼承，類中所有的方法都被隱式地指定為 **final**，所以沒有辦法覆寫它們。你可以在 final 類中的方法加上 **final** 修飾符，但不會增加任何意義。

### final 忠告

在設計類時將一個方法指明為 final 看起來是明智的。你可能會覺得沒人會覆寫那個方法。有時這是對的。

但請留意你的假設。通常來說，預見一個類如何被復用是很困難的，特別是通用類。如果將一個方法指定為 **final**，可能會防止其他程式設計師的項目中透過繼承來復用你的類，而這僅僅是因為你沒有想到它被以那種方式使用。

Java 標準類庫就是一個很好的例子。尤其是 Java 1.0/1.1 的 **Vector** 類被廣泛地使用，然而它的所有方法出於"效率"考慮（然而並沒有提升效率，只是幻覺）全被指定為 **final** ，如果不指定 **final** 的話，可能會更加有用[^1]。很容易想到，你可能會繼承並覆寫這麼一個基礎類，但是設計者們認為這麼做不合適。有兩個諷刺的原因。第一，**Stack** 繼承自 **Vector**，就是說 **Stack** 是個 **Vector**，但從邏輯上來說不對。儘管如此，Java 設計者們仍然這麼做，在用這種方式建立 **Stack** 時，他們應該意識到了 **final** 方法過於約束。

第二，**Vector** 中的很多重要方法，比如 `addElement()` 和 `elementAt()` 方法都是同步的。在“並發編程”一章中會看到同步會導致很大的執行開銷，可能會抹煞 **final** 帶來的好處。這加強了程式設計師永遠無法正確猜到最佳化應該發生在何處的觀點。如此笨拙的設計卻出現在每個人都要使用的標準庫中，太糟糕了。慶幸的是，現代 Java 容器用 **ArrayList** 代替了 **Vector**，它的行為要合理得多。不幸的是，仍然有很多新程式碼使用舊的集合類庫，其中就包括 **Vector**。

Java 1.0/1.1 標準類庫中另一個重要的類是 **Hashtable**（後來被 **HashMap** 取代），它不含任何 **final** 方法。本書中其他地方也提到，很明顯不同的類是由不同的人設計的。**Hashtable** 就比 **Vector** 中的方法名簡潔得多，這又是一條證據。對於類庫的使用者來說，這是一個本不應該如此草率的事情。這種不規則的情況造成使用者需要做更多的工作——這是對粗糙的設計和程式碼的又一諷刺。

<!-- Initialization and Class Loading -->

## 類初始化和載入

在許多傳統語言中，程式在啟動時一次性全部載入。接著初始化，然後程式開始執行。必須仔細控制這些語言的初始化過程，以確保 **statics** 初始化的順序不會造成麻煩。在 C++ 中，如果一個 **static** 期望使用另一個 **static**，而另一個 **static** 還沒有初始化，就會出現問題。

Java 中不存在這樣的問題，因為它採用了一種不同的方式載入。因為 Java 中萬物皆物件，所以載入活動就容易得多。記住每個類的編譯程式碼都存在於它自己獨立的文件中。該文件只有在使用程式碼時才會被載入。一般可以說“類的程式碼在首次使用時載入“。這通常是指建立類的第一個物件，或者是訪問了類的 **static** 屬性或方法。構造器也是一個 **static** 方法儘管它的 **static** 關鍵字是隱式的。因此，準確地說，一個類當它任意一個 **static** 成員被訪問時，就會被載入。

首次使用時就是 **static** 初始化發生時。所有的 **static** 物件和 **static** 程式碼塊在載入時按照文字的順序（在類中定義的順序）依次初始化。**static** 變數只被初始化一次。

### 繼承和初始化

了解包括繼承在內的整個初始化過程是有幫助的，這樣可以對所發生的一切有全域性的把握。考慮下面的例子：

```java
// reuse/Beetle.java
// The full process of initialization
class Insect {
    private int i = 9;
    protected int j;
    
    Insect() {
        System.out.println("i = " + i + ", j = " + j);
        j = 39;
    }
    
    private static int x1 = printInit("static Insect.x1 initialized");
    
    static int printInit(String s) {
        System.out.println(s);
        return 47;
    }
}

public class Beetle extends Insect {
    private int k = printInit("Beetle.k.initialized");
    
    public Beetle() {
        System.out.println("k = " + k);
        System.out.println("j = " + j);
    }
    
    private static int x2 = printInit("static Beetle.x2 initialized");
    
    public static void main(String[] args) {
        System.out.println("Beetle constructor");
        Beetle b = new Beetle();
    }
}
```

輸出：

```
static Insect.x1 initialized
static Beetle.x2 initialized
Beetle constructor
i = 9, j = 0
Beetle.k initialized
k = 47
j = 39
```

當執行 **java Beetle**，首先會試圖訪問 **Beetle** 類的 `main()` 方法（一個靜態方法），載入器啟動並找出 **Beetle** 類的編譯程式碼（在名為 **Beetle.class** 的文件中）。在載入過程中，編譯器注意到有一個基類，於是繼續載入基類。不論是否建立了基類的物件，基類都會被載入。（可以嘗試把建立基類物件的程式碼注釋掉證明這點。）

如果基類還存在自身的基類，那麼第二個基類也將被載入，以此類推。接下來，根基類（例子中根基類是 **Insect**）的 **static** 的初始化開始執行，接著是衍生類，以此類推。這點很重要，因為衍生類中 **static** 的初始化可能依賴基類成員是否被正確地初始化。

至此，必要的類都載入完畢，物件可以被建立了。首先，物件中的所有基本類型變數都被置為預設值，物件引用被設為 **null** —— 這是透過將物件記憶體設為二進位制零值一舉生成的。接著會呼叫基類的構造器。本例中是自動呼叫的，但是你也可以使用 **super** 呼叫指定的基類構造器（在 **Beetle** 構造器中的第一步操作）。基類構造器和衍生類構造器一樣以相同的順序經歷相同的過程。當基類構造器完成後，實例變數按文字順序初始化。最終，構造器的剩餘部分被執行。

<!-- Summary -->

## 本章小結

繼承和組合都是從已有類型建立新類型。組合將已有類型作為新類型底層實現的一部分，繼承復用的是介面。

使用繼承時，衍生類具有基類介面，因此可以向上轉型為基類，這對於多態至關重要，在下一章你將看到。

儘管在物件導向編程時極力強調繼承，但在開始設計時，優先使用組合（或委託），只有當確實需要時再使用繼承。組合更具靈活性。另外，透過對成員類型使用繼承的技巧，可以在執行時改變成員的類型和行為。因此，可以在執行時改變組合物件的行為。

在設計一個系統時，目標是發現或建立一系列類，每個類有特定的用途，而且既不應太大（包括太多功能難以復用），也不應太小（不添加其他功能就無法使用）。如果設計變得過於複雜，透過將現有類分割為更小的部分而添加更多的物件，通常是有幫助的。

當開始設計一個系統時，記住程式開發是一個增量過程，正如人類學習。它依賴實驗，你可以儘可能多做分析，然而在項目開始時仍然無法知道所有的答案。如果把項目視作一個有機的，進化著的生命去培養，而不是視為像摩天大樓一樣快速見效，就能獲得更多的成功和更迅速的回饋。繼承和組合正是可以讓你執行如此實驗的物件導向編程中最基本的兩個工具。

[^1]: Java 1.4 開始已將 **Vector** 類大多數方法的 **final** 去掉

[^2]: 方法簽名——方法名和參數類型的合稱

[^3]: 重寫——覆蓋同名方法，使用與基類中完全相同的方法簽名和返回類型[^4]

[^4]: 在java 1.4版本以前，重寫方法的返回值類型被要求必須與被重寫方法一致，但是在java 5.0中放寬了這一個限制，添加了對協變返回類型的支援，在重寫的時候，重寫方法的返回值類型可以是被重寫方法返回值類型的子類。
   

<!-- 分頁 -->

<div style="page-break-after: always;"></div>
