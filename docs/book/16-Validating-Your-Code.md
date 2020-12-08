[TOC]

<!-- Validating Your Code -->

# 第十六章 程式碼校驗

>你永遠不能保證你的程式碼是正確的，你只能證明它是錯的。

讓我們先暫停程式語言特性的學習，看看一些程式碼基礎知識。特別是能讓你的程式碼更加健壯的知識。

<!-- Testing -->

## 測試

### 如果沒有測試過，它就是不能工作的。

Java是一個靜態類型的語言，程式設計師經常對一種程式語言明顯的安全性感到過於舒適，“能透過編譯器，那就是沒問題的”。但靜態類型檢查是一種非常局限性的測試，只是說明編譯器接受你程式碼中的語法和基本類型規則，並不意味著你的程式碼達到程式的目標。隨著你程式碼經驗的豐富，你逐漸了解到你的程式碼從來沒有滿足過這些目標。邁向程式碼校驗的第一步就是建立測試，針對你的目標檢查程式碼的行為。

#### 單元測試

這個過程是將整合測試構建到你建立的所有程式碼中，並在每次構建系統時執行這些測試。這樣，構建過程不僅能檢查語法的錯誤，同時也能檢查語義的錯誤。

“單元”是指測試一小部分程式碼 。通常，每個類都有測試來檢查它所有方法的行為。“系統”測試則是不同的，它檢查的是整個程式是否滿足要求。

C 風格的語言，尤其是 C++，通常會認為性能比安全更重要。用 Java 編程比 C++（一般認為大概快兩倍）快的原因是 Java 的安全性保障：比如垃圾回收以及改良的類型檢測等特性。透過將單元測試整合到構建過程中，你擴大了這個安全保障，因而有了更快的開發效率。當發現設計或實現的缺陷時，可以更容易、更大膽地重構你的程式碼。

我自己的測試經歷開始於我意識到要確保書中程式碼的正確性，書中的所有程式必須能夠透過合適的構建系統自動提取、編譯。這本書所使用的構建系統是 Gradle。 你只要在安裝 JDK 後輸入 **gradlew compileJava**，就能編譯本書的所有程式碼。自動提取和自動編譯的效果對本書程式碼的質量是如此的直接和引人注目，（在我看來）這會很快成為任何編程書籍的必備條件——你怎麼能相信沒有編譯的程式碼呢? 我還發現我可以使用搜尋和取代在整本書進行大範圍的修改，如果引入了一個錯誤，程式碼提取器和構建系統就會清除它。隨著程式越來越複雜，我在系統中發現了一個嚴重的漏洞。編譯程式毫無疑問是重要的第一步， 對於一本要出版的書而言，這看來是相當具有革命意義的發現（由於出版壓力， 你經常打開一本程式設計的書會發現書中程式碼的錯誤）。但是，我收到了來自讀者回饋程式碼中存在語義問題。當然，這些問題可以透過執行程式碼發現。我在早期實現一個自動化執行測試系統時嘗試了一些不太有效的方式，但迫於出版壓力，我明白我的程式絕對有問題，並會以 bug 報告的方式讓我自食惡果。我也經常收到讀者的抱怨說，我沒有顯示足夠的程式碼輸出。我需要驗證程式的輸出，並且在書中顯示驗證的輸出。我以前的意見是讀者應該一邊看書一邊執行程式碼，許多讀者就是這麼做的並且從中受益。然而，這種態度背後的原因是，我無法保證書中的輸出是正確的。從經驗來看，我知道隨著時間的推移，會發生一些事情，使得輸出不再正確（或者一開始就不正確）。為了解決這個問題，我利用 Python 建立了一個工具（你將在下載的範例中找到此工具）。本書中的大多數程式都產生控制台輸出，該工具將該輸出與原始碼清單末尾的注釋中顯示的預期輸出進行比較，所以讀者可以看到預期的輸出，並且知道這個輸出已經被構建程式驗證過。

#### JUnit

最初的 JUnit 發布於 2000 年，大概是基於 Java 1.0，因此不能使用 Java 的反射工具。因此，用舊的 JUnit 編寫單元測試是一項相當繁忙和冗長的工作。我發現這個設計令人不爽，並編寫了自己的單元測試框架作為 [註解](./Annotations.md) 一章的範例。這個框架走向了另一個極端，“嘗試最簡單可行的方法”（極限編程中的一個關鍵短語）。從那之後，JUnit 透過反射和註解得到了極大的改進，大大簡化了編寫單元測試程式碼的過程。在 Java8 中，他們甚至增加了對 lambdas 表達式的支援。本書使用當時最新的 Junit5 版本

在 JUnit 最簡單的使用中，使用 **@Test** 註解標記表示測試的每個方法。JUnit 將這些方法標識為單獨的測試，並一次設定和執行一個測試，採取措施避免測試之間的副作用。

讓我們嘗試一個簡單的例子。**CountedList** 繼承 **ArrayList** ，添加訊息來追蹤有多少個 **CountedLists** 被建立：

```java
// validating/CountedList.java
// Keeps track of how many of itself are created.
package validating;
import java.util.*;	
public class CountedList extends ArrayList<String> {
    private static int counter = 0;
    private int id = counter++;
    public CountedList() {
        System.out.println("CountedList #" + id);
    }
	public int getId() { return id; }
}
```

標準做法是將測試放在它們自己的子目錄中。測試還必須放在包中，以便 JUnit 能發現它們:

```java
// validating/tests/CountedListTest.java
// Simple use of JUnit to test CountedList.
package validating;
import java.util.*;
import org.junit.jupiter.api.*;
import static org.junit.jupiter.api.Assertions.*;
public class CountedListTest {
private CountedList list;
	@BeforeAll
    static void beforeAllMsg() {
        System.out.println(">>> Starting CountedListTest");
    }
    
    @AfterAll
    static void afterAllMsg() {
        System.out.println(">>> Finished CountedListTest");
    }
    
    @BeforeEach
    public void initialize() {
    	list = new CountedList();
    	System.out.println("Set up for " + list.getId());
        for(int i = 0; i < 3; i++)
            list.add(Integer.toString(i));
    }
    
    @AfterEach
    public void cleanup() {
    	System.out.println("Cleaning up " + list.getId());
    }
    
    @Test
    public void insert() {
        System.out.println("Running testInsert()");
        assertEquals(list.size(), 3);
        list.add(1, "Insert");
        assertEquals(list.size(), 4);
        assertEquals(list.get(1), "Insert");
    }
    
    @Test
    public void replace() {
    	System.out.println("Running testReplace()");
    	assertEquals(list.size(), 3);
    	list.set(1, "Replace");
   		assertEquals(list.size(), 3);
    	assertEquals(list.get(1), "Replace");
    }
    	
    // A helper method to simplify the code. As
    // long as it's not annotated with @Test, it will
    // not be automatically executed by JUnit.
    private void compare(List<String> lst, String[] strs) {
        assertArrayEquals(lst.toArray(new String[0]), strs);
    }
    
    @Test
    public void order() {
        System.out.println("Running testOrder()");
        compare(list, new String[] { "0", "1", "2" });
    }
    
    @Test
    public void remove() {
    	System.out.println("Running testRemove()");
    	assertEquals(list.size(), 3);
    	list.remove(1);
    	assertEquals(list.size(), 2);
    	compare(list, new String[] { "0", "2" });
    }
    
    @Test
    public void addAll() {
    	System.out.println("Running testAddAll()");
    	list.addAll(Arrays.asList(new String[] {
    	"An", "African", "Swallow"}));
    	assertEquals(list.size(), 6);
    	compare(list, new String[] { "0", "1", "2",
    	"An", "African", "Swallow" });
    }
}

/* Output:
>>> Starting CountedListTest
CountedList #0
Set up for 0
Running testRemove()
Cleaning up 0
CountedList #1
Set up for 1
Running testReplace()
Cleaning up 1
CountedList #2
Set up for 2
Running testAddAll()
Cleaning up 2
CountedList #3
Set up for 3
Running testInsert()
Cleaning up 3
CountedList #4
Set up for 4
Running testOrder()
Cleaning up 4
>>> Finished CountedListTest
*/
```

**@BeforeAll** 註解是在任何其他測試操作之前執行一次的方法。 **@AfterAll** 是所有其他測試操作之後只執行一次的方法。兩個方法都必須是靜態的。

**@BeforeEach**註解是通常用於建立和初始化公共物件的方法，並在每次測試前執行。可以將所有這樣的初始化放在測試類的建構子中，儘管我認為 **@BeforeEach** 更加清晰。JUnit為每個測試建立一個物件，確保測試執行之間沒有副作用。然而，所有測試的所有物件都是同時建立的(而不是在測試之前建立物件)，所以使用 **@BeforeEach** 和建構子之間的唯一區別是 **@BeforeEach** 在測試前直接呼叫。在大多數情況下，這不是問題，如果你願意，可以使用建構子方法。

如果你必須在每次測試後執行清理（如果修改了需要復原的靜態文件，打開文件需要關閉，打開資料庫或者網路連接，etc），那就用註解 **@AfterEach**。

每個測試建立一個新的 **CountedListTest** 物件，任何非靜態成員變數也會在同一時間建立。然後為每個測試呼叫 **initialize()** ，於是 list 被賦值為一個新的用字串“0”、“1” 和 “2” 初始化的 **CountedList** 物件。觀察 **@BeforeEach** 和 **@AfterEach** 的行為，這些方法在初始化和清理測試時顯示有關測試的訊息。

**insert()** 和 **replace()** 示範了典型的測試方法。JUnit 使用 **@Test** 註解發現這些方法，並將每個方法作為測試執行。在方法內部，你可以執行任何所需的操作並使用 JUnit 斷言方法（以"assert"開頭）驗證測試的正確性（更全面的"assert"說明可以在 Junit 文件裡找到）。如果斷言失敗，將顯示導致失敗的表達式和值。這通常就足夠了，但是你也可以使用每個 JUnit 斷言語句的重載版本，它包含一個字串，以便在斷言失敗時顯示。

斷言語句不是必須的；你可以在沒有斷言的情況下執行測試，如果沒有異常，則認為測試是成功的。

**compare()** 是“helper方法”的一個例子，它不是由 JUnit 執行的，而是被類中的其他測試使用。只要沒有 **@Test** 註解，JUnit 就不會執行它，也不需要特定的簽名。在這裡，**compare()** 是私有方法 ，表示僅在測試類中使用，但他同樣可以是 **public** 。其餘的測試方法透過將其重構為 **compare()** 方法來消除重複的程式碼。

本書使用 **build.gradle** 控制測試，執行本章節的測試，使用指令：`gradlew validating:test`，Gradle 不會執行已經執行過的測試，所以如果你沒有得到測試結果，得先執行：`gradlew validating:clean`。

可以用下面這個指令執行本書的所有測試：

**gradlew test**

儘管可以用最簡單的方法，如 **CountedListTest.java** 所示那樣，JUnit 還包括大量的測試結構，你可以到[官網](junit.org)上學習它們。

JUnit 是 Java 最流行的單元測試框架，但也有其它可以替代的。你可以透過網際網路發現更適合的那一個。

#### 測試覆蓋率的幻覺

