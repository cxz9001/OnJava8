[TOC]

<!-- Arrays -->

# 第二十一章 陣列


> 在 [初始化和清理](book/06-Housekeeping.md) 一章的最後，你已經學過如何定義和初始化一個陣列。

簡單來看，陣列需要你去建立和初始化，你可以透過下標對陣列元素進行訪問，陣列的大小不會改變。大多數時候你只需要知道這些，但有時候你必須在陣列上進行更複雜的操作，你也可能需要在陣列和更加靈活的 **集合** (Collection)之間做出評估。因此本章我們將對陣列進行更加深入的分析。

**注意：** 隨著 Java Collection 和 Stream 類中進階功能的不斷增加，日常編程中使用陣列的需求也在變少，所以你暫且可以放心地略讀甚至跳過這一章。但是，即使你自己避免使用陣列，也總會有需要閱讀別人陣列程式碼的那一天。那時候，本章依然在這裡等著你來翻閱。

<!-- Why Arrays are Special -->

## 陣列特性

明明還有很多其他的辦法來儲存物件，那麼是什麼令陣列如此特別？

將陣列和其他類型的集合區分開來的原因有三：效率，類型，儲存基本資料類型的能力。在 Java 中，使用陣列儲存和隨機訪問物件引用序列是非常高效的。陣列是簡單的線性序列，這使得對元素的訪問變得非常快。然而這種高速也是有代價的，代價就是陣列物件的大小是固定的，且在該陣列的生存期內不能更改。

速度通常並不是問題，如果有問題，你儲存和檢索物件的方式也很少是罪魁禍首。你應該總是從 **ArrayList** (來自 [集合](book/12-Collections.md ))開始，它將陣列封裝起來。必要時，它會自動分配更多的陣列空間，建立新陣列，並將舊陣列中的引用移動到新陣列。這種靈活性需要開銷，所以一個 **ArrayList** 的效率不如陣列。在極少的情況下效率會成為問題，所以這種時候你可以直接使用陣列。


陣列和集合(Collections)都不能濫用。不管你使用陣列還是集合，如果你越界，你都會得到一個 **RuntimeException** 的異常提醒，這表明你的程式中存在錯誤。


在泛型前，其他的集合類以一種寬泛的方式處理物件（就好像它們沒有特定類型一樣）。事實上，這些集合類把儲存物件的類型預設為 **Object**，也就是 Java 中所有類的基類。而陣列是優於 **預泛型** (pre-generic)集合類的，因為你建立一個陣列就可以儲存特定類型的資料。這意味著你獲得了一個編譯時的類型檢查，而這可以防止你插入錯誤的資料類型，或者搞錯你正在提取的資料類型。


當然，不管在編譯時還是執行時，Java都會阻止你犯向物件發送不正確消息的錯誤。然而不管怎樣，使用陣列都不會有更大的風險。比較好的地方在於，如果編譯器報錯，最終的使用者更容易理解拋出異常的含義。


一個陣列可以儲存基本資料類型，而一個預泛型的集合不可以。然而對於泛型而言，集合可以指定和檢查他們儲存物件的類型，而透過 **自動裝箱** (autoboxing)機制，集合表現地就像它們可以儲存基本資料類型一樣，因為這種轉換是自動的。

下面給出一例用於比較陣列和泛型集合：

```java
// arrays/CollectionComparison.java
// (c)2017 MindView LLC: see Copyright.txt
// We make no guarantees that this code is fit for any purpose.
// Visit http://OnJava8.com for more book information.
import java.util.*;
import onjava.*;
import static onjava.ArrayShow.*;

class BerylliumSphere {
  private static long counter;
  private final long id = counter++;
  @Override
  public String toString() {
    return "Sphere " + id;
  }
}

public class CollectionComparison {
  public static void main(String[] args) {
    BerylliumSphere[] spheres =
      new BerylliumSphere[10];
    for(int i = 0; i < 5; i++)
      spheres[i] = new BerylliumSphere();
    show(spheres);
    System.out.println(spheres[4]);

    List<BerylliumSphere> sphereList = Suppliers.create(
      ArrayList::new, BerylliumSphere::new, 5);
    System.out.println(sphereList);
    System.out.println(sphereList.get(4));

    int[] integers = { 0, 1, 2, 3, 4, 5 };
    show(integers);
    System.out.println(integers[4]);

    List<Integer> intList = new ArrayList<>(
      Arrays.asList(0, 1, 2, 3, 4, 5));
    intList.add(97);
    System.out.println(intList);
    System.out.println(intList.get(4));
  }
}
/* Output:
[Sphere 0, Sphere 1, Sphere 2, Sphere 3, Sphere 4,
null, null, null, null, null]
Sphere 4
[Sphere 5, Sphere 6, Sphere 7, Sphere 8, Sphere 9]
Sphere 9
[0, 1, 2, 3, 4, 5]
4
[0, 1, 2, 3, 4, 5, 97]
4
*/
```

**Suppliers.create()** 方法在[泛型](book/20-Generics.md)一章中被定義。上面兩種儲存物件的方式都是有類型檢查的，唯一比較明顯的區別就是陣列使用 **[ ]** 來隨機存取元素，而一個 **List** 使用諸如 `add()` 和 `get()` 等方法。陣列和 **ArrayList** 之間的相似是設計者有意為之，所以在概念上，兩者很容易切換。但是就像你在[集合](book/12-Collections.md)中看到的，集合的功能明顯多於陣列。隨著 Java 自動裝箱技術的出現，透過集合使用基本資料類型幾乎和透過陣列一樣簡單。陣列唯一剩下的優勢就是效率。然而，當你解決一個更加普遍的問題時，陣列可能限制太多，這種情形下，您可以使用集合類。


### 用於顯示陣列的實用程式

在本章中，我們處處都要顯示陣列。Java 提供了 **Arrays.toString()** 來將陣列轉換為可讀字串，然後可以在控制台上顯示。然而這種方式視覺上噪音太大，所以我們建立一個小的庫來完成這項工作。

```java
// onjava/ArrayShow.java
// (c)2017 MindView LLC: see Copyright.txt
// We make no guarantees that this code is fit for any purpose.
// Visit http://OnJava8.com for more book information.
package onjava;
import java.util.*;

public interface ArrayShow {
  static void show(Object[] a) {
    System.out.println(Arrays.toString(a));
  }
  static void show(boolean[] a) {
    System.out.println(Arrays.toString(a));
  }
  static void show(byte[] a) {
    System.out.println(Arrays.toString(a));
  }
  static void show(char[] a) {
    System.out.println(Arrays.toString(a));
  }
  static void show(short[] a) {
    System.out.println(Arrays.toString(a));
  }
  static void show(int[] a) {
    System.out.println(Arrays.toString(a));
  }
  static void show(long[] a) {
    System.out.println(Arrays.toString(a));
  }
  static void show(float[] a) {
    System.out.println(Arrays.toString(a));
  }
  static void show(double[] a) {
    System.out.println(Arrays.toString(a));
  }
  // Start with a description:
  static void show(String info, Object[] a) {
    System.out.print(info + ": ");
    show(a);
  }
  static void show(String info, boolean[] a) {
    System.out.print(info + ": ");
    show(a);
  }
  static void show(String info, byte[] a) {
    System.out.print(info + ": ");
    show(a);
  }
  static void show(String info, char[] a) {
    System.out.print(info + ": ");
    show(a);
  }
  static void show(String info, short[] a) {
    System.out.print(info + ": ");
    show(a);
  }
  static void show(String info, int[] a) {
    System.out.print(info + ": ");
    show(a);
  }
  static void show(String info, long[] a) {
    System.out.print(info + ": ");
    show(a);
  }
  static void show(String info, float[] a) {
    System.out.print(info + ": ");
    show(a);
  }
  static void show(String info, double[] a) {
    System.out.print(info + ": ");
    show(a);
  }
}
```

第一個方法適用於物件陣列，包括那些包裝基本資料類型的陣列。所有的方法重載對於不同的資料類型是必要的。

第二組重載方法可以讓你顯示帶有訊息 **字串** 前綴的陣列。

為了簡單起見，你通常可以靜態地匯入它們。

<!-- Arrays are First-Class Objects -->

## 一等物件

不管你使用的什麼類型的陣列，陣列中的資料集實際上都是對堆中真正物件的引用。陣列是儲存指向其他物件的引用的物件，陣列可以隱式地建立，作為陣列初始化語法的一部分，也可以顯式地建立，比如使用一個 **new** 表達式。陣列物件的一部分（事實上，你唯一可以使用的方法）就是唯讀的 **length** 成員函數，它能告訴你陣列物件中可以儲存多少元素。**[ ]** 語法是你訪問陣列物件的唯一方式。

下面的例子總結了初始化陣列的多種方式，並且展示了如何給不同的陣列物件分配陣列引用。同時也可以看出物件陣列和基元陣列在使用上是完全相同的。唯一的不同之處就是物件陣列儲存的是物件的引用，而基元陣列則直接儲存基本資料類型的值。

```java
// arrays/ArrayOptions.java
// Initialization & re-assignment of arrays
import java.util.*;
import static onjava.ArrayShow.*;

public class ArrayOptions {
  public static void main(String[] args) {
    // Arrays of objects:
    BerylliumSphere[] a; // Uninitialized local
    BerylliumSphere[] b = new BerylliumSphere[5];

    // The references inside the array are
    // automatically initialized to null:
    show("b", b);
    BerylliumSphere[] c = new BerylliumSphere[4];
    for(int i = 0; i < c.length; i++)
      if(c[i] == null) // Can test for null reference
        c[i] = new BerylliumSphere();

    // Aggregate initialization:
    BerylliumSphere[] d = {
      new BerylliumSphere(),
      new BerylliumSphere(),
      new BerylliumSphere()
    };

    // Dynamic aggregate initialization:
    a = new BerylliumSphere[]{
      new BerylliumSphere(), new BerylliumSphere(),
    };
    // (Trailing comma is optional)

    System.out.println("a.length = " + a.length);
    System.out.println("b.length = " + b.length);
    System.out.println("c.length = " + c.length);
    System.out.println("d.length = " + d.length);
    a = d;
    System.out.println("a.length = " + a.length);

    // Arrays of primitives:
    int[] e; // Null reference
    int[] f = new int[5];

    // The primitives inside the array are
    // automatically initialized to zero:
    show("f", f);
    int[] g = new int[4];
    for(int i = 0; i < g.length; i++)
      g[i] = i*i;
    int[] h = { 11, 47, 93 };

    //  Compile error: variable e not initialized:
    //- System.out.println("e.length = " + e.length);
    System.out.println("f.length = " + f.length);
    System.out.println("g.length = " + g.length);
    System.out.println("h.length = " + h.length);
    e = h;
    System.out.println("e.length = " + e.length);
    e = new int[]{ 1, 2 };
    System.out.println("e.length = " + e.length);
  }
}
/* Output:
b: [null, null, null, null, null]
a.length = 2
b.length = 5
c.length = 4
d.length = 3
a.length = 3
f: [0, 0, 0, 0, 0]
f.length = 5
g.length = 4
h.length = 3
e.length = 3
e.length = 2
*/
```

陣列 **a** 是一個未初始化的本機變數，編譯器不會允許你使用這個引用直到你正確地對其進行初始化。陣列 **b** 被初始化成一系列指向 **BerylliumSphere** 物件的引用，但是並沒有真正的 **BerylliumSphere** 物件被儲存在陣列中。儘管你仍然可以獲得這個陣列的大小，因為 **b** 指向合法物件。這帶來了一個小問題：你無法找出到底有多少元素儲存在陣列中，因為 **length** 只能告訴你陣列可以儲存多少元素；這就是說，陣列物件的大小並不是真正儲存在陣列中物件的個數。然而，當你建立一個陣列物件，其引用將自動初始化為 **null**，因此你可以通過檢查特定陣列元素中的引用是否為 **null** 來判斷其中是否有物件。基元陣列也有類似的機制，比如自動將數值類型初始化為 **0**，char 型初始化為 **(char)0**，布爾類型初始化為 **false**。

陣列 **c** 展示了建立陣列物件後給陣列中各元素分配 **BerylliumSphere** 物件。陣列 **d** 展示了建立陣列物件的聚合初始化語法（隱式地使用 **new** 在堆中建立物件，就像 **c** 一樣）並且初始化成 **BeryliumSphere** 物件，這一切都在一條語句中完成。

下一個陣列初始化可以被看做是一個“動態聚合初始化”。 **d** 使用的聚合初始化必須在 **d** 定義處使用，但是使用第二種語法，你可以在任何地方建立和初始化陣列物件。例如，假設 **hide()** 是一個需要使用一系列的 **BeryliumSphere**物件。你可以這樣呼叫它：

```Java
hide(d);
```

你也可以動態地建立你用作參數傳遞的陣列：

```Java
hide(new BerylliumSphere[]{
    new BerlliumSphere(),
    new BerlliumSphere()
});
```

很多情況下這種語法寫程式碼更加方便。

表達式：

```Java
a = d;
```

顯示了你如何獲取指向一個陣列物件的引用並將其分配給另一個陣列物件。就像你可以處理其他類型的物件引用。現在 **a** 和 **d** 都指向了堆中的同一個陣列物件。

**ArrayOptions.java** 的第二部分展示了基元陣列的語法就像對像陣列一樣，除了基元陣列直接儲存基本資料類型的值。

<!-- Returning an Array -->

## 返回陣列

假設你寫了一個方法，這個方法不是返回一個元素，而是返回多個元素。對 C++/C 這樣的語言來說這是很困難的，因為你無法返回一個陣列，只能是返回一個指向陣列的指標。這會帶來一些問題，因為對陣列生存期的控制變得很混亂，這會導致記憶體洩露。

而在 Java 中，你只需返回陣列，你永遠不用為陣列擔心，只要你需要它，它就可用，垃圾收集器會在你用完後把它清理乾淨。

