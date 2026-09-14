# 当前游戏：按此顺序配置

1. translation.toml：游戏 ID、源语言、目标语言和模型后端。
2. sources/resources.json：真实原文资源，格式见[契约](../docs/adapters.md)。
3. styles/<语言>.md：语气、人物口吻和界面要求。
4. glossary/<语言>.json：显式术语译法，规则见[说明](../docs/glossary.md)。

从仓库根目录运行 sync 获取资源，再运行 plan 离线查看待办。配置模型密钥后执行 translate、publish 和 check，或用 update 执行完整更新。

默认输出为 ../translations/zh-Hans。project.cache 统一设置缓存根目录，框架按项目 ID 和目标语言自动隔离；修改 ID 无需再改缓存路径。新增语言只需添加目标配置，按需指定输出位置。

适配器负责提供真实材料；原文没出现的标题、摘要和人物信息不需要补造。框架提示词位于 workflow/prompts/，日常风格修改放在这里的 styles/。
