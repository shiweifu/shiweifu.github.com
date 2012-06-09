Type: Blog Post (Markdown)
Blog: Exception
Link: http://shiweifu.sinaapp.com/?p=30
Post: 30
Title: 利用Python的动态特性，重构代码
Slug: %e5%88%a9%e7%94%a8python%e7%9a%84%e5%8a%a8%e6%80%81%e7%89%b9%e6%80%a7%ef%bc%8c%e9%87%8d%e6%9e%84%e4%bb%a3%e7%a0%81
Postformat: standard
Status: publish
Date: 2012-05-13 15:49:35 +0800
Pings: On
Comments: On
Category: 未分类

使用Python 的动态特性，可以优化掉很多冗余的代码。
编写oauth程序的时候，会遇到这种情况：

*GET /blocks/ids	获取用户黑名单id列表      *
*GET /blocks/blocking	获取黑名单上用户资料  *
*POST /blocks/create	把指定id用户加入黑名单*   
*GET /blocks/exists	检查用户是否被加入了黑名单 *  
*POST /blocks/destroy	将指定id用户解除黑名单*

这些都是饭否开放的REST接口，用户通过调用这些接口，来实现对应的功能，然后我写出了类似这样的代码：

<pre lang="Python" colla="-"> 
class UrlToken(object):
    """docstring for UrlToken"""
    url = ""
    method = -1
    must_has = "status"
    def __init__(self, _url, _method):
        super(UrlToken, self).__init__()
        self.url = _url
        self.method = _method
		
class FanfouStatusHandle(FanfouHandle):
    """docstring for FanfouUsersHandle"""        

    def __init__(self, _account):
        super(FanfouStatusHandle2, self).__init__()
        self.account = _account
        self.headers = self.account.get_headers()
        self.consumer_token  = self.account.get_consumer_token()
    

    def update(self, status):
		""" update user state"""
		tok = UrlToken("POST", "/status/update.json")
		_http_call(tok)

    def destroy(self, msg_id = None):
        """ destroy fanfou status by msg_id """
		tok = UrlToken("POST", "/status/destroy.json")
		_http_call(tok)
		
		
class FanfouAccountHandle(FanfouHandle):
    def __init__(self, _account):
        super(FanfouStatusHandle2, self).__init__()
        self.account = _account
        self.headers = self.account.get_headers()
        self.consumer_token  = self.account.get_consumer_token()

	def verify_credentials(self):
		""" verify_credentials """
		tok = UrlToken("GET", "/account/verify_credentials.json")
		_http_call(tok)
</pre>

这么写，用户调用很方便，每个接口模块写成一个类，再写个lib类，把这些单独的类再都实现一遍，然后调用就成了：
lib.statuses.update(status="hello lib")
用起来还算舒服，但看着一坨一坨的重复代码，太ugly了。

因为之前写的大多是C语言的程序，所以对反射这种高级货不太了解，基本上对这些动态特性，停留在“知道”。
知道有hasattr,getattr这些用于自省的特性，但从来没用过，那就拿这个程序下手。

其实调用最终都是调到_http_call这里去，不同的是：
1、函数名
2、UrlToken

可以通过重载__getattr__来获得要调用的函数名，然后在里面写个wrapper，用闭包来做不同函数的调用。
<pre lang="Python" colla="-"> 

class Handle:
	def __init__(self, _toks):
		#toks是个字典结构，里面以kv的形式存储了需要的UrlToken
		self.toks = _toks
		
	def __getattr__(self, attr):
		if self.toks.has_key(attr) == False:
			raise NotImplemetedError
		tok = self.toks[attr]
		
		def	wrapper(**kw):
			return _http_call(tok,kw)
</pre>

这么写之后，只需要构建一个大的UrlToken集合，里面有不同的地址和调用方式再封装个FanfouLib的类就可以了。

具体的代码，请参看项目。
地址：https://github.com/shiweifu/fanfoulib
