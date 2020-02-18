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
