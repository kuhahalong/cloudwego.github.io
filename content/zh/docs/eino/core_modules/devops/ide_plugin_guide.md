---
Description: ""
date: "2025-01-17"
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

# **EinoDev Graph 调试**

## 简介

> 💡
> 对使用 eino 框架编写出的编排产物（graph，chain）进行可视化调试（调试能力包含： 1. 编排产物的可视化渲染；2. 以及从任意可操作的节点开始，mock 节点的输入并进行调试运行）

## 项目配置&服务启动

通过本用户手册，你可以使用我们提供的 Goland IDE 插件（本期仅支持 Goland IDE）来对你使用 eino 框架编写出的编排产物进行可视化调试

> 教程中呈现的 demo 的项目地址：[https://code.byted.org/flow/eino-examples/tree/master/einodev](https://code.byted.org/flow/eino-examples/tree/master/einodev)
>
> [【Eino 可视化调试】：IDE  Render  Schema 定义](https://bytedance.larkoffice.com/docx/JXsNdNLxRo6GGpxdIamc8oannGh#TQ1ldJtTbohwTUxTfavcmgpDngc)

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

## 插件使用

### 配置调试地址

如上所述，拿到 IP 和 Port；将 IP 和 Port 配置到 IDE 插件中

<table><tbody><tr>
<td>
<img src="/img/eino/SaNObxTpxobskFxyb1kchWKGnXJ.png" />
</td>
<td>
<img src="/img/eino/QhUfbSmyXo4EB6xTvxWcUYb2nJf.png" />
</td>
</tr></tbody></table>

### 查看可视化编排产物

确保你要调试的编排产物的 compile 代码被执行一次，然后可以在 IDE 插件中看到你注册的 graph；
![](/img/eino/GsNqb0Jo0oU3C1xsd6tczvn2nag.png)

### 调试编排产物

调试可以从 START 节点或者 START 后的任意的可操作节点开始

1. 从 START 节点开始调试：直接点击 Test Run，然后输入 mock 的 input（如果 input 是复杂结构的话，会自动对 input 的结构进行推断）然后点击确定，开始执行你的 graph，每个 node 的结果会在下方显示
2. 从任意的可操作节点开始调试：

比如，从第二个 node 开始执行

## 常见问题

### 为什么渲染出的可视化 graph 上有些节点是不可操作的？

- **对于 Graph 中存在的节点 Node 是否支持调试，目前有如下规定：**
  - 基础类型 与 基础类型的指针支持调试操作
    - String/Number/Bool 以及其指针类型
  - Struct 结构体类型
    - 结构体中 Fields 字段至少存在一个可支持调试类型则该节点可支持调试
  - map 类型 有如下规定
    - Map Key 必须为基础类型
    - Map Value  ① 基础类型； ② 可操作的 map，slice ，struct 类型及其指针
  - Slice 类型
    - ① 基础类型  ；② 可操作的 map，slice ，struct 类型及其指针
- **对于 Graph 图中 Start 节点****/Test Run****是否支持调试，有如下规定：**
  - Graph 中 Start 节点之下的 Nodes 没有设置 input key 场景
    - 如 Graph 定义时指定了 Input 入参可支持调试，则 graph 调试入参为此 Input 类型
    - 如 Graph 定义时 Input 入参不支持调试(interface 接口类型），则通过推断 Nodes 节点是否支持调试，如 Nodes 节点 Input 入参支持调试，则 Graph 的入参为此 Nodes 节点的 Input 可调试类型 。
  - Graph 中 Start 节点之下的 Nodes 存在设置 input key 场景
    - 如果主 Graph 下的 Nodes input 支持调试，则此 Graph 支持调试，由于 Nodes 设置了 inputkey ，调试时，需按着 inputkey 对应的 Node 类型设置入参格式。

### StateGraph 如何调试

> 💡
> 引用最新的 einodev，用户不需要再主动注入 GenLocalState 方法，就可以直接调试 StateGraph。

~~用户用 eino 创建 StateGraph 时，需要提供 State 的实例化方法。 相应地，使用 einodev 调试 StateGraph 也需要提供 State 的实例化方法。~~

~~在 CompileCallbacks 中使用 einodev 提供的~~~~WithGenLocalState~~~~这一 option 方法注册你的 State 的实例化方法。~~

示例：[https://code.byted.org/flow/eino-examples/blob/master/einodev/graph/state_graph.go](https://code.byted.org/flow/eino-examples/blob/master/einodev/graph/state_graph.go)

# **Eino 可视化编排**

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

### Demo

demo 实现了一个简单的 ReAct Agent，包含 ChatTemplate、ChatModel、Lambda 3 个组件以及 1 个分支。

> 💡
> 生成的模板代码 demo 地址：[https://code.byted.org/flowdevops/eino_nclc_demo](https://code.byted.org/flowdevops/eino_nclc_demo)

# **Q&A**

### 安装插件出现报错

尝试升级 GoLand 到较新版本
![](/img/eino/img_v3_02gq_fc0bb51b-7a5b-4dfa-ad72-37e232839e6g.jpg)
