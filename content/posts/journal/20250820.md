+++
title = 'RESTful API極淺談'
date = 2025-08-20T22:34:59+08:00
draft = false
tags = ["Web Development"]
+++

## 什麼是RESTful API
最煩面試被問這類型的問題，就是其實你會但很容易沒回答好。

RESTful API顧名思義就是一種符合REST風格的API設計。

REST全名是REpresentational State Transfer - 表徵狀態轉換，在Web開發情況下，「表徵」指的是資源的某種格式呈現，可以是JSON、XML或是前端最常見的HTML；而對於使用者來說，跟網站互動像是查詢商品資訊、發廢文、取消訂單等就是所謂的「狀態轉換」。

而RESTful這種設計風格，就是當表徵狀態轉換時，能夠明確地跟Server做溝通。重點在於要「明確」，指的是讓工程師們可以直觀地從這個HTTP請求看出這支API想幹嘛，因此RESTful API設計時會明確規範兩件事情：
1. Nouns: 欲操作的資源的identifier，指的是URLs，例如查看訂單的URL可能會是 `https://hankmallapi/v1/orders`

2. Verb: 表明要對Nonus做的操作，包含創建、查看、更新、刪除（CRUD）。由於Web開發時實作的通訊協定是HTTP，分別對應HTTP method的Get、Post、Put(or Patch)、Delete

因此我們在看RESTful API文件時都會是一個HTTP method配一個URL，例如：`Get` `https://hankmallapi/v1/orders`，可以很直觀地猜這支API是要獲取所有的訂單資料。

以上論述只涵蓋了RESTful的一部分，Roy Thomas Fielding在提出時還講了幾個限制: 
1. Client - Server
2. Statelessness
3. Cacheability
4. Uniform Interface
5. Layered System
6. Code-On-Demand

小結: RESTful只是一種設計風格，它很適合Public API或是那種Customer-facing的Service，這種Resource導向的風格簡單易懂，設計上也天然符合HTTP協定，但是每個公司對RESTful API的實作上會有些許不同，大部分時候還是要遵循公司文件的規則來開發。

除了RESTful以外，常見的還有GraphQL、JSON-RPC、gRPC，SSE等等，改天再來講講RESTful API其他特性以及什麼時候要選擇RESTful。
