# Chapter 11 Associative Containers
**associative container**用於有效率的查找以及取得key，兩種主要的associative container:
* `map`:  
元素為key-value pair，key作為map的index，而value則為key所綁定的資料。  
應用:字典(key為單字，value為解釋與定義)
* `set`:  
元素只有單一key，對於查詢給定的key是否存在很有效率。  
應用:黑名單。

標準函式庫提供了八種associative container，它們之間的差異由三組性質決定:  
(1)	是`map`或是`set`。  
(2)	是single key或允許multiple key。  
(3)	順序或非順序(*使用hash function*)儲存元素。  
定義這些容器的標頭檔如下:  
(1)	`<map>` :定義`map`與`multimap`。  
(2)	`<set>` :定義`set`與`multiset`。  
(3)	`<unordered_map>` :定義`unordered_map`與`unordered_multimap`。  
(4)	`<unordered_set>` :定義`unordered_set`與`unordered_multiset`。  

## 11.1 Using an Associative Container
#### Using a `map`
>**Example**  
經典應用:計算單字出現次數，程式碼如下:
```c++
// count the number of times each word occurs in the input
map<string, size_t> word_count; // empty map from string to size_t
string word;
while (cin >> word)
        ++word_count[word];   // fetch and increment the counter for word
for (const auto &w : word_count) // for each element in the map
    // print the results
    cout <<  w.first << " occurs " << w.second
         << ((w.second > 1) ? " times" : " time") << endl;
```
**程式說明**  
(1)	當定義`map`時，要同時標明`key`跟`value`的型別，在此例中當我們對`word_count`使用下標時是用`string`型別，並且會得到與該`string`對應的`size_t` counter。  
(2)	接著`while`迴圈負責讀取每個單字，並把讀入的每個單字都用於`word_count`的下標，假設此時`word`不在`map`裡面，則subscript operator會創造一個key為`word`的元素，並且對應的值為`0`，之後`++`會增加該value的值。  
(3)	最後range for迴圈會迭代整個`map`，`w`會是一個型別為`pair`的物件(見11.2.3)，`pair`是一種擁有兩個data element的模板型別，而`map`使用的`pair`會有一個`first`成員做為key以及`second`成員對應於其value，所以最後一條語句的效果為印出每個單字以及它的出現次數，輸出的樣子如下:  
```unix
Although occurs 1 time
Before occurs 1 time
an occurs 1 time
and occurs 1 time
 ...
```
#### Using a `set`
>**Example**  
延續上面的範例，如果我們不想計入“the,” “and,” “or,”等單字的話，可以使用`set`來存放這些單字，並只計算不在此`set`裡的單字:  
```c++
// count the number of times each word occurs in the input
map<string, size_t> word_count; // empty map from string to size_t
set<string> exclude = {"The", "But", "And", "Or", "An", "A",
                                              "the", "but", "and", "or", "an",
"a"};
string word;
while (cin >> word)
    // count only words that are not in exclude
    if (exclude.find(word) == exclude.end())
        ++word_count[word];   // fetch and increment the counter for word
```
**程式說明**  
(1)	當定義`set`的時候，要標明element的型別，在此例中為`string`。  
(2)	針對此行進行講解   
```c++
if (exclude.find(word) == exclude.end())
```
當我們呼叫`find`會回傳一個iterator，如果給定的`word`(也就是key)在`set`裡面，則該iterator指向該key；反之如果不在`set`中，則該iterator為off-the-end iterator，因此在此例中如果`word`不在`exclude`中則會更新該`word`的counter值。

## 11.2 Overview of the Associative Containers
associative container不支援sequential-container position-specific operation，例如`push_back`，也不支援接收一個元素及一個count為引數的`insert`，而且associative container的iterator是雙向(**bidirectional iterator**)的(見10.5.1)。

### 11.2.1 Defining an Associative Container
每個associative container都有default constructor來創建空的容器，也可以用別的相同型別容器或是給定範圍的值(只要能轉換成該型別)來初始化，新標準下也能list initialize元素:  
```c++
map<string, size_t> word_count;  // empty
// list initialization
set<string> exclude = {"the", "but", "and", "or", "an", "a",
                                              "The", "But", "And", "Or", "An",
"A"};
// three elements; authors maps last name to first
map<string, string> authors = { {"Joyce", "James"},
                                {"Austen", "Jane"},
								{"Dickens", "Charles"} };
```
當初始化`map`，我們必須同時在花括號內提供一對key(第一個值)跟value(第二個值):
```c++
{key, value}
```

