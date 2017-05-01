
# 使 ruby 的 http 连接 keepalive

    vincent on 2017.5.1

---

## 问题背景

 * SA 系统事件发送接口调用非常频繁
   - 每天有近 300 万次 http 请求
   - 峰值估计有 20k rpm
 * 不定期出现连接超时错误
   - 平均每天报 1000 个 http 连接超时错误

---

## 使连接 keepalive 的好处

 * http 1.0 每次请求都通过 TCP 三次握手建立连接，请求完成后销毁连接
 * http 1.1 实现 keepalive 特性，使 http connection 得已复用
 * 避免每次请求都建立和销毁连接，一次建立的 connection 可以完成上百个请求
 * http keepalive 需要客户端和服务端同时支持

---

## 如何实现

 * ruby 的标准库 net/http 没有提供 keepalive 特性
 * 使用第三方 gem: net-http-persistent
 * 使用效果：大幅简单 http 连接超时错误

---

## 使用介绍

使用方法很简单

```ruby
  require 'net/http/persistent'

  uri = URI 'http://example.com/awesome/web/service'

  http = Net::HTTP::Persistent.new 'my_app_name'

  # perform a GET
  response = http.request uri

  # create a POST
  post_uri = uri + 'create'
  post = Net::HTTP::Post.new post_uri.path
  post.set_form_data 'some' => 'cool data'

  # perform the POST, the URI is always required
  response = http.request post_uri, post
```


---

## 实现原理

 * 使用 connection-pool 建立连接池
 * 每次请求从连接池取出 connection
 * 请求结束后把 connection 放入连接池，不关闭连接
 * 每个 connection 可以设置最多接收请求上限

---

## 其它方案

ruby `http` gems
```ruby
  begin
    # create HTTP client with persistent connection to api.icndb.com:
    http = HTTP.persistent "http://api.icndb.com"

    # issue multiple requests using same connection:
    jokes = 100.times.map { http.get("/jokes/random").to_s }
  ensure
    # close underlying connection when you don't need it anymore
    http.close if http
  end
```

---

## 总结

 * http 连接 keepalive 大幅提升连接使用效率，降低连接超时错误发生率
 * 使用 net-http-persistent gem 为 ruby net/http 标准库提供 keepavlie 能力
 * http keepalive 使用请求特别频繁的情景，如果请求不频繁，没有使用必要

---

## 参考资料

 * [net-http-persistent gem](https://github.com/drbrain/net-http-persistent)
 * [sa-sdk-ruby](https://github.com/xiewenwei/sa-sdk-ruby)
 * [Persistent Connections (keep alive) of http gem](https://github.com/httprb/http/wiki/Persistent-Connections-%28keep-alive%29)
