---
layout: post
title: 'Ruby 快速筆記'
date: 2015-04-04 12:00:00
categories: Ruby Rails
---

## Ruby 快速筆記

* Ruby 中的字串需要包在 `"` 或者 `'` 中不過兩者有一點微妙的差別，在雙引號中可以使用逸出字元(\n \r \\) 等等，不過單引號會原封不動的輸出除了 `\\` 例外
* 在呼叫方法的時候 Ruby 可以省略 () 
* puts 和 print 的差別是 puts 會在行末加上 \n，如果是用 , 分開不同的參數輸出則每個參數後面都會有 \n

* 第三個輸出的 method 是 p ，puts 和 print 不管輸出的是數字或字串都只會顯示內容

{% highlight ruby %}
puts "1" # => 1
puts 1   # => 1
p "1" # => "1"
p 1   # => 1
{% endhighlight %}

* 用 p 輸出可以得知值的型別，值得注意的是如果用 p 輸出，單引號和雙引號會有差別 因為 p 不會轉譯逸出字元所以雙引號會被直接輸出

{% highlight ruby %}
p "Hello \n" # => "Hello \n"
# 但是單引號本來就會直接輸出逸出字元, 如下:
puts '\n' # => \n 所以骨子裡是 '\\n'

# 因此遇上 p 的時候會輸出 `\\n`
p '\n' #= '\\n'
{% endhighlight %}

* 設定語系

{% highlight bash %}
$ ruby -E utf-8 say_chinese.rb
{% endhighlight %}

* 陣列的多種初始化方式

{% highlight ruby %}
a = Array.new # => []
a = [] # => []

# Array.new(size, default_value)
a = Array(2, 3) # => [3, 3]

{% endhighlight %}

* 取得 User Input

{% highlight ruby %}
text = gets.chomp
{% endhighlight %}

* 格式化文字

{% highlight ruby %}
"String".upcase # => STRING
"String".downcase # => string
"string".capitalize # => String
{% endhighlight %}

* 條件控制式

{% highlight ruby %}
if / elsif / else

if true
  puts "Yes"
elsif true
  puts "I don't know"
else
  puts "No"
end
{% endhighlight %}

* unless

{% highlight ruby %}
unless false => excute
unless true => do nothing
unless = if not

# unless = if not

unless false
  puts 'It will excute'
end
{% endhighlight %}

* while

{% highlight ruby %}
counter = 1
while counter < 11
  puts counter
  counter = counter + 1
end
{% endhighlight %}

* while true => 執行 => false 停止
* until false => 執行 => true 就停止

* for

{% highlight ruby %}
for num in 1..3 # 1, 2, 3
  puts num
end

for num in 1...3 # 1, 2
  puts num
end
{% endhighlight %}

* loop

{% highlight ruby %}
loop { print "Hello, World!" }

i = 0
loop do
  i += 1
  puts i
  break if i > 5
end

# 迴圈中 skip
loop do
  i += 1
  next if i % 2 == 0
  print i
end

array.each do |i|
 puts i
end

5.times do
  puts "Ruby!"
end
{% endhighlight %}


* 轉字串

{% highlight ruby %}
1.to_s
{% endhighlight %}

* `reverse` 陣列的方法，單純反轉 index

{% highlight ruby %}
puts "Please input words: "
text = gets.chomp

words = text.split(' ')

frequencies = Hash.new(0)
words.each { | word | frequencies[word]  += 1}

frequencies = frequencies.sort_by{ | word, frequence | frequence }
frequencies.reverse!

frequencies.each do | word, frequence |
    puts word + " " + frequence.to_s
end
{% endhighlight %}

* is_a?

{% highlight ruby %}
1.is_a? Integer # => true
"string".is_a? Integer # => false
{% endhighlight %}

* 確認該值為 Boolean

{% highlight ruby %}
# checking whether foo is a boolean
!!foo == foo
{% endhighlight %}

* 字串轉 symbol / symbol 轉字串

{% highlight ruby %}
"string".to_sym
:symbol.to_s
{% endhighlight %}

* 關於排序

{% highlight ruby %}
arr = [4, 3, 2, 1]
arr.sort!

arr.sort! { |x, y| x <=> y} # 小到大
arr.sort! { |x, y| y <=> x} # 大到小
arr.sort! do 
  if x > y
     1
  elsif x < y
     -1
  else
     0
  end
end

# x > y = 1, x < y = -1 時是小到大
{% endhighlight %}

* Hash

{% highlight ruby %}
hash = Hash.new
# 設定預設值
hash = Hash.new('a default')
hash['nonexists'] # => a default

hash = {}
hash = {
  "one": 1
}

hash = {
  :one => 1
}

hash = {
  one: 1
}

hash = Hash["xxx" => 1]

movie_rating = {
  iron_man: 4,
  super_man: 3
}
good_movies = movie_rating.select { |k, v| v > 2 }

movie_rating.each_value { |v| puts v }
movie_rating.each_key { | k | puts k }
{% endhighlight %}

* 產出 Range Array

