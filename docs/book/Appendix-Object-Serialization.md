[TOC]

<!-- Appendix: Object Serialization -->
# 附錄:物件序列化

當你建立物件時，只要你需要，它就會一直存在，但是在程式終止時，無論如何它都不會繼續存在。儘管這麼做肯定是有意義的，但是仍舊存在某些情況，如果物件能夠在程式不執行的情況下仍能存在併儲存其訊息，那將非常有用。這樣，在下次執行程式時，該物件將被重建並且擁有的訊息與在程式上次執行時它所擁有的訊息相同。當然，你可以透過將訊息寫入檔案或資料庫來達到相同的效果，但是在使萬物都成為物件的精神中，如果能夠將一個物件聲明為是“持久性”的，並為我們處理掉所有細節，那將會顯得十分方便。

Java 的物件序列化將那些實現了 Serializable 介面的物件轉換成一個位元組序列，並能夠在以後將這個位元組序列完全復原為原來的物件。這一過程甚至可透過網路進行，這意味著序列化機制能自動彌補不同作業系統之間的差異。也就是說，可以在執行 Windows 系統的電腦上建立一個物件，將其序列化，透過網路將它發送給一台執行 Unix 系統的電腦，然後在那裡準確地重新組裝，而卻不必擔心資料在不同機器上的表示會不同，也不必關心宇節的順序或者其他任何細節。

就其本身來說，物件序列化可以實現輕量級持久性（lightweight persistence），“持久性”意味著一個物件的生存週期並不取決於程式是否正在執行它可以生存於程式的呼叫之間。透過將一個序列化物件寫入磁碟，然後在重新呼叫程式時復原該物件，就能夠實現持久性的效果。之所以稱其為“輕量級”，是因為不能用某種"persistent"（持久）關鍵字來簡單地定義一個物件，並讓系統自動維護其他細節問題（儘管將來有可能實現）。相反，物件必須在程式中顯式地序列化（serialize）和反序列化還原（deserialize），如果需要個更嚴格的持久性機制，可以考慮像 Hibernate 之類的工具。

物件序列化的概念加入到語言中是為了支援兩種主要特性。一是 Java 的遠端方法呼叫（Remote Method Invocation，RMI），它使存活於其他電腦上的物件使用起來就像是存活於本機上一樣。當向遠端物件發送消息時，需要透過物件序列化來傳輸參數和返回值。

再者，對 Java Beans 來說，物件的序列化也是必需的（在撰寫本文時被視為失敗的技術），使用一個 Bean 時，一般情況下是在設計階段對它的狀態訊息進行配置。這種狀態訊息必須儲存下來，並在程式啟動時進行後期復原，這種具體工作就是由物件序列化完成的。

只要物件實現了 Serializable 介面（該介面僅是一個標記介面，不包括任何方法），物件的序列化處理就會非常簡單。當序列化的概念被加入到語言中時，許多標準庫類都發生了改變，以便具備序列化特性-其中包括所有基本資料類型的封裝器、所有容器類以及許多其他的東西。甚至 Class 物件也可以被序列化。

要序列化一個物件，首先要建立某些 OutputStream 物件，然後將其封裝在一個 ObjectOutputStream 物件內。這時，只需呼叫 writeObject() 即可將物件序列化，並將其發送給 OutputStream（物件化序列是基於位元組的，因要使用 InputStream 和 OutputStream 繼承層次結構）。要反向進行該過程（即將一個序列還原為一個物件），需要將一個 InputStream 封裝在 ObjectInputStream 內，然後呼叫 readObject()。和往常一樣，我們最後獲得的是一個引用，它指向一個向上轉型的 Object，所以必須向下轉型才能直接設定它們。

物件序列化特別“聰明”的一個地方是它不僅儲存了物件的“全景圖”，而且能追蹤物件內所包含的所有引用，並儲存那些物件；接著又能對物件內包含的每個這樣的引用進行追蹤，依此類推。這種情況有時被稱為“物件網”，單個物件可與之建立連接，而且它還包含了物件的引用陣列以及成員物件。如果必須保持一套自己的物件序列化機制，那麼維護那些可追蹤到所有連結的程式碼可能會顯得非常麻煩。然而，由於 Java 的物件序列化似乎找不出什麼缺點，所以請儘量不要自己動手，讓它用最佳化的演算法自動維護整個物件網。下面這個例子透過對連結的物件生成一個 worm（蠕蟲）對序列化機制進行了測試。每個物件都與 worm 中的下一段連結，同時又與屬於不同類（Data）的物件引用陣列連結：

```java
// serialization/Worm.java
// Demonstrates object serialization
import java.io.*;
import java.util.*;
class Data implements Serializable {
    private int n;
    Data(int n) { this.n = n; }
    @Override
    public String toString() {
        return Integer.toString(n);
    }
}
public class Worm implements Serializable {
    private static Random rand = new Random(47);
    private Data[] d = {
            new Data(rand.nextInt(10)),
            new Data(rand.nextInt(10)),
            new Data(rand.nextInt(10))
    };
    private Worm next;
    private char c;
    // Value of i == number of segments
    public Worm(int i, char x) {
        System.out.println("Worm constructor: " + i);
        c = x;
        if(--i > 0)
            next = new Worm(i, (char)(x + 1));
    }
    public Worm() {
        System.out.println("No-arg constructor");
    }
    @Override
    public String toString() {
        StringBuilder result = new StringBuilder(":");
        result.append(c);
        result.append("(");
        for(Data dat : d)
            result.append(dat);
        result.append(")");
        if(next != null)
            result.append(next);
        return result.toString();
    }
    public static void
    main(String[] args) throws ClassNotFoundException,
            IOException {
        Worm w = new Worm(6, 'a');
        System.out.println("w = " + w);
        try(
                ObjectOutputStream out = new ObjectOutputStream(
                        new FileOutputStream("worm.dat"))
        ) {
            out.writeObject("Worm storage\n");
            out.writeObject(w);
        }
        try(
                ObjectInputStream in = new ObjectInputStream(
                        new FileInputStream("worm.dat"))
        ) {
            String s = (String)in.readObject();
            Worm w2 = (Worm)in.readObject();
            System.out.println(s + "w2 = " + w2);
        }
        try(
                ByteArrayOutputStream bout =
                        new ByteArrayOutputStream();
                ObjectOutputStream out2 =
                        new ObjectOutputStream(bout)
        ) {
            out2.writeObject("Worm storage\n");
            out2.writeObject(w);
            out2.flush();
            try(
                    ObjectInputStream in2 = new ObjectInputStream(
                            new ByteArrayInputStream(
                                    bout.toByteArray()))
            ) {
                String s = (String)in2.readObject();
                Worm w3 = (Worm)in2.readObject();
                System.out.println(s + "w3 = " + w3);
            }
        }
    }
}
```

