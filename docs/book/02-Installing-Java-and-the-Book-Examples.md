[TOC]

# 第二章 安裝Java和本書用例

現在，我們來為這次閱讀之旅做些準備吧！

在開始學習 Java 之前，你必須要先安裝好 Java 和本書的原始碼範例。因為考慮到可能有“專門的初學者”從本書開始學習程式，所以我會詳細地教你如何使用命令列。 如果你已經有此方面的經驗了，可以跳過這段安裝說明。如果你對此處描述的任何術語或過程仍不清楚，還可以透過 [Google](https://google.com/) 搜尋找到答案。具體的問題或困難請試著在 [StackOverflow](https://stackoverflow.com/) 上提問。或者去 [YouTube](https://youtube.com) 看有沒有相關的安裝說明。

## 編輯器

首先你需要安裝一個編輯器來建立和修改本書用例裡的 Java 程式碼。有可能你還需要使用編輯器來更改系統配置檔案。

相比一些重量級的 IDE（Integrated Development Environments，整合開發環境），如 Eclipse、NetBeans 和 IntelliJ IDEA (譯者註：做項目強烈推薦IDEA)，編輯器是一種更純粹的文字編輯器。如果你已經有了一個用著順手的 IDE，那就可以直接用了。為了方便後面的學習和統一下教學環境，我推薦大家使用 Atom 這個編輯器。大家可以在 [atom.io](https://atom.io) 上下載。

Atom 是一個免費開源、易於安裝且跨平台（支援 Window、Mac和Linux）的文字編輯器。內建支援 Java 文件。相比 IDE 的厚重，它比較輕量級，是學習本書的理想工具。Atom 包含了許多方便的編輯功能，相信你一定會愛上它！更多關於 Atom 使用的細節問題可以到它的網站上尋找。

還有很多其他的編輯器。有一種亞文化的群體，他們熱衷於爭論哪個更好用！如果你找到一個你更喜歡的編輯器，換一種使用也沒什麼難度。重要的是，你要找一個用著舒服的。

## Shell

如果你之前沒有接觸過編程，那麼有可能對 Shell（命令列視窗） 不太熟悉。shell 的歷史可以追溯到早期的計算時代，當時在電腦上的操作是都透過輸入指令進行的，電腦透過回顯響應。所有的操作都是基於文字的。

儘管和現在的圖形使用者介面相比，Shell 操作方式很原始。但是同時 shell 也為我們提供了許多有用的功能特性。在學習本書的過程中，我們會經常使用到 Shell，包括現在這部分的安裝，還有執行 Java 程式。

Mac：單擊聚光燈（螢幕右上角的放大鏡圖示），然後鍵入 `terminal`。單擊看起來像小電視螢幕的應用程式（你也可以單擊“return”）。這就啟動了你的使用者下的 shell 視窗。

windows：首先，透過目錄打開 windows 資源管理器：

- Windows 7: 單擊螢幕左下角的“開始”圖示，輸入“explorer”後按確認鍵。
- Windows 8: 按 Windows+Q，輸入 “explorer” 後按確認鍵。
- Windows 10: 按 Windows+E 打開資源管理器，導航到所需目錄，單擊視窗左上角的“文件“頁籤，選擇“打開 Window PowerShell”啟動 Shell。

Linux: 在 home 目錄打開 Shell。

- Debian: 按 Alt+F2， 在彈出的對話框中輸入“gnome-terminal”
- Ubuntu: 在螢幕中滑鼠右擊，選擇 “打開終端”，或者按住 Ctrl+Alt+T
- Redhat: 在螢幕中滑鼠右擊，選擇 “打開終端”
- Fedora: 按 Alt+F2，在彈出的對話框中輸入“gnome-terminal”

**目錄**

目錄是 Shell 的基礎元素之一。目錄用來儲存文件和其他目錄。目錄就好比樹的分支。如果書籍是你系統上的一個目錄，並且它有兩個其他目錄作為分支，例如數學和藝術，那麼我們就可以說你有一個書籍目錄，它包含數學和藝術兩個子目錄。注意：Windows 使用 `\` 而不是 `/` 來分隔路徑。

**Shell基本操作**

我在這展示的 Shell 操作和系統中大體相同。出於本書的原因，下面列舉一些在 Shell 中的基本操作：

```bash
更改目錄： cd <路徑> 
          cd .. 移動到上級目錄 
          pushd <路徑> 記住來源的同時移動到其他目錄，popd 返回來源

目錄列舉： ls 列舉出目前目錄下所有的文件和子目錄名（不包含隱藏文件），
             可以選擇使用萬用字元 * 來縮小搜尋範圍。
             範例(1)： 列舉所有以“.java”結尾的文件，輸入 ls *.java (Windows: dir *.java)
             範例(2)： 列舉所有以“F”開頭，“.java”結尾的文件，輸入ls F*.java (Windows: dir F*.java)

建立目錄： 
    Mac/Linux 系統：mkdir  
              範例：mkdir books 
    Windows   系統：md 
              範例：md books

移除文件： 
    Mac/Linux 系統：rm
              範例：rm somefile.java
    Windows   系統：del 
              範例：del somefile.java

移除目錄： 
    Mac/Linux 系統：rm -r
              範例：rm -r books
    Windows   系統：deltree 
              範例：deltree books

重複指令： !!  重複上條指令
              範例：!n 重複倒數第n條指令

指令歷史：     
    Mac/Linux 系統：history
    Windows   系統：按 F7 鍵

文件解壓：
    Linux/Mac 都有命令列解壓程式 unzip，你可以透過網際網路為 Windows 安裝命令列解壓程式 unzip。
    圖形介面下（Windows 資源管理器，Mac Finder，Linux Nautilus 或其他等效軟體）右鍵單擊該文件，
    在 Mac 上選擇“open”，在 Linux 上選擇“extract here”，或在 Windows 上選擇“extract all…”。
    要了解關於 shell 的更多訊息，請在維基百科中搜尋 Windows shell，Mac/Linux使用者可搜尋 bash shell。

```


## Java安裝

為了編譯和執行程式碼範例，首先你必須安裝 JDK（Java Development Kit，JAVA 軟體開發工具包）。本書中採用的是 JDK 8。


**Windows**

1. 以下為 Chocolatey 的[安裝說明](https://chocolatey.org/)。
2. 在命令列提示符下輸入下面的指令，等待片刻，結束後 Java 安裝完成並自動完成環境變數設定。

```bash
 choco install jdk8
```

**Macintosh**

Mac 系統自帶的 Java 版本太老，為了確保本書的程式碼範例能被正確執行，你必須將它先更新到 Java 8。我們需要管理員權限來執行下面的步驟：

1. 以下為 HomeBrew 的[安裝說明](https://brew.sh/)。安裝完成後執行指令 `brew update` 更新到最新版本
2. 在命令列下執行下面的指令來安裝 Java。

```bash
 brew cask install java
```

當以上安裝都完成後，如果你有需要，可以使用訪客帳戶來執行本書中的程式碼範例。

**Linux**

* **Ubuntu/Debian**：

```bash
     sudo apt-get update
     sudo apt-get install default-jdk
```
* **Fedora/Redhat**：

```bash
    su-c "yum install java-1.8.0-openjdk"(註：執行引號內的內容就可以安裝)
```


## 校驗安裝

打開新的命令列輸入：

```bash
java -version
```

正常情況下 你應該看到以下類似訊息(版本號訊息可能不一樣）：

```bash
java version "1.8.0_112"
Java(TM) SE Runtime Environment (build 1.8.0_112-b15)
Java HotSpot(TM) 64-Bit Server VM (build 25.112-b15, mixed mode)
```
如果提示指令找不到或者無法被識別，請根據安裝說明重試；如果還不行，嘗試到 [StackOverflow](https://stackoverflow.com/search?q=installing+java) 尋找答案。

## 安裝和執行程式碼範例

當 Java 安裝完畢，下一步就是安裝本書的程式碼範例了。安裝步驟所有平台一致：

1. 從 [GitHub 倉庫](https://github.com/BruceEckel/OnJava8-Examples/archive/master.zip)中下載本書程式碼範例
2. 解壓到你所選目錄裡。
3. 使用 Windows 資源管理器，Mac Finder，Linux 的 Nautilus 或其他等效工具瀏覽，在該目錄下打開 Shell。
4. 如果你在正確的目錄中，你應該看到該目錄中名為 gradlew 和 gradlew.bat 的文件，以及許多其他文件和目錄。目錄與書中的章節相對應。
5. 在shell中輸入下面的指令執行：

```bash
     Windows 系統：
          gradlew run

     Mac/Linux 系統：
        ./gradlew run
```

第一次安裝時 Gradle 需要安裝自身和其他的相關的包，請稍等片刻。安裝完成後，後續的安裝將會快很多。

**注意**： 第一次執行 gradlew 指令時必須連接網際網路。

**Gradle 基礎任務**

本書構建的大量 Gradle 任務都可以自動執行。Gradle 使用約定大於配置的方式，簡單設定即可具備高可用性。本書中“一起去騎行”的某些任務不適用於此或無法執行成功。以下是你通常會使用上的 Gradle 任務列表：

```bash
    編譯本書中的所有 java 文件，除了部分錯誤示範的
    gradlew compileJava

    編譯並執行 java 文件（某些文件是庫元件）
    gradlew run

    執行所有的單元測試（在本書第16章會有詳細介紹）
    gradlew test

    編譯並執行一個具體的範例程式
    gradlew <本書章節>:<範例名稱>
    範例：gradlew objects:HelloDate
```
<!-- 分頁 -->

<div style="page-break-after: always;"></div>
