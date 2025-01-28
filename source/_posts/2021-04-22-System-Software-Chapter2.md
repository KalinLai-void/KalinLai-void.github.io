---
title: 【System Software】 Chapter 2 - Assembler
author: Karlun Lai
tags:
  - 'System Software'
  - '系統程式'
  - 'SIC/XE'
categories:
  - 'System Software'
mathjax: true
date: 2021-04-22 06:21:14
---
### Role of Assembler
![](/images/System-Software-Chapter2/assembler.png)

<!--more-->

#### Basic SIC assembler directive
指示(directive)與指令(instruction)不同的是，**「指令」為讓CPU可以實際執行的命令（e.g. LDA, ADD...）；而「指示」並非真正的指令，主要是為了告訴Assembler要做什麼事（e.g. 保留空間、程式開始與結束...）**
- START, END 程式開始與結束
  - e.g. START 1000 => 代表此程式從記憶體位址0x1000開始（如下圖）
- WORD 宣告一個Word的標籤做為**常數**使用
- BYTE 宣告數個Byte的標籤做為**常數**使用
- RESW 保留一個Word的空間當作**變數**使用
- RESB 保留數個Byte的空間當作**變數**使用
- p.s. 常數：不可修改；變數：可修改（後來再賦予值）

#### SIC Code Example (含Object Code)
![](/images/System-Software-Chapter2/SIC_Example.png)

#### Fundamental functions of assembler
1. 將 助憶碼(mnemonic operation codes) 轉換成機器碼 
   - e.g. LDA => 0x00
2. 將 符號運算元(symbolic operands) 轉換為對應的記憶體位址 
   - e.g. RDREC => 0x2039
3. 依照指令格式建立指令
4. 將常數資料轉換成機器內部的表示方式
   - e.g. ```EOF BYTE C'EOF'``` => 454F46
5. 產生目的碼程式 (Object program) 和組譯列表

#### obj檔內容（皆以16進制表示）
下圖為上面範例放入obj檔的樣子
![](/images/System-Software-Chapter2/SIC_Example_OBJ.png)
其中：
- record (每行首個字元)
  - H (Header)：Obj檔第一行
    - 該行第2~7個字元：程式名稱
    - 該行第8~13個字元：程式起始位址
    - 該行第14~19個字元：程式總共使用的記憶體長度
      - 以上圖為例的話，2079-1000+1=107A
  - T (Text)：指令碼
    - 該行第2~7個字元：該行的起始位址
    - 該行第8~9個字元：該行長度
    - 該行10~69個字元：Object Code，以16進制表示
    - OBJ檔中，Object Code每Byte為2字元，因此**每行最多只能放30 Bytes**
      - (69-10+1)/2 = 30
      - 指令普遍為格式3，指令碼為6個字元，則<u>普遍情況下，該行最多只能存放(69-10+1)/6=10個指令</u>
  - E (End)：Obj檔最後一行
    - 該行第2~7個字元：程式中第一個可執行指令的位址

#### Address Translation Problem
##### Forward reference (前向參考問題)
- 組譯時會遇到的問題
- 參考的標籤位於該行後方（尚未定義）

##### Solution : Two passes assembler
- Pass 1
  - 分配記憶體位址給程式中每一行指令
  - 判斷指令長度(透過OPTAB)
  - 計算LOCCTR
  - **儲存所有標籤(Labels)的位址到SYMTAB**
  - 對一些指示要進行處理
- Pass 2
  - 透過SYMTAB，帶入Pass 1沒有處理的symbol
  - 透過OPTABLE產生機器碼
  - 藉由BYTE, WORD定義，生成常數標籤的數值
  - 寫入obj檔和組譯列表

###### 細節
![](/images/System-Software-Chapter2/two-pass.png)
- 運算碼表 (Operation Code Table, OPTAB)
  - **儲存助憶指令與機器碼之關連**
  - 使用Hash Table，使用助憶碼作為key
  - Pass 1時用來檢驗運算碼及計算指令長度
  - Pass 2時用來轉換運算碼至機器碼
  - Static結構
- 符號表 (Symbol Table, SYMTAB)
  - **儲存Labels與其對應記憶體位址**
  - 包含Labels名稱、位址、長度、型態和error flags
  - 使用Hash Table加速
  - Dynamic結構
- 程式計數器 (Location Counter, LOCCTR)
  - 初始值為START指派的值
  - 記錄下一個指令的位址
- Intermediate file
  - 由pass1寫入並當pass2的input
  - 包含：
    - 在Pass 1所得到的記憶體位址
    - 錯誤指示（避免Pass 1發現錯，卻還進入Pass 2）

#### 一些SIC/XE指令符號代表的定址模式
- 定址模式請參考[這裡](https://karlunlai-void.github.io/2021/04/22/System-Software-Chapter1/#Addressing-Modes-1)
- +: Extended Addressing Mode
  - e.g. ```+JSUB```
- #: Immediate Addressing Mode
  - e.g. ```LDA #3```
- @: Indirect Addressing Mode
  - e.g. ```J @RETADR```
- ,X: Indexing Addressing Mode
  - e.g. ```LDA ARRAY, X```
- 如沒以上符號
  - **原則上，組譯時先嘗試使用PC相對定址**
  - 若超出範圍則試著使用Base定址
  - 若兩者都不適用則會有錯誤訊息

#### Program Relocation (程式重新定址問題)
- SIC有一個absolute loader (絕對載入器)
  - 程式必須被載入到指定的起始位址
  - **如果被載入到與指定的不同，則指令的位址需要更改**
- 因OS記憶體配置較彈性，因此程式每次執行可能會被放在不同的起始位址
![](/images/System-Software-Chapter2/relocation.png)

##### Relocatable program
- Assembler無法自己改變記憶體位址
  - 但可透過OBJ檔告訴loader記憶體位址需要被修正。
- 會受影響的指令碼（需被Relocate的指令）
  - SIC版本
    - **常數資料的指令碼絕對不會影響**
      - e.g. ```EOF BYTE C'EOF'``` => 454F46
    - 大多指令都是直接定址，因此幾乎都會影響
    - 所以暫時逃避不談xD
  - SIC/XE版本
    - **只有Extended定址會受影響**，因為其他定址模式都是相對的
  - 簡單來說，**相對定址和常數資料的指令碼不受影響**
- OBJ檔中的修正方式：
  - 使用record
    - M (Modification)：修正會放在E record前
      - **該行第2~7個字元：需修改的位址欄位與程式起始位址的距離**
      - 該行第8~9個字元：修正長度（以Nibble為單位，1 Nibble = 0.5 Byte）
    - e.g. 有一XE的obj檔如下
    ![](/images/System-Software-Chapter2/XE_Example_OBJ.png)
      - 標示如上圖所示，每兩個16進制字元為1 Byte
        - 因此藍標，M 000007 05
          - 去Relocation相對程式起始位址第7個Byte
          - 並修正5個Nibbles（2.5個Bytes）
          - 第7個Byte是10，那為何是從0開始取5 Nibbles，而不是從1開始取？
            - 原因：**因為存放在記憶體的方式是littleEndian**，所以1個Nibble是從第7個Byte的後面開始算
        - 其他以此類推

### Sysmbol-Defining Statements
#### Literals