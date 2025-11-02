# 利用 AI Studio Build 页面实现 Gemini API 免费代理

> **补充**: 用指纹浏览器开个新窗口登录 Google，然后到指纹浏览器编辑窗口，把 Cookie 复制出来用，然后删除浏览器窗口就行，这个 Cookie 超稳!!!

> **说明**: 简单玩完才发现已经有过类似项目了，不过可能好处是更简单了点吧，毕竟 Go 构建直接是二进制文件，而且也做了多个网页窗口连接的轮询均衡。

## 项目简介

本项目通过巧妙利用 Google AI Studio Build 页面的内部代理机制，实现了一个本地代理服务，让用户可以在无需 API Key 的情况下，免费调用 Gemini API。

## 🚀 项目起因

灵感来源于 [Linux.do 论坛](https://linux.do/t/topic/702361)，其中提到了一个关键发现：

> 在 Google AI Studio 的 Build 页面里，构建出的 App 调用模型时，是不需要输入 API Key 的。

这是因为 Build 页面的 iframe 会 hook (劫持) JavaScript 的 fetch 函数，如果请求的端点是 Gemini API (`generativelanguage.googleapis.com`)，流量会通过一个 Google 提供的官方代理。

既然 AI Studio 已经为我们做好了 API 请求的代理，我们便可以利用这个特性，将我们自己的 Gemini API 请求转发到这个 Build 页面，让它为我们"代劳"。

## 💡 核心思路

通过创建一个本地代理服务器和一个特制的 AI Studio Build 页面，将本地的 Gemini API 请求通过 WebSocket 协议转发到 Build 页面，再由该页面利用 Google 内置的代理访问 Gemini API，从而实现免 API Key 调用。

## 🛠️ 项目组成

### 1. 本地代理服务端 (Local Proxy Server)

- **功能**: 监听本地端口，接收标准的 Gemini API 请求，并通过 WebSocket 将其转发给前端页面
- **源码/下载**: [GitHub - cliouo/aistudio-build-proxy](https://github.com/cliouo/aistudio-build-proxy)

### 2. AI Studio Build Demo 页面

- **功能**: 一个部署在 Google AI Studio 上的简单网页，负责建立 WebSocket 连接，接收来自本地代理的请求，然后使用 fetch 调用 Gemini API
- **在线地址**: [https://aistudio.google.com/apps/drive/1C9ch97X-CfJ5AQhBo0nYgbkfkoD3CESy?showPreview=true](https://aistudio.google.com/apps/drive/1C9ch97X-CfJ5AQhBo0nYgbkfkoD3CESy?showPreview=true)

## 📋 使用方法

### 第一步：下载并运行服务端

1. 前往项目的 GitHub Releases 页面下载对应您操作系统的可执行文件
2. 解压文件，在命令行/终端中直接运行它
3. 服务启动后，将默认监听在本地地址：`http://127.0.0.1:5345`

### 第二步：连接代理前端

1. 在浏览器中打开 [AI Studio Build Demo 链接](https://aistudio.google.com/apps/drive/1C9ch97X-CfJ5AQhBo0nYgbkfkoD3CESy?showPreview=true)
2. 页面加载后，点击 "Connect WS" 按钮
3. 如果一切正常，状态会从 "Not connected" 变为 "Connected"，表示网页已成功连接到您本地运行的代理服务端

*注意：原文档中提到的图片尺寸信息为 2091×1449 233 KB*

### 第三步：配置 AI 对话工具

现在，您可以在自己的 AI 对话工具（如 LobeChat, Cherry Studio, One API 等）中配置 Gemini 模型：

- **提供商类型**: 选择 Gemini
- **API 地址 (Base URL)**: 填入本地代理服务的地址 `http://127.0.0.1:5345`
- **API Key**: 留空即可

*注意：原文档中提到的配置示例图片尺寸分别为 522×484 6.8 KB 和 805×806 29.1 KB*

### 第四步：开始使用

配置完成后，您就可以像正常使用一样与 Gemini 模型进行对话了。所有请求都会通过上述搭建的代理通道进行。

*注意：原文档中提到的使用示例图片尺寸为 1125×534 33.1 KB*

## ⚠️ 注意事项

- Gemini 1.5 Pro 等隐藏的模型仅支持流式调用
- 会边报错边输出的模型用不了

## 🌐 技术原理解析

整个请求的流量路径如下：

```
[AI 对话工具 (如 Cherry)] → [本地代理服务 (localhost:5345)] 
↓ WebSocket
[AI Studio Build Demo 网页] → [AI Studio 内置代理] → [Google Gemini API]
```

1. **发起请求**: 您的 AI 对话工具向 `http://127.0.0.1:5345` 发送一个标准的 Gemini API 请求
2. **本地拦截与转发**: 本地代理服务接收到这个 HTTP 请求，但它自己不去调用 Google API，而是将请求内容通过 WebSocket 连接发送给已经打开的 Build Demo 网页
3. **网页中转**: Build Demo 页面中的 JavaScript 代码接收到 WebSocket 消息，并使用 fetch 函数向真正的 Gemini API 地址发起请求
4. **代理劫持**: 由于该网页运行在 AI Studio 的 iframe 环境中，这个 fetch 请求被 AI Studio 的内置机制劫持，并通过 Google 的官方代理发出。这个代理会自动处理身份验证，因此不需要 API Key
5. **响应返回**: Gemini API 的响应沿着原路返回，最终呈现在您的 AI 对话工具中
