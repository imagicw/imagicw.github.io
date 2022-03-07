# 树莓派的笔记


因为近期开发学习需要一台服务器，正好手头有闲置的树莓派3B+，所以就打算利用起来。运行环境 `ubuntu 20.04.1 64位`

以下是一些备忘以及遇到的问题便记录了下来，日后可以参考。

### 修改默认账号

需要进入root账号后进行修改，ubuntu 需要修改 `/etc/ssh/sshd_config` 中 `PermitRootLogin yes` ，全部完成后需将其改为默认值。


```sh
# 修改用户名 ubuntu 为 imagic
usermod -l imagic ubuntu

# 修改组名 ubuntu 为 imagic
groupmod -n imagic ubuntu

# 更改 ubuntu 的 home 目录为 imagic
mv /home/ubuntu /home/imagic

# 修改 /etc/passwd 中 imagic 的 home 的位置
usermod -d /home/imagic imagic
```

在 sudoers 将 ubuntu 的信息修改成 `imagic ALL=(ALL) NOPASSWD: ALL` 。


### mongoDB 社区版

#### 安装

```sh
# 导入MongoDB 公共 GPG 密钥 mongodb5.0 不支持 armv7
wget -qO - https://www.mongodb.org/static/pgp/server-4.4.asc | sudo apt-key add -

// 添加 mongodb 软件源
echo "deb [ arch=amd64,arm64 ] https://repo.mongodb.org/apt/ubuntu focal/mongodb-org/4.4 multiverse" | sudo tee /etc/apt/sources.list.d/mongodb-org-4.4.list

sudo apt-get update
sudo apt-get install mongodb-org
```

需要安装指定版本的 mongoDB 则使用如下命令

```sh
sudo apt-get install -y mongodb-org=4.4.1 mongodb-org-server=4.4.1 mongodb-org-shell=4.4.1 mongodb-org-mongos=4.4.1 mongodb-org-tools=4.4.1
```

#### 运行

```sh
# 启动、重启、停止、状态
sudo service mongod start
sudo service mongod restart
sudo service mongod stop
sudo service mongod status
```

#### 移除

```sh
# 移除
sudo apt-get purge mongodb-org*
## 删除数据库及日志
sudo rm -r /var/log/mongodb
sudo rm -r /var/lib/mongodb
```

```sh

curl -sL https://deb.nodesource.com/setup_14.x | sudo bash -  


#tcp
iptables -t nat -N clash
iptables -t nat -A clash -d 0.0.0.0/8 -j RETURN
iptables -t nat -A clash -d 10.0.0.0/8 -j RETURN
iptables -t nat -A clash -d 127.0.0.0/8 -j RETURN
iptables -t nat -A clash -d 169.254.0.0/16 -j RETURN
iptables -t nat -A clash -d 172.16.0.0/12 -j RETURN
iptables -t nat -A clash -d 192.168.0.0/16 -j RETURN
iptables -t nat -A clash -d 224.0.0.0/4 -j RETURN
iptables -t nat -A clash -d 240.0.0.0/4 -j RETURN
iptables -t nat -A clash -d 10.0.0.3 -j RETURN
iptables -t nat -A clash -p tcp -j REDIRECT --to-port 7892
iptables -t nat -I PREROUTING -p tcp -d 8.8.8.8 -j REDIRECT --to-port 7892
iptables -t nat -I PREROUTING -p tcp -d 8.8.4.4 -j REDIRECT --to-port 7892
iptables -t nat -A PREROUTING -p tcp -j clash
iptables -t nat -A OUTPUT -p tcp -d 198.18.0.0/16 -j REDIRECT --to-port 7892

#udp
ip rule add fwmark 1 table 100
ip route add local default dev lo table 100
iptables -t mangle -N clash
iptables -t mangle -A clash -d 0.0.0.0/8 -j RETURN
iptables -t mangle -A clash -d 10.0.0.0/8 -j RETURN
iptables -t mangle -A clash -d 127.0.0.0/8 -j RETURN
iptables -t mangle -A clash -d 169.254.0.0/16 -j RETURN
iptables -t mangle -A clash -d 172.16.0.0/12 -j RETURN
iptables -t mangle -A clash -d 192.168.0.0/16 -j RETURN
iptables -t mangle -A clash -d 224.0.0.0/4 -j RETURN
iptables -t mangle -A clash -d 240.0.0.0/4 -j RETURN
iptables -t mangle -A clash -d 10.0.0.3 -j RETURN
iptables -t mangle -A clash -p udp -j TPROXY --on-port 7892 --tproxy-mark 1
iptables -t mangle -A OUTPUT -p udp -d 198.18.0.0/16 -j MARK --set-mark 1
iptables -t mangle -A PREROUTING -p udp -j clash
iptables -t nat -N CLASH_DNS
iptables -t nat -F CLASH_DNS 
iptables -t nat -A CLASH_DNS -p udp -j REDIRECT --to-port 1053
iptables -t nat -I OUTPUT -p udp --dport 53 -j CLASH_DNS
iptables -t nat -I PREROUTING -p udp --dport 53 -j REDIRECT --to 1053
iptables -t nat -A CLASH -d 240.0.0.0/4 -j RETURN
```
