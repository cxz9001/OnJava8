[TOC]

<!-- Strings -->

# 第十八章 字串
>字串操作毫無疑問是電腦程式設計中最常見的行為之一。

在 Java 大展身手的 Web 系統中更是如此。在本章中，我們將深入學習在 Java 語言中應用最廣泛的 `String` 類，並研究與之相關的類及工具。

<!-- Immutable Strings -->

## 字串的不可變
`String` 物件是不可變的。查看 JDK 文件你就會發現，`String` 類中每一個看起來會修改 `String` 值的方法，實際上都是建立了一個全新的 `String` 物件，以包含修改後的字串內容。而最初的 `String` 物件則絲毫未動。

看看下面的程式碼：
```java
// strings/Immutable.java
public class Immutable { 
    public static String upcase(String s) { 
        return s.toUpperCase(); 
    } 
    public static void main(String[] args) { 
        String q = "howdy";
        System.out.println(q); // howdy 
        String qq = upcase(q); 
        System.out.println(qq); // HOWDY 
        System.out.println(q); // howdy 
    } 
} 
/* Output: 
howdy
HOWDY 
howdy
*/ 
```
當把 `q` 傳遞給 `upcase()` 方法時，實際傳遞的是引用的一個複製。其實，每當把 String 物件作為方法的參數時，都會複製一份引用，而該引用所指向的物件其實一直待在單一的物理位置上，從未動過。

回到 `upcase()` 的定義，傳入其中的引用有了名字 `s`，只有 `upcase()` 執行的時候，局部引用 `s` 才存在。一旦 `upcase()` 執行結束，`s` 就消失了。當然了，`upcase()` 的返回值，其實是最終結果的引用。這足以說明，`upcase()` 返回的引用已經指向了一個新的物件，而 `q` 仍然在原來的位置。

`String` 的這種行為正是我們想要的。例如：

```java
String s = "asdf";
String x = Immutable.upcase(s);
```
難道你真的希望 `upcase()` 方法改變其參數嗎？對於一個方法而言，參數是為該方法提供訊息的，而不是想讓該方法改變自己的。在閱讀這段程式碼時，讀者自然會有這樣的感覺。這一點很重要，正是有了這種保障，才使得程式碼易於編寫和閱讀。


<!-- Overloading + vs. StringBuilder -->
## + 的重載與 StringBuilder
`String` 物件是不可變的，你可以給一個 `String` 物件添加任意多的別名。因為 `String` 是唯讀的，所以指向它的任何引用都不可能修改它的值，因此，也就不會影響到其他引用。

不可變性會帶來一定的效率問題。為 `String` 物件重載的 `+` 操作符就是一個例子。重載的意思是，一個操作符在用於特定的類時，被賦予了特殊的意義（用於 `String` 的 `+` 與 `+=` 是 Java 中僅有的兩個重載過的操作符，Java 不允許程式設計師重載任何其他的操作符 [^1]）。

操作符 `+` 可以用來連接 `String`：
```java
// strings/Concatenation.java

public class Concatenation {
    public static void main(String[] args) { 
        String mango = "mango"; 
        String s = "abc" + mango + "def" + 47; 
        System.out.println(s);
    } 
}
/* Output:
abcmangodef47 
*/
```
可以想像一下，這段程式碼是這樣工作的：`String` 可能有一個 `append()` 方法，它會生成一個新的 `String` 物件，以包含“abc”與 `mango` 連接後的字串。該物件會再建立另一個新的 `String` 物件，然後與“def”相連，生成另一個新的物件，依此類推。

這種方式當然是可行的，但是為了生成最終的 `String` 物件，會產生一大堆需要垃圾回收的中間物件。我猜想，Java 設計者一開始就是這麼做的（這也是軟體設計中的一個教訓：除非你用程式碼將系統實現，並讓它執行起來，否則你無法真正了解它會有什麼問題），然後他們發現其性能相當糟糕。

想看看以上程式碼到底是如何工作的嗎？可以用 JDK 自帶的 `javap` 工具來反編譯以上程式碼。指令如下：
```java
javap -c Concatenation
```
這裡的 `-c` 標誌表示將生成 JVM 位元組碼。我們剔除不感興趣的部分，然後做細微的修改，於是有了以下的位元組碼：
```x86asm
public static void main(java.lang.String[]); 
 Code:
  Stack=2, Locals=3, Args_size=1
  0: ldc #2; //String mango 
  2: astore_1 
  3: new #3; //class StringBuilder 
  6: dup 
  7: invokespecial #4; //StringBuilder."<init>":() 
  10: ldc #5; //String abc 
  12: invokevirtual #6; //StringBuilder.append:(String) 
  15: aload_1 
  16: invokevirtual #6; //StringBuilder.append:(String) 
  19: ldc #7; //String def 
  21: invokevirtual #6; //StringBuilder.append:(String) 
  24: bipush 47 
  26: invokevirtual #8; //StringBuilder.append:(I) 
  29: invokevirtual #9; //StringBuilder.toString:() 
  32: astore_2 
  33: getstatic #10; //Field System.out:PrintStream;
  36: aload_2 
  37: invokevirtual #11; //PrintStream.println:(String) 
  40: return
```
如果你有組語語言的經驗，以上程式碼應該很眼熟(其中的 `dup` 和 `invokevirtual` 語句相當於Java虛擬機上的組語語句。即使你完全不了解組語語言也無需擔心)。需要重點注意的是：編譯器自動引入了 `java.lang.StringBuilder` 類。雖然原始碼中並沒有使用 `StringBuilder` 類，但是編譯器卻自作主張地使用了它，就因為它更高效。

在這裡，編譯器建立了一個 `StringBuilder` 物件，用於構建最終的 `String`，並對每個字串呼叫了一次 `append()` 方法，共計 4 次。最後呼叫 `toString()` 生成結果，並存為 `s` (使用的指令為 `astore_2`)。

現在，也許你會覺得可以隨意使用 `String` 物件，反正編譯器會自動為你做性能最佳化。可是在這之前，讓我們更深入地看看編譯器能為我們最佳化到什麼程度。下面的例子採用兩種方式生成一個 `String`：方法一使用了多個 `String` 物件；方法二在程式碼中使用了 `StringBuilder`。
```java
// strings/WhitherStringBuilder.java

public class WhitherStringBuilder { 
    public String implicit(String[] fields) { 
        String result = ""; 
        for(String field : fields) { 
            result += field;
        }
        return result; 
    }
    public String explicit(String[] fields) { 
        StringBuilder result = new StringBuilder(); 
        for(String field : fields) { 
            result.append(field); 
        } 
        return result.toString(); 
    }
}
```
現在執行 `javap -c WhitherStringBuilder`，可以看到兩種不同方法（我已經去掉不相關的細節）對應的位元組碼。首先是 `implicit()` 方法：
```x86asm
public java.lang.String implicit(java.lang.String[]); 
0: ldc #2 // String 
2: astore_2
3: aload_1 
4: astore_3 
5: aload_3 
6: arraylength 
7: istore 4 
9: iconst_0 
10: istore 5 
12: iload 5 
14: iload 4 
16: if_icmpge 51 
19: aload_3 
20: iload 5 
22: aaload 
23: astore 6 
25: new #3 // StringBuilder 
28: dup 
29: invokespecial #4 // StringBuilder."<init>"
32: aload_2 
33: invokevirtual #5 // StringBuilder.append:(String) 
36: aload 6 
38: invokevirtual #5 // StringBuilder.append:(String;) 
41: invokevirtual #6 // StringBuilder.toString:() 
44: astore_2 
45: iinc 5, 1 
48: goto 12 
51: aload_2 
52: areturn
```
注意從第 16 行到第 48 行構成了一個循環體。第 16 行：對堆疊中的運算元進行“大於或等於的整數比較運算”，循環結束時跳轉到第 51 行。第 48 行：重新回到循環體的起始位置（第 12 行）。注意：`StringBuilder` 是在循環內構造的，這意味著每進行一次循環，會建立一個新的 `StringBuilder` 物件。

下面是 `explicit()` 方法對應的位元組碼：
```x86asm
public java.lang.String explicit(java.lang.String[]); 
0: new #3 // StringBuilder 
3: dup
4: invokespecial #4 // StringBuilder."<init>" 
7: astore_2 
8: aload_1 
9: astore_3 
10: aload_3 
11: arraylength 
12: istore 4 
14: iconst_0 
15: istore 5 
17: iload 5 
19: iload 4 
21: if_icmpge 43 
24: aload_3 
25: iload 5 
27: aaload 
28: astore 6 
30: aload_2 
31: aload 6 
33: invokevirtual #5 // StringBuilder.append:(String) 
36: pop
37: iinc 5, 1 
40: goto 17 
43: aload_2 
44: invokevirtual #6 // StringBuilder.toString:() 
47: areturn
```
可以看到，不僅循環部分的程式碼更簡短、更簡單，而且它只生成了一個 `StringBuilder` 物件。顯式地建立 `StringBuilder` 還允許你預先為其指定大小。如果你已經知道最終字串的大概長度，那預先指定 `StringBuilder` 的大小可以避免頻繁地重新分配緩衝。

因此，當你為一個類編寫 `toString()` 方法時，如果字串操作比較簡單，那就可以信賴編譯器，它會為你合理地構造最終的字串結果。但是，如果你要在 `toString()` 方法中使用循環，且可能有性能問題，那麼最好自己建立一個 `StringBuilder` 物件，用它來構建最終結果。請參考以下範例：
```java
// strings/UsingStringBuilder.java 

import java.util.*; 
import java.util.stream.*; 
public class UsingStringBuilder { 
    public static String string1() { 
        Random rand = new Random(47);
        StringBuilder result = new StringBuilder("["); 
        for(int i = 0; i < 25; i++) { 
            result.append(rand.nextInt(100)); 
            result.append(", "); 
        } 
        result.delete(result.length()-2, result.length()); 
        result.append("]");
        return result.toString(); 
    } 
    public static String string2() { 
        String result = new Random(47)
            .ints(25, 0, 100)
            .mapToObj(Integer::toString)
            .collect(Collectors.joining(", "));
        return "[" + result + "]"; 
    } 
    public static void main(String[] args) { 
        System.out.println(string1()); 
        System.out.println(string2()); 
    }
} 
/* Output: 
[58, 55, 93, 61, 61, 29, 68, 0, 22, 7, 88, 28, 51, 89, 
9, 78, 98, 61, 20, 58, 16, 40, 11, 22, 4] 
[58, 55, 93, 61, 61, 29, 68, 0, 22, 7, 88, 28, 51, 89,
9, 78, 98, 61, 20, 58, 16, 40, 11, 22, 4] 
*/ 
```
在方法 `string1()` 中，最終結果是用 `append()` 語句拼接起來的。如果你想走捷徑，例如：`append(a + ": " + c)`，編譯器就會掉入陷阱，從而為你另外建立一個 `StringBuilder` 物件處理括號內的字串操作。如果不確定該用哪種方式，隨時可以用 `javap` 來分析你的程式。

