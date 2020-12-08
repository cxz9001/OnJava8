[TOC]

# 第五章 控制流

> 程式必須在執行過程中控制它的世界並做出選擇。 在 Java 中，你需要執行控制語句來做出選擇。

Java 使用了 C 的所有執行控制語句，因此對於熟悉 C/C++ 編程的人來說，這部分內容駕輕就熟。大多數程序導向程式語言都有共通的某種控制語句。在 Java 中，涉及的關鍵字包括 **if-else，while，do-while，for，return，break** 和選擇語句 **switch**。 Java 並不支援備受詬病的 **goto**（儘管它在某些特殊場景中依然是最行之有效的方法）。 儘管如此，在 Java 中我們仍舊可以進行類似的邏輯跳轉，但較之典型的 **goto** 用法限制更多。


## true和false

所有的條件語句都利用條件表達式的“真”或“假”來決定執行路徑。舉例：
`a == b`。它利用了條件表達式 `==` 來比較 `a` 與 `b` 的值是否相等。 該表達式返回 `true` 或 `false`。程式碼範例：

```java
// control/TrueFalse.java
public class TrueFalse {
	public static void main(String[] args) {
		System.out.println(1 == 1);
		System.out.println(1 == 2);
	}
}
```

輸出結果：

```
true false 
```

透過上一章的學習，我們知道任何關係運算符都可以產生條件語句。 **注意**：在 Java 中使用數值作為布林值是非法的，即便這種操作在 C/C++ 中是被允許的（在這些語言中，“真”為非零，而“假”是零）。如果想在布爾測試中使用一個非布林值，那麼首先需要使用條件表達式來產生 **boolean** 類型的結果，例如 `if(a != 0)`。

## if-else

**if-else** 語句是控制程式執行流程最基本的形式。 其中 `else` 是可選的，因此可以有兩種形式的 `if`。程式碼範例：

```java
if(Boolean-expression) 
	“statement” 
```

或

```java
if(Boolean-expression) 
	“statement”
else
  “statement”
```

布爾表達式（Boolean-expression）必須生成 **boolean** 類型的結果，執行語句 `statement` 既可以是以分號 `;` 結尾的一條簡單語句，也可以是包含在大括號 `{}` 內的的複合語句 —— 封閉在大括號內的一組簡單語句。 凡本書中提及“statement”一詞，皆表示類似的執行語句。

下面是一個有關 **if-else** 語句的例子。`test()` 方法可以告知你兩個數值之間的大小關係。程式碼範例：

```java
// control/IfElse.java
public class IfElse {
  static int result = 0;
  static void test(int testval, int target) {
    if(testval > target)
      result = +1;
    else if(testval < target) // [1]
      result = -1;
    else
      result = 0; // Match
  }

  public static void main(String[] args) {
    test(10, 5);
    System.out.println(result);
    test(5, 10);
    System.out.println(result);
    test(5, 5);
    System.out.println(result);
  }
}
```

輸出結果：

```
1
-1
0
```

<sub>**註解**：`else if` 並非新關鍵字，它僅是 `else` 後緊跟的一條新 `if` 語句。</sub>

Java 和 C/C++ 同屬“自由格式”的程式語言，但通常我們會在 Java 控制流程語句中採用首部縮排的規範，以便程式碼更具可讀性。

<!--Iteration Statements-->
## 疊代語句

**while**，**do-while** 和 **for** 用來控制循環語句（有時也稱疊代語句）。只有控制循環的布爾表達式計算結果為 `false`，循環語句才會停止。 


### while

**while** 循環的形式是：

```java
while(Boolean-expression) 
  statement
```

執行語句會在每一次循環前，判斷布爾表達式返回值是否為 `true`。下例可產生隨機數，直到滿足特定條件。程式碼範例：

```java
// control/WhileTest.java
// 示範 while 循環
public class WhileTest {
  static boolean condition() {
    boolean result = Math.random() < 0.99;
    System.out.print(result + ", ");
    return result;
  }
  public static void main(String[] args) {
    while(condition())
      System.out.println("Inside 'while'");
    System.out.println("Exited 'while'");
  }
}
```

