[TOC]

<!-- Appendix: Collection Topics -->
# 附錄:集合主題

> 本附錄是一些比[第十二章 集合]()中介紹的更進階的內容。

<!-- Sample Data -->
## 範例資料

這裡建立一些樣本資料用於集合範例。 以下資料將顏色名稱與HTML顏色的RGB值相關聯。請注意，每個鍵和值都是唯一的：

```java
// onjava/HTMLColors.java
// Sample data for collection examples
package onjava;
import java.util.*;
import java.util.stream.*;
import java.util.concurrent.*;

public class HTMLColors {
  public static final Object[][] ARRAY = {
    { 0xF0F8FF, "AliceBlue" },
    { 0xFAEBD7, "AntiqueWhite" },
    { 0x7FFFD4, "Aquamarine" },
    { 0xF0FFFF, "Azure" },
    { 0xF5F5DC, "Beige" },
    { 0xFFE4C4, "Bisque" },
    { 0x000000, "Black" },
    { 0xFFEBCD, "BlanchedAlmond" },
    { 0x0000FF, "Blue" },
    { 0x8A2BE2, "BlueViolet" },
    { 0xA52A2A, "Brown" },
    { 0xDEB887, "BurlyWood" },
    { 0x5F9EA0, "CadetBlue" },
    { 0x7FFF00, "Chartreuse" },
    { 0xD2691E, "Chocolate" },
    { 0xFF7F50, "Coral" },
    { 0x6495ED, "CornflowerBlue" },
    { 0xFFF8DC, "Cornsilk" },
    { 0xDC143C, "Crimson" },
    { 0x00FFFF, "Cyan" },
    { 0x00008B, "DarkBlue" },
    { 0x008B8B, "DarkCyan" },
    { 0xB8860B, "DarkGoldenRod" },
    { 0xA9A9A9, "DarkGray" },
    { 0x006400, "DarkGreen" },
    { 0xBDB76B, "DarkKhaki" },
    { 0x8B008B, "DarkMagenta" },
    { 0x556B2F, "DarkOliveGreen" },
    { 0xFF8C00, "DarkOrange" },
    { 0x9932CC, "DarkOrchid" },
    { 0x8B0000, "DarkRed" },
    { 0xE9967A, "DarkSalmon" },
    { 0x8FBC8F, "DarkSeaGreen" },
    { 0x483D8B, "DarkSlateBlue" },
    { 0x2F4F4F, "DarkSlateGray" },
    { 0x00CED1, "DarkTurquoise" },
    { 0x9400D3, "DarkViolet" },
    { 0xFF1493, "DeepPink" },
    { 0x00BFFF, "DeepSkyBlue" },
    { 0x696969, "DimGray" },
    { 0x1E90FF, "DodgerBlue" },
    { 0xB22222, "FireBrick" },
    { 0xFFFAF0, "FloralWhite" },
    { 0x228B22, "ForestGreen" },
    { 0xDCDCDC, "Gainsboro" },
    { 0xF8F8FF, "GhostWhite" },
    { 0xFFD700, "Gold" },
    { 0xDAA520, "GoldenRod" },
    { 0x808080, "Gray" },
    { 0x008000, "Green" },
    { 0xADFF2F, "GreenYellow" },
    { 0xF0FFF0, "HoneyDew" },
    { 0xFF69B4, "HotPink" },
    { 0xCD5C5C, "IndianRed" },
    { 0x4B0082, "Indigo" },
    { 0xFFFFF0, "Ivory" },
    { 0xF0E68C, "Khaki" },
    { 0xE6E6FA, "Lavender" },
    { 0xFFF0F5, "LavenderBlush" },
    { 0x7CFC00, "LawnGreen" },
    { 0xFFFACD, "LemonChiffon" },
    { 0xADD8E6, "LightBlue" },
    { 0xF08080, "LightCoral" },
    { 0xE0FFFF, "LightCyan" },
    { 0xFAFAD2, "LightGoldenRodYellow" },
    { 0xD3D3D3, "LightGray" },
    { 0x90EE90, "LightGreen" },
    { 0xFFB6C1, "LightPink" },
    { 0xFFA07A, "LightSalmon" },
    { 0x20B2AA, "LightSeaGreen" },
    { 0x87CEFA, "LightSkyBlue" },
    { 0x778899, "LightSlateGray" },
    { 0xB0C4DE, "LightSteelBlue" },
    { 0xFFFFE0, "LightYellow" },
    { 0x00FF00, "Lime" },
    { 0x32CD32, "LimeGreen" },
    { 0xFAF0E6, "Linen" },
    { 0xFF00FF, "Magenta" },
    { 0x800000, "Maroon" },
    { 0x66CDAA, "MediumAquaMarine" },
    { 0x0000CD, "MediumBlue" },
    { 0xBA55D3, "MediumOrchid" },
    { 0x9370DB, "MediumPurple" },
    { 0x3CB371, "MediumSeaGreen" },
    { 0x7B68EE, "MediumSlateBlue" },
    { 0x00FA9A, "MediumSpringGreen" },
    { 0x48D1CC, "MediumTurquoise" },
    { 0xC71585, "MediumVioletRed" },
    { 0x191970, "MidnightBlue" },
    { 0xF5FFFA, "MintCream" },
    { 0xFFE4E1, "MistyRose" },
    { 0xFFE4B5, "Moccasin" },
    { 0xFFDEAD, "NavajoWhite" },
    { 0x000080, "Navy" },
    { 0xFDF5E6, "OldLace" },
    { 0x808000, "Olive" },
    { 0x6B8E23, "OliveDrab" },
    { 0xFFA500, "Orange" },
    { 0xFF4500, "OrangeRed" },
    { 0xDA70D6, "Orchid" },
    { 0xEEE8AA, "PaleGoldenRod" },
    { 0x98FB98, "PaleGreen" },
    { 0xAFEEEE, "PaleTurquoise" },
    { 0xDB7093, "PaleVioletRed" },
    { 0xFFEFD5, "PapayaWhip" },
    { 0xFFDAB9, "PeachPuff" },
    { 0xCD853F, "Peru" },
    { 0xFFC0CB, "Pink" },
    { 0xDDA0DD, "Plum" },
    { 0xB0E0E6, "PowderBlue" },
    { 0x800080, "Purple" },
    { 0xFF0000, "Red" },
    { 0xBC8F8F, "RosyBrown" },
    { 0x4169E1, "RoyalBlue" },
    { 0x8B4513, "SaddleBrown" },
    { 0xFA8072, "Salmon" },
    { 0xF4A460, "SandyBrown" },
    { 0x2E8B57, "SeaGreen" },
    { 0xFFF5EE, "SeaShell" },
    { 0xA0522D, "Sienna" },
    { 0xC0C0C0, "Silver" },
    { 0x87CEEB, "SkyBlue" },
    { 0x6A5ACD, "SlateBlue" },
    { 0x708090, "SlateGray" },
    { 0xFFFAFA, "Snow" },
    { 0x00FF7F, "SpringGreen" },
    { 0x4682B4, "SteelBlue" },
    { 0xD2B48C, "Tan" },
    { 0x008080, "Teal" },
    { 0xD8BFD8, "Thistle" },
    { 0xFF6347, "Tomato" },
    { 0x40E0D0, "Turquoise" },
    { 0xEE82EE, "Violet" },
    { 0xF5DEB3, "Wheat" },
    { 0xFFFFFF, "White" },
    { 0xF5F5F5, "WhiteSmoke" },
    { 0xFFFF00, "Yellow" },
    { 0x9ACD32, "YellowGreen" },
  };
  public static final Map<Integer,String> MAP =
    Arrays.stream(ARRAY)
      .collect(Collectors.toMap(
        element -> (Integer)element[0],
        element -> (String)element[1],
        (v1, v2) -> { // Merge function
          throw new IllegalStateException();
        },
        LinkedHashMap::new
      ));
  // Inversion only works if values are unique:
  public static <V, K> Map<V, K>
  invert(Map<K, V> map) {
    return map.entrySet().stream()
      .collect(Collectors.toMap(
        Map.Entry::getValue,
        Map.Entry::getKey,
        (v1, v2) -> {
          throw new IllegalStateException();
        },
        LinkedHashMap::new
      ));
  }
  public static final Map<String,Integer>
    INVMAP = invert(MAP);
  // Look up RGB value given a name:
  public static Integer rgb(String colorName) {
    return INVMAP.get(colorName);
  }
  public static final List<String> LIST =
    Arrays.stream(ARRAY)
      .map(item -> (String)item[1])
      .collect(Collectors.toList());
  public static final List<Integer> RGBLIST =
    Arrays.stream(ARRAY)
      .map(item -> (Integer)item[0])
      .collect(Collectors.toList());
  public static
  void show(Map.Entry<Integer,String> e) {
    System.out.format(
      "0x%06X: %s%n", e.getKey(), e.getValue());
  }
  public static void
  show(Map<Integer,String> m, int count) {
    m.entrySet().stream()
      .limit(count)
      .forEach(e -> show(e));
  }
  public static void show(Map<Integer,String> m) {
    show(m, m.size());
  }
  public static
  void show(Collection<String> lst, int count) {
    lst.stream()
      .limit(count)
      .forEach(System.out::println);
  }
  public static void show(Collection<String> lst) {
    show(lst, lst.size());
  }
  public static
  void showrgb(Collection<Integer> lst, int count) {
    lst.stream()
      .limit(count)
      .forEach(n -> System.out.format("0x%06X%n", n));
  }
  public static void showrgb(Collection<Integer> lst) {
    showrgb(lst, lst.size());
  }
  public static
  void showInv(Map<String,Integer> m, int count) {
    m.entrySet().stream()
      .limit(count)
      .forEach(e ->
        System.out.format(
          "%-20s  0x%06X%n", e.getKey(), e.getValue()));
  }
  public static void showInv(Map<String,Integer> m) {
    showInv(m, m.size());
  }
  public static void border() {
    System.out.println(
      "******************************");
  }
}
```

**MAP** 是使用Streams（[第十四章 流式編程]()）建立的。 二維陣列 **ARRAY** 作為流傳輸到 **Map** 中，但請注意我們不僅僅是使用簡單版本的 `Collectors.toMap()` 。 那個版本生成一個 **HashMap** ，它使用散列函數來控制對鍵的排序。 為了保留原來的順序，我們必須將鍵值對直接放入 **TreeMap** 中，這意味著我們需要使用更複雜的 `Collectors.toMap()` 版本。這需要兩個函數從每個流元素中提取鍵和值，就像簡單版本的`Collectors.toMap()` 一樣。 然後它需要一個*合併函數*（merge function），它解決了與同一個鍵相關的兩個值之間的衝突。這裡的資料已經預先審查過，因此絕不會發生這種情況，如果有的話，這裡會拋出異常。最後，傳遞生成所需類型的空map的函數，然後用流來填充它。

`rgb()` 方法是一個便捷函數（convenience function），它接受顏色名稱 **String** 參數並生成其數字RGB值。為此，我們需要一個反轉版本的 **COLORS** ，它接受一個 **String**鍵並尋找RGB的 **Integer** 值。 這是透過 `invert()` 方法實現的，如果任何 **COLORS** 值不唯一，則拋出異常。

我們還建立包含所有名稱的 **LIST** ，以及包含十六進位制表示法的RGB值的 **RGBLIST** 。

第一個 `show()` 方法接受一個 **Map.Entry** 並顯示以十六進位制表示的鍵，以便輕鬆地對原始 **ARRAY** 進行雙重檢查。 名稱以 **show** 開頭的每個方法都會重載兩個版本，其中一個版本採用 **count** 參數來指示要顯示的元素數量，第二個版本顯示序列中的所有元素。

這裡是一個基本的測試：

```java
// collectiontopics/HTMLColorTest.java
import static onjava.HTMLColors.*;

public class HTMLColorTest {
  static final int DISPLAY_SIZE = 20;
  public static void main(String[] args) {
    show(MAP, DISPLAY_SIZE);
    border();
    showInv(INVMAP, DISPLAY_SIZE);
    border();
    show(LIST, DISPLAY_SIZE);
    border();
    showrgb(RGBLIST, DISPLAY_SIZE);
  }
}
/* Output:
0xF0F8FF: AliceBlue
0xFAEBD7: AntiqueWhite
0x7FFFD4: Aquamarine
0xF0FFFF: Azure
0xF5F5DC: Beige
0xFFE4C4: Bisque
0x000000: Black
0xFFEBCD: BlanchedAlmond
0x0000FF: Blue
0x8A2BE2: BlueViolet
0xA52A2A: Brown
0xDEB887: BurlyWood
0x5F9EA0: CadetBlue
0x7FFF00: Chartreuse
0xD2691E: Chocolate
0xFF7F50: Coral
0x6495ED: CornflowerBlue
0xFFF8DC: Cornsilk
0xDC143C: Crimson
0x00FFFF: Cyan
******************************
AliceBlue             0xF0F8FF
AntiqueWhite          0xFAEBD7
Aquamarine            0x7FFFD4
Azure                 0xF0FFFF
Beige                 0xF5F5DC
Bisque                0xFFE4C4
Black                 0x000000
BlanchedAlmond        0xFFEBCD
Blue                  0x0000FF
BlueViolet            0x8A2BE2
Brown                 0xA52A2A
BurlyWood             0xDEB887
CadetBlue             0x5F9EA0
Chartreuse            0x7FFF00
Chocolate             0xD2691E
Coral                 0xFF7F50
CornflowerBlue        0x6495ED
Cornsilk              0xFFF8DC
Crimson               0xDC143C
Cyan                  0x00FFFF
******************************
AliceBlue
AntiqueWhite
Aquamarine
Azure
Beige
Bisque
Black
BlanchedAlmond
Blue
BlueViolet
Brown
BurlyWood
CadetBlue
Chartreuse
Chocolate
Coral
CornflowerBlue
Cornsilk
Crimson
Cyan
******************************
0xF0F8FF
0xFAEBD7
0x7FFFD4
0xF0FFFF
0xF5F5DC
0xFFE4C4
0x000000
0xFFEBCD
0x0000FF
0x8A2BE2
0xA52A2A
0xDEB887
0x5F9EA0
0x7FFF00
0xD2691E
0xFF7F50
0x6495ED
0xFFF8DC
0xDC143C
0x00FFFF
*/
```

可以看到，使用 **LinkedHashMap** 確實能夠保留 **HTMLColors.ARRAY** 的順序。

<!-- List Behavior -->
## List行為

**Lists** 是儲存和檢索物件（次於陣列）的最基本方法。基本列表操作包括：

- `add()` 用於插入元素
- `get()` 用於隨機訪問元素
- `iterator()` 獲取序列上的一個 **Iterator**
- `stream()` 生成元素的一個 **Stream**

列表構造方法始終保留元素的添加順序。

以下範例中的方法各自涵蓋了一組不同的行為：每個 **List** 可以執行的操作（ `basicTest()` ），使用 **Iterator** （ `iterMotion()` ）遍歷序列，使用 **Iterator** （ `iterManipulation()` ）更改內容，查看 **List** 操作（ `testVisual()` ）的效果，以及僅可用於 **LinkedLists** 的操作：

```java
// collectiontopics/ListOps.java
// Things you can do with Lists
import java.util.*;
import onjava.HTMLColors;

public class ListOps {
  // Create a short list for testing:
  static final List<String> LIST =
    HTMLColors.LIST.subList(0, 10);
  private static boolean b;
  private static String s;
  private static int i;
  private static Iterator<String> it;
  private static ListIterator<String> lit;
  public static void basicTest(List<String> a) {
    a.add(1, "x"); // Add at location 1
    a.add("x"); // Add at end
    // Add a collection:
    a.addAll(LIST);
    // Add a collection starting at location 3:
    a.addAll(3, LIST);
    b = a.contains("1"); // Is it in there?
    // Is the entire collection in there?
    b = a.containsAll(LIST);
    // Lists allow random access, which is cheap
    // for ArrayList, expensive for LinkedList:
    s = a.get(1); // Get (typed) object at location 1
    i = a.indexOf("1"); // Tell index of object
    b = a.isEmpty(); // Any elements inside?
    it = a.iterator(); // Ordinary Iterator
    lit = a.listIterator(); // ListIterator
    lit = a.listIterator(3); // Start at location 3
    i = a.lastIndexOf("1"); // Last match
    a.remove(1); // Remove location 1
    a.remove("3"); // Remove this object
    a.set(1, "y"); // Set location 1 to "y"
    // Keep everything that's in the argument
    // (the intersection of the two sets):
    a.retainAll(LIST);
    // Remove everything that's in the argument:
    a.removeAll(LIST);
    i = a.size(); // How big is it?
    a.clear(); // Remove all elements
  }
  public static void iterMotion(List<String> a) {
    ListIterator<String> it = a.listIterator();
    b = it.hasNext();
    b = it.hasPrevious();
    s = it.next();
    i = it.nextIndex();
    s = it.previous();
    i = it.previousIndex();
  }
  public static void iterManipulation(List<String> a) {
    ListIterator<String> it = a.listIterator();
    it.add("47");
    // Must move to an element after add():
    it.next();
    // Remove the element after the new one:
    it.remove();
    // Must move to an element after remove():
    it.next();
    // Change the element after the deleted one:
    it.set("47");
  }
  public static void testVisual(List<String> a) {
    System.out.println(a);
    List<String> b = LIST;
    System.out.println("b = " + b);
    a.addAll(b);
    a.addAll(b);
    System.out.println(a);
    // Insert, remove, and replace elements
    // using a ListIterator:
    ListIterator<String> x =
      a.listIterator(a.size()/2);
    x.add("one");
    System.out.println(a);
    System.out.println(x.next());
    x.remove();
    System.out.println(x.next());
    x.set("47");
    System.out.println(a);
    // Traverse the list backwards:
    x = a.listIterator(a.size());
    while(x.hasPrevious())
      System.out.print(x.previous() + " ");
    System.out.println();
    System.out.println("testVisual finished");
  }
  // There are some things that only LinkedLists can do:
  public static void testLinkedList() {
    LinkedList<String> ll = new LinkedList<>();
    ll.addAll(LIST);
    System.out.println(ll);
    // Treat it like a stack, pushing:
    ll.addFirst("one");
    ll.addFirst("two");
    System.out.println(ll);
    // Like "peeking" at the top of a stack:
    System.out.println(ll.getFirst());
    // Like popping a stack:
    System.out.println(ll.removeFirst());
    System.out.println(ll.removeFirst());
    // Treat it like a queue, pulling elements
    // off the tail end:
    System.out.println(ll.removeLast());
    System.out.println(ll);
  }
  public static void main(String[] args) {
    // Make and fill a new list each time:
    basicTest(new LinkedList<>(LIST));
    basicTest(new ArrayList<>(LIST));
    iterMotion(new LinkedList<>(LIST));
    iterMotion(new ArrayList<>(LIST));
    iterManipulation(new LinkedList<>(LIST));
    iterManipulation(new ArrayList<>(LIST));
    testVisual(new LinkedList<>(LIST));
    testLinkedList();
  }
}
/* Output:
[AliceBlue, AntiqueWhite, Aquamarine, Azure, Beige,
Bisque, Black, BlanchedAlmond, Blue, BlueViolet]
b = [AliceBlue, AntiqueWhite, Aquamarine, Azure, Beige,
Bisque, Black, BlanchedAlmond, Blue, BlueViolet]
[AliceBlue, AntiqueWhite, Aquamarine, Azure, Beige,
Bisque, Black, BlanchedAlmond, Blue, BlueViolet,
AliceBlue, AntiqueWhite, Aquamarine, Azure, Beige,
Bisque, Black, BlanchedAlmond, Blue, BlueViolet,
AliceBlue, AntiqueWhite, Aquamarine, Azure, Beige,
Bisque, Black, BlanchedAlmond, Blue, BlueViolet]
[AliceBlue, AntiqueWhite, Aquamarine, Azure, Beige,
Bisque, Black, BlanchedAlmond, Blue, BlueViolet,
AliceBlue, AntiqueWhite, Aquamarine, Azure, Beige, one,
Bisque, Black, BlanchedAlmond, Blue, BlueViolet,
AliceBlue, AntiqueWhite, Aquamarine, Azure, Beige,
Bisque, Black, BlanchedAlmond, Blue, BlueViolet]
Bisque
Black
[AliceBlue, AntiqueWhite, Aquamarine, Azure, Beige,
Bisque, Black, BlanchedAlmond, Blue, BlueViolet,
AliceBlue, AntiqueWhite, Aquamarine, Azure, Beige, one,
47, BlanchedAlmond, Blue, BlueViolet, AliceBlue,
AntiqueWhite, Aquamarine, Azure, Beige, Bisque, Black,
BlanchedAlmond, Blue, BlueViolet]
BlueViolet Blue BlanchedAlmond Black Bisque Beige Azure
Aquamarine AntiqueWhite AliceBlue BlueViolet Blue
BlanchedAlmond 47 one Beige Azure Aquamarine
AntiqueWhite AliceBlue BlueViolet Blue BlanchedAlmond
Black Bisque Beige Azure Aquamarine AntiqueWhite
AliceBlue
testVisual finished
[AliceBlue, AntiqueWhite, Aquamarine, Azure, Beige,
Bisque, Black, BlanchedAlmond, Blue, BlueViolet]
[two, one, AliceBlue, AntiqueWhite, Aquamarine, Azure,
Beige, Bisque, Black, BlanchedAlmond, Blue, BlueViolet]
two
two
one
BlueViolet
[AliceBlue, AntiqueWhite, Aquamarine, Azure, Beige,
Bisque, Black, BlanchedAlmond, Blue]
*/
```

