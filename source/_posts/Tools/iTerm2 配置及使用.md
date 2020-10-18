---
title: iTerm2 配置及使用技巧
date: 2018-04-20 21:09:40
tags: 开发工具
categories:
  - 开发工具
---

## 1. 技巧：

### 1. 选中

双击选中，三击选中整行，选中即复制。即任何选中状态的字符串都被放到了系统剪切板中。

### 2. command

- 可以拖拽选中的字符串；
- 点击 url：调用默认浏览器访问该网址；
- 点击文件：调用默认程序打开文件

### 3. 快捷键

- 切换 tab：⌘+←, ⌘+→
- 切换分屏：⌘+[/]
- 新建 tab：⌘+t；
- 切分屏幕：⌘+d 水平切分，⌘+Shift+d 垂直切分；
- 智能查找，支持正则查找：⌘+f。

| 命令                                                 | 说明           |
| ---------------------------------------------------- | -------------- |
| command + t                                          | 新建标签       |
| command + w                                          | 关闭标签       |
| command + 数字 command + 左右方向键                  | 切换标签       |
| command + enter                                      | 切换全屏       |
| command + f                                          | 查找           |
| command + d                                          | 垂直分屏       |
| command + shift + d                                  | 水平分屏       |
| command + option + 方向键 command + [ 或 command + ] | 切换屏幕       |
| command + ;                                          | 查看历史命令   |
| command + shift + h                                  | 查看剪贴板历史 |
| ctrl + u                                             | 清除当前行     |
| ctrl + l                                             | 清屏           |
| ctrl + a                                             | 到行首         |
| ctrl + e                                             | 到行尾         |
| ctrl + f/b                                           | 前进后退       |
| ctrl + p                                             | 上一条命令     |
| ctrl + r                                             | 搜索命令历史   |

### 4. 自动完成

- cmd+; 自动完成
- cmd+shift+h 历史记录

### 5. autojump

- 使用 j 目录名，无需完整的即可。
- 输入 d 显示之前跳过的目录

### 6. suggestions

```Bash
git clone https://github.com/zsh-users/zsh-autosuggestions ~/.oh-my-zsh/custom/plugins/zsh-autosuggestions.git
```

在 .zshrc 中添加 `zsh-autosuggestions` 接着执行如下命令

```Bash
➜  ~ source .zshrc
➜  ~ source .oh-my-zsh/custom/plugins/zsh-autosuggestions/zsh-autosuggestions.zsh
```

即可使用了！