輸出為：

```
Worm constructor: 6
Worm constructor: 5
Worm constructor: 4
Worm constructor: 3
Worm constructor: 2
Worm constructor: 1
w = :a(853):b(119):c(802):d(788):e(199):f(881)
Worm storage
w2 = :a(853):b(119):c(802):d(788):e(199):f(881)
Worm storage
w3 = :a(853):b(119):c(802):d(788):e(199):f(881)
```

更有趣的是，Worm 內的 Data 物件陣列是用隨機數初始化的（這樣就不用懷疑編譯器保留了某種原始訊息），每個 Worm 段都用一個 char 加以標記。該 char 是在遞迴生成連結的 Worm 列表時自動產生的。要建立一個 Worm，必須告訴構造器你所希望的它的長度。在產生下一個引用時，要呼叫 Worm 構造器，並將長度減 1，以此類推。最後一個 next 引用則為 null（空），表示已到達 Worm 的尾部

以上這些操作都使得事情變得更加複雜，從而加大了物件序列化的難度。然而，真正的序列化過程卻是非常簡單的。一旦從另外某個流建立了 ObjectOutputstream，writeObject() 就會將物件序列化。注意也可以為一個 String 呼叫 writeObject() 也可以用與 DataOutputStream 相同的方法寫人所有基本資料類型（它們具有同樣的介面）。

有兩段看起來相似的獨立的程式碼。一個讀寫的是文件，而另一個讀寫的是位元組陣列（ByteArray），可利用序列化將物件讀寫到任何 DatalnputStream 或者 DataOutputStream。

從輸出中可以看出，被還原後的物件確實包含了原物件中的所有連結。

注意在對一個 Serializable 物件進行還原的過程中，沒有呼叫任何構造器，包括預設的構造器。整個物件都是透過從 InputStream 中取得資料復原而來的。

<!-- Finding the Class -->

## 尋找類

你或許會奇怪，將一個物件從它的序列化狀態中復原出來，有哪些工作是必須的呢？舉個例子來說，假如我們將一個物件序列化，並透過網路將其作為文件傳送給另一台電腦，那麼，另一台電腦上的程式可以只利用該文件內容來還原這個物件嗎？

回答這個問題的最好方法就是做一個實驗。下面這個檔案位於本章的子目錄下：

```java
// serialization/Alien.java
// A serializable class
import java.io.*;
public class Alien implements Serializable {}
```

而用於建立和序列化一個 Alien 物件的文件也位於相同的目錄下：

```java
// serialization/FreezeAlien.java
// Create a serialized output file
import java.io.*;
public class FreezeAlien {
    public static void main(String[] args) throws Exception {
        try(
                ObjectOutputStream out = new ObjectOutputStream(
                        new FileOutputStream("X.file"));
        ) {
            Alien quellek = new Alien();
            out.writeObject(quellek);
        }
    }
}
```

一旦該程式被編譯和執行，它就會在 c12 目錄下產生一個名為 X.file 的文件。以下程式碼位於一個名為 xiles 的子目錄下：

```java
// serialization/xfiles/ThawAlien.java
// Recover a serialized file
// {java serialization.xfiles.ThawAlien}
// {RunFirst: FreezeAlien}
package serialization.xfiles;
import java.io.*;
public class ThawAlien {
    public static void main(String[] args) throws Exception {
        ObjectInputStream in = new ObjectInputStream(
                new FileInputStream(new File("X.file")));
        Object mystery = in.readObject();
        System.out.println(mystery.getClass());
    }
}
```

輸出為：

```java
class Alien
```

為了正常執行，必須保證 Java 虛擬機能找到相關的.class 文件。

<!-- Controlling Serialization -->

## 控制序列化

正如大家所看到的，預設的序列化機制並不難操縱。然而，如果有特殊的需要那又該怎麼辦呢？例如，也許要考慮特殊的安全問題，而且你不希望物件的某一部分被序列化；或者一個物件被還原以後，某子物件需要重新建立，從而不必將該子物件序列化。

在這些特殊情況下，可透過實現 Externalizable 介面——代替實現 Serializable 介面-來對序列化過程進行控制。這個 Externalizable 介面繼承了 Serializable 介面，同時增添了兩個方法：writeExternal() 和 readExternal()。這兩個方法會在序列化和反序列化還原的過程中被自動呼叫，以便執行一些特殊操作。

下面這個例子展示了 Externalizable 介面方法的簡單實現。注意 Blip1 和 Blip2 除了細微的差別之外，幾乎完全一致（研究一下程式碼，看看你能否發現）：

