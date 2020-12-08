[TOC]

<!-- Patterns -->
# 第二十五章 設計模式


<!-- The Pattern Concept -->
## 概念
最初，你可以將模式視為解決特定類問題的一種特別巧妙且有深刻見解的方法。這就像前輩已經從所有角度去解決問題，並提出了最通用，最靈活的解決方案。問題可能是你之前看到並解決過的問題，但你的解決方案可能沒有你在模式中體現的那種完整性。

雖然它們被稱為“設計模式”，但它們實際上並不與設計領域相關聯。模式似乎與傳統的分析、設計和實現的思維方式不同。相反，模式在程式中體現了一個完整的思想，因此它有時會出現在分析階段或進階設計階段。因為模式在程式碼中有一個直接的實現，所以你可能不會期望模式在低級設計或實現之前出現(而且通常在到達這些階段之前，你不會意識到需要一個特定的模式)。

模式的基本概念也可以看作是程式設計的基本概念:添加抽象層。當你抽象一些東西的時候，就像在剝離特定的細節，而這背後最重要的動機之一是:
> **將易變的事物與不變的事物分開**

另一種方法是，一旦你發現程式的某些部分可能因某種原因而發生變化，你要保持這些變化不會引起整個程式碼中其他變化。 如果程式碼更容易理解，那麼維護起來會更容易。

通常，開發一個優雅且易維護設計中最困難的部分是發現我稱之為變化的載體（也就是最易改變的地方）。這意味著找到系統中最重要的變化，換而言之，找到變化會導致最嚴重後果的地方。一旦發現變化載體，就可以圍繞構建設計的焦點。

因此，設計模式的目標是隔離程式碼中的更改。 如果以這種方式去看，你已經在本書中看到了設計模式。 例如，繼承可以被認為是一種設計模式（雖然是由編譯器實現的）。它允許你表達所有具有相同介面的物件（即保持相同的行為）中的行為差異（這就是變化的部分）。組合也可以被視為一種模式，因為它允許你動態或靜態地更改實現類的物件，從而改變類的工作方式。

你還看到了設計模式中出現的另一種模式：疊代器（Java 1.0和1.1隨意地將其稱為列舉; Java 2 集合才使用Iterator）。當你逐個選擇元素時並逐步處理，這會隱藏集合的特定實現。疊代器允許你編寫通用程式碼，該程式碼對序列中的所有元素執行操作，而不考慮序列的構建方式。因此，你的通用程式碼可以與任何可以生成疊代器的集合一起使用。

即使模式是非常有用的，但有些人斷言：
> **設計模式代表語言的失敗。**

這是一個非常重要的見解，因為一個模式在 C++ 有意義，可能在JAVA或者其他語言中就沒有意義。出於這個原因，所以一個模式可能出現在設計模式書上，不意味著應用於你的程式語言是有用的。

我認為“語言失敗”這個觀點是有道理的，但是我也認為這個觀點過於簡單化。如果你試圖解決一個特定的問題，而你使用的語言沒有直接提供支持你使用的技巧，你可以說這個是語言的失敗。但是，你使用特定的技巧的頻率的是多少呢？也許平衡是對的：當你使用特定的技巧的時候，你必須付出更多的努力，但是你又沒有足夠的理由去使得語言支援這個技術。另一方面，沒有語言的支援，使用這種技術常常會很混亂，但是在語言支援下，你可能會改變編程方式（例如，Java 8流實現此目的）。

### 單例模式
也許單例模式是最簡單的設計模式，它是一種提供一個且只有一個物件實例的方法。這在java庫中使用，但是這有個更直接的範例：

```java
// patterns/SingletonPattern.java
interface Resource {
    int getValue();
    void setValue(int x);
}

/*
* 由於這不是從Cloneable基類繼承而且沒有添加可複製性，
* 因此將其設定為final可防止透過繼承添加可複製性。
* 這也實現了執行緒安全的延遲初始化：
*/
final class Singleton {
    private static final class ResourceImpl implements Resource {
        private int i;
        private ResourceImpl(int i) {
            this.i = i;
        }
        public synchronized int getValue() {
            return i;
        }
        public synchronized void setValue(int x) {
            i = x;
        }
    }

    private static class ResourceHolder {
        private static Resource resource = new ResourceImpl(47);
    }
    public static Resource getResource() {
        return ResourceHolder.resource;
    }
}

public class SingletonPattern {
    public static void main(String[] args) {
        Resource r = Singleton.getResource();
        System.out.println(r.getValue());
        Resource s2 = Singleton.getResource();
        s2.setValue(9);
        System.out.println(r.getValue());
        try {     
             // 不能這麼做，會發生：compile-time error（編譯時錯誤）.     
             // Singleton s3 = (Singleton)s2.clone();    
             } catch(Exception e) {      
                 throw new RuntimeException(e);    
             }  
        }
} /* Output: 47 9 */
```
建立單例的關鍵是防止用戶端程式設計師直接建立物件。 在這裡，這是透過在Singleton類中將Resource的實現作為私有類來實現的。

此時，你將決定如何建立物件。在這裡，它是按需建立的，在第一次訪問的時候建立。 該物件是私有的，只能透過public getResource（）方法訪問。


懶惰地建立物件的原因是它嵌套的私有類resourceHolder在首次引用之前不會載入（在getResource（）中）。當Resource物件載入的時候，靜態初始化塊將被呼叫。由於JVM的工作方式，這種靜態初始化是執行緒安全的。為保證執行緒安全，Resource中的getter和setter是同步的。

### 模式分類

“設計模式”一書討論了23種不同的模式，分為以下三種類別（所有這些模式都圍繞著可能變化的特定方面）。

1. **建立型**：如何建立物件。 這通常涉及隔離物件建立的細節，這樣你的程式碼就不依賴於具體的物件的類型，因此在添加新類型的物件時不會更改。單例模式（Singleton）被歸類為創作模式，本章稍後你將看到Factory Method的範例。

