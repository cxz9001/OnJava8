[TOC]

<!-- Enumerations -->

# 第二十二章 列舉

> 關鍵字 enum 可以將一組具名的值的有限集合建立為一種新的類型，而這些具名的值可以作為一般的程式元件使用。這是一種非常有用的功能

在[初始化和清理 ]() 這章結束的時候，我們已經簡單地介紹了列舉的概念。現在，你對 Java 已經有了更深刻的理解，因此可以更深入地學習 Java 中的列舉了。你將在本章中看到，使用 enum 可以做很多有趣的事情，同時，我們也會深入其他的 Java 特性，例如泛型和反射。在這個過程中，我們還將學習一些設計模式。

<!-- Basic enum Features -->

## 基本 enum 特性

我們已經在[初始化和清理 ]() 這章章看到，呼叫 enum 的 values() 方法，可以遍歷 enum 實例 .values() 方法返回 enum 實例的陣列，而且該陣列中的元素嚴格保持其在 enum 中聲明時的順序，因此你可以在循環中使用 values() 返回的陣列。

建立 enum 時，編譯器會為你生成一個相關的類，這個類繼承自 Java.lang.Enum。下面的例子示範了 Enum 提供的一些功能：

```java
// enums/EnumClass.java
// Capabilities of the Enum class
enum Shrubbery { GROUND, CRAWLING, HANGING }
public class EnumClass {
    public static void main(String[] args) {
        for(Shrubbery s : Shrubbery.values()) {
            System.out.println(
                    s + " ordinal: " + s.ordinal());
            System.out.print(
                    s.compareTo(Shrubbery.CRAWLING) + " ");
            System.out.print(
                    s.equals(Shrubbery.CRAWLING) + " ");
            System.out.println(s == Shrubbery.CRAWLING);
            System.out.println(s.getDeclaringClass());
            System.out.println(s.name());
            System.out.println("********************");
        }
// Produce an enum value from a String name:
        for(String s :
                "HANGING CRAWLING GROUND".split(" ")) {
            Shrubbery shrub =
                    Enum.valueOf(Shrubbery.class, s);
            System.out.println(shrub);
        }
    }
}
```

輸出：

```
GROUND ordinal: 0
-1 false false
class Shrubbery
GROUND
********************
CRAWLING ordinal: 1
0 true true
class Shrubbery
CRAWLING
********************
HANGING ordinal: 2
1 false false
class Shrubbery
HANGING
********************
HANGING
CRAWLING
GROUND
```

ordinal() 方法返回一個 int 值，這是每個 enum 實例在聲明時的次序，從 0 開始。可以使用==來比較 enum 實例，編譯器會自動為你提供 equals() 和 hashCode() 方法。Enum 類實現了 Comparable 介面，所以它具有 compareTo() 方法。同時，它還實現了 Serializable 介面。

如果在 enum 實例上呼叫 getDeclaringClass() 方法，我們就能知道其所屬的 enum 類。

name() 方法返回 enum 實例聲明時的名字，這與使用 toString() 方法效果相同。valueOf() 是在 Enum 中定義的 static 方法，它根據給定的名字返回相應的 enum 實例，如果不存在給定名字的實例，將會拋出異常。

### 將靜態類型匯入用於 enum

先看一看 [初始化和清理 ]() 這章中 Burrito.java 的另一個版本：

```java
// enums/SpicinessEnum.java
package enums;
public enum SpicinessEnum {
    NOT, MILD, MEDIUM, HOT, FLAMING
}
// enums/Burrito2.java
// {java enums.Burrito2}
package enums;
import static enums.SpicinessEnum.*;
public class Burrito2 {
    SpicinessEnum degree;
    public Burrito2(SpicinessEnum degree) {
        this.degree = degree;
    }
    @Override
    public String toString() {
        return "Burrito is "+ degree;
    }
    public static void main(String[] args) {
        System.out.println(new Burrito2(NOT));
        System.out.println(new Burrito2(MEDIUM));
        System.out.println(new Burrito2(HOT));
    }
}
```

輸出為：

```
Burrito is NOT
Burrito is MEDIUM
Burrito is HOT
```

使用 static import 能夠將 enum 實例的標識符帶入目前的命名空間，所以無需再用 enum 類型來修飾 enum 實例。這是一個好的想法嗎？或者還是顯式地修飾 enum 實例更好？這要看程式碼的複雜程度了。編譯器可以確保你使用的是正確的類型，所以唯一需要擔心的是，使用靜態匯入會不會導致你的程式碼令人難以理解。多數情況下，使用 static import 還是有好處的，不過，程式設計師還是應該對具體情況進行具體分析。

注意，在定義 enum 的同一個文件中，這種技巧無法使用，如果是在預設包中定義 enum，這種技巧也無法使用（在 Sun 內部對這一點顯然也有不同意見）。

<!-- Adding Methods to an enum -->

## 方法添加

除了不能繼承自一個 enum 之外，我們基本上可以將 enum 看作一個一般的類。也就是說我們可以向 enum 中添加方法。enum 甚至可以有 main() 方法。

一般來說，我們希望每個列舉實例能夠返回對自身的描述，而不僅僅只是預設的 toString() 實現，這只能返回列舉實例的名字。為此，你可以提供一個構造器，專門負責處理這個額外的訊息，然後添加一個方法，返回這個描述訊息。看一看下面的範例：

```java
// enums/OzWitch.java
// The witches in the land of Oz
public enum OzWitch {
    // Instances must be defined first, before methods:
    WEST("Miss Gulch, aka the Wicked Witch of the West"),
    NORTH("Glinda, the Good Witch of the North"),
    EAST("Wicked Witch of the East, wearer of the Ruby " +
            "Slippers, crushed by Dorothy's house"),
    SOUTH("Good by inference, but missing");
    private String description;
    // Constructor must be package or private access:
    private OzWitch(String description) {
        this.description = description;
    }
    public String getDescription() { return description; }
    public static void main(String[] args) {
        for(OzWitch witch : OzWitch.values())
            System.out.println(
                    witch + ": " + witch.getDescription());
    }
}
```

輸出為：

```
WEST: Miss Gulch, aka the Wicked Witch of the West
NORTH: Glinda, the Good Witch of the North
EAST: Wicked Witch of the East, wearer of the Ruby
Slippers, crushed by Dorothy's house
SOUTH: Good by inference, but missing
```

注意，如果你打算定義自己的方法，那麼必須在 enum 實例序列的最後添加一個分號。同時，Java 要求你必須先定義 enum 實例。如果在定義 enum 實例之前定義了任何方法或屬性，那麼在編譯時就會得到錯誤訊息。

enum 中的構造器與方法和普通的類沒有區別，因為除了有少許限制之外，enum 就是一個普通的類。所以，我們可以使用 enum 做許多事情（雖然，我們一般只使用普通的列舉類型）

在這個例子中，雖然我們有意識地將 enum 的構造器聲明為 private，但對於它的可訪問性而言，其實並沒有什麼變化，因為（即使不聲明為 private）我們只能在 enum 定義的內部使用其構造器建立 enum 實例。一旦 enum 的定義結束，編譯器就不允許我們再使用其構造器來建立任何實例了。

### 覆蓋 enum 的方法

覆蓋 toSring() 方法，給我們提供了另一種方式來為列舉實例生成不同的字串描述訊息。
在下面的範例中，我們使用的就是實例的名字，不過我們希望改變其格式。覆蓋 enum 的 toSring() 方法與覆蓋一般類的方法沒有區別：

```java
// enums/SpaceShip.java
import java.util.stream.*;
public enum SpaceShip {
    SCOUT, CARGO, TRANSPORT,
    CRUISER, BATTLESHIP, MOTHERSHIP;
    @Override
    public String toString() {
        String id = name();
        String lower = id.substring(1).toLowerCase();
        return id.charAt(0) + lower;
    }
    public static void main(String[] args) {
        Stream.of(values())
                .forEach(System.out::println);
    }
}
```

輸出為：

```
Scout
Cargo
Transport
Cruiser
Battleship
Mothership
```

toString() 方法透過呼叫 name() 方法取得 SpaceShip 的名字，然後將其修改為只有首字母大寫的格式。



<!-- enums in switch Statements -->

## switch 語句中的 enum

在 switch 中使用 enum，是 enum 提供的一項非常便利的功能。一般來說，在 switch 中只能使用整數值，而列舉實例天生就具備整數值的次序，並且可以透過 ordinal() 方法取得其次序（顯然編譯器幫我們做了類似的工作），因此我們可以在 switch 語句中使用 enum。

雖然一般情況下我們必須使用 enum 類型來修飾一個 enum 實例，但是在 case 語句中卻不必如此。下面的例子使用 enum 構造了一個小型狀態機：

```java
// enums/TrafficLight.java
// Enums in switch statements
// Define an enum type:
enum Signal { GREEN, YELLOW, RED, }

public class TrafficLight {
    Signal color = Signal.RED;
    public void change() {
        switch(color) {
           // Note you don't have to say Signal.RED
            // in the case statement:
            case RED: color = Signal.GREEN;
                break;
            case GREEN: color = Signal.YELLOW;
                break;
            case YELLOW: color = Signal.RED;
                break;
        }
    }
    @Override
    public String toString() {
        return "The traffic light is " + color;
    }
    public static void main(String[] args) {
        TrafficLight t = new TrafficLight();
        for(int i = 0; i < 7; i++) {
            System.out.println(t);
            t.change();
        }
    }
}
```

輸出為：

```
The traffic light is RED
The traffic light is GREEN
The traffic light is YELLOW
The traffic light is RED
The traffic light is GREEN
The traffic light is YELLOW
The traffic light is RED
```

編譯器並沒有抱怨 switch 中沒有 default 語句，但這並不是因為每一個 Signal 都有對應的 case 語句。如果你注釋掉其中的某個 case 語句，編譯器同樣不會抱怨什麼。這意味著，你必須確保自己覆蓋了所有的分支。但是，如果在 case 語句中呼叫 return，那麼編譯器就會抱怨缺少 default 語句了。這與是否覆蓋了 enum 的所有實例無關。



