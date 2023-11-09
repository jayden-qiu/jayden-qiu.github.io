---
title: Go使用GRPC
date: 2023-11-09 10:58:06
tags:
categories:
- GRPC
description: 详情点击阅读全文
---

# 安装工具
```shell
	go install google.golang.org/protobuf/cmd/protoc-gen-go@latest
# 	protoc --go_out=. proto/*.proto

	go install github.com/envoyproxy/protoc-gen-validate@latest
# 	protoc --go_out=. --validate_out="lang=go:."  proto/*.proto

	go install google.golang.org/grpc/cmd/protoc-gen-go-grpc@latest
# 	protoc --go-grpc_out=. proto/*.proto

	go install github.com/favadi/protoc-go-inject-tag@latest
# 	protoc-go-inject-tag -input=./protocol/example/*.pb.go

	go install github.com/golang/mock/mockgen@latest
# 	mockgen -source=db.go -destination=db_mock.go -package=main

	go install github.com/grpc-ecosystem/grpc-gateway/v2/protoc-gen-grpc-gateway@latest
# 	protoc --grpc-gateway_out=. hello.proto

	go install github.com/grpc-ecosystem/grpc-gateway/v2/protoc-gen-openapiv2@latest
# 	protoc --openapiv2_out=. hello.proto
```

# 编写proto
```proto
syntax = "proto3";

package example;

import "proto/common/validate.proto";

option go_package = "./protocol/user;userPb";

service UserSrv {
  // 获取用户信息
  rpc GetUserInfo(GetUserInfoRequest) returns (GetUserInfoReply);
}

message GetUserInfoRequest {
  // @gotags: gorm:"-"
  int64 user_id = 1 [(validate.rules).int64.gt = 0];
}

message GetUserInfoReply {
  string name = 1;
}
```

# 生成代码
```shell
	protoc --go_out=. --validate_out="lang=go:." --go-grpc_out=. --grpc-gateway_out=. --grpc-gateway_opt generate_unbound_methods=true --openapiv2_out=. --openapiv2_opt generate_unbound_methods=true proto/user.proto
	protoc-go-inject-tag -input=./protocol/user/*.pb.go
```

# GRPC 服务端
```go
// GrpcRun
func GrpcRun(cfg *configs.Config) {
	config := cfg.Register

	// 证书认证
	// creds, err := credentials.NewServerTLSFromFile("server.crt", "server.key")
	// if err != nil {
	//     log.Fatal(err)
	// }

	// 注册拦截器
	opt := []grpc.ServerOption{
		grpc.UnaryInterceptor(tools.ValidateInterceptor),
	}

	server := grpc.NewServer(opt...)

	// 注册service

	grpc_health_v1.RegisterHealthServer(server, health.NewServer()) // 健康检查

	userPb.RegisterUserSrvServer(server, logic.NewUserSrvServer()) // 用户服务

	reflection.Register(server) // grpccurl调试

	// 监听端口
	listener, err := net.Listen("tcp", ":"+config.GrpcPort)
	if err != nil {
		log.Fatalf("[GrpcRun] Listen err = %v", err)
	}

	tools.Log.Info(ctx, "launch grpc %s success at %s", config.Name, "grpc://"+tools.GetOwnIP()+":"+config.GrpcPort)

	// 开启服务
	if err = server.Serve(listener); err != nil {
		log.Fatalf("[GrpcRun] Serve err: %v", err)
	}
}
```

# GRPC 客户端，ip方式寻址
```go
// GetUserSrvClientByIp
func GetUserSrvClientByIp() {

	opts := []grpc.DialOption{
		grpc.WithTransportCredentials(insecure.NewCredentials()),
		// grpc.WithBlock(), // 阻塞
	}

	if conn, err := grpc.Dial("127.0.0.1:50051", opts...); err != nil {
		tools.Log.Error(ctx, "[greeterByIp] Dial err = %v", err)
	} else {
		UserSrvClient = userPb.NewUserSrvClient(conn)
	}

	// defer func(conn *grpc.ClientConn) {
	// 	err := conn.Close()
	// 	if err != nil {
	// 		panic(err)
	// 	}
	// }(client)
}
```

# GRPC 客户端，consul方式寻址
```go
// GetUserSrvClientByConsul
func GetUserSrvClientByConsul() {
	opts := []grpc.DialOption{
		grpc.WithTransportCredentials(insecure.NewCredentials()),
		grpc.WithDefaultServiceConfig(`{"loadBalancingPolicy":"round_robin"}`), // 负载均衡
	}

	if conn, err := grpc.Dial("consul://127.0.0.1:8500/grpc.app.server.service", opts...); err != nil {
		tools.Log.Error(ctx, "[greeter] Dial err = %v", err)
	} else {
		UserSrvClient = userPb.NewUserSrvClient(conn)
	}
}
```

# 客户端请求示例
```go
req := examplePb.HelloRequest{
	Name: "看看请求",
}
rsp, err := GreeterClient.SayHello(context.Background(), &req)
if err != nil {
	t.Errorf("could not greet: %v", err)
}
t.Errorf("%v", rsp)
```

# GRPC Gateway 方式，开启 HTTP 服务
```go
// GatewayRun
func GatewayRun(cfg *configs.Config) {

	ctx := context.Background()

	mux := runtime.NewServeMux()

	endpoint := ":" + cfg.Register.GrpcPort

	opts := []grpc.DialOption{
		grpc.WithTransportCredentials(insecure.NewCredentials()),
	}

	if err := userPb.RegisterUserSrvHandlerFromEndpoint(ctx, mux, endpoint, opts); err != nil {
		tools.Log.Error(ctx, "[GatewayRun] RegisterUserSrvHandlerFromEndpoint err = %v", err)
		return
	}

	tools.Log.Info(ctx, "launch http %s success at %s", cfg.Register.Name, "http://"+tools.GetOwnIP()+":"+cfg.Register.HttpPort)

	if err := http.ListenAndServe(":"+cfg.Register.HttpPort, mux); err != nil {
		tools.Log.Error(ctx, "[GatewayRun] ListenAndServe err = %v", err)
	}
}
```
