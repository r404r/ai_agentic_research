# Omnigent 零基础使用指南

> 本指南是本研究仓库（`ai_agentic_research`）的产出，面向研究对象 `omnigent` 编写。
> `omnigent` 是只读研究对象，本指南未对其仓库做任何修改；指南正文中的命令仅供用户在
> 自己的环境中使用。详见 [`AGENTS.md`](../AGENTS.md) 的项目铁则。

这份指南写给第一次接触命令行、编程工具和 AI agent 的用户。你不需要先理解
Python、Node.js、Git 或服务器概念；按步骤做即可。遇到看不懂的提示时，不要急着
跳过，先看本文的“常见问题和排错”。

## 你将完成什么

完成本指南后，你应该能做到：

- 在自己的电脑上安装 Omnigent。
- 打开 Omnigent 的网页界面。
- 连接一个可用的 AI 模型。
- 启动第一个 agent，并让它帮你处理一个简单任务。
- 知道如何停止、更新和排查常见问题。

## 先了解三个词

| 词 | 可以理解成 | 在 Omnigent 里的作用 |
| --- | --- | --- |
| Agent | 会执行任务的 AI 助手 | 你把任务交给它，它可以聊天、读文件、运行工具或调用其他 agent。 |
| Model | Agent 的大脑 | 例如 Claude、OpenAI、OpenRouter、Ollama 或其他兼容服务。 |
| Server / Web UI | 本机打开的网页控制台 | 默认地址是 `http://localhost:6767`，用于查看聊天、文件、终端和会话。 |

## 开始前准备

### 1. 准备一台电脑

推荐使用 macOS 或 Linux。Windows 用户建议先安装 WSL，然后在 WSL 的 Linux
终端中按 Linux 方式操作。

### 2. 打开终端

终端是输入命令的窗口。

- macOS：打开“终端”应用。
- Linux：打开系统自带的 Terminal。
- Windows + WSL：打开 Ubuntu 或你安装的 WSL 发行版。

本文中的灰色代码块都是要粘贴到终端里执行的命令。粘贴后按回车。

### 3. 准备一个模型账号

Omnigent 需要连接 AI 模型。任选一种即可：

- Anthropic API key。
- OpenAI API key。
- 已登录的 Claude Code CLI 或 Codex CLI。
- OpenRouter、LiteLLM、Ollama、Azure 等兼容 OpenAI 或 Anthropic 的网关。

如果你还没有这些账号，也可以先完成安装，等 `omnigent setup` 提示你配置模型时再处理。

## 第一步：安装 Omnigent

推荐使用一键安装脚本：

```bash
curl -fsSL https://raw.githubusercontent.com/omnigent-ai/omnigent/main/scripts/install_oss.sh | sh
```

安装过程中如果提示缺少工具，按提示确认安装即可。常见依赖包括：

- `uv`：安装和管理 Python 工具。
- `git`：下载代码和管理项目文件。
- Node.js 22 LTS 或更新版本：运行 Claude、Codex、Pi 等 coding harness。
- `tmux`：让 agent 的终端会话保持运行。
- Linux 上的 `bubblewrap`：给本机 agent 终端提供隔离沙箱。

安装完成后，确认命令可用：

```bash
omnigent --help
```

如果这个命令能显示帮助信息，说明安装成功。`omnigent` 也有一个短命令名 `omni`，
两者等价。

## 第二步：启动第一个本机会话

在终端输入：

```bash
omnigent
```

第一次运行时，Omnigent 会尝试发现你电脑里已有的模型登录信息，例如
`ANTHROPIC_API_KEY`、`OPENAI_API_KEY`、已登录的 `claude` CLI 或 `codex` CLI。
如果它让你选择模型或凭据，选择你已经准备好的那一项。

启动后你会看到一个本地网页地址，默认是：

```text
http://localhost:6767
```

把这个地址复制到浏览器打开。终端和浏览器显示的是同一个会话：你可以在终端聊天，
也可以在网页里继续。

## 第三步：如果需要，配置模型

如果第一次运行没有找到可用模型，执行：

```bash
omnigent setup
```

然后按提示添加凭据。你通常只需要知道三件事：

