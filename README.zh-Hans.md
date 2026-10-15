# hermes-agent-telegram-checklist

[English](README.md) · **简体中文**

> 本文译自英文 [README.md](https://github.com/bablobanov/hermes-agent-telegram-checklist/blob/bfa65546966471b3fbbc6967b0512069dc20d043/README.md)（提交 `bfa6554696`）。如有出入，以英文版为准；英文版更新后，本译文可能滞后。

**为 [Hermes Agent](https://github.com/NousResearch/hermes-agent) 提供 Telegram 原生清单（待办事项列表）**：通过 Telethon 用户会话，在聊天和话题中创建、读取、追加和勾选真正可交互的清单消息。

[![CI](https://github.com/bablobanov/hermes-agent-telegram-checklist/actions/workflows/ci.yml/badge.svg)](https://github.com/bablobanov/hermes-agent-telegram-checklist/actions/workflows/ci.yml)
![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)
![Python 3.9+](https://img.shields.io/badge/python-3.9%2B-blue.svg)
![Hermes skill](https://img.shields.io/badge/Hermes-skill-8A2BE2.svg)

![一个 Hermes 风格的机器人正在勾选 Telegram 原生待办清单里的事项](assets/hero.png)

## 这是什么

一个单文件的 Python CLI，用来管理 **Telegram 原生待办清单**：带可点选复选框和进度计数的真正交互式对象，而不是 Markdown 列表项。智能体（或者你自己在 shell 里）可以：

- 在收藏夹、白名单内的群组或指定话题中**创建**（create）清单
- 以结构化 JSON **读取**（get）清单：任务 id、完成标记、共享标记
- **追加**（append）任务，或把任务**切换**（toggle）为已完成 / 未完成
- 按文档中的滚动重建（roll-over）流程**重建**（rebuild）清单
- 在写入任何内容之前，离线校验由智能体调研得出的**计划**（plan，`plan.json`）

它作为 [Nous Research](https://nousresearch.com) 出品的 [Hermes Agent](https://github.com/NousResearch/hermes-agent) 的技能（skill）发布，附带驱动智能体的 [`SKILL.md`](SKILL.md)；但脚本本身不依赖任何框架：单个文件，只用标准库 + Telethon，输入输出都是严格的 JSON，任何智能体框架或普通 shell 都能调用。

## 为什么需要它：Bot API 做不到

Telegram Bot API 确实有 [`sendChecklist`](https://core.telegram.org/bots/api#sendchecklist) 和 `editMessageChecklist`，但它们只能*代表已连接的企业账号*调用，也就是只能用在 Telegram Business 账号与其客户的私聊里。普通机器人根本无法把清单发到群组或话题中。要在任意聊天或话题里创建原生待办清单，唯一的办法是以**用户会话**走 [MTProto](https://core.telegram.org/api/todo)，而这正是本技能借助 [Telethon](https://docs.telethon.dev) 所做的事。注意服务端的规则：**创建**清单要求执行操作的账号开通 Telegram Premium；读取和勾选完成则不需要。

## 功能

- `create`：直接创建（`--title` / `--task ...`），或基于校验通过的 `plan.json` 创建（`--from-plan`）
- `get`：以 JSON 读取任意清单消息：带 id 的任务、完成状态、共享标记
- `append`：向已有清单追加任务（带服务端上限保护和 id 溢出保护）
- `toggle`：一次调用把任务标记为已完成 / 未完成（`--done 2 --undone 1`）
- `list-topics`：列出群组的话题，方便智能体选对话题（如果该聊天是按话题加入白名单的，则只列出允许的话题）
- `plan`：对智能体调研得出的计划做完全离线的校验：目标必须在白名单内、任务文本中必须带有证据链接、查重、Telegram 限制
- 针对“重建清单”类请求的滚动重建流程（见 [`SKILL.md`](SKILL.md)）
- 纯按需调用的 CLI：没有守护进程，没有后台监听，不抓取消息
- 每次写入后都会从服务端重新读取，结果放在 `verified` 中：输出告诉你实际发生了什么，而不是原本打算做什么

## 演示

以下录自针对收藏夹的一次真实运行（`message_id` 已脱敏）：

```console
$ python3 telethon_checklist.py create --from-plan plan.json
{"ok": true, "kind": "checklist-create", "chat": "me", "thread": null, "message_id": 12345,
 "title": "OSS smoke", "tasks_created": 2,
 "verified": {"title": "OSS smoke", "total": 2, "done": 0,
              "others_can_append": false, "others_can_complete": false,
              "tasks": [{"id": 1, "text": "step one", "done": false},
                        {"id": 2, "text": "step two", "done": false}]},
 "warnings": []}

$ python3 telethon_checklist.py toggle --chat me --message-id 12345 --done 2
{"ok": true, "kind": "checklist-toggle", "chat": "me", "message_id": 12345,
 "marked_done": [2], "marked_undone": [],
 "verified": {"title": "OSS smoke", "total": 2, "done": 1, ...},
 "warnings": []}
```

清单在 Telegram 中显示为真正可交互的卡片：点一下复选框就能勾选，进度计数对聊天里的所有人同步更新。

## 环境要求

- Python 3.9+
- [Telethon](https://docs.telethon.dev) >= 1.44（首个包含 MTProto 待办类型的版本）
- 一个 Telegram 账号，以及在 [my.telegram.org](https://my.telegram.org) 获取的 API 凭据（`api_id` / `api_hash`）
- 执行操作的账号需开通 Telegram Premium **才能创建**清单（服务端限制；`get` / `toggle` 不需要）

## 安装 / 配置

1. 安装 Telethon：

   ```bash
   pip install "telethon>=1.44"
   ```

2. 在 [my.telegram.org](https://my.telegram.org) -> “API development tools” 中获取 `api_id` / `api_hash`。

3. 配置环境变量（也可以把同样的几行写进 `~/.hermes/.env`：脚本两处都会读取，环境变量优先）：

   | 变量 | 必填 | 含义 |
   |---|---|---|
   | `TELETHON_API_ID` | 是 | 数字形式的 API id |
   | `TELETHON_API_HASH` | 是 | API hash |
   | `TELETHON_SESSION` | 否 | 会话文件路径；默认为 `~/.hermes/telethon/user.session` |
   | `TELETHON_CHECKLIST_CHATS` | 否 | 收藏夹之外的聊天 / 话题白名单，每条命令都会检查（见下文） |
   | `HERMES_HOME` | 否 | 覆盖 `~/.hermes` |

4. 进行一次交互式登录，生成会话文件（技能本身严格非交互；如果会话未授权，它会干脆地拒绝执行）：

   ```python
   import asyncio, os
   from telethon import TelegramClient

   API_ID = int(os.environ["TELETHON_API_ID"])
   API_HASH = os.environ["TELETHON_API_HASH"]
   PATH = os.path.expanduser("~/.hermes/telethon/user")  # becomes user.session

   async def main():
       os.makedirs(os.path.dirname(PATH), exist_ok=True)
       client = TelegramClient(PATH, API_ID, API_HASH)
       await client.start()          # asks for phone + login code once
       me = await client.get_me()
       print("authorized as", me.username or me.id)
       await client.disconnect()

   asyncio.run(main())
   ```

5. 把允许技能访问的聊天加入白名单，读和写同样受限。收藏夹（`me`）始终允许；其他聊天都必须显式列出：

   ```bash
   # whole chat -1001234567890, plus chat -1009876543210 restricted to forum topic 33 only
   TELETHON_CHECKLIST_CHATS="-1001234567890,-1009876543210:33"
   ```

   只接受负数 id（群组 / 频道）：与个人用户的对话按设计不在支持范围内。话题 id 从 2 开始；General 话题不能单独按话题放行（要发到 General，请放行整个聊天并省略 `--thread`）。

## 用法

每条命令都只向 stdout 输出**一个 JSON 对象**，退出码为 `0`（`"ok": true`）或 `1`（`"ok": false, "error": ...`）。stderr 始终为空，永远不会出现 traceback。唯一面向人的例外是 `-h` / `--help`。

直接创建（任务由用户口述）：

```console
$ python3 telethon_checklist.py create --title "Groceries" --task "milk" --task "bread" --chat me
{"ok": true, "kind": "checklist-create", "chat": "me", "thread": null, "message_id": 12345,
 "title": "Groceries", "tasks_created": 2, "verified": {...}, "warnings": []}
```

读回清单：

```console
$ python3 telethon_checklist.py get --chat me --message-id 12345
{"ok": true, "kind": "checklist-get", "chat": "me", "message_id": 12345, "title": "Groceries",
 "done": 0, "total": 2,
 "tasks": [{"id": 1, "text": "milk", "done": false}, {"id": 2, "text": "bread", "done": false}],
 "others_can_append": false, "others_can_complete": false, "warnings": []}
```

勾选与追加：

```console
$ python3 telethon_checklist.py toggle --chat me --message-id 12345 --done 2
$ python3 telethon_checklist.py append --chat me --message-id 12345 --task "eggs"
```

列出白名单中某个群组的话题：

```console
$ TELETHON_CHECKLIST_CHATS="-1001234567890" python3 telethon_checklist.py list-topics --chat -1001234567890
{"ok": true, "kind": "list-topics", "chat": -1001234567890, "total": 3, "listed": 3,
 "topics": [{"id": 1, "title": "General", "closed": false},
            {"id": 2, "title": "Roadmap", "closed": false},
            {"id": 33, "title": "Ops", "closed": false}], "warnings": []}
```

发到某个话题：

```console
$ python3 telethon_checklist.py create --title "Sprint 12" --task "ship it" --chat -1001234567890 --thread 33
```

计划流程（用于“从聊天里收集任务”类请求：智能体负责调研，脚本负责校验）：

```console
$ python3 telethon_checklist.py plan --file plan.json          # fully offline validation
$ python3 telethon_checklist.py create --from-plan plan.json   # the only write, at the very end
```

`plan.json` 包含目标、任务以及每个任务的来源证据；设置 `"collected_from_chat": true` 时，脚本强制要求每条任务文本都包含一个直达来源的 `https://t.me/...` 链接，而且必须是完整 URL。完整的约定和示例见 [`SKILL.md`](SKILL.md)。

试运行（dry-run）的确切语义：

- `plan` 和 `create --dry-run` **完全离线**：根本不会创建 Telethon 客户端，因此“什么都没发送”由结构本身保证
- `append --dry-run` 和 `toggle --dry-run` 会通过网络**读取**线上的清单（基于真实的当前状态做校验），但不发送任何写操作
- 写命令在写入后会重新读取清单；请以 `verified` 为准，并查看 `warnings`。在服务端没有返回消息 id 的少数情况下，`message_id` 为 `null` 并附带明确的警告，此时写入很可能已经生效（重试前先去聊天里确认）

重建 / 滚动重建：原生待办事项创建后**无法编辑文本**。要重建清单：先 `get` 旧清单，把未完成的事项连同新事项一起带过去，`create` 一个新清单，然后可以选择在旧清单里对已迁移的事项执行 `toggle --done`。删除旧消息需要用户手动操作：脚本有意不提供删除功能。

## 作为 Hermes 技能使用

用 [skills.sh](https://skills.sh) CLI 一行安装（适用于 Claude Code 及其他兼容 Agent Skills 的智能体）：

```bash
npx skills add bablobanov/hermes-agent-telegram-checklist
```

也可以把文件夹直接放进智能体的技能目录（对 Hermes 来说，就是类似 `skills/telegram-checklist/` 这样的文件夹，里面放 `SKILL.md` 和 `telethon_checklist.py`）。[`SKILL.md`](SKILL.md) 是面向智能体的约定：触发短语（“建一个 Telegram 清单”“把这个聊天里的任务收集成清单”“勾选第 2 项”）、两种创建模式（口述与调研）、带逐项来源证据的 plan.json 调研约定、语义查重规则、滚动重建流程，以及核验职责。脚本是强制执行层：智能体无论哪里出错，CLI 都会按失败即拒绝（fail-closed）的原则拒绝执行，并如实返回 JSON 错误。

## 安全与负责任的使用

- **每条命令都检查白名单。** 收藏夹和 `TELETHON_CHECKLIST_CHATS` 之外的目标一律拒绝。聊天检查离线进行，早于任何网络访问和凭据读取。按话题的条目（`-100...:33`）对读和写同样生效：`get`、`append` 和 `toggle` 会拒绝位于允许话题之外的清单（拒绝信息不会透露它实际所在的话题），`list-topics` 只按 id 获取允许的话题，从不请求该聊天的其他话题。1.1.1 之前，`get` 和 `list-topics` 只检查聊天。
- **只处理清单。** 它能发出的写请求只有清单相关的几种（带待办负载的 `SendMedia`、`AppendTodoList`、`ToggleTodoCompleted`）。不发普通消息，不发私信，不发邀请，不抓取内容。
- **文件里不放密钥。** 凭据在运行时从环境变量或 `~/.hermes/.env` 读取，不会回显。仓库自带的 [`.gitignore`](.gitignore) 会把会话文件和 env 文件挡在 git 之外。
- **会话文件卫生。** 会话文件的权限会用 chmod 收紧到 `0600`（尽力而为，仅限 POSIX）。
- **用户账号要守规矩。** 它通过 MTProto 操作的是一个真实的用户账号。请按需、低频地使用，尊重 `FloodWaitError`（脚本会报出需要等待的时间，从不自动重试），并遵守 [Telegram 服务条款](https://telegram.org/tos)：滥用可能导致账号受限或被封禁。
- **提示词注入。** 标题和任务是来自用户请求和已核实来源的数据；智能体绝不能执行聊天消息里出现的指令。计划校验器在这方面有帮助：启用 `collected_from_chat` 时，每个任务的文本里都必须带上来源链接，出处始终可见。

报告漏洞：见 [SECURITY.md](SECURITY.md)。

## 工作原理

Telethon >= 1.44 自带 MTProto 待办类型。脚本构造 `TodoList` / `TodoItem` 对象（文本用 `TextWithEntities`），包装进 `InputMediaTodo`，再通过 `SendMediaRequest` 发送（发到话题时附带 `InputReplyToMessage`）。读取时从目标消息中解析 `MessageMediaToDo`；修改由 `AppendTodoListRequest` 和 `ToggleTodoCompletedRequest` 完成，`GetForumTopicsRequest` 用于列出话题。各项限制沿用服务端 [appConfig](https://core.telegram.org/api/config) 的默认值：30 个任务、标题 255、任务文本 200，均按 UTF-16 码元计算（一个 emoji 算 2 个）。这些限制在客户端强制执行，超限时如实报错，而不是悄悄截断。每次写入后都会重新读取清单，并与预期做比对；不一致之处以 `warnings` 呈现，绝不伪装成成功。

## 局限

- **用的是用户会话，不是机器人**：你操作的是真实账号；服务端要求有 Premium 才能创建清单
- **仅按需运行**：没有后台监听；技能只在被调用时才行动
- **不能删除，也不能改文本**：Telegram 不允许编辑待办事项的文本；重建走滚动重建流程，删除旧消息仍需手动
- **Telegram 上限**：每个清单 30 个任务，标题 255 个 UTF-16 码元，每个任务 200 个
- **话题列表**：`list-topics` 返回前 100 个话题（超出时会有警告提示）；服务端的 `total` 计数可能与列出的数量相差一个（Telegram 往往不把 General 话题算在内）。对于按话题加入白名单的聊天，只列出允许的话题；某个允许的 id 在该聊天中不存在时，会给出警告
- 仅支持群组和频道（负数 id）；用户之间的对话有意不在支持范围内

## 参与贡献

见 [CONTRIBUTING.md](CONTRIBUTING.md)。离线测试套件（150 个测试，无需联网，也无需安装 Telethon）用 `python3 test_telethon_checklist.py` 运行；针对收藏夹、可直接复制粘贴的真机冒烟测试见 [examples/quickstart.md](examples/quickstart.md)。

## 许可证

[MIT](LICENSE)。

## 免责声明

本项目与 Telegram、Nous Research、Anthropic 均无隶属、背书或其他关联。它通过 MTProto 自动操作一个普通的 Telegram **用户账号**；你有责任在 Telegram 服务条款和适用法律允许的范围内使用它。
