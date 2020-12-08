[TOC]

<!-- Implementation Hiding -->
# 第七章 封裝

> *訪問控制（Access control）*（或者*隱藏實現（implementation hiding）*）與“最初的實現不恰當”有關。

所有優秀的作者——包括那些編寫軟體的人——都知道一件好的作品都是經過反覆打磨才變得優秀的。如果你把一段程式碼置於某個位置一段時間，過一會重新來看，你可能發現更好的實現方式。這是*重構*（refactoring）的原動力之一，重構就是重寫可工作的程式碼，使之更加可讀，易懂，因而更易維護。

但是，在修改和完善程式碼的願望下，也存在巨大的壓力。通常，一些使用者（*用戶端程式設計師（client programmers）*）希望你的程式碼在某些方面保持不變。所以你想修改程式碼，但他們希望程式碼保持不變。由此引出了物件導向設計中的一個基本問題：“如何區分變動的事物和不變的事物”。

這個問題對於類庫（library）而言尤其重要。類庫的使用者必須依賴他們所使用的那部分類庫，並且知道如果使用了類庫的新版本，不需要改寫程式碼。另一方面，類庫的開發者必須有修改和改進類庫的自由，並保證客戶程式碼不會受這些改動影響。

這可以透過約定解決。例如，類庫開發者必須同意在修改類庫中的一個類時，不會移除已有的方法，因為那樣將會破壞用戶端程式設計師的程式碼。與之相反的情況更加複雜。在有成員屬性的情況下，類庫開發者如何知道哪些屬性被用戶端程式設計師使用？這同樣會發生在那些只為實現類庫類而建立的方法上，它們也不是設計成可供用戶端程式設計師呼叫的。如果類庫開發者想刪除舊的實現，添加新的實現，結果會怎樣呢？任何這些成員的改動都可能破環用戶端程式設計師的程式碼。因此類庫開發者會被束縛，不能修改任何事物。

為了解決這一問題，Java 提供了*訪問修飾符*（access specifier）供類庫開發者指明哪些對於用戶端程式設計師是可用的，哪些是不可用的。訪問控制權限的等級，從“最大權限”到“最小權限”依次是：**public**，**protected**，*包訪問權限（package access）*（沒有關鍵字）和 **private**。根據上一段的內容，你可能會想，作為一名類庫設計者，你會儘可能將一切都設為 **private**，僅向用戶端程式設計師暴露你願意他們使用的方法。這就是你通常所做的，儘管這與那些使用其他語言（尤其是 C）編程以及習慣了不受限制地訪問任何東西的人們的直覺相違背。

然而，類庫元件的概念和對類庫元件訪問的控制仍然不完善。其中仍然存在問題就是如何將類庫元件捆綁到一個內聚的類庫單元中。Java 中透過 **package** 關鍵字加以控制，類在相同包下還是在不同包下，會影響訪問修飾符。所以在這章開始，你將會學習如何將類庫元件置於同一個包下，之後你就能明白訪問修飾符的全部含義。

<!-- package: the Library Unit -->

## 包的概念

包內包含一組類，它們被組織在一個單獨的*命名空間*（namespace）下。

例如，標準 Java 發布中有一個工具庫，它被組織在 **java.util** 命名空間下。**java.util** 中含有一個類，叫做 **ArrayList**。使用 **ArrayList** 的一種方式是用其全名 **java.util.ArrayList**。

```java
// hiding/FullQualification.java

public class FullQualification {
    public static void main(String[] args) {
        java.util.ArrayList list = new java.util.ArrayList();
    }
}
```

這種方式使得程式冗長乏味，因此你可以換一種方式，使用 **import** 關鍵字。如果需要匯入某個類，就需要在 **import** 語句中聲明：

```java
// hiding/SingleImport.java
import java.util.ArrayList;

public class SingleImport {
    public static void main(String[] args) {
        ArrayList list = new ArrayList();
    }
}
```

現在你就可以不加限定詞，直接使用 **ArrayList** 了。但是對於 **java.util** 包下的其他類，你還是不能用。要匯入其中所有的類，只需使用 **\*** ，就像本書中其他範例那樣：
```java
import java.util.*
```

之所以使用匯入，是為了提供一種管理命名空間的機制。所有類名之間都是相互隔離的。類 **A** 中的方法 `f()` 不會與類 **B** 中具有相同簽名的方法 `f()` 衝突。但是如果類名衝突呢？假設你建立了一個 **Stack** 類，打算安裝在一台已經有別人所寫的 **Stack** 類的機器上，該怎麼辦呢？這種類名的潛在衝突，正是我們需要在 Java 中對命名空間進行完全控制的原因。為了解決衝突，我們為每個類建立一個唯一標識符組合。

到目前為止的大部分範例都只存在單個文件，並為本機使用的，所以尚未受到包名的干擾。但是，這些範例其實已經位於包中了，叫做“未命名”包或*預設包*（default package）。這當然是一種選擇，為了簡單起見，本書其餘部分會儘可能採用這種方式。但是，如果你打算為相同機器上的其他 Java 程式建立友好的類庫或程式時，就必須仔細考慮以防類名衝突。

