---
title: Django源码分析之执行入口
---

### 魔法门
一般我们启动django，最简单的方法是进入project 目录，这时目录结构是这样的

![][image-1]

然后我们执行python manage.py runserver，程序就开始执行了。

那django是如何从一个命令就启动整个server，启动的流程是如何的？

### 踏门而入

打开目录下的manage.py，内容是这样的：

	#!/usr/bin/env python
	import os
	import sys
	
	if __name__ == "__main__":
	    os.environ.setdefault("DJANGO_SETTINGS_MODULE", "django_learning.settings")
	
	    from django.core.management import execute_from_command_line
	
	    execute_from_command_line(sys.argv)

看来manage.py只是把命令行参数传给django.core.management模块中的execute\_from\_command\_line 函数。

查看execute\_from\_command\_line函数，可以发现实际执行的是ManagementUtility类的excute方法：

	def execute(self):
	        """
	        Given the command-line arguments, this figures out which subcommand is
	        being run, creates a parser appropriate to that command, and runs it.
	        """
	        try:
	            subcommand = self.argv[1]
	        except IndexError:
	            subcommand = 'help'  # Display help if no arguments were given.
	
	        # Preprocess options to extract --settings and --pythonpath.
	        # These options could affect the commands that are available, so they
	        # must be processed early.
	        parser = CommandParser(None, usage="%(prog)s subcommand [options] [args]", add_help=False)
	        parser.add_argument('--settings')
	        parser.add_argument('--pythonpath')
	        parser.add_argument('args', nargs='*')  # catch-all
	        try:
	            options, args = parser.parse_known_args(self.argv[2:])
	            handle_default_options(options)
	        except CommandError:
	            pass  # Ignore any option errors at this point.
	
	        no_settings_commands = [
	            'help', 'version', '--help', '--version', '-h',
	            'compilemessages', 'makemessages',
	            'startapp', 'startproject',
	        ]
	
	        try:
	            settings.INSTALLED_APPS
	        except ImproperlyConfigured as exc:
	            self.settings_exception = exc
	            # A handful of built-in management commands work without settings.
	            # Load the default settings -- where INSTALLED_APPS is empty.
	            if subcommand in no_settings_commands:
	                settings.configure()
	
	        if settings.configured:
	            # Start the auto-reloading dev server even if the code is broken.
	            # The hardcoded condition is a code smell but we can't rely on a
	            # flag on the command class because we haven't located it yet.
	            if subcommand == 'runserver' and '--noreload' not in self.argv:
	                try:
	                    autoreload.check_errors(django.setup)()
	                except Exception:
	                    # The exception will be raised later in the child process
	                    # started by the autoreloader. Pretend it didn't happen by
	                    # loading an empty list of applications.
	                    apps.all_models = defaultdict(OrderedDict)
	                    apps.app_configs = OrderedDict()
	                    apps.apps_ready = apps.models_ready = apps.ready = True
	
	            # In all other cases, django.setup() is required to succeed.
	            else:
	                django.setup()
	
	        self.autocomplete()
	
	        if subcommand == 'help':
	            if '--commands' in args:
	                sys.stdout.write(self.main_help_text(commands_only=True) + '\n')
	            elif len(options.args) < 1:
	                sys.stdout.write(self.main_help_text() + '\n')
	            else:
	                self.fetch_command(options.args[0]).print_help(self.prog_name, options.args[0])
	        # Special-cases: We want 'django-admin --version' and
	        # 'django-admin --help' to work, for backwards compatibility.
	        elif subcommand == 'version' or self.argv[1:] == ['--version']:
	            sys.stdout.write(django.get_version() + '\n')
	        elif self.argv[1:] in (['--help'], ['-h']):
	            sys.stdout.write(self.main_help_text() + '\n')
	        else:
	            self.fetch_command(subcommand).run_from_argv(self.argv)

其中
	parser = CommandParser(None, usage="%(prog)s subcommand [options] [args]", add_help=False)
	''         parser.add_argument('--settings')
	''         parser.add_argument('--pythonpath')
	''         parser.add_argument('args', nargs='*')  # catch-all
	''         try:
	''             options, args = parser.parse_known_args(self.argv[2:])
	''             handle_default_options(options)
	''         except CommandError:
	''             pass  # Ignore any option errors at this point.
	
CommandParser其实类似于Argparse的一个解析命令行参数的类，从代码里可以看出我们可以直接在命令行指定settings文件和pythonpath。

	no_settings_commands = [
	            'help', 'version', '--help', '--version', '-h',
	            'compilemessages', 'makemessages',
	            'startapp', 'startproject',
	        ]
	try:
	            settings.INSTALLED_APPS
	except ImproperlyConfigured as exc:
	            self.settings_exception = exc
	            # A handful of built-in management commands work without settings.
	            # Load the default settings -- where INSTALLED_APPS is empty.
	            if subcommand in no_settings_commands:
	                settings.configure()

这块代码就可以解释我们执行python manage.py start project 时django在背后会调用settings.configure方法，这里的settings是指django.conf.LazySettings的一个实例，configure方法其实就是使用django.conf.global\_settings.py中的默认设置创建一份新的配置文件，作为我们新创建的project的settings.py

	if settings.configured:
	            # Start the auto-reloading dev server even if the code is broken.
	            # The hardcoded condition is a code smell but we can't rely on a
	            # flag on the command class because we haven't located it yet.
	            if subcommand == 'runserver' and '--noreload' not in self.argv:
	                try:
	                    autoreload.check_errors(django.setup)()
	                except Exception:
	                    # The exception will be raised later in the child process
	                    # started by the autoreloader. Pretend it didn't happen by
	                    # loading an empty list of applications.
	                    apps.all_models = defaultdict(OrderedDict)
	                    apps.app_configs = OrderedDict()
	                    apps.apps_ready = apps.models_ready = apps.ready = True
	
	            # In all other cases, django.setup() is required to succeed.
	            else:
	                django.setup()

