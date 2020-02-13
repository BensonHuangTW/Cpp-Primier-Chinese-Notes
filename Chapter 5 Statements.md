# Chapter 5 Statements
## 5.1 Simple Statements
把expression加上`;`之後就變成一條**expression statement**了
>**Example**  
``` c++
ival + 5;
```
但上面的語句沒什麼用，因為加法被運算完之後結果沒有被使用。

#### Null Statements
最簡單的語句為空語句:
``` c++
; //null statement
```
空語句可以用在語法上需要一條語句但邏輯上不需要的時候。
>**Example**  
``` c++
//重複讀取數據直到文件尾部或是讀到的值恰為sought 的值
while (cin >> s && s != sought)
    ; // null statement
```
上面的程式也可以這樣寫:
```
while (cin >> s && s != sought)
    { } // empty block(等等會講到)
```

#### Beware of Missing or Extraneous Semicolons
要注意的是null statement仍然是一個語句，較無害的情況:
>**Example**  
```c++
ival = v1 + v2;; // ok:第二個分號頂多就是一個多餘的statement
```
然而，如果是在while或if的condition後面，則可能會完全改變程式的運行結果:
``` c++
// disaster: extra semicolon: loop body is this null statement
while (iter != svec.end()) ; // the while body is the empty statement
    ++iter; // 此行指令並不在while的迴圈裡面!!
```

#### Compound Statements (Blocks)
複合語句為花括號{}圍起來(可為空)的語句跟聲明的序列，也被稱為**block**。一個block就是一個作用域(scope)，用在語法需要一個單一語句但邏輯上需要更多的時候，例如:
``` c++
while (val <= 10) {
    sum += val; // assigns sum + val to sum
    ++val; // add 1 to val
}
```
當我們將語句們包在花括號裡時，我們把它們變成了一條(複合)語句(single (compound) statement)。

#### 5.2. Statement Scope
我們可以在`if`, `switch`, `while`,或是`for`語句的控制結構(control structure)裡面定義變數，但他們的作用範圍*只限定在該語句*之中，在語句結束之後就失效了。
>**Example**  
``` c++
while (int i = get_num()) // i is created and initialized on each iteration
    cout << i << endl; //ok
i = 0; // error: i is not accessible outside the loop(while語句結束了)
```

## 5.3. Conditional Statements
### 5.3.1. The if Statement
兩種`if`的形式:
(1)	simple if:
``` c++
if(condition)
	statement
```
(2)	if else:
``` c++
if(condition)
	statement1
else
	statement2
```
#### Dangling else
Q.如何知道某個給定的`else`跟哪個`if`匹配?  
e.g.下面的`else`是跟外層還是內層`if`匹配?  
``` c++
if (grade % 10 >= 3)
    if (grade % 10 > 7)
        lettergrade += '+'; 
else
    lettergrade += '-'; 
```
Ans.  
C++制定的規則:else與離它*最近*且*尚未與其他*`else`*匹配的*`if`*匹配*，因此上面的例子中`if`是與內層if匹配!如果要讓else與外層if匹配，可以利用花括號(因為花括號內的語句是屬於block的，所以只能作用在該block內，只要block結束後就代表內層`if`也結束了):
``` c++
if (grade % 10 >= 3) {
    if (grade % 10 > 7)
        lettergrade += '+';  // grades ending in 8 or 9 get a +
} else                  // curlies force the else to go with the outer if
    lettergrade += '-'; // grades ending in 0, 1, or 2 will get a minus
```

### 5.3.2. The switch Statement
>**Example**  
``` c++
while (cin >> ch) {
    // if ch is a vowel, increment the appropriate counter
    switch (ch) {
        case 'a':
            ++aCnt;
            break;
        case 'e':
            ++eCnt;
            break;
        case 'i':
            ++iCnt;
            break;
        case 'o':
            ++oCnt;
            break;
        case 'u':
            ++uCnt;
            break;
    }
}
```

當執行到`switch`，他會計算接在旁邊括號裡的expression，該expression可以為一個變數的初始化宣告(initialized variable declaration)，並且被轉換成*整數型別*，其結果再跟每個跟在`case`後面的expression的值做比較(comparison)，當遇到相同的(整數型別)值時，該`case`後的內容就開始被執行直到`switch`的結尾或遇到`break`語句。而case跟跟在她後面的值和稱為**case label**，必須是`const`的整數expression。如果兩個case label有相同的值，則會發生錯誤。
>**Example**  
``` c++
char ch = getVal();
int ival = 42;
switch(ch) {  
       case 3.14: // error: noninteger as case label  
       case ival: // error: nonconstant as case label  
       case '5':  
       case 53 : // error:與'5'經過整數轉換後的值相同  
   // . . .
```
case label 不需要換行。
>**Example**  
``` c++
switch(ch)
{
    // 計算母音字母的出現次數(所以故意不用break)
    case 'a': case 'e': case 'i': case 'o': case 'u':
       ++vowelCnt;
       break;
}
```
建議:通常很少會不用break，如果不用時最好註記一下原因。