一個 Java 原始碼文件稱為一個*編譯單元（compilation unit）*（有時也稱*翻譯單元（translation unit）*）。每個編譯單元的副檔名必須是 **.java**。在編譯單元中可以有一個 **public** 類，它的類名必須與檔案名相同（包括大小寫，但不包括副檔名 **.java**）。每個編譯單元中只能有一個 **public** 類，否則編譯器不接受。如果這個編譯單元中還有其他類，那麼在包之外是無法訪問到這些類的，因為它們不是 **public** 類，此時它們為主 **public** 類提供“支援”類 。

### 程式碼組織

當編譯一個 **.java** 文件時，**.java** 文件的每個類都會有一個輸出文件。每個輸出的檔案名和 **.java** 文件中每個類的類名相同，只是副檔名是 **.class**。因此，在編譯少量的 **.java** 文件後，會得到大量的 **.class** 文件。如果你使用過編譯型語言，那麼你可能習慣編譯後產生一個中間文件（通常稱為“obj”文件），然後與使用連結器（建立可執行文件）或類庫生成器（建立類庫）產生的其他同類文件打包到一起的情況。這不是 Java 工作的方式。在 Java 中，可執行程式是一組 **.class** 文件，它們可以打包壓縮成一個 Java 文件文件（JAR，使用 **jar** 文件生成器）。Java 解釋器負責尋找、載入和解釋這些文件。

類庫是一組類文件。每個來源文件通常都含有一個 **public** 類和任意數量的非 **public** 類，因此每個文件都有一個 **public** 元件。如果把這些元件集中在一起，就需要使用關鍵字 **package**。

如果你使用了 **package** 語句，它必須是文件中除了注釋之外的第一行程式碼。當你如下這樣寫：

```java
package hiding;
```

意味著這個編譯單元是一個名為 **hiding** 類庫的一部分。換句話說，你正在聲明的編譯單元中的 **public** 類名稱位於名為 **hiding** 的保護傘下。任何人想要使用該名稱，必須指明完整的類名或者使用 **import** 關鍵字匯入 **hiding** 。（注意，Java 包名按慣例一律小寫，即使中間的單詞也需要小寫，與駝峰命名不同）

例如，假設檔案名是 **MyClass.java** ，這意味著文件中只能有一個 **public** 類，且類名必須是 **MyClass**（大小寫也與檔案名相同）：

```java
// hiding/mypackage/MyClass.java
package hiding.mypackage;

public class MyClass {
    // ...
}
```

現在，如果有人想使用 **MyClass** 或 **hiding.mypackage** 中的其他 **public** 類，就必須使用關鍵字 **import** 來使 **hiding.mypackage** 中的名稱可用。還有一種選擇是使用完整的名稱：

```java
// hiding/QualifiedMyClass.java

public class QualifiedMyClass {
    public static void main(String[] args) {
        hiding.mypackage.MyClass m = new hiding.mypackage.MyClass();
    }
}
```

關鍵字 **import** 使之更簡潔：

```java
// hiding/ImportedMyClass.java
import hiding.mypackage.*;

public class ImportedMyClass {
    public static void main(String[] args) {
        MyClass m = new MyClass();
    }
}	
```

**package** 和 **import** 這兩個關鍵字將單一的全域命名空間分隔開，從而避免名稱衝突。

### 建立獨一無二的包名

你可能注意到，一個包從未真正被打包成單一的文件，它可以由很多 **.class** 文件構成，因而事情就變得有點複雜了。為了避免這種情況，一種合乎邏輯的做法是將特定包下的所有 **.class** 文件都放在一個目錄下。也就是說，利用作業系統的文件結構的層次性。這是 Java 解決混亂問題的一種方式；稍後你還會在我們介紹 **jar** 工具時看到另一種方式。

將所有的文件放在一個子目錄還解決了其他的兩個問題：建立獨一無二的包名和尋找可能隱藏於目錄結構某處的類。這是透過將 **.class** 文件所在的路徑位置編碼成 **package** 名稱來實現的。按照慣例，**package** 名稱是類的建立者的反順序的 Internet 域名。如果你遵循慣例，因為 Internet 域名是獨一無二的，所以你的 **package** 名稱也應該是獨一無二的，不會發生名稱衝突。如果你沒有自己的域名，你就得構造一組不大可能與他人重複的組合（比如你的姓名），來建立獨一無二的 package 名稱。如果你打算發布 Java 程式碼，那麼花些力氣去獲取一個域名是值得的。

此技巧的第二部分是把 **package** 名稱分解成你機器上的一個目錄，所以當 Java 解釋器必須要載入一個 .class 文件時，它能定位到 **.class** 文件所在的位置。首先，它找出環境變數 **CLASSPATH**（透過作業系統設定，有時也能透過 Java 的安裝程式或基於 Java 的工具設定）。**CLASSPATH** 包含一個或多個目錄，用作尋找 .**class** 文件的根目錄。從根目錄開始，Java 解釋器獲取包名並將每個句點取代成反斜線，生成一個基於根目錄的路徑名（取決於你的作業系統，包名 **foo.bar.baz** 變成 **foo\bar\baz** 或 **foo/bar/baz** 或其它）。然後這個路徑與 **CLASSPATH** 的不同項連接，解釋器就在這些目錄中尋找與你所建立的類名稱相關的 **.class** 文件（解釋器還會尋找某些涉及 Java 解釋器所在位置的標準目錄）。

