[TOC]

<!-- Streams -->
# 第十四章 流式編程

> 集合最佳化了物件的儲存，而流和物件的處理有關。

流是一系列與特定儲存機制無關的元素——實際上，流並沒有“儲存”之說。

使用流，無需疊代集合中的元素，就可以從管道提取和操作元素。這些管道通常被組合在一起，形成一系列對流進行操作的管道。

在大多數情況下，將物件儲存在集合中是為了處理他們，因此你將會發現你將把編程的主要焦點從集合轉移到了流上。流的一個核心好處是，它使得程式更加短小並且更易理解。當 Lambda 表達式和方法引用（method references）和流一起使用的時候會讓人感覺自成一體。流使得 Java 8 更具吸引力。

舉個例子，假如你要隨機展示 5 至 20 之間不重複的整數並進行排序。實際上，你的關注點首先是建立一個有序集合。圍繞這個集合進行後續的操作。但是使用流式編程，你就可以簡單陳述你想做什麼：

```java
// streams/Randoms.java
import java.util.*;
public class Randoms {
    public static void main(String[] args) {
        new Random(47)
            .ints(5, 20)
            .distinct()
            .limit(7)
            .sorted()
            .forEach(System.out::println);
    }
}
```

輸出結果：

```
6
10
13
16
17
18
19
```

首先，我們給 **Random** 物件一個種子（以便程式再次執行時產生相同的輸出）。`ints()` 方法產生一個流並且 `ints()` 方法有多種方式的重載 — 兩個參數限定了產生的數值的邊界。這將生成一個隨機整數流。我們用中間流操作（intermediate stream operation） `distinct()` 使流中的整數不重複，然後使用 `limit()` 方法獲取前 7 個元素。接下來使用 `sorted()` 方法排序。最終使用 `forEach()` 方法遍歷輸出，它根據傳遞給它的函數對流中的每個物件執行操作。在這裡，我們傳遞了一個可以在控制台顯示每個元素的方法引用：`System.out::println` 。

注意 `Randoms.java` 中沒有聲明任何變數。流可以在不使用賦值或可變資料的情況下對有狀態的系統建模，這非常有用。

聲明式編程（Declarative programming）是一種編程風格，它聲明想要做什麼，而非指明如何做。正如我們在函數式編程中所看到的。你會發現指令式編程的形式更難以理解。程式碼範例：

```java
// streams/ImperativeRandoms.java
import java.util.*;
public class ImperativeRandoms {
    public static void main(String[] args) {
        Random rand = new Random(47);
        SortedSet<Integer> rints = new TreeSet<>();
        while(rints.size() < 7) {
            int r = rand.nextInt(20);
            if(r < 5) continue;
            rints.add(r);
        }
        System.out.println(rints);
    }
}
```

輸出結果：

```
[7, 8, 9, 11, 13, 15, 18]
```

在 `Randoms.java` 中，我們無需定義任何變數，但在這裡我們定義了 3 個變數： `rand`，`rints` 和 `r`。由於 `nextInt()` 方法沒有下限的原因（其內建的下限永遠為 0），這段程式碼實現起來更複雜。所以我們要生成額外的值來過濾小於 5 的結果。

注意，你必須用力的研究才能弄明白`ImperativeRandoms.java`程式在幹什麼。而在 `Randoms.java` 中，程式碼直接告訴了你它正在做什麼。這種語義的清晰性是使用Java 8 流式編程的重要原因之一。

像在 `ImperativeRandoms.java` 中那樣顯式地編寫疊代過程的方式稱為外部疊代。而在 `Randoms.java` 中，你看不到任何上述的疊代過程，所以它被稱為內部疊代，這是流式編程的一個核心特徵。內部疊代產生的程式碼可讀性更強，而且能更簡單的使用多核處理器。透過放棄對疊代過程的控制，可以把控制權交給並行化機制。我們將在[並發編程](24-Concurrent-Programming.md)一章中學習這部分內容。

另一個重要方面，流是懶載入的。這代表著它只在絕對必要時才計算。你可以將流看作“延遲列表”。由於計算延遲，流使我們能夠表示非常大（甚至無限）的序列，而不需要考慮記憶體問題。

<!-- Java 8 Stream Support -->

## 流支援

Java 設計者面臨著這樣一個難題：現存的大量類庫不僅為 Java 所用，同時也被應用在整個 Java 生態圈數百萬行的程式碼中。如何將一個全新的流的概念融入到現有類庫中呢？

比如在 **Random** 中添加更多的方法。只要不改變原有的方法，現有程式碼就不會受到干擾。

一個大的挑戰來自於使用介面的庫。集合類是其中關鍵的一部分，因為你想把集合轉為流。但是如果你將一個新方法添加到介面，那就破壞了每一個實現介面的類，因為這些類都沒有實現你添加的新方法。

Java 8 採用的解決方案是：在[介面](10-Interfaces.md)中添加被 `default`（`預設`）修飾的方法。透過這種方案，設計者們可以將流式（*stream*）方法平滑地嵌入到現有類中。流方法預置的操作幾乎已滿足了我們平常所有的需求。流操作的類型有三種：建立流，修改流元素（中間操作， Intermediate Operations），消費流元素（終端操作， Terminal Operations）。最後一種類型通常意味著收集流元素（通常是到集合中）。

下面我們來看一下每種類型的流操作。

<!-- Stream Creation -->
## 流建立

你可以透過 `Stream.of()` 很容易地將一組元素轉化成為流（`Bubble` 類在本章的後面定義）：

```java
// streams/StreamOf.java
import java.util.stream.*;
public class StreamOf {
    public static void main(String[] args) {
        Stream.of(new Bubble(1), new Bubble(2), new Bubble(3))
            .forEach(System.out::println);
        Stream.of("It's ", "a ", "wonderful ", "day ", "for ", "pie!")
            .forEach(System.out::print);
        System.out.println();
        Stream.of(3.14159, 2.718, 1.618)
            .forEach(System.out::println);
    }
}
```

輸出結果：

```
Bubble(1)
Bubble(2)
Bubble(3)
It's a wonderful day for pie!
3.14159
2.718
1.618
```

除此之外，每個集合都可以透過呼叫 `stream()` 方法來產生一個流。程式碼範例：

```java
// streams/CollectionToStream.java
import java.util.*;
import java.util.stream.*;
public class CollectionToStream {
    public static void main(String[] args) {
        List<Bubble> bubbles = Arrays.asList(new Bubble(1), new Bubble(2), new Bubble(3));
        System.out.println(bubbles.stream()
            .mapToInt(b -> b.i)
            .sum());
        
        Set<String> w = new HashSet<>(Arrays.asList("It's a wonderful day for pie!".split(" ")));
        w.stream()
         .map(x -> x + " ")
         .forEach(System.out::print);
        System.out.println();
        
        Map<String, Double> m = new HashMap<>();
        m.put("pi", 3.14159);
        m.put("e", 2.718);
        m.put("phi", 1.618);
        m.entrySet().stream()
                    .map(e -> e.getKey() + ": " + e.getValue())
                    .forEach(System.out::println);
    }
}
```

輸出結果：

```
6
a pie! It's for wonderful day
phi: 1.618
e: 2.718
pi: 3.14159
```

在建立 `List<Bubble>` 物件之後，我們只需要簡單地呼叫所有集合中都有的 `stream()`。中間操作 `map()` 會獲取流中的所有元素，並且對流中元素應用操作從而產生新的元素，並將其傳遞到後續的流中。通常 `map()` 會獲取物件並產生新的物件，但在這裡產生了特殊的用於數值類型的流。例如，`mapToInt()` 方法將一個物件流（object stream）轉換成為包含整型數字的 `IntStream`。同樣，針對 `Float` 和 `Double` 也有類似名字的操作。

我們透過呼叫字串的 `split()`（該方法會根據參數來分割字串）來獲取元素用於定義變數 `w`。稍後你會知道 `split()` 參數可以是十分複雜，但在這裡我們只是根據空格來分割字串。

為了從 **Map** 集合中產生流資料，我們首先呼叫 `entrySet()` 產生一個物件流，每個物件都包含一個 `key` 鍵以及與其相關聯的 `value` 值。然後分別呼叫 `getKey()` 和 `getValue()` 獲取值。

### 隨機數流

`Random` 類被一組生成流的方法增強了。程式碼範例：

```java
// streams/RandomGenerators.java
import java.util.*;
import java.util.stream.*;
public class RandomGenerators {
    public static <T> void show(Stream<T> stream) {
        stream
        .limit(4)
        .forEach(System.out::println);
        System.out.println("++++++++");
    }
    
    public static void main(String[] args) {
        Random rand = new Random(47);
        show(rand.ints().boxed());
        show(rand.longs().boxed());
        show(rand.doubles().boxed());
        // 控制上限和下限：
        show(rand.ints(10, 20).boxed());
        show(rand.longs(50, 100).boxed());
        show(rand.doubles(20, 30).boxed());
        // 控制流大小：
        show(rand.ints(2).boxed());
        show(rand.longs(2).boxed());
        show(rand.doubles(2).boxed());
        // 控制流的大小和界限
        show(rand.ints(3, 3, 9).boxed());
        show(rand.longs(3, 12, 22).boxed());
        show(rand.doubles(3, 11.5, 12.3).boxed());
    }
}
```

