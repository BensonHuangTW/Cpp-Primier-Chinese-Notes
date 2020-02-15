# Chapter 6 Functions
## 6.1 Function Basics
#### Parameters and Arguments
**Argument**(引數):用來初始化函數之參數的initializers。
舉例:
`fact`函數的定義:
``` c++
int fact(int val)
{
    int ret = 1; // local variable to hold the result as we calculate it
    while (val > 1)
        ret *= val--;  // assign ret * val to ret and decrement val
    return ret;        // return the result
}
```
在此函數中，`val`為參數。若我們在`main()`裡面呼叫`fact`函數的時候:
``` c++
int main()
{
    int j = fact(5);  // j equals 120, i.e., the result of fact(5)
    cout << "5! is " << j << endl;
    return 0;
}
```
此時括號中的5即為引數。儘管引數的擺放有順序，但是計算引數時卻未必依照該順序逐一計算，compiler可以任意挑選哪個引數先被計算。引數的型別限制與initializer的型別規則試一致的，我們必須傳入與函數參數相同數目的引數，這保證了參數在函數被呼叫的時候必定會被初始化。
例(延續上方的`fact`函數):
``` c++
fact("hello");       // error: wrong argument type
fact();            // error: too few arguments
fact(42, 10, 0);     // error: too many arguments
fact(3.14);        // ok: argument is converted to int (且為implicitly，經過修剪)
```
故: fact(3.14)同義於fact(3)。

#### Function Parameter List
函數可以沒參數，但定義時一定要有括號:
``` c++
void f1(){ /* ... */ }     // implicit void parameter list
```
為了與C語言相容，也可以用下面的語法:
``` c++
void f2(void){ /* ... */ } // explicit void parameter list
```
就算兩個參數型別一樣，仍必須把兩個型別都寫出來:
``` c++
int f3(int v1, v2) { /* ... */ }     // error
int f4(int v1, int v2) { /* ... */ }  // ok
```

#### Function Return Type
函數可以不回傳值(使用`void`)，但不能回傳一個陣列型別或者函數型別，後面的章節會提到如何定義回傳(1)指向陣列的pointer(或是reference)以及(2)指向函數的指標方式(6.7)。

### 6.1.1 Local Objects
在C++裡面，名稱有**作用域**(**scope**)，而物件則有**生命週期**(**lifetime**)。  
(1)名稱的作用域是指該名稱在程式中可見(visible)的部分。  
(2)物件的生命週期是指在程式執行過程中，該物件存在的時間。  
函數的block形成一個新的作用域，而函數的:  
(1)參數  
(2)定義在函數體裡面的變數  
我們稱作**局部變數**(local variables)。它們只在函數的內部可見，而且會隱藏外層作用域中擁有相同名稱的聲明(They are “local” to that function and hide declarations of the same name made in an outer scope.)。  
定義於所有函數外部的物件存在於整個程式執行的過程中，而局部變數的生命週期取決於它被定義的方式。

#### Automatic Objects
對應於一般的局部變數所對應的物件來說，它們被創建於當函數的控制路徑(control path)通過該變數的定義時，並且在通過該變數被定義的block末端被銷毀。這種只存在於一個block之中的物件稱為automatic objects，當該block執行結束後，裡面定義之automatic objects的值就會變成未定義的。而參數是一種automatic objects。  
函數體中，作為局部變數的automatic objects如果在被定義時若沒有initializer，則會被默認初始化，這代表未被初始化的內建型別會有未定義的值(若在函數體外則會被初始化成0，見2.2.1)。

#### Local static Objects
當我們用`static`定義某個局部變數時，可以使它的生命週期超越函數被呼叫的時候，**Local `static` Objects**在該變數的定義被第一次執行之前就被初始化了，並且直到程式終止才被銷毀。  
>**Example**  
``` c++
size_t count_calls()
{
    static size_t ctr = 0;  // value will persist across calls
    return ++ctr;
}
int main()
{
    for (size_t i = 0; i != 10; ++i)
        cout << count_calls() << endl;
    return 0;
}
```
上述程式輸出結果為:
```
1
2
3
4
5
6
7
8
9
10
```
如果一個局部靜態變數沒被explicit initialized，則它會被value initialized(3.3.1)，也就是說內建類型的type會被初始化成0。

### 6.1.2 Function Declarations
如同其他的名稱一樣，函數的名子必須在使用前就被聲明。如同變數一樣，一個函數只能被定義一次但可以被多次聲明(例外:見15.3)。函數的聲明與定義的差別在於函數聲明沒有函數體，而是用`;`取代之，因此參數不需要有名子，可以被忽略，但為了讓使用者知道函數的功能也可以加上名子。  
>**Example**
``` c++
// parameter names chosen to indicate that the iterators denote a range of values to print
void print(vector<int>::const_iterator beg,
           vector<int>::const_iterator end);
```
上面的`beg`與`end`都可以被省略。  
函數聲明的三個元素:(1)回傳型別(2)函數名稱(3)參數型別  
這些元素描述了函數的interface(接口)，函數的聲明又被稱作**function prototype**(函數原型)。

