# Chapter 6 Functions
## 6.1 Function Basics
#### Parameters and Arguments
**Argument**(引數):用來初始化函數之參數的initializers。
舉例:
`fact`函數的定義:
``` c++
int fact(int val)
{
    int ret = 1; // local variable to hold the result as we calculate it
    while (val > 1)
        ret *= val--;  // assign ret * val to ret and decrement val
    return ret;        // return the result
}
```
在此函數中，`val`為參數。若我們在`main()`裡面呼叫`fact`函數的時候:
``` c++
int main()
{
    int j = fact(5);  // j equals 120, i.e., the result of fact(5)
    cout << "5! is " << j << endl;
    return 0;
}
```
此時括號中的5即為引數。儘管引數的擺放有順序，但是計算引數時卻未必依照該順序逐一計算，compiler可以任意挑選哪個引數先被計算。引數的型別限制與initializer的型別規則試一致的，我們必須傳入與函數參數相同數目的引數，這保證了參數在函數被呼叫的時候必定會被初始化。
例(延續上方的`fact`函數):
``` c++
fact("hello");       // error: wrong argument type
fact();            // error: too few arguments
fact(42, 10, 0);     // error: too many arguments
fact(3.14);        // ok: argument is converted to int (且為implicitly，經過修剪)
```
故: fact(3.14)同義於fact(3)。

#### Function Parameter List
函數可以沒參數，但定義時一定要有括號:
``` c++
void f1(){ /* ... */ }     // implicit void parameter list
```
為了與C語言相容，也可以用下面的語法:
``` c++
void f2(void){ /* ... */ } // explicit void parameter list
```
就算兩個參數型別一樣，仍必須把兩個型別都寫出來:
``` c++
int f3(int v1, v2) { /* ... */ }     // error
int f4(int v1, int v2) { /* ... */ }  // ok
```

#### Function Return Type
函數可以不回傳值(使用`void`)，但不能回傳一個陣列型別或者函數型別，後面的章節會提到如何定義回傳(1)指向陣列的pointer(或是reference)以及(2)指向函數的指標方式(6.7)。

### 6.1.1 Local Objects
在C++裡面，名稱有**作用域**(**scope**)，而物件則有**生命週期**(**lifetime**)。  
(1)名稱的作用域是指該名稱在程式中可見(visible)的部分。  
(2)物件的生命週期是指在程式執行過程中，該物件存在的時間。  
函數的block形成一個新的作用域，而函數的:  
(1)參數  
(2)定義在函數體裡面的變數  
我們稱作**局部變數**(local variables)。它們只在函數的內部可見，而且會隱藏外層作用域中擁有相同名稱的聲明(They are “local” to that function and hide declarations of the same name made in an outer scope.)。  
定義於所有函數外部的物件存在於整個程式執行的過程中，而局部變數的生命週期取決於它被定義的方式。

#### Automatic Objects
對應於一般的局部變數所對應的物件來說，它們被創建於當函數的控制路徑(control path)通過該變數的定義時，並且在通過該變數被定義的block末端被銷毀。這種只存在於一個block之中的物件稱為automatic objects，當該block執行結束後，裡面定義之automatic objects的值就會變成未定義的。而參數是一種automatic objects。  
函數體中，作為局部變數的automatic objects如果在被定義時若沒有initializer，則會被默認初始化，這代表未被初始化的內建型別會有未定義的值(若在函數體外則會被初始化成0，見2.2.1)。

#### Local static Objects
當我們用`static`定義某個局部變數時，可以使它的生命週期超越函數被呼叫的時候，**Local `static` Objects**在該變數的定義被第一次執行之前就被初始化了，並且直到程式終止才被銷毀。  
>**Example**  
``` c++
size_t count_calls()
{
    static size_t ctr = 0;  // value will persist across calls
    return ++ctr;
}
int main()
{
    for (size_t i = 0; i != 10; ++i)
        cout << count_calls() << endl;
    return 0;
}
```
上述程式輸出結果為:
```
1
2
3
4
5
6
7
8
9
10
```
如果一個局部靜態變數沒被explicit initialized，則它會被value initialized(3.3.1)，也就是說內建類型的type會被初始化成0。
