[TOC]

<!-- File -->

# 第十七章 文件
>在醜陋的 Java I/O 編程方式誕生多年以後，Java終於簡化了文件讀寫的基本操作。

這種"困難方式"的全部細節都在 [Appendix: I/O Streams](./Appendix-IO-Streams.md)。如果你讀過這個部分，就會認同 Java 設計者毫不在意他們的使用者的體驗這一觀念。打開並讀取文件對於大多數程式語言來說是非常常用的，由於 I/O 糟糕的設計以至於
很少有人能夠在不依賴其他參考程式碼的情況下完成打開文件的操作。

好像 Java 設計者終於意識到了 Java 使用者多年來的痛苦，在 Java7 中對此引入了巨大的改進。這些新元素被放在 **java.nio.file** 包下面，過去人們通常把 **nio** 中的 **n** 理解為 **new** 即新的 **io**，現在更應該當成是 **non-blocking** 非阻塞 **io**(**io**就是*input/output輸入/輸出*)。**java.nio.file** 庫終於將 Java 文件操作帶到與其他程式語言相同的水準。最重要的是 Java8 新增的 streams 與文件結合使得文件操作編程變得更加優雅。我們將看一下文件操作的兩個基本元件：

1. 文件或者目錄的路徑；
2. 文件本身。

<!-- File and Directory Paths -->
## 文件和目錄路徑

一個 **Path** 物件表示一個文件或者目錄的路徑，是一個跨作業系統（OS）和文件系統的抽象，目的是在構造路徑時不必關注底層作業系統，程式碼可以在不進行修改的情況下執行在不同的作業系統上。**java.nio.file.Paths** 類包含一個重載方法 **static get()**，該方法接受一系列 **String** 字串或一個*統一資源標識符*(URI)作為參數，並且進行轉換返回一個 **Path** 物件：

```java
// files/PathInfo.java
import java.nio.file.*;
import java.net.URI;
import java.io.File;
import java.io.IOException;

public class PathInfo {
    static void show(String id, Object p) {
        System.out.println(id + ": " + p);
    }

    static void info(Path p) {
        show("toString", p);
        show("Exists", Files.exists(p));
        show("RegularFile", Files.isRegularFile(p));
        show("Directory", Files.isDirectory(p));
        show("Absolute", p.isAbsolute());
        show("FileName", p.getFileName());
        show("Parent", p.getParent());
        show("Root", p.getRoot());
        System.out.println("******************");
    }
    public static void main(String[] args) {
        System.out.println(System.getProperty("os.name"));
        info(Paths.get("C:", "path", "to", "nowhere", "NoFile.txt"));
        Path p = Paths.get("PathInfo.java");
        info(p);
        Path ap = p.toAbsolutePath();
        info(ap);
        info(ap.getParent());
        try {
            info(p.toRealPath());
        } catch(IOException e) {
           System.out.println(e);
        }
        URI u = p.toUri();
        System.out.println("URI: " + u);
        Path puri = Paths.get(u);
        System.out.println(Files.exists(puri));
        File f = ap.toFile(); // Don't be fooled
    }
}

/* 輸出:
Windows 10
toString: C:\path\to\nowhere\NoFile.txt
Exists: false
RegularFile: false
Directory: false
Absolute: true
FileName: NoFile.txt
Parent: C:\path\to\nowhere
Root: C:\
******************
toString: PathInfo.java
Exists: true
RegularFile: true
Directory: false
Absolute: false
FileName: PathInfo.java
Parent: null
Root: null
******************
toString: C:\Users\Bruce\Documents\GitHub\onjava\
ExtractedExamples\files\PathInfo.java
Exists: true
RegularFile: true
Directory: false
Absolute: true
FileName: PathInfo.java
Parent: C:\Users\Bruce\Documents\GitHub\onjava\
ExtractedExamples\files
Root: C:\
******************
toString: C:\Users\Bruce\Documents\GitHub\onjava\
ExtractedExamples\files
Exists: true
RegularFile: false
Directory: true
Absolute: true
FileName: files
Parent: C:\Users\Bruce\Documents\GitHub\onjava\
ExtractedExamples
Root: C:\
******************
toString: C:\Users\Bruce\Documents\GitHub\onjava\
ExtractedExamples\files\PathInfo.java
Exists: true
RegularFile: true
Directory: false
Absolute: true
FileName: PathInfo.java
Parent: C:\Users\Bruce\Documents\GitHub\onjava\
ExtractedExamples\files
Root: C:\
******************
URI: file:///C:/Users/Bruce/Documents/GitHub/onjava/
ExtractedExamples/files/PathInfo.java
true
*/
```

