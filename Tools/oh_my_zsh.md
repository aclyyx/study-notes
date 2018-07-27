# 打造功能强大的命令行

今天才刚刚发现这么强大的工具，好东西什么时候知道都不晚， __Oh My Zsh__ 配合 __iTerm2__ 一起服用效果更佳。  

## 安装 Oh My Zsh
安装方法请参考 [__Oh My Zsh__ 官方 GitHub 说明][1]

## 配置 Oh My Zsh

安装后，通过命令``vi ~/.zshrc``编辑 zsh 配置文件实现对其进行配置。  

### 给命令行加点颜色

在``.zshrc``文件末尾添加下面的配置信息。 

````
#开启颜色
autoload -U colors && colors

export PS1="%{$fg[red]%}%D %*%{$reset_color%} %B%n%b%{$fg[green]%}@%{$reset_color%}%{$fg[white]%}%d%b%{$reset_color%} $ "
````

### 自动提示功能

安装 __zsh-autosuggestions__ 添加自动提示功能，安装方法请看 [zsh-autosuggestions 官方 GitHub 说明][3]。

## 安装 iTerm

到[__这里__][2]下载并安装即可。

[1]: https://github.com/robbyrussell/oh-my-zsh "Oh My Zsh"
[2]: https://www.iterm2.com "iTerm2 官网"
[3]: https://github.com/zsh-users/zsh-autosuggestions "zsh-autosuggestions"