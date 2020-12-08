[TOC]



<!-- Appendix: Understanding equals() and hashCode() -->
# 附錄:理解equals和hashCode方法
假設有一個容器使用hash函數，當你建立一個放到這個容器時，你必須定義 **hashCode()** 函數和 **equals()** 函數。這兩個函數一起被用於hash容器中的查詢操作。


<!-- A Canonical equals() -->
## equals規範
當你建立一個類的時候，它自動繼承自 **Objcet** 類。如果你不覆寫 **equals()** ，你將會獲得 **Objcet** 物件的 **equals()** 函數。預設情況下，這個函數會比較物件的地址。所以只有你在比較同一個物件的時候，你才會獲得**true**。預設的情況是"區分度最高的"。

```java
// equalshashcode/DefaultComparison.java
class DefaultComparison {
    private int i, j, k;
    DefaultComparison(int i, int j, int k) {
        this.i = i;
        this.j = j;
        this.k = k;
    }
    
    public static void main(String[] args) {
        DefaultComparison 
        a = new DefaultComparison(1, 2, 3),
        b = new DefaultComparison(1, 2, 3);
        System.out.println(a == a);
        System.out.println(a == b);
    } 
} 
/*
Output:
true
false
*/

```
通常你會希望放寬這個限制。一般來說如果兩個物件有相同的類型和相同的欄位，你會認為這兩個物件相等，但也會有一些你不想加入 **equals()** 函數中來比較的欄位。這是類型設計的一部分。

一個合適的 **equals()**函數必須滿足以下五點條件：
1. 反身性：對於任何 **x**， **x.equals(x)** 應該返回 **true**。
2. 對稱性：對於任何 **x** 和 **y**， **x.equals(y)** 應該返回 **true**當且僅當 **y.equals(x)** 返回 **true** 。
3. 傳遞性：對於任何**x**,**y**,還有**z**，如果 **x.equals(y)** 返回 **true** 並且 **y.equals(z)** 返回 **true**，那麼  **x.equals(z)** 應該返回 **true**。
4. 一致性：對於任何 **x**和**y**，在物件沒有被改變的情況下，多次呼叫 **x.equals(y)** 應該總是返回 **true** 或者**false**。
5. 對於任何非**null**的**x**，**x.equals(null)**應該返回**false**。

下面是滿足這些條件的測試，並且判斷物件是否和自己相等（我們這裡稱呼其為**右值**）：
1. 如果**右值**是**null**，那麼不相等。
2. 如果**右值**是**this**，那麼兩個物件相等。
3. 如果**右值**不是同一個類型或者子類，那麼兩個物件不相等。
4. 如果所有上面的檢查通過了，那麼你必須決定 **右值** 中的哪些欄位是重要的，然後比較這些欄位。
Java 7 引入了 **Objects** 類型來幫助這個流程，這樣我們能夠寫出更好的 **equals()** 函數。

下面的例子比較了不同類型的 **Equality**類。為了避免重複的程式碼，我們使用*工廠函數設計模*式來實現樣例。 **EqualityFactory**介面提供**make()**函數來生成一個**Equaity**物件，這樣不同的**EqualityFactory**能夠生成**Equality**不同的子類。

```java
// equalshashcode/EqualityFactory.java
import java.util.*;
interface EqualityFactory {
    Equality make(int i, String s, double d);
}
```
現在我們來定義 **Equality**，它包含三個欄位（所有的欄位我們認為在比較中都很重要）和一個 **equals()** 函數用來滿足上述的四種檢查。建構子展示了它的類名來保證我們在執行我們想要的測試：

```java
// equalshashcode/Equality.java
import java.util.*;
public class Equality {
    protected int i;
    protected String s;
    protected double d;
    public Equality(int i, String s, double d) {
        this.i = i;
        this.s = s;
        this.d = d;
        System.out.println("made 'Equality'");
    } 
    
    @Override
    public boolean equals(Object rval) {
        if(rval == null)
            return false;
        if(rval == this)
            return true;
        if(!(rval instanceof Equality))
            return false;
        Equality other = (Equality)rval;
        if(!Objects.equals(i, other.i))
            return false;
        if(!Objects.equals(s, other.s))
            return false;
        if(!Objects.equals(d, other.d))return false;
            return true;
    } 
    
    public void test(String descr, String expected, Object rval) {
        System.out.format("-- Testing %s --%n" + "%s instanceof Equality: %s%n" +
        "Expected %s, got %s%n",
        descr, descr, rval instanceof Equality,
        expected, equals(rval));
    } 
    
    public static void testAll(EqualityFactory eqf) {
        Equality
        e = eqf.make(1, "Monty", 3.14),
        eq = eqf.make(1, "Monty", 3.14),
        neq = eqf.make(99, "Bob", 1.618);
        e.test("null", "false", null);
        e.test("same object", "true", e);
        e.test("different type",
        "false", Integer.valueOf(99));e.test("same values", "true", eq);
        e.test("different values", "false", neq);
    } 
    
    public static void main(String[] args) {
        testAll( (i, s, d) -> new Equality(i, s, d));
    } 
    
} 
/*
Output:
made 'Equality'
made 'Equality'
made 'Equality'
-- Testing null --
null instanceof Equality: false
Expected false, got false
-- Testing same object --
same object instanceof Equality: true
Expected true, got true
-- Testing different type --
different type instanceof Equality: false
Expected false, got false-- Testing same values --
same values instanceof Equality: true
Expected true, got true
-- Testing different values --
different values instanceof Equality: true
Expected false, got false
*/
```

 **testAll()** 執行了我們期望的所有不同類型物件的比較。它使用工廠建立了**Equality**物件。

