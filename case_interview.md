# 面试题

## Linux命令相关

### 1.awk

#### 1.1现有两个文件a与b，内容分别为：
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

### 2.字符串反转

请使用任一语言对”abcd”字符串进行反转，输出”dcba”

#### 2.1 python

##### 2.1.1 使用字符串内置方法，设置步长为1
```
s='abcd'
print s[::-1]
dcba

原理是：This is extended slice syntax. It works by doing [begin:end:step] - by leaving begin and end off and specifying a step of -1, it reverses a string.
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
s=’abcd’
l = list(s)
print '' .join(x for x in l)
dcba
```

#### 2.2 shell
```
# echo "abcd" |awk -F "" '{for(i=NF;i>1;i--)printf("%s",$i);print $1}'
dcba
```