#### Forgetting a break Is a Common Source of Bugs
建議:最好連最後一個case的最後都要加break，之後要新增case的時候才不會漏掉。

#### The default Label
當*沒有*case label符合switch expression的值時才會執行`default`之後的語句，default label不一定要在最後面(可以接其它label)。
label不能單一出現，必須至少在後面有一個語句或是其他label  
⇒如果default label(或其他lable)放在switch的結尾處，至少要接個空語句或空括號。

#### Variable Definitions inside the Body of a switch
Q. `switch`執行時可能會跨過一些case labels，如果剛好裡面有變數的定義 (variable definition) 會發生什麼事呢?  
Ans.  
如果某處*帶有初值*的變數在作用域外面(out of scope)，而且該變數在另一處是在作用域裡面(in scope)，則從前一處跳到後一處為非法行為。例如:
``` c++
case true:
// this switch statement is illegal because these initializations might be bypassed
    string file_name; // error: 當此case被跳，file_name (已被implicitly initialized) (見2.2.1)也會被跳過。
    int ival = 0; // error: control bypasses an explicitly initialized
    int jval; // ok:因為jval並沒被初始化。
    break;
case false:
    // ok: jval is in scope but is uninitialized
    jval = next_num(); // ok: assign a value to jval
    if (file_name.empty()) // file_name is in scope but wasn't initialized
        // ...
```
此例中，帶有初值的`ival`與`file_name`在case true中帶有初值，當case true被跳過時，它們是out of scope，然而在跳到case false時它們卻又是in scope，因而產生編譯錯誤。
想要解決這個問題，我們可以將變量定義在block裡面，這樣就能確保它們在接下來的lable是out of scope的:
``` c++
case true:
    {
       // ok: declaration statement within a statement block
       string file_name = get_file_name();
       // ...
    }
    break;
case false:
       if (file_name.empty())  // error: file_name is not in scope
```

## 5.4 Iterative Statement
`while` & `for` : 執行body之前先測條件。
`do while` : 先執行body再測條件。

### 5.4.1 The while Statement
語法:
``` c++
while (condition)
	statement
```
condition不可為空，可為expression或初始化變數的宣告，若condition為`false`則不執行statement。

### 5.4.1 Traditional for Statement
語法:
``` c++
for (init-statement condition; expression)
	statement
```
或可想成:
``` c++
for(initializer; condition; expression)
	statement
```
expression每次iteration結束後才會被運算。定義在for header的任何物件只有在該for loop的body裡面才可見。

#### Multiple Definition in the for Header
如同一般的declaration，init-statement中可以一次定義好幾個objects，然而init-statement中只能有一個declaration statement(見2.3)。所以所有被定義的變數都必須要有相同的base type(general declaration = base type + declarators )。  
>**Example**  
``` c++
// remember the size of v and stop when we get to the original last element
for (decltype(v.size()) i = 0, sz = v.size(); i != sz; ++i)
    v.push_back(v[i]);
```
在這個loop的init-statement中我們定義了`i` (做為index)以及`sz`(用於loop control)，並且他們的base type都一樣。

#### Omitting parts of the for Header
for header中的initializer、condition或是 expression皆可以為空語句(但要注意是否有跳離loop的機制)。
(1)	initializer為空  
注意就算是空的也必須加上`;`。
>**Example**  
``` c++
auto beg = v.begin();
for ( /* null */; beg != v.end() && *beg >= 0; ++beg)
    ; // no work to do
```
(2)	condition為空  
這等於是將condition直接設為`true`。例如:
``` c++
for (int i = 0; /* no condition */ ; ++i) {
    // process i; code inside the loop must stop the iteration!
}
```
(3)	expression為空  
>**Example**  
``` c++
vector<int> v;
for (int i; cin >> i; /* no expression */ )
   v.push_back(i);
```

