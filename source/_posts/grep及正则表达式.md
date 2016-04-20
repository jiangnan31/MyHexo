title: grep及正则表达式
date: 2014-11-05 11:11:26
categories: 转载
tags: [Regex, Linux]
---

首先要记住的是: 正则表达式与通配符不一样,它们表示的含义并不相同!
正则表达式只是一种表示法,只要工具支持这种表示法， 那么该工具就可以处理正则表达式的字符串。vim、grep、awk 、sed 都支持正则表达式，也正是因为由于它们支持正则，才显得它们强大；

## 1. 基础正则表达式 
<!--more-->
grep -[acinv] '搜索内容串' filename 
-a 以文本文件方式搜索 
-c 计算找到的符合行的次数 
-i 忽略大小写 
-n 顺便输出行号 
-v 反向选择，即找 没有搜索字符串的行 
其中搜索串可以是正则表达式! 

### 1.1 搜索关键字 
搜索有the的行,并输出行号
```shell
[jiangnan04@cp01-rdqa-dev361.cp01.baidu.com document]$ grep -n 'the' crm-web.log
5:11/04 10:53:15 [scheduleFactory_Worker-9] INFO  utils.BNSUtils   .getUsedJobTrackerInfo:237 - the used jobtracker is {"serviceName":"melody-cal-bnsMonitor.MP3.91fzcm","hostName":"yf-mp3-melody-cal00.yf01","port":27017,"isUsed":1}
49:11/04 10:54:15 [scheduleFactory_Worker-9] INFO  utils.BNSUtils   .getUsedJobTrackerInfo:237 - the used jobtracker is {"serviceName":"melody-cal-bnsMonitor.MP3.91fzcm","hostName":"yf-mp3-melody-cal00.yf01","port":27017,"isUsed":1}
```
搜 索没有the的行,并输出行号 (-v反选)
```shell
$ grep -nv 'the' crm-web.log
```
### 1.2 利 用[]搜索集合字符 
[] 表示其中的某一个字符 ，例如[ade] 表示a或d或e 
```shell
[jiangnan04@cp01-rdqa-dev361.cp01.baidu.com document]$ grep -n 'bi[gt]log' crm-web.log

75:  11/04 10:54:21 [timing-1] INFO  service.OpenDataSourceService.run:349 - [biglog] datasource has no change
76:  11/04 10:54:21 [timing-1] INFO  service.OpenDataSourceService.run:349 - [bitlog] datasource has no change
```

<span style="color:red">可以用^符号做[]内的前缀，表示除[]内的字符之外的字符。</span> 
比如搜索oo前没有g的字符串所在的行. 使用 '[^g]oo' 作搜索字符串 
```shell
[jiangnan04@cp01-rdqa-dev361.cp01.baidu.com document]$ grep -n '[^g]oo' crm-web.log
41:  http://bns.noah.baidu.com/webfoot/index.php?r=Group/GroupInfo_v2&groupName=group.cal-melody.MP3.ai
78:  apple is my favorite food
79:  Football is not use feet only
80:  google is the best tools for search keyword
81:  goooooog test.
```

<span style="color:red">[] 内可以用范围表示</span>，比如[a-z] 表示小写字母,[0-9] 表示0~9的数字, [A-Z] 则是大写字母们。[a-zA-Z0-9]表示所有数字与英文字符。 当然也可以配合^来排除字符。 
搜索包含数字的行 
```shell
[jiangnan04@cp01-rdqa-dev361.cp01.baidu.com document]$ grep -n '[7-9]' crm-web.log 
```

<span style="color:red">行首与行尾字符 '^' \$</span>

