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