2. **構造型**：設計物件以滿足特定的項目約束。它們處理物件與其他物件連接的方式，以確保系統中的更改不需要更改這些連接。

3. **行為型**：處理程序中特定類型的操作的物件。這些封裝要執行的過程，例如解釋語言、實現請求、遍歷序列(如在疊代器中)或實現演算法。本章包含觀察者和訪問者模式的例子。

《設計模式》一書中每個設計模式都有單獨的一個章節，每個章節都有一個或者多個例子，通常使用C++，但有時也使用SmallTalk。 本章不重複設計模式中顯示的所有模式，因為該書獨立存在，應單獨研究。 相反，你會看到一些範例，可以為你提供關於模式的理解以及它們如此重要的原因。

<!-- Building Application Frameworks -->
## 構建應用程式框架

應用程式框架允許您從一個類或一組類開始，建立一個新的應用程式，重用現有類中的大部分程式碼，並根據需要覆蓋一個或多個方法來訂製應用程式。

**模板方法模式**

應用程式框架中的一個基本概念是模板方法模式，它通常隱藏在底層，透過呼叫基類中的各種方法來驅動應用程式(為了建立應用程式，您已經覆蓋了其中的一些方法)。

模板方法模式的一個重要特性是它是在基類中定義的，並且不能更改。它有時是一個 **private** 方法，但實際上總是 **final**。它呼叫其他基類方法(您覆蓋的那些)來完成它的工作,但是它通常只作為初始化過程的一部分被呼叫(因此框架使用者不一定能夠直接呼叫它)。

```Java
// patterns/TemplateMethod.java
// Simple demonstration of Template Method

abstract class ApplicationFramework {
    ApplicationFramework() {
        templateMethod();
    }

    abstract void customize1();

    abstract void customize2();

    // "private" means automatically "final":
    private void templateMethod() {
        IntStream.range(0, 5).forEach(
                n -> {
                    customize1();
                    customize2();
                });
    }
}

// Create a new "application":
class MyApp extends ApplicationFramework {
    @Override
    void customize1() {
        System.out.print("Hello ");
    }

    @Override
    void customize2() {
        System.out.println("World!");
    }
}

public class TemplateMethod {
    public static void main(String[] args) {
        new MyApp();
    }
}
/* Output:
Hello World!
Hello World!
Hello World!
Hello World!
Hello World!
*/
```

基類建構子負責執行必要的初始化，然後啟動執行應用程式的“engine”(模板方法模式)(在GUI應用程式中，這個“engine”是主事件循環)。框架使用者只提供
**customize1()** 和 **customize2()** 的定義，然後“應用程式”已經就緒執行。

![](images/designproxy.png)

<!-- Fronting for an Implementation -->
## 面向實現

代理模式和橋接模式都提供了在程式碼中使用的代理類;完成工作的真正類隱藏在這個代理類的後面。當您在代理中呼叫一個方法時，它只是反過來呼叫實現類中的方法。這兩種模式非常相似，所以代理模式只是橋接模式的一種特殊情況。人們傾向於將兩者合併,稱為代理模式，但是術語“代理”有一個長期的和專門的含義，這可能解釋了這兩種模式不同的原因。基本思想很簡單:從基類衍生代理，同時衍生一個或多個提供實現的類:建立代理物件時，給它一個可以呼叫實際工作類的方法的實現。


在結構上，代理模式和橋接模式的區別很簡單:代理模式只有一個實現，而橋接模式有多個實現。在設計模式中被認為是不同的:代理模式用於控制對其實現的訪問，而橋接模式允許您動態更改實現。但是，如果您擴展了“控制對實現的訪問”的概念，那麼這兩者就可以完美地結合在一起

**代理模式**

如果我們按照上面的關係圖實現，它看起來是這樣的:

```Java
// patterns/ProxyDemo.java
// Simple demonstration of the Proxy pattern
interface ProxyBase {
    void f();

    void g();

    void h();
}

class Proxy implements ProxyBase {
    private ProxyBase implementation;

    Proxy() {
        implementation = new Implementation();
    }
    // Pass method calls to the implementation:
    @Override
    public void f() { implementation.f(); }
    @Override
    public void g() { implementation.g(); }
    @Override
    public void h() { implementation.h(); }
}

class Implementation implements ProxyBase {
    public void f() {
        System.out.println("Implementation.f()");
    }

    public void g() {
        System.out.println("Implementation.g()");
    }

    public void h() {
        System.out.println("Implementation.h()");
    }
}

public class ProxyDemo {
    public static void main(String[] args) {
        Proxy p = new Proxy();
        p.f();
        p.g();
        p.h();
    }
}
/*
Output:
Implementation.f()
Implementation.g()
Implementation.h()
*/
```

具體實現不需要與代理物件具有相同的介面;只要代理物件以某種方式“代表具體實現的方法呼叫，那麼基本思想就算實現了。然而，擁有一個公共介面是很方便的，因此具體實現必須實現代理物件呼叫的所有方法。

**狀態模式**

狀態模式向代理物件添加了更多的實現，以及在代理物件的生命週期內從一個實現切換到另一種實現的方法:

```Java
// patterns/StateDemo.java // Simple demonstration of the State pattern
interface StateBase {
    void f();

    void g();

    void h();

    void changeImp(StateBase newImp);
}

class State implements StateBase {
    private StateBase implementation;

    State(StateBase imp) {
        implementation = imp;
    }

    @Override
    public void changeImp(StateBase newImp) {
        implementation = newImp;
    }

    // Pass method calls to the implementation:
    @Override
    public void f() {
        implementation.f();
    }

    @Override
    public void g() {
        implementation.g();
    }

    @Override
    public void h() {
        implementation.h();
    }
}

class Implementation1 implements StateBase {
    @Override
    public void f() {
        System.out.println("Implementation1.f()");
    }

    @Override
    public void g() {
        System.out.println("Implementation1.g()");
    }

    @Override
    public void h() {
        System.out.println("Implementation1.h()");
    }

    @Override
    public void changeImp(StateBase newImp) {
    }
}

class Implementation2 implements StateBase {
    @Override
    public void f() {
        System.out.println("Implementation2.f()");
    }

    @Override
    public void g() {
        System.out.println("Implementation2.g()");
    }

    @Override
    public void h() {
        System.out.println("Implementation2.h()");
    }

    @Override
    public void changeImp(StateBase newImp) {
    }
}

public class StateDemo {
    static void test(StateBase b) {
        b.f();
        b.g();
        b.h();
    }

    public static void main(String[] args) {
        StateBase b =
                new State(new Implementation1());
        test(b);
        b.changeImp(new Implementation2());
        test(b);
    }
}
/* Output:
Implementation1.f()
Implementation1.g()
Implementation1.h()
Implementation2.f()
Implementation2.g()
Implementation2.h()
*/
```

在main()中，首先使用第一個實現，然後改變成第二個實現。代理模式和狀態模式的區別在於它們解決的問題。設計模式中描述的代理模式的常見用途如下:

1. 遠端代理。它在不同的地址空間中代理物件。遠端方法呼叫(RMI)編譯器rmic會自動為您建立一個遠端代理。

2. 虛擬代理。這提供了“懶載入”來根據需要建立“昂貴”的物件。

3. 保護代理。當您希望對代理物件有權限訪問控制時使用。

4. 智慧引用。要在被代理的物件被訪問時添加其他操作。例如，跟蹤特定物件的引用數量，來實現寫時複製用法，和防止物件別名。一個更簡單的例子是跟蹤特定方法的呼叫數量。您可以將Java引用視為一種保護代理，因為它控制在堆上實例物件的訪問(例如，確保不使用空引用)。

在設計模式中，代理模式和橋接模式並不是相互關聯的，因為它們被賦予(我認為是任意的)不同的結構。橋接模式,特別是使用一個單獨的實現，但這似乎對我來說是不必要的,除非你確定該實現是你無法控制的(當然有可能，但是如果您編寫所有程式碼，那麼沒有理由不從單基類的優雅中受益)。此外，只要代理物件控制對其“前置”物件的訪問，代模式理就不需要為其實現使用相同的基類。不管具體情況如何，在代理模式和橋接模式中，代理物件都將方法呼叫傳遞給具體實現物件。

**狀態機**

橋接模式允許程式設計師更改實現，狀態機利用一個結構來自動地將實現更改到下一個。目前實現表示系統所處的狀態，系統在不同狀態下的行為不同(因為它使用橋接模式)。基本上，這是一個利用物件的“狀態機”。將系統從一種狀態移動到另一種狀態的程式碼通常是模板方法模式，如下例所示:

```Java
// patterns/state/StateMachineDemo.java
// The StateMachine pattern and Template method
// {java patterns.state.StateMachineDemo}
package patterns.state;

import onjava.Nap;

interface State {
    void run();
}

abstract class StateMachine {
    protected State currentState;

    protected abstract boolean changeState();

    // Template method:
    protected final void runAll() {
        while (changeState()) // Customizable
            currentState.run();
    }
}

// A different subclass for each state:

class Wash implements State {
    @Override
    public void run() {
        System.out.println("Washing");
        new Nap(0.5);
    }
}

class Spin implements State {
    @Override
    public void run() {
        System.out.println("Spinning");
        new Nap(0.5);
    }
}

class Rinse implements State {
    @Override
    public void run() {
        System.out.println("Rinsing");
        new Nap(0.5);
    }
}

class Washer extends StateMachine {
    private int i = 0;
    // The state table:
    private State[] states = {
            new Wash(), new Spin(),
            new Rinse(), new Spin(),
    };

    Washer() {
        runAll();
    }

    @Override
    public boolean changeState() {
        if (i < states.length) {
            // Change the state by setting the
            // surrogate reference to a new object:
            currentState = states[i++];
            return true;
        } else
            return false;
    }
}

public class StateMachineDemo {
    public static void main(String[] args) {
        new Washer();
    }
}
/* Output:
Washing
Spinning
Rinsing
Spinning
*/
```

在這裡，控制狀態的類(本例中是狀態機)負責決定下一個狀態。然而，狀態物件本身也可以決定下一步移動到什麼狀態，通常基於系統的某種輸入。這是更靈活的解決方案。


<!-- Factories: Encapsulating Object Creation -->
## 工廠模式

當你發現必須將新類型添加到系統中時，合理的第一步是使用多態性為這些新類型建立一個通用介面。這會將你系統中的其餘程式碼與要添加的特定類型的訊息分開，使得可以在不改變現有程式碼的情況下添加新類型……或者看起來如此。起初，在這種設計中，似乎你必須更改程式碼的唯一地方就是你繼承新類型的地方，但這並不是完全正確的。 你仍然必須建立新類型的物件，並且在建立時必須指定要使用的確切構造器。因此，如果建立物件的程式碼分布在整個應用程式中，那麼在添加新類型時，你將遇到相同的問題——你仍然必須追查你程式碼中新類型礙事的所有地方。恰好是類型的建立礙事，而不是類型的使用（透過多態處理），但是效果是一樣的：添加新類型可能會引起問題。

解決方案是強制物件的建立都透過通用工廠進行，而不是允許建立程式碼在整個系統中傳播。 如果你程式中的所有程式碼都必須執行透過該工廠建立你的一個物件，那麼在添加新類時只需要修改工廠即可。

由於每個物件導向的程式都會建立物件，並且很可能會透過添加新類型來擴展程式，因此工廠是最通用的設計模式之一。