#### Initializing a multimap or multiset
`map`或`set`的key不能重複，然而multimap和multiset沒有這種限制，也就是說對於同一個key來說可以有多個元素，就像字典中的單字對應多個定義。
>**Example**  
下面程式用同一個`vector<int>`來分別初始化`set`跟`multiset`，且`vector`中的
元素為0~9各重複出現一次:
```c++
// define a vector with 20 elements, holding two copies of each number from 0 to 9
vector<int> ivec;
for (vector<int>::size_type i = 0; i != 10; ++i) {
    ivec.push_back(i);
    ivec.push_back(i);  // 同一個數字複製兩次進ivec
}
// iset holds unique elements from ivec; miset holds all 20 elements
set<int> iset(ivec.cbegin(), ivec.cend());
multiset<int> miset(ivec.cbegin(), ivec.cend());
cout << ivec.size() << endl;    // prints 20
cout << iset.size() << endl;    // prints 10，因為不能有重複的key
cout << miset.size() << endl;   // prints 20，可以有重複的key
```

### 11.2.2 Requirements on Key Type
對於ordered container(`map`, `multimap`, `set`以及`multiset` )，key的型別一定要定義能夠比較元素大小的方式，默認下是用`<`來比較。

#### Key Types for Ordered Containers
我們可以提供別的操作來取代<作為比較的依據，但其大小關係必須滿足下列的性質:  
* 若k1<k2，則k2≮k1(asymmetric)。
* 若k1<k2且k2<k3，則k1<k3(transitive)。  
* 若k1≮k2且k2≮k1，則k1與k2為等價的，且等價具遞移性。  
若兩個key為等價的，則容器會把它們視為相等，對應的value只會有一個，使用任一個key都能存取該value。  

#### Using a Comparison Function for the Key Type
如果使用我們自行定義的函式來取代`<`，則定義容器的語法如下:  
```c++
容器種類<key, (value,) 函式的pointer> name;
```
>**Example**  
假設我們要定義元素型別`Sales_data`的`multiset`，並且自行定義比較函式如下:  
```c++
bool  compareIsbn(const  Sales_data  &lhs,  const  Sales_data
&rhs)
{
    return lhs.isbn() < rhs.isbn();
}
```
則在創立容器時，要額外提供指向`compareIsbn`的pointer:
```c++
// bookstore can have several transactions with the same ISBN
// elements in bookstore will be in ISBN order
multiset<Sales_data, decltype(compareIsbn)*> 
    bookstore(compareIsbn);
```
注意的是`*`是*必加*的，但`compareIsbn`前面不加`&`也可以(見6.7)。

### 11.2.3 The pair Type
`pair`為標準函式庫定義於`utility`標頭檔的型別，是一個模板，因此在創建它時要提供兩個型別名稱(可以不同):  
```c++
pair<string, string> anon;       // holds two strings
pair<string, size_t> word_count; // holds a string and an size_t
pair<string, vector<int>> line;  // holds string and vector<int>
```
`pair`的default constructor會value initialize它的data member，因此上面的`word_count`的`size_t`成員值會是`0`，`string`成員的值為空字串。
我們也可以提供initializer給data member:
```
pair<string, string> author{“James”, “Joyce”}; 
//若想用多個引數的constructor或是default constructor，
//則要把引數列在內層{}裡面。
pair的成員為public，名稱為first跟second，可用member access notation存取。
```
#### A Function to Create pair Objects
如果函式要回傳`pair`的話，在新標準下可以使用list initialize來回傳值:  
```c++
pair<string, int>
process(vector<string> &v)
{
     // process v
     if (!v.empty())
         return {v.back(), v.back().size()}; // list initialize(不用標pair型別)
     else
         return pair<string, int>(); // explicitly  constructed  return
value
}
```
不過在舊的標準中我們不能這樣用，可以用下面的方式替代:
```c++
if (!v.empty())
    return pair<string, int>(v.back(), v.back().size());
```
或者也能用函式庫提供的`make_pair`函式來用引數產生適當型別的新`pair`:
```c++
if (!v.empty())
     return make_pair(v.back(), v.back().size());
```
