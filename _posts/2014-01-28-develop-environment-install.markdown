---
layout: post
title: 'Develop environment install note'
date: 2014-01-28 12:06:00
categories: System osx
---

[Solarized](http://ethanschoonover.com/solarized)
[SublimeText 3 Document](http://www.sublimetext.com/docs/3/osx_command_line.html)
[SublimeText 3 Tips](http://blog.generalassemb.ly/sublime-text-3-tips-tricks-shortcuts/)
[SublimeText 3 Theme](http://www.masnun.com/2013/07/08/beautiful-themes-for-sublime-text-3.html)
[SublimeText 3 Javascript plugin](http://www.wikihow.com/Create-a-Javascript-Console-in-Sublime-Text)
[iTerm2](http://www.iterm2.com/#/section/home)
[iTerm2 Color Themes](http://iterm2colorschemes.com/)
[iTerm2 login with zsh](http://stackoverflow.com/questions/1276703/how-to-make-zsh-run-as-a-login-shell-on-mac-os-x-in-iterm)
[oh-my-zsh](https://github.com/robbyrussell/oh-my-zsh)
備註：會直接詢問密碼強迫更改成 default zsh ，使用 `iTerm2` 建議不要直接改成預設。
接著在`~/.zshrc`修改樣板爲`agnoster`並在最下方補上 `export DEFAULT_USER=[name]`。

{% highlight bash %}
$ chsh -s /bin/bash # Mofidy to default bash
{% endhighlight %}


{% highlight js %}
export DEFAULT_USER=""
{% endhighlight %}

{% highlight js %}
"caret_style": "phase",
"caret_extra_bottom": 0,
"caret_extra_top": 0,
"caret_extra_width": 2,
{% endhighlight %}

[~/.ssh/config](http://www.lainme.com/doku.php/blog/2011/01/%E4%BD%BF%E7%94%A8ssh_config)




