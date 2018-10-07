# **apache+modwsgi部署Django项目**

### **1、无root权限安装Python**

​	参考：https://blog.csdn.net/salonhuang/article/details/70049544

参考：<https://blog.csdn.net/salonhuang/article/details/70049544>

```shell
    cd /mnt/project/python3.5/bin/
    tar -zxvf
    ./configure --enable-shared \
    --prefix=/mnt/project/python3.5
    make -j 4 && make install
```

​	安装完后会报错

```bash
    ./python3.5: error while loading shared libraries: libpython3.5m.so.1.0: cannot open shared 	object file: No such file or directory
```

​	使用以下命令

 ```shell
	export LD_LIBRARY_PATH=/mnt/project/python3.5/lib:$LD_LIBRARY_PATH
 ```



### **2、无root权限安装Apache**

​	(1)参考：https://blog.csdn.net/jaye16/article/details/78506547?locationNum=3&fps=1

​	apr-util安装可能会提示缺少expat不能解析xml的错误，需要安装expat，然后在./configure 时加入--with-expat="PATH_expat"。

​	apache安装可能会提示缺少pcre，需要安装pcre-devel，然后在./configure 时加入--with-pcre2="PATH_pcre2"。

​	(2)安装步骤：

​	   1)下载文件

| &emsp;  &emsp; apache: https://httpd.apache.org/             |
| :----------------------------------------------------------- |
| &emsp;  &emsp; expat: https://sourceforge.net/projects/expat/files/?source=navbar.tar.bz2 |
| &emsp;  &emsp; apr和apr util: https://apr.apache.org/download.cgi |
| &emsp;  &emsp; openssl: https://www.openssl.org/source/      |
| &emsp;  &emsp; pcre: https://ftp.pcre.org/pub/pcre/          |

​	  2)解压安装

```shell
解压安装openssl:	./config -fPIC --prefix=" /mnt/project/ssl" && make && make install
解压安装apr:	./configure --prefix=" /mnt/project/apr" && make && make install
解压安装expat:	./configure --prefix="/mnt/project/expat" && make && make install
解压安装apr-util:	./configure --prefix="/mnt/project/apr-util" 
                              --with-apr="/mnt/project/apr" 
                              --with-expat="/mnt/project/expat" && make && make install
解压安装pcre:	./configure --prefix=/mnt/project/pcre2" && make && make install
```

​	  3)解压安装apache 

```shell
    ./configure \
    --prefix="/mnt/project/apache" \
    --with-apr="/mnt/project/apr" \
    --with-apr-util="/mnt/project/apr-util" \
    --with-ssl="/mnt/project/ssl" \
    --with-pcre2="/mnt/project/software/pcre2" \
    && make && make install

```



### **3、mod_wsgi的安装和配置**

​	参考：<https://www.linuxidc.com/Linux/2017-03/142211.htm>

​	先下载解压mod_wsgi，安装mod_wsgi

```shell
    ./configure \
    --with-apxs=/mnt/project/apache/bin/apxs \
    --with-python=/mnt/project/python3.5/bin/python3 \
    --prefix=/mnt/project/mod_wsgi && make && make install

    chmod 755 /mnt/project/apache/modules/mod_wsgi.so

```



### **4、Django项目**

​	(1)安装django：

```shell
	pip install django
```

​	(2)克隆Django项目：

```shell
    git clone https://github.com/boogalooYison/django_project.git
```

​	(3)为项目新建python虚拟环境：

```shell
	pip install virtualenv

	virtualenv -p /usr/bin/python3 envname
```

​	(4)用pip给虚拟环境安装相关的Python包：

```shell
    cd envname/bin
    source activate
    pip install -r requirement.txt （requirement.txt为在项目里）

    退出时： deactivate
```



### **5、修改Apache的配置文件httpd.conf**

​	(1)使用项目内的 httpd.conf 覆盖原始的apache配置文件

​	(2)httpd.conf里需修改的

​	  1）SERVER_ROOT,django_ROOTV , django_ENV 和django_ENV_NAME 

​	  2）修改 Listen（监听端口）, User/Group, ServerAdmin；ServerName前面的#号去掉，改为主机名称

​	  3）有时网页可视化的处理时间会较长导致 504GatewayTimeout，所以增加apache的等待时间

​	        取消httpd.conf文件里 Include conf/extra/httpd-default.conf 的注释在…../conf/extra/httpd-default.conf文件里修改Timeout的时间



### **6、网站部署**

​	(1)数据库配置

​	  1)初始化数据库

```shell
    python manage.py makemigrations
    python manage.py migrate
```

​	  2)设置admin

```shell
	python manage.py createsuperuser
```

​	  3)运行基因数据载入脚本

```shell
	python help_script/import_genereg.py
```

​	(2)设置配置文件(settings)的相关内容:DB_PATH ,software_FILE,TOOLS等

​	(3)补充数据库文件

​	    在db文件夹下,建立数据库文件的软连接



### **7、设置定时任务（如crontab）**

​	每日清理 TMP_PATH 下的文件

```shell
    1   0   *   *   *   rm -f /mnt/project/work/django_name/tmp/* \
    && touch /mnt/project/work/django_name/tmp/DAILY_DELETE_FILES_IN_THIS_PATH
```



### **8、启动Apache服务**

​	./apachectl start