## Maven生命周期

Maven有三套相互独立的生命周期，分别是clean、default和site。clean生命周期的目的是清理项目，default生命周期的目的是构建项目，site生命周期的目的是建立站点项目。

每个生命周期包含一些阶段（phase），这些阶段是有顺序的，并且后面的阶段依赖前面的阶段，用户和Maven最直接的交互方式就是调用这些生命周期阶段。以clean声明周期为例，它包含的阶段有pre-clean，clean和post-clean。当用户调用pre-clean时，只有pre-clean的阶段得意执行；当用户调用clean的时候，pre-clean和clean阶段会得意顺序执行；当用户调用post-clean时，pre-clean，clean和post-clean 会得以顺序执行。

### clean 生命周期

clean 生命周期的目的是清理项目，包含三个阶段：

1. `pre-clean` 执行一些清理前需要完成的工作
2. `clean` 清理上一次构建生成的文件
3. `post-clean` 执行一些清理后需要完成的工作

### default 生命周期

defaul 生命周期定义了真正构建时所需要执行的所有步骤，他是所有生命周期中最核心的部分，其包含的阶段如下：

1. `validate`
2. `initialize`
3. `generate-sources`
4. `process-sources` 处理项目主资源文件。一般来说，是对 `src/main/resources` 目录的内容进行变量替换等工作后，复制到项目输出的主 `classpath` 目录中。
5. `generate-resources`
6. `process-resources`
7. `compile` 编译项目的主源码。一般来说，是编译 `src/main/java` 目录下的 Java 文件至项目输出的主 `classpath` 目录中。
8. `process-classes`
9. `generate-test-resources`
10. `process-test-resources` 处理项目测试资源文件。一般来说，是对 `src/test/resources` 目录的内容进行变量替换等工作后，复制到项目输出的测试 `classpath` 目录中。
11. `test-compile` 编译项目的测试代码。一般来说，是编译 `src/test/java` 目录项的Java文件至项目输出的测试 `classpath` 目录中。
12. `process-test-classes`
13. `test` 使用单元测试框架运行测试，测试代码不会被打包或者部署。
14. `prepare-package`
15. `package` 接受编译好的代码，打包成可发布的格式。
16. `pre-integration-test`
17. `integration-test`
18. `post-integration`
19. `verify`
20. `install` 将包安装到Maven本地仓库，共本地其他Maven项目使用。
21. `deploy` 将最终的包复制到远程仓库，共其他开发人员和Maven项目使用。


### site 生命周期

site 声明周期的目的是建立和发布站点项目。Maven 能够基于POM所包含的信息，自动生成一个友好的站点，包含如下阶段：

1. `pre-site` 执行一些在生成项目站点之前需要完成的工作。
2. `site` 生成项目站点文档。
3. `post-site` 执行一些在生成项目之后需要完成的工作。
4. `site-deploy` 将生成的项目站点发布到服务器上。

### 命令行与生命周期

从命令行执行Maven任务的最主要的方式就是调用Maven的生命周期阶段。各个生命周期是相互独立的，而一个生命周期的阶段时有前后依赖关系的。

1. $mvn clean: 该命令调用`clean`生命周期的`clean`阶段。实际执行的阶段为`clean`生命周期的`pre-clean`和`clean`阶段。
2. $mvn test: 该命令调用`default`生命周期的`test`阶段，实际执行的阶段为`default`生命周期的`validate`、`initialize`等，直到`test`的所有阶段。
3. $mvn clean install: 该命令调用`clean`声明周期的`clean`和`default`生命周期的`install`阶段。实际执行的阶段为`clean`声明周期的`pre-clean`、`clean`阶段，以及`default`声明周期从`validate`到`install`的所有阶段。该命令结合了两个生命周期，在执行真正的项目构建之前清理项目是一个很好的实践。
4. $mvn clean deploy site-deploy: 该命令调用`clean`生命周期的`clean`阶段、`default`生命周期的`deploy`阶段，以及`site`生命周期的`site-deploy`阶段。实际执行的阶段为`clean`声明周期的`pre-clean`、`clean`阶段，以及`default`的所有阶段，以及`site`生命周期中的所有阶段。