輸出結果：

```
-1172028779
1717241110
-2014573909
229403722
++++++++
2955289354441303771
3476817843704654257
-8917117694134521474
4941259272818818752
++++++++
0.2613610344283964
0.0508673570556899
0.8037155449603999
0.7620665811558285
++++++++
16
10
11
12
++++++++
65
99
54
58
++++++++
29.86777681078574
24.83968447804611
20.09247112332014
24.046793846338723
++++++++
1169976606
1947946283
++++++++
2970202997824602425
-2325326920272830366
++++++++
0.7024254510631527
0.6648552384607359
++++++++
6
7
7
++++++++
17
12
20
++++++++
12.27872414236691
11.732085449736195
12.196509449817267
++++++++
```

為了消除冗餘程式碼，我建立了一個泛型方法 `show(Stream<T> stream)` （在講解泛型之前就使用這個特性，確實有點作弊，但是回報是值得的）。類型參數 `T` 可以是任何類型，所以這個方法對 **Integer**、**Long** 和 **Double** 類型都生效。但是 **Random** 類只能生成基本類型 **int**， **long**， **double** 的流。幸運的是， `boxed()` 流操作將會自動地把基本類型包裝成為對應的裝箱類型，從而使得 `show()` 能夠接受流。

我們可以使用 **Random** 為任意物件集合建立 **Supplier**。如下是一個文字文件提供字串物件的例子。

Cheese.dat 文件內容：

```
// streams/Cheese.dat
Not much of a cheese shop really, is it?
Finest in the district, sir.
And what leads you to that conclusion?
Well, it's so clean.
It's certainly uncontaminated by cheese.
```

我們透過 **File** 類將 Cheese.dat 文件的所有行讀取到 `List<String>` 中。程式碼範例：

```java
// streams/RandomWords.java
import java.util.*;
import java.util.stream.*;
import java.util.function.*;
import java.io.*;
import java.nio.file.*;
public class RandomWords implements Supplier<String> {
    List<String> words = new ArrayList<>();
    Random rand = new Random(47);
    RandomWords(String fname) throws IOException {
        List<String> lines = Files.readAllLines(Paths.get(fname));
        // 略過第一行
        for (String line : lines.subList(1, lines.size())) {
            for (String word : line.split("[ .?,]+"))
                words.add(word.toLowerCase());
        }
    }
    public String get() {
        return words.get(rand.nextInt(words.size()));
    }
    @Override
    public String toString() {
        return words.stream()
            .collect(Collectors.joining(" "));
    }
    public static void main(String[] args) throws Exception {
        System.out.println(
            Stream.generate(new RandomWords("Cheese.dat"))
                .limit(10)
                .collect(Collectors.joining(" ")));
    }
}
```

輸出結果：

```
it shop sir the much cheese by conclusion district is
```

在這裡可以看到 `split()` 更複雜的運用。在構造器裡，每一行都被 `split()` 透過方括號內的空格或其它標點符號分割。在方括號後面的 `+` 表示 `+` 前面的東西可以出現一次或者多次。

你會發現建構子使用指令式編程（外部疊代）進行循環。在以後的例子中，你會看到我們是如何去除指令式編程的使用。這種舊的形式雖不是特別糟糕，但使用流會讓人感覺更好。

在`toString()` 和`main()`方法中你看到了 `collect()` 操作，它根據參數來結合所有的流元素。當你用 `Collectors.joining()`作為 `collect()` 的參數時，將得到一個`String` 類型的結果，該結果是流中的所有元素被`joining()`的參數隔開。還有很多不同的 `Collectors` 用於產生不同的結果。

在主方法中，我們提前看到了 **Stream.**`generate()` 的用法，它可以把任意  `Supplier<T>` 用於生成 `T` 類型的流。


### int 類型的範圍

`IntStream` 類提供了  `range()` 方法用於生成整型序列的流。編寫循環時，這個方法會更加便利：

```java
// streams/Ranges.java
import static java.util.stream.IntStream.*;
public class Ranges {
    public static void main(String[] args) {
        // 傳統方法:
        int result = 0;
        for (int i = 10; i < 20; i++)
            result += i;
        System.out.println(result);
        // for-in 循環:
        result = 0;
        for (int i : range(10, 20).toArray())
            result += i;
        System.out.println(result);
        // 使用流:
        System.out.println(range(10, 20).sum());
    }
}
```

輸出結果：

```
145
145
145
```

在主方法中的第一種方式是我們傳統編寫 `for` 循環的方式；第二種方式，我們使用 `range()` 建立了流並將其轉化為陣列，然後在 `for-in` 程式碼塊中使用。但是，如果你能像第三種方法那樣全程使用流是更好的。我們對範圍中的數字進行求和。在流中可以很方便的使用 `sum()` 操作求和。

注意 **IntStream.**`range()` 相比 `onjava.Range.range()` 擁有更多的限制。這是由於其可選的第三個參數，後者允許步長大於 1，並且可以從大到小來生成。

實用小功能 `repeat()` 可以用來取代簡單的 `for` 循環。程式碼範例：

```java
// onjava/Repeat.java
package onjava;
import static java.util.stream.IntStream.*;
public class Repeat {
    public static void repeat(int n, Runnable action) {
        range(0, n).forEach(i -> action.run());
    }
}
```

其產生的循環更加清晰：

```java
// streams/Looping.java
import static onjava.Repeat.*;
public class Looping {
    static void hi() {
        System.out.println("Hi!");
    }
    public static void main(String[] args) {
        repeat(3, () -> System.out.println("Looping!"));
        repeat(2, Looping::hi);
    }
}
```

輸出結果：

```
Looping!
Looping!
Looping!
Hi!
Hi!
```

原則上，在程式碼中包含並解釋 `repeat()` 並不值得。誠然它是一個相當透明的工具，但結果取決於你的團隊和公司的運作方式。

### generate()

參照 `RandomWords.java` 中 **Stream.**`generate()` 搭配 `Supplier<T>` 使用的例子。程式碼範例：

```java
// streams/Generator.java
import java.util.*;
import java.util.function.*;
import java.util.stream.*;

public class Generator implements Supplier<String> {
    Random rand = new Random(47);
    char[] letters = "ABCDEFGHIJKLMNOPQRSTUVWXYZ".toCharArray();
    
    public String get() {
        return "" + letters[rand.nextInt(letters.length)];
    }
    
    public static void main(String[] args) {
        String word = Stream.generate(new Generator())
                            .limit(30)
                            .collect(Collectors.joining());
        System.out.println(word);
    }
}
```

輸出結果：

```
YNZBRNYGCFOWZNTCQRGSEGZMMJMROE
```

使用 `Random.nextInt()` 方法來挑選字母表中的大寫字母。`Random.nextInt()` 的參數代表可以接受的最大的隨機數範圍，所以使用陣列邊界是經過深思熟慮的。

如果要建立包含相同物件的流，只需要傳遞一個生成那些物件的 `lambda` 到 `generate()` 中：

```java
// streams/Duplicator.java
import java.util.stream.*;
public class Duplicator {
    public static void main(String[] args) {
        Stream.generate(() -> "duplicate")
              .limit(3)
              .forEach(System.out::println);
    }
}
```

輸出結果：

```
duplicate
duplicate
duplicate
```

如下是在本章之前例子中使用過的 `Bubble` 類。**注意**它包含了自己的靜態生成器（Static generator）方法。

```java
// streams/Bubble.java
import java.util.function.*;
public class Bubble {
    public final int i;
    
    public Bubble(int n) {
        i = n;
    }
    
    @Override
    public String toString() {
        return "Bubble(" + i + ")";
    }
    
    private static int count = 0;
    public static Bubble bubbler() {
        return new Bubble(count++);
    }
}
```

由於 `bubbler()` 與 `Supplier<Bubble>` 是介面相容的，我們可以將其方法引用直接傳遞給 **Stream.**`generate()`：

```java
// streams/Bubbles.java
import java.util.stream.*;
public class Bubbles {
    public static void main(String[] args) {
        Stream.generate(Bubble::bubbler)
              .limit(5)
              .forEach(System.out::println);
    }
}
```

輸出結果：

```
Bubble(0)
Bubble(1)
Bubble(2)
Bubble(3)
Bubble(4)
```

這是建立單獨工廠類（Separate Factory class）的另一種方式。在很多方面它更加整潔，但是這是一個對於程式碼組織和品味的問題——你總是可以建立一個完全不同的工廠類。

### iterate()

`Stream.iterate()` 產生的流的第一個元素是種子（iterate方法的第一個參數），然後將種子傳遞給方法（iterate方法的第二個參數）。方法執行的結果被添加到流（作為流的第二個元素），並儲存起來作為下次呼叫 `iterate()`時的第一個參數，以此類推。我們可以利用 `iterate()` 生成一個斐波那契數列。程式碼範例：

```java
// streams/Fibonacci.java
import java.util.stream.*;
public class Fibonacci {
    int x = 1;
    
    Stream<Integer> numbers() {
        return Stream.iterate(0, i -> {
            int result = x + i;
            x = i;
            return result;
        });
    }
    
    public static void main(String[] args) {
        new Fibonacci().numbers()
                       .skip(20) // 過濾前 20 個
                       .limit(10) // 然後取 10 個
                       .forEach(System.out::println);
    }
}
```

