[TOC]

<!-- Annotations -->

# 第二十三章 註解

註解（也被稱為中繼資料）為我們在程式碼中添加訊息提供了一種形式化的方式，使我們可以在稍後的某個時刻更容易的使用這些資料。

註解在一定程度上是把中繼資料和原始碼文件結合在一起的趨勢所激發的，而不是儲存在外部文件。這同樣是物件 C# 語言對於 Java 語言特性壓力的一種回應。

註解是 Java 5 所引入的眾多語言變化之一。它們提供了 Java 無法表達的但是你需要完整表述程式所需的訊息。因此，註解使得我們可以以編譯器驗證的格式儲存程式的額外訊息。註解可以生成描述符文件，甚至是新的類定義，並且有助於減輕編寫“樣板”程式碼的負擔。透過使用註解，你可以將中繼資料儲存在 Java 原始碼中。並擁有如下優勢：簡單易讀的程式碼，編譯器類型檢查，使用 annotation API 為自己的註解構造處理工具。即使 Java 定義了一些類型的中繼資料，但是一般來說註解類型的添加和如何使用完全取決於你。

註解的語法十分簡單，主要是在現有語法中添加 @ 符號。Java 5 引入了前三種定義在 **java.lang** 包中的註解：

- **@Override**：表示目前的方法定義將覆蓋基類的方法。如果你不小心拼寫錯誤，或者方法簽名被錯誤拼寫的時候，編譯器就會發出錯誤提示。
- **@Deprecated**：如果使用該註解的元素被呼叫，編譯器就會發出警告訊息。
- **@SuppressWarnings**：關閉不當的編譯器警告訊息。
- **@SafeVarargs**：在 Java 7 中加入用於禁止對具有泛型varargs參數的方法或建構子的呼叫方發出警告。
- **@FunctionalInterface**：Java 8 中加入用於表示類型聲明為函數式介面。

還有 5 種額外的註解類型用於創造新的註解。你將會在這一章學習它們。

每當建立涉及重複工作的類或介面時，你通常可以使用註解來自動化和簡化流程。例如在 Enterprise JavaBean（EJB）中的許多額外工作就是透過註解來消除的。

註解的出現可以替代一些現有的系統，例如 XDoclet，它是一種獨立的文件化工具，專門設計用來生成註解風格的文件。與之相比，註解是真正語言層級的概念，以前構造出來就享有編譯器的類型檢查保護。註解在原始碼級別儲存所有訊息而不是透過注釋文字，這使得程式碼更加整潔和便於維護。透過使用擴展的 annotation API 或稍後在本章節可以看到的外部的位元組碼工具類庫，你會擁有對原始碼及位元組碼強大的檢查與操作能力。

<!-- Basic Syntax -->

## 基本語法

<!-- Writing Annotation Processors -->

在下面的例子中，使用 `@Test` 對 `testExecute()` 進行註解。該註解本身不做任何事情，但是編譯器要保證其類路徑上有 `@Test` 註解的定義。你將在本章看到，我們透過註解建立了一個工具用於執行這個方法：

```java
// annotations/Testable.java
package annotations;
import onjava.atunit.*;
public class Testable {
    public void execute() {
        System.out.println("Executing..");
    }
    @Test
    void testExecute() { execute(); }
}
```

被註解標註的方法和其他方法沒有任何區別。在這個例子中，註解 `@Test` 可以和任何修飾符共同用於方法，諸如 **public**、**static** 或 **void**。從語法的角度上看，註解和修飾符的使用方式是一致的。

### 定義註解

如下是一個註解的定義。註解的定義看起來很像介面的定義。事實上，它們和其他 Java 介面一樣，也會被編譯成 class 文件。

```java
// onjava/atunit/Test.java
// The @Test tag
package onjava.atunit;
import java.lang.annotation.*;
@Target(ElementType.METHOD)
@Retention(RetentionPolicy.RUNTIME)
public @interface Test {}
```

除了 @ 符號之外， `@Test` 的定義看起來更像一個空介面。註解的定義也需要一些元註解（meta-annotation），比如 `@Target` 和 `@Retention`。`@Target` 定義你的註解可以應用在哪裡（例如是方法還是欄位）。`@Retention` 定義了註解在哪裡可用，在原始碼中（SOURCE），class文件（CLASS）中或者是在執行時（RUNTIME）。

註解通常會包含一些表示特定值的元素。當分析處理註解的時候，程式或工具可以利用這些值。註解的元素看起來就像介面的方法，但是可以為其指定預設值。

不包含任何元素的註解稱為標記註解（marker annotation），例如上例中的 `@Test` 就是標記註解。

下面是一個簡單的註解，我們可以用它來追蹤項目中的用例。程式設計師可以使用該註解來標註滿足特定用例的一個方法或者一組方法。於是，項目經理可以透過統計已經實現的用例來掌控項目的進展，而開發者在維護項目時可以輕鬆的找到用例用於更新，或者他們可以除錯系統中業務邏輯。

```java
// annotations/UseCase.java
import java.lang.annotation.*;
@Target(ElementType.METHOD)
@Retention(RetentionPolicy.RUNTIME)
public @interface UseCase {
    int id();
    String description() default "no description";
}
```

注意 **id** 和 **description** 與方法定義類似。由於編譯器會對 **id** 進行類型檢查，因此將跟蹤資料庫與用例文件和原始碼相關聯是可靠的方式。**description** 元素擁有一個 **default** 值，如果在註解某個方法時沒有給出 **description** 的值。則該註解的處理器會使用此元素的預設值。

在下面的類中，有三個方法被註解為用例：

```java
// annotations/PasswordUtils.java
import java.util.*;
public class PasswordUtils {
    @UseCase(id = 47, description =
            "Passwords must contain at least one numeric")
    public boolean validatePassword(String passwd) {
        return (passwd.matches("\\w*\\d\\w*"));
    }
    @UseCase(id = 48)
    public String encryptPassword(String passwd) {
        return new StringBuilder(passwd)
                .reverse().toString();
    }
    @UseCase(id = 49, description =
            "New passwords can't equal previously used ones")
    public boolean checkForNewPassword(
            List<String> prevPasswords, String passwd) {
        return !prevPasswords.contains(passwd);
    }
}
```

註解的元素在使用時表現為 名-值 對的形式，並且需要放置在 `@UseCase` 聲明之後的括號內。在 `encryptPassword()` 方法的註解中，並沒有給出 **description** 的值，所以在 **@interface UseCase** 的註解處理器分析處理這個類的時候會使用該元素的預設值。

你應該能夠想像到如何使用這套工具來“勾勒”出將要建造的系統，然後在建造的過程中逐漸實現系統的各項功能。

### 元註解

Java 語言中目前有 5 種標準註解（前面介紹過），以及 5 種元註解。元註解用於註解其他的註解

| 註解        | 解釋                                                         |
| ----------- | ------------------------------------------------------------ |
| @Target     | 表示註解可以用於哪些地方。可能的 **ElementType** 參數包括：<br/>**CONSTRUCTOR**：構造器的聲明；<br/>**FIELD**：欄位聲明（包括 enum 實例）；<br/>**LOCAL_VARIABLE**：局部變數聲明；<br/>**METHOD**：方法聲明；<br/>**PACKAGE**：包聲明；<br/>**PARAMETER**：參數聲明；<br/>**TYPE**：類、介面（包括註解類型）或者 enum 聲明。 |
| @Retention  | 表示註解訊息儲存的時長。可選的 **RetentionPolicy** 參數包括：<br/>**SOURCE**：註解將被編譯器丟棄；<br/>**CLASS**：註解在 class 文件中可用，但是會被 VM 丟棄；<br/>**RUNTIME**：VM 將在執行期也保留註解，因此可以透過反射機制讀取註解的訊息。 |
| @Documented | 將此註解儲存在 Javadoc 中                                    |
| @Inherited  | 允許子類繼承父類的註解                                       |
| @Repeatable | 允許一個註解可以被使用一次或者多次（Java 8）。               |

大多數時候，程式設計師定義自己的註解，並編寫自己的處理器來處理他們。

## 編寫註解處理器

如果沒有用於讀取註解的工具，那麼註解不會比注釋更有用。使用註解中一個很重要的部分就是，建立與使用註解處理器。Java 擴展了反射機制的 API 用於幫助你創造這類工具。同時他還提供了 javac 編譯器鉤子在編譯時使用註解。

下面是一個非常簡單的註解處理器，我們用它來讀取被註解的 **PasswordUtils** 類，並且使用反射機制來尋找 **@UseCase** 標記。給定一組 **id** 值，然後列出在 **PasswordUtils** 中找到的用例，以及缺失的用例。

```java
// annotations/UseCaseTracker.java
import java.util.*;
import java.util.stream.*;
import java.lang.reflect.*;
public class UseCaseTracker {
    public static void
    trackUseCases(List<Integer> useCases, Class<?> cl) {
        for(Method m : cl.getDeclaredMethods()) {
            UseCase uc = m.getAnnotation(UseCase.class);
            if(uc != null) {
                System.out.println("Found Use Case " +
                        uc.id() + "\n " + uc.description());
                useCases.remove(Integer.valueOf(uc.id()));
            }
        }
        useCases.forEach(i ->
                System.out.println("Missing use case " + i));
    }
    public static void main(String[] args) {
        List<Integer> useCases = IntStream.range(47, 51)
                .boxed().collect(Collectors.toList());
        trackUseCases(useCases, PasswordUtils.class);
    }
}
```

輸出為：