#### Function Declarations Go in Header Files
建議:將函數於header file中宣告，並把該header file include到要用的source file中，並於source file中定義。  
優點:  
(1)確保同一函數的聲明保持一致。  
(2)當函數的interface要改動時只需改變header file中的聲明即可。  

## 6.2 Argument Passing
參數的初始化與變數的初始化運作方式是相同的。  
根據參數的類型可將引數被傳遞的方式分為下面兩種:  
(1)	**passed by value**  
參數不是reference，則引數的值會被複製到參數，至此之後，參數與引數為*互相獨立*的物件，我們又稱此函數被**called by value**。  
(2)	**passed by reference**  
參數為reference，則參數會被綁定到引數上(與一般的reference一樣)，我們又稱此函數被**called by reference**。  

### 6.2.1 Passing Arguments by Value
改變參數的值不會影響到傳入該函數的引數。  

#### Point Parameters
跟其他非reference的值一樣，當我們將指標當成參數，會把引數指標的值複製到參數中，不過透過指標我們可以indirect access它指向的物件(與引數指向的物件一樣)。  
>**Example**  
``` c++
// function that takes a pointer and sets the pointed-to value to zero
void reset(int *ip)
{
    *ip = 0;  // changes the value of the object to which ip points
    ip = 0;   // changes only the local copy of ip; the argument is unchanged
}
```
>**Best Practice**  
習慣C語言程式設計的程式設計師常使用point parameters參訪(access)參數外的變數，而在C++中，程式設計師則通常使用reference parameters。

### 6.2.2 Passing Arguments by Reference
對參數的操作同時就是對參照的引數進行操作。

#### Using Reference to Avoid Copies
複製一個很大的class type或是containers是沒有效率的一件事，甚至有些class type(包括IO型別)根本就不能被複製，這時候我們就會用reference形式的參數來操作這些對象，避免複製的發生。  
>**Example**
當我們想比較兩個可能很長的字串的長度，我們想避免複製，因此我們將使用reference類型的參數。由於比較長度並不需要改變字串的內容，因此我們將參數定義成對`const`的reference:
``` c++
// compare the length of two strings
bool isShorter(const string &s1, const string &s2)
{
    return s1.size() < s2.size();
}
```

>**Best Practice**  
如果沒要在函數中改變reference parameters的值，則應該將其reference to `const`。

#### Using Reference Parameters to Return Additional Information
函數雖然只能回傳一個值，但透過reference parameters我們可以傳出多個值。
>**Example**  
想要知道某個字元在字串第一次出現的(1)位置(2)出現次數，我們可以定義出一個新的型別來包含這兩種資訊，然而一個更簡單的作法可以透過reference parameters來達成:
``` c++
// returns the index of the first occurrence of c in s
// the reference parameter occurs counts how often c occurs
string::size_type find_char(const string &s, char c,
                           string::size_type &occurs)
{
    auto ret = s.size();   // position of the first occurrence, if any
    occurs = 0;            // set the occurrence count parameter
    for (decltype(ret) i = 0; i != s.size(); ++i) {
        if (s[i] == c) {
            if (ret == s.size())
                ret = i;   // remember the first occurrence of c
            ++occurs;      // increment the occurrence count
         }
    }
    return ret;            // count is returned implicitly in occurs
}
```
當`s`為字串，`ctr`為一個`size_type`(見3.2.2)的物件，我們呼叫`find_char`如下:
``` c++ 
auto index = find_char(s, 'o', ctr);
```
最後我們可以獲得`o`字元出現的第一個位置於`index`中，出現次數則呈現於`ctr`之中。

### 6.2.3 const Parameters and Arguments
在2.4.3的討論中我們知道`const`物件的複製是忽略top-level const的，所以當我們用複製引數的方式來初始化參數的時候，top-level `const`是會被忽略的。也就是說參數的top-level `const`不會被考慮進去，所以我們可以把`const`或是non`const`物件傳給一個有top-level `const`的參數中，例如:
``` c++
Int i = 5;
void fcn(const int i) { /* fcn can read but not write to i */ }
也因此這造成了以下的結果:
void fcn(const int i) { /* fcn can read but not write to i */ }
void fcn(int i) { /* . . . */ } // error: redefines fcn(int)
```
C++裡面雖然允許函數的重載，然而上面兩個可載入的引數型別是完全相同的(因為參數i的top-level const會被忽略)，如果上面程式碼合法會造成不知道到底要傳入的函數是哪一個，因此被視作違法。

