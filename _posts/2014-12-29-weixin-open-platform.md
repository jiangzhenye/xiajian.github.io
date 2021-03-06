---
layout: post
title: "微信公众平台API"
description: "微信API，开发学习笔记"
category: api
---

## 前言

我发现，我学生时代的“作业”拖到最后一刻毛病又犯了。我把修改微信菜单的任务，硬生生的拖了一个星期，眼看，拖不下去了，这不，就来做了。话说，上周，我都干了些啥？！

新的时代，有新的机遇。我自已想了想，什么都要自己搞自己弄，不见得好，何不利用现成的，API也好，框架也好。全栈式的含义是，是对事物有全景式的认识，而不是盲人摸象、鼠目寸光。或许，微信是一个不错的机遇，所以，我在多看上买了本微信开发的书。人，毕竟不能逆着时代潮流走，何不凭借这阵浪潮，会击长空一番。我为我自己的这一番见识和文采，洋洋自得。

## 接口介绍

公众号，接口权限，API文档。

### 接入指南

第1步: 设置三个参数 - URL : 接受微信服务器数据的地址、Token(授权token，生成签名，安全性)、EncodingAESKey(加密密钥，明文模式，兼容模式，安全模式)。

> Token是用户设定的字符串，是微信后台与公众帐号server之间的密钥。

第2步 验证URL有效性， GET请求，四个参数(signature - 加密签名、随机数，随机签名组合, timestamp, nonce - 随机数, echostr - 随机字符串 )，文档中的php代码转换为Ruby代码示例如下: 
    
    ```ruby
    def validate_request
      signature = params[:signature] # timestamp，signature以及Token之间SHA1加密生成的字符串。
      timestamp = params[:timestamp] # 时间戳
      nonce     = params[:nonce]     # 随机数
      echostr   = params[:echostr]
      encrypt_str = Digest::SHA1.hexdigest([timestamp, nonce, Settings.wx_token].sort.join)   
      render text: "Forbidden", status: 403  unless encrypt_str == signature
    end
    ```

第3步，URL验证成功即称为开发者。公众号分类型: 服务号，认证号(获得众多接口权限)。消息(OpenID)和菜单事件，推送响应URL。仅支持80接口。

实例：主要都是服务号的例子

开发者规范 ： 接口规范，调用频率，模板消息，用户数据等

## 基础支持

`access_token`是公共号的全局唯一码，所有接口调用都需要。有效期为2小时，定期刷新，多逻辑业务，考虑中转服务器包装access_token。

中转服务器提供的3种功能(类似ORM对数据库的连接的封装): 

1. 获取最新可用的access_token
2. 过期自动更新
3. 提供主动刷新给业务点

这里，要注意对`access_token`的处理要集中，此外，token的长度至少512字节。公共平台使用AppID和AppSecret调用接口，获取
`access_token`，接口调用使用HTTPS协议。

URL接口: `https://api.weixin.qq.com/cgi-bin/token?grant_type=client_credential&appid=APPID&secret=APPSECRET`

参数说明 

   参数      | 是否必须 |	说明
------------ | -------- | ---------------------------------------------------
`grant_type` | 是 	    | 获取`access_token`填写`client_credential`授权类型
appid 	     | 是 	    | 第三方用户唯一凭证
secret 	     | 是 	    | 第三方用户唯一凭证密钥，即appsecret

> 我猜`grant_type`是加密模式，但又不是特别确定。文档中，也没明说。

返回的JSON数据如下: 

```
# 成功
{"access_token":"ACCESS_TOKEN","expires_in":7200}
# 失败
{"errcode":40013,"errmsg":"invalid appid"}
```

> 感：API什么的，果然都是传递的json数据。但自己看到，公司发送给微信数据是xml格式。

API调用的频率具有一定的限制，每日计数，看来，其服务器上存在一个专门计数和审计的功能模块。