{% highlight ruby %}
a_z = ("a".."z").to_a
[1, 2, 3].zip([4, 5, 6]) #=> [[1, 4], [2, 5], [3, 6]]
{% endhighlight %}

* 計算效能

{% highlight ruby %}
performance = Benchmark.realtime do
  # do your code
end
{% endhighlight %}

* 因為 block 本身不是物件，所以不能夠用變數來存取他。也因此有了 Proc 物件的存在。

* Proc 物件的用法可以在加入 method 的參數使用 `&` 

{% highlight ruby %}
def greeter
  yield
end

phrase = Proc.new { puts "Hello, Ruby!" }

greeter &phrase

# 或者直接運行
phrase.call

# 進階用法把 method 當作 Proc 傳入
strings = ["1", "2", "3"]
nums = strings.map(&:to_i)

# 範例
def greeter
    yield("your name")
end

hi = Proc.new { |name| puts "Hi #{name}" }
hi.call
greeter &hi
{% endhighlight %}

## 相等的 methods

* collect = map
* to_sym = intern

* lambda 與 Proc
Proc 和 lambda 都是物件，大致上行為一致
例如:

{% highlight ruby %}
lambda { puts "Hello!" }
Proc.new { puts "Hello!" }
{% endhighlight %}

最主要的兩個差別是 `lambda` 會確認 arguments 的數量，Proc 不會
Proc 會幫你把沒傳入的參數帶入 nil

{% highlight ruby %}
l = lambda {|x, y| puts x  }
p = Proc.new { |x, y| puts x }
l.call(1) # ArgumentError: wrong number of arguments (1 for 2)
p.call(1) # Work
{% endhighlight %}

第二個最恐怖的差異是 Proc return 之後就不回原本的 method 了控制權由他掌控，而 lambda 會把控制權交回給呼叫的 method

* yield 指的是呼叫 block 這件事，而 Proc 是用來包 block 的物件，lambda 也是不過骨子裡還是 Proc 物件。`&` 的意思是把 Proc 或 lambda 轉回 block 所以我們可以這樣用

{% highlight ruby %}
def meow
 yield
end
l = lambda { puts "I'm lambda" }
p = Proc.new { puts "I am Proc" }
meow &l
meow &p

# 當要把 block 當作實際參數時則

def bark(o)
  o.call
end
bark l
bark p
{% endhighlight %}

## 歸納 block, Proc, lambda
* 一個 `block` 只是一小段程式碼被包在 `do ... end` 或者 `{ }` 裡面。他本身並不是一個物件，不過他可以被傳進方法(method) e.g `each`, `select`, `collect`, `map`，由於其不是一個物件所以不能夠直接存進變數中。

* 一個 Proc 物件可以儲存一個 block，然後我們可以重複使用。

* lambda 就跟 Proc 很類似，不過它在乎您傳入的參數的數量，且當 method 呼叫時他會把控制權交回給 method 並不像 Proc 會立即回傳。

{% highlight ruby %}
def batman_ironman_proc
  victor = Proc.new { return "Batman will win!" }
  victor.call
  "Iron Man will win!"
end

puts batman_ironman_proc

def batman_ironman_lambda
  victor = lambda { return "Batman will win!" }
  victor.call
  "Iron Man will win!"
end

puts batman_ironman_lambda
{% endhighlight %}

* 其 Proc 的指標。如果要取得該 Proc 的指標，需要在最後一個參數前面加上 ’&’，這東西只能有一個，且必須放在最後面，否則都會跳出 syntax 

{% highlight ruby %}
def f3(n, p)
  p[n] # call proc p
  # 'p[n]' is equivalent to 'p.call(n)'
  # 'yield n' will not work unless a block was given, but notice that the block has nothing to do with parameter 'p'
end
f3('Tony', Proc.new{|name| puts name}) # 'Proc.new' is equivalent to 'Kernel::proc'
{% endhighlight %}

* & 的意義是當我們把 block 傳進定義的 method 參數時把該 block 轉成 Proc (背地裏是取得該指標)

[參考文章 Proc & lambda](http://tonytonyjan.net/2011/08/12/ruby-block-proc-lambda/)

## [百分比符號的用法](http://teohm.com/blog/2012/10/15/start-using-ruby-percent-notation/)

{% highlight ruby %}
%(interpolated string (#{ "default" }))
  #=> "interpolated string (default)"
%Q(interpolated string (#{ "default" }))
  #=> "interpolated string (default)"
%q(non-interpolated string)
  #=> "non-interpolated string"
%r(#{ "interpolated" } regexp)i
  #=> /interpolated regexp/i
%w(non-interpolated\ string  separated\ by\ whitespaces)
  #=> ['non-interpolated string', 'separated by whitespaces']
%W(interpolated\ string #{ "separated by whitespaces" })
  #=> ['interpolated string', 'separated by whitespaces']
%s(non-interpolated symbol)
  #=> :'non-interpolated symbol'
%x(echo #{ "interpolated shell command" })
  #=> "interpolated shell command\n"
{% endhighlight %}