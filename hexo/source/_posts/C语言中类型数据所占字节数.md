---
title: C语言中类型数据所占字节数
date: 2016-08-19 00:02:52
tags:
categories: 内核
---

类型所占字节数和机器字长及编译器有关系，所以，`int`，`long int`，`short int`的宽度都可能随编译器而异。但有几条铁定的原则（**ANSI / ISO** 制订的）：

1. `sizeof(short int) <= sizeof(int)`
2. `sizeof(int) <= sizeof(long int)`
3. `short int` 至少应为 16位（2字节）
4. `long int` 至少应为 32位。


## 16位 编译器

| 类型               | 字节长度 |
|:------------------|---------|
|`char`             | 1个字节  |
|`void *`(指针变量)  | 2个字节  |
|`short int`        | 2个字节  |
|`int`              | 2个字节  |
|`unsigned int`     | 2个字节  |
|`float`            | 4个字节  |
|`double`           | 8个字节  |
|`long`             | 4个字节  |
|`unsigned long`    | 4个字节  |
|`long long`        | 8个字节  |


## 32位编译器

| 类型               | 字节长度 |
|:------------------|---------|
|`char`             | 1个字节  |
|`void *`(指针变量)  | 4个字节  |
|`short int`        | 2个字节  |
|`int`              | 4个字节  |
|`unsigned int`     | 4个字节  |
|`float`            | 4个字节  |
|`double`           | 8个字节  |
|`long`             | 4个字节  |
|`unsigned long`    | 4个字节  |
|`long long`        | 8个字节  |


## 64位编译器

| 类型               | 字节长度 |
|:------------------|---------|
|`char`             | 1个字节  |
|`void *`(指针变量)  | 8个字节  |
|`short int`        | 2个字节  |
|`int`              | 4个字节  |
|`unsigned int`     | 4个字节  |
|`float`            | 4个字节  |
|`double`           | 8个字节  |
|`long`             | 8个字节  |
|`unsigned long`    | 8个字节  |
|`long long`        | 8个字节  |
