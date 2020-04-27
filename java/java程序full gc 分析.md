现象

![img](https://blog-image-1253555052.cos.ap-guangzhou.myqcloud.com/20200328104005.png)

![img](https://blog-image-1253555052.cos.ap-guangzhou.myqcloud.com/20200328103426.png)

![](https://blog-image-1253555052.cos.ap-guangzhou.myqcloud.com/20200330142755.png)



```shell
sh-4.1# jstat -gc 1
 S0C    S1C    S0U    S1U      EC       EU        OC         OU       MC     MU    CCSC   CCSU   YGC     YGCT    FGC    FGCT     GCT
512.0  512.0   0.0    0.0   1396736.0   0.0    2796544.0   319380.6  131072.0 126449.5 14720.0 13909.6   3455   52.561  853   556.718  609.279
sh-4.1#
```

![](https://blog-image-1253555052.cos.ap-guangzhou.myqcloud.com/20200330143044.png)



dubbo 大量close_wait

![](https://blog-image-1253555052.cos.ap-guangzhou.myqcloud.com/20200330152241.png)

**先看一下**`jinfo`

``` shell
Attaching to process ID 1, please wait...
Debugger attached successfully.
Server compiler detected.
JVM version is 25.121-b13
Java System Properties:

java.runtime.name = Java(TM) SE Runtime Environment
java.vm.version = 25.121-b13
sun.boot.library.path = /usr/local/jdk1.8/jre/lib/amd64
module = web
java.vendor.url = http://java.oracle.com/
java.vm.vendor = Oracle Corporation
path.separator = :
file.encoding.pkg = sun.io
java.vm.name = Java HotSpot(TM) 64-Bit Server VM
dev_meta = http://apollo.dev.ecqun.com:8080
sun.os.patch.level = unknown
env = pro
Log4jContextSelector = org.apache.logging.log4j.core.async.AsyncLoggerContextSelector
hostName = user-center-web-k8s.pro.g1.a-5f467b6866-bs5j4
sun.java.launcher = SUN_STANDARD
user.country = US
user.dir = /ec/apps/user-center
java.vm.specification.name = Java Virtual Machine Specification
java.runtime.version = 1.8.0_121-b13
java.awt.graphicsenv = sun.awt.X11GraphicsEnvironment
consul = 172.16.0.63
os.arch = amd64
java.endorsed.dirs = /usr/local/jdk1.8/jre/lib/endorsed
fat_meta = http://apollo.test.ecqun.com:8080
thrift.port = 7012
line.separator =

java.io.tmpdir = /tmp
java.vm.specification.vendor = Oracle Corporation
os.name = Linux
sun.jnu.encoding = UTF-8
skywalking.agent.application_code = user-center-web
java.library.path = /usr/java/packages/lib/amd64:/usr/lib64:/lib64:/lib:/usr/lib
sun.nio.ch.bugLevel =
apollo.autoUpdateInjectedSpringProperties = false
dubbo.protocol.port = 20760
java.specification.name = Java Platform API Specification
java.class.version = 52.0
sun.management.compiler = HotSpot 64-Bit Tiered Compilers
pro_meta = http://apollo.ecqun.com:8080
os.version = 3.10.0-693.2.2.el7.x86_64
dubbo.registry.file = /root/.dubbo/dubbo-registry-user-center-web.cache
user.home = /root
user.timezone = Asia/Shanghai
java.awt.printerjob = sun.print.PSPrinterJob
file.encoding = utf-8
java.specification.version = 1.8
user.name = root
java.class.path = :./lib/user-center-0.0.1-SNAPSHOT.jar:./lib/activation-1.1.jar:./lib/aide-core-1.0-20180523.084538-1.jar:./lib/aide-integration-1.0-20180523.084550-1.jar:./lib/aide-proxy-thrift-1.0-20180523.084546-1.jar:./lib/aide-registry-consul-1.0-20180523.084542-1.jar:./lib/aliyun-sdk-oss-2.0.7.jar:./lib/animal-sniffer-annotations-1.14.jar:./lib/annotations-2.0.3.jar:./lib/aopalliance-1.0.jar:./lib/aopalliance-repackaged-2.4.0-b34.jar:./lib/apollo-client-0.11.0-20180730.062229-2.jar:./lib/apollo-core-0.11.0-20180730.062219-4.jar:./lib/asm-3.3.jar:./lib/asm-5.0.3.jar:./lib/aspectjrt-1.8.7.jar:./lib/aspectjweaver-1.8.13.jar:./lib/avro-1.8.1.jar:./lib/bcprov-jdk15-1.45.jar:./lib/bcprov-jdk15on-1.52.jar:./lib/bval-core-0.5.jar:./lib/bval-jsr303-0.5.jar:./lib/byte-buddy-1.6.8.jar:./lib/cache-api-1.0.0.jar:./lib/cglib-nodep-2.2.2.jar:./lib/classmate-1.4.0.jar:./lib/commons-beanutils-1.8.0.jar:./lib/commons-beanutils-core-1.8.3.jar:./lib/commons-codec-1.9.jar:./lib/commons-collections-3.2.1.jar:./lib/commons-compress-1.8.1.jar:./lib/commons-configuration-1.10.jar:./lib/commons-dbcp-1.4.jar:./lib/commons-digester-1.6.jar:./lib/commons-email-1.2.jar:./lib/commons-httpclient-3.1.jar:./lib/commons-lang-2.3.jar:./lib/commons-lang3-3.1.jar:./lib/commons-logging-1.1.3.jar:./lib/commons-logging-api-1.1.jar:./lib/commons-pool-1.6.jar:./lib/commons-pool2-2.0.jar:./lib/commons-validator-1.3.1.jar:./lib/configuration-0.119.jar:./lib/consul-api-1.4.0.jar:./lib/consul-client-1.0.0.jar:./lib/converter-jackson-2.0.2.jar:./lib/corp-common-0.0.1-20200304.034408-314.jar:./lib/corp-dubbo-api-0.0.1-20181218.073428-1.jar:./lib/cos_api-5.4.4.jar:./lib/curator-client-2.8.0.jar:./lib/curator-framework-2.8.0.jar:./lib/curator-recipes-2.8.0.jar:./lib/cxf-api-2.3.1.jar:./lib/cxf-common-schemas-2.3.1.jar:./lib/cxf-common-utilities-2.3.1.jar:./lib/cxf-rt-bindings-soap-2.3.1.jar:./lib/cxf-rt-bindings-xml-2.3.1.jar:./lib/cxf-rt-core-2.3.1.jar:./lib/cxf-rt-databinding-jaxb-2.3.1.jar:./lib/cxf-rt-frontend-jaxws-2.3.1.jar:./lib/cxf-rt-frontend-simple-2.3.1.jar:./lib/cxf-rt-transports-http-2.3.1.jar:./lib/cxf-rt-ws-addr-2.3.1.jar:./lib/cxf-rt-ws-security-2.3.1.jar:./lib/cxf-tools-common-2.3.1.jar:./lib/dcenter-discuss-dubbo-0.0.1-20180620.055306-204.jar:./lib/disruptor-3.3.7.jar:./lib/dubbo-2.5.3.jar:./lib/ec-commons-0.0.3-SNAPSHOT.jar:./lib/ec-watcher-0.0.1-20151125.034127-9.jar:./lib/error_prone_annotations-2.0.18.jar:./lib/ezmorph-1.0.6.jar:./lib/fastjson-1.2.60.jar:./lib/framework-core-0.0.3-SNAPSHOT.jar:./lib/freemarker-2.3.16.jar:./lib/fst-2.47.jar:./lib/geronimo-javamail_1.4_spec-1.7.1.jar:./lib/grpc-context-1.17.1.jar:./lib/grpc-core-1.17.1.jar:./lib/grpc-netty-1.17.1.jar:./lib/grpc-protobuf-1.17.1.jar:./lib/grpc-protobuf-lite-1.17.1.jar:./lib/grpc-stub-1.17.1.jar:./lib/gson-2.2.2.jar:./lib/guava-22.0.jar:./lib/guice-4.1.0.jar:./lib/guice-multibindings-4.0.jar:./lib/h2-1.2.135.jar:./lib/helper-1.0.1.jar:./lib/hibernate-validator-4.0.2.GA.jar:./lib/hk2-api-2.4.0-b34.jar:./lib/hk2-locator-2.4.0-b34.jar:./lib/hk2-utils-2.4.0-b34.jar:./lib/httpclient-4.4.1.jar:./lib/httpcore-4.4.1.jar:./lib/j2objc-annotations-1.1.jar:./lib/jackson-annotations-2.9.0.jar:./lib/jackson-core-2.9.9.jar:./lib/jackson-core-asl-1.9.13.jar:./lib/jackson-databind-2.9.9.1.jar:./lib/jackson-dataformat-avro-2.9.9.jar:./lib/jackson-dataformat-cbor-2.9.9.jar:./lib/jackson-dataformat-msgpack-0.8.12.jar:./lib/jackson-dataformat-smile-2.8.7.jar:./lib/jackson-dataformat-yaml-2.9.9.jar:./lib/jackson-datatype-guava-2.7.4.jar:./lib/jackson-datatype-jdk8-2.7.4.jar:./lib/jackson-mapper-asl-1.9.13.jar:./lib/javassist-3.15.0-GA.jar:./lib/javax.annotation-api-1.3.2.jar:./lib/javax.inject-1.jar:./lib/javax.inject-2.4.0-b34.jar:./lib/javax.servlet-api-3.1.0.jar:./lib/jaxb-api-2.1.jar:./lib/jaxb-impl-2.1.3.jar:./lib/jchardet-1.0.jar:./lib/jdom-1.1.jar:./lib/jedis-2.6.2.jar:./lib/jersey-client-2.22.2.jar:./lib/jersey-common-2.22.2.jar:./lib/jersey-guava-2.22.2.jar:./lib/jetty-http-9.3.0.M2.jar:./lib/jetty-io-9.3.0.M2.jar:./lib/jetty-server-9.3.0.M2.jar:./lib/jetty-util-9.3.0.M2.jar:./lib/jline-0.9.94.jar:./lib/jmxutils-1.18.jar:./lib/joda-time-2.9.6.jar:./lib/jodd-bean-3.7.1.jar:./lib/jodd-core-3.7.1.jar:./lib/jodis-0.1.2.jar:./lib/jol-core-0.1.jar:./lib/jopt-simple-3.2.jar:./lib/json-lib-2.4-jdk15.jar:./lib/jsoup-1.9.2.jar:./lib/jsr305-1.3.9.jar:./lib/jul-to-slf4j-1.7.25.jar:./lib/kafka_2.11-0.8.2.1.jar:./lib/kafka-clients-0.8.2.1.jar:./lib/kryo-3.0.3.jar:./lib/libphonenumber-8.6.0.jar:./lib/libthrift-0.9.2.jar:./lib/log-0.119.jar:./lib/log4j-1.2-api-2.11.0.jar:./lib/log4j-api-2.11.0.jar:./lib/log4j-core-2.11.0.jar:./lib/log4jdbc-1.2.jar:./lib/log4j-slf4j-impl-2.11.0.jar:./lib/log4j-web-2.11.0.jar:./lib/lombok-1.16.22.jar:./lib/lz4-1.3.0.jar:./lib/mail-1.4.7.jar:./lib/mapstruct-1.2.0.Final.jar:./lib/maven-plugin-api-2.0.jar:./lib/metrics-core-2.2.0.jar:./lib/mina-core-2.0.9.jar:./lib/mina-integration-beans-2.0.9.jar:./lib/minlog-1.3.0.jar:./lib/mongo-java-driver-3.0.0.jar:./lib/msgpack-core-0.8.12.jar:./lib/mybatis-3.3.0.jar:./lib/mybatis-spring-1.2.3.jar:./lib/mysql-connector-java-5.1.35.jar:./lib/neethi-2.0.4.jar:./lib/netty-3.6.10.Final.jar:./lib/netty-buffer-4.1.30.Final.jar:./lib/netty-codec-4.1.30.Final.jar:./lib/netty-codec-http2-4.1.30.Final.jar:./lib/netty-codec-http-4.1.30.Final.jar:./lib/netty-codec-socks-4.1.30.Final.jar:./lib/netty-common-4.1.30.Final.jar:./lib/netty-handler-4.1.30.Final.jar:./lib/netty-handler-proxy-4.1.30.Final.jar:./lib/netty-resolver-4.1.30.Final.jar:./lib/netty-tcnative-boringssl-static-1.1.33.Fork23.jar:./lib/netty-transport-4.1.30.Final.jar:./lib/netty-transport-native-epoll-4.1.8.Final-linux-x86_64.jar:./lib/nifty-client-0.23.0.jar:./lib/nifty-core-0.23.0.jar:./lib/nifty-ssl-0.23.0.jar:./lib/objenesis-2.4.jar:./lib/okhttp-3.8.0.jar:./lib/okio-1.13.0.jar:./lib/opencensus-api-0.17.0.jar:./lib/opencensus-contrib-grpc-metrics-0.17.0.jar:./lib/osgi-resource-locator-1.0.1.jar:./lib/paranamer-2.7.jar:./lib/poi-3.12.jar:./lib/poi-ooxml-3.12.jar:./lib/poi-ooxml-schemas-3.12.jar:./lib/protobuf-java-3.5.1.jar:./lib/proto-google-common-protos-1.0.0.jar:./lib/rapid-core-4.0.5.jar:./lib/rapid-generator-4.0.6.jar:./lib/rapid-maven-plugin-1.0.jar:./lib/rapid-plugins-4.0.6.jar:./lib/reactive-streams-1.0.0.jar:./lib/reactor-core-2.0.8.RELEASE.jar:./lib/reactor-stream-2.0.8.RELEASE.jar:./lib/redisson-2.8.2.jar:./lib/reflectasm-1.10.1.jar:./lib/retrofit-2.3.0.jar:./lib/scala-library-2.11.5.jar:./lib/scala-parser-combinators_2.11-1.0.2.jar:./lib/scala-xml_2.11-1.0.2.jar:./lib/serializer-2.7.1.jar:./lib/slf4j-api-1.6.0.jar:./lib/slice-0.10.jar:./lib/snakeyaml-1.23.jar:./lib/snappy-java-1.1.2.6.jar:./lib/spring-aop-4.1.6.RELEASE.jar:./lib/spring-aspects-4.1.6.RELEASE.jar:./lib/spring-beans-4.1.6.RELEASE.jar:./lib/spring-boot-2.0.4.RELEASE.jar:./lib/spring-boot-autoconfigure-2.0.4.RELEASE.jar:./lib/spring-boot-starter-2.0.4.RELEASE.jar:./lib/spring-boot-starter-logging-2.0.4.RELEASE.jar:./lib/springbt-common-coss-0.0.1-20181023.005855-24.jar:./lib/spring-context-4.1.6.RELEASE.jar:./lib/spring-context-support-4.1.6.RELEASE.jar:./lib/spring-core-4.1.6.RELEASE.jar:./lib/spring-data-redis-1.6.1.RELEASE.jar:./lib/spring-expression-4.1.6.RELEASE.jar:./lib/springfox-core-2.9.2.jar:./lib/springfox-schema-2.9.2.jar:./lib/springfox-spi-2.9.2.jar:./lib/springfox-spring-web-2.9.2.jar:./lib/springfox-swagger2-2.9.2.jar:./lib/springfox-swagger-common-2.9.2.jar:./lib/springfox-swagger-ui-2.9.2.jar:./lib/spring-jdbc-4.1.6.RELEASE.jar:./lib/spring-jms-3.2.3.RELEASE.jar:./lib/spring-orm-4.1.6.RELEASE.jar:./lib/spring-plugin-core-1.2.0.RELEASE.jar:./lib/spring-plugin-metadata-1.2.0.RELEASE.jar:./lib/spring-test-4.1.6.RELEASE.jar:./lib/spring-tx-4.1.6.RELEASE.jar:./lib/spring-web-4.1.6.RELEASE.jar:./lib/spring-webmvc-4.1.6.RELEASE.jar:./lib/stats-0.119.jar:./lib/stax2-api-3.0.2.jar:./lib/stax-api-1.0.1.jar:./lib/stax-api-1.0-2.jar:./lib/super-csv-2.3.1.jar:./lib/swagger-annotations-1.5.20.jar:./lib/swagger-bootstrap-ui-1.8.3.jar:./lib/swagger-models-1.5.20.jar:./lib/swift-annotations-0.23.1.jar:./lib/swift-codec-0.23.1.jar:./lib/swift-service-0.23.1.jar:./lib/trendrr-nsq-client-1.4.2.jar:./lib/trendrr-oss-1.0.jar:./lib/units-0.119.jar:./lib/validation-api-1.0.0.GA.jar:./lib/velocity-1.7.jar:./lib/woodstox-core-asl-4.0.8.jar:./lib/wsdl4j-1.6.2.jar:./lib/wss4j-1.5.10.jar:./lib/xalan-2.7.1.jar:./lib/xdiamond-client-1.0.5.jar:./lib/xdiamond-common-1.0.1.jar:./lib/xml-apis-1.0.b2.jar:./lib/xmlbeans-2.6.0.jar:./lib/xml-resolver-1.2.jar:./lib/XmlSchema-1.4.7.jar:./lib/xmlsec-1.4.4.jar:./lib/xsqlbuilder-1.0.jar:./lib/xz-1.5.jar:./lib/zero-allocation-hashing-0.7.jar:./lib/zkclient-0.1.jar:./lib/zkclient-0.5.jar:./lib/zookeeper-3.4.5.jar:/ec/apps/ec-apm-agent/ec-apm-agent-1.3.0/apm-agent.jar:/ec/apps/tingyun/tingyun/tingyun-agent-java.jar
java.vm.specification.version = 1.8
sun.arch.data.model = 64
sun.java.command = com.ec.usercenter.appmain.Launcher
java.home = /usr/local/jdk1.8/jre
user.language = en
java.specification.vendor = Oracle Corporation
app.id = user-center
awt.toolkit = sun.awt.X11.XToolkit
ec.file.encoding = utf-8
java.vm.info = mixed mode
consul.discovery.tags = urlprefix-:7012,proto=tcp
java.version = 1.8.0_121
java.ext.dirs = /usr/local/jdk1.8/jre/lib/ext:/usr/java/packages/lib/ext
sun.boot.class.path = /usr/local/jdk1.8/jre/lib/resources.jar:/usr/local/jdk1.8/jre/lib/rt.jar:/usr/local/jdk1.8/jre/lib/sunrsasign.jar:/usr/local/jdk1.8/jre/lib/jsse.jar:/usr/local/jdk1.8/jre/lib/jce.jar:/usr/local/jdk1.8/jre/lib/charsets.jar:/usr/local/jdk1.8/jre/lib/jfr.jar:/usr/local/jdk1.8/jre/classes
java.vendor = Oracle Corporation
file.separator = /
java.vendor.url.bug = http://bugreport.sun.com/bugreport/
sun.io.unicode.encoding = UnicodeLittle
sun.cpu.endian = little
processName = user-center-web
sun.cpu.isalist =

VM Flags:
Non-default VM flags: -XX:CICompilerCount=12 -XX:InitialHeapSize=4294967296 -XX:MaxHeapSize=4294967296 -XX:MaxMetaspaceSize=134217728 -XX:MaxNewSize=1431306240 -XX:MetaspaceSize=134217728 -XX:MinHeapDeltaBytes=524288 -XX:NewSize=1431306240 -XX:OldSize=2863661056 -XX:+PrintGC -XX:+PrintGCTimeStamps -XX:+UseCompressedClassPointers -XX:+UseCompressedOops -XX:+UseParallelGC
Command line:  -Dthrift.port=7012 -Ddubbo.protocol.port=20760 -Dconsul.discovery.tags=urlprefix-:7012,proto=tcp -Dfile.encoding=utf-8 -Denv=pro -Dmodule=web -Dconsul=172.16.0.63 -Xms4096m -Xmx4096m -XX:MetaspaceSize=128m -XX:MaxMetaspaceSize=128m -Dec.file.encoding=utf-8 -DLog4jContextSelector=org.apache.logging.log4j.core.async.AsyncLoggerContextSelector -Dfile.encoding=utf-8-DhostName=user-center-web-k8s.pro.g1.a-5f467b6866-bs5j4 -DprocessName=user-center-web -Dapp.id=user-center -Ddev_meta=http://apollo.dev.ecqun.com:8080 -Dfat_meta=http://apollo.test.ecqun.com:8080 -Dpro_meta=http://apollo.ecqun.com:8080 -Dapollo.autoUpdateInjectedSpringProperties=false -javaagent:/ec/apps/ec-apm-agent/ec-apm-agent-1.3.0/apm-agent.jar -Dskywalking.agent.application_code=user-center-web -javaagent:/ec/apps/tingyun/tingyun/tingyun-agent-java.jar -verbose:gc -XX:+PrintGCTimeStamps -Xloggc:/data/logs/user-center-web-log/user-center.gc.log -Ddubbo.registry.file=/root/.dubbo/dubbo-registry-user-center-web.cache
```





```shell
 Meta@user-center-web-k8s user-center-web-log]# java -XX:+PrintFlagsFinal | grep
    uintx InitialBootClassLoaderMetaspaceSize       = 4194304                             {product}
    uintx MaxMetaspaceExpansion                     = 5451776                             {product}
    uintx MaxMetaspaceFreeRatio                     = 70                                  {product}
    uintx MaxMetaspaceSize                          = 18446744073709547520                    {product}
    uintx MetaspaceSize                             = 21807104                            {pd product}
    uintx MinMetaspaceExpansion                     = 339968                              {product}
    uintx MinMetaspaceFreeRatio                     = 40                                  {product}
     bool TraceMetadataHumongousAllocation          = false                               {product}
     bool UseLargePagesInMetaspace                  = false                               {product}
```

## GC (Allocation Failure)

看log里头99%都是GC (Allocation Failure)造成的young gc。Allocation Failure表示向young generation(eden)给新对象申请空间，但是young generation(eden)剩余的合适空间不够所需的大小导致的minor gc。