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