#### Pointer or Reference Parameters and const
參數被初始化的方式與變數初始化的規則一樣，特別要記住我們可以用一個non`const`的物件初始化一個有low-level `const`的參數，反過來則不行。  
>**Example**  
``` c++
// function that takes a reference to an int and sets the given object to zero
void reset(int &i)  // i is just another name for the object passed to reset
{
    i = 0;  // changes the value of the object to which i refers
}

// function that takes a pointer and sets the pointed-to value to zero
void reset(int *ip)
{
    *ip = 0;  // changes the value of the object to which ip points
    ip = 0;   // changes only the local copy of ip; the argument is unchanged
}
```

#### Use Reference to const When Possible
當我們用reference的方式傳遞參數，應*盡可能*的使用reference to `const`的方式，這是因為如果用一般的reference會有下列的缺點:  
(1)	使得該函數的呼叫者(caller)以為它會改變引數的值  
(2)	可以傳入的引數類型被限縮(因為const不能轉成nonconst)  
(3)	以上的負面效應甚至可能會被放大  
>**Example**  
``` c++
//函數A的程式碼如下(使用reference to const傳遞引數)
int A(const string &s1){
		//……
		return B(s1,i);
}
//如果B的定義如下(未使用reference to const傳遞引數)
int B(string &s2, int a){
		//……
}
```
以上的程式碼會在編譯時就發生錯誤，這是因為`s1`本身是一個reference to a`const string`，然而s2只允許非`const`的`string`傳入，因此發生錯誤，雖然看起來我們可以把`A`的定義改成`int A(string &s1)`就不會發生編譯錯誤，然而這麼做卻只是把這種設計不良的錯誤給擴大而已。

#### 6.2.4. Array Parameters
由於array不能被copy(見3.5.1)，而且經常被轉換成pointer(見3.5.3)，當我們把array傳入函式時，其實*傳入的是一個指標*。雖然我們不能把array傳入函式，我們仍可以把參數寫成看起來像array的樣子:
``` c++
//以下三種宣告表面上看起來不同，實際上卻是等價的
//每種函式都有一個型別為const int*的參數
void print(const int*);
void print(const int[]);   // 表示該函數需要一個array
void print(const int[10]);  // 只有說明用途，但是編譯器"不會"檢查傳入的陣列大小
```
當我們呼叫`print`時，編譯器只會檢查引數的型別是否為`const int*`，也就是說當我們把array傳入`print`時，它會自動被轉成指向該array第一個元素的pointer，與該array的大小無關。

>**Warning**
就如同任何使用到array的程式碼，我們須注意把array當參數的函式中，使用該參數時是否有超過array的邊界。  

由於array是被當成pointer傳入函式的，所以函式該array的大小，必須由caller提供額外資訊，以下為三種常用的方法:
(1)	Using a Marker to Specify the Extent of an Array  
例: 
C-style的字串最後一個字元一定都是空字元(null character)，因此我們可以用它來處理字串。
>**Example**  
``` c++
void print(const char *cp){
    if (cp)          // if cp is not a null pointer
      while (*cp) // so long as the character it point to is not a null character
          cout << *cp++; // print the character and advance the pointer
}
```

(2)	Using the Standard Library Conventions  
可以利用類似標準函式庫中的iterator，將指向第一個與最後一個元素的pointer傳入函數中  
>**Example**  
``` c++
void print(const int *beg, const int *end){
    // print every element starting at beg up to but not including end
    while (beg != end)
        cout << *beg++ << endl; // print the current element
                                // and advance the pointer
}
```
當我們要呼叫該函數時，可以利用`begin`與`end`函數(見3.5.3)來提供這兩個pointer:  
``` c++
int j[2] = {0, 1};
// j is converted to a pointer to the first element in j
// the second argument is a pointer to one past the end of j
print(begin(j), end(j)); // begin and end functions, see § 3.5.3
```

(3)	Explicitly Passing a Size Parameter  
此法常用在C語言與叫老的C++程式中，第二個參數指出了該陣列的大小:  
``` c++
// const int ia[] is equivalent to const int* ia
// size is passed explicitly and used to control access to elements of ia 
void print(const int ia[], size_t size)
{
    for (size_t i = 0; i != size; ++i) {
        cout << ia[i] << endl;
    }
 }
```

#### Array Reference Parameters
我們也能用reference to array(見3.5.1)的方式定義參數:  
``` c++
// ok: param eter is a reference to an array; the dimension is part of the type
void print(int (&arr)[10])
{
    for (auto elem : arr)
        cout << elem << endl;
}
```

>**Note**  
包住`&arr`的括號是*必須*的(見3.5.1):
``` c++
f(int &arr[10])   // error: declares arr as an array of references
f(int (&arr)[10]) // ok: arr is a reference to an array of ten ints
```
雖然這種傳遞方式比較安全，但array的大小被限制了，降低了實用性:  
``` c++
int i = 0, j[2] = {0, 1};
int k[10] = {0,1,2,3,4,5,6,7,8,9};
print(&i);   // error: argument is not an array of ten ints
print(j);    // error: argument is not an array of ten ints
print(k);    // ok: argument is an array of ten ints
```

