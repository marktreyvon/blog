---

layout: post

title: "网安之脚本小子笔记"

author: "markt"

---

# 网安之脚本小子笔记

### 根据URL判断静动态、什么系统：

一般：	后缀添加index.html不报错则为静态（以.html结尾的网页）
		后缀中有asp jsp php aspx 的为动态，也可据此判断出脚本语言
linux对大小写敏感，windows不敏感，所以可以改变某一字母来判断其使用的系统

## 搭建网站：

?	1.利用服务器系统内置软件设置搭建；
	2.利用小旋风aspweb搭建ASP+Access轻量级网站；
	3.利用APMServ搭建PHP+MySQL网站。

## linux：

?	关注内核版本号：主版本号.次版本号.补丁次数  次版本号奇数表示开发版，不同的内存在不同的内核漏洞可以利用；
	磁盘分区表示；
	目录结构：
		bin	用户命令
		sbin 	管理员命令	
		boot	引导程序
		dev	存储程序
		etc	配置文件
		home	用户目录，类似window用户目录
		lib	函数库
		media mnt	光盘等外来存储介质
		opt	外围大型程序
		proc	开机生成进程信息
		selinux	控制程序？
		sys	系统配置
		usr	外部程序
		var	日志、网站根目录

## linux 常用命令：

内部命令：
		命令字 [选项][参数]
辅助操作：
	Tab	自动补齐
	\	强制换行
Ctrl+：	U	清空至行首
	K	至行尾
	L	清屏
	C	取消本次命令
ls		查看目录
ifconfig	查看IP
uname		查看系统信息	-r	内核版本号
hostname	查看主机名
cat /proc/cpuinfo 查看cpu信息
cat /proc/meminfo 查看内存信息
cat		查看文本内容
halt  shutdown  poweroff 关机
reboot 		重启
cd 		切换工作目录
pwd		查看工作目录
du		统计目录、文件大小
mkdir		创建目录
touch		创建文件
ln		创建连接（快捷方式）
cp		复制
rm		删除
mv 		移动
find		查找文件
chmod 766 readme.txt 	其中，0 表示没有权限；1表示可执行权限；2表示写权限；4表示读权限； 
那么766 即表示把这个文件设置为创建者拥有所有权限，而同组用户与其他用户只拥有读写权限。
wc		统计文件信息（行数、单词数、字符数）
解压压缩、安装NPM（红帽系统下软件包）略
useradd		添加用户
route		查看路由
netstat		查看网络连接情况

## vi命令：

?	vi [option][+n] [file]

### 三种模式：

?	命令模式	默认 按Esc返回命令模式
	文本模式	
			a/A/i/I/o/O 在光标后、行末、光标前、行首、光标下新行、光标上新行
	底行模式	:

### 命令：

[n]dd	剪切n行
p		粘贴
u		撤回
.		重复前一命令

### 底行：

set nu 	显示行号
q		退出
!		强制执行
w		保存
/ ? +string	向下/向上搜索
n		调到第n行

## 网卡类型：

eth0	以太网
lo	虚拟回环设备
fddi0	光纤
tr	令牌环
ppp0	PPP协议的串口设备

## 渗透信息收集：

?	dns收集、敏感目录、端口探测、Google Hack、子域探测、旁站探测、C段查询、整站识别、waf探测、工具网站

DNS搜集：		IP、whois、站长工具、netcraft
IP查询：			站长之家、netcraft、kali下——dnsenum、dnswalk、dig、lbd
敏感目录收集：	后台、上传、管理目录等——burpsuit、御剑

判断网站CMS类型：	脚本语言、操作系统、搭建平台、CMS厂商
工具：			wvs、wwwscan、站长工具、whatweb、Google Hack

爆库：
	知道数据库位置后直接下载下来得到管理员账号信息
	技巧：管理员偷懒改数据后缀名mdb为asp等，下载后改回来就可以

网站后台查找：
	利用弱口令默认后台；admin、login、manage
	根据CMS类型、网站默认链接
	利用工具
	robots.txt
	google hack
	网站编辑器
管理员密码破解：(MD5)
	hydra
	PKAV HTTP Fuzzer 有验证码时
	Discuz
网站漏洞利用：
	百度CMS具体版本漏洞

## SQL注入：

 判断数据库类型：
	添加'根据报错信息；
	特定支持的函数；
	特定的系统表；
	辅助符号： version
需要判断：
	注入点字段长度  通过orderby+数字猜解
	管理员账户表名，字段名	穷举
	可注入字段位置	select 1,2,…… from
	具体数据长度	函数len
	具体数据内容	通过判断ascII码函数asc()

### Access注入：

	后缀名：.mdb
	打开工具：辅臣、破障
	找有传入值的URL
	判断数据库类型：
 偏移注入：猜到表名没猜到列名的情况	
	通过套公式随机得到表中数据
实例：https://www.cnblogs.com/xishaonian/p/6054320.html

### mssql注入：

	端口1433
	通过差异备份上传一句话木马

### MySQL注入：

	5.0利用information_schema表（需要转换进制），4.0利用sqlmap
	读取函数load_file():
		使用//或者\，只能绝对路径
		如果win2003+IIS可以先搜索c:/.../inetsrv/metabase.xml
	写入函数 outer file()

### oracle注入：

	类似access，判断表名，列名，根据ascii码判断数据字符

### 三种提交方式的注入：get、post、cookie

	get：通常访问页面都是
	post：提交表单
	工具 pangolin、sqlmap
 post：找提交到的URL，利用工具
 cookie：可以理解为将原本注入的参数由普通的get/post方式提交改变为转换成cookie再提交给网站，相当于多了一个打包的过程

### 参数注入：数字型、字符型、搜索型

	数字、字符都是普通的注入：' and 1=1 #
	搜索型注入：用到通配符： %' and 1=1 and '#'='
		主要是搜索框
		工具：burp suite  sqlmap
		先burp抓包，然后利用sqlmap分析

### 伪静态注入

	去掉后缀html 直接写
	or 利用url信息自行构造可以注入的url

