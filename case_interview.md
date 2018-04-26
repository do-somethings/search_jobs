# 面试题

## 一、Linux命令相关

### 1.awk

#### 1.1 操作2个文件
现有两个文件a与b，内容分别为：
```
[root@test ~]# cat a
北京 电信
云南 联通
[root@test ~]# cat b
北京 电信 1
北京 联通 3
云南 联通 6
```
请写出对应命令，输出以下内容：
```
北京 电信 1
云南 联通 6
```

```
[root@test ~]# awk 'NR==FNR{a[$1,$2]=$3}NR>FNR{print $1,$2,a[$1,$2]}' b a
北京 电信 1
云南 联通 6
[root@test ~]# awk 'NR==FNR{a[$1,$2]=$3;next}{print $1,$2,a[$1,$2]}' b a
北京 电信 1
云南 联通 6
[root@test ~]# awk 'NR==FNR{a[$1$2]=$3;next}{print $1,$2,a[$1$2]}' b a  
北京 电信 1
云南 联通 6
[root@test ~]# awk 'NR==FNR{a[$1$2]=$0;next}{ temp=$1$2; if (temp in a) print a[temp],$3}' a b
北京 电信 1
云南 联通 6
[root@test ~]# awk 'NR==FNR{a[$1$2]=$0;next}{for (i in a) {if (i==$1$2) print a[i],$3} }' a b 
北京 电信 1
云南 联通 6

```
#### 1.2 请使用sed/awk/grep的任一命令输出以下文件中的网卡名，即"wlp1s0

文件内容

```
[root@test ~]# cat test.file 
IP-address      HW-type     Flags       HW-address            Mask     Device
130.48.0.1       0x1         0x2         80:4b:c7:10:3e:41     *        wlp1s0
```
对应命令
```
[root@test ~]# awk '/^[0-9]/{print $6}' test.file 
wlp1s0
[root@test ~]# awk 'NR==2{print $NF}' test.file 
wlp1s0
[root@test ~]# awk 'NR>1{print $6}' test.file                       
wlp1s0
[root@test ~]# awk -F "[ ]+" 'NR==1{for(i=1;i<=NF;i++) f[$i]=i;next}{print $(f["Device"])}' test.file        
wlp1s0
(next代表进行下一轮循环，{print $(f["Device"])}其实打印的是NR==2即第二行对应的内容)
[root@test ~]# sed -nr '/^[0-9]/s#(.*)\*[^a-z]*(.*)#\2#gp' test.file 
wlp1s0
```


### 2.字符串反转

请使用任一语言对”abcd”字符串进行反转，输出”dcba”

#### 2.1 python

##### 2.1.1 使用字符串内置方法，设置步长为1
```
s='abcd'
print s[::-1]
dcba
```

**原理：**

- 解释一：

This is extended slice syntax. It works by doing [begin:end:step] 
-by leaving begin and end off and specifying a step of -1, it reverses a string.

- 解释二：

