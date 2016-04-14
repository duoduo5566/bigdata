# CentOS卸载OpenJdk

标签（空格分隔）： Linux

---
安装好CentOS后，想要安装指定版本JDK，安装之前检查是否已安装：

    [root@node1 data]# java -version
    java version "1.7.0_45"
    OpenJDK Runtime Environment (rhel-2.4.3.3.el6-x86_64 u45-b15)
    OpenJDK 64-Bit Server VM (build 24.45-b08, mixed mode)

发现系统已经安装了OpenJDK，进一步查看装了哪些JDK：

    [root@node1 data]#  rpm -qa | grep java
    tzdata-java-2013g-1.el6.noarch
    java-1.7.0-openjdk-1.7.0.45-2.4.3.3.el6.x86_64
    java-1.6.0-openjdk-1.6.0.0-1.66.1.13.0.el6.x86_64

卸载自带的OpenJDK：

    [root@node1 data]# rpm -e --nodeps java-1.6.0-openjdk-1.6.0.0-1.66.1.13.0.el6.x86_64
    [root@node1 data]# rpm -e --nodeps java-1.7.0-openjdk-1.7.0.45-2.4.3.3.el6.x86_64
    [root@node1 data]# rpm -e --nodeps tzdata-java-2013g-1.el6.noarch

查看是否卸载干净：

    [root@node1 data]#  rpm -qa | grep java
    [root@node1 data]# java -version
    bash: /usr/bin/java: No such file or directory

至此，OpenJDK已经卸载完成。


