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

mcp/client.py文件

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
mcp/register.py

McpServerRegistry(类)：
                    方法：
                        load_and_connect_all(启动时读取持久化配置，重连所有 server)
                        add(添加mcp server)
                        remove(移除mcp server)
                        list_servers(列出已注册的mcp server)
                        _connect(连接mcp server并完成工具注册)
                        _load_raw_configs(加载配置文件)
                        _save(保存配置文件)

agent/tools/base.py

方法：
    normalize_tool_result(归一化工具结果)

Tool(类)：
        execute(执行工具，返回字符串结果)  抽象方法继承此类的类必须实现此方法
        validate_params(校验参数，返回错误列表（空列表表示校验通过）)
        _validate(递归校验值是否符合 schema，返回错误列表)
        to_schema(转换为 OpenAI function calling 格式)


                        
agent/tools/register.py

ToolRegistry(类)：
                set_context(设置当前会话上下文（channel、chat_id 等），供工具按需读取)
                get_context(获取当前会话上下文)
                register(完成工具注册)
                unregister(完成工具注销)
                has_tool(列出已注册工具名字)
                get_tool(获取指定名字的工具)
                get_registered_names(返回当前已注册工具名集合)
                get_schemas(返回 OpenAI function calling 格式的工具定义列表。names 为 None 时返回全量；否则只返回指定名称的工具)
                get_always_on_names(返回标记为 always_on 的工具名称集合)
                get_documents(返回所有已注册工具的索引文档列表)
                get_deferred_names(返回所有 deferred 工具名，按来源分组)
                execute(执行工具返回结果)
                get_schemas_as_doc_results(将工具名列表转为与 search() 相同格式的结果列表)
                get_mcp_server_names(返回当前已注册的所有 MCP server 名称)
                get_tool_names_by_source(返回指定来源的所有工具名)
                search(关键词搜索工具目录，返回匹配的工具信息列表)
                

agent/tool_runtime.py 

方法：
    prepare_toolset(将已有的工具列表转化为特定格式)
    build_tool_schemas(创建发送给LLM Provider的工具声明)
    build_tool_map(创建快速查找所需工具的字典)
    tool_call_signature(用于检测agent是否陷入死循环)
    format_tool_calls(工具格式化)
    append_assistant_tool_calls(将工具调用列表添加到对话消息中)
    append_tool_result(将工具调用回复添加到对话消息中)