在 `basicTest()` 和 `iterMotion()` 中，方法呼叫是為了展示正確的語法，儘管獲取了返回值，但不會使用它。在某些情況下，根本不會去獲取返回值。在使用這些方法之前，請查看JDK文件中這些方法的完整用法。

<!-- Set Behavior -->
## Set行為

**Set** 的主要用處是測試成員身份，不過也可以將其用作刪除重複元素的工具。如果不關心元素順序或並發性， **HashSet** 總是最好的選擇，因為它是專門為了快速尋找而設計的（這裡使用了在[附錄：理解equals和hashCode方法]()章節中探討的散列函數）。

其它的 **Set** 實現產生不同的排序行為：

```java
// collectiontopics/SetOrder.java
import java.util.*;
import onjava.HTMLColors;

public class SetOrder {
  static String[] sets = {
    "java.util.HashSet",
    "java.util.TreeSet",
    "java.util.concurrent.ConcurrentSkipListSet",
    "java.util.LinkedHashSet",
    "java.util.concurrent.CopyOnWriteArraySet",
  };
  static final List<String> RLIST =
    new ArrayList<>(HTMLColors.LIST);
  static {
    Collections.reverse(RLIST);
  }
  public static void
  main(String[] args) throws Exception {
    for(String type: sets) {
      System.out.format("[-> %s <-]%n",
        type.substring(type.lastIndexOf('.') + 1));
      @SuppressWarnings("unchecked")
      Set<String> set = (Set<String>)
        Class.forName(type).newInstance();
      set.addAll(RLIST);
      set.stream()
        .limit(10)
        .forEach(System.out::println);
    }
  }
}
/* Output:
[-> HashSet <-]
MediumOrchid
PaleGoldenRod
Sienna
LightSlateGray
DarkSeaGreen
Black
Gainsboro
Orange
LightCoral
DodgerBlue
[-> TreeSet <-]
AliceBlue
AntiqueWhite
Aquamarine
Azure
Beige
Bisque
Black
BlanchedAlmond
Blue
BlueViolet
[-> ConcurrentSkipListSet <-]
AliceBlue
AntiqueWhite
Aquamarine
Azure
Beige
Bisque
Black
BlanchedAlmond
Blue
BlueViolet
[-> LinkedHashSet <-]
YellowGreen
Yellow
WhiteSmoke
White
Wheat
Violet
Turquoise
Tomato
Thistle
Teal
[-> CopyOnWriteArraySet <-]
YellowGreen
Yellow
WhiteSmoke
White
Wheat
Violet
Turquoise
Tomato
Thistle
Teal
*/
```

這裡需要使用 **@SuppressWarnings(“unchecked”)** ，因為這裡將一個 **String** （可能是任何東西）傳遞給了 `Class.forName(type).newInstance()` 。編譯器並不能保證這是一次成功的操作。

**RLIST** 是 **HTMLColors.LIST** 的反轉版本。因為 `Collections.reverse()` 是透過修改參數來執行反向操作，而不是返回包含反向元素的新 **List** ，所以該呼叫在 **static** 塊內執行。  **RLIST** 可以防止我們意外地認為 **Set** 對其結果進行了排序。

**HashSet** 的輸出結果似乎沒有可辨別的順序，因為它是基於散列函數的。 **TreeSet** 和 **ConcurrentSkipListSet** 都對它們的元素進行了排序，它們都實現了 **SortedSet** 介面來標識這個特點。因為實現該介面的 **Set** 按順序排列，所以該介面還有一些其他的可用操作。 **LinkedHashSet** 和 **CopyOnWriteArraySet** 儘管沒有用於標識的介面，但它們還是保留了元素的插入順序。

**ConcurrentSkipListSet** 和 **CopyOnWriteArraySet** 是執行緒安全的。

在附錄的最後，我們將了解在非 **HashSet** 實現的 **Set** 上添加額外排序的效能成本，以及不同實現中的任何其他功能的成本。

<!-- Using Functional Operations with any Map -->
## 在Map中使用函數式操作

與 **Collection** 介面一樣，`forEach()` 也內建在 **Map** 介面中。但是如果想要執行任何其他的基本功能操作，比如 `map()` ，`flatMap()` ，`reduce()` 或 `filter()` 時，該怎麼辦？ 查看 **Map** 介面發現並沒有這些。

可以透過 `entrySet()` 連接到這些方法，該方法會生成一個由 **Map.Entry** 物件組成的 **Set** 。這個 **Set** 包含 `stream()` 和 `parallelStream()` 方法。只需要記住一件事，這裡正在使用的是 **Map.Entry** 物件：

```java
// collectiontopics/FunctionalMap.java
// Functional operations on a Map
import java.util.*;
import java.util.stream.*;
import java.util.concurrent.*;
import static onjava.HTMLColors.*;

public class FunctionalMap {
  public static void main(String[] args) {
    MAP.entrySet().stream()
      .map(Map.Entry::getValue)
      .filter(v -> v.startsWith("Dark"))
      .map(v -> v.replaceFirst("Dark", "Hot"))
      .forEach(System.out::println);
  }
}
/* Output:
HotBlue
HotCyan
HotGoldenRod
HotGray
HotGreen
HotKhaki
HotMagenta
HotOliveGreen
HotOrange
HotOrchid
HotRed
HotSalmon
HotSeaGreen
HotSlateBlue
HotSlateGray
HotTurquoise
HotViolet
*/
```

生成 **Stream** 後，所有的基本功能方法，甚至更多就都可以使用了。

<!-- Selecting Parts of a Map -->
## 選擇Map片段

由 **TreeMap** 和 **ConcurrentSkipListMap** 實現的 **NavigableMap** 介面解決了需要選擇Map片段的問題。下面是一個範例，使用了 **HTMLColors** ：

```java
// collectiontopics/NavMap.java
// NavigableMap produces pieces of a Map
import java.util.*;
import java.util.concurrent.*;
import static onjava.HTMLColors.*;

public class NavMap {
  public static final
  NavigableMap<Integer,String> COLORS =
    new ConcurrentSkipListMap<>(MAP);
  public static void main(String[] args) {
    show(COLORS.firstEntry());
    border();
    show(COLORS.lastEntry());
    border();
    NavigableMap<Integer, String> toLime =
      COLORS.headMap(rgb("Lime"), true);
    show(toLime);
    border();
    show(COLORS.ceilingEntry(rgb("DeepSkyBlue") - 1));
    border();
    show(COLORS.floorEntry(rgb("DeepSkyBlue") - 1));
    border();
    show(toLime.descendingMap());
    border();
    show(COLORS.tailMap(rgb("MistyRose"), true));
    border();
    show(COLORS.subMap(
      rgb("Orchid"), true,
      rgb("DarkSalmon"), false));
  }
}
/* Output:
0x000000: Black
******************************
0xFFFFFF: White
******************************
0x000000: Black
0x000080: Navy
0x00008B: DarkBlue
0x0000CD: MediumBlue
0x0000FF: Blue
0x006400: DarkGreen
0x008000: Green
0x008080: Teal
0x008B8B: DarkCyan
0x00BFFF: DeepSkyBlue
0x00CED1: DarkTurquoise
0x00FA9A: MediumSpringGreen
0x00FF00: Lime
******************************
0x00BFFF: DeepSkyBlue
******************************
0x008B8B: DarkCyan
******************************
0x00FF00: Lime
0x00FA9A: MediumSpringGreen
0x00CED1: DarkTurquoise
0x00BFFF: DeepSkyBlue
0x008B8B: DarkCyan
0x008080: Teal
0x008000: Green
0x006400: DarkGreen
0x0000FF: Blue
0x0000CD: MediumBlue
0x00008B: DarkBlue
0x000080: Navy
0x000000: Black
******************************
0xFFE4E1: MistyRose
0xFFEBCD: BlanchedAlmond
0xFFEFD5: PapayaWhip
0xFFF0F5: LavenderBlush
0xFFF5EE: SeaShell
0xFFF8DC: Cornsilk
0xFFFACD: LemonChiffon
0xFFFAF0: FloralWhite
0xFFFAFA: Snow
0xFFFF00: Yellow
0xFFFFE0: LightYellow
0xFFFFF0: Ivory
0xFFFFFF: White
******************************
0xDA70D6: Orchid
0xDAA520: GoldenRod
0xDB7093: PaleVioletRed
0xDC143C: Crimson
0xDCDCDC: Gainsboro
0xDDA0DD: Plum
0xDEB887: BurlyWood
0xE0FFFF: LightCyan
0xE6E6FA: Lavender
*/
```

在主方法中可以看到 **NavigableMap** 的各種功能。 因為 **NavigableMap** 具有鍵順序，所以它使用了 `firstEntry()` 和 `lastEntry()` 的概念。呼叫 `headMap()` 會生成一個 **NavigableMap** ，其中包含了從 **Map** 的開頭到 `headMap()` 參數中所指向的一組元素，其中 **boolean** 值指示結果中是否包含該參數。呼叫 `tailMap()` 執行了類似的操作，只不過是從參數開始到 **Map** 的末尾。 `subMap()` 則允許生成 **Map** 中間的一部分。

`ceilingEntry()` 從目前鍵值對向上搜尋下一個鍵值對，`floorEntry()` 則是向下搜尋。 `descendingMap()` 反轉了 **NavigableMap** 的順序。

如果需要透過分割 **Map** 來簡化所正在解決的問題，則 **NavigableMap** 可以做到。具有類似的功能的其它集合實現也可以用來幫助解決問題。

<!-- Filling Collections -->
## 填充集合

與 **Arrays** 一樣，這裡有一個名為 **Collections** 的伴隨類（companion class），包含了一些 **static** 的實用方法，其中包括一個名為 `fill()` 的方法。 `fill()` 只複製整個集合中的單個物件引用。此外，它僅適用於 **List** 物件，但結果列表可以傳遞給構造方法或 `addAll()` 方法：

```java
// collectiontopics/FillingLists.java
// Collections.fill() & Collections.nCopies()
import java.util.*;

class StringAddress {
  private String s;
  StringAddress(String s) { this.s = s; }
  @Override
  public String toString() {
    return super.toString() + " " + s;
  }
}

public class FillingLists {
  public static void main(String[] args) {
    List<StringAddress> list = new ArrayList<>(
      Collections.nCopies(4,
        new StringAddress("Hello")));
    System.out.println(list);
    Collections.fill(list,
      new StringAddress("World!"));
    System.out.println(list);
  }
}
/* Output:
[StringAddress@15db9742 Hello, StringAddress@15db9742
Hello, StringAddress@15db9742 Hello,
StringAddress@15db9742 Hello]
[StringAddress@6d06d69c World!, StringAddress@6d06d69c
World!, StringAddress@6d06d69c World!,
StringAddress@6d06d69c World!]
*/
```

這個範例展示了兩種使用對單個物件的引用來填充 **Collection** 的方法。 第一個： `Collections.nCopies()` ，建立一個 **List**，並傳遞給 **ArrayList** 的構造方法，進而填充了 **ArrayList** 。

**StringAddress** 中的 `toString()` 方法呼叫了 `Object.toString()` ，它先生成類名，後跟著物件的雜湊碼的無符號十六進位制表示（雜湊嗎由 `hashCode()` 方法生成）。 輸出顯示所有的引用都指向同一個物件。呼叫第二個方法 `Collections.fill()` 後也是如此。 `fill()` 方法的用處非常有限，它只能取代 **List** 中已有的元素,而且不會添加新元素，

### 使用 Suppliers 填充集合

[第二十章 泛型]()章節中介紹的 **onjava.Suppliers** 類為填充集合提供了通用解決方案。 這是一個使用 **Suppliers** 初始化幾種不同類型的 **Collection** 的範例：

```java
// collectiontopics/SuppliersCollectionTest.java
import java.util.*;
import java.util.function.*;
import java.util.stream.*;
import onjava.*;

class Government implements Supplier<String> {
  static String[] foundation = (
    "strange women lying in ponds " +
    "distributing swords is no basis " +
    "for a system of government").split(" ");
  private int index;
  @Override
  public String get() {
    return foundation[index++];
  }
}

public class SuppliersCollectionTest {
  public static void main(String[] args) {
    // Suppliers class from the Generics chapter:
    Set<String> set = Suppliers.create(
      LinkedHashSet::new, new Government(), 15);
    System.out.println(set);
    List<String> list = Suppliers.create(
      LinkedList::new, new Government(), 15);
    System.out.println(list);
    list = new ArrayList<>();
    Suppliers.fill(list, new Government(), 15);
    System.out.println(list);

    // Or we can use Streams:
    set = Arrays.stream(Government.foundation)
      .collect(Collectors.toSet());
    System.out.println(set);
    list = Arrays.stream(Government.foundation)
      .collect(Collectors.toList());
    System.out.println(list);
    list = Arrays.stream(Government.foundation)
      .collect(Collectors
        .toCollection(LinkedList::new));
    System.out.println(list);
    set = Arrays.stream(Government.foundation)
      .collect(Collectors
        .toCollection(LinkedHashSet::new));
    System.out.println(set);
  }
}
/* Output:
[strange, women, lying, in, ponds, distributing,
swords, is, no, basis, for, a, system, of, government]
[strange, women, lying, in, ponds, distributing,
swords, is, no, basis, for, a, system, of, government]
[strange, women, lying, in, ponds, distributing,
swords, is, no, basis, for, a, system, of, government]
[ponds, no, a, in, swords, for, is, basis, strange,
system, government, distributing, of, women, lying]
[strange, women, lying, in, ponds, distributing,
swords, is, no, basis, for, a, system, of, government]
[strange, women, lying, in, ponds, distributing,
swords, is, no, basis, for, a, system, of, government]
[strange, women, lying, in, ponds, distributing,
swords, is, no, basis, for, a, system, of, government]
*/
```

**LinkedHashSet** 中的的元素按插入順序排列，因為它維護一個鍊表來儲存該順序。

但是請注意範例的第二部分：大多數情況下都可以使用 **Stream** 來建立和填充 **Collection** 。在本例中的 **Stream** 版本不需要聲明 **Supplier** 所想要建立的元素數量;，它直接吸收了 **Stream** 中的所有元素。

儘可能優先選擇 **Stream** 來解決問題。

### Map Suppliers

使用 **Supplier** 來填充 **Map** 時需要一個 **Pair** 類，因為每次呼叫一個 **Supplier** 的 `get()` 方法時，都必須生成一對物件（一個鍵和一個值）：

```java
// onjava/Pair.java
package onjava;

public class Pair<K, V> {
  public final K key;
  public final V value;
  public Pair(K k, V v) {
    key = k;
    value = v;
  }
  public K key() { return key; }
  public V value() { return value; }
  public static <K,V> Pair<K, V> make(K k, V v) {
    return new Pair<K,V>(k, v);
  }
}
```

**Pair** 是一個唯讀的 *資料傳輸物件* （Data Transfer Object）或 *信使* （Messenger）。 這與[第二十章 泛型]()章節中的 **Tuple2** 基本相同，但名字更適合 **Map** 初始化。我還添加了靜態的 `make()` 方法，以便為建立 **Pair** 物件提供一個更簡潔的名字。

Java 8 的 **Stream** 提供了填充 **Map** 的便捷方法：

