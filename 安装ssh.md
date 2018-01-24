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
The authenticity of host '192.168.253.107 (192.168.253.107)' can't be established.
ECDSA key fingerprint is e2:94:c5:d5:40:69:ac:1b:88:04:5b:5e:61:37:ab:56.
Are you sure you want to continue connecting (yes/no)? yes
Warning: Permanently added '192.168.253.107' (ECDSA) to the list of known hosts.
root@192.168.253.107's password: 
authorized_keys_spark2                        100%  416     0.4KB/s   00:00   
```

## 切换到192.168.253.107
```
# cd /root/.ssh

# ls
authorized_keys_spark1  authorized_keys_spark2  id_rsa  id_rsa.pub

# cat authorized_keys_spark1 >> authorized_keys

# ls
authorized_keys         authorized_keys_spark2  id_rsa.pub
authorized_keys_spark1  id_rsa

# cat authorized_keys_spark2 >> authorized_keys

# ls
authorized_keys         authorized_keys_spark2  id_rsa.pub
authorized_keys_spark1  id_rsa

# scp authorized_keys root@192.168.253.108:/root/.ssh
The authenticity of host '192.168.253.108 (192.168.253.108)' can't be established.
ECDSA key fingerprint is 19:19:9a:b6:f5:28:9b:69:fd:90:5b:da:fa:ef:9b:b5.
Are you sure you want to continue connecting (yes/no)? yes
Warning: Permanently added '192.168.253.108' (ECDSA) to the list of known hosts.
root@192.168.253.108's password: 
authorized_keys                               100%  832     0.8KB/s   00:00 

# ls
authorized_keys         authorized_keys_spark2  id_rsa.pub
authorized_keys_spark1  id_rsa                  known_hosts
```

## 切换到192.168.253.108
```
# cd /root/.ssh

# ls
authorized_keys  authorized_keys_spark2  id_rsa  id_rsa.pub  known_hosts
```

# 在各个节点中设置authorized_keys读写权限
```
# cd /root/.ssh

# chmod 400 authorized_keys
```

# 测试ssh无密码登录是否生效
ssh连接远程主机时，会检查主机的公钥。如果是第一次连接该主机，会显示该主机的公钥摘要，提示用户是否信任该主机：
```
# ssh root@spark2
The authenticity of host 'spark2 (192.168.253.108)' can't be established.
ECDSA key fingerprint is 19:19:9a:b6:f5:28:9b:69:fd:90:5b:da:fa:ef:9b:b5.
Are you sure you want to continue connecting (yes/no)?
```
    当输入 yes 也就是选择接受之后，就会将该主机的公钥添加到文件 ~/.ssh/known_hosts 中。当再次连接该主机时，就不会再提示该问题了。
```
Are you sure you want to continue connecting (yes/no)? yes
Warning: Permanently added 'spark2' (ECDSA) to the list of known hosts.
Last login: Wed Jan 24 14:33:50 2018

# ifconfig
eno16777736: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500
        inet 192.168.253.108  netmask 255.255.255.0  broadcast 192.168.253.255
        inet6 fe80::20c:29ff:fe59:d6cf  prefixlen 64  scopeid 0x20<link>
        ether 00:0c:29:59:d6:cf  txqueuelen 1000  (Ethernet)
        RX packets 2532  bytes 865403 (845.1 KiB)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 2419  bytes 407063 (397.5 KiB)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0

# exit
logout
Connection to spark2 closed.
```

# 附加说明
## 登录时提示Agent admitted failure to sign using the key
```
# ssh root@spark2
Agent admitted failure to sign using the key.
root@spark2's password: 
```

解决方法：
```
# ssh-add
Identity added: /root/.ssh/id_rsa (/root/.ssh/id_rsa)
```

## 修改ssh对主机的public_key的检查等级
```
# ssh -o StrictHostKeyChecking=no|ask|yes  IP地址
```
参数说明：
  no，最不安全的级别，如果连接server的key在本地不存在，那么就自动添加到文件中（默认是known_hosts），并且给出一个警告。
  ask，默认的级别，如果连接和key不匹配，给出提示，并拒绝登录。
  yes，最安全的级别，如果连接与key不匹配，就拒绝连接，不会有信息提示。
