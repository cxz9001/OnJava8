[TOC]

<!-- Functional Programming -->
# 第十三章 函數式編程



> 函數式程式語言操縱程式碼片段就像運算元據一樣容易。 雖然 Java 不是函數式語言，但 Java 8 Lambda 表達式和方法引用 (Method References) 允許你以函數式編程。

在電腦時代早期，記憶體是稀缺和昂貴的。幾乎每個人都用組語語言編程。人們雖然知道編譯器，但編譯器生成的程式碼很低效，比手工編碼的組語程式多很多位元組，僅僅想到這一點，人們還是選擇組語語言。

通常，為了使程式能在有限的記憶體上執行，在程式執行時，程式設計師透過修改記憶體中的程式碼，使程式可以執行不同的操作，用這種方式來節省程式碼空間。這種技術被稱為**自修改程式碼** （self-modifying code）。只要程式小到幾個人就能夠維護所有棘手和難懂的組語程式碼，你就能讓程式執行起來。

隨著記憶體和處理器變得更便宜、更快。C 語言出現並被大多陣列語程式設計師認為更“進階”。人們發現使用 C 可以顯著提高生產力。同時，使用 C 建立自修改程式碼仍然不難。

隨著硬體越來越便宜，程式的規模和複雜性都在增長。這一切只是讓程式工作變得困難。我們想方設法使程式碼更加一致和易懂。使用純粹的自修改程式碼造成的結果就是：我們很難確定程式在做什麼。它也難以測試：除非你想一點點測試輸出，程式碼轉換和修改等等過程？

然而，使用程式碼以某種方式操縱其他程式碼的想法也很有趣，只要能保證它更安全。從程式碼建立，維護和可靠性的角度來看，這個想法非常吸引人。我們不用從頭開始編寫大量程式碼，而是從易於理解、充分測試及可靠的現有小塊開始，最後將它們組合在一起以建立新程式碼。難道這不會讓我們更有效率，同時創造更健壯的程式碼嗎？

這就是**函數式編程**（FP）的意義所在。透過合併現有程式碼來生成新功能而不是從頭開始編寫所有內容，我們可以更快地獲得更可靠的程式碼。至少在某些情況下，這套理論似乎很有用。在這一過程中，函數式語言已經產生了優雅的語法，這些語法對於非函數式語言也適用。

你也可以這樣想：

OO（object oriented，物件導向）是抽象資料，FP（functional programming，函數式編程）是抽象行為。

純粹的函數式語言在安全性方面更進一步。它強加了額外的約束，即所有資料必須是不可變的：設定一次，永不改變。將值傳遞給函數，該函數然後生成新值但從不修改自身外部的任何東西（包括其參數或該函數範圍之外的元素）。當強制執行此操作時，你知道任何錯誤都不是由所謂的副作用引起的，因為該函數僅建立並返回結果，而不是其他任何錯誤。

更好的是，“不可變物件和無副作用”範式解決了並發編程中最基本和最棘手的問題之一（當程式的某些部分同時在多個處理器上執行時）。這是可變共享狀態的問題，這意味著程式碼的不同部分（在不同的處理器上執行）可以嘗試同時修改同一塊記憶體（誰贏了？沒人知道）。如果函數永遠不會修改現有值但只生成新值，則不會對記憶體產生爭用，這是純函數式語言的定義。 因此，經常提出純函數式語言作為並行編程的解決方案（還有其他可行的解決方案）。

需要提醒大家的是，函數式語言背後有很多動機，這意味著描述它們可能會有些混淆。它通常取決於各種觀點：為“並行編程”，“程式碼可靠性”和“程式碼建立和庫復用”。[^1] 關於函數式編程能高效建立更健壯的程式碼這一觀點仍存在部分爭議。雖然已有一些好的範例[^2]，但還不足以證明純函數式語言就是解決編程問題的最佳方法。

FP 思想值得融入非 FP 語言，如 Python。Java 8 也從中吸收並支援了 FP。我們將在此章探討。


<!-- Old vs. New -->
## 新舊對比


通常，傳遞給方法的資料不同，結果不同。如果我們希望方法在呼叫時行為不同，該怎麼做呢？結論是：只要能將程式碼傳遞給方法，我們就可以控制它的行為。此前，我們透過在方法中建立包含所需行為的物件，然後將該物件傳遞給我們想要控制的方法來完成此操作。下面我們用傳統形式和 Java 8 的方法引用、Lambda 表達式分別示範。程式碼範例：

```java
// functional/Strategize.java

interface Strategy {
  String approach(String msg);
}

class Soft implements Strategy {
  public String approach(String msg) {
    return msg.toLowerCase() + "?";
  }
}

class Unrelated {
  static String twice(String msg) {
    return msg + " " + msg;
  }
}

public class Strategize {
  Strategy strategy;
  String msg;
  Strategize(String msg) {
    strategy = new Soft(); // [1]
    this.msg = msg;
  }

  void communicate() {
    System.out.println(strategy.approach(msg));
  }

  void changeStrategy(Strategy strategy) {
    this.strategy = strategy;
  }

  public static void main(String[] args) {
    Strategy[] strategies = {
      new Strategy() { // [2]
        public String approach(String msg) {
          return msg.toUpperCase() + "!";
        }
      },
      msg -> msg.substring(0, 5), // [3]
      Unrelated::twice // [4]
    };
    Strategize s = new Strategize("Hello there");
    s.communicate();
    for(Strategy newStrategy : strategies) {
      s.changeStrategy(newStrategy); // [5]
      s.communicate(); // [6]
    }
  }
}
```

輸出結果:

```
hello there?
HELLO THERE!
Hello
Hello there Hello there
```

**Strategy** 介面提供了單一的 `approach()` 方法來承載函數式功能。透過建立不同的 **Strategy** 物件，我們可以建立不同的行為。

我們一般透過建立一個實現**Strategy**介面的類來實現這種行為，正如在**Soft**裡所做的。

- **[1]** 在 **Strategize** 中，你可以看到 **Soft** 作為預設策略，在建構子中賦值。

- **[2]** 一種較為簡潔且更加自然的方法是建立一個**匿名內部類**。即便如此，仍有相當數量的冗餘程式碼。你總需要仔細觀察後才會發現：“哦，我明白了，原來這裡使用了匿名內部類。”

- **[3]** Java 8 的 Lambda 表達式，其參數和函數體被箭頭 `->` 分隔開。箭頭右側是從 Lambda 返回的表達式。它與單獨定義類和採用匿名內部類是等價的，但程式碼少得多。

- **[4]** Java 8 的**方法引用**，它以 `::` 為特徵。 `::` 的左邊是類或物件的名稱， `::` 的右邊是方法的名稱，但是沒有參數列表。

- **[5]** 在使用預設的 **Soft**  策略之後，我們逐步遍歷陣列中的所有 **Strategy**，並透過呼叫 `changeStrategy()` 方法將每個 **Strategy** 傳入變數 `s` 中。

- **[6]** 現在，每次呼叫 `communicate()` 都會產生不同的行為，具體取決於此刻正在使用的策略**程式碼物件**。我們傳遞的是行為，而並不僅僅是資料。[^3]

在 Java 8 之前，我們能夠透過 **[1]** 和 **[2]** 的方式傳遞功能。然而，這種語法的讀寫非常笨拙，並且我們別無選擇。方法引用和 Lambda 表達式的出現讓我們可以在需要時**傳遞功能**，而不是僅在必要時才這麼做。

<!-- Lambda Expressions -->

## Lambda表達式


Lambda 表達式是使用**最小可能**語法編寫的函數定義：

1. Lambda 表達式產生函數，而不是類。 雖然在 JVM（Java Virtual Machine，Java 虛擬機）上，一切都是類，但是幕後有各種操作執行讓 Lambda 看起來像函數 —— 作為程式設計師，你可以高興地假裝它們“就是函數”。