`StringBuilder` 提供了豐富而全面的方法，包括 `insert()`、`replace()`、`substring()`，甚至還有`reverse()`，但是最常用的還是 `append()` 和 `toString()`。還有 `delete()`，上面的例子中我們用它刪除最後一個逗號和空格，以便添加右括號。

`string2()` 使用了 `Stream`，這樣程式碼更加簡潔美觀。可以證明，`Collectors.joining()` 內部也是使用的 `StringBuilder`，這種寫法不會影響性能！

`StringBuilder `是 Java SE5 引入的，在這之前用的是 `StringBuffer`。後者是執行緒安全的（參見[並發編程](./24-Concurrent-Programming.md)），因此開銷也會大些。使用 `StringBuilder` 進行字串操作更快一點。

<!-- Unintended Recursion -->

## 意外遞迴
Java 中的每個類從根本上都是繼承自 `Object`，標準集合類也是如此，它們都有 `toString()` 方法，並且覆蓋了該方法，使得它生成的 `String` 結果能夠表達集合自身，以及集合包含的物件。例如 `ArrayList.toString()`，它會遍歷 `ArrayList` 中包含的所有物件，呼叫每個元素上的 `toString()` 方法：
```java
// strings/ArrayListDisplay.java 
import java.util.*;
import java.util.stream.*; 
import generics.coffee.*;
public class ArrayListDisplay { 
    public static void main(String[] args) {
        List<Coffee> coffees = 
            Stream.generate(new CoffeeSupplier())
                .limit(10)
                .collect(Collectors.toList()); 
        System.out.println(coffees); 
    } 
}
/* Output: 
[Americano 0, Latte 1, Americano 2, Mocha 3, Mocha 4, 
Breve 5, Americano 6, Latte 7, Cappuccino 8, Cappuccino 9] 
*/ 
```
如果你希望 `toString()` 列印出類的記憶體地址，也許你會考慮使用 `this` 關鍵字：
```java
// strings/InfiniteRecursion.java 
// Accidental recursion 
// {ThrowsException} 
// {VisuallyInspectOutput} Throws very long exception
import java.util.*;
import java.util.stream.*;

public class InfiniteRecursion { 
    @Override 
    public String toString() { 
        return " InfiniteRecursion address: " + this + "\n";
    } 
    public static void main(String[] args) { 
        Stream.generate(InfiniteRecursion::new) 
            .limit(10) 
            .forEach(System.out::println); 
    } 
} 
```
當你建立了 `InfiniteRecursion` 物件，並將其列印出來的時候，你會得到一串很長的異常訊息。如果你將該 `InfiniteRecursion` 物件存入一個 `ArrayList` 中，然後列印該 `ArrayList`，同樣也會拋出異常。其實，當執行到如下程式碼時：
```java
"InfiniteRecursion address: " + this 
```
這裡發生了自動類型轉換，由 `InfiniteRecursion` 類型轉換為 `String` 類型。因為編譯器發現一個 `String` 物件後面跟著一個 “+”，而 “+” 後面的物件不是 `String`，於是編譯器試著將 `this` 轉換成一個 `String`。它怎麼轉換呢？正是透過呼叫 `this` 上的 `toString()` 方法，於是就發生了遞迴呼叫。

如果你真的想要列印物件的記憶體地址，應該呼叫 `Object.toString()` 方法，這才是負責此任務的方法。所以，不要使用 `this`，而是應該呼叫 `super.toString()` 方法。


<!-- Operations on Strings -->
## 字串操作
以下是 `String` 物件具備的一些基本方法。重載的方法歸納在同一行中：

| 方法 | 參數，重載版本 | 作用 |
| ---- | ---- | ---- |
| 構造方法 | 預設版本，`String`，`StringBuilder`，`StringBuffer`，`char`陣列，`byte`陣列 | 建立`String`物件 |
| `length()` | | `String`中字元的個數 |
| `charAt()` | `int`索引|獲取`String`中索引位置上的`char` |
| `getChars()`，`getBytes()` | 待複製部分的開始和結束索引，複製的目標陣列，目標陣列的開始索引 | 複製`char`或`byte`到一個目標陣列中 |
| `toCharArray()` | | 生成一個`char[]`，包含`String`中的所有字元 |
| `equals()`，`equalsIgnoreCase()` | 與之進行比較的`String` | 比較兩個`String`的內容是否相同。如果相同，結果為`true` |
| `compareTo()`，`compareToIgnoreCase()` | 與之進行比較的`String` | 按詞典順序比較`String`的內容，比較結果為負數、零或正數。注意，大小寫不等價 |
| `contains()` | 要搜尋的`CharSequence` | 如果該`String`物件包含參數的內容，則返回`true` |
| `contentEquals()` | 與之進行比較的`CharSequence`或`StringBuffer` | 如果該`String`物件與參數的內容完全一致，則返回`true` |
| `isEmpty()` | | 返回`boolean`結果，以表明`String`物件的長度是否為0 |
| `regionMatches()` | 該`String`的索引偏移量，另一個`String`及其索引偏移量，要比較的長度。重載版本增加了“忽略大小寫”功能|返回`boolean`結果，以表明所比較區域是否相等 |
| `startsWith()` | 可能的起始`String`。重載版本在參數中增加了偏移量 | 返回`boolean`結果，以表明該`String`是否以傳入參數開始 |
| `endsWith()` | 該`String`可能的後綴`String` | 返回`boolean`結果，以表明此參數是否是該字串的後綴 |
| `indexOf()`，`lastIndexOf()` | 重載版本包括：`char`，`char`與起始索引，`String`，`String`與起始索引 | 如果該`String`並不包含此參數，就返回-1；否則返回此參數在`String`中的起始索引。`lastIndexOf`()是從後往前搜尋 |
| `matches()` | 一個正規表示式 | 返回`boolean`結果，以表明該`String`和給出的正規表示式是否匹配 |
| `split()` | 一個正規表示式。可選參數為需要分割的最大數量 | 按照正規表示式分割`String`，返回一個結果陣列 |
| `join()`（Java8引入的）|分隔符，待拼字元序列。用分隔符將字元序列拼接成一個新的`String` |用分隔符拼接字元片段，產生一個新的`String` |
| `substring()`（即`subSequence()`）| 重載版本：起始索引；起始索引+終止索引 | 返回一個新的`String`物件，以包含參數指定的子串 |
| `concat()` | 要連接的`String` |返回一個新的`String`物件，內容為原始`String`連接上參數`String` |
| `replace()` | 要取代的字元，用來進行取代的新字元。也可以用一個`CharSequence`取代另一個`CharSequence` |返回取代字元後的新`String`物件。如果沒有取代發生，則返回原始的`String`物件 |
| `replaceFirst()` |要取代的正規表示式，用來進行取代的`String` | 返回取代首個目標字串後的`String`物件 |
| `replaceAll()` | 要取代的正規表示式，用來進行取代的`String` | 返回取代所有目標字串後的`String`物件 |
| `toLowerCase()`，`toUpperCase()` | | 將字元的大小寫改變後，返回一個新的`String`物件。如果沒有任何改變，則返回原始的`String`物件|
| `trim()` | |將`String`兩端的空白符刪除後，返回一個新的`String`物件。如果沒有任何改變，則返回原始的`String`物件|
| `valueOf()`（`static`）|重載版本：`Object`；`char[]`；`char[]`，偏移量，與字元個數；`boolean`；`char`；`int`；`long`；`float`；`double` | 返回一個表示參數內容的`String` |
| `intern()` | | 為每個唯一的字元序列生成一個且僅生成一個`String`引用 |
| `format()` | 要格式化的字串，要取代到格式化字串的參數 | 返回格式化結果`String` |

從這個表可以看出，當需要改變字串的內容時，`String` 類的方法都會返回一個新的 `String` 物件。同時，如果內容不改變，`String` 方法只是返回原始物件的一個引用而已。這可以節約儲存空間以及避免額外的開銷。

本章稍後還將介紹正規表示式在 `String` 方法中的應用。

<!-- Formatting Output -->

## 格式化輸出
在長久的等待之後，Java SE5 終於推出了 C 語言中 `printf()` 風格的格式化輸出這一功能。這不僅使得控制輸出的程式碼更加簡單，同時也給與Java開發者對於輸出格式與排列更強大的控制能力。
### `printf()`
C 語言的 `printf()` 並不像 Java 那樣連接字串，它使用一個簡單的格式化字串，加上要插入其中的值，然後將其格式化輸出。 `printf()` 並不使用重載的 `+` 操作符（C語言沒有重載）來連接引號內的字串或字串變數，而是使用特殊的占位符來表示資料將來的位置。而且它還將插入格式化字串的參數，以逗號分隔，排成一行。例如：
```c
System.out.printf("Row 1: [%d %f]%n", x, y);
```
這一行程式碼在執行的時候，首先將 `x` 的值插入到 `%d_` 的位置，然後將 `y` 的值插入到 `%f` 的位置。這些占位符叫做*格式修飾符*，它們不僅指明了插入資料的位置，同時還指明了將會插入什麼類型的變數，以及如何格式化。在這個例子中 `%d` 表示 `x` 是一個整數，`%f` 表示 `y` 是一個浮點數（`float` 或者 `double`）。
### `System.out.format()`
Java SE5 引入了 `format()` 方法，可用於 `PrintStream` 或者 `PrintWriter` 物件（你可以在 [附錄:流式 I/O](./Appendix-IO-Streams.md) 了解更多內容），其中也包括 `System.out` 物件。`format()` 方法模仿了 C 語言的 `printf()`。如果你比較懷舊的話，也可以使用 `printf()`。以下是一個簡單的範例：
```java
// strings/SimpleFormat.java 

public class SimpleFormat {   
    public static void main(String[] args) {     
        int x = 5;     
        double y = 5.332542;     
        // The old way: 
        System.out.println("Row 1: [" + x + " " + y + "]");     
        // The new way:     
        System.out.format("Row 1: [%d %f]%n", x, y);     
        // or     
        System.out.printf("Row 1: [%d %f]%n", x, y);   
    } 
} 
/* Output: 
Row 1: [5 5.332542] 
Row 1: [5 5.332542] 
Row 1: [5 5.332542] 
*/
```
可以看到，`format()` 和  `printf()` 是等價的，它們只需要一個簡單的格式化字串，加上一串參數即可，每個參數對應一個格式修飾符。

