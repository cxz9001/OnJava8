[TOC]

<!-- Exceptions -->
# 第十五章 異常

> Java 的基本理念是“結構不佳的程式碼不能執行”。

改進的錯誤復原機制是提高程式碼健壯性的最強有力的方式。錯誤復原在我們所編寫的每一個程式中都是基本的要素，但是在 Java 中它顯得格外重要，因為 Java 的主要目標之一就是建立供他人使用的程式構件。

發現錯誤的理想時機是在編譯階段，也就是在你試圖執行程式之前。然而，編譯期間並不能找出所有的錯誤，餘下的問題必須在執行期間解決。這就需要錯誤源能透過某種方式，把適當的訊息傳遞給某個接收者——該接收者將知道如何正確處理這個問題。

> 要想建立健壯的系統，它的每一個構件都必須是健壯的。

Java 使用異常來提供一致的錯誤報告模型，使得構件能夠與用戶端程式碼可靠地溝通問題。

Java 中的異常處理的目的在於透過使用少於目前數量的程式碼來簡化大型、可靠的程式的生成，並且透過這種方式可以使你更加確信：你的應用中沒有未處理的錯誤。異常的相關知識學起來並非艱澀難懂，並且它屬於那種可以使你的項目受益明顯、立竿見影的特性之一。

因為異常處理是 Java 中唯一官方的錯誤報告機制，並且透過編譯器強制執行，所以不學習異常處理的話，你也就只能寫出書中那麼些例子了。本章將教你如何編寫正確的異常處理程序，以及當方法出問題的時候，如何產生自訂的異常。

<!-- Concepts -->

## 異常概念

C 以及其他早期語言常常具有多種錯誤處理模式，這些模式往往建立在約定俗成的基礎之上，而並不屬於語言的一部分。通常會返回某個特殊值或者設定某個標誌，並且假定接收者將對這個返回值或標誌進行檢查，以判定是否發生了錯誤。然而，隨著時間的推移，人們發現，高傲的程式設計師們在使用程式庫的時候更傾向於認為：“對，錯誤也許會發生，但那是別人造成的，不關我的事”。所以，程式設計師不去檢查錯誤情形也就不足為奇了（何況對某些錯誤情形的檢查確實很無聊）。如果的確在每次呼叫方法的時候都徹底地進行錯誤檢查，程式碼很可能會變得難以閱讀。正是由於程式設計師還仍然用這些方式拼湊系統，所以他們拒絕承認這樣一個事實：對於構造大型、健壯、可維護的程式而言，這種錯誤處理模式已經成為了主要障礙。

解決的辦法是，用強制規定的形式來消除錯誤處理過程中隨心所欲的因素。這種做法由來已久，對異常處理的實現可以追溯到 20 世紀 60 年代的作業系統，甚至於 BASIC 語言中的“on error goto”語句。而 C++的異常處理機制基於 Ada，Java 中的異常處理機制則建立在 C++ 的基礎之上（儘管看起來更像 Object Pascal）。

“異常”這個詞有“我對此感到意外”的意思。問題出現了，你也許不清楚該如何處理，但你的確知道不應該置之不理，你要停下來，看看是不是有別人或在別的地方，能夠處理這個問題。只是在目前的環境中還沒有足夠的訊息來解決這個問題，所以就把這個問題提交到一個更進階別的環境中，在那裡將作出正確的決定。

異常往往能降低錯誤處理程式碼的複雜度。如果不使用異常，那麼就必須檢查特定的錯誤，並在程式中的許多地方去處理它。而如果使用異常，那就不必在方法呼叫處進行檢查，因為異常機制將保證能夠捕獲這個錯誤。理想情況下，只需在一個地方處理錯誤，即所謂的異常處理程序中。這種方式不僅節省程式碼，而且把“描述在正常執行過程中做什麼事”的程式碼和“出了問題怎麼辦”的程式碼相分離。總之，與以前的錯誤處理方法相比，異常機制使程式碼的閱讀、編寫和除錯工作更加井井有條。

<!-- Basic Exceptions -->

## 基本異常

異常情形（exceptional condition）是指阻止當前方法或作用域繼續執行的問題。把異常情形與普通問題相區分很重要，所謂的普通問題是指，在目前環境下能得到足夠的訊息，總能處理這個錯誤。而對於異常情形，就不能繼續下去了，因為在目前環境下無法獲得必要的訊息來解決問題。你所能做的就是從目前環境跳出，並且把問題提交給上一級環境。這就是拋出異常時所發生的事情。

除法就是一個簡單的例子。除數有可能為 0，所以先進行檢查很有必要。但除數為 0 代表的究竟是什麼意思呢？透過目前正在解決的問題環境，或許能知道該如何處理除數為 0 的情況。但如果這是一個意料之外的值，你也不清楚該如何處理，那就要拋出異常，而不是順著原來的路徑繼續執行下去。

當拋出異常後，有幾件事會隨之發生。首先，同 Java 中其他對像的建立一樣，將使用 new 在堆上建立異常物件。然後，目前的執行路徑（它不能繼續下去了）被終止，並且從目前環境中彈出對異常物件的引用。此時，異常處理機制接管程式，並開始尋找一個恰當的地方來繼續處理程序。這個恰當的地方就是異常處理程序，它的任務是將程式從錯誤狀態中復原，以使程式能要嘛換一種方式執行，要嘛繼續執行下去。

舉一個拋出異常的簡單例子。對於物件引用 t，傳給你的時候可能尚未被初始化。所以在使用這個物件引用呼叫其方法之前，會先對引用進行檢查。可以建立一個代表錯誤訊息的物件，並且將它從目前環境中“拋出”，這樣就把錯誤訊息傳播到了“更大”的環境中。這被稱為*拋出一個異常*，看起來像這樣：

```java
if(t == null)
    throw new NullPointerException();
```

這就拋出了異常，於是在目前環境下就不必再為這個問題操心了，它將在別的地方得到處理。具體是哪個“地方”後面很快就會介紹。

異常允許你將做的每件事都當作一個事務來考慮，而異常守護著這些事務：“...事務的基本保障是，我們需要的分布式計算的異常處理機制。事務相當於電腦中的契約法。如果任何事出現了錯誤，我們只需要丟棄整個計算。”你也可以將異常看作一種內建的“復原”（undo）系統，因為（在細心使用時）你在程式中可以有各種復原點。一旦程式的一個部分失敗了，異常將“復原”到一個已知的穩定點上。

<!-- Exception Arguments -->

### 異常參數

與使用 Java 中的其他物件一樣，我們總是用 new 在堆上建立異常物件，這也伴隨著儲存空間的分配和構造器的呼叫。所有標準異常類都有兩個構造器：一個是無參構造器；另一個是接受字串作為參數，以便能把相關訊息放入異常物件的構造器：

```java
throw new NullPointerException("t = null");
```

不久讀者將看到，要把這個字串的內容提取出來可以有多種不同的方法。

關鍵字 **throw** 將產生許多有趣的結果。在使用 **new** 建立了異常物件之後，此物件的引用將傳給 **throw**。儘管異常物件的類型通常與方法設計的返回類型不同，但從效果上看，它就像是從方法“返回”的。可以簡單地把異常處理看成一種不同的返回機制，當然若過分強調這種類比的話，就會有麻煩了。另外還能用拋出異常的方式從目前的作用域退出。在這兩種情況下，將會返回一個異常物件，然後退出方法或作用域。

拋出異常與方法正常返回的相似之處到此為止。因為異常返回的“地點”與普通方法呼叫返回的“地點”完全不同。（異常將在一個恰當的異常處理程序中得到解決，它的位置可能離異常被拋出的地方很遠，也可能會跨越方法呼叫堆疊的許多層級。）

此外，能夠拋出任意類型的 **Throwable** 物件，它是異常類型的根類。通常，對於不同類型的錯誤，要拋出相應的異常。錯誤訊息可以儲存在異常物件內部或者用異常類的名稱來暗示。上一層環境透過這些訊息來決定如何處理異常。（通常，唯一的訊息只有異常的類型名，而在異常物件內部沒有任何有意義的訊息。）

## 異常捕獲

要明白異常是如何被捕獲的，必須首先理解監控區域（guarded region）的概念。它是一段可能產生異常的程式碼，並且後面跟著處理這些異常的程式碼。

### try 語句塊

如果在方法內部拋出了異常（或者在方法內部呼叫的其他方法拋出了異常），這個方法將在拋出異常的過程中結束。要是不希望方法就此結束，可以在方法內設定一個特殊的塊來捕獲異常。因為在這個塊裡“嘗試”各種（可能產生異常的）方法呼叫，所以稱為 try 塊。它是跟在 try 關鍵字之後的普通程式塊：

```java
try {
    // Code that might generate exceptions
}
```

對於不支援異常處理的程式語言，要想仔細檢查錯誤，就得在每個方法呼叫的前後加上設定和錯誤檢查的程式碼，甚至在每次呼叫同一方法時也得這麼做。有了異常處理機制，可以把所有動作都放在 try 塊裡，然後只需在一個地方就可以捕獲所有異常。這意味著你的程式碼將更容易編寫和閱讀，因為程式碼的意圖和錯誤檢查不是混淆在一起的。

### 異常處理程序

當然，拋出的異常必須在某處得到處理。這個“地點”就是異常處理程序，而且針對每個要捕獲的異常，得準備相應的處理程序。異常處理程序緊跟在 try 塊之後，以關鍵字 catch 表示：

```java
try {
    // Code that might generate exceptions
} catch(Type1 id1) {
    // Handle exceptions of Type1
} catch(Type2 id2) {
    // Handle exceptions of Type2
} catch(Type3 id3) {
    // Handle exceptions of Type3
}
// etc.
```

每個 catch 子句（異常處理程序）看起來就像是接收且僅接收一個特殊類型的參數的方法。可以在處理程序的內部使用標識符（id1，id2 等等），這與方法參數的使用很相似。有時可能用不到標識符，因為異常的類型已經給了你足夠的訊息來對異常進行處理，但標識符並不可以省略。

異常處理程序必須緊跟在 try 塊之後。當異常被拋出時，異常處理機制將負責搜尋參數與異常類型相匹配的第一個處理程序。然後進入 catch 子句執行，此時認為異常得到了處理。一旦 catch 子句結束，則處理程序的尋找過程結束。注意，只有匹配的 catch 子句才能得到執行；這與 switch 語句不同，switch 語句需要在每一個 case 後面跟一個 break，以避免執行後續的 case 子句。

注意在 try 塊的內部，許多不同的方法呼叫可能會產生類型相同的異常，而你只需要提供一個針對此類型的異常處理程序。

### 終止與復原

異常處理理論上有兩種基本模型。Java 支援終止模型（它是 Java 和 C++所支援的模型）。在這種模型中，將假設錯誤非常嚴重，以至於程式無法返回到異常發生的地方繼續執行。一旦異常被拋出，就表明錯誤已無法挽回，也不能回來繼續執行。

另一種稱為復原模型。意思是異常處理程序的工作是修正錯誤，然後重新嘗試呼叫出問題的方法，並認為第二次能成功。對於復原模型，通常希望異常被處理之後能繼續處理程序。如果想要用 Java 實現類似復原的行為，那麼在遇見錯誤時就不能拋出異常，而是呼叫方法來修正該錯誤。或者，把 try 塊放在 while 循環裡，這樣就不斷地進入 try 塊，直到得到滿意的結果。

在過去，使用支援復原模型異常處理的作業系統的程式設計師們最終還是轉向使用類似“終止模型”的程式碼，並且忽略復原行為。所以雖然復原模型開始顯得很吸引人，但不是很實用。其中的主要原因可能是它所導致的耦合：復原性的處理程序需要了解異常拋出的地點，這勢必要包含依賴於拋出位置的非通用性程式碼。這增加了程式碼編寫和維護的困難，對於異常可能會從許多地方拋出的大型程式來說，更是如此。

<!-- Creating Your Own Exceptions -->

## 自訂異常

不必拘泥於 Java 已有的異常類型。Java異常體系不可能預見你將報告的所有錯誤，所以你可以建立自己的異常類，來表示你的程式中可能遇到的問題。

要自己定義異常類，必須從已有的異常類繼承，最好是選擇意思相近的異常類繼承（不過這樣的異常並不容易找）。建立新的異常類型最簡單的方法就是讓編譯器為你產生無參構造器，所以這幾乎不用寫多少程式碼：

```java
// exceptions/InheritingExceptions.java
// Creating your own exceptions
class SimpleException extends Exception {}

public class InheritingExceptions {
    public void f() throws SimpleException {
        System.out.println(
                "Throw SimpleException from f()");
        throw new SimpleException();
    }
    public static void main(String[] args) {
        InheritingExceptions sed =
                new InheritingExceptions();
        try {
            sed.f();
        } catch(SimpleException e) {
            System.out.println("Caught it!");
        }
    }
}
```

輸出為：

```
Throw SimpleException from f()
Caught it!
```