輸出結果：

```
6765
10946
17711
28657
46368
75025
121393
196418
317811
514229
```

斐波那契數列將數列中最後兩個元素進行求和以產生下一個元素。`iterate()` 只能記憶結果，因此我們需要利用一個變數 `x` 追蹤另外一個元素。

在主方法中，我們使用了一個之前沒有見過的 `skip()` 操作。它根據參數丟棄指定數量的流元素。在這裡，我們丟棄了前 20 個元素。

### 流的建造者模式

在建造者模式（Builder design pattern）中，首先建立一個 `builder` 物件，然後將建立流所需的多個訊息傳遞給它，最後`builder` 物件執行”建立“流的操作。**Stream** 庫提供了這樣的 `Builder`。在這裡，我們重新審視文件讀取並將其轉換成為單詞流的過程。程式碼範例：

```java
// streams/FileToWordsBuilder.java
import java.io.*;
import java.nio.file.*;
import java.util.stream.*;

public class FileToWordsBuilder {
    Stream.Builder<String> builder = Stream.builder();
    
    public FileToWordsBuilder(String filePath) throws Exception {
        Files.lines(Paths.get(filePath))
             .skip(1) // 略過開頭的注釋行
             .forEach(line -> {
                  for (String w : line.split("[ .?,]+"))
                      builder.add(w);
              });
    }
    
    Stream<String> stream() {
        return builder.build();
    }
    
    public static void main(String[] args) throws Exception {
        new FileToWordsBuilder("Cheese.dat")
            .stream()
            .limit(7)
            .map(w -> w + " ")
            .forEach(System.out::print);
    }
}
```

輸出結果：

```
Not much of a cheese shop really
```

**注意**，構造器會添加文件中的所有單詞（除了第一行，它是包含文件路徑訊息的注釋），但是其並沒有呼叫 `build()`。只要你不呼叫 `stream()` 方法，就可以繼續向 `builder` 物件中添加單詞。

在該類的更完整形式中，你可以添加一個標誌位用於查看 `build()` 是否被呼叫，並且可能的話增加一個可以添加更多單詞的方法。在 `Stream.Builder` 呼叫 `build()` 方法後繼續嘗試添加單詞會產生一個異常。

### Arrays

`Arrays` 類中含有一個名為 `stream()` 的靜態方法用於把陣列轉換成為流。我們可以重寫 `interfaces/Machine.java` 中的主方法用於建立一個流，並將 `execute()` 應用於每一個元素。程式碼範例：

```java
// streams/Machine2.java
import java.util.*;
import onjava.Operations;
public class Machine2 {
    public static void main(String[] args) {
        Arrays.stream(new Operations[] {
            () -> Operations.show("Bing"),
            () -> Operations.show("Crack"),
            () -> Operations.show("Twist"),
            () -> Operations.show("Pop")
        }).forEach(Operations::execute);
    }
}
```

輸出結果：

```
Bing
Crack
Twist
Pop
```

`new Operations[]` 表達式動態建立了 `Operations` 物件的陣列。

`stream()` 同樣可以產生 **IntStream**，**LongStream** 和 **DoubleStream**。

```java
// streams/ArrayStreams.java
import java.util.*;
import java.util.stream.*;

public class ArrayStreams {
    public static void main(String[] args) {
        Arrays.stream(new double[] { 3.14159, 2.718, 1.618 })
            .forEach(n -> System.out.format("%f ", n));
        System.out.println();
        
        Arrays.stream(new int[] { 1, 3, 5 })
            .forEach(n -> System.out.format("%d ", n));
        System.out.println();
        
        Arrays.stream(new long[] { 11, 22, 44, 66 })
            .forEach(n -> System.out.format("%d ", n));
        System.out.println();
        
        // 選擇一個子域:
        Arrays.stream(new int[] { 1, 3, 5, 7, 15, 28, 37 }, 3, 6)
            .forEach(n -> System.out.format("%d ", n));
    }
}
```

輸出結果：

```
3.141590 2.718000 1.618000
1 3 5
11 22 44 66
7 15 28
```

最後一次 `stream()` 的呼叫有兩個額外的參數。第一個參數告訴 `stream()` 從陣列的哪個位置開始選擇元素，第二個參數用於告知在哪裡停止。每種不同類型的 `stream()` 都有類似的操作。

### 正規表示式

Java 的正規表示式將在[字串](18-Strings.md)這一章節詳細介紹。Java 8 在 `java.util.regex.Pattern` 中增加了一個新的方法 `splitAsStream()`。這個方法可以根據傳入的公式將字元序列轉化為流。但是有一個限制，輸入只能是 **CharSequence**，因此不能將流作為 `splitAsStream()` 的參數。

我們再一次查看將文件轉換為單詞的過程。這一次，我們使用流將文件轉換為一個字串，接著使用正規表示式將字串轉化為單詞流。

```java
// streams/FileToWordsRegexp.java
import java.io.*;
import java.nio.file.*;
import java.util.stream.*;
import java.util.regex.Pattern;
public class FileToWordsRegexp {
    private String all;
    public FileToWordsRegexp(String filePath) throws Exception {
        all = Files.lines(Paths.get(filePath))
        .skip(1) // First (comment) line
        .collect(Collectors.joining(" "));
    }
    public Stream<String> stream() {
        return Pattern
        .compile("[ .,?]+").splitAsStream(all);
    }
    public static void
    main(String[] args) throws Exception {
        FileToWordsRegexp fw = new FileToWordsRegexp("Cheese.dat");
        fw.stream()
          .limit(7)
          .map(w -> w + " ")
          .forEach(System.out::print);
        fw.stream()
          .skip(7)
          .limit(2)
          .map(w -> w + " ")
          .forEach(System.out::print);
    }
}
```

輸出結果：

```
Not much of a cheese shop really is it
```

在構造器中我們讀取了文件中的所有內容（跳過第一行注釋，並將其轉化成為單行字串）。現在，當你呼叫 `stream()` 的時候，可以像往常一樣獲取一個流，但這次你可以多次呼叫 `stream()` 在已儲存的字串中建立一個新的流。這裡有個限制，整個文件必須儲存在記憶體中；在大多數情況下這並不是什麼問題，但是這損失了流操作非常重要的優勢：

1. “不需要把流儲存起來。”當然，流確實需要一些內部儲存，但儲存的只是序列的一小部分，和儲存整個序列不同。
2. 它們是懶載入計算的。

幸運的是，我們稍後就會知道如何解決這個問題。

<!-- Intermediate Operations -->

## 中間操作

中間操作用於從一個流中獲取物件，並將物件作為另一個流從後端輸出，以連接到其他操作。

### 跟蹤和除錯

`peek()` 操作的目的是幫助除錯。它允許你無修改地查看流中的元素。程式碼範例：

```java
// streams/Peeking.java
class Peeking {
    public static void main(String[] args) throws Exception {
        FileToWords.stream("Cheese.dat")
        .skip(21)
        .limit(4)
        .map(w -> w + " ")
        .peek(System.out::print)
        .map(String::toUpperCase)
        .peek(System.out::print)
        .map(String::toLowerCase)
        .forEach(System.out::print);
    }
}
```

輸出結果：

```
Well WELL well it IT it s S s so SO so
```

`FileToWords` 稍後定義，但它的功能實現好像和之前我們看到的差不多：產生字串物件的流。之後在其透過管道時呼叫 `peek()` 進行處理。

因為 `peek()` 符合無返回值的 **Consumer** 函數式介面，所以我們只能觀察，無法使用不同的元素來取代流中的物件。

### 流元素排序

在 `Randoms.java` 中，我們熟識了 `sorted()` 的預設比較器實現。其實它還有另一種形式的實現：傳入一個 **Comparator** 參數。程式碼範例：

```java
// streams/SortedComparator.java
import java.util.*;
public class SortedComparator {
    public static void main(String[] args) throws Exception {
        FileToWords.stream("Cheese.dat")
        .skip(10)
        .limit(10)
        .sorted(Comparator.reverseOrder())
        .map(w -> w + " ")
        .forEach(System.out::print);
    }
}
```

輸出結果：

```
you what to the that sir leads in district And
```

`sorted()` 預設了一些預設的比較器。這裡我們使用的是反轉“自然排序”。當然你也可以把 Lambda 函數作為參數傳遞給 `sorted()`。

### 移除元素

* `distinct()`：在 `Randoms.java` 類中的 `distinct()` 可用於消除流中的重複元素。相比建立一個 **Set** 集合，該方法的工作量要少得多。

* `filter(Predicate)`：若元素傳遞給過濾函數產生的結果為`true` ，則過濾操作保留這些元素。

在下例中，`isPrime()` 作為過濾器函數，用於檢測質數。

```java
// streams/Prime.java
import java.util.stream.*;
import static java.util.stream.LongStream.*;
public class Prime {
    public static Boolean isPrime(long n) {
        return rangeClosed(2, (long)Math.sqrt(n))
        .noneMatch(i -> n % i == 0);
    }
    public LongStream numbers() {
        return iterate(2, i -> i + 1)
        .filter(Prime::isPrime);
    }
    public static void main(String[] args) {
        new Prime().numbers()
        .limit(10)
        .forEach(n -> System.out.format("%d ", n));
        System.out.println();
        new Prime().numbers()
        .skip(90)
        .limit(10)
        .forEach(n -> System.out.format("%d ", n));
    }
}
```