舉例來說，讓我們重新看一下**Shape**系統。 首先，我們需要一個用於所有範例的基本框架。 如果無法建立**Shape**物件，則需要拋出一個合適的異常：

```java
// patterns/shapes/BadShapeCreation.java package patterns.shapes;
public class BadShapeCreation extends RuntimeException {
    public BadShapeCreation(String msg) {
        super(msg);
    }
}
```

接下來，是一個**Shape**基類：

```java
// patterns/shapes/Shape.java
package patterns.shapes;
public class Shape {
	private static int counter = 0;
    private int id = counter++;
    @Override
    public String toString(){
        return getClass().getSimpleName() + "[" + id + "]";
    }
    public void draw() {
        System.out.println(this + " draw");
    }
    public void erase() {
        System.out.println(this + " erase");
    }
}
```

該類自動為每一個**Shape**物件建立一個唯一的`id`。

`toString()`使用執行期訊息來發現特定的**Shape**子類的名字。

現在我們能很快建立一些**Shape**子類了：

```java
// patterns/shapes/Circle.java
package patterns.shapes;
public class Circle extends Shape {}
```

```java
// patterns/shapes/Square.java
package patterns.shapes;
public class Square extends Shape {}
```

```java
// patterns/shapes/Triangle.java
package patterns.shapes;
public class Triangle extends Shape {} 
```

工廠是具有能夠建立物件的方法的類。 我們有幾個範例版本，因此我們將定義一個介面：

```java
// patterns/shapes/FactoryMethod.java
package patterns.shapes;
public interface FactoryMethod {
    Shape create(String type);
}
```

`create()`接收一個參數，這個參數使其決定要建立哪一種**Shape**物件，這裡是`String`，但是它其實可以是任何資料集合。物件的初始化資料（這裡是字串）可能來自系統外部。 這個例子將測試工廠：

```java
// patterns/shapes/FactoryTest.java
package patterns.shapes;
import java.util.stream.*;
public class FactoryTest {
    public static void test(FactoryMethod factory) {
        Stream.of("Circle", "Square", "Triangle",
                  "Square", "Circle", "Circle", "Triangle")
        .map(factory::create)
        .peek(Shape::draw)
        .peek(Shape::erase)
        .count(); // Terminal operation
    }
} 
```

在主函數`main()`裡，要記住除非你在最後使用了一個終結操作，否則**Stream**不會做任何事情。在這裡，`count()`的值被丟棄了。

建立工廠的一種方法是顯式建立每種類型：

```java
// patterns/ShapeFactory1.java
// A simple static factory method
import java.util.*;
import java.util.stream.*;
import patterns.shapes.*;
public class ShapeFactory1 implements FactoryMethod {
    public Shape create(String type) {
        switch(type) {
            case "Circle": return new Circle();
            case "Square": return new Square();
            case "Triangle": return new Triangle();
            default: throw new BadShapeCreation(type);
        }
    }
    public static void main(String[] args) {
        FactoryTest.test(new ShapeFactory1());
    }
}
```

輸出結果：

```java
Circle[0] draw
Circle[0] erase
Square[1] draw
Square[1] erase
Triangle[2] draw
Triangle[2] erase
Square[3] draw
Square[3] erase
Circle[4] draw
Circle[4] erase
Circle[5] draw
Circle[5] erase
Triangle[6] draw
Triangle[6] erase 
```

`create()`現在是添加新類型的Shape時系統中唯一需要更改的其他程式碼。

### 動態工廠

前面例子中的**靜態**`create()`方法強制所有建立操作都集中在一個位置，因此這是添加新類型的**Shape**時唯一必須更改程式碼的地方。這當然是一個合理的解決方案，因為它把建立物件的過程限制在一個框內。但是，如果你在添加新類時無需修改任何內容，那就太好了。 以下版本使用反射在首次需要時將**Shape**的構造器動態載入到工廠列表中：

```java
// patterns/ShapeFactory2.java
import java.util.*;
import java.lang.reflect.*;
import java.util.stream.*;
import patterns.shapes.*;
public class ShapeFactory2 implements FactoryMethod {
    Map<String, Constructor> factories = new HashMap<>();
    static Constructor load(String id) {
        System.out.println("loading " + id);
        try {
            return Class.forName("patterns.shapes." + id)
                .getConstructor();
        } catch(ClassNotFoundException |
                NoSuchMethodException e) {
            throw new BadShapeCreation(id);
        }
    }
    public Shape create(String id) {
        try {
            return (Shape)factories
                .computeIfAbsent(id, ShapeFactory2::load)
                .newInstance();
        } catch(InstantiationException |
                IllegalAccessException |
                InvocationTargetException e) {
            throw new BadShapeCreation(id);
        }
    }
    public static void main(String[] args) {
        FactoryTest.test(new ShapeFactory2());
    }
}
```

輸出結果：

```java
loading Circle
Circle[0] draw
Circle[0] erase
loading Square
Square[1] draw
Square[1] erase
loading Triangle
Triangle[2] draw
Triangle[2] erase
Square[3] draw
Square[3] erase
Circle[4] draw
Circle[4] erase
Circle[5] draw
Circle[5] erase
Triangle[6] draw
Triangle[6] erase
```

和之前一樣，`create()`方法基於你傳遞給它的**String**參數生成新的**Shape**s，但是在這裡，它是透過在**HashMap**中尋找作為鍵的**String**來實現的。 返回的值是一個構造器，該構造器用於透過呼叫`newInstance()`建立新的**Shape**物件。

然而，當你開始執行程式時，工廠的`map`為空。`create()`使用`map`的`computeIfAbsent()`方法來尋找構造器（如果該構造器已存在於`map`中）。如果不存在則使用`load()`計算出該構造器，並將其插入到`map`中。 從輸出中可以看到，每種特定類型的**Shape**都是在第一次請求時才載入的，然後只需要從`map`中檢索它。

