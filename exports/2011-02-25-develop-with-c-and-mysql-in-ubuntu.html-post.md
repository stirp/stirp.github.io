### 一. 开发环境的准备
#####1. 首先要安装好Linux，这里用的是Ubuntu 10.10
#####2. 安装MySQL系列软件包一、开发环境的准备
#####3. 首先要安装好Linux，这里用的是Ubuntu 10.10
#####4. 安装MySQL系列软件包，这边10.10系统仓库里面的是MySQL5.1版本：  

``` 
sudo apt-get install mysql-server
sudo apt-get install mysql-client
sudo apt-get install libmysqlclient15-dev
```
### 二. 检查MySQL服务的状态
##### 1. 查看当前的mysql服务状态
``sudo /etc/init.d/mysql status``
##### 2. 也可以用以下检查mysql服务是否有启动，如果结果为空，则没有启动
`sudo netstat -tap | grep mysql `或`ps -ef | grep mysql`
###三. 启动/停止/重启MySQL服务
#####1. 启动：`sudo /etc/init.d/mysql start`
#####2. 停止：`sudo /etc/init.d/mysql stop`
#####3. 重启：`sudo /etc/init.d/mysql restart`
###四. 在命令行使用MySQL客户端访问数据库
#####1. 访问本地主机：
``mysql -uuser -ppassword db_name``
#####2. 访问远程主机：
``mysql -hhost -uuser -ppassword db_name``
备注：由于默认的配置是只能从本机访问，只要注释掉/etc/mysql/my.cnf里面的bind-address这行，就可以让远程主机访问了。
#####3、执行管理操作：
使用mysqladmin及相关参数
#####4、备份 db_name 数据库：
``mysqldump -uroot -p --default-character-set=utf8 --opt     --extended-insert=false --triggers -R --hex-blob -x db_name > bak.sql``
#####5、恢复db_name数据库：
`mysql -uroot -p db_name < bak.sql`
#####6、备份tbl_name 数据表：
``select * into outfile '/usr/local/mysql/f.txt' fields terminated by '|' from tbl_name;``
#####7、把文件/home/a.txt导入数据库中的 tbl_name表
``mysql> load data local infile '/home/a.txt'  into table tbl_name fields terminated by ',' lines terminated by '\r\n';``
#####8、设置mysql数据库root的初始密码

```
shell> mysql -u root
mysql> SET PASSWORD FOR ''@'localhost' = PASSWORD('newpwd');
mysql> SET PASSWORD FOR ''@'host_name' = PASSWORD('newpwd');
```
#####9、为普通用户修改密码：
``mysql> SET PASSWORD FOR 'user_name'@'host_name' = PASSWORD('newpwd');``
#####10、建立超级用户账户，具有完全的权限可以做任何事情：

```
mysql> GRANT ALL PRIVILEGES ON *.* TO 'monty'@'localhost'
->     IDENTIFIED BY 'some_pass' WITH GRANT OPTION;
mysql> GRANT ALL PRIVILEGES ON *.* TO 'monty'@'%'
->     IDENTIFIED BY 'some_pass' WITH GRANT OPTION;
```
备注：第一句建立用于本机连接的帐户，第二句建立用于从其他主机连接的帐户。
#####11、建立帐户custom，可以访问bankaccount数据库，但只能从本机访问
```
mysql> GRANT SELECT,INSERT,UPDATE,DELETE,CREATE,DROP
->     ON bankaccount.*
->     TO 'custom'@'localhost'
->     IDENTIFIED BY 'obscure';
```
###五、Linux下C开发MySQL数据库应用程序的简要流程
编译：`gcc -o bin_name hello.c -I /usr/include/mysql -L /usr/lib/mysql -lmysqlclient -lz -lm`
#####1、定义MYSQL变量
``MYSQL mysql;``
#####2、初始化MYSQL变量
``mysql_init(&mysql);``
#####3、连接到MySQL数据库
``mysql_real_connect(&mysql,”hostname”,”username”,”password”,”database”,0,NULL,0);``
#####4、指定本连接的字符集（可选，www.linuxidc.com主要为了确认客户端与服务器之间使用一致的字符集进行通信）
``mysql_query( &mysql , "SET NAMES 'utf8'");``
#####5、执行SQL语句
``mysql_real_query( &mysql, SQL, SQL_LEN ) ;``
#####6、获取查询结果