```java
// collectiontopics/StreamFillMaps.java
import java.util.*;
import java.util.function.*;
import java.util.stream.*;
import onjava.*;

class Letters
implements Supplier<Pair<Integer,String>> {
  private int number = 1;
  private char letter = 'A';
  @Override
  public Pair<Integer,String> get() {
    return new Pair<>(number++, "" + letter++);
  }
}

public class StreamFillMaps {
  public static void main(String[] args) {
    Map<Integer,String> m =
      Stream.generate(new Letters())
      .limit(11)
      .collect(Collectors
        .toMap(Pair::key, Pair::value));
    System.out.println(m);

    // Two separate Suppliers:
    Rand.String rs = new Rand.String(3);
    Count.Character cc = new Count.Character();
    Map<Character,String> mcs = Stream.generate(
      () -> Pair.make(cc.get(), rs.get()))
      .limit(8)
      .collect(Collectors
        .toMap(Pair::key, Pair::value));
    System.out.println(mcs);

    // A key Supplier and a single value:
    Map<Character,String> mcs2 = Stream.generate(
      () -> Pair.make(cc.get(), "Val"))
      .limit(8)
      .collect(Collectors
        .toMap(Pair::key, Pair::value));
    System.out.println(mcs2);
  }
}
/* Output:
{1=A, 2=B, 3=C, 4=D, 5=E, 6=F, 7=G, 8=H, 9=I, 10=J,
11=K}
{b=btp, c=enp, d=ccu, e=xsz, f=gvg, g=mei, h=nne,
i=elo}
{p=Val, q=Val, j=Val, k=Val, l=Val, m=Val, n=Val,
o=Val}
*/
```

上面的範例中出現了一個模式，可以使用它來建立一個自動建立和填充 **Map** 的工具：

```java
// onjava/FillMap.java
package onjava;
import java.util.*;
import java.util.function.*;
import java.util.stream.*;

public class FillMap {
  public static <K, V> Map<K,V>
  basic(Supplier<Pair<K,V>> pairGen, int size) {
    return Stream.generate(pairGen)
      .limit(size)
      .collect(Collectors
        .toMap(Pair::key, Pair::value));
  }
  public static <K, V> Map<K,V>
  basic(Supplier<K> keyGen,
        Supplier<V> valueGen, int size) {
    return Stream.generate(
      () -> Pair.make(keyGen.get(), valueGen.get()))
      .limit(size)
      .collect(Collectors
        .toMap(Pair::key, Pair::value));
  }
  public static <K, V, M extends Map<K,V>>
  M create(Supplier<K> keyGen,
           Supplier<V> valueGen,
           Supplier<M> mapSupplier, int size) {
    return Stream.generate( () ->
      Pair.make(keyGen.get(), valueGen.get()))
        .limit(size)
        .collect(Collectors
          .toMap(Pair::key, Pair::value,
                 (k, v) -> k, mapSupplier));
  }
}
```

basic() 方法生成一個預設的 **Map** ，而 `create()` 方法允許指定一個確切的 **Map** 類型，並返回那個確切的類型。

下面是一個測試：

```java
// collectiontopics/FillMapTest.java
import java.util.*;
import java.util.function.*;
import java.util.stream.*;
import onjava.*;

public class FillMapTest {
  public static void main(String[] args) {
    Map<String,Integer> mcs = FillMap.basic(
      new Rand.String(4), new Count.Integer(), 7);
    System.out.println(mcs);
    HashMap<String,Integer> hashm =
      FillMap.create(new Rand.String(4),
        new Count.Integer(), HashMap::new, 7);
    System.out.println(hashm);
    LinkedHashMap<String,Integer> linkm =
      FillMap.create(new Rand.String(4),
        new Count.Integer(), LinkedHashMap::new, 7);
    System.out.println(linkm);
  }
}
/* Output:
{npcc=1, ztdv=6, gvgm=3, btpe=0, einn=4, eelo=5,
uxsz=2}
{npcc=1, ztdv=6, gvgm=3, btpe=0, einn=4, eelo=5,
uxsz=2}
{btpe=0, npcc=1, uxsz=2, gvgm=3, einn=4, eelo=5,
ztdv=6}
*/
```

<!-- Custom Collection and Map using Flyweight -->
## 使用享元（Flyweight）自訂Collection和Map

本節介紹如何建立自訂 **Collection** 和 **Map** 實現。每個 **java.util** 中的集合都有自己的 **Abstract** 類，它提供了該集合的部分實現，因此只需要實現必要的方法來生成所需的集合。你將看到透過繼承 **java.util.Abstract** 類來建立自訂 **Map** 和 **Collection** 是多麼簡單。例如，要建立一個唯讀的 **Set** ，則可以從 **AbstractSet** 繼承並實現 `iterator()` 和 `size()` 。最後一個範例是生成測試資料的另一種方法。生成的集合通常是唯讀的，並且所提供的方法最少。

該解決方案還示範了 *享元* （Flyweight）設計模式。當普通解決方案需要太多物件時，或者當生成普通物件占用太多空間時，可以使用享元。享元設計模式將物件的一部分外部化（externalizes）。相比於把物件的所有內容都包含在物件中，這樣做使得物件的部分或者全部可以在更有效的外部表中尋找，或透過一些節省空間的其他計算生成。

下面是一個可以是任何大小的 **List** ，並且（有效地）使用 **Integer** 資料進行預初始化。要從 **AbstractList** 建立唯讀 **List** ，必須實現 `get()` 和 `size()`：

```java
// onjava/CountingIntegerList.java
// List of any length, containing sample data
// {java onjava.CountingIntegerList}
package onjava;
import java.util.*;

public class CountingIntegerList
extends AbstractList<Integer> {
  private int size;
  public CountingIntegerList() { size = 0; }
  public CountingIntegerList(int size) {
    this.size = size < 0 ? 0 : size;
  }
  @Override
  public Integer get(int index) {
    return index;
  }
  @Override
  public int size() { return size; }
  public static void main(String[] args) {
    List<Integer> cil =
      new CountingIntegerList(30);
    System.out.println(cil);
    System.out.println(cil.get(500));
  }
}
/* Output:
[0, 1, 2, 3, 4, 5, 6, 7, 8, 9, 10, 11, 12, 13, 14, 15,
16, 17, 18, 19, 20, 21, 22, 23, 24, 25, 26, 27, 28, 29]
500
*/
```

只有當想要限制 **List** 的長度時， **size** 值才是重要的，就像在主方法中那樣。即使在這種情況下， `get()` 也會產生任何值。

這個類是享元模式的一個簡潔的例子。當需要的時候， `get()` “計算”所需的值，因此沒必要儲存和初始化實際的底層 **List** 結構。

在大多數程式中，這裡所儲存的儲存結構永遠都不會改變。但是，它允許用非常大的 **index** 來呼叫 `List.get()` ，而 **List** 並不需要填充到這麼大。此外，還可以在程式中大量使用 **CountingIntegerLists** 而無需擔心儲存問題。實際上，享元的一個好處是它允許使用更好的抽象而不用擔心資源。

可以使用享元設計模式來實現具有任何大小資料集的其他“初始化”自訂集合。下面是一個 **Map** ，它為每一個 **Integer** 鍵產生唯一的值：

```java
// onjava/CountMap.java
// Unlimited-length Map containing sample data
// {java onjava.CountMap}
package onjava;
import java.util.*;
import java.util.stream.*;

public class CountMap
extends AbstractMap<Integer,String> {
  private int size;
  private static char[] chars =
    "ABCDEFGHIJKLMNOPQRSTUVWXYZ".toCharArray();
  private static String value(int key) {
    return
      chars[key % chars.length] +
      Integer.toString(key / chars.length);
  }
  public CountMap(int size) {
    this.size = size < 0 ? 0 : size;
  }
  @Override
  public String get(Object key) {
    return value((Integer)key);
  }
  private static class Entry
  implements Map.Entry<Integer,String> {
    int index;
    Entry(int index) { this.index = index; }
    @Override
    public boolean equals(Object o) {
      return o instanceof Entry &&
        Objects.equals(index, ((Entry)o).index);
    }
    @Override
    public Integer getKey() { return index; }
    @Override
    public String getValue() {
      return value(index);
    }
    @Override
    public String setValue(String value) {
      throw new UnsupportedOperationException();
    }
    @Override
    public int hashCode() {
      return Objects.hashCode(index);
    }
  }
  @Override
  public Set<Map.Entry<Integer,String>> entrySet() {
    // LinkedHashSet retains initialization order:
    return IntStream.range(0, size)
      .mapToObj(Entry::new)
      .collect(Collectors
        .toCollection(LinkedHashSet::new));
  }
  public static void main(String[] args) {
    final int size = 6;
    CountMap cm = new CountMap(60);
    System.out.println(cm);
    System.out.println(cm.get(500));
    cm.values().stream()
      .limit(size)
      .forEach(System.out::println);
    System.out.println();
    new Random(47).ints(size, 0, 1000)
      .mapToObj(cm::get)
      .forEach(System.out::println);
  }
}
/* Output:
{0=A0, 1=B0, 2=C0, 3=D0, 4=E0, 5=F0, 6=G0, 7=H0, 8=I0,
9=J0, 10=K0, 11=L0, 12=M0, 13=N0, 14=O0, 15=P0, 16=Q0,
17=R0, 18=S0, 19=T0, 20=U0, 21=V0, 22=W0, 23=X0, 24=Y0,
25=Z0, 26=A1, 27=B1, 28=C1, 29=D1, 30=E1, 31=F1, 32=G1,
33=H1, 34=I1, 35=J1, 36=K1, 37=L1, 38=M1, 39=N1, 40=O1,
41=P1, 42=Q1, 43=R1, 44=S1, 45=T1, 46=U1, 47=V1, 48=W1,
49=X1, 50=Y1, 51=Z1, 52=A2, 53=B2, 54=C2, 55=D2, 56=E2,
57=F2, 58=G2, 59=H2}
G19
A0
B0
C0
D0
E0
F0

Y9
J21
R26
D33
Z36
N16
*/
```

要建立一個唯讀的 **Map** ，則從 **AbstractMap** 繼承並實現 `entrySet()` 。私有的 `value()` 方法計算任何鍵的值，並在 `get()` 和 `Entry.getValue()` 中使用。可以忽略 **CountMap** 的大小。

這裡是使用了 **LinkedHashSet** 而不是建立自訂 **Set** 類，因此並未完全實現享元。只有在呼叫 `entrySet()` 時才會生成此物件。

現在建立一個更複雜的享元。這個範例中的資料集是世界各國及其首都的 **Map** 。 `capitals()` 方法生成一個國家和首都的 **Map** 。 `names()` 方法生成一個由國家名字組成的 **List** 。 當給定了表示所需大小的 **int** 參數時，兩種方法都生成對應大小的列表片段：

```java
// onjava/Countries.java
// "Flyweight" Maps and Lists of sample data
// {java onjava.Countries}
package onjava;
import java.util.*;

public class Countries {
  public static final String[][] DATA = {
    // Africa
    {"ALGERIA","Algiers"},
    {"ANGOLA","Luanda"},
    {"BENIN","Porto-Novo"},
    {"BOTSWANA","Gaberone"},
    {"BURKINA FASO","Ouagadougou"},
    {"BURUNDI","Bujumbura"},
    {"CAMEROON","Yaounde"},
    {"CAPE VERDE","Praia"},
    {"CENTRAL AFRICAN REPUBLIC","Bangui"},
    {"CHAD","N'djamena"},
    {"COMOROS","Moroni"},
    {"CONGO","Brazzaville"},
    {"DJIBOUTI","Dijibouti"},
    {"EGYPT","Cairo"},
    {"EQUATORIAL GUINEA","Malabo"},
    {"ERITREA","Asmara"},
    {"ETHIOPIA","Addis Ababa"},
    {"GABON","Libreville"},
    {"THE GAMBIA","Banjul"},
    {"GHANA","Accra"},
    {"GUINEA","Conakry"},
    {"BISSAU","Bissau"},
    {"COTE D'IVOIR (IVORY COAST)","Yamoussoukro"},
    {"KENYA","Nairobi"},
    {"LESOTHO","Maseru"},
    {"LIBERIA","Monrovia"},
    {"LIBYA","Tripoli"},
    {"MADAGASCAR","Antananarivo"},
    {"MALAWI","Lilongwe"},
    {"MALI","Bamako"},
    {"MAURITANIA","Nouakchott"},
    {"MAURITIUS","Port Louis"},
    {"MOROCCO","Rabat"},
    {"MOZAMBIQUE","Maputo"},
    {"NAMIBIA","Windhoek"},
    {"NIGER","Niamey"},
    {"NIGERIA","Abuja"},
    {"RWANDA","Kigali"},
    {"SAO TOME E PRINCIPE","Sao Tome"},
    {"SENEGAL","Dakar"},
    {"SEYCHELLES","Victoria"},
    {"SIERRA LEONE","Freetown"},
    {"SOMALIA","Mogadishu"},
    {"SOUTH AFRICA","Pretoria/Cape Town"},
    {"SUDAN","Khartoum"},
    {"SWAZILAND","Mbabane"},
    {"TANZANIA","Dodoma"},
    {"TOGO","Lome"},
    {"TUNISIA","Tunis"},
    {"UGANDA","Kampala"},
    {"DEMOCRATIC REPUBLIC OF THE CONGO (ZAIRE)",
     "Kinshasa"},
    {"ZAMBIA","Lusaka"},
    {"ZIMBABWE","Harare"},
    // Asia
    {"AFGHANISTAN","Kabul"},
    {"BAHRAIN","Manama"},
    {"BANGLADESH","Dhaka"},
    {"BHUTAN","Thimphu"},
    {"BRUNEI","Bandar Seri Begawan"},
    {"CAMBODIA","Phnom Penh"},
    {"CHINA","Beijing"},
    {"CYPRUS","Nicosia"},
    {"INDIA","New Delhi"},
    {"INDONESIA","Jakarta"},
    {"IRAN","Tehran"},
    {"IRAQ","Baghdad"},
    {"ISRAEL","Jerusalem"},
    {"JAPAN","Tokyo"},
    {"JORDAN","Amman"},
    {"KUWAIT","Kuwait City"},
    {"LAOS","Vientiane"},
    {"LEBANON","Beirut"},
    {"MALAYSIA","Kuala Lumpur"},
    {"THE MALDIVES","Male"},
    {"MONGOLIA","Ulan Bator"},
    {"MYANMAR (BURMA)","Rangoon"},
    {"NEPAL","Katmandu"},
    {"NORTH KOREA","P'yongyang"},
    {"OMAN","Muscat"},
    {"PAKISTAN","Islamabad"},
    {"PHILIPPINES","Manila"},
    {"QATAR","Doha"},
    {"SAUDI ARABIA","Riyadh"},
    {"SINGAPORE","Singapore"},
    {"SOUTH KOREA","Seoul"},
    {"SRI LANKA","Colombo"},
    {"SYRIA","Damascus"},
    {"TAIWAN (REPUBLIC OF CHINA)","Taipei"},
    {"THAILAND","Bangkok"},
    {"TURKEY","Ankara"},
    {"UNITED ARAB EMIRATES","Abu Dhabi"},
    {"VIETNAM","Hanoi"},
    {"YEMEN","Sana'a"},
    // Australia and Oceania
    {"AUSTRALIA","Canberra"},
    {"FIJI","Suva"},
    {"KIRIBATI","Bairiki"},
    {"MARSHALL ISLANDS","Dalap-Uliga-Darrit"},
    {"MICRONESIA","Palikir"},
    {"NAURU","Yaren"},
    {"NEW ZEALAND","Wellington"},
    {"PALAU","Koror"},
    {"PAPUA NEW GUINEA","Port Moresby"},
    {"SOLOMON ISLANDS","Honaira"},
    {"TONGA","Nuku'alofa"},
    {"TUVALU","Fongafale"},
    {"VANUATU","Port Vila"},
    {"WESTERN SAMOA","Apia"},
    // Eastern Europe and former USSR
    {"ARMENIA","Yerevan"},
    {"AZERBAIJAN","Baku"},
    {"BELARUS (BYELORUSSIA)","Minsk"},
    {"BULGARIA","Sofia"},
    {"GEORGIA","Tbilisi"},
    {"KAZAKSTAN","Almaty"},
    {"KYRGYZSTAN","Alma-Ata"},
    {"MOLDOVA","Chisinau"},
    {"RUSSIA","Moscow"},
    {"TAJIKISTAN","Dushanbe"},
    {"TURKMENISTAN","Ashkabad"},
    {"UKRAINE","Kyiv"},
    {"UZBEKISTAN","Tashkent"},
    // Europe
    {"ALBANIA","Tirana"},
    {"ANDORRA","Andorra la Vella"},
    {"AUSTRIA","Vienna"},
    {"BELGIUM","Brussels"},
    {"BOSNIA-HERZEGOVINA","Sarajevo"},
    {"CROATIA","Zagreb"},
    {"CZECH REPUBLIC","Prague"},
    {"DENMARK","Copenhagen"},
    {"ESTONIA","Tallinn"},
    {"FINLAND","Helsinki"},
    {"FRANCE","Paris"},
    {"GERMANY","Berlin"},
    {"GREECE","Athens"},
    {"HUNGARY","Budapest"},
    {"ICELAND","Reykjavik"},
    {"IRELAND","Dublin"},
    {"ITALY","Rome"},
    {"LATVIA","Riga"},
    {"LIECHTENSTEIN","Vaduz"},
    {"LITHUANIA","Vilnius"},
    {"LUXEMBOURG","Luxembourg"},
    {"MACEDONIA","Skopje"},
    {"MALTA","Valletta"},
    {"MONACO","Monaco"},
    {"MONTENEGRO","Podgorica"},
    {"THE NETHERLANDS","Amsterdam"},
    {"NORWAY","Oslo"},
    {"POLAND","Warsaw"},
    {"PORTUGAL","Lisbon"},
    {"ROMANIA","Bucharest"},
    {"SAN MARINO","San Marino"},
    {"SERBIA","Belgrade"},
    {"SLOVAKIA","Bratislava"},
    {"SLOVENIA","Ljuijana"},
    {"SPAIN","Madrid"},
    {"SWEDEN","Stockholm"},
    {"SWITZERLAND","Berne"},
    {"UNITED KINGDOM","London"},
    {"VATICAN CITY","Vatican City"},
    // North and Central America
    {"ANTIGUA AND BARBUDA","Saint John's"},
    {"BAHAMAS","Nassau"},
    {"BARBADOS","Bridgetown"},
    {"BELIZE","Belmopan"},
    {"CANADA","Ottawa"},
    {"COSTA RICA","San Jose"},
    {"CUBA","Havana"},
    {"DOMINICA","Roseau"},
    {"DOMINICAN REPUBLIC","Santo Domingo"},
    {"EL SALVADOR","San Salvador"},
    {"GRENADA","Saint George's"},
    {"GUATEMALA","Guatemala City"},
    {"HAITI","Port-au-Prince"},
    {"HONDURAS","Tegucigalpa"},
    {"JAMAICA","Kingston"},
    {"MEXICO","Mexico City"},
    {"NICARAGUA","Managua"},
    {"PANAMA","Panama City"},
    {"ST. KITTS AND NEVIS","Basseterre"},
    {"ST. LUCIA","Castries"},
    {"ST. VINCENT AND THE GRENADINES","Kingstown"},
    {"UNITED STATES OF AMERICA","Washington, D.C."},
    // South America
    {"ARGENTINA","Buenos Aires"},
    {"BOLIVIA","Sucre (legal)/La Paz(administrative)"},
    {"BRAZIL","Brasilia"},
    {"CHILE","Santiago"},
    {"COLOMBIA","Bogota"},
    {"ECUADOR","Quito"},
    {"GUYANA","Georgetown"},
    {"PARAGUAY","Asuncion"},
    {"PERU","Lima"},
    {"SURINAME","Paramaribo"},
    {"TRINIDAD AND TOBAGO","Port of Spain"},
    {"URUGUAY","Montevideo"},
    {"VENEZUELA","Caracas"},
  };
  // Use AbstractMap by implementing entrySet()
  private static class FlyweightMap
  extends AbstractMap<String,String> {
    private static class Entry
    implements Map.Entry<String,String> {
      int index;
      Entry(int index) { this.index = index; }
      @Override
      public boolean equals(Object o) {
        return o instanceof FlyweightMap &&
          Objects.equals(DATA[index][0], o);
      }
      @Override
      public int hashCode() {
        return Objects.hashCode(DATA[index][0]);
      }
      @Override
      public String getKey() { return DATA[index][0]; }
      @Override
      public String getValue() {
        return DATA[index][1];
      }
      @Override
      public String setValue(String value) {
        throw new UnsupportedOperationException();
      }
    }
    // Implement size() & iterator() for AbstractSet:
    static class EntrySet
    extends AbstractSet<Map.Entry<String,String>> {
      private int size;
      EntrySet(int size) {
        if(size < 0)
          this.size = 0;
        // Can't be any bigger than the array:
        else if(size > DATA.length)
          this.size = DATA.length;
        else
          this.size = size;
      }
      @Override
      public int size() { return size; }
      private class Iter
      implements Iterator<Map.Entry<String,String>> {
        // Only one Entry object per Iterator:
        private Entry entry = new Entry(-1);
        @Override
        public boolean hasNext() {
          return entry.index < size - 1;
        }
        @Override
        public Map.Entry<String,String> next() {
          entry.index++;
          return entry;
        }
        @Override
        public void remove() {
          throw new UnsupportedOperationException();
        }
      }
      @Override
      public
      Iterator<Map.Entry<String,String>> iterator() {
        return new Iter();
      }
    }
    private static
    Set<Map.Entry<String,String>> entries =
      new EntrySet(DATA.length);
    @Override
    public Set<Map.Entry<String,String>> entrySet() {
      return entries;
    }
  }
  // Create a partial map of 'size' countries:
  static Map<String,String> select(final int size) {
    return new FlyweightMap() {
      @Override
      public Set<Map.Entry<String,String>> entrySet() {
        return new EntrySet(size);
      }
    };
  }
  static Map<String,String> map = new FlyweightMap();
  public static Map<String,String> capitals() {
    return map; // The entire map
  }
  public static Map<String,String> capitals(int size) {
    return select(size); // A partial map
  }
  static List<String> names =
    new ArrayList<>(map.keySet());
  // All the names:
  public static List<String> names() { return names; }
  // A partial list:
  public static List<String> names(int size) {
    return new ArrayList<>(select(size).keySet());
  }
  public static void main(String[] args) {
    System.out.println(capitals(10));
    System.out.println(names(10));
    System.out.println(new HashMap<>(capitals(3)));
    System.out.println(
      new LinkedHashMap<>(capitals(3)));
    System.out.println(new TreeMap<>(capitals(3)));
    System.out.println(new Hashtable<>(capitals(3)));
    System.out.println(new HashSet<>(names(6)));
    System.out.println(new LinkedHashSet<>(names(6)));
    System.out.println(new TreeSet<>(names(6)));
    System.out.println(new ArrayList<>(names(6)));
    System.out.println(new LinkedList<>(names(6)));
    System.out.println(capitals().get("BRAZIL"));
  }
}
/* Output:
{ALGERIA=Algiers, ANGOLA=Luanda, BENIN=Porto-Novo,
BOTSWANA=Gaberone, BURKINA FASO=Ouagadougou,
BURUNDI=Bujumbura, CAMEROON=Yaounde, CAPE VERDE=Praia,
CENTRAL AFRICAN REPUBLIC=Bangui, CHAD=N'djamena}
[ALGERIA, ANGOLA, BENIN, BOTSWANA, BURKINA FASO,
BURUNDI, CAMEROON, CAPE VERDE, CENTRAL AFRICAN
REPUBLIC, CHAD]
{BENIN=Porto-Novo, ANGOLA=Luanda, ALGERIA=Algiers}
{ALGERIA=Algiers, ANGOLA=Luanda, BENIN=Porto-Novo}
{ALGERIA=Algiers, ANGOLA=Luanda, BENIN=Porto-Novo}
{ALGERIA=Algiers, ANGOLA=Luanda, BENIN=Porto-Novo}
[BENIN, BOTSWANA, ANGOLA, BURKINA FASO, ALGERIA,
BURUNDI]
[ALGERIA, ANGOLA, BENIN, BOTSWANA, BURKINA FASO,
BURUNDI]
[ALGERIA, ANGOLA, BENIN, BOTSWANA, BURKINA FASO,
BURUNDI]
[ALGERIA, ANGOLA, BENIN, BOTSWANA, BURKINA FASO,
BURUNDI]
[ALGERIA, ANGOLA, BENIN, BOTSWANA, BURKINA FASO,
BURUNDI]
Brasilia
*/
```

