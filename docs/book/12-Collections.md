[TOC]

<!-- Collections -->
# 第十二章 集合

> 如果一個程式只包含固定數量的物件且物件的生命週期都是已知的，那麼這是一個非常簡單的程式。

通常，程式總是根據執行時才知道的某些條件去建立新的物件。在此之前，無法知道所需物件的數量甚至確切類型。為了解決這個普遍的程式問題，需要在任意時刻和任意位置建立任意數量的物件。因此，不能依靠建立命名的引用來持有每一個物件：
```java
MyType aReference;
```
因為從來不會知道實際需要多少個這樣的引用。

大多數程式語言都提供了某種方法來解決這個基本問題。Java有多種方式儲存物件（確切地說，是物件的引用）。例如前面曾經學習過的陣列，它是編譯器支援的類型。陣列是儲存一組物件的最有效的方式，如果想要儲存一組基本類型資料，也推薦使用陣列。但是陣列具有固定的大小尺寸，而且在更一般的情況下，在寫程式的時候並不知道將需要多少個物件，或者是否需要更複雜的方式來儲存物件，因此陣列尺寸固定這一限制就顯得太過受限了。

**java.util** 庫提供了一套相當完整的*集合類*（collection classes）來解決這個問題，其中基本的類型有 **List** 、 **Set** 、 **Queue** 和 **Map**。這些類型也被稱作*容器類*（container classes），但我將使用Java類庫使用的術語。集合提供了完善的方法來儲存物件，可以使用這些工具來解決大量的問題。

集合還有一些其它特性。例如， **Set** 對於每個值都只儲存一個物件， **Map** 是一個關聯陣列，允許將某些物件與其他物件關聯起來。Java集合類都可以自動地調整自己的大小。因此，與陣列不同，在編程時，可以將任意數量的物件放置在集合中，而不用關心集合應該有多大。

儘管在 Java 中沒有直接的關鍵字支援[^1]，但集合類仍然是可以顯著增強編程能力的基本工具。在本章中，將介紹 Java 集合類庫的基本知識，並重點介紹一些典型用法。這裡將專注於在日常編程中使用的集合。稍後，在[附錄：集合主題]()中，還將學習到其餘的那些集合和相關功能，以及如何使用它們的更多詳細訊息。

<!-- Generics and Type-Safe Collections -->
## 泛型和類型安全的集合

使用 Java 5 之前的集合的一個主要問題是編譯器允許你向集合中插入不正確的類型。例如，考慮一個 **Apple** 物件的集合，這裡使用最基本最可靠的 **ArrayList** 。現在，可以把 **ArrayList** 看作“可以自動擴充自身尺寸的陣列”來看待。使用 **ArrayList** 相當簡單：建立一個實例，用 `add()` 插入物件；然後用 `get()` 來訪問這些物件，此時需要使用索引，就像陣列那樣，但是不需要方括號[^2]。 **ArrayList** 還有一個 `size()` 方法，來說明集合中包含了多少個元素，所以不會不小心因陣列越界而引發錯誤（透過拋出*執行時異常*，[異常]()章節介紹了異常）。

在本例中， **Apple** 和 **Orange** 都被放到了集合中，然後將它們取出。正常情況下，Java編譯器會給出警告，因為這個範例沒有使用泛型。在這裡，使用特定的註解來抑制警告訊息。註解以“@”符號開頭，可以帶參數。這裡的 `@SuppressWarning` 註解及其參數表示只抑制“unchecked”類型的警告（[註解]()章節將介紹更多有關注解的訊息）：

```java
// collections/ApplesAndOrangesWithoutGenerics.java
// Simple collection use (suppressing compiler warnings)
// {ThrowsException}
import java.util.*;

class Apple {
  private static long counter;
  private final long id = counter++;
  public long id() { return id; }
}

class Orange {}

public class ApplesAndOrangesWithoutGenerics {
  @SuppressWarnings("unchecked")
  public static void main(String[] args) {
    ArrayList apples = new ArrayList();
    for(int i = 0; i < 3; i++)
      apples.add(new Apple());
    // No problem adding an Orange to apples:
    apples.add(new Orange());
    for(Object apple : apples) {
      ((Apple) apple).id();
      // Orange is detected only at run time
    }
  }
}
/* Output:
___[ Error Output ]___
Exception in thread "main"
java.lang.ClassCastException: Orange cannot be cast to
Apple
        at ApplesAndOrangesWithoutGenerics.main(ApplesA
ndOrangesWithoutGenerics.java:23)
*/
```

**Apple** 和 **Orange** 是截然不同的，它們除了都是 **Object** 之外沒有任何共同點（如果一個類沒有顯式地聲明繼承自哪個類，那麼它就自動繼承自 **Object**）。因為 **ArrayList** 儲存的是 **Object** ，所以不僅可以透過 **ArrayList** 的 `add()` 方法將 **Apple** 物件放入這個集合，而且可以放入 **Orange** 物件，這無論在編譯期還是執行時都不會有問題。當使用 **ArrayList** 的 `get()` 方法來取出你認為是 **Apple** 的物件時，得到的只是 **Object** 引用，必須將其轉型為 **Apple**。然後需要將整個表達式用括號括起來，以便在呼叫 **Apple** 的 `id()` 方法之前，強制執行轉型。否則，將會產生語法錯誤。

在執行時，當嘗試將 **Orange** 物件轉為 **Apple** 時，會出現輸出中顯示的錯誤。

在[泛型]()章節中，你將了解到使用 Java 泛型來建立類可能很複雜。但是，使用預先定義的泛型類卻相當簡單。例如，要定義一個用於儲存 **Apple** 物件的 **ArrayList** ，只需要使用 **ArrayList\<Apple\>** 來代替 **ArrayList** 。角括號括起來的是*類型參數*（可能會有多個），它指定了這個集合實例可以儲存的類型。

透過使用泛型，就可以在編譯期防止將錯誤類型的物件放置到集合中[^3]。下面還是這個範例，但是使用了泛型：
```java
// collections/ApplesAndOrangesWithGenerics.java
import java.util.*;

public class ApplesAndOrangesWithGenerics {
  public static void main(String[] args) {
    ArrayList<Apple> apples = new ArrayList<>();
    for(int i = 0; i < 3; i++)
      apples.add(new Apple());
    // Compile-time error:
    // apples.add(new Orange());
    for(Apple apple : apples) {
      System.out.println(apple.id());
    }
  }
}
/* Output:
0
1
2
*/
```

在 **apples** 定義的右側，可以看到 `new ArrayList<>()` 。這有時被稱為“菱形語法”（diamond syntax）。在 Java 7 之前，必須要在兩端都進行類型聲明，如下所示：

```java
ArrayList<Apple> apples = new ArrayList<Apple>();
```

隨著類型變得越來越複雜，這種重複產生的程式碼非常混亂且難以閱讀。程式設計師發現所有類型訊息都可以從左側獲得，因此，編譯器沒有理由強迫右側再重複這些。雖然*類型推斷*（type inference）只是個很小的請求，Java 語言團隊仍然欣然接受並進行了改進。

有了 **ArrayList** 聲明中的類型指定，編譯器會阻止將 **Orange** 放入 **apples** ，因此，這會成為一個編譯期錯誤而不是執行時錯誤。

使用泛型，從 **List** 中獲取元素不需要強制類型轉換。因為 **List** 知道它持有什麼類型，因此當呼叫 `get()` 時，它會替你執行轉型。因此，使用泛型，你不僅知道編譯器將檢查放入集合的物件類型，而且在使用集合中的物件時也可以獲得更清晰的語法。

當指定了某個類型為泛型參數時，並不僅限於只能將確切類型的物件放入集合中。向上轉型也可以像作用於其他類型一樣作用於泛型：
```java
// collections/GenericsAndUpcasting.java
import java.util.*;

class GrannySmith extends Apple {}
class Gala extends Apple {}
class Fuji extends Apple {}
class Braeburn extends Apple {}

public class GenericsAndUpcasting {
  public static void main(String[] args) {
    ArrayList<Apple> apples = new ArrayList<>();
    apples.add(new GrannySmith());
    apples.add(new Gala());
    apples.add(new Fuji());
    apples.add(new Braeburn());
    for(Apple apple : apples)
      System.out.println(apple);
  }
}
/* Output:
GrannySmith@15db9742
Gala@6d06d69c
Fuji@7852e922
Braeburn@4e25154f
*/
```

因此，可以將 **Apple** 的子類型添加到被指定為儲存 **Apple** 物件的集合中。

程式的輸出是從 **Object** 預設的 `toString()` 方法產生的，該方法列印類名，後面跟著物件的散列碼的無符號十六進位制表示（這個散列碼是透過 `hashCode()` 方法產生的）。將在[附錄：理解equals和hashCode方法]()中了解有關散列碼的內容。

<!-- Basic Concepts -->
## 基本概念

Java集合類庫採用“持有物件”（holding objects）的思想，並將其分為兩個不同的概念，表示為類庫的基本介面：

1. **集合（Collection）** ：一個獨立元素的序列，這些元素都服從一條或多條規則。**List** 必須以插入的順序儲存元素， **Set** 不能包含重複元素， **Queue** 按照*排隊規則*來確定物件產生的順序（通常與它們被插入的順序相同）。
2. **映射（Map）** ： 一組成對的“鍵值對”物件，允許使用鍵來尋找值。 **ArrayList** 使用數字來尋找物件，因此在某種意義上講，它是將數字和物件關聯在一起。  **map** 允許我們使用一個物件來尋找另一個物件，它也被稱作*關聯陣列*（associative array），因為它將物件和其它物件關聯在一起；或者稱作*字典*（dictionary），因為可以使用一個鍵物件來尋找值物件，就像在字典中使用單詞尋找定義一樣。 **Map** 是強大的程式工具。

儘管並非總是可行，但在理想情況下，你編寫的大部分程式碼都在與這些介面打交道，並且唯一需要指定所使用的精確類型的地方就是在建立的時候。因此，可以像下面這樣建立一個 **List** ：

```java
List<Apple> apples = new ArrayList<>();
```

請注意， **ArrayList** 已經被向上轉型為了 **List** ，這與之前範例中的處理方式正好相反。使用介面的目的是，如果想要改變具體實現，只需在建立時修改它就行了，就像下面這樣：

```java
List<Apple> apples = new LinkedList<>();
```

因此，應該建立一個具體類的物件，將其向上轉型為對應的介面，然後在其餘程式碼中都是用這個介面。

這種方式並非總是有效的，因為某些具體類有額外的功能。例如， **LinkedList** 具有 **List** 介面中未包含的額外方法，而 **TreeMap** 也具有在 **Map** 介面中未包含的方法。如果需要使用這些方法，就不能將它們向上轉型為更通用的介面。

**Collection** 介面概括了*序列*的概念——一種存放一組物件的方式。下面是個簡單的範例，用 **Integer** 物件填充了一個 **Collection** （這裡用 **ArrayList** 表示），然後列印集合中的每個元素：
```java
// collections/SimpleCollection.java
import java.util.*;

public class SimpleCollection {
  public static void main(String[] args) {
    Collection<Integer> c = new ArrayList<>();
    for(int i = 0; i < 10; i++)
      c.add(i); // Autoboxing
    for(Integer i : c)
      System.out.print(i + ", ");
  }
}
/* Output:
0, 1, 2, 3, 4, 5, 6, 7, 8, 9,
*/
```

這個例子僅使用了 **Collection** 中的方法（即 `add()` ），所以使用任何繼承自 **Collection** 的類的物件都可以正常工作。但是 **ArrayList** 是最基本的序列類型。

`add()` 方法的名稱就表明它是在 **Collection** 中添加一個新元素。但是，文件中非常詳細地敘述到 `add()` “要確保這個 **Collection** 包含指定的元素。”這是因為考慮到了 **Set** 的含義，因為在 **Set**中，只有當元素不存在時才會添加元素。在使用 **ArrayList** ，或任何其他類型的 **List** 時，`add()` 總是表示“把它放進去”，因為 **List** 不關心是否存在重複元素。