2. Lambda 語法儘可能少，這正是為了使 Lambda 易於編寫和使用。

我們在 **Strategize.java** 中看到了一個 Lambda 表達式，但還有其他語法變體：

```java
// functional/LambdaExpressions.java

interface Description {
  String brief();
}

interface Body {
  String detailed(String head);
}

interface Multi {
  String twoArg(String head, Double d);
}

public class LambdaExpressions {

  static Body bod = h -> h + " No Parens!"; // [1]

  static Body bod2 = (h) -> h + " More details"; // [2]

  static Description desc = () -> "Short info"; // [3]

  static Multi mult = (h, n) -> h + n; // [4]

  static Description moreLines = () -> { // [5]
    System.out.println("moreLines()");
    return "from moreLines()";
  };

  public static void main(String[] args) {
    System.out.println(bod.detailed("Oh!"));
    System.out.println(bod2.detailed("Hi!"));
    System.out.println(desc.brief());
    System.out.println(mult.twoArg("Pi! ", 3.14159));
    System.out.println(moreLines.brief());
  }
}
```

輸出結果：

```
Oh! No Parens!
Hi! More details
Short info
Pi! 3.14159
moreLines()
from moreLines()
```

我們從三個介面開始，每個介面都有一個單獨的方法（很快就會理解它的重要性）。但是，每個方法都有不同數量的參數，以便示範 Lambda 表達式語法。

任何 Lambda 表達式的基本語法是：

1. 參數。

2. 接著 `->`，可視為“產出”。

3. `->` 之後的內容都是方法體。

  - **[1]** 當只用一個參數，可以不需要括號 `()`。 然而，這是一個特例。

  - **[2]** 正常情況使用括號 `()` 包裹參數。 為了保持一致性，也可以使用括號 `()` 包裹單個參數，雖然這種情況並不常見。

  - **[3]** 如果沒有參數，則必須使用括號 `()` 表示空參數列表。

  - **[4]** 對於多個參數，將參數列表放在括號 `()` 中。

到目前為止，所有 Lambda 表達式方法體都是單行。 該表達式的結果自動成為 Lambda 表達式的返回值，在此處使用 **return** 關鍵字是非法的。 這是 Lambda 表達式簡化相應語法的另一種方式。

**[5]** 如果在 Lambda 表達式中確實需要多行，則必須將這些行放在花括號中。 在這種情況下，就需要使用 **return**。

Lambda 表達式通常比**匿名內部類**產生更易讀的程式碼，因此我們將在本書中儘可能使用它們。

### 遞迴

遞迴函數是一個自我呼叫的函數。可以編寫遞迴的 Lambda 表達式，但需要注意：遞迴方法必須是實例變數或靜態變數，否則會出現編譯時錯誤。 我們將為每個案例建立一個範例。

這兩個範例都需要一個接受 **int** 型參數並生成 **int** 的介面：

```java
// functional/IntCall.java

interface IntCall {
  int call(int arg);
}
```

整數 n 的階乘將所有小於或等於 n 的正整數相乘。 階乘函數是一個常見的遞迴範例：

```java
// functional/RecursiveFactorial.java

public class RecursiveFactorial {
  static IntCall fact;
  public static void main(String[] args) {
    fact = n -> n == 0 ? 1 : n * fact.call(n - 1);
    for(int i = 0; i <= 10; i++)
      System.out.println(fact.call(i));
  }
}
```

輸出結果：

```
1
1
2
6
24
120
720
5040
40320
362880
3628800
```

這裡，`fact` 是一個靜態變數。 注意使用三元 **if-else**。 遞迴函數將一直呼叫自己，直到 `i == 0`。所有遞迴函數都有“停止條件”，否則將無限遞迴併產生異常。

我們可以將 `Fibonacci` 序列用遞迴的 Lambda 表達式來實現，這次使用實例變數：

```java
// functional/RecursiveFibonacci.java

public class RecursiveFibonacci {
  IntCall fib;

  RecursiveFibonacci() {
    fib = n -> n == 0 ? 0 :
               n == 1 ? 1 :
               fib.call(n - 1) + fib.call(n - 2);
  }
  
  int fibonacci(int n) { return fib.call(n); }

  public static void main(String[] args) {
    RecursiveFibonacci rf = new RecursiveFibonacci();
    for(int i = 0; i <= 10; i++)
      System.out.println(rf.fibonacci(i));
  }
}
```

輸出結果：

```
0
1
1
2
3
5
8
13
21
34
55
```

將 `Fibonacci` 序列中的最後兩個元素求和來產生下一個元素。

<!-- method references-->

## 方法引用


Java 8 方法引用沒有歷史包袱。方法引用組成：類名或物件名，後面跟 `::` [^4]，然後跟方法名稱。

```java
// functional/MethodReferences.java

import java.util.*;

interface Callable { // [1]
  void call(String s);
}

class Describe {
  void show(String msg) { // [2]
    System.out.println(msg);
  }
}

public class MethodReferences {
  static void hello(String name) { // [3]
    System.out.println("Hello, " + name);
  }
  static class Description {
    String about;
    Description(String desc) { about = desc; }
    void help(String msg) { // [4]
      System.out.println(about + " " + msg);
    }
  }
  static class Helper {
    static void assist(String msg) { // [5]
      System.out.println(msg);
    }
  }
  public static void main(String[] args) {
    Describe d = new Describe();
    Callable c = d::show; // [6]
    c.call("call()"); // [7]

    c = MethodReferences::hello; // [8]
    c.call("Bob");

    c = new Description("valuable")::help; // [9]
    c.call("information");

    c = Helper::assist; // [10]
    c.call("Help!");
  }
}
```

輸出結果：

```
call()
Hello, Bob
valuable information
Help!
```

**[1]** 我們從單一方法介面開始（同樣，你很快就會了解到這一點的重要性）。

**[2]** `show()` 的簽名（參數類型和返回類型）符合 **Callable** 的 `call()` 的簽名。

**[3]** `hello()` 也符合 `call()` 的簽名。 

**[4]**  `help()` 也符合，它是靜態內部類中的非靜態方法。

**[5]** `assist()` 是靜態內部類中的靜態方法。

**[6]** 我們將 **Describe** 物件的方法引用賦值給 **Callable** ，它沒有 `show()` 方法，而是 `call()` 方法。 但是，Java 似乎接受用這個看似奇怪的賦值，因為方法引用符合 **Callable** 的 `call()` 方法的簽名。

**[7]** 我們現在可以透過呼叫 `call()` 來呼叫 `show()`，因為 Java 將 `call()` 映射到 `show()`。

**[8]** 這是一個**靜態**方法引用。

**[9]** 這是 **[6]** 的另一個版本：對已實例化物件的方法的引用，有時稱為*綁定方法引用*。

**[10]** 最後，獲取靜態內部類中靜態方法的引用與 **[8]** 中透過外部類引用相似。

上例只是簡短的介紹，我們很快就能看到方法引用的所有不同形式。

### Runnable介面

**Runnable** 介面自 1.0 版以來一直在 Java 中，因此不需要匯入。它也符合特殊的單方法介面格式：它的方法 `run()` 不帶參數，也沒有返回值。因此，我們可以使用 Lambda 表達式和方法引用作為 **Runnable**：

```java
// functional/RunnableMethodReference.java

// 方法引用與 Runnable 介面的結合使用

class Go {
  static void go() {
    System.out.println("Go::go()");
  }
}

public class RunnableMethodReference {
  public static void main(String[] args) {

    new Thread(new Runnable() {
      public void run() {
        System.out.println("Anonymous");
      }
    }).start();

    new Thread(
      () -> System.out.println("lambda")
    ).start();

    new Thread(Go::go).start();
  }
}
```