`String` 類也有一個 `static format()` 方法，可以格式化字串。

### `Formatter` 類
在 Java 中，所有的格式化功能都是由 `java.util.Formatter` 類處理的。可以將 `Formatter` 看做一個翻譯器，它將你的格式化字串與資料翻譯成需要的結果。當你建立一個 `Formatter` 物件時，需要向其構造器傳遞一些訊息，告訴它最終的結果將向哪裡輸出：
```java
// strings/Turtle.java 
import java.io.*;
import java.util.*;

public class Turtle {   
    private String name;   
    private Formatter f;  
    public Turtle(String name, Formatter f) {
        this.name = name;     
        this.f = f;   
    }   
    public void move(int x, int y) {     
        f.format("%s The Turtle is at (%d,%d)%n",       
            name, x, y);   
    }
    public static void main(String[] args) {    
        PrintStream outAlias = System.out;     
        Turtle tommy = new Turtle("Tommy",
            new Formatter(System.out));     
        Turtle terry = new Turtle("Terry",       
            new Formatter(outAlias));     
        tommy.move(0,0);     
        terry.move(4,8);     
        tommy.move(3,4);     
        terry.move(2,5);     
        tommy.move(3,3);     
        terry.move(3,3);   
    } 
} 
/* Output: 
Tommy The Turtle is at (0,0) 
Terry The Turtle is at (4,8) 
Tommy The Turtle is at (3,4) 
Terry The Turtle is at (2,5) 
Tommy The Turtle is at (3,3) 
Terry The Turtle is at (3,3) 
*/
```
格式化修飾符 `%s` 表明這裡需要 `String` 參數。

所有的 `tommy` 都將輸出到 `System.out`，而所有的 `terry` 則都輸出到 `System.out` 的一個別名中。`Formatter` 的重載構造器支援輸出到多個路徑，不過最常用的還是 `PrintStream()`（如上例）、`OutputStream` 和 `File`。你可以在 [附錄:流式 I/O](././Appendix-IO-Streams.md) 中了解更多訊息。
### 格式化修飾符
在插入資料時，如果想要最佳化空格與對齊，你需要更精細複雜的格式修飾符。以下是其通用語法：
```java
%[argument_index$][flags][width][.precision]conversion 
```
最常見的應用是控制一個欄位的最小長度，這可以透過指定 *width* 來實現。`Formatter `物件透過在必要時添加空格，來確保一個欄位至少達到設定長度。預設情況下，資料是右對齊的，不過可以透過使用 `-` 標誌來改變對齊方向。

與 *width* 相對的是 *precision*，用於指定最大長度。*width* 可以應用於各種類型的資料轉換，並且其行為方式都一樣。*precision* 則不然，當應用於不同類型的資料轉換時，*precision* 的意義也不同。在將 *precision* 應用於 `String` 時，它表示列印 `string` 時輸出字元的最大數量。而在將 *precision* 應用於浮點數時，它表示小數部分要顯示出來的位數（預設是 6 位小數），如果小數位數過多則舍入，太少則在尾部補零。由於整數沒有小數部分，所以 *precision* 無法應用於整數，如果你對整數應用 *precision*，則會觸發異常。

下面的程式應用格式修飾符來列印一個購物收據。這是 *Builder* 設計模式的一個簡單實現，即先建立一個初始物件，然後逐漸添加新東西，最後呼叫 `build()` 方法完成構建：
```java
// strings/ReceiptBuilder.java 
import java.util.*; 

public class ReceiptBuilder {   
    private double total = 0;   
    private Formatter f =     
        new Formatter(new StringBuilder());   
    public ReceiptBuilder() {     
        f.format(       
          "%-15s %5s %10s%n", "Item", "Qty", "Price");     
        f.format(       
          "%-15s %5s %10s%n", "----", "---", "-----");   
        }   
    public void add(String name, int qty, double price) {     
        f.format("%-15.15s %5d %10.2f%n", name, qty, price);     
        total += price * qty;   
    }  
    public String build() {     
        f.format("%-15s %5s %10.2f%n", "Tax", "",       
          total * 0.06);     
        f.format("%-15s %5s %10s%n", "", "", "-----");     
        f.format("%-15s %5s %10.2f%n", "Total", "",       
          total * 1.06);     
        return f.toString();   
    }   
    public static void main(String[] args) {     
        ReceiptBuilder receiptBuilder =       
          new ReceiptBuilder();     
        receiptBuilder.add("Jack's Magic Beans", 4, 4.25);     
        receiptBuilder.add("Princess Peas", 3, 5.1);     
        receiptBuilder.add(       
          "Three Bears Porridge", 1, 14.29);     
        System.out.println(receiptBuilder.build());   
    } 
} 
/* Output: 
Item              Qty      Price 
----              ---      ----- 
Jack's Magic Be     4       4.25 
Princess Peas       3       5.10 
Three Bears Por     1      14.29 
Tax                         2.80 
                           ----- 
Total                      49.39 
*/ 
```
透過傳入一個 `StringBuilder` 物件到 `Formatter` 的構造器，我們指定了一個容器來構建目標 `String`。你也可以透過不同的構造器參數，把結果輸出到標準輸出，甚至是一個文件裡。

正如你所見，透過相當簡潔的語法，`Formatter ` 提供了對空格與對齊的強大控制能力。在該程式中，為了恰當地控制間隔，格式化字串被重複利用了多遍。
### `Formatter` 轉換
下面的表格展示了最常用的類型轉換：

| 類型 | 含義 |
| :----: | :---- |
| `d` | 整型（十進位制） |
| `c` | Unicode字元 |
| `b` | Boolean值 |
| `s` | String |
| `f` | 浮點數（十進位制） |
| `e` | 浮點數（科學計數） |
| `x` | 整型（十六進位制） |
| `h` | 散列碼（十六進位制） |
| `%` | 字面值“%” |

下面的程式示範了這些轉換是如何工作的：

```java
// strings/Conversion.java 
import java.math.*;
import java.util.*; 

public class Conversion {   
    public static void main(String[] args) {     
        Formatter f = new Formatter(System.out); 

        char u = 'a';     
        System.out.println("u = 'a'");     
        f.format("s: %s%n", u);     
        // f.format("d: %d%n", u);     
        f.format("c: %c%n", u);     
        f.format("b: %b%n", u);     
        // f.format("f: %f%n", u);     
        // f.format("e: %e%n", u);     
        // f.format("x: %x%n", u);     
        f.format("h: %h%n", u); 

        int v = 121;     
        System.out.println("v = 121");    
        f.format("d: %d%n", v);     
        f.format("c: %c%n", v);     
        f.format("b: %b%n", v);     
        f.format("s: %s%n", v);     
        // f.format("f: %f%n", v);     
        // f.format("e: %e%n", v);     
        f.format("x: %x%n", v);     
        f.format("h: %h%n", v); 

        BigInteger w = new BigInteger("50000000000000");     
        System.out.println(       
          "w = new BigInteger(\"50000000000000\")");     
        f.format("d: %d%n", w);     
        // f.format("c: %c%n", w);     
        f.format("b: %b%n", w);     
        f.format("s: %s%n", w);     
        // f.format("f: %f%n", w);     
        // f.format("e: %e%n", w);     
        f.format("x: %x%n", w);     
        f.format("h: %h%n", w); 

        double x = 179.543;     
        System.out.println("x = 179.543");     
        // f.format("d: %d%n", x);     
        // f.format("c: %c%n", x);     
        f.format("b: %b%n", x);     
        f.format("s: %s%n", x);     
        f.format("f: %f%n", x);     
        f.format("e: %e%n", x);     
        // f.format("x: %x%n", x);     
        f.format("h: %h%n", x); 

        Conversion y = new Conversion();     
        System.out.println("y = new Conversion()"); 

        // f.format("d: %d%n", y);     
        // f.format("c: %c%n", y);     
        f.format("b: %b%n", y);     
        f.format("s: %s%n", y);     
        // f.format("f: %f%n", y);     
        // f.format("e: %e%n", y);     
        // f.format("x: %x%n", y);     
        f.format("h: %h%n", y); 

        boolean z = false;     
        System.out.println("z = false");     
        // f.format("d: %d%n", z);     
        // f.format("c: %c%n", z);     
        f.format("b: %b%n", z);     
        f.format("s: %s%n", z);     
        // f.format("f: %f%n", z);     
        // f.format("e: %e%n", z);     
        // f.format("x: %x%n", z);     
        f.format("h: %h%n", z);   
    } 
} 
/* Output: 
u = 'a' 
s: a 
c: a 
b: true 
h: 61 
v = 121 
d: 121 
c: y 
b: true 
s: 121 
x: 79 
h: 79 
w = new BigInteger("50000000000000") 
d: 50000000000000 
b: true 
s: 50000000000000 
x: 2d79883d2000 
h: 8842a1a7 
x = 179.543 
b: true 
s: 179.543 
f: 179.543000 
e: 1.795430e+02 
h: 1ef462c 
y = new Conversion() 
b: true 
s: Conversion@15db9742 
h: 15db9742 
z = false 
b: false 
s: false
h: 4d5 
*/
```
被注釋的程式碼表示，針對相應類型的變數，這些轉換是無效的。如果執行這些轉換，則會觸發異常。

