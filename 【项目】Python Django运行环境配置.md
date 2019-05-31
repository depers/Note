# Python Django运行环境配置

## 1.安装python及相关软件

* 安装python2.7.5
* 安装mysql
* 安装相关包
  * 安装virtualenv，`pip install vitualenv`
  * 激活虚拟环境`source env/bin/activate`
  * 进入项目目录，通过` pip freeze > requirements.txt `将本地的虚拟环境安装包相信信息导出来
  * 然后`pip install -r requirements.txt`安装依赖包

* 安装uwsgi，输入`pip install uwsgi`

**注意：安装MySQL-python-1.2.5问题，出现错误：`EnvironmentError: mysql_config not found`**

解决办法：`yum install mysql-devel`

## 2.配置nginx文件

* bravedawn.cn.conf

  ```nginx
  upstream django {
  	server 127.0.0.1:8000; # 本地服务端口
  }
  # configuration of the server
  
  server {
  	# the port your site will be served on
  	listen      80;
  	# the domain name it will serve for
  	server_name www.bravedawn.cn; # 配置域名
  	charset     utf-8;
  
  	# max upload size
  	client_max_body_size 75M;   # adjust to taste
  
  	# Django media
  	location /media  {
  	    alias /usr/local/webapp/pythonweb/blog/media; # 设置media文件位置
  	}
  
  	location /static {
  	    alias /usr/local/webapp/pythonweb/blog/static; # 设置static文件位置
  	}
  
  	# Finally, send all non-media requests to the Django server.
  	location / {
  	    uwsgi_pass  django;
  	    include     uwsgi_params; # the uwsgi_params file you installed
  	}
  }
  ```

* 拉取所有需要的static file 到同一个目录

  * 在django的setting文件中，添加下面一行内容：

    STATIC_ROOT = os.path.join(BASE_DIR, "static/")

  * 运行命令
    `python manage.py collectstatic`

* 修改setting.py

  ```python
  # SECURITY WARNING: don't run with debug turned on in production!
  DEBUG = False
  ```

* 重启nginx

## 3.配置文件启动uwsgi

* uwsgi.ini

  ```
   # mysite_uwsgi.ini file
  [uwsgi]
  
  # Django-related settings
  # the base directory (full path)
  chdir=/usr/local/webapp/pythonweb/blog
  # Django's wsgi file
  module=blog.wsgi
  # the virtualenv (full path)
  
  # process-related settings
  # master
  master=true
  # maximum number of worker processes
  processes=10
  # the socket (use the full path to be safe
  socket=127.0.0.1:8000
  # ... with appropriate permissions - may be needed
  # chmod-socket = 664
  # clear environment on exit
  vacuum=true
  virtualenv=/usr/local/webapp/pythonweb/blog_venv
  logto=/tmp/mylog.log
  ```

  注：

  * chdir： 表示需要操作的目录，也就是项目的目录
        

  * module： wsgi文件的路径
        

  * processes： 进程数
        

  * virtualenv：虚拟环境的目录

* 启动uwsgi，输入`uwsgi -i uwsgi.ini &`