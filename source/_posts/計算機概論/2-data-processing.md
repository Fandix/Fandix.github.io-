---
title: 計算機概論 - 資料處理 Data manipulation
date: 2022-02-17 17:24:40
tags:
- 計算機概論

categories:
- CS
---

本章是對計算機概論的`資料處理`做一個整理與筆記，主要是介紹電腦是如何處理資料並且與週邊裝置進行溝通，會探索一些計算機架構的基本原理，也會使用機械語言指令進行一些程式設計。

<img src="https://images.idgesg.net/images/article/2018/03/blue-mother-board_circuitry_computer-chip_processor_harddrive-100751586-large.jpg?auto=webp&quality=85,70" alt="drawing" width="860"/>

<!-- more -->

# 計算機架構
電腦控制茲了的電路稱為`中央處理單元 (Central processing unit)` 或 `CPU (處理器)`。

## CPU 的基本原理
CPU 包含三個主要部份
1. **算數 / 邏輯元件 (arithmetic/logic unit)**: 負責處理資料運算的電路，比如加減法的運算。
2. **控制元件 (control unit)**: 負責協調機器內部運作的電路。
3. **暫存器元件 (register unit)**: 負責儲存資料的地方，類似於主記憶體的儲存單元，稱為 `Registers` 主要是將資料暫存在 CPU 中。

為了傳送位元字串，CPU 與主記憶體之間由一條稱為`匯流排`的線路連接。

如果要將儲存在記憶體的數值進行加總，好型以下的步驟：
1. 將要加總的數值從記憶體中取出並存入暫存器中
2. 把要加總的另一個數也從記憶體中取出存放在另一個暫存器
3. 啟動`加法電路`將剛取出的兩個值進行加總，將輸出結果存放在另一個暫存器中
4. 將結果存回記憶體

> 快取記憶體 (Cache memory): 讓資料傳輸變成是在暫存器與快取記憶體之間，任何資料的變動都先存放在快取記憶體中，等待適當的時機再一起存回主記憶體，可以讓 CPU 更快速地執行計算，避免與主記憶體進行溝通而造成時間上的延遲。 

# 機械語言
能夠對 CPU 進行操作的指令以及編碼系統稱為機械語言 (machine language)，機械指令可以分為三類：
## 資料傳輸
要求資料從一個地方移動到另一個地方的指令，資料在移動的時候很少會刪除原有位置上的資料，傳輸指令比較像是`複製資料`而非移動資料。

- 以某記憶體儲存單位元內的資料移動到某個暫存器的指令稱為 `LOAD`
- 以某暫存器內資料移動到某記憶體儲存單位元的指令稱為 `STORE`

資料傳輸指令中有一組很重要的指令群，用來與 CPU 和主記憶體之外的裝置進行通訊命令，比如鍵盤, 螢幕...，這些稱為`輸入輸出指令 (I/O instructions)`。
## 運算邏輯
顧名思義負責執行各種運算，包含`基本運算`、`布林運算 (AND, OR, XOR...)`、`資料位移運算 (SHIFT, ROTATE...)`
## 控制
包含一些引導程式執行的指令，比如 `JUMP`, `BRANCH` 是用來引導 CPU 去執行指令清單的任何一個指令，

# 程式執行
除存在記憶體中的程式會複製到所需要的指令到 CPU 中，一但到了 CPU 就會將每個指令進行`解碼`並確實執行，除非有 `JUMP` 這個指令，否則都是按照儲存在記憶體中的順序執行。

CPU 中有兩個專用的暫存器: `指令暫存器 (instrucvtion register)` 和 `程式計數器 (program counter)`，指令暫存器用於儲存正在執行的指令，程式計數器儲存下一個指令的位置。

而 CPU 會不斷重複的執行三個步驟，這三個步驟就是所謂的`機械週期 (machine cycle)`，這三個步驟分別為`提取`, `解碼`, `執行`。
<img src="https://ithelp.ithome.com.tw/upload/images/20220218/20124767HkSZwb5Hkr.png" alt="drawing" width="800"/>

> 電腦時脈 (clock): 稱為振盪器電路，會產生`脈衝`以協調電腦內部的運作，脈衝越快電腦執行一個機械週期的時間也越快。

- **提取**: 會讀取兩個記憶體的內容，CPU 將收到的指令存入指令暫存器中，接著把程式計數器增加為 2，使得程式計數器保有下一個指令在記憶體中的位置。
- **解碼**: 指令存入指令暫存器中後，CPU 就會開始解碼
- **執行**: CPU 會啟動適當的電路來執行解碼後的指令

# 算數 / 邏輯指令
邏輯運算是有 `AND`, `OR`, `XOR` 他們有各自不同的功用。

主要的用途在能夠將某個位元字串放入 0 而不影響另一個位元字串，這種 AND 運算的用途就是`遮罩運算 (masking)`，遮罩運算的其中一組運算元稱為`遮罩(mask)`，用於決定另一組運算元中的那些位元會對結果有影響。

`AND` 運算也可以用來複製位元字串，其中非複製的部分為 `0`，除了 AND 之外 `OR` 也可以用來產生一個位元字串的複製，只是非複製的部分要放置 `1`，`XOR` 運算主要用於產生任何位元字串的`補數`，任何位元組與`全為 1`的遮罩進行 XOR 就能產生該位元組的補數

```
     11111111
XOR  10101010
--------------
     01010101
```

