﻿[TOC]

<!-- Appendix: Programming Guidelines -->
# 附錄:程式指南

> 本附錄包含了有助於指導你進行低級程式設計和編寫程式碼的建議。

當然，這些只是指導方針，而不是規則。我們的想法是將它們用作靈感，並記住偶爾會違反這些指導方針的特殊情況。

<!-- Design -->
## 設計

1. **優雅總是會有回報**。從短期來看，似乎需要更長的時間才能找到一個真正優雅的問題解決方案，但是當該解決方案第一次應用並能輕鬆適應新情況，而不需要數小時，數天或數月的掙扎時，你會看到獎勵（即使沒有人可以測量它們）。它不僅為你提供了一個更容易構建和除錯的程式，而且它也更容易理解和維護，這也正是經濟價值所在。這一點可以透過一些經驗來理解，因為當你想要使一段程式碼變得優雅時，你可能看起來效率不是很高。抵制急於求成的衝動，它只會減慢你的速度。

2. **先讓它工作，然後再讓它變快**。即使你確定一段程式碼非常重要並且它是你系統中的主要瓶頸**，也是如此。不要這樣做。使用儘可能簡單的設計使系統首先執行。然後如果速度不夠快，請對其進行分析。你幾乎總會發現“你的”瓶頸不是問題。節省時間，才是真正重要的東西。

3. **記住“分而治之”的原則**。如果所面臨的問題太過混亂**，就去想像一下程式的基本操作，因為存在一個處理困難部分的神奇“片段”（piece）。該“片段”是一個物件，編寫使用該物件的程式碼，然後查看該物件並將其困難部分封裝到其他物件中，等等。

4. **將類建立者與類使用者（用戶端程式設計師）分開**。類使用者是“客戶”，不需要也不想知道類幕後發生了什麼事。類建立者必須是設計類的專家，他們編寫類，以便新手程式設計師都可以使用它，並仍然可以在應用程式中穩健地工作。將該類視為其他類的*服務提供者*（service provider）。只有對其它類透明，才能很容易地使用這個類。

5. **建立類時，給類取個清晰的名字，就算不需要注釋也能理解這個類**。你的目標應該是使用戶端程式設計師的介面在概念上變得簡單。為此，在適當時使用方法重載來建立直觀，易用的介面。

6. **你的分析和設計必須至少能夠產生系統中的類、它們的公共介面以及它們與其他類的關係，尤其是基類**。 如果你的設計方法產生的不止於此，就該問問自己，該方法生成的所有部分是否在程式的生命週期內都具有價值。如果不是，那麼維護它們會很耗費精力。對於那些不會影響他們生產力的東西，開發團隊的成員往往不會去維護，這是許多設計方法都沒有考慮的生活現實。

7. **讓一切自動化**。首先在編寫類之前，編寫測試程式碼，並將其與類保持一致。透過構建工具自動執行測試。你可能會使用事實上的標準Java構建工具Gradle。這樣，透過執行測試程式碼可以自動驗證任何更改，將能夠立即發現錯誤。因為你知道自己擁有測試框架的安全網，所以當發現需要時，可以更大膽地進行徹底的更改。請記住，語言的巨大改進來自內建的測試，包括類型檢查，異常處理等，但這些內建功能很有限，你必須完成剩下的工作，針對具體的類或程式，去完善這些測試內容，從而建立一個強大的系統。

8. **在編寫類之前，先編寫測試程式碼，以驗證類的設計是完善的**。如果不編寫測試程式碼，那麼就不知道類是什麼樣的。此外，透過編寫測試程式碼，往往能夠激發出類中所需的其他功能或約束。而這些功能或約束並不總是出現在分析和設計過程中。測試還會提供範例程式碼，顯示了如何使用這個類。

9. **所有的軟體設計問題，都可以透過引入一個額外的間接概念層次（extra level of conceptual indirection）來解決**。這個軟體工程的基本規則[^1]是抽象的基礎，是物件導向編程的主要特徵。在物件導向編程中，我們也可以這樣說：“如果你的程式碼太複雜，就要生成更多的物件。”