在16.1.1中會學到該如何傳入reference to任意大小的array

#### Passing a Multidimensional Array
在C++裡，多維陣列其實就是array of arrays，就如同之前的array一樣，傳入函式的其實是一個指向該array第一個元素的pointer，且為指向一個array的pointer，而第二層(以及之後的層)的array的size是該array的元素型別的一部分，所以必須被標明。  
例如說如果我們要傳入一個二維陣列，可以這樣寫:  
``` c++
// matrix points to the first element in an array whose elements are arrays of ten ints
void print(int (*matrix)[10], int rowSize) { /* . . . */ }
```
上面宣告matrix為一個指向大小為10的`int` arrays的pointer。我們也可以使用array的語法來定義(且與上面的宣告等價)，跟上面一樣，編譯器會忽略第一個維度的大小，所以最好忽略它:  
``` c++
void print(int matrix[][10], int rowSize) { /* . . . */ }
```

>**Note**  
包住*matrix的括號是必要的:
``` c++
int *matrix[10];   // array of ten pointers
int (*matrix)[10]; // pointer to an array of ten ints
```
### 6.2.5 main: Handling Command-Line Options
`main`即為一個將陣列傳入函式的例子，之前所定義的`main`函式的parameter list都是空的:
``` c++
int main() { … }
```
然而有時候我們需要把引數傳入`main`裡面，這些引數常被用來給使用者標示想要該程式執行的operation，假設`main`程式存在於名為`prog`的執行檔中，則我們可以用下面的方式傳遞option給該程式:  
``` unix
Prog –d –o ofile data 0
``` 
(在linux中則可在執行檔所在目錄中輸入` ./ Prog –d –o ofile data 0 `)
上面的command-line option會被傳入兩個(optional)參數:
``` c++
int main(int argc, char *argv[]) { … }
```
第二個參數是一組指向C-style character string的pointer所成之陣列，第一個`argc`參數則傳遞了該陣列的字串數目，我們也可以用另一種方式定義`main`:
``` c++
int main(int argc, char **argv) { … }  //標明argv指向一個char*
```
當引數被傳進`main`，`argv`中第一個元素指向的會是該程式的名稱或者空字串(而非user input)，接下來的元素則會傳遞我們在command line提供的引數，而最後一個pointer的下一個元素則必為`0`(Null pointer)。
>**Example**  
若prog.c的程式碼如下:
``` c++
#include <stdio.h>
#include <string.h>
int main(int argc, char *argv[]){
	
	printf("the value of argc is %d\n",argc);
	int count = 0;
	while(count < argc){
		printf("argv [%d] =  %s\n",count,argv[count]);
		count ++;
	}
	return 0;
}
```
則將其編譯成執行檔後在該目錄command line輸入:
``` linux
Prog –d –o ofile data 0 
```
所得到的輸出為:
``` linux
the value of argc is 5
argv [0] =  ./prog
argv [1] =  -d
argv [2] =  -o
argv [3] =  ofile
argv [4] =  data0
```
### 6.2.6 Functions with Varying Parameters
當引數的數量是未知時，新標準有兩種主要方法來傳遞數量變動的引數:  
(1)	`initializer_list` Parameters  
適用於未知數量的引數型別都是相同的情況，定義於`initializer_list`標頭檔中，其用法和`vector`類似，當我們要定義一個`initializer_list`時，必須標明它元素的型別:  
``` c++
initializer_list<string> ls; // initializer_list of strings
initializer_list<int> li;    // initializer_list of ints
```
與`vector`不同的是`initializer_list`的元素永遠都是常量，`initializer_list`也有自己的`begin`與`end` iterators，功能如之前所述
>**Example**  
``` c++
void error_msg(initializer_list<string> il)
{
    for (auto beg = il.begin(); beg != il.end(); ++beg)
        cout << *beg << " " ;
    cout << endl;
}
```
當我們傳遞一連串值到`initializer_list`參數時，必須要包在花括號裡:
``` c++
// expected, actual are strings
if (expected != actual)
    error_msg({"functionX", expected, actual});
else
    error_msg({"functionX", "okay"});
```
(2)	variadic template  
適用於當未知數量的引數之型別*不一致*時，見16.4。  

## 6.3 Return Types and the return Statement
`return`語句使執行中的函數終止並將控制權歸還到該函數被呼叫的地方，其形式有兩種:  
``` c++
return
return expression;
```

### 6.3.1 Functions with No Return a Value
不包含回傳值的`return`只會被用於回傳型別為`void`的函式，並且`void`函式並不要求一定要有`return`，有一個implicit的`return`會放在該函式的最後一條語句之後。通常`void`函式會使用`return`來中途離開該函數，用法類似`break`語句。例如當我們想交換兩物件時，當兩個物件相等時，可用`return`來直接跳過交換的部分:
``` c++
void swap(int &v1, int &v2)
{
    // if the values are already the same, no need to swap, just return
    if (v1 == v2)
        return;
    // if we're here, there's work to do
    int tmp = v2;
    v2 = v1;
    v1 = tmp;
    // no explicit return necessary
}
```
`void`函式也可以用到`return expression`這種形式，然而它只能回傳某個也是回傳`void`的函式，其他不是`void`函式的expression會導致編譯錯誤。

