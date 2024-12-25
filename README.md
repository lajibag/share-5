<h1>第五组：季亚辉 孙玉洁 周奥 耿浩文</h1>
<h2>任务一：samba文件共享服务器</h2>

<h3>1.配置华为开源的openEuler镜像源</h3>

```
Wget -0 /etc/yum.repos.d/openEulerOS.repo https://mirrors.huaweicloud.com/repository/conf/openeuler_x86_64.repo
```
#清理 yum 的缓存:
```
yum clean all
```
#更新 yum 的缓存:
```
yum makecache
```

<h3>2.安装samba，samba公共组件和samba客户端</h3>

```
dnf -y install samba samba-common samba-client
```

![](https://github.com/lajibag/share-5/blob/master/%E5%9B%BE%E7%89%871.png)

![](https://github.com/lajibag/share-5/blob/master/%E5%9B%BE%E7%89%872.png)
<h3>3.启动samba，设置成系统启动时自动启动</h3>

```
Systemctl start smb; systemctl enable smb
```

观察139端口是否启动：
```
netstat -lantp | grep 139
```
<h3>3.创建用户环境</h3>
创建samba所用到的两个目录
```
mkdir /var/share /var/smb
```

修改这两个文件的权限
```
chmod 777 /var/share /var/smb
```
创建smb用户
```
useradd -s /sbin/nologin -M smb
```
设置用户密码
```
smbpasswd -a smb
```
修改etc/samba中的samba文件
```
vim /etc/samba/smb.conf
```

在global下的security = user后面添加一行代码使其可以匿名登录

![](https://github.com/lajibag/share-5/blob/master/%E5%9B%BE%E7%89%873.png)

配置share目录权限

![](https://github.com/lajibag/share-5/blob/master/%E5%9B%BE%E7%89%874.png)

重启samba服务器并查看samba的状态
```
systemctl restart smb
systemctl status smb
```
<h3>4.实现共享文件</h3>
将smb目录添加到smb用户以及群组的权限中
```
chown smb:smb /var/smb
```
再次编辑配置文件
```
vim /etc/samba/smb.conf
```

在share下面添加smb相关配置

![](https://github.com/lajibag/share-5/blob/master/%E5%9B%BE%E7%89%875.png)


重启samba服务器并查看samba的状态
```
systemctl restart smb
systemctl status smb
```

![](https://github.com/lajibag/share-5/blob/master/%E5%9B%BE%E7%89%876.png)

<h3>5.编写脚本实现对samba的简单运维</h3>
创建并编辑脚本文件
```
vim /root/backup.sh
```

![](https://github.com/lajibag/share-5/blob/master/%E5%9B%BE%E7%89%877.png)

为其添加可执行权限
```
chmod +x /root/backup.sh
```

运行脚本
```
sh /root/backup.sh
```

检测是否打包完成
```
ls -l /var/smb
```

设置定期的周期性任务
```
crontab -e
```

输入
```
0 22 * * * sh  /root/backup.sh  //每天执行一次backup.sh脚本
```
<h3>6.查看samba日志</h3>
```
ls /var/log | grep samba
```
进入samba目录
看后5行日志
```
cd /var/log/samba/
tail log.smbd -5
```

![](https://github.com/lajibag/share-5/blob/master/%E5%9B%BE%E7%89%878.png)