二維陣列 **String DATA** 是 **public** 的，因此可以在別處使用。 **FlyweightMap** 必須實現 `entrySet()` 方法，該方法需要一個自訂 **Set** 實現和一個自訂 **Map.Entry** 類。這是實現享元的另一種方法：每個 **Map.Entry** 物件儲存它自身的索引，而不是實際的鍵和值。當呼叫 `getKey()` 或 `getValue()` 時，它使用索引返回相應的 **DATA** 元素。 **EntrySet** 確保它的 **size** 不大於 **DATA** 。

享元的另一部分在 **EntrySet.Iterator** 中實現。相比於為 **DATA** 中的每個資料對建立一個 **Map.Entry** 物件，這裡每個疊代器只有一個 **Map.Entry** 物件。 **Entry** 物件作為資料的視窗，它只包含 **String** 靜態陣列的索引。每次為疊代器呼叫 `next()` 時，**Entry** 中的索引都會遞增，因此它會指向下一個資料對，然後從 `next()` 返回 **Iterators** 的單個 **Entry** 物件。[^1]

`select()` 方法生成一個包含所需大小的 **EntrySet** 的 **FlyweightMap** ，這用於在主方法中示範的重載的 `capitals()` 和 `names()` 方法。

<!-- Collection Functionality -->
## 集合功能

下面這個表格展示了可以對 **Collection** 執行的所有操作（不包括自動繼承自 **Object** 的方法），因此，可以用 **List** ， **Set** ， **Queue** 或 **Deque** 執行這裡的所有操作（這些介面可能也提供了一些其他的功能）。**Map** 不是從 **Collection** 繼承的，所以要單獨處理它。

| 方法名 | 描述 |
| :---: | :--- |
| **boolean add(T)** | 確保集合包含該泛型類型 **T** 的參數。如果不添加參數，則返回 **false** 。 （這是一種“可選”方法，將在下一節中介紹。） |
| **boolean addAll(Collection\<? extends T\>)** | 添加參數集合中的所有元素。只要有元素被成功添加則返回 **true**。（“可選的”） |
| **void clear()** | 刪除集合中的所有元素。（“可選的”） |
| **boolean contains(T)** | 如果目標集合包含該泛型類型 **T** 的參數，則返回 **true** 。 |
| **boolean containsAll(Collection\<?\>)** | 如果目標集合包含參數集合中的所有元素，則返回 **true** |
| **boolean isEmpty()** | 如果集合為空，則返回 **true** |
| **Iterator\<T\> iterator() Spliterator\<T\> spliterator()** | 返回一個疊代器來遍歷集合中的元素。 **Spliterators** 更複雜一些，它用在併發場景 |
| **boolean remove(Object)** | 如果目標集合包含該參數，則在集合中刪除該參數，如果成功刪除則返回 **true** 。（“可選的”） |
| **boolean removeAll(Collection\<?\>)** | 刪除目標集合中，參數集合所包含的全部元素。如果有元素被成功刪除則返回 **true** 。 （“可選的”） |
| **boolean removeIf(Predicate\<? super E\>)** | 刪除此集合中，滿足給定斷言（predicate）的所有元素 |
| **Stream\<E\> stream() Stream\<E\> parallelStream()** | 返回由該 **Collection** 中元素所組成的一個 **Stream** |
| **int size()** | 返回集合中所包含元素的個數 |
| **Object[] toArrat()** | 返回包含該集合所有元素的一個陣列 |
| **\<T\> T[] toArray(T[] a)** | 返回包含該集合所有元素的一個陣列。結果的執行時類型是參數陣列而不是普通的 **Object** 陣列。 |

這裡沒有提供用於隨機訪問元素的 **get()** 方法，因為 **Collection** 還包含 **Set** ，它維護自己的內部排序，所以隨機訪問尋找就沒有意義了。因此，要尋找 **Collection** 中的元素必須使用疊代器。

下面這個範例示範了 **Collection** 的所有方法。這裡以 **ArrayList** 為例：

```java
// collectiontopics/CollectionMethods.java
// Things you can do with all Collections
import java.util.*;
import static onjava.HTMLColors.*;

public class CollectionMethods {
  public static void main(String[] args) {
    Collection<String> c =
        new ArrayList<>(LIST.subList(0, 4));
    c.add("ten");
    c.add("eleven");
    show(c);
    border();
    // Make an array from the List:
    Object[] array = c.toArray();
    // Make a String array from the List:
    String[] str = c.toArray(new String[0]);
    // Find max and min elements; this means
    // different things depending on the way
    // the Comparable interface is implemented:
    System.out.println(
      "Collections.max(c) = " + Collections.max(c));
    System.out.println(
      "Collections.min(c) = " + Collections.min(c));
    border();
    // Add a Collection to another Collection
    Collection<String> c2 =
        new ArrayList<>(LIST.subList(10, 14));
    c.addAll(c2);
    show(c);
    border();
    c.remove(LIST.get(0));
    show(c);
    border();
    // Remove all components that are
    // in the argument collection:
    c.removeAll(c2);
    show(c);
    border();
    c.addAll(c2);
    show(c);
    border();
    // Is an element in this Collection?
    String val = LIST.get(3);
    System.out.println(
      "c.contains(" + val  + ") = " + c.contains(val));
    // Is a Collection in this Collection?
    System.out.println(
      "c.containsAll(c2) = " + c.containsAll(c2));
    Collection<String> c3 =
      ((List<String>)c).subList(3, 5);
    // Keep all the elements that are in both
    // c2 and c3 (an intersection of sets):
    c2.retainAll(c3);
    show(c2);
    // Throw away all the elements
    // in c2 that also appear in c3:
    c2.removeAll(c3);
    System.out.println(
      "c2.isEmpty() = " +  c2.isEmpty());
    border();
    // Functional operation:
    c = new ArrayList<>(LIST);
    c.removeIf(s -> !s.startsWith("P"));
    c.removeIf(s -> s.startsWith("Pale"));
    // Stream operation:
    c.stream().forEach(System.out::println);
    c.clear(); // Remove all elements
    System.out.println("after c.clear():" + c);
  }
}
/* Output:
AliceBlue
AntiqueWhite
Aquamarine
Azure
ten
eleven
******************************
Collections.max(c) = ten
Collections.min(c) = AliceBlue
******************************
AliceBlue
AntiqueWhite
Aquamarine
Azure
ten
eleven
Brown
BurlyWood
CadetBlue
Chartreuse
******************************
AntiqueWhite
Aquamarine
Azure
ten
eleven
Brown
BurlyWood
CadetBlue
Chartreuse
******************************
AntiqueWhite
Aquamarine
Azure
ten
eleven
******************************
AntiqueWhite
Aquamarine
Azure
ten
eleven
Brown
BurlyWood
CadetBlue
Chartreuse
******************************
c.contains(Azure) = true
c.containsAll(c2) = true
c2.isEmpty() = true
******************************
PapayaWhip
PeachPuff
Peru
Pink
Plum
PowderBlue
Purple
after c.clear():[]
*/
```

為了只示範 **Collection** 介面的方法，而沒有其它額外的內容，所以這裡建立包含不同資料集的 **ArrayList** ，並向上轉型為 **Collection** 物件。

<!-- Optional Operations -->
## 可選操作

在 **Collection** 介面中執行各種添加和刪除操作的方法是 *可選操作* （optional operations）。這意味著實現類不需要為這些方法提供功能定義。

這是一種非常不尋常的定義介面的方式。正如我們所知，介面是一種合約（contract）。它表達的意思是，“無論你如何選擇實現這個介面，我保證你可以將這些消息發送到這個物件”（我在這裡使用術語“介面”來描述正式的 **interface** 關鍵字和“任何類或子類都支援的方法”的更一般含義）。但“可選”操作違反了這一基本原則，它表示呼叫某些方法不會執行有意義的行為。相反，它們會拋出異常！這看起來似乎遺失了編譯時的類型安全性。

其實沒那麼糟糕。如果操作是可選的，編譯器仍然能夠限制你僅呼叫該介面中的方法。它不像動態語言那樣，可以為任何物件呼叫任何方法，並在執行時尋找特定的呼叫是否可行。[^2]此外，大多數將 **Collection** 作為參數的方法僅從該 **Collection** 中讀取，並且 **Collection** 的所有“讀取”方法都不是可選的。

為什麼要將方法定義為“可選”的？因為這樣做可以防止設計中的介面爆炸。集合庫的其他設計往往會產生令人困惑的過多介面來描述主題的每個變體。這甚至使得不可能捕獲到介面中的所有特殊情況，因為總有人能發明一個新的介面。“不支援的操作（unsupported operation）”這種方式實現了Java集合庫的一個重要目標：集合要易於學習和使用。不支援的操作是一種特殊情況，可以推遲到必要的時候。但是，要使用此方法：

1. **UnsupportedOperationException** 必須是一個罕見的事件。也就是說，對於大多數類，所有操作都應該起作用，並且只有在特殊情況下才應該不支援某項操作。這在Java集合庫中是正確的，因為99%的時間使用到的類 —— **ArrayList** ， **LinkedList** ， **HashSet** 和 **HashMap** ，以及其他具體實現，都支援所有操作。該設計確實為建立一個新的 **Collection** 提供了一個“後門”，可以不為 **Collection** 介面中的所有方法都提供有意義的定義，這些定義仍然適合現有的類庫。

2. 當不支援某個操作時， **UnsupportedOperationException** 應該出現在實現階段，而不是在將產品發送給客戶之後。畢竟，這個異常表示編程錯誤：錯誤地使用了一個具體實現。

值得注意的是，不支援的操作只能在執行時檢測到，因此這代表動態類型檢查。如果你來自像 C++ 這樣的靜態類型語言，Java 可能看起來只是另一種靜態類型語言。當然， Java 肯定有靜態類型檢查，但它也有大量的動態類型，因此很難說它只是靜態語言或動態語言。一旦你開始注意到這一點，你就會開始看到 Java 中動態類型檢查的其他範例。

<!-- Unsupported Operations -->
### 不支援的操作

不支援的操作的常見來源是由固定大小的資料結構所支援的集合。使用 `Arrays.asList()` 方法將陣列轉換為 **List** 時，就會得到這樣的集合。此外，還可以選擇使用 **Collections** 類中的“不可修改（unmodifiable）”方法使任何集合（包括 **Map** ）拋出 **UnsupportedOperationException** 異常。此範例展示了這兩種情況：

```java
// collectiontopics/Unsupported.java
// Unsupported operations in Java collections
import java.util.*;

public class Unsupported {
  static void
  check(String description, Runnable tst) {
    try {
      tst.run();
    } catch(Exception e) {
      System.out.println(description + "(): " + e);
    }
  }
  static void test(String msg, List<String> list) {
    System.out.println("--- " + msg + " ---");
    Collection<String> c = list;
    Collection<String> subList = list.subList(1,8);
    // Copy of the sublist:
    Collection<String> c2 = new ArrayList<>(subList);
    check("retainAll", () -> c.retainAll(c2));
    check("removeAll", () -> c.removeAll(c2));
    check("clear", () -> c.clear());
    check("add", () -> c.add("X"));
    check("addAll", () -> c.addAll(c2));
    check("remove", () -> c.remove("C"));
    // The List.set() method modifies the value but
    // doesn't change the size of the data structure:
    check("List.set", () -> list.set(0, "X"));
  }
  public static void main(String[] args) {
    List<String> list = Arrays.asList(
      "A B C D E F G H I J K L".split(" "));
    test("Modifiable Copy", new ArrayList<>(list));
    test("Arrays.asList()", list);
    test("unmodifiableList()",
      Collections.unmodifiableList(
        new ArrayList<>(list)));
  }
}
/* Output:
--- Modifiable Copy ---
--- Arrays.asList() ---
retainAll(): java.lang.UnsupportedOperationException
removeAll(): java.lang.UnsupportedOperationException
clear(): java.lang.UnsupportedOperationException
add(): java.lang.UnsupportedOperationException
addAll(): java.lang.UnsupportedOperationException
remove(): java.lang.UnsupportedOperationException
--- unmodifiableList() ---
retainAll(): java.lang.UnsupportedOperationException
removeAll(): java.lang.UnsupportedOperationException
clear(): java.lang.UnsupportedOperationException
add(): java.lang.UnsupportedOperationException
addAll(): java.lang.UnsupportedOperationException
remove(): java.lang.UnsupportedOperationException
List.set(): java.lang.UnsupportedOperationException
*/
```

