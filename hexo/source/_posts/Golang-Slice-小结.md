---
title: Golang - Slice 小结
date: 2017-06-25 21:13:30
tags:
categories: Language
---

slice 切片是对底层数组 Array 的封装，slice 就可以看作一个长度可变的数组使用。slice 可以通过两种方式定义，一种是从原数组中衍生，一种是通过make函数定义，本质上来说都是一样的，doit是在内存中通过数组的初始化的方式开辟一块内存，然后将其划分为若干个小块用来存储数组元素，然后slice就去引用整个或者局部数组元素。

- 从数组中切片构建slice

  ```
  a := [10]int {1,2,3,4,5,6,7,8,9,0}
  s := a[2 : 8]
  fmt.Println(s)
  ```

- 通过 make 函数定义 slice

  ```
  ss := make([]int, 5, 20)
  fmt.Println(ss)
  ```

  ​

**slice 的容量**，如果通过 make 函数创建 slice 的时候指定了容量参数，那内存管理器会根据指定的容量的值先划分一块儿内存空间，然后才在其中按长度，存放数组元素，多余的部分处于空闲状态。

在 slice 上追加元素的时候，首先会到这块儿空闲的内存中，如果添加的元素的个数超过了容量的值，内存管理器会重新划分一块容量值为原容量2倍大小的内存空间。append 函数返回的是一个新构造的 slice，虽然其指向的 Underlay Buffer 可能保持不变（未发生 Reallocate 情况时），因此如果要完整实现 slice 的 append，则需要将 append 的返回值拷贝给原有的 slice。

```go
a := [5]int{1, 2, 3, 4, 5}

s := a[:2]
fmt.Printf("&s=%p, &s[0]=%p, s=%x\n", &s, &s[0], s)

s1 := append(s, 13, 14, 15)
fmt.Printf("&s1=%p, &s1[0]=%p, s1=%x\n", &s1, &s1[0], s1)

s = s1
fmt.Printf("&s=%p, &s[0]=%p, s=%x\n", &s, &s[0], s)

----

&s =0xc420014580,  &s[0]=0xc420016390,  s=[1 2]
&s1=0xc4200145c0, &s1[0]=0xc420016390, s1=[1 2 d e f]
&s =0xc420014580,  &s[0]=0xc420016390,  s=[1 2 d e f]
```



<!-- more -->



**slice是引用类型**
slice 是对源数组的一个引用，改变 slice 中的元素的值，实际上就是改变原数组的元素的值。不论是append追加操作，还是赋值操作，都会影响元数组或其他引用同一数组的 slice 的值。slice 进行数组引用的时候，其实是将指针指向了内存中具体元素的地址，如数组的内存地址。



**slice 引用传递发生意外**
上边我们一直说，slice是引用类型，指向的都是内存中的同一块内存，不过实际应用中，却会发生意外，这种情况只有在像切片 append 元素的时候出现。slice 的处理机制是这样的，当 slice 的容量还有空闲的时候，append进来的元素会直接使用空闲的容量空间，但是一旦append进来的元素超过了原来指定的容量，内存管理器就会开辟一块更大的内存空间，用于存储多出来的元素，并且会将原来的元素复制一份，放到这块新开辟的内存空间。

```go
a := [5]int{1, 2, 3, 4, 5}
s := a[0:]

fmt.Printf("%p\n", s)
s = append(s, 13, 14, 15)
fmt.Printf("%p\n", s)

---

0xc420072000
0xc420078000
```



**Reference**

http://www.cnblogs.com/jshdaxia/p/5897894.html

https://stackoverflow.com/questions/16589983/are-channels-passed-by-reference-implicitly



