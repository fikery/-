将centos7+python3.6+django2.0+nginx+uwsgi+git组合的服务器架设Django站点总结如下。

费了好大劲，查过了几十篇博客问答，才将这个流程完整的打通，当然因为之前完全没有接触过这方面，所以也学到了很多非常基础的上线建站的新知识。

### 一.安装python3.5

由于centos7本身安装了Python2.7，而且不少系统命令都会用到，所以采用另外安装python3的方式。

1.首先安装依赖包
>yum -y groupinstall "Development tools"

>yum -y install zlib-devel bzip2-devel openssl-devel ncurses-devel sqlite-devel readline-devel tk-devel gdbm-devel db4-devel libpcap-devel xz-devel

2.下载python3
>wget https://www.python.org/ftp/python/3.6.5/Python-3.6.5.tar.xz

3.解压并编译安装
>tar -xvJf  Python-3.6.5.tar.xz

>cd Python-3.6.5

>./configure --prefix=/usr/local/python3

>make && make install

4.创建软链接
>ln -s /usr/local/python3/bin/python3 /usr/bin/python3

>ln -s /usr/local/python3/bin/pip3 /usr/bin/pip3

5.检查版本
>python -V

>python3 -V

以上便安装好了python3.5

### 二.安装Django2.0+virtualenv+virtualenvwrapper

1.由于python3.5内置了pip3，所以可以直接使用pip3
>pip3 install django

2.测试后发现pip3安装的django并不能直接使用django-admin，提示找不到，所以要使用软链接将django-admin转移一下。我们先找到django-admin安装的位置，再创建软链接，这样就可以直接使用django-admin了
>find / -name django-admin

     /usr/local/python3/bin/django-admin

>ln -s /usr/local/python3/bin/django-admin /usr/bin/django-admin

3.安装虚拟环境，经测试这个时候virtualenv除了像上面django-admin的软链接方式，还可以通过pip直接安装，这样会默认创建python2的环境，不过好在可以指定创建的python版本，得到python3的虚拟环境。
>pip install virtualenv

4.正常使用pip3安装强化版wrapper
>pip3 install virtualenvwrapper

5.查看.sh的位置
>which virtualenvwrapper.sh

     /usr/bin/virtualenvwrapper.sh

6.创建virtualenv文件夹
>mkdir /root/virtualenv

7.配置.bashrc
>vi ~/.bashrc

8.添加下面三行内容到文件末尾
>export WORKON_HOME=/root/virtualenv

>export VIRTUALENVWRAPPER_PYTHON=/usr/bin/python3

>. /usr/bin/virtualenvwrapper.sh

9.生效.bashrc
>. ~/.bashrc

10.使用virtualenv创建虚拟环境venv，注意-p指定python版本
>mkvirtualenv -p /usr/bin/python3 venv

>workon venv #进入虚拟环境

>deactivate #退出虚拟环境

>rmvirtualenv venv #删除虚拟环境

>workon #列出所有虚拟环境

11.进入虚拟环境，创建django项目
>workon venv

>django-admin startproject mblog

>cd mblog

>python manage.py startapp mainsite

12.修改settings.py并测试页面
     
     ALLOWED_HOSTS=['*']
     LANGUAGE_CODE='zh-Hans'
     TIME_ZONE='Asia/Shanghai'

>python manage.py runserver 0.0.0.0:8000

以上便是配置好了虚拟环境下的django项目。由于创建虚拟环境采用的是python3，所以在这个venv创建的django项目下，可以直接使用python，pip等命令，指向的是python3的命令。同时在centos7上不能直接runserver，必须是0.0.0.0:8000，外网才能访问成功ip:8000。当然前提是8000端口开放了，不过腾讯云服务器centos7系统默认是没有开放端口的，所以接下来要进行防火墙和端口的开放。

### 三.配置防火墙和端口

1.centos7采用的是firewall而不是iptables
>systemctl status firewalld #查看防火墙状态

>systemctl start firewalld #启动

>systemctl disable firewalld #停用

>systemctl stop firewalld #禁用

2.启动防火墙，查看开放端口
>firewall-cmd --list-ports

3.开启/关闭80端口
>firewall-cmd --zone=public --add-port=80/tcp --permanent

>firewall-cmd --zone=public --remove-port=80/tcp --permanent

4.重启防火墙
>firewall-cmd --reload

以上便是开启了80端口，外网可以访问 http://ip 了，对于我们的django项目来说，自然是还要开启8000端口。为了测试方便，我在这里还开启了8001端口。对于有些情况下，8000端口被占用了，可以杀死占用8000端口的进程，重新使用端口
>fuser -k 8000/tcp

### 四.配置uwsgi+nginx
1.使用pip3直接安装
>pip3 install uwsgi

2.为了在终端中使用uwsgi命令，执行以下命令
>ln -s /usr/local/python3/bin/uwsgi /usr/bin/uwsgi3

3.建一个demo.py , 输入这段代码
    
    def application(env, start_response):
        start_response(‘200 OK’, [(‘Content-Type’,’text/html’)]) 
    return [b”Hello World”]

4.执行命令
>uwsgi --http :8000 --wsgi-file demo.py #虚拟环境下直接uwsgi即可

5.浏览器输入http:// IP:8000 就可以看到 hello world 了