<!-- The Mystery of values() -->

## values 方法的神秘之處

前面已經提到，編譯器為你建立的 enum 類都繼承自 Enum 類。然而，如果你研究一下 Enum 類就會發現，它並沒有 values() 方法。可我們明明已經用過該方法了，難道存在某種“隱藏的”方法嗎？我們可以利用反射機制編寫一個簡單的程式，來查看其中的究竟：

```java
// enums/Reflection.java
// Analyzing enums using reflection
import java.lang.reflect.*;
import java.util.*;
import onjava.*;
enum Explore { HERE, THERE }
public class Reflection {
    public static
    Set<String> analyze(Class<?> enumClass) {
        System.out.println(
                "_____ Analyzing " + enumClass + " _____");
        System.out.println("Interfaces:");
        for(Type t : enumClass.getGenericInterfaces())
            System.out.println(t);
        System.out.println(
                "Base: " + enumClass.getSuperclass());
        System.out.println("Methods: ");
        Set<String> methods = new TreeSet<>();
        for(Method m : enumClass.getMethods())
            methods.add(m.getName());
        System.out.println(methods);
        return methods;
    }
    public static void main(String[] args) {
        Set<String> exploreMethods =
                analyze(Explore.class);
        Set<String> enumMethods = analyze(Enum.class);
        System.out.println(
                "Explore.containsAll(Enum)? " +
                        exploreMethods.containsAll(enumMethods));
        System.out.print("Explore.removeAll(Enum): ");
        exploreMethods.removeAll(enumMethods);
        System.out.println(exploreMethods);
// Decompile the code for the enum:
        OSExecute.command(
                "javap -cp build/classes/main Explore");
    }
}
```

輸出為：

```java
_____ Analyzing class Explore _____
Interfaces:
Base: class java.lang.Enum
Methods:
[compareTo, equals, getClass, getDeclaringClass,
hashCode, name, notify, notifyAll, ordinal, toString,
valueOf, values, wait]
_____ Analyzing class java.lang.Enum _____
Interfaces:
java.lang.Comparable<E>
interface java.io.Serializable
Base: class java.lang.Object
Methods:
[compareTo, equals, getClass, getDeclaringClass,
hashCode, name, notify, notifyAll, ordinal, toString,
valueOf, wait]
Explore.containsAll(Enum)? true
Explore.removeAll(Enum): [values]
Compiled from "Reflection.java"
final class Explore extends java.lang.Enum<Explore> {
  public static final Explore HERE;
  public static final Explore THERE;
  public static Explore[] values();
  public static Explore valueOf(java.lang.String);
  static {};
}
```

答案是，values() 是由編譯器添加的 static 方法。可以看出，在建立 Explore 的過程中，編譯器還為其添加了 valueOf() 方法。這可能有點令人迷惑，Enum 類不是已經有 valueOf() 方法了嗎。

不過 Enum 中的 valueOf() 方法需要兩個參數，而這個新增的方法只需一個參數。由於這裡使用的 Set 只儲存方法的名字，而不考慮方法的簽名，所以在呼叫 Explore.removeAll(Enum) 之後，就只剩下[values] 了。

從最後的輸出中可以看到，編譯器將 Explore 標記為 final 類，所以無法繼承自 enum，其中還有一個 static 的初始化子句，稍後我們將學習如何重定義該句。

由於擦除效應（在[泛型 ]() 章節中介紹過），反編譯無法得到 Enum 的完整訊息，所以它展示的 Explore 的父類只是一個原始的 Enum，而非事實上的 Enum\<Explore\>。

由於 values() 方法是由編譯器插入到 enum 定義中的 static 方法，所以，如果你將 enum 實例向上轉型為 Enum，那麼 values() 方法就不可訪問了。不過，在 Class 中有一個 getEnumConstants() 方法，所以即便 Enum 介面中沒有 values() 方法，我們仍然可以透過 Class 物件取得所有 enum 實例。

```java
// enums/UpcastEnum.java
// No values() method if you upcast an enum
enum Search { HITHER, YON }
public class UpcastEnum {
    public static void main(String[] args) {
        Search[] vals = Search.values();
        Enum e = Search.HITHER; // Upcast
// e.values(); // No values() in Enum
        for(Enum en : e.getClass().getEnumConstants())
            System.out.println(en);
    }
}
```

輸出為：

```
HITHER
YON
```

因為 getEnumConstants() 是 Class 上的方法，所以你甚至可以對不是列舉的類呼叫此方法：

```java
// enums/NonEnum.java
public class NonEnum {
    public static void main(String[] args) {
        Class<Integer> intClass = Integer.class;
        try {
            for(Object en : intClass.getEnumConstants())
                System.out.println(en);
        } catch(Exception e) {
            System.out.println("Expected: " + e);
        }
    }
}
```

輸出為：

```java
Expected: java.lang.NullPointerException
```

只不過，此時該方法返回 null，所以當你試圖使用其返回的結果時會發生異常。

<!-- Implements, not Inherits -->

## 實現而非繼承

我們已經知道，所有的 enum 都繼承自 Java.lang.Enum 類。由於 Java 不支援多重繼承，所以你的 enum 不能再繼承其他類：

```java
enum NotPossible extends Pet { ... // Won't work
```

然而，在我們建立一個新的 enum 時，可以同時實現一個或多個介面：

```java
// enums/cartoons/EnumImplementation.java
// An enum can implement an interface
// {java enums.cartoons.EnumImplementation}
package enums.cartoons;
import java.util.*;
import java.util.function.*;
enum CartoonCharacter
        implements Supplier<CartoonCharacter> {
    SLAPPY, SPANKY, PUNCHY,
    SILLY, BOUNCY, NUTTY, BOB;
    private Random rand =
            new Random(47);
    @Override
    public CartoonCharacter get() {
        return values()[rand.nextInt(values().length)];
    }
}
public class EnumImplementation {
    public static <T> void printNext(Supplier<T> rg) {
        System.out.print(rg.get() + ", ");
    }
    public static void main(String[] args) {
// Choose any instance:
        CartoonCharacter cc = CartoonCharacter.BOB;
        for(int i = 0; i < 10; i++)
            printNext(cc);
    }
}
```

輸出為：

```
BOB, PUNCHY, BOB, SPANKY, NUTTY, PUNCHY, SLAPPY, NUTTY,
NUTTY, SLAPPY,
```

這個結果有點奇怪，不過你必須要有一個 enum 實例才能呼叫其上的方法。現在，在任何接受 Supplier 參數的方法中，例如 printNext()，都可以使用 CartoonCharacter。

<!-- Random Selection -->

## 隨機選擇

就像你在 CartoonCharacter.get() 中看到的那樣，本章中的很多範例都需要從 enum 實例中進行隨機選擇。我們可以利用泛型，從而使得這個工作更一般化，並將其加入到我們的工具庫中。

```java
// onjava/Enums.java
package onjava;
import java.util.*;
public class Enums {
    private static Random rand = new Random(47);
    
    public static <T extends Enum<T>> T random(Class<T> ec) {
        return random(ec.getEnumConstants());
    }
    
    public static <T> T random(T[] values) {
        return values[rand.nextInt(values.length)];
    }
}
```

古怪的語法\<T extends Enum\<T\>\> 表示 T 是一個 enum 實例。而將 Class\<T\> 作為參數的話，我們就可以利用 Class 物件得到 enum 實例的陣列了。重載後的 random() 方法只需使用 T[] 作為參數，因為它並不會呼叫 Enum 上的任何操作，它只需從陣列中隨機選擇一個元素即可。這樣，最終的返回類型正是 enum 的類型。

下面是 random() 方法的一個簡單範例：

```java
// enums/RandomTest.java
import onjava.*;
enum Activity { SITTING, LYING, STANDING, HOPPING,
    RUNNING, DODGING, JUMPING, FALLING, FLYING }
    
public class RandomTest {
    public static void main(String[] args) {
        for(int i = 0; i < 20; i++)
            System.out.print(
                    Enums.random(Activity.class) + " ");
    }
}
```

輸出為：

```
STANDING FLYING RUNNING STANDING RUNNING STANDING LYING
DODGING SITTING RUNNING HOPPING HOPPING HOPPING RUNNING
STANDING LYING FALLING RUNNING FLYING LYING
```



<!-- Using Interfaces for Organization -->

## 使用介面組織列舉

無法從 enum 繼承子類有時很令人沮喪。這種需求有時源自我們希望擴展原 enum 中的元素，有時是因為我們希望使用子類將一個 enum 中的元素進行分組。

在一個介面的內部，建立實現該介面的列舉，以此將元素進行分組，可以達到將列舉元素分類組織的目的。舉例來說，假設你想用 enum 來表示不同類別的食物，同時還希望每個 enum 元素仍然保持 Food 類型。那可以這樣實現：

```java
// enums/menu/Food.java
// Subcategorization of enums within interfaces
package enums.menu;
public interface Food {
    enum Appetizer implements Food {
        SALAD, SOUP, SPRING_ROLLS;
    }
    enum MainCourse implements Food {
        LASAGNE, BURRITO, PAD_THAI,
        LENTILS, HUMMOUS, VINDALOO;
    }
    enum Dessert implements Food {
        TIRAMISU, GELATO, BLACK_FOREST_CAKE,
        FRUIT, CREME_CARAMEL;
    }
    enum Coffee implements Food {
        BLACK_COFFEE, DECAF_COFFEE, ESPRESSO,
        LATTE, CAPPUCCINO, TEA, HERB_TEA;
    }
}
```

對於 enum 而言，實現介面是使其子類化的唯一辦法，所以嵌入在 Food 中的每個 enum 都實現了 Food 介面。現在，在下面的程式中，我們可以說“所有東西都是某種類型的 Food"。

