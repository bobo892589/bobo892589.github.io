---
title: jenkins和sonar环境配置MAC版
date: 2017-07-10 17:34:43
tags: macOS jenkins sonar
---
## 系统工具安装
### Xcode 安装
打开一下Xcode安装CommandLineTools，尝试一下xcodebuild命令，如果不行，指定一下Xcode安装目录
```
sudo xcode-select -switch /Applications/Xcode.app/Contents/Developer
```
### Atom 编辑器（随意）

### Java 安装  
版本最新就好了，下Mac 版的dmg文件点击安装，环境设置都省了。   
javasdk 下载地址：   
http://www.oracle.com/technetwork/java/javase/downloads/index.html
### Homebrew 安装
官网 https://brew.sh/index_zh-cn.html ,`macOS`软件包管理器，有各种工具，非常有用，必装。安装命令（或者去官网查看）：
```
/usr/bin/ruby -e "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install)"
```
### Android SDK安装
### Maven 安装
主要是用来打包sonar的扩展包jar
```
brew install maven
# mvn package 打包命令
```
### Xcpretty 安装
格式化xcode的输出，以及生成sonar需要的编译信息
```
sudo gem install xcpretty
```
### OCLint 安装
```
brew cask install oclint
```
### Cocoapods 安装
```
sudo gem install cocoapods
pod setup
```
### Gradle安装
```
brew install gradle
```
### Groovy 安装（随意）
`brew install groovy`来安装运行环境
## Mysql 安装和配置
sonar需要使用数据库
### 安装和启动
安装
```
brew install mysql
```
启动
```
mysql.server start
```
配置root
```
mysql_secure_installation
```
配置为系统服务
```
brew services start mysql
```
### 配置sonar的用户和数据库实例
创建sonar用户和数据库sonar_qube，赋予sonar权限。
```
CREATE DATABASE sonar_qube;
CREATE USER 'sonar'@'localhost' IDENTIFIED BY '你要设置的密码';
GRANT ALL PRIVILEGES ON sonar_qube.* TO 'sonar'@'localhost';
FLUSH PRIVILEGES;
```

### phpMyAdmin 安装（随意）
mysql的可视化工具，装一个方便点。官网https://www.phpmyadmin.net 下载一个
压缩包，解压，文件夹改名为phpMyAdmin，直接移动到`/Library/WebServer/Documents`,修改`/etc/apache2/httpd.conf`配置，打开注释php5_module
```
LoadModule php5_module libexec/apache2/libphp5.so
```
启动apachectl
```
sudo apachectl start  #如果已经启动则restart
```
打开 http://localhost/phpMyAdmin/ 可以看到了。如果登陆时提示No such file or directory，phpMyAdmin默认读取路径跟实际路径ln一下，运行
```
sudo mkdir /var/mysql
sudo ln -s /tmp/mysql.sock /var/mysql/mysql.sock
```
## Sonar 安装和配置
Sonar主要包含两个部分，SonarQube和sonar-scanner，SonarQube就是sonar的web服务提供可视化界面，sonar-scanner是用于扫描分析项目的代码静态结果提交到SonarQube。下载SonarQube5.3版本，对应的sonar-scanner2.5版本（由于开源的oc插件支持的原因），下载地址：https://www.sonarqube.org/downloads/

### 配置和启动
修改配置
```
#修改 <install_directory>/conf/sonar.properties
#数据库相关配置
sonar.jdbc.username=sonar
sonar.jdbc.password=替换设置的密码
sonar.jdbc.url=jdbc:mysql://localhost:3306/sonar_qube?useUnicode=true&characterEncoding=utf8&rewriteBatchedStatements=true&useConfigs=maxPerformance
#admin的配置
sonar.sorceEncoding=UTF-8
sonar.login=admin
sonar.password=替换设置的密码
```
启动
```
cd <install_directory>//bin/macosx-universal-64/

#./sonar.sh console Debug信息（最好使用该命令试试能否正常的启动）
#./sonar.sh start   启动服务
#./sonar.sh stop    停止服务
#./sonar.sh restart 重启服务
```

