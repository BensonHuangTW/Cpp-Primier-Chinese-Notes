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

## 9.3 Sequential Container Operations
以下內容是專屬於sequential container的操作。
**Table 9.5: Operations That Add Elements to a Sequential Container**  
![image](https://github.com/BensonHuangTW/Cpp-Primier-Chinese-Notes/blob/master/images/ch9/9.5.jpg)
 
### 9.3.1 Adding Elements to a Sequential Container
除了`array`之外的函式庫容器都有動態改變容器大小的特性，Table 9.5列出了在sequential container增加元素的操作(對不同的容器效能上會有不同表現)。

>**KEY CONCEPT: CONTAINER ELEMENTS ARE COPIES**  
當我們用一個物件來初始化容器或是把它插入容器時，放進容器的是該引數物件的copy，而非物件本身。

#### Using push_back
除了`forward_list`和`array`之外的sequential container都支援`push_back`，功能為在容器的尾部增加元素:
```c++
// read from standard input, putting each word onto the end of container
string word;
while (cin >> word)
    container.push_back(word);

Using push_front
list, forward_list以及deque支援push_front，功能為在容器的前端增加元素:
list<int> ilist;
// add elements to the start of ilist
for (size_t ix = 0; ix != 4; ++ix)
    ilist.push_front(ix); //loop結束後ilist的元素順序:3,2,1,0
```

#### Adding Elements at a Specified Point in the Container
`vector`, `deque`, `list`和`string`支援`insert`功能(`forward_list`有特殊版本，見9.3.4)，為讓我們插入零個或多個元素到容器的任意位置中，每種版本的insert函式都接收一個iterator作為第一個引數，用來標示插入元素的位置，插入的位置為該iterator標示位置的前一個位置。
>**Example**  
```c++
slist.insert(iter, "Hello!"); 	// insert "Hello!" just before iter
slist.insert(slist.end(),”Bye!”); //Bye會是slist的最後一個元素
```
有些容器不支援`push_front`，但我們可以用insert來把元素放到該容器的前端:
```c++
vector<string> svec;
list<string> slist;
// equivalent to calling slist.push_front("Hello!");
slist.insert(slist.begin(), "Hello!");
// no push_front on vector but we can insert before begin()
// warning: inserting anywhere but at the end of a vector might be slow
svec.insert(svec.begin(), "Hello!");
```
>**WARNING**  
雖然可以在`vector`, `deque`和`string`的任何地方插入元素，但這麼做可能會很吃效能。

#### Inserting a Range of Elements
另一種版本的`insert`可以指定插入同一個元素的數量:
```c++
svec.insert(svec.end(), 10, "Anna"); //在尾部連續插入10個Anna
```
也可以從別的容器的一部分複製元素近來，只要額外加上一對iterator來標示複製的範圍:
```c++
vector<string> v = {"quasi", "simba", "frollo", "scar"};
//在slist的前端插入v的末兩個元素
slist.insert(slist.begin(), v.end() - 2, v.end());
```
或者直接插入braced list的一串元素:
```c++
slist.insert(slist.end(), {"these", "words", "will",
                           "go", "at", "the", "end"});
```
但不能再插入容器的同時又丟入一對該容器的iterator:
```c++
// run-time error: iterators denoting the range to copy from
// must not refer to the same container as the one we are changing
slist.insert(slist.begin(), slist.begin(), slist.end());
```
新版本的C++中，上面三種版本的insert都會回傳一個標示第一個被插入元素的iterator。
>**Example**  
```c++
list<string> 1st;
auto iter = 1st.begin();
while (cin >> word)
   iter = 1st.insert(iter, word); //在此程式等同於呼叫 push_front
```

#### Using the Emplace Operations
新標準C++引入了三個新成員:`emplace_front`, `emplace`, `emplace_back`函式(效果對應於`push_front`, `insert`, `push_back`)，它們的功能是直接在容器管理的空間中直接創建物件，而非只是複製元素，當我們要呼叫`emplace`成員時，必須依照元素的constructor需要的引數來傳入適當的引數:
>**Example**  
```c++
// construct a Sales_data object at the end of c
// uses the three-argument Sales_data constructor
c.emplace_back("978-0590353403", 25, 15.99);
// error: there is no version of push_back that takes three arguments
c.push_back("978-0590353403", 25, 15.99);
// ok: we create a temporary Sales_data object to pass to push_back
c.push_back(Sales_data("978-0590353403", 25, 15.99));
// iter refers to an element in c, which holds Sales_data elements
c.emplace_back(); // uses the Sales_data default constructor
c.emplace(iter, "999-999999999"); // uses Sales_data(string)
// uses the Sales_data constructor that takes an ISBN, a count, and a price
c.emplace_front("978-0590353403", 25, 15.99);
```
### 9.3.2 Accessing Elements
**Table 9.6: Operations to Access Elements in a Sequential Container**  
![image](https://github.com/BensonHuangTW/Cpp-Primier-Chinese-Notes/blob/master/images/ch9/9.6.jpg)
如果容器為空，則使用access operation會導致未定義的結果。
`front`:回傳對第一個元素的reference，所有容器皆能使用。
`back`:回傳對最後一個元素的reference，除了forward_list以外的容器都能使用。
>**Example**  
```c++
// 在呼叫back或front前要先檢查容器中是否有元素
if (!c.empty()) {
    // val 和 val2都是c中第一個元素的copy而非reference(原因見2.5.2)
    auto val = *c.begin(), val2 = c.front();
    // val3 和val4 都是c中最後一個元素的copy
    auto last = c.end();
	//last標示的是最後一個元素的下一個位置，要先--才能獲得最後一個元素
    auto val3 = *(--last); // can't decrement forward_list iterators
    auto val4 = c.back();  // not supported by forward_list
}
```
#### The Access Members Return Reference
`front`, `back`, `subscript`以及`at`回傳的是reference，如果該容器為`const`物件則會回傳reference to const，而當我們用`auto`去儲存這些函式的回傳結果時記得要定義成reference type才會取得該元素的reference:
```c++
if (!c.empty()) {
    c.front()  = 42;      // assigns 42 to the first element in c
    auto &v =  c.back();  // get a reference to the last element
    v = 1024;             // changes the element in c
    auto v2 =  c.back();  // v2 is not a reference; it's a copy of c.back()
    v2 = 0;               // no change to the element in c
}
```

#### Subscripting and Safe Random Access
`at(n)`與subscript operator(`string`, `vector`, `deque`和`array`支援)不同的地方是它在index超過容器的合法範圍時會丟出`out_of_range`異常，後者則不會:
```c++
vector<string> svec; // empty vector
cout << svec[0];     // run-time error: there are no elements in svec!
cout << svec.at(0);  // throws an out_of_range exception
```
### 9.3.3 Erasing Elements
#### The pop_front and pop_back Members
這兩個操作回傳的是`void`，因此如果要保留該元素的值之前必須在`pop`前就存好:
```c++
while (!ilist.empty()) {
    process(ilist.front()); // do something with the current top of ilist
    ilist.pop_front();      // done; remove the first element
}
```
#### Removing an Element from within the Container
`erase`成員負責移除容器中特定位置的元素，有兩種版本:  
(1)	傳入單一iterator，刪除該iterator所指元素。  
(2)	傳入一對iterator，標示想要刪除的元素範圍。  
以上兩種`erase`都會回傳一個指向最後一個被刪除元素下一個位置的iterator，也就是說如果j是i的下一個元素，則`erase(i)`會回傳一個指向j的iterator。
>**Example**  
```c++
刪除list中的奇數元素，則程式碼如下:
list<int> lst = {0,1,2,3,4,5,6,7,8,9};
auto it = lst.begin();
while (it != lst.end())
    if (*it % 2)             // if the element is odd
        it = lst.erase(it);  // erase this element
    else
        ++it;
```
**Table 9.7: erase Operations on Sequential Containers**
 
我們不能對空容器做Table 9.7中的操作。
>**Warning**  
Table 9.7中的成員並不會檢查傳入它的引數是否合法，我們必須自己確認。

#### Removing Multiple Elements
傳入一對iterator版本的`erase`讓我們刪除給定範圍的元素:
```c++
// delete the range of elements between two iterators
// returns an iterator to the element just after the last removed element
elem1 = slist.erase(elem1, elem2); // after the call elem1 == elem2
```
上面程式碼中，`elem1`指向我們想刪除的第一個元素，而`elem2`是指向我們想刪除之最後一個元素的後一位。如果想刪除所有的元素，可以使用`clear`或是傳入`begin`與`end`到`erase`中:
```c++
slist.clear(); // delete all the elements within the container
slist.erase(slist.begin(), slist.end()); // 與上方等價
```
### 9.3.4 Specialized forward_list Operations
**Table 9.8: Operations to Insert or Remove Elements in a forward_list**  
![image](https://github.com/BensonHuangTW/Cpp-Primier-Chinese-Notes/blob/master/images/ch9/9.8.jpg)

`forward_list`有其特有的插入及移除元素的版本，原因解釋如下:  
 elem1 → elem2 → elem3 → elem4  
若我們要移除上圖中的elem3，則必改變elem2使之指向elem4:
  elem1 → elem2 → elem4  
然而由於`forward_list`是單向串列，很難找尋elem3的前一個元素，因此對於`forward_list`來說，插入與移除的動作是針對給定元素的下一個元素做改變，也因此`forward_list`沒有`insert`, `emplace`, 以及`erase`操作，而是用`insert_after`, `emplace_after`以及`erase_after`來取代，舉例來說，上圖就是呼叫`erase`來對elem2的下一個元素(elem3)做移除。為了可以對串列的第一個元素進行操作，`forward_list`另外定義了`before_begin`，回傳一個off-the-beginning iterator(指向第一個元素前的一個不存在的元素)，讓我們可以對第一個元素進行以上操作。
>**Example**  
舉例來說，若要移除`forward_list`中的奇數元素，可以利用追蹤一對iterator來達成(可與9.3.3的例子比較):  
```c++
forward_list<int> flst = {0,1,2,3,4,5,6,7,8,9};
auto prev = flst.before_begin(); // denotes element "off the start" of flst
auto curr = flst.begin();        // denotes the first element in flst
while (curr != flst.end())  {        //仍有元素等待被處理
    if (*curr % 2)                     //該元素為奇數(case(a))
        curr = flst.erase_after(prev); // erase it and move curr
    else {					//case(b)
        prev = curr;        // move the iterators to denote the next
        ++curr;           // element and one before the next element
    }
}
```
>**程式說明**  
(1)curr標記的是我們正在檢查的元素，prev則標記它的上一個元素。  
(2)由於首先檢查的是第一個元素，因此用befor_beging來初始化prev。  
(3)while loop有兩種case:  
&nbsp;(a)	當我們找到奇數元素時，我們把prev傳給erase_after，因此被刪除的是curr標示的元素，此時把curr設為指向被刪除元素的下一個元素(由erase_after回傳)，繼續檢查下一個元素。  
&nbsp;(b)	若不是奇數元素，則兩個iterator都向下移一位。  

## 9.4 How a vector Grows
為了達成random access功能，`vector`的元素必須以相鄰的方式儲存，所以當該連續空間不夠容納新元素時，必須將全部原本的元素都搬移到新的足夠大空間去，並釋放舊空間，這會造成效能的低落，為了解決此問題，vector採取了預留空間的策略減少記憶體的重新配置，使得效能大幅提高。

#### Members to Manage Capacity
**Table 9.10. Container Size Management**  
![image](https://github.com/BensonHuangTW/Cpp-Primier-Chinese-Notes/blob/master/images/ch9/9.10.jpg)  

我們可以利用Table 9.10的操作來對上述的配置策略做調整。`capacity`告訴我們容器在被迫配置新空間前所能容納的元素數量，而`reverse`則讓我們能告訴容器它準備要擁有多少元素，而傳入`reverse`的數必須大於`vector`當前的容量才會改變它的容量，也就是說`reverse`絕對*不會縮小*該容器所使用的空間。
>**Note**  
`reserve`並不會改變容器內元素的數量，它只會影響`vector`預先配置的記憶體空間多寡。

C++11後，可`用shrink_to_fit`對`deque`,`vector`或`string`發出釋放不必要記憶體空間的請求，然而它的實作卻允許*可以忽略這個請求*。

#### capacity and size
示意圖:  
![image](https://github.com/BensonHuangTW/Cpp-Primier-Chinese-Notes/blob/master/images/ch9/9.4-1.jpg)  
事實上，只要沒有出現超出`vector`容量的操作，`vector`必定*不會*重新配置它的元素。每種對`vector`的實作都要保證`push_back`進n個元素進初始為空`vector`的時間複雜度為O(n)。
>**Note**  
`vector`的實作可以自由制定自已的配置策略，但在被強迫的情況之外的話，它絕對不能配置新的記憶體空間。

## 9.5 Additional string Operations
`string`提供了六種不同的搜尋函式，每種都有四個重載版本，Table 9.14列出了這些成員函式以及他們的參數形式:
**Table 9.14. string Search Operations**  
 ![image](https://github.com/BensonHuangTW/Cpp-Primier-Chinese-Notes/blob/master/images/ch9/9.14.jpg)  
每種搜尋回傳的型別為`string::size_type`，代表的是符合匹配結果處的index，如果沒有匹配成功的話，則會回傳名為`string::npos`的`static`(見7.6)成員，函式庫將它定義為`const string::size_type`型別，初始值為`-1`，但由於`string::size_type`是`unsigned`型別，因此`npos`會等於所有`string`可以有的最大大小(見2.1.2)。
>**WARNING**  
由於`string`的搜尋函式傳回的是`string::size_type`型別，是`unsigned`型別，也因此使用`int`或其它有號型別接收這些函式的回傳結果是不好的做法。

`find`函式回傳第一個與引數匹配的index，如果沒有匹配則回傳`npos`
>**Example**  
```c++
string name("AnnaBelle");
auto pos1 = name.find("Anna"); // pos1 == 0
string的搜尋是有大小寫區別的:
string lowercase("annabelle");
pos1 = lowercase.find("Anna");   // pos1 == npos
```
`find_first_of`的功能則稍微不同，它尋找的是欲搜尋字串中第一個有在目標字串所包含字元的字元位置。  
>**Example**  
```c++
string numbers("0123456789"), name("r2d2");
// returns 1, i.e., the index of the first digit in name
auto pos = name.find_first_of(numbers);
```
而`find_first_not_of`的功能則與`find_first_of`相反，它尋找的是欲搜尋字串中第一個不包含於目標字串所含字元的位置:  
>**Example**  
```c++
string dept("03714p3");
// returns 5, which is the index to the character 'p'
auto pos = dept.find_first_not_of(numbers);
```

## 9.6 Container Adaptors
概念:  
adaptor在函式庫中是一個廣義的概念，有container、iterator、function adaptors等，基本上來說，它是一套*用某事物模擬另一事物行為的機制*。

container adaptor接收一個已存在的容器型別並使他的行為變成像另一種型別，例如說stack adaptor接收一個sequential container並使它如一個真的stack般運作，下表為所有container adaptors共有的操作以及型別。  
**Table 9.17: Operations and Types Common to the Container Adaptors Type**
 ![image](https://github.com/BensonHuangTW/Cpp-Primier-Chinese-Notes/blob/master/images/ch9/9.17.jpg) 
 
#### Defining an Adaptor
每個adaptor都定義兩種constructor:  
(1)default constructor(A a;):創建一個空物件。  
(2)`A  a(c)`:接收一個容器`c`，並以複製該容器的方式初始化adaptor，舉例來說，假設`deq`是一個`deque<int>`，則我們可以用`deq`來初始化一個新的`stack`:  
```c++
stack<int> stk(deq); //將deq的元素複製進stk
```
默認下，`stack`與`queue`是用`deque`來實作的，`priority_queue`則是用`vector`，我們可以加上想要實作之容器的型別當成第二個引數來創建adaptor:
```c++
// empty stack implemented on top of vector
stack<string, vector<string>> str_stk;
// str_stk2 is implemented on top of vector and initially holds a copy of svec
stack<string, vector<string>> str_stk2(svec);
```
然而adaptor會限制可實做的容器型別，規定如下:  
(1)對所有的adaptor來說以下的容器不得使用:
&nbsp;&nbsp;(a)array
&nbsp;&nbsp;原因:所有的adaptor都要求容器有新增和移除元素的功能。
&nbsp;&nbsp;(b)forward_list
&nbsp;&nbsp;原因: :所有的adaptor都要求能增加、移除、存取容器中最後一個元素。
(5)對於stack來說:除了array與forward_list以外的容器皆可使用。  
(6)對於queue來說:  
&nbsp;&nbsp;(a)list或deque可使用。  
&nbsp;&nbsp;原因:要求容器有push_back, pop_back, back操作。
&nbsp;&nbsp;(b)vector不可使用。  
&nbsp;&nbsp;原因同上。  
(7)對於priority_queue來說:  
&nbsp;&nbsp;(a)vector或deque可使用。  
&nbsp;&nbsp;(b)list不可使用。  

#### Stack Adaptor
**Table 9.18: Stack Operations in Additional to Those in Table 9.17**
![image](https://github.com/BensonHuangTW/Cpp-Primier-Chinese-Notes/blob/master/images/ch9/9.18.jpg) 
>**Example**  
```c++
stack<int> intStack;  // empty stack
// fill up the stack
for (size_t ix = 0; ix != 10; ++ix)
    intStack.push(ix);   // intStackholds 0 ... 9 inclusive
while (!intStack.empty()) {    // while there are still values in intStack
    int value = intStack.top();
    // code that uses value
intStack.pop(); // pop the top element, and repeat
}
```

#### The Queue Adaptors
**Table 9.19: queue, priority_queue Operations in Addition to Table 9.17**
![image](https://github.com/BensonHuangTW/Cpp-Primier-Chinese-Notes/blob/master/images/ch9/9.19.jpg)   
默認下，`priority_queue`的優先權是由`<`作用在元素上而得的，我們在11.2.2會提到覆蓋這個默認的方式。

