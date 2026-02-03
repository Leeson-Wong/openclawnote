# OpenClaw 项目架构 - Mermaid 图表

生成时间：2026-01-31 00:53
进度：第5次汇报

## 项目概览

OpenClaw 是一个个人AI助手系统，核心特点是：
- 多渠道支持（WhatsApp, Telegram, Slack, Discord, Google Chat, Signal, iMessage等）
- Gateway控制平面 + Agent助手产品的架构
- 支持TTS（语音合成）、Canvas（画布）、浏览器控制等功能
- 支持本地运行，模型可配置（Anthropic, OpenAI等）

---

## 1. 整体架构图

```mermaid
flowchart TB
    subgraph UserInterface["用户界面层"]
        CLI["CLI (openclaw.mjs)"]
        TUI["TUI (终端界面)"]
        WebUI["Web UI"]
        NativeApp["Native App<br/>macOS/iOS/Android"]
    end

    subgraph Gateway["Gateway 控制平面"]
        GatewayServer["Gateway Server<br/>WebSocket/HTTP"]
        Config["配置管理"]
        SessionMgr["会话管理"]
        Cron["定时任务"]
        Channels["渠道注册表"]
        Protocol["协议层"]
    end

    subgraph AgentCore["Agent 核心"]
        AgentRunner["Agent Runner"]
        PI["PI Agent Core"]
        Models["模型管理"]
        Skills["技能系统"]
        Tools["工具系统"]
    end

    subgraph Channels["通信渠道"]
        WhatsApp["WhatsApp (Baileys)"]
        Telegram["Telegram (grammy)"]
        Slack["Slack"]
        Discord["Discord"]
        Signal["Signal"]
        iMessage["iMessage"]
        Line["Line"]
    end

    subgraph Features["功能模块"]
        TTS["TTS (语音合成)"]
        Canvas["Canvas (画布)"]
        Browser["Browser (浏览器)"]
        Media["媒体处理"]
        Memory["记忆系统"]
    end

    subgraph Infra["基础设施"]
        DB["数据库 (SQLite)"]
        Storage["文件存储"]
        Logging["日志系统"]
        Security["安全模块"]
    end

    UserInterface --> Gateway
    Gateway --> AgentCore
    Gateway --> Channels
    Gateway --> Features
    Gateway --> Infra

    AgentRunner --> PI
    AgentRunner --> Models
    AgentRunner --> Skills
    AgentRunner --> Tools

    Channels --> Gateway

    TTS --> Gateway
    Canvas --> Gateway
    Browser --> Gateway
    Media --> Gateway
    Memory --> Gateway

    Config --> Gateway
    SessionMgr --> Gateway
    Cron --> Gateway
    Channels --> Gateway
    Protocol --> Gateway

    Gateway --> DB
    Gateway --> Storage
    Gateway --> Logging
    Gateway --> Security
```

---

## 2. 源码目录结构图

```mermaid
flowchart TB
    subgraph src["src/ - 源码根目录"]
        subgraph Core["核心模块"]
            gateway["gateway/ - Gateway控制平面"]
            agents["agents/ - Agent核心逻辑"]
            plugins["plugins/ - 插件系统"]
            sessions["sessions/ - 会话管理"]
        end

        subgraph ChannelsModule["渠道模块"]
            channels["channels/ - 渠道核心"]
            whatsapp["whatsapp/ - WhatsApp"]
            telegram["telegram/ - Telegram"]
            slack["slack/ - Slack"]
            discord["discord/ - Discord"]
            signal["signal/ - Signal"]
            imessage["imessage/ - iMessage"]
            line["line/ - Line"]
        end

        subgraph Interface["界面模块"]
            cli["cli/ - CLI"]
            commands["commands/ - 命令"]
            tui["tui/ - TUI"]
            web["web/ - Web"]
            wizard["wizard/ - 向导"]
        end

        subgraph FeaturesModule["功能模块"]
            tts["tts/ - TTS"]
            browser["browser/ - 浏览器"]
            canvas_host["canvas-host/ - Canvas"]
            media["media/ - 媒体"]
            memory["memory/ - 记忆"]
            cron["cron/ - 定时任务"]
        end

        subgraph InfraModule["基础设施"]
            infra["infra/ - 基础设施"]
            config["config/ - 配置"]
            logging["logging/ - 日志"]
            security["security/ - 安全"]
            process["process/ - 进程"]
            hooks["hooks/ - Git hooks"]
        end

        subgraph Others["其他模块"]
            types["types/ - 类型定义"]
            utils["utils/ - 工具"]
            shared["shared/ - 共享"]
            providers["providers/ - 提供者"]
            docs["docs/ - 文档"]
            test_helpers["test-helpers/ - 测试助手"]
            test_utils["test-utils/ - 测试工具"]
            scripts["scripts/ - 脚本"]
        end
    end
```