```java
Found Use Case 48
no description
Found Use Case 47
Passwords must contain at least one numeric
Found Use Case 49
New passwords can't equal previously used ones
Missing use case 50
```

這個程式用了兩個反射的方法：`getDeclaredMethods()`  和 `getAnnotation()`，它們都屬於 **AnnotatedElement** 介面（**Class**，**Method** 與 **Field** 類都實現了該介面）。`getAnnotation()` 方法返回指定類型的註解物件，在本例中就是 “**UseCase**”。如果被註解的方法上沒有該類型的註解，返回值就為 **null**。我們透過呼叫 `id()` 和 `description()` 方法來提取元素值。注意 `encryptPassword()` 方法在註解的時候沒有指定 **description** 的值，因此處理器在處理它對應的註解時，通過 `description()` 取得的是預設值 “no description”。

### 註解元素

在 **UseCase.java** 中定義的 **@UseCase** 的標籤包含 int 元素 **id** 和 String 元素 **description**。註解元素可用的類型如下所示：

- 所有基本類型（int、float、boolean等）
- String
- Class
- enum
- Annotation
- 以上類型的陣列

如果你使用了其他類型，編譯器就會報錯。注意，也不允許使用任何包裝類型，但是由於自動裝箱的存在，這不算是什麼限制。註解也可以作為元素的類型。稍後你會看到，註解嵌套是一個非常有用的技巧。

### 預設值限制

編譯器對於元素的預設值有些過於挑剔。首先，元素不能有不確定的值。也就是說，元素要嘛有預設值，要嘛就在使用註解時提供元素的值。

這裡有另外一個限制：任何非基本類型的元素， 無論是在原始碼聲明時還是在註解介面中定義預設值時，都不能使用 null 作為其值。這個限制使得處理器很難表現一個元素的存在或者缺失的狀態，因為在每個註解的聲明中，所有的元素都存在，並且具有相應的值。為了繞開這個約束，可以自訂一些特殊的值，比如空字串或者負數用於表達某個元素不存在。

```java
// annotations/SimulatingNull.java
import java.lang.annotation.*;
@Target(ElementType.METHOD)
@Retention(RetentionPolicy.RUNTIME)
public @interface SimulatingNull {
    int id() default -1;
    String description() default "";
}
```

這是一個在定義註解的習慣用法。

### 生成外部文件

當有些框架需要一些額外的訊息才能與你的原始碼協同工作，這種情況下注解就會變得十分有用。像 Enterprise JavaBeans （EJB3 之前）這樣的技術，每一個 Bean 都需要大量的介面和部署描述文件，而這些就是“樣板”文件。Web Service，自訂標籤庫以及物件/關係映射工具（例如 Toplink 和 Hibernate）通常都需要 XML 描述文件，而這些文件脫離於程式碼之外。除了定義 Java 類，程式設計師還必須忍受沉悶，重複的提供某些訊息，例如類名和包名等已經在原始類中提供過的訊息。每當你使用外部描述文件時，他就擁有了一個類的兩個獨立訊息源，這經常導致程式碼的同步問題。同時這也要求了為項目工作的程式設計師在知道如何編寫 Java 程式的同時，也必須知道如何編輯描述文件。

假設你想提供一些基本的物件/關係映射功能，能夠自動生成資料庫表。你可以使用 XML 描述文件來指明類的名字、每個成員以及資料庫映射的相關訊息。但是，透過使用註解，你可以把所有訊息都儲存在 **JavaBean** 來源文件中。為此你需要一些用於定義資料庫表名稱、資料庫列以及將 SQL 類型映射到屬性的註解。

以下是一個註解的定義，它告訴註解處理器應該建立一個資料庫表：

```java
// annotations/database/DBTable.java
package annotations.database;
import java.lang.annotation.*;
@Target(ElementType.TYPE) // Applies to classes only
@Retention(RetentionPolicy.RUNTIME)
public @interface DBTable {
    String name() default "";
}
```

在 `@Target` 註解中指定的每一個 **ElementType** 就是一個約束，它告訴編譯器，這個自訂的註解只能用於指定的類型。你可以指定 **enum ElementType** 中的一個值，或者以逗號分割的形式指定多個值。如果想要將註解應用於所有的 **ElementType**，那麼可以省去 `@Target` 註解，但是這並不常見。

注意 **@DBTable** 中有一個 `name()` 元素，該註解透過這個元素為處理器建立資料庫時提供表的名字。

如下是修飾欄位的註解：

```java
// annotations/database/Constraints.java
package annotations.database;
import java.lang.annotation.*;
@Target(ElementType.FIELD)
@Retention(RetentionPolicy.RUNTIME)
public @interface Constraints {
    boolean primaryKey() default false;
    boolean allowNull() default true;
    boolean unique() default false;
}
```

```java
// annotations/database/SQLString.java
package annotations.database;
import java.lang.annotation.*;
@Target(ElementType.FIELD)
@Retention(RetentionPolicy.RUNTIME)
public @interface SQLString {
    int value() default 0;
    String name() default "";
    Constraints constraints() default @Constraints;
}
```

```java
// annotations/database/SQLInteger.java
package annotations.database;
import java.lang.annotation.*;
@Target(ElementType.FIELD)
@Retention(RetentionPolicy.RUNTIME)
public @interface SQLInteger {
    String name() default "";
    Constraints constraints() default @Constraints;
}
```

**@Constraints** 註解允許處理器提供資料庫表的中繼資料。**@Constraints** 代表了資料庫通常提供的約束的一小部分，但是它所要表達的思想已經很清楚了。`primaryKey()`，`allowNull()` 和 `unique()` 元素明顯的提供了預設值，從而使得在大多數情況下，該註解的使用者不需要輸入太多東西。

另外兩個 **@interface** 定義的是 SQL 類型。如果希望這個框架更有價值的話，我們應該為每個 SQL 類型都定義相應的註解。不過作為範例，兩個元素足夠了。

這些 SQL 類型具有 `name()` 元素和 `constraints()` 元素。後者利用了嵌套註解的功能，將資料庫列的類型約束訊息嵌入其中。注意 `constraints()` 元素的預設值是 **@Constraints**。由於在 **@Constraints** 註解類型之後，沒有在括號中指明 **@Constraints** 元素的值，因此，**constraints()** 的預設值為所有元素都為預設值的 **@Constraints** 註解。如果要使得嵌入的  **@Constraints**  註解中的 `unique()` 元素為 true，並作為 `constraints()` 元素的預設值，你可以像如下定義：

```java
// annotations/database/Uniqueness.java
// Sample of nested annotations
package annotations.database;
public @interface Uniqueness {
    Constraints constraints()
            default @Constraints(unique = true);
}
```

下面是一個簡單的，使用了如上註解的類：

```java
// annotations/database/Member.java
package annotations.database;
@DBTable(name = "MEMBER")
public class Member {
    @SQLString(30) String firstName;
    @SQLString(50) String lastName;
    @SQLInteger Integer age;
    @SQLString(value = 30,
            constraints = @Constraints(primaryKey = true))
    String reference;
    static int memberCount;
    public String getReference() { return reference; }
    public String getFirstName() { return firstName; }
    public String getLastName() { return lastName; }
    @Override
    public String toString() { return reference; }
    public Integer getAge() { return age; }
}
```

類註解 **@DBTable** 註解給定了元素值 MEMBER，它將會作為表的名字。類的屬性 **firstName** 和 **lastName** 都被註解為 **@SQLString** 類型並且給了預設元素值分別為 30 和 50。這些註解都有兩個有趣的地方：首先，他們都使用了嵌入的 **@Constraints** 註解的預設值；其次，它們都是用了捷徑特性。如果你在註解中定義了名為 **value** 的元素，並且在使用該註解時，**value** 為唯一一個需要賦值的元素，你就不需要使用名—值對的語法，你只需要在括號中給出 **value** 元素的值即可。這可以應用於任何合法類型的元素。這也限制了你必須將元素命名為 **value**，不過在上面的例子中，這樣的註解語句也更易於理解：

```java
@SQLString(30)
```

處理器將在建立表的時候使用該值設定 SQL 列的大小。

預設值的語法雖然很靈巧，但是它很快就變的複雜起來。以 **reference** 欄位的註解為例，上面擁有 **@SQLString** 註解，但是這個欄位也將成為表的主鍵，因此在嵌入的 **@Constraint** 註解中設定 **primaryKey** 元素的值。這時事情就變的複雜了。你不得不為這個嵌入的註解使用很長的鍵—值對的形式，來指定元素名稱和 **@interface** 的名稱。同時，由於有特殊命名的 **value** 也不是唯一需要賦值的元素，因此不能再使用捷徑特性。如你所見，最終結果不算清晰易懂。

### 替代方案

可以使用多種不同的方式來定義自己的註解用於上述任務。例如，你可以使用一個單一的註解類 **@TableColumn**，它擁有一個 **enum** 元素，元素值定義了 **STRING**，**INTEGER**，**FLOAT** 等類型。這消除了每個 SQL 類型都需要定義一個 **@interface** 的負擔，不過也使得用額外訊息修飾 SQL 類型變的不可能，這些額外的訊息例如長度或精度等，都可能是非常有用的。

你也可以使用一個 **String** 類型的元素來描述實際的 SQL 類型，比如 “VARCHAR(30)” 或者 “INTEGER”。這使得你可以修飾 SQL 類型，但是這也將 Java 類型到 SQL 類型的映射綁在了一起，這不是一個好的設計。你並不想在資料庫更改之後重新編譯你的程式碼；如果我們只需要告訴註解處理器，我們正在使用的是什麼“口味（favor）”的 SQL，然後註解處理器來為我們處理 SQL 類型的細節，那將是一個優雅的設計。