### 中文扩展包（Chinese Pack）
源码地址：https://github.com/SonarQubeCommunity/sonar-l10n-zh clone源码在本地打包，注意源码要切换到支持sonar5.3的版本，建议是`0ff5e8f`。
```
cd <clone_directory>  #就是在pom.xml文件的目录
mvn package
```
### ObjectiveC扩展包（ObjectiveC）
sonar-objective-c的原作者已经不再更新了，`fork`里面一直还在更新的是：https://github.com/Backelite/sonar-objective-c ，在之前也用过：https://github.com/mjdetullio/sonar-objective-c/network ，不过Backelite已经支持到oclint0.11了，基本是最新的，所以采用是Backelite的版本。先copy里面的`profile-oclint.xml`和`rules.txt`这两个文件出来， 切换到版本`12b2a7a`再把原先的两个文件覆盖回去，这样保证最新rules，这样基本不会出现找不到规则的错误。
```
cd <clone_directory>  #就是在pom.xml文件的目录
mvn package
```
如果出现没找到规则的错误
```
ERROR: Caused by: The rule 'OCLint:switch statements don't need default when fully covered' does not exist.
```
这种错误跑一下源码里面的`updateOCLintRules.groovy`groovy脚本，再重新打包。
```
cd <clone_directory>
sudo groovy updateOCLintRules.groovy
```
或者手动新增，修改org/sonar/plugins/oclint的profile-oclint.xml和rules.txt文件
，内容跟那个错误一样，再重新打包。profile-oclint.xml追加：
```
<rule>
   <repositoryKey>OCLint</repositoryKey>
   <key>switch statements don't need default when fully covered</key>
</rule>
```
rules.txt追加：
```
----------

Summary:

Severity: 1
Category: OCLint

switch statements don't need default when fully covered
```
### Android扩展包
源码地址：https://github.com/ofields/sonar-android ，使用1.1版本。


### sonar配置为系统服务
使用Mac特有的launchctl来启动时加载    
写一个plist文件
```
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
  <dict>
    <key>Label</key>
    <string>com.launchctl.sonarqube</string>
    <key>ProgramArguments</key>
    <array>
      <string><sonarqube_install_dir>/bin/macosx-universal-64/sonar.sh</string>
      <string>start</string>
    </array>
    <key>RunAtLoad</key>
    <true/>
  </dict>
</plist>
```
launchctl常用命令
```
# 加载plist
launchctl load <sonarqube_install_dir>/com.launchctl.sonarqube.plist
# 卸载plist
launchctl unload <sonarqube_install_dir>/com.launchctl.sonarqube.plist
# 立刻执行plist的内容
launchctl start <sonarqube_install_dir>/com.launchctl.sonarqube.plist
# 立刻停止plist的内容
launchctl stop <sonarqube_install_dir>/com.launchctl.sonarqube.plist
# 列出所有plist
launchctl list
```
注意：launchctl unload和stop并未停止sonar的服务，还需要调用一下`<sonarqube_install_dir>/bin/macosx-universal-64/sonar.sh stop`。
### 配置外网访问

修改<install_directory>/conf/sonar.properties，修改host，访问路径/sonar，禁止直接访问。
```
sonar.web.host=127.0.0.1
sonar.web.context=/sonar
sonar.web.port=9090
```
修改/etc/apache2/httpd.conf，用路径代理/sonar访问。
```
#加载 mod_proxy 模块
#确保 Apache 的配置文件 /etc/apache2/httpd.conf 内有打开以下两行：
LoadModule proxy_module libexec/apache2/mod_proxy.so
LoadModule proxy_http_module libexec/apache2/mod_proxy_http.so
#配置 Apache 站点
```
```
ProxyPass         /sonar  http://localhost:9090/sonar
ProxyPassReverse  /sonar  http://localhost:9090/sonar
ProxyRequests     Off
# Local reverse proxy authorization override
# Most unix distribution deny proxy by default (ie /etc/apache2/mods-enabled/proxy.conf in Ubuntu)
<Proxy http://localhost:9090/sonar*>
  Order deny,allow
  Allow from all
</Proxy>
```
然后重启apachectl和sonar。