在 **main()** 裡，請注意對 **testAll()** 的呼叫很簡單。因為**EqualityFactory**有著單一的函數，它能夠和lambda表達式一起使用來表示 **make()** 函數。

上述的 **equals()** 函數非常繁瑣，並且我們能夠將其簡化成規範的形式，請注意：
1. **instanceof**檢查減少了**null**檢查的需要。
2. 和**this**的比較是多餘的。一個正確書寫的 **equals()** 函數能正確地和自己比較。


因為 **&&** 是一個短路比較，它會在第一次遇到失敗的時候退出並返回**false**。所以，透過使用 **&&** 將檢查連結起來，我們可以寫出更精簡的 **equals()** 函數：

```java
// equalshashcode/SuccinctEquality.java
import java.util.*;
public class SuccinctEquality extends Equality {
    public SuccinctEquality(int i, String s, double d) {
        super(i, s, d);
        System.out.println("made 'SuccinctEquality'");
    } 
    
    @Override
    public boolean equals(Object rval) {
        return rval instanceof SuccinctEquality &&
        Objects.equals(i, ((SuccinctEquality)rval).i) &&
        Objects.equals(s, ((SuccinctEquality)rval).s) &&
        Objects.equals(d, ((SuccinctEquality)rval).d);
    } 
    public static void main(String[] args) {
        Equality.testAll( (i, s, d) ->
        new SuccinctEquality(i, s, d));
    } 
    
}
/* Output:
made 'Equality'
made 'SuccinctEquality'
made 'Equality'
made 'SuccinctEquality'
made 'Equality'
made 'SuccinctEquality'
-- Testing null --
null instanceof Equality: false
Expected false, got false
-- Testing same object --
same object instanceof Equality: true
Expected true, got true
-- Testing different type --
different type instanceof Equality: false
Expected false, got false
-- Testing same values --
same values instanceof Equality: true
Expected true, got true
-- Testing different values --different values instanceof Equality: true
Expected false, got false
*/
```
對於每個 **SuccinctEquality**，基類建構子在衍生類建構子前被呼叫，輸出顯示我們依然獲得了正確的結果，你可以發現短路返回已經發生了，不然的話，**null**測試和“不同類型”的測試會在 **equals()** 函數下面的比較中強制轉化的時候拋出異常。
 **Objects.equals()** 會在你組合其他類型的時候發揮很大的作用。

```java
// equalshashcode/ComposedEquality.java
import java.util.*;
class Part {
    String ss;
    double dd;
    
    Part(String ss, double dd) {
        this.ss = ss;
        this.dd = dd;
    }
    
    @Override
    public boolean equals(Object rval) {
        return rval instanceof Part &&
        Objects.equals(ss, ((Part)rval).ss) &&
        Objects.equals(dd, ((Part)rval).dd);
    } 
    
} 
    
public class ComposedEquality extends SuccinctEquality {
    Part part;
    public ComposedEquality(int i, String s, double d) {
        super(i, s, d);
        part = new Part(s, d);
        System.out.println("made 'ComposedEquality'");
    }
    @Override
    public boolean equals(Object rval) {
        return rval instanceof ComposedEquality &&
        super.equals(rval) &&
        Objects.equals(part,
        ((ComposedEquality)rval).part);
        
    } 
        
    public static void main(String[] args) {
        Equality.testAll( (i, s, d) ->
        new ComposedEquality(i, s, d));
    }
}
/*
Output:
made 'Equality'
made 'SuccinctEquality'
made 'ComposedEquality'
made 'Equality'
made 'SuccinctEquality'
made 'ComposedEquality'
made 'Equality'
made 'SuccinctEquality'
made 'ComposedEquality'
-- Testing null --null instanceof Equality: false
Expected false, got false
-- Testing same object --
same object instanceof Equality: true
Expected true, got true
-- Testing different type --
different type instanceof Equality: false
Expected false, got false
-- Testing same values --
same values instanceof Equality: true
Expected true, got true
-- Testing different values --
different values instanceof Equality: true
Expected false, got false
*/
```
注意super.equals()這個呼叫，沒有必要重新發明它（因為你不總是有權限訪問基類所有的必要欄位）

<!--Equality Across Subtypes -->
### 不同子類的相等性
繼承意味著兩個不同子類的物件當其向上轉型的時候可以是相等的。假設你有一個Animal物件的集合。這個集合天然接受**Animal**的子類。在這個例子中是**Dog**和**Pig**。每個**Animal**有一個**name**和**size**，還有唯一的內部**id**數字。

我們透過**Objects**類，以規範的形式定義 **equals()**函數和**hashCode()**。但是我們只能在基類**Animal**中定義他們。並且我們在這兩個函數中沒有包含**id**欄位。從**equals()**函數的角度看待，這意味著我們只關心它是否是**Animal**，而不關心是否是**Animal**的某個子類。