第三種可行的方案是一起使用兩個註解，**@Constraints** 和相應的 SQL 類型（例如，**@SQLInteger**）去註解同一個欄位。這可能會讓程式碼有些混亂，但是編譯器允許你對同一個目標使用多個註解。在 Java 8，在使用多個註解的時候，你可以重複使用同一個註解。

### 註解不支援繼承

你不能使用 **extends** 關鍵字來繼承 **@interfaces**。這真是一個遺憾，如果可以定義 **@TableColumn** 註解（參考前面的建議），同時嵌套一個 **@SQLType** 類型的註解，將成為一個優雅的設計。按照這種方式，你可以透過繼承 **@SQLType** 來創造各種 SQL 類型。例如 **@SQLInteger** 和 **@SQLString**。如果支援繼承，就會大大減少打字的工作量並且使得語法更整潔。在 Java 的未來版本中，似乎沒有任何關於讓註解支援繼承的提案，所以在目前情況下，上例中的解決方案可能已經是最佳方案了。

### 實現處理器

下面是一個註解處理器的例子，他將讀取一個類文件，檢查上面的資料庫註解，並生成用於建立資料庫的 SQL 指令：

```java
// annotations/database/TableCreator.java
// Reflection-based annotation processor
// {java annotations.database.TableCreator
// annotations.database.Member}
package annotations.database;

import java.lang.annotation.Annotation;
import java.lang.reflect.Field;
import java.util.ArrayList;
import java.util.List;

public class TableCreator {
    public static void
    main(String[] args) throws Exception {
        if (args.length < 1) {
            System.out.println(
                    "arguments: annotated classes");
            System.exit(0);
        }
        for (String className : args) {
            Class<?> cl = Class.forName(className);
            DBTable dbTable = cl.getAnnotation(DBTable.class);
            if (dbTable == null) {
                System.out.println(
                        "No DBTable annotations in class " +
                                className);
                continue;
            }
            String tableName = dbTable.name();
            // If the name is empty, use the Class name:
            if (tableName.length() < 1)
                tableName = cl.getName().toUpperCase();
            List<String> columnDefs = new ArrayList<>();
            for (Field field : cl.getDeclaredFields()) {
                String columnName = null;
                Annotation[] anns =
                        field.getDeclaredAnnotations();
                if (anns.length < 1)
                    continue; // Not a db table column
                if (anns[0] instanceof SQLInteger) {
                    SQLInteger sInt = (SQLInteger) anns[0];
                    // Use field name if name not specified
                    if (sInt.name().length() < 1)
                        columnName = field.getName().toUpperCase();
                    else
                        columnName = sInt.name();
                    columnDefs.add(columnName + " INT" +
                            getConstraints(sInt.constraints()));
                }
                if (anns[0] instanceof SQLString) {
                    SQLString sString = (SQLString) anns[0];
                    // Use field name if name not specified.
                    if (sString.name().length() < 1)
                        columnName = field.getName().toUpperCase();
                    else
                        columnName = sString.name();
                    columnDefs.add(columnName + " VARCHAR(" +
                            sString.value() + ")" +
                            getConstraints(sString.constraints()));
                }
                StringBuilder createCommand = new StringBuilder(
                        "CREATE TABLE " + tableName + "(");
                for (String columnDef : columnDefs)
                    createCommand.append(
                            "\n " + columnDef + ",");
                // Remove trailing comma
                String tableCreate = createCommand.substring(
                        0, createCommand.length() - 1) + ");";
                System.out.println("Table Creation SQL for " +
                        className + " is:\n" + tableCreate);
            }
        }
    }

    private static String getConstraints(Constraints con) {
        String constraints = "";
        if (!con.allowNull())
            constraints += " NOT NULL";
        if (con.primaryKey())
            constraints += " PRIMARY KEY";
        if (con.unique())
            constraints += " UNIQUE";
        return constraints;
    }
}
```

輸出為：

```sql
Table Creation SQL for annotations.database.Member is:
CREATE TABLE MEMBER(
    FIRSTNAME VARCHAR(30));
Table Creation SQL for annotations.database.Member is:
CREATE TABLE MEMBER(
    FIRSTNAME VARCHAR(30),
    LASTNAME VARCHAR(50));
Table Creation SQL for annotations.database.Member is:
CREATE TABLE MEMBER(
    FIRSTNAME VARCHAR(30),
    LASTNAME VARCHAR(50),
    AGE INT);
Table Creation SQL for annotations.database.Member is:
CREATE TABLE MEMBER(
    FIRSTNAME VARCHAR(30),
    LASTNAME VARCHAR(50),
    AGE INT,
    REFERENCE VARCHAR(30) PRIMARY KEY);
```

主方法會循環處理命令列傳入的每一個類名。每一個類都是用 ` forName()` 方法進行載入，並使用 `getAnnotation(DBTable.class)` 來檢查該類是否帶有 **@DBTable** 註解。如果存在，將表名儲存起來。然後讀取這個類的所有欄位，並使用 `getDeclaredAnnotations()` 進行檢查。這個方法返回一個包含特定欄位上所有註解的陣列。然後使用 **instanceof** 操作符判斷這些註解是否是 **@SQLInteger** 或者 **@SQLString** 類型。如果是的話，在對應的處理塊中將構造出相應的資料庫列的字串片段。注意，由於註解沒有繼承機制，如果要獲取近似多態的行為，使用 `getDeclaredAnnotations()` 似乎是唯一的方式。

嵌套的 **@Constraint** 註解被傳遞給 `getConstraints()`方法，並用它來構造一個包含 SQL 約束的 String 物件。

需要提醒的是，上面示範的技巧對於真實的物件/映射關係而言，是十分幼稚的。使用 **@DBTable** 的註解來獲取表的名稱，這使得如果要修改表的名字，則迫使你重新編譯 Java 程式碼。這種效果並不理想。現在已經有了很多可用的框架，用於將物件映射到資料庫中，並且越來越多的框架開始使用註解了。

<!-- Using javac to Process Annotations -->

## 使用javac處理註解

通過 **javac**，你可以透過建立編譯時（compile-time）註解處理器在 Java 來源文件上使用註解，而不是編譯之後的 class 文件。但是這裡有一個重大限制：你不能透過處理器來改變原始碼。唯一影響輸出的方式就是建立新的文件。

如果你的註解處理器建立了新的來源文件，在新一輪處理中註解會檢查來源文件本身。工具在檢測一輪之後持續循環，直到不再有新的來源文件產生。然後它編譯所有的來源文件。

每一個你編寫的註解都需要處理器，但是 **javac** 可以非常容易的將多個註解處理器合併在一起。你可以指定多個需要處理的類，並且你可以添加監聽器用於監聽註解處理完成後接到通知。

本節中的範例將幫助你開始學習，但如果你必須深入學習，請做好反覆學習，大量訪問 Google 和 StackOverflow 的準備。

### 最簡單的處理器

讓我們開始定義我們能想到的最簡單的處理器，只是為了編譯和測試。如下是註解的定義：

```java
// annotations/simplest/Simple.java
// A bare-bones annotation
package annotations.simplest;
import java.lang.annotation.Retention;
import java.lang.annotation.RetentionPolicy;
import java.lang.annotation.Target;
import java.lang.annotation.ElementType;
@Retention(RetentionPolicy.SOURCE)
@Target({ElementType.TYPE, ElementType.METHOD,
        ElementType.CONSTRUCTOR,
        ElementType.ANNOTATION_TYPE,
        ElementType.PACKAGE, ElementType.FIELD,
        ElementType.LOCAL_VARIABLE})
public @interface Simple {
    String value() default "-default-";
}
```

**@Retention** 的參數現在為 **SOURCE**，這意味著註解不會在存留在編譯後的程式碼。這在編譯時處理註解是沒有必要的，它只是指出，在這裡，**javac** 是唯一有機會處理註解的代理。

**@Target** 聲明了幾乎所有的目標類型（除了 **PACKAGE**） ，同樣是為了示範。下面是一個測試範例。

```java
// annotations/simplest/SimpleTest.java
// Test the "Simple" annotation
// {java annotations.simplest.SimpleTest}
package annotations.simplest;
@Simple
public class SimpleTest {
    @Simple
    int i;
    @Simple
    public SimpleTest() {}
    @Simple
    public void foo() {
        System.out.println("SimpleTest.foo()");
    }
    @Simple
    public void bar(String s, int i, float f) {
        System.out.println("SimpleTest.bar()");
    }
    @Simple
    public static void main(String[] args) {
        @Simple
        SimpleTest st = new SimpleTest();
        st.foo();
    }
}
```

輸出為：

```java
SimpleTest.foo()
```

在這裡我們使用 **@Simple** 註解了所有 **@Target** 聲明允許的地方。

**SimpleTest.java** 只需要 **Simple.java** 就可以編譯成功。當我們編譯的時候什麼都沒有發生。

**javac** 允許 **@Simple** 註解（只要它存在）在我們建立處理器並將其 hook 到編譯器之前，不做任何事情。

如下是一個十分簡單的處理器，其所作的事情就是把註解相關的訊息列印出來：