### 多態工廠

《設計模式》這本書強調指出，採用“工廠方法”模式的原因是可以從基本工廠中繼承出不同類型的工廠。 再次修改範例，使工廠方法位於單獨的類中：

```java
// patterns/ShapeFactory3.java
// Polymorphic factory methods
import java.util.*;
import java.util.function.*;
import java.util.stream.*;
import patterns.shapes.*;
interface PolymorphicFactory {
    Shape create();
}
class RandomShapes implements Supplier<Shape> {
    private final PolymorphicFactory[] factories;
    private Random rand = new Random(42);
    RandomShapes(PolymorphicFactory... factories){
        this.factories = factories;
    }
    public Shape get() {
        return factories[ rand.nextInt(factories.length)].create();
    }
}
public class ShapeFactory3 {
    public static void main(String[] args) {
        RandomShapes rs = new RandomShapes(
            Circle::new,
            Square::new,
            Triangle::new);
        Stream.generate(rs)
            .limit(6)
            .peek(Shape::draw)
            .peek(Shape::erase)
            .count();
    }
}
```

輸出結果：

```java
Triangle[0] draw
Triangle[0] erase
Circle[1] draw
Circle[1] erase
Circle[2] draw
Circle[2] erase
Triangle[3] draw
Triangle[3] erase
Circle[4] draw
Circle[4] erase
Square[5] draw
Square[5] erase 
```

**RandomShapes**實現了**Supplier \<Shape>**，因此可用於透過`Stream.generate()`建立**Stream**。 它的構造器採用**PolymorphicFactory**物件的可變參數列表。 變數參數列表以陣列形式出現，因此列表是以陣列形式在內部儲存的。`get()`方法隨機獲取此陣列中一個物件的索引，並在結果上呼叫`create()`以產生新的**Shape**物件。 添加新類型的**Shape**時，**RandomShapes**構造器是唯一需要更改的地方。 請注意，此構造器需要**Supplier \<Shape>**。 我們傳遞給其**Shape**構造器的方法引用，該引用可滿足**Supplier \<Shape>**約定，因為Java 8支援結構一致性。

鑑於**ShapeFactory2.java**可能會拋出異常，使用此方法則沒有任何異常——它在編譯時完全確定。

### 抽象工廠

抽象工廠模式看起來像我們之前所見的工廠物件，但擁有不是一個工廠方法而是幾個工廠方法， 每個工廠方法都會建立不同種類的物件。 這個想法是在建立工廠物件時，你決定如何使用該工廠建立的所有物件。 《設計模式》中提供的範例實現了跨各種圖形使用者介面（GUI）的可移植性：你建立一個適合你正在使用的GUI的工廠物件，然後從中請求選單，按鈕，滑塊等等，它將自動為GUI建立適合該項目版本的元件。 因此，你可以將從一個GUI更改為另一個所產生的影響隔離限制在一處。 作為另一個範例，假設你正在建立一個通用遊戲環境來支援不同類型的遊戲。 使用抽象工廠看起來就像下文那樣：

```java
// patterns/abstractfactory/GameEnvironment.java
// An example of the Abstract Factory pattern
// {java patterns.abstractfactory.GameEnvironment}
package patterns.abstractfactory;
import java.util.function.*;
interface Obstacle {
    void action();
}

interface Player {
    void interactWith(Obstacle o);
}

class Kitty implements Player {
    @Override
    public void interactWith(Obstacle ob) {
        System.out.print("Kitty has encountered a ");
        ob.action();
    }
}

class KungFuGuy implements Player {
    @Override
    public void interactWith(Obstacle ob) {
        System.out.print("KungFuGuy now battles a ");
        ob.action();
    }
}

class Puzzle implements Obstacle {
    @Override
    public void action() {
        System.out.println("Puzzle");
    }
}

class NastyWeapon implements Obstacle {
    @Override
    public void action() {
        System.out.println("NastyWeapon");
    }
}

// The Abstract Factory:
class GameElementFactory {
    Supplier<Player> player;
    Supplier<Obstacle> obstacle;
}

// Concrete factories:
class KittiesAndPuzzles extends GameElementFactory {
    KittiesAndPuzzles() {
        player = Kitty::new;
        obstacle = Puzzle::new;
    }
}

class KillAndDismember extends GameElementFactory {
    KillAndDismember() {
        player = KungFuGuy::new;
        obstacle = NastyWeapon::new;
    }
}

public class GameEnvironment {
    private Player p;
    private Obstacle ob;

    public GameEnvironment(GameElementFactory factory) {
        p = factory.player.get();
        ob = factory.obstacle.get();
    }
    public void play() {
        p.interactWith(ob);
    }
    public static void main(String[] args) {
        GameElementFactory kp = new KittiesAndPuzzles(), kd = new KillAndDismember();
        GameEnvironment g1 = new GameEnvironment(kp), g2 = new GameEnvironment(kd);
        g1.play();
        g2.play();
    }
}

```

輸出結果：

```java
Kitty has encountered a Puzzle
KungFuGuy now battles a NastyWeapon
```

在這種環境中，**Player**物件與**Obstacle**物件進行互動，但是根據你所玩遊戲的類型，存在不同類型的玩家和障礙物。 你可以透過選擇特定的**GameElementFactory**來確定遊戲的類型，然後**GameEnvironment**控制遊戲的設定和玩法。 在此範例中，設定和玩法非常簡單，但是這些活動（初始條件和狀態變化）可以決定遊戲的大部分結果。 這裡，**GameEnvironment**不是為繼承而設計的，儘管這樣做很有意義。 它還包含“雙重調度”和“工廠方法”的範例，稍後將對這兩個範例進行說明。

<!-- Function Objects -->

## 函數物件

一個 *函數物件* 封裝了一個函數。其特點就是將被呼叫函數的選擇與那個函數被呼叫的位置進行解耦。