---

## 3. 协议层架构图

```mermaid
flowchart TB
    subgraph Protocol["Gateway协议层"]
        subgraph ProtocolSchemas["协议Schema定义"]
            frames["frames.ts - 帧协议"]
            agent["agent.ts - Agent相关"]
            channels["channels.ts - 渠道相关"]
            config["config.ts - 配置相关"]
            cron["cron.ts - 定时任务"]
            devices["devices.ts - 设备配对"]
            sessions["sessions.ts - 会话相关"]
            nodes["nodes.ts - 节点相关"]
            exec_approvals["exec-approvals.ts - 执行审批"]
            snapshot["snapshot.ts - 快照"]
            wizard["wizard.ts - 向导"]
            logs_chat["logs-chat.ts - 聊天日志"]
            types["types.ts - 类型聚合"]
            protocol_schemas["protocol-schemas.ts - 协议聚合"]
            agents_models_skills["agents-models-skills.ts - Agent/模型/技能"]
            primitives["primitives.ts - 基础类型"]
            error_codes["error-codes.ts - 错误码"]
        end
    end
```

---

## 4. Skills系统架构图

```mermaid
flowchart TB
    subgraph Skills["Skills系统 (54个技能)"]
        subgraph Productivity["生产力工具"]
            notion["notion - Notion笔记"]
            obsidian["obsidian - Obsidian笔记"]
            apple_notes["apple-notes - Apple Notes"]
            apple_reminders["apple-reminders - Apple提醒事项"]
            blogwatcher["blogwatcher - 博客监控"]
            local_places["local_places - 本地地点"]
            food_order["food-order - 食物订购"]
            ordercli["ordercli - 订单CLI"]
        end

        subgraph Social["社交与通讯"]
            discord["discord - Discord"]
            imsg["imsg - iMessage"]
            slack["slack - Slack"]
            bluebubbles["bluebubbles - BlueBubbles"]
        end

        subgraph AIModels["AI模型"]
            gemini["gemini - Gemini"]
            openai_whisper["openai-whisper - OpenAI Whisper"]
            openai_whisper_api["openai-whisper-api - OpenAI Whisper API"]
            openai_image_gen["openai-image-gen - OpenAI图像生成"]
            oracle["oracle - Oracle"]
        end

        subgraph Coding["编程与开发"]
            github["github - GitHub"]
            coding_agent["coding-agent - 编码代理"]
            eightctl["eightctl - Eight控制"]
        end

        subgraph Media["媒体与内容"]
            nano_pdf["nano-pdf - Nano PDF"]
            canvas["canvas - Canvas"]
            camsnap["camsnap - 相机快照"]
            gifgrep["gifgrep - GIF搜索"]
        end

        subgraph System["系统与工具"]
            1password["1password - 1Password密码管理"]
            clawdhub["clawdhub - Clawd Hub"]
            model_usage["model-usage - 模型使用"]
            peekaboo["peekaboo - 暴露"]
            sag["sag - 语音"]
            sonoscli["sonoscli - Sonos控制"]
            trello["trello - Trello"]
        end

        subgraph Misc["其他"]
            bird["bird - Bird"]
            blucli["blucli - Blue CLI"]
            gog["gog - GOG"]
            goplaces["goplaces - 地点"]
            himalaya["himalaya - Himalaya"]
            mcporter["mcporter - MC Porter"]
            nano_banana_pro["nano-banana-pro - Nano Banana Pro"]
            openhue["openhue - Philips Hue"]
            spotify_player["spotify-player - Spotify播放器"]
            thingmac["thingmac - Thing Mac"]
            voice_call["voice-call - 语音通话"]
            webview["webview - Web视图"]
            workflow["workflow - 工作流"]
        end
    end
```

---

## 5. Extensions系统架构图

