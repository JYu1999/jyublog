---
title: "SOLID 原則（二）：OCP & DIP"
date: 2023-06-18T15:52:19+08:00
draft: false
keywords:
- SOLID
- 物件導向
- OOP
- Object Oriented Programming
- System Architecture
categories:
- software_design_and_architecture
image: "0.webp"
---

## 前言

在上一篇 [SOLID 原則（一）：Top-down 看 SOLID](https://jyu1999.com/post/solid/) 有說明了 SOLID 的目的，從這篇開始逐一解說～

大多數的文章都是從 SRP 開始照順序介紹，但我自己覺得 SRP 是最難懂的...

所以我從我覺得比較簡單的，逐一介紹，希望能降低讀者的閱讀門檻XD

---

## OCP(Open-Closed Principle)

## What is OCP?

這個原則相對簡單，我們先來看一下定義：

> _Objects or entities should be open for extension but closed for modification._

其實這個定義我覺得很難以理解，我們看一下 _Clean Architecture_ 這本書是怎麼說的：

> _軟體製品的行為應該是可擴展的，而且沒必要修改該製品_

我們再來看一下[維基百科](https://zh.wikipedia.org/wiki/%E5%BC%80%E9%97%AD%E5%8E%9F%E5%88%99)：

> _這意味著一個實體是允許在不改變它的原始碼的前提下變更它的行為_

大家可以想一下，假設我裝了一個 Google Chrome 的 plugin，是要把每個頁面的字都變成白色的。

如果這個 plugin 直接去改 Chrome 的原始碼，會發生什麼事情呢？

首先可能是如果 plugin 要改很多地方的話，之後可能會很難把它改回來。

再來，怎麼確認改了這些地方，不會造成其他地方的 bug？

因此，比較理想的方式是，這個 plugin 可能會去繼承（or implement）Google Chrome 用來控制字體的 Class（or Interface），然後去覆蓋掉原本的設定。

這樣我可以保證其他用到這個 Class(Interface) 的程式不會出問題，到時候要改回去也很簡單。

如果想看範例的，可以看看 [SOLID: The First 5 Principles of Object Oriented Design](https://www.digitalocean.com/community/conceptual-articles/s-o-l-i-d-the-first-five-principles-of-object-oriented-design#open-closed-principle)


---


## DIP(Dependency Inversion Principle)

### What is DIP?

一樣，先來看一下定義：

> _Entities must depend on abstractions, not on concretions. It states that the high-level module must not depend on the low-level module, but they should depend on abstractions._

定義看不懂沒關係，但最好先記著，下面的解釋會一直圍繞著定義。

### Example

我們先來看看實際的範例（[範例來源](https://www.digitalocean.com/community/conceptual-articles/s-o-l-i-d-the-first-five-principles-of-object-oriented-design#dependency-inversion-principle)）。

假設我們今天要實作一個 `PasswordReminder` 的 Class 

```php
class MySQLConnection
{
    public function connect()
    {
        // handle the database connection
        return 'Database connection';
    }
}

class PasswordReminder
{
    private $dbConnection;

    public function __construct(MySQLConnection $dbConnection)
    {
        $this->dbConnection = $dbConnection;
    }
}
```

這個 `MySQLConnection` 就是 low-level module，`PasswordReminder` 是 high-level module。

但根據定義：_high-level module must not depend on the low-level module_，我們知道 `PasswordReminder` 依賴 `MySQLConnection` 是違反 DIP 的。`PasswordReminder` 應該要 _depend on abstractions_ 才對。

知道是知道啦，但是為什麼不行？我們可以試想一個情況：

> 主管突然宣布：「從今天開始，我們的 Database 不用 MySQL 了，改用 PostgreSQL 吧！」

這時候工程師可能需要寫一個新的 `PostgreSQLConnection`，然後更改 `PasswordReminder`：

```php
class PasswordReminder
{
    private $dbConnection;

    public function __construct(PostgreSQLConnection $dbConnection)
    {
        $this->dbConnection = $dbConnection;
    }
}
```

乍看之下很合理，但仔細想想就會有不對勁了：__我換個 DB Driver，甘 `PasswordReminder` 什麼事啊？__
而且其實也違反了 OCP。

也就是說，`PasswordReminder` 其實不需要知道底層是用什麼資料庫實現的，它所需要的只是建立連線而已。

ok，所以我們可以把 `connect()` 這個行為「抽象化」，讓 `MySQLConnection` 這個 low-level module，`PasswordReminder` 這個 high-level module，都可以 _depend on abstraction_：

```php
interface DBConnectionInterface
{
    public function connect();
}
```

這個 interface 只有一個 `connect()` method，然後 `MySQLConnection` 會去 implements 這個 interface。

此外，在 `PasswordReminder` 裡面，我們也可以把 type hint 換成 `DBConnectionInterface`，不用再去管用到什麼 DB Driver 了！

```php
class MySQLConnection implements DBConnectionInterface //MySQLConnection 去 implements 這個 interface
{
    public function connect()
    {
        // handle the database connection
        return 'Database connection';
    }
}

class PasswordReminder 
{
    private $dbConnection;

    public function __construct(DBConnectionInterface $dbConnection) //把 type hint 換成 DBConnectionInterface
    {
        $this->dbConnection = $dbConnection;
    }
}
```

所以如果有人需要用到 `PasswordReminder` 這個功能的話，在使用的時候就要一併指定 DB Driver:

```php
$password = new PasswordReminder(MySQLConnection)
```

然後如果老闆提出要換資料庫的話，我們就新增一個 `PostgreConnection`：

```php
class PostgreConnection implements DBConnectionInterface 
{
    public function connect()
    {
        // handle the database connection
        return 'Database connection';
    }
}

// 要使用 PasswordReminder 的話：
$password = new PasswordReminder(PostgreSQLConnection)
```

這樣的做法其實是一種將低層模組(low-level module)的控制權從原來的高層模組(high-level module)中抽離，將兩者的耦合只放在抽象層(abstraction)上。[^1]

[^1]: [使人瘋狂的 SOLID 原則：依賴反向原則 (Dependency Inversion Principle)](https://medium.com/@ChunYeung/%E4%BD%BF%E4%BA%BA%E7%98%8B%E7%8B%82%E7%9A%84-solid-%E5%8E%9F%E5%89%87-%E4%BE%9D%E8%B3%B4%E5%8F%8D%E5%90%91%E5%8E%9F%E5%89%87-dependency-inversion-principle-a74ca045d776)

### 過猶不及

那可能有人會想到，如果我們用更宏觀的角度來看系統的話，我們寫的 code 不是一樣依賴於 Framework 和 Database，甚至是 Operation System 嗎？

是也沒錯，但我們可以來看看 _Clean Architecture_ 怎麼說：

> _我們想要避免依賴的是系統中 **容易變化** 的具體元素_。它們是我們正在發展且正在經常變化模組。
> 
> 基於上述理由，在應用 DIP 時，我們傾向於忽略作業系統和平台機制的穩定背景。我們容忍那些具體的依賴關係，因為我們信任它們不會改變。

所以其實我們可以忽略對於 DB, Framework 乃至於 OS 的依賴～

## 留白

> 😅
> 其實本來還想寫更多的，但考量到這篇文章設定是給初入 SOLID 的人看的，就先留白吧～~~絕對不是因為我自己也看不懂~~
> 
> 但怕有些人求知慾太旺盛，提供一些關鍵字以供參考
> 
> 在 OCP 的部分，本來想要補充實際應用案例。（Controller, Presenter, Interactor 那些）
> 
> 在 DIP 的部分，還想要補充 Factory Pattern 和 _Clean Architecture_ 提到的編碼實戰守則。
> 
> 就留給大家自己研究，也有可能我有空會再寫 XD 


