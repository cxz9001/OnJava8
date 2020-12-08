[TOC]

<!-- Appendix: Data Compression -->
# 附錄:資料壓縮

Java I/O 類庫提供了可以讀寫壓縮格式流的類。你可以將其他 I/O 類包裝起來用於提供壓縮功能。

這些類不是從 **Reader** 和 **Writer** 類衍生的，而是 **InputStream** 和 **OutputStream** 層級結構的一部分。這是由於壓縮庫處理的是位元組，而不是字元。但是，你可能會被迫混合使用兩種類型的流（請記住，你可以使用 **InputStreamReader** 和 **OutputStreamWriter**，這兩個類可以在位元組類型和字元類型之間輕鬆轉換）。

| 壓縮類                   | 功能                                                         |
| ------------------------ | ------------------------------------------------------------ |
| **CheckedInputStream**   | `getCheckSum()` 可以對任意 **InputStream** 計算校驗和（而不只是解壓） |
| **CheckedOutputStream**  | `getCheckSum()` 可以對任意 **OutputStream** 計算校驗和（而不只是壓縮） |
| **DeflaterOutputStream** | 壓縮類的基類                                                 |
| **ZipOutputStream**      | **DeflaterOutputStream** 類的一種，用於壓縮資料到 Zip 文件結構 |
| **GZIPOutputStream**     | **DeflaterOutputStream** 類的一種，用於壓縮資料到 GZIP 文件結構 |
| **InflaterInputStream**  | 解壓類的基類                                                 |
| **ZipInputStream**       | **InflaterInputStream** 類的一種，用於解壓 Zip 文件結構的資料 |
| **GZIPInputStream**      | **InflaterInputStream** 類的一種，用於解壓 GZIP 文件結構的資料 |

儘管存在很多壓縮演算法，但是 Zip 和 GZIP 可能是最常見的。你可以使用許多用於讀取和寫入這些格式的工具，來輕鬆操作壓縮資料。

<!-- Simple Compression with GZIP -->

## 使用 Gzip 簡單壓縮

<!-- Multifile Storage with Zip -->

GZIP 介面十分簡單，因此當你有一個需要壓縮的資料流（而不是一個包含不同資料分片的容器）時，使用 GZIP 更為合適。如下是一個壓縮單個文件的範例：

```java
// compression/GZIPcompress.java
// (c)2017 MindView LLC: see Copyright.txt
// We make no guarantees that this code is fit for any purpose.
// Visit http://OnJava8.com for more book information.
// {java GZIPcompress GZIPcompress.java}
// {VisuallyInspectOutput}

public class GZIPcompress {
    public static void main(String[] args) {
        if (args.length == 0) {
            System.out.println(
                    "Usage: \nGZIPcompress file\n" +
                            "\tUses GZIP compression to compress " +
                            "the file to test.gz");
            System.exit(1);
        }
        try (
                InputStream in = new BufferedInputStream(
                        new FileInputStream(args[0]));
                BufferedOutputStream out =
                        new BufferedOutputStream(
                                new GZIPOutputStream(
                                        new FileOutputStream("test.gz")))
        ) {
            System.out.println("Writing file");
            int c;
            while ((c = in.read()) != -1)
                out.write(c);
        } catch (IOException e) {
            throw new RuntimeException(e);
        }
        System.out.println("Reading file");
        try (
                BufferedReader in2 = new BufferedReader(
                        new InputStreamReader(new GZIPInputStream(
                                new FileInputStream("test.gz"))))
        ) {
            in2.lines().forEach(System.out::println);
        } catch (IOException e) {
            throw new RuntimeException(e);
        }
    }
}

```

使用壓縮類非常簡單，你只需要把你的輸出流包裝在 **GZIPOutputStream** 或 **ZipOutputStream** 中，將輸入流包裝在 **GZIPInputStream** 或 **ZipInputStream**。其他的一切就只是普通的 I/O 讀寫。這是面向字元流和面向位元組流的混合範例；in 使用 Reader 類，而 **GZIPOutputStreams** 建構子只能接受 **OutputStream** 物件，而不能接受 **Writer** 物件。當打開文件的時候，**GZIPInputStream** 會轉換成為 **Reader**。

## 使用 zip 多文件儲存

