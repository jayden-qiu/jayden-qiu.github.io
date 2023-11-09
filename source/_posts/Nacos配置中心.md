---
title: Nacos配置中心
date: 2023-11-09 09:44:33
tags:
categories:
- Go微服务组件
description: 详情点击阅读全文
---

# 配置文件
```yaml
# dev formal
namespace: dev
server-config:
  context-path: /nacos
  ip-addr: 127.0.0.1
  port: 8849
  scheme: http

client-config:
  namespace-id: 5bb8c24e-f263-40bd-bf8b-71f75272f314
  timeout-ms: 1000
  not-load-cache-at-start: false
  log-dir: ./tmp
  cache-dir: ./tmp
  log-level: warn

config-param:
  data-id: config
  group: "dev" # 用 namespace 做 group

```

# 连接配置中心，获取配置文件
```go 
var configClient config_client.IConfigClient
var configCenterUrl string

// DownOnlineConfig
func DownOnlineConfig(localCfg *configs.ConfigCenter) {

	if localCfg == nil || localCfg.Namespace == "" {
		return
	}

	// 配置中心信息
	sConfig := localCfg.ServerConfig

	// ping失败就退出
	configCenterUrl = sConfig.IpAddr
	if !tools.PingUrl(configCenterUrl) {
		tools.Log.Error(ctx, "[DownOnlineConfig] PingUrl fail")
		return
	}

	var err error
	serverConfigs := []constant.ServerConfig{
		{
			ContextPath: sConfig.ContextPath, // Nacos的ContextPath
			IpAddr:      sConfig.IpAddr,
			Port:        uint64(sConfig.Port),
			Scheme:      sConfig.Scheme, // Nacos的服务地址前缀
		},
	}

	// 客户端信息
	cConfig := localCfg.ClientConfig
	clientConfig := constant.ClientConfig{
		NamespaceId:         cConfig.NamespaceId,
		TimeoutMs:           uint64(cConfig.TimeoutMs),
		NotLoadCacheAtStart: cConfig.NotLoadCacheAtStart,
		LogDir:              cConfig.LogDir,
		CacheDir:            cConfig.CacheDir,
		LogLevel:            cConfig.LogLevel,
	}

	// 创建动态配置客户端
	if configClient, err = clients.NewConfigClient(
		vo.NacosClientParam{
			ClientConfig:  &clientConfig,
			ServerConfigs: serverConfigs,
		},
	); err != nil {
		tools.Log.Error(ctx, "[DownOnlineConfig] NewConfigClient err = %v", err)
		return
	}

	// 获取配置
	content, err := configClient.GetConfig(vo.ConfigParam{
		DataId: localCfg.ConfigParam.DataId,
		Group:  localCfg.Namespace})
	if err != nil {
		tools.Log.Error(ctx, "[DownOnlineConfig] GetConfig err = %v", err)
		return
	}

	// 保存配置文件
	if err = os.WriteFile(`./config.yaml`, []byte(content), 0666); err != nil {
		tools.Log.Error(ctx, "[DownOnlineConfig] WriteFile err = %v", err)
	}
}
```

# 监听到配置变化，更新服务
```go
// listenConfig 监听配置中心变化
func listenConfig(localCfg *configs.ConfigCenter) {

	configParam := localCfg.ConfigParam

	for {
		if err := configClient.ListenConfig(vo.ConfigParam{
			DataId: configParam.DataId,
			Group:  localCfg.Namespace,
			OnChange: func(namespace, group, dataId, data string) {
				if err := os.WriteFile(`./config.yaml`, []byte(data), 0666); err != nil {
					tools.Log.Error(ctx, "[listenConfig] WriteFile err = %v", err)
				}
				// TODO
				// cfg := ReadOnlineConfig() // 更新配置
				// pkg.Init(cfg)             // 重载工具
				tools.Log.Error(ctx, "[listenConfig] configs reload success")
			},
		}); err != nil {
			tools.Log.Error(ctx, "[listenConfig] ListenConfig err = %v", err)
			return
		}
	}
}
```