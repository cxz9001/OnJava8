[TOC]

<!-- Concurrent Programming -->

# 第二十四章 並發編程

> 愛麗絲：“我可不想到瘋子中間去”
>
> 貓咪：“啊，那沒轍了，我們這都是瘋子。我瘋了，你也瘋了”
>
> 愛麗絲：“你怎麼知道我瘋了”。
>
> 貓咪：“你一定是瘋了，否則你就不會來這裡” ——愛麗絲夢遊仙境 第 6 章。


在本章之前，我們慣用一種簡單順序的敘事方式來編程，有點類似文學中的意識流：第一件事發生了，然後是第二件，第三件……總之，我們完全掌握著事情發生的進展和順序。如果將值設定為 5，再看時它已變成 47 的話，結果就很匪夷所思了。

現在，我們來到了陌生的並發世界。這樣的結果一點都不奇怪，因為你原來信賴的一切都不再可靠。它可能有效，也可能無效。更可能得是，它在某些情況下會起作用，而在另一些情況下則不會。只有了解了這些情況，我們才能正確地行事。

作為類比，我們正常生活是發生在經典牛頓力學中的。物體具有質量：會墜落並轉移動量。電線有電阻，光直線傳播。假如我們進入極小、極熱、極冷、或是極大的世界（我們不能生存），這些現象就會發生變化。我們無法判斷某物體是粒子還是波，光是否受到重力影響，一些物質還會變為超導體。

假設我們處在多條故事線並行的間諜小說裡，非單一意識流地敘事：第一個間諜在岩石底留下了微縮膠片。當第二個間諜來取時，膠片可能已被第三個間諜拿走。小說並沒有交代此處的細節。所以直到故事結尾，我們都沒搞清楚到底發生了什麼事。

