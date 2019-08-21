## windows maven idea 开发环境安装
 1. [下载源码](http://ftp.cuhk.edu.hk/pub/packages/apache.org/maven/maven-3/)
 
 2. 本次下载的apache-maven-3.6.0-src.zip
 
 3. 解压到idea项目目录
   + E:\project\github\apache-maven-3.6.1
   
 4. idea import maven 项目
   + maven-artifact
   + maven-builder-support
   + maven-compat
   + maven-core
   + maven-embedder
   + maven-model
   + maven-model-builder
   + maven-plugin-api
   + maven-repository-metadata
   + maven-resolver-provider
   + maven-settings
   + maven-settings-builder
   + maven-slf4j-provider
   
 5. 首先还需要在windows安装上maven的命令行工具mvn
 
 6. 打开命令行工具
 
 7. 进入maven项目根目录
   + cd E:\project\github\apache-maven-3.6.1
   
 8. 执行安装命令
   + mvn clean package -DskipTests
 9. 安装完成
 
 10. 启动MavenCli
   + maven-embedder下打开MavenCli
   + 启动debug 
 
### 配置debug参数
 * 参考%MAVEN_HOME%/bin/mvn.cmd
   + 当执行: mvn clean package -DskipTests -f other_pom.xml -P staging
   + mvn执行过程
      - 参考 maven-mvn-cmd.md
 
#### 用户配置参数
 * 通过main参数传递
 * 配置-Dname=value
 * 可用配置
   + style.color=[always|never|auto]
      - 默认auto
      - 