### 5.4.3 Range for Statement
新的C++標準中，新增了range for 語句，語法如下:
``` c++
for (declaration : expression)
	statement
```
expression必須代表  
(1)	一個sequence如braced initializer list (見3.3.1)。  
(2)	一個陣列。  
(3)	擁有`begin`跟`end`這種回傳迭代器(iterator)的物件，如`vector`或`string`。  
declaration定義了一個變數，序列中的每一個元素都必須能轉換成該變數的型別，為了確保這件事，最簡單的方式就是用`auto`型別說明符(見2.5.2)，如果要對該元素writing則要用reference (見下例)。
在每次迭代中，當statement被執行完後，declaration中的control variable都會被定義而且初始化成序列中的下一個元素，當所有的元素都被處理完後，結束循環。
>**Example**  
``` c++
vector<int> v = {0,1,2,3,4,5,6,7,8,9};
// range variable must be a reference so we can write to the elements 
for (auto &r : v) // for each element in v
 r *= 2; // 將v中的每個元素*2(因為對元素進行writing，所以要用到reference)
```
以上的code其實等同於:
``` c++
for (auto beg = v.begin(), end = v.end(); beg != end; ++beg)
{
    auto &r = *beg; // r must be a reference so we can change the element
    r *= 2; // 將v中的每個元素*2
}
```
3.3.2中有提到，我們*不能*用range for為`vector`添加新元素(其他container也是)，這是因為在for range中，`end()`的值會先被存起來，如果我們新增或移除序列中的元素，`end`的值可能就會變成無效。

### 5.4.4 The do while Statement
語法:
``` c++
do
	statement
while (condition);
```
statement會先被執行，接下來才會檢查condition，如果為`true`則重複執行，`false`則終止loop。condition中使用的變數必須在`do while`的*body外*被定義。由於condition在statement結束後才會被evaluated，`do while` loop並不允許在condition中定義變數，例如:
``` c++
do {
    // . . .
    mumble(foo);
} while (int foo = get_foo()); // error: declaration in a do condition
```
上面的code中`do`會先執行`mumble(foo)`，然而此時`foo`根本就還沒被定義!

## 5.5 Jump Statements
### 5.5.1 The break Statement
適用範圍:  
(1)	`while`  
(2)	`do while`  
(3)	`for`  
(4)	`switch`  
功能:中止距離最近的正在循環中的(enclosing)loop或switch，且只影響該最近的loop或switch:  
``` c++
string buf;
while (cin >> buf && !buf.empty()) {
    switch(buf[0]) {
    case '-':
        // process up to the first blank
        for (auto it = buf.begin()+1; it != buf.end(); ++it)
{
              if (*it == ' ')
                   break; // #1, 離開for loop
              // . . .
        }
        // break #1將程式控制轉移到此
        // remaining '-' processing:
        break; // #2, 離開switch語句
    case '+':
        // . . .
    } // end switch
   // end of switch: break #2 將程式控制轉移到此
} // end while
```
### 5.5.2 The continue Statement
功能:結束距離最近的(enclosing)loop當前的迭代，並且立刻開始下一次的迭代。
適用範圍:  
(1)	`while`  
(2)	`do while`  
(3)	`for`  
單一的`switch`並不能用`continue`，除非是被包覆於`while`、`do while`或`for`中的`switch`。

### 5.5.3 The goto Statement
功能:無條件跳躍到同一個函數裡的某一個statement。
語法:
``` c++
goto lable;
```
其中，label是用於識別(標記)某條語句的identifier，一個帶標籤語句(labeled statement)是一種特殊語句，在他前面有一個identifier跟冒號。
>**Example**  
``` c++
end: return; // labeled statement;可當作goto的目標
```
其中，label identifier不會被其他變數的名稱或其他identifier而影響，也就是說，一個label的identifier(也就是名稱)可以跟其他程式裡的實體(entity，e.g.變數、物件、函數) identifier相同而不影響彼此。
與`switch`(5.3.2)相同，如果某處帶有初值的變數在作用域外面(out of scope)，而且該變數在另一處是在作用域裡面(in scope)，則從前一處跳到後一處為非法行為，例如:\
``` c++
    // . . .
    goto end;
    int ix = 10; // error: goto bypasses an initialized variable definition(out of scope)
end:
    // error: code here could use ix but the goto bypassed its declaration
    ix = 42;//(in scope)
```
`goto`可以倒退回某個變數被定義之前，該變數會*先被破壞*後再被建構。
>**Example**  
``` c++
// backward jump over an initialized variable definition is okay
  begin:
    int sz = get_size();
    if (sz <= 0) {
          goto begin;
    }
```
## 5.6. try Blocks and Exception Handling
異常處理(exception handling)用於當程式檢測到某個問題無法被解決，而且還會使得程式無法進行的時候。異常處理包括了對異常的(1)偵測(2)處理這兩部分。
在C++中，異常處理包括了:
(1)	`throw` expressions  
偵測異常的部分，用於指出遇到了無法處理的問題，可以說`throw`引發了異常(throw raises an exception)。
(2)	`try` blocks  
處理異常的部分，一個`try` block的開頭是關鍵字`try`，以一個或數個`catch` clauses 做結尾，在`try` block裡面被丟出(偵測)的異常通常由`catch` clauses中的某個`catch` clauses處理，所以它們被稱作exception handler。
(3)	exception classes  
用來傳遞關於`throw`跟相關`catch`之間的訊息(about what happened)的一套物件(見5.6.3)。

