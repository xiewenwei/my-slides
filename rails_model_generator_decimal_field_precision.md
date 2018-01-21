# 指定生成 rails model 时数值字段的精度

Rails 项目中常常使用 `Model Generator` 生成 `Model` 相关代码，一般需要指定数值类型精度和小数位，如 `decimal(10,2)`。Rails 可以传递给 Generator 参数指定数值字段的精度和小数位，例如我们需要生成带 price 字段的 Product 模型，命令行如下：

```shell
rails g model Product "price:decimal{10,2}"
```

生成的代码如下：

```ruby
class CreateProducts < ActiveRecord::Migration[5.1]
  def change
    create_table :products do |t|
      t.decimal :price, precision: 10, scale: 2

      t.timestamps
    end
  end
end
```

特别注意：必须对字段名和类型定义加上引号，如果不加引号，生成的代码如下：

```ruby
class CreateProducts < ActiveRecord::Migration[5.1]
  def change
    create_table : products do |t|
      t.decimal10 :price
      t.decimal2 :price

      t.timestamps
    end
  end
end
```

这显然不是我们期望得到的。另外还有一种格式可以避免使用引号，如下所示：

```shell
rails g model Product price:decimal{10-2}
```

不使用引号，但是大括号里的逗号变成了减号。