輸出結果：

```
2 3 5 7 11 13 17 19 23 29
467 479 487 491 499 503 509 521 523 541
```

`rangeClosed()` 包含了上限值。如果不能整除，即餘數不等於 0，則 `noneMatch()` 操作返回 `true`，如果出現任何等於 0 的結果則返回 `false`。 `noneMatch()` 操作一旦有失敗就會退出。

### 應用函數到元素

- `map(Function)`：將函數操作應用在輸入流的元素中，並將返回值傳遞到輸出流中。

- `mapToInt(ToIntFunction)`：操作同上，但結果是 **IntStream**。

- `mapToLong(ToLongFunction)`：操作同上，但結果是 **LongStream**。

- `mapToDouble(ToDoubleFunction)`：操作同上，但結果是 **DoubleStream**。

在這裡，我們使用 `map()` 映射多種函數到一個字串流中。程式碼範例：

```java
// streams/FunctionMap.java
import java.util.*;
import java.util.stream.*;
import java.util.function.*;
class FunctionMap {
    static String[] elements = { "12", "", "23", "45" };
    static Stream<String>
    testStream() {
        return Arrays.stream(elements);
    }
    static void test(String descr, Function<String, String> func) {
        System.out.println(" ---( " + descr + " )---");
        testStream()
        .map(func)
        .forEach(System.out::println);
    }
    public static void main(String[] args) {
        test("add brackets", s -> "[" + s + "]");
        test("Increment", s -> {
            try {
                return Integer.parseInt(s) + 1 + "";
            }
            catch(NumberFormatException e) {
                return s;
            }
        }
        );
        test("Replace", s -> s.replace("2", "9"));
        test("Take last digit", s -> s.length() > 0 ?
        s.charAt(s.length() - 1) + "" : s);
    }
}
```

輸出結果：

```
 ---( add brackets )---
[12]
[]
[23]
[45]
 ---( Increment )---
13

24
46
 ---( Replace )---
19

93
45
 ---( Take last digit )---
2

3
5

```

在上面的自增範例中，我們用 `Integer.parseInt()` 嘗試將一個字串轉化為整數。如果字串不能被轉化成為整數就會拋出 `NumberFormatException` 異常，此時我們就回過頭來把原始字串放到輸出流中。

在以上例子中，`map()` 將一個字串映射為另一個字串，但是我們完全可以產生和接收類型完全不同的類型，從而改變流的資料類型。下面程式碼範例：

```java
// streams/FunctionMap2.java
// Different input and output types （不同的輸入輸出類型）
import java.util.*;
import java.util.stream.*;
class Numbered {
    final int n;
    Numbered(int n) {
        this.n = n;
    }
    @Override
    public String toString() {
        return "Numbered(" + n + ")";
    }
}
class FunctionMap2 {
    public static void main(String[] args) {
        Stream.of(1, 5, 7, 9, 11, 13)
        .map(Numbered::new)
        .forEach(System.out::println);
    }
}
```

輸出結果：

```
Numbered(1)
Numbered(5)
Numbered(7)
Numbered(9)
Numbered(11)
Numbered(13)
```

我們將獲取到的整數透過構造器 `Numbered::new` 轉化成為 `Numbered` 類型。

如果使用 **Function** 返回的結果是數值類型的一種，我們必須使用合適的 `mapTo數值類型` 進行替代。程式碼範例：

```java
// streams/FunctionMap3.java
// Producing numeric output streams（ 產生數值輸出流）
import java.util.*;
import java.util.stream.*;
class FunctionMap3 {
    public static void main(String[] args) {
        Stream.of("5", "7", "9")
        .mapToInt(Integer::parseInt)
        .forEach(n -> System.out.format("%d ", n));
        System.out.println();
        Stream.of("17", "19", "23")
        .mapToLong(Long::parseLong)
        .forEach(n -> System.out.format("%d ", n));
        System.out.println();
        Stream.of("17", "1.9", ".23")
        .mapToDouble(Double::parseDouble)
        .forEach(n -> System.out.format("%f ", n));
    }
}
```

輸出結果：

```
5 7 9
17 19 23
17.000000 1.900000 0.230000
```

遺憾的是，Java 設計者並沒有盡最大努力去消除基本類型。

### 在 `map()` 中組合流

假設我們現在有了一個傳入的元素流，並且打算對流元素使用 `map()` 函數。現在你已經找到了一些可愛並獨一無二的函數功能，但是問題來了：這個函數功能是產生一個流。我們想要產生一個元素流，而實際卻產生了一個元素流的流。

`flatMap()` 做了兩件事：將產生流的函數應用在每個元素上（與 `map()` 所做的相同），然後將每個流都扁平化為元素，因而最終產生的僅僅是元素。

`flatMap(Function)`：當 `Function` 產生流時使用。

`flatMapToInt(Function)`：當 `Function` 產生 `IntStream` 時使用。

`flatMapToLong(Function)`：當 `Function` 產生 `LongStream` 時使用。

`flatMapToDouble(Function)`：當 `Function` 產生 `DoubleStream` 時使用。

為了弄清它的工作原理，我們從傳入一個刻意設計的函數給  `map()` 開始。該函數接受一個整數並產生一個字串流：

```java
// streams/StreamOfStreams.java
import java.util.stream.*;
public class StreamOfStreams {
    public static void main(String[] args) {
        Stream.of(1, 2, 3)
        .map(i -> Stream.of("Gonzo", "Kermit", "Beaker"))
        .map(e-> e.getClass().getName())
        .forEach(System.out::println);
    }
}
```

輸出結果：

```
java.util.stream.ReferencePipeline$Head
java.util.stream.ReferencePipeline$Head
java.util.stream.ReferencePipeline$Head
```

我們天真地希望能夠得到字串流，但實際得到的卻是“Head”流的流。我們可以使用 `flatMap()` 解決這個問題：

```java
// streams/FlatMap.java
import java.util.stream.*;
public class FlatMap {
    public static void main(String[] args) {
        Stream.of(1, 2, 3)
        .flatMap(i -> Stream.of("Gonzo", "Fozzie", "Beaker"))
        .forEach(System.out::println);
    }
}
```

輸出結果：

```
Gonzo
Fozzie
Beaker
Gonzo
Fozzie
Beaker
Gonzo
Fozzie
Beaker
```

從映射返回的每個流都會自動扁平為組成它的字串。

下面是另一個示範，我們從一個整數流開始，然後使用每一個整數去建立更多的隨機數。

```java
// streams/StreamOfRandoms.java
import java.util.*;
import java.util.stream.*;
public class StreamOfRandoms {
    static Random rand = new Random(47);
    public static void main(String[] args) {
        Stream.of(1, 2, 3, 4, 5)
            .flatMapToInt(i -> IntStream.concat(
        rand.ints(0, 100).limit(i), IntStream.of(-1)))
            .forEach(n -> System.out.format("%d ", n));
    }
}
```

輸出結果：

```
58 -1 55 93 -1 61 61 29 -1 68 0 22 7 -1 88 28 51 89 9 -1
```

在這裡我們引入了 `concat()`，它以參數順序組合兩個流。 如此，我們在每個隨機 `Integer` 流的末尾添加一個 -1 作為標記。你可以看到最終流確實是從一組扁平流中建立的。

因為 `rand.ints()` 產生的是一個 `IntStream`，所以我必須使用 `flatMap()`、`concat()` 和 `of()` 的特定整數形式。

讓我們再看一下將文件劃分為單詞流的任務。我們最後使用到的是 **FileToWordsRegexp.java**，它的問題是需要將整個文件讀入行列表中 —— 顯然需要儲存該列表。而我們真正想要的是建立一個不需要中間儲存層的單詞流。

下面，我們再使用 ` flatMap()` 來解決這個問題：

```java
// streams/FileToWords.java
import java.nio.file.*;
import java.util.stream.*;
import java.util.regex.Pattern;
public class FileToWords {
    public static Stream<String> stream(String filePath) throws Exception {
        return Files.lines(Paths.get(filePath))
        .skip(1) // First (comment) line
        .flatMap(line ->
        Pattern.compile("\\W+").splitAsStream(line));
    }
}
```

`stream()` 現在是一個靜態方法，因為它可以自己完成整個流建立過程。

注意：`\\W+` 是一個正規表示式。表示“非單詞字元”，`+` 表示“可以出現一次或者多次”。小寫形式的 `\\w` 表示“單詞字元”。

我們之前遇到的問題是 `Pattern.compile().splitAsStream()` 產生的結果為流，這意味著當我們只是想要一個簡單的單詞流時，在傳入的行流（stream of lines）上呼叫 `map()` 會產生一個單詞流的流。幸運的是，`flatMap()`  可以將元素流的流扁平化為一個簡單的元素流。或者，我們可以使用 `String.split()` 生成一個陣列，其可以被 `Arrays.stream()` 轉化成為流：

```java
.flatMap(line -> Arrays.stream(line.split("\\W+"))))
```

因為有了真正的流（而不是`FileToWordsRegexp.java` 中基於集合儲存的流），所以每次需要一個新的流時，我們都必須從頭開始建立，因為流不能被復用：

