---
title: Jaeger链路追踪
date: 2023-11-09 10:00:53
tags:
categories:
- Go微服务组件
description: 详情点击阅读全文
---

# 配置
```yaml
tracing_url: http://127.0.0.1:14268/api/traces
```

# 初始化
```go
// "go.opentelemetry.io/otel"
// "go.opentelemetry.io/otel/attribute"
// "go.opentelemetry.io/otel/exporters/jaeger"
// "go.opentelemetry.io/otel/sdk/resource"
// tracesdk "go.opentelemetry.io/otel/sdk/trace"
// semconv "go.opentelemetry.io/otel/semconv/v1.17.0"

var tp *tracesdk.TracerProvider

// initTracing 链路追踪初始化
func initTracing(cfg *configs.Config) {
	var err error
	tp, err = tracerProvider(cfg)
	if err != nil {
		tools.Log.Error(ctx, "[initTracing] tracerProvider err = %v", err)
		return
	}
	// Register our TracerProvider as the global so any imported
	// instrumentation in the future will default to using it.
	otel.SetTracerProvider(tp)
}

// tracerProvider returns an OpenTelemetry TracerProvider configured to use
// the Jaeger exporter that will send spans to the provided url. The returned
// TracerProvider will also use a Resource configured with all the information
// about the application.
func tracerProvider(cfg *configs.Config) (*tracesdk.TracerProvider, error) {
	// Create the Jaeger exporter
	exp, err := jaeger.New(jaeger.WithCollectorEndpoint(jaeger.WithEndpoint(cfg.TracingUrl)))
	if err != nil {
		return nil, err
	}

	srvAddr := tools.GetOwnIP() + ":" + cfg.Register.GrpcPort

	TracerProvider := tracesdk.NewTracerProvider(
		// Always be sure to batch in production.
		tracesdk.WithBatcher(exp),
		// Record information about this application in a Resource.
		tracesdk.WithResource(resource.NewWithAttributes(
			semconv.SchemaURL,
			semconv.ServiceName(cfg.Register.Name),
			attribute.String("namespace", cfg.Namespace),
			attribute.String("address", srvAddr),
		)),
	)

	return TracerProvider, nil
}
```

# 使用
## server 初始化
```go
func NewUserSrvServer() userPb.UserSrvServer {
	return &usrv{
		tracer: otel.Tracer("UserSrvServer"),
	}
}
```

## 正常使用
```go
	ctx, span := u.tracer.Start(ctx, "GetUserInfo")
	defer span.End()
```

## 可以设置一些key 信息
```go
	_, span := tracer.Start(ctx, "getUserInfoName")
	span.SetAttributes(attribute.Key("testset").String("value"))
	defer span.End()
```

# 退出链路追踪
```go
func exitTracing() {
	if tp == nil {
		return
	}
	if err := tp.Shutdown(context.Background()); err != nil {
		tools.Log.Error(ctx, "[exitTracing] Shutdown err = %v", err)
	}
}
```
