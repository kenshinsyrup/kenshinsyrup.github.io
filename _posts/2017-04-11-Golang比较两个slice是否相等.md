---
layout: post #post
title: Golang比较两个slice是否相等 #post title
categories: Program #post category, seperated by space
tags: Golang #post tag, seperated by space
---

开发中经常会遇到需要比较两个slice包含的元素是否相同的情况，一般来说有两个思路：

- ``reflect``比较的方法
- 循环遍历比较的方法

这里用检查两个字符串slice是否相等的例子来测试一下这两种思路的效率<strike>我当然知道你知道reflect方法效率更差啦</strike>

## reflect比较的方法

```
func StringSliceReflectEqual(a, b []string) bool {
    return reflect.DeepEqual(a, b)
}
```

这个写法很简单，就是直接使用``reflect``包的``reflect.DeepEqual``方法来比较``a``和``b``是否相等

这是我最初完成这个需求的方式，年轻嘛，比较天真，觉得``reflect``啊，高端大气，而且一行代码搞定，简洁有力，给自己默默的点个赞

## 循环遍历比较的方法

```
func StringSliceEqual(a, b []string) bool {
    if len(a) != len(b) {
        return false
    }

    if (a == nil) != (b == nil) {
        return false
    }

    for i := range a {
        if a[i] != b[i] {
            return false
        }
    }

    return true
}
```

以上是我们项目中的使用的用来比较字符串slice是否相等的一个函数，代码逻辑很简单；先比较长度是否相等，``否``则``false``；再比较两个slice是否都为nil或都不为nil，``否``则``false``；再比较对应索引处两个slice的元素是否相等，``否``则``false``；前面都为``是``则``true``

需要注意

```
if (a == nil) != (b == nil) {
    return false
}
```

这段代码是必须的，虽然如果没有这段代码，在大多数情况下，上面的函数可以正常工作，但是增加这段代码的作用是与``reflect.DeepEqual``的结果保持一致：``[]int{} != []int(nil)``

## benchmark测试效率

我们都知道Golang中reflect效率很低，所以虽然循环遍历的方法看起来很啰嗦，但是如果真的效率比reflect方法高很多，就只能忍痛放弃reflect了

使用benchmark来简单的测试下二者的效率

benchmark StringSliceEqual

```
func BenchmarkEqual(b *testing.B) {
    sa := []string{"q", "w", "e", "r", "t"}
    sb := []string{"q", "w", "a", "s", "z", "x"}
    b.ResetTimer()
    for n := 0; n < b.N; n++ {
        StringSliceEqual(sa, sb)
    }
}
```

benchmark StringSliceReflectEqual

```
func BenchmarkDeepEqual(b *testing.B) {
    sa := []string{"q", "w", "e", "r", "t"}
    sb := []string{"q", "w", "a", "s", "z", "x"}
    b.ResetTimer()
    for n := 0; n < b.N; n++ {
        StringSliceReflectEqual(sa, sb)
    }
}
```

上面两个函数中，``b.ResetTimer()``一般用于准备时间比较长的时候重置计时器减少准备时间带来的误差，这里可用可不用

在测试文件所在目录执行``go test -bench=.``命令

![benchmark对比测试结果](http://i32.photobucket.com/albums/d1/kenshinsyrup/Kenshinsyrup/2017-04-11-post01/benchmark_zpscf8uwozk.png)

在我的电脑，使用循环遍历的方式，3.43纳秒完成一次比较；使用reflect的方式，208纳米完成一次操作，效率对比十分明显

## BCE优化

Golang提供BCE特性，即[Bounds-checking elimination](https://en.wikipedia.org/wiki/Bounds-checking_elimination)，关于Golang中的BCE，推荐一篇大牛博客[Bounds Check Elimination (BCE) In Golang 1.7](http://www.tapirgames.com/blog/golang-1.7-bce)

```
func StringSliceEqualBCE(a, b []string) bool {
	if len(a) != len(b) {
		return false
	}

	if (a == nil) != (b == nil) {
		return false
	}

	b = b[:len(a)]
	for i, v := range a {
		if v != b[i] {
			return false
		}
	}

	return true
}

```

上述代码通过``b = b[:len(a)]``处的bounds check能够明确保证``v != b[i]``中的``b[i]``不会出现越界错误

类似的，完成benchmark函数

```
func BenchmarkEqualBCE(b *testing.B) {
	sa := []string{"q", "w", "e", "r", "t"}
	sb := []string{"q", "w", "a", "s", "z", "x"}
	b.ResetTimer()
	for n := 0; n < b.N; n++ {
		StringSliceEqualBCE(sa, sb)
	}
}
```

在测试文件所在目录执行``go test -bench=.``命令

![benchmark对比测试结果](http://i32.photobucket.com/albums/d1/kenshinsyrup/Kenshinsyrup/2017-04-11-post01/benchmark1_zpseuvqxkor.png)

看起来提升并不明显啊 （╯‵□′）╯︵┴─┴ 

但是随着Golang的优化，应该会越来越明显吧  ┬─┬ ノ( ' - 'ノ)

## 结论

推荐使用StringSliceEqualBCE形式的比较函数<strike>反正不用reflect比较就不会被领导骂被同事喷</strike>

代码已经上传到了我的Github[仓库](https://github.com/kenshinsyrup/AllGolangDemo/tree/master/CompareSlice)可以下载测试