```java
// annotations/simplest/SimpleProcessor.java
// A bare-bones annotation processor
package annotations.simplest;
import javax.annotation.processing.*;
import javax.lang.model.SourceVersion;
import javax.lang.model.element.*;
import java.util.*;
@SupportedAnnotationTypes(
        "annotations.simplest.Simple")
@SupportedSourceVersion(SourceVersion.RELEASE_8)
public class SimpleProcessor
        extends AbstractProcessor {
    @Override
    public boolean process(
            Set<? extends TypeElement> annotations,
            RoundEnvironment env) {
        for(TypeElement t : annotations)
            System.out.println(t);
        for(Element el :
                env.getElementsAnnotatedWith(Simple.class))
            display(el);
        return false;
    }
    private void display(Element el) {
        System.out.println("==== " + el + " ====");
        System.out.println(el.getKind() +
                " : " + el.getModifiers() +
                " : " + el.getSimpleName() +
                " : " + el.asType());
        if(el.getKind().equals(ElementKind.CLASS)) {
            TypeElement te = (TypeElement)el;
            System.out.println(te.getQualifiedName());
            System.out.println(te.getSuperclass());
            System.out.println(te.getEnclosedElements());
        }
        if(el.getKind().equals(ElementKind.METHOD)) {
            ExecutableElement ex = (ExecutableElement)el;
            System.out.print(ex.getReturnType() + " ");
            System.out.print(ex.getSimpleName() + "(");
            System.out.println(ex.getParameters() + ")");
        }
    }
}
```

（舊的，失效的）**apt** 版本的處理器需要額外的方法來確定支援哪些註解以及支援的 Java 版本。不過，你現在可以簡單的使用 **@SupportedAnnotationTypes** 和 **@SupportedSourceVersion** 註解（這是一個很好的用註解簡化程式碼的範例）。

你唯一需要實現的方法就是 `process()`，這裡是所有行為發生的地方。第一個參數告訴你哪個註解是存在的，第二個參數保留了剩餘訊息。我們所做的事情只是列印了註解（這裡只存在一個），可以看 **TypeElement** 文件中的其他行為。透過使用 `process()` 的第二個操作，我們循環所有被 **@Simple** 註解的元素，並且針對每一個元素呼叫我們的 `display()` 方法。所有 **Element** 展示了自身的基本訊息；例如，`getModifiers()` 告訴你它是否為 **public** 和 **static** 。

**Element** 只能執行那些編譯器解析的所有基本物件共有的操作，而類和方法之類的東西有額外的訊息需要提取。所以（如果你閱讀了正確的文件，但是我沒有在任何文件中找到——我不得不透過 StackOverflow 尋找線索）你檢查它是哪種 **ElementKind**，然後將其向下轉換為更具體的元素類型，注入針對 CLASS 的 TypeElement 和針對 METHOD 的ExecutableElement。此時，可以為這些元素呼叫其他方法。

動態向下轉型（在編譯期不進行檢查）並不像是 Java 的做事方式，這非常不直觀這也是為什麼我從未想過要這樣做事。相反，我花了好幾天的時間，試圖發現你應該如何訪問這些訊息，而這些訊息至少在某種程度上是用不起作用的恰當方法簡單明瞭的。我還沒有遇到任何東西說上面是規範的形式，但在我看來是。

如果只是透過平常的方式來編譯 **SimpleTest.java**，你不會得到任何結果。為了得到註解輸出，你必須增加一個 **processor** 標誌並且連接註解處理器類

```shell
javac -processor annotations.simplest.SimpleProcessor SimpleTest.java
```

現在編譯器有了輸出

```shell
annotations.simplest.Simple
==== annotations.simplest.SimpleTest ====
CLASS : [public] : SimpleTest : annotations.simplest.SimpleTest
annotations.simplest.SimpleTest
java.lang.Object
i,SimpleTest(),foo(),bar(java.lang.String,int,float),main(java.lang.String[])
==== i ====
FIELD : [] : i : int
==== SimpleTest() ====
CONSTRUCTOR : [public] : <init> : ()void
==== foo() ====
METHOD : [public] : foo : ()void
void foo()
==== bar(java.lang.String,int,float) ====
METHOD : [public] : bar : (java.lang.String,int,float)void
void bar(s,i,f)
==== main(java.lang.String[]) ====
METHOD : [public, static] : main : (java.lang.String[])void
void main(args)
```

這給了你一些可以發現的東西，包括參數名和類型、返回值等。

### 更複雜的處理器

當你建立用於 javac 註解處理器時，你不能使用 Java 的反射特性，因為你處理的是原始碼，而並非是編譯後的 class 文件。各種 mirror[^3 ] 解決這個問題的方法是，透過允許你在未編譯的原始碼中查看方法、欄位和類型。

如下是一個用於提取類中方法的註解，所以它可以被抽取成為一個介面：

```java
// annotations/ifx/ExtractInterface.java
// javac-based annotation processing
package annotations.ifx;
import java.lang.annotation.*;
@Target(ElementType.TYPE)
@Retention(RetentionPolicy.SOURCE)
public @interface ExtractInterface {
    String interfaceName() default "-!!-";
}
```

**RetentionPolicy** 的值為 **SOURCE**，這是為了在提取類中的介面之後不再將註解訊息保留在 class 文件中。接下來的測試類提供了一些公用方法，這些方法可以成為介面的一部分：

```java
// annotations/ifx/Multiplier.java
// javac-based annotation processing
// {java annotations.ifx.Multiplier}
package annotations.ifx;
@ExtractInterface(interfaceName="IMultiplier")
public class Multiplier {
    public boolean flag = false;
    private int n = 0;
    public int multiply(int x, int y) {
        int total = 0;
        for(int i = 0; i < x; i++)
            total = add(total, y);
        return total;
    }
    public int fortySeven() { return 47; }
    private int add(int x, int y) {
        return x + y;
    }
    public double timesTen(double arg) {
        return arg * 10;
    }
    public static void main(String[] args) {
        Multiplier m = new Multiplier();
        System.out.println(
                "11 * 16 = " + m.multiply(11, 16));
    }
}
```

輸出為：

```java
11 * 16 = 176
```

**Multiplier** 類（只能處理正整數）擁有一個 `multiply()` 方法，這個方法會多次呼叫私有方法 `add()` 來模擬乘法操作。` add()` 是私有方法，因此不能成為介面的一部分。其他的方法提供了語法多樣性。註解被賦予 **IMultiplier** 的 **InterfaceName** 作為要建立的介面的名稱。

這裡有一個編譯時處理器用於提取有趣的方法，並建立一個新的 interface 原始碼文件（這個來源文件將會在下一輪中被自動編譯）：

```java
// annotations/ifx/IfaceExtractorProcessor.java
// javac-based annotation processing
package annotations.ifx;
import javax.annotation.processing.*;
import javax.lang.model.SourceVersion;
import javax.lang.model.element.*;
import javax.lang.model.util.*;
import java.util.*;
import java.util.stream.*;
import java.io.*;
@SupportedAnnotationTypes(
        "annotations.ifx.ExtractInterface")
@SupportedSourceVersion(SourceVersion.RELEASE_8)
public class IfaceExtractorProcessor
        extends AbstractProcessor {
    private ArrayList<Element>
            interfaceMethods = new ArrayList<>();
    Elements elementUtils;
    private ProcessingEnvironment processingEnv;
    @Override
    public void init(
            ProcessingEnvironment processingEnv) {
        this.processingEnv = processingEnv;
        elementUtils = processingEnv.getElementUtils();
    }
    @Override
    public boolean process(
            Set<? extends TypeElement> annotations,
            RoundEnvironment env) {
        for(Element elem:env.getElementsAnnotatedWith(
                ExtractInterface.class)) {
            String interfaceName = elem.getAnnotation(
                    ExtractInterface.class).interfaceName();
            for(Element enclosed :
                    elem.getEnclosedElements()) {
                if(enclosed.getKind()
                        .equals(ElementKind.METHOD) &&
                        enclosed.getModifiers()
                                .contains(Modifier.PUBLIC) &&
                        !enclosed.getModifiers()
                                .contains(Modifier.STATIC)) {
                    interfaceMethods.add(enclosed);
                }
            }
            if(interfaceMethods.size() > 0)
                writeInterfaceFile(interfaceName);
        }
        return false;
    }
    private void
    writeInterfaceFile(String interfaceName) {
        try(
                Writer writer = processingEnv.getFiler()
                        .createSourceFile(interfaceName)
                        .openWriter()
        ) {
            String packageName = elementUtils
                    .getPackageOf(interfaceMethods
                            .get(0)).toString();
            writer.write(
                    "package " + packageName + ";\n");
            writer.write("public interface " +
                    interfaceName + " {\n");
            for(Element elem : interfaceMethods) {
                ExecutableElement method =
                        (ExecutableElement)elem;
                String signature = " public ";
                signature += method.getReturnType() + " ";
                signature += method.getSimpleName();
                signature += createArgList(
                        method.getParameters());
                System.out.println(signature);
                writer.write(signature + ";\n");
            }
            writer.write("}");
        } catch(Exception e) {
            throw new RuntimeException(e);
        }
    }
    private String createArgList(
            List<? extends VariableElement> parameters) {
        String args = parameters.stream()
                .map(p -> p.asType() + " " + p.getSimpleName())
                .collect(Collectors.joining(", "));
        return "(" + args + ")";
    }
}
```

**Elements** 物件實例 **elementUtils** 是一組靜態方法的工具；我們用它來尋找 **writeInterfaceFile()** 中含有的包名。

`getEnclosedElements()`方法會透過指定的元素生成所有的“閉包”元素。在這裡，這個類閉包了它的所有元素。透過使用 `getKind()` 我們會找到所有的 **public** 和 **static** 方法，並將其添加到 **interfaceMethods** 列表中。接下來 `writeInterfaceFile()` 使用 **interfaceMethods** 列表裡面的值生成新的介面定義。注意，在 `writeInterfaceFile()` 使用了向下轉型到 **ExecutableElement**，這使得我們可以獲取所有的方法訊息。**createArgList()** 是一個幫助方法，用於生成參數列表。

