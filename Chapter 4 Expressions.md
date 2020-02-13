# Chapter 4 Expressions
## 4.6 The Member Access Operators
點(`.`)與箭號(`->`)運算服提供了member access，點運算符從class type的物件中獲得成員，而箭號則是結合了dereference與member access兩種功能，因此它是作用於*指向該物件的pointer中*。因此`ptr->mem`與`(*ptr).mem`是同義的:
``` c++
string s1 = "a string", *p = &s1;
auto n = s1.size(); // run the size member of the string s1
n = (*p).size();    // run size on the object to which p points
n = p->size();      // equivalent to (*p).size()
```
## 4.8 The Bitwise Operators
bitwise operator以整數型別作為運算元，在17.2會看到這些運算子也可以用在名為`bitset`的型別(可變動位元數量的函式庫型別)。
如果運算元為small integer(e.g. `short`)，則會先被promote(見4.11.1)到更大的整數型別，運算元可為有號或無號數，但如果是負號的話，則sign bit的處理方式是machine-dependent的，而且左移它還會導致未定義結果。
`WARNING`
既然無法保證signed bit會怎麼被處理，使用bitwise operator的時候*強烈建議*使用unsigned型別。  
**Table 4.3. Bitwise Operators (Left Associative)**
![image](https://user-images.githubusercontent.com/55428505/66105133-26a70280-e5ed-11e9-8a5a-5e431c6ba58b.png)

#### Bitwise Shift Operators
當我們使用`<<`或`>>`時，代表將其左側運算元做左移或右移(位移bit數由右方運算元決定)，並且獲得一個位移後值的copy(可能經過promote)，右方運算元不能為負或是超出結果的bit數，否則會是undefined的，被移出的位元會被丟棄。
>**Example**  
![image](https://user-images.githubusercontent.com/55428505/66105165-463e2b00-e5ed-11e9-98a6-725b36422058.png)  
(1)	左移運算子(left-shift operator, the `<<` operator) :於右側插入0，要注意當平移的是`unsigned char`或`unsigned short`時，會先被提升至`signed int`(見4.11.1)。  
(2)	右移運算子(right-shift operator, the `>>` operator) :當運算對象為`unsigned`或非負整數時於左側插入0，但當其為有號數的複數時，結果視實作方式決定。

#### Bitwise NOT Operator
NOT運算子(`~`operator)產生一個翻轉運算對象位元(1變0，0變1)的值。
>**Example**  
![image](https://user-images.githubusercontent.com/55428505/66105248-961cf200-e5ed-11e9-953e-281fba977390.png)  
在此例中，`char`會被提升成`int`，其值不變，但會在左側(高位)增加24個0，然後再進行翻轉。

#### Bitwise AND, OR, and XOR Operators
>**Example**  
![image](https://user-images.githubusercontent.com/55428505/66105328-c5336380-e5ed-11e9-847e-0ad4b5ac7b0b.png)

>**WARNING**  
要注意上面的AND與OR符號分別為`&`和`|`，與邏輯運算元的`&&`和`||`*不同*。

#### Shift Operators (aka IO Operators) Are Left Associative
位移運算元(也就是輸出輸入運算元)為左結合。
>**Example**  
``` c++
cout << "hi" << " there" << endl;
```
等同於執行
``` c++
( (cout << "hi") << " there" ) << endl;
```
此外，位移運算子的優先權為中等，比算數運算子低，但高於relational, assignment以及conditional operators。
>**Example**  
``` c++
cout << 42 + 10;   // ok: + has higher precedence, so the sum is printed
cout << (10 < 42); // ok: parentheses force intended grouping; prints 1
cout << 10 < 42;   // error: attempt to compare cout to 42!
```
#### Using Bitwise Operators
以下為使用bitwise operator的一個例子:
>**Example**  
利用位元來記錄30位學生是否通過考試，首先用`unsigned`整數來記錄每次考試:
``` c++
unsigned long quiz1 = 0; // we'll use this value as a collection of bits
```
`quiz1`是一個`unsigned long`，它在任何機器上都至少有32位元。當我們想要表示編號27號的學生通過考試，可以利用`unsigned long`整數的字面值`1`:
``` c++
1UL << 27 // generate a value with only bit number 27 set
```
`1UL`擁有一個低位元的1以及(至少)31位元的0，我們使用`unsigned long`是因為`int`只保證至少有16位元，而我們至少需要27個，之後再將其左移27位:
``` c++
quiz1 |= 1UL << 27; // indicate student number 27 passed
```
接下來我們用OR來更新`quiz1`的值，可以使用compound assignment的形式:
``` c++
quiz1 |= 1UL << 27; // indicate student number 27 passed
```
以上等同於
``` c++
quiz1 = quiz1 | 1UL << 27; // equivalent to quiz1 | = 1UL << 27;
```
如果我們想把27號學生標為不及格，則可以使用AND及NOT:
``` c++
quiz1 &= ~(1UL << 27); // student number 27 failed
```
最後，如果我們想要查看該學生的狀態的話可以使用AND:
``` c++
bool status = quiz1 & (1UL << 27); // how did student number 27 do?
```
## 4.9 The sizeof Operator
`sizeof`運算子回傳了一個expression或是type name*所佔的byte數*，為右結合，並且其回傳結果為constant expression(見2.4.4)，型別為`size_t`見3.5.2)，有兩種形式:
``` c++
sizeof (type)
sizeof expr
```
`sizeof`並*不會*計算它的運算元:
``` c++
Sales_data data, *p;
sizeof(Sales_data); // size required to hold an object of type Sales_data
sizeof data; // size of data's type, i.e., sizeof(Sales_data)
sizeof p;    // size of a pointer
sizeof *p;   // size of the type to which p points, i.e., sizeof(Sales_data)
//上列等價於sizeof (*p);
sizeof data.revenue; // size of the type of Sales_data's revenue member
sizeof Sales_data::revenue; // alternative way to get the size of revenue
```
即使上面的`p`為無效指針也沒關係，因為`sizeof`並不需要dereference該pointer來獲得它所指向的物件型別。
`sizeof`回傳的結果部分取決於它的型別:  
(1)	`sizeof char`的結果一定是1。  
(2)	`sizeof`某個參考型別回傳該被參考的型別產生之一個物件的大小。  
(3)	`sizeof`一個pointer回傳一個pointer的大小。  
(4)	`sizeof`一個被dereference的指針回傳該pointer指向的型別產生之一個物件的大小，且該pointer可為無效pointer。  
(5)	`sizeof`一個陣列會回傳整個陣列的大小，而非指標的大小，等同於sizeof該陣列元素的型別再乘上元素個數。  
(6)	`sizeof`一個`string`或`vector`只會回傳這些型別固定部分的大小，而非這些物件所使用的大小。  
>**Example**  
由於`sizeof`回傳整個陣列大小，因此可以用它來除以元素的大小來獲得元素個數:
``` c++
// sizeof(ia)/sizeof(*ia) returns the number of elements in ia
constexpr size_t sz = sizeof(ia)/sizeof(*ia);
int arr2[sz];  // ok sizeof returns a constant expression § 2.4.4 (p. 65)
```
## 4.11 Type Conversions
>**Definition**  
Two types are related if there is a **conversion** between them.  
>⇒當需要某一type的value (or object) 時，可以用跟它 related的value (or object)代替。

>**Example**  
int ival = 3.541 + 3;  
⇒C++會根據一套類型自動轉換規則將兩種type轉成共同(common)的type再求值。  
⇒又稱**implicit conversions**
大多情況下，算術類型的type會盡量轉換成保持精度(precision)的type。  
⇒上例的3會被轉換成double，運算結果為double，然後才initialize。故結果被轉換成6(int)來initialize ival(要配合初始化類型)。

一些implicit conversions:
(1) 大部分的expression中，小的整數類型(smaller than int)先被轉成較大的整數類型。  
(2) 條件(conditions)中，非bool expression被轉成bool。  
(3) 初始化: initializer is converted to the type of the variable。  
(4) 賦值:右邊轉成左邊。  
(5) arithmetic and relational expressions:轉成common type。  
呼叫函式時也會發生(Ch6)。

###  4.11.1 The Arithmetic Conversions
把一種算術類型轉成另一種。
規則:
(1) 小類型轉成最寬的類型。e.g.出現`long double`，則其他的operand都轉成`long double`。  
(2) 整數與浮點數類型同時出現，則整數被轉成適當的浮點數類型。  

#### Integral Promotions
Integral promotions convert the small integral types to a larger integral type.  
⇒如果`int`容納的下就轉`int`，否則轉`unsigned int`。e.g. bool promote to 0 or 1.
e.g. bool, char, signed char, unsigned char, short, and unsigned short

較大的`char`類型(`wchar_t`, `char16_t`, and `char32_t`)轉成最小可容納的整數類型(`int`, `unsigned int`, `long`, `unsigned long`, `long long`, or `unsigned long long`)

Operands of Unsigned Type
*先進行Integral promotions*，之後:  
Case (1) resulting type(s) match :  
no further conversion is needed.  
Case (2) both (possibly promot ed) operands have the same signedness :                                                                                                                                                                  the operand with the smaller type is converted to the larger type.  
Case (3) signedness 不同且 the type of the unsigned operand *is the same as or
larger* than that of the signed operand :  
*the signed operand is converted to unsigned.*  
e.g. given an unsigned int and an int, the int is converted to unsigned int (如果該int為負值，則會有2.1.2的問題e.g.-1變成很大的正值)  
Case (4) signedness 不同且signed operand 比 unsigned operand 大:  
result is machine dependent.If all values in the unsigned type fit in the larger type, then the unsigned operand is converted to the signed type.If the values don’t fit, then the signed operand is converted to the unsigned type.  
e.g. if the operands are `long` and `nsigned int`, and `int` and `long` have the
same size, the `long` will be converted to `unsigned int`. If the long type has more
bits, then the `unsigned int` will be converted to `long`.  
**Type size 參照(表2.1)**  
![image](https://user-images.githubusercontent.com/55428505/66817057-b0e85280-ef6d-11e9-9517-a569dbf6b7ca.png)

#### Understanding the Arithmetic Conversions
``` c++
bool  flag;         char  cval;
short  sval;        unsigned short  usval;
int  ival;          unsigned int   uival;
long  lval;         unsigned long  ulval;
float  fval;         double   dval;
3.14159L + 'a'; // 'a' 先promoted to int, 然後int converted to long double(case2)
dval + ival;//ival converted to double(case2)
dval + fval; //fval converted to double(case2)
ival = dval; //dval converted (by truncation) to int(assignment)
flag = dval; //if dval is 0, then flag is false, otherwise true(assignment)
cval + fval; //cval promoted to int, then that int converted to float(case2)
sval + cval; //sval and cval promoted to int(case1)
ival + ulval; //ival converted to unsigned long(case3)
usval + ival; //promotion depends on the size of unsigned short and int
uival + lval; //conversion depends on the size of unsigned int and long
```

### 4.11.2 Other Implicit Conversions
#### Array to Pointer Conversions
大部分的expression中，當我們用array時，它會被自動轉成一個指向該array第一個元素的pointer:
>**Example**  
int ia[10];   // array of ten ints
int* ip = ia; // convert ia to a pointer to the first element
例外:
(1)	使用`decltype`時。
(2)	使用取址運算子(`&`)、`sizeof`或是`typeid`(見19.2.2)。
(3)	使用reference來初始化array(when we initialize a reference to an
array)時。  

>**Example**  
```c++
int (&arrRef)[10] = arr;
```

#### Pointer Conversions
(3)	`0`或者`nullptr`可被轉換成任何的指針型態。
(4)	指向非`const`型態的指針能被轉成`void*`，指向`const`型態的指針能被轉成`const void*`。
(5)	有關繼承的type(見15.2.2)

#### Conversions to bool
有種自動轉換是把算數或指針類型轉成`bool`:如果該算數或指針類型的值為`0`，則轉成`false`，其他的值則轉為`true`。
>**Example**  
``` c++
char *cp = get_string();
if (cp) /* ... */  // true if the pointer cp is not zero
while (*cp) /* ...  */ // true if *cp is not the null character
```

#### Conversion to const
可以將指向非`const` type的指針轉換成對應的指向`cosnt` type的指針，reference也可以(if `T` is a type, we can convert a pointer or a reference to `T` into a pointer or reference to `const T`)，但逆向則不行。
>**Example**  
``` c++
int i;
const int &j = i;  // convert a nonconst to a reference to const int
const int *p = &i; // convert address of a nonconst to the address of a const
int &r = j, *q = p; // error: conversion from const to nonconst not allowed
```

#### Conversions Defined by Class Types
class可以定義由compiler自動執行的轉換，但是每次只能執行一種類型的轉換(7.5.4)。
一些常見的class-type conversions:  
(1)	當要用`string`類型時，可以從C-style的字串(字元陣列)轉換。  
(2)	在條件中讀入`istream`時，IO libery定義了從`istream`(`cin`)到`bool`的轉換:  
(i)當最後一次讀取成功，則轉換成`true`  
(ii) 當最後一次讀取成功，則轉換成`false`  
>**Example**  
``` c++
string s, t = "a value";//character string literal converted to type string
while (cin >> s) ;//while condition converts cin to bool
```

### 4.11.3 Explicit Conversions
>**Example**  
有時候，我們想將兩個`int`相除的結果存進`double`中:
``` c++
int i,j;
double slope = i / j; //然而此時的slope值為0而非0.5(∵會先進行整數除法，在將結果轉換成double)
```
為了達成目的，我們必須先將`i`或`j`先轉成`double`，另一個int才會遵循4.11.1的規則先轉成`double`再做運算。
⇒使用`cast`(強制類型轉換)來要求Explicit Conversions

#### Named Casts(命名的強制類型轉換)
語法:
``` c++
cast-name<type>(expression);
```
⇒其中cast-name有四種(`static_cast`, `dynamic_cast`, `const_cast`, and `reinterpret_cast`)，決定了轉換的模式，type則是我們要轉成的類型。

#### static_cast
只要不包含low-level `const`的類型轉換都能經由`static_cast`來達成，舉上面的例子來說:
``` c++
//使用類型強制轉換來達成浮點數的除法
double slope = static_cast<double>(j) / i;
```
當我們要把較大的算術類型賦值給較小的算術類型的時候，`static_cast`就派上用場了，它告訴compiler我們注意到了並且不在乎可能的精度損失(如果直接賦值compiler可能會發出警告)。
`static_cast`也可以用來執行compiler無法自動執行的類型轉換，例如:
```c++
void* p = &d; // ok: address of any nonconst object can be stored in a void*
// ok: converts void* back to the original pointer type(但要確保d是double)
double *dp = static_cast<double*>(p);
```
當我們用`static_cast`將`void*`所儲存的指標做強制轉換回該類型的指標時，我們保證了位址值是不變的，但我們要確保轉換後得到的類型就是指針所指的類型，否則會產生undefined的結果。

#### dynamic_cast
詳見19.2

#### const_cast
只能改變作用對象的low-level `const`，但無法改變該對象的類型。
>**Example**  
.
``` c++
const char *pc;
char *p = const_cast<char*>(pc); //  ok:但如果透過p寫值是undefined的
```
我們將上面把`const`物件轉成nonconst類型的行為稱作“casts away the `const`”(去掉`const`性質)，只要我們這麼做，compiler就不再阻止我們對該對象進行writing了，如果該對象原本為`const`的話，用`const_cast`來write它的話會產生undefined的結果。
只有`const_cast`可以拿來改變expression的const屬性，用其它類型的name cast做的話會產生編譯錯誤。
>**Example**  
``` c++
const char *cp;
// error: static_cast can't cast away const
char *q = static_cast<char*>(cp);
static_cast<string>(cp); // ok: converts string literal to string
const_cast<string>(cp);  // error: const_cast only changes constness
```

#### reinterpret_cast
A `reinterpret_cast` generally performs a low-level reinterpretation of the bit
pattern of its operands.
>**Example**  
``` c++
int *ip;
char *pc = reinterpret_cast<char*>(ip);
```
該記得的是`pc`所指的真實對象是一個`int`而非字元。如果把`pc`當成一般的字元指針可能在運行時發生錯誤，例如:
``` c++
string str(pc);
```
這可能導致異常的運行時行為。而且當我們用`int`的地址將`pc`初始化時，compiler會當成合法的而不會發出警告，並且`pc`會被當成`char*`來看待。這使得追蹤bug的時候變得更辛苦。

▲建議:盡量避免使用casts!(尤其是`reinterpret_cast`)
