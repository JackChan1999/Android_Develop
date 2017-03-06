```
1. 查看目录下有什么文件/目录
    > ls            //list列出目录的文件信息
    > ls  -l 或ll   //list -list以“详细信息”查看目录文件
    > ls  -a        //list  -all查看目录“全部”(包括隐藏文件)文件
    > ls  -al       //list  -all list 查看目录“全部”(包括隐藏文件)文件,以“详细信息”展示
    > ls  目录      //查看指定目录下有什么文件
    > ls -i         //查看文件索引号码

2. 进行目录切换
    > cd  dirname       //进行目录切换
    > cd  ..            //向上级目录切换
    > cd  ~    或 cd     //直接切换到当前用户对应的家目录

3. 查看完整的操作位置
    > pwd

4. 用户切换
    > su -  或  su - root       //向root用户切换
    > exit          //退回到原用户

    > su 用户名     //普通用户切换

    多次使用su指令，会造成用户的“叠加”：
    (su和exit最好匹配使用)
    jinnan--->root--->jinnan--->root--->jinnan

5. 查看当前用户是谁
    > whoami

6. 图形界面 与 命令界面 切换
    root用户可以切换
    ># init 3
    ># init 5

7. 查看一个指令对应的执行程序文件在哪
    > which  指令


8. 目录相关操作
    1) 创建目录 make directory
    > mkdir  目录名字
    > mkdir -p newdir/newdir/newdir       //递归方式创建多个连续目录

      //新的多级目录数目如果大于等于2个，就要使用-p参数
      mkdir      dir/newdir                //不用-p参数
      mkdir  -p  dir/newdir/newdir         //使用-p参数
      mkdir  -p  newdir/newdir/newdir      //使用-p参数

    2) 移动目录(文件和目录)  move
    > mv  dir1  dir2            //把dir1移动到dir2目录下
    > mv  dir1/dir2  dir3       //把dir2移动到dir3目录下
    > mv  dir1/dir2  dir3/dir4  //把dir2移动到dir4目录下
    > mv  dir1/dir2  ./         //把dir2移动到当前目录下

    3) 改名字  (文件和目录)
    > mv  dir1  newdir          //修改dir1的名字为newdir

    mv是“移动” 和 “改名字” 合并的指令
    > mv  dir1  ./newdir            //dir1移动到当前目录下 并改名字为newdir
    > mv  dir1/dir2  dir3           //dir2移动到dir3目录下， 并改名字为“原名”
    > mv  dir1/dir2  dir3/newdir    //dir2移动到dir3目录下，并改名字为“newdir”
    > mv  dir1/dir2  dir3/dir4      //dir2移动到dir4目录下， 并改名字为“原名”
    > mv  dir1/dir2  dir3/dir4/newdir   //dir2移动到dir4目录下， 并改名字为“newdir”

    4) 复制(改名字)(文件和目录) copy
    ① 文件的复制
    > cp  file1  dir/newfile2         //file1被复制一份到dir目录下，并改名字为“newfile2”
    > cp  file1  dir               //file1被复制一份到dir目录下，并改名字为“原名”
    > cp  dir1/filea  dir2/newfile  //filea被复制一份到dir2目录下，并改名字为“newfile”
    ② 目录的复制(需要设置-r[recursive递归]参数，无视目录的层次)
    > cp -r dir1   dir2             //dir1被复制到dir2目录下,并改名字为"原名"
    > cp -r  dir1/dir2  dir3/newdir  //dir2被复制到dir3目录下,并改名字为"newdir"
    > cp -r  dir1/dir2  dir3/dir4   //dir2被复制到dir4目录下,并改名字为"原名"
    > cp -r  dir1/dir2  dir3/dir4/newdir   //dir2被复制到dir4目录下,并改名字为"newdir"
    > cp -r  dir1  ../../newdir     //dir1被复制到上两级目录下,并改名字为"newdir"

    ⑤ 删除(文件和目录)remove
    > rm  文件
    > rm -r  目录           //-r[recursive递归]递归方式删除目录
    > rm -rf  文件/目录     //-r force  递归强制方式删除文件
                            force强制，不需要额外的提示
      rm  -rf  /

9. 文件操作
    1) 查看文件内容
        cat  filename       //打印文件内容到输出终端
        more  filename      //通过敲回车方式逐行查看文件的各个行内容
                            //默认从第一行开始查看
                            //不支持回看
                            //q 退出查看

        less                //通过“上下左右”键查看文件的各个部分内容
                            //支持回看
                            //q 退出查看

        head -n filename    //查看文件的前n行内容
        tail -n filename    //查看文件的最末尾n行内容

        wc filename         //查看文件的行数

    2) 创建文件
        > touch  dir1/filename
        > touch  filename
    3) 给文件追加内容
        > echo 内容 > 文件名称      //把“内容”以[覆盖写]方式追加给“文件”
        > echo 内容 >>  文件名称    //把“内容”以[追加]形式写给“文件”
        (如果文件不存在会创建文件)

10. 用户操作
    配置文件：/etc/passwd
    1) 创建用户 user add
    ># useradd
    ># useradd  liming          //创建liming用户，同时会创建一个同名的组出来
    ># useradd  -g 组别编号  username   //把用户的组别设置好，避免创建同名的组出来
    ># useradd  -g 组编号  -u 用户编号  -d 家目录   username

    2) 修改用户 user modify
    ># usermod  -g 组编号  -u 用户编号  -d 家目录  -l 新名字  username
    (修改家目录时需要手动创建之)

    3) 删除用户 user delete
    ># userdel  username
    ># userdel -r  username    //删除用户同时删除其家目录


    4) 给用户设置密码，使其登录系统
    > passwd  用户名

11. 组别操作
    配置文件： /etc/group
    1) 创建组 group add
    ># groupadd  music
    ># groupadd  movie
    ># groupadd  php

    2) 修改组 group modify
    ># groupmod  -g gid  -n 新名字  groupname

    3) 删除组 group delete
    ># groupdel  groupname    //组下边如果有用户存在，就禁止删除

12. 查看指令可设置的参数
    > man 指令

13. 给文件设置权限
    1) 字母相对方式设置权限
    //  针对一个组别设置权限，其他组别权限没有变化，称为“相对方式”权限设置
    chmod指令
    chmod u+rwx  filename  //给filename文件的主人增加“读、写、执行”权限
    chmod g-rx  filename   //给filename文件的同组用户 删除“读、执行”权限

    chmod u+/-rwx,g+/-rwx,o+/-rwx  filename
    说明：
    ① 每个单元"+"  "-"只能使用一个
    ② 可以同时给一个组或多个组设置权限，组别之间使用","分割
    ③ 每个单元的权限可以是"rwx"中的一个或多个
    >chmod u+w,g-rx,o+rw  filename   //给filename文件主人增加写权限，同组删除读、执行权限，其他组增加读、写权限
    >chmod u+w,u-x  filename     //给filename文件主人“增加写权限”同时“删除执行权限”

    chmod +/-rwx  filename  //无视具体组别，统一给全部的组设置权限
    >chmod +rw  filename    //给filename全部用户增加“读、写”权限

    2) 数字绝对方式设置权限
    r读:4      w写:2      x执行:1
    0: 没有权限
    1：执行
    2：写
    3：写、执行
    4：读
    5：读、执行
    6：读、写
    7：读、写、执行

    chmod  ABC  filename    //ABC分别代表主人、同组、其他组用户的数字权限
    >chmod 753  filename    //主人读、写、执行；同组读、执行；其他组写、执行

    问：字母相对 和 数字绝对 方式权限设置取舍？
    答：修改的权限相对“比较少”的时候使用“字母”方式
        相反，权限变动“非常多”的时候就使用“数字”方式


14. 在文件中查找内容
    grep  被搜寻内容   文件
    > grep  hello   passwd      //在passwd文件中搜索hello内容
                                //会把hello所在行的内容都打印到终端显示

15. 计算文件占据磁盘空间大小
    > du  -h  文件(目录)


16. 文件查找
    find  查找目录  选项 选项值  选项 选项值 ...

    1) -name选项 根据名字进行查找
        > find  /  -name  passwd[完整名称]      //"递归遍历"系统全部目录，寻找名称等于"passwd"的文件
        > find  /  -name  "pas*"[模糊查找]      //在系统全部目录，模糊查找一个名字是“pas”开始的文件
        > find  /  -name  "*er*"                //文件名字有出现“er”字样即可，不要位置
    2) 限制查找的目录层次 -maxdepth  -mindepth
       -maxdepth 限制查找的最深目录
       -mindepth 限制查找的最浅目录
       > find  /  -maxdepth 4 -name passwd
       > find  /  -maxdepth 4 -mindepth 3 -name passwd
    3) 根据大小为条件进行文件查找
        -size  +/-数字
                +号表示大小大于某个范围
                -号表示大小小于某个范围
        大小单位：
            -size  5    //单位是“512字节”  5*512字节
            -size  10c  //单位是“字节”     10字节
            -size  3k   //单位是“千字节”   3*1024字节
            -size  6M   //单位是“1024*千字节”   6M兆字节
        > find  ./  -size  14c     //在当前目录查找大小等于14字节的文件
        > find  /  -size +50M       //在系统全部目录里边查找大小大于50M的文件
```
# Linux-根目录介绍

