<!-- TOC -->

- [关于跨语言远程过程调用框架（RPC）——thrift的学习](#关于跨语言远程过程调用框架rpcthrift的学习)
    - [安装和官方样例测试：](#安装和官方样例测试)
        - [windows环境下的编译器的安装：](#windows环境下的编译器的安装)
        - [编译并安装开发环境：](#编译并安装开发环境)
        - [官方测试：](#官方测试)
    - [相同语言的服务调用测试——java：](#相同语言的服务调用测试java)
        - [编写IDL文件来描述被访问的接口：](#编写idl文件来描述被访问的接口)
        - [从IDL生成java文件：](#从idl生成java文件)
        - [用java实现定义的接口：](#用java实现定义的接口)
        - [实现java的服务端代码：](#实现java的服务端代码)
        - [实现java的客户端代码：](#实现java的客户端代码)
        - [测试与服务端通信：](#测试与服务端通信)
    - [跨语言的服务调用测试——java和C++调用交互：](#跨语言的服务调用测试java和c调用交互)
        - [windows环境下thrift源代码的编译：](#windows环境下thrift源代码的编译)
            - [依赖文件清单：](#依赖文件清单)
            - [依赖关系说明：](#依赖关系说明)
            - [编译过程：](#编译过程)
        - [从IDL生成cpp源文件：](#从idl生成cpp源文件)
        - [编写C++的客户端：](#编写c的客户端)
            - [cpp的客户端开发中添加依赖于boost提供的文件：](#cpp的客户端开发中添加依赖于boost提供的文件)
            - [cpp的客户端开发中添加依赖于thrift提供的文件：](#cpp的客户端开发中添加依赖于thrift提供的文件)
            - [cpp的客户端开发中添加依赖于openssl提供的文件：](#cpp的客户端开发中添加依赖于openssl提供的文件)
            - [编译出现问题：](#编译出现问题)
            - [手动编译openssl：](#手动编译openssl)
        - [编写C++的服务端：](#编写c的服务端)
            - [windows环境下编写C++的服务端：](#windows环境下编写c的服务端)
            - [linux环境下thrift源代码的编译：](#linux环境下thrift源代码的编译)
                - [指定版本的boost编译：](#指定版本的boost编译)
                - [指定版本的OpenSSL编译：](#指定版本的openssl编译)
                - [指定版本的libevent编译：](#指定版本的libevent编译)
                - [使用thrift默认configure进行编译：](#使用thrift默认configure进行编译)
                - [采用系统支持包，使用cmake对thrift进行编译：](#采用系统支持包使用cmake对thrift进行编译)
                - [单独对thrift的C++库支持进行编译：](#单独对thrift的c库支持进行编译)
            - [linux下的C++服务端编写测试：](#linux下的c服务端编写测试)
    - [使用thrift完成其他语言之间的相互调用：](#使用thrift完成其他语言之间的相互调用)
    - [thrift基本语法学习：](#thrift基本语法学习)
    - [使用thrift做JDBC开发：](#使用thrift做jdbc开发)
    - [与其他PRC框架对比：](#与其他prc框架对比)
    - [源代码分析：](#源代码分析)

<!-- /TOC -->

# 关于跨语言远程过程调用框架（RPC）——thrift的学习

因为最近项目需要将已经有的java服务提供给其他开发人员使用，那么跨语言的过程调用就显得非常重要，facebook开源了他的RPC框架thrift能够很好的解决不同语言之间的过程调用问题，现在学习记录如下。

## 安装和官方样例测试：

### windows环境下的编译器的安装：
访问官方网站的[下载链接](http://thrift.apache.org/download)，可以直接获windows下编译好的可执行文件，这个文件的作用是将IDL转换为不同语言的源代码，被称为Thrift compiler。
新建目录，C:\Program Files\thrift，把下载好的thrift-0.10.0.exe文件放在里面，然后修改文件名为thrift.exe，然后把C:\Program Files\thrift添加到windows下面的环境变量PATH中。
在cmd命令行中输入：
```shell
thrift -version
```
可以看到当前thrift编译器的版本信息，表示可以使用thrift编译器将IDL进行转换了，编译器安装成功。

### 编译并安装开发环境：
因为thrift可执行文件只是包含了转换，没有提供其他程序开发时的库支持，所以需要在每一个平台上具体的进行编译来获取支持开发的lib库，例如java的jar包，C++的静态库等。
可以参考[官方编译文档](http://thrift.apache.org/docs/install/)查看具体每一个平台上的编译和安装。
对于java开发而言，已经给出了编译好的maven源，可以直接使用；对于C++而言，因为没有完善的库管理方案，就只能自己手动编译了。

### 官方测试：
从官方网站的下载页，获取thrift的源代码包，目前最近为：thrift-0.10.0.tar.gz。解压后进入tutorial目录。
这个目录中包含官方提供的shared.thrift和tutorial.thrift这两个IDL（Interface Definition Language）文件所描述服务的被支持的各种语言的测试文件，可以选择我们需要进行测试的语言进行测试。
所谓的IDL，就是接口定义，thrift通过自己定义的语法来对接口进行描述，从而使用这种DSL来完成对不同程序开发语言接口实现的转换。
拷贝官方的IDL文件到一个空目录，然后执行：
```shell
thrift -r --gen cpp tutorial.thrift
```
从IDL生成其他语言的命令的一般格式为：
```shell
thrift --gen <language> <Thrift filename>
```

这个命令会生成一个gen-cpp文件夹，里面包含这两个IDL文件同名的文件夹来包含对应的C++的接口实现，我们需要做的就是补全这个接口实现中的客户端和服务端。
因为这个是官方的教程文件，所以已经在tutorial.thrift同级的目录下放置了cpp文件夹，其中就包含了我们原本需要自己实现的客户端和服务端代码。
我们只需要将这个文件夹下的源代码添加到工程中，并且引入生成的cpp文件到这个工程中，添加需要的boots库和thrift静态库等到工程中就可以进行编译测试了。


## 相同语言的服务调用测试——java：
在了解了上述基本流程后，我们实际编写一个供java客户端调用的java服务端。选择java是因为整体流程相对适中，不需要C++进行本地编译，也不想动态语言过于简单，能够很好的描述整个开发流程。

具体流程为：使用thrift定义的IDL作为中间接口，完成实现这个接口的java服务端，然后提供调用这个接口的java客户端。
> - 参考文档：[Apache Thrift - 可伸缩的跨语言服务开发框架](https://www.ibm.com/developerworks/cn/java/j-lo-apachethrift/)

### 编写IDL文件来描述被访问的接口：
新建一个空的hello.thrift文件，输入：
```thrift
namespace java service.demo 
service Hello{ 
    string helloString(1:string para) 
    i32 helloInt(1:i32 para) 
    bool helloBoolean(1:bool para) 
    void helloVoid() 
    string helloNull() 
}
```
这个文件就定义了一个服务Hello，并且这个服务中定义了五个方法，每个方法包含一个方法名，参数列表和返回类型。每个参数包括参数序号，参数类型以及参数名。
需要注意的是上述内容使用了thrift的IDL语言进行编写，具体细节后续进行学习。
可以将这个thrift文件放在java工程的resource文件夹下，供后续修改时重新编译使用。

### 从IDL生成java文件：
使用命令：
```shell
thrift --gen java hello.thrift
```
会在当前目录下生成文件：
```shell
│  hello.thrift
│
└─gen-java
    └─service
        └─demo
           └─Hello.java
```

正好对应于namespace中设置的包路径：service.demo。
这个Hello.java文件就是IDL中描述接口的java接口定义，我们需要将这个接口定义进行实现，从而完成客户端和服务端具体实现。

打开Hello.java可以看到主体代码为：
```java
public class Hello {

  public interface Iface {

    public java.lang.String helloString(java.lang.String para) throws org.apache.thrift.TException;

    public int helloInt(int para) throws org.apache.thrift.TException;

    public boolean helloBoolean(boolean para) throws org.apache.thrift.TException;

    public void helloVoid() throws org.apache.thrift.TException;

    public java.lang.String helloNull() throws org.apache.thrift.TException;

  }
  
  public static class Client extends org.apache.thrift.TServiceClient implements Iface {
    ...
  }

  public static class AsyncClient extends org.apache.thrift.async.TAsyncClient implements AsyncIface {
      ...
  }
  
  public void helloInt(int para, org.apache.thrift.async.AsyncMethodCallback<java.lang.Integer> resultHandler) throws org.apache.thrift.TException {
      ...
  }
  
  public static class helloInt_call extends org.apache.thrift.async.TAsyncMethodCall<java.lang.Integer> {
      ...
  }
  
  public void helloBoolean(boolean para, org.apache.thrift.async.AsyncMethodCallback<java.lang.Boolean> resultHandler) throws org.apache.thrift.TException {
      ...
  }

  public static class helloBoolean_call extends org.apache.thrift.async.TAsyncMethodCall<java.lang.Boolean> {
      ...
  }
  ...

  public static class Processor<I extends Iface> extends org.apache.thrift.TBaseProcessor<I> implements org.apache.thrift.TProcessor {
      ...
  }
  
  public static class AsyncProcessor<I extends AsyncIface> extends org.apache.thrift.TBaseAsyncProcessor<I> {
      ...
  }
  
  ...
}
```
可以看到这个文件包含了在 Hello.thrift 文件中描述的服务 Hello 的接口定义，即 Hello.Iface 接口，以及服务调用的底层通信细节，包括客户端的调用逻辑 Hello.Client 以及服务器端的处理逻辑 Hello.Processor，用于构建客户端和服务器端的功能。
具体的生成文件的细节需要后续继续学习。

### 用java实现定义的接口：
创建 HelloServiceImpl.java 文件并实现 Hello.java 文件中的 Hello.Iface 接口，内容为：
```java
package service.demo;

import org.apache.thrift.TException;

public class HelloServiceImpl implements Hello.Iface { 
    @Override 
    public boolean helloBoolean(boolean para) throws TException { 
        return para; 
    } 
    @Override 
    public int helloInt(int para) throws TException { 
        try { 
            Thread.sleep(20000); 
        } catch (InterruptedException e) { 
            e.printStackTrace(); 
        } 
        return para; 
    } 
    @Override 
    public String helloNull() throws TException { 
        return null; 
    } 
    @Override 
    public String helloString(String para) throws TException { 
        return para; 
    } 
    @Override 
    public void helloVoid() throws TException { 
        System.out.println("Hello World"); 
    } 
 }
```
这个代码完成了IDL中接口的具体实现细节，也就是完成了接口的功能实现。

### 实现java的服务端代码：
创建服务器端实现代码，将 HelloServiceImpl 作为具体的处理器传递给 Thrift 服务器，具体的代码为：
```java
package service.demo;

import org.apache.thrift.TProcessor;
import org.apache.thrift.protocol.TBinaryProtocol;
import org.apache.thrift.protocol.TBinaryProtocol.Factory;
import org.apache.thrift.server.TServer;
import org.apache.thrift.server.TSimpleServer;
import org.apache.thrift.transport.TServerSocket;
import org.apache.thrift.transport.TTransportException;

public class HelloServiceServer {
    public static void main(String[] args) {
        try {
            TServerSocket serverTransport = new TServerSocket(7911);
            Factory proFactory = new TBinaryProtocol.Factory();
            TProcessor processor = new Hello.Processor(new HelloServiceImpl());
            TServer server = new TSimpleServer(new TServer.Args(serverTransport).processor(processor));
            //TServer server = new TThreadPoolServer(processor, serverTransport,  proFactory);
            System.out.println("Start server on port 7911...");
            server.serve();
        } catch (TTransportException e) {
            e.printStackTrace();
        }
    }
}
```
这段代码就完成了接口的java端服务器的实现，通过运行这个代码，就得到了IDL中描述接口的具体实现的服务端。

将上述使用IDL生成的Hello.java源文件，还有两个自己手动补充的源文件均放置在service.demo包路径下，使用IDEA新建gradle工程，然后编辑build.gradle文件内容，添加thrift支持：
```gradle
group 'hellmonky'
version '1.0-SNAPSHOT'
apply plugin: 'java'
sourceCompatibility = 1.8

repositories {
    mavenCentral()
}

dependencies {
    testCompile group: 'junit', name: 'junit', version: '4.11'

    compile("org.apache.thrift:libthrift:0.10.0")
}

task runnbaleJar(type: Jar) {
    from files(sourceSets.main.output.classesDir)
    from configurations.runtime.asFileTree.files.collect { zipTree(it) }
    manifest {
        attributes 'Main-Class': 'service.demo.HelloServiceServer'
    }
}
```
需要注意的是，其中的task runnbaleJar创建了一个完整的可执行的服务端jar包，否则会因为库的不完整报JNI错误。然后使用：
```java
java -jar hellmonky-1.0-SNAPSHOT.jar
```
回显：
```java
SLF4J: Failed to load class "org.slf4j.impl.StaticLoggerBinder".
SLF4J: Defaulting to no-operation (NOP) logger implementation
SLF4J: See http://www.slf4j.org/codes.html#StaticLoggerBinder for further details.
Start server on port 7911...
```
就表示已经正常的启动运行java的后台服务了。

### 实现java的客户端代码：
在完成了上述满足接口规范的java后端服务器开发后，就可以继续从IDL中出发来完成其他语言的客户端的开发，这儿为了简化理解，还是使用java来开发客户端。
在当前工程目录下新建包client，然后实现HelloServiceClient.java文件：
```java
package service.demo.client;

/**
 * Created by 文涛 on 2017/2/14.
 */
import org.apache.thrift.TException;
import org.apache.thrift.protocol.TBinaryProtocol;
import org.apache.thrift.protocol.TProtocol;
import org.apache.thrift.transport.TSocket;
import org.apache.thrift.transport.TTransport;
import org.apache.thrift.transport.TTransportException;
import service.demo.autogen.Hello;

public class HelloServiceClient {
    public static void main(String[] args) {
        try {
            TTransport transport = new TSocket("localhost", 7911);
            transport.open();
            TProtocol protocol = new TBinaryProtocol(transport);
            Hello.Client client = new Hello.Client(protocol);
            client.helloVoid();
            transport.close();
        } catch (TTransportException e) {
            e.printStackTrace();
        } catch (TException e) {
            e.printStackTrace();
        }
    }
}
```

### 测试与服务端通信：
将当前HelloServiceClient.java文件作为执行入口，可以看到执行结果返回为：
```shell
SLF4J: Failed to load class "org.slf4j.impl.StaticLoggerBinder".
SLF4J: Defaulting to no-operation (NOP) logger implementation
SLF4J: See http://www.slf4j.org/codes.html#StaticLoggerBinder for further details.
Received 1

Process finished with exit code 0
```

同时，在server的启动窗口可以看到执行返回：
```shell
SLF4J: Failed to load class "org.slf4j.impl.StaticLoggerBinder".
SLF4J: Defaulting to no-operation (NOP) logger implementation
SLF4J: See http://www.slf4j.org/codes.html#StaticLoggerBinder for further details.
Start server on port 7911...
Hello World
```
至此，一个使用thrift的IDL定义的接口的java服务端实现和客户端调用接口就可以正常运行了。


## 跨语言的服务调用测试——java和C++调用交互：
完成了上述相同语言的RPC调用后，可以思考一下跨语言的服务调用过程。既然是语言无关的，那么最好的方式就是通过web进行交互，因为web本身的设计就考虑到了多语言多运行环境的交互。

在实际工程开发中，往往是已经存在一个后台服务，需要面向其他语言开发者提供服务的访问，常见的方式就是通过JSON作为数据，然后使用HTTP或者Socket作为访问方式来进行语言无关的服务提供。之前就是使用RESTFul API作为服务，C++端使用curl访问HTTP进行交互。
但是有时候也需要首先定义交互接口，然后客户端和服务端同步开发来完成敏捷开发迭代。这个时候就需要首先确定接口规范，然后各自进行开发，使用thrift是一个不错的选择。

不论上述两种情况如何，通过thrift来描述中间的交互接口是一个非常不错的方式，方便前后端分离开发。现在我们还是以上述Hello服务为例，看看C++的客户端该如何实现，从而用java作为后台服务端，供C++客户端访问服务。

### windows环境下thrift源代码的编译：
> -  参考文档：
[Using Thrift with C++](https://thrift.apache.org/lib/cpp)
[Windows Setup](http://thrift.apache.org/docs/install/windows)
[Apache Thrift 在Windows下的安装与开发](http://blog.csdn.net/colouroo/article/details/38588297)
[Thrift在Windows及Linux平台下的安装和使用示例](http://cpper.info/2016/03/06/Thrift-Install-And-Example.html)
[thrift在windows的编译/安装--c++版](http://www.cnblogs.com/mumuxinfei/p/3715721.html)


因为要使用C++进行开发，所以需要针对thrift源代码进行编译来获取对C++的支持。
进入目录：
```shell
..\thrift-0.10.0\lib\cpp
```
可以看出，在windows下提供了vs的工程文件，也可以通过cmake来完成平台无关的thrift编译。最后可以编译生成thrift的lib库文件和头文件供开发人员使用。
现在就用vs工程文件打开的方式对thrift进行编译。

#### 依赖文件清单：
[thrift-0.10.0.tar.gz源码包]()
VS2010
boost库，根据thrift发布版本要求，使用的boost1.53以及上版本，这儿选择sourceforge上发布的预编译好的vs2010的32位安装包:[boost_1_63_0-msvc-10.0-32.exe](https://sourceforge.net/projects/boost/files/?source=navbar)
libevent库，这里用的[libevent-2.1.8-stable.tar.gz](http://libevent.org/)
openssl库，虽然官方给出的[下载链接](https://www.openssl.org/source/)中为了安全不包含二进制包，但是为了方便起见，从[Win32 OpenSSL](http://slproweb.com/products/Win32OpenSSL.html)项目页下载到已经编译好的开发包使用。

#### 依赖关系说明：
Thrift.sln，里面有libthrift和libthriftnb两个工程，其中libthrift工程是常规的阻塞型server端（单线程server，一个连接一个线程server，线程池server），libthriftnb工程是非阻塞（non-blocking）模式的服务server端，也只有编译libthriftnb时才需要依赖libevent库，否则可以不编译libevent库；
thrift使用ssl协议来保证网络传输的安全性。

#### 编译过程：
首先：
安装boost和openssl

然后：从源代码编译libevent。
对于libevent库的编译比较麻烦，因为官方没有给出cmake等工程管理文件，而是使用vs的控制台nmake进行编译，参考：[windows下编译及使用libevent](http://www.cnblogs.com/luxiaoxun/p/3603399.html)。
（1）对libevent库编译，并整理代码和文件提供给thrift使用：
按照上述教程对libevent编译完毕，项目下建一个Lib目录，将上面三个lib文件copy到该目录下。
新建一个include目录，将..\libevent-2.1.8-stable\include下的文件和文件夹copy到该目录下，然后将..\libevent-2.1.8-stable\include\WIN32-Code\nmake\下的文件和目录也copy到该目录下，2个event2目录下的文件可合并一起。
（2）在thrift的nb库编译中添加上述新建的include和lib文件夹，完成对thrift的nb库的编译。
（3）打开thrift工程文件，添加boost、openssl和libevent库的头文件和静态链接库位置，完成编译。


### 从IDL生成cpp源文件：
和java程序相同，为了从IDL的描述中获取cpp源文件，也需要使用thrift来生成一次，进入hello.thrift所在目录，执行：
```shell
thrift --gen cpp hello.thrift
```
会生成gen-cpp文件夹，包含以下文件：
```shell
|─ghello.thrift
│
├─gen-cpp
│      Hello.cpp
│      Hello.h
│      hello_constants.cpp
│      hello_constants.h
│      Hello_server.skeleton.cpp
│      hello_types.cpp
│      hello_types.h
```

### 编写C++的客户端：
需要注意的是，上述生成的cpp文件，只是包含了IDL的服务端相关代码，并不包含客户端调用的代码，所以需要自己新建并编写客户端代码，然后删除生成的HelloService_server.skeleton.cpp，将上述其他的生成的代码一起编译才能正确获取客户端可执行文件。
这是因为客户端的服务代码编写基本上都是通过使用thrift在不同程序开发语言中提供的库来完成远程访问，基本上都非常直接，不需要涉及服务端实现细节的部分。

新建client.cpp文件：
```cpp
#include <stdio.h>
#include <string>

#include <boost/make_shared.hpp>

#include <thrift/protocol/TBinaryProtocol.h>
#include <thrift/transport/TSocket.h>
#include <thrift/transport/TTransportUtils.h>

#include "hello_types.h"
#include "Hello.h"

using namespace boost;
using namespace apache::thrift;
using namespace apache::thrift::protocol;
using namespace apache::thrift::transport;

int main(int argc, char** argv){
    shared_ptr<TTransport> socket(new TSocket("192.168.15.1", 7911));
    shared_ptr<TTransport> transport(new TBufferedTransport(socket));
    shared_ptr<TProtocol> protocol(new TBinaryProtocol(transport));
    HelloClient client(protocol);
	try {
		transport->open();
		client.helloVoid();
		transport->close();
	}
	catch(TException& tx) {
		printf("ERROR:%s\n",tx.what());
	}
}
```
然后对当前项目添加外部依赖。

#### cpp的客户端开发中添加依赖于boost提供的文件：
```shell
#include <boost/make_shared.hpp>
```
需要使用和thrift的编译版依赖的boots库相同的版本。位于boost二进制安装路径下:
```shell
C:\boost_1_63_0\lib32-msvc-10.0\libboost_thread-vc100-mt-gd-1_63.lib
C:\boost_1_63_0\lib32-msvc-10.0\libboost_system-vc100-mt-gd-1_63.lib
C:\boost_1_63_0\lib32-msvc-10.0\libboost_atomic-vc100-mt-gd-1_63.lib
C:\boost_1_63_0\lib32-msvc-10.0\libboost_date_time-vc100-mt-gd-1_63.lib
C:\boost_1_63_0\lib32-msvc-10.0\libboost_chrono-vc100-mt-gd-1_63.lib
```

#### cpp的客户端开发中添加依赖于thrift提供的文件：
```shell
#include <thrift/protocol/TBinaryProtocol.h>
#include <thrift/transport/TSocket.h>
#include <thrift/transport/TTransportUtils.h>
```
位于thrift源代码包的lib文件夹下：
```shell
..\thrift-0.10.0\lib\cpp\src
```
所以在工程中（cmake文件等工程描述文件中）将这个路径添加到项目的附加库头文件依赖路径。
然后添加thrift的静态库：
..\thrift-0.10.0\lib\cpp\Debug\libthrift.lib

#### cpp的客户端开发中添加依赖于openssl提供的文件：
```shell
C:\OpenSSL-Win32\lib\VC\static\libeay32MTd.lib
C:\OpenSSL-Win32\lib\VC\static\ssleay32MTd.lib
```

#### 编译出现问题：
按照上述步骤添加依赖之后，出现：
```shell
1>ssleay32MTd.lib(t1_lib.obj) : error LNK2001: 无法解析的外部符号 ___report_rangecheckfailure
1>libeay32MTd.lib(b_print.obj) : error LNK2019: 无法解析的外部符号 ___report_rangecheckfailure，该符号在函数 _fmtfp 中被引用
1>libeay32MTd.lib(obj_dat.obj) : error LNK2001: 无法解析的外部符号 ___report_rangecheckfailure
1>libeay32MTd.lib(b_dump.obj) : error LNK2001: 无法解析的外部符号 ___report_rangecheckfailure
1>libeay32MTd.lib(pem_lib.obj) : error LNK2001: 无法解析的外部符号 ___report_rangecheckfailure
```
[网上搜索](https://github.com/pyca/cryptography/issues/2024)发现为Win32 OpenSSL使用vs2012编译导致了错误，所以最好的方法就是更新编译器到2012，或者自己编译对应版本的openssl。

#### 手动编译openssl：
既然问题出在openssl上，那么就采用自己手动编译的方式来解决。
从[官方网站](http://downloads.activestate.com/ActivePerl/releases/5.22.3.2204/ActivePerl-5.22.3.2204-MSWin32-x86-64int-401627.exe)下载最新的ActivePerl安装包，完成安装。然后打开vs的控制台，按照如下步骤执行：
```shell
perl Configure VC-WIN32 no-asm --prefix=C:\openssl_lib

ms\do_ms.bat

nmake -f ms\ntdll.mak
nmake -f ms\nt.mak

nmake -f ms\ntdll.mak test
nmake -f ms\nt.mak test

nmake -f ms\ntdll.mak install
nmake -f ms\nt.mak install
```
如果要编译64位，首先需要打开vs的64位控制台，然后修改perl的配置为：
```shell
perl Configure VC-WIN64A no-asm --prefix=C:\openssl_lib
```
还要注意要关闭asm，否则会编译失败，或者使用masm进行预编译处理。

然后使用这个openssl来对thrift进行重新编译，并且客户端的编译也正常通过，完成了C++客户端对java服务器的调用。

> - 参考文档：
[编译openssl出错](http://bbs.csdn.net/topics/390986380)
[VS2010编译OpenSSL](http://blog.csdn.net/zhangmiaoping23/article/details/52815974)
[在Windows系统上安装OpenSSL及在VS2010中使用OpenSSL](http://blog.csdn.net/iw1210/article/details/50947654)


### 编写C++的服务端：
上述java教程中实现了相同语言的服务端和客户端，上一节又直接使用thrift来使用了C++开发了客户端，这一节使用cpp来开发IDL对应的服务端，然后使用java进行调用。

#### windows环境下编写C++的服务端：
按照上一节的内容，使用thrift生成代码，新建vs空项目，然后将gen-cpp生成的文件放在项目中，并且修改默认生成的Hello_server.skeleton.cpp文件，补全接口函数的实现：
```cpp
// This autogenerated skeleton file illustrates how to build a server.
// You should copy it to another filename to avoid overwriting it.

#include <iostream>
#include <stdexcept>
#include <sstream>

#include <boost/make_shared.hpp>

#include <thrift/concurrency/ThreadManager.h>
#include <thrift/concurrency/PlatformThreadFactory.h>
#include <thrift/protocol/TBinaryProtocol.h>
#include <thrift/server/TSimpleServer.h>
#include <thrift/server/TThreadPoolServer.h>
#include <thrift/server/TThreadedServer.h>
#include <thrift/transport/TServerSocket.h>
#include <thrift/transport/TSocket.h>
#include <thrift/transport/TTransportUtils.h>
#include <thrift/TToString.h>

#include "Hello.h"

#pragma comment(lib, "libthrift.lib")

using namespace std;
using namespace boost;
using namespace ::apache::thrift;
using namespace ::apache::thrift::protocol;
using namespace ::apache::thrift::transport;
using namespace ::apache::thrift::server;



class HelloHandler : virtual public HelloIf {
 public:
  HelloHandler() {
    // Your initialization goes here
  }

  void helloString(std::string& _return, const std::string& para) {
    // Your implementation goes here
    printf("helloString\n");
  }

  int32_t helloInt(const int32_t para) {
    // Your implementation goes here
    printf("helloInt\n");
	return 0;
  }

  bool helloBoolean(const bool para) {
    // Your implementation goes here
    printf("helloBoolean\n");
	return true;
  }

  void helloVoid() {
    // Your implementation goes here
    printf("helloVoid\n");
  }

  void helloNull(std::string& _return) {
    // Your implementation goes here
    printf("helloNull\n");
  }

};

int main(int argc, char **argv) {
  int port = 9090;
  boost::shared_ptr<HelloHandler> handler(new HelloHandler());
  boost::shared_ptr<TProcessor> processor(new HelloProcessor(handler));
  boost::shared_ptr<TServerTransport> serverTransport(new TServerSocket(port));
  boost::shared_ptr<TTransportFactory> transportFactory(new TBufferedTransportFactory());
  boost::shared_ptr<TProtocolFactory> protocolFactory(new TBinaryProtocolFactory());

  TSimpleServer server(processor, serverTransport, transportFactory, protocolFactory);
  server.serve();
  return 0;
}
```
补全返回值。然后添加附加的目录和库。完成编译。
ps:目前编译失败，报错信息为：
```shell
1>Hello_server.skeleton.obj : error LNK2019: 无法解析的外部符号 "public: virtual void __thiscall apache::thrift::server::TServerFramework::serve(void)" (?serve@TServerFramework@server@thrift@apache@@UAEXXZ)，该符号在函数 _main 中被引用
1>libthrift.lib(TSimpleServer.obj) : error LNK2001: 无法解析的外部符号 "public: virtual void __thiscall apache::thrift::server::TServerFramework::serve(void)" (?serve@TServerFramework@server@thrift@apache@@UAEXXZ)
1>libthrift.lib(TSimpleServer.obj) : error LNK2019: 无法解析的外部符号 "public: virtual __thiscall apache::thrift::server::TServerFramework::~TServerFramework(void)" (??1TServerFramework@server@thrift@apache@@UAE@XZ)，该符号在函数 __unwindfunclet$??0TSimpleServer@server@thrift@apache@@QAE@ABV?$shared_ptr@VTProcessorFactory@thrift@apache@@@boost@@ABV?$shared_ptr@VTServerTransport@transport@thrift@apache@@@5@ABV?$shared_ptr@VTTransportFactory@transport@thrift@apache@@@5@ABV?$shared_ptr@VTProtocolFactory@protocol@thrift@apache@@@5@@Z$0 中被引用
1>libthrift.lib(TSimpleServer.obj) : error LNK2019: 无法解析的外部符号 "public: virtual void __thiscall apache::thrift::server::TServerFramework::setConcurrentClientLimit(__int64)" (?setConcurrentClientLimit@TServerFramework@server@thrift@apache@@UAEX_J@Z)，该符号在函数 "public: __thiscall apache::thrift::server::TSimpleServer::TSimpleServer(class boost::shared_ptr<class apache::thrift::TProcessorFactory> const &,class boost::shared_ptr<class apache::thrift::transport::TServerTransport> const &,class boost::shared_ptr<class apache::thrift::transport::TTransportFactory> const &,class boost::shared_ptr<class apache::thrift::protocol::TProtocolFactory> const &)" (??0TSimpleServer@server@thrift@apache@@QAE@ABV?$shared_ptr@VTProcessorFactory@thrift@apache@@@boost@@ABV?$shared_ptr@VTServerTransport@transport@thrift@apache@@@5@ABV?$shared_ptr@VTTransportFactory@transport@thrift@apache@@@5@ABV?$shared_ptr@VTProtocolFactory@protocol@thrift@apache@@@5@@Z) 中被引用
1>libthrift.lib(TSimpleServer.obj) : error LNK2019: 无法解析的外部符号 "public: __thiscall apache::thrift::server::TServerFramework::TServerFramework(class boost::shared_ptr<class apache::thrift::TProcessorFactory> const &,class boost::shared_ptr<class apache::thrift::transport::TServerTransport> const &,class boost::shared_ptr<class apache::thrift::transport::TTransportFactory> const &,class boost::shared_ptr<class apache::thrift::protocol::TProtocolFactory> const &)" (??0TServerFramework@server@thrift@apache@@QAE@ABV?$shared_ptr@VTProcessorFactory@thrift@apache@@@boost@@ABV?$shared_ptr@VTServerTransport@transport@thrift@apache@@@5@ABV?$shared_ptr@VTTransportFactory@transport@thrift@apache@@@5@ABV?$shared_ptr@VTProtocolFactory@protocol@thrift@apache@@@5@@Z)，该符号在函数 "public: __thiscall apache::thrift::server::TSimpleServer::TSimpleServer(class boost::shared_ptr<class apache::thrift::TProcessorFactory> const &,class boost::shared_ptr<class apache::thrift::transport::TServerTransport> const &,class boost::shared_ptr<class apache::thrift::transport::TTransportFactory> const &,class boost::shared_ptr<class apache::thrift::protocol::TProtocolFactory> const &)" (??0TSimpleServer@server@thrift@apache@@QAE@ABV?$shared_ptr@VTProcessorFactory@thrift@apache@@@boost@@ABV?$shared_ptr@VTServerTransport@transport@thrift@apache@@@5@ABV?$shared_ptr@VTTransportFactory@transport@thrift@apache@@@5@ABV?$shared_ptr@VTProtocolFactory@protocol@thrift@apache@@@5@@Z) 中被引用
1>libthrift.lib(TSimpleServer.obj) : error LNK2001: 无法解析的外部符号 "public: virtual void __thiscall apache::thrift::server::TServerFramework::stop(void)" (?stop@TServerFramework@server@thrift@apache@@UAEXXZ)
1>libthrift.lib(TSimpleServer.obj) : error LNK2001: 无法解析的外部符号 "public: virtual __int64 __thiscall apache::thrift::server::TServerFramework::getConcurrentClientLimit(void)const " (?getConcurrentClientLimit@TServerFramework@server@thrift@apache@@UBE_JXZ)
1>libthrift.lib(TSimpleServer.obj) : error LNK2001: 无法解析的外部符号 "public: virtual __int64 __thiscall apache::thrift::server::TServerFramework::getConcurrentClientCount(void)const " (?getConcurrentClientCount@TServerFramework@server@thrift@apache@@UBE_JXZ)
1>libthrift.lib(TSimpleServer.obj) : error LNK2001: 无法解析的外部符号 "public: virtual __int64 __thiscall apache::thrift::server::TServerFramework::getConcurrentClientCountHWM(void)const " (?getConcurrentClientCountHWM@TServerFramework@server@thrift@apache@@UBE_JXZ)
1>libthrift.lib(TSimpleServer.obj) : error LNK2019: 无法解析的外部符号 "public: __thiscall apache::thrift::server::TServerFramework::TServerFramework(class boost::shared_ptr<class apache::thrift::TProcessor> const &,class boost::shared_ptr<class apache::thrift::transport::TServerTransport> const &,class boost::shared_ptr<class apache::thrift::transport::TTransportFactory> const &,class boost::shared_ptr<class apache::thrift::protocol::TProtocolFactory> const &)" (??0TServerFramework@server@thrift@apache@@QAE@ABV?$shared_ptr@VTProcessor@thrift@apache@@@boost@@ABV?$shared_ptr@VTServerTransport@transport@thrift@apache@@@5@ABV?$shared_ptr@VTTransportFactory@transport@thrift@apache@@@5@ABV?$shared_ptr@VTProtocolFactory@protocol@thrift@apache@@@5@@Z)，该符号在函数 "public: __thiscall apache::thrift::server::TSimpleServer::TSimpleServer(class boost::shared_ptr<class apache::thrift::TProcessor> const &,class boost::shared_ptr<class apache::thrift::transport::TServerTransport> const &,class boost::shared_ptr<class apache::thrift::transport::TTransportFactory> const &,class boost::shared_ptr<class apache::thrift::protocol::TProtocolFactory> const &)" (??0TSimpleServer@server@thrift@apache@@QAE@ABV?$shared_ptr@VTProcessor@thrift@apache@@@boost@@ABV?$shared_ptr@VTServerTransport@transport@thrift@apache@@@5@ABV?$shared_ptr@VTTransportFactory@transport@thrift@apache@@@5@ABV?$shared_ptr@VTProtocolFactory@protocol@thrift@apache@@@5@@Z) 中被引用
1>libthrift.lib(TSimpleServer.obj) : error LNK2019: 无法解析的外部符号 "public: __thiscall apache::thrift::server::TServerFramework::TServerFramework(class boost::shared_ptr<class apache::thrift::TProcessorFactory> const &,class boost::shared_ptr<class apache::thrift::transport::TServerTransport> const &,class boost::shared_ptr<class apache::thrift::transport::TTransportFactory> const &,class boost::shared_ptr<class apache::thrift::transport::TTransportFactory> const &,class boost::shared_ptr<class apache::thrift::protocol::TProtocolFactory> const &,class boost::shared_ptr<class apache::thrift::protocol::TProtocolFactory> const &)" (??0TServerFramework@server@thrift@apache@@QAE@ABV?$shared_ptr@VTProcessorFactory@thrift@apache@@@boost@@ABV?$shared_ptr@VTServerTransport@transport@thrift@apache@@@5@ABV?$shared_ptr@VTTransportFactory@transport@thrift@apache@@@5@2ABV?$shared_ptr@VTProtocolFactory@protocol@thrift@apache@@@5@3@Z)，该符号在函数 "public: __thiscall apache::thrift::server::TSimpleServer::TSimpleServer(class boost::shared_ptr<class apache::thrift::TProcessorFactory> const &,class boost::shared_ptr<class apache::thrift::transport::TServerTransport> const &,class boost::shared_ptr<class apache::thrift::transport::TTransportFactory> const &,class boost::shared_ptr<class apache::thrift::transport::TTransportFactory> const &,class boost::shared_ptr<class apache::thrift::protocol::TProtocolFactory> const &,class boost::shared_ptr<class apache::thrift::protocol::TProtocolFactory> const &)" (??0TSimpleServer@server@thrift@apache@@QAE@ABV?$shared_ptr@VTProcessorFactory@thrift@apache@@@boost@@ABV?$shared_ptr@VTServerTransport@transport@thrift@apache@@@5@ABV?$shared_ptr@VTTransportFactory@transport@thrift@apache@@@5@2ABV?$shared_ptr@VTProtocolFactory@protocol@thrift@apache@@@5@3@Z) 中被引用
1>libthrift.lib(TSimpleServer.obj) : error LNK2019: 无法解析的外部符号 "public: __thiscall apache::thrift::server::TServerFramework::TServerFramework(class boost::shared_ptr<class apache::thrift::TProcessor> const &,class boost::shared_ptr<class apache::thrift::transport::TServerTransport> const &,class boost::shared_ptr<class apache::thrift::transport::TTransportFactory> const &,class boost::shared_ptr<class apache::thrift::transport::TTransportFactory> const &,class boost::shared_ptr<class apache::thrift::protocol::TProtocolFactory> const &,class boost::shared_ptr<class apache::thrift::protocol::TProtocolFactory> const &)" (??0TServerFramework@server@thrift@apache@@QAE@ABV?$shared_ptr@VTProcessor@thrift@apache@@@boost@@ABV?$shared_ptr@VTServerTransport@transport@thrift@apache@@@5@ABV?$shared_ptr@VTTransportFactory@transport@thrift@apache@@@5@2ABV?$shared_ptr@VTProtocolFactory@protocol@thrift@apache@@@5@3@Z)，该符号在函数 "public: __thiscall apache::thrift::server::TSimpleServer::TSimpleServer(class boost::shared_ptr<class apache::thrift::TProcessor> const &,class boost::shared_ptr<class apache::thrift::transport::TServerTransport> const &,class boost::shared_ptr<class apache::thrift::transport::TTransportFactory> const &,class boost::shared_ptr<class apache::thrift::transport::TTransportFactory> const &,class boost::shared_ptr<class apache::thrift::protocol::TProtocolFactory> const &,class boost::shared_ptr<class apache::thrift::protocol::TProtocolFactory> const &)" (??0TSimpleServer@server@thrift@apache@@QAE@ABV?$shared_ptr@VTProcessor@thrift@apache@@@boost@@ABV?$shared_ptr@VTServerTransport@transport@thrift@apache@@@5@ABV?$shared_ptr@VTTransportFactory@transport@thrift@apache@@@5@2ABV?$shared_ptr@VTProtocolFactory@protocol@thrift@apache@@@5@3@Z) 中被引用
1>C:\Users\iscas\Desktop\thrift\thriftClientDemo\Debug\thriftServerDemo.exe : fatal error LNK1120: 11 个无法解析的外部命令
1>
1>生成失败。
```
目测为链接库的实现存在问题，导致函数无法正确的找到。
目前没有在网上找到相关的解决方法，准备尝试在linux系统下进行测试了验证是否为编译器或者平台问题。

#### linux环境下thrift源代码的编译：
由于在windows环境下测试失败，为了检查平台还是操作系统问题，现在尝试在centos7_x64环境下对thrift进行编译和测试。

##### 指定版本的boost编译：
参考文档：[Linux编译和安装boost库](http://blog.csdn.net/this_capslock/article/details/47170313)
按照上述教程，从[官方网站](https://sourceforge.net/projects/boost/files/boost/)下载源代码包，然后执行如下步骤：
```shell
tar zxvf boost_1_63_0.tar.gz
cd boost_1_63_0
./bootstrap.sh --with-libraries=all --with-toolset=gcc
```
命令执行完成后看到显示如下即为成功：
```shell
Building Boost.Build engine with toolset gcc... tools/build/src/engine/bin.linuxx86_64/b2
Detecting Python version... 2.7
Detecting Python root... /usr
Unicode/ICU support for Boost.Regex?... not found.
Generating Boost.Build configuration in project-config.jam...

Bootstrapping is done. To build, run:

    ./b2
    
To adjust configuration, edit 'project-config.jam'.
Further information:

   - Command line help:
     ./b2 --help
     
   - Getting started guide: 
     http://www.boost.org/more/getting_started/unix-variants.html
     
   - Boost.Build documentation:
     http://www.boost.org/build/doc/html/index.html
```
接下来，就需要使用生成的b2来对boost进行编译：
```shell
./b2 toolset=gcc
```
时间比较久，最终会显示类似：
```shell
ln-UNIX stage/lib/libboost_type_erasure.so
common.mkdir bin.v2/libs/wave/build/gcc-4.8.5/release/threading-multi
gcc.compile.c++ bin.v2/libs/wave/build/gcc-4.8.5/release/threading-multi/instantiate_cpp_exprgrammar.o
gcc.compile.c++ bin.v2/libs/wave/build/gcc-4.8.5/release/threading-multi/instantiate_cpp_grammar.o
gcc.compile.c++ bin.v2/libs/wave/build/gcc-4.8.5/release/threading-multi/instantiate_cpp_literalgrs.o
gcc.compile.c++ bin.v2/libs/wave/build/gcc-4.8.5/release/threading-multi/instantiate_defined_grammar.o
gcc.compile.c++ bin.v2/libs/wave/build/gcc-4.8.5/release/threading-multi/instantiate_predef_macros.o
gcc.compile.c++ bin.v2/libs/wave/build/gcc-4.8.5/release/threading-multi/instantiate_re2c_lexer.o
gcc.compile.c++ bin.v2/libs/wave/build/gcc-4.8.5/release/threading-multi/instantiate_re2c_lexer_str.o
gcc.compile.c++ bin.v2/libs/wave/build/gcc-4.8.5/release/threading-multi/token_ids.o
gcc.compile.c++ bin.v2/libs/wave/build/gcc-4.8.5/release/threading-multi/wave_config_constant.o
common.mkdir bin.v2/libs/wave/build/gcc-4.8.5/release/threading-multi/cpplexer
common.mkdir bin.v2/libs/wave/build/gcc-4.8.5/release/threading-multi/cpplexer/re2clex
gcc.compile.c++ bin.v2/libs/wave/build/gcc-4.8.5/release/threading-multi/cpplexer/re2clex/aq.o
gcc.compile.c++ bin.v2/libs/wave/build/gcc-4.8.5/release/threading-multi/cpplexer/re2clex/cpp_re.o
gcc.link.dll bin.v2/libs/wave/build/gcc-4.8.5/release/threading-multi/libboost_wave.so.1.63.0
common.copy stage/lib/libboost_wave.so.1.63.0
ln-UNIX stage/lib/libboost_wave.so
...failed updating 56 targets...
...skipped 6 targets...
...updated 1077 targets...
```
的信息，表示编译完成。

接下来就需要进行连接和安装了：
```shell
mkdir /home/wentao/boost_lib
./b2 install --prefix=/home/wentao/boost_lib
```
--prefix=/usr用来指定boost的安装目录，不加此参数的话默认的头文件在/usr/local/include/boost目录下，库文件在/usr/local/lib/目录下。这里把安装目录指定为--prefix=/home/wentao/boost_lib则boost会直接安装到这个文件夹下，可以忽略对环境变量的影响。


需要注意的是，如果boost检测到了系统安装了python，那么就会默认开启boost.python支持，这个时候需要系统安装python-devel，否则会无法找到头文件报相关的错误：
```shell
./boost/python/detail/wrap_python.hpp:50:23: fatal error: pyconfig.h: No such file or directory
```
也就是最后出现：
```shell
...failed updating 56 targets...
```
错误的原因，可以通过命令：
```shell
yum -y install python-devel
```
安装python的开发包支持，然后重新执行b2来进行编译，最终结果为：
```shell
common.copy stage/lib/libboost_python.so.1.63.0
ln-UNIX stage/lib/libboost_python.so
...updated 62 targets...


The Boost C++ Libraries were successfully built!

The following directory should be added to compiler include paths:

    /home/wentao/boost_1_63_0

The following directory should be added to linker library paths:

    /home/wentao/boost_1_63_0/stage/lib
```
参考文档：[boost.python编译及示例](http://blog.csdn.net/majianfei1023/article/details/46781581)

> - 如果不需要指定boost版本，可以直接使用yum来对boost进行安装：yum install boost boost-devel boost-doc

##### 指定版本的OpenSSL编译：
安装openssl需要zlib和perl支持，因为centos7默认自带了perl环境，所以这儿只需要添加zlib的开发包：
```shell
yum install zlib-devel
```
返回结果为：
```shell
Loaded plugins: fastestmirror
Loading mirror speeds from cached hostfile
 * base: mirrors.tuna.tsinghua.edu.cn
 * extras: mirrors.tuna.tsinghua.edu.cn
 * updates: mirrors.tuna.tsinghua.edu.cn
Resolving Dependencies
--> Running transaction check
---> Package zlib-devel.x86_64 0:1.2.7-17.el7 will be installed
--> Finished Dependency Resolution

Dependencies Resolved

=============================================================================================================================================================================================
 Package                                        Arch                                       Version                                            Repository                                Size
=============================================================================================================================================================================================
Installing:
 zlib-devel                                     x86_64                                     1.2.7-17.el7                                       base                                      50 k

Transaction Summary
=============================================================================================================================================================================================
Install  1 Package

Total download size: 50 k
Installed size: 132 k
Is this ok [y/d/N]: y
Downloading packages:
zlib-devel-1.2.7-17.el7.x86_64.rpm                                                                                                                                    |  50 kB  00:00:00     
Running transaction check
Running transaction test
Transaction test succeeded
Running transaction
  Installing : zlib-devel-1.2.7-17.el7.x86_64                                                                                                                                            1/1 
  Verifying  : zlib-devel-1.2.7-17.el7.x86_64                                                                                                                                            1/1 

Installed:
  zlib-devel.x86_64 0:1.2.7-17.el7                                                                                                                                                           

Complete!
```
然后就可以对OpenSSL进行编译安装了：
```shell
./config --prefix=/home/wentao/openssl_lib shared zlib-dynamic
```
这儿使用zlib-dynamic，是因为已经全局安装了zlib-devel，通过动态库可以获取支持，而不再需要zlib的源代码嵌入编译。
继续编译安装：
```shell
make
make install
```

> - 参考文档：
[编译安装openssl时使用参数zlib-dynamic和zlib有什么区别？](https://segmentfault.com/q/1010000008259139)
[Beyond Linux® From Scratch - Version 2017-02-15 Chapter 4. Security](http://www.linuxfromscratch.org/blfs/view/svn/postlfs/openssl.html)
[Linux之编译安装openssl来升级系统openssl](https://www.dwhd.org/20160811_122603.html)

##### 指定版本的libevent编译：
还是按照一般的库编译方式进行：
```shell
tar zxvf libevent-2.1.8-stable.tar.gz
cd libevent-2.1.8-stable
./configure --prefix=/home/wentao/libevent_lib
make
make install
```

##### 使用thrift默认configure进行编译：
上述如果不需要依赖于特定的库版本，可以直接通过一条命令准备好编译thrift的所有库：
```shell
sudo yum -y install boost-devel libevent-devel zlib-devel openssl-devel
```
然后解压缩thrift，并且进入lib/cpp文件夹下，准备编译C++的开发库支持：
```shell
tar zxvf thrift-0.10.0.tar.gz
cd thrift-0.10.0
```
然后按照[官方文档](https://thrift.apache.org/docs/BuildingFromSource)，对整个thrift进行配置和编译，包含thrift编译器和库支持。查看我们需要配置的内容：
```shell
./configure --help
```
具体的就需要设置上述我们自定义版本库路径设置：
```shell
  --with-boost[=ARG]      use Boost library from a standard location
                          (ARG=yes), from the specified location (ARG=<path>),
                          or disable it (ARG=no) [ARG=yes]
  --with-boost-libdir=LIB_DIR
                          Force given directory for boost libraries. Note that
                          this will override library path detection, so use
                          this parameter only if default library detection
                          fails and you know exactly where your boost
                          libraries are located.
  --with-openssl=DIR      root of the OpenSSL directory
  --with-libevent[=DIR]   use libevent [default=yes]. Optionally specify the
                          root prefix dir where libevent is installed
```
最后的配置脚本为：
```shell
./configure --prefix='/home/wentao/thrift_lib' --with-boost='/home/wentao/boost_lib/'  --with-openssl='/home/wentao/openssl_lib/' --with-libevent='/home/wentao/libevent_lib'
```
返回结果中就会包含C++的库支持选项为打开状态，表示会编译C++的开发库了。其他检查没有错误之后，就可以执行编译和安装了：
```shell
make
make install
```
在make完毕进行链接时由于找不到openssl相关的库导致生成失败，感觉这个是configure中设置的部分没有起作用，看看是否有官方解决方法。

##### 采用系统支持包，使用cmake对thrift进行编译：
上述configure中设置了相关的库路径后，还是会遇到ld找不到动态库的问题，导致链接失败而无法正常生成文件，所以还是采用linux下预先安装相关包完成依赖，然后使用cmake或者configure的方式进行编译。
在thrift解压缩目录下新建cmakebuild目录，实现源外编译：
```shell
mkdir cmakebuild
cd cmakebuild
ccmake ..
```
完成相关选项的检查和填写后，输入g生成makefile，然后开始编译：
```shell
make
make install
```

需要注意的是，因为thrift编译器运行需要加载libthrift相关的库才可以运行，所以需要将cmake中设置的install路径中的lib文件夹添加到ld查询路径中，编辑/etc/ld.so.conf，然后添加这个路径到其中，然后运行：
```shell
/sbin/ldconfig
```
刷新ld查询路径，就可以使用：./thrift -version，查询是否编译成功了。

如果要对C支持，需要安装glibc的开发包：
```shell
yum install glibc glibc-devel glibc-headers
```
但是目前自己测试的系统中已经安装了这些包，但是还是没有生成C相关的开发库文件。

> - 参考文档：[Apache Thrift - CMake build](https://github.com/apache/thrift/tree/master/build/cmake)

##### 单独对thrift的C++库支持进行编译：
需要安装cmake来对thrift的库依赖进行配置：
```shell
yum -y install cmake
```
然后使用cmake进行配置，新建一个build目录放置cmake生成的中间文件，然后检查编译需要的依赖：
```shell
mkdir /home/wentao/thrift_build
cd thrift_build
ccmake /home/wentao/thrift-0.10.0/lib/cpp/CMakeList.txt
```
然后使用t打开高级选项，填入需要依赖的库路径，完成后输入c保存。
回显：
```shell
-- The C compiler identification is GNU 4.8.5
-- The CXX compiler identification is GNU 4.8.5
-- Check for working C compiler: /usr/bin/cc
-- Check for working C compiler: /usr/bin/cc -- works
-- Detecting C compiler ABI info
-- Detecting C compiler ABI info - done
-- Check for working CXX compiler: /usr/bin/c++
-- Check for working CXX compiler: /usr/bin/c++ -- works
-- Detecting CXX compiler ABI info
-- Detecting CXX compiler ABI info - done
CMake Error at CMakeLists.txt:20 (include_directories):
  include_directories given empty-string as include directory.


-- Could NOT find OpenSSL, try to set the path to OpenSSL root folder in the system variable OPENSSL_ROOT_DIR (missing:  OPENSSL_LIBRARIES OPENSSL_INCLUDE_DIR) 
CMake Error at CMakeLists.txt:162 (include):
  include could not find load file:

    ThriftMacros


CMake Error at CMakeLists.txt:164 (ADD_LIBRARY_THRIFT):
  Unknown CMake command "ADD_LIBRARY_THRIFT".


CMake Warning (dev) in CMakeLists.txt:
  No cmake_minimum_required command is present.  A line of code such as

    cmake_minimum_required(VERSION 2.8)

  should be added at the top of the file.  The version specified may be lower
  if you wish to support older CMake versions for this project.  For more
  information run "cmake --help-policy CMP0000".
This warning is for project developers.  Use -Wno-dev to suppress it.

-- Configuring incomplete, errors occurred!
See also "/home/wentao/thrift-0.10.0/lib/cpp/CMakeFiles/CMakeOutput.log".
```
根据提示，我们需要在命令行中指定需要依赖的文件路径：
```shell
set OPENSSL_LIBRARIES='/home/wentao/openssl_lib/lib/' 
set OPENSSL_INCLUDE_DIR='/home/wentao/openssl_lib/include/'
cmake ../CMakeLists.txt -G 'Unix Makefiles'
```
还是失败，不知道有什么解决方法。

#### linux下的C++服务端编写测试：
还是使用和windows一样的服务端代码，但是为了方便起见，采用cmake来管理工程，方便在各种平台上的测试切换。
首先编写cmake文件来对thrift服务端源代码进行管理：
```cmake

```



## 使用thrift完成其他语言之间的相互调用：
既然thrift可以提供多种语言的RPC通信过程，除了C++，再尝试使用脚本类语言作为通信对象。

> - 参考文档：
[使用thrift做c++,java和python的相互调用](http://jinghong.iteye.com/blog/1222713)
[利用thrift实现js与C#通讯的例子](http://www.cnblogs.com/xxxteam/archive/2013/04/15/3023159.html)

## thrift基本语法学习：


## 使用thrift做JDBC开发：
就自己的开发经验来看，使用thrift来对JDBC开发还是只是替换了socket通信的部分，而没有对整个JDBC的框架开发带来其他变化。也就是说将当前自己编写的socket服务器通过thrift来完成替换，其他部分均保持不变。


## 与其他PRC框架对比：
[](http://colobu.com/2016/09/05/benchmarks-of-popular-rpc-frameworks/)
[RPC框架性能基本比较测试](http://www.useopen.net/blog/2015/rpc-performance.html)
[开源RPC（gRPC/Thrift）框架性能评测](http://www.eit.name/blog/read.php?566)

## 源代码分析：
[Thrift源码剖析](http://yanyiwu.com/work/2014/10/17/thrift-source-code-illustration.html)
[Thrift源码分析](https://www.kancloud.cn/digest/thrift/118984)