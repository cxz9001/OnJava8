[TOC]

<!-- Appendix: New I/O -->
# 附錄:新IO


> Java 新I/O 庫是在 1.4 版本引入到 `java.nio.*` 包中的，旨在更快速。

實際上，新 I/O 使用 **NIO**（同步非阻塞）的方式重寫了老的 I/O 了，因此它獲得了 **NIO** 的種種優點。即使我們不顯式地使用 **NIO** 方式來編寫程式碼，也能帶來性能和速度的提高。這種提升不僅僅體現在文件讀寫（File I/O），同時也體現在網路讀寫（Network I/O）中。例如，網路編程。

速度的提升來自於使用了更接近作業系統 I/O 執行方式的結構：**Channel**（通道） 和 **Buffer**（緩衝區）。我們可以想像一個煤礦：通道就是連接礦層（資料）的礦井，緩衝區是運送煤礦的小車。透過小車裝煤，再從車裡取礦。換句話說，我們不能直接和 **Channel** 互動; 我們需要與 **Buffer** 互動並將 **Buffer** 中的資料發送到 **Channel** 中；**Channel** 需要從 **Buffer** 中提取或放入資料。

本篇我們將深入探討 `nio` 包。雖然 像 I/O 流這樣的進階庫使用了 **NIO**，但多數時候，我們考慮這個層次的問題。使用Java 7 和 8 版本，理想情況下我們甚至不必費心去處理 I/O 流。當然，一些特殊情況除外。在[文件](./17-Files.md)（**File**）一章中基本涵蓋了我們日常使用的相關內容。只有在遇到性能瓶頸（例如記憶體映射文件）或建立自己的 I/O 庫時，我們才需要去理解 **NIO**。


<!-- ByteBufferS -->
## ByteBuffer


有且僅有 **ByteBuffer**（位元組緩衝區，儲存原始位元組的緩衝區）這一類型可直接與通道互動。查看 `java.nio.`**ByteBuffer** 的 JDK 文件，你會發現它是相當基礎的：透過初始化某個大小的儲存空間，再使用一些方法以原始位元組形式或原始資料類型來放置和獲取資料。但是我們無法直接存放物件，即使是最基本的 **String** 類型資料。這是一個相當底層的操作，也正因如此，使得它與大多數作業系統的映射更加高效。


舊式 I/O 中的三個類分別被更新成 **FileChannel**（文件通道），分別是：**FileInputStream**、**FileOutputStream**，以及用於讀寫的 **RandomAccessFile** 類。

注意，這些都是符合底層 **NIO** 特性的位元組操作流。 另外，還有 **Reader** 和 **Writer** 字元模式的類是不產生通道的。但 `java.nio.channels.`**Channels** 類具有從通道中生成 **Reader** 和 **Writer** 的實用方法。

下面來練習上述三種類型的流生成可讀、可寫、可讀/寫的通道：

```java
// (c)2017 MindView LLC: see Copyright.txt
// 我們不保證這段程式碼用於其他用途時是否有效
// 訪問 http://OnJava8.com 了解更多訊息
// 從流中獲取通道
import java.nio.*;
import java.nio.channels.*;
import java.io.*;

public class GetChannel {
  private static String name = "data.txt";
  private static final int BSIZE = 1024;
  public static void main(String[] args) {
    // 寫入一個文件:
    try(
      FileChannel fc = new FileOutputStream(name)
        .getChannel()
    ) {
      fc.write(ByteBuffer
        .wrap("Some text ".getBytes()));
    } catch(IOException e) {
      throw new RuntimeException(e);
    }
    // 在文件尾添加：
    try(
      FileChannel fc = new RandomAccessFile(
        name, "rw").getChannel()
    ) {
      fc.position(fc.size()); // 移動到結尾
      fc.write(ByteBuffer
        .wrap("Some more".getBytes()));
    } catch(IOException e) {
      throw new RuntimeException(e);
    }
    // 讀取文件e:
    try(
      FileChannel fc = new FileInputStream(name)
        .getChannel()
    ) {
      ByteBuffer buff = ByteBuffer.allocate(BSIZE);
      fc.read(buff);
      buff.flip();
      while(buff.hasRemaining())
        System.out.write(buff.get());
    } catch(IOException e) {
      throw new RuntimeException(e);
    }
    System.out.flush();
  }
}
```

輸出結果：

```
Some text Some more
```

我們這裡所講的任何流類，都可以透過呼叫 `getChannel( )` 方法生成一個 **FileChannel**（文件通道）。**FileChannel** 的操作相當基礎：操作 **ByteBuffer**  來用於讀寫，並獨占式訪問和鎖定文件區域(稍後將對此進行描述)。

將位元組放入 **ByteBuffer** 的一種方法是直接呼叫 `put()` 方法將一個或多個位元組放入 **ByteBuffer**；當然也可以是其它基本類型的資料。此外，參考上例，我們還可以呼叫 `wrap()` 方法包裝現有位元組陣列到 **ByteBuffer**。執行此操作時，不會複製底層陣列，而是將其用作生成的 **ByteBuffer** 儲存。這樣產生的 **ByteBuffer** 是陣列“支援”的。

data.txt 文件被 **RandomAccessFile** 重新打開。**注意**，你可以在文件中移動 **FileChannel**。 在這裡，它被移動到末尾，以便添加額外的寫操作。

對於唯讀訪問，必須使用靜態 `allocate()` 方法顯式地分配 **ByteBuffer**。**NIO** 的目標是快速移動大量資料，因此 **ByteBuffer** 的大小應該很重要 —— 實際上，這裡設置的 1K 都可能偏小了(我們在工作中應該反覆測試以找到最佳大小)。