可以使用 *for-in* 語法來遍歷所有的 **Collection** ，就像這裡所展示的那樣。在本章的後續部分，還將學習到一個更靈活的概念，*疊代器*。

<!-- Adding Groups of Elements -->

## 添加元素組

在 **java.util** 包中的 **Arrays** 和 **Collections** 類中都有很多實用的方法，可以在一個 **Collection** 中添加一組元素。

`Arrays.asList()` 方法接受一個陣列或是逗號分隔的元素列表（使用可變參數），並將其轉換為 **List** 物件。 `Collections.addAll()` 方法接受一個 **Collection** 物件，以及一個陣列或是一個逗號分隔的列表，將其中元素添加到 **Collection** 中。下面的範例展示了這兩個方法，以及更通用的 、所有 **Collection** 類型都包含的`addAll()` 方法：

```java
// collections/AddingGroups.java
// Adding groups of elements to Collection objects
import java.util.*;

public class AddingGroups {
  public static void main(String[] args) {
    Collection<Integer> collection =
      new ArrayList<>(Arrays.asList(1, 2, 3, 4, 5));
    Integer[] moreInts = { 6, 7, 8, 9, 10 };
    collection.addAll(Arrays.asList(moreInts));
    // Runs significantly faster, but you can't
    // construct a Collection this way:
    Collections.addAll(collection, 11, 12, 13, 14, 15);
    Collections.addAll(collection, moreInts);
    // Produces a list "backed by" an array:
    List<Integer> list = Arrays.asList(16,17,18,19,20);
    list.set(1, 99); // OK -- modify an element
    // list.add(21); // Runtime error; the underlying
                     // array cannot be resized.
  }
}
```

**Collection** 的構造器可以接受另一個 **Collection**，用它來將自身初始化。因此，可以使用 `Arrays.asList()` 來為這個構造器產生輸入。但是， `Collections.addAll()` 執行得更快，而且很容易構建一個不包含元素的 **Collection** ，然後呼叫 `Collections.addAll()` ，因此這是首選方式。

`Collection.addAll()` 方法只能接受另一個 **Collection** 作為參數，因此它沒有 `Arrays.asList()` 或 `Collections.addAll()` 靈活。這兩個方法都使用可變參數列表。

也可以直接使用 `Arrays.asList()` 的輸出作為一個 **List** ，但是這裡的底層實現是陣列，沒辦法調整大小。如果嘗試在這個 **List** 上呼叫 `add()` 或 `remove()`，由於這兩個方法會嘗試修改陣列大小，所以會在執行時得到“Unsupported Operation（不支援的操作）”錯誤：

```java
// collections/AsListInference.java
import java.util.*;

class Snow {}
class Powder extends Snow {}
class Light extends Powder {}
class Heavy extends Powder {}
class Crusty extends Snow {}
class Slush extends Snow {}

public class AsListInference {
  public static void main(String[] args) {
    List<Snow> snow1 = Arrays.asList(
      new Crusty(), new Slush(), new Powder());
    //- snow1.add(new Heavy()); // Exception

    List<Snow> snow2 = Arrays.asList(
      new Light(), new Heavy());
    //- snow2.add(new Slush()); // Exception

    List<Snow> snow3 = new ArrayList<>();
    Collections.addAll(snow3,
      new Light(), new Heavy(), new Powder());
    snow3.add(new Crusty());

    // Hint with explicit type argument specification:
    List<Snow> snow4 = Arrays.<Snow>asList(
       new Light(), new Heavy(), new Slush());
    //- snow4.add(new Powder()); // Exception
  }
}
```

在 **snow4** 中，注意 `Arrays.asList()` 中間的“暗示”（即 `<Snow>` ），告訴編譯器 `Arrays.asList()` 生成的結果 **List** 類型的實際目標類型是什麼。這稱為*顯式類型參數說明*（explicit type argument specification）。

<!-- Printing Collections -->
## 集合的列印

必須使用 `Arrays.toString()` 來生成陣列的可列印形式。但是列印集合無需任何幫助。下面是一個例子，這個例子中也介紹了基本的Java集合：
```java
// collections/PrintingCollections.java
// Collections print themselves automatically
import java.util.*;

public class PrintingCollections {
  static Collection
  fill(Collection<String> collection) {
    collection.add("rat");
    collection.add("cat");
    collection.add("dog");
    collection.add("dog");
    return collection;
  }
  static Map fill(Map<String, String> map) {
    map.put("rat", "Fuzzy");
    map.put("cat", "Rags");
    map.put("dog", "Bosco");
    map.put("dog", "Spot");
    return map;
  }
  public static void main(String[] args) {
    System.out.println(fill(new ArrayList<>()));
    System.out.println(fill(new LinkedList<>()));
    System.out.println(fill(new HashSet<>()));
    System.out.println(fill(new TreeSet<>()));
    System.out.println(fill(new LinkedHashSet<>()));
    System.out.println(fill(new HashMap<>()));
    System.out.println(fill(new TreeMap<>()));
    System.out.println(fill(new LinkedHashMap<>()));
  }
}
/* Output:
[rat, cat, dog, dog]
[rat, cat, dog, dog]
[rat, cat, dog]
[cat, dog, rat]
[rat, cat, dog]
{rat=Fuzzy, cat=Rags, dog=Spot}
{cat=Rags, dog=Spot, rat=Fuzzy}
{rat=Fuzzy, cat=Rags, dog=Spot}
*/
```

這顯示了Java集合庫中的兩個主要類型。它們的區別在於集合中的每個“槽”（slot）儲存的元素個數。 **Collection** 類型在每個槽中只能儲存一個元素。此類集合包括： **List** ，它以特定的順序儲存一組元素； **Set** ，其中元素不允許重複； **Queue** ，只能在集合一端插入物件，並從另一端移除物件（就本例而言，這只是查看序列的另一種方式，因此並沒有顯示它）。 **Map** 在每個槽中存放了兩個元素，即*鍵* (key)和與之關聯的*值* (value)。

預設的列印行為，使用集合提供的 `toString()` 方法即可生成可讀性很好的結果。 **Collection** 列印出的內容用方括號括住，每個元素由逗號分隔。 **Map** 則由大括號括住，每個鍵和值用等號連接（鍵在左側，值在右側）。

第一個 `fill()` 方法適用於所有類型的 **Collection** ，這些類型都實現了 `add()` 方法以添加新元素。

**ArrayList** 和 **LinkedList** 都是 **List** 的類型，從輸出中可以看出，它們都按插入順序儲存元素。兩者之間的區別不僅在於執行某些類型的操作時的效能，而且 **LinkedList** 包含的操作多於 **ArrayList** 。本章後面將對這些內容進行更全面的探討。

**HashSet** ， **TreeSet** 和 **LinkedHashSet** 是 **Set** 的類型。從輸出中可以看到， **Set** 僅儲存每個相同項中的一個，並且不同的 **Set** 實現儲存元素的方式也不同。 **HashSet** 使用相當複雜的方法儲存元素，這在[附錄：集合主題]()中進行了探討。現在只需要知道，這種技術是檢索元素的最快方法，因此，儲存順序看起來沒有什麼意義（通常只關心某事物是否是 **Set** 的成員，而儲存順序並不重要）。如果儲存順序很重要，則可以使用 **TreeSet** ，它把物件按照比較規則來排序；還有 **LinkedHashSet** ，它把物件按照被添加的先後順序來排序。

**Map** （也稱為*關聯陣列*）使用*鍵*來尋找物件，就像一個簡單的資料庫。所關聯的物件稱為*值*。 假設有一個 **Map** 將美國州名與它們的首府聯繫在一起，如果想要俄亥俄州（Ohio）的首府，可以用“Ohio”作為鍵來尋找，幾乎就像使用陣列下標一樣。正是由於這種行為，對於每個鍵， **Map** 只儲存一次。

`Map.put(key, value)` 添加一個所想要添加的值並將它與一個鍵（用來尋找值）相關聯。 `Map.get(key)` 生成與該鍵相關聯的值。上面的範例僅添加鍵值對，並沒有執行尋找。這將在稍後展示。

請注意，這裡沒有指定（或考慮） **Map** 的大小，因為它會自動調整大小。 此外， **Map** 還知道如何列印自己，它會顯示相關聯的鍵和值。

本例使用了 **Map** 的三種基本風格： **HashMap** ， **TreeMap** 和 **LinkedHashMap** 。

鍵和值儲存在 **HashMap** 中的順序不是插入順序，因為 **HashMap** 實現使用了非常快速的演算法來控制順序。 **TreeMap** 把所有的鍵按照比較規則來排序， **LinkedHashMap** 在保持 **HashMap** 尋找速度的同時按照鍵的插入順序來排序。

<!-- List -->

## 列表List

**List**承諾將元素儲存在特定的序列中。 **List** 介面在 **Collection** 的基礎上添加了許多方法，允許在 **List** 的中間插入和刪除元素。

有兩種類型的 **List** ：

- 基本的 **ArrayList** ，擅長隨機訪問元素，但在 **List** 中間插入和刪除元素時速度較慢。
- **LinkedList** ，它透過代價較低的在 **List** 中間進行的插入和刪除操作，提供了最佳化的順序訪問。 **LinkedList** 對於隨機訪問來說相對較慢，但它具有比 **ArrayList** 更大的特徵集。

下面的範例匯入 **typeinfo.pets** ，超前使用了[類型訊息]()一章中的類庫。這個類庫包含了 **Pet** 類層次結構，以及用於隨機生成 **Pet** 物件的一些工具類。此時不需要了解完整的詳細訊息，只需要知道兩點：

1. 有一個 **Pet** 類，以及 **Pet** 的各種子類型。
2. 靜態的 `Pets.arrayList()` 方法返回一個填充了隨機選取的 **Pet** 物件的 **ArrayList**：

```java
// collections/ListFeatures.java
import typeinfo.pets.*;
import java.util.*;

public class ListFeatures {
  public static void main(String[] args) {
    Random rand = new Random(47);
    List<Pet> pets = Pets.list(7);
    System.out.println("1: " + pets);
    Hamster h = new Hamster();
    pets.add(h); // Automatically resizes
    System.out.println("2: " + pets);
    System.out.println("3: " + pets.contains(h));
    pets.remove(h); // Remove by object
    Pet p = pets.get(2);
    System.out.println(
      "4: " +  p + " " + pets.indexOf(p));
    Pet cymric = new Cymric();
    System.out.println("5: " + pets.indexOf(cymric));
    System.out.println("6: " + pets.remove(cymric));
    // Must be the exact object:
    System.out.println("7: " + pets.remove(p));
    System.out.println("8: " + pets);
    pets.add(3, new Mouse()); // Insert at an index
    System.out.println("9: " + pets);
    List<Pet> sub = pets.subList(1, 4);
    System.out.println("subList: " + sub);
    System.out.println("10: " + pets.containsAll(sub));
    Collections.sort(sub); // In-place sort
    System.out.println("sorted subList: " + sub);
    // Order is not important in containsAll():
    System.out.println("11: " + pets.containsAll(sub));
    Collections.shuffle(sub, rand); // Mix it up
    System.out.println("shuffled subList: " + sub);
    System.out.println("12: " + pets.containsAll(sub));
    List<Pet> copy = new ArrayList<>(pets);
    sub = Arrays.asList(pets.get(1), pets.get(4));
    System.out.println("sub: " + sub);
    copy.retainAll(sub);
    System.out.println("13: " + copy);
    copy = new ArrayList<>(pets); // Get a fresh copy
    copy.remove(2); // Remove by index
    System.out.println("14: " + copy);
    copy.removeAll(sub); // Only removes exact objects
    System.out.println("15: " + copy);
    copy.set(1, new Mouse()); // Replace an element
    System.out.println("16: " + copy);
    copy.addAll(2, sub); // Insert a list in the middle
    System.out.println("17: " + copy);
    System.out.println("18: " + pets.isEmpty());
    pets.clear(); // Remove all elements
    System.out.println("19: " + pets);
    System.out.println("20: " + pets.isEmpty());
    pets.addAll(Pets.list(4));
    System.out.println("21: " + pets);
    Object[] o = pets.toArray();
    System.out.println("22: " + o[3]);
    Pet[] pa = pets.toArray(new Pet[0]);
    System.out.println("23: " + pa[3].id());
  }
}
/* Output:
1: [Rat, Manx, Cymric, Mutt, Pug, Cymric, Pug]
2: [Rat, Manx, Cymric, Mutt, Pug, Cymric, Pug, Hamster]
3: true
4: Cymric 2
5: -1
6: false
7: true
8: [Rat, Manx, Mutt, Pug, Cymric, Pug]
9: [Rat, Manx, Mutt, Mouse, Pug, Cymric, Pug]
subList: [Manx, Mutt, Mouse]
10: true
sorted subList: [Manx, Mouse, Mutt]
11: true
shuffled subList: [Mouse, Manx, Mutt]
12: true
sub: [Mouse, Pug]
13: [Mouse, Pug]
14: [Rat, Mouse, Mutt, Pug, Cymric, Pug]
15: [Rat, Mutt, Cymric, Pug]
16: [Rat, Mouse, Cymric, Pug]
17: [Rat, Mouse, Mouse, Pug, Cymric, Pug]
18: false
19: []
20: true
21: [Manx, Cymric, Rat, EgyptianMau]
22: EgyptianMau
23: 14
*/
```