輸出結果：

```
Anonymous
lambda
Go::go()
```

**Thread** 物件將 **Runnable** 作為其建構子參數，並具有會呼叫 `run()` 的方法  `start()`。 注意這裡只有**匿名內部類**才要求顯式聲明 `run()` 方法。


<!-- Unbound Method References -->
### 未綁定的方法引用


未綁定的方法引用是指沒有關聯物件的普通（非靜態）方法。 使用未綁定的引用時，我們必須先提供物件：

```java
// functional/UnboundMethodReference.java

// 沒有方法引用的物件

class X {
  String f() { return "X::f()"; }
}

interface MakeString {
  String make();
}

interface TransformX {
  String transform(X x);
}

public class UnboundMethodReference {
  public static void main(String[] args) {
    // MakeString ms = X::f; // [1]
    TransformX sp = X::f;
    X x = new X();
    System.out.println(sp.transform(x)); // [2]
    System.out.println(x.f()); // 同等效果
  }
}
```

輸出結果：

```
X::f()
X::f()
```


到目前為止，我們已經見過了方法引用和對應介面的簽名（參數類型和返回類型）一致的幾個賦值例子。 在 **[1]** 中，我們嘗試同樣的做法，把 `X` 的 `f()` 方法引用賦值給 **MakeString**。結果即使 `make()` 與 `f()` 具有相同的簽名，編譯也會報“invalid method reference”（無效方法引用）錯誤。 問題在於，這裡其實還需要另一個隱藏參數參與：我們的老朋友 `this`。 你不能在沒有 `X` 物件的前提下呼叫 `f()`。 因此，`X :: f` 表示未綁定的方法引用，因為它尚未“綁定”到物件。

要解決這個問題，我們需要一個 `X` 物件，因此我們的介面實際上需要一個額外的參數，正如在 **TransformX** 中看到的那樣。 如果將 `X :: f` 賦值給 **TransformX**，在 Java 中是允許的。我們必須做第二個心理調整——使用未綁定的引用時，函數式方法的簽名（介面中的單個方法）不再與方法引用的簽名完全匹配。 原因是：你需要一個物件來呼叫方法。

**[2]** 的結果有點像腦筋急轉彎。我拿到未綁定的方法引用，並且呼叫它的`transform()`方法，將一個X類的物件傳遞給它，最後使得 `x.f()` 以某種方式被呼叫。Java知道它必須拿第一個參數，該參數實際就是`this` 物件，然後對此呼叫方法。

如果你的方法有更多個參數，就以第一個參數接受`this`的模式來處理。

```java
// functional/MultiUnbound.java

// 未綁定的方法與多參數的結合運用

class This {
  void two(int i, double d) {}
  void three(int i, double d, String s) {}
  void four(int i, double d, String s, char c) {}
}

interface TwoArgs {
  void call2(This athis, int i, double d);
}

interface ThreeArgs {
  void call3(This athis, int i, double d, String s);
}

interface FourArgs {
  void call4(
    This athis, int i, double d, String s, char c);
}

public class MultiUnbound {
  public static void main(String[] args) {
    TwoArgs twoargs = This::two;
    ThreeArgs threeargs = This::three;
    FourArgs fourargs = This::four;
    This athis = new This();
    twoargs.call2(athis, 11, 3.14);
    threeargs.call3(athis, 11, 3.14, "Three");
    fourargs.call4(athis, 11, 3.14, "Four", 'Z');
  }
}
```

需要指出的是，我將類命名為 **This**，並將函數式方法的第一個參數命名為 **athis**，但你在生產級程式碼中應該使用其他名字，以防止混淆。

### 建構子引用

你還可以捕獲建構子的引用，然後透過引用呼叫該建構子。

```java
// functional/CtorReference.java

class Dog {
  String name;
  int age = -1; // For "unknown"
  Dog() { name = "stray"; }
  Dog(String nm) { name = nm; }
  Dog(String nm, int yrs) { name = nm; age = yrs; }
}

interface MakeNoArgs {
  Dog make();
}

interface Make1Arg {
  Dog make(String nm);
}

interface Make2Args {
  Dog make(String nm, int age);
}

public class CtorReference {
  public static void main(String[] args) {
    MakeNoArgs mna = Dog::new; // [1]
    Make1Arg m1a = Dog::new;   // [2]
    Make2Args m2a = Dog::new;  // [3]

    Dog dn = mna.make();
    Dog d1 = m1a.make("Comet");
    Dog d2 = m2a.make("Ralph", 4);
  }
}
```

**Dog** 有三個建構子，函數式介面內的 `make()` 方法反映了建構子參數列表（ `make()` 方法名稱可以不同）。

**注意**我們如何對 **[1]**，**[2]** 和 **[3]** 中的每一個使用 `Dog :: new`。 這三個建構子只有一個相同名稱：`:: new`，但在每種情況下賦值給不同的介面，編譯器可以從中知道具體使用哪個建構子。

編譯器知道呼叫函數式方法（本例中為 `make()`）就相當於呼叫建構子。

<!-- Functional Interfaces -->
## 函數式介面


方法引用和 Lambda 表達式都必須被賦值，同時賦值需要類型訊息才能使編譯器保證類型的正確性。尤其是Lambda 表達式，它引入了新的要求。 程式碼範例：

```java
x -> x.toString()
```

我們清楚這裡返回類型必須是 **String**，但 `x` 是什麼類型呢？

Lambda 表達式包含 *類型推導* （編譯器會自動推匯出類型訊息，避免了程式設計師顯式地聲明）。編譯器必須能夠以某種方式推匯出 `x` 的類型。

下面是第二個程式碼範例：

```java
(x, y) -> x + y
```

現在 `x` 和 `y` 可以是任何支援 `+` 運算符連接的資料類型，可以是兩個不同的數值類型或者是 一個 **String** 加任意一種可自動轉換為 **String** 的資料類型（這包括了大多數類型）。 但是，當 Lambda 表達式被賦值時，編譯器必須確定 `x` 和 `y` 的確切類型以生成正確的程式碼。

該問題也適用於方法引用。 假設你要傳遞 `System.out :: println` 到你正在編寫的方法 ，你怎麼知道傳遞給方法的參數的類型？

為了解決這個問題，Java 8 引入了 `java.util.function` 包。它包含一組介面，這些介面是 Lambda 表達式和方法引用的目標類型。 每個介面只包含一個抽象方法，稱為 *函數式方法* 。

在編寫介面時，可以使用 `@FunctionalInterface` 註解強制執行此“函數式方法”模式：

```java
// functional/FunctionalAnnotation.java

@FunctionalInterface
interface Functional {
  String goodbye(String arg);
}

interface FunctionalNoAnn {
  String goodbye(String arg);
}

/*
@FunctionalInterface
interface NotFunctional {
  String goodbye(String arg);
  String hello(String arg);
}
產生錯誤訊息:
NotFunctional is not a functional interface
multiple non-overriding abstract methods
found in interface NotFunctional
*/

public class FunctionalAnnotation {
  public String goodbye(String arg) {
    return "Goodbye, " + arg;
  }
  public static void main(String[] args) {
    FunctionalAnnotation fa =
      new FunctionalAnnotation();
    Functional f = fa::goodbye;
    FunctionalNoAnn fna = fa::goodbye;
    // Functional fac = fa; // Incompatible
    Functional fl = a -> "Goodbye, " + a;
    FunctionalNoAnn fnal = a -> "Goodbye, " + a;
  }
}
```

`@FunctionalInterface` 註解是可選的; Java 會在 `main()` 中把 **Functional** 和 **FunctionalNoAnn** 都當作函數式介面來看待。 在 `NotFunctional` 的定義中可看出`@FunctionalInterface` 的作用：當介面中抽象方法多於一個時產生編譯期錯誤。