下面，我們返回一個 **字串** 陣列：

```Java
// arrays/IceCreamFlavors.java
// Returning arrays from methods
import java.util.*;
import static onjava.ArrayShow.*;

public class IceCreamFlavors {
  private static SplittableRandom rand =
    new SplittableRandom(47);
  static final String[] FLAVORS = {
    "Chocolate", "Strawberry", "Vanilla Fudge Swirl",
    "Mint Chip", "Mocha Almond Fudge", "Rum Raisin",
    "Praline Cream", "Mud Pie"
  };
  public static String[] flavorSet(int n) {
    if(n > FLAVORS.length)
      throw new IllegalArgumentException("Set too big");
    String[] results = new String[n];
    boolean[] picked = new boolean[FLAVORS.length];
    for(int i = 0; i < n; i++) {
      int t;
      do
        t = rand.nextInt(FLAVORS.length);
      while(picked[t]);
      results[i] = FLAVORS[t];
      picked[t] = true;
    }
    return results;
  }
  public static void main(String[] args) {
    for(int i = 0; i < 7; i++)
      show(flavorSet(3));
  }
}
/* Output:
[Praline Cream, Mint Chip, Vanilla Fudge Swirl]
[Strawberry, Vanilla Fudge Swirl, Mud Pie]
[Chocolate, Strawberry, Vanilla Fudge Swirl]
[Rum Raisin, Praline Cream, Chocolate]
[Mint Chip, Rum Raisin, Mocha Almond Fudge]
[Mocha Almond Fudge, Mud Pie, Vanilla Fudge Swirl]
[Mocha Almond Fudge, Mud Pie, Mint Chip]
*/
```

**flavorset()** 建立名為 **results** 的 **String** 類型的陣列。 這個陣列的大小 **n** 取決於你傳進方法的參數。然後從陣列 **FLAVORS** 中隨機選擇 flavors 並且把它們放進 **results** 裡並返回。返回一個陣列就像返回其他任何物件一樣，實際上返回的是引用。陣列是在 **flavorSet()** 中或者是在其他什麼地方建立的並不重要。垃圾收集器會清理你用完的陣列，你需要的陣列則會保留。

如果你必須要返回一系列不同類型的元素，你可以使用 [泛型](book/20-Generics.md) 中介紹的 **元組** 。

注意，當 **flavorSet()** 隨機選擇 flavors，它應該確保某個特定的選項沒被選中。這在一個 **do** 循環中執行，它將一直做出隨機選擇直到它發現一個元素不在 **picked** 陣列中。（一個字串

比較將顯示出隨機選中的元素是不是已經存在於 **results** 陣列中）。如果成功了，它將添加條目並且尋找下一個（ **i** 遞增）。輸出結果顯示 **flavorSet()** 每一次都是按照隨機順序選擇 flavors。

一直到現在，隨機數都是透過 **java.util.Random** 類生成的，這個類從 Java 1.0 就有，甚至更新過以提供 Java 8 流。現在我們可以介紹 Java 8 中的 **SplittableRandom** ,它不僅能在併行操作使用（你最終會學到），而且提供了一個高品質的隨機數。這本書的剩餘部分都使用  **SplittableRandom** 。

<!-- Multidimensional Arrays -->

## 多維陣列

要建立多維的基元陣列，你要用大括號來界定陣列中的向量：

```Java
// arrays/MultidimensionalPrimitiveArray.java
import java.util.*;

public class MultidimensionalPrimitiveArray {
  public static void main(String[] args) {
    int[][] a = {
      { 1, 2, 3, },
      { 4, 5, 6, },
    };
    System.out.println(Arrays.deepToString(a));
  }
}
/* Output:
[[1, 2, 3], [4, 5, 6]]
*/。
```

每個嵌套的大括號都代表了陣列的一個維度。

這個例子使用 **Arrays.deepToString()** 方法，將多維陣列轉換成 **String** 類型，就像輸出中顯示的那樣。

你也可以使用 **new** 分配陣列。這是一個使用 **new** 表達式分配的三維陣列：

```Java
// arrays/ThreeDWithNew.java
import java.util.*;

public class ThreeDWithNew {
  public static void main(String[] args) {
    // 3-D array with fixed length:
    int[][][] a = new int[2][2][4];
    System.out.println(Arrays.deepToString(a));
  }
}
/* Output:
[[[0, 0, 0, 0], [0, 0, 0, 0]], [[0, 0, 0, 0], [0, 0, 0,
0]]]
*/
```

倘若你不對基元陣列進行顯式的初始化，它的值會自動初始化。而物件陣列將被初始化為 **null** 。

組成矩陣的陣列中每一個向量都可以是任意長度的（這叫做不規則陣列）：

```Java
// arrays/RaggedArray.java
import java.util.*;

public class RaggedArray {
  static int val = 1;
  public static void main(String[] args) {
    SplittableRandom rand = new SplittableRandom(47);
    // 3-D array with varied-length vectors:
    int[][][] a = new int[rand.nextInt(7)][][];
    for(int i = 0; i < a.length; i++) {
      a[i] = new int[rand.nextInt(5)][];
      for(int j = 0; j < a[i].length; j++) {
        a[i][j] = new int[rand.nextInt(5)];
        Arrays.setAll(a[i][j], n -> val++); // [1]
      }
    }
    System.out.println(Arrays.deepToString(a));
  }
}
/* Output:
[[[1], []], [[2, 3, 4, 5], [6]], [[7, 8, 9], [10, 11,
12], []]]
*/
```

第一個 **new** 建立了一個陣列，這個陣列首元素長度隨機，其餘的則不確定。第二個 **new** 在 for 循環中給陣列填充了第二個元素，第三個 **new**  為陣列的最後一個索引填充元素。

* **[1]** Java 8 增加了 **Arrays.setAll()** 方法,其使用生成器來生成插入陣列中的值。此生成器符合函數式介面 **IntUnaryOperator** ，只使用一個非 **預設** 的方法 **ApplyAsint(int運算元)** 。 **Arrays.setAll()** 傳遞目前陣列索引作為運算元，因此一個選項是提供 **n -> n** 的 lambda 表達式來顯示陣列的索引（在上面的程式碼中很容易嘗試）。這裡，我們忽略索引，只是插入遞增計數器的值。

非基元的物件陣列也可以定義為不規則陣列。這裡，我們收集了許多使用大括號的 **new** 表達式：

```Java
// arrays/MultidimensionalObjectArrays.java
import java.util.*;

public class MultidimensionalObjectArrays {
  public static void main(String[] args) {
    BerylliumSphere[][] spheres = {
      { new BerylliumSphere(), new BerylliumSphere() },
      { new BerylliumSphere(), new BerylliumSphere(),
        new BerylliumSphere(), new BerylliumSphere() },
      { new BerylliumSphere(), new BerylliumSphere(),
        new BerylliumSphere(), new BerylliumSphere(),
        new BerylliumSphere(), new BerylliumSphere(),
        new BerylliumSphere(), new BerylliumSphere() },
    };
    System.out.println(Arrays.deepToString(spheres));
  }
}
/* Output:
[[Sphere 0, Sphere 1], [Sphere 2, Sphere 3, Sphere 4,
Sphere 5], [Sphere 6, Sphere 7, Sphere 8, Sphere 9,
Sphere 10, Sphere 11, Sphere 12, Sphere 13]]
*/
```

陣列初始化時使用自動裝箱技術：

```Java
// arrays/AutoboxingArrays.java
import java.util.*;

public class AutoboxingArrays {
  public static void main(String[] args) {
    Integer[][] a = { // Autoboxing:
      { 1, 2, 3, 4, 5, 6, 7, 8, 9, 10 },
      { 21, 22, 23, 24, 25, 26, 27, 28, 29, 30 },
      { 51, 52, 53, 54, 55, 56, 57, 58, 59, 60 },
      { 71, 72, 73, 74, 75, 76, 77, 78, 79, 80 },
    };
    System.out.println(Arrays.deepToString(a));
  }
}
/* Output:
[[1, 2, 3, 4, 5, 6, 7, 8, 9, 10], [21, 22, 23, 24, 25,
26, 27, 28, 29, 30], [51, 52, 53, 54, 55, 56, 57, 58,
59, 60], [71, 72, 73, 74, 75, 76, 77, 78, 79, 80]]
*/
```

以下是如何逐個構建非基元的物件陣列：

```Java
// arrays/AssemblingMultidimensionalArrays.java
// Creating multidimensional arrays
import java.util.*;

public class AssemblingMultidimensionalArrays {
  public static void main(String[] args) {
    Integer[][] a;
    a = new Integer[3][];
    for(int i = 0; i < a.length; i++) {
      a[i] = new Integer[3];
      for(int j = 0; j < a[i].length; j++)
        a[i][j] = i * j; // Autoboxing
    }
    System.out.println(Arrays.deepToString(a));
  }
}
/* Output:
[[0, 0, 0], [0, 1, 2], [0, 2, 4]]
*/
```

**i * j** 在這裡只是為了向 **Integer** 中添加有趣的值。

**Arrays.deepToString()** 方法同時適用於基元陣列和物件陣列：

```JAVA
// arrays/MultiDimWrapperArray.java
// (c)2017 MindView LLC: see Copyright.txt
// We make no guarantees that this code is fit for any purpose.
// Visit http://OnJava8.com for more book information.
// Multidimensional arrays of "wrapper" objects
import java.util.*;

public class MultiDimWrapperArray {
  public static void main(String[] args) {
    Integer[][] a1 = { // Autoboxing
      { 1, 2, 3, },
      { 4, 5, 6, },
    };
    Double[][][] a2 = { // Autoboxing
      { { 1.1, 2.2 }, { 3.3, 4.4 } },
      { { 5.5, 6.6 }, { 7.7, 8.8 } },
      { { 9.9, 1.2 }, { 2.3, 3.4 } },
    };
    String[][] a3 = {
      { "The", "Quick", "Sly", "Fox" },
      { "Jumped", "Over" },
      { "The", "Lazy", "Brown", "Dog", "&", "friend" },
    };
    System.out.println(
      "a1: " + Arrays.deepToString(a1));
    System.out.println(
      "a2: " + Arrays.deepToString(a2));
    System.out.println(
      "a3: " + Arrays.deepToString(a3));
  }
}
/* Output:
a1: [[1, 2, 3], [4, 5, 6]]
a2: [[[1.1, 2.2], [3.3, 4.4]], [[5.5, 6.6], [7.7,
8.8]], [[9.9, 1.2], [2.3, 3.4]]]
a3: [[The, Quick, Sly, Fox], [Jumped, Over], [The,
Lazy, Brown, Dog, &, friend]]
*/
```

同樣的，在 **Integer** 和 **Double** 陣列中，自動裝箱可為你建立包裝器物件。

<!-- Arrays and Generics -->
## 泛型陣列

一般來說，陣列和泛型並不能很好的結合。你不能實例化參數化類型的陣列：

```Java
Peel<Banana>[] peels = new Peel<Banana>[10]; // Illegal
```

類型擦除需要刪除參數類型訊息，而且陣列必須知道它們所儲存的確切類型，以強制保證類型安全。

但是，可以參數化陣列本身的類型：

```Java
// arrays/ParameterizedArrayType.java

class ClassParameter<T> {
  public T[] f(T[] arg) { return arg; }
}

class MethodParameter {
  public static <T> T[] f(T[] arg) { return arg; }
}

public class ParameterizedArrayType {
  public static void main(String[] args) {
    Integer[] ints = { 1, 2, 3, 4, 5 };
    Double[] doubles = { 1.1, 2.2, 3.3, 4.4, 5.5 };
    Integer[] ints2 =
      new ClassParameter<Integer>().f(ints);
    Double[] doubles2 =
      new ClassParameter<Double>().f(doubles);
    ints2 = MethodParameter.f(ints);
    doubles2 = MethodParameter.f(doubles);
  }
}
```

比起使用參數化類，使用參數化方法很方便。您不必為應用它的每個不同類型都實例化一個帶有參數的類，但是可以使它成為 **靜態** 的。你不能總是選擇使用參數化方法而不用參數化的類，但通常參數化方法是更好的選擇。

你不能建立泛型類型的陣列，這種說法並不完全正確。是的，編譯器不會讓你 *實例化* 一個泛型的陣列。但是，它將允許您建立對此類陣列的引用。例如：

```Java
List<String>[] ls;
```

無可爭議的，這可以透過編譯。儘管不能建立包含泛型的實際陣列物件，但是你可以建立一個非泛型的陣列並對其進行強制類型轉換：

```Java
// arrays/ArrayOfGenerics.java
import java.util.*;

public class ArrayOfGenerics {
  @SuppressWarnings("unchecked")
  public static void main(String[] args) {
    List<String>[] ls;
    List[] la = new List[10];
    ls = (List<String>[])la; // Unchecked cast
    ls[0] = new ArrayList<>();

    //- ls[1] = new ArrayList<Integer>();
    // error: incompatible types: ArrayList<Integer>
    // cannot be converted to List<String>
    //     ls[1] = new ArrayList<Integer>();
    //             ^

    // The problem: List<String> is a subtype of Object
    Object[] objects = ls; // So assignment is OK
    // Compiles and runs without complaint:
    objects[1] = new ArrayList<>();

    // However, if your needs are straightforward it is
    // possible to create an array of generics, albeit
    // with an "unchecked cast" warning:
    List<BerylliumSphere>[] spheres =
      (List<BerylliumSphere>[])new List[10];
    Arrays.setAll(spheres, n -> new ArrayList<>());
  }
}
```

一旦你有了對 **List<String>[]** 的引用 , 你會發現多了一些編譯時檢查。問題是陣列是協變的，所以 **List<String>[]** 也是一個 **Object[]**  ，你可以用這來將 **ArrayList<Integer> ** 分配進你的陣列，在編譯或者執行時都不會出錯。

