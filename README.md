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
### Global變數說明:  
```cpp  
vector<string> inputall, label;  
//inputall: 用以存放每行輸入之指令資訊，待後續處理  
//label: 存放label名稱，以利後續branch發生時尋找目標位置  
```  
```cpp  
vector<int>lbnum; //存各label之下行指令的位置(程式內之行數)  
```  
```cpp  
int entry = 8; // 將entry預設為8   
```  
```cpp  
int reg[31]; //存放registor內的值  
```  
```cpp  
string** bht; //branch history table  
```  
```cpp  
int current = 0;//當前pc  
```   
```cpp  
int* miss; // misprediction次數  
```  

### 詳細程式碼說明 

```cpp  
int main() {  
	reg[0] = 0;  
	string input; 	 
	int p, pc;  
	getline(cin, input);  
	int cou = 0;  
	bht = new string*[entry];  
		for (int i = 0; i < entry; ++i)   
		bht[i] = new string[5];  
	for (int i = 0; i < entry; ++i){//初始  
		bht[i][0] ="00";   
		for (int j = 1; j < 5; ++j)  
			bht[i][j] = "SN";  
	}  
	miss = new int[entry]();  
```  
> reg[0]為R0不可更改  
> input取各行 cin的指令  
> p為一暫存值，為找尋 string中某字元的位置  
> pc紀錄當前的執行位置  
> cou紀錄當前的讀取行數  
> bht 各 entry有5格並做初始  
> miss存各 entry預測累計錯誤次數  
```cpp  
	bool push = true;  
	while (input[0] != cin.eof()) {  
		push = true;  
		p = input.find("//", 0); //解決註解  
		if (p != input.npos) {   
			input = input.substr(0, p);  
			if (p == 0) {  
				cou--;  
				push = false;  
			}  
		}  
		p = input.find(':', 0); //處理label  
		if (p != input.npos) {  
			label.push_back(input.substr(0, p));  
			lbnum.push_back(cou);  
			if (p + 1 < input.length()) {//若label與inst未分行  
				input = input.substr(p + 1, input.length());   
				if (input[0] == ' ')  
					input = input.substr(1, input.length());  
			}  
			else {  
				push = false;  
				cou--;  
			}  
		}  
		if(push)  
			inputall.push_back(input);  
		getline(cin, input);  
		cou++;  
	}  
```  
> 處理每行輸入之字串，首先屏除註解，留下指令。   
> 尋找 label，處理如 label後續接指令的狀況，並將 label及其下一指令存入陣列中  
> 如非上述狀況，就將該行指令存入inputall，待後續作處理  



  