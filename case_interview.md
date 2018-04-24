# 面试题

## Linux命令相关
### awk
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