列印行都編了號，因此可從輸出追溯到原始碼。 第 1 行輸出展示了原始的由 **Pet** 組成的 **List** 。 與陣列不同， **List** 可以在建立後添加或刪除元素，並自行調整大小。這正是它的重要價值：一種可修改的序列。在第 2 行輸出中可以看到添加一個 **Hamster** 的結果，該物件將被追加到列表的末尾。

可以使用 `contains()` 方法確定物件是否在列表中。如果要刪除一個物件，可以將該物件的引用傳遞給 `remove()` 方法。同樣，如果有一個物件的引用，可以使用 `indexOf()` 在 **List** 中找到該物件所在位置的下標號，如第 4 行輸出所示中所示。

當確定元素是否是屬於某個 **List** ，尋找某個元素的索引，以及透過引用從 **List** 中刪除元素時，都會用到 `equals()` 方法（根類 **Object** 的一個方法）。每個 **Pet** 被定義為一個唯一的物件，所以即使列表中已經有兩個 **Cymrics** ，如果再建立一個新的 **Cymric** 物件並將其傳遞給 `indexOf()` 方法，結果仍為 **-1** （表示未找到），並且嘗試呼叫 `remove()` 方法來刪除這個物件將返回 **false** 。對於其他類， `equals()` 的定義可能有所不同。例如，如果兩個 **String** 的內容相同，則這兩個 **String** 相等。因此，為了防止出現意外，請務必注意 **List** 行為會根據 `equals()` 行為而發生變化。

第 7、8 行輸出展示了刪除與 **List** 中的物件完全匹配的物件是成功的。

可以在 **List** 的中間插入一個元素，就像在第 9 行輸出和它之前的程式碼那樣。但這會帶來一個問題：對於 **LinkedList** ，在列表中間插入和刪除都是廉價操作（在本例中，除了對列表中間進行的真正的隨機訪問），但對於 **ArrayList** ，這可是代價高昂的操作。這是否意味著永遠不應該在 **ArrayList** 的中間插入元素，並最好是轉換為 **LinkedList** ？不，它只是意味著你應該意識到這個問題，如果你開始在某個 **ArrayList** 中間執行很多插入操作，並且程式開始變慢，那麼你應該看看你的 **List** 實現有可能就是罪魁禍首（發現此類瓶頸的最佳方式是使用分析器 profiler）。最佳化是一個很棘手的問題，最好的策略就是置之不顧，直到發現必須要去擔心它了（儘管去理解這些問題總是一個很好的主意）。

`subList()` 方法可以輕鬆地從更大的列表中建立切片，當將切片結果傳遞給原來這個較大的列表的 `containsAll()` 方法時，很自然地會得到 **true**。請注意，順序並不重要，在第 11、12 行輸出中可以看到，在 **sub** 上呼叫直觀命名的 `Collections.sort()` 和 `Collections.shuffle()` 方法，不會影響 `containsAll()` 的結果。 `subList()` 所產生的列表的幕後支援就是原始列表。因此，對所返回列表的更改都將會反映在原始列表中，反之亦然。

`retainAll()` 方法實際上是一個“集合交集”操作，在本例中，它保留了同時在 **copy** 和 **sub** 中的所有元素。請再次注意，所產生的結果行為依賴於 `equals()` 方法。

第 14 行輸出展示了使用索引號來刪除元素的結果，與透過物件引用來刪除元素相比，它顯得更加直觀，因為在使用索引時，不必擔心 `equals()` 的行為。

`removeAll()` 方法也是基於 `equals()` 方法執行的。 顧名思義，它會從 **List** 中刪除在參數 **List** 中的所有元素。

`set()` 方法的命名顯得很不合時宜，因為它與 **Set** 類存在潛在的衝突。在這裡使用“replace”可能更適合，因為它的功能是用第二個參數取代索引處的元素（第一個參數）。

第 17 行輸出表明，對於 **List** ，有一個重載的 `addAll()` 方法可以將新列表插入到原始列表的中間位置，而不是僅能用 **Collection** 的 `addAll()` 方法將其追加到列表的末尾。

第 18 - 20 行輸出展示了 `isEmpty()` 和 `clear()` 方法的效果。

第 22、23 行輸出展示了如何使用 `toArray()` 方法將任意的 **Collection** 轉換為陣列。這是一個重載方法，其無參版本返回一個 **Object** 陣列，但是如果將目標類型的陣列傳遞給這個重載版本，那麼它會生成一個指定類型的陣列（假設它通過了類型檢查）。如果參數陣列太小而無法容納 **List** 中的所有元素（就像本例一樣），則 `toArray()` 會建立一個具有合適尺寸的新陣列。 **Pet** 物件有一個 `id()` 方法，可以在所產生的陣列中的物件上呼叫這個方法。

<!-- Iterators -->

## 疊代器Iterators

在任何集合中，都必須有某種方式可以插入元素並再次獲取它們。畢竟，儲存事物是集合最基本的工作。對於 **List** ， `add()` 是插入元素的一種方式， `get()` 是獲取元素的一種方式。

如果從更高層次的角度考慮，會發現這裡有個缺點：要使用集合，必須對集合的確切類型編程。這一開始可能看起來不是很糟糕，但是考慮下面的情況：如果原本是對 **List** 編碼的，但是後來發現如果能夠將相同的程式碼應用於 **Set** 會更方便，此時應該怎麼做？或者假設想從一開始就編寫一段通用程式碼，它不知道或不關心它正在使用什麼類型的集合，因此它可以用於不同類型的集合，那麼如何才能不重寫程式碼就可以應用於不同類型的集合？

*疊代器*（也是一種設計模式）的概念實現了這種抽象。疊代器是一個物件，它在一個序列中移動並選擇該序列中的每個物件，而用戶端程式設計師不知道或不關心該序列的底層結構。另外，疊代器通常被稱為*輕量級物件*（lightweight object）：建立它的代價小。因此，經常可以看到一些對疊代器有些奇怪的約束。例如，Java 的 **Iterator** 只能單向移動。這個 **Iterator** 只能用來：

1. 使用 `iterator()` 方法要求集合返回一個 **Iterator**。 **Iterator** 將準備好返回序列中的第一個元素。
2. 使用 `next()` 方法獲得序列中的下一個元素。
3. 使用 `hasNext()` 方法檢查序列中是否還有元素。
4. 使用 `remove()` 方法將疊代器最近返回的那個元素刪除。

為了觀察它的工作方式，這裡再次使用[類型訊息]()章節中的 **Pet** 工具：

```java
// collections/SimpleIteration.java
import typeinfo.pets.*;
import java.util.*;

public class SimpleIteration {
  public static void main(String[] args) {
    List<Pet> pets = Pets.list(12);
    Iterator<Pet> it = pets.iterator();
    while(it.hasNext()) {
      Pet p = it.next();
      System.out.print(p.id() + ":" + p + " ");
    }
    System.out.println();
    // A simpler approach, when possible:
    for(Pet p : pets)
      System.out.print(p.id() + ":" + p + " ");
    System.out.println();
    // An Iterator can also remove elements:
    it = pets.iterator();
    for(int i = 0; i < 6; i++) {
      it.next();
      it.remove();
    }
    System.out.println(pets);
  }
}
/* Output:
0:Rat 1:Manx 2:Cymric 3:Mutt 4:Pug 5:Cymric 6:Pug
7:Manx 8:Cymric 9:Rat 10:EgyptianMau 11:Hamster
0:Rat 1:Manx 2:Cymric 3:Mutt 4:Pug 5:Cymric 6:Pug
7:Manx 8:Cymric 9:Rat 10:EgyptianMau 11:Hamster
[Pug, Manx, Cymric, Rat, EgyptianMau, Hamster]
*/
```

有了 **Iterator** ，就不必再為集合中元素的數量操心了。這是由 `hasNext()` 和 `next()` 關心的事情。

如果只是想向前遍歷 **List** ，並不打算修改 **List** 物件本身，那麼使用 *for-in* 語法更加簡潔。

**Iterator** 還可以刪除由 `next()` 生成的最後一個元素，這意味著在呼叫 `remove()` 之前必須先呼叫 `next()` 。[^4]

在集合中的每個物件上執行操作，這種思想十分強大，並且貫穿於本書。

現在考慮建立一個 `display()` 方法，它不必知曉集合的確切類型：

```java
// collections/CrossCollectionIteration.java
import typeinfo.pets.*;
import java.util.*;

public class CrossCollectionIteration {
  public static void display(Iterator<Pet> it) {
    while(it.hasNext()) {
      Pet p = it.next();
      System.out.print(p.id() + ":" + p + " ");
    }
    System.out.println();
  }
  public static void main(String[] args) {
    List<Pet> pets = Pets.list(8);
    LinkedList<Pet> petsLL = new LinkedList<>(pets);
    HashSet<Pet> petsHS = new HashSet<>(pets);
    TreeSet<Pet> petsTS = new TreeSet<>(pets);
    display(pets.iterator());
    display(petsLL.iterator());
    display(petsHS.iterator());
    display(petsTS.iterator());
  }
}
/* Output:
0:Rat 1:Manx 2:Cymric 3:Mutt 4:Pug 5:Cymric 6:Pug
7:Manx
0:Rat 1:Manx 2:Cymric 3:Mutt 4:Pug 5:Cymric 6:Pug
7:Manx
0:Rat 1:Manx 2:Cymric 3:Mutt 4:Pug 5:Cymric 6:Pug
7:Manx
5:Cymric 2:Cymric 7:Manx 1:Manx 3:Mutt 6:Pug 4:Pug
0:Rat
*/
```

`display()` 方法不包含任何有關它所遍歷的序列的類型訊息。這也展示了 **Iterator** 的真正威力：能夠將遍歷序列的操作與該序列的底層結構分離。出於這個原因，我們有時會說：疊代器統一了對集合的訪問方式。

我們可以使用 **Iterable** 介面生成上一個範例的更簡潔版本，該介面描述了“可以產生 **Iterator** 的任何東西”：

