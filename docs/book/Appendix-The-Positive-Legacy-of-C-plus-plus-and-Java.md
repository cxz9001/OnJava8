﻿[TOC]

<!-- Appendix: The Positive Legacy of C++ and Java -->
# 附錄:C++和Java的優良傳統

> 在各種討論聲中，有一些人認為C++是一種設計糟糕的語言。 我認為理解C++和Java語言的選擇有助於了解更大的視角。

也就是說，我幾乎不再使用C++了。當我使用它的時候，要嘛是用來檢查遺留程式碼，要嘛是編寫性能關鍵（performance-critical）部分，程式通常儘可能小，以便用其他語言編寫的其他程式來呼叫。

因為我在最初的8年裡一直在C++標準委員會工作，所以我見證了那些被做出的決定。它們都經過了極其謹慎的考慮，遠遠超過了許多在Java中做出的決定。

然而，正如人們正確地指出的那樣，由此產生的語言使用起來既複雜又痛苦，而且只要我一段時間不使用它，我就會忘記那些古怪的規則。在我寫書的時候，我是從第一原理（first principles）處了解這些規則的，而不是記住了它們。

為了理解C++語言為何既令人不愉快且複雜，同時又是精心設計的，必須要牢記C++中所有內容的主要設計決策：與C. Bjarne Stroustrup（該語言的創造者，即“C++之父”）的相容性決定。這樣的設計似乎是為了可以讓大量的C程式設計師透明地轉移到物件（代指C++）上：允許他們在C++下編譯他們的C程式碼。這是一個巨大的限制，一直是C++最大的優勢......而且也是它的禍根。這就是使得C++成功的原因，也是使它複雜的原因。

它也欺騙了那些不太了解C++的Java設計師。例如，他們認為運算符重載對於程式設計師來說很難正確使用。這在C++中基本上是正確的，因為C++既有堆疊分配又有堆分配，你必須重載運算符來處理所有情況而且不要造成記憶體洩漏。這確實很困難。然而，Java有單一的記憶體分配機制和一個垃圾收集器，這使得運算符重載變得微不足道，正如C＃中那樣（但在早於Java的Python中已經可以看到）。但多年來，來自Java團隊的一貫態度是“運算符重載太複雜了”。這裡還有許多決策，所做的事明顯不應該是他們做的。正是由於這些原因，讓我有了蔑視Gosling（即“Java之父”）和Java團隊決策的名聲。（Java 7和8由於某種原因包含了更好的決策。但是向後相容性這個約束總是會阻礙真正的改進。語言永遠不會是它本來的樣子。）

還有很多其他的例子。“為了提高效率，必須包含基本類型”；堅持“萬物皆物件”是正確的；當對性能有要求的時候，提供一個陷阱門（trap door）來做低級別的活動（lower-level activities）（這裡也可以使用hotspot技術透明地提高性能，正如他們最終做的那樣）；不能直接使用浮點處理器去計算超越函數，它用軟體來完成。我已經儘可能多地提出了這樣的問題，但我得到的卻一直是類似“這是Java方式”這樣的回覆。

當我提出關於泛型的設計有多糟糕的時候，我得到了相同的回覆，以及“我們必須向後相容那樣以前用Java做出的決策”（即使它們是糟糕的決策）。最近越來越多的人已經獲得了足夠的泛型經驗，可以發現泛型真的很難用。事實上，C++模板更強大、更一致（現在更容易使用，因為編譯器的錯誤消息是可以容忍的）。人們一直在認真對待物化（reification），這可能是有用的東西，但是在那種被嚴格約束所削弱的設計中並沒有多大影響。

這樣的例子還有很多很多。這是否意味著Java失敗了？絕對不。Java將程式設計師的主流帶入了垃圾收集、虛擬機和一致的錯誤處理模型的世界。由於它的所有缺陷，它將我們提升到了一個水平，現在我們已經準備好使用更進階別的語言了。

有一點，C++是領先的語言，人們認為它總是如此。許多人對Java有同樣的看法，但由於JVM，Java使得取代自己變得更加容易。現在有可能會有人建立一種新語言，並使其在短時間內像Java一樣高效執行。以前，為新語言開發一個正確有效的編譯器需要花費大部分開發時間。

這種情況已經發生了，包括像Scala這樣的進階靜態語言，以及動態語言，新的且可移植的，如Groovy，Clojure，JRuby和Jython。這是未來，並且過渡很順暢，因為可以很輕易地將這些新語言與現有Java程式碼結合使用，並且必要時可以重寫那些在Java中的瓶頸。

在撰寫本文時，Java是世界上的頭號程式語言。然而，Java最終將會減弱，就像C++一樣，淪只在特殊情況下使用（或者只是用來支援傳統的程式碼，因為它不能像C++那樣和硬體連接）。但是無意中的好處，也是Java真正意外的光彩之處在於它為自己的替代品創造了一條非常暢通的道路，即使Java本身已經達到了無法再發展的程度。未來所有的語言都應該從中學習：要嘛建立一個可以重構的文化（像Python和Ruby做的那樣），要嘛就讓競爭者茁壯成長。

<!-- 分頁 -->
<div style="page-break-after: always;"></div>
