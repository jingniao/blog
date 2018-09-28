---
title: Python的pickle以及其他的序列化
mathjax: false
date: 2018-09-28 22:24:10
tags: python,go,pickle
categories:
---
之前在论坛上看到说`Python`内存占用的问题，有个回复说`不少人还认为pickle是直接将内存中的数据到处到文件里了`，很羞愧，看到这句话之前，我也是这么认为的……
这篇文章就是对这些进行简单的摸底，做到心中有数，从这么看来说，之前写的部分代码坑不小……
<!-- more -->
一、python的字典内存占用以及pickle输出大小
```python
import sys
import pickle
result = {}

for i in range(1000000):
    result[str(i).encode('utf-8')] = str(i).encode('utf-8')

print(sys.getsizeof(result))
with open('test.pk', 'wb') as f:
    pickle.dump(result, f)
```
在我本机 `windows10 python 3.7`，内存中：41943144，导出后的pickle文件大小为25778963
内存多了62%  
这是`str:str` 
`int:int`  41943144: 9739352，大概膨胀了4倍多
key范围扩大到 40000000后，内存占用为2525MB


# 二、golang内存占用


1-n的key范围     内存
1000000         81MB
2000000         162MB
3000000         168MB
4000000         320MB
5000000         330MB
10000000        650MB
20000000        1281MB
40000000        2540MB
在`map`中，平均每增加一个`int:int` key，内存增加66`byte`
将`int`更换为`int32 int32`再尝试
40000000        1417MB 降低还是很明显
代码如下
```go
package main

import (
	"fmt"
	"unsafe"
)

func int2int() map[int32]int32 {
	var res = make(map[int32]int32)
	var i int32
	for i = 0; i < 40000000; i++ {
		res[i] = i
	}
	return res
}

func main() {
	i2i := int2int()
	var aa int32
	fmt.Println(unsafe.Sizeof(aa))
	fmt.Println(len(i2i))
	fmt.Scanln()
}
```
# 小结
这只是简单的对比，没有说哪个优秀之类的，并且这个对比如果硬要对比的话肯定有不少地方不公平，但是单单对`map int:int` 这种类型来说，`python`跟`golang`的内存占用差距比我想象中的要小

# todo
`go`中类似`python`的`pickle`的是`encoding/gob`？有空补上对比