总结一下在winserver2008下基于IIS8和wfastcgi架设django2.0站点。
参考[主要配置]( https://blog.csdn.net/gzlaiyonghao/article/details/70243639) 和[css消失修复](https://blog.csdn.net/qq_18075613/article/details/56970016) 

## 一.安装python3.6

如果是安装的`anaconda`，可以直接默认安装，如果是直接安装`python3.6`则需要注意不要安装在默认目录，可安装在`c:\\python36\`文件夹下，防止后续出现调用`python`的权限等各种问题。安装时注意选择环境变量的配置。

## 二.安装wfastcgi
    
    pip install wfastcgi #安装wfastcgi模块  
    wfastcgi-enable #启用并输出所在地址
    
## 三.配置web.config

在`IIS`中新建一个网站，把域名分配过去就好。然后在`manager.py`的同级目录新建一个文本文件`web.config`，里面的内容有个模板可以套：

    <?xml version="1.0" encoding="UTF-8"?>
    <configuration>
        <system.webServer>
            <handlers>
                <add name="Python FastCGI" 
                     path="*" 
                     verb="*" 
                     modules="FastCgiModule" 
                     scriptProcessor="<Path to Python>\python.exe|<Path to Python>\lib\site-packages\wfastcgi.py" 
                     resourceType="Unspecified" 
                     requireAccess="Script"/>
            </handlers>
        </system.webServer>
        <appSettings>
            <add key="WSGI_HANDLER" value="django.core.wsgi.get_wsgi_application()" />
            <add key="PYTHONPATH" value="<Path to Django project>" />
            <add key="DJANGO_SETTINGS_MODULE" value="<Django App>.settings" />
        </appSettings>
    </configuration>

`scriptProcessor`的值，要改为前文说过的运行`wfastcgi`输出的那个值。`PYTHONPATH`的`value`要改为`manager.py`的那个目录，也就是你项目的根目录。`DJANGO_SETTINGS_MODULE`的`value`中的`<Django App>`要改为你的项目名。

## 四.复制admin文件的样式文件

1.在app目录下新增static文件夹，然后在`settings.py`中设置`STATIC_ROOT`路径为上述文件夹路径，然后执行`python manager.py collectstatic`，可将静态文件复制到static文件夹下。

2.在上述static文件夹下新建`web.config`文件，模板如下

    <?xml version="1.0" encoding="UTF-8"?>  
    <configuration>  
      <system.webServer>  
        <!-- this configuration overrides the FastCGI handler to let IIS serve the static files -->  
        <handlers>  
        <clear/>  
          <add name="StaticFile" path="*" verb="*" modules="StaticFileModule" resourceType="File" requireAccess="Read" />  
        </handlers>  
      </system.webServer>  
    </configuration>  
    
3.打开IIS管理器，点击网站，右键选择添加虚拟路径，命名可以是static，路径为上述static文件夹路径

4.重新运行IIS即可样式正常