因為 `Arrays.asList()` 生成的 **List** 由一個固定大小的陣列所支援，所以唯一支援的操作是那些不改變陣列大小的操作。任何會導致更改基礎資料結構大小的方法都會產生 **UnsupportedOperationException** 異常，來說明這是對不支援的方法的呼叫（編程錯誤）。

請注意，始終可以將 `Arrays.asList()` 的結果作為一個參數傳遞給任何 **Collection** 的構造方法（或使用 `addAll()` 方法或靜態的 `Collections.addAll()` 方法）來建立一個允許使用所有方法的一般集合，在主方法中第一次呼叫 `test()` 時顯示了這種情況。這種呼叫產生了一個新的可調整大小的底層資料結構。

**Collections** 類中的“unmodifiable”方法會將集合包裝一個代理中，如果執行任何想要修改集合的操作，則該代理會生成 **UnsupportedOperationException** 異常。使用這些方法的目的是生成一個“常量”集合物件。稍後將描述“unmodifiable“集合方法的完整列表。

`test()` 中的最後一個 `check()` 用於測試**List** 的 `set()` 方法。這裡，“不支援的操作”技術的粒度（granularity）就派上用場了，得到的“介面”可以透過一種方法在 `Arrays.asList()` 返回的物件和 `Collections.unmodifiableList()` 返回的物件之間變換。 `Arrays.asList()` 返回固定大小的 **List** ，而 `Collections.unmodifiableList()` 生成無法更改的 **List** 。如輸出中所示， `Arrays.asList()` 返回的 **List** 中的元素是可以修改的，因為這不會違反該 **List** 的“固定大小”特性。但很明顯， `unmodifiableList()` 的結果不應該以任何方式修改。如果使用介面來描述，則需要兩個額外的介面，一個具有可用的 `set()` 方法，而另一個沒有。 **Collection** 的各種不可修改的子類型都將需要額外的介面。

如果一個方法將一個集合作為它的參數，那麼它的文件應該說明必須實現哪些可選方法。

<!-- Sets and Storage Order -->
## Set和儲存順序

[第十二章 集合]()章節中的 **Set** 有關範例對 **Set** 的基本操作做了很好的介紹。 但是，這些範例可以方便地使用預定義的 Java 類型，例如 **Integer** 和 **String** ，它們可以在集合中使用。在建立自己的類型時請注意， **Set** （以及稍後會看到的 **Map** ）需要一種維護儲存順序的方法，該順序因 **Set** 的不同實現而異。因此，不同的 **Set** 實現不僅具有不同的行為，而且它們對可以放入特定 **Set** 中的物件類型也有不同的要求：

| **Set** 類型 | 約束 |
| :---: | :--- |
| **Set(interface)** | 添加到 **Set** 中的每個元素必須是唯一的，否則，**Set** 不會添加重複元素。添加到 **Set** 的元素必須至少定義 `equals()` 方法以建立物件唯一性。 **Set** 與 **Collection** 具有完全相同的介面。 **Set** 介面不保證它將以任何特定順序維護其元素。 |
| **HashSet\*** | 注重快速尋找元素的集合，其中元素必須定義 `hashCode()` 和 `equals()` 方法。 |
| **TreeSet** | 由樹支援的有序 **Set**。這樣，就可以從 **Set** 中獲取有序序列，其中元素必須實現 **Comparable** 介面。 |
| **LinkedHashSet** | 具有 **HashSet** 的尋找速度，但在內部使用鍊表維護元素的插入順序。因此，當在遍歷 **Set** 時，結果將按元素的插入順序顯示。元素必須定義 `hashCode()` 和 `equals()` 方法。 |

**HashSet** 上的星號表示，在沒有其他約束的情況下，這應該是你的預設選擇，因為它針對速度進行了最佳化。

定義 `hashCode()` 方法在[附錄:理解equals和hashCode方法]()中進行了描述。必須為散列和樹儲存結構建立 `equals()` 方法，但只有當把類放在 **HashSet** 中時才需要 `hashCode()` （當然這很有可能，因為 **HashSet** 通常應該是作為 **Set** 實現的首選）或 **LinkedHashSet** 。 但是，作為一種良好的程式風格，在覆蓋 `equals()` 時應始終覆蓋 `hashCode()` 。

下面的範例示範了成功使用具有特定 **Set** 實現的類型所需的方法：

```java
// collectiontopics/TypesForSets.java
// Methods necessary to put your own type in a Set
import java.util.*;
import java.util.function.*;
import java.util.Objects;

class SetType {
  protected int i;
  SetType(int n) { i = n; }
  @Override
  public boolean equals(Object o) {
    return o instanceof SetType &&
      Objects.equals(i, ((SetType)o).i);
  }
  @Override
  public String toString() {
    return Integer.toString(i);
  }
}

class HashType extends SetType {
  HashType(int n) { super(n); }
  @Override
  public int hashCode() {
    return Objects.hashCode(i);
  }
}

class TreeType extends SetType
implements Comparable<TreeType> {
  TreeType(int n) { super(n); }
  @Override
  public int compareTo(TreeType arg) {
    return Integer.compare(arg.i, i);
    // Equivalent to:
    // return arg.i < i ? -1 : (arg.i == i ? 0 : 1);
  }
}

public class TypesForSets {
  static <T> void
  fill(Set<T> set, Function<Integer, T> type) {
    for(int i = 10; i >= 5; i--) // Descending
      set.add(type.apply(i));
    for(int i = 0; i < 5; i++) // Ascending
      set.add(type.apply(i));
  }
  static <T> void
  test(Set<T> set, Function<Integer, T> type) {
    fill(set, type);
    fill(set, type); // Try to add duplicates
    fill(set, type);
    System.out.println(set);
  }
  public static void main(String[] args) {
    test(new HashSet<>(), HashType::new);
    test(new LinkedHashSet<>(), HashType::new);
    test(new TreeSet<>(), TreeType::new);
    // Things that don't work:
    test(new HashSet<>(), SetType::new);
    test(new HashSet<>(), TreeType::new);
    test(new LinkedHashSet<>(), SetType::new);
    test(new LinkedHashSet<>(), TreeType::new);
    try {
      test(new TreeSet<>(), SetType::new);
    } catch(Exception e) {
      System.out.println(e.getMessage());
    }
    try {
      test(new TreeSet<>(), HashType::new);
    } catch(Exception e) {
      System.out.println(e.getMessage());
    }
  }
}
/* Output:
[0, 1, 2, 3, 4, 5, 6, 7, 8, 9, 10]
[10, 9, 8, 7, 6, 5, 0, 1, 2, 3, 4]
[10, 9, 8, 7, 6, 5, 4, 3, 2, 1, 0]
[1, 6, 8, 6, 2, 7, 8, 9, 4, 10, 7, 5, 1, 3, 4, 9, 9,
10, 5, 3, 2, 0, 4, 1, 2, 0, 8, 3, 0, 10, 6, 5, 7]
[3, 1, 4, 8, 7, 6, 9, 5, 3, 0, 10, 5, 5, 10, 7, 8, 8,
9, 1, 4, 10, 2, 6, 9, 1, 6, 0, 3, 2, 0, 7, 2, 4]
[10, 9, 8, 7, 6, 5, 0, 1, 2, 3, 4, 10, 9, 8, 7, 6, 5,
0, 1, 2, 3, 4, 10, 9, 8, 7, 6, 5, 0, 1, 2, 3, 4]
[10, 9, 8, 7, 6, 5, 0, 1, 2, 3, 4, 10, 9, 8, 7, 6, 5,
0, 1, 2, 3, 4, 10, 9, 8, 7, 6, 5, 0, 1, 2, 3, 4]
SetType cannot be cast to java.lang.Comparable
HashType cannot be cast to java.lang.Comparable
*/
```

為了證明特定 **Set** 需要哪些方法，同時避免程式碼重複，這裡建立了三個類。基類 **SetType** 儲存一個 **int** 值，並透過 `toString()` 方法列印它。由於儲存在 **Set** 中的所有類都必須具有 `equals()` ，因此該方法也放在基類中。基於 `int i` 來判斷元素是否相等。

**HashType** 繼承自 **SetType** ，並添加了 `hashCode()` 方法，該方法對於 **Set** 的散列實現是必需的。

要在任何類型的有序集合中使用物件，由 **TreeType** 實現的 **Comparable** 介面都是必需的，例如 **SortedSet** （ **TreeSet** 是其唯一實現）。在 `compareTo()` 中，請注意我沒有使用“簡單明瞭”的形式： `return i-i2` 。雖然這是一個常見的程式錯誤，但只有當 **i** 和 **i2** 是“無符號（unsigned）”整型時才能正常工作（如果 Java 有一個“unsigned”關鍵字的話，不過它沒有）。它破壞了 Java 的有符號 **int** ，它不足以代表兩個有符號整數的差異。如果 **i** 是一個大的正整數而 **j** 是一個大的負整數， `i-j` 將溢位並返回一個負值，這不是我們所需要的。

通常希望 `compareTo()` 方法生成與 `equals()` 方法一致的自然順序。如果 `equals()` 對於特定比較產生 **true**，則 `compareTo()` 應該為該比較返回結果 零，並且如果 `equals()` 為比較產生 **false** ，則 `compareTo()` 應該為該比較產生非零結果。

在 **TypesForSets** 中， `fill()` 和 `test()` 都是使用泛型定義的，以防止程式碼重複。為了驗證 **Set** 的行為， `test()` 在測試集上呼叫 `fill()` 三次，嘗試引入重複的物件。 `fill()` 方法的參數可以接收任意一個 **Set** 類型，以及生成該類型的 **Function** 物件。因為此範例中使用的所有物件都有一個帶有單個 **int** 參數的構造方法，所以可以將構造方法作為此 **Function** 傳遞，它將提供用於填充 **Set** 的物件。

請注意， `fill()` 方法按降序添加前五個元素，按升序添加後五個元素，以此來指出生成的儲存順序。輸出顯示 **HashSet** 按升序保留元素，但是，在[附錄:理解equals和hashCode方法]()中，你會發現這只是偶然的，因為散列會建立自己的儲存順序。這裡只是因為元素是一個簡單的 **int** ，在這種情況下它是升序的。 **LinkedHashSet** 按照插入順序儲存元素，**TreeSet** 按排序順序維護元素（在此範例中因為 `compareTo()` 的實現方式，所以元素按降序排列。）

特定的 **Set** 類型一般都有所必需的操作，如果嘗試使用沒能正確支援這些操作的類型，那麼事情就會出錯。將沒有重新定義 `hashCode()` 方法的 **SetType** 或 **TreeType** 物件放入任何散列實現會導致重複值，因此違反了 **Set** 的主要契約。 這是相當令人不安的，因為這甚至不產生執行時錯誤。但是，預設的 `hashCode()` 是合法的，所以即使它是不正確的，這也是合法的行為。確保此類程式正確性的唯一可靠方法是將單元測試合併到構建系統中。

如果嘗試在 **TreeSet** 中使用沒有實現 **Comparable** 介面的類型，則會得到更明確的結果：當 **TreeSet** 嘗試將物件用作一個 **Comparable** 時，將會拋出異常。

<!-- SortedSet -->
### SortedSet

**SortedSet** 中的元素保證按排序規則順序， **SortedSet** 介面中的以下方法可以產生其他功能：

- `Comparator comparator()` ：生成用於此 **Set** 的**Comparator** 或 **null** 來用於自然排序。
- `Object first()` ：返回第一個元素。
- `Object last()` ：返回最後一個元素。
- `SortedSet subSet(fromElement，toElement)` ：使用 **fromElement** （包含）和 **toElement** （不包括）中的元素生成此 **Set** 的一個檢視。
- `SortedSet headSet(toElement)` ：使用順序在 **toElement** 之前的元素生成此 **Set** 的一個檢視。
- `SortedSet tailSet(fromElement)` ：使用順序在 **fromElement** 之後（包含 **fromElement** ）的元素生成此 **Set** 的一個檢視。

下面是一個簡單的示範：

```java
// collectiontopics/SortedSetDemo.java
import java.util.*;
import static java.util.stream.Collectors.*;

public class SortedSetDemo {
  public static void main(String[] args) {
    SortedSet<String> sortedSet =
      Arrays.stream(
        "one two three four five six seven eight"
        .split(" "))
        .collect(toCollection(TreeSet::new));
    System.out.println(sortedSet);
    String low = sortedSet.first();
    String high = sortedSet.last();
    System.out.println(low);
    System.out.println(high);
    Iterator<String> it = sortedSet.iterator();
    for(int i = 0; i <= 6; i++) {
      if(i == 3) low = it.next();
      if(i == 6) high = it.next();
      else it.next();
    }
    System.out.println(low);
    System.out.println(high);
    System.out.println(sortedSet.subSet(low, high));
    System.out.println(sortedSet.headSet(high));
    System.out.println(sortedSet.tailSet(low));
  }
}
/* Output:
[eight, five, four, one, seven, six, three, two]
eight
two
one
two
[one, seven, six, three]
[eight, five, four, one, seven, six, three]
[one, seven, six, three, two]
*/
```

注意， **SortedSet** 表示“根據物件的比較函數進行排序”，而不是“根據插入順序”。可以使用 **LinkedHashSet** 保留元素的插入順序。

<!-- Queues -->
## 佇列

有許多 **Queue** 實現，其中大多數是為並發應用程式設計的。許多實現都是透過排序行為而不是性能來區分的。這是一個涉及大多數 **Queue** 實現的基本範例，包括基於並發的佇列。佇列將元素從一端放入並從另一端取出：

```java
// collectiontopics/QueueBehavior.java
// Compares basic behavior
import java.util.*;
import java.util.stream.*;
import java.util.concurrent.*;

public class QueueBehavior {
  static Stream<String> strings() {
    return Arrays.stream(
      ("one two three four five six seven " +
       "eight nine ten").split(" "));
  }
  static void test(int id, Queue<String> queue) {
    System.out.print(id + ": ");
    strings().map(queue::offer).count();
    while(queue.peek() != null)
      System.out.print(queue.remove() + " ");
    System.out.println();
  }
  public static void main(String[] args) {
    int count = 10;
    test(1, new LinkedList<>());
    test(2, new PriorityQueue<>());
    test(3, new ArrayBlockingQueue<>(count));
    test(4, new ConcurrentLinkedQueue<>());
    test(5, new LinkedBlockingQueue<>());
    test(6, new PriorityBlockingQueue<>());
    test(7, new ArrayDeque<>());
    test(8, new ConcurrentLinkedDeque<>());
    test(9, new LinkedBlockingDeque<>());
    test(10, new LinkedTransferQueue<>());
    test(11, new SynchronousQueue<>());
  }
}
/* Output:
1: one two three four five six seven eight nine ten
2: eight five four nine one seven six ten three two
3: one two three four five six seven eight nine ten
4: one two three four five six seven eight nine ten
5: one two three four five six seven eight nine ten
6: eight five four nine one seven six ten three two
7: one two three four five six seven eight nine ten
8: one two three four five six seven eight nine ten
9: one two three four five six seven eight nine ten
10: one two three four five six seven eight nine ten
11:
*/
```

**Deque** 介面也繼承自 **Queue** 。 除優先度佇列外，**Queue** 按照元素的插入順序生成元素。 在此範例中，**SynchronousQueue** 不會產生任何結果，因為它是一個阻塞佇列，其中每個插入操作必須等待另一個執行緒執行相應的刪除操作，反之亦然。

<!-- Priority Queues -->
### 優先度佇列

考慮一個待辦事項列表，其中每個物件包含一個 **String** 以及主要和次要優先度值。透過實現 **Comparable** 介面來控制此待辦事項列表的順序：

```java
// collectiontopics/ToDoList.java
// A more complex use of PriorityQueue
import java.util.*;

class ToDoItem implements Comparable<ToDoItem> {
  private char primary;
  private int secondary;
  private String item;
  ToDoItem(String td, char pri, int sec) {
    primary = pri;
    secondary = sec;
    item = td;
  }
  @Override
  public int compareTo(ToDoItem arg) {
    if(primary > arg.primary)
      return +1;
    if(primary == arg.primary)
      if(secondary > arg.secondary)
        return +1;
      else if(secondary == arg.secondary)
        return 0;
    return -1;
  }
  @Override
  public String toString() {
    return Character.toString(primary) +
      secondary + ": " + item;
  }
}

class ToDoList {
  public static void main(String[] args) {
    PriorityQueue<ToDoItem> toDo =
      new PriorityQueue<>();
    toDo.add(new ToDoItem("Empty trash", 'C', 4));
    toDo.add(new ToDoItem("Feed dog", 'A', 2));
    toDo.add(new ToDoItem("Feed bird", 'B', 7));
    toDo.add(new ToDoItem("Mow lawn", 'C', 3));
    toDo.add(new ToDoItem("Water lawn", 'A', 1));
    toDo.add(new ToDoItem("Feed cat", 'B', 1));
    while(!toDo.isEmpty())
      System.out.println(toDo.remove());
  }
}
/* Output:
A1: Water lawn
A2: Feed dog
B1: Feed cat
B7: Feed bird
C3: Mow lawn
C4: Empty trash
*/
```

這展示了透過優先度佇列自動排序待辦事項。

<!-- Deque -->
### 雙端佇列

**Deque** （雙端佇列）就像一個佇列，但是可以從任一端添加和刪除元素。 Java 6為 **Deque** 添加了一個顯式介面。以下是對實現了 **Deque** 的類的最基本的 **Deque** 方法的測試：

