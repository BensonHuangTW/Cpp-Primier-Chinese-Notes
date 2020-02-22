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
