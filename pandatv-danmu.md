---
title: 熊猫直播弹幕解析
date: 2017-8-28
tags: ["爬虫"]
categories: "爬虫"
---

王校长的熊猫直播太有名气了，这一篇就讲讲熊猫直播的弹幕怎么用代码解析。<!-- more -->

找入口这事情实在是太快了，打开熊猫tv的web端，直接就发现了这个东西（如下图）。。。第一部这么快就完成了。。。这个 chat_addr_list 就是聊天室的入口地址。

![](panda-danmu-1.png)

```javascript
const request = require('request');
let chatRoomInfo = null;
let roomid = '843942';
request({
  uri: `http://riven.panda.tv/chatroom/getinfo?roomid=${roomid}`
}, (err, res, body) => {
  if (!err && res && res.statusCode === 200) {
    let roomInfo = null;
    try {
      roomInfo = JSON.parse(body);
    } catch (e) {
    }
    if (roomInfo && roomInfo.errno === 0) {
      chatRoomInfo = roomInfo.data;
    }
  }
});
```

然后就是尝试连接这个 chatroom 服务器了

```javascript
const net = require('net'),
      url = require('url');
let uri = url.parse('socket://' + chatRoomInfo.chat_addr_list[0]);
let s = net.connect({
  port: uri.port,
  host: uri.hostname
}, () => {
  // 发送登录信息
  handleLogin(s);

  // 发送心跳数据，30秒一次
  setInterval(() => handleHeartbeat(s), 30000);
});

// 用wireshark监视熊猫web端的代码发送的消息体看，大致的协议如下，有消息体+6的长度构成
function sendData(s, msg) {
  let data = new Buffer(msg.length + 6);
  data.writeInt16BE(6, 0);
  data.writeInt16BE(2, 2);
  data.writeInt16BE(msg.length, 4);
  data.write(msg, 6);
  s.write(data);
}

// 发送登录数据包
function handleLogin(s) {
  let msg = 'u:' + chatRoomInfo['rid']
    + '@' + chatRoomInfo['appid']
    + '\nk:1\nt:300\nts:' + chatRoomInfo['ts']
    + '\nsign:' + chatRoomInfo['sign']
    + '\nauthtype:' + chatRoomInfo['authType'];
  sendData(s, msg);
}

// 发送心跳数据包
function handleHeartbeat(s) {
  let data = new Buffer(4);
  data.writeInt16BE(6, 0);
  data.writeInt16BE(0, 2);
  s.write(data);
}
```

下面要解析熊猫tv的协议里边的接收到的数据了

```javascript
let completeMsg = [ ];
s.on('data', (chunk) => {
  completeMsg.push(chunk);
  chunk = Buffer.concat(completeMsg);
  if (chunk.readInt16BE(0) == 6 && chunk.readInt16BE(2) == 6) {
    completeMsg = [ ];
    // 连接弹幕服务器帧头，代表连接成功了
  } else if (chunk.readInt16BE(0) == 6 && chunk.readInt16BE(2) == 6) {                                           
    // 连接弹幕服务器响应
  } else if (chunk.readInt16BE(0) == 6 && chunk.readInt16BE(2) == 3) {
    //接收到消息
    var msg = getMsg(chunk);
    if (msg[0].length < msg[1]) {
        // console.log('parted');
        // 这是一条不完整的数据包，需要等待下一个数据包合并
    } else {
        analyseMsg(msg[0]);
        completeMsg = [ ];
    }
  } else if (chunk.readInt16BE(0) == 6 && chunk.readInt16BE(2) == 1) {
    //心跳保持服务器返回的值
    completeMsg = [ ];
  } else {
    console.log(chunk.toString());
    completeMsg = [ ];
  }
});
s.on('error', (err) => {
  console.error(err);
});
s.on('close', () => {
  console.error('close');
});

function getMsg(chunk) {
  var msgLen = chunk.readInt16BE(4);
  var msg = chunk.slice(6, 6 + msgLen);
  var offset = 6 + msgLen;
  msgLen = chunk.readInt32BE(offset);
  offset += 4;
  let msgInfo = [];
  msgInfo.push(chunk.slice(offset, offset + msgLen));
  msgInfo.push(msgLen);
  return msgInfo;
}

function analyseMsg(totalMsg) {
  while (totalMsg.length > 0) {
    var IGNORE_LEN = 12;
    totalMsg = totalMsg.slice(IGNORE_LEN);
    var msgLen = totalMsg.readInt32BE(0);
    var msg = totalMsg.slice(4, 4 + msgLen);
    onProcessMessage(JSON.parse(msg));
    totalMsg = totalMsg.slice(4 + msgLen);
  }
}

const MSGTYPE = {
  COMMENT: '1', // 评论
  BAMBOO: '206', //竹子
  GIFTS: '306', //礼物(不包括竹子)
  SYS_GIFT_MSG: '311', //系统礼物广播
  POPULAR: '207' //人气值
};

function onProcessMessage(msg) {
  switch (msg.type) {
    case MSGTYPE.COMMENT:
      // 评论消息体样本
      // {
      //     type: '1',
      //     time: 1484393314,
      //     data: {
      //         from: {
      //             __plat: 'ios',
      //             badge: '',
      //             identity: '30',
      //             level: '1',
      //             msgcolor: '',
      //             nickName: 'ArkeAx',
      //             rid: '50062040',
      //             sp_identity: '0',
      //             userName: ''
      //         },
      //         to: {
      //             toroom: '99217'
      //         },
      //         content: '麦爹'
      //     }
      // }
      console.log(`${msg.data.from.nickname}: ${msg.data.content}`);
      break;
    default:
      console.log(msg);
}
```

上面就解析了一个评论消息体的样本，其他的消息体结构自行解析吧~ 有空我把这篇文章的 demo 代码上传上去，不过不能保证上传时间就是了，有空可以follow一下我的github。话说回来下一篇讲讲比较少有的解析虎牙直播的弹幕消息吧，感觉更加有含金量。

