# esshade
Hbase与Elasticsearch的jar包冲突解决办法
问题

当项目中同时集成Hbase和Elasticsearch时，经常遇到依赖包冲突的问题，如com.google.guava，org.joda等。造成guava冲突是因为集成Hbase和ES时都用到了guava包，但是两者要求的版本不一样，ES 2.0 版本以上的要求guava（19.0+），Hbase 1.0要求的guava为16.0，如果把guava统一为16.0，则ES会因为guava的版本太低而报错。joda也如此。所以，要同时使用Hbase和ES，就得解决这个冲突。

解决办法

1、首先，新建maven项目，对pom.xml进行如下配置：

<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>
    <groupId>com.es</groupId>
    <artifactId>esshade</artifactId>
    <version>1.0-SNAPSHOT</version>
    <packaging>jar</packaging>
    <properties>
        <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
        <maven.compiler.source>1.8</maven.compiler.source>
        <maven.compiler.target>1.8</maven.compiler.target>
    </properties>
    <dependencies>   
        <!-- elasticsearch -->
        <dependency>
            <groupId>org.elasticsearch</groupId>
            <artifactId>elasticsearch</artifactId>
            <version>2.4.1</version>
        </dependency>
        <dependency>
            <groupId>org.elasticsearch.plugin</groupId>
            <artifactId>delete-by-query</artifactId>
            <version>2.4.1</version>
        </dependency>
        <!-- elasticsearch 依赖库结束 -->
    </dependencies>
    <build>
        <plugins>
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-shade-plugin</artifactId>
                <version>2.4.1</version>
                <configuration>
                    <createDependencyReducedPom>false</createDependencyReducedPom>
                </configuration>
                <executions>
                    <execution>
                        <phase>package</phase>
                        <goals>
                            <goal>shade</goal>
                        </goals>
                        <configuration>
                            <relocations>
                                <relocation>
                                    <pattern>com.google.guava</pattern>
                                    <shadedPattern>my.elasticsearch.guava</shadedPattern>
                                </relocation>
                                <relocation>
                                    <pattern>org.joda</pattern>
                                    <shadedPattern>my.elasticsearch.joda</shadedPattern>
                                </relocation>
                                <relocation>
                                    <pattern>com.google.common</pattern>
                                    <shadedPattern>my.elasticsearch.common</shadedPattern>
                                </relocation>
                                <relocation>
                                    <pattern>com.google.thirdparty</pattern>
                                    <shadedPattern>my.elasticsearch.thirdparty</shadedPattern>
                                </relocation>
                            </relocations>
                            <transformers>
                                <transformer implementation="org.apache.maven.plugins.shade.resource.ManifestResourceTransformer" />
                            </transformers>
                        </configuration>
                    </execution>
                </executions>
            </plugin>
        </plugins>
    </build>
    <repositories>
        <repository>
            <id>elasticsearch-releases</id>
            <url>http://maven.elasticsearch.org/releases</url>
            <releases>
                <enabled>true</enabled>
                <updatePolicy>daily</updatePolicy>
            </releases>
            <snapshots>
                <enabled>false</enabled>
            </snapshots>
        </repository>
    </repositories>
</project>

pom.xml文件配置好后，构建，得到jar包。如下图，依赖关系加入.m2文件夹里。

如上配置完成后，其实就是将com.google.guava等4个可能有冲突的jar包通过maven-shade-plugin插件迁移后，重新打个jar包从而使得在引入这个jar包时能够使用该jar包自己的依赖而不是使用外部依赖。

2、Hbase与ES的融合
在同时集成Hbase与ES的项目中，加入Hbase的依赖包，并加入步骤1中构建的依赖包：

<dependency>
            <groupId>org.apache.hadoop</groupId>
            <artifactId>hadoop-hdfs</artifactId>
            <version>2.6.4</version>
        </dependency>
        <dependency>
            <groupId>org.apache.hadoop</groupId>
            <artifactId>hadoop-client</artifactId>
            <version>2.6.4</version>
        </dependency>
        <dependency>
            <groupId>org.apache.hbase</groupId>
            <artifactId>hbase-client</artifactId>
            <version>1.2.1</version>
        </dependency>
        <dependency>
            <groupId>org.apache.hbase</groupId>
            <artifactId>hbase</artifactId>
            <version>1.2.1</version>
            <type>pom</type>
        </dependency>
        <dependency>
            <groupId>com.google.guava</groupId>
            <artifactId>guava</artifactId>
            <version>16.0.1</version>
        </dependency>
        <dependency>
            <groupId>com.es</groupId>
            <artifactId>esshade</artifactId>
            <version>1.0-SNAPSHOT</version>
            <exclusions>
                <exclusion>
                    <groupId>org.elasticsearch</groupId>
                    <artifactId>elasticsearch</artifactId>
                </exclusion>
            </exclusions>
        </dependency>
配置好了，重新清理构建，测试一下，Hbase和ES就可以同时使用了。