測試覆蓋率，同樣也稱為程式碼覆蓋率，度量程式碼的測試百分比。百分比越高，測試的覆蓋率越大。這裡有很多[方法](https://en.wikipedia.org/wiki/Code_coverage)

計算覆蓋率，還有有幫助的文章[Java程式碼覆蓋工具](https://en.wikipedia.org/wiki/Java_Code_Coverage_Tools)。

對於沒有知識但處於控制地位的人來說，很容易在沒有任何了解的情況下也有概念認為 100% 的測試覆蓋是唯一可接受的值。這有一個問題，因為 100% 並不意味著是對測試有效性的良好測量。你可以測試所有需要它的東西，但是只需要 65% 的覆蓋率。如果需要 100% 的覆蓋，你將浪費大量時間來生成剩餘的程式碼，並且在向項目添加程式碼時浪費的時間更多。

當分析一個未知的程式碼庫時，測試覆蓋率作為一個粗略的度量是有用的。如果覆蓋率工具報告的值特別低（比如，少於百分之40），則說明覆蓋不夠充分。然而，一個非常高的值也同樣值得懷疑，這表明對編程領域了解不足的人迫使團隊做出了武斷的決定。覆蓋工具的最佳用途是發現程式碼庫中未測試的部分。但是，不要依賴覆蓋率來得到測試質量的任何訊息。

<!-- Preconditions -->

## 前置條件

前置條件的概念來自於契約式設計(**Design By Contract, DbC**), 利用斷言機制實現。我們從 Java 的斷言機制開始來介紹 DBC，最後使用Google的 Guava 庫作為前置條件。

#### 斷言（Assertions）

斷言通過驗證在程式執行期間滿足某些條件，從而增加了程式的健壯性。舉例，假設在一個物件中有一個數值欄位表示日曆上的月份。這個數字總是介於 1-12 之間。斷言可以檢查這個數字，如果超出了該範圍，則報告錯誤。如果在方法的內部，則可以使用斷言檢查參數的有效性。這些是確保程式正確的重要測試，但是它們不能在編譯時被檢查，並且它們不屬於單元測試的範圍。

#### Java 斷言語法

你可以透過其它程式設計架構來模擬斷言的效果，因此，在 Java 中包含斷言的意義在於它們易於編寫。斷言語句有兩種形式 : 

assert boolean-expression；

assert boolean-expression: information-expression;

兩者似乎告訴我們 **“我斷言這個布爾表達式會產生 true”**， 否則，將拋出 **AssertionError** 異常。

**AssertionError** 是 **Throwable** 的衍生類，因此不需要異常說明。

不幸的是，第一種斷言形式的異常不會生成包含布爾表達式的任何訊息（與大多數其他語言的斷言機制相反）。

下面是第一種形式的例子：

```java
// validating/Assert1.java

// Non-informative style of assert
// Must run using -ea flag:
// {java -ea Assert1}
// {ThrowsException}
public class Assert1 {
    public static void main(String[] args) {
        assert false;
    }
}

/* Output:
___[ Error Output ]___
Exception in thread "main" java.lang.AssertionError
at Assert1.main(Assert1.java:9)
*/
```

如果你正常執行程式，沒有任何特殊的斷言標誌，則不會發生任何事情。你需要在執行程式時顯式啟用斷言。一種簡單的方法是使用 **-ea** 標誌， 它也可以表示為: **-enableassertion**， 這將執行程式並執行任何斷言語句。

輸出中並沒有包含多少有用的訊息。另一方面，如果你使用 **information-expression** ， 將生成一條有用的消息作為異常堆疊跟蹤的一部分。最有用的 **information-expression** 通常是一串針對程式設計師的文字：

```java
// validating/Assert2.java
// Assert with an information-expression
// {java Assert2 -ea}
// {ThrowsException}

public class Assert2 {
    public static void main(String[] args) {
    assert false:
    "Here's a message saying what happened";
    }
}
/* Output:
___[ Error Output ]___
Exception in thread "main" java.lang.AssertionError:
Here's a message saying what happened
at Assert2.main(Assert2.java:8)
*/
```

**information-expression** 可以產生任何類型的物件，因此，通常將構造一個包含物件值的更複雜的字串，它包含失敗的斷言。

你還可以基於類名或包名打開或關閉斷言；也就是說，你可以對整個包啟用或禁用斷言。實現這一點的詳細訊息在 JDK 的斷言文件中。此特性對於使用斷言的大型項目來說很有用當你想打開或關閉某些斷言時。但是，日誌記錄（*Logging*）或者除錯（*Debugging*）,可能是捕獲這類訊息的更好工具。

你還可以透過編程的方式透過連結到類載入器物件（**ClassLoader**）來控制斷言。類載入器中有幾種方法允許動態啟用和禁用斷言，其中 **setDefaultAssertionStatus ()** ,它為之後載入的所有類設定斷言狀態。因此，你可以像下面這樣悄悄地開啟斷言：

```java
// validating/LoaderAssertions.java
// Using the class loader to enable assertions
// {ThrowsException}
public class LoaderAssertions {
public static void main(String[] args) {

	ClassLoader.getSystemClassLoader().
        setDefaultAssertionStatus(true);
		new Loaded().go();
	}
}

class Loaded {
    public void go() {
    assert false: "Loaded.go()";
    }
}
/* Output:
___[ Error Output ]___
Exception in thread "main" java.lang.AssertionError:
Loaded.go()
at Loaded.go(LoaderAssertions.java:15)
at
LoaderAssertions.main(LoaderAssertions.java:9)
*/
```

這消除了在執行程式時在命令列上使用 **-ea** 標誌的需要，使用 **-ea** 標誌啟用斷言可能同樣簡單。當交付獨立產品時，可能必須設定一個執行腳本讓使用者能夠啟動程式，配置其他啟動參數，這麼做是有意義的。然而，決定在程式執行時啟用斷言可以使用下面的 **static** 塊來實現這一點，該語句位於系統的主類中：

```java
static {
    boolean assertionsEnabled = false;
    // Note intentional side effect of assignment:
    assert assertionsEnabled = true;
    if(!assertionsEnabled)
        throw new RuntimeException("Assertions disabled");
}
```

如果啟用斷言，然後執行 **assert** 語句，**assertionsEnabled** 變為 **true** 。斷言不會失敗，因為分配的返回值是賦值的值。如果不啟用斷言，**assert** 語句不執行，**assertionsEnabled** 保持false，將導致異常。

#### Guava斷言

因為啟用 Java 本機斷言很麻煩，Guava 團隊添加一個始終啟用的用來取代斷言的 **Verify** 類。他們建議靜態匯入 **Verify** 方法：

```java
// validating/GuavaAssertions.java
// Assertions that are always enabled.

import com.google.common.base.*;
import static com.google.common.base.Verify.*;
public class GuavaAssertions {
    public static void main(String[] args) {
    	verify(2 + 2 == 4);
    	try {
    		verify(1 + 2 == 4);
   	 	} catch(VerifyException e) {
    		System.out.println(e);
    	}
        
		try {
			verify(1 + 2 == 4, "Bad math");
		} catch(VerifyException e) {
			System.out.println(e.getMessage());
		}
        
		try {
			verify(1 + 2 == 4, "Bad math: %s", "not 4");
		} catch(VerifyException e) {
        	System.out.println(e.getMessage());
        }
        
        String s = "";
        s = verifyNotNull(s);
        s = null;
        try {
            verifyNotNull(s);
        } catch(VerifyException e) {
        	System.out.println(e.getMessage());
        }
        
        try {
        	verifyNotNull(
        		s, "Shouldn't be null: %s", "arg s");
        } catch(VerifyException e) {
        	System.out.println(e.getMessage());
        }
	}
}
/* Output:
com.google.common.base.VerifyException
Bad math
Bad math: not 4
expected a non-null reference
Shouldn't be null: arg s
*/
```

這裡有兩個方法，使用變數 **verify()** 和 **verifyNotNull()** 來支援有用的錯誤消息。注意，**verifyNotNull()** 內建的錯誤消息通常就足夠了，而 **verify()** 太一般，沒有有用的預設錯誤消息。

#### 使用斷言進行契約式設計

*契約式設計(DbC)*是 Eiffel 語言的發明者 Bertrand Meyer 提出的一個概念，透過確保物件遵循某些規則來幫助建立健壯的程式。這些規則是由正在解決的問題的性質決定的，這超出了編譯器可以驗證的範圍。雖然斷言沒有直接實現 **DBC**（Eiffel 語言也是如此），但是它們建立了一種非正式的 DbC 編程風格。DbC 假定服務供應者與該服務的消費者或客戶之間存在明確指定的契約。在物件導向編程中，服務通常由物件提供，物件的邊界 — 供應者和消費者之間的劃分 — 是物件類的介面。當用戶端呼叫特定的公共方法時，它們希望該呼叫具有特定的行為：物件狀態改變，以及一個可預測的返回值。

**Meyer** 認為：

1.應該明確指定行為，就好像它是一個契約一樣。

2.透過實現某些執行時檢查來保證這種行為，他將這些檢查稱為前置條件、後置條件和不變項。

不管你是否同意，第一條總是對的，在大多數情況下，DbC 確實是一種有用的方法。（我認為，與任何解決方案一樣，它的有用性也有界限。但如果你知道這些界限，你就知道什麼時候去嘗試。）尤其是，設計過程中一個有價值的部分是特定類 DbC 約束的表達式；如果無法指定約束，則你可能對要構建的內容了解得不夠。

#### 檢查指令

詳細研究 DbC 之前，思考最簡單使用斷言的辦法，**Meyer** 稱它為檢查指令。檢查指令說明你確信程式碼中的某個特定屬性此時已經得到滿足。檢查指令的思想是在程式碼中表達非明顯性的結論，而不僅僅是為了驗證測試，也同樣為了將來能夠滿足閱讀者而有一個文件。

在化學領域，你也許會用一種純液體去滴定測量另一種液體，當達到一個特定的點時，液體變藍了。從兩個液體的顏色上並不能明顯看出；這是複雜反應的一部分。滴定完成後一個有用的檢查指令是能夠斷定液體變藍了。

檢查指令是對你的程式碼進行補充，當你可以測試並闡明物件或程式的狀態時，應該使用它。

#### 前置條件

前置條件確保用戶端(呼叫此方法的程式碼)履行其部分契約。這意味著在方法呼叫開始時幾乎總是會檢查參數（在你用那個方法做任何操作之前）以此保證它們的呼叫在方法中是合適的。因為你永遠無法知道用戶端會傳遞給你什麼，前置條件是確保檢查的一個好做法。

#### 後置條件

後置條件測試你在方法中所做的操作的結果。這段程式碼放在方法呼叫的末尾，在 **return** 語句之前(如果有的話)。對於長時間、複雜的方法，在返回計算結果之前需要對計算結果進行驗證（也就是說，在某些情況下，由於某種原因，你不能總是相信結果)，後置條件很重要，但是任何時候你可以描述方法結果上的約束時，最好將這些約束在程式碼中表示為後置條件。

#### 不變性

不變性保證了必須在方法呼叫之間維護的物件的狀態。但是，它並不會阻止方法在執行過程中暫時偏離這些保證，它只是在說物件的狀態訊息應該總是遵守狀態規則：

**1**. 在進入該方法時。

**2**.  在離開方法之前。

此外，不變性是構造後對於物件狀態的保證。

根據這個描述，一個有效的不變性被定義為一個方法，可能被命名為 **invariant()** ，它在構造之後以及每個方法的開始和結束時呼叫。方法以如下方式呼叫：

assert invariant();

這樣，如果出於性能原因禁用斷言，就不會產生開銷。

#### 放鬆 DbC 檢查或非嚴格的 DbC

儘管 Meyer 強調了前置條件、後置條件和不變性的價值以及在開發過程中使用它們的重要性，他承認在一個產品中包含所有 DbC 程式碼並不總是實際的。你可以基於對特定位置的程式碼的信任程度放鬆 DbC 檢查。以下是放鬆檢查的順序，最安全到最不安全：

**1**. 不變性檢查在每個方法一開始的時候是不能進行的，因為在每個方法結束的時候進行不變性檢查能保證一開始的時候物件處於有效狀態。也就是說，通常情況下，你可以相信物件的狀態不會在方法呼叫之間發生變化。這是一個非常安全的假設，你可以只在程式碼末尾使用不變性檢查來編寫程式碼。

**2**. 接下來禁用後置條件檢查，當你進行合理的單元測試以驗證方法是否返回了適當的值時。因為不變性檢查是觀察物件的狀態，後置條件檢查僅在方法期間驗證計算結果，因此可能會被丟棄，以便進行單元測試。單元測試不會像執行時後置條件檢查那樣安全，但是它可能已經足夠了，特別是當對自己的程式碼有信心時。

**3**. 如果你確信方法主體沒有把物件改成無效狀態，則可以禁用方法呼叫末尾的不變性檢查。可以透過白盒單元測試(透過訪問私有欄位的單元測試來驗證物件狀態)來驗證這一點。儘管它可能沒有呼叫 **invariant()** 那麼穩妥，可以將不變性檢查從執行時測試 “遷移” 到構建時測試(透過單元測試)，就像使用後置條件一樣。

**4**. 禁用前置條件檢查，但除非這是萬不得已的情況下。因為這是最不安全、最不明智的選擇，因為儘管你知道並且可以控制自己的程式碼，但是你無法控制用戶端可能會傳遞給方法的參數。然而，某些情況下對性能要求很高，透過分析得到前置條件造成了這個瓶頸，而且你有某種合理的保證用戶端不會違反前置條件(比如自己編寫用戶端的情況下)，那麼禁用前置條件檢查是可接受的。

你不應該直接刪除檢查的程式碼，而只需要禁用檢查(添加注釋)。這樣如果發現錯誤，就可以輕鬆地復原檢查以快速發現問題。

#### DbC + 單元測試

下面的例子示範了將契約式設計中的概念與單元測試相結合的有效性。它顯示了一個簡單的先進先出(FIFO)佇列，該佇列實現為一個“循環”陣列，即以循環方式使用的陣列。當到達陣列的末尾時，將繞回到開頭。

我們可以對這個佇列做一些契約定義:

**1**.   前置條件(用於put())：不允許將空元素添加到佇列中。

**2**.   前置條件(用於put())：將元素放入完整佇列是非法的。

**3**.   前置條件(用於get())：試圖從空佇列中獲取元素是非法的。

**4**.   後置條件用於get())：不能從陣列中生成空元素。

