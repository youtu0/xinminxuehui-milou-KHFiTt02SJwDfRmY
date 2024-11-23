
在之前讲解 Nacos 注册中心的过程中，我曾简要提到过 gRPC，主要是因为 Nacos 的最新版已经采用了 gRPC 作为其核心通信协议。这一变化带来了显著的性能优化，尤其在心跳检测、健康检查等接口的消息传输上，gRPC 可以有效减少网络负担和延迟，从而提高系统的整体效率。


所以，今天我们将简要了解一下 gRPC 这种通信协议是如何运作的，并通过一个简单的 HelloWorld 示例来展示它的基本使用方式。


# gRPC


gRPC（全称：gRPC Remote Procedure Call）是一个由 Google 开发的高性能、开源的远程过程调用（RPC）框架，它基于 HTTP/2 协议，并且采用了 Protocol Buffers（Protobuf）作为接口定义语言。gRPC 旨在简化和优化微服务架构中的服务间通信，提供高效、可靠的通信机制，适用于大规模分布式系统。


gRPC 的通信流程大致如下：


* 接口定义：开发者使用 Protobuf 定义服务接口（.proto 文件）。该文件描述了服务的 RPC 方法、请求和响应消息类型。
* 代码生成：使用 Protobuf 编译器（protoc）根据 .proto 文件生成对应语言的客户端和服务器代码。
* 客户端调用：客户端通过 gRPC 客户端 API 调用远程方法，发送请求并接收响应。gRPC 客户端和服务器之间的通信是透明的，客户端只需要像调用本地方法一样调用远程方法。
* 服务器实现：服务器端实现 .proto 文件中定义的 RPC 方法，并通过 gRPC 框架处理客户端的请求。
* 通信协议：gRPC 使用 HTTP/2 协议进行高效的通信，基于 Protobuf 编码和解码请求和响应数据。


# 快速入门


## 新建项目


按照我们刚才讨论的通信流程，接下来我们将通过一个简单的示例来实现这一过程。首先，我们需要创建一个新的项目，项目的名称可以根据个人喜好进行命名。如图所示：