注意，程式中的每個變數都用到了 `b` 轉換。雖然它對各種類型都是合法的，但其行為卻不一定與你想像的一致。對於 `boolean` 基本類型或 `Boolean` 物件，其轉換結果是對應的 `true` 或 `false`。但是，對其他類型的參數，只要該參數不為 `null`，其轉換結果永遠都是 `true`。即使是數字 0，轉換結果依然為 `true`，而這在其他語言中（包括C），往往轉換為 `false`。所以，將 `b` 應用於非布爾類型的物件時請格外小心。

還有許多不常用的類型轉換與格式修飾符選項，你可以在 JDK 文件中的 `Formatter` 類部分找到它們。
### `String.format()`
Java SE5 也參考了 C 中的 `sprintf()` 方法，以生成格式化的 `String` 物件。`String.format()` 是一個 `static` 方法，它接受與 `Formatter.format()` 方法一樣的參數，但返回一個 `String` 物件。當你只需使用一次 `format()` 方法的時候，`String.format()` 用起來很方便。例如：
```java
// strings/DatabaseException.java 

public class DatabaseException extends Exception {   
    public DatabaseException(int transactionID,     
      int queryID, String message) {     
      super(String.format("(t%d, q%d) %s", transactionID,         
        queryID, message));   
    }   
    public static void main(String[] args) {     
      try {       
        throw new DatabaseException(3, 7, "Write failed");     
      } catch(Exception e) {       
        System.out.println(e);     
      }   
    } 
} 
/* 
Output: 
DatabaseException: (t3, q7) Write failed 
*/
```
其實在 `String.format()` 內部，它也是建立了一個 `Formatter` 物件，然後將你傳入的參數轉給 `Formatter`。不過，與其自己做這些事情，不如使用便捷的 `String.format()` 方法，何況這樣的程式碼更清晰易讀。
#### 一個十六進位制轉儲（dump）工具
在第二個例子中，我們把二進位制文件轉換為十六進位制格式。下面的小工具使用了 `String.format()` 方法，以可讀的十六進位制格式將位元組陣列列印出來：
```java
// strings/Hex.java 
// {java onjava.Hex} 
package onjava;
import java.io.*; 
import java.nio.file.*; 

public class Hex {   
    public static String format(byte[] data) {     
        StringBuilder result = new StringBuilder();     
        int n = 0;     
        for(byte b : data) {       
            if(n % 16 == 0)         
                result.append(String.format("%05X: ", n));       
            result.append(String.format("%02X ", b));       
            n++;       
            if(n % 16 == 0) result.append("\n");     
        }     
        result.append("\n");     
        return result.toString();   
    }   
    public static void main(String[] args) throws Exception {  
        if(args.length == 0)       
            // Test by displaying this class file:       
            System.out.println(format(         
                Files.readAllBytes(Paths.get(           
                  "build/classes/main/onjava/Hex.class"))));     
        else       
            System.out.println(format(         
                Files.readAllBytes(Paths.get(args[0]))));   
    } 
} 
/* Output: (First 6 Lines) 
00000: CA FE BA BE 00 00 00 34 00 61 0A 00 05 00 31 07 
00010: 00 32 0A 00 02 00 31 08 00 33 07 00 34 0A 00 35 
00020: 00 36 0A 00 0F 00 37 0A 00 02 00 38 08 00 39 0A 
00030: 00 3A 00 3B 08 00 3C 0A 00 02 00 3D 09 00 3E 00 
00040: 3F 08 00 40 07 00 41 0A 00 42 00 43 0A 00 44 00 
00050: 45 0A 00 14 00 46 0A 00 47 00 48 07 00 49 01 00
                  ... 
*/
```
為了打開及讀入二進位制文件，我們用到了另一個工具 `Files.readAllBytes()`，這已經在 [Files章節](./17-Files.md) 介紹過了。這裡的 `readAllBytes()` 方法將整個文件以 `byte` 陣列的形式返回。

<!-- Regular Expressions -->

## 正規表示式
很久之前，*正規表示式*就已經整合到標準 Unix 工具集之中，例如 sed、awk 和程式語言之中了，如 Python 和Perl（有些人認為正是正規表示式促成了 Perl 的成功）。而在 Java 中，字串操作還主要集中於`String`、`StringBuffer` 和 `StringTokenizer` 類。與正規表示式相比較，它們只能提供相當簡單的功能。

正規表示式是一種強大而靈活的文字處理工具。使用正規表示式，我們能夠以編程的方式，構造複雜的文字模式，並對輸入 `String` 進行搜尋。一旦找到了匹配這些模式的部分，你就能隨心所欲地對它們進行處理。初學正規表示式時，其語法是一個難點，但它確實是一種簡潔、動態的語言。正規表示式提供了一種完全通用的方式，能夠解決各種 `String` 處理相關的問題：匹配、選擇、編輯以及驗證。
### 基礎
一般來說，正規表示式就是以某種方式來描述字串，因此你可以說：“如果一個字串含有這些東西，那麼它就是我正在找的東西。”例如，要找一個數字，它可能有一個負號在最前面，那你就寫一個負號加上一個問號，就像這樣：
```java
-?
```
要描述一個整數，你可以說它有一位或多位阿拉伯數字。在正規表示式中，用 `\d` 表示一位數字。如果在其他語言中使用過正規表示式，那你可能就能發現 Java 對反斜線 \ 的不同處理方式。在其他語言中，`\\` 表示“我想要在正規表示式中插入一個普通的（字面上的）反斜線，請不要給它任何特殊的意義。”而在Java中，`\\` 的意思是“我要插入一個正規表示式的反斜線，所以其後的字元具有特殊的意義。”例如，如果你想表示一位數字，那麼正規表示式應該是 `\\d`。如果你想插入一個普通的反斜線，應該這樣寫 `\\\`。不過換行符和定位符號之類的東西只需要使用單眼斜線：`\n\t`。 [^2]

要表示“一個或多個之前的表達式”，應該使用 `+`。所以，如果要表示“可能有一個負號，後面跟著一位或多位數字”，可以這樣：
```java
-?\\d+ 
```
應用正規表示式最簡單的途徑，就是利用 `String` 類內建的功能。例如，你可以檢查一個 `String` 是否匹配如上所述的正規表示式：
```java
// strings/IntegerMatch.java 

public class IntegerMatch {  
    public static void main(String[] args) {     
        System.out.println("-1234".matches("-?\\d+"));    
        System.out.println("5678".matches("-?\\d+"));     
        System.out.println("+911".matches("-?\\d+"));     
        System.out.println("+911".matches("(-|\\+)?\\d+"));   
    }
}
/* Output: 
true 
true 
false 
true 
*/ 
```
前兩個字串都滿足對應的正規表示式，匹配成功。第三個字串以 `+` 開頭，這也是一個合法的符號，但與對應的正規表示式卻不匹配。因此，我們的正規表示式應該描述為：“可能以一個加號或減號開頭”。在正規表示式中，用括號將表達式進行分組，用豎線 ` | ` 表示或操作。也就是：
```java
(-|\\+)? 
```
這個正規表示式表示字串的起始字元可能是一個 `-` 或 `+`，或者二者都沒有（因為後面跟著 `?` 修飾符）。因為字元 `+` 在正規表示式中有特殊的意義，所以必須使用 `\\` 將其轉義，使之成為表達式中的一個普通字元。

`String`類還自帶了一個非常有用的正規表示式工具——`split()` 方法，其功能是“將字串從正規表示式匹配的地方切開。”

```java
// strings/Splitting.java import java.util.*; 

public class Splitting {
    public static String knights =   
      "Then, when you have found the shrubbery, " +
      "you must cut down the mightiest tree in the " +
      "forest...with... a herring!";
    public static void split(String regex) {
        System.out.println(
          Arrays.toString(knights.split(regex)));
        }
    public static void main(String[] args) {
        split(" "); // Doesn't have to contain regex chars
        split("\\W+"); // Non-word characters
        split("n\\W+"); // 'n' followed by non-words
    }
}
/* Output:
[Then,, when, you, have, found, the, shrubbery,, you,
must, cut, down, the, mightiest, tree, in, the,
forest...with..., a, herring!]
[Then, when, you, have, found, the, shrubbery, you,
must, cut, down, the, mightiest, tree, in, the, forest,
with, a, herring]
[The, whe, you have found the shrubbery, you must cut
dow, the mightiest tree i, the forest...with... a
herring!]
*/
```
首先看第一個語句，注意這裡用的是普通的字元作為正規表示式，其中並不包含任何特殊字元。因此第一個 `split()` 只是按空格來劃分字串。

第二個和第三個 `split()` 都用到了 `\\W`，它的意思是一個非單詞字元（如果 W 小寫，`\\w`，則表示一個單詞字元）。透過第二個例子可以看到，它將標點字元刪除了。第三個 `split()` 表示“字母 `n` 後面跟著一個或多個非單詞字元。”可以看到，在原始字串中，與正規表示式匹配的部分，在最終結果中都不存在了。

`String.split()` 還有一個重載的版本，它允許你限制字串分割的次數。

用正規表示式進行取代操作時，你可以只取代第一處匹配，也可以取代所有的匹配：
```java
// strings/Replacing.java 

