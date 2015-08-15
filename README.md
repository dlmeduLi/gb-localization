# UCSC Genome Browser 的本地化安装
> UCSC Genome Browser的安装主要参考UCSC Genome Browser官方网站的指导进行。安装过程主要按照源代码包中的设置说明文档进行。各部分的安装设置参考的文件将在各个部分详细说明。
> (文档更新日期2015-04)

## 源代码下载

源代码下载部分的主要链接为UCSC关于下载页面的说明部分： [UCSC Source Downloads](http://hgdownload.soe.ucsc.edu/downloads.html#source_downloads)。需要说明的是，除了Genome Browser，UCSC还提供了一些关于还有其他生物信息学工具包（command line bioinformatic utilities）。这些包的安装和配置方式列在了上面的下载链接之中。这些工具包的安装过程可以参考[userApps installation README](http://genome-source.cse.ucsc.edu/gitweb/?p=kent.git;a=blob;f=src/userApps/README)。

对于Genome Browser，我们可以下载其完整的源代码包。这里我们使用git从UCSC网站的代码仓库里面直接下载：
```shell
git clone git://genome-source.cse.ucsc.edu/kent.git
```
Genome Browser的版本代码为`kent`。当上面的命令将从UCSC中下载完整的源代码包到当地当前文件夹下的`./kent`中（关于git工具的使用及工作原理情自行搜索）。就可以参照其中的说明文件进行下一步的安装操作。
下面是一些关于下载源代码以及更新源代码的相关命令：

- 获取Beta分支 
```shell
cd kent
git checkout -t -b beta origin/beta
```
- 获取最新版本的UCSC
```shell
git pull
```
- 网页版代码查看
[http://genome-source.cse.ucsc.edu/gitweb/](http://genome-source.cse.ucsc.edu/gitweb/)

更加详细或者最新的说明可以参考UCSC [Git](http://genome.ucsc.edu/admin/git.html)页面。

## 服务器版本及相关工具

要正常安装配置Genome Browser，必须保证系统中有以下的工具存在：`rsync`，`git`， `mysql`。同时需要配置好`Apache`服务器以及`MySql`数据库。不同的系统，配置方式有些细微的不同。这里我们使用的操作系统是CentOS6.5。我们将在这样的机器上完成Genome Browser的配置。在这个例子中我们使用的是CenOS系统：
```shell
cat /etc/centos-release
```
> CentOS release 6.6 (Final)

## Apache服务器的配置
Apache服务器的安装在不同的系统上是不同的，网上有非常多的详细安装配置教程可以参考。
### 第一步：安装Apache服务器
```shell
sudo yum update
sudo yum install httpd
```
安装过程中需要确认的地方按`y`继续即可。

### 第二步：配置Apache服务器
CentOS中Apache的配置文件包含在`/etc/httpd/conf/httpd.conf`文件中。一个推荐的好习惯是安装完之后马上对该文件进行备份，备份到自己的根目录下：
```shell
sudo cp /etc/httpd/conf/httpd.conf ~/httpd.conf.backup
```
一般在`/etc/httpd/conf.d/`目录下的所有以`.conf`结尾的文件都被认为是Apache的配置文件。推荐将自己定义的配置文件都放在改文件夹下面。注意重要的配置文件一定要**备份**，**备份**再**备份**

编辑`/etc/httpd/conf/httpd.conf`文件，找到
```shell
#ServerName www.example.com:80
```
在这一行的后面加上一行，配置服务器名称：
```shell
ServerName localhost
```

### 第三步：重启httpd服务
每次做完配置文件的修改，我们都需要重新启动httpd服务：
```shell
sudo service httpd restart
```
Centos7
```shell
sudo systemctl start httpd.service
```
### 第四步：重启自动启动
```shell
sudo chkconfig httpd on
```
Centos7
```shell
sudo systemctl enable httpd
```
> **注意**：
> 
> 在有的版本的CentOS上80端口是默认关闭的，需要将防火墙打开。
> ```shell
> sudo iptables -I INPUT -p tcp --dport 80 -j ACCEPT
> ```
> 如果问题解决记得要保存设置
> ```shell
> sudo service iptables save
> ```

> **Centos 7**
> ```shell
> firewall-cmd --add-service=http --permanent
> ```
> reload rules:
> ```shell
> firewall-cmd --reload
> ```

### 第四步：服务器设置
```shell
XBitHack on
<Directory /usr/local/apache/htdocs>
    Options +Includes
</Directory>
```

## 安装并配置MySql数据库

https://devops.profitbricks.com/tutorials/install-mysql-on-centos-7/

```shell
chcon -R -t mysqld_db_t /home/user/data


setenvforce 0
```
### 第一步： 安装并启动Mysql
```shell
sudo yum install mysql-server
sudo /sbin/service mysqld start
```
### 第二步：配置Mysql
```shell
sudo /usr/bin/mysql_secure_installation
```
### 第三步：配置重启自动启动
```shell
sudo chkconfig mysqld on
```

## 配置Genome Browser安装环境
USCS官方网站上给出了使用他们自己的脚本配置安装Genome Browser的[说明文档](http://genome-source.cse.ucsc.edu/gitweb/?p=kent.git;a=blob_plain;f=src/product/scripts/README;hb=HEAD)。我们讲按照文档的说明一步一步进行操作。

### 第一步： 拷贝安装脚本
我们从之前使用git下载下来的`kent`源代码文件夹中将`./kent/src/product/scripts`文件夹拷贝一份到常用的文件夹下面，这里我们拷贝到`~/GenomeBrowser/`下面：
```shell
cd kent
cp -r src/product/scripts/ ~/GenomeBrowser/
```
需要将这个文件夹重新拷贝一份的目的是为了，在以后的更新过程中，保证我们使用的这一份`scripts`不变。

### 第二步：编辑`browserEnvironment.txt`文件
这个文件位于`scripts`文件夹中，我们需要在新拷贝的文件夹下编辑并按照我们自己系统的情况来设置这个文件。这个文件包含了所有关于Genome Browser安装的环境变量，有了这些，脚本就知道该如何进行自动安装配置。
有以下几个关键变量我们需要注意：
1.  `KENTHOME`变量
```
#   KENTHOME - directory where you want the kent source tree and built
#   binaries to exist.  This is typically your $HOME/ directory
export KENTHOME="$HOME"
```
这个变量指定了`kent`源代码将要下载和编译安装的位置。默认位于`~/kent`下面。可以按照自己的需要修改成期望安装到的文件夹下。

2. `BROWSERHOME` 变量
``` 
# BROWSERHOME - directory where cgi-bin/ trash/ and htdocs/ should exist
#   This is typically something like /var/www or /usr/local/apache
export BROWSERHOME="/usr/local/apache"
```
这个变量指定了apache服务器的根目录（也就是`/etc/httpd/conf/httpd.conf`文件中`DocumentRoot`的指定目录的上一层目录。如果你配置Apache服务器的时候使用了`Virtual Host`，那么这个目录就应该是VirtualHost中指定的`DocumentRoot`）。

3. `MYSQLLIBS`变量
```
# You will need to find out where your MySQL libs are.
#   see notes in: README.building.source about finding them
export MYSQLLIBS="/usr/lib64/mysql/libmysqlclient.a -lz"
export MYSQLINC="/usr/include/mysql"
```
CentOS中一些版本不提供`libmysqlclient.a`文件，如果实在找不到相应的静态库，我们可以使用动态库链接来代替。
```shell
cd /usr/lib64/mysql/
ln -s libmysqlclient.so.16.0.0 libmysqlclient.a
```

4. `AUTH_MACHINE`变量
```
# protect these scripts by specifying the single machine on which they
#   should run and the single user name which should be running them.
# for the AUTH_MACHINE you may need to see what uname -n says
export AUTH_MACHINE="yourMachineNamie"
export AUTH_USER="yourUserName"
```
这些变量指定了服务器的机器名称以及维护机器的人的名称。

```sql
GRANT SELECT, INSERT, UPDATE, DELETE, FILE, CREATE, DROP, ALTER, CREATE TEMPORARY TABLES on ${DB}.* TO browser@localhost IDENTIFIED BY 'genome';
```
修改`ex.MySQLUserPerms.sh`创建用户

http://tecadmin.net/change-default-mysql-data-directory-in-linux/

```shell
libmysqlclient.a -ldl
```


Please follow the under mentioned instructions to turn off the MySQL strict mode. Make the following changes in the "my.ini/my.cnf":

    1.  Look for the following line:
        sql-mode = "STRICT_TRANS_TABLES,NO_AUTO_CREATE_USER,NO_ENGINE_SUBSTITUTION"

    2.  Change it to:
        sql-mode="" (Blank)

    3. Restart the MySQL service.
## hg.conf


