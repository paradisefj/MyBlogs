## 鸟哥的Linux私房菜

### 第三章 主机规划与磁盘分区

#### 各硬件装置在Linux中的文件名

| 装置 | 装置在Linux中的文件名 |
| ---- | ---- |
| IDE硬盘 | /dev/hd[a-d] |
| SCSI/SATA/USB 硬盘机 | /dev/sd[a-p] |
| USB快闪碟 | /dev/sd[a-p] | 
| 软盘 | /dev/fd[0-1] | 
| 打印机 | 25针：/dev/lp[0-2] <br/> USB: /dev/usb/lp[0-15] |
| 鼠标 | USB: /dev/usb/mouse[0-15] <br/>  PS2: /dev/psaux |
| 当前CDROM/DVDROM | /dev/cdrom |
| 当前的鼠标 | /dev/mouse |
| 磁带机 | IDE: /dev/ht0  <br/> SCSI: /dev/st0 | 


#### 磁盘分区

磁盘的第一个扇区记录了：
1. 主要启动记录区（MBR）：可以安装开机管理程序的地方；
2. 分割表：记录整个硬盘的分割状态，有64 bytes；

延伸分割和逻辑分割的特性：
1. 主要分割与延伸分割最多可以有四个
2. 延伸分割最多只能有一个
3. 逻辑分割是有延伸分割持续分割出来的分隔槽
4. 能够被格式化后，作为数据存取的分隔槽为主要分割和逻辑分割，延伸分割无法格式化
5. 逻辑分割的数量依操作系统而不同，在Linux系统中，IDE磁盘最多有59个逻辑分割（5号到63号），SATA硬盘则有11个逻辑分割（5号到15号）
6. 如果延伸分割被破坏，所有的逻辑分割都会被删除

文件系统与目录树的关系（挂载）

所谓的【挂载】就是利用一个目录当成进入点，将磁盘分区槽的数据放置在该目录下；也就是说，进入该目录就可以读取该分割槽的意思。