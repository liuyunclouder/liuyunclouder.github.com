---
title: Django源码分析之权限系统\_ 擒贼先擒王
---

### 乍见
Django内置的权限系统已经很完善了，加上django-guardian提供的功能，基本上能满足大部分的权限需求。暂且不说django-guardian，我们先来看下Django内置的权限系统：django.contrib.auth 包。
![][image-1]


### 相识
一般权限系统分为全局权限和对象权限。Django只提供了一个对象权限的框架，具体实现由第三方库django-gardian完成。我们只看全局权限。

先来看auth包暴露出哪些接口。

django.contrib.auth.\_\_init\_\_.py

	def load_backend(path):
	    return import_string(path)()
	
	
	def _get_backends(return_tuples=False):
	    backends = []
	    for backend_path in settings.AUTHENTICATION_BACKENDS:
	        backend = load_backend(backend_path)
	        backends.append((backend, backend_path) if return_tuples else backend)
	    if not backends:
	        raise ImproperlyConfigured(
	            'No authentication backends have been defined. Does '
	            'AUTHENTICATION_BACKENDS contain anything?'
	        )
	    return backends
	
	
	def get_backends():
	    return _get_backends(return_tuples=False)

前三个方法都是为了加载backends。一个backend其实就是一个class，必须实现authenticate和get\_user两个方法。每当我们这样验证用户时

	authenticate(username='username', password='password')

django就会去调用这些backend class，用其提供的方法去验证用户权限。那django是如何知道要调用哪些backend class呢？答案就在settings.py中，默认为

	AUTHENTICATION_BACKENDS = ['django.contrib.auth.backends.ModelBackend']


那Django是如何调用这些backend class的呢？

	def authenticate(**credentials):
	    """
	    If the given credentials are valid, return a User object.
	    """
	    for backend, backend_path in _get_backends(return_tuples=True):
	        try:
	            inspect.getcallargs(backend.authenticate, **credentials)
	        except TypeError:
	            # This backend doesn't accept these credentials as arguments. Try the next one.
	            continue
	
	        try:
	            user = backend.authenticate(**credentials)
	        except PermissionDenied:
	            # This backend says to stop in our tracks - this user should not be allowed in at all.
	            return None
	        if user is None:
	            continue
	        # Annotate the user object with the path of the backend.
	        user.backend = backend_path
	        return user
	
	    # The credentials supplied are invalid to all backends, fire signal
	    user_login_failed.send(sender=__name__,
	            credentials=_clean_credentials(credentials))

由此可见，Django会在第一个验证正确的backend class调用完成后停止，或者碰到PermissionDenied异常也会停止，所以backend class的顺序也很重要。可以添加自定义的backend class。

	def login(request, user):
	    """
	    Persist a user id and a backend in the request. This way a user doesn't
	    have to reauthenticate on every request. Note that data set during
	    the anonymous session is retained when the user logs in.
	    """
	    session_auth_hash = ''
	    if user is None:
	        user = request.user
	    if hasattr(user, 'get_session_auth_hash'):
	        session_auth_hash = user.get_session_auth_hash()
	
	    if SESSION_KEY in request.session:
	        if _get_user_session_key(request) != user.pk or (
	                session_auth_hash and
	                request.session.get(HASH_SESSION_KEY) != session_auth_hash):
	            # To avoid reusing another user's session, create a new, empty
	            # session if the existing session corresponds to a different
	            # authenticated user.
	            request.session.flush()
	    else:
	        request.session.cycle_key()
	    request.session[SESSION_KEY] = user._meta.pk.value_to_string(user)
	    request.session[BACKEND_SESSION_KEY] = user.backend
	    request.session[HASH_SESSION_KEY] = session_auth_hash
	    if hasattr(request, 'user'):
	        request.user = user
	    rotate_token(request)
	    user_logged_in.send(sender=user.__class__, request=request, user=user)

login方法，顾名思义，登录用户，同时设置好session，最后发送登入成功通知

	def logout(request):
	    """
	    Removes the authenticated user's ID from the request and flushes their
	    session data.
	    """
	    # Dispatch the signal before the user is logged out so the receivers have a
	    # chance to find out *who* logged out.
	    user = getattr(request, 'user', None)
	    if hasattr(user, 'is_authenticated') and not user.is_authenticated():
	        user = None
	    user_logged_out.send(sender=user.__class__, request=request, user=user)
	
	    # remember language choice saved to session
	    language = request.session.get(LANGUAGE_SESSION_KEY)
	
	    request.session.flush()
	
	    if language is not None:
	        request.session[LANGUAGE_SESSION_KEY] = language
	
	    if hasattr(request, 'user'):
	        from django.contrib.auth.models import AnonymousUser
	        request.user = AnonymousUser()

相对的，logout方法，负责登出用户，清理session，最后设置当前用户为匿名用户

	def get_user_model():
	    """
	    Returns the User model that is active in this project.
	    """
	    try:
	        return django_apps.get_model(settings.AUTH_USER_MODEL)
	    except ValueError:
	        raise ImproperlyConfigured("AUTH_USER_MODEL must be of the form 'app_label.model_name'")
	    except LookupError:
	        raise ImproperlyConfigured(
	            "AUTH_USER_MODEL refers to model '%s' that has not been installed" % settings.AUTH_USER_MODEL
	        )

Django不推荐直接使用User class，而是通知get\_user\_model方法获取当前的用户class（或者使用settins.AUTH\_USER\_MODEL）。这是为了防止因为开发者使用了自定义用户class而导致的信息错误。


	def update_session_auth_hash(request, user):
	    """
	    Updating a user's password logs out all sessions for the user if
	    django.contrib.auth.middleware.SessionAuthenticationMiddleware is enabled.
	
	    This function takes the current request and the updated user object from
	    which the new session hash will be derived and updates the session hash
	    appropriately to prevent a password change from logging out the session
	    from which the password was changed.
	    """
	    if hasattr(user, 'get_session_auth_hash') and request.user == user:
	        request.session[HASH_SESSION_KEY] = user.get_session_auth_hash()

最后这个方法的使用场景很少。一般我们更新用户密码时，会在session中清除用户登录信息，导致用户需要重新登录。而使用update\_session\_auth\_hash我们就可以在更新用户密码的同时更新用户的session信息，这样，用户就不需要重新登录了。


### 回想

擒贼先擒王，以上都是django.contrib.auth包中的\_\_init\_\_.py入口文件中的内容，背后还有很多“能工巧匠”，否则怎么支撑起auth整套权限系统？后续文章会一一介绍。




_

[image-1]:	/images/django_auth_init.png