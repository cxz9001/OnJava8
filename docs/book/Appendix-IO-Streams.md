[TOC]

<!-- Appendix: I/O Streams -->
# 附錄:流式IO

> Java 7 引入了一種簡單明瞭的方式來讀寫文件和操作目錄。大多情況下，[文件](./17-Files.md)這一章所介紹的那些庫和技術就足夠你用了。但是，如果你必須面對一些特殊的需求和比較底層的操作，或者處理一些老版本的程式碼，那麼你就必須了解本附錄中的內容。

對於程式語言的設計者來說，實現良好的輸入/輸出（I/O）系統是一項比較艱難的任務，不同實現方案的數量就可以證明這點。其中的挑戰似乎在於要涵蓋所有的可能性，你不僅要覆蓋到不同的 I/O 源和 I/O 接收器（如文件、控制台、網路連接等），還要實現多種與它們進行通信的方式（如順序、隨機訪問、緩衝、二進位制、字元、按行和按字等）。

Java 類庫的設計者透過建立大量的類來解決這一難題。一開始，你可能會對 Java I/O 系統提供了如此多的類而感到不知所措。Java 1.0 之後，Java 的 I/O 類庫發生了明顯的改變，在原來面向位元組的類中添加了面向字元和基於 Unicode 的類。在 Java 1.4 中，為了改進性能和功能，又添加了 `nio` 類（全稱是 “new I/O”，Java 1.4 引入，到現在已經很多年了）。這部分在[附錄：新 I/O](./Appendix-New-IO.md) 中介紹。

因此，要想充分理解 Java I/O 系統以便正確運用它，我們需要學習一定數量的類。另外，理解 I/O 類庫的演化過程也很有必要，因為如果缺乏歷史的眼光，很快我們就會對什麼時候該使用哪些類，以及什麼時候不該使用它們而感到困惑。

程式語言的 I/O 類庫經常使用**流**這個抽象概念，它將所有資料源或者資料接收器表示為能夠產生或者接收資料片的物件。

> **注意**：Java 8 函數式編程中的 `Stream` 類和這裡的 I/O stream 沒有任何關係。這又是另一個例子，如果再給設計者一次重來的機會，他們將使用不同的術語。

I/O 流封鎖了實際的 I/O 裝置中處理資料的細節：

1. 位元組流對應原生的二進位制資料；
2. 字元流對應字元資料，它會自動處理與本機字元集之間的轉換；
3. 緩衝流可以提高性能，透過減少底層 API 的呼叫次數來最佳化 I/O。

從 JDK 文件的類層次結構中可以看到，Java 類庫中的 I/O 類分成了輸入和輸出兩部分。在設計 Java 1.0 時，類庫的設計者們就決定讓所有與輸入有關係的類都繼承自 `InputStream`，所有與輸出有關係的類都繼承自 `OutputStream`。所有從 `InputStream` 或 `Reader` 衍生而來的類都含有名為 `read()` 的基本方法，用於讀取單個位元組或者位元組陣列。同樣，所有從 `OutputStream` 或 `Writer` 衍生而來的類都含有名為 `write()` 的基本方法，用於寫單個位元組或者位元組陣列。但是，我們通常不會用到這些方法，它們之所以存在是因為別的類可以使用它們，以便提供更有用的介面。

我們很少使用單一的類來建立流物件，而是透過疊合多個物件來提供所期望的功能（這是**裝飾器設計模式**）。為了建立一個流，你卻要建立多個物件，這也是 Java I/O 類庫讓人困惑的主要原因。

這裡我只會提供這些類的概述，並假定你會使用 JDK 文件來獲取它們的詳細訊息（比如某個類的所以方法的詳細列表）。

<!-- Types of InputStream -->
## 輸入流類型

