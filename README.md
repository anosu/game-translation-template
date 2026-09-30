# Game Translation Template

不绑定特定游戏的 **Codex Agent 翻译模板**。适配器提取真实原文，框架保存快照、找出缺失字典键并校验发布；Agent 自行阅读上下文、翻译和审校。框架不截断原文，也不替 Agent 管理上下文或拆分会话。

交付格式始终是**精确原文 → 译文**字典。VN 对话保留台词顺序和说话人供 Agent 理解语境；说话人、标题或摘要不会凭空生成。名称、剧情、标题和不同表字段可以分别交付，也不要求每款游戏都有这些内容。

## 运行示例

需要 Python 3.14+、uv 和 Node.js 22.18+ 或 24+。从仓库根目录运行：

```sh
uv sync --frozen
npm ci
npm run workflow -- sync
npm run workflow -- plan
```

默认使用 [game/translation.toml](game/translation.toml) 和本地 [JSON 示例原文](game/sources/resources.json)：2 份资源、3 条待译项。运行这些命令不需要后端 API 密钥；示例的 `sync` 只读取本地文件，`plan` 只检查快照与已有译文。换成自定义适配器后，`sync` 可能访问游戏服务器，获取范围和成本由适配器决定。

## 接入自己的游戏

1. 编辑 [game/translation.toml](game/translation.toml)：设置项目 ID、源语言、目标语言及交付目录。单游戏项目直接使用 `game/` 即可。
2. 将真实原文提取为 `dialogue`（有序台词）或 `text`（分组字符串）资源。现成 JSON 可替换 [示例资源](game/sources/resources.json)；需要下载、解密或解析游戏文件时，实现一个 [`collect(request)` 适配器](docs/adapters.md)。不需要补造原文不存在的字段。
3. 按需编写 [翻译风格](game/styles/zh-Hans.md)、[术语表](docs/glossary.md)；若 `names.json` 等已有字典应作为标准译名，在目标语言配置中声明 `term_sources`。
4. 再运行 `sync`、`plan` 检查提取范围与待译项。计划标出缺失的字典键及关联资源；已有译文不会因为重新规划而自动重翻。

JSON 输入可以是单个资源、资源数组或目录。资源的 `output` 和可选 `path` 决定字典位置：例如 `novels/001.json` 的根字典、`names.json` 的根字典，或 `master.json` 中 `mActionPatterns.name` 的嵌套字典。格式与边界见[资源契约](docs/adapters.md)；包含 names、完整剧情、多表 master 和独立标题的可运行示例见[这里](docs/examples/dictionaries/README.md)。

## 翻译与发布

在 `game/translation.toml` 中设置支持 Responses API 和工具调用的后端地址与模型，并把密钥放在 `api_key_env` 指定的**环境变量**中（示例为 `MODEL_API_KEY`）；不要写入仓库。然后运行：

```sh
npm run workflow -- translate
npm run workflow -- publish
npm run workflow -- check
```

`translate` 为每个目标语言启动一个 Agent 会话。Agent 从原文快照和已有字典按需读取，批量提交原文到译文；已校验答案保存在 Git 忽略的工作目录。再次运行 `translate` 会启动新会话并处理剩余任务，不恢复上次的对话。`publish` 合并到指定字典，`check` 验证结构和格式，但不能代替人工审校，也不保证远端原文最新或覆盖全游戏。需要一次执行全流程时使用 `npm run workflow -- update`。

GitHub Actions 的 [Update Translations](.github/workflows/update_translation.yml) 默认 `plan_only=true`，只同步和规划，不调用模型或发布。实际翻译须配置相应仓库 Secret、模型，并显式关闭 `plan_only`。可选的 `--limit` 只控制本次规划的资源范围，不限制 Agent 阅读原文或拆分会话。更多命令、恢复方式与 CI 配置见[使用手册](docs/usage.md)。

## 项目导航

| 路径 | 用途 |
| --- | --- |
| [game/](game/README.md) | 当前游戏的配置、示例原文、风格和术语 |
| [adapters/](adapters/README.md) | 内置 JSON 接入与自定义游戏适配器 |
| [translations/](translations/) | 按目标语言保存的交付字典 |
| [workflow/](workflow/) | 快照、规划、Agent 启动、校验与发布实现；参见[架构设计](docs/workflow-design.md) |
| [docs/](docs/) | [使用手册](docs/usage.md)、输入契约和完整示例 |
| [tests/](tests/) | 离线测试；开发检查为 `npm run lint`、`npm run typecheck`、`npm test` |

`.cache/` 存放原文快照和已保存的答案等工作状态，默认不提交。同仓库确需维护多个游戏时，再使用 `npm run workflow -- init projects/my-game --source-language ja --target zh-Hans,en`，并通过 `--config` 选择项目。
