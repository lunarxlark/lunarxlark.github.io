---
title : Gemfileの書き方
tags : ["Gem", "Gemfile"]
date : 2018-09-28
published : true
---

Rails公式ドキュメントからGemfileの書き方を抜粋

<!--more-->

```ruby
source 'https://rubygems.org'

# install時のlatest version
gem 'rake'

# x.x.x以上のバージョンが必要
gem 'serverspec' >=x.x.x

# x.x.x以上、y.y.y以下のバージョンが必要
gem 'serverspec' >=x.x.x, <y.y.y

# x.1からx.9はOK。メジャーバージョンが上がると不可
gem 'serverspec' ~> x.0

# version固定
gem 'rails', '3,2,1'

# always latest version
gem 'rails', :git => 'git://github.com/rails/rails.git'
```

ref: [gemfile](http://railsdoc.com/references/gemfile)
