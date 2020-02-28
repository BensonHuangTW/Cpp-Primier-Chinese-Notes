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
此外，我們不一定要傳遞整個容器的範圍給`find`，也可以查找某個子範圍，但要注意第二個iterator標示的是該範圍最後一個元素的下一個元素(one past the last element of the subrange)。
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

#### Introducing `back_inserter`
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
>**程式說明**  
>假設`words`原本的內容為:  
>|the|quick|red|fox|jumps|over|the|slow|red|turtle|
>|---|-----|---|---|-----|----|---|----|---|------|
>
>(1)	經過`sort`排序後，`words`內容變成:  
>
>|Fox|jumps|over|quick|red|red|slow|the|the|turtle|  
>|---|-----|----|-----|---|---|----|---|---|------|
>
>(2)	`unique`會把input range內相鄰值重複的元素移走，並回傳一個iterator代表最後一個`unique`的元素的下一位置:
>
>|Fox|jumps|over|quick|red|slow|the|turtle|???*(end_unique代表位置)*|???|
>|---|-----|----|-----|---|----|---|------|------------------------|---|
>  
>(在該`end_unique`iteartor後的元素仍然存在，但我們不知道它們的值為何)  
>(3)	最後再用容器本身的把不必要的元素(`end_unique`後)剔除，最後獲得:  
>
>|Fox|jumps|over|quick|red|slow|the|turtle|
>|---|-----|----|-----|---|----|---|------|

## 10.3 Customizing Operations
很多演算法都會比較輸入序列的元素大小，當我們不想使用默認下的`<`或是`==`運算子來進行比較時，函式庫也定義了這些演算法的另一種版本，讓我們能夠提供自己的操作來取代默認的運算子。
### 10.3.1 Passing a Function to an Algorithm
假設我們想印出`vector`經過上一個例子的`elimDups`呼叫後的樣子，但是想以`word`的長度優先作為排序依據，再以字母順序來排序相同長度的`word`，因此我們必須使用另一種重載版本的`sort`，它接收第三個參數，稱作**predicate**。

>**Predicates**  
一個predicate是一個可以被呼叫並且回傳一個可以當成條件式的expression，函式庫演算法可使用的predicate不是unary predicate(單一參數)就是binary predicate(兩個版本)，演算法會以input range中的元素來呼叫給定的predicate，也因此元素型別必須能夠轉換成該predicate的參數型別。
>**Example**  
使用`word`長度來作為排序`vector<string>`的依據:
```c++
// comparison function to be used to sort by word length
bool isShorter(const string &s1, const string &s2)
{
    return s1.size() < s2.size();
}
// sort on word length, shortest to longest
sort(words.begin(), words.end(), isShorter);
```  
#### Sorting Algorithms
stable sort會保持同樣大小的元素在原先的順序。
>**Example**  
接續本節開頭的想法，若我們想以`word`的長度優先作為排序依據，再以字母順序來排序相同長度的`word`，就可以呼叫`stable_sort`來將原本依照字母順序排序的內容進行`word`長度的排序，而不改變同樣長度`word`原先的排序:
```c++
elimDups(words); // put words in alphabetical order and remove duplicates
// resort by length, maintaining alphabetical order among words of the same length
stable_sort(words.begin(), words.end(), isShorter);
for (const auto &s : words)  // no need to copy the strings
    cout << s << " ";  // print each element separated by a space
cout << endl;
如果我們執行上方的程式碼於10.2.3的vector，會得到以下輸出:
fox red the over slow jumps quick turtle
```
### 10.3.2 Lambda Expression
演算法雖然可以利用predicate來放入我們自定義的處理方式，然而卻限制了predicate引數的數量及型別，但有時我們會需要額外的引數來進行處理，要解決這種問題我們可以引入另一種功能，見下方。

#### Introducing Lambdas
我們可以傳入任何種類的callable object給演算法，只要物件或是expression是可以使用call operator的話，就稱之是callable。以下為callable的四種類型:  
(1)	函式。  
(2)	函式指標。  
(3)	重載call operator的class(見14.8)。  
(4)	lambda expression。  
一個lambda expression代表了程式碼中的一個callable單元，可以想成是一個沒有名子的`inline`函式，其格式如下:  
```c++
[capture list]  (parameter list)  -> return type  { function body }
```
capture list :位於包覆該lambda expression函式內的區域變數列表(可為空)。  
parameter list、return type以及function body :與一般的函式相同。  
與一般函式不同的點是，lambda*必須使用trailing return*(見6.3.3)來標明它的回傳型別。    
我們可以忽略參數列表跟回傳型別，但是一定要包含capture list跟function body:  
```c++
auto f = [] { return 42; };
```
在此我們定義了`f`為一個callable object，接收0個參數並回傳`42`，我們以呼叫函式的方式來呼叫一個lambda:
```c++
cout << f() << endl;  // prints 42
```
忽略lambda中的`[]`以及參數列表等同於將它們標示為空，忽略回傳型別的話，則會自動從function body推斷回傳型別。
>**Note**  
如果lambda的function body不是單一的`return`語句的話，也沒指定回傳型別的話，則回傳的是`void`。(如果想要回傳的不是`void`，就必須用trailing return標明回傳型別)

