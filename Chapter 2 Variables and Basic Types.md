# Chapter 2 Variables and Basic Types
## 2.2 Variables
#### Default Initialization
當我們在定義變數的時候沒有提供initializer，則它會被default initialized，default value取決於該變數的型別以及被定義的地方，內建型別物件的值取決於他被定義的地方:

(1)  如果定義於所有函式之外，則會被初始化為0。  
(2)  如果定義於函數之內，則這些變數為**uninitialized**(例外:static變數，見6.1.1)，並且他們的值為未定義的(undefined)，複製或access這種變數是錯誤的。  
大部分的class可以默認初始化，例如如果我們不給string明確的initializer時，會獲得一個空字串，有些class不允許默認初始化，當我們沒在創建它的物件時提供initializer的話編譯器會complain。

### 2.2.2 Variable Declarations and Definitions
為了支援separate compilation，C++將聲明(declaration)與定義(definition)做了區別:  
(1)	**聲明(declaration)**  
使得一個名稱(name)被程式所知道(makes a name known to the program)。  
(2)	**定義(definition)**  
創建該名稱的實體(creates the associated entity)。  
一個變數的聲明標示了該變數的(1)型別(2)名稱，而一個變數的定義就是一種聲明，除了標示型別及名稱外，定義還在記憶體中配置了空間(allocates storage)且可能為該變數提供了初始值。
如果要獲得一個不是定義的聲明，我們加入關鍵字extern並且不提供一個explicit initializer:
``` C++
extern int i;     // declares but does not define i
int j;          // declares and defines j
```
任何包含explicit initializer的聲明即為一個定義:
```C++
extern double pi = 3.1416; // definition
```

上面的語法可行，但是這麼做就蓋掉extern的作用了。但是在函數中這麼做會發生錯誤。  
注意:變數*只能被定義一次*，但是可以被聲明很多次!

## 2.3 Compound Types
廣義上來說，一個宣告的結構為**base type**加上一串**declarator**，每個declarator都為一個變數命名並且給予該變數與base type相關的型別。  
>**Example**
``` C++
int &reval;
```
根據以上的程式碼

base type: ```int```
declarator: ```&reval```

### 2.3.1 References
reference定義了某個物件的另一個名稱:
``` C++
int ival = 1024;
int &refVal = ival;    // refVal refers to (is another name for) ival
int &refVal2;        // error: a reference must be initialized
```
當我們定義一個reference，他會被綁訂(bind)於他的initializer而不是複製其值，而且*沒有*方法能將reference重新綁訂於另一個物件上，因此reference*一定要被初始化*。

>**Note**  
新標準導入了新種類的reference: rvalue reference(見13.6.1)，主要用於class內部，技術上來說當我們提及reference時指的是lvalue reference。

#### A Reference Is an Alias
reference並非物件，僅是某個已存在物件的別名，當reference被定義後，任何對它的操作實際上是*對它所綁定物件的操作*:
``` c++
refVal = 2;      // assigns 2 to the object to which refVal refers, i.e., to ival
int ii = refVal; // same as ii = ival
```
以此類推，當我們使用reference作為initializer時，實際上是在使用它所綁定的物件:
``` c++
// ok: refVal3 is bound to the object to which refVal is bound, i.e., to ival
int &refVal3 = refVal;
// initializes i from the value in the object to which refVal is bound
int i = refVal; // ok: initializes i to the same value as ival
```

#### Reference Definitions
我們可以在單一的定義中定義多個reference，但是每個為reference的identifier前面一定要加上`&`:
``` c++
int i = 1024, i2 = 2048;  // i and i2 are both ints
int &r = i, r2 = i2;      // r is a reference bound to i; r2 is an int(不是reference)
int i3 = 1024, &ri = i3;  // i3 is an int; ri is a reference bound to i3
int &r3 = i3, &r4 = i2;   // both r3 and r4 are references
```

## 2.4 `const` Qualifier
當我們想將使某個變數的值不能被改變，則定義該變數為`const`。  
>**Example**
``` c++
cons tint bufSize = 512;
```
任何試圖對`bufSize`賦值的動作皆為違法:
``` c++
bufSize = 512;  //錯誤，試圖寫入該const物件
```
由於我們不能改變`const`物件的值，它們*一定*要被初始化。

#### Initialization and const 
物件的`const`性質只有在某個操作會改變該物件的值才會發生作用，其餘與一般物件相同(可初始化別人、進行算術運算等等…)。