```java
// collections/CrossCollectionIteration2.java
import typeinfo.pets.*;
import java.util.*;

public class CrossCollectionIteration2 {
  public static void display(Iterable<Pet> ip) {
    Iterator<Pet> it = ip.iterator();
    while(it.hasNext()) {
      Pet p = it.next();
      System.out.print(p.id() + ":" + p + " ");
    }
    System.out.println();
  }
  public static void main(String[] args) {
    List<Pet> pets = Pets.list(8);
    LinkedList<Pet> petsLL = new LinkedList<>(pets);
    HashSet<Pet> petsHS = new HashSet<>(pets);
    TreeSet<Pet> petsTS = new TreeSet<>(pets);
    display(pets);
    display(petsLL);
    display(petsHS);
    display(petsTS);
  }
}
/* Output:
0:Rat 1:Manx 2:Cymric 3:Mutt 4:Pug 5:Cymric 6:Pug
7:Manx
0:Rat 1:Manx 2:Cymric 3:Mutt 4:Pug 5:Cymric 6:Pug
7:Manx
0:Rat 1:Manx 2:Cymric 3:Mutt 4:Pug 5:Cymric 6:Pug
7:Manx
5:Cymric 2:Cymric 7:Manx 1:Manx 3:Mutt 6:Pug 4:Pug
0:Rat
*/
```

這裡所有的類都是 **Iterable** ，所以現在對 `display()` 的呼叫顯然更簡單。

<!-- ListIterator -->
### ListIterator

**ListIterator** 是一個更強大的 **Iterator** 子類型，它只能由各種 **List** 類生成。 **Iterator** 只能向前移動，而 **ListIterator** 可以雙向移動。它可以生成疊代器在列表中指向位置的後一個和前一個元素的索引，並且可以使用 `set()` 方法取代它訪問過的最近一個元素。可以透過呼叫 `listIterator()` 方法來生成指向 **List** 開頭處的 **ListIterator** ，還可以透過呼叫 `listIterator(n)` 建立一個一開始就指向列表索引號為 **n** 的元素處的 **ListIterator** 。 下面的範例示範了所有這些能力：

```java
// collections/ListIteration.java
import typeinfo.pets.*;
import java.util.*;

public class ListIteration {
  public static void main(String[] args) {
    List<Pet> pets = Pets.list(8);
    ListIterator<Pet> it = pets.listIterator();
    while(it.hasNext())
      System.out.print(it.next() +
        ", " + it.nextIndex() +
        ", " + it.previousIndex() + "; ");
    System.out.println();
    // Backwards:
    while(it.hasPrevious())
      System.out.print(it.previous().id() + " ");
    System.out.println();
    System.out.println(pets);
    it = pets.listIterator(3);
    while(it.hasNext()) {
      it.next();
      it.set(Pets.get());
    }
    System.out.println(pets);
  }
}
/* Output:
Rat, 1, 0; Manx, 2, 1; Cymric, 3, 2; Mutt, 4, 3; Pug,
5, 4; Cymric, 6, 5; Pug, 7, 6; Manx, 8, 7;
7 6 5 4 3 2 1 0
[Rat, Manx, Cymric, Mutt, Pug, Cymric, Pug, Manx]
[Rat, Manx, Cymric, Cymric, Rat, EgyptianMau, Hamster,
EgyptianMau]
*/
```

`Pets.get()` 方法用來從位置 3 開始取代 **List** 中的所有 Pet 物件。

<!-- LinkedList -->

## 鍊表LinkedList

**LinkedList** 也像 **ArrayList** 一樣實現了基本的 **List** 介面，但它在 **List** 中間執行插入和刪除操作時比 **ArrayList** 更高效。然而,它在隨機訪問操作效率方面卻要遜色一些。

**LinkedList 還添加了一些方法，使其可以被用作堆疊、佇列或雙端佇列（deque）** 。在這些方法中，有些彼此之間可能只是名稱有些差異，或者只存在些許差異，以使得這些名字在特定用法的上下文環境中更加適用（特別是在 **Queue** 中）。例如：

- `getFirst()` 和 `element()` 是相同的，它們都返回列表的頭部（第一個元素）而並不刪除它，如果 **List** 為空，則拋出 **NoSuchElementException** 異常。 `peek()` 方法與這兩個方法只是稍有差異，它在列表為空時返回 **null** 。
- `removeFirst()` 和 `remove()` 也是相同的，它們刪除並返回列表的頭部元素，並在列表為空時拋出 **NoSuchElementException** 異常。 `poll()` 稍有差異，它在列表為空時返回 **null** 。
- `addFirst()` 在列表的開頭插入一個元素。
- `offer()` 與 `add()` 和 `addLast()` 相同。 它們都在列表的尾部（末尾）添加一個元素。
- `removeLast()` 刪除並返回列表的最後一個元素。

下面的範例展示了這些功能之間基本的相似性和差異性。它並不是重複執行 **ListFeatures.java** 中所示的行為：

```java
// collections/LinkedListFeatures.java
import typeinfo.pets.*;
import java.util.*;

public class LinkedListFeatures {
  public static void main(String[] args) {
    LinkedList<Pet> pets =
      new LinkedList<>(Pets.list(5));
    System.out.println(pets);
    // Identical:
    System.out.println(
      "pets.getFirst(): " + pets.getFirst());
    System.out.println(
      "pets.element(): " + pets.element());
    // Only differs in empty-list behavior:
    System.out.println("pets.peek(): " + pets.peek());
    // Identical; remove and return the first element:
    System.out.println(
      "pets.remove(): " + pets.remove());
    System.out.println(
      "pets.removeFirst(): " + pets.removeFirst());
    // Only differs in empty-list behavior:
    System.out.println("pets.poll(): " + pets.poll());
    System.out.println(pets);
    pets.addFirst(new Rat());
    System.out.println("After addFirst(): " + pets);
    pets.offer(Pets.get());
    System.out.println("After offer(): " + pets);
    pets.add(Pets.get());
    System.out.println("After add(): " + pets);
    pets.addLast(new Hamster());
    System.out.println("After addLast(): " + pets);
    System.out.println(
      "pets.removeLast(): " + pets.removeLast());
  }
}
/* Output:
[Rat, Manx, Cymric, Mutt, Pug]
pets.getFirst(): Rat
pets.element(): Rat
pets.peek(): Rat
pets.remove(): Rat
pets.removeFirst(): Manx
pets.poll(): Cymric
[Mutt, Pug]
After addFirst(): [Rat, Mutt, Pug]
After offer(): [Rat, Mutt, Pug, Cymric]
After add(): [Rat, Mutt, Pug, Cymric, Pug]
After addLast(): [Rat, Mutt, Pug, Cymric, Pug, Hamster]
pets.removeLast(): Hamster
*/
```

`Pets.list()` 的結果被傳遞給 **LinkedList** 的構造器，以便使用它來填充 **LinkedList** 。如果查看 **Queue** 介面就會發現，它在 **LinkedList** 的基礎上添加了 `element()` ， `offer()` ， `peek()` ， `poll()` 和 `remove()` 方法，以使其可以成為一個 **Queue** 的實現。 **Queue** 的完整範例將在本章稍後給出。

<!-- Stack -->

## 堆疊Stack

堆疊是“後進先出”（LIFO）集合。它有時被稱為*疊加堆疊*（pushdown stack），因為最後“壓入”（push）堆疊的元素，第一個被“彈出”（pop）堆疊。經常用來類比堆疊的事物是帶有彈簧支架的自助餐廳工具列。最後裝入的工具列總是最先拿出來使用的。

Java 1.0 中附帶了一個 **Stack** 類，結果設計得很糟糕（為了向後相容，我們被迫一直忍受 Java 中的舊設計錯誤）。Java 6 添加了 **ArrayDeque** ，其中包含直接實現堆疊功能的方法：

```java
// collections/StackTest.java
import java.util.*;

public class StackTest {
  public static void main(String[] args) {
    Deque<String> stack = new ArrayDeque<>();
    for(String s : "My dog has fleas".split(" "))
      stack.push(s);
    while(!stack.isEmpty())
      System.out.print(stack.pop() + " ");
  }
}
/* Output:
fleas has dog My
*/
```

即使它是作為一個堆疊在使用，我們仍然必須將其聲明為 **Deque** 。有時一個名為 **Stack** 的類更能把事情講清楚：

```java
// onjava/Stack.java
// A Stack class built with an ArrayDeque
package onjava;
import java.util.Deque;
import java.util.ArrayDeque;

public class Stack<T> {
  private Deque<T> storage = new ArrayDeque<>();
  public void push(T v) { storage.push(v); }
  public T peek() { return storage.peek(); }
  public T pop() { return storage.pop(); }
  public boolean isEmpty() { return storage.isEmpty(); }
  @Override
  public String toString() {
    return storage.toString();
  }
}
```

這裡引入了使用泛型的類定義的最簡單的可能範例。類名稱後面的 **\<T\>** 告訴編譯器這是一個參數化類型，而其中的類型參數 **T** 會在使用類時被實際類型取代。基本上，這個類是在聲明“我們在定義一個可以持有 **T** 類型物件的 **Stack** 。” **Stack** 是使用 **ArrayDeque** 實現的，而 **ArrayDeque** 也被告知它將持有 **T** 類型物件。注意， `push()` 接受類型為 **T** 的物件，而 `peek()` 和 `pop()` 返回類型為 **T** 的物件。 `peek()` 方法將返回堆疊頂元素，但並不將其從堆疊頂刪除，而 `pop()` 刪除並返回頂部元素。

如果只需要堆疊的行為，那麼使用繼承是不合適的，因為這將產生一個具有 **ArrayDeque** 的其它所有方法的類（在[附錄：集合主題]()中將會看到， **Java 1.0** 設計者在建立 **java.util.Stack** 時，就犯了這個錯誤）。使用組合，可以選擇要公開的方法以及如何命名它們。

下面將使用 **StackTest.java** 中的相同程式碼來示範這個新的 **Stack** 類：

```java
// collections/StackTest2.java
import onjava.*;

public class StackTest2 {
  public static void main(String[] args) {
    Stack<String> stack = new Stack<>();
    for(String s : "My dog has fleas".split(" "))
      stack.push(s);
    while(!stack.isEmpty())
      System.out.print(stack.pop() + " ");
  }
}
/* Output:
fleas has dog My
*/
```

如果想在自己的程式碼中使用這個 **Stack** 類，當在建立其實例時，就需要完整指定包名，或者更改這個類的名稱；否則，就有可能會與 **java.util** 包中的 **Stack** 發生衝突。例如，如果我們在上面的例子中匯入 **java.util.***，那麼就必須使用包名來防止衝突：

```java
// collections/StackCollision.java

public class StackCollision {
  public static void main(String[] args) {
    onjava.Stack<String> stack = new onjava.Stack<>();
    for(String s : "My dog has fleas".split(" "))
      stack.push(s);
    while(!stack.isEmpty())
      System.out.print(stack.pop() + " ");
    System.out.println();
    java.util.Stack<String> stack2 =
      new java.util.Stack<>();
    for(String s : "My dog has fleas".split(" "))
      stack2.push(s);
    while(!stack2.empty())
      System.out.print(stack2.pop() + " ");
  }
}
/* Output:
fleas has dog My
fleas has dog My
*/
```

儘管已經有了 **java.util.Stack** ，但是 **ArrayDeque** 可以產生更好的 **Stack** ，因此更可取。

還可以使用顯式匯入來控制對“首選” **Stack** 實現的選擇：
```java
import onjava.Stack;
```

現在,任何對 **Stack** 的引用都將選擇 **onjava** 版本，而在選擇 **java.util.Stack** 時，必須使用全限定名稱（full qualification）。

<!-- Set -->
## 集合Set

**Set** 不儲存重複的元素。 如果試圖將相同物件的多個實例添加到 **Set** 中，那麼它會阻止這種重複行為。  **Set** 最常見的用途是測試歸屬性，可以很輕鬆地詢問某個物件是否在一個 **Set** 中。因此，尋找通常是 **Set** 最重要的操作，因此通常會選擇 **HashSet** 實現，該實現針對快速尋找進行了最佳化。