透過使用 `allocateDirect()` 而不是 `allocate()` 來生成與作業系統具備更高耦合度的“直接”緩衝區，也有可能獲得更高的速度。然而，這種分配的開銷更大，而且實際效果因作業系統的不同而有所不同，因此，在工作中你必須再次測試程式，以檢驗直接緩衝區是否能為你帶來速度上的優勢。

一旦呼叫 **FileChannel** 類的 `read()` 方法將位元組資料儲存到 **ByteBuffer** 中，你還必須呼叫緩衝區上的 `flip()` 方法來準備好提取位元組（這聽起來有點粗糙，實際上這已是非常低層的操作，且為了達到最高速度）。如果要進一步呼叫 `read()` 來使用 **ByteBuffer** ，還需要每次 `clear()` 來準備緩衝區。下面是個簡單的程式碼範例:

```java
// newio/ChannelCopy.java

// 使用 channels and buffers 移動文件
// {java ChannelCopy ChannelCopy.java test.txt}
import java.nio.*;
import java.nio.channels.*;
import java.io.*;

public class ChannelCopy {
  private static final int BSIZE = 1024;
  public static void main(String[] args) {
    if(args.length != 2) {
      System.out.println(
        "arguments: sourcefile destfile");
      System.exit(1);
    }
    try(
      FileChannel in = new FileInputStream(
        args[0]).getChannel();
      FileChannel out = new FileOutputStream(
        args[1]).getChannel()
    ) {
      ByteBuffer buffer = ByteBuffer.allocate(BSIZE);
      while(in.read(buffer) != -1) {
        buffer.flip(); // 準備寫入
        out.write(buffer);
        buffer.clear();  // 準備讀取
      }
    } catch(IOException e) {
      throw new RuntimeException(e);
    }
  }
}

```

**第一個FileChannel** 用於讀取；**第二個FileChannel** 用於寫入。當 **ByteBuffer** 分配好儲存，呼叫 **FileChannel** 的 `read()` 方法返回 **-1**（毫無疑問，這是來源於 Unix 和 C 語言）時，說明輸入流讀取完了。在每次 `read()` 將資料放入緩衝區之後，`flip()` 都會準備好緩衝區，以便 `write()` 提取它的訊息。在 `write()` 之後，資料仍然在緩衝區中，我們需要 `clear()` 來重設所有內部指標，以便在下一次 `read()` 中接受資料。 

但是，上例並不是處理這種操作的理想方法。方法 `transferTo()` 和 `transferFrom()` 允許你直接連接此通道到彼通道：

```java
// newio/TransferTo.java

// 使用 transferTo() 在通道間傳輸
// {java TransferTo TransferTo.java TransferTo.txt}
import java.nio.channels.*;
import java.io.*;

public class TransferTo {
  public static void main(String[] args) {
    if(args.length != 2) {
      System.out.println(
        "arguments: sourcefile destfile");
      System.exit(1);
    }
    try(
      FileChannel in = new FileInputStream(
        args[0]).getChannel();
      FileChannel out = new FileOutputStream(
        args[1]).getChannel()
    ) {
      in.transferTo(0, in.size(), out);
      // Or:
      // out.transferFrom(in, 0, in.size());
    } catch(IOException e) {
      throw new RuntimeException(e);
    }
  }
}
```

可能不會經常用到，但知道這一點很好。


<!-- Converting Data -->
## 資料轉換


為了將 **GetChannel.java** 文件中的訊息列印出來。在 Java 中，我們每次提取一個位元組的資料並將其轉換為字元。看起來很簡單 —— 如果你有看過 `java.nio.`**CharBuffer**  類，你會發現一個 `toString()` 方法。該方法的作用是“返回一個包含此緩衝區字元的字串”。

既然 **ByteBuffer** 可以透過 **CharBuffer** 類的 `asCharBuffer()` 方法查看，那我們就來嘗試一樣。從下面輸出語句的第一行可以看出，這並不正確：

```java
// newio/BufferToText.java
// (c)2017 MindView LLC: see Copyright.txt
// 我們無法保證該程式碼是否適用於其他用途。
// 訪問 http://OnJava8.com 了解更多本書訊息。
// text 和 ByteBuffers 互轉
import java.nio.*;
import java.nio.channels.*;
import java.nio.charset.*;
import java.io.*;

public class BufferToText {
  private static final int BSIZE = 1024;
  public static void main(String[] args) {

    try(
      FileChannel fc = new FileOutputStream(
        "data2.txt").getChannel()
    ) {
      fc.write(ByteBuffer.wrap("Some text".getBytes()));
    } catch(IOException e) {
      throw new RuntimeException(e);
    }

    ByteBuffer buff = ByteBuffer.allocate(BSIZE);

    try(
      FileChannel fc = new FileInputStream(
        "data2.txt").getChannel()
    ) {
      fc.read(buff);
    } catch(IOException e) {
      throw new RuntimeException(e);
    }

    buff.flip();
    // 無法執行
    System.out.println(buff.asCharBuffer());
    // 使用預設系統預設編碼解碼
    buff.rewind();
    String encoding =
      System.getProperty("file.encoding");
    System.out.println("Decoded using " +
      encoding + ": "
      + Charset.forName(encoding).decode(buff));

    // 編碼和列印
    try(
      FileChannel fc = new FileOutputStream(
        "data2.txt").getChannel()
    ) {
      fc.write(ByteBuffer.wrap(
        "Some text".getBytes("UTF-16BE")));
    } catch(IOException e) {
      throw new RuntimeException(e);
    }

    // 嘗試再次讀取：
    buff.clear();
    try(
      FileChannel fc = new FileInputStream(
        "data2.txt").getChannel()
    ) {
      fc.read(buff);
    } catch(IOException e) {
      throw new RuntimeException(e);
    }

    buff.flip();
    System.out.println(buff.asCharBuffer());
    // 通過 CharBuffer 寫入：
    buff = ByteBuffer.allocate(24);
    buff.asCharBuffer().put("Some text");

    try(
      FileChannel fc = new FileOutputStream(
        "data2.txt").getChannel()
    ) {
      fc.write(buff);
    } catch(IOException e) {
      throw new RuntimeException(e);
    }
    // 讀取和顯示：
    buff.clear();

    try(
      FileChannel fc = new FileInputStream(
        "data2.txt").getChannel()
    ) {
      fc.read(buff);
    } catch(IOException e) {
      throw new RuntimeException(e);
    }

    buff.flip();
    System.out.println(buff.asCharBuffer());
  }
}
```