### 6.3.2 Functions That Return a Value
C++不能確保回傳結果的正確性(correctness)，但它能保證每個`return`會有適當的型別:
``` c++
// incorrect return values, this code will not compile
bool str_subrange(const string &str1, const string &str2)
{
    // same sizes: return normal equality test
    if (str1.size() == str2.size())
        return str1 == str2;   // ok: == returns bool
    // find the size of the smaller string
    auto size = (str1.size() < str2.size())
                ? str1.size() : str2.size();
    // look at each element up to the size of the smaller string
    for (decltype(size) i = 0; i != size; ++i) {
        if (str1[i] != str2[i])
            return; // error #1: no return value; compiler should detect this
error
    }
    // error #2: control might flow off the end of the function without a return
    // the compiler might not detect this error
}
```
上面的error #2如果沒被編譯器偵測出來，則在執行時會發生未定義的情況。

#### How Values Are Returned
傳回值的方式就跟變數與參數被初始化時一樣:回傳值在函式呼叫處被用來初始化一個暫時量，該暫時量即為函式回傳的結果。  
>**Example**  
``` c++
// return the plural version of word if ctr is greater than 1
string make_plural(size_t ctr, const string &word,
                               const string &ending)
{
    return (ctr > 1) ? word + ending : word;
}
```
就跟其他reference一樣，如果函式回傳的是一個reference，則回傳的就是被refer的物件的另一個name而已。
>**Example**  
``` c++
// return a reference to the shorter of two strings
const  string  &shorterString(const  string  &s1,  const  string
&s2)
{
    return s1.size() <= s2.size() ? s1 : s2;
}
```
上例中，參數以及回傳值的型別都是reference to const string，因此這些`string`在函數被呼叫以及回傳結果時都不會經過copy。

#### Never Return a Reference or Pointer to a Local Object
當一個函式完成(complete)之後，其記憶體空間也被釋放，所以當函式終止後，指向其local objects的pointer或reference*都會失效*:
``` c++
//災難: this function returns a reference to a local object
const string &manip()
{
    string ret;
   // transform ret in some way
   if (!ret.empty())
  return ret;     // WRONG: returning a reference to a local object!
   else
       return "Empty"; // WRONG: "Empty" is a local temporary string
}
```
同樣地，回傳指向該函式local objects的pointer也是不對的。

#### Functions That Return Class Types and the Call Operator
call operator跟dot(`.`)與arrow operator(`->`)(見4.1.2)有相同的優先權，而且它們全都是左結合。也就是說如果某函式回傳一個物件型別的reference或pointer的話，就能用該回傳結果去呼叫該物件的成員。  
>**Example**  
``` c++
// call the size member of the string returned by shorterString
auto sz = shorterString(s1, s2).size();	//sz得到s1或s2的size
```

#### Reference Returns Are Lvalues
對於回傳reference的函式進行呼叫會得到一個左值，故我們可以對其賦值(如果不是`const` reference的話)，其他的回傳型別則得到右值。
>**Example**  
``` c++
char &get_val(string &str, string::size_type ix)
{
    return str[ix]; // get_val assumes the given index is valid
}
int main()
{
    string s("a value");
    cout << s << endl;   // prints a value
    get_val(s, 0) = 'A'; // changes s[0] to A
    cout << s << endl;   // prints A value
    return 0;
}
``` 
#### Returning a Pointer to an Array
array無法被copy，故函式*無法回傳一個array*，但我們可以回傳一個指向array的指針或是array的reference，其函式宣告語法與變數宣告類似，只是函式的宣告多了一個緊跟在函式名稱之後的parameters list，如下:
``` c++
Type (*function(parameter_list))[dimension]
```
>**Example**  
假設`func`函式有一個`int`參數，並回傳一個指向大小為`10`的`int` array的pointer則根據上面的語法其宣告如下:
``` c++
int (*func(int i))[10];
```
這種宣告方式略為複雜，因此我們可以利用下列幾種方式來簡化:  
(1)	使用type alias(見2.5.1)  
>**Example**  
``` c++
typedef int arrT[10];  // arrT is a synonym for the type array of ten ints
using arrtT = int[10]; // equivalent declaration of arrT; see § 2.5.1
arrT* func(int i);     // func returns a pointer to an array of five ints
```
(2)	使用**Trailing Return Type**  
trailing return可被定義於任何的函式之中，但在處理回傳複雜型別的函式中最有用，trailing return type跟在parameter list接上`->`之後，並且將`auto`放在原本return type該出現的地方:
``` c++
// fcn takes an int argument and returns a pointer to an array of ten ints
auto func(int i) -> int(*)[10];
```
(3)	Using `decltype`:  
``` c++
int odd[] = {1,3,5,7,9};
int even[] = {0,2,4,6,8};
// returns a pointer to an array of five int elements
decltype(odd) *arrPtr(int i)
{
    return (i % 2) ? &odd : &even; // returns a pointer to the array
}
```
## 6.4 Overloaded Functions
>**Note**  
`main`函式無法被重載

