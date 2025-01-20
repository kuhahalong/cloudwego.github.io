---
Description: ""
date: "2025-01-20"
lastmod: ""
tags: []
title: Eino IDE 插件使用指南
weight: 0
---

# 背景

> [Eino 用户手册](/zh/docs/eino)
>
> Eino 是 Go AI 集成组件的研发框架，提供了 AI 应用相关的常用组件以及集成组件编排能力，为了更好的辅助开发者使用 Eino，我们提供了 **GoLand Eino IDE 插件**，现在就[安装插件](/zh/docs/eino/core_modules/devops/ide_plugin_guide)安装插件，助你高效开发 🚀

# 插件信息

## 简介

![](/img/eino/PQliwa6v2hpKcYb4cVFcAvAAnsh.png)

<table><tbody><tr>
<td>

<strong>EinoDev Graph 调试</strong>
<img src="/img/eino/NE5lb0yWLo8EsXxQlSFc7HPDn6T.png" />
<img src="/img/eino/VFcgbc8ojoIyGbxKlzjc3U1FnWc.png" />

</td>
<td>

<strong>EinoDev Graph 编排</strong>
<img src="/img/eino/BYzHbgtFWo7yt1xC4JCcyKwFnig.png" />
<img src="/img/eino/Y0cGbUJ5roaVB9xE6M3cIa3enve.png" />

</td>
</tr></tbody></table>

**Eino 代码可视化展示**

<table><tbody><tr>
<td>
<img src="/img/eino/FceEbZBjWoy8RSxFd2Ech1fance.png" />

</td>
<td>
<img src="/img/eino/AATBb5fvIo8J38x36o9c6D7WnRA.png" />
<img src="/img/eino/JkX7baXSOo0vm3xLhaxcatOMn1g.png" />
</td>
</tr></tbody></table>

## 安装插件

<table><tbody><tr>
<td>
1. 进入<strong>GoLand</strong>，点击<strong>设置</strong>
<img src="/img/eino/NTESbVXFtoXITZxWt5EcOCNmnVh.png" />
</td>
<td>
1. 进入<strong>插件</strong>，选择<strong>管理插件仓库</strong>
<img src="/img/eino/Yhz5b2r1goSKkMxDQj8c9xAUnNg.png" />
</td>
</tr></tbody></table>

<table><tbody><tr>
<td>
1. 点击「+」，填入<strong>https://fornax.bytedance.net/api/fe-plugin/jet-brains.xml</strong>，点击确定
<img src="/img/eino/HXVib7Y0Oo6merxvcJNcR8cHnQb.png" />
</td>
<td>
1. 进入「Marketplace」，搜索<strong>Eino</strong>，点击安装并<strong>重启</strong>IDE
<img src="/img/eino/LQxcbj5xcoFog3xjtwCc9Cq7nse.png" />

</td>
</tr></tbody></table>

1. 插件安装完毕 🎉 可以在 IDE 右侧插件列表中看到 EinoDev 图标
   ![](/img/eino/TJ56bmvQ3oREDfxAjcNc8eFKn3c.png)

# 

### 定义编排产物

首先，你已经使用 eino 编写出来了一个 graph（本文以 graph 为例，chain 的可视化调试步骤和 graph 完全一致），如下所示：

```go
// 这是一个简单的有3个自定义node的graph
g := compose.NewGraph[string, string]()
err := g.AddLambdaNode("node_1", compose.InvokableLambda(func(ctx context.Context, input string) (output string, err error) {
    return input + " process by node_1,", nil
}))
err = g.AddLambdaNode("node_2", compose.InvokableLambda(func(ctx context.Context, input string) (output string, err error) {
    return input + " process by node_2,", nil
}))
err = g.AddLambdaNode("node_3", compose.InvokableLambda(func(ctx context.Context, input string) (output string, err error) {
    return input + " process by node_3,", nil
}))

err = g.AddEdge(compose._START_, "node_1")
err = g.AddEdge("node_1", "node_2")
err = g.AddEdge("node_2", "node_3")
err = g.AddEdge("node_3", compose._END_)
```

### 引用 einodev

其次，在你的项目中引用 einodev，执行如下命令

```go
go get code.byted.org/flow/einodev@latest
```

### 增加调试 server 启动代码

然后，在你的 main.go 文件中的 main 函数增加调试 server 启动代码（业务方可以自己判断是否需要 boe 环境才启动 einodev 可视化调试 server）

