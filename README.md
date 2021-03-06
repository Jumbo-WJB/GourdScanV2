# Gourdscan

## Changelog 

### v2.1
   
* Web界面改进，统一控制所有代理和扫描线程，统一管理各种参数，首页刷新时间自定义。   
* 新增登录页以及session控制。   
* 线程控制，sqlmap选项，及各种扫描方式自定义。   
* waiting, finished, vulnerable, running等列表的展示及一键清空。   
* 各种代理模块自定义设置，并且可以直接在web界面启动，无需另开窗口。   
* redis.conf中加入了Daemon为True，可后台运行，无需另开窗口，提供redis中的各个删除接口，无需再连接redis操作。   
* 可以查看每个数据包及其详细漏洞payload，config中可以自定义后缀黑名单。   
* 所有规则重定义，并且支持自定义规则。   


## 安装依赖：

### Linux

1. 安装系统依赖

* apt-get install redis-server

2. 安转 python 类库

**基础模块**

`
$ pip install -r requirements.txt
`

**其他模块**

`
$ wget https://github.com/dugsong/libdnet/archive/master.zip && unzip master.zip 
`

`
$ wget http://dfn.dl.sourceforge.net/sourceforge/pylibpcap/pylibpcap-0.6.4.tar.gz && tar zxf pylibpcap-0.6.4.tar.gz
`

### Windows

 _TODO_

### OSX

 _TODO_


## 使用方法：

```
$ python cli.py
```
或

```
$ gourdscan
```
**conf.json**

> 默认平台用户名密码为：admin:Y3rc_admin
> 默认redis密码为：Y3rc_Alw4ys_B3_W1th_Y0u
> 如果有勾选sqlmap api scan选项，请在服务器上开启sqlmap api。


## 关于扫描规则：

1.每一个rule文件(除了sqlmap rule)都是由1个到N个level的rule构成，在config中的Scan_level可以设置，如果level小于等于Scan_level，即会调用该rule，请将误报率较高的rule之等级相应提高，Scan_level的设
    置可以设置到适合的等级，其中正则型模块的规则中，敏感字符必须转义，比如:, [], ()等字符！

2.sqlireflect rule: 
    正则型模块，所有规则中的正则敏感字符都要转义，可以通过re模块搜索出来的sql注入，包括报错注入和union注入
    (union注入一般不需要检测，而且检测比较麻烦，规则中是作为语法放在10级，存在一定的不准确性，一般不建议使
    用，或者请修改后使用)，requests中是payload，responses中是使用re模块搜索的关键字，每个requests
    的结果都会在该couple的responses中匹配每一行关键字，由于requests中可能存在空格，而且需要保持合适
    的格式(每条规则会通过String.strip()函数去掉空格和换行)，所以可以使用%20代替空格，xml中使用&lt;&gt;
    代替<>。

3.sqlitime rule: 
    时间盲注规则，所有跟延时数字有关的用TIME_VAR代替，只需要写入规则，时间判断则由系统控制，首先延时5秒，
    如果发现延时大于4.5秒，再延时10秒，如果延时大于9.5秒而且第二次延时大约是第一次延时的两倍，则判断存在注入。

4.sqlibool rule: 
    布尔型盲注规则，前两个规则都是True判断，后两个规则都是False判断，在扫描时，如果发现第一个和第三个判断出
    现了不同的结果后，会引入第二个和第四个判断进行深度判断，防止误报，规则无需转义，无需编码。

5.xss rule: 
    非正则型模块，通过"string in string"方式探测，无需转义，该规则为反射型xss规则，将参数用规则替代，并且
    不会测试post的参数，如果在response里面发现了一模一样的字符串，则判断为存在漏洞，第四级的rule误报可能比
    较高，请谨慎使用。

6.xpath rule: 
    xpath注入规则，默认只有一个等级，加上payload，如果在response里面发现了任意一条指定的匹配，则触发警告。

7.ldap rule: ldap注入规则，默认只有一个等级，同上。

8.lfi rule: 正则型模块，该规则为lfi规则，默认只有一个等级，同上。

9.sqlmap rule: sqlmap规则，目前gourdscan默认规则是sqlmap默认规则，其中有很多常见的选择，比如risk,
    level，tamper，可以自行设置，url，database，data，taskid等参数不允许设置，在每次添加规则后，会将所有
    设置update到每个任务上去, 但是最后无论多少个类型的漏洞，只会显示一个payload。

10.默认在每种类型规则中一旦发现匹配就返回，所以只会有一个message，特别是遇到时间盲注，速度会更快点，如果
    想更改，可以把设置中的“Only_get_one_rule_match”改为False


## 关于测试及数据：