public class Replacing {
    static String s = Splitting.knights;   
    public static void main(String[] args) {
        System.out.println(
          s.replaceFirst("f\\w+", "located"));
        System.out.println(       
          s.replaceAll("shrubbery|tree|herring","banana"));   
    } 
}
/* Output: 
Then, when you have located the shrubbery, you must cut 
down the mightiest tree in the forest...with... a 
herring! 
Then, when you have found the banana, you must cut down
the mightiest banana in the forest...with... a banana! 
*/
```
第一個表達式要匹配的是，以字母 `f` 開頭，後面跟一個或多個字母（注意這裡的 `w` 是小寫的）。並且只取代掉第一個匹配的部分，所以 “found” 被取代成 “located”。

第二個表達式要匹配的是三個單詞中的任意一個，因為它們以豎線分割表示“或”，並且取代所有匹配的部分。

稍後你會看到，`String` 之外的正規表示式還有更強大的取代工具，例如，可以透過方法呼叫執行取代。而且，如果正規表示式不是只使用一次的話，非 `String` 物件的正規表示式明顯具備更佳的效能。
### 建立正規表示式
我們首先從正規表示式可能存在的構造集中選取一個很有用的子集，以此開始學習正規表示式。正規表示式的完整構造子列表，請參考JDK文件 `java.util.regex` 包中的 `Pattern`類。

| 表達式 | 含義 |
| :---- | :---- |
| `B` | 指定字元`B` |
| `\xhh` | 十六進位制值為`0xhh`的字元 |
| `\uhhhh` | 十六進製表現為`0xhhhh`的Unicode字元 |
| `\t` | 定位符號Tab |
| `\n` | 換行符 |
| `\r` | Enter |
| `\f` | 換頁 |
| `\e` | 轉義（Escape） |

當你學會了使用字元類（character classes）之後，正規表示式的威力才能真正顯現出來。以下是一些建立字元類的典型方式，以及一些預定義的類：

| 表達式 | 含義 |
| :---- | :---- |
| `.` | 任意字元 |
| `[abc]` |包含`a`、`b`或`c`的任何字元（和`a|b|c`作用相同）|
| `[^abc]` | 除`a`、`b`和`c`之外的任何字元（否定） |
| `[a-zA-Z]` | 從`a`到`z`或從`A`到`Z`的任何字元（範圍） |
| `[abc[hij]]` | `a`、`b`、`c`、`h`、`i`、`j`中的任意字元（與`a|b|c|h|i|j`作用相同）（合併） |
| `[a-z&&[hij]]` | 任意`h`、`i`或`j`（交） |
| `\s` | 空白符（空格、tab、換行、換頁、Enter） |
| `\S` | 非空白符（`[^\s]`） |
| `\d` | 數位（`[0-9]`） |
| `\D` | 非數位（`[^0-9]`） |
| `\w` | 詞字元（`[a-zA-Z_0-9]`） |
| `\W` | 非詞字元（`[^\w]`） |

這裡只列出了部分常用的表達式，你應該將JDK文件中 `java.util.regex.Pattern` 那一頁加入瀏覽器書籤中，以便在需要的時候方便查詢。

| 邏輯操作符 | 含義 |
| :----: | :---- |
| `XY` | `Y`跟在`X`後面 |
| `X|Y` | `X`或`Y` |
| `(X)` | 捕獲組（capturing group）。可以在表達式中用`\i`引用第i個捕獲組 |

下面是不同的邊界匹配符：

| 邊界匹配符 | 含義 |
| :----: | :---- |
| `^` | 一行的開始 |
| `$` | 一行的結束 |
| `\b` | 詞的邊界 |
| `\B` | 非詞的邊界 |
| `\G` | 前一個匹配的結束 |

作為示範，下面的每一個正規表示式都能成功匹配字元序列“Rudolph”：
```java
// strings/Rudolph.java 

public class Rudolph {   
    public static void main(String[] args) {     
        for(String pattern : new String[]{       
          "Rudolph",       
          "[rR]udolph",       
          "[rR][aeiou][a-z]ol.*",       
          "R.*" })       
        System.out.println("Rudolph".matches(pattern));   
    } 
} 
/* Output: 
true 
true 
true 
true 
*/
```
我們的目的並不是編寫最難理解的正規表示式，而是儘量編寫能夠完成任務的、最簡單以及最必要的正規表示式。一旦真正開始使用正規表示式了，你就會發現，在編寫新的表達式之前，你通常會參考程式碼中已經用到的正規表示式。
### 量詞
量詞描述了一個模式捕獲輸入文字的方式：

+ **貪婪型**：
量詞總是貪婪的，除非有其他的選項被設定。貪婪表達式會為所有可能的模式發現儘可能多的匹配。導致此問題的一個典型理由就是假定我們的模式僅能匹配第一個可能的字元組，如果它是貪婪的，那麼它就會繼續往下匹配。

+ **勉強型**：
用問號來指定，這個量詞匹配滿足模式所需的最少字元數。因此也被稱作懶惰的、最少匹配的、非貪婪的或不貪婪的。

+ **佔有型**：
目前，這種類型的量詞只有在 Java 語言中才可用（在其他語言中不可用），並且也更進階，因此我們大概不會立刻用到它。當正規表示式被應用於 `String` 時，它會產生相當多的狀態，以便在匹配失敗時可以回溯。而“佔有的”量詞並不儲存這些中間狀態，因此它們可以防止回溯。它們常常用於防止正規表示式失控，因此可以使正規表示式執行起來更高效。

| 貪婪型 | 勉強型 | 佔有型 | 如何匹配 |
| ---- | ---- | ---- | ---- |
| `X?` | `X??` | `X?+` | 一個或零個`X` |
| `X*` | `X*?` | `X*+` | 零個或多個`X` |
| `X+` | `X+?` | `X++` | 一個或多個`X` |
| `X{n}` | `X{n}?` | `X{n}+` | 恰好`n`次`X` |
| `X{n,}` | `X{n,}?` | `X{n,}+` | 至少`n`次`X` |
| `X{n,m}` | `X{n,m}?` | `X{n,m}+` | `X`至少`n`次，但不超過`m`次 |

應該非常清楚地意識到，表達式 `X` 通常必須要用圓括號括起來，以便它能夠按照我們期望的效果去執行。例如：
```java
abc+
```
看起來它似乎應該匹配1個或多個`abc`序列，如果我們把它應用於輸入字串`abcabcabc`，則實際上會獲得3個匹配。然而，這個表達式實際上表示的是：匹配`ab`，後面跟隨1個或多個`c`。要表明匹配1個或多個完整的字串`abc`，我們必須這樣表示：
```java
(abc)+
```
你會發現，在使用正規表示式時很容易混淆，因為它是一種在 Java 之上的新語言。
### `CharSequence`
介面 `CharSequence` 從 `CharBuffer`、`String`、`StringBuffer`、`StringBuilder` 類中抽象出了字元序列的一般化定義：
```java
interface CharSequence {   
    char charAt(int i);   
    int length();
    CharSequence subSequence(int start, int end);
    String toString(); 
}
```
因此，這些類都實現了該介面。多數正規表示式操作都接受 `CharSequence` 類型參數。
### `Pattern` 和 `Matcher`
通常，比起功能有限的 `String` 類，我們更願意構造功能強大的正規表示式物件。只需匯入 `java.util.regex`包，然後用 `static Pattern.compile()` 方法來編譯你的正規表示式即可。它會根據你的 `String` 類型的正規表示式生成一個 `Pattern` 物件。接下來，把你想要檢索的字串傳入 `Pattern` 物件的 `matcher()` 方法。`matcher()` 方法會生成一個 `Matcher` 物件，它有很多功能可用（可以參考 `java.util.regext.Matcher` 的 JDK 文件）。例如，它的 `replaceAll()` 方法能將所有匹配的部分都取代成你傳入的參數。

作為第一個範例，下面的類可以用來測試正規表示式，看看它們能否匹配一個輸入字串。第一個控制台參數是將要用來搜尋匹配的輸入字串，後面的一個或多個參數都是正規表示式，它們將被用來在輸入的第一個字串中尋找匹配。在Unix/Linux上，命令列中的正規表示式必須用引號括起來。這個程式在測試正規表示式時很有用，特別是當你想驗證它們是否具備你所期待的匹配功能的時候。[^3]
```java
// strings/TestRegularExpression.java 
// Simple regular expression demonstration 
// {java TestRegularExpression 
// abcabcabcdefabc "abc+" "(abc)+" } 
import java.util.regex.*; 

public class TestRegularExpression {
    public static void main(String[] args) {     
        if(args.length < 2) {     
            System.out.println(       
              "Usage:\njava TestRegularExpression " +         
              "characterSequence regularExpression+");      
            System.exit(0);    
        }
        System.out.println("Input: \"" + args[0] + "\"");     
        for(String arg : args) {       
            System.out.println(         
              "Regular expression: \"" + arg + "\"");       
            Pattern p = Pattern.compile(arg);       
            Matcher m = p.matcher(args[0]);       
            while(m.find()) {         
                System.out.println(           
                  "Match \"" + m.group() + "\" at positions " +           
                m.start() + "-" + (m.end() - 1));       
            }     
        }  
    }
}
/* Output: 
Input: "abcabcabcdefabc" 
Regular expression: "abcabcabcdefabc" 
Match "abcabcabcdefabc" at positions 0-14 
Regular expression: "abc+" 
Match "abc" at positions 0-2 
Match "abc" at positions 3-5 
Match "abc" at positions 6-8 
Match "abc" at positions 12-14 
Regular expression: "(abc)+"
Match "abcabcabc" at positions 0-8 
Match "abc" at positions 12-14 
*/
```
還可以在控制台參數中加入`“(abc){2,}”`，看看執行結果。

`Pattern` 物件表示編譯後的正規表示式。從這個例子可以看到，我們使用已編譯的 `Pattern` 物件上的 `matcher()` 方法，加上一個輸入字串，從而共同構造了一個 `Matcher` 物件。同時，`Pattern` 類還提供了一個`static`方法：

```java
static boolean matches(String regex, CharSequence input)
```
該方法用以檢查 `regex` 是否匹配整個 `CharSequence` 類型的 `input` 參數。編譯後的 `Pattern` 物件還提供了 `split()` 方法，它從匹配了 `regex` 的地方分割輸入字串，返回分割後的子字串 `String` 陣列。

透過呼叫 `Pattern.matcher()` 方法，並傳入一個字串參數，我們得到了一個 `Matcher` 物件。使用 `Matcher` 上的方法，我們將能夠判斷各種不同類型的匹配是否成功：
```java
boolean matches() 
boolean lookingAt() 
boolean find() 
boolean find(int start)
```
其中的 `matches()` 方法用來判斷整個輸入字串是否匹配正規表示式模式，而 `lookingAt()` 則用來判斷該字串（不必是整個字串）的起始部分是否能夠匹配模式。
### `find()`
`Matcher.find()` 方法可用來在 `CharSequence` 中尋找多個匹配。例如：

```java
// strings/Finding.java 
import java.util.regex.*; 