```java
// serialization/Blips.java
// Simple use of Externalizable & a pitfall
import java.io.*;
class Blip1 implements Externalizable {
    public Blip1() {
        System.out.println("Blip1 Constructor");
    }
    @Override
    public void writeExternal(ObjectOutput out)
            throws IOException {
        System.out.println("Blip1.writeExternal");
    }
    @Override
    public void readExternal(ObjectInput in)
            throws IOException, ClassNotFoundException {
        System.out.println("Blip1.readExternal");
    }
}
class Blip2 implements Externalizable {
    Blip2() {
        System.out.println("Blip2 Constructor");
    }
    @Override
    public void writeExternal(ObjectOutput out)
            throws IOException {
        System.out.println("Blip2.writeExternal");
    }
    @Override
    public void readExternal(ObjectInput in)
            throws IOException, ClassNotFoundException {
        System.out.println("Blip2.readExternal");
    }
}
public class Blips {
    public static void main(String[] args) {
        System.out.println("Constructing objects:");
        Blip1 b1 = new Blip1();
        Blip2 b2 = new Blip2();
        try(
                ObjectOutputStream o = new ObjectOutputStream(
                        new FileOutputStream("Blips.serialized"))
        ) {
            System.out.println("Saving objects:");
            o.writeObject(b1);
            o.writeObject(b2);
        } catch(IOException e) {
            throw new RuntimeException(e);
        }
        // Now get them back:
        System.out.println("Recovering b1:");
        try(
                ObjectInputStream in = new ObjectInputStream(
                        new FileInputStream("Blips.serialized"))
        ) {
            b1 = (Blip1)in.readObject();
        } catch(IOException | ClassNotFoundException e) {
            throw new RuntimeException(e);
        }
        // OOPS! Throws an exception:
        //- System.out.println("Recovering b2:");
        //- b2 = (Blip2)in.readObject();
    }
}
```

輸出為：

```
Constructing objects:
Blip1 Constructor
Blip2 Constructor
Saving objects:
Blip1.writeExternal
Blip2.writeExternal
Recovering b1:
Blip1 Constructor
Blip1.readExternal
```

沒有復原 Blip2 對像的原因是那樣做會導致一個異常。你找出 Blip1 和 Blip2 之間的區別了嗎？Blipl 的構造器是“公共的”（public），Blip2 的構造器卻不是，這樣就會在復原時造成異常。試試將 Blip2 的構造器變成 public 的，然後刪除//注釋標記，看看是否能得到正確的結果。

復原 b1 後，會呼叫 Blip1 預設構造器。這與復原一個 Serializable 物件不同。對於 Serializable 物件，物件完全以它儲存的二進位制位為基礎來構造，而不呼叫構造器。而對於一個 Externalizable 物件，所有普通的預設構造器都會被呼叫（包括在欄位定義時的初始化），然後呼叫 readExternal() 必須注意這一點--所有預設的構造器都會被呼叫，才能使 Externalizable 物件產生正確的行為。

下面這個例子示範了如何完整儲存和復原一個 Externalizable 物件：

```java
// serialization/Blip3.java
// Reconstructing an externalizable object
import java.io.*;
public class Blip3 implements Externalizable {
    private int i;
    private String s; // No initialization
    public Blip3() {
        System.out.println("Blip3 Constructor");
// s, i not initialized
    }
    public Blip3(String x, int a) {
        System.out.println("Blip3(String x, int a)");
        s = x;
        i = a;
// s & i initialized only in non-no-arg constructor.
    }
    @Override
    public String toString() { return s + i; }
    @Override
    public void writeExternal(ObjectOutput out)
            throws IOException {
        System.out.println("Blip3.writeExternal");
// You must do this:
        out.writeObject(s);
        out.writeInt(i);
    }
    @Override
    public void readExternal(ObjectInput in)
            throws IOException, ClassNotFoundException {
        System.out.println("Blip3.readExternal");
// You must do this:
        s = (String)in.readObject();
        i = in.readInt();
    }
    public static void main(String[] args) {
        System.out.println("Constructing objects:");
        Blip3 b3 = new Blip3("A String ", 47);
        System.out.println(b3);
        try(
                ObjectOutputStream o = new ObjectOutputStream(
                        new FileOutputStream("Blip3.serialized"))
        ) {
            System.out.println("Saving object:");
            o.writeObject(b3);
        } catch(IOException e) {
            throw new RuntimeException(e);
        }
// Now get it back:
        System.out.println("Recovering b3:");
        try(
                ObjectInputStream in = new ObjectInputStream(
                        new FileInputStream("Blip3.serialized"))
        ) {
            b3 = (Blip3)in.readObject();
        } catch(IOException | ClassNotFoundException e) {
            throw new RuntimeException(e);
        }
        System.out.println(b3);
    }
}
```

輸出為：

```
Constructing objects:
Blip3(String x, int a)
A String 47
Saving object:
Blip3.writeExternal
Recovering b3:
Blip3 Constructor
Blip3.readExternal
A String 47
```

其中，欄位 s 和 i 只在第二個構造器中初始化，而不是在預設的構造器中初始化。這意味著假如不在 readExternal() 中初始化 s 和 i，s 就會為 null，而 i 就會為零（因為在建立物件的第一步中將物件的儲存空間清理為 0）。如果注釋掉跟隨於"You must do this”後面的兩行程式碼，然後執行程式，就會發現當物件被還原後，s 是 null，而 i 是零。

我們如果從一個 Externalizable 物件繼承，通常需要呼叫基類版本的 writeExternal() 和 readExternal() 來為基類元件提供恰當的儲存和復原功能。

