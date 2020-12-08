[TOC]

<!-- Appendix: Low-Level Concurrency -->
# 附錄:並發底層原理

> 儘管不建議你自己編寫底層 Java 並發程式碼，但是這樣通常有助於了解它是如何工作的。

[並發編程](./24-Concurrent-Programming.md) 章節中介紹了一些用於進階並發的概念，包括為 Java 並發編程而最新提出的，更安全的概念（ parallel Streams 和 CompletableFutures  ）。本附錄則介紹在 Java 中底層並發概念，因此在閱讀本篇時，你能有所了解掌握這些程式碼。你還會將進一步了解並發的普遍問題。

在 Java 的早期版本中, 底層並發概念是並發編程的重要組成部分。我們會著眼於圍繞這些技巧的複雜性以及為何你應該避免它們而談。 “並發編程” 章節展示最新的 Java 版本(尤其是 Java 8)所提供的改進技巧，這些技巧使得並發的使用，如果本來不容易使用，也會變得更容易些。

<!-- What is a Thread? -->
## 什麼是執行緒？

並發將程式劃分成獨立分離執行的任務。每個任務都由一個 *執行執行緒* 來驅動，我們通常將其簡稱為 *執行緒* 。而一個 *執行緒* 就是作業系統行程中單一順序的控制流。因此，單個行程可以有多個並發執行的任務，但是你的程式使得每個任務都好像有自己的處理器一樣。此執行緒模型為編程帶來了便利，它簡化了在單一程式中處理變戲法般的多任務過程。作業系統則從處理器上分配時間片到你程式的所有執行緒中。

Java 並發的核心機制是 **Thread** 類，在該語言最初版本中， **Thread （執行緒）** 是由程式設計師直接建立和管理的。隨著語言的發展以及人們發現了更好的一些方法，中間層機制 - 特別是 **Executor** 框架  - 被添加進來，以消除自己管理執行緒時候的心理負擔（及錯誤）。 最終，甚至發展出比 **Executor** 更好的機制，如 [並發編程](./24-Concurrent-Programming.md) 一章所示。

**Thread（執行緒）** 是將任務關聯到處理器的軟體概念。雖然建立和使用 **Thread**  類看起來與任何其他類都很相似，但實際上它們是非常不同的。當你建立一個 **Thread** 時，JVM 將分配一大塊記憶體到專為執行緒保留的特殊區域上，用於提供執行任務時所需的一切，包括：

* 程式計數器，指明要執行的下一個 JVM 位元組碼指令。
* 用於支援 Java 程式碼執行的堆疊，包含有關此執行緒已到達當時執行位置所呼叫方法的訊息。它也包含每個正在執行的方法的所有局部變數(包括原語和堆物件的引用)。每個執行緒的堆疊通常在 64K 到 1M 之間 [^1] 。
* 第二個則用於 native code（本機方法程式碼）執行的堆疊
* *thread-local variables* （執行緒本機變數）的儲存區域
* 用於控制執行緒的狀態管理變數

包括 `main()` 在內的所有程式碼都會在某個執行緒內執行。 每當呼叫一個方法時，目前程式計數器被推到該執行緒的堆疊上，然後堆疊指標向下移動以足夠來建立一個堆疊幀，其堆疊幀裡儲存該方法的所有局部變數，參數和返回值。所有基本類型變數都直接在堆疊上，雖然方法中建立（或方法中使用）物件的任何引用都位於堆疊幀中，但物件本身存於堆中。這僅且只有一個堆，被程式中所有執行緒所共享。

除此以外，執行緒必須綁定到作業系統，這樣它就可以在某個時候連接到處理器。這是作為執行緒構建過程的一部分為你管理的。Java 使用底層作業系統中的機制來管理執行緒的執行。

### 最佳執行緒數

如果你查看第 24 章 [並發編程](./24-Concurrent-Programming.md) 中使用 *CachedThreadPool* 的用例，你會發現 **ExecutorService** 為每個我們提交的任務分配一個執行緒。然而，並行流（**parallel Stream**）在 [**CountingStream.java** ](https://github.com/BruceEckel/OnJava8-Examples/blob/master/concurrent/CountingStream.java
) 中只分配了 8 個執行緒（id 中 1-7 為工作執行緒，8 為  `main()` 方法的主執行緒，它巧妙地將其用作額外的並行流）。如果你嘗試提高 `range()` 方法中的上限值，你會看到沒有建立額外的執行緒。這是為什麼？

我們可以查出目前機器上處理器的數量：

```Java
// lowlevel/NumberOfProcessors.java

public class NumberOfProcessors {
  public static void main(String[] args) {
    System.out.println(
    Runtime.getRuntime().availableProcessors());
  }
}
/* Output:
8
*/
```

在我的機器上（使用英特爾酷睿i7），我有四個核心，每個核心呈現兩個*超執行緒*（指一種硬體技巧，能在單個處理器上產生非常快速的上下文切換，在某些情況下可以使核心看起來像執行兩個硬體執行緒）。雖然這是 “最近” 電腦上的常見配置(在撰寫本文時)，但你可能會看到不同的結果，包括 **CountingStream.java ** 中同等數量的預設執行緒。

你的作業系統可能有辦法來查出關於處理器的更多訊息，例如，在Windows 10上，按下 “開始” 鍵，輸入 “任務管理器” 和 Enter 鍵。點擊 “詳細訊息” 。選擇 “性能” 標籤,你將會看到各式各樣的關於你的硬體訊息,包括“核心” 和 “邏輯處理器” 。

事實證明，“通用”執行緒的最佳數量就算是可用處理器的數量(對於特定的問題可能不是這樣)。這原因來自在Java執行緒之間切換上下文的代價：儲存被掛斷執行緒的目前狀態，並檢索另一個執行緒的目前狀態，以便從它進入掛斷的位置繼續執行。對於 8 個處理器和 8 個（計算密集型）Java執行緒，JVM 在執行這8個任務時從不需要切換上下文。對於比處理器數量少的任務，分配更多執行緒沒有幫助。

定義了 “邏輯處理器” 數量的 Intel 超執行緒，但並沒有增加計算能力 - 該特性在硬體級別維護額外的執行緒上下文，從而加快了上下文切換，這有助於提高使用者介面的響應能力。對於計算密集型任務，請考慮將執行緒數量與物理核心(而不是超執行緒)的數量匹配。儘管Java認為每個超執行緒都是一個處理器，但這似乎是由於 Intel 對超執行緒的過度行銷造成的錯誤。儘管如此，為了簡化編程，我只允許 JVM 決定預設的執行緒數。 你將需要試驗你的產品應用。 這並不意味著將執行緒數與處理器數相匹配就適用於所有問題; 相反，它主要用於計算密集型解決方案。

### 我可以建立多少個執行緒？

Thread（執行緒）物件的最大部分是用於執行方法的 Java 堆疊。查看 Thread （執行緒）物件的大小因作業系統而異。該程式透過建立 Thread 物件來測試它，直到 JVM 記憶體不足為止：

```java
// lowlevel/ThreadSize.java
// {ExcludeFromGradle} Takes a long time or hangs
import java.util.concurrent.*;
import onjava.Nap;

public class ThreadSize {
  static class Dummy extends Thread {
    @Override
    public void run() { new Nap(1); }
  }
  public static void main(String[] args) {
    ExecutorService exec =
      Executors.newCachedThreadPool();
    int count = 0;
    try {
      while(true) {
        exec.execute(new Dummy());
        count++;
      }
    } catch(Error e) {
      System.out.println(
      e.getClass().getSimpleName() + ": " + count);
      System.exit(0);
    } finally {
      exec.shutdown();
    }
  }
}
```

只要你不斷遞交任務，**CachedThreadPool** 就會繼續建立執行緒。將 **Dummy** 物件遞交到 `execute()` 方法以開始任務，如果執行緒池無可用執行緒，則分配一個新執行緒。執行的暫停方法 `pause()` 執行時間必須足夠長，使任務不會開始即完成(從而為新任務釋放現有執行緒)。只要任務不斷進入而沒有完成，**CachedThreadPool** 最終就會耗盡記憶體。

我並不總是能夠在我嘗試的每台機器上造成記憶體不足的錯誤。在一台機器上，我看到這樣的結果:

```shell
> java ThreadSize
OutOfMemoryError: 2816
```

我們可以使用 **-Xss** 標記減少每個執行緒堆疊分配的記憶體大小。允許的最小執行緒堆疊大小是 64k:

```shell
>java -Xss64K ThreadSize
OutOfMemoryError: 4952
```

如果我們將執行緒堆疊大小增加到 2M ，我們就可以分配更少的執行緒。

```shell
>java -Xss2M ThreadSize
OutOfMemoryError: 722
```

Windows 作業系統預設堆疊大小是 320K，我們可以通過驗證它給出的數字與我們完全不設定堆疊大小時的數字是大致相同:

```shell
>java -Xss320K ThreadSize
OutOfMemoryError: 2816
```

你還可以使用 **-Xmx** 標誌增加 JVM 的最大記憶體分配:

```shell
>java -Xss64K -Xmx5M ThreadSize
OutOfMemoryError: 5703
```

請注意的是作業系統還可能對允許的執行緒數施加限制。

因此，“我可以擁有多少執行緒”這一問題的答案是“幾千個”。但是，如果你發現自己分配了數千個執行緒，那麼你可能需要重新考慮你的做法; 恰當的問題是“我需要多少執行緒？”

### The WorkStealingPool (工作竊取執行緒池)

這是一個 **ExecutorService** ，它使用所有可用的(由JVM報告) 處理器自動建立執行緒池。

```java
// lowlevel/WorkStealingPool.java
import java.util.stream.*;
import java.util.concurrent.*;

class ShowThread implements Runnable {
  @Override
  public void run() {
    System.out.println(
    Thread.currentThread().getName());
  }
}

public class WorkStealingPool {
  public static void main(String[] args)
    throws InterruptedException {
    System.out.println(
      Runtime.getRuntime().availableProcessors());
    ExecutorService exec =
      Executors.newWorkStealingPool();
    IntStream.range(0, 10)
      .mapToObj(n -> new ShowThread())
      .forEach(exec::execute);
    exec.awaitTermination(1, TimeUnit.SECONDS);
  }
}
/* Output:
8
ForkJoinPool-1-worker-2
ForkJoinPool-1-worker-1
ForkJoinPool-1-worker-2
ForkJoinPool-1-worker-3
ForkJoinPool-1-worker-2
ForkJoinPool-1-worker-1
ForkJoinPool-1-worker-3
ForkJoinPool-1-worker-1
ForkJoinPool-1-worker-4
ForkJoinPool-1-worker-2
*/
```