1. 线程默认20个，扫描线程比较消耗资源，如果主机非服务器或者配置低，不建议开太多线程，从速度和资源占用的角度
    出发，我们更建议使用sqlmap api，而停掉内置的sql扫描 :) ，当然，非常欢迎各位黑阔向我们贡献代码和以及更加科学
    的扫描规则。

2. 数据包扫描独立于扫描器设置中的线程，所以可能会出现实际扫描线程多于设置的情况，如果某packet没有query也没
    有post data，则扫描直接跳过。

3. 经过测试，除sqlmap api以外规则全部开启的情况下，scan_level设置为3，如果没有任何漏洞，每个参数测试完成
    时间在4-5分钟左右，如果使用sqlmap api而不使用内置规则，每个参数测试时间在2-3分钟左右。

4. redis中存储的数据包结构：request={'headers':headers,'host':host,'url':url,'method':method,
    'postdata':postdata,'hash':url_hash,'uri':uri} json格式存储全数据包和uri(是否https)，以及其漏
    洞等级，漏洞警示。最后整体base64编码。

5. 代理黑名单、代理白名单、后缀名黑名单均可在config中设置，注意：每个域名、后缀用英文逗号隔开，不可以有空格。
    由于proxy中是通过str.endswith()判断，所以可以写入一级域名表示所有二级域名。

6. proxy_io.py相对于mix_proxy来说，更加稳定，所以是stable proxy，同时在对于http请求上速度也快得多，
    但是由于调用了tornado web模块，在多线程模式下无法stop(除非将web平台也同时stop)，多进程模式下无法执行，所以
    目前只能实现伪关闭，关闭后无法执行代理的功能，这种状态下也占用不了多少资源，如果想真正关闭，请重启平台。多线
    程模式下，ctrl+c可能无法关闭平台，需要ctrl+z，然后ps -a|grep python ，关闭start.py的pid。


## 注意选项：

1.登录后请立即修改用户名密码，并且请严格按照格式定义config中的选项，后台只对输入做了安全过滤，没有检查
    number是否合法，如果非要把number改成string造成扫描出错等情况，就不是平台的问题了。

2.如果同时开启扫描及抓包模块，确实抓包界面会刷新很快，由于其参数名未变，会被各个抓包模块过滤掉，所以如果使用自带扫描基本不会被记录入redis，但是，sqlmap在扫描的中会自己添加一个参数测试，这样就被抓包模块捕捉到了，形成恶性循环，故sqlmap和scapy抓包不能同时使用！！！

3.如果发现你的包输出了但是没有push到waiting或者running队列中，基本说明你的包重复了，此时如果没有什么特别重
    要的事情，可以点击任何一个抓包模块中的flush db按钮，删除整个redis中的队列，之后就会重新记录到redis中。

4.建议使用sqlmap的同时不要使用内置sql注入扫描模块！

5.如果出现了配置文件的问题，一般都是输入了非法字符等，这种错误对安全没有影响，只是会导致平台出错关闭。出现该
    错误后可以到github上重新wget一个下来改改使用，或者做个备份。


## Docker

**创建镜像** 

`
$ sudo docker build -t gourdscan:2.1 .  
`

**创建容器**

`
$ docker run -d -p 10000:22 c42a228eea05 /usr/sbin/sshd -D
`
**登陆服务器**

`
$ ssh root@localhost -p 10000
`

_用户名: `root`，密码: `Y3rc_admin`_


## QA

* 问：测试https的时候，即使信任了站点，只能出来首页的文字 
> 即使信任了某个站点，也不意味着信任了其子域名或其他域名，如果该站点从其他站点加载https数据，则需要将
> 其一样信任，chrome下具体方式是打开浏览器开发者选项(f12或command+i)，点击"network"，重新刷新页面，
> 双击红色的url，并信任该域，再次刷新重复上述步骤，直到可以完全加载出来。

* 问：引入scapy module的时候显示缺少libdnet 
> 在所需环境中有libdnet及其依赖的安装方式

* 问: Windows安装模块的时候提示“python version 2.7 is required, but not found”  
> python 64位版本在注册表中的信息与32位版本的不一样，所以在注册表里面找不到python的位置，谷歌搜
> 索下”python version 2.7 is required“就出来了。

* 问：Linux下启动redis提示“Bad directive or wrong number of arguments” 
> redis版本太低，建议官网下载3.2以上版本再使用本conf文件，或者不使用提供这里的conf文件。


## 其他选项：

1.只允许同时有200个session(session_size=6600)存在，为了方便，session过期时间为300天。不过以上设置均可以在config里面自行设置。

2.默认数据页中每页100个结果，可以在config中修改。

3.本系统开放源代码，尚有各种不足，欢迎各位提交pull request。
