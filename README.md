# esshade
Hbase与Elasticsearch的jar包冲突解决办法

当项目中同时集成Hbase和Elasticsearch时，经常遇到依赖包冲突的问题，如com.google.guava，org.joda等。造成guava冲突是因为集成Hbase和ES时都用到了guava包，但是两者要求的版本不一样，ES 2.0 版本以上的要求guava（19.0+），Hbase 1.0要求的guava为16.0，如果把guava统一为16.0，则ES会因为guava的版本太低而报错。joda也如此。所以，要同时使用Hbase和ES，就得解决这个冲突。

解决办法

1、首先，新建maven项目，对pom.xml进行配置，pom.xml文件配置好后，构建，得到jar包。依赖关系加入.m2文件夹里。

如上配置完成后，其实就是将com.google.guava等4个可能有冲突的jar包通过maven-shade-plugin插件迁移后，重新打个jar包从而使得在引入这个jar包时能够使用该jar包自己的依赖而不是使用外部依赖。

2、Hbase与ES的融合
在同时集成Hbase与ES的项目中，加入Hbase的依赖包，并加入步骤1中构建的依赖包；
配置好了，重新清理构建，测试一下，Hbase和ES就可以同时使用了。
