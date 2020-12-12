---
title: Rails 複習筆記
categories: Program
tags: [ruby rails]
---

```bash
# 安裝 GPG keys
# 參考官網 https://rvm.io/rvm/install
$ gpg --keyserver hkp://keys.gnupg.net --recv-keys 409B6B1796C275462A1703113804BB82D39DC0E3 7D2BAF1CF37B13E2069D6956105BD0E739499BDB
$ curl -sSL https://get.rvm.io | bash

# 更新 rvm
$ rvm get head

# 列出可用的 ruby
$ rvm list known

# 安裝
$ rvm install 2.4.1

# 列出 local 安裝的 ruby
$ rvm list

# 切換版本
$ rvm use 2.4.1

# 設定預設值
$ rvm use 2.4.1 --default

# 使用系統預設 ruby
$ rvm system

# 移除
$ rvm uninstall 2.4.1

# 查看 rvm 資訊
$ rvm info

# 安裝 rails
$ gem install rails -v 5.1.1

# 啟動 server
# bin/ 是 binstub
$ bin/rails server
$ bundle exec rails server
$ rails server # 為了避免使用跟 Gemfile 不同版本的指令，建議使用上兩者

```


```
name # 區域變數 defalut: 
$name # 全域變數 defualt: nil
@name # 實體變數 default: nil
@@name # 類別變數 default: 
```

引號的其他方式

```
%Q() = ""
%q() = ''
```

字串與 Symbol

```ruby
"name".to_sym
# :name

"name".intern
# :name

%s(name)
# :name

:name.to_s
# "name"

:name.id2name
# "name"
```