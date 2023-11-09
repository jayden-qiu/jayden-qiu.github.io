---
title: Gorm数据库驱动
date: 2023-11-09 10:17:03
tags:
categories:
- Go微服务组件
description: 详情点击阅读全文
---

# 配置
```yaml
mysql:
  path: 127.0.0.1
  port: 3306
  config: charset=utf8mb4&parseTime=True&loc=Local
  db-name: localpro
  username: root
  password: root
```

# 初始化
```go
// "gorm.io/driver/mysql"
// "gorm.io/gorm"

// DB 数据库驱动
var DB *gorm.DB

// initGorm 初始化数据库
func initGorm(cfg *configs.Config) {
	mcfg := cfg.Mysql

	dsn := fmt.Sprintf("%s:%s@tcp(%s:%s)/%s?%s", mcfg.Username, mcfg.Password, mcfg.Path, mcfg.Port, mcfg.DbName, mcfg.Config)

	// newLogger := logger.New(
	// 	log.New(os.Stdout, "\r\n", log.Ldate), // io writer（日志输出的目标，前缀和日志包含的内容——译者注）
	// 	logger.Config{
	// 		SlowThreshold:             time.Second, // 慢 SQL 阈值
	// 		LogLevel:                  logger.Info, // 日志级别
	// 		IgnoreRecordNotFoundError: true,        // 忽略ErrRecordNotFound（记录未找到）错误
	// 		Colorful:                  true,        // 彩色打印
	// 	},
	// )

	DB, _ = gorm.Open(mysql.Open(dsn)) // &gorm.Config{Logger: newLogger}
}
```