```java
// enums/menu/TypeOfFood.java
// {java enums.menu.TypeOfFood}
package enums.menu;
import static enums.menu.Food.*;
public class TypeOfFood {
    public static void main(String[] args) {
        Food food = Appetizer.SALAD;
        food = MainCourse.LASAGNE;
        food = Dessert.GELATO;
        food = Coffee.CAPPUCCINO;
    }
}
```

如果 enum 類型實現了 Food 介面，那麼我們就可以將其實例向上轉型為 Food，所以上例中的所有東西都是 Food。

然而，當你需要與一大堆類型打交道時，介面就不如 enum 好用了。例如，如果你想建立一個“列舉的列舉”，那麼可以建立一個新的 enum，然後用其實例包裝 Food 中的每一個 enum 類：

```java
// enums/menu/Course.java
package enums.menu;
import onjava.*;
public enum Course {
    APPETIZER(Food.Appetizer.class),
    MAINCOURSE(Food.MainCourse.class),
    DESSERT(Food.Dessert.class),
    COFFEE(Food.Coffee.class);
    private Food[] values;
    private Course(Class<? extends Food> kind) {
        values = kind.getEnumConstants();
    }
    public Food randomSelection() {
        return Enums.random(values);
    }
}
```

每一個 Course 的實例都將其對應的 Class 物件作為構造器的參數。通過 getEnumConstants0 方法，可以從該 Class 物件中取得某個 Food 子類的所有 enum 實例。這些實例在 randomSelection() 中被用到。因此，透過從每一個 Course 實例中隨機地選擇一個 Food，我們便能夠生成一份選單：

```java
// enums/menu/Meal.java
// {java enums.menu.Meal}
package enums.menu;
public class Meal {
    public static void main(String[] args) {
        for(int i = 0; i < 5; i++) {
            for(Course course : Course.values()) {
                Food food = course.randomSelection();
                System.out.println(food);
            }
            System.out.println("***");
        }
    }
}
```

輸出為：

```
SPRING_ROLLS
VINDALOO
FRUIT
DECAF_COFFEE
***
SOUP
VINDALOO
FRUIT
TEA
***
SALAD
BURRITO
FRUIT
TEA
***
SALAD
BURRITO
CREME_CARAMEL
LATTE
***
SOUP
BURRITO
TIRAMISU
ESPRESSO
***
```

在這個例子中，我們透過遍歷每一個 Course 實例來獲得“列舉的列舉”的值。稍後，在 VendingMachine.java 中，我們會看到另一種組織列舉實例的方式，但其也有一些其他的限制。

此外，還有一種更簡潔的管理列舉的辦法，就是將一個 enum 嵌套在另一個 enum 內。就像這樣：

```java
// enums/SecurityCategory.java
// More succinct subcategorization of enums
import onjava.*;
enum SecurityCategory {
    STOCK(Security.Stock.class),
    BOND(Security.Bond.class);
    Security[] values;
    SecurityCategory(Class<? extends Security> kind) {
        values = kind.getEnumConstants();
    }
    interface Security {
        enum Stock implements Security {
            SHORT, LONG, MARGIN
        }
        enum Bond implements Security {
            MUNICIPAL, JUNK
        }
    }
    public Security randomSelection() {
        return Enums.random(values);
    }
    public static void main(String[] args) {
        for(int i = 0; i < 10; i++) {
            SecurityCategory category =
                    Enums.random(SecurityCategory.class);
            System.out.println(category + ": " +
                    category.randomSelection());
        }
    }
}
```

輸出為：

```
BOND: MUNICIPAL
BOND: MUNICIPAL
STOCK: MARGIN
STOCK: MARGIN
BOND: JUNK
STOCK: SHORT
STOCK: LONG
STOCK: LONG
BOND: MUNICIPAL
BOND: JUNK
```

Security 介面的作用是將其所包含的 enum 組合成一個公共類型，這一點是有必要的。然後，SecurityCategory 才能將 Security 中的 enum 作為其構造器的參數使用，以起到組織的效果。

如果我們將這種方式應用於 Food 的例子，結果應該這樣：

```java
// enums/menu/Meal2.java
// {java enums.menu.Meal2}
package enums.menu;
import onjava.*;
public enum Meal2 {
    APPETIZER(Food.Appetizer.class),
    MAINCOURSE(Food.MainCourse.class),
    DESSERT(Food.Dessert.class),
    COFFEE(Food.Coffee.class);
    private Food[] values;
    private Meal2(Class<? extends Food> kind) {
        values = kind.getEnumConstants();
    }
    public interface Food {
        enum Appetizer implements Food {
            SALAD, SOUP, SPRING_ROLLS;
        }
        enum MainCourse implements Food {
            LASAGNE, BURRITO, PAD_THAI,
            LENTILS, HUMMOUS, VINDALOO;
        }
        enum Dessert implements Food {
            TIRAMISU, GELATO, BLACK_FOREST_CAKE,
            FRUIT, CREME_CARAMEL;
        }
        enum Coffee implements Food {
            BLACK_COFFEE, DECAF_COFFEE, ESPRESSO,
            LATTE, CAPPUCCINO, TEA, HERB_TEA;
        }
    }
    public Food randomSelection() {
        return Enums.random(values);
    }
    public static void main(String[] args) {
        for(int i = 0; i < 5; i++) {
            for(Meal2 meal : Meal2.values()) {
                Food food = meal.randomSelection();
                System.out.println(food);
            }
            System.out.println("***");
        }
    }
}
```

輸出為：

```
SPRING_ROLLS
VINDALOO
FRUIT
DECAF_COFFEE
***
SOUP
VINDALOO
FRUIT
TEA
***
SALAD
BURRITO
FRUIT
TEA
***
SALAD
BURRITO
CREME_CARAMEL
LATTE
***
SOUP
BURRITO
TIRAMISU
ESPRESSO
***
```

其實，這僅僅是重新組織了一下程式碼，不過多數情況下，這種方式使你的程式碼具有更清晰的結構。

<!-- Using EnumSet Instead of Flags -->

## 使用 EnumSet 替代 Flags

Set 是一種集合，只能向其中添加不重複的物件。當然，enum 也要求其成員都是唯一的，所以 enum 看起來也具有集合的行為。不過，由於不能從 enum 中刪除或添加元素，所以它只能算是不太有用的集合。Java SE5 引入 EnumSet，是為了透過 enum 建立一種替代品，以替代傳統的基於 int 的“位標誌”。這種標誌可以用來表示某種“開/關”訊息，不過，使用這種標誌，我們最終操作的只是一些 bit，而不是這些 bit 想要表達的概念，因此很容易寫出令人難以理解的程式碼。

EnumSet 的設計充分考慮到了速度因素，因為它必須與非常高效的 bit 標誌相競爭（其操作與 HashSet 相比，非常地快），就其內部而言，它（可能）就是將一個 long 值作為位元向量，所以 EnumSet 非常快速高效。使用 EnumSet 的優點是，它在說明一個二進位制位是否存在時，具有更好的表達能力，並且無需擔心性能。

EnumSet 中的元素必須來自一個 enum。下面的 enum 表示在一座大樓中，警報感測器的安放位置：

```java
// enums/AlarmPoints.java
package enums;
public enum AlarmPoints {
    STAIR1, STAIR2, LOBBY, OFFICE1, OFFICE2, OFFICE3,
    OFFICE4, BATHROOM, UTILITY, KITCHEN
}
```

然後，我們用 EnumSet 來跟蹤報警器的狀態：

```java
// enums/EnumSets.java
// Operations on EnumSets
// {java enums.EnumSets}
package enums;
import java.util.*;
import static enums.AlarmPoints.*;
public class EnumSets {
    public static void main(String[] args) {
        EnumSet<AlarmPoints> points =
                EnumSet.noneOf(AlarmPoints.class); // Empty
        points.add(BATHROOM);
        System.out.println(points);
        points.addAll(
                EnumSet.of(STAIR1, STAIR2, KITCHEN));
        System.out.println(points);
        points = EnumSet.allOf(AlarmPoints.class);
        points.removeAll(
                EnumSet.of(STAIR1, STAIR2, KITCHEN));
        System.out.println(points);
        points.removeAll(
                EnumSet.range(OFFICE1, OFFICE4));
        System.out.println(points);
        points = EnumSet.complementOf(points);
        System.out.println(points);
    }
}
```

輸出為：

```java
[BATHROOM]
[STAIR1, STAIR2, BATHROOM, KITCHEN]
[LOBBY, OFFICE1, OFFICE2, OFFICE3, OFFICE4, BATHROOM,
UTILITY]
[LOBBY, BATHROOM, UTILITY]
[STAIR1, STAIR2, OFFICE1, OFFICE2, OFFICE3, OFFICE4,
KITCHEN]
```

使用 static import 可以簡化 enum 常量的使用。EnumSet 的方法的名字都相當直觀，你可以查閱 JDK 文件找到其完整詳細的描述。如果仔細研究了 EnumSet 的文件，你還會發現 of() 方法被重載了很多次，不但為可變數量參數進行了重載，而且為接收 2 至 5 個顯式的參數的情況都進行了重載。這也從側面表現了 EnumSet 對性能的關注。因為，其實只使用單獨的 of() 方法解決可變參數已經可以解決整個問題了，但是對比顯式的參數，會有一點性能損失。採用現在這種設計，當你只使用 2 到 5 個參數呼叫 of() 方法時，你可以呼叫對應的重載過的方法（速度稍快一點），而當你使用一個參數或多過 5 個參數時，你呼叫的將是使用可變參數的 of() 方法。注意，如果你只使用一個參數，編譯器並不會構造可變參數的陣列，所以與呼叫只有一個參數的方法相比，也就不會有額外的效能損耗。

EnumSet 的基礎是 long，一個 long 值有 64 位，而一個 enum 實例只需一位 bit 表示其是否存在。
也就是說，在不超過一個 long 的表達能力的情況下，你的 EnumSet 可以應用於最多不超過 64 個元素的 enum。如果 enum 超過了 64 個元素會發生什麼事呢？