**Filer**是 `getFiler()` 生成的，並且是 **PrintWriter** 的一種實例，可以用於建立新文件。我們使用 **Filer** 物件，而不是原生的 **PrintWriter** 原因是，這個物件可以執行 **javac** 追蹤你建立的新文件，這使得它可以在新一輪中檢查新文件中的註解並編譯文件。

如下是一個命令列，可以在編譯的時候使用處理器：

```shell
javac -processor annotations.ifx.IfaceExtractorProcessor Multiplier.java
```

新生成的 **IMultiplier.java** 的文件，正如你透過查看上面處理器的 `println()` 語句所猜測的那樣，如下所示：

```java
package annotations.ifx;
public interface IMultiplier {
    public int multiply(int x, int y);
    public int fortySeven();
    public double timesTen(double arg);
}
```

這個類同樣會被 **javac** 編譯（在某一輪中），所以你會在同一個目錄中看到 **IMultiplier.class** 文件。

<!-- Annotation-Based Unit Testing -->

## 基於註解的單元測試

單元測試是對類中每個方法提供一個或者多個測試的一種事件，其目的是為了有規律的測試一個類中每個部分是否具備正確的行為。在 Java 中，最著名的單元測試工具就是 **JUnit**。**JUnit** 4 版本已經包含了註解。在註解版本之前的 JUnit 一個最主要的問題是，為了啟動和執行 **JUnit** 測試，有大量的“儀式”需要標註。這種負擔已經減輕了一些，**但是**註解使得測試更接近“可以工作的最簡單的測試系統”。

在註解版本之前的 JUnit，你必須建立一個單獨的文件來儲存單元測試。透過註解，我們可以將單元測試整合在需要被測試的類中，從而將單元測試的時間和麻煩降到了最低。這種方式有額外的好處，就是使得測試私有方法和公有方法變的一樣容易。

這個基於註解的測試框架叫做 **@Unit**。其最基本的測試形式，可能也是你使用的最多的一個註解是 **@Test**，我們使用 **@Test** 來標記測試方法。測試方法不帶參數，並返回 **boolean** 結果來說明測試方法成功或者失敗。你可以任意命名它的測試方法。同時 **@Unit** 測試方法可以是任意你喜歡的訪問修飾方法，包括 **private**。

要使用 **@Unit**，你必須匯入 **onjava.atunit** 包，並且使用 **@Unit** 的測試標記為合適的方法和欄位打上標籤（在接下來的例子中你會學到），然後讓你的構建系統對編譯後的類執行 **@Unit**，下面是一個簡單的例子：

```java
// annotations/AtUnitExample1.java
// {java onjava.atunit.AtUnit
// build/classes/main/annotations/AtUnitExample1.class}
package annotations;
import onjava.atunit.*;
import onjava.*;
public class AtUnitExample1 {
    public String methodOne() {
        return "This is methodOne";
    }
    public int methodTwo() {
        System.out.println("This is methodTwo");
        return 2;
    }
    @Test
    boolean methodOneTest() {
        return methodOne().equals("This is methodOne");
    }
    @Test
    boolean m2() { return methodTwo() == 2; }
    @Test
    private boolean m3() { return true; }
    // Shows output for failure:
    @Test
    boolean failureTest() { return false; }
    @Test
    boolean anotherDisappointment() {
        return false;
    }
}
```

輸出為：

```java
annotations.AtUnitExample1
. m3
. methodOneTest
. m2 This is methodTwo
. failureTest (failed)
. anotherDisappointment (failed)
(5 tests)
>>> 2 FAILURES <<<
annotations.AtUnitExample1: failureTest
annotations.AtUnitExample1: anotherDisappointment
```

使用 **@Unit** 進行測試的類必須定義在某個包中（即必須包括 **package** 聲明）。

**@Test** 註解被置於 `methodOneTest()`、 `m2()`、`m3()`、`failureTest()` 以及 `anotherDisappointment()` 方法之前，它們告訴 **@Unit** 方法作為單元測試來執行。同時 **@Test** 確保這些方法沒有任何參數並且返回值為 **boolean** 或者 **void**。當你填寫單元測試時，唯一需要做的就是決定測試是成功還是失敗，（對於返回值為 **boolean** 的方法）應該返回 **ture** 還是 **false**。

如果你熟悉 **JUnit**，你還將注意到 **@Unit** 輸出的訊息更多。你會看到現在正在執行的測試的輸出更有用，最後它會告訴你導致失敗的類和測試。

你並非必須將測試方法嵌入到原來的類中，有時候這種事情根本做不到。要生產一個非嵌入式的測試，最簡單的方式就是繼承：

```java
// annotations/AUExternalTest.java
// Creating non-embedded tests
// {java onjava.atunit.AtUnit
// build/classes/main/annotations/AUExternalTest.class}
package annotations;
import onjava.atunit.*;
import onjava.*;
public class AUExternalTest extends AtUnitExample1 {
    @Test
    boolean _MethodOne() {
        return methodOne().equals("This is methodOne");
    }
    @Test
    boolean _MethodTwo() {
        return methodTwo() == 2;
    }
}
```

輸出為：

```java
annotations.AUExternalTest
. tMethodOne
. tMethodTwo This is methodTwo
OK (2 tests)
```

這個範例還表現出靈活命名的價值。在這裡，**@Test** 方法被命名為下劃線前綴加上要測試的方法名稱（我並不認為這是一種理想的命名形式，這只是表現一種可能性罷了）。

你也可以使用組合來建立非嵌入式的測試：

```java
// annotations/AUComposition.java
// Creating non-embedded tests
// {java onjava.atunit.AtUnit
// build/classes/main/annotations/AUComposition.class}
package annotations;
import onjava.atunit.*;
import onjava.*;
public class AUComposition {
    AtUnitExample1 testObject = new AtUnitExample1();
    @Test
    boolean tMethodOne() {
        return testObject.methodOne()
                .equals("This is methodOne");
    }
    @Test
    boolean tMethodTwo() {
        return testObject.methodTwo() == 2;
    }
}
```

輸出為：

```java
annotations.AUComposition
. tMethodTwo This is methodTwo
. tMethodOne
OK (2 tests)
```

因為在每一個測試裡面都會建立 **AUComposition** 物件，所以建立新的成員變數 **testObject** 用於以後的每一個測試方法。

因為 **@Unit** 中沒有 **JUnit** 中特殊的 **assert** 方法，不過另一種形式的 **@Test** 方法仍然允許返回值為 **void**（如果你還想使用 **true** 或者 **false** 的話，也可以使用 **boolean** 作為方法返回值類型）。為了表示測試成功，可以使用 Java 的 **assert** 語句。Java 斷言機制需要你在 java 命令列行加上 **-ea** 標誌來開啟，但是 **@Unit** 已經自動開啟了該功能。要表示測試失敗的話，你甚至可以使用異常。**@Unit** 的設計目標之一就是儘可能減少添加額外的語法，而 Java 的 **assert** 和異常對於報告錯誤而言，即已經足夠了。一個失敗的 **assert** 或者從方法從拋出的異常都被視為測試失敗，但是 **@Unit** 不會在這個失敗的測試上卡住，它會繼續執行，直到所有測試完畢，下面是一個範例程式：

```java
// annotations/AtUnitExample2.java
// Assertions and exceptions can be used in @Tests
// {java onjava.atunit.AtUnit
// build/classes/main/annotations/AtUnitExample2.class}
package annotations;
import java.io.*;
import onjava.atunit.*;
import onjava.*;
public class AtUnitExample2 {
    public String methodOne() {
        return "This is methodOne";
    }
    public int methodTwo() {
        System.out.println("This is methodTwo");
        return 2;
    }
    @Test
    void assertExample() {
        assert methodOne().equals("This is methodOne");
    }
    @Test
    void assertFailureExample() {
        assert 1 == 2: "What a surprise!";
    }
    @Test
    void exceptionExample() throws IOException {
        try(FileInputStream fis =
                    new FileInputStream("nofile.txt")) {} // Throws
    }
    @Test
    boolean assertAndReturn() {
        // Assertion with message:
        assert methodTwo() == 2: "methodTwo must equal 2";
        return methodOne().equals("This is methodOne");
    }
}
```

輸出為：

```java
annotations.AtUnitExample2
. exceptionExample java.io.FileNotFoundException:
nofile.txt (The system cannot find the file specified)
(failed)
. assertExample
. assertAndReturn This is methodTwo
. assertFailureExample java.lang.AssertionError: What
a surprise!
(failed)
(4 tests)
>>> 2 FAILURES <<<
annotations.AtUnitExample2: exceptionExample
annotations.AtUnitExample2: assertFailureExample
```

如下是一個使用非嵌入式測試的例子，並且使用了斷言，它將會對 **java.util.HashSet** 進行一些簡單的測試：

```java
// annotations/HashSetTest.java
// {java onjava.atunit.AtUnit
// build/classes/main/annotations/HashSetTest.class}
package annotations;
import java.util.*;
import onjava.atunit.*;
import onjava.*;
public class HashSetTest {
    HashSet<String> testObject = new HashSet<>();
    @Test
    void initialization() {
        assert testObject.isEmpty();
    }
    @Test
    void _Contains() {
        testObject.add("one");
        assert testObject.contains("one");
    }
    @Test
    void _Remove() {
        testObject.add("one");
        testObject.remove("one");
        assert testObject.isEmpty();
    }
}
```