如果你知道你不會進行向上類型轉換，你的需求相對簡單，那麼可以建立一個泛型陣列，它將提供基本的編譯時類型檢查。然而，一個泛型 **Collection** 實際上是一個比泛型陣列更好的選擇。

一般來說，您會發現泛型在類或方法的邊界上是有效的。在內部，擦除常常會使泛型不可使用。所以，就像下面的例子，不能建立泛型類型的陣列：

```Java
// arrays/ArrayOfGenericType.java

public class ArrayOfGenericType<T> {
  T[] array; // OK
  @SuppressWarnings("unchecked")
  public ArrayOfGenericType(int size) {
    // error: generic array creation:
    //- array = new T[size];
    array = (T[])new Object[size]; // unchecked cast
  }
  // error: generic array creation:
  //- public <U> U[] makeArray() { return new U[10]; }
}

```

擦除再次從中作梗，這個例子試圖建立已經擦除的類型陣列，因此它們是未知的類型。你可以建立一個 **物件** 陣列，然後對其進行強制類型轉換，但如果沒有 **@SuppressWarnings** 注釋，你將會得到一個 "unchecked" 警告，因為陣列實際上不真正支援而且將對類型 **T** 動態檢查 。這就是說，如果我建立了一個 **String[]** , Java將在編譯時和執行時強制執行，我只能在陣列中放置字串物件。然而，如果我建立一個 **Object[]** ,我可以把除了基元類別型外的任何東西放入陣列。



<!-- Arrays.fill() -->
## Arrays的fill方法

通常情況下，當對陣列和程式進行實驗時，能夠很輕易地生成充滿測試資料的陣列是很有幫助的。 Java 標準庫 **Arrays** 類包括一個普通的 **fill()** 方法，該方法將單個值複製到整個陣列，或者在物件陣列的情況下，將相同的引用複製到整個陣列：

```Java
// arrays/FillingArrays.java
// Using Arrays.fill()
import java.util.*;
import static onjava.ArrayShow.*;

public class FillingArrays {
  public static void main(String[] args) {
    int size = 6;
    boolean[] a1 = new boolean[size];
    byte[] a2 = new byte[size];
    char[] a3 = new char[size];
    short[] a4 = new short[size];
    int[] a5 = new int[size];
    long[] a6 = new long[size];
    float[] a7 = new float[size];
    double[] a8 = new double[size];
    String[] a9 = new String[size];
    Arrays.fill(a1, true);
    show("a1", a1);
    Arrays.fill(a2, (byte)11);
    show("a2", a2);
    Arrays.fill(a3, 'x');
    show("a3", a3);
    Arrays.fill(a4, (short)17);
    show("a4", a4);
    Arrays.fill(a5, 19);
    show("a5", a5);
    Arrays.fill(a6, 23);
    show("a6", a6);
    Arrays.fill(a7, 29);
    show("a7", a7);
    Arrays.fill(a8, 47);
    show("a8", a8);
    Arrays.fill(a9, "Hello");
    show("a9", a9);
    // Manipulating ranges:
    Arrays.fill(a9, 3, 5, "World");
    show("a9", a9);
  }
}gedan
/* Output:
a1: [true, true, true, true, true, true]
a2: [11, 11, 11, 11, 11, 11]
a3: [x, x, x, x, x, x]
a4: [17, 17, 17, 17, 17, 17]
a5: [19, 19, 19, 19, 19, 19]
a6: [23, 23, 23, 23, 23, 23]
a7: [29.0, 29.0, 29.0, 29.0, 29.0, 29.0]
a8: [47.0, 47.0, 47.0, 47.0, 47.0, 47.0]
a9: [Hello, Hello, Hello, Hello, Hello, Hello]
a9: [Hello, Hello, Hello, World, World, Hello]
*/

```

你既可以填充整個陣列，也可以像最後兩個語句所示，填充一系列的元素。但是由於你只能使用單個值呼叫 **Arrays.fill()** ，因此結果並非特別有用。


<!-- Arrays.setAll() -->
## Arrays的setAll方法

在Java 8中， 在**RaggedArray.java** 中引入並在 **ArrayOfGenerics.java.Array.setAll()** 中重用。它使用一個生成器並生成不同的值，可以選擇基於陣列的索引元素（透過訪問目前索引，生成器可以讀取陣列值並對其進行修改）。 **static Arrays.setAll()** 的重載簽名為：

* **void setAll(int[] a, IntUnaryOperator gen)**
* **void setAll(long[] a, IntToLongFunction gen)**
* **void setAll(double[] a, IntToDoubleFunctiongen)**
* **<T> void setAll(T[] a, IntFunction<? extendsT> gen)**

除了 **int** , **long** , **double** 有特殊的版本，其他的一切都由泛型版本處理。生成器不是 **Supplier** 因為它們不帶參數，並且必須將 **int** 陣列索引作為參數。

```java
// arrays/SimpleSetAll.java

import java.util.*;
import static onjava.ArrayShow.*;

class Bob {
  final int id;
  Bob(int n) { id = n; }
  @Override
  public String toString() { return "Bob" + id; }
}

public class SimpleSetAll {
  public static final int SZ = 8;
  static int val = 1;
  static char[] chars = "abcdefghijklmnopqrstuvwxyz"
    .toCharArray();
  static char getChar(int n) { return chars[n]; }
  public static void main(String[] args) {
    int[] ia = new int[SZ];
    long[] la = new long[SZ];
    double[] da = new double[SZ];
    Arrays.setAll(ia, n -> n); // [1]
    Arrays.setAll(la, n -> n);
    Arrays.setAll(da, n -> n);
    show(ia);
    show(la);
    show(da);
    Arrays.setAll(ia, n -> val++); // [2]
    Arrays.setAll(la, n -> val++);
    Arrays.setAll(da, n -> val++);
    show(ia);
    show(la);
    show(da);

    Bob[] ba = new Bob[SZ];
    Arrays.setAll(ba, Bob::new); // [3]
    show(ba);

    Character[] ca = new Character[SZ];
    Arrays.setAll(ca, SimpleSetAll::getChar); // [4]
    show(ca);
  }
}
/* Output:
[0, 1, 2, 3, 4, 5, 6, 7]
[0, 1, 2, 3, 4, 5, 6, 7]
[0.0, 1.0, 2.0, 3.0, 4.0, 5.0, 6.0, 7.0]
[1, 2, 3, 4, 5, 6, 7, 8]
[9, 10, 11, 12, 13, 14, 15, 16]
[17.0, 18.0, 19.0, 20.0, 21.0, 22.0, 23.0, 24.0]
[Bob0, Bob1, Bob2, Bob3, Bob4, Bob5, Bob6, Bob7]
[a, b, c, d, e, f, g, h]
*/

```

* **[1]** 這裡，我們只是將陣列索引作為值插入陣列。這將自動轉化為 **long** 和 **double** 版本。
* **[2]** 這個函數只需要接受索引就能產生正確結果。這個，我們忽略索引值並且使用 **val** 生成結果。
* **[3]** 方法引用有效，因為 **Bob** 的構造器接收一個 **int** 參數。只要我們傳遞的函數接收一個 **int** 參數且能產生正確的結果，就認為它完成了工作。
* **[4]** 為了處理除了  **int** ，**long** ，**double** 之外的基元類別型，請為基元建立包裝類的陣列。然後使用 **setAll()** 的泛型版本。請注意，**getChar（）** 生成基元類別型，因此這是自動裝箱到 **Character** 。


<!-- Incremental Generators -->
## 增量生成

這是一個方法庫，用於為不同類型生成增量值。