**5**.   不變性：包含物件的區域不能包含任何空元素。

**6**.   不變性：不包含物件的區域必須只有空值。

下面是實現這些規則的一種方式，為每個 DbC 元素類型使用顯式的方法呼叫。

首先，我們建立一個專用的 **Exception**：
```java
  // validating/CircularQueueException.java
  package validating;
  public class CircularQueueException extends RuntimeException {
          public CircularQueueException(String why) {
          super(why);
      }
  }
```
它用來報告 **CircularQueue** 中出現的錯誤：

```java
  // validating/CircularQueue.java
  // Demonstration of Design by Contract (DbC)
  package validating;
  import java.util.*;
  public class CircularQueue {
      private Object[] data;
      private int in = 0, // Next available storage space 
      out = 0; // Next gettable object
               // Has it wrapped around the circular queue?
      private boolean wrapped = false;
      public CircularQueue(int size) {
          data = new Object[size];
          // Must be true after construction:
          assert invariant();
      }
      
      public boolean empty() {
          return !wrapped && in == out;
      }
      
      public boolean full() {
      	  return wrapped && in == out;
      }
      
  	  public boolean isWrapped() { return wrapped; }
      
      public void put(Object item) {
      	  precondition(item != null, "put() null item");
      	  precondition(!full(),
      	  "put() into full CircularQueue");
      	  assert invariant();
      	  data[in++] = item;
      	  if(in >= data.length) {
              in = 0;
              wrapped = true;
      	  }
  		  assert invariant();
  	  }
      
      public Object get() {
      	  precondition(!empty(),
      	  "get() from empty CircularQueue");
      	  assert invariant();
      	  Object returnVal = data[out];
      	  data[out] = null;
      	  out++;
          if(out >= data.length) {
              out = 0;
              wrapped = false;
          }
          assert postcondition(
          returnVal != null,
          "Null item in CircularQueue");
          assert invariant();
          return returnVal;
      }
      
  	  // Design-by-contract support methods:
      private static void precondition(boolean cond, String msg) {
          if(!cond) throw new CircularQueueException(msg);
      }
      
      private static boolean postcondition(boolean cond, String msg) {
          if(!cond) throw new CircularQueueException(msg);
      	  return true;
      }
      
      private boolean invariant() {
          // Guarantee that no null values are in the
          // region of 'data' that holds objects:
          for(int i = out; i != in; i = (i + 1) % data.length)
              if(data[i] == null)
                  throw new CircularQueueException("null in CircularQueue");
                  // Guarantee that only null values are outside the
                  // region of 'data' that holds objects:
          if(full()) return true;
          for(int i = in; i != out; i = (i + 1) % data.length)
               if(data[i] != null)
                   throw new CircularQueueException(
                        "non-null outside of CircularQueue range: " + dump());
          return true;
      }
      
      public String dump() {
          return "in = " + in +
              ", out = " + out +
              ", full() = " + full() +
              ", empty() = " + empty() +
              ", CircularQueue = " + Arrays.asList(data);
      }
  }
```

**in** 計數器指示陣列中下一個入隊物件所在的位置。**out** 計數器指示下一個出隊物件來自何處。**wrapped** 的flag表示入隊和出隊指標順序是否變換, 為**false** 表示**in**在**out**之前,為**true**則順序相反。當**in**和 **out** 重合時，佇列為空(如果**wrapped**為 **false** )或滿(如果 **wrapped** 為 **true** )。

**put()** 和 **get()** 方法呼叫 **precondition()** ，**postcondition()**, 和 **invariant**()，這些都是在類中定義的私有方法。前置**precondition()** 和 **postcondition()** 是用來闡明程式碼的輔助方法。

注意，**precondition()** 返回 **void** , 因為它不與斷言一起使用。按照之前所說的，通常你會在程式碼中保留前置條件。透過將它們封裝在 **precondition()**  方法呼叫中，如果你不得不做出關掉它們的可怕舉動，你會有更好的選擇。

**postcondition()** 和 **invariant()** 都返回一個布林值，因此可以在 **assert** 語句中使用它們。此外，如果出於性能考慮禁用斷言，則根本不存在方法呼叫。**invariant()** 對物件執行內部有效性檢查，如果你在每個方法呼叫的開始和結束都這樣做，這是一個花銷巨大的操作，就像 **Meyer** 建議的那樣。所以， 用程式碼清晰地表明是有幫助的，它幫助我除錯了實現。此外，如果你對程式碼實現做任何更改，那麼 **invariant()** 將確保你沒有破壞程式碼，將不變性測試從方法呼叫移到單元測試程式碼中是相當簡單的。如果你的單元測試是足夠的，那麼你應當對不變性保持一定的信心。

**dump()** 幫助方法返回一個包含所有資料的字串，而不是直接列印資料。這允許我們用這部分訊息做更多事。  

現在我們可以為類建立 JUnit 測試:

  ```java
  // validating/tests/CircularQueueTest.java
  package validating;
  import org.junit.jupiter.api.*;
  import static org.junit.jupiter.api.Assertions.*;
  public class CircularQueueTest {
      private CircularQueue queue = new CircularQueue(10);
      private int i = 0;
      
      @BeforeEach
      public void initialize() {
          while(i < 5) // Pre-load with some data
              queue.put(Integer.toString(i++));
  	  }
      
      // Support methods:
      private void showFullness() {
          assertTrue(queue.full());
          assertFalse(queue.empty());
          System.out.println(queue.dump());
      }
      
      private void showEmptiness() {
          assertFalse(queue.full());
          assertTrue(queue.empty());
          System.out.println(queue.dump());
      }
      
      @Test
      public void full() {
          System.out.println("testFull");
          System.out.println(queue.dump());
          System.out.println(queue.get());
          System.out.println(queue.get());
          while(!queue.full())
              queue.put(Integer.toString(i++));
              String msg = "";
          try {
          	  queue.put("");
          } catch(CircularQueueException e) {
          	  msg = e.getMessage();
          	  ∂System.out.println(msg);
          }
          assertEquals(msg, "put() into full CircularQueue");
          showFullness();
      }
      
      @Test
      public void empty() {
          System.out.println("testEmpty");
          while(!queue.empty())
      		  System.out.println(queue.get());
      		  String msg = "";
          try {
              queue.get();
          } catch(CircularQueueException e) {
              msg = e.getMessage();
              System.out.println(msg);
          }
          assertEquals(msg, "get() from empty CircularQueue");
          showEmptiness();
      }
      @Test
      public void nullPut() {
          System.out.println("testNullPut");
          String msg = "";
          try {
          	  queue.put(null);
          } catch(CircularQueueException e) {
              msg = e.getMessage();
              System.out.println(msg);
          }
          assertEquals(msg, "put() null item");
      }
      
      @Test
      public void circularity() {
      	  System.out.println("testCircularity");
      	  while(!queue.full())
      		  queue.put(Integer.toString(i++));
      		  showFullness();
      		  assertTrue(queue.isWrapped());
          
          while(!queue.empty())
          	  System.out.println(queue.get());
          	  showEmptiness();
          
          while(!queue.full())
          	  queue.put(Integer.toString(i++));
          	  showFullness();
          
          while(!queue.empty())
          	  System.out.println(queue.get());
          	  showEmptiness();
       }
   }
  /* Output:
  testNullPut
  put() null item
  testCircularity
  in = 0, out = 0, full() = true, empty() = false,
  CircularQueue =
  [0, 1, 2, 3, 4, 5, 6, 7, 8, 9]
  0
  1
  2
  3
  4
  5
  6
  7
  8
  9
  in = 0, out = 0, full() = false, empty() = true,
  CircularQueue =
  [null, null, null, null, null, null, null, null, null,
  null]
  in = 0, out = 0, full() = true, empty() = false,
  CircularQueue =
  [10, 11, 12, 13, 14, 15, 16, 17, 18, 19]
  10
  11
  12
  13
  14
  15
  16
  17
  18
  19
  in = 0, out = 0, full() = false, empty() = true,
  CircularQueue =
  [null, null, null, null, null, null, null, null, null,
  null]
  testFull
  in = 5, out = 0, full() = false, empty() = false,
  CircularQueue =
  [0, 1, 2, 3, 4, null, null, null, null, null]
  0
  1
  put() into full CircularQueue
  in = 2, out = 2, full() = true, empty() = false,
  CircularQueue =
  [10, 11, 2, 3, 4, 5, 6, 7, 8, 9]
  testEmpty
  0
  1
  2
  3
  4
  get() from empty CircularQueue
  in = 5, out = 5, full() = false, empty() = true,
  CircularQueue =
  [null, null, null, null, null, null, null, null, null,
  null]
  */
  ```

**initialize()** 添加了一些資料，因此每個測試的 **CircularQueue** 都是部分滿的。**showFullness()** 和 **showempty()** 表明 **CircularQueue** 是滿的還是空的，這四種測試方法中的每一種都確保了 **CircularQueue** 功能在不同的地方正確執行。

透過將 Dbc 和單元測試結合起來，你不僅可以同時使用這兩種方法，還可以有一個遷移路徑—你可以將一些 Dbc 測試遷移到單元測試中，而不是簡單地禁用它們，這樣你仍然有一定程度的測試。

#### 使用Guava前置條件

在非嚴格的 DbC 中，前置條件是 DbC 中你不想刪除的那一部分，因為它可以檢查方法參數的有效性。那是你沒有辦法控制的事情，所以你需要對其檢查。因為 Java 在預設情況下禁用斷言，所以通常最好使用另外一個始終驗證方法參數的庫。

Google的 Guava 庫包含了一組很好的前置條件測試，這些測試不僅易於使用，而且命名也足夠好。在這裡你可以看到它們的簡單用法。庫的設計人員建議靜態匯入前置條件:

```java
// validating/GuavaPreconditions.java
// Demonstrating Guava Preconditions
import java.util.function.*;
import static com.google.common.base.Preconditions.*;
public class GuavaPreconditions {
    static void test(Consumer<String> c, String s) {
        try {
            System.out.println(s);
            c.accept(s);
            System.out.println("Success");
        } catch(Exception e) {
            String type = e.getClass().getSimpleName();
            String msg = e.getMessage();
            System.out.println(type +
            (msg == null ? "" : ": " + msg));
        }
    }
    
    public static void main(String[] args) {
        test(s -> s = checkNotNull(s), "X");
        test(s -> s = checkNotNull(s), null);
        test(s -> s = checkNotNull(s, "s was null"), null);
        test(s -> s = checkNotNull(
        s, "s was null, %s %s", "arg2", "arg3"), null);
        test(s -> checkArgument(s == "Fozzie"), "Fozzie");
        test(s -> checkArgument(s == "Fozzie"), "X");
        test(s -> checkArgument(s == "Fozzie"), null);
        test(s -> checkArgument(
        s == "Fozzie", "Bear Left!"), null);
        test(s -> checkArgument(
        s == "Fozzie", "Bear Left! %s Right!", "Frog"),
        null);
        test(s -> checkState(s.length() > 6), "Mortimer");
        test(s -> checkState(s.length() > 6), "Mort");
        test(s -> checkState(s.length() > 6), null);
        test(s ->
        checkElementIndex(6, s.length()), "Robert");
        test(s ->
        checkElementIndex(6, s.length()), "Bob");
        test(s ->
        checkElementIndex(6, s.length()), null);
        test(s ->
        checkPositionIndex(6, s.length()), "Robert");
        test(s ->
        checkPositionIndex(6, s.length()), "Bob");
        test(s ->
        checkPositionIndex(6, s.length()), null);
        test(s -> checkPositionIndexes(
        0, 6, s.length()), "Hieronymus");
        test(s -> checkPositionIndexes(
        0, 10, s.length()), "Hieronymus");
        test(s -> checkPositionIndexes(
        0, 11, s.length()), "Hieronymus");
        test(s -> checkPositionIndexes(
        -1, 6, s.length()), "Hieronymus");
        test(s -> checkPositionIndexes(
        7, 6, s.length()), "Hieronymus");
        test(s -> checkPositionIndexes(
        0, 6, s.length()), null);
    }
}
/* Output:
X
Success
null
NullPointerException
null
NullPointerException: s was null
null
NullPointerException: s was null, arg2 arg3
Fozzie
Success
X
IllegalArgumentException
null
IllegalArgumentException
null
IllegalArgumentException: Bear Left!
null
IllegalArgumentException: Bear Left! Frog Right!
Mortimer
Success
Mort
IllegalStateException
null
NullPointerException
Robert
IndexOutOfBoundsException: index (6) must be less than
size (6)
Bob
IndexOutOfBoundsException: index (6) must be less than
size (3)
null
NullPointerException
Robert
Success
Bob
IndexOutOfBoundsException: index (6) must not be
greater than size (3)
null
NullPointerException
Hieronymus
Success
Hieronymus
Success
Hieronymus
IndexOutOfBoundsException: end index (11) must not be
greater than size (10)
Hieronymus
IndexOutOfBoundsException: start index (-1) must not be
negative
Hieronymus
IndexOutOfBoundsException: end index (6) must not be	
less than start index (7)
null
NullPointerException
*/
```

雖然 Guava 的前置條件適用於所有類型，但我這裡只示範 **字串（String）** 類型。**test()** 方法需要一個Consumer<String>，因此我們可以傳遞一個 lambda 表達式作為第一個參數，傳遞給 lambda 表達式的字串作為第二個參數。它顯示字串，以便在查看輸出時確定方向，然後將字串傳遞給 lambda 表達式。try 塊中的第二個 **println**() 僅在 lambda 表達式成功時才顯示; 否則 catch 塊將捕獲並顯示錯誤訊息。注意 **test()** 方法消除了多少重複的程式碼。

每個前置條件都有三種不同的重載形式：一個什麼都沒有，一個帶有簡單字串消息，以及帶有一個字串和取代值。為了提高效率，只允許 **%s** (字串類型)取代標記。在上面的例子中，示範了**checkNotNull()** 和 **checkArgument()** 這兩種形式。但是它們對於所有前置條件方法都是相同的。注意 **checkNotNull()** 的返回參數， 所以你可以在表達式中內聯使用它。下面是如何在建構子中使用它來防止包含 **Null** 值的物件構造：

```java
/ validating/NonNullConstruction.java
import static com.google.common.base.Preconditions.*;
public class NonNullConstruction {
    private Integer n;
    private String s;
    NonNullConstruction(Integer n, String s) {
        this.n = checkNotNull(n);	
        this.s = checkNotNull(s);
    }
    public static void main(String[] args) {
        NonNullConstruction nnc =
        new NonNullConstruction(3, "Trousers");
    }
}
```

**checkArgument()** 接受布爾表達式來對參數進行更具體的測試， 失敗時拋出  **IllegalArgumentException**，**checkState()** 用於測試物件的狀態（例如，不變性檢查），而不是檢查參數，並在失敗時拋出 **IllegalStateException** 。

最後三個方法在失敗時拋出 **IndexOutOfBoundsException**。**checkElementIndex**() 確保其第一個參數是列表、字串或陣列的有效元素索引，其大小由第二個參數指定。**checkPositionIndex()** 確保它的第一個參數在 0 到第二個參數(包括第二個參數)的範圍內。 **checkPositionIndexes()**  檢查 **[first_arg, second_arg]** 是一個列表的有效子列表，由第三個參數指定大小的字串或陣列。

所有的 Guava 前置條件對於基本類型和物件都有必要的重載。

<!-- Test-Driven Development -->

## 測試驅動開發

之所以可以有測試驅動開發（TDD）這種開發方式，是因為如果你在設計和編寫程式碼時考慮到了測試，那麼你不僅可以寫出可測試性更好的程式碼，而且還可以得到更好的程式碼設計。 一般情況下這個說法都是正確的。 一旦我想到“我將如何測試我的程式碼？”，這個想法將使我的程式碼產生變化，並且往往是從“可測試”轉變為“可用”。

純粹的 TDD 主義者會在實現新功能之前就為其編寫測試，這稱為測試優先的開發。 我們採用一個簡易的範例程式來進行說明，它的功能是反轉 **String** 中字元的大小寫。 讓我們隨意添加一些約束：**String** 必須小於或等於30個字元，並且必須只包含字母，空格，逗號和句號(英文)。

此範例與標準 TDD 不同，因為它的作用在於接收 **StringInverter** 的不同實現，以便在我們逐步滿足測試的過程中來體現類的演變。 為了滿足這個要求，將 **StringInverter** 作為介面：

```java
// validating/StringInverter.java
package validating;

interface StringInverter {
	String invert(String str);
}
```

現在我們透過可以編寫測試來表述我們的要求。 以下所述通常不是你編寫測試的方式，但由於我們在此處有一個特殊的約束：我們要對 **StringInverter **多個版本的實現進行測試，為此，我們利用了 JUnit5 中最複雜的新功能之一：動態測試生成。 顧名思義，透過它你可以使你所編寫的程式碼在執行時生成測試，而不需要你對每個測試顯式編碼。 這帶來了許多新的可能性，特別是在明確地需要編寫一整套測試而令人望而卻步的情況下。

JUnit5 提供了幾種動態生成測試的方法，但這裡使用的方法可能是最複雜的。  **DynamicTest.stream() **方法採用了：

- 物件集合上的疊代器 (versions) ，這個疊代器在不同組的測試中是不同的。 疊代器生成的物件可以是任何類型，但是只能有一種物件生成，因此對於存在多個不同的物件類型時，必須人為地將它們打包成單個類型。
- **Function**，它從疊代器獲取物件並生成描述測試的 **String** 。
- **Consumer**，它從疊代器獲取物件並包含基於該物件的測試程式碼。

在此範例中，所有程式碼將在 **testVersions()**  中進行組合以防止程式碼重複。 疊代器生成的物件是對 **DynamicTest** 的不同實現，這些物件體現了對介面不同版本的實現：

```java
// validating/tests/DynamicStringInverterTests.java
package validating;
import java.util.*;
import java.util.function.*;
import java.util.stream.*;
import org.junit.jupiter.api.*;
import static org.junit.jupiter.api.Assertions.*;
import static org.junit.jupiter.api.DynamicTest.*;

class DynamicStringInverterTests {
	// Combine operations to prevent code duplication:
	Stream<DynamicTest> testVersions(String id,
		Function<StringInverter, String> test) {
		List<StringInverter> versions = Arrays.asList(
			new Inverter1(), new Inverter2(),
			new Inverter3(), new Inverter4());
		return DynamicTest.stream(
			versions.iterator(),
			inverter -> inverter.getClass().getSimpleName(),
			inverter -> {
				System.out.println(
					inverter.getClass().getSimpleName() +
						": " + id);
				try {
					if(test.apply(inverter) != "fail")
						System.out.println("Success");
				} catch(Exception | Error e) {
					System.out.println(
						"Exception: " + e.getMessage());
				}
			}
		);
	}
    String isEqual(String lval, String rval) {
		if(lval.equals(rval))
			return "success";
		System.out.println("FAIL: " + lval + " != " + rval);
		return "fail";
	}
    @BeforeAll
	static void startMsg() {
		System.out.println(
			">>> Starting DynamicStringInverterTests <<<");
	}
    @AfterAll
	static void endMsg() {
		System.out.println(
			">>> Finished DynamicStringInverterTests <<<");
	}
	@TestFactory
	Stream<DynamicTest> basicInversion1() {
		String in = "Exit, Pursued by a Bear.";
		String out = "eXIT, pURSUED BY A bEAR.";
		return testVersions(
			"Basic inversion (should succeed)",
			inverter -> isEqual(inverter.invert(in), out)
		);
	}
	@TestFactory
	Stream<DynamicTest> basicInversion2() {
		return testVersions(
			"Basic inversion (should fail)",
			inverter -> isEqual(inverter.invert("X"), "X"));
	}
	@TestFactory
	Stream<DynamicTest> disallowedCharacters() {
		String disallowed = ";-_()*&^%$#@!~`0123456789";
		return testVersions(
			"Disallowed characters",
			inverter -> {
				String result = disallowed.chars()
					.mapToObj(c -> {
						String cc = Character.toString((char)c);
						try {
							inverter.invert(cc);
							return "";
						} catch(RuntimeException e) {
							return cc;
						}
					}).collect(Collectors.joining(""));
				if(result.length() == 0)
					return "success";
				System.out.println("Bad characters: " + result);
				return "fail";
			}
		);
	}
    @TestFactory
	Stream<DynamicTest> allowedCharacters() {
		String lowcase = "abcdefghijklmnopqrstuvwxyz ,.";
		String upcase = "ABCDEFGHIJKLMNOPQRSTUVWXYZ ,.";
		return testVersions(
			"Allowed characters (should succeed)",
            inverter -> {
				assertEquals(inverter.invert(lowcase), upcase);
				assertEquals(inverter.invert(upcase), lowcase);
				return "success";
			}
		);
	}
	@TestFactory
	Stream<DynamicTest> lengthNoGreaterThan30() {
		String str = "xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx";
		assertTrue(str.length() > 30);
		return testVersions(
			"Length must be less than 31 (throws exception)",
			inverter -> inverter.invert(str)
		);
	}
	@TestFactory
	Stream<DynamicTest> lengthLessThan31() {
		String str = "xxxxxxxxxxxxxxxxxxxxxxxxxxxxxx";
		assertTrue(str.length() < 31);
		return testVersions(
			"Length must be less than 31 (should succeed)",
			inverter -> inverter.invert(str)
		);
	}
}
```

在一般的測試中，你可能認為在進行一個結果為失敗的測試時應該停止程式碼構建。 但是在這裡，我們只希望系統報告問題，但仍然繼續執行，以便你可以看到不同版本的 **StringInverter** 的效果。

每個使用 **@TestFactory** 注釋的方法都會生成一個 **DynamicTest** 物件的 **Stream**（通過 **testVersions()** ），每個 JUnit 都像一般的 **@Test** 方法一樣執行。

現在測試都已經準備好了，我們就可以開始實現 **StringInverter **了。 我們從一個僅返回其參數的假的實現類開始：

```java
// validating/Inverter1.java
package validating;
public class Inverter1 implements StringInverter {
	public String invert(String str) { return str; }
}
```

接下來我們實現反轉操作：

```java
// validating/Inverter2.java
package validating;
import static java.lang.Character.*;
public class Inverter2 implements StringInverter {
	public String invert(String str) {
		String result = "";
		for(int i = 0; i < str.length(); i++) {
			char c = str.charAt(i);
			result += isUpperCase(c) ?
					  toLowerCase(c) :
					  toUpperCase(c);
		}
		return result;
	}
}
```

現在添加程式碼以確保輸入不超過30個字元：

```java
// validating/Inverter3.java
package validating;
import static java.lang.Character.*;
public class Inverter3 implements StringInverter {
	public String invert(String str) {
		if(str.length() > 30)
			throw new RuntimeException("argument too long!");
		String result = "";
		for(int i = 0; i < str.length(); i++) {
			char c = str.charAt(i);
			result += isUpperCase(c) ?
					  toLowerCase(c) :
					  toUpperCase(c);
        }
		return result;
	}
}
```

最後，我們排除了不允許的字元：

```java
// validating/Inverter4.java
package validating;
import static java.lang.Character.*;
public class Inverter4 implements StringInverter {
	static final String ALLOWED =
		"abcdefghijklmnopqrstuvwxyz ,." +
		"ABCDEFGHIJKLMNOPQRSTUVWXYZ";
	public String invert(String str) {
		if(str.length() > 30)
			throw new RuntimeException("argument too long!");
		String result = "";
		for(int i = 0; i < str.length(); i++) {
			char c = str.charAt(i);
			if(ALLOWED.indexOf(c) == -1)
				throw new RuntimeException(c + " Not allowed");
			result += isUpperCase(c) ?
					  toLowerCase(c) :
					  toUpperCase(c);
		}
		return result;
	}
}
```

你將從測試輸出中看到，每個版本的 **Inverter** 都幾乎能透過所有測試。 當你在進行測試優先的開發時會有相同的體驗。

**DynamicStringInverterTests.java** 僅是為了顯示 TDD 過程中不同 **StringInverter** 實現的開發。 通常，你只需編寫一組如下所示的測試，並修改單個 **StringInverter** 類直到它滿足所有測試：

```java
// validating/tests/StringInverterTests.java
package validating;
import java.util.*;
import java.util.stream.*;
import org.junit.jupiter.api.*;
import static org.junit.jupiter.api.Assertions.*;

