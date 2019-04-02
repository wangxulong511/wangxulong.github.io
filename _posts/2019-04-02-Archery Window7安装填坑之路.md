# archery
SQL审核查询平台，查询支持(MySQL/MsSQL/Redis/PostgreSQL)、MySQL优化(SQLAdvisor|SOAR|SQLTuning)、慢日志管理、表结构对比、会话管理、阿里云RDS管理

[github地址](https://github.com/hhyo/Archery)
官方建议 docker 部署  
window7 环境手动安装  

window 下安装  
git clone https://github.com/hhyo/Archery.git  
在window7 下安装python依赖模块  
pip3 install -r requirements.txt -i https://mirrors.ustc.edu.cn/pypi/web/simple/

安装mysqlclient时遇到的错误 
microsoft visual c++ 14.0 is required

```
需要安装
Visual C++ 2015 Build Tools
百度云archery依赖包

安装 MySQL Connector/Python (Archived Versions)  
https://downloads.mysql.com/archives/c-python/

```
安装阿里云rds模块报错  

```
安装 pycrypto 
pycrypto安装出错的问题 intmax_t C:\Program Files (x86)\Windows Kits\10\include\10.0.10240.0\ucrt\inttypes.  
解决地址： 
https://wangxulong511.github.io/2019/03/25/python3.6%E5%AE%89%E8%A3%85pycrypto%E6%8A%A5%E9%94%99/  

安装 python-ldap  
pip install python-ldap==3.2.0 报错如下： 

    LDAPObject.c
    c:\users\sz_jst122\appdata\local\temp\pip-install-nk3xgzal\python-ldap\modules\constants.h(7): fatal error C1083: Cannot open include file: 'lber.h': No such file or directory
    error: command 'C:\\Program Files (x86)\\Microsoft Visual Studio 14.0\\VC\\BIN\\x86_amd64\\cl.exe' failed with exit status 2

解决办法
在 https://www.lfd.uci.edu/~gohlke/pythonlibs/#python-ldap
下载 python_ldap‑3.2.0‑cp36‑cp36m‑win_amd64.whl



修复在python3中ImportError: No module named 'winrandom'错误
!(https://www.jianshu.com/p/532dbf349b3e)

```
百度云archery依赖包  
链接：https://pan.baidu.com/s/19CSyRbZlqOrrP_RYDVqDWg 
提取码：t14q 