採用繼承的方式可能會更簡單，也沒有一些其他的約束。

對每一個單元測試而言，**@Unit** 都會使用預設的無參構造器，為該測試類所屬的類建立出一個新的實例。並在此新建立的物件上執行測試，然後丟棄該物件，以免對其他測試產生副作用。如此建立物件導致我們依賴於類的預設構造器。如果你的類沒有預設構造器，或者物件需要複雜的構造過程，那麼你可以建立一個 **static** 方法專門負責構造物件，然後使用 **@TestObjectCreate** 註解標記該方法，例子如下：

```java
// annotations/AtUnitExample3.java
// {java onjava.atunit.AtUnit
// build/classes/main/annotations/AtUnitExample3.class}
package annotations;
import onjava.atunit.*;
import onjava.*;
public class AtUnitExample3 {
    private int n;
    public AtUnitExample3(int n) { this.n = n; }
    public int getN() { return n; }
    public String methodOne() {
        return "This is methodOne";
    }
    public int methodTwo() {
        System.out.println("This is methodTwo");
        return 2;
    }
    @TestObjectCreate
    static AtUnitExample3 create() {
        return new AtUnitExample3(47);
    }
    @Test
    boolean initialization() { return n == 47; }
    @Test
    boolean methodOneTest() {
        return methodOne().equals("This is methodOne");
    }
    @Test
    boolean m2() { return methodTwo() == 2; }
}
```

輸出為：

```java
annotations.AtUnitExample3
. initialization
. m2 This is methodTwo
. methodOneTest
OK (3 tests)
```

**@TestObjectCreate** 修飾的方法必須聲明為 **static** ，且必須返回一個你正在測試的類型物件，這一切都由 **@Unit** 負責確保成立。

有的時候，你需要向單元測試中增加一些欄位。這時候可以使用 **@TestProperty** 註解，由它註解的欄位表示只在單元測試中使用（因此，在你將產品發布給客戶之前，他們應該被刪除）。在下面的例子中，一個 **String** 通過 `String.split()` 方法進行分割，從其中讀取一個值，這個值將會被生成測試物件：

```java
// annotations/AtUnitExample4.java
// {java onjava.atunit.AtUnit
// build/classes/main/annotations/AtUnitExample4.class}
// {VisuallyInspectOutput}
package annotations;
import java.util.*;
import onjava.atunit.*;
import onjava.*;
public class AtUnitExample4 {
    static String theory = "All brontosauruses " +
            "are thin at one end, much MUCH thicker in the " +
            "middle, and then thin again at the far end.";
    private String word;
    private Random rand = new Random(); // Time-based seed
    public AtUnitExample4(String word) {
        this.word = word;
    }
    public String getWord() { return word; }
    public String scrambleWord() {
        List<Character> chars = Arrays.asList(
                ConvertTo.boxed(word.toCharArray()));
        Collections.shuffle(chars, rand);
        StringBuilder result = new StringBuilder();
        for(char ch : chars)
            result.append(ch);
        return result.toString();
    }
    @TestProperty
    static List<String> input =
            Arrays.asList(theory.split(" "));
    @TestProperty
    static Iterator<String> words = input.iterator();
    @TestObjectCreate
    static AtUnitExample4 create() {
        if(words.hasNext())
            return new AtUnitExample4(words.next());
        else
            return null;
    }
    @Test
    boolean words() {
        System.out.println("'" + getWord() + "'");
        return getWord().equals("are");
    }
    @Test
    boolean scramble1() {
// Use specific seed to get verifiable results:
        rand = new Random(47);
        System.out.println("'" + getWord() + "'");
        String scrambled = scrambleWord();
        System.out.println(scrambled);
        return scrambled.equals("lAl");
    }
    @Test
    boolean scramble2() {
        rand = new Random(74);
        System.out.println("'" + getWord() + "'");
        String scrambled = scrambleWord();
        System.out.println(scrambled);
        return scrambled.equals("tsaeborornussu");
    }
}
```

輸出為：

```java
annotations.AtUnitExample4
. words 'All'
(failed)
. scramble1 'brontosauruses'
ntsaueorosurbs
(failed)
. scramble2 'are'
are
(failed)
(3 tests)
>>> 3 FAILURES <<<
annotations.AtUnitExample4: words
annotations.AtUnitExample4: scramble1
annotations.AtUnitExample4: scramble2
```

**@TestProperty** 也可以用來標記那些只在測試中使用的方法，但是它們本身不是測試方法。

如果你的測試物件需要執行某些初始化工作，並且使用完成之後還需要執行清理工作，那麼可以選擇使用 **static** 的  **@TestObjectCleanup** 方法，當測試物件使用結束之後，該方法會為你執行清理工作。在下面的範例中，**@TestObjectCleanup** 為每一個測試物件都打開了一個文件，因此必須在丟棄測試的時候關閉該文件：

```java
// annotations/AtUnitExample5.java
// {java onjava.atunit.AtUnit
// build/classes/main/annotations/AtUnitExample5.class}
package annotations;
import java.io.*;
import onjava.atunit.*;
import onjava.*;
public class AtUnitExample5 {
    private String text;
    public AtUnitExample5(String text) {
        this.text = text;
    }
    @Override
    public String toString() { return text; }
    @TestProperty
    static PrintWriter output;
    @TestProperty
    static int counter;
    @TestObjectCreate
    static AtUnitExample5 create() {
        String id = Integer.toString(counter++);
        try {
            output = new PrintWriter("Test" + id + ".txt");
        } catch(IOException e) {
            throw new RuntimeException(e);
        }
        return new AtUnitExample5(id);
    }
    @TestObjectCleanup
    static void cleanup(AtUnitExample5 tobj) {
        System.out.println("Running cleanup");
        output.close();
    }
    @Test
    boolean test1() {
        output.print("test1");
        return true;
    }
    @Test
    boolean test2() {
        output.print("test2");
        return true;
    }
    @Test
    boolean test3() {
        output.print("test3");
        return true;
    }
}
```

輸出為：

```java
annotations.AtUnitExample5
. test1
Running cleanup
. test3
Running cleanup
. test2
Running cleanup
OK (3 tests)
```

在輸出中我們可以看到，清理方法會在每個測試方法結束之後自動執行。

### 在 @Unit 中使用泛型

泛型為 **@Unit** 出了一個難題，因為我們不可能“通用測試”。我們必須針對某個特定類型的參數或者參數集才能進行測試。解決方法十分簡單，讓測試類繼承自泛型類的一個特定版本即可：

下面是一個 **stack** 的簡單實現：

```java
// annotations/StackL.java
// A stack built on a LinkedList
package annotations;
import java.util.*;
public class StackL<T> {
    private LinkedList<T> list = new LinkedList<>();
    public void push(T v) { list.addFirst(v); }
    public T top() { return list.getFirst(); }
    public T pop() { return list.removeFirst(); }
}
```

為了測試 String 版本，我們直接讓測試類繼承一個 Stack\<String\> ：

```java
// annotations/StackLStringTst.java
// Applying @Unit to generics
// {java onjava.atunit.AtUnit
// build/classes/main/annotations/StackLStringTst.class}
package annotations;
import onjava.atunit.*;
import onjava.*;
public class
StackLStringTst extends StackL<String> {
    @Test
    void tPush() {
        push("one");
        assert top().equals("one");
        push("two");
        assert top().equals("two");
    }
    @Test
    void tPop() {
        push("one");
        push("two");
        assert pop().equals("two");
        assert pop().equals("one");
    }
    @Test
    void tTop() {
        push("A");
        push("B");
        assert top().equals("B");
        assert top().equals("B");
    }
}
```

輸出為：

```java
annotations.StackLStringTst
. tTop
. tPush
. tPop
OK (3 tests)
```

這種方法存在的唯一缺點是，繼承使我們失去了訪問被測試的類中 **private** 方法的能力。這對你非常重要，那你要嘛把 private 方法變為 **protected**，要嘛添加一個非 **private** 的 **@TestProperty** 方法，由它來呼叫 **private** 方法（稍後我們會看到，**AtUnitRemover** 會刪除產品中的 **@TestProperty** 方法）。

**@Unit** 搜尋那些包含合適註解的類文件，然後執行 **@Test** 方法。我的主要目標就是讓 **@Unit** 測試系統儘可能的透明，使得人們使用它的時候只需要添加 **@Test** 註解，而不需要特殊的編碼和知識（現在版本的 **JUnit** 符合這個實踐）。不過，如果說編寫測試不會遇到任何困難，也不太可能，因此 **@Unit** 會儘量讓這些困難變的微不足道，希望透過這種方式，你們會更樂意編寫測試。

### 實現 @Unit

首先我們需要定義所有的註解類型。這些都是簡單的標籤，並且沒有任何欄位。@Test 標籤在本章開頭已經定義過了，這裡是其他所需要的註解：

```java
// onjava/atunit/TestObjectCreate.java
// The @Unit @TestObjectCreate tag
package onjava.atunit;
import java.lang.annotation.*;
@Target(ElementType.METHOD)
@Retention(RetentionPolicy.RUNTIME)
public @interface TestObjectCreate {}
```

```java
// onjava/atunit/TestObjectCleanup.java
// The @Unit @TestObjectCleanup tag
package onjava.atunit;
import java.lang.annotation.*;
@Target(ElementType.METHOD)
@Retention(RetentionPolicy.RUNTIME)
public @interface TestObjectCleanup {}
```