```java
// streams/FileToWordsTest.java
import java.util.stream.*;
public class FileToWordsTest {
    public static void main(String[] args) throws Exception {
        FileToWords.stream("Cheese.dat")
        .limit(7)
        .forEach(s -> System.out.format("%s ", s));
        System.out.println();
        FileToWords.stream("Cheese.dat")
        .skip(7)
        .limit(2)
        .forEach(s -> System.out.format("%s ", s));
    }
}
```

輸出結果：

```
Not much of a cheese shop really
is it
```

在 `System.out.format()` 中的 `%s` 表明參數為 **String** 類型。

<!-- Optional -->
## Optional類

在我們學習終端操作（Terminal Operations）之前，我們必須考慮在一個空流中獲取元素會發生什麼事。我們喜歡沿著“快樂路徑”[^1]把流連接起來，同時假設流不會中斷。然而，在流中放置 `null` 卻會輕易令其中斷。那麼是否存在某種物件，可以在持有流元素的同時，即使在我們尋找的元素不存在時，也能友好地對我們進行提示（也就是說，不會產生異常）？

**Optional** 可以實現這樣的功能。一些標準流操作返回 **Optional** 物件，因為它們並不能保證預期結果一定存在。包括：

- `findFirst()` 返回一個包含第一個元素的 **Optional** 物件，如果流為空則返回 **Optional.empty**
- `findAny()` 返回包含任意元素的 **Optional** 物件，如果流為空則返回 **Optional.empty**
- `max()` 和 `min()` 返回一個包含最大值或者最小值的 **Optional** 物件，如果流為空則返回 **Optional.empty**

 `reduce()` 不再以 `identity` 形式開頭，而是將其返回值包裝在 **Optional** 中。（`identity` 物件成為其他形式的 `reduce()` 的預設結果，因此不存在空結果的風險）

對於數字流 **IntStream**、**LongStream** 和 **DoubleStream**，`average()` 會將結果包裝在 **Optional** 以防止流為空。

以下是對空流進行所有這些操作的簡單測試：

```java
// streams/OptionalsFromEmptyStreams.java
import java.util.*;
import java.util.stream.*;
class OptionalsFromEmptyStreams {
    public static void main(String[] args) {
        System.out.println(Stream.<String>empty()
             .findFirst());
        System.out.println(Stream.<String>empty()
             .findAny());
        System.out.println(Stream.<String>empty()
             .max(String.CASE_INSENSITIVE_ORDER));
        System.out.println(Stream.<String>empty()
             .min(String.CASE_INSENSITIVE_ORDER));
        System.out.println(Stream.<String>empty()
             .reduce((s1, s2) -> s1 + s2));
        System.out.println(IntStream.empty()
             .average());
    }
}
```

輸出結果：

```
Optional.empty
Optional.empty
Optional.empty
Optional.empty
Optional.empty
OptionalDouble.empty
```

當流為空的時候你會獲得一個 **Optional.empty** 物件，而不是拋出異常。**Optional** 擁有 `toString()` 方法可以用於展示有用訊息。

注意，空流是透過 `Stream.<String>empty()` 建立的。如果你在沒有任何上下文環境的情況下呼叫 `Stream.empty()`，Java 並不知道它的資料類型；這個語法解決了這個問題。如果編譯器擁有了足夠的上下文訊息，比如：

```java
Stream<String> s = Stream.empty();
```

就可以在呼叫 `empty()` 時推斷類型。

這個範例展示了 **Optional** 的兩個基本用法：

```java
// streams/OptionalBasics.java
import java.util.*;
import java.util.stream.*;
class OptionalBasics {
    static void test(Optional<String> optString) {
        if(optString.isPresent())
            System.out.println(optString.get()); 
        else
            System.out.println("Nothing inside!");
    }
    public static void main(String[] args) {
        test(Stream.of("Epithets").findFirst());
        test(Stream.<String>empty().findFirst());
    }
}
```

輸出結果：

```
Epithets
Nothing inside!
```

當你接收到 **Optional** 物件時，應首先呼叫 `isPresent()` 檢查其中是否包含元素。如果存在，可使用 `get()` 獲取。

<!-- Convenience Functions -->

### 便利函數

有許多便利函數可以解包 **Optional** ，這簡化了上述“對所包含的物件的檢查和執行操作”的過程：

- `ifPresent(Consumer)`：當值存在時呼叫 **Consumer**，否則什麼也不做。
- `orElse(otherObject)`：如果值存在則直接返回，否則生成 **otherObject**。
- `orElseGet(Supplier)`：如果值存在則直接返回，否則使用 **Supplier** 函數生成一個可替代物件。
- `orElseThrow(Supplier)`：如果值存在直接返回，否則使用 **Supplier** 函數生成一個異常。

如下是針對不同便利函數的簡單示範：

```java
// streams/Optionals.java
import java.util.*;
import java.util.stream.*;
import java.util.function.*;
public class Optionals {
    static void basics(Optional<String> optString) {
        if(optString.isPresent())
            System.out.println(optString.get()); 
        else
            System.out.println("Nothing inside!");
    }
    static void ifPresent(Optional<String> optString) {
        optString.ifPresent(System.out::println);
    }
    static void orElse(Optional<String> optString) {
        System.out.println(optString.orElse("Nada"));
    }
    static void orElseGet(Optional<String> optString) {
        System.out.println(
        optString.orElseGet(() -> "Generated"));
    }
    static void orElseThrow(Optional<String> optString) {
        try {
            System.out.println(optString.orElseThrow(
            () -> new Exception("Supplied")));
        } catch(Exception e) {
            System.out.println("Caught " + e);
        }
    }
    static void test(String testName, Consumer<Optional<String>> cos) {
        System.out.println(" === " + testName + " === ");
        cos.accept(Stream.of("Epithets").findFirst());
        cos.accept(Stream.<String>empty().findFirst());
    }
    public static void main(String[] args) {
        test("basics", Optionals::basics);
        test("ifPresent", Optionals::ifPresent);
        test("orElse", Optionals::orElse);
        test("orElseGet", Optionals::orElseGet);
        test("orElseThrow", Optionals::orElseThrow);
    }
}
```

輸出結果：

```
=== basics ===
Epithets
Nothing inside!
=== ifPresent ===
Epithets
=== orElse ===
Epithets
Nada
=== orElseGet ===
Epithets
Generated
=== orElseThrow ===
Epithets
Caught java.lang.Exception: Supplied
```

`test()` 透過傳入所有方法都適用的 **Consumer** 來避免重複程式碼。

`orElseThrow()` 通過 **catch** 關鍵字來捕獲拋出的異常。更多細節，將在 [異常](./15-Exceptions.md) 這一章節中學習。

<!-- Creating Optionals -->

### 建立 Optional

當我們在自己的程式碼中加入 **Optional** 時，可以使用下面 3 個靜態方法：

- `empty()`：生成一個空 **Optional**。
- `of(value)`：將一個非空值包裝到 **Optional** 裡。
- `ofNullable(value)`：針對一個可能為空的值，為空時自動生成 **Optional.empty**，否則將值包裝在 **Optional** 中。

下面來看看它是如何工作的。程式碼範例：

```java
// streams/CreatingOptionals.java
import java.util.*;
import java.util.stream.*;
import java.util.function.*;
class CreatingOptionals {
    static void test(String testName, Optional<String> opt) {
        System.out.println(" === " + testName + " === ");
        System.out.println(opt.orElse("Null"));
    }
    public static void main(String[] args) {
        test("empty", Optional.empty());
        test("of", Optional.of("Howdy"));
        try {
            test("of", Optional.of(null));
        } catch(Exception e) {
            System.out.println(e);
        }
        test("ofNullable", Optional.ofNullable("Hi"));
        test("ofNullable", Optional.ofNullable(null));
    }
}
```

輸出結果：

```
=== empty ===
Null
=== of ===
Howdy
java.lang.NullPointerException
=== ofNullable ===
Hi
=== ofNullable ===
Null
```

我們不能透過傳遞 `null` 到 `of()` 來建立 `Optional` 物件。最安全的方法是， 使用 `ofNullable()` 來優雅地處理 `null`。

### Optional 物件操作

當我們的流管道生成了 **Optional** 物件，下面 3 個方法可使得 **Optional** 的後續能做更多的操作：

- `filter(Predicate)`：對 **Optional** 中的內容應用**Predicate** 並將結果返回。如果 **Optional** 不滿足 **Predicate** ，將 **Optional** 轉化為空 **Optional** 。如果 **Optional** 已經為空，則直接返回空**Optional** 。

- `map(Function)`：如果 **Optional** 不為空，應用 **Function**  於 **Optional** 中的內容，並返回結果。否則直接返回 **Optional.empty**。

- `flatMap(Function)`：同 `map()`，但是提供的映射函數將結果包裝在 **Optional** 物件中，因此 `flatMap()` 不會在最後進行任何包裝。

以上方法都不適用於數值型 **Optional**。一般來說，流的 `filter()` 會在 **Predicate** 返回 `false` 時移除流元素。而 `Optional.filter()` 在失敗時不會刪除 **Optional**，而是將其保留下來，並轉化為空。下面請看程式碼範例：