```java
// equalshashcode/SubtypeEquality.java
import java.util.*;
enum Size { SMALL, MEDIUM, LARGE }
class Animal {
    private static int counter = 0;
    private final int id = counter++;
    private final String name;
    private final Size size;
    Animal(String name, Size size) {
        this.name = name;
        this.size = size;
    } 
    @Override
    public boolean equals(Object rval) {
        return rval instanceof Animal &&
        // Objects.equals(id, ((Animal)rval).id) && // [1]
        Objects.equals(name, ((Animal)rval).name) &&
        Objects.equals(size, ((Animal)rval).size);
    } 
    
    @Override
    public int hashCode() {
        return Objects.hash(name, size);
        // return Objects.hash(name, size, id); // [2]
    } 
    
    @Override
    public String toString() {
        return String.format("%s[%d]: %s %s %x",
        getClass().getSimpleName(), id,
        name, size, hashCode());
    } 
} 
    
class Dog extends Animal {
    Dog(String name, Size size) {
    super(name, size);
    } 
} 

class Pig extends Animal {
    Pig(String name, Size size) {
    super(name, size);
    } 
} 
    
public class SubtypeEquality {
    public static void main(String[] args) {
        Set<Animal> pets = new HashSet<>();
        pets.add(new Dog("Ralph", Size.MEDIUM));
        pets.add(new Pig("Ralph", Size.MEDIUM));
        pets.forEach(System.out::println);
    } 
} 
/*
Output:
Dog[0]: Ralph MEDIUM a752aeee
*/
```
如果我們只考慮類型的話，某些情況下它的確說得通——只從基類的角度看待問題，這是李氏取代原則的基石。這個程式碼完美符合取代理論因為衍生類沒有添加任何額外不再基類中的額外函數。衍生類只是在表現上不同，而不是在介面上。（當然這不是常態）

但是當我們提供了兩個有著相同資料的不同的物件類型，然後將他們放置在 **HashSet<Animal>** 中。只有他們中的一個能存活。這強調了 **equals()** 不是完美的數學理論，而只是機械般的理論。
 **hashCode()** 和 **equals()** 必須能夠允許類型在hash資料結構中正常工作。例子中 **Dog** 和 **Pig** 會被映射到同 **HashSet** 的同一個桶中。這個時候，**HashSet** 回退到 **equals()** 來區分物件，但是 **equals()** 也認為兩個物件是相同的。**HashSet**因為已經有一個相同的物件了，所以沒有添加 **Pig**。
我們依然能夠透過使得其他欄位物件不同來讓例子能夠正常工作。在這裡每個 **Animal** 已經有了一個獨一無二的 **id** ，所以你能夠取消  **equals()** 函數中的 **[1]** 行注釋，或者取消 **hashCode()** 函數中的 **[2]** 行注釋。按照規範，你應該同時完成這兩個操作，如此能夠將所有“不變的”欄位包含在兩個操作中（“不變”所以 **equals()** 和 **hashCode()** 在雜湊資料結構中的排序和取值時，不會生成不同的值。我將“不變的”放在引號中因為你必須計算出是否已經發生變化）。

> **旁註**： 在**hashCode()**中，如果你只能夠使用一個欄位，使用**Objcets.hashCode()**。如果你使用多個欄位，那麼使用 **Objects.hash()**。

我們也可以透過標準方式，將 **equals()** 定義在子類中（不包含 **id** ）解決這個問題：

```java
// equalshashcode/SubtypeEquality2.java
import java.util.*;
class Dog2 extends Animal {
    Dog2(String name, Size size) {
        super(name, size);
    } 
    
    @Override
    public boolean equals(Object rval) {
        return rval instanceof Dog2 &&super.equals(rval);
    } 
} 

class Pig2 extends Animal {
    Pig2(String name, Size size) {
    super(name, size);
    } 
    
    @Override
    public boolean equals(Object rval) {
        return rval instanceof Pig2 &&
        super.equals(rval);
    } 
}

public class SubtypeEquality2 {
    public static void main(String[] args) {
        Set<Animal> pets = new HashSet<>();
        pets.add(new Dog2("Ralph", Size.MEDIUM));
        pets.add(new Pig2("Ralph", Size.MEDIUM));
        pets.forEach(System.out::println);
    }
} 
/*
Output:
Dog2[0]: Ralph MEDIUM a752aeee
Pig2[1]: Ralph MEDIUM a752aeee
*/
```
注意 **hashCode()** 是獨一無二的，但是因為物件不再 **equals()** ，所以兩個函數都出現在**HashSet**中。另外，**super.equals()** 意味著我們不需要訪問基類的**private**欄位。


一種說法是Java從**equals()** 和**hashCode()** 的定義中分離了可替代性。我們仍然能夠將**Dog**和**Pig**放置在 **Set\<Animal\>** 中，無論 **equals()** 和 **hashCode()** 是如何定義的，但是物件不會在雜湊資料結構中正常工作，除非這些函數能夠被合理定義。不幸的是，**equals()** 不總是和 **hashCode()** 一起使用，這在你嘗試為了某個特殊類型避免定義它的時候會讓問題複雜化。並且這也是為什麼遵循規範是有價值的。然而這會變得更加複雜，因為你不總是需要定義其中一個函數。



<!-- Hashing and Hash Codes -->

## 雜湊和雜湊碼

在 [集合]() 章節中，我們使用預先定義的類作為 HashMap 的鍵。這些範例之所以有用，是因為預定義的類具有所有必需的連線，以使它們正確地充當鍵。

當建立自己的類作為HashMap的鍵時，會發生一個常見的陷阱，從而忘記進行必要的接線。例如，考慮一個將Earthhog 物件與 Prediction 物件匹配的天氣預報系統。這似乎很簡單：使用Groundhog作為鍵，使用Prediction作為值：

```java
// equalshashcode/Groundhog.java
// Looks plausible, but doesn't work as a HashMap key
public class Groundhog {
    protected int number;
    public Groundhog(int n) { number = n; }
    @Override
    public String toString() {
        return "Groundhog #" + number;
    }
}
```

```java
// equalshashcode/Prediction.java
// Predicting the weather
import java.util.*;
public class Prediction {
    private static Random rand = new Random(47);
    @Override
    public String toString() {
        return rand.nextBoolean() ?
                "Six more weeks of Winter!" : "Early Spring!";
    }
}
```

