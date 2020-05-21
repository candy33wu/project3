# Project 3: Branch Prediction
## 主要目標:
此程式編寫目的為實現2-Bit branch prediction，該功能為預期當前branch是否taken，利用過去歷程記錄(BHT)來輔助達成預測功能。

## 程式運行方式:

**該程式執行軟體為Visual 2019。**  
請將程式碼複製至上述軟體中，再編譯執行。  

## 簡要使用說明:  
- 如需改變BHT之entry個數，請至程式碼第10行做更改(當前預設為8 entry)。  
- Input: 使用鍵盤輸入一段組合語言，輸入完畢後請按下2次enter鍵，即可執行；其中，每段指令皆須做換行分隔 (可執行之基本指令包含: add, addi, sub, label, li, and B-type)   
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
> 處理每行輸入之字串，首先偋除註解，留下指令。   
> 尋找 label，處理如 label後續接指令的狀況，並將 label及其下一指令存入陣列中。     
> 如非上述狀況，就將該行指令存入inputall，待後續作處理。      
```cpp  
while(current< inputall.size()){//執行
		if (inputall[current].find("ui", 0) != inputall[current].npos) {
			continue;
		}
		else if (inputall[current][0] == 'l' || inputall[current].find("i", 0) != inputall[current].npos || inputall[current].find("jalr", 0) != inputall[current].npos) {
			iType(inputall[current], current);
		}
		else if (inputall[current][0] == 's' && inputall[current].find(" ", 0) == 2) {
			continue;
		}
		else if (inputall[current][0] == 'b') {
			bType(inputall[current], current);
		}
		else if (inputall[current].find("jal", 0) != inputall[current].npos) {
			continue;
		}
		else {
			rType(inputall[current], current);
		}
		current++;
	}
}
```    
> 依照各指令字元出現之規律，找到其對應 type。      
> 呼叫相對之 function做處理。      
```cpp  
void rType(string input, int pc)
{
	string fun7, fun3, op, str[4];
	int p,rs2,rs1,rd;
	p = input.find(" ", 0);
	str[0] = input.substr(0, p);
	input = input.substr(p + 1, input.length());
	for (int i = 1; i < 4; i++) {
		p = input.find(",", 0);
		str[i] = input.substr(1, p);
		input = input.substr(p + 1, input.length());
	}
	rs2 = atoi(str[3].c_str());
	rs1 = atoi(str[2].c_str());
	rd = atoi(str[1].c_str());
	if (str[0] == "sub")
		reg[rd]= reg[rs1] - reg[rs2];
	else if(str[0]=="add")
		reg[rd] = reg[rs1] + reg[rs2];
}

void iType(string input, int pc)
{
	string fun3, op, str[4];
	int p, imm, rs1, rd;
	p = input.find(" ", 0);
	str[0] = input.substr(0, p);
	input = input.substr(p + 1, input.length());
	if (str[0] == "li") {                              //li
		p = input.find(",", 0);
		str[1] = input.substr(1, p-1);
		input = input.substr(p + 1, input.length());
		str[2] = input.substr(0, input.length());
		rs1 = atoi(str[2].c_str());
		rd = atoi(str[1].c_str());
		reg[rd] = rs1;
	}
	else if(str[0] == "addi") {
		for (int i = 1; i < 3; i++) {
			p = input.find(",", 0);
			str[i] = input.substr(1, p);
			input = input.substr(p + 1, input.length());
		}
		str[3] = input.substr(0, input.length());
		imm = atoi(str[3].c_str());
		rs1 = atoi(str[2].c_str());
		rd = atoi(str[1].c_str());
		reg[rd] = reg[rs1] + imm;
	}
}
```
> 處理字串並且找到對應的 register，做出相應的指令動作。     
> 其中 I-type function 含 addi 及 li。       
> 而 R-type 含 add 及 sub 指令。     