編譯器建立了無參構造器，它將自動呼叫基類的無參構造器。本例中不會得到像 SimpleException(String) 這樣的構造器，這種構造器也不實用。你將看到，對異常來說，最重要的部分就是類名，所以本例中建立的異常類在大多數情況下已經夠用了。

本例的結果被顯示在控制台。你也可以透過寫入 System.err 而將錯誤發送給標準錯誤流。通常這比把錯誤訊息輸出到 System.out 要好，因為 System.out 也許會被重定向。如果把結果送到 System.err，它就不會隨 System.out 一起被重定向，所以使用者就更容易注意到它。

你也可以為異常類建立一個接受字串參數的構造器：

```java
// exceptions/FullConstructors.java
class MyException extends Exception {
    MyException() {}
    MyException(String msg) { super(msg); }
}
public class FullConstructors {
    public static void f() throws MyException {
        System.out.println("Throwing MyException from f()");
        throw new MyException();
    }
    public static void g() throws MyException {
        System.out.println("Throwing MyException from g()");
        throw new MyException("Originated in g()");
    }
    public static void main(String[] args) {
        try {
            f();
        } catch(MyException e) {
            e.printStackTrace(System.out);
        }
        try {
            g();
        } catch(MyException e) {
            e.printStackTrace(System.out);
        }
    }
}
```

輸出為：

```
Throwing MyException from f()
MyException
    at FullConstructors.f(FullConstructors.java:11)
    at FullConstructors.main(FullConstructors.java:19)
Throwing MyException from g()
MyException: Originated in g()
    at FullConstructors.g(FullConstructors.java:15)
    at FullConstructors.main(FullConstructors.java:24)
```

新增的程式碼非常簡短：兩個構造器定義了 MyException 類型物件的建立方式。對於第二個構造器，使用 super 關鍵字明確呼叫了其基類構造器，它接受一個字串作為參數。

在異常處理程序中，呼叫了在 Throwable 類聲明（Exception 即從此類繼承）的 printStackTrace() 方法。就像從輸出中看到的，它將列印“從方法呼叫處直到異常拋出處”的方法呼叫序列。這裡，訊息被發送到了 System.out，並自動地被捕獲和顯示在輸出中。但是，如果呼叫預設版本：

```java
e.printStackTrace();
```

訊息就會被輸出到標準錯誤流。

### 異常與記錄日誌

你可能還想使用 java.util.logging 工具將輸出記錄到日誌中。基本的日誌記錄功能還是相當簡單易懂的：

```java
// exceptions/LoggingExceptions.java
// An exception that reports through a Logger
// {ErrorOutputExpected}
import java.util.logging.*;
import java.io.*;
class LoggingException extends Exception {
    private static Logger logger =
            Logger.getLogger("LoggingException");
    LoggingException() {
        StringWriter trace = new StringWriter();
        printStackTrace(new PrintWriter(trace));
        logger.severe(trace.toString());
    }
}
public class LoggingExceptions {
    public static void main(String[] args) {
        try {
            throw new LoggingException();
        } catch(LoggingException e) {
            System.err.println("Caught " + e);
        }
        try {
            throw new LoggingException();
        } catch(LoggingException e) {
            System.err.println("Caught " + e);
        }
    }
}
```

輸出為：

```
___[ Error Output ]___
May 09, 2017 6:07:17 AM LoggingException <init>
SEVERE: LoggingException
at
LoggingExceptions.main(LoggingExceptions.java:20)
Caught LoggingException
May 09, 2017 6:07:17 AM LoggingException <init>
SEVERE: LoggingException
at
LoggingExceptions.main(LoggingExceptions.java:25)
Caught LoggingException
```

靜態的 Logger.getLogger() 方法建立了一個 String 參數相關聯的 Logger 物件（通常與錯誤相關的包名和類名），這個 Logger 物件會將其輸出發送到 System.err。向 Logger 寫入的最簡單方式就是直接呼叫與日誌記錄消息的級別相關聯的方法，這裡使用的是 severe()。為了產生日誌記錄消息，我們欲獲取異常拋出處的堆疊軌跡，但是 printStackTrace() 不會預設地產生字串。為了獲取字串，我們需要使用重載的 printStackTrace() 方法，它接受一個 java.io.PrintWriter 物件作為參數（PrintWriter 會在 [附錄：I/O 流 ](./Appendix-IO-Streams.md) 一章詳細介紹）。如果我們將一個 java.io.StringWriter 物件傳遞給這個 PrintWriter 的構造器，那麼透過呼叫 toString() 方法，就可以將輸出抽取為一個 String。

儘管由於 LoggingException 將所有記錄日誌的基礎設施都構建在異常自身中，使得它所使用的方式非常方便，並因此不需要用戶端程式設計師的干預就可以自動執行，但是更常見的情形是我們需要捕獲和記錄其他人編寫的異常，因此我們必須在異常處理程序中生成日誌消息；

```java
// exceptions/LoggingExceptions2.java
// Logging caught exceptions
// {ErrorOutputExpected}
import java.util.logging.*;
import java.io.*;
public class LoggingExceptions2 {
    private static Logger logger =
            Logger.getLogger("LoggingExceptions2");
    static void logException(Exception e) {
        StringWriter trace = new StringWriter();
        e.printStackTrace(new PrintWriter(trace));
        logger.severe(trace.toString());
    }
    public static void main(String[] args) {
        try {
            throw new NullPointerException();
        } catch(NullPointerException e) {
            logException(e);
        }
    }
}
```

輸出結果為：

```
___[ Error Output ]___
May 09, 2017 6:07:17 AM LoggingExceptions2 logException
SEVERE: java.lang.NullPointerException
at
LoggingExceptions2.main(LoggingExceptions2.java:17)
```

還可以更進一步自訂異常，比如加入額外的構造器和成員：

```java
// exceptions/ExtraFeatures.java
// Further embellishment of exception classes
class MyException2 extends Exception {
    private int x;
    MyException2() {}
    MyException2(String msg) { super(msg); }
    MyException2(String msg, int x) {
        super(msg);
        this.x = x;
    }
    public int val() { return x; }
    @Override
    public String getMessage() {
        return "Detail Message: "+ x
                + " "+ super.getMessage();
    }
}
public class ExtraFeatures {
    public static void f() throws MyException2 {
        System.out.println(
                "Throwing MyException2 from f()");
        throw new MyException2();
    }
    public static void g() throws MyException2 {
        System.out.println(
                "Throwing MyException2 from g()");
        throw new MyException2("Originated in g()");
    }
    public static void h() throws MyException2 {
        System.out.println(
                "Throwing MyException2 from h()");
        throw new MyException2("Originated in h()", 47);
    }
    public static void main(String[] args) {
        try {
            f();
        } catch(MyException2 e) {
            e.printStackTrace(System.out);
        }
        try {
            g();
        } catch(MyException2 e) {
            e.printStackTrace(System.out);
        }
        try {
            h();
        } catch(MyException2 e) {
            e.printStackTrace(System.out);
            System.out.println("e.val() = " + e.val());
        }
    }
}
```

輸出為：

```
Throwing MyException2 from f()
MyException2: Detail Message: 0 null
at ExtraFeatures.f(ExtraFeatures.java:24)
at ExtraFeatures.main(ExtraFeatures.java:38)
Throwing MyException2 from g()
MyException2: Detail Message: 0 Originated in g()
at ExtraFeatures.g(ExtraFeatures.java:29)
at ExtraFeatures.main(ExtraFeatures.java:43)
Throwing MyException2 from h()
MyException2: Detail Message: 47 Originated in h()
at ExtraFeatures.h(ExtraFeatures.java:34)
at ExtraFeatures.main(ExtraFeatures.java:48)
e.val() = 47
```

新的異常添加了欄位 x 以及設定 x 值的構造器和讀取資料的方法。此外，還覆蓋了 Throwable.
getMessage() 方法，以產生更詳細的訊息。對於異常類來說，getMessage() 方法有點類似於 toString() 方法。

既然異常也是物件的一種，所以可以繼續修改這個異常類，以得到更強的功能。但要記住，使用程式包的用戶端程式設計師可能僅僅只是查看一下拋出的異常類型，其他的就不管了（大多數 Java 庫裡的異常都是這麼用的），所以對異常所添加的其他功能也許根本用不上。

## 異常聲明

Java 鼓勵人們把方法可能會拋出的異常告知使用此方法的用戶端程式設計師。這是種優雅的做法，它使得呼叫者能確切知道寫什麼樣的程式碼可以捕獲所有潛在的異常。當然，如果提供了原始碼，用戶端程式設計師可以在原始碼中尋找 throw 語句來獲知相關訊息，然而程式庫通常並不與原始碼一起發布。為了預防這樣的問題，Java 提供了相應的語法（並強制使用這個語法），使你能以禮貌的方式告知用戶端程式設計師某個方法可能會拋出的異常類型，然後用戶端程式設計師就可以進行相應的處理。這就是異常說明，它屬於方法聲明的一部分，緊跟在形式參數列表之後。

異常說明使用了附加的關鍵字 throws，後面接一個所有潛在異常類型的列表，所以方法定義可能看起來像這樣：

```java
void f() throws TooBig, TooSmall, DivZero { // ...
```

但是，要是這樣寫：

```java
void f() { // ...
```

就表示此方法不會拋出任何異常（除了從 RuntimeException 繼承的異常，它們可以在沒有異常說明的情況下被拋出，這些將在後面進行討論）。

程式碼必須與異常說明保持一致。如果方法裡的程式碼產生了異常卻沒有進行處理，編譯器會發現這個問題並提醒你：要嘛處理這個異常，要嘛就在異常說明中表明此方法將產生異常。透過這種自頂向下強制執行的異常說明機制，Java 在編譯時就可以保證一定水平的異常正確性。

不過還是有個能“作弊”的地方：可以聲明方法將拋出異常，實際上卻不拋出。編譯器相信了這個聲明，並強制此方法的使用者像真的拋出異常那樣使用這個方法。這樣做的好處是，為異常先占個位子，以後就可以拋出這種異常而不用修改已有的程式碼。在定義抽象基類和介面時這種能力很重要，這樣衍生類或介面實現就能夠拋出這些預先聲明的異常。

這種在編譯時被強制檢查的異常稱為被檢查的異常。

## 捕獲所有異常

可以只寫一個異常處理程序來捕獲所有類型的異常。透過捕獲異常類型的基類 Exception，就可以做到這一點（事實上還有其他的基類，但 Exception 是所有編程行為相關的基類）：

```java
catch(Exception e) {
    System.out.println("Caught an exception");
}
```

這將捕獲所有異常，所以最好把它放在處理程序列表的末尾，以防它搶在其他處理程序之前先把異常捕獲了。

因為 Exception 是與編程有關的所有異常類的基類，所以它不會含有太多具體的訊息，不過可以呼叫它從其基類 Throwable 繼承的方法：

```java
String getMessage()
String getLocalizedMessage()
```

用來獲取詳細訊息，或用本機語言表示的詳細訊息。

```java
String toString()
```

返回對 Throwable 的簡單描述，要是有詳細訊息的話，也會把它包含在內。

```java
void printStackTrace()
void printStackTrace(PrintStream)
void printStackTrace(java.io.PrintWriter)
```

列印 Throwable 和 Throwable 的呼叫堆疊軌跡。呼叫堆疊顯示了“把你帶到異常拋出地點”的方法呼叫序列。其中第一個版本輸出到標準錯誤，後兩個版本允許選擇要輸出的流（在[附錄 I/O 流 ]() 中，你將會理解為什麼有兩種不同的流）。

```java
Throwable fillInStackTrace()
```

用於在 Throwable 物件的內部記錄堆疊幀的目前狀態。這在程式重新拋出錯誤或異常（很快就會講到）時很有用。

此外，也可以使用 Throwable 從其基類 Object（也是所有類的基類）繼承的方法。對於異常來說，getClass() 也許是個很好用的方法，它將返回一個表示此物件類型的物件。然後可以使用 getName() 方法查詢這個 Class 物件包含包訊息的名稱，或者使用只產生類名稱的 getSimpleName() 方法。

下面的例子示範了如何使用 Exception 類型的方法：

```java
// exceptions/ExceptionMethods.java
// Demonstrating the Exception Methods
public class ExceptionMethods {
    public static void main(String[] args) {
        try {
            throw new Exception("My Exception");
        } catch(Exception e) {
            System.out.println("Caught Exception");
            System.out.println(
                    "getMessage():" + e.getMessage());
            System.out.println("getLocalizedMessage():" +
                    e.getLocalizedMessage());
            System.out.println("toString():" + e);
            System.out.println("printStackTrace():");
            e.printStackTrace(System.out);
        }
    }
}
```

輸出為：

```java
Caught Exception
getMessage():My Exception
getLocalizedMessage():My Exception
toString():java.lang.Exception: My Exception
printStackTrace():
java.lang.Exception: My Exception
at ExceptionMethods.main(ExceptionMethods.java:7)
```

可以發現每個方法都比前一個提供了更多的訊息一一實際上它們每一個都是前一個的超集。

