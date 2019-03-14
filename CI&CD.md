1. 买域名 百度搜
2. 买服务器 百度搜便宜的
3. 域名绑定服务器
4. 相关软件安装
  - linux shell   命令
    ssh -p 28450 root@144.34.151.57
    下面还有一些简单命令
  - jenkins
    1. 查看是否有java环境
        ``` 
        java -version
        ```
        output 输出
        ```shell
        java version "1.7.0_25"

        Open JDK Runtime Environment (rhel-2.3.10.4.el6_4-x86_64)

        Open JDK 64-Bit Server VM (build 23.7-b01, mixed mode)
        ```
        则java环境没问题
    2. 配置java环境
      1. 安装JDK 使用解压安装，不然找不到JAVA_HOME地址
      - 卸载之前JDK
        ```shell
          # 查看是否安装Java
          java -version
          # 查看Java的安装包信息
          rpm -qa | grep java
          # 卸载原装Java,<java_package>为查找到的安装包信息
          rpm -e --nodeps <java_package>
        ```
      - 使用wget url命令下载自己想要的JDK 解压 安装
        ```shell
        wget Url
        # 创建安装目录
        mkdir -p /usr/local/java
        # 解压
        tar -xzvf jdk-8u151-linux-i586.tar.gz
        ```
          **注意：** 下载JDK有一个问题 如果直接用wget 连接网址 下载下来的JDK是无法通过的 前面还要加一段命令
          wget **--no-check-certificate --no-cookies --header "Cookie: oraclelicense=accept-securebackup-cookie"** http://download.oracle.com/otn-pub/java/jdk/8u191-b12/2787e4a523244c269598db4e85c51e0c/jdk-8u191-linux-x64.tar.gz
          ![JDK](https://static-s.styd.cn/201903141352/jDK.png)

      - 配置环境变量
        ```shell 
          vi /etc/profile
        ```

        ```text
          export JAVA_HOME=/usr/local/java/jdk1.8.0_151 # 自己安装JDK的版本号
          export CLASSPATH=.:$JAVA_HOME/lib/tool.jar:$JAVA_HOME/lib/dt.jar
          export PATH=$PATH:$JAVA_HOME/bin
        ```
    3. 下载tomcat 配置tomcat环境 开机启动
      - tomcat安装
        - 安装 
          ```shell
          wget tomcatUrl
          tar -xzvf apache-tomcat-8.5.23.tar.gz -C /usr/local/java
          ```
        - 配置Tomcat的环境变量
          在/etc/profile文件后再追加一条TOMCAT的环境变量
            ```text
              # 在/etc/profile文件末尾追加TOMCAT的环境变量
              export CATALINA_HOME=/usr/local/java/apache-tomcat-8.5.23
            ```
        - 配置开机启动
          重启防火墙
          ```shell
          service iptables restart
          ```
          配置服务
            - 将上述tomcat中的catalina.sh复制到/etc/init.d/并命名为tomcat
            ```shell
            cp /usr/lib/tomcat/apache-tomcat-9.0.16/bin/catalina.sh /etc/init.d/tomcat9
            chmod 755 /etc/init.d/tomcat7
            ```
            接着对拷贝的脚本进行修改,添加chkconfig和description两行注释,同时加上JAVA_HOME和CATALINA_HOME声明
            ```text
            #chkconfig: 2345 10 90
            #description: tomcat8 service
            export JAVA_HOME=/software/jdk1.7.0_51
            export CATALINA_HOME=/software/apache-tomcat-7.0.55
            ```
            使用chkconfig -add tomcat7 将其添加为服务
            ```shell
            chkconfig --add tomcat7
            ```
            配置开机启动
            ```shell 
            chkconfig tomcat7 on
            ```
            配置防火墙打开80端口

            ```js
            /sbin/iptables -I INPUT -p tcp --dport 80 -j ACCEPT 
            /sbin/iptables -I INPUT -p tcp --dport 8080 -j ACCEPT 
            /etc/init.d/iptables save
            service iptables restart
            ```
          ### 一些细节问题
            1.JAVA_HOME， JRE_HOME，CLASS_PATH，PATH四个环境变量一定要确认无误（我就是在这翻车了）。 
            2.RH-Firewall-1-INPUT 表示进出规则在一个链上。（我参照22端口的设置做，第二次翻车） 
            3.【chkconfig:2345 10 90】 2345表示系统运行级别是2，3，4或者5时都启动此服务，20，是启动的优先级，80是关闭的优先级，如果启动优先级配置的数太小时如0时，则有可能启动不成功，因为此时可能其依赖的网络服务还没有启动，从而导致自启动失败。 
            4.在配置服务时要再一次配置环境变量，因为原有的profile的环境变量在此处不会生效。 
            5.配置启动服务的环境变量时要加上CATALINA_HOME指向tomcat文件夹根目录。

    4. 下载 jenkins.war 包 拷贝到tomcat webapps目录中
    5. 重启tomcat
    6. 访问 http://ip:8080/jenkins
    
  - nginx 用来做代理
    **来个简单的**
    ```nginx
      server {
        listen       80;
        server_name  www.fezlx.xyz;
        location / {
            root: html;
            index  index.html index.htm;
        }
        error_page   500 502 503 504  /50x.html;
        location = /50x.html {
            root   html;
        }
    }
    ```
    如果需要访问跟目录 root目录的静态资源 一定要给nginx权限 **user toot**配置到nginx.conf最顶端
    这样就可以简单配置一个web服务器  绑定服务器端口 绑定域名 绑定服务器静态资源
    
    **弄一个 一个nginx服务器绑定多个域名 访问多个资源**
    ```nginx
      upstream mysvr {
        server 127.0.0.1:3009; 
      }
    upstream hodorsvr {
      server 127.0.0.1:4000; 
    }          
    server {
        listen       80;
        server_name  mysite.fezlx.xyz;
        location / {
            proxy_pass http://mysvr;
        }
        error_page   500 502 503 504  /50x.html;
        location = /50x.html {
            root   html;
        }
    }
    server {
        listen       80;
        server_name  www.fezlx.xyz;
        location / {
            proxy_pass http://hodorsvr;
        }

        error_page   500 502 503 504  /50x.html;
        location = /50x.html {
            root   html;
        }
    }
    server {
        listen       4000;
        server_name  localhost;
        location / {
            proxy_pass http://hodorsvr;
        }

        error_page   500 502 503 504  /50x.html;
        location = /50x.html {
            root   html;
        }
    }
    server {
        listen       3009;
        server_name  localhost;
        location / {
            proxy_pass http://hodorsvr;
        }

        error_page   500 502 503 504  /50x.html;
        location = /50x.html {
            root   html;
        }
    }

    ```
    ```shell
    /usr/local/webserver/nginx/sbin/nginx -t
    ```
    重启nginx
    ```shell
    /usr/local/webserver/nginx/sbin/nginx -s reload
    ```

  - gitlab webhook 配置
