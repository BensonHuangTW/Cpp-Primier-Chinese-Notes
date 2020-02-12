# Chapter 3 Strings, Vectors, and Arrays
## 3.2 Library string Type
本章節範例皆假設已有以下程式碼:
``` c++
#include <strung>
using std::string
```

### 3.2.2 Operations on strings
**Table 3.2 String Operations**  
![image](https://user-images.githubusercontent.com/55428505/66101454-7d0e4400-e5e1-11e9-9c33-b47d3f1dc919.png)
#### Reading and Writing strings
可用`iostream`函式庫來讀或寫`string`:
``` c++
// Note: #include and using declarations must be added to compile this code
int main()
{
    string s;          // empty string
    cin >> s;          // read a whitespace-separated string into s
    cout << s << endl; // write s to the output
    return 0;
}
```
`string`輸入運算子(`is>>s`)會*忽略任何前方的whitespace*(e.g.空白、換行、tabs)並且開始讀取字元*直到遇見下一個whitespace*，例如如果我們輸入  *Hello World!* (前方及後方都有space)，則輸出會是*Hello*(沒有其他space)。就跟內建型別的輸入與輸出一樣，string operators回傳它左側的運算元作為結果，因此可以連續的讀或寫:
``` c++
string s1, s2;
cin >> s1 >> s2; // read first input into s1, second into s2
cout << s1 << s2 << endl; // write both strings
```
如果我們的輸入還是   *Hello World!*   ，則輸出會是”*HelloWorld!*”。

#### Reading an Unknown Number of strings
如果不知道輸入的`string`數量，可以這樣寫:
``` c++
int main()
{
    string word;
    while (cin >> word)       // read until end-of-file
        cout << word << endl; // write each word followed by a new line
    return 0;
}
```
上面的`while`會在每次讀取完成後測試括號裡面的stream，直到遇到*檔案結束(end-of-file)或是無效輸出*才會跳離`while`。

#### Using getline to Read an Entire Line
有時候我們不想忽略輸入中的whitespace，這種情況下我們可以使用`getline`代替`>>`運算元，該函式需傳入一個input stream以及一個`string`，它會讀取該stream*直到讀進第一個換行字元*，並把結果(*不*包含換行字元)存進該`string`，只要看到第一個換行字元它就會停止讀取並回傳結果，如果輸入的第一個字元就是換行字元，則該`string`為空，`getline`會回傳它自己的isream引數，也就是說我們可以同上面把它當成測試條件:
``` c++
int main()
{
    string line;
    // read input a line at a time until end-of-file
    while (getline(cin, line))
        cout << line << endl;  //因為line不會有換行字元，因此自己加上endl
return 0;
}
```

## 3.3 Library vector Type
`vector`是一種**class template**，C++還定義了function template。template本身並非class或函式，它可以想成是讓編譯器創造class或是函式所遵循的指令，這個過程稱作**instantiation**，當使用模板時我們必須*提供額外的資訊*來產稱特定型別的class或函式，以`vector`來說就是它所持有物件的型別:
``` c++
vector<int> ivec;             // ivec holds objects of type int
vector<Sales_item> Sales_vec; // holds Sales_items
```

>**Note**  
`vector`不是一種型別，而是模板，從`vector`產生的型別*一定要包含element的型別*。
由於reference不是物件，因此沒有`vector` of reference，但我們可以有element本身為`vector`的`vector`:
``` c++
vector<vector<string>> file;  // vector whose elements are vectors
```
>**WARNING**  
有些編譯器要求element為`vector`的`vector`在宣告時必須在`>`之間*加上空格*(較老式的宣告):
``` c++
vector<vector<int>  >
```

### 3.3.1 Defining and Initializing vectors
**Table 3.4: Ways to Initialize a vector**  
![image](https://user-images.githubusercontent.com/55428505/66102606-7681cb80-e5e5-11e9-9137-3c5437734ea6.png)  
`vector`的默認初始化會產生一個空的`vector`:
``` c++
vector<string> svec; // default initialization; svec has no elements
```
我們也可以在初始化`vector`時提供element的初始值，當我們複製`vector`時，新`vector`的每個element都是原`vector`(兩者型別要一樣)之element的一份拷貝:
``` c++
vector<int> ivec;             // initially empty
// give ivec some values
vector<int> ivec2(ivec);      // copy elements of ivec into ivec2
vector<int> ivec3 = ivec;     // copy elements of ivec into ivec3
vector<string> svec(ivec2);   // error: svec holds strings, not ints
```
#### List Initialization a vector
新標準下可以使用包含*0*至數個element初始值的花括號來list initialize一個`vector`:
``` c++
vector<string> articles = {"a", "an", "the"};
```
但我們不能用`()`來這麼做:
``` c++
vector<string> v2("a", "an", "the");  // error
```
#### Creating a Specified Number of Elements
可以提供`vector`一個數量以及element value來一次創造多個相同的element:
``` c++
vector<int> ivec(10, -1);      // ten int elements, each initialized to -1
vector<string> svec(10, "hi!"); // ten strings; each element is "hi!"
```

#### Value Initialization
我們常常只會提供大小而忽略值，在這種情況中函式庫會創造**value-initialized element initializer**，這種由函式庫所產生的值被用來初始化容器(container)中的每個element，其值取決於`vector`所儲存的element型別，如果是內建型別(例如`int`)，則element initializer的值會是`0`，假如是class型別(例如`string`)，則為它自己的default initialization:
``` c++
vector<int> ivec(10);    // ten elements, each initialized to 0
vector<string> svec(10); // ten elements, each an empty string
```
這種形式的初始化有兩種限制:  
(1)	如果`vector`的element*無法*被默認初始化，則*必須提供initial element value*。  
(2)	如果未提供初始值，必須使用direct form of initialization:  
``` c++
vector<int> vi = 10;   // error: must use direct initialization to supply a size
```
#### List Initializer or Element Count?
初始化規則:  
(1)	當我們使用`()`提供值，是在用這些值來*建構*(construct)物件(用來呼叫constructor，詳見第7章)。  
(2)	當我們使用`{}`代表的是如果可能的話，我們會list initialize該物件，然而當沒辦法list initialize時，則裡面的值會用來建構該物件。  
>**Example**  
``` c++
vector<int> v1(10);    // v1有10個值為0的element
vector<int> v2{10};    // v2 有1個值為10的element
vector<int> v3(10, 1); // v3有10個值為1的element
vector<int> v4{10, 1}; // v4有兩個值分別為10跟1的element
vector<string> v5{"hi"}; // list initialization: v5 has one element
vector<string> v6("hi");  // error: can't construct a vector from a string literal
vector<string> v7{10}; // v7 has ten default-initialized elements
				  // 因為無法list initialize，故啟用constructor
vector<string> v8{10, "hi"}; // v8 has ten elements with value "hi"
```
### 3.2.2 Adding Elements to a vector
使用`push_back`來為`vector`添加元素(at runtime):
``` c++
// read words from the standard input and store them as elements in a vector
string word;
vector<string> text;       // empty vector
while (cin >> word) {
    text.push_back(word);  // append word to text
}
```
### 3.3.3 Other vector Operations ###
**Table 3.5: vector Operations**  
![image](https://user-images.githubusercontent.com/55428505/66103126-53f0b200-e5e7-11e9-9860-f80e0ad50a8a.png)

#### Subscripting Does Not Add Elements(不能用下標增加element)
下面程式碼嘗試為`ivec`增加10個element:
``` c++
vector<int> ivec;   // empty vector
for (decltype(ivec.size()) ix = 0; ix != 10; ++ix)
    ivec[ix] = ix;  // disaster: ivec has no elements
```
這會造成錯誤!因為ivec為空的vector，它根本沒有element能夠subscript!，正確的添加方式是使用push_back:
```
for (decltype(ivec.size()) ix = 0; ix != 10; ++ix)
    ivec.push_back(ix);  // ok: adds a new element with value ix
```
>**WARNING**  
使用subscript operator於`vector`或`string`只能用於取*已經存在*的element，*不能*用它來添加element。

## 3.4 Introducing Iterators
迭代器(iterator)是一套可以用來探訪`vector`, `string`…中元素的機制，所有標準函式庫中的容器(library container)都有iterator，但並非都有subscript operator。iterator與pointer類似，提供了對物件的非直接存取(indirect access)，我們可以用iterator來取得某個元素，也有移至另一個元素的操作，並且可能為無效或有效的:標記容器中某個元素或是*最後一個元素的下個位置*(position one past the last element)的iterator是有效的，其他則為*無效*。

#### 3.4.1 Using Iterators
與pointer不同的是iterator不是用address of operator(`&`)來取得的，而是在擁有iterator的型別中會提供回傳iterator的成員，分別命名為`begin`與`end`，成員`begin`回傳了標示*第一個元素*(如果有的話)的iterator:
``` c++
// the compiler determines the type of b and e; see § 2.5.2 (p. 68)
// b denotes the first element and e denotes one past the last element in v
auto b = v.begin(), e = v.end(); // b and e have the same type(其型別是什麼不太重要)
```
上面由`end`回傳的iterator位於尾元素的下一位置(one past the end)，它標記的是一個不存在的尾後(off the end)元素，故又稱作off-the-end iterator或是the end iterator，如果該容器為空，則`begin`與`end`會回傳*相同的iterator*。

#### Iterator Operations
**Table 3.6: Standard Container Iterator Operations**
![image](https://user-images.githubusercontent.com/55428505/66103451-65868980-e5e8-11e9-9b2a-39717d3db0d1.png)  
可以用`==`或`!=`來比較兩個iterator是否相同，當兩個iterator相等時，*必定*為下列兩種情況:  
(1)	它們標示同一個元素。  
(2)	他們都是off-the-end iterator  
就像pointer一樣我們透過`*`來獲得iterator標示的元素，對一個無效或off-the-end iterator使用dereference會導致為定義的結果。
>**Example**  
假設我們想把字串的第一個字元變成大寫，可以寫下面的程式:
``` c++
string s("some string");
if (s.begin() != s.end()) { // make sure s is not empty
    auto it = s.begin();    // it denotes the first character in s
    *it = toupper(*it);     // make that character uppercase
}
```
上面的`if`是為了確保字串不為空，避免我們對一個不存在的元素dereference，如果依照上面的程式運行，則s最後的內容為:
*Some string*

#### Moving Iterators from One Element to Another
Iterator使用increment operator(`++`)來移動至下一個元素。
>**Note**  
由於`end`回傳的iterator標示的並非元素，因此不能對它使用`++`或`*`。

>**Example**  
延續上方的例子，若我們想要把第一個字(word)變成大寫，則可以這樣寫:
``` c++
// process characters in s until we run out of characters or we hit a whitespace
for  (auto it = s.begin(); it != s.end() && !isspace(*it); ++it)
    *it = toupper(*it); // capitalize the current character
```
上面的`!isspace`(`*it`)在遇到空白字元時代表一個word的結束，因此會跳出`for`。

>**KEY CONCEPT: GENERIC PROGRAMMING**  
上面的例子裡面我們看到在`for` *loop裡面我們用的是*`!=`*而非*`<`，這是屬於C++的程式風格，原因是在標準函式庫的容器裡面大多的iterator都沒有`<`運算子，使用`!=`的話就不用擔心了。

#### Iterator Types
我們通常不用知道iterator的準確型別，擁有iterator的函式庫型別定義了兩種名為`iterator`和`const_iterator`的型別來代表實際的iterator型別，舉`vector`與`string`來說:
``` c++
vector<int>::iterator it; 		 // it 可以讀寫vector<int> 的元素
string::iterator it2;    		 // it2可以讀寫string裡的字元
vector<int>::const_iterator it3;  // it3 可以讀取但不能寫入元素
string::const_iterator  it4;     // it4 可以讀取但不能寫入字元
```

#### The begin and end Operations
如果物件為`const`，則`begin`與`end`回傳的型別為`const_iterator`，如果不是則回傳型別為`iterator`:
``` c++
vector<int> v;
const vector<int> cv;
auto it1 = v.begin();  // it1的型別為 vector<int>::iterator
auto it2 = cv.begin(); // it2的型別為 vector<int>::const_iterator
```
當我們只想讀取元素時，最好的方式是使用`const`型別來避免改變到該元素，因此新標準提供了兩個名為`cbegin`與`cend`的函數，*無論該容器是否為*`const`*，都會回傳型別為*`const_iterator`*的iterator*:
``` c++
auto it3 = v.cbegin();  // it3 has type vector<int>::const_iterator
```
#### Combining Dereference and Member Access
當我們想要透過dereference來獲得該元素並使用它的成員時，由於運算優先權的原因，我們*必須*加上括號再用member selector(`.`)來取得該成員。
>**Example**  
假如有一個元素為`string`的`vector`，我們想要用`string`的empty成員函式來檢查`string`是否為空，假設`it`為該`vector`的iterator，則我們應該這樣做:
``` c++
(*it).empty()
```
上面的()是必要的，如果沒加上的話會造成錯誤:
``` c++
*it.empty()   
```
以上語法等價於
``` c++
*(it.empty())
```
但是`it`是iterator，根本就沒有`empty`成員函式，因而造成錯誤，而使用arrow operator(`->`)則可以將deference與member access的動作結合起來，也就是說:
`it->mem`會*等價*於`(*it).mem`。

#### Some vector Operations Invalidate Iterators
詳見9.3.6以及5.4.3。
>**WARNING**  
別在使用iterators的迴圈中為容器添加新元素。

### 3.4.2 Iterator Arithmetic
**Table 3.7. Operations Supported by vector and string Iterators**  
![image](https://user-images.githubusercontent.com/55428505/66104252-8ea81980-e5ea-11e9-97f8-351889ae6db9.png)

# 3.5 Arrays
### 3.5.1 Defining and Initializing Built-in Arrays
array在宣告時，其大小必須是constant expression(見2.4.4):
``` c++
unsigned cnt = 42;          // not a constant expression
constexpr unsigned sz = 42; // constant expression
                       // constexpr see § 2.4.4 (p. 66)
int arr[10];             // array of ten ints
int *parr[sz];           // array of 42 pointers to int
string bad[cnt];         // error: cnt is not a constant expression
string strs[get_size()]; // ok if get_size is constexpr, error otherwise
```
#### Understanding Complicated Array Declarations
array是物件，因此我們可以定義指向array的pointer以及reference to arrays:
``` c++
int *ptrs[10];            //  ptrs is an array of ten pointers to int
int &refs[10] = /* ? */;  //  error: no arrays of references(因為references並非物件)
int (*Parray)[10] = &arr; //  Parray points to an array of ten ints
int (&arrRef)[10] = arr;  //  arrRef refers to an array of ten ints
```
要理解array的宣告的一個較簡單的讀法是從裡到外讀，例如本例中的`Parray`，我們從括號內的`*Parray`得知`Parray`是一個pointer，接著我們往右看觀察出`Parray`指向一個大小為10的array，之後往左看，我們看到該array的元素型別為`int`，最後我們知道`Parray`是一個指向大小為10的整數陣列的pointer。類似地，`arrRef`是一個reference，並且refers to一個大小為10的整數陣列。

### 3.5.2 Accessing the Elements of an Array
當我們要使用變數來當成array的下標時，通常會定義型別為`size_t`的變數，`size_t`定義於`stddef`標頭檔，是一種machine-specific的unsigned type，它保證有足夠的大小來代表任何記憶物件的大小(size)。

### 3.5.3 Pointers and Arrays
#### The Library begin and end Functions
`begin`和`end`是兩個把array當成引數的函數，他們定義於`iterator`標頭檔中。並且分別回傳指向該array的第一個與最後一個元素的pointer。
>**Example**  
``` c++
int ia[] = {0,1,2,3,4,5,6,7,8,9}; // ia is an array of ten ints
int *beg = begin(ia); 		 // pointer to the first element in ia
int *last = end(ia); 			 // pointer one past the last element in ia
```