```java
// enums/BigEnumSet.java
import java.util.*;
public class BigEnumSet {
    enum Big { A0, A1, A2, A3, A4, A5, A6, A7, A8, A9,
        A10, A11, A12, A13, A14, A15, A16, A17, A18, A19,
        A20, A21, A22, A23, A24, A25, A26, A27, A28, A29,
        A30, A31, A32, A33, A34, A35, A36, A37, A38, A39,
        A40, A41, A42, A43, A44, A45, A46, A47, A48, A49,
        A50, A51, A52, A53, A54, A55, A56, A57, A58, A59,
        A60, A61, A62, A63, A64, A65, A66, A67, A68, A69,
        A70, A71, A72, A73, A74, A75 }
    public static void main(String[] args) {
        EnumSet<Big> bigEnumSet = EnumSet.allOf(Big.class);
        System.out.println(bigEnumSet);
    }
}
```

輸出為：

```java
[A0, A1, A2, A3, A4, A5, A6, A7, A8, A9, A10, A11, A12,
A13, A14, A15, A16, A17, A18, A19, A20, A21, A22, A23,
A24, A25, A26, A27, A28, A29, A30, A31, A32, A33, A34,
A35, A36, A37, A38, A39, A40, A41, A42, A43, A44, A45,
A46, A47, A48, A49, A50, A51, A52, A53, A54, A55, A56,
A57, A58, A59, A60, A61, A62, A63, A64, A65, A66, A67,
A68, A69, A70, A71, A72, A73, A74, A75]
```

顯然，EnumSet 可以應用於多過 64 個元素的 enum，所以我猜測，Enum 會在必要的時候增加一個 long。

<!-- Using EnumMap -->

## 使用 EnumMap

EnumMap 是一種特殊的 Map，它要求其中的鍵（key）必須來自一個 enum，由於 enum 本身的限制，所以 EnumMap 在內部可由陣列實現。因此 EnumMap 的速度很快，我們可以放心地使用 enum 實例在 EnumMap 中進行尋找操作。不過，我們只能將 enum 的實例作為鍵來呼叫 put() 可方法，其他操作與使用一般的 Map 差不多。

下面的例子示範了*指令設計模式*的用法。一般來說，指令模式首先需要一個只有單一方法的介面，然後從該介面實現具有各自不同的行為的多個子類。接下來，程式設計師就可以構造指令物件，並在需要的時候使用它們了：

```java
// enums/EnumMaps.java
// Basics of EnumMaps
// {java enums.EnumMaps}
package enums;
import java.util.*;
import static enums.AlarmPoints.*;
interface Command { void action(); }
public class EnumMaps {
    public static void main(String[] args) {
        EnumMap<AlarmPoints,Command> em =
                new EnumMap<>(AlarmPoints.class);
        em.put(KITCHEN,
                () -> System.out.println("Kitchen fire!"));
        em.put(BATHROOM,
                () -> System.out.println("Bathroom alert!"));
        for(Map.Entry<AlarmPoints,Command> e:
                em.entrySet()) {
            System.out.print(e.getKey() + ": ");
            e.getValue().action();
        }
        try { // If there's no value for a particular key:
            em.get(UTILITY).action();
        } catch(Exception e) {
            System.out.println("Expected: " + e);
        }
    }
}
```

輸出為：

```
BATHROOM: Bathroom alert!
KITCHEN: Kitchen fire!
Expected: java.lang.NullPointerException
```

與 EnumSet 一樣，enum 實例定義時的次序決定了其在 EnumMap 中的順序。

main() 方法的最後部分說明，enum 的每個實例作為一個鍵，總是存在的。但是，如果你沒有為這個鍵呼叫 put() 方法來存入相應的值的話，其對應的值就是 null。

與常量相關的方法（constant-specific methods 將在下一節中介紹）相比，EnumMap 有一個優點，那 EnumMap 允許程式設計師改變值物件，而常量相關的方法在編譯期就被固定了。稍後你會看到，在你有多種類型的 enum，而且它們之間存在互操作的情況下，我們可以用 EnumMap 實現多路分發（multiple dispatching）。

<!-- Constant-Specific Methods -->

## 常量特定方法

Java 的 enum 有一個非常有趣的特性，即它允許程式設計師為 enum 實例編寫方法，從而為每個 enum 實例賦予各自不同的行為。要實現常量相關的方法，你需要為 enum 定義一個或多個 abstract 方法，然後為每個 enum 實例實現該抽象方法。參考下面的例子：

```java
// enums/ConstantSpecificMethod.java
import java.util.*;
import java.text.*;
public enum ConstantSpecificMethod {
    DATE_TIME {
        @Override
        String getInfo() {
            return
                    DateFormat.getDateInstance()
                            .format(new Date());
        }
    },
    CLASSPATH {
        @Override
        String getInfo() {
            return System.getenv("CLASSPATH");
        }
    },
    VERSION {
        @Override
        String getInfo() {
            return System.getProperty("java.version");
        }
    };
    abstract String getInfo();
    public static void main(String[] args) {
        for(ConstantSpecificMethod csm : values())
            System.out.println(csm.getInfo());
    }
}
```

輸出為：

```java
May 9, 2017
C:\Users\Bruce\Documents\GitHub\on-
java\ExtractedExamples\\gradle\wrapper\gradle-
wrapper.jar
1.8.0_112
```

透過相應的 enum 實例，我們可以呼叫其上的方法。這通常也稱為表驅動的程式碼（table-driven code，請注意它與前面提到的指令模式的相似之處）。

在物件導向的程式設計中，不同的行為與不同的類關聯。而透過常量相關的方法，每個 enum 實例可以具備自己獨特的行為，這似乎說明每個 enum 實例就像一個獨特的類。在上面的例子中，enum 實例似乎被當作其“超類”ConstantSpecificMethod 來使用，在呼叫 getInfo() 方法時，體現出多態的行為。

然而，enum 實例與類的相似之處也僅限於此了。我們並不能真的將 enum 實例作為一個類型來使用：

```java
// enums/NotClasses.java
// {javap -c LikeClasses}
enum LikeClasses {
    WINKEN {
        @Override
        void behavior() {
            System.out.println("Behavior1");
        }
    },
    BLINKEN {
        @Override
        void behavior() {
            System.out.println("Behavior2");
        }
    },
    NOD {
        @Override
        void behavior() {
            System.out.println("Behavior3");
        }
    };
    abstract void behavior();
}
public class NotClasses {
    // void f1(LikeClasses.WINKEN instance) {} // Nope
}
```

輸出為（前 12 行）：

```
Compiled from "NotClasses.java"
abstract class LikeClasses extends
java.lang.Enum<LikeClasses> {
public static final LikeClasses WINKEN;
public static final LikeClasses BLINKEN;
public static final LikeClasses NOD;
public static LikeClasses[] values();
Code:
0: getstatic #2 // Field
$VALUES:[LLikeClasses;
3: invokevirtual #3 // Method
"[LLikeClasses;".clone:()Ljava/lang/Object;
...
```

在方法 f1() 中，編譯器不允許我們將一個 enum 實例當作 class 類型。如果我們分析一下編譯器生成的程式碼，就知道這種行為也是很正常的。因為每個 enum 元素都是一個 LikeClasses 類型的 static final 實例。

同時，由於它們是 static 實例，無法訪問外部類的非 static 元素或方法，所以對於內部的 enum 的實例而言，其行為與一般的內部類並不相同。

再看一個更有趣的關於洗車的例子。每個顧客在洗車時，都有一個選擇選單，每個選擇對應一個不同的動作。可以將一個常量相關的方法關聯到一個選擇上，再使用一個 EnumSet 來儲存客戶的選擇：

```java
// enums/CarWash.java
import java.util.*;
public class CarWash {
    public enum Cycle {
        UNDERBODY {
            @Override
            void action() {
                System.out.println("Spraying the underbody");
            }
        },
        WHEELWASH {
            @Override
            void action() {
                System.out.println("Washing the wheels");
            }
        },
        PREWASH {
            @Override
            void action() {
                System.out.println("Loosening the dirt");
            }
        },
        BASIC {
            @Override
            void action() {
                System.out.println("The basic wash");
            }
        },
        HOTWAX {
            @Override
            void action() {
                System.out.println("Applying hot wax");
            }
        },
        RINSE {
            @Override
            void action() {
                System.out.println("Rinsing");
            }
        },
        BLOWDRY {
            @Override
            void action() {
                System.out.println("Blowing dry");
            }
        };
        abstract void action();
    }
    EnumSet<Cycle> cycles =
            EnumSet.of(Cycle.BASIC, Cycle.RINSE);
    public void add(Cycle cycle) {
        cycles.add(cycle);
    }
    public void washCar() {
        for(Cycle c : cycles)
            c.action();
    }
    @Override
    public String toString() {
        return cycles.toString();
    }
    public static void main(String[] args) {
        CarWash wash = new CarWash();
        System.out.println(wash);
        wash.washCar();
// Order of addition is unimportant:
        wash.add(Cycle.BLOWDRY);
        wash.add(Cycle.BLOWDRY); // Duplicates ignored
        wash.add(Cycle.RINSE);
        wash.add(Cycle.HOTWAX);
        System.out.println(wash);
        wash.washCar();
    }
}
```

輸出為：

```
[BASIC, RINSE]
The basic wash
Rinsing
[BASIC, HOTWAX, RINSE, BLOWDRY]
The basic wash
Applying hot wax
Rinsing
Blowing dry
```

與使用匿名內部類相比較，定義常量相關方法的語法更高效、簡潔。

這個例子也展示了 EnumSet 了一些特性。因為它是一個集合，所以對於同一個元素而言，只能出現一次，因此對同一個參數重複地呼叫 add0 方法會被忽略掉（這是正確的行為，因為一個 bit 位開關只能“打開”一次），同樣地，向 EnumSet 添加 enum 實例的順序並不重要，因為其輸出的次序決定於 enum 實例定義時的次序。

除了實現 abstract 方法以外，程式設計師是否可以覆蓋常量相關的方法呢？答案是肯定的，參考下面的程式：