工作竊取演算法允許已經耗盡輸入佇列中的工作項的執行緒從其他佇列“竊取”工作項。目標是在處理器之間分配工作項，從而最大限度地利用所有可用的處理器來完成計算密集型任務。這項演算法也用於 Java 的fork/join 框架。

<!-- Catching Exceptions -->
## 異常捕獲

這可能會讓你感到驚訝：

```java
// lowlevel/SwallowedException.java
import java.util.concurrent.*;

public class SwallowedException {
  public static void main(String[] args)
    throws InterruptedException {
    ExecutorService exec =
      Executors.newSingleThreadExecutor();
    exec.submit(() -> {
      throw new RuntimeException();
    });
    exec.shutdown();
  }
}
```

這個程式什麼也不輸出（然而，如果你用 **execute** 方法取代 `submit()` 方法，你就將會看到異常拋出。這說明在執行緒中拋出異常是很棘手的，需要特別注意的事情。

你無法捕獲到從執行緒逃逸的異常。一旦異常越過了任務的 `run()` 方法，它就會傳遞至控制台，除非你採取特殊步驟來捕獲此類錯誤異常。

下面是一個拋出異常的程式碼，該異常會傳遞到它的 `run()` 方法之外，而 `main()` 方法會顯示執行它時會發生什麼事：

```java
// lowlevel/ExceptionThread.java
// {ThrowsException}
import java.util.concurrent.*;

public class ExceptionThread implements Runnable {
  @Override
  public void run() {
    throw new RuntimeException();
  }
  public static void main(String[] args) {
    ExecutorService es =
      Executors.newCachedThreadPool();
    es.execute(new ExceptionThread());
    es.shutdown();
  }
}
/* Output:
___[ Error Output ]___
Exception in thread "pool-1-thread-1"
java.lang.RuntimeException
        at ExceptionThread.run(ExceptionThread.java:8)
        at java.util.concurrent.ThreadPoolExecutor.runW
orker(ThreadPoolExecutor.java:1142)
        at java.util.concurrent.ThreadPoolExecutor$Work
er.run(ThreadPoolExecutor.java:617)
        at java.lang.Thread.run(Thread.java:745)
*/
```

輸出是(經過調整一些限定符以適應閱讀)：

```
Exception in thread "pool-1-thread-1" RuntimeException
  at ExceptionThread.run(ExceptionThread.java:9)
  at ThreadPoolExecutor.runWorker(...)
  at ThreadPoolExecutor$Worker.run(...)
  at java.lang.Thread.run(Thread.java:745)
```

即使在 `main()` 方法體內包裹 **try-catch** 程式碼塊來捕獲異常也不成功：

```java
// lowlevel/NaiveExceptionHandling.java
// {ThrowsException}
import java.util.concurrent.*;

public class NaiveExceptionHandling {
  public static void main(String[] args) {
    ExecutorService es =
      Executors.newCachedThreadPool();
    try {
      es.execute(new ExceptionThread());
    } catch(RuntimeException ue) {
      // This statement will NOT execute!
      System.out.println("Exception was handled!");
    } finally {
      es.shutdown();
    }
  }
}
/* Output:
___[ Error Output ]___
Exception in thread "pool-1-thread-1"
java.lang.RuntimeException
        at ExceptionThread.run(ExceptionThread.java:8)
        at java.util.concurrent.ThreadPoolExecutor.runW
orker(ThreadPoolExecutor.java:1142)
        at java.util.concurrent.ThreadPoolExecutor$Work
er.run(ThreadPoolExecutor.java:617)
        at java.lang.Thread.run(Thread.java:745)
*/
```

這會產生與前一個範例相同的結果:未捕獲異常。

為解決這個問題，需要改變 **Executor** （執行器）生成執行緒的方式。 **Thread.UncaughtExceptionHandler** 是一個添加給每個 **Thread** 物件，用於進行異常處理的介面。

當該執行緒即將死於未捕獲的異常時，將自動呼叫 `Thread.UncaughtExceptionHandler.uncaughtException()`
 方法。為了呼叫該方法，我們建立一個新的 **ThreadFactory** 類型來讓 **Thread.UncaughtExceptionHandler** 物件附加到每個它所新建立的 **Thread**（執行緒）物件上。我們賦值該工廠物件給 **Executors** 物件的 方法，讓它的方法來生成新的 **ExecutorService** 物件：

```java
// lowlevel/CaptureUncaughtException.java
import java.util.concurrent.*;

class ExceptionThread2 implements Runnable {
  @Override
  public void run() {
    Thread t = Thread.currentThread();
    System.out.println("run() by " + t.getName());
    System.out.println(
      "eh = " + t.getUncaughtExceptionHandler());
    throw new RuntimeException();
  }
}

class MyUncaughtExceptionHandler implements
Thread.UncaughtExceptionHandler {
  @Override
  public void uncaughtException(Thread t, Throwable e) {
    System.out.println("caught " + e);
  }
}

class HandlerThreadFactory implements ThreadFactory {
  @Override
  public Thread newThread(Runnable r) {
    System.out.println(this + " creating new Thread");
    Thread t = new Thread(r);
    System.out.println("created " + t);
    t.setUncaughtExceptionHandler(
      new MyUncaughtExceptionHandler());
    System.out.println(
      "eh = " + t.getUncaughtExceptionHandler());
    return t;
  }
}

public class CaptureUncaughtException {
  public static void main(String[] args) {
    ExecutorService exec =
      Executors.newCachedThreadPool(
        new HandlerThreadFactory());
    exec.execute(new ExceptionThread2());
    exec.shutdown();
  }
}
/* Output:
HandlerThreadFactory@4e25154f creating new Thread
created Thread[Thread-0,5,main]
eh = MyUncaughtExceptionHandler@70dea4e
run() by Thread-0
eh = MyUncaughtExceptionHandler@70dea4e
caught java.lang.RuntimeException
*/
```

額外會在程式碼中添加跟蹤機制，用來驗證工廠物件建立的執行緒是否獲得新 **UncaughtExceptionHandler** 。現在未捕獲的異常由 **uncaughtException** 方法捕獲。

上面的範例根據具體情況來設定處理器。如果你知道你將要在程式碼中處處使用相同的異常處理器，那麼更簡單的方式是在 **Thread** 類中設定一個 **static**（靜態） 欄位，並將這個處理器設定為預設的未捕獲異常處理器：

```java
// lowlevel/SettingDefaultHandler.java
import java.util.concurrent.*;

public class SettingDefaultHandler {
  public static void main(String[] args) {
    Thread.setDefaultUncaughtExceptionHandler(
      new MyUncaughtExceptionHandler());
    ExecutorService es =
      Executors.newCachedThreadPool();
    es.execute(new ExceptionThread());
    es.shutdown();
  }
}
/* Output:
caught java.lang.RuntimeException
*/
```

只有在每個執行緒沒有設定異常處理器時候，預設處理器才會被呼叫。系統會檢查執行緒專有的版本，如果沒有，則檢查是否執行緒組中有專有的 `uncaughtException()` 方法；如果都沒有，就會呼叫 **defaultUncaughtExceptionHandler** 方法。

可以將此方法與 **CompletableFuture** 的改進方法進行比較。

<!-- Sharing Resources -->
## 資源共享

你可以將單執行緒程式看作一個孤獨的實體，在你的問題空間中移動並同一時間只做一件事。因為只有一個實體，你永遠不會想到兩個實體試圖同時使用相同資源的問題：問題猶如兩個人試圖同時停放在同一個空間，同時走過一扇門，甚至同時說話。

透過並發，事情不再孤單，但現在兩個或更多任務可能會相互干擾。如果你不阻止這種衝突，你將有兩個任務同時嘗試訪問同一個銀行帳戶，列印到同一個印表機，調整同一個閥門，等等。

### 資源競爭

當你啟動一個任務來執行某些工作時，可以透過兩種不同的方式捕獲該工作的結果:透過副作用或透過返回值。

從編程方式上看，副作用似乎更容易:你只需使用結果來操作環境中的某些東西。例如，你的任務可能會執行一些計算，然後直接將其結果寫入集合。

伴隨這種方式的問題是集合通常是共享資源。當執行多個任務時，任何任務都可能同時讀寫 *共享資源* 。這揭示了 *資源競爭* 問題，這是處理任務時的主要陷阱之一。

在單執行緒系統中，你不需要考慮資源競爭，因為你永遠不可能同時做多件事。當你有多個任務時，你就必須始終防止資源競爭。

解決此問題的的一種方法是使用能夠應對資源競爭的集合，如果多個任務同時嘗試對此類集合進行寫入，那麼此類集合可以應付該問題。在 Java 並發庫中，你將發現許多嘗試解決資源競爭問題的類；在本附錄中，你將看到其中的一些，但覆蓋範圍並不全面。

請思考以下的範例，其中一個任務負責生成偶數，其他任務則負責消費這些數字。在這裡，消費者任務的唯一工作就是檢查偶數的有效性。

我們將定義消費者任務 **EvenChecker** 類，以便在後續範例中可復用。為了將 **EvenChecker** 與我們的各種實驗生成器類解耦，我們首先建立名為 **IntGenerator** 的抽象類，它包含 **EvenChecker** 必須知道的最低必要方法：它包含 `next()` 方法，以及可以取消它執行生成的方法。

```java
// lowlevel/IntGenerator.java
import java.util.concurrent.atomic.AtomicBoolean;

public abstract class IntGenerator {
  private AtomicBoolean canceled =
    new AtomicBoolean();
  public abstract int next();
  public void cancel() { canceled.set(true); }
  public boolean isCanceled() {
    return canceled.get();
  }
}
```

`cancel()` 方法改變 **AtomicBoolean** 類型的 **canceled** 標誌位的狀態， 而 `isCanceled()` 方法則告訴標誌位是否設定。因為 **canceled** 標誌位是 **AtomicBoolean** 類型，由於它是原子性的，這意味著分配和值返回等簡單操作發生時沒有中斷的可能性，因此你無法在這些簡單操作中看到該欄位處於中間狀態。你將在本附錄的後面部分了解有關原子性和 **Atomic** 類的更多訊息

任何 **IntGenerator** 都可以使用下面的 **EvenChecker** 類進行測試:

```java
// lowlevel/EvenChecker.java
import java.util.*;
import java.util.stream.*;
import java.util.concurrent.*;
import onjava.TimedAbort;

public class EvenChecker implements Runnable {
  private IntGenerator generator;
  private final int id;
  public EvenChecker(IntGenerator generator, int id) {
    this.generator = generator;
    this.id = id;
  }
  @Override
  public void run() {
    while(!generator.isCanceled()) {
      int val = generator.next();
      if(val % 2 != 0) {
        System.out.println(val + " not even!");
        generator.cancel(); // Cancels all EvenCheckers
      }
    }
  }
  // Test any IntGenerator:
  public static void test(IntGenerator gp, int count) {
    List<CompletableFuture<Void>> checkers =
      IntStream.range(0, count)
        .mapToObj(i -> new EvenChecker(gp, i))
        .map(CompletableFuture::runAsync)
        .collect(Collectors.toList());
    checkers.forEach(CompletableFuture::join);
  }
  // Default value for count:
  public static void test(IntGenerator gp) {
    new TimedAbort(4, "No odd numbers discovered");
    test(gp, 10);
  }
}
```