所以 XOR 可以用來反轉 RGB 點陣圖圖像的所有位元，產生一個顏色反轉的彩色圖像，明亮的地方變深色反之亦然。

## 循環與位移運算式
循環與位移的運算提供了在暫存器中移動位元的方法，常用來解決`排比 (alignment)` 問題，假設一個暫存器含有一位元組的空間，如果要將該位元組往右移動一個位元，可以想像最右端的位元從邊緣掉落，這樣最左邊就多了一個空位，那麼如何處理掉落的那個位元呢？有一種方式是將最右邊掉落的位元補在左邊空出的位置，這種方式稱`循環位移 (circular shift)` 也稱為 `循環 (rotation)`，所以如果同一個位元組往右移動 8 次，其結果就是原是的位元字串。

另一種方式是把掉落的位元捨棄，把最左邊空位補上 `0`，這種方式稱為`邏輯位移 (logical shift)`，不管哪種位移方式都要注意保留正負號位元，能夠保留著正負號位元不變的位移有時候稱為`算數位移 (arithmetic shifts)`。

## 算數運算式
雖然運算式中有`加法`, `減法`, `乘法`, `除法`，但減法可以藉由`加法`和`取補數`來模擬，此外乘法是重複的加法而除法則是重複的減法，所以有些小型的 CPU 只設計了加法或是只設計加法與減法。

# 主設備之間的通訊
主記憶體和 CPU 是電腦的核心，本章將探討電腦的核心該如何與週邊設備 (大容量儲存裝置、鍵盤、滑鼠...) 進行溝通，甚至是如何與其他電腦進行溝通

## 控制器的作用
電腦與其他裝置的通訊一般是經由一個`中介裝置`來控制，這個裝置稱為`控制器 (controller)`，控制器透過電線連接到電腦內部的裝置或透過電腦後面的`連接阜 (port)` 與電腦外部的裝置相連，控制器可將電腦與週邊設備的訊息和資料進行轉譯在來回傳遞，每個控制器與電腦的通訊都是通過匯流排連接，與 CPU 和主記憶體之間所連接的匯流排一樣。

在某些電腦的設計中，CPU 和主記憶體溝通用的 `LOAD` 和 `STORE` 指令，也可以直接用在控制器的資料傳輸與接受上面，在這種情況控制器會對應到一組`唯一的地址`，並主記憶體會刻意跳過這個地址，因次當 CPU 通過匯流排要 STORE 一個位元字串到一個指定給控制器的地址時，該位元字串會被寫到`控制器`而不是記憶體中，同樣的如果 CPU 要讀取這個位置的資料的話，會 LOAD 控制器從而接收資料而非記憶體，這種通訊方式稱為`記憶體對映輸入輸出 (memory-mapped I/O)`。

<img src="https://ithelp.ithome.com.tw/upload/images/20220218/20124767nruwCQ2Y2X.png" alt="drawing" width="800"/>


## 直接記憶體存取
由於控制器連接到電腦的匯流排上，他可以在 CPU 沒有使用到匯流排的毫微秒空擋與記憶體進行通訊，這種能存取主記憶體的能力稱為`記憶體存取 (direct memory access, DMA)`，比如要從磁碟中讀取資料，CPU 可以傳送編碼過的位元字串到磁碟連接的控制器上並要求控制器讀取磁碟中的資料在存入指定的記憶體區域，所以當控制器在處理資料的儲存時 CPU 就可以趁這個空擋執行其他工作，可以不必為了等待速度較慢的資料傳輸而浪費計算資源。

不過使用 `DMA` 也會有缺點，如果 CPU 和控制器都需要使用到匯流排，那們他們之間的競爭會使匯流排成為效能上的一個障礙，稱為`范紐曼瓶頸 (von Neumann bottleneek)`。

## 交握
當兩個設備要進行雙向溝同時，不能由其中一個設備盲目的傳送資料給另一個設備，這樣會讓資料遺失的機率大幅增加，因次兩個設備互相溝通需要有`交握 (handshaking)` 的過程，這是電腦與周圍設備交換關於設備狀況的資訊並協調彼此的運作，交握通常包含著一個`狀態字 (status word)`，是由週邊設備產生出來的位元字串並傳送給控制器，狀態字是反映出這個週邊設備的狀態。

## 通訊媒介
計算機設備通訊留兩種路徑處理：`平行`, `串列`，這兩種方法是指訊號是經由什麼分式傳輸
1. **平行傳輸 (parallel communication)**: 數個訊號會被同時傳送，每個訊號都經由`不同的線路`，這種方式可以快速的傳送資料但需要相對複雜的通訊路徑。
2. **串列通訊 ()**: 使用`單一線路`連接傳送訊號，相較於平行傳輸只需要簡單的路徑。

## 通訊速率
位元字串的傳輸速率是以`每秒位元 (bits per second, bps)` 來衡量，在英文縮寫裡小寫 b 代表`位元`大寫 B 代表`元組`。

最高傳輸速率取決於通訊路徑的類別和應用的技術，最高傳輸速率通常相當於通訊路徑的`頻寬`，雖然頻寬也有容量的定義但當提到一個通訊路徑有很高的頻寬或提供`寬頻服務`時，意味著該通訊路徑能提供高速的位元傳輸速率，同時也能負載大量的資訊。

# Reference
- [computer science - AN OVERVIEW](https://hostnezt.com/cssfiles/computerscience/Computer%20Science%20An%20Overview%2011th%20Ed%20.J%20Glenn%20Brookshear.pdf)