*《設計模式》* 中也提到了這個術語，但是沒有使用。然而，*函數物件* 的話題卻在那本書的很多模式中被反覆論及。

### 指令模式

從最直觀的角度來看，*指令模式* 就是一個函數物件：一個作為物件的函數。我們可以將 *函數物件* 作為參數傳遞給其他方法或者物件，來執行特定的操作。

在Java 8之前，想要產生單個函數的效果，我們必須明確將方法包含在物件中，而這需要太多的儀式了。而利用Java 8的lambda特性， *指令模式* 的實現將是微不足道的。

```java
// patterns/CommandPattern.java
import java.util.*;

public class CommandPattern {
  public static void main(String[] args) {
    List<Runnable> macro = Arrays.asList(
      () -> System.out.print("Hello "),
      () -> System.out.print("World! "),
      () -> System.out.print("I'm the command pattern!")
    );
    macro.forEach(Runnable::run);
  }
}
/* Output:
Hello World! I'm the command pattern!
*/
```

*指令模式* 的主要特點是允許向一個方法或者物件傳遞一個想要的動作。在上面的例子中，這個物件就是 **macro** ，而 *指令模式* 提供了將一系列需要一起執行的動作集進行排隊的方法。在這裡，*指令模式* 允許我們動態的建立新的行為，通常情況下我們需要編寫新的程式碼才能完成這個功能，而在上面的例子中，我們可以透過解釋執行一個腳本來完成這個功能（如果需要實現的東西很複雜請參考解釋器模式）。

*《設計模式》* 認為“指令模式是回調的物件導向的替代品”。儘管如此，我認為"back"（回來）這個詞是callback（回調）這一概念的基本要素。也就是說，我認為回調（callback）實際上是返回到回調的建立者所在的位置。另一方面，對於 *指令* 物件，通常只需建立它並將其交給某種方法或物件，而不是自始至終以其他方式聯繫指令物件。不管怎樣，這就是我對它的看法。在本章的後面內容中，我將會把一組設計模式放在“回調”的標題下面。

### 策略模式

*策略模式* 看起來像是從同一個基類繼承而來的一系列 *指令* 類。但是仔細查看 *指令模式*，你就會發現它也具有同樣的結構：一系列分層次的 *函數物件*。不同之處在於，這些函數物件的用法和策略模式不同。就像前面的 `io/DirList.java` 那個例子，使用 *指令* 是為了解決特定問題 -- 從一個列表中選擇文件。“不變的部分”是被呼叫的那個方法，而變化的部分被分離出來放到 *函數物件* 中。我認為 *指令模式* 在編碼階段提供了靈活性，而 *策略模式* 的靈活性在執行時才會體現出來。儘管如此，這種區別卻是非常模糊的。

另外，*策略模式* 還可以添加一個“上下文（context）”，這個上下文（context）可以是一個代理類（surrogate class），用來控制對某個特定 *策略* 物件的選擇和使用。就像 *橋接模式* 一樣！下面我們來一探究竟：

```java
// patterns/strategy/StrategyPattern.java
// {java patterns.strategy.StrategyPattern}
package patterns.strategy;
import java.util.function.*;
import java.util.*;

// The common strategy base type:
class FindMinima {
  Function<List<Double>, List<Double>> algorithm;
}

// The various strategies:
class LeastSquares extends FindMinima {
  LeastSquares() {
    // Line is a sequence of points (Dummy data):
    algorithm = (line) -> Arrays.asList(1.1, 2.2);
  }
}

class Perturbation extends FindMinima {
  Perturbation() {
    algorithm = (line) -> Arrays.asList(3.3, 4.4);
  }
}

class Bisection extends FindMinima {
  Bisection() {
    algorithm = (line) -> Arrays.asList(5.5, 6.6);
  }
}

// The "Context" controls the strategy:
class MinimaSolver {
  private FindMinima strategy;
  MinimaSolver(FindMinima strat) {
    strategy = strat;
  }
  List<Double> minima(List<Double> line) {
    return strategy.algorithm.apply(line);
  }
  void changeAlgorithm(FindMinima newAlgorithm) {
    strategy = newAlgorithm;
  }
}

public class StrategyPattern {
  public static void main(String[] args) {
    MinimaSolver solver = 
      new MinimaSolver(new LeastSquares());
    List<Double> line = Arrays.asList(
      1.0, 2.0, 1.0, 2.0, -1.0,
      3.0, 4.0, 5.0, 4.0 );
    System.out.println(solver.minima(line)); 
    solver.changeAlgorithm(new Bisection()); 
    System.out.println(solver.minima(line));
  }
}
/* Output:
[1.1, 2.2]
[5.5, 6.6]
*/
```

`MinimaSolver` 中的 `changeAlgorithm()` 方法將一個不同的策略插入到了 `私有` 域 `strategy` 中，這使得在呼叫 `minima()` 方法時，可以使用新的策略。

我們可以透過將上下文注入到 `FindMinima` 中來簡化我們的解決方法。

```java
// patterns/strategy/StrategyPattern2.java // {java patterns.strategy.StrategyPattern2}
package patterns.strategy;
import java.util.function.*;
import java.util.*;

// "Context" is now incorporated:
class FindMinima2 {
  Function<List<Double>, List<Double>> algorithm;
  FindMinima2() { leastSquares(); } // default
  // The various strategies:
  void leastSquares() {
    algorithm = (line) -> Arrays.asList(1.1, 2.2);
  }
  void perturbation() {
    algorithm = (line) -> Arrays.asList(3.3, 4.4);
  }
  void bisection() {
    algorithm = (line) -> Arrays.asList(5.5, 6.6);
  }
  List<Double> minima(List<Double> line) {
    return algorithm.apply(line);
  }
}

public class StrategyPattern2 {
  public static void main(String[] args) {
    FindMinima2 solver = new FindMinima2();
    List<Double> line = Arrays.asList(
      1.0, 2.0, 1.0, 2.0, -1.0,
      3.0, 4.0, 5.0, 4.0 );
    System.out.println(solver.minima(line));
    solver.bisection();
    System.out.println(solver.minima(line));
  }
}
/* Output:
[1.1, 2.2]
[5.5, 6.6]
*/
```