autoreload.check\_errors(django.setup)()其实也是调用django.setup方法，而django.setup方法

	def setup():
	    """
	    Configure the settings (this happens as a side effect of accessing the
	    first setting), configure logging and populate the app registry.
	    """
	    from django.apps import apps
	    from django.conf import settings
	    from django.utils.log import configure_logging
	
	    configure_logging(settings.LOGGING_CONFIG, settings.LOGGING)
	    apps.populate(settings.INSTALLED_APPS)

负责初始化日志模块以及所有应用.

### 抽丝剥茧 

剩下的代码最重要的就是这一句：

	self.fetch_command(subcommand).run_from_argv(self.argv)

fetch\_command会根据subcommand（这是我们执行python manage.py rumserver时传入的第二个参数：runserver），去django.core.management.commands中查找对应的command类，然后把所有命令行参数传给run\_from\_argv方法并执行，在runserver这个示例中，最终会调用django.utils.autoreload中的python\_reloader或者jython\_reloader新开一个线程：

	def python_reloader(main_func, args, kwargs):
	    if os.environ.get("RUN_MAIN") == "true":
	        thread.start_new_thread(main_func, args, kwargs)
	        try:
	            reloader_thread()
	        except KeyboardInterrupt:
	            pass
	    else:
	        try:
	            exit_code = restart_with_reloader()
	            if exit_code < 0:
	                os.kill(os.getpid(), -exit_code)
	            else:
	                sys.exit(exit_code)
	        except KeyboardInterrupt:
	            pass

这里的main\_func是commands/runserver.py中的inner\_run方法：

	def inner_run(self, *args, **options):
	        # If an exception was silenced in ManagementUtility.execute in order
	        # to be raised in the child process, raise it now.
	        autoreload.raise_last_exception()
	
	        threading = options.get('use_threading')
	        shutdown_message = options.get('shutdown_message', '')
	        quit_command = 'CTRL-BREAK' if sys.platform == 'win32' else 'CONTROL-C'
	
	        self.stdout.write("Performing system checks...\n\n")
	        self.check(display_num_errors=True)
	        self.check_migrations()
	        now = datetime.now().strftime('%B %d, %Y - %X')
	        if six.PY2:
	            now = now.decode(get_system_encoding())
	        self.stdout.write(now)
	        self.stdout.write((
	            "Django version %(version)s, using settings %(settings)r\n"
	            "Starting development server at http://%(addr)s:%(port)s/\n"
	            "Quit the server with %(quit_command)s.\n"
	        ) % {
	            "version": self.get_version(),
	            "settings": settings.SETTINGS_MODULE,
	            "addr": '[%s]' % self.addr if self._raw_ipv6 else self.addr,
	            "port": self.port,
	            "quit_command": quit_command,
	        })
	
	        try:
	            handler = self.get_handler(*args, **options)
	            run(self.addr, int(self.port), handler,
	                ipv6=self.use_ipv6, threading=threading)
	        except socket.error as e:
	            # Use helpful error messages instead of ugly tracebacks.
	            ERRORS = {
	                errno.EACCES: "You don't have permission to access that port.",
	                errno.EADDRINUSE: "That port is already in use.",
	                errno.EADDRNOTAVAIL: "That IP address can't be assigned to.",
	            }
	            try:
	                error_text = ERRORS[e.errno]
	            except KeyError:
	                error_text = force_text(e)
	            self.stderr.write("Error: %s" % error_text)
	            # Need to use an OS exit because sys.exit doesn't work in a thread
	            os._exit(1)
	        except KeyboardInterrupt:
	            if shutdown_message:
	                self.stdout.write(shutdown_message)
	            sys.exit(0)

最关键的是这两条语句：

	handler = self.get_handler(*args, **options)
	run(self.addr, int(self.port), handler,ipv6=self.use_ipv6, threading=threading)

get\_handler会返回django.core.servers.basehttp中定义的一个application（其实就是我们project下的wigs.py中定义的application）

这是run函数的内容

	def run(addr, port, wsgi_handler, ipv6=False, threading=False):
	    server_address = (addr, port)
	    if threading:
	        httpd_cls = type(str('WSGIServer'), (socketserver.ThreadingMixIn, WSGIServer), {})
	    else:
	        httpd_cls = WSGIServer
	    httpd = httpd_cls(server_address, WSGIRequestHandler, ipv6=ipv6)
	    if threading:
	        # ThreadingMixIn.daemon_threads indicates how threads will behave on an
	        # abrupt shutdown; like quitting the server by the user or restarting
	        # by the auto-reloader. True means the server will not wait for thread
	        # termination before it quits. This will make auto-reloader faster
	        # and will prevent the need to kill the server manually if a thread
	        # isn't terminating correctly.
	        httpd.daemon_threads = True
	    httpd.set_app(wsgi_handler)
	    httpd.serve_forever()

可以看出run函数其实就是启动一个WSGIServer实例（WSGIServer继承python内置类simple\_server.WSGIServer），并把handler设置为前面get\_handler的返回值

### 水落石出

这样，一条python manage.py runserver命令的执行生命周期就一览无余了。
接下来，server就开始接收请求了。




_

[image-1]:	/images/django_project.png