因此，為了正常執行，我們不僅需要在 writeExternal() 方法（沒有任何預設行為來為 Externalizable 物件寫入任何成員物件）中將來自物件的重要訊息寫入，還必須在 readExternal() 方法中復原資料。起先，可能會有一點迷惑，因為 Externalizable 物件的預設構造行為使其看起來似乎像某種自動發生的儲存與復原操作。但實際上並非如此。

### transient 關鍵字

當我們對序列化進行控制時，可能某個特定子物件不想讓 Java 的序列化機制自動儲存與復原。如果子物件表示的是我們不希望將其序列化的敏感訊息（如密碼），通常就會面臨這種情況。即使物件中的這些訊息是 private（私有）屬性，一經序列化處理，人們就可以透過讀取文件或者攔截網路傳輸的方式來訪問到它。

有一種辦法可防止物件的敏感部分被序列化，就是將類實現為 Externalizable，如前面所示。這樣一來，沒有任何東西可以自動序列化，並且可以在 writeExternal() 內部只對所需部分進行顯式的序列化。

然而，如果我們正在操作的是一個 Seralizable 物件，那麼所有序列化操作都會自動進行。為了能夠予以控制，可以用 transient（瞬時）關鍵字逐個欄位地關閉序列化，它的意思是“不用麻煩你儲存或復原資料——我自己會處理的"。

例如，假設某個 Logon 物件儲存某個特定的登入工作階段訊息，登入的合法性通過校驗之後，我們想把資料儲存下來，但不包括密碼。為做到這一點，最簡單的辦法是實現 Serializable，並將 password 欄位標誌為 transient，下面是具體的程式碼：

```java
// serialization/Logon.java
// Demonstrates the "transient" keyword
import java.util.concurrent.*;
import java.io.*;
import java.util.*;
import onjava.Nap;
public class Logon implements Serializable {
    private Date date = new Date();
    private String username;
    private transient String password;
    public Logon(String name, String pwd) {
        username = name;
        password = pwd;
    }
    @Override
    public String toString() {
        return "logon info: \n username: " +
                username + "\n date: " + date +
                "\n password: " + password;
    }
    public static void main(String[] args) {
        Logon a = new Logon("Hulk", "myLittlePony");
        System.out.println("logon a = " + a);
        try(
                ObjectOutputStream o =
                        new ObjectOutputStream(
                                new FileOutputStream("Logon.dat"))
        ) {
            o.writeObject(a);
        } catch(IOException e) {
            throw new RuntimeException(e);
        }
        new Nap(1);
// Now get them back:
        try(
                ObjectInputStream in = new ObjectInputStream(
                        new FileInputStream("Logon.dat"))
        ) {
            System.out.println(
                    "Recovering object at " + new Date());
            a = (Logon)in.readObject();
        } catch(IOException | ClassNotFoundException e) {
            throw new RuntimeException(e);
        }
        System.out.println("logon a = " + a);
    }
}
```

輸出為：

```
logon a = logon info:
username: Hulk
date: Tue May 09 06:07:47 MDT 2017
password: myLittlePony
Recovering object at Tue May 09 06:07:49 MDT 2017
logon a = logon info:
username: Hulk
date: Tue May 09 06:07:47 MDT 2017
password: null
```

可以看到，其中的 date 和 username 是一般的（不是 transient 的），所以它們會被自動序列化。而 password 是 transient 的，所以不會被自動儲存到磁碟；另外，自動序列化機制也不會嘗試去復原它。當物件被復原時，password 就會變成 null。注意，雖然 toString() 是用重載後的+運算符來連接 String 物件，但是 null 引用會被自動轉換成字串 null。

我們還可以發現：date 欄位被儲存了到磁碟並從磁碟上被復原了出來，而且沒有再重新生成。由於 Externalizable 物件在預設情況下不儲存它們的任何欄位，所以 transient 關鍵字只能和 Serializable 物件一起使用。

### Externalizable 的替代方法

如果不是特別堅持實現 Externalizable 介面，那麼還有另一種方法。我們可以實現 Serializable 介面，並添加（注意我說的是“添加”，而非“覆蓋”或者“實現”）名為 writeObject() 和 readObject() 的方法。這樣一旦物件被序列化或者被反序列化還原，就會自動地分別呼叫這兩個方法。也就是說，只要我們提供了這兩個方法，就會使用它們而不是預設的序列化機制。

這些方法必須具有準確的方法特徵簽名：

```java
private void writeObject(ObjectOutputStream stream) throws IOException

private void readObject(ObjectInputStream stream) throws IOException, ClassNotFoundException
```

從設計的觀點來看，現在事情變得真是不可思議。首先，我們可能會認為由於這些方法不是基類或者 Serializable 介面的一部分，所以應該在它們自己的介面中進行定義。但是注意它們被定義成了 private，這意味著它們僅能被這個類的其他成員呼叫。然而，實際上我們並沒有從這個類的其他方法中呼叫它們，而是 ObjectOutputStream 和 ObjectInputStream 物件的 writeObject() 和 readobject() 方法呼叫你的物件的 writeObject() 和 readObject() 方法（注意關於這裡用到的相同方法名，我儘量抑制住不去謾罵。一句話：混亂）。讀者可能想知道 ObjectOutputStream 和 ObjectInputStream 物件是怎樣訪問你的類中的 private 方法的。我們只能假設這正是序列化神奇的一部分。

在介面中定義的所有東西都自動是 public 的，因此如果 writeObject() 和 readObject() 必須是 private 的，那麼它們不會是介面的一部分。因為我們必須要完全遵循其方法特徵簽名，所以其效果就和實現了介面一樣。

在呼叫 ObjectOutputStream.writeObject() 時，會檢查所傳遞的 Serializable 物件，看看是否實現了它自己的 writeObject()。如果是這樣，就跳過正常的序列化過程並呼叫它的 writeObiect()。readObject() 的情形與此相同。