#### Passing Arguments to a Lambda
lambda不能有默認引數，因此呼叫lambda時必須傳入和參數個數相同的引數，一旦參數被初始化，function body也開始執行。
>**Example**  
承10.3.1中的例子，`isShorter`函式的行為可用以下的lambda表達:
```c++
[](const string &a, const string &b)
{ return a.size()<b.size();}
```
空的capture list表示這個lambda不會用到任何它所處函式中的區域變數。我們的`stable_sort`可以利用lambda改寫如下:
```c++
//用size來做為排序依據，但維持相同size字串的原先順序:
Stable_sort(words.begin(), words.end(), 
[](const string&a, const string &b)
{ return a.size() < b.size();});
```
當`stable_sort`需要比對兩個元素時，就會呼叫給定的lambda expression。

#### Using the Capture List
如果lambda在函式內，而且想使用該函式(稱做surrounding function)內的某些區域變數的話，就必須將這些變數納入capture list中才能使用。
>**Example**  
回到本小節最初的問題，作為一個例子，假設我們想要把經過上例程式排序後的序列中大於某個長度的字串都印出來，則可以使用`find_if`演算法，它接收一對標定範圍的iterator，第三個引數則是一個predicate，它會呼叫predicate在每個元素上，並且一旦碰到predicate回傳非0值的第一個元素，`find_if`就會回傳指向該元素的iterator(若都沒有則回傳end iterator)。然而該predicate必須為unary predicate，因此我們不能傳入額外的參數標示任意的篩選長度，這時我們就能利用capture list來導引lambda取得額外需要的變數了，在此例中，lambda會捕捉`sz`這個變數以及一個`string`參數，並拿該`string`的長度和`sz`作比較:
```c++
[sz](const string &a)
    { return a.size() >= sz; };
如果沒有在capture list放入sz，則會導致程式無法編譯:
// error: sz not captured
[](const string &a)
    { return a.size() >= sz; };
```

#### Calling `find_if`
使用上面的lambda，我們可以找到指向第一個長度至少為`sz`字串的iterator(如果沒有的話會回傳`word.end()`的copy):
```c++
// get an iterator to the first element whose size() is >= sz
auto wc = find_if(words.begin(), words.end(),
            [sz](const string &a)
                { return a.size() >= sz; });
```                
我們可以用`find_if`回傳的iterator來計算有多少個元素在該iterator與`words`的尾端之間:
```c++
/ compute the number of elements with size >= sz
auto count = words.end() - wc;
cout << count << " " << make_plural(count, "word", "s") //使用6.3.2中的函式
     << " of length " << sz << " or longer" << endl;
```

#### The `for_each` Algorithm
最後一部分是把`words`中長度大於`sz`的字串印出來，我們使用`for_each`，它接收一個callable object並把每個input range中的元素拿來呼叫它:
```c++
// compute the number of elements with size >= sz
auto count = words.end() - wc;
cout << count << " " << make_plural(count, "word", "s")
     << " of length " << sz << " or longer" << endl;
```     
在此例中，capture list為空，這是因為我們只用它來取得定義在surrounding function中的nonstatic變數，而lambda可以直接使用定義於該surrounding function外部的名稱，因此`cout`(定義於`iostream`標頭檔)是可以直接被使用的。

#### Putting It All Together
綜合以上的功能，我們把它們聚集起來成一個函式:
```c++
void biggies(vector<string> &words,
             vector<string>::size_type sz)
{
        elimDups(words);  // put  words  in  alphabetical  order  and  remove
duplicates
    // sort words by size, but maintain alphabetical order for words of the same size
    stable_sort(words.begin(), words.end(),
                [](const string &a, const string &b)
                  { return a.size() < b.size();});
    // get an iterator to the first element whose size() is >= sz
    auto wc = find_if(words.begin(), words.end(),
                [sz](const string &a)
                    { return a.size() >= sz; });
    // compute the number of elements with size >= sz
    auto count = words.end() - wc;
    cout << count << " " << make_plural(count, "word", "s")
         << " of length " << sz << " or longer" << endl;
    // print words of the given size or longer, each one followed by a space
    for_each(wc, words.end(),
             [](const string &s){cout << s << " ";});
    cout << endl;
}
```