public class Finding {   
    public static void main(String[] args) {     
        Matcher m = Pattern.compile("\\w+")       
          .matcher(         
            "Evening is full of the linnet's wings");     
        while(m.find())       
            System.out.print(m.group() + " ");   
        System.out.println();     
        int i = 0;     
        while(m.find(i)) {       
            System.out.print(m.group() + " ");       
            i++;     
        }   
    }
}
/* Output: 
Evening is full of the linnet s wings
Evening vening ening ning ing ng g is is s full full 
ull ll l of of f the the he e linnet linnet innet nnet 
net et t s s wings wings ings ngs gs s 
*/
```
模式 `\\w+` 將字串劃分為詞。`find()` 方法像疊代器那樣向前遍歷輸入字串。而第二個重載的 `find()` 接收一個整型參數，該整數表示字串中字元的位置，並以其作為搜尋的起點。從結果可以看出，後一個版本的 `find()` 方法能夠根據其參數的值，不斷重新設定搜尋的起始位置。
### 組（Groups）
組是用括號劃分的正規表示式，可以根據組的編號來引用某個組。組號為 0 表示整個表達式，組號 1 表示被第一對括號括起來的組，以此類推。因此，下面這個表達式，
```java
A(B(C))D
```
中有三個組：組 0 是 `ABCD`，組 1 是 `BC`，組 2 是 `C`。

`Matcher` 物件提供了一系列方法，用以獲取與組相關的訊息：

+ `public int groupCount()` 返回該匹配器的模式中的分組數目，組 0 不包括在內。
+ `public String group()` 返回前一次匹配操作（例如 `find()`）的第 0 組（整個匹配）。
+ `public String group(int i)` 返回前一次匹配操作期間指定的組號，如果匹配成功，但是指定的組沒有匹配輸入字串的任何部分，則將返回 `null`。
+ `public int start(int group)` 返回在前一次匹配操作中尋找到的組的起始索引。
+ `public int end(int group)` 返回在前一次匹配操作中尋找到的組的最後一個字元索引加一的值。

下面是正規表示式組的例子：
```java
// strings/Groups.java
import java.util.regex.*; 

public class Groups {   
    public static final String POEM =     
      "Twas brillig, and the slithy toves\n" +     
      "Did gyre and gimble in the wabe.\n" +     
      "All mimsy were the borogoves,\n" +     
      "And the mome raths outgrabe.\n\n" +     
      "Beware the Jabberwock, my son,\n" +     
      "The jaws that bite, the claws that catch.\n" +     
      "Beware the Jubjub bird, and shun\n" +     
      "The frumious Bandersnatch.";   
    public static void main(String[] args) {     
        Matcher m = Pattern.compile(
          "(?m)(\\S+)\\s+((\\S+)\\s+(\\S+))$")       
          .matcher(POEM);     
        while(m.find()) {       
            for(int j = 0; j <= m.groupCount(); j++)         
                System.out.print("[" + m.group(j) + "]");       
            System.out.println();     
        }   
    } 
}
/* Output: 
[the slithy toves][the][slithy toves][slithy][toves] 
[in the wabe.][in][the wabe.][the][wabe.] 
[were the borogoves,][were][the 
borogoves,][the][borogoves,] 
[mome raths outgrabe.][mome][raths 
outgrabe.][raths][outgrabe.] 
[Jabberwock, my son,][Jabberwock,][my son,][my][son,] 
[claws that catch.][claws][that catch.][that][catch.] 
[bird, and shun][bird,][and shun][and][shun] 
[The frumious Bandersnatch.][The][frumious 
Bandersnatch.][frumious][Bandersnatch.] 
*/
```
這首詩來自於 Lewis Carroll 所寫的 *Through the Looking Glass* 中的 “Jabberwocky”。可以看到這個正規表示式模式有許多圓括號分組，由任意數目的非空白符（`\\S+`）及隨後的任意數目的空白符（`\\s+`）所組成。目的是捕獲每行的最後3個詞，每行最後以 `\$` 結束。不過，在正常情況下是將 `\$` 與整個輸入序列的末端相匹配。所以我們一定要顯式地告知正規表示式注意輸入序列中的換行符。這可以由序列開頭的模式標記 `(?m)` 來完成（模式標記馬上就會介紹）。
### `start()` 和 `end()`
在匹配操作成功之後，`start()` 返回先前匹配的起始位置的索引，而 `end()` 返回所匹配的最後字元的索引加一的值。匹配操作失敗之後（或先於一個正在進行的匹配操作去嘗試）呼叫 `start()` 或 `end()` 將會產生 `IllegalStateException`。下面的範例還同時展示了 `matches()` 和 `lookingAt()` 的用法 [^4]：
```java
// strings/StartEnd.java 
import java.util.regex.*; 

public class StartEnd {
    public static String input =
      "As long as there is injustice, whenever a\n" +     
      "Targathian baby cries out, wherever a distress\n" +     
      "signal sounds among the stars " +     
      "... We'll be there.\n"+     
      "This fine ship, and this fine crew ...\n" +     
      "Never give up! Never surrender!";   
    private static class Display {
        private boolean regexPrinted = false;     
        private String regex;
        Display(String regex) { this.regex = regex; }     
    
        void display(String message) {       
            if(!regexPrinted) {         
                System.out.println(regex);         
                regexPrinted = true;       
            }       
            System.out.println(message);     
        }   
    }   
  
    static void examine(String s, String regex) {     
        Display d = new Display(regex);     
        Pattern p = Pattern.compile(regex);     
        Matcher m = p.matcher(s);     
        while(m.find())       
            d.display("find() '" + m.group() +         
              "' start = "+ m.start() + " end = " + m.end());     
        if(m.lookingAt()) // No reset() necessary       
            d.display("lookingAt() start = "         
              + m.start() + " end = " + m.end());     
        if(m.matches()) // No reset() necessary       
            d.display("matches() start = "         
              + m.start() + " end = " + m.end());   
    }
    
    public static void main(String[] args) {     
        for(String in : input.split("\n")) {       
            System.out.println("input : " + in);       
            for(String regex : new String[]{"\\w*ere\\w*",         
              "\\w*ever", "T\\w+", "Never.*?!"})         
                examine(in, regex);     
        }   
    } 
} 
/* Output: 
input : As long as there is injustice, whenever a 
\w*ere\w* 
find() 'there' start = 11 end = 16 
\w*ever 
find() 'whenever' start = 31 end = 39 
input : Targathian baby cries out, wherever a distress 
\w*ere\w* 
find() 'wherever' start = 27 end = 35 
\w*ever 
find() 'wherever' start = 27 end = 35 
T\w+ find() 'Targathian' start = 0 end = 10 
lookingAt() start = 0 end = 10 
input : signal sounds among the stars ... We'll be 
there. 
\w*ere\w* 
find() 'there' start = 43 end = 48 
input : This fine ship, and this fine crew ... 
T\w+ find() 'This' start = 0 end = 4
lookingAt() start = 0 end = 4 
input : Never give up! Never surrender! 
\w*ever 
find() 'Never' start = 0 end = 5 
find() 'Never' start = 15 end = 20 
lookingAt() start = 0 end = 5 
Never.*?! 
find() 'Never give up!' start = 0 end = 14 
find() 'Never surrender!' start = 15 end = 31 
lookingAt() start = 0 end = 14 
matches() start = 0 end = 31 
*/ 
```
注意，`find()` 可以在輸入的任意位置定位正規表示式，而 `lookingAt()` 和 `matches()` 只有在正規表示式與輸入的最開始處就開始匹配時才會成功。`matches()` 只有在整個輸入都匹配正規表示式時才會成功，而 `lookingAt()` [^5] 只要輸入的第一部分匹配就會成功。
### `Pattern` 標記
`Pattern` 類的 `compile()` 方法還有另一個版本，它接受一個標記參數，以調整匹配行為：

```java
Pattern Pattern.compile(String regex, int flag)
```
其中的 `flag` 來自以下 `Pattern` 類中的常量

| 編譯標記 | 效果 |
| ---- |---- |
| `Pattern.CANON_EQ` | 當且僅當兩個字元的完全規範分解相匹配時，才認為它們是匹配的。例如，如果我們指定這個標記，表達式`\u003F`就會匹配字串`?`。預設情況下，匹配不考慮規範的等價性 |
| `Pattern.CASE_INSENSITIVE(?i)` | 預設情況下，大小寫不敏感的匹配假定只有US-ASCII字元集中的字元才能進行。這個標記允許模式匹配不考慮大小寫（大寫或小寫）。透過指定`UNICODE_CASE`標記及結合此標記。基於Unicode的大小寫不敏感的匹配就可以開啟了 |
| `Pattern.COMMENTS(?x)` | 在這種模式下，空格符將被忽略掉，並且以`#`開始直到行末的注釋也會被忽略掉。透過嵌入的標記表達式也可以開啟Unix的行模式 |
| `Pattern.DOTALL(?s)` | 在dotall模式下，表達式`.`匹配所有字元，包括行終止符。預設情況下，`.`不會匹配行終止符 |
| `Pattern.MULTILINE(?m)` | 在多行模式下，表達式`^`和`$`分別匹配一行的開始和結束。`^`還匹配輸入字串的開始，而`$`還匹配輸入字串的結尾。預設情況下，這些表達式僅匹配輸入的完整字串的開始和結束 |
| `Pattern.UNICODE_CASE(?u)` | 當指定這個標記，並且開啟`CASE_INSENSITIVE`時，大小寫不敏感的匹配將按照與Unicode標準相一致的方式進行。預設情況下，大小寫不敏感的匹配假定只能在US-ASCII字元集中的字元才能進行 |
| `Pattern.UNIX_LINES(?d)` | 在這種模式下，在`.`、`^`和`$`的行為中，只識別行終止符`\n` |

在這些標記中，`Pattern.CASE_INSENSITIVE`、`Pattern.MULTILINE` 以及 `Pattern.COMMENTS`（對聲明或文件有用）特別有用。請注意，你可以直接在正規表示式中使用其中的大多數標記，只需要將上表中括號括起來的字元插入到正規表示式中，你希望它起作用的位置即可。

