---
title: "elasticsearch-bulk"
date: 2019-11-18T10:56:59+08:00
tags: ["elasticsearch"]
categories: ["elasticsearch"]
draft: false
---
### elasticsearch 使用rest-client进行批量操作性能测试

    之前做交易所项目， 大量实时数据存储在mongodb中，导致在查询账单、K线会出现查询慢的情况。在优化mongodb的同时，寻找其他
    nosql数据库做备用，所以才有了本文的性能测试。

##### elasticsearch rest client选择
    
    low level rest client
        1、对elasticsearch版本要求低，封装组件之后，很多项目后续都可以使用；凭借原生json语句。
    
    high level rest client
        1、对elasticsearch版本要求相当高，必须一致，引入的elasticsearch版本也要一致，否则容易报错；
        笔者因为这个吃了不少亏。组件是封装好的，直接拿来使用即可。
    
    在测试环节，elasticsearch使用的版本是7.1.0， 笔者优先使用了low level rest client，对其进行了原生json语句封装。在测试基本创建和其他基本操作时，
    效果都不错，但是在批量操作时发现执行比较慢，只能进行优化。

#### elasticsearch批量操作优化

    1. 调整刷新频率 refresh_interval = -1，手动刷新
    2. 仅备份数据时，可以把副本先关掉，完成后再打开 "number_of_replicas":0
    3. 每次数据不能超过5-15MB,数量最好不要超过1000条。

#### 测试结果

    1、 low client:   每个文档大约0.016MB
        1. 500个        200ms     
        2. 1000个       500ms
        3. 2000个       1100ms
        4. 4000个       2300ms
    可以看出来 随着文档树越多，时间也越长。 如果es要引入交易所，批量必须要达到10000-20000/s,  所以这个不能满足.

    2、high client： 每个文档大约0.016MB
    同步请求
        1. 500个        250ms
        2. 1000个       480ms
        3. 2000个       970ms
        4. 4000个       1900ms
    
    异步请求  适合做数据备份或者拷贝
        1. 500个        17ms    后台处理回调数据
        2. 1000个       46ms 
        3. 2000个       77ms
        4. 4000ge       180ms


#### 结果

    笔者最终没有使用elasticsearch，场景所需的条件达不到，只能先放弃，此处仅是记录批量操作性能，方便后期使用。