`FindMinima2` 封裝了不同的演算法，也包含了“上下文”（Context），所以它便可以在一個單獨的類中控制演算法的選擇了。

### 責任鏈模式

*責任鏈模式* 也許可以被看作一個使用了 *策略* 物件的“遞迴的動態一般化”。此時我們進行一次呼叫，在一個鏈序列中的每個策略都試圖滿足這個呼叫。這個過程直到有一個策略成功滿足該呼叫或者到達鏈序列的末尾才結束。在遞迴方法中，一個方法將反覆呼叫它自身直至達到某個終止條件；使用責任鏈，一個方法會呼叫相同的基類方法（擁有不同的實現），這個基類方法將會呼叫基類方法的其他實現，如此反覆直至達到某個終止條件。

除了呼叫某個方法來滿足某個請求以外，鏈中的多個方法都有機會滿足這個請求，因此它有點專家系統的意味。由於責任鏈實際上就是一個鍊表，它能夠動態建立，因此它可以看作是一個更一般的動態構建的 `switch` 語句。

在上面的 `StrategyPattern.java` 例子中，我們可能想自動發現一個解決方法。而 *責任鏈* 就可以達到這個目的：

```java
// patterns/chain/ChainOfResponsibility.java
// Using the Functional interface
// {java patterns.chain.ChainOfResponsibility}
package patterns.chain;
import java.util.*;
import java.util.function.*;

class Result {
  boolean success;
  List<Double> line;
  Result(List<Double> data) {
    success = true;
    line = data;
  }
  Result() {
    success = false;
    line = Collections.<Double>emptyList();
  }
}

class Fail extends Result {}

interface Algorithm {
  Result algorithm(List<Double> line);
}

class FindMinima {
  public static Result leastSquares(List<Double> line) {
    System.out.println("LeastSquares.algorithm");
    boolean weSucceed = false;
    if(weSucceed) // Actual test/calculation here
      return new Result(Arrays.asList(1.1, 2.2));
    else // Try the next one in the chain:
      return new Fail();
  }
  public static Result perturbation(List<Double> line) {
    System.out.println("Perturbation.algorithm");
    boolean weSucceed = false;
    if(weSucceed) // Actual test/calculation here
      return new Result(Arrays.asList(3.3, 4.4));
    else
      return new Fail();
  }
  public static Result bisection(List<Double> line) {
    System.out.println("Bisection.algorithm");
    boolean weSucceed = true;
    if(weSucceed) // Actual test/calculation here
      return new Result(Arrays.asList(5.5, 6.6));
    else
      return new Fail();
    }
  static List<Function<List<Double>, Result>>
    algorithms = Arrays.asList(
      FindMinima::leastSquares,
      FindMinima::perturbation,
      FindMinima::bisection
    );
  public static Result minima(List<Double> line) {
    for(Function<List<Double>, Result> alg :
        algorithms) {
      Result result = alg.apply(line);
      if(result.success)
        return result;
    }
    return new Fail();
  }
}

public class ChainOfResponsibility {
  public static void main(String[] args) {
    FindMinima solver = new FindMinima();
    List<Double> line = Arrays.asList(
      1.0, 2.0, 1.0, 2.0, -1.0,
      3.0, 4.0, 5.0, 4.0);
    Result result = solver.minima(line);
    if(result.success)
      System.out.println(result.line);
    else
      System.out.println("No algorithm found");
  }
}
/* Output:
LeastSquares.algorithm
Perturbation.algorithm
Bisection.algorithm
[5.5, 6.6]
*/
```

我們從定義一個 `Result` 類開始，這個類包含一個 `success` 標誌，因此接收者就可以知道演算法是否成功執行，而 `line` 變數儲存了真實的資料。當演算法執行失敗時， `Fail` 類可以作為返回值。要注意的是，當演算法執行失敗時，返回一個 `Result` 物件要比拋出一個異常更加合適，因為我們有時可能並不打算解決這個問題，而是希望程式繼續執行下去。

每一個 `Algorithm` 介面的實現，都實現了不同的 `algorithm()` 方法。在 `FindMinama` 中，將會建立一個演算法的列表（這就是所謂的“鏈”），而 `minima()` 方法只是遍歷這個列表，然後找到能夠成功執行的演算法而已。

<!-- Changing the Interface -->
## 改變介面

有時候我們需要解決的問題很簡單，僅僅是“我沒有需要的介面”而已。有兩種設計模式用來解決這個問題：*適配器模式* 接受一種類型並且提供一個對其他類型的介面。*外觀模式* 為一組類建立了一個介面，這樣做只是為了提供一種更方便的方法來處理庫或資源。

### 適配器模式（Adapter）

當我們手頭有某個類，而我們需要的卻是另外一個類，我們就可以透過 *適配器模式* 來解決問題。唯一需要做的就是產生出我們需要的那個類，有許多種方法可以完成這種適配。