### 多重捕獲

如果有一組具有相同基類的異常，你想使用同一方式進行捕獲，那你直接 catch 它們的基類型。但是，如果這些異常沒有共同的基類型，在 Java 7 之前，你必須為每一個類型編寫一個 catch：

```java
// exceptions/SameHandler.java
class EBase1 extends Exception {}
class Except1 extends EBase1 {}
class EBase2 extends Exception {}
class Except2 extends EBase2 {}
class EBase3 extends Exception {}
class Except3 extends EBase3 {}
class EBase4 extends Exception {}
class Except4 extends EBase4 {}

public class SameHandler {
    void x() throws Except1, Except2, Except3, Except4 {}
    void process() {}
    void f() {
        try {
            x();
        } catch(Except1 e) {
            process();
        } catch(Except2 e) {
            process();
        } catch(Except3 e) {
            process();
        } catch(Except4 e) {
            process();
        }
    }
}
```

通過 Java 7 的多重捕獲機制，你可以使用“或”將不同類型的異常組合起來，只需要一行 catch 語句：

```java
// exceptions/MultiCatch.java
public class MultiCatch {
    void x() throws Except1, Except2, Except3, Except4 {}
    void process() {}
    void f() {
        try {
            x();
        } catch(Except1 | Except2 | Except3 | Except4 e) {
            process();
        }
    }
}
```

或者以其他的組合方式：

```java
// exceptions/MultiCatch2.java
public class MultiCatch2 {
    void x() throws Except1, Except2, Except3, Except4 {}
    void process1() {}
    void process2() {}
    void f() {
        try {
            x();
        } catch(Except1 | Except2 e) {
            process1();
        } catch(Except3 | Except4 e) {
            process2();
        }
    }
}
```

這對書寫更整潔的程式碼很有幫助。

### 堆疊軌跡

printStackTrace() 方法所提供的訊息可以透過 getStackTrace() 方法來直接訪問，這個方法將返回一個由堆疊軌跡中的元素所構成的陣列，其中每一個元素都表示堆疊中的一楨。元素 0 是堆疊頂元素，並且是呼叫序列中的最後一個方法呼叫（這個 Throwable 被建立和拋出之處）。陣列中的最後一個元素和堆疊底是呼叫序列中的第一個方法呼叫。下面的程式是一個簡單的示範範例：

```java
// exceptions/WhoCalled.java
// Programmatic access to stack trace information
public class WhoCalled {
    static void f() {
// Generate an exception to fill in the stack trace
        try {
            throw new Exception();
        } catch(Exception e) {
            for(StackTraceElement ste : e.getStackTrace())
                System.out.println(ste.getMethodName());
        }
    }
    static void g() { f(); }
    static void h() { g(); }
    public static void main(String[] args) {
        f();
        System.out.println("*******");
        g();
        System.out.println("*******");
        h();
    }
}
```

輸出為：

```
f
main
*******
f
g
main
*******
f
g
h
main
```

這裡，我們只列印了方法名，但實際上還可以列印整個 StackTraceElement，它包含其他附加的訊息。

### 重新拋出異常

有時希望把剛捕獲的異常重新拋出，尤其是在使用 Exception 捕獲所有異常的時候。既然已經得到了對目前異常物件的引用，可以直接把它重新拋出：

```java
catch(Exception e) {
    System.out.println("An exception was thrown");
    throw e;
}
```

重拋異常會把異常拋給上一級環境中的異常處理程序，同一個 try 塊的後續 catch 子句將被忽略。此外，異常物件的所有訊息都得以保持，所以高一級環境中捕獲此異常的處理程序可以從這個異常物件中得到所有訊息。

如果只是把目前異常物件重新拋出，那麼 printStackTrace() 方法顯示的將是原來異常拋出點的呼叫堆疊訊息，而並非重新拋出點的訊息。要想更新這個訊息，可以呼叫 fillInStackTrace() 方法，這將返回一個 Throwable 物件，它是透過把目前呼叫堆疊訊息填入原來那個異常物件而建立的，就像這樣：

```java
// exceptions/Rethrowing.java
// Demonstrating fillInStackTrace()
public class Rethrowing {
    public static void f() throws Exception {
        System.out.println(
                "originating the exception in f()");
        throw new Exception("thrown from f()");
    }
    public static void g() throws Exception {
        try {
            f();
        } catch(Exception e) {
            System.out.println(
                    "Inside g(), e.printStackTrace()");
            e.printStackTrace(System.out);
            throw e;
        }
    }
    public static void h() throws Exception {
        try {
            f();
        } catch(Exception e) {
            System.out.println(
                    "Inside h(), e.printStackTrace()");
            e.printStackTrace(System.out);
            throw (Exception)e.fillInStackTrace();
        }
    }
    public static void main(String[] args) {
        try {
            g();
        } catch(Exception e) {
            System.out.println("main: printStackTrace()");
            e.printStackTrace(System.out);
        }
        try {
            h();
        } catch(Exception e) {
            System.out.println("main: printStackTrace()");
            e.printStackTrace(System.out);
        }
    }
}
```

輸出為：

```java
originating the exception in f()
Inside g(), e.printStackTrace()
java.lang.Exception: thrown from f()
at Rethrowing.f(Rethrowing.java:8)
at Rethrowing.g(Rethrowing.java:12)
at Rethrowing.main(Rethrowing.java:32)
main: printStackTrace()
java.lang.Exception: thrown from f()
at Rethrowing.f(Rethrowing.java:8)
at Rethrowing.g(Rethrowing.java:12)
at Rethrowing.main(Rethrowing.java:32)
originating the exception in f()
Inside h(), e.printStackTrace()
java.lang.Exception: thrown from f()
at Rethrowing.f(Rethrowing.java:8)
at Rethrowing.h(Rethrowing.java:22)
at Rethrowing.main(Rethrowing.java:38)
main: printStackTrace()
java.lang.Exception: thrown from f()
at Rethrowing.h(Rethrowing.java:27)
at Rethrowing.main(Rethrowing.java:38)
```

呼叫 fillInStackTrace() 的那一行就成了異常的新發生地了。

有可能在捕獲異常之後拋出另一種異常。這麼做的話，得到的效果類似於使用 fillInStackTrace()，有關原來異常發生點的訊息會遺失，剩下的是與新的拋出點有關的訊息：

```java
// exceptions/RethrowNew.java
// Rethrow a different object from the one you caught
class OneException extends Exception {
    OneException(String s) { super(s); }
}
class TwoException extends Exception {
    TwoException(String s) { super(s); }
}
public class RethrowNew {
    public static void f() throws OneException {
        System.out.println(
                "originating the exception in f()");
        throw new OneException("thrown from f()");
    }
    public static void main(String[] args) {
        try {
            try {
                f();
            } catch(OneException e) {
                System.out.println(
                        "Caught in inner try, e.printStackTrace()");
                e.printStackTrace(System.out);
                throw new TwoException("from inner try");
            }
        } catch(TwoException e) {
            System.out.println(
                    "Caught in outer try, e.printStackTrace()");
            e.printStackTrace(System.out);
        }
    }
}
```

輸出為：

```java
originating the exception in f()
Caught in inner try, e.printStackTrace()
OneException: thrown from f()
at RethrowNew.f(RethrowNew.java:16)
at RethrowNew.main(RethrowNew.java:21)
Caught in outer try, e.printStackTrace()
TwoException: from inner try
at RethrowNew.main(RethrowNew.java:26)
```

最後那個異常僅知道自己來自 main()，而對 f() 一無所知。

永遠不必為清理前一個異常物件而擔心，或者說為異常物件的清理而擔心。它們都是用 new 在堆上建立的物件，所以垃圾回收器會自動把它們清理掉。

### 精準的重新拋出異常

在 Java 7 之前，如果捕捉到一個異常，重新拋出的異常類型只能與原異常完全相同。這導致程式碼不精確，Java 7修復了這個問題。所以在 Java 7 之前，這無法編譯：

```java
class BaseException extends Exception {}
class DerivedException extends BaseException {}

public class PreciseRethrow {
    void catcher() throws DerivedException {
        try {
            throw new DerivedException();
        } catch(BaseException e) {
            throw e;
        }
    }
}
```

因為 catch 捕獲了一個 BaseException，編譯器強迫你聲明 catcher() 拋出 BaseException，即使它實際上拋出了更具體的 DerivedException。從 Java 7 開始，這段程式碼就可以編譯，這是一個很小但很有用的修復。

### 異常鏈

常常會想要在捕獲一個異常後拋出另一個異常，並且希望把原始異常的訊息儲存下來，這被稱為異常鏈。在 JDK1.4 以前，程式設計師必須自己編寫程式碼來儲存原始異常的訊息。現在所有 Throwable 的子類在構造器中都可以接受一個 cause（因由）物件作為參數。這個 cause 就用來表示原始異常，這樣透過把原始異常傳遞給新的異常，使得即使在目前位置建立並拋出了新的異常，也能透過這個異常鏈追蹤到異常最初發生的位置。

有趣的是，在 Throwable 的子類中，只有三種基本的異常類提供了帶 cause 參數的構造器。它們是 Error（用於 Java 虛擬機報告系統錯誤）、Exception 以及 RuntimeException。如果要把其他類型的異常連結起來，應該使用 initCause() 方法而不是構造器。

下面的例子能讓你在執行時動態地向 DynamicFields 物件添加欄位：

```java
// exceptions/DynamicFields.java
// A Class that dynamically adds fields to itself to
// demonstrate exception chaining
class DynamicFieldsException extends Exception {}
public class DynamicFields {
    private Object[][] fields;
    public DynamicFields(int initialSize) {
        fields = new Object[initialSize][2];
        for(int i = 0; i < initialSize; i++)
            fields[i] = new Object[] { null, null };
    }
    @Override
    public String toString() {
        StringBuilder result = new StringBuilder();
        for(Object[] obj : fields) {
            result.append(obj[0]);
            result.append(": ");
            result.append(obj[1]);
            result.append("\n");
        }
        return result.toString();
    }
    private int hasField(String id) {
        for(int i = 0; i < fields.length; i++)
            if(id.equals(fields[i][0]))
                return i;
        return -1;
    }
    private int getFieldNumber(String id)
            throws NoSuchFieldException {
        int fieldNum = hasField(id);
        if(fieldNum == -1)
            throw new NoSuchFieldException();
        return fieldNum;
    }
    private int makeField(String id) {
        for(int i = 0; i < fields.length; i++)
            if(fields[i][0] == null) {
                fields[i][0] = id;
                return i;
            }
// No empty fields. Add one:
        Object[][] tmp = new Object[fields.length + 1][2];
        for(int i = 0; i < fields.length; i++)
            tmp[i] = fields[i];
        for(int i = fields.length; i < tmp.length; i++)
            tmp[i] = new Object[] { null, null };
        fields = tmp;
// Recursive call with expanded fields:
        return makeField(id);
    }
    public Object
    getField(String id) throws NoSuchFieldException {
        return fields[getFieldNumber(id)][1];
    }
    public Object setField(String id, Object value)
            throws DynamicFieldsException {
        if(value == null) {
// Most exceptions don't have a "cause"
// constructor. In these cases you must use
// initCause(), available in all
// Throwable subclasses.
            DynamicFieldsException dfe =
                    new DynamicFieldsException();
            dfe.initCause(new NullPointerException());
            throw dfe;
        }
        int fieldNumber = hasField(id);
        if(fieldNumber == -1)
            fieldNumber = makeField(id);
        Object result = null;
        try {
            result = getField(id); // Get old value
        } catch(NoSuchFieldException e) {
// Use constructor that takes "cause":
            throw new RuntimeException(e);
        }
        fields[fieldNumber][1] = value;
        return result;
    }
    public static void main(String[] args) {
        DynamicFields df = new DynamicFields(3);
        System.out.println(df);
        try {
            df.setField("d", "A value for d");
            df.setField("number", 47);
            df.setField("number2", 48);
            System.out.println(df);
            df.setField("d", "A new value for d");
            df.setField("number3", 11);
            System.out.println("df: " + df);
            System.out.println("df.getField(\"d\") : "
                    + df.getField("d"));
            Object field =
                    df.setField("d", null); // Exception
        } catch(NoSuchFieldException |
                DynamicFieldsException e) {
            e.printStackTrace(System.out);
        }
    }
}
```

輸出為：

```java
null: null
null: null
null: null
d: A value for d
number: 47
number2: 48
df: d: A new value for d
number: 47
number2: 48
number3: 11

df.getField("d") : A new value for d
DynamicFieldsException
at DynamicFields.setField(DynamicFields.java:65)
at DynamicFields.
Caused by: java.lang.NullPointerException
at DynamicFields.setField(DynamicFields.java:67)
... 1 more
```