```java
// streams/OptionalFilter.java
import java.util.*;
import java.util.stream.*;
import java.util.function.*;
class OptionalFilter {
    static String[] elements = {
            "Foo", "", "Bar", "Baz", "Bingo"
    };
    static Stream<String> testStream() {
        return Arrays.stream(elements);
    }
    static void test(String descr, Predicate<String> pred) {
        System.out.println(" ---( " + descr + " )---");
        for(int i = 0; i <= elements.length; i++) {
            System.out.println(
                    testStream()
                            .skip(i)
                            .findFirst()
                            .filter(pred));
        }
    }
    public static void main(String[] args) {
        test("true", str -> true);
        test("false", str -> false);
        test("str != \"\"", str -> str != "");
        test("str.length() == 3", str -> str.length() == 3);
        test("startsWith(\"B\")",
                str -> str.startsWith("B"));
    }
}
```

輸出結果：

```
---( true )---
Optional[Foo]
Optional[]
Optional[Bar]
Optional[Baz]
Optional[Bingo]
Optional.empty
---( false )---
Optional.empty
Optional.empty
Optional.empty
Optional.empty
Optional.empty
Optional.empty
---( str != "" )---
Optional[Foo]
Optional.empty
Optional[Bar]
Optional[Baz]
Optional[Bingo]
Optional.empty
---( str.length() == 3 )---
Optional[Foo]
Optional.empty
Optional[Bar]
Optional[Baz]
Optional.empty
Optional.empty
---( startsWith("B") )---
Optional.empty
Optional.empty
Optional[Bar]
Optional[Baz]
Optional[Bingo]
Optional.empty
```

即使輸出看起來像流，要特別注意 `test()` 中的 for 循環。每一次的for循環都重新啟動流，然後跳過for循環索引指定的數量的元素，這就是流只剩後續元素的原因。然後呼叫`findFirst()` 獲取剩餘元素中的第一個元素，並包裝在一個 `Optional`物件中。

**注意**，不同於普通 for 循環，這裡的索引值範圍並不是 `i < elements.length`， 而是 `i <= elements.length`。所以最後一個元素實際上超出了流。方便的是，這將自動成為 **Optional.empty**，你可以在每一個測試的結尾中看到。

同 `map()` 一樣 ， `Optional.map()` 執行一個函數。它僅在 **Optional** 不為空時才執行這個映射函數。並將 **Optional** 的內容提取出來，傳遞給映射函數。程式碼範例：

```java
// streams/OptionalMap.java
import java.util.Arrays;
import java.util.function.Function;
import java.util.stream.Stream;

class OptionalMap {
    static String[] elements = {"12", "", "23", "45"};

    static Stream<String> testStream() {
        return Arrays.stream(elements);
    }

    static void test(String descr, Function<String, String> func) {
        System.out.println(" ---( " + descr + " )---");
        for (int i = 0; i <= elements.length; i++) {
            System.out.println(
                    testStream()
                            .skip(i)
                            .findFirst() // Produces an Optional
                            .map(func));
        }
    }

    public static void main(String[] args) {
        // If Optional is not empty, map() first extracts
        // the contents which it then passes
        // to the function:
        test("Add brackets", s -> "[" + s + "]");
        test("Increment", s -> {
            try {
                return Integer.parseInt(s) + 1 + "";
            } catch (NumberFormatException e) {
                return s;
            }
        });
        test("Replace", s -> s.replace("2", "9"));
        test("Take last digit", s -> s.length() > 0 ?
                s.charAt(s.length() - 1) + "" : s);
    }
    // After the function is finished, map() wraps the
    // result in an Optional before returning it:
}
```

輸出結果：

```
---( Add brackets )---
Optional[[12]]
Optional[[]]
Optional[[23]]
Optional[[45]]
Optional.empty
---( Increment )---
Optional[13]
Optional[]
Optional[24]
Optional[46]
Optional.empty
---( Replace )---
Optional[19]
Optional[]
Optional[93]
Optional[45]
Optional.empty
---( Take last digit )---
Optional[2]
Optional[]
Optional[3]
Optional[5]
Optional.empty
```

映射函數的返回結果會自動包裝成為 **Optional**。**Optional.empty** 會被直接跳過。

**Optional** 的 `flatMap()` 應用於已生成 **Optional** 的映射函數，所以 `flatMap()` 不會像 `map()` 那樣將結果封裝在 **Optional** 中。程式碼範例：

```java
// streams/OptionalFlatMap.java
import java.util.Arrays;
import java.util.Optional;
import java.util.function.Function;
import java.util.stream.Stream;

class OptionalFlatMap {
    static String[] elements = {"12", "", "23", "45"};

    static Stream<String> testStream() {
        return Arrays.stream(elements);
    }

    static void test(String descr,
                     Function<String, Optional<String>> func) {
        System.out.println(" ---( " + descr + " )---");
        for (int i = 0; i <= elements.length; i++) {
            System.out.println(
                    testStream()
                            .skip(i)
                            .findFirst()
                            .flatMap(func));
        }
    }

    public static void main(String[] args) {
        test("Add brackets",
                s -> Optional.of("[" + s + "]"));
        test("Increment", s -> {
            try {
                return Optional.of(
                        Integer.parseInt(s) + 1 + "");
            } catch (NumberFormatException e) {
                return Optional.of(s);
            }
        });
        test("Replace",
                s -> Optional.of(s.replace("2", "9")));
        test("Take last digit",
                s -> Optional.of(s.length() > 0 ?
                        s.charAt(s.length() - 1) + ""
                        : s));
    }
}
```

輸出結果：

```
---( Add brackets )---
Optional[[12]]
Optional[[]]
Optional[[23]]
Optional[[45]]
Optional.empty
 ---( Increment )---
Optional[13]
Optional[]
Optional[24]
Optional[46]
Optional.empty
 ---( Replace )---
Optional[19]
Optional[]
Optional[93]
Optional[45]
Optional.empty
 ---( Take last digit )---
Optional[2]
Optional[]
Optional[3]
Optional[5]
Optional.empty
```

同 `map()`，`flatMap()` 將提取非空 **Optional** 的內容並將其應用在映射函數。唯一的區別就是 `flatMap()` 不會把結果包裝在 **Optional** 中，因為映射函數已經被包裝過了。在如上範例中，我們已經在每一個映射函數中顯式地完成了包裝，但是很顯然 `Optional.flatMap()` 是為那些自己已經生成 **Optional** 的函數而設計的。

<!-- Streams of Optionals -->
### Optional 流

假設你的生成器可能產生 `null` 值，那麼當用它來建立流時，你會自然地想到用  **Optional** 來包裝元素。如下是它的樣子，程式碼範例：

```java
// streams/Signal.java
import java.util.*;
import java.util.stream.*;
import java.util.function.*;
public class Signal {
    private final String msg;
    public Signal(String msg) { this.msg = msg; }
    public String getMsg() { return msg; }
    @Override
    public String toString() {
        return "Signal(" + msg + ")";
    }
    static Random rand = new Random(47);
    public static Signal morse() {
        switch(rand.nextInt(4)) {
            case 1: return new Signal("dot");
            case 2: return new Signal("dash");
            default: return null;
        }
    }
    public static Stream<Optional<Signal>> stream() {
        return Stream.generate(Signal::morse)
                .map(signal -> Optional.ofNullable(signal));
    }
}
```

當我們使用這個流的時候，必須要弄清楚如何解包 **Optional**。程式碼範例：

```java
// streams/StreamOfOptionals.java
import java.util.*;
import java.util.stream.*;
public class StreamOfOptionals {
    public static void main(String[] args) {
        Signal.stream()
                .limit(10)
                .forEach(System.out::println);
        System.out.println(" ---");
        Signal.stream()
                .limit(10)
                .filter(Optional::isPresent)
                .map(Optional::get)
                .forEach(System.out::println);
    }
}
```

輸出結果：

```java
Optional[Signal(dash)]
Optional[Signal(dot)]
Optional[Signal(dash)]
Optional.empty
Optional.empty
Optional[Signal(dash)]
Optional.empty
Optional[Signal(dot)]
Optional[Signal(dash)]
Optional[Signal(dash)]
---
Signal(dot)
Signal(dot)
Signal(dash)
Signal(dash)
```

在這裡，我們使用 `filter()` 來保留那些非空 **Optional**，然後在 `map()` 中使用 `get()` 獲取元素。由於每種情況都需要定義“空值”的含義，所以通常我們要為每個應用程式採用不同的方法。

<!-- Terminal Operations -->

## 終端操作


以下操作將會獲取流的最終結果。至此我們無法再繼續往後傳遞流。可以說，終端操作（Terminal Operations）總是我們在流管道中所做的最後一件事。

<!-- Convert to an Array -->

### 陣列
- `toArray()`：將流轉換成適當類型的陣列。
- `toArray(generator)`：在特殊情況下，生成自訂類型的陣列。

當我們需要得到陣列類型的資料以便於後續操作時，上面的方法就很有用。假設我們需要復用流產生的隨機數時，就可以這麼使用。程式碼範例:

```java
// streams/RandInts.java
package streams;
import java.util.*;
import java.util.stream.*;
public class RandInts {
    private static int[] rints = new Random(47).ints(0, 1000).limit(100).toArray();
    public static IntStream rands() {
        return Arrays.stream(rints);
    }
}
```