10. **間接（indirection）應具有意義（與準則9一致）**。這個含義可以簡單到“將常用程式碼放在單個方法中。”如果添加沒有意義的間接（抽象，封裝等）級別，那麼它就像沒有足夠的間接性那樣糟糕。

11. **使類儘可能原子化**。 為每個類提供一個明確的目的，它為其他類提供一致的服務。如果你的類或系統設計變得過於複雜，請將複雜類分解為更簡單的類。最直觀的指標是尺寸大小，如果一個類很大，那麼它可能是做的事太多了，應該被分割。建議重新設計類的線索是：
    - 一個複雜的*switch*語句：考慮使用多態。
    - 大量方法涵蓋了很多不同類型的操作：考慮使用多個類。
    - 大量成員變數涉及很多不同的特徵：考慮使用多個類。
    - 其他建議可以參見Martin Fowler的*Refactoring: Improving the Design of Existing Code*（重構：改善既有程式碼的設計）（Addison-Wesley 1999）。

12. **注意長參數列表**。那樣方法呼叫會變得難以編寫，讀取和維護。相反，嘗試將方法移動到更合適的類，並且（或者）將物件作為參數傳遞。

13. **不要重複自己**。如果一段程式碼出現在衍生類的許多方法中，則將該程式碼放入基類中的單個方法中，並從衍生類方法中呼叫它。這樣不僅可以節省程式碼空間，而且可以輕鬆地傳播更改。有時，發現這個通用程式碼會為介面添加有價值的功能。此指南的更簡單版本也可以在沒有繼承的情況下發生：如果類具有重複程式碼的方法，則將該重複程式碼放入一個公共方，法並在其他方法中呼叫它。

14. **注意*switch*語句或鏈式*if-else*子句**。一個*類型檢查編碼*（type-check coding）的指示器意味著需要根據某種類型訊息選擇要執行的程式碼（確切的類型最初可能不明顯）。很多時候可以用繼承和多態取代這種程式碼，多態方法呼叫將會執行類型檢查，並提供了更可靠和更容易的可擴展性。 

15. **從設計的角度，尋找和分離那些因不變的事物而改變的事物**。也就是說，在不強制重新設計的情況下搜尋可能想要更改的系統中的元素，然後將這些元素封裝在類中。

16. **不要透過子類擴展基本功能**。如果一個介面元素對於類來說是必不可少的，則它應該在基類中，而不是在衍生期間添加。如果要在繼承期間添加方法，請考慮重新設計。

17. **少即是多**。從一個類的最小介面開始，儘可能小而簡單，以解決手頭的問題，但不要試圖預測類的所有使用方式。在使用該類時，就將會了解如何擴展介面。但是，一旦這個類已經在使用了，就無法在不破壞用戶端程式碼的情況下縮小介面。如果必須添加更多方法，那很好，它不會破壞程式碼。但即使新方法取代舊方法的功能，也只能是保留現有介面（如果需要，可以結合底層實現中的功能）。如果必須透過添加更多參數來擴展現有方法的介面，請使用新參數建立重載方法，這樣，就不會影響到對現有方法的任何呼叫。

18. **大聲讀出你的類以確保它們合乎邏輯**。將基類和衍生類之間的關係稱為“is-a”，將成員物件稱為“has-a”。

19. **在需要在繼承和組合之間作決定時，問一下自己是否必須向上轉換為基類型**。如果不是，則使用組合（成員物件）更好。這可以消除對多種基類型的感知需求（perceived need）。如果使用繼承，則使用者會認為他們應該向上轉型。

20. **注意重載**。方法不應該基於參數的值而有條件地執行程式碼。在這裡，應該建立兩個或多個重載方法。

21. **使用異常層次結構**，最好是從標準Ja​​va異常層次結構中的特定適當類衍生。然後，捕獲異常的人可以為特定類型的異常編寫處理程序，然後為基類型編寫處理程序。如果添加新的衍生異常，現有用戶端程式碼仍將透過基類型捕獲異常。