我已經在這一章第一個程式的 **main()** 方法添加了第一行用於展示作業系統的名稱，因此你可以看到不同作業系統之間存在哪些差異。理想情況下，差別會相對較小，並且使用 **/** 或者 **\\** 路徑分隔符進行分隔。你可以看到我執行在Windows 10 上的程式輸出。

當 **toString()** 方法生成完整形式的路徑，你可以看到 **getFileName()** 方法總是返回目前檔案名。
透過使用 **Files** 工具類(我們接下來將會更多地使用它)，可以測試一個文件是否存在，測試是否是一個"普通"文件還是一個目錄等等。"Nofile.txt"這個範例展示我們描述的文件可能並不在指定的位置；這樣可以允許你建立一個新的路徑。"PathInfo.java"存在於目前目錄中，最初它只是沒有路徑的檔案名，但它仍然被檢測為"存在"。一旦我們將其轉換為絕對路徑，我們將會得到一個從"C:"盤(因為我們是在Windows機器下進行測試)開始的完整路徑，現在它也擁有一個父路徑。“真實”路徑的定義在文件中有點模糊，因為它取決於具體的文件系統。例如，如果檔案名不區分大小寫，即使路徑由於大小寫的緣故而不是完全相同，也可能得到肯定的匹配結果。在這樣的平台上，**toRealPath()** 將返回實際情況下的 **Path**，並且還會刪除任何冗餘元素。

這裡你會看到 **URI** 看起來只能用於描述文件，實際上 **URI** 可以用於描述更多的東西；通過 [維基百科](https://en.wikipedia.org/wiki/Uniform_Resource_Identifier) 可以了解更多細節。現在我們成功地將 **URI** 轉為一個 **Path** 物件。

最後，你會在 **Path** 中看到一些有點欺騙的東西，這就是呼叫 **toFile()** 方法會生成一個 **File** 物件。聽起來似乎可以得到一個類似文件的東西(畢竟被稱為 **File** )，但是這個方法的存在僅僅是為了向後相容。雖然看起來應該被稱為"路徑"，實際上卻應該表示目錄或者文件本身。這是個非常草率並且令人困惑的命名，但是由於 **java.nio.file** 的存在我們可以安全地忽略它的存在。

### 選取路徑部分片段
**Path** 物件可以非常容易地生成路徑的某一部分：

```java
// files/PartsOfPaths.java
import java.nio.file.*;

public class PartsOfPaths {
    public static void main(String[] args) {
        System.out.println(System.getProperty("os.name"));
        Path p = Paths.get("PartsOfPaths.java").toAbsolutePath();
        for(int i = 0; i < p.getNameCount(); i++)
            System.out.println(p.getName(i));
        System.out.println("ends with '.java': " +
        p.endsWith(".java"));
        for(Path pp : p) {
            System.out.print(pp + ": ");
            System.out.print(p.startsWith(pp) + " : ");
            System.out.println(p.endsWith(pp));
        }
        System.out.println("Starts with " + p.getRoot() + " " + p.startsWith(p.getRoot()));
    }
}

/* 輸出:
Windows 10
Users
Bruce
Documents
GitHub
on-java
ExtractedExamples
files
PartsOfPaths.java
ends with '.java': false
Users: false : false
Bruce: false : false
Documents: false : false
GitHub: false : false
on-java: false : false
ExtractedExamples: false : false
files: false : false
PartsOfPaths.java: false : true
Starts with C:\ true
*/

```
可以透過 **getName()** 來索引 **Path** 的各個部分，直到達到上限 **getNameCount()**。**Path** 也實現了 **Iterable** 介面，因此我們也可以透過增強的 for-each 進行遍歷。請注意，即使路徑以 **.java** 結尾，使用 **endsWith()** 方法也會返回 **false**。這是因為使用 **endsWith()** 比較的是整個路徑部分，而不會包含文件路徑的後綴。透過使用 **startsWith()** 和 **endsWith()** 也可以完成路徑的遍歷。但是我們可以看到，遍歷 **Path** 物件並不包含根路徑，只有使用 **startsWith()** 檢測根路徑時才會返回 **true**。

### 路徑分析
**Files** 工具類包含一系列完整的方法用於獲得 **Path** 相關的訊息。

```java
// files/PathAnalysis.java
import java.nio.file.*;
import java.io.IOException;

public class PathAnalysis {
    static void say(String id, Object result) {
        System.out.print(id + ": ");
        System.out.println(result);
    }
    