#### By Default, const Objects Are Local to a File
假設我們定義了一個`const` object如下:
``` c++
const int bufSize = 512;
```
編譯器通常會在編譯過程中把使用到該變量的地方替換成它對應的值。在此例中，編譯器會用`512`這個值來產生code代替`bufSize`的code，因此，編譯器必須能夠看見它的initializer才行，所以在一個被分割成多個檔案的程式中，每個檔案都必須都有對該`const`物件的定義，為了支持這個做法，同時避免對同一個變數的重複定義，默認情況下，`const`物件被設定成只在該檔案內有效，也就是說，若多個檔案中出現了同名的`const`變數，其實就像是在不同的檔案中分別定義了不同的變數一樣。  
>**Example**
``` c++
//在a.h中已經定一了一個const int變數i了
//a.h的程式碼如下
#ifndef A_H
#define A_H
const int i = 0 ;
#endif
```
``` c++
//儘管a.c include了a.h，卻能夠再次定義const int i
//a.cpp程式碼如下
#include <iostream>
#include "a.h"
int main(){
 const int i = 1 ;	//這裡的i與a.h中的i被當成不同的const變數
 std::cout << i;	//結果為1
}
```
有時候我們想要在不同的檔案中共享同一個`const`變數，但是它的initializer不是一個常量expression，在這種情況下，如果我們不想要編譯器在不同的檔案裡分別產生不同的變數，而是能夠讓該`const`物件表現的跟其他nonconst變數一樣，只在一個檔案中定義`const`，而在多個檔案中使用它。
想要解決這個問題(define a single instance of a `const` variable)，我們在該`const`變數的定義以及宣告都使用關鍵字`extern`。  
>**Example**
``` c++
// file_1.cc defines and initializes a const that is accessible to other files
extern const int bufSize = fcn();
// file_1.h
extern const int bufSize; // same bufSize as defined in file_1.cc
```
在此例中，file_1.cc定義且初始化了`bufSize`，但`bufSize`是`const`，為了讓其他檔案能夠使用它，我們使用`extern`。在/file_1.h對`bufSize`的宣告仍用了`extern`，這表示(signify)`bufSize`並非該檔案特有的(not local to the file)而且其定義會在其他檔案中出現。  
>**Example**
``` c++
//a.h的程式碼如下
#ifndef A_H
#define A_H

 extern const int i = 2 ;

#endif
```
``` c++
//a.cpp程式碼如下
#include <iostream>
#include "a.h"
 extern const int i ;	//無法重新定義i
int main(){
 std::cout << i; 	//輸出結果為2
}
```
>**Note**  
我們如果想在多個檔案共享一個`const`物件，則我們要用*extern定義它*。

### 2.4.1 Reference to `const`
我們可以把reference綁定到`const`物件上，但不能透過該reference改變它的值:
``` c++
const int ci = 1024;
const int &r1 = ci;   // ok: both reference and underlying object are const
r1 = 42;              // error: r1 is a reference to const
int &r2 = ci;         // error: non const reference to a const object
```  
>**Terminology : const Reference is a Reference to const**  
Reference並非一個物件，所以沒有是否為`const`的問題(因為它只能綁定一個對象，不能再改變，意義上來說一定是`const`)，因此當我們說const reference其實是在說reference to const(對常量做reference)。