##  /bin      
binary  二进制
许多“指令”对应的可“执行程序文件”目录
ls   pwd   init等等

## /sbin
super  binary  超级的 二进制
许多“指令”对应的可“执行程序文件”目录
该目录文件对应指令都是"root"用户可以执行的指令
例如：init

## /usr    
unix  system  resource (unix系统资源文件目录)
该目录类似win系统的 C:/Program  files 目录
该目录经常用于安装各种软件

软件安装完毕会形成对应的指令，该指令对应的可执行程序文件就存放在以下目录
/usr/bin
许多“指令”对应的可“执行程序文件”目录
/usr/sbin
root用户执行的指令 对应的 可“执行程序文件”目录

## /dev
device  系统硬件设备目录(linux系统所有的硬件都通过文件表示)
例如：/dev/cdrom是光驱
      /dev/sda  是第一块scsi硬盘

## /home
用户的"家目录"
给系统每增加一个“普通用户”的同时，都会在该目录为该用户设置一个文件目录
代表该用户的“家目录”，用户后期使用系统的时候会首先进入其家目录
家目录名字默认与当前用户名字一致
用户对家目录拥有绝对最高的权限。


## /root
该目录是root管理员的家目录，root用户登录系统后首先进入该目录

## /proc
内存映射目录，该目录可以查看系统的相关硬件信息

## /var
variable  可变的、易变的
该目录存储的文件经常会发生变动(增加、修改、删除)
经常用于部署项目程序文件
/var/www/shop
/var/www/book

## /boot
系统启动核心目录，用于存储系统启动文件

## /etc
系统主要配置文件目录 例如  
/etc/passwd  用于存储用户信息的文件  
/etc/group   用于存储组别信息的文件

## /lib
library
系统资源文件类库目录

## /selinux
secure enhanced  linux 安全增强型linux
对系统形成保护
会对给系统安装软件时有干扰作用