每個 DynamicFields 物件都含有一個陣列，其元素是“成對的物件”。第一個物件表示欄位標識符（一個字串），第二個表示欄位值，值的類型可以是除基本類型外的任意類型。當建立物件的時候，要合理估計一下需要多少欄位。當呼叫 setField() 方法的時候，它將試圖透過標識修改已有欄位值，否則就建一個新的欄位，並把值放入。如果空間不夠了，將建立一個更長的陣列，並把原來陣列的元素複製進去。如果你試圖為欄位設定一個空值，將拋出一個 DynamicFieldsException 異常，它是透過使用 initCause() 方法把 NullPointerException 物件插入而建立的。

至於返回值，setField() 將用 getField() 方法把此位置的舊值取出，這個操作可能會拋出 NoSuchFieldException 異常。如果用戶端程式設計師呼叫了 getField() 方法，那麼他就有責任處理這個可能拋出的 NoSuchFieldException 異常，但如果異常是從 setField() 方法裡拋出的，這種情況將被視為編程錯誤，所以就使用接受 cause 參數的構造器把 NoSuchFieldException 異常轉換為 RuntimeException 異常。

你會注意到，toString() 方法使用了一個 StringBuilder 來建立其結果。在 [字串](./Strings.md) 這章中你將會了解到更多的關於 StringBuilder 的知識，但是只要你編寫設計循環的 toString() 方法，通常都會想使用它，就像本例一樣。

`main()` 方法中的 catch 子句看起來不同 - 它使用相同的子句處理兩種不同類型的異常，這兩種不同的異常透過“或（|）”符號結合起來。 Java 7 的這項功能有助於減少程式碼重複，並使你更容易指定要捕獲的確切類型，而不是簡單地捕獲一個基類型。你可以透過這種方式組合多種異常類型。

<!-- Standard Java Exceptions -->

## Java 標準異常

Throwable 這個 Java 類被用來表示任何可以作為異常被拋出的類。Throwable 物件可分為兩種類型（指從 Throwable 繼承而得到的類型）：Error 用來表示編譯時和系統錯誤（除特殊情況外，一般不用你關心）；Exception 是可以被拋出的基本類型，在 Java 類庫、使用者方法以及執行時故障中都可能拋出 Exception 型異常。所以 Java 程式設計師關心的基類型通常是 Exception。要想對異常有全面的了解，最好去瀏覽一下 HTML 格式的 Java 文件（可以從 java.sun.com 下載）。為了對不同的異常有個感性的認識，這麼做是值得的。但很快你就會發現，這些異常除了名稱外其實都差不多。同時，Java 中異常的數目在持續增加，所以在書中簡單羅列它們毫無意義。所使用的第三方類庫也可能會有自己的異常。對異常來說，關鍵是理解概念以及如何使用。

基本理念是用異常的名稱代表發生的問題。異常的名稱應該可以望文知意。異常並非全是在 java.lang 包裡定義的；有些異常是用來支援其他像 util、net 和 io 這樣的程式包，這些異常可以透過它們的完整名稱或者從它們的父類中看出端倪。比如，所有的輸入/輸出異常都是從 java.io.IOException 繼承而來的。

### 特例：RuntimeException

在本章的第一個例子中：

```java
if(t == null)
    throw new NullPointerException();
```

如果必須對傳遞給方法的每個引用都檢查其是否為 null（因為無法確定呼叫者是否傳入了非法引用），這聽起來著實嚇人。幸運的是，這不必由你親自來做，它屬於 Java 的標準執行時檢測的一部分。如果對 null 引用進行呼叫，Java 會自動拋出 NullPointerException 異常，所以上述程式碼是多餘的，儘管你也許想要執行其他的檢查以確保 NullPointerException 不會出現。

屬於執行時異常的類型有很多，它們被 java 自動拋出，所以不必在異常說明中把它們列出來。非常方便的是，透過將這些異常設定為 `RuntimeException`的子類而把它們歸類起來，這是繼承的一個絕佳例子：建立具有相同特徵和行為的一組類型。

RuntimeException 代表的是編程錯誤：

1. 無法預料的錯誤。比如從你控制範圍之外傳遞進來的 null 引用。
2. 作為程式設計師，應該在程式碼中進行檢查的錯誤。（比如對於 ArrayIndexOutOfBoundsException，就得注意一下陣列的大小了。）在一個地方發生的異常，常常會在另一個地方導致錯誤。

在這些情況下使用異常很有好處，它們能給除錯帶來便利。

如果不捕獲這種類型的異常會發生什麼事呢？因為編譯器沒有在這個問題上對異常說明進行強制檢查，RuntimeException 類型的異常也許會穿越所有的執行路徑直達 main() 方法，而不會被捕獲。要明白到底發生了什麼事，可以試試下面的例子：

```java
// exceptions/NeverCaught.java
// Ignoring RuntimeExceptions
// {ThrowsException}
public class NeverCaught {
    static void f() {
        throw new RuntimeException("From f()");
    }
    static void g() {
        f();
    }
    public static void main(String[] args) {
        g();
    }
}
```

輸出結果為：

```java
___[ Error Output ]___
Exception in thread "main" java.lang.RuntimeException:
From f()
at NeverCaught.f(NeverCaught.java:7)
at NeverCaught.g(NeverCaught.java:10)
at NeverCaught.main(NeverCaught.java:13)
```

如果 RuntimeException 沒有被捕獲而直達 main()，那麼在程式退出前將呼叫異常的 printStackTrace() 方法。

你會發現，RuntimeException（或任何從它繼承的異常）是一個特例。對於這種異常類型，編譯器不需要異常說明，其輸出被報告給了 System.err。

請務必記住：程式碼中只有 RuntimeException（及其子類）類型的異常可以被忽略，因為編譯器強制要求處理所有受檢查類型的異常。

值得注意的是：不應把 Java 的異常處理機制當成是單一用途的工具。是的，它被設計用來處理一些煩人的執行時錯誤，這些錯誤往往是由程式碼控制能力之外的因素導致的；然而，它對於發現某些編譯器無法檢測到的程式錯誤，也是非常重要的。

<!-- Performing Cleanup with finally -->

## 使用 finally 進行清理

有一些程式碼片段，可能會希望無論 try 塊中的異常是否拋出，它們都能得到執行。這通常適用於記憶體回收之外的情況（因為回收由垃圾回收器完成），為了達到這個效果，可以在異常處理程序後面加上 finally 子句。完整的異常處理程序看起來像這樣：

```java
try {
// The guarded region: Dangerous activities
// that might throw A, B, or C
} catch(A a1) {
// Handler for situation A
} catch(B b1) {
// Handler for situation B
} catch(C c1) {
// Handler for situation C
} finally {
// Activities that happen every time
}
```

為了證明 finally 子句總能執行，可以試試下面這個程式：

```java
// exceptions/FinallyWorks.java
// The finally clause is always executed
class ThreeException extends Exception {}
public class FinallyWorks {
    static int count = 0;
    public static void main(String[] args) {
        while(true) {
            try {
				// Post-increment is zero first time:
                if(count++ == 0)
                    throw new ThreeException();
                System.out.println("No exception");
            } catch(ThreeException e) {
                System.out.println("ThreeException");
            } finally {
                System.out.println("In finally clause");
                if(count == 2) break; // out of "while"
            }
        }
    }
}
```

輸出為：

```
ThreeException
In finally clause
No exception
In finally clause
```

從輸出中發現，無論異常是否被拋出，finally 子句總能被執行。這也為解決 Java 不允許我們回到異常拋出點這一問題，提供了一個思路。如果將 try 塊放在循環裡，就可以設定一種在程式執行前一定會遇到的異常狀況。還可以加入一個 static 類型的計數器或者別的裝置，使循環在結束以前能嘗試一定的次數。這將使程式的健壯性更上一個台階。

### finally 用來做什麼？

對於沒有垃圾回收和解構子自動呼叫機制的語言來說，finally 非常重要。它能使程式設計師保證：無論 try 塊裡發生了什麼事，記憶體總能得到釋放。但 Java 有垃圾回收機制，所以記憶體釋放不再是問題。而且，Java 也沒有解構子可供呼叫。那麼，Java 在什麼情況下才能用到 finally 呢？

當要把除記憶體之外的資源復原到它們的初始狀態時，就要用到 finally 子句。這種需要清理的資源包括：已經打開的文件或網路連接，在螢幕上畫的圖形，甚至可以是外部世界的某個開關，如下面例子所示：

```java
// exceptions/Switch.java
public class Switch {
    private boolean state = false;
    public boolean read() { return state; }
    public void on() {
        state = true;
        System.out.println(this);
    }
    public void off() {
        state = false;
        System.out.println(this);
    }
    @Override
    public String toString() {
        return state ? "on" : "off";
    }
}
// exceptions/OnOffException1.java
public class OnOffException1 extends Exception {}
// exceptions/OnOffException2.java
public class OnOffException2 extends Exception {}
// exceptions/OnOffSwitch.java
// Why use finally?
public class OnOffSwitch {
    private static Switch sw = new Switch();
    public static void f()
            throws OnOffException1, OnOffException2 {}
    public static void main(String[] args) {
        try {
            sw.on();
			// Code that can throw exceptions...
            f();
            sw.off();
        } catch(OnOffException1 e) {
            System.out.println("OnOffException1");
            sw.off();
        } catch(OnOffException2 e) {
            System.out.println("OnOffException2");
            sw.off();
        }
    }
}
```

輸出為：

```
on
off
```

程式的目的是要確保 main() 結束的時候開關必須是關閉的，所以在每個 try 塊和異常處理程序的末尾都加入了對 sw.off() 方法的呼叫。但也可能有這種情況：異常被拋出，但沒被處理程序捕獲，這時 sw.off() 就得不到呼叫。但是有了 finally，只要把 try 塊中的清理程式碼移放在一處即可：

```java
// exceptions/WithFinally.java
// Finally Guarantees cleanup
public class WithFinally {
    static Switch sw = new Switch();
    public static void main(String[] args) {
        try {
            sw.on();
			// Code that can throw exceptions...
            OnOffSwitch.f();
        } catch(OnOffException1 e) {
            System.out.println("OnOffException1");
        } catch(OnOffException2 e) {
            System.out.println("OnOffException2");
        } finally {
            sw.off();
        }
    }
}
```

輸出為：

```java
on
off
```

這裡 sw.off() 被移到一處，並且保證在任何情況下都能得到執行。

甚至在異常沒有被目前的異常處理程序捕獲的情況下，異常處理機制也會在跳到更高一層的異常處理程序之前，執行 finally 子句：

```java
// exceptions/AlwaysFinally.java
// Finally is always executed
class FourException extends Exception {}
public class AlwaysFinally {
    public static void main(String[] args) {
        System.out.println("Entering first try block");
        try {
            System.out.println("Entering second try block");
            try {
                throw new FourException();
            } finally {
                System.out.println("finally in 2nd try block");
            }
        } catch(FourException e) {
            System.out.println(
                    "Caught FourException in 1st try block");
        } finally {
            System.out.println("finally in 1st try block");
        }
    }
}
```

輸出為：

```java
Entering first try block
Entering second try block
finally in 2nd try block
Caught FourException in 1st try block
finally in 1st try block
```

當涉及 break 和 continue 語句的時候，finally 子句也會得到執行。請注意，如果把 finally 子句和帶標籤的 break 及 continue 配合使用，在 Java 裡就沒必要使用 goto 語句了。

### 在 return 中使用 finally

因為 finally 子句總是會執行，所以可以從一個方法內的多個點返回，仍然能保證重要的清理工作會執行：

```java
// exceptions/MultipleReturns.java
public class MultipleReturns {
    public static void f(int i) {
        System.out.println(
                "Initialization that requires cleanup");
        try {
            System.out.println("Point 1");
            if(i == 1) return;
            System.out.println("Point 2");
            if(i == 2) return;
            System.out.println("Point 3");
            if(i == 3) return;
            System.out.println("End");
            return;
        } finally {
            System.out.println("Performing cleanup");
        }
    }
    public static void main(String[] args) {
        for(int i = 1; i <= 4; i++)
            f(i);
    }
}
```

輸出為：

```java
Initialization that requires cleanup
Point 1
Performing cleanup
Initialization that requires cleanup
Point 1
Point 2
Performing cleanup
Initialization that requires cleanup
Point 1
Point 2
Point 3
Performing cleanup
Initialization that requires cleanup
Point 1
Point 2
Point 3
End
Performing cleanup
```

從輸出中可以看出，從何處返回無關緊要，finally 子句永遠會執行。

### 缺憾：異常遺失

遺憾的是，Java 的異常實現也有瑕疵。異常作為程式出錯的標誌，決不應該被忽略，但它還是有可能被輕易地忽略。用某些特殊的方式使用 finally 子句，就會發生這種情況：

```java
// exceptions/LostMessage.java
// How an exception can be lost
class VeryImportantException extends Exception {
    @Override
    public String toString() {
        return "A very important exception!";
    }
}
class HoHumException extends Exception {
    @Override
    public String toString() {
        return "A trivial exception";
    }
}
public class LostMessage {
    void f() throws VeryImportantException {
        throw new VeryImportantException();
    }
    void dispose() throws HoHumException {
        throw new HoHumException();
    }
    public static void main(String[] args) {
        try {
            LostMessage lm = new LostMessage();
            try {
                lm.f();
            } finally {
                lm.dispose();
            }
        } catch(VeryImportantException | HoHumException e) {
            System.out.println(e);
        }
    }
}
```

