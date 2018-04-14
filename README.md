# Django学习笔记
1.在Django MTV架构下网站开发的步骤
    
    1.需求分析
    2.数据库设计
    3.设计模板文件(.html界面)
    4.创建并启用虚拟环境
    5.安装Django等包
    6.django-admin startproject生成项目
    7.python manage.py startapp 创建APP
    8.创建templates模板文件夹，放入网页模板.html文件
    9.创建static文件夹，放入静态文件
    10.修改settings.py
    11.编辑models.py，创建数据库表
    12.编辑views.py，导入models.py中创建的模型类
    13.编辑admin.py，导入models.py中创建的模型类，并使用admin.site.register注册
    14.编辑views.py，设计处理数据的模块
    15.编辑urls.py，导入views.py中定义的模块类，并创建网址和views.py中定义的模块类的对应关系
    16.执行python manage.py makemigrations
    17.执行python manage.py migrate
    18.执行python manage.py runserver测试网站

2.Djang的ORM操作最重要的是找到数据项（记录），把它放到一个变量p中，然后可以针对p做各种操作，最后调用p.save()就可以完成修改并反映到数据库中。

3.model是Django表示数据的模式，以python的类为基础在models.py中设置数据项与数据格式，基本上每个类对应一个数据库中的表。

4.view是Django最重要的程序逻辑所在的地方，网站大部分的程序设计放在这里。

5.view.py和urls.py的逻辑是，在urls.py中设置网址对应，然后在views.py中编写一个显示函数，通过HttpResponse传送出想要显示的数据。

6.view中不应该处理网页呈现的问题，出现HTML标记的语言都应该在template文件中处理，相对而言view中只负责变量的传入传出。

