[TOC]

<!-- Appendix: Standard I/O -->
# 附錄:標準IO

>*標準 I/O*這個術語參考Unix中的概念，指程式所使用的單一訊息流（這種思想在大多數作業系統中，也有相似形式的實現）。

程式的所有輸入都可以來自於*標準輸入*，其所有輸出都可以流向*標準輸出*，並且其所有錯誤訊息均可以發送到*標準錯誤*。*標準 I/O* 的意義在於程式之間可以很容易地連接起來，一個程式的標準輸出可以作為另一個程式的標準輸入。這是一個非常強大的工具。

## 從標準輸入中讀取

遵循標準 I/O 模型，Java 提供了標準輸入流 `System.in`、標準輸出流 `System.out` 和標準錯誤流 `System.err`。在本書中，你已經了解到如何使用 `System.out`將資料寫到標準輸出。 `System.out` 已經預先包裝[^1]成了 `PrintStream` 物件。標準錯誤流 `System.err` 也預先包裝為 `PrintStream` 物件，但是標準輸入流 `System.in` 是原生的沒有經過包裝的 `InputStream`。這意味著儘管可以直接使用標準輸出流 `System.out` 和標準錯誤流 `System.err`，但是在讀取 `System.in` 之前必須先對其進行包裝。

我們通常一次一行地讀取輸入。為了實現這個功能，將 `System.in` 包裝成 `BufferedReader` 來使用，這要求我們用 `InputStreamReader` 把 `System.in` 轉換[^2]成 `Reader` 。下面這個例子將鍵入的每一行顯示出來：

```java
// standardio/Echo.java
// How to read from standard input
import java.io.*;
import onjava.TimedAbort;

public class Echo {
    public static void main(String[] args) {
        TimedAbort abort = new TimedAbort(2);
        new BufferedReader(
                new InputStreamReader(System.in))
                .lines()
                .peek(ln -> abort.restart())
                .forEach(System.out::println);
        // Ctrl-Z or two seconds inactivity
        // terminates the program
    }
}
```

`BufferedReader` 提供了 `lines()` 方法，返回類型是 `Stream<String>` 。這顯示出流模型的的靈活性：僅使用標準輸入就能很好地工作。 `peek()` 方法重啟 `TimeAbort`，只要保證至少每隔兩秒有輸入就能夠使程式保持開啟狀態。

## 將 System.out 轉換成 PrintWriter

`System.out` 是一個 `PrintStream`，而 `PrintStream` 是一個`OutputStream`。 `PrintWriter` 有一個把 `OutputStream` 作為參數的構造器。因此，如果你需要的話，可以使用這個構造器把 `System.out` 轉換成 `PrintWriter` 。

```java
// standardio/ChangeSystemOut.java
// Turn System.out into a PrintWriter

import java.io.*;

public class ChangeSystemOut {
    public static void main(String[] args) {
        PrintWriter out =
                new PrintWriter(System.out, true);
        out.println("Hello, world");
    }
}
```

輸出結果：

```
Hello, world
```

要使用 `PrintWriter` 帶有兩個參數的構造器，並設定第二個參數為 `true`，從而使能自動重新整理到輸出緩衝區的功能；否則，可能無法看到列印輸出。

## 重定向標準 I/O

Java的 `System` 類提供了簡單的 `static` 方法呼叫，從而能夠重定向標準輸入流、標準輸出流和標準錯誤流：
- setIn（InputStream）
- setOut（PrintStream）
- setErr(PrintStream)

如果我們突然需要在顯示器上建立大量的輸出，而這些輸出滾動的速度太快以至於無法閱讀時，重定向輸出就顯得格外有用，可把輸出內容重定向到文件中供後續查看。對於我們想重複測試特定的使用者輸入序列的命令列程式來說，重定向輸入就很有價值。下例簡單示範了這些方法的使用：