```go
~~err := einodev.Run(ctx) ~~// Run方法废弃
err := einodev.Init(ctx)
if err != nil {
    fmt.Printf("[eino dev] init failed, err=%v\n", err)
    return
}
```

> 💡
> 注意：
>
> 1. einodev.Init 的执行必须要在 graph compile 之前，否则调试插件无法获取到调试信息
> 2. 用户需要自己保证 einodev.Init 执行后主进程不能退出

![](/img/eino/TSdub0YHbo3lcFxwqBqc0Ywpnwh.png)

调试 server 默认会使用 52538 作为调试 server 的端口号，如果你想要指定特定的端口号，可以使用 einodev.WithDevServerPort("***") 这一 option 方法来设置你的自定义端口号

### 启动调试

接下来，在本地或者 cloudDev 或者 tce boe 中启动你的服务，然后确保至少执行过一次 graph 的 compile 方法（只有 graph 的 compile 方法被执行过一次，调试服务才能获取到你构建的 graph 的详细信息）

在 demo 中，对应的注册代码为
![](/img/eino/OqQOb42SnoIp8tx7Q9zcIM3knjb.png)

1. 如果你在本地调试，ip 填写 127.0.0.1 即可
   ![](/img/eino/I8qmbwQTRokk7pxcl0Zck8lonUg.png)
2. 如果你在 CloudDev 中进行调试

在 CloudDev 界面上拿到容器的 ip 地址，port 为 einodev 监听的端口（默认为 52538）

<table><tbody><tr>
<td>
<img src="/img/eino/W85TbfXAroDhm5xeLJEcmqh1nHe.png" />
</td>
<td>
<img src="/img/eino/IOpvbgPybo5mZ7xMvt5cbGaOnlc.png" />
</td>
</tr></tbody></table>

1. 如果在 tce boe 中启动的你的服务，port 需要手动配置，如下所示

首先，添加调试 sever 的端口号（默认为 52538）,然后升级下你的服务；
![](/img/eino/L13cbvPOSoDOTnxhHg0cTdEendb.png)

然后找到配置的端口号映射出来的随机端口号（图中的 11011）
![](/img/eino/SDQ3bdVlDo412MxXDwCcOo8inBc.png)

然后拿到 ip 地址
![](/img/eino/Shizb1lO3ozpUCxDtw0cgWdBnBg.png)

# 

## 简介

> 💡
> 在 UI 上完成 eino graph 编排，并导出编排模板代码到任意路径

## 插件使用

### 使用

<table><tbody><tr>
<td>
1. 可视化编排入口
<img src="/img/eino/GLeTbJbzRoGYLWxDe5acbh3Jnhh.png" />
</td>
<td>
1. 新创建 Graph 或者选择历史 Graph
<img src="/img/eino/Ff4lbcKGEoS0qWxpcc1cqzfynMb.png" />
</td>
</tr></tbody></table>

<table><tbody><tr>
<td>
1.  选择新创建 Graph 后，填写 Graph 名称以及节点触发模式
<img src="/img/eino/PVyYbnmj1oVSHjxNtgZcfbmBnOe.png" />
</td>
<td>
1. 按需选择具体类型组件
<img src="/img/eino/VlHWbIjCIoeeQ1xQDzycD3ZsnCf.png" />
</td>
</tr></tbody></table>

<table><tbody><tr>
<td>
1. 完善节点信息，包括 NodeKey 和 组件构造方法名
<img src="/img/eino/R8Dtb33DVo7l6wxQIqHcr3qjntf.png" />
<img src="/img/eino/FL2ObLWlQo8J6Gx3x4Pcr2C5nAg.png" />
</td>
<td>
1. 完成连线
<img src="/img/eino/MKJ1boEx3oRsSdxLHY2cPBoinxf.png" />

可选择自动优化布局
<img src="/img/eino/X50gbC08HopiYzxQPv1cal7jn8h.png" />

</td>
</tr></tbody></table>

<table><tbody><tr>
<td>
1. 选择导出 Graph ，点击确认（自定义路径是绝对路径）
<img src="/img/eino/EO0nbfj4ZoNT7KxFNuicay8Zn4e.png" />
</td>
<td>
1. 在 IDE 左边，选中项目文件夹右键并选择 reload from disk 后，即可看见生成代码（优化中）
<img src="/img/eino/PaolbW5XdobsDDxJRVGcoJdFndb.png" />
</td>
</tr></tbody></table>