```java
// enums/OverrideConstantSpecific.java
public enum OverrideConstantSpecific {
    NUT, BOLT,
    WASHER {
        @Override
        void f() {
            System.out.println("Overridden method");
        }
    };
    void f() {
        System.out.println("default behavior");
    }
    public static void main(String[] args) {
        for(OverrideConstantSpecific ocs : values()) {
            System.out.print(ocs + ": ");
            ocs.f();
        }
    }
}
```

輸出為：

```
NUT: default behavior
BOLT: default behavior
WASHER: Overridden method
```

雖然 enum 有某些限制，但是一般而言，我們還是可以將其看作是類。

### 使用 enum 的職責鏈

在職責鏈（Chain of Responsibility）設計模式中，程式設計師以多種不同的方式來解決一個問題，然後將它們連結在一起。當一個請求到來時，它遍歷這個鏈，直到鏈中的某個解決方案能夠處理該請求。

透過常量相關的方法，我們可以很容易地實現一個簡單的職責鏈。我們以一個郵局的模型為例。郵局需要以儘可能通用的方式來處理每一封郵件，並且要不斷嘗試處理郵件，直到該郵件最終被確定為一封死信。其中的每一次嘗試可以看作為一個策略（也是一個設計模式），而完整的處理方式列表就是一個職責鏈。

我們先來描述一下郵件。郵件的每個關鍵特徵都可以用 enum 來表示。程式將隨機地生成 Mail 物件，如果要減小一封郵件的 GeneralDelivery 為 YES 的機率，那最簡單的方法就是多建立幾個不是 YES 的 enum 實例，所以 enum 的定義看起來有點古怪。

我們看到 Mail 中有一個 randomMail() 方法，它負責隨機地建立用於測試的郵件。而 generator() 方法生成一個 Iterable 物件，該物件在你呼叫 next() 方法時，在其內部使用 randomMail() 來建立 Mail 物件。這樣的結構使程式設計師可以透過呼叫 Mail.generator() 方法，很容易地構造出一個 foreach 循環：

```java
// enums/PostOffice.java
// Modeling a post office
import java.util.*;
import onjava.*;
class Mail {
    // The NO's reduce probability of random selection:
    enum GeneralDelivery {YES,NO1,NO2,NO3,NO4,NO5}
    enum Scannability {UNSCANNABLE,YES1,YES2,YES3,YES4}
    enum Readability {ILLEGIBLE,YES1,YES2,YES3,YES4}
    enum Address {INCORRECT,OK1,OK2,OK3,OK4,OK5,OK6}
    enum ReturnAddress {MISSING,OK1,OK2,OK3,OK4,OK5}
    GeneralDelivery generalDelivery;
    Scannability scannability;
    Readability readability;
    Address address;
    ReturnAddress returnAddress;
    static long counter = 0;
    long id = counter++;
    @Override
    public String toString() { return "Mail " + id; }
    public String details() {
        return toString() +
                ", General Delivery: " + generalDelivery +
                ", Address Scanability: " + scannability +
                ", Address Readability: " + readability +
                ", Address Address: " + address +
                ", Return address: " + returnAddress;
    }
    // Generate test Mail:
    public static Mail randomMail() {
        Mail m = new Mail();
        m.generalDelivery =
                Enums.random(GeneralDelivery.class);
        m.scannability =
                Enums.random(Scannability.class);
        m.readability =
                Enums.random(Readability.class);
        m.address = Enums.random(Address.class);
        m.returnAddress =
                Enums.random(ReturnAddress.class);
        return m;
    }
    public static
    Iterable<Mail> generator(final int count) {
        return new Iterable<Mail>() {
            int n = count;
            @Override
            public Iterator<Mail> iterator() {
                return new Iterator<Mail>() {
                    @Override
                    public boolean hasNext() {
                        return n-- > 0;
                    }
                    @Override
                    public Mail next() {
                        return randomMail();
                    }
                    @Override
                    public void remove() { // Not implemented
                        throw new UnsupportedOperationException();
                    }
                };
            }
        };
    }
}
public class PostOffice {
    enum MailHandler {
        GENERAL_DELIVERY {
            @Override
            boolean handle(Mail m) {
                switch(m.generalDelivery) {
                    case YES:
                        System.out.println(
                                "Using general delivery for " + m);
                        return true;
                    default: return false;
                }
            }
        },
        MACHINE_SCAN {
            @Override
            boolean handle(Mail m) {
                switch(m.scannability) {
                    case UNSCANNABLE: return false;
                    default:
                        switch(m.address) {
                            case INCORRECT: return false;
                            default:
                                System.out.println(
                                        "Delivering "+ m + " automatically");
                                return true;
                        }
                }
            }
        },
        VISUAL_INSPECTION {
            @Override
            boolean handle(Mail m) {
                switch(m.readability) {
                    case ILLEGIBLE: return false;
                    default:
                        switch(m.address) {
                            case INCORRECT: return false;
                            default:
                                System.out.println(
                                        "Delivering " + m + " normally");
                                return true;
                        }
                }
            }
        },
        RETURN_TO_SENDER {
            @Override
            boolean handle(Mail m) {
                switch(m.returnAddress) {
                    case MISSING: return false;
                    default:
                        System.out.println(
                                "Returning " + m + " to sender");
                        return true;
                }
            }
        };
        abstract boolean handle(Mail m);
    }
    static void handle(Mail m) {
        for(MailHandler handler : MailHandler.values())
            if(handler.handle(m))
                return;
        System.out.println(m + " is a dead letter");
    }
    public static void main(String[] args) {
        for(Mail mail : Mail.generator(10)) {
            System.out.println(mail.details());
            handle(mail);
            System.out.println("*****");
        }
    }
}
```

輸出為：

```
Mail 0, General Delivery: NO2, Address Scanability:
UNSCANNABLE, Address Readability: YES3, Address
Address: OK1, Return address: OK1
Delivering Mail 0 normally
*****
Mail 1, General Delivery: NO5, Address Scanability:
YES3, Address Readability: ILLEGIBLE, Address Address:
OK5, Return address: OK1
Delivering Mail 1 automatically
*****
Mail 2, General Delivery: YES, Address Scanability:
YES3, Address Readability: YES1, Address Address: OK1,
Return address: OK5
Using general delivery for Mail 2
*****
Mail 3, General Delivery: NO4, Address Scanability:
YES3, Address Readability: YES1, Address Address:
INCORRECT, Return address: OK4
Returning Mail 3 to sender
*****
Mail 4, General Delivery: NO4, Address Scanability:
UNSCANNABLE, Address Readability: YES1, Address
Address: INCORRECT, Return address: OK2
Returning Mail 4 to sender
*****
Mail 5, General Delivery: NO3, Address Scanability:
YES1, Address Readability: ILLEGIBLE, Address Address:
OK4, Return address: OK2
Delivering Mail 5 automatically
*****
Mail 6, General Delivery: YES, Address Scanability:
YES4, Address Readability: ILLEGIBLE, Address Address:
OK4, Return address: OK4
Using general delivery for Mail 6
*****
Mail 7, General Delivery: YES, Address Scanability:
YES3, Address Readability: YES4, Address Address: OK2,
Return address: MISSING
Using general delivery for Mail 7
*****
Mail 8, General Delivery: NO3, Address Scanability:
YES1, Address Readability: YES3, Address Address:
INCORRECT, Return address: MISSING
Mail 8 is a dead letter
*****
Mail 9, General Delivery: NO1, Address Scanability:
UNSCANNABLE, Address Readability: YES2, Address
Address: OK1, Return address: OK4
Delivering Mail 9 normally
*****
```

職責鏈由 enum MailHandler 實現，而 enum 定義的次序決定了各個解決策略在應用時的次序。對每一封郵件，都要按此順序嘗試每個解決策略，直到其中一個能夠成功地處理該郵件，如果所有的策略都失敗了，那麼該郵件將被判定為一封死信。

### 使用 enum 的狀態機

列舉類型非常適合用來建立狀態機。一個狀態機可以具有有限個特定的狀態，它通常根據輸入，從一個狀態轉移到下一個狀態，不過也可能存在瞬時狀態（transient states），而一旦任務執行結束，狀態機就會立刻離開瞬時狀態。

每個狀態都具有某些可接受的輸入，不同的輸入會使狀態機從目前狀態轉移到不同的新狀態。由於 enum 對其實例有嚴格限制，非常適合用來表現不同的狀態和輸入。一般而言，每個狀態都具有一些相關的輸出。

自動售貸機是一個很好的狀態機的例子。首先，我們用一個 enum 定義各種輸入：

```java
// enums/Input.java
import java.util.*;
public enum Input {
    NICKEL(5), DIME(10), QUARTER(25), DOLLAR(100),
    TOOTHPASTE(200), CHIPS(75), SODA(100), SOAP(50),
    ABORT_TRANSACTION {
        @Override
        public int amount() { // Disallow
            throw new RuntimeException("ABORT.amount()");
        }
    },
    STOP { // This must be the last instance.
        @Override
        public int amount() { // Disallow
            throw new RuntimeException("SHUT_DOWN.amount()");
        }
    };
    int value; // In cents
    Input(int value) { this.value = value; }
    Input() {}
    int amount() { return value; }; // In cents
    static Random rand = new Random(47);
    public static Input randomSelection() {
        // Don't include STOP:
        return values()[rand.nextInt(values().length - 1)];
    }
}
```

注意，除了兩個特殊的 Input 實例之外，其他的 Input 都有相應的價格，因此在介面中定義了 amount（方法。然而，對那兩個特殊 Input 實例而言，呼叫 amount（方法並不合適，所以如果程式設計師呼叫它們的 amount）方法就會有異常拋出（在介面內定義了一個方法，然後在你呼叫該方法的某個實現時就會拋出異常），這似乎有點奇怪，但由於 enum 的限制，我們不得不採用這種方式。