還有另外一個技巧。在你的 writeObject() 內部，可以呼叫 defaultWriteObject() 來選擇執行預設的 writeObject()。類似地，在 readObject() 內部，我們可以呼叫 defaultReadObject()，下面這個簡單的例子示範了如何對一個 Serializable 物件的儲存與復原進行控制：

```java
// serialization/SerialCtl.java
// Controlling serialization by adding your own
// writeObject() and readObject() methods
import java.io.*;
public class SerialCtl implements Serializable {
    private String a;
    private transient String b;
    public SerialCtl(String aa, String bb) {
        a = "Not Transient: " + aa;
        b = "Transient: " + bb;
    }
    @Override
    public String toString() { return a + "\n" + b; }
    private void writeObject(ObjectOutputStream stream)
            throws IOException {
        stream.defaultWriteObject();
        stream.writeObject(b);
    }
    private void readObject(ObjectInputStream stream)
            throws IOException, ClassNotFoundException {
        stream.defaultReadObject();
        b = (String)stream.readObject();
    }
    public static void main(String[] args) {
        SerialCtl sc = new SerialCtl("Test1", "Test2");
        System.out.println("Before:\n" + sc);
        try (
                ByteArrayOutputStream buf =
                        new ByteArrayOutputStream();
                ObjectOutputStream o =
                        new ObjectOutputStream(buf);
        ) {
            o.writeObject(sc);
// Now get it back:
            try (
                    ObjectInputStream in =
                            new ObjectInputStream(
                                    new ByteArrayInputStream(
                                            buf.toByteArray()));
            ) {
                SerialCtl sc2 = (SerialCtl)in.readObject();
                System.out.println("After:\n" + sc2);
            }
        } catch(IOException | ClassNotFoundException e) {
            throw new RuntimeException(e);
        }
    }
}
```

輸出為：

```
Before:
Not Transient: Test1
Transient: Test2
After:
Not Transient: Test1
Transient: Test2
```

在這個例子中，有一個 String 欄位是普通欄位，而另一個是 transient 欄位，用來證明非 transient 欄位由 defaultWriteObject() 方法儲存，而 transient 欄位必須在程式中明確儲存和復原。欄位是在構造器內部而不是在定義處進行初始化的，以此可以證實它們在反序列化還原期間沒有被一些自動化機制初始化。

如果我們打算使用預設機制寫入物件的非 transient 部分，那麼必須呼叫 defaultwriteObject() 作為 writeObject() 中的第一個操作，並讓 defaultReadObject() 作為 readObject() 中的第一個操作。這些都是奇怪的方法呼叫。例如，如果我們正在為 ObjectOutputStream 呼叫 defaultWriteObject() 且沒有傳遞任何參數，然而不知何故它卻可以執行，並且知道物件的引用以及如何寫入非 transient 部分。真是奇怪之極。

對 transient 物件的儲存和復原使用了我們比較熟悉的程式碼。請再考慮一下在這裡所發生的事情。在 main0）中，建立 SerialCtl 物件，然後將其序列化到 ObjectOutputStream（注意在這種情況下，使用的是緩衝區而不是文件-這對於 ObjectOutputStream 來說是完全一樣的）。序列化發生在下面這行程式碼當中

```java
o.writeObject(sc);
```

writeObject() 方法必須檢查 sc，判斷它是否擁有自己的 writeObject() 方法（不是檢查介面——這裡根本就沒有介面，也不是檢查類的類型，而是利用反射來真正地搜尋方法）。如果有，那麼就會使用它。對 readObject() 也採用了類似的方法。或許這是解決這個問題的唯一切實可行的方法，但它確實有點古怪。

### 版本控制

有時可能想要改變可序列化類的版本（比如源類的物件可能儲存在資料庫中）。雖然 Java 支援這種做法，但是你可能只在特殊的情況下才這樣做，此外，還需要對它有相當深程度的了解（在這裡我們就不再試圖達到這一點）。從 http://java.oracle.com 下的 JDK 文件中對這一主題進行了非常徹底的論述。

<!-- Using Persistence -->

## 使用持久化

一個比較誘人的使用序列化技術的想法是：儲存程式的一些狀態，以便我們隨後可以很容易地將程式復原到目前狀態。但是在我們能夠這樣做之前，必須回答幾個問題。如果我們將兩個物件-它們都具有指向第三個物件的引用-進行序列化，會發生什麼情況？當我們從它們的序列化狀態復原這兩個物件時，第三個物件會只出現一次嗎？如果將這兩個物件序列化成獨立的文件，然後在程式碼的不同部分對它們進行反序列化還原，又會怎樣呢？

下面這個例子說明了上述問題：

```java
// serialization/MyWorld.java
import java.io.*;
import java.util.*;
class House implements Serializable {}
class Animal implements Serializable {
    private String name;
    private House preferredHouse;
    Animal(String nm, House h) {
        name = nm;
        preferredHouse = h;
    }
    @Override
    public String toString() {
        return name + "[" + super.toString() +
                "], " + preferredHouse + "\n";
    }
}
public class MyWorld {
    public static void main(String[] args) {
        House house = new House();
        List<Animal> animals = new ArrayList<>();
        animals.add(
                new Animal("Bosco the dog", house));
        animals.add(
                new Animal("Ralph the hamster", house));
        animals.add(
                new Animal("Molly the cat", house));
        System.out.println("animals: " + animals);
        try(
                ByteArrayOutputStream buf1 =
                        new ByteArrayOutputStream();
                ObjectOutputStream o1 =
                        new ObjectOutputStream(buf1)
        ) {
            o1.writeObject(animals);
            o1.writeObject(animals); // Write a 2nd set
// Write to a different stream:
            try(
                    ByteArrayOutputStream buf2 = new ByteArrayOutputStream();
                    ObjectOutputStream o2 = new ObjectOutputStream(buf2)
            ) {
                o2.writeObject(animals);
// Now get them back:
                try(
                        ObjectInputStream in1 =
                                new ObjectInputStream(
                                        new ByteArrayInputStream(
                                                buf1.toByteArray()));
                        ObjectInputStream in2 =
                                new ObjectInputStream(
                                        new ByteArrayInputStream(
                                                buf2.toByteArray()))
                ) {
                    List
                            animals1 = (List)in1.readObject(),
                            animals2 = (List)in1.readObject(),
                            animals3 = (List)in2.readObject();
                    System.out.println(
                            "animals1: " + animals1);
                    System.out.println(
                            "animals2: " + animals2);
                    System.out.println(
                            "animals3: " + animals3);
                }
            }
        } catch(IOException | ClassNotFoundException e) {
            throw new RuntimeException(e);
        }
    }
}
```

