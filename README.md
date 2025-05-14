# J2MCP
使用java开发开箱即用的MCP插件

## 功能特点

- 🚀 支持群聊和私聊消息转发
- 🔔 支持@消息识别和处理
- 🔄 支持消息队列和重试机制
- 🔒 支持API Token认证
- 🌐 支持多Webhook地址配置
- 📝 实时日志显示
- 🎯 支持自定义监听对象
- 💾 配置文件自动保存
- 🔄 自动重连和错误恢复
- 🔍 支持文件编码自动检测

## 系统要求

- Windows 10 或更高版本
- Python 3.9 或更高版本（仅源码运行需要）
- 微信 PC 版 3.9.x

## 教程
https://zerotrue.xyz/article/1d6484d9-4c67-80eb-926d-ff6fb1588f60

## 安装步骤

### 方式一：直接运行（推荐）

1. 从 [Releases](https://github.com/Obito-404/RuoYuBot/releases) 页面下载最新版本的 `RuoYuBot.exe`
2. 双击运行 `RuoYuBot.exe`
3. 首次运行会自动创建配置文件

### 方式二：源码运行

1. 克隆仓库：
```bash
git clone https://github.com/Obito-404/RuoYuBot.git
cd ruoyubot
```

2. 安装依赖：
```bash
# 创建虚拟环境（推荐）
python -m venv venv
.\venv\Scripts\activate  # Windows
source venv/bin/activate  # Linux/Mac

# 安装依赖
pip install -r requirements.txt
```

3. 运行程序：
```bash
python main.py
```

4. 打包命令：
```bash
 pyinstaller --noconfirm --onefile --windowed --name "RuoYuBot" --icon "D:/ruoyubot/icon.ico" --hidden-import comtypes --hidden-import comtypes.stream --hidden-import comtypes.gen --hidden-import win32com --hidden-import win32com.client "D:/ruoyubot/main.py"
```

## 依赖说明

主要依赖包：
- wxauto: 微信自动化操作
- requests: HTTP请求
- flask: Web服务器
- pywin32: Windows系统API
- chardet: 文件编码检测
- werkzeug: Flask服务器组件

## 配置说明

程序首次运行时会自动创建`config.ini`配置文件，包含以下配置项：

- `listen_list`: 需要监听的群聊或好友名称（用逗号分隔）
- `webhook_urls`: 消息转发目标地址（每行一个）
- `port`: 本地webhook服务器端口
- `retry_count`: 消息发送重试次数
- `retry_delay`: 重试延迟时间（秒）
- `log_level`: 日志级别
- `api_token`: API认证令牌

## 使用方法

1. 启动程序后，会自动打开微信（如果未运行）
2. 在GUI界面中配置：
   - 添加需要监听的群聊或好友
   - 设置webhook回调地址
   - 配置API Token
   - 设置本地端口

3. 点击"开始"按钮启动服务

## Webhook接口说明

### 接收消息格式

```json
{
    "target_user": "群聊名称/好友名称",
    "message": "消息内容",
    "timestamp": "2024-01-01 12:00:00",
    "is_group": true/false,
    "sender": "发送者名称",
    "chat_name": "群聊名称",
    "is_at_me": true/false
}
```

### 发送消息格式

```json
{
    "target_user": "群聊名称/好友名称",
    "message": "要发送的消息",
    "is_group": true/false,
    "at_list": ["要@的成员列表"]
}
```

## 注意事项

1. 确保微信已登录并保持在线
2. 监听对象名称需要严格匹配（区分大小写）
3. 建议定期检查日志，及时处理异常情况
4. 请妥善保管API Token，避免泄露
5. 配置文件使用UTF-8编码，支持中文

## 常见问题

1. Q: 程序无法启动微信，提示无效窗口句柄？
   A: 请确保已安装微信PC版3.9.x版本，并且已经登录。

2. Q: 消息转发失败？
   A: 检查webhook地址是否正确，网络连接是否正常，API Token是否配置正确。

3. Q: 无法收到群聊消息？
   A: 确认群聊名称是否正确，是否已添加到监听列表。

4. Q: 配置文件出现乱码？
   A: 程序会自动检测并转换配置文件编码为UTF-8。

## 贡献指南

欢迎提交Issue和Pull Request来帮助改进项目。

## 许可证

MIT License

## 联系方式

如有问题或建议，请提交Issue。 



## 1、前言
最近 Anthropic 主导发布了 MCP (Model Context Protocol，模型上下文协议)后，着实真真火了一把。熟悉 A大模型的人对 Anthropic 应该不会陌生，Claude 3.5 Sonnet 模型就是他们发布的，包括现在的最强编程 Al 模型3.7 Sonnet。今天我们来刨析下什么是 MCP，AI 大模型下，需要 MCP 吗?

## 2、什么是 MCP?
MCP(Model Context Protocol)模型上下文协议，是一种适用于 A 大模型与数据源交互的标准协议。旨在实现跨模型、跨会话的上下文信息持久化与动态共享。它通过标准化接口实现模型间的上下文传递、版本控制和协同推理，解决复杂 AI任务中的上下文碎片化问题。
那为什么需要重新定义 A| 的上下文管理? 在我们最开始使用 A| 大模型的时候，经常会遇到以下一些问题，如:上下文可能莫名的丢失，导致会话的语义出现断层。再如:不同模型之间，比如 ChatGPT和 DeepSeek 之间记忆不互通，或者不同版本之间无法追踪上下文的变更历史等等，这些其实都是信息孤岛。
因此，我们需要有一个专门的协议或者说技术能跨模型、跨版本串联我们的上下文，解决记忆互通的问题。如果我们观察整个 AI 自动化的发展过程，我们会发现 AI自动化分三个阶段:AIChat、AlComposer、AIAgent.
·AI Chat 通常只是提供建议和代码片段，人机协作的模式通常需要用户手动进行复制，调试，集成。·Al Composer 相比 AIChat 支持了局部的代码自动修改，我们只需要确认接受或拒绝接受即可。但是也局限于
代码的上下文操作。·Al Agent 可以完成端到端的任务闭环。他是一个完全的智能体，自主决策，修改，调整我们所需的代码块，或者自主提取需要的信息。我们只需要负责监控动作就行。但是跨系统的协作能力有所欠缺。因此，进一步的演化路线自然是为了完善 AlAgent 的跨模型或跨系统而实现。他是一个中间层，可以作为 AIAgent 的智能路由中枢。
熟悉 Java 分布式的朋友应该会发现，这个其实很想分布式的发展进程。而 MCP 更多像是一个 RPC 的标准协议一样。不确定我这么比喻是否恰当。

## 3、MCP 架构
模型上下文协议(MCP)遵循客户端-主机-服务器架构，其中每个主机可以运行多个客户端实例。这种架构使用户能够跨应用程序集成 AI功能，同时保持明确的安全边界并隔离问题。MCP 基于 JSON-RPC 构建，提供有状态会话协议，专注于客户端和服务器之间的上下文交换和采样协调。官方的 MCP 系统架构图:
![image](https://github.com/user-attachments/assets/982ee53c-236a-402d-85ab-c2af86faa6db)

可以发现 MCP 基于三层分层模式:
**MCP Host(主机应用)**:运行 A| 模型或代理的宿主程序，如 Claude 桌面版、某IDE 中的 AI助手等。主机应用通过内置的 MCP 客户端与外部建立连接，是连接的发起方。
**MCP Cient(客户端)**:嵌入在主机应用中的协议客户端组件。每个MCP 客户端与一个特定的 MCP 服务器保持一对一的连接，用于向服务器发送请求或接收响应。一个主机应用中可以运行多个 MCP 客户端，从而同时连接多个不同的服务器。
**MCP Server(服务器)**:独立运行的轻量程序，封装了某一数据源或服务的具体能力，通过标准化的 MCP 接口对外提供。每个 MCP 服务器相当于一个“适配器”将底层的数据源/工具(本地文件、数据库、第三方API等)的功能以统一格式暴露出来，供客户端调用。
**本地数据源**:部署在用户本地环境中的数据资源，例如本机文件系统、数据库、应用服务等。MCP 服务器可以受控地访问这些资源，并将内容提供给 AI模型使用。例如，一个本地 MCP 服务器可以读取电脑文件或查询本地 SQLite 数据库，然后将结果发给模型作为参考。
**远程服务**:通过网络提供的外部系统或在线服务(通常通过HTTPAPI访问)。MCP 服务器也可以连接到这些远程服务获取教据。比如，可以有一个 MCP 服务器连接 Slack的 Web AP|，代表 Al助手执行发送消息、读取频道记录等操作。

## 4、MCP & JAVA 支持
MCP 提供了多种编程语言的支持，如 Python，Java，Kotlin，TypeScript等。这里以 Java SDK为例，官方也提供了相关文档:https://modelcontextprotocol.io/sdk/java/mcp-overviewSpring Al也已经支持了 MCP 了。

### 4.1、多种传输方式
MCP 提供了两种不同的传输实现。
·默认传输方式:基于 Stdio，HTTP SSE
基于 Spring 的传输:WebFlux SSE，WebMVC SSE4.2、JAVA 应用的基础架构 JAVA SDK 也是遵循分层结构:
![image](https://github.com/user-attachments/assets/7320829e-28e5-4ace-b9ed-e5057ff95123)
客户端/服务器层(McpClient/McpServer):两者都使用 McpSession 进行同步/异步操作，其中McpClient 处理客户端协议操作，McpServer管理服务器端协议操作。
会话层(McpSession):使用 DefaultMcpSession 实现管理通信模式和状态
传输层(McpTransport):通过以下方式处理JSON-RPC消息序列化/反序列化
核心模块中的 StdioTransport(stdin/stdout)
专用传输模块(Java HttpClient、Spring WebFlux、Spring WebMVC)中的 HTTP SSE 传输

**MCP 客户端是模型上下文协议(MCP)架构中的关键组件，负责建立和管理与 MCP 服务器的连接。它实现协议的客户端。**
![image](https://github.com/user-attachments/assets/3a7a60dc-cd08-478d-b131-3d76ae55e8c3)


**MCP 服务器是模型上下文协议(MCP)架构中的基础组件，用于为客户提供工具、资源和功能。它实现协议的服务器端。**
![image](https://github.com/user-attachments/assets/91dacd76-9f72-4e39-bfbb-f0b7cac98080)


### 4.2、JAVA构建的 MCP Server
MCP 服务器可以提供三种主要类型的功能:
**1.Resources**: 资源。客户端可以读取的类似文件的数据(如 API 响应或文件内容)。
**2.Tools: 工具**。可由 LLM(经用户批准)调用的函数。
**3.Prompts**:提示。帮助用户完成特定任务的预先编写的摸板
由于 Spring A| 已经同步支持了 MCP，因此我们这里使用 Spring AI MCP 自动换配来快速入门。环境要求:Java 17+，Spring Boo
3.3.X+。

### 4.3 MCP Client 调用
#### 4.3.1、Cherry Studio MCP服务调用
https://cherry-ai.com/download
![image](https://github.com/user-attachments/assets/1cc703c1-3fce-4dd4-b8bc-a42c6f9407f3)

![image](https://github.com/user-attachments/assets/6b4a7137-a574-4e4c-9149-3c775408d238)

![image](https://github.com/user-attachments/assets/0d9b7230-d855-46e2-8df2-4cbcc643b57b)
在对话框输入: 调用mcp weather 服务的 getWeatherForecastByLocation 工具，经纬度参数如下，结果请用中文输出"latitude",“47.6062","longitude",“-122.3321'
调用成功结果
![image](https://github.com/user-attachments/assets/0478265a-09bf-49da-bc60-308ec403ed94)



我们可以使用一些桌面工县如 Claude Desktop、Cursor 等来引入我们的 MCP Server。我自己试了下 Claude，但是 Claude 需要国外的手机号进行注册才行，咋没这个条件啊。又试了下Cursor，发现这货只支持 Http SSE 方式。我这里只是为了做演示，所以就算了，不想封装了。 因此这里我直接编写一个 MCP Client 来做测试。
我们创建一个 MCP Client 工程:mcp-client-spring-ai，引入 maven 依赖:


