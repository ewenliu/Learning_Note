# Windows10 搭建 kafka环境

[TOC]

## 一、安装java

### 1.1 下载java

华为云镜像jdk下载地址：

> https://repo.huaweicloud.com/java/jdk/11.0.2+7/jdk-11.0.2_windows-x64_bin.exe

### 1.2 java环境变量配置
win10 下添加java环境变量如下：

> 计算机 --> 高级系统设置 --> 高级 --> 环境变量 --> path 中将java的二进制可执行文件根目录添加进去 如C:\Program Files\Java\jdk-11.0.2\bin


## 二、安装kafka

### 2.1 下载kafka

清华大学kafka镜像下载地址：

> https://mirrors.tuna.tsinghua.edu.cn/apache/kafka/2.4.0/kafka_2.11-2.4.0.tgz

### 2.2 安装kafka

下载完解压即可


## 三、kafka启动

### 3.1 启动ZooKeeper
进入kafka目录\bin\windows，我的目录D:\kafka\kafka_2.11-2.4.0\bin\windows，执行如下：

```cmd

zookeeper-server-start.bat ..\..\config\zookeeper.properties

D:\kafka\kafka_2.11-2.4.0\bin\windows>zookeeper-server-start.bat ..\..\config\zookeeper.properties
[2020-01-02 16:17:52,702] INFO Reading configuration from: ..\..\config\zookeeper.properties (org.apache.zookeeper.server.quorum.QuorumPeerConfig)
[2020-01-02 16:17:52,704] WARN ..\..\config\zookeeper.properties is relative. Prepend .\ to indicate that you're sure! (org.apache.zookeeper.server.quorum.QuorumPeerConfig)
[2020-01-02 16:17:52,704] WARN \tmp\zookeeper is relative. Prepend .\ to indicate that you're sure! (org.apache.zookeeper.server.quorum.QuorumPeerConfig)
[2020-01-02 16:17:52,706] INFO clientPortAddress is 0.0.0.0/0.0.0.0:2181 (org.apache.zookeeper.server.quorum.QuorumPeerConfig)
[2020-01-02 16:17:52,706] INFO secureClientPort is not set (org.apache.zookeeper.server.quorum.QuorumPeerConfig)
[2020-01-02 16:17:52,709] INFO autopurge.snapRetainCount set to 3 (org.apache.zookeeper.server.DatadirCleanupManager)
[2020-01-02 16:17:52,709] INFO autopurge.purgeInterval set to 0 (org.apache.zookeeper.server.DatadirCleanupManager)
[2020-01-02 16:17:52,710] INFO Purge task is not scheduled. (org.apache.zookeeper.server.DatadirCleanupManager)
[2020-01-02 16:17:52,710] WARN Either no config or no quorum defined in config, running  in standalone mode (org.apache.zookeeper.server.quorum.QuorumPeerMain)
[2020-01-02 16:17:52,712] INFO Log4j found with jmx enabled. (org.apache.zookeeper.jmx.ManagedUtil)
[2020-01-02 16:17:52,728] INFO Reading configuration from: ..\..\config\zookeeper.properties (org.apache.zookeeper.server.quorum.QuorumPeerConfig)
[2020-01-02 16:17:52,728] WARN ..\..\config\zookeeper.properties is relative. Prepend .\ to indicate that you're sure! (org.apache.zookeeper.server.quorum.QuorumPeerConfig)
[2020-01-02 16:17:52,729] WARN \tmp\zookeeper is relative. Prepend .\ to indicate that you're sure! (org.apache.zookeeper.server.quorum.QuorumPeerConfig)
[2020-01-02 16:17:52,729] INFO clientPortAddress is 0.0.0.0/0.0.0.0:2181 (org.apache.zookeeper.server.quorum.QuorumPeerConfig)
[2020-01-02 16:17:52,729] INFO secureClientPort is not set (org.apache.zookeeper.server.quorum.QuorumPeerConfig)
[2020-01-02 16:17:52,730] INFO Starting server (org.apache.zookeeper.server.ZooKeeperServerMain)
[2020-01-02 16:17:52,733] INFO zookeeper.snapshot.trust.empty : false (org.apache.zookeeper.server.persistence.FileTxnSnapLog)
[2020-01-02 16:17:52,744] INFO Server environment:zookeeper.version=3.5.6-c11b7e26bc554b8523dc929761dd28808913f091, built on 10/08/2019 20:18 GMT (org.apache.zookeeper.server.ZooKeeperServer)
[2020-01-02 16:17:52,744] INFO Server environment:host.name=DESKTOP-6GAGLK9 (org.apache.zookeeper.server.ZooKeeperServer)
[2020-01-02 16:17:52,745] INFO Server environment:java.version=11.0.2 (org.apache.zookeeper.server.ZooKeeperServer)
[2020-01-02 16:17:52,745] INFO Server environment:java.vendor=Oracle Corporation (org.apache.zookeeper.server.ZooKeeperServer)
[2020-01-02 16:17:52,745] INFO Server environment:java.home=C:\Program Files\Java\jdk-11.0.2 (org.apache.zookeeper.server.ZooKeeperServer)
[2020-01-02 16:17:52,746] INFO Server environment:java.class.path=D:\kafka\kafka_2.11-2.4.0\libs\activation-1.1.1.jar;D:\kafka\kafka_2.11-2.4.0\libs\aopalliance-repackaged-2.5.0.jar;D:\kafka\kafka_2.11-2.4.0\libs\argparse4j-0.7.0.jar;D:\kafka\kafka_2.11-2.4.0\libs\audience-annotations-0.5.0.jar;D:\kafka\kafka_2.11-2.4.0\libs\commons-cli-1.4.jar;D:\kafka\kafka_2.11-2.4.0\libs\commons-lang3-3.8.1.jar;D:\kafka\kafka_2.11-2.4.0\libs\connect-api-2.4.0.jar;D:\kafka\kafka_2.11-2.4.0\libs\connect-basic-auth-extension-2.4.0.jar;D:\kafka\kafka_2.11-2.4.0\libs\connect-file-2.4.0.jar;D:\kafka\kafka_2.11-2.4.0\libs\connect-json-2.4.0.jar;D:\kafka\kafka_2.11-2.4.0\libs\connect-mirror-2.4.0.jar;D:\kafka\kafka_2.11-2.4.0\libs\connect-mirror-client-2.4.0.jar;D:\kafka\kafka_2.11-2.4.0\libs\connect-runtime-2.4.0.jar;D:\kafka\kafka_2.11-2.4.0\libs\connect-transforms-2.4.0.jar;D:\kafka\kafka_2.11-2.4.0\libs\guava-20.0.jar;D:\kafka\kafka_2.11-2.4.0\libs\hk2-api-2.5.0.jar;D:\kafka\kafka_2.11-2.4.0\libs\hk2-locator-2.5.0.jar;D:\kafka\kafka_2.11-2.4.0\libs\hk2-utils-2.5.0.jar;D:\kafka\kafka_2.11-2.4.0\libs\jackson-annotations-2.10.0.jar;D:\kafka\kafka_2.11-2.4.0\libs\jackson-core-2.10.0.jar;D:\kafka\kafka_2.11-2.4.0\libs\jackson-databind-2.10.0.jar;D:\kafka\kafka_2.11-2.4.0\libs\jackson-dataformat-csv-2.10.0.jar;D:\kafka\kafka_2.11-2.4.0\libs\jackson-datatype-jdk8-2.10.0.jar;D:\kafka\kafka_2.11-2.4.0\libs\jackson-jaxrs-base-2.10.0.jar;D:\kafka\kafka_2.11-2.4.0\libs\jackson-jaxrs-json-provider-2.10.0.jar;D:\kafka\kafka_2.11-2.4.0\libs\jackson-module-jaxb-annotations-2.10.0.jar;D:\kafka\kafka_2.11-2.4.0\libs\jackson-module-paranamer-2.10.0.jar;D:\kafka\kafka_2.11-2.4.0\libs\jackson-module-scala_2.11-2.10.0.jar;D:\kafka\kafka_2.11-2.4.0\libs\jakarta.activation-api-1.2.1.jar;D:\kafka\kafka_2.11-2.4.0\libs\jakarta.annotation-api-1.3.4.jar;D:\kafka\kafka_2.11-2.4.0\libs\jakarta.inject-2.5.0.jar;D:\kafka\kafka_2.11-2.4.0\libs\jakarta.ws.rs-api-2.1.5.jar;D:\kafka\kafka_2.11-2.4.0\libs\jakarta.xml.bind-api-2.3.2.jar;D:\kafka\kafka_2.11-2.4.0\libs\javassist-3.22.0-CR2.jar;D:\kafka\kafka_2.11-2.4.0\libs\javax.servlet-api-3.1.0.jar;D:\kafka\kafka_2.11-2.4.0\libs\javax.ws.rs-api-2.1.1.jar;D:\kafka\kafka_2.11-2.4.0\libs\jaxb-api-2.3.0.jar;D:\kafka\kafka_2.11-2.4.0\libs\jersey-client-2.28.jar;D:\kafka\kafka_2.11-2.4.0\libs\jersey-common-2.28.jar;D:\kafka\kafka_2.11-2.4.0\libs\jersey-container-servlet-2.28.jar;D:\kafka\kafka_2.11-2.4.0\libs\jersey-container-servlet-core-2.28.jar;D:\kafka\kafka_2.11-2.4.0\libs\jersey-hk2-2.28.jar;D:\kafka\kafka_2.11-2.4.0\libs\jersey-media-jaxb-2.28.jar;D:\kafka\kafka_2.11-2.4.0\libs\jersey-server-2.28.jar;D:\kafka\kafka_2.11-2.4.0\libs\jetty-client-9.4.20.v20190813.jar;D:\kafka\kafka_2.11-2.4.0\libs\jetty-continuation-9.4.20.v20190813.jar;D:\kafka\kafka_2.11-2.4.0\libs\jetty-http-9.4.20.v20190813.jar;D:\kafka\kafka_2.11-2.4.0\libs\jetty-io-9.4.20.v20190813.jar;D:\kafka\kafka_2.11-2.4.0\libs\jetty-security-9.4.20.v20190813.jar;D:\kafka\kafka_2.11-2.4.0\libs\jetty-server-9.4.20.v20190813.jar;D:\kafka\kafka_2.11-2.4.0\libs\jetty-servlet-9.4.20.v20190813.jar;D:\kafka\kafka_2.11-2.4.0\libs\jetty-servlets-9.4.20.v20190813.jar;D:\kafka\kafka_2.11-2.4.0\libs\jetty-util-9.4.20.v20190813.jar;D:\kafka\kafka_2.11-2.4.0\libs\jopt-simple-5.0.4.jar;D:\kafka\kafka_2.11-2.4.0\libs\kafka-clients-2.4.0.jar;D:\kafka\kafka_2.11-2.4.0\libs\kafka-log4j-appender-2.4.0.jar;D:\kafka\kafka_2.11-2.4.0\libs\kafka-streams-2.4.0.jar;D:\kafka\kafka_2.11-2.4.0\libs\kafka-streams-examples-2.4.0.jar;D:\kafka\kafka_2.11-2.4.0\libs\kafka-streams-scala_2.11-2.4.0.jar;D:\kafka\kafka_2.11-2.4.0\libs\kafka-streams-test-utils-2.4.0.jar;D:\kafka\kafka_2.11-2.4.0\libs\kafka-tools-2.4.0.jar;D:\kafka\kafka_2.11-2.4.0\libs\kafka_2.11-2.4.0-javadoc.jar;D:\kafka\kafka_2.11-2.4.0\libs\kafka_2.11-2.4.0-javadoc.jar.asc;D:\kafka\kafka_2.11-2.4.0\libs\kafka_2.11-2.4.0-scaladoc.jar;D:\kafka\kafka_2.11-2.4.0\libs\kafka_2.11-2.4.0-scaladoc.jar.asc;D:\kafka\kafka_2.11-2.4.0\libs\kafka_2.11-2.4.0-sources.jar;D:\kafka\kafka_2.11-2.4.0\libs\kafka_2.11-2.4.0-sources.jar.asc;D:\kafka\kafka_2.11-2.4.0\libs\kafka_2.11-2.4.0-test-sources.jar;D:\kafka\kafka_2.11-2.4.0\libs\kafka_2.11-2.4.0-test-sources.jar.asc;D:\kafka\kafka_2.11-2.4.0\libs\kafka_2.11-2.4.0-test.jar;D:\kafka\kafka_2.11-2.4.0\libs\kafka_2.11-2.4.0-test.jar.asc;D:\kafka\kafka_2.11-2.4.0\libs\kafka_2.11-2.4.0.jar;D:\kafka\kafka_2.11-2.4.0\libs\kafka_2.11-2.4.0.jar.asc;D:\kafka\kafka_2.11-2.4.0\libs\log4j-1.2.17.jar;D:\kafka\kafka_2.11-2.4.0\libs\lz4-java-1.6.0.jar;D:\kafka\kafka_2.11-2.4.0\libs\maven-artifact-3.6.1.jar;D:\kafka\kafka_2.11-2.4.0\libs\metrics-core-2.2.0.jar;D:\kafka\kafka_2.11-2.4.0\libs\netty-buffer-4.1.42.Final.jar;D:\kafka\kafka_2.11-2.4.0\libs\netty-codec-4.1.42.Final.jar;D:\kafka\kafka_2.11-2.4.0\libs\netty-common-4.1.42.Final.jar;D:\kafka\kafka_2.11-2.4.0\libs\netty-handler-4.1.42.Final.jar;D:\kafka\kafka_2.11-2.4.0\libs\netty-resolver-4.1.42.Final.jar;D:\kafka\kafka_2.11-2.4.0\libs\netty-transport-4.1.42.Final.jar;D:\kafka\kafka_2.11-2.4.0\libs\netty-transport-native-epoll-4.1.42.Final.jar;D:\kafka\kafka_2.11-2.4.0\libs\netty-transport-native-unix-common-4.1.42.Final.jar;D:\kafka\kafka_2.11-2.4.0\libs\osgi-resource-locator-1.0.1.jar;D:\kafka\kafka_2.11-2.4.0\libs\paranamer-2.8.jar;D:\kafka\kafka_2.11-2.4.0\libs\plexus-utils-3.2.0.jar;D:\kafka\kafka_2.11-2.4.0\libs\reflections-0.9.11.jar;D:\kafka\kafka_2.11-2.4.0\libs\rocksdbjni-5.18.3.jar;D:\kafka\kafka_2.11-2.4.0\libs\scala-collection-compat_2.11-2.1.2.jar;D:\kafka\kafka_2.11-2.4.0\libs\scala-java8-compat_2.11-0.9.0.jar;D:\kafka\kafka_2.11-2.4.0\libs\scala-library-2.11.12.jar;D:\kafka\kafka_2.11-2.4.0\libs\scala-logging_2.11-3.9.2.jar;D:\kafka\kafka_2.11-2.4.0\libs\scala-reflect-2.11.12.jar;D:\kafka\kafka_2.11-2.4.0\libs\slf4j-api-1.7.28.jar;D:\kafka\kafka_2.11-2.4.0\libs\slf4j-log4j12-1.7.28.jar;D:\kafka\kafka_2.11-2.4.0\libs\snappy-java-1.1.7.3.jar;D:\kafka\kafka_2.11-2.4.0\libs\validation-api-2.0.1.Final.jar;D:\kafka\kafka_2.11-2.4.0\libs\zookeeper-3.5.6.jar;D:\kafka\kafka_2.11-2.4.0\libs\zookeeper-jute-3.5.6.jar;D:\kafka\kafka_2.11-2.4.0\libs\zstd-jni-1.4.3-1.jar (org.apache.zookeeper.server.ZooKeeperServer)
[2020-01-02 16:17:52,747] INFO Server environment:java.library.path=C:\Program Files\Java\jdk-11.0.2\bin;C:\Windows\Sun\Java\bin;C:\Windows\system32;C:\Windows;C:\Python37\Scripts\;C:\Python37\;C:\Windows\system32;C:\Windows;C:\Windows\System32\Wbem;C:\Windows\System32\WindowsPowerShell\v1.0\;C:\Windows\System32\OpenSSH\;C:\Program Files\Git\cmd;C:\Users\ewenliu\AppData\Local\Microsoft\WindowsApps;C:\Program Files\MySQL\MySQL Server 5.7\bin;C:\Program Files\Java\jdk-11.0.2\bin;;. (org.apache.zookeeper.server.ZooKeeperServer)
[2020-01-02 16:17:52,747] INFO Server environment:java.io.tmpdir=C:\Users\ewenliu\AppData\Local\Temp\ (org.apache.zookeeper.server.ZooKeeperServer)
[2020-01-02 16:17:52,748] INFO Server environment:java.compiler=<NA> (org.apache.zookeeper.server.ZooKeeperServer)
[2020-01-02 16:17:52,749] INFO Server environment:os.name=Windows 10 (org.apache.zookeeper.server.ZooKeeperServer)
[2020-01-02 16:17:52,750] INFO Server environment:os.arch=amd64 (org.apache.zookeeper.server.ZooKeeperServer)
[2020-01-02 16:17:52,750] INFO Server environment:os.version=10.0 (org.apache.zookeeper.server.ZooKeeperServer)
[2020-01-02 16:17:52,750] INFO Server environment:user.name=ewenliu (org.apache.zookeeper.server.ZooKeeperServer)
[2020-01-02 16:17:52,751] INFO Server environment:user.home=C:\Users\ewenliu (org.apache.zookeeper.server.ZooKeeperServer)
[2020-01-02 16:17:52,751] INFO Server environment:user.dir=D:\kafka\kafka_2.11-2.4.0\bin\windows (org.apache.zookeeper.server.ZooKeeperServer)
[2020-01-02 16:17:52,752] INFO Server environment:os.memory.free=491MB (org.apache.zookeeper.server.ZooKeeperServer)
[2020-01-02 16:17:52,752] INFO Server environment:os.memory.max=512MB (org.apache.zookeeper.server.ZooKeeperServer)
[2020-01-02 16:17:52,752] INFO Server environment:os.memory.total=512MB (org.apache.zookeeper.server.ZooKeeperServer)
[2020-01-02 16:17:52,754] INFO minSessionTimeout set to 6000 (org.apache.zookeeper.server.ZooKeeperServer)
[2020-01-02 16:17:52,754] INFO maxSessionTimeout set to 60000 (org.apache.zookeeper.server.ZooKeeperServer)
[2020-01-02 16:17:52,755] INFO Created server with tickTime 3000 minSessionTimeout 6000 maxSessionTimeout 60000 datadir \tmp\zookeeper\version-2 snapdir \tmp\zookeeper\version-2 (org.apache.zookeeper.server.ZooKeeperServer)
[2020-01-02 16:17:52,770] INFO Using org.apache.zookeeper.server.NIOServerCnxnFactory as server connection factory (org.apache.zookeeper.server.ServerCnxnFactory)
[2020-01-02 16:17:52,772] INFO Configuring NIO connection handler with 10s sessionless connection timeout, 2 selector thread(s), 16 worker threads, and 64 kB direct buffers. (org.apache.zookeeper.server.NIOServerCnxnFactory)
[2020-01-02 16:17:52,776] INFO binding to port 0.0.0.0/0.0.0.0:2181 (org.apache.zookeeper.server.NIOServerCnxnFactory)
[2020-01-02 16:17:52,791] INFO zookeeper.snapshotSizeFactor = 0.33 (org.apache.zookeeper.server.ZKDatabase)
[2020-01-02 16:17:52,794] INFO Snapshotting: 0x0 to \tmp\zookeeper\version-2\snapshot.0 (org.apache.zookeeper.server.persistence.FileTxnSnapLog)
[2020-01-02 16:17:52,798] INFO Snapshotting: 0x0 to \tmp\zookeeper\version-2\snapshot.0 (org.apache.zookeeper.server.persistence.FileTxnSnapLog)
[2020-01-02 16:17:52,819] INFO Using checkIntervalMs=60000 maxPerMinute=10000 (org.apache.zookeeper.server.ContainerManager)
[2020-01-02 16:18:01,972] INFO Creating new log file: log.1 (org.apache.zookeeper.server.persistence.FileTxnLog)

```

### 3.2 启动Kafka

进入kafka目录\bin\windows，我的目录D:\kafka\kafka_2.11-2.4.0\bin\windows，执行如下：

```cmd
kafka-server-start.bat ..\..\config\server.properties

D:\kafka\kafka_2.11-2.4.0\bin\windows>kafka-server-start.bat ..\..\config\server.properties
[2020-01-02 16:18:01,633] INFO Registered kafka:type=kafka.Log4jController MBean (kafka.utils.Log4jControllerRegistration$)
[2020-01-02 16:18:01,894] INFO starting (kafka.server.KafkaServer)
[2020-01-02 16:18:01,895] INFO Connecting to zookeeper on localhost:2181 (kafka.server.KafkaServer)
[2020-01-02 16:18:01,912] INFO [ZooKeeperClient Kafka server] Initializing a new session to localhost:2181. (kafka.zookeeper.ZooKeeperClient)
[2020-01-02 16:18:01,918] INFO Client environment:zookeeper.version=3.5.6-c11b7e26bc554b8523dc929761dd28808913f091, built on 10/08/2019 20:18 GMT (org.apache.zookeeper.ZooKeeper)
[2020-01-02 16:18:01,918] INFO Client environment:host.name=DESKTOP-6GAGLK9 (org.apache.zookeeper.ZooKeeper)
[2020-01-02 16:18:01,918] INFO Client environment:java.version=11.0.2 (org.apache.zookeeper.ZooKeeper)
[2020-01-02 16:18:01,919] INFO Client environment:java.vendor=Oracle Corporation (org.apache.zookeeper.ZooKeeper)
[2020-01-02 16:18:01,919] INFO Client environment:java.home=C:\Program Files\Java\jdk-11.0.2 (org.apache.zookeeper.ZooKeeper)
[2020-01-02 16:18:01,919] INFO Client environment:java.class.path=D:\kafka\kafka_2.11-2.4.0\libs\activation-1.1.1.jar;D:\kafka\kafka_2.11-2.4.0\libs\aopalliance-repackaged-2.5.0.jar;D:\kafka\kafka_2.11-2.4.0\libs\argparse4j-0.7.0.jar;D:\kafka\kafka_2.11-2.4.0\libs\audience-annotations-0.5.0.jar;D:\kafka\kafka_2.11-2.4.0\libs\commons-cli-1.4.jar;D:\kafka\kafka_2.11-2.4.0\libs\commons-lang3-3.8.1.jar;D:\kafka\kafka_2.11-2.4.0\libs\connect-api-2.4.0.jar;D:\kafka\kafka_2.11-2.4.0\libs\connect-basic-auth-extension-2.4.0.jar;D:\kafka\kafka_2.11-2.4.0\libs\connect-file-2.4.0.jar;D:\kafka\kafka_2.11-2.4.0\libs\connect-json-2.4.0.jar;D:\kafka\kafka_2.11-2.4.0\libs\connect-mirror-2.4.0.jar;D:\kafka\kafka_2.11-2.4.0\libs\connect-mirror-client-2.4.0.jar;D:\kafka\kafka_2.11-2.4.0\libs\connect-runtime-2.4.0.jar;D:\kafka\kafka_2.11-2.4.0\libs\connect-transforms-2.4.0.jar;D:\kafka\kafka_2.11-2.4.0\libs\guava-20.0.jar;D:\kafka\kafka_2.11-2.4.0\libs\hk2-api-2.5.0.jar;D:\kafka\kafka_2.11-2.4.0\libs\hk2-locator-2.5.0.jar;D:\kafka\kafka_2.11-2.4.0\libs\hk2-utils-2.5.0.jar;D:\kafka\kafka_2.11-2.4.0\libs\jackson-annotations-2.10.0.jar;D:\kafka\kafka_2.11-2.4.0\libs\jackson-core-2.10.0.jar;D:\kafka\kafka_2.11-2.4.0\libs\jackson-databind-2.10.0.jar;D:\kafka\kafka_2.11-2.4.0\libs\jackson-dataformat-csv-2.10.0.jar;D:\kafka\kafka_2.11-2.4.0\libs\jackson-datatype-jdk8-2.10.0.jar;D:\kafka\kafka_2.11-2.4.0\libs\jackson-jaxrs-base-2.10.0.jar;D:\kafka\kafka_2.11-2.4.0\libs\jackson-jaxrs-json-provider-2.10.0.jar;D:\kafka\kafka_2.11-2.4.0\libs\jackson-module-jaxb-annotations-2.10.0.jar;D:\kafka\kafka_2.11-2.4.0\libs\jackson-module-paranamer-2.10.0.jar;D:\kafka\kafka_2.11-2.4.0\libs\jackson-module-scala_2.11-2.10.0.jar;D:\kafka\kafka_2.11-2.4.0\libs\jakarta.activation-api-1.2.1.jar;D:\kafka\kafka_2.11-2.4.0\libs\jakarta.annotation-api-1.3.4.jar;D:\kafka\kafka_2.11-2.4.0\libs\jakarta.inject-2.5.0.jar;D:\kafka\kafka_2.11-2.4.0\libs\jakarta.ws.rs-api-2.1.5.jar;D:\kafka\kafka_2.11-2.4.0\libs\jakarta.xml.bind-api-2.3.2.jar;D:\kafka\kafka_2.11-2.4.0\libs\javassist-3.22.0-CR2.jar;D:\kafka\kafka_2.11-2.4.0\libs\javax.servlet-api-3.1.0.jar;D:\kafka\kafka_2.11-2.4.0\libs\javax.ws.rs-api-2.1.1.jar;D:\kafka\kafka_2.11-2.4.0\libs\jaxb-api-2.3.0.jar;D:\kafka\kafka_2.11-2.4.0\libs\jersey-client-2.28.jar;D:\kafka\kafka_2.11-2.4.0\libs\jersey-common-2.28.jar;D:\kafka\kafka_2.11-2.4.0\libs\jersey-container-servlet-2.28.jar;D:\kafka\kafka_2.11-2.4.0\libs\jersey-container-servlet-core-2.28.jar;D:\kafka\kafka_2.11-2.4.0\libs\jersey-hk2-2.28.jar;D:\kafka\kafka_2.11-2.4.0\libs\jersey-media-jaxb-2.28.jar;D:\kafka\kafka_2.11-2.4.0\libs\jersey-server-2.28.jar;D:\kafka\kafka_2.11-2.4.0\libs\jetty-client-9.4.20.v20190813.jar;D:\kafka\kafka_2.11-2.4.0\libs\jetty-continuation-9.4.20.v20190813.jar;D:\kafka\kafka_2.11-2.4.0\libs\jetty-http-9.4.20.v20190813.jar;D:\kafka\kafka_2.11-2.4.0\libs\jetty-io-9.4.20.v20190813.jar;D:\kafka\kafka_2.11-2.4.0\libs\jetty-security-9.4.20.v20190813.jar;D:\kafka\kafka_2.11-2.4.0\libs\jetty-server-9.4.20.v20190813.jar;D:\kafka\kafka_2.11-2.4.0\libs\jetty-servlet-9.4.20.v20190813.jar;D:\kafka\kafka_2.11-2.4.0\libs\jetty-servlets-9.4.20.v20190813.jar;D:\kafka\kafka_2.11-2.4.0\libs\jetty-util-9.4.20.v20190813.jar;D:\kafka\kafka_2.11-2.4.0\libs\jopt-simple-5.0.4.jar;D:\kafka\kafka_2.11-2.4.0\libs\kafka-clients-2.4.0.jar;D:\kafka\kafka_2.11-2.4.0\libs\kafka-log4j-appender-2.4.0.jar;D:\kafka\kafka_2.11-2.4.0\libs\kafka-streams-2.4.0.jar;D:\kafka\kafka_2.11-2.4.0\libs\kafka-streams-examples-2.4.0.jar;D:\kafka\kafka_2.11-2.4.0\libs\kafka-streams-scala_2.11-2.4.0.jar;D:\kafka\kafka_2.11-2.4.0\libs\kafka-streams-test-utils-2.4.0.jar;D:\kafka\kafka_2.11-2.4.0\libs\kafka-tools-2.4.0.jar;D:\kafka\kafka_2.11-2.4.0\libs\kafka_2.11-2.4.0-javadoc.jar;D:\kafka\kafka_2.11-2.4.0\libs\kafka_2.11-2.4.0-javadoc.jar.asc;D:\kafka\kafka_2.11-2.4.0\libs\kafka_2.11-2.4.0-scaladoc.jar;D:\kafka\kafka_2.11-2.4.0\libs\kafka_2.11-2.4.0-scaladoc.jar.asc;D:\kafka\kafka_2.11-2.4.0\libs\kafka_2.11-2.4.0-sources.jar;D:\kafka\kafka_2.11-2.4.0\libs\kafka_2.11-2.4.0-sources.jar.asc;D:\kafka\kafka_2.11-2.4.0\libs\kafka_2.11-2.4.0-test-sources.jar;D:\kafka\kafka_2.11-2.4.0\libs\kafka_2.11-2.4.0-test-sources.jar.asc;D:\kafka\kafka_2.11-2.4.0\libs\kafka_2.11-2.4.0-test.jar;D:\kafka\kafka_2.11-2.4.0\libs\kafka_2.11-2.4.0-test.jar.asc;D:\kafka\kafka_2.11-2.4.0\libs\kafka_2.11-2.4.0.jar;D:\kafka\kafka_2.11-2.4.0\libs\kafka_2.11-2.4.0.jar.asc;D:\kafka\kafka_2.11-2.4.0\libs\log4j-1.2.17.jar;D:\kafka\kafka_2.11-2.4.0\libs\lz4-java-1.6.0.jar;D:\kafka\kafka_2.11-2.4.0\libs\maven-artifact-3.6.1.jar;D:\kafka\kafka_2.11-2.4.0\libs\metrics-core-2.2.0.jar;D:\kafka\kafka_2.11-2.4.0\libs\netty-buffer-4.1.42.Final.jar;D:\kafka\kafka_2.11-2.4.0\libs\netty-codec-4.1.42.Final.jar;D:\kafka\kafka_2.11-2.4.0\libs\netty-common-4.1.42.Final.jar;D:\kafka\kafka_2.11-2.4.0\libs\netty-handler-4.1.42.Final.jar;D:\kafka\kafka_2.11-2.4.0\libs\netty-resolver-4.1.42.Final.jar;D:\kafka\kafka_2.11-2.4.0\libs\netty-transport-4.1.42.Final.jar;D:\kafka\kafka_2.11-2.4.0\libs\netty-transport-native-epoll-4.1.42.Final.jar;D:\kafka\kafka_2.11-2.4.0\libs\netty-transport-native-unix-common-4.1.42.Final.jar;D:\kafka\kafka_2.11-2.4.0\libs\osgi-resource-locator-1.0.1.jar;D:\kafka\kafka_2.11-2.4.0\libs\paranamer-2.8.jar;D:\kafka\kafka_2.11-2.4.0\libs\plexus-utils-3.2.0.jar;D:\kafka\kafka_2.11-2.4.0\libs\reflections-0.9.11.jar;D:\kafka\kafka_2.11-2.4.0\libs\rocksdbjni-5.18.3.jar;D:\kafka\kafka_2.11-2.4.0\libs\scala-collection-compat_2.11-2.1.2.jar;D:\kafka\kafka_2.11-2.4.0\libs\scala-java8-compat_2.11-0.9.0.jar;D:\kafka\kafka_2.11-2.4.0\libs\scala-library-2.11.12.jar;D:\kafka\kafka_2.11-2.4.0\libs\scala-logging_2.11-3.9.2.jar;D:\kafka\kafka_2.11-2.4.0\libs\scala-reflect-2.11.12.jar;D:\kafka\kafka_2.11-2.4.0\libs\slf4j-api-1.7.28.jar;D:\kafka\kafka_2.11-2.4.0\libs\slf4j-log4j12-1.7.28.jar;D:\kafka\kafka_2.11-2.4.0\libs\snappy-java-1.1.7.3.jar;D:\kafka\kafka_2.11-2.4.0\libs\validation-api-2.0.1.Final.jar;D:\kafka\kafka_2.11-2.4.0\libs\zookeeper-3.5.6.jar;D:\kafka\kafka_2.11-2.4.0\libs\zookeeper-jute-3.5.6.jar;D:\kafka\kafka_2.11-2.4.0\libs\zstd-jni-1.4.3-1.jar (org.apache.zookeeper.ZooKeeper)
[2020-01-02 16:18:01,921] INFO Client environment:java.library.path=C:\Program Files\Java\jdk-11.0.2\bin;C:\Windows\Sun\Java\bin;C:\Windows\system32;C:\Windows;C:\Python37\Scripts\;C:\Python37\;C:\Windows\system32;C:\Windows;C:\Windows\System32\Wbem;C:\Windows\System32\WindowsPowerShell\v1.0\;C:\Windows\System32\OpenSSH\;C:\Program Files\Git\cmd;C:\Users\ewenliu\AppData\Local\Microsoft\WindowsApps;C:\Program Files\MySQL\MySQL Server 5.7\bin;C:\Program Files\Java\jdk-11.0.2\bin;;. (org.apache.zookeeper.ZooKeeper)
[2020-01-02 16:18:01,923] INFO Client environment:java.io.tmpdir=C:\Users\ewenliu\AppData\Local\Temp\ (org.apache.zookeeper.ZooKeeper)
[2020-01-02 16:18:01,923] INFO Client environment:java.compiler=<NA> (org.apache.zookeeper.ZooKeeper)
[2020-01-02 16:18:01,923] INFO Client environment:os.name=Windows 10 (org.apache.zookeeper.ZooKeeper)
[2020-01-02 16:18:01,924] INFO Client environment:os.arch=amd64 (org.apache.zookeeper.ZooKeeper)
[2020-01-02 16:18:01,924] INFO Client environment:os.version=10.0 (org.apache.zookeeper.ZooKeeper)
[2020-01-02 16:18:01,924] INFO Client environment:user.name=ewenliu (org.apache.zookeeper.ZooKeeper)
[2020-01-02 16:18:01,925] INFO Client environment:user.home=C:\Users\ewenliu (org.apache.zookeeper.ZooKeeper)
[2020-01-02 16:18:01,925] INFO Client environment:user.dir=D:\kafka\kafka_2.11-2.4.0\bin\windows (org.apache.zookeeper.ZooKeeper)
[2020-01-02 16:18:01,925] INFO Client environment:os.memory.free=978MB (org.apache.zookeeper.ZooKeeper)
[2020-01-02 16:18:01,926] INFO Client environment:os.memory.max=1024MB (org.apache.zookeeper.ZooKeeper)
[2020-01-02 16:18:01,926] INFO Client environment:os.memory.total=1024MB (org.apache.zookeeper.ZooKeeper)
[2020-01-02 16:18:01,928] INFO Initiating client connection, connectString=localhost:2181 sessionTimeout=6000 watcher=kafka.zookeeper.ZooKeeperClient$ZooKeeperClientWatcher$@14d14731 (org.apache.zookeeper.ZooKeeper)
[2020-01-02 16:18:01,934] INFO Setting -D jdk.tls.rejectClientInitiatedRenegotiation=true to disable client-initiated TLS renegotiation (org.apache.zookeeper.common.X509Util)
[2020-01-02 16:18:01,945] INFO jute.maxbuffer value is 4194304 Bytes (org.apache.zookeeper.ClientCnxnSocket)
[2020-01-02 16:18:01,952] INFO zookeeper.request.timeout value is 0. feature enabled= (org.apache.zookeeper.ClientCnxn)
[2020-01-02 16:18:01,954] INFO [ZooKeeperClient Kafka server] Waiting until connected. (kafka.zookeeper.ZooKeeperClient)
[2020-01-02 16:18:01,960] INFO Opening socket connection to server localhost/0:0:0:0:0:0:0:1:2181. Will not attempt to authenticate using SASL (unknown error) (org.apache.zookeeper.ClientCnxn)
[2020-01-02 16:18:01,963] INFO Socket connection established, initiating session, client: /0:0:0:0:0:0:0:1:49357, server: localhost/0:0:0:0:0:0:0:1:2181 (org.apache.zookeeper.ClientCnxn)
[2020-01-02 16:18:01,988] INFO Session establishment complete on server localhost/0:0:0:0:0:0:0:1:2181, sessionid = 0x100684539df0000, negotiated timeout = 6000 (org.apache.zookeeper.ClientCnxn)
[2020-01-02 16:18:01,990] INFO [ZooKeeperClient Kafka server] Connected. (kafka.zookeeper.ZooKeeperClient)
[2020-01-02 16:18:02,230] INFO Cluster ID = IJQyv_kfSGu6rmPdjmOgPw (kafka.server.KafkaServer)
[2020-01-02 16:18:02,234] WARN No meta.properties file under dir D:\tmp\kafka-logs\meta.properties (kafka.server.BrokerMetadataCheckpoint)
[2020-01-02 16:18:02,274] INFO KafkaConfig values:
        advertised.host.name = null
        advertised.listeners = null
        advertised.port = null
        alter.config.policy.class.name = null
        alter.log.dirs.replication.quota.window.num = 11
        alter.log.dirs.replication.quota.window.size.seconds = 1
        authorizer.class.name =
        auto.create.topics.enable = true
        auto.leader.rebalance.enable = true
        background.threads = 10
        broker.id = 0
        broker.id.generation.enable = true
        broker.rack = null
        client.quota.callback.class = null
        compression.type = producer
        connection.failed.authentication.delay.ms = 100
        connections.max.idle.ms = 600000
        connections.max.reauth.ms = 0
        control.plane.listener.name = null
        controlled.shutdown.enable = true
        controlled.shutdown.max.retries = 3
        controlled.shutdown.retry.backoff.ms = 5000
        controller.socket.timeout.ms = 30000
        create.topic.policy.class.name = null
        default.replication.factor = 1
        delegation.token.expiry.check.interval.ms = 3600000
        delegation.token.expiry.time.ms = 86400000
        delegation.token.master.key = null
        delegation.token.max.lifetime.ms = 604800000
        delete.records.purgatory.purge.interval.requests = 1
        delete.topic.enable = true
        fetch.purgatory.purge.interval.requests = 1000
        group.initial.rebalance.delay.ms = 0
        group.max.session.timeout.ms = 1800000
        group.max.size = 2147483647
        group.min.session.timeout.ms = 6000
        host.name =
        inter.broker.listener.name = null
        inter.broker.protocol.version = 2.4-IV1
        kafka.metrics.polling.interval.secs = 10
        kafka.metrics.reporters = []
        leader.imbalance.check.interval.seconds = 300
        leader.imbalance.per.broker.percentage = 10
        listener.security.protocol.map = PLAINTEXT:PLAINTEXT,SSL:SSL,SASL_PLAINTEXT:SASL_PLAINTEXT,SASL_SSL:SASL_SSL
        listeners = null
        log.cleaner.backoff.ms = 15000
        log.cleaner.dedupe.buffer.size = 134217728
        log.cleaner.delete.retention.ms = 86400000
        log.cleaner.enable = true
        log.cleaner.io.buffer.load.factor = 0.9
        log.cleaner.io.buffer.size = 524288
        log.cleaner.io.max.bytes.per.second = 1.7976931348623157E308
        log.cleaner.max.compaction.lag.ms = 9223372036854775807
        log.cleaner.min.cleanable.ratio = 0.5
        log.cleaner.min.compaction.lag.ms = 0
        log.cleaner.threads = 1
        log.cleanup.policy = [delete]
        log.dir = /tmp/kafka-logs
        log.dirs = /tmp/kafka-logs
        log.flush.interval.messages = 9223372036854775807
        log.flush.interval.ms = null
        log.flush.offset.checkpoint.interval.ms = 60000
        log.flush.scheduler.interval.ms = 9223372036854775807
        log.flush.start.offset.checkpoint.interval.ms = 60000
        log.index.interval.bytes = 4096
        log.index.size.max.bytes = 10485760
        log.message.downconversion.enable = true
        log.message.format.version = 2.4-IV1
        log.message.timestamp.difference.max.ms = 9223372036854775807
        log.message.timestamp.type = CreateTime
        log.preallocate = false
        log.retention.bytes = -1
        log.retention.check.interval.ms = 300000
        log.retention.hours = 168
        log.retention.minutes = null
        log.retention.ms = null
        log.roll.hours = 168
        log.roll.jitter.hours = 0
        log.roll.jitter.ms = null
        log.roll.ms = null
        log.segment.bytes = 1073741824
        log.segment.delete.delay.ms = 60000
        max.connections = 2147483647
        max.connections.per.ip = 2147483647
        max.connections.per.ip.overrides =
        max.incremental.fetch.session.cache.slots = 1000
        message.max.bytes = 1000012
        metric.reporters = []
        metrics.num.samples = 2
        metrics.recording.level = INFO
        metrics.sample.window.ms = 30000
        min.insync.replicas = 1
        num.io.threads = 8
        num.network.threads = 3
        num.partitions = 1
        num.recovery.threads.per.data.dir = 1
        num.replica.alter.log.dirs.threads = null
        num.replica.fetchers = 1
        offset.metadata.max.bytes = 4096
        offsets.commit.required.acks = -1
        offsets.commit.timeout.ms = 5000
        offsets.load.buffer.size = 5242880
        offsets.retention.check.interval.ms = 600000
        offsets.retention.minutes = 10080
        offsets.topic.compression.codec = 0
        offsets.topic.num.partitions = 50
        offsets.topic.replication.factor = 1
        offsets.topic.segment.bytes = 104857600
        password.encoder.cipher.algorithm = AES/CBC/PKCS5Padding
        password.encoder.iterations = 4096
        password.encoder.key.length = 128
        password.encoder.keyfactory.algorithm = null
        password.encoder.old.secret = null
        password.encoder.secret = null
        port = 9092
        principal.builder.class = null
        producer.purgatory.purge.interval.requests = 1000
        queued.max.request.bytes = -1
        queued.max.requests = 500
        quota.consumer.default = 9223372036854775807
        quota.producer.default = 9223372036854775807
        quota.window.num = 11
        quota.window.size.seconds = 1
        replica.fetch.backoff.ms = 1000
        replica.fetch.max.bytes = 1048576
        replica.fetch.min.bytes = 1
        replica.fetch.response.max.bytes = 10485760
        replica.fetch.wait.max.ms = 500
        replica.high.watermark.checkpoint.interval.ms = 5000
        replica.lag.time.max.ms = 10000
        replica.selector.class = null
        replica.socket.receive.buffer.bytes = 65536
        replica.socket.timeout.ms = 30000
        replication.quota.window.num = 11
        replication.quota.window.size.seconds = 1
        request.timeout.ms = 30000
        reserved.broker.max.id = 1000
        sasl.client.callback.handler.class = null
        sasl.enabled.mechanisms = [GSSAPI]
        sasl.jaas.config = null
        sasl.kerberos.kinit.cmd = /usr/bin/kinit
        sasl.kerberos.min.time.before.relogin = 60000
        sasl.kerberos.principal.to.local.rules = [DEFAULT]
        sasl.kerberos.service.name = null
        sasl.kerberos.ticket.renew.jitter = 0.05
        sasl.kerberos.ticket.renew.window.factor = 0.8
        sasl.login.callback.handler.class = null
        sasl.login.class = null
        sasl.login.refresh.buffer.seconds = 300
        sasl.login.refresh.min.period.seconds = 60
        sasl.login.refresh.window.factor = 0.8
        sasl.login.refresh.window.jitter = 0.05
        sasl.mechanism.inter.broker.protocol = GSSAPI
        sasl.server.callback.handler.class = null
        security.inter.broker.protocol = PLAINTEXT
        security.providers = null
        socket.receive.buffer.bytes = 102400
        socket.request.max.bytes = 104857600
        socket.send.buffer.bytes = 102400
        ssl.cipher.suites = []
        ssl.client.auth = none
        ssl.enabled.protocols = [TLSv1.2, TLSv1.1, TLSv1]
        ssl.endpoint.identification.algorithm = https
        ssl.key.password = null
        ssl.keymanager.algorithm = SunX509
        ssl.keystore.location = null
        ssl.keystore.password = null
        ssl.keystore.type = JKS
        ssl.principal.mapping.rules = DEFAULT
        ssl.protocol = TLS
        ssl.provider = null
        ssl.secure.random.implementation = null
        ssl.trustmanager.algorithm = PKIX
        ssl.truststore.location = null
        ssl.truststore.password = null
        ssl.truststore.type = JKS
        transaction.abort.timed.out.transaction.cleanup.interval.ms = 60000
        transaction.max.timeout.ms = 900000
        transaction.remove.expired.transaction.cleanup.interval.ms = 3600000
        transaction.state.log.load.buffer.size = 5242880
        transaction.state.log.min.isr = 1
        transaction.state.log.num.partitions = 50
        transaction.state.log.replication.factor = 1
        transaction.state.log.segment.bytes = 104857600
        transactional.id.expiration.ms = 604800000
        unclean.leader.election.enable = false
        zookeeper.connect = localhost:2181
        zookeeper.connection.timeout.ms = 6000
        zookeeper.max.in.flight.requests = 10
        zookeeper.session.timeout.ms = 6000
        zookeeper.set.acl = false
        zookeeper.sync.time.ms = 2000
 (kafka.server.KafkaConfig)
[2020-01-02 16:18:02,283] INFO KafkaConfig values:
        advertised.host.name = null
        advertised.listeners = null
        advertised.port = null
        alter.config.policy.class.name = null
        alter.log.dirs.replication.quota.window.num = 11
        alter.log.dirs.replication.quota.window.size.seconds = 1
        authorizer.class.name =
        auto.create.topics.enable = true
        auto.leader.rebalance.enable = true
        background.threads = 10
        broker.id = 0
        broker.id.generation.enable = true
        broker.rack = null
        client.quota.callback.class = null
        compression.type = producer
        connection.failed.authentication.delay.ms = 100
        connections.max.idle.ms = 600000
        connections.max.reauth.ms = 0
        control.plane.listener.name = null
        controlled.shutdown.enable = true
        controlled.shutdown.max.retries = 3
        controlled.shutdown.retry.backoff.ms = 5000
        controller.socket.timeout.ms = 30000
        create.topic.policy.class.name = null
        default.replication.factor = 1
        delegation.token.expiry.check.interval.ms = 3600000
        delegation.token.expiry.time.ms = 86400000
        delegation.token.master.key = null
        delegation.token.max.lifetime.ms = 604800000
        delete.records.purgatory.purge.interval.requests = 1
        delete.topic.enable = true
        fetch.purgatory.purge.interval.requests = 1000
        group.initial.rebalance.delay.ms = 0
        group.max.session.timeout.ms = 1800000
        group.max.size = 2147483647
        group.min.session.timeout.ms = 6000
        host.name =
        inter.broker.listener.name = null
        inter.broker.protocol.version = 2.4-IV1
        kafka.metrics.polling.interval.secs = 10
        kafka.metrics.reporters = []
        leader.imbalance.check.interval.seconds = 300
        leader.imbalance.per.broker.percentage = 10
        listener.security.protocol.map = PLAINTEXT:PLAINTEXT,SSL:SSL,SASL_PLAINTEXT:SASL_PLAINTEXT,SASL_SSL:SASL_SSL
        listeners = null
        log.cleaner.backoff.ms = 15000
        log.cleaner.dedupe.buffer.size = 134217728
        log.cleaner.delete.retention.ms = 86400000
        log.cleaner.enable = true
        log.cleaner.io.buffer.load.factor = 0.9
        log.cleaner.io.buffer.size = 524288
        log.cleaner.io.max.bytes.per.second = 1.7976931348623157E308
        log.cleaner.max.compaction.lag.ms = 9223372036854775807
        log.cleaner.min.cleanable.ratio = 0.5
        log.cleaner.min.compaction.lag.ms = 0
        log.cleaner.threads = 1
        log.cleanup.policy = [delete]
        log.dir = /tmp/kafka-logs
        log.dirs = /tmp/kafka-logs
        log.flush.interval.messages = 9223372036854775807
        log.flush.interval.ms = null
        log.flush.offset.checkpoint.interval.ms = 60000
        log.flush.scheduler.interval.ms = 9223372036854775807
        log.flush.start.offset.checkpoint.interval.ms = 60000
        log.index.interval.bytes = 4096
        log.index.size.max.bytes = 10485760
        log.message.downconversion.enable = true
        log.message.format.version = 2.4-IV1
        log.message.timestamp.difference.max.ms = 9223372036854775807
        log.message.timestamp.type = CreateTime
        log.preallocate = false
        log.retention.bytes = -1
        log.retention.check.interval.ms = 300000
        log.retention.hours = 168
        log.retention.minutes = null
        log.retention.ms = null
        log.roll.hours = 168
        log.roll.jitter.hours = 0
        log.roll.jitter.ms = null
        log.roll.ms = null
        log.segment.bytes = 1073741824
        log.segment.delete.delay.ms = 60000
        max.connections = 2147483647
        max.connections.per.ip = 2147483647
        max.connections.per.ip.overrides =
        max.incremental.fetch.session.cache.slots = 1000
        message.max.bytes = 1000012
        metric.reporters = []
        metrics.num.samples = 2
        metrics.recording.level = INFO
        metrics.sample.window.ms = 30000
        min.insync.replicas = 1
        num.io.threads = 8
        num.network.threads = 3
        num.partitions = 1
        num.recovery.threads.per.data.dir = 1
        num.replica.alter.log.dirs.threads = null
        num.replica.fetchers = 1
        offset.metadata.max.bytes = 4096
        offsets.commit.required.acks = -1
        offsets.commit.timeout.ms = 5000
        offsets.load.buffer.size = 5242880
        offsets.retention.check.interval.ms = 600000
        offsets.retention.minutes = 10080
        offsets.topic.compression.codec = 0
        offsets.topic.num.partitions = 50
        offsets.topic.replication.factor = 1
        offsets.topic.segment.bytes = 104857600
        password.encoder.cipher.algorithm = AES/CBC/PKCS5Padding
        password.encoder.iterations = 4096
        password.encoder.key.length = 128
        password.encoder.keyfactory.algorithm = null
        password.encoder.old.secret = null
        password.encoder.secret = null
        port = 9092
        principal.builder.class = null
        producer.purgatory.purge.interval.requests = 1000
        queued.max.request.bytes = -1
        queued.max.requests = 500
        quota.consumer.default = 9223372036854775807
        quota.producer.default = 9223372036854775807
        quota.window.num = 11
        quota.window.size.seconds = 1
        replica.fetch.backoff.ms = 1000
        replica.fetch.max.bytes = 1048576
        replica.fetch.min.bytes = 1
        replica.fetch.response.max.bytes = 10485760
        replica.fetch.wait.max.ms = 500
        replica.high.watermark.checkpoint.interval.ms = 5000
        replica.lag.time.max.ms = 10000
        replica.selector.class = null
        replica.socket.receive.buffer.bytes = 65536
        replica.socket.timeout.ms = 30000
        replication.quota.window.num = 11
        replication.quota.window.size.seconds = 1
        request.timeout.ms = 30000
        reserved.broker.max.id = 1000
        sasl.client.callback.handler.class = null
        sasl.enabled.mechanisms = [GSSAPI]
        sasl.jaas.config = null
        sasl.kerberos.kinit.cmd = /usr/bin/kinit
        sasl.kerberos.min.time.before.relogin = 60000
        sasl.kerberos.principal.to.local.rules = [DEFAULT]
        sasl.kerberos.service.name = null
        sasl.kerberos.ticket.renew.jitter = 0.05
        sasl.kerberos.ticket.renew.window.factor = 0.8
        sasl.login.callback.handler.class = null
        sasl.login.class = null
        sasl.login.refresh.buffer.seconds = 300
        sasl.login.refresh.min.period.seconds = 60
        sasl.login.refresh.window.factor = 0.8
        sasl.login.refresh.window.jitter = 0.05
        sasl.mechanism.inter.broker.protocol = GSSAPI
        sasl.server.callback.handler.class = null
        security.inter.broker.protocol = PLAINTEXT
        security.providers = null
        socket.receive.buffer.bytes = 102400
        socket.request.max.bytes = 104857600
        socket.send.buffer.bytes = 102400
        ssl.cipher.suites = []
        ssl.client.auth = none
        ssl.enabled.protocols = [TLSv1.2, TLSv1.1, TLSv1]
        ssl.endpoint.identification.algorithm = https
        ssl.key.password = null
        ssl.keymanager.algorithm = SunX509
        ssl.keystore.location = null
        ssl.keystore.password = null
        ssl.keystore.type = JKS
        ssl.principal.mapping.rules = DEFAULT
        ssl.protocol = TLS
        ssl.provider = null
        ssl.secure.random.implementation = null
        ssl.trustmanager.algorithm = PKIX
        ssl.truststore.location = null
        ssl.truststore.password = null
        ssl.truststore.type = JKS
        transaction.abort.timed.out.transaction.cleanup.interval.ms = 60000
        transaction.max.timeout.ms = 900000
        transaction.remove.expired.transaction.cleanup.interval.ms = 3600000
        transaction.state.log.load.buffer.size = 5242880
        transaction.state.log.min.isr = 1
        transaction.state.log.num.partitions = 50
        transaction.state.log.replication.factor = 1
        transaction.state.log.segment.bytes = 104857600
        transactional.id.expiration.ms = 604800000
        unclean.leader.election.enable = false
        zookeeper.connect = localhost:2181
        zookeeper.connection.timeout.ms = 6000
        zookeeper.max.in.flight.requests = 10
        zookeeper.session.timeout.ms = 6000
        zookeeper.set.acl = false
        zookeeper.sync.time.ms = 2000
 (kafka.server.KafkaConfig)
[2020-01-02 16:18:02,304] INFO [ThrottledChannelReaper-Fetch]: Starting (kafka.server.ClientQuotaManager$ThrottledChannelReaper)
[2020-01-02 16:18:02,307] INFO [ThrottledChannelReaper-Produce]: Starting (kafka.server.ClientQuotaManager$ThrottledChannelReaper)
[2020-01-02 16:18:02,308] INFO [ThrottledChannelReaper-Request]: Starting (kafka.server.ClientQuotaManager$ThrottledChannelReaper)
[2020-01-02 16:18:02,326] INFO Log directory D:\tmp\kafka-logs not found, creating it. (kafka.log.LogManager)
[2020-01-02 16:18:02,333] INFO Loading logs. (kafka.log.LogManager)
[2020-01-02 16:18:02,339] INFO Logs loading complete in 5 ms. (kafka.log.LogManager)
[2020-01-02 16:18:02,349] INFO Starting log cleanup with a period of 300000 ms. (kafka.log.LogManager)
[2020-01-02 16:18:02,352] INFO Starting log flusher with a default period of 9223372036854775807 ms. (kafka.log.LogManager)
[2020-01-02 16:18:02,660] INFO Awaiting socket connections on 0.0.0.0:9092. (kafka.network.Acceptor)
[2020-01-02 16:18:02,689] INFO [SocketServer brokerId=0] Created data-plane acceptor and processors for endpoint : EndPoint(null,9092,ListenerName(PLAINTEXT),PLAINTEXT) (kafka.network.SocketServer)
[2020-01-02 16:18:02,691] INFO [SocketServer brokerId=0] Started 1 acceptor threads for data-plane (kafka.network.SocketServer)
[2020-01-02 16:18:02,704] INFO [ExpirationReaper-0-Produce]: Starting (kafka.server.DelayedOperationPurgatory$ExpiredOperationReaper)
[2020-01-02 16:18:02,705] INFO [ExpirationReaper-0-Fetch]: Starting (kafka.server.DelayedOperationPurgatory$ExpiredOperationReaper)
[2020-01-02 16:18:02,706] INFO [ExpirationReaper-0-DeleteRecords]: Starting (kafka.server.DelayedOperationPurgatory$ExpiredOperationReaper)
[2020-01-02 16:18:02,707] INFO [ExpirationReaper-0-ElectLeader]: Starting (kafka.server.DelayedOperationPurgatory$ExpiredOperationReaper)
[2020-01-02 16:18:02,717] INFO [LogDirFailureHandler]: Starting (kafka.server.ReplicaManager$LogDirFailureHandler)
[2020-01-02 16:18:02,742] INFO Creating /brokers/ids/0 (is it secure? false) (kafka.zk.KafkaZkClient)
[2020-01-02 16:18:02,761] INFO Stat of the created znode at /brokers/ids/0 is: 24,24,1577953082755,1577953082755,1,0,0,72172240570875904,200,0,24
 (kafka.zk.KafkaZkClient)
[2020-01-02 16:18:02,762] INFO Registered broker 0 at path /brokers/ids/0 with addresses: ArrayBuffer(EndPoint(DESKTOP-6GAGLK9,9092,ListenerName(PLAINTEXT),PLAINTEXT)), czxid (broker epoch): 24 (kafka.zk.KafkaZkClient)
[2020-01-02 16:18:02,816] INFO [ExpirationReaper-0-topic]: Starting (kafka.server.DelayedOperationPurgatory$ExpiredOperationReaper)
[2020-01-02 16:18:02,819] INFO [ExpirationReaper-0-Heartbeat]: Starting (kafka.server.DelayedOperationPurgatory$ExpiredOperationReaper)
[2020-01-02 16:18:02,820] INFO [ExpirationReaper-0-Rebalance]: Starting (kafka.server.DelayedOperationPurgatory$ExpiredOperationReaper)
[2020-01-02 16:18:02,833] INFO Successfully created /controller_epoch with initial epoch 0 (kafka.zk.KafkaZkClient)
[2020-01-02 16:18:02,835] INFO [GroupCoordinator 0]: Starting up. (kafka.coordinator.group.GroupCoordinator)
[2020-01-02 16:18:02,837] INFO [GroupCoordinator 0]: Startup complete. (kafka.coordinator.group.GroupCoordinator)
[2020-01-02 16:18:02,839] INFO [GroupMetadataManager brokerId=0] Removed 0 expired offsets in 2 milliseconds. (kafka.coordinator.group.GroupMetadataManager)
[2020-01-02 16:18:02,850] INFO [ProducerId Manager 0]: Acquired new producerId block (brokerId:0,blockStartProducerId:0,blockEndProducerId:999) by writing to Zk with path version 1 (kafka.coordinator.transaction.ProducerIdManager)
[2020-01-02 16:18:02,869] INFO [TransactionCoordinator id=0] Starting up. (kafka.coordinator.transaction.TransactionCoordinator)
[2020-01-02 16:18:02,871] INFO [TransactionCoordinator id=0] Startup complete. (kafka.coordinator.transaction.TransactionCoordinator)
[2020-01-02 16:18:02,871] INFO [Transaction Marker Channel Manager 0]: Starting (kafka.coordinator.transaction.TransactionMarkerChannelManager)
[2020-01-02 16:18:02,888] INFO [ExpirationReaper-0-AlterAcls]: Starting (kafka.server.DelayedOperationPurgatory$ExpiredOperationReaper)
[2020-01-02 16:18:02,905] INFO [/config/changes-event-process-thread]: Starting (kafka.common.ZkNodeChangeNotificationListener$ChangeEventProcessThread)
[2020-01-02 16:18:02,920] INFO [SocketServer brokerId=0] Started data-plane processors for 1 acceptors (kafka.network.SocketServer)
[2020-01-02 16:18:02,922] INFO Kafka version: 2.4.0 (org.apache.kafka.common.utils.AppInfoParser)
[2020-01-02 16:18:02,923] INFO Kafka commitId: 77a89fcf8d7fa018 (org.apache.kafka.common.utils.AppInfoParser)
[2020-01-02 16:18:02,923] INFO Kafka startTimeMs: 1577953082920 (org.apache.kafka.common.utils.AppInfoParser)
[2020-01-02 16:18:02,925] INFO [KafkaServer id=0] started (kafka.server.KafkaServer)
```

## 四、kafka测试

### 4.1 创建一个topic
命令和执行结果如下
```cmd
kafka-topics.bat --create --zookeeper localhost:2181 --replication-factor 1 --partitions 1 --topic kafka_test

D:\kafka\kafka_2.11-2.4.0\bin\windows>kafka-topics.bat --create --zookeeper localhost:2181 --replication-factor 1 --partitions 1 --topic kafka_test
WARNING: Due to limitations in metric names, topics with a period ('.') or underscore ('_') could collide. To avoid issues it is best to use either, but not both.
Created topic kafka_test.
```

查看topic
```cmd
kafka-topics.bat --list --zookeeper localhost:2181

D:\kafka\kafka_2.11-2.4.0\bin\windows>kafka-topics.bat --list --zookeeper localhost:2181
kafka_test

```

### 4.2 启动生产者

```cmd
kafka-console-producer.bat --broker-list localhost:9092 --topic kafka_test
```

### 4.3 启动消费者
```cmd
kafka-console-consumer.bat --bootstrap-server localhost:9092 --topic kafka_test --from-beginning

# 在生产者cmd中输入文字并且回车，就能再消费者cmd中看到对应的消息
```