輸出結果：

```
????
Decoded using windows-1252: Some text
Some text
Some textNULNULNUL
```

緩衝區包含普通位元組，為了將這些位元組轉換為字元，我們必須在輸入時對它們進行編碼(這樣它們輸出時就有意義了)，或者在輸出時對它們進行解碼。我們可以使用 `java.nio.charset.`**Charset** 字元集工具類來完成。程式碼範例：

```java
// newio/AvailableCharSets.java
// (c)2017 MindView LLC: see Copyright.txt
// 我們無法保證該程式碼是否適用於其他用途。
// 訪問 http://OnJava8.com 了解更多本書訊息。
// 展示 Charsets 和 aliases
import java.nio.charset.*;
import java.util.*;

public class AvailableCharSets {

  public static void main(String[] args) {
      SortedMap<String,Charset> charSets =
      Charset.availableCharsets();

    for(String csName : charSets.keySet()) {
      System.out.print(csName);
      Iterator aliases = charSets.get(csName)
        .aliases().iterator();
      if(aliases.hasNext())
        System.out.print(": ");
        
      while(aliases.hasNext()) {
        System.out.print(aliases.next());
        if(aliases.hasNext())
          System.out.print(", ");
      }
      System.out.println();
    }
  }
}
```

輸出結果：

```
Big5: csBig5
Big5-HKSCS: big5-hkscs, big5hk, Big5_HKSCS, big5hkscs
CESU-8: CESU8, csCESU-8
EUC-JP: csEUCPkdFmtjapanese, x-euc-jp, eucjis,
Extended_UNIX_Code_Packed_Format_for_Japanese, euc_jp,
eucjp, x-eucjp
EUC-KR: ksc5601-1987, csEUCKR, ksc5601_1987, ksc5601,
5601,
euc_kr, ksc_5601, ks_c_5601-1987, euckr
GB18030: gb18030-2000
GB2312: gb2312, euc-cn, x-EUC-CN, euccn, EUC_CN,
gb2312-80,
gb2312-1980
                  ...
```



回到 **BufferToText.java** 中，如果你 `rewind()` 緩衝區（回到資料的開頭），使用該平台的預設字元集 `decode()` 資料，那麼生成的 **CharBuffer** 資料將在控制台上正常顯示。可以透過 `System.getProperty(“file.encoding”)` 方法來查看平台預設字元集名稱。傳遞該名稱參數到 `Charset.forName()` 方法可以生成對應的 `Charset` 物件用於解碼字串。

另一種方法是使用字元集 `encode()` 方法，該字元集在讀取文件時生成可列印的內容，如你在 **BufferToText.java**  的第三部分中所看到的。上例中，**UTF-16BE** 被用於將文字寫入檔案，當文字被讀取時，你所要做的就是將其轉換為 **CharBuffer**，並生成預期的文字。

最後，如果將 **CharBuffer** 寫入 **ByteBuffer**，你會看到發生了什麼(更多詳情，稍後了解)。**注意**，為 **ByteBuffer** 分配了24個位元組，按照每個字元占用 2 個自位元組， 12 個字元的空間已經足夠了。由於“some text”只有 9 個字元，受其 `toString()` 方法影響，剩下的 0 位元組部分也出現在了 **CharBuffer** 的展示中，如輸出所示。


<!-- Fetching Primitives -->
## 基本類型獲取


雖然 **ByteBuffer** 只包含位元組，但它包含了一些方法，用於從其所包含的位元組中生成各種不同的基本類型資料。程式碼範例：

```java
// newio/GetData.java
// (c)2017 MindView LLC: see Copyright.txt
// 我們無法保證該程式碼是否適用於其他用途。
// 訪問 http://OnJava8.com 了解更多本書訊息。
// 從 ByteBuffer 中獲取不同的資料展示
import java.nio.*;

public class GetData {
  private static final int BSIZE = 1024;
  public static void main(String[] args) {
    ByteBuffer bb = ByteBuffer.allocate(BSIZE);
    // 自動分配 0 到 ByteBuffer:
    int i = 0;
    while(i++ < bb.limit())
      if(bb.get() != 0)
        System.out.println("nonzero");
    System.out.println("i = " + i);
    bb.rewind();
    // 儲存和讀取 char 陣列:
    bb.asCharBuffer().put("Howdy!");
    char c;
    while((c = bb.getChar()) != 0)
      System.out.print(c + " ");
    System.out.println();
    bb.rewind();
    // 儲存和讀取 short:
    bb.asShortBuffer().put((short)471142);
    System.out.println(bb.getShort());
    bb.rewind();
    // 儲存和讀取 int:
    bb.asIntBuffer().put(99471142);
    System.out.println(bb.getInt());
    bb.rewind();
    // 儲存和讀取 long:
    bb.asLongBuffer().put(99471142);
    System.out.println(bb.getLong());
    bb.rewind();
    // 儲存和讀取 float:
    bb.asFloatBuffer().put(99471142);
    System.out.println(bb.getFloat());
    bb.rewind();
    // 儲存和讀取 double:
    bb.asDoubleBuffer().put(99471142);
    System.out.println(bb.getDouble());
    bb.rewind();
  }
}
```