你還可以透過“或”(`|`)操作符組合多個標記的功能：
```java
// strings/ReFlags.java 
import java.util.regex.*; 

public class ReFlags {   
    public static void main(String[] args) {     
        Pattern p =  Pattern.compile("^java",       
          Pattern.CASE_INSENSITIVE | Pattern.MULTILINE);     
        Matcher m = p.matcher(       
          "java has regex\nJava has regex\n" +       
          "JAVA has pretty good regular expressions\n" +       
          "Regular expressions are in Java");     
        while(m.find())       
            System.out.println(m.group());   
    } 
}
/* Output: 
java 
Java 
JAVA 
*/
```
在這個例子中，我們建立了一個模式，它將匹配所有以“java”、“Java”和“JAVA”等開頭的行，並且是在設定了多行標記的狀態下，對每一行（從字元序列的第一個字元開始，至每一個行終止符）都進行匹配。注意，`group()` 方法只返回已匹配的部分。
### `split()`
`split()`方法將輸入 `String` 斷開成 `String` 物件陣列，斷開邊界由正規表示式確定：

```java
String[] split(CharSequence input) 
String[] split(CharSequence input, int limit)
```
這是一個快速而方便的方法，可以按照通用邊界斷開輸入文字：
```java
// strings/SplitDemo.java 
import java.util.regex.*; 
import java.util.*; 

public class SplitDemo {  
    public static void main(String[] args) {     
        String input =       
          "This!!unusual use!!of exclamation!!points";     
        System.out.println(Arrays.toString(       
        Pattern.compile("!!").split(input)));     
        // Only do the first three:     
        System.out.println(Arrays.toString(       
        Pattern.compile("!!").split(input, 3)));   
    }
}
/* Output: 
[This, unusual use, of exclamation, points] 
[This, unusual use, of exclamation!!points]
*/
```
第二種形式的 `split()` 方法可以限制將輸入分割成字串的數量。
### 取代操作
正規表示式在進行文字取代時特別方便，它提供了許多方法：
+ `replaceFirst(String replacement)` 以參數字串 `replacement` 取代掉第一個匹配成功的部分。
+ `replaceAll(String replacement)` 以參數字串 `replacement` 取代所有匹配成功的部分。
+ `appendReplacement(StringBuffer sbuf, String replacement)` 執行漸進式的取代，而不是像 `replaceFirst()` 和 `replaceAll()` 那樣只取代第一個匹配或全部匹配。這是一個非常重要的方法。它允許你呼叫其他方法來生成或處理 `replacement`（`replaceFirst()` 和 `replaceAll()` 則只能使用一個固定的字串），使你能夠以編程的方式將目標分割成組，從而具備更強大的取代功能。
+ `appendTail(StringBuffer sbuf)` 在執行了一次或多次 `appendReplacement()` 之後，呼叫此方法可以將輸入字串餘下的部分複製到 `sbuf` 中。

下面的程式示範了如何使用這些取代方法。開頭部分注釋掉的文字，就是正規表示式要處理的輸入字串：
```java
// strings/TheReplacements.java 
import java.util.regex.*; 
import java.nio.file.*; 
import java.util.stream.*;

/*! Here's a block of text to use as input to 
    the regular expression matcher. Note that we 
    first extract the block of text by looking for 
    the special delimiters, then process the     
    extracted block. !*/

public class TheReplacements {   
    public static void main(String[] args) throws Exception {     
        String s = Files.lines(       
          Paths.get("TheReplacements.java"))       
          .collect(Collectors.joining("\n"));     
        // Match specially commented block of text above:     
        Matcher mInput = Pattern.compile(       
          "/\\*!(.*)!\\*/", Pattern.DOTALL).matcher(s);     
        if(mInput.find())       
            s = mInput.group(1); // Captured by parentheses     
        // Replace two or more spaces with a single space:     
        s = s.replaceAll(" {2,}", " ");     
        // Replace 1+ spaces at the beginning of each     
        // line with no spaces. Must enable MULTILINE mode:     
        s = s.replaceAll("(?m)^ +", "");     
        System.out.println(s);     
        s = s.replaceFirst("[aeiou]", "(VOWEL1)");     
        StringBuffer sbuf = new StringBuffer();     
        Pattern p = Pattern.compile("[aeiou]");     
        Matcher m = p.matcher(s);     
        // Process the find information as you     
        // perform the replacements:     
        while(m.find())      
            m.appendReplacement(sbuf, m.group().toUpperCase());     
        // Put in the remainder of the text:     
        m.appendTail(sbuf);     
        System.out.println(sbuf);
    } 
}
/* Output: 
Here's a block of text to use as input to 
the regular expression matcher. Note that we 
first extract the block of text by looking for 
the special delimiters, then process the 
extracted block. 
H(VOWEL1)rE's A blOck Of tExt tO UsE As InpUt tO 
thE rEgUlAr ExprEssIOn mAtchEr. NOtE thAt wE 
fIrst ExtrAct thE blOck Of tExt by lOOkIng fOr 
thE spEcIAl dElImItErs, thEn prOcEss thE 
ExtrActEd blOck. 
*/
```
此處使用上一章介紹過的 [`Files`](./17-Files.md) 類打開並讀入檔案。`Files.lines()` 返回一個 `Stream` 物件，包含讀入的所有行，`Collectors.joining()` 在每一行的結尾追加參數字元序列，最終拼接成一個 `String` 物件。

`mInput` 匹配 `/*!` 和 `！*/` 之間的所有文字（注意分組的括號）。接下來，將存在兩個或兩個以上空格的地方，縮減為一個空格，並且刪除每行開頭部分的所有空格（為了使每一行都達到這個效果，而不僅僅是刪除文字開頭部分的空格，這裡特地開啟了多行模式）。這兩個取代操作所使用的的 `replaceAll()` 是 `String` 物件自帶的方法，在這裡，使用此方法更方便。注意，因為這兩個取代操作都只使用了一次 `replaceAll()`，所以，與其編譯為 `Pattern`，不如直接使用 `String` 的 `replaceAll()` 方法，而且開銷也更小些。

`replaceFirst()` 只對找到的第一個匹配進行取代。此外，`replaceFirst()` 和 `replaceAll()` 方法用來取代的只是普通字串，所以，如果想對這些取代字串進行某些特殊處理，這兩個方法時無法勝任的。如果你想要那麼做，就應該使用 `appendReplacement()` 方法。該方法允許你在執行取代的過程中，操作用來取代的字串。在這個例子中，先構造了 `sbuf` 用來儲存最終結果，然後用 `group()` 選擇一個組，並對其進行處理，將正規表示式找到的母音字母取代成大些字母。一般情況下，你應該遍歷執行所有的取代操作，然後再呼叫 `appendTail()` 方法，但是，如果你想模擬 `replaceFirst()`（或取代n次）的行為，那就只需要執行一次取代，然後呼叫 `appendTail()` 方法，將剩餘未處理的部分存入 `sbuf` 即可。

同時，`appendReplacement()` 方法還允許你透過 `\$g` 直接找到匹配的某個組，這裡的 `g` 就是組號。然而，它只能應付一些簡單的處理，無法實現類似前面這個例子中的功能。
### `reset()`
通過 `reset()` 方法，可以將現有的 `Matcher` 物件應用於一個新的字元序列：
```java
// strings/Resetting.java 
import java.util.regex.*; 

public class Resetting {   
    public static void main(String[] args) throws Exception {     
        Matcher m = Pattern.compile("[frb][aiu][gx]")       
          .matcher("fix the rug with bags");     
        while(m.find())       
            System.out.print(m.group() + " ");     
        System.out.println();     
        m.reset("fix the rig with rags");     
        while(m.find())       
            System.out.print(m.group() + " ");   
    } 
} 
/* Output: 
fix rug bag 
fix rig rag 
*/
```
使用不帶參數的 `reset()` 方法，可以將 `Matcher` 物件重新設定到目前字元序列的起始位置。
### 正規表示式與 Java I/O
到目前為止，我們看到的例子都是將正規表示式用於靜態的字串。下面的例子將向你示範，如何應用正規表示式在一個文件中進行搜尋匹配操作。`JGrep.java` 的靈感源自於 Unix 上的 *grep*。它有兩個參數：檔案名以及要匹配的正規表示式。輸出的是每行有匹配的部分以及匹配部分在行中的位置。
```java
// strings/JGrep.java 
// A very simple version of the "grep" program 
// {java JGrep 
// WhitherStringBuilder.java 'return|for|String'} 
import java.util.regex.*; 
import java.nio.file.*; 
import java.util.stream.*;

public class JGrep {  
    public static void main(String[] args) throws Exception {    
        if(args.length < 2) {      
            System.out.println(        
              "Usage: java JGrep file regex");      
            System.exit(0);   
        }    
        Pattern p = Pattern.compile(args[1]);    
        // Iterate through the lines of the input file:    
        int index = 0;    
        Matcher m = p.matcher("");    
        for(String line: Files.readAllLines(Paths.get(args[0]))) {      
            m.reset(line);      
            while(m.find())        
                System.out.println(index++ + ": " +          
                  m.group() + ": " + m.start());    
        }  
    } 
} 
/* Output: 
0: for: 4 
1: for: 4 
*/
```
`Files.readAllLines()` 返回一個 `List<String>` 物件，這意味著可以用 *for-in* 進行遍歷。雖然可以在 `for` 循環內部建立一個新的 `Matcher` 物件，但是，在循環體外建立一個空的 `Matcher` 物件，然後用 `reset()` 方法每次為 `Matcher` 載入一行輸入，這種處理會有一定的效能最佳化。最後用 `find()` 搜尋結果。

這裡讀入的測試參數是 `JGrep.java` 文件，然後搜尋以 `[Ssct]` 開頭的單詞。

如果想要更深入地學習正規表示式，你可以閱讀 Jeffrey E. F. Friedl 的《精通正規表示式（第2版）》。網路上也有很多正規表示式的介紹，你還可以從 Perl 和 Python 等其他語言的文件中找到有用的訊息。