仔細觀察在定義 `f` 和 `fna` 時發生了什麼事。 `Functional` 和 `FunctionalNoAnn` 聲明了是介面，然而被賦值的只是方法 `goodbye()`。首先，這只是一個方法而不是類；其次，它甚至都不是實現了該介面的類中的方法。這是添加到Java 8中的一點小魔法：如果將方法引用或 Lambda 表達式賦值給函數式介面（類型需要匹配），Java 會適配你的賦值到目標介面。 編譯器會在後台把方法引用或 Lambda 表達式包裝進實現目標介面的類的實例中。

雖然 `FunctionalAnnotation` 確實符合 `Functional` 模型，但是 Java不允許我們像`fac`定義的那樣，將 `FunctionalAnnotation` 直接賦值給 `Functional`，因為 `FunctionalAnnotation` 並沒有顯式地去實現 `Functional` 介面。唯一的驚喜是，Java 8 允許我們將函數賦值給介面，這樣的語法更加簡單漂亮。

`java.util.function` 包旨在建立一組完整的目標介面，使得我們一般情況下不需再定義自己的介面。主要因為基本類型的存在，導致預定義的介面數量有少許增加。 如果你了解命名模式，顧名思義就能知道特定介面的作用。

 以下是基本命名準則：

1. 如果只處理物件而非基本類型，名稱則為 `Function`，`Consumer`，`Predicate` 等。參數類型透過泛型添加。

2. 如果接收的參數是基本類型，則由名稱的第一部分表示，如 `LongConsumer`，`DoubleFunction`，`IntPredicate` 等，但返回基本類型的 `Supplier` 介面例外。

3. 如果返回值為基本類型，則用 `To` 表示，如 `ToLongFunction <T>` 和 `IntToLongFunction`。

4. 如果返回值類型與參數類型相同，則是一個 `Operator` ：單個參數使用 `UnaryOperator`，兩個參數使用 `BinaryOperator`。

5. 如果接收參數並返回一個布林值，則是一個 **謂詞** (`Predicate`)。

6. 如果接收的兩個參數類型不同，則名稱中有一個 `Bi`。

下表描述了 `java.util.function` 中的目標類型（包括例外情況）：

| **特徵** |**函數式方法名**|**範例**|
| :---- | :----: | :----: |
|無參數； <br> 無返回值|**Runnable** <br> (java.lang)  <br>  `run()`|**Runnable**|
|無參數； <br> 返回類型任意|**Supplier** <br> `get()` <br> `getAs類型()`| **Supplier`<T>`  <br> BooleanSupplier  <br> IntSupplier  <br> LongSupplier  <br> DoubleSupplier**|
|無參數； <br> 返回類型任意|**Callable** <br> (java.util.concurrent)  <br> `call()`|**Callable`<V>`**|
|1 參數； <br> 無返回值|**Consumer** <br> `accept()`|**`Consumer<T>` <br> IntConsumer <br> LongConsumer <br> DoubleConsumer**|
|2 參數 **Consumer**|**BiConsumer** <br> `accept()`|**`BiConsumer<T,U>`**|
|2 參數 **Consumer**； <br> 第一個參數是 引用； <br> 第二個參數是 基本類型|**Obj類型Consumer** <br> `accept()`|**`ObjIntConsumer<T>` <br> `ObjLongConsumer<T>` <br> `ObjDoubleConsumer<T>`**|
|1 參數； <br> 返回類型不同|**Function** <br> `apply()` <br> **To類型** 和 **類型To類型** <br> `applyAs類型()`|**Function`<T,R>` <br> IntFunction`<R>` <br> `LongFunction<R>` <br> DoubleFunction`<R>` <br> ToIntFunction`<T>` <br> `ToLongFunction<T>` <br> `ToDoubleFunction<T>` <br> IntToLongFunction <br> IntToDoubleFunction <br> LongToIntFunction <br> LongToDoubleFunction <br> DoubleToIntFunction <br> DoubleToLongFunction**|
|1 參數； <br> 返回類型相同|**UnaryOperator** <br> `apply()`|**`UnaryOperator<T>` <br> IntUnaryOperator <br> LongUnaryOperator <br> DoubleUnaryOperator**|
|2 參數，類型相同； <br> 返回類型相同|**BinaryOperator** <br> `apply()`|**`BinaryOperator<T>` <br> IntBinaryOperator <br> LongBinaryOperator <br> DoubleBinaryOperator**|
|2 參數，類型相同; <br> 返回整型|Comparator <br> (java.util) <br> `compare()`|**`Comparator<T>`**|
|2 參數； <br> 返回布爾型|**Predicate** <br> `test()`|**`Predicate<T>` <br> `BiPredicate<T,U>` <br> IntPredicate <br> LongPredicate <br> DoublePredicate**|
|參數基本類型； <br> 返回基本類型|**類型To類型Function** <br> `applyAs類型()`|**IntToLongFunction <br> IntToDoubleFunction <br> LongToIntFunction <br> LongToDoubleFunction <br> DoubleToIntFunction <br> DoubleToLongFunction**|
|2 參數； <br>類型不同|**Bi操作** <br> (不同方法名)|**`BiFunction<T,U,R>` <br> `BiConsumer<T,U>` <br> `BiPredicate<T,U>` <br> `ToIntBiFunction<T,U>` <br> `ToLongBiFunction<T,U>` <br> `ToDoubleBiFunction<T>`**|


此表僅提供些一般方案。透過上表，你應該或多或少能自行推匯出你所需要的函數式介面。

可以看出，在建立 `java.util.function` 時，設計者們做出了一些選擇。 

例如，為什麼沒有 `IntComparator`，`LongComparator` 和 `DoubleComparator` 呢？有 `BooleanSupplier` 卻沒有其他表示 **Boolean** 的介面；有通用的 `BiConsumer` 卻沒有用於 **int**，**long** 和 **double** 的 `BiConsumers` 變體（我理解他們為什麼放棄這些介面）。這到底是疏忽還是有人認為其他組合使用得很少呢（他們是如何得出這個結論的）？

你還可以看到基本類型給 Java 添加了多少複雜性。該語言的第一版中就包含了基本類型，原因是考慮效率問題（該問題很快就紓解了）。現在，在語言的生命週期裡，我們一直忍受語言設計的糟糕選擇所帶來的影響。

下面列舉了基於 Lambda 表達式的所有不同 **Function** 變體的範例：

```java
// functional/FunctionVariants.java

import java.util.function.*;

class Foo {}

class Bar {
  Foo f;
  Bar(Foo f) { this.f = f; }
}

class IBaz {
  int i;
  IBaz(int i) {
    this.i = i;
  }
}

class LBaz {
  long l;
  LBaz(long l) {
    this.l = l;
  }
}

class DBaz {
  double d;
  DBaz(double d) {
    this.d = d;
  }
}

public class FunctionVariants {
  static Function<Foo,Bar> f1 = f -> new Bar(f);
  static IntFunction<IBaz> f2 = i -> new IBaz(i);
  static LongFunction<LBaz> f3 = l -> new LBaz(l);
  static DoubleFunction<DBaz> f4 = d -> new DBaz(d);
  static ToIntFunction<IBaz> f5 = ib -> ib.i;
  static ToLongFunction<LBaz> f6 = lb -> lb.l;
  static ToDoubleFunction<DBaz> f7 = db -> db.d;
  static IntToLongFunction f8 = i -> i;
  static IntToDoubleFunction f9 = i -> i;
  static LongToIntFunction f10 = l -> (int)l;
  static LongToDoubleFunction f11 = l -> l;
  static DoubleToIntFunction f12 = d -> (int)d;
  static DoubleToLongFunction f13 = d -> (long)d;

  public static void main(String[] args) {
    Bar b = f1.apply(new Foo());
    IBaz ib = f2.apply(11);
    LBaz lb = f3.apply(11);
    DBaz db = f4.apply(11);
    int i = f5.applyAsInt(ib);
    long l = f6.applyAsLong(lb);
    double d = f7.applyAsDouble(db);
    l = f8.applyAsLong(12);
    d = f9.applyAsDouble(12);
    i = f10.applyAsInt(12);
    d = f11.applyAsDouble(12);
    i = f12.applyAsInt(13.0);
    l = f13.applyAsLong(13.0);
  }
}
```