`InputStream` 表示那些從不同資料源產生輸入的類，如[表 I/O-1](#table-io-1) 所示，這些資料源包括：

1. 位元組陣列；
2. `String` 物件；
3. 文件；
4. “管道”，工作方式與實際生活中的管道類似：從一端輸入，從另一端輸出；
5. 一個由其它種類的流組成的序列，然後我們可以把它們匯聚成一個流；
6. 其它資料源，如 Internet 連接。

每種資料源都有相應的 `InputStream` 子類。另外，`FilterInputStream` 也屬於一種 `InputStream`，它的作用是為“裝飾器”類提供基類。其中，“裝飾器”類可以把屬性或有用的介面與輸入流連接在一起，這個我們稍後再討論。

<span id="table-io-1">**表 I/O-1 `InputStream` 類型**</span>

|  類  | 功能 | 構造器參數 | 如何使用 |
| :--: | :-- | :-------- | :----- |
| `ByteArrayInputStream`    | 允許將記憶體的緩衝區當做 `InputStream` 使用 | 緩衝區，位元組將從中取出 | 作為一種資料源：將其與 `FilterInputStream` 物件相連以提供有用介面 |
| `StringBufferInputStream` | 將 `String` 轉換成 `InputStream` | 字串。底層實現實際使用 `StringBuffer` | 作為一種資料源：將其與 `FilterInputStream` 物件相連以提供有用介面 |
| `FileInputStream`         | 用於從文件中讀取訊息 | 字串，表示檔案名、文件或 `FileDescriptor` 物件 | 作為一種資料源：將其與 `FilterInputStream` 物件相連以提供有用介面 |
| `PipedInputStream`        | 產生用於寫入相關 `PipedOutputStream` 的資料。實現“管道化”概念 | `PipedOutputSteam`           | 作為多執行緒中的資料源：將其與 `FilterInputStream` 物件相連以提供有用介面 |
| `SequenceInputStream`     | 將兩個或多個 `InputStream` 物件轉換成一個 `InputStream` | 兩個 `InputStream` 物件或一個容納 `InputStream` 物件的容器 `Enumeration` | 作為一種資料源：將其與 `FilterInputStream` 物件相連以提供有用介面          |
| `FilterInputStream`       | 抽象類，作為“裝飾器”的介面。其中，“裝飾器”為其它的 `InputStream` 類提供有用的功能。見[表 I/O-3](#table-io-3) | 見[表 I/O-3](#table-io-3) | 見[表 I/O-3](#table-io-3) |

<!-- Types of OutputStream -->
## 輸出流類型

如[表 I/O-2](#table-io-2) 所示，該類別的類決定了輸出所要去往的目標：位元組陣列（但不是 `String`，當然，你也可以用位元組陣列自己建立）、文件或管道。

另外，`FilterOutputStream` 為“裝飾器”類提供了一個基類，“裝飾器”類把屬性或者有用的介面與輸出流連接了起來，這些稍後會討論。

<span id="table-io-2">**表 I/O-2：`OutputStream` 類型**</span>

| 類 | 功能 | 構造器參數 | 如何使用 |
| :--: | :-- | :-------- | :----- |
| `ByteArrayOutputStream` | 在記憶體中建立緩衝區。所有送往“流”的資料都要放置在此緩衝區 | 緩衝區初始大小（可選） | 用於指定資料的目的地：將其與 `FilterOutputStream` 物件相連以提供有用介面 |
| `FileOutputStream`      | 用於將訊息寫入檔案 | 字串，表示檔案名、文件或 `FileDescriptor` 物件 | 用於指定資料的目的地：將其與 `FilterOutputStream` 物件相連以提供有用介面 |
| `PipedOutputStream`     | 任何寫入其中的訊息都會自動作為相關 `PipedInputStream` 的輸出。實現“管道化”概念 | `PipedInputStream` | 指定用於多執行緒的資料的目的地：將其與 `FilterOutputStream` 物件相連以提供有用介面 |
| `FilterOutputStream`    | 抽象類，作為“裝飾器”的介面。其中，“裝飾器”為其它 `OutputStream` 提供有用功能。見[表 I/O-4](#table-io-4) | 見[表 I/O-4](#table-io-4) | 見[表 I/O-4](#table-io-4) |

<!-- Adding Attributes and Useful Interfaces -->

## 添加屬性和有用的介面

裝飾器在[泛型](./20-Generics.md)這一章引入。Java I/O 類庫需要多種不同功能的組合，這正是使用裝飾器模式的原因所在[^1]。而之所以存在 **filter**（過濾器）類，是因為讓抽象類 **filter** 作為所有裝飾器類的基類。裝飾器必須具有和它所裝飾物件相同的介面，但它也可以擴展介面，不過這種情況只發生在個別 **filter** 類中。

但是，裝飾器模式也有一個缺點：在編寫程式的時候，它給我們帶來了相當多的靈活性（因為我們可以很容易地對屬性進行混搭），但它同時也增加了程式碼的複雜性。Java I/O 類庫操作不便的原因在於：我們必須建立許多類（“核心” I/O 類型加上所有的裝飾器）才能得到我們所希望的單個 I/O 物件。

`FilterInputStream` 和 `FilterOutputStream` 是用來提供裝飾器類介面以控制特定輸入流 `InputStream` 和 輸出流 `OutputStream` 的兩個類，但它們的名字並不是很直觀。`FilterInputStream` 和 `FilterOutputStream` 分別從 I/O 類庫中的基類 `InputStream` 和 `OutputStream` 衍生而來，這兩個類是建立裝飾器的必要條件（這樣它們才能為所有被裝飾的物件提供統一介面）。

### 通過 `FilterInputStream` 從 `InputStream` 讀取

`FilterInputStream` 類能夠完成兩件截然不同的事情。其中，`DataInputStream` 允許我們讀取不同的基本資料類型和 `String` 類型的物件（所有方法都以 “read” 開頭，例如 `readByte()`、`readFloat()`等等）。搭配其對應的 `DataOutputStream`，我們就可以透過資料“流”將基本資料類型的資料從一個地方遷移到另一個地方。具體是那些“地方”是由[表 I/O-1](#table-io-1) 中的那些類決定的。

其它 `FilterInputStream` 類則在內部修改 `InputStream` 的行為方式：是否緩衝，是否保留它所讀過的行（允許我們查詢行數或設定行數），以及是否允許把單個字元推回輸入流等等。最後兩個類看起來就像是為了建立編譯器提供的（它們被添加進來可能是為了對“用 Java 構建編譯器”實現提供支援），因此我們在一般編程中不會用到它們。

在實際應用中，不管連接的是什麼 I/O 裝置，我們基本上都會對輸入進行緩衝。所以當初 I/O 類庫如果能預設都讓輸入進行緩衝，同時將無緩衝輸入作為一種特殊情況（或者只是簡單地提供一個方法呼叫），這樣會更加合理，而不是像現在這樣迫使我們基本上每次都得手動添加緩衝。
<!-- 譯者註：感覺第四版中文版（536頁）把上面這一段的意思弄反了 -->

<span id="table-io-3">**表 I/O-3：`FilterInputStream` 類型**</span>

| 類 | 功能 | 構造器參數 | 如何使用 |
| :--: | :-- | :-------- | :----- |
| `DataInputStream` | 與 `DataOutputStream` 搭配使用，按照移植方式從流讀取基本資料類型（`int`、`char`、`long` 等） | `InputStream` | 包含用於讀取基本資料類型的全部介面 |
| `BufferedInputStream`      | 使用它可以防止每次讀取時都得進行實際寫操作。代表“使用緩衝區” | `InputStream`，可以指定緩衝區大小（可選） | 本質上不提供介面，只是向行程添加緩衝功能。與介面物件搭配 |
| `LineNumberInputStream`     | 跟蹤輸入流中的行號，可呼叫 `getLineNumber()` 和 `setLineNumber(int)` | `InputStream` | 僅增加了行號，因此可能要與介面物件搭配使用 |
| `PushbackInputStream`    | 具有能彈出一個位元組的緩衝區，因此可以將讀到的最後一個字元回退 | `InputStream` | 通常作為編譯器的掃描器，我們可能永遠也不會用到 |

### 通過 `FilterOutputStream` 向 `OutputStream` 寫入

與 `DataInputStream` 對應的是 `DataOutputStream`，它可以將各種基本資料類型和 `String` 類型的物件格式化輸出到“流”中，。這樣一來，任何機器上的任何 `DataInputStream` 都可以讀出它們。所有方法都以 “write” 開頭，例如 `writeByte()`、`writeFloat()` 等等。

`PrintStream` 最初的目的就是為了以可視化格式列印所有基本資料類型和 `String` 類型的物件。這和 `DataOutputStream` 不同，後者的目的是將資料元素置入“流”中，使 `DataInputStream` 能夠可移植地重構它們。

`PrintStream` 內有兩個重要方法：`print()` 和 `println()`。它們都被重載了，可以列印各種各種資料類型。`print()` 和 `println()` 之間的差異是，後者在操作完畢後會添加一個換行符。

`PrintStream` 可能會造成一些問題，因為它捕獲了所有 `IOException`（因此，我們必須使用 `checkError()` 自行測試錯誤狀態，如果出現錯誤它會返回 `true`）。另外，`PrintStream` 沒有處理好國際化問題。這些問題都在 `PrintWriter` 中得到了解決，這在後面會講到。

`BufferedOutputStream` 是一個修飾符，表明這個“流”使用了緩衝技術，因此每次向流寫入的時候，不是每次都會執行物理寫操作。我們在進行輸出操作的時候可能會經常用到它。

<span id="table-io-4">**表 I/O-4：`FilterOutputStream` 類型**</span>

| 類 | 功能 | 構造器參數 | 如何使用 |
| :--: | :-- | :-------- | :----- |
| `DataOutputStream` | 與 `DataInputStream` 搭配使用，因此可以按照移植方式向流中寫入基本資料類型（`int`、`char`、`long` 等） | `OutputStream` | 包含用於寫入基本資料類型的全部介面 |
| `PrintStream`      | 用於產生格式化輸出。其中 `DataOutputStream` 處理資料的儲存，`PrintStream` 處理顯示 | `OutputStream`，可以用 `boolean` 值指示是否每次換行時清空緩衝區（可選） | 應該是對 `OutputStream` 物件的 `final` 封裝。可能會經常用到它 |
| `BufferedOutputStream`     | 使用它以避免每次發送資料時都進行實際的寫操作。代表“使用緩衝區”。可以呼叫 `flush()` 清空緩衝區 | `OutputStream`，可以指定緩衝區大小（可選） | 本質上並不提供介面，只是向行程添加緩衝功能。與介面物件搭配 |

<!-- Readers & Writers -->

## Reader和Writer

Java 1.1 對基本的 I/O 流類庫做了重大的修改。你初次遇到 `Reader` 和 `Writer` 時，可能會以為這兩個類是用來替代 `InputStream` 和 `OutputStream` 的，但實際上並不是這樣。儘管一些原始的“流”類庫已經過時了（如果使用它們，編譯器會發出警告），但是 `InputStream` 和 `OutputStream` 在面向位元組 I/O 這方面仍然發揮著極其重要的作用，而 `Reader` 和 `Writer` 則提供相容 Unicode 和面向字元 I/O 的功能。另外：

1. Java 1.1 往 `InputStream` 和 `OutputStream` 的繼承體系中又添加了一些新類，所以這兩個類顯然是不會被取代的；

2. 有時我們必須把來自“位元組”層級結構中的類和來自“字元”層次結構中的類結合起來使用。為了達到這個目的，需要用到“適配器（adapter）類”：`InputStreamReader` 可以把 `InputStream` 轉換為 `Reader`，而 `OutputStreamWriter` 可以把 `OutputStream` 轉換為 `Writer`。

設計 `Reader` 和 `Writer` 繼承體系主要是為了國際化。老的 I/O 流繼承體系僅支援 8 位元的位元組流，並且不能很好地處理 16 位元的 Unicode 字元。由於 Unicode 用於字元國際化（Java 本身的 `char` 也是 16 位元的 Unicode），所以添加 `Reader` 和 `Writer` 繼承體系就是為了讓所有的 I/O 操作都支援 Unicode。另外，新類庫的設計使得它的操作比舊類庫要快。

### 資料的來源和去處

幾乎所有原始的 Java I/O 流類都有相應的 `Reader` 和 `Writer` 類來提供原生的 Unicode 操作。但是在某些場合，面向位元組的 `InputStream` 和 `OutputStream` 才是正確的解決方案。特別是 `java.util.zip` 類庫就是面向位元組而不是面向字元的。因此，最明智的做法是儘量**嘗試**使用 `Reader` 和 `Writer`，一旦程式碼沒辦法成功編譯，你就會發現此時應該使用面向位元組的類庫了。

下表展示了在兩個繼承體系中，訊息的來源和去處（即資料物理上來自哪裡又去向哪裡）之間的對應關係：

| 來源與去處：Java 1.0 類 | 相應的 Java 1.1 類 |
| :-------------------: | :--------------: |
| `InputStream`  |  `Reader` <br/> 適配器：`InputStreamReader` |
| `OutputStream`  |  `Writer` <br/> 適配器：`OutputStreamWriter` |
| `FileInputStream`  | `FileReader` |
| `FileOutputStream`  |  `FileWriter` |
| `StringBufferInputStream`（已棄用） | `StringReader` |
| （無相應的類） |  `StringWriter` |
| `ByteArrayInputStream`  |  `CharArrayReader` |
| `ByteArrayOutputStream`  | `CharArrayWriter` |
| `PipedInputStream`  |  `PipedReader` |
| `PipedOutputStream`  | `PipedWriter` |

總的來說，這兩個不同的繼承體系中的介面即便不能說完全相同，但也是非常相似的。

### 更改流的行為

對於 `InputStream` 和 `OutputStream` 來說，我們會使用 `FilterInputStream` 和 `FilterOutputStream` 的裝飾器子類來修改“流”以滿足特殊需要。`Reader` 和 `Writer` 的類繼承體系沿用了相同的思想——但是並不完全相同。

在下表中，左右之間對應關係的近似程度現比上一個表格更加粗略一些。造成這種差別的原因是類的組織形式不同，`BufferedOutputStream` 是 `FilterOutputStream` 的子類，但 `BufferedWriter` 卻不是 `FilterWriter` 的子類（儘管 `FilterWriter` 是抽象類，但卻沒有任何子類，把它放在表格里只是占個位置，不然你可能奇怪 `FilterWriter` 上哪去了）。然而，這些類的介面卻又十分相似。

| 過濾器：Java 1.0 類 | 相應 Java 1.1 類 |
| :---------------  | :-------------- |
| `FilterInputStream` | `FilterReader` |
| `FilterOutputStream` | `FilterWriter` (抽象類，沒有子類) |
| `BufferedInputStream` | `BufferedReader`（也有 `readLine()`) |
| `BufferedOutputStream` | `BufferedWriter` |
| `DataInputStream` | 使用 `DataInputStream`（ 如果必須用到 `readLine()`，那你就得使用 `BufferedReader`。否則，一般情況下就用 `DataInputStream` |
| `PrintStream` | `PrintWriter` |
| `LineNumberInputStream` | `LineNumberReader` |
| `StreamTokenizer` | `StreamTokenizer`（使用具有 `Reader` 參數的構造器） |
| `PushbackInputStream` | `PushbackReader` |

有一條限制需要明確：一旦要使用 `readLine()`，我們就不應該用 `DataInputStream`（否則，編譯時會得到使用了過時方法的警告），而應該使用 `BufferedReader`。除了這種情況之外的情形中，`DataInputStream` 仍是 I/O 類庫的首選成員。

為了使用時更容易過渡到 `PrintWriter`，它提供了一個既能接受 `Writer` 物件又能接受任何 `OutputStream` 物件的構造器。`PrintWriter` 的格式化介面實際上與 `PrintStream` 相同。

Java 5 添加了幾種 `PrintWriter` 構造器，以便在將輸出寫入時簡化文件的建立過程，你馬上就會見到它們。

其中一種 `PrintWriter` 構造器還有一個執行**自動 flush**[^2] 的選項。如果構造器設定了該選項，就會在每個 `println()` 呼叫之後，自動執行 flush。

### 未發生改變的類

有一些類在 Java 1.0 和 Java 1.1 之間未做改變。

| 以下這些 Java 1.0 類在 Java 1.1 中沒有相應類 |
| --- |
| `DataOutputStream` |
| `File` |
| `RandomAccessFile` |
| `SequenceInputStream` |

特別是 `DataOutputStream`，在使用時沒有任何變化；因此如果想以可傳輸的格式儲存和檢索資料，請用 `InputStream` 和 `OutputStream` 繼承體系。

<!-- Off By Itself: RandomAccessFile -->
## RandomAccessFile類

`RandomAccessFile` 適用於由大小已知的紀錄組成的文件，所以我們可以使用 `seek()` 將文件指標從一條紀錄移動到另一條紀錄，然後對記錄進行讀取和修改。文件中記錄的大小不一定都相同，只要我們能確定那些記錄有多大以及它們在文件中的位置即可。

最初，我們可能難以相信 `RandomAccessFile` 不是 `InputStream` 或者 `OutputStream` 繼承體系中的一部分。除了實現了 `DataInput` 和 `DataOutput` 介面（`DataInputStream` 和 `DataOutputStream` 也實現了這兩個介面）之外，它和這兩個繼承體系沒有任何關係。它甚至都不使用 `InputStream` 和 `OutputStream` 類中已有的任何功能。它是一個完全獨立的類，其所有的方法（大多數都是 `native` 方法）都是從頭開始編寫的。這麼做是因為 `RandomAccessFile` 擁有和別的 I/O 類型本質上不同的行為，因為我們可以在一個文件內向前和向後移動。在任何情況下，它都是自我獨立的，直接繼承自 `Object`。

從本質上來講，`RandomAccessFile` 的工作方式類似於把 `DataIunputStream` 和 `DataOutputStream` 組合起來使用。另外它還有一些額外的方法，比如使用 `getFilePointer()` 可以得到目前文件指標在文件中的位置，使用 `seek()` 可以移動文件指標，使用 `length()` 可以得到文件的長度。另外，其構造器還需要傳入第二個參數（和 C 語言中的 `fopen()` 相同）用來表示我們是準備對文件進行 “隨機讀”（r）還是“讀寫”（rw）。它並不支援只寫文件，從這點來看，如果當初 `RandomAccessFile` 能設計成繼承自 `DataInputStream`，可能也是個不錯的實現方式。

在 Java 1.4 中，`RandomAccessFile` 的大多數功能（但不是全部）都被 nio 中的**記憶體映射文件**（mmap）取代，詳見[附錄：新 I/O](./Appendix-New-IO.md)。

<!-- Typical Uses of I/O Streams -->

## IO流典型用途

儘管我們可以用不同的方式來組合 I/O 流類，但常用的也就其中幾種。你可以下面的例子可以作為 I/O 典型用法的基本參照（在你確定無法使用[文件](./17-Files.md)這一章所述的庫之後）。

在這些範例中，異常處理都被簡化為將異常傳遞給控制台，但是這樣做只適用於小型的範例和工具。在你自己的程式碼中，你需要考慮更加複雜的錯誤處理方式。

### 緩衝輸入文件

如果想要打開一個文件進行字元輸入，我們可以使用一個 `FileInputReader` 物件，然後傳入一個 `String` 或者 `File` 物件作為檔案名。為了提高速度，我們希望對那個文件進行緩衝，那麼我們可以將所產生的引用傳遞給一個 `BufferedReader` 構造器。`BufferedReader` 提供了 `line()` 方法，它會產生一個 `Stream<String>` 物件：

```java
// iostreams/BufferedInputFile.java
// {VisuallyInspectOutput}
import java.io.*;
import java.util.stream.*;

public class BufferedInputFile {
    public static String read(String filename) {
        try (BufferedReader in = new BufferedReader(
                new FileReader(filename))) {
            return in.lines()
                    .collect(Collectors.joining("\n"));
        } catch (IOException e) {
            throw new RuntimeException(e);
        }
    }

    public static void main(String[] args) {
        System.out.print(
                read("BufferedInputFile.java"));
    }
}
```

`Collectors.joining()` 在其內部使用了一個 `StringBuilder` 來累加其執行結果。該文件會透過 `try-with-resources` 子句自動關閉。

### 從記憶體輸入

下面範例中，從 `BufferedInputFile.read()` 讀入的 `String` 被用來建立一個 `StringReader` 物件。然後呼叫其 `read()` 方法，每次讀取一個字元，並把它顯示在控制台上：

```java
// iostreams/MemoryInput.java
// {VisuallyInspectOutput}
import java.io.*;

public class MemoryInput {
    public static void
    main(String[] args) throws IOException {
        StringReader in = new StringReader(
                BufferedInputFile.read("MemoryInput.java"));
        int c;
        while ((c = in.read()) != -1)
            System.out.print((char) c);
    }
}
```

注意 `read()` 是以 `int` 形式返回下一個位元組，所以必須類型轉換為 `char` 才能正確列印。

### 格式化記憶體輸入

要讀取格式化資料，我們可以使用 `DataInputStream`，它是一個面向位元組的 I/O 類（不是面向字元的）。這樣我們就必須使用 `InputStream` 類而不是 `Reader` 類。我們可以使用 `InputStream` 以位元組形式讀取任何資料（比如一個文件），但這裡使用的是字串。

```java
// iostreams/FormattedMemoryInput.java
// {VisuallyInspectOutput}
import java.io.*;

public class FormattedMemoryInput {
    public static void main(String[] args) {
        try (
            DataInputStream in = new DataInputStream(
                new ByteArrayInputStream(
                    BufferedInputFile.read(
                        "FormattedMemoryInput.java")
                            .getBytes()))
        ) {
            while (true)
                System.out.write((char) in.readByte());
        } catch (EOFException e) {
            System.out.println("\nEnd of stream");
        } catch (IOException e) {
            throw new RuntimeException(e);
        }
    }
}
```

`ByteArrayInputStream` 必須接收一個位元組陣列，所以這裡我們呼叫了 `String.getBytes()` 方法。所產生的的 `ByteArrayInputStream` 是一個適合傳遞給 `DataInputStream` 的 `InputStream`。

如果我們用 `readByte()` 從 `DataInputStream` 一次一個位元組地讀取字元，那麼任何位元組的值都是合法結果，因此返回值不能用來檢測輸入是否結束。取而代之的是，我們可以使用 `available()` 方法得到剩餘可用字元的數量。下面例子示範了怎麼一次一個位元組地讀取文件：

```java
// iostreams/TestEOF.java
// Testing for end of file
// {VisuallyInspectOutput}
import java.io.*;

public class TestEOF {
    public static void main(String[] args) {
        try (
            DataInputStream in = new DataInputStream(
                new BufferedInputStream(
                    new FileInputStream("TestEOF.java")))
        ) {
            while (in.available() != 0)
                System.out.write(in.readByte());
        } catch (IOException e) {
            throw new RuntimeException(e);
        }
    }
}
```

注意，`available()` 的工作方式會隨著所讀取媒介類型的不同而有所差異，它的字面意思就是“在沒有阻塞的情況下所能讀取的位元組數”。對於文件，能夠讀取的是整個文件；但是對於其它類型的“流”，可能就不是這樣，所以要謹慎使用。

我們也可以透過捕獲異常來檢測輸入的末尾。但是，用異常作為控制流是對異常的一種錯誤使用方式。

### 基本文件的輸出

`FileWriter` 物件用於向文件寫入資料。實際使用時，我們通常會用 `BufferedWriter` 將其包裝起來以增加緩衝的功能（可以試試移除此包裝來感受一下它對性能的影響——緩衝往往能顯著地增加 I/O 操作的效能）。在本例中，為了提供格式化功能，它又被裝飾成了 `PrintWriter`。按照這種方式建立的資料文件可作為普通文字文件來讀取。

```java
// iostreams/BasicFileOutput.java
// {VisuallyInspectOutput}
import java.io.*;

public class BasicFileOutput {
    static String file = "BasicFileOutput.dat";

    public static void main(String[] args) {
        try (
            BufferedReader in = new BufferedReader(
                new StringReader(
                     BufferedInputFile.read(
                         "BasicFileOutput.java")));
                PrintWriter out = new PrintWriter(
                    new BufferedWriter(new FileWriter(file)))
        ) {
            in.lines().forEach(out::println);
        } catch (IOException e) {
            throw new RuntimeException(e);
        }
        // Show the stored file:
        System.out.println(BufferedInputFile.read(file));
    }
}
```

`try-with-resources` 語句會自動 flush 並關閉文件。

### 文字文件輸出捷徑

Java 5 在 `PrintWriter` 中添加了一個輔助構造器，有了它，你在建立並寫入檔案時，就不必每次都手動執行一些裝飾的工作。下面的程式碼使用這種捷徑重寫了 `BasicFileOutput.java`：

```java
// iostreams/FileOutputShortcut.java
// {VisuallyInspectOutput}
import java.io.*;

public class FileOutputShortcut {
    static String file = "FileOutputShortcut.dat";

    public static void main(String[] args) {
        try (
            BufferedReader in = new BufferedReader(
                new StringReader(BufferedInputFile.read(
                    "FileOutputShortcut.java")));
            // Here's the shortcut:
            PrintWriter out = new PrintWriter(file)
        ) {
            in.lines().forEach(out::println);
        } catch (IOException e) {
            throw new RuntimeException(e);
        }
        System.out.println(BufferedInputFile.read(file));
    }
}
```

使用這種方式仍具備了緩衝的功能，只是現在不必自己手動添加緩衝了。但遺憾的是，其它常見的寫入任務都沒有捷徑，因此典型的 I/O 流依舊涉及大量冗餘的程式碼。本書[文件](./17-Files.md)一章中介紹的另一種方式，對此類任務進行了極大的簡化。

### 儲存和復原資料

`PrintWriter` 是用來對可讀的資料進行格式化。但如果要輸出可供另一個“流”復原的資料，我們可以用 `DataOutputStream` 寫入資料，然後用 `DataInputStream` 復原資料。當然，這些流可能是任何形式，在下面的範例中使用的是一個文件，並且對讀寫都進行了緩衝。注意 `DataOutputStream` 和 `DataInputStream` 是面向位元組的，因此要使用 `InputStream` 和 `OutputStream` 體系的類。

```java
// iostreams/StoringAndRecoveringData.java
import java.io.*;

public class StoringAndRecoveringData {
    public static void main(String[] args) {
        try (
            DataOutputStream out = new DataOutputStream(
                new BufferedOutputStream(
                    new FileOutputStream("Data.txt")))
        ) {
            out.writeDouble(3.14159);
            out.writeUTF("That was pi");
            out.writeDouble(1.41413);
            out.writeUTF("Square root of 2");
        } catch (IOException e) {
            throw new RuntimeException(e);
        }
        try (
            DataInputStream in = new DataInputStream(
                new BufferedInputStream(
                    new FileInputStream("Data.txt")))
        ) {
            System.out.println(in.readDouble());
            // Only readUTF() will recover the
            // Java-UTF String properly:
            System.out.println(in.readUTF());
            System.out.println(in.readDouble());
            System.out.println(in.readUTF());
        } catch (IOException e) {
            throw new RuntimeException(e);
        }
    }
}
```

輸出結果：

```
3.14159
That was pi
1.41413
Square root of 2
```

如果我們使用 `DataOutputStream` 進行資料寫入，那麼 Java 就保證了即便讀和寫資料的平台多麼不同，我們仍可以使用 `DataInputStream` 準確地讀取資料。這一點很有價值，眾所周知，人們曾把大量精力耗費在資料的平台相關性問題上。但現在，只要兩個平台上都有 Java，就不會存在這樣的問題[^3]。

當我們使用 `DastaOutputStream` 時，寫字串並且讓 `DataInputStream` 能夠復原它的唯一可靠方式就是使用 UTF-8 編碼，在這個範例中是用 `writeUTF()` 和 `readUTF()` 來實現的。UTF-8 是一種多位元組格式，其編碼長度根據實際使用的字元集會有所變化。如果我們使用的只是 ASCII 或者幾乎都是 ASCII 字元（只占 7 位元），那麼就顯得及其浪費空間和頻寬，所以 UTF-8 將 ASCII 字元編碼成一個位元組的形式，而非 ASCII 字元則編碼成兩到三個位元組的形式。另外，字串的長度儲存在 UTF-8 字串的前兩個位元組中。但是，`writeUTF()` 和 `readUTF()` 使用的是一種適用於 Java 的 UTF-8 變體（JDK 文件中有這些方法的詳盡描述），因此如果我們用一個非 Java 程式讀取用 `writeUTF()` 所寫的字串時，必須編寫一些特殊的程式碼才能正確讀取。

有了 `writeUTF()` 和 `readUTF()`，我們就可以在 `DataOutputStream` 中把字串和其它資料類型混合使用。因為字串完全可以作為 Unicode 格式儲存，並且可以很容易地使用 `DataInputStream` 來復原它。

`writeDouble()` 將 `double` 類型的數字儲存在流中，並用相應的 `readDouble()` 復原它（對於其它的書類型，也有類似的方法用於讀寫）。但是為了保證所有的讀方法都能夠正常工作，我們必須知道流中資料項所在的確切位置，因為極有可能將儲存的 `double` 資料作為一個簡單的位元組序列、`char` 或其它類型讀入。因此，我們必須：要嘛為文件中的資料採用固定的格式；要嘛將額外的訊息儲存到文件中，透過解析額外訊息來確定資料的存放位置。注意，物件序列化和 XML （二者都在[附錄：物件序列化](Appendix-Object-Serialization.md)中介紹）是儲存和讀取複雜資料結構的更簡單的方式。

### 讀寫隨機訪問文件

使用 `RandomAccessFile` 就像是使用了一個 `DataInputStream` 和 `DataOutputStream` 的結合體（因為它實現了相同的介面：`DataInput` 和 `DataOutput`）。另外，我們還可以使用 `seek()` 方法移動文件指標並修改對應位置的值。

在使用 `RandomAccessFile` 時，你必須清楚文件的結構，否則沒辦法正確使用它。`RandomAccessFile` 有一套專門的方法來讀寫基本資料類型的資料和 UTF-8 編碼的字串：

```java
// iostreams/UsingRandomAccessFile.java
import java.io.*;

public class UsingRandomAccessFile {
    static String file = "rtest.dat";

    public static void display() {
        try (
            RandomAccessFile rf =
                new RandomAccessFile(file, "r")
        ) {
            for (int i = 0; i < 7; i++)
                System.out.println(
                    "Value " + i + ": " + rf.readDouble());
            System.out.println(rf.readUTF());
        } catch (IOException e) {
            throw new RuntimeException(e);
        }
    }

    public static void main(String[] args) {
        try (
            RandomAccessFile rf =
                new RandomAccessFile(file, "rw")
        ) {
            for (int i = 0; i < 7; i++)
                rf.writeDouble(i * 1.414);
            rf.writeUTF("The end of the file");
            rf.close();
            display();
        } catch (IOException e) {
            throw new RuntimeException(e);
        }
        try (
            RandomAccessFile rf =
                new RandomAccessFile(file, "rw")
        ) {
            rf.seek(5 * 8);
            rf.writeDouble(47.0001);
            rf.close();
            display();
        } catch (IOException e) {
            throw new RuntimeException(e);
        }
    }
}
```

輸出結果：

```
Value 0: 0.0
Value 1: 1.414
Value 2: 2.828
Value 3: 4.242
Value 4: 5.656
Value 5: 7.069999999999999
Value 6: 8.484
The end of the file
Value 0: 0.0
Value 1: 1.414
Value 2: 2.828
Value 3: 4.242
Value 4: 5.656
Value 5: 47.0001
Value 6: 8.484
The end of the file
```

`display()` 方法打開了一個文件，並以 `double` 值的形式顯示了其中的七個元素。在 `main()` 中，首先建立了文件，然後打開並修改了它。因為 `double` 總是 8 位元組長，所以如果要用 `seek()` 定位到第 5 個（從 0 開始計數） `double` 值，則要傳入的地址值應該為 `5*8`。

正如前面所訴，雖然 `RandomAccess` 實現了 `DataInput` 和 `DataOutput` 介面，但實際上它和 I/O 繼承體系中的其它部分是分離的。它不支援裝飾，故而不能將其與 `InputStream` 及 `OutputStream` 子類中的任何一個組合起來，所以我們也沒辦法給它添加緩衝的功能。

該類的構造器還有第二個必選參數：我們可以指定讓 `RandomAccessFile` 以“唯讀”（r）方式或“讀寫”
（rw）方式打開文件。

除此之外，還可以使用 `nio` 中的“記憶體映射文件”代替 `RandomAccessFile`，這在[附錄：新 I/O](Appendix-New-IO.md)中有介紹。

<!-- Summary -->
## 本章小結

Java 的 I/O 流類庫的確能夠滿足我們的基本需求：我們可以透過控制台、文件、記憶體塊，甚至網路進行讀寫。透過繼承，我們可以建立新類型的輸入和輸出物件。並且我們甚至可以透過重新定義“流”所接受物件類型的 `toString()` 方法，進行簡單的擴展。當我們向一個期望收到字串的方法傳送一個非字串物件時，會自動呼叫物件的 `toString()` 方法（這是 Java 中有限的“自動類型轉換”功能之一）。

在 I/O 流類庫的文件和設計中，仍留有一些沒有解決的問題。例如，我們打開一個文件用於輸出，如果在我們試圖覆蓋這個文件時能拋出一個異常，這樣會比較好（有的程式系統只有當該文件不存在時，才允許你將其作為輸出文件打開）。在 Java 中，我們應該使用一個 `File` 物件來判斷文件是否存在，因為如果我們用 `FileOutputStream` 或者 `FileWriter` 打開，那麼這個文件肯定會被覆蓋。

I/O 流類庫讓我們喜憂參半。它確實挺有用的，而且還具有可移植性。但是如果我們沒有理解“裝飾器”模式，那麼這種設計就會顯得不是很直觀。所以，它的學習成本相對較高。而且它並不完善，比如說在過去，我不得不編寫相當數量的程式碼去實現一個讀取文字文件的工具——所幸的是，Java 7 中的 nio 消除了此類需求。

一旦你理解了裝飾器模式，並且開始在某些需要這種靈活性的場景中使用該類庫，那麼你就開始能從這種設計中受益了。到那時候，為此額外多寫幾行程式碼的開銷應該不至於讓人覺得太麻煩。但還是請務必檢查一下，確保使用[文件](./17-Files.md)一章中的庫和技術沒辦法解決問題後，再考慮使用本章的 I/O 流庫。

[^1]: 很難說這就是一個很好的設計選擇，尤其是與其它程式語言中簡單的 I/O 類庫相比較。但它確實是如此選擇的一個正當理由。

[^2]: 譯者註：“flush” 直譯是“清空”，意思是把緩衝中的資料清空，輸送到對應的目的地（如文件和螢幕）。

[^3]: XML 是另一種方式，可以解決在不同計算平台之間移動資料，而不依賴於所有平台上都有 Java 這一問題。XML 將在[附錄：物件序列化](./Appendix-Object-Serialization.md)一章中進行介紹。

<!-- 分頁 -->

<div style="page-break-after: always;"></div>