輸出為：

```
animals: [Bosco the dog[Animal@15db9742],
House@6d06d69c
, Ralph the hamster[Animal@7852e922], House@6d06d69c
, Molly the cat[Animal@4e25154f], House@6d06d69c
]
animals1: [Bosco the dog[Animal@7ba4f24f],
House@3b9a45b3
, Ralph the hamster[Animal@7699a589], House@3b9a45b3
, Molly the cat[Animal@58372a00], House@3b9a45b3
]
animals2: [Bosco the dog[Animal@7ba4f24f],
House@3b9a45b3
, Ralph the hamster[Animal@7699a589], House@3b9a45b3
, Molly the cat[Animal@58372a00], House@3b9a45b3
]
animals3: [Bosco the dog[Animal@4dd8dc3],
House@6d03e736
, Ralph the hamster[Animal@568db2f2], House@6d03e736
, Molly the cat[Animal@378bf509], House@6d03e736
]
```



這裡有一件有趣的事：我們可以透過一個位元組陣列來使用物件序列化，從而實現對任何可 Serializable 物件的“深度複製"（deep copy）—— 深度複製意味著我們複製的是整個物件網，而不僅僅是基本物件及其引用。複製物件將在本書的 [附錄：傳遞和返回物件 ]() 一章中進行深入地探討。

在這個例子中，Animal 物件包含有 House 類型的欄位。在 main() 方法中，建立了一個 Animal 列表並將其兩次序列化，分別送至不同的流。當其被反序列化還原並被列印時，我們可以看到所示的執行某次執行後的結果（每次執行時，物件將會處在不同的記憶體地址）。

當然，我們期望這些反序列化還原後的物件地址與原來的地址不同。但請注意，在 animals1 和 animals2 中卻出現了相同的地址，包括二者共享的那個指向 House 物件的引用。另一方面，當復原 animals3 時，系統無法知道另一個流內的物件是第一個流內的物件的別名，因此它會產生出完全不同的物件網。

只要將任何物件序列化到單一流中，就可以復原出與我們寫出時一樣的物件網，並且沒有任何意外重複複製出的物件。當然，我們可以在寫出第一個物件和寫出最後一個物件期間改變這些物件的狀態，但是這是我們自己的事，無論物件在被序列化時處於什麼狀態（無論它們和其他物件有什麼樣的連接關係），它們都可以被寫出。

最安全的做法是將其作為“原子”操作進行序列化。如果我們序列化了某些東西，再去做其他一些工作，再來序列化更多的東西，如此等等，那麼將無法安全地儲存系統狀態。取而代之的是，將構成系統狀態的所有物件都置入單一容器內，並在一個操作中將該容器直接寫出。然後同樣只需一次方法呼叫，即可以將其復原。

下面這個例子是一個想像的電腦輔助設計（CAD）系統，該例示範了這一方法。此外，它還引入了 static 欄位的問題：如果我們查看 JDK 文件，就會發現 Class 是 Serializable 的，因此只需直接對 Class 物件序列化，就可以很容易地儲存 static 欄位。在任何情況下，這都是一種明智的做法。

```java
// serialization/AStoreCADState.java
// Saving the state of a fictitious CAD system
import java.io.*;
import java.util.*;
import java.util.stream.*;
enum Color { RED, BLUE, GREEN }
abstract class Shape implements Serializable {
    private int xPos, yPos, dimension;
    private static Random rand = new Random(47);
    private static int counter = 0;
    public abstract void setColor(Color newColor);
    public abstract Color getColor();
    Shape(int xVal, int yVal, int dim) {
        xPos = xVal;
        yPos = yVal;
        dimension = dim;
    }
    public String toString() {
        return getClass() + "color[" + getColor() +
                "] xPos[" + xPos + "] yPos[" + yPos +
                "] dim[" + dimension + "]\n";
    }
    public static Shape randomFactory() {
        int xVal = rand.nextInt(100);
        int yVal = rand.nextInt(100);
        int dim = rand.nextInt(100);
        switch(counter++ % 3) {
            default:
            case 0: return new Circle(xVal, yVal, dim);
            case 1: return new Square(xVal, yVal, dim);
            case 2: return new Line(xVal, yVal, dim);
        }
    }
}
class Circle extends Shape {
    private static Color color = Color.RED;
    Circle(int xVal, int yVal, int dim) {
        super(xVal, yVal, dim);
    }
    public void setColor(Color newColor) {
        color = newColor;
    }
    public Color getColor() { return color; }
}
class Square extends Shape {
    private static Color color = Color.RED;
    Square(int xVal, int yVal, int dim) {
        super(xVal, yVal, dim);
    }
    public void setColor(Color newColor) {
        color = newColor;
    }
    public Color getColor() { return color; }
}
class Line extends Shape {
    private static Color color = Color.RED;
    public static void
    serializeStaticState(ObjectOutputStream os)
            throws IOException { os.writeObject(color); }
    public static void
    deserializeStaticState(ObjectInputStream os)
            throws IOException, ClassNotFoundException {
        color = (Color)os.readObject();
    }
    Line(int xVal, int yVal, int dim) {
        super(xVal, yVal, dim);
    }
    public void setColor(Color newColor) {
        color = newColor;
    }
    public Color getColor() { return color; }
}
public class AStoreCADState {
    public static void main(String[] args) {
        List<Class<? extends Shape>> shapeTypes =
                Arrays.asList(
                        Circle.class, Square.class, Line.class);
        List<Shape> shapes = IntStream.range(0, 10)
                .mapToObj(i -> Shape.randomFactory())
                .collect(Collectors.toList());
        // Set all the static colors to GREEN:
        shapes.forEach(s -> s.setColor(Color.GREEN));
        // Save the state vector:
        try(
                ObjectOutputStream out =
                        new ObjectOutputStream(
                                new FileOutputStream("CADState.dat"))
        ) {
            out.writeObject(shapeTypes);
            Line.serializeStaticState(out);
            out.writeObject(shapes);
        } catch(IOException e) {
            throw new RuntimeException(e);
        }
        // Display the shapes:
        System.out.println(shapes);
    }
}
```