![image](https://img2024.cnblogs.com/blog/1423484/202411/1423484-20241119161016391-1907631678.png)


### 项目结构


接下来，我们将在刚才创建的项目中进一步细化结构，我们整体的项目结构如下：



> │ pom.xml
> │
> ├─grpc\-api
> │ │ .gitignore
> │ │ pom.xml
> │ │
> │ ├─src
> │ │ ├─main
> │ │ │ ├─java
> │ │ │ │ └─org
> │ │ │ │ └─xiaoyu
> │ │ │ │ │ Main.java
> │ │ │ │ └─test
> │ │ │ └─proto
> │ │ │ hello.proto
> ├─grpc\-client
> │ │ pom.xml
> │ ├─src
> │ │ ├─main
> │ │ │ ├─java
> │ │ │ │ └─org
> │ │ │ │ └─xiaoyu
> │ │ │ │ │ Main.java
> │ │ │ └─resources
> │ │ │ application.yml
> └─grpc\-server
> │ pom.xml
> ├─src
> │ ├─main
> │ │ ├─java
> │ │ │ └─org
> │ │ │ └─xiaoyu
> │ │ │ │ Main.java
> │ │ │ └─service
> │ │ │ HelloWorldController.java
> │ │ └─resources
> │ │ application.yaml


跟着上面的目录结构，我们需要创建子项目，如图所示：


![image](https://img2024.cnblogs.com/blog/1423484/202411/1423484-20241119161025146-1913332693.png)


接下来，我们将配置父项目与各子项目之间的依赖关系，以确保它们能够正确地协同工作。


### 项目依赖


父项目依赖如下：



```
xml version="1.0" encoding="UTF-8"?
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0modelVersion>
    <parent>
        <groupId>org.springframework.bootgroupId>
        <artifactId>spring-boot-starter-parentartifactId>
        <version>2.6.7version>
        <relativePath /> 
    parent>
    <groupId>org.xiaoyugroupId>
    <artifactId>grpc-demoartifactId>
    <version>1.0-SNAPSHOTversion>
    <packaging>pompackaging>
    <modules>
        <module>grpc-clientmodule>
        <module>grpc-servermodule>
        <module>grpc-apimodule>
    modules>

    <properties>
        <maven.compiler.source>8maven.compiler.source>
        <maven.compiler.target>8maven.compiler.target>
        <project.build.sourceEncoding>UTF-8project.build.sourceEncoding>
    properties>

    <dependencies>
        <dependency>
            <groupId>io.grpcgroupId>
            <artifactId>grpc-protobufartifactId>
            <version>1.51.0version>
        dependency>

        <dependency>
            <groupId>io.grpcgroupId>
            <artifactId>grpc-stubartifactId>
            <version>1.51.0version>
        dependency>

        <dependency>
            <groupId>io.grpcgroupId>
            <artifactId>grpc-netty-shadedartifactId>
            <version>1.51.0version>
        dependency>

        <dependency>
            <groupId>com.google.protobufgroupId>
            <artifactId>protobuf-java-utilartifactId>
            <version>3.7.1version>
        dependency>

        <dependency>
            <groupId>com.googlecode.protobuf-java-formatgroupId>
            <artifactId>protobuf-java-formatartifactId>
            <version>1.4version>
        dependency>

    dependencies>

    <build>
        <extensions>
            <extension>
                <groupId>kr.motd.mavengroupId>
                <artifactId>os-maven-pluginartifactId>
                <version>1.6.2version>
            extension>
        extensions>
        <plugins>
            <plugin>
                <groupId>org.xolstice.maven.pluginsgroupId>
                <artifactId>protobuf-maven-pluginartifactId>
                <version>0.6.1version>
                <configuration>
                    <protocArtifact>com.google.protobuf:protoc:3.12.0:exe:${os.detected.classifier}protocArtifact>
                    <pluginId>grpc-javapluginId>
                    <pluginArtifact>io.grpc:protoc-gen-grpc-java:1.34.1:exe:${os.detected.classifier}pluginArtifact>
                    
                    <outputDirectory>${project.basedir}/src/main/javaoutputDirectory>
                    
                    <clearOutputDirectory>falseclearOutputDirectory>
                configuration>
                <executions>
                    <execution>
                        <goals>
                            <goal>compilegoal>
                            <goal>compile-customgoal>
                        goals>
                    execution>
                executions>
            plugin>

            
            <plugin>
                <groupId>org.codehaus.mojogroupId>
                <artifactId>build-helper-maven-pluginartifactId>
                <version>3.0.0version>
                <executions>
                    
                    <execution>
                        <id>add-sourceid>
                        <phase>generate-sourcesphase>
                        <goals>
                            <goal>add-sourcegoal>
                        goals>
                        <configuration>
                            <sources>
                                <source>${project.basedir}/src/main/gensource>
                                <source>${project.basedir}/src/main/javasource>
                            sources>
                        configuration>
                    execution>
                executions>
            plugin>

        plugins>
    build>
project>

```

client项目依赖如下：



```
xml version="1.0" encoding="UTF-8"?
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0modelVersion>
    <parent>
        <groupId>org.xiaoyugroupId>
        <artifactId>grpc-demoartifactId>
        <version>1.0-SNAPSHOTversion>
    parent>

    <artifactId>grpc-clientartifactId>

    <properties>
        <maven.compiler.source>8maven.compiler.source>
        <maven.compiler.target>8maven.compiler.target>
        <project.build.sourceEncoding>UTF-8project.build.sourceEncoding>
    properties>
    <dependencies>
        <dependency>
            <groupId>org.springframework.bootgroupId>
            <artifactId>spring-boot-starterartifactId>
        dependency>
        <dependency>
            <groupId>org.springframework.bootgroupId>
            <artifactId>spring-boot-starter-webartifactId>
        dependency>
        <dependency>
            <groupId>net.devhgroupId>
            <artifactId>grpc-client-spring-boot-starterartifactId>
            <version>2.14.0.RELEASEversion>
        dependency>
        <dependency>
            <groupId>org.xiaoyugroupId>
            <artifactId>grpc-apiartifactId>
            <version>1.0-SNAPSHOTversion>
            <scope>compilescope>
        dependency>
    dependencies>
project>

```

server的项目依赖如下：



```
xml version="1.0" encoding="UTF-8"?
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0modelVersion>
    <parent>
        <groupId>org.xiaoyugroupId>
        <artifactId>grpc-demoartifactId>
        <version>1.0-SNAPSHOTversion>
    parent>

    <artifactId>grpc-serverartifactId>

    <properties>
        <maven.compiler.source>8maven.compiler.source>
        <maven.compiler.target>8maven.compiler.target>
        <project.build.sourceEncoding>UTF-8project.build.sourceEncoding>
    properties>

    <dependencies>
        <dependency>
            <groupId>net.devhgroupId>
            <artifactId>grpc-server-spring-boot-starterartifactId>
            <version>2.14.0.RELEASEversion>
        dependency>
        <dependency>
            <groupId>org.xiaoyugroupId>
            <artifactId>grpc-apiartifactId>
            <version>1.0-SNAPSHOTversion>
            <scope>compilescope>
        dependency>
    dependencies>

project>

```

### proto接口定义


接下来，我们需要生成客户端和服务端的接口定义，并基于这些接口定义自动生成相关的代码。接口定义是整个 gRPC 通信的核心，它将明确服务端提供的 API 接口及其对应的消息格式，这为客户端和服务端之间的通信提供了基础。


为了高效地生成这些接口代码，我们可以借助一些自动化工具和 AI 助手来加速这一过程，简单问下即可。如图所示：


![image](https://img2024.cnblogs.com/blog/1423484/202411/1423484-20241119161039820-172982213.png)


然后复制过来简单改一下，代码如下：



```
syntax = "proto3";
import "google/protobuf/any.proto";

package org.xiaoyu.test;

option java_multiple_files = true;
option java_package = "org.xiaoyu.test";
option objc_class_prefix = "HelloWorld";

// 定义服务
service Greeter {
    // 定义一个 SayHello 方法，接收一个 HelloRequest 类型的请求，并返回一个 HelloReply 类型的响应
    rpc SayHello (HelloRequest) returns (HelloReply) {}
}

// 定义请求消息类型
message HelloRequest {
    string name = 1;
}

// 定义响应消息类型
message HelloReply {
    string message = 1;
}

```

完成接口定义后，我们只需通过执行 `mvn compile` 命令，直接对 API 项目进行编译，即可自动生成与接口定义相关的所有代码类。这个过程将会根据我们在 `.proto` 文件中定义的 gRPC 服务和消息结构，自动生成相应的客户端和服务端代码，包括 Java 类、存根（stub）、消息类等。


![image](https://img2024.cnblogs.com/blog/1423484/202411/1423484-20241119161046997-289114413.png)


## 服务端


接下来，如果在编译或生成代码的过程中仍然遇到问题，或者对某些步骤存在疑问，完全可以继续向 AI 助手寻求帮助。AI 助手可以为我们提供针对性的问题解决方案和调试建议，无论是编译错误、依赖问题，还是代码生成后的一些配置问题，都能快速给出指导。如图所示：


![image](https://img2024.cnblogs.com/blog/1423484/202411/1423484-20241119161053953-770741252.png)


在生成了相关代码后，我们可以直接将这些代码复制到 `server` 项目中，方便我们进行服务端的开发和集成。同样地，当我们开始进行具体业务逻辑的实现时，AI 助手也能发挥重要作用。在实现服务端逻辑时，AI 助手不仅能自动补全代码，还可以基于项目的上下文和需求，智能推荐最佳的实现方式。


![image](https://img2024.cnblogs.com/blog/1423484/202411/1423484-20241119161102124-1896685056.png)


最终代码如下，很简单：



```
@GrpcService
public class HelloWorldController extends GreeterGrpc.GreeterImplBase {

    @Override
    public void sayHello(HelloRequest request, StreamObserver responseObserver) {

        responseObserver.onNext(HelloReply.newBuilder().setMessage("xiaoyu: Hello " + request.getName()).build());
        responseObserver.onCompleted();
    }
}

```

### 服务端配置


我们需要简单配置一下服务端的监听接口，如下：



```
grpc:
  server:
    port: 9090

```

剩下的步骤就是启动我们已经集成了 gRPC 服务的 Spring 项目。当我们运行项目时，Spring Boot 应用会自动加载并初始化相关的 gRPC 服务配置，并开始监听指定的端口。


![image](https://img2024.cnblogs.com/blog/1423484/202411/1423484-20241119161108822-839646631.png)


## 客户端


在客户端部分，我们需要进行一些手动配置，虽然目前尚未找到能够直接启动并自动配置的注解或工具。在这个阶段，客户端代码的编写相对简单，主要是利用从 API 项目生成的客户端代码来完成对服务端的请求调用。尽管这部分无法完全自动化，我们可以通过直接使用生成的客户端代码来手动构建请求对象，并发起 gRPC 调用，从而实现与服务端的通信。如下：



```
public class Main {
    public static void main(String[] args) {
        ManagedChannel channel = ManagedChannelBuilder.forAddress("localhost", 9090)
                .usePlaintext()
                .build();

        // 创建一元式阻塞式存根
        GreeterGrpc.GreeterBlockingStub blockingStub = GreeterGrpc.newBlockingStub(channel);

        // 创建请求对象
        HelloRequest request = HelloRequest.newBuilder()
                .setName("World")
                .build();

        // 发送请求信息并接收响应
        HelloReply response = blockingStub.sayHello(request);

        // 处理信息响应
        System.out.println("Received  response: " + response);

    }
}

```

只需直接运行项目，即可成功启动并完成服务的所有配置。与其他服务接口的调用方式类似，我们在这里无需编写任何额外的接口调用代码，看下运行结果，如图所示：


![image](https://img2024.cnblogs.com/blog/1423484/202411/1423484-20241119161114418-1479676816.png)


# 总结


通过本文的讲解，我们了解了 gRPC 作为一种高效的通信协议在微服务架构中的应用，特别是在与 Nacos 集成时带来的性能优化。gRPC 的高效性，得益于其基于 HTTP/2 协议和 Protobuf 的数据传输方式，使得跨服务通信更加迅速且可靠。本文通过 HelloWorld 示例演示了从接口定义到服务端和客户端的实现流程，充分展示了 gRPC 的强大功能和易用性。


掌握这一协议，不仅能够提升服务间的通信效率，也为开发更具扩展性的分布式系统奠定了基础。最终，gRPC 作为微服务架构中的关键组件，其提供的性能优化和便捷性，必将在未来的项目中发挥重要作用。




---


我是努力的小雨，一名 Java 服务端码农，潜心研究着 AI 技术的奥秘。我热爱技术交流与分享，对开源社区充满热情。同时也是一位腾讯云创作之星、阿里云专家博主、华为云云享专家、掘金优秀作者。


💡 我将不吝分享我在技术道路上的个人探索与经验，希望能为你的学习与成长带来一些启发与帮助。


🌟 欢迎关注努力的小雨！🌟


 本博客参考[豆荚加速器](https://yirou.org)。转载请注明出处！
