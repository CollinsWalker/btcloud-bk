# Linux面板官方更新包修改记录

查询最新版本号：https://www.bt.cn/api/panel/get_version?is_version=1

官方更新包下载链接：http://download.bt.cn/install/update/LinuxPanel-版本号.zip

假设搭建的宝塔第三方云端网址是 http://www.example.com

- 将class文件夹里面所有的.so文件删除

- 将PluginLoader.py复制到class文件夹

- 批量解密模块文件：执行 php think decrypt classdir <面板class文件夹路径>

- 全局搜索替换 https://api.bt.cn => http://www.example.com

- 全局搜索替换 https://www.bt.cn/api/ => http://www.example.com/api/（需排除clearModel.py、ipsModel.py）

- 全局搜索替换 http://download.bt.cn/install/update6.sh => http://www.example.com/install/update6.sh

- class/ajax.py 文件 \#是否执行升级程序 下面的 public.get_url() 改成 public.GetConfigValue('home')

  class/jobs.py 文件 \#尝试升级到独立环境 下面的 public.get_url() 改成 public.GetConfigValue('home')

  class/system.py 文件 RepPanel和UpdatePro方法内的 public.get_url() 改成 public.GetConfigValue('home')

- class/public.py 在 

  ```python
  def GetConfigValue(key):
  ```

  这一行下面加上

  ```python
  if key == 'home': return 'http://www.example.com'
  ```

  在 def is_bind(): 这一行下面加上 return True

  在 def check_domain_cloud(domain): 这一行下面加上 return

- class/panelPlugin.py 文件，download_icon方法内替换 public.GetConfigValue('home') => 'https://www.bt.cn'

- class/panelPlugin.py 文件，删除public.total_keyword(get.query)这一行

- class/panelPlugin.py 文件，set_pyenv方法内，temp_file = public.readFile(filename)这行代码下面加上

  ```python
  temp_file = temp_file.replace('wget -O Tpublic.sh', '#wget -O Tpublic.sh')
  temp_file = temp_file.replace('\cp -rpa Tpublic.sh', '#\cp -rpa Tpublic.sh')
  temp_file = temp_file.replace('http://download.bt.cn/install/public.sh', 'http://www.example.com/install/public.sh')
  ```
  
- install/install_soft.sh 在bash执行之前加入以下代码

  ```shell
  sed -i "s/http:\/\/download.bt.cn\/install\/public.sh/http:\/\/www.example.com\/install\/public.sh/" lib.sh
  sed -i "/wget -O Tpublic.sh/d" $name.sh
  ```
  
- install/public.sh 用官网最新版的[public.sh](http://download.bt.cn/install/public.sh)替换，并去除最下面bt_check一行

- 去除无用的定时任务：task.py 文件  删除以下几行

  "update_software_list": update_software_list,

  "check_panel_msg": check_panel_msg,

- 去除首页广告：BTPanel/static/js/index.js 文件删除最下面index.recommend_paid_version()这一行

- 去除内页广告：BTPanel/templates/default/layout.html 删除getPaymentStatus();这一行

- 去除首页自动检测更新，避免频繁请求云端：BTPanel/static/js/index.js 文件注释掉bt.system.check_update这一段代码外的setTimeout

- [可选]去除各种计算题：复制bt.js到 BTPanel/static/ ，在 BTPanel/templates/default/layout.html 的\</body\>前面加入

  ```javascript
  <script src="/static/bt.js"></script>
  ```

- [可选]去除创建网站自动创建的垃圾文件：在class/panelSite.py，分别删除

  htaccess = self.sitePath+'/.htaccess'

  index = self.sitePath+'/index.html'

  doc404 = self.sitePath+'/404.html'

  这3行及分别接下来的4行代码

- [可选]关闭未绑定域名提示页面：在class/panelSite.py，root /www/server/nginx/html改成return 400

- [可选]关闭自动生成访问日志：在 BTPanel/\_\_init\_\_.py  删除public.write_request_log()这一行


解压安装包panel6.zip，将更新包改好的文件覆盖到里面，然后重新打包，即可更新安装包。（

别忘了删除class文件夹里面所有的.so文件）

