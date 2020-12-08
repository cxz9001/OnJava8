[TOC]

<!-- Appendix: Passing and Returning Objects -->
# 附錄:物件傳遞和返回

> 到現在為止，你已經對“傳遞”物件實際上是傳遞引用這一想法想法感到滿意。

在許多程式語言中，你可以使用該語言的“一般”方式來傳遞物件，並且大多數情況下一切正常。 但是通常會出現這種情況，你必須做一些不平常的事情，突然事情變得更加複雜。 Java也不例外，當您傳遞物件並對其進行操作時，準確了解正在發生的事情很重要。 本附錄提供了這種見解。

提出本附錄問題的另一種方法是，如果你之前使用類似C++的程式語言，則是“ Java是否有指標？” Java中的每個物件標識符（除原語外）都是這些指標之一，但它們的用法是不僅受編譯器的約束，而且受執行時系統的約束。 換一種說法，Java有指標，但沒有指標演算法。 這些就是我一直所說的“引用”，您可以將它們視為“安全指標”，與小學的安全剪刀不同-它們不敏銳，因此您不費吹灰之力就無法傷害自己，但是它們有時可能很乏味。

<!-- Passing References -->

## 傳遞引用

<!-- Making Local Copies -->

當你將引用傳遞給方法時，它仍指向同一物件。 一個簡單的實驗示範了這一點：

```java
// references/PassReferences.java
public class PassReferences {
public static void f(PassReferences h) {
    	System.out.println("h inside f(): " + h);
    }
    public static void main(String[] args) {
        PassReferences p = new PassReferences();
        System.out.println("p inside main(): " + p);
        f(p);
    }
}
/* Output:
p inside main(): PassReferences@15db9742
h inside f(): PassReferences@15db9742
*/
```

方法  `toString() ` 在列印語句中自動呼叫，並且 `PassReferences` 直接從 `Object` 繼承而無需重新定義 `toString（）` 。 因此，使用的是 `Object` 的 `toString（）` 版本，它列印出物件的類，然後列印出該物件所在的地址（不是引用，而是實際的物件儲存）。

## 本機複製


<!-- Controlling Cloneability -->
## 控制複製


<!-- Immutable Classes -->
## 不可變類


<!-- Summary -->
## 本章小結



<!-- 分頁 -->
<div style="page-break-after: always;"></div>
