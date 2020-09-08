# Linux日常使用配置


# 配置软件换源
## 备份及修改
```bash
sudo cp /etc/apt/sources.list /etc/apt/sources.list_backup
sudo gedit /etc/apt/sources.list
```

## IPV6源
[点我生成配置](https://lug.ustc.edu.cn/repogen/)

# 配置Neovim
编辑配置文件`~/.config/nvim/init.vim`
```bash
call plug#begin('~/.vim/plugged')

Plug 'scrooloose/nerdtree'
Plug 'majutsushi/tagbar'

call plug#end()

let mapleader=","
syntax enable
set encoding=utf-8
set tabstop=4
set autoindent smartindent shiftround
set shiftwidth=4
set softtabstop=4
set number
set ignorecase
set background=dark
set smartcase
set nobackup
set autoread
set noerrorbells
set novisualbell
set scrolloff=7
set showmode
set expandtab
set smarttab
set shiftround


" NerdTree
    if isdirectory(expand("~/.vim/plugged/nerdtree"))
        map <C-e> :NERDTreeToggle<CR>
        map <leader>e :NERDTreeFind<CR>
        nmap <leader>nt :NERDTreeFind<CR>
		autocmd bufenter * if (winnr("$") == 1 && exists("b:NERDTreeType") &&b:NERDTreeType == "primary") | q | endif

        let NERDTreeShowBookmarks=1
        let NERDTreeIgnore=['\.py[cd]$', '\~$', '\.swo$', '\.swp$', '^\.git$', '^\.hg$', '^\.svn$', '\.bzr$']
        let NERDTreeChDirMode=0
        let NERDTreeQuitOnOpen=1
        let NERDTreeMouseMode=2
        let NERDTreeShowHidden=1
        let NERDTreeKeepTreeInNewTab=1
        let g:nerdtree_tabs_open_on_gui_startup=0
    endif

" Tabbar
     if isdirectory(expand("~/.vim/plugged/tagbar/"))
         nnoremap <silent> <leader>t :TagbarToggle<CR>
     endif

```

# 配置Shell
说实话，Ubuntu的默认Shell并不是很好用，所以我们选择fish：
```bash
sudo apt install fish
```
然后安装[oh-my-fish](https://github.com/oh-my-fish/oh-my-fish/tree/master/docs/zh-CN)即可。考虑到为了兼容BASH，所以仅将终端设置为Fish，编辑 `~/.bashrc`:
```bash
exec fish
```

# 配置tmux
tmux用于管理多终端，十分方便，以下是个人设置`.tmux.conf`
```bash
# 修改 ctrl+b 前缀为 ctrl+a
set -g prefix C-a
unbind C-b
bind C-a send-prefix
set-option -g prefix2 `
# 绑定重载 settings 的热键
bind r source-file ~/.tmux.conf \; display-message "Config reloaded.."

# 设置为vi编辑模式
setw -g mode-keys vi # 设置为vi编辑模式
bind Escape copy-mode # 绑定esc键为进入复制模式
bind -T copy-mode-vi v send-keys -X begin-selection # 绑定v键为开始选择文本
#bind -T copy-mode-vi y send-keys -X copy-selection-and-cancel # 绑定y键为复制选中文本

set-option -g default-command 'exec reattach-to-user-namespace -l fish'
bind-key -T copy-mode-vi 'y' send-keys -X copy-pipe-and-cancel 'reattach-to-user-namespace pbcopy'
bind-key -T copy-mode-vi MouseDragEnd1Pane send -X copy-pipe-and-cancel "pbcopy"

bind-key C-c run-shell 'tmux save-buffer - | reattach-to-user-namespace pbcopy'
bind-key C-v run-shell 'reattach-to-user-namespace pbpaste | tmux load-buffer - \; paste-buffer -d'

# 设置window的起始下标为1
set -g base-index 1
# 设置pane的起始下标为1
set -g pane-base-index 1

#-- base --#
set -g default-terminal "screen-256color"
set -g display-time 3000
set -g history-limit 65535

# 鼠标支持
set-option -g mouse on
# 关闭默认窗口标题
set -g set-titles off

#-- bindkeys --#
unbind '"'
bind - splitw -v -c '#{pane_current_path}'
unbind %
bind | splitw -h -c '#{pane_current_path}'

bind c new-window -c "#{pane_current_path}"

# 定义上下左右键为hjkl键
bind -r k select-pane -U
bind -r j select-pane -D
bind -r h select-pane -L
bind -r l select-pane -R

# 定义面板边缘调整的^k ^j ^h ^l快捷键
bind -r ^k resizep -U 10 # upward (prefix Ctrl+k)
bind -r ^j resizep -D 10 # downward (prefix Ctrl+j)
bind -r ^h resizep -L 10 # to the left (prefix Ctrl+h)
bind -r ^l resizep -R 10 # to the right (prefix Ctrl+l)

# 定义交换面板的键
bind ^u swap-pane -U
bind ^d swap-pane -D

bind e lastp 
bind ^e last
# bind q killp

#bind '~' splitw htop
#bind ! splitw ncmpcpp
bind m command-prompt "splitw 'exec man %%'"
bind @ command-prompt "splitw 'exec perldoc -t -f %%'"
bind * command-prompt "splitw 'exec perldoc -t -v %%'"
bind % command-prompt "splitw 'exec perldoc -t %%'"
bind / command-prompt "splitw 'exec ri -T %% | less'"

# 输出日志到桌面
bind P pipe-pane -o "cat >>~/Desktop/#W.log" \; display "Toggled logging to ~/Desktop/#W.log"

#-- statusbar --#
set -g status-right-attr bright
set -g status-bg black
set -g status-fg yellow

# 设置状态栏高亮
setw -g window-status-current-attr bright
# 设置状态栏红底白字
setw -g window-status-current-bg red
setw -g window-status-current-fg white

# 设置状态栏列表左对齐
set -g status-justify left
# 非当前window有内容更新时在状态栏通知
setw -g monitor-activity on
set -g status-interval 1

#set -g visual-activity on

setw -g automatic-rename off
setw -g allow-rename off
# 最大化（默认为z，增加模拟的b指令）
unbind b
bind b run ". ~/.tmux/zoom"

set -g status-keys vi

# plugin-manager
set -g @plugin 'tmux-plugins/tpm'
set -g @plugin 'tmux-plugins/tmux-sensible'

# plugins
set -g @plugin 'tmux-plugins/tmux-resurrect'
# set -g @plugin 'tmux-plugins/tmux-continuum'

# load-plugins-without-manager
# run-shell ~/.tmux/tmux-resurrect/resurrect.tmux
# run-shell ~/.tmux/tmux-continuum/continuum.tmux

# plugins-settings
set -g @resurrect-strategy-vim 'session' # for vim
set -g @resurrect-strategy-nvim 'session' # for neovim
# set -g @continuum-save-interval '0'
# set -g @continuum-restore 'on'

set -g status-right 'Continuum status: #{continuum_status}'
set -wg window-status-format " #I:#W "
setw -g window-status-current-format " #I:#W "
set -wg window-status-separator ""
set -g message-style "bg=#202529, fg=#91A8BA"

# set -g @resurrect-capture-pane-contents 'on' # 恢复面板内容
```

# 配置中文输入法
推荐RIME
```bash
sudo apt install ibus-rime
```

# 配置Java
## 使用APT快捷配置
```bash
sudo add-apt-repository ppa:webupd8team/java
sudo apt update
sudo apt install oracle-java8-installer
sudo update-java-alternatives -s java-8-oracle
```
当然也可以手动配置，方法如下

## 下载JDK
[下载地址](www.oracle.com/technetwork/java/javase/downloads/index.html)

## 解压安装
推荐将JDK安装到这个路径 /opt/java

## 配置环境变量
```bash
vim ~/.bashrc
```
在打开的文件的末尾添加
```bash
export JAVA_HOME=/opt/java/jdk1.8.0_162
export JRE_HOME=${JAVA_HOME}/jre
export CLASSPATH=.:${JAVA_HOME}/lib:${JRE_HOME}/lib
export PATH=$PATH:${JAVA_HOME}/bin
```
保存退出，然后输入下面的命令来使之生效
```bash
source ~/.bashrc
```

## 测试
打开一个终端，输入下面命令：
```bash
java -version
```

# 配置播放器
推荐使用SMPlayer
```bash
sudo apt install smplayer
```

# 配置录屏直播软件
obs是很有名的一个开源直播软件，并且录屏功能强大
```bash
sudo add-apt-repository ppa:kirillshkrogalev/ffmpeg-next
sudo apt-get update && sudo apt-get install ffmpeg

sudo add-apt-repository ppa:obsproject/obs-studio
sudo apt-get update && sudo apt-get install obs-studio
```

# 配置图像处理软件
推荐GIMP
```bash
sudo apt install gimp
```

# 常见问题汇总
## 文件系统错误修复
最常见的就是文件系统错误的问题，解决办法是列出分区的挂载点然后依次检查有问题的分区，具体如下：
```bash
df -hT
```
这个命令可以列出所有分区的情况，也可以查看fstab文件获取信息：
```bash
cat /etc/fastb
```
确认有问题的分区后，使用fsck命令修复分区：
```bash
fsck -y /dev/sdax
```
-y参数可以避免手动输入yes

完成后输入
```bash
exit
```
或者重启即可进入系统了，如果还有其他问题可以根据提示google依次解决。

## 屏幕亮度无法调节
一般安装对应显卡的闭源驱动即可解决该问题，如果不能可以安装 xbacklight 来间接调节亮度
```bash
sudo apt install xbacklight
```
具体使用可以参考xbacklight的使用帮助

## 无线网不可用
一般安装对应的网卡驱动即可解决该问题，最近在Linux mint 18上遇到了比较奇葩的无线网卡问题，需要在 /etc/modprobe.d/blacklist.conf 文件中增加一行
```bash
blacklist acer_wmi
```
重启后无线网卡正常工作

## 网易云音乐1.1版本无法启动和hidpi支持
编辑网易云音乐启动文件
```bash
vim /usr/share/applications/netease-cloud-music.desktop
```
主要修改`Exec`这一行，取消SESSION_MANAGER关联修复启动问题，添加--force-device-scale-factor=2强制缩放至两陪以适配hidpi
```bash
[Desktop Entry]
Version=1.0
Type=Application
Name=NetEase Cloud Music
Name[zh_CN]=网易云音乐
Name[zh_TW]=網易雲音樂
Comment=NetEase Cloud Music
Comment[zh_CN]=网易云音乐
Comment[zh_TW]=網易雲音樂
Icon=netease-cloud-music
Exec=sh -c "unset SESSION_MANAGER && netease-cloud-music --force-device-scale-factor=2 %U"
Categories=AudioVideo;Player;
Terminal=false
StartupNotify=true
StartupWMClass=netease-cloud-music
MimeType=audio/aac;audio/flac;audio/mp3;audio/mp4;audio/mpeg;audio/ogg;audio/x-ape;audio/x-flac;audio/x-mp3;audio/x-mpeg;audio/x-ms-wma;audio/x-vorbis;audio/x-vorbis+ogg;audio/x-wav;
```