`test()` 方法開啟了許多訪問同一個 **IntGenerator** 的 **EvenChecker**。**EvenChecker** 任務們會不斷讀取和測試與其關聯的 **IntGenerator** 物件中的生成值。如果 **IntGenerator** 導致失敗，`test()` 方法會報告並返回。

依賴於 **IntGenerator** 物件的所有 **EvenChecker** 任務都會檢查它是否已被取消。如果 `generator.isCanceled()` 返回值為 true ，則 `run()` 方法返回。 任何 **EvenChecker** 任務都可以在 **IntGenerator** 上呼叫 `cancel()` ，這會導致使用該 **IntGenerator** 的其他所有 **EvenChecker** 正常關閉。

在本設計中，共享公共資源（ **IntGenerator** ）的任務會監視該資源的終止訊號。這消除所謂的競爭條件，其中兩個或更多的任務競爭響應某個條件並因此衝突或不一致結果的情況。

你必須仔細考慮並防止並發系統失敗的所有可能途徑。例如，一個任務不能依賴於另一個任務，因為任務關閉的順序無法得到保證。這裡，透過使任務依賴於非任務物件，我們可以消除潛在的競爭條件。

一般來說，我們假設 `test()` 方法最終失敗，因為各個 **EvenChecker** 的任務在 **IntGenerator** 處於 “不恰當的” 狀態時，仍能夠訪問其中的訊息。但是，直到 **IntGenerator** 完成許多循環之前，它可能無法檢測到問題，具體取決於作業系統的詳細訊息和其他實現細節。為確保本書的自動構建不會卡住，我們使用 **TimedAbort** 類，在此處定義：

```java
// onjava/TimedAbort.java
// Terminate a program after t seconds
package onjava;
import java.util.concurrent.*;

public class TimedAbort {
  private volatile boolean restart = true;
  public TimedAbort(double t, String msg) {
    CompletableFuture.runAsync(() -> {
      try {
        while(restart) {
          restart = false;
          TimeUnit.MILLISECONDS
            .sleep((int)(1000 * t));
        }
      } catch(InterruptedException e) {
        throw new RuntimeException(e);
      }
      System.out.println(msg);
      System.exit(0);
    });
  }
  public TimedAbort(double t) {
    this(t, "TimedAbort " + t);
  }
  public void restart() { restart = true; }
}
```

我們使用 lambda 表達式建立一個 **Runnable** ，該表達式使用 **CompletableFuture** 的 `runAsync()` 靜態方法執行。  `runAsync()` 方法的值會立即返回。 因此，**TimedAbort** 不會保持任何打開的任務，否則已完成任務，但如果它需要太長時間，它仍將終止該任務（ **TimedAbort** 有時被稱為守護行程）。

**TimedAbort** 還允許你 `restart()` 方法重啟任務，在有某些有用的活動進行時保持程式打開。

我們可以看到正在執行的 **TimedAbort** 範例:

```java
// lowlevel/TestAbort.java
import onjava.*;

public class TestAbort {
  public static void main(String[] args) {
    new TimedAbort(1);
    System.out.println("Napping for 4");
    new Nap(4);
  }
}
/* Output:
Napping for 4
TimedAbort 1.0
*/
```

如果你注釋掉 **Nap** 建立實列那行，程式執行會立即退出，表明 **TimedAbort** 沒有維持程式打開。

我們將看到第一個 **IntGenerator** 範例有一個生成一系列偶數值的 `next()` 方法：

```java
// lowlevel/EvenProducer.java
// When threads collide
// {VisuallyInspectOutput}

public class EvenProducer extends IntGenerator {
  private int currentEvenValue = 0;
  @Override
  public int next() {
    ++currentEvenValue; // [1]
    ++currentEvenValue;
    return currentEvenValue;
  }
  public static void main(String[] args) {
    EvenChecker.test(new EvenProducer());
  }
}
/* Output:
419 not even!
425 not even!
423 not even!
421 not even!
417 not even!
*/
```
* [1] 一個任務有可能在另外一個任務執行第一個對 **currentEvenValue** 的自增操作之後，但是沒有執行第二個操作之前，呼叫 `next()` 方法。這將使這個值處於 “不恰當” 的狀態。

為了證明這是可能發生的， `EvenChecker.test()` 建立了一組 **EventChecker** 物件，以連續讀取 **EvenProducer** 的輸出並測試檢查每個數值是否都是偶數。如果不是，就會報告錯誤，而程式也將關閉。

多執行緒程式的部分問題是，即使存在 bug ，如果失敗的可能性很低，程式仍然可以正確顯示。

重要的是要注意到自增操作自身需要多個步驟，並且在自增過程中任務可能會被執行緒機制掛斷 - 也就是說，在 Java 中，自增不是原子性的操作。因此，如果不保護任務，即使單純的自增也不是執行緒安全的。

該範例程式並不總是在第一次非偶數產生時終止。所有任務都不會立即關閉，這是並發程式的典型特徵。

### 解決資源競爭

前面的範例揭示了當你使用執行緒時的基本問題：你永遠不知道執行緒哪個時刻執行。想像一下坐在一張桌子上，用叉子，將最後一塊食物放在盤子上，當叉子到達時，食物突然消失...僅因為你的執行緒被掛斷而另一個用餐者進來吃了食物了。這就是在編寫並發程式時要處理的問題。為了使並發工作有效，你需要某種方式來阻止兩個任務訪問同一個資源，至少在關鍵時期是這樣。

防止這種衝突的方法就是當資源被一個任務使用時，在其上加鎖。第一個訪問某項資源的任務必須鎖定這項資源，使其他任務在其被解鎖之前，就無法訪問它，而在其被解鎖時候，另一個任務就可以鎖定並使用它，以此類推。如果汽車前排座位是受限資源，那麼大喊著 “衝呀” 的孩子就會（在這次旅途過程中）獲得該資源的鎖。

為了解決執行緒衝突的問題，基本的並發方案將序列化訪問共享資源。這意味著一次只允許一個任務訪問共享資源。這通常是透過在訪問資源的程式碼片段周圍加上一個子句來實現的，該子句一次只允許一個任務訪問這段程式碼。因為這個子句產生 *互斥* 效果，所以這種機制的通常稱為是 *mutex* （互斥量）。

考慮一下屋子裡的浴室：多個人（即多個由執行緒驅動的任務）都希望能獨立使用浴室（即共享資源）。為了使用浴室，一個人先敲門來看看是否可用。如果沒人的話，他就能進入浴室並鎖上門。任何其他想使用浴室的任務就會被 “阻擋”，因此這些任務就在門口等待，直到浴室是可用的。

當浴室使用完畢，就是時候給其他任務進入，這時比喻就有點不準確了。事實上沒有人排隊，我們也不知道下一個使用浴室是誰，因為執行緒調度機制並不是確定性的。相反，就好像在浴室前面有一組被阻止的任務一樣，當鎖定浴室的任務解鎖並出現時，執行緒調度機制將會決定下一個要進入的任務。

Java 以提供關鍵字 **synchronized** 的形式，為防止資源衝突提供了內建支援。當任務希望執行被 **synchronized** 關鍵字保護的程式碼片段的時候，Java 編譯器會生成程式碼以查看鎖是否可用。如果可用，該任務獲取鎖，執行程式碼，然後釋放鎖。

共享資源一般是以物件形式存在的記憶體片段，但也可以是文件、I/O 埠，或者類似印表機的東西。要控制對共享資源的訪問，得先把它包裝進一個物件。然後把任何訪問該資源的方法標記為 **synchronized** 。 如果一個任務在呼叫其中一個 **synchronized** 方法之內，那麼在這個任務從該方法返回之前，其他所有要呼叫該物件的 **synchronized** 方法的任務都會被阻塞。

通常你會將欄位設為 **private**，並僅透過方法訪問這些欄位。你可用透過使用 **synchronized** 關鍵字聲明方法來防止資源衝突。如下所示：

```java
synchronized void f() { /* ... */ }
synchronized void g() { /* ... */ }
```

所有物件都自動包含獨立的鎖（也稱為 *monitor*，即監視器）。當你呼叫物件上任何 **synchronized** 方法，此物件將被加鎖，並且該物件上的的其他 **synchronized** 方法呼叫只有等到前一個方法執行完成並釋放了鎖之後才能被呼叫。如果一個任務對物件呼叫了 `f()` ，對於同一個物件而言，就只能等到 `f()` 呼叫結束並釋放了鎖之後，其他任務才能呼叫 `f()` 和 `g()`。所以，某個特定物件的所有 **synchronized** 方法共享同一個鎖，這個鎖可以防止多個任務同時寫入物件記憶體。

在使用並發時，將欄位設為 **private** 特別重要；否則，**synchronized** 關鍵字不能阻止其他任務直接訪問欄位，從而產生資源衝突。

一個執行緒可以獲取物件的鎖多次。如果一個方法呼叫在同一個物件上的第二個方法，而後者又在同一個物件上呼叫另一個方法，就會發生這種情況。 JVM 會跟蹤物件被鎖定的次數。如果物件已解鎖，則其計數為 0 。當一個執行緒首次獲得鎖時，計數變為 1 。每次同一執行緒在同一物件上獲取另一個鎖時，計數就會自增。顯然，只有首先獲得鎖的執行緒才允許多次獲取多個鎖。每當執行緒離開 **synchronized** 方法時，計數遞減，直到計數變為 0 ，完全釋放鎖以給其他執行緒使用。每個類也有一個鎖（作為該類的 **Class** 物件的一部分），因此 **synchronized** 靜態方法可以在類範圍的基礎上彼此鎖定，不讓同時訪問靜態資料。

你應該什麼時候使用同步呢？可以永遠 *Brian* 的同步法則[^2]。

> 如果你正在寫一個變數，它可能接下來被另一個執行緒讀取，或者正在讀取一個上一次已經被另一個執行緒寫過的變數，那麼你必須使用同步，並且，讀寫執行緒都必須用相同的監視器鎖同步。

如果在你的類中有超過一個方法在處理臨界資料，那麼你必須同步所有相關方法。如果只同步其中一個方法，那麼其他方法可以忽略物件鎖，並且可以不受懲罰地呼叫。這是很重要的一點：每個訪問臨界共享資源的方法都必須被同步，否則將不會正確地工作。

### 同步控制 EventProducer