    public static void main(String[] args) throws IOException {
        System.out.println(System.getProperty("os.name"));
        Path p = Paths.get("PathAnalysis.java").toAbsolutePath();
        say("Exists", Files.exists(p));
        say("Directory", Files.isDirectory(p));
        say("Executable", Files.isExecutable(p));
        say("Readable", Files.isReadable(p));
        say("RegularFile", Files.isRegularFile(p));
        say("Writable", Files.isWritable(p));
        say("notExists", Files.notExists(p));
        say("Hidden", Files.isHidden(p));
        say("size", Files.size(p));
        say("FileStore", Files.getFileStore(p));
        say("LastModified: ", Files.getLastModifiedTime(p));
        say("Owner", Files.getOwner(p));
        say("ContentType", Files.probeContentType(p));
        say("SymbolicLink", Files.isSymbolicLink(p));
        if(Files.isSymbolicLink(p))
            say("SymbolicLink", Files.readSymbolicLink(p));
        if(FileSystems.getDefault().supportedFileAttributeViews().contains("posix"))
            say("PosixFilePermissions",
        Files.getPosixFilePermissions(p));
    }
}

/* 輸出:
Windows 10
Exists: true
Directory: false
Executable: true
Readable: true
RegularFile: true
Writable: true
notExists: false
Hidden: false
size: 1631
FileStore: SSD (C:)
LastModified: : 2017-05-09T12:07:00.428366Z
Owner: MINDVIEWTOSHIBA\Bruce (User)
ContentType: null
SymbolicLink: false
*/
```
在呼叫最後一個測試方法 **getPosixFilePermissions()** 之前我們需要確認一下目前文件系統是否支援 **Posix** 介面，否則會拋出執行時異常。

### **Paths**的增減修改
我們必須能透過對 **Path** 物件增加或者刪除一部分來構造一個新的 **Path** 物件。我們使用 **relativize()** 移除 **Path** 的根路徑，使用 **resolve()** 添加 **Path** 的尾路徑(不一定是“可發現”的名稱)。

對於下面程式碼中的範例，我使用 **relativize()** 方法從所有的輸出中移除根路徑，部分原因是為了示範，部分原因是為了簡化輸出結果，這說明你可以使用該方法將絕對路徑轉為相對路徑。
這個版本的程式碼中包含 **id**，以便於跟蹤輸出結果：

```java
// files/AddAndSubtractPaths.java
import java.nio.file.*;
import java.io.IOException;

public class AddAndSubtractPaths {
    static Path base = Paths.get("..", "..", "..").toAbsolutePath().normalize();
    
    static void show(int id, Path result) {
        if(result.isAbsolute())
            System.out.println("(" + id + ")r " + base.relativize(result));
        else
            System.out.println("(" + id + ") " + result);
        try {
            System.out.println("RealPath: " + result.toRealPath());
        } catch(IOException e) {
            System.out.println(e);
        }
    }
    
    public static void main(String[] args) {
        System.out.println(System.getProperty("os.name"));
        System.out.println(base);
        Path p = Paths.get("AddAndSubtractPaths.java").toAbsolutePath();
        show(1, p);
        Path convoluted = p.getParent().getParent()
        .resolve("strings").resolve("..")
        .resolve(p.getParent().getFileName());
        show(2, convoluted);
        show(3, convoluted.normalize());
        Path p2 = Paths.get("..", "..");
        show(4, p2);
        show(5, p2.normalize());
        show(6, p2.toAbsolutePath().normalize());
        Path p3 = Paths.get(".").toAbsolutePath();
        Path p4 = p3.resolve(p2);
        show(7, p4);
        show(8, p4.normalize());
        Path p5 = Paths.get("").toAbsolutePath();
        show(9, p5);
        show(10, p5.resolveSibling("strings"));
        show(11, Paths.get("nonexistent"));
    }
}

/* 輸出:
Windows 10
C:\Users\Bruce\Documents\GitHub
(1)r onjava\
ExtractedExamples\files\AddAndSubtractPaths.java
RealPath: C:\Users\Bruce\Documents\GitHub\onjava\
ExtractedExamples\files\AddAndSubtractPaths.java
(2)r on-java\ExtractedExamples\strings\..\files
RealPath: C:\Users\Bruce\Documents\GitHub\onjava\
ExtractedExamples\files
(3)r on-java\ExtractedExamples\files
RealPath: C:\Users\Bruce\Documents\GitHub\onjava\
ExtractedExamples\files
(4) ..\..
RealPath: C:\Users\Bruce\Documents\GitHub\on-java
(5) ..\..
RealPath: C:\Users\Bruce\Documents\GitHub\on-java
(6)r on-java
RealPath: C:\Users\Bruce\Documents\GitHub\on-java
(7)r on-java\ExtractedExamples\files\.\..\..
RealPath: C:\Users\Bruce\Documents\GitHub\on-java
(8)r on-java
RealPath: C:\Users\Bruce\Documents\GitHub\on-java
(9)r on-java\ExtractedExamples\files
RealPath: C:\Users\Bruce\Documents\GitHub\onjava\
ExtractedExamples\files
(10)r on-java\ExtractedExamples\strings
RealPath: C:\Users\Bruce\Documents\GitHub\onjava\
ExtractedExamples\strings
(11) nonexistent
java.nio.file.NoSuchFileException:
C:\Users\Bruce\Documents\GitHub\onjava\
ExtractedExamples\files\nonexistent
*/
```
我還為 **toRealPath()** 添加了更多的測試，這是為了擴展和規則化，防止路徑不存在時拋出執行時異常。

<!-- Directories -->

## 目錄
**Files** 工具類包含大部分我們需要的目錄操作和文件操作方法。出於某種原因，它們沒有包含刪除目錄樹相關的方法，因此我們將實現並將其添加到 **onjava** 庫中。

```java
// onjava/RmDir.java
package onjava;

