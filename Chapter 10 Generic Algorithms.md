# 10.1 Overview
標準函式庫的容器只定義了少許的操作，卻定義了一系列*跨容器*(稱為generic)的演算法以供使用，大部分的泛型演算法(**generic algorithm**)都定義在`algorithm`標頭檔，部分數值演算法(numeric algorithm)定義在`numeric`標頭檔。總體來說，泛型演算法並不會直接作用於容器上，而是針對iterator進行操作(也因此橫跨了不同種類的容器)。
>**Example**  
STL的`find`演算法讓我們能某個值是否存在於容器中:
```c++
int val = 42; //想查找的值
// result will denote the element we want if it's in vec, or vec.cend() if not
auto result = find(vec.cbegin(), vec.cend(), val); //前兩個參數代表搜尋範圍
// report the result
cout << "The value " << val
     << (result == vec.cend()
           ? " is not present" : " is present") << endl;
```
上面的`find`會回傳一個iterator標示第一個擁有該值的元素，如果沒有符合的元素的話則回傳參數的第二個iterator，代表搜尋失敗，由於`find`作用在iterator上，我們也可以用在其他不同的容器上。
>**Example**  
假設`v`為一個元素為`string`的`list`:
```c++
string val = "a value";  // value we'll look for
// this call to find looks through string elements in a list
auto result = find(1st.cbegin(), 1st.cend(), val);
```
而由於指標在內建陣列的行為就跟iterator一樣，我們可以用`find`來查找內建型陣列:
```c++
int ia[] = {27, 210, 12, 47, 109, 83};
int val = 83;
int* result = find(begin(ia), end(ia), val); //begin與end函式見3.5.3
```
此外，我們不一定要傳遞整個容器的範圍給find，也可以查找某個子範圍，但要注意第二個iterator標示的是該範圍最後一個元素的下一個元素(one past the last element of the subrange)。
>**Example**  
以下的`find`會查找的元素為`ia[1]`,`ia[2]`以及`i[3]`:
```c++
// search the elements starting from ia[1] up to but not including ia[4]
auto result = find(ia + 1, ia + 4, val);
```
泛型算法雖然*不依賴容器種類*，卻會*依賴元素的型別*，舉上面例子來說，元素型別必須定義`==`運算子才能使用`find`。
>**KEY CONCEPT: ALGORITHMS NEVER EXECUTE CONTAINER OPERATIONS**  
泛型演算法只會作用在iterator上(包括iterator的operation)，並不會使用容器本身的operation，這導致了演算法可能會改變元素的值，但並不會改變該容器的大小，也就是直接插入或移除元素，在10.4.1會看到某種特殊的iterator會有增加容器元素的效應，然而演算法本身永遠不會這麼做。

## 10.2 A First Look at the Algorithms
絕大部分的演算法作用在一個範圍的元素上，該範圍稱作input range，接收input range的演算法會使用頭兩個參數來標定該範圍，這兩個參數為iterator，標記的是該範圍的(1)第一個元素以及(2)最後一個元素的下一個元素。

### 10.2.1 Read-Only Algorithms
此種函式不會改寫元素的值，例如`find`, `count`函式等。
>**Example**  
`accumulate`函式有三個參數，前兩個標記範圍，第三個則為總和的初值:
```c++
// sum the elements in vec starting the summation with the value 0
int sum = accumulate(vec.cbegin(), vec.cend(), 0); //sum = 元素總和 + 0
```

#### Algorithms and Element Types
使用`accumulate`要注意的一點是元素的型別必須能跟總和(第三個參數)的型別進行加法。
>**Example**  
假設`v`是一個元素為`string`的`vector`，則我們可以這樣把`vector`的`string`元素連接起來:
```c++
string sum = accumulate(v.cbegin(), v.cend(), string(""));
```
然而以下直接回傳字串字面值的呼叫方式卻會導致編譯錯誤:
```c++
// error: no + on const char*
string sum = accumulate(v.cbegin(), v.cend(), "");
```
造成錯誤的原因是如果我們傳入字面值字串，則總和的型別是`const char*`，然而`const char*`並沒有定義`+`運算子，因而造成編譯錯誤。
>**Best Practice**  
通常最好用`cbeging()`跟`cend()`來呼叫只讀不寫元素的演算法，然而當我們想要用演算法回傳的iterator來改變元素值，就必須傳入`beging()`跟`end()`。

#### Algorithms That Operate on Two Sequences
`equal`函式可以判斷兩組序列是(回傳`true`)否(回傳`false`)為相同值，他有三個iterator參數，前兩個標記第一個序列的範圍，第三個參數則標記第二個序列的第一個元素:
```c++
// roster2 should have at least as many elements as roster1
equal(roster1.cbegin(), roster1.cend(), roster2.cbegin());
```
由於`equal`只作用在iterator上，我們可以用它來比較*不同容器*中的序列，並且兩序列元素的型別也不必相同，只要確保能夠使用`==`比較即可，但要注意的是它假定第二個序列的數量和第一個序列的數量是一致的。