支援 Zip 格式的庫比 GZIP 庫更廣泛。有了它，你可以輕鬆儲存多個文件，甚至還有一個單獨的類可以輕鬆地讀取 Zip 文件。該庫使用標準 Zip 格式，因此它可以與目前可在 Internet 上下載的所有 Zip 工具無縫協作。以下範例與前一個範例具有相同的形式，但它可以根據需要處理任意數量的命令列參數。此外，它還顯示了 **Checksum** 類計算和驗證文件的校驗和。有兩種校驗和類型：Adler32（更快）和 CRC32（更慢但更準確）。

```java
// compression/ZipCompress.java
// (c)2017 MindView LLC: see Copyright.txt
// We make no guarantees that this code is fit for any purpose.
// Visit http://OnJava8.com for more book information.
// Uses Zip compression to compress any
// number of files given on the command line
// {java ZipCompress ZipCompress.java}
// {VisuallyInspectOutput}
public class ZipCompress {
    public static void main(String[] args) {
        try (
                FileOutputStream f =
                        new FileOutputStream("test.zip");
                CheckedOutputStream csum =
                        new CheckedOutputStream(f, new Adler32());
                ZipOutputStream zos = new ZipOutputStream(csum);
                BufferedOutputStream out =
                        new BufferedOutputStream(zos)
        ) {
            zos.setComment("A test of Java Zipping");
            // No corresponding getComment(), though.
            for (String arg : args) {
                System.out.println("Writing file " + arg);
                try (
                        InputStream in = new BufferedInputStream(
                                new FileInputStream(arg))
                ) {
                    zos.putNextEntry(new ZipEntry(arg));
                    int c;
                    while ((c = in.read()) != -1)
                        out.write(c);
                }
                out.flush();
            }
            // Checksum valid only after the file is closed!
            System.out.println(
                    "Checksum: " + csum.getChecksum().getValue());
        } catch (IOException e) {
            throw new RuntimeException(e);
        }
        // Now extract the files:
        System.out.println("Reading file");
        try (
                FileInputStream fi =
                        new FileInputStream("test.zip");
                CheckedInputStream csumi =
                        new CheckedInputStream(fi, new Adler32());
                ZipInputStream in2 = new ZipInputStream(csumi);
                BufferedInputStream bis =
                        new BufferedInputStream(in2)
        ) {
            ZipEntry ze;
            while ((ze = in2.getNextEntry()) != null) {
                System.out.println("Reading file " + ze);
                int x;
                while ((x = bis.read()) != -1)
                    System.out.write(x);
            }
            if (args.length == 1)
                System.out.println(
                        "Checksum: " + csumi.getChecksum().getValue());
        } catch (IOException e) {
            throw new RuntimeException(e);
        }
        // Alternative way to open and read Zip files:
        try (
                ZipFile zf = new ZipFile("test.zip")
        ) {
            Enumeration e = zf.entries();
            while (e.hasMoreElements()) {
                ZipEntry ze2 = (ZipEntry) e.nextElement();
                System.out.println("File: " + ze2);
                // ... and extract the data as before
            }
        } catch (IOException e) {
            throw new RuntimeException(e);
        }
    }
}
```

對於要添加到存檔的每個文件，必須呼叫 `putNextEntry()` 並傳遞 **ZipEntry** 物件。 **ZipEntry** 物件包含一個擴展介面，用於獲取和設定 Zip 文件中該特定條目的所有可用資料：名稱，壓縮和未壓縮大小，日期，CRC 校驗和，額外欄位資料，注釋，壓縮方法以及它是否是目錄條目。但是，即使 Zip 格式有設定密碼的方法，Java 的 Zip 庫也不支援。雖然 **CheckedInputStream** 和 **CheckedOutputStream** 都支援 Adler32 和 CRC32 校驗和，但 **ZipEntry** 類僅支援 CRC 介面。這是對基礎 Zip 格式的限制，但它可能會限制你使用更快的 Adler32。

要提取文件，**ZipInputStream** 有一個 `getNextEntry()` 方法，這個方法在有文件存在的情況下呼叫，會返回下一個 **ZipEntry**。作為一個更簡潔的替代方法，你可以使用 **ZipFile** 物件讀取該文件，該物件具有方法 entries() 返回一個包裹 **ZipEntries** 的 **Enumeration**。

