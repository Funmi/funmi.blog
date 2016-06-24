---
title: App开发过程中常见问题分析
date: 2016-06-24 09:38:04
tags:
   - 问题分析
---

## 数组越界

### 取值写死了

之前调试的时候，取了三条数据，所以取值的时候就按照index,0,1,2来取值，忽略了后台没有返回值的情况下去取值就会报数组越界。例如：
``` objc
NSMutableArray *array = [[NSMutableArray alloc]initWithObjects:@1,@2,@3, nil];
NSNumber *number = [array objectAtIndex:4];
```

### for循环取值越界
列表中时count+1行，然而取值时还是按照顺序来取值。
<!-- more -->
例如：
``` objc
NSMutableArray *array = [[NSMutableArray alloc]initWithObjects:@1,@2,@3, nil];
NSInteger count = array.count;
for (NSInteger i = 1; i <= count; i++) {
    NSLog(@"%@", [array objectAtIndex:i]);
}
```
for循环取值是下标是[0, count-1]的闭区间。

### 数据返回为空
数据返回为空是指后台返回的数据数组是空的，然而没有做空判断就去取值，导致数组越界的异常。

所以取值前需要先判断数组的长度然后再根据数组长度来取值。
``` objc
if (array && array.count > 5) {
    NSLog(@"%@", [array objectAtIndex:5])
}
```
### 多层for循环取值混乱

嵌套循环取值，取值错误把i取成了j，j取成了i，这是需要特别小心的地方。比如下面这个例子：

``` objc
NSMutableArray *arrayA = [[NSMutableArray alloc]initWithObjects:@1,@2,@3,@4, nil];
NSMutableArray *arrayB = [[NSMutableArray alloc]initWithObjects:@1,@2,@3, nil];
NSInteger i = 0, j = 0;
for (; i <= arrayA.count; i++) {
  NSLog(@"%@", [arrayA objectAtIndex:i]);
  for (; j < arrayB.count; j++) {
    NSLog(@"%@", [arrayB objectAtIndex:i]);
  }
}
```

## 页面刷新问题

比较常见的页面刷新问题主要有，当你做了某些操作时，，其它页面的相关联的数据需要得到及时的刷新。

## 内存溢出问题

### 创建的实例没有释放
这里举个例子，体现得最明显的地方就是地图的释放，如果你在离开这个页面的时候不释放地图控件，当你来回切换地图界面和非地图界面时将会出现卡死的情况。

### block的使用
oc情况下使用block使用self.xx导致循环引用无法释放。
### 代理的使用
oc情况下声明代理的使用用了strong而不是assign。
## 逻辑处理
1. 情况考虑过少。
2. 情况复杂就先放下，抱有侥幸心理。

## 后台的一些问题
很多时候App的一些错误都是由后台造成的，比较常见的问题有：
### 返回字段不统一
比如说同样一个对象User，有时候返回userName,有时侯返回name。
### 返回状态码不统一
一会儿成功状态码是200，一会儿是300。
### 字段名称更改
之前调试都是好好的，突然一下没有获取到值了，原因很有可能就是后台接口返回的字段改变了。
> 注意：后台字段一定要统一不要随意更改，若有更改需要通知App组相关开发人员。

### 字段删减
这个问题和上面差不多，返回突然少了一个字段。
### 分页
后台设计接口的时候一定要考虑到分页，只要有列表的页面都应该考虑到分页的情况，不要到后面才想起要加分页，这个问题要从一开始设计接口的时候就要尽量避免。

## 需求理解

这个是最要命的问题，人家要做一辆火车，而你需求理解错误造出了一辆汽车。到了后期交付的时候，这可怎么交付啊！

### 没有看原型图
很多时候，原型图都还没有理解透彻就已经开始在编码了，这会出现什么情况呢？这有可能会直接导致你写的程序就是bug，没有一个功能是可以交付的。
### 前后台理解不一致