```java
// standardio/Redirecting.java
// Demonstrates standard I/O redirection
import java.io.*;

public class Redirecting {
    public static void main(String[] args) {
        PrintStream console = System.out;
        try (
                BufferedInputStream in = new BufferedInputStream(
                        new FileInputStream("Redirecting.java"));
                PrintStream out = new PrintStream(
                        new BufferedOutputStream(
                                new FileOutputStream("Redirecting.txt")))
        ) {
            System.setIn(in);
            System.setOut(out);
            System.setErr(out);
            new BufferedReader(
                    new InputStreamReader(System.in))
                    .lines()
                    .forEach(System.out::println);
        } catch (IOException e) {
            throw new RuntimeException(e);
        } finally {
            System.setOut(console);
        }
    }
}
```

該程式將文件中內容載入到標準輸入，並把標準輸出和標準錯誤重定向到另一個文件。它在程式的開始儲存了最初對 `System.out` 物件的引用，並且在程式結束時將系統輸出復原到了該物件上。

I/O重定向操作的是位元組流而不是字元流，因此使用 `InputStream` 和 `OutputStream`，而不是 `Reader` 和 `Writer`。

<!-- Process Control -->
## 執行控制

你經常需要在Java內部直接執行作業系統的程式，並控制這些程式的輸入輸出，Java類庫提供了執行這些操作的類。

一項常見的任務是執行程式並將輸出結果發送到控制台。本節包含了一個可以簡化此任務的實用工具。

在使用這個工具時可能會產生兩種類型的錯誤：導致異常的普通錯誤——對於這些錯誤我們只需要重新拋出一個 `RuntimeException` 即可，以及行程自身的執行過程中導致的錯誤——我們需要用單獨的異常來報告這些錯誤：

```java
// onjava/OSExecuteException.java
package onjava;

public class OSExecuteException extends RuntimeException {
    public OSExecuteException(String why) {
        super(why);
    }
}
```

為了執行程式，我們需要傳遞給 `OSExecute.command()` 一個 `String command`，我們可以在控制台鍵入同樣的指令執行程式。該指令傳遞給 `java.lang.ProcessBuilder` 的構造器（需要將其作為 `String` 物件的序列），然後啟動生成的 `ProcessBuilder` 物件。

```java
// onjava/OSExecute.java
// Run an operating system command
// and send the output to the console
package onjava;
import java.io.*;

public class OSExecute {
    public static void command(String command) {
        boolean err = false;
        try {
            Process process = new ProcessBuilder(
                    command.split(" ")).start();
            try (
                    BufferedReader results = new BufferedReader(
                            new InputStreamReader(
                                    process.getInputStream()));
                    BufferedReader errors = new BufferedReader(
                            new InputStreamReader(
                                    process.getErrorStream()))
            ) {
                results.lines()
                        .forEach(System.out::println);
                err = errors.lines()
                        .peek(System.err::println)
                        .count() > 0;
            }
        } catch (IOException e) {
            throw new RuntimeException(e);
        }
        if (err)
            throw new OSExecuteException(
                    "Errors executing " + command);
    }
}
```

為了捕獲在程式執行時產生的標準輸出流，我們可以呼叫 `getInputStream()`。這是因為 `InputStream` 是我們可以從中讀取訊息的流。

這裡這些行只是被列印了出來，但是你也可以從 `command()` 捕獲和返回它們。

該程式的錯誤被發送到了標準錯誤流，可以呼叫 `getErrorStream()` 捕獲。如果存在任何錯誤，它們都會被列印並且拋出 `OSExcuteException` ，以便呼叫程式處理這個問題。

下面是展示如何使用 `OSExecute` 的範例：

```java
// standardio/OSExecuteDemo.java
// Demonstrates standard I/O redirection
// {javap -cp build/classes/main OSExecuteDemo}
import onjava.*;

public class OSExecuteDemo {}
```

這裡使用 `javap` 反編譯器（隨JDK發布）來反編譯程式，編譯結果：

```
Compiled from "OSExecuteDemo.java"
public class OSExecuteDemo {
  public OSExecuteDemo();
}
```

[^1]: 譯者註：這裡用到了**裝飾器模式**。

[^2]: 譯者註：這裡用到了**適配器模式**。

<!-- 分頁 -->

<div style="page-break-after: always;"></div>
