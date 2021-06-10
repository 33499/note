#### inux使用

``` shell
su #切换root用户
su ysx #切换用户
touch file1 #创建一个文件
touch file2 file3 file4 #创建 多个文件
touch folder3/f3 #在folder3文件夹下创建f3文件

mkdir folder3 #创建文件夹
mkdir folder3/f3 #在folder3下创建f3文件夹

cp  file1 file1copy #把file1文件内容复制到file1copy文件内容中
cp -i file1 file1copy #i代表interactive互动的意思，输入y表示确定覆盖file1copy文件
cp file1 folder1/ #把file1文件复制到folder1文件夹下
cp -R folder1/ folder2/ #把folder1文件夹下的文件全部都复制到folder2文件夹下
cp file* folder2/ #把文件名以file开头的文件全部复制到folder2文件夹下
cp file1copy file2 folder1/ #把前面的文件复制到folder1文件夹下

mv file1 folder1/ #把file文件移动到folder1文件夹下面
mv file2 file2rename #相当于重命名file2文件

rm file3 #删除文件3
rm * #删除当前文件夹下所有
rm -r folder3 #删除有文件的文件夹 r代表递归删除，一个一个文件删除

cat t.py #把t.py内容放到屏幕上显示
cat t.py > t1.py #把t.py内容放到t1.py中
cat t.py t1.py > t2.py #把t.py和t1.py整合放到t2.py中

```

> 文件权限

drwxr-xr-x     d 代表是文件夹directory, r:read, w: write, x:execution

每三个分别代表user，group，others对这个文件的权限

```shell
chmod u+r t1.py  #修改user的权限，加上可读
ls -l t1.py  #查看文件权限
chmod u-r t1.py #减去可读权限，此时不可cat t1.py
chmod a-r t1.py #减去user，group, others所以的可读权限
chmod ug+rwx t1.py
chmod ug-rw t1.py
```

``` shell
tar -zxvf v2.26.2.tar.gz  #解压文件
tar -zxvf /root/jdk-8u161-linux-x64.tar.gz -C ./     #解压文件解压到当前文件夹下

```

**configure文件是一个可执行的脚本文件，它有很多选项，在待安装的源码目录下使用命令./configure –help可以输出详细的选项列表。**

***其中--prefix选项是配置安装目录，如果不配置该选项，安装后可执行文件默认放在/usr /local/bin，库文件默认放在/usr/local/lib，配置文件默认放在/usr/local/etc，其它的资源文件放在/usr /local/share，比较凌乱。***

如果配置了--prefix，如：

$ ./configure --prefix=/usr/local/test1

安装后的所有资源文件都会被放在/usr/local/test目录中，不会分散到其他目录。

```shell
#指定安装的路径，configure是一个可执行的脚本语句，产生makefile文件,配置编译安装环境
./configure prefix=/usr/local/python3  

#make与makeinstall是两个命令，在你./configuration生成了Makefile之后执行编译安装；
make && make install

#为某一个文件在另外一个位置建立一个同不的链接，这个命令最常用的参数是-s
ln -s [源文件] [目标文件]
ln -s /usr/local/python3/bin/python3 /usr/bin/python3 
ln -s /usr/local/python3/bin/pip3 /usr/bin/pip3

rpm -qa|grep mariadb  #查询已安装的安装包
yum -y remove mariadb-server-5.5.56-2.el7.x86_64  #删除
```

### 进程管理

``` bash
ps -aux|grep mysql   #ps -aux 查看所有的进程
#| 在Linux中叫管道符  A|B
# grep 查找文件中符合条件的字符串!

#进程树
pstree -pu
     -p 显示父id
     -u 显示用户组
kill -9 进程的id
```

### 防火墙

``` bash
#查看防火墙状态
systemctl status firewalld 
#开启firewalld.service服务
service firewalld start
#重启
service firewalld restart
#关闭
service firewalld stop

#查看防火墙规则
firewall-cmd --list-all  #查看全部信息
firewall-cmd --list-ports #查看端口信息

#开启端口
firewall-cmd --zone=public --add-port=80/tcp --permanent #开端口命令
systemctl restart firewalld.service #重启防火墙
#命令含义
--zone #作用域
--add-port=80/tcp #添加端口，格式为：端口/通讯协议
--permanent #永久生效，没有此参数重启后失效
```

### 前后端打包部署

***vue前端***   

linux上打包，部署（用Nginx服务器）

```shell
#在项目目录下
#安装依赖
npm install --unsafe-perm --registry=https://registry.npm.taobao.org
#打包生成dist
npm run build:prod


```

修改nginx.conf文件，修改root和修改location加上转发代理。

***凡是到prod-api的请求路径都转发到后端服务器地址上***

![image-20201123193055567](C:\Users\ysx\AppData\Roaming\Typora\typora-user-images\image-20201123193055567.png)

![image-20201123193215634](C:\Users\ysx\AppData\Roaming\Typora\typora-user-images\image-20201123193215634.png)



springboot后端

```shell
#打jar包
mvn package
#运行jar包
nohup java -jar ruoyi.jar &
#查看运行的jar包
ps -aux | grep java
ps -ef|grep tomcat #查看tomcat运行的端口号
kill -9 端口号
#实施检测日志文件
tail -f filename
```

网卡设置

```shell
vim /etc/sysconfig/network-scripts/ifcfg-ens33
systemctl restart network.service
dhclient #自动获取ip
```

<mark>**查看ssh的安装包 ：rpm -qa | grep ssh**</mark>
<mark>**查看ssh是否安装成功 ：ps -ef | grep ssh**</mark>

```
yum install -y yum-utils


yum-config-manager \
    --add-repo \
    http://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo
```