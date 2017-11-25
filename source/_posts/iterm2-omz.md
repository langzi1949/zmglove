---
title: iTerm2+oh my zsh+agnoster 最好用的终端
date: 2017-10-08 14:10:41
tags:
    - iTerm2 
---
最近在同事的Mac上看到了一款可能是全宇宙最牛逼的终端，细细打听，原来使用的是iTerm2+omz+agnoster量身打造的。
![""](/images/iTerm_1.jpg)
<!-- more-->

* [安装iTerm2](#iTerm_1)
* [安装Oh My Bash](#iTerm_2)
* [配置agnoster主题](#iTerm_3)


<span id="iTerm_1"><font size="4">安装iTerm2</font></span>
iTerm2官方下载地址  <a href="http://www.iterm2.com/downloads.html">http://www.iterm2.com/downloads.html</a>

---
<span id="iTerm_2"><font size="4">安装Oh My Bash</font></span>
1.通过`cat /etc/shells`命令可以查看当前系统可以使用哪些shell；
```bash
# List of acceptable shells for chpass(1).
# Ftpd will not allow users to connect who are not using
# one of these shells.
/bin/bash
/bin/csh
/bin/ksh
/bin/sh
/bin/tcsh
/bin/zsh
```
2.通过`echo $SHELL`命令可以查看我们当前正在使用的shell；
```bash
# Mac系统中默认的shell为bash shell
/bin/bash
```
3.如果当前的shell不是zsh，我们可以通过`chsh -s /bin/zsh`命令可以将shell切换为shell之zsh，终端重启之后即可生效。

4.将shell切换为zsh之后，我们就可以安装Oh My ZSH了
官方推荐的安装方法为：
```bash
sh -c "$(curl -fsSL https://raw.github.com/robbyrussell/oh-my-zsh/master/tools/install.sh)"
```
出现以下提示就说明已经安装成功！
![""](/images/iTerm_2.png)

---
<span id="iTerm_3"><font size="4">配置agnoster主题</font></span>
Oh My Zsh提供的所有主题在线预览:
<a href="https://github.com/robbyrussell/oh-my-zsh/wiki/Themes">https://github.com/robbyrussell/oh-my-zsh/wiki/Themes</a>
安装成功之后，我们可以通过`vi ~/.zshrc`，设置`ZSH_THEME="agnoster"`对主题进行修改。
注意，agnoster主题能否设置成功，还依赖于以下东西：

* [Solarized Dark配色方案](http://ethanschoonover.com/solarized)
> 下载完成之后解压，在iTerm2的Preferences——Profiles——colors——Load Presets——Solarized Dark即可设置终端配色

* [特殊字体安装](https://github.com/powerline/fonts)
> 下载完成之后解压，执行其中的install.sh文件，在iTerm2的Preferences——Profiles——Text中同时将Regular Font和Non—ASCII Font设置为Meslo LG M DZ Regular for Powerline即可





