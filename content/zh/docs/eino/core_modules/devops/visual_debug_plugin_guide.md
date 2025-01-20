---
Description: ""
date: "2025-01-20"
lastmod: ""
tags: []
title: EinoDev 可视化调试插件功能指南
weight: 3
---

# 简介

> 💡
> 对使用 Eino 框架编写的编排产物（Graph，Chain）进行可视化调试，包括：
>
> 1. 编排产物可视化渲染；
> 2. 从可操作的任意节点开始，mock 输入进行调试。

# 快速开始

## 下载 eino-example

> github 仓库：_[https://github.com/cloudwego/eino-examples](https://github.com/cloudwego/eino-examples)_

```bash
# HTTPS
git clone https://github.com/cloudwego/eino-examples.git 

# SSH
git clone git@github.com:cloudwego/eino-examples.git
```

## 安装依赖

在项目目录下依次执行以下指令

```bash
# 1. Pull latest devops repository 
go get github.com/cloudwego/eino-ext/devops@latest

# 2. Cleans and updates go.mod and go.sum
go mod tidy
```

## 运行 Demo

进入 `eino-examples/devops/debug/main.go`，运行 `main.go`。因为插件会同时在本地启动一个 HTTP 服务用于连接用户服务进程，所以会弹出接入网络警告，点击允许。
![](/img/eino/OvNcbzeONoNocVxWRWec4UTQn7e.png)

## 配置调试地址

<table><tbody><tr>
<td>

1.点击左侧或正中间调试功能进入调试配置
<img src="/img/eino/Cqm7bOc5aoeIAdx0Su4cTx64nnB.png" />

</td>
<td>

2.点击配置调试地址
<img src="/img/eino/Wy7abH4QZoJzLQxAlLscmnnDnHh.png" />

</td>
</tr></tbody></table>

<table><tbody><tr>
<td>

3.填入 127.0.0.1:52538
<img src="/img/eino/SHvXbcIRko3tA0xUgFQcMH6Vned.png" />

</td>
<td>

4.点击确认进入调试界面，选择要调试的 Graph
<img src="/img/eino/NAQIbC4yxoKcsRx3tmkc9ZjEnAg.png" />

</td>
</tr></tbody></table>

## 开始调试

<table><tbody><tr>
<td>

1.点击<pre>Test Run</pre>从 start 节点开始执行
<img src="/img/eino/LHU1b6ULtoWRvyxkSSscwr74nBf.png" />

</td>
<td>

2.输入<pre>"hello eino"</pre>，点击确认
<img src="/img/eino/JfEhbOQnzoLQH9xiOB6cPDGCnLf.png" />

</td>
</tr></tbody></table>

<table><tbody><tr>
<td>

3.在调试区域展示有各个节点的输入和输出
<img src="/img/eino/YwZ7bNIvNo0Ab5x5BQWcLrsCnPf.png" />

</td>
<td>

4.点击 Input 和 Output 切换查看节点信息
<img src="/img/eino/B0dkbbObjoiyLixnTEvcECzBn0e.png" />

</td>
</tr></tbody></table>

# 功能一览

## 本地或远程调试

目标调试编排产物无论是在本地电脑还是在远程服务器，都可以通过配置 IP:Port ，主动连接到目标调试对象所在的服务器。
![](/img/eino/Ha4KbcNlZoPJUMxEQYxcWx51nPd.png)

## 编排拓扑可视化

支持 Graph 和 Chain 编排拓扑可视化。
![](/img/eino/R8EYbfenDoeMnfxGjJ9cZ6Hjnff.png)

## 从任意节点开始调试

![](/img/eino/UovrbOrhfooaPcxJVgvcZj8dnQe.png)

## 查看节点执行结果

每个节点执行结果都会按执行顺序展示在调试区域，包括：输入、输出、执行耗时
![](/img/eino/MTC2bA1zRovMwCxZBKzcjzmnnpf.png)

# 从零开始调试

## 使用 Eino 进行编排

插件支持对 Graph 和 Chain 的编排产物进行调试，假设你已经有编排代码如下

```go
func RegisterSimpleGraph(ctx context.Context) {
    g := compose.NewGraph[string, string]()
    _ = g.AddLambdaNode("node_1", compose.InvokableLambda(func(ctx context.Context, input string) (output string, err error) {
       return input + " process by node_1,", nil
    }))
    _ = g.AddLambdaNode("node_2", compose.InvokableLambda(func(ctx context.Context, input string) (output string, err error) {
       return input + " process by node_2,", nil
    }))
    _ = g.AddLambdaNode("node_3", compose.InvokableLambda(func(ctx context.Context, input string) (output string, err error) {
       return input + " process by node_3,", nil
    }))

    _ = g.AddEdge(compose.START, "node_1")
    _ = g.AddEdge("node_1", "node_2")
    _ = g.AddEdge("node_2", "node_3")
    _ = g.AddEdge("node_3", compose.END)

    _, err := g.Compile(ctx)
    if err != nil {
       logs.Errorf("compile graph failed, err=%v", err)
       return
    }
}
```

## 安装依赖

在项目目录下依次执行以下指令

```bash
# 1. Pull latest devops repository 
go get github.com/cloudwego/eino-ext/devops@latest

# 2. Cleans and updates go.mod and go.sum
go mod tidy
```

## 调用调试初始化函数

因为调试需要在用户主进程中启动一个 HTTP 服务，以用作与本地调试插件交互，所以用户需要主动调用一次 _github.com/cloudwego/eino-ext/devops_ 中的 `Init()` 来启动调试服务。

> 💡
> 注意事项
>
> 1. 确保目标调试的编排产物至少执行过一次 `Compile()`。
> 2. `devops.Init()` 的执行必须要在 Graph/Chain 调用 `Compile()` 之前。
> 3. 用户需要保证 `devops.Init()` 执行后主进程不能退出。

如在 `main()` 函数中增加调试服务启动代码

```go
// 1.调用调试服务初始化函数
err := devops.Init(ctx)
if err != nil {
    logs.Errorf("[eino dev] init failed, err=%v", err)
    return
}

// 2.编译目标调试的编排产物
RegisterSimpleGraph(ctx)
```

## 运行用户进程

在本地电脑或者远程环境中运行你的进程，并保证主进程不会退出。

在 _github.com/cloudwego/eino-examples/devops/debug/main.go__ _中，`main()` 代码如下

```go
func main() {
    ctx := context.Background()

    _// Init eino devops server_
_    _err := devops.Init(ctx)
    if err != nil {
       logs.Errorf("[eino dev] init failed, err=%v", err)
       return
    }

    _// Register chain, graph and state_graph for demo use_
_    _chain.RegisterSimpleChain(ctx)
    graph.RegisterSimpleGraph(ctx)
    graph.RegisterSimpleStateGraph(ctx)

    _// Blocking process exits_
_    _sigs := make(chan os.Signal, 1)
    signal.Notify(sigs, syscall.SIGINT, syscall.SIGTERM)
    <-sigs

    _// Exit_
_    _logs.Infof("[eino dev] shutting down\n")
}
```

## 配置调试地址

- **IP**：用户进程所在服务器的 IP 地址。
  - 用户进程运行在本地电脑，则填写 `127.0.0.1`；
  - 用户进程运行在远程服务器上，则填写远程服务器的 IP 地址，兼容 IPv4 和 IPv6 。
- **Port**：调试服务监听的端口，默认是 `52538`，可通过 WithDevServerPort 这一 option 方法进行修改

> 💡
> 注意事项
>
> - 本地电脑调试：系统可能会弹出网络接入警告，允许接入即可。
> - 远程服务器调试：需要你保证端口可访问。

IP 和 Port 配置完成后，点击确认，调试插件会自动连接到目标调试服务器。如果成功连接，连接状态指示器会变成绿色。
![](/img/eino/XnWRblIfroE1f2xENJ0cafzknKe.png)

## 选择目标调试编排产物

确保你目标调试的编排产物至少执行过一次 `Compile()`。因为调试设计是面向编排产物实例，所以如果多次执行 `Compile()`，会在调试服务中注册多个编排产物，继而在选择列表中看到多个可调试目标。
![](/img/eino/R0KVbWPXLoeOSmxuM8jcQ8CanDh.png)

## 开始调试

调试支持从任意节点开始调试，包括 start 节点和其他中间节点。

1. 从 START 节点开始调试：直接点击 Test Run，然后输入 mock 的 input（如果 input 是复杂结构的话，会自动对 input 的结构进行推断）然后点击确定，开始执行你的 graph，每个 node 的结果会在下方显示
   ![](/img/eino/FwW3b8NUkoahCkxqPi5cvHjInje.png)
   ![](/img/eino/JKFZbjqtVosHlnxViabc2botnkd.png)
2. 从任意的可操作节点开始调试：

比如，从第二个 node 开始执行
![](/img/eino/LSsxbZVKmoXs02xu2m4cDP2jndf.png)
![](/img/eino/FSsLbpTKzoYgthxzPyccaEyanse.png)

## 查看执行结果

从 START 节点开始调试，点击 Test Run 后，在插件下方查看调试结果。
![](/img/eino/Tg2fbTtxzohB5exwH4ScewuRnQh.png)

从任意的可操作节点进行调试，在插件下方查看调试结果。
![](/img/eino/GKWabv4eQofx2hx9m7HcUkNUn2e.png)