### 批量创建用户
直接使用sonar服务本身webapi，更详细文档参考https://sonarcloud.io/web_api/
```
#!/bin/bash
ACCOUNTS=('user1' 'user2' 'user3')
for ACCOUNT in ${ACCOUNTS[*]}
do
	echo $ACCOUNT
  curl -u admin:<admin_password> -d "login=$ACCOUNT&name=$ACCOUNT&password=$ACCOUNT" "http://xxx.xxx.xxx.xxx/sonar/api/users/create"
done
```

## Jenkins 安装和配置
### 安装和启动
安装
```
brew install jenkins
```
启动为系统服务
```
brew services start jenkins
```
打开http://localhost:8080 按照提示进行admin用户设置
### 配置外网访问
修改/usr/local/Cellar/jenkins/2.69/homebrew.mxcl.jenkins.plist
```
<string>--prefix=/jenkins</string>
```
使用系统的`Apache`代理
```
#加载 mod_proxy 模块
#确保 Apache 的配置文件 /etc/apache2/httpd.conf 内有打开以下两行：
LoadModule proxy_module libexec/apache2/mod_proxy.so
LoadModule proxy_http_module libexec/apache2/mod_proxy_http.so
```
```
#配置 Apache 站点
ProxyPass         /jenkins  http://localhost:8080/jenkins
ProxyPassReverse  /jenkins  http://localhost:8080/jenkins
ProxyRequests     Off

# Local reverse proxy authorization override
# Most unix distribution deny proxy by default (ie /etc/apache2/mods-enabled/proxy.conf in Ubuntu)
<Proxy http://localhost:8080/jenkins*>
  Order deny,allow
  Allow from all
</Proxy>
```

### Jenkins的全局配置

系统设置：`Environment variables`的`ANDROID_HOME`，`SonarQube servers`的必填参数。   
Global Tool Configuration : JDK，Gradle，SonarQube Scanner需要配置相关的安装路径。

### Jenkins需额外安装的插件
Extensible Choice Parameter plugin :动态参数，可以使用groovy脚本   
Git Parameter Plug-In :Git分支参数   
SonarQube Scanner for Jenkins :Sonar必须要   
Gradle Plugin :可以不用写脚本调用Gradle

### 批量创建用户
进入http://<sever_url>/script，使用groovy脚本来批量创建用户，创建脚本如下：
```
def accounts = ['user1','user2','user3']
for(int i = 0;i<accounts.size;i++) {
  println(accounts.get(i))
  jenkins.model.Jenkins.instance.securityRealm.createAccount(accounts.get(i), accounts.get(i))
}
```

### 脚本需要注意
直接使用脚本的构建在编写脚本时需要注意，脚本的上下文环境跟服务器的上下文环境是不一样，所以经常会遇到`command not found`的错误，那么需要使用绝对路径来使用一些命令，或者把服务器的`PATH` `export`进去，还有设置一下`utf8`的全局编码。
```
export PATH=$PATH:/usr/local/bin:/usr/bin:/bin:/usr/sbin:/sbin
export LC_ALL=en_US.UTF-8
```

## Sonar构建使用到的shell命令
iOS
```
cd <project_dir>
xcodebuild -scheme XXXX -workspace XXXX.xcworkspace -sdk iphonesimulator -configuration Debug clean build | tee xcodebuild.log | xcpretty --report json-compilation-database --output compile_commands.json
oclint-json-compilation-database -v -e Pods -- -report-type=pmd -o=sonar-reports/oclint.xml
//项目根目录下需要配置好的sonar-project.properties文件
<sonar-scanner_dir>sonar-runner -X

```
Android
```
cd <project_dir>
gradle build
//项目根目录下需要配置好的sonar-project.properties文件
<sonar-scanner_dir>sonar-runner -X
```