#### Defining Overloaded Functions
函數允許回傳型別的不同，但是如果僅僅是回傳型別不同也會引發錯誤。
``` c++
Record lookup(const Account&);
int lookup(const int&); //ok
bool lookup(const Account&);   // error: only the return type is different
```

#### Determining Whether Two Parameter Types Differ
以下為兩組看似不同實則完全一樣的函數宣告:
``` c++
// each pair declares the same function
Record lookup(const Account &acct);
Record lookup(const Account&); // parameter names are ignored
typedef Phone Telno;
Record lookup(const Phone&);
Record lookup(const Telno&); // Telno and Phone are the same type
```

#### Overloading and const Parameters
在6.2.3有提到top-level `const`不會影響到物件是否能傳入函式中，也就是擁有top-level `const`的參數和沒有top-level `const`的參數其實無法辨別:
``` c++
Record lookup(Phone);
Record lookup(const Phone);   // redeclares Record lookup(Phone)
Record lookup(Phone*);
Record lookup(Phone* const);  // redeclares Record lookup(Phone*)
```
另一方面，low-level `const`的有無是可以重載成兩個函式的:
``` c++
//使用const and nonconst references 或 pointers have different parameters
// declarations for four independent, overloaded functions
Record lookup(Account&);       // function that takes a reference to account
Record lookup(const Account&);  //new function that takes a const reference
Record  lookup(Account*);     // new function, takes a pointer to Account
Record lookup(const Account*); // new function, takes a pointer to const
```
雖然看起來第二個以及第四個被宣告的函數可以藉由轉換把non`const`的參數傳入，然而在6.6.1會提到，編譯器偏好non`const`的函式版本，也就是第一個及第三個。

#### const_cast and Overloading
在4.11.3中有提到`const_cast`的功能，它在函數重載的部分很有用:  
>**Example**  
假設我們已經寫好了以下的函式:
``` c++
// return a reference to the shorter of two strings
const string &shorterString(const string &s1, const string &s2){
    return s1.size() <= s2.size() ? s1 : s2;
}
```
上面的函式輸入與回傳的是對`const string`的reference，如果用兩個對non`const string`的reference呼叫該函式，會得到一個對`const string`的reference，但如果今天我們想得到的是一個對`string`的reference (沒有`const`)，我們可以利用`const_cast`來重載另一個版本的函式:
``` c++
string &shorterString(string &s1, string &s2){
auto &r = shorterString(const_cast<const string&>(s1),
    			    const_cast<const string&>(s2));
return const_cast<string&>(r);
}
```
首先我們把參數變成reference to `const string`，接著傳入原來版本的函式中，得到reference to `const string`的結果後再轉回reference to `string`。

### 6.4.1 Overloading and Scope
如果我們在inner scope宣告一個name，則該name會隱藏在outer scope宣告之相同的name，也就是說name並不能跨越scope重載:
``` c++
string read();
void print(const string &);
void print(double);   // overloads the print function
void fooBar(int ival)
{
    bool read = false; // new scope: hides the outer declaration of read
    string s = read(); // error: read is a bool variable, not a function
    // bad practice: usually it's a bad idea to declare functions at local scope
    void print(int);  // new scope: 隱藏先前所宣告的print
    print("Value: "); // error: print(const string &) is hidden
    print(ival);      // ok: print(int) is visible
    print(3.14);      // ok: calls print(int); print(double) is hidden
}
```
## 6.5 Features for Specialized Uses
### 6.5.1 Default Arguments
如果我們想讓參數有自己的默認引數，我們可以將它如一般的變數宣告成default arguments:
``` c++
typedef string::size_type sz;  // typedef see § 2.5.1 (p. 67)
string screen(sz ht = 24, sz wid = 80, char backgrnd = ' ');
```
必須注意如果某個參數有default arguments，則*之後*所有的參數都*必須有default arguments*。

#### Calling Functions with Default Arguments
當我們要使用default arguments時，只要在呼叫函式的時候忽略輸入該引數即可。  
>**Example**  
``` c++
string window;
window = screen();  // equivalent to screen(24,80,' ')
window = screen(66);// equivalent to screen(66,80,' ')
window = screen(66, 256);      // screen(66,256,' ')
window = screen(66, 256, '#'); // screen(66,256,'#')
```

默認引數只能負責填補右後端缺少的引數:
``` c++
window = screen(, , '?'); // error: can omit only trailing arguments
window = screen('?');     /* 編譯成功:但calls screen('?',80,' ')(因為?被轉換成63並初始化到ht中)*/
```