```java
// equalshashcode/SpringDetector.java
// What will the weather be?
import java.util.*;
import java.util.stream.*;
import java.util.function.*;
import java.lang.reflect.*;
public class SpringDetector {
    public static <T extends Groundhog>
    void detectSpring(Class<T> type) {
        try {
            Constructor<T> ghog =
                    type.getConstructor(int.class);
            Map<Groundhog, Prediction> map =
                    IntStream.range(0, 10)
                            .mapToObj(i -> {
                                try {
                                    return ghog.newInstance(i);
                                } catch(Exception e) {
                                    throw new RuntimeException(e);
                                }
                            })
                            .collect(Collectors.toMap(
                                    Function.identity(),
                                    gh -> new Prediction()));
            map.forEach((k, v) ->
                    System.out.println(k + ": " + v));
            Groundhog gh = ghog.newInstance(3);
            System.out.println(
                    "Looking up prediction for " + gh);
            if(map.containsKey(gh))
                System.out.println(map.get(gh));
            else
                System.out.println("Key not found: " + gh);
        } catch(NoSuchMethodException |
                IllegalAccessException |
                InvocationTargetException |
                InstantiationException e) {
            throw new RuntimeException(e);
        }
    }
    public static void main(String[] args) {
        detectSpring(Groundhog.class);
    }
}
/* Output:
Groundhog #3: Six more weeks of Winter!
Groundhog #0: Early Spring!
Groundhog #8: Six more weeks of Winter!
Groundhog #6: Early Spring!
Groundhog #4: Early Spring!
Groundhog #2: Six more weeks of Winter!
Groundhog #1: Early Spring!
Groundhog #9: Early Spring!
Groundhog #5: Six more weeks of Winter!
Groundhog #7: Six more weeks of Winter!
Looking up prediction for Groundhog #3
Key not found: Groundhog #3
*/
```

每個 Groundhog 都被賦予了一個常數，因此你可以透過如下的方式在 HashMap 中尋找對應的 Prediction。“給我一個和 Groundhog#3 相關聯的 Prediction”。而 Prediction 透過一個隨機生成的 boolean 來選擇天氣。`detectSpring()` 方法透過反射來實例化 Groundhog 類，或者它的子類。稍後，當我們繼承一種新型的“Groundhog ”以解決此處示範的問題時，這將派上用場。

這裡的 HashMap 被 Groundhog  和其相關聯的 Prediction 充滿。並且上面展示了 HashMap 裡面填充的內容。接下來我們使用填充了常數 3 的 Groundhog 作為 key 用於尋找對應的 Prediction 。（這個鍵值對肯定在 Map 中）。

這看起來十分簡單，但是這樣做並沒有奏效 —— 它無法找到數字3這個鍵。問題出在Groundhog自動地繼承自基類Object，所以這裡使用Object的hashCode0方法生成散列碼，而它預設是使用物件的地址計算散列碼。因此，由Groundhog(3)生成的第一個實例的散列碼與由Groundhog(3)生成的第二個實例的散列碼是不同的，而我們正是使用後者進行尋找的。

我們需要恰當的重寫hashCode()方法。但是它仍然無法正常執行，除非你同時重寫 equals()方法，它也是Object的一部分。HashMap使用equals()判斷目前的鍵是否與表中存在的鍵相同。

這是因為預設的Object.equals()只是比較物件的地址，所以一個Groundhog(3)並不等於另一個Groundhog(3)，因此，如果要使用自己的類作為HashMap的鍵，必須同時重載hashCode()和equals()，如下所示：

```java
// equalshashcode/Groundhog2.java
// A class that's used as a key in a HashMap
// must override hashCode() and equals()
import java.util.*;
public class Groundhog2 extends Groundhog {
    public Groundhog2(int n) { super(n); }
    @Override
    public int hashCode() { return number; }
    @Override
    public boolean equals(Object o) {
        return o instanceof Groundhog2 &&
                Objects.equals(
                        number, ((Groundhog2)o).number);
    }
}
```

```java
// equalshashcode/SpringDetector2.java
// A working key
public class SpringDetector2 {
    public static void main(String[] args) {
        SpringDetector.detectSpring(Groundhog2.class);
    }
}
/* Output:
Groundhog #0: Six more weeks of Winter!
Groundhog #1: Early Spring!
Groundhog #2: Six more weeks of Winter!
Groundhog #3: Early Spring!
Groundhog #4: Early Spring!
Groundhog #5: Six more weeks of Winter!
Groundhog #6: Early Spring!
Groundhog #7: Early Spring!
Groundhog #8: Six more weeks of Winter!
Groundhog #9: Six more weeks of Winter!
Looking up prediction for Groundhog #3
Early Spring!
*/
```

Groundhog2.hashCode0返回Groundhog的標識數位（編號）作為散列碼。在此例中，程式設計師負責確保不同的Groundhog具有不同的編號。hashCode()並不需要總是能夠返回唯一的標識碼（稍後你會理解其原因），但是equals() 方法必須嚴格地判斷兩個物件是否相同。此處的equals()是判斷Groundhog的號碼，所以作為HashMap中的鍵，如果兩個Groundhog2物件具有相同的Groundhog編號，程式就出錯了。

如何定義 equals() 方法在上一節 [equals 規範]()中提到了。輸出表明我們現在的輸出是正確的。

### 理解 hashCode

前面的例子只是正確解決問題的第一步。它只說明，如果不為你的鍵覆蓋hashCode() 和equals() ，那麼使用散列的資料結構（HashSet，HashMap，LinkedHashst或LinkedHashMap）就無法正確處理你的鍵。然而，要很好地解決此問題，你必須了解這些資料結構的內部構造。

首先，使用散列的目的在於：想要使用一個物件來尋找另一個物件。不過使用TreeMap或者你自己實現的Map也可以達到此目的。與散列實現相反，下面的範例用一對ArrayLists實現了一個Map，與AssociativeArray.java不同，這其中包含了Map介面的完整實現，因此提供了entrySet()方法：