**Set** 具有與 **Collection** 相同的介面，因此沒有任何額外的功能，不像前面兩種不同類型的 **List** 那樣。實際上， **Set** 就是一個 **Collection**  ，只是行為不同。（這是繼承和多態思想的典型應用：表現不同的行為。）**Set** 根據物件的“值”確定歸屬性，更複雜的內容將在[附錄：集合主題]()中介紹。

下面是使用存放 **Integer** 物件的 **HashSet** 的範例：

```java
// collections/SetOfInteger.java
import java.util.*;

public class SetOfInteger {
  public static void main(String[] args) {
    Random rand = new Random(47);
    Set<Integer> intset = new HashSet<>();
    for(int i = 0; i < 10000; i++)
      intset.add(rand.nextInt(30));
    System.out.println(intset);
  }
}
/* Output:
[0, 1, 2, 3, 4, 5, 6, 7, 8, 9, 10, 11, 12, 13, 14, 15,
16, 17, 18, 19, 20, 21, 22, 23, 24, 25, 26, 27, 28, 29]
*/
```

在 0 到 29 之間的 10000 個隨機整數被添加到 **Set** 中，因此可以想像每個值都重複了很多次。但是從結果中可以看到，每一個數只有一個實例出現在結果中。

早期 Java 版本中的 **HashSet** 產生的輸出沒有明顯的順序。這是因為出於對速度的追求， **HashSet** 使用了散列，請參閱[附錄：集合主題]()一章。由 **HashSet** 維護的順序與 **TreeSet** 或 **LinkedHashSet** 不同，因為它們的實現具有不同的元素儲存方式。 **TreeSet** 將元素儲存在紅-黑樹資料結構中，而 **HashSet** 使用散列函數。  **LinkedHashSet** 也使用散列來提高查詢速度，但是似乎使用了鍊表來維護元素的插入順序。顯然，散列演算法有改動，以至於現在（上述範例中的HashSet ） **Integer** 是有序的。但是，您不應該依賴此行為（下面例子就沒有排序）：

```java
// collections/SetOfString.java
import java.util.*;

public class SetOfString {
  public static void main(String[] args) {
    Set<String> colors = new HashSet<>();
    for(int i = 0; i < 100; i++) {
      colors.add("Yellow");
      colors.add("Blue");
      colors.add("Red");
      colors.add("Red");
      colors.add("Orange");
      colors.add("Yellow");
      colors.add("Blue");
      colors.add("Purple");
    }
    System.out.println(colors);
  }
}
/* Output:
[Red, Yellow, Blue, Purple, Orange]
*/
```

**String** 物件似乎沒有排序。要對結果進行排序，一種方法是使用 **TreeSet** 而不是 **HashSet** ：

```java
// collections/SortedSetOfString.java
import java.util.*;

public class SortedSetOfString {
  public static void main(String[] args) {
    Set<String> colors = new TreeSet<>();
    for(int i = 0; i < 100; i++) {
      colors.add("Yellow");
      colors.add("Blue");
      colors.add("Red");
      colors.add("Red");
      colors.add("Orange");
      colors.add("Yellow");
      colors.add("Blue");
      colors.add("Purple");
    }
    System.out.println(colors);
  }
}
/* Output:
[Blue, Orange, Purple, Red, Yellow]
*/
```

最常見的操作之一是使用 `contains()` 測試成員歸屬性，但也有一些其它操作，這可能會讓你想起在小學學過的維恩圖（譯者註：利用圖形的交合表示多個集合之間的邏輯關係）：

```java
// collections/SetOperations.java
import java.util.*;

public class SetOperations {
  public static void main(String[] args) {
    Set<String> set1 = new HashSet<>();
    Collections.addAll(set1,
      "A B C D E F G H I J K L".split(" "));
    set1.add("M");
    System.out.println("H: " + set1.contains("H"));
    System.out.println("N: " + set1.contains("N"));
    Set<String> set2 = new HashSet<>();
    Collections.addAll(set2, "H I J K L".split(" "));
    System.out.println(
      "set2 in set1: " + set1.containsAll(set2));
    set1.remove("H");
    System.out.println("set1: " + set1);
    System.out.println(
      "set2 in set1: " + set1.containsAll(set2));
    set1.removeAll(set2);
    System.out.println(
      "set2 removed from set1: " + set1);
    Collections.addAll(set1, "X Y Z".split(" "));
    System.out.println(
      "'X Y Z' added to set1: " + set1);
  }
}
/* Output:
H: true
N: false
set2 in set1: true
set1: [A, B, C, D, E, F, G, I, J, K, L, M]
set2 in set1: false
set2 removed from set1: [A, B, C, D, E, F, G, M]
'X Y Z' added to set1: [A, B, C, D, E, F, G, M, X, Y,
Z]
*/
```

這些方法名都是自解釋的，JDK 文件中還有一些其它的方法。

能夠產生每個元素都唯一的列表是相當有用的功能。例如，假設想要列出上面的 **SetOperations.java** 文件中的所有單詞，透過使用本書後面介紹的 `java.nio.file.Files.readAllLines()` 方法，可以打開一個文件，並將其作為一個 **List\<String>** 讀取，每個 **String** 都是輸入文件中的一行：

```java
// collections/UniqueWords.java
import java.util.*;
import java.nio.file.*;

public class UniqueWords {
  public static void
  main(String[] args) throws Exception {
    List<String> lines = Files.readAllLines(
      Paths.get("SetOperations.java"));
    Set<String> words = new TreeSet<>();
    for(String line : lines)
      for(String word : line.split("\\W+"))
        if(word.trim().length() > 0)
          words.add(word);
    System.out.println(words);
  }
}
/* Output:
[A, B, C, Collections, D, E, F, G, H, HashSet, I, J, K,
L, M, N, Output, Set, SetOperations, String, System, X,
Y, Z, add, addAll, added, args, class, collections,
contains, containsAll, false, from, import, in, java,
main, new, out, println, public, remove, removeAll,
removed, set1, set2, split, static, to, true, util,
void]
*/
```

我們逐步瀏覽文件中的每一行，並使用 `String.split()` 將其分解為單詞，這裡使用正規表示式 **\\\ W +** ，這意味著它會依據一個或多個（即 **+** ）非單詞字母來分割字串（正規表示式將在[字串]()章節介紹）。每個結果單詞都會添加到 **Set words** 中。因為它是 **TreeSet** ，所以對結果進行排序。這裡，排序是按*字典順序*（lexicographically）完成的，因此大寫字母和小寫字母是分開的。如果想按*字母順序*（alphabetically）對其進行排序，可以向  **TreeSet** 構造器傳入 **String.CASE_INSENSITIVE_ORDER** 比較器（比較器是一個建立排序順序的物件）：

```java
// collections/UniqueWordsAlphabetic.java
// Producing an alphabetic listing
import java.util.*;
import java.nio.file.*;

public class UniqueWordsAlphabetic {
  public static void
  main(String[] args) throws Exception {
    List<String> lines = Files.readAllLines(
      Paths.get("SetOperations.java"));
    Set<String> words =
      new TreeSet<>(String.CASE_INSENSITIVE_ORDER);
    for(String line : lines)
      for(String word : line.split("\\W+"))
        if(word.trim().length() > 0)
          words.add(word);
    System.out.println(words);
  }
}
/* Output:
[A, add, addAll, added, args, B, C, class, collections,
contains, containsAll, D, E, F, false, from, G, H,
HashSet, I, import, in, J, java, K, L, M, main, N, new,
out, Output, println, public, remove, removeAll,
removed, Set, set1, set2, SetOperations, split, static,
String, System, to, true, util, void, X, Y, Z]
*/
```

**Comparator** 比較器將在[陣列]()章節詳細介紹。

<!-- Map -->
## 映射Map

將物件映射到其他物件的能力是解決編程問題的有效方法。例如，考慮一個程式，它被用來檢查 Java 的 **Random** 類的隨機性。理想情況下， **Random** 會產生完美的數字分布，但為了測試這一點，則需要生成大量的隨機數，並計算落在各種範圍內的數字個數。  **Map** 可以很容易地解決這個問題。在本例中，鍵是 **Random** 生成的數字，而值是該數字出現的次數：

```java
// collections/Statistics.java
// (c)2017 MindView LLC: see Copyright.txt
// We make no guarantees that this code is fit for any purpose.
// Visit http://OnJava8.com for more book information.
// Simple demonstration of HashMap
import java.util.*;

public class Statistics {
  public static void main(String[] args) {
    Random rand = new Random(47);
    Map<Integer, Integer> m = new HashMap<>();
    for(int i = 0; i < 10000; i++) {
      // Produce a number between 0 and 20:
      int r = rand.nextInt(20);
      Integer freq = m.get(r); // [1]
      m.put(r, freq == null ? 1 : freq + 1);
    }
    System.out.println(m);
  }
}
/* Output:
{0=481, 1=502, 2=489, 3=508, 4=481, 5=503, 6=519,
7=471, 8=468, 9=549, 10=513, 11=531, 12=521, 13=506,
14=477, 15=497, 16=533, 17=509, 18=478, 19=464}
*/
```

- **[1]** 自動包裝機制將隨機生成的 **int** 轉換為可以與 **HashMap** 一起使用的 **Integer** 引用（不能使用基本類型的集合）。如果鍵不在集合中，則 `get()` 返回 **null** （這意味著該數字第一次出現）。否則， `get()` 會為鍵生成與之關聯的 **Integer** 值，然後該值被遞增（自動包裝機制再次簡化了表達式，但實際上確實發生了對 **Integer** 的裝箱和拆箱）。

接下來的範例將使用一個 **String** 描述來尋找 **Pet** 物件。它還展示了透過使用 `containsKey()` 和 `containsValue()` 方法去測試一個 **Map** ，以查看它是否包含某個鍵或某個值：

```java
// collections/PetMap.java
import typeinfo.pets.*;
import java.util.*;

public class PetMap {
  public static void main(String[] args) {
    Map<String, Pet> petMap = new HashMap<>();
    petMap.put("My Cat", new Cat("Molly"));
    petMap.put("My Dog", new Dog("Ginger"));
    petMap.put("My Hamster", new Hamster("Bosco"));
    System.out.println(petMap);
    Pet dog = petMap.get("My Dog");
    System.out.println(dog);
    System.out.println(petMap.containsKey("My Dog"));
    System.out.println(petMap.containsValue(dog));
  }
}
/* Output:
{My Dog=Dog Ginger, My Cat=Cat Molly, My
Hamster=Hamster Bosco}
Dog Ginger
true
true
*/
```

**Map** 與陣列和其他的 **Collection** 一樣，可以輕鬆地擴展到多個維度：只需要建立一個 **Map** ，其值也是 **Map** (這些 **Map** 的值可以是其他集合，甚至是其他的 **Map** )。因此，能夠很容易地將集合組合起來以快速生成強大的資料結構。例如，假設你正在追蹤有多個寵物的人，只需要一個 **Map\<Person, List\<Pet>>** 即可：