輸出為：

```java
[class Circlecolor[GREEN] xPos[58] yPos[55] dim[93]
, class Squarecolor[GREEN] xPos[61] yPos[61] dim[29]
, class Linecolor[GREEN] xPos[68] yPos[0] dim[22]
, class Circlecolor[GREEN] xPos[7] yPos[88] dim[28]
, class Squarecolor[GREEN] xPos[51] yPos[89] dim[9]
, class Linecolor[GREEN] xPos[78] yPos[98] dim[61]
, class Circlecolor[GREEN] xPos[20] yPos[58] dim[16]
, class Squarecolor[GREEN] xPos[40] yPos[11] dim[22]
, class Linecolor[GREEN] xPos[4] yPos[83] dim[6]
, class Circlecolor[GREEN] xPos[75] yPos[10] dim[42]
]
```

Shape 類實現了 Serializable，所以任何自 Shape 繼承的類也都會自動是 Serializable 的。每個 Shape 都含有資料，而且每個衍生自 Shape 的類都包含一個 static 欄位，用來確定各種 Shape 類型的顏色（如果將 static 欄位置入基類，只會產生一個 static 欄位，因為 static 欄位不能在衍生類中複製）。可對基類中的方法進行重載，以便為不同的類型設定顏色（static 方法不會動態綁定，所以這些都是普通的方法）。每次呼叫 randomFactory() 方法時，它都會使用不同的隨機數作為 Shape 的資料，從而建立不同的 Shape。

在 main() 中，一個 ArrayList 用於儲存 Class 物件，而另一個用於儲存幾何形狀。

復原物件相當直觀：

```java
// serialization/RecoverCADState.java
// Restoring the state of the fictitious CAD system
// {RunFirst: AStoreCADState}
import java.io.*;
import java.util.*;
public class RecoverCADState {
    @SuppressWarnings("unchecked")
    public static void main(String[] args) {
        try(
                ObjectInputStream in =
                        new ObjectInputStream(
                                new FileInputStream("CADState.dat"))
        ) {
// Read in the same order they were written:
            List<Class<? extends Shape>> shapeTypes =
                    (List<Class<? extends Shape>>)in.readObject();
            Line.deserializeStaticState(in);
            List<Shape> shapes =
                    (List<Shape>)in.readObject();
            System.out.println(shapes);
        } catch(IOException | ClassNotFoundException e) {
            throw new RuntimeException(e);
        }
    }
}
```

輸出為：

```java
[class Circlecolor[RED] xPos[58] yPos[55] dim[93]
, class Squarecolor[RED] xPos[61] yPos[61] dim[29]
, class Linecolor[GREEN] xPos[68] yPos[0] dim[22]
, class Circlecolor[RED] xPos[7] yPos[88] dim[28]
, class Squarecolor[RED] xPos[51] yPos[89] dim[9]
, class Linecolor[GREEN] xPos[78] yPos[98] dim[61]
, class Circlecolor[RED] xPos[20] yPos[58] dim[16]
, class Squarecolor[RED] xPos[40] yPos[11] dim[22]
, class Linecolor[GREEN] xPos[4] yPos[83] dim[6]
, class Circlecolor[RED] xPos[75] yPos[10] dim[42]
]
```

可以看到，xPos，yPos 以及 dim 的值都被成功地儲存和復原了，但是對 static 訊息的讀取卻出現了問題。所有讀回的顏色應該都是“3”，但是真實情況卻並非如此。Circle 的值為 1（定義為 RED），而 Square 的值為 0（記住，它們是在構造器中被初始化的）。看起來似乎 static 資料根本沒有被序列化！確實如此——儘管 Class 類是 Serializable 的，但它卻不能按我們所期望的方式執行。所以假如想序列化 static 值，必須自己動手去實現。

這正是 Line 中的 serializeStaticState() 和 deserializeStaticState() 兩個 static 方法的用途。可以看到，它們是作為儲存和讀取過程的一部分被顯式地呼叫的。（注意必須維護寫入序列化文件和從該文件中讀回的順序。）因此，為了使 CADStatejava 正確運轉起來，你必須：

1. 為幾何形狀添加 serializeStaticState() 和 deserializeStaticState()
2. 移除 ArrayList shapeTypes 以及與之有關的所有程式碼。
3. 在幾何形狀內添加對新的序列化和反序列化還原靜態方法的呼叫。

