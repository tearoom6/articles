Rails ActiveRecord model で各フィールドのデフォルト値を設定するには、主に以下の3つの方法があります。

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
  red: 255,
  green: 255,
  blue: 255,
  transparent: true,
)
#<Color:0x000056263769b780 id: nil, name: "white", red: 255, green: 255, blue: 255, transparent: true, alias: nil>
```

#### 方法2: model の hook method を使う

2つ目の方法は、 model の hook method (特に `after_initialize`) を使う方法です。

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
  red: 255,
  green: 255,
  blue: 255,
  transparent: true,
)
#<Color:0x000055776333fe20 id: nil, name: "white", red: 255, green: 255, blue: 255, transparent: true, alias: nil>
```

#### 方法3: Active::Record attributes API を使う (Rails 5 -)

コメントがあったので、こちらも追記しておきます。 Rails 5 で導入された Active::Record attributes API を用いる方法です。

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
  attribute :name,        :string,  default: 'black'
  attribute :red,         :integer, default: 0
  attribute :green,       :integer, default: 0
  attribute :blue,        :integer, default: 0
  attribute :transparent, :boolean, default: false
end
```

もちろん、この方法も同様に機能します。

```ruby
black = Color.new
#<Color:0x000055c77bdb4fc8 id: nil, name: "black", red: 0, green: 0, blue: 0, transparent: false, alias: nil>

transparent_white = Color.new(
  name: :white,
  red: 255,
  green: 255,
  blue: 255,
  transparent: true,
)
#<Color:0x000055c77b1dfb30 id: nil, name: "white", red: 255, green: 255, blue: 255, transparent: true, alias: nil>
```

#### まとめ

このように、いずれの方法でも、似たような挙動を実装することが可能です。
複数を併用する場合、組み合わせによっては、意味がないばかりか、デフォルト値を変更したつもりができていないみたいなバグを生みかねないので、やめたほうがいいです。

どの方法も pros/cons があるので、状況に応じて適切な方法を使うのがいいでしょう。
例えば、それぞれの方法には、以下のようなメリットが考えられます。

- 方法1: SQL で直接 INSERT するような場合にも同じデフォルト値を使える
- 方法2: 他のカラムの値に応じて、デフォルト値を可変にしたいような場合に対応できる
- 方法3: 方法2と同様に、動的にデフォルト値をセットできる他、 DB 上のカラムタイプと Rails モデルでの型が一致する必要がなくなる

コメントを受けて追記した最後の attributes API というのは、僕もこの記事を最初に書いた後に知ったんですが、方法1のやり方で Rails が内部的にやってくれている DB スキーマ <=> Rails モデルの相互変換を API として外部提供して、カスタマイズできるようにしてくれたものです。

方法2 に比べて優位性があるのは間違いないですが、ただ、方法1を必ずしも凌駕するかといえば、私自身はどうだろうという気がします。本来の目的は、ドキュメントにもあるように、標準の "DB スキーマ <=> Rails モデル" 変換を、必要に応じて上書きする場合に使用するものであると思われますし、デフォルト値のセットのためだけに使うのは若干 too heavy なように感じます。また、スキーマ定義との重複が生じるので、使い方によっては DRY 原則に反することになります。 (これはむしろ Django 的な、モデルに書く形のスキーマ定義ができない Rails 側の問題と言えますが。モデルで定義した属性を必要に応じて、 DB 向けに上書きできたほうがいい。) とはいえ、DB 依存を極力避けて、 Rails 側に実装を寄せるという点ではベストと言えます。

いずれにしても、一貫性のある方法で実装するのが大事なのは間違いないので、プロジェクトごとに指針をきちんと決めるといいでしょう。

> References

- [ruby on rails - How can I set default values in ActiveRecord? - Stack Overflow](https://stackoverflow.com/questions/328525/)
- [ActiveRecord のデフォルト値 - Qiita](https://qiita.com/akishin/items/71f3225185f45b3ade23)
- [Rails ActiveRecord with Database Column Defaults | End Point](https://www.endpoint.com/blog/2014/02/07/rails-activerecord-with-database-column)
- [ActiveRecord::Attributes::ClassMethods](https://api.rubyonrails.org/classes/ActiveRecord/Attributes/ClassMethods.html#method-i-attribute)
- [rails/5_0_release_notes.md at v5.0.0 · rails/rails](https://github.com/rails/rails/blob/v5.0.0/guides/source/5_0_release_notes.md#active-record-attributes-api)
- [Railsでモデルのカラムのデフォルト値をセットする方法 - 動かざることバグの如し](https://thr3a.hatenablog.com/entry/20190914/1568472769)