要讀取校驗和，你必須以某種方式訪問關聯的 **Checksum** 物件。這裡保留了對 **CheckedOutputStream** 和 **CheckedInputStream** 物件的引用，但你也可以保持對 **Checksum** 物件的引用。 Zip 流中的一個令人困惑的方法是 `setComment()`。如 **ZipCompress** 所示。在 Java 中，你可以在編寫文件時設定注釋，但是沒有辦法復原 **ZipInputStream** 中的注釋。注釋似乎僅透過 **ZipEntry** 在逐個條目的基礎上完全支援。

使用 GZIP 或 Zip 庫時，你不僅被限制於文件——你可以壓縮任何內容，包括透過網路連接發送的資料。



<!-- Java Archives (Jars) -->

## Java 的 jar

Zip 格式也用於 JAR（Java ARchive）檔案格式，這是一種將一組文件收集到單個壓縮文件中的方法，就像 Zip 一樣。但是，與 Java 中的其他所有內容一樣，JAR 文件是跨平台的，因此你不必擔心平台問題。你還可以將音訊和圖像文件像類文件一樣包含在其中。

JAR 文件由一個包含壓縮文件集合的文件和一個描述它們的“清單（manifest）”組成。（你可以建立自己的清單文件；否則，jar 程式將為你執行此操作。）你可以在 JDK 文件中，找到更多關於 JAR 清單的訊息。

JDK 附帶的 jar 工具會自動壓縮你選擇的文件。你可以在命令列上呼叫它：

```shell
jar [options] destination [manifest] inputfile(s)
```

選項是一組字母（不需要連字號或任何其他指示符）。 Unix / Linux 使用者會注意到這些選項與 tar 指令選項的相似性。這些是：

| 選項       | 功能                                                         |
| ---------- | ------------------------------------------------------------ |
| **c**      | 建立一個新的或者空的歸檔文件                                 |
| **t**      | 列出內容目錄                                                 |
| **x**      | 提取所有文件                                                 |
| **x** file | 提取指定的文件                                               |
| **f**      | 這代表著，“傳遞文件的名稱。”如果你不使用它，jar 假定它的輸入將來自標準輸入，或者，如果它正在建立一個文件，它的輸出將轉到標準輸出。 |
| **m**      | 代表第一個參數是使用者建立的清單文件的名稱。                   |
| **v**      | 生成詳細的輸出用於表述 jar 所作的事情                        |
| **0**      | 僅儲存文件;不壓縮文件（用於建立放在類路徑中的 JAR 文件）。   |
| **M**      | 不要自動建立清單文件                                         |

如果放入 JAR 文件的文件中包含子目錄，則會自動添加該子目錄，包括其所有子目錄等。還會保留路徑訊息。

以下是一些呼叫 jar 的典型方法。以下指令建立名為 myJarFile 的 JAR 文件。 jar 包含目前目錄中的所有類文件，以及自動生成的清單文件：

```shell
jar cf myJarFile.jar *.class
```

下一個指令與前面的範例類似，但它添加了一個名為 myManifestFile.mf 的使用者建立的清單文件。 ：

```shell
jar cmf myJarFile.jar myManifestFile.mf *.class
```

這個指令輸出了 myJarFile.jar 中的文件目錄：

```shell
jar tf myJarFile.jar
```

如下添加了一個“verbose”的標誌，用於生成更多關於 myJarFile.jar 中文件的詳細訊息：

```shell
jar tvf myJarFile.jar
```

假設 audio，classes 和 image 都是子目錄，它將所有子目錄組合到文件 myApp.jar 中。還包括“verbose”標誌，以便在 jar 程式工作時提供額外的回饋：

```shell
jar cvf myApp.jar audio classes image
```

如果你在建立 JAR 文件時使用了 0（零） 選項，該文件將會被取代在你的類路徑（CLASSPATH）中：

```shell
CLASSPATH="lib1.jar;lib2.jar;"
```

然後 Java 可以搜尋到 lib1.jar 和 lib2.jar 的類文件。

jar 工具不像 Zip 實用程式那樣通用。例如，你無法將文件添加或更新到現有 JAR 文件；只能從頭開始建立 JAR 文件。

此外，你無法將文件移動到 JAR 文件中，在移動文件時將其刪除。

但是，在一個平台上建立的 JAR 文件可以透過任何其他平台上的 jar 工具透明地讀取（這個問題有時會困擾 Zip 實用程式）。

<!-- 分頁 -->
<div style="page-break-after: always;"></div>
