---
layout: post
title:  '爬知乎用户信息'
categories: Python scrapy
author: CHH
---

* content
{:toc}




## 常见错误

- 500，internal server error，连请求 robots.txt 也是这个状态码，通常加 User-Agent 即可。可通过 chrome://version/ 查看。
- 401，unauthorized，请求头加 authorization。

## URL 分析
从知乎大 V [vczh](https://www.zhihu.com/people/excited-vczh/followers) 开始，爬取的是用户信息。
递归查询方法是：分别通过关注他的人、他关注的人获取新用户。
所以 URL 有三条：用户本身信息、关注他的人的URL、他关注的人的URL。

### 用户本身信息
把鼠标停留在某个用户头像上，发现 XHR 请求路径是：
 ```
https://www.zhihu.com/api/v4/members/fizzy?
include=allow_message%2Cis_followed%2Cis_following%2Cis_org%2Cis_blocking%2C
employments%2Canswer_count%2Cfollower_count%2Carticles_count%2Cgender%2C
badge%5B%3F(type%3Dbest_answerer)%5D.topics
 ```
经多个用户对比发现只有用户 ID 不同（上面 URL 对应是 fizzy），其余都一样。
再观察发现，用户 ID 就是用户信息中的 url_token 字段。
根据用户信息来定义 items。

![用户字段](https://upload-images.jianshu.io/upload_images/5690299-7610e8471f3b65fe.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)



### 关注他的人
同样是 XHR 请求。
```
https://www.zhihu.com/api/v4/members/excited-vczh/followers?
include=data%5B*%5D.answer_count%2Carticles_count%2Cgender%2Cfollower_count%2C
is_followed%2Cis_following%2Cbadge%5B%3F(type%3Dbest_answerer)%5D.topics
&offset=20&limit=20
```
经多个用户对比发现只有用户 ID 不同（上面 URL 对应是 excited-vczh），其余都一样。

## 他关注的人
```
https://www.zhihu.com/api/v4/members/excited-vczh/followees?
include=data%5B*%5D.answer_count%2Carticles_count%2Cgender%2Cfollower_count%2C
is_followed%2Cis_following%2Cbadge%5B%3F(type%3Dbest_answerer)%5D.topics
&offset=0&limit=20
```
一个是 followers、另一个是 followees。其他完全一样。

## 回调方法分析
三条 URL 对应回调方法三条。
分析用户的 URL，回调方法分析出用户的信息字段。
还有，为了进行迭代，还应回调用户的关注的人，以及关注用户的人的方法。

关注他的人，回调分析用户的方法。还有为了迭代，进行翻页，回调本方法。

他关注的人，同关注他的人。