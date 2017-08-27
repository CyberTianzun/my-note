---
title: 斗鱼弹幕解析
date: 2017-5-19
tags: ["爬虫"]
categories: "爬虫"
---

因为前面一段时间经常做直播网站的逆向，这篇文章就简单介绍一下斗鱼直播的弹幕解析，并且讨论一下两种渠道的接入方式。<!-- more -->方法一是由斗鱼开发论坛提供的官方文档专用于给第三方接入的，方法二就是逆向斗鱼的 Web 端代码了。以下的弹幕爬虫代码全部用 `node` 实现。

## 方法一

斗鱼的开发论坛实际上提供了一份文档（[点我下载](斗鱼弹幕服务器第三方接入协议v1.4.1.pdf)），斗鱼提供了 `openbarrage.douyutv.com` 供第三方采集弹幕，虽然数据不是非常全，不过还是先从这里开始吧：

从官方文档里边的 `二、协议相关` 得知斗鱼接口协议的基本组成

<table style="text-align:center"><thead><tr><td>字节</td><td>Byte 0</td><td>Byte 1</td><td>Byte 2</td><td>Byte 3</td></tr></thead><tbody><tr>  <td>长度</td>  <td colspan="4">消息长度</td></tr><tr>  <td rowspan="2">头部</td>  <td colspan="4">消息长度</td></tr><tr>  <td colspan="2">消息类型</td>  <td>加密字段</td>  <td>保密字段</td></tr><tr>  <td>数据部分</td>  <td colspan="4">数据部分（结尾必须为"\0"）</td></tr></tbody></table>

因此来构造一个给 socket 发消息的方法

```javascript
/**
 * 发送消息
 * @param  {socket} s   socket实例
 * @param  {string} msg 消息体
 * @return {undefined}
 */
function sendData(s, msg) {
  // 字符串消息的length + 4个字节的长度 + 4个字节的长度 + 2个字节的消息类型 +
  //      1个字节的加密字段 + 1个字节的保密字段 + 1个"\0"的长度
  // 总共为 字符串消息的length + 13个字节
  let data = new Buffer(msg.length + 13);
  // 写4个字节的长度
  data.writeInt32LE(msg.length + 9, 0);
  // 写4个字节的长度
  data.writeInt32LE(msg.length + 9, 4);
  // 这个是固定的
  data.writeInt32LE(689, 8);
  // 写消息，并添加'\0'
  data.write(msg + '\0', 12);
  s.write(data);
}
```

按照文档写一下发送登录消息的方法

```javascript
function login(s, roomid = '67373') {
  sendData(s, `type@=loginreq/roomid@=${roomid}/`);
}
```

按照文档写一下发送加入房间消息的方法

```javascript
function joinGroup(s, roomid = '67373', gid = '-9999') {
  sendData(s, `type@=joingroup/rid@=${roomid}/gid@=${gid}/`);
}
```

然后就可以开始连接斗鱼的接口咯

```javascript
const net = require('net');

let s = net.connect({
  port: 8601,
  host: 'openbarrage.douyutv.com'
}, () => login(s));

s.on('data', (chunk) => console.log(chunk.toString()));

s.on('error', (err) => console.error(err));
```

然后运行一下会出现下面这样的消息：

```
���type@=loginres/userid@=0/roomgroup@=0/pg@=0/sessionid@=0/username@=/nickname@=/live_stat@=0/is_illegal@=0/ill_ct@=/ill_ts@=0/now@=0/ps@=0/es@=0/it@=0/its@=0/npv@=0/best_dlev@=0/cur_lev@=0/nrc@=3815286320/ih@=0/sid@=70103/sahf@=0/
```

看样子的登录成功了，这个响应的消息体看样子和我们发出的消息体是完全一样的格式。收到了loginres之后，来做一个joingroup吧。修改一下下面的代码：