```java
// collectiontopics/SimpleDeques.java
// Very basic test of Deques
import java.util.*;
import java.util.concurrent.*;
import java.util.function.*;

class CountString implements Supplier<String> {
  private int n = 0;
  CountString() {}
  CountString(int start) { n = start; }
  @Override
  public String get() {
    return Integer.toString(n++);
  }
}

public class SimpleDeques {
  static void test(Deque<String> deque) {
    CountString s1 = new CountString(),
                s2 = new CountString(20);
    for(int n = 0; n < 8; n++) {
        deque.offerFirst(s1.get());
        deque.offerLast(s2.get()); // Same as offer()
    }
    System.out.println(deque);
    String result = "";
    while(deque.size() > 0) {
      System.out.print(deque.peekFirst() + " ");
      result += deque.pollFirst() + " ";
      System.out.print(deque.peekLast() + " ");
      result += deque.pollLast() + " ";
    }
    System.out.println("\n" + result);
  }
  public static void main(String[] args) {
    int count = 10;
    System.out.println("LinkedList");
    test(new LinkedList<>());
    System.out.println("ArrayDeque");
    test(new ArrayDeque<>());
    System.out.println("LinkedBlockingDeque");
    test(new LinkedBlockingDeque<>(count));
    System.out.println("ConcurrentLinkedDeque");
    test(new ConcurrentLinkedDeque<>());
  }
}
/* Output:
LinkedList
[7, 6, 5, 4, 3, 2, 1, 0, 20, 21, 22, 23, 24, 25, 26,
27]
7 27 6 26 5 25 4 24 3 23 2 22 1 21 0 20
7 27 6 26 5 25 4 24 3 23 2 22 1 21 0 20
ArrayDeque
[7, 6, 5, 4, 3, 2, 1, 0, 20, 21, 22, 23, 24, 25, 26,
27]
7 27 6 26 5 25 4 24 3 23 2 22 1 21 0 20
7 27 6 26 5 25 4 24 3 23 2 22 1 21 0 20
LinkedBlockingDeque
[4, 3, 2, 1, 0, 20, 21, 22, 23, 24]
4 24 3 23 2 22 1 21 0 20
4 24 3 23 2 22 1 21 0 20
ConcurrentLinkedDeque
[7, 6, 5, 4, 3, 2, 1, 0, 20, 21, 22, 23, 24, 25, 26,
27]
7 27 6 26 5 25 4 24 3 23 2 22 1 21 0 20
7 27 6 26 5 25 4 24 3 23 2 22 1 21 0 20
*/
```

我只使用了 **Deque** 方法的“offer”和“poll”版本，因為當 **LinkedBlockingDeque** 的大小有限時，這些方法不會拋出異常。請注意， **LinkedBlockingDeque** 僅填充到它的限制大小為止，然後忽略額外的添加。

<!-- Understanding Maps -->
## 理解Map

正如在[第十二章 集合]()章節中所了解到的，**Map**（也稱為 *關聯陣列* ）維護鍵值關聯（對），因此可以使用鍵來尋找值。標準 Java 庫包含不同的 **Map** 基本實現，例如 **HashMap** ， **TreeMap** ， **LinkedHashMap** ， **WeakHashMap** ， **ConcurrentHashMap** 和 **IdentityHashMap** 。 它們都具有相同的基本 **Map** 介面，但它們的行為不同，包括效率，鍵值對的儲存順序和呈現順序，儲存物件的時間，如何在多執行緒程式中工作，以及如何確定鍵的相等性。 **Map** 介面的實現數量應該告訴你一些關於此工具重要性的訊息。

為了更深入地了解 **Map** ，學習如何構造關聯陣列會很有幫助。下面是一個非常簡單的實現：

```java
// collectiontopics/AssociativeArray.java
// Associates keys with values

public class AssociativeArray<K, V> {
  private Object[][] pairs;
  private int index;
  public AssociativeArray(int length) {
    pairs = new Object[length][2];
  }
  public void put(K key, V value) {
    if(index >= pairs.length)
      throw new ArrayIndexOutOfBoundsException();
    pairs[index++] = new Object[]{ key, value };
  }
  @SuppressWarnings("unchecked")
  public V get(K key) {
    for(int i = 0; i < index; i++)
      if(key.equals(pairs[i][0]))
        return (V)pairs[i][1];
    return null; // Did not find key
  }
  @Override
  public String toString() {
    StringBuilder result = new StringBuilder();
    for(int i = 0; i < index; i++) {
      result.append(pairs[i][0].toString());
      result.append(" : ");
      result.append(pairs[i][1].toString());
      if(i < index - 1)
        result.append("\n");
    }
    return result.toString();
  }
  public static void main(String[] args) {
    AssociativeArray<String,String> map =
      new AssociativeArray<>(6);
    map.put("sky", "blue");
    map.put("grass", "green");
    map.put("ocean", "dancing");
    map.put("tree", "tall");
    map.put("earth", "brown");
    map.put("sun", "warm");
    try {
      map.put("extra", "object"); // Past the end
    } catch(ArrayIndexOutOfBoundsException e) {
      System.out.println("Too many objects!");
    }
    System.out.println(map);
    System.out.println(map.get("ocean"));
  }
}
/* Output:
Too many objects!
sky : blue
grass : green
ocean : dancing
tree : tall
earth : brown
sun : warm
dancing
*/
```

關聯陣列中的基本方法是 `put()` 和 `get()` ，但為了便於顯示，重寫了 `toString()` 方法以列印鍵值對。為了顯示它的工作原理，主方法載入一個帶有字串對的 **AssociativeArray** 並列印生成的映射，然後呼叫其中一個值的 `get()` 方法。

要使用 `get()` 方法，可以傳入要尋找的 **key** ，它將生成相關聯的值作為結果，如果找不到則返回 **null** 。 `get()` 方法使用可能是效率最低的方法來定位值：從陣列的頭部開始並使用 `equals()` 來比較鍵。但這裡是側重於簡單，而不是效率。

這個版本很有啟發性，但它不是很有效，而且它只有一個固定的大小，這是不靈活的。幸運的是， **java.util** 中的那些 **Map** 沒有這些問題。

<!-- Performance -->
### 性能

性能是 **Map** 的基本問題，在 `get()` 中使用線性方法搜尋一個鍵時會非常慢。這就是 **HashMap** 要加速的地方。它使用一個稱為 *雜湊碼* 的特殊值來替代慢速搜尋一個鍵。雜湊碼是一種從相關物件中獲取一些訊息並將其轉換為該物件的“相對唯一” **int** 的方法。 `hashCode()` 是根類 **Object** 中的一個方法，因此所有 Java 物件都可以生成雜湊碼。 **HashMap** 獲取物件的 `hashCode()` 並使用它來快速搜尋鍵。這就使得效能有了顯著的提升。[^3]

以下是基本的 **Map** 實現。 **HashMap**上的星號表示，在沒有其他約束的情況下，這應該是你的預設選擇，因為它針對速度進行了最佳化。其他實現強調其他特性，因此不如 **HashMap** 快。

| **Map** 實現 | 描述 |
| :---: | :--- |
| **HashMap\*** | 基於雜湊表的實現。（使用此類來代替 **Hashtable** 。）為插入和定位鍵值對提供了常數時間性能。可以透過構造方法調整性能，這些構造方法允許你設定雜湊表的容量和裝填因子。 |
| **LinkedHashMap** | 與 **HashMap** 類似，但是當遍歷時，可以按插入順序或最近最少使用（LRU）順序獲取鍵值對。只比 **HashMap** 略慢，一個例外是在疊代時，由於其使用鍊表維護內部順序，所以會更快些。 |
| **TreeMap** | 基於紅黑樹的實現。當查看鍵或鍵值對時，它們按排序順序（由 **Comparable** 或 **Comparator** 確定）。 **TreeMap** 的側重點是按排序順序獲得結果。 **TreeMap** 是唯一使用 `subMap()` 方法的 **Map** ，它返回紅黑樹的一部分。 |
| **WeakHashMap** | 一種具有 *弱鍵*（weak keys） 的 **Map** ，為了解決某些類型的問題，它允許釋放 **Map** 所引用的物件。如果在 **Map** 外沒有對特定鍵的引用，則可以對該鍵進行垃圾回收。 |
| **ConcurrentHashMap** | 不使用同步鎖定的執行緒安全 **Map** 。這在[第二十四章 並發編程]() 一章中討論。 |
| **IdentityHashMap** | 使用 `==` 而不是 `equals()` 來比較鍵。僅用於解決特殊問題，不適用於一般用途。 |

散列是在 **Map** 中儲存元素的最常用方法。

**Map** 中使用的鍵的要求與 **Set** 中的元素的要求相同。可以在 **TypesForSets.java** 中看到這些。任何鍵必須具有 `equals()` 方法。如果鍵用於散列映射，則它還必須具有正確的 `hashCode()` 方法。如果鍵在 **TreeMap** 中使用，則必須實現 **Comparable** 介面。

以下範例使用先前定義的 **CountMap** 測試資料集顯示透過 **Map** 介面可用的操作：

```java
// collectiontopics/MapOps.java
// Things you can do with Maps
import java.util.concurrent.*;
import java.util.*;
import onjava.*;

public class MapOps {
  public static
  void printKeys(Map<Integer,String> map) {
    System.out.print("Size = " + map.size() + ", ");
    System.out.print("Keys: ");
    // Produce a Set of the keys:
    System.out.println(map.keySet());
  }
  public static
  void test(Map<Integer,String> map) {
    System.out.println(
      map.getClass().getSimpleName());
    map.putAll(new CountMap(25));
    // Map has 'Set' behavior for keys:
    map.putAll(new CountMap(25));
    printKeys(map);
    // Producing a Collection of the values:
    System.out.print("Values: ");
    System.out.println(map.values());
    System.out.println(map);
    System.out.println("map.containsKey(11): " +
      map.containsKey(11));
    System.out.println(
      "map.get(11): " + map.get(11));
    System.out.println("map.containsValue(\"F0\"): "
      + map.containsValue("F0"));
    Integer key = map.keySet().iterator().next();
    System.out.println("First key in map: " + key);
    map.remove(key);
    printKeys(map);
    map.clear();
    System.out.println(
      "map.isEmpty(): " + map.isEmpty());
    map.putAll(new CountMap(25));
    // Operations on the Set change the Map:
    map.keySet().removeAll(map.keySet());
    System.out.println(
      "map.isEmpty(): " + map.isEmpty());
  }
  public static void main(String[] args) {
    test(new HashMap<>());
    test(new TreeMap<>());
    test(new LinkedHashMap<>());
    test(new IdentityHashMap<>());
    test(new ConcurrentHashMap<>());
    test(new WeakHashMap<>());
  }
}
/* Output: (First 11 Lines)
HashMap
Size = 25, Keys: [0, 1, 2, 3, 4, 5, 6, 7, 8, 9, 10, 11,
12, 13, 14, 15, 16, 17, 18, 19, 20, 21, 22, 23, 24]
Values: [A0, B0, C0, D0, E0, F0, G0, H0, I0, J0, K0,
L0, M0, N0, O0, P0, Q0, R0, S0, T0, U0, V0, W0, X0, Y0]
{0=A0, 1=B0, 2=C0, 3=D0, 4=E0, 5=F0, 6=G0, 7=H0, 8=I0,
9=J0, 10=K0, 11=L0, 12=M0, 13=N0, 14=O0, 15=P0, 16=Q0,
17=R0, 18=S0, 19=T0, 20=U0, 21=V0, 22=W0, 23=X0, 24=Y0}
map.containsKey(11): true
map.get(11): L0
map.containsValue("F0"): true
First key in map: 0
Size = 24, Keys: [1, 2, 3, 4, 5, 6, 7, 8, 9, 10, 11,
12, 13, 14, 15, 16, 17, 18, 19, 20, 21, 22, 23, 24]
map.isEmpty(): true
map.isEmpty(): true
                  ...
*/
```

`printKeys()` 方法示範了如何生成 **Map** 的 **Collection** 檢視。 `keySet()` 方法生成一個由 **Map** 中的鍵組成的 **Set** 。 列印 `values()` 方法的結果會生成一個包含 **Map** 中所有值的 **Collection** 。（請注意，鍵必須是唯一的，但值可以包含重複項。）由於這些 **Collection** 由 **Map** 支援，因此 **Collection** 中的任何更改都會反映在所關聯的 **Map** 中。

程式的其餘部分提供了每個 **Map** 操作的簡單範例，並測試了每種基本類型的 **Map** 。

<!-- SortedMap -->
### SortedMap

使用 **SortedMap** （由 **TreeMap** 或 **ConcurrentSkipListMap** 實現），鍵保證按排序順序，這允許在 **SortedMap** 介面中使用這些方法來提供其他功能：

- `Comparator comparator()` ：生成用於此 **Map** 的比較器， **null** 表示自然排序。
- `T firstKey()` ：返回第一個鍵。
- `T lastKey()` ：返回最後一個鍵。
- `SortedMap subMap(fromKey，toKey)` ：生成此 **Map** 的檢視，其中鍵從 **fromKey**（包括），到 **toKey** （不包括）。
- `SortedMap headMap(toKey)` ：使用小於 **toKey** 的鍵生成此 **Map** 的檢視。
- `SortedMap tailMap(fromKey)` ：使用大於或等於 **fromKey** 的鍵生成此 **Map** 的檢視。

這是一個類似於 **SortedSetDemo.java** 的範例，顯示了 **TreeMap** 的這種額外行為：

```java
// collectiontopics/SortedMapDemo.java
// What you can do with a TreeMap
import java.util.*;
import onjava.*;

public class SortedMapDemo {
  public static void main(String[] args) {
    TreeMap<Integer,String> sortedMap =
      new TreeMap<>(new CountMap(10));
    System.out.println(sortedMap);
    Integer low = sortedMap.firstKey();
    Integer high = sortedMap.lastKey();
    System.out.println(low);
    System.out.println(high);
    Iterator<Integer> it =
      sortedMap.keySet().iterator();
    for(int i = 0; i <= 6; i++) {
      if(i == 3) low = it.next();
      if(i == 6) high = it.next();
      else it.next();
    }
    System.out.println(low);
    System.out.println(high);
    System.out.println(sortedMap.subMap(low, high));
    System.out.println(sortedMap.headMap(high));
    System.out.println(sortedMap.tailMap(low));
  }
}
/* Output:
{0=A0, 1=B0, 2=C0, 3=D0, 4=E0, 5=F0, 6=G0, 7=H0, 8=I0,
9=J0}
0
9
3
7
{3=D0, 4=E0, 5=F0, 6=G0}
{0=A0, 1=B0, 2=C0, 3=D0, 4=E0, 5=F0, 6=G0}
{3=D0, 4=E0, 5=F0, 6=G0, 7=H0, 8=I0, 9=J0}
*/
```

這裡，鍵值對按照鍵的排序順序進行排序。因為 **TreeMap** 中存在順序感，所以“位置”的概念很有意義，因此可以擁有第一個、最後一個元素或子圖。

<!-- LinkedHashMap -->
### LinkedHashMap

**LinkedHashMap** 針對速度進行雜湊處理，但在遍歷期間也會按插入順序生成鍵值對（ `System.out.println()` 可以遍歷它，因此可以看到遍歷的結果）。 此外，可以在構造方法中配置 **LinkedHashMap** 以使用基於訪問的 *最近最少使用*（LRU） 演算法，因此未訪問的元素（因此是刪除的候選者）會出現在列表的前面。 這樣可以輕鬆建立一個能夠定期清理以節省空間的程式。下面是一個顯示這兩個功能的簡單範例：

```java
// collectiontopics/LinkedHashMapDemo.java
// What you can do with a LinkedHashMap
import java.util.*;
import onjava.*;

public class LinkedHashMapDemo {
  public static void main(String[] args) {
    LinkedHashMap<Integer,String> linkedMap =
      new LinkedHashMap<>(new CountMap(9));
    System.out.println(linkedMap);
    // Least-recently-used order:
    linkedMap =
      new LinkedHashMap<>(16, 0.75f, true);
    linkedMap.putAll(new CountMap(9));
    System.out.println(linkedMap);
    for(int i = 0; i < 6; i++)
      linkedMap.get(i);
    System.out.println(linkedMap);
    linkedMap.get(0);
    System.out.println(linkedMap);
  }
}
/* Output:
{0=A0, 1=B0, 2=C0, 3=D0, 4=E0, 5=F0, 6=G0, 7=H0, 8=I0}
{0=A0, 1=B0, 2=C0, 3=D0, 4=E0, 5=F0, 6=G0, 7=H0, 8=I0}
{6=G0, 7=H0, 8=I0, 0=A0, 1=B0, 2=C0, 3=D0, 4=E0, 5=F0}
{6=G0, 7=H0, 8=I0, 1=B0, 2=C0, 3=D0, 4=E0, 5=F0, 0=A0}
*/
```

這些鍵值對確實是按照插入順序進行遍歷，即使對於LRU版本也是如此。 但是，在LRU版本中訪問前六項（僅限）後，最後三項將移至列表的前面。然後，當再次訪問“ **0** ”後，它移動到了列表的後面。

<!-- Utilities -->
## 集合工具類

集合有許多獨立的實用工具程式，在 **java.util.Collections** 中表示為靜態方法。之前已經見過其中一些，例如 `addAll()` ， `reverseOrder()` 和 `binarySearch()` 。以下是其他內容（同步和不可修改的實用工具程式將在後面的章節中介紹）。在此表中，在需要的時候使用了泛型：

