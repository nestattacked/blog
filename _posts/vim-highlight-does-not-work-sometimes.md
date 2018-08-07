---
title: VIM语法高亮偶尔失效
excerpt: 使用VIM编辑文本或者跳转到文本的其他位置，有时会出现语法高亮失效的情况，这是能够解决的。
recommend: 5
tags:
  - VIM
categories:
  - 技术
  - TroubleShoot
date: 2018-08-07 21:54:58
thumbnail: cover.jpg
---
# 失效的原因

在处理奇怪的输入时，VIM可能会变成不知所措，因为它并不知道如何处理奇怪的输入，这最终会导致语法高亮失效甚至是错误高亮。例如在编写VUE的单文件组件文件时，由于js、css、html代码共存，VIM需要依据三种语法进行高亮，这极易引起混乱从而导致语法高亮失效。

# 解决办法

## syntax sync fromstart

`syntax sync fromstart` 是VIM用来重新从头分析语法高亮的命令，该命令能够重新矫正语法高亮，从而解决失效的问题。所以当语法高亮失效的时候，只要在命令模式下执行该命令即可。

## 自动执行

如果觉得手动输入麻烦，可以直接在配置文件中写入下方代码：

```
autocmd BufEnter * :syntax sync fromstart
```

配置完成后，每当我们输入文本，VIM就会执行一次矫正来保证高亮的正确。这样做会导致性能下降，但实际使用情况还行，可能在编辑大文件的情况下会出现卡顿吧。

## map

如果性能出现问题，我们只能回到手动矫正的办法了。为了方便，可以通过VIM的`map`，将矫正命令映射到某些快捷键上。下方代码能够映射到`F12`上，代码写入配置文件即可。

```
noremap <F12> <Esc>:syntax sync fromstart<CR>
inoremap <F12> <C-o>:syntax sync fromstart<CR>
```

# 参考资料

1. [Fix syntax highlighting](http://vim.wikia.com/wiki/Fix_syntax_highlighting)
2. [Learn Vimscript the Hard Way](http://learnvimscriptthehardway.stevelosh.com/)