為了理解這點，比如說我的域名 **MindviewInc.com**，將之反轉並全部改為小寫後就是 **com.mindviewinc**，這將作為我建立的類的獨一無二的全域名稱。（com、edu、org等副檔名之前在 Java 包中都是大寫，但是 Java 2 之後都統一用小寫。）我決定再建立一個名為 **simple** 的類庫，從而細分名稱：

```java
package com.mindviewinc.simple;
```

這個包名可以用作下面兩個文件的命名空間保護傘：

```java
// com/mindviewinc/simple/Vector.java
// Creating a package
package com.mindviewinc.simple;

public class Vector {
    public Vector() {
        System.out.println("com.mindviewinc.simple.Vector");
    }
}
```

如前所述，**package** 語句必須是文件的第一行非注釋程式碼。第二個文件看起來差不多：

```java
// com/mindviewinc/simple/List.java
// Creating a package
package com.mindviewinc.simple;

public class List {
    System.out.println("com.mindview.simple.List");
}
```

這兩個文件都位於我機器上的子目錄中，如下：

```
C:\DOC\Java\com\mindviewinc\simple
```

（注意，本書的每個文件的第一行注釋都指明了文件在原始碼目錄樹中的位置——供本書的自動程式碼提取工具使用。）

如果你回頭看這個路徑，會看到包名 **com.mindviewinc.simple**，但是路徑的第一部分呢？CLASSPATH 環境變數會處理它。我機器上的環境變數部分如下：

```
CLASSPATH=.;D:\JAVA\LIB;C:\DOC\Java
```

CLASSPATH 可以包含多個不同的搜尋路徑。

但是在使用 JAR 文件時，有點不一樣。你必須在類路徑寫清楚 JAR 文件的實際名稱，不能僅僅是 JAR 文件所在的目錄。因此，對於一個名為 **grape.jar** 的 JAR 文件，類路徑應包括：

```
CLASSPATH=.;D\JAVA\LIB;C:\flavors\grape.jar
```

一旦設定好類路徑，下面的文件就可以放在任意目錄：

```java
// hiding/LibTest.java
// Uses the library
import com.mindviewinc.simple.*;

public class LibTest {
    public static void main(String[] args) {
        Vector v = new Vector();
        List l = new List();
    }
}
```

輸出：

```
com.mindviewinc.simple.Vector
com.mindviewinc.simple.List
```

當編譯器遇到匯入 **simple** 庫的 **import** 語句時，它首先會在 CLASSPATH 指定的目錄中尋找子目錄 **com/mindviewinc/simple**，然後從已編譯的文件中找出名稱相符者（對 **Vector** 而言是 **Vector.class**，對 **List** 而言是 **List.class**）。注意，這兩個類和其中要訪問的方法都必須是 **public** 修飾的。