這些被作為內部類來生成容易記住的名字；比如，為了使用 **Integer** 工具你可以用 **new Conut.Interger()** , 如果你想要使用基本資料類型 **int** 工具，你可以用 **new Count.Pint()**  (基本類型的名字不能被直接使用，所以它們都在前面添加一個 **P**  來表示基本資料類型'primitive', 我們的第一選擇是使用基本類型名字後面跟著下劃線，比如 **int_** 和 **double_**  ,但是這種方式違背Java的命名習慣）。每個包裝類的生成器都使用 **get()** 方法實現了它的 **Supplier** 。要使用**Array.setAll()** ，一個重載的 **get(int n)** 方法要接受（並忽略）其參數，以便接受 **setAll()** 傳遞的索引值。

注意，透過使用包裝類的名稱作為內部類名，我們必須呼叫 **java.lang** 包來保證我們可以使用實際包裝類的名字：

```java
// onjava/Count.java
// Generate incremental values of different types
package onjava;
import java.util.*;
import java.util.function.*;
import static onjava.ConvertTo.*;

public interface Count {
  class Boolean
  implements Supplier<java.lang.Boolean> {
    private boolean b = true;
    @Override
    public java.lang.Boolean get() {
      b = !b;
      return java.lang.Boolean.valueOf(b);
    }
    public java.lang.Boolean get(int n) {
      return get();
    }
    public java.lang.Boolean[] array(int sz) {
      java.lang.Boolean[] result =
        new java.lang.Boolean[sz];
      Arrays.setAll(result, n -> get());
      return result;
    }
  }
  class Pboolean {
    private boolean b = true;
    public boolean get() {
      b = !b;
      return b;
    }
    public boolean get(int n) { return get(); }
    public boolean[] array(int sz) {
      return primitive(new Boolean().array(sz));
    }
  }
  class Byte
  implements Supplier<java.lang.Byte> {
    private byte b;
    @Override
    public java.lang.Byte get() { return b++; }
    public java.lang.Byte get(int n) {
      return get();
    }
    public java.lang.Byte[] array(int sz) {
      java.lang.Byte[] result =
        new java.lang.Byte[sz];
      Arrays.setAll(result, n -> get());
      return result;
    }
  }
  class Pbyte {
    private byte b;
    public byte get() { return b++; }
    public byte get(int n) { return get(); }
    public byte[] array(int sz) {
      return primitive(new Byte().array(sz));
    }
  }
  char[] CHARS =
    "abcdefghijklmnopqrstuvwxyz".toCharArray();
  class Character
  implements Supplier<java.lang.Character> {
    private int i;
    @Override
    public java.lang.Character get() {
      i = (i + 1) % CHARS.length;
      return CHARS[i];
    }
    public java.lang.Character get(int n) {
      return get();
    }
    public java.lang.Character[] array(int sz) {
      java.lang.Character[] result =
        new java.lang.Character[sz];
      Arrays.setAll(result, n -> get());
      return result;
    }
  }
  class Pchar {
    private int i;
    public char get() {
      i = (i + 1) % CHARS.length;
      return CHARS[i];
    }
    public char get(int n) { return get(); }
    public char[] array(int sz) {
      return primitive(new Character().array(sz));
    }
  }
  class Short
  implements Supplier<java.lang.Short> {
    short s;
    @Override
    public java.lang.Short get() { return s++; }
    public java.lang.Short get(int n) {
      return get();
    }
    public java.lang.Short[] array(int sz) {
      java.lang.Short[] result =
        new java.lang.Short[sz];
      Arrays.setAll(result, n -> get());
      return result;
    }
  }
  class Pshort {
    short s;
    public short get() { return s++; }
    public short get(int n) { return get(); }
    public short[] array(int sz) {
      return primitive(new Short().array(sz));
    }
  }
  class Integer
  implements Supplier<java.lang.Integer> {
    int i;
    @Override
    public java.lang.Integer get() { return i++; }
    public java.lang.Integer get(int n) {
      return get();
    }
    public java.lang.Integer[] array(int sz) {
      java.lang.Integer[] result =
        new java.lang.Integer[sz];
      Arrays.setAll(result, n -> get());
      return result;
    }
  }
  class Pint implements IntSupplier {
    int i;
    public int get() { return i++; }
    public int get(int n) { return get(); }
    @Override
    public int getAsInt() { return get(); }
    public int[] array(int sz) {
      return primitive(new Integer().array(sz));
    }
  }
  class Long
  implements Supplier<java.lang.Long> {
    private long l;
    @Override
    public java.lang.Long get() { return l++; }
    public java.lang.Long get(int n) {
      return get();
    }
    public java.lang.Long[] array(int sz) {
      java.lang.Long[] result =
        new java.lang.Long[sz];
      Arrays.setAll(result, n -> get());
      return result;
    }
  }
  class Plong implements LongSupplier {
    private long l;
    public long get() { return l++; }
    public long get(int n) { return get(); }
    @Override
    public long getAsLong() { return get(); }
    public long[] array(int sz) {
      return primitive(new Long().array(sz));
    }
  }
  class Float
  implements Supplier<java.lang.Float> {
    private int i;
    @Override
    public java.lang.Float get() {
      return java.lang.Float.valueOf(i++);
    }
    public java.lang.Float get(int n) {
      return get();
    }
    public java.lang.Float[] array(int sz) {
      java.lang.Float[] result =
        new java.lang.Float[sz];
      Arrays.setAll(result, n -> get());
      return result;
    }
  }
  class Pfloat {
    private int i;
    public float get() { return i++; }
    public float get(int n) { return get(); }
    public float[] array(int sz) {
      return primitive(new Float().array(sz));
    }
  }
  class Double
  implements Supplier<java.lang.Double> {
    private int i;
    @Override
    public java.lang.Double get() {
      return java.lang.Double.valueOf(i++);
    }
    public java.lang.Double get(int n) {
      return get();
    }
    public java.lang.Double[] array(int sz) {
      java.lang.Double[] result =
        new java.lang.Double[sz];
      Arrays.setAll(result, n -> get());
      return result;
    }
  }
  class Pdouble implements DoubleSupplier {
    private int i;
    public double get() { return i++; }
    public double get(int n) { return get(); }
    @Override
    public double getAsDouble() { return get(0); }
    public double[] array(int sz) {
      return primitive(new Double().array(sz));
    }
  }
}

```

對於 **int** ，**long** ，**double** 這三個有特殊 **Supplier** 介面的原始資料類型來說，**Pint** ， **Plong** 和 **Pdouble** 實現了這些介面。

這裡是對 **Count** 的測試，這同樣給我們提供了如何使用它的例子：

```java
// arrays/TestCount.java
// Test counting generators
import java.util.*;
import java.util.stream.*;
import onjava.*;
import static onjava.ArrayShow.*;

public class TestCount {
  static final int SZ = 5;
  public static void main(String[] args) {
    System.out.println("Boolean");
    Boolean[] a1 = new Boolean[SZ];
    Arrays.setAll(a1, new Count.Boolean()::get);
    show(a1);
    a1 = Stream.generate(new Count.Boolean())
      .limit(SZ + 1).toArray(Boolean[]::new);
    show(a1);
    a1 = new Count.Boolean().array(SZ + 2);
    show(a1);
    boolean[] a1b =
      new Count.Pboolean().array(SZ + 3);
    show(a1b);

    System.out.println("Byte");
    Byte[] a2 = new Byte[SZ];
    Arrays.setAll(a2, new Count.Byte()::get);
    show(a2);
    a2 = Stream.generate(new Count.Byte())
      .limit(SZ + 1).toArray(Byte[]::new);
    show(a2);
    a2 = new Count.Byte().array(SZ + 2);
    show(a2);
    byte[] a2b = new Count.Pbyte().array(SZ + 3);
    show(a2b);

    System.out.println("Character");
    Character[] a3 = new Character[SZ];
    Arrays.setAll(a3, new Count.Character()::get);
    show(a3);
    a3 = Stream.generate(new Count.Character())
      .limit(SZ + 1).toArray(Character[]::new);
    show(a3);
    a3 = new Count.Character().array(SZ + 2);
    show(a3);
    char[] a3b = new Count.Pchar().array(SZ + 3);
    show(a3b);

    System.out.println("Short");
    Short[] a4 = new Short[SZ];
    Arrays.setAll(a4, new Count.Short()::get);
    show(a4);
    a4 = Stream.generate(new Count.Short())
      .limit(SZ + 1).toArray(Short[]::new);
    show(a4);
    a4 = new Count.Short().array(SZ + 2);
    show(a4);
    short[] a4b = new Count.Pshort().array(SZ + 3);
    show(a4b);

    System.out.println("Integer");
    int[] a5 = new int[SZ];
    Arrays.setAll(a5, new Count.Integer()::get);
    show(a5);
    Integer[] a5b =
      Stream.generate(new Count.Integer())
        .limit(SZ + 1).toArray(Integer[]::new);
    show(a5b);
    a5b = new Count.Integer().array(SZ + 2);
    show(a5b);
    a5 = IntStream.generate(new Count.Pint())
      .limit(SZ + 1).toArray();
    show(a5);
    a5 = new Count.Pint().array(SZ + 3);
    show(a5);

    System.out.println("Long");
    long[] a6 = new long[SZ];
    Arrays.setAll(a6, new Count.Long()::get);
    show(a6);
    Long[] a6b = Stream.generate(new Count.Long())
      .limit(SZ + 1).toArray(Long[]::new);
    show(a6b);
    a6b = new Count.Long().array(SZ + 2);
    show(a6b);
    a6 = LongStream.generate(new Count.Plong())
      .limit(SZ + 1).toArray();
    show(a6);
    a6 = new Count.Plong().array(SZ + 3);
    show(a6);

    System.out.println("Float");
    Float[] a7 = new Float[SZ];
    Arrays.setAll(a7, new Count.Float()::get);
    show(a7);
    a7 = Stream.generate(new Count.Float())
      .limit(SZ + 1).toArray(Float[]::new);
    show(a7);
    a7 = new Count.Float().array(SZ + 2);
    show(a7);
    float[] a7b = new Count.Pfloat().array(SZ + 3);
    show(a7b);

    System.out.println("Double");
    double[] a8 = new double[SZ];
    Arrays.setAll(a8, new Count.Double()::get);
    show(a8);
    Double[] a8b =
      Stream.generate(new Count.Double())
        .limit(SZ + 1).toArray(Double[]::new);
    show(a8b);
    a8b = new Count.Double().array(SZ + 2);
    show(a8b);
    a8 = DoubleStream.generate(new Count.Pdouble())
      .limit(SZ + 1).toArray();
    show(a8);
    a8 = new Count.Pdouble().array(SZ + 3);
    show(a8);
  }
}
/* Output:
Boolean
[false, true, false, true, false]
[false, true, false, true, false, true]
[false, true, false, true, false, true, false]
[false, true, false, true, false, true, false, true]
Byte
[0, 1, 2, 3, 4]
[0, 1, 2, 3, 4, 5]
[0, 1, 2, 3, 4, 5, 6]
[0, 1, 2, 3, 4, 5, 6, 7]
Character
[b, c, d, e, f]
[b, c, d, e, f, g]
[b, c, d, e, f, g, h]
[b, c, d, e, f, g, h, i]
Short
[0, 1, 2, 3, 4]
[0, 1, 2, 3, 4, 5]
[0, 1, 2, 3, 4, 5, 6]
[0, 1, 2, 3, 4, 5, 6, 7]
Integer
[0, 1, 2, 3, 4]
[0, 1, 2, 3, 4, 5]
[0, 1, 2, 3, 4, 5, 6]
[0, 1, 2, 3, 4, 5]
[0, 1, 2, 3, 4, 5, 6, 7]
Long
[0, 1, 2, 3, 4]
[0, 1, 2, 3, 4, 5]
[0, 1, 2, 3, 4, 5, 6]
[0, 1, 2, 3, 4, 5]
[0, 1, 2, 3, 4, 5, 6, 7]
Float
[0.0, 1.0, 2.0, 3.0, 4.0]
[0.0, 1.0, 2.0, 3.0, 4.0, 5.0]
[0.0, 1.0, 2.0, 3.0, 4.0, 5.0, 6.0]
[0.0, 1.0, 2.0, 3.0, 4.0, 5.0, 6.0, 7.0]
Double
[0.0, 1.0, 2.0, 3.0, 4.0]
[0.0, 1.0, 2.0, 3.0, 4.0, 5.0]
[0.0, 1.0, 2.0, 3.0, 4.0, 5.0, 6.0]
[0.0, 1.0, 2.0, 3.0, 4.0, 5.0]
[0.0, 1.0, 2.0, 3.0, 4.0, 5.0, 6.0, 7.0]
*/

```

注意到原始陣列類型 **int[]** ，**long[]** ，**double[]** 可以直接被 **Arrays.setAll()** 填充，但是其他的原始類型都要求用包裝器類型的陣列。

通過 **Stream.generate()** 建立的包裝陣列顯示了 **toArray（）** 的重載用法，在這裡你應該提供給它要建立的陣列類型的構造器。

<!-- Random Generators -->
## 隨機生成

我們可以按照 **Count.java** 的結構建立一個生成隨機值的工具：

```java
// onjava/Rand.java
// (c)2017 MindView LLC: see Copyright.txt
// We make no guarantees that this code is fit for any purpose.
// Visit http://OnJava8.com for more book information.
// Generate random values of different types
package onjava;
import java.util.*;
import java.util.function.*;
import static onjava.ConvertTo.*;

public interface Rand {
  int MOD = 10_000;
  class Boolean
  implements Supplier<java.lang.Boolean> {
    SplittableRandom r = new SplittableRandom(47);
    @Override
    public java.lang.Boolean get() {
      return r.nextBoolean();
    }
    public java.lang.Boolean get(int n) {
      return get();
    }
    public java.lang.Boolean[] array(int sz) {
      java.lang.Boolean[] result =
        new java.lang.Boolean[sz];
      Arrays.setAll(result, n -> get());
      return result;
    }
  }
  class Pboolean {
    public boolean[] array(int sz) {
      return primitive(new Boolean().array(sz));
    }
  }
  class Byte
  implements Supplier<java.lang.Byte> {
    SplittableRandom r = new SplittableRandom(47);
    @Override
    public java.lang.Byte get() {
      return (byte)r.nextInt(MOD);
    }
    public java.lang.Byte get(int n) {
      return get();
    }
    public java.lang.Byte[] array(int sz) {
      java.lang.Byte[] result =
        new java.lang.Byte[sz];
      Arrays.setAll(result, n -> get());
      return result;
    }
  }
  class Pbyte {
    public byte[] array(int sz) {
      return primitive(new Byte().array(sz));
    }
  }
  class Character
  implements Supplier<java.lang.Character> {
    SplittableRandom r = new SplittableRandom(47);
    @Override
    public java.lang.Character get() {
      return (char)r.nextInt('a', 'z' + 1);
    }
    public java.lang.Character get(int n) {
      return get();
    }
    public java.lang.Character[] array(int sz) {
      java.lang.Character[] result =
        new java.lang.Character[sz];
      Arrays.setAll(result, n -> get());
      return result;
    }
  }
  class Pchar {
    public char[] array(int sz) {
      return primitive(new Character().array(sz));
    }
  }
  class Short
  implements Supplier<java.lang.Short> {
    SplittableRandom r = new SplittableRandom(47);
    @Override
    public java.lang.Short get() {
      return (short)r.nextInt(MOD);
    }
    public java.lang.Short get(int n) {
      return get();
    }
    public java.lang.Short[] array(int sz) {
      java.lang.Short[] result =
        new java.lang.Short[sz];
      Arrays.setAll(result, n -> get());
      return result;
    }
  }
  class Pshort {
    public short[] array(int sz) {
      return primitive(new Short().array(sz));
    }
  }
  class Integer
  implements Supplier<java.lang.Integer> {
    SplittableRandom r = new SplittableRandom(47);
    @Override
    public java.lang.Integer get() {
      return r.nextInt(MOD);
    }
    public java.lang.Integer get(int n) {
      return get();
    }
    public java.lang.Integer[] array(int sz) {
      int[] primitive = new Pint().array(sz);
      java.lang.Integer[] result =
        new java.lang.Integer[sz];
      for(int i = 0; i < sz; i++)
        result[i] = primitive[i];
      return result;
    }
  }
  class Pint implements IntSupplier {
    SplittableRandom r = new SplittableRandom(47);
    @Override
    public int getAsInt() {
      return r.nextInt(MOD);
    }
    public int get(int n) { return getAsInt(); }
    public int[] array(int sz) {
      return r.ints(sz, 0, MOD).toArray();
    }
  }
  class Long
  implements Supplier<java.lang.Long> {
    SplittableRandom r = new SplittableRandom(47);
    @Override
    public java.lang.Long get() {
      return r.nextLong(MOD);
    }
    public java.lang.Long get(int n) {
      return get();
    }
    public java.lang.Long[] array(int sz) {
      long[] primitive = new Plong().array(sz);
      java.lang.Long[] result =
        new java.lang.Long[sz];
      for(int i = 0; i < sz; i++)
        result[i] = primitive[i];
      return result;
    }
  }
  class Plong implements LongSupplier {
    SplittableRandom r = new SplittableRandom(47);
    @Override
    public long getAsLong() {
      return r.nextLong(MOD);
    }
    public long get(int n) { return getAsLong(); }
    public long[] array(int sz) {
      return r.longs(sz, 0, MOD).toArray();
    }
  }
  class Float
  implements Supplier<java.lang.Float> {
    SplittableRandom r = new SplittableRandom(47);
    @Override
    public java.lang.Float get() {
      return (float)trim(r.nextDouble());
    }
    public java.lang.Float get(int n) {
      return get();
    }
    public java.lang.Float[] array(int sz) {
      java.lang.Float[] result =
        new java.lang.Float[sz];
      Arrays.setAll(result, n -> get());
      return result;
    }
  }
  class Pfloat {
    public float[] array(int sz) {
      return primitive(new Float().array(sz));
    }
  }
  static double trim(double d) {
    return
      ((double)Math.round(d * 1000.0)) / 100.0;
  }
  class Double
  implements Supplier<java.lang.Double> {
    SplittableRandom r = new SplittableRandom(47);
    @Override
    public java.lang.Double get() {
      return trim(r.nextDouble());
    }
    public java.lang.Double get(int n) {
      return get();
    }
    public java.lang.Double[] array(int sz) {
      double[] primitive =
        new Rand.Pdouble().array(sz);
      java.lang.Double[] result =
        new java.lang.Double[sz];
      for(int i = 0; i < sz; i++)
        result[i] = primitive[i];
      return result;
    }
  }
  class Pdouble implements DoubleSupplier {
    SplittableRandom r = new SplittableRandom(47);
    @Override
    public double getAsDouble() {
      return trim(r.nextDouble());
    }
    public double get(int n) {
      return getAsDouble();
    }
    public double[] array(int sz) {
      double[] result = r.doubles(sz).toArray();
      Arrays.setAll(result,
        n -> result[n] = trim(result[n]));
      return result;
    }
  }
  class String
  implements Supplier<java.lang.String> {
    SplittableRandom r = new SplittableRandom(47);
    private int strlen = 7; // Default length
    public String() {}
    public String(int strLength) {
      strlen = strLength;
    }
    @Override
    public java.lang.String get() {
      return r.ints(strlen, 'a', 'z' + 1)
        .collect(StringBuilder::new,
                 StringBuilder::appendCodePoint,
                 StringBuilder::append).toString();
    }
    public java.lang.String get(int n) {
      return get();
    }
    public java.lang.String[] array(int sz) {
      java.lang.String[] result =
        new java.lang.String[sz];
      Arrays.setAll(result, n -> get());
      return result;
    }
  }
}

```

對於除了 **int** 、 **long** 和 **double** 之外的所有基本類型元素生成器，只生成陣列，而不是 Count 中看到的完整操作集。這只是一個設計選擇，因為本書不需要額外的功能。

下面是對所有 **Rand** 工具的測試：

```java
// arrays/TestRand.java
// Test random generators
import java.util.*;
import java.util.stream.*;
import onjava.*;
import static onjava.ArrayShow.*;

public class TestRand {
  static final int SZ = 5;
  public static void main(String[] args) {
    System.out.println("Boolean");
    Boolean[] a1 = new Boolean[SZ];
    Arrays.setAll(a1, new Rand.Boolean()::get);
    show(a1);
    a1 = Stream.generate(new Rand.Boolean())
      .limit(SZ + 1).toArray(Boolean[]::new);
    show(a1);
    a1 = new Rand.Boolean().array(SZ + 2);
    show(a1);
    boolean[] a1b =
      new Rand.Pboolean().array(SZ + 3);
    show(a1b);

    System.out.println("Byte");
    Byte[] a2 = new Byte[SZ];
    Arrays.setAll(a2, new Rand.Byte()::get);
    show(a2);
    a2 = Stream.generate(new Rand.Byte())
      .limit(SZ + 1).toArray(Byte[]::new);
    show(a2);
    a2 = new Rand.Byte().array(SZ + 2);
    show(a2);
    byte[] a2b = new Rand.Pbyte().array(SZ + 3);
    show(a2b);

    System.out.println("Character");
    Character[] a3 = new Character[SZ];
    Arrays.setAll(a3, new Rand.Character()::get);
    show(a3);
    a3 = Stream.generate(new Rand.Character())
      .limit(SZ + 1).toArray(Character[]::new);
    show(a3);
    a3 = new Rand.Character().array(SZ + 2);
    show(a3);
    char[] a3b = new Rand.Pchar().array(SZ + 3);
    show(a3b);

    System.out.println("Short");
    Short[] a4 = new Short[SZ];
    Arrays.setAll(a4, new Rand.Short()::get);
    show(a4);
    a4 = Stream.generate(new Rand.Short())
      .limit(SZ + 1).toArray(Short[]::new);
    show(a4);
    a4 = new Rand.Short().array(SZ + 2);
    show(a4);
    short[] a4b = new Rand.Pshort().array(SZ + 3);
    show(a4b);

    System.out.println("Integer");
    int[] a5 = new int[SZ];
    Arrays.setAll(a5, new Rand.Integer()::get);
    show(a5);
    Integer[] a5b =
      Stream.generate(new Rand.Integer())
        .limit(SZ + 1).toArray(Integer[]::new);
    show(a5b);
    a5b = new Rand.Integer().array(SZ + 2);
    show(a5b);
    a5 = IntStream.generate(new Rand.Pint())
      .limit(SZ + 1).toArray();
    show(a5);
    a5 = new Rand.Pint().array(SZ + 3);
    show(a5);

    System.out.println("Long");
    long[] a6 = new long[SZ];
    Arrays.setAll(a6, new Rand.Long()::get);
    show(a6);
    Long[] a6b = Stream.generate(new Rand.Long())
      .limit(SZ + 1).toArray(Long[]::new);
    show(a6b);
    a6b = new Rand.Long().array(SZ + 2);
    show(a6b);
    a6 = LongStream.generate(new Rand.Plong())
      .limit(SZ + 1).toArray();
    show(a6);
    a6 = new Rand.Plong().array(SZ + 3);
    show(a6);

    System.out.println("Float");
    Float[] a7 = new Float[SZ];
    Arrays.setAll(a7, new Rand.Float()::get);
    show(a7);
    a7 = Stream.generate(new Rand.Float())
      .limit(SZ + 1).toArray(Float[]::new);
    show(a7);
    a7 = new Rand.Float().array(SZ + 2);
    show(a7);
    float[] a7b = new Rand.Pfloat().array(SZ + 3);
    show(a7b);

    System.out.println("Double");
    double[] a8 = new double[SZ];
    Arrays.setAll(a8, new Rand.Double()::get);
    show(a8);
    Double[] a8b =
      Stream.generate(new Rand.Double())
        .limit(SZ + 1).toArray(Double[]::new);
    show(a8b);
    a8b = new Rand.Double().array(SZ + 2);
    show(a8b);
    a8 = DoubleStream.generate(new Rand.Pdouble())
      .limit(SZ + 1).toArray();
    show(a8);
    a8 = new Rand.Pdouble().array(SZ + 3);
    show(a8);

    System.out.println("String");
    String[] s = new String[SZ - 1];
    Arrays.setAll(s, new Rand.String()::get);
    show(s);
    s = Stream.generate(new Rand.String())
      .limit(SZ).toArray(String[]::new);
    show(s);
    s = new Rand.String().array(SZ + 1);
    show(s);

    Arrays.setAll(s, new Rand.String(4)::get);
    show(s);
    s = Stream.generate(new Rand.String(4))
      .limit(SZ).toArray(String[]::new);
    show(s);
    s = new Rand.String(4).array(SZ + 1);
    show(s);
  }
}
/* Output:
Boolean
[true, false, true, true, true]
[true, false, true, true, true, false]
[true, false, true, true, true, false, false]
[true, false, true, true, true, false, false, true]
Byte
[123, 33, 101, 112, 33]
[123, 33, 101, 112, 33, 31]
[123, 33, 101, 112, 33, 31, 0]
[123, 33, 101, 112, 33, 31, 0, -72]
Character
[b, t, p, e, n]
[b, t, p, e, n, p]
[b, t, p, e, n, p, c]
[b, t, p, e, n, p, c, c]
Short
[635, 8737, 3941, 4720, 6177]
[635, 8737, 3941, 4720, 6177, 8479]
[635, 8737, 3941, 4720, 6177, 8479, 6656]
[635, 8737, 3941, 4720, 6177, 8479, 6656, 3768]
Integer
[635, 8737, 3941, 4720, 6177]
[635, 8737, 3941, 4720, 6177, 8479]
[635, 8737, 3941, 4720, 6177, 8479, 6656]
[635, 8737, 3941, 4720, 6177, 8479]
[635, 8737, 3941, 4720, 6177, 8479, 6656, 3768]
Long
[6882, 3765, 692, 9575, 4439]
[6882, 3765, 692, 9575, 4439, 2638]
[6882, 3765, 692, 9575, 4439, 2638, 4011]
[6882, 3765, 692, 9575, 4439, 2638]
[6882, 3765, 692, 9575, 4439, 2638, 4011, 9610]
Float
[4.83, 2.89, 2.9, 1.97, 3.01]
[4.83, 2.89, 2.9, 1.97, 3.01, 0.18]
[4.83, 2.89, 2.9, 1.97, 3.01, 0.18, 0.99]
[4.83, 2.89, 2.9, 1.97, 3.01, 0.18, 0.99, 8.28]
Double
[4.83, 2.89, 2.9, 1.97, 3.01]
[4.83, 2.89, 2.9, 1.97, 3.01, 0.18]
[4.83, 2.89, 2.9, 1.97, 3.01, 0.18, 0.99]
[4.83, 2.89, 2.9, 1.97, 3.01, 0.18]
[4.83, 2.89, 2.9, 1.97, 3.01, 0.18, 0.99, 8.28]
String
[btpenpc, cuxszgv, gmeinne, eloztdv]
[btpenpc, cuxszgv, gmeinne, eloztdv, ewcippc]
[btpenpc, cuxszgv, gmeinne, eloztdv, ewcippc, ygpoalk]
[btpe, npcc, uxsz, gvgm, einn, eelo]
[btpe, npcc, uxsz, gvgm, einn]
[btpe, npcc, uxsz, gvgm, einn, eelo]
*/

```

注意（除了 **String** 部分之外），這段程式碼與 **TestCount.java** 中的程式碼相同，**Count** 被 **Rand** 取代。

<!-- Generics and Primitive Arrays -->
## 泛型和基本陣列
在本章的前面，我們被提醒，泛型不能和基元一起工作。在這種情況下，我們必須從基元陣列轉換為包裝類型的陣列，並且還必須從另一個方向轉換。下面是一個轉換器可以同時對所有類型的資料執行操作：

```java
// onjava/ConvertTo.java
// (c)2017 MindView LLC: see Copyright.txt
// We make no guarantees that this code is fit for any purpose.
// Visit http://OnJava8.com for more book information.
package onjava;

public interface ConvertTo {
  static boolean[] primitive(Boolean[] in) {
    boolean[] result = new boolean[in.length];
    for(int i = 0; i < in.length; i++)
      result[i] = in[i]; // Autounboxing
    return result;
  }
  static char[] primitive(Character[] in) {
    char[] result = new char[in.length];
    for(int i = 0; i < in.length; i++)
      result[i] = in[i];
    return result;
  }
  static byte[] primitive(Byte[] in) {
    byte[] result = new byte[in.length];
    for(int i = 0; i < in.length; i++)
      result[i] = in[i];
    return result;
  }
  static short[] primitive(Short[] in) {
    short[] result = new short[in.length];
    for(int i = 0; i < in.length; i++)
      result[i] = in[i];
    return result;
  }
  static int[] primitive(Integer[] in) {
    int[] result = new int[in.length];
    for(int i = 0; i < in.length; i++)
      result[i] = in[i];
    return result;
  }
  static long[] primitive(Long[] in) {
    long[] result = new long[in.length];
    for(int i = 0; i < in.length; i++)
      result[i] = in[i];
    return result;
  }
  static float[] primitive(Float[] in) {
    float[] result = new float[in.length];
    for(int i = 0; i < in.length; i++)
      result[i] = in[i];
    return result;
  }
  static double[] primitive(Double[] in) {
    double[] result = new double[in.length];
    for(int i = 0; i < in.length; i++)
      result[i] = in[i];
    return result;
  }
  // Convert from primitive array to wrapped array:
  static Boolean[] boxed(boolean[] in) {
    Boolean[] result = new Boolean[in.length];
    for(int i = 0; i < in.length; i++)
      result[i] = in[i]; // Autoboxing
    return result;
  }
  static Character[] boxed(char[] in) {
    Character[] result = new Character[in.length];
    for(int i = 0; i < in.length; i++)
      result[i] = in[i];
    return result;
  }
  static Byte[] boxed(byte[] in) {
    Byte[] result = new Byte[in.length];
    for(int i = 0; i < in.length; i++)
      result[i] = in[i];
    return result;
  }
  static Short[] boxed(short[] in) {
    Short[] result = new Short[in.length];
    for(int i = 0; i < in.length; i++)
      result[i] = in[i];
    return result;
  }
  static Integer[] boxed(int[] in) {
    Integer[] result = new Integer[in.length];
    for(int i = 0; i < in.length; i++)
      result[i] = in[i];
    return result;
  }
  static Long[] boxed(long[] in) {
    Long[] result = new Long[in.length];
    for(int i = 0; i < in.length; i++)
      result[i] = in[i];
    return result;
  }
  static Float[] boxed(float[] in) {
    Float[] result = new Float[in.length];
    for(int i = 0; i < in.length; i++)
      result[i] = in[i];
    return result;
  }
  static Double[] boxed(double[] in) {
    Double[] result = new Double[in.length];
    for(int i = 0; i < in.length; i++)
      result[i] = in[i];
    return result;
  }
}
```

**primitive()** 的每個版本都建立一個準確長度的適當基元陣列，然後從包裝類的 **in** 陣列中複製元素。如果任何包裝的陣列元素是 **null** ，你將得到一個異常（這是合理的—否則無法選擇有意義的值進行取代)。注意在這個任務中自動裝箱如何發生。

