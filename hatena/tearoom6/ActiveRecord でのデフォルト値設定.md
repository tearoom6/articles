Rails ActiveRecord model で各フィールドのデフォルト値を設定するには、主に以下の2つの方法があります。

#### 方法1: Database の column の default 値を使う

Database の table schema で column の `default` 値を指定します。

```ruby
class CreateColors < ActiveRecord::Migration[5.0]
  def change
    create_table :colors do |t|
      t.string  :name,                                         default: 'black'
      t.integer :red,   unsigned: true, limit: 1, null: false, default: 0
      t.integer :green, unsigned: true, limit: 1, null: false, default: 0
      t.integer :blue,  unsigned: true, limit: 1, null: false, default: 0
      t.boolean :transparent,                     null: false, default: false
      t.string  :alias
    end
  end
end
```
```ruby
class Color < ApplicationRecord
end
```

すると、model の instance 初期化時に、その値が属性のデフォルト値としてセットされます。 (Rails が気を利かせて、スキーマから取得した値を自動で入れてくれています)

```ruby
black = Color.new
#<Color:0x0000562637d38ed0 id: nil, name: "black", red: 0, green: 0, blue: 0, transparent: false, alias: nil>
```

明示的に別の値と初期値として渡した場合は、デフォルト値は使われません。

```ruby
transparent_white = Color.new(
  name: :white,
  red: 256,
  green: 256,
  blue: 256,
  transparent: true,
)
#<Color:0x000056263769b780 id: nil, name: "white", red: 256, green: 256, blue: 256, transparent: true, alias: nil>
```

#### 方法2: model の hook method を使う

もう一つの方法は、 model の hook method (特に `after_initialize`) を使う方法です。

```ruby
class CreateColors < ActiveRecord::Migration[5.0]
  def change
    create_table :colors do |t|
      t.string  :name
      t.integer :red,   unsigned: true, limit: 1, null: false
      t.integer :green, unsigned: true, limit: 1, null: false
      t.integer :blue,  unsigned: true, limit: 1, null: false
      t.boolean :transparent,                     null: false
      t.string  :alias
    end
  end
end
```
```ruby
class Color < ApplicationRecord
  after_initialize :set_default_values
  # Use this if you wanna set default values only when creating a new record.
  #after_initialize :set_default_values, if: :new_record?

  private

  def set_default_values
    self.name        ||= 'black'
    self.red         ||= 0
    self.green       ||= 0
    self.blue        ||= 0
    self.transparent ||= false if self.transparent.nil? # Be careful in case of boolean.
  end
end
```

この方法も先程と同様に機能します。

```ruby
black = Color.new
#<Color:0x0000557767b46070 id: nil, name: "black", red: 0, green: 0, blue: 0, transparent: false, alias: nil>

transparent_white = Color.new(
  name: :white,
  red: 256,
  green: 256,
  blue: 256,
  transparent: true,
)
#<Color:0x000055776333fe20 id: nil, name: "white", red: 256, green: 256, blue: 256, transparent: true, alias: nil>
```

#### まとめ

このように、いずれの方法でも、似たような挙動を実装することが可能です。
ただし、両方を併用することは意味がないばかりか、デフォルト値を変更したつもりができていないみたいなバグを生みかねないので、やめたほうがいいです。

どちらの方法も pros/cons があるので、状況に応じて適切な方法を使うのがいいでしょう。
例えば、双方、以下のようなメリットが考えられます。

- 方法1: SQL で直接 INSERT するような場合にも同じデフォルト値を使える
- 方法2: 他のカラムの値に応じて、デフォルト値を可変にしたいような場合に対応できる

> References

- [ruby on rails - How can I set default values in ActiveRecord? - Stack Overflow](https://stackoverflow.com/questions/328525/)
- [ActiveRecord のデフォルト値 - Qiita](https://qiita.com/akishin/items/71f3225185f45b3ade23)
- [Rails ActiveRecord with Database Column Defaults | End Point](https://www.endpoint.com/blog/2014/02/07/rails-activerecord-with-database-column)