public class StringInverterTests {
	StringInverter inverter = new Inverter4();
	@BeforeAll
	static void startMsg() {
		System.out.println(">>> StringInverterTests <<<");
	}
    @Test
	void basicInversion1() {
		String in = "Exit, Pursued by a Bear.";
		String out = "eXIT, pURSUED BY A bEAR.";
		assertEquals(inverter.invert(in), out);
	}
	@Test
	void basicInversion2() {
		expectThrows(Error.class, () -> {
			assertEquals(inverter.invert("X"), "X");
		});
	}
	@Test
	void disallowedCharacters() {
		String disallowed = ";-_()*&^%$#@!~`0123456789";
		String result = disallowed.chars()
			.mapToObj(c -> {
				String cc = Character.toString((char)c);
				try {
					inverter.invert(cc);
					return "";
				} catch(RuntimeException e) {
					return cc;
				}
			}).collect(Collectors.joining(""));
		assertEquals(result, disallowed);
	}
	@Test
	void allowedCharacters() {
		String lowcase = "abcdefghijklmnopqrstuvwxyz ,.";
		String upcase = "ABCDEFGHIJKLMNOPQRSTUVWXYZ ,.";
		assertEquals(inverter.invert(lowcase), upcase);
		assertEquals(inverter.invert(upcase), lowcase);
	}
	@Test
	void lengthNoGreaterThan30() {
		String str = "xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx";
		assertTrue(str.length() > 30);
		expectThrows(RuntimeException.class, () -> {
			inverter.invert(str);
		});
	}
    @Test
	void lengthLessThan31() {
		String str = "xxxxxxxxxxxxxxxxxxxxxxxxxxxxxx";
		assertTrue(str.length() < 31);
		inverter.invert(str);
	}
}
```

你可以透過這種方式進行開發：一開始在測試中建立你期望程式應有的所有特性，然後你就能在實現中一步步添加功能，直到所有測試通過。 完成後，你還可以在將來透過這些測試來得知（或讓其他任何人得知）當修復錯誤或添加功能時，程式碼是否被破壞了。 TDD的目標是產生更好，更周全的測試，因為在完全實現之後嘗試實現完整的測試覆蓋通常會產生匆忙或無意義的測試。

### 測試驅動 vs. 測試優先

雖然我自己還沒有達到測試優先的意識水平，但我最感興趣的是來自測試優先中的“測試失敗的書籤”這一概念。 當你離開你的工作一段時間後，重新回到工作進展中，甚至找到你離開時工作到的地方有時會很有挑戰性。 然而，以失敗的測試為書籤能讓你找到之前停止的地方。 這似乎讓你能更輕鬆地暫時離開你的工作，因為不用擔心找不到工作進展的位置。

純粹的測試優先編程的主要問題是它假設你事先了解了你正在解決的問題。 根據我自己的經驗，我通常是從實驗開始，而只有當我處理問題一段時間後，我對它的理解才會達到能給它編寫測試的程度。 當然，偶爾會有一些問題在你開始之前就已經完全定義，但我個人並不常遇到這些問題。 實際上，可能用“*面向測試的開發* ( *Test-Oriented Development* )”這個短語來描述編寫測試良好的程式碼或許更好。

<!-- Logging -->

## 日誌

### 日誌會給出正在執行的程式的各種訊息。

在除錯程式中，日誌可以是普通狀態資料，用於顯示程式執行過程（例如，安裝程式可能會記錄安裝過程中採取的步驟，儲存文件的目錄，程式的啟動值等）。

在除錯期間，日誌也能帶來好處。 如果沒有日誌，你可能會嘗試通過插入 **println()** 語句來列印出程式的行為。 本書中的一些例子使用了這種技術，並且在沒有除錯器的情況下（下文中很快就會介紹這樣一個主題），它就是你唯一的工具。 但是，一旦你確定程式正常執行，你可能會將 **println()** 語句注釋或者刪除。 然而，如果你遇到更多錯誤，你可能又需要執行它們。因此，如果能夠只在需要時輕鬆啟用輸出程式狀態就好多了。

程式設計師在日誌包可供使用之前，都只能依賴 Java 編譯器移除未呼叫的程式碼。 如果 **debug** 是一個 **static final boolean **，你就可以這麼寫：

```java
if(debug) {
	System.out.println("Debug info");
}
```

然後，當 **debug **為 **false **時，編譯器將移除大括號內的程式碼。 因此，未呼叫的程式碼不會對執行時產生影響。 使用這種方法，你可以在整個程式中放置跟蹤程式碼，並輕鬆啟用和關閉它。 但是，該技術的一個缺點是你必須重新編譯程式碼才能啟用和關閉跟蹤語句。因此，透過更改配置檔案來修改日誌屬性，從而起到啟用跟蹤語句但不用重新編譯程式會更方便。

業內普遍認為標準 Java 發行版本中的日誌包 **(java.util.logging)** 的設計相當糟糕。 大多數人會選擇其他的替代日誌包。如 *Simple Logging Facade for Java(SLF4J)* ,它為多個日誌框架提供了一個封裝好的呼叫方式，這些日誌框架包括 **java.util.logging** ， **logback** 和 **log4j **。 SLF4J 允許使用者在部署時插入所需的日誌框架。

SLF4J 提供了一個複雜的工具來報告程式的訊息，它的效率與前面範例中的技術幾乎相同。 對於非常簡單的訊息日誌記錄，你可以執行以下操作：

```java
// validating/SLF4JLogging.java
import org.slf4j.*;
public class SLF4JLogging {
	private static Logger log =
		LoggerFactory.getLogger(SLF4JLogging.class);
	public static void main(String[] args) {
		log.info("hello logging");
	}
}
/* Output:
2017-05-09T06:07:53.418
[main] INFO SLF4JLogging - hello logging
*/
```

日誌輸出中的格式和訊息，甚至輸出是否正常或“錯誤”都取決於 SLF4J 所連接的後端程式包是怎樣實現的。 在上面的範例中，它連接到的是 **logback** 庫（透過本書的 **build.gradle** 文件），並顯示為標準輸出。

如果我們修改 **build.gradle** 從而使用內建在 JDK 中的日誌包作為後端，則輸出顯示為錯誤輸出，如下所示：

**Aug 16, 2016 5:40:31 PM InfoLogging main**
**INFO: hello logging**

日誌系統會檢測日誌消息處所在的類名和方法名。 但它不能保證這些名稱是正確的，所以不要糾結於其準確性。

### 日誌等級

SLF4J 提供了多個等級的日誌消息。下面這個例子以“嚴重性”的遞增順序對它們作出示範：

```java
// validating/SLF4JLevels.java
import org.slf4j.*;
public class SLF4JLevels {
	private static Logger log =
		LoggerFactory.getLogger(SLF4JLevels.class);
	public static void main(String[] args) {
		log.trace("Hello");
		log.debug("Logging");
		log.info("Using");
		log.warn("the SLF4J");
		log.error("Facade");
	}
}
/* Output:
2017-05-09T06:07:52.846
[main] TRACE SLF4JLevels - Hello
2017-05-09T06:07:52.849
[main] DEBUG SLF4JLevels - Logging
2017-05-09T06:07:52.849
[main] INFO SLF4JLevels - Using
2017-05-09T06:07:52.850
[main] WARN SLF4JLevels - the SLF4J
2017-05-09T06:07:52.851
[main] ERROR SLF4JLevels - Facade
*/
```

你可以按等級來尋找消息。 級別通常設定在單獨的配置檔案中，因此你可以重新配置而無需重新編譯。 配置檔案格式取決於你使用的後端日誌包實現。 如 **logback** 使用 XML ：

```xml
<!-- validating/logback.xml -->
<?xml version="1.0" encoding="UTF-8"?>
<configuration>
	<appender name="STDOUT"
		class="ch.qos.logback.core.ConsoleAppender">
		<encoder>
			<pattern>
%d{yyyy-MM-dd'T'HH:mm:ss.SSS}
[%thread] %-5level %logger - %msg%n
			</pattern>
		</encoder>
	</appender>
	<root level="TRACE">
		<appender-ref ref="STDOUT" />
	</root>
</configuration>
```

你可以嘗試將 **<root level =“TRACE"> **行更改為其他級別，然後重新執行該程式查看日誌輸出的更改情況。 如果你沒有寫 **logback.xml** 文件，日誌系統將採取預設配置。

這只是 SLF4J 最簡單的介紹和一般的日誌消息，但也足以作為使用日誌的基礎 - 你可以沿著這個進行更長久的學習和實踐。你可以查閱 [SLF4J 文件](http://www.slf4j.org/manual.html)來獲得更深入的訊息。

<!-- Debugging -->

## 除錯

儘管聰明地使用 **System.out** 或日誌訊息能給我們帶來對程式行為的有效見解，但對於困難問題來說，這種方式就顯得笨拙且耗時了。

你也可能需要更加深入地理解程式，僅依靠列印日誌做不到。此時你需要除錯器。除了比列印語句更快更輕易地展示訊息以外，除錯器還可以設定斷點，並在程式執行到這些斷點處暫停程式。

使用除錯器，可以展示任何時刻的程式狀態，查看變數的值，一步一步執行程式，連接遠端執行的程式等等。特別是當你構建較大規模的系統（bug 容易被掩埋）時，熟練使用除錯器是值得的。

### 使用 JDB 除錯

Java 除錯器（JDB）是 JDK 內建的命令列工具。從除錯的指令和命令列介面兩方面看的話，JDB 至少從概念上是 GNU 除錯器（GDB，受 Unix DB 的影響）的繼承者。JDB 對於學習除錯和執行簡單的除錯任務來說是有用的，而且知道只要安裝了 JDK 就可以使用 JDB 是有幫助的。然而，對於大型項目來說，你可能想要一個圖形化的除錯器，這在後面會描述。

假設你寫了如下程式：

```java
// validating/SimpleDebugging.java
// {ThrowsException}
public class SimpleDebugging {
    private static void foo1() {
        System.out.println("In foo1");
        foo2();
    }
    
    private static void foo2() {
        System.out.println("In foo2");
        foo3();
    }
    
    private static void foo3() {
        System.out.println("In foo3");
        int j = 1;
        j--;
        int i = 5 / j;
    }
    
