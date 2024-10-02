---
title: "看完 Heptabase 官方教學的想法"
date: 2024-10-02T19:04:52+08:00
draft: false
---
開始執行我的學習規劃後幾天，我意識到一個問題是：我的學習方式很久沒有精進了。

倒不是說原本的學習方式出了什麼問題， 可是現在 AI 越來越先進，然後好用的軟體產品越來越多，理論上我應該要有更好的方式來進行學習。

對於高強度學習來說，學習方式會大幅影響到我的學習效率，也和學習成效有高度正相關。

所以我決定先暫停手邊的所有 Tasks， 專心來改善這一塊。

目標是讓我自己在**吸收知識、整理知識、使用知識的流程更順暢、更有效率**。

首先，我開始嘗試調查一些學習方式， 找到了 Heptabase 的創辦人所拍的影片。

看完三集之後，我不僅了解 Heptabase 使用方式，也一併反省了過去在學習上所遇到的問題。

## Reusable

首先第一點是「reusable」。 他的影片中非常強調這個概念， 也就是所有我們的筆記都應該要預期，它在未來會被大量重複使用。

> Heptabase Fundamentals 101: Se... → The reason that we designed card library this way is because we believe that knowledge should be reusable across topics.

這個應該也是卡片盒筆記法的核心觀念。

過往我的筆記， 為了追求結構完整，常常會寫得非常的冗長，文字、敘述非常的多。 這樣的後果是：

第一，我不喜歡寫筆記，因為每次都要寫很久。

第二，如果我需要回去查看我筆記所記錄的內容，會很難快速找到我想要的，所以我寧願去 Google，甚至現在多了 Perplexity or Felo 可以用，反而能夠更快地得到答案。

而卡片盒筆記法可以很好的避免這兩件事情。

卡片盒筆記法的核心概念，就是要讓我們在需要的時候能夠快速找到我們過去記錄下來的的知識。 我們要能快速地理解、知道當時的思考脈絡。

> Heptabase Fundamentals 101: Se... → When you revisit them, you can immediately remember how you think even after a long time.

為了達到上述的目的， 我們需要將知識點盡可能地切分。 每張卡片都盡量記錄著單一個知識或想法即可。 

好處是我們未來不僅很快就可以找到這張卡片，也可以在很短的時間就理解這張卡片在講的內容。 

而且因為紀錄的東西足夠單一，不會像文章一樣有很多不相干的內容，所以我們可以很方便地重複使用這個知識點。

這其實讓我想到軟體開發的概念，也就是所謂的 SRP（Single Responsibility Principle）：單一職責原則。SRP 最重要的概念就是「一個模組應該只對唯一的一個角色負責」。

如果一個 Function 做了太多事情，裡面有太多的邏輯都綁在一起， 可能包含了接收前端的 Request、Validate 前端的 Request、進行商業邏輯上的資料處理和確認、 存進資料庫、進行錯誤處理、回傳給前端 Response。

這個時候，如果我們想要去重複使用這一個 Function 裡面的某一個功能，假設是進行錯誤處理的部分好了，我們根本就沒有辦法單獨取出，去重複進行使用。

而且當我要對這個 Function 進行測試的時候，因為所有的變因排列組合太過複雜，導致測試困難。

也就是高耦合（Coupling）的現象。

再往下延伸，這也跟軟體專案管理概念很像：應該要盡可能地把一個 Issue， 盡可能切分成夠細的 subtasks。不過這部分先不展開來討論了。

## Re-structure

再來第二點是影片中有提到 Whiteboard 的設計。

> Heptabase Fundamentals 103: Ma... → Whiteboard is particularly good at breaking down complex knowledge into atomic pieces and connect and visualize them to better see the relationship between ideas.

可以看到他裡面強調兩個概念，第一個是我們要去拆解複雜的知識，第二個是 我們要能清楚的知道知識之間彼此的關聯跟架構。

第一個概念其實就是上面提到的「卡片」。

不過上面有提到，「卡片」所解決的問題是要讓知識能被重複使用，但學習這件事情並不是把知識單純的分解就可以了。

> Heptabase Fundamentals 101: Se... → The essence of learning is you have consumed all this information, but then you restructure it in your own way

就像我們在寫程式的時候，可能有很多個 Function or Class。他已經被分解好了，但我們仍然需要知道他們彼此之間的關係，才有辦法組成一個新的 Function。



## When to create Whiteboard ?

看到這裡我開始思考，那什麼情境適合創建一個 Whiteboard？或者說具體應該要怎麼使用？

> Heptabase Fundamentals 101: Se... → think about a topic that you care about and then create a whiteboard

他給出的答案意外的簡單，其實只要任何你現在在意的事情，都可以是一個 Whiteboard？

我想原因應該是因為：現在在意的事情，代表我們需要解決某個問題。而因為要解決這個問題，所以我們需要進行知識的重新整理。 

而他也提到了一個重要的概念：

> Heptabase Fundamentals 102: Or... → you don't really need to figure out your organization from the very beginning. Once a whiteboard grow big, that's the best time you think about creating the sub-whiteboard

以前我會花非常多的時間，在筆記架構的構想上。希望能想出一套完美的解決方案，讓我以後都不用再調整架構。

但筆記架構會需要隨著規模改變而需要進行調整，因為規模改變會伴隨著一些從來沒有想過的問題。

所以其實，就先做下去就對了XD

## Next-up

再來預計想研究一下 Readwise 官方影片、詹雨安 Blog，還有參考其他人是如何整合 Readwise 和 Heptabase 的。

此外，對於如何使用 Heptabase 進行語言學習也尚需研究一下。

再進一步的話，是如何透過筆記和 AI 的整合，打造一個 Knowledge Agent，進一步提升資料可重用性。不過這不是短期內最需要被解決的事情。

## Reference

- [Heptabase Fundamentals 101: Sense-making with whiteboards](https://www.youtube.com/watch?v=HgvR2QkfwG0&t=38s)

- [Heptabase Fundamentals 102: Organizing topics with nested whiteboards and tab groups](https://www.youtube.com/watch?v=zlGRxZHlDgM)

- [Heptabase Fundamentals 103: Managing card databases with tags and properties](https://www.youtube.com/watch?v=4kwIfzIJ0o0)

- [深入淺出單一職責原則 Single Responsibility Principle](https://www.jyt0532.com/2020/03/18/srp/)


