# Project 3: Branch Prediction
## 主要目標:
此程式編寫目的為實現2-Bit branch prediction，該功能為預期當前branch是否taken，利用過去歷程記錄(BHT)來輔助達成預測功能。

## 程式運行方式:

**該程式執行軟體為Visual 2019。**  
請將程式碼複製至上述軟體中，再編譯執行。  

## 簡要使用說明:  
- 如需改變BHT之entry個數，請至程式碼第10行做更改(當前預設為8 entry)。  
- Input: 使用鍵盤輸入一段組合語言，輸入完畢後請按下2次enter鍵，即可執行；其中，每段指令皆須做換行分隔 (可執行之基本指令包含: add, addi, sub, label, and B-type)   
> 輸入範例  
![avatar](https://upload.cc/i1/2020/05/20/PCYsLK.jpg)  

- Output: 螢幕顯示該段組合語言運行時，各branch當下之prediction及實際發生狀況和該branch詳細之資訊(屬於何entry、misprediction次數、當前branch之指令)  
> 輸出範例  
![avatar](https://upload.cc/i1/2020/05/20/T9D8NQ.jpg)  

## 程式結構說明:  

### 基本原理:  
2-bit prediction是每當遇到branch時就去尋找該對應的entry的BHT，  
其中hisory是由前2次該entry實作branch的結果來決定的(T為1，NT為0)，藉由history去尋找對應的prediction。  

![avatar](https://upload.cc/i1/2020/05/20/TMurV3.gif)  
### 引用函式庫說明:  
```cpp  
#include<iostream> //負責輸出/輸入  
```  
```cpp  
#include<string>  //負責字串處理  
```  
```cpp  
#include<vector> //container  
```  
```cpp  
#include <cstdlib> //包含atoi函式(string 轉int)  
```  



  