#### Default Argument Declarations
函數雖然能被多次宣告，然而default arguments只能被宣告一次，之前已被宣告過的在下一次宣告就不能再次宣告default arguments了:
``` c++
// no default for the height or width parameters
string screen(sz, sz, char = ' ');
string screen(sz, sz, char = '*'); 		  // error: redeclaration
string  screen(sz = 24, sz = 80, char);   // ok: adds default
```
#### Default Argument Initializers
local variables不能作為默認引數，除此之外只要能透過轉換任何expression都能當成默認引數:
``` c++
// the declarations of wd, def, and ht must appear outside a function
sz wd = 80;
char def = ' ';
sz ht();
string screen(sz = ht(), sz = wd, char = def);
string window = screen(); // calls screen(ht(), 80, ' ')

void f2()
{
    def =   '*';   // changes the value of a default argument
    sz wd = 100; // hides the outer definition of wd but does not change the default
    window = screen(); // calls screen(ht(), 80, '*')
}
```
當我們呼叫`f2`時，`def`的值被改成`*`，造成我們呼叫screen時其默認引數的值也被改變。

### 6.5.2 Inline and constexpr Functions
在6.3.2中我們定義了這個函式:
``` c++
const  string  &shorterString(const  string  &s1,  const  string
&s2)
{
    return s1.size() <= s2.size() ? s1 : s2;
}
```
雖然函式內容只有一條語句，但是它兼具了以下的優點(與不用函式直接coding比起來):  
- 易讀  
- 確保在每一處的行為都一致  
- 若要改變內容只需在函數定義處做修改就好  
- 可以被其他應用重複利用  
然而一般在呼叫函式的時候會經過許多工作，降低了效能。  

#### inline Functions Avoid Function Call Overhead
為了避免效能的降低，我們可以標記函式為`inline`，這麼做就會要求編譯器在該函數被呼叫時，直接在該處展開它的程式碼。
>**Example**  
``` c++
cout << shorterString(s1, s2) << endl;
```
如果用了inline，可能會在編譯時被展開成下面的樣子:
``` c++
cout << (s1.size() < s2.size() ? s1 : s2) << endl;
```
如此一來便降低了運行時的負擔。其定義方式為在return type的前面加上`inline`:
``` c++
// inline version: find the shorter of two strings
inline const string &
shorterString(const string &s1, const string &s2)
{
        return s1.size() <= s2.size() ? s1 : s2;
}
```

>**Note**  
`inline`只是向編譯器提出要求，編譯器*可以無視*這個要求。

一般來說，`inline`用於規模較小、直接且常呼叫的函式，很多編譯器不在recursive functions支援`inline`。

#### constexpr Functions
`constexpr` Functions是可以被用在constant expression裡面的函式，constexpr Functions被定義的方式與其他函式相同，然而有幾個限制:  
(1)	return type 以及所有的參數都必須是literal type。  
(2)	函式的body必須要恰好有一個`return`語句。  
``` c++
constexpr int new_sz() { return 42; }
constexpr int foo = new_sz();  // ok: foo is a constant expression
```
我們允許`constexpr`函式的回傳值不是常量:
``` c++
// scale(arg) is a constant expression if arg is a constant expression
constexpr size_t scale(size_t cnt) { return new_sz() * cnt; }
上面的scale函式如果傳入的引數是const expression則輸出也為const expression反之則不是:
int arr[scale(2)]; // ok: scale(2) is a constant expression
int i = 2;         // i is not a constant expression
int a2[scale(i)];  // error: scale(i) is not a constant expression
``` 

>**Note**  
`constexpr`函式回傳的不一定是`const` expression。

#### Put inline and constexpr Functions in Header Files
為了一致性，通常`inline`與`constexpr`會直接定義於標頭檔中。

## 6.6Function Matching
例.決定下列哪個函式會被呼叫
``` c++
void f();
void f(int);
void f(int, int);
void f(double, double = 3.14);
f(5.6);  // calls void f(double, double)
```
`步驟1`  
檢查(挑選candidate functions)  
(1)	函式name  
(2)	函式在呼叫處是否可見visible  
⇒以上四個符合  

`步驟2`  
檢查  
(1)	引數個數  
(2)	引數型別與參數型別是否相符或可透過轉換  
⇒ `void f(int) `與` void f(double, double = 3.14) `相符(第二個有默認引數)  

`步驟3`  
找出與該呼叫最佳匹配的函式。  
何謂最佳?即引數與參數的型別越相近越佳(e.g.不須經由轉換比需要經由轉換佳)。  

