Claude Desktop <-> MCP Server (本项目) <-> Kubernetes API Server
连接流程：
第一层连接（Claude Desktop 到 MCP Server）：
Claude Desktop 通过配置文件（claude_desktop_config.json）知道如何启动和连接 MCP Server
使用标准输入输出（STDIO）或 SSE（Server-Sent Events）进行通信
通信采用 MCP（Model Context Protocol）协议，这是一个专门设计用于 AI 助手和工具之间通信的协议
第二层连接（MCP Server 到 Kubernetes）：
MCP Server 使用 @kubernetes/client-node 库连接到 Kubernetes 集群
使用本地的 kubeconfig 配置（通常在 ~/.kube/config）来获取集群访问凭证
通过 Kubernetes API 与集群进行交互

具体实现：
让我们看看关键的代码部分：

const server = new Server(
  {
    name: serverConfig.name,
    version: serverConfig.version,
  },
  serverConfig
);

服务器使用 @modelcontextprotocol/sdk 创建一个 MCP 服务器实例。

sequenceDiagram
    participant CD as Claude Desktop
    participant MCP as MCP Server
    participant KM as KubernetesManager
    participant K8S as Kubernetes API

    CD->>MCP: 发送命令（通过STDIO/SSE）
    MCP->>KM: 解析命令并调用相应方法
    KM->>K8S: 使用 client-node 库调用 K8S API
    K8S-->>KM: 返回结果
    KM-->>MCP: 处理并格式化结果
    MCP-->>CD: 返回格式化的响应

    总结工作原理：
配置阶段：
Claude Desktop 读取配置文件，知道如何启动 MCP 服务器
MCP 服务器启动时加载 kubeconfig，建立与 K8s 集群的连接
运行时：
用户在 Claude Desktop 中输入命令
Claude Desktop 通过 STDIO/SSE 将命令发送给 MCP 服务器
MCP 服务器解析命令，使用 KubernetesManager 执行相应的 K8s API 调用
结果返回给 Claude Desktop 显示给用户
特点：
使用标准的 Kubernetes client-node 库
支持所有主要的 Kubernetes 操作（pods、deployments、services 等）
实现了资源跟踪和清理机制
支持端口转发等高级功能
这种架构使得 Claude Desktop 可以通过 MCP 协议安全地与 Kubernetes 集群交互，而不需要直接访问 Kubernetes API。同时，由于使用了标准的 Kubernetes 客户端库，确保了操作的可靠性和安全性。
