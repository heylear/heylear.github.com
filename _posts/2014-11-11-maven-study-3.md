--- 
layout: post 
title: "Maven 项目实战备忘" 
date: 2014-11-11 
categories: 构建工具
tags: 构建 maven java
--- 

#### 配置jdk1.7 Profile
```
<profile>
    <id>jdk1.7</id>
    <activation>
    	<!-- 默认使用此Profile -->
    	<activeByDefault>true</activeByDefault>
        <jdk>1.7</jdk>
    </activation>
    <properties>
        <maven.compiler.source>1.7</maven.compiler.source>
    	<maven.compiler.target>1.7</maven.compiler.target>
    	<maven.compiler.compilerVersion>1.7</maven.compiler.compilerVersion>
    </properties>
</profile>
```

### 配置私有仓库 Profile
```
<profile> 
  <id>nexus</id>  
  <repositories> 
    <repository> 
      <id>central</id>  
      <name>central</name>  
      <url>http://10.108.6.22:8082/nexus/content/groups/public/</url>  
      <releases> 
        <enabled>true</enabled> 
      </releases>  
      <snapshots> 
        <enabled>true&gt;</enabled> 
      </snapshots> 
    </repository> 
  </repositories>  
  <pluginRepositories> 
    <pluginRepository> 
      <id>central</id>  
      <name>central</name>  
      <url>http://10.108.6.22:8082/nexus/content/groups/public/</url>  
      <releases> 
        <enabled>true</enabled> 
      </releases>  
      <snapshots> 
        <enabled>true&gt;</enabled> 
      </snapshots> 
    </pluginRepository> 
  </pluginRepositories> 
</profile>
```

### 配置jetty插件 
```
<plugin> 
  <groupId>org.mortbay.jetty</groupId>  
  <artifactId>maven-jetty-plugin</artifactId>  
  <version>6.1.26</version>  
  <configuration> 
    <scanIntervalSeconds>3</scanIntervalSeconds>  
    <connectors> 
      <connector implementation="org.mortbay.jetty.nio.SelectChannelConnector"> 
        <port>9090</port> 
      </connector> 
    </connectors>  
    <contextPath>/anjiilpapi</contextPath>  
    <!--<webAppConfig>
    <defaultsDescriptor>src/main/resources/webdefault.xml</defaultsDescriptor>
    </webAppConfig>--> 
  </configuration>  
  <executions> 
    <execution> 
      <id>start-jetty</id>  
      <phase>pre-integration-test</phase>  
      <goals> 
        <goal>run</goal> 
      </goals>  
      <configuration> 
        <scanIntervalSeconds>0</scanIntervalSeconds>  
        <daemon>true</daemon> 
      </configuration> 
    </execution> 
  </executions> 
</plugin>
```

### 配置war打包插件---使用特殊的profile以实现不同环境打包不同文件 

```
<!-- Profile -->
<profile>
	<id>test</id>
	<properties>
		<package.env>test</package.env>
	</properties>
</profile>
<profile>
	<id>prd</id>
	<properties>
		<package.env>prd</package.env>
	</properties>
</profile>

<!-- Plugins -->
<plugin> 
  <groupId>org.apache.maven.plugins</groupId>  
  <artifactId>maven-war-plugin</artifactId>  
  <configuration> 
    <webResources> 
      <resource> 
        <directory>src/main/resources/${package.env}/prop</directory>  
        <targetPath>WEB-INF/classes</targetPath>  
        <filtering>true</filtering> 
      </resource>  
      <resource> 
        <directory>src/main/resources/${package.env}/conf/spring</directory>  
        <targetPath>WEB-INF/classes/conf/spring</targetPath>  
        <filtering>true</filtering> 
      </resource> 
    </webResources> 
  </configuration> 
</plugin>
```