構建並發程式好比玩[搭積木 ](https://en.wikipedia.org/wiki/Jenga) 遊戲。每拉出一塊放在塔頂時都有崩塌的可能。每個積木塔和應用程式都是獨一無二的，有著自己的作用。你在某個系統構建中學到的知識並不一定適用於下一個系統。

本章是對並發概念最基本的介紹。雖然我們用到了現代的 Java 8 工具來示範原理，但還遠未及全面論述並發。我的目標是為你提供足夠的基礎知識，使你能夠把握問題的複雜性和危險性，從而安全地渡過這片鯊魚肆虐的困難水域。

更多繁瑣和底層的細節，請參閱附錄：[並發底層原理 ](./Appendix-Low-Level-Concurrency.md)。要進一步深入這個領域，你還必須閱讀 *Brian Goetz* 等人的 《Java Concurrency in Practice》。在撰寫本文時，該書已有十多年的歷史了，但它仍包含我們必須要了解和明白的知識要點。理想情況下，本章和上述附錄是閱讀該書的良好前提。另外，*Bill Venner* 的 《Inside the Java Virtual Machine》也很值得一看。它詳細描述了 JVM 的內部工作方式，包括執行緒。


<!-- The Terminology Problem -->

## 術語問題

術語“並發”，“並行”，“多任務”，“多處理”，“多執行緒”，分布式系統（可能還有其他）在整個編程文獻中都以多種相互衝突的方式使用，並且經常被混為一談。
*Brian Goetz* 在他 2016 年《從並發到並行》的演講中指出了這一點，之後提出了合理的二分法：

- 並發是關於正確有效地控制對共享資源的訪問。

- 並行是使用額外的資源來更快地產生結果。

這些定義很好，但是我們已有幾十年混亂使用和抗拒解決此問題的歷史了。一般來說，當人們使用“並發”這個詞時，他們的意思是“所有的一切”。事實上，我自己也經常陷入這樣的想法。在大多數書籍中，包括 *Brian Goetz* 的 《Java Concurrency in Practice》，都在標題中使用這個詞。

“並發”通常表示：不止一個任務正在執行。而“並行”幾乎總是代表：不止一個任務同時執行。現在你能看到問題所在了嗎？“並行”也有不止一個任務正在執行的語義在裡面。區別就在於細節：究竟是怎麼“執行”的。此外，還有一些場景重疊：為並行編寫的程式有時在單處理器上執行，而一些並發編程系統可以利用多處理器。

還有另一種方法，在減速發生的地方寫下定義（原文 Here’s another approach, writing the definitions around where the
slowdown occurs）：

**並發**

同時完成多任務。無需等待目前任務完成即可執行其他任務。“並發”解決了程式因外部控制而無法進一步執行的阻塞問題。最常見的例子就是 I/O 操作，任務必須等待資料輸入（在一些例子中也稱阻塞）。這個問題常見於 I/O 密集型任務。

**並行**

同時在多個位置完成多任務。這解決了所謂的 CPU 密集型問題：將程式分為多部分，在多個處理器上同時處理不同部分來加快程式執行效率。

上面的定義說明了這兩個術語令人困惑的原因：兩者的核心都是“同時完成多個任務”，不過並行增加了跨多個處理器的分布。更重要的是，它們可以解決不同類型的問題：並行可能對解決 I / O 密集型問題沒有任何好處，因為問題不在於程式的整體執行速度，而在於 I/O 阻塞。而嘗試在單個處理器上使用並發來解決計算密集型問題也可能是浪費時間。兩種方法都試圖在更短的時間內完成更多工作，但是它們實現加速的方式有所不同，這取決於問題施加的約束。

這兩個概念混合在一起的一個主要原因是包括 Java 在內的許多程式語言使用相同的機制 - **執行緒**來實現並發和並行。

我們甚至可以嘗試以更細的粒度去進行定義（然而這並不是標準化的術語）：

- **純並發**：仍然在單個 CPU 上執行任務。純並發系統比順序系統更快地產生結果，但是它的執行速度不會因為處理器的增加而變得更快。
- **並發-並行**：使用並發技術，結果程式可以利用更多處理器更快地產生結果。
- **並行-並發**：使用平行程式技術編寫，如果只有一個處理器，結果程式仍然可以執行（Java 8 **Streams** 就是一個很好的例子）。
- **純並行**：除非有多個處理器，否則不會執行。

在某些情況下，這可能是一個有用的分類法。

支援並發性的語言和庫似乎是[抽象洩露（Leaky Abstraction）](https://en.wikipedia.org/wiki/Leaky_abstraction) 一詞的完美候選。抽象的目標是“抽象出”那些對於手頭想法不重要的東西，以封鎖不必要的細節。如果抽象是有漏洞的，那些碎片和細節就會不斷重新聲明自己是重要的，無論你廢了多少功夫來隱藏它們。

我開始懷疑是否真的有高度抽象。因為當編寫這類程式時，底層的系統、工具，甚至是關於 CPU 快取如何工作的細節，都永遠不會被封鎖。最後，如果你非常小心，你創作的東西在特定的情況下工作，但在其他情況下不工作。有時是兩台機器的配置方式不同，有時是程式的估計負載不同。這不是 Java 特有的 - 這是並發和並行編程的本質。

你可能會認為[純函數式 ](https://en.wikipedia.org/wiki/Purely_functional) 語言沒有這些限制。實際上，純函數式語言解決了大量並發問題，所以如果你正在解決一個困難的並發問題，你可以考慮用純函數語言編寫這個部分。但最終，如果你編寫一個使用佇列的系統，例如，如果該系統沒有被正確地調整，並且輸入速率也沒有被正確地估計或限制（在不同的情況下，限制意味著具有不同的影響的不同東西），該佇列要嘛被填滿並阻塞，要嘛溢位。最後，你必須了解所有細節，任何問題都可能會破壞你的系統。這是一種非常不同的程式方式。

<!-- A New Definition ofConcurrencyFor -->
### 並發的新定義

幾十年來，我一直在努力解決各種形式的並發問題，其中一個最大的挑戰一直是簡單地定義它。在撰寫本章的過程中，我終於有了這樣的洞察力，我認為可以定義它：
>** 並發性是一系列性能技術，專注於減少等待**

這實際上是一個相當複雜的表述，所以我將其分解：

- 這是一個集合：包含許多不同的方法來解決這個問題。這是使定義並發性如此具有挑戰性的問題之一，因為技術差異很大。
- 這些是性能技術：就是這樣。並發的關鍵點在於讓你的程式執行得更快。在 Java 中，並發是非常棘手和困難的，所以絕對不要使用它，除非你有一個重大的效能問題 - 即使這樣，使用最簡單的方法產生你需要的效能，因為並發很快變得無法管理。
- “減少等待”部分很重要而且微妙。無論（例如）你執行多少個處理器，你只能在等待發生時產生效益。如果你發起 I/O 請求並立即獲得結果，沒有延遲，因此無需改進。如果你在多個處理器上執行多個任務，並且每個處理器都以滿容量執行，並且沒有任務需要等待其他任務，那麼嘗試提高吞吐量是沒有意義的。並發的唯一機會是如果程式的某些部分被迫等待。等待可以以多種形式出現 - 這解釋了為什麼存在如此不同的並發方法。

值得強調的是，這個定義的有效性取決於“等待”這個詞。如果沒有什麼可以等待，那就沒有機會去加速。如果有什麼東西在等待，那麼就會有很多方法可以加快速度，這取決於多種因素，包括系統執行的配置，你要解決的問題類型以及其他許多問題。
<!-- Concurrency Superpowers -->

## 並發的超能力

想像一下，你置身於一部科幻電影。你必須在一棟大樓中找到一個東西，它被小心而巧妙地隱藏在大樓一千萬個房間中的一間。你進入大樓，沿著走廊走下去。走廊是分開的。

一個人完成這項任務要花上一百輩子的時間。

現在假設你有一個奇怪的超能力。你可以將自己一分為二，然後在繼續前進的同時將另一半送到另一個走廊。每當你在走廊或樓梯上遇到分隔到下一層時，你都會重複這個分裂的技巧。最終，整個建築中的每個走廊的終點都有一個你。

每個走廊都有一千個房間。此時你的超能力變得弱了一點，你只能複製 50 個自己來並發搜尋走廊裡面的房間。

一旦複製體進入房間，它必須搜尋房間的每個角落。這時它切換到了第二種超能力。它分裂成了一百萬個奈米機器人，每個機器人都會飛到或爬到房間裡一些看不見的地方。你不需要了解這種功能 - 一旦你開啟它就會自動工作。在他們自己的控制下，奈米機器人開始行動，搜尋房間然後回來重新組裝成你，突然間，不知怎麼的，你就知道這間房間裡有沒有那個東西。

我很想說，“並發就是剛才描述的置身於科幻電影中的超能力“就像你自己可以一分為二然後解決更多的問題一樣簡單。但是問題在於，我們來描述這種現象的任何模型最終都是洩漏抽象的（leaky abstraction)。

以下是其中一個漏洞：在理想的世界中，每次複製自己時，還需要複製一個物理處理器來執行該複製。這當然是不現實的——實際上，你的機器上一般只有 4 個或 8 個處理器核心（編寫本文時的典型情況）。你也可能更多，但仍有很多情況下只有一個單核處理器。在關於抽象的討論中，分配物理處理器核心這本身就是抽象的洩露，甚至也可以支配你的決策。

讓我們在科幻電影中改變一些東西。現在當每個複製搜尋者最終到達一扇門時，他們必須敲門並等到有人開門。如果每個搜尋者都有一個處理器核心，這沒有問題——只是空閒等待直到有人開門。但是如果我們只有 8 個處理器核心卻有幾千個搜尋者，我們不希望處理器僅僅因為某個搜尋者恰好在等待回答中被鎖住而閒置下來。相反，我們希望將處理器應用於可以真正執行工作的搜尋者身上，因此需要將處理器從一個任務切換到另一個任務的機制。

許多模型能夠有效地隱藏處理器的數量，允許你假裝有很多個處理器。但在某些情況下，這種方法會失效，這時你必須知道處理器核心的真實數量，以便處理這個問題。

最大的影響之一取決於您是使用單核處理器還是多核處理器。如果你只有單核處理器，那麼任務切換的成本也由該核心承擔，將並發技術應用於你的系統會使它執行得更慢。

這可能會讓你以為，在單核處理器的情況下，編寫並發程式碼是沒有意義的。然而，有些情況下，並發模型會產生更簡單的程式碼，光是為了這個目的就值得捨棄一些性能。

在複製體敲門等待的情況下，即使單核處理器系統也能從並發中受益，因為它可以從等待（阻塞）的任務切換到準備執行的任務。但是如果所有任務都可以一直執行那麼切換的成本反而會使任務變慢，在這種情況下，如果你有多個行程，並發通常只會有意義。

如果你正在嘗試破解某種密碼，在同一時間內參與破解的執行緒越多，你越快得到答案的可能性就越大。每個執行緒都能持續使用你所分配的處理器時間，在這種情況下（CPU 密集型問題），你程式碼中的執行緒數應該和你擁有的處理器的核心數保持一致。

在接聽電話的客戶服務部門，你只有一定數量的員工，但是你的部門可能會打來很多電話。這些員工（處理器）一次只能接聽一個電話直到打完，此時其它打來的電話必須排隊等待。

在“鞋匠和精靈”的童話故事中，鞋匠有很多工作要做，當他睡著時，出現了一群精靈來為他製作鞋子。這裡的工作是分布式的，但即使使用大量的物理處理器，在製造鞋子的某些部件時也會產生限制——例如，如果鞋底的製作時間最長，這就限制了鞋子的製作速度，這也會改變你設計解決方案的方式。

因此，你要解決的問題推動了解決方案的設計。將一個問題分解成“獨立執行”的子任務，這是一種美好的抽象，然後就是殘酷的現實：物理現實不斷地侵入和動搖這個抽象。

這只是問題的一部分：考慮一個製作蛋糕的工廠。我們以某種方式把製作蛋糕的任務分給了工人們，但是現在是時候讓工人把蛋糕放在盒子裡了。那裡有一個盒子，準備存放蛋糕。但是在工人把蛋糕放進盒子之前，另一個工人就衝過去，把蛋糕放進盒子裡，砰！這兩個蛋糕撞到一起砸壞了。這是常見的“共享記憶體”問題，會產生所謂的競態條件（race condition），其結果取決於哪個工人能先把蛋糕放進盒子裡（通常使用鎖機制來解決問題，因此一個工作人員可以先抓住一個盒子並防止蛋糕被砸爛）。

當“同時”執行的任務相互干擾時，就會出現問題。這可能以一種微妙而偶然的方式發生，因此可以說並發是“可以論證的確定性，但實際上是不確定性的”。也就是說，假設你很小心地編寫並發程式，而且通過了程式碼檢查可以正確執行。然而實際上，我們編寫的並發程式大部分情況下都能正常執行，但是在一些情況下會失敗。這些情況可能永遠不會發生，或者在你在測試期間幾乎很難發現它們。實際上，編寫測試程式碼通常無法為並發程式生成故障條件。由此產生的失敗只會偶爾發生，因此它們以客戶投訴的形式出現。這是學習並發中最強有力的論點之一：如果你忽略它，你可能會受傷。

因此，並發似乎充滿了危險，如果這讓你有點害怕，這可能是一件好事。儘管 Java 8 在併發性方面做出了很大改進，但仍然沒有像編譯時驗證 (compile-time verification) 或受檢查的異常 (checked exceptions) 那樣的安全網來告訴你何時出現錯誤。關於並發，你只能依靠自己，只有知識淵博、保持懷疑和積極進取的人，才能用 Java 編寫可靠的並發程式碼。

<!-- Concurrency is for Speed -->
<!-- 不知道是否可以找到之前翻譯的針對速度感覺太直了 -->

## 並發為速度而生

在聽說並發編程的問題之後，你可能會想知道它是否值得這麼麻煩。答案是“不，除非你的程式執行速度不夠快。”並且在決定用它之前你會想要仔細思考。不要隨便跳進並發編程的悲痛之中。如果有一種方法可以在更快的機器上執行你的程式，或者如果你可以對其進行分析並發現瓶頸並在該位置取代更快的演算法，那麼請執行此操作。只有在顯然沒有其他選擇時才開始使用並發，然後僅在必要的地方去使用它。

速度問題一開始聽起來很簡單：如果你想要一個程式執行得更快，將其分解為多個部分，並在單獨的處理器上執行每個部分。隨著我們提高時鐘速度的能力耗盡（至少對傳統晶片而言），速度的提高是出現在多核處理器的形式而不是更快的晶片。為了使程式執行得更快，你必須學會利用那些額外的處理器（譯者註：處理器一般代表 CPU 的一個邏輯核心），這是並發所帶來的好處之一。

對於多處理器機器，可以在這些處理器之間分配多個任務，這可以顯著提高吞吐量。強大的多處理器 Web 伺服器通常就是這種情況，它可以在程式中為 CPU 分配大量使用者請求，每個請求分配一個執行緒。

但是，並發通常可以提高在單處理器上執行的程式的效能。這聽起來有點違反直覺。如果你仔細想想，由於上下文切換的成本增加（從一個任務切換到另一個任務），在單個處理器上執行的並發程式實際上應該比程式的所有部分順序執行具有更多的開銷。從表面上看，將程式的所有部分作為單個任務執行，並且節省上下文切換的成本，這樣看似乎更划算。

使這個問題變得有些不同的是阻塞。如果程式中的某個任務由於程式控制之外的某種情況而無法繼續（通常是 I/O），我們就稱該任務或執行緒已阻塞（在我們的科幻故事中，就是複製人已經敲門並等待它打開）。如果沒有並發，整個程式就會停下來，直到外部條件發生變化。但是，如果使用並發編寫程式，則當一個任務被阻塞時，程式中的其他任務可以繼續執行，因此整個程式得以繼續執行。事實上，從性能的角度來看，如果沒有任務會阻塞，那麼在單處理器機器上使用並發是沒有意義的。

單處理器系統中性能改進的一個常見例子是事件驅動編程，特別是使用者介面編程。考慮一個程式執行一些耗時操作，最終忽略使用者輸入導致無響應。如果你有一個“退出”按鈕，你不想在你編寫的每段程式碼中都檢查它的狀態（輪詢）。這會產生笨拙的程式碼，也無法保證程式設計師不會忘了檢查。沒有並發，生成可響應使用者介面的唯一方法是讓所有任務都定期檢查使用者輸入。透過建立單獨的執行緒以執行使用者輸入的響應，能夠讓程式保證一定程度的響應能力。

實現並發的一種簡單方式是使用作業系統級別的行程。與執行緒不同，行程是在其自己的地址空間中執行的獨立程式。行程的優勢在於，因為作業系統通常將一個行程與另一個行程隔離，因此它們不會相互干擾，這使得行程編程相對容易。相比之下，執行緒之間會共享記憶體和 I/O 等資源，因此編寫多執行緒程式最基本的困難，在於協調不同執行緒驅動的任務之間對這些資源的使用，以免這些資源同時被多個任務訪問。
<!-- 文獻引用未加，因為暫時沒看到很好的解決辦法 -->
有些人甚至提倡將行程作為唯一合理的並發實現方式[^1]，但遺憾的是，通常存在數量和開銷方面的限制，從而阻止了行程在併發範圍內的適用性（最終你會習慣標準的並發限制，“這種方法適用於一些情況但不適用於其他情況”）

一些程式語言旨在將並發任務彼此隔離。這些通常被稱為_函數式語言_，其中每個函數呼叫不產生副作用（不會干擾到其它函數），所以可以作為獨立的任務來驅動。Erlang 就是這樣一種語言，它包括一個任務與另一個任務進行通信的安全機制。如果發現程式的某一部分必須大量使用並發，並且在嘗試構建該部分時遇到了過多的問題，那麼可以考慮使用這些專用的並發語言建立程式的這個部分。
<!-- 文獻標記 -->
Java 採用了更傳統的方法[^2]，即在順序語言之上添加對執行緒的支援而不是在多任務作業系統中分叉外部行程，執行緒是在表示處理程序的單個行程內建立任務。

並發會帶來各種成本，包括複雜性成本，但可以被程式設計、資源平衡和使用者便利性方面的改進所抵消。通常，並發性使你能夠建立更低耦合的設計；另一方面，你必須特別關注那些使用了並發操作的程式碼。

<!-- The Four Maxims of Java Concurrency -->
## Java 並發的四句格言

在經歷了多年 Java 並發的實踐之後，我總結了以下四個格言：
>1.不要用它（避免使用並發）
>
>2.沒有什麼是真的，一切可能都有問題
>
>3.僅僅是它能執行，並不意味著它沒有問題
>
>4.你必須理解它（逃不掉並發）

這些格言專門針對 Java 的並發設計問題，儘管它們也可以適用於其他一些語言。但是，確實存在旨在防止這些問題的語言。

### 1.不要用它

（而且不要自己去實現它）

避免陷入並發所帶來的玄奧問題的最簡單方法就是不要用它。儘管嘗試一些簡單的東西可能很誘人，也似乎足夠安全，但是陷阱卻是無窮且微妙的。如果你能避免使用它，你的生活將會輕鬆得多。

使用並發唯一的正當理由是速度。如果你的程式執行速度不夠快——這裡要小心，因為僅僅想讓它執行得更快不是正當理由——應該首先用一個分析器（參見程式碼校驗章中分析和最佳化）來發現你是否可以執行其他一些最佳化。

如果你被迫使用並發，請採取最簡單，最安全的方法來解決問題。使用知名的庫並儘可能少地自己編寫程式碼。對於並發，就沒有“太簡單了”——自作聰明是你的敵人。

### 2.沒有什麼是真的，一切可能都有問題

不使用並發編程，你已經預料到你的世界具有確定的順序和一致性。對於變數賦值這樣簡單的操作，很明顯它應該總是能夠正常工作。

在併發領域，有些事情可能是真的而有些事情卻不是，以至於你必須假設沒有什麼是真的。你必須質疑一切。即使將變數設定為某個值也可能不會按預期的方式工作，事情從這裡開始迅速惡化。我已經熟悉了這樣一種感覺：我認為應該明顯奏效的東西，實際上卻行不通。

在非並發編程中你可以忽略的各種事情，在併發下突然變得很重要。例如，你必須了解處理器快取以及保持本機快取與主記憶體一致的問題，你必須理解物件構造的深層複雜性，這樣你的建構子就不會意外地暴露資料，以致於被其它執行緒更改。這樣的例子不勝列舉。

雖然這些主題過於複雜，無法在本章中給你提供專業知識（同樣，請參見 Java Concurrency in Practice），但你必須了解它們。

### 3.僅僅是它能執行，並不意味著它沒有問題

我們很容易編寫出一個看似正常實則有問題的並發程式，而且問題只有在極少的情況下才會顯現出來——在程式部署後不可避免地會成為使用者問題（投訴）。

- 你不能驗證出並發程式是正確的，你只能（有時）驗證出它是不正確的。
- 大多數情況下你甚至沒辦法驗證：如果它出問題了，你可能無法檢測到它。
- 你通常無法編寫有用的測試，因此你必須依靠程式碼檢查和對並發的深入了解來發現錯誤。
- 即使是有效的程式也只能在其設計參數下工作。當超出這些設計參數時，大多數並發程式會以某種方式失敗。

在其他 Java 主題中，我們養成了決定論的觀念。一切都按照語言的承諾的（或暗示的）發生，這是令人欣慰的也是人們所期待的——畢竟，程式語言的意義就是讓機器做我們想要它做的事情。從確定性編程的世界進入並發編程領域，我們遇到了一種稱為 [鄧寧-克魯格效應](https://en.wikipedia.org/wiki/Dunning%E2%80%93Kruger_effect) 的認知偏差，可以概括為“無知者無畏”，意思是：“相對不熟練的人擁有著虛幻的優越感，錯誤地評估他們的能力遠高於實際。

我自己的經驗是，無論你是多麼確定你的程式碼是_執行緒安全_的，它都可能是有問題的。你可以很容易地了解所有的問題，然後幾個月或幾年後你會發現一些概念，讓你意識到你編寫的大多數程式碼實際上都容易受到並發 bug 的影響。當某些程式碼不正確時，編譯器不會告訴你。為了使它正確，在研究程式碼時，必須將並發性的所有問題都放在前腦中。

在 Java 的所有非並發領域，“沒有明顯的 bug 而且沒有編譯報錯“似乎意味著一切都好。但對於並發，它沒有任何意義。在這種情況你最糟糕的表現就是“自信”。

### 4.你必須理解它

在格言 1-3 之後，你可能會對並發性感到害怕，並且認為，“到目前為止，我已經避免了它，也許我可以繼續避免它。

這是一種理性的反應。你可能知道其他更好地被設計用於構建並發程式的程式語言——甚至是在 JVM 上執行的語言（從而提供與 Java 的輕鬆通信），例如 Clojure 或 Scala。為什麼不用這些語言來編寫並發部分，然後用Java來做其他的事情呢?

唉，你不能輕易逃脫：

- 即使你從未顯示地建立一個執行緒，你使用的框架也可能——例如，Swing 圖形使用者介面（GUI）庫，或者像 **Timer** 類（計時器）那樣簡單的東西。
- 最糟糕的是：當你建立元件時，必須假設這些元件可能會在多執行緒環境中重用。即使你的解決方案是放棄並聲明你的元件是“非執行緒安全的”，你仍然必須充分了解這樣一個語句的重要性及其含義。

人們有時會認為並發對於介紹語言的書來說太進階了，因此不適合放在其中。他們認為並發是一個獨立的主題，並且對於少數出現在日常的程式設計中的情況（例如圖形使用者介面），可以用特殊的慣用法來處理。如果你可以迴避，為什麼還要介紹這麼複雜的主題呢？

唉，如果是這樣就好了。遺憾的是，對於執行緒何時出現在 Java 程式中，這不是你能決定的。僅僅是你自己沒有啟動執行緒，並不代表你就可以迴避編寫使用執行緒的程式碼。例如，Web 系統是最常見的 Java 應用之一，本質上是多執行緒的 Web 伺服器，通常包含多個處理器，而並行是利用這些處理器的理想方式。儘管這樣的系統看起來很簡單，但你必須理解並發才能正確地編寫它。

Java 是一種多執行緒語言，不管你有沒有意識到並發問題，它就在那裡。因此，有很多使用並發的 Java 程式，要嘛只是偶然執行，要嘛大部分時間都在執行，並且會因為未被發現的並發缺陷而三不五時地神秘崩潰。有時這種崩潰是相對溫和的，但有時它意味著遺失有價值的資料，如果你沒有意識到並發問題，你最終可能會把問題歸咎於其他地方而不是你的程式碼中。如果將程式移動到多處理器系統中，這些類型的問題還會被暴露或放大。基本上，了解並發可以使你意識到明顯正確的程式也可能會表現出錯誤的行為。

<!-- The Brutal Truth -->
## 殘酷的真相

當人類開始烹飪他們的食物時，他們大大減少了他們的身體分解和消化食物所需的能量。烹飪創造了一個“外化的胃”，從而釋放出能量去發展其他的能力。火的使用促成了文明。

我們現在透過電腦和網路技術創造了一個“外化大腦”，開始了第二次基本轉變。雖然我們只是觸及表面，但已經引發了其他轉變，例如設計生物機制的能力，並且已經看到文化演變的顯著加速（過去，人們透過旅遊進行文化交流，但現在他們開始在網際網路上做這件事）。這些轉變的影響和好處已經超出了科幻作家預測它們的能力（他們在預測文化和個人變化，甚至技術轉變的次要影響方面都特別困難）。

有了這種根本性的人類變化，看到許多破壞和失敗的實驗並不令人驚訝。實際上，進化依賴於無數的實驗，其中大多數都失敗了。這些實驗是向前發展的必要條件。

Java 是在充滿自信，熱情和睿智的氛圍中建立的。在發明一種程式語言時，很容易感覺語言的初始可塑性會持續存在一樣，你可以把某些東西拿出來，如果不能解決問題，那麼就修復它。程式語言以這種方式是獨一無二的 - 它們經歷了類似水的改變：氣態，液態和最終的固態。在氣態階段，靈活性似乎是無限的，並且很容易認為它總是那樣。一旦人們開始使用你的語言，變化就會變得更加嚴重，環境變得更加黏稠。語言設計的過程本身就是一門藝術。

緊迫感來自網際網路的最初興起。它似乎是一場比賽，第一個透過起跑線的人將“獲勝”（事實上，Java，JavaScript 和 PHP 等語言的流行程度可以證明這一點）。唉，透過匆忙設計語言而產生的認知負荷和技術債務最終會趕上我們。

[Turing completeness](https://en.wikipedia.org/wiki/Turing_completeness) 是不足夠的;語言需要更多的東西：它們必須能夠創造性地表達，而不是用不必要的東西來衡量我們。解放我們的心理能力只是為了扭轉並再次陷入困境，這是毫無意義的。我承認，儘管存在這些問題，我們已經完成了令人驚奇的事情，但我也知道如果沒有這些問題我們能做得更多。

熱情使原始 Java 設計師加入了一些似乎有必要的特性。信心（以及氣態的初始語言）讓他們認為任何問題隨後都可以解決。在時間軸的某個地方，有人認為任何加入 Java 的東西是固定的和永久性的 -他們非常有信心，並相信第一個決定永遠是正確的，因此我們看到 Java 的體系中充斥著糟糕的決策。其中一些決定最終沒有什麼後果;例如，你可以告訴人們不要使用 Vector，但只能在語言中繼續保留它以便對之前版本的支援。

執行緒包含在 Java 1.0 中。當然，對 java 來說支援並發是一個很基本的設計決定，該特性影響了這個語言的各個角落，我們很難想像以後在以後的版本添加它。公平地說，當時並不清楚基本的並發性是多少。像 C 這樣的其他語言能夠將執行緒視為一個附加功能，因此 Java 設計師也紛紛效仿，包括一個 Thread 類和必要的 JVM 支援（這比你想像的要複雜得多）。

C 語言是程序導向語言，這限制了它的野心。這些限制使附加執行緒庫合理。當採用原始模型並將其貼到複雜語言中時，Java 的大規模擴展迅速暴露了基本問題。在 Thread 類中的許多方法的棄用以及後續的進階庫浪潮中，這種情況變得明顯，這些庫試圖提供更好的並發抽象。

不幸的是，為了在更進階別的語言中獲得並發性，所有語言功能都會受到影響，包括最基本的功能，例如標識符代表可變值。在簡化並發編程中，所有函數和方法中為了保持事物不變和防止副作用都要做出巨大的改變（這些是純函數式程式語言的基礎），但當時對於主流語言的建立者來說似乎是奇怪的想法。最初的 Java 設計師要嘛沒有意識到這些選擇，要嘛認為它們太不同了，並且會勸退許多潛在的語言使用者。我們可以慷慨地說，語言設計社群當時根本沒有足夠的經驗來理解調整在執行緒庫中的影響。

Java 實驗告訴我們，結果是悄然災難性的。程式設計師很容易陷入認為 Java 執行緒並不那麼困難的陷阱。表面上看起來正常工作的程式實際上充滿了微妙的並發 bug。

為了獲得正確的並發性，語言功能必須從頭開始設計並考慮並發性。木已成舟；Java 將不再是為並發而設計的語言，而只是一種允許並發的語言。

儘管有這些基本的不可修復的缺陷，但令人印象深刻的是它已經走了這麼遠。Java 的後續版本添加了庫，以便在使用並發時提升抽象級別。事實上，我根本不會想到有可能在 Java 8 中進行改進：並行流和 **CompletableFutures**  - 這是驚人的史詩般的變化，我會驚奇地重複的查看它[^3]。

這些改進非常有用，我們將在本章重點介紹並行流和 **CompletableFutures** 。雖然它們可以大大簡化你對並發和後續程式碼的思考方式，但基本問題仍然存在：由於 Java 的原始設計，程式碼的所有部分仍然很脆弱，你仍然必須理解這些複雜和微妙的問題。Java 中的執行緒絕不是簡單或安全的;那種經歷必須降級為另一種更新的語言。

<!-- The Rest of the Chapter -->
## 本章其餘部分

這是我們將在本章的其餘部分介紹的內容。請記住，本章的重點是使用最新的進階 Java 並髮結構。相比於舊的替代品，使用這些會使你的生活更加輕鬆。但是，你仍會在遺留程式碼中遇到一些低級工具。有時，你可能會被迫自己使用其中的一些。附錄：[並發底層原理 ](./Appendix-Low-Level-Concurrency.md) 包含一些更原始的 Java 並發元素的介紹。

- Parallel Streams（並行流）
到目前為止，我已經強調了 Java 8 Streams 提供的改進語法。現在該語法（作為一個粉絲，我希望）會使你感到舒適，你可以獲得額外的好處：你可以透過簡單地將 parallel() 添加到表達式來並行化流。這是一種簡單，強大，坦率地說是利用多處理器的驚人方式

添加 parallel() 來提高速度似乎是微不足道的，但是，唉，它就像你剛剛在[殘酷的真相 ](#The-Brutal-Truth) 中學到的那樣簡單。我將示範並解釋一些盲目添加 parallel() 到 Stream 表達式的缺陷。

- 建立和執行任務
任務是一段可以獨立執行的程式碼。為了解釋建立和執行任務的一些基礎知識，本節介紹了一種比並行流或 CompletableFutures 更簡單的機制：Executor。執行者管理一些低級 Thread 物件（Java 中最原始的並髮形式）。你建立一個任務，然後將其交給 Executor 去執行。

有多種類型的 Executor 用於不同的目的。在這裡，我們將展示規範形式，代表建立和執行任務的最簡單和最佳方法。

- 終止長時間執行的任務
任務獨立執行，因此需要一種機制來關閉它們。典型的方法使用了一個標誌，這引入了共享記憶體的問題，我們將使用 Java 的“Atomic”庫來迴避它。
- Completable Futures
當你將衣服帶到乾洗店時，他們會給你一張收據。你繼續完成其他任務，當你的衣服洗乾淨時你可以把它取走。收據是你與乾洗店在後台執行的任務的連接。這是 Java 5 中引入的 Future 的方法。

Future 比以前的方法更方便，但你仍然必須出現並用收據取出乾洗，如果任務沒有完成你還需要等待。對於一系列操作，Futures 並沒有真正幫助那麼多。

Java 8 CompletableFuture 是一個更好的解決方案：它允許你將操作連結在一起，因此你不必將程式碼寫入介面排序操作。有了 CompletableFuture 完美的結合，就可以更容易地做出“採購原料，組合成分，烹飪食物，提供食物，收拾餐具，儲存餐具”等一系列鏈式操作。

- 死鎖
某些任務必須去**等待 - 阻塞**來獲得其他任務的結果。被阻止的任務有可能等待另一個被阻止的任務，另一個被阻止的任務也在等待其他任務，等等。如果被阻止的任務鏈循環到第一個，沒有人可以取得任何進展，你就會陷入死鎖。

如果在執行程式時沒有立即出現死鎖，則會出現最大的問題。你的系統可能容易出現死鎖，並且只會在某些條件下死鎖。程式可能在某個平台上（例如在你的開發機器）執行正常，但是當你將其部署到不同的硬體時會開始死鎖。

死鎖通常源於細微的程式錯誤;一系列無辜的決定，最終意外地建立了一個依賴循環。本節包含一個經典範例，示範了死鎖的特性。

* 努力，複雜，成本

我們將透過模擬建立披薩的過程完成本章，首先使用並行流實現它，然後是 CompletableFutures。這不僅僅是兩種方法的比較，更重要的是探索你應該投入多少工作來使你的程式變得更快。

<!-- Parallel Streams -->
## 並行流

Java 8 流的一個顯著優點是，在某些情況下，它們可以很容易地並行化。這來自庫的仔細設計，特別是流使用內部疊代的方式 - 也就是說，它們控制著自己的疊代器。特別是，他們使用一種特殊的疊代器，稱為 Spliterator，它被限制為易於自動分割。我們只需要念 `.parallel()` 就會產生魔法般的結果，流中的所有內容都作為一組並行任務執行。如果你的程式碼是使用 Streams 編寫的，那麼並行化以提高速度似乎是一種瑣事

例如，考慮來自 Streams 的 Prime.java。尋找質數可能是一個耗時的過程，我們可以看到該程式的計時：

```java
// concurrent/ParallelPrime.java
import java.util.*;
import java.util.stream.*;
import static java.util.stream.LongStream.*;
import java.io.*;
import java.nio.file.*;
import onjava.Timer;

public class ParallelPrime {
    static final int COUNT = 100_000;
    public static boolean isPrime(long n){
        return rangeClosed(2, (long)Math.sqrt(n)).noneMatch(i -> n % i == 0);
        }
    public static void main(String[] args)
        throws IOException {
        Timer timer = new Timer();
        List<String> primes =
            iterate(2, i -> i + 1)
                .parallel()              // [1]
                .filter(ParallelPrime::isPrime)
                .limit(COUNT)
                .mapToObj(Long::toString)
                .collect(Collectors.toList());
        System.out.println(timer.duration());
        Files.write(Paths.get("primes.txt"), primes, StandardOpenOption.CREATE);
        }
    }
```

輸出結果：

```
    Output:
    1224
```

請注意，這不是微基準測試，因為我們計時整個程式。我們將資料儲存在磁碟上以防止編譯器過激的最佳化;如果我們沒有對結果做任何事情，那麼一個進階的編譯器可能會觀察到程式沒有意義並且終止了計算（這不太可能，但並非不可能）。請注意使用 nio2 庫編寫文件的簡單性（在[文件 ](./17-Files.md) 一章中有描述）。

當我注釋掉[1] parallel() 行時，我的結果用時大約是 parallel() 的三倍。

並行流似乎是一個甜蜜的交易。你所需要做的就是將編程問題轉換為流，然後插入 parallel() 以加快速度。實際上，有時候這很容易。但遺憾的是，有許多陷阱。

- parallel() 不是靈丹妙藥

作為對流和並行流的不確定性的探索，讓我們看一個看似簡單的問題：對增長的數字序列進行求和。事實證明有大量的方式去實現它，並且我將冒險用計時器將它們進行比較 - 我會儘量小心，但我承認我可能會在計時程式碼執行時遇到許多基本陷阱之一。結果可能有一些缺陷（例如 JVM 沒有“熱身”），但我認為它仍然提供了一些有用的指示。

我將從一個計時方法 **timeTest()** 開始，它採用 **LongSupplier** ，測量 **getAsLong()** 呼叫的長度，將結果與 **checkValue** 進行比較並顯示結果。

請注意，一切都必須嚴格使用 **long** ；我花了一些時間發現隱蔽的溢位，然後才意識到在重要的地方錯過了 **long** 。

所有關於時間和記憶體的數字和討論都是指“我的機器”。你的經歷可能會有所不同。

```java
// concurrent/Summing.java
import java.util.stream.*;
import java.util.function.*;
import onjava.Timer;
public class Summing {
    static void timeTest(String id, long checkValue,    LongSupplier operation){
        System.out.print(id + ": ");
        Timer timer = new Timer();
        long result = operation.getAsLong();
        if(result == checkValue)
            System.out.println(timer.duration() + "ms");
        else
            System.out.format("result: %d%ncheckValue: %d%n", result, checkValue);
        }
    public static final int SZ = 100_000_000;
    // This even works:
    // public static final int SZ = 1_000_000_000;
    public static final long CHECK = (long)SZ * ((long)SZ + 1)/2; // Gauss's formula
    public static void main(String[] args){
        System.out.println(CHECK);
        timeTest("Sum Stream", CHECK, () ->
        LongStream.rangeClosed(0, SZ).sum());
        timeTest("Sum Stream Parallel", CHECK, () ->
        LongStream.rangeClosed(0, SZ).parallel().sum());
        timeTest("Sum Iterated", CHECK, () ->
        LongStream.iterate(0, i -> i + 1)
        .limit(SZ+1).sum());
        // Slower & runs out of memory above 1_000_000:
        // timeTest("Sum Iterated Parallel", CHECK, () ->
        //   LongStream.iterate(0, i -> i + 1)
        //     .parallel()
        //     .limit(SZ+1).sum());
    }
}
```

輸出結果：

```
5000000050000000
Sum Stream: 167ms
Sum Stream Parallel: 46ms
Sum Iterated: 284ms
```

**CHECK** 值是使用 Carl Friedrich Gauss（高斯）在 1700 年代後期還在上小學的時候建立的公式計算出來的.

 **main()**  的第一個版本使用直接生成 **Stream**  並呼叫 **sum()**  的方法。我們看到流的好處在於即使 SZ 為十億（1_000_000_000）程式也可以很好地處理而沒有溢位（為了讓程式執行得快一點，我使用了較小的數字）。使用 **parallel()**  的基本範圍操作明顯更快。

如果使用 **iterate()**  來生成序列，則減速是相當明顯的，可能是因為每次生成數字時都必須呼叫 lambda。但是如果我們嘗試並行化，當 **SZ** 超過一百萬時，結果不僅比非並行版本花費的時間更長，而且也會耗盡記憶體（在某些機器上）。當然，當你可以使用 **range()** 時，你不會使用 **iterate()**  ，但如果你生成的東西不是簡單的序列，你必須使用 **iterate()**  。應用 **parallel()**  是一個合理的嘗試，但會產生令人驚訝的結果。我們將在後面的部分中探討記憶體限制的原因，但我們可以對流並行演算法進行初步觀察：

- 流並行性將輸入資料分成多個部分，因此演算法可以應用於那些單獨的部分。
- 陣列分割成本低，分割均勻且對分割的大小有著完美的掌控。
- 鍊表沒有這些屬性;“分割”一個鍊表僅僅意味著把它分成“第一元素”和“其餘元素”，這相對無用。
- 無狀態生成器的行為類似於陣列;上面使用的 **range()**  就是無狀態的。
- 疊代生成器的行為類似於鍊表; **iterate()**  是一個疊代生成器。

現在讓我們嘗試通過在陣列中填儲值並對陣列求和來解決問題。因為陣列只分配了一次，所以我們不太可能遇到垃圾收集時序問題。

首先我們將嘗試一個充滿原始 **long** 的陣列：

```java
// concurrent/Summing2.java
// {ExcludeFromTravisCI}import java.util.*;
public class Summing2 {
    static long basicSum(long[] ia) {
        long sum = 0;
        int size = ia.length;
        for(int i = 0; i < size; i++)
            sum += ia[i];return sum;
    }
    // Approximate largest value of SZ before
    // running out of memory on mymachine:
    public static final int SZ = 20_000_000;
    public static final long CHECK = (long)SZ * ((long)SZ + 1)/2;
    public static void main(String[] args) {
        System.out.println(CHECK);
        long[] la = newlong[SZ+1];
        Arrays.parallelSetAll(la, i -> i);
        Summing.timeTest("Array Stream Sum", CHECK, () ->
        Arrays.stream(la).sum());
        Summing.timeTest("Parallel", CHECK, () ->
        Arrays.stream(la).parallel().sum());
        Summing.timeTest("Basic Sum", CHECK, () ->
        basicSum(la));// Destructive summation:
        Summing.timeTest("parallelPrefix", CHECK, () -> {
            Arrays.parallelPrefix(la, Long::sum);
        return la[la.length - 1];
        });
    }
}
```

輸出結果：

```
200000010000000
Array Stream
Sum: 104ms
Parallel: 81ms
Basic Sum: 106ms
parallelPrefix: 265ms
```

第一個限制是記憶體大小；因為陣列是預先分配的，所以我們不能建立幾乎與以前版本一樣大的任何東西。並行化可以加快速度，甚至比使用 **basicSum()**  循環更快。有趣的是， **Arrays.parallelPrefix()**  似乎實際上減慢了速度。但是，這些技術中的任何一種在其他條件下都可能更有用 - 這就是為什麼你不能做出任何確定性的聲明，除了“你必須嘗試一下”。

最後，考慮使用包裝類 **Long** 的效果：

```java
// concurrent/Summing3.java
// {ExcludeFromTravisCI}
import java.util.*;
public class Summing3 {
    static long basicSum(Long[] ia) {
        long sum = 0;
        int size = ia.length;
        for(int i = 0; i < size; i++)
            sum += ia[i];
            return sum;
    }
    // Approximate largest value of SZ before
    // running out of memory on my machine:
    public static final int SZ = 10_000_000;
    public static final long CHECK = (long)SZ * ((long)SZ + 1)/2;
    public static void main(String[] args) {
        System.out.println(CHECK);
        Long[] aL = newLong[SZ+1];
        Arrays.parallelSetAll(aL, i -> (long)i);
        Summing.timeTest("Long Array Stream Reduce", CHECK, () ->
        Arrays.stream(aL).reduce(0L, Long::sum));
        Summing.timeTest("Long Basic Sum", CHECK, () ->
        basicSum(aL));
        // Destructive summation:
        Summing.timeTest("Long parallelPrefix",CHECK, ()-> {
            Arrays.parallelPrefix(aL, Long::sum);
            return aL[aL.length - 1];
            });
    }
}
```

輸出結果：

```
50000005000000
Long Array
Stream Reduce: 1038ms
Long Basic
Sum: 21ms
Long parallelPrefix: 3616ms
```

現在可用的記憶體量大約減半，並且所有情況下所需的時間都會很長，除了 **basicSum()** ，它只是循環遍歷陣列。令人驚訝的是， **Arrays.parallelPrefix()**  比任何其他方法都要花費更長的時間。

我將 **parallel()**  版本分開了，因為在上面的程式中執行它導致了一個冗長的垃圾收集，扭曲了結果：

```java
// concurrent/Summing4.java
// {ExcludeFromTravisCI}
import java.util.*;
public class Summing4 {
    public static void main(String[] args) {
        System.out.println(Summing3.CHECK);
        Long[] aL = newLong[Summing3.SZ+1];
        Arrays.parallelSetAll(aL, i -> (long)i);
        Summing.timeTest("Long Parallel",
        Summing3.CHECK, () ->
        Arrays.stream(aL)
        .parallel()
        .reduce(0L,Long::sum));
    }
}
```

輸出結果：

```
50000005000000
Long Parallel: 1014ms
```

它比非 parallel() 版本略快，但並不顯著。

導致時間增加的一個重要原因是處理器記憶體快取。使用 **Summing2.java** 中的原始 **long** ，陣列 **la** 是連續的記憶體。處理器可以更容易地預測該陣列的使用，並使快取充滿下一個需要的陣列元素。訪問快取比訪問主記憶體快得多。似乎 **Long parallelPrefix**  計算受到影響，因為它為每個計算讀取兩個陣列元素，並將結果寫回到陣列中，並且每個都為 **Long** 生成一個超出快取的引用。

使用 **Summing3.java** 和 **Summing4.java** ，**aL** 是一個 **Long** 陣列，它不是一個連續的資料陣列，而是一個連續的 **Long** 物件引用陣列。儘管該陣列可能會在快取中出現，但指向的物件幾乎總是不在快取中。

這些範例使用不同的 SZ 值來顯示記憶體限制。

為了進行時間比較，以下是 SZ 設定為最小值 1000 萬的結果：

**Sum Stream: 69msSum
Stream Parallel: 18msSum
Iterated: 277ms
Array Stream Sum: 57ms
Parallel: 14ms
Basic Sum: 16ms
parallelPrefix: 28ms
Long Array Stream Reduce: 1046ms
Long Basic Sum: 21ms
Long parallelPrefix: 3287ms
Long Parallel: 1008ms** 

雖然 Java 8 的各種內建“並行”工具非常棒，但我認為它們被視為神奇的靈丹妙藥：“只需添加 parallel() 並且它會更快！” 我希望我已經開始表明情況並非所有都是如此，並且盲目地應用內建的“並行”操作有時甚至會使執行速度明顯變慢。

- parallel()/limit() 交點

使用 **parallel()**  時會有更複雜的問題。從其他語言中吸取的流機制被設計為大約是一個無限的流模型。如果你擁有有限數量的元素，則可以使用集合以及為有限大小的集合設計的關聯演算法。如果你使用無限流，則使用針對流最佳化的演算法。

Java 8 將兩者合併起來。例如，**Collections** 沒有內建的 **map()**  操作。在 **Collection** 和 **Map** 中唯一類似流的批次處理操作是 **forEach()**  。如果要執行 **map()**  和 **reduce()**  等操作，必須首先將 **Collection** 轉換為存在這些操作的 **Stream** :

```java
// concurrent/CollectionIntoStream.java
import onjava.*;
import java.util.*;
import java.util.stream.*;
public class CollectionIntoStream {
    public static void main(String[] args) {
    List<String> strings = Stream.generate(new Rand.String(5))
    .limit(10)
    .collect(Collectors.toList());
    strings.forEach(System.out::println);
    // Convert to a Stream for many more options:
    String result = strings.stream()
    .map(String::toUpperCase)
    .map(s -> s.substring(2))
    .reduce(":", (s1, s2) -> s1 + s2);
    System.out.println(result);
    }
}
```

輸出結果：

```
btpen
pccux
szgvg
meinn
eeloz
tdvew
cippc
ygpoa
lkljl
bynxt
:PENCUXGVGINNLOZVEWPPCPOALJLNXT
```

**Collection** 確實有一些批次處理操作，如 **removeAll()** ，**removeIf()**  和 **retainAll()**  ，但這些都是破壞性的操作。**ConcurrentHashMap** 對 **forEach** 和 **reduce** 操作有特別廣泛的支援。

在許多情況下，只在集合上呼叫 **stream()**  或者 **parallelStream()**  沒有問題。但是，有時將 **Stream** 與 **Collection** 混合會產生意想不到的結果。這是一個有趣的難題：

```java
// concurrent/ParallelStreamPuzzle.java
import java.util.*;
import java.util.function.*;
import java.util.stream.*;
public class ParallelStreamPuzzle {
    static class IntGenerator
    implements Supplier<Integer> {
        private int current = 0;
        @Override
        public Integer get() {
            return current++;
        }
    }
    public static void main(String[] args) {
        List<Integer> x = Stream.generate(new IntGenerator())
        .limit(10)
        .parallel()  // [1]
        .collect(Collectors.toList());
        System.out.println(x);
    }
}
/* Output:
[0, 1, 2, 3, 4, 5, 6, 7, 8, 9]
*/
```

如果[1] 注釋執行它，它會產生預期的：
**[0, 1, 2, 3, 4, 5, 6, 7, 8, 9]** 
每次。但是包含了 parallel()，它看起來像一個隨機數生成器，帶有輸出（從一次執行到下一次執行不同），如：
**[0, 3, 6, 8, 11, 14, 17, 20, 23, 26]** 
這樣一個簡單的程式怎麼會如此糟糕呢？讓我們考慮一下我們在這裡要實現的目標：“並行生成。”那意味著什麼？一堆執行緒都在從一個生成器取值，然後以某種方式選擇有限的結果集？程式碼看起來很簡單，但它變成了一個特別棘手的問題。

為了看到它，我們將添加一些儀器。由於我們正在處理執行緒，因此我們必須將任何跟蹤訊息捕獲到並發資料結構中。在這裡我使用 **ConcurrentLinkedDeque** ：

```java
// concurrent/ParallelStreamPuzzle2.java
import java.util.*;
import java.util.function.*;
import java.util.stream.*;
import java.util.concurrent.*;
import java.util.concurrent.atomic.*;
import java.nio.file.*;
public class ParallelStreamPuzzle2 {
    public static final Deque<String> TRACE =
    new ConcurrentLinkedDeque<>();
    static class
    IntGenerator implements Supplier<Integer> {
        private AtomicInteger current =
        new AtomicInteger();
        @Override
        public Integer get() {
            TRACE.add(current.get() + ": " +Thread.currentThread().getName());
            return current.getAndIncrement();
        }
    }
    public static void main(String[] args) throws Exception {
    List<Integer> x = Stream.generate(newIntGenerator())
    .limit(10)
    .parallel()
    .collect(Collectors.toList());
    System.out.println(x);
    Files.write(Paths.get("PSP2.txt"), TRACE);
    }
}
```

輸出結果：

```
[0, 1, 2, 3, 4, 5, 6, 7, 8, 9]
```

current 是使用執行緒安全的 **AtomicInteger**  類定義的，可以防止競爭條件；**parallel()**  允許多個執行緒呼叫 **get()** 。

在查看 **PSP2.txt**  .**IntGenerator.get()**  被呼叫 1024 次時，你可能會感到驚訝。

```
0: main
1: ForkJoinPool.commonPool-worker-1
2: ForkJoinPool.commonPool-worker-2
3: ForkJoinPool.commonPool-worker-2
4: ForkJoinPool.commonPool-worker-1
5: ForkJoinPool.commonPool-worker-1
6: ForkJoinPool.commonPool-worker-1
7: ForkJoinPool.commonPool-worker-1
8: ForkJoinPool.commonPool-worker-4
9: ForkJoinPool.commonPool-worker-4
10: ForkJoinPool.commonPool-worker-4
11: main
12: main
13: main
14: main
15: main...10
17: ForkJoinPool.commonPool-worker-110
18: ForkJoinPool.commonPool-worker-610
19: ForkJoinPool.commonPool-worker-610
20: ForkJoinPool.commonPool-worker-110
21: ForkJoinPool.commonPool-worker-110
22: ForkJoinPool.commonPool-worker-110
23: ForkJoinPool.commonPool-worker-1
```

這個塊大小似乎是內部實現的一部分（嘗試使用`limit()` 的不同參數來查看不同的塊大小）。將`parallel()`與`limit()`結合使用可以預取一串值，作為流輸出。

試著想像一下這裡發生了什麼事：一個流抽象出無限序列，按需生成。當你要求它並行產生流時，你要求所有這些執行緒儘可能地呼叫`get()`。添加`limit()`，你說“只需要這些。”基本上，當你為了隨機輸出而選擇將`parallel()`與`limit()`結合使用時，這種方法可能對你正在解決的問題有效。但是當你這樣做時，你必須明白。這是一個僅限專家的功能，而不是要爭辯說“Java 弄錯了”。

什麼是更合理的方法來解決問題？好吧，如果你想生成一個 int 流，你可以使用 IntStream.range()，如下所示：

```java
// concurrent/ParallelStreamPuzzle3.java
// {VisuallyInspectOutput}
import java.util.*;
import java.util.stream.*;
public class ParallelStreamPuzzle3 {
    public static void main(String[] args) {
    List<Integer> x = IntStream.range(0, 30)
        .peek(e -> System.out.println(e + ": " +Thread.currentThread()
        .getName()))
        .limit(10)
        .parallel()
        .boxed()
        .collect(Collectors.toList());
        System.out.println(x);
    }
}
```

輸出結果：

```
8: main
6: ForkJoinPool.commonPool-worker-5
3: ForkJoinPool.commonPool-worker-7
5: ForkJoinPool.commonPool-worker-5
1: ForkJoinPool.commonPool-worker-3
2: ForkJoinPool.commonPool-worker-6
4: ForkJoinPool.commonPool-worker-1
0: ForkJoinPool.commonPool-worker-4
7: ForkJoinPool.commonPool-worker-1
9: ForkJoinPool.commonPool-worker-2
[0, 1, 2, 3, 4, 5, 6, 7, 8, 9]
```

為了表明 **parallel()** 確實有效，我添加了一個對 **peek()**  的呼叫，這是一個主要用於除錯的流函數：它從流中提取一個值並執行某些操作但不影響從流向下傳遞的元素。注意這會干擾執行緒行為，但我只是嘗試在這裡做一些事情，而不是實際除錯任何東西。

你還可以看到 **boxed()**  的添加，它接受 **int** 流並將其轉換為 **Integer** 流。

現在我們得到多個執行緒產生不同的值，但它只產生 10 個請求的值，而不是 1024 個產生 10 個值。

它更快嗎？一個更好的問題是：什麼時候開始有意義？當然不是這麼小的一套；上下文切換的代價遠遠超過並行性的任何加速。很難想像什麼時候用並行生成一個簡單的數字序列會有意義。如果你要生成的東西需要很高的成本，它可能有意義 - 但這都是猜測。只有通過測試我們才能知道用並行是否有效。記住這句格言：“首先使它工作，然後使它更快地工作 - 只有當你必須這樣做時。”將 **parallel()**  和 **limit()**  結合使用僅供專家操作（把話說在前面，我不認為自己是這裡的專家）。

- 並行流只看起來很容易

實際上，在許多情況下，並行流確實可以毫不費力地更快地產生結果。但正如你所見，僅僅將 **parallel()**  加到你的 Stream 操作上並不一定是安全的事情。在使用 **parallel()**  之前，你必須了解並行性如何幫助或損害你的操作。一個基本認知錯誤就是認為使用並行性總是一個好主意。事實上並不是。Stream 意味著你不需要重寫所有程式碼便可以並行執行它。但是流的出現並不意味著你可以不用理解並行的原理以及不用考慮並行是否真的有助於實現你的目標。

## 建立和執行任務

如果無法透過並行流實現並發，則必須建立並執行自己的任務。稍後你將看到執行任務的理想 Java 8 方法是 CompletableFuture，但我們將使用更基本的工具介紹概念。

Java 並發的歷史始於非常原始和有問題的機制，並且充滿了各種嘗試的改進。這些主要歸入附錄：[低級並發 (Appendix: Low-Level Concurrency)](./Appendix-Low-Level-Concurrency.md)。在這裡，我們將展示一個規範形式，表示建立和執行任務的最簡單，最好的方法。與並發中的所有內容一樣，存在各種變體，但這些變體要嘛降級到該附錄，要嘛超出本書的範圍。

- Tasks and Executors

在 Java 的早期版本中，你透過直接建立自己的 Thread 物件來使用執行緒，甚至將它們子類化以建立你自己的特定“任務執行緒”物件。你手動呼叫了建構子並自己啟動了執行緒。

建立所有這些執行緒的開銷變得非常重要，現在不鼓勵採用手動操作方法。在 Java 5 中，添加了類來為你處理執行緒池。你可以將任務建立為單獨的類型，然後將其交給 ExecutorService 以執行該任務，而不是為每種不同類型的任務建立新的 Thread 子類型。ExecutorService 為你管理執行緒，並且在執行任務後重新循環執行緒而不是丟棄執行緒。

首先，我們將建立一個幾乎不執行任務的任務。它“sleep”（暫停執行）100 毫秒，顯示其標識符和正在執行任務的執行緒的名稱，然後完成：

```java
// concurrent/NapTask.java
import onjava.Nap;
public class NapTask implements Runnable {
    final int id;
    public NapTask(int id) {
        this.id = id;
        }
    @Override
    public void run() {
        new Nap(0.1);// Seconds
        System.out.println(this + " "+
            Thread.currentThread().getName());
        }
    @Override
    public String toString() {
        return"NapTask[" + id + "]";
    }
}
```

這只是一個 **Runnable** ：一個包含 **run()**  方法的類。它沒有包含實際執行任務的機制。我們使用 **Nap** 類中的“sleep”：

```java
// onjava/Nap.java
package onjava;
import java.util.concurrent.*;
public class Nap {
    public Nap(double t) { // Seconds
        try {
            TimeUnit.MILLISECONDS.sleep((int)(1000 * t));
        } catch(InterruptedException e){
            throw new RuntimeException(e);
        }
    }
    public Nap(double t, String msg) {
        this(t);
        System.out.println(msg);
    }
}
```
為了消除異常處理的視覺干擾，這被定義為實用程式。第二個建構子在超時時顯示一條消息

對 **TimeUnit.MILLISECONDS.sleep()**  的呼叫獲取“目前執行緒”並在參數中將其置於休眠狀態，這意味著該執行緒被掛斷。這並不意味著底層處理器停止。作業系統將其切換到其他任務，例如在你的電腦上執行另一個視窗。OS 任務管理器定期檢查 **sleep()**  是否超時。當它執行時，執行緒被“喚醒”並給予更多處理時間。

你可以看到 **sleep()**  拋出一個受檢的 **InterruptedException** ;這是原始 Java 設計中的一個工件，它透過突然斷開它們來終止任務。因為它往往會產生不穩定的狀態，所以後來不鼓勵終止。但是，我們必須在需要或仍然發生終止的情況下捕獲異常。

要執行任務，我們將從最簡單的方法--SingleThreadExecutor 開始:

```java
//concurrent/SingleThreadExecutor.java
import java.util.concurrent.*;
import java.util.stream.*;
import onjava.*;
public class SingleThreadExecutor {
    public static void main(String[] args) {
        ExecutorService exec =
            Executors.newSingleThreadExecutor();
        IntStream.range(0, 10)
            .mapToObj(NapTask::new)
            .forEach(exec::execute);
        System.out.println("All tasks submitted");
        exec.shutdown();
        while(!exec.isTerminated()) {
            System.out.println(
            Thread.currentThread().getName()+
            " awaiting termination");
            new Nap(0.1);
        }
    }
}
```

輸出結果：

```
All tasks submitted
main awaiting termination
main awaiting termination
NapTask[0] pool-1-thread-1
main awaiting termination
NapTask[1] pool-1-thread-1
main awaiting termination
NapTask[2] pool-1-thread-1
main awaiting termination
NapTask[3] pool-1-thread-1
main awaiting termination
NapTask[4] pool-1-thread-1
main awaiting termination
NapTask[5] pool-1-thread-1
main awaiting termination
NapTask[6] pool-1-thread-1
main awaiting termination
NapTask[7] pool-1-thread-1
main awaiting termination
NapTask[8] pool-1-thread-1
main awaiting termination
NapTask[9] pool-1-thread-1
```

首先請注意，沒有 **SingleThreadExecutor** 類。**newSingleThreadExecutor()**  是 **Executors**  中的一個工廠方法，它建立特定類型的 **ExecutorService** [^4]

我建立了十個 NapTasks 並將它們提交給 ExecutorService，這意味著它們開始自己執行。然而，在此期間，main() 繼續做事。當我執行 callexec.shutdown() 時，它告訴 ExecutorService 完成已經提交的任務，但不接受任何新任務。此時，這些任務仍然在執行，因此我們必須等到它們在退出 main() 之前完成。這是通過檢查 exec.isTerminated() 來實現的，這在所有任務完成後變為 true。

請注意，main() 中執行緒的名稱是 main，並且只有一個其他執行緒 pool-1-thread-1。此外，交錯輸出顯示兩個執行緒確實同時執行。

如果你只是呼叫 exec.shutdown()，程式將完成所有任務。也就是說，不需要 **while(!exec.isTerminated())** 。

```java
// concurrent/SingleThreadExecutor2.java
import java.util.concurrent.*;
import java.util.stream.*;
public class SingleThreadExecutor2 {
    public static void main(String[] args)throws InterruptedException {
        ExecutorService exec
        =Executors.newSingleThreadExecutor();
        IntStream.range(0, 10)
            .mapToObj(NapTask::new)
            .forEach(exec::execute);
        exec.shutdown();
    }
}
```

輸出結果：

```
NapTask[0] pool-1-thread-1
NapTask[1] pool-1-thread-1
NapTask[2] pool-1-thread-1
NapTask[3] pool-1-thread-1
NapTask[4] pool-1-thread-1
NapTask[5] pool-1-thread-1
NapTask[6] pool-1-thread-1
NapTask[7] pool-1-thread-1
NapTask[8] pool-1-thread-1
NapTask[9] pool-1-thread-1
```

一旦你 callexec.shutdown()，嘗試提交新任務將拋出 RejectedExecutionException。

```java
// concurrent/MoreTasksAfterShutdown.java
import java.util.concurrent.*;
public class MoreTasksAfterShutdown {
    public static void main(String[] args) {
        ExecutorService exec
        =Executors.newSingleThreadExecutor();
        exec.execute(newNapTask(1));
        exec.shutdown();
        try {
            exec.execute(newNapTask(99));
        } catch(RejectedExecutionException e) {
            System.out.println(e);
        }
    }
}
```

輸出結果：

```
java.util.concurrent.RejectedExecutionException: TaskNapTask[99] rejected from java.util.concurrent.ThreadPoolExecutor@4e25154f[Shutting down, pool size = 1,active threads = 1, queued tasks = 0, completed tasks =0]NapTask[1] pool-1-thread-1
```

**exec.shutdown()**  的替代方法是 **exec.shutdownNow()**  ，它除了不接受新任務外，還會嘗試通過中斷任務來停止任何目前正在執行的任務。同樣，中斷是錯誤的，容易出錯並且不鼓勵。

- 使用更多執行緒

使用執行緒的重點是（幾乎總是）更快地完成任務，那麼我們為什麼要限制自己使用 SingleThreadExecutor 呢？查看執行 **Executors** 的 Javadoc，你將看到更多選項。例如 CachedThreadPool：

```java
// concurrent/CachedThreadPool.java
import java.util.concurrent.*;
import java.util.stream.*;
public class CachedThreadPool {
    public static void main(String[] args) {
        ExecutorService exec
        =Executors.newCachedThreadPool();
        IntStream.range(0, 10)
        .mapToObj(NapTask::new)
        .forEach(exec::execute);
        exec.shutdown();
    }
}
```

輸出結果：

```
NapTask[7] pool-1-thread-8
NapTask[4] pool-1-thread-5
NapTask[1] pool-1-thread-2
NapTask[3] pool-1-thread-4
NapTask[0] pool-1-thread-1
NapTask[8] pool-1-thread-9
NapTask[2] pool-1-thread-3
NapTask[9] pool-1-thread-10
NapTask[6] pool-1-thread-7
NapTask[5] pool-1-thread-6
```

當你執行這個程式時，你會發現它完成得更快。這是有道理的，每個任務都有自己的執行緒，所以它們都並行執行，而不是使用相同的執行緒來順序執行每個任務。這似乎沒毛病，很難理解為什麼有人會使用 SingleThreadExecutor。

要理解這個問題，我們需要一個更複雜的任務：

```java
// concurrent/InterferingTask.java
public class InterferingTask implements Runnable {
    final int id;
    private static Integer val = 0;
    public InterferingTask(int id) {
        this.id = id;
    }
    @Override
    public void run() {
        for(int i = 0; i < 100; i++)
        val++;
    System.out.println(id + " "+
        Thread.currentThread().getName() + " " + val);
    }
}

```

每個任務增加 val 一百次。這似乎很簡單。讓我們用 CachedThreadPool 嘗試一下：

```java
// concurrent/CachedThreadPool2.java
import java.util.concurrent.*;
import java.util.stream.*;
public class CachedThreadPool2 {
    public static void main(String[] args) {
    ExecutorService exec
    =Executors.newCachedThreadPool();
    IntStream.range(0, 10)
    .mapToObj(InterferingTask::new)
    .forEach(exec::execute);
    exec.shutdown();
    }
}
```

輸出結果：

```
0 pool-1-thread-1 200
1 pool-1-thread-2 200
4 pool-1-thread-5 300
5 pool-1-thread-6 400
8 pool-1-thread-9 500
9 pool-1-thread-10 600
2 pool-1-thread-3 700
7 pool-1-thread-8 800
3 pool-1-thread-4 900
6 pool-1-thread-7 1000
```

輸出不是我們所期望的，並且從一次執行到下一次執行會有所不同。問題是所有的任務都試圖寫入 val 的單個實例，並且他們正在踩著彼此的腳趾。我們稱這樣的類是執行緒不安全的。讓我們看看 SingleThreadExecutor 會發生什麼事：

```java
// concurrent/SingleThreadExecutor3.java
import java.util.concurrent.*;
import java.util.stream.*;
public class SingleThreadExecutor3 {
    public static void main(String[] args)throws InterruptedException {
        ExecutorService exec
        =Executors.newSingleThreadExecutor();
        IntStream.range(0, 10)
        .mapToObj(InterferingTask::new)
        .forEach(exec::execute);
        exec.shutdown();
    }
}
```

輸出結果：

```
0 pool-1-thread-1 100
1 pool-1-thread-1 200
2 pool-1-thread-1 300
3 pool-1-thread-1 400
4 pool-1-thread-1 500
5 pool-1-thread-1 600
6 pool-1-thread-1 700
7 pool-1-thread-1 800
8 pool-1-thread-1 900
9 pool-1-thread-1 1000
```

現在我們每次都得到一致的結果，儘管 **InterferingTask** 缺乏執行緒安全性。這是 SingleThreadExecutor 的主要好處 - 因為它一次執行一個任務，這些任務不會相互干擾，因此強加了執行緒安全性。這種現象稱為執行緒封閉，因為在單執行緒上執行任務限制了它們的影響。執行緒封閉限制了加速，但可以節省很多困難的除錯和重寫。

- 產生結果

因為 **InterferingTask** 是一個 **Runnable** ，它沒有返回值，因此只能使用副作用產生結果 - 操縱緩衝值而不是返回結果。副作用是並發編程中的主要問題之一，因為我們看到了 **CachedThreadPool2.java** 。**InterferingTask** 中的 **val** 被稱為可變共享狀態，這就是問題所在：多個任務同時修改同一個變數會產生競爭。結果取決於首先在終點線上執行哪個任務，並修改變數（以及其他可能性的各種變化）。

避免競爭條件的最好方法是避免可變的共享狀態。我們可以稱之為自私的孩子原則：什麼都不分享。

使用 **InterferingTask** ，最好刪除副作用並返回任務結果。為此，我們建立 **Callable** 而不是 **Runnable** ：

```java
// concurrent/CountingTask.java
import java.util.concurrent.*;
public class CountingTask implements Callable<Integer> {
    final int id;
    public CountingTask(int id) { this.id = id; }
    @Override
    public Integer call() {
    Integer val = 0;
    for(int i = 0; i < 100; i++)
        val++;
    System.out.println(id + " " +
        Thread.currentThread().getName() + " " + val);
    return val;
    }
}

```

**call() 完全獨立於所有其他 CountingTasks 生成其結果**，這意味著沒有可變的共享狀態

**ExecutorService** 允許你使用 **invokeAll()** 啟動集合中的每個 Callable：

```java
// concurrent/CachedThreadPool3.java
import java.util.*;
import java.util.concurrent.*;
import java.util.stream.*;
public class CachedThreadPool3 {
    public static Integer extractResult(Future<Integer> f) {
        try {
            return f.get();
        } catch(Exception e) {
            throw new RuntimeException(e);
        }
    }
    public static void main(String[] args)throws InterruptedException {
    ExecutorService exec =
    Executors.newCachedThreadPool();
    List<CountingTask> tasks =
        IntStream.range(0, 10)
            .mapToObj(CountingTask::new)
            .collect(Collectors.toList());
        List<Future<Integer>> futures =
            exec.invokeAll(tasks);
        Integer sum = futures.stream()
            .map(CachedThreadPool3::extractResult)
            .reduce(0, Integer::sum);
        System.out.println("sum = " + sum);
        exec.shutdown();
    }
}
```

輸出結果：

```
1 pool-1-thread-2 100
0 pool-1-thread-1 100
4 pool-1-thread-5 100
5 pool-1-thread-6 100
8 pool-1-thread-9 100
9 pool-1-thread-10 100
2 pool-1-thread-3 100
3 pool-1-thread-4 100
6 pool-1-thread-7 100
7 pool-1-thread-8 100
sum = 1000

```

只有在所有任務完成後，**invokeAll()** 才會返回一個 **Future** 列表，每個任務一個 **Future** 。**Future** 是 Java 5 中引入的機制，允許你提交任務而無需等待它完成。在這裡，我們使用 **ExecutorService.submit()** :

```java
// concurrent/Futures.java
import java.util.*;
import java.util.concurrent.*;
import java.util.stream.*;
public class Futures {
    public static void main(String[] args)throws InterruptedException, ExecutionException {
    ExecutorService exec
        =Executors.newSingleThreadExecutor();
    Future<Integer> f =
        exec.submit(newCountingTask(99));
    System.out.println(f.get()); // [1]
    exec.shutdown();
    }
}
```

輸出結果：

```
99 pool-1-thread-1 100
100
```

- [1] 當你的任務在尚未完成的 **Future** 上呼叫 **get()** 時，呼叫會阻塞（等待）直到結果可用。

但這意味著，在 **CachedThreadPool3.java** 中，**Future** 似乎是多餘的，因為 **invokeAll()** 甚至在所有任務完成之前都不會返回。但是，這裡的 Future 並不用於延遲結果，而是用於捕獲任何可能發生的異常。

還要注意在 **CachedThreadPool3.java.get()** 中拋出異常，因此 **extractResult()** 在 Stream 中執行此提取。

因為當你呼叫 **get()** 時，**Future** 會阻塞，所以它只能解決等待任務完成才暴露問題。最終，**Futures** 被認為是一種無效的解決方案，現在不鼓勵，我們推薦 Java 8 的 **CompletableFuture** ，這將在本章後面探討。當然，你仍會在遺留庫中遇到 Futures。

我們可以使用並行 Stream 以更簡單，更優雅的方式解決這個問題:

```java
// concurrent/CountingStream.java
// {VisuallyInspectOutput}
import java.util.*;
import java.util.concurrent.*;
import java.util.stream.*;
public class CountingStream {
    public static void main(String[] args) {
        System.out.println(
            IntStream.range(0, 10)
                .parallel()
                .mapToObj(CountingTask::new)
                .map(ct -> ct.call())
                .reduce(0, Integer::sum));
    }
}
```

輸出結果：

```
1 ForkJoinPool.commonPool-worker-3 100
8 ForkJoinPool.commonPool-worker-2 100
0 ForkJoinPool.commonPool-worker-6 100
2 ForkJoinPool.commonPool-worker-1 100
4 ForkJoinPool.commonPool-worker-5 100
9 ForkJoinPool.commonPool-worker-7 100
6 main 100
7 ForkJoinPool.commonPool-worker-4 100
5 ForkJoinPool.commonPool-worker-2 100
3 ForkJoinPool.commonPool-worker-3 100
1000
```

這不僅更容易理解，而且我們需要做的就是將 `parallel()` 插入到其他順序操作中，然後一切都在同時執行。

- Lambda 和方法引用作為任務

在 **java8**  有了 **lambdas**  和方法引用，你不需要受限於只能使用  **Runnable**  和 **Callable**  。因為 java8 的 **lambdas**  和方法引用可以透過匹配方法簽名來使用（即，它支援結構一致性），所以我們可以將非 **Runnable**  或 **Callable**  的參數傳遞給 `ExecutorService` : 

```java
// concurrent/LambdasAndMethodReferences.java
import java.util.concurrent.*;
class NotRunnable {
    public void go() {
        System.out.println("NotRunnable");
    }
}
class NotCallable {
    public Integer get() {
        System.out.println("NotCallable");
        return 1;
    }
}
public class LambdasAndMethodReferences {
    public static void main(String[] args)throws InterruptedException {
    ExecutorService exec =
        Executors.newCachedThreadPool();
    exec.submit(() -> System.out.println("Lambda1"));
    exec.submit(new NotRunnable()::go);
    exec.submit(() -> {
        System.out.println("Lambda2");
        return 1;
    });
    exec.submit(new NotCallable()::get);
    exec.shutdown();
    }
}
```

輸出結果：

```
Lambda1
NotCallable
NotRunnable
Lambda2

```

這裡，前兩個 **submit()** 呼叫可以改為呼叫 **execute()** 。所有 **submit()** 呼叫都返回 **Futures** ，你可以在後兩次呼叫的情況下提取結果。

<!-- Terminating Long-Running Tasks -->
## 終止耗時任務

並發程式通常使用長時間執行的任務。可呼叫任務在完成時返回值;雖然這給它一個有限的壽命，但仍然可能很長。可執行的任務有時被設定為永遠執行的後台行程。你經常需要一種方法在正常完成之前停止 **Runnable** 和 **Callable** 任務，例如當你關閉程式時。

最初的 Java 設計提供了中斷執行任務的機制（為了向後相容，仍然存在）;中斷機制包括阻塞問題。中斷任務既亂又複雜，因為你必須了解可能發生中斷的所有可能狀態，以及可能導致的資料遺失。使用中斷被視為反對模式，但我們仍然被迫接受。

InterruptedException，因為設計的向後相容性殘留。

任務終止的最佳方法是設定任務週期性檢查的標誌。然後任務可以透過自己的 shutdown 行程並正常終止。不是在任務中隨機關閉執行緒，而是要求任務在到達了一個較好時自行終止。這總是產生比中斷更好的結果，以及更容易理解的更合理的程式碼。

以這種方式終止任務聽起來很簡單：設定任務可以看到的 **boolean**  flag。編寫任務，以便定期檢查標誌並執行正常終止。這實際上就是你所做的，但是有一個複雜的問題：我們的舊剋星，共同的可變狀態。如果該標誌可以被另一個任務操縱，則存在碰撞可能性。

在研究 Java 文獻時，你會發現很多解決這個問題的方法，經常使用 **volatile** 關鍵字。我們將使用更簡單的技術並避免所有易變的參數，這些都在[附錄：低級並發 ](./Appendix-Low-Level-Concurrency.md) 中有所涉及。

Java 5 引入了 **Atomic** 類，它提供了一組可以使用的類型，而不必擔心並發問題。我們將添加 **AtomicBoolean** 標誌，告訴任務清理自己並退出。

```java
// concurrent/QuittableTask.java
import java.util.concurrent.atomic.AtomicBoolean;import onjava.Nap;
public class QuittableTask implements Runnable {
    final int id;
    public QuittableTask(int id) {
        this.id = id;
    }
    private AtomicBoolean running =
        new AtomicBoolean(true);
    public void quit() {
        running.set(false);
    }
    @Override
    public void run() {
        while(running.get())         // [1]
            new Nap(0.1);
        System.out.print(id + " ");  // [2]
    }
}

```

雖然多個任務可以在同一個實例上成功呼叫 **quit()** ，但是 **AtomicBoolean** 可以防止多個任務同時實際修改 **running** ，從而使 **quit()** 方法成為執行緒安全的。

- [1]:只要執行標誌為 true，此任務的 run() 方法將繼續。
- [2]: 顯示僅在任務退出時發生。

需要 **running AtomicBoolean** 證明編寫 Java program 並發時最基本的困難之一是，如果 **running** 是一個普通的布林值，你可能無法在處理程序中看到問題。實際上，在這個例子中，你可能永遠不會有任何問題 - 但是程式碼仍然是不安全的。編寫表明該問題的測試可能很困難或不可能。因此，你沒有任何回饋來告訴你已經做錯了。通常，你編寫執行緒安全程式碼的唯一方法就是透過了解事情可能出錯的所有細微之處。

作為測試，我們將啟動很多 QuittableTasks 然後關閉它們。嘗試使用較大的 COUNT 值

```java
// concurrent/QuittingTasks.java
import java.util.*;
import java.util.stream.*;
import java.util.concurrent.*;
import onjava.Nap;
public class QuittingTasks {
    public static final int COUNT = 150;
    public static void main(String[] args) {
    ExecutorService es =
    Executors.newCachedThreadPool();
    List<QuittableTask> tasks =
    IntStream.range(1, COUNT)
        .mapToObj(QuittableTask::new)
        .peek(qt -> es.execute(qt))
        .collect(Collectors.toList());
    new Nap(1);
    tasks.forEach(QuittableTask::quit);    es.shutdown();
    }
}
```

輸出結果：

```
24 27 31 8 11 7 19 12 16 4 23 3 28 32 15 20 63 60 68 6764 39 47 52 51 55 40 43 48 59 44 56 36 35 71 72 83 10396 92 88 99 100 87 91 79 75 84 76 115 108 112 104 107111 95 80 147 120 127 119 123 144 143 116 132 124 128
136 131 135 139 148 140 2 126 6 5 1 18 129 17 14 13 2122 9 10 30 33 58 37 125 26 34 133 145 78 137 141 138 6274 142 86 65 73 146 70 42 149 121 110 134 105 82 117106 113 122 45 114 118 38 50 29 90 101 89 57 53 94 4161 66 130 69 77 81 85 93 25 102 54 109 98 49 46 97
```

我使用 **peek()** 將 **QuittableTasks** 傳遞給 **ExecutorService** ，然後將這些任務收集到 **List.main()** 中，只要任何任務仍在執行，就會阻止程式退出。即使為每個任務按順序呼叫 quit() 方法，任務也不會按照它們建立的順序關閉。獨立執行的任務不會確定性地響應訊號。

## CompletableFuture 類

作為介紹，這裡是使用 CompletableFutures 在 QuittingTasks.java 中：

```java
// concurrent/QuittingCompletable.java
import java.util.*;
import java.util.stream.*;
import java.util.concurrent.*;
import onjava.Nap;
public class QuittingCompletable {
    public static void main(String[] args) {
    List<QuittableTask> tasks =
        IntStream.range(1, QuittingTasks.COUNT)
            .mapToObj(QuittableTask::new)
            .collect(Collectors.toList());
        List<CompletableFuture<Void>> cfutures =
        tasks.stream()
            .map(CompletableFuture::runAsync)
            .collect(Collectors.toList());
        new Nap(1);
        tasks.forEach(QuittableTask::quit);
        cfutures.forEach(CompletableFuture::join);
    }
}
```

輸出結果：

```
7 8 9 10 11 12 13 14 15 16 17 18 19 20 21 22 23 24 2526 27 28 29 30 31 32 33 34 6 35 4 38 39 40 41 42 43 4445 46 47 48 49 50 51 52 53 54 55 56 57 58 59 60 61 6263 64 65 66 67 68 69 70 71 72 73 74 75 76 77 78 79 8081 82 83 84 85 86 87 88 89 90 91 92 93 94 95 96 97 9899 100 101 102 103 104 105 106 107 108 109 110 111 1121 113 114 116 117 118 119 120 121 122 123 124 125 126127 128 129 130 131 132 133 134 135 136 137 138 139 140141 142 143 144 145 146 147 148 149 5 115 37 36 2 3
```

任務是一個 `List<QuittableTask>`，就像在 `QuittingTasks.java` 中一樣，但是在這個例子中，沒有 `peek()` 將每個 `QuittableTask` 提交給 `ExecutorService`。相反，在建立 `cfutures` 期間，每個任務都交給 `CompletableFuture::runAsync`。這執行 `VerifyTask.run()` 並返回 `CompletableFuture<Void>` 。因為 `run()` 不返回任何內容，所以在這種情況下我只使用 `CompletableFuture` 呼叫 `join()` 來等待它完成。

在本例中需要注意的重要一點是，執行任務不需要使用 `ExecutorService`。而是直接交給 `CompletableFuture` 管理 (不過你可以向它提供自己定義的 `ExectorService`)。您也不需要呼叫 `shutdown()`;事實上，除非你像我在這裡所做的那樣顯式地呼叫 `join()`，否則程式將儘快退出，而不必等待任務完成。

這個例子只是一個起點。你很快就會看到 `ComplempleFuture` 能夠做得更多。

### 基本用法

這是一個帶有靜態方法 **work()** 的類，它對該類的物件執行某些工作：

```java
// concurrent/Machina.java
import onjava.Nap;
public class Machina {
    public enum State {
        START, ONE, TWO, THREE, END;
        State step() {
            if(equals(END))
            return END;
          return values()[ordinal() + 1];
        }
    }
    private State state = State.START;
    private final int id;
    public Machina(int id) {
        this.id = id;
    }
    public static Machina work(Machina m) {
        if(!m.state.equals(State.END)){
            new Nap(0.1);
            m.state = m.state.step();
        }
        System.out.println(m);
        return m;
    }
    @Override
    public String toString() {
        return"Machina" + id + ": " + (state.equals(State.END)? "complete" : state);
    }
}

```

這是一個有限狀態機，一個微不足道的機器，因為它沒有分支......它只是從頭到尾遍歷一條路徑。**work()** 方法將機器從一個狀態移動到下一個狀態，並且需要 100 毫秒才能完成“工作”。

**CompletableFuture** 可以被用來做的一件事是, 使用 **completedFuture()** 將它感興趣的物件進行包裝。

```java
// concurrent/CompletedMachina.java
import java.util.concurrent.*;
public class CompletedMachina {
    public static void main(String[] args) {
        CompletableFuture<Machina> cf =
        CompletableFuture.completedFuture(
            new Machina(0));
        try {
            Machina m = cf.get();  // Doesn't block
        } catch(InterruptedException |
            ExecutionException e) {
        throw new RuntimeException(e);
        }
    }
}
```

**completedFuture()** 建立一個“已經完成”的 **CompletableFuture** 。對這樣一個未來做的唯一有用的事情是 **get()** 裡面的物件，所以這看起來似乎沒有用。注意 **CompletableFuture** 被輸入到它包含的物件。這個很重要。

通常，**get()** 在等待結果時阻塞呼叫執行緒。此塊可以透過 **InterruptedException** 或 **ExecutionException** 中斷。在這種情況下，阻止永遠不會發生，因為 **CompletableFuture** 已經完成，所以結果立即可用。

當我們將 **handle()** 包裝在 **CompletableFuture** 中時，發現我們可以在 **CompletableFuture** 上添加操作來處理所包含的物件，使得事情變得更加有趣：

```java
// concurrent/CompletableApply.java
import java.util.concurrent.*;
public class CompletableApply {
    public static void main(String[] args) {
        CompletableFuture<Machina> cf =
        CompletableFuture.completedFuture(
            new Machina(0));
        CompletableFuture<Machina> cf2 =
            cf.thenApply(Machina::work);
        CompletableFuture<Machina> cf3 =
            cf2.thenApply(Machina::work);
        CompletableFuture<Machina> cf4 =
            cf3.thenApply(Machina::work);
        CompletableFuture<Machina> cf5 =
            cf4.thenApply(Machina::work);
    }
}
```

**輸出結果**：

```
Machina0: ONE
Machina0: TWO
Machina0: THREE
Machina0: complete
```

`thenApply()` 應用一個接收輸入並產生輸出的函數。在本例中，`work()` 函數產生的類型與它所接收的類型相同 （`Machina`），因此每個 `CompletableFuture`添加的操作的返回類型都為 `Machina`，但是 (類似於流中的 `map()` ) 函數也可以返回不同的類型，這將體現在返回類型上。

你可以在此處看到有關 **CompletableFutures** 的重要訊息：它們會在你執行操作時自動解包並重新包裝它們所攜帶的物件。這使得編寫和理解程式碼變得更加簡單， 而不會在陷入在麻煩的細節中。

我們可以消除中間變數並將操作連結在一起，就像我們使用 Streams 一樣：

```java
// concurrent/CompletableApplyChained.javaimport java.util.concurrent.*;
import onjava.Timer;
public class CompletableApplyChained {
    public static void main(String[] args) {
        Timer timer = new Timer();
        CompletableFuture<Machina> cf =
        CompletableFuture.completedFuture(
            new Machina(0))
                  .thenApply(Machina::work)
                  .thenApply(Machina::work)
                  .thenApply(Machina::work)
                  .thenApply(Machina::work);
        System.out.println(timer.duration());
    }
}
```

輸出結果：

```
Machina0: ONE
Machina0: TWO
Machina0: THREE
Machina0: complete
514
```

這裡我們還添加了一個 `Timer`，它的功能在每一步都顯性地增加 100ms 等待時間之外，還將 `CompletableFuture` 內部 `thenApply` 帶來的額外開銷給體現出來了。 
**CompletableFutures**  的一個重要好處是它們鼓勵使用私有子類原則（不共享任何東西）。預設情況下，使用 **thenApply()**  來應用一個不對外通信的函數 - 它只需要一個參數並返回一個結果。這是函數式編程的基礎，並且它在併發特性方面非常有效[^5]。並行流和 `ComplempleFutures` 旨在支援這些原則。只要你不決定共享資料（共享非常容易導致意外發生）你就可以編寫出相對安全的並發程式。

回調 `thenApply()` 一旦開始一個操作，在完成所有任務之前，不會完成 **CompletableFuture**  的構建。雖然這有時很有用，但是開始所有任務通常更有價值，這樣就可以執行繼續前進並執行其他操作。我們可透過`thenApplyAsync()` 來實現此目的：

```java
// concurrent/CompletableApplyAsync.java
import java.util.concurrent.*;
import onjava.*;
public class CompletableApplyAsync {
    public static void main(String[] args) {
        Timer timer = new Timer();
        CompletableFuture<Machina> cf =
            CompletableFuture.completedFuture(
                new Machina(0))
                .thenApplyAsync(Machina::work)
                .thenApplyAsync(Machina::work)
                .thenApplyAsync(Machina::work)
                .thenApplyAsync(Machina::work);
            System.out.println(timer.duration());
            System.out.println(cf.join());
            System.out.println(timer.duration());
    }
}
```

輸出結果：

```
116
Machina0: ONE
Machina0: TWO
Machina0:THREE
Machina0: complete
Machina0: complete
552
```

同步呼叫 (我們通常使用的那種) 意味著：“當你完成工作時，才返回”，而非同步呼叫以意味著： “立刻返回並繼續後續工作”。 正如你所看到的，`cf` 的建立現在發生的更快。每次呼叫 `thenApplyAsync()` 都會立刻返回，因此可以進行下一次呼叫，整個呼叫鏈路完成速度比以前快得多。

事實上，如果沒有回調 `cf.join()` 方法，程式會在完成其工作之前退出。而 `cf.join()` 直到 cf 操作完成之前，阻止 `main()` 行程結束。我們還可以看出本範例大部分時間消耗在 `cf.join()` 這。

這種“立即返回”的非同步能力需要 `CompletableFuture` 庫進行一些秘密（`client` 無感）工作。特別是，它將你需要的操作鏈儲存為一組回調。當操作的第一個鏈路（後台操作）完成並返回時，第二個鏈路（後台操作）必須獲取生成的 `Machina` 並開始工作，以此類推！ 但這種非同步機制沒有我們可以透過程式呼叫堆疊控制的普通函數呼叫序列，它的呼叫鏈路順序會遺失，因此它使用一個函數地址來儲存的回調來解決這個問題。

幸運的是，這就是你需要了解的有關回調的全部訊息。程式設計師將這種人為製造的混亂稱為 callback hell（回調地獄）。透過非同步呼叫，`CompletableFuture` 幫你管理所有回調。 除非你知道你系統中的一些特定邏輯會導致某些改變，或許你更想使用非同步呼叫來實現程式。

- 其他操作

當你查看`CompletableFuture`的 `Javadoc` 時，你會看到它有很多方法，但這個方法的大部分來自不同操作的變體。例如，有 `thenApply()`，`thenApplyAsync()` 和第二種形式的 `thenApplyAsync()`，它們使用 `Executor` 來執行任務 (在本書中，我們忽略了 `Executor` 選項)。

下面的範例展示了所有"基本"操作，這些操作既不涉及組合兩個 `CompletableFuture`，也不涉及異常 (我們將在後面介紹)。首先，為了提供簡潔性和方便性，我們應該重用以下兩個實用程式:

```java
package onjava;
import java.util.concurrent.*;

public class CompletableUtilities {
  // Get and show value stored in a CF:
  public static void showr(CompletableFuture<?> c) {
    try {
      System.out.println(c.get());
    } catch(InterruptedException
            | ExecutionException e) {
      throw new RuntimeException(e);
    }
  }
  // For CF operations that have no value:
  public static void voidr(CompletableFuture<Void> c) {
    try {
      c.get(); // Returns void
    } catch(InterruptedException
            | ExecutionException e) {
      throw new RuntimeException(e);
    }
  }
}
```

`showr()` 在 `CompletableFuture<Integer>` 上呼叫 `get()`，並顯示結果，`try/catch` 兩個可能會出現的異常。

`voidr()` 是 `CompletableFuture<Void>` 的 `showr()` 版本，也就是說，`CompletableFutures` 只為任務完成或失敗時顯示訊息。

為簡單起見，下面的 `CompletableFutures` 只包裝整數。`cfi()` 是一個便利的方法，它把一個整數包裝在一個完整的 `CompletableFuture<Integer>` :

```java
// concurrent/CompletableOperations.java
import java.util.concurrent.*;
import static onjava.CompletableUtilities.*;

public class CompletableOperations {
    static CompletableFuture<Integer> cfi(int i) {
        return
                CompletableFuture.completedFuture(
                        Integer.valueOf(i));
    }

    public static void main(String[] args) {
        showr(cfi(1)); // Basic test
        voidr(cfi(2).runAsync(() ->
                System.out.println("runAsync")));
        voidr(cfi(3).thenRunAsync(() ->
                System.out.println("thenRunAsync")));
        voidr(CompletableFuture.runAsync(() ->
                System.out.println("runAsync is static")));
        showr(CompletableFuture.supplyAsync(() -> 99));
        voidr(cfi(4).thenAcceptAsync(i ->
                System.out.println("thenAcceptAsync: " + i)));
        showr(cfi(5).thenApplyAsync(i -> i + 42));
        showr(cfi(6).thenComposeAsync(i -> cfi(i + 99)));
        CompletableFuture<Integer> c = cfi(7);
        c.obtrudeValue(111);
        showr(c);
        showr(cfi(8).toCompletableFuture());
        c = new CompletableFuture<>();
        c.complete(9);
        showr(c);
        c = new CompletableFuture<>();
        c.cancel(true);
        System.out.println("cancelled: " +
                c.isCancelled());
        System.out.println("completed exceptionally: " +
                c.isCompletedExceptionally());
        System.out.println("done: " + c.isDone());
        System.out.println(c);
        c = new CompletableFuture<>();
        System.out.println(c.getNow(777));
        c = new CompletableFuture<>();
        c.thenApplyAsync(i -> i + 42)
                .thenApplyAsync(i -> i * 12);
        System.out.println("dependents: " +
                c.getNumberOfDependents());
        c.thenApplyAsync(i -> i / 2);
        System.out.println("dependents: " +
                c.getNumberOfDependents());
    }
}
```

**輸出結果** ：

```
1
runAsync
thenRunAsync
runAsync is static
99
thenAcceptAsync: 4
47
105
111
8
9
cancelled: true
completed exceptionally: true
done: true
java.util.concurrent.CompletableFuture@6d311334[Complet ed exceptionally]
777
dependents: 1
dependents: 2
```

- `main()` 包含一系列可由其 `int` 值引用的測試。
  - `cfi(1)` 示範了 `showr()` 正常工作。
  - `cfi(2)` 是呼叫 `runAsync()` 的範例。由於 `Runnable` 不產生返回值，因此使用了返回 `CompletableFuture <Void>` 的`voidr()` 方法。
  - 注意使用 `cfi(3)`,`thenRunAsync()` 效果似乎與 上例 `cfi(2)` 使用的 `runAsync()`相同，差異在後續的測試中體現：
    - `runAsync()` 是一個 `static` 方法，所以你通常不會像`cfi(2)`一樣呼叫它。相反你可以在 `QuittingCompletable.java` 中使用它。
    - 後續測試中表明 `supplyAsync()` 也是靜態方法，區別在於它需要一個 `Supplier` 而不是`Runnable`, 並產生一個`CompletableFuture<Integer>` 而不是 `CompletableFuture<Void>`。
  - `then` 系列方法將對現有的 `CompletableFuture<Integer>` 進一步操作。
    - 與 `thenRunAsync()` 不同，`cfi(4)`，`cfi(5)` 和`cfi(6)` "then" 方法的參數是未包裝的 `Integer`。
    - 透過使用 `voidr()`方法可以看到: 
      - `AcceptAsync()`接收了一個 `Consumer`，因此不會產生結果。
      - `thenApplyAsync()` 接收一個`Function`, 並生成一個結果（該結果的類型可以不同於其輸入類型）。
      - `thenComposeAsync()` 與 `thenApplyAsync()`非常相似，唯一區別在於其 `Function` 必須產生已經包裝在`CompletableFuture`中的結果。
  - `cfi(7)` 範例示範了 `obtrudeValue()`，它強制將值作為結果。
  - `cfi(8)` 使用 `toCompletableFuture()` 從 `CompletionStage` 生成一個`CompletableFuture`。
  - `c.complete(9)` 顯示了如何透過給它一個結果來完成一個`task`（`future`）（與 `obtrudeValue()` 相對，後者可能會迫使其結果取代該結果）。
  - 如果你呼叫 `CompletableFuture`中的 `cancel()`方法，如果已經完成此任務，則正常結束。 如果尚未完成，則使用 `CancellationException` 完成此 `CompletableFuture`。
  - 如果任務（`future`）完成，則 **getNow()** 方法返回`CompletableFuture`的完成值，否則返回`getNow()`的取代參數。
  - 最後，我們看一下依賴 (`dependents`) 的概念。如果我們將兩個`thenApplyAsync()`呼叫鏈路到`CompletableFuture`上，則依賴項的數量不會增加，保持為 1。但是，如果我們另外將另一個`thenApplyAsync()`直接附加到`c`，則現在有兩個依賴項：兩個一起的鏈路和另一個單獨附加的鏈路。
    - 這表明你可以使用一個`CompletionStage`，當它完成時，可以根據其結果衍生多個新任務。



### 結合 CompletableFuture

第二種類型的 `CompletableFuture` 方法採用兩種 `CompletableFuture` 並以各異方式將它們組合在一起。就像兩個人在比賽一樣, 一個`CompletableFuture`通常比另一個更早地到達終點。這些方法允許你以不同的方式處理結果。
為了測試這一點，我們將建立一個任務，它有一個我們可以控制的定義了完成任務所需要的時間量的參數。 
CompletableFuture 先完成:
```java
// concurrent/Workable.java
import java.util.concurrent.*;
import onjava.Nap;

public class Workable {
    String id;
    final double duration;

    public Workable(String id, double duration) {
        this.id = id;
        this.duration = duration;
    }

    @Override
    public String toString() {
        return "Workable[" + id + "]";
    }

    public static Workable work(Workable tt) {
        new Nap(tt.duration); // Seconds
        tt.id = tt.id + "W";
        System.out.println(tt);
        return tt;
    }

    public static CompletableFuture<Workable> make(String id, double duration) {
        return CompletableFuture
                .completedFuture(
                        new Workable(id, duration)
                )
                .thenApplyAsync(Workable::work);
    }
}
```

在 `make()`中，`work()`方法應用於`CompletableFuture`。`work()`需要一定的時間才能完成，然後它將字母 W 附加到 id 上，表示工作已經完成。

現在我們可以建立多個競爭的 `CompletableFuture`，並使用 `CompletableFuture` 庫中的各種方法來進行操作:

```java
// concurrent/DualCompletableOperations.java
import java.util.concurrent.*;
import static onjava.CompletableUtilities.*;

public class DualCompletableOperations {
    static CompletableFuture<Workable> cfA, cfB;

    static void init() {
        cfA = Workable.make("A", 0.15);
        cfB = Workable.make("B", 0.10); // Always wins
    }

    static void join() {
        cfA.join();
        cfB.join();
        System.out.println("*****************");
    }

    public static void main(String[] args) {
        init();
        voidr(cfA.runAfterEitherAsync(cfB, () ->
                System.out.println("runAfterEither")));
        join();

        init();
        voidr(cfA.runAfterBothAsync(cfB, () ->
                System.out.println("runAfterBoth")));
        join();

        init();
        showr(cfA.applyToEitherAsync(cfB, w -> {
            System.out.println("applyToEither: " + w);
            return w;
        }));
        join();

        init();
        voidr(cfA.acceptEitherAsync(cfB, w -> {
            System.out.println("acceptEither: " + w);
        }));
        join();

        init();
        voidr(cfA.thenAcceptBothAsync(cfB, (w1, w2) -> {
            System.out.println("thenAcceptBoth: "
                    + w1 + ", " + w2);
        }));
        join();

        init();
        showr(cfA.thenCombineAsync(cfB, (w1, w2) -> {
            System.out.println("thenCombine: "
                    + w1 + ", " + w2);
            return w1;
        }));
        join();

        init();
        CompletableFuture<Workable>
                cfC = Workable.make("C", 0.08),
                cfD = Workable.make("D", 0.09);
        CompletableFuture.anyOf(cfA, cfB, cfC, cfD)
                .thenRunAsync(() ->
                        System.out.println("anyOf"));
        join();

        init();
        cfC = Workable.make("C", 0.08);
        cfD = Workable.make("D", 0.09);
        CompletableFuture.allOf(cfA, cfB, cfC, cfD)
                .thenRunAsync(() ->
                        System.out.println("allOf"));
        join();
    }
}
```

**輸出結果**：

```
Workable[BW]
runAfterEither
Workable[AW]
*****************
Workable[BW]
Workable[AW]
runAfterBoth
*****************
Workable[BW]
applyToEither: Workable[BW]
Workable[BW]
Workable[AW]
*****************
Workable[BW]
acceptEither: Workable[BW]
Workable[AW]
*****************
Workable[BW]
Workable[AW]
thenAcceptBoth: Workable[AW], Workable[BW]
****************
 Workable[BW]
 Workable[AW]
 thenCombine: Workable[AW], Workable[BW]
 Workable[AW]
 *****************
 Workable[CW]
 anyOf
 Workable[DW]
 Workable[BW]
 Workable[AW]
 *****************
 Workable[CW]
 Workable[DW]
 Workable[BW]
 Workable[AW]
 *****************
 allOf
```

- 為了方便訪問， 將 `cfA` 和 `cfB` 定義為 `static`的。 
  - `init()`方法用於 `A`, `B` 初始化這兩個變數，因為 `B` 總是給出比`A`較短的延遲，所以總是 `win` 的一方。
  - `join()` 是在兩個方法上呼叫 `join()` 並顯示邊框的另一個便利方法。
- 所有這些 “`dual`” 方法都以一個 `CompletableFuture` 作為呼叫該方法的物件，第二個 `CompletableFuture` 作為第一個參數，然後是要執行的操作。
- 透過使用 `showr()` 和 `voidr()` 可以看到，“`run`”和“`accept`”是終端操作，而“`apply`”和“`combine`”則生成新的 `payload-bearing` (承載負載) 的 `CompletableFuture`。
- 方法的名稱不言自明，你可以透過查看輸出來驗證這一點。一個特別有趣的方法是 `combineAsync()`，它等待兩個 `CompletableFuture` 完成，然後將它們都交給一個 `BiFunction`，這個 `BiFunction` 可以將結果加入到最終的 `CompletableFuture` 的有效負載中。


### 模擬

作為使用 `CompletableFuture` 將一系列操作組合的範例，讓我們模擬一下製作蛋糕的過程。在第一階段，我們準備並將原料混合成麵糊:

```java
// concurrent/Batter.java
import java.util.concurrent.*;
import onjava.Nap;

public class Batter {
    static class Eggs {
    }

    static class Milk {
    }

    static class Sugar {
    }

    static class Flour {
    }

    static <T> T prepare(T ingredient) {
        new Nap(0.1);
        return ingredient;
    }

    static <T> CompletableFuture<T> prep(T ingredient) {
        return CompletableFuture
                .completedFuture(ingredient)
                .thenApplyAsync(Batter::prepare);
    }

    public static CompletableFuture<Batter> mix() {
        CompletableFuture<Eggs> eggs = prep(new Eggs());
        CompletableFuture<Milk> milk = prep(new Milk());
        CompletableFuture<Sugar> sugar = prep(new Sugar());
        CompletableFuture<Flour> flour = prep(new Flour());
        CompletableFuture
                .allOf(eggs, milk, sugar, flour)
                .join();
        new Nap(0.1); // Mixing time
        return CompletableFuture.completedFuture(new Batter());
    }
}
```

每種原料都需要一些時間來準備。`allOf()` 等待所有的配料都準備好，然後使用更多些的時間將其混合成麵糊。接下來，我們把單批麵糊放入四個平底鍋中烘烤。產品作為 `CompletableFutures`  流返回：

```java
// concurrent/Baked.java

import java.util.concurrent.*;
import java.util.stream.*;
import onjava.Nap;

public class Baked {
    static class Pan {
    }

    static Pan pan(Batter b) {
        new Nap(0.1);
        return new Pan();
    }

    static Baked heat(Pan p) {
        new Nap(0.1);
        return new Baked();
    }

    static CompletableFuture<Baked> bake(CompletableFuture<Batter> cfb) {
        return cfb
                .thenApplyAsync(Baked::pan)
                .thenApplyAsync(Baked::heat);
    }

    public static Stream<CompletableFuture<Baked>> batch() {
        CompletableFuture<Batter> batter = Batter.mix();
        return Stream.of(
                bake(batter),
                bake(batter),
                bake(batter),
                bake(batter)
        );
    }
}
```

最後，我們製作了一批糖，並用它對蛋糕進行糖化：

```java
// concurrent/FrostedCake.java

import java.util.concurrent.*;
import java.util.stream.*;
import onjava.Nap;

final class Frosting {
    private Frosting() {
    }

    static CompletableFuture<Frosting> make() {
        new Nap(0.1);
        return CompletableFuture
                .completedFuture(new Frosting());
    }
}

public class FrostedCake {
    public FrostedCake(Baked baked, Frosting frosting) {
        new Nap(0.1);
    }

    @Override
    public String toString() {
        return "FrostedCake";
    }

    public static void main(String[] args) {
        Baked.batch().forEach(
                baked -> baked
                        .thenCombineAsync(Frosting.make(),
                                (cake, frosting) ->
                                        new FrostedCake(cake, frosting))
                        .thenAcceptAsync(System.out::println)
                        .join());
    }
}
```

一旦你習慣了這種背後的想法, `CompletableFuture` 它們相對易於使用。

### 異常

與 `CompletableFuture` 在處理鏈中包裝物件的方式相同，它也會緩衝異常。這些在處理時呼叫者是無感的，但僅當你嘗試提取結果時才會被告知。為了說明它們是如何工作的，我們首先建立一個類，它在特定的條件下拋出一個異常:

```java
// concurrent/Breakable.java
import java.util.concurrent.*;
public class Breakable {
    String id;
    private int failcount;

    public Breakable(String id, int failcount) {
        this.id = id;
        this.failcount = failcount;
    }

    @Override
    public String toString() {
        return "Breakable_" + id + " [" + failcount + "]";
    }

    public static Breakable work(Breakable b) {
        if (--b.failcount == 0) {
            System.out.println(
                    "Throwing Exception for " + b.id + ""
            );
            throw new RuntimeException(
                    "Breakable_" + b.id + " failed"
            );
        }
        System.out.println(b);
        return b;
    }
}
```

當`failcount` > 0，且每次將物件傳遞給 `work()` 方法時， `failcount - 1` 。當`failcount - 1 = 0` 時，`work()` 將拋出一個異常。如果傳給 `work()` 的 `failcount = 0` ，`work()` 永遠不會拋出異常。

注意，異常訊息此範例中被拋出（ `RuntimeException` )

在下面範例  `test()` 方法中，`work()` 多次應用於 `Breakable`，因此如果 `failcount` 在範圍內，就會拋出異常。然而，在測試`A`到`E`中，你可以從輸出中看到拋出了異常，但它們從未出現:

```java
// concurrent/CompletableExceptions.java
import java.util.concurrent.*;
public class CompletableExceptions {
    static CompletableFuture<Breakable> test(String id, int failcount) {
        return CompletableFuture.completedFuture(
                new Breakable(id, failcount))
                .thenApply(Breakable::work)
                .thenApply(Breakable::work)
                .thenApply(Breakable::work)
                .thenApply(Breakable::work);
    }

    public static void main(String[] args) {
        // Exceptions don't appear ...
        test("A", 1);
        test("B", 2);
        test("C", 3);
        test("D", 4);
        test("E", 5);
        // ... until you try to fetch the value:
        try {
            test("F", 2).get(); // or join()
        } catch (Exception e) {
            System.out.println(e.getMessage());
        }
        // Test for exceptions:
        System.out.println(
                test("G", 2).isCompletedExceptionally()
        );
        // Counts as "done":
        System.out.println(test("H", 2).isDone());
        // Force an exception:
        CompletableFuture<Integer> cfi =
                new CompletableFuture<>();
        System.out.println("done? " + cfi.isDone());
        cfi.completeExceptionally(
                new RuntimeException("forced"));
        try {
            cfi.get();
        } catch (Exception e) {
            System.out.println(e.getMessage());
        }
    }
}
```

輸出結果：

```
Throwing Exception for A
Breakable_B [1]
Throwing Exception for B
Breakable_C [2]
Breakable_C [1]
Throwing Exception for C
Breakable_D [3]
Breakable_D [2]
Breakable_D [1]
Throwing Exception for D
Breakable_E [4]
Breakable_E [3]
Breakable_E [2]
Breakable_E [1]
Breakable_F [1]
Throwing Exception for F
java.lang.RuntimeException: Breakable_F failed
Breakable_G [1]
Throwing Exception for G
true
Breakable_H [1]
Throwing Exception for H
true
done? false
java.lang.RuntimeException: forced
```

測試 `A` 到 `E` 執行到拋出異常，然後…並沒有將拋出的異常暴露給呼叫方。只有在測試 F 中呼叫 `get()` 時，我們才會看到拋出的異常。
測試 `G` 表明，你可以首先檢查在處理期間是否拋出異常，而不拋出該異常。然而，test `H` 告訴我們，不管異常是否成功，它仍然被視為已“完成”。
程式碼的最後一部分展示了如何將異常插入到 `CompletableFuture` 中，而不管是否存在任何失敗。
在連接或獲取結果時，我們使用 `CompletableFuture` 提供的更複雜的機制來自動響應異常，而不是使用粗糙的 `try-catch`。
你可以使用與我們看到的所有 `CompletableFuture`  相同的表單來完成此操作:在鏈中插入一個  `CompletableFuture` 呼叫。有三個選項 `exceptionally()`，`handle()`， `whenComplete()`:

```java
// concurrent/CatchCompletableExceptions.java
import java.util.concurrent.*;
public class CatchCompletableExceptions {
    static void handleException(int failcount) {
        // Call the Function only if there's an
        // exception, must produce same type as came in:
        CompletableExceptions
                .test("exceptionally", failcount)
                .exceptionally((ex) -> { // Function
                    if (ex == null)
                        System.out.println("I don't get it yet");
                    return new Breakable(ex.getMessage(), 0);
                })
                .thenAccept(str ->
                        System.out.println("result: " + str));

        // Create a new result (recover):
        CompletableExceptions
                .test("handle", failcount)
                .handle((result, fail) -> { // BiFunction
                    if (fail != null)
                        return "Failure recovery object";
                    else
                        return result + " is good";
                })
                .thenAccept(str ->
                        System.out.println("result: " + str));

        // Do something but pass the same result through:
        CompletableExceptions
                .test("whenComplete", failcount)
                .whenComplete((result, fail) -> { // BiConsumer
                    if (fail != null)
                        System.out.println("It failed");
                    else
                        System.out.println(result + " OK");
                })
                .thenAccept(r ->
                        System.out.println("result: " + r));
    }

    public static void main(String[] args) {
        System.out.println("**** Failure Mode ****");
        handleException(2);
        System.out.println("**** Success Mode ****");
        handleException(0);
    }
}
```

輸出結果：

```
**** Failure Mode ****
Breakable_exceptionally [1]
Throwing Exception for exceptionally
result: Breakable_java.lang.RuntimeException:
Breakable_exceptionally failed [0]
Breakable_handle [1]
Throwing Exception for handle
result: Failure recovery object
Breakable_whenComplete [1]
Throwing Exception for whenComplete
It failed
**** Success Mode ****
Breakable_exceptionally [-1]
Breakable_exceptionally [-2]
Breakable_exceptionally [-3]
Breakable_exceptionally [-4]
result: Breakable_exceptionally [-4]
Breakable_handle [-1]
Breakable_handle [-2]
Breakable_handle [-3]
Breakable_handle [-4]
result: Breakable_handle [-4] is good
Breakable_whenComplete [-1]
Breakable_whenComplete [-2]
Breakable_whenComplete [-3]
Breakable_whenComplete [-4]
Breakable_whenComplete [-4] OK
result: Breakable_whenComplete [-4]
```

- `exceptionally()`  參數僅在出現異常時才執行。`exceptionally()`  局限性在於，該函數只能返回輸入類型相同的值。

- `exceptionally()` 透過將一個好的物件插入到流中來復原到一個可行的狀態。

- `handle()` 一致被呼叫來查看是否發生異常（必須檢查 fail 是否為 true）。

  - 但是 `handle()` 可以生成任何新類型，所以它允許執行處理，而不是像使用 `exceptionally()`那樣簡單地復原。

  - `whenComplete()` 類似於 handle()，同樣必須測試它是否失敗，但是參數是一個消費者，並且不修改傳遞給它的結果物件。


### 流異常（Stream Exception）

透過修改 **CompletableExceptions.java** ，看看 **CompletableFuture** 異常與流異常有何不同：

```java
// concurrent/StreamExceptions.java
import java.util.concurrent.*;
import java.util.stream.*;
public class StreamExceptions {
    
    static Stream<Breakable> test(String id, int failcount) {
        return Stream.of(new Breakable(id, failcount))
                .map(Breakable::work)
                .map(Breakable::work)
                .map(Breakable::work)
                .map(Breakable::work);
    }

    public static void main(String[] args) {
        // No operations are even applied ...
        test("A", 1);
        test("B", 2);
        Stream<Breakable> c = test("C", 3);
        test("D", 4);
        test("E", 5);
        // ... until there's a terminal operation:
        System.out.println("Entering try");
        try {
            c.forEach(System.out::println);   // [1]
        } catch (Exception e) {
            System.out.println(e.getMessage());
        }
    }
}
```

輸出結果：

```
Entering try
Breakable_C [2]
Breakable_C [1]
Throwing Exception for C
Breakable_C failed
```

使用 `CompletableFuture`，我們可以看到測試 A 到 E 的進展，但是使用流，在你應用一個終端操作之前（e.g. `forEach()`），什麼都不會暴露給 Client 

`CompletableFuture` 執行工作並捕獲任何異常供以後檢索。比較這兩者並不容易，因為 `Stream` 在沒有終端操作的情況下根本不做任何事情——但是流絕對不會儲存它的異常。

### 檢查性異常

`CompletableFuture` 和 `parallel Stream` 都不支援包含檢查性異常的操作。相反，你必須在呼叫操作時處理檢查到的異常，這會產生不太優雅的程式碼：

```java
// concurrent/ThrowsChecked.java
import java.util.stream.*;
import java.util.concurrent.*;

public class ThrowsChecked {
    class Checked extends Exception {}

    static ThrowsChecked nochecked(ThrowsChecked tc) {
        return tc;
    }

    static ThrowsChecked withchecked(ThrowsChecked tc) throws Checked {
        return tc;
    }

    static void testStream() {
        Stream.of(new ThrowsChecked())
                .map(ThrowsChecked::nochecked)
                // .map(ThrowsChecked::withchecked); // [1]
                .map(
                        tc -> {
                            try {
                                return withchecked(tc);
                            } catch (Checked e) {
                                throw new RuntimeException(e);
                            }
                        });
    }

    static void testCompletableFuture() {
        CompletableFuture
                .completedFuture(new ThrowsChecked())
                .thenApply(ThrowsChecked::nochecked)
                // .thenApply(ThrowsChecked::withchecked); // [2]
                .thenApply(
                        tc -> {
                            try {
                                return withchecked(tc);
                            } catch (Checked e) {
                                throw new RuntimeException(e);
                            }
                        });
    }
}
```

如果你試圖像使用 `nochecked()` 那樣使用` withchecked()` 的方法引用，編譯器會在 `[1]` 和 `[2]` 中報錯。相反，你必須寫出 lambda 表達式 (或者編寫一個不會拋出異常的包裝器方法)。

## 死鎖

由於任務可以被阻塞，因此一個任務有可能卡在等待另一個任務上，而後者又在等待別的任務，這樣一直下去，知道這個鏈條上的任務又在等待第一個任務釋放鎖。這得到了一個任務之間相互等待的連續循環， 沒有哪個執行緒能繼續， 這稱之為死鎖[^6]
如果你執行一個程式，而它馬上就死鎖了， 你可以立即跟蹤下去。真正的問題在於，程式看起來工作良好， 但是具有潛在的死鎖危險。這時， 死鎖可能發生，而事先卻沒有任何徵兆， 所以 `bug` 會潛伏在你的程式例，直到客戶發現它出乎意料的發生（以一種幾乎肯定是很難重現的方式發生）。因此在編寫並發程式的時候，進行仔細的程式設計以防止死鎖是關鍵部分。
埃德斯·迪克斯特拉（`Essger Dijkstra`）發明的“哲學家進餐"問題是經典的死鎖例證。基本描述指定了五位哲學家（此處顯示的範例允許任何數目）。這些哲學家將花部分時間思考，花部分時間就餐。他們在思考的時候並不需要任何共享資源；但是他們使用的餐具數量有限。在最初的問題描述中，餐具是叉子，需要兩個叉子才能從桌子中間的碗裡取出義大利麵。常見的版本是使用筷子， 顯然，每個哲學家都需要兩根筷子才能吃飯。
引入了一個困難：作為哲學家，他們的錢很少，所以他們只能買五根筷子（更一般地講，筷子的數量與哲學家相同）。他們圍在桌子周圍，每人之間放一根筷子。 當一個哲學家要就餐時，該哲學家必須同時持有左邊和右邊的筷子。如果任一側的哲學家都在使用所需的筷子，則我們的哲學家必須等待，直到可得到必須的筷子。

**StickHolder**  類透過將單根筷子保持在大小為 1 的 **BlockingQueue** 中來管理它。**BlockingQueue** 是一個設計用於在併發程式中安全使用的集合，如果你呼叫 take() 並且佇列為空，則它將阻塞（等待）。將新元素放入佇列後，將釋放該塊並返回該值：

```java
// concurrent/StickHolder.java
import java.util.concurrent.*;
public class StickHolder {
    private static class Chopstick {
    }

    private Chopstick stick = new Chopstick();
    private BlockingQueue<Chopstick> holder =
            new ArrayBlockingQueue<>(1);

    public StickHolder() {
        putDown();
    }

    public void pickUp() {
        try {
            holder.take(); // Blocks if unavailable
        } catch (InterruptedException e) {
            throw new RuntimeException(e);
        }
    }

    public void putDown() {
        try {
            holder.put(stick);
        } catch (InterruptedException e) {
            throw new RuntimeException(e);
        }
    }
}
```

為簡單起見，`Chopstick`(`static`) 實際上不是由 `StickHolder` 生產的，而是在其類中保持私有的。

如果您呼叫了`pickUp()`，而 `stick` 不可用，那麼`pickUp()`將阻塞該 `stick`，直到另一個哲學家呼叫`putDown()` 將 `stick` 返回。 

注意，該類中的所有執行緒安全都是透過 `BlockingQueue` 實現的。

每個哲學家都是一項任務，他們試圖把筷子分別 `pickUp()` 在左手和右手上，這樣筷子才能吃東西，然後透過 `putDown()` 放下 `stick`。

```java
// concurrent/Philosopher.java
public class Philosopher implements Runnable {
    private final int seat;
    private final StickHolder left, right;

    public Philosopher(int seat, StickHolder left, StickHolder right) {
        this.seat = seat;
        this.left = left;
        this.right = right;
    }

    @Override
    public String toString() {
        return "P" + seat;
    }

    @Override
    public void run() {
        while (true) {
            // System.out.println("Thinking");   // [1]
            right.pickUp();
            left.pickUp();
            System.out.println(this + " eating");
            right.putDown();
            left.putDown();
        }
    }
}
```

沒有兩個哲學家可以同時成功呼叫 take() 同一隻筷子。另外，如果一個哲學家已經拿過筷子，那麼下一個試圖拿起同一根筷子的哲學家將阻塞，等待其被釋放。

結果是一個看似無辜的程式陷入了死鎖。我在這裡使用陣列而不是集合，只是因為這種語法更簡潔：

```java
// concurrent/DiningPhilosophers.java
// Hidden deadlock
// {ExcludeFromGradle} Gradle has trouble
import java.util.*;
import java.util.concurrent.*;
import onjava.Nap;

public class DiningPhilosophers {
    private StickHolder[] sticks;
    private Philosopher[] philosophers;

    public DiningPhilosophers(int n) {
        sticks = new StickHolder[n];
        Arrays.setAll(sticks, i -> new StickHolder());
        philosophers = new Philosopher[n];
        Arrays.setAll(philosophers, i ->
                new Philosopher(i,
                        sticks[i], sticks[(i + 1) % n]));    // [1]
        // Fix by reversing stick order for this one:
        // philosophers[1] =                     // [2]
        //   new Philosopher(0, sticks[0], sticks[1]);
        Arrays.stream(philosophers)
                .forEach(CompletableFuture::runAsync); // [3]
    }

    public static void main(String[] args) {
        // Returns right away:
        new DiningPhilosophers(5);               // [4]
        // Keeps main() from exiting:
        new Nap(3, "Shutdown");
    }
}
```

- 當你停止查看輸出時，該程式將死鎖。但是，根據你的電腦配備，你可能不會看到死鎖。看來這取決於電腦上的核心數[^7]。兩個核心不會產生死鎖，但兩核以上卻很容易產生死鎖。
- 此行為使該範例更好地說明了死鎖，因為你可能正在具有 2 核的電腦上編寫程式（如果確實是導致問題的原因），並且確信該程式可以正常工作，只能啟動它將其安裝在另一台電腦上時出現死鎖。請注意，不能因為你沒或不容易看到死鎖，這並不意味著此程式不會在 2 核機器上發生死鎖。 該程式仍然有死鎖傾向，只是很少發生——可以說是最糟糕的情況，因為問題不容易出現。
- 在 `DiningPhilosophers` 的構造方法中，每個哲學家都獲得一個左右筷子的引用。除最後一個哲學家外，都是透過把哲學家放在下一雙空閒筷子之間來初始化： 
  - 最後一位哲學家得到了第 0 根筷子作為他的右筷子，所以圓桌就完成。
  - 那是因為最後一位哲學家正坐在第一個哲學家的旁邊，而且他們倆都共用零筷子。[1] 顯示了以 n 為模數選擇的右筷子，將最後一個哲學家繞到第一個哲學家的旁邊。
- 現在，所有哲學家都可以嘗試吃飯，每個哲學家都在旁邊等待哲學家放下筷子。
  - 為了讓每個哲學家在[3] 上執行，呼叫 `runAsync()`，這意味著 DiningPhilosophers 的建構子立即返回到[4]。
  - 如果沒有任何東西阻止 `main()` 完成，程式就會退出，不會做太多事情。
  - `Nap` 物件阻止 `main()` 退出，然後在三秒後強制退出 (假設/可能是) 死鎖程式。
  - 在給定的配置中，哲學家幾乎不花時間思考。因此，他們在吃東西的時候都爭著用筷子，而且往往很快就會陷入僵局。你可以改變這個:

1. 透過增加[4] 的值來添加更多哲學家。

2. 在 Philosopher.java 中取消注釋行[1]。

任一種方法都會減少死鎖的可能性，這表明編寫並發程式並認為它是安全的危險，因為它似乎“在我的機器上執行正常”。你可以輕鬆地說服自己該程式沒有死鎖，即使它不是。這個範例相當有趣，因為它示範了看起來可以正確執行，但實際上會可能發生死鎖的程式。

要修正死鎖問題，你必須明白，當以下四個條件同時滿足時，就會發生死鎖：

1) 互斥條件。任務使用的資源中至少有一個不能共享的。 這裡，一根筷子一次就只能被一個哲學家使用。
2) 至少有一個任務它必須持有一個資源且正在等待獲取一個被目前別的任務持有的資源。也就是說，要發生死鎖，哲學家必須拿著一根筷子並且等待另一根。
3) 資源不能被任務搶占， 任務必須把資源釋放當作普通事件。哲學家很有禮貌，他們不會從其它哲學家那裡搶筷子。
4) 必須有循環等待， 這時，一個任務等待其它任務所持有的資源， 後者又在等待另一個任務所持有的資源， 這樣一直下去，知道有一個任務在等待第一個任務所持有的資源， 使得大家都被鎖住。 在 `DiningPhilosophers.java` 中， 因為每個哲學家都試圖先得到右邊的 筷子, 然後得到左邊的 筷子, 所以發生了循環等待。

