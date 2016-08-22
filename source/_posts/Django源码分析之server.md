---
title: Django源码分析之server
---

### 乍见
Django内置的server基本包括两部分：django.core.servers和django.core.handlers
![][image-1]


### 相识
servers.basehttp是Django自身提供的一个用于开发测试的server模块，其中提供的WSGIServer、ServerHandler、WSGIRequestHandler其实都是属于WSGI server，django只不过是对python内置的WSGI模块simple\_server做的一层包装。

handlers package包括base.py和wsgi.py两个模块。
base.py中只定义了一个BaseHandler类，它复杂一些基础的功能，比如加载中间件，处理异常，获取响应数据等

wsgi.py才是主角，其中最重要的就是WSGIHandler类，它继承了base.py中的BaseHandler，只添加了一个\_\_call\_\_方法，那为什么添加这个方法呢？

django.core.wsgi

	import django
	from django.core.handlers.wsgi import WSGIHandler
	
	
	def get_wsgi_application():
	    """
	    The public interface to Django's WSGI support. Should return a WSGI
	    callable.
	
	    Allows us to avoid making django.core.handlers.WSGIHandler public API, in
	    case the internal WSGI implementation changes or moves in the future.
	    """
	    django.setup()
	    return WSGIHandler()

可见我们启动server时传入的application其实是WSGIHandler的一个实例，而根据WSGI规范，这个application必须要是callable，python中的callable包括函数、方法以及任何定义了\_\_call\_\_方法的对象

回到handlers.wsgi

	class WSGIHandler(base.BaseHandler):
	    initLock = Lock()
	    request_class = WSGIRequest
	
	    def __call__(self, environ, start_response):
	        # Set up middleware if needed. We couldn't do this earlier, because
	        # settings weren't available.
	        if self._request_middleware is None:
	            with self.initLock:
	                try:
	                    # Check that middleware is still uninitialized.
	                    if self._request_middleware is None:
	                        self.load_middleware()
	                except:
	                    # Unload whatever middleware we got
	                    self._request_middleware = None
	                    raise
	
	        set_script_prefix(get_script_name(environ))
	        signals.request_started.send(sender=self.__class__, environ=environ)
	        try:
	            request = self.request_class(environ)
	        except UnicodeDecodeError:
	            logger.warning('Bad Request (UnicodeDecodeError)',
	                exc_info=sys.exc_info(),
	                extra={
	                    'status_code': 400,
	                }
	            )
	            response = http.HttpResponseBadRequest()
	        else:
	            response = self.get_response(request)
	
	        response._handler_class = self.__class__
	
	        status = '%s %s' % (response.status_code, response.reason_phrase)
	        response_headers = [(str(k), str(v)) for k, v in response.items()]
	        for c in response.cookies.values():
	            response_headers.append((str('Set-Cookie'), str(c.output(header=''))))
	        start_response(force_str(status), response_headers)
	        if getattr(response, 'file_to_stream', None) is not None and environ.get('wsgi.file_wrapper'):
	            response = environ['wsgi.file_wrapper'](response.file_to_stream)
	        return response

由于\_\_call\_\_是一个请求的入口，它需要调用BaseHandler中定义的方法去执行加载中间件等一系列操作和异常处理，除此之外，WSGIHandler还会处理cookie、触发signal等。

至于如何返回响应，具体可看handlers.base模块，大概就是找出请求的path，通过匹配路由，找到并执行用户定义的view方法，执行中间件处理方法，最后返回响应。当然，还有一系列的异常处理。

### 回想

这部分的内容需要对WSGi协议有个大致的了解，可以参考：[http://xiaorui.cc/2016/04/16/%E6%89%93%E9%80%A0mvc%E6%A1%86%E6%9E%B6%E4%B9%8Bwsgi%E5%8D%8F%E8%AE%AE%E7%9A%84%E4%BC%98%E7%BC%BA%E7%82%B9%E5%8F%8A%E6%8E%A5%E5%8F%A3%E5%AE%9E%E7%8E%B0/][1]





[1]:	http://xiaorui.cc/2016/04/16/%E6%89%93%E9%80%A0mvc%E6%A1%86%E6%9E%B6%E4%B9%8Bwsgi%E5%8D%8F%E8%AE%AE%E7%9A%84%E4%BC%98%E7%BC%BA%E7%82%B9%E5%8F%8A%E6%8E%A5%E5%8F%A3%E5%AE%9E%E7%8E%B0/

[image-1]:	/images/django_server.png