輸出結果：

```
true, Inside 'while'
true, Inside 'while'
true, Inside 'while'
true, Inside 'while'
true, Inside 'while'
...________...________...________...________...
true, Inside 'while'
true, Inside 'while'
true, Inside 'while'
true, Inside 'while'
false, Exited 'while'
```

`condition()` 方法使用到了 **Math** 庫的**靜態**方法 `random()`。該方法的作用是產生 0 和 1 之間 (包括 0，但不包括 1) 的一個 **double** 值。

**result** 的值是透過比較運算符 `<` 產生的 **boolean** 類型的結果。當控制台輸出 **boolean** 型值時，會自動將其轉換為對應的文字形式 `true` 或 `false`。此處 `while` 條件表達式代表：“僅在 `condition()` 返回 `false` 時停止循環”。


### do-while

**do-while** 的格式如下：

```java
do 
	statement
while(Boolean-expression);
```

**while** 和 **do-while** 之間的唯一區別是：即使條件表達式返回結果為 `false`， **do-while** 語句也至少會執行一次。 在 **while** 循環體中，如布爾表達式首次返回的結果就為 `false`，那麼循環體內的語句不會被執行。實際應用中，**while** 形式比 **do-while** 更為常用。


### for

**for** 循環可能是最常用的疊代形式。 該循環在第一次疊代之前執行初始化。隨後，它會執行布爾表達式，並在每次疊代結束時，進行某種形式的步進。**for** 循環的形式是：

```java
for(initialization; Boolean-expression; step)
  statement
```

初始化 (initialization) 表達式、布爾表達式 (Boolean-expression) ，或者步進 (step) 運算，都可以為空。每次疊代之前都會判斷布爾表達式的結果是否成立。一旦計算結果為 `false`，則跳出 **for** 循環體並繼續執行後面程式碼。 每次循環結束時，都會執行一次步進。

**for** 循環通常用於“計數”任務。程式碼範例：

```java
// control/ListCharacters.java

public class ListCharacters {
  public static void main(String[] args) {
    for(char c = 0; c < 128; c++)
      if(Character.isLowerCase(c))
        System.out.println("value: " + (int)c +
          " character: " + c);
  }
}
```

輸出結果（前 10 行）：

```
value: 97 character: a
value: 98 character: b
value: 99 character: c
value: 100 character: d
value: 101 character: e
value: 102 character: f
value: 103 character: g
value: 104 character: h
value: 105 character: i
value: 106 character: j
  ...
```


**注意**：變數 **c** 是在 **for** 循環執行時才被定義的，並不是在主方法的開頭。**c** 的作用域範圍僅在 **for** 循環體內。

傳統的程序導向語言如 C 需要先在程式碼塊（block）前定義好所有變數才能夠使用。這樣編譯器才能在建立塊時，為這些變數分配記憶體空間。在 Java 和 C++ 中，我們可以在整個塊使用變數聲明，並且可以在需要時才定義變數。 這種自然的編碼風格使我們的程式碼更容易被人理解 [^1]。

上例使用了 **java.lang.Character** 包裝類，該類不僅包含了基本類型 `char` 的值，還封裝了一些有用的方法。例如這裡就用到了靜態方法 `isLowerCase()` 來判斷字元是否為小寫。

<!--The Comma Operator-->

#### 逗號操作符

在 Java 中逗號運算符（這裡並非指我們平常用於分隔定義和方法參數的逗號分隔符）僅有一種用法：在 **for** 循環的初始化和步進控制中定義多個變數。我們可以使用逗號分隔多個語句，並按順序計算這些語句。**注意**：要求定義的變數類型相同。程式碼範例：

```java
// control/CommaOperator.java

public class CommaOperator {
  public static void main(String[] args) {
    for(int i = 1, j = i + 10; i < 5; i++, j = i * 2) {
      System.out.println("i = " + i + " j = " + j);
    }
  }
}
```

輸出結果：

```
i = 1 j = 11
i = 2 j = 4
i = 3 j = 6
i = 4 j = 8
```