## 6.7 Pointers to Functions
如同其他pointer，function pointer指向一個特定型別的函式，一個函式的型別取決於它的回傳型別以及參數型別，但是函式名稱不包含其中，例如:  
``` c++
// compares lengths of two strings
bool lengthCompare(const string &, const string &);
```
上面的函式型別為``` bool(const string&, const string&)```，當我們要宣告一個指向此函式的指標時，我們將指標宣告於函式名稱原本該出現的地方:
``` c++
// pf points to a function returning bool that takes two const string references
bool (*pf)(const string &, const string &);  // 未初始化的指標
```
>**Note**  
包圍住`*pf`的括號必須存在，否則我們會宣告成一個回傳`bool` pointer的函式:
``` c++
// declares a function named pf that returns a bool*
bool *pf(const string &, const string &);
```

#### Using Function pointers
當我們將函式的名稱當成值使用時，它會自動被轉成一個pointer，比如，我們可以藉由下面兩種方式將`lengthCompare`的位址賦值給`pf`:
``` c++
pf = lengthCompare;  // pf now points to the function named lengthCompare
pf = &lengthCompare; // 與上面等價:address-of operator is optional
```
此外我們可以使用function pointer來使用它所指向的函式:
``` c++
bool b1 = pf("hello", "goodbye");    // calls lengthCompare
bool b2 = (*pf)("hello", "goodbye"); //與上面等價
bool b3 = lengthCompare("hello", "goodbye"); // 同上
```
兩個指向不同函式型別的指針之間不存在轉換，但我們可以用nullptr跟0來賦值，表示他們並未指向任何函式:
``` c++
string::size_type sumLength(const string&, const string&);
bool cstringCompare(const char*, const char*);
pf = 0;              // ok: pf points to no function
pf = sumLength;      // error: return type differs
pf = cstringCompare; // error: parameter types differ
pf = lengthCompare;  // ok: function and pointer types match exactly
```

#### Pointers to Overloaded Functions
當函式有重載的情形，編譯器會根據指針的型別來決定指向哪一個，指針型別必須與某個重載函式的型別確實匹配:
``` c++
void ff(int*);
void ff(unsigned int);
void (*pf1)(unsigned int) = ff;  // pf1 points to ff(unsigned)
void (*pf2)(int) = ff;    // error: no ff with a matching parameter list
double (*pf3)(int*) = ff; // error: return type of ff and pf3 don't match
``` 

#### Function Pointer Parameters
我們不能直接把參數的型別定義成函式的型別，但可以是函式的指針，就如同array一樣，我們可以把參數寫的像函式，但實際上它是被當成指標看待:
``` c++
// 第三個參數是函式型別，但它被自動當成pointer to function對待
void useBigger(const string &s1, const string &s2,
               bool pf(const string &, const string &));
//與上方等價:explicitly define the parameter as a pointer to function
void useBigger(const string &s1, const string &s2,
               bool (*pf)(const string &, const string &));
我們可以直接把函式名稱傳入該函式，它會被自動轉成pointer:
// automatically converts the function lengthCompare to a pointer to function
useBigger(s1, s2, lengthCompare);
```
某些函數的型別很複雜，如果搭配`typedef`跟`decltype`，可以簡化程式碼:
``` c++
// Func and Func2 have function type
typedef bool Func(const string&, const string&);
typedef decltype(lengthCompare) Func2; // equivalent type
```
``` c++
// FuncP and FuncP2 have pointer to function type
typedef bool(*FuncP)(const string&, const string&);
typedef decltype(lengthCompare) *FuncP2;  // equivalent type
```
該注意的是`decltype`回傳的是函式型別而非指標，如果要的是指標的話則*一定*要加上`*`，有了上面的簡化，`useBigger`可以這樣宣告:
``` c++
// equivalent declarations of useBigger using type aliases
void useBigger(const string&, const string&, Func);
void useBigger(const string&, const string&, FuncP2);
```

#### Returning a Pointer to Function
函式不能直接回傳函式型別，但可以回傳pointer to function，一個比較容易的方法是使用type alias來簡化語法:
``` c++
using F = int(int*, int);     // F is a function type, not a pointer
using PF = int(*)(int*, int); // PF is a pointer type
```
要記住的是不像在參數中函數型別可以自動轉為pointer to function，在宣告中我們必須清楚標明該回傳型別是一個pointer:
``` c++
PF f1(int); // ok: PF is a pointer to function; f1 returns a pointer to function
F f1(int);  // error: F is a function type; f1 can't return a function
F *f1(int); // ok: explicitly specify that the return type is a pointer to function
```
當然我們也可以直接宣告，只是比較麻煩:
``` c++
int (*f1(int))(int*, int);
```
另外我們也可以使用trailing return(見6.3.3):
``` c++
auto f1(int) -> int (*)(int*, int);
```

#### Using auto or decltype for Function Pointer Types
如果知道了我們想要回傳的是哪個函數，我們可以用`decltype`來簡化回傳型別的程式碼。  
>**Example**  
``` c++
string::size_type sumLength(const string&, const string&);
string::size_type largerLength(const string&, const string&);
// depending on the value of its string parameter,
// getFcn returns a pointer to sumLength or to largerLength
decltype(sumLength) *getFcn(const string &);
```