VendingMachine 對輸入的第一個反應是將其歸類為 Category enum 中的某一個 enum 實例，這可以透過 switch 實現。下面的例子示範了 enum 是如何使程式碼變得更加清晰且易於管理的：

```java
// enums/VendingMachine.java
// {java VendingMachine VendingMachineInput.txt}
import java.util.*;
import java.io.IOException;
import java.util.function.*;
import java.nio.file.*;
import java.util.stream.*;
enum Category {
    MONEY(Input.NICKEL, Input.DIME,
            Input.QUARTER, Input.DOLLAR),
    ITEM_SELECTION(Input.TOOTHPASTE, Input.CHIPS,
            Input.SODA, Input.SOAP),
    QUIT_TRANSACTION(Input.ABORT_TRANSACTION),
    SHUT_DOWN(Input.STOP);
    private Input[] values;
    Category(Input... types) { values = types; }
    private static EnumMap<Input,Category> categories =
            new EnumMap<>(Input.class);
    static {
        for(Category c : Category.class.getEnumConstants())
            for(Input type : c.values)
                categories.put(type, c);
    }
    public static Category categorize(Input input) {
        return categories.get(input);
    }
}

public class VendingMachine {
    private static State state = State.RESTING;
    private static int amount = 0;
    private static Input selection = null;
    enum StateDuration { TRANSIENT } // Tagging enum
    enum State {
        RESTING {
            @Override
            void next(Input input) {
                switch(Category.categorize(input)) {
                    case MONEY:
                        amount += input.amount();
                        state = ADDING_MONEY;
                        break;
                    case SHUT_DOWN:
                        state = TERMINAL;
                    default:
                }
            }
        },
        ADDING_MONEY {
            @Override
            void next(Input input) {
                switch(Category.categorize(input)) {
                    case MONEY:
                        amount += input.amount();
                        break;
                    case ITEM_SELECTION:
                        selection = input;
                        if(amount < selection.amount())
                            System.out.println(
                                    "Insufficient money for " + selection);
                        else state = DISPENSING;
                        break;
                    case QUIT_TRANSACTION:
                        state = GIVING_CHANGE;
                        break;
                    case SHUT_DOWN:
                        state = TERMINAL;
                    default:
                }
            }
        },
        DISPENSING(StateDuration.TRANSIENT) {
            @Override
            void next() {
                System.out.println("here is your " + selection);
                amount -= selection.amount();
                state = GIVING_CHANGE;
            }
        },
        GIVING_CHANGE(StateDuration.TRANSIENT) {
            @Override
            void next() {
                if(amount > 0) {
                    System.out.println("Your change: " + amount);
                    amount = 0;
                }
                state = RESTING;
            }
        },
        TERMINAL {@Override
        void output() { System.out.println("Halted"); } };
        private boolean isTransient = false;
        State() {}
        State(StateDuration trans) { isTransient = true; }
        void next(Input input) {
            throw new RuntimeException("Only call " +
                    "next(Input input) for non-transient states");
        }
        void next() {
            throw new RuntimeException(
                    "Only call next() for " +
                            "StateDuration.TRANSIENT states");
        }
        void output() { System.out.println(amount); }
    }
    static void run(Supplier<Input> gen) {
        while(state != State.TERMINAL) {
            state.next(gen.get());
            while(state.isTransient)
                state.next();
            state.output();
        }
    }
    public static void main(String[] args) {
        Supplier<Input> gen = new RandomInputSupplier();
        if(args.length == 1)
            gen = new FileInputSupplier(args[0]);
        run(gen);
    }
}

// For a basic sanity check:
class RandomInputSupplier implements Supplier<Input> {
    @Override
    public Input get() {
        return Input.randomSelection();
    }
}

// Create Inputs from a file of ';'-separated strings:
class FileInputSupplier implements Supplier<Input> {
    private Iterator<String> input;
    FileInputSupplier(String fileName) {
        try {
            input = Files.lines(Paths.get(fileName))
                    .skip(1) // Skip the comment line
                    .flatMap(s -> Arrays.stream(s.split(";")))
                    .map(String::trim)
                    .collect(Collectors.toList())
                    .iterator();
        } catch(IOException e) {
            throw new RuntimeException(e);
        }
    }
    @Override
    public Input get() {
        if(!input.hasNext())
            return null;
        return Enum.valueOf(Input.class, input.next().trim());
    }
}
```

輸出為：

```
25
50
75
here is your CHIPS
0
100
200
here is your TOOTHPASTE
0
25
35
Your change: 35
0
25
35
Insufficient money for SODA
35
60
70
75
Insufficient money for SODA
75
Your change: 75
0
Halted
```

由於用 switch 語句從 enum 實例中進行選擇是最常見的一種方式（請注意，為了使 enum 在 switch 語句中的使用變得簡單，我們是需要付出其他代價的），所以，我們經常遇到這樣的問題：將多個 enum 進行分類時，“我們希望在什麼 enum 中使用 switch 語句？”我們透過 VendingMachine 的例子來研究一下這個問題。對於每一個 State，我們都需要在輸入動作的基本分類中進行尋找：使用者塞入鈔票，選擇了某個貨物，操作被取消，以及機器停止。然而，在這些基本分類之下，我們又可以塞人不同類型的鈔票，可以選擇不同的貨物。Category enum 將不同類型的 Input 進行分組，因而，可以使用 categorize0 方法為 switch 語句生成恰當的 Cateroy 實例。並且，該方法使用的 EnumMap 確保了在其中進行查詢時的效率與安全。

如果讀者仔細研究 VendingMachine 類，就會發現每種狀態的不同之處，以及對於輸入的不同響應，其中還有兩個瞬時狀態。在 run() 方法中，狀態機等待著下一個 Input，並一直在各個狀態中移動，直到它不再處於瞬時狀態。

透過兩種不同的 Generator 物件，我們可以使用不同的 Supplier 物件來測試 VendingMachine，首先是 RandomInputSupplier，它會不停地生成除了 SHUT-DOWN 之外的各種輸入。透過長時間地執行 RandomInputSupplier，可以起到健全測試（sanity test）的作用，能夠確保該狀態機不會進入一個錯誤狀態。另一個是 FileInputSupplier，使用文件以文字的方式來描述輸入，然後將它們轉換成 enum 實例，並建立對應的 Input 物件。上面的程式使用的正是如下的文字文件：

```
// enums/VendingMachineInput.txt
QUARTER; QUARTER; QUARTER; CHIPS;
DOLLAR; DOLLAR; TOOTHPASTE;
QUARTER; DIME; ABORT_TRANSACTION;
QUARTER; DIME; SODA;
QUARTER; DIME; NICKEL; SODA;
ABORT_TRANSACTION;
STOP;
```

FileInputSupplier 建構子將此文件轉換為流，並跳過注釋行。然後它使用 String.split() 以分號進行分割。這會生成一個 String 陣列，並可以透過將其轉換為 Stream，然後應用 flatMap() 來將其輸入到流中。其輸出結果將去除所有空格空格，並轉換為 List\<String\>，且從中獲取 Iterator\<String\>。

這種設計有一個缺陷，它要求 enum State 實例訪問的 VendingMachine 屬性必須聲明為 static，這意味著，你只能有一個 VendingMachine 實例。不過如果我們思考一下實際的（嵌入式 Java）應用，這也許並不是一個大問題，因為在一台機器上，我們可能只有一個應用程式。

<!-- Multiple Dispatching -->

## 多路分發

當你要處理多種互動類型時，程式可能會變得相當雜亂。舉例來說，如果一個系統要分析和執行數學表達式。我們可能會聲明 Number.plus(Number)，Number.multiple(Number) 等等，其中 Number 是各種數字物件的超類。然而，當你聲明 a.plus(b) 時，你並不知道 a 或 b 的確切類型，那你如何能讓它們正確地互動呢？

你可能從未思考過這個問題的答案.Java 只支援單路分發。也就是說，如果要執行的操作包含了不止一個類型未知的物件時，那麼 Java 的動態綁定機制只能處理其中一個的類型。這就無法解決我們上面提到的問題。所以，你必須自己來判定其他的類型，從而實現自己的動態線定行為。

解決上面問題的辦法就是多路分發（在那個例子中，只有兩個分發，一般稱之為兩路分發）.多態只能發生在方法呼叫時，所以，如果你想使用兩路分發，那麼就必須有兩個方法呼叫：第一個方法呼叫決定第一個未知類型，第二個方法呼叫決定第二個未知的類型。要利用多路分發，程式設計師必須為每一個類型提供一個實際的方法呼叫，如果你要處理兩個不同的類型體系，就需要為每個類型體系執行一個方法呼叫。一般而言，程式設計師需要有設定好的某種配置，以便一個方法呼叫能夠引出更多的方法呼叫，從而能夠在這個過程中處理多種類型。為了達到這種效果，我們需要與多個方法一同工作：因為每個分發都需要一個方法呼叫。在下面的例子中（實現了 “剪刀、石頭、布”遊戲，也稱為 RoShamBo）對應的方法是 compete() 和 eval()，二者都是同一個類型的成員，它們可以產生三種 Outcome 實例中的一個作為結果：