```javascript
let chunkCache = '';
let combineBeforeChunk = false;

// 简单解析一下文档里边所谓的 STT 序列化
function wrapMessage(msg) {
  let result = { };
  let equalIndex = -1;
  for(let piece of msg.split('/')) {
    if (piece === '\u0000') { // 结束标记
      break;
    }
    equalIndex = piece.indexOf('@=');
    result[piece.substring(0, equalIndex)] = piece.substring(equalIndex + 2);
  }
  result.__type = result.type;
  return result;
}

// 处理好的消息
function receivedMessage(msg) {
  if (msg.type == 'chatmsg') {
    console.log(`${msg.nn}: ${msg.txt}`);
  } else {
    // 其他未知类型的消息
  }
}

// 处理socket收到的chunk
s.on('data', (chunk) => {
  let msg = chunk.slice(12).toString();
  if (msg.startsWith('type@=loginres')) {
    joinGroup(s);
  } else {
    if (msg.endsWith('\u0000')) {
      // 需要合并 chunk
      if (combineBeforeChunk === true) {
        receivedMessage(wrapMessage(chunkCache + msg));
        chunkCache = '';
        combineBeforeChunk = false;
      } else {
        receivedMessage(wrapMessage(msg));
      }
    } else {
      combineBeforeChunk = true;
      chunkCache += msg;
    }
  }
});
```

## 方法二

之所以会有方法二是因为，在方法一中的消息体中发现，虽然弹幕消息是比较完整的，但是缺少了很多一些客户端或者web端才有的消息，比如一些特殊的礼物、火箭、广播等等。因此逆向了web端，来尝试获取更加详细的数据。

首先在网站的代码中，在网页上找到了礼物信息和服务器的入口，下面用代码把这些数据取出来：

```javascript
const should = require('should'),
  request = require('request');

let roomurl = 'https://www.douyu.com/3q3q';
let roomid;
let roomstatus;
let servers;
let gifts;
let danmuServers;

request({
  uri: roomurl,
  callback: (err, res, body) => {
    res.statusCode.should.be.exactly(200);

    roomid = body.match(/\\?"room_id\\?":(\d+),/)[1];
    roomstatus = body.match(/\\?"show_status\\?":(\d+),/)[1];
    servers = JSON.parse(unescape(body.match(/\\?"server_config\\?":\\?"([.%0-9A-Za-z]+)\\?"/)[1]));
    let giftstring = body.match(/\$ROOM\.giftBatterConfig = (.*);/);
    if (giftstring) {
      gifts = JSON.parse(giftstring[1]);
    } else {
      giftstring = body.match(/"\$ROOM.giftList": "(.+)",/);
      if (giftstring) {
        gifts = JSON.parse(unescape(giftstring[1]));
      }
    }
    console.log(roomid);
    console.log(roomstatus);
    console.log(servers);
    console.log(gifts);
  }
});
```

拿到了一系列的服务器的ip和端口，理论上来说，在大批量连接的时候应该做一下均负载处理，不过按照逆向出来的结果，连上去并且发送签名：