```java
// equalshashcode/SlowMap.java
// A Map implemented with ArrayLists
import java.util.*;
import onjava.*;
public class SlowMap<K, V> extends AbstractMap<K, V> {
    private List<K> keys = new ArrayList<>();
    private List<V> values = new ArrayList<>();
    @Override
    public V put(K key, V value) {
        V oldValue = get(key); // The old value or null
        if(!keys.contains(key)) {
            keys.add(key);
            values.add(value);
        } else
            values.set(keys.indexOf(key), value);
        return oldValue;
    }
    @Override
    public V get(Object key) { // key: type Object, not K
        if(!keys.contains(key))
            return null;
        return values.get(keys.indexOf(key));
    }
    @Override
    public Set<Map.Entry<K, V>> entrySet() {
        Set<Map.Entry<K, V>> set= new HashSet<>();
        Iterator<K> ki = keys.iterator();
        Iterator<V> vi = values.iterator();
        while(ki.hasNext())
            set.add(new MapEntry<>(ki.next(), vi.next()));
        return set;
    }
    public static void main(String[] args) {
        SlowMap<String,String> m= new SlowMap<>();
        m.putAll(Countries.capitals(8));
        m.forEach((k, v) ->
                System.out.println(k + "=" + v));
        System.out.println(m.get("BENIN"));
        m.entrySet().forEach(System.out::println);
    }
}
/* Output:
CAMEROON=Yaounde
ANGOLA=Luanda
BURKINA FASO=Ouagadougou
BURUNDI=Bujumbura
ALGERIA=Algiers
BENIN=Porto-Novo
CAPE VERDE=Praia
BOTSWANA=Gaberone
Porto-Novo
CAMEROON=Yaounde
ANGOLA=Luanda
BURKINA FASO=Ouagadougou
BURUNDI=Bujumbura
ALGERIA=Algiers
BENIN=Porto-Novo
CAPE VERDE=Praia
BOTSWANA=Gaberone
*/
```

put()方法只是將鍵與值放入相應的ArrayList。為了與Map介面保持一致，它必須返回舊的鍵，或者在沒有任何舊鍵的情況下返回null。

同樣遵循了Map規範，get()會在鍵不在SlowMap中的時候產生null。如果鍵存在，它將被用來尋找表示它在keys列表中的位置的數值型索引，並且這個數字被用作索引來產生與values列表相關聯的值。注意，在get()中key的類型是Object，而不是你所期望的參數化類型K（並且是在AssociativeArrayjava中真正使用的類型），這是將泛型注入到Java語言中的時刻如此之晚所導致的結果-如果泛型是Java語言最初就具備的屬性，那麼get()就可以執行其參數的類型。

Map.entrySet() 方法必須產生一個Map.Entry物件集。但是，Map.Entry是一個介面，用來描述依賴於實現的結構，因此如果你想要建立自己的Map類型，就必須同時定義Map.Entry的實現：

```java
// equalshashcode/MapEntry.java
// A simple Map.Entry for sample Map implementations
import java.util.*;
public class MapEntry<K, V> implements Map.Entry<K, V> {
    private K key;
    private V value;
    public MapEntry(K key, V value) {
        this.key = key;
        this.value = value;
    }
    @Override
    public K getKey() { return key; }
    @Override
    public V getValue() { return value; }
    @Override
    public V setValue(V v) {
        V result = value;
        value = v;
        return result;
    }
    @Override
    public int hashCode() {
        return Objects.hash(key, value);
    }
    @SuppressWarnings("unchecked")
    @Override
    public boolean equals(Object rval) {
        return rval instanceof MapEntry &&
                Objects.equals(key,
                        ((MapEntry<K, V>)rval).getKey()) &&
                Objects.equals(value,
                        ((MapEntry<K, V>)rval).getValue());
    }
    @Override
    public String toString() {
        return key + "=" + value;
    }
}
```

這裡 equals 方法的實現遵循了[equals 規範]()。在 Objects 類中有一個非常熟悉的方法可以幫助建立 hashCode() 方法： Objects.hash()。當你定義含有超過一個屬性的物件的 `hashCode()` 時，你可以使用這個方法。如果你的物件只有一個屬性，可以直接使用 ` Objects.hashCode()`。

儘管這個解決方案非常簡單，並且看起來在SlowMap.main() 的瑣碎測試中可以正常工作，但是這並不是一個恰當的實現，因為它建立了鍵和值的副本。entrySet()  的恰當實現應該在Map中提供檢視，而不是副本，並且這個檢視允許對原始映射表進行修改（副本就不行）。

### 為了速度而散列

SlowMap.java 說明了建立一種新的Map並不困難。但是正如它的名稱SlowMap所示，它不會很快，所以如果有更好的選擇，就應該放棄它。它的問題在於對鍵的查詢，鍵沒有按照任何特定順序儲存，所以只能使用簡單的線性查詢，而線性查詢是最慢的查詢方式。

散列的價值在於速度：散列使得查詢得以快速進行。由於瓶頸位於鍵的查詢速度，因此解決方案之一就是保持鍵的排序狀態，然後使用Collections.binarySearch()進行查詢。

散列則更進一步，它將鍵儲存在某處，以便能夠很快找到。儲存一組元素最快的資料結構是陣列，所以使用它來表示鍵的訊息（請小心留意，我是說鍵的訊息，而不是鍵本身）。但是因為陣列不能調整容量，因此就有一個問題：我們希望在Map中儲存數量不確定的值，但是如果鍵的數量被陣列的容量限制了，該怎麼辦呢？