```mermaid
flowchart TB
    subgraph Extensions["Extensions系统 (渠道扩展)"]
        subgraph CoreChannels["核心渠道"]
            whatsapp["bluebubbles - BlueBubbles (WhatsApp)"]
            discord["discord - Discord"]
            slack["slack - Slack"]
            googlechat["googlechat - Google Chat"]
            line["line - Line"]
            signal["signal - Signal"]
            imessage["imessage - iMessage"]
            matrix["matrix - Matrix"]
            mattermost["mattermost - Mattermost"]
            msteams["msteams - Microsoft Teams"]
            nextcloud["nextcloud-talk - Nextcloud Talk"]
        end

        subgraph AuthProviders["认证提供者"]
            copilot_proxy["copilot-proxy - GitHub Copilot代理"]
            google_antigravity["google-antigravity-auth - Google Antigravity"]
            google_gemini["google-gemini-cli-auth - Google Gemini CLI"]
        end

        subgraph LLMTasks["LLM任务"]
            llm_task["llm-task - LLM任务处理"]
        end

        subgraph Monitoring["监控与诊断"]
            diagnostics_otel["diagnostics-otel - OpenTelemetry诊断"]
        end

        subgraph Memory["记忆系统"]
            memory_core["memory-core - 记忆核心"]
            memory_lancedb["memory-lancedb - LanceDB记忆"]
        end

        subgraph Other["其他"]
            lobster["lobster - Lobster"]
        end
    end
```

---

## 6. 数据流图

```mermaid
sequenceDiagram
    participant User as 用户
    participant Channel as 渠道<br/>(WhatsApp/Telegram等)
    participant Gateway as Gateway
    participant Agent as Agent核心
    participant Model as AI模型
    participant Tools as 工具系统

    User->>Channel: 发送消息
    Channel->>Gateway: 转发消息
    Gateway->>Agent: 处理请求
    Agent->>Model: 调用AI模型
    Model-->>Agent: 返回响应
    Agent->>Tools: 执行工具（如需要）
    Tools-->>Agent: 返回结果
    Agent-->>Gateway: 返回最终响应
    Gateway-->>Channel: 转发响应
    Channel-->>User: 显示消息

    Note over User,Channel: 支持多渠道并发
    Note over Agent,Tools: 支持工具调用（浏览器、TTS等）
```

---

## 7. Gateway核心文件结构图

```mermaid
flowchart TD
    subgraph GatewayCore["Gateway核心"]
        boot["boot.ts - 启动入口"]
        server["server.ts - 服务器"]
        probe["probe.ts - 探针"]
        config_reload["config-reload.ts - 配置重载"]
        control_ui["control-ui.ts - 控制UI"]
        control_ui_shared["control-ui-shared.ts - 控制UI共享"]
        openresponses_http["openresponses-http.ts - HTTP响应"]
        server_restart_sentinel["server-restart-sentinel.ts - 重启哨兵"]
        live_image_probe["live-image-probe.ts - 镜像探针"]

        subgraph ServerMethods["服务器方法"]
            server_methods_list["server-methods-list.ts"]
        end

        subgraph ServerSubscriptions["服务器订阅"]
            server_node_subscriptions["server-node-subscriptions.test.ts"]
            server_node_events_types["server-node-events-types.ts"]
        end

        subgraph ToolsInvoke["工具调用"]
            tools_invoke_http["tools-invoke-http.ts"]
        end
    end
```

---

## 8. Config模块文件结构图

