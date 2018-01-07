
## 支持子命令的 `ruby` 命令行程序

在 `ruby` 中很容易写命令行程序，它的标准库 `OptionParser` 对命令行选项提供很好支持，使用起来十分简单，通过它提供的例子就可以快速掌握它的使用方法。

下面是一个简单的例子：

```ruby
require 'optparse'

options = {}
OptionParser.new do |opts|
  opts.banner = "Usage: example.rb [options]"

  opts.on("-v", "--[no-]verbose", "Run verbosely") do |v|
    options[:verbose] = v
  end
end.parse!

p options
p ARGV
```

详细使用说明见 [OptionParser 文档](https://docs.ruby-lang.org/en/2.1.0/OptionParser.html)

如果要实现子命令，`OptionParser` 也能够简便支持，这是不能直接用 `parse!` 方法，而是应该用 `order!` 方法。下面提供我在 `iceman` 这个项目中支持子命令的例子(为减少篇幅，代码有部分删减)。

```ruby
require 'optparse'
require 'yaml'
require 'iceman/action'

def parse_command
  options = {
    config: 'config/hosts',
    hosts: 'icecream',
    env: 'development'
  }

  subtext = <<HELP
  Commands:
    start   :  启动一个 wechat 实例
    stop    :  关闭最后启动的 wecaht 实例
    work    :  启动监听处理命令
  使用 'iceman COMMAND --help' 查看每个命令更详细信息.
HELP

  global = OptionParser.new do |opts|
    opts.banner = "Usage: iceman [options] command [options]"

    opts.separator "  options:"
    opts.on("-v", "--[no-]verbose", "Run verbosely") do |value|
      options[:verbose] = value
    end

    opts.on('-c', '--config CONFIG_FILE', 'Config file (default: config/hosts)') do |value|
      options[:config] = value
    end

    opts.on('-h', '--hosts HOSTS', 'Iceland hosts (default icecream)') do |value|
      options[:hosts] = value
    end
    opts.separator ""
    opts.separator subtext
  end

  subcommands = {
    'start' => OptionParser.new do |opts|
        opts.banner = "Usage: start"
    end,
    'stop' => OptionParser.new do |opts|
        opts.banner = "Usage: stop"
    end,
    'work' => OptionParser.new do |opts|
      opts.banner = "Usage: work [options]"
      opts.on('-e', '--env environment', 'Environment (default development)') do |value|
        options[:env] = value
      end
      opts.on("-d", "--[no-]daemonize", 'Run daemonized in the background (default: false)') do |value|
        options[:daemonize] = value
      end
    end
  }

  global.order!
  command = ARGV.shift
  abort("没有 command") unless command

  if (subcommand = subcommands[command])
    subcommand.order!
    puts "Command: #{command}"
    puts "options: #{options}"
  else
    abort("错误的 command #{command}")
  end
  [command, options]
end

def run!
  work_dir = File.expand_path("../../", __FILE__)
  Dir.chdir work_dir

  command, options = parse_command

  # start worker or execute action
  if command == 'work'
    require 'iceman/worker'
    worker = Iceman::Worker.new options
    worker.run
  else
    action = Iceman::Action.new command, options
    action.execute
  end
end

# main entry
run!

```
从上面的例子中可以看出，`order!` 首先解析子命令前的命令行选项，这是通过 ARGV.shift 可以获得子命令，再通过后续的 `order!` 得到子命令的选项。

