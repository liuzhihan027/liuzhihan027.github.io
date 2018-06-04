---
layout: post
title:  "hive防止数据倾斜"
categories: Hive
tags:  hive sql
author: ZhL
---

* content
{:toc}

# 1. 什么是数据倾斜?
所谓数据倾斜就是在map/reduce程序执行时，reduce的节点大部分执行完毕，但是有一个或者几个reduce节点运行很慢，导致整个程序的处理时间很长，这是因为某一个key的条数比其他key多很多（有时是百倍或者千倍之多），这条key所在的reduce节点所处理的数据量比其他节点就大很多，从而导致某几个节点迟迟运行不完.



# 2. 解决办法

本人遇到过很多数据倾斜的情况,也在网上查阅过很多资料,主要分为以下几个解决方法:

## 2.1 参数调节

几个常用的参数

>- hive.map.aggr=true;

Map 端部分聚合，相当于Combiner

>- hive.groupby.skewindata=true;

针对GROUP BY过程的数据倾斜优化参数

>- hive.exec.reducers.max=500000;

控制最大reduce的数量,默认值为999,一般来说reduce的个数是由map的大小来决定的,当map数比较小时此参数无效,reduce越多资源消耗越多

>- hive.exec.reducers.bytes.per.reducer=640000000;

这个参数控制一个job会有多少个reducer来处理，依据的是输入文件的总大小,默认1GB。


## 2.2 SQL调节

在sql的层面对语句进进行调节:

- JOIN操作

驱动表去重,去特殊值,在不影响业务的情况下尽量减小驱动表,选取或构造key相对均匀的数据进行join

- 大表JOIN小表

如果小表很小(一般表达小<=25m),在主语句中使用 **/*+ mapjoin(b)*/** 将小表载入内存进行计算,此时整个job在map阶段就进行了join操作,reducer个数为0,一般hive中能够配置自动识别join过程中的表的大小将小表载入内存(小表默认大小为25m).

当两个表都是很大的情况下,mapjoin参数无效,如果想要强制将其中一个表载入内存可使用:

>- hive.mapjoin.smalltable.filesize=25000000;

此参数设置可以使用mapjoin的小表的最大占用空间(慎用),如将一非常大的表载入内存会影响整个集群的运行,不要问我怎么知道的.

- COUNT DISTINCT 优化

此语句会将表中的数据(或上一步得到的数据)全部加载到一个reducer上执行,效果可想而知,可使用group by方式进行优化:

原语句:

```sql

SELECT key,COUNT(DISTINCT value) AS nvalue
FROM (
  SELECT key,value
  FROM table
)a
GROUP BY key;

```

GROUP BY修改后的语句:
```sql

SELECT key,COUNT(1) AS nvalue
FROM (
  SELECT key,value
  FROM table
  GROUP BY key,value
)a
GROUP BY key;

```

由于使用了group by 语句将数据经suffer过程分发到了不同的reducer上执行,并发度提高,每一个reducer上处理的数据量相对较小,效率大大提升

- GROUP BY 维度过小优化

如果数据量较大,key值过小,在使用group by 的时候也会导致单reducer上处理的数据过大,效率缓慢,对于这种情况可添加一个sum(函数)进行优化:

```sql

SELECT key, sum(nvalue)
FROM (
  SELECT key, substring(value ,0,4) sub_value, count(1) nvalue  
  FROM (
    SELECT key,value
    FROM table
    GROUP BY key,value
  ) aa
  GROUP BY key, substring(value ,0,4)
) b
GROUP BY key;

```

此语句的思想是将整体去重的数据以value截取的部分内容作为中间value,将key和sub_value的整体作为key进行聚合,统计数目,之后将同一个key下的值使用sum()函数进行相加,这样使整合成的整体的key分布相对均匀,能够有效避免key少value多的情况.





## 代码

获取剩余时间的代码如下：

```js
/**
 * 获取剩余时间
 * @param  {Number} endTime    截止时间
 * @param  {Number} deviceTime 设备时间
 * @param  {Number} serverTime 服务端时间
 * @return {Object}            剩余时间对象
 */
let getRemainTime = (endTime, deviceTime, serverTime) => {
    let t = endTime - Date.parse(new Date()) - serverTime + deviceTime
    let seconds = Math.floor((t / 1000) % 60)
    let minutes = Math.floor((t / 1000 / 60) % 60)
    let hours = Math.floor((t / (1000 * 60 * 60)) % 24)
    let days = Math.floor(t / (1000 * 60 * 60 * 24))
    return {
        'total': t,
        'days': days,
        'hours': hours,
        'minutes': minutes,
        'seconds': seconds
    }
}
```

<del>获取服务器时间可以使用 mtop 接口 `mtop.common.getTimestamp` </del>

然后可以通过下面的方式来使用：

```js
// 获取服务端时间（获取服务端时间代码略）
getServerTime((serverTime) => {

    //设置定时器
    let intervalTimer = setInterval(() => {

        // 得到剩余时间
        let remainTime = getRemainTime(endTime, deviceTime, serverTime)

        // 倒计时到两个小时内
        if (remainTime.total <= 7200000 && remainTime.total > 0) {
            // do something

        //倒计时结束
        } else if (remainTime.total <= 0) {
            clearInterval(intervalTimer);
            // do something
        }
    }, 1000)
})
```

这样的的写法也可以做到准确倒计时，同时也比较简洁。不需要隔段时间再去同步一次服务端时间。

## 补充

在写倒计时的时候遇到了一个坑这里记录一下。

**千万别在倒计时结束的时候请求接口**。会让服务端瞬间 QPS 峰值达到非常高。

![](https://img.alicdn.com/tfs/TB1LBzjOpXXXXcnXpXXXXXXXXXX-154-71.png)

![image](https://github.com/liuzhihan027/liuzhihan027.github.io/raw/master/)

如果在倒计时结束的时候要使用新的数据渲染页面，正确的做法是：

在倒计时结束前的一段时间里，先请求好数据，倒计时结束后，再渲染页面。

关于倒计时，如果你有什么更好的解决方案，欢迎评论交流。