輸出結果：

```
i = 1025
H o w d y !
12390
99471142
99471142
9.9471144E7
9.9471142E7
```

在分配 **ByteBuffer** 之後，我們檢查並確認它的 1,024 元素被初始化為 0。（截至到達 `limit()` 結果的位置）。

將基本類型資料插入 **ByteBuffer** 的最簡單方法就是使用 `asCharBuffer()`、`asShortBuffer()` 等方法獲取該緩衝區適當的“檢視”（View），然後呼叫該“檢視”的 `put()` 方法。

這是針對每種基本資料類型執行的。其中唯一有點奇怪的是 **ShortBuffer** 的 `put()`，它需要類型強制轉換。其他檢視緩衝區不需要在其 `put()` 方法中進行轉換。


<!-- View Buffers -->
## 檢視緩衝區


“檢視緩衝區”（view buffer）是透過特定的基本類型的視窗來查看底層 **ByteBuffer**。**ByteBuffer** 仍然是“支援”檢視的實際儲存，因此對檢視所做的任何更改都反映在對 **ByteBuffer** 中的資料的修改中。

如前面的範例所示，這方便地將基本類型插入 **ByteBuffer**。檢視緩衝區還可以從 **ByteBuffer** 讀取基本類型資料，每次單個（**ByteBuffer** 規定），或者批次讀取到陣列。下面是一個透過 **IntBuffer** 在 **ByteBuffer** 中操作 **int** 的例子：

```java
// newio/IntBufferDemo.java
// (c)2017 MindView LLC: see Copyright.txt
// 我們無法保證該程式碼是否適用於其他用途。
// 訪問 http://OnJava8.com 了解更多本書訊息。
// 利用 IntBuffer 儲存 int 資料到 ByteBuffer
import java.nio.*;

public class IntBufferDemo {
  private static final int BSIZE = 1024;
  public static void main(String[] args) {
    ByteBuffer bb = ByteBuffer.allocate(BSIZE);
    IntBuffer ib = bb.asIntBuffer();
    // 儲存 int 陣列：
    ib.put(new int[]{ 11, 42, 47, 99, 143, 811, 1016 });
    //絕對位置讀寫：
    System.out.println(ib.get(3));
    ib.put(3, 1811);
    // 在重設緩衝區前設定新的限制
    
    ib.flip();
    while(ib.hasRemaining()) {
      int i = ib.get();
      System.out.println(i);
    }
  }
}
```

輸出結果：

```
99
11
42
47
1811
143
811
1016
```

`put()` 方法重載，首先用於儲存 **int** 陣列。下面的 `get()` 和 `put()` 方法呼叫直接訪問底層 **ByteBuffer** 中的 **int** 位置。**注意**，透過直接操作 **ByteBuffer** ，這些絕對位置訪問也可以用於基本類型。

一旦底層 **ByteBuffer** 透過檢視緩衝區填充了 **int** 或其他基本類型，那麼就可以直接將該 **ByteBuffer** 寫入通道。你可以輕鬆地從通道讀取資料，並使用檢視緩衝區將所有內容轉換為特定的基本類型。下面是一個例子，透過在同一個 **ByteBuffer** 上生成不同的檢視緩衝區，將相同的位元組序列解釋為 **short**、**int**、**float**、**long** 和 **double**。程式碼範例：

```java
// newio/ViewBuffers.java
// (c)2017 MindView LLC: see Copyright.txt
// 我們無法保證該程式碼是否適用於其他用途。
// 訪問 http://OnJava8.com 了解更多本書訊息。
import java.nio.*;

public class ViewBuffers {
  public static void main(String[] args) {
    ByteBuffer bb = ByteBuffer.wrap(
      new byte[]{ 0, 0, 0, 0, 0, 0, 0, 'a' });
    bb.rewind();
    System.out.print("Byte Buffer ");
    while(bb.hasRemaining())
      System.out.print(
        bb.position()+ " -> " + bb.get() + ", ");
    System.out.println();
    CharBuffer cb =
      ((ByteBuffer)bb.rewind()).asCharBuffer();
    System.out.print("Char Buffer ");
    while(cb.hasRemaining())
      System.out.print(
        cb.position() + " -> " + cb.get() + ", ");
    System.out.println();
    FloatBuffer fb =
      ((ByteBuffer)bb.rewind()).asFloatBuffer();
    System.out.print("Float Buffer ");
    while(fb.hasRemaining())
      System.out.print(
        fb.position()+ " -> " + fb.get() + ", ");
    System.out.println();
    IntBuffer ib =
      ((ByteBuffer)bb.rewind()).asIntBuffer();
    System.out.print("Int Buffer ");
    while(ib.hasRemaining())
      System.out.print(
        ib.position()+ " -> " + ib.get() + ", ");
    System.out.println();
    LongBuffer lb =
      ((ByteBuffer)bb.rewind()).asLongBuffer();
    System.out.print("Long Buffer ");
    while(lb.hasRemaining())
      System.out.print(
        lb.position()+ " -> " + lb.get() + ", ");
    System.out.println();
    ShortBuffer sb =
      ((ByteBuffer)bb.rewind()).asShortBuffer();
    System.out.print("Short Buffer ");
    while(sb.hasRemaining())
      System.out.print(
        sb.position()+ " -> " + sb.get() + ", ");
    System.out.println();
    DoubleBuffer db =
      ((ByteBuffer)bb.rewind()).asDoubleBuffer();
    System.out.print("Double Buffer ");
    while(db.hasRemaining())
      System.out.print(
        db.position()+ " -> " + db.get() + ", ");
  }
}
```

