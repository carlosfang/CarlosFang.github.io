---
layout: post
title: golang time adddate 
date: 2024-04-11 9:08:12
categories: 开发 
tags:  golang 开发小经验
excerpt: time 的 adddate 方法出现的一个问题
---

周末线上出现一个问题，3月份的订单没有算出来，从数据库表数据上看，订单数据已经是完成状态，但数据为空。

业务上的逻辑是，每天会计算上个月的订单和当月的订单和。一月的订单周期是一个有1日12点到下个月的1日12点。

如前当3月31日，上个月的周期是2月1日12点到3月1日12点；本月的周期是3月1点12到当前时间点。 

逻辑在处理上个月的订单时用了 `AddDate`

```go
lastMonth := time.AddDate(0, -1, 0).Format("200601")
```

问题就出现在这里。 当前时间是3月31时，time.AddDate(0, -1, 0) 之后，是 20240302 ，还是3月份，并不符合预期。

如下代码：

```go
package main

import (
	"fmt"
	"time"
)

func main() {

	today := time.Date(2024, 2, 29, 0, 0, 0, 0, time.Local)
	nextDay := today.AddDate(0, -1, 0)
	fmt.Printf("today :%v nextDay :%+v \n", today.Format("20060102"), nextDay.Format("20060102"))

	today = time.Date(2024, 3, 31, 0, 0, 0, 0, time.Local)
	nextDay = today.AddDate(0, -1, 0)
	fmt.Printf("today :%v nextDay :%+v \n", today.Format("20060102"), nextDay.Format("20060102"))

	today = time.Date(2024, 4, 30, 0, 0, 0, 0, time.Local)
	nextDay = today.AddDate(0, -1, 0)
	fmt.Printf("today :%v nextDay :%+v \n", today.Format("20060102"), nextDay.Format("20060102"))

	today = time.Date(2024, 5, 31, 0, 0, 0, 0, time.Local)
	nextDay = today.AddDate(0, -1, 0)
	fmt.Printf("today :%v nextDay :%+v \n", today.Format("20060102"), nextDay.Format("20060102"))

	today = time.Date(2024, 6, 30, 0, 0, 0, 0, time.Local)
	nextDay = today.AddDate(0, -1, 0)
	fmt.Printf("today :%v nextDay :%+v \n", today.Format("20060102"), nextDay.Format("20060102"))

	today = time.Date(2024, 7, 31, 0, 0, 0, 0, time.Local)
	nextDay = today.AddDate(0, -1, 0)
	fmt.Printf("today :%v nextDay :%+v \n", today.Format("20060102"), nextDay.Format("20060102"))

	today = time.Date(2024, 8, 31, 0, 0, 0, 0, time.Local)
	nextDay = today.AddDate(0, -1, 0)
	fmt.Printf("today :%v nextDay :%+v \n", today.Format("20060102"), nextDay.Format("20060102"))

	today = time.Date(2024, 9, 30, 0, 0, 0, 0, time.Local)
	nextDay = today.AddDate(0, -1, 0)
	fmt.Printf("today :%v nextDay :%+v \n", today.Format("20060102"), nextDay.Format("20060102"))

	today = time.Date(2024, 10, 31, 0, 0, 0, 0, time.Local)
	nextDay = today.AddDate(0, -1, 0)
	fmt.Printf("today :%v nextDay :%+v \n", today.Format("20060102"), nextDay.Format("20060102"))

	today = time.Date(2024, 11, 30, 0, 0, 0, 0, time.Local)
	nextDay = today.AddDate(0, -1, 0)
	fmt.Printf("today :%v nextDay :%+v \n", today.Format("20060102"), nextDay.Format("20060102"))

	today = time.Date(2024, 12, 31, 0, 0, 0, 0, time.Local)
	nextDay = today.AddDate(0, -1, 0)
	fmt.Printf("today :%v nextDay :%+v \n", today.Format("20060102"), nextDay.Format("20060102"))

}

```

输出：

```bash 
today :20240229 nextDay :20240129
today :20240331 nextDay :20240302
today :20240430 nextDay :20240330
today :20240531 nextDay :20240501
today :20240630 nextDay :20240530
today :20240731 nextDay :20240701
today :20240831 nextDay :20240731
today :20240930 nextDay :20240830
today :20241031 nextDay :20241001
today :20241130 nextDay :20241030
today :20241231 nextDay :20241201
```
其中，如  20240331、20240531、20240731、20241031 这一些时算的上月时间就出错，它依然是本月的时间。

解决这个问题最简单，最快的方式是引用库 [https://github.com/Andrew-M-C/go.timeconv](https://github.com/Andrew-M-C/go.timeconv)

```go
package main

import (
	"fmt"
	"time"
	"github.com/Andrew-M-C/go.timeconv"
)

func main() {
	t := time.Date(2019, 1, 31, 0, 0, 0, 0, time.UTC)
	nt := t.AddDate(0, 1, 0)	// Add one month
	fmt.Printf("%v\n", nt)		// 2019-03-03 00:00:00 +0000 UTC, not expected

	nt = timeconv.AddDate(t, 0, 1, 0)
	fmt.Printf("%v\n", nt)		// 2019-02-28 00:00:00 +0000 UTC
}
```


----
1、令人困惑的 Go time.AddDate [https://learnku.com/articles/71760](https://learnku.com/articles/71760)

2、Go time 包中的 AddDate 的逻辑避坑指南 [https://cloud.tencent.com/developer/article/1803695](https://cloud.tencent.com/developer/article/1803695)