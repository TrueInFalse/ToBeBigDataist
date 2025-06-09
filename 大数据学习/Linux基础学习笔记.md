# Linux基础


## Linux基础知识清单

| 类别          | 关键词                                                    | 建议内容               |
| ----------- | ------------------------------------------------------ | ------------------ |
| **文件系统操作**  | `ls` `cd` `rm` `mv` `cp` `touch` `mkdir` `find` `grep` | 最基本的生存能力           |
| **权限和用户管理** | `chmod` `chown` `sudo` `passwd` `su` `useradd`         | 学会读懂 `drwxr-xr-x`  |
| **进程与网络管理** | `ps` `top` `netstat` `ping` `lsof`                     | 日常排查进程问题、端口冲突      |
| **压缩打包**    | `tar` `zip` `unzip` `gzip`                             | 部署必备               |
| **文本处理三剑客** | `awk` `sed` `cut`                                      | 数据处理神器（后期超有用）      |
| **脚本与调度**   | `for/while` `if` `cron` `bash`                         | 自动化调度 Shell 脚本     |
| **日志定位**    | `tail -f` `cat` `less` `grep`                          | 分析 Hive 报错、HDFS 状态 |


- Linux内核、虚拟机、查看虚拟机网卡状态Win+R输入`ncpa.cpl`查看是否有VMnet1和VMnet8；
- Linux命令行效率远大于图形化界面；`ifconfig`显示当前系统所有的网络接口的详细参数；Linux中路径使用`/`；
- Linux命令基础格式：`command [-options：可选，控制命令执行细节] [parameter：可选，命令的参数，命令指向的目标]`
- `ls`平铺当前工作目录下内容；`ls [-a(看所有文件，包括隐藏文件) -l(竖着输出) -h(以易于阅读的形式现实文件大小)] [指定目录]`
- `cd [指定路径]`：切换工作目录（转到指定目录或者返回最初HOME）；`pwd`查看当前目录；
- 相对路径与绝对路径：`cd`：`cd ..`退一级，`cd ../..`退两级，`~`表示HOME目录，`.`当前目录；
- `mkdir [-p（多级创建）] Linux路径`；-p多级创建就是那一条路径上的没有的文件夹全部创建出来；
- `touch Linux路径`：创建文件；`cat`(查看全部内容)、`more`(查看部分内容，空格翻页，q退出)；
- `cp [-r（复制文件夹需要加上）] 参数1 参数2`，`mv 参数1 参数2`（可以改名字）；
- `rm [-r(用于删除文件夹) -f(强制删除)] 参数1 参数2 ... 参数N`；`*`通配符，`test*`以test开头的，`*test`以test结尾的， `*test*`只要是包含test的；
- `which 要查找的命令`；
- find按照文件名查找文件：`find 起始命令 -name "被查找文件名"`，find+通配符，find按照文件大小查找文件：`find 起始路径 -size +|-n[kMG]`,+、-表示大于小于，n表示大小数字，kMG分别表示kb、MB、GB
- `grep [-n] 关键字 文件路径`：从文件中通过关键字过滤文件行；
- `wc [-c -m -l -w] 文件路径`：-c统计bytes数量，-m统计字符数量，-l统计行数，-w统计单词数量，什么都不加的时候就是`行数词数字节数`；
- `命令1 | 命令2` 管道符，将左边的结果作为右边的输入；例如`cat test.txt | grep itheima`
- `echo "输出的内容"`：在命令行内输出指定内容（prin函数）；“反引号 ` ”，反引号(飘号)内的内容会被当做命令执行；
- 重定向符：`>`将左侧命令的结果**覆盖**写入到符号右侧指定的文件中，`>>`将左侧命令的结果**追加**写入到符号右侧指定的文件中（下一行）；
- `tail [-f -num] Linux路径`，-f表示持续跟踪，-num表示查看尾部多少行默认10：用于查看文件尾部的内容，跟踪文件的最新更改。
- vim编辑器：命令模式（很多快捷键）、输入模式、底线命令模式（:进入）
- `su [-(环境变量，最好添加)] [用户名]`：用于切换账户，扩减权限；`sudo 其他命令`：临时以root身份执行，并非所有用户有用sudo权限，需要为普通用户配置sudo认证；
- 用户与用户组：`groupadd 用户组名`、`groupdel 用户组名`分别为添加删除用户组；`useradd[-g -d]用户名`、`userdel[-r]用户名`、`id[]`、`usermod -aG 用户组 用户名 `：创建用户、删除用户、查看用户所属组、修改用户所属组；`getent passwd/group`查看系统中全部用户信息/全部用户组信息，表示`[用户名][密码（x）][用户ID][组ID][描述信息（无用）][HOME目录][执行终端（默认bash）]`。
- 查看权限控制信息：
  - <img src="https://github.com/user-attachments/assets/7f429978-4c68-4b30-9849-5d1774f0e89d" width="600">
  - r、w、x：代表读权限(eg:ls)、写权限、执行权限(eg:cd)
- `chmod [-R] 权限 文件或文件夹`：修改文件文件夹的权限信息，-R对文件夹内全部内容应用同样的操作；
  - 0：---、1：--x、2：-w-、3：-wx、4：r--、5：r-x、6：rw-、7：rwx（r=4、w=2、x=1）
- `chown [-R] [用户][:][用户组] 文件或文件夹`：修改文件文件夹的所属用户和用户组（此命令限于root用户）
- `Ctrl+c`强制退出、`Ctrl+d`退出或者登出、`history`历史命令搜索(!倒着搜索最近的命令)、`Ctrl+r`输入内容匹配历史命令
- `yum [-y] [install | remove | search] 软件名称`
- `systemctl start | stop | status | enable | disable 服务名`
- `ln -s 参数1 参数2`；创建软连接，类似于快捷方式，比如`ln -s /etc/yum.conf ~/yum.conf`
- `date [-d] [+格式化字符串]`：可以在命令行中查看系统的时间，-d一般用于日期计算；修改Linux时区；
- IP地址：`ifconfig`来查看，`127.0.0.1`代指本机，`0.0.0.0`代指本机·确定绑定关系·允许任意IP访问；主机名`hostname`，修改`hostnamectl set-hostname 新的名字`；域名解析，替代直接记忆IP地址；
- 网络请求和下载：`ping [-c num] ip或主机名`，检查指定的网络服务器是否是可联通状态；`wget [-b] url`，在命令行内下载网络文件，-b后台下载；`curl [-O] url`，发送http网络请求用于下载文件获取信息等，-O是否下载；
- Linux超大号小区，6万多端口，`公认端口：1~1023；注册端口：1024~49151；动态端口：49152~65535`；
- 查看端口占用：`nmap 被查看的IP地址`，`netstat -anp|grep 端口号`；
- 进程管理：`ps [-e -f]`，查看Linux系统中的进程信息（-e显示全部进程，-f展示全部信息，一般-ef）；`kill [-9] 进程ID`，关闭进程（-9表示强制关闭）；
- 查看系统资源占用：`top`直接进入，默认5秒刷新一次，Ctrl+c或者q退出；有很多参数可选：-p、-d、-c、-n、-b、-i、-u；top支持交互式选型（多，各种按键）；
  - <img src="https://github.com/user-attachments/assets/85648e1a-686b-48a8-b916-39c8e48c75c8" width="300">
  - <img src="https://github.com/user-attachments/assets/eba4fb7b-232b-4278-af7d-76743f3cd1f4" width="300">
  - <img src="https://github.com/user-attachments/assets/d7d1288d-2016-413b-8c32-2b66141c7114" width="300">
  - <img src="https://github.com/user-attachments/assets/df2f7876-2fa5-4915-bc09-11db7556a254" width="300">

- 磁盘信息监控：`df [-h]`查看磁盘使用情况，-h更人性化展示；`iostat [-x][num1][num2]`，-x显示更多信息，num1刷新间隔，num2刷新次数；
  - <img src="https://github.com/user-attachments/assets/210481cd-b54a-4675-988a-450d97ef88aa" width="300">
- 网络状态监控：`sar -n DEV num1 num2`，查看网络的相关统计信息
- 环境变量：`env`直接查看Linux中的环境变量（key=value键值对形式的）；`PATH`记录了系统执行任何命令的搜索路径（：隔开）；`$符号`相当于`${key}=value`函数，输出value；
- 自行设置环境变量：临时设置`export 变量名=变量值`；永久生效需要更改~/.bashrc文件（当前用户），/etc/profile文件（所有用户）；
- 上传、下载：图形化（FinalShell最下方）；`rz：上传（很慢，建议拖拽） sz：下载`；
- 压缩与解压：`tar [-c -v -x -f -z -C] 参数1 参数2 ... 参数N`，`.tar 简单封装 .gz 极大压缩`；常用组合`-cvf tar模式`、`-zcvf gz模式`
  
```
-c，创建压缩文件，用于压缩模式；
-v ，显示压缩、解压过程，用于查看进度；
-x ，解压模式；
-f ，要创建的文件，或要解压的文件（-f 选项必须在所有选项中位置处于最后一个）；
-z ， gzip 模式，不使用 -z 就是普通的 tarball 格式；
-C ，选择解压的目的地，用于解压模式
```

- `zip [-r] 参数1 参数2 ... 参数N`：压缩文件为zip压缩包；`unzip [-d] 参数`：解压zip压缩包（-d指定解压位置）；






















