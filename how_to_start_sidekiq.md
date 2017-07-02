
# 如何指定不同机器的 Sidekiq 启动参数

## 问题概要

上周的 `Sidekiq Queue 使用` 谈到需要在启动 Sidekiq 时指定 queue 参数启动
如： `bundle exec sidekiq -q sq0,1 -q default,1`
我们使用 `capistrano` 和 `capistrano-sidekiq` 部署应用，研究了一番，发现没办法通过已有配置直接指定不同机器的 Sidekiq 启动参数。

## 解决方法

仔细研究 `capistrano-sidekiq` 启动实例的方法，发现 sidekiq 是根据 role 启动的，详细代码如下（文件 lib/tasks/sidekiq.rake）：
```ruby
  desc 'Start sidekiq'
  task :start do
    on roles fetch(:sidekiq_role) do |role|
      switch_user(role) do
        for_each_process do |pid_file, idx|
          start_sidekiq(pid_file, idx) unless pid_process_exists?(pid_file)
        end
      end
    end
  end
```
```ruby
  def start_sidekiq(pid_file, idx = 0)
    args = []
    args.push "--index #{idx}"
    args.push "--pidfile #{pid_file}"

    # 忽略部分代码 ...

    Array(fetch(:sidekiq_queue)).each do |queue|
      args.push "--queue #{queue}"
    end

    execute :sidekiq, args.compact.join(' ')
  end
```

如果要干预不同机器的启动参数，可以为每台机器设置不同的 role，然后设置不同 role 的启动 参数选项，在启动时根据 role 获取参数就可以了。

具体修改如下：

```ruby
  desc 'Start sidekiq'
  task :start do
    on roles fetch(:sidekiq_role) do |role|
      switch_user(role) do
        for_each_process do |pid_file, idx|
          start_sidekiq(pid_file, idx, role) unless pid_process_exists?(pid_file)
        end
      end
    end
  end
```
```ruby
  def start_sidekiq(pid_file, idx = 0, role)
    args = []
    args.push "--index #{idx}"
    args.push "--pidfile #{pid_file}"

    # 忽略部分代码 ...

    sidekiq_queue = fetch(:"#{role}_queue") || fetch(:sidekiq_queue)

    Array(sidekiq_queue).each do |queue|
      args.push "--queue #{queue}"
    end

    execute :sidekiq, args.compact.join(' ')
  end
```

主要的修改是为方法 start_sidekiq 增加 role 参数，启动时传入 role 参数，在 start_sidekiq 根据 role 获取 sidekiq_queue 参数。

这样问题得到解决。详细代码见 https://github.com/xiewenwei/capistrano-sidekiq/blob/feature/sidekiq-queue-for-role/lib/capistrano/tasks/sidekiq.rake