^表示行的开头，\$表示行的结尾( 不是字符，是位置）那么‘^$' 就表示空行,因为只有 
行首和行尾。 
这里^与[]里面使用的^意义不同。它表示^后面的串是在行的开头。 
比如搜索  app在开头的行 
```shell
[jiangnan04@cp01-rdqa-dev361.cp01.baidu.com document]$ grep -n '^$' crm-web.log 
77:
[jiangnan04@cp01-rdqa-dev361.cp01.baidu.com document]$ grep -n '^  app' crm-web.log 
78:  apple is my favorite food
```

搜索以小写字母开头的行 
```shell
[jiangnan04@cp01-rdqa-dev361.cp01.baidu.com document]$ grep -n '^[a-z]' crm-web.log
78:apple is my favorite food
79:Xmeadgsadg
80:Football is not use feet only
81:google is the best tools for search keywords 
82:goooooog test.
```
<span style="color:red">不知道为什么会有79和80行。。。</sapn>

搜索开头不是数字的行 
```shell
[jiangnan04@cp01-rdqa-dev361.cp01.baidu.com document]$ grep -n '^[^0-9]' crm-web.log
52:current info:yf-mp3-melody-cal00.yf01:27017
53:BNS info:yf-mp3-melody-cal00.yf01:27017
76:apple is my favorite food
77:Xmeadgsadg
78:Football is not use feet only
79:google is the best tools for search keywords 
80:goooooog test.
```

$表示它前面的串是在行的结尾，比如 '\\.' 表示 . 在一行的结尾 
搜索末尾是.的行 
```shell
[jiangnan04@cp01-rdqa-dev361.cp01.baidu.com document]$ grep -n '\.$' crm-web.log //.
```
是正则表达式的特殊符号，所以要用\转义 
```shell
[jiangnan04@cp01-rdqa-dev361.cp01.baidu.com document]$ grep -n '\.$' crm-web.log //.
crm-web.log:78:Football is not use feet only.
crm-web.log:80:goooooog test.
```

注意在MS的系统下生成的文本文件，换行会加上一个 ^M 字符。所以最后的字符会是隐藏的^M ,在处理Windows 
下面的文本时要特别注意！ 
可以用cat dos_file | tr -d '\r' > unix_file 来删除^M符号。 ^M==\r 

那么'^$' 就表示只有行首行尾的空行拉！ 

```shell
#搜索空行 
[jiangnan04@cp01-rdqa-dev361.cp01.baidu.com document]$  grep -n '^$' crm-web.log
75:
#搜索非空行 
[jiangnan04@cp01-rdqa-dev361.cp01.baidu.com document]$  grep -vn '^$' crm-web.log
....
```

<span style="color:red">任意一个字符. 与重复字符 * </span>

<span style="color:red">在bash中*代表通配符，用来代表任意个 字符</sapn>，但是在正则表达式中，他含义不同，*表示有0个或多个 某个字符。 
例如 oo*, 表示第一个o一定存在，第二个o可以有一个或多个，也可以没有，因此代表至少一个o. 

点. 代表一个任意字符，必须存在。 g??d 可以用 'g..d' 表示。 good ,gxxd ,gabd .....都符合。 

```shell
[jiangnan04@cp01-rdqa-dev361.cp01.baidu.com document]$ grep -n 'bi..og' crm-web.log
73:11/04 10:54:21 [timing-1] INFO  service.OpenDataSourceService.run:349 - [biglog] datasource has no change
74:11/04 10:54:21 [timing-1] INFO  service.OpenDataSourceService.run:349 - [bitlog] datasource has no change
```

搜索两个o以上的字符串 (//前两个o一定存在，第三个o可没有，也可有多个。 )
```shell
[jiangnan04@cp01-rdqa-dev361.cp01.baidu.com document]$ grep -n 'ooo*' crm-web.log
39:http://bns.noah.baidu.com/webfoot/index.php?r=Group/GroupInfo_v2&groupName=group.cal-melody.MP3.ai
76:apple is my favorite food
78:Football is not use feet only.
79:google is the best tools for search keywords.
80:goooooog test.
[jiangnan04@cp01-rdqa-dev361.cp01.baidu.com document]$ grep -n 'oooo*' crm-web.log
80:goooooog test.
```

搜索g开头和结尾，中间是至少一个o的字符串，即gog, goog....gooog...等 
```shell
[jiangnan04@cp01-rdqa-dev361.cp01.baidu.com document]$ grep -n 'goo*g' crm-web.log
79:google is the best tools for search keywords.
80:goooooog test.
```

搜索g开头和结尾的字符串在的行 
```shell
[jiangnan04@cp01-rdqa-dev361.cp01.baidu.com document]$ grep -n 'g.*t' crm-web.log   
 // .*表示 0个或多个任意字符 
1:"Open Source" is a good mechanism to develop programs. 
14:The gd software is a library for drafting programs. 
18:google is the best tools for search keyword. 
19:goooooogle yes! 
20:go! go! Let's go. 
```

<span style="color:red">限定连续重复字符的范围 { } </sapn> 

. * 只能限制0个或多个， 如果要确切的限制字符重复数量，就用{范围} 。范围是数字用,隔开 2,5 表示2~5个, 
2表示2个，2, 表示2到更多个 
注意，由于{ }在SHELL中有特殊意义，因此作为正则表达式用的时候要用\转义一下。 

搜索包含两个o的字符串的行。 
```shell
[jiangnan04@cp01-rdqa-dev361.cp01.baidu.com document]$ grep -n 'o\{2\}' crm-web.log
39:http://bns.noah.baidu.com/webfoot/index.php?r=Group/GroupInfo_v2&groupName=group.cal-melody.MP3.ai
76:apple is my favorite food
78:Football is not use feet only.
79:google is the best tools for search keywords.
80:goooooog test.
[jiangnan04@cp01-rdqa-dev361.cp01.baidu.com document]$ grep -n 'o\{3\}' crm-web.log
80:goooooog test.
```

```shell
#搜索g后面跟2~5个o,后面再跟一个g的字符串的行。
[jiangnan04@cp01-rdqa-dev361.cp01.baidu.com document]$ grep -n 'go\{2,5\}g' crm-web.log
79:google is the best tools for search keywords.
#搜索包含g后面跟2个以上o,后面再跟g的行。。 
[jiangnan04@cp01-rdqa-dev361.cp01.baidu.com document]$ grep -n 'go\{2,\}g' crm-web.log
79:google is the best tools for search keywords.
80:goooooog test.
```

注意，相让[]中的^ － 不表现特殊意义，可以放在[]里面内容的后面。 
'[^a-z\.!^ -]' 表示没有小写字母，没有. 没有!, 没有空格，没有- 的 串，注意[]里面有个小空格。 

另外shell 里面的反向选择为[!range], 正则里面是 [^range] <span style="color:red">?</span>


## 2. 扩展正则表达式 

扩展正则表达式是对基础正则表达式添加了几个特殊构成的。 
它令某些操作更加方便。 
比如我们要去除 空白行和行首为数字的行， 会这样用： 
```shell
[jiangnan04@cp01-rdqa-dev361.cp01.baidu.com document]$ grep -v '^$' crm-web.log  | grep -v '^[0-9]'
{"flag":1,"jobtracker":[{"serviceName":"crm-cal-bnsSinglePointMonitor.MP3.91fzcm","hostName":"tc-mp3-melody-web01.tc","port":27017,"isUsed":0},{"serviceName":"melody-cal-bnsMonitor.MP3.91fzcm","hostName":"yf-mp3-melody-cal00.yf01","port":27017,"isUsed":1}]}
current info:yf-mp3-melody-cal00.yf01:27017
BNS info:yf-mp3-melody-cal00.yf01:27017
apple is my favorite food
Xmeadgsadg
Football is not use feet only.
google is the best tools for search keywords.
goooooog test.
# 管道继续过滤
[jiangnan04@cp01-rdqa-dev361.cp01.baidu.com document]$ grep -v '^$' crm-web.log  | grep -v '^[0-9]' | grep -n '^[a-z]'
2:http://bns.noah.baidu.com/webfoot/index.php?r=Group/GroupInfo_v2&groupName=group.cal-melody.MP3.ai
6:current info:yf-mp3-melody-cal00.yf01:27017
7:BNS info:yf-mp3-melody-cal00.yf01:27017
8:apple is my favorite food
9:Xmeadgsadg
10:Football is not use feet only.
11:google is the best tools for search keywords.
12:goooooog test.
```

然而使用支持扩展正则表达式的 egrep 与扩展特殊符号 | ，会方便许多。 
注意grep只支持基础表达式， 而egrep 支持扩展的， 其实 egrep 是 grep -E 的别名而已。因此grep -E 支持扩展正则。 
那么: 
```shell
[jiangnan04@cp01-rdqa-dev361.cp01.baidu.com document]$ egrep -v '^$|^[0-9]|^{' crm-web.log  
http://bns.noah.baidu.com/webfoot/index.php?r=Group/GroupInfo_v2&groupName=group.cal-melody.MP3.ai
current info:yf-mp3-melody-cal00.yf01:27017
BNS info:yf-mp3-melody-cal00.yf01:27017
apple is my favorite food
Xmeadgsadg
Football is not use feet only.
google is the best tools for search keywords.
goooooog test.
```
<span style="color:red">这里| 表示或的关系。 即满足 ^$ 、 ^# 或者 ^{ 的字符串。 </span>

这里列出几个扩展特殊符号： 
＋， 于 . * 作用类似，表示 一个或多个重复字符。 
?， 于 . * 作用类似，表示0个或一个字符。 
｜，表示或关系，比如 'gd|good|dog' 表示有gd,good或dog的串 
（），将部分内容合成一个单元组。 比如 要搜索 glad 或 good 可以这样 'g(la|oo)d' 
<span style="color:red">()的好处是可以对小组使用 + ? * 等。 </span>
比如要搜索A和C开头结尾，中间有至少一个(xyz) 的串，可以这样 : 'A(xyz)+C'


转载自：[http://www.jb51.net/article/31207.htm](http://www.jb51.net/article/31207.htm)
参考：[http://www.jb51.net/tools/zhengze.html](http://www.jb51.net/tools/zhengze.html)