    public static void main(String[] args) {
        foo1();
    }
}
/* Output
In foo1
In foo2
In foo3
__[Error Output]__
Exception in thread "main"
java.lang.ArithmeticException: /by zero 
at 
SimpleDebugging.foo3(SimpleDebugging.java:17)
at 
SimpleDebugging.foo2(SimpleDebugging.java:11)
at
SimpleDebugging.foo1(SimpleDebugging.java:7)
at
SimpleDebugging.main(SimpleDebugging.java:20)
```

首先看方法 `foo3()`，問題很明顯：除數是 0。但是假如這段程式碼被埋沒在大型程式中（像這裡的呼叫序列暗示的那樣）而且你不知道從哪裡開始尋找問題。結果呢，異常會給出足夠的訊息讓你定位問題。然而，假設事情更加複雜，你必須更加深入程式中來獲得比異常提供的更多的訊息。

為了執行 JDB，你需要在編譯 **SimpleDebugging.java** 時加上 **-g** 標記，從而告訴編譯器生成編譯訊息。然後使用如下指令開始除錯程式：

**jdb SimpleDebugging**

接著 JDB 就會執行，出現命令列提示。你可以輸入 **?** 查看可用的 JDB 指令。

這裡展示了如何使用互動式追蹤一個問題的除錯歷程：

**Initializing jdb...**

**> catch Exception**

`>` 表明 JDB 在等待輸入指令。指令 **catch Exception** 在任何拋出異常的地方設定斷點（然而，即使你不顯式地設定斷點，除錯器也會停止— JDB 中好像是預設在異常拋出處設定了異常）。接著命令列會給出如下響應：

**Deferring exception catch Exception.**

**It will be set after the class is loaded.**

繼續輸入：

**> run**

現在程式將執行到下個斷點處，在這個例子中就是異常發生的地方。下面是執行 **run** 指令的結果：

**run SimpleDebugging**

**Set uncaught java.lang.Throwable**

**Set deferred uncaught java.lang.Throwable**

**>**

**VM Started: In foo1**

**In foo2**

**In foo3**

**Exception occurred: java.lang.ArithmeticException**

**(uncaught)"thread=main",**

**SimpleDebugging.foo3(),line=16 bci=15**

**16 int i = 5 / j**

程式執行到第16行時發生異常，但是 JDB 在異常發生時就不復存在。除錯器還展示了是哪一行導致了異常。你可以使用 **list** 將導致程式終止的執行點列出來：

**main[1] list**

**12 private static void foo3() {**

**13 System.out.println("In foo3");**

**14 int j = 1;**

**15 j--;**

**16 => int i = 5 / j;**

**17 }**

**18 public static void main(String[] args) {**

**19 foo1();**

**20 }**

**21 }**

**/* Output:**

上述 `=>` 展示了程式將繼續執行的執行點。你可以使用指令 **cont**(continue) 繼續執行，但是會導致 JDB 在異常發生時退出並列印出堆疊軌跡訊息。

指令 **locals** 能轉儲所有的局部變數值：

**main[1] locals**

**Method arguments:**

**Local variables:**

**j = 0**

指令 **wherei** 列印進入目前執行緒的方法堆疊中的堆疊幀訊息：

**main[1] wherei**

**[1] SimpleDebugging.foo3(SimpleDebugging.java:16), pc =15**

**[2] SimpleDebugging.foo2(SimpleDebugging.java:10), pc = 8**

**[3] SimpleDebugging.foo1(SimpleDebugging.java:6), pc = 8**

**[4] SimpleDebugging.main(SimpleDebugging.java:19), pc = 10**

**wherei** 後的每一行代表一個方法呼叫和呼叫返回點（由程式計數器顯示數值）。這裡的呼叫序列是 **main()**, **foo1()**, **foo2()** 和 **foo3()**。

因為指令 **list** 展示了執行停止的地方，所以你通常有足夠的訊息得知發生了什麼並修復它。指令 **help** 將會告訴你更多關於 **jdb** 的用法，但是在花更多的時間學習它之前必須明白命令列除錯器往往需要花費更多的精力得到結果。使用 **jdb** 學習除錯的基礎部分，然後轉而學習圖形介面除錯器。

### 圖形化除錯器

使用類似 JDB 的命令列除錯器是不方便的。它需要顯式的指令去查看變數的狀態(**locals**, **dump**)，列出原始碼中的執行點(**list**)，尋找系統中的執行緒(**threads**)，設定斷點(**stop in**, **stop at**)等等。使用圖形化除錯器只需要點擊幾下，不需要使用顯式的指令就能使用這些特性，而且能查看被除錯程式的最新細節。

因此，儘管你可能一開始用 JDB 嘗試除錯，但是你將發現使用圖形化除錯器能更加高效、更快速地追蹤 bug。IBM 的 Eclipse，Oracle 的 NetBeans 和 JetBrains 的 IntelliJ 這些整合開發環境都含有面向  Java 語言的好用的圖形化除錯器。

<!-- Benchmarking -->

## 基準測試

> 我們應該忘掉微小的效率提升，說的就是這些 97% 的時間做的事：過早的最佳化是萬惡之源。
>
> ​                                                                                                                       —— Donald Knuth

如果你發現自己正在過早最佳化的滑坡上，你可能浪費了幾個月的時間(如果你雄心勃勃的話)。通常，一個簡單直接的編碼方法就足夠好了。如果你進行了不必要的最佳化，就會使你的程式碼變得無謂的複雜和難以理解。

基準測試意味著對程式碼或演算法片段進行計時看哪個跑得更快，與下一節的分析和最佳化截然相反，分析最佳化是觀察整個程式，找到程式中最耗時的部分。

可以簡單地對一個程式碼片段的執行計時嗎？在像 C 這樣直接的程式語言中，這個方法的確可行。在像 Java 這樣擁有複雜的執行時系統的程式語言中，基準測試變得更有挑戰性。為了生成可靠的資料，環境設定必須控制諸如 CPU 頻率，節能特性，其他執行在相同機器上的行程，最佳化器選項等等。

### 微基準測試

寫一個計時工具類從而比較不同程式碼塊的執行速度是具有吸引力的。看起來這會產生一些有用的資料。比如，這裡有一個簡單的 **Timer** 類，可以用以下兩種方式使用它：

1. 建立一個 **Timer** 物件，執行一些操作然後呼叫 **Timer** 的 **duration()** 方法產生以毫秒為單位的執行時間。
2. 向靜態的 **duration()** 方法中傳入 **Runnable**。任何符合 **Runnable** 介面的類都有一個函數式方法 **run()**，該方法沒有入參，且沒有返回。

```java
// onjava/Timer.java
package onjava;
import static java.util.concurrent.TimeUnit.*;

public class Timer {
    private long start = System.nanoTime();
    
    public long duration() {
        return NANOSECONDS.toMillis(System.nanoTime() - start);
    }
    
    public static long duration(Runnable test) {
        Timer timer = new Timer();
        test.run();
        return timer.duration();
    }
}
```

這是一個很直接的計時方式。難道我們不能只執行一些程式碼然後看它的執行時長嗎？

有許多因素會影響你的結果，即使是生成提示符也會造成計時的混亂。這裡舉一個看起來天真的例子，它使用了 標準的 Java **Arrays** 庫（後面會詳細介紹）：

```java
// validating/BadMicroBenchmark.java
// {ExcludeFromTravisCI}
import java.util.*;
import onjava.Timer;

public class BadMicroBenchmark {
    static final int SIZE = 250_000_000;
    
    public static void main(String[] args) {
        try { // For machines with insufficient memory
            long[] la = new long[SIZE];
            System.out.println("setAll: " + Timer.duration(() -> Arrays.setAll(la, n -> n)));
            System.out.println("parallelSetAll: " + Timer.duration(() -> Arrays.parallelSetAll(la, n -> n)));
        } catch (OutOfMemoryError e) {
            System.out.println("Insufficient memory");
            System.exit(0);
        }
    }
    
}
/* Output
setAll: 272
parallelSetAll: 301
```

**main()** 方法的主體包含在 **try** 語句塊中，因為一台機器用光記憶體後會導致構建停止。

對於一個長度為 250,000,000 的 **long** 型（僅僅差一點就會讓大部分機器記憶體溢位）陣列，我們比較了 **Arrays.setAll()** 和 **Arrays.parallelSetAll()** 的效能。這個並行的版本會嘗試使用多個處理器加快完成任務（儘管我在這一節談到了一些並行的概念，但是在 [並發編程](./24-Concurrent-Programming.md) 章節我們才會詳細討論這些 ）。然而非並行的版本似乎執行得更快，儘管在不同的機器上結果可能不同。

**BadMicroBenchmark.java** 中的每一步操作都是獨立的，但是如果你的操作依賴於同一資源，那麼並行版本執行的速度會驟降，因為不同的行程會競爭相同的那個資源。

```java
// validating/BadMicroBenchmark2.java
// Relying on a common resource

import java.util.*;
import onjava.Timer;

public class BadMicroBenchmark2 {
    static final int SIZE = 5_000_000;
    
    public static void main(String[] args) {
        long[] la = new long[SIZE];
        Random r = new Random();
        System.out.println("parallelSetAll: " + Timer.duration(() -> Arrays.parallelSetAll(la, n -> r.nextLong())));
        System.out.println("setAll: " + Timer.duration(() -> Arrays.setAll(la, n -> r.nextLong())));
        SplittableRandom sr = new SplittableRandom();
        System.out.println("parallelSetAll: " + Timer.duration(() -> Arrays.parallelSetAll(la, n -> sr.nextLong())));
        System.out.println("setAll: " + Timer.duration(() -> Arrays.setAll(la, n -> sr.nextLong())));
    }
}
/* Output
parallelSetAll: 1147
setAll: 174
parallelSetAll: 86
setAll: 39
```

**SplittableRandom** 是為並行演算法設計的，它當然看起來比普通的 **Random** 在 **parallelSetAll()** 中執行得更快。 但是看起來還是比非並發的 **setAll()** 執行時間更長，有點難以置信（也許是真的，但我們不能透過一個壞的微基準測試得到這個結論）。

這隻考慮了微基準測試的問題。Java 虛擬機 Hotspot 也非常影響性能。如果你在測試前沒有透過執行程式碼給 JVM 預熱，那麼你就會得到“冷”的結果，不能反映出程式碼在 JVM 預熱之後的執行速度（假如你執行的應用沒有在預熱的 JVM 上執行，你就可能得不到所預期的效能，甚至可能減緩速度）。

最佳化器有時可以檢測出你建立了沒有使用的東西，或者是部分程式碼的執行結果對程式沒有影響。如果它最佳化掉你的測試，那麼你可能得到不好的結果。

一個良好的微基準測試系統能自動地彌補像這樣的問題（和很多其他的問題）從而產生合理的結果，但是建立這麼一套系統是非常棘手，需要深入的知識。

### JMH 的引入

截止目前為止，唯一能產生像樣結果的 Java 微基準測試系統就是 Java Microbenchmarking Harness，簡稱 JMH。本書的 **build.gradle** 自動引入了 JMH 的設定，所以你可以輕鬆地使用它。

你可以在命令列編寫 JMH 程式碼並執行它，但是推薦的方式是讓 JMH 系統為你執行測試；**build.gradle** 文件已經配置成只需要一條指令就能執行 JMH 測試。

JMH 嘗試使基準測試變得儘可能簡單。例如，我們將使用 JMH 重新編寫 **BadMicroBenchmark.java**。這裡只有 **@State** 和 **@Benchmark** 這兩個註解是必要的。其餘的註解要嘛是為了產生更多易懂的輸出，要嘛是加快基準測試的執行速度（JMH 基準測試通常需要執行很長時間）：

```java
// validating/jmh/JMH1.java
package validating.jmh;
import java.util.*;
import org.openjdk.jmh.annotations.*;
import java.util.concurrent.TimeUnit;

@State(Scope.Thread)
@BenchmarkMode(Mode.AverageTime)
@OutputTimeUnit(TimeUnit.MICROSECONDS)
// Increase these three for more accuracy:
@Warmup(iterations = 5)
@Measurement(iterations = 5)
@Fork(1)
public class JMH1 {
    private long[] la;
    
    @Setup
    public void setup() {
        la = new long[250_000_000];
    }
    
    @Benchmark
    public void setAll() {
        Arrays.setAll(la, n -> n);
    }
    
    public void parallelSetAll() {
        Arrays.parallelSetAll(la, n -> n);
    }
}
```

“forks” 的預設值是 10，意味著每個測試都執行 10 次。為了減少執行時間，這裡使用了 **@Fork** 註解來減少這個次數到 1。我還使用了 **@Warmup** 和 **@Measurement** 註解將它們預設的執行次數從 20 減少到 5 次。儘管這降低了整體的準確率，但是結果幾乎與使用預設值相同。可以嘗試將 **@Warmup**、**@Measurement** 和 **@Fork** 都注釋掉然後看使用它們的預設值，結果會有多大顯著的差異；一般來說，你應該只能看到長期執行的測試使錯誤因素減少，而結果沒有多大變化。