| 方法 | 描述 |
| :--- | :--- |
| **checkedCollection(Collection\<T> c, Class\<T> type)** <br><br> **checkedList(List\<T> list, Class\<T> type)** <br><br> **checkedMap(Map\<K, V> m, Class\<K> keyType, Class\<V> valueType)** <br><br> **checkedSet(Set\<T> s, Class\<T> type)** <br><br> **checkedSortedMap(SortedMap\<K, V> m, Class\<K> keyType, Class\<V> valueType)** <br><br> **checkedSortedSet(SortedSet\<T> s, Class\<T> type)** | 生成 **Collection** 的動態類型安全檢視或 **Collection** 的特定子類型。 當無法使用靜態檢查版本時使用這個版本。 <br><br> 這些方法的使用在[第九章 多態]()章節的“動態類型安全”標題下進行了展示。 |
| **max(Collection)** <br><br> **min(Collection)** | 使用 **Collection** 中物件的自然比較方法生成參數集合中的最大或最小元素。 |
| **max(Collection, Comparator)** <br><br> **min(Collection, Comparator)** | 使用 **Comparator** 指定的比較方法生成參數集合中的最大或最小元素。 |
| **indexOfSubList(List source, List target)** | 返回 **target** 在 **source** 內第一次出現的起始索引，如果不存在則返回 -1。 |
| **lastIndexOfSubList(List source, List target)** | 返回 **target** 在 **source** 內最後一次出現的起始索引，如果不存在則返回 -1。 |
| **replaceAll(List\<T> list, T oldVal, T newVal)** | 用 **newVal** 取代列表中所有的 **oldVal** 。 |
| **reverse(List)**| 反轉列表 |
| **reverseOrder()** <br><br> **reverseOrder(Comparator\<T>)** | 返回一個 **Comparator** ，它與集合中實現了 **comparable\<T>** 介面的物件的自然順序相反。第二個版本顛倒了所提供的 **Comparator** 的順序。 |
| **rotate(List, int distance)** | 將所有元素向前移動 **distance** ，將尾部的元素移到開頭。（譯者註：即循環移動） |
| **shuffle(List)** <br><br> **shuffle(List, Random)**  | 隨機置換指定列表（即打亂順序）。第一個版本使用了預設的隨機化源，或者也可以使用第二個版本，提供自己的隨機化源。 |
| **sort(List\<T>)** <br><br> **sort(List\<T>, Comparator\<? super T> c)** | 第一個版本使用元素的自然順序排序該 **List\<T>** 。第二個版本根據提供的 **Comparator** 排序。 |
| **copy(List\<? super T> dest, List\<? extends T> src)** | 將 **src** 中的元素複製到 **dest** 。 |
| **swap(List, int i, int j)** | 交換 **List** 中位置 **i** 和 位置 **j** 的元素。可能比你手工編寫的速度快。 |
| **fill(List\<? super T>, T x)** | 用 **x** 取代 **List** 中的所有元素。|
| **nCopies(int n, T x)** | 返回大小為 **n** 的不可變 **List\<T>** ，其引用都指向 **x** 。 |
| **disjoint(Collection, Collection)** | 如果兩個集合沒有共同元素，則返回 **true** 。 |
| **frequency(Collection, Object x)** | 返回 **Collection** 中，等於 **x** 的元素個數。 |
| **emptyList()** <br><br> **emptyMap()** <br><br> **emptySet()** | 返回不可變的空 **List** ， **Map** 或 **Set** 。這些是泛型的，因此生成的 **Collection** 可以被參數化為所需的類型。 |
| **singleton(T x)** <br><br> **singletonList(T x)** <br><br> **singletonMap(K key, V value)** | 生成一個不可變的 **List** ， **Set** 或 **Map** ，其中只包含基於給定參數的單個元素。 |
| **list(Enumeration\<T> e)** | 生成一個 **ArrayList\<T>** ，其中元素為（舊式） **Enumeration** （ **Iterator** 的前身）中的元素。用於從遺留程式碼向新式轉換。 |
| **enumeration(Collection\<T>)** | 為參數集合生成一個舊式的 **Enumeration\<T>** 。 |

請注意， `min（)` 和 `max()` 使用 **Collection** 物件，而不使用 **List** ，因此不必擔心是否應對 **Collection** 進行排序。（如前所述，在執行 `binarySearch()` 之前，將會對 **List** 或陣列進行`sort()` 排序。）

下面是一個範例，展示了上表中大多數實用工具程式的基本用法：

```java
// collectiontopics/Utilities.java
// Simple demonstrations of the Collections utilities
import java.util.*;

public class Utilities {
  static List<String> list = Arrays.asList(
    "one Two three Four five six one".split(" "));
  public static void main(String[] args) {
    System.out.println(list);
    System.out.println("'list' disjoint (Four)?: " +
      Collections.disjoint(list,
        Collections.singletonList("Four")));
    System.out.println(
      "max: " + Collections.max(list));
    System.out.println(
      "min: " + Collections.min(list));
    System.out.println(
      "max w/ comparator: " + Collections.max(list,
      String.CASE_INSENSITIVE_ORDER));
    System.out.println(
      "min w/ comparator: " + Collections.min(list,
      String.CASE_INSENSITIVE_ORDER));
    List<String> sublist =
      Arrays.asList("Four five six".split(" "));
    System.out.println("indexOfSubList: " +
      Collections.indexOfSubList(list, sublist));
    System.out.println("lastIndexOfSubList: " +
      Collections.lastIndexOfSubList(list, sublist));
    Collections.replaceAll(list, "one", "Yo");
    System.out.println("replaceAll: " + list);
    Collections.reverse(list);
    System.out.println("reverse: " + list);
    Collections.rotate(list, 3);
    System.out.println("rotate: " + list);
    List<String> source =
      Arrays.asList("in the matrix".split(" "));
    Collections.copy(list, source);
    System.out.println("copy: " + list);
    Collections.swap(list, 0, list.size() - 1);
    System.out.println("swap: " + list);
    Collections.shuffle(list, new Random(47));
    System.out.println("shuffled: " + list);
    Collections.fill(list, "pop");
    System.out.println("fill: " + list);
    System.out.println("frequency of 'pop': " +
      Collections.frequency(list, "pop"));
    List<String> dups =
      Collections.nCopies(3, "snap");
    System.out.println("dups: " + dups);
    System.out.println("'list' disjoint 'dups'?: " +
      Collections.disjoint(list, dups));
    // Getting an old-style Enumeration:
    Enumeration<String> e =
      Collections.enumeration(dups);
    Vector<String> v = new Vector<>();
    while(e.hasMoreElements())
      v.addElement(e.nextElement());
    // Converting an old-style Vector
    // to a List via an Enumeration:
    ArrayList<String> arrayList =
      Collections.list(v.elements());
    System.out.println("arrayList: " + arrayList);
  }
}
/* Output:
[one, Two, three, Four, five, six, one]
'list' disjoint (Four)?: false
max: three
min: Four
max w/ comparator: Two
min w/ comparator: five
indexOfSubList: 3
lastIndexOfSubList: 3
replaceAll: [Yo, Two, three, Four, five, six, Yo]
reverse: [Yo, six, five, Four, three, Two, Yo]
rotate: [three, Two, Yo, Yo, six, five, Four]
copy: [in, the, matrix, Yo, six, five, Four]
swap: [Four, the, matrix, Yo, six, five, in]
shuffled: [six, matrix, the, Four, Yo, five, in]
fill: [pop, pop, pop, pop, pop, pop, pop]
frequency of 'pop': 7
dups: [snap, snap, snap]
'list' disjoint 'dups'?: true
arrayList: [snap, snap, snap]
*/
```

輸出解釋了每種實用方法的行為。請注意由於大小寫的緣故，普通版本的 `min()` 和 `max()` 與帶有 **String.CASE_INSENSITIVE_ORDER** 比較器參數的版本的區別。

<!-- Sorting and Searching Lists -->
### 排序和搜尋列表

用於執行排序和搜尋 **List** 的實用工具程式與用於排序物件陣列的程式具有相同的名字和方法簽名，只不過是 **Collections** 的靜態方法而不是 **Arrays** 。 這是一個使用 **Utilities.java** 中的 **list** 資料的範例：

```java
// collectiontopics/ListSortSearch.java
// Sorting/searching Lists with Collections utilities
import java.util.*;

public class ListSortSearch {
  public static void main(String[] args) {
    List<String> list =
      new ArrayList<>(Utilities.list);
    list.addAll(Utilities.list);
    System.out.println(list);
    Collections.shuffle(list, new Random(47));
    System.out.println("Shuffled: " + list);
    // Use ListIterator to trim off last elements:
    ListIterator<String> it = list.listIterator(10);
    while(it.hasNext()) {
      it.next();
      it.remove();
    }
    System.out.println("Trimmed: " + list);
    Collections.sort(list);
    System.out.println("Sorted: " + list);
    String key = list.get(7);
    int index = Collections.binarySearch(list, key);
    System.out.println(
      "Location of " + key + " is " + index +
      ", list.get(" + index + ") = " +
      list.get(index));
    Collections.sort(list,
      String.CASE_INSENSITIVE_ORDER);
    System.out.println(
      "Case-insensitive sorted: " + list);
    key = list.get(7);
    index = Collections.binarySearch(list, key,
      String.CASE_INSENSITIVE_ORDER);
    System.out.println(
      "Location of " + key + " is " + index +
      ", list.get(" + index + ") = " +
      list.get(index));
  }
}
/* Output:
[one, Two, three, Four, five, six, one, one, Two,
three, Four, five, six, one]
Shuffled: [Four, five, one, one, Two, six, six, three,
three, five, Four, Two, one, one]
Trimmed: [Four, five, one, one, Two, six, six, three,
three, five]
Sorted: [Four, Two, five, five, one, one, six, six,
three, three]
Location of six is 7, list.get(7) = six
Case-insensitive sorted: [five, five, Four, one, one,
six, six, three, three, Two]
Location of three is 7, list.get(7) = three
*/
```

就像使用陣列進行搜尋和排序一樣，如果使用 **Comparator** 進行排序，則必須使用相同的 **Comparator** 執行 `binarySearch()` 。

該程式還示範了 **Collections** 中的 `shuffle()` 方法，該方法隨機打亂了 **List** 的順序。 **ListIterator** 是在打亂後的列表中的特定位置建立的，用於從該位置刪除元素，直到列表末尾。

<!-- Making a Collection or Map Unmodifiable -->
### 建立不可修改的 Collection 或 Map

通常，建立 **Collection** 或 **Map** 的唯讀版本會很方便。 **Collections** 類透過將原始集合傳遞給一個方法然後返回一個唯讀版本的集合。 對於 **Collection** （如果不能將 **Collection** 視為更具體的類型）， **List** ， **Set** 和 **Map** ，這類方法有許多變體。這個範例展示了針對每種類型，正確構建唯讀版本集合的方法：

```java
// collectiontopics/ReadOnly.java
// Using the Collections.unmodifiable methods
import java.util.*;
import onjava.*;

public class ReadOnly {
  static Collection<String> data =
    new ArrayList<>(Countries.names(6));
  public static void main(String[] args) {
    Collection<String> c =
      Collections.unmodifiableCollection(
        new ArrayList<>(data));
    System.out.println(c); // Reading is OK
    //- c.add("one"); // Can't change it

    List<String> a = Collections.unmodifiableList(
        new ArrayList<>(data));
    ListIterator<String> lit = a.listIterator();
    System.out.println(lit.next()); // Reading is OK
    //- lit.add("one"); // Can't change it

    Set<String> s = Collections.unmodifiableSet(
      new HashSet<>(data));
    System.out.println(s); // Reading is OK
    //- s.add("one"); // Can't change it

    // For a SortedSet:
    Set<String> ss =
      Collections.unmodifiableSortedSet(
        new TreeSet<>(data));

    Map<String,String> m =
      Collections.unmodifiableMap(
        new HashMap<>(Countries.capitals(6)));
    System.out.println(m); // Reading is OK
    //- m.put("Ralph", "Howdy!");

    // For a SortedMap:
    Map<String,String> sm =
      Collections.unmodifiableSortedMap(
        new TreeMap<>(Countries.capitals(6)));
  }
}
/* Output:
[ALGERIA, ANGOLA, BENIN, BOTSWANA, BURKINA FASO,
BURUNDI]
ALGERIA
[BENIN, BOTSWANA, ANGOLA, BURKINA FASO, ALGERIA,
BURUNDI]
{BENIN=Porto-Novo, BOTSWANA=Gaberone, ANGOLA=Luanda,
BURKINA FASO=Ouagadougou, ALGERIA=Algiers,
BURUNDI=Bujumbura}
*/
```

為特定類型呼叫 “unmodifiable” 方法不會導致編譯時檢查，但是一旦發生轉換，對修改特定集合內容的任何方法呼叫都將產生 **UnsupportedOperationException** 異常。

在每種情況下，在將集合設定為唯讀之前，必須使用有意義的資料填充集合。填充完成後，最好的方法是用 “unmodifiable” 方法呼叫生成的引用取代現有引用。這樣，一旦使得內容無法修改，那麼就不會冒有意外更改內容的風險。另一方面，此工具還允許將可修改的集合保留為類中的**私有**集合，並從方法呼叫處返回對該集合的唯讀引用。所以，你可以在類內修改它，但其他人只能讀它。

<!-- Synchronizing a Collection or Map -->
### 同步 Collection 或 Map

**synchronized** 關鍵字是多執行緒主題的重要組成部分，更複雜的內容在[第二十四章 並發編程]()中介紹。在這裡，只需要注意到 **Collections** 類包含一種自動同步整個集合的方法。 語法類似於 “unmodifiable” 方法：

```java
// collectiontopics/Synchronization.java
// Using the Collections.synchronized methods
import java.util.*;

public class Synchronization {
  public static void main(String[] args) {
    Collection<String> c =
      Collections.synchronizedCollection(
        new ArrayList<>());
    List<String> list = Collections
      .synchronizedList(new ArrayList<>());
    Set<String> s = Collections
      .synchronizedSet(new HashSet<>());
    Set<String> ss = Collections
      .synchronizedSortedSet(new TreeSet<>());
    Map<String,String> m = Collections
      .synchronizedMap(new HashMap<>());
    Map<String,String> sm = Collections
      .synchronizedSortedMap(new TreeMap<>());
  }
}
```

最好立即透過適當的 “synchronized” 方法傳遞新集合，如上所示。這樣，就不會意外地暴露出非同步版本。

<!-- Fail Fast -->
#### Fail Fast

Java 集合還具有防止多個行程修改集合內容的機制。如果目前正在疊代集合，然後有其他一些行程介入並插入，刪除或更改該集合中的物件，則會出現此問題。也許在集合中已經遍歷過了那個元素，也許還沒有遍歷到，也許在呼叫 `size()` 之後集合的大小會縮小...有許多災難情景。 Java 集合庫使用一種 *fail-fast* 的機制，該機制可以檢測到除了目前行程引起的更改之外，其它任何對集合的更改操作。如果它檢測到其他人正在修改集合，則會立即生成 **ConcurrentModificationException** 異常。這就是“fail-fast”的含義——它不會在以後使用更複雜的演算法嘗試檢測問題（快速失敗）。

透過建立疊代器並向疊代器指向的集合中添加元素，可以很容易地看到操作中的 fail-fast 機制，如下所示：

```java
// collectiontopics/FailFast.java
// Demonstrates the "fail-fast" behavior
import java.util.*;

public class FailFast {
  public static void main(String[] args) {
    Collection<String> c = new ArrayList<>();
    Iterator<String> it = c.iterator();
    c.add("An object");
    try {
      String s = it.next();
    } catch(ConcurrentModificationException e) {
      System.out.println(e);
    }
  }
}
/* Output:
java.util.ConcurrentModificationException
*/
```

異常來自於在從集合中獲得疊代器之後，又嘗試在集合中添加元素。程式的兩個部分可能會修改同一個集合，這種可能性的存在會產生不確定狀態，因此異常會通知你更改程式碼。在這種情況下，應先將所有元素添加到集合，然後再獲取疊代器。

**ConcurrentHashMap** ， **CopyOnWriteArrayList** 和 **CopyOnWriteArraySet** 使用了特定的技術來避免產生 **ConcurrentModificationException** 異常。

<!-- Holding References -->
## 持有引用

**java.lang.ref** 中庫包含一組類，這些類允許垃圾收集具有更大的靈活性。特別是當擁有可能導致記憶體耗盡的大物件時，這些類特別有用。這裡有三個從抽象類 **Reference** 繼承來的類： **SoftReference** （軟引用）， **WeakReference** （弱引用）和 **PhantomReference** （虛引用）繼承了三個類。如果一個物件只能透過這其中的一個 **Reference** 物件訪問，那麼這三種類型每個都為垃圾收集器提供不同級別的間接引用（indirection）。

如果一個物件是 *可達的*（reachable），那麼意味著在程式中的某個位置可以找到該物件。這可能意味著在堆疊上有一個直接引用該物件的普通引用，但也有可能是引用了一個對該物件有引用的物件，這可以有很多中間環節。如果某個物件是可達的，則垃圾收集器無法釋放它，因為它仍然被程式所使用。如果某個物件是不可達的，則程式無法使用它，那麼垃圾收集器回收該物件就是安全的。

使用 **Reference** 物件繼續保持對該物件的引用，以到達該物件，但也允許垃圾收集器釋放該物件。因此，程式可以使用該物件，但如果記憶體即將耗盡，則允許釋放該物件。

可以透過使用 **Reference** 物件作為你和普通引用之間的中介（代理）來實現此目的。此外，必須沒有物件的普通引用（未包含在 **Reference** 物件中的物件）。如果垃圾收集器發現物件可透過普通引用訪問，則它不會釋放該物件。

按照 **SoftReference** ， **WeakReference** 和 **PhantomReference** 的順序，每個都比前一個更“弱”，並且對應於不同的可達性級別。軟引用用於實現對記憶體敏感的快取。弱引用用於實現“規範化映射”（ canonicalized mappings）——物件的實例可以在程式的多個位置同時使用，以節省儲存，但不會阻止其鍵（或值）被回收。虛引用用於調度 pre-mortem 清理操作，這是一種比 Java 終結機制（Java finalization mechanism）更靈活的方式。

使用 **SoftReference** 和 **WeakReference** ，可以選擇是否將它們放在 **ReferenceQueue** （用於 pre-mortem 清理操作的裝置）中，但 **PhantomReference** 只能在 **ReferenceQueue** 上構建。下面是一個簡單的示範：

