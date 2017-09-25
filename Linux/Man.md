## 鸟哥的Linux私房菜

### 第五章 首次登入与在线求助 man page


#### X window

1. 重新启动 X Window的方法
 - 直接注销
 - [Alt] + [Ctrl] + [Backspace]
2. X window 与文本模式的切换

 Linux预设的情况下会提供留个Terminal来让使用者登入，切换的方式为使用：[Ctrl] + [Alt] + [F1] ~ [F6]
 - [Ctrl] + [Alt] + [F1] ~ [F6] : 文字登入接口 tty1~tty6 终端机；
 - [Ctrl] + [Alt] + [F7] : 图形接口桌面

#### 基础指令的操作，date,cal, bc
1. 指令
`command [-options] parameter1 parameter2`
2. 显示日期：date
3. 显示日历：`cal [month][year]`
4. 简单的计算器：bc

#### 重要的几个热键 [Tab],[Ctrl]-C, [Ctrl]-D

1. [Tab] 
  功能：【命令补全】和【档案补全】

2. [Ctrl] + C
  功能：将正在运作中的指令中断

3. [Ctrl] + D
  功能：【键盘输入接受（End Of File）】，也可以代替 exit 的输入。离开文字接口。


#### man page

man中数字所代表的意义：

| 代号 | 代表内容 |
| -- | -- |
| 1 | 用户在 shell 环境中可以操作的指令或者可执行文件 |
| 2 | 系统核心可呼出的函数与工具等 |
| 3 | 一些常用的函数和函数库，大部分为 C 的函数库(libc) |
| 4 | 装置档案的说明，通常在/dev 下的档案 |
| 5 | 配置文件或者是某些档案的格式 |
| 6 | 游戏 |
| 7 | 惯例与协议等，例如Linux文件系统、网络协议、ASCII code 等等的说明 |
| 8 | 系统管理员可空的管理指令 |
| 9 | 根kernel有关的文件 | 


man page 内容

| 代号 | 内容说明 |
| -- | -- |
| NAME | 简短的指令、数据名称说明 |
| SYNOPSIS | 简短的指令下达语法(syntax)简介 |
| DESCRIPTION | 较为完整的说明，这部分最好仔细看看 |
| OPTIONS | 针对SYNOPSIS部分中，有列举的所有可用的选项说明 |
| COMMANDS | 当这个程序（软件）在执行的时候，可以在此程序（软件）中下达的指令 |
| FILES | 这个程序或数据所使用或者参考或连结到的某些档案 |
| SEE ALSO | 可以参考的，跟这个指令或数据有关的其他说明 |
| EXAMPLE | 一些可以参考的范例 |
| BUGS |  |

man 进入后可以使用的按键

| 按键 | 进行工作 |
| -- | -- |
| 空格键 | 向下翻一页 |
| [Page Down] | 向下翻一页 |
| [Page Up] | 向上翻一页 |
| [Home] | 回到第一页 |
| [End] | 回到最后一页 |
| /string | 向下搜索 |
| ?string | 向上搜索 |
| n,N | 利用/或者？来搜索字符串时，可以用n来继续下一个搜索，可以用 N 来进行【反向】搜索 |
| q | 结束这次的man page |

man page 的数据通常情况下放在/usr/share/man目录下
man -f [指令名称] 获取更多与指令相关的信息，相当于指令 whatis
man -k [关键词] 根据关键词查找指令，相当于指令 apropos

#### info page


#### nano

#### 关机 sync,shutdown,reboot,halt,poweroff,init

