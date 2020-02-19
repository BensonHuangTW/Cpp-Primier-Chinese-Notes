# Chapter 9 Sequential Containers
## 9.1 Overview of the Sequential Containers
**Table 9.1: Sequential Container Types**
![image](https://github.com/BensonHuangTW/Cpp-Primier-Chinese-Notes/blob/master/images/ch9/9.1.jpg)
不同的container會在增刪元素(element)與非順序存取(nonsequential access)間做不同的效能折衷(trade-off)，並且存放元素的策略不同也導致了某些操作效能的不同，舉例來說:  
* `string`與`vector`:  
(1)	以*連續*記憶體存放元素。  
(2)	可以*快速地*用index來計算元素的位址(支援random access)。  
(3)	在中間增加或移除元素的*速度較慢*。  
原因:連續配置的策略導致後面的元素都要跟著移動。  
*	`list`與`forward_list`:  
(1)	可*快速*增加或移除容器中任何一處的元素。  
(2)	*不支援*random access存取元素，只能透過迭代整個容器。  
(3)	記憶體負擔經常*很大*(與`vector`, `deque`, `array`相比)。  
*	`deque`:  
(1)	支援快速的random access。  
(2)	在中間增加或移除元素可能(potentially)很慢。  
(3)	在兩端增加或移除元素的速度快(與`list`與`forward_list`相比)。  
*	`forward_list`:  
(1)	與最佳的手寫singly linked list相當。  
(2)	沒有`size`運算(因為會造成負擔，不滿足(1))，但對其他容器來說`size`運算很快。  
*	`array`:  
(1)	更加安全、容易使用(與內建array相比)。  
(2)	有*固定大小*，因此不支援增加或移除元素或是改變容器大小。  

## 9.2 Container Library Overview
**Table 9.2. Container Operations**
![image](https://github.com/BensonHuangTW/Cpp-Primier-Chinese-Notes/blob/master/images/ch9/9.2.1.jpg)
![image](https://github.com/BensonHuangTW/Cpp-Primier-Chinese-Notes/blob/master/images/ch9/9.2.2.jpg)
容器定義於與他們自己名稱相同的標頭檔中，且他們是class template，因此在初始化時必須提供額外資訊，大部分(但非全部)的容器都必須提供元素的型別:  
```c++
list<Sales_data>   // list that holds Sales_data objects
deque<double>      // deque that holds doubles
```
#### Constraints on Types That a Container Can Hold
我們可以定義元素本身為容器的容器:
```c++
vector<vector<string>> lines;      // vector of vectors
```
>**Note**  
舊式編譯器可能會要求>之間要有空白，以上方為例:
```c++
vector<vector<string>  >
```

雖然容器可以存放幾乎任何型別的元素，但有些容器裡的操作會對元素型別有要求，如果無法符合的話雖然可以定義裝載該型別的容器，卻只能使用那些符合規定的操作。
>**Example**
接收一個大小作為引數的sequential container constructor要求元素的型別要有default constructor，如果元素的型別沒有的話，則雖然可以定義該容器，卻不能使用該種constructor:
```c++
// 假設noDefault 沒有default constructor
vector<noDefault> v1(10, init); // ok: element initializer supplied
vector<noDefault>  v2(10);            // error: must supply an element initializer
```
 **Table 9.3. Defining and Initializing Containers**
 ![image](https://github.com/BensonHuangTW/Cpp-Primier-Chinese-Notes/blob/master/images/ch9/9.3.jpg)
### 9.2.1 Iterators
所有容器都能用Table3.6的操作，除了`forward_list`不能使用decrement operator(--)，且Table3.7只能用於`string`, `vector`, `deque`與`array`的iterator。

#### Iterator Ranges
iterator range由一對iterator(通常稱作`begin`與`end`或是`first`與`last`)來標示容器的範圍，而`end`標示的是one past the last element，而非最後一個元素，用數學符號表示的話就是**left-inclusive interval**:  
[begin, end)  
必須要注意的是`end`可以等於`begin`但不能在`begin`前面。

#### Programming Implications of Using Left-Inclusive Ranges
根據上面的規範可以推導出一個有效的iterator range具有以下的性質:  
(1)	如果`begin`與`end`相等，則range為空。  
(2)	如果`begin`不等於`end`，代表在該range內至少有一個元素，且`begin`代表的是第一個元素。    
(3)	我們可以增加`begin`直到`begin`與`end`相等。

### 9.2.5 Assignment and swap
**Table 9.4. Container Assignment Operations**
![image](https://github.com/BensonHuangTW/Cpp-Primier-Chinese-Notes/blob/master/images/ch9/9.4.jpg)
賦值相關的操作列於上表，賦值運算子會用右側運算子的元素的copy取代掉*整個*左側運算子的元素:
```c++
c1 = c2;      // replace the contents of c1 with a copy of the elements in c2
c1 = {a,b,c}; // after the assignment c1 has size 3(與右邊list內值的數量相同)
```
函式庫`array`型別允許賦值(內建的array則不行)，但左右側的運算元*必須有相同的大小*:
```c++
array<int, 10> a1 = {0,1,2,3,4,5,6,7,8,9};
array<int, 10> a2 = {0}; // elements all have value 0
a1 = a2;  // replaces elements in a1
a2 = {0}; // 錯誤:不能用braced list來賦值array!
```
#### Using assign (Sequential Containers Only)
sequential container定義了名為`assign`的成員函式來讓我們用*不同型別但相容*的容器或是容器的子序列來賦值，它用給定範圍之元素的copy取代整個左側運算元的元素。
> **Example**  
``` c++
//we can use assign to assign a range of char* values from a vector into a list of string:
list<string> names;
vector<const char*> oldstyle;
names = oldstyle;  // error: container types don't match
// ok: can convert from const char*to string
names.assign(oldstyle.cbegin(), oldstyle.cend()); //由iterator標記賦值的range
```
>  **WARNING**
別把正在呼叫`assign`的容器的iterator拿來傳入`assign`(因為該容器既有的元素正在被取代)。

第二種版本的`assign`接收一個整數值和元素值，它把該容器的元素用指定數量及值的元素取代:
```c++
// equivalent to slist1.clear();
// followed by slist1.insert(slist1.begin(), 10, "Hiya!");
list<string> slist1(1);     // one element, which is the empty string
slist1.assign(10, "Hiya!"); // ten elements; each one is Hiya !
```
#### Using swap
swap把兩個相同型別容器的內容互相交換:
```c++
vector<string> svec1(10); // vector with ten elements
vector<string> svec2(24); // vector with 24 elements
swap(svec1, svec2);
```
容器使用`swap`*很快速(常數時間)* ，這是因為只有資料結構被交換，而非元素本身，除了`array`之外(O(n)，因為會交換元素本身)，這代表iterator, reference與pointer在呼叫`swap`之後還是有效(除了`string`會變成無效)，它們還是指向交換前的那個元素，只是跑到了另一個容器而已，而對`array`來說該元素的值會變成對應之交換過後的值。  
新版函式庫還提供了另一個非成員版本的`swap`，見上表，它在泛型程式很重要，統一使用非成員版本的`swap`是個好習慣。

### 9.2.6 Container Size Operations
每種容器都提供三種容器大小相關的操作，只有`forward_list`除外(只有`max_size`與`empty`):  
(1)	`size`:回傳容器中元素的數量。  
(2)	`empty`:若容器大小為零則回傳true，否則回傳false。  
(3)	`max_size`:回傳一個大於或等於該種容器所能持有的元素數量。  

### 9.2.7 Relational Operations
所有的容器接支援equality operator(`==`和`!=`)，除了unordered associative container以外的容器都支援relational operator(`>`, `>=`, `<`, `<=`)，左右兩側的容器以及元素型別一定要相同，也就是說我們不能用`list<int>`或`vector<double>`來跟`vector<int>`做比較，兩容器做比較的規則如下:  
(1)	如果兩容器的元素序列互不為彼此的起始子序列(e.g. {1,3,4}為{1,3,4,5,6}的起始子序列)，則比較結果取決於第一個不相等的元素。  
(2)	如果容器a的元素序列為b的初始子序列，則a<b。  
(3)	如果兩容器的大小與元素一致，則相等，否則必不相等。  
```c++
vector<int> v1 = { 1, 3, 5, 7, 9, 12 };
vector<int> v2 = { 1, 3, 9 };
vector<int> v3 = { 1, 3, 5, 7 };
vector<int> v4 = { 1, 3, 5, 7, 9, 12 };
v1 < v2  // true; v1 and v2 differ at element [2]: v1[2] is less than v2[2]
v1 < v3  // false; all elements are equal, but v3 has fewer of them;
v1 == v4 // true; each element is equal and v1 and v4 have the same size()
v1 == v2 // false; v2 has fewer elements than v1
```
#### Relational Operators Use Their Element’s Relational Operator
equality operator使用的是元素的`==`運算子，而relational operator使用的是元素的`<`運算子。
>**Note**  
只有當元素的型別有定義適當的comparison operator能使用relation operator。