为了后面nginx配置，现在需要找到uwsgi_params的路径，输入命令find / -name uwsgi_params即可找到，我的路径是/usr/local/nginx/conf/uwsgi_params。

6.安装nginx，根据一个教程来安装的，没有采用yum方式
>wget http://nginx.org/download/nginx-1.13.7.tar.gz

>tar -zxvf nginx-1.13.7.tar.gz

>./configure #如果需要配置ssl登录需要加上--with-http_ssl_module

>make && make install

7.nginx一般默认安装好的路径为/usr/local/nginx，在/usr/local/nginx/conf/中打开nginx.conf，加入以下内容

    server {
    listen 80; #暴露给外部访问的端口
    server_name localhost;
        charset utf-8;
    access_log /root/project/hello/access_log;
    error_log  /root/project/hello/error_log;
    client_max_body_size 75M;

    location / {
        uwsgi_pass 127.0.0.1:8000; #外部访问80端口转发到8000端口
        include    /usr/local/nginx/conf/uwsgi_params;
    }
    location /static {
        alias /root/mblog/static/; #项目静态路径设置
    }
    }

根据静态文件的用户组，还需要修改配置文件
     user root;

8.wq保存后进入/usr/local/nginx/sbin/目录，执行./nginx -t命令先检查配置文件是否有错，没有错就执行./nginx。如果这个时候报错80端口被占用了，可以先执行 fuser -k 80/tcp，再执行./nginx

9.接下来继续配置uwsgi。新建一个hello_uwsgi.ini文件

    [uwsgi]
    socket = 127.0.0.1:8000
    chdir= /root/mblog
    module=mblog.wsgi
    master = true
    processes=2
    threads=2
    max-requests=2000
    chmod-socket=664
    vacuum=true
    daemonize = /root/mblog/uwsgi.log #设置uwsgi后台运行

>uwsgi --ini uwsgi.ini #启动uwsgi

以上配置即可在centos7系统的云服务器上部署django站点，通过外网访问ip即可对应django的8000端口站点。值得注意的是，此时上线django的管理界面是样式错乱的，所以还需要进一步处理。

### 五.上线样式处理
1.django管理后台界面样式错乱，此时需要在settings.py中指定STATIC_ROOT路径，例如 

     STATIC_ROOT='/root/mblog/static'

这里的路径和nginx.conf中对location /static的配置是对应的。然后执行
>python manage.py collectstatic

将admin的静态文件转移到STATIC_ROOT目录下，此时刷新浏览器的django管理后台即可看到正常样式。

2.经测试发现虽然后台管理界面正常了，但是访问主页还是没有加载模板文件，经过翻箱倒柜查资料，偶然间发现执行
>pkill uwsgi

发现主页即可恢复正常...为啥捏？？？

### 六.安装配置git
1.查看Git版本信息,如果版本较旧则更新
>git --version

2.依赖库安装
>yum install curl-devel expat-devel gettext-devel openssl-devel zlib-devel

>yum install gcc perl-ExtUtils-MakeMaker

3.如果原有的git版本过低，需要先移除git
>yum remove git

4.下载新版的Git源码包并编译安装
>cd /usr/local

>mkdir git

>cd git

>wget https://github.com/git/git/archive/v2.10.5.tar.gz

>tar -xzvf v2.10.5.tar.gz

>cd git-2.10.5

>make prefix=/usr/local/git all

>make prefix=/usr/local/git install

5.添加到环境变量
>vim /etc/profile //配置环境变量的文件，在这个文件中任意位置添加export PATH="/usr/local/git/bin:$PATH"

>source /etc/profile //使配置立即生效

6.将git设置为默认路径
>ln -s /usr/local/git/bin/git-upload-pack /usr/bin/git-upload-pack 

>ln -s /usr/local/git/bin/git-receive-pack /usr/bin/git-receive-pack

7.前面几步就可以安装好git，然后要使用git在github上克隆项目，还要设置好密钥对
>cd ~/.ssh

>ssh-keygen -t rsa -C 119@qq.com 

8.一路回车，然后输入
>ssh-add id_rsa

若出现： Could not open a connection to your authenticationagent.的提示，就先输入：ssh-agent bash，再使用 ssh-add id_rsa，然后设置密码。最后把id_rsa.pub文件的内容复制下来
>vim id_rsa.pub //按V键进入视图模式可以选择复制

然后登陆上github，在Setting>SSH and GPG keys 中 新建新的ssh key，名字随便起，内容贴上刚才复制出来的东西，保存。

9.创建git文档库。
进入venv虚拟环境下，cd到mblog文件夹
>git init

>git add .

>git commit -m '1 commit' //add和commit命令是更新本地文档库

>git remote add origin https://github.com/fikery/test.git

如果这时候报错已经存在origin，执行
>git remote rm origin 

>git push -u origin master

以上即可把本地项目上传到github托管。

10.对github上的项目进行clone，
>git clone git://github.com/fikery/test.git

11.对于一个经过多次操作的git文档库，一般是先pull更新，再push提交
>git pull(如果报错，则采用git pull origin master --allow-unrelated-histories)

>git git push -u origin master

ok，以上便是全部的基本配置流程，搞了我好几天，都是对Linux系统不熟悉，对web不了解导致的，哎，以后还是要多学习一个，涨点姿势方为上策呀~