輸出結果：

```
Byte Buffer 0 -> 0, 1 -> 0, 2 -> 0, 3 -> 0, 4 -> 0, 5
-> 0, 6 -> 0, 7 -> 97,
Char Buffer 0 -> NUL, 1 -> NUL, 2 -> NUL, 3 -> a,
Float Buffer 0 -> 0.0, 1 -> 1.36E-43,
Int Buffer 0 -> 0, 1 -> 97,
Long Buffer 0 -> 97,
Short Buffer 0 -> 0, 1 -> 0, 2 -> 0, 3 -> 97,
Double Buffer 0 -> 4.8E-322,
```


**ByteBuffer** 通過“包裝”一個 8 位元組陣列生成，然後透過所有不同基本類型的檢視緩衝區顯示該陣列。下圖顯示了從不同類型的緩衝區讀取資料時，資料顯示的差異：

![1554546258113](../images/1554546258113.png)

<!-- Endians -->
### 位元組儲存次序

不同的機器可以使用不同的位元組儲存順序（Endians）來儲存資料。“高位優先”（Big Endian）：將最重要的位元組放在最低記憶體地址中，而“低位優先”（Little Endian）：將最重要的位元組放在最高記憶體地址中。

當儲存大於單位元組的資料時，如 **int**、**float** 等，我們可能需要考慮位元組排序問題。**ByteBuffer** 以“高位優先”形式儲存資料；透過網路發送的資料總是使用“高位優先”形式。我們可以 使用 **ByteOrder** 的 `order()` 方法和參數 **ByteOrder.BIG_ENDIAN** 或 **ByteOrder.LITTLE_ENDIAN** 來改變它的位元組儲存次序。

下例是一個包含兩個位元組的 **ByteBuffer** ：

![1554546378822](../images/1554546378822.png)


將資料作為 **short** 型來讀取（`ByteBuffer.asshortbuffer()`)），生成數字 97 （00000000 01100001）。更改為“低位優先”後 將生成數字 24832 （01100001 00000000）。

這顯示了位元組順序的變化取決於位元組儲存次序設定:

```java
// newio/Endians.java
// (c)2017 MindView LLC: see Copyright.txt
// 我們無法保證該程式碼是否適用於其他用途。
// 訪問 http://OnJava8.com 了解更多本書訊息。
// 不同位元組儲存次序的儲存
import java.nio.*;
import java.util.*;

public class Endians {
  public static void main(String[] args) {
    ByteBuffer bb = ByteBuffer.wrap(new byte[12]);
    bb.asCharBuffer().put("abcdef");
    System.out.println(Arrays.toString(bb.array()));
    bb.rewind();
    bb.order(ByteOrder.BIG_ENDIAN);
    bb.asCharBuffer().put("abcdef");
    System.out.println(Arrays.toString(bb.array()));
    bb.rewind();
    bb.order(ByteOrder.LITTLE_ENDIAN);
    bb.asCharBuffer().put("abcdef");
    System.out.println(Arrays.toString(bb.array()));
  }
}
```

輸出結果：

```
[0, 97, 0, 98, 0, 99, 0, 100, 0, 101, 0, 102]
[0, 97, 0, 98, 0, 99, 0, 100, 0, 101, 0, 102]
[97, 0, 98, 0, 99, 0, 100, 0, 101, 0, 102, 0]
```

**ByteBuffer** 分配空間將 **charArray** 中的所有位元組作為外部緩衝區儲存，因此可以呼叫 `array()` 方法來顯示底層位元組。`array()` 方法是“可選的”，你只能在陣列支援的緩衝區上呼叫它，否則將拋出 **UnsupportedOperationException** 異常。

**charArray** 通過 **CharBuffer** 檢視插入到 **ByteBuffer** 中。當顯示底層位元組時，預設排序與後續“高位”相同，而“地位”交換位元組

<!-- Data Manipulation with Buffers -->
## 緩衝區資料操作


下圖說明了 **nio** 類之間的關係，展示了如何移動和轉換資料。例如，要將位元組陣列寫入檔案，使用 **ByteBuffer.**`wrap()` 方法包裝位元組陣列，使用 `getChannel()` 在 **FileOutputStream** 上打開通道，然後從 **ByteBuffer** 將資料寫入 **FileChannel**。

![1554546452861](../images/1554546452861.png)

**ByteBuffer** 是將資料移入和移出通道的唯一方法，我們只能建立一個獨立的基本類型緩衝區，或者使用 `as` 方法從 **ByteBuffer** 獲得一個新緩衝區。也就是說，不能將基本類型緩衝區轉換為 **ByteBuffer**。但我們能夠透過檢視緩衝區將基本類型資料移動到 **ByteBuffer** 中或移出 **ByteBuffer**。

 

### 緩衝區細節

緩衝區由資料和四個索引組成，以有效地訪問和操作該資料：mark、position、limit 和 capacity（標記、位置、限制和容量）。伴隨著的還有一組方法可以設定和重設這些索引，並可查詢它們的值。

|  |  |
| :----- | :----- |
| **capacity()** | 返回緩衝區的 capacity |
|**clear()** |清除緩衝區，將 position 設定為零並 設 limit 為 capacity;可呼叫此方法來覆蓋現有緩衝區|
|**flip()** | 將 limit 設定為 position，並將 position 設定為 0;此方法用於準備緩衝區，以便在資料寫入緩衝區後進行讀取|
|**limit()** |返回 limit 的值|
|**limit(int limit)**| 重設 limit|
|**mark()**  |設定 mark 為目前的 position |
|**position()** |返回 position |
|**position(int pos)**| 設定 position|
|**remaining()** |返回 limit 到 position |
|**hasRemaining()**| 如果在 position 與 limit 中間有元素，返回 `true`|