import java.nio.file.*;
import java.nio.file.attribute.BasicFileAttributes;
import java.io.IOException;

public class RmDir {
    public static void rmdir(Path dir) throws IOException {
        Files.walkFileTree(dir, new SimpleFileVisitor<Path>() {
            @Override
            public FileVisitResult visitFile(Path file, BasicFileAttributes attrs) throws IOException {
                Files.delete(file);
                return FileVisitResult.CONTINUE;
            }
            
            @Override
            public FileVisitResult postVisitDirectory(Path dir, IOException exc) throws IOException {
                Files.delete(dir);
                return FileVisitResult.CONTINUE;
            }
        });
    }
}
```
刪除目錄樹的方法實現依賴於 **Files.walkFileTree()**，"walking" 目錄樹意味著遍歷每個子目錄和文件。*Visitor* 設計模式提供了一種標準機制來訪問集合中的每個物件，然後你需要提供在每個物件上執行的操作。
此操作的定義取決於實現的 **FileVisitor** 的四個抽象方法，包括：

    1.  **preVisitDirectory()**：在訪問目錄中條目之前在目錄上執行。 
    2.  **visitFile()**：執行目錄中的每一個文件。  
    3.  **visitFileFailed()**：呼叫無法訪問的文件。   
    4.  **postVisitDirectory()**：在訪問目錄中條目之後在目錄上執行，包括所有的子目錄。

為了簡化，**java.nio.file.SimpleFileVisitor** 提供了所有方法的預設實現。這樣，在我們的匿名內部類中，我們只需要重寫非標準行為的方法：**visitFile()** 和 **postVisitDirectory()** 實現刪除文件和刪除目錄。兩者都應該返回標誌位決定是否繼續訪問(這樣就可以繼續訪問，直到找到所需要的)。
作為探索目錄操作的一部分，現在我們可以有條件地刪除已存在的目錄。在以下例子中，**makeVariant()** 接受基本目錄測試，並透過旋轉部件列表生成不同的子目錄路徑。這些旋轉與路徑分隔符 **sep** 使用 **String.join()** 貼在一起，然後返回一個 **Path** 物件。

```java
// files/Directories.java
import java.util.*;
import java.nio.file.*;
import onjava.RmDir;

public class Directories {
    static Path test = Paths.get("test");
    static String sep = FileSystems.getDefault().getSeparator();
    static List<String> parts = Arrays.asList("foo", "bar", "baz", "bag");
    
    static Path makeVariant() {
        Collections.rotate(parts, 1);
        return Paths.get("test", String.join(sep, parts));
    }
    
    static void refreshTestDir() throws Exception {
        if(Files.exists(test))
        RmDir.rmdir(test);
        if(!Files.exists(test))
        Files.createDirectory(test);
    }
    
    public static void main(String[] args) throws Exception {
        refreshTestDir();
        Files.createFile(test.resolve("Hello.txt"));
        Path variant = makeVariant();
        // Throws exception (too many levels):
        try {
            Files.createDirectory(variant);
        } catch(Exception e) {
            System.out.println("Nope, that doesn't work.");
        }
        populateTestDir();
        Path tempdir = Files.createTempDirectory(test, "DIR_");
        Files.createTempFile(tempdir, "pre", ".non");
        Files.newDirectoryStream(test).forEach(System.out::println);
        System.out.println("*********");
        Files.walk(test).forEach(System.out::println);
    }
    
    static void populateTestDir() throws Exception  {
        for(int i = 0; i < parts.size(); i++) {
            Path variant = makeVariant();
            if(!Files.exists(variant)) {
                Files.createDirectories(variant);
                Files.copy(Paths.get("Directories.java"),
                    variant.resolve("File.txt"));
                Files.createTempFile(variant, null, null);
            }
        }
    }
}

