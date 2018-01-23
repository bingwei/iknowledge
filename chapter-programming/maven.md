# Maven

## 中心仓库配置

配置文件位置：

1. $MAVEN_HOME/conf/settings.xml
1. $HOME/.m2/settings.xml

```xml
<mirror>
  <id>alimaven</id>
  <mirrorOf>central</mirrorOf>
  <name>aliyun maven</name>
  <url>http://maven.aliyun.com/nexus/content/repositories/central/</url>
</mirror>
```

## 测试

指定测试某一类或某一方法

```Shell
mvn test -Dtest=PagedFileTest
```

```Shell
mvn test -Dtest=PagedFileTest#disposePage
```