從緩衝區插入和提取資料的方法透過更新索引來反映所做的更改。下例使用一種非常簡單的演算法（交換相鄰字元）來對 **CharBuffer** 中的字元進行加擾和解擾。程式碼範例：

```java
// newio/UsingBuffers.java
// (c)2017 MindView LLC: see Copyright.txt
// 我們無法保證該程式碼是否適用於其他用途。
// 訪問 http://OnJava8.com 了解更多本書訊息。
import java.nio.*;

public class UsingBuffers {
  private static
  void symmetricScramble(CharBuffer buffer) {
    while(buffer.hasRemaining()) {
      buffer.mark();
      char c1 = buffer.get();
      char c2 = buffer.get();
      buffer.reset();
      buffer.put(c2).put(c1);
    }
  }

  public static void main(String[] args) {
    char[] data = "UsingBuffers".toCharArray();
    ByteBuffer bb =
      ByteBuffer.allocate(data.length * 2);
    CharBuffer cb = bb.asCharBuffer();
    cb.put(data);
    System.out.println(cb.rewind());
    symmetricScramble(cb);
    System.out.println(cb.rewind());
    symmetricScramble(cb);
    System.out.println(cb.rewind());
  }
}
```

輸出結果：

```
UsingBuffers
sUniBgfuefsr
UsingBuffers
```

雖然可以透過使用 **char** 陣列呼叫 `wrap()` 直接生成 **CharBuffer**，但是底層的 **ByteBuffer** 將被分配，而 **CharBuffer** 將作為 **ByteBuffer** 上的檢視生成。這強調了目標始終是操作 **ByteBuffer**，因為它與通道互動。

下面是程式在 `symmetricgrab()` 方法入口時緩衝區的樣子:

![1554546627710](../images/1554546627710.png)

position 指向緩衝區中的第一個元素，capacity 和 limit 緊接在最後一個元素之後。在`symmetricgrab()` 中，**while** 循環疊代到 position 等於 limit。當在緩衝區上呼叫相對位置的 `get()` 或 `put()` 函數時，緩衝區的位置會發生變化。你可以呼叫絕對位置的 `get()` 和 `put()` 方法，它們包含索引參數：`get()` 或 `put()` 發生的位置。這些方法不修改緩衝區 position 的值。

當控制項進入 **while** 循環時，使用 `mark()` 設定 mark 的值。緩衝區的狀態為：

![1554546666685](../images/1554546666685.png)

兩個相對 `get()` 呼叫將前兩個字元的值儲存在變數 `c1` 和 `c2` 中。在這兩個呼叫之後，緩衝區看起來是這樣的：

![1554546693664](../images/1554546693664.png)

為了執行交換，我們在位置 0 處編寫 `c2`，在位置 1 處編寫 `c1`。我們可以使用絕對 `put()` 方法來實現這一點，或者用 `reset()` 方法，將 position 的值設定為 mark：

![1554546847181](../images/1554546847181.png)

兩個 `put()` 方法分別編寫 `c2` 和 `c1` ：

![1554546861836](../images/1554546861836.png)

在下一次循環中，將 mark 設定為 position 的目前值：

![1554546881189](../images/1554546881189.png)

 該過程將繼續，直到遍歷整個緩衝區為止。在 **while** 循環的末尾，position 位於緩衝區的末尾。如果顯示緩衝區，則只顯示位置和限制之間的字元。因此，要顯示緩衝區的全部內容，必須使用 `rewind()` 將 position 設定為緩衝區的開始位置。這是 `rewind()` 呼叫後緩衝區的狀態（mark 的值變成未定義）：

![1554546890132](../images/1554546890132.png)

再次呼叫 `symmetricgrab()` 方法時，**CharBuffer** 將經歷相同的過程並復原到原始狀態。


<!-- Memory-Mapped Files -->
##  記憶體映射文件

記憶體映射文件能讓你建立和修改那些因為太大而無法放入記憶體的文件。有了記憶體映射文件，你就可以認為文件已經全部讀進了記憶體，然後把它當成一個非常大的陣列來訪問。這種解決辦法能大大簡化修改文件的程式碼：

```java
// newio/LargeMappedFiles.java
// (c)2017 MindView LLC: see Copyright.txt
// 我們無法保證該程式碼是否適用於其他用途。
// 訪問 http://OnJava8.com 了解更多本書訊息。
// 使用記憶體映射來建立一個大文件
import java.nio.*;
import java.nio.channels.*;
import java.io.*;

public class LargeMappedFiles {
  static int length = 0x8000000; // 128 MB
  public static void
  main(String[] args) throws Exception {
    try(
      RandomAccessFile tdat =
        new RandomAccessFile("test.dat", "rw")
    ) {
      MappedByteBuffer out = tdat.getChannel().map(
        FileChannel.MapMode.READ_WRITE, 0, length);
      for(int i = 0; i < length; i++)
        out.put((byte)'x');
      System.out.println("Finished writing");
      for(int i = length/2; i < length/2 + 6; i++)
        System.out.print((char)out.get(i));
    }
  }
}
```

輸出結果：

```
Finished writing
xxxxxx
```

為了讀寫，我們從 **RandomAccessFile** 開始，獲取該文件的通道，然後呼叫 `map()` 來生成 **MappedByteBuffer** ，這是一種特殊的直接緩衝區。你必須指定要在文件中映射的區域的起始點和長度—這意味著你可以選擇映射大文件的較小區域。