### 10.2.2 Algorithms That Write Container Elements
有些演算法會賦予元素新值，當我們給定這些演些法覆寫的元素數量時，必須確保要覆寫的序列的元素數是足夠的，如果沒有則可能會造成嚴重錯誤。有些演算法則是對input range內的元素進行覆寫，這種演算法就不會有上述的潛在風險。
>**Example**  
`fill`演算法接收一對標定範圍的iterator以及元素值，並把該值賦值給該範圍內的元素:
```c++
fill(vec.begin(), vec.end(), 0);  //把每個元素重設為0
// set a subsequence of the container to 10
fill(vec.begin(), vec.begin() + vec.size()/2, 10);
```

#### Algorithms Do Not Check Write Operations
有些演算法只接收單一iterator來標示目的地，表示開始進行賦值的元素。
>**Example**  
`fill_n`函式的參數為一個iterator(目的地)、一個數量(賦值個數)以及一個元素值(賦給元素的值)，語法如下:
```c++
fill_n(dest, n, val)
```
`fill_n`假定了從`dest`開始至少還有n個元素，我們可以使用`fill_n`來替`vector`的元素賦予新值:
```c++
vector<int> vec;  // empty vector
// use vec giving it various values
fill_n(vec.begin(), vec.size(), 0); // reset all the elements of vec to 0
```
常見的錯誤是呼叫`fill_n`或類似演算法用於空的容器上:
```c++
vector<int> vec;  // empty vector
// disaster: attempts to write to ten (nonexistent) elements in vec
fill_n(vec.begin(), 10, 0);
```

#### Introducing back_inserter
一個確保有足夠元素提供給演算法的方法是使用**insert iterator**(更詳細的介紹在10.4.1)，它是一個可以增加容器元素的iterator，在這裡我們先使用定義於iterator標頭檔的`back_iterator`於改寫容器的演算法中，它接收一個對容器的reference並回傳一個綁定於該容器的insert_iterator，透過該iterator賦值它會呼叫`push_back`來新增擁有該值的元素進該容器。
>**Example**  
```c++
vector<int> vec; // empty vector
auto it = back_inserter(vec);  // assigning through it adds elements to vec
*it = 42; // vec now has one element with value 42
```
我們常用`back_insert`來產生當成演算法目的地的iterator:
```c++
vector<int> vec; // empty vector
// ok: back_inserter creates an insert iterator that adds elements to vec
fill_n(back_inserter(vec), 10, 0);  // appends ten elements to vec
```
上面的`fill_n`會透過insert iterator進行賦值，間接呼叫了`vec`的`push_back`，最後，`fill_n`會增加10個值為`0`的元素到`vec`的尾端。

#### Copy Algorithms
`copy`演算法接收三個iterator，前兩個標記了複製的範圍，第三個則標記目的容器的起始位置，然後把input range裡面的元素複製到目的容器，但要注意目的容器的大小是否恰當。
>**Example**  
我們可以用copy來複製一個內見型別的陣列:
```c++
int a1[] = {0,1,2,3,4,5,6,7,8,9};
int a2[sizeof(a1)/sizeof(*a1)];  // a2 has the same size as a1(見4.9)
// ret points just past the last element copied into a2
auto ret = copy(begin(a1), end(a1), a2);  // copy a1 into a2
```
在此例，`ret`會指向最後一個被複製進`v2`之元素的下一個位置。

許多演算法提供了被稱作”copying”的版本，這些演算法會計算元素的新值，然而不會放回原本輸入的序列，而是創造一個新的序列來放置結果。
>**Example**  
`replace`函式可以把序列內有特定值的元素改成另一個值:
```c++
//把所有值為0的元素替換成42 
replace(ilst.begin(), ilst.end(), 0, 42);
```
當我們想要原本的序列保持不變，則可以呼叫`replace_copy`，它接收了第三個iterator來標明放置被改變的序列結果:
```c++
// use back_inserter to grow destination as needed
replace_copy(ilst.cbegin(), ilst.cend(),
             back_inserter(ivec), 0, 42);
```
呼叫`replace_copy`後，`ilst`不會變動，而`ivec`則會擁有`ilst`的全部內容，只是原先值為0的元素通通變成`42`。

### 10.2.3 Algorithms That Recorder Container Elements
`sort`型別使用元素的`<`運算子將容器做排序。
>**Example**  
當我們想要依序列出文本中出現的單字時，可先把每個單字存入`vector`中，再利用以下程式碼:
```c++
void elimDups(vector<string> &words)
{
    // sort words alphabetically so we can find the duplicates
    sort(words.begin(), words.end());
    // unique reorders the input range so that each word appears once in the
    // front portion of the range and returns an iterator one past the unique range
    auto end_unique = unique(words.begin(), words.end());
    // erase uses a vector operation to remove the nonunique elements
    words.erase(end_unique, words.end());
}
```

