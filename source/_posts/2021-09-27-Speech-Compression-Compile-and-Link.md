---
title: 【Speech Compression】PortAudio 程式編譯及連結 portaudio_x86.lib
date: 2021-09-27 19:10:01
tags:
  - "語音壓縮"
  - "Speech Compression"
  - "PortAudio"
categories:
  - "Speech Compression"
  - "PortAudio"
---

經由上一篇 - [PortAudio環境設定 (for Windows using Visual Studio)](https://kalinlai-void.github.io/2021/09/13/Speech-Compression-intro/)，我們已經知道該如何產生出 portaudio_x86.lib 與 portaudio_x86.dll 這兩個檔案。

而本篇要將專案連結至 portaudio_x86.lib，並將 PortAudio 專案做為函式庫，在使用 portaudio_x86.dll，嘗試撰寫語音程式。

<!--more-->

### 電腦如何將 Source Code 編譯為可執行檔？
這是系統程式的內容，這邊簡單複習，就不贅述了。

如下圖所示，有一 Source Code 為 hello.cpp，內有匯入一些標頭檔 (Header)，會交由編譯器 (Compiler) 去編譯，而後產生出 hello.obj，再藉由連結器 (Linker)，將函式庫 (Library) *.lib 連結，最後產生出 hello.exe 。

而我們執行 hello.exe 後，會將該執行檔及會使用到的動態連結函式庫 (Dynamic Linking Library) *.dll 檔（可以想像成程式的擴充元件），一起放入載入器 (Loader) 中，最後放到記憶體內執行程式。

![](/images/Speech-Compression-Compile-and-Link/Compile-a-Source-Code-to-an-Executable-File.png)

而接下來我們要來撰寫 Portaudio 的程式了！

### Coding : PortAudio version
1. 首先，開啟 Visual Studio，創建一個 C++ 空白專案，並加入一個 C++ File (.cpp)。然後寫入以下程式：
```C++
#include <stdio.h>
#include "portaudio.h"

int main() {
    printf("%x\n", Pa_GetVersion()); // the version is represented by hexadecimal.
    printf("%s\n", Pa_GetVersionText());
    return 0;
}
```

2. 此時會發現 IDE 在檢查語法時，找不到 portaudio.h，因此我們需要在此專案加入該 Header：
   - 開啟專案設定，找到 ```C/C++ -> General -> Addiotnal Include Directories```，並將其設定為**PortAudio 專案底下 portaudio.h 之根目錄**。如下圖紅框所示：
    ![](/images/Speech-Compression-Compile-and-Link/Additional-Include-Directories.png)


3. 此時編譯 (Compiler) 出 .obj 檔後，再建置 (Build) 專案時，會發現程式無法建置，會出現以下錯誤訊息：
   - ```unresolved external symbol```
   - 這是因為 Build 專案會嘗試將所需要的 .lib 檔透過 Linker 與 剛才編譯出的 .obj 檔連結，但此時發現找不到所需的 .lib 檔。

4. 那接下來我們需要在專案設定中，加入 portaudio_x86.lib 做為外部函式庫：
   - 開啟專案設定，找到 ```Linker -> Input -> Addition Dependencies```，並將其設定為 **portaudio_x86.lib 的完整路徑**。如下圖紅框所示:
    ![](/images/Speech-Compression-Compile-and-Link/Addition-Dependencies.png)

5. 此時 Build 就會成功，並產生執行檔 .exe。但此時執行後，他會跟你說「找不到 portaudio_x86.dll」。
   - 解決辦法很簡單，就是將上次 Build 出來的 portaudio_x86.dll，複製到本專案底下的 Debug 資料夾內（**就是 .dll 與 .exe 檔需位於同一層**）。
   - 此時執行程式就可以正常 print PortAudio 的版本資訊。（130700h 即代表 19.7.0）
    ![](/images/Speech-Compression-Compile-and-Link/PA01-result.png)

### Programming with PortAudio
1. 寫一個會被 PortAudio 用來做聲音處理所需呼叫的 Callback Function。
2. 初始化 PortAudio Library 並開啟一個聲音 I/O 串流 (Stream)。
3. 開始串流。此時所撰寫的 Callback Function 將會被 PortAudio 重複呼叫執行。
4. 在你的 Callback Function 中，可以從 inputBuffer 讀取聲音資料，也可以將資料寫入 outputBuffer。
5. 透過 CallBack Function retrun 1 或呼叫停止串流的 Function 來停止串流。
6. 關閉串流並終止 PortAudio。

下圖是大略的架構圖，藍色部分是 PortAudio 幫我們做好的部分，我們需要撰寫的其實只有綠色部分。
![](/images/Speech-Compression-Compile-and-Link/Structure-of-PortAudio-Porgramming.png)

而 PortAudio 專案內其實有蠻多範例程式碼可以看看的，都在 examples 與 test 資料夾內。接下來我們將以 paex_saw.c 做為範例。

### Example : Play a sawtooth wave
官方範例程式碼：[from Github](https://github.com/PortAudio/portaudio/blob/master/examples/paex_saw.c) （其實就是 PortAudio 專案內 examples 資料夾中的 paex_saw.c）

> 此程式碼會播放一段鋸齒波 (sawtooth wave) 的聲音，但聲音可能不太悅耳，因此建議將音量調低。

官方教學文件的範例也是拿這個程式來講解，可參閱下方的[參考資料](#參考資料)

#### 此程式可以分成兩個區塊
##### Writing a Callback Function（[官方文件](http://www.portaudio.com/docs/v19-doxydocs/writing_a_callback.html)）
```C++
typedef struct
{
    float left_phase;
    float right_phase;
} paTestData;

static int patestCallback(  const void* inputBuffer, void* outputBuffer,
                            unsigned long framesPerBuffer,
                            const PaStreamCallbackTimeInfo * timeInfo,
                            PaStreamCallbackFlags statusFlags,
                            void* userData  )
{
    /* Cast data passed through stream to our structure. */
    paTestData *data = (paTestData*)userData;
    float* out = (float*)outputBuffer;
    unsigned int i;
    (void)inputBuffer; /* Prevent unused variable warning. */
        
    for (i = 0; i < framesPerBuffer; i++)
    {
        *out++ = data->left_phase;  /* left */
        *out++ = data->right_phase;  /* right */
        /* Generate simple sawtooth phaser that ranges between -1.0 and 1.0. */
        data->left_phase += 0.01f;
        /* When signal reaches top, drop back down. */
        if (data->left_phase >= 1.0f) data->left_phase -= 2.0f;
        /* higher pitch so we can distinguish left and right. */
        data->right_phase += 0.03f;
        if (data->right_phase >= 1.0f) data->right_phase -= 2.0f;
    }
    return 0;
}
```
> 此 function 會在左右聲道各播放一個鋸齒波。
 
![](/images/Speech-Compression-Compile-and-Link/Sawtooth-wave.png)
##### Main Function
- Initialize PortAudio （[官方文件](http://www.portaudio.com/docs/v19-doxydocs/initializing_portaudio.html)）
在使用 PortAudio 前，**務必**呼叫 Pa_Initialize() 來做初始化。
```C++
/* Initialize library before making any other calls. */
err = Pa_Initialize();
if (err != paNoError) goto error;
```

- Open a stream （[官方文件](http://www.portaudio.com/docs/v19-doxydocs/open_default_stream.html)）
```C++
#define SAMPLE_RATE (44100) // 取樣頻率，44.1k 是 CD 所使用的頻率
static paTestData data;
...
PaStream *stream;

/* Open an audio I/O stream. */
err = Pa_OpenDefaultStream( &stream,
                            0,              /* no input channels */
                            2,              /* stereo output */
                            paFloat32,      /* 32 bit floating point output */
                            SAMPLE_RATE,
                            256,            /* frames per buffer */
                            patestCallback, /* this is your callback function */
                            &data           /*This is a pointer that will be passed to your callback*/
                        );
    
if (err != paNoError) goto error;
```
> Q: 什麼是「Stream（串流）」？
> A: 就只是一個 buffer。
> 它的內容會藉由 Callback Function 自動填滿，且會在 Stream 開始後播放。

- Start, Stop, and Abort a stream （[官方文件](http://www.portaudio.com/docs/v19-doxydocs/start_stop_abort.html)）
  - 開啟串流
  ```C++
  err = Pa_StartStream(stream);
  if (err != paNoError) goto error;
  ```
  > 當呼叫 Pa_StartStream() 後，PortAudio 將會開始呼叫我們寫的 Callback Function (如上方的 patestCallback() )，直到我們停止串流為止
 
  - 關閉串流
  ```C++
  err = Pa_StopStream(stream);
  if (err != paNoError) goto error;
  ```
  > Pa_StopStream() 主要是確保在 Callback Function 中處理的 buffer 都已被播放，這可能會導致延遲。或者，可使用 Pa_AbortStream() 取代，在某些平台上可能會比較快（但可能不會播放完整個 buffer）。

- Closing a stream and terminating PortAudio
  - 當我們完成一個串流後，我們應該要將它關掉並釋放電腦資源
  ```C++
  err = Pa_CloseStream(stream);
  if (err != paNoError) goto error;
  ``` 
  - 而最後很重要的一點，整個程式完成後，**務必**終止 PortAudio
  ```C++
  err = Pa_Terminate( );
  if( err != paNoError ) goto error;
  ``` 

- 顯示錯誤訊息
以上程式碼片段當 err 接收到錯誤碼時，會 goto error，而以下是 error 的程式碼片段：
```C++
error:
    Pa_Terminate(); // 終止 PortAudio
    fprintf(stderr, "An error occured while using the portaudio stream\n");
    fprintf(stderr, "Error number: %d\n", err); // 印出錯誤碼
    fprintf(stderr, "Error message: %s\n", Pa_GetErrorText(err)); // 印出錯誤訊息
    return err;
``` 
> P.S. stderr 是 C/C++ 中的標準錯誤訊息 (standard error message)，代表訊息將於畫面上或 Windows terminal 輸出

### 參考資料
1. 上課用的資料：[http://ipv6.ncnu.org/Course/RTSP/week02-link.html](http://ipv6.ncnu.org/Course/RTSP/week02-link.html)
2. 官方教學文件：[http://www.portaudio.com/docs/v19-doxydocs/tutorial_start.html](http://www.portaudio.com/docs/v19-doxydocs/tutorial_start.html)