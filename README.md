# Game Translation Template

不绑定特定游戏的翻译工作流，支持 JSON 原文、自定义资源适配器、多语言翻译、校验与发布。

## 从哪里开始

**翻译自己的游戏，先看 [game/translation.toml](game/translation.toml)。** 修改项目名称和语言，再替换 [示例资源](game/sources/resources.json)，即可离线预览待办；实际翻译时再配置模型。VN 保留有序台词，其他文本按原始资料分组；最终在指定文件和字典路径下输出原文到译文。

| 你要做什么 | 看这里 |
| --- | --- |
| 配置游戏、放入原文、维护风格和术语 | [game/](game/README.md) |
| 运行翻译、恢复进度、配置 CI 或静态服务 | [使用手册](docs/usage.md) |
| 接入特殊资源格式或游戏服务器 | [适配器指南](adapters/README.md) |
| names、剧情、多表 master.json 和独立标题如何组合 | [完整示例](docs/examples/dictionaries/README.md) |
| 修改框架 | [架构设计](docs/workflow-design.md)，然后看 workflow/ |

## 先试跑

需要 Python 3.14+、uv 和 Node.js 22.18+ 或 24+。在仓库根目录运行：

```sh
uv sync --frozen
npm ci
npm run workflow -- sync
npm run workflow -- plan
```

默认读取 game/translation.toml，示例的 2 份资源应产生 3 条字典待办，无需模型密钥。sync 执行获取，plan 是不联网的预览。实际翻译前配置模型和密钥，再依次执行 translate、publish、check；update 是完整更新的快捷入口。

JSON 输入可用单份资源、资源数组或目录；通常无需配置资源 ID 或缓存路径。已有 dialogue/text 项目升级时，保留配置和译文，在原工作目录重新 plan、translate 即可。

## 目录只分六类

```text
game/           当前游戏：配置、原文、风格、术语
translations/   交付产物：按目标语言保存译文
adapters/       游戏接入：内置 JSON 与自定义适配器
workflow/       框架实现：命令、规划、翻译、发布及内部提示词
docs/           使用手册、设计、契约和示例
tests/          自动化测试
```

根目录其余文件是依赖、构建、CI 和静态服务配置，初次使用无需逐个阅读。临时状态在被 Git 忽略的 .cache/ 中。

需要同仓库维护多个游戏时，再使用 `npm run workflow -- init projects/my-game --source-language ja --target zh-Hans,en`，后续通过 `--config projects/my-game/translation.toml` 选择；单游戏项目直接修改 game/ 即可。
