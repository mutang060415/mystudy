# Linux最常用的命令

### 一、Linux的目录结构

![Linux目录结构](C:\Users\mutang\Desktop\python\Linux目录结构.jpg)

```python
bin (binaries)---存放二进制可执行文件
sbin (super user binaries)---存放二进制可执行文件，只有root才能访问
etc (etcetera)---存放系统配置文件
usr (unix shared resources)---用于存放共享的系统资源
home ---存放用户文件的根目录
root ---超级用户目录
dev (devices)---用于存放设备文件
lib (library)---存放跟文件系统中的程序运行所需要的共享库及内核模块
mnt (mount)---系统管理员安装临时文件系统的安装点
boot ---存放用于系统引导时使用的各种文件
tmp (temporary)---用于存放各种临时文件
var (variable)---用于存放运行时需要改变数据的文件
```

### 二、Linux常用命令

```python
命令格式：命令 -选项 参数 （选项和参数可以为空）
如：ls -la /usr
```

##### 2.1 操作文件及目录

![Linux操作文件及目录1](C:\Users\mutang\Desktop\python\Linux操作文件及目录1.png)

![Linux操作文件及目录2](C:\Users\mutang\Desktop\python\Linux操作文件及目录2.jpg)

![Linux操作文件及目录3](C:\Users\mutang\Desktop\python\Linux操作文件及目录3.png)

##### 2.2 系统常用命令

![Linux系统常用命令1](C:\Users\mutang\Desktop\python\Linux系统常用命令1.png)

![Linux系统常用命令2](C:\Users\mutang\Desktop\python\Linux系统常用命令2.png)

![Linux系统常用命令3](C:\Users\mutang\Desktop\python\Linux系统常用命令3.png)

##### 2.3 压缩/解压缩

![Linux压缩解压缩命令](C:\Users\mutang\Desktop\python\Linux压缩解压缩命令.png)

##### 2.4 文件权限操作

```
Linux文件权限的描述格式解读：
```

![Linux文件权限解读1](C:\Users\mutang\Desktop\python\Linux文件权限解读1.jpg)

```python
r ---可读权限，w ---可写权限，x ---可执行权限（也可以用二进制表示 111 110 100 --> 764）
第1位：文件类型（d 代表目录，- 代表普通文件，l 代表链接文件）
第2-4位：所属用户权限，用u（user）表示
第5-7位：所属组权限，用g（group）表示
第8-10位：其他用户权限，用o（other）表示
第2-10位：表示所有的权限，用a（all）表示
```

![Linux权限解读2](C:\Users\mutang\Desktop\python\Linux权限解读2.jpg)

### 三、Linux系统常用快捷键及符号命令 

![Linux常用快捷键](C:\Users\mutang\Desktop\python\Linux常用快捷键.png)

### 四、vim编辑器

```
vim是Linux上最常用的文本编辑器而且功能非常强大。只有命令，没有菜单！
```

![Linuxvim编辑器](C:\Users\mutang\Desktop\python\Linuxvim编辑器.jpg)

##### 4.1 修改文本

![Linux编辑器修改文本](C:\Users\mutang\Desktop\python\Linux编辑器修改文本.png)

##### 4.2 定位命令

![Linux编辑器定位命令](C:\Users\mutang\Desktop\python\Linux编辑器定位命令.png)

##### 4.3 替换和取消命令

![Linux编辑器替换和取消命令](C:\Users\mutang\Desktop\python\Linux编辑器替换和取消命令.png)

##### 4.4 删除命令

![Linux编辑器删除命令](C:\Users\mutang\Desktop\python\Linux编辑器删除命令.png)

##### 4.5 常用快捷键

![Linux编辑器常用快捷键](C:\Users\mutang\Desktop\python\Linux编辑器常用快捷键.png)

### 五：定时任务

![定时任务](C:\Users\mutang\Desktop\python\定时任务.png)

```
crontab -u //设定特定用户的定时服务
crontab -l //列出当前用户定时服务内容
crontab -r //删除当前用户的定时服务
crontab -e //编辑当前用户的定时服务

分 时 日 月 周 执行的命令
简单点儿记就是分时日月周，*代表“每”的意思

使用(-)可以划定范围
如：0 0-3 * * *  脚本  表示每天0-3点整执行脚本

使用(,)可以枚举时间
如: 0,15,30,45 * * * * 脚本  表示每个小时的0分，15分，30分，45分会执行脚本

使用(/)可以指定间隔
如：* */8 * * * 脚本  表示每8小时执行脚本

组合用法
0-20/10 * * * * 脚本  表示在前20分钟内每隔10分钟执行脚本

举例：
①30 3，12 * * * /bin/sh /scripts/oldboy.sh
每天凌晨3点半和中午12点半的时刻执行/scripts/oldboy.sh脚本

②30 */6 * * * /bin/sh /scripts/oldboy.sh
每6小时30分执行一次/scripts/oldboy.sh脚本

③30 8-18/2 * * * /bin/sh /scripts/oldboy.sh
在每天的8点到18点之间，每隔2小时的   半点时刻执行/scripts/oldboy.sh

④30 21 * * * /application/apache/bin/apachectl graceful
每天晚上9点半重启apache

⑤45 4 1,10,22 * * /application/apache/bin/apachectl graceful
每月1日10日22日的4点45分重启apache

⑥10 1 * * 6,0 /application/apache/bin/apachectl graceful
每周六和周日的凌晨1点10分重启apache

⑦0,30 18-23 * * * /application/apache/bin/apachectl graceful
每天的18点到23点每隔30分钟重启一次apache。
提示：最后一次执行任务时23：30分

⑧0 */1 * * * /application/apache/bin/apachectl graceful
每小时重启一次apache
```