**MappedByteBuffer**  繼承了 **ByteBuffer**，所以擁有**ByteBuffer** 全部的方法。這裡只展示了 `put()` 和 `get()` 的最簡單用法，但是你也可以使用 `asCharBuffer()` 等方法。

使用前面的程式建立的文件長度為 128MB，可能比你的作業系統單次所允許的操作的記憶體要大。該文件似乎可以同時訪問，因為它只有一部分被帶進記憶體，而其他部分被交換出去。這樣，一個非常大的文件（最多 2GB）可以很容易地修改。**注意**，作業系統底層的文件映射工具用於性能的最大化。

<!-- Performance -->
### 性能

雖然舊的 I/O 流的效能透過使用 **NIO** 實現得到了改進，但是映射文件訪問往往要快得多。下例帶來一個簡單的效能比較。程式碼範例：

```java
// newio/MappedIO.java
// 我們無法保證該程式碼是否適用於其他用途。
// 訪問 http://OnJava8.com 了解更多本書訊息。
import java.util.*;
import java.nio.*;
import java.nio.channels.*;
import java.io.*;

public class MappedIO {
  private static int numOfInts =      4_000_000;
  private static int numOfUbuffInts = 100_000;
  private abstract static class Tester {
    private String name;
    Tester(String name) {
      this.name = name;
    }

    public void runTest() {
      System.out.print(name + ": ");
      long start = System.nanoTime();
      test();
      double duration = System.nanoTime() - start;
      System.out.format("%.3f%n", duration/1.0e9);
    }

    public abstract void test();
  }

  private static Tester[] tests = {
    new Tester("Stream Write") {
      @Override
      public void test() {
        try(
          DataOutputStream dos =
            new DataOutputStream(
              new BufferedOutputStream(
                new FileOutputStream(
                  new File("temp.tmp"))))
        ) {
          for(int i = 0; i < numOfInts; i++)
            dos.writeInt(i);
        } catch(IOException e) {
          throw new RuntimeException(e);
        }
      }
    },
    new Tester("Mapped Write") {
      @Override
      public void test() {
        try(
          FileChannel fc =
            new RandomAccessFile("temp.tmp", "rw")
              .getChannel()
        ) {
          IntBuffer ib =
            fc.map(FileChannel.MapMode.READ_WRITE,
              0, fc.size()).asIntBuffer();
          for(int i = 0; i < numOfInts; i++)
            ib.put(i);
        } catch(IOException e) {
          throw new RuntimeException(e);
        }
      }
    },
    new Tester("Stream Read") {
      @Override
      public void test() {
        try(
          DataInputStream dis =
            new DataInputStream(
              new BufferedInputStream(
                new FileInputStream("temp.tmp")))
        ) {
          for(int i = 0; i < numOfInts; i++)
            dis.readInt();
        } catch(IOException e) {
          throw new RuntimeException(e);
        }
      }
    },
    new Tester("Mapped Read") {
      @Override
      public void test() {
        try(
          FileChannel fc = new FileInputStream(
            new File("temp.tmp")).getChannel()
        ) {
          IntBuffer ib =
            fc.map(FileChannel.MapMode.READ_ONLY,
              0, fc.size()).asIntBuffer();
          while(ib.hasRemaining())
            ib.get();
        } catch(IOException e) {
          throw new RuntimeException(e);
        }
      }
    },
    new Tester("Stream Read/Write") {
      @Override
      public void test() {
        try(
          RandomAccessFile raf =
            new RandomAccessFile(
              new File("temp.tmp"), "rw")
        ) {
          raf.writeInt(1);
          for(int i = 0; i < numOfUbuffInts; i++) {
            raf.seek(raf.length() - 4);
            raf.writeInt(raf.readInt());
          }
        } catch(IOException e) {
          throw new RuntimeException(e);
        }
      }
    },
    new Tester("Mapped Read/Write") {
      @Override
      public void test() {
        try(
          FileChannel fc = new RandomAccessFile(
            new File("temp.tmp"), "rw").getChannel()
        ) {
          IntBuffer ib =
            fc.map(FileChannel.MapMode.READ_WRITE,
              0, fc.size()).asIntBuffer();
          ib.put(0);
          for(int i = 1; i < numOfUbuffInts; i++)
            ib.put(ib.get(i - 1));
        } catch(IOException e) {
          throw new RuntimeException(e);
        }
      }
    }
  };
  public static void main(String[] args) {
    Arrays.stream(tests).forEach(Tester::runTest);
  }
}
```

輸出結果：

```
Stream Write: 0.615
Mapped Write: 0.050
Stream Read: 0.577
Mapped Read: 0.015
Stream Read/Write: 4.069
Mapped Read/Write: 0.013
```

**Tester** 使用了模板方法（Template Method）模式，它為匿名內部子類中定義的 `test()` 的各種實現建立一個測試框架。每個子類都執行一種測試，因此 `test()` 方法還提供了執行各種I/O 活動的原型。

雖然映射的寫似乎使用 **FileOutputStream**，但是文件映射中的所有輸出必須使用 **RandomAccessFile**，就像前面程式碼中的讀/寫一樣。

請注意，`test()` 方法包括初始化各種 I/O 物件的時間，因此，儘管映射文件的設定可能很昂貴，但是與流 I/O 相比，總體收益非常可觀。

<!-- File Locking -->
## 文件鎖定

文件鎖定可同步訪問，因此文件可以共享資源。但是，爭用同一文件的兩個執行緒可能位於不同的 JVM 中，或者一個可能是 Java 執行緒，另一個可能是作業系統中的本機執行緒。文件鎖對其他作業系統行程可見，因為 Java 文件鎖定直接映射到本機作業系統鎖定工具。

