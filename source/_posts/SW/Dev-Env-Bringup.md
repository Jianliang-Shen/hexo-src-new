---
title: 工作环境和工具使用
date: 2023-08-10 19:11:42
tags:
    - Linux
    - Tool
categories:
    - Software
---

软件开发常用配置。

<!-- more -->

## vscode配置

拼写检查插件：code spell checker
修改代码长度垂直标尺：editor.rulers
代码格式化风格：
```json
    "C_Cpp.clang_format_fallbackStyle": "{
        BasedOnStyle: Google,
        UseTab: Never,
        IndentWidth: 4,
        TabWidth: 4,
        BreakBeforeBraces: Attach,
        AllowShortIfStatementsOnASingleLine: false,
        IndentCaseLabels: false,
        ColumnLimit: 100,
        AccessModifierOffset: -4,
        NamespaceIndentation: All,
        FixNamespaceComments: false
    }",
```

关闭git作者和日期提示：

```json
"gitlens.codeLens.enabled": false, 
"gitlens.codeLens.authors.enabled": false
```

TODO highlight关键字:
```json
    "todohighlight.keywords": [
        {
            "text": "TODO:",
            "color": "red",
            "backgroundColor": "yellow",
        },
    ]
```
## ssh远程

```bash
ssh-keygen
cat cat .ssh/id_rsa.pub

# 在服务器
echo "<pub>" >> .ssh/authorized_keys

# 在客户端
ssh <user-name>@<host-ip>
```

修改别名，打开`.ssh/config`

```txt
Host <Name>
  HostName <IP>
  User <User name>

代理跳转：

Host <Name>
  HostName <跳板机IP>
  User <跳板机用户>
  ProxyJump <目标机用户>@<目标机IP>
```

### Powershell 添加alias

