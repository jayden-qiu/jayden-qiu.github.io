---
title: Elastic数据库驱动
date: 2023-11-09 10:44:25
tags:
categories:
- Go微服务组件
description: 详情点击阅读全文
---

# 配置
```yaml
elastic:
  addresses:
    - http://localhost:9200
  username:
  password:
```

# 初始化
```go
// elasticsearch7 "github.com/elastic/go-elasticsearch/v7"

// ES es7客户端
var ES *elasticsearch7.Client

// initES 初始化es
func initES(cfg *configs.Config) {
	defer wg.Done()
	if cfg == nil || len(cfg.Elastic.Addresses) == 0 {
		return
	}

	var err error
	if ES, err = elasticsearch7.NewClient(elasticsearch7.Config{
		Addresses: cfg.Elastic.Addresses,
		Username:  cfg.Elastic.Username,
		Password:  cfg.Elastic.Password,
	}); err != nil {
		tools.Log.Error(ctx, "[initES] err = %v", err)
	}
}
```