22. **有時簡單的聚合可以完成工作**。航空公司的“乘客舒適系統”由獨立的元素組成：座位，空調，影視等，但必須在飛機上建立許多這樣的元素。你建立私有成員並建立一個全新的介面了嗎？如果不是，在這種情況下，元件也應該是公共介面的一部分，因此應該建立公共成員物件。這些物件有自己的私有實現，這些實現仍然是安全的。請注意，簡單聚合不是經常使用的解決方案，但確實會有時候會用到。

23. **考慮客戶程式設計師和維護程式碼的人的觀點**。設計類以便儘可能直觀地被使用。預測要進行的更改，並精心設計類，以便輕鬆地進行更改。

24. **注意“巨型物件症候群”**（giant object syndrome）。這通常是程式設計師的痛苦，他們是物件導向編程的新手，總是編寫程序導向程式並將其貼上在一個或兩個巨型物件中。除應用程式框架外，物件代表應用程式中的概念，而不是應用程式本身。

25. **如果你必須做一些醜陋的事情，至少要把類內的醜陋本地化**。

26. **如果必須做一些不可移植的事情，那就對這件事情做一個抽象，並在一個類中進行本地化**。這種額外的間接級別可防止在整個程式中擴散這種不可移植性。 （這個原則也體現在*橋接*模式中，等等）。

27. **物件不應該僅僅只是持有一些資料**。它們也應該有明確的行為。有時候，“資料傳輸物件”（data transfer objects）是合適的，但只有在泛型集合不合適時，才被明確用於打包和傳輸一組元素。

28. **在從現有類建立新類時首先選擇組合**。僅在設計需要時才使用繼承。如果在可以使用組合的地方使用繼承，那麼設計將會變得很複雜，這是沒必要的。

29. **使用繼承和覆蓋方法來表達行為的差異，而不是使用欄位來表示狀態的變化**。如果發現一個類使用了狀態變數，並且有一些方法是基於這些變數切換行為的，那麼請重新設計它，以表示子類和覆蓋方法中的行為差異。一個極端的反例是繼承不同的類來表示顏色，而不是使用“顏色”欄位。

30. **注意*協變*（variance）**。兩個語義不同的物件可能具有相同的操作或職責。為了從繼承中受益，會試圖讓其中一個成為另一個的子類，這是一種很自然的誘惑。這稱為協變，但沒有真正的理由去強制聲明一個並不存在的父子類關係。更好的解決方案是建立一個通用基類，並為兩者生成一個介面，使其成為這個通用基類的衍生類。這仍然可以從繼承中受益，並且這可能是關於設計的一個重要發現。

31. **在繼承期間注意*限定*（limitation）**。最明確的設計為繼承的類增加了新的功能。含糊的設計在繼承期間刪除舊功能而不添加新功能。但是規則是用來打破的，如果是透過呼叫一個舊的類庫來工作，那麼將一個現有類限制在其子類型中，可能比重構層次結構更有效，因此新類適合在舊類的上層。

32. **使用設計模式來消除“裸功能”（naked functionality）**。也就是說，如果類只需要建立一個物件，請不要推進應用程式並寫下注釋“只生成一個。”應該將其包裝成一個單例（singleton）。如果主程式中有很多亂七八糟的程式碼去建立物件，那麼找一個像工廠方法一樣的建立模式，可以在其中封裝建立過程。消除“裸功能”不僅會使程式碼更易於理解和維護，而且還會使其能夠更加防範應對後面的善意維護者（well-intentioned maintainers）。

33. **注意“分析癱瘓”（analysis paralysis）**。記住，不得不經常在不了解整個項目的情況下推進項目，並且通常了解那些未知因素的最好、最快的方式是進入下一步而不是嘗試在腦海中弄清楚。在獲得解決方案之前，往往無法知道解決方案。Java有內建的防火牆，讓它們為你工作。你在一個類或一組類中的錯誤不會破壞整個系統的完整性。

34. **如果認為自己有很好的分析，設計或實施，請做一個演練**。從團隊外部帶來一些人，不一定是顧問，但可以是公司內其他團體的人。用一雙新眼睛評審你的工作，可以在一個更容易修復它們的階段發現問題，而不僅僅是把大量時間和金錢全扔到演練過程中。

<!-- Implementation -->
## 實現