對於 Java 新手而言，設定 CLASSPATH 是一件麻煩的事（我最初使用時是這麼覺得的），後面版本的 JDK 更加智慧。你會發現當你安裝好 JDK 時，即使不設定 CLASSPATH，也能夠編譯和執行基本的 Java 程式。但是，為了編譯和執行本書的程式碼範例（從[https://github.com/BruceEckel/OnJava8-examples](https://github.com/BruceEckel/OnJava8-examples) 取得），你必須將本書程式碼樹的基本目錄加入到 CLASSPATH 中（ gradlew 指令管理自身的 CLASSPATH，所以如果你想直接使用 javac 和 java，不用 Gradle 的話，就需要設定 CLASSPATH）。

### 衝突

如果透過 **\*** 匯入了兩個包含相同名字類名的類庫，會發生什麼事？例如，假設程式如下：

```java
import com.mindviewinc.simple.*;
import java.util.*;
```

因為 **java.util.*** 也包含了 **Vector** 類，這就存在潛在的衝突。但是只要你不寫導致衝突的程式碼，就不會有問題——這樣很好，否則就得做很多類型檢查工作來防止那些根本不會出現的衝突。

現在如果要建立一個 Vector 類，就會出現衝突：

```java
Vector v = new Vector();
```

這裡的 **Vector** 類指的是誰呢？編譯器不知道，讀者也不知道。所以編譯器報錯，強制你明確指明。對於標準的 Java 類 **Vector**，你可以這麼寫：

```java
java.util.Vector v = new java.util.Vector();
```

這種寫法完全指明了 **Vector** 類的位置（配合 CLASSPATH），那麼就沒有必要寫 **import java.util.*** 語句，除非使用其他來自 **java.util** 中的類。

或者，可以匯入單個類以防衝突——只要不在同一個程式中使用有衝突的名字（若使用了有衝突的名字，必須明確指明全名）。

### 訂製工具庫

具備了以上知識，現在就可以建立自己的工具庫來減少重複的程式碼了。

一般來說，我會使用反轉後的域名來命名要建立的工具包，比如 **com.mindviewinc.util** ，但為了簡化，這裡我把工具包命名為 **onjava**。

比如，下面是“控制流”一章中使用到的 `range()` 方法，採用了 for-in 語法進行簡單的遍歷：

```java
// onjava/Range.java
// Array creation methods that can be used without
// qualifiers, using static imports:
package onjava;

public class Range {
    // Produce a sequence [0,n)
    public static int[] range(int n) {
        int[] result = new int[n];
        for (int i = 0; i < n; i++) {
            result[i] = i;
        }
        return result;
    }
    // Produce a sequence [start..end)
    public static int[] range(int start, int end) {
        int sz = end - start;
        int[] result = new int[sz];
        for (int i = 0; i < sz; i++) {
            result[i] = start + i;
        }
        return result;
    }
    // Produce sequence [start..end) incrementing by step
    public static int[] range(int start, int end, int step) {
        int sz = (end - start) / step;
        int[] result = new int[sz];
        for (int i = 0; i < sz; i++) {
            result[i] = start + (i * step);
        }
        return result;
    }
}
```

這個文件的位置一定是在某個以一個 CLASSPATH 位置開始，然後接著是 **onjava** 的目錄下。編譯完之後，就可以在系統的任何地方使用 **import onjava** 語句來使用這些方法了。

從現在開始，無論何時你建立了有用的新工具，都可以把它加入到自己的類庫中。在本書中，你將會看到更多的元件加入到 **onjava** 庫。

### 使用 import 改變行為

Java 沒有 C 的*條件編譯*（conditional compilation）功能，該功能使你不必更改任何程式碼而能夠切換開關產生不同的行為。Java 之所以去掉此功能，可能是因為 C 在絕大多數情況下使用該功能解決跨平台問題：程式碼的不同部分要根據不同的平台來編譯。而 Java 自身就是跨平台設計的，這個功能就沒有必要了。

但是，條件編譯還有其他的用途。除錯是一個很常見的用途，除錯功能在開發過程中是開啟的，在發布的產品中是禁用的。可以透過改變匯入的 **package** 來實現這一目的，修改的方法是將程式中的程式碼從除錯版改為發布版。這個技術可用於任何種類的條件程式碼。

### 使用包的忠告

當建立一個包時，包名就隱含了目錄結構。這個包必須位於包名指定的目錄中，該目錄必須在以 CLASSPATH 開始的目錄中可以查詢到。 最初使用關鍵字 **package** 可能會有點不順，因為除非遵守“包名對應目錄路徑”的規則，否則會收到很多意外的執行時錯誤訊息如找不到特定的類，即使這個類就位於同一目錄中。如果你收到類似訊息，嘗試把 **package** 語句注釋掉，如果程式能執行的話，你就知道問題出現在哪裡了。

注意，編譯過的程式碼通常位於與原始碼的不同目錄中。這是很多工程的標準，而且整合開發環境（IDE）通常會自動為我們做這些。必須保證 JVM 通過 CLASSPATH 能找到編譯後的程式碼。

<!-- Java Access Specifiers -->

## 訪問權限修飾符

Java 訪問權限修飾符 **public**，**protected** 和 **private** 位於定義的類名，屬性名和方法名之前。每個訪問權限修飾符只能控制它所修飾的物件。

如果不提供訪問修飾符，就意味著"包訪問權限"。所以無論如何，萬物都有某種形式的訪問控制權。接下來的幾節中，你將學習各種類型的訪問權限。

### 包訪問權限

本章之前的所有範例要嘛使用 **public** 訪問修飾符，要嘛就沒使用修飾符（*預設訪問權限（default access）*）。預設訪問權限沒有關鍵字，通常被稱為*包訪問權限（package access）*（有時也稱為 **friendly**）。這意味著目前包中的所有其他類都可以訪問那個成員。對於這個包之外的類，這個成員看起來是 **private** 的。由於一個編譯單元（即一個文件）只能隸屬於一個包，所以透過包訪問權限，位於同一編譯單元中的所有類彼此之間都是可訪問的。

包訪問權限可以把相關類聚到一個包下，以便它們能輕易地相互訪問。包裡的類賦予了它們包訪問權限的成員相互訪問的權限，所以你"擁有”了包內的程式碼。只能透過你所擁有的程式碼去訪問你所擁有的其他程式碼，這樣規定很有意義。構建包訪問權限機制是將類聚集在包中的重要原因之一。在許多語言中，在文件中組織定義的方式是任意的，但是在 Java 中你被強制以一種合理的方式組織它們。另外，你可能會將不應該對目前包中的類具有訪問權限的類排除在包外。

類控制著哪些程式碼有權訪問自己的成員。其他包中的程式碼不能一上來就說"嗨，我是 **Bob** 的朋友！"，然後想看到 **Bob** 的 **protected**、包訪問權限和 **private** 成員。取得對成員的訪問權的唯一方式是：

1. 使成員成為 **public**。那麼無論是誰，無論在哪，都可以訪問它。
2. 賦予成員預設包訪問權限，不用加任何訪問修飾符，然後將其他類放在相同的包內。這樣，其他類就可以訪問該成員。
3. 在"復用"這一章你將看到，繼承的類既可以訪問 **public** 成員，也可以訪問 **protected** 成員（但不能訪問 **private** 成員）。只有當兩個類處於同一個包內，它才可以訪問包訪問權限的成員。但現在不用擔心繼承和 **protected**。
4. 提供訪問器（accessor）和修改器（mutator）方法（有時也稱為"get/set" 方法），從而讀取和改變值。

### public: 介面訪問權限

當你使用關鍵字 **public**，就意味著緊隨 public 後聲明的成員對於每個人都是可用的，尤其是使用類庫的用戶端程式設計師更是如此。假設定義了一個包含下面編譯單元的 **dessert** 包：

```java
// hiding/dessert/Cookie.java
// Creates a library
package hiding.dessert;

public class Cookie {
    public Cookie() {
        System.out.println("Cookie constructor");
    }
    
    void bite() {
        System.out.println("bite");
    }
}
```

記住，**Cookie.java** 文件產生的類文件必須位於名為 **dessert** 的子目錄中，該子目錄在 **hiding** （表明本書的"封裝"章節）下，它必須在 CLASSPATH 的幾個目錄之下。不要錯誤地認為 Java 總是會將目前目錄視作尋找行為的起點之一。如果你的 CLASSPATH 中沒有 **.**，Java 就不會尋找目前目錄。

現在，使用 **Cookie** 建立一個程式：
```java
// hiding/Dinner.java
// Uses the library
import hiding.dessert.*;

public class Dinner {
    public static void main(String[] args) {
        Cookie x = new Cookie();
        // -x.bite(); // Can't access
    }
}
```

輸出：

```
Cookie constructor
```

你可以建立一個 **Cookie** 物件，因為它構造器和類都是 **public** 的。（後面會看到更多 **public** 的概念）但是，在 **Dinner.java** 中無法訪問到 **Cookie** 物件中的 `bite()` 方法，因為 `bite()` 只提供了包訪問權限，因而在 **dessert** 包之外無法訪問，編譯器禁止你使用它。

### 預設包

你可能驚訝地發現，以下程式碼儘管看起來破壞了規則，但是仍然可以編譯：

```java
// hiding/Cake.java
// Accesses a class in a separate compilation unit
class Cake {
    public static void main(String[] args) {
        Pie x = new Pie();
        x.f();
    }
}
```

輸出：

```
Pie.f()
```

同一目錄下的第二個文件：

```java
// hiding/Pie.java
// The other class
class Pie {
    void f() {
        System.out.println("Pie.f()");
    }
}
```

最初看起來這兩個文件毫不相關，但在 **Cake** 中可以建立一個 **Pie** 物件並呼叫它的 `f()` 方法。（注意，你的 CLASSPATH 中一定得有 **.**，這樣文件才能編譯）通常會認為 **Pie** 和  `f()` 具有包訪問權限，因此不能被 **Cake** 訪問。它們的確具有包訪問權限，這是部分正確。**Cake.java** 可以訪問它們是因為它們在相同的目錄中且沒有給自己設定明確的包名。Java 把這樣的文件看作是隸屬於該目錄的預設包中，因此它們為該目錄中所有的其他文件都提供了包訪問權限。

### private: 你無法訪問

關鍵字 **private** 意味著除了包含該成員的類，其他任何類都無法訪問這個成員。同一包中的其他類無法訪問 **private** 成員，因此這等於說是自己隔離自己。另一方面，讓許多人合作建立一個包也是有可能的。使用 **private**，你可以自由地修改那個被修飾的成員，無需擔心會影響同一包下的其他類。

預設的包訪問權限通常提供了足夠的隱藏措施；記住，使用類的用戶端程式設計師無法訪問包訪問權限成員。這樣做很好，因為預設訪問權限是一種我們常用的權限（同時也是一種在忘記添加任何訪問權限時自動得到的權限）。因此，通常考慮的是把哪些成員聲明成 **public** 供用戶端程式設計師使用。所以，最初不常使用關鍵字 **private**，因為程式沒有它也可以照常工作。然而，使用 **private** 是非常重要的，尤其是在多執行緒環境中。（在"並發編程"一章中將看到）。

以下是一個使用 **private** 的例子：

```java
// hiding/IceCream.java
// Demonstrates "private" keyword

class Sundae {
    private Sundae() {}
    static Sundae makeASundae() {
        return new Sundae();
    }
}

public class IceCream {
    public static void main(String[] args) {
        //- Sundae x = new Sundae();
        Sundae x = Sundae.makeASundae();
    }
}
```

以上展示了 **private** 的用武之地：控制如何建立物件，防止別人直接訪問某個特定的構造器（或全部構造器）。例子中，你無法透過構造器建立一個 **Sundae** 物件，而必須呼叫 `makeASundae()` 方法建立物件。

任何可以肯定只是該類的"助手"方法，都可以聲明為 **private**，以確保不會在包中的其他地方誤用它，也防止了你會去改變或刪除它。將方法聲明為 **private** 確保了你擁有這種選擇權。

對於類中的 **private** 屬性也是一樣。除非必須公開底層實現（這種情況很少見），否則就將屬性聲明為 **private**。然而，不能因為類中某個物件的引用是 **private**，就認為其他物件也無法擁有該物件的 **public** 引用（參見附錄：物件傳遞和返回）。

### protected: 繼承訪問權限

要理解 **protected** 的訪問權限，我們在內容上需要作一點跳躍。首先，在介紹本書"復用"章節前，你不必真正理解本節的內容。但為了內容的完整性，這裡作了簡要介紹，舉了個使用 **protected** 的例子。

關鍵字 **protected** 處理的是繼承的概念，透過繼承可以利用一個現有的類——我們稱之為基類，然後添加新成員到現有類中而不必碰現有類。我們還可以改變類的現有成員的行為。為了從一個類中繼承，需要聲明新類 extends 一個現有類，像這樣：

```java
class Foo extends Bar {}
```

類定義的其他部分看起來是一樣的。

如果你建立了一個新包，並從另一個包繼承類，那麼唯一能訪問的就是被繼承類的 **public** 成員。（如果在同一個包中繼承，就可以操作所有的包訪問權限的成員。）有時，基類的建立者會希望某個特定成員能被繼承類訪問，但不能被其他類訪問。這時就需要使用 **protected**。**protected** 也提供包訪問權限，也就是說，相同包內的其他類可以訪問 **protected** 元素。

回顧下先前的文件 **Cookie.java**，下面的類不能呼叫包訪問權限的方法 `bite()`：

```java
// hiding/ChocolateChip.java
// Can't use package-access member from another package
import hiding.dessert.*;

public class ChocolateChip extends Cookie {
    public ChocolateChip() {
        System.out.println("ChocolateChip constructor");
    } 
    
    public void chomp() {
        //- bite(); // Can't access bite
    }
    
    public static void main(String[] args) {
        ChocolateChip x = new ChocolateChip();
        x.chomp();
    }
}
```

輸出：

```
Cookie constructor
ChocolateChip constructor
```

如果類 **Cookie** 中存在一個方法 `bite()`，那麼它的任何子類中都存在 `bite()` 方法。但是因為 `bite()` 具有包訪問權限並且位於另一個包中，所以我們在這個包中無法使用它。你可以把它聲明為 **public**，但這樣一來每個人都能訪問它，這可能也不是你想要的。如果你將 **Cookie** 改成如下這樣：

```java
// hiding/cookie2/Cookie.java
package hiding.cookie2;

public class Cookie {
    public Cookie() {
        System.out.println("Cookie constructor");
    }
    
    protected void bite() {
        System.out.println("bite");
    }
}
```

這樣，`bite()` 對於所有繼承 **Cookie** 的類，都是可訪問的：

```java
// hiding/ChocolateChip2.java
import hiding.cookie2.*;

public class ChocolateChip2 extends Cookie {
    public ChocoalteChip2() {
        System.out.println("ChocolateChip2 constructor");
    }
    
    public void chomp() {
        bite(); // Protected method
    }
    
    public static void main(String[] args) {
        ChocolateChip2 x = new ChocolateChip2();
        x.chomp();
    }
}
```

輸出：

```
Cookie constructor
ChocolateChip2 constructor
bite
```

儘管 `bite()` 也具有包訪問權限，但它不是 **public** 的。

### 包訪問權限 Vs Public 構造器

當你定義一個具有包訪問權限的類時，你可以在類中定義一個 public 構造器，編譯器不會報錯：

```java
// hiding/packageaccess/PublicConstructor.java
package hiding.packageaccess;

class PublicConstructor {
    public PublicConstructor() {}
}
```

有一個 Checkstyle 工具，你可以執行指令 **gradlew hiding:checkstyleMain** 使用它，它會指出這種寫法是虛假的，而且從技術上來說是錯誤的。實際上你不能從包外訪問到這個 **public** 構造器：

```java
// hiding/CreatePackageAccessObject.java
// {WillNotCompile}
import hiding.packageaccess.*;

public class CreatePackageAcessObject {
    public static void main(String[] args) {
        new PublicConstructor();
    }
}
```

如果你編譯下這個類，會得到編譯錯誤訊息：

```
CreatePackageAccessObject.java:6:error:
PublicConstructor is not public in hiding.packageaccess;
cannot be accessed from outside package
new PublicConstructor();
^
1 error
```

因此，在一個具有包訪問權限的類中定義一個 **public** 的構造器並不能真的使這個構造器成為 **public**，在聲明的時候就應該標記為編譯時錯誤。

<!-- Interface and Implementation -->

## 介面和實現

訪問控制通常被稱為*隱藏實現*（implementation hiding）。將資料和方法包裝進類中並把具體實現隱藏被稱作是*封裝*（encapsulation）。其結果就是一個同時帶有特徵和行為的資料類型。

出於兩個重要的原因，訪問控制在資料類型內部劃定了邊界。第一個原因是確立用戶端程式設計師可以使用和不能使用的邊界。可以在結構中建立自己的內部機制而不必擔心用戶端程式設計師偶爾將內部實現作為他們可以使用的介面的一部分。

這直接引出了第二個原因：將介面與實現分離。如果在一組程式中使用介面，而用戶端程式設計師只能向 **public** 介面發送消息的話，那麼就可以自由地修改任何不是 **public** 的事物（例如包訪問權限，protected，或 private 修飾的事物），卻不會破壞用戶端程式碼。

為了清晰起見，你可以採用一種建立類的風格：**public** 成員放在類的開頭，接著是 **protected** 成員，包訪問權限成員，最後是 **private** 成員。這麼做的好處是類的使用者可以從頭讀起，首先會看到對他們而言最重要的部分（public 成員，因為可以從文件外訪問它們），直到遇到非 **public** 成員時停止閱讀，下面就是內部實現了：

```java
// hiding/OrganizedByAccess.java

public class OrganizedByAccess {
    public void pub1() {/* ... */}
    public void pub2() {/* ... */}
    public void pub3() {/* ... */}
    private void priv1() {/* ... */}
    private void priv2() {/* ... */}
    private void priv3() {/* ... */}
    private int i;
    // ...
}
```

這麼做只能是程式閱讀起來稍微容易一些，因為實現和介面還是混合在一起。也就是說，你仍然能看到原始碼——實現部分，因為它就在類中。另外，javadoc 提供的注釋文件功能降低了程式碼的可讀性對用戶端程式設計師的重要性。將介面展現給類的使用者實際上是類瀏覽器的任務，類瀏覽器會展示所有可用的類，並告訴你如何使用它們（比如說哪些成員可用）。在 Java 中，JDK 文件起到了類瀏覽器的作用。

<!-- Class Access -->

## 類訪問權限

訪問權限修飾符也可以用於確定類庫中的哪些類對於類庫的使用者是可用的。如果希望某個類可以被用戶端程式設計師使用，就把關鍵字 **public** 作用於整個類的定義。這甚至控制著用戶端程式設計師能否建立類的物件。

為了控制一個類的訪問權限，修飾符必須出現在關鍵字 **class** 之前：

```java
public class Widget {
```

如果你的類庫名是 **hiding**，那麼任何用戶端程式設計師都可以透過如下聲明訪問 **Widget**：

```java
import hiding.Widget;
```

或者

```java
import hiding.*;
```

這裡有一些額外的限制：

1. 每個編譯單元（即每個文件）中只能有一個 **public** 類。這表示，每個編譯單元有一個公共的介面用 **public** 類表示。該介面可以包含許多支援包訪問權限的類。一旦一個編譯單元中出現一個以上的 **public** 類，編譯就會報錯。
2. **public** 類的名稱必須與含有該編譯單元的檔案名相同，包括大小寫。所以對於 **Widget** 來說，檔案名必須是 **Widget.java**，不能是 **widget.java** 或 **WIDGET.java**。再次強調，如果名字不匹配，編譯器會報錯。
3. 雖然不是很常見，但是編譯單元內沒有 **public** 類也是可能的。這時可以隨意命名文件（儘管隨意命名會讓程式碼的閱讀者和維護者感到困惑）。

如果是一個在 **hiding** 包中的類，只用來完成 **Widget** 或 **hiding** 包下一些其他 **public** 類所要執行的任務，怎麼設定它的訪問權限呢？ 你不想自找麻煩為用戶端程式設計師建立說明文件，並且你認為不久後會完全改變原有方案並將舊版本刪除，取代成新版本。為了保留此靈活性，需要確保用戶端程式設計師不依賴隱藏在 **hiding** 中的任何特定細節，那麼把 **public** 關鍵字從類中去掉，給予它包訪問權限，就可以了。

當你建立了一個包訪問權限的類，把類中的屬性聲明為 **private** 仍然是有意義的——應該儘可能將所有屬性都聲明為 **private**，但是通常把方法聲明成與類（包訪問權限）相同的訪問權限也是合理的。一個包訪問權限的類只能被用於包內，除非強制將某些方法聲明為 **public**，這種情況下，編譯器會告訴你。

注意，類既不能是 **private** 的（這樣除了該類自身，任何類都不能訪問它），也不能是 **protected** 的。所以對於類的訪問權限只有兩種選擇：包訪問權限或者 **public**。為了防止類被外界訪問，可以將所有的構造器聲明為 **private**，這樣只有你自己能建立物件（在類的 static 成員中）：

```java
// hiding/Lunch.java
// Demonstrates class access specifiers. Make a class
// effectively private with private constructors:

class Soup1 {
    private Soup1() {}
    
    public static Soup1 makeSoup() { // [1]
        return new Soup1();
    }
}

class Soup2 {
    private Soup2() {}
    
    private static Soup2 ps1 = new Soup2(); // [2]
    
    public static Soup2 access() {
        return ps1;
    }
    
    public void f() {}
}
// Only one public class allowed per file:
public class Lunch {
    void testPrivate() {
        // Can't do this! Private constructor:
        //- Soup1 soup = new Soup1();
    }
    
    void testStatic() {
        Soup1 soup = Soup1.makeSoup();
    }
    
    void testSingleton() {
        Soup2.access().f();
    }
}
```

可以像 [1] 那樣透過 **static** 方法建立物件，也可以像 [2] 那樣先建立一個靜態物件，當使用者需要訪問它時返回物件的引用即可。

到目前為止，大部分的方法要嘛返回 void，要嘛返回基本類型，所以 [1] 處的定義乍看之下會有點困惑。方法名（**makeSoup**）前面的 **Soup1** 表明了方法返回的類型。到目前為止，這裡經常是 **void**，即不返回任何東西。然而也可以返回物件的引用，就像這裡一樣。這個方法返回了對 **Soup1** 類物件的引用。

**Soup1** 和 **Soup2** 展示了如何透過將你所有的構造器聲明為 **private** 的方式防止直接建立某個類的物件。記住，如果你不顯式地建立構造器，編譯器會自動為你建立一個無參構造器（沒有參數的構造器）。如果我們編寫了無參構造器，那麼編譯器就不會自動建立構造器了。將構造器聲明為 **private**，那麼誰也無法建立該類的物件了。但是現在別人該怎麼使用這個類呢？上述例子給出了兩個選擇。在 **Soup1** 中，有一個 **static** 方法，它的作用是建立一個新的 **Soup1** 物件並返回物件的引用。如果想要在返回引用之前在 **Soup1** 上做一些額外操作，或是記錄建立了多少個 **Soup1** 物件（可以用來限制數量），這種做法是有用的。

**Soup2** 用到了所謂的*設計模式*（design pattern）。這種模式叫做*單例模式*（singleton），因為它只允許建立類的一個物件。**Soup2** 類的物件是作為 **Soup2** 的 **static** **private** 成員而建立的，所以有且只有一個，你只能透過 **public** 修飾的 `access()` 方法訪問到這個物件。

<!-- Summary -->

## 本章小結

無論在什麼樣的關係中，劃定一些供各成員共同遵守的界限是很重要的。當你建立了一個類庫，也就與該類庫的使用者產生了聯繫，他們是類庫的用戶端程式設計師，需要使用你的類庫建立應用或更大的類庫。

沒有規則，用戶端程式設計師就可以對類的所有成員為所欲為，即使你希望他們不要操作部分成員。這種情況下，所有事物都是公開的。

本章討論了類庫是如何透過類構建的：首先，介紹了將一組類打包到類庫的方式，其次介紹了類如何控制對其成員的訪問。

據估計，用 C 語言開發項目，當程式碼量達到 5 萬行和 10 萬行時就會出現問題，因為 C 語言只有單一的命名空間，名稱開始衝突造成額外的管理開銷。在 Java 中，關鍵字 **package**，包命名模式和關鍵字 **import** 給了你對於名稱的完全控制權，因此可以輕易地避免名稱衝突的問題。

控制成員訪問權限有兩個原因。第一個原因是使用戶不要接觸他們不該接觸的部分，這部分對於類內部來說是必要的，但是不屬於用戶端程式設計師所需介面的一部分。因此將方法和屬性聲明為 **private** 對於用戶端程式設計師來說是一種服務，可以讓他們清楚地看到什麼是重要的，什麼可以忽略。這可以簡化他們對類的理解。

第二個也是最重要的原因是為了讓類庫設計者更改類內部的工作方式，而不用擔心會影響到用戶端程式設計師。比如最初以某種方式建立一個類，隨後發現如果更改程式碼結構可以極大地提高執行速度。如果介面與實現被明確地隔離和保護，你可以實現這一目的，而不必強制用戶端程式設計師重新編寫程式碼。訪問權限控制確保用戶端程式設計師不會依賴某個類的底層實現的任何部分。

當你具備更改底層實現的能力時，不但可以自由地改善設計，還可能會隨意地犯錯。無論如何細心地計劃和設計，都有可能犯錯。當了解到犯錯是相對安全的時候，你可以更加放心地實驗，更快地學會，更快地完成項目。

類的 **public** 介面是使用者真正看到的，所以在分析和設計階段決定這部分介面是最重要的部分。儘管如此，你仍然有改變的空間。如果最初沒有建立出正確的介面，可以添加更多的方法，只要你不刪除那些用戶端程式設計師已經在他們的程式碼中使用的東西。

注意到訪問權限控制關注的是類庫建立者和外部使用者之間的關係，這是一種交流方式。很多情況下，事實並非如此。例如，你自己編寫了所有的程式碼，或者在一個小組中工作，所有的東西都放在同一個包下。這些情況下，交流方式則是另外一種，此時嚴格地遵循訪問權限規則也許不是最佳選擇，預設（包）訪問權限也許就足夠好了。

<!-- 分頁 -->

<div style="page-break-after: always;"></div>