```java

// collections/MapOfList.java
// {java collections.MapOfList}
package collections;
import typeinfo.pets.*;
import java.util.*;

public class MapOfList {
  public static final Map<Person, List< ? extends Pet>>
    petPeople = new HashMap<>();
  static {
    petPeople.put(new Person("Dawn"),
      Arrays.asList(
        new Cymric("Molly"),
        new Mutt("Spot")));
    petPeople.put(new Person("Kate"),
      Arrays.asList(new Cat("Shackleton"),
        new Cat("Elsie May"), new Dog("Margrett")));
    petPeople.put(new Person("Marilyn"),
      Arrays.asList(
        new Pug("Louie aka Louis Snorkelstein Dupree"),
        new Cat("Stanford"),
        new Cat("Pinkola")));
    petPeople.put(new Person("Luke"),
      Arrays.asList(
        new Rat("Fuzzy"), new Rat("Fizzy")));
    petPeople.put(new Person("Isaac"),
      Arrays.asList(new Rat("Freckly")));
  }
  public static void main(String[] args) {
    System.out.println("People: " + petPeople.keySet());
    System.out.println("Pets: " + petPeople.values());
    for(Person person : petPeople.keySet()) {
      System.out.println(person + " has:");
      for(Pet pet : petPeople.get(person))
        System.out.println("    " + pet);
    }
  }
}
/* Output:
People: [Person Dawn, Person Kate, Person Isaac, Person
Marilyn, Person Luke]
Pets: [[Cymric Molly, Mutt Spot], [Cat Shackleton, Cat
Elsie May, Dog Margrett], [Rat Freckly], [Pug Louie aka
Louis Snorkelstein Dupree, Cat Stanford, Cat Pinkola],
[Rat Fuzzy, Rat Fizzy]]
Person Dawn has:
    Cymric Molly
    Mutt Spot
Person Kate has:
    Cat Shackleton
    Cat Elsie May
    Dog Margrett
Person Isaac has:
    Rat Freckly
Person Marilyn has:
    Pug Louie aka Louis Snorkelstein Dupree
    Cat Stanford
    Cat Pinkola
Person Luke has:
    Rat Fuzzy
    Rat Fizzy
*/
```

**Map** 可以返回由其鍵組成的 **Set** ，由其值組成的 **Collection** ，或者其鍵值對的 **Set** 。 `keySet()` 方法生成由在 **petPeople** 中的所有鍵組成的 **Set** ，它在 *for-in* 語句中被用來遍歷該 **Map** 。

<!-- Queue -->

## 佇列Queue

佇列是一個典型的“先進先出”（FIFO）集合。 即從集合的一端放入事物，再從另一端去獲取它們，事物放入集合的順序和被取出的順序是相同的。佇列通常被當做一種可靠的將物件從程式的某個區域傳輸到另一個區域的途徑。佇列在[並發編程]()中尤為重要，因為它們可以安全地將物件從一個任務傳輸到另一個任務。

**LinkedList** 實現了 **Queue** 介面，並且提供了一些方法以支援佇列行為，因此 **LinkedList** 可以用作 **Queue** 的一種實現。 透過將 **LinkedList** 向上轉換為 **Queue** ，下面的範例使用了在 **Queue** 介面中的 **Queue** 特有(Queue-specific)方法：

```java
// collections/QueueDemo.java
// Upcasting to a Queue from a LinkedList
import java.util.*;

public class QueueDemo {
  public static void printQ(Queue queue) {
    while(queue.peek() != null)
      System.out.print(queue.remove() + " ");
    System.out.println();
  }
  public static void main(String[] args) {
    Queue<Integer> queue = new LinkedList<>();
    Random rand = new Random(47);
    for(int i = 0; i < 10; i++)
      queue.offer(rand.nextInt(i + 10));
    printQ(queue);
    Queue<Character> qc = new LinkedList<>();
    for(char c : "Brontosaurus".toCharArray())
      qc.offer(c);
    printQ(qc);
  }
}
/* Output:
8 1 1 1 5 14 3 1 0 1
B r o n t o s a u r u s
*/
```

`offer()` 是 **Queue** 的特有方法之一，它在允許的情況下，在佇列的尾部插入一個元素，或者返回 **false** 。 `peek()` 和 `element()` 都返回隊頭元素而不刪除它，但如果佇列為空，則 `peek()` 返回 **null** ， 而 `element()` 拋出 **NoSuchElementException** 。 `poll()` 和 `remove()` 都刪除並返回隊頭元素，但如果佇列為空，則 `poll()` 返回 **null** ，而 `remove()` 拋出 **NoSuchElementException** 。

自動包裝機制會自動將 `nextInt()` 的 **int** 結果轉換為 **queue** 所需的 **Integer** 物件，並將 **char c** 轉換為 **qc** 所需的 **Character** 物件。 **Queue** 介面窄化了對 **LinkedList** 方法的訪問權限，因此只有適當的方法才能使用，因此能夠訪問到的 **LinkedList** 的方法會變少（這裡實際上可以將 **Queue** 強制轉換回 **LinkedList** ，但至少我們不鼓勵這樣做）。

**Queue** 的特有方法提供了獨立而完整的功能。 換句話說， **Queue** 無需呼叫繼承自 **Collection** 的方法，（只依靠 **Queue** 的特有方法）就有佇列的功能。

<!-- PriorityQueue -->
### 優先度佇列PriorityQueue

先進先出（FIFO）描述了最典型的*佇列規則*（queuing discipline）。佇列規則是指在給定佇列中的一組元素的情況下，確定下一個彈出佇列的元素的規則。先進先出：下一個彈出的元素應該是等待時間最長的那一個。

*優先度佇列* ：下一個彈出的元素是最需要的元素（具有最高的優先度）。例如，在機場，飛機即將起飛的、尚未登機的乘客可能會被拉出隊伍（作最優先的處理）。如果構建了一個消息傳遞系統，某些消息比其他消息更重要，應該儘快處理，而不管它們到達時間先後。在Java 5 中添加了 **PriorityQueue** ，以便自動實現這種行為。

當在 **PriorityQueue** 上呼叫 `offer()` 方法來插入一個物件時，該物件會在佇列中被排序。[^5]預設的排序使用佇列中物件的*自然順序*（natural order），但是可以透過提供自己的 **Comparator** 來修改這個順序。 **PriorityQueue** 確保在呼叫 `peek()` ， `poll()` 或 `remove()` 方法時，獲得的元素將是佇列中優先度最高的元素。

讓 **PriorityQueue** 與 **Integer** ， **String** 和 **Character** 這樣的內建類型一起工作易如反掌。在下面的範例中，第一組值與前一個範例中的隨機值相同，可以看到它們從 **PriorityQueue** 中彈出的順序與前一個範例不同：

```java
// collections/PriorityQueueDemo.java
import java.util.*;

public class PriorityQueueDemo {
  public static void main(String[] args) {
    PriorityQueue<Integer> priorityQueue =
      new PriorityQueue<>();
    Random rand = new Random(47);
    for(int i = 0; i < 10; i++)
      priorityQueue.offer(rand.nextInt(i + 10));
    QueueDemo.printQ(priorityQueue);

    List<Integer> ints = Arrays.asList(25, 22, 20,
      18, 14, 9, 3, 1, 1, 2, 3, 9, 14, 18, 21, 23, 25);
    priorityQueue = new PriorityQueue<>(ints);
    QueueDemo.printQ(priorityQueue);
    priorityQueue = new PriorityQueue<>(
        ints.size(), Collections.reverseOrder());
    priorityQueue.addAll(ints);
    QueueDemo.printQ(priorityQueue);

    String fact = "EDUCATION SHOULD ESCHEW OBFUSCATION";
    List<String> strings =
      Arrays.asList(fact.split(""));
    PriorityQueue<String> stringPQ =
      new PriorityQueue<>(strings);
    QueueDemo.printQ(stringPQ);
    stringPQ = new PriorityQueue<>(
      strings.size(), Collections.reverseOrder());
    stringPQ.addAll(strings);
    QueueDemo.printQ(stringPQ);

    Set<Character> charSet = new HashSet<>();
    for(char c : fact.toCharArray())
      charSet.add(c); // Autoboxing
    PriorityQueue<Character> characterPQ =
      new PriorityQueue<>(charSet);
    QueueDemo.printQ(characterPQ);
  }
}
/* Output:
0 1 1 1 1 1 3 5 8 14
1 1 2 3 3 9 9 14 14 18 18 20 21 22 23 25 25
25 25 23 22 21 20 18 18 14 14 9 9 3 3 2 1 1
      A A B C C C D D E E E F H H I I L N N O O O O S S
S T T U U U W
W U U U T T S S S O O O O N N L I I H H F E E E D D C C
C B A A
  A B C D E F H I L N O S T U W
*/
```

**PriorityQueue** 是允許重複的，最小的值具有最高的優先度（如果是 **String** ，空格也可以算作值，並且比字母的優先度高）。為了展示如何透過提供自己的 **Comparator** 物件來改變順序，第三個對 **PriorityQueue\<Integer>** 構造器的呼叫，和第二個對 **PriorityQueue\<String>** 的呼叫使用了由 `Collections.reverseOrder()` （Java 5 中新添加的）產生的反序的 **Comparator** 。

最後一部分添加了一個 **HashSet** 來消除重複的 **Character**。

**Integer** ， **String** 和 **Character** 可以與 **PriorityQueue** 一起使用，因為這些類已經內建了自然排序。如果想在 **PriorityQueue** 中使用自己的類，則必須包含額外的功能以產生自然排序，或者必須提供自己的 **Comparator** 。在[附錄：集合主題]()中有一個更複雜的範例來示範這種情況。

<!-- Collection vs. Iterator -->
## 集合與疊代器

**Collection** 是所有序列集合共有的根介面。它可能會被認為是一種“附屬介面”（incidental interface），即因為要表示其他若干個介面的共性而出現的介面。此外，**java.util.AbstractCollection** 類提供了 **Collection** 的預設實現，你可以建立 **AbstractCollection** 的子類型來避免不必要的程式碼重複。

使用介面的一個理由是它可以使我們建立更通用的程式碼。透過針對介面而非具體實現來編寫程式碼，我們的程式碼可以應用於更多類型的物件。[^6]因此，如果所編寫的方法接受一個 **Collection** ，那麼該方法可以應用於任何實現了 **Collection** 的類——這也就使得一個新類可以選擇去實現 **Collection** 介面，以便該方法可以使用它。標準 C++ 類庫中的集合並沒有共同的基類——集合之間的所有共性都是透過疊代器實現的。在 Java 中，遵循 C++ 的方式看起來似乎很明智，即用疊代器而不是 **Collection** 來表示集合之間的共性。但是，這兩種方法綁定在了一起，因為實現 **Collection** 就意味著需要提供 `iterator()` 方法：

```java
// collections/InterfaceVsIterator.java
import typeinfo.pets.*;
import java.util.*;

public class InterfaceVsIterator {
  public static void display(Iterator<Pet> it) {
    while(it.hasNext()) {
      Pet p = it.next();
      System.out.print(p.id() + ":" + p + " ");
    }
    System.out.println();
  }
  public static void display(Collection<Pet> pets) {
    for(Pet p : pets)
      System.out.print(p.id() + ":" + p + " ");
    System.out.println();
  }
  public static void main(String[] args) {
    List<Pet> petList = Pets.list(8);
    Set<Pet> petSet = new HashSet<>(petList);
    Map<String, Pet> petMap = new LinkedHashMap<>();
    String[] names = ("Ralph, Eric, Robin, Lacey, " +
      "Britney, Sam, Spot, Fluffy").split(", ");
    for(int i = 0; i < names.length; i++)
      petMap.put(names[i], petList.get(i));
    display(petList);
    display(petSet);
    display(petList.iterator());
    display(petSet.iterator());
    System.out.println(petMap);
    System.out.println(petMap.keySet());
    display(petMap.values());
    display(petMap.values().iterator());
  }
}
/* Output:
0:Rat 1:Manx 2:Cymric 3:Mutt 4:Pug 5:Cymric 6:Pug
7:Manx
0:Rat 1:Manx 2:Cymric 3:Mutt 4:Pug 5:Cymric 6:Pug
7:Manx
0:Rat 1:Manx 2:Cymric 3:Mutt 4:Pug 5:Cymric 6:Pug
7:Manx
0:Rat 1:Manx 2:Cymric 3:Mutt 4:Pug 5:Cymric 6:Pug
7:Manx
{Ralph=Rat, Eric=Manx, Robin=Cymric, Lacey=Mutt,
Britney=Pug, Sam=Cymric, Spot=Pug, Fluffy=Manx}
[Ralph, Eric, Robin, Lacey, Britney, Sam, Spot, Fluffy]
0:Rat 1:Manx 2:Cymric 3:Mutt 4:Pug 5:Cymric 6:Pug
7:Manx
0:Rat 1:Manx 2:Cymric 3:Mutt 4:Pug 5:Cymric 6:Pug
7:Manx
*/
```

