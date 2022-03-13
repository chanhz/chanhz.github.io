---
title: "Golang封装BitMap"
date: 2019-05-29T15:26:18+08:00
draft: false
tags: ["algo"]
categories: ["algo"]
---

BitMap用位来实现集合元素的查找、去重，在处理海量数据时拥有良好的空间复杂度。



场景 from《编程珠玑》：

> 给一台普通PC，2G内存，要求处理一个包含40亿个不重复并且没有排过序的无符号的int整数，给出一个整数，问如果快速地判断这个整数是否在文件40亿个数据当中？

### 方法一：使用 map[uint32]bool

无符号整数的区间范围是[0,4294967295],

使用整数作key，bool 作值，40亿个整数的需要：

40亿*byte 约等于 4GB.满足不了需求。



### 方法二： BitMap

申请一个 uint64 的数组，每个数的二进制值可以代表64个数。

40亿*bit 约等于 512 MB.



简单实现：

```go
package bitmap

import (
    "bytes"
    "fmt"
)
// BitMap 用以解决海量数据寻找重复
// 判断个别元素是否在海量数据当中
type Bitmap struct {
    words  []uint64 // 用该数组作为集合存储40亿个整数
    length int
}

func New() *Bitmap {
    return &Bitmap{}
}
func (bitmap *Bitmap) Has(num int) bool {
    word, bit := num/64, uint(num%64)
    return word < len(bitmap.words) && (bitmap.words[word]&(1<<bit)) != 0
}

func (bitmap *Bitmap) Add(num int) {
    // word 为数组指针
    // bit 为数组元素位的指针
    word, bit := num/64, uint(num%64)
    for word >= len(bitmap.words) {
        bitmap.words = append(bitmap.words, 0)// 分配新的 word，初始化为0
    }
    // 判断num是否已经存在bitmap中
    if bitmap.words[word]&(1<<bit) == 0 { // val & 10000 如果为 0 说明不存在
        bitmap.words[word] |= 1 << bit // 将该值置为 1 | 或运算符可以保留原数据，并将0值置为 1
        bitmap.length++             
    }
}

func (bitmap *Bitmap) Delete(num int) {
    word, bit := num/64, uint(num%64)
    for word >= len(bitmap.words) {
        return
    }
    // 判断num是否已经存在bitmap中
    fmt.Printf("1 << %d = %b\n", bit, 1<<bit)
    fmt.Printf("^(1 << %d) = %b\n", bit, uint64(^(1 << bit)))
    fmt.Printf("bitmap.words[word] = %b\n",bitmap.words[word])
    if bitmap.words[word] & 1<<bit == 0 { // val & 10000 如果为 0 说明不存在
        bitmap.words[word] &= ^(1 << bit) // 取反,相与 01111
        bitmap.length--               
    } else {
        return
    }
}

func (bitmap *Bitmap) Len() int {
    return bitmap.length
}

// 输出集合元素
func (bitmap *Bitmap) String() string {
    var buf bytes.Buffer
    buf.WriteByte('{')
    for i, v := range bitmap.words {
        if v == 0 {
            continue
        }
        for j := uint(0); j < 64; j++ { // j 是 bit 的位置
            if v&(1<<j) != 0 {
                if buf.Len() > len("{") {
                    buf.WriteByte(' ')
                }
                fmt.Fprintf(&buf, "%d", 64*uint(i)+j) // i 是 word的指针， j 是 bit 的指针
            }
        }
    }
    buf.WriteByte('}')
    fmt.Fprintf(&buf,"\nLength: %d", bitmap.length)
    return buf.String()
}
```



Refer

- [golang bitmap(位图)](https://zhuanlan.zhihu.com/p/37285799)







 