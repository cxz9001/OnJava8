[TOC]

<!-- Appendix: Javadoc -->
# 附錄:文件注釋

編寫程式碼文件的最大問題可能是維護該文件。如果文件和程式碼是分開的，每次更改程式碼時都要很繁瑣地再去更改文件。解決方案似乎很簡單：將程式碼連結到文件。最簡單的方法是將所有內容放在同一個文件中。然而，要完成這個任務，需要一個特殊的注釋語法來標記文件，以及一個工具將這些注釋提取為有用的形式，這就是Java所做的。

提取注釋的工具稱為Javadoc，它是 JDK 安裝的一部分。它使用Java編譯器中的一些技術來尋找特殊的注釋標記。它不僅提取由這些標記所標記的訊息，還提取與注釋相鄰的類名或方法名。透過這種方式，您就可以用最少的工作量來生成合適的程式文件。

Javadoc的輸出是一個html文件，可以用web瀏覽器查看它。有了Javadoc，就有一個簡單的標準來建立文件，因此你可以期望所有Java庫都有文件。

此外，你可以編寫自己的Javadoc處理程序doclet，對javadoc處理的訊息做特殊的處理（例如以不同的格式生成輸出）。

以下是對Javadoc基礎知識的介紹和概述。在 JDK 文件中可以找到完整的描述。

## 句法規則

所有Javadoc指令都發生在以 `/**` 開頭(但仍然以 `*/`)結尾)的注釋中。使用Javadoc有兩種主要方法：嵌入HTML或使用“doc標籤”。獨立的doc標籤是以 **@** 開頭並且放在注釋行的開頭的指令(注釋行開頭的`*`將被忽略)。內聯的doc標籤可以出現在Javadoc注釋的任何位置，它也以 `@` 開頭，但被花括號包圍。

有三種類型的注釋文件，它們對應於注釋前面的元素:類、欄位或方法。也就是說，類注釋出現在類定義之前，欄位注釋出現在欄位定義之前，方法注釋出現在方法定義之前。舉個簡單的例子:

```java
// javadoc/Documentation1.java 
/** 一個類注釋 */
public class Documentation1 {
    /** 一個屬性注釋 */
    public int i;
    /** 一個方法注釋 */ 
    public void f() {}
}

```

Javadoc僅處理 **公共成員** 和 **繼承訪問權限成員** 的注釋文件。 預設情況下，將忽略對 **私有成員** 和**包訪問權限成員**的注釋（請參閱["隱藏實現"](/docs/book/07-Implementation-Hiding.md)一章），並且你將看不到任何輸出。 這是有道理的，因為從用戶端程式設計師的視角看，在文件外部只有 **公共成員** 和 **繼承訪問權限成員** 是可用的。 你可以使用 **-private** 標誌來包含 **私有成員**。

要透過Javadoc處理前面的程式碼，指令是：

```cmd
javadoc Documentation1.java
```

這將產生一組HTML文件。如果你在瀏覽器中打開index.html，將看到輸出的結果與其他Java文件具有相同的標準格式，因此使用者對這種格式很熟悉，並可以輕鬆地瀏覽你的類。

## 內嵌 HTML

Javadoc不作修改地將HTML程式碼傳遞到生成的HTML文件。這使你可以充分利用HTML。但是這樣做的主要目的是讓你格式化程式碼，例如：

```java
// javadoc/Documentation2.java
/** <pre>
* System.out.println(new Date());
* </pre>
*/
public class Documentation2 {}
```

你也可以像在其他任何Web文件中一樣使用HTML來格式化說明中的文字：

```java
// javadoc/Documentation3.java
/** You can <em>even</em> insert a list:
* <ol>
* <li> Item one
* <li> Item two
* <li> Item three
* </ol>
*/
public class Documentation3 {}
```

請注意，在文件注釋中，Javadoc會刪除行首的星號以及前導空格。 Javadoc重新格式化了所有內容，使其符合文件的標準外觀。不要將`<h1>` 和`<hr>` 之類的標題用作嵌入式HTML，因為Javadoc會插入自己的標題，你插入的標題將對其產生干擾。