a[起始位(包含):结束位(不包含):差值]，[详见Stack Overflow](https://stackoverflow.com/questions/5846004/unable-to-reverse-lists-in-python-getting-nonetype-as-list)
```
a = range(20)
Slices let you take a piece of an array. it works like a[beginIndexIncluded:endIndexExcluded:Step]
a[::-1] includes all elements of a, but starts with the last one and ends with the first (reversal) 
```


##### 2.1.2 使用内置的reversed()方法

```
print '' .join(reversed(s))
dcba
```
##### 2.1.3 for循环，从右到左输出
range(start, stop[, step])

```
#!/usr/bin/python
# -*- coding:utf-8 -*-
def reverse_string(s):
    s_list=[]
    for i in range(len(s)-1,-1,-1):
        s_list.append(s[i])
    return '' .join(x for x in s_list)

if __name__ == '__main__':
    string='abcd'
    print 'The original_string is: {0}' .format(string) 
    print 'The reversed string is: {0}' .format(reverse_string(string))
```
或
```
print ''.join(s[i] for i in range(len(s)-1, -1, -1)) 
dcba
```
##### 2.1.4 借用列表reverse()方法
```
s='abcd'
l = list(s)
**注意列表的reverse()方法返回值为None，若想有返回值可使用reversed()方法,即reversed(l)**
# use reverse()
l.reverse()
print '' .join(x for x in l)
# use reversed()
print '' .join(x for x in reversed(list(s)))
dcba
```

#### 2.2 shell
```
# echo "abcd" |awk -F "" '{for(i=NF;i>1;i--)printf("%s",$i);print $1}'
dcba
```

### 3. locate与slocate区别
```
slocate只搜索当前用户有权限的文件或目录，locate会搜索所有文件

Slocate takes into account file and directory permissions when searching. slocate won't list files in directories 
that a user does not have permissions to list.
```

### 4. linux下比较两个目录下的文件
```
[root@test ~]# ls test1
1  2
[root@test ~]# ls test2
1
[root@test ~]# diff -r test1 test2
Only in test1: 2
```

### 5. 列出网络排查故障的几个命令

```
ping:检查连通性
telnet/nc：探测端口是否开放
telnet 192.168.1.1 80
nc –zv –w 5 192.168.1.1 80
traceroute/tracepath/mtr：路由跟踪
nslookup/dig：域名解析
ethtool：查看网卡物理连接状态
ifconfig eth0/ip link ls eth0网卡是否正常启用
route –n：查看内核路由表
iptables –nL：查看防火墙规则
iftop：查看连接占用的网络带宽
tcpdump：杀手锏-抓包
```
### 6. 屏蔽所有机器对某台机器80端口的访问

```
iptables -A INPUT -p tcp --dport 80 -j DROP
```

### 7. 查看当前系统每个IP的连接数

```
netstat -ntu | awk -F "[ :]+" '/^tcp/{num[$4]++}END{for(i in num) print num[i],i}' |sort -nr
```

### 8. 每天的早上6点到12点中，每隔2小时创建一个test.txt文件

```
* 6,12/2 * * * /bin/touch test.txt
```

### 9. 查找linux系统/data1/目录下以log和gz结尾的30天没有修改的文件，并删除。

```
find /data1 -type f -iname "*.log" -mtime +30 -o -iname "*.gz" -mtime +30 –delete
find /data1 -type f \( -name "*.gz" -o -name "*.log" \) -mtime +30 –delete
find /data1 -type f -regex '.*\(\.gz\|\.log\)' -mtime +30 –delete
find /data1 -type f -regextype posix-extended -regex '.*.(log|gz)' -mtime +30 –delete
```

### 10. nginx日志处理，计算平均响应时间，取top10IP

一段nginx日志如下：
```
mobile.internal.sina.com.cn 10.75.24.59 0.012s -[16/Apr/2018:16:03:56 +0800] - "/api/phone/getphones" - 200 256 "-" -  "Jakarta Commons-HttpClient/3.1"
```

通过nginx访问日志：
- 计算平均响应时间

**注意需要去掉第三列中的单位s，否则无法进行数学运算。**

```
awk 'BEGIN{num=0;sum=0}{l=length($3);rt=substr($3,1,l-1);num++;sum+=rt}END{printf "%0.3f\n", sum/num}' nginx.log
```

- 按IP访问量列出前十名
```
awk '{num[$2]++}END{for(i in num) print num[i],i}' |sort –nr nginx.log
```



## 二、服务相关

### 1.nginx

#### 1.1 简述X-Forwarded-For作用

```
X-Forwarded-For 是一个 HTTP 扩展头部，主要是为了让 Web 服务器获取访问用户的真实 IP 地址。

X-Forwarded-For是用于记录代理信息的，每经过一级代理(匿名代理除外)，代理服务器都会把这次请求的来源IP追加在X-Forwarded-For中。
```

#### 1.2 nginx跨域配置

CORS（Cross-Origin Resource Sharing 跨源资源共享），当一个请求url的协议、域名、端口三者之间任意一与当前页面地址不同即为跨域。
```
http {
......
add_header Access-Control-Allow-Origin '*';
add_header Access-Control-Allow-Headers 'X-Requested-With,Content-Type';
add_header Access-Control-Allow-Methods 'GET,POST,OPTIONS';
......
}
```
这样就可以实现GET,POST,OPTIONS的跨域请求的支持 
也可以 add_header Access-Control-Allow-Origin http://test.51testing.com; --指定允许的url;

#### 1.3 nginx中location 匹配的优先级
示例1：
```
(location = 完整路径) > (location 完整路径) >(location ^~ 路径) >(location ~* 正则) >(location 路径)
>(location /)
比如：
 = /poechant/a.jpg  > /poechant/a.jpg > ^~ /poechant/ > ~* \.jpg$ > /poechant/ > /
```
**同等条件下带正则表达式的优先级低**

示例2：
```
location 匹配的优先级:
(location = 完整路径) > (location 完整路径) >(location ^~ 路径) >(location ~* 正则) >(location 路径)
>(location /)

A. location = 完整路径：   location = /test/index.html     经常用与首页，即location = /    常用
B. location 完整路径：     location /test/index.html
C. location ^~ 路径:       location ^~ /test   不支持正则， 最长前缀匹配规则， 如/a1/a2/和/a1/两者间，会匹配进前者，与配置文件顺序无关
D. location ~* 正则:       location ~* /.*\.(html|htm|php)$              常用
E. location 路径:          location  /test
F. location / ：           location /       通用规则，匹配到所有       常用

匹配优先级：
A>B>C>D>E>F
```


### 2.HTTP

#### 2.1 线上业务日志出现5xx,列出排查思路

- 5xx状态码含义

```
（1）500 Internal Server Error：
服务器遇到了一个未曾预料的状况，导致了它无法完成对请求的处理。一般来说，这个问题都会在服务器的程序码出错时出现。

（2）501 NotImplemented：
服务器不支持当前请求所需要的某个功能。服务器无法识别请求的方法，并且无法支持其对任何资源的请求。

（3）502 Bad Gateway：
作为网关或者代理工作的服务器尝试执行请求时，从上游服务器接收到无效的响应。

（4）503 Service Unavailable：
由于临时的服务器维护或者过载，服务器当前无法处理请求。这个状况是临时的，并且将在一段时间以后恢复。

（5）504 Gateway Timeout：
作为网关或者代理工作的服务器尝试执行请求时，未能及时从上游服务器（URI标识出的服务器，比如HTTP、FTP、LDAP）或者辅助服务器（比如DNS）收到响应。
```

- 排查思路

* 502
 
```
1.php-fpm资源耗尽
解决方案：
去调整php-fpm.conf中的pm.max_children数值
2.php-fpm.conf文件设置的不是nginx的主和组
解决方案：
编辑php-fpm文件需要在这个php-fpm文件里面设置nginx的用户主和组
3.流量攻击导致
查看php-cgi是否在运行
有时候由于网站流量过大或者其它原因，导致php-cgi直接down掉
解决方案：
启动php-cgi
4.FastCGI执行时间过长
根据实际情况调高以下参数值
fastcgi_connect_timeout 200;
fastcgi_send_timeout 200;
fastcgi_read_timeout 200;
```
    
* 504

```
Nginx 504 Gateway Time-out的含义是没有请求到可以执行的PHP-CGI。
如果已有的PHP请求处理较慢，新的PHP一直处于等待状态，直至超过Nginx的 fastcgi_read_timeout的值，就会出现504 Gateway timeout的错误
```


## 三、虚拟化

### 1. docker

#### 1.1 docker有几种网络模式

- host模式

与宿主机共用网络栈，IP和端口，容器本身看起来就像是个普通的进程，它暴露的端口可直接通过宿主机访问。相比于bridge模式，host模式有显著的性能优势(因为走的是宿主机的网络栈，而不是docker deamon为容器虚拟的网络栈)。docker host上已经使用的端口在容器中就不能再用了

- container模式

使用--net=container:container_id/container_name多个容器使用共同的网络，看到的ip是一样的，两个容器除了网络方面，其他的如文件系统、进程列表等还是隔离的。

- none模式

这种网络模式下容器只有lo回环网络，没有其他网卡。none网络可以在容器创建时通过--network=none来指定。这种类型的网络没有办法联网，封闭的网络能很好的保证容器的安全性。

- bridge模式

**默认网络模式**，此模式下，容器有自己的独立的Network Namespace。简单来说，Docker在宿主机上虚拟了一个子网络，宿主机上所有容器均在这个子网络中获取IP，这个子网通过网桥挂在宿主机网络上。Docker通过NAT技术确保容器可与宿主机外部网络交互。

- overlay模式

容器在两个跨主机进行通信的时候，是使用overlay network这个网络模式进行通信。

- macvlan模式

通过为物理网卡创建Macvlan子接口，允许一块物理网卡拥有多个独立的MAC地址和IP地址。虚拟出来的子接口将直接暴露在底层物理网络中。从外界看来，就像是把网线分成多股，分别接到了不同的主机上一样。Macvlan是Linux内核支持的网络接口。要求的Linux内部版本是v3.9–3.19和4.0+。

- 官网介绍及相关博客

[docker network-drivers from official site](https://docs.docker.com/network/#network-drivers)

[understanding-docker-networking-drivers-use-cases](https://blog.docker.com/2016/12/understanding-docker-networking-drivers-use-cases/)

#### 1.2 进程关联性

假设一台物理机上启动有多个docker，怎么查这台机器上的一个进程是运行在哪个docker上。

以nginx为例说明，可参考下图。

![2.jpg](https://i.loli.net/2018/04/26/5ae1a4365048c.jpg)

```
使用ps –ef |grep nginx查看nginx守护进程对应的PPID1 4084
使用ps –ef |grep docker|grep PPID1找到对应containerd-shim的PPID2 2526
使用ps –ef |grep docker|grep PPID2找到对应containerd进程的PPID3 2521
使用ps –ef |grep docker|grep PPID3找到对应docker启动进程

Docker、containerd和containerd-shim之间的关系：
当启动容器之后，docker-containerd进程（也是containerd组件）会创建docker-containerd-shim进程
```

## 四、 系统&内核&调优

### 1. 列出一台机器负载高的排查思路
```
(1）uptime 与w命令查看平均负载
(2）使用top命令查看负载，在top下按“1”查看CPU核心数量，shift+"c"按cpu使用率大小排序，shif+"p"按内存使用率高低排序；
(3) top -Hp 进程PID,找到相关线程PID
(4) 使用iostat -x 1 10命令来监控io的输入输出是否过大, 看“util”的百分比，就是IO使用率
(5）使用free -m命令查看系统内存

其他命令：
/iotop/vmstat/perf/strace/ltrace/sysdig
```

### 2. 系统调优方面都包括哪些工作，以linux为例，请简明阐述，并举一些参数为例

* 1)磁盘
 
禁用 atime 日志记录特性,atime 是最近访问文件的时间，每当访问文件时，底层文件系统必须记录这个时间戳。因为系统管理员很少使用 atime，禁用它可以减少磁盘访问时间。禁用这个特性的方法是，在 /etc/fstab 的第四列中添加 noatime 选项。

示例：
```
/dev/VolGroup00/LogVol00 /                      ext3    defaults,noatime        1 1
LABEL=/boot             /boot                   ext3    defaults,noatime        1 2
devpts                  /dev/pts                devpts  gid=5,mode=620  0 0
tmpfs                   /dev/shm                tmpfs   defaults        0 0
proc                    /proc                   proc    defaults        0 0
sysfs                   /sys                    sysfs   defaults        0 0
LABEL=SWAP-hdb2         swap                    swap    defaults        0 0
LABEL=SWAP-hda3         swap                    swap    defaults        0 0
```

为让这一修改生效，不需要重新引导；只需重新挂装每个文件系统。例如，为了重新挂装根文件系统，运行 mount / -o remount。

* 2)内核

将这些设置添加到/etc/sysctl.conf中，在下一次引导系统时，或者下一次运行 sysctl -p /etc/sysctl.conf 时，这些设置就会生效。
```
# Use TCP syncookies when needed
net.ipv4.tcp_syncookies = 1
设置启用 TCP SYN cookie。SYN cookie 特性可以识别出 SYN 泛滥（SYN flood） 的网络攻击，并使用一种优雅的方法保留队列中的空间。
sysctl -w net.ipv4.tcp_synack_retries=2   # syn-ack握手状态重试次数，默认5，遭受syn-flood攻击时改为1或2
sysctl -w net.ipv4.tcp_syn_retries=2      # 外向syn握手重试次数，默认4
# Enable TCP window scaling
net.ipv4.tcp_window_scaling: = 1
启用 TCP 窗口伸缩使客户机能够以更高的速度下载数据。TCP 允许在未从远程端收到确认的情况下发送多个数据包，默认设置是最多 64 KB，在与延迟比较大的远程客户机进行通信时这个设置可能不够。窗口伸缩会在头中启用更多的位，从而增加窗口大小。
# Increase TCP max buffer size
net.core.rmem_max = 16777216
net.core.wmem_max = 16777216
# Increase Linux autotuning TCP buffer limits
net.ipv4.tcp_rmem = 4096 87380 16777216 
net.ipv4.tcp_wmem = 4096 65536 16777216
以上四个配置项增加 TCP 发送和接收缓冲区。这使应用程序可以更快地丢掉它的数据，从而为另一个请求服务。还可以强化远程客户机在服务器繁忙时发送数据的能力。
# Increase number of ports available
net.ipv4.ip_local_port_range = 1024 65000
增加可用的本地端口数量，这样就增加了可以同时服务的最大连接数量。
```

* 3)系统

a.修改文件打开数

> /bin/echo ' *    -       nofile  65535' >>/etc/security/limits.conf

b.关闭SELINUX

> sed -i 's/SELINUX=enforcing/SELINUX=disabled/' /etc/selinux/config
> setenforce 0

c.ssh连接优化

> PermitEmptyPasswords no
> UseDNS no
> GSSAPIAuthentication no

d.精简开机自启动项目

```
/sbin/chkconfig --list|/bin/egrep -v "3:off|crond|sshd|network|rsyslog" |/bin/cut-f1 \
|/bin/awk '{print"chkconfig --level 345",$1,"off"}'| /bin/bash

/sbin/chkconfig|/bin/grep "3:on" &&\
/bin/echo -e "\033[36;5m startup services optimize successful\033[0m"
```

e.记录历史命令
```
sed -i "/^export PROMPT_COMMAND=.*/d" /root/.bash_profile

echo "export PROMPT_COMMAND='{ msg=\$(history 1 | { read x y; echo \$y; });user=\$(whoami); echo \$(date \"+%Y-%m-%d %H:%M:%S\"):\$user:\`pwd\`/:\$msg ---- \$(who am i); } >> /tmp/\`hostname\`.\`whoami\`.history-timestamp'" >> /root/.bash_profile
```



## 五、 监控

### 1.1 收集监控指标

需要监控一台服务器的哪些指标，如果是这台机器是web服务器还需要监控哪些项目。


* 基础指标

```
a.Load Avg
b.CPU使用率
CPU利用率：User Time <= 70%，System Time <= 35%，User Time + System Time <= 70%。
c.磁盘使用率
越低越好
d.带宽
出方向带宽，下行带宽
进方向带宽，上行带宽
e.http连接数
```

* web服务器需要监控的项目

以nginx为例

```
a.nginx占用CPU使用率，越低越好
b.nginx内存占用，越低越好
c.nginx启动端口的状态
d.nginx响应时间，可通过curl中的"%{time_total}"获取
e.nginx每秒HTTP请求数，越少越好
f.异常状态码(4xx/5xx)占比
g.下载速度监控,request size/request time
```


## 六、 线上发布

请简述下你们公司的发布流程。


> 测试->灰度->线上，具体的发布流程图，涉及到每个环节的技术要点
> 包括但不限于nginx&php，nginx&tomcat...