另一個要注意的問題是安全，因為序列化也會將 private 資料儲存下來。如果你關心安全問題，那麼應將其標記成 transient，但是這之後，還必須設計一種安全的儲存訊息的方法，以便在執行復原時可以復位那些 private 變數。

## XML

物件序列化的一個重要限制是它只是 Java 的解決方案：只有 Java 程式才能反序列化這種物件。一種更具互操作性的解決方案是將資料轉換為 XML 格式，這可以使其被各式各樣的平台和語言使用。

因為 XML 十分流行，所以用它來編程時的各種選擇不勝列舉，包括隨 JDK 發布的 javax.xml.*類庫。我選擇使用 Elliotte Rusty Harold 的開源 XOM 類庫（可從 www.xom.nu 下載並獲得文件），因為它看起來最簡單，同時也是最直觀的用 Java 產生和修改 XML 的方式。另外，XOM 還強調了 XML 的正確性。

作為一個範例，假設有一個 APerson 物件，它包含姓和名，你想將它們序列化到 XML 中。下面的 APerson 類有一個 getXML() 方法，它使用 XOM 來產生被轉換為 XML 的 Element 物件的 APerson 資料；還有一個構造器，接受 Element 並從中抽取恰當的 APerson 資料（注意，XML 範例都在它們自己的子目錄中）：

```java
// serialization/APerson.java
// Use the XOM library to write and read XML
// nu.xom.Node comes from http://www.xom.nu
import nu.xom.*;
import java.io.*;
import java.util.*;
public class APerson {
    private String first, last;
    public APerson(String first, String last) {
        this.first = first;
        this.last = last;
    }
    // Produce an XML Element from this APerson object:
    public Element getXML() {
        Element person = new Element("person");
        Element firstName = new Element("first");
        firstName.appendChild(first);
        Element lastName = new Element("last");
        lastName.appendChild(last);
        person.appendChild(firstName);
        person.appendChild(lastName);
        return person;
    }
    // Constructor restores a APerson from XML:
    public APerson(Element person) {
        first = person
                .getFirstChildElement("first").getValue();
        last = person
                .getFirstChildElement("last").getValue();
    }
    @Override
    public String toString() {
        return first + " " + last;
    }
    // Make it human-readable:
    public static void
    format(OutputStream os, Document doc)
            throws Exception {
        Serializer serializer =
                new Serializer(os,"ISO-8859-1");
        serializer.setIndent(4);
        serializer.setMaxLength(60);
        serializer.write(doc);
        serializer.flush();
    }
    public static void main(String[] args) throws Exception {
        List<APerson> people = Arrays.asList(
                new APerson("Dr. Bunsen", "Honeydew"),
                new APerson("Gonzo", "The Great"),
                new APerson("Phillip J.", "Fry"));
        System.out.println(people);
        Element root = new Element("people");
        for(APerson p : people)
            root.appendChild(p.getXML());
        Document doc = new Document(root);
        format(System.out, doc);
        format(new BufferedOutputStream(
                new FileOutputStream("People.xml")), doc);
    }
}
```

輸出為：

```xml
[Dr. Bunsen Honeydew, Gonzo The Great, Phillip J. Fry]
<?xml version="1.0" encoding="ISO-8859-1"?>
<people>
    <person>
        <first>Dr. Bunsen</first>
        <last>Honeydew</last>
    </person>
    <person>
        <first>Gonzo</first>
        <last>The Great</last>
    </person>
    <person>
        <first>Phillip J.</first>
        <last>Fry</last>
    </person>
</people>
```

XOM 的方法都具有相當的自解釋性，可以在 XOM 文件中找到它們。XOM 還包含一個 Serializer 類，你可以在 format() 方法中看到它被用來將 XML 轉換為更具可讀性的格式。如果只呼叫 toXML()，那麼所有東西都會混在一起，因此 Serializer 是一種便利工具。

從 XML 文件中反序列化 Person 物件也很簡單：


```java
// serialization/People.java
// nu.xom.Node comes from http://www.xom.nu
// {RunFirst: APerson}
import nu.xom.*;
import java.io.File;
import java.util.*;
public class People extends ArrayList<APerson> {
    public People(String fileName) throws Exception {
        Document doc =
                new Builder().build(new File(fileName));
        Elements elements =
                doc.getRootElement().getChildElements();
        for(int i = 0; i < elements.size(); i++)
            add(new APerson(elements.get(i)));
    }
    public static void main(String[] args) throws Exception {
        People p = new People("People.xml");
        System.out.println(p);
    }
}
/* Output:
[Dr. Bunsen Honeydew, Gonzo The Great, Phillip J. Fry]
*/
```

People 構造器使用 XOM 的 Builder.build() 方法打開並讀取一個文件，而 getChildElements() 方法產生了一個 Elements 列表（不是標準的 Java List，只是一個擁有 size() 和 get() 方法的物件，因為 Harold 不想強制人們使用特定版本的 Java，但是仍舊希望使用類型安全的容器）。在這個列表中的每個 Element 都表示一個 Person 物件，因此它可以傳遞給第二個 Person 構造器。注意，這要求你提前知道 XML 文件的確切結構，但是這經常會有些問題。如果文件結構與你預期的結構不匹配，那麼 XOM 將拋出異常。對你來說，如果你缺乏有關將來的 XML 結構的訊息，那麼就有可能會編寫更複雜的程式碼去探測 XML 文件，而不是只對其做出假設。

為了獲取這些範例去編譯它們，你必須將 XOM 發布包中的 JAR 文件放置到你的類路徑中。

這裡只給出了用 Java 和 XOM 類庫進行 XML 編程的簡介，更詳細的訊息可以瀏覽 www.xom.nu 。



<!-- 分頁 -->

<div style="page-break-after: always;"></div>
