##Pulsar安装部署
1. pulsar官网:http://pulsar.apache.org/zh-CN/;最新稳定版本:2.2.1;下载地址:http://mirror.cc.columbia.edu/pub/software/apache/pulsar/pulsar-2.2.1/apache-pulsar-2.2.1-bin.tar.gz
2. 解压进入conf文件夹下，并使用vi或vim打开文件client.conf，里面的webServiceUrl和brokerServiceUrl字段中对应的IP为服务器IP，可自行修改，默认localhost
3. 验证pulsar功能参考:https://www.cnblogs.com/leejack/p/9544360.html
##Go语言环境部署
1. 下载二进制包：go1.4.linux-amd64.tar.gz。下载地址:https://dl.google.com/go/go1.11.5.linux-amd64.tar.gz

2. 将下载的二进制包解压至 /usr/local目录。
`tar -C /usr/local -xzf go1.4.linux-amd64.tar.gz`
3. 将 /usr/local/go/bin 目录添加至PATH环境变量：
`export PATH=$PATH:/usr/local/go/bin`
4. 验证： 终端执行`go version`

##pulsar C++客户端环境部署 
>*  在安装Go库之前必须要确保已经成功安装了C++客户端库
1. 下载两个deb安装包
client: http://mirror.olnevhost.net/pub/apache/pulsar/pulsar-2.2.1/DEB/apache-pulsar-client.deb
client-devel: http://mirrors.ocf.berkeley.edu/apache/pulsar/pulsar-2.2.1/DEB/apache-pulsar-client-dev.deb
2. 使用dpkg安装下载的两个deb包
`sudo dpkg -i xxxx.deb`
##Pulsar Go客户端环境部署
>* 能访问goole的用户参考官网说明使用go get方式下载部署,国内普通用户使用以下方法
1. 去github下载pulsar2.2版本的压缩包 https://github.com/apache/pulsar/tree/branch-2.2
2. 解压压缩包，将pulsar-client-go文件夹移动到$GOPATH/src/gihub.com/apache/pulsar中
3. 使用时引入 
```go
import (
    "github.com/apache/pulsar/pulsar-client-go/pulsar"
)
```
4. 验证是否成功
>* go run 操作前确保已经启动pulsar单机服务
>* 新建test_pulsar.go，将以下代码复制粘贴到文件中
```go
package main

import (
    "fmt"
    "runtime"
    "context"
    "github.com/apache/pulsar/pulsar-client-go/pulsar"
    "log"
)

func main (){
    fmt.Println("Pulsar Producer")

    ctx := context.Background()

    //实例化Pulsar client
    client,err := pulsar.NewClient(pulsar.ClientOptions{
        URL:"pulsar://127.0.0.1:6650",  //xx.xx.xx.xx代表Pulsar IP
        OperationTimeoutSeconds:5,
        MessageListenerThreads:runtime.NumCPU()/2,
    })

    if err !=  nil {
        log.Fatalf("Could not instantiate Pulsar client:%v",err)
    }


    // 创建producer
    producer,err := client.CreateProducer(pulsar.ProducerOptions{
        Topic:"my-topic",
    })

    if err != nil {
        log.Fatalf("Could not instantiate Pulsar producer:%v",err)
    }

    defer producer.Close()

    msg := pulsar.ProducerMessage{
        Payload:[]byte("Hello,This is a message from Pulsar Producer!"),
    }

    if err := producer.Send(ctx,msg);err != nil {
        log.Fatalf("Producer could not send message:%v",err)
    }

}
```
>* 执行`go run tets_pulsar.go` 输出如下:
```js
Pulsar Producer
2019/02/18 16:41:19 INFO  | ConnectionPool:63 | Created connection for pulsar://127.0.0.1:6650
2019/02/18 16:41:19 INFO  | ClientConnection:285 | [127.0.0.1:45812 -> 127.0.0.1:6650] Connected to broker
2019/02/18 16:41:19 INFO  | HandlerBase:53 | [persistent://public/default/my-topic, ] Getting connection from pool
2019/02/18 16:41:19 INFO  | ConnectionPool:63 | Created connection for pulsar://yngee-VirtualBox:6650
2019/02/18 16:41:19 INFO  | ClientConnection:287 | [127.0.0.1:45814 -> 127.0.0.1:6650] Connected to broker through proxy. Logical broker: pulsar://yngee-VirtualBox:6650
2019/02/18 16:41:19 INFO  | ProducerImpl:155 | [persistent://public/default/my-topic, ] Created producer on broker [127.0.0.1:45814 -> 127.0.0.1:6650] 
2019/02/18 16:41:19 INFO  | ProducerImpl:467 | [persistent://public/default/my-topic, standalone-1-2] Closed producer
```
##可能遇到的问题
* 关于golang/x下的依赖缺少问题
  * `git clone https://github.com/golang/net.git $GOPATH/src/github.com/golang/net`
  * `git clone https://github.com/golang/sys.git $GOPATH/src/github.com/golang/sys`
  * `git clone https://github.com/golang/tools.git $GOPATH/src/github.com/golang/tools`
  * `ln -s $GOPATH/src/github.com/golang $GOPATH/src/golang.org/x`
* 版本问题导致提示C代码缺少头文件
  * 确保$GOPATH/src/gihub.com/apache/pulsar/pulsar-client-go 是branch2.2分支下的
##参考文章
[Apache Pulsar单机环境及Go语言开发环境搭建](https://www.cnblogs.com/leejack/p/9544360.html) 
[解决golang.org/x包无法下载的问题](https://blog.csdn.net/kangyunqiang/article/details/78396866)
[pulsar项目官方github issues/3094](https://github.com/apache/pulsar/issues/3094)
[官方文档](http://pulsar.apache.org/zh-CN/)