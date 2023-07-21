####openvpn 搭建
##安装python3.7
```
yum install gcc* automake zlib-devel libjpeg-devel giflib-devel freetype-devel
yum -y install make gcc-c++ cmake bison-devel ncurses-devel
wget https://www.python.org/ftp/python/3.7.13/Python-3.7.13.tar.xz
tar -xf Python-3.7.13.tar.xz
./configure --prefix=/usr/local/python3
make && make install
```

##安装openvpn
```
wget https://openvpn.net/downloads/openvpn-as-latest-CentOS7.x86_64.rpm
wget https://openvpn.net/downloads/openvpn-as-bundled-clients-latest.rpm
yum -y install openvpn-*.rpm
```

##破解openvpn
```
[root@vm-24-13-centos openvpnas]# cd /root/openvpnas
[root@vm-24-13-centos openvpnas]# mkdir compile && cd $_
[root@vm-24-13-centos compile]# cp /usr/local/openvpn_as/lib/python/pyovpn-2.0-py3.6.egg .
[root@vm-24-13-centos compile]# unzip -q ./pyovpn-2.0-py3.6.egg
[root@vm-24-13-centos compile]# ls
common  EGG-INFO  pyovpn  pyovpn-2.0-py3.6.egg
[root@vm-24-13-centos compile]# cd pyovpn/lic/
[root@vm-24-13-centos lic]# ls -a
.  ..  conf  info.pyc  __init__.pyc  ino.pyc  lbcs.pyc  lbq.pyc  lic_entry.pyc  licerror.pyc  lichelper.pyc  lickey.pyc  licser.pyc  licstore.pyc  liman.pyc  lspci.pyc  prop.pyc  uprop.pyc  vprop.pyc
[root@vm-24-13-centos lic]# mv uprop.pyc uprop2.pyc
[root@vm-24-13-centos lic]# vim uprop.py
[root@vm-24-13-centos lic]# cat uprop.py
from pyovpn.lic import uprop2
old_figure = None
 
def new_figure(self, licdict):
      ret = old_figure(self, licdict)
      ret['concurrent_connections'] = 6666
      return ret
 
for x in dir(uprop2):
      if x[:2] == '__':
         continue
      if x == 'UsageProperties':
         exec('old_figure = uprop2.UsageProperties.figure')
         exec('uprop2.UsageProperties.figure = new_figure')
      exec('%s = uprop2.%s' % (x, x))
[root@vm-24-13-centos lic]# python -O -m compileall uprop.py && mv __pycache__/uprop.*.pyc uprop.pyc
Compiling 'uprop.py'...
 
# 最后打包一下就结束了
[root@vm-24-13-centos lic]# cd ../../
[root@vm-24-13-centos compile]# zip -rq pyovpn-2.0-py3.6.egg ./pyovpn ./EGG-INFO ./common
# 替换补丁包
cp ./pyovpn-2.0-py3.6.egg /usr/local/openvpn_as/lib/python/
# 重启openvpn
systemctl restart openvpnas.service
```

grep -i 'password.$' /usr/local/openvpn_as/init.log