/* 輸出:
Nope, that doesn't work.
test\bag
test\bar
test\baz
test\DIR_5142667942049986036
test\foo
test\Hello.txt
*********
test
test\bag
test\bag\foo
test\bag\foo\bar
test\bag\foo\bar\baz
test\bag\foo\bar\baz\8279660869874696036.tmp
test\bag\foo\bar\baz\File.txt
test\bar
test\bar\baz
test\bar\baz\bag
test\bar\baz\bag\foo
test\bar\baz\bag\foo\1274043134240426261.tmp
test\bar\baz\bag\foo\File.txt
test\baz
test\baz\bag
test\baz\bag\foo
test\baz\bag\foo\bar
test\baz\bag\foo\bar\6130572530014544105.tmp
test\baz\bag\foo\bar\File.txt
test\DIR_5142667942049986036
test\DIR_5142667942049986036\pre7704286843227113253.non
test\foo
test\foo\bar
test\foo\bar\baz
test\foo\bar\baz\bag
test\foo\bar\baz\bag\5412864507741775436.tmp
test\foo\bar\baz\bag\File.txt
test\Hello.txt
*/
```
首先，**refreshTestDir()** 用於檢測 **test** 目錄是否已經存在。若存在，則使用我們新工具類 **rmdir()** 刪除其整個目錄。檢查是否 **exists** 是多餘的，但我想說明一點，因為如果你對於已經存在的目錄呼叫 **createDirectory()** 將會拋出異常。**createFile()** 使用參數 **Path** 建立一個空文件; **resolve()** 將檔案名添加到 **test Path** 的末尾。

我們嘗試使用 **createDirectory()** 來建立多級路徑，但是這樣會拋出異常，因為這個方法只能建立單級路徑。我已經將 **populateTestDir()** 作為一個單獨的方法，因為它將在後面的例子中被重用。對於每一個變數 **variant**，我們都能使用 **createDirectories()** 建立完整的目錄路徑，然後使用此文件的副本以不同的目標名稱填充該終端目錄。然後我們使用 **createTempFile()** 生成一個暫存檔。

在呼叫 **populateTestDir()** 之後，我們在 **test** 目錄下面建立一個暫存資料夾。請注意，**createTempDirectory()** 只有名稱的前綴選項。與 **createTempFile()** 不同，我們再次使用它將暫存檔放入新的暫存資料夾中。你可以從輸出中看到，如果未指定後綴，它將預設使用".tmp"作為後綴。

為了展示結果，我們首次使用看起來很有希望的 **newDirectoryStream()**，但事實證明這個方法只是返回 **test** 目錄內容的 Stream 流，並沒有更多的內容。要獲取目錄樹的全部內容的流，請使用 **Files.walk()**。

<!-- File Systems -->

## 文件系統
為了完整起見，我們需要一種方法尋找文件系統相關的其他訊息。在這裡，我們使用靜態的 **FileSystems** 工具類獲取"預設"的文件系統，但你同樣也可以在 **Path** 物件上呼叫 **getFileSystem()** 以獲取建立該 **Path** 的文件系統。你可以獲得給定 *URI* 的文件系統，還可以構建新的文件系統(對於支持它的作業系統)。
```java
// files/FileSystemDemo.java
import java.nio.file.*;

public class FileSystemDemo {
    static void show(String id, Object o) {
        System.out.println(id + ": " + o);
    }
    
    public static void main(String[] args) {
        System.out.println(System.getProperty("os.name"));
        FileSystem fsys = FileSystems.getDefault();
        for(FileStore fs : fsys.getFileStores())
            show("File Store", fs);
        for(Path rd : fsys.getRootDirectories())
            show("Root Directory", rd);
        show("Separator", fsys.getSeparator());
        show("UserPrincipalLookupService",
            fsys.getUserPrincipalLookupService());
        show("isOpen", fsys.isOpen());
        show("isReadOnly", fsys.isReadOnly());
        show("FileSystemProvider", fsys.provider());
        show("File Attribute Views",
        fsys.supportedFileAttributeViews());
    }
}
/* 輸出:
Windows 10
File Store: SSD (C:)
Root Directory: C:\
Root Directory: D:\
Separator: \
UserPrincipalLookupService:
sun.nio.fs.WindowsFileSystem$LookupService$1@15db9742
isOpen: true
isReadOnly: false
FileSystemProvider:
sun.nio.fs.WindowsFileSystemProvider@6d06d69c
File Attribute Views: [owner, dos, acl, basic, user]
*/
```
一個 **FileSystem** 物件也能生成 **WatchService** 和 **PathMatcher** 物件，將會在接下來兩章中詳細講解。

<!-- Watching a Path -->

## 路徑監聽
通過 **WatchService** 可以設定一個行程對目錄中的更改做出響應。在這個例子中，**delTxtFiles()** 作為一個單獨的任務執行，該任務將遍歷整個目錄並刪除以 **.txt** 結尾的所有文件，**WatchService** 會對文件刪除操作做出反應：

```java
// files/PathWatcher.java
// {ExcludeFromGradle}
import java.io.IOException;
import java.nio.file.*;
import static java.nio.file.StandardWatchEventKinds.*;
import java.util.concurrent.*;