```java
// enums/Outcome.java
package enums;
public enum Outcome { WIN, LOSE, DRAW }
// enums/RoShamBo1.java
// Demonstration of multiple dispatching
// {java enums.RoShamBo1}
package enums;
        import java.util.*;
        import static enums.Outcome.*;
interface Item {
    Outcome compete(Item it);
    Outcome eval(Paper p);
    Outcome eval(Scissors s);
    Outcome eval(Rock r);
}
class Paper implements Item {
    @Override
    public Outcome compete(Item it) {
        return it.eval(this);
    }
    @Override
    public Outcome eval(Paper p) { return DRAW; }
    @Override
    public Outcome eval(Scissors s) { return WIN; }
    @Override
    public Outcome eval(Rock r) { return LOSE; }
    @Override
    public String toString() { return "Paper"; }
}
class Scissors implements Item {
    @Override
    public Outcome compete(Item it) {
        return it.eval(this);
    }
    @Override
    public Outcome eval(Paper p) { return LOSE; }
    @Override
    public Outcome eval(Scissors s) { return DRAW; }
    @Override
    public Outcome eval(Rock r) { return WIN; }
    @Override
    public String toString() { return "Scissors"; }
}
class Rock implements Item {
    @Override
    public Outcome compete(Item it) {
        return it.eval(this);
    }
    @Override
    public Outcome eval(Paper p) { return WIN; }
    @Override
    public Outcome eval(Scissors s) { return LOSE; }
    @Override
    public Outcome eval(Rock r) { return DRAW; }
    @Override
    public String toString() { return "Rock"; }
}
public class RoShamBo1 {
    static final int SIZE = 20;
    private static Random rand = new Random(47);
    public static Item newItem() {
        switch(rand.nextInt(3)) {
            default:
            case 0: return new Scissors();
            case 1: return new Paper();
            case 2: return new Rock();
        }
    }
    public static void match(Item a, Item b) {
        System.out.println(
                a + " vs. " + b + ": " + a.compete(b));
    }
    public static void main(String[] args) {
        for(int i = 0; i < SIZE; i++)
            match(newItem(), newItem());
    }
}
```

輸出為：

```
Rock vs. Rock: DRAW
Paper vs. Rock: WIN
Paper vs. Rock: WIN
Paper vs. Rock: WIN
Scissors vs. Paper: WIN
Scissors vs. Scissors: DRAW
Scissors vs. Paper: WIN
Rock vs. Paper: LOSE
Paper vs. Paper: DRAW
Rock vs. Paper: LOSE
Paper vs. Scissors: LOSE
Paper vs. Scissors: LOSE
Rock vs. Scissors: WIN
Rock vs. Paper: LOSE
Paper vs. Rock: WIN
Scissors vs. Paper: WIN
Paper vs. Scissors: LOSE
Paper vs. Scissors: LOSE
Paper vs. Scissors: LOSE
Paper vs. Scissors: LOSE
```

Item 是這幾種類型的介面，將會被用作多路分發。RoShamBo1.match() 有兩個 Item 參數，透過呼叫 Item.compete90) 方法開始兩路分發。要判定 a 的類型，分發機制會在 a 的實際類型的 compete（內部起到分發的作用。compete() 方法透過呼叫 eval() 來為另一個類型實現第二次分法。

將自身（this）作為參數呼叫 evalo，能夠呼叫重載過的 eval() 方法，這能夠保留第一次分發的類型訊息。當第二次分發完成時，你就能夠知道兩個 Item 物件的具體類型了。

要配置好多路分發需要很多的工序，不過要記住，它的好處在於方法呼叫時的優雅的話法，這避免了在一個方法中判定多個物件的類型的醜陋程式碼，你只需說，“嘿，你們兩個，我不在乎你們是什麼類型，請你們自己交流！”不過，在使用多路分發前，請先明確，這種優雅的程式碼對你確實有重要的意義。

### 使用 enum 分發

直接將 RoShamBol.java 翻譯為基於 enum 的版本是有問題的，因為 enum 實例不是類型，不能將 enum 實例作為參數的類型，所以無法重載 eval() 方法。不過，還有很多方式可以實現多路分發，並從 enum 中獲益。

一種方式是使用構造器來初始化每個 enum 實例，並以“一組”結果作為參數。這二者放在一塊，形成了類似查詢表的結構：

```java
// enums/RoShamBo2.java
// Switching one enum on another
// {java enums.RoShamBo2}
package enums;
import static enums.Outcome.*;
public enum RoShamBo2 implements Competitor<RoShamBo2> {
    PAPER(DRAW, LOSE, WIN),
    SCISSORS(WIN, DRAW, LOSE),
    ROCK(LOSE, WIN, DRAW);
    private Outcome vPAPER, vSCISSORS, vROCK;
    RoShamBo2(Outcome paper,
              Outcome scissors, Outcome rock) {
        this.vPAPER = paper;
        this.vSCISSORS = scissors;
        this.vROCK = rock;
    }
    @Override
    public Outcome compete(RoShamBo2 it) {
        switch(it) {
            default:
            case PAPER: return vPAPER;
            case SCISSORS: return vSCISSORS;
            case ROCK: return vROCK;
        }
    }
    public static void main(String[] args) {
        RoShamBo.play(RoShamBo2.class, 20);
    }
}
```

輸出為：

```
ROCK vs. ROCK: DRAW
SCISSORS vs. ROCK: LOSE
SCISSORS vs. ROCK: LOSE
SCISSORS vs. ROCK: LOSE
PAPER vs. SCISSORS: LOSE
PAPER vs. PAPER: DRAW
PAPER vs. SCISSORS: LOSE
ROCK vs. SCISSORS: WIN
SCISSORS vs. SCISSORS: DRAW
ROCK vs. SCISSORS: WIN
SCISSORS vs. PAPER: WIN
SCISSORS vs. PAPER: WIN
ROCK vs. PAPER: LOSE
ROCK vs. SCISSORS: WIN
SCISSORS vs. ROCK: LOSE
PAPER vs. SCISSORS: LOSE
SCISSORS vs. PAPER: WIN
SCISSORS vs. PAPER: WIN
SCISSORS vs. PAPER: WIN
SCISSORS vs. PAPER: WIN
```

在 compete() 方法中，一旦兩種類型都被確定了，那麼唯一的操作就是返回結果 Outcome 然而，你可能還需要呼叫其他的方法，（例如）甚至是呼叫在構造器中指定的某個指令物件上的方法。

RoShamBo2.javal 之前的例子短小得多，而且更直接，更易於理解。注意，我們仍然是使用兩路分發來判定兩個物件的類型。在 RoShamBol.java 中，兩次分發都是透過實際的方法呼叫實現，而在這個例子中，只有第一次分發是實際的方法呼叫。第二個分發使用的是 switch，不過這樣做是安全的，因為 enum 限制了 switch 語句的選擇分支。

在程式碼中，enum 被單獨抽取出來，因此它可以應用在其他例子中。首先，Competitor 介面定義了一種類型，該類型的物件可以與另一個 Competitor 相競爭：

```java
// enums/Competitor.java
// Switching one enum on another
package enums;
public interface Competitor<T extends Competitor<T>> {
    Outcome compete(T competitor);
}
```

然後，我們定義兩個 static 方法（static 可以避免顯式地指明參數類型），第一個是 match() 方法，它會為一個 Competitor 物件呼叫 compete() 方法，並與另一個 Competitor 物件作比較。在這個例子中，我們看到，match()方法的參數需要是 Competitor\<T\> 類型。但是在 play() 方法中，類型參數必須同時是 Enum\<T\> 類型（因為它將在 Enums.random() 中使用）和 Competitor\<T\> 類型（因為它將被傳遞給 match() 方法）：

```java
// enums/RoShamBo.java
// Common tools for RoShamBo examples
package enums;
import onjava.*;
public class RoShamBo {
    public static <T extends Competitor<T>>
    void match(T a, T b) {
        System.out.println(
                a + " vs. " + b + ": " + a.compete(b));
    }
    public static <T extends Enum<T> & Competitor<T>>
    void play(Class<T> rsbClass, int size) {
        for(int i = 0; i < size; i++)
            match(Enums.random(rsbClass),Enums.random(rsbClass));
    }
}
```

play() 方法沒有將類型參數 T 作為返回值類型，因此，似乎我們應該在 Class\<T\> 中使用萬用字元來代替上面的參數聲明。然而，萬用字元不能擴展多個基類，所以我們必須採用以上的表達式。

### 使用常量相關的方法

常量相關的方法允許我們為每個 enum 實例提供方法的不同實現，這使得常量相關的方法似乎是實現多路分發的完美解決方案。不過，透過這種方式，enum 實例雖然可以具有不同的行為，但它們仍然不是類型，不能將其作為方法簽名中的參數類型來使用。最好的辦法是將 enum 用在 switch 語句中，見一下例：

```java
// enums/RoShamBo3.java
// Using constant-specific methods
// {java enums.RoShamBo3}
package enums;
import static enums.Outcome.*;
public enum RoShamBo3 implements Competitor<RoShamBo3> {
    PAPER {
        @Override
        public Outcome compete(RoShamBo3 it) {
            switch(it) {
                default: // To placate the compiler
                case PAPER: return DRAW;
                case SCISSORS: return LOSE;
                case ROCK: return WIN;
            }
        }
    },
    SCISSORS {
        @Override
        public Outcome compete(RoShamBo3 it) {
            switch(it) {
                default:
                case PAPER: return WIN;
                case SCISSORS: return DRAW;
                case ROCK: return LOSE;
            }
        }
    },
    ROCK {
        @Override
        public Outcome compete(RoShamBo3 it) {
            switch(it) {
                default:
                case PAPER: return LOSE;
                case SCISSORS: return WIN;
                case ROCK: return DRAW;
            }
        }
    };
    @Override
    public abstract Outcome compete(RoShamBo3 it);
    public static void main(String[] args) {
        RoShamBo.play(RoShamBo3.class, 20);
    }
}
```

輸出為：

```
ROCK vs. ROCK: DRAW
SCISSORS vs. ROCK: LOSE
SCISSORS vs. ROCK: LOSE
SCISSORS vs. ROCK: LOSE
PAPER vs. SCISSORS: LOSE
PAPER vs. PAPER: DRAW
PAPER vs. SCISSORS: LOSE
ROCK vs. SCISSORS: WIN
SCISSORS vs. SCISSORS: DRAW
ROCK vs. SCISSORS: WIN
SCISSORS vs. PAPER: WIN
SCISSORS vs. PAPER: WIN
ROCK vs. PAPER: LOSE
ROCK vs. SCISSORS: WIN
SCISSORS vs. ROCK: LOSE
PAPER vs. SCISSORS: LOSE
SCISSORS vs. PAPER: WIN
SCISSORS vs. PAPER: WIN
SCISSORS vs. PAPER: WIN
SCISSORS vs. PAPER: WIN
```

