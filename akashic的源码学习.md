# main.py的三个入口

## python main.py init
在init中完成一下任务：
①把config.toml文件中的配置加载进来
②创建保存记忆的文件mardown文件（ 待提取事实、近期上下文摘要、事件日志、自我认知、长期记忆、主动推送规则文件）
③创建json文件（MCP server 注册表、定时任务列表、信息源列表、表情包清单）
④创建技能文件目录（包括用户自定义 skill 目录和用户自定义 Drift skill 目录）
⑤创建数据库文件（包括会话存储、trace 数据库、proactive 状态数据库、proactive 配额）
⑥复制健康观察模板文件

## python main.py cil

作用：完成对现有服务器端连接

客户端类型：界面式CLI、纯文本式CLI

"""
    后端服务 / agent server
        ↑↓
     socket
        ↑↓
    CLI / TUI 界面
    socket是前端界面与后端服务连接的通道
    """

## 工具部分

McpClient(类)：
            属性：tool_infos
            方法：
                connect(启动子进程，完成握手，获取工具列表)
                call(调用远端工具，返回结果字符串)
                disconnect(终止子进程)
                _send(发送消息)
                _recv(接收消息)
                _drain_stderr(后台读取 stderr，防止缓冲区阻塞)
                _build_timeout_message(构建回复超时的消息的信息)
                _new_id