```java
// patterns/adapt/Adapter.java
// Variations on the Adapter pattern
// {java patterns.adapt.Adapter}
package patterns.adapt;

class WhatIHave {
  public void g() {}
  public void h() {}
}

interface WhatIWant {
  void f();
}

class ProxyAdapter implements WhatIWant {
  WhatIHave whatIHave;
  ProxyAdapter(WhatIHave wih) {
    whatIHave = wih;
  }
  @Override
  public void f() {
    // Implement behavior using
    // methods in WhatIHave:
    whatIHave.g();
    whatIHave.h();
  }
}

class WhatIUse {
  public void op(WhatIWant wiw) {
    wiw.f();
  }
}

// Approach 2: build adapter use into op():
class WhatIUse2 extends WhatIUse {
  public void op(WhatIHave wih) {
    new ProxyAdapter(wih).f();
  }
}

// Approach 3: build adapter into WhatIHave:
class WhatIHave2 extends WhatIHave implements WhatIWant {
  @Override
  public void f() {
    g();
    h();
  }
}

// Approach 4: use an inner class:
class WhatIHave3 extends WhatIHave {
  private class InnerAdapter implements WhatIWant {
    @Override
    public void f() {
      g();
      h();
    }
  }
  public WhatIWant whatIWant() {
    return new InnerAdapter();
  }
}

public class Adapter {
  public static void main(String[] args) {
    WhatIUse whatIUse = new WhatIUse();
    WhatIHave whatIHave = new WhatIHave();
    WhatIWant adapt= new ProxyAdapter(whatIHave);
    whatIUse.op(adapt);
    // Approach 2:
    WhatIUse2 whatIUse2 = new WhatIUse2();
    whatIUse2.op(whatIHave);
    // Approach 3:
    WhatIHave2 whatIHave2 = new WhatIHave2();
    whatIUse.op(whatIHave2);
    // Approach 4:
    WhatIHave3 whatIHave3 = new WhatIHave3();
    whatIUse.op(whatIHave3.whatIWant());
  }
}
```

我想冒昧的借用一下術語“proxy”（代理），因為在 *《設計模式》* 裡，他們堅持認為一個代理（proxy）必須擁有和它所代理的對像一模一樣的介面。但是，如果把這兩個詞一起使用，叫做“代理適配器（proxy adapter）”，似乎更合理一些。

### 外觀模式（Façade）

當我想方設法試圖將需求初步（first-cut）轉化成物件的時候，通常我使用的原則是：

>“把所有醜陋的東西都隱藏到物件裡去”。

基本上說，*外觀模式* 做的就是這件事情。如果我們有一堆讓人頭暈的類以及互動（Interactions），而它們又不是用戶端程式設計師必須了解的，那我們就可以為用戶端程式設計師建立一個介面只提供那些必要的功能。

外觀模式經常被實現為一個符合單例模式（Singleton）的抽象工廠（abstract factory）。當然，你可以透過建立包含 **靜態** 工廠方法（static factory methods）的類來達到上述效果。

```java
// patterns/Facade.java

class A { A(int x) {} }

class B { B(long x) {} }

class C { C(double x) {} }

// Other classes that aren't exposed by the
// facade go here ...
public class Facade {
  static A makeA(int x) { return new A(x); }
  static B makeB(long x) { return new B(x); }
  static C makeC(double x) { return new C(x); }
  public static void main(String[] args) {
    // The client programmer gets the objects
    // by calling the static methods:
    A a = Facade.makeA(1);
    B b = Facade.makeB(1);
    C c = Facade.makeC(1.0);
  }
}
```

《設計模式》給出的例子並不是真正的 *外觀模式* ，而僅僅是一個類使用了其他的類而已。

#### 包（Package）作為外觀模式的變體

我感覺，*外觀模式* 更傾向於“過程式的（procedural）”，也就是非物件導向的（non-object-oriented）：我們是透過呼叫某些函數才得到物件。它和抽象工廠（Abstract factory）到底有多大差別呢？*外觀模式* 關鍵的一點是隱藏某個庫的一部分類（以及它們的互動），使它們對於用戶端程式設計師不可見，這樣那些類的介面就更加簡練和易於理解了。

其實，這也正是 Java 的 packaging（包）的功能所完成的事情：在庫以外，我們只能建立和使用被聲明為公共（public）的那些類；所有非公共（non-public）的類只能被同一 package 的類使用。看起來，*外觀模式* 似乎是 Java 內嵌的一個功能。

公平起見，*《設計模式》* 主要是寫給 C++ 讀者的。儘管 C++ 有命名空間（namespaces）機制來防止全域變數和類名稱之間的衝突，但它並沒有提供類隱藏的機制，而在 Java 裡我們可以透過聲明 non-public 類來實現這一點。我認為，大多數情況下 Java 的 package 功能就足以解決針對 *外觀模式* 的問題了。

<!-- Interpreter: Run-Time Flexibility -->
## 解釋器：執行時的彈性

如果程式的使用者需要更好的執行時彈性，例如建立腳本來增加需要的系統功能，你就能使用解釋器設計模式。這個模式下，你可以建立一個語言解釋器並將它嵌入你的程式內。

在開發程式的過程中，設計自己的語言並為它構建一個解釋器是一件讓人分心且耗時的事。最好的解決方案就是復用程式碼：使用一個已經構建好並被除錯過的解釋器。Python 語言可以免費地嵌入營利性的應用中而不需要任何的協議許可、授權費或者是任何的聲明。此外，有一個完全使用 Java 位元組碼實現的 Python 版本（叫做 Jython）， 能夠輕易地合併到 Java 程式中。Python 是一門非常易學習的腳本語言，程式碼的讀寫很有邏輯性。它支援函數與物件，有大量的可用庫，並且可執行在所有的平台上。你可以在 [www.Python.org](https://www.python.org/) 上下載 Python 並了解更多訊息。

<!-- Callbacks -->
## 回調


<!-- Multiple Dispatching -->
## 多次調度


<!-- Pattern Refactoring -->
## 模式重構


<!-- Abstracting Usage -->
## 抽象用法


<!-- Multiple Dispatching -->
## 多次派遣


<!-- The Visitor Pattern -->
## 訪問者模式


<!-- RTTI Considered Harmful? -->
## RTTI的優劣


<!-- Summary -->
## 本章小結





<!-- 分頁 -->

<div style="page-break-after: always;"></div>
