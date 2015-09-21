### Bool型SSRF的思考与实践

	0x00 Bool型SSRF 
	0x01 SSRF利用的基本思路
	0x02 两者之间的区别
	0x03 Bool型SSRF利用方法
	0x04 Struts2在Bool型SSRF中的利用
	0x05 Other(想到什么写什么)

#### 0x00 Bool型SSRF
什么是Bool型SSRF, 没听说过. 其实我也没有听说过. 只是我也不知道该怎么描述就起了这样一个名称.

Bool型SSRF:简单来说就是仅返回True 或 False的SSRF. 就以我前两天我挖掘的一个搜狐SSRF为例, 只有服务器端正确响应HTTP请求并且只有响应码为200的时候，返回Success，其余全部返回Failed. 这就是一个典型的Bool型SSRF.

#### 0x01 SSRF利用的基本思路
Wooyun上有很多SSRF典型的案例, 可以说让人拍案惊奇. 但是没有一个关于BOOL型SSRF的利用案例(可能我没看到吧). 这次挖掘到一个搜狐的Bool型SSRF, 对于我这个只是简单理解SSRF原理没有任何实战的渣渣来说, 难度还真大. 不过想想,以前没有接触过SSRF,这次就把你玩透. 翻阅学习Wooyun上的案例, 了解SSRF利用的基本思路:

	内网探测->应用识别->攻击Payload->Payload Result

	1. 内网探测: 内网主机信息收集
	2. 应用识别: 主机应用识别(可以通过Barner和应用指纹进行识别)
	3. 攻击Payload: 根据应用识别的应用,加载不同的攻击Payload(最常用莫属于Struts2)
	4. Payload Result: 返回相应Payload的执行信息

#### 0x02 两者之间的区别