上例中 **int** 類型聲明包含了 `i` 和 `j`。實際上，在初始化部分我們可以定義任意數量的同類型變數。**注意**：在 Java 中，僅允許 **for** 循環在控制表達式中定義變數。 我們不能將此方法與其他的循環語句和選擇語句中一起使用。同時，我們可以看到：無論在初始化還是在步進部分，語句都是順序執行的。

## for-in 語法

Java 5 引入了更為簡潔的“增強版 **for** 循環”語法來操縱陣列和集合。（更多細節，可參考 [陣列](./21-Arrays.md) 和 [集合](./12-Collections.md) 章節內容）。大部分文件也稱其為 **for-each** 語法，但因為了不與 Java 8 新添的 `forEach()` 產生混淆，因此我稱之為 **for-in** 循環。 （Python 已有類似的先例，如：**for x in sequence**）。**注意**：你可能會在其他地方看到不同叫法。

**for-in** 無需你去建立 **int** 變數和步進來控制循環計數。 下面我們來遍歷獲取 **float** 陣列中的元素。程式碼範例：

```java
// control/ForInFloat.java

import java.util.*;

public class ForInFloat {
  public static void main(String[] args) {
    Random rand = new Random(47);
    float[] f = new float[10];
    for(int i = 0; i < 10; i++)
      f[i] = rand.nextFloat();
    for(float x : f)
      System.out.println(x);
  }
}
```

輸出結果：

```
0.72711575
0.39982635
0.5309454
0.0534122
0.16020656
0.57799757
0.18847865
0.4170137
0.51660204
0.73734957
```

上例中我們展示了傳統 **for** 循環的用法。接下來再來看一下 **for-in** 的用法。程式碼範例：

```java
for(float x : f) {
```

這條語句定義了一個 **float** 類型的變數 `x`，繼而將每一個 `f` 的元素賦值給它。

任何一個返回陣列的方法都可以使用 **for-in** 循環語法來遍曆元素。例如 **String** 類有一個方法 `toCharArray()`，返回值類型為 **char** 陣列，我們可以很容易地在 **for-in** 循環中遍歷它。程式碼範例：

```java
// control/ForInString.java

public class ForInString {
  public static void main(String[] args) {
    for(char c: "An African Swallow".toCharArray())
      System.out.print(c + " ");
  }
}
```

輸出結果：

```
A n   A f r i c a n   S w a l l o w
```

很快我們能在 [集合](./12-Collections.md) 章節裡學習到，**for-in** 循環適用於任何可疊代（*iterable*）的 物件。

通常，**for** 循環語句都會在一個整型數值序列中步進。程式碼範例：

```java
for(int i = 0; i < 100; i++)
```

正因如此，除非先建立一個 **int** 陣列，否則我們無法使用 **for-in** 循環來操作。為簡化測試過程，我已在 `onjava` 包中封裝了 **Range** 類，利用其 `range()` 方法可自動生成恰當的陣列。

在 [封裝](./07-Implementation-Hiding.md)（Implementation Hiding）這一章裡我們介紹了靜態匯入（static import），無需了解細節就可以直接使用。 有關靜態匯入的語法，可以在 **import** 語句中看到：

```java
// control/ForInInt.java

import static onjava.Range.*;

public class ForInInt {
  public static void main(String[] args) {
    for(int i : range(10)) // 0..9
      System.out.print(i + " ");
    System.out.println();
    for(int i : range(5, 10)) // 5..9
      System.out.print(i + " ");
    System.out.println();
    for(int i : range(5, 20, 3)) // 5..20 step 3
      System.out.print(i + " ");
    System.out.println();
    for(int i : range(20, 5, -3)) // Count down
      System.out.print(i + " ");
    System.out.println();
  }
}
```

輸出結果：
```
0 1 2 3 4 5 6 7 8 9
5 6 7 8 9
5 8 11 14 17
20 17 14 11 8
```

