总结一下在`winserver2008`下基于`IIS8`和`wfastcgi`架设`django2.0`站点。
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

1.在app目录下新增`static`文件夹，然后在`settings.py`中设置`STATIC_ROOT`路径为上述文件夹路径，然后执行`python manager.py collectstatic`，可将静态文件复制到`static`文件夹下。

2.在上述`static`文件夹下新建`web.config`文件，模板如下

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
    
3.打开`IIS管理器`，点击网站，右键选择添加虚拟路径，命名可以是`static`，路径为上述`static`文件夹路径。

4.重新运行IIS即可样式正常，外网可以正常访问。

## 五.其他注意事项

1.在APP目录下新增`static`文件夹后，还要将project目录下的`static`文件夹内的内容，包括css、image和js文件拷贝到新的目录下，这样才能保证网页的样式正常。事实上，如果按照新版的django官方教程，`static`文件夹应当建立在app目录下，包括`templates`文件夹也是如此。旧版的网上教程，普遍是将这些静态文件和模板文件放在了project目录下，因此如果根据新版(个人看的是django2.0)官方教程，可以避免不少的麻烦操作。

2.在将网站部署到IIS上时发现，url中有中文的时候，`urls.py`路由会将中文按照`gb2312`编码，但是进入`views.py`后解码却是`utf-8`，因此这导致了url链接有中文字符引发后续操作异常。但奇怪的是，在本地测试的时候，并不会出现这个问题，测试发现本地均是以`utf-8`编码和解码的。这个问题理论上可以有多种解决方式：传递中文字符之前进行指定编码，后面再指定解码，或者是在view.py中进行编码的单独处理。这里采用后者，仅仅两行代码

    dataQ=urllib.parse.quote(data).replace('%25','%')
    dataUq=urllib.parse.unquote(dataQ,encoding='gb2312')

使用quote重新编码，然后再使用unquote重新解码得到正确中文，这里以例子进行说明：例如url中包括`武汉`中文字符，其url编码的`utf-8`编码是a `%e6%ad%a6%e6%b1%89`，`gb2312`编码是b `%ce%e4%ba%ba`,理论上当`url.py`中捕获到中文时，应该采用a方式编码，但是实际上采用了b方式编码，然后在`views.py`中解码时采用了a方式解码，结果解码出异常码c `%ce人`(如果采用网上的url编码网站进行测试会发现，对b进行a方式解码，会得到`�人`，但是`views.py`中进行Httpresponse()实际输出得到的是c。另外对`%ce`进行a方式的url解码得到`�`,但是反过来编码却不是，疑惑ing...)。因此现在针对c进行还原处理，`dataQ`得到的是b,然后dataUq得到`武汉`字符。