這些 Lambda 表達式嘗試生成適合函數簽名的最簡程式碼。 在某些情況下有必要進行強制類型轉換，否則編譯器會報截斷錯誤。

`main()`中的每個測試都顯示了 `Function` 介面中不同類型的 `apply()` 方法。 每個都產生一個與其關聯的 Lambda 表達式的呼叫。

方法引用有自己的小魔法：

```java
/ functional/MethodConversion.java

import java.util.function.*;

class In1 {}
class In2 {}

public class MethodConversion {
  static void accept(In1 i1, In2 i2) {
    System.out.println("accept()");
  }
  static void someOtherName(In1 i1, In2 i2) {
    System.out.println("someOtherName()");
  }
  public static void main(String[] args) {
    BiConsumer<In1,In2> bic;

    bic = MethodConversion::accept;
    bic.accept(new In1(), new In2());

    bic = MethodConversion::someOtherName;
    // bic.someOtherName(new In1(), new In2()); // Nope
    bic.accept(new In1(), new In2());
  }
}
```

輸出結果：

```
accept()
someOtherName()
```

查看 `BiConsumer` 的文件，你會看到它的函數式方法為 `accept()` 。 的確，如果我們將方法命名為 `accept()`，它就可以作為方法引用。 但是我們也可用不同的名稱，比如 `someOtherName()`。只要參數類型、返回類型與 `BiConsumer` 的 `accept()` 相同即可。

因此，在使用函數介面時，名稱無關緊要——只要參數類型和返回類型相同。 Java 會將你的方法映射到介面方法。 要呼叫方法，可以呼叫介面的函數式方法名（在本例中為 `accept()`），而不是你的方法名。

現在我們來看看，將方法引用應用於基於類的函數式介面（即那些不包含基本類型的函數式介面）。下面的例子中，我建立了適合函數式方法簽名的最簡單的方法：

```java
// functional/ClassFunctionals.java

import java.util.*;
import java.util.function.*;

class AA {}
class BB {}
class CC {}

public class ClassFunctionals {
  static AA f1() { return new AA(); }
  static int f2(AA aa1, AA aa2) { return 1; }
  static void f3(AA aa) {}
  static void f4(AA aa, BB bb) {}
  static CC f5(AA aa) { return new CC(); }
  static CC f6(AA aa, BB bb) { return new CC(); }
  static boolean f7(AA aa) { return true; }
  static boolean f8(AA aa, BB bb) { return true; }
  static AA f9(AA aa) { return new AA(); }
  static AA f10(AA aa1, AA aa2) { return new AA(); }
  public static void main(String[] args) {
    Supplier<AA> s = ClassFunctionals::f1;
    s.get();
    Comparator<AA> c = ClassFunctionals::f2;
    c.compare(new AA(), new AA());
    Consumer<AA> cons = ClassFunctionals::f3;
    cons.accept(new AA());
    BiConsumer<AA,BB> bicons = ClassFunctionals::f4;
    bicons.accept(new AA(), new BB());
    Function<AA,CC> f = ClassFunctionals::f5;
    CC cc = f.apply(new AA());
    BiFunction<AA,BB,CC> bif = ClassFunctionals::f6;
    cc = bif.apply(new AA(), new BB());
    Predicate<AA> p = ClassFunctionals::f7;
    boolean result = p.test(new AA());
    BiPredicate<AA,BB> bip = ClassFunctionals::f8;
    result = bip.test(new AA(), new BB());
    UnaryOperator<AA> uo = ClassFunctionals::f9;
    AA aa = uo.apply(new AA());
    BinaryOperator<AA> bo = ClassFunctionals::f10;
    aa = bo.apply(new AA(), new AA());
  }
}
```

請**注意**，每個方法名稱都是隨意的（如 `f1()`，`f2()`等）。正如你剛才看到的，一旦將方法引用賦值給函數介面，我們就可以呼叫與該介面關聯的函數方法。 在此範例中為 `get()`、`compare()`、`accept()`、`apply()` 和 `test()`。

<!-- Functional Interfaces with More Arguments -->

### 多參數函數式介面

`java.util.functional` 中的介面是有限的。比如有 `BiFunction`，但也僅此而已。 如果需要三參數函數的介面怎麼辦？ 其實這些介面非常簡單，很容易查看 Java 庫原始碼並自行建立。程式碼範例：

```java
// functional/TriFunction.java

@FunctionalInterface
public interface TriFunction<T, U, V, R> {
    R apply(T t, U u, V v);
}
```

簡單測試，驗證它是否有效：

```java
// functional/TriFunctionTest.java

public class TriFunctionTest {
  static int f(int i, long l, double d) { return 99; }
  public static void main(String[] args) {
    TriFunction<Integer, Long, Double, Integer> tf =
      TriFunctionTest::f;
    tf = (i, l, d) -> 12;
  }
}
```

這裡我們同時測試了方法引用和 Lambda 表達式。

### 缺少基本類型的函數

讓我們重溫一下 `BiConsumer`，看看我們將如何建立各種缺失的預定義組合，涉及 **int**，**long** 和 **double** （基本類型）：

```java
// functional/BiConsumerPermutations.java

import java.util.function.*;

public class BiConsumerPermutations {
  static BiConsumer<Integer, Double> bicid = (i, d) ->
    System.out.format("%d, %f%n", i, d);
  static BiConsumer<Double, Integer> bicdi = (d, i) ->
    System.out.format("%d, %f%n", i, d);
  static BiConsumer<Integer, Long> bicil = (i, l) ->
    System.out.format("%d, %d%n", i, l);
  public static void main(String[] args) {
    bicid.accept(47, 11.34);
    bicdi.accept(22.45, 92);
    bicil.accept(1, 11L);
  }
}
```

輸出結果：

```
47, 11.340000
92, 22.450000
1, 11
```

這裡使用 `System.out.format()` 來顯示。它類似於 `System.out.println()` 但提供了更多的顯示選項。 這裡，`%f` 表示我將 `n` 作為浮點值給出，`%d` 表示 `n` 是一個整數值。 這其中可以包含空格，輸入 `%n` 會換行 — 當然使用傳統的 `\n` 也能換行，但 `%n` 是自動跨平台的，這是使用 `format()` 的另一個原因。

上例只是簡單使用了合適的包裝類型，而裝箱和拆箱負責它與基本類型之間的來迴轉換。 又比如，我們可以將包裝類型和`Function`一起使用，而不去用各種針對基本類型的預定義介面。程式碼範例：

```java
// functional/FunctionWithWrapped.java

import java.util.function.*;

public class FunctionWithWrapped {
  public static void main(String[] args) {
    Function<Integer, Double> fid = i -> (double)i;
    IntToDoubleFunction fid2 = i -> i;
  }
}
```

如果沒有強制轉換，則會收到錯誤消息：“Integer cannot be converted to Double”（**Integer** 無法轉換為 **Double**），而使用 **IntToDoubleFunction** 就沒有此類問題。 **IntToDoubleFunction** 介面的原始碼是這樣的：

```java
@FunctionalInterface 
public interface IntToDoubleFunction { 
  double applyAsDouble(int value); 
}
```