`range()` 方法已被 [重載](./06-Housekeeping.md#方法重載)（重載：同名方法，參數列表或類型不同）。上例中 `range()` 方法有多種重載形式：第一種產生從 0 至範圍上限（不包含）的值；第二種產生參數一至參數二（不包含）範圍內的整數值；第三種形式有一個步進值，因此它每次的增量為該值；第四種 `range()` 表明還可以遞減。`range()` 無參方法是該生成器最簡單的版本。有關內容會在本書稍後介紹。

`range()` 的使用提高了程式碼可讀性，讓 **for-in** 循環在本書中適應更多的程式碼範例場景。

請注意，`System.out.print()` 不會輸出換行符，所以我們可以分段輸出同一行。

*for-in* 語法可以節省我們編寫程式碼的時間。 更重要的是，它提高了程式碼可讀性以及更好地描述程式碼意圖（獲取陣列的每個元素）而不是詳細說明這操作細節（建立索引，並用它來選擇陣列元素） 本書推薦使用 *for-in* 語法。

## return

在 Java 中有幾個關鍵字代表無條件分支，這意味無需任何測試即可發生。這些關鍵字包括 **return**，**break**，**continue** 和跳轉到帶標籤語句的方法，類似於其他語言中的 **goto**。

**return** 關鍵字有兩方面的作用：1.指定一個方法返回值 (在方法返回類型非 **void** 的情況下)；2.退出當前方法，並返回作用 1 中值。我們可以利用 `return` 的這些特點來改寫上例 `IfElse.java` 文件中的 `test()` 方法。程式碼範例：

```java
// control/TestWithReturn.java

public class TestWithReturn {
  static int test(int testval, int target) {
    if(testval > target)
      return +1;
    if(testval < target)
      return -1;
    return 0; // Match
  }

  public static void main(String[] args) {
    System.out.println(test(10, 5));
    System.out.println(test(5, 10));
    System.out.println(test(5, 5));
  }
}
```

輸出結果：

```
1
-1
0
```

這裡不需要 `else`，因為該方法執行到 `return` 就結束了。

如果在方法簽名中定義了返回值類型為 **void**，那麼在程式碼執行結束時會有一個隱式的 **return**。 也就是說我們不用在總是在方法中顯式地包含 **return** 語句。 **注意**：如果你的方法聲明的返回值類型為非 **void** 類型，那麼則必須確保每個程式碼路徑都返回一個值。

## break 和 continue

在任何疊代語句的主體內，都可以使用 **break** 和 **continue** 來控制循環的流程。 其中，**break** 表示跳出目前循環體。而 **continue** 表示停止本次循環，開始下一次循環。

下例向大家展示 **break** 和 **continue** 在 **for**、**while** 循環中的使用。程式碼範例：

```java
// control/BreakAndContinue.java
// Break 和 continue 關鍵字

import static onjava.Range.*;

public class BreakAndContinue {
  public static void main(String[] args) {
    for(int i = 0; i < 100; i++) { // [1]
      if(i == 74) break; // 跳出循環
      if(i % 9 != 0) continue; // 下一次循環
      System.out.print(i + " ");
    }
    System.out.println();
    // 使用 for-in 循環:
    for(int i : range(100)) { // [2]
      if(i == 74) break; // 跳出循環
      if(i % 9 != 0) continue; // 下一次循環
      System.out.print(i + " ");
    }
    System.out.println();
    int i = 0;
    //  "無限循環":
    while(true) { // [3]
      i++;
      int j = i * 27;
      if(j == 1269) break; // 跳出循環
      if(i % 10 != 0) continue; // 循環頂部
      System.out.print(i + " ");
    }
  }
}
```

輸出結果:

```
0 9 18 27 36 45 54 63 72
0 9 18 27 36 45 54 63 72
10 20 30 40
```

  <sub>**[1]** 在這個 **for** 循環中，`i` 的值永遠不會達到 100，因為一旦 `i` 等於 74，**break** 語句就會中斷循環。通常，只有在不知道中斷條件何時滿足時，才需要 **break**。因為 `i` 不能被 9 整除，**continue** 語句就會使循環從頭開始。這使 **i** 遞增)。如果能夠整除，則將值顯示出來。</sub>
  <sub>**[2]** 使用 **for-in** 語法，結果相同。</sub>
  <sub>**[3]** 無限 **while** 循環。循環內的 **break** 語句可中止循環。**注意**，**continue** 語句可將控制權移回循環的頂部，而不會執行 **continue** 之後的任何操作。 因此，只有當 `i` 的值可被 10 整除時才會輸出。在輸出中，顯示值 0，因為 `0％9` 產生 0。還有一種無限循環的形式： `for(;;)`。 在編譯器看來，它與 `while(true)` 無異，使用哪種完全取決於你的程式品味。</sub>