因為必須滿足所有條件才能導致死鎖，所以要阻止死鎖的話，只需要破壞其中一個即可。在此程式中，防止死鎖的一種簡單方法是打破第四個條件。之所以會發生這種情況，是因為每個哲學家都嘗試按照特定的順序撿起自己的筷子：先右後左。因此，每個哲學家都有可能在等待左手的同時握住右手的筷子，從而導致循環等待狀態。但是，如果其中一位哲學家嘗試首先拿起左筷子，則該哲學家決不會阻止緊鄰右方的哲學家拿起筷子，從而排除了循環等待。

在 **DiningPhilosophers.java** 中，取消注釋[1] 和其後的一行。這將原來的哲學家[1] 取代為筷子顛倒的哲學家。透過確保第二位哲學家撿起並在右手之前放下左筷子，我們消除了死鎖的可能性。
這只是解決問題的一種方法。你也可以透過防止其他情況之一來解決它。
沒有語言支援可以幫助防止死鎖；你有責任透過精心設計來避免這種情況。對於試圖除錯死鎖程式的人來說，這些都不是安慰。當然，避免並發問題的最簡單，最好的方法是永遠不要共享資源-不幸的是，這並不總是可能的。



## 構造方法非執行緒安全

當你在腦子裡想像一個物件構造的過程，你會很容易認為這個過程是執行緒安全的。畢竟，在物件初始化完成前對外不可見，所以又怎會對此產生爭議呢？確實，[Java 語言規範 ](https://docs.oracle.com/javase/specs/jls/se8/html/jls-8.html#jls-8.8.3) (JLS) 自信滿滿地陳述道：“*沒必要使構造器的執行緒同步，因為它會鎖定正在構造的物件，直到構造器完成初始化後才對其他執行緒可見。*”

不幸的是，物件的構造過程如其他操作一樣，也會受到共享記憶體並發問題的影響，只是作用機制可能更微妙罷了。

設想一下使用一個 **static**  欄位為每個物件自動建立唯一標識符的過程。為了測試其不同的實現過程，我們從一個介面開始。程式碼範例：

```java
//concurrent/HasID.java
public interface HasID {
    int getID();
}
```

然後 **StaticIDField**  類顯式地實現該介面。程式碼範例：

```java
// concurrent/StaticIDField.java
public class StaticIDField implements HasID {
    private static int counter = 0;
    private int id = counter++;
    public int getID() { return id; }
}
```

如你所想，該類是個簡單無害的類，它甚至都沒一個顯式的構造器來引發問題。當我們執行多個用於建立此類物件的執行緒時，究竟會發生什麼事？為了搞清楚這點，我們做了以下測試。程式碼範例：

```java
// concurrent/IDChecker.java
import java.util.*;
import java.util.function.*;
import java.util.stream.*;
import java.util.concurrent.*;
import com.google.common.collect.Sets;
public class IDChecker {
    public static final int SIZE = 100_000;

    static class MakeObjects implements
        Supplier<List<Integer>> {
        private Supplier<HasID> gen;

        MakeObjects(Supplier<HasID> gen) {
            this.gen = gen;
        }

        @Override public List<Integer> get() {
            return Stream.generate(gen)
            .limit(SIZE)
            .map(HasID::getID)
            .collect(Collectors.toList());
        }
    }

    public static void test(Supplier<HasID> gen) {
        CompletableFuture<List<Integer>>
        groupA = CompletableFuture.supplyAsync(new
            MakeObjects(gen)),
        groupB = CompletableFuture.supplyAsync(new
            MakeObjects(gen));

        groupA.thenAcceptBoth(groupB, (a, b) -> {
            System.out.println(
                Sets.intersection(
                Sets.newHashSet(a),
                Sets.newHashSet(b)).size());
            }).join();
    }
}
```

**MakeObjects**  類是一個生產者類，包含一個能夠產生 List\<Integer> 類型的列表物件的 `get()` 方法。透過從每個 `HasID` 物件提取 `ID` 並放入列表中來生成這個列表物件，而 `test()` 方法則建立了兩個並行的 **CompletableFuture**  物件，用於執行 **MakeObjects**  生產者類，然後獲取執行結果。

使用 Guava 庫中的 **Sets.`intersection()` 方法，計算出這兩個返回的 List\<Integer> 物件中有多少相同的 `ID`（使用Google Guava 庫裡的方法比使用官方的 `retainAll()` 方法速度快得多）。

現在我們可以測試上面的 **StaticIDField**  類了。程式碼範例：

```java
// concurrent/TestStaticIDField.java
public class TestStaticIDField {

    public static void main(String[] args) {
        IDChecker.test(StaticIDField::new);
    }
}
```

輸出結果：

```
    13287
```

結果中出現了很多重複項。很顯然，純靜態 `int` 用於構造過程並不是執行緒安全的。讓我們使用 **AtomicInteger**  來使其變為執行緒安全的。程式碼範例：

```java
// concurrent/GuardedIDField.java
import java.util.concurrent.atomic.*;
public class GuardedIDField implements HasID {  
    private static AtomicInteger counter = new
        AtomicInteger();

    private int id = counter.getAndIncrement();

    public int getID() { return id; }

    public static void main(String[] args) {                IDChecker.test(GuardedIDField::new);
    }
}
```

輸出結果：

```
    0
```

構造器有一種更微妙的狀態共享方式：透過構造器參數：

```java
// concurrent/SharedConstructorArgument.java
import java.util.concurrent.atomic.*;
interface SharedArg{
    int get();
}

class Unsafe implements SharedArg{
    private int i = 0;

    public int get(){
        return i++;
    }
}

class Safe implements SharedArg{
    private static AtomicInteger counter = new AtomicInteger();

    public int get(){
        return counter.getAndIncrement();
    }
}

class SharedUser implements HasID{
    private final int id;

    SharedUser(SharedArg sa){
        id = sa.get();
    }

    @Override
    public int getID(){
        return id;
    }
}

public class SharedConstructorArgument{
    public static void main(String[] args){
        Unsafe unsafe = new Unsafe();
        IDChecker.test(() -> new SharedUser(unsafe));

        Safe safe = new Safe();
        IDChecker.test(() -> new SharedUser(safe));
    }
}
```

輸出結果：

```
    24838
    0
```

在這裡，**SharedUser**  構造器實際上共享了相同的參數。即使 **SharedUser**  以完全無害且合理的方式使用其自己的參數，其構造器的呼叫方式也會引起衝突。**SharedUser**  甚至不知道它是以這種方式呼叫的，更不必說控制它了。

同步構造器並不被 java 語言所支援，但是透過使用同步語塊來建立你自己的同步構造器是可能的（請參閱附錄：[並發底層原理 ](./Appendix-Low-Level-Concurrency.md)，來進一步了解同步關鍵字—— `synchronized`）。儘管 JLS（java 語言規範）這樣陳述道：“……它會鎖定正在構造的物件”，但這並不是真的——構造器實際上只是一個靜態方法，因此同步構造器實際上會鎖定該類的 Class 物件。我們可以透過建立自己的靜態物件並鎖定它，來達到與同步構造器相同的效果：

```java
// concurrent/SynchronizedConstructor.java

import java.util.concurrent.atomic.*;

class SyncConstructor implements HasID{
    private final int id;
    private static Object constructorLock =
        new Object();

    SyncConstructor(SharedArg sa){
        synchronized (constructorLock){
            id = sa.get();
        }
    }

    @Override
    public int getID(){
        return id;
    }
}

public class SynchronizedConstructor{
    public static void main(String[] args){
        Unsafe unsafe = new Unsafe();
        IDChecker.test(() -> new SyncConstructor(unsafe));
    }
}
```

輸出結果：

```
    0
```

**Unsafe** 類的共享使用現在就變得安全了。另一種方法是將構造器設為私有（因此可以防止繼承），並提供一個靜態 Factory 方法來生成新物件：

```java
// concurrent/SynchronizedFactory.java
import java.util.concurrent.atomic.*;

final class SyncFactory implements HasID{
    private final int id;

    private SyncFactory(SharedArg sa){
        id = sa.get();
    }

    @Override
    public int getID(){
        return id;
    }

    public static synchronized SyncFactory factory(SharedArg sa){
        return new SyncFactory(sa);
    }
}

public class SynchronizedFactory{
    public static void main(String[] args){
        Unsafe unsafe = new Unsafe();
        IDChecker.test(() -> SyncFactory.factory(unsafe));
    }
}
```

輸出結果：

```
    0
```

透過同步靜態工廠方法，可以在構造過程中鎖定 **Class**  物件。

這些範例充分表明了在並發 Java 程式中檢測和管理共享狀態有多困難。即使你採取“不共享任何內容”的策略，也很容易產生意外的共享事件。

## 複雜性和代價

假設你正在做披薩，我們把從整個流程的目前步驟到下一個步驟所需的工作量，在這裡一一表示為列舉變數的一部分：

```java
// concurrent/Pizza.java import java.util.function.*;

import onjava.Nap;
public class Pizza{
    public enum Step{
        DOUGH(4), ROLLED(1), SAUCED(1), CHEESED(2),
        TOPPED(5), BAKED(2), SLICED(1), BOXED(0);
        int effort;// Needed to get to the next step 

        Step(int effort){
            this.effort = effort;
        }

        Step forward(){
            if (equals(BOXED)) return BOXED;
            new Nap(effort * 0.1);
            return values()[ordinal() + 1];
        }
    }

    private Step step = Step.DOUGH;
    private final int id;

    public Pizza(int id){
        this.id = id;
    }

    public Pizza next(){
        step = step.forward();
        System.out.println("Pizza " + id + ": " + step);
        return this;
    }

    public Pizza next(Step previousStep){
        if (!step.equals(previousStep))
            throw new IllegalStateException("Expected " +
                      previousStep + " but found " + step);
        return next();
    }

    public Pizza roll(){
        return next(Step.DOUGH);
    }

    public Pizza sauce(){
        return next(Step.ROLLED);
    }

    public Pizza cheese(){
        return next(Step.SAUCED);
    }

    public Pizza toppings(){
        return next(Step.CHEESED);
    }

    public Pizza bake(){
        return next(Step.TOPPED);
    }

    public Pizza slice(){
        return next(Step.BAKED);
    }

    public Pizza box(){
        return next(Step.SLICED);
    }

    public boolean complete(){
        return step.equals(Step.BOXED);
    }

    @Override
    public String toString(){
        return "Pizza" + id + ": " + (step.equals(Step.BOXED) ? "complete" : step);
    }
}
```

這隻算得上是一個平凡的狀態機，就像 **Machina** 類一樣。 

製作一個披薩，當披薩最終被放在盒子中時，就算完成最終任務了。 如果一個人在做一個披薩，那麼所有步驟都是線性進行的，即一個接一個地進行：

```java
// concurrent/OnePizza.java 

import onjava.Timer;

public class OnePizza{
    public static void main(String[] args){
        Pizza za = new Pizza(0);
        System.out.println(Timer.duration(() -> {
            while (!za.complete()) za.next();
        }));
    }
}
```

輸出結果：

```
Pizza 0: ROLLED 
Pizza 0: SAUCED 
Pizza 0: CHEESED 
Pizza 0: TOPPED 
Pizza 0: BAKED 
Pizza 0: SLICED 
Pizza 0: BOXED 
	1622 
```

時間以毫秒為單位，加總所有步驟的工作量，會得出與我們的期望值相符的數字。 如果你以這種方式製作了五個披薩，那麼你會認為它花費的時間是原來的五倍。 但是，如果這還不夠快怎麼辦？ 我們可以從嘗試並行流方法開始：

```java
// concurrent/PizzaStreams.java
// import java.util.*; import java.util.stream.*;

import onjava.Timer;

public class PizzaStreams{
    static final int QUANTITY = 5;

    public static void main(String[] args){
        Timer timer = new Timer();
        IntStream.range(0, QUANTITY)
            .mapToObj(Pizza::new)
            .parallel()//[1]
        	.forEach(za -> { while(!za.complete()) za.next(); }); 			System.out.println(timer.duration());
    }
}
```

輸出結果：

```
Pizza 2: ROLLED
Pizza 0: ROLLED
Pizza 1: ROLLED
Pizza 4: ROLLED
Pizza 3:ROLLED
Pizza 2:SAUCED
Pizza 1:SAUCED
Pizza 0:SAUCED
Pizza 4:SAUCED
Pizza 3:SAUCED
Pizza 2:CHEESED
Pizza 1:CHEESED
Pizza 0:CHEESED
Pizza 4:CHEESED
Pizza 3:CHEESED
Pizza 2:TOPPED
Pizza 1:TOPPED
Pizza 0:TOPPED
Pizza 4:TOPPED
Pizza 3:TOPPED
Pizza 2:BAKED
Pizza 1:BAKED
Pizza 0:BAKED
Pizza 4:BAKED
Pizza 3:BAKED
Pizza 2:SLICED
Pizza 1:SLICED
Pizza 0:SLICED
Pizza 4:SLICED
Pizza 3:SLICED
Pizza 2:BOXED
Pizza 1:BOXED
Pizza 0:BOXED
Pizza 4:BOXED
Pizza 3:BOXED
1739
```

現在，我們製作五個披薩的時間與製作單個披薩的時間就差不多了。 嘗試刪除標記為[1] 的行後，你會發現它花費的時間是原來的五倍。 你還可以嘗試將 **QUANTITY** 更改為 4、8、10、16 和 17，看看會有什麼不同，並猜猜看為什麼會這樣。

**PizzaStreams**  類產生的每個並行流在它的`forEach()`內完成所有工作，如果我們將其各個步驟用映射的方式一步一步處理，情況會有所不同嗎？

```java
// concurrent/PizzaParallelSteps.java 

import java.util.*;
import java.util.stream.*;
import onjava.Timer;

public class PizzaParallelSteps{
    static final int QUANTITY = 5;

    public static void main(String[] args){
        Timer timer = new Timer();
        IntStream.range(0, QUANTITY)
            .mapToObj(Pizza::new)
            .parallel()
            .map(Pizza::roll)
            .map(Pizza::sauce)
            .map(Pizza::cheese)
            .map(Pizza::toppings)
            .map(Pizza::bake)
            .map(Pizza::slice)
            .map(Pizza::box)
            .forEach(za -> System.out.println(za));
        System.out.println(timer.duration());
    }
} 
```

輸出結果：

```
Pizza 2: ROLLED 
Pizza 0: ROLLED 
Pizza 1: ROLLED 
Pizza 4: ROLLED 
Pizza 3: ROLLED 
Pizza 1: SAUCED 
Pizza 0: SAUCED 
Pizza 2: SAUCED 
Pizza 3: SAUCED 
Pizza 4: SAUCED 
Pizza 1: CHEESED 
Pizza 0: CHEESED 
Pizza 2: CHEESED 
Pizza 3: CHEESED 
Pizza 4: CHEESED 
Pizza 0: TOPPED 
Pizza 2: TOPPED
Pizza 1: TOPPED 
Pizza 3: TOPPED 
Pizza 4: TOPPED 
Pizza 1: BAKED 
Pizza 2: BAKED 
Pizza 0: BAKED 
Pizza 4: BAKED 
Pizza 3: BAKED 
Pizza 0: SLICED 
Pizza 2: SLICED 
Pizza 1: SLICED 
Pizza 3: SLICED 
Pizza 4: SLICED 
Pizza 1: BOXED 
Pizza1: complete 
Pizza 2: BOXED 
Pizza 0: BOXED 
Pizza2: complete 
Pizza0: complete 
Pizza 3: BOXED
Pizza 4: BOXED 
Pizza4: complete 
Pizza3: complete 
1738 
```

答案是“否”，事後看來這並不奇怪，因為每個披薩都需要按順序執行步驟。因此，沒辦法透過分步執行操作來進一步提高速度，就像上文的 `PizzaParallelSteps.java` 裡面展示的一樣。

我們可以使用 **CompletableFutures**  重寫這個例子：

```java
// concurrent/CompletablePizza.java 

import java.util.*;
import java.util.concurrent.*;
import java.util.stream.*;
import onjava.Timer;

public class CompletablePizza{
    static final int QUANTITY = 5;

    public static CompletableFuture<Pizza> makeCF(Pizza za){
        return CompletableFuture
                .completedFuture(za)
            .thenApplyAsync(Pizza::roll)
            .thenApplyAsync(Pizza::sauce)
            .thenApplyAsync(Pizza::cheese)
            .thenApplyAsync(Pizza::toppings)
            .thenApplyAsync(Pizza::bake)
            .thenApplyAsync(Pizza::slice)
            .thenApplyAsync(Pizza::box);
    }

    public static void show(CompletableFuture<Pizza> cf){
        try{
            System.out.println(cf.get());
        } catch (Exception e){
            throw new RuntimeException(e);
        }
    }

    public static void main(String[] args){
        Timer timer = new Timer();
        List<CompletableFuture<Pizza>> pizzas =
                IntStream.range(0, QUANTITY)
            .mapToObj(Pizza::new)
            .map(CompletablePizza::makeCF)
            .collect(Collectors.toList());
        System.out.println(timer.duration());
        pizzas.forEach(CompletablePizza::show);
        System.out.println(timer.duration());
    }
}
```

輸出結果：

```
169 
Pizza 0: ROLLED 
Pizza 1: ROLLED 
Pizza 2: ROLLED 
Pizza 4: ROLLED 
Pizza 3: ROLLED 
Pizza 1: SAUCED 
Pizza 0: SAUCED 
Pizza 2: SAUCED 
Pizza 4: SAUCED
Pizza 3: SAUCED 
Pizza 0: CHEESED 
Pizza 4: CHEESED 
Pizza 1: CHEESED 
Pizza 2: CHEESED 
Pizza 3: CHEESED 
Pizza 0: TOPPED 
Pizza 4: TOPPED 
Pizza 1: TOPPED 
Pizza 2: TOPPED 
Pizza 3: TOPPED 
Pizza 0: BAKED 
Pizza 4: BAKED 
Pizza 1: BAKED 
Pizza 3: BAKED 
Pizza 2: BAKED 
Pizza 0: SLICED 
Pizza 4: SLICED 
Pizza 1: SLICED 
Pizza 3: SLICED
Pizza 2: SLICED 
Pizza 4: BOXED 
Pizza 0: BOXED 
Pizza0: complete 
Pizza 1: BOXED 
Pizza1: complete 
Pizza 3: BOXED 
Pizza 2: BOXED 
Pizza2: complete 
Pizza3: complete 
Pizza4: complete 
1797 
```

並行流和 **CompletableFutures**  是 Java 並發工具箱中最先出發達的技術。 你應該始終首先選擇其中之一。 當一個問題很容易平行處理時，或者說，很容易把資料分解成相同的、易於處理的各個部分時，使用並行流方法處理最為合適（而如果你決定不借助它而由自己完成，你就必須擼起袖子，深入研究 **Spliterator** 的文件）。

而當工作的各個部分內容各不相同時，使用 **CompletableFutures**  是最好的選擇。比起面向資料，**CompletableFutures**  更像是面向任務的。

對於披薩問題，結果似乎也沒有什麼不同。實際上，並行流方法看起來更簡潔，僅出於這個原因，我認為並行流作為解決問題的首次嘗試方法更具吸引力。

由於製作披薩總需要一定的時間，無論你使用哪種並發方法，你能做到的最好情況，是在製作一個披薩的相同時間內製作 n 個披薩。 在這裡當然很容易看出來，但是當你處理更複雜的問題時，你就可能忘記這一點。 通常，在項目開始時進行粗略的計算，就能很快弄清楚最大可能的並行吞吐量，這可以防止你因為採取無用的加快執行速度的舉措而忙得團團轉。

使用 **CompletableFutures**  或許可以輕易地帶來重大收益，但是在嘗試更進一步時需要倍加小心，因為額外增加的成本和工作量會非常容易遠遠超出你之前拚命擠出的那一點點收益。

## 本章小結

需要並發的唯一理由是“等待太多”。這也可以包括使用者介面的響應速度，但是由於 Java 用於構建使用者介面時並不高效，因此[^8] 這僅僅意味著“你的程式執行速度還不夠快”。

如果並發很容易，則沒有理由拒絕並發。 正因為並發實際上很難，所以你應該仔細考慮是否值得為此付出努力，並考慮你能否以其他方式提升速度。

例如，遷移到更快的硬體（這可能比消耗程式設計師的時間要便宜得多）或者將程式分解成多個部分，然後在不同的機器上執行這些部分。

奧卡姆剃刀是一個經常被誤解的原則。 我看過至少一部電影，他們將其定義為”最簡單的解決方案是正確的解決方案“，就好像這是某種毋庸置疑的法律。實際上，這是一個準則：面對多種方法時，請先嘗試需要最少假設的方法。 在編程世界中，這已演變為“嘗試可能可行的最簡單的方法”。當你了解了特定工具的知識時——就像你現在了解了有關並發性的知識一樣，你可能會很想使用它，或者提前規定你的解決方案必須能夠“速度飛快”，從而來證明從一開始就進行並發設計是合理的。但是，我們的奧卡姆剃刀編程版本表示你應該首先嘗試最簡單的方法（這種方法開發起來也更便宜），然後看看它是否足夠好。

由於我出身於底層學術背景（物理學和電腦工程），所以我很容易想到所有小輪子轉動的成本。我確定使用最簡單的方法不夠快的場景出現的次數已經數不過來了，但是嘗試後卻發現它實際上綽綽有餘。

### 缺點

並發編程的主要缺點是：

1. 在執行緒等待共享資源時會降低速度。 

2. 執行緒管理產生額外 CPU 開銷。

3. 糟糕的設計決策帶來無法彌補的複雜性。

4. 諸如飢餓，競速，死鎖和活鎖（多執行緒各自處理單個任務而整體卻無法完成）之類的問題。

5. 跨平台的不一致。 透過一些範例，我發現了某些電腦上很快出現的競爭狀況，而在其他電腦上卻沒有。 如果你在後者上開發程式，則在分發程式時可能會感到非常驚訝。

另外，並發的應用是一門藝術。 Java 旨在允許你建立儘可能多的所需要的物件來解決問題——至少在理論上是這樣。[^9] 但是，執行緒不是典型的物件：每個執行緒都有其自己的執行環境，包括堆疊和其他必要的元素，使其比普通物件大得多。 在大多數環境中，只能在記憶體用光之前建立數千個 **Thread** 物件。通常，你只需要幾個執行緒即可解決問題，因此一般來說建立執行緒沒有什麼限制，但是對於某些設計而言，它會成為一種約束，可能迫使你使用完全不同的方案。

### 共享記憶體陷阱

並發性的主要困難之一是因為可能有多個任務共享一個資源（例如物件中的記憶體），並且你必須確保多個任務不會同時讀取和更改該資源。

我花了多年的時間研究並發。 我了解到你永遠無法相信使用共享記憶體並發的程式可以正常工作。 你可以輕易發現它是錯誤的，但永遠無法證明它是正確的。 這是眾所周知的並發原則之一。[^10]

我遇到了許多人，他們對編寫正確的執行緒程式的能力充滿信心。 我偶爾開始認為我也可以做好。 對於一個特定的程式，我最初是在只有單個 CPU 的機器上編寫的。 那時我能夠說服自己該程式是正確的，因為我以為我對 Java 工具很了解。 而且在我的單 CPU 電腦上也沒有失敗。而到了具有多個 CPU 的電腦，程式出現問題不能執行後，我感到很驚訝，但這還只是眾多問題中的一個而已。 這不是 Java 的錯； “寫一次，到處執行”，在單核與多核電腦間無法擴展到並發編程領域。這是並發編程的基本問題。 實際上你可以在單 CPU 機器上發現一些並發問題，但是在多執行緒實際上真的在併行執行的多 CPU 機器上，就會出現一些其他問題。

再舉一個例子，哲學家就餐的問題可以很容易地進行調整，因此幾乎不會產生死鎖，這會給你一種一切都棒極了的印象。當涉及到共享記憶體並發編程時，你永遠不應該對自己的程式能力變得過於自信。

### This Albatross is Big

如果你對 Java 並發感到不知所措，那說明你身處在一家出色的公司裡。你可以訪問 **Thread** 類的[Javadoc](https://docs.oracle.com/javase/8/docs/api/java/lang/Thread.html) 頁面， 看一下哪些方法現在是 **Deprecated** （廢棄的）。這些是 Java 語言設計者犯過錯的地方，因為他們在設計語言時對並發性了解不足。

事實證明，在 Java 的後續版本中添加的許多庫解決方案都是無效的，甚至是無用的。 幸運的是，Java 8 中的並行 **Streams** 和 **CompletableFutures** 都非常有價值。但是當你使用舊程式碼時，仍然會遇到舊的解決方案。

在本書的其他地方，我談到了 Java 的一個基本問題：每個失敗的實驗都永遠嵌入在語言或庫中。 Java 並發強調了這個問題。儘管有不少錯誤，但錯誤並不是那麼多，因為有很多不同的嘗試方法來解決問題。 好的方面是，這些嘗試產生了更好，更簡單的設計。 不利之處在於，在找到好的方法之前，你很容易迷失於舊的設計中。

### 其他類庫

本章重點介紹了相對安全易用的並行工具流和 **CompletableFutures** ，並且僅涉及 Java 標準庫中一些更細粒度的工具。 為避免你不知所措，我沒有介紹你可能實際在實踐中使用的某些庫。我們使用了幾個 **Atomic** （原子）類，**ConcurrentLinkedDeque** ，**ExecutorService** 和 **ArrayBlockingQueue** 。附錄：[並發底層原理 ](./Appendix-Low-Level-Concurrency.md) 涵蓋了其他一些內容，但是你還想探索 **java.util.concurrent** 的 Javadocs。 但是要小心，因為某些庫元件已被新的更好的元件所取代。

### 考慮為並發設計的語言

通常，請謹慎地使用並發。 如果需要使用它，請嘗試使用最現代的方法：並行流或 **CompletableFutures** 。 這些功能旨在（假設你不嘗試共享記憶體）使你擺脫麻煩（在 Java 的世界範圍內）。

如果你的並發問題變得比進階 Java 構造所支援的問題更大且更複雜，請考慮使用專為並發設計的語言，僅在需要並發的程式部分中使用這種語言是有可能的。 在撰寫本文時，JVM 上最純粹的功能語言是 Clojure（Lisp 的一種版本）和 Frege（Haskell 的一種實現）。這些使你可以在其中編寫應用程式的並發部分語言，並透過 JVM 輕鬆地與你的主要 Java 程式碼進行互動。 或者，你可以選擇更複雜的方法，即透過外部功能介面（FFI）將 JVM 之外的語言與另一種為並發設計的語言進行通信。[^11]

你很容易被一種語言綁定，迫使自己嘗試使用該語言來做所有事情。 一個常見的範例是構建 HTML / JavaScript 使用者介面。 這些工具確實很難使用，令人討厭，並且有許多庫允許你透過使用自己喜歡的語言編寫程式碼來生成這些工具（例如，**Scala.js** 允許你在 Scala 中完成程式碼）。

心理上的便利是一個合理的考慮因素。 但是，我希望我在本章（以及附錄：[並發底層原理 ](./Appendix-Low-Level-Concurrency.md)）中已經表明 Java 並發是一個你可能無法逃離很深的洞。 與 Java 語言的任何其他部分相比，在視覺上檢查程式碼同時記住所有陷阱所需要的的知識要困難得多。

無論使用特定的語言、庫使得並發看起來多麼簡單，都要將其視為一種妖術，因為總是有東西會在你最不期望出現的時候咬你。

### 擴展閱讀

《Java Concurrency in Practice》，出自 Brian Goetz，Tim Peierls， Joshua Bloch，Joseph Bowbeer，David Holmes 和 Doug Lea (Addison Wesley，2006 年)——這些基本上就是 Java 並發世界中的名人名單了《Java Concurrency in Practice》第二版，出自 Doug Lea (Addison-Wesley，2000 年)。儘管這本書出版時間遠遠早於 Java 5 發布，但 Doug 的大部分工作都寫入了 **java.util.concurrent** 庫。因此，這本書對於全面理解並發問題至關重要。 它超越了 Java，討論了跨語言和技術的並發編程。 儘管它在某些地方可能很鈍，但值得多次重讀（最好是在兩個月之間進行消化）。 道格（Doug）是世界上為數不多的真正了解並發編程的人之一，因此這是值得的。

[^1]:例如,Eric-Raymond 在“Unix 編程藝術”（Addison-Wesley，2004）中提出了一個很好的案例。
[^2]:可以說，試圖將並發性用於後續語言是一種注定要失敗的方法，但你必須得出自己的結論
[^3]:有人談論在 Java——10 中圍繞泛型做一些類似的基本改進，這將是非常令人難以置信的。
[^4]:這是一種有趣的，雖然不一致的方法。通常，我們期望在公共介面上使用顯式類表示不同的行為
[^5]:不，永遠不會有純粹的功能性 Java。我們所能期望的最好的是一種在 JVM 上執行的全新語言。
[^6]:當兩個任務能夠更改其狀態以使它們不會被阻止但它們從未取得任何有用的進展時，你也可以使用活動鎖。
[^7]: 而不是超執行緒；通常每個核心有兩個超執行緒，並且在詢問核心數量時，本書所使用的 Java 版本會報告超執行緒的數量。超執行緒產生了更快的上下文切換，但是只有實際的核心才真的工作，而不是超執行緒。 ↩
[^8]: 庫就在那裡用於呼叫，而語言本身就被設計用於此目的，但實際上它很少發生，以至於可以說”沒有“。↩
[^9]: 舉例來說，如果沒有 Flyweight 設計模式，在工程中建立數百萬個物件用於有限元分析可能在 Java 中不可行。↩
[^10]: 在科學中，雖然從來沒有一種理論被證實過，但是一種理論必須是可證偽的才有意義。而對於並發性，我們大部分時間甚至都無法得到這種可證偽性。↩
[^11]: 儘管 **Go** 語言顯示了 FFI 的前景，但在撰寫本文時，它並未提供跨所有平台的解決方案。

<!-- 分頁 -->

<div style="page-break-after: always;"></div>
