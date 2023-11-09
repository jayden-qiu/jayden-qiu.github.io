---
title: Viper配置读取
date: 2023-11-09 10:47:03
tags:
categories:
- Go微服务组件
description: 详情点击阅读全文
---

# 使用
```go
// ReadOnlineConfig 读取业务配置
func ReadOnlineConfig() (cfg *configs.Config) {

	cfg = &configs.Config{}

	v := viper.New()

	v.SetConfigFile("config.yaml")

	if err := v.ReadInConfig(); err != nil {
		tools.Log.Error(ctx, "[readOnlineConfig] ReadInConfig err = %v", err)
		return
	}

	if err := v.Unmarshal(cfg); err != nil {
		tools.Log.Error(ctx, "[readOnlineConfig] Unmarshal err = %v", err)
	}
	return
}
```