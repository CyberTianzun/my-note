---
title: 爬虫抓取同花顺新三板数据时Token的获取流程
date: 2017-11-29
tags: "爬虫"
categories: "爬虫"
---

在控制台中，我使用了一下语句来尝试请求数据

```javascript
$.ajax({jsonp:false, url: 'http://d.10jqka.com.cn/v2/fiverange/sb_430127/last.js', dataType: "jsonp",jsonpCallback: 'quotebridge_v2_fiverange_sb_430127_last',success: function(d, status, xhr){ console.log(d); }})
```

神奇的事情发生了，每一次删掉了cookie中的v字段请求，请求之后就会重新附带上v字段。并且，断网，不发请求也会刷新v字段，看来v字段是一个本地计算的token了。然后我用一个最简单的办法来排除生成本地token的代码在哪里。。把所有的js应用都去掉，逐个加上，找到了代码在 `http://s.thsi.cn/js/chameleon/chameleon.min.1511844.js` 这个文件。把代码做简单的格式化后找到生成token的入口在以下位置:

![Init方法](pic01.png)
![Update方法](pic02.png)

首先看到的就是那个非常显然的 `yr.encode()`，在此之前，构造了一个叫做 O 的数组，下面先要把 O 数组里边是什么东西搞清楚。我使用了最笨的办法，把下标通通打印到控制台，对比每一项都是什么意义：

![](pic03.png)
![](pic04.png)

然后大致有这么一个结论

| 序号 | 含义 |
|:--|:--|
| 0 | 随机数，31位正整数 |
| 1 | 服务器时间戳，固定值 1511843501 |
| 2 | 本地时间戳 |
| 3 | strhash(navigator.userAgent), 对navigator.userAgent做了一下strhash，我的为 1300455701 ，浏览器的userAgent为 "Mozilla/5.0 (Macintosh; Intel Mac OS X 10_12_6) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/62.0.3202.94 Safari/537.36" |
| 4 | br.getPlatform() |
| 5 | br.getBrowserIndex() |
| 6 | br.getPluginNum() |
| 7 | Ar.getMouseMove() |
| 8 | Ar.getMouseClick() |
| 9 | Ar.getMouseWhell() |
| 10 | Ar.getKeyDown() |
| 11 | Ar.getClickPos().x |
| 12 | Ar.getClickPos().y |
| 13 | br.getBrowserFeature() |
| 14 | 未知，默认给0 |
| 16 | 每次更新累加，至少为1 |
| 17 | 未知，默认给0 |

构造出一个长度为18的数组之后，需要写进一个长度为43的字节流里边，每个字段的长度按照 `[4, 4, 4, 4, 1, 1, 1, 3, 2, 2, 2, 2, 2, 2, 2, 4, 2, 1]` 来一一对应。

然后现在需要分析和还原 `Er.encode(r)` 做了什么，跟踪了代码到，encode方法实际在：

![encode方法](pic05.png)
![yr方法表](pic06.png)

从encode方法所在的对象向外暴露的名字来看, `function U` 是 `encode`, `function H` 叫做 `base64Encode`。大致还原里面的内容之后整理出来的算法如下：

```javascript
const data = [174, 45, 244, 35, 90, 28, 230, 173, 90, 30, 17, 21, 77, 131, 97, 21, 7, 10, 4, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 14, 108, 0, 0, 0, 0, 0, 0, 0, 1, 0];

function U(r) {
    // function G
    var B = 0;
    for (var A = 0; A < r.length; A++) {
        B = (B << 5) - B + r[A];
        console.log(B);
    }
    var u = B & 0xff;
    console.log(u);

    var c = [2, u];

    // function X
    for (var n = 0, e = 2; n < r.length; n++) {
        c[e++] = r[n] ^ u & 0xff;
        u = ~(u * 131);
    }

    return (new Buffer(c)).toString('base64').replace('+', '-').replace('/', '_');
}

console.log(U(data));
```

使用新生成的token附带在Cookie里边的v字段试一试能不能请求成功吧：

```bash
curl 'http://d.10jqka.com.cn/v2/fiverange/sb_430127/last.js' \
-H 'DNT: 1' \
-H 'Accept-Encoding: gzip, deflate' \
-H 'Accept-Language: en,zh-CN;q=0.9,zh;q=0.8,zh-TW;q=0.7,ja;q=0.6' \
-H 'User-Agent: Mozilla/5.0 (Macintosh; Intel Mac OS X 10_12_6) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/62.0.3202.94 Safari/537.36' -H 'Accept: */*' \
-H 'Referer: http://stockpage.10jqka.com.cn/430127/' \
-H 'Cookie: v=Ao3xjb3H63Bd409ecML012JjnagHasE8S54lEM8SySSTxqMUl7rRDNvuNeFf' \
-H 'Proxy-Connection: keep-alive' \
-H 'If-Modified-Since: Tue, 28 Nov 2017 04:19:49 GMT' \
--compressed
```

返回下面数据

```
quotebridge_v2_fiverange_sb_430127_last({"items":{"24":"9.00","25":"4000.00","26":"8.98","27":"8000.00","28":"8.97","29":"3000.00","150":"8.87","151":"8000.00","154":"6.10","155":"1000.00","30":"9.00","31":"3000.00","32":"9.06","33":"2000.00","34":"9.10","35":"10930.00","152":"9.18","153":"109000.00","156":"9.39","157":"1000.00","6":"9.06"}})
```

大功告成了，oye!