兩個版本的 `display()` 方法都可以使用 **Map** 或 **Collection** 的子類型來工作。 而且**Collection** 介面和 **Iterator** 都將 `display()` 方法與低層集合的特定實現解耦。

在本例中，這兩種方式都可以奏效。事實上， **Collection** 要更方便一點，因為它是 **Iterable** 類型，因此在 `display(Collection)` 的實現中可以使用 *for-in* 構造，這使得程式碼更加清晰。

當需要實現一個不是 **Collection** 的外部類時，由於讓它去實現 **Collection** 介面可能非常困難或麻煩，因此使用 **Iterator** 就會變得非常吸引人。例如，如果我們透過繼承一個持有 **Pet** 物件的類來建立一個 **Collection** 的實現，那麼我們必須實現 **Collection** 所有的方法，縱使我們不會在 `display()` 方法中使用它們，也要這樣做。儘管透過繼承 **AbstractCollection** 會容易些，但是 **AbstractCollection** 還有 `iterator()` 和 `size()`沒有實現（抽象方法），而 **AbstractCollection** 中的其它方法會用到它們，因此必須以自己的方式實現這兩個方法：

```java
// collections/CollectionSequence.java
import typeinfo.pets.*;
import java.util.*;

public class CollectionSequence
extends AbstractCollection<Pet> {
  private Pet[] pets = Pets.array(8);
  @Override
  public int size() { return pets.length; }
  @Override
  public Iterator<Pet> iterator() {
    return new Iterator<Pet>() { // [1]
      private int index = 0;
      @Override
      public boolean hasNext() {
        return index < pets.length;
      }
      @Override
      public Pet next() { return pets[index++]; }
      @Override
      public void remove() { // Not implemented
        throw new UnsupportedOperationException();
      }
    };
  }
  public static void main(String[] args) {
    CollectionSequence c = new CollectionSequence();
    InterfaceVsIterator.display(c);
    InterfaceVsIterator.display(c.iterator());
  }
}
/* Output:
0:Rat 1:Manx 2:Cymric 3:Mutt 4:Pug 5:Cymric 6:Pug
7:Manx
0:Rat 1:Manx 2:Cymric 3:Mutt 4:Pug 5:Cymric 6:Pug
7:Manx
*/
```

`remove()` 方法是一個“可選操作”，在[附錄：集合主題]()中詳細介紹。 這裡可以不必實現它，如果你呼叫它，它將拋出異常。

- **[1]** 你可能會認為，因為 `iterator()` 返回 **Iterator\<Pet>** ，匿名內部類定義可以使用菱形語法，Java可以推斷出類型。但這不起作用，類型推斷仍然非常有限。

這個例子表明，如果實現了 **Collection** ，就必須也實現 `iterator()` ，而單獨只實現 `iterator()` 和繼承 **AbstractCollection** 相比，並沒有容易多少。但是，如果類已經繼承了其他的類，那麼就沒辦法再繼承 **AbstractCollection** 了。在這種情況下，要實現 **Collection** ，就必須實現該介面中的所有方法。此時，繼承並提供建立疊代器的能力（單獨只實現 `iterator()`）要容易得多：

```java
// collections/NonCollectionSequence.java
import typeinfo.pets.*;
import java.util.*;

class PetSequence {
  protected Pet[] pets = Pets.array(8);
}

public class NonCollectionSequence extends PetSequence {
  public Iterator<Pet> iterator() {
    return new Iterator<Pet>() {
      private int index = 0;
      @Override
      public boolean hasNext() {
        return index < pets.length;
      }
      @Override
      public Pet next() { return pets[index++]; }
      @Override
      public void remove() { // Not implemented
        throw new UnsupportedOperationException();
      }
    };
  }
  public static void main(String[] args) {
    NonCollectionSequence nc =
      new NonCollectionSequence();
    InterfaceVsIterator.display(nc.iterator());
  }
}
/* Output:
0:Rat 1:Manx 2:Cymric 3:Mutt 4:Pug 5:Cymric 6:Pug
7:Manx
*/
```

生成 **Iterator** 是將序列與消費該序列的方法連接在一起的耦合度最小的方式，並且與實現 **Collection** 相比，它在序列類上所施加的約束也少得多。

<!-- for-in and Iterators -->
## for-in和疊代器

到目前為止，*for-in* 語法主要用於陣列，但它也適用於任何 **Collection** 物件。實際上在使用 **ArrayList** 時，已經看到了一些使用它的範例，下面是它的通用性的證明：

```java
// collections/ForInCollections.java
// All collections work with for-in
import java.util.*;

public class ForInCollections {
  public static void main(String[] args) {
    Collection<String> cs = new LinkedList<>();
    Collections.addAll(cs,
      "Take the long way home".split(" "));
    for(String s : cs)
      System.out.print("'" + s + "' ");
  }
}
/* Output:
'Take' 'the' 'long' 'way' 'home'
*/
```

由於 **cs** 是一個 **Collection** ，因此該程式碼展示了使用 *for-in* 是所有 **Collection** 物件的特徵。

這樣做的原因是 Java 5 引入了一個名為 **Iterable** 的介面，該介面包含一個能夠生成 **Iterator** 的 `iterator()` 方法。*for-in* 使用此 **Iterable** 介面來遍歷序列。因此，如果建立了任何實現了 **Iterable** 的類，都可以將它用於 *for-in* 語句中：

```java
// collections/IterableClass.java
// Anything Iterable works with for-in
import java.util.*;

public class IterableClass implements Iterable<String> {
  protected String[] words = ("And that is how " +
    "we know the Earth to be banana-shaped."
    ).split(" ");
  @Override
  public Iterator<String> iterator() {
    return new Iterator<String>() {
      private int index = 0;
      @Override
      public boolean hasNext() {
        return index < words.length;
      }
      @Override
      public String next() { return words[index++]; }
      @Override
      public void remove() { // Not implemented
        throw new UnsupportedOperationException();
      }
    };
  }
  public static void main(String[] args) {
    for(String s : new IterableClass())
      System.out.print(s + " ");
  }
}
/* Output:
And that is how we know the Earth to be banana-shaped.
*/
```

`iterator()` 返回的是實現了 **Iterator\<String>** 的匿名內部類的實例，該匿名內部類可以遍歷陣列中的每個單詞。在主方法中，可以看到 **IterableClass** 確實可以用於 *for-in* 語句。

在 Java 5 中，許多類都是 **Iterable** ，主要包括所有的 **Collection** 類（但不包括各種 **Maps** ）。 例如，下面的程式碼可以顯示所有的作業系統環境變數：

```java
// collections/EnvironmentVariables.java
// {VisuallyInspectOutput}
import java.util.*;

public class EnvironmentVariables {
  public static void main(String[] args) {
    for(Map.Entry entry: System.getenv().entrySet()) {
      System.out.println(entry.getKey() + ": " +
        entry.getValue());
    }
  }
}
```

`System.getenv()` [^7]返回一個 **Map** ， `entrySet()` 產生一個由 **Map.Entry** 的元素構成的 **Set** ，並且這個 **Set** 是一個 **Iterable** ，因此它可以用於 *for-in* 循環。

*for-in* 語句適用於陣列或者其它任何 **Iterable** ，但這並不代表陣列一定是 **Iterable** ，也不會發生任何自動裝箱：

```java
// collections/ArrayIsNotIterable.java
import java.util.*;

public class ArrayIsNotIterable {
  static <T> void test(Iterable<T> ib) {
    for(T t : ib)
      System.out.print(t + " ");
  }
  public static void main(String[] args) {
    test(Arrays.asList(1, 2, 3));
    String[] strings = { "A", "B", "C" };
    // An array works in for-in, but it's not Iterable:
    //- test(strings);
    // You must explicitly convert it to an Iterable:
    test(Arrays.asList(strings));
  }
}
/* Output:
1 2 3 A B C
*/
```

嘗試將陣列作為一個 **Iterable** 參數傳遞會導致失敗。這說明不存在任何從陣列到 **Iterable** 的自動轉換; 必須手工執行這種轉換。

<!-- The Adapter Method Idiom -->
### 適配器方法慣用法

如果現在有一個 **Iterable** 類，你想要添加一種或多種在 *for-in* 語句中使用這個類的方法，應該怎麼做呢？例如，你希望可以選擇正向還是反向遍歷一個單詞列表。如果直接繼承這個類，並重寫 `iterator()` 方法，則只能取代現有的方法，而不能實現遍歷順序的選擇。

一種解決方案是所謂*適配器方法*（Adapter Method）的慣用法。“適配器”部分來自於設計模式，因為必須要提供特定的介面來滿足 *for-in* 語句。如果已經有一個介面並且需要另一個介面時，則編寫適配器就可以解決這個問題。
在這裡，若希望在預設的正向疊代器的基礎上，添加產生反向疊代器的能力，因此不能使用重寫，相反，而是添加了一個能夠生成 **Iterable** 物件的方法，該物件可以用於 *for-in* 語句。這使得我們可以提供多種使用 *for-in* 語句的方式：

```java
// collections/AdapterMethodIdiom.java
// The "Adapter Method" idiom uses for-in
// with additional kinds of Iterables
import java.util.*;

class ReversibleArrayList<T> extends ArrayList<T> {
  ReversibleArrayList(Collection<T> c) {
    super(c);
  }
  public Iterable<T> reversed() {
    return new Iterable<T>() {
      public Iterator<T> iterator() {
        return new Iterator<T>() {
          int current = size() - 1;
          public boolean hasNext() {
            return current > -1;
          }
          public T next() { return get(current--); }
          public void remove() { // Not implemented
            throw new UnsupportedOperationException();
          }
        };
      }
    };
  }
}

public class AdapterMethodIdiom {
  public static void main(String[] args) {
    ReversibleArrayList<String> ral =
      new ReversibleArrayList<String>(
        Arrays.asList("To be or not to be".split(" ")));
    // Grabs the ordinary iterator via iterator():
    for(String s : ral)
      System.out.print(s + " ");
    System.out.println();
    // Hand it the Iterable of your choice
    for(String s : ral.reversed())
      System.out.print(s + " ");
  }
}
/* Output:
To be or not to be
be to not or be To
*/
```

在主方法中，如果直接將 **ral** 物件放在 *for-in* 語句中，則會得到（預設的）正向疊代器。但是如果在該物件上呼叫 `reversed()` 方法，它會產生不同的行為。

透過使用這種方式，可以在 **IterableClass.java** 範例中添加兩種適配器方法：

```java
// collections/MultiIterableClass.java
// Adding several Adapter Methods
import java.util.*;

public class MultiIterableClass extends IterableClass {
  public Iterable<String> reversed() {
    return new Iterable<String>() {
      public Iterator<String> iterator() {
        return new Iterator<String>() {
          int current = words.length - 1;
          public boolean hasNext() {
            return current > -1;
          }
          public String next() {
            return words[current--];
          }
          public void remove() { // Not implemented
            throw new UnsupportedOperationException();
          }
        };
      }
    };
  }
  public Iterable<String> randomized() {
    return new Iterable<String>() {
      public Iterator<String> iterator() {
        List<String> shuffled =
          new ArrayList<String>(Arrays.asList(words));
        Collections.shuffle(shuffled, new Random(47));
        return shuffled.iterator();
      }
    };
  }
  public static void main(String[] args) {
    MultiIterableClass mic = new MultiIterableClass();
    for(String s : mic.reversed())
      System.out.print(s + " ");
    System.out.println();
    for(String s : mic.randomized())
      System.out.print(s + " ");
    System.out.println();
    for(String s : mic)
      System.out.print(s + " ");
  }
}
/* Output:
banana-shaped. be to Earth the know we how is that And
is banana-shaped. Earth that how the be And we know to
And that is how we know the Earth to be banana-shaped.
*/
```

