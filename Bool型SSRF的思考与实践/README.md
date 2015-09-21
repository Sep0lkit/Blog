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

#### 0x02 两者之间的区别