<!--The Infamous “Goto”-->
## 惡名昭彰的 goto

[**goto** 關鍵字](https://en.wikipedia.org/wiki/Goto) 很早就在程式設計語言中出現。事實上，**goto** 起源於[組語](https://en.wikipedia.org/wiki/Assembly_language)（assembly language）語言中的程式控制：“若條件 A 成立，則跳到這裡；否則跳到那裡”。如果你讀過由編譯器編譯後的程式碼，你會發現在其程式控制中充斥了大量的跳轉。較之組語產生的程式碼直接執行在硬體 CPU 中，Java 也會產生自己的“組語程式碼”（位元組碼），只不過它是執行在 Java 虛擬機裡的（Java Virtual Machine）。

一個原始碼級別跳轉的 **goto**，為何招致名譽掃地呢？若程式總是從一處跳轉到另一處，還有什麼辦法能識別程式碼的控制流程呢？隨著 *Edsger Dijkstra*發表著名的 “Goto 有害” 論（*Goto considered harmful*）以後，**goto** 便從此失寵。甚至有人建議將它從關鍵字中剔除。

正如上述提及的經典情況，我們不應走向兩個極端。問題不在 **goto**，而在於過度使用 **goto**。在極少數情況下，**goto** 實際上是控制流程的最佳方式。

儘管 **goto** 仍是 Java 的一個保留字，但其並未被正式啟用。可以說， Java 中並不支援 **goto**。然而，在 **break** 和 **continue** 這兩個關鍵字的身上，我們仍能看出一些 **goto** 的影子。它們並不屬於一次跳轉，而是中斷循環語句的一種方法。之所以把它們納入 **goto** 問題中一起討論，是由於它們使用了相同的機制：標籤。

“標籤”是後面跟一個冒號的標識符。程式碼範例：

```java
label1:
```

對 Java 來說，唯一用到標籤的地方是在循環語句之前。進一步說，它實際需要緊靠在循環語句的前方 —— 在標籤和循環之間置入任何語句都是不明智的。而在循環之前設定標籤的唯一理由是：我們希望在其中嵌套另一個循環或者一個開關。這是由於 **break** 和 **continue** 關鍵字通常只中斷目前循環，但若搭配標籤一起使用，它們就會中斷並跳轉到標籤所在的地方開始執行。程式碼範例：

```java
label1:
outer-iteration { 
  inner-iteration {
  // ...
  break; // [1] 
  // ...
  continue; // [2] 
  // ...
  continue label1; // [3] 
  // ...
  break label1; // [4] 
  } 
}
```

<sub>**[1]** **break** 中斷內部循環，並在外部循環結束。</sub>
<sub>**[2]** **continue** 移回內部循環的起始處。但在條件 3 中，**continue label1** 卻同時中斷內部循環以及外部循環，並移至 **label1** 處。</sub>
<sub>**[3]** 隨後，它實際是繼續循環，但卻從外部循環開始。</sub>
<sub>**[4]** **break label1** 也會中斷所有循環，並回到 **label1** 處，但並不重新進入循環。也就是說，它實際是完全中止了兩個循環。</sub>

下面是 **for** 循環的一個例子：

```java
// control/LabeledFor.java
// 搭配“標籤 break”的 for 循環中使用 break 和 continue

public class LabeledFor {
  public static void main(String[] args) {
    int i = 0;
    outer: // 此處不允許存在執行語句
    for(; true ;) { // 無限循環
      inner: // 此處不允許存在執行語句
      for(; i < 10; i++) {
        System.out.println("i = " + i);
        if(i == 2) {
          System.out.println("continue");
          continue;
        }
        if(i == 3) {
          System.out.println("break");
          i++; // 否則 i 永遠無法獲得自增 
               // 獲得自增 
          break;
        }
        if(i == 7) {
          System.out.println("continue outer");
          i++;  // 否則 i 永遠無法獲得自增 
                // 獲得自增 
          continue outer;
        }
        if(i == 8) {
          System.out.println("break outer");
          break outer;
        }
        for(int k = 0; k < 5; k++) {
          if(k == 3) {
            System.out.println("continue inner");
            continue inner;
          }
        }
      }
    }
    // 在此處無法 break 或 continue 標籤
  }
}
```

輸出結果：

```
i = 0
continue inner
i = 1
continue inner
i = 2
continue
i = 3
break
i = 4
continue inner
i = 5
continue inner
i = 6
continue inner
i = 7
continue outer
i = 8
break outer
```


注意 **break** 會中斷 **for** 循環，而且在抵達 **for** 循環的末尾之前，遞增表達式不會執行。由於 **break** 跳過了遞增表達式，所以遞增會在 `i==3` 的情況下直接執行。在 `i==7` 的情況下，`continue outer` 語句也會到達循環頂部，而且也會跳過遞增，所以它也是直接遞增的。

如果沒有 **break outer** 語句，就沒有辦法在一個內部循環裡找到出外部循環的路徑。這是由於 **break** 本身只能中斷最內層的循環（對於 **continue** 同樣如此）。 當然，若想在中斷循環的同時退出方法，簡單地用一個 **return** 即可。

下面這個例子向大家展示了帶標籤的 **break** 以及 **continue** 語句在 **while** 循環中的用法：

```java
// control/LabeledWhile.java
// 帶標籤的 break 和 conitue 在 while 循環中的使用

public class LabeledWhile {
  public static void main(String[] args) {
    int i = 0;
    outer:
    while(true) {
      System.out.println("Outer while loop");
      while(true) {
        i++;
        System.out.println("i = " + i);
        if(i == 1) {
          System.out.println("continue");
          continue;
        }
        if(i == 3) {
          System.out.println("continue outer");
          continue outer;
        }
        if(i == 5) {
          System.out.println("break");
          break;
        }
        if(i == 7) {
          System.out.println("break outer");
          break outer;
        }
      }
    }
  }
}
```

輸出結果：

```
Outer while loop
i = 1
continue
i = 2
i = 3
continue outer
Outer while loop
i = 4
i = 5
break
Outer while loop
i = 6
i = 7
break outer
```

同樣的規則亦適用於 **while**：

1. 簡單的一個 **continue** 會退回最內層循環的開頭（頂部），並繼續執行。

2. 帶有標籤的 **continue** 會到達標籤的位置，並重新進入緊接在那個標籤後面的循環。

3. **break** 會中斷目前循環，並移離目前標籤的末尾。

4. 帶標籤的 **break** 會中斷目前循環，並移離由那個標籤指示的循環的末尾。

大家要記住的重點是：在 Java 裡需要使用標籤的唯一理由就是因為有循環嵌套存在，而且想從多層嵌套中 **break** 或 **continue**。

**break** 和 **continue** 標籤在編碼中的使用頻率相對較低 (此前的語言中很少使用或沒有先例)，所以我們很少在程式碼裡看到它們。

在 *Dijkstra* 的 **“Goto 有害”** 論文中，他最反對的就是標籤，而非 **goto**。他觀察到 BUG 的數量似乎隨著程式中標籤的數量而增加[^2]。標籤和 **goto** 使得程式難以分析。但是，Java 標籤不會造成這方面的問題，因為它們的應用場景受到限制，無法用於以臨時方式傳輸控制。由此也引出了一個有趣的情形：對語言能力的限制，反而使它這項特性更加有價值。


## switch

**switch** 有時也被劃歸為一種選擇語句。根據整數表達式的值，**switch** 語句可以從一系列程式碼中選出一段去執行。它的格式如下：

```java
switch(integral-selector) {
	case integral-value1 : statement; break;
	case integral-value2 : statement;	break;
	case integral-value3 : statement;	break;
	case integral-value4 : statement;	break;
	case integral-value5 : statement;	break;
	// ...
	default: statement;
}
```

其中，**integral-selector** （整數選擇因子）是一個能夠產生整數值的表達式，**switch** 能夠將這個表達式的結果與每個 **integral-value** （整數值）相比較。若發現相符的，就執行對應的語句（簡單或複合語句，其中並不需要括號）。若沒有發現相符的，就執行  **default** 語句。

在上面的定義中，大家會注意到每個 **case** 均以一個 **break** 結尾。這樣可使執行流程跳轉至 **switch** 主體的末尾。這是構建 **switch** 語句的一種傳統方式，但 **break** 是可選的。若省略 **break，** 會繼續執行後面的 **case** 語句的程式碼，直到遇到一個 **break** 為止。通常我們不想出現這種情況，但對有經驗的程式設計師來說，也許能夠善加利用。注意最後的 **default** 語句沒有 **break**，因為執行流程已到了 **break** 的跳轉目的地。當然，如果考慮到編程風格方面的原因，完全可以在 **default** 語句的末尾放置一個 **break**，儘管它並沒有任何實際的作用。

**switch** 語句是一種實現多路選擇的乾淨俐落的一種方式（比如從一系列執行路徑中挑選一個）。但它要求使用一個選擇因子，並且必須是 **int** 或 **char** 那樣的整數值。例如，假若將一個字串或者浮點數作為選擇因子使用，那麼它們在 switch 語句裡是不會工作的。對於非整數類型（Java 7 以上版本中的 String 型除外），則必須使用一系列 **if** 語句。 在[下一章的結尾](./06-Housekeeping.md#列舉類型) 中，我們將會了解到**列舉類型**被用來搭配 **switch** 工作，並優雅地解決了這種限制。

下面這個例子可隨機生成字母，並判斷它們是母音還是子音字母：

```java
// control/VowelsAndConsonants.java

// switch 執行語句的示範
import java.util.*;

public class VowelsAndConsonants {
  public static void main(String[] args) {
    Random rand = new Random(47);
    for(int i = 0; i < 100; i++) {
      int c = rand.nextInt(26) + 'a';
      System.out.print((char)c + ", " + c + ": ");
      switch(c) {
        case 'a':
        case 'e':
        case 'i':
        case 'o':
        case 'u': System.out.println("vowel");
                  break;
        case 'y':
        case 'w': System.out.println("Sometimes vowel");
                  break;
        default:  System.out.println("consonant");
      }
    }
  }
}
```

輸出結果：

```
y, 121: Sometimes vowel
n, 110: consonant
z, 122: consonant
b, 98: consonant
r, 114: consonant
n, 110: consonant
y, 121: Sometimes vowel
g, 103: consonant
c, 99: consonant
f, 102: consonant
o, 111: vowel
w, 119: Sometimes vowel
z, 122: consonant
  ...
```

由於 `Random.nextInt(26)` 會產生 0 到 25 之間的一個值，所以在其上加上一個偏移量 `a`，即可產生小寫字母。在 **case** 語句中，使用單引號引起的字元也會產生用於比較的整數值。

請注意 **case** 語句能夠堆疊在一起，為一段程式碼形成多重匹配，即只要符合多種條件中的一種，就執行那段特別的程式碼。這時也應該注意將 **break** 語句置於特定 **case** 的末尾，否則控制流程會繼續往下執行，處理後面的 **case**。在下面的語句中：

```java
int c = rand.nextInt(26) + 'a';
```

此處 `Random.nextInt()` 將產生 0~25 之間的一個隨機 **int** 值，它將被加到 `a` 上。這表示 `a` 將自動被轉換為 **int** 以執行加法。為了把 `c` 當作字元列印，必須將其轉型為 **char**；否則，將會輸出整數。

<!-- Switching on Strings -->

## switch 字串

Java 7 增加了在字串上 **switch** 的用法。 下例展示了從一組 **String** 中選擇可能值的傳統方法，以及新式方法：

```java
// control/StringSwitch.java

public class StringSwitch {
  public static void main(String[] args) {
    String color = "red";
    // 老的方式: 使用 if-then 判斷
    if("red".equals(color)) {
      System.out.println("RED");
    } else if("green".equals(color)) {
      System.out.println("GREEN");
    } else if("blue".equals(color)) {
      System.out.println("BLUE");
    } else if("yellow".equals(color)) {
      System.out.println("YELLOW");
    } else {
      System.out.println("Unknown");
    }
    // 新的方法: 字串搭配 switch
    switch(color) {
      case "red":
        System.out.println("RED");
        break;
      case "green":
        System.out.println("GREEN");
        break;
      case "blue":
        System.out.println("BLUE");
        break;
      case "yellow":
        System.out.println("YELLOW");
        break;
      default:
        System.out.println("Unknown");
        break;
    }
  }
}
```

輸出結果：

```
RED
RED
```


一旦理解了 **switch**，你會明白這其實就是一個邏輯擴展的語法糖。新的編碼方式能使得結果更清晰，更易於理解和維護。

作為 **switch** 字串的第二個例子，我們重新訪問 `Math.random()`。 它是否產生從 0 到 1 的值，包括還是不包括值 1 呢？在數學術語中，它屬於 (0,1)、[0,1)、(0,1]、[0,1] 中的哪種呢？（方括號表示“包括”，而括號表示“不包括”）

下面是一個可能提供答案的測試程式。 所有命令列參數都作為 **String** 物件傳遞，因此我們可以 **switch** 參數來決定要做什麼。 那麼問題來了：如果使用者不提供參數 ，索引到 `args` 的陣列就會導致程式失敗。 解決這個問題，我們需要預先檢查陣列的長度，若長度為 0，則使用**空字串** `""` 替代；否則，選擇 `args` 陣列中的第一個元素：

```java
// control/RandomBounds.java

// Math.random() 會產生 0.0 和 1.0 嗎？
// {java RandomBounds lower}
import onjava.*;

public class RandomBounds {
  public static void main(String[] args) {
    new TimedAbort(3);
    switch(args.length == 0 ? "" : args[0]) {
      case "lower":
        while(Math.random() != 0.0)
          ; // 保持重試
        System.out.println("Produced 0.0!");
        break;
      case "upper":
        while(Math.random() != 1.0)
          ; // 保持重試
        System.out.println("Produced 1.0!");
        break;
      default:
        System.out.println("Usage:");
        System.out.println("\tRandomBounds lower");
        System.out.println("\tRandomBounds upper");
        System.exit(1);
    }
  }
}
```

要執行該程式，請鍵入以下任一指令：

```java
java RandomBounds lower 
// 或者
java RandomBounds upper
```

使用 `onjava` 包中的 **TimedAbort** 類可使程式在三秒後中止。從結果來看，似乎 `Math.random()` 產生的隨機值裡不包含 0.0 或 1.0。 這就是該測試容易混淆的地方：若要考慮 0 至 1 之間所有不同 **double** 數值的可能性，那麼這個測試的耗費的時間可能超出一個人的壽命了。 這裡我們直接給出正確的結果：`Math.random()` 的結果集範圍包含 0.0 ，不包含 1.0。 在數學術語中，可用 [0,1) 來表示。由此可知，我們必須小心分析實驗並了解它們的局限性。


## 本章小結

本章總結了我們對大多數程式語言中出現的基本特性的探索：計算，運算符優先度，類型轉換，選擇和疊代。 現在讓我們準備好，開始步入物件導向和函數式編程的世界吧。 下一章的內容涵蓋了 Java 編程中的重要問題：物件的[初始化和清理](./06-Housekeeping.md)。緊接著，還會介紹[封裝](./07-Implementation-Hiding.md)（implementation hiding）的核心概念。

<!--下面是腳註-->
[^1]: 在早期的語言中，許多決策都是基於讓編譯器設計者的體驗更好。 但在現代語言設計中，許多決策都是為了提高語言使用者的體驗，儘管有時會有妥協 —— 這通常會讓語言設計者後悔。

[^2]: **注意**，此處觀點似乎難以讓人信服，很可能只是一個因認知偏差而造成的[因果關係謬誤](https://en.wikipedia.org/wiki/Correlation_does_not_imply_causation) 的例子。

<!-- 分頁 -->

<div style="page-break-after: always;"></div>