参考[使用powershell的set-alias与function 为ssh设置别名](https://juejin.cn/post/6844903621486723086)

```txt
function ssh-vega20-ip {
    ssh shenjianliang@172.21.20.225
}

set-alias vega20 ssh-vega20-ip
```

## zsh

<details>
<summary>zsh安装和主题设置</summary>

```bash
brew install zsh
sudo apt-get install zsh
chsh -s /bin/zsh
sh -c "$(curl -fsSL https://raw.github.com/ohmyzsh/ohmyzsh/master/tools/install.sh)"
# sh -c "$(wget https://raw.github.com/ohmyzsh/ohmyzsh/master/tools/install.sh -O -)"

# 主题
git clone --depth=1 https://github.com/romkatv/powerlevel10k.git ${ZSH_CUSTOM:-$HOME/.oh-my-zsh/custom}/themes/powerlevel10k
# ZSH_THEME="powerlevel10k/powerlevel10k"

# 自动补全
git clone https://github.com/zsh-users/zsh-autosuggestions ${ZSH_CUSTOM:-~/.oh-my-zsh/custom}/plugins/zsh-autosuggestions
# plugins=(
#     # other plugins...
#     zsh-autosuggestions  # 插件之间使用空格隔开
# )

# 语法
git clone https://github.com/zsh-users/zsh-syntax-highlighting.git ${ZSH_CUSTOM:-~/.oh-my-zsh/custom}/plugins/zsh-syntax-highlighting

# 使配置生效
source ~/.zshrc

# 安装z跳转， 参考 https://cloud.tencent.com/developer/article/1694849
git clone https://github.com/rupa/z.git

# 遇到窗口大小的问题
apt-get install xterm
echo "resize >> /dev/null" >> ~/.zshrc
```

</details>

## docker快速使用

<details>
<summary>docker guideline</summary>

```bash
$ docker images

$ docker rmi <images_id>   # 删除镜像
$ docker rm <container_id> # 删除容器

$ docker run -id -w /root --name sjl_docker \
    -v /path/to/testcase:/testspace -v /opt/rocm:/opt/rocm \
    -v /etc/localtime:/etc/localtime -v /etc/timezone:/etc/timezone \
    --privileged \
    --device=/dev/kfd --device=/dev/dri \
    --group-add video \                    
    10.65.42.71:9092/rocm-env:ubuntu-18.04

$ docker exec -it sjl_docker bash # 进入容器命令行，zsh则进入zsh
$ docker commit <container_id>    # 生成镜像

# 创建容器
$ docker run -id -w /root -v /etc/localtime:/etc/localtime -v /etc/timezone:/etc/ timezone -v /sys/kernel/debug:/sys/kernel/debug --privileged --device=/dev/kfd --device=/dev/dri --group-add video --network=host --name=sjl 10.65.42.71:9092/rocm-env:ubuntu-18.04
 
# 拷贝.vim .vimrc .zshrc .oh-my-zsh 搭建环境
docker cp myfiles sjl:/root
 
# 保存镜像
$ docker commit -a sjl sjl sjl_prj:v1.0 # -a 指定作者
$ docker images
REPOSITORY                     TAG                 IMAGE ID            CREATED             SIZE
sjl_prj                        v1.0                cef938d062da        4 seconds ago       7.39GB
 
# 打包
$ docker save -o sjl_prj_v1.0.tar sjl_prj
$ tar zcvf sjl_prj_v1.0.tar.gz sjl_prj_v1.0.tar
# 或者运行 docker save sjl_prj | gzip > sjl_prj_v1.0.tar.gz
 
 
$ tar zxvf sjl_prj_v1.0.tar.gz
$ docker load -i sjl_prj_v1.0.tar
be2ff0588f45: Loading layer [==================================================>]  2.703GB/2.703GB
aa2ce01e6fd8: Loading layer [==================================================>]  666.5MB/666.5MB
Loaded image: sjl_prj:v1.0
$ docker images
REPOSITORY                            TAG            IMAGE ID       CREATED                  SIZE
sjl_prj                               v1.0           cef938d062da   Less than a second ago   7.39GB
 
# 启动 docker
$ docker run -id -w /root -v /home/shenjianliang/docker_rocm5.2:/work --privileged --device=/dev/kfd --device=/dev/dri --group-add video --network=host --name=sjl_rocm sjl_prj:v1.0

```

</details>

## 建立Samba和Windows访问

```bash
udo apt-get install samba
sudo cp /etc/samba/smb.conf /etc/samba/smb.conf.bak
sudo vim /etc/samba/smb.conf
 
sudo chmod 777 /home/shenjianliang/work
sudo smbpasswd -a shenjianliang    # 创建密码
/etc/init.d/smbd restart  # 选择自己的账户名
```

<details>
<summary>smb.conf</summary>

```txt
[global]
   guest account = shenjianliang
 
security = user
[public]
    path = /home/shenjianliang/work
    public = yes
    writeable = yes
    browseable = yes
    guest ok = yes
    create mask = 0644
    acl allow execute always = yes
```

</details>

打开windows文件管理器右键，添加一个网络位置 `\\<ip>\public`认证即可。

![](/img/os/samba.png)

## 配置Vim

在～下新建`.vimrc`文件

<details>
<summary>.vimrc</summary>

```bash
" File: .vimrc
" Author: Jake Zimmerman <jake@zimmerman.io>
"
" How I configure Vim :P
"

" Gotta be first
set nocompatible

filetype off

set rtp+=~/.vim/bundle/Vundle.vim
call vundle#begin()

Plugin 'VundleVim/Vundle.vim'

" ----- Making Vim look good ------------------------------------------
Plugin 'altercation/vim-colors-solarized'
Plugin 'tomasr/molokai'
Plugin 'vim-airline/vim-airline'
Plugin 'vim-airline/vim-airline-themes'

" ----- Vim as a programmer's text editor -----------------------------
Plugin 'scrooloose/nerdtree'
Plugin 'jistr/vim-nerdtree-tabs'
" Plugin 'vim-syntastic/syntastic'
Plugin 'xolox/vim-misc'
Plugin 'xolox/vim-easytags'
Plugin 'majutsushi/tagbar'
Plugin 'ctrlpvim/ctrlp.vim'
Plugin 'vim-scripts/a.vim'

" ----- Working with Git ----------------------------------------------
Plugin 'airblade/vim-gitgutter'
Plugin 'tpope/vim-fugitive'

" ----- Other text editing features -----------------------------------
Plugin 'Raimondi/delimitMate'

" ----- man pages, tmux -----------------------------------------------
Plugin 'jez/vim-superman'
Plugin 'christoomey/vim-tmux-navigator'

" ----- Syntax plugins ------------------------------------------------
Plugin 'jez/vim-c0'
Plugin 'jez/vim-ispc'
Plugin 'kchmck/vim-coffee-script'

" ---- Extras/Advanced plugins ----------------------------------------
" Highlight and strip trailing whitespace
"Plugin 'ntpeters/vim-better-whitespace'
" Easily surround chunks of text
"Plugin 'tpope/vim-surround'
" Align CSV files at commas, align Markdown tables, and more
"Plugin 'godlygeek/tabular'
" Automaticall insert the closing HTML tag
"Plugin 'HTML-AutoCloseTag'
" Make tmux look like vim-airline (read README for extra instructions)
"Plugin 'edkolev/tmuxline.vim'
" All the other syntax plugins I use
"Plugin 'ekalinin/Dockerfile.vim'
"Plugin 'digitaltoad/vim-jade'
"Plugin 'tpope/vim-liquid'
"Plugin 'cakebaker/scss-syntax.vim'

call vundle#end()

filetype plugin indent on

" --- General settings ---
set backspace=indent,eol,start
set ruler
set number
set showcmd
set incsearch
set hlsearch

syntax on

set mouse=a

" We need this for plugins like Syntastic and vim-gitgutter which put symbols
" in the sign column.
hi clear SignColumn

" ----- Plugin-Specific Settings --------------------------------------

" ----- altercation/vim-colors-solarized settings -----
" Toggle this to "light" for light colorscheme
set background=dark

" Uncomment the next line if your terminal is not configured for solarized
"let g:solarized_termcolors=256

" Set the colorscheme
" colorscheme solarized


" ----- bling/vim-airline settings -----
" Always show statusbar
set laststatus=2

" Fancy arrow symbols, requires a patched font
" To install a patched font, run over to
"     https://github.com/abertsch/Menlo-for-Powerline
" download all the .ttf files, double-click on them and click "Install"
" Finally, uncomment the next line
"let g:airline_powerline_fonts = 1

" Show PASTE if in paste mode
let g:airline_detect_paste=1

" Show airline for tabs too
let g:airline#extensions#tabline#enabled = 1

" Use the solarized theme for the Airline status bar
let g:airline_theme='solarized'

" ----- jistr/vim-nerdtree-tabs -----
" Open/close NERDTree Tabs with \t
nmap <silent> <leader>t :NERDTreeTabsToggle<CR>
" To have NERDTree always open on startup
let g:nerdtree_tabs_open_on_console_startup = 1


" ----- scrooloose/syntastic settings -----
let g:syntastic_error_symbol = '✘'
let g:syntastic_warning_symbol = "▲"
augroup mySyntastic
  au!
  au FileType tex let b:syntastic_mode = "passive"
augroup END


" ----- xolox/vim-easytags settings -----
" Where to look for tags files
set tags=./tags;,~/.vimtags
" Sensible defaults
let g:easytags_events = ['BufReadPost', 'BufWritePost']
let g:easytags_async = 1
let g:easytags_dynamic_files = 2
let g:easytags_resolve_links = 1
let g:easytags_suppress_ctags_warning = 1

" ----- majutsushi/tagbar settings -----
" Open/close tagbar with \b
nmap <silent> <leader>b :TagbarToggle<CR>
" Uncomment to open tagbar automatically whenever possible
"autocmd BufEnter * nested :call tagbar#autoopen(0)


" ----- airblade/vim-gitgutter settings -----
" In vim-airline, only display "hunks" if the diff is non-zero
let g:airline#extensions#hunks#non_zero_only = 1


" ----- Raimondi/delimitMate settings -----
let delimitMate_expand_cr = 1
augroup mydelimitMate
  au!
  au FileType markdown let b:delimitMate_nesting_quotes = ["`"]
  au FileType tex let b:delimitMate_quotes = ""
  au FileType tex let b:delimitMate_matchpairs = "(:),[:],{:},`:'"
  au FileType python let b:delimitMate_nesting_quotes = ['"', "'"]
augroup END

" ----- jez/vim-superman settings -----
" better man page support
noremap K :SuperMan <cword><CR>
```

</details>

下载vim插件管理软件：
```bash
git clone https://github.com/VundleVim/Vundle.vim.git ~/.vim/bundle/Vundle.vim
vim +PluginInstall +qall
```

## Mac配置iTerm2

参考[iTerm2安装配置使用指南——保姆级](https://zhuanlan.zhihu.com/p/550022490)
