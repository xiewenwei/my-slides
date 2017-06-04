
# 去除 Rails 烦人的 “translation_missing”

## 问题背景

在排查系统问题时，常常在异常监控系统 `errbit` 中发现类似 `translation_missing...` 消息，仔细探查原因发现是由于应用使用的是: `:zh-CN` 的 locale，但是 `:zh-CN` 配置文件中没有提供相应的配置导致了问题。解决问题的方法是在 `zh-CN.yml` 增加相应的配置，虽说这样可以解决问题，不过总觉得有些麻烦，因为大部分情况下当没有 `:zh-CN` 配置的时候，使用 `:en` 配置应该就可以了。为了达到这个效果，仔细研究了 rails 使用的 `i18n` 组件和让人困惑的 `rails-i18n` 组件，有一些以前未意识到的有意思的发现，特地和大家分享一下。

## Rails 项目默认使用 i18n，但默认并没有使用 rails-i18n

`i18n` 是一个简单纯粹的 ruby gem，提供国际化语言处理标准接口和处理方法，它可以用在 rails 应用中，也可以用在其它非 rails 应用中。rails-i18n 是一个特别为 rails 提供 `i18n` 国际化语言配置和处理的 gem，只用在 rails 应用中。比如让人困惑的是 rails 项目默认依赖 `i18n`，但是默认并不依赖 `rails-i18n`，需要使用的话需要自己手动引入，引入的方法是在 Gemfile 中增加 `gem 'rails-i18n'` 配置申明。声明 gem 之后，通常还需要在 `config/application.rb` 中增加如下配置：

```ruby
    config.i18n.available_locales = [:en, :"zh-CN"]
    config.i18n.default_locale = :"zh-CN"
```

## 如何实现 :zh-CN locale 未配置时，使用 :en locale 的配置

要实现这个效果得使用 `i18n` 的异常处理机制，`i18n` 在没有找到配置信息时会抛出异常，可以通过截获异常进行处理。
解决方法如下，在 rails 项目的 `config/initializers` 目录下增加文件 `i18n_missing_translations.rb`，文件中代码如下所示：

```ruby
  i18n_simple_backend = I18n::Backend::Simple.new
  old_handler = I18n.exception_handler
  I18n.exception_handler = lambda do |exception, locale, key, options|
    case exception
      when I18n::MissingTranslation
        i18n_simple_backend.translate(:en, key, options || {})
      else
        old_handler.call(exception, locale, key, options)
    end
  end
```

## 总结：
  + 需要手动引入 `rails-i18n`，同时配置 `:zh-CN` 和 `:en` 作为 available_locales
  + 提供 `i18n` 异常处理代码实现未找到 `:zh-CN` 配置时，还能使用 `:en` locale