下面是對 **ConvertTo** 中所有方法的測試：

```java
// arrays/TestConvertTo.java
import java.util.*;
import onjava.*;
import static onjava.ArrayShow.*;
import static onjava.ConvertTo.*;

public class TestConvertTo {
  static final int SIZE = 6;
  public static void main(String[] args) {
    Boolean[] a1 = new Boolean[SIZE];
    Arrays.setAll(a1, new Rand.Boolean()::get);
    boolean[] a1p = primitive(a1);
    show("a1p", a1p);
    Boolean[] a1b = boxed(a1p);
    show("a1b", a1b);

    Byte[] a2 = new Byte[SIZE];
    Arrays.setAll(a2, new Rand.Byte()::get);
    byte[] a2p = primitive(a2);
    show("a2p", a2p);
    Byte[] a2b = boxed(a2p);
    show("a2b", a2b);

    Character[] a3 = new Character[SIZE];
    Arrays.setAll(a3, new Rand.Character()::get);
    char[] a3p = primitive(a3);
    show("a3p", a3p);
    Character[] a3b = boxed(a3p);
    show("a3b", a3b);

    Short[] a4 = new Short[SIZE];
    Arrays.setAll(a4, new Rand.Short()::get);
    short[] a4p = primitive(a4);
    show("a4p", a4p);
    Short[] a4b = boxed(a4p);
    show("a4b", a4b);

    Integer[] a5 = new Integer[SIZE];
    Arrays.setAll(a5, new Rand.Integer()::get);
    int[] a5p = primitive(a5);
    show("a5p", a5p);
    Integer[] a5b = boxed(a5p);
    show("a5b", a5b);

    Long[] a6 = new Long[SIZE];
    Arrays.setAll(a6, new Rand.Long()::get);
    long[] a6p = primitive(a6);
    show("a6p", a6p);
    Long[] a6b = boxed(a6p);
    show("a6b", a6b);

    Float[] a7 = new Float[SIZE];
    Arrays.setAll(a7, new Rand.Float()::get);
    float[] a7p = primitive(a7);
    show("a7p", a7p);
    Float[] a7b = boxed(a7p);
    show("a7b", a7b);

    Double[] a8 = new Double[SIZE];
    Arrays.setAll(a8, new Rand.Double()::get);
    double[] a8p = primitive(a8);
    show("a8p", a8p);
    Double[] a8b = boxed(a8p);
    show("a8b", a8b);
  }
}
/* Output:
a1p: [true, false, true, true, true, false]
a1b: [true, false, true, true, true, false]
a2p: [123, 33, 101, 112, 33, 31]
a2b: [123, 33, 101, 112, 33, 31]
a3p: [b, t, p, e, n, p]
a3b: [b, t, p, e, n, p]
a4p: [635, 8737, 3941, 4720, 6177, 8479]
a4b: [635, 8737, 3941, 4720, 6177, 8479]
a5p: [635, 8737, 3941, 4720, 6177, 8479]
a5b: [635, 8737, 3941, 4720, 6177, 8479]
a6p: [6882, 3765, 692, 9575, 4439, 2638]
a6b: [6882, 3765, 692, 9575, 4439, 2638]
a7p: [4.83, 2.89, 2.9, 1.97, 3.01, 0.18]
a7b: [4.83, 2.89, 2.9, 1.97, 3.01, 0.18]
a8p: [4.83, 2.89, 2.9, 1.97, 3.01, 0.18]
a8b: [4.83, 2.89, 2.9, 1.97, 3.01, 0.18]
*/
```

