# Sidekiq Queue 使用

## 问题概要

最近一个项目开发中遇到一个 Sidekiq 的问题：希望能够指定 Sidekiq 任务处理的机器。
通常情况我们会把 Sidekiq 部署在多台机器，任务在那台机器执行是随机的，但是这个项目由于一些特殊的原因需要这些任务能够固定在一台机器上执行。

## 设计思路

仔细研究 Sidekiq 之后发现可以利用 Sidekiq 的 Queue 选项解决该问题。利用 Sidekiq 的 queue 属性，每一个启动的 Sidekiq 进程除了处理 default queue，还处理一个自定义的 queue。另外每一个需要指定机器运行的任务在分派时设定它的 Queue。

## 涉及技术要点

* 启动 Sidekiq 时指定 queue 参数启动
 `bundle exec sidekiq -q sq0,1 -q default,1`

* 分派任务是指定其 Queue 属性
 `DemoWorker.set(queue: :sq0).perform_aync(params)`

* 获取 Sidekiq 中所有 queue 列表

 ```ruby
   queues = Sidekiq::Queue.all
   names = queues.map(&:name).select {|i| i =~ /^sq/}
   # ["sq0", "sq1"]
 ```

 通过这个方法很好的解决了指定机器处理任务的问题。