```javascript
let s = net.connect({
  host: servers[0].ip,
  port: servers[0].port
}, () => {
  // handle login
  let ts = parseInt(Date.now() / 1000);
  let dv = uuid.v1().replace(/-/g,'').toUpperCase();
  let vk = md5(ts + "7oE9nPEG9xXV69phU31FYCLUagKeYtsF" + dv);
  send(s, `type@=loginreq/username@=/ct@=0/password@=/roomid@=${roomid}/devid@=${dv}/rt@=${ts}/vk@=${vk}/ver@=20150929/`);
});
let completeMsg = [ ];
let currentSize = 0;
s.on('data', (chunk) => {
  if (chunk.readUInt8(chunk.length - 1) === 0x00 && chunk.readUInt8(chunk.length - 2) === 0x2f) {
    // is end message
    if (completeMsg.length === 0) {
      unwrapMessages(chunk);
    } else {
      completeMsg.push(chunk);
      currentSize += chunk.length;
      unwrapMessages(Buffer.concat(completeMsg, currentSize));
      completeMsg = [ ];
      currentSize = 0;
    }
  } else {
    completeMsg.push(chunk);
    currentSize += chunk.length;
  }
});
s.on('error', (err) => {
  console.error(err);
});
s.on('close', () => {
  console.error('close');
});

function unwrapMessages(buf) {
  let data = buf;
  while(data.length > 13) {
    let length = data.readInt32LE(0) - 9;
    let msg = data.slice(12, length + 13);
    processMessage(parseSingleLayoutSTT(msg.toString()));
    data = data.slice(length + 13);
  }
}

function parseSingleLayoutSTT(msg) {
  let result = { };
  let equalIndex = -1;
  for(let piece of msg.split('/')) {
    if (piece === '\u0000' || piece.length === 0) { // 结束标记
      break;
    }
    equalIndex = piece.indexOf('@=');
    result[piece.substring(0, equalIndex)] = piece.substring(equalIndex + 2);
  }
  return result;
}

function processMessage(m) {
  if (m.type == 'setmsggroup') {
    if (/@S/.test(m.gid)) {
      gid = m.gid.split('@S')[0];
    } else {
      gid = m.gid;
    }
  } else if (m.type == 'msgrepeaterlist') {
    danmuServers = [ ];
    for(let piece of m.list.replace(/@S/g, '/').replace(/@A/g, '@').split('/')) {
      if (piece === '\u0000' || piece.length === 0) { // 结束标记
        break;
      }
      danmuServers.push(parseSingleLayoutSTT(piece.replace(/@S/g, '/').replace(/@A/g, '@')));
    }
  }
}
```

在与上面端口的通信过程中，拿到了服务器分配的 gid 和真实的弹幕服务器地址和端口号，才是接收弹幕的开始：

```javascript
let s = net.connect({
  host: danmuServers[0].ip,
  port: danmuServers[0].port
}, () => {
  // handle login
  send(s, `type@=loginreq/username@=visitor34807350/password@=1234567890123456/roomid@=${roomid}/`);
  // send(s, `type@=loginreq/roomid@=${roomid}/`);
  setTimeout(() => {
    // send(s, `type@=joingroup/rid@=${roomid}/gid@=${gid}/`);
    send(s, `type@=joingroup/rid@=${roomid}/gid@=-9999/`);
    // send(s, `type@=joingroup/rid@=${roomid}/gid@=${gid}/`);
  }, 3000);
});
let completeMsg = [ ];
let currentSize = 0;
s.on('data', (chunk) => {
  if (chunk.readUInt8(chunk.length - 1) === 0x00 && chunk.readUInt8(chunk.length - 2) === 0x2f) {
    // is end message
    if (completeMsg.length === 0) {
      unwrapMessages(chunk);
    } else {
      completeMsg.push(chunk);
      currentSize += chunk.length;
      unwrapMessages(Buffer.concat(completeMsg, currentSize));
      completeMsg = [ ];
      currentSize = 0;
    }
  } else {
    completeMsg.push(chunk);
    currentSize += chunk.length;
  }
});
let timer = setInterval(() => {
  let ts = parseInt(Date.now() / 1000);
  send(s, `type@=keeplive/tick@=${ts}/`);
}, 40000);
s.on('error', (err) => {
  clearInterval(timer);
});
s.on('close', () => {
  console.error('close');
});
```

简单来说，有几个重要的地方需要注意，斗鱼有两个长连接需要保持，第一个长连接会以心跳的形式刷新斗鱼官方页面上的那个在线人数，而第二个长连接则包含了所有的数据信息。至于 demo 的代码，有空再贴到 github 吧，感兴趣的可以 follow 一下，不确定具体时间。贴一个简单的结果图：

![](effect.png)

PS: Q叔的直播间大半夜的人也真多啊！
