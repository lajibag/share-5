1.	<h3>配置华为开源的openEuler镜像源</h3>
```
Wget -0 /etc/yum.repos.d/openEulerOS.repo https://mirrors.huaweicloud.com/repository/conf/openeuler_x86_64.repo
```
清理 yum 的缓存:
```
yum clean all
```
更新 yum 的缓存:
```
yum makecache
```
2.	安装samba，samba公共组件和samba客户端
```
dnf -y install samba samba-common samba-client
```
3.	启动samba，设置成系统启动时自动启动
```
Systemctl start smb; systemctl enable smb
```
观察139端口是否启动：
```
netstat -lantp | grep 139
```
4.	创建samba所用到的两个目录
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
```
map to guest =Bad User
```
重启samba服务器
```
systemctl restart smb
```
查看samba的状态
```
systemctl status smb
```
5.	将smb目录添加到smb用户以及群组的权限中
```
chown smb:smb /var/smb
```
再次编辑配置文件
```
vim /etc/samba/smb.conf
```
在share下面添加smb相关配置
```
[smb]
        comment = smb
        path = /var/smb
        write list = smb
        browseable = yes
        writeable = yes
        read list = smb
        valid users = smb
        create mask = 0777
        directory mask = 0777

```
重启samba服务器
```
systemctl restart smb
```
查看samba的状态
```
systemctl status smb
```
6.	编写脚本实现对samba的简单运维
创建并编辑脚本文件
```
vim /root/backup.sh
```
```
#! /bin/sh
mkdir /var/backup    //用于存放备份文件
cp -r /var/share/ /var/backup/  //复制共享目录
tar -zcPvf /var/smb/backup$(date +%Y%m%d).tar.gz /var/backup  //压缩备份文件，并添加当前日期名
rm -rf /var/backup/     //删除旧备份文件
find /var/smb/ -mtime +30 -name "*.tar.gz" -exec rm -rf {} \;
```

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
7.	查看samba日志
```
ls /var/log | grep samba
```
进入samba目录
看后5行日志
```
cd /var/log/samba/
tail log.smbd -5
```