```mermaid
flowchart TD
    subgraph ConfigModule["Config模块"]
        config["config.ts - 配置核心"]
        schema["schema.ts - 配置Schema"]
        paths["paths.ts - 路径配置"]
        defaults["defaults.ts - 默认配置"]
        validation["validation.ts - 配置验证"]

        subgraph Types["类型定义"]
            types_channels["types.channels.ts"]
            types_models["types.models.ts"]
            types_skills["types.skills.ts"]
            types_agents["types.agents.ts"]
            types_plugins["types.plugins.ts"]
            types_whatsapp["types.whatsapp.ts"]
            types_telegram["types.telegram.ts"]
            types_discord["types.discord.ts"]
            types_slack["types.slack.ts"]
            types_signal["types.signal.ts"]
            types_imessage["types.imessage.ts"]
            types_googlechat["types.googlechat.ts"]
            types_msteams["types.msteams.ts"]
            types_tts["types.tts.ts"]
            types_tools["types.tools.ts"]
            types_browser["types.browser.ts"]
            types_sandbox["types.sandbox.ts"]
            types_gateway["types.gateway.ts"]
            types_queue["types.queue.ts"]
            types_auth["types.auth.ts"]
            types_messages["types.messages.ts"]
            types_hooks["types.hooks.ts"]
            types_base["types.base.ts"]
            types_openclaw["types.openclaw.ts"]
        end

        subgraph ZodSchema["Zod Schema"]
            zod_schema["zod-schema.ts"]
            zod_schema_channels["zod-schema.channels.ts"]
            zod_schema_agents["zod-schema.agents.ts"]
            zod_schema_agent_runtime["zod-schema.agent-runtime.ts"]
            zod_schema_session["zod-schema.session.ts"]
            zod_schema_providers["zod-schema.providers.ts"]
            zod_schema_providers_core["zod-schema.providers-core.ts"]
            zod_schema_providers_whatsapp["zod-schema.providers-whatsapp.ts"]
            zod_schema_hooks["zod-schema.hooks.ts"]
            zod_schema_core["zod-schema.core.ts"]
            zod_schema_approvals["zod-schema.approvals.ts"]
            zod_schema_agent_defaults["zod-schema.agent-defaults.ts"]
        end

        subgraph Helpers["辅助功能"]
            merge_config["merge-config.ts"]
            merge_patch["merge-patch.ts"]
            includes["includes.ts"]
            normalize_paths["normalize-paths.ts"]
            env_substitution["env-substitution.ts"]
            cache_utils["cache-utils.ts"]
        end

        subgraph Legacy["遗留支持"]
            legacy["legacy.ts"]
            legacy_shared["legacy.shared.ts"]
            legacy_migrate["legacy-migrate.ts"]
            legacy_migrations["legacy.migrations.ts"]
            legacy_rules["legacy.rules.ts"]
        end
    end
```

---

## 9. Sessions模块文件结构图

```mermaid
flowchart TD
    subgraph SessionsModule["Sessions模块"]
        store["store.ts - 会话存储"]
        metadata["metadata.ts - 元数据管理"]
        main_session["main-session.ts - 主会话"]
        session_key["session-key.ts - 会话键"]
        types["types.ts - 类型定义"]
        reset["reset.ts - 重置"]
        transcript["transcript.ts - 转录"]
        transcript_events["transcript-events.ts - 转录事件"]

        subgraph Helpers["辅助功能"]
            session_key_utils["session-key-utils.ts"]
            session_label["session-label.ts"]
            level_overrides["level-overrides.ts"]
            model_overrides["model-overrides.ts"]
        end

        subgraph Policy["策略"]
            send_policy["send-policy.ts - 发送策略"]
        end

        subgraph IO["IO操作"]
            io["io.ts"]
            sessions["sessions.ts"]
        end
    end
```

---

## 10. 部署架构图

```mermaid
flowchart TB
    subgraph Deployment["部署架构"]
        subgraph UserDevice["用户设备"]
            MacOS["Mac设备"]
            IOS["iOS设备"]
            Android["Android设备"]
            Linux["Linux服务器"]
            Windows["Windows (WSL2)"]
        end

        subgraph Installation["安装方式"]
            NPM["NPM全局安装"]
            PNPM["PNPM全局安装"]
            BUN["Bun全局安装"]
            Source["源码编译"]
            Docker["Docker容器"]
            Nix["Nix包管理"]
        end

        subgraph Daemon["守护进程"]
            Launchd["launchd (macOS)"]
            Systemd["systemd (Linux)"]
            Service["Windows Service"]
        end

        subgraph Runtime["运行时"]
            Node22["Node.js ≥22"]
            TypeScript["TypeScript 5.9.3"]
            Pnpm["pnpm 10.23.0"]
        end

        UserDevice --> Installation
        Installation --> Daemon
        Daemon --> Runtime

        MacOS --> Launchd
        Linux --> Systemd
        Windows --> Service

        NPM --> Node22
        PNPM --> Node22
        BUN --> Node22
        Source --> Node22
        Docker --> Node22
        Nix --> Node22
    end
```

---

## 总结

已创建10个详细的Mermaid架构图，完整覆盖OpenClaw项目的各个方面：
1. 整体架构图 - 系统层级关系
2. 源码目录结构图 - 52个源码目录组织
3. 协议层架构图 - 17个协议Schema
4. Skills系统架构图 - 54个技能分类
5. Extensions系统架构图 - 20+个扩展分类
6. 数据流图 - 消息流转过程
7. Gateway核心文件结构图 - 核心模块文件
8. Config模块文件结构图 - 配置系统详细结构
9. Sessions模块文件结构图 - 会话管理系统
10. 部署架构图 - 多平台部署方案

所有架构文档已保存至 `/home/admin/clawd/openclaw-architecture.md`