注意，第二個方法 `random()` 沒有建立它自己的 **Iterator** ，而是直接返回被打亂的 **List** 中的 **Iterator** 。

從輸出中可以看到， `Collections.shuffle()` 方法不會影響到原始陣列，而只是打亂了 **shuffled** 中的引用。之所以這樣，是因為 `randomized()` 方法用一個 **ArrayList** 將 `Arrays.asList()` 的結果包裝了起來。如果這個由 `Arrays.asList()` 生成的 **List** 被直接打亂，那麼它將修改底層陣列，如下所示：

```java
// collections/ModifyingArraysAsList.java
import java.util.*;

public class ModifyingArraysAsList {
  public static void main(String[] args) {
    Random rand = new Random(47);
    Integer[] ia = { 1, 2, 3, 4, 5, 6, 7, 8, 9, 10 };
    List<Integer> list1 =
      new ArrayList<>(Arrays.asList(ia));
    System.out.println("Before shuffling: " + list1);
    Collections.shuffle(list1, rand);
    System.out.println("After shuffling: " + list1);
    System.out.println("array: " + Arrays.toString(ia));

    List<Integer> list2 = Arrays.asList(ia);
    System.out.println("Before shuffling: " + list2);
    Collections.shuffle(list2, rand);
    System.out.println("After shuffling: " + list2);
    System.out.println("array: " + Arrays.toString(ia));
  }
}
/* Output:
Before shuffling: [1, 2, 3, 4, 5, 6, 7, 8, 9, 10]
After shuffling: [4, 6, 3, 1, 8, 7, 2, 5, 10, 9]
array: [1, 2, 3, 4, 5, 6, 7, 8, 9, 10]
Before shuffling: [1, 2, 3, 4, 5, 6, 7, 8, 9, 10]
After shuffling: [9, 1, 6, 3, 7, 2, 5, 10, 4, 8]
array: [9, 1, 6, 3, 7, 2, 5, 10, 4, 8]
*/
```

在第一種情況下， `Arrays.asList()` 的輸出被傳遞給了 **ArrayList** 的構造器，這將建立一個引用 **ia** 的元素的 **ArrayList** ，因此打亂這些引用不會修改該陣列。但是，如果直接使用 `Arrays.asList(ia)` 的結果，這種打亂就會修改 **ia** 的順序。重要的是要注意 `Arrays.asList()` 生成一個 **List** 物件，該物件使用底層陣列作為其物理實現。如果對 **List** 物件做了任何修改，又不想讓原始陣列被修改，那麼就應該在另一個集合中建立一個副本。

<!-- Summary -->
## 本章小結

Java 提供了許多儲存物件的方法：

1. 陣列將數字索引與物件相關聯。它儲存類型明確的物件，因此在尋找物件時不必對結果做類型轉換。它可以是多維的，可以儲存基本類型的資料。雖然可以在執行時建立陣列，但是一旦建立陣列，就無法更改陣列的大小。

2. **Collection** 儲存單一的元素，而 **Map** 包含相關聯的鍵值對。使用 Java 泛型，可以指定集合中儲存的物件的類型，因此不能將錯誤類型的物件放入集合中，並且在從集合中獲取元素時，不必進行類型轉換。各種 **Collection** 和各種 **Map** 都可以在你向其中添加更多的元素時，自動調整其尺寸大小。集合不能儲存基本類型，但自動裝箱機制會負責執行基本類型和集合中儲存的包裝類型之間的雙向轉換。

3. 像陣列一樣， **List** 也將數字索引與物件相關聯，因此，陣列和 **List** 都是有序集合。

4. 如果要執行大量的隨機訪問，則使用 **ArrayList** ，如果要經常從表中間插入或刪除元素，則應該使用 **LinkedList** 。

5. 佇列和堆疊的行為是透過 **LinkedList** 提供的。

6. **Map** 是一種將物件（而非數字）與物件相關聯的設計。 **HashMap** 專為快速訪問而設計，而 **TreeMap** 保持鍵始終處於排序狀態，所以沒有 **HashMap** 快。 **LinkedHashMap** 按插入順序儲存其元素，但使用散列提供快速訪問的能力。

7. **Set** 不接受重複元素。 **HashSet** 提供最快的查詢速度，而 **TreeSet** 保持元素處於排序狀態。 **LinkedHashSet** 按插入順序儲存其元素，但使用散列提供快速訪問的能力。

8. 不要在新程式碼中使用遺留類 **Vector** ，**Hashtable** 和 **Stack** 。

瀏覽一下Java集合的簡圖（不包含抽象類或遺留元件）會很有幫助。這裡僅包括在一般情況下會碰到的介面和類。（譯者註：下圖為原著PDF中的截圖，可能由於未知原因存在問題。這裡可參考譯者繪製版[^8]）

![simple collection taxonomy](../images/simple-collection-taxonomy.png)

### 簡單集合分類

可以看到，實際上只有四個基本的集合元件： **Map** ， **List** ， **Set** 和 **Queue** ，它們各有兩到三個實現版本（**Queue** 的 **java.util.concurrent** 實現未包含在此圖中）。最常使用的集合用黑色粗線線框表示。

虛線框表示介面，實線框表示普通的（具體的）類。帶有空心箭頭的虛線表示特定的類實現了一個介面。實心箭頭表示某個類可以生成箭頭指向的類的物件。例如，任何 **Collection** 都可以生成 **Iterator** ， **List** 可以生成 **ListIterator** （也能生成普通的 **Iterator** ，因為 **List** 繼承自 **Collection** ）。

下面的範例展示了各種不同的類在方法上的差異。實際程式碼來自[泛型]()章節，在這裡只是呼叫它來產生輸出。程式的輸出還展示了在每個類或介面中所實現的介面：

```java
// collections/CollectionDifferences.java
import onjava.*;

public class CollectionDifferences {
  public static void main(String[] args) {
    CollectionMethodDifferences.main(args);
  }
}
/* Output:
Collection: [add, addAll, clear, contains, containsAll,
equals, forEach, hashCode, isEmpty, iterator,
parallelStream, remove, removeAll, removeIf, retainAll,
size, spliterator, stream, toArray]
Interfaces in Collection: [Iterable]
Set extends Collection, adds: []
Interfaces in Set: [Collection]
HashSet extends Set, adds: []
Interfaces in HashSet: [Set, Cloneable, Serializable]
LinkedHashSet extends HashSet, adds: []
Interfaces in LinkedHashSet: [Set, Cloneable,
Serializable]
TreeSet extends Set, adds: [headSet,
descendingIterator, descendingSet, pollLast, subSet,
floor, tailSet, ceiling, last, lower, comparator,
pollFirst, first, higher]
Interfaces in TreeSet: [NavigableSet, Cloneable,
Serializable]
List extends Collection, adds: [replaceAll, get,
indexOf, subList, set, sort, lastIndexOf, listIterator]
Interfaces in List: [Collection]
ArrayList extends List, adds: [trimToSize,
ensureCapacity]
Interfaces in ArrayList: [List, RandomAccess,
Cloneable, Serializable]
LinkedList extends List, adds: [offerFirst, poll,
getLast, offer, getFirst, removeFirst, element,
removeLastOccurrence, peekFirst, peekLast, push,
pollFirst, removeFirstOccurrence, descendingIterator,
pollLast, removeLast, pop, addLast, peek, offerLast,
addFirst]
Interfaces in LinkedList: [List, Deque, Cloneable,
Serializable]
Queue extends Collection, adds: [poll, peek, offer,
element]
Interfaces in Queue: [Collection]
PriorityQueue extends Queue, adds: [comparator]
Interfaces in PriorityQueue: [Serializable]
Map: [clear, compute, computeIfAbsent,
computeIfPresent, containsKey, containsValue, entrySet,
equals, forEach, get, getOrDefault, hashCode, isEmpty,
keySet, merge, put, putAll, putIfAbsent, remove,
replace, replaceAll, size, values]
HashMap extends Map, adds: []
Interfaces in HashMap: [Map, Cloneable, Serializable]
LinkedHashMap extends HashMap, adds: []
Interfaces in LinkedHashMap: [Map]
SortedMap extends Map, adds: [lastKey, subMap,
comparator, firstKey, headMap, tailMap]
Interfaces in SortedMap: [Map]
TreeMap extends Map, adds: [descendingKeySet,
navigableKeySet, higherEntry, higherKey, floorKey,
subMap, ceilingKey, pollLastEntry, firstKey, lowerKey,
headMap, tailMap, lowerEntry, ceilingEntry,
descendingMap, pollFirstEntry, lastKey, firstEntry,
floorEntry, comparator, lastEntry]
Interfaces in TreeMap: [NavigableMap, Cloneable,
Serializable]
*/
```

除 **TreeSet** 之外的所有 **Set** 都具有與 **Collection** 完全相同的介面。**List** 和 **Collection** 存在著明顯的不同，儘管 **List** 所要求的方法都在 **Collection** 中。另一方面，在 **Queue** 介面中的方法是獨立的，在建立具有 **Queue** 功能的實現時，不需要使用 **Collection** 方法。最後， **Map** 和 **Collection** 之間唯一的交集是 **Map** 可以使用 `entrySet()` 和 `values()` 方法來產生 **Collection** 。

請注意，標記介面 **java.util.RandomAccess** 附加到了 **ArrayList** 上，但不附加到 **LinkedList** 上。這為根據特定 **List** 動態改變其行為的演算法提供了訊息。

從物件導向的繼承層次結構來看，這種組織結構確實有些奇怪。但是，當了解了 **java.util** 中更多的有關集合的內容後（特別是在[附錄：集合主題]()中的內容），就會發現除了繼承結構有點奇怪外，還有更多的問題。集合類庫一直以來都是設計難題——解決這些問題涉及到要去滿足經常彼此之間互為牽制的各方面需求。所以要做好準備，在各處做出妥協。

儘管存在這些問題，但 Java 集合仍是在日常工作中使用的基本工具，它可以使程式更簡潔、更強大、更有效。你可能需要一段時間才能熟悉集合類庫的某些方面，但我想你很快就會找到自己的路子，來獲得和使用這個類庫中的類。

[^1]: 許多語言，例如 Perl ，Python 和 Ruby ，都有集合的本機支援。

[^2]: 這裡是操作符重載的用武之地，C++和C#的集合類都使用操作符重載生成了更簡潔的語法。

[^3]: 在[泛型]()章節的末尾，有個關於這個問題是否很嚴重的討論。但是，[泛型]()章節還將展示Java泛型遠不止是類型安全的集合這麼簡單。

[^4]: `remove()` 是一個所謂的“可選”方法（還有其它這樣的方法），這意味著並非所有的 **Iterator** 實現都必須實現該方法。這個問題將在[附錄：集合主題]()中介紹。但是，標準 Java 庫集合實現了 `remove()` ，因此在[附錄：集合主題]()章節之前，都不必擔心這個問題。

[^5]: 這實際上依賴於具體實現。優先度佇列演算法通常會按插入順序排序（維護一個*堆*），但它們也可以在刪除時選擇最重要的元素。 如果一個物件在佇列中等待時，它的優先度會發生變化，那麼演算法的選擇就很重要。

[^6]: 有些人鼓吹不加思索地建立介面對應一個類甚至每個類中的所有可能的方法組合。 但我相信介面的意義不應該僅限於機械地重複方法組合，因此我傾向於看到增加介面的價值才建立它。

[^7]: 這在 Java 5 之前是不可用的，因為該方法被認為與作業系統的耦合度過緊，因此違反“一次編寫，處處執行”的原則。現在卻提供它，這一事實表明， Java 的設計者們更加務實了。

[^8]: 下面是譯者繪製的 Java 集合框架簡圖，黃色為介面，綠色為抽象類，藍色為具體類。虛線箭頭表示實現關係，實線箭頭表示繼承關係。
![collection](../images/collection.png)
![map](../images/map.png)

<!-- 分頁 -->

<div style="page-break-after: always;"></div>