因為我們可以簡單地寫 `Function <Integer，Double>` 並產生正常的結果，所以用基本類型（`IntToDoubleFunction`）的唯一理由是可以避免傳遞參數和返回結果過程中的自動拆裝箱，進而提升性能。

似乎是考慮到使用頻率，某些函數類型並沒有預定義。

當然，如果因為缺少針對基本類型的函數式介面造成了性能問題，你可以輕鬆編寫自己的介面（ 參考 Java 原始碼）——儘管這裡出現性能瓶頸的可能性不大。

<!-- Higher-Order Functions-->
## 高階函數


這個名字可能聽起來令人生畏，但是：[高階函數](https://en.wikipedia.org/wiki/Higher-order_function)（Higher-order Function）只是一個消費或產生函數的函數。

我們先來看看如何產生一個函數：

```java
// functional/ProduceFunction.java

import java.util.function.*;

interface
FuncSS extends Function<String, String> {} // [1]

public class ProduceFunction {
  static FuncSS produce() {
    return s -> s.toLowerCase(); // [2]
  }
  public static void main(String[] args) {
    FuncSS f = produce();
    System.out.println(f.apply("YELLING"));
  }
}
```

輸出結果：
```
yelling
```

這裡，`produce()` 是高階函數。

**[1]** 使用繼承，可以輕鬆地為專用介面建立別名。

**[2]** 使用 Lambda 表達式，可以輕鬆地在方法中建立和返回一個函數。

要消費一個函數，消費函數需要在參數列表正確地描述函數類型。程式碼範例：

```java
// functional/ConsumeFunction.java

import java.util.function.*;

class One {}
class Two {}

public class ConsumeFunction {
  static Two consume(Function<One,Two> onetwo) {
    return onetwo.apply(new One());
  }
  public static void main(String[] args) {
    Two two = consume(one -> new Two());
  }
}
```

當基於消費函數生成新函數時，事情就變得相當有趣了。程式碼範例如下：

```java
// functional/TransformFunction.java

import java.util.function.*;

class I {
  @Override
  public String toString() { return "I"; }
}

class O {
  @Override
  public String toString() { return "O"; }
}

public class TransformFunction {
  static Function<I,O> transform(Function<I,O> in) {
    return in.andThen(o -> {
      System.out.println(o);
      return o;
    });
  }
  public static void main(String[] args) {
    Function<I,O> f2 = transform(i -> {
      System.out.println(i);
      return new O();
    });
    O o = f2.apply(new I());
  }
}
```

輸出結果：

```
I
O
```

在這裡，`transform()` 生成一個與傳入的函數具有相同簽名的函數，但是你可以生成任何你想要的類型。

這裡使用到了 `Function` 介面中名為 `andThen()` 的預設方法，該方法專門用於操作函數。 顧名思義，在呼叫 `in` 函數之後呼叫 `andThen()`（還有個 `compose()` 方法，它在 `in` 函數之前應用新函數）。 要附加一個 `andThen()` 函數，我們只需將該函數作為參數傳遞。 `transform()` 產生的是一個新函數，它將 `in` 的動作與 `andThen()` 參數的動作結合起來。

<!-- Closures -->

## 閉包


在上一節的 `ProduceFunction.java` 中，我們從方法中返回 Lambda 函數。 雖然過程簡單，但是有些問題必須再回過頭來探討一下。

**閉包**（Closure）一詞總結了這些問題。 它非常重要，利用閉包可以輕鬆生成函數。

考慮一個更複雜的 Lambda，它使用函數作用域之外的變數。 返回該函數會發生什麼事？ 也就是說，當你呼叫函數時，它對那些 “外部 ”變數引用了什麼?  如果語言不能自動解決，那問題將變得非常棘手。 能夠解決這個問題的語言被稱作 *支援閉包*，或者稱作 *詞法定界*（*lexically scoped* ，基於詞法作用域的）( 也有用術語 *變數捕獲* *variable capture* 稱呼的)。Java 8 提供了有限但合理的閉包支援，我們將用一些簡單的例子來研究它。

首先，下列方法返回一個函數，該函數訪問物件欄位和方法參數：

```java
// functional/Closure1.java

import java.util.function.*;

public class Closure1 {
  int i;
  IntSupplier makeFun(int x) {
    return () -> x + i++;
  }
}
```

但是，仔細考慮一下，`i` 的這種用法並非是個大難題，因為物件很可能在你呼叫 `makeFun()` 之後就存在了——實際上，垃圾收集器幾乎肯定會保留以這種方式被綁定到現存函數的物件[^5]。當然，如果你對同一個物件多次呼叫 `makeFun()` ，你最終會得到多個函數，它們共享 `i` 的儲存空間：
```java
// functional/SharedStorage.java

import java.util.function.*;

public class SharedStorage {
  public static void main(String[] args) {
    Closure1 c1 = new Closure1();
    IntSupplier f1 = c1.makeFun(0);
    IntSupplier f2 = c1.makeFun(0);
    IntSupplier f3 = c1.makeFun(0);
    System.out.println(f1.getAsInt());
    System.out.println(f2.getAsInt());
    System.out.println(f3.getAsInt());
  }
}
```

輸出結果：

```
0
1
2
```

每次呼叫 `getAsInt()` 都會增加 `i`，表明儲存是共享的。

如果 `i` 是 `makeFun()` 的局部變數怎麼辦？ 在正常情況下，當 `makeFun()` 完成時 `i` 就消失。 但它仍可以編譯：

```java
// functional/Closure2.java

import java.util.function.*;

public class Closure2 {
  IntSupplier makeFun(int x) {
    int i = 0;
    return () -> x + i;
  }
}
```

由 `makeFun()` 返回的 `IntSupplier` “關住了” `i` 和 `x`，因此即使`makeFun()`已執行完畢，當你呼叫返回的函數時`i` 和 `x`仍然有效，而不是像正常情況下那樣在 `makeFun()` 執行後 `i` 和`x`就消失了。 但請注意，我沒有像 `Closure1.java` 那樣遞增 `i`，因為會產生編譯時錯誤。程式碼範例：

```java
// functional/Closure3.java

// {WillNotCompile}
import java.util.function.*;

public class Closure3 {
  IntSupplier makeFun(int x) {
    int i = 0;
    // x++ 和 i++ 都會報錯：
    return () -> x++ + i++;
  }
}
```

`x` 和 `i` 的操作都犯了同樣的錯誤：被 Lambda 表達式引用的局部變數必須是 `final` 或者是等同 `final` 效果的。

如果使用 `final` 修飾 `x`和 `i`，就不能再遞增它們的值了。程式碼範例：

```java
// functional/Closure4.java

import java.util.function.*;

public class Closure4 {
  IntSupplier makeFun(final int x) {
    final int i = 0;
    return () -> x + i;
  }
}
```

那麼為什麼在 `Closure2.java` 中， `x` 和 `i` 非 `final` 卻可以執行呢？

這就叫做**等同 final 效果**（Effectively Final）。這個術語是在 Java 8 才開始出現的，表示雖然沒有明確地聲明變數是 `final` 的，但是因變數值沒被改變過而實際有了 `final` 同等的效果。 如果局部變數的初始值永遠不會改變，那麼它實際上就是 `final` 的。

如果 `x` 和 `i` 的值在方法中的其他位置發生改變（但不在返回的函數內部），則編譯器仍將視其為錯誤。每個遞增操作則會分別產生錯誤消息。程式碼範例：

```java
// functional/Closure5.java

// {無法編譯成功}
import java.util.function.*;

public class Closure5 {
  IntSupplier makeFun(int x) {
    int i = 0;
    i++;
    x++;
    return () -> x + i;
  }
}
```

**等同 final 效果**意味著可以在變數聲明前加上 **final** 關鍵字而不用更改任何其餘程式碼。 實際上它就是具備 `final` 效果的，只是沒有明確說明。

在閉包中，在使用 `x` 和 `i` 之前，透過將它們賦值給 `final` 修飾的變數，我們解決了 `Closure5.java` 中遇到的問題。程式碼範例：

```java

// functional/Closure6.java

import java.util.function.*;

public class Closure6 {
  IntSupplier makeFun(int x) {
    int i = 0;
    i++;
    x++;
    final int iFinal = i;
    final int xFinal = x;
    return () -> xFinal + iFinal;
  }
}
```

上例中 `iFinal` 和 `xFinal` 的值在賦值後並沒有改變過，因此在這裡使用 `final` 是多餘的。

如果改用包裝類型會是什麼情況呢？我們可以把`int`類型改為`Integer`類型研究一下：

```java
// functional/Closure7.java

// {無法編譯成功}
import java.util.function.*;

public class Closure7 {
  IntSupplier makeFun(int x) {
    Integer i = 0;
    i = i + 1;
    return () -> x + i;
  }
}
```

編譯器非常聰明地識別到變數 `i` 的值被更改過。 包裝類型可能是被特殊處理了，我們再嘗試一下 **List**：

```java
// functional/Closure8.java

import java.util.*;
import java.util.function.*;

public class Closure8 {
  Supplier<List<Integer>> makeFun() {
    final List<Integer> ai = new ArrayList<>();
    ai.add(1);
    return () -> ai;
  }
  public static void main(String[] args) {
    Closure8 c7 = new Closure8();
    List<Integer>
      l1 = c7.makeFun().get(),
      l2 = c7.makeFun().get();
    System.out.println(l1);
    System.out.println(l2);
    l1.add(42);
    l2.add(96);
    System.out.println(l1);
    System.out.println(l2);
  }
}
```

輸出結果：

```
[1]
[1]
[1, 42]
[1, 96]
```

可以看到，這次一切正常。我們改變了 **List** 的內容卻沒產生編譯時錯誤。透過觀察本例的輸出結果，我們發現這看起來非常安全。這是因為每次呼叫 `makeFun()` 時，其實都會建立並返回一個全新而非共享的 `ArrayList`。也就是說，每個閉包都有自己獨立的 `ArrayList`，它們之間互不干擾。

請**注意**我已經聲明 `ai` 是 `final` 的了。儘管在這個例子中你可以去掉 `final` 並得到相同的結果（試試吧！）。 應用於物件引用的 `final` 關鍵字僅表示不會重新賦值引用。 它並不代表你不能修改物件本身。

我們來看看 `Closure7.java` 和 `Closure8.java` 之間的區別。我們看到：在 `Closure7.java` 中變數 `i` 有過重新賦值。 也許這就是觸發**等同 final 效果**錯誤消息的原因。

```java
// functional/Closure9.java

// {無法編譯成功}
import java.util.*;
import java.util.function.*;

public class Closure9 {
  Supplier<List<Integer>> makeFun() {
    List<Integer> ai = new ArrayList<>();
    ai = new ArrayList<>(); // Reassignment
    return () -> ai;
  }
}
```

上例，重新賦值引用會觸發錯誤消息。如果只修改指向的物件則沒問題，只要沒有其他人獲得對該物件的引用（這意味著你有多個實體可以修改物件，此時事情會變得非常混亂），基本上就是安全的[^6]。

讓我們回顧一下 `Closure1.java`。那麼現在問題來了：為什麼變數 `i` 被修改編譯器卻沒有報錯呢。 它既不是 `final` 的，也不是**等同 final 效果**的。因為 `i` 是外部類的成員，所以這樣做肯定是安全的（除非你正在建立共享可變記憶體的多個函數）。是的，你可以辯稱在這種情況下不會發生變數捕獲（Variable Capture）。但可以肯定的是，`Closure3.java` 的錯誤消息是專門針對局部變數的。因此，規則並非只是 “在 Lambda 之外定義的任何變數必須是 `final` 的或**等同 final 效果**” 那麼簡單。相反，你必須考慮捕獲的變數是否是**等同 final 效果**的。 如果它是物件中的欄位（實例變數），那麼它有獨立的生命週期，不需要任何特殊的捕獲以便稍後在呼叫 Lambda 時存在。（註：結論是——Lambda 可以沒有限制地引用 實例變數和靜態變數。但 局部變數必須顯式聲明為final，或事實上是final 。）

<!-- Inner Classes as Closures -->

### 作為閉包的內部類

我們可以使用匿名內部類重寫之前的例子:

```java
// functional/AnonymousClosure.java

import java.util.function.*;

public class AnonymousClosure {
  IntSupplier makeFun(int x) {
    int i = 0;
    // 同樣規則的應用:
    // i++; // 非等同 final 效果
    // x++; // 同上
    return new IntSupplier() {
      public int getAsInt() { return x + i; }
    };
  }
}
```

實際上只要有內部類，就會有閉包（Java 8 只是簡化了閉包操作）。在 Java 8 之前，變數 `x` 和 `i` 必須被明確聲明為 `final`。在 Java 8 中，內部類的規則放寬，包括**等同 final 效果**。

<!-- Function Composition -->
## 函陣列合


函陣列合（Function Composition）意為“多個函陣列合成新函數”。它通常是函數式編程的基本組成部分。在前面的 `TransformFunction.java` 類中，就有一個使用 `andThen()` 的函陣列合範例。一些 `java.util.function` 介面中包含支援函陣列合的方法 [^7]。

| 組合方法 | 支援介面 |
| :----- | :----- |
| `andThen(argument)` <br> 執行原操作,再執行參數操作 | **Function <br> BiFunction <br> Consumer <br> BiConsumer <br> IntConsumer <br> LongConsumer <br> DoubleConsumer <br> UnaryOperator <br> IntUnaryOperator <br> LongUnaryOperator <br> DoubleUnaryOperator <br> BinaryOperator** |
| `compose(argument)` <br> 執行參數操作,再執行原操作 | **Function <br> UnaryOperator <br> IntUnaryOperator <br> LongUnaryOperator <br> DoubleUnaryOperator** |
| `and(argument)`  <br> 原謂詞(Predicate)和參數謂詞的短路**邏輯與** | **Predicate <br> BiPredicate <br> IntPredicate <br> LongPredicate <br> DoublePredicate** |
| `or(argument)` <br> 原謂詞和參數謂詞的短路**邏輯或** | **Predicate <br> BiPredicate <br> IntPredicate <br> LongPredicate <br> DoublePredicate** |
| `negate()` <br> 該謂詞的**邏輯非**| **Predicate <br> BiPredicate <br> IntPredicate <br> LongPredicate <br> DoublePredicate** |


下例使用了 `Function` 裡的 `compose()`和 `andThen()`。程式碼範例：

```java
// functional/FunctionComposition.java

import java.util.function.*;

public class FunctionComposition {
  static Function<String, String>
    f1 = s -> {
      System.out.println(s);
      return s.replace('A', '_');
    },
    f2 = s -> s.substring(3),
    f3 = s -> s.toLowerCase(),
    f4 = f1.compose(f2).andThen(f3);
  public static void main(String[] args) {
    System.out.println(
      f4.apply("GO AFTER ALL AMBULANCES"));
  }
}
```

輸出結果：

```
AFTER ALL AMBULANCES
_fter _ll _mbul_nces
```

這裡我們重點看正在建立的新函數 `f4`。它呼叫 `apply()` 的方式與一般幾乎無異[^8]。

當 `f1` 獲得字串時，它已經被`f2` 剝離了前三個字元。這是因為 `compose（f2）` 表示 `f2` 的呼叫發生在 `f1` 之前。

下例是 謂詞(`Predicate`) 的邏輯運算示範.程式碼範例：

```java
// functional/PredicateComposition.java

import java.util.function.*;
import java.util.stream.*;

public class PredicateComposition {
  static Predicate<String>
    p1 = s -> s.contains("bar"),
    p2 = s -> s.length() < 5,
    p3 = s -> s.contains("foo"),
    p4 = p1.negate().and(p2).or(p3);
  public static void main(String[] args) {
    Stream.of("bar", "foobar", "foobaz", "fongopuckey")
      .filter(p4)
      .forEach(System.out::println);
  }
}
```

輸出結果：

```
foobar
foobaz
```

`p4` 獲取到了所有謂詞(`Predicate`)並組合成一個更複雜的謂詞。解讀：如果字串中不包含 `bar` 且長度小於 5，或者它包含 `foo` ，則結果為 `true`。

正因它產生如此清晰的語法，我在主方法中採用了一些小技巧，並借用了下一章的內容。首先，我建立了一個字串物件的流，然後將每個物件傳遞給 `filter()` 操作。 `filter()` 使用 `p4` 的謂詞來確定物件的去留。最後我們使用 `forEach()` 將 `println` 方法引用應用在每個留存的物件上。

從輸出結果我們可以看到 `p4` 的工作流程：任何帶有 `"foo"` 的字串都得以保留，即使它的長度大於 5。 `"fongopuckey"` 因長度超出且不包含 `foo` 而被丟棄。

<!-- Currying and  Partial Evaluation -->

## 柯里化和部分求值

[柯里化](https://en.wikipedia.org/wiki/Currying)（Currying）的名稱來自於其發明者之一 *Haskell Curry*。他可能是電腦領域唯一姓氏和名字都命名過重要概念的人（另外就是 Haskell 程式語言）。 柯里化意為：將一個多參數的函數，轉換為一系列單參數函數。

```java
// functional/CurryingAndPartials.java

import java.util.function.*;

public class CurryingAndPartials {
   // 未柯里化:
   static String uncurried(String a, String b) {
      return a + b;
   }
   public static void main(String[] args) {
      // 柯里化的函數:
      Function<String, Function<String, String>> sum =
         a -> b -> a + b; // [1]

      System.out.println(uncurried("Hi ", "Ho"));

      Function<String, String>
        hi = sum.apply("Hi "); // [2]
      System.out.println(hi.apply("Ho"));

      // 部分應用:
      Function<String, String> sumHi =
        sum.apply("Hup ");
      System.out.println(sumHi.apply("Ho"));
      System.out.println(sumHi.apply("Hey"));
   }
}
```

輸出結果：

```
Hi Ho
Hi Ho
Hup Ho
Hup Hey
```

**[1]** 這一連串的箭頭很巧妙。*注意*，在函數介面聲明中，第二個參數是另一個函數。

**[2]** 柯里化的目的是能夠透過提供一個參數來建立一個新函數，所以現在有了一個“帶參函數”和剩下的 “自由函數”（free argumnet） 。實際上，你從一個雙參數函數開始，最後得到一個單參數函數。

我們可以透過添加級別來柯里化一個三參數函數：

```java
// functional/Curry3Args.java

import java.util.function.*;

public class Curry3Args {
   public static void main(String[] args) {
      Function<String,
        Function<String,
          Function<String, String>>> sum =
            a -> b -> c -> a + b + c;
      Function<String,
        Function<String, String>> hi =
          sum.apply("Hi ");
      Function<String, String> ho =
        hi.apply("Ho ");
      System.out.println(ho.apply("Hup"));
   }
}
```

輸出結果：

```
Hi Ho Hup
```

對於每個級別的箭頭級聯（Arrow-cascading），你都要在類型聲明中包裹另一層 **Function**。

處理基本類型和裝箱時，請使用適當的函數式介面：

```java
// functional/CurriedIntAdd.java

import java.util.function.*;

public class CurriedIntAdd {
  public static void main(String[] args) {
    IntFunction<IntUnaryOperator>
      curriedIntAdd = a -> b -> a + b;
    IntUnaryOperator add4 = curriedIntAdd.apply(4);
    System.out.println(add4.applyAsInt(5));
	  }
}
```

輸出結果：

```
9
```

可以在網際網路上找到更多的柯里化範例。通常它們是用 Java 之外的語言實現的，但如果理解了柯里化的基本概念，你可以很輕鬆地用 Java 實現它們。

<!-- Pure Functional Programming -->
## 純函數式編程


即使沒有函數式支援，像 C 這樣的基礎語言，也可以按照一定的原則編寫純函數式程式。Java 8 讓函數式編程更簡單，不過我們要確保一切是 `final` 的，同時你的所有方法和函數沒有副作用。因為 Java 在本質上並非是不可變語言，所以編譯器對我們犯的錯誤將無能為力。

這種情況下，我們可以借助第三方工具[^9]，但使用 Scala 或 Clojure 這樣的語言可能更簡單。因為它們從一開始就是為保持不變性而設計的。你可以採用這些語言來編寫你的 Java 項目的一部分。如果必須要用純函數式編寫，則可以用 Scala（需要遵循一些規則） 或 Clojure （遵循的規則更少）。雖然 Java 支援[並發編程](./24-Concurrent-Programming.md)，但如果這是你項目的核心部分，你應該考慮在項目部分功能中使用 `Scala` 或 `Clojure` 之類的語言。

<!-- Summary -->
## 本章小結


Lambda 表達式和方法引用並沒有將 Java 轉換成函數式語言，而是提供了對函數式編程的支援。這對 Java 來說是一個巨大的改進。因為這允許你編寫更簡潔明瞭，易於理解的程式碼。在下一章中，你會看到它們在流式編程中的應用。相信你會像我一樣，喜歡上流式編程。

這些特性滿足了很多羨慕Clojure、Scala 這類更函數化語言的程式設計師，並且阻止了Java程式設計師轉向那些更函數化的語言（就算不能阻止，起碼提供了更好的選擇）。

但是，Lambdas 和方法引用遠非完美，我們永遠要為 Java 設計者早期的草率決定付出代價。特別是沒有泛型 Lambda，所以 Lambda 在 Java 中並非一等公民。雖然我不否認 Java 8 的巨大改進，但這意味著和許多 Java 特性一樣，它終究還是會讓人感覺沮喪和雞肋。

當你遇到學習困難時，請記住透過 IDE（NetBeans、IntelliJ Idea 和 Eclipse）獲得幫助，因為 IDE 可以智慧提示你何時使用 Lambda 表達式或方法引用，甚至有時還能為你最佳化程式碼。

<!--下面是腳註-->

[^1]: 功能貼上在一起的方法的確有點與眾不同，但它仍不失為一個庫。
[^2]: 例如,這個電子書是利用 [Pandoc](http://pandoc.org/) 製作出來的，它是用純函數式語言 [Haskell](https://www.haskell.org/) 編寫的一個程式 。
[^3]: 有時函數式語言將其描述為“程式碼即資料”。
[^4]: 這個語法來自 C++。
[^5]: 我還沒有驗證過這種說法。
[^6]: 當你理解了[並發編程](./24-Concurrent-Programming.md)章節的內容，你就能明白為什麼更改共享變數 “不是執行緒安全的” 的了。
[^7]: 介面能夠支援方法的原因是它們是 Java 8 預設方法，你將在下一章中了解到。
[^8]: 一些語言，如 Python，允許像呼叫其他函數一樣呼叫組合函數。但這是 Java，所以我們做做可為之事。
[^9]: 例如，[Immutables](https://immutables.github.io/) 和 [Mutability Detector](https://mutabilitydetector.github.io/MutabilityDetector/)。


<!-- 分頁 -->
<div style="page-break-after: always;"></div>
