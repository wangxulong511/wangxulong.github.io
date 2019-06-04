启动scrapy的时候报如下错误

```
raceback (most recent call last):
  File "/usr/local/python3/lib/python3.6/site-packages/Twisted-17.1.0-py3.6-linux-x86_64.egg/twisted/internet/defer.py", line 1301,
 in _inlineCallbacks
    result = g.send(result)
  File "/usr/local/python3/lib/python3.6/site-packages/scrapy/crawler.py", line 82, in crawl
    yield self.engine.open_spider(self.spider, start_requests)
ModuleNotFoundError: No module named '_sqlite3'
```

python3 import 导入报错  
```
[root@centos-jstdb-1 sqlite-autoconf-3280000]# python3
Python 3.6.6 (default, Aug 21 2018, 17:05:32) 
[GCC 4.4.7 20120313 (Red Hat 4.4.7-18)] on linux
Type "help", "copyright", "credits" or "license" for more information.
>>> import sqlite3
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
  File "/usr/local/lib/python3.6/sqlite3/__init__.py", line 23, in <module>
    from sqlite3.dbapi2 import *
  File "/usr/local/lib/python3.6/sqlite3/dbapi2.py", line 27, in <module>
    from _sqlite3 import *
ModuleNotFoundError: No module named '_sqlite3'
>>> 
```


针对这个问题，只能采取最原始的办法来进行安装，具体操作如下;
（1）安装sqlite3的包

$ wget https://www.sqlite.org/2017/sqlite-autoconf-3170000.tar.gz --no-check-certificate
$ tar zxvf sqlite-autoconf-3170000.tar.gz
$ cd sqlite-autoconf-3170000
$ ./configure --prefix=/usr/local/sqlite3 --disable-static --enable-fts5 --enable-json1 CFLAGS="-g -O2 -DSQLITE_ENABLE_FTS3=1 -DSQLITE_ENABLE_FTS4=1 -DSQLITE_ENABLE_RTREE=1"

（2）对python3进行重新编译
$ cd Python-3.6.2
$ LD_RUN_PATH=/usr/local/sqlite3/lib ./configure LDFLAGS="-L/usr/local/sqlite3/lib" CPPFLAGS="-I /usr/local/sqlite3/include"
$ LD_RUN_PATH=/usr/local/sqlite3/lib make
$ LD_RUN_PATH=/usr/local/sqlite3/lib sudo make install
通过上述安装过程，终于可以顺利实现python对sqlite3的支持了。


# python3安装sqlite3库
> https://www.jianshu.com/p/3ca63eb17553
# Python3.6安装sqlite3的终极解决办法
> https://blog.csdn.net/sparkexpert/article/details/79118448

# ModuleNotFoundError: No module named '_sqlite3'
> https://blog.csdn.net/weixin_43229107/article/details/83689294

