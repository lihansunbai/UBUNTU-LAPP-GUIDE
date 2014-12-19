A guide of install LAPP in Ubuntu14.04
手动安装
一、	利用apt-get
apt-get install postgresql-9.3
安装PostgreSQL9.3
PostgreSQL 数据默认会创建一个postgres的帐号用户作为数据库的管理员，密码是随机的，所以：
首先，我们需要重置“postgres”用户的密码。
命令行如下：
sudo -u postgres psql(或者sudo su postgres -c psql)
—>运行psql
postgres=# ALTER USER postgres WITH PASSWORD ‘postgres’;
—>修改postgres的密码为postgres
postgres=# \q—>退出
修改了数据库中的密码之后，我们还需要修改linux下的用户 “postgres”的密码：设定成与数据库中postgres帐号相同的密码，即postgres。
sudo passwd -d postgres———>删除密码
sudo -u postgres passwd(或者 sudo su postgres -c passwd)—–>创建密码
然后输入跟之前一样的密码。*/
现在，我们就可以在数据库服务器上用 postgres帐号通过psql或者pgAdmin等等客户端操作数据库了。
使用命令行创建用户和数据库：
sudo -u postgres createuser -D -P mynewuser—->-D该用户没有创建数据库的权利，-P提示输入密码，后面的选项都可以省略，命令执行的时候会提示用户选择yes或者no
sudo -u postgres createdb -O mynewuser mydatabase—>-O设定所有者为mynewuser
或者使用psql创建用户和数据库：
首先利用psql登录postgresql服务器:sudo -u postgres psql[ -U postgres -h 127.0.0.1]—->运行psql,后面[]中的内容可选，用于登录服务器，默认会登录本机；
然后在psql程序中创建用户和数据库:用户名和数据库名称加上引号后才区分大小写，否则会自动转换成小写:
create user “mynewuser” with password ‘mynewuser nocreatedb;
create database “mydatabase” with owner=mynewuser;

apt-get install apache2
安装apache
apt-get install libapache2-mod-php5 php5
安装php
apt-get install php5-gd php5-pgsql
安装PostgreSQL对于php的扩展程序

sudo gedit /etc/apache2/apache2.conf
在打开的文件中加上
AddType application/x-httpd-php .php .htm .html即可。
二、	纯源码安装
首先分析一下整个安装的思路，我们需要的是三个主要的软件
1.	PostgreSQL 对于PostgreSQL，可以单独作为一个数据库在本地运行。
2.	Apache 对于Apache，作为服务器端，需要合理配置。
3.	PHP 对于PHP，需要和Apache和PostgreSQL连接合理使用
所以我们可以初步拟定安装方案：
1.	安装PostgreSQL
2.	编译、安装Apache
3.	编译、安装PHP

安装PostgreSQL
理论上可以在一个系统中安装多个数据库，比如MySQL、SQLite、PostgreSQL共存。但是，这样风险较大，容易导致某一个数据库不能正常使用。而且这样的共存还需要特定的安装顺序，比如，MySQL和PostgreSQL共存，就只能先装PostgreSQL在装MySQL，否则PostgreSQL就不能被PHP识别。
首先下载PostgreSQL的源码，这里使用的是9.3.5。然后在默认./configure情况下编译，make,make install。如果说在编译p的时候需不需要设置什么特别的设置，在这里我认为一般的配置应用来说没有太多的特别设置。编译PostgreSQL的时间比较长，不过还有更长的是p的另一个管理软件pgadmin3。
安装成功后我们需要进行用户设置和初始化数据库。由于PostgreSQL不允许在root环境下对数据库进行操作，所以我们可以新建一个用户对数据库进行操作管理。具体方法可以参照之前的创建方式。
现在要修改系统环境变量，我们选择修改/etc/profile或者/etc/environment这两个文件中的一个来指定一个全局的系统环境变量。
这里个人推荐修改/etc/environment，因为这样可以保证所有用户，包括使用sudo命令进行操作时所有的环境变量都能识别。
根据官方文档给出的说明，在环境变量中添加：
LD_LIBRARY_PATH=/usr/local/pgsql/lib
export LD_LIBRARY_PATH
这样可以避免出现以下问题：
psql: error in loading shared libraries
libpq.so.2.1: cannot open shared object file: No such file or directory
再添加$PATH:
PATH=/usr/local/pgsql/bin:$PATH
export PATH
MANPATH=/usr/local/pgsql/man:$MANPATH
export MANPATH
现在新建用户postgres然后设置密码。并且使用用户登录，使用initdb 初始化数据库。这里我们可以创建一个test数据库来测试一下。
安装Apache2
这里同样采用编译源代码的方式来安装Apache。之前你要先确定你已经安装了APR 、APR-Util和Perl-Compatible Regular Expressions Library (PCRE)。如果没有安装请自行安装这个不在本文的讨论范围之内。
首先下载源码。开始编译
./configure --with-apr=DIR|FILE --with-apr-util=DIR|FILE --enable-module=so  ;
Make;
Make install;
安装完成之后转移到安装目录中，找到/conf/httpd.conf文件，编辑文件中的
AddType application/x-httpd-php .php .phtml .php3 .inc  
AddType application/x-httpd-php-source .phps  
这一步是修改Apache的配置文件支持PHP5.

安装PHP
同样下载源码，开始编译：
./configure--with-pgsql[=DIR]--with-pdo-pgsql[=DIR]  --without-pdo-sqlite[=DIR]
最好取消sqlite的支持，这样可以保证数据库连接的稳定。我多次尝试过让他们共存但是没有成功所以只留下PostgreSQL就可以了。
Make；
Make install；
在编译的文件中找到php.ini-dist，编辑文件中，Pgsql有关的extension，去掉之前的分号，然后把文件复制到/usr/local/lib下，重命名为php.ini。这个操作是打开了PHP对PostgreSQL的扩展支持。然后重启Apache。
这样就完成了整个安装过程。
现在写一个PHP，在其中调用phpinfo（）；
如果在PDO中看到pgsql，并且其中有PostgreSQL的信息就说明安装成功。

完全自动安装
 bitnami LAPP 安装包。
 https://bitnami.com/stack/lamp      LAMP
 https://bitnami.com/stack/lapp      LAPP
利用这个安装包一键解决所有问题，直接配置为可以使用的状态。
感谢Google为我找到解决问题的简单粗暴的方法。