```java
// collectiontopics/References.java
// Demonstrates Reference objects
import java.lang.ref.*;
import java.util.*;

class VeryBig {
  private static final int SIZE = 10000;
  private long[] la = new long[SIZE];
  private String ident;
  VeryBig(String id) { ident = id; }
  @Override
  public String toString() { return ident; }
  @Override
  protected void finalize() {
    System.out.println("Finalizing " + ident);
  }
}

public class References {
  private static ReferenceQueue<VeryBig> rq =
    new ReferenceQueue<>();
  public static void checkQueue() {
    Reference<? extends VeryBig> inq = rq.poll();
    if(inq != null)
      System.out.println("In queue: " + inq.get());
  }
  public static void main(String[] args) {
    int size = 10;
    // Or, choose size via the command line:
    if(args.length > 0)
      size = Integer.valueOf(args[0]);
    LinkedList<SoftReference<VeryBig>> sa =
      new LinkedList<>();
    for(int i = 0; i < size; i++) {
      sa.add(new SoftReference<>(
        new VeryBig("Soft " + i), rq));
      System.out.println(
        "Just created: " + sa.getLast());
      checkQueue();
    }
    LinkedList<WeakReference<VeryBig>> wa =
      new LinkedList<>();
    for(int i = 0; i < size; i++) {
      wa.add(new WeakReference<>(
        new VeryBig("Weak " + i), rq));
      System.out.println(
        "Just created: " + wa.getLast());
      checkQueue();
    }
    SoftReference<VeryBig> s =
      new SoftReference<>(new VeryBig("Soft"));
    WeakReference<VeryBig> w =
      new WeakReference<>(new VeryBig("Weak"));
    System.gc();
    LinkedList<PhantomReference<VeryBig>> pa =
      new LinkedList<>();
    for(int i = 0; i < size; i++) {
      pa.add(new PhantomReference<>(
        new VeryBig("Phantom " + i), rq));
      System.out.println(
        "Just created: " + pa.getLast());
      checkQueue();
    }
  }
}
/* Output: (First and Last 10 Lines)
Just created: java.lang.ref.SoftReference@15db9742
Just created: java.lang.ref.SoftReference@6d06d69c
Just created: java.lang.ref.SoftReference@7852e922
Just created: java.lang.ref.SoftReference@4e25154f
Just created: java.lang.ref.SoftReference@70dea4e
Just created: java.lang.ref.SoftReference@5c647e05
Just created: java.lang.ref.SoftReference@33909752
Just created: java.lang.ref.SoftReference@55f96302
Just created: java.lang.ref.SoftReference@3d4eac69
Just created: java.lang.ref.SoftReference@42a57993
...________...________...________...________...
Just created: java.lang.ref.PhantomReference@45ee12a7
In queue: null
Just created: java.lang.ref.PhantomReference@330bedb4
In queue: null
Just created: java.lang.ref.PhantomReference@2503dbd3
In queue: null
Just created: java.lang.ref.PhantomReference@4b67cf4d
In queue: null
Just created: java.lang.ref.PhantomReference@7ea987ac
In queue: null
*/
```

當執行此程式（將輸出重定向到文字文件以查看頁面中的輸出）時，將會看到物件是被垃圾收集了的，雖然仍然可以透過 **Reference** 物件訪問它們（使用 `get()` 來獲取實際的物件引用）。 還可以看到 **ReferenceQueue** 始終生成包含 **null** 物件的 **Reference** 。 要使用它，請從特定的 **Reference** 類繼承，並為新類添加更多有用的方法。


<!-- The WeakHashMap -->
### WeakHashMap

集合類庫中有一個特殊的 **Map** 來儲存弱引用： **WeakHashMap** 。 此類可以更輕鬆地建立規範化映射。在這種映射中，可以透過僅僅建立一個特定值的實例來節省儲存空間。當程式需要該值時，它會尋找映射中的現有物件並使用它（而不是從頭開始建立一個）。 該映射可以將值作為其初始化的一部分，但更有可能的是在需要時建立該值。

由於這是一種節省儲存空間的技術，因此 **WeakHashMap** 允許垃圾收集器自動清理鍵和值，這是非常方便的。不能對放在 **WeakHashMap** 中的鍵和值做任何特殊操作，它們由 map 自動包裝在 **WeakReference** 中。當鍵不再被使用的時候才允許清理，如下所示：

```java
// collectiontopics/CanonicalMapping.java
// Demonstrates WeakHashMap
import java.util.*;

class Element {
  private String ident;
  Element(String id) { ident = id; }
  @Override
  public String toString() { return ident; }
  @Override
  public int hashCode() {
    return Objects.hashCode(ident);
  }
  @Override
  public boolean equals(Object r) {
    return r instanceof Element &&
      Objects.equals(ident, ((Element)r).ident);
  }
  @Override
  protected void finalize() {
    System.out.println("Finalizing " +
      getClass().getSimpleName() + " " + ident);
  }
}

class Key extends Element {
  Key(String id) { super(id); }
}

class Value extends Element {
  Value(String id) { super(id); }
}

public class CanonicalMapping {
  public static void main(String[] args) {
    int size = 1000;
    // Or, choose size via the command line:
    if(args.length > 0)
      size = Integer.valueOf(args[0]);
    Key[] keys = new Key[size];
    WeakHashMap<Key,Value> map =
      new WeakHashMap<>();
    for(int i = 0; i < size; i++) {
      Key k = new Key(Integer.toString(i));
      Value v = new Value(Integer.toString(i));
      if(i % 3 == 0)
        keys[i] = k; // Save as "real" references
      map.put(k, v);
    }
    System.gc();
  }
}
```

**Key** 類必須具有 `hashCode()` 和 `equals()` ，因為它將被用作散列資料結構中的鍵。 `hashCode()` 的內容在[附錄：理解hashCode和equals方法]()中進行了描述。

執行程式，你會看到垃圾收集器每三個鍵跳過一次。對該鍵的普通引用也被放置在 **keys** 陣列中，因此這些物件不能被垃圾收集。

<!-- Java 1.0/1.1 Collections -->
## Java 1.0 / 1.1 的集合類

不幸的是，許多程式碼是使用 Java 1.0 / 1.1 中的集合編寫的，甚至新程式碼有時也是使用這些類編寫的。編寫新程式碼時切勿使用舊集合。舊的集合類有限，所以關於它們的討論不多。由於它們是不合時宜的，所以我會儘量避免過分強調一些可怕的設計決定。

<!-- Vector & Enumeration -->
### Vector 和 Enumeration

Java 1.0 / 1.1 中唯一的自擴展序列是 **Vector** ，因此它被用於很多地方。它的缺陷太多了，無法在這裡描述（參見《Java編程思想》第1版，可從[www.OnJava8.com](www.OnJava8.com)免費下載）。基本上，你可以將它看作是具有冗長且笨拙的方法名稱的 **ArrayList** 。在修訂後的 Java 集合庫中，**Vector** 已經被調整適配過，因此可以作為 **Collection** 和 **List** 來使用。事實證明這有點不正常，集合類庫仍然包含它只是為了支援舊的 Java 程式碼，但這會讓一些人誤以為 **Vector** 已經變得更好了。

疊代器的 Java 1.0 / 1.1 版本選擇建立一個新名稱“enumeration”，而不是使用每個人都熟悉的術語（“iterator”）。 **Enumeration** 介面小於 **Iterator** ，只包含兩個方法，並且它使用更長的方法名稱：如果還有更多元素，則 `boolean hasMoreElements()` 返回 `true` ， `Object nextElement()` 返回此enumeration的下一個元素 （否則會拋出異常）。

**Enumeration** 只是一個介面，而不是一個實現，甚至新的類庫有時仍然使用舊的 **Enumeration** ，這是不幸的，但通常是無害的。應該總是在自己的程式碼中使用 **Iterator** ，但要做好準備應對那些提供 **Enumeration** 的類庫。

此外，可以使用 `Collections.enumeration()` 方法為任何 **Collection** 生成 **Enumeration** ，如下例所示：

```java
// collectiontopics/Enumerations.java
// Java 1.0/1.1 Vector and Enumeration
import java.util.*;
import onjava.*;

public class Enumerations {
  public static void main(String[] args) {
    Vector<String> v =
      new Vector<>(Countries.names(10));
    Enumeration<String> e = v.elements();
    while(e.hasMoreElements())
      System.out.print(e.nextElement() + ", ");
    // Produce an Enumeration from a Collection:
    e = Collections.enumeration(new ArrayList<>());
  }
}
/* Output:
ALGERIA, ANGOLA, BENIN, BOTSWANA, BURKINA FASO,
BURUNDI, CAMEROON, CAPE VERDE, CENTRAL AFRICAN
REPUBLIC, CHAD,
*/
```

要生成 **Enumeration** ，可以呼叫 `elements()` ，然後可以使用它來執行向前疊代。

最後一行建立一個 **ArrayList** ，並使用 `enumeration() ` 來將 **ArrayList** 適配為一個 **Enumeration** 。 因此，如果有舊程式碼需要使用 **Enumeration** ，你仍然可以使用新集合。

<!-- Hashtable -->
### Hashtable

正如你在本附錄中的效能比較中所看到的，基本的 **Hashtable** 與 **HashMap** 非常相似，甚至方法名稱都相似。在新程式碼中沒有理由使用 **Hashtable** 而不是 **HashMap** 。

<!-- Stack -->
### Stack

之前使用 **LinkedList** 引入了堆疊的概念。 Java 1.0 / 1.1 **Stack** 的奇怪之處在於，不是以組合方式使用 **Vector** ，而是繼承自 **Vector** 。 因此它具有 **Vector** 的所有特徵和行為以及一些額外的 **Stack** 行為。很難去知道設計師是否有意識地認為這樣做是有用的，或者它是否只是太天真了，無論如何，它在進入發行版之前顯然沒有經過審查，所以這個糟糕的設計仍然存在（但不要使用它）。

這是 **Stack** 的簡單示範，向堆疊中放入列舉中每一個類型的 **String** 形式。它還展示了如何輕鬆地將 **LinkedList** 用作堆疊，或者使用在[第十二章：集合]()章節中建立的 **Stack** 類：

```java
// collectiontopics/Stacks.java
// Demonstration of Stack Class
import java.util.*;

enum Month { JANUARY, FEBRUARY, MARCH, APRIL,
  MAY, JUNE, JULY, AUGUST, SEPTEMBER,
  OCTOBER, NOVEMBER }

public class Stacks {
  public static void main(String[] args) {
    Stack<String> stack = new Stack<>();
    for(Month m : Month.values())
      stack.push(m.toString());
    System.out.println("stack = " + stack);
    // Treating a stack as a Vector:
    stack.addElement("The last line");
    System.out.println(
      "element 5 = " + stack.elementAt(5));
    System.out.println("popping elements:");
    while(!stack.empty())
      System.out.print(stack.pop() + " ");

    // Using a LinkedList as a Stack:
    LinkedList<String> lstack = new LinkedList<>();
    for(Month m : Month.values())
      lstack.addFirst(m.toString());
    System.out.println("lstack = " + lstack);
    while(!lstack.isEmpty())
      System.out.print(lstack.removeFirst() + " ");

    // Using the Stack class from
    // the Collections Chapter:
    onjava.Stack<String> stack2 =
      new onjava.Stack<>();
    for(Month m : Month.values())
      stack2.push(m.toString());
    System.out.println("stack2 = " + stack2);
    while(!stack2.isEmpty())
      System.out.print(stack2.pop() + " ");

  }
}
/* Output:
stack = [JANUARY, FEBRUARY, MARCH, APRIL, MAY, JUNE,
JULY, AUGUST, SEPTEMBER, OCTOBER, NOVEMBER]
element 5 = JUNE
popping elements:
The last line NOVEMBER OCTOBER SEPTEMBER AUGUST JULY
JUNE MAY APRIL MARCH FEBRUARY JANUARY lstack =
[NOVEMBER, OCTOBER, SEPTEMBER, AUGUST, JULY, JUNE, MAY,
APRIL, MARCH, FEBRUARY, JANUARY]
NOVEMBER OCTOBER SEPTEMBER AUGUST JULY JUNE MAY APRIL
MARCH FEBRUARY JANUARY stack2 = [NOVEMBER, OCTOBER,
SEPTEMBER, AUGUST, JULY, JUNE, MAY, APRIL, MARCH,
FEBRUARY, JANUARY]
NOVEMBER OCTOBER SEPTEMBER AUGUST JULY JUNE MAY APRIL
MARCH FEBRUARY JANUARY
*/
```

**String** 形式是由 **Month** 中的列舉常量生成的，使用 `push()` 壓入到堆疊中，然後使用 `pop()` 從堆疊頂部取出。為了說明一點，將 **Vector** 的操作也在 **Stack** 物件上執行， 這是可能的，因為憑藉繼承， **Stack** 是 **Vector** 。 因此，可以在 **Vector** 上執行的所有操作也可以在 **Stack** 上執行，例如 `elementAt()` 。

如前所述，在需要堆疊行為時使用 **LinkedList** ，或者從 **LinkedList** 類建立的 **onjava.Stack** 類。

<!-- BitSet -->
### BitSet

**BitSet** 用於有效地儲存大量的開關訊息。僅從尺寸大小的角度來看它是有效的，如果你正在尋找有效的訪問，它比使用本機陣列（native array）稍慢。

此外， **BitSet** 的最小大小是 **long** ：64位。這意味著如果你要儲存更小的東西，比如8位， **BitSet** 就是浪費，如果尺寸有問題，你最好建立自己的類，或者只是用一個陣列來儲存你的標誌。（只有在你建立許多包含開關訊息列表的物件時才會出現這種情況，並且只應根據分析和其他指標來決定。如果你做出此決定只是因為您認為 **BitSet** 太大，那麼最終會產生不必要的複雜性並且浪費大量時間。）

當添加更多元素時，普通集合會擴展， **BitSet**也會這樣做。以下範例顯示了 **BitSet** 的工作原理：

```java
// collectiontopics/Bits.java
// Demonstration of BitSet
import java.util.*;

public class Bits {
  public static void printBitSet(BitSet b) {
    System.out.println("bits: " + b);
    StringBuilder bbits = new StringBuilder();
    for(int j = 0; j < b.size() ; j++)
      bbits.append(b.get(j) ? "1" : "0");
    System.out.println("bit pattern: " + bbits);
  }
  public static void main(String[] args) {
    Random rand = new Random(47);
    // Take the LSB of nextInt():
    byte bt = (byte)rand.nextInt();
    BitSet bb = new BitSet();
    for(int i = 7; i >= 0; i--)
      if(((1 << i) &  bt) != 0)
        bb.set(i);
      else
        bb.clear(i);
    System.out.println("byte value: " + bt);
    printBitSet(bb);

    short st = (short)rand.nextInt();
    BitSet bs = new BitSet();
    for(int i = 15; i >= 0; i--)
      if(((1 << i) &  st) != 0)
        bs.set(i);
      else
        bs.clear(i);
    System.out.println("short value: " + st);
    printBitSet(bs);

    int it = rand.nextInt();
    BitSet bi = new BitSet();
    for(int i = 31; i >= 0; i--)
      if(((1 << i) &  it) != 0)
        bi.set(i);
      else
        bi.clear(i);
    System.out.println("int value: " + it);
    printBitSet(bi);

    // Test bitsets >= 64 bits:
    BitSet b127 = new BitSet();
    b127.set(127);
    System.out.println("set bit 127: " + b127);
    BitSet b255 = new BitSet(65);
    b255.set(255);
    System.out.println("set bit 255: " + b255);
    BitSet b1023 = new BitSet(512);
    b1023.set(1023);
    b1023.set(1024);
    System.out.println("set bit 1023: " + b1023);
  }
}
/* Output:
byte value: -107
bits: {0, 2, 4, 7}
bit pattern: 101010010000000000000000000000000000000000
0000000000000000000000
short value: 1302
bits: {1, 2, 4, 8, 10}
bit pattern: 011010001010000000000000000000000000000000
0000000000000000000000
int value: -2014573909
bits: {0, 1, 3, 5, 7, 9, 11, 18, 19, 21, 22, 23, 24,
25, 26, 31}
bit pattern: 110101010101000000110111111000010000000000
0000000000000000000000
set bit 127: {127}
set bit 255: {255}
set bit 1023: {1023, 1024}
*/
```

隨機數生成器用於建立隨機 **byte** ， **short** 和 **int** ，並且每個都在 **BitSet** 中轉換為相應的位模式。這樣可以正常工作，因為 **BitSet** 是64位，所以這些都不會導致它的大小增加，然後建立更大的 **BitSet** 。 請注意， **BitSet** 會根據需要進行擴展。

對於可以命名的固定標誌集， **EnumSet** （參見[第二十二章：列舉]()章節）通常比 **BitSet** 更好，因為 **EnumSet** 允許操作名稱而不是數字位位置，從而可以減少錯誤。 **EnumSet** 還可以防止意外地添加新的標記位置，這可能會導致一些嚴重的，難以發現的錯誤。使用 **BitSet** 而不是 **EnumSet** 的唯一原因是，不知道在執行時需要多少標誌，或者為標誌分配名稱是不合理的，或者需要 **BitSet** 中的一個特殊操作（請參閱 **BitSet** 和 **EnumSet** 的 JDK 文件）。

<!-- Summary -->
## 本章小結

集合可以說是程式語言中最常用的工具。有些語言（例如Python）甚至將基本集合元件（列表，映射和集合）作為內建函數包含在其中。

正如在[第十二章：集合]()章節中看到的那樣，可以使用集合執行許多非常有用的操作，而不需要太多努力。但是，在某些時候，為了正確地使用它們而不得不更多地了解集合，特別是，必須充分了解散列操作以編寫自己的 `hashCode()` 方法（並且必須知道何時需要），並且你必須充分了解各種集合實現，以根據你的需求選擇合適的集合。本附錄涵蓋了這些概念，並討論了有關集合庫的其他有用詳細訊息。你現在應該已經準備好在日常編程任務中使用 Java 集合了。

集合庫的設計很困難（大多數庫設計問題都是如此）。在 C++ 中，集合類涵蓋了許多不同類的基礎。這比之前可用的 C++ 集合類更好，但它沒有很好地轉換為 Java 。在另一個極端，我看到了一個由單個類“collection”組成的集合庫，它同時充當線性序列和關聯陣列。 Java 集合庫試圖在功能和複雜性之間取得平衡。結果在某些地方看起來有點奇怪。與早期 Java 庫中的一些決策不同，這些奇怪的不是事故，而是在基於複雜性的權衡下而仔細考慮的決策。


[^1]: **java.util** 中的 **Map** 使用 **Map** 的 `getKey()` 和 `getValue()` 執行批次複製，因此這是有效的。如果自訂 **Map** 只是複製整個 **Map.Entry** ，那麼這種方法就會出現問題。

[^2]: 雖然當我用這種方式描述它的時候聽起來很奇怪而且好像沒什麼用處，但在[第十九章 類型訊息]()章節中已經看到過，這種動態行為也可以非常強大有用。

[^3]: 如果這些加速仍然無法滿足性能需求，則可以透過編寫自己的 **Map** 並將其自訂為特定類型來進一步加速表尋找，以避免因向 **物件** 轉換而導致的延遲。為了達到更高的效能水平，速度愛好者可以使用 Donald Knuth 的《電腦程式設計藝術（第3卷）：排序與尋找》（第二版），將溢位桶列表（overflow bucket lists）取代為具有兩個額外優勢的陣列：它們可以針對磁碟儲存進行最佳化，並且它們可以節省大部分建立和回收個別記錄（individual records）的時間。

<!-- 分頁 -->
<div style="page-break-after: always;"></div>