需要使用顯式的 gradle 指令才能執行基準測試（在範例程式碼的根目錄處執行）。這能防止耗時的基準測試執行其他的 **gradlew** 指令：

**gradlew validating:jmh**

這會花費幾分鐘的時間，取決於你的機器(如果沒有註解上的調整，可能需要幾個小時)。控制台會顯示 **results.txt** 文件的路徑，這個文件統計了執行結果。注意，**results.txt** 包含這一章所有 **jmh** 測試的結果：**JMH1.java**，**JMH2.java** 和 **JMH3.java**。

因為輸出是絕對時間，所以在不同的機器和作業系統上結果各不相同。重要的因素不是絕對時間，我們真正觀察的是一個演算法和另一個演算法的比較，尤其是哪一個執行得更快，快多少。如果你在自己的機器上執行測試，你將看到不同的結果卻有著相同的模式。

我在大量的機器上執行了這些測試，儘管不同的機器上得到的絕對值結果不同，但是相對值保持著合理的穩定性。我只列出了 **results.txt** 中適當的片段並加以編輯使輸出更加易懂，而且內容大小適合頁面。所有測試中的 **Mode** 都以 **avgt** 展示，代表 “平均時長”。**Cnt**（測試的數目）的值是 200，儘管這裡的一個例子中配置的 **Cnt** 值是 5。**Units** 是 **us/op**，是 “Microseconds per operation” 的縮寫，因此，這個值越小代表性能越高。

我同樣也展示了使用 warmups、measurements 和 forks 預設值的輸出。我刪除了範例中相應的註解，就是為了獲取更加準確的測試結果（這將花費數小時）。結果中數字的模式應該仍然看起來相同，不論你如何執行測試。

下面是 **JMH1.java** 的執行結果：

**Benchmark Score**

**JMH1.setAll 196280.2**

**JMH1.parallelSetAll 195412.9**

即使像 JMH 這麼進階的基準測試工具，基準測試的過程也不容易，練習時需要倍加小心。這裡測試產生了反直覺的結果：並行的版本 **parallelSetAll()** 花費了與非並行版本的 **setAll()** 相同的時間，兩者似乎都執行了相當長的時間。

當建立這個範例時，我假設如果我們要測試陣列初始化的話，那麼使用非常大的陣列是有意義的。所以我選擇了儘可能大的陣列；如果你實驗的話會發現一旦陣列的大小超過 2億5000萬，你就開始會得到記憶體溢位的異常。然而，在這麼大的陣列上執行大量的操作從而震盪記憶體系統，產生無法預料的結果是有可能的。不管這個假設是否正確，看起來我們正在測試的並非是我們想測試的內容。

考慮其他的因素：

C：用戶端執行操作的執行緒數量

P：並行演算法使用的並行數量

N：陣列的大小：**10^(2*k)**，通常來說，**k=1..7** 足夠來練習不同的快取占用。

Q：setter 的操作成本

這個 C/P/N/Q 模型在早期 JDK 8 的 Lambda 開發期間浮出水面，大多數並行的 Stream 操作(**parallelSetAll()** 也基本相似)都滿足這些結論：**N*Q**(主要工作量)對於並發性能尤為重要。並行演算法在工作量較少時可能實際執行得更慢。

在一些情況下操作競爭如此激烈使得並行毫無幫助，而不管 **N*Q** 有多大。當 **C** 很大時，**P** 就變得不太相關（內部並行在大量的外部並行面前顯得多餘）。此外，在一些情況下，並行分解會讓相同的 **C** 個用戶端執行得比它們順序執行程式碼更慢。

基於這些訊息，我們重新執行測試，並在這些測試中使用不同大小的陣列（改變 **N**）：

```java
// validating/jmh/JMH2.java
package validating.jmh;
import java.util.*;
import org.openjdk.jmh.annotations.*;
import java.util.concurrent.TimeUnit;

@State(Scope.Thread)
@BenchmarkMode(Mode.AverageTime)
@OutputTimeUnit(TimeUnit.MICROSECONDS)
@Warmup(iterations = 5)
@Measurement(iterations = 5)
@Fork(1)
public class JMH2 {

    private long[] la;

    @Param({
            "1",
            "10",
            "100",
            "1000",
            "10000",
            "100000",
            "1000000",
            "10000000",
            "100000000",
            "250000000"
    })
    int size;

    @Setup
    public void setup() {
        la = new long[size];
    }

    @Benchmark
    public void setAll() {
        Arrays.setAll(la, n -> n);
    }

    @Benchmark
    public void parallelSetAll() {
        Arrays.parallelSetAll(la, n -> n);
    }
}
```

**@Param** 會自動地將其自身的值注入到變數中。其自身的值必須是字串類型，並可以轉化為適當的類型，在這個例子中是 **int** 類型。

下面是已經編輯過的結果，包含精確計算出的加速數值：

| JMH2 Benchmark     | Size      | Score %    | Speedup |
| ------------------ | --------- | ---------- | ------- |
| **setAll**         | 1         | 0.001      |         |
| **parallelSetAll** | 1         | 0.036      | 0.028   |
| **setAll**         | 10        | 0.005      |         |
| **parallelSetAll** | 10        | 3.965      | 0.001   |
| **setAll**         | 100       | 0.031      |         |
| **parallelSetAll** | 100       | 3.145      | 0.010   |
| **setAll**         | 1000      | 0.302      |         |
| **parallelSetAll** | 1000      | 3.285      | 0.092   |
| **setAll**         | 10000     | 3.152      |         |
| **parallelSetAll** | 10000     | 9.669      | 0.326   |
| **setAll**         | 100000    | 34.971     |         |
| **parallelSetAll** | 100000    | 20.153     | 1.735   |
| **setAll**         | 1000000   | 420.581    |         |
| **parallelSetAll** | 1000000   | 165.388    | 2.543   |
| **setAll**         | 10000000  | 8160.054   |         |
| **parallelSetAll** | 10000000  | 7610.190   | 1.072   |
| **setAll**         | 100000000 | 79128.752  |         |
| **parallelSetAll** | 100000000 | 76734.671  | 1.031   |
| **setAll**         | 250000000 | 199552.121 |         |
| **parallelSetAll** | 250000000 | 191791.927 | 1.040   |
可以看到當陣列大小達到 10 萬左右時，**parallelSetAll()** 開始反超，而後趨於與非並行的執行速度相同。即使它執行速度上勝了，看起來也不足以證明由於並行的存在而使速度變快。

**setAll()/parallelSetAll()** 中工作的計算量起很大影響嗎？在前面的例子中，我們所做的只有對陣列的賦值操作，這可能是最簡單的任務。所以即使 **N** 值變大，**N*Q** 也仍然沒有達到巨大，所以看起來像是我們沒有為並行提供足夠的機會（JMH 提供了一種模擬變數 Q 的途徑；如果想了解更多的話，可搜尋 **Blackhole.consumeCPU**）。

我們透過使方法 **f()** 中的任務變得更加複雜，從而產生更多的並行機會：

```java
// validating/jmh/JMH3.java
package validating.jmh;
import java.util.*;
import org.openjdk.jmh.annotations.*;
import java.util.concurrent.TimeUnit;

@State(Scope.Thread)
@BenchmarkMode(Mode.AverageTime)
@OutputTimeUnit(TimeUnit.MICROSECONDS)
@Warmup(iterations = 5)
@Measurement(iterations = 5)
@Fork(1)
public class JMH3 {
    private long[] la;

    @Param({
            "1",
            "10",
            "100",
            "1000",
            "10000",
            "100000",
            "1000000",
            "10000000",
            "100000000",
            "250000000"
    })
    int size;

    @Setup
    public void setup() {
        la = new long[size];
    }

    public static long f(long x) {
        long quadratic = 42 * x * x + 19 * x + 47;
        return Long.divideUnsigned(quadratic, x + 1);
    }

    @Benchmark
    public void setAll() {
        Arrays.setAll(la, n -> f(n));
    }

    @Benchmark
    public void parallelSetAll() {
        Arrays.parallelSetAll(la, n -> f(n));
    }
}
```

**f()** 方法提供了更加複雜且耗時的操作。現在除了簡單的給陣列賦值外，**setAll()** 和 **parallelSetAll()** 都有更多的工作去做，這肯定會影響結果。

| JMH2 Benchmark     | Size      | Score %     | Speedup |
| ------------------ | --------- | ----------- | ------- |
| **setAll**         | 1         | 0.012       |         |
| **parallelSetAll** | 1         | 0.047       | 0.255   |
| **setAll**         | 10        | 0.107       |         |
| **parallelSetAll** | 10        | 3.894       | 0.027   |
| **setAll**         | 100       | 0.990       |         |
| **parallelSetAll** | 100       | 3.708       | 0.267   |
| **setAll**         | 1000      | 133.814     |         |
| **parallelSetAll** | 1000      | 11.747      | 11.391  |
| **setAll**         | 10000     | 97.954      |         |
| **parallelSetAll** | 10000     | 37.259      | 2.629   |
| **setAll**         | 100000    | 988.475     |         |
| **parallelSetAll** | 100000    | 276.264     | 3.578   |
| **setAll**         | 1000000   | 9203.103    |         |
| **parallelSetAll** | 1000000   | 2826.974    | 3.255   |
| **setAll**         | 10000000  | 92144.951   |         |
| **parallelSetAll** | 10000000  | 28126.202   | 3.276   |
| **setAll**         | 100000000 | 921701.863  |         |
| **parallelSetAll** | 100000000 | 266750.543  | 3.455   |
| **setAll**         | 250000000 | 2299127.273 |         |
| **parallelSetAll** | 250000000 | 538173.425  | 4.272   |

可以看到當陣列的大小達到 1000 左右時，**parallelSetAll()** 的執行速度反超了 **setAll()**。看來 **parallelSetAll()** 嚴重依賴陣列中計算的複雜度。這正是基準測試的價值所在，因為我們已經得到了關於 **setAll()** 和 **parallelSetAll()** 間微妙的訊息，知道在何時使用它們。

這顯然不是從閱讀 Javadocs 就能得到的。

大多數時候，JMH 的簡單應用會產生好的結果（正如你將在本書後面例子中所見），但是我們從這裡知道，你不能一直假定 JMH 會產生好的結果。 JMH 網站上的範例可以幫助你開始。

<!-- Profiling and Optimizing -->

## 剖析和最佳化

有時你必須檢測程式執行時間花在哪裡，從而看是否可以最佳化那一塊的效能。剖析器可以找到這些導致程式慢的地方，因而你可以找到最輕鬆，最明顯的方式加快程式執行速度。

剖析器收集的訊息能顯示程式哪一部分消耗記憶體，哪個方法最耗時。一些剖析器甚至能關閉垃圾回收，從而幫助限定記憶體分配的模式。

剖析器還可以幫助檢測程式中的執行緒死鎖。注意剖析和基準測試的區別。剖析關注的是已經執行在真實資料上的整個程式，而基準測試關注的是程式中隔離的片段，通常是去最佳化演算法。

安裝 Java 開發工具包（JDK）時會順便安裝一個虛擬的剖析器，叫做 **VisualVM**。它會被自動安裝在與 **javac** 相同的目錄下，你的執行路徑應該已經包含該目錄。啟動 VisualVM 的控制台指令是：

**> jvisualvm**

執行該指令後會彈出一個視窗，其中包括一些指向說明訊息的連結。

### 最佳化準則

- 避免為了性能犧牲程式碼的可讀性。
- 不要獨立地看待性能。衡量與帶來的收益相比所需投入的工作量。
- 程式的大小很重要。性能最佳化通常只對執行了長時間的大型項目有價值。性能通常不是小項目的關注點。
- 執行起來程式比一心鑽研它的效能具有更高的優先度。一旦你已經有了可工作的程式，如有必要的話，你可以使用剖析器提高它的效率。只有當性能是關鍵因素時，才需要在設計/開發階段考慮性能。
- 不要猜測瓶頸發生在哪。執行剖析器，讓剖析器告訴你。
- 無論何時有可能的話，顯式地設定實例為 null 表明你不再用它。這對垃圾收集器來說是個有用的暗示。
- **static final** 修飾的變數會被 JVM 最佳化從而提高程式的執行速度。因而程式中的常量應該聲明 **static final**。

