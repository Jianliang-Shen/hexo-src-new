---
title: Cmder美化WSL Ubuntu
date: 2019-08-10 14:06:52
index_img: /img/post_pics/index_img/cmder_wsl_ubuntu.JPG
tags:
    - Linux
categories: 
    - OS
---
WSL Ubuntu的界面比较简陋，推荐使用cmder改良一下终端。  
![](/img/post_pics/index_img/cmder_wsl_ubuntu.JPG)

<!-- more -->  

- [下载安装](#下载安装)
- [进入wsl ubuntu](#进入wsl-ubuntu)
- [配置](#配置)
- [修改ubutnu文件夹底色](#修改ubutnu文件夹底色)
  
## 下载安装
[cmder](https://cmder.net/)，下载完整版，直接运行即可。  
## 进入wsl ubuntu
在setting > start up中添加 `%windir%\system32\wsl.exe ~   -cur_console:p5`，进入linux子系统。  
`-cur_console:p5`是最新的为解决vim中无法使用方向键的补丁。  
## 配置
大多数配置依照个人习惯，分屏快捷键可以搜索split。  
## 修改ubutnu文件夹底色
``` bash
cd ~
dircolors -p > .dircolors
vi .dircolors
修改DIR的颜色，找到下面这段（编辑器中有配色预览和注释）

RESET 0 # reset to "normal" color                                        
DIR 04;36 # directory                                                    
LINK 01;36 # symbolic link. (If you set this to 'target' instead of a    
 # numerical value, the color is as for the file pointed to.)            
MULTIHARDLINK 00 # regular file with more than one link                  
FIFO 40;33 # pipe                                                        
SOCK 01;35 # socket                                                      
DOOR 01;35 # door                                                        
BLK 40;33;01 # block device driver                                       
CHR 40;33;01 # character device driver                                   
ORPHAN 40;31;01 # symlink to nonexistent file, or non-stat'able file ... 
MISSING 00 # ... and the files they point to                             
SETUID 37;41 # file that is setuid (u+s)                                 
SETGID 30;43 # file that is setgid (g+s)
CAPABILITY 30;41 # file with capability
STICKY_OTHER_WRITABLE 04;36 # dir that is sticky and other-writable (+t,o+w)
OTHER_WRITABLE 04;36 # dir that is other-writable (o+w) and not sticky
STICKY 37;44 # dir with the sticky bit set (+t) and not other-writable
# This is for files with execute permission:
EXEC 01;32
```
修改bashrc或者zshrc，在bashrc中有如下内容：
``` bash
if [ -x /usr/bin/dircolors ]; then                                                        
    test -r ~/.dircolors && eval "$(dircolors -b ~/.dircolors)" || eval "$(dircolors -b)" 
    alias ls='ls --color=auto'                                                            
    #alias dir='dir --color=auto'                                                         
    #alias vdir='vdir --color=auto'                                                       
                                                                                          
    alias grep='grep --color=auto'                                                        
    alias fgrep='fgrep --color=auto'                                                      
    alias egrep='egrep --color=auto'                                                      
fi                                                                                        
```
拷贝至zsh中，重新启动终端，ubutnu显示的文件夹再也不是亮瞎人的绿色。