public class PathWatcher {
    static Path test = Paths.get("test");
    
    static void delTxtFiles() {
        try {
            Files.walk(test)
            .filter(f ->
                f.toString()
                .endsWith(".txt"))
                .forEach(f -> {
                try {
                    System.out.println("deleting " + f);
                    Files.delete(f);
                } catch(IOException e) {
                    throw new RuntimeException(e);
                }
            });
        } catch(IOException e) {
            throw new RuntimeException(e);
        }
    }
    
    public static void main(String[] args) throws Exception {
        Directories.refreshTestDir();
        Directories.populateTestDir();
        Files.createFile(test.resolve("Hello.txt"));
        WatchService watcher = FileSystems.getDefault().newWatchService();
        test.register(watcher, ENTRY_DELETE);
        Executors.newSingleThreadScheduledExecutor()
        .schedule(PathWatcher::delTxtFiles,
        250, TimeUnit.MILLISECONDS);
        WatchKey key = watcher.take();
        for(WatchEvent evt : key.pollEvents()) {
            System.out.println("evt.context(): " + evt.context() +
            "\nevt.count(): " + evt.count() +
            "\nevt.kind(): " + evt.kind());
            System.exit(0);
        }
    }
}
/* Output:
deleting test\bag\foo\bar\baz\File.txt
deleting test\bar\baz\bag\foo\File.txt
deleting test\baz\bag\foo\bar\File.txt
deleting test\foo\bar\baz\bag\File.txt
deleting test\Hello.txt
evt.context(): Hello.txt
evt.count(): 1
evt.kind(): ENTRY_DELETE
*/
```

**delTxtFiles()** 中的 **try** 程式碼塊看起來有些多餘，因為它們捕獲的是同一種類型的異常，外部的 **try** 語句似乎已經足夠了。然而出於某種原因，Java 要求兩者都必須存在(這也可能是一個 bug)。還要注意的是在 **filter()** 中，我們必須顯式地使用 **f.toString()** 轉為字串，否則我們呼叫 **endsWith()** 將會與整個 **Path** 物件進行比較，而不是路徑名稱字串的一部分進行比較。

一旦我們從 **FileSystem** 中得到了 **WatchService** 物件，我們將其註冊到 **test** 路徑以及我們感興趣的項目的變數參數列表中，可以選擇 **ENTRY_CREATE**，**ENTRY_DELETE** 或 **ENTRY_MODIFY**(其中建立和刪除不屬於修改)。

因為接下來對 **watcher.take()** 的呼叫會在發生某些事情之前停止所有操作，所以我們希望 **deltxtfiles()** 能夠並行執行以便生成我們感興趣的事件。為了實現這個目的，我透過呼叫 **Executors.newSingleThreadScheduledExecutor()** 產生一個 **ScheduledExecutorService** 物件，然後呼叫 **schedule()** 方法傳遞所需函數的方法引用，並且設定在執行之前應該等待的時間。

此時，**watcher.take()** 將等待並阻塞在這裡。當目標事件發生時，會返回一個包含 **WatchEvent** 的 **Watchkey** 物件。展示的這三種方法是能對 **WatchEvent** 執行的全部操作。

查看輸出的具體內容。即使我們正在刪除以 **.txt** 結尾的文件，在 **Hello.txt** 被刪除之前，**WatchService** 也不會被觸發。你可能認為，如果說"監視這個目錄"，自然會包含整個目錄和下面子目錄，但實際上：只會監視給定的目錄，而不是下面的所有內容。如果需要監視整個樹目錄，必須在整個樹的每個子目錄上放置一個 **Watchservice**。

```java
// files/TreeWatcher.java
// {ExcludeFromGradle}
import java.io.IOException;
import java.nio.file.*;
import static java.nio.file.StandardWatchEventKinds.*;
import java.util.concurrent.*;

public class TreeWatcher {

    static void watchDir(Path dir) {
        try {
            WatchService watcher =
            FileSystems.getDefault().newWatchService();
            dir.register(watcher, ENTRY_DELETE);
            Executors.newSingleThreadExecutor().submit(() -> {
                try {
                    WatchKey key = watcher.take();
                    for(WatchEvent evt : key.pollEvents()) {
                        System.out.println(
                        "evt.context(): " + evt.context() +
                        "\nevt.count(): " + evt.count() +
                        "\nevt.kind(): " + evt.kind());
                        System.exit(0);
                    }
                } catch(InterruptedException e) {
                    return;
                }
            });
        } catch(IOException e) {
            throw new RuntimeException(e);
        }
    }
    
    public static void main(String[] args) throws Exception {
        Directories.refreshTestDir();
        Directories.populateTestDir();
        Files.walk(Paths.get("test"))
            .filter(Files::isDirectory)
            .forEach(TreeWatcher::watchDir);
        PathWatcher.delTxtFiles();
    }
}

