<!-- TOC -->

- [1. 背景](#1-背景)
- [2. 环境准备](#2-环境准备)
- [3. 实现步骤](#3-实现步骤)
  - [3.1. 构建Gradle项目](#31-构建gradle项目)
  - [3.2. 编写build.gradle](#32-编写buildgradle)
    - [3.2.1. plugins](#321-plugins)
    - [3.2.2. javaPlatform](#322-javaplatform)
    - [3.2.3. dependencies定义依赖](#323-dependencies定义依赖)
    - [3.2.4. publishing发布](#324-publishing发布)
  - [3.3. publishing发布中央仓库](#33-publishing发布中央仓库)
    - [3.3.1. plugins](#331-plugins)
    - [3.3.2. publishing](#332-publishing)
- [4. 应用](#4-应用)

<!-- /TOC -->

## 1. 背景

我们在做多模块开发任务过程中，Java 框架会有很多外部依赖，这些外部依赖还会有版本兼容之间的问题，为了解决每个任务模块所依赖外部 `JAR` 之间版本，将一组外部依赖 `JAR` 所兼容或者包含的版本在某个模块中统一管理。

回想下，我们在 `Maven` 工程中使用 `Spring-Boot` ， `spring-boot-dependencies`  帮我们完成管理了一套所以来的版本，我们在使用过程中只要引入外部依赖时，仅要声明 `groupId` 和 `artifactId` 即可。 不用再担心版本号冲突问题。

~~~xml

<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-dependencies</artifactId>
    <version>${spring-boot.version}</version>
    <type>pom</type>
    <scope>import</scope>
</dependency>

~~~

因为项目构建技术在发展，我们新项目全部改用 `Gradle` 技术，所以只演示 `Gradle` 构建 `BOM` 的过程。

 `Gradle` 在早先的时候并不具备原生 `BOM`，只能通过其他非常不合理的手段或者方式引用 `Maven BOM`，在 6.0  版本，  `Gradle` 官方提供一款名为 `java-platform` 插件来实现类似 `BOM` 的方式，较此前的实现方式，更加简洁。

## 2. 环境准备

- IDEA 2021+
- Gradle 7.0

![20220821194706](https://abram.oss-cn-shanghai.aliyuncs.com/blog/gateway/apisix/20220821194706.png)

详细环境信息参考 [[Gradle-01：0基础入门]]

## 3. 实现步骤

### 3.1. 构建Gradle项目

- IDEA 中点击菜单 `File` -> `New` -> `Project` -> `Gradle` -> `Java` ，然后点击 `Next` 

![20220821195011](https://abram.oss-cn-shanghai.aliyuncs.com/blog/gradle/env/20220821195011.png)

- 填写项目信息，然后点击 Finish

![20220821195113](https://abram.oss-cn-shanghai.aliyuncs.com/blog/gradle/env/20220821195113.png)

- 等待构建依赖环境

![20220821195149](https://abram.oss-cn-shanghai.aliyuncs.com/blog/gradle/env/20220821195149.png)

![20220821195221](https://abram.oss-cn-shanghai.aliyuncs.com/blog/gradle/env/20220821195221.png)

最后将通过 `IDEA` 构建项目 `build.gradle` 中的 `dependencies` 和 `test` 节点删除。

~~~gradle

dependencies {
    testImplementation 'org.junit.jupiter:junit-jupiter-api:5.7.0'
    testRuntimeOnly 'org.junit.jupiter:junit-jupiter-engine:5.7.0'
}

test {
    useJUnitPlatform()
}

~~~

### 3.2. 编写build.gradle

#### 3.2.1. plugins

- `java-platform`   :  `Gradle` 官方提供 `BOM` 插件
- `maven-publish`   : 构建将项目发布的插件

~~~gradle

plugins {
    // 引入 java-platform 插件
    id 'java-platform' 
    // 发布插件，可用来发布 BOM 或 jar到本地与远程仓库
    id 'maven-publish' 
}

~~~

配置完成上述插件完后，此时编译项目，会在右侧 `Gradle` 任务模块下多个任务分类，名为 `publishing` 。

![20220821200324](https://abram.oss-cn-shanghai.aliyuncs.com/blog/gradle/env/20220821200324.png)

#### 3.2.2. javaPlatform

该配置为防止用户错误地引入依赖，而不是引入依赖约束，如果引入依赖会报错失败。通过这个配置可以让 `Gradle` 允许引入依赖。 

这是一个可选项。

~~~gradle

javaPlatform {
    allowDependencies()
}

~~~

#### 3.2.3. dependencies定义依赖

演示需要，只定义两个外部三方依赖。

~~~gradle

dependencies {
    constraints {
        api 'io.github.rothschil:common-utils:1.2.6.RELEASE'
        api 'io.github.rothschil:persistence-mybatis:1.2.6.RELEASE'
    }
}

~~~

#### 3.2.4. publishing发布

这里只是将项目发布到 `Maven` 本地仓库。

- 执行 `Gradle` 任务 `publishToMavenLocal` 

- `IDEA`  执行发布很快，因为当前任务少

![20220821201306](https://abram.oss-cn-shanghai.aliyuncs.com/blog/gradle/env/20220821201306.png)

-  `Maven` 本地仓库就有我们刚才发布的模块

![20220821201442](https://abram.oss-cn-shanghai.aliyuncs.com/blog/gradle/env/20220821201442.png)

~~~gradle

publishing {
    publications {
        myMvnLocalRepertory(MavenPublication) {
            from components.javaPlatform
        }
    }
}

~~~

### 3.3. publishing发布中央仓库

 `Gradle`  发布中央仓库，可以参考 [[Gradle-03：Gradle项目发布中央仓库]]

#### 3.3.1. plugins

~~~gradle

plugins {
    id 'java-platform'
    id 'maven-publish'
    id 'signing'
    id 'jacoco'
}

~~~

#### 3.3.2. publishing

需要在中央仓库中额外申请，申请过程不做赘述，自行百度。

~~~gradle

oss_name=${OSS申请}
oss_password=${OSS申请过程中密码}

publishing {
    publications {
        toSonaType(MavenPublication) {
            groupId  "$project.group"
            artifactId "$project.name"
            version "$project.version"
            from components.javaPlatform
            pom {
                name = "rothschil-common"
                description = "Pippin, was a Hobbit of the Shire, and one of Frodo Baggins' youngest, but closest friends. He was a member of the Fellowship of the Ring and later became the thirty-second Thain of the Shire"
                url = "https://github.com/rothschil/rothschil-common"

                // 添加你的 git 仓库 信息
                scm {
                    connection= "scm:git:https://github.com/rothschil/rothschil-common.git"
                    developerConnection= "scm:git:https://github.com/rothschil/rothschil-common.git"
                    url= "https://github.com/rothschil/rothschil-common"
                }

                licenses {
                    license {
                        name ="The Apache License, Version 2.0"
                        url ="http://www.apache.org/licenses/LICENSE-2.0.txt"
                    }
                }
                developers {
                    // 添加开发者描述
                    developer {
                        id ="xx"
                        name ="xxx"
                        email ="xx"
                    }
                }
            }
        }
    }
    repositories {

        maven {
            name 'snapshot'
            url 'https://s01.oss.sonatype.org/content/repositories/snapshots/'
            credentials {
                username(oss_name)
                password(oss_password)
            }
        }

        maven {
            name 'release'
            url 'https://s01.oss.sonatype.org/content/repositories/releases/'
            credentials {
                username(oss_name)
                password(oss_password)
            }
        }
    }
}

signing {
    sign publishing.publications.toSonaType
}

~~~

## 4. 应用

- repositories: 我是将 `BOM` 发布在本地 `Maven` 仓库，引用 `mavenLocal` 本地库
- implementation:   platform("io.github.rothschil:common-bom:1.0.0-SNAPSHOT") 配置我发布的 `BOM` 依赖，记得带版本号，此时定义再用 `io.github.rothschil:persistence-mybatis` 已经不需要带版本号。


~~~gradle

plugins {
    id 'java'
}

group 'io.github.rothschil'
version '1.0-SNAPSHOT'

repositories {
    mavenLocal()
}

dependencies {
    implementation platform("io.github.rothschil:common-bom:1.0.0-SNAPSHOT")
    implementation("io.github.rothschil:persistence-mybatis")
}


~~~

在 `External Libraries`  中发现已经有我们需要的依赖包。

![20220821203633](https://abram.oss-cn-shanghai.aliyuncs.com/blog/gradle/env/20220821203633.png)