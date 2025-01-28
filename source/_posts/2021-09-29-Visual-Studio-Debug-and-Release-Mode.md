---
title: Visula Studio Debug & Release Mode
date: 2021-09-29 05:00:11
tags:
  - "Visual Studio"
  - "Debug Mode"
  - "Release Mode"
---

![](/images/Visual-Studio-Debug-and-Release-Mode/Debug-and-Release-Config.png)

本篇會來釐清 Visual Studio 中的 Debug & Release Mode，以及其相關設定。

其實兩者並無本質上的界線，他們只是一組編譯選項的標籤，編譯器只會按照選項上的設定來執行。也就是說，**兩者僅是不同設定的編譯器設定檔**。

<!--more-->

### Debug Mode
通常稱為測試/偵錯模式，在此模式下編譯時，程式並不會做任何優化，且附加很多測試資料，因此執行效率較慢、檔案大小較大，但會便於開發者開發與除錯。

### Release Mode
通常稱為發布模式，由於是要給使用者使用，所以在此模式編譯後，程式會做各種優化，因此執行效率較佳、檔案大小較小。

Release 版本的 exe 檔，連結的目標是標準的 MFC (Microsoft Foundation Library) dll，這些在安裝 Windows 時就已經會安裝上去了（位於 Windows 的 System 或 System32 資料夾底下），所以**Release 版本可以在沒有安裝 VC++ 的電腦上執行**。反之，Debug 版本則是調用測試版本的 dll（在 VS 環境安裝時會裝上，如VC++），所以 Debug 版本在沒有安裝 VC++ 的電腦上就不能執行，除非手動 copy 所需用到的 dll，但這樣太麻煩了。

而在 Release Mode 中不能做一些 Debug 的行為，如設中斷點後，開始偵錯，而後編輯再繼續。如嘗試這麼做的話，會跳錯，如下圖所示：

![](/images/Visual-Studio-Debug-and-Release-Mode/Release-Mode-cant-edit-and-continue.png)

### Debug & Release 參數定義
#### Debug 版本
|參數 | 定義|
|:-:|:-|
|/MDd /MLd 或 /MTd|使用 Debug runtime library（測試版本的執行時函式庫）。|
|/Od|關閉優化開關。|
|/D "_DEBUG"|相當於 #define _DEBUG，打開編譯測試程式開關（主要針對 assert 函式）。|
|/ZI|創建 Edit and continue（編輯繼續）資料庫，在測試過程中如果修改了程式碼不需重新編譯。|
|/GZ|可以幫助捕獲記憶體的錯誤。|
|/Gm|打開最小化重 link 開關，減少 link 時間。|

#### Release 版本
|參數 | 定義|
|:-:|:-|
|/MD /ML 或 /MT|使用 Release runtime library（發布版本的執行時函式庫）。|
|/O1 或 /O2|優化開關，使程式最小或最快。|
|/D "NDEBUG"|關閉條件編譯測試程式開關（即不編譯 assert 函式）。|
|/GF|合併重複的字串，並將字串常量放到 ROM，防止被修改。|。


### 參考資料
1. Microsoft 官方文件：[https://docs.microsoft.com/zh-tw/visualstudio/debugger/how-to-set-debug-and-release-configurations?view=vs-2019](https://docs.microsoft.com/zh-tw/visualstudio/debugger/how-to-set-debug-and-release-configurations?view=vs-2019)
2. [https://stackoverflow.com/questions/933739/what-is-the-difference-between-release-and-debug-modes-in-visual-studio/938665](https://stackoverflow.com/questions/933739/what-is-the-difference-between-release-and-debug-modes-in-visual-studio/938665)
3. [http://johnniebooks.blogspot.com/2009/05/debug-release.html](http://johnniebooks.blogspot.com/2009/05/debug-release.html)