/* Output:
deleting test\bag\foo\bar\baz\File.txt
deleting test\bar\baz\bag\foo\File.txt
evt.context(): File.txt
evt.count(): 1
evt.kind(): ENTRY_DELETE
*/
```

在 **watchDir()** 方法中給 **WatchSevice** 提供參數 **ENTRY_DELETE**，並啟動一個獨立的執行緒來監視該**Watchservice**。這裡我們沒有使用 **schedule()** 進行啟動，而是使用 **submit()** 啟動執行緒。我們遍歷整個目錄樹，並將 **watchDir()** 應用於每個子目錄。現在，當我們執行 **deltxtfiles()** 時，其中一個 **Watchservice** 會檢測到每一次文件刪除。

<!-- Finding Files -->

## 文件尋找
到目前為止，為了找到文件，我們一直使用相當粗糙的方法，在 `path` 上呼叫 `toString()`，然後使用 `string` 操作查看結果。事實證明，`java.nio.file` 有更好的解決方案：透過在 `FileSystem` 物件上呼叫 `getPathMatcher()` 獲得一個 `PathMatcher`，然後傳入您感興趣的模式。模式有兩個選項：`glob` 和 `regex`。`glob` 比較簡單，實際上功能非常強大，因此您可以使用 `glob` 解決許多問題。如果您的問題更複雜，可以使用 `regex`，這將在接下來的 `Strings` 一章中解釋。

在這裡，我們使用 `glob` 尋找以 `.tmp` 或 `.txt` 結尾的所有 `Path`：

```java
// files/Find.java
// {ExcludeFromGradle}
import java.nio.file.*;

public class Find {
    public static void main(String[] args) throws Exception {
        Path test = Paths.get("test");
        Directories.refreshTestDir();
        Directories.populateTestDir();
        // Creating a *directory*, not a file:
        Files.createDirectory(test.resolve("dir.tmp"));

        PathMatcher matcher = FileSystems.getDefault()
          .getPathMatcher("glob:**/*.{tmp,txt}");
        Files.walk(test)
          .filter(matcher::matches)
          .forEach(System.out::println);
        System.out.println("***************");

        PathMatcher matcher2 = FileSystems.getDefault()
          .getPathMatcher("glob:*.tmp");
        Files.walk(test)
          .map(Path::getFileName)
          .filter(matcher2::matches)
          .forEach(System.out::println);
        System.out.println("***************");

        Files.walk(test) // Only look for files
          .filter(Files::isRegularFile)
          .map(Path::getFileName)
          .filter(matcher2::matches)
          .forEach(System.out::println);
    }
}
/* Output:
test\bag\foo\bar\baz\5208762845883213974.tmp
test\bag\foo\bar\baz\File.txt
test\bar\baz\bag\foo\7918367201207778677.tmp
test\bar\baz\bag\foo\File.txt
test\baz\bag\foo\bar\8016595521026696632.tmp
test\baz\bag\foo\bar\File.txt
test\dir.tmp
test\foo\bar\baz\bag\5832319279813617280.tmp
test\foo\bar\baz\bag\File.txt
***************
5208762845883213974.tmp
7918367201207778677.tmp
8016595521026696632.tmp
dir.tmp
5832319279813617280.tmp
***************
5208762845883213974.tmp
7918367201207778677.tmp
8016595521026696632.tmp
5832319279813617280.tmp
*/
```

在 `matcher` 中，`glob` 表達式開頭的 `**/` 表示“目前目錄及所有子目錄”，這在當你不僅僅要匹配目前目錄下特定結尾的 `Path` 時非常有用。單 `*` 表示“任何東西”，然後是一個點，然後大括號表示一系列的可能性---我們正在尋找以 `.tmp` 或 `.txt` 結尾的東西。您可以在 `getPathMatcher()` 文件中找到更多詳細訊息。

`matcher2` 只使用 `*.tmp`，通常不匹配任何內容，但是添加 `map()` 操作會將完整路徑減少到末尾的名稱。

注意，在這兩種情況下，輸出中都會出現 `dir.tmp`，即使它是一個目錄而不是一個文件。要只尋找文件，必須像在最後 `files.walk()` 中那樣對其進行篩選。

<!-- Reading & Writing Files -->
## 文件讀寫
此時，我們可以對路徑和目錄做任何事情。 現在讓我們看一下操縱文件本身的內容。

如果一個文件很“小”，也就是說“它執行得足夠快且占用記憶體小”，那麼 `java.nio.file.Files` 類中的實用程式將幫助你輕鬆讀寫文字和二進位制文件。

`Files.readAllLines()` 一次讀取整個文件（因此，“小”文件很有必要），產生一個`List<String>`。 對於範例文件，我們將重用`streams/Cheese.dat`：

```java
// files/ListOfLines.java
import java.util.*;
import java.nio.file.*;