辅助资料： [返回码说明](http://mp.weixin.qq.com/wiki/17/fa4e1434e57290788bde25603fa2fcbd.html), [接口频率说明](http://mp.weixin.qq.com/wiki/0/2e2239fa5f49388d5b5136ecc8e0e440.html), [上传下载多媒体文件](http://mp.weixin.qq.com/wiki/10/78b15308b053286e2a66b33f0f0f5fb6.html)

> 问题: 为何文档的html命名是随机生成的，揭示了其显示的使用了何种技术吗？

## 消息体签名及加密

加密解密功能: 兼容模式和安全模式, URL新增量参数(加密类型和消息体签名), 对称加密算法AES，密钥EncodingAESKey。微信提供[代码示例](http://mp.weixin.qq.com/wiki/static/assets/a5a22f38cb60228cb32ab61d9e4c414b.zip)，可直接引用到项目中。

> 注: 有机会，可以仿写个Ruby版本的，参考其官方文档和代码示例。

## 接受消息

消息的接受和处理，需要进行消息的验证。可以接受的消息，有文本、图片、语音、视频、地理位置、链接等消息。还可以接受事件推送，比如，关注/取消，
扫码，地理位置，以及自定义菜单事件。

> 注: 接受是微信接受的消息，推送是微信推送给客户的事件。POST请求，使用XML进行通信，这里需要使用ruby中解析XML类库。

## 发送消息

## 用户管理

## 多客服功能

## 自定义菜单

工作指定的任务，即修改菜单相关。自定义菜单为 3 × 5，支持8种类型的按钮事件: 

1. click：点击推事件

用户点击click类型按钮后，微信服务器会通过消息接口推送消息类型为event	的结构给开发者（参考消息接口指南），并且带上按钮中开发者填写的key值，开发者可以通过自定义的key值与用户进行交互；

2. view：跳转URL

用户点击view类型按钮后，微信客户端将会打开开发者在按钮中填写的网页URL，可与网页授权获取用户基本信息接口结合，获得用户基本信息。

3. scancode_push：扫码推事件

用户点击按钮后，微信客户端将调起扫一扫工具，完成扫码操作后显示扫描结果（如果是URL，将进入URL），且会将扫码的结果传给开发者，开发者可以下发消息。

4. scancode_waitmsg：扫码推事件且弹出“消息接收中”提示框

用户点击按钮后，微信客户端将调起扫一扫工具，完成扫码操作后，将扫码的结果传给开发者，同时收起扫一扫工具，然后弹出“消息接收中”提示框，随后可能会收到开发者下发的消息。

5. pic_sysphoto：弹出系统拍照发图

用户点击按钮后，微信客户端将调起系统相机，完成拍照操作后，会将拍摄的相片发送给开发者，并推送事件给开发者，同时收起系统相机，随后可能会收到开发者下发的消息。

6. pic_photo_or_album：弹出拍照或者相册发图

用户点击按钮后，微信客户端将弹出选择器供用户选择“拍照”或者“从手机相册选择”。用户选择后即走其他两种流程。

7. pic_weixin：弹出微信相册发图器

用户点击按钮后，微信客户端将调起微信相册，完成选择操作后，将选择的相片发送给开发者的服务器，并推送事件给开发者，同时收起相册，随后可能会收到开发者下发的消息。

8. location_select：弹出地理位置选择器

用户点击按钮后，微信客户端将调起地理位置选择工具，完成选择操作后，将选择的地理位置发送给开发者的服务器，同时收起位置选择工具，随后可能会收到开发者下发的消息。


## 智能接口

即所谓的语义理解功能和相关技术。

## weixin JS接口

> 页面，js，微信难不成内置了个浏览器，可以调用js接口什么的, 微信果真集成个自己开的浏览器。


## 后记

每学一个新的东西，就是重新构建一套知识体系的过程。如果，构建的完善的话，就形成了一本书。

## 参考文献

1. [微信开发者文档](http://mp.weixin.qq.com/wiki/17/2d4265491f12608cd170a95559800f2d.html)
2. [微信公众平台开发教程](http://www.cnblogs.com/yank/category/539657.html)
