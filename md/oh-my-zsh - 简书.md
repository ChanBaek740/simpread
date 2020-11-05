\> 本文由 \[简悦 SimpRead\](http://ksria.com/simpread/) 转码， 原文地址 \[www.jianshu.com\](https://www.jianshu.com/p/2f1ba0e9dc6b)

*   查看当前环境 shell

```
echo $SHELL
```

*   查看系统自带哪些 shell

```
cat /etc/shells
```

*   如果没有`/bin/zsh`的话，安装 zsh

```
yum install zsh # centos
brew install zsh # mac
```

*   将 zsh 设置为默认的 shell

```
chsh -s /bin/zsh
```

*   再次确认下`echo $SHELL`看下当前默认 shell 有没有改，没有的话要重启终端。
    
*   安装 oh-my-zsh
    

```
git clone git://github.com/robbyrussell/oh-my-zsh.git ~/.oh-my-zsh
cp ~/.oh-my-zsh/templates/zshrc.zsh-template ~/.zshrc
```

*   查看可用的主题

```
ls ~/.oh-my-zsh/themes
```

*   修改主题，编辑`~/.zshrc`

```
ZSH\_THEME="candy"
```

> *   参考各种插件快捷键等  
>     [https://segmentfault.com/a/1190000013612471](https://links.jianshu.com/go?to=https%3A%2F%2Fsegmentfault.com%2Fa%2F1190000013612471)

*   超炫酷的主题  
    [https://blog.biezhi.me/2018/11/build-a-beautiful-mac-terminal-environment.html](https://links.jianshu.com/go?to=https%3A%2F%2Fblog.biezhi.me%2F2018%2F11%2Fbuild-a-beautiful-mac-terminal-environment.html)

*   安装自动补全插件

```
cd ~/.oh-my-zsh/custom/plugins/
git clone https://github.com/zsh-users/zsh-autosuggestions
vi ~/.zshrc
```

加上插件

![](http://upload-images.jianshu.io/upload_images/7720139-165b1ae7e02e3dbd.png)

补全命令只要输入前几个词，按⬆️按键就可以找历史记录

*   安装自动高亮插件

```
cd ~/.oh-my-zsh/custom/plugins/
git clone https://github.com/zsh-users/zsh-syntax-highlighting.git
vi ~/.zshrc
```

请务必保证插件顺序，zsh-syntax-highlighting 必须在最后一个。

![](http://upload-images.jianshu.io/upload_images/7720139-b4d13cf3dd910712.png)

然后在文件的最后一行添加：source ~/.oh-my-zsh/custom/plugins/zsh-syntax-highlighting/zsh-syntax-highlighting.zsh

> 总结：人不装逼会死

![](http://upload-images.jianshu.io/upload_images/7720139-c633b1a675c08de6.png) image.png

补充：[oh-my-zsh 主题支持 conda 虚拟环境](https://links.jianshu.com/go?to=https%3A%2F%2Fwww.muzaozong.com%2F%25E6%259D%2582%25E9%25A1%25B9%2Foh-my-zsh%25E4%25B8%25BB%25E9%25A2%2598%25E6%2594%25AF%25E6%258C%2581conda%25E8%2599%259A%25E6%258B%259F%25E7%258E%25AF%25E5%25A2%2583%2F%23%25E5%2589%258D%25E6%258F%2590)