public class ListOfLines {
    public static void main(String[] args) throws Exception {
        Files.readAllLines(
        Paths.get("../streams/Cheese.dat"))
        .stream()
        .filter(line -> !line.startsWith("//"))
        .map(line ->
            line.substring(0, line.length()/2))
        .forEach(System.out::println);
    }
}
/* Output:
Not much of a cheese
Finest in the
And what leads you
Well, it's
It's certainly uncon
*/
```

跳過注釋行，其餘的內容每行只列印一半。 這實現起來很簡單：你只需將 `Path` 傳遞給 `readAllLines()` （以前的 java 實現這個功能很複雜）。`readAllLines()` 有一個重載版本，包含一個 `Charset` 參數來儲存文件的 Unicode 編碼。

`Files.write()` 被重載以寫入 `byte` 陣列或任何 `Iterable` 物件（它也有 `Charset` 選項）：

```java
// files/Writing.java
import java.util.*;
import java.nio.file.*;

public class Writing {
    static Random rand = new Random(47);
    static final int SIZE = 1000;
    
    public static void main(String[] args) throws Exception {
        // Write bytes to a file:
        byte[] bytes = new byte[SIZE];
        rand.nextBytes(bytes);
        Files.write(Paths.get("bytes.dat"), bytes);
        System.out.println("bytes.dat: " + Files.size(Paths.get("bytes.dat")));

        // Write an iterable to a file:
        List<String> lines = Files.readAllLines(
          Paths.get("../streams/Cheese.dat"));
        Files.write(Paths.get("Cheese.txt"), lines);
        System.out.println("Cheese.txt: " + Files.size(Paths.get("Cheese.txt")));
    }
}
/* Output:
bytes.dat: 1000
Cheese.txt: 199
*/
```

我們使用 `Random` 來建立一個隨機的 `byte` 陣列; 你可以看到生成的檔案大小是 1000。

一個 `List` 被寫入檔案，任何 `Iterable` 對像也可以這麼做。

如果檔案大小有問題怎麼辦？ 比如說：

1. 文件太大，如果你一次性讀完整個文件，你可能會耗盡記憶體。

2. 您只需要在文件的中途工作以獲得所需的結果，因此讀取整個文件會浪費時間。

`Files.lines()` 方便地將文件轉換為行的 `Stream`：

```java
// files/ReadLineStream.java
import java.nio.file.*;

public class ReadLineStream {
    public static void main(String[] args) throws Exception {
        Files.lines(Paths.get("PathInfo.java"))
          .skip(13)
          .findFirst()
          .ifPresent(System.out::println);
    }
}
/* Output:
    show("RegularFile", Files.isRegularFile(p));
*/
```

這對本章中第一個範例程式碼做了流式處理，跳過 13 行，然後選擇下一行並將其列印出來。

`Files.lines()` 對於把文件處理行的傳入流時非常有用，但是如果你想在 `Stream` 中讀取，處理或寫入怎麼辦？這就需要稍微複雜的程式碼：

```java
// files/StreamInAndOut.java
import java.io.*;
import java.nio.file.*;
import java.util.stream.*;

public class StreamInAndOut {
    public static void main(String[] args) {
        try(
          Stream<String> input =
            Files.lines(Paths.get("StreamInAndOut.java"));
          PrintWriter output =
            new PrintWriter("StreamInAndOut.txt")
        ) {
            input.map(String::toUpperCase)
              .forEachOrdered(output::println);
        } catch(Exception e) {
            throw new RuntimeException(e);
        }
    }
}
```

因為我們在同一個塊中執行所有操作，所以這兩個文件都可以在相同的 try-with-resources 語句中打開。`PrintWriter` 是一個舊式的 `java.io` 類，允許你“列印”到一個文件，所以它是這個應用的理想選擇。如果你看一下 `StreamInAndOut.txt`，你會發現它裡面的內容確實是大寫的。

<!-- Summary -->

## 本章小結
雖然本章對文件和目錄操作做了相當全面的介紹，但是仍然有沒被介紹的類庫中的功能——一定要研究 `java.nio.file` 的 Javadocs，尤其是 `java.nio.file.Files` 這個類。

Java 7 和 8 對於處理文件和目錄的類庫做了大量改進。如果您剛剛開始使用 Java，那麼您很幸運。在過去，它令人非常不愉快，我確信 Java 設計者以前對於文件操作不夠重視才沒做簡化。對於初學者來說這是一件很棒的事，對於教學者來說也一樣。我不明白為什麼花了這麼長時間來解決這個明顯的問題，但不管怎麼說它被解決了，我很高興。使用文件現在很簡單，甚至很有趣，這是你以前永遠想不到的。

<!-- 分頁 -->

<div style="page-break-after: always;"></div>
