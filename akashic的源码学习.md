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

agent/mcp/client.py文件

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
agent/mcp/register.py

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


agent/tool_bundles.py
    方法：
        build_readonly_research_tools(根据参数返回工具包)

agent/bootstrap/tools.py    

CoreRuntime(类)：
                方法：
                    start()
                    stop()
                    
方法：
    build_registered_tools()
    _build_loop_deps()
    _consolidate_and_save()
    build_core_runtime()

## 记忆部分

memory2/retriever.py

Retriever(类)：
            方法：
                retrieve(embed query → cosine search → 返回命中条目列表)
                embed(仅做 embedding，不触发 vector_search)
                retrieve_with_vec(复用已有 query_vec 做本地 vector_search，跳过 embedding 步骤)
                _select_injection_sections(1. 筛选条目 2. 按段落准备格式化文本)
                _select_for_injection(保留 _select_injection_sections返回的selected字段)
                build_injection_block(次流程：筛选条目 → 分段格式化 → 应用字符预算)
            
            
memory2/hyde_enhancer.py

HyDEEnhancer(类)：
                方法：
                    generate_hypothesis(生成假想记忆条目。失败/超时返回 None，调用方降级为原始检索)
                    _build_default_prompt(构建默认提示词)--静态方法
                    augment(双路检索并返回所有不同条目)
                    _union_dedup(留 raw 全部结果（含原始分数），追加 hyde 中 raw 没有的条目)
                
                
memory2/memorizer.py             

Memorizer(类)：
            方法：
                save_item(直接保存,不做任何去重和冲突处理)
                save_item_with_supersede(保存前先淘汰/合并旧条目)
                save_from_consolidation(将对话历史（短期窗口）中值得长期保留的信息提取出来，持久化到长期记忆库的过程)
                _merge_summary_text(对新旧摘要进行拼接)--静态方法
                _pick_explicit_merge_target(挑选指定相似度的聚合摘要)--静态方法
                merge_item(将更新后的摘要存入数据库)
                
            
memory2/post_response_worker.py

PostResponseMemoryWorker(类)：
                            方法：
                                _consume_budget(计算剩余token budget)--静态方法
                                _preview_text(格式化文本到固定长度)--静态方法
                                _collect_explicit_memorized(从 tool_chain 收集本轮 memorize tool 显式写入的 summary 和 DB id)
                                _handle_invalidations(检测用户明确指出 agent 旧行为有误的情况，无需替代规则即直接 supersede 旧条目)
                                _extract_invalidation_topics(从用户消息中提取被明确声明为有误/需废弃的 agent 行为主题)
                                _check_invalidate(用户声明旧行为有误时，判断哪些旧条目应被 supersede（无需新规则替代）)
                            


memory2/injection_planner.py




memory2/store.py

MemoryStore2(类)：
               方法：
                    _migrate_existing_to_vec(启动时将 memory_items 中尚未同步到 vec_items 的 embedding 迁移过去)
                    _vec_insert(向 vec_items 插入一条向量（幂等：先删再插）。维度不匹配时静默跳过)
                    _vec_delete(从 vec_items 批量删除)
                    upsert_item(写入或强化一条记忆。返回 'new:id' 或 'reinforced:id)
                    upsert_consolidation_event(原子写入 consolidation event：同一 source_ref 最多写一次)
                    has_consolidation_source_ref(获取指定ref的压缩记忆)
                     mark_superseded(将指定条目标记为已退休)
                     mark_superseded_batch(批量将指定条目标记为已退休)
                     get_items_by_ids()



core/memory/port.py

MemoryStore Memorizer Retriever的统一调用接口
MemoryPort(Protocol)：
                    方法：
                        read_long_term()
                        read_profile()
                        write_long_term()
                        read_self()
                        write_self()
                        read_recent_context()
                        write_recent_context()
                        read_pending()
                        append_pending()
                        append_pending_once()
                        snapshot_pending()
                        commit_pending_snapshot()
                        rollback_pending_snapshot()
                        append_history()
                        append_history_once()
                        read_history()
                        append_journal()
                        get_memory_context()
                        has_long_term_memory()
                        retrieve_related()
                        embed_query()
                        retrieve_related_vec()
                        build_injection_block()
                        save_item()
                        save_item_with_supersede()
                        save_from_consolidation()
                        reinforce_items_batch()
                        keyword_match_procedures()

MemoryStore Memorizer Retriever的适配器
DefaultMemoryPort(适配器)：
                        属性：
                            _v1_store--返回MemoryStore的对象
                        方法：
                            read_long_term()
                            read_profile()
                            write_long_term()
                            read_self()
                            write_self()
                            read_recent_context()
                            write_recent_context()
                            read_pending()
                            append_pending()
                            append_pending_once()
                            snapshot_pending()
                            commit_pending_snapshot()
                            rollback_pending_snapshot()
                            append_history()
                            append_history_once()
                            read_history()
                            append_journal()
                            get_memory_context()
                            has_long_term_memory()
                            retrieve_related()
                            embed_query()
                            retrieve_related_vec()
                            build_injection_block()
                            save_item()
                            save_item_with_supersede()
                            save_from_consolidation()
                            reinforce_items_batch()
                            keyword_match_procedures()
                        
                        
                    
                    















