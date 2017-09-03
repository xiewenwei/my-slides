
# 在查询表单中使用 `simple_form`

## 问题描述

在 Rails 中，`simple_form` 是表单生成的利器，它的功能强大灵活，代码简洁清晰，生成的表单美观大方，在薄荷的项目中应用广泛。不过它有一个限制是必须基于 model object 生成表单，如果没有 model object 就无法使用了。在后台管理中，有一类常见的表单是查询表单，如订单、用户查询等等，以前在实现这类功能的使用通常就直接使用 form_tag，基于 params 构造查询，无论页面层代码还是控制、服务层代码都很繁琐。

最近开发一个程序又遇到同样的问题，程序的功能是比较普遍的，就是基于分类、状态、下单时间和付款时间查询筛选订单，然后显示订单列表，以及导出订单。功能涉及到一个查询表单，如果基于 form_tag 来实现并不复杂，只是对比一直使用的 `simple_form` 显得十分繁琐，于是仔细研究了 `simple_form` 中的解决方法，发现通过自定一个 search form object，可以比较优雅的解决这个问题。

## 定义 search form object

首先要定义一个 search form object，这个 object 必须具备 ActiveModel 一些特性，这样才能在 `simple_form` 中使用，对于订单查询的表单，我建立了一个 `OrderQueryForm`，代码如下：

```ruby
class OrderQueryForm
  include ActiveModel::Validations
  include ActiveModel::Conversion
  extend ActiveModel::Naming

  attr_accessor :state, :sale_id, :created_on, :paid_on

  def initialize(attributes = {})
    attributes.each do |name, value|
      send("#{name}=", value)
    end
  end

  def persisted?
    false
  end
end
```

## 使用 `simple_form` 构建表单

定义 search form object 后，接着就可以通过 `OrderQueryForm` 的实例构建 simple_form 了，示例代码如下：

```ruby
  <%= simple_form_for :query, url: admin_orders_url, method: :get, html: {class: 'form-inline'} do |f| %>
    <%= f.input :state, label: '订单状态：', as: :select,
      collection: Order.state.options, include_blank: true %>&nbsp;
    <%= f.input :sale_id, label: '接龙：', as: :select,
      collection: Sale.select("title,id").order("updated_at desc"),
      label_method: :title, value_method: :id, include_blank: true %>&nbsp;
    <%= f.input :paid_on, label: '付款日期：', placeholder: '如 2017-08-20' %>&nbsp;
    <%= f.input :created_on, label: '下单日期：', placeholder: '如 2017-08-30' %>
    <%= f.button :submit, "查询" %>
  <% end %>
```

## 使用 query model object

配合前端页面，controller 中使用 `OrderQueryForm` 实例的方法如下：

```ruby
  def query_params
    if params[:query]
      params.require(:query).permit(:state, :sale_id, :created_on, :paid_on)
    else
      {}
    end
  end
```

使用 `@query` 查询过滤 orders 代码如下：
```ruby
  @query = OrderQueryForm.new query_params
  orders = Order.includes({order_items: :product})
  @query.filter(orders)
```

其中 `OrderQueryForm` 的 filter 方法定义如下

```ruby
class OrderQueryForm
  # 省略其它代码...
  def filter(orders)
    if state.present?
      orders = orders.with_state(state)
    end

    if sale_id.present?
      orders = orders.where(sale_id: sale_id)
    end

    if created_on.present?
      date = Date.parse(created_on)
      orders = orders.where("created_at >= ? and created_at < ?", date, date + 1.days)
    end

    if paid_on.present?
      date = Date.parse(paid_on)
      orders = orders.where("paid_at >= ? and paid_at < ?", date, date + 1.days)
    end
    orders
  end
end
```

## 总结

至此，通过建立一个 search form object，可以在普通查询中使用 simple_form 构建表单，同时在控制和服务层能比较优雅构建数据查询方法。