```java
// onjava/atunit/TestProperty.java
// The @Unit @TestProperty tag
package onjava.atunit;
import java.lang.annotation.*;
// Both fields and methods can be tagged as properties:
@Target({ElementType.FIELD, ElementType.METHOD})
@Retention(RetentionPolicy.RUNTIME)
public @interface TestProperty {}
```

所有測試的保留屬性都為 **RUNTIME**，這是因為 **@Unit** 必須在編譯後的程式碼中發現這些註解。

要實現系統並執行測試，我們還需要反射機制來提取註解。下面這個程式透過註解中的訊息，決定如何構造測試物件，並在測試物件上執行測試。正是由於註解幫助，這個程式才會如此短小而直接：

```java
// onjava/atunit/AtUnit.java
// An annotation-based unit-test framework
// {java onjava.atunit.AtUnit}
package onjava.atunit;
import java.lang.reflect.*;
import java.io.*;
import java.util.*;
import java.nio.file.*;
import java.util.stream.*;
import onjava.*;
public class AtUnit implements ProcessFiles.Strategy {
    static Class<?> testClass;
    static List<String> failedTests= new ArrayList<>();
    static long testsRun = 0;
    static long failures = 0;
    public static void
    main(String[] args) throws Exception {
        ClassLoader.getSystemClassLoader()
                .setDefaultAssertionStatus(true); // Enable assert
        new ProcessFiles(new AtUnit(), "class").start(args);
        if(failures == 0)
            System.out.println("OK (" + testsRun + " tests)");
        else {
            System.out.println("(" + testsRun + " tests)");
            System.out.println(
                    "\n>>> " + failures + " FAILURE" +
                            (failures > 1 ? "S" : "") + " <<<");
            for(String failed : failedTests)
                System.out.println(" " + failed);
        }
    }
    @Override
    public void process(File cFile) {
        try {
            String cName = ClassNameFinder.thisClass(
                    Files.readAllBytes(cFile.toPath()));
            if(!cName.startsWith("public:"))
                return;
            cName = cName.split(":")[1];
            if(!cName.contains("."))
                return; // Ignore unpackaged classes
            testClass = Class.forName(cName);
        } catch(IOException | ClassNotFoundException e) {
            throw new RuntimeException(e);
        }
        TestMethods testMethods = new TestMethods();
        Method creator = null;
        Method cleanup = null;
        for(Method m : testClass.getDeclaredMethods()) {
            testMethods.addIfTestMethod(m);
            if(creator == null)
                creator = checkForCreatorMethod(m);
            if(cleanup == null)
                cleanup = checkForCleanupMethod(m);
        }
        if(testMethods.size() > 0) {
            if(creator == null)
                try {
                    if(!Modifier.isPublic(testClass
                            .getDeclaredConstructor()
                            .getModifiers())) {
                        System.out.println("Error: " + testClass +
                                " no-arg constructor must be public");
                        System.exit(1);
                    }
                } catch(NoSuchMethodException e) {
// Synthesized no-arg constructor; OK
                }
            System.out.println(testClass.getName());
        }
        for(Method m : testMethods) {
            System.out.print(" . " + m.getName() + " ");
            try {
                Object testObject = createTestObject(creator);
                boolean success = false;
                try {
                    if(m.getReturnType().equals(boolean.class))
                        success = (Boolean)m.invoke(testObject);
                    else {
                        m.invoke(testObject);
                        success = true; // If no assert fails
                    }
                } catch(InvocationTargetException e) {
// Actual exception is inside e:
                    System.out.println(e.getCause());
                }
                System.out.println(success ? "" : "(failed)");
                testsRun++;
                if(!success) {
                    failures++;
                    failedTests.add(testClass.getName() +
                            ": " + m.getName());
                }
                if(cleanup != null)
                    cleanup.invoke(testObject, testObject);
            } catch(IllegalAccessException |
                    IllegalArgumentException |
                    InvocationTargetException e) {
                throw new RuntimeException(e);
            }
        }
    }
    public static
    class TestMethods extends ArrayList<Method> {
        void addIfTestMethod(Method m) {
            if(m.getAnnotation(Test.class) == null)
                return;
            if(!(m.getReturnType().equals(boolean.class) ||
                    m.getReturnType().equals(void.class)))
                throw new RuntimeException("@Test method" +
                        " must return boolean or void");
            m.setAccessible(true); // If it's private, etc.
            add(m);
        }
    }
    private static
    Method checkForCreatorMethod(Method m) {
        if(m.getAnnotation(TestObjectCreate.class) == null)
            return null;
        if(!m.getReturnType().equals(testClass))
            throw new RuntimeException("@TestObjectCreate " +
                    "must return instance of Class to be tested");
        if((m.getModifiers() &
                java.lang.reflect.Modifier.STATIC) < 1)
            throw new RuntimeException("@TestObjectCreate " +
                    "must be static.");
        m.setAccessible(true);
        return m;
    }
    private static
    Method checkForCleanupMethod(Method m) {
        if(m.getAnnotation(TestObjectCleanup.class) == null)
            return null;
        if(!m.getReturnType().equals(void.class))
            throw new RuntimeException("@TestObjectCleanup " +
                    "must return void");
        if((m.getModifiers() &
                java.lang.reflect.Modifier.STATIC) < 1)
            throw new RuntimeException("@TestObjectCleanup " +
                    "must be static.");
        if(m.getParameterTypes().length == 0 ||
                m.getParameterTypes()[0] != testClass)
            throw new RuntimeException("@TestObjectCleanup " +
                    "must take an argument of the tested type.");
        m.setAccessible(true);
        return m;
    }
    private static Object
    createTestObject(Method creator) {
        if(creator != null) {
            try {
                return creator.invoke(testClass);
            } catch(IllegalAccessException |
                    IllegalArgumentException |
                    InvocationTargetException e) {
                throw new RuntimeException("Couldn't run " +
                        "@TestObject (creator) method.");
            }
        } else { // Use the no-arg constructor:
            try {
                return testClass.newInstance();
            } catch(InstantiationException |
                    IllegalAccessException e) {
                throw new RuntimeException(
                        "Couldn't create a test object. " +
                                "Try using a @TestObject method.");
            }
        }
    }
}
```

雖然它可能是“過早的重構”（因為它只在書中使用過一次），**AtUnit.java** 使用了 **ProcessFiles** 工具逐步判斷命令列中的參數，決定它是一個目錄還是文件，並採取相應的行為。這可以應用於不同的解決方法，是因為它包含了一個 可用於自訂的 **Strategy** 介面：

```java
// onjava/ProcessFiles.java
package onjava;
import java.io.*;
import java.nio.file.*;
public class ProcessFiles {
    public interface Strategy {
        void process(File file);
    }
    private Strategy strategy;
    private String ext;
    public ProcessFiles(Strategy strategy, String ext) {
        this.strategy = strategy;
        this.ext = ext;
    }
    public void start(String[] args) {
        try {
            if(args.length == 0)
                processDirectoryTree(new File("."));
            else
                for(String arg : args) {
                    File fileArg = new File(arg);
                    if(fileArg.isDirectory())
                        processDirectoryTree(fileArg);
                    else {
// Allow user to leave off extension:
                        if(!arg.endsWith("." + ext))
                            arg += "." + ext;
                        strategy.process(
                                new File(arg).getCanonicalFile());
                    }
                }
        } catch(IOException e) {
            throw new RuntimeException(e);
        }
    }
    public void processDirectoryTree(File root) throws IOException {
        PathMatcher matcher = FileSystems.getDefault()
                .getPathMatcher("glob:**/*.{" + ext + "}");
        Files.walk(root.toPath())
                .filter(matcher::matches)
                .forEach(p -> strategy.process(p.toFile()));
    }
}
```

**AtUnit** 類實現了 **ProcessFiles.Strategy**，其包含了一個 `process()` 方法。在這種方式下，**AtUnit** 實例可以作為參數傳遞給 **ProcessFiles** 構造器。第二個構造器的參數告訴 **ProcessFiles** 如尋找所有包含 “class” 副檔名的文件。

如下是一個簡單的使用範例：

```java
// annotations/DemoProcessFiles.java
import onjava.ProcessFiles;
public class DemoProcessFiles {
    public static void main(String[] args) {
        new ProcessFiles(file -> System.out.println(file),
                "java").start(args);
    }
}
```

輸出為：

```java
.\AtUnitExample1.java
.\AtUnitExample2.java
.\AtUnitExample3.java
.\AtUnitExample4.java
.\AtUnitExample5.java
.\AUComposition.java
.\AUExternalTest.java
.\database\Constraints.java
.\database\DBTable.java
.\database\Member.java
.\database\SQLInteger.java
.\database\SQLString.java
.\database\TableCreator.java
.\database\Uniqueness.java
.\DemoProcessFiles.java
.\HashSetTest.java
.\ifx\ExtractInterface.java
.\ifx\IfaceExtractorProcessor.java
.\ifx\Multiplier.java
.\PasswordUtils.java
.\simplest\Simple.java
.\simplest\SimpleProcessor.java
.\simplest\SimpleTest.java
.\SimulatingNull.java
.\StackL.java
.\StackLStringTst.java
.\Testable.java
.\UseCase.java
.\UseCaseTracker.java
```

如果沒有命令列參數，這個程式會遍歷目前的目錄樹。你還可以提供多個參數，這些參數可以是類文件（帶或不帶.class副檔名）或目錄。

回到我們對 **AtUnit.java** 的討論，因為 **@Unit** 會自動找到可測試的類和方法，所以不需要“套件”機制。

