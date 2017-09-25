## Maven插件解析机制

### 插件仓库

与依赖构件一样，插件构建同样基于坐标存储在Maven仓库中。需要的时候，Maven会从本地仓库中寻找插件，如果不存在，则从远程仓库查找。
插件的远程仓库使用 `pluginRepositories` 和 `pluginRepository` 配置。

### 插件默认 groupId

在POM中配置插件的时候，如果该插件是Maven的官方插件（即如果其 `groupId`为`org.apache.maven.plugins`)，就可以省略`groupId`配置。

> 不推荐上述做法

### 解析插件版本

Maven在超级POM中为所有核心插件设定了版本，超级POM是所有Maven项目的父POM，所有项目都继承这个超级POM的配置。这些插件包括`maven-clean-plugin`、`maven-compile-plugin`、`maven-surefire-plugin`等。
如果用户使用某个插件时没有设定版本，而这个插件又不属于核心插件的范畴，Maven就会去检查所有仓库中可用的版本，然后做成选择。Maven遍历本地仓罗和所有远程插件仓库，将该路径下的仓库元数据归并后，就能计算出`lastest`和`release`。Maven2 使用`lastest`版本，Maven3中使用`release`。

Maven3 超级POM的位置 $MAVEN_HOME/lib/maven-model-builder-x.x.x.jar 中 org/apache/maven/model/pom-4.0.0.xml