輸出為：

```
A trivial exception
```

從輸出中可以看到，VeryImportantException 不見了，它被 finally 子句裡的 HoHumException 所取代。這是相當嚴重的缺陷，因為異常可能會以一種比前面例子所示更微妙和難以察覺的方式完全遺失。相比之下，C++把“前一個異常還沒處理就拋出下一個異常”的情形看成是糟糕的程式錯誤。也許在 Java 的未來版本中會修正這個問題（另一方面，要把所有拋出異常的方法，如上例中的 dispose() 方法，全部打包放到 try-catch 子句裡面）。

一種更加簡單的遺失異常的方式是從 finally 子句中返回：

```java
// exceptions/ExceptionSilencer.java
public class ExceptionSilencer {
    public static void main(String[] args) {
        try {
            throw new RuntimeException();
        } finally {
            // Using 'return' inside the finally block
            // will silence any thrown exception.
            return;
        }
    }
}
```

如果執行這個程式，就會看到即使方法裡拋出了異常，它也不會產生任何輸出。

<!-- Exception Restrictions -->

## 異常限制

當覆蓋方法的時候，只能拋出在基類方法的異常說明裡列出的那些異常。這個限制很有用，因為這意味著與基類一起工作的程式碼，也能和匯出類一起正常工作（這是物件導向的基本概念），異常也不例外。

下面例子示範了這種（在編譯時）施加在異常上面的限制：

```java
// exceptions/StormyInning.java
// Overridden methods can throw only the exceptions
// specified in their base-class versions, or exceptions
// derived from the base-class exceptions
class BaseballException extends Exception {}
class Foul extends BaseballException {}
class Strike extends BaseballException {}
abstract class Inning {
    Inning() throws BaseballException {}
    public void event() throws BaseballException {
// Doesn't actually have to throw anything
    }
    public abstract void atBat() throws Strike, Foul;
    public void walk() {} // Throws no checked exceptions
}
class StormException extends Exception {}
class RainedOut extends StormException {}
class PopFoul extends Foul {}
interface Storm {
    void event() throws RainedOut;
    void rainHard() throws RainedOut;
}
public class StormyInning extends Inning implements Storm {
    // OK to add new exceptions for constructors, but you
// must deal with the base constructor exceptions:
    public StormyInning()
            throws RainedOut, BaseballException {}
    public StormyInning(String s)
            throws BaseballException {}
    // Regular methods must conform to base class:
//- void walk() throws PopFoul {} //Compile error
// Interface CANNOT add exceptions to existing
// methods from the base class:
//- public void event() throws RainedOut {}
// If the method doesn't already exist in the
// base class, the exception is OK:
    @Override
    public void rainHard() throws RainedOut {}
    // You can choose to not throw any exceptions,
// even if the base version does:
    @Override
    public void event() {}
    // Overridden methods can throw inherited exceptions:
    @Override
    public void atBat() throws PopFoul {}
    public static void main(String[] args) {
        try {
            StormyInning si = new StormyInning();
            si.atBat();
        } catch(PopFoul e) {
            System.out.println("Pop foul");
        } catch(RainedOut e) {
            System.out.println("Rained out");
        } catch(BaseballException e) {
            System.out.println("Generic baseball exception");
        }
// Strike not thrown in derived version.
        try {
// What happens if you upcast?
            Inning i = new StormyInning();
            i.atBat();
// You must catch the exceptions from the
// base-class version of the method:
        } catch(Strike e) {
            System.out.println("Strike");
        } catch(Foul e) {
            System.out.println("Foul");
        } catch(RainedOut e) {
            System.out.println("Rained out");
        } catch(BaseballException e) {
            System.out.println("Generic baseball exception");
        }
    }
}
```

在 Inning 類中，可以看到構造器和 event() 方法都聲明將拋出異常，但實際上沒有拋出。這種方式使你能強制使用者去捕獲可能在覆蓋後的 event() 版本中增加的異常，所以它很合理。這對於抽象方法同樣成立，比如 atBat()。

介面 Storm 包含了一個在 Inning 中定義的方法 event() 和一個不在 Inning 中定義的方法 rainHard()。這兩個方法都拋出新的異常 RainedOut，如果 StormyInning 類在擴展 Inning 類的同時又實現了 Storm 介面，那麼 Storm 裡的 event() 方法就不能改變在 Inning 中的 event() 方法的異常介面。否則的話，在使用基類的時候就不能判斷是否捕獲了正確的異常，所以這也很合理。當然，如果介面裡定義的方法不是來自於基類，比如 rainHard()，那麼此方法拋出什麼樣的異常都沒有問題。

異常限制對構造器不起作用。你會發現 StormyInning 的構造器可以拋出任何異常，而不必理會基類構造器所拋出的異常。然而，因為基類構造器必須以這樣或那樣的方式被呼叫（這裡預設構造器將自動被呼叫），衍生類構造器的異常說明必須包含基類構造器的異常說明。

衍生類構造器不能捕獲基類構造器拋出的異常。

StormyInning.walk() 不能透過編譯是因為它拋出了一個 Inning.walk() 中沒有聲明的異常。如果編譯器允許這麼做的話，就可以編寫呼叫Inning.walk()卻不處理任何異常的程式碼。 但是當使用 `Inning`衍生類的物件時，就會拋出異常，從而導致程式出現問題。透過強制衍生類遵守基類方法的異常說明，物件的可取代性得到了保證。

覆蓋後的 event() 方法表明，衍生類版的方法可以不拋出任何異常，即使基類版的方法拋出了異常。因為這樣做不會破壞那些假定基類版的方法會拋出異常的程式碼。類似的情況出現在 `atBat()`上，它拋出的異常`PopFoul`是由基類版`atBat()`拋出的`Foul` 異常衍生而來。如果你寫的程式碼同 `Inning` 一起工作，並且呼叫了 `atBat()`的話，那麼肯定能捕獲 `Foul` 。又因為 `PopFoul` 是由 `Foul`衍生而來，因此異常處理程序也能捕獲 `PopFoul`。

最後一個有趣的地方在 `main()`。如果處理的剛好是 Stormylnning 物件的話，編譯器只要求捕獲這個類所拋出的異常。但是如果將它向上轉型成基類型，那麼編譯器就會準確地要求捕獲基類的異常。所有這些限制都是為了能產生更為健壯的異常處理程式碼。

儘管在繼承過程中，編譯器會對異常說明做強制要求，但異常說明本身並不屬於方法類型的一部分，方法類型是由方法的名字與參數的類型組成的。因此，不能基於異常說明來重載方法。此外，一個出現在基類方法的異常說明中的異常，不一定會出現在衍生類方法的異常說明裡。這點同繼承的規則明顯不同，在繼承中，基類的方法必須出現在衍生類裡，換句話說，在繼承和覆蓋的過程中，某個特定方法的“異常說明的介面”不是變大了而是變小了——這恰好和類介面在繼承時的情形相反。

<!-- Constructors -->

## 構造器

有一點很重要，即你要時刻詢問自己“如果異常發生了，所有東西能被正確的清理嗎？"儘管大多數情況下是非常安全的，但涉及構造器時，問題就出現了。構造器會把物件設定成安全的初始狀態，但還會有別的動作，比如打開一個文件，這樣的動作只有在物件使用完畢並且使用者呼叫了特殊的清理方法之後才能得以清理。如果在構造器內拋出了異常，這些清理行為也許就不能正常工作了。這意味著在編寫構造器時要格外細心。

你也許會認為使用 finally 就可以解決問題。但問題並非如此簡單，因為 finally 會每次都執行清理程式碼。如果構造器在其執行過程中半途而廢，也許該物件的某些部分還沒有被成功建立，而這些部分在 finally 子句中卻是要被清理的。

在下面的例子中，建立了一個 InputFile 類，它能打開一個文件並且每次讀取其中的一行。這裡使用了 Java 標準輸入/輸出庫中的 FileReader 和 BufferedReader 類（將在 [附錄：I/O 流 ](./Appendix-IO-Streams.md) 中討論），這些類的基本用法很簡單，你應該很容易明白：

```java
// exceptions/InputFile.java
// Paying attention to exceptions in constructors
import java.io.*;
public class InputFile {
    private BufferedReader in;
    public InputFile(String fname) throws Exception {
        try {
            in = new BufferedReader(new FileReader(fname));
            // Other code that might throw exceptions
        } catch(FileNotFoundException e) {
            System.out.println("Could not open " + fname);
            // Wasn't open, so don't close it
            throw e;
        } catch(Exception e) {
            // All other exceptions must close it
            try {
                in.close();
            } catch(IOException e2) {
                System.out.println("in.close() unsuccessful");
            }
            throw e; // Rethrow
        } finally {
        // Don't close it here!!!
        }
    }
    public String getLine() {
        String s;
        try {
            s = in.readLine();
        } catch(IOException e) {
            throw new RuntimeException("readLine() failed");
        }
        return s;
    }
    public void dispose() {
        try {
            in.close();
            System.out.println("dispose() successful");
        } catch(IOException e2) {
            throw new RuntimeException("in.close() failed");
        }
    }
}
```

InputFile 的構造器接受字串作為參數，該字串表示所要打開的檔案名。在 try 塊中，會使用此檔案名建立 FileReader 物件。FileReader 物件本身用處並不大，但可以用它來建立 BufferedReader 物件。注意，使用 InputFile 的好處之一是把兩步操作合而為一。

如果 FileReader 的構造器失敗了，將拋出 FileNotFoundException 異常。對於這個異常，並不需要關閉文件，因為這個文件還沒有被打開。而任何其他捕獲異常的 catch 子句必須關閉文件，因為在它們捕獲到異常之時，文件已經打開了（當然，如果還有其他方法能拋出 FileNotFoundException，這個方法就顯得有些投機取巧了。這時，通常必須把這些方法分別放到各自的 try 塊裡），close() 方法也可能會拋出異常，所以儘管它已經在另一個 catch 子句塊裡了，還是要再用一層 try-catch，這對 Java 編譯器而言只不過是多了一對花括號。在本機做完處理之後，異常被重新拋出，對於構造器而言這麼做是很合適的，因為你總不希望去誤導呼叫方，讓他認為“這個物件已經建立完畢，可以使用了”。

在本例中，由於 finally 會在每次完成構造器之後都執行一遍，因此它實在不該是呼叫 close() 關閉文件的地方。我們希望文件在 InputFlle 物件的整個生命週期內都處於打開狀態。

getLine() 方法會返回表示文件下一行內容的字串。它呼叫了能拋出異常的 readLine()，但是這個異常已經在方法內得到處理，因此 getLine() 不會拋出任何異常。在設計異常時有一個問題：應該把異常全部放在這一層處理；還是先處理一部分，然後再向上層拋出相同的（或新的）異常；又或者是不做任何處理直接向上層拋出。如果用法恰當的話，直接向上層拋出的確能簡化編程。在這裡，getLine() 方法將異常轉換為 RuntimeException，表示一個編程錯誤。

使用者在不再需要 InputFile 物件時，就必須呼叫 dispose() 方法，這將釋放 BufferedReader 和/或 FileReader 物件所占用的系統資源（比如文件句柄），在使用完 InputFile 物件之前是不會呼叫它的。可能你會考慮把上述功能放到 finalize() 裡面，但我在 [封裝](./Housekeeping.md) 講過，你不知道 finalize() 會不會被呼叫（即使能確定它將被呼叫，也不知道在什麼時候呼叫），這也是 Java 的缺陷：除了記憶體的清理之外，所有的清理都不會自動發生。所以必須告訴用戶端程式設計師，這是他們的責任。

對於在構造階段可能會拋出異常，並且要求清理的類，最安全的使用方式是使用嵌套的 try 子句：

```java
// exceptions/Cleanup.java
// Guaranteeing proper cleanup of a resource
public class Cleanup {
    public static void main(String[] args) {
        try {
            InputFile in = new InputFile("Cleanup.java");
            try {
                String s;
                int i = 1;
                while((s = in.getLine()) != null)
                    ; // Perform line-by-line processing here...
            } catch(Exception e) {
                System.out.println("Caught Exception in main");
                e.printStackTrace(System.out);
            } finally {
                in.dispose();
            }
        } catch(Exception e) {
            System.out.println(
                    "InputFile construction failed");
        }
    }
}
```

輸出為：

```
dispose() successful
```

請仔細觀察這裡的邏輯：對 InputFile 物件的構造在其自己的 try 語句塊中有效，如果構造失敗，將進入外部的 catch 子句，而 dispose() 方法不會被呼叫。但是，如果構造成功，我們肯定想確保物件能夠被清理，因此在構造之後立即建立了一個新的 try 語句塊。執行清理的 finally 與內部的 try 語句塊相關聯。在這種方式中，finally 子句在構造失敗時是不會執行的，而在構造成功時將總是執行。