**AtUnit.java** 中存在的一個我們必須要解決的問題是，當它發現類文件時，類檔案名中的限定類名（包括包）不明顯。為了發現這個訊息，必須解析類文件 - 這不是微不足道的，但也不是不可能的。 找到 .class 文件時，會打開它並讀取其二進位制資料並將其傳遞給 `ClassNameFinder.thisClass()`。 在這裡，我們正在進入“位元組碼工程”領域，因為我們實際上正在分析類文件的內容：

```java
// onjava/atunit/ClassNameFinder.java
// {java onjava.atunit.ClassNameFinder}
package onjava.atunit;
import java.io.*;
import java.nio.file.*;
import java.util.*;
import onjava.*;
public class ClassNameFinder {
    public static String thisClass(byte[] classBytes) {
        Map<Integer,Integer> offsetTable = new HashMap<>();
        Map<Integer,String> classNameTable = new HashMap<>();
        try {
            DataInputStream data = new DataInputStream(
                    new ByteArrayInputStream(classBytes));
            int magic = data.readInt(); // 0xcafebabe
            int minorVersion = data.readShort();
            int majorVersion = data.readShort();
            int constantPoolCount = data.readShort();
            int[] constantPool = new int[constantPoolCount];
            for(int i = 1; i < constantPoolCount; i++) {
                int tag = data.read();
                // int tableSize;
                switch(tag) {
                    case 1: // UTF
                        int length = data.readShort();
                        char[] bytes = new char[length];
                        for(int k = 0; k < bytes.length; k++)
                            bytes[k] = (char)data.read();
                        String className = new String(bytes);
                        classNameTable.put(i, className);
                        break;
                    case 5: // LONG
                    case 6: // DOUBLE
                        data.readLong(); // discard 8 bytes
                        i++; // Special skip necessary
                        break;
                    case 7: // CLASS
                        int offset = data.readShort();
                        offsetTable.put(i, offset);
                        break;
                    case 8: // STRING
                        data.readShort(); // discard 2 bytes
                        break;
                    case 3: // INTEGER
                    case 4: // FLOAT
                    case 9: // FIELD_REF
                    case 10: // METHOD_REF
                    case 11: // INTERFACE_METHOD_REF
                    case 12: // NAME_AND_TYPE
                    case 18: // Invoke Dynamic
                        data.readInt(); // discard 4 bytes
                        break;
                    case 15: // Method Handle
                        data.readByte();
                        data.readShort();
                        break;
                    case 16: // Method Type
                        data.readShort();
                        break;
                    default:
                        throw
                                new RuntimeException("Bad tag " + tag);
                }
            }
            short accessFlags = data.readShort();
            String access = (accessFlags & 0x0001) == 0 ?
                    "nonpublic:" : "public:";
            int thisClass = data.readShort();
            int superClass = data.readShort();
            return access + classNameTable.get(
                    offsetTable.get(thisClass)).replace('/', '.');
        } catch(IOException | RuntimeException e) {
            throw new RuntimeException(e);
        }
    }
    // Demonstration:
    public static void main(String[] args) throws Exception {
        PathMatcher matcher = FileSystems.getDefault()
                .getPathMatcher("glob:**/*.class");
// Walk the entire tree:
        Files.walk(Paths.get("."))
                .filter(matcher::matches)
                .map(p -> {
                    try {
                        return thisClass(Files.readAllBytes(p));
                    } catch(Exception e) {
                        throw new RuntimeException(e);
                    }
                })
                .filter(s -> s.startsWith("public:"))
// .filter(s -> s.indexOf('$') >= 0)
                .map(s -> s.split(":")[1])
                .filter(s -> !s.startsWith("enums."))
                .filter(s -> s.contains("."))
                .forEach(System.out::println);
    }
}
```

輸出為：

```java
onjava.ArrayShow
onjava.atunit.AtUnit$TestMethods
onjava.atunit.AtUnit
onjava.atunit.ClassNameFinder
onjava.atunit.Test
onjava.atunit.TestObjectCleanup
onjava.atunit.TestObjectCreate
onjava.atunit.TestProperty
onjava.BasicSupplier
onjava.CollectionMethodDifferences
onjava.ConvertTo
onjava.Count$Boolean
onjava.Count$Byte
onjava.Count$Character
onjava.Count$Double
onjava.Count$Float
onjava.Count$Integer
onjava.Count$Long
onjava.Count$Pboolean
onjava.Count$Pbyte
onjava.Count$Pchar
onjava.Count$Pdouble
onjava.Count$Pfloat
onjava.Count$Pint
onjava.Count$Plong
onjava.Count$Pshort
onjava.Count$Short
onjava.Count
onjava.CountingIntegerList
onjava.CountMap
onjava.Countries
onjava.Enums
onjava.FillMap
onjava.HTMLColors
onjava.MouseClick
onjava.Nap
onjava.Null
onjava.Operations
onjava.OSExecute
onjava.OSExecuteException
onjava.Pair
onjava.ProcessFiles$Strategy
onjava.ProcessFiles
onjava.Rand$Boolean
onjava.Rand$Byte
onjava.Rand$Character
onjava.Rand$Double
onjava.Rand$Float
onjava.Rand$Integer
onjava.Rand$Long
onjava.Rand$Pboolean
onjava.Rand$Pbyte
onjava.Rand$Pchar
onjava.Rand$Pdouble
onjava.Rand$Pfloat
onjava.Rand$Pint
onjava.Rand$Plong
onjava.Rand$Pshort
onjava.Rand$Short
onjava.Rand$String
onjava.Rand
onjava.Range
onjava.Repeat
onjava.RmDir
onjava.Sets
onjava.Stack
onjava.Suppliers
onjava.TimedAbort
onjava.Timer
onjava.Tuple
onjava.Tuple2
onjava.Tuple3
onjava.Tuple4
onjava.Tuple5
onjava.TypeCounter
```

 雖然無法在這裡介紹其中所有的細節，但是每個類文件都必須遵循一定的格式，而我已經盡力用有意義的欄位來表示這些從 **ByteArrayInputStream** 中提取出來的資料片段。透過施加在輸入流上的讀操作，你能看出每個訊息片的大小。例如每一個類的頭 32 個 bit 總是一個 “神秘數字” **0xcafebabe**，而接下來的兩個 **short** 值是版本訊息。常量池包含了程式的常量，所以這是一個可變的值。接下來的 **short** 告訴我們這個常量池有多大，然後我們為其建立一個尺寸合適的陣列。常量池中的每一個元素，其長度可能是固定式，也可能是可變的值，因此我們必須檢查每一個常量的起始標記，然後才能知道該怎麼做，這就是 switch 語句的工作。我們並不打算精確的分析類中所有的資料，僅僅是從文件的起始一步一步的走，直到取得我們所需的訊息，因此你會發現，在這個過程中我們丟棄了大量的資料。關於類的訊息都儲存在 **classNameTable** 和 **offsetTable** 中。在讀取常量池之後，就找到了 **this_class** 訊息，這是 **offsetTable** 的一個坐標，透過它可以找到進入  **classNameTable** 的坐標，然後就可以得到我們所需的類的名字了。

現在讓我們回到 **AtUtil.java** 中，process() 方法中擁有了類的名字，然後檢查它是否包含“.”，如果有就表示該類定義於一個包中。沒有包的類會被忽略。如果一個類在包中，那麼我們就可以使用標準的類載入器透過 `Class.forName()`  將其載入進來。現在我們可以對這個類進行 **@Unit** 註解的分析工作了。

我們只需要關注三件事：首先是 **@Test** 方法，它們被儲存在 **TestMehtods** 列表中，然後檢查其是否具有 @TestObjectCreate 和 **@TestObjectCleanup****** 方法。從程式碼中可以看到，我們透過呼叫相應的方法來查詢註解從而找到這些方法。

每找到一個 @Test 方法，就列印出來目前類的名字，於是觀察者立刻就可以知道發生了什麼事。接下來開始執行測試，也就是列印出方法名，然後呼叫 createTestObject() （如果存在一個加了 @TestObjectCreate 註解的方法），或者呼叫預設構造器。一旦建立出來測試物件，如果呼叫其上的測試方法。如果測試的返回值為 boolean，就捕獲該結果。如果測試方法沒有返回值，那麼就沒有異常發生，我們就假設測試成功，反之，如果當 assert 失敗或者有任何異常拋出的時候，就說明測試失敗，這時將異常訊息列印出來以顯示錯誤的原因。如果有失敗的測試發生，那麼還要統計失敗的次數，並將失敗所屬的類和方法加入到 failedTests 中，以便最後報告給使用者。

<!-- Summary -->

## 本章小結

註解是 Java 引入的一項非常受歡迎的補充，它提供了一種結構化，並且具有類型檢查能力的新途徑，從而使得你能夠為程式碼中加入中繼資料，而且不會導致程式碼雜亂並難以閱讀。使用註解能夠幫助我們避免編寫累贅的部署描述性文件，以及其他的生成文件。而 Javadoc 中的 @deprecated 被 @Deprecated 註解所替代的事實也說明，與注釋性文字相比，註解絕對更適用於描述類相關的訊息。

Java 提供了很少的內建註解。這意味著如果你在別處找不到可用的類庫，那麼就只能自己建立新的註解以及相應的處理器。透過將註解處理器連結到 javac，你可以一步完成編譯新生成的文件，簡化了構造過程。

API 的提供方和框架將會將註解作為他們工具的一部分。通過 @Unit 系統，我們可以想像，註解會極大的改變我們的 Java 編程體驗。

<!-- 分頁 -->

<div style="page-break-after: always;"></div>

[^3 ]: The Java designers coyly suggest that a mirror is where you find a reflection.