透過在 **EvenProducer.java** 文件中添加 **synchronized** 關鍵字，可以防止不希望的執行緒訪問：

```java
// lowlevel/SynchronizedEvenProducer.java
// Simplifying mutexes with the synchronized keyword
import onjava.Nap;

public class
SynchronizedEvenProducer extends IntGenerator {
  private int currentEvenValue = 0;
  @Override
  public synchronized int next() {
    ++currentEvenValue;
    new Nap(0.01); // Cause failure faster
    ++currentEvenValue;
    return currentEvenValue;
  }
  public static void main(String[] args) {
    EvenChecker.test(new SynchronizedEvenProducer());
  }
}
/* Output:
No odd numbers discovered
*/
```

在兩個自增操作之間插入 `Nap()` 構造器方法，以提高在 **currentEvenValue** 是奇數的狀態時上下文切換的可能性。因為互斥鎖可以阻止多個任務同時進入臨界區，所有這不會產生失敗。第一個進入 `next()` 方法的任務將獲得鎖，任何試圖獲取鎖的後續任務都將被阻塞，直到第一個任務釋放鎖。此時，調度機制選擇另一個等待鎖的任務。透過這種方式，任何時刻只能有一個任務透過互斥鎖保護的程式碼。

<!-- The volatile Keyword -->
## volatile 關鍵字

**volatile** 可能是 Java 中最微妙和最難用的關鍵字。幸運的是，在現代 Java 中，你幾乎總能避免使用它，如果你確實看到它在程式碼中使用，你應該保持懷疑態度和懷疑 - 這很有可能程式碼是過時的，或者編寫程式碼的人不清楚使用它在大體上（或兩者都有）易變性（**volatile**） 或並發性的後果。

使用 **volatile** 有三個理由。

### 字分裂

當你的 Java 資料類型足夠大（在 Java 中 **long** 和 **double** 類型都是 64 位），寫入變數的過程分兩步進行，就會發生 *Word tearing* （字分裂）情況。 JVM 被允許將64位數量的讀寫作為兩個單獨的32位操作執行[^3]，這增加了在讀寫過程中發生上下文切換的可能性，因此其他任務會看到不正確的結果。這被稱為 *Word tearing* （字分裂），因為你可能只看到其中一部分修改後的值。基本上，任務有時可以在第一步之後但在第二步之前讀取變數，從而產生垃圾值（對於例如 **boolean** 或 **int** 類型的小變數是沒有問題的；任何 **long** 或 **double** 類型則除外）。

在缺乏任何其他保護的情況下，用 **volatile** 修飾符定義一個 **long** 或 **double** 變數，可阻止字分裂情況。然而，如果使用 **synchronized** 或 **java.util.concurrent.atomic** 類之一保護這些變數，則 **volatile** 將被取代。此外，**volatile** 不會影響到增量操作並不是原子操作的事實。

### 可見性