~~~~{c}
MYSQL_RES *result;
result = mysql_store_result( &mysql ) ;
row_count = ( int )mysql_num_rows( result ) ;
field_count = ( int )mysql_num_fields( result ) ;
~~~~

#####7、循环调用mysql_fetch_row ，以便获取查询结果的每一行

~~~~{c}
MYSQL_ROW row;
row = mysql_fetch_row( result ) ;//下一次再调用便自动指向result的下一行
printf(“%s\n”,row[index]);//注意转换各个字段的类型
~~~~

#####8、释放资源

~~~~{c}
mysql_free_result( result ) ;
~~~~

#####9、关闭MYSQL连接

~~~~{c}
mysql_close(&mysql);
~~~~

###六、一个简单的例子

~~~~{c}
//打印出本地主机的pim数据库里面的classmate数据表的第一、第二列。用户名kitty，密码a1。
//编译：gcc -o bin_name hello.c -I /usr/include/mysql -L /usr/lib/mysql -lmysqlclient -lz -lm

#include <stdio.h>
#include <string.h>
#include <stdlib.h>
#include <mysql.h>

int main( int argc , char* argv[] )
{
	MYSQL mysql;
	//连接mysql
	if(connect_mysql(&mysql))
	{
	return 1;
	}
	//载入数据
	if( load_classmate(&mysql) )
	{
	mysql_close(&mysql);
	return 1;
	}
	//Do something here.
	//关闭mysql
	mysql_close(&mysql);
	return 0;
	}
	//连接MYSQL数据库
	int connect_mysql(MYSQL* mysql)
	{
		printf( "Initializing mysql................." );
		if ( !mysql_init(mysql) )
		{
			return 1;
		}
		printf( "Done\r\n" );
		printf( "Connectiong to mysql..............." );
		if (!mysql_real_connect(mysql,"localhost","kitty","a1","pim",0,NULL,0))
		{
			fprintf(stderr, "Error: %s\r\n", mysql_error(mysql));
			return 1;
		}
		//设置mysql连接的字符集
		mysql_query( mysql, "SET NAMES 'utf8'" );
		printf( "Done\r\n" );
		return 0;
	}
	//加载数据表
	int load_classmate(MYSQL *mysql)
	{
		int ret,field_count,row_count,i;
		int *lengths;
		char *query="select * from classmate";
		MYSQL_RES *result;
		MYSQL_ROW row;
		ret = mysql_real_query( mysql, query, strlen(query) ) ;
		if(ret != 0)
		{
		printf("加载不了classmate的数据。\n");
		return 1;
	}
	result = mysql_store_result( mysql ) ;
	row_count = ( int )mysql_num_rows( result ) ;
	field_count = ( int )mysql_num_fields( result ) ;
	if( result == NULL && field_count == 0 )
	{
		printf("查询不到数据。");
		return 1;
	}
	for( i = 0 ; i < row_count ; i++ )
	{
		row = mysql_fetch_row( result ) ;
		printf( "%2d | %8s \r\n", atoi( row[ 0 ] ), row [ 1 ] ) ;
	}
	mysql_free_result( result ) ;
	return 0;
}
~~~~

###七、MySQL数据库中文乱码的解决办法
MySQL的字符集和校对规则有4个级别的默认设置：服务器级、数据库级、表级和连接级。我们解决中文乱码的方法就顺着这几个级别来分别设置。比较常见的是由于没有设置连接级的字符集导致的乱码。
#####1、服务器级字符集
服务器级也就是当服务器启动时根据配置文件中的字符集来加载。当前的服务器字符集和校对规则可以用作character_set_server和collation_server系统变量的值。在运行时能够改变这些变量的值。针对服务器级，我们有两个方法可以解决。
######a、重新编译MySQL
更改设定值的一个方法是通过重新编译。如果希望在从源程序构建时更改默认服务器字符集和校对规则，使用：--with-charset和--with-collation作为configure的参量。例如：
``shell> ./configure --with-charset=utf8``
######b、不重新编译，只修改当前配置文件
打开/etc/mysql/my.cnf修改编码：
在[mysqld]下添加：`default-character-set=utf8`
在[client]下添加：`default-character-set=utf8`
在[mysql]下添加：`default-character-set=utf8`
然后重启mysql应该就可以了！
#####2、数据库级字符集
默认数据库的字符集和校对规则可以用作character_set_database和 collation_database系统变量。无论何时默认数据库更改了，服务器都设置这两个变量的值。如果没有 默认数据库，这两个变量与相应的服务器级别的变量（character_set_server和collation_server）具有相同的值。
######a、重新建数据库，并指定默认字符集
数据库级是在新建数据库时指定该数据库默认的字符集。
``mysql>create database db_name character set charset_name ;``