```java
// newio/FileLocking.java
// (c)2017 MindView LLC: see Copyright.txt
// 我們無法保證該程式碼是否適用於其他用途。
// 訪問 http://OnJava8.com 了解更多本書訊息。
import java.nio.channels.*;
import java.util.concurrent.*;
import java.io.*;

public class FileLocking {
  public static void main(String[] args) {
    try(
      FileOutputStream fos =
        new FileOutputStream("file.txt");
      FileLock fl = fos.getChannel().tryLock()
    ) {
      if(fl != null) {
        System.out.println("Locked File");
        TimeUnit.MILLISECONDS.sleep(100);
        fl.release();
        System.out.println("Released Lock");
      }
    } catch(IOException | InterruptedException e) {
      throw new RuntimeException(e);
    }
  }
}
```

輸出結果：

```
Locked File
Released Lock
```

透過呼叫 **FileChannel** 上的 `tryLock()` 或 `lock()`，可以獲得整個文件的 **FileLock**。（**SocketChannel**、**DatagramChannel** 和 **ServerSocketChannel** 不需要鎖定，因為它們本質上是單行程實體；通常不會在兩個行程之間共享一個網路通訊端）。

`tryLock()` 是非阻塞的。它試圖獲取鎖，若不能獲取（當其他行程已經持有相同的鎖，並且它不是共享的），它只是從方法呼叫返回。

`lock()` 會阻塞，直到獲得鎖，或者呼叫 `lock()` 的執行緒中斷，或者呼叫 `lock()` 方法的通道關閉。使用 **FileLock.**`release()` 釋放鎖。

還可以使用

 > `tryLock(long position, long size, boolean shared)`

  或 

 > `lock(long position, long size, boolean shared)` 
 
 鎖定文件的一部分，鎖住 **size-position** 區域。第三個參數指定是否共享此鎖。

雖然零參數鎖定方法適應檔案大小的變化，但是如果檔案大小發生變化，具有固定大小的鎖不會發生變化。如果從一個位置到另一個位置獲得一個鎖，並且文件的增長超過了 position + size ，那麼超出 position + size 的部分沒有被鎖定。零參數鎖定方法鎖定整個文件，即使它在增長。

底層作業系統必須提供對獨占鎖或共享鎖的支援。如果作業系統不支援共享鎖並且對一個作業系統發出請求，則使用獨占鎖。可以使用 **FileLock.**`isShared()` 查詢鎖的類型（共享或獨占）。

<!-- Locking Portions of a Mapped File -->
### 映射文件的部分鎖定

文件映射通常用於非常大的文件。你可能需要鎖定此類文件的某些部分，以便其他行程可以修改未鎖定的部分。例如，資料庫必須同時對許多使用者可用。這裡你可以看到兩個執行緒，每個執行緒都鎖定文件的不同部分:


```java
// newio/LockingMappedFiles.java
// (c)2017 MindView LLC: see Copyright.txt
// 我們無法保證該程式碼是否適用於其他用途。
// 訪問 http://OnJava8.com 了解更多本書訊息。
// Locking portions of a mapped file
import java.nio.*;
import java.nio.channels.*;
import java.io.*;

public class LockingMappedFiles {
  static final int LENGTH = 0x8FFFFFF; // 128 MB
  static FileChannel fc;
  public static void
  main(String[] args) throws Exception {
    fc = new RandomAccessFile("test.dat", "rw")
      .getChannel();
    MappedByteBuffer out = fc.map(
      FileChannel.MapMode.READ_WRITE, 0, LENGTH);
    for(int i = 0; i < LENGTH; i++)
      out.put((byte)'x');
    new LockAndModify(out, 0, 0 + LENGTH/3);
    new LockAndModify(
      out, LENGTH/2, LENGTH/2 + LENGTH/4);
  }

  private static class LockAndModify extends Thread {
    private ByteBuffer buff;
    private int start, end;
    LockAndModify(ByteBuffer mbb, int start, int end) {
      this.start = start;
      this.end = end;
      mbb.limit(end);
      mbb.position(start);
      buff = mbb.slice();
      start();
    }

    @Override
    public void run() {
      try {
        // Exclusive lock with no overlap:
        FileLock fl = fc.lock(start, end, false);
        System.out.println(
          "Locked: "+ start +" to "+ end);
        // Perform modification:
        while(buff.position() < buff.limit() - 1)
          buff.put((byte)(buff.get() + 1));
        fl.release();
        System.out.println(
          "Released: " + start + " to " + end);
      } catch(IOException e) {
        throw new RuntimeException(e);
      }
    }
  }
}
```

輸出結果：

```
Locked: 75497471 to 113246206
Locked: 0 to 50331647
Released: 75497471 to 113246206
Released: 0 to 50331647
```


**LockAndModify** 執行緒類設定緩衝區並建立要修改的 `slice()`，在 `run()` 中，鎖在文件通道上獲取（不能在緩衝區上獲取鎖—只能在通道上獲取鎖）。`lock()` 的呼叫非常類似於獲取物件上的執行緒鎖 —— 現在有了一個“臨界區”，可以對文件的這部分進行獨占訪問。[^1]

當 JVM 退出或關閉獲取鎖的通道時，鎖會自動釋放，但是你也可以顯式地呼叫 **FileLock** 物件上的 `release()`，如上所示。


<!-- 腳註 -->
[^1]:更多詳情可參考[附錄:並發底層原理](./book/Appendix-Low-Level-Concurrency.md)。

<!-- 分頁 -->
<div style="page-break-after: always;"></div>