在每種情況下，原始陣列都是為包裝類型建立的，並使用 **Arrays.setAll()** 填充，正如我們在 **TestCouner.java** 中所做的那樣（這也驗證了 **Arrays.setAll()** 是否能同 **Integer** ，**Long** ，和 **Double** ）。然後 **ConvertTo.primitive()**  將包裝器陣列轉換為對應的基元陣列，**ConverTo.boxed()** 將其轉換回來。

<!-- Modifying Existing Array Elements -->
## 陣列元素修改

傳遞給 **Arrays.setAll()** 的生成器函數可以使用它接收到的陣列索引修改現有的陣列元素:

```JAVA
// arrays/ModifyExisting.java

import java.util.*;
import onjava.*;
import static onjava.ArrayShow.*;

public class ModifyExisting {
    public static void main(String[] args) {
        double[] da = new double[7];
        Arrays.setAll(da, new Rand.Double()::get);
        show(da);
        Arrays.setAll(da, n -> da[n] / 100); // [1]
        show(da);

    }
}

/* Output:
[4.83, 2.89, 2.9, 1.97, 3.01, 0.18, 0.99]
[0.0483, 0.028900000000000002, 0.028999999999999998,
0.0197, 0.0301, 0.0018, 0.009899999999999999]
*/

```

[1] Lambdas在這裡特別有用，因為陣列總是在lambda表達式的範圍內。


<!-- An Aside On Parallelism -->
## 陣列並行

我們很快就不得不面對並行的主題。例如，“並行”一詞在許多Java庫方法中使用。您可能聽說過類似“平行程式執行得更快”這樣的說法，這是有道理的—當您可以有多個處理器時，為什麼只有一個處理器在您的程式上工作呢? 如果您認為您應該利用其中的“並行”，這是很容易被原諒的。
要是這麼簡單就好了。不幸的是，透過採用這種方法，您可以很容易地編寫比非並行版本執行速度更慢的程式碼。在你深刻理解所有的問題之前，並行編程看起來更像是一門藝術而非科學。
以下是簡短的版本:用簡單的方法編寫程式碼。不要開始處理並行性，除非它成為一個問題。您仍然會遇到並行性。在本章中，我們將介紹一些為並行執行而編寫的Java庫方法。因此，您必須對它有足夠的了解，以便進行基本的討論，並避免出現錯誤。

在閱讀並發編程這一章之後，您將更深入地理解它(但是，唉，這還遠遠不夠。只是這些的話，充分理解這個主題是不可能的)。
在某些情況下，即使您只有一個處理器，無論您是否顯式地嘗試並行，並行實現是惟一的、最佳的或最符合邏輯的選擇。它是一個可以一直使用的工具，所以您必須了解它的相關問題。

最好從資料的角度來考慮並行性。對於大量資料(以及可用的額外處理器)，並行可能會有所幫助。但您也可能使事情變得更糟。

在本書的其餘部分，我們將遇到不同的情況:

- 1、所提供的惟一選項是並行的。這很簡單，因為我們別無選擇，只能使用它。這種情況是比較罕見的。

- 2、有多個選項，但是並行版本(通常是最新的版本)被設計成在任何地方都可以使用(甚至在那些不關心並行性的程式碼中)，如案例#1。我們將按預期使用並行版本。

- 3、案例1和案例2並不經常發生。相反，您將遇到某些演算法的兩個版本，一個用於並行使用，另一個用於正常使用。我將描述並行的一個，但不會在普通程式碼中使用它，因為它也許會產生所有可能的問題。

我建議您在自己的程式碼中採用這種方法。

