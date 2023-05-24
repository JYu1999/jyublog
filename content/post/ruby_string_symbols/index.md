---
title: "String and Symbol in Ruby"
date: 2023-05-24T13:34:44+08:00
draft: false
tags:
  - Ruby
  - RubyMine
categories:
  - ruby_on_rails
image: "1.png"
keywords:
  - 後端
  - Backend
  - Ruby
  - Ruby on Rails
  - RoR
  - Rubymine
  - Server   
---

在寫 Rails 的時候，常常會看到使用 Symbol 的狀況：

```Ruby
class User < ApplicationRecord
  belongs_to :permission
  belongs_to :department

  has_secure_password

  validates :email, presence: true, format: { with: /\A([^@\s]+)@((?:[-a-z0-9]+\.)+[a-z]{2,})\z/i, message: "must be a valid email address"}
end
```

看了很久還是沒有很習慣這種寫法，所以來寫一篇文章整理一下自己的思緒。


---

## String
Ruby 的 string 後面是預設可以加上一些 method 的，例如：

```Ruby
name = JYu
puts name.size  # => 3
puts name.reverse # => uYJ
puts name.upcase # => JYU
```

這跟其他語言其實蠻不一樣的，比如說在 PHP 裡面要知道 String 長度的話，可能會用以下寫法：

```PHP
echo strlen("Hello");
```

這是因為 Ruby 是完全物件導向的，在 Ruby 中每樣東西都是物件，包括字串和數字。所有的方法都是對物件呼叫，不會看到全域函式[^1]。

[^1]:[Rails 實戰聖經：Ruby 程式語言入門](https://ihower.tw/rails/ruby.html) 

> String 還有很多好用的 method 可以用，但這篇文章最主要是要了解 Symbol，所以不列出太多～

---

## Symbol

Symbol 的寫法是在字前面加個冒號，如 `:name` 或 `:title`，不是英文也可以，如 `:性別`。但通常還是以英文為主。

Symbol 是一個 `Symbol` 類別的實體，可以用來表示某個狀態，例如：

```Ruby
class Order
  attr_reader :status

  def initialize(items, status = :pending)
    @items = items
    @status = status
  end

  def compete
    @status = :complete
  end
end

order = Order.new(["item A", "item B", "item C"])

if order.status == :pending
  puts "order is pending"
end
```

這裡的 `:pending` 和 `:complete` 代表的是 pending 和 complete 這兩個「狀態」。Symbol 是一種「帶有名字的物件」，正如其名，Symbol 就是符號，這個符號可用來表示「已完成」或「未完成」。[^2]
在上面這個例子，其實也可以用字串。

[^2]: [為你自己學 Ruby on Rails：數字、字串、陣列、範圍、雜湊、符號](https://railsbook.tw/chapters/06-ruby-basic-2#symbol_class)

### 所以 Symbol 和 String 的不同？

光看上面的例子可能無法知道兩者真正的差異。我們可以用 `object_id` 這個 method 去印出 Ruby 內部的記憶體配置編號：

```Ruby
puts "JYU".object_id # => 3720
puts "JYU".object_id # => 3920
puts :JYU.object_id # => 3395748
puts :JYU.object_id # => 3395748
```

可以發現兩個字串雖然內容相同，也是不同的物件；但 Symbol 只要內容相同，就是相同的物件。這種特性讓 Symbol 的主要用途是作為 Hash 的 Key。[^3]

[^3]:[Rails 實戰聖經：Ruby 程式語言入門](https://ihower.tw/rails/ruby.html)

所以 Symbol 相對於 String，也更省記憶體一些。

#### 字串的內容可以變，但 Symbol 不行
Symbol 和字串都有一些好用的 method，例如 `length`、`upcase` 或 `downcase` 等等。不過 Symbol 本身是不能被修改的，但字串可以：

```Ruby
:JYu.length # => 3
:JYu.upcase # => :JYU
:JYu[2] # => "u"
"JYu"[2] = "U" # => "U"
:JYu[2] = "U" # => undefined method `[]=' for :JYu:Symbol (NoMethodError)
```

#### Symbol 的 Comparison 比字串快

這裡我們直接來看高見龍的測試[^4]：
```Ruby
require 'benchmark'
loop_times = 100000000

str = Benchmark.measure do
  loop_times.times do
    "hello" == "hello"
  end
end.total

sym = Benchmark.measure do
  loop_times.times do
    :hello == :hello
  end
end.total

puts "Benchmark"
puts "String: #{str}"
puts "Symbol: #{sym}"

# => Benchmark
# => String: 12.299999999999999
# => Symbol: 5.750000000000002
```

Symbol 的速度明顯快很多，因為 Symbol 在比較的時候，是直接比對 `object_id`，而字串還需要逐一比對字母。所以字串在做比較的時間複雜度，會隨著字母數量而增加；而 Symbol 的時間複雜度是固定的。

[^4]: [為你自己學 Ruby on Rails：數字、字串、陣列、範圍、雜湊、符號](https://railsbook.tw/chapters/06-ruby-basic-2#symbol_class)

---

## 使用時機

### Symbol 適合當 Hash 裡的 Key

前面有提到 Symbol 的比較速度比較快，所以很適合當 Hash 的 key:

```Ruby
profile = { name: "JYu", age: 23 } # => {:name=>"JYu", :age=>23} 
```

### 字串的 method 比較多

雖然 Symbol 可以當字串用，但是 Symbol 內建的方法沒有像 String 這麼豐富，所以如果想要字串那些好用的功能，就應該選用字串。







