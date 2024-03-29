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

​    解压到文件夹，配置环境变量

> 安装protoc-gen-go

- 在终端直接执行 `go get -u github.com/golang/protobuf/protoc-gen-go`，可以在你的%GOPATH%/bin路径下找到一个 protoc-gen-go.exe

### linux

#### 编译

common.proto

```protobuf
syntax = "proto3";

// 指定所在包名
package pb;
// 前面是生成代码位置;后是别名
option go_package = "test/;test";
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

##### 添加RPC服务

> 语法

```protobuf
service 服务名 {
    rpc 函数名（参数：消息体） returns (返回值：消息)
}
```

> 例子

```protobuf
message People {
    string name = 1;
}
message Student {
    string age = 2;
}
service hellow {
    rpc HelloWorld(People) returns(Student);
}
```

> 知识点

默认，protobuf编译期间，不编译服务。要想编译，要用gRPC

使用的编译命令为：

`protoc --go-grpc_out=./ --go_out=./  *.proto  ` 

## GRPC

### 安装

` go get -u -v google.golang.org/grpc`

### 使用

> 定义proto文件

```protobuf
syntax = "proto3";
option go_package="./;product";

service ProductInfo {
  //添加商品
  rpc addProduct(Product) returns (ProductId);
  //获取商品
  rpc getProduct(ProductId) returns (Product);
}

message Product {
  string id = 1;
  string name = 2;
  string description = 3;
}

message ProductId {
  string value = 1;
}
```

> 编译

`protoc --go-grpc_out=./ --go_out=./ *.proto `

> 服务端

```go
package main

import (
	"context"
	"github.com/gofrs/uuid"
	"google.golang.org/grpc"
	"google.golang.org/grpc/codes"
	"google.golang.org/grpc/status"
	"log"
	"net"
	"test/product"
)

type server struct {
	productMap map[string]*product.Product
	product.UnimplementedProductInfoServer
}

//添加商品
func (s *server) AddProduct(ctx context.Context, req *product.Product) (resp *product.ProductId, err error) {
	resp = &product.ProductId{}
	out, err := uuid.NewV4()
	if err != nil {
		return resp, status.Errorf(codes.Internal, "err while generate the uuid ", err)
	}

	req.Id = out.String()
	if s.productMap == nil {
		s.productMap = make(map[string]*product.Product)
	}

	s.productMap[req.Id] = req
	resp.Value = req.Id
	return
}

//获取商品
func (s *server) GetProduct(ctx context.Context, req *product.ProductId) (resp *product.Product, err error) {
	if s.productMap == nil {
		s.productMap = make(map[string]*product.Product)
	}

	resp = s.productMap[req.Value]
	return
}

func main() {
	listener, err := net.Listen("tcp", ":8890")
	if err != nil {
		log.Println("net listen err ", err)
		return
	}

	s := grpc.NewServer()
	product.RegisterProductInfoServer(s, &server{})

	if err := s.Serve(listener); err != nil {
		log.Println("failed to serve...", err)
		return
	}
}

```

> 客户端

```go
package main

import (
	"context"
	"google.golang.org/grpc"
	"log"
	"test/product"
)

const (
	address = "localhost:8890"
)

func main() {
	conn, err := grpc.Dial(address, grpc.WithInsecure())
	if err != nil {
		log.Println("did not connect.", err)
		return
	}
	defer conn.Close()

	client := product.NewProductInfoClient(conn)
	ctx := context.Background()

	id := AddProduct(ctx, client)
	GetProduct(ctx, client, id)
	select {}
}

// 添加一个测试的商品
func AddProduct(ctx context.Context, client product.ProductInfoClient) (id string) {
	aMac := &product.Product{Name: "Mac Book Pro 2019", Description: "From Apple Inc."}
	productId, err := client.AddProduct(ctx, aMac)
	if err != nil {
		log.Println("add product fail.", err)
		return
	}
	log.Println("add product success, id = ", productId.Value)
	return productId.Value
}

// 获取一个商品
func GetProduct(ctx context.Context, client product.ProductInfoClient, id string) {
	p, err := client.GetProduct(ctx, &product.ProductId{Value: id})
	if err != nil {
		log.Println("get product err.", err)
		return
	}
	log.Printf("get prodcut success : %+v\n", p)
}

```