比如：
``mysql>create database db_name character set utf8 ;``
######b、不重新建数据库，通过alter database来修改数据库的默认字符集
如果之前在建表的时候没有指定character set charset_name，也可以通过以下重新指定：     `mysql>alter database db_name character set charset_name ;`

比如：
``mysql>alter database db_name character set utf8 ;``
#####3、表级字符集
如果在表定义中没有指定表字符集和校对规则，则默认使用数据库字符集和校对规则。
######a、重新建表，并指定默认字符集
表级字符集是在建立表的时候指定该表的默认字符集。
``mysql>create table tbl_name (column_list) default character set charset_name ;``
######b、不重新建表，通过alter table来修改表的默认字符集
如果之前在建表的时候没有指定character set charset_name，也可以通过以下重新指定：     
``mysql>alter table tbl_name default character set charset_name ;``
备注：列级别的字符集和表级别字符集的规则类似，也可以在建表的语句里面指定，若没有指定则默认使用表级别的字符集，若表也没有指定默认字符集，则往上推。 
#####4、连接级字符集
从以下几个方面来理解连接级字符集：（类似一种网络协议，服务器、客户端之间预先设置的通信规则）
######a、当查询离开客户端后，在查询中使用哪种字符集？
服务器使用character_set_client变量作为客户端发送的查询中使用的字符集。
######b、服务器接收到查询后应该转换为哪种字符集？
转换时，服务器使用character_set_connection和collation_connection系统变量。它将客户端发送的查询从character_set_client系统变量转换到character_set_connection（除非字符串文字具有象_latin1或_utf8的引介词）。collation_connection对比较文字字符串是重要的。对于列值的字符串比较，它不重要，因为列具有更高的校对规则优先级。
######c、服务器发送结果集或返回错误信息到客户端之前应该转换为哪种字符集？
character_set_results变量指示服务器返回查询结果到客户端使用的字符集。包括结果数据，例如列值和结果元数据（如列名）。
我们能够调整这些变量的设置，有两个语句影响连接字符集：
``SET NAMES 'charset_name'``
``SET CHARACTER SET charset_name``

SET NAMES显示客户端发送的SQL语句中使用什么字符集。因此，SET NAMES 'cp1251'语句告诉服务器“将来从这个客户端传来的信息采用字符集cp1251”。它还为服务器发送回客户端的结果指定了字符集。（例如，如果你使用一个SELECT语句，它表示列值使用了什么字符集。）
SET NAMES 'x'语句与这三个语句等价：

```
mysql> SET character_set_client = x;
mysql> SET character_set_results = x;
mysql> SET character_set_connection = x;
```
要设置连接级的字符集，可以在编写MySQL数据库应用程序时，在客户端程序发
起连接到服务器后，执行一条语句，如下情形：

~~~~{c}
mysql_real_connect(&mysql,”hostname”,”username”,”password”,”database”,0,NULL,0);
mysql_query( &mysql , "SET NAMES 'utf8'");
~~~~
这样便可指定本次连接使用的默认字符集为utf8了。
###八、一些MYSQL常用的命令
#####1.列出MYSQL支持的所有字符集： 
``SHOW CHARACTER SET;``
#####2.当前MYSQL服务器字符集设置 
``SHOW VARIABLES LIKE 'character_set_%';``
#####3.当前MYSQL服务器字符集校验设置 
``SHOW VARIABLES LIKE 'collation_%';``
#####4.显示某数据库字符集设置 
``show create database 数据库名;``
#####5.显示某数据表字符集设置 
``show create table 表名;``
#####6.修改数据库字符集 
``alter database 数据库名 default character set 'utf8';``
#####7.修改数据表字符集 
``alter table 表名 default character set 'utf8';``
#####8.建库时指定字符集 
``create database 数据库名 character set gbk collate gbk_chinese_ci;``
#####9.建表时指定字符集 
``CREATE TABLE `mysqlcode` ( `id` TINYINT( 255 ) UNSIGNED NOT NULL AUTO_INCREMENT PRIMARY KEY , `content` VARCHAR( 255 ) NOT NULL ) TYPE = MYISAM CHARACTER SET gbk COLLATE gbk_chinese_ci;``
###九、参考
MySQL 5.1 官方简体中文版参考手册
