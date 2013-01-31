---
layout: post
title: "使用Faraday上传图片到新浪微博"
date: 2013-01-31 20:48
comments: true
categories: 
---

>[Faraday](https://github.com/lostisland/faraday) 是一个轻量灵活的 HTTP client.使用 Faraday 可以轻松的帮助我们上传含图片内容的信息到新浪微博。

新浪微博的 `statuses/upload` 接口要求请求必须是 `multipart/form-data` 编码格式，并且 `pic` 参数为 `binary` 类型。[Faraday](https://github.com/lostisland/faraday) 几乎为我们实现了所有的要求，我们不需要自己来拼接这种请求报头和报文，下面是我的一个实例：

```
conn = Faraday.new(:url => 'https://upload.api.weibo.com') do |faraday|
  faraday.request :multipart
  faraday.adapter :net_http
end

conn.post "/2/statuses/upload.json", {
  :access_token => topic.user.weibo_token,
  :status => URI.encode(status),
  :pic => Faraday::UploadIO.new(pic_path, 'image/jpeg')
}

```