答案就是：陣列並不儲存鍵本身。而是透過鍵物件生成一個數字，將其作為陣列的下標。這個數字就是散列碼，由定義在Object中的、且可能由你的類覆蓋的hashCode()方法（在電腦科學的術語中稱為散列函數）生成。

於是查詢一個值的過程首先就是計算散列碼，然後使用散列碼查詢陣列。如果能夠保證沒有衝突（如果值的數量是固定的，那麼就有可能），那可就有了一個完美的散列函數，但是這種情況只是特例。[^1]。通常，衝突由外部連結處理：陣列並不直接儲存值，而是儲存值的 list。然後對 list中的值使用equals()方法進行線性的查詢。這部分的查詢自然會比較慢，但是，如果散列函數好的話，陣列的每個位置就只有較少的值。因此，不是查詢整個list，而是快速地跳到陣列的某個位置，只對很少的元素進行比較。這便是HashMap會如此快的原因。

理解了散列的原理，我們就能夠實現一個簡單的散列Map了：

```java
// equalshashcode/SimpleHashMap.java
// A demonstration hashed Map
import java.util.*;
import onjava.*;
public
class SimpleHashMap<K, V> extends AbstractMap<K, V> {
    // Choose a prime number for the hash table
// size, to achieve a uniform distribution:
    static final int SIZE = 997;
    // You can't have a physical array of generics,
// but you can upcast to one:
    @SuppressWarnings("unchecked")
    LinkedList<MapEntry<K, V>>[] buckets =
            new LinkedList[SIZE];
    @Override
    public V put(K key, V value) {
        V oldValue = null;
        int index = Math.abs(key.hashCode()) % SIZE;
        if(buckets[index] == null)
            buckets[index] = new LinkedList<>();
        LinkedList<MapEntry<K, V>> bucket = buckets[index];
        MapEntry<K, V> pair = new MapEntry<>(key, value);
        boolean found = false;
        ListIterator<MapEntry<K, V>> it =
                bucket.listIterator();
        while(it.hasNext()) {
            MapEntry<K, V> iPair = it.next();
            if(iPair.getKey().equals(key)) {
                oldValue = iPair.getValue();
                it.set(pair); // Replace old with new
                found = true;
                break;
            }
        }
        if(!found)
            buckets[index].add(pair);
        return oldValue;
    }
    @Override
    public V get(Object key) {
        int index = Math.abs(key.hashCode()) % SIZE;
        if(buckets[index] == null) return null;
        for(MapEntry<K, V> iPair : buckets[index])
            if(iPair.getKey().equals(key))
                return iPair.getValue();
        return null;
    }
    @Override
    public Set<Map.Entry<K, V>> entrySet() {
        Set<Map.Entry<K, V>> set= new HashSet<>();
        for(LinkedList<MapEntry<K, V>> bucket : buckets) {
            if(bucket == null) continue;
            for(MapEntry<K, V> mpair : bucket)
                set.add(mpair);
        }
        return set;
    }
    public static void main(String[] args) {
        SimpleHashMap<String,String> m =
                new SimpleHashMap<>();
        m.putAll(Countries.capitals(8));
        m.forEach((k, v) ->
                System.out.println(k + "=" + v));
        System.out.println(m.get("BENIN"));
        m.entrySet().forEach(System.out::println);
    }
}
/* Output:
CAMEROON=Yaounde
ANGOLA=Luanda
BURKINA FASO=Ouagadougou
BURUNDI=Bujumbura
ALGERIA=Algiers
BENIN=Porto-Novo
CAPE VERDE=Praia
BOTSWANA=Gaberone
Porto-Novo
CAMEROON=Yaounde
ANGOLA=Luanda
BURKINA FASO=Ouagadougou
BURUNDI=Bujumbura
ALGERIA=Algiers
BENIN=Porto-Novo
CAPE VERDE=Praia
BOTSWANA=Gaberone
*/
```

由於散列表中的“槽位”（slot）通常稱為桶位（bucket），因此我們將表示實際散列表的陣列命名為bucket，為使散列分布均勻，桶的數量通常使用質數[^2]。注意，為了能夠自動處理衝突，使用了一個LinkedList的陣列；每一個新的元素只是直接添加到list尾的某個特定桶位中。即使Java不允許你建立泛型陣列，那你也可以建立指向這種陣列的引用。這裡，向上轉型為這種陣列是很方便的，這樣可以防止在後面的程式碼中進行額外的轉型。

對於put()  方法，hashCode() 將針對鍵而被呼叫，並且其結果被強制轉換為正數。為了使產生的數字適合bucket陣列的大小，取模操作符將按照該陣列的尺寸取模。如果陣列的某個位置是 null，這表示還沒有元素被散列至此，所以，為了儲存剛散列到該定位的物件，需要建立一個新的LinkedList。一般的過程是，查看目前位置的ist中是否有相同的元素，如果有，則將舊的值賦給oldValue，然後用新的值取代舊的值。標記found用來跟蹤是否找到（相同的）舊的鍵值對，如果沒有，則將新的對添加到list的末尾。

get()方法按照與put()方法相同的方式計算在buckets陣列中的索引（這很重要，因為這樣可以保證兩個方法可以計算出相同的位置）如果此位置有LinkedList存在，就對其進行查詢。

注意，這個實現並不意味著對性能進行了調優，它只是想要展示散列映射表執行的各種操作。如果你瀏覽一下java.util.HashMap的原始碼，你就會看到一個調過優的實現。同樣，為了簡單，SimpleHashMap使用了與SlowMap相同的方式來實現entrySet()，這個方法有些過於簡單，不能用於通用的Map。

### 重寫 hashCode()

在明白了如何散列之後，編寫自己的hashCode()就更有意義了。

