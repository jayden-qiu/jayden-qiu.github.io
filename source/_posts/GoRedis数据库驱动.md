---
title: GoRedis数据库驱动
date: 2023-11-09 10:17:31
tags:
categories:
- Go微服务组件
description: 详情点击阅读全文
---

# 配置
```yaml
redis:
  db: 0
  addr: 127.0.0.1:6379
  password:
```

# 初始化
```go
// "github.com/go-redis/redis/v8"

// RDB redis驱动
var RDB *redis.Client

// initRedis 初始化redis
func initRedis(cfg *configs.Config) {
	RDB = redis.NewClient(&redis.Options{
		Addr:     cfg.Redis.Addr,
		Password: cfg.Redis.Password,
		DB:       cfg.Redis.DB,
	})

	if err := RDB.Ping(context.Background()).Err(); err != nil {
		tools.Log.Error(ctx, "[initRedis] err = %v", err)
	}
}
```
