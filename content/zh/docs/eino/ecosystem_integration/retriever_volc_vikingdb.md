---
Description: ""
date: "2025-01-07"
lastmod: ""
tags: []
title: Retriever - volc VikingDB
weight: 0
---

## **基本介绍**

火山引擎 VikingDB 检索器是 Retriever 接口的一个实现，火山引擎 VikingDB 是火山引擎提供的向量数据库服务，提供了高性能的向量检索能力，本组件通过火山引擎 VikingDB Go SDK 与服务进行交互。该组件实现了 [[🚧]Eino: Retriever 使用说明](/zh/docs/eino/core_modules/components/retriever_guide)

## **使用方式**

### **组件初始化**

火山引擎 VikingDB 检索器通过 `NewRetriever` 函数进行初始化，主要配置参数如下：

```go
retriever, err := NewRetriever(ctx, &RetrieverConfig{
    // 服务配置
    Host:              "api.volcengineapi.com", // 服务地址
    Region:            "cn-beijing",            // 区域
    AK:                "your-ak",               // 访问密钥 ID
    SK:                "your-sk",               // 访问密钥密码
    Scheme:            "https",                 // 协议
    ConnectionTimeout: 30,                      // 连接超时时间（秒）
    
    // 数据配置
    Collection: "collection-name",  // 集合名称
    Index:      "index-name",      // 索引名称
    
    // 向量化配置
    EmbeddingConfig: EmbeddingConfig{
        UseBuiltin:   true,        // 是否使用内置向量化
        ModelName:    "model-name",// 模型名称
        UseSparse:    true,        // 是否使用稀疏向量
        DenseWeight:  0.5,         // 稠密向量权重
        Embedding:    embedder,    // 自定义向量化器
    },
    
    // 检索配置
    Partition:      "partition",   // 分区名称
    TopK:           ptrOf(100),   // 返回结果数量
    ScoreThreshold: ptrOf(0.7),   // 相似度阈值
    
    // 过滤配置
    FilterDSL: map[string]any{    // DSL 过滤条件
        "term": map[string]any{
            "field": "value",
        },
    },
})
```

### **检索文档**

文档检索通过 `Retrieve` 方法实现：

```go
docs, err := retriever.Retrieve(ctx, "查询文本", retriever.WithTopK(5))
```

### **完整使用示例**

#### **基本检索**

```go
package main

import (
    "context"
    
    "github.com/cloudwego/eino-ext/components/retriever/volc_vikingdb"
)

func main() {
    ctx := context.Background()
    
    // 初始化检索器
    r, err := volc_vikingdb.NewRetriever(ctx, &volc_vikingdb.RetrieverConfig{
        Host:       "api.volcengineapi.com",
        Region:     "cn-beijing",
        AK:         "your-ak",
        SK:         "your-sk",
        Collection: "your-collection",
        Index:      "your-index",
        EmbeddingConfig: volc_vikingdb.EmbeddingConfig{
            UseBuiltin: true,
            ModelName:  "model-name",
            UseSparse:  true,
            DenseWeight: 0.5,
        },
        TopK: ptrOf(5),
    })
    if err != nil {
        panic(err)
    }
    
    // 执行检索
    docs, err := r.Retrieve(ctx, "如何使用 VikingDB？")
    if err != nil {
        panic(err)
    }
    
    // 处理结果
    for _, doc := range docs {
        println("文档ID:", doc.ID)
        println("内容:", doc.Content)
        println("相似度:", doc.MetaData["_score"])
    }
}
```

#### **自定义向量化**

```go
package main

import (
    "context"
    
    "github.com/cloudwego/eino-ext/components/retriever/volc_vikingdb"
    "github.com/cloudwego/eino/components/embedding"
)

func main() {
    ctx := context.Background()
    
    // 初始化向量化器 （以 openai 为例）
    embedder, err := &openai.NewEmbedder(ctx, &openai.EmbeddingConfig{})
    if err != nil {
        panic(err)
    }
    
    // 初始化检索器
    r, err := volc_vikingdb.NewRetriever(ctx, &volc_vikingdb.RetrieverConfig{
        Host:       "api.volcengineapi.com",
        Region:     "cn-beijing",
        AK:         "your-ak",
        SK:         "your-sk",
        Collection: "your-collection",
        Index:      "your-index",
        EmbeddingConfig: volc_vikingdb.EmbeddingConfig{
            UseBuiltin: false,
            Embedding:  embedder,
        },
    })
    if err != nil {
        panic(err)
    }
    
    // 执行检索
    docs, err := r.Retrieve(ctx, "查询文本")
    if err != nil {
        panic(err)
    }
    
    // 处理结果
    for _, doc := range docs {
        println(doc.Content)
    }
}
```

## **相关文档**

- [Eino: Retriever 使用说明](/zh/docs/eino/core_modules/components/retriever_guide)
- [火山引擎 VikingDB 文档](https://www.volcengine.com/docs/84313)火山引擎 VikingDB 文档
