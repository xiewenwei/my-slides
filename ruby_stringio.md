
# Ruby 中通过 StringIO 处理文件读写

这周一直忙金问号项目，就简单分享金问号项目中遇到的一个小问题的解决方法。

金问号需要通过 http 请求微信服务接口，http 请求需要带 cookie。金问号项目使用的 http client gem 是 `http`，这个 gem 使用 `http-cookie` gem 实现 cookie 相关功能，`http-cookie` 提供方法保存和加载 cookie。

`http-cookie` 提供的保存和加载接口如下所示：

* HTTP::CookieJar#save(writable, *options)
* HTTP::CookieJar#load(readable, *options)

通常的调用方法如下：

```ruby
  # save
  cookie_jar.save file_path
  # load
  cookie_jar.load file_path
```

但是金问号的情况需要把 cookie 和其它会话相关数据保存在一起，而且是要存储在 redis 中而不是在单一文件里，这就需要获取 cookie_jar 序列化结果的字节流。

一开始考虑用 Marshal 序列化 cookie_jar 对象，经过测试没有成功，因为 cookie_jar 中带有 IO 类型对象，无法使用 Marshal 序列化。

仔细阅读 `http-cookie` 代码后发现，可以给 save 和 load 方法传递 IO 对象，通过这个 IO 对象返回或传入数据，具体用到 Ruby 的 StringIO 对象。

实际代码如下所示：

```ruby
  # 保持 cookies 代码，cookies 是 HTTP::CookieJar 实例
  cookies_io = StringIO.new
  cookies.save cookies_io
  cookies_data = cookies_io.string
```

```ruby
  # 加载 cookies 代码
  # get cookies_data
  cookies_io = StringIO.new cookies_data
  cookies = HTTP::CookieJar.new
  cookies.load cookies_io
```

通过使用 StringIO 对象，实现了类文件的数据传输。
