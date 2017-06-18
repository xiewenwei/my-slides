
# Rails 5.1 版本带来测试的新特性

最近翻看 Ruby on Rails 5.1 Release Notes，发现这个版本带来的特性真不少，其中关于测试有两个：一是增加了 `System tests`, 二是增加了 `Rails test` 命令运行器。

## `System test`

以前写系统验收测试的话，可以使用 `Capybara` gem。Rails 5.1 直接把 `Capybara` 整合进来，让系统验收测试更加简单方便。

生成 system test 可以通过一个新增的 Rails generator: system_test。示例如下：

```
bin/rails g system_test projects_test
```

这条命令将在 `test/system` 目录下生成文件 `projects_test.rb`，默认内容如下：

```ruby
require "application_system_test_case"

class ProjectsTest < ApplicationSystemTestCase
  test "visiting the index" do
    visit projects_url

    assert_selector "h1", text: "Project"
  end
end
```

需要注意的是 Rails 下运行 system_test 使用命令是 `rails test:system`。默认的 `rake test` 是不会运行 system_test 的。

## Rails test 命令运行器

Rails 5.1 还带来一个新的 test 命令运行器，通过它可以比 `rake test` 更好的控制测试运行的行为。

我觉得十分有用的点有两个：一是可以指定运行某一个测试文件（或者某一个目录的所有文件）的测试，例如：

```ruby
  # 运行单个文件的测试或单个目录测试
  bin/rails test test/services test/integration/login_test.rb
```

另一个十分有用的特性是可以在测试运行完毕，才统一打出所有失败或错误的测试，只需要增加 `-d, --defer-output` 命令选项，再也不用逐个翻页查找出问题的测试了。

通过 `rails test --help` 可以详细了解该命令的基本使用。

## 小结

Rails 5.1 带来的 `System tests` 和 `Rails test 命令` 让写测试更简单高效了，非常棒的改进，值得尽快升级使用。
