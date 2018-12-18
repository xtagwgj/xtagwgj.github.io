---
title: Kotlin 集合
tags:
  - Kotlin
  - Android
date: 2017-08-17 16:39:59
categories:
---

### 容器 可变/不可变

```java
List<out T> 只读list; MutableList<T>;
Set<out T>/MutableSet<T>
Map<K, out V>/MutableMap<K, V>

//示例
val numbers: MutableList<Int> = mutableListOf(1, 2, 3)
val readOnlyView: List<Int> = numbers
println(numbers)        // prints "[1, 2, 3]"
numbers.add(4)
println(readOnlyView)   // prints "[1, 2, 3, 4]"
readOnlyView.clear()    // -> does not compile
```

### MutableList

```java
//示例
val numbers: MutableList<Int> = mutableListOf(1, 2, 3)
val readOnlyView: List<Int> = numbers
println(numbers)        // prints "[1, 2, 3]"
numbers.add(4)
println(readOnlyView)   // prints "[1, 2, 3, 4]"
readOnlyView.clear()    // -> does not compile

//属性
size    //尺寸
lastIndex   //最尾元素的索引值

//方法
isNotEmpty()

forEach     //循环
forEachIndexed  //带索引的循环

//取值
component1()    //获取第1(2/3/4/5/)个元素
elementAtOrElse //返回指定E,或index越界时返回入参函数的执行结果
getOrElse   //取值，如果越界则返回传入的默认值
getOrNull   //取值，越界则返回null

contains    //包含
containsAll

//删除
remove
removeAll   //删除有所满足入参函数的元素
drop(n) //list,返回删除前n个元素后的新list
dropLast

slice   //子list
sort()  //排序
sortBy  //指定排序
sorted  //返回新的排序后的list


//去重
distinct()  //list
distinctBy(T->{})   //list,使用入参函数处理

//统计--判断
all     //boolean,每个元素都满足入参函数
any     //任一满足
count   //满足入参函数E的个数
none    //都不满足

//转换成其它类型
toHashSet
toSet
toSortedSet
zip 


//转换
reverse     //反转列表
reversed
asReversed  //list,生成反转列表
associate   //map,将E按入参函数转成pair返回
associateBy //map,将E按入参函数转成K返回
map         //对每一个元素做变换后返回新的list
mapNotNull  
flatMap     //对每一个元素做变换后返回新的list
mapIndexed  //函数入参带index
mapIndexedNotNull   //list，只返回非null值
minus
minusAssign
plus
plusAssign
take    //返回前n个元素
takeLast


//过滤--新增
filter      //list,返回符合入参函数的E
filterNot
filterNotNull   //返回非null元素
filterIsInstance    //list,返回特定类型的元素
groupBy     //map, 分组
intersect   //set,两个容器中都包含的元素

//查找
find    //返回第一个符合入参函数的元素或null
findLast
indexOf //返回第一个的索引值，无则-1
indexOfFirst    //返回第一个符合入参条件的元素索引
indexOfLast 
last    //最后一个入参是否符合
lastIndexOf     //最后一个符合条件元素索引，或-1
lastOrNull  //最有一个元素是否符合,或null
single  //未找到则抛出异常
singleOrNull


//最大，最小
max()   //返回最大值的元素
maxBy   //经过入参函数处理过的最大值
min()
minBy

//求和--递减
fold    //从左往右，在基础值上求和,  xx.fold(9){t,a->a+t}
foldRight   //从右往左求和
reduce  //从左往右递减
reduceIndexed
sumBy   //求和 
```

### MutableMap
```java
//基本示例
val readWriteMap = hashMapOf("foo" to 1, "bar" to 2)
println(readWriteMap["foo"])  // prints "1"
val snapshot: Map<String, Int> = HashMap(readWriteMap)

//属性
size    //map尺寸
keys    // Returns a mutable Set of all keys 
values  // Returns a mutable Set of all values 

isEmpty()   //判空
isNotEmpty
orEmpty()   //如果为null，则返回新map
forEach((key,value)->{})    //循环

get     //取值
getOrDefault    //有则取值，无则返回传入的默认值
getOrElse       //有则返回值，无则返回传入函数的返回值
getOrPut        //有则返回，无则设值
getValue        //key无对应值时抛出异常

clear() 
put(key: K, value: V)
putAll(from: Map<out K, V>)
remove(key: K)

//统计/判断
all     //boolean, 所有元素匹配
any     //boolean,任一元素匹配
count   //int, 返回符合入参函数的规则个数
none()  //boolean,如果为空
none((k,v)->{}) //boolean,所有元素都不匹配  

contains    //boolean,是否包含
containsValue   //boolean

//过滤--新增
filter  //map,按规则过滤返回新map
filterNot   //map,反向
minus       //map,返回删除key(s)后的新map
minusAssign     //map,删除对应的key(s)
plus        //map,返回添加新pair(s)后的新map
plusAssign  //添加到当前map
toSortedMap

//转换
map         //list, pair用入参函数转换后放入list中返回, 和flatmap区别是入参函数的入参不同
flattmap    //list,按入参函数将pair转成元素
mapKeys     //map, 将key处理后当做新key
mapValues       //map, 将value处理后当做新value
toMutableMap    //转成可变map

//查找
maxBy   //<K,V>, 返回入参函数处理后值最大的那一个pair (null if there are no entries)
MaxWith //<K,V>,入参比较函数
minBY
minWith
```