上例將100個數值範圍在 0 到 1000 之間的隨機數流轉換成為陣列並將其儲存在 `rints` 中。這樣一來，每次呼叫 `rands()` 的時候可以重複獲取相同的整數流。

<!-- Apply a Final Operation to Every Element -->
### 循環

- `forEach(Consumer)`常見如 `System.out::println` 作為 **Consumer** 函數。
- `forEachOrdered(Consumer)`： 保證 `forEach` 按照原始流順序操作。

第一種形式：無序操作，僅在引入並行流時才有意義。在 [並發編程](24-Concurrent-Programming.md) 章節之前我們不會深入研究這個問題。這裡簡單介紹一下 `parallel()`：可實現多處理器並行操作。實現原理為將流分割為多個（通常數目為 CPU 核心數）並在不同處理器上分別執行操作。因為我們採用的是內部疊代，而不是外部疊代，所以這是可能實現的。

`parallel()` 看似簡單，實則棘手。更多內容將在稍後的 [並發編程](24-Concurrent-Programming.md) 章節中學習。

下例引入 `parallel()` 來幫助理解 `forEachOrdered(Consumer)` 的作用和使用場景。程式碼範例：

```java
// streams/ForEach.java
import java.util.*;
import java.util.stream.*;
import static streams.RandInts.*;
public class ForEach {
    static final int SZ = 14;
    public static void main(String[] args) {
        rands().limit(SZ)
                .forEach(n -> System.out.format("%d ", n));
        System.out.println();
        rands().limit(SZ)
                .parallel()
                .forEach(n -> System.out.format("%d ", n));
        System.out.println();
        rands().limit(SZ)
                .parallel()
                .forEachOrdered(n -> System.out.format("%d ", n));
    }
}
```

輸出結果：

```
258 555 693 861 961 429 868 200 522 207 288 128 551 589
551 861 429 589 200 522 555 693 258 128 868 288 961 207
258 555 693 861 961 429 868 200 522 207 288 128 551 589
```

為了方便測試不同大小的流，我們抽離出了 `SZ` 變數。然而即使 `SZ` 值為14也產生了有趣的結果。在第一個流中，未使用 `parallel()` ，因此以元素從 `rands()`出來的順序輸出結果。在第二個流中，引入`parallel()` ，即便流很小，輸出的結果的順序也和前面不一樣。這是由於多處理器並行操作的原因，如果你將程式多執行幾次，你會發現輸出都不相同，這是多處理器並行操作的不確定性造成的結果。

在最後一個流中，同時使用了 `parallel()` 和 `forEachOrdered()` 來強制保持原始流順序。因此，對非並行流使用 `forEachOrdered()` 是沒有任何影響的。

<!-- Collecting -->

### 集合

- `collect(Collector)`：使用 **Collector** 收集流元素到結果集合中。
- `collect(Supplier, BiConsumer, BiConsumer)`：同上，第一個參數 **Supplier** 建立了一個新的結果集合，第二個參數 **BiConsumer** 將下一個元素收集到結果集合中，第三個參數 **BiConsumer** 用於將兩個結果集合合併起來。

在這裡我們只是簡單介紹了幾個 **Collectors** 的運用範例。實際上，它還有一些非常複雜的操作實現，可透過查看 `java.util.stream.Collectors` 的 API 文件了解。例如，我們可以將元素收集到任意一種特定的集合中。

假設我們現在為了保證元素有序，將元素儲存在 **TreeSet** 中。**Collectors** 裡面沒有特定的 `toTreeSet()`，但是我們可以透過將集合的建構子引用傳遞給 `Collectors.toCollection()`，從而構建任何類型的集合。下面我們來將一個文件中的單詞收集到 **TreeSet** 集合中。程式碼範例：

```java
// streams/TreeSetOfWords.java
import java.util.*;
import java.nio.file.*;
import java.util.stream.*;
public class TreeSetOfWords {
    public static void
    main(String[] args) throws Exception {
        Set<String> words2 =
                Files.lines(Paths.get("TreeSetOfWords.java"))
                        .flatMap(s -> Arrays.stream(s.split("\\W+")))
                        .filter(s -> !s.matches("\\d+")) // No numbers
                        .map(String::trim)
                        .filter(s -> s.length() > 2)
                        .limit(100)
                        .collect(Collectors.toCollection(TreeSet::new));
        System.out.println(words2);
    }
}
```

輸出結果：

```
[Arrays, Collectors, Exception, Files, Output, Paths,
Set, String, System, TreeSet, TreeSetOfWords, args,
class, collect, file, filter, flatMap, get, import,
java, length, limit, lines, main, map, matches, new,
nio, numbers, out, println, public, split, static,
stream, streams, throws, toCollection, trim, util,
void, words2]
```

**Files.**`lines()` 打開 **Path** 並將其轉換成為由行組成的流。下一行程式碼以一個或多個非單詞字元（`\\W+`）為分界，對每一行進行分割，結果是產生一個陣列，然後使用 **Arrays.**`stream()` 將陣列轉化成為流，最後`flatMap()`將各行形成的多個單詞流，扁平映射為一個單詞流。使用 `matches(\\d+)` 尋找並移除全部是數字的字串（注意,`words2` 是透過的）。然後用 **String.**`trim()` 去除單詞兩邊的空白，`filter()` 過濾所有長度小於3的單詞，並只獲取前100個單詞，最後將其儲存到 **TreeSet** 中。

我們也可以在流中生成 **Map**。程式碼範例：

```java
// streams/MapCollector.java
import java.util.*;
import java.util.stream.*;
class Pair {
    public final Character c;
    public final Integer i;
    Pair(Character c, Integer i) {
        this.c = c;
        this.i = i;
    }
    public Character getC() { return c; }
    public Integer getI() { return i; }
    @Override
    public String toString() {
        return "Pair(" + c + ", " + i + ")";
    }
}
class RandomPair {
    Random rand = new Random(47);
    // An infinite iterator of random capital letters:
    Iterator<Character> capChars = rand.ints(65,91)
            .mapToObj(i -> (char)i)
            .iterator();
    public Stream<Pair> stream() {
        return rand.ints(100, 1000).distinct()
                .mapToObj(i -> new Pair(capChars.next(), i));
    }
}
public class MapCollector {
    public static void main(String[] args) {
        Map<Integer, Character> map =
                new RandomPair().stream()
                        .limit(8)
                        .collect(
                                Collectors.toMap(Pair::getI, Pair::getC));
        System.out.println(map);
    }
}
```

輸出結果：

```
{688=W, 309=C, 293=B, 761=N, 858=N, 668=G, 622=F, 751=N}
```

**Pair** 只是一個基礎的資料物件。**RandomPair** 建立了隨機生成的 **Pair** 物件流。在 Java 中，我們不能直接以某種方式組合兩個流。所以我建立了一個整數流，並且使用 `mapToObj()` 將整數流轉化成為 **Pair** 流。 **capChars**的隨機大寫字母疊代器建立了流，然後`next()`讓我們可以在`stream()`中使用這個流。就我所知，這是將多個流組合成新的物件流的唯一方法。

在這裡，我們只使用最簡單形式的 `Collectors.toMap()`，這個方法只需要兩個從流中獲取鍵和值的函數。還有其他重載形式，其中一種當是鍵發生衝突時，使用一個函數來處理衝突。

大多數情況下，`java.util.stream.Collectors` 中預設的 **Collector** 就能滿足我們的要求。除此之外，你還可以使用第二種形式的 `collect()`。 我把它留作更進階的練習，下例給出基本用法：

```java
// streams/SpecialCollector.java
import java.util.*;
import java.util.stream.*;
public class SpecialCollector {
    public static void main(String[] args) throws Exception {
        ArrayList<String> words =
                FileToWords.stream("Cheese.dat")
                        .collect(ArrayList::new,
                                ArrayList::add,
                                ArrayList::addAll);
        words.stream()
                .filter(s -> s.equals("cheese"))
                .forEach(System.out::println);
    }
}
```

輸出結果：

```
cheese
cheese
```

在這裡， **ArrayList** 的方法已經做了你所需要的操作，但更有可能的是，如果你必須使用這種形式的 `collect()`，就要自己建立特定的定義。

<!-- Combining All Stream Elements -->

### 組合

- `reduce(BinaryOperator)`：使用 **BinaryOperator** 來組合所有流中的元素。因為流可能為空，其返回值為 **Optional**。
- `reduce(identity, BinaryOperator)`：功能同上，但是使用 **identity** 作為其組合的初始值。因此如果流為空，**identity** 就是結果。
- `reduce(identity, BiFunction, BinaryOperator)`：更複雜的使用形式（暫不介紹），這裡把它包含在內，因為它可以提高效率。通常，我們可以顯式地組合 `map()` 和 `reduce()` 來更簡單的表達它。

下面來看一下 `reduce` 的程式碼範例：

```java
// streams/Reduce.java
import java.util.*;
import java.util.stream.*;
class Frobnitz {
    int size;
    Frobnitz(int sz) { size = sz; }
    @Override
    public String toString() {
        return "Frobnitz(" + size + ")";
    }
    // Generator:
    static Random rand = new Random(47);
    static final int BOUND = 100;
    static Frobnitz supply() {
        return new Frobnitz(rand.nextInt(BOUND));
    }
}
public class Reduce {
    public static void main(String[] args) {
        Stream.generate(Frobnitz::supply)
                .limit(10)
                .peek(System.out::println)
                .reduce((fr0, fr1) -> fr0.size < 50 ? fr0 : fr1)
                .ifPresent(System.out::println);
    }
}
```