這種通用的清理慣用法在構造器不拋出任何異常時也應該運用，其基本規則是：在建立需要清理的物件之後，立即進入一個 try-finally 語句塊：

```java
// exceptions/CleanupIdiom.java
// Disposable objects must be followed by a try-finally
class NeedsCleanup { // Construction can't fail
    private static long counter = 1;
    private final long id = counter++;
    public void dispose() {
        System.out.println(
                "NeedsCleanup " + id + " disposed");
    }
}
class ConstructionException extends Exception {}
class NeedsCleanup2 extends NeedsCleanup {
    // Construction can fail:
    NeedsCleanup2() throws ConstructionException {}
}
public class CleanupIdiom {
    public static void main(String[] args) {
        // [1]:
        NeedsCleanup nc1 = new NeedsCleanup();
        try {
        // ...
        } finally {
            nc1.dispose();
        }
        // [2]:
        // If construction cannot fail,
        // you can group objects:
        NeedsCleanup nc2 = new NeedsCleanup();
        NeedsCleanup nc3 = new NeedsCleanup();
        try {
        // ...
        } finally {
            nc3.dispose(); // Reverse order of construction
            nc2.dispose();
        }
        // [3]:
        // If construction can fail you must guard each one:
        try {
            NeedsCleanup2 nc4 = new NeedsCleanup2();
            try {
                NeedsCleanup2 nc5 = new NeedsCleanup2();
                try {
                // ...
                } finally {
                    nc5.dispose();
                }
            } catch(ConstructionException e) { // nc5 const.
                System.out.println(e);
            } finally {
                nc4.dispose();
            }
        } catch(ConstructionException e) { // nc4 const.
            System.out.println(e);
        }
    }
}
```

輸出為：

```
NeedsCleanup 1 disposed
NeedsCleanup 3 disposed
NeedsCleanup 2 disposed
NeedsCleanup 5 disposed
NeedsCleanup 4 disposed
```

- [1] 相當簡單，遵循了在可去除物件之後緊跟 try-finally 的原則。如果物件構造不會失敗，就不需要任何 catch。
- [2] 為了構造和清理，可以看到將具有不能失敗的構造器的物件分組在一起。
- [3] 展示了如何處理那些具有可以失敗的構造器，且需要清理的物件。為了正確處理這種情況，事情變得很棘手，因為對於每一個構造，都必須包含在其自己的 try-finally 語句塊中，並且每一個物件構造必須都跟隨一個 try-finally 語句塊以確保清理。

本例中異常處理的混亂情形，有力的論證了應該建立不會拋出異常的構造器，儘管這並不總會實現。

注意，如果 dispose() 可以拋出異常，那麼你可能需要額外的 try 語句塊。基本上，你應該仔細考慮所有的可能性，並確保正確處理每一種情況。

<!-- Try-With-Resources -->

## Try-With-Resources 用法

上一節的內容可能讓你有些頭痛。在考慮所有可能失敗的方法時，找出放置所有 try-catch-finally 塊的位置變得令人生畏。確保沒有任何故障路徑，使系統遠離不穩定狀態，這非常具有挑戰性。

`InputFile.java` 是一個特別棘手的情況，因為文件被打開（伴隨所有可能因此產生的異常），然後它在物件的生命週期中保持打開狀態。每次呼叫 `getLine()`都可能導致異常，而且 `dispose()`也是這種情況。這個例子只是好在它顯示了事情可以混亂到什麼地步。它還表明了你應該儘量不要那樣設計程式碼（當然，你經常會遇到這種無法選擇的程式碼設計的情況，因此你仍然必須要理解它）。

InputFile.java 一個更好的實現方式是如果建構子讀取文件並在內部緩衝它 —— 這樣，文件的打開，讀取和關閉都發生在建構子中。或者，如果讀取和儲存文件不切實際，你可以改為生成 Stream。理想情況下，你可以設計成如下的樣子：

```java
// exceptions/InputFile2.java
import java.io.*;
import java.nio.file.*;
import java.util.stream.*;
public class InputFile2 {
    private String fname;

    public InputFile2(String fname) {
        this.fname = fname;
    }

    public Stream<String> getLines() throws IOException {
        return Files.lines(Paths.get(fname));
    }

    public static void
    main(String[] args) throws IOException {
        new InputFile2("InputFile2.java").getLines()
                .skip(15)
                .limit(1)
                .forEach(System.out::println);
    }
}
```

輸出為：

```
main(String[] args) throws IOException {
```

現在，getLines() 全權負責打開文件並建立 Stream。

你不能總是輕易地迴避這個問題。有時會有以下問題：

1. 需要資源清理
2. 需要在特定的時刻進行資源清理，比如你離開作用域的時候（在通常情況下意味著透過異常進行清理）。

一個常見的例子是 `java.io.FileInputStream` （將會在 [附錄：I/O 流 ](./Appendix-IO-Streams.md) 中提到）。要正確使用它，你必須編寫一些棘手的樣板程式碼：

```java
// exceptions/MessyExceptions.java
import java.io.*;
public class MessyExceptions {
    public static void main(String[] args) {
        InputStream in = null;
        try {
            in = new FileInputStream(
                    new File("MessyExceptions.java"));
            int contents = in.read();
            // Process contents
        } catch(IOException e) {
            // Handle the error
        } finally {
            if(in != null) {
                try {
                    in.close();
                } catch(IOException e) {
                    // Handle the close() error
                }
            }
        }
    }
}
```

當 finally 子句有自己的 try 塊時，感覺事情變得過於複雜。

幸運的是，Java 7 引入了 try-with-resources 語法，它可以非常清楚地簡化上面的程式碼：

```java
// exceptions/TryWithResources.java
import java.io.*;
public class TryWithResources {
    public static void main(String[] args) {
        try(
                InputStream in = new FileInputStream(
                        new File("TryWithResources.java"))
        ) {
            int contents = in.read();
            // Process contents
        } catch(IOException e) {
            // Handle the error
        }
    }
}
```

