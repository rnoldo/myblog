今天要在Mac上装一个apache + mysql + mod_wsgi + django环境，在安装mod_wsgi的时候我用了两种方式：
#brew install mod_wsgi
我机子上装了homebrew（建议你也去装一下这个神器），当执行`brew install mod_wsgi`的时候出现了如下错误：

	No available formula for mod_wsgi
	

这种方式不行，我打算下载源码，自己编译安装。

#下载源码，自行安装
步骤如下：

* 下载源码：[下载地址](http://code.google.com/p/modwsgi/wiki/DownloadTheSoftware?tm=2)
* `tar -xvf mod_wsgi-3.4.tar.gz`
* `cd mod_wsgi-3.4`
* `./configure`
* `make`

执行到`make`的时候，报了以下错误

	/usr/share/apr-1/build-1/libtool --tag=CC --mode=link /usr/bin/clang -o mod_wsgi.la  -rpath /usr/libexec/apache2 -module -avoid-version    mod_wsgi.lo -L/usr/local/lib -Wl,-F/usr/local/Cellar/python/2.7.3/Frameworks -framework Python -u _PyMac_Error -arch x86_64 -ldl -framework CoreFoundation /Applications/Xcode.app/Contents/Developer/Toolchains/OSX10.8.xctoolchain/usr/bin/cc ${wl}-undefined ${wl}dynamic_lookup -o .libs/mod_wsgi.so -bundle  .libs/mod_wsgi.o  -L/usr/local/lib -ldl  -Wl,-F/usr/local/Cellar/python/2.7.3/Frameworks -framework Python -arch x86_64 -framework CoreFoundation /usr/share/apr-1/build-1/libtool: line 4574: /Applications/Xcode.app/Contents/Developer/Toolchains/OSX10.8.xctoolchain/usr/bin/cc: No such file or directory apxs:Error: Command failed with rc=8323072 
	.
	make: *** [mod_wsgi.la] Error 1

搜索了半天，终于找到了homebrew的一个issuse下，这里对两种安装方法的问题都给了解决方案：

1. 对于第一种，执行以下步骤
	* brew tap homebrew/apache
	* `[ "$(sw_vers -productVersion | sed 's/^\(10\.[0-9]\).*/\1/')" = "10.8" ] && bash -c "[ -d /Applications/Xcode.app/Contents/Developer/Toolchains/XcodeDefault.xctoolchain ] && sudo bash -c 'cd /Applications/Xcode.app/Contents/Developer/Toolchains/ && ln -vs XcodeDefault.xctoolchain OSX10.8.xctoolchain' || sudo bash -c 'mkdir -vp /Applications/Xcode.app/Contents/Developer/Toolchains/OSX10.8.xctoolchain/usr && cd /Applications/Xcode.app/Contents/Developer/Toolchains/OSX10.8.xctoolchain/usr && ln -vs /usr/bin'"`
	* brew install mod_wsgi
2. 对于第二种安装方式，执行如下命令之后即可正常make和make install了
	* `[ "$(sw_vers -productVersion | sed 's/^\(10\.[0-9]\).*/\1/')" = "10.8" ] && bash -c "[ -d /Applications/Xcode.app/Contents/Developer/Toolchains/XcodeDefault.xctoolchain ] && sudo bash -c 'cd /Applications/Xcode.app/Contents/Developer/Toolchains/ && ln -vs XcodeDefault.xctoolchain OSX10.8.xctoolchain' || sudo bash -c 'mkdir -vp /Applications/Xcode.app/Contents/Developer/Toolchains/OSX10.8.xctoolchain/usr && cd /Applications/Xcode.app/Contents/Developer/Toolchains/OSX10.8.xctoolchain/usr && ln -vs /usr/bin'"`
	
	
产生这个问题的原因这里就不详细解释了，有兴趣的请看以下两个链接:

* <https://github.com/mxcl/homebrew/issues/13919>
* <https://github.com/Homebrew/homebrew-apache> 