<!-- Scanning Input -->
## 掃描輸入
到目前為止，從文件或標準輸入讀取資料還是一件相當痛苦的事情。一般的解決辦法就是讀入一行文字，對其進行分詞，然後使用 `Integer`、`Double` 等類的各種解析方法來解析資料：
```java
// strings/SimpleRead.java 
import java.io.*;

public class SimpleRead {  
    public static BufferedReader input =    
      new BufferedReader(new StringReader(    
        "Sir Robin of Camelot\n22 1.61803"));  
    public static void main(String[] args) {    
        try {      
            System.out.println("What is your name?");      
            String name = input.readLine();      
            System.out.println(name);      
            System.out.println("How old are you? " +        
              "What is your favorite double?");      
            System.out.println("(input: <age> <double>)");      
            String numbers = input.readLine();      
            System.out.println(numbers);      
            String[] numArray = numbers.split(" ");      
            int age = Integer.parseInt(numArray[0]);      
            double favorite = Double.parseDouble(numArray[1]);      
            System.out.format("Hi %s.%n", name);      
            System.out.format("In 5 years you will be %d.%n", age + 5);      
            System.out.format("My favorite double is %f.", favorite / 2);    
        } catch(IOException e) {      
            System.err.println("I/O exception");    
        }  
    } 
}
/* Output: 
What is your name? 
Sir Robin of Camelot 
How old are you? What is your favorite double? 
(input: <age> <double>) 
22 1.61803
Hi Sir Robin of Camelot. 
In 5 years you will be 27. 
My favorite double is 0.809015. 
*/
```
`input` 欄位使用的類來自 `java.io`，[附錄:流式 I/O](./Appendix-IO-Streams.md) 詳細介紹了相關內容。`StringReader` 將 `String` 轉化為可讀的流物件，然後用這個物件來構造 `BufferedReader` 物件，因為我們要使用 `BufferedReader` 的 `readLine()` 方法。最終，我們可以使用 `input` 物件一次讀取一行文字，就像從控制台讀入標準輸入一樣。

`readLine()` 方法將一行輸入轉為 `String` 物件。如果每一行資料正好對應一個輸入值，那這個方法也還可行。但是，如果兩個輸入值在同一行中，事情就不好辦了，我們必須分解這個行，才能分別解析所需的輸入值。在這個例子中，分解的操作發生在建立 `numArray`時。

終於，Java SE5 新增了 `Scanner` 類，它可以大大減輕掃描輸入的工作負擔：
```java
// strings/BetterRead.java 
import java.util.*; 

public class BetterRead {
  public static void main(String[] args) {
    Scanner stdin = new Scanner(SimpleRead.input);
    System.out.println("What is your name?");
    String name = stdin.nextLine();
    System.out.println(name);
    System.out.println(
      "How old are you? What is your favorite double?");
    System.out.println("(input: <age> <double>)");
    int age = stdin.nextInt();
    double favorite = stdin.nextDouble();
    System.out.println(age);
    System.out.println(favorite);
    System.out.format("Hi %s.%n", name);
    System.out.format("In 5 years you will be %d.%n",
      age + 5);
    System.out.format("My favorite double is %f.",
      favorite / 2);
  }
}
/* Output: 
What is your name? 
Sir Robin of Camelot 
How old are you? What is your favorite double? 
(input: <age> <double>) 
22 
1.61803 
Hi Sir Robin of Camelot. 
In 5 years you will be 27. 
My favorite double is 0.809015. 
*/
```
`Scanner` 的構造器可以接收任意類型的輸入物件，包括 `File`、`InputStream`、`String` 或者像此例中的`Readable` 實現類。`Readable` 是 Java SE5 中新加入的一個介面，表示“具有 `read()` 方法的某種東西”。上一個例子中的 `BufferedReader` 也歸於這一類。

有了 `Scanner`，所有的輸入、分詞、以及解析的操作都隱藏在不同類型的 `next` 方法中。普通的 `next()` 方法返回下一個 `String`。所有的基本類型（除 `char` 之外）都有對應的 `next` 方法，包括 `BigDecimal` 和 `BigInteger`。所有的 next 方法，只有在找到一個完整的分詞之後才會返回。`Scanner` 還有相應的 `hasNext` 方法，用以判斷下一個輸入分詞是否是所需的類型，如果是則返回 `true`。

在 `BetterRead.java` 中沒有用 `try` 區塊捕獲`IOException`。因為，`Scanner` 有一個假設，在輸入結束時會拋出 `IOException`，所以 `Scanner` 會把 `IOException` 吞掉。不過，通過 `ioException()` 方法，你可以找到最近發生的異常，因此，你可以在必要時檢查它。
### `Scanner` 分隔符
預設情況下，`Scanner` 根據空白字元對輸入進行分詞，但是你可以用正規表示式指定自己所需的分隔符：
```java
// strings/ScannerDelimiter.java 
import java.util.*;
public class ScannerDelimiter {  
    public static void main(String[] args) {    
        Scanner scanner = new Scanner("12, 42, 78, 99, 42");    
        scanner.useDelimiter("\\s*,\\s*");    
        while(scanner.hasNextInt())    
            System.out.println(scanner.nextInt());  
    } 
}
/* Output: 
12 
42 
78 
99 
42 
*/
```
這個例子使用逗號（包括逗號前後任意的空白字元）作為分隔符，同樣的技術也可以用來讀取逗號分隔的文件。我們可以用 `useDelimiter()` 來設定分隔符，同時，還有一個 `delimiter()` 方法，用來返回目前正在作為分隔符使用的 `Pattern` 物件。
### 用正規表示式掃描
除了能夠掃描基本類型之外，你還可以使用自訂的正規表示式進行掃描，這在掃描複雜資料時非常有用。下面的例子將掃描一個防火牆日誌檔案中的威脅資料：
```java
// strings/ThreatAnalyzer.java 
import java.util.regex.*; 
import java.util.*;
public class ThreatAnalyzer { 
    static String threatData =    
      "58.27.82.161@08/10/2015\n" +   
      "204.45.234.40@08/11/2015\n" +    
      "58.27.82.161@08/11/2015\n" +    
      "58.27.82.161@08/12/2015\n" +    
      "58.27.82.161@08/12/2015\n" +
      "[Next log section with different data format]";  
    public static void main(String[] args) { 
        Scanner scanner = new Scanner(threatData);    
        String pattern = "(\\d+[.]\\d+[.]\\d+[.]\\d+)@" +      
            "(\\d{2}/\\d{2}/\\d{4})";    
        while(scanner.hasNext(pattern)) {      
            scanner.next(pattern);      
            MatchResult match = scanner.match();      
            String ip = match.group(1);      
            String date = match.group(2);      
            System.out.format(        
                "Threat on %s from %s%n", date,ip);    
        }  
    } 
} 
/* Output: 
Threat on 08/10/2015 from 58.27.82.161 
Threat on 08/11/2015 from 204.45.234.40 
Threat on 08/11/2015 from 58.27.82.161 
Threat on 08/12/2015 from 58.27.82.161 
Threat on 08/12/2015 from 58.27.82.161 
*/  
```
當 `next()` 方法配合指定的正規表示式使用時，將找到下一個匹配該模式的輸入部分，呼叫 `match()` 方法就可以獲得匹配的結果。如上所示，它的工作方式與之前看到的正規表示式匹配相似。

在配合正規表示式使用掃描時，有一點需要注意：它僅僅針對下一個輸入分詞進行匹配，如果你的正規表示式中含有分隔符，那永遠不可能匹配成功。

<!-- StringTokenizer -->

## StringTokenizer類
在 Java 引入正規表示式（J2SE1.4）和 `Scanner` 類（Java SE5）之前，分割字串的唯一方法是使用 `StringTokenizer` 來分詞。不過，現在有了正規表示式和 `Scanner`，我們可以使用更加簡單、更加簡潔的方式來完成同樣的工作了。下面的例子中，我們將 `StringTokenizer` 與另外兩種技術簡單地做了一個比較：
```java
// strings/ReplacingStringTokenizer.java 
import java.util.*; 

public class ReplacingStringTokenizer {   
    public static void main(String[] args) {     
        String input =       
          "But I'm not dead yet! I feel happy!";     
        StringTokenizer stoke = new StringTokenizer(input);     
        while(stoke.hasMoreElements())       
            System.out.print(stoke.nextToken() + " ");     
        System.out.println(); 
        System.out.println(Arrays.toString(input.split(" ")));     
        Scanner scanner = new Scanner(input);     
        while(scanner.hasNext())       
            System.out.print(scanner.next() + " ");   
    }
} 
/* Output: 
But I'm not dead yet! I feel happy! 
[But, I'm, not, dead, yet!, I, feel, happy!] 
But I'm not dead yet! I feel happy! 
*/
```
使用正規表示式或 `Scanner` 物件，我們能夠以更加複雜的模式來分割一個字串，而這對於 `StringTokenizer` 來說就很困難了。基本上，我們可以放心地說，`StringTokenizer` 已經可以廢棄不用了。


<!-- Summary -->
## 本章小結
過去，Java 對於字串操作的技術相當不完善。不過隨著近幾個版本的升級，我們可以看到，Java 已經從其他語言中吸取了許多成熟的經驗。到目前為止，它對字串操作的支援已經很完善了。不過，有時你還要在細節上注意效率問題，例如恰當地使用 `StringBuilder` 等。


[^1]: C++允許編程人員任意重載操作符。這通常是很複雜的過程（參見Prentice Hall於2000年編寫的《Thinking in C++（第2版）》第10章），因此Java設計者認為這是很糟糕的功能，不應該納入到Java中。起始重載操作符並沒有糟糕到只能自己去重載的地步，但具有諷刺意味的是，與C++相比，在Java中使用操作符重載要容易得多。這一點可以在Python(參見[www.Python.org](http://www.python.org))和C#中看到，它們都有垃圾回收機制，操作符重載也簡單易懂。


[^2]: Java並非在一開始就支援正規表示式，因此這個令人費解的語法是硬塞進來的。


[^3]: 網路上還有很多實用並且成熟的正規表示式工具。


[^4]: input來自於[Galaxy Quest](https://en.wikipedia.org/wiki/Galaxy_Quest)中Taggart司令的一篇演講。


[^5]: 我不知道他們是如何想出這個方法名的，或者它到底指的什麼。這只是程式碼審查很重要的原因之一。

<!-- 分頁 -->

<div style="page-break-after: always;"></div>