在 Java 7 之前，try 後面總是跟著一個 {，但是現在可以跟一個帶括號的定義 ——這裡是我們建立的 FileInputStream 物件。括號內的部分稱為資源規範頭（resource specification header）。現在 `in` 在整個 try 塊的其餘部分都是可用的。更重要的是，無論你如何退出 try 塊（正常或透過異常），和以前的 finally 子句等價的程式碼都會被執行，並且不用編寫那些雜亂而棘手的程式碼。這是一項重要的改進。

它是如何工作的？ try-with-resources 定義子句中建立的物件（在括號內）必須實現  `java.lang.AutoCloseable` 介面，這個介面只有一個方法：`close()`。當在 Java 7 中引入 `AutoCloseable` 時，許多介面和類被修改以實現它；查看 Javadocs 中的 AutoCloseable，可以找到所有實現該介面的類列表，其中包括 `Stream` 物件：

```java
// exceptions/StreamsAreAutoCloseable.java
import java.io.*;
import java.nio.file.*;
import java.util.stream.*;
public class StreamsAreAutoCloseable {
    public static void
    main(String[] args) throws IOException{
        try(
                Stream<String> in = Files.lines(
                        Paths.get("StreamsAreAutoCloseable.java"));
                PrintWriter outfile = new PrintWriter(
                        "Results.txt"); // [1]
        ) {
            in.skip(5)
                    .limit(1)
                    .map(String::toLowerCase)
                    .forEachOrdered(outfile::println);
        } // [2]
    }
}
```

- [1] 你在這裡可以看到其他的特性：資源規範頭中可以包含多個定義，並且透過分號進行分割（最後一個分號是可選的）。規範頭中定義的每個物件都會在 try 語句塊執行結束之後呼叫 close() 方法。
- [2] try-with-resources 裡面的 try 語句塊可以不包含 catch 或者 finally 語句而獨立存在。在這裡，IOException 被 main() 方法拋出，所以這裡並不需要在 try 後面跟著一個 catch 語句塊。

Java 5 中的 Closeable 已經被修改，修改之後的介面繼承了 AutoCloseable 介面。所以所有實現了 Closeable 介面的物件，都支援了  try-with-resources 特性。

### 揭示細節

為了研究 try-with-resources 的基本機制，我們將建立自己的 AutoCloseable 類：

```java
// exceptions/AutoCloseableDetails.java
class Reporter implements AutoCloseable {
    String name = getClass().getSimpleName();
    Reporter() {
        System.out.println("Creating " + name);
    }
    public void close() {
        System.out.println("Closing " + name);
    }
}
class First extends Reporter {}
class Second extends Reporter {}
public class AutoCloseableDetails {
    public static void main(String[] args) {
        try(
                First f = new First();
                Second s = new Second()
        ) {
        }
    }
}
```

輸出為：

```
Creating First
Creating Second
Closing Second
Closing First
```

退出 try 塊會呼叫兩個物件的 close() 方法，並以與建立順序相反的順序關閉它們。順序很重要，因為在這種情況下，Second 物件可能依賴於 First 物件，因此如果 First 在第 Second 關閉時已經關閉。 Second 的 close() 方法可能會嘗試訪問 First 中不再可用的某些功能。

假設我們在資源規範頭中定義了一個不是 AutoCloseable 的物件

```java
// exceptions/TryAnything.java
// {WillNotCompile}
class Anything {}
public class TryAnything {
    public static void main(String[] args) {
        try(
                Anything a = new Anything()
        ) {
        }
    }
}
```

正如我們所希望和期望的那樣，Java 不會讓我們這樣做，並且出現編譯時錯誤。

如果其中一個建構子拋出異常怎麼辦？

```java
// exceptions/ConstructorException.java
class CE extends Exception {}
class SecondExcept extends Reporter {
    SecondExcept() throws CE {
        super();
        throw new CE();
    }
}
public class ConstructorException {
    public static void main(String[] args) {
        try(
                First f = new First();
                SecondExcept s = new SecondExcept();
                Second s2 = new Second()
        ) {
            System.out.println("In body");
        } catch(CE e) {
            System.out.println("Caught: " + e);
        }
    }
}
```

輸出為：

```
Creating First
Creating SecondExcept
Closing First
Caught: CE
```

現在資源規範頭中定義了 3 個物件，中間的物件拋出異常。因此，編譯器強制我們使用 catch 子句來捕獲建構子異常。這意味著資源規範頭實際上被 try 塊包圍。

正如預期的那樣，First 建立時沒有發生意外，SecondExcept 在建立期間拋出異常。請注意，不會為 SecondExcept 呼叫 close()，因為如果建構子失敗，則無法假設你可以安全地對該物件執行任何操作，包括關閉它。由於 SecondExcept 的異常，Second 物件實例 s2 不會被建立，因此也不會有清除事件發生。

如果沒有建構子拋出異常，但在 try 的主體中可能拋出異常，那麼你將再次被強制要求提供一個catch 子句：

```java
// exceptions/BodyException.java
class Third extends Reporter {}
public class BodyException {
    public static void main(String[] args) {
        try(
                First f = new First();
                Second s2 = new Second()
        ) {
            System.out.println("In body");
            Third t = new Third();
            new SecondExcept();
            System.out.println("End of body");
        } catch(CE e) {
            System.out.println("Caught: " + e);
        }
    }
}
```

輸出為：

```java
Creating First
Creating Second
In body
Creating Third
Creating SecondExcept
Closing Second
Closing First
Caught: CE
```

請注意，第 3 個物件永遠不會被清除。那是因為它不是在資源規範頭中建立的，所以它沒有被保護。這很重要，因為 Java 在這裡沒有以警告或錯誤的形式提供指導，因此像這樣的錯誤很容易漏掉。實際上，如果依賴某些整合開發環境來自動重寫程式碼，以使用 try-with-resources 特性，那麼它們（在撰寫本文時）通常只會保護它們遇到的第一個物件，而忽略其餘的物件。

最後，讓我們看一下拋出異常的 close() 方法：

```java
// exceptions/CloseExceptions.java
class CloseException extends Exception {}
class Reporter2 implements AutoCloseable {
    String name = getClass().getSimpleName();
    Reporter2() {
        System.out.println("Creating " + name);
    }
    public void close() throws CloseException {
        System.out.println("Closing " + name);
    }
}
class Closer extends Reporter2 {
    @Override
    public void close() throws CloseException {
        super.close();
        throw new CloseException();
    }
}
public class CloseExceptions {
    public static void main(String[] args) {
        try(
                First f = new First();
                Closer c = new Closer();
                Second s = new Second()
        ) {
            System.out.println("In body");
        } catch(CloseException e) {
            System.out.println("Caught: " + e);
        }
    }
}
```

輸出為：

```
Creating First
Creating Closer
Creating Second
In body
Closing Second
Closing Closer
Closing First
Caught: CloseException
```

從技術上講，我們並沒有被迫在這裡提供一個 catch 子句；你可以透過 **main() throws CloseException** 的方式來報告異常。但 catch 子句是放置錯誤處理程式碼的典型位置。

請注意，因為所有三個物件都已建立，所以它們都以相反的順序關閉 - 即使 Closer.close() 拋出異常也是如此。仔細想想，這就是你想要的結果。但如果你必須親手編寫所有的邏輯，或許會遺失一些東西並使得邏輯出錯。想想那些程式設計師沒有考慮 Clean up 的所有影響並且出錯的程式碼。因此，如果可以，你應當始終使用 try-with-resources。這個特性有助於生成更簡潔，更易於理解的程式碼。

<!-- Exception Matching -->

## 異常匹配

拋出異常的時候，異常處理系統會按照程式碼的書寫順序找出“最近”的處理程序。找到匹配的處理程序之後，它就認為異常將得到處理，然後就不再繼續尋找。

尋找的時候並不要求拋出的異常同處理程序所聲明的異常完全匹配。衍生類的物件也可以匹配其基類的處理程序，就像這樣：

```java
// exceptions/Human.java
// Catching exception hierarchies
class Annoyance extends Exception {}
class Sneeze extends Annoyance {}
public class Human {
    public static void main(String[] args) {
        // Catch the exact type:
        try {
            throw new Sneeze();
        } catch(Sneeze s) {
            System.out.println("Caught Sneeze");
        } catch(Annoyance a) {
            System.out.println("Caught Annoyance");
        }
        // Catch the base type:
        try {
            throw new Sneeze();
        } catch(Annoyance a) {
            System.out.println("Caught Annoyance");
        }
    }
}
```

輸出為：

```java
Caught Sneeze
Caught Annoyance
```

Sneeze 異常會被第一個匹配的 catch 子句捕獲，也就是程式裡的第一個。然而如果將這個 catch 子句刪掉，只留下 Annoyance 的 catch 子句，該程式仍然能執行，因為這次捕獲的是 Sneeze 的基類。換句話說，catch（Annoyance a）會捕獲 Annoyance 以及所有從它衍生的異常。這一點非常有用，因為如果決定在方法裡加上更多衍生異常的話，只要客戶程式設計師捕獲的是基類異常，那麼它們的程式碼就無需更改。

如果把捕獲基類的 catch 子句放在最前面，以此想把衍生類的異常全給“封鎖”掉，就像這樣：

```java
try {
    throw new Sneeze();
} catch(Annoyance a) {
    // ...
} catch(Sneeze s) {
    // ...
}
```

此時，編譯器會發現 Sneeze 的 catch 子句永遠得不到執行，因此它會向你報告錯誤。

<!-- Alternative Approaches -->

## 其他可選方式

異常處理系統就像一個活門（trap door），使你能放棄程式的正常執行序列。當“異常情形”發生的時候，正常的執行已變得不可能或者不需要了，這時就要用到這個“活門"。異常代表了當前方法不能繼續執行的情形。開發異常處理系統的原因是，如果為每個方法所有可能發生的錯誤都進行處理的話，任務就顯得過於繁重了，程式設計師也不願意這麼做。結果常常是將錯誤忽略。應該注意到，開發異常處理的初衷是為了方便程式設計師處理錯誤。

異常處理的一個重要原則是“只有在你知道如何處理的情況下才捕獲異常"。實際上，異常處理的一個重要目標就是把處理錯誤的程式碼同錯誤發生的地點相分離。這使你能在一段程式碼中專注於要完成的事情，至於如何處理錯誤，則放在另一段程式碼中完成。這樣一來，主要程式碼就不會與錯誤處理邏輯混在一起，也更容易理解和維護。透過允許一個處理程序去處理多個出錯點，異常處理還使得錯誤處理程式碼的數量趨於減少。

“被檢查的異常”使這個問題變得有些複雜，因為它們強制你在可能還沒準備好處理錯誤的時候被迫加上 catch 子句，這就導致了吞食則有害（harmful if swallowed）的問題：

```java
try {
    // ... to do something useful
} catch(ObligatoryException e) {} // Gulp!
```

程式設計師們只做最簡單的事情（包括我自己，在本書第 1 版中也有這個問題），常常是無意中"吞食”了異常，然而一旦這麼做，雖然能透過編譯，但除非你記得複查並改正程式碼，否則異常將會遺失。異常確實發生了，但“吞食”後它卻完全消失了。因為編譯器強迫你立刻寫程式碼來處理異常，所以這種看起來最簡單的方法，卻可能是最糟糕的做法。

當我意識到犯了這麼大一個錯誤時，簡直嚇了一大跳，在本書第 2 版中，我在處理程序裡透過列印堆疊軌跡的方法“修補”了這個問題（本章中的很多例子還是使用了這種方法，看起來還是比較合適的），雖然這樣可以跟蹤異常的行為，但是仍舊不知道該如何處理異常。這一節，我們來研究一下“被檢查的異常”及其併發症，以及採用什麼方法來解決這些問題。

這個話題看起來簡單，但實際上它不僅複雜，更重要的是還非常多變。總有人會頑固地堅持自己的立場，聲稱正確答案（也是他們的答案）是顯而易見的。我覺得之所以會有這種觀點，是因為我們使用的工具已經不是 ANSI 標準出台前的像 C 那樣的弱類型語言，而是像 C++ 和 Java 這樣的“強靜態類型語言”（也就是編譯時就做類型檢查的語言），這是前者所無法比擬的。當剛開始這種轉變的時候（就像我一樣），會覺得它帶來的好處是那樣明顯，好像類型檢查總能解決所有的問題。在此，我想結合我自己的認識過程，告訴讀者我是怎樣從對類型檢查的絕對迷信變成持懷疑態度的，當然，很多時候它還是非常有用的，但是當它擋住我們的去路並成為障礙的時候，我們就得跨過去。只是這條界限往往並不是很清晰（我最喜歡的一句格言是：所有模型都是錯誤的，但有些是能用的）。


### 歷史

異常處理起源於 PL/1 和 Mesa 之類的系統中，後來又出現在 CLU、Smalltalk、Modula-3、Ada、Eiffel、C++、Python、Java 以及後 Java 語言 Ruby 和 C# 中。Java 的設計和 C++ 很相似，只是 Java 的設計者去掉了一些他們認為 C++設計得不好的東西。

為了能向程式設計師提供一個他們更願意使用的錯誤處理和復原的框架，異常處理機制很晚才被加入 C++ 標準化過程中，這是由 C++ 的設計者 Bjarne Stroustrup 所倡議。C++ 的異常模型主要借鑑了 CLU 的做法。然而，當時其他語言已經支援異常處理了：包括 Ada、Smalltalk（兩者都有異常處理，但是都沒有異常說明），以及 Modula-3（它既有異常處理也有異常說明）。

Liskov 和 Snyder 在他們的一篇討論該主題的獨創性論文中指出，用瞬時風格（transient fashion）報告錯誤的語言（如 C 中）有一個主要缺陷，那就是：

> “....每次呼叫的時候都必須執行條件測試，以確定會產生何種結果。這使程式難以閱讀並且有可能降低執行效率，因此程式設計師們既不願意指出，也不願意處理異常。”

因此，異常處理的初衷是要消除這種限制，但是我們又從 Java 的“被檢查的異常”中看到了這種程式碼。他們繼續寫道：

> “....要求程式設計師把異常處理程序的程式碼文字附接到會引發異常的呼叫上，這會降低程式的可讀性，使得程式的正常思路被異常處理給破壞了。”

C++ 中異常的設計參考了 CLU 方式。Stroustrup 聲稱其目標是減少復原錯誤所需的程式碼。我想他這話是說給那些通常情況下都不寫 C 的錯誤處理的程式設計師們聽的，因為要把那麼多程式碼放到那麼多地方實在不是什麼好差事。所以他們寫 C 程式的習慣是，忽略所有的錯誤，然後使用除錯器來跟蹤錯誤。這些程式設計師知道，使用異常就意味著他們要寫一些通常不用寫的、“多出來的”程式碼。因此，要把他們拉到“使用錯誤處理”的正軌上，“多出來的”程式碼決不能太多。我認為，評價 Java 的“被檢查的異常”的時候，這一點是很重要的。

C++ 從 CLU 那裡還帶來另一種思想：異常說明。這樣，就可以用編程的方式在方法簽名中聲明這個方法將會拋出異常。異常說明有兩個目的：一個是“我的程式碼會產生這種異常，這由你來處理”。另一個是“我的程式碼忽略了這些異常，這由你來處理”。學習異常處理的機制和語法的時候，我們一直在關注“你來處理”部分，但這裡特別值得注意的事實是，我們通常都忽略了異常說明所表達的完整含義。

C++ 的異常說明不屬於函數的類型訊息。編譯時唯一要檢查的是異常說明是不是前後一致；比如，如果函數或方法會拋出某些異常，那麼它的重載版本或者衍生版本也必須拋出同樣的異常。與 Java 不同，C++ 不會在編譯時進行檢查以確定函數或方法是不是真的拋出異常，或者異常說明是不是完整（也就是說，異常說明有沒有精確描述所有可能被拋出的異常）。這樣的檢查只發生在執行期間。如果拋出的異常與異常說明不符，C++ 會呼叫標準類庫的 unexpected() 函數。

值得注意的是，由於使用了模板，C++ 的標準類庫實現裡根本沒有使用異常說明。在 Java 中，對於泛型用於異常說明的方式存在著一些限制。

### 觀點

首先，值得注意的是 Java 有效的發明了“被檢查的異常”（很明顯是受 C++ 異常說明的啟發，以及異常說明通常並不煩擾 C++ 程式設計師的事實），但是，這還只是一次嘗試，目前還沒有別的語言選擇複製這種做法。

其次，僅從示意性的例子和小程式來看，“被檢查的異常”的好處很明顯。但是當程式開始變大的時候，就會帶來一些微妙的問題。當然，程式不是一下就變大的，這有個過程。如果把不適用於大項目的語言用於小項目，當這些項目不斷膨脹時，突然有一天你會發現，原來可以管理的東西，現在已經變得無法管理了。這就是我所說的過多的類型檢查，特別是“被檢查的異常"所造成的問題。

看來程式的規模是個重要因素。由於很多討論都用小程式來做示範，因此這並不足以說明問題。一名 C# 的設計人員發現：

> “僅從小程式來看，會認為異常說明能增加開發人員的效率，並提高程式碼的質量；但考察大項目的時候，結論就不同了-開發效率下降了，而程式碼質量只有微不足道的提高，甚至毫無提高”。

談到未被捕獲的異常的時候，CLU 的設計師們認為：

> “我們覺得強迫程式設計師在不知道該採取什麼措施的時候提供處理程序，是不現實的。”

在解釋為什麼“函數沒有異常說明就表示可以拋出任何異常”的時候，Stroustrup 這樣認為：

> “但是，這樣一來幾乎所有的函數都得提供異常說明了，也就都得重新編譯，而且還會妨礙它同其他語言的互動。這樣會迫使程式設計師違反異常處理機制的約束，他們會寫欺騙程式來掩蓋異常。這將給沒有注意到這些異常的人造成一種虛假的安全感。”
>

我們已經看到這種破壞異常機制的行為了-就在 Java 的“被檢查的異常”裡。

Martin Fowler（UML Distilled，Refactoring 和 Analysis Patterns 的作者）給我寫了下面這段話：

> “...總體來說，我覺得異常很不錯，但是 Java 的”被檢查的異常“帶來的麻煩比好處要多。”

我現在認為 Java 的重要進步是統一了錯誤報告模式，所有錯誤都用異常來報告。這沒有在 C++ 中發生，原因是為了向後相容 C ，直接忽略錯誤的舊模式在 C++ 中依然是可用的。如果想使用異常，可以始終用異常來報告，如果不想這樣做，異常可以拋到最高的級別（比如控制台）。當 Java 修改 C++ 的模式來讓異常成為報告錯誤的唯一方式時，受檢查的異常的額外限制可能就變得不那麼必要了。

過去，我曾堅定地認為“被檢查的異常”和強靜態類型檢查對開發健壯的程式是非常必要的。但是，我看到的以及我使用一些動態（類型檢查）語言的親身經歷告訴我，這些好處實際上是來自於：

1. 不在於編譯器是否會強制程式設計師去處理錯誤，而是要有一致的、使用異常來報告錯誤的模型。
2. 不在於什麼時候進行檢查，而是一定要有類型檢查。也就是說，必須強制程式使用正確的類型，至於這種強制施加於編譯時還是執行時，那倒沒關係。

此外，減少編譯時施加的約束能顯著提高程式設計師的程式效率。事實上，反射和泛型就是用來補償靜態類型檢查所帶來的過多限制，在本書很多例子中都會見到這種情形。

我已經聽到有人在指責了，他們認為這種言論會令我名譽掃地，會讓文明墮落，會導致更高比例的項目失敗。他們的信念是應該在編譯時指出所有錯誤，這樣才能挽救項目，這種信念可以說是無比堅定的；其實更重要的是要理解編譯器的能力限制。在 http://MindView.net/Books/BetterJava 上的補充材料中，我強調了自動構建過程和單元測試的重要性，比起把所有的東西都說成是語法錯誤，它們的效果可以說是事半功倍。下面這段話是至理名言：

> 好的程式設計語言能幫助程式設計師寫出好程式，但無論哪種語言都避免不了程式設計師用它寫出壞程式。

不管怎麼說，要讓 Java 把“被檢查的異常”從語言中去除，這種可能性看來非常渺茫。對語言來說，這個變化可能太激進了點，況且 Sun 的支援者們也非常強大。Sun 有完全向後相容的歷史和策略，實際上所有 Sun 的軟體都能在 Sun 的硬體上執行，無論它們有多麼古老。然而，如果發現有些“被檢查的異常”擋住了路，尤其是發現你不得不去對付那些不知道該如何處理的異常，還是有些辦法的。

### 把異常傳遞給控制台

在簡單的程式中，不用寫多少程式碼就能保留異常的最簡單的方法，就是把它們從 `main()` 傳遞到控制台。例如，為了讀取訊息而打開一個文件（在[文件](17-Files.md) 章節中將詳細介紹），必須對 `FilelnputStream` 進行打開和關閉操作，這就可能會產生異常。對於簡單的程式，可以像這樣做（本書中很多地方採用了這種方法）：

```java
// exceptions/MainException.java
import java.util.*;
import java.nio.file.*;
public class MainException {
    // Pass exceptions to the console:
    public static void main(String[] args) throws Exception {
        // Open the file:
        List<String> lines = Files.readAllLines(
                Paths.get("MainException.java"));
        // Use the file ...
    }
}
```

注意，main() 作為一個方法也可以有異常說明，這裡異常的類型是 Exception，它也是所有“被檢查的異常”的基類。透過把它傳遞到控制台，就不必在 main() 裡寫 try-catch 子句了。（不過，實際的文件輸人輸出操作比這個例子要複雜得多。你將會在 [文件](./Files.md) 和 [附錄：I/O 流](./Appendix-IO-Streams.md) 章節中學到更多）

### 把“被檢查的異常”轉換為“不檢查的異常”

當編寫自己使用的簡單程式時，從 `main()` 中拋出異常是很方便的，但這並不總是有用。真正的問題是，當在一個普通方法裡呼叫別的方法時發現：“我不知道該如何處理這個異常，但是不能把它'吞掉'或者列印一些無用的消息。”有了異常鏈，一個簡單的解決辦法就出現了。可以透過將一個“被檢查的異常”傳遞給`RuntimeException` 的構造器，從而將它包裝進 `RuntimeException` 裡，就像這樣：

```java
try {
    // ... to do something useful
} catch(IDontKnowWhatToDoWithThisCheckedException e) {
    throw new RuntimeException(e);
}
```

如果想把“被檢查的異常”這種功能“封鎖”掉的話，這看起來像是一個好辦法。不用“吞下”異常，也不必把它放到方法的異常說明裡面，而異常鏈還能保證你不會遺失任何原始異常的訊息。

這種技巧給了你一種選擇，你可以不寫 try-catch 子句和/或異常說明，直接忽略異常，讓它自己沿著呼叫堆疊往上“冒泡”，同時，還可以用 getCause() 捕獲並處理特定的異常，就像這樣：

```java
// exceptions/TurnOffChecking.java
// "Turning off" Checked exceptions
import java.io.*;
class WrapCheckedException {
    void throwRuntimeException(int type) {
        try {
            switch(type) {
                case 0: throw new FileNotFoundException();
                case 1: throw new IOException();
                case 2: throw new
                        RuntimeException("Where am I?");
                default: return;
            }
        } catch(IOException | RuntimeException e) {
            // Adapt to unchecked:
            throw new RuntimeException(e);
        }
    }
}
class SomeOtherException extends Exception {}
public class TurnOffChecking {
    public static void main(String[] args) {
        WrapCheckedException wce =
                new WrapCheckedException();
        // You can call throwRuntimeException() without
        // a try block, and let RuntimeExceptions
        // leave the method:
        wce.throwRuntimeException(3);
        // Or you can choose to catch exceptions:
        for(int i = 0; i < 4; i++)
            try {
                if(i < 3)
                    wce.throwRuntimeException(i);
                else
                    throw new SomeOtherException();
            } catch(SomeOtherException e) {
                System.out.println(
                        "SomeOtherException: " + e);
            } catch(RuntimeException re) {
                try {
                    throw re.getCause();
                } catch(FileNotFoundException e) {
                    System.out.println(
                            "FileNotFoundException: " + e);
                } catch(IOException e) {
                    System.out.println("IOException: " + e);
                } catch(Throwable e) {
                    System.out.println("Throwable: " + e);
                }
            }
    }
}
```

輸出為：

```
FileNotFoundException: java.io.FileNotFoundException
IOException: java.io.IOException
Throwable: java.lang.RuntimeException: Where am I?
SomeOtherException: SomeOtherException
```

`WrapCheckedException.throwRuntimeException()` 包含可生成不同類型異常的程式碼。這些異常被捕獲並包裝進`RuntimeException` 物件，所以它們成了這些執行時異常的原因（"cause"）。

在 TurnOfChecking 裡，可以不用 try 塊就呼叫 throwRuntimeException()，因為它沒有拋出“被檢查的異常”。但是，當你準備好去捕獲異常的時候，還是可以用 try 塊來捕獲任何你想捕獲的異常的。應該捕獲 try 塊肯定會拋出的異常，這裡就是 SomeOtherException，RuntimeException 要放到最後去捕獲。然後把 getCause() 的結果（也就是被包裝的那個原始異常）拋出來。這樣就把原先的那個異常給提取出來了，然後就可以用它們自己的 catch 子句進行處理。

這種把被檢查的異常用 `RuntimeException` 包裝起來的技術，將在本書餘下部分使用。另一種解決方案是建立自己的 `RuntimeException` 的子類。這樣的話，異常捕獲將不被強制要求，但是任何人都可以在需要的時候捕獲這些異常。

<!-- Exception Guidelines -->

## 異常指南

應該在下列情況下使用異常：

1. 儘可能使用 try-with-resource。
2. 在恰當的級別處理問題。（在知道該如何處理的情況下才捕獲異常。）
3. 解決問題並且重新呼叫產生異常的方法。
4. 進行少許修補，然後繞過異常發生的地方繼續執行。
5. 用別的資料進行計算，以代替方法預計會返回的值。
6. 把目前執行環境下能做的事情儘量做完，然後把相同的異常重拋到更高層。
7. 把目前執行環境下能做的事情儘量做完，然後把不同的異常拋到更高層。
8. 終止程式。
9. 進行簡化。（如果你的異常模式使問題變得太複雜，那用起來會非常痛苦也很煩人。）
10. 讓類庫和程式更安全。（這既是在為除錯做短期投資，也是在為程式的健壯性做長期投資。）

<!-- Summary -->

## 本章小結

異常是 Java 程式設計不可分割的一部分，如果不了解如何使用它們，那你只能完成很有限的工作。正因為如此，本書專門在此介紹了異常——對於許多類庫（例如提到過的 I/O 庫），如果不處理異常，你就無法使用它們。

異常處理的優點之一就是它使得你可以在某處集中精力處理你要解決的問題，而在另一處處理你編寫的這段程式碼中產生的錯誤。儘管異常通常被認為是一種工具，使得你可以在執行時報告錯誤並從錯誤中復原，但是我一直懷疑到底有多少時候“復原”真正得以實現了，或者能夠得以實現。我認為這種情況少於 10%，並且即便是這 10%，也只是將堆疊展開到某個已知的穩定狀態，而並沒有實際執行任何種類的復原性行為。無論這是否正確，我一直相信“報告”功能是異常的精髓所在. Java 堅定地強調將所有的錯誤都以異常形式報告的這一事實，正是它遠遠超過如 C++ 這類語言的長處之一，因為在 C++ 這類語言中，需要以大量不同的方式來報告錯誤，或者根本就沒有提供錯誤報告功能。一致的錯誤報告系統意味著，你再也不必對所寫的每一段程式碼，都質問自己“錯誤是否正在成為漏網之魚？”（只要你沒有“吞嚥”異常，這是關鍵所在！）。

就像你將要在後續章節中看到的，透過將這個問題甩給其他程式碼-即使你是透過拋出 RuntimeException 來實現這一點的--你在設計和實現時，便可以專注於更加有趣和富有挑戰性的問題了。

## 後記：Exception Bizarro World

（來自於 2011 年的一篇博文）

我的朋友 James Ward 正在嘗試使用 JDBC 建立一些非常簡單的教學範例，但不斷被受檢查的異常所挫敗。他把 Howard Lewis Ship 的帖子“[被檢查的異常的悲劇](http://tapestryjava.blogspot.com/2011/05/tragedy-of-checked-exceptions.html)”指給我看。讓 James 尤其沮喪的是，即使做一些本來很簡單的事情，也必須在一個個環裡跳來跳去。即使在 `finally` 塊中，他也不得不放入更多的 `try-catch` 子句，因為關閉連接也會導致異常。這些麻煩事的終點在哪裡？本來只是做一些簡單的事，但卻被強制要求在一個個環裡跳來跳去（注意，try-with-resources語句可以顯著改善這種情況）。

我們開始討論 Go 程式語言，我很著迷，因為Rob Pike等人。我們已經清楚地提出了許多關於語言設計的非常尖銳和基本的問題。基本上，他們已經採取了我們開始接受的有關語言的所有內容，並詢問“為什麼？”關於每一種語言。學習這門語言真的讓你思考和懷疑。

我的印象是Go團隊不做任何臆斷，只有在明確一個特徵是必須的時候才改進語言。他們似乎並不擔心做出破壞舊程式碼的更改 ，因為他們建立了一個重寫工具，當做出更改的時候，重寫工具將為你重寫程式碼。這使他們將語言變成一個前進的實驗，以發現真正需要的東西，而不是做 Big Upfront Design。

他們做出的最有趣的決定之一是完全排除異常。你沒有看錯 —— 他們不只是遺漏了經過檢查的異常情況。他們遺漏了所有異常情況。

替代方案非常簡單，起初它幾乎看起來像 C 一樣。因為 Go 從一開始就包含了元組，所以你可以輕鬆地從函數呼叫中返回兩個物件：

```go
result, err := functionCall()
```

（ `:=` 告訴 Go 語言在這裡定義 `result` 和 `err`，並且推斷它們的類型）

就是這樣：對於每次呼叫，您都會獲得結果物件和錯誤物件。您可以立即檢查錯誤（這是典型的，因為如果某些操作失敗，則不太可能繼續下一步），或者稍後檢查是否有效。

起初這看起來很原始，是向“古代”的回歸。但到目前為止，我發現 Go 中的決定都經過了很好的考慮，值得深思。我的反應是因為我的大腦是異常的嗎？這會如何影響 James 的問題？

它發生在我身上，我已經將異常處理視為一種並行執行路徑。如果你遇到異常，你會跳出正常的路徑進入這個並行執行路徑，這是一種“奇異世界”，你不再做你寫的東西，而是跳進 catch 和 finally 子句。正是這種替代執行路徑的世界導致了 James 抱怨的問題。

James  創造了一個物件。理想的情況下。物件建立不會導致潛在的異常，因此你必須抓住它們。你必須透過 try-finally 跟蹤建立以確保清理發生（Python團隊意識到清理不是一個特殊的條件，而是一個單獨的問題，所以他們建立了一個不同的語言構造 - 以便停止混淆二者）。任何導致異常的呼叫都會停止正常的執行路徑並跳轉（透過並行bizarro-world）到 catch 子句。

關於異常的一個基本假設是，我們透過在塊結束時收集所有錯誤處理程式碼而不是在它們發生時處理錯誤來獲益。在這兩種情況下，我們都會停止正常執行，但是異常處理有一個自動機制，它會將你從正常的執行路徑中拋出，跳轉到你的並行異常世界，然後在正確的處理程序中再次彈出你。

跳入奇異的世界會給 James 帶來問題，它為所有程式設計師增加了更多的工作：因為你無法知道什麼時候會發生什麼事（你可以隨時進入奇怪的世界），你必須添加一些 try 塊來確保沒有任何東西從裂縫中滑落。您最終必須進行額外的程式以補償異常機制（它似乎類似於補償共享記憶體並發所需的額外工作）。

Go 團隊採取了大膽的舉動，質疑所有這些，並說，“讓我們毫無例外地嘗試它，看看會發生什麼事。”是的，這意味著你通常會在發生錯誤的地方處理錯誤，而不是最後將它們聚集在一起 try 塊。但這也意味著關於一件事的程式碼是本地化的，也許這並不是那麼糟糕。這也可能意味著您無法輕鬆組合常見的錯誤處理程式碼（除非您確定了常用程式碼並將其放入函數中，也不是那麼糟糕）。但這絕對意味著您不必擔心有多個可能的執行路徑而且所有這些都需要。

<!-- 分頁 -->

<div style="page-break-after: always;"></div>
