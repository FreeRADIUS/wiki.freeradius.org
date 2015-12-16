thanks to others who provide content to related topic, for new comer's convenience, i just collect them here.
1. setup tomcat, refer to http://jingyan.baidu.com/article/b0b63dbfedb1e34a483070cd.html,
   note: 1. the guide refer to a fixed version of tomcat, may not be the one you will download, so change it
         2. the tomcat controller was not found, so i just go to /Library/tomcat/bin and run ./startup.sh instead to start tomcat

2. setup eclipse and using your tomcat server as runtime server, refer to http://jingyan.baidu.com/article/e9fb46e192e0147521f76614.html
   note: 1. you may meet 404 when you startup tomcat server from eclipse, this is because :
      通过eclipse来启动tomcat会碰到“访问http://localhost:8080出现404错误”这样的问题，需要在eclipse中进行一系列的设置才行。
      解决：打开eclipse的server视图，双击你配置的那个tomcat，打开编辑窗口，查看server locations，看看是否选择了第一个选项（默认是第一个选项），即  
            use workspace metadata(does not modify tomcat instation),你应该选择第二个选项，use tomcat installation.然后你再启动服务就可以了。
            如果server location中的三个单选框呈灰色不可编辑，那么你就要把先前导入的删除掉，重新导入一次，修改server locatioins。

3. preparation: maven

3.1下载maven的bin包，解压，配置到环境变量里面去
1）、首先到Maven官网下载安装文件，比如 ，下载文件为apache-maven-3.0.3-bin.tar.gz
2）、配置环境变量
[android(0)@liangbingmatoMacBook-Pro ~]$ cd  ~
[android(0)@liangbingmatoMacBook-Pro ~]$ open  -e .bash_profile 
添加的环境变量如下：具体看你解压在哪里具体的配置
MAVEN_HOME=/User/android/apache-maven-3.0.3
export MAVEN_HOME
export PATH=${PATH}:${MAVEN_HOME}/bin
 
执行 [android(0)@liangbingmatoMacBook-Pro ~]$ source .bash_profile 保存环境变量
 
检验是否成功：mvn  -v ，输出如下
liangbingmatoMacBook-Pro:~ android$ mvn -v
Apache Maven 3.0.3 (r1075438; 2011-03-01 01:31:09+0800)
Maven home: /usr/share/maven
Java version: 1.6.0_26, vendor: Apple Inc.
Java home: /System/Library/Java/JavaVirtualMachines/1.6.0.jdk/Contents/Home
Default locale: en_US, platform encoding: EUC_CN
OS name: "mac os x", version: "10.7", arch: "x86_64", family: "mac"

3.2 安装eclipse插件 m2eclipse

4. startup sprint with a sample project download from spring.io: http://spring.io/guides/gs/rest-service/