#### Initialization and References to `const`
我們可以把一個對`const`物件的reference(Reference to const)綁定到非`const`的物件、literal或是一般的expression(只要可以轉換成該reference的type即可(e.g. `coust &int` 可綁定到double expression)。  
>**Example**
``` c++
int i = 42;
const int &r1 = i;      // we can bind a const int& to a plain int object
const int &r2 = 42;     // ok: r1 is a reference to const
const int &r3 = r1 * 2; // ok: r3 is a reference to const
int &r4 = r1 * 2;        // error: r4 is a plain, non const reference
```
要理解上面的規則，我們可以從下列的例子說明:
``` c++
double dval = 3.14;
const int &ri = dval;
```
`dval`是一個浮點數，為了確保`ri`的值為整數，編譯器會將程式碼轉換成類似下面的東西:
``` c++
const int temp = dval;   // create a temporary const int from the double
const int &ri = temp;    // bind ri to that temporary
```
我們看到了`ri`其實是被綁定到一個**temporary** 物件，這是由編譯器在需要一個空間儲存expression運算後的結果時所創建的一個*未命名*的物件(通常在C++中被簡稱為temporary)，所以說，`ri`並非綁定於`dval`中，而是一個`const`的temporary，因此如果`ri`不是一個`const`reference，會被編譯器視作違法的。

#### A Reference to `const` May Refer to an Object That Is Not `const`
`const` reference綁定的物件不一定就是`const`的，它只代表我們*透過該reference*能做的事是受`const`限制的，而它被綁定的對象*未必*受到相同的限制。  
>**Example**
``` c++
int i = 42;
int &r1 = i;          // r1 bound to i
const int &r2 = i;     // r2 also bound to i; but cannot be used to change i
r1 = 0;              // r1 is not const; i is now 0
r2 = 0;              // error: r2 is a reference to const
```
在此例中，`r2`綁定的對象是nonconst的，但我們不能透過`r2`來改變`i`的值(因為r2是`const` reference)，但我們可以透過`r1`來改變`i`的值。

### 2.4.2 Pointers and `const`
如同reference一樣，我們可以定義指向`const`或nonconst物件的pointer，類比於reference to `const`，我們不能用**pointer to const**來改動它所指向的物件，而且我們只能透過pointer to `const`來儲存`const`物件的位址:
``` c++
const double pi = 3.14;   // pi is const; its value may not be changed
double *ptr = &pi;        // error: ptr is a plain pointer(一定要用pointer to const)
const double *cptr = &pi; // ok: cptr may point to a double that is const
*cptr = 42;               // error: cannot assign to *cptr
```
同樣地，我們也可以把一個pointer to `const`指向一個nonconst物件(但不能透過該pointer改變該物件的值):
``` c++
double dval = 3.14;       // dval is a double; its value can be changed
cptr = &dval;             // ok: but can't change dval through cptr
```
>**Tip**  
可以把pointers to `const` 或是 reference to `const`想像成它們指向或綁定的是一個`const`物件。

### `const` pointer
``` c++
int errNumb = 0;
int *const curErr = &errNumb;    // curErr 永遠指向errNumb
const double pi = 3.14159;
const double *const  pip  = &pi; // pip 是一個指向const 物件的const pointer 
```
我們可以用2.3.3提到的由右往左讀法來理解這些宣告的意思:首先最靠近`curErr`的符號是`const`，代表`curErr`是一個`const`物件，該物件的型別則由剩下的部份決定，接下來的符號是`*`，代表它是一個pointer，最後的base type部分則將`curErr`的型別給完全決定了，它是一個`const` pointer，並且指向一個`int`型別的物件，我們不能改變它儲存的位址值，但能改變它所指向物件(也就是`errNumb`)的值。類似地，`pip`為一個指向`const double`的`const pointer`，我們不但不能改變它儲存的位址值，也不能改變它所指向物件(也就是`pi`)的值:
``` c++
*pip = 2.72;     // error: pip is a pointer to const
// if the object to which curErr points (i.e., errNumb) is nonzero
if (*curErr) {
    errorHandler();
    *curErr = 0; // ok: reset the value of the object to which curErr is bound
}
```
### 2.4.3 Top-Level `const`
從上一小節可知，一個pointer是否`const`與其可指向的物件是否為`const`是可以獨立討論的:我們使用**top-level const**來表示該指標*本身*為`const`，而用**low-level const**來表示該指標*可以指向*`const`*物件*。
更廣義來說，top-level const指的是該物件本身為`const`，top-level const可以出現在任何一種物件型別中(ex.內建算數型別、class type或pointer type等)，而low-level const則出現在複合型別(e.g. pointer or reference)的base type，因此pointer可以同時擁有top-level const與low-level const(而且兩者互相獨立)。  
>**Example**
``` c++
int i = 0;
int *const p1 = &i;  // we can't change the value of p1; const is top-level
const int ci = 42;   // we cannot change ci; const is top-level
const int *p2 = &ci; // we can change p2; const is low-level
const int *const p3 = p2; // right-most const is top-level, left-most is not
const int &r = ci;  // const in reference types is always low-level
```
當我們copy某個物件時，top-level const會被*自動忽略*:
``` c++
i = ci;  // ok: copying the value of ci; top-level const in ci is ignored
p2 = p3; // ok: pointed-to type matches; top-level const in p3 is ignored
```
然而，low-level不會被忽略，當我們copy一個物件時，兩邊的物件都必須有相同的low-level，或者除非兩者的型別之間可以轉換，一般來說nonconst*可以*轉換成`const`，`const`卻*不行*轉換成nonconst:
``` c++
int *p = p3; // error: p3 has a low-level const but p doesn't
p2 = p3;     // ok: p2 has the same low-level const qualification as p3
p2 = &i;     // ok: we can convert int* to const int*
int &r = ci; // error: can't bind an ordinary int& to a const int object
const int &r2 = i; // ok: can bind const int& to plain int
```
### 2.4.4 `constexpr` and Constant Expressions
若一個expression的值不能改變且在*編譯的時候就能被計算*，則稱為**const expression**，字面值(literal)就是一種const expression。一個從constant expression初始化而來的cosnt物件也是一個const expression，一個物件(或expression)是否為const expression取決於它的型別以及initializer，例如:
``` c++
const int max_files = 20;    // max_files is a constant expression
const int limit = max_files + 1; // limit is a constant expression
int staff_size = 27;       // staff_size is not a constant expression
const int sz = get_size(); // sz is not a constant expression
```
`staff_size`雖然是用字面值初始化，然而它的值卻能被改變；而`sz`雖然是`const`，它initializer的值卻不能在編譯的時候就被算出來，因此兩者都不是constant expression。在C++的某些情況下會要求使用constant expression。

#### `constexpr` Variables
為了保證某個變數是constant expression，我們可以用`constexpr`來叫編譯器確保這件事，宣告成`constexpr`的變數不但是`const`而且還一定要用constant expression來初始化:
``` c++
constexpr int mf = 20;        // 20 is a constant expression
constexpr int limit = mf + 1; // mf + 1 is a constant expression
constexpr int sz = size();    // ok only if size is a constexpr function
```
雖然一般的函式不能拿來當作`constexpr`變數的initializer，但我們可以定義某些特定的函式為`constexpr`來當作initializer，見6.5.2。

#### Literal Types
由於constant expression要在編譯時就被計算出來，因此能在`constexpr`宣告中使用的型別也受到限制，能在`constexpr`宣告中使用的型別稱為字面值 (或常值)型別(literal types)。算數型別、reference、pointer都算是字面值型別，而*IO以及string型別都不是字面值型別*，因此我們不能將它們定義成`constexpr`，我們在7.5.6以及19.3會有更多關於字面值型別的討論。
儘管reference以及pointer都可以定義成`constexpr`，能用來初始化它們的物件卻受到嚴格限制，我們可以用字面值`nullptr`或是`0`來初始化`constexpr` pointer，或者可以將它們指向(或綁定)擁有固定位址的物件，在6.1.1會解釋到，在函式內部定義的物件通常(例外:`static`變數)不是存在固定位址中的，因此我們不能用來初始化`constexpr` pointer或被`constexpr` pointer指向，而定義於函式外的變數則可以(`static`變數也可)。

#### Pointers and `constexpr`
一個重點是當我們在`constexpr`宣告中定義一個pointer時，`constexpr` specifier是*作用在pointer上面，而非它指向的物件型別*:
``` c++
const int *p = nullptr;     // p is a pointer to a const int
constexpr int *q = nullptr; // q is a const pointer to int
```
儘管表面上看起來很像，但是`p`跟`q`卻截然不同:`p`是一個指向`const`的指標，而`q`卻是一個constant pointer，造成這種差異的原因是因為`constexpr`是作用於它定義之物件的top-level const上，因此constexpr pointer可以指向`const`物件或非`const`物件型別:
``` c++
constexpr int *np = nullptr; // np is a constant pointer to int that is null
int j = 0;
constexpr int i = 42;  // type of i is const int
// i and j must be defined outside any function
constexpr const int *p = &i; // p is a constant pointer to the const int i
constexpr int *p1 = &j;      // p1 is a constant pointer to the int j
```

## 2.5 Dealing with Types
### 2.5.2 The `auto` Type Specifier
有時候要判斷一個變數的型別非常困難，而`auto`的功能是叫編譯器自動從initializer去推出型別，因此，使用`auto`來當型別的變數*一定*要有initializer:
``` c++
// the type of item is deduced from the type of the result of adding val1 and val2
auto item = val1 + val2; // item initialized to the result of val1 + val2
```  
>**Example**  
如果`val1`與`val2`的型別`double`，則`item`的型別就是`double`。
如同其他type specifier，可以在`auto`中定義多個變數以及複合型別，但要注意initializer的型別*必須保持一致*:
``` c++
auto i = 0, *p = &i;      // ok: i is int and p is a pointer to int
auto sz = 0, pi = 3.14;   // error: inconsistent types for sz and pi
```
#### Compound Types, `const`, and `auto`
編譯器從`auto`得出的型別未必和initializer的型別保持一致，有時候編譯器會做一些調整來達成一般的初始化規則。  
(1) 當我們使用reference當作initializer時，編譯器將會使用該reference所參考之物件的型別當成`auto`的type，*而非reference本身*:
``` c++
int i = 0, &r = i;
auto a = r;  // a is an int (r is an alias for i, which has type int)
```  
(2) `auto`一般會忽略top-level const，但low-level const則會保存:
``` c++
const int ci = i, &cr = ci;
auto b = ci;  // b is an int (top-level const in ci is dropped)
auto c = cr;  // c is an int (cr is an alias for ci whose const is top-level)
auto d = &i;  // d is an int*(& of an int object is int*)
auto e = &ci; // e is const int*(& of a const object is low-level const)
```
如果想要有top-level const，則必須清楚的表示:
``` c++
const auto f = ci; // deduced type of ci is int; f has type const int
```
同樣的道理，我們也可以有對auto-deduced type的參考，與一般初始化規則一樣:
``` c++
auto &g = ci;       // g is a const int& that is bound to ci
auto &h = 42;       // error: we can't bind a plain reference to a literal
const auto &j = 42; // ok: we can bind a const reference to a literal
```
必須記住reference和pointer只是某個特定declarator的一部份，而非base type的一部份:
``` c++
auto k = ci, &l = i;    // k is int; l is int&
auto &m = ci, *p = &ci; // m is a const int&;p is a pointer to const int
// error: type deduced from i is int; type deduced from &ci is const int
auto &n = i, *p2 = &ci;
```
### 2.5.3 The `decltype` Type Specifier
有的時候我們想從某個expression來定義一個變數，但是又不希望用該expression初始化那個變數，這時候就可以利用`decltype`，它將operand的型別回傳，編譯器會從expression決定型別但*不會計算它*:
``` c++
decltype(f()) sum = x; // sum has whatever type f returns
```
此例中，編譯器不會呼叫`f()`，但會根據呼叫`f()`會得到的型別來決定`sum`的型別。
與`auto`不同的地方在於，當`decltype`的作用對象為一個變數的時候，它會回傳*包含top-level const以及reference的型別*:
``` c++
const int ci = 0, &cj = ci;
decltype(ci) x = 0; // x has type const int
decltype(cj) y = x; // y has type const int& and is bound to x
decltype(cj) z;     // error: z is a reference and must be initialized
```
#### `decltype` and Reference
當我們將`decltype`作用在*非變數*的expression上，會得到該expression獲得的型別，而且當該expression獲得左值(見4.1.1)的時候，*會得到一個reference型別*:
``` c++
// decltype of an expression can be a reference type
int i = 42, *p = &i, &r = i;
decltype(r + 0) b;  // ok: addition yields an int; b is an (uninitialized) int
decltype(*p) c;     // error: c is int& and must be initialized
```
這裡的`r`為一個reference，所以`decltype(r)`是一個reference型別，如果我們想要的是該reference所參考對象的型別，則我們可以在expression中使用`r`，例如`r+0`，這麼做可以獲得一個非reference型別。
此外deference operator也是獲得reference型別的另外一個例子，由於我們dereference一個pointer，我們得到了該pointer所指的物件(左值)，因此我們得到的是`int&`而非一般的`int`。
另外一個與`auto`重要的不同是`decltype`會根據給定expression的形式(form)來推論得到的型別，其中容易讓人混淆的是將變數包在括號中會影響`decltype`回傳的型別，當變數沒有被括號時，得到的是該變數的型別，然而如果我們把它包在括號裡時，編譯器會將`decltype`的operand視為一個expression，而變數是一個左值expression，因此使用`decltype`時會獲得一個reference:
``` c++
// decltype of a parenthesized variable is always a reference
decltype((i)) d;    // error: d is int& and must be initialized
decltype(i) e;      // ok: e is an (uninitialized) int
```

## 2.6 Defining Our Own Data Structure
### 2.6.3 Writing Our Own Header Files
在19.7中會提到可以在函數中定義class，然而這種class的功能有限，因此通常來說，在給定的source file中，我們不會在函數裡面定義class，當class定義於函數外面，他們在該source file中只能被定義一次。為了確保不同的檔案中使用定義相同的class，通常會把class定義於標頭檔(header files)中，例如`string`類別就是定義於`string`標頭檔之中。
標頭檔中通常包含只能被定義一次的實體，例如class definitions、`const `與 `constexpr`變數。

