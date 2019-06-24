shell 命令
for i in `ls`;do cd $i;/home/centos/tomcat/sh/server2.sh start *.jar .;cd -;done