[http://gee.cs.oswego.edu/dl/html/StreamParallelGuidance.html](要進一步了解為什麼這是一個難題，請參閱Doug Lea的文章。)

**parallelSetAll()**

流式編程產生優雅的程式碼。例如，假設我們想要建立一個數值由從零開始填充的長陣列：

```JAVA
// arrays/CountUpward.java

import java.util.stream.LongStream;

public class CountUpward {
    static long[] fillCounted(int size) {
        return LongStream.iterate(0, i -> i + 1).limit(size).toArray();
    }

    public static void main(String[] args) {
        long[] l1 = fillCounted(20); // No problem
        show(l1);
        // On my machine, this runs out of heap space:
        // - long[] l2 = fillCounted(10_000_000);
    }
}

/* Output:
[0, 1, 2, 3, 4, 5, 6, 7, 8, 9, 10, 11, 12, 13, 14, 15,
16, 17, 18, 19]
*/
```

**流** 實際上可以儲存到將近1000萬，但是之後就會耗盡堆空間。一般的 **setAll()** 是有效的，但是如果我們能更快地處理如此大量的數字，那就更好了。
我們可以使用 **setAll()** 初始化更大的陣列。如果速度成為一個問題，**Arrays.parallelSetAll()** 將(可能)更快地執行初始化(請記住並行性中描述的問題)。

```JAVA

// arrays/ParallelSetAll.java

import onjava.*;
import java.util.Arrays;

public class ParallelSetAll {
    static final int SIZE = 10_000_000;

    static void intArray() {
        int[] ia = new int[SIZE];
        Arrays.setAll(ia, new Rand.Pint()::get);
        Arrays.parallelSetAll(ia, new Rand.Pint()::get);
    }

    static void longArray() {
        long[] la = new long[SIZE];
        Arrays.setAll(la, new Rand.Plong()::get);
        Arrays.parallelSetAll(la, new Rand.Plong()::get);
    }

    public static void main(String[] args) {
        intArray();
        longArray();
    }
}
```

陣列分配和初始化是在單獨的方法中執行的，因為如果兩個陣列都在 **main()** 中分配，它會耗盡記憶體(至少在我的機器上是這樣。還有一些方法可以告訴Java在啟動時分配更多的記憶體)。



<!-- Arrays Utilities -->
## Arrays工具類

您已經看到了 **java.util.Arrays** 中的 **fill()** 和 **setAll()/parallelSetAll()** 。該類包含許多其他有用的 **靜態** 程式方法，我們將對此進行研究。

概述:

- **asList()**: 獲取任何序列或陣列，並將其轉換為一個 **列表集合** （集合章節介紹了此方法）。

- **copyOf()**：以新的長度建立現有陣列的新副本。

- **copyOfRange()**：建立現有陣列的一部分的新副本。

- **equals()**：比較兩個陣列是否相等。

- **deepEquals()**：多維陣列的相等性比較。

- **stream()**：生成陣列元素的流。

- **hashCode()**：生成陣列的雜湊值(您將在附錄中了解這意味著什麼:理解equals()和hashCode())。

- **deepHashCode()**: 多維陣列的雜湊值。

- **sort()**：排序陣列

- **parallelSort()**：對陣列進行並行排序，以提高速度。

- **binarySearch()**：在已排序的陣列中尋找元素。

- **parallelPrefix()**：使用提供的函數並行累積(以獲得速度)。基本上，就是陣列的reduce()。

- **spliterator()**：從陣列中產生一個Spliterator;這是本書沒有涉及到的流的進階部分。

- **toString()**：為陣列生成一個字串表示。你在整個章節中經常看到這種用法。

- **deepToString()**：為多維陣列生成一個字串。你在整個章節中經常看到這種用法。對於所有基本類型和物件，所有這些方法都是重載的。

<!-- Copying an Array -->
## 陣列複製

與使用for循環手工執行複製相比，**copyOf()** 和 **copyOfRange()** 複製陣列要快得多。這些方法被重載以處理所有類型。

我們從複製 **int** 和 **Integer** 陣列開始:
```JAVA
// arrays/ArrayCopying.java
// Demonstrate Arrays.copy() and Arrays.copyOf()

import onjava.*;

import java.util.Arrays;

import static onjava.ArrayShow.*;

class Sup {
    // Superclass
    private int id;

    Sup(int n) {
        id = n;
    }

    @Override
    public String toString() {
        return getClass().getSimpleName() + id;
    }
}

class Sub extends Sup { // Subclass

    Sub(int n) {
        super(n);
    }
}

public class ArrayCopying {
    public static final int SZ = 15;

    public static void main(String[] args) {
        int[] a1 = new int[SZ];
        Arrays.setAll(a1, new Count.Integer()::get);
        show("a1", a1);
        int[] a2 = Arrays.copyOf(a1, a1.length); // [1]
        // Prove they are distinct arrays:
        Arrays.fill(a1, 1);
        show("a1", a1);
        show("a2", a2);
        // Create a shorter result:
        a2 = Arrays.copyOf(a2, a2.length / 2); // [2]
        show("a2", a2);
        // Allocate more space:
        a2 = Arrays.copyOf(a2, a2.length + 5);
        show("a2", a2);
        // Also copies wrapped arrays:
        Integer[] a3 = new Integer[SZ]; // [3]
        Arrays.setAll(a3, new Count.Integer()::get);
        Integer[] a4 = Arrays.copyOfRange(a3, 4, 12);
        show("a4", a4);
        Sub[] d = new Sub[SZ / 2];
        Arrays.setAll(d, Sub::new); // Produce Sup[] from Sub[]:
        Sup[] b = Arrays.copyOf(d, d.length, Sup[].class); // [4]
        show(b); // This "downcast" works fine:
        Sub[] d2 = Arrays.copyOf(b, b.length, Sub[].class); // [5]
        show(d2); // Bad "downcast" compiles but throws exception:
        Sup[] b2 = new Sup[SZ / 2];
        Arrays.setAll(b2, Sup::new);
        try {
            Sub[] d3 = Arrays.copyOf(b2, b2.length, Sub[].class); // [6]
        } catch (Exception e) {
            System.out.println(e);
        }
    }
}
/* Output: a1: [0, 1, 2, 3, 4, 5, 6, 7, 8, 9, 10, 11, 12, 13, 14] a1: [1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1]
           a2:[0, 1, 2, 3, 4, 5, 6, 7, 8, 9, 10, 11, 12, 13, 14]a2:[0, 1, 2, 3, 4, 5, 6]a2:[
           0, 1, 2, 3, 4, 5, 6, 0, 0, 0, 0, 0]a4:[4, 5, 6, 7, 8, 9, 10, 11][Sub0, Sub1, Sub2, Sub3, Sub4, Sub5, Sub6][
           Sub0, Sub1, Sub2, Sub3, Sub4, Sub5, Sub6]java.lang.ArrayStoreException */

```

[1] 這是複製的基本方法;只需給出返回的複製陣列的大小。這對於編寫需要調整儲存大小的演算法很有幫助。複製之後，我們把a1的所有元素都設為1，以證明a1的變化不會影響a2中的任何東西。

[2] 透過更改最後一個參數，我們可以縮短或延長返回的複製陣列。

[3] **copyOf()** 和 **copyOfRange()** 也可以使用包裝類型。**copyOfRange()** 需要一個開始和結束索引。

[4] **copyOf()** 和 **copyOfRange()** 都有一個版本，該版本透過在方法呼叫的末尾添加目標類型來建立不同類型的陣列。我首先想到的是，這可能是一種從原生陣列生成包裝陣列的方法，反之亦然。
但這沒用。它的實際用途是“向上轉換”和“向下轉換”陣列。也就是說，如果您有一個子類型(衍生類型)的陣列，而您想要一個基類型的陣列，那麼這些方法將生成所需的陣列。

[5] 您甚至可以成功地“向下強制轉換”，並從超類型的陣列生成子類型的陣列。這個版本執行良好，因為我們只是“upcast”。

[6] 這個“陣列轉換”將編譯，但是如果類型不相容，您將得到一個執行時異常。在這裡，強制將基類型轉換為衍生類型是非法的，因為衍生物件中可能有基物件中沒有的屬性和方法。

實例表明，原生陣列和物件陣列都可以被複製。但是，如果複製物件的陣列，那麼只複製引用—不複製物件本身。這稱為淺複製(有關更多細節，請參閱附錄:傳遞和返回物件)。

還有一個方法 **System.arraycopy()** ，它將一個陣列複製到另一個已經分配的陣列中。這將不會執行自動裝箱或自動移除—兩個陣列必須是完全相同的類型。

<!-- Comparing Arrays -->
## 陣列比較

**陣列** 提供了 **equals()** 來比較一維陣列，以及 **deepEquals()** 來比較多維陣列。對於所有原生類型和物件，這些方法都是重載的。

陣列相等的含義：陣列必須有相同數量的元素，並且每個元素必須與另一個陣列中的對應元素相等，對每個元素使用 **equals()**(對於原生類型，使用原生類型的包裝類的 **equals()** 方法;例如，int的Integer.equals()。

```JAVA
// arrays/ComparingArrays.java
// Using Arrays.equals()

import java.util.*;
import onjava.*;

public class ComparingArrays {
    public static final int SZ = 15;

    static String[][] twoDArray() {
        String[][] md = new String[5][];
        Arrays.setAll(md, n -> new String[n]);
        for (int i = 0; i < md.length; i++) Arrays.setAll(md[i], new Rand.String()::get);
        return md;
    }

    public static void main(String[] args) {
        int[] a1 = new int[SZ], a2 = new int[SZ];
        Arrays.setAll(a1, new Count.Integer()::get);
        Arrays.setAll(a2, new Count.Integer()::get);
        System.out.println("a1 == a2: " + Arrays.equals(a1, a2));
        a2[3] = 11;
        System.out.println("a1 == a2: " + Arrays.equals(a1, a2));
        Integer[] a1w = new Integer[SZ], a2w = new Integer[SZ];
        Arrays.setAll(a1w, new Count.Integer()::get);
        Arrays.setAll(a2w, new Count.Integer()::get);
        System.out.println("a1w == a2w: " + Arrays.equals(a1w, a2w));
        a2w[3] = 11;
        System.out.println("a1w == a2w: " + Arrays.equals(a1w, a2w));
        String[][] md1 = twoDArray(), md2 = twoDArray();
        System.out.println(Arrays.deepToString(md1));
        System.out.println("deepEquals(md1, md2): " + Arrays.deepEquals(md1, md2));
        System.out.println("md1 == md2: " + Arrays.equals(md1, md2));
        md1[4][1] = "#$#$#$#";
        System.out.println(Arrays.deepToString(md1));
        System.out.println("deepEquals(md1, md2): " + Arrays.deepEquals(md1, md2));
    }
}

/* Output:
a1 == a2: true
a1 == a2: false
a1w == a2w: true
a1w == a2w: false
[[], [btpenpc], [btpenpc, cuxszgv], [btpenpc, cuxszgv,
 gmeinne], [btpenpc, cuxszgv, gmeinne, eloztdv]]
 deepEquals(md1, md2): true
 md1 == md2: false
 [[], [btpenpc], [btpenpc, cuxszgv], [btpenpc, cuxszgv,
 gmeinne], [btpenpc, #$#$#$#, gmeinne, eloztdv]]
 deepEquals(md1, md2): false
 */
```

最初，a1和a2是完全相等的，所以輸出是true，但是之後其中一個元素改變了，這使得結果為false。a1w和a2w是對一個封裝類型陣列重複該練習。

**md1** 和 **md2** 是透過 **twoDArray()** 以相同方式初始化的多維字串陣列。注意，**deepEquals()** 返回 **true**，因為它執行了適當的比較，而普通的 **equals()** 錯誤地返回 **false**。如果我們更改陣列中的一個元素，**deepEquals()** 將檢測它。

<!-- Streams and Arrays -->
## 流和陣列

**stream()** 方法很容易從某些類型的陣列中生成元素流。

```JAVA
// arrays/StreamFromArray.java

import java.util.*;
import onjava.*;

public class StreamFromArray {
    public static void main(String[] args) {
        String[] s = new Rand.String().array(10);
        Arrays.stream(s).skip(3).limit(5).map(ss -> ss + "!").forEach(System.out::println);
        int[] ia = new Rand.Pint().array(10);
        Arrays.stream(ia).skip(3).limit(5)
                .map(i -> i * 10).forEach(System.out::println);
        Arrays.stream(new long[10]);
        Arrays.stream(new double[10]);
        // Only int, long and double work:
        // - Arrays.stream(new boolean[10]);
        // - Arrays.stream(new byte[10]);
        // - Arrays.stream(new char[10]);
        // - Arrays.stream(new short[10]);
        // - Arrays.stream(new float[10]);
        // For the other types you must use wrapped arrays:
        float[] fa = new Rand.Pfloat().array(10);
        Arrays.stream(ConvertTo.boxed(fa));
        Arrays.stream(new Rand.Float().array(10));
    }
}
/* Output:
    eloztdv!
    ewcippc!
    ygpoalk!
    ljlbynx!
    taprwxz!
    47200
    61770
    84790
    66560
    37680
*/
```

只有“原生類型” **int**、**long** 和 **double** 可以與 **Arrays.stream()** 一起使用;對於其他的，您必須以某種方式獲得一個包裝類型的陣列。

通常，將陣列轉換為流來生成所需的結果要比直接運算元組容易得多。請注意，即使流已經“用完”(您不能重複使用它)，您仍然擁有該陣列，因此您可以以其他方式使用它----包括生成另一個流。

<!-- Sorting Arrays -->
## 陣列排序

根據物件的實際類型執行比較排序。一種方法是為不同的類型編寫對應的排序方法，但是這樣的程式碼不能復用。

編程設計的一個主要目標是“將易變的元素與穩定的元素分開”，在這裡，保持不變的程式碼是一般的排序演算法，但是變化的是物件的比較方式。因此，使用策略設計模式而不是將比較程式碼放入許多不同的排序原始碼中。使用策略模式時，變化的程式碼部分被封裝在一個單獨的類(策略物件)中。

您將一個策略物件交給相同的程式碼，該程式碼使用策略模式來實現其演算法。透過這種方式，您將使用相同的排序程式碼，使不同的物件表達不同的比較方式。

Java有兩種方式提供比較功能。第一種方法是透過實現 **java.lang.Comparable** 介面的原生方法。這是一個簡單的介面，只含有一個方法 **compareTo()**。該方法接受另一個與參數類型相同的物件作為參數，如果目前物件小於參數，則產生一個負值;如果參數相等，則產生零值;如果目前物件大於參數，則產生一個正值。

這裡有一個類，它實現了 **Comparable** 介面並示範了可比性，而且使用Java標準庫方法 **Arrays.sort()**:

```JAVA
// arrays/CompType.java
// Implementing Comparable in a class

import onjava.*;

import java.util.Arrays;
import java.util.SplittableRandom;

import static onjava.ArrayShow.*;

public class CompType implements Comparable<CompType> {
    private static int count = 1;
    private static SplittableRandom r = new SplittableRandom(47);
    int i;
    int j;

    public CompType(int n1, int n2) {
        i = n1;
        j = n2;
    }

    public static CompType get() {
        return new CompType(r.nextInt(100), r.nextInt(100));
    }

    public static void main(String[] args) {
        CompType[] a = new CompType[12];
        Arrays.setAll(a, n -> get());
        show("Before sorting", a);
        Arrays.sort(a);
        show("After sorting", a);
    }

    @Override
    public String toString() {
        String result = "[i = " + i + ", j = " + j + "]";
        if (count++ % 3 == 0) result += "\n";
        return result;
    }

    @Override
    public int compareTo(CompType rv) {
        return (i < rv.i ? -1 : (i == rv.i ? 0 : 1));
    }
}
/* Output:
Before sorting: [[i = 35, j = 37], [i = 41, j = 20], [i = 77, j = 79] ,
                [i = 56, j = 68], [i = 48, j = 93],
                [i = 70, j = 7] , [i = 0, j = 25],
                [i = 62, j = 34], [i = 50, j = 82] ,
                [i = 31, j = 67], [i = 66, j = 54],
                [i = 21, j = 6] ]
After sorting: [[i = 0, j = 25], [i = 21, j = 6], [i = 31, j = 67] ,
               [i = 35, j = 37], [i = 41, j = 20], [i = 48, j = 93] ,
               [i = 50, j = 82], [i = 56, j = 68], [i = 62, j = 34] ,
               [i = 66, j = 54], [i = 70, j = 7], [i = 77, j = 79] ]
*/
```

當您定義比較方法時，您有責任決定將一個物件與另一個物件進行比較意味著什麼。這裡，在比較中只使用i值和j值
將被忽略。

**get()** 方法透過使用隨機值初始化CompType物件來構建它們。在 **main()** 中，**get()** 與 **Arrays.setAll()** 一起使用，以填充一個 **CompType類型** 陣列，然後對其排序。如果沒有實現 **Comparable介面**，那麼當您試圖呼叫 **sort()** 時，您將在執行時獲得一個 **ClassCastException** 。這是因為 **sort()** 將其參數轉換為 **Comparable類型**。

現在假設有人給了你一個沒有實現 **Comparable介面** 的類，或者給了你一個實現 **Comparable介面** 的類，但是你不喜歡它的工作方式而願意有一個不同的對於此類型的比較方法。為了解決這個問題，建立一個實現 **Comparator** 介面的單獨的類(在集合一章中簡要介紹)。它有兩個方法，**compare()** 和 **equals()**。但是，除了特殊的效能需求外，您不需要實現 **equals()**，因為無論何時建立一個類，它都是隱式地繼承自 **Object**，**Object** 有一個equals()。您可以只使用預設的 **Object equals()** 來滿足介面的規範。

集合類(注意複數;我們將在下一章節討論它) 包含一個方法 **reverseOrder()**，它生成一個來 **Comparator**（比較器）反轉自然排序順序。這可以應用到比較物件：

```JAVA
// arrays/Reverse.java
// The Collections.reverseOrder() Comparator

import onjava.*;

import java.util.Arrays;
import java.util.Collections;

import static onjava.ArrayShow.*;

public class Reverse {
    public static void main(String[] args) {
        CompType[] a = new CompType[12];
        Arrays.setAll(a, n -> CompType.get());
        show("Before sorting", a);
        Arrays.sort(a, Collections.reverseOrder());
        show("After sorting", a);
    }
}
/* Output:
Before sorting: [[i = 35, j = 37], [i = 41, j = 20],
                [i = 77, j = 79] , [i = 56, j = 68],
                [i = 48, j = 93], [i = 70, j = 7],
                [i = 0, j = 25], [i = 62, j = 34],
                [i = 50, j = 82] , [i = 31, j = 67],
                [i = 66, j = 54], [i = 21, j = 6] ]
After sorting: [[i = 77, j = 79], [i = 70, j = 7],
                [i = 66, j = 54] , [i = 62, j = 34],
                [i = 56, j = 68], [i = 50, j = 82] ,
                [i = 48, j = 93], [i = 41, j = 20],
                [i = 35, j = 37] , [i = 31, j = 67],
                [i = 21, j = 6], [i = 0, j = 25] ]
*/
```

您還可以編寫自己的比較器。這個比較CompType物件基於它們的j值而不是它們的i值:

```JAVA
// arrays/ComparatorTest.java
// Implementing a Comparator for a class

import onjava.*;

import java.util.Arrays;
import java.util.Comparator;

import static onjava.ArrayShow.*;

class CompTypeComparator implements Comparator<CompType> {
    public int compare(CompType o1, CompType o2) {
        return (o1.j < o2.j ? -1 : (o1.j == o2.j ? 0 : 1));
    }
}

public class ComparatorTest {
    public static void main(String[] args) {
        CompType[] a = new CompType[12];
        Arrays.setAll(a, n -> CompType.get());
        show("Before sorting", a);
        Arrays.sort(a, new CompTypeComparator());
        show("After sorting", a);
    }
}
/* Output:
Before sorting:[[i = 35, j = 37], [i = 41, j = 20], [i = 77, j = 79] ,
                [i = 56, j = 68], [i = 48, j = 93], [i = 70, j = 7] ,
                [i = 0, j = 25], [i = 62, j = 34], [i = 50, j = 82],
                [i = 31, j = 67], [i = 66, j = 54], [i = 21, j = 6] ]
After sorting: [[i = 21, j = 6], [i = 70, j = 7], [i = 41, j = 20] ,
                [i = 0, j = 25], [i = 62, j = 34], [i = 35, j = 37] ,
                [i = 66, j = 54], [i = 31, j = 67], [i = 56, j = 68] ,
                [i = 77, j = 79], [i = 50, j = 82], [i = 48, j = 93] ]
*/
```

<!-- Using Arrays.sort() -->
## Arrays.sort 的使用

使用內建的排序方法，您可以對實現了 **Comparable** 介面或具有 **Comparator** 的任何物件陣列 或 任何原生陣列進行排序。這裡我們生成一個隨機字串物件陣列並對其排序:

```JAVA
// arrays/StringSorting.java
// Sorting an array of Strings

import onjava.*;

import java.util.Arrays;
import java.util.Collections;

import static onjava.ArrayShow.*;

public class StringSorting {
    public static void main(String[] args) {
        String[] sa = new Rand.String().array(20);
        show("Before sort", sa);
        Arrays.sort(sa);
        show("After sort", sa);
        Arrays.sort(sa, Collections.reverseOrder());
        show("Reverse sort", sa);
        Arrays.sort(sa, String.CASE_INSENSITIVE_ORDER);
        show("Case-insensitive sort", sa);
    }
}
/* Output:
Before sort: [btpenpc, cuxszgv, gmeinne, eloztdv, ewcippc,
            ygpoalk, ljlbynx, taprwxz, bhmupju, cjwzmmr,
            anmkkyh, fcjpthl, skddcat, jbvlgwc, mvducuj,
            ydpulcq, zehpfmm, zrxmclh, qgekgly, hyoubzl]

After sort: [anmkkyh, bhmupju, btpenpc, cjwzmmr, cuxszgv,
            eloztdv, ewcippc, fcjpthl, gmeinne, hyoubzl,
            jbvlgwc, ljlbynx, mvducuj, qgekgly, skddcat,
            taprwxz, ydpulcq, ygpoalk, zehpfmm, zrxmclh]

Reverse sort: [zrxmclh, zehpfmm, ygpoalk, ydpulcq,taprwxz,
            skddcat, qgekgly, mvducuj, ljlbynx, jbvlgwc,
            hyoubzl, gmeinne, fcjpthl, ewcippc, eloztdv,
            cuxszgv, cjwzmmr, btpenpc, bhmupju, anmkkyh]

Case-insensitive sort: [anmkkyh, bhmupju, btpenpc, cjwzmmr,
                cuxszgv, eloztdv, ewcippc, fcjpthl, gmeinne,
                hyoubzl, jbvlgwc, ljlbynx, mvducuj, qgekgly,
                skddcat, taprwxz, ydpulcq, ygpoalk, zehpfmm, zrxmclh]
*/
```

注意字串排序演算法中的輸出。它是字典式的，所以它把所有以大寫字母開頭的單詞放在前面，然後是所有以小寫字母開頭的單詞。(電話簿通常是這樣分類的。)無論大小寫，要將單詞組合在一起，請使用 **String.CASE_INSENSITIVE_ORDER** ，如對sort()的最後一次呼叫所示。

Java標準庫中使用的排序演算法被設計為最適合您正在排序的類型----原生類型的快速排序和物件的歸併排序。


<!-- Sorting in Parallel -->
## 並行排序

如果排序性能是一個問題，那麼可以使用 **Java 8 parallelSort()**，它為所有不可預見的情況(包括陣列的排序區域或使用了比較器)提供了重載版本。為了查看相比於普通的sort(), **parallelSort()** 的優點，我們使用了用來驗證程式碼時的 **JMH**：

```java
// arrays/jmh/ParallelSort.java
package arrays.jmh;

import onjava.*;
import org.openjdk.jmh.annotations.*;

import java.util.Arrays;

@State(Scope.Thread)
public class ParallelSort {
    private long[] la;

    @Setup
    public void setup() {
        la = new Rand.Plong().array(100_000);
    }

    @Benchmark
    public void sort() {
        Arrays.sort(la);
    }

    @Benchmark
    public void parallelSort() {
        Arrays.parallelSort(la);
    }
}
```

**parallelSort()** 演算法將大陣列分割成更小的陣列，直到陣列大小達到極限，然後使用普通的 **Arrays .sort()** 方法。然後合併結果。該演算法需要不大於原始陣列的額外工作空間。

您可能會看到不同的結果，但是在我的機器上，並行排序將速度提高了大約3倍。由於並行版本使用起來很簡單，所以很容易考慮在任何地方使用它，而不是
**Arrays.sort ()**。當然，它可能不是那麼簡單—看看微基準測試。


<!-- Searching with Arrays.binarySearch() -->
## binarySearch二分尋找

一旦陣列被排序，您就可以透過使用 **Arrays.binarySearch()** 來執行對特定項的快速搜尋。但是，如果嘗試在未排序的陣列上使用 **binarySearch()**，結果是不可預測的。下面的範例使用 **Rand.Pint** 類來建立一個填充隨機整形值的陣列，然後呼叫 **getAsInt()** (因為 **Rand.Pint** 是一個 **IntSupplier**)來產生搜尋值:

```JAVA
// arrays/ArraySearching.java
// Using Arrays.binarySearch()

import onjava.*;

import java.util.Arrays;

import static onjava.ArrayShow.*;

public class ArraySearching {
    public static void main(String[] args) {
        Rand.Pint rand = new Rand.Pint();
        int[] a = new Rand.Pint().array(25);
        Arrays.sort(a);
        show("Sorted array", a);
        while (true) {
            int r = rand.getAsInt();
            int location = Arrays.binarySearch(a, r);
            if (location >= 0) {
                System.out.println("Location of " + r + " is " + location + ", a[" + location + "] is " + a[location]);
                break; // Out of while loop
            }
        }
    }
}
/* Output:
Sorted array: [125, 267, 635, 650, 1131, 1506, 1634, 2400, 2766,
               3063, 3768, 3941, 4720, 4762, 4948, 5070, 5682,
               5807, 6177, 6193, 6656, 7021, 8479, 8737, 9954]
Location of 635 is 2, a[2] is 635
*/
```

在while循環中，隨機值作為搜尋項生成，直到在陣列中找到其中一個為止。

如果找到了搜尋項，**Arrays.binarySearch()** 將生成一個大於或等於零的值。否則，它將產生一個負值，表示如果手動維護已排序的陣列，則應該插入元素的位置。產生的值是 -(插入點) - 1 。插入點是大於鍵的第一個元素的索引，如果陣列中的所有元素都小於指定的鍵，則是 **a.size()** 。

如果陣列包含重複的元素，則無法保證找到其中的那些重複項。搜尋演算法不是為了支援重複的元素，而是為了容忍它們。如果需要沒有重複元素的排序列表，可以使用 **TreeSet** (用於維持排序順序)或 **LinkedHashSet** (用於維持插入順序)。這些類自動為您處理所有的細節。只有在出現性能瓶頸的情況下，才應該使用手工維護的陣列取代這些類中的一個。

如果使用比較器(原語陣列不允許使用比較器進行排序)對物件陣列進行排序，那麼在執行 **binarySearch()** (使用重載版本的binarySearch())時必須包含相同的比較器。例如，可以修改 **StringSorting.java** 來執行搜尋:

```JAVA
// arrays/AlphabeticSearch.java
// Searching with a Comparator

import onjava.*;

import java.util.Arrays;

import static onjava.ArrayShow.*;

public class AlphabeticSearch {
    public static void main(String[] args) {
        String[] sa = new Rand.String().array(30);
        Arrays.sort(sa, String.CASE_INSENSITIVE_ORDER);
        show(sa);
        int index = Arrays.binarySearch(sa, sa[10], String.CASE_INSENSITIVE_ORDER);
        System.out.println("Index: " + index + "\n" + sa[index]);
    }
}
/* Output:
[anmkkyh, bhmupju, btpenpc, cjwzmmr, cuxszgv, eloztdv, ewcippc,
ezdeklu, fcjpthl, fqmlgsh, gmeinne, hyoubzl, jbvlgwc, jlxpqds,
ljlbynx, mvducuj, qgekgly, skddcat, taprwxz, uybypgp, vjsszkn,
vniyapk, vqqakbm, vwodhcf, ydpulcq, ygpoalk, yskvett, zehpfmm,
zofmmvm, zrxmclh]
Index: 10 gmeinne
*/
```
比較器必須作為第三個參數傳遞給重載的 **binarySearch()** 。在本例中，成功是有保證的，因為搜尋項是從陣列本身中選擇的。

<!-- Accumulating with parallelPrefix() -->
## parallelPrefix並行前綴

沒有“prefix()”方法，只有 **parallelPrefix()**。這類似於 **Stream** 類中的 **reduce()** 方法:它對前一個元素和目前元素執行一個操作，並將結果放入目前元素位置:

```JAVA
// arrays/ParallelPrefix1.java

import onjava.*;

import java.util.Arrays;

import static onjava.ArrayShow.*;

public class ParallelPrefix1 {
    public static void main(String[] args) {
        int[] nums = new Count.Pint().array(10);
        show(nums);
        System.out.println(Arrays.stream(nums).reduce(Integer::sum).getAsInt());
        Arrays.parallelPrefix(nums, Integer::sum);
        show(nums);
        System.out.println(Arrays.stream(new Count.Pint().array(6)).reduce(Integer::sum).getAsInt());
    }
}
/* Output:
[0, 1, 2, 3, 4, 5, 6, 7, 8, 9]
45
[0, 1, 3, 6, 10, 15, 21, 28, 36, 45]
15
*/
```

這裡我們對陣列應用Integer::sum。在位置0中，它將先前計算的值(因為沒有先前的值)與原始陣列位置0中的值組合在一起。在位置1中，它獲取之前計算的值(它只是儲存在位置0中)，並將其與位置1中先前計算的值相結合。依次往復。

使用 **Stream.reduce()**，您只能得到最終結果，而使用 **Arrays.parallelPrefix()**，您還可以得到所有中間計算，以確保它們是有用的。注意，第二個 **Stream.reduce()** 計算的結果已經在 **parallelPrefix()** 計算的陣列中。

使用字串可能更清楚:

```JAVA
// arrays/ParallelPrefix2.java

import onjava.*;

import java.util.Arrays;

import static onjava.ArrayShow.*;

public class ParallelPrefix2 {
    public static void main(String[] args) {
        String[] strings = new Rand.String(1).array(8);
        show(strings);
        Arrays.parallelPrefix(strings, (a, b) -> a + b);
        show(strings);
    }
}
/* Output:
[b, t, p, e, n, p, c, c]
[b, bt, btp, btpe, btpen, btpenp, btpenpc, btpenpcc]
*/
```

如前所述，使用流進行初始化非常優雅，但是對於大型陣列，這種方法可能會耗盡堆空間。使用 **setAll()** 執行初始化更節省記憶體:

```JAVA
// arrays/ParallelPrefix3.java
// {ExcludeFromTravisCI}

import java.util.Arrays;

public class ParallelPrefix3 {
    static final int SIZE = 10_000_000;

    public static void main(String[] args) {
        long[] nums = new long[SIZE];
        Arrays.setAll(nums, n -> n);
        Arrays.parallelPrefix(nums, Long::sum);
        System.out.println("First 20: " + nums[19]);
        System.out.println("First 200: " + nums[199]);
        System.out.println("All: " + nums[nums.length - 1]);
    }
}
/* Output:
First 20: 190
First 200: 19900
All: 49999995000000
*/
```
因為正確使用 **parallelPrefix()** 可能相當複雜，所以通常應該只在存在記憶體或速度問題(或兩者都有)時使用。否則，**Stream.reduce()** 應該是您的首選。


<!-- Summary -->
## 本章小結

Java為固定大小的低級陣列提供了合理的支援。這種陣列強調的是性能而不是靈活性，就像C和c++陣列模型一樣。在Java的最初版本中，固定大小的低級陣列是絕對必要的，這不僅是因為Java設計人員選擇包含原生類型(也考慮到性能)，還因為那個版本對集合的支援非常少。因此，在早期的Java版本中，選擇陣列總是合理的。

在Java的後續版本中，集合支援得到了顯著的改進，現在集合在除性能外的所有方面都優於陣列，即使這樣，集合的效能也得到了顯著的改進。正如本書其他部分所述，無論如何，性能問題通常不會出現在您設想的地方。

使用自動裝箱和泛型，在集合中儲存原生類型是毫不費力的，這進一步鼓勵您用集合取代低級陣列。由於泛型產生類型安全的集合，陣列在這方面也不再有優勢。

如本章所述，當您嘗試使用泛型時，您將看到泛型對陣列是相當不友好的。通常，即使可以讓泛型和陣列以某種形式一起工作(在下一章中您將看到)，在編譯期間仍然會出現“unchecked”警告。

有幾次，當我們討論特定的例子時，我直接從Java語言設計人員那裡聽到我應該使用集合而不是陣列(我使用陣列來示範特定的技術，所以我沒有這個選項)。

所有這些問題都表明，在使用Java的最新版本進行編程時，應該“優先選擇集合而不是陣列”。只有當您證明性能是一個問題(並且切換到一個陣列實際上會有很大的不同)時，才應該重構到陣列。這是一個相當大膽的聲明，但是有些語言根本沒有固定大小的低級陣列。它們只有可調整大小的集合，而且比C/C++/java風格的陣列功能多得多。例如，Python有一個使用基本陣列語法的列表類型，但是具有更大的功能—您甚至可以從它繼承:

```Python
# arrays/PythonLists.py

aList=[1,2,3,4,5]print(type(aList)) #<type 'list'>
print(aList) # [1,2,3,4,5]
        print(aList[4]) # 5Basic list indexing
        aList.append(6) # lists can be resized
        aList+=[7,8] # Add a list to a list
        print(aList) # [1,2,3,4,5,6,7,8]
        aSlice=aList[2:4]
        print(aSlice) # [3,4]

class MyList(list): # Inherit from list
        # Define a method;'this'pointer is explicit:
        def getReversed(self):
            reversed=self[:] # Copy list using slices
            reversed.reverse() # Built-in list method
            return reversed
        # No'new'necessary for object creation:
        list2=MyList(aList)
        print(type(list2)) #<class '__main__.MyList'>
        print(list2.getReversed()) # [8,7,6,5,4,3,2,1]
        output="""
        <class 'list'>
        [1, 2, 3, 4, 5]
        5
        [1, 2, 3, 4, 5, 6, 7, 8]
        [3, 4]
        <class '__main__.MyList'>
        [8, 7, 6, 5, 4, 3, 2, 1]
        """

```
前一章介紹了基本的Python語法。在這裡，透過用方括號包圍以逗號分隔的物件序列來建立列表。結果是一個執行時類型為list的物件(print語句的輸出顯示為同一行中的注釋)。列印列表的結果與在Java中使用Arrays.toString()的結果相同。
透過將 : 操作符放在索引操作中，透過切片來建立列表的子序列。list類型有更多的內建操作，通常只需要序列類型。
MyList是一個類定義;基類放在括號內。在類內部，def語句生成方法，該方法的第一個參數在Java中自動與之等價，除了在Python中它是顯式的，而且標識符self是按約定使用的(它不是關鍵字)。注意建構子是如何自動繼承的。

雖然一切在Python中真的是一個物件(包括整數和浮點類型),你仍然有一個安全門,因為你可以最佳化性能關鍵型的部分程式碼編寫擴展的C, c++或使用特殊的工具設計容易加速您的Python程式碼(有很多)。透過這種方式，可以在不影響性能改進的情況下保持物件的純度。

PHP甚至更進一步，它只有一個陣列類型，既充當int索引陣列，又充當關聯陣列(Map)。

在經歷了這麼多年的Java發展之後，我們可以很有趣地推測，如果重新開始，設計人員是否會將原生類型和低級陣列放在該語言中(同樣在JVM上執行的Scala語言不包括這些)。如果不考慮這些，就有可能開發出一種真正純粹的物件導向語言(儘管有這樣的說法，Java並不是一種純粹的物件導向語言，這正是因為它的底層缺陷)。關於效率的最初爭論總是令人信服的，但是隨著時間的推移，我們已經看到了從這個想法向更高層次的元件(如集合)的演進。此外，如果集合可以像在某些語言中一樣構建到核心語言中，那麼編譯器就有更好的機會進行最佳化。

撇開““Green-fields”的推測不談，我們肯定會被陣列所困擾，當你閱讀程式碼時就會看到它們。然而，集合幾乎總是更好的選擇。


<!-- 分頁 -->

<div style="page-break-after: always;"></div>