輸出結果：

```
Frobnitz(58)
Frobnitz(55)
Frobnitz(93)
Frobnitz(61)
Frobnitz(61)
Frobnitz(29)
Frobnitz(68)
Frobnitz(0)
Frobnitz(22)
Frobnitz(7)
Frobnitz(29)
```

**Frobnitz** 包含一個可生成自身的生成器 `supply()` ；因為 `supply()` 方法作為一個 `Supplier<Frobnitz>` 是簽名相容的，我們可以把 `supply()` 作為一個方法引用傳遞給 `Stream.generate()` （這種簽名相容性被稱作結構一致性）。我們使用了沒有“初始值”作為第一個參數的 `reduce()`方法，所以產生的結果是 **Optional** 類型。`Optional.ifPresent()` 方法只有在結果非空的時候才會呼叫 `Consumer<Frobnitz>` （`println` 方法可以被呼叫是因為 **Frobnitz** 可以透過 `toString()` 方法轉換成 **String**）。

Lambda 表達式中的第一個參數 `fr0` 是 `reduce()` 中上一次呼叫的結果。而第二個參數 `fr1` 是從流傳遞過來的值。

`reduce()` 中的 Lambda 表達式使用了三元表達式來獲取結果，當 `fr0` 的 `size` 值小於 50 的時候，將 `fr0` 作為結果，否則將序列中的下一個元素即 `fr1`作為結果。當取得第一個 `size` 值小於 50 的 `Frobnitz`，只要得到這個結果就會忽略流中其他元素。這是個非常奇怪的限制， 但也確實讓我們對 `reduce()` 有了更多的了解。

<!-- Matching -->
### 匹配

- `allMatch(Predicate)` ：如果流的每個元素提供給 **Predicate** 都返回 true ，結果返回為 true。在第一個 false 時，則停止執行計算。
- `anyMatch(Predicate)`：如果流的任意一個元素提供給 **Predicate** 返回 true ，結果返回為 true。在第一個 true 是停止執行計算。
- `noneMatch(Predicate)`：如果流的每個元素提供給 **Predicate** 都返回 false 時，結果返回為 true。在第一個 true 時停止執行計算。

我們已經在 `Prime.java` 中看到了 `noneMatch()` 的範例；`allMatch()` 和 `anyMatch()` 的用法基本上是等同的。下面我們來探究一下短路行為。為了消除冗餘程式碼，我們建立了 `show()`。首先我們必須知道如何統一地描述這三個匹配器的操作，然後再將其轉換為 **Matcher** 介面。程式碼範例：

```java
// streams/Matching.java
// Demonstrates short-circuiting of *Match() operations
import java.util.stream.*;
import java.util.function.*;
import static streams.RandInts.*;

interface Matcher extends BiPredicate<Stream<Integer>, Predicate<Integer>> {}
        
public class Matching {
    static void show(Matcher match, int val) {
        System.out.println(
                match.test(
                        IntStream.rangeClosed(1, 9)
                                .boxed()
                                .peek(n -> System.out.format("%d ", n)),
                        n -> n < val));
    }
    public static void main(String[] args) {
        show(Stream::allMatch, 10);
        show(Stream::allMatch, 4);
        show(Stream::anyMatch, 2);
        show(Stream::anyMatch, 0);
        show(Stream::noneMatch, 5);
        show(Stream::noneMatch, 0);
    }
}
```

輸出結果：

```
1 2 3 4 5 6 7 8 9 true
1 2 3 4 false
1 true
1 2 3 4 5 6 7 8 9 false
1 false
1 2 3 4 5 6 7 8 9 true
```

**BiPredicate** 是一個二元謂詞，它接受兩個參數並返回 true 或者 false。第一個參數是我們要測試的流，第二個參數是一個謂詞 **Predicate**。**Matcher** 可以匹配所有的 **Stream::\*Match** 方法，所以可以將每一個**Stream::\*Match**方法引用傳遞到 `show()` 中。對`match.test()` 的呼叫會被轉換成 對方法引用**Stream::\*Match** 的呼叫。

`show()` 接受一個**Matcher**和一個 `val` 參數，`val` 在判斷測試 `n < val`中指定了最大值。`show()` 方法生成了整數1-9組成的一個流。`peek()`用來展示在測試短路之前測試進行到了哪一步。從輸出中可以看到每次都發生了短路。

### 尋找

- `findFirst()`：返回第一個流元素的 **Optional**，如果流為空返回 **Optional.empty**。
- `findAny(`：返回含有任意流元素的 **Optional**，如果流為空返回 **Optional.empty**。

程式碼範例：

```java
// streams/SelectElement.java
import java.util.*;
import java.util.stream.*;
import static streams.RandInts.*;
public class SelectElement {
    public static void main(String[] args) {
        System.out.println(rands().findFirst().getAsInt());
        System.out.println(
                rands().parallel().findFirst().getAsInt());
        System.out.println(rands().findAny().getAsInt());
        System.out.println(
                rands().parallel().findAny().getAsInt());
    }
}
```

輸出結果：

```
258
258
258
242
```

無論流是否為並行化，`findFirst()` 總是會選擇流中的第一個元素。對於非並行流，`findAny()`會選擇流中的第一個元素（即使從定義上來看是選擇任意元素）。在這個例子中，用 `parallel()` 將流並行化，以展示 `findAny()` 不選擇流的第一個元素的可能性。

如果必須選擇流中最後一個元素，那就使用 `reduce()`。程式碼範例：

```java
// streams/LastElement.java
import java.util.*;
import java.util.stream.*;
public class LastElement {
    public static void main(String[] args) {
        OptionalInt last = IntStream.range(10, 20)
                .reduce((n1, n2) -> n2);
        System.out.println(last.orElse(-1));
        // Non-numeric object:
        Optional<String> lastobj =
                Stream.of("one", "two", "three")
                        .reduce((n1, n2) -> n2);
        System.out.println(
                lastobj.orElse("Nothing there!"));
    }
}
```

輸出結果：

```
19
three
```

`reduce()`  的參數只是用最後一個元素取代了最後兩個元素，最終只生成最後一個元素。如果為數字流，你必須使用相近的數字 **Optional** 類型（ numeric optional type），否則使用 **Optional** 類型，就像上例中的 `Optional<String>`。

<!-- Informational -->

### 訊息

- `count()`：流中的元素個數。
- `max(Comparator)`：根據所傳入的 **Comparator** 所決定的“最大”元素。
- `min(Comparator)`：根據所傳入的 **Comparator** 所決定的“最小”元素。

**String** 類型有預設的 **Comparator** 實現。程式碼範例：

```java
// streams/Informational.java
import java.util.stream.*;
import java.util.function.*;
public class Informational {
    public static void
    main(String[] args) throws Exception {
        System.out.println(
                FileToWords.stream("Cheese.dat").count());
        System.out.println(
                FileToWords.stream("Cheese.dat")
                        .min(String.CASE_INSENSITIVE_ORDER)
                        .orElse("NONE"));
        System.out.println(
                FileToWords.stream("Cheese.dat")
                        .max(String.CASE_INSENSITIVE_ORDER)
                        .orElse("NONE"));
    }
}
```

輸出結果：

```
32
a
you
```

`min()` 和 `max()` 的返回類型為 **Optional**，這需要我們使用 `orElse()`來解包。

<!-- Information for Numeric Streams -->
### 數字流訊息

- `average()` ：求取流元素平均值。
- `max()` 和 `min()`：數值流操作無需 **Comparator**。
- `sum()`：對所有流元素進行求和。
- `summaryStatistics()`：生成可能有用的資料。目前並不太清楚這個方法存在的必要性，因為我們其實可以用更直接的方法獲得需要的資料。

```java
// streams/NumericStreamInfo.java
import java.util.stream.*;
import static streams.RandInts.*;
public class NumericStreamInfo {
    public static void main(String[] args) {
        System.out.println(rands().average().getAsDouble());
        System.out.println(rands().max().getAsInt());
        System.out.println(rands().min().getAsInt());
        System.out.println(rands().sum());
        System.out.println(rands().summaryStatistics());
    }
}
```

輸出結果：

```
507.94
998
8
50794
IntSummaryStatistics{count=100, sum=50794, min=8, average=507.940000, max=998}
```

上例操作對於 **LongStream** 和 **DoubleStream** 同樣適用。

## 本章小結

流式操作改變並極大地提升了 Java 語言的可編程性，並可能極大地阻止了 Java 編程人員向諸如 Scala 這種函數式語言的流轉。在本書的剩餘部分，我們將儘可能地使用流。

[^1]: 在軟體或訊息建模的上下文中，快樂路徑(有時稱為快樂流)是沒有異常或錯誤條件的預設場景。例如，驗證信用卡號的函數的快樂路徑應該是任何驗證規則都不會出現錯誤的地方，從而讓執行成功地繼續到最後，生成一個積極的響應。[見 wikipedia: happy path](https://en.wikipedia.org/wiki/Happy_path)

<!-- 分頁 -->

<div style="page-break-after: always;"></div>
