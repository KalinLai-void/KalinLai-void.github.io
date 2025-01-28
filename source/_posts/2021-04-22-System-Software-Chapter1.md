---
title: 【System Software】 Chapter 1 - Background
author: Karlun Lai
tags:
  - 'System Software'
  - '系統程式'
  - 'SIC/XE'
categories:
  - 'System Software'
mathjax: true
date: 2021-04-22 00:01:17
---

## Simplified Instruction Computer 簡化指令電腦(SIC)
- SIC是一個**假想的電腦**，用來解釋系統程式的架構
- SIC有兩種版本
  - Standard model (**SIC**)
  - Extension version (SIC/**XE**)
    - 相比SIC，更多設備指令
    - 但更貴
    - 支持**向上相容 (upward compatible)** ，代表XE電腦可以執行SIC指令，但SIC電腦無法執行XE指令

<!--more-->

### SIC Machine Architecture
#### Memory
- 1 Bytes = 8 bits
- **1 Word = 3 Bytes (24 bits)**
- Total memory is **32768 ($2^{15}$) bytes (32KB)**

#### Registers
| Mnemonic | Number | Special use|
|:-:|:-:|:-|
| A | 0 | 累加(Accumulator) 暫存器 <br>用在算術運算 |
| X | 1 | 索引(Index) 暫存器 <br>用在定址 |
| L | 2 | 連結(Linkage) 暫存器 <br>用於儲存跳到副程式指令(Jump to Subroutine, JSUB)返回的位址 |
| PC | 8 | 程式計數器(Program counter) <br>儲存下一個要取出來執行的指令的位址 |
| SW | 9 | 狀態字組(Status word) <br>儲存不同的資訊，包括條件碼(Condition Code, CC)，主要是機器使用 |

#### Data Formats
- 整數以24 bits(1 word)的二進制碼儲存
- 用2的補數表示負數值
- 字元以8 bits的ASCII Code儲存
- 無浮點數表示法

#### Instruction Formats
![](/images/System-Software-Chapter1/SIC_Instruction_Formats.png)
可拆分為
- Opcode(指令碼)：8 bits，剩下的位元數(24-15-1=8)
- X：1 bit，用來表示是否為索引定址模式(indexed-addressing mode)
- Address：15 bits，因為記憶體大小為$2^{15}$ bytes，因此15個bits就可以表示所有記憶體位置

#### Addressing Modes
| Mode | Indication | Target address calculation |
| - | - | - |
| Direct | x = 0 | TA = address |
| Indexed | x = 1 | TA = address + (X) |

p.s. (X) = X暫存器內的值

#### Instruction Set
- Load & Store
  - LDA, LDX 將一個記憶體的資料（必為1 word）載入A暫存器 or X暫存器
  - STA, STX 將A暫存器 or X暫存器儲存至一個記憶體
  - LDCH 載入1 byte的記憶體資料至A暫存器（**剩餘位數補0**）
  - STCH 將A暫存器的值儲存至一個記憶體（1 bytey資料）
- Integer Arithmetic Operations
  - ADD, SUB, MUL, DIV, etc.
    - **將A暫存器與某記憶體中的資料（必為1 word）運算，並儲存在A暫存器**
    - e.g. ADD 0030 → A暫存器 = (A) + (0030)
- Comparison
  - COMP 比較A暫存器內的值與一個記憶體位址中的資料，並**設置一個CC再儲存至SW暫存器**
- Coditional Jump Instruction
  - JLT, JEQ, JGT (<, =, >)
  - 根據SW暫存器的值，去決定是否跳到該記憶體位址，因此需與COMP指令搭配
  - e.g. 
    ```
    COMP 0030   // (A)與(0030)比較
    JLT 3120    // 若(A)<(0030)，則將PC暫存器的值設為3120
    ```
- Subroutine Jump Instructions
  - JSUB 跳到某副程式
  - RSUB 無需放任何參數在後方，跳到L暫存器內的值
  - **使用L暫存器來存取回傳的位址**
- Input & Output & Test Device
  - 輸入輸出
    - 一次只處理1 byte，並且只存取A暫存器的最右邊8 bits
    - 每個裝置都有特定的記憶體位址，**如果要從該裝置輸出or輸入，就是去該位址做存取**
    - RD (Read Device), WD (Write Data)
    - e.g. [Input & Output](#Input-amp-Output)
  - TD (Test Device)
- 其他指令
  - 搭配索引定指模式(Indexed-addressing mode)
    - 如下方Code => [Looping and Indexing](#Looping-amp-Indexing)
  - TIX
    - (X) = (X) + 1
    - 加完後X暫存器會與後方參數比較，若相等就會設置CC給SW暫存器
    - 常用在迴圈處理，如下方Code => [Looping and Indexing](#Looping-amp-Indexing)
- **沒有記憶體對記憶體的搬移指令**
  - 不過可以經過Coding，如下方Code => [Load & Store](#Load-amp-Store)

### SIC/XE Machine Architecture
#### Memory
- Total memory is **$2^{20}$ bytes (1MB)**

#### Registers
除了原本SIC有的暫存器外，還多了以下暫存器：

| Mnemonic | Number | Special use|
|:-:|:-:|:-|
| B | 3 | 基底(Base) 暫存器 <br>用在定址 |
| S | 4 | 一般工作(General working) 暫存器 <br>無特定用途 |
| T | 5 | 一般工作(General working) 暫存器 <br>無特定用途 |
| F | 6 | 浮點累加器(Floating-point accumulator)，有48 bits <br>用於浮點運算 |

p.s. 本課程不討論浮點數處理，因此該筆記不整理浮點數相關

#### Instruction Formats
基本上，沒有特定格式狀況下，都是Format 3
![](/images/System-Software-Chapter1/XE_Instruction_Formats.png)

#### Addressing Modes
Instruction Format 3 & Format 4中間那6 bits (n i x b p e)，分別代表一種定址模式，而**b、p兩個位元會互斥（b=1則p=0，反之p=1則b=0）**
- Simple
  - SIC: n = i = 0，其實就是SIC的instruction format
    - **bpe會變成disp的一部分**，TA = bpe + disp
  - XE: n = i = 1
    - Base relative 基底定址
      - 相對於Base暫存器
      - **n=1, i=1, b=1, p=0**
      - TA = (B) + disp ($0 \leq disp \leq 4095$)
      - ![](/images/System-Software-Chapter1/Base_relative_addressing_mode.png)
    - Program-counter relative PC相對定址
      - 相對於PC暫存器
      - **n=1, i=1, b=0, p=1**
      - TA = (PC) + disp ($-2048 \leq disp \leq 2047$)
      - ![](/images/System-Software-Chapter1/PC_relative_addressing_mode.png)
    - Direct 直接定址
      - **n=1, i=1, b=0, p=0**
      - TA = disp ($0 \leq disp \leq 4095$)
      - ![](/images/System-Software-Chapter1/Direct_addressing_mode.png)
- Immediate 立即定址
  - 要運算的東西，直接放在disp
  - **n=0, i=1, x=0**
  - operand = disp
  - ![](/images/System-Software-Chapter1/Immediate_addressing_mode.png)
- Indirect 間接定址
  - **n=1, i=0, x=0**
  - TA = (disp)
  - ![](/images/System-Software-Chapter1/Indirect_addressing_mode.png)
- Indexing 索引定址
  - **x=1**，可搭配其他定址模式
  - TA = (X) + Original TA
  - ![](/images/System-Software-Chapter1/Indexing_addressing_mode.png)
- Extended 
  - <font style="color:red">Format 4</font>
  - **e=1**

##### Addressing Mode Example
![](/images/System-Software-Chapter1/Addressing_Mode_Example.png)

| Hex | Opcode | n | i | x | b | p | e | disp/address | TA |
|-|-|-|-|-|-|-|-|-|-|
| 032600 | 000000 | 1 | 1 | 0 | 0 | 1 | 0 | 0110 0000 0000 | 3600 |
| 03C300 | 000000 | 1 | 1 | 1 | 1 | 0 | 0 | 0011 0000 0000 | 6390 |
| 022030 | 000000 | 1 | 0 | 0 | 0 | 1 | 0 | 0000 0011 0000 | 3030 |
| 010030 | 000000 | 0 | 1 | 0 | 0 | 0 | 0 | 0000 0011 0000 | 30 |
| 003600 | 000000 | 0 | 0 | 0 | 0 | 1 | 1 | 0110 0000 0000 | 3600 |
| 0310C303 | 000000 | 1 | 1 | 0 | 0 | 0 | 1 | 0000 1100 0011 0000 0011 | C303 |

#### Instruction Set
- load/store
  - LDB, STB, stc.
- Register move (<font style="color:red">Format 2</font>)
  - RMO
- Register-Register arithmetic (<font style="color:red">Format 2</font>)
  - ADDR, SUBR, MULR, DIVR

### Code Example
#### Load & Store
- SIC
  ```
  LDA   FIVE        // 將FIVE載入至A暫存器
  STA   ALPHA       // 將A暫存器的值存至ALPHA
  LDCH  CHARZ       // 將CHARZ載入A暫存器，此時(A)=0x000091
  STCH  C1          // 將A暫存器的值存至C1
    .
    .
    .
  /** 以下為「指示(Directive)」，詳細至Chapter 2了解 **/
  ALPHA RESW 1      // 宣告ALPHA，並保留1 word大小的空間
  FIVE  WORD 5      // 宣告FIVE，並將常數5放入(1 word大小)
  CHARZ BYTE C'Z'   // 宣告CHARZ，並放入「Z」的ASCII Code
                    // C''代表放入ASCII Code
  C1    RESB 1      // 宣告C1，並保留1 byte大小的空間
  ```
- XE
  ```
  LDA   #5          // 「#」為立即值(使用Immediate Addressing Mode)
  STA   ALPHA
  LDA   #90         // 使用立即定址將ASCII Code為90的字元存進A暫存器
  STCH  C1
    .
    .
    .
  ALPHA RESW  1
  C1    RESB  1
  ```

#### Arithmetic Operation
- 以下Code先假設ALPHA, BETA, GAMMA, DELTA, INCR已載入值
- SIC
  ```
  LDA   ALPHA     // 將ALPHA的值存入A暫存器
  ADD   INCR      // A暫存器 += INCR
  SUB   ONE       // A暫存器 -= 1
  STA   BETA      // 將A暫存器的值存入BETA
  LDA   GAMMA     // 將GAMMA的值存入A暫存器
  ADD   INCR      // A暫存器 += INCR
  SUB   ONE       // A暫存器 -= 1
  STA   DELTA     // 將A暫存器的值存入DELTA
    .
    .
    .
  ONE     WORD 	1
  ALPHA   RESW	1
  BETA    RESW	1
  GAMMA   RESW 	1
  DELTA   RESW	1
  INCR    RESW 	1
  ```
- XE
  ```
  LDS   INCR      // 將INCR的值存入S
  LDA   ALPHA     // 將ALPHA的值存入A
  ADDR  S, A      // S + A 後放回A
  SUB   #1        // A暫存器 -= 1
  STA   BETA      // 將A暫存器的值存入BETA
  LDA   GAMMA     // 將GAMMA的值存入A
  ADDR  S, A      // S + A 後放回A
  SUB   #1        // A暫存器 -= 1
  STA   DELTA     // 將A暫存器的值存入DELTA
    .
    .
    .
  ALPHA   RESW  1
  BETA    RESW  1
  GAMMA   RESW  1
  DELTA   RESW  1
  INCR    RESW  1
  ```

#### Looping & Indexing
- SIC
  ```
          LDX   ZERO      // 初始化X暫存器為0
  MOVECH  LDCH  STR1, X   // 從A暫存器讀取index為X的一個字元進去
          STCH  STR2, X   // 將A暫存器的值存入STR2的index X
          TIX   ELEVEN    // 先將X暫存器值+1後跟ELEVEN比對值放入CC（SW暫存器）
          JLT   MOVECH    // 如果CC是小於則跳回MOVECH
    .
    .
    .
  STR1    BYTE  C'TEST STRING'	//將後面字串傳成ASCII Code並丟入STR1
  STR2    RESB  11
  ZERO    WORD  0
  ELEVEN  WORD  11
  ```
- XE
  ```
          LDT   #11       // 初始化T暫存器為11
          LDX   #0        // 初始化X暫存器為0
  MOVECH  LDCH  STR1,X    // 從A暫存器讀取index為X的一個字元進去
          STCH  STR2,X    // 將A暫存器的值存入STR2的index X
          TIXR  T         // 先將X暫存器值+1後跟ELEVEN比對值放入CC（SW暫存器）
          JLT   MOVECH    // 如果CC是小於則跳回MOVECH
    .
    .
    .
  STR1  BYTE  C'TEST STRING'
  STR2  RESB  11
  ```

#### Input & Output
- SIC
  ```
  INLOOP  TD    INDEV   // 測試輸入裝置
          JEQ   INLOOP  // 迴圈直到輸入裝置準備好
          RD    INDEV   // 讀一個字元進入A暫存器
          STCH  DATA    // 把A暫存器的字元存入DATA
    .
    .
  OUTLP   TD    OUTDEV  // 測試輸入裝置
          JEQ   OUTLP   // 迴圈直到輸入裝置準備好
          LDCH  DATA    // 從DATA位址讀一個字元進A暫存器
          WD    OUTDEV  // 將字元輸出
    .
    .
  INDEV   BYTE  X'F1'   // 輸入裝置代號F1
  OUTDEV  BYTE  X'05'   // 輸出裝置代號05
                        // 由X''可知為16進制
  DATA    RESB  1
  ```