第二個問題屬於 [Java 並發的四句格言](./24-Concurrent-Programming.md#四句格言)裡第二句格言 “一切都重要” 的部分。你必須假設每個任務擁有自己的處理器，並且每個處理器都有自己的本機記憶體快取。該快取准許處理器執行的更快，因為處理器並不總是需要從比起使用快取顯著花費更多時間的主記憶體中獲取資料。

出現這個問題是因為 Java 嘗試儘可能地提高執行效率。快取的主要目的是避免從主記憶體中讀取資料。當並發時，有時不清楚 Java 什麼時候應該將值從主記憶體重新整理到本機快取 — 而這個問題稱為 *快取一致性* （ *cache coherence* ）。

每個執行緒都可以在處理器快取中儲存變數的本機副本。將欄位定義為 **volatile** 可以防止這些編譯器最佳化，這樣讀寫就可以直接進入記憶體，而不會被快取。一旦該欄位發生寫操作，所有任務的讀操作都將看到更改。如果一個 **volatile** 欄位剛好儲存在本機快取，則會立即將其寫入主記憶體，並且該欄位的任何讀取都始終發生在主記憶體中。

**volatile** 應該在何時適用於變數：

1. 該變數同時被多個任務訪問。
2. 這些訪問中至少有一個是寫操作。
3. 你嘗試避免同步 （在現代 Java 中，你可以使用進階工具來避免進行同步）。

舉個例子，如果你使用變數作為停止任務的標誌值。那麼該變數至少必須聲明為 **volatile** （儘管這並不一定能保證這種標誌的執行緒安全）。否則，當一個任務更改標誌值時，這些更改可以儲存在本機處理器快取中，而不會重新整理到主記憶體。當另一個任務查看標記值時，它不會看到更改。我更喜歡在 [並發編程](./24-Concurrent-Programming.md) 中 [終止耗時任務](./24-Concurrent-Programming.md#終止耗時任務) 章節中使用 **AtomicBoolean** 類型作為標誌值的辦法

任務對其自身變數所做的任何寫操作都始終對該任務可見，因此，如果只在任務中使用變數，你不需要使其變數聲明為 **volatile** 。

如果單個執行緒對變數寫入而其他執行緒只讀取它，你可以放棄該變數聲明為 **volatile**。通常，如果你有多個執行緒對變數寫入，**volatile** 無法解決你的問題，並且你必須使用 **synchronized** 來防止競爭條件。 這有一個特殊的例外：可以讓多個執行緒對該變數寫入，*只要它們不需要先讀取它並使用該值建立新值來寫入變數* 。如果這些多個執行緒在結果中使用舊值，則會出現競爭條件，因為其餘一個執行緒之一可能會在你的執行緒進行計算時修改該變數。即使你開始做對了，想像一下在程式碼修改或維護過程中忘記和引入一個重大變化是多麼容易，或者對於不理解問題的不同程式設計師來說是多麼容易（這在 Java 中尤其成問題因為程式設計師傾向於嚴重依賴編譯時檢查來告訴他們，他們的程式碼是否正確）。

重要的是要理解原子性和可見性是兩個不同的概念。在非 **volatile** 變數上的原子操作是不能保證是否將其重新整理到主記憶體。

同步也會讓主記憶體重新整理，所以如果一個變數完全由 **synchronized** 的方法或程式碼段(或者 **java.util.concurrent.atomic** 庫裡類型之一)所保護，則不需要讓變數用 **volatile**。

### 重排與 *Happen-Before* 原則

只要結果不會改變程式表現，Java 可以透過重排指令來最佳化性能。然而，重排可能會影響本機處理器快取與主記憶體互動的方式，從而產生細微的程式 bug 。直到 Java 5 才理解並解決了這個無法阻止重排的問題。現在，**volatile** 關鍵字可以阻止重排 **volatile** 變數周圍的讀寫指令。這種重排規則稱為 *happens before* 擔保原則 。

這項原則保證在 **volatile** 變數讀寫之前發生的指令先於它們的讀寫之前發生。同樣，任何跟隨 **volatile** 變數之後讀寫的操作都保證發生在它們的讀寫之後。例如：

```java
// lowlevel/ReOrdering.java

public class ReOrdering implements Runnable {
  int one, two, three, four, five, six;
  volatile int volaTile;
  @Override
  public void run() {
    one = 1;
    two = 2;
    three = 3;
    volaTile = 92;
    int x = four;
    int y = five;
    int z = six;
  }
}
```

例子中 **one**，**two**，**three** 變數賦值操作就可以被重排，只要它們都發生在 **volatile** 變數寫操作之前。同樣，只要 **volatile** 變數寫操作發生在所有語句之前， **x**，**y**，**z** 語句可以被重排。這種 **volatile** （易變性）操作通常稱為 *memory barrier* （記憶體屏障）。 *happens before* 擔保原則確保 **volatile** 變數的讀寫指令不能跨過記憶體屏障進行重排。

*happens before* 擔保原則還有另一個作用：當執行緒向一個 **volatile** 變數寫入時，在執行緒寫入之前的其他所有變數（包括非 **volatile** 變數）也會重新整理到主記憶體。當執行緒讀取一個 **volatile** 變數時，它也會讀取其他所有變數（包括非 **volatile** 變數）與 **volatile** 變數一起重新整理到主記憶體。儘管這是一個重要的特性，它解決了 Java 5 版本之前出現的一些非常狡猾的 bug ，但是你不應該依賴這項特性來“自動”使周圍的變數變得易變性 （ **volatile** ）的 。如果你希望變數是易變性 （ **volatile** ）的，那麼維護程式碼的任何人都應該清楚這一點。

### 什麼時候使用 volatile

對於 Java 早期版本，編寫一個證明需要 **volatile** 的範例並不難。如果你進行搜尋，你可以找到這樣的例子，但是如果你在 Java 8 中嘗試這些例子，它們就不起作用了(我沒有找到任何一個)。我努力寫這樣一個例子，但沒什麼用。這可能原因是 JVM 或者硬體，或兩者都得到了改進。這種效果對現有的應該  **volatile** （易變性） 但不 **volatile** 的儲存的程式是有益的；對於此類程式，失誤發生的頻率要低得多，而且問題更難追蹤。

如果你嘗試使用 **volatile** ，你可能更應該嘗試讓一個變數執行緒安全而不是引起同步的成本。因為 **volatile** 使用起來非常微妙和棘手，所以我建議根本不要使用它;相反，請使用本附錄後面介紹的 **java.util.concurrent.atomic** 裡面類之一。它們以比同步低得多的成本提供了完全的執行緒安全性。

如果你正在嘗試除錯其他人的並發程式碼，請首先尋找使用 **volatile** 的程式碼並將其取代為**Atomic** 變數。除非你確定程式設計師對並發性有很高的理解，否則它們很可能會誤用 **volatile** 。

<!-- Atomicity -->
## 原子性

在 Java 執行緒的討論中，經常反覆提交但不正確的知識是：“原子操作不需要同步”。 一個 *原子操作* 是不能被執行緒調度機制中斷的操作；一旦操作開始，那麼它一定可以在可能發生的“上下文切換”之前（切換到其他執行緒執行）執行完畢。依賴於原子性是很棘手且很危險的，如果你是一個並發編程專家，或者你得到了來自這樣的專家的幫助，你才應該使用原子性來代替同步，如果你認為自己足夠聰明可以應付這種玩火似的情況，那麼請接受下面的測試：

> Goetz 測試：如果你可以編寫用於現代微處理器的高性能 JVM ，那麼就有資格考慮是否可以避免同步[^4] 。

了解原子性是很有用的，並且知道它與其他進階技術一起用於實現一些更加巧妙的  **java.util.concurrent** 庫元件。 但是要堅決抵制自己依賴它的衝動。

原子性可以應用於除 **long** 和 **double** 之外的所有基本類型之上的 “簡單操作”。對於讀寫和寫入除 **long** 和 **double** 之外的基本類型變數這樣的操作，可以保證它們作為不可分 (原子) 的操作執行。


因為原子操作不能被執行緒機制中斷。專家程式設計師可以利用這個來編寫無鎖程式碼（*lock-free code*），這些程式碼不需要被同步。但即使這樣也過於簡單化了。有時候，甚至看起來應該是安全的原子操作，實際上也可能不安全。本書的讀者通常不會通過前面提到的 Goetz 測試，因此也就不具備用原子操作來取代同步的能力。嘗試著移除同步通常是一種表示不成熟最佳化的訊號，並且會給你帶來大量的麻煩，可能不會獲得太多或任何的好處。

在多核處理器系統，相對於單核處理器而言，可見性問題遠比原子性問題多得多。一個任務所做的修改，即使它們是原子性的，也可能對其他任務不可見（例如，修改只是暫時性儲存在本機處理器快取中），因此不同的任務對應用的狀態有不同的檢視。另一方面，同步機制強制多核處理器系統上的一個任務做出的修改必須在應用程式中是可見的。如果沒有同步機制，那麼修改時可見性將無法確認。

什麼才屬於原子操作時？對於屬性中的值做賦值和返回操作通常都是原子性的，但是在 C++ 中，甚至下面的操作都可能是原子性的：

```c++
i++; // Might be atomic in C++
i += 2; // Might be atomic in C++
```

但是在 C++ 中，這取決於編譯器和處理器。你無法編寫出依賴於原子性的 C++ 跨平台程式碼，因為 C++ [^5]沒有像 Java 那樣的一致 *記憶體模型* （memory model）。

在 Java 中，上面的操作肯定不是原子性的，正如下面的方法產生的 JVM 指令中可以看到的那樣：

```java
// lowlevel/NotAtomic.java
// {javap -c NotAtomic}
// {VisuallyInspectOutput}

public class NotAtomic {
  int i;
  void f1() { i++; }
  void f2() { i += 3; }
}
/* Output:
Compiled from "NotAtomic.java"
public class NotAtomic {
  int i;

  public NotAtomic();
    Code:
       0: aload_0
       1: invokespecial #1 // Method
java/lang/Object."<init>":()V
       4: return

  void f1();
    Code:
       0: aload_0
       1: dup
       2: getfield      #2 // Field
i:I
       5: iconst_1
       6: iadd
       7: putfield      #2 // Field
i:I
      10: return

  void f2();
    Code:
       0: aload_0
       1: dup
       2: getfield      #2 // Field
i:I
       5: iconst_3
       6: iadd
       7: putfield      #2 // Field
i:I
      10: return
}
*/
```

每條指令都會產生一個 “get” 和 “put”，它們之間還有一些其他指令。因此在獲取指令和放置指令之間，另有一個任務可能會修改這個屬性，所有，這些操作不是原子性的。

讓我們透過定義一個抽象類來測試原子性的概念，這個抽象類的方法是將一個整數類型進行偶數自增，並且 `run()` 不斷地呼叫這個方法:

```java
// lowlevel/IntTestable.java
import java.util.function.*;

public abstract class
IntTestable implements Runnable, IntSupplier {
  abstract void evenIncrement();
  @Override
  public void run() {
    while(true)
      evenIncrement();
  }
}
```

**IntSupplier** 是一個帶 `getAsInt()` 方法的函數式介面。

現在我們可以建立一個測試，它作為一個獨立的任務啟動 `run()` 方法 ，然後獲取值來檢查它們是否為偶數:

```java
// lowlevel/Atomicity.java
import java.util.concurrent.*;
import onjava.TimedAbort;

public class Atomicity {
  public static void test(IntTestable it) {
    new TimedAbort(4, "No failures found");
    CompletableFuture.runAsync(it);
    while(true) {
      int val = it.getAsInt();
      if(val % 2 != 0) {
        System.out.println("failed with: " + val);
        System.exit(0);
      }
    }
  }
}
```

很容易盲目地應用原子性的概念。在這裡，`getAsInt()` 似乎是安全的原子性方法：

```java
// lowlevel/UnsafeReturn.java
import java.util.function.*;
import java.util.concurrent.*;

public class UnsafeReturn extends IntTestable {
  private int i = 0;
  public int getAsInt() { return i; }
  public synchronized void evenIncrement() {
    i++; i++;
  }
  public static void main(String[] args) {
    Atomicity.test(new UnsafeReturn());
  }
}
/* Output:
failed with: 79
*/
```

但是， `Atomicity.test()` 方法還是出現有非偶數的失敗。儘管，返回 **i** 變數確實是原子操作，但是同步缺失允許了在物件處於不穩定的中間狀態時讀取值。最重要的是，由於 **i** 也不是 **volatile** 變數，所以存在可見性問題。包括 `getValue()` 和 `evenIncrement()` 都必須同步(這也顧及到沒有使用 **volatile** 修飾的 **i** 變數):

```java
// lowlevel/SafeReturn.java
import java.util.function.*;
import java.util.concurrent.*;

public class SafeReturn extends IntTestable {
  private int i = 0;
  public synchronized int getAsInt() { return i; }
  public synchronized void evenIncrement() {
    i++; i++;
  }
  public static void main(String[] args) {
    Atomicity.test(new SafeReturn());
  }
}
/* Output:
No failures found
*/
```

只有並發編程專家有能力去嘗試做像前面例子情況的最佳化；再次強調，請遵循 Brain 的同步法則。

### Josh 的序號

作為第二個範例，考慮某些更簡單的東西：建立一個產生序號的類，靈感啟發於 Joshua Bloch 的 *Effective Java Programming Language Guide* (Addison-Wesley 出版社, 2001) 第 190 頁。每次呼叫 `nextSerialNumber()` 都必須返回唯一值。

```java
// lowlevel/SerialNumbers.java

public class SerialNumbers {
  private volatile int serialNumber = 0;
  public int nextSerialNumber() {
    return serialNumber++; // Not thread-safe
  }
}
```

**SerialNumbers**  是你可以想像到最簡單的類，如果你具備 C++ 或者其他底層的知識背景，你可能會認為自增是一個原子操作，因為 C++ 的自增操作通常被單個微處理器指令所實現（儘管不是以任何一致，可靠，跨平台的方式）。但是，正如前面所提到的，Java 自增操作不是原子性的，並且操作同時涉及讀取和寫入，因此即使在這樣一個簡單的操作中，也存在有執行緒問題的空間。

我們在這裡加入 volatile ，看看它是否有幫助。然而，真正的問題是 `nextSerialNumber()` 方法在不進行執行緒同步的情況下訪問共享的可變變數值。

為了測試 **SerialNumbers**，我們將建立一個不會耗盡記憶體的集合，假如需要很長時間來檢測問題。這裡展示的 **CircularSet** 重用了儲存 **int** 變數的記憶體，最終新值會覆蓋舊值(複製的速度通常發生足夠快，你也可以使用  **java.util.Set** 來代替):

```java
// lowlevel/CircularSet.java
// Reuses storage so we don't run out of memory
import java.util.*;

public class CircularSet {
  private int[] array;
  private int size;
  private int index = 0;
  public CircularSet(int size) {
    this.size = size;
    array = new int[size];
    // Initialize to a value not produced
    // by SerialNumbers:
    Arrays.fill(array, -1);
  }
  public synchronized void add(int i) {
    array[index] = i;
    // Wrap index and write over old elements:
    index = ++index % size;
  }
  public synchronized boolean contains(int val) {
    for(int i = 0; i < size; i++)
      if(array[i] == val) return true;
    return false;
  }
}
```

`add()` 和 `contains()` 方法是執行緒同步的，以防止執行緒衝突。
The add() and contains() methods are synchronized to prevent thread collisions.

**SerialNumberChecker** 類包含一個儲存最近序號的 **CircularSet** 變數，以及一個填充數值給 **CircularSet**  和確保它裡面的序號是唯一的 `run()` 方法。

```java
// lowlevel/SerialNumberChecker.java
// Test SerialNumbers implementations for thread-safety
import java.util.concurrent.*;
import onjava.Nap;

public class SerialNumberChecker implements Runnable {
  private CircularSet serials = new CircularSet(1000);
  private SerialNumbers producer;
  public SerialNumberChecker(SerialNumbers producer) {
    this.producer = producer;
  }
  @Override
  public void run() {
    while(true) {
      int serial = producer.nextSerialNumber();
      if(serials.contains(serial)) {
        System.out.println("Duplicate: " + serial);
        System.exit(0);
      }
      serials.add(serial);
    }
  }
  static void test(SerialNumbers producer) {
    for(int i = 0; i < 10; i++)
      CompletableFuture.runAsync(
        new SerialNumberChecker(producer));
    new Nap(4, "No duplicates detected");
  }
}
```

`test()` 方法建立多個任務來競爭單獨的 **SerialNumbers** 物件。這時參於競爭的的 SerialNumberChecker 任務們就會試圖生成重複的序號（這情況在具有更多核心處理器的機器上發生得更快）。

當我們測試基本的 **SerialNumbers** 類，它會失敗（產生重複序號）：

```java
// lowlevel/SerialNumberTest.java

public class SerialNumberTest {
  public static void main(String[] args) {
    SerialNumberChecker.test(new SerialNumbers());
  }
}
/* Output:
Duplicate: 148044
*/
```

**volatile** 在這裡沒有幫助。要解決這個問題，將 **synchronized** 關鍵字添加到 `nextSerialNumber()` 方法 :

```java
// lowlevel/SynchronizedSerialNumbers.java

public class
SynchronizedSerialNumbers extends SerialNumbers {
  private int serialNumber = 0;
  public synchronized int nextSerialNumber() {
    return serialNumber++;
  }
  public static void main(String[] args) {
    SerialNumberChecker.test(
      new SynchronizedSerialNumbers());
  }
}
/* Output:
No duplicates detected
*/
```

**volatile** 不再是必需的，因為 **synchronized** 關鍵字保證了 volatile （易變性） 的特性。

讀取和賦值原語應該是安全的原子操作。然後，正如在 **UnsafeReturn.java** 中所看到，使用原子操作訪問處於不穩定中間狀態的物件仍然很容易。對這個問題做出假設既棘手又危險。最明智的做法就是遵循 Brian 的同步規則(如果可以，首先不要共享變數)。

### 原子類

Java 5 引入了專用的原子變數類，例如 **AtomicInteger**、**AtomicLong**、**AtomicReference** 等。這些提供了原子性升級。這些快速、無鎖的操作，它們是利用了現代處理器上可用的機器級原子性。

下面，我們可以使用 **atomicinteger** 重寫 **unsafereturn.java** 範例：

```java
// lowlevel/AtomicIntegerTest.java
import java.util.concurrent.*;
import java.util.concurrent.atomic.*;
import java.util.*;
import onjava.*;

public class AtomicIntegerTest extends IntTestable {
  private AtomicInteger i = new AtomicInteger(0);
  public int getAsInt() { return i.get(); }
  public void evenIncrement() { i.addAndGet(2); }
  public static void main(String[] args) {
    Atomicity.test(new AtomicIntegerTest());
  }
}
/* Output:
No failures found
*/
```

現在，我們透過使用 **AtomicInteger** 來消除了 **synchronized** 關鍵字。

下面使用 **AtomicInteger** 來重寫 **SynchronizedEvenProducer.java** 範例：

```java
// lowlevel/AtomicEvenProducer.java
// Atomic classes: occasionally useful in regular code
import java.util.concurrent.atomic.*;

public class AtomicEvenProducer extends IntGenerator {
  private AtomicInteger currentEvenValue =
    new AtomicInteger(0);
  @Override
  public int next() {
    return currentEvenValue.addAndGet(2);
  }
  public static void main(String[] args) {
    EvenChecker.test(new AtomicEvenProducer());
  }
}
/* Output:
No odd numbers discovered
*/
```

再次，使用 **AtomicInteger** 消除了對所有其他同步方式的需要。

下面是一個使用 **AtomicInteger** 實現 **SerialNumbers** 的例子:

```java
// lowlevel/AtomicSerialNumbers.java
import java.util.concurrent.atomic.*;

public class
AtomicSerialNumbers extends SerialNumbers {
  private AtomicInteger serialNumber =
    new AtomicInteger();
  public int nextSerialNumber() {
    return serialNumber.getAndIncrement();
  }
  public static void main(String[] args) {
    SerialNumberChecker.test(
      new AtomicSerialNumbers());
  }
}
/* Output:
No duplicates detected
*/
```

這些都是對單一欄位的簡單範例； 當你建立更複雜的類時，你必須確定哪些欄位需要保護，在某些情況下，你可能仍然最後在方法上使用 **synchronized** 關鍵字。

<!-- Critical Sections -->
## 臨界區

有時，你只是想防止多執行緒訪問方法中的部分程式碼，而不是整個方法。要隔離的程式碼部分稱為臨界區，它使用我們用於保護整個方法相同的 **synchronized** 關鍵字建立，但使用不同的語法。語法如下， **synchronized** 指定某個物件作為鎖用於同步控制花括號內的程式碼：

```java
synchronized(syncObject) {
  // This code can be accessed
  // by only one task at a time
}
```

這也被稱為 *同步控制塊* （synchronized block）；在進入此段程式碼前，必須得到 **syncObject** 物件的鎖。如果一些其他任務已經得到這個鎖，那麼就得等到鎖被釋放以後，才能進入臨界區。當發生這種情況時，嘗試獲取該鎖的任務就會掛斷。執行緒調度會定期回來並檢查鎖是否已經釋放；如果釋放了鎖則喚醒任務。

使用同步控制塊而不是同步控制整個方法的主要動機是性能（有時，演算法確實聰明，但還是要特別警惕來自並發性問題上的聰明）。下面的範例示範了同步控制程式碼塊而不是整個方法可以使方法更容易被其他任務訪問。該範例會統計成功訪問 `method()` 的計數並且發起一些任務來嘗試競爭呼叫 `method()` 方法。

```java
// lowlevel/SynchronizedComparison.java
// speeds up access.
import java.util.*;
import java.util.stream.*;
import java.util.concurrent.*;
import java.util.concurrent.atomic.*;
import onjava.Nap;

abstract class Guarded {
  AtomicLong callCount = new AtomicLong();
  public abstract void method();
  @Override
  public String toString() {
    return getClass().getSimpleName() +
      ": " + callCount.get();
  }
}

class SynchronizedMethod extends Guarded {
  public synchronized void method() {
    new Nap(0.01);
    callCount.incrementAndGet();
  }
}

class CriticalSection extends Guarded {
  public void method() {
    new Nap(0.01);
    synchronized(this) {
      callCount.incrementAndGet();
    }
  }
}

class Caller implements Runnable {
  private Guarded g;
  Caller(Guarded g) { this.g = g; }
  private AtomicLong successfulCalls =
    new AtomicLong();
  private AtomicBoolean stop =
    new AtomicBoolean(false);
  @Override
  public void run() {
    new Timer().schedule(new TimerTask() {
      public void run() { stop.set(true); }
    }, 2500);
    while(!stop.get()) {
      g.method();
      successfulCalls.getAndIncrement();
    }
    System.out.println(
      "-> " + successfulCalls.get());
  }
}

public class SynchronizedComparison {
  static void test(Guarded g) {
    List<CompletableFuture<Void>> callers =
      Stream.of(
        new Caller(g),
        new Caller(g),
        new Caller(g),
        new Caller(g))
        .map(CompletableFuture::runAsync)
        .collect(Collectors.toList());
    callers.forEach(CompletableFuture::join);
    System.out.println(g);
  }
  public static void main(String[] args) {
    test(new CriticalSection());
    test(new SynchronizedMethod());
  }
}
/* Output:
-> 243
-> 243
-> 243
-> 243
CriticalSection: 972
-> 69
-> 61
-> 83
-> 36
SynchronizedMethod: 249
*/
```

**Guarded** 類負責跟蹤 **callCount** 中成功呼叫 `method()`  的次數。**SynchronizedMethod** 的方式是同步控制整個 `method` 方法，而 **CriticalSection** 的方式是使用同步控制塊來僅同步 `method` 方法的一部分程式碼。這樣，耗時的 **Nap** 物件可以被排除到同步控制塊外。輸出會顯示 **CriticalSection** 中可用的 `method()` 有多少。

請記住，使用同步控制塊是有風險；它要求你確切知道同步控制塊外的非同步程式碼是實際上要執行緒安全的。

**Caller** 是嘗試在給定的時間週期內儘可能多地呼叫 `method()` 方法（並報告呼叫次數）的任務。為了構建這個時間週期，我們會使用雖然有點過時但仍然可以很好地工作的 **java.util.Timer** 類。此類接收一個 **TimerTask** 參數, 但該參數並不是函數式介面，所以我們不能使用 **lambda** 表達式，必須顯式建立該類物件（在這種情況下，使用匿名內部類）。當超時的時候，定時物件將設定 **AtomicBoolean** 類型的 **stop** 欄位為 true ，這樣循環就會退出。

`test()` 方法接收一個 **Guarded** 類物件並建立四個 **Caller** 任務。所有這些任務都添加到同一個 **Guarded** 物件上，因此它們競爭來獲取使用 `method()` 方法的鎖。

你通常會看到從一次執行到下一次執行的輸出變化。結果表明， **CriticalSection** 方式比起 **SynchronizedMethod** 方式允許更多地訪問 `method()` 方法。這通常是使用 **synchronized** 塊取代同步控制整個方法的原因：允許其他任務更多訪問(只要這樣做是執行緒安全的)。

### 在其他物件上同步

**synchronized** 塊必須給定一個在其上進行同步的物件。並且最合理的方式是，使用其方法正在被呼叫的目前物件： **synchronized(this)**，這正是前面範例中 **CriticalSection** 採取的方式。在這種方式中，當 **synchronized** 塊獲得鎖的時候，那麼該物件其他的 **synchronized** 方法和臨界區就不能被呼叫了。因此，在進行同步時，臨界區的作用是減小同步的範圍。

有時必須在另一個物件上同步，但是如果你要這樣做，就必須確保所有相關的任務都是在同一個任務上同步的。下面的範例示範了當物件中的方法在不同的鎖上同步時，兩個任務可以同時進入同一物件：

```java
// lowlevel/SyncOnObject.java
// Synchronizing on another object
import java.util.*;
import java.util.stream.*;
import java.util.concurrent.*;
import onjava.Nap;

class DualSynch {
  ConcurrentLinkedQueue<String> trace =
    new ConcurrentLinkedQueue<>();
  public synchronized void f(boolean nap) {
    for(int i = 0; i < 5; i++) {
      trace.add(String.format("f() " + i));
      if(nap) new Nap(0.01);
    }
  }
  private Object syncObject = new Object();
  public void g(boolean nap) {
    synchronized(syncObject) {
      for(int i = 0; i < 5; i++) {
        trace.add(String.format("g() " + i));
        if(nap) new Nap(0.01);
      }
    }
  }
}

public class SyncOnObject {
  static void test(boolean fNap, boolean gNap) {
    DualSynch ds = new DualSynch();
    List<CompletableFuture<Void>> cfs =
      Arrays.stream(new Runnable[] {
        () -> ds.f(fNap), () -> ds.g(gNap) })
        .map(CompletableFuture::runAsync)
        .collect(Collectors.toList());
    cfs.forEach(CompletableFuture::join);
    ds.trace.forEach(System.out::println);
  }
  public static void main(String[] args) {
    test(true, false);
    System.out.println("****");
    test(false, true);
  }
}
/* Output:
f() 0
g() 0
g() 1
g() 2
g() 3
g() 4
f() 1
f() 2
f() 3
f() 4
****
f() 0
g() 0
f() 1
f() 2
f() 3
f() 4
g() 1
g() 2
g() 3
g() 4
*/
```

`DualSync.f()` 方法（透過同步整個方法）在 **this** 上同步，而 `g()` 方法有一個在 **syncObject** 上同步的 **synchronized** 塊。因此，這兩個同步是互相獨立的。在 `test()` 方法中執行的兩個呼叫 `f()` 和 `g()` 方法的獨立任務示範了這一點。**fNap** 和 **gNap** 標誌變數分別指示 `f()` 和 `g()` 是否應該在其 **for** 循環中呼叫 `Nap()` 方法。例如，當 f() 執行緒休眠時 ，該執行緒繼續持有它的鎖，但是你可以看到這並不阻止呼叫 `g()` ，反之亦然。

### 使用顯式鎖物件

**java.util.concurrent** 庫包含在 **java.util.concurrent.locks** 中定義的顯示互斥鎖機制。 必須顯式地建立，鎖定和解鎖 **Lock** 物件，因此它產出的程式碼沒有內建 **synchronized** 關鍵字那麼優雅。然而，它在解決某些類型的問題時更加靈活。下面是使用顯式 **Lock** 物件重寫 **SynchronizedEvenProducer.java** 程式碼：

```java
// lowlevel/MutexEvenProducer.java
// Preventing thread collisions with mutexes
import java.util.concurrent.locks.*;
import onjava.Nap;

public class MutexEvenProducer extends IntGenerator {
  private int currentEvenValue = 0;
  private Lock lock = new ReentrantLock();
  @Override
  public int next() {
    lock.lock();
    try {
      ++currentEvenValue;
      new Nap(0.01); // Cause failure faster
      ++currentEvenValue;
      return currentEvenValue;
    } finally {
      lock.unlock();
    }
  }
  public static void main(String[] args) {
    EvenChecker.test(new MutexEvenProducer());
  }
}
/*
No odd numbers discovered
*/
```
**MutexEvenProducer** 添加一個名為 **lock** 的互斥鎖並在 `next()` 中使用 `lock()` 和 `unlock()` 方法建立一個臨界區。當你使用 **Lock** 物件時，使用下面顯示的習慣用法很重要：在呼叫 `Lock()` 之後，你必須放置 **try-finally** 語句，該語句在 **finally** 子句中帶有 `unlock()` 方法 - 這是確保鎖總是被釋放的惟一方法。注意，**return** 語句必須出現在 **try** 子句中，以確保 **unlock()** 不會過早發生並將資料暴露給第二個任務。

儘管 **try-finally** 比起使用 **synchronized** 關鍵字需要用得更多程式碼，但它也代表了顯式鎖物件的優勢之一。如果使用 **synchronized** 關鍵字失敗，就會拋出異常，但是你沒有機會進行任何清理以保持系統處於良好狀態。而使用顯式鎖物件，可以使用 **finally** 子句在系統中維護適當的狀態。

一般來說，當你使用 **synchronized** 的時候，需要編寫的程式碼更少，並且使用者出錯的機會也大大減少，因此通常只在解決特殊問題時使用顯式鎖物件。例如，使用 **synchronized** 關鍵字，你不能嘗試獲得鎖並讓其失敗，或者你在一段時間內嘗試獲得鎖，然後放棄 - 為此，你必須使用這個並發庫。

```java
// lowlevel/AttemptLocking.java
// Locks in the concurrent library allow you
// to give up on trying to acquire a lock
import java.util.concurrent.*;
import java.util.concurrent.locks.*;
import onjava.Nap;

public class AttemptLocking {
  private ReentrantLock lock = new ReentrantLock();
  public void untimed() {
    boolean captured = lock.tryLock();
    try {
      System.out.println("tryLock(): " + captured);
    } finally {
      if(captured)
        lock.unlock();
    }
  }
  public void timed() {
    boolean captured = false;
    try {
      captured = lock.tryLock(2, TimeUnit.SECONDS);
    } catch(InterruptedException e) {
      throw new RuntimeException(e);
    }
    try {
      System.out.println(
        "tryLock(2, TimeUnit.SECONDS): " + captured);
    } finally {
      if(captured)
        lock.unlock();
    }
  }
  public static void main(String[] args) {
    final AttemptLocking al = new AttemptLocking();
    al.untimed(); // True -- lock is available
    al.timed();   // True -- lock is available
    // Now create a second task to grab the lock:
    CompletableFuture.runAsync( () -> {
        al.lock.lock();
        System.out.println("acquired");
    });
    new Nap(0.1);  // Give the second task a chance
    al.untimed(); // False -- lock grabbed by task
    al.timed();   // False -- lock grabbed by task
  }
}
/* Output:
tryLock(): true
tryLock(2, TimeUnit.SECONDS): true
acquired
tryLock(): false
tryLock(2, TimeUnit.SECONDS): false
*/
```

**ReentrantLock** 可以嘗試或者放棄獲取鎖，因此如果某些任務已經擁有鎖，你可以決定放棄並執行其他操作，而不是一直等到鎖釋放，就像 `untimed()` 方法那樣。而在 `timed()` 方法中，則嘗試獲取可能在 2 秒後沒成功而放棄的鎖。在 `main()` 方法中，一個單獨的執行緒被匿名類所建立，並且它會獲得鎖，因此讓 `untimed()` 和 `timed() ` 方法有東西可以去競爭。

顯式鎖比起內建同步鎖提供更細粒度的加鎖和解鎖控制。這對於實現專門的同步並髮結構，比如用於遍歷鍊表節點的 *交替鎖* ( *hand-over-hand locking* ) ，也稱為 *鎖耦合* （ *lock coupling* ）- 該遍歷程式碼要求必須在目前節點的解鎖之前捕獲下一個節點的鎖。

<!-- Library Components -->
## 庫元件

**java.util.concurrent** 庫提供大量旨在解決並發問題的類，可以幫助你生成更簡單，更魯棒的並發程式。但請注意，這些工具是比起並行流和 **CompletableFuture** 更底層的機制。

在本節中，我們將看一些使用不同元件的範例，然後討論一下 *lock-free*（無鎖） 庫元件是如何工作的。

### DelayQueue

這是一個無界阻塞佇列 （ **BlockingQueue** ），用於放置實現了 **Delayed** 介面的物件，其中的物件只能在其到期時才能從佇列中取走。這種佇列是有序的，因此隊首物件的延遲到期的時間最長。如果沒有任何延遲到期，那麼就不會有隊首元素，並且 `poll()` 將返回 **null**（正因為這樣，你不能將 **null** 放置到這種佇列中）。

下面是一個範例，其中的 **Delayed** 物件自身就是任務，而 **DelayedTaskConsumer** 將最“緊急”的任務（到期時間最長的任務）從佇列中取出，然後執行它。注意的是這樣 **DelayQueue** 就成為了優先度佇列的一種變體。

```java
// lowlevel/DelayQueueDemo.java
import java.util.*;
import java.util.stream.*;
import java.util.concurrent.*;
import static java.util.concurrent.TimeUnit.*;

class DelayedTask implements Runnable, Delayed {
  private static int counter = 0;
  private final int id = counter++;
  private final int delta;
  private final long trigger;
  protected static List<DelayedTask> sequence =
    new ArrayList<>();
  DelayedTask(int delayInMilliseconds) {
    delta = delayInMilliseconds;
    trigger = System.nanoTime() +
      NANOSECONDS.convert(delta, MILLISECONDS);
    sequence.add(this);
  }
  @Override
  public long getDelay(TimeUnit unit) {
    return unit.convert(
      trigger - System.nanoTime(), NANOSECONDS);
  }
  @Override
  public int compareTo(Delayed arg) {
    DelayedTask that = (DelayedTask)arg;
    if(trigger < that.trigger) return -1;
    if(trigger > that.trigger) return 1;
    return 0;
  }
  @Override
  public void run() {
    System.out.print(this + " ");
  }
  @Override
  public String toString() {
    return
      String.format("[%d] Task %d", delta, id);
  }
  public String summary() {
    return String.format("(%d:%d)", id, delta);
  }
  public static class EndTask extends DelayedTask {
    EndTask(int delay) { super(delay); }
    @Override
    public void run() {
      sequence.forEach(dt ->
        System.out.println(dt.summary()));
    }
  }
}

public class DelayQueueDemo {
  public static void
  main(String[] args) throws Exception {
    DelayQueue<DelayedTask> tasks =
      Stream.concat( // Random delays:
        new Random(47).ints(20, 0, 4000)
          .mapToObj(DelayedTask::new),
        // Add the summarizing task:
        Stream.of(new DelayedTask.EndTask(4000)))
      .collect(Collectors
        .toCollection(DelayQueue::new));
    while(tasks.size() > 0)
      tasks.take().run();
  }
}
/* Output:
[128] Task 12 [429] Task 6 [551] Task 13 [555] Task 2
[693] Task 3 [809] Task 15 [961] Task 5 [1258] Task 1
[1258] Task 20 [1520] Task 19 [1861] Task 4 [1998] Task
17 [2200] Task 8 [2207] Task 10 [2288] Task 11 [2522]
Task 9 [2589] Task 14 [2861] Task 18 [2868] Task 7
[3278] Task 16 (0:4000)
(1:1258)
(2:555)
(3:693)
(4:1861)
(5:961)
(6:429)
(7:2868)
(8:2200)
(9:2522)
(10:2207)
(11:2288)
(12:128)
(13:551)
(14:2589)
(15:809)
(16:3278)
(17:1998)
(18:2861)
(19:1520)
(20:1258)
*/
```

**DelayedTask** 包含一個稱為 **sequence** 的 **List&lt;DelayedTask&gt;** ，它儲存了任務被建立的順序，因此我們可以看到排序是按照實際發生的順序執行的。

**Delay** 介面有一個方法， `getDelay()` ， 該方法用來告知延遲到期有多長時間，或者延遲在多長時間之前已經到期了。這個方法強制我們去使用 **TimeUnit** 類，因為這就是參數類型。這會產生一個非常方便的類，因為你可以很容易地轉換單位而無需作任何聲明。例如，**delta** 的值是以毫秒為單位儲存的，但是 `System.nanoTime()` 產生的時間則是以奈秒為單位的。你可以轉換 **delta** 的值，方法是聲明它的單位以及你希望以什麼單位來表示，就像下面這樣：

```java
NANOSECONDS.convert(delta, MILLISECONDS);
```

在 `getDelay()` 中， 所希望的單位是作為 **unit** 參數傳遞進來的，你使用它將目前時間與觸發時間之間的差轉換為呼叫者要求的單位，而無需知道這些單位是什麼（這是*策略*設計模式的一個簡單範例，在這種模式中，演算法的一部分是作為參數傳遞進來的）。

為了排序， **Delayed** 介面還繼承了 **Comparable** 介面，因此必須實現 `compareTo()` , 使其可以產生合理的比較。

從輸出中可以看到，任務建立的順序對執行順序沒有任何影響 - 相反，任務是按照所期望的延遲順序所執行的。

### PriorityBlockingQueue

這是一個很基礎的優先度佇列，它具有可阻塞的讀取操作。在下面的範例中， **Prioritized** 物件會被賦予優先度編號。幾個 **Producer** 任務的實例會插入 **Prioritized** 物件到 **PriorityBlockingQueue** 中，但插入之間會有隨機延時。然後，單個 **Consumer** 任務在執行 `take()` 時會顯示多個選項，**PriorityBlockingQueue** 會將目前具有最高優先度的 **Prioritized** 物件提供給它。

在 **Prioritized** 中的靜態變數 **counter** 是 **AtomicInteger** 類型。這是必要的，因為有多個 **Producer** 並行執行；如果不是 **AtomicInteger** 類型，你將會看到重複的 **id** 號。 這個問題在 [並發編程](./24-Concurrent-Programming.md) 的 [建構子非執行緒安全](./24-Concurrent-Programming.md) 一節中討論過。

```java
// lowlevel/PriorityBlockingQueueDemo.java
import java.util.*;
import java.util.stream.*;
import java.util.concurrent.*;
import java.util.concurrent.atomic.*;
import onjava.Nap;

class Prioritized implements Comparable<Prioritized>  {
  private static AtomicInteger counter =
    new AtomicInteger();
  private final int id = counter.getAndIncrement();
  private final int priority;
  private static List<Prioritized> sequence =
    new CopyOnWriteArrayList<>();
  Prioritized(int priority) {
    this.priority = priority;
    sequence.add(this);
  }
  @Override
  public int compareTo(Prioritized arg) {
    return priority < arg.priority ? 1 :
      (priority > arg.priority ? -1 : 0);
  }
  @Override
  public String toString() {
    return String.format(
      "[%d] Prioritized %d", priority, id);
  }
  public void displaySequence() {
    int count = 0;
    for(Prioritized pt : sequence) {
      System.out.printf("(%d:%d)", pt.id, pt.priority);
      if(++count % 5 == 0)
        System.out.println();
    }
  }
  public static class EndSentinel extends Prioritized {
    EndSentinel() { super(-1); }
  }
}

class Producer implements Runnable {
  private static AtomicInteger seed =
    new AtomicInteger(47);
  private SplittableRandom rand =
    new SplittableRandom(seed.getAndAdd(10));
  private Queue<Prioritized> queue;
  Producer(Queue<Prioritized> q) {
    queue = q;
  }
  @Override
  public void run() {
    rand.ints(10, 0, 20)
      .mapToObj(Prioritized::new)
      .peek(p -> new Nap(rand.nextDouble() / 10))
      .forEach(p -> queue.add(p));
    queue.add(new Prioritized.EndSentinel());
  }
}

class Consumer implements Runnable {
  private PriorityBlockingQueue<Prioritized> q;
  private SplittableRandom rand =
    new SplittableRandom(47);
  Consumer(PriorityBlockingQueue<Prioritized> q) {
    this.q = q;
  }
  @Override
  public void run() {
    while(true) {
      try {
        Prioritized pt = q.take();
        System.out.println(pt);
        if(pt instanceof Prioritized.EndSentinel) {
          pt.displaySequence();
          break;
        }
        new Nap(rand.nextDouble() / 10);
      } catch(InterruptedException e) {
        throw new RuntimeException(e);
      }
    }
  }
}

public class PriorityBlockingQueueDemo {
  public static void main(String[] args) {
    PriorityBlockingQueue<Prioritized> queue =
      new PriorityBlockingQueue<>();
    CompletableFuture.runAsync(new Producer(queue));
    CompletableFuture.runAsync(new Producer(queue));
    CompletableFuture.runAsync(new Producer(queue));
    CompletableFuture.runAsync(new Consumer(queue))
      .join();
  }
}
/* Output:
[15] Prioritized 2
[17] Prioritized 1
[17] Prioritized 5
[16] Prioritized 6
[14] Prioritized 9
[12] Prioritized 0
[11] Prioritized 4
[11] Prioritized 12
[13] Prioritized 13
[12] Prioritized 16
[14] Prioritized 18
[15] Prioritized 23
[18] Prioritized 26
[16] Prioritized 29
[12] Prioritized 17
[11] Prioritized 30
[11] Prioritized 24
[10] Prioritized 15
[10] Prioritized 22
[8] Prioritized 25
[8] Prioritized 11
[8] Prioritized 10
[6] Prioritized 31
[3] Prioritized 7
[2] Prioritized 20
[1] Prioritized 3
[0] Prioritized 19
[0] Prioritized 8
[0] Prioritized 14
[0] Prioritized 21
[-1] Prioritized 28
(0:12)(2:15)(1:17)(3:1)(4:11)
(5:17)(6:16)(7:3)(8:0)(9:14)
(10:8)(11:8)(12:11)(13:13)(14:0)
(15:10)(16:12)(17:12)(18:14)(19:0)
(20:2)(21:0)(22:10)(23:15)(24:11)
(25:8)(26:18)(27:-1)(28:-1)(29:16)
(30:11)(31:6)(32:-1)
*/
```

與前面的範例一樣，**Prioritized** 物件的建立順序在 **sequence** 的 **list** 物件上所記入，以便與實際執行順序進行比較。 **EndSentinel** 是用於告知 **Consumer** 物件關閉的特殊類型。

**Producer** 使用 **AtomicInteger** 變數為 **SplittableRandom** 設定隨機生成種子，以便不同的 **Producer** 生成不同的佇列。 這是必需的，因為多個生產者並行建立，如果不是這樣，建立過程並不會是執行緒安全的。

**Producer** 和 **Consumer** 通過 **PriorityBlockingQueue** 相互連接。因為阻塞佇列的性質提供了所有必要的同步，因為阻塞佇列的性質提供了所有必要的同步，請注意，顯式同步是並不需要的 — 從佇列中讀取資料時，你不用考慮佇列中是否有任何元素，因為佇列在沒有元素時將阻塞讀取。

### 無鎖集合

[集合](./12-Collections.md) 章節強調集合是基本的程式工具，這也要求包含並發性。因此，早期的集合比如 **Vector** 和 **Hashtable** 有許多使用 **synchronized** 機制的方法。當這些集合不是在多執行緒應用中使用時，這就導致了不可接收的開銷。在 Java 1.2 版本中，新的集合庫是非同步的，而給 **Collection** 類賦予了各種 **static** **synchronized** 修飾的方法來同步不同的集合類型。雖然這是一個改進，因為它讓你可以選擇是否對集合使用同步，但是開銷仍然基於同步鎖定。 Java 5 版本添加新的集合類型，專門用於增加執行緒安全性能，使用巧妙的技術來消除鎖定。

無鎖集合有一個有趣的特性：只要讀取者僅能看到已完成修改的結果，對集合的修改就可以同時發生在讀取發生時。這是透過一些策略實現的。為了讓你了解它們是如何工作的，我們來看看其中的一些。

#### 複製策略

使用“複製”策略，修改是在資料結構一部分的單獨副本（或有時是整個資料的副本）上進行的，並且在整個修改過程期間這個副本是不可見的。僅當修改完成時，修改後的結構才與“主”資料結構安全地交換，然後讀取者才會看到修改。

在 **CopyOnWriteArrayList** ，寫入操作會複製整個底層陣列。保留原來的陣列，以便在修改複製的陣列時可以執行緒安全地進行讀取。當修改完成後，原子操作會將其交換到新陣列中，以便新的讀取操作能夠看到新陣列內容。 **CopyOnWriteArrayList** 的其中一個好處是，當多個疊代器遍歷和修改列表時，它不會拋出 **ConcurrentModificationException** 異常，因此你不用就像過去必須做的那樣，編寫特殊的程式碼來防止此類異常。

**CopyOnWriteArraySet** 使用 **CopyOnWriteArrayList** 來實現其無鎖行為。

**ConcurrentHashMap** 和 **ConcurrentLinkedQueue** 使用類似的技術來允許並發讀寫，但是只複製和修改集合的一部分，而不是整個集合。然而，讀取者仍然不會看到任何不完整的修改。**ConcurrentHashMap** **不會拋出concurrentmodificationexception** 異常。

#### 比較並交換 (CAS)

在 比較並交換 (CAS) 中，你從記憶體中獲取一個值，並在計算新值時保留原始值。然後使用 CAS 指令，它將原始值與目前記憶體中的值進行比較，如果這兩個值是相等的，則將記憶體中的舊值取代為計算新值的結果，所有操作都在一個原子操作中完成。如果原始值比較失敗，則不會進行交換，因為這意味著另一個執行緒同時修改了記憶體。在這種情況下，你的程式碼必須再次嘗試，獲取一個新的原始值並重複該操作。

如果記憶體僅輕量競爭，CAS操作幾乎總是在沒有重複嘗試的情況下完成，因此它非常快。相反，**synchronized** 操作需要考慮每次獲取和釋放鎖的成本，這要昂貴得多，而且沒有額外的好處。隨著記憶體競爭的增加，使用 CAS 的操作會變慢，因為它必須更頻繁地重複自己的操作，但這是對更多資源競爭的動態響應。這確實是一種優雅的方法。

最重要的是，許多現代處理器的組語語言中都有一條 CAS 指令，並且也被 JVM 中的 CAS 操作(例如 **Atomic** 類中的操作)所使用。CAS 指令在硬體層面中是原子性的，並且與你所期望的操作一樣快。

<!-- Summary -->
## 本章小結

本附錄主要是為了讓你在遇到底層並發程式碼時能對此有一定的了解，儘管本文還遠沒對這個主題進行全面的討論。為此，你需要先從閱讀由 Brian Goetz, Tim Peierls, Joshua Bloch, Joseph Bowbeer, David Holmes, and Doug Lea (Addison-Wesley 出版社, 2006)所著作的 *Java Concurrency in Practice* （國內譯名：Java並發編程實戰）開始了解。理想情況下，這本書會完全嚇跑你在 Java 中嘗試去編寫底層並發程式碼。如果沒有，那麼你幾乎肯定患上了達克效應(DunningKruger Effect)，這是一種認知偏差，“你知道的越少，對自己的能力就越有信心”。請記住，目前的語言設計人員仍然在清理早期語言設計人員過於自信造成的混亂(例如，查看 Thread 類中有多少方法被棄用，而 volatile 直到 Java 5 才正確工作)。

以下是並發編程的步驟:

1. 不要使用它。想一些其他方法來使你寫的程式變的更快。
2. 如果你必須使用它，請使用在 [並發編程](./24-Concurrent-Programming.md) - parallel Streams and CompletableFutures 中展示的現代進階工具。
3. 不要在任務間共享變數，在任務之間必須傳遞的任何訊息都應該使用 Java.util.concurrent 庫中的並發資料結構。
4. 如果必須在任務之間共享變數，請使用 java.util.concurrent.atomic 裡面其中一種類型，或在任何直接或間接訪問這些變數的方法上應用 synchronized。 當你不這樣做時，很容易被愚弄，以為你已經把所有東西都包括在內。 說真的，嘗試使用步驟 3。
5. 如果步驟 4 產生的結果太慢，你可以嘗試使用volatile 或其他技術來調整程式碼，但是如果你正在閱讀本書並認為你已經準備好嘗試這些方法，那麼你就超出了你的深度。 返回步驟＃1。

通常可以只使用 java.util.concurrent 庫元件來編寫並發程式，完全避免來自應用 volatile 和 synchronized 的挑戰。注意，我可以透過 [並發編程](./24-Concurrent-Programming.md)  中的範例來做到這一點。

[^1]: 在某些平台上，特別是 Windows ，預設值可能非常難以查明。你可以使用 -Xss 標誌調整堆疊大小。

[^2]: 引自 Brian Goetz, Java Concurrency in Practice 一書的作者 , 該書由 Brian Goetz, Tim Peierls, Joshua Bloch, Joseph Bowbeer, David Holmes, and Doug Lea 聯合著作 (Addison-Wesley 出版社, 2006)。↩

[^3]: 請注意，在64位處理器上可能不會發生這種情況，從而消除了這個問題。

[^4]: 這個測試的推論是，“如果某人表示執行緒是容易並且簡單的，請確保這個人沒有對你的項目做出重要的決策。如果那個人已經做出，那麼你就已經陷入麻煩之中了。”

[^5]: 這在即將產生的 C++ 的標準中得到了補救。

<!-- 分頁 -->
<div style="page-break-after: always;"></div>
