---
title: Consul服务注册
date: 2023-11-09 09:26:07
tags:
categories:
- Go微服务组件
description: 详情点击阅读全文
---

# 配置文件
```yaml
register:
  name: grpc.app.server # 服务名
  grpc_port: 8080 # grpc端口，不可以为0
  http_port: 80 # http端口，0表示不开启http
  addr: 127.0.0.1:8500 # 注册中心地址
  timeout: 3 # 超时控制
  tags: # 服务下的service
    - user
  check: # 健康检查
    timeout: 5s # 超时
    interval: 5s # 检查间隔
    deregister_critical_service_after: 10s # 检查失败后10s取消注册
```

# GRPC 服务注册
## 生成 GRPC 服务注册对象
```go
var (
	registerClient *api.Client
	serviceID      string
)

// serviceRegister 服务注册
func serviceRegister(cfg *configs.Config) {
	rgsConfig := cfg.Register

	var err error
	serviceID = uuid.New().String()
	defaultCfg := api.DefaultConfig()
	defaultCfg.Address = rgsConfig.Addr

	if registerClient, err = api.NewClient(defaultCfg); err != nil {
		tools.Log.Error(ctx, "[serviceRegister] NewClient err = %v", err)
	}

	// 生成注册对象
	srvAddr := tools.GetOwnIP() + ":" + rgsConfig.GrpcPort
	port, _ := strconv.Atoi(rgsConfig.GrpcPort)
	register := api.AgentServiceRegistration{
		ID:      serviceID,
		Address: srvAddr,
		Port:    port,
		Name:    rgsConfig.Name,
		Tags:    rgsConfig.Tags,
		Check: &api.AgentServiceCheck{ // 健康检查对象
			GRPC:                           srvAddr,
			Timeout:                        rgsConfig.Check.Timeout,                        // 超时
			Interval:                       rgsConfig.Check.Interval,                       // 检查间隔
			DeregisterCriticalServiceAfter: rgsConfig.Check.DeregisterCriticalServiceAfter, //注册失败10s后取消注册
		},
	}

	// 开始注册
	if err := registerClient.Agent().ServiceRegister(&register); err != nil {
		tools.Log.Error(ctx, "[ServiceRegister] 服务注册 err = %v", err)
	}
}
```

## GRPC 需注册健康检测服务
```go
// "google.golang.org/grpc/health"
// "google.golang.org/grpc/health/grpc_health_v1"
grpc_health_v1.RegisterHealthServer(server, health.NewServer())
```

# HTTP 服务注册
## 生成 HTTP 服务注册对象
```go
// ConsulRegister 服务注册
func ConsulRegister() {
	conn()
	consulConfig := global.CONFIG.Consul

	// 生成注册对象
	register := new(api.AgentServiceRegistration)
	register.ID = serviceID
	register.Address = utils.Ip
	register.Port = utils.Port
	register.Name = consulConfig.Name
	register.Tags = consulConfig.Tags

	// 生成对应的检查对象
	register.Check = &api.AgentServiceCheck{
		HTTP:                           "http://" + utils.GetLocalIp() + ":8080/health",
		Timeout:                        "5s",  // 超时
		Interval:                       "5s",  // 检查间隔
		DeregisterCriticalServiceAfter: "10s", //注册失败10s后取消注册
	}

	// 开始注册
	err := consulClient.Agent().ServiceRegister(register)
	if err != nil {
		log.Fatalf("服务注册失败 %s", err.Error())
	}
}
```

## HTTP 需注册 /health 进行响应
```go
r.GET("/health", response.Ok)
```

# consul 其他方法
```go
// serviceDeregister 服务注销
func serviceDeregister() {
	if registerClient == nil {
		return
	}
	if err := registerClient.Agent().ServiceDeregister(serviceID); err != nil {
		tools.Log.Error(ctx, "[ServiceDeregister] 服务注销 err = %v", err)
	}
}

// Services 全部服务
func Services() (map[string]*api.AgentService, error) {
	return registerClient.Agent().Services()
}

// ServicesWithFilter 服务过滤
func ServicesWithFilter(serviceName string) (map[string]*api.AgentService, error) {
	return registerClient.Agent().ServicesWithFilter("Service == " + serviceName)
}

// ServiceTarget 服务地址
func ServiceTarget(cfg *configs.Config, serviceName string) string {
	return "consul://" + cfg.Register.Addr + "/" + serviceName
}
```