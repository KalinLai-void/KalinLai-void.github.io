---
title: 【TGDF2021】遊戲線上內容的管理與交付（Unity）
date: 2021-07-10 12:56:33
tags:
  - TGDF台北遊戲開發者論壇
  - TGDF2021
  - 遊戲製程與管理
  - Unity
  - "Addressable Asset System"
  - "Cloud Content Delivery Network"
categories:
  - TGDF2021_台北遊戲開發者論壇
---
講者：Arturo Nereu

[演講影片](https://youtu.be/u2ng_19lYvY)

本議程較偏 Demo，使用 Unity，以手遊為例。
使用 Unity 的 **Addressable Asset System** 來管理素材；
使用 Unity 的**雲端內容交付系統（Cloud Content Delivery，CCD）**來推送遊戲內容給玩家們。
<!--more-->

講者介紹
---
- Unity 營運解決方案的技術推廣工程師
- 講者Email：arturo@unity3d.com
- [Twitter](https://twitter.com/arturonereu)

為什麼需要管理素材？
---
- editor 允許我們把所有素材放進去了
- 然而現在的遊戲越來越複雜 -> **素材量增加**
- 提供更好的玩家體驗，因此需進行素材管理
  - e.g. 日後更新，只需透過雲端安裝即可
- 本議程以手遊做示範，當然也可適用於 PC / Console
  - **只是方式與今天介紹的較為不同**，不過想法上是相同的

Unity Cloud Content Delivery (CCD)
---
[官網](https://unity.com/products/cloud-content-delivery)
[官方文件](https://docs.unity3d.com/Manual/UnityCCD.html)

- 此 Unity 服務支援「**跨引擎**」，但針對 unity 有較好的整合
- 後端有強大的內容傳遞網路 (Content Delivery Network，CDN) 支援

### How it works?
1. 將執行檔和遊戲資源分開
    - 執行檔上傳至遊戲商店
    - 遊戲資源上傳至 CCD
2. 遊戲發行後，獲得某資源的玩家會自行從 CCD 下載
    - 只將內容傳遞給需要的使用者

### Concept
- Bucket （儲存桶）
  - Container of Asset
  - 儲存資料的最大單位
  - 可以擁有多個 Bucket
    - 可依環境區分。e.g. 測試版與正式版
- Entry
  - 相當於在 Bucket 內的檔案
- Release
  - 一個 Bucket 內的 Entry 的快照 (snapshot)，可以做為版本釋出
- Badge
  - 用來標註 Release 用，用戶端的 App 可以透過 Badge 來下載特定 Release
  - 一個 Release 只能指派一個 Badge
- The promotion process（推進流程）
  ![](/images/TGDF2021_Addressables-and-Cloud-Content-Delivery/The-promotion-process.png)

Unity Addressable Asset System
---
- 可以從 Package Manager 安裝套件
  - ```Window -> Package Manager```，確認 Addressable 是否安裝
  - 安裝完後在```Window -> Asset Management```內會看到 Addressable 選項
- 用單一網址去存取資源，可以設定是要從本地或者其他網址存取
- 可以將不必在執行時就需要的資源先分裝在外部伺服器，需要時再載入
- 可以對不同種類資源做分組
- 協助管理資源相依性問題，比如打包一個場景，他會把場景需要的素材也打包進去
- 要把資源加入到 Addressable，可以拖曳到視窗內，或者在 inspector 上勾選 Addressable
- Addressable 主要是使用非同步方式進行載入或下載
  - 非同步：如果想載入一個場景，當開始載入場景時，若場景不存在，會嘗試下載，但遊戲接下來的畫面更新仍可正常執行，直到場景載入完成並出現。
  - 同步：如果想載入一個場景，需等到所有內容都載入完畢才能繼續。
  - ![](/images/TGDF2021_Addressables-and-Cloud-Content-Delivery/Addressable-API.png)
- 用 ```Addressable.AsyncOperationHandle``` 來監聽 addressable 作業完成的事件

Cloud Content Delivery
---
- 前往 [Unity Dashboard](https://dashboard.unity3d.com)
- 先在 Dashboard 建立專案
- ```展開左上角的圖示 -> Cloud Content Delivery -> Project -> 選擇專案```，然後點左邊的 Onboarding 就會引導所有流程
- 創建好 Bucket 後就可以拿到 Addressables remote path，在 Unity 設定貼上後就可以開始用
- 接著在 Addressables -> Group 裡做 build，就會產生對應平台的 Asset Bundles，產生的就上傳到 Bucket
- 增加 Release 與 Badge 後就可以回到 Unity 測試
- 可以再 Addressables -> Group 中，修改執行模式成 Existing Builds，就會實際使用到雲端上的資料