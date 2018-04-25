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

原理：
解释一：
This is extended slice syntax. It works by doing [begin:end:step] 
-by leaving begin and end off and specifying a step of -1, it reverses a string.
解释二：
a[起始位(包含):结束位(不包含):差值]
[详见Stack Overflow](https://stackoverflow.com/questions/5846004/unable-to-reverse-lists-in-python-getting-nonetype-as-list)
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

### 4.  linux下比较两个目录下的文件
```
[root@test ~]# ls test1
1  2
[root@test ~]# ls test2
1
[root@test ~]# diff -r test1 test2
Only in test1: 2
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

 - 1