所有類型的注釋文件（類，欄位和方法）都可以支援嵌入式HTML。

## 範例標籤

以下是一些可用於程式碼文件的Javadoc標記。在嘗試使用Javadoc進行任何認真的操作之前，請查閱JDK文件中的Javadoc參考，以了解使用Javadoc的所有不同方法。

### @see

這個標籤可以將其它的類連結到本文件中。Javadoc 用 `@see` 標籤產生連結到其它類的的HTML。格式為：

```java
@see classname
@see fully-qualified-classname
@see fully-qualified-classname#method-name
```

每個都向生成的文件中添加超連結的“另請參閱”條目。 Javadoc 不會檢查超連結的有效性。

### {@link package.class#member label}

和 @see 非常相似，不同之處在於它可以內聯使用，並使用標籤作為超連結文字，而不是“另請參閱”。

### {@docRoot}

生成文件根目錄的相對路徑。對於顯式超連結到文件樹中的頁面很有用。

### {@inheritDoc}

將文件從此類的最近基類繼承到目前文件注釋中。

### @version

其形式為：

```java
@version version-information
```

其中 version-information 是你認為適合包含的任何重要訊息。當在Javadoc命令列上放置 -version 標誌時，特別在生成的HTML文件中用於生成version訊息。

### @author

其形式為：

```
@author author-information
```

 author-information 大機率是你的名字，但是一樣可以包含你的 email 地址或者其他合適的訊息。當在 Javadoc 命令列上放置 -author 標誌的時候，在生成的HTML文件中特別註明了作者訊息。

你可以對作者列表使用多個作者標籤，但是必須連續放置它們。所有作者訊息都集中在生成的HTML中的單個段落中。

### @since

此標記指示此程式碼的版本開始使用特定功能。例如，它出現在HTML Java文件中，以指示功能首次出現的JDK版本。

### @param

這將生成有關方法參數的文件：

```java
@param parameter-name description
```

其中 `parameter-name` 是方法參數列表中的標識符， `description` 是可以在後續行中繼續的文字。當遇到新的文件標籤時，這個說明被視為結束。`@param`標籤可以使用任意次，大概每個參數使用一次。

### @return

這記錄了返回值：

```java
@return description
```

其中description給出了返回值的含義。它可延續到後面的行內。

### @throws

一個方法可以產生許多不同類型的異常，所有這些異常都需要描述。異常標記的形式為：

```java
@throws fully-qualified-class-name description
```

*fully-qualified-class-name* 給出異常類的確切名稱，並且 description （可在後面的行內繼續展開）告訴你為什麼這個特定類型的異常會在方法呼叫後出現。

### @deprecated

表示已被改進的功能取代的功能。deprecated 標記建議你不要再使用此功能，因為將來它有可能被刪除。使用標記為 `@deprecated` 的方法會使編譯器發出警告。在Java 5中，`@deprecated`  Javadoc 標記被 `@Deprecated` *註解* 取代（在[註解]()一章中進行了描述）。

## 文件範例

**objects/HelloDate.java** 是帶有文件注釋的例子。

```java
// javadoc/HelloDateDoc.java
import java.util.*;
/** The first On Java 8 example program.
 * Displays a String and today's date.
 * @author Bruce Eckel
 * @author www.MindviewInc.com
 * @version 5.0
 */
public class HelloDateDoc {
    /** Entry point to class & application.
     * @param args array of String arguments
     * @throws exceptions No exceptions thrown
     */
    public static void main(String[] args) {
        System.out.println("Hello, it's: ");
        System.out.println(new Date());
    }
}
/* Output:
Hello, it's:
Tue May 09 06:07:27 MDT 2017
*/
```

你可以在Java標準庫的原始碼中找到許多Javadoc注釋文件的範例。



<!-- 分頁 -->

<div style="page-break-after: always;"></div>