首先，你無法控制bucket陣列的下標值的產生。這個值依賴於具體的HashMap物件的容量，而容量的改變與容器的充滿程度和負載因子（本章稍後會介紹這個術語）有關。hashCode()生成的結果，經過處理後成為桶位的下標（在SimpleHashMap中，只是對其取模，模數為bucket陣列的大小）。

設計hashCode()時最重要的因素就是：無論何時，對同一個物件呼叫hashCode()都應該生成同樣的值。如果在將一個物件用put()添加進HashMap時產生一個hashCode()值，而用get()取出時卻產生了另一個hashCode()值，那麼就無法重新取得該物件了。所以，如果你的hashCode()方法依賴於物件中易變的資料，使用者就要當心了，因為此資料發生變化時，hashCode()就會生成一個不同的散列碼，相當於產生了一個不同的鍵。

此外，也不應該使hashCode()依賴於具有唯一性的物件訊息，尤其是使用this值，這只能產生很糟糕的hashCode()，因為這樣做無法生成一個新的鍵，使之與put()中原始的鍵值對中的鍵相同。這正是SpringDetector.java的問題所在，因為它預設的hashCode0使用的是物件的地址。所以，應該使用物件內有意義的識別訊息。

下面以String類為例。String有個特點：如果程式中有多個String物件，都包含相同的字串序列，那麼這些String物件都映射到同一塊記憶體區域。所以new String("hello")生成的兩個實例，雖然是相互獨立的，但是對它們使用hashCode()應該生成同樣的結果。透過下面的程式可以看到這種情況：

```java
// equalshashcode/StringHashCode.java
public class StringHashCode {
    public static void main(String[] args) {
        String[] hellos = "Hello Hello".split(" ");
        System.out.println(hellos[0].hashCode());
        System.out.println(hellos[1].hashCode());
    }
}
/* Output:
69609650
69609650
*/
```

對於String而言，hashCode() 明顯是基於String的內容的。

因此，要想使hashCode()  實用，它必須速度快，並且必須有意義。也就是說，它必須基於物件的內容生成散列碼。記得嗎，散列碼不必是獨一無二的（應該更關注生成速度，而不是唯一性），但是透過 hashCode() 和 equals() ，必須能夠完全確定物件的身份。

因為在生成桶的下標前，hashCode()還需要做進一步的處理，所以散列碼的生成範圍並不重要，只要是int即可。

還有另一個影響因素：好的hashCode() 應該產生分布均勻的散列碼。如果散列碼都集中在一塊，那麼HashMap或者HashSet在某些區域的負載會很重，這樣就不如分布均勻的散列函數快。

在Effective Java Programming Language Guide（Addison-Wesley 2001）這本書中，Joshua Bloch為怎樣寫出一份像樣的hashCode()給出了基本的指導： 

1. 給int變數result賦予某個非零值常量，例如17。
2. 為物件內每個有意義的欄位（即每個可以做equals）操作的欄位計算出一個int散列碼c：

| 欄位類型                                               | 計算公式                                                     |
| ------------------------------------------------------ | ------------------------------------------------------------ |
| boolean                                                | c = (f ? 0 : 1)                                              |
| byte ,  char ,  short , or  int                        | c = (int)f                                                   |
| long                                                   | c = (int)(f ^ (f>>>32))                                      |
| float                                                  | c = Float.floatToIntBits(f);                                 |
| double                                                 | long l =Double.doubleToLongBits(f); <br>c = (int)(l ^ (l >>> 32)) |
| Object , where  equals() calls equals() for this field | c = f.hashCode()                                             |
| Array                                                  | 應用以上規則到每一個元素中                                   |

3. 合併計算得到的散列碼： **result = 37 * result + c;​**
4.  返回 result。
5. 檢查hashCode()最後生成的結果，確保相同的物件有相同的散列碼。

下面便是遵循這些指導的一個例子。提示，你沒有必要書寫像如下的程式碼 —— 相反，使用 `Objects.hash()` 去用於散列多欄位的物件（如同在本例中的那樣），然後使用 `Objects.hashCode()` 如散列單欄位的物件。

```java
// equalshashcode/CountedString.java
// Creating a good hashCode()
import java.util.*;
public class CountedString {
    private static List<String> created =
            new ArrayList<>();
    private String s;
    private int id = 0;
    public CountedString(String str) {
        s = str;
        created.add(s);
// id is the total number of instances
// of this String used by CountedString:
        for(String s2 : created)
            if(s2.equals(s))
                id++;
    }
    @Override
    public String toString() {
        return "String: " + s + " id: " + id +
                " hashCode(): " + hashCode();
    }
    @Override
    public int hashCode() {
// The very simple approach:
// return s.hashCode() * id;
// Using Joshua Bloch's recipe:
        int result = 17;
        result = 37 * result + s.hashCode();
        result = 37 * result + id;
        return result;
    }
    @Override
    public boolean equals(Object o) {
        return o instanceof CountedString &&
                Objects.equals(s, ((CountedString)o).s) &&
                Objects.equals(id, ((CountedString)o).id);
    }
    public static void main(String[] args) {
        Map<CountedString,Integer> map = new HashMap<>();
        CountedString[] cs = new CountedString[5];
        for(int i = 0; i < cs.length; i++) {
            cs[i] = new CountedString("hi");
            map.put(cs[i], i); // Autobox int to Integer
        }
        System.out.println(map);
        for(CountedString cstring : cs) {
            System.out.println("Looking up " + cstring);
            System.out.println(map.get(cstring));
        }
    }
}
/* Output:
{String: hi id: 4 hashCode(): 146450=3, String: hi id:
5 hashCode(): 146451=4, String: hi id: 2 hashCode():
146448=1, String: hi id: 3 hashCode(): 146449=2,
String: hi id: 1 hashCode(): 146447=0}
Looking up String: hi id: 1 hashCode(): 146447
0
Looking up String: hi id: 2 hashCode(): 146448
1
Looking up String: hi id: 3 hashCode(): 146449
2
Looking up String: hi id: 4 hashCode(): 146450
3
Looking up String: hi id: 5 hashCode(): 146451
4
*/
```