雖然這種方式可以工作，但是卻不甚合理，如果採用 RoShamB02.java 的解決方案，那麼在添加一個新的類型時，只需更少的程式碼，而且也更直接。

:然而，RoShamBo3.java 還可以壓縮簡化一下：

```java
// enums/RoShamBo4.java
// {java enums.RoShamBo4}
package enums;
public enum RoShamBo4 implements Competitor<RoShamBo4> {
    ROCK {
        @Override
        public Outcome compete(RoShamBo4 opponent) {
            return compete(SCISSORS, opponent);
        }
    },
    SCISSORS {
        @Override
        public Outcome compete(RoShamBo4 opponent) {
            return compete(PAPER, opponent);
        }
    },
    PAPER {
        @Override
        public Outcome compete(RoShamBo4 opponent) {
            return compete(ROCK, opponent);
        }
    };
    Outcome compete(RoShamBo4 loser, RoShamBo4 opponent) {
        return ((opponent == this) ? Outcome.DRAW
                : ((opponent == loser) ? Outcome.WIN
                : Outcome.LOSE));
    }
    public static void main(String[] args) {
        RoShamBo.play(RoShamBo4.class, 20);
    }
}
```

輸出為：

```
PAPER vs. PAPER: DRAW
SCISSORS vs. PAPER: WIN
SCISSORS vs. PAPER: WIN
SCISSORS vs. PAPER: WIN
ROCK vs. SCISSORS: WIN
ROCK vs. ROCK: DRAW
ROCK vs. SCISSORS: WIN
PAPER vs. SCISSORS: LOSE
SCISSORS vs. SCISSORS: DRAW
PAPER vs. SCISSORS: LOSE
SCISSORS vs. ROCK: LOSE
SCISSORS vs. ROCK: LOSE
PAPER vs. ROCK: WIN
PAPER vs. SCISSORS: LOSE
SCISSORS vs. PAPER: WIN
ROCK vs. SCISSORS: WIN
SCISSORS vs. ROCK: LOSE
SCISSORS vs. ROCK: LOSE
SCISSORS vs. ROCK: LOSE
SCISSORS vs. ROCK: LOSE
```

其中，具有兩個參數的 compete() 方法執行第二個分發，該方法執行一系列的比較，其行為類似 switch 語句。這個版本的程式更簡短，不過卻比較難理解，對於一個大型系統而言，難以理解的程式碼將導致整個系統不夠健壯。

### 使用 EnumMap 進行分發

使用 EnumMap 能夠實現“真正的”兩路分發。EnumMap 是為 enum 專門設計的一種效能非常好的特殊 Map。由於我們的目的是摸索出兩種未知的類型，所以可以用一個 EnumMap 的 EnumMap 來實現兩路分發：

```java
// enums/RoShamBo5.java
// Multiple dispatching using an EnumMap of EnumMaps
// {java enums.RoShamBo5}
package enums;
import java.util.*;
import static enums.Outcome.*;
enum RoShamBo5 implements Competitor<RoShamBo5> {
    PAPER, SCISSORS, ROCK;
    static EnumMap<RoShamBo5,EnumMap<RoShamBo5,Outcome>>
            table = new EnumMap<>(RoShamBo5.class);
    static {
        for(RoShamBo5 it : RoShamBo5.values())
            table.put(it, new EnumMap<>(RoShamBo5.class));
        initRow(PAPER, DRAW, LOSE, WIN);
        initRow(SCISSORS, WIN, DRAW, LOSE);
        initRow(ROCK, LOSE, WIN, DRAW);
    }
    static void initRow(RoShamBo5 it,
                        Outcome vPAPER, Outcome vSCISSORS, Outcome vROCK) {
        EnumMap<RoShamBo5,Outcome> row =
                RoShamBo5.table.get(it);
        row.put(RoShamBo5.PAPER, vPAPER);
        row.put(RoShamBo5.SCISSORS, vSCISSORS);
        row.put(RoShamBo5.ROCK, vROCK);
    }
    @Override
    public Outcome compete(RoShamBo5 it) {
        return table.get(this).get(it);
    }
    public static void main(String[] args) {
        RoShamBo.play(RoShamBo5.class, 20);
    }
}
```

輸出為：

```
ROCK vs. ROCK: DRAW
SCISSORS vs. ROCK: LOSE
SCISSORS vs. ROCK: LOSE
SCISSORS vs. ROCK: LOSE
PAPER vs. SCISSORS: LOSE
PAPER vs. PAPER: DRAW
PAPER vs. SCISSORS: LOSE
ROCK vs. SCISSORS: WIN
SCISSORS vs. SCISSORS: DRAW
ROCK vs. SCISSORS: WIN
SCISSORS vs. PAPER: WIN
SCISSORS vs. PAPER: WIN
ROCK vs. PAPER: LOSE
ROCK vs. SCISSORS: WIN
SCISSORS vs. ROCK: LOSE
PAPER vs. SCISSORS: LOSE
SCISSORS vs. PAPER: WIN
SCISSORS vs. PAPER: WIN
SCISSORS vs. PAPER: WIN
SCISSORS vs. PAPER: WIN
```

該程式在一個 static 子句中初始化 EnumMap 物件，具體見表格似的 initRow() 方法呼叫。請注意 compete() 方法，您可以看到，在一行語句中發生了兩次分發。

### 使用二維陣列

我們還可以進一步簡化實現兩路分發的解決方案。我們注意到，每個 enum 實例都有一個固定的值（基於其聲明的次序），並且可以透過 ordinal() 方法取得該值。因此我們可以使用二維陣列，將競爭者映射到競爭結果。採用這種方式能夠獲得最簡潔、最直接的解決方案（很可能也是最快速的，雖然我們知道 EnumMap 內部其實也是使用陣列實現的）。

```java
We can simplify the solution even more by noting that each  enum instance has a fixed
        value (based on its declaration order) and that  ordinal() produces this value. A two-
        dimensional array mapping the competitors onto the outcomes produces the smallest
        and most straightforward solution (and possibly the fastest, although remember that
        EnumMap uses an internal array):
// enums/RoShamBo6.java
// Enums using "tables" instead of multiple dispatch
// {java enums.RoShamBo6}
        package enums;
        import static enums.Outcome.*;
enum RoShamBo6 implements Competitor<RoShamBo6> {
    PAPER, SCISSORS, ROCK;
    private static Outcome[][] table = {
            { DRAW, LOSE, WIN }, // PAPER
            { WIN, DRAW, LOSE }, // SCISSORS
            { LOSE, WIN, DRAW }, // ROCK
    };
    @Override
    public Outcome compete(RoShamBo6 other) {
        return table[this.ordinal()][other.ordinal()];
    }
    public static void main(String[] args) {
        RoShamBo.play(RoShamBo6.class, 20);
    }
}
```

輸出為：

```
ROCK vs. ROCK: DRAW
SCISSORS vs. ROCK: LOSE
SCISSORS vs. ROCK: LOSE
SCISSORS vs. ROCK: LOSE
PAPER vs. SCISSORS: LOSE
PAPER vs. PAPER: DRAW
PAPER vs. SCISSORS: LOSE
ROCK vs. SCISSORS: WIN
SCISSORS vs. SCISSORS: DRAW
ROCK vs. SCISSORS: WIN
SCISSORS vs. PAPER: WIN
SCISSORS vs. PAPER: WIN
ROCK vs. PAPER: LOSE
ROCK vs. SCISSORS: WIN
SCISSORS vs. ROCK: LOSE
PAPER vs. SCISSORS: LOSE
SCISSORS vs. PAPER: WIN
SCISSORS vs. PAPER: WIN
SCISSORS vs. PAPER: WIN
SCISSORS vs. PAPER: WIN
```

table 與前一個例子中 initRow() 方法的呼叫次序完全相同。

與前面一個例子相比，這個程式碼雖然簡短，但表達能力卻更強，部分原因是其程式碼更易於理解與修改，而且也更直接。不過，由於它使用的是陣列，所以這種方式不太“安全”。如果使用一個大型陣列，可能會不小心使用了錯誤的尺寸，而且，如果你的測試不能覆蓋所有的可能性，有些錯誤可能會從你眼前溜過。

事實上，以上所有的解決方案只是各種不同類型的表罷了。不過，分析各種表的表現形式，找出最適合的那一種，還是很有價值的。注意，雖然上例是最簡潔的一種解決方案，但它也是相當僵硬的方案，因為它只能針對給定的常量輸入產生常量輸出。然而，也沒有什麼特別的理由阻止你用 table 來生成功能物件。對於某類問題而言，“表驅動式編碼”的概念具有非常強大的功能。

<!-- Summary -->

## 本章小結

雖然列舉類型本身並不是特別複雜，但我還是將本章安排在全書比較靠後的位置，這是因為，程式設計師可以將 enum 與 Java 語言的其他功能結合使用，例如多態、泛型和反射。

雖然 Java 中的列舉比 C 或 C++中的 enum 更成熟，但它仍然是一個“小”功能，Java 沒有它也已經（雖然有點笨拙）存在很多年了。而本章正好說明了一個“小”功能所能帶來的價值。有時恰恰因為它，你才能夠優雅而乾淨地解決問題。正如我在本書中一再強調的那樣，優雅與清晰很重要，正是它們區別了成功的解決方案與失敗的解決方案。而失敗的解決方案就是因為其他人無法理解它。

關於清晰的話題，Java 1.0 對術語 enumeration 的選擇正是一個不幸的反例。對於一個專門用於從序列中選擇每一個元素的物件而言，Java 竟然沒有使用更通用、更普遍接受的術語 iterator 來表示它（參見[集合 ]() 章節），有些語言甚至將列舉的資料類型稱為 “enumerators”！Java 修正了這個錯誤，但是 Enumeration 介面已經無法輕易地抹去了，因此它將一直存在於舊的（甚至有些新的）程式碼、類庫以及文件中。



<!-- 分頁 -->

<div style="page-break-after: always;"></div>
