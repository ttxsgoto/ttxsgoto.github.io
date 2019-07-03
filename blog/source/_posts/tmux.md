---
title: 运维 Tmux日常
date: 2017-03-29 21:06:09
tags:
  - Tmux
categories:
  - 运维
---
#### 安装
```
# Mac中
ruby -e "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install)"
brew install tmux
# Ubuntu中
apt-get install tmux
```
Session可以包含多个Window, 每个Window又可以包含多个Pane

#### Session操作
```
tmux new -s xxx #新建session会话
C-b : new -s abc # 在当前session中新建一个session，并保证之前session依然存在
tmux a -t xxx #进入xxx会话
C-b ? #列出所有快捷键，按q返回
C-b d #当前会话，返回shell；tmux attach 重新进入之前的会话
C-b s #选择并切换会话，在开启多个会话时使用
```
#### Window操作
```
C-b c 创建一个新窗口
C-b & 关闭当前窗口
C-b w 列出所有的窗口选择
C-b 窗口号(例如窗口号为1的, 则C-b 1)
C-b , 重命名当前窗口，便于识别各个窗口
```
#### Pane操作
```
C-b % 横向分Terminal
C-b " 纵向分Terminal
C-b 方向键 在自由选择各面板
C-b x 关闭当前pane
C-b q 显示面板编号
```

#### 配置文件(~/.tmux.conf)
```
#设置前缀为Ctrl + a
set -g prefix C-a
 
#解除Ctrl+b 与前缀的对应关系
unbind C-b
 
#将r 设置为加载配置文件，并显示"reloaded!"信息
bind r source-file ~/.tmux.conf \; display "~/.tmux Reloaded!"
 
#window 水平分割和纵向分割
bind | split-window -h -c
bind - split-window -v -c
 
#方向移动设置
#up
bind -n C-k select-pane -U
#down
bind -n C-j select-pane -D
#left
bind -n C-h select-pane -L
#right
bind -n C-l select-pane -R
 
#设置鼠标操作
#set -g mouse on
 
# 设置终端类型为256色
#set -g default-terminal "screen-256color"
 
# 设置窗口分割的边框颜色
set -g pane-border-fg green
set -g pane-border-bg black
 
# 设置当前窗口分割的边框颜色
#set -g pane-active-border-fg white
#set -g pane-active-border-bg yellow
 
set -g status-bg colour236
set -g status-fg colour68
 
#copy-mode 将快捷键设置为vi 模式
setw -g mode-keys vi
 
# 设置状态栏左部宽度  默认为10
set -g status-left-length 35
# 设置状态栏左部显示内容。
set -g status-left "#[fg=colour252,bold,bg=colour243] 🌺  S: #S #[fg=colour250,bg=colour239] 🏵  W: #I #[fg=yellow,bg=colour237] ☘ #[fg=colour250,bg=colour237] P: #P#[default]"
# 设置状态栏右部宽度
set -g status-right-length 48
# 设置状态栏右部内容，这里设置为时间信息
set -g status-right "#[fg=colour251,bold,bg=colour237] 🗓  %Y-%b-%d #[fg=colour251,bold,bg=colour239] ⏱  %R #[fg=colour251,bold,bg=colour243] 🐳  #(ifconfig | grep 'inet.*netmask.*broadcast' | awk '{print $2}')"
# 窗口信息居中显示
set -g status-justify centre
# 设置状态栏更新时间 每60秒更新一次，默认是15秒更新
set -g status-interval 60
```
#### 参考链接
https://github.com/liuchengxu/dotfiles/blob/master/tmux.conf
https://github.com/wklken/k-tmux/blob/master/tmux.conf
https://my.oschina.net/am313/blog/865915
https://gist.github.com/ryerh/14b7c24dfd623ef8edc7