36.  **遵循編碼慣例**。有很多不同的約定，例如，[Google使用的約定](https://google.github.io/styleguide/javaguide.html)（本書中的程式碼儘可能地遵循這些約定）。如果堅持使用其他語言的編碼風格，那麼讀者就會很難去閱讀。無論決定採用何種編碼約定，都要確保它們在整個項目中保持一致。整合開發環境通常包含內建的重新格式化（reformatter）和檢查器（checker）。

37. **無論使用何種編碼風格，如果你的團隊（甚至更好是公司）對其進行標準化，它就確實會產生重大影響**。這意味著，如果不符合這個標準，那麼每個人都認為修復別人的編碼風格是公平的遊戲。標準化的價值在於解析程式碼可以花費較少的腦力，因此可以更專注於程式碼的含義。

38. **遵循標準的大寫規則**。類名的第一個字母大寫。欄位，方法和物件（引用）的第一個字母應為小寫。所有標識符應該將各個單詞組合在一起，並將所有中間單詞的首字母大寫。例如：

    - **ThisIsAClassName**
    - **thisIsAMethodOrFieldName**

    將 **static final** 類型的標識符的所有字母全部大寫，並用下劃線分隔各個單詞，這些標識符在其定義中具有常量初始值。這表明它們是編譯時常量。
    - **包是一個特例**，它們都是小寫的字母，即使是中間詞。域擴展（com，org，net，edu等）也應該是小寫的。這是Java 1.1和Java 2之間的變化。

39. **不要建立自己的“裝飾”私有欄位名稱**。這通常以前置下劃線和字元的形式出現。匈牙利命名法（譯者註：一種命名規範，基本原則是：變數名=屬性+類型+物件描述。Win32程式風格採用這種命名法，如`WORD wParam1;LONG lParam2;HANDLE hInstance`）是最糟糕的例子，你可以在其中附加額外字元用於指示資料類型，用途，位置等，就好像你正在編寫組語語言一樣，編譯器根本沒有提供額外的幫助。這些符號令人困惑，難以閱讀，並且難以執行和維護。讓類和包來指定名稱範圍。如果認為必須裝飾名稱以防止混淆，那麼程式碼就可能過於混亂，這應該被簡化。

40. 在建立一般用途的類時，**遵循“規範形式”**。包括**equals()**，**hashCode()**，**toString()**，**clone()**的定義（實現**Cloneable**，或選擇其他一些物件複製方法，如序列化），並實現**Comparable**和**Serializable**。

41. **對讀取和更改私有欄位的方法使用“get”，“set”和“is”命名約定**。這種做法不僅使類易於使用，而且也是命名這些方法的標準方法，因此讀者更容易理解。

42. **對於所建立的每個類，請包含該類的JUnit測試**（請參閱*junit.org*以及[第十六章：程式碼校驗]()中的範例）。無需刪除測試程式碼即可在項目中使用該類，如果進行更改，則可以輕鬆地重新執行測試。測試程式碼也能成為如何使用這個類的範例。

43. **有時需要繼承才能訪問基類的protected成員**。這可能導致對多種基類型的感知需求（perceived need）。如果不需要向上轉型，則可以首先衍生一個新類來執行受保護的訪問。然後把該新類作為使用它的任何類中的成員物件，以此來代替直接繼承。

44. **為了提高效率，避免使用*final*方法**。只有在分析後發現方法呼叫是瓶頸時，才將**final**用於此目的。

45. **如果兩個類以某種功能方式相互關聯（例如集合和疊代器），則嘗試使一個類成為另一個類的內部類**。這不僅強調了類之間的關聯，而且透過將類嵌套在另一個類中，可以允許在單個包中重用類名。Java集合庫透過在每個集合類中定義內部**Iterator**類來實現此目的，從而為集合提供通用介面。使用內部類的另一個原因是作為**私有**實現的一部分。這裡，內部類將有利於實現隱藏，而不是上面提到的類關聯和防止命名空間汙染。

46. **只要你注意到類似乎彼此之間具有高耦合，請考慮如果使用內部類可能獲得的編碼和維護改進**。內部類的使用不會解耦類，而是明確耦合關係，並且更方便。

47. **不要成為過早最佳化的犧牲品**。過早最佳化是很瘋狂的行為。特別是，不要擔心編寫（或避免）本機方法（native methods），將某些方法設定為**final**，或者在首次構建系統時調整程式碼以使其高效。你的主要目標應該是驗證設計。即使設計需要一定的效率，也*先讓它工作，然後再讓它變快*。

48. **保持作用域儘可能小，以便能見度和物件的壽命儘可能小**。這減少了在錯誤的上下文中使用物件並隱藏了難以發現的bug的機會。例如，假設有一個集合和一段疊代它的程式碼。如果複製該程式碼以用於一個新集合，那麼可能會意外地將舊集合的大小用作新集合的疊代上限。但是，如果舊集合比較大，則會在編譯時捕獲錯誤。

49. **使用標準Java庫中的集合**。熟練使用它們，將會大大提高工作效率。首選**ArrayList**用於序列，**HashSet**用於集合，**HashMap**用於關聯陣列，**LinkedList**用於堆疊（而不是**Stack**，儘管也可以建立一個適配器來提供堆疊介面）和佇列（也可以使用適配器，如本書所示）。當使用前三個時，將其分別向上轉型為**List**，**Set**和**Map**，那麼就可以根據需要輕鬆更改為其他實現。

50. **為使整個程式健壯，每個元件必須健壯**。在所建立的每個類中，使用Java所提供的所有工具，如訪問控制，異常，類型檢查，同步等。這樣，就可以在構建系統時安全地進入下一級抽象。

51. **編譯時錯誤優於執行時錯誤**。嘗試儘可能在錯誤發生點處理錯誤。在最近的處理程序中盡其所能地捕獲它能處理的所有異常。在目前層面處理所能處理的所有異常，如果解決不了，就重新拋出異常。 

52. **注意長方法定義**。方法應該是簡短的功能單元，用於描述和實現類介面的離散部分。維護一個冗長而複雜的方法是很困難的，而且代價很大，並且這個方法可能是試圖做了太多事情。如果看到這樣的方法，這表明，至少應該將它分解為多種方法。也可能建議去建立一個新類。小的方法也可以促進類重用。（有時方法必須很大，但它們應該只做一件事。）

53. **保持“儘可能私有”**。一旦公開了你的類庫中的一個方面（一個方法，一個類，一個欄位），你就永遠無法把它拿回來。如果這樣做，就將破壞某些人的現有程式碼，迫使他們重寫和重新設計。如果你只公開了必須公開的內容，就可以輕易地改變其他一切，而不會對其他人造成影響，而且由於設計趨於發展，這是一個重要的自由。透過這種方式，更改具體實現將對衍生類造成的影響最小。在處理多執行緒時，私有尤其重要，只有**私有**欄位可以防止不同步使用。具有包訪問權限的類應該仍然具有**私有**欄位，但通常有必要提供包訪問權限的方法而不是將它們**公開**。

54. **大量使用注釋，並使用*Javadoc commentdocumentation*語法生成程式文件**。但是，注釋應該為程式碼增加真正的意義，如果注釋只是重申了程式碼已經清楚表達的內容，這是令人討厭的。請注意，Java類和方法名稱的典型詳細訊息減少了對某些注釋的需求。

55. **避免使用“魔法數字”**。這些是指寫死到程式碼中的數字。如果後續必須要更改它們，那將是一場噩夢，因為你永遠不知道“100”是指“陣列大小”還是“完全不同的東西”。相反，建立一個帶有描述性名稱的常量並在整個程式中使用常量標識符。這使程式更易於理解，更易於維護。

56. **在建立構造方法時，請考慮異常**。最好的情況是，構造方法不會做任何拋出異常的事情。次一級的最佳方案是，該類僅由健壯的類組成或繼承自健壯的類，因此如果拋出異常則不需要處理。否則，必須清除**finally**子句中的組合類。如果構造方法必然失敗，則適當的操作是拋出異常，因此呼叫者不會認為該物件是正確建立的而盲目地繼續下去。

57. **在構造方法內部，只需要將物件設定為正確的狀態**。主動避免呼叫其他方法（**final**方法除外），因為這些方法可以被其他人覆蓋，從而在構造期間產生意外結果。（有關詳細訊息，請參閱[第六章：初始化和清理]()章節。）較小，較簡單的構造方法不太可能拋出異常或導致問題。

58. **如果類在用戶端程式設計師用完物件時需要進行任何清理，請將清理程式碼放在一個明確定義的方法中**，並使用像 **dispose()** 這樣的名稱來清楚地表明其目的。另外，在類中放置一個 **boolean** 標誌來指示是否呼叫了 **dispose()** ，因此 **finalize()** 可以檢查“終止條件”（參見[第六章：初始化和清理]()章節）。

59. ***finalize()* 的職責只能是驗證物件的“終止條件”以進行除錯**。（參見[第六章：初始化和清理]()一章）在特殊情況下，可能需要釋放垃圾收集器無法釋放的記憶體。因為可能無法為物件呼叫垃圾收集器，所以無法使用 **finalize()** 執行必要的清理。為此，必須建立自己的 **dispose()** 方法。在類的 **finalize()** 方法中，檢查以確保物件已被清理，如果沒有被清理，則拋出一個衍生自**RuntimeException**的異常，以指示編程錯誤。在依賴這樣的計劃之前，請確保 **finalize()** 適用於你的系統。（可能需要呼叫 **System.gc()** 來確保此行為。）

60. **如果必須在特定範圍內清理物件（除了透過垃圾收集器），請使用以下準則：** 初始化物件，如果成功，立即進入一個帶有 **finally** 子句的 **try** 塊，並在 **finally**中執行清理操作。

61. **在繼承期間覆蓋 *finalize()* 時，記得呼叫 *super.finalize()***。（如果是直接繼承自 **Object** 則不需要這樣做。）呼叫 **super.finalize()** 作為重寫的 **finalize()** 的最終行為而不是在第一行呼叫它，這樣可以確保基類元件在需要時仍然有效。

62. **建立固定大小的物件集合時，將它們轉換為陣列，** 尤其是在從方法中返回此集合時。這樣就可以獲得陣列編譯時類型檢查的好處，並且陣列的接收者可能不需要在陣列中強制轉換物件來使用它們。請注意，集合庫的基類 **java.util.Collection** 有兩個 **toArray()** 方法來完成此任務。

63. **優先選擇 *介面* 而不是 *抽象類***。如果知道某些東西應該是基類，那麼第一選擇應該是使其成為一個介面，並且只有在需要方法定義或成員變數時才將其更改為抽象類。一個介面關心用戶端想要做什麼，而一個類傾向於關注（或允許）實現細節。

64. **為了避免非常令人沮喪的經歷，請確保類路徑中的每個名稱只對應一個不在包中的類**。否則，編譯器可以首先找到具有相同名稱的其他類，並報告沒有意義的錯誤消息。如果你懷疑自己有類路徑問題，請嘗試在類路徑的每個起始點尋找具有相同名稱的 **.class** 文件。理想情況下，應該將所有類放在包中。

65. **注意意外重載**。如果嘗試覆蓋基類方法但是拼寫錯誤，則最終會添加新方法而不是覆蓋現有方法。但是，這是完全合法的，因此在編譯時或執行時不會獲得任何錯誤消息，但程式碼將無法正常工作。始終使用 **@Override** 注釋來防止這種情況。

66. **注意過早最佳化**。先讓它工作，然後再讓它變快。除非發現程式碼的特定部分存在性能瓶頸。除非是使用分析器發現瓶頸，否則過早最佳化會浪費時間。性能調整所隱藏的額外成本是程式碼將變得難以理解和維護。

67. **請注意，相比於編寫程式碼，程式碼被閱讀的機會更多**。清晰的設計可能產生易於理解的程式，但注釋，詳細解釋，測試和範例是非常寶貴的，它們可以幫助你和你的所有後繼者。如果不出意外，試圖從JDK文件中找出有用訊息的挫敗感應該可以說服你。

[^1]: Andrew Koenig向我解釋了它。

<!-- 分頁 -->
<div style="page-break-after: always;"></div>