### 5.6.1 A throw Expression
程式使用`throw` expressions偵測並發出異常，其包含關鍵字`throw`以及跟隨在後的一個expression，該expression決定了被丟出的異常種類。
`throw`語句通常後面接的是分號，從而構成一個expression statement。
例子:
把兩筆同一本書(ISBN碼相同)的訂單做加法，如果ISBN碼不同則列印錯誤訊息並離開，程式碼如下:
``` c++
Sales_item item1, item2;
cin >> item1 >> item2;
// 檢查item1跟item2是否代表同一本書
if (item1.isbn() == item2.isbn()) {
    cout << item1 + item2 << endl;
    return 0; // indicate success
} else {
    cerr << "Data must refer to same ISBN"<< endl;  /*cerr是標準輸出中的其中一種，用於表示警告或error(見1.2)*/
    return -1; // indicate failure
}
```
但是在較實際的程式中，通常會將兩個物件相加的code跟與和user互動的code分離開來，我們可以用使用丟出異常的方式代替error indicator將程式改寫:
``` c++
// 檢查item1跟item2是否代表同一本書
if (item1.isbn() != item2.isbn())
    throw runtime_error("Data must refer to same ISBN");
// if we're still here, the ISBNs are the same
cout << item1 + item2 << endl;
```
此例中，如果ISBN碼不同，程式會丟出一個型別為`runtime_error`的物件，這會終止目前的函式並將控制權交給能夠處理這個異常的handler。
注意:我們必須給出一個`string`或是C-type的字元字串來initialize一個`runtime_error`(該字串提供此問題的額外資訊)。

### 5.6.2 The try Block
通用語法:
``` c++
try {
    program-statements
} catch (exception-declaration) {
    handler-statements
} catch (exception-declaration) {
    handler-statements
} // . . .
```
`catch`語法主要包含三個部分:  
(1)	關鍵字`catch`  
(2)	括號包住的一個物件聲明(declaration)，稱為exception declaration，該物件可能沒命名。  
(3)	一個block  
當某個`catch`被選擇來處理異常的時候，它的block會被執行，執行完畢之後，程式會從該`try`的最後一個`catch` clause後繼續執行。
就像其它block一樣，在`try` block裡面宣告的變數在block外就無法獲得了，就連在接在後面的`catch` clauses裡面也一樣。

#### Writing a Handler
前面的例子裡，我們利用`throw`來避免不同ISBN碼訂單相加，然後使用5.6.1提到的: “將兩個物件相加的code跟與和user互動的code分離開來”的概念，利用`try` block來實作之:
``` c++
while (cin >> item1 >> item2) {
    try {
        // execute code that will add the two Sales_items
// if the addition fails, the code throws a runtime_error exception
if (item1.isbn() != item2.isbn())
   		 throw runtime_error("Data must refer to same ISBN");
// if we're still here, the ISBNs are the same
cout << item1 + item2 << endl;
    } catch (runtime_error err) {
        // 提醒讀者ISBN碼要一樣並詢問是否要重新輸入
        cout << err.what()
             << "\nTry Again?  Enter y or n" << endl;
        char c;
        cin >> c;
        if (!cin || c == 'n')
            break; // break out of the while loop	
    }
}
```
程式碼中的`err.what()`是類別`runtime_error`的一個member function，每個STL的異常類別中都有定義名為`what`的member function，它不須輸入參數並且回傳一個C-style的字元字串(也就是`const char*`)，它是拷貝該物件被初始化時我們傳入的那個字串(本例中為"Data must refer to same ISBN")，因此當異常被丟出的時候，`catch`中的code會印出:
```
Data must refer to same ISBN
Try Again?  Enter y or n
```
### 5.6.3 Standard Exceptions
C++的STL定義了一些用於回報函式中異常的類別，它們被定義於以下四個標頭檔中:
(1)	`exception`標頭檔:  
定義了最廣義的一種異常類別:exception，它只回報異常的發生而沒其他訊息。  
(2)	`stdexcept` 標頭檔:  
定義數種異常類別，見下表。  
**Table 5.1. Standard Exception Classes Defined in <stdexcept>**
![image](https://user-images.githubusercontent.com/55428505/66983683-b79cd400-f0eb-11e9-989f-7b0a810dc2c2.png)
(3)	`new` 標頭檔:  
定義了`bad_alloc` 異常類別(見12.1.2)。  
(4)	`type_info` 標頭檔:  
定義了`bad_cast` 異常類別(見19.2)。  