CountedString由一個String和一個id組成，此id代表包含相同String的CountedString物件的編號。所有的String都被儲存在static ArrayList中，在構造器中透過選代遍歷此ArrayList完成對id的計算。

hashCode()和equals() 都基於CountedString的這兩個欄位來生成結果；如果它們只基於String或者只基於id，不同的物件就可能產生相同的值。

在main）中，使用相同的String建立了多個CountedString物件。這說明，雖然String相同，但是由於id不同，所以使得它們的散列碼並不相同。在程式中，HashMap被列印了出來，因此可以看到它內部是如何儲存元素的（以無法辨別的次序），然後單獨查詢每一個鍵，以此證明查詢機制工作正常。

作為第二個範例，請考慮Individual類，它被用作[類型訊息]()中所定義的typeinfo.pet類庫的基類。Individual類在那一章中就用到了，而它的定義則放到了本章，因此你可以正確地理解其實現。

在這裡取代了手工去計算 `hashCode()`，我們使用了更合適的方式 ` Objects.hash() `：

```java
// typeinfo/pets/Individual.java
package typeinfo.pets;
import java.util.*;
public class
Individual implements Comparable<Individual> {
    private static long counter = 0;
    private final long id = counter++;
    private String name;
    public Individual(String name) { this.name = name; }
    // 'name' is optional:
    public Individual() {}
    @Override
    public String toString() {
        return getClass().getSimpleName() +
                (name == null ? "" : " " + name);
    }
    public long id() { return id; }
    @Override
    public boolean equals(Object o) {
        return o instanceof Individual &&
                Objects.equals(id, ((Individual)o).id);
    }
    @Override
    public int hashCode() {
        return Objects.hash(name, id);
    }
    @Override
    public int compareTo(Individual arg) {
        // Compare by class name first:
        String first = getClass().getSimpleName();
        String argFirst = arg.getClass().getSimpleName();
        int firstCompare = first.compareTo(argFirst);
        if(firstCompare != 0)
            return firstCompare;
        if(name != null && arg.name != null) {
            int secondCompare = name.compareTo(arg.name);
            if(secondCompare != 0)
                return secondCompare;
        }
        return (arg.id < id ? -1 : (arg.id == id ? 0 : 1));
    }
}
```

compareTo() 方法有一個比較結構，因此它會產生一個排序序列，排序的規則首先按照實際類型排序，然後如果有名字的話，按照name排序，最後按照建立的順序排序。下面的範例說明了它是如何工作的：

```java
// equalshashcode/IndividualTest.java
import collections.MapOfList;
import typeinfo.pets.*;
import java.util.*;
public class IndividualTest {
    public static void main(String[] args) {
        Set<Individual> pets = new TreeSet<>();
        for(List<? extends Pet> lp :
                MapOfList.petPeople.values())
            for(Pet p : lp)
                pets.add(p);
        pets.forEach(System.out::println);
    }
}
/* Output:
Cat Elsie May
Cat Pinkola
Cat Shackleton
Cat Stanford
Cymric Molly
Dog Margrett
Mutt Spot
Pug Louie aka Louis Snorkelstein Dupree
Rat Fizzy
Rat Freckly
Rat Fuzzy
*/
```

由於所有的寵物都有名字，因此它們首先按照類型排序，然後在同類型中按照名字排序。

<!-- Tuning a HashMap -->

## 調優 HashMap

我們有可能手動調優HashMap以提高其在特定應用程式中的效能。為了理解調整HashMap時的效能問題，一些術語是必要的：

- 容量（Capacity）：表中儲存的桶數量。
- 初始容量（Initial Capacity）：當表被建立時，桶的初始個數。 HashMap 和 HashSet 有可以讓你指定初始容量的構造器。
- 個數（Size）：目前儲存在表中的鍵值對的個數。
- 負載因子（Load factor）：通常表現為 $\frac{size}{capacity}$。當負載因子大小為 0 的時候表示為一個空表。當負載因子大小為 0.5 表示為一個半滿表（half-full table），以此類推。輕負載的表幾乎沒有衝突，因此是插入和尋找的最佳選擇（但會減慢使用疊代器進行遍歷的過程）。 HashMap 和 HashSet 有可以讓你指定負載因子的構造器。當表內容量達到了負載因子，集合就會自動擴充為原始容量（桶的數量）的兩倍，並且會將原始的物件儲存在新的桶集合中（也被稱為 rehashing）

HashMap 中負載因子的大小為 0.75（當表內容量大小不足四分之三的時候，不會發生 rehashing 現象）。這看起來是一個非常好的同時考慮到時間和空間消耗的平衡策略。更高的負載因子會減少空間的消耗，但是會增加查詢的耗時。重要的是，查詢操作是你使用的最頻繁的一個操作（包括 `get()` 和 `put()` 方法）。

如果你知道儲存在 HashMap 中確切的條目個數，直接建立一個足夠容量大小的 HashMap，以避免自動發生的 rehashing 操作。

[^1]: 
[^2]:  事實證明，質數實際上並不是散列桶的理想容量。近來，（經過廣泛的測試）Java的散列函數都使用2的整數次方。對現代的處理器來說，除法與求餘數是最慢的操作。使用2的整數次方長度的散列表，可用掩碼代替除法。

<!-- 分頁 -->

<div style="page-break-after: always;"></div>
