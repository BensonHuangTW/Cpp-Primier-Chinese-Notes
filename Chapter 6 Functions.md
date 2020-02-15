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