```cpp
void bType(string input, int pc)
{
	string  fun3, op, str[4], i12, i10, i4, i11;
	int p,imm, rs2, rs1,sta;
	p = input.find(" ", 0);
	str[0] = input.substr(0, p);
	input = input.substr(p + 1, input.length());
	for (int i = 1; i < 3; i++) {
		p = input.find(",", 0);
		str[i] = input.substr(1, p-1);
		input = input.substr(p + 1, input.length());
	}
	str[3] = input.substr(0, input.length());
	if (str[3][str[3].length() - 1] == ' ') 
		str[3] = str[3].substr(0, str[3].length()-1);
	rs2 = atoi(str[2].c_str());
	rs1 = atoi(str[1].c_str());
	sta = state(bht[current % entry][0]);
	cout << "entry: " << current % entry << "         " << inputall[current] << endl;
	cout << "(" << bht[current % entry][0] << "," << bht[current % entry][1] << ","
		<< bht[current % entry][2] << "," << bht[current % entry][3] << "," << bht[current % entry][4] << ") ";
	if (str[0] == "beq") {
		if (reg[rs1] == reg[rs2])
		{
			if (bht[current % entry][sta][1] == 'N') {
				miss[current % entry]++;
				cout << "N "<<"T" << "                misprediction: " << miss[current % entry];			
				if (bht[current % entry][sta] == "SN") 
					bht[current % entry][sta] = "WN";
				else if (bht[current % entry][sta] == "WN") 
					bht[current % entry][sta] = "WT";			
			}
			else {
				if (bht[current % entry][sta] == "WT")
					bht[current % entry][sta] = "ST";
				cout << "T " << "T" << "                misprediction: " << miss[current % entry];
			}
			bht[current % entry][0] += '1';
			bht[current % entry][0] = bht[current % entry][0].substr(1, 2);
			current = findoffset(str[3]);
			current--;
		}
		else {
			if (bht[current % entry][sta][1] == 'N') {
				cout << "N " << "N" << "                misprediction: " << miss[current % entry];
				if (bht[current % entry][sta] == "WN")
					bht[current % entry][sta] = "SN";
			}
			else
			{
				miss[current % entry]++;
				cout << "T " << "N" << "                misprediction: " << miss[current % entry];			
				if (bht[current % entry][sta] == "ST")
					bht[current % entry][sta] = "WT";
				else if (bht[current % entry][sta] == "WT")
					bht[current % entry][sta] = "WN";
			}
			bht[current % entry][0] += '0';
			bht[current % entry][0] = bht[current % entry][0].substr(1, 2);
		}
	}
	else if (str[0] == "bne") {
		if (reg[rs1] != reg[rs2])
		{
			if (bht[current % entry][sta][1] == 'N') {
				miss[current % entry]++;
				cout << "N " << "T" << "                misprediction: " << miss[current % entry];				
				if (bht[current % entry][sta] == "SN")
					bht[current % entry][sta] = "WN";
				else if (bht[current % entry][sta] == "WN")
					bht[current % entry][sta] = "WT";
			}
			else {
				if (bht[current % entry][sta] == "WT")
					bht[current % entry][sta] = "ST";
				cout << "T " << "T" << "                misprediction: " << miss[current % entry];
			}
			bht[current % entry][0] += '1';
			bht[current % entry][0] = bht[current % entry][0].substr(1, 2);
			current = findoffset(str[3]);
			current--;
		}
		else {
			if (bht[current % entry][sta][1] == 'N') {
				cout << "N " << "N" << "                misprediction: " << miss[current % entry];
				if (bht[current % entry][sta] == "WN")
					bht[current % entry][sta] = "SN";
			}
			else
			{		
				miss[current % entry]++;
				cout << "T " << "N" << "                misprediction: " << miss[current % entry];
				if (bht[current % entry][sta] == "ST")
					bht[current % entry][sta] = "WT";
				else if (bht[current % entry][sta] == "WT")
					bht[current % entry][sta] = "WN";
			}
			bht[current % entry][0] += '0';
			bht[current % entry][0] = bht[current % entry][0].substr(1, 2);
		}
	}
		
	else if (str[0] == "blt") {
		if (reg[rs1] < reg[rs2])
		{
			if (bht[current % entry][sta][1] == 'N') {
				miss[current % entry]++;
				cout << "N " << "T" << "                misprediction: " << miss[current % entry];
				if (bht[current % entry][sta] == "SN")
					bht[current % entry][sta] = "WN";
				else if (bht[current % entry][sta] == "WN")
					bht[current % entry][sta] = "WT";
			}
			else {
				if (bht[current % entry][sta] == "WT")
					bht[current % entry][sta] = "ST";
				cout << "T " << "T" << "                misprediction: " << miss[current % entry];
			}
			bht[current % entry][0] += '1';
			bht[current % entry][0] = bht[current % entry][0].substr(1, 2);
			current = findoffset(str[3]);
			current--;
		}
		else {
			if (bht[current % entry][sta][1] == 'N') {
				cout << "N " << "N" << "                misprediction: " << miss[current % entry];
				if (bht[current % entry][sta] == "WN")
					bht[current % entry][sta] = "SN";
			}
			else
			{			
				miss[current % entry]++;
				cout << "T " << "N" << "                misprediction: " << miss[current % entry];
				if (bht[current % entry][sta] == "ST")
					bht[current % entry][sta] = "WT";
				else if (bht[current % entry][sta] == "WT")
					bht[current % entry][sta] = "WN";
			}
			bht[current % entry][0] += '0';
			bht[current % entry][0] = bht[current % entry][0].substr(1, 2);
		}
	}
	else if (str[0] == "bge") {
		if (reg[rs1] >= reg[rs2])
		{
			if (bht[current % entry][sta][1] == 'N') {
				miss[current % entry]++;
				cout << "N " << "T" << "                misprediction: " << miss[current % entry];
				if (bht[current % entry][sta] == "SN")
					bht[current % entry][sta] = "WN";
				else if (bht[current % entry][sta] == "WN")
					bht[current % entry][sta] = "WT";
			}
			else {
				if (bht[current % entry][sta] == "WT")
					bht[current % entry][sta] = "ST";
				cout << "T " << "T" << "                misprediction: " << miss[current % entry];
			}
			bht[current % entry][0] += '1';
			bht[current % entry][0] = bht[current % entry][0].substr(1, 2);
			current = findoffset(str[3]);
			current--;
		}
		else {
			if (bht[current % entry][sta][1] == 'N') {
				cout << "N " << "N" << "                misprediction: " << miss[current % entry];
				if (bht[current % entry][sta] == "WN")
					bht[current % entry][sta] = "SN";
			}
			else
			{
				miss[current % entry]++;
				cout << "T " << "N" << "                misprediction: " << miss[current % entry];
				if (bht[current % entry][sta] == "ST")
					bht[current % entry][sta] = "WT";
				else if (bht[current % entry][sta] == "WT")
					bht[current % entry][sta] = "WN";
			}
			bht[current % entry][0] += '0';
			bht[current % entry][0] = bht[current % entry][0].substr(1, 2);
		}
	}
	cout << endl << endl;
}
```  
> 處理字串，並找到對應的 register，做出相對的指令動作。    
> 實作判斷該 branch 是否需taken跳至指定位置續接執行。        
> 其中2-bit prediction依照前面所述之原理實作出其程式碼。    
```cpp
int state(string history) {//對照
 if (history == "00")
  return 1;
 if (history == "01")
  return 2;
 if (history == "10")
  return 3;
 if (history == "11")
  return 4;
}
```  
> 將 history對應至 bht中之陣列位置  

```cpp
int findoffset(string str) {//找label位置
	for (int i = 0; i < label.size(); ++i) {
		if (str == label[i]) {
			return lbnum[i];
		}
	}
}
```  
> 尋找 label下一指令位置  

## 常見問題  
- 輸入多餘的空格  
- register必須由"R"+ 數字 構成  
- label 後若不直接續接程式碼，須採換行分割，不可有多餘之空白  
 

  