<!-- Style Checking -->

## 風格檢測

當你在一個團隊中工作時(包括尤其是開源項目)，讓每個人遵循相同的程式碼風格是非常有幫助的。這樣閱讀項目的程式碼時，不會因為風格的不同產生思維上的中斷。然而，如果你習慣了某種不同的程式碼風格，那麼記住項目中所有的風格準則對你來說可能是困難的。幸運的是，存在可以指出你程式碼中不符合風格準則的工具。

一個流行的風格檢測器是 **Checkstyle**。查看本書 [範例程式碼](https://github.com/BruceEckel/OnJava8-Examples) 中的 **gradle.build** 和 **checkstyle.xml** 文件中配置程式碼風格的方式。checkstyle.xml 是一個常用檢測的集合，其中一些檢測被注釋掉了以允許使用本書中的程式碼風格。

執行所有風格檢測的指令是：

**gradlew checkstyleMain**

一些文件仍然產生了風格檢測警告，通常是因為這些例子展示了你在生產程式碼中不會使用的樣例。

你還可以針對一個具體的章節執行程式碼檢測。例如，下面指令會執行 [Annotations](./23-Annotations.md) 章節的風格檢測：

**gradlew annotations:checkstyleMain**

<!-- Static Error Analysis -->

## 靜態錯誤分析

儘管 Java 的靜態類型檢測可以發現基本的語法錯誤，其他的分析工具可以發現躲避 **javac** 檢測的更加複雜的bug。一個這樣的工具叫做 **Findbugs**。本書 [範例程式碼](https://github.com/BruceEckel/OnJava8-Examples) 中的 **build.gradle** 文件包含了 Findbugs 的配置，所以你可以輸入如下指令：

**gradlew findbugsMain**

這會為每一章生成一個名為 **main.html** 的報告，報告中會說明程式碼中潛在的問題。Gradle 指令的輸出會告訴你每個報告在何處。

當你查看報告時，你將會看到很多 false positive 的情況，即程式碼沒問題卻報告了問題。我在一些文件中展示了不要做一些事的程式碼確實是正確的。

當我最初看到本書的 Findbugs 報告時，我發現了一些不是技術錯誤的地方，但能使我改善程式碼。如果你正在尋找 bug，那麼在除錯之前執行 Findbugs 是值得的，因為這將可能節省你數小時的時間找到問題。

<!-- Code Reviews -->

## 程式碼重審

單元測試能找到明顯重要的 bug 類型，風格檢測和 Findbugs 能自動執行程式碼重審，從而發現額外的問題。最終你走到了必須人為參與進來的地步。程式碼重審是一個或一群人的一段程式碼被另一個或一群人閱讀和評估的眾多方式之一。這最初看起來會使人不安，而且需要情感信任，但它的目的肯定不是羞辱任何人。它的目標是找到程式中的錯誤，程式碼重審是最成功的能做到這點的途徑之一。可惜的是，它們也經常被認為是“過於昂貴的”（有時這會成為程式設計師避免程式碼被重審時感到尷尬的藉口）。

程式碼重審可以作為結對編程的一部分，作為程式碼簽入過程的一部分（另一個程式設計師自動安排上審查新程式碼的任務）或使用群組預排的方式，即每個人閱讀程式碼並討論之。後一種方式對於分享知識和營造程式碼文化是極其有益的。

<!-- Pair Programming -->

## 結對編程

結對編程是指兩個程式設計師一起編程的實踐活動。通常來說，一個人“驅動”（敲擊鍵盤，輸入代碼），另一人（觀察者或指引者）重審和分析程式碼，同時也要思考策略。這產生了一種即時的程式碼重審。通常程式設計師會定期地互換角色。

結對編程有很多好處，但最顯著的是分享知識和防止阻塞。最佳傳遞訊息的方式之一就是一起解決問題，我已經在很多次研討會使用了結對編程，都取得了很好的效果（同時，研討會上的眾人可以透過這種方式互相了解對方）。而且兩個人一起工作時，可以更容易地推進開發的進展，而只有一個程式設計師的話，可能被輕易地卡住。結對編程的程式設計師通常可以從工作中感到更高的滿足感。有時很難向管理人員們推行結對編程，因為他們可能覺得兩個程式設計師解決同一個問題的效率比他們分開解決不同問題的效率低。儘管短期內是這樣，但是結對編程能帶來更高的程式碼質量；除了結對編程的其他益處，如果你眼光長遠的話，這會產生更高的生產力。

維基百科上這篇 [結對編程的文章](https://en.wikipedia.org/wiki/Pair_programming) 可以作為你深入了解結對編程的開始。

<!-- Refactoring -->

## 重構

技術負債是指疊代發展的軟體中為了應急而生的醜陋解決方案從而導致設計難以理解，程式碼難以閱讀的部分。特別是當你必須修改和增加新特性的時候，這會造成麻煩。

重構可以矯正技術負債。重構的關鍵是它能改善程式碼設計，結構和可讀性（因而減少程式碼負債），但是它不能改變程式碼的行為。

很難向管理人員推行重構：“我們將投入很多工作不是增加新的特性，當我們完成時，外界無感知變化。但是相信我們，事情會變得更加美好”。不幸的是，管理人員意識到重構的價值時都為時已晚了：當他們提出增加新的特性時，你不得不告訴他們做不到，因為程式碼基底已經埋藏了太多的問題，試圖增加新特性可能會使軟體崩潰，即使你能想出怎麼做。

### 重構基石

在開始重構程式碼之前，你需要有以下三個系統的支撐：

1. 測試（通常，JUnit 測試作為最小的根基），因此你能確保重構不會改變程式碼的行為。
2. 自動構建，因而你能輕鬆地構建程式碼，執行所有的測試。透過這種方式做些小修改並確保修改不會破壞任何事物是毫不費力的。本書使用的是 Gradle 構建系統，你可以在 [程式碼範例](https://github.com/BruceEckel/OnJava8-Examples) 的 **build.gradle** 文件中查看範例。
3. 版本控制，以便你能回退到可工作的程式碼版本，能夠一直記錄重構的每一步。

本書的程式碼託管在 [Github](https://github.com/BruceEckel/OnJava8-Examples) 上，使用的是 **git** 版本控制系統。

沒有這三個系統的支援，重構幾乎是不可能的。確實，沒有這些系統，起初維護和增加程式碼是一個巨大的挑戰。令人意外的是，有很多成功的公司竟然在沒有這三個系統的情況下在相當長的時間裡勉強過得去。然而，對於這樣的公司來說，在他們遇到嚴重的問題之前，這只是個時間問題。

維基百科上的 [重構文章](https://en.wikipedia.org/wiki/Code_refactoring) 提供了更多的細節。

<!-- Continuous Integration -->

## 持續整合

在軟體開發的早期，人們只能一次處理一步，所以他們堅信他們總是在經歷快樂之旅，每個開發階段無縫進入下一個。這種錯覺經常被稱為軟體開發中的“瀑布流模型”。很多人告訴我瀑布流是他們的選擇方法，好像這是一個選擇工具，而不僅是一廂情願。

在這片童話的土地上，每一步都按照指定的預計時間準時完美結束，然後下一步開始。當最後一步結束時，所有的部件都可以無縫地滑在一起，瞧，一個裝載產品誕生了！

當然，現實中沒有事能按計劃或預計時間運作。相信它應該，然後當它不能時更加相信，只會使整件事變得更糟。否認證據不會產生好的結果。

除此之外，產品本身經常也不是對客戶有價值的事物。有時一大堆的特性完全是浪費時間，因為創造出這些特性需求的人不是客戶而是其他人。

因為受流水工作線的思路影響，所以每個開發階段都有自己的團隊。上游團隊的延期傳遞到下游團隊，當到了需要進行測試和整合的時候，這些團隊被指望趕上預期時間，當他們必然做不到時，就認為他們是“差勁的團隊成員”。不可能的時間安排和負相關的結合產生了自實現的預期：只有最絕望的開發者才會樂意做這些工作。

另外，商學院培養出的管理人員仍然被訓練成只在已有的流程上做一些改動——這些流程都是基於工業時代製造業的想法上。注重培養創造力而不是墨守成規的商學院仍然很稀有。終於一些編程領域的人們再也忍受不了這種情況並開始進行實驗。最初一些實驗叫做“極限編程”，因為它們與工業時代的思想完全不同。隨著實驗展示的結果，這些思想開始看起來像是常識。這些實驗逐漸形成了如今顯而易見的觀點——儘管非常小——即把生產可運作的產品交到客戶手中，詢問他們 (A) 是否想要它 (B) 是否喜歡它工作的方式 (C) 還希望有什麼其他有用的功能特性。然後這些訊息回饋給開發，從而繼續產出一個新版本。版本不斷疊代，項目最終演變成為客戶帶來真正價值的事物。

這完全顛倒了瀑布流開發的方式。你停止假設你要處理產品測試和把部署"作為最後一步"這類的事情。相反，每件事從開始到結束必須都在進行——即使一開始產品幾乎沒有任何特性。這麼做對於在開發週期的早期發現更多問題有巨大的益處。此外，不是做大量宏大超前的計劃和花費時間金錢在許多無用的特性上，而是一直都能從顧客那得到回饋。當客戶不再需要其他特性時，你就完成了。這節省了大量的時間和金錢，並提高了顧客的滿意度。

有許多不同的想法導向這種方式，但是目前首要的術語叫持續整合（CI）。CI 與導向 CI 的想法之間的不同之處在於 CI 是一種獨特的機械式的過程，過程中涵蓋了這些想法；它是一種定義好的做事方式。事實上，它定義得如此明確以至於整個過程是自動化的。

目前 CI 技術的高峰是持續整合伺服器。這是一台獨立的機器或虛擬機，通常是由第三方公司託管的完全獨立的服務。這些公司通常免費提供基本服務，如果你需要額外的特性如更多的處理器或記憶體或者專門的工具或系統，你需要付費。CI 伺服器起初是完全空白狀態，即只是可用的作業系統的最小配置。這很重要因為你可能之前在你的開發機器上安裝過一些程式，卻沒有在你的構建和部署系統中包含它。正如重構一樣，持續整合需要分布式版本管理，自動構建和自動測試系統作為基礎。通常來說，CI 伺服器會綁定到你的版本控制倉庫上。當 CI 伺服器發現倉庫中有改變時，就會拉取最新版本的程式碼，並按照 CI 腳本中的過程處理。這包括安裝所有必要的工具和類庫（記住，CI 伺服器起初只有一個乾淨、基本的作業系統），所以如果過程中出現任何問題，你都可以發現它們。接著它會執行腳本中定義的構建和測試操作；通常腳本中使用的指令與人們在安裝和測試中使用的指令完全相同。如果執行成功或失敗，CI 伺服器會有許多種方式匯報給你，包括在你的程式碼倉庫上顯示一個簡單的標記。

使用持續整合，每次你合進倉庫時，這些改變都會被從頭到尾驗證。透過這種方式，一旦出現問題你能立即發現。甚至當你準備交付一個產品的新版本時，都不會有延遲或其他必要的額外步驟（在任何時刻都可以交付叫做持續交付）。

本書的範例程式碼都是在 Travis-CI(基於 Linux 的系統) 和 AppVeyor(Windows) 上自動測試的。你可以在 [Gihub倉庫](https://github.com/BruceEckel/OnJava8-Examples) 上的 Readme 看到透過/失敗的標記。

<!-- Summary -->

## 本章小結

"它在我的機器上正常工作了。" "我們不會運載你的機器！"

程式碼校驗不是單一的過程或技術。每種方法只能發現特定類型的 bug，作為程式設計師的你在開發過程中會明白每個額外的技術都能增加程式碼的可靠性和強健性。校驗不僅能在開發過程中，還能在為應用添加新功能的整個項目期間幫你發現更多的錯誤。現代化開發意味著比僅僅編寫程式碼更多的內容，每種你在開發過程中融入的測試技術—— 包括而且尤其是你建立的能適應特定應用的自訂工具——都會帶來更好、更快和更加愉悅的開發過程，同時也能為客戶提供更高的價值和滿意度體驗。

<!-- 分頁 -->
