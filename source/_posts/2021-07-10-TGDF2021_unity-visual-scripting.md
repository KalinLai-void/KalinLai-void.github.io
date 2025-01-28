---
title: 【TGDF2021】Unity可視化程式（Visual Scripting）專案開發
date: 2021-07-10 11:58:56
tags:
  - TGDF台北遊戲開發者論壇
  - TGDF2021
  - Unity
  - 遊戲程式
  - "Visual Scripting"
categories:
  - TGDF2021_台北遊戲開發者論壇
---
講者：達哥/Kelvin Lo

[演講影片](https://youtu.be/bQ7ClW_nsdk)
此筆記不會介紹如何使用，不過影片內講師有操作，或自行上網找資源。

其實就是 Unity 中的藍圖系統 (Blueprint System)、可視化程式 (Visual Scripting)。
<!--more-->

講者介紹
---
服務於 Unity 已經有 8 年，主要任務為提供台灣開發者使用 Unity 上的技術支援，同時也在各大平台推廣 Unity。同時也是 TGDF 的常客，致力於把最新的 Unity 內容同步的傳達到台灣開發者社群。

Unity Visual Scripting
---
- 跨功能的節點式 (Node-based) 設計系統
  - Shader Graph
  - Visual Effect Graph
  - **Visual Scripting Tool** <- 本次議程重點
- 之前的 Unity 若想使用這類可式化節點設計工具，需安裝第三方套件
  - e.g. PlayMaker, Bolt, etc.
- **而在 Unity 收購 Bolt 這個套件，並整合至 Unity2021**
  - 但若是2021以前的版本想要使用 Visual Scripting，可以直接去 Unity Asset Store 下載 Bolt
    - 雖然有些功能會和整合進來的不太一樣，但邏輯相通
    - **舊版檔案格式與新版檔案格式並不相容**，後面會介紹如何轉換
- 如何運作？
  - 圖形化的程式產生器，可以不用coding
  - 相當於 Unreal 中的藍圖系統 (Blueprint System)

部分功能與名詞介紹
---
- Scrip Machine 表示單一的腳本狀態
- State Machine 有包含狀態機控制，適合怪物 AI
- Backboard 裡面會定義不同生命週期的變數
  - Graph: 只有自己這張圖可以使用
  - Object: 可以在同個物件，但不同圖使用
  - Scene: 可以在同個場景內共用，無法跨場景
  - App: 整個 App 內都可以用
  - Saved: 會在遊戲結束後也會儲存著

專案轉換舊版本 Bolt 至 Unity 2021
---
- 在網路上搜尋 Unity Visual Scripting 或 Bolt，會找到很多網路教學或專案，但很多專案在 Unity 2021 前就分享了，所以現在用 Unity 2021 開啟這些專案可能會壞掉。
- 可以在 Unity 內的 ```Project Setting -> Visual Scripting``` 裡面 **Fix Missing Scripts**
  - 會嘗試把舊版本 Bolt 升級成 Unity 2021 的版本

講者觀點
---
- Visual Scripting 誰適合使用？
  - **程式設計師**
    - 因為概念邏輯相通，也很程式導向，能最快進入狀況。
    - 但排斥感可能也最大，因為有限制。
  - PM / 企劃人員
    - **適合做 Prototype 開發** ，因為搭建很快，不需寫code，也不用麻煩程式設計師。
  - 任何想要開發遊戲的開發者
    - 可能對自己的程式能力沒有信心
    - 多了個開發方法，降低了門檻
- 會取代傳統程式碼嗎？
  - 不會，應該會共生。
  - 也可能產生出「**可視化程式設計師**」xD

Q & A
---
1. 我問的問題：請問 Unity 的 Visual Scripting 是否能串聯(讀取) C# script內的function和變數（像 Unreal 的 Blueprint 那樣）
      - 可以在同一個物件寫另外的 C# 腳本，變數可以直接取用。
2. Visual Scripting 在多人工作上的 Merge 是否有問題，檔案是文本基礎嗎？（因為 Unreal 的 Blueprint 就有這種問題）
      - 他是 Component based，所以是以元件的形式跟著物件的，跟其他元件邏輯一樣，並非文本格式就是了。多人工作建議還是一個人負責編輯。
3. 請問如用 Visual Scripting 做完 Prototype 後，想轉成 C# Script 可以直接轉換？或需要手工重寫？
      - 如果你點元件旁的「...」，可以看到一個「Edit script」，裡面會顯示腳本，但是這個腳本是含框架的，所以會比較難讀懂，有可能會比你重寫來的慢，取決於圖的複雜度。