- 你要把凭据给哪个 agent 使用，例如 Claude、Codex 或 OpenAI agents。
- 你使用的是 API key、订阅登录、网关，还是 Databricks。
- 你的 key 或网关地址是什么。

常见选择：

| 场景 | 建议选择 |
| --- | --- |
| 有 OpenAI API key | 添加 OpenAI API key，给 Codex 或 OpenAI agents 使用。 |
| 有 Anthropic API key | 添加 Anthropic API key，给 Claude 使用。 |
| 有 OpenRouter | 添加 Gateway，并填写 OpenRouter 的 base URL 和 key。 |
| 本机运行 Ollama | 添加 Gateway，base URL 通常是 `http://localhost:11434/v1`。 |

配置完成后，再运行：

```bash
omnigent
```

## 第四步：让 agent 做一个简单任务

你可以先让它处理低风险任务，例如：

```text
请总结当前文件夹里有哪些文件，并告诉我哪些看起来像项目说明文档。
```

或：

```text
请帮我写一份三段式会议纪要模板，保存为 meeting-notes-template.md。
```

如果 agent 想读取文件、写文件或运行命令，Omnigent 可能会按策略要求你确认。看清楚它
要做什么，再选择允许或拒绝。

## 第五步：使用网页界面

本地网页界面通常在：

```text
http://localhost:6767
```

在网页里你可以：

- 查看当前聊天。
- 观察 agent 的工具调用和终端输出。
- 管理会话。
- 分享当前会话给同一网络里能访问你电脑的人。
- 在手机浏览器打开同一地址继续查看。手机需要和电脑在同一个网络里；如果
  `localhost` 不可用，需要使用电脑的局域网 IP 地址，例如 `http://192.168.x.x:6767`。

如果你想先启动网页服务，再从网页创建新会话，可以执行：

```bash
omnigent server start
omnigent host
```

然后打开 `http://localhost:6767`，点击 New Chat，并选择这台机器作为 host。

## 第六步：运行示例 agent

仓库里带了两个示例，适合熟悉 Omnigent 的工作方式：

```bash
omnigent run examples/polly/
omnigent run examples/debby/
```

- Polly：偏项目协作，会把任务分给多个 coding sub-agent。
- Debby：偏头脑风暴，会让两个不同风格的 agent 一起回答。

如果你是第一次使用，建议先从普通的 `omnigent` 命令开始；等熟悉后再运行示例。

## 第七步：停止和更新

停止本地服务和正在运行的后台组件：

```bash
omnigent stop
```

查看本地服务状态：

```bash
omnigent server status
```

更新 Omnigent：

```bash
omni upgrade
```

只检查是否有新版本：

```bash
omni upgrade --check
```

## 常见问题和排错

### 终端提示 `command not found: omnigent`

说明系统还找不到 Omnigent 命令。先关闭并重新打开终端，再试：

```bash
omnigent --help
```

如果仍然失败，重新执行安装命令，并留意安装脚本最后提示的 PATH 设置。

### 浏览器打不开 `http://localhost:6767`

按顺序检查：

1. 终端里的 `omnigent` 或 `omnigent server start` 是否仍在运行。
2. 执行 `omnigent server status` 查看服务状态。
3. 如果端口被占用，先执行 `omnigent stop`，再重新启动。

### 运行 agent 时提示没有模型或凭据

执行：

```bash
omnigent setup
```

添加至少一个可用模型凭据。添加后重新运行 `omnigent`。

### Linux 上提示缺少 `bubblewrap` 或 `bwrap`

Linux 本机 agent 终端需要 `bubblewrap` 提供隔离。使用系统包管理器安装，例如：

```bash
sudo apt install bubblewrap
```

安装后重新运行 Omnigent。

### 不确定 agent 要执行的命令是否安全

先拒绝，然后让 agent 解释这条命令会做什么、会修改哪些文件、是否会访问网络。确认
清楚后再重新允许。不要把不理解的命令直接粘贴到系统终端里执行。

## 下一步

- 想了解完整命令和功能：回到仓库根目录的 `README.md`。
- 想写自己的 agent：阅读 `docs/AGENT_YAML_SPEC.md`。
- 想控制 agent 的权限和花费：阅读 `docs/POLICIES.md`。
- 想部署到服务器或让手机在外网访问：阅读 `deploy/README.md`。
