# 设置Host映射文件
```
# vim /etc/hosts
192.168.253.107 spark1
192.168.253.108 spark2

# systemctl restart network
```

# 关闭防火墙
```
# systemctl stop firewalld
```

# 关闭SElinux
```
# getenforce
Enforcing

# vim /etc/selinux/config
SELINUX=disabled

# reboot
```

# 修改SSH配置文件
```
# vim /etc/ssh/sshd_config
RSAAuthentication yes
PubkeyAuthentication yes
AuthorizedKeysFile .ssh/authorized_keys

# systemctl restart sshd
```

# 配置SSH无密码登录
## 用root账户登录192.168.253.107
```
# ssh-keygen -t rsa  --生成公钥，一路回车

# cd ~

# ls -a --显示隐藏目录
.                .config    .gnome2_private  install.log.syslog  .scala_history
..               .cshrc     .gnote           .local              .ssh
.abrt            .dbus      .gnupg           .mozilla            .tcshrc
anaconda-ks.cfg  Desktop    .gstreamer-0.10  Music               Templates
.bash_history    Documents  .gtk-bookmarks   .nautilus           .themes
.bash_logout     Downloads  .gvfs            .oracle_jre_usage   .thumbnails
.bash_profile    .esd_auth  .ICEauthority    Pictures            Videos
.bashrc          .gconf     .icons           Public              .viminfo
.bashrc~         .gconfd    .imsettings.log  .pulse              .xinputrc
.cache           .gnome2    install.log      .pulse-cookie

# cd .ssh

# ls
id_rsa  id_rsa.pub

# cp id_rsa.pub authorized_keys_spark1

# ls
authorized_keys_spark1  id_rsa  id_rsa.pub
```

## 用root账户登录192.168.253.108
```
# ssh-keygen -t rsa

# cd ~

# ls -a

# cd .ssh

# ls
id_rsa  id_rsa.pub

# cp id_rsa.pub authorized_keys_spark2

# ls
authorized_keys_spark2  id_rsa  id_rsa.pub

# scp authorized_keys_spark2 root@192.168.253.107:/root/.ssh
```

## 切换到192.168.253.107
~~~javascript
[root@hadoop1 .ssh]# cd /root/.ssh
[root@hadoop1 .ssh]# ls
authorized_keys_hadoop1  authorized_keys_hadoop2  id_rsa  id_rsa.pub
[root@hadoop1 .ssh]# cat authorized_keys_hadoop1 >> authorized_keys
[root@hadoop1 .ssh]# ls
authorized_keys          authorized_keys_hadoop2  id_rsa.pub
authorized_keys_hadoop1  id_rsa
[root@hadoop1 .ssh]# cat authorized_keys_hadoop2 >> authorized_keys
[root@hadoop1 .ssh]# scp authorized_keys root@192.168.29.132:/root/.ssh
[root@hadoop1 .ssh]# ls
authorized_keys          authorized_keys_hadoop2  id_rsa.pub
authorized_keys_hadoop1  id_rsa                   known_hosts
~~~

##### 切换到192.168.29.132
~~~javascript
[root@hadoop2 .ssh]# cd /root/.ssh
[root@hadoop2 .ssh]# ls
authorized_keys  authorized_keys_hadoop2  id_rsa  id_rsa.pub  known_hosts
~~~

##### 在各个节点中设置authorized_keys读写权限
~~~javascript
# cd /root/.ssh
# chmod 400 authorized_keys
~~~

##### 测试ssh无密码登录是否生效
ssh连接远程主机时，会检查主机的公钥。如果是第一次连接该主机，会显示该主机的公钥摘要，提示用户是否信任该主机：
~~~javascript
[root@hadoop1 .ssh]# ssh root@hadoop2
The authenticity of host 'hadoop2 (192.168.29.132)' can't be established.
RSA key fingerprint is ad:6e:15:9b:c9:d1:9c:09:1a:a5:91:a0:77:af:cd:e2.
Are you sure you want to continue connecting (yes/no)?
~~~

~~~
当输入 yes 也就是选择接受之后，就会将该主机的公钥添加到文件 ~/.ssh/known_hosts 中。当再次连接该主机时，就不会再提示该问题了。
~~~

~~~javascript
Are you sure you want to continue connecting (yes/no)? yes
Warning: Permanently added 'hadoop2' (RSA) to the list of known hosts.

[root@hadoop1 .ssh]# ssh root@hadoop2
Last login: Thu Oct 13 09:55:45 2016 from hadoop1
[root@hadoop2 ~]# ls
anaconda-ks.cfg  Documents  install.log         Music     Public     Videos
Desktop          Downloads  install.log.syslog  Pictures  Templates
[root@hadoop2 ~]# exit
logout
Connection to hadoop2 closed.
~~~

#### 附加说明
##### 登录时提示Agent admitted failure to sign using the key
~~~javascript
[root@hadoop1 .ssh]# ssh root@hadoop2
Agent admitted failure to sign using the key.
root@hadoop2's password: 
~~~

解决方法：
~~~javascript
[root@hadoop1 .ssh]# ssh-add
Identity added: /root/.ssh/id_rsa (/root/.ssh/id_rsa)
~~~

##### 修改ssh对主机的public_key的检查等级

~~~javascript
# ssh -o StrictHostKeyChecking=no|ask|yes  IP地址
~~~

~~~
参数说明：
  no，最不安全的级别，如果连接server的key在本地不存在，那么就自动添加到文件中（默认是known_hosts），并且给出一个警告。
  ask，默认的级别，如果连接和key不匹配，给出提示，并拒绝登录。
  yes，最安全的级别，如果连接与key不匹配，就拒绝连接，不会有信息提示。
~~~
