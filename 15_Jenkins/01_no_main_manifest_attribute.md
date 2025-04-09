### 项目背景：

基于`java 19`的项目，在项目`pom.xml`文件中没有如下配置：

```xml
    <build>
        <plugins>
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
                <version>3.4.1</version>
                <configuration>
                    <mainClass>com.chow.AssistantApplication</mainClass>
                </configuration>
                <executions>
                    <execution>
                        <goals>
                            <goal>repackage</goal>
                        </goals>
                    </execution>
                </executions>
            </plugin>
        </plugins>
    </build>
```

使用`jenkins`进行构建时，没有报错，但是启动`docker`镜像时报错：

```bash
no main manifest attribute, in /app.jar
```

后经排查发现，使用`maven`工具生成的`jar`包中，缺少了 `Main-Class` 和 `Start-Class`。

原始`MANIFEST.MF`文件内容如下：

```xml
Manifest-Version: 1.0
Created-By: Maven JAR Plugin 3.4.1
Build-Jdk-Spec: 19
```

这表明 **Spring Boot Maven 插件未生效**。



### 可能原因分析：

#### 原因 1：**插件版本不兼容或未显式指定版本**

Spring Boot 2.x 和 3.x 的插件配置差异较大，如果未显式指定插件版本，可能导致默认版本与 Java 19 不兼容，从而跳过 `repackage` 目标。

------

#### 原因 2：**Maven 打包流程被其他插件干扰**

如果项目中同时配置了 `maven-jar-plugin` 或其他打包插件，可能会覆盖 `spring-boot-maven-plugin` 生成的 MANIFEST.MF。

------

#### 原因 3：**未执行 `repackage` 目标**

`spring-boot-maven-plugin` 的核心逻辑是通过 `repackage` 目标生成可执行 JAR。如果构建流程中未触发此目标（例如手动跳过了某些阶段），则插件不会生效。



### 解决步骤：

#### 1. 显式指定插件版本

在 `pom.xml` 中显式指定与 Spring Boot 版本匹配的插件版本：

```xml
<build>
    <plugins>
        <plugin>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-maven-plugin</artifactId>
            <!-- 显式指定版本（需与Spring Boot版本一致） -->
            <version>3.4.1</version>  <!-- 假设使用Spring Boot 3.4.1 -->
            <configuration>
                <mainClass>com.chow.AssistantApplication</mainClass>
            </configuration>
            <!-- 关键：绑定到Maven生命周期 -->
            <executions>
                <execution>
                    <goals>
                        <goal>repackage</goal>
                    </goals>
                </execution>
            </executions>
        </plugin>
    </plugins>
</build>
```

- **注意**：`spring-boot-maven-plugin` 的 `repackage` 目标需要绑定到 Maven 的 `package` 生命周期阶段。如果未显式绑定，插件可能不会执行关键的重打包操作。



#### 2. 执行完整的 Maven 打包命令

确保使用以下命令构建项目：

```shell
mvn clean package
```



#### 3. 修复后，重新打包，生成`MANIFEST.MF`文件内容如下：

```xml
Manifest-Version: 1.0
Created-By: Maven JAR Plugin 3.4.1
Build-Jdk-Spec: 19
Main-Class: org.springframework.boot.loader.launch.JarLauncher
Start-Class: com.chow.AssistantApplication
Spring-Boot-Version: 3.4.1
Spring-Boot-Classes: BOOT-INF/classes/
Spring-Boot-Lib: BOOT-INF/lib/
Spring-Boot-Classpath-Index: BOOT-INF/classpath.idx
Spring-Boot-Layers-Index: BOOT-INF/layers.idx
```





### 常见问题排查表

| 现象                 | 可能原因                      | 解决方案                        |
| :------------------- | :---------------------------- | :------------------------------ |
| 无 `Main-Class`      | 插件未执行 `repackage`        | 添加 `<executions>` 绑定        |
| 找不到 `Start-Class` | 主类路径配置错误              | 检查 `<mainClass>` 配置         |
| 复制了原始 JAR       | Dockerfile 中通配符匹配错误   | 使用 `target/assistant-*.jar`   |
| 构建日志无插件输出   | 插件版本与 Spring Boot 不兼容 | 确保插件版本与 Spring Boot 一致 |