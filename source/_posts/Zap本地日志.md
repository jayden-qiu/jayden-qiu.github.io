---
title: Zap本地日志
date: 2023-11-09 10:16:34
tags:
categories:
- Go微服务组件
description: 详情点击阅读全文
---

# 配置
```yaml
log:
  level: -1 # debug(-1)、info(0)、warn(1)、error(2)
  filename: ./log/grpc.log # 日志存放位置
  max-size: 1 # 最大1m
  max-backups: 5 # 最多5个备份
  max-age: 7 # 只保留7天
  compress: false # 是否压缩
```

# 初始化
```go
// GetLog
func GetLog(cfg *configs.Config) *zap.SugaredLogger {

	writeSyncer := getLogWriter(cfg)

	encoder := getEncoder()

	core := zapcore.NewCore(encoder, writeSyncer, zapcore.Level(cfg.Log.Level))

	logger := zap.New(core, zap.AddCaller())

	// defer logger.Sync() // flushes buffer, if any

	return logger.Sugar()
}

// getEncoder 编码器(如何写入日志)
func getEncoder() zapcore.Encoder {

	encoderConfig := zap.NewProductionEncoderConfig()

	encoderConfig.EncodeTime = zapcore.TimeEncoderOfLayout("2006-01-02 15:04:05") // 转义时间戳

	encoderConfig.EncodeLevel = zapcore.CapitalLevelEncoder // 大写字母表示

	return zapcore.NewConsoleEncoder(encoderConfig)
}

// getLogWriter 日志分割(大小等信息)
func getLogWriter(cfg *configs.Config) zapcore.WriteSyncer {

	lumberJackLogger := &lumberjack.Logger{
		Filename:   cfg.Log.Filename,
		MaxSize:    cfg.Log.MaxSize,    // 最大xM
		MaxBackups: cfg.Log.MaxBackups, // 最多x个备份
		MaxAge:     cfg.Log.MaxAge,     // 保存x天
		Compress:   cfg.Log.Compress,
	}

	return zapcore.AddSync(lumberJackLogger)
}
```

# 使用
全局保存 *zap.SugaredLogger，后使用
```go
log.Infof("%v",data)
```