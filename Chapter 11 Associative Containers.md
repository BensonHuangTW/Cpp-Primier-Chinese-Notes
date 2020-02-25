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
## 11.3 Operations on Associative Container Iterators
**Table 11.3: Associative Container Additional Type Aliases**  
![image](https://github.com/BensonHuangTW/Cpp-Primier-Chinese-Notes/blob/master/images/ch10/11.3.jpg)  
除了Table 9.2中的型別之外，associative container還定義了上表的型別，代表容器的key跟value型別，對於`set`類型別，`key_type`就是`value_type`，且`mapped_type`(map類才有定義)為`const`的:  
```c++
set<string>::value_type v1;      // v1 is a string
set<string>::key_type v2;        // v2 is a string
map<string, int>::value_type v3; // v3 is a pair<const string, int>
map<string, int>::key_type v4;   // v4 is a string
map<string, int>::mapped_type v5; // v5 is an int
```

### 11.3.1 Associative Container Iterators
對`map`類來說，`value_type`是一個`pair`型別，且它的`first`持有的是`const` key，`second`則持有value:  
```c++
// get an iterator to an element in word_count
auto map_it = word_count.begin();
// *map_it is a reference to a pair<const string, size_t> object
cout << map_it->first;          // prints the key for this element
cout << " " << map_it->second;  // prints the value of the element
map_it->first = "new key";      // error: key is const
++map_it->second;     // ok: we can change the value through an iterator
```

#### Iterators for sets Are const
雖然`set`類型別定義了`iterator`跟`const_iterator`，但這兩種型別都只提供了對元素的唯讀功能，因此我們只能用`set`的iterator來進行讀取而不能改寫元素值:
```c++
set<int> iset = {0,1,2,3,4,5,6,7,8,9};
set<int>::iterator set_it = iset.begin();
if (set_it != iset.end()) {
    *set_it = 42;            // error: keys in a set are read-only
    cout << *set_it << endl;	  // ok: can read the key
}
```
#### Iterating across an Associative Container
`map`跟`set`類型別也提供`begin`跟`end`操作讓我們可以迭帶整個容器。
>**Example**  
11.1的範例中的迴圈也可以這樣寫
```c++
// get an iterator positioned on the first element
auto map_it = word_count.cbegin();
// compare the current iterator to the off-the-end iterator
while (map_it != word_count.cend()) {
    // dereference the iterator to print the element key--value pairs
    cout << map_it->first << " occurs "
         << map_it->second << " times" << endl;
    ++map_it;  // increment the iterator to denote the next element
}
```
>**Note**  
上方的迴圈寫法在sequential associative container會以key的排序順序(升序)來探訪整個容器。

### 11.3.2 Adding Elements
**Table 11.4: Associative Container insert Operations**  
![image](https://github.com/BensonHuangTW/Cpp-Primier-Chinese-Notes/blob/master/images/ch10/11.4.jpg)  
`insert`成員可以增加一個或給定範圍內的元素，在key不能重複的容器中(e.g. `map`與`set`)，插入一個已經存在的key沒有任何作用:
```c++
vector<int> ivec = {2,4,6,8,2,4,6,8};    // ivec has eight elements(重複一次)
set<int> set2;                           // empty set
set2.insert(ivec.cbegin(), ivec.cend()); // set2 has four elements
set2.insert({1,3,5,7,1,3,5,7});      // set2 now has 4+4=8 elements
```
#### Adding Elements to a map
當我們以`map`使用`insert`時，插入的元素型別是`pair`:
```c++
// four ways to add word to word_count
word_count.insert({word, 1});			//新標準下可用
word_count.insert(make_pair(word, 1));
word_count.insert(pair<string, size_t>(word, 1));
word_count.insert(map<string, size_t>::value_type(word, 1));
```
#### Testing the Return from insert
`insert`回傳的型別取決於容器型別以及參數:
對於不能有重複key的容器而言，只增加單一元素版本的`insert`以及`emplace`回傳的是`pair`型別讓我們知道插入的動作是否有成功，該`pair`的`first`為一個指向給定key對應元素的iterator；`second`則是一個`bool`，若為`false`則代表該key已經存在，插入動作沒有效果，若為`true`則代表key本不存在於容器中，元素成功被插入。
>**Example**  
重寫紀錄單字出現次數的程式如下:
```c++
// more verbose way to count number of times each word occurs in the input
map<string, size_t> word_count; // empty map from string to size_t
string word;
while (cin >> word) {
    // inserts an element with key equal to word and value 1;
    // if word is already in word_count, insert does nothing
    auto ret = word_count.insert({word, 1});
    if (!ret.second)         // 代表該單字已經在word_count中
       ++ret.first->second;  // 增加counter數
}
```
#### Unwinding the Syntax
上面範例的最後一行語句等價於下列語句:
```c++
++((ret.first)->second); // equivalent expression
```
解析如下:  
(1)	`ret`持有`insert`回傳的元素，也就是一個`pair`。  
(2)	`ret.first`是該`pair`的`first`成員，是`map`的iterator，指向`map`中對應於給定key的元素。  
(3)	`ret.first->`dereference該iterator來取得`map`的元素，也是一個`pair`。  
(4)	`ret.first->second`為該`map`元素的value。  
(5)	`++ret.first->second`增加了該value。  

#### Adding Elements to multiset or multimap
對於允許重複key的容器，接收單一元素作為參數版本的`insert`回傳的是一個iterator(因為插入一定成功，不再需要`bool`)，指向被插入的元素。
>**Example**  
```c++
multimap<string, string> authors;
// adds the first element with the key Barth, John
authors.insert({"Barth, John", "Sot-Weed Factor"});
// ok: adds the second element with the key Barth, John
authors.insert({"Barth, John", "Lost in the Funhouse"});
```
### 11.3.3 Erasing Elements
**Table 11.5: Removing Elements from an Associative Container**  
![image](https://github.com/BensonHuangTW/Cpp-Primier-Chinese-Notes/blob/master/images/ch10/11.5.jpg)  
除了傳入一個或一對iterator版本的`erase`外(用法見9.3.3)外associative container還提供接收單一`key_type`作為引數的`erase`，它會刪除給定key所對應的所有元素，並且回傳被刪除元素的數量。
>**Example**  
延續之前單字計數器，我們可以在移除特定單字後顯示移除結果
```c++
// erase on a key returns the number of elements removed
if (word_count.erase(removal_word))
     cout << "ok: " << removal_word << " removed\n";
else cout << "oops: " << removal_word << " not found!\n";:
```
### 11.3.4 Subscripting a map
**Table 11.6: Subscript Operation for map and unordered_map**  
![image](https://github.com/BensonHuangTW/Cpp-Primier-Chinese-Notes/blob/master/images/ch10/11.6.jpg)  
只有`map`跟`unorered_map`提供subscript operator以及對應的`at`函式(見9.3.2)。  
理由:對於multi類的容器單一key可能有多個元素，對於`set`來說key就是元素，以上兩種情況皆不適合使用。  
與其他已知的subscript operator類似，`map`的subscript operator接收一個index，也就是key，並前往該key對應的value(獲得value的lvalue，而非`pair`元素)，然而如果該key不存在的話，則會以value initialize創建一個新元素並插入該容器給對應的key。  
>**Example**  
```c++
map <string, size_t> word_count; // empty map
// insert a value-initialized element with key Anna; then assign 1 to its value
word_count["Anna"] = 1;
```
**程式講解**  
(1)	首先會搜尋`word_count`有沒有key值為Anna的元素，此例中沒有。  
(2)	於是創建一個key-value `pair`插入`word_count`中，key為一個值為`Anna`的`const string`，value被value initialize為`0`。  
(3)	該value被賦值成`1`。  

### 11.3.5 Accessing Elements
**Table 11.7: Operations to Find Elements in an Associative Container**  
![image](https://github.com/BensonHuangTW/Cpp-Primier-Chinese-Notes/blob/master/images/ch10/11.7.jpg)  

如果我們只在乎某個特定的元素是否存在，最好使用`find`；`count`則提供了額外的功能:如果元素存在，它還會計算擁有相同key的元素個數:
```c++
set<int> iset = {0,1,2,3,4,5,6,7,8,9};
iset.find(1);   // returns an iterator that refers to the element with key == 1
iset.find(11);  // returns the iterator == iset.end()
iset.count(1);  // returns 1
iset.count(11); // returns 0
```
#### Using find Instead of Subscript for maps
使用subscript的副作用:當該key不存在於`map`裡面時，*它會插入一個帶著該key的元素*，而有時候我們並不想要改變該`map`，這時我們應該使用`find`:  
```c++
if (word_count.find("foobar") == word_count.end())
    cout << "foobar is not in the map" << endl;
```
#### Finding Elements in a multimap or multiset
multimap和multiset的key是可以重複的，當某些元素在裡面重複某個key時，這些元素會是相鄰的。
>**Example**  
當有一個`multimap`存放作者名(key)和書名(value)，而我們想印出某個作者所有的著作時，最顯而易見的方式為利用`find`跟`count`:
```c++
string search_item("Alain de Botton"); // 想要找的作者
auto entries = authors.count(search_item); // 該作者的著作數量(相同key元素量)
auto iter = authors.find(search_item); // 指向第一個含該作者著作元素的iterator
//因為相同key的元素會相鄰，因此用iter拜訪下entries個元素
// loop through the number of entries there are for this author
while(entries) {
    cout << iter->second << endl; // print each title
    ++iter;     // advance to the next title
    --entries;  // keep track of how many we've printed
}
```  
下面還會提供另外兩種寫法。  

#### A Different, Iterator-Oriented Solution
`lower_bound`跟`upper_bound`都接收一個key並回傳一個iterator，如果該key在容器之中，則:  
(1)	`lower_bound`會回傳iterator指向第一個擁有該key之元素實體。  
(2)	`upper_bound`會回傳iterator指向最後一個擁有該key之元素實體的下個位置。  
而如果該key不在容器之中，則`lower_bound`跟`upper_bound`會回傳指向相同位置的iterator，並且在該位置插入該key後並不會違反key的順序，綜上所述，呼叫`lower_bound`跟`upper_bound`於同一個key上會得到一個iterator range來標示擁有該key的元素。  
>**Example**  
延續上一個例子，我們可以把程式改寫成以下:
```c++
// definitions of authors and search_item as above
// beg and end denote the range of elements for this author
for (auto beg = authors.lower_bound(search_item),
          end = authors.upper_bound(search_item);
     beg != end; ++beg)
    cout << beg->second << endl; // print each title
```
### The equal_range Function
呼叫`equal_range`會回傳一個`pair`，它的效用就是把上面`lower_bound`跟`upper_bound`回傳的兩個指標分別放在`pair`的`first`跟`second`。  
>**Example**  
承上個範例，程式也能這樣寫:
```c++
// definitions of authors and search_item as above
// pos holds iterators that denote the range of elements for this key
for (auto pos = authors.equal_range(search_item);
     pos.first != pos.second; ++pos.first)
    cout << pos.first->second << endl; // print each title
```
## 11.4 The Unordered Containers
新標準定義了四種unordered associative container，這些容器使用了雜湊函數(hash function)以及key的`==`運算子，適合用於元素之間沒有特別順序關係或是排序成本太大或不必要時。  
除了管理雜湊的操作之外，unordered associative container也和ordered container一樣提供諸如`find`, `insert`等等的操作，因此使用方式幾乎和前面的方式一樣，但是unordered container 內的元素不會依照順序排列。  
>**Example**  
用`unordered_map`改寫11.1第一個範例的程式:  
```c++
// count occurrences, but the words won't be in alphabetical order
unordered_map<string, size_t> word_count;
string word;
while (cin >> word)
    ++word_count[word]; // fetch and increment the counter for word
for (const auto &w : word_count) // for each element in the map
     // print the results
     cout <<  w.first << " occurs " << w.second
          << ((w.second > 1) ? " times" : " time") << endl;
```  
跟本來程式唯一不同的地方就只是容器的種類而已，如果用原本的輸入的話，則程式結果為:  
```unix
containers. occurs 1 time
use occurs 1 time
can occurs 1 time
examples occurs 1 time
...
```
可以看見輸出並沒有照字母順序排列。  

#### Managing the Buckets
unordered container其實就是一堆bucket(用來存放元素)的集合，若要存取某元素，它的值經過雜湊函數運算(相對快速)後，就會對應到所在的bucket，如果bucket內有太多元素，則查找的效能會明顯下降，unordered container提供了一系列的函式來管理bucket，見下表。  
**Table 11.8. Unordered Container Management Operations**  
![image](https://github.com/BensonHuangTW/Cpp-Primier-Chinese-Notes/blob/master/images/ch10/11.8.jpg)  




