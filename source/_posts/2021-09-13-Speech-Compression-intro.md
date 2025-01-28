---
title: 【Speech Compression】PortAudio環境設定 (for Windows using Visual Studio)
date: 2021-09-13 17:46:58
tags:
  - "語音壓縮"
  - "Speech Compression"
  - "PortAudio"
categories:
  - "Speech Compression"
  - "PortAudio"
---

什麼是 PortAudio？
---
PortAudio 是一個免費、跨平台且開源的聲音 I/O 函式庫，讓開發者可以使用 C/C++ 撰寫簡單的聲音程式，並且可以在 Windows、MacOS 及 Unix 等多平台上編譯並執行，且許多應用程式也是使用 PortAudio 作為聲音的I/O。

PortAudio 提供了非常簡單的API，供開發者錄音、播放聲音等。同時也包括正弦波輸出、處理音頻輸入（吉他Fuzz等）以及列出能夠使用的聲音裝置清單...等。

<!--more-->

安裝 PortAudio
---
1. 進入 [PortAudio 官網](http://www.portaudio.com/)下載 PortAudio 壓縮檔
2. 解壓縮檔案

環境設定
---
1. 進入路徑 ```portaudio\build\msvc```
2. 使用 Visual Studio 打開檔案 ```portaudio.sln```
3. 開啟專案設定（Project -> Properties）
   - 上方的 Configurations 及 Platforms 設為「All configurations」及「All platforms」
     - 如下圖紅框所示
    ![](/images/Speech-Compression-intro/Configs-and-Platforms.png)

   - 左方導覽列，進入 ```C/C++ -> Code Generation```
     - Runtime Library 設為 **Multi-threaded (/MT)**
       - 語音通訊一定是多執行緒
     - Floating Point Model 設為 **Precise (/fp:precise)**
       - 浮點數的模型，這邊設為精確
     - Struct Number Alignment 設為 **Default** （<u>如遇到問題改為 4 Bytes 應該就可以了</u>）
       - Memeory 的對齊
     - 如下圖紅框所示
      ![](/images/Speech-Compression-intro/Code-Generation-Settings.png)

   - 左方導覽列，進入 ```C/C++ -> Optimization```
     - 將 Omit Frame Pointers 設為 **Yes**
     - 如下圖紅框所示
      ![](/images/Speech-Compression-intro/Optimization-Settings.png)

   - 左方導覽列，進入 ```C/C++ -> Preprocessor```
     - 將 Preprocessor Definition 設為以下：
        ```
        PA_USE_ASIO=0
        PA_USE_DS=0
        PA_USE_WMME=1
        PA_USE_WASAPI=1
        PA_USE_WDMKS=1
        PA_USE_SKELETON=0
        ```
     - 設為 0 的代表「不使用該功能」，反之，1 則「啟用」
       - **而目前我們不需要使用 ASIO，所以將其關閉**
     - 而上面那些 ASIO、DS、WMME、WASAPI、WDMKS、SKELETON 就是一些 OS 的驅動程式 (Driver)，可以查看 **[參考資料2](https://www.sweetwater.com/sweetcare/articles/roland-difference-between-asio-wdm-mme-drivers/)** 來看他們功能的解釋
     - 如下圖紅框所示
      ![](/images/Speech-Compression-intro/Preprocessor-Settings.png)

建置與問題排解
---
- 此時其實就可以 Build 了，只是 Build 後會發現以下錯誤：
```
Error C1083 Cannot open include file: 'asiosys.h': No such file or directory portaudio portaudio/portaudio/src/hostapi/asiopa_asio.cpp 115
```
- 主要原因是因為，前面**我們將 ASIO 設為不啟用，所以他也就抓不到該 Driver**，因此我們要將有使用到該 Driver 的部分從專案中移除
- 移除與 ASIO 有關的檔案或資料
   - 將 ```Source Files/hostapi/``` 中的 ASIO 資料夾從專案中移除
   - 進入路徑 ```portaudio\build\msvc```，把 portaudio.def 檔案打開，並將 <strong>PaAsis_* (@50~@55) 的部分全數移除</strong>
- 接下來就可以再次建置了，如果這邊沒問題了那就成功了，如果有跳錯就繼續往下看
  - 可能會跳出以下錯誤，主要是因為這個開源碼在開發時，還是 Win7、XP 的年代，Win10 電腦抓不到此SDK
    ```
    Error: Windows SDK Version 8.1 was not found.
    ```
  - 因此我們要手動更新一下 SDK
    - 打開 Microsoft Visual Studio Installer，**更新版本**（不過事實上去看時可能會發現 Win10 SDK 已經安裝）
    - 如 Win10 SDK 已安裝（或更新完了），就在 Solution Explorer 視窗中右鍵專案，並選擇 **Retarget Project**，並選擇 Win 10 的 SDK
- Rebuild Solution，理論上就沒問題了
- 最後就會發現專案根目錄（```portaudio\build\msvc```）底下新增一個 Win32 的資料夾，點到最裡面就會找到 **portaudio_x86.lib 與 portaudio_x86.dll**，即是我們會用到的檔案
  - 補充：dll 檔是動態擴充元件，是 lib 檔在執行時會調用的項目，詳細可以參閱系統程式 (System Software) 的資料
![](/images/Speech-Compression-intro/result.png)

官方教學文件
---
接下來可以參閱官方提供的教學文件（下方[參考資料第2點](#參考資料)），裡面也有上面的步驟，不過因有課程需求，所以此篇有改過一些設定，因此可以看看官方是如何帶過下載與編譯的部分！

參考資料
---
1. 上課用的資料：http://ipv6.ncnu.org/Course/RTSP/portaudio.html
2. 官方教學文件：[http://www.portaudio.com/docs/v19-doxydocs/tutorial_start.html](http://www.portaudio.com/docs/v19-doxydocs/tutorial_start.html)
3. Windows audio drivers 解釋：https://www.sweetwater.com/sweetcare/articles/roland-difference-between-asio-wdm-mme-drivers/