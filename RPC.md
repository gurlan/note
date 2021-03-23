





# RPC协议

## RPC实用步骤

> 服务端

1.注册rpc服务对象，给对象绑定方法

```go
rpc.RegisterName("服务名","回调对象")
```

2.创建监听器

```go
listener,err := net.Listen()
```

3.建立连接

```go
conn,err := listener.Accept()
```

4.将连接绑定到rpc服务

```go
rpc.ServeConn(conn)
```

> 客户端

1.用rpc连接服务端

```go
conn,err :=rpc.Dial()
```

2.调用远程函数

```go
conn.Call("服务名.方法名"，传入参数，&传出参数)
```

## RPC相关函数

1.注册rpc服务

```go
/**
name：serviceName 
rcvr: 对应的rpc对象 
       方法必须是导出的 -- 包对外可见，首字母大写
       方法必须有两个参数，都是导出类型、内建类型
       方法的第二个参数必须是指针
       方法只有一个error接口类型的返回值
*/ 
func (server *Server) RegisterName(name string, rcvr interface{}) error
```

2.绑定rpc服务

```go
func (server *Server) ServeConn(conn io.ReadWriteCloser)
```

3.调用远程函数

```go
func (client *Client) Call(serviceMethod string, args interface{}, reply interface{}) error
```

## 简单实现

server.go

```go
package main

import (
	"fmt"
	"net"
	"net/rpc"
)

func main() {
	// 注册rpc服务
	err := rpc.RegisterName("world", new(World))
	if err != nil {
		fmt.Println("服务注册失败")
		return
	}
	// 设置监听
	listener, err := net.Listen("tcp", "127.0.0.1:8081")
	if err != nil {
		fmt.Println("net.Listen error:", err)
		return
	}
	fmt.Println("开始监听")
	defer listener.Close()
	// 建立连接
	conn, err := listener.Accept()
	if err != nil {
		fmt.Println("net.Accept error:", err)
		return
	}
	defer conn.Close()
	fmt.Println("连接建立成功")
	// 绑定服务
	//rpc.ServeConn(conn)
    jsonrpc.ServeConn(conn)
}


type World struct {

}

func (this *World) HelloWorld(name string, resp *string) error {
	*resp = name+"你好"
	return nil
}

```

client.go

```go
package main

import (
	"fmt"
	"net/rpc"
)

func main() {
	// 连接服务器
	// conn, err := rpc.Dial("tcp", "127.0.0.1:8081")
	conn, err := jsonrpc.Dial("tcp", "111.230.180.43:8090")
	if err != nil {
		fmt.Println("连接失败")
		return
	}
	defer conn.Close()
	// 调用远程方法
	resp := ""
	err = conn.Call("world.HelloWorld", "gaojian", &resp)
	if err != nil {
		fmt.Println("服务调用失败")
		return
	}
	fmt.Println(resp)
}

```

json 版RPC

- 实用nc -l ip port 充当服务器

- client.go充当客户端。发起通信，会产生乱码

  因为：go rpc使用了go语言特有的序列化gob，其他语言不能解析

- 使用通用的序列化

## RPC封装

### 服务端封装

```go
// 定义接口
type MyInterface struct{
     HelloWorld (string,*string)error
}
// 封装注册服务方法
func RegisterService (i MyInterface){
    rpc.RegisterName("hellow",i)
}
```

### 客户端封装

```go
// 定义类
type MyClient struct{
    c *rpc.Client
}
// 初始化客户端
func  InitClient(addr string)error{
    conn,_ :=jsonrpc.Dial("tcp",addr)
    return MyClient{c:conn}
}
// 绑定类方法
func (this *MyClient) HelloWorld(a string,b *string) error
{
    return this.c.Call("hello.HelloWorld",a,b)  
}
```

# protobuf

## 安装

### win

> 安装protoc

  下载地址： https://github.com/protocolbuffers/protobuf/releases 

​	解压到文件夹，配置环境变量

> 安装protoc-gen-go

- 在终端直接执行 `go get -u github.com/golang/protobuf/protoc-gen-go`，可以在你的%GOPATH%/bin路径下找到一个 protoc-gen-go.exe

###    linux

## 编译	

common.proto

```protobuf
syntax = "proto3";

// 指定所在包名
package pb;
// 不指定该选项没有编译过 不知道为啥
option go_package = "test/";
// 定义消息体
message Response {
    // 状态编码
    int32 code = 1;
    // 状态编码具体错误描述
    string message = 2;
    // 返回数据
    bytes result = 3;
}

```



