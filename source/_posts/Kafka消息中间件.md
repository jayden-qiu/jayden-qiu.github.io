---
title: Kafka消息中间件
date: 2023-11-09 10:34:08
tags:
categories:
- Go微服务组件
description: 详情点击阅读全文
---

# 配置
```yaml
kafka:
  producer_url:
    - 127.0.0.1:9093
  consumer_url:
    - 127.0.0.1:9093
```

# 初始化
```go
// "github.com/Shopify/sarama"

var (
	Producer sarama.SyncProducer
	Consumer sarama.Consumer
)

func initKafka(cfg *configs.Config) {
	var err error

	// config := sarama.NewConfig()
	// config.Producer.RequiredAcks = sarama.WaitForAll          // 发送完数据需要leader和follow都确认
	// config.Producer.Partitioner = sarama.NewRandomPartitioner // 新选出一个partition
	// config.Producer.Return.Successes = true                   // 成功交付的消息将在success channel返回

	if len(cfg.Kafka.ProducerUrl) != 0 {
		Producer, err = sarama.NewSyncProducer(cfg.Kafka.ProducerUrl, nil)
		if err != nil {
			tools.Log.Error(ctx, "[initKafka] Producer err = %v", err)
		}
	}

	if len(cfg.Kafka.ConsumerUrl) != 0 {
		Consumer, err = sarama.NewConsumer(cfg.Kafka.ConsumerUrl, nil)
		if err != nil {
			tools.Log.Error(ctx, "[initKafka] Consumer err = %v", err)
		}
	}
}
```

# 生产示例
```go
msg := &sarama.ProducerMessage{
	Topic: "web_log",
	Key:   sarama.StringEncoder("imkey"),
	Value: sarama.StringEncoder("imvalue"),
}

pid, offset, err := Producer.SendMessage(msg)
if err != nil {
	fmt.Println("send msg failed, err:", err)
	return
}
fmt.Printf("pid:%v offset:%v\n", pid, offset)
```

# 消费示例
```go
partitionList, err := Consumer.Partitions("web_log") // 根据topic取到所有的分区
if err != nil {
	return
}
// 遍历所有的分区
for partition := range partitionList {
	// 针对每个分区创建一个对应的分区消费者
	pc, err := Consumer.ConsumePartition("web_log", int32(partition), sarama.OffsetNewest)
	if err != nil {
		return
	}
	defer pc.AsyncClose()

	go func(sarama.PartitionConsumer) { // 异步从每个分区消费信息
		for msg := range pc.Messages() {
			fmt.Printf("Partition:%d Offset:%d Key:%v Value:%v", msg.Partition, msg.Offset, string(msg.Key), string(msg.Value))
		}
	}(pc)
}
```