### 10.3.3 Lambda Captures and Returns
當我們定義一個lambda時，編譯器會產生一個新class(未命名)來與該lambda對應(詳見14.8.1)，當我們將某個lambda傳入函式時，我們同時定義了一個新的型別以及一個該型別的物件，使用`auto`的時候也是。默認下，由lambda產生的class會包含data member來對應該lambda捕捉的變數，並且該data member在lambda物件被創建時被初始化。

#### Capture by Value
就和函式參數的傳遞一樣，我們可以用兩種方式來捕捉變數:by value以及by reference。
就像是by value的參數傳遞，必須確保該變數能夠被複製，但和參數不同的是該被捕捉變數的值在該lambda被創建時就被複製了，而不是在lambda被呼叫時:
```c++
void fcn1()
{
    size_t v1 = 42;  // local variable
    // copies v1 into the callable object named f
    auto f = [v1] { return v1; };
    v1 = 0; //不會影響到f被呼叫時v1的值
    auto j = f(); // j is 42; f stored a copy of v1 when we created it
}
```

#### Capture by Reference
我們也可以用by reference的方式捕捉變數(reference capture):
```c++
void fcn2()
{
    size_t v1 = 42;  // local variable
    // the object f2 contains a reference to v1
    auto f2 = [&v1] { return v1; };
    v1 = 0;
    auto j = f2(); // j is 0; f2 refers to v1; it doesn't store it
}
```
reference capture和reference return有相同的問題(見6.3.2)，我們確保注意那個被參考的物件在該lambda在執行的時候還存在，reference capture有時候是必要的，例如`ostream`物件不能被複製，因此要使用reference capture:
```c++
void biggies(vector<string> &words,
             vector<string>::size_type sz,
             ostream &os = cout, char c = ' ')
{
    // code to reorder words as before
    // statement to print count revised to print to os
    for_each(words.begin(), words.end(),
             [&os, c](const string &s) { os << s << c; });
}
```
上面的`c`跟`os`在該lambda被執行的時候都存在，因此沒有問題。
我們也可以從函數回傳一個lambda，該函數可以直接回傳一個callable object或者data member有callable object的class物件，同樣要注意的是該lambda不能有reference captures(因為它會參考該函式內的local object然而在函式終止時會被銷毀，因此會變成是參考一個不存在的物件)。
>**WARNING**  
當我們用by reference的方式捕捉變數時，必須確保該變數在lambda被執行時還是存在的。  
>**ADVICE: KEEP YOUR LAMBDA CAPTUREs SIMPLE**  
當我們捕捉pointer, iterator，或是用by reference來捕捉變數時，必須確保變數在lambda被執行時還是存在的，此外還必須注意另一點:變數的捕捉與lambda的執行可能是有間隔的，必須留意這段間隔中的code可能會改變這些變數的值，造成我們預想之外的效果，綜上所述，我們應該盡可能的減少被捕捉的變數，並且盡量避免捕捉pointer或是reference。

#### Implicit Captures
如果我們想使用surrounding function的區域變數，但又不想要清楚列在capture list內(explicit captures)的話，可以在capture list裡面使用&或=(implicit capture):  
(1)	`&`:告訴編譯器用by reference的方式捕捉變數。  
(2)	`=`:告訴編譯器用by value的方式捕捉變數。  
>**Example**  
```c++
// sz implicitly captured by value
wc = find_if(words.begin(), words.end(),
             [=](const string &s)
                { return s.size() >= sz; });
```
如果我們想要用by value的方式捕捉某些變數，但其它的變數想以by reference方式捕捉，則可以混合implicit capture跟explicit capture:
```c++
void biggies(vector<string> &words,
             vector<string>::size_type sz,
             ostream &os = cout, char c = ' ')
{
    // other processing as before
    // os implicitly captured by reference; c explicitly captured by value
    for_each(words.begin(), words.end(),
             [&, c](const string &s) { os << s << c; });
    // os explicitly captured by reference; c implicitly captured by value
    for_each(words.begin(), words.end(),
             [=, &os](const string &s) { os << s << c; });
}
```
當我們混合implicit capture跟explicit capture時，capture list的第一項一定要是`&`或是`=`，它代表默認下的capturer模式(by reference 或是 by value)，且後方的explicit capture一定要使用另一種捕捉模式(e.g.假如是`&`，則後面都不能加`&`)。  
**Table 10.1: Lambda Capture List**  
![image](https://github.com/BensonHuangTW/Cpp-Primier-Chinese-Notes/blob/master/images/ch10/10.1.jpg)
