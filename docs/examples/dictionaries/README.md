# 名称、剧情、多表字典和标题

这是可直接运行的规范化资源示例，已经完成游戏资料到 Resource 的提取。原始脚本、表文件和标题记录的解析仍由游戏适配器负责。

从仓库根目录运行：

```sh
npm run workflow -- sync --config docs/examples/dictionaries/translation.toml
npm run workflow -- plan --config docs/examples/dictionaries/translation.toml
```

应得到 5 份资源、10 条待办。无需模型密钥。实际翻译前设置模型和密钥，再运行 translate、publish、check，并传同一个 --config。

| 资源 | 输出 |
| --- | --- |
| names | names.json 根部的原始名称 → 译名 |
| novel/1 | novels/1.json 根部的台词原文 → 译文 |
| mActionPatterns/name | master.json 内 mActionPatterns.name 下的原文 → 译文 |
| mActiveSkillSideEffectFilters/ml_name | 同一 master.json 内 mActiveSkillSideEffectFilters.ml_name[] 下的原文 → 译文 |
| titles | titles.json 根部的标题原文 → 译文 |

生成文件位于本示例的 generated/zh-Hans/。ml_name[] 是字面对象键，不是数组索引。标题确实存在才放入上下文；context 内的标题不自动翻译，因此需要交付时另有 titles 资源。

term_sources 配置声明 names.json 为标准名称来源，所以 names 资源会优先处理，无需再在每条名称上标注。适配器应汇总本次所选剧情中出现的所有非空名称并去重；框架按当前语言的 names.json 补齐新名称。历史名称不必重抓，也不复制到 glossary。

手工修改 names.json 后，后续重新规划的剧情会使用新译法。glossary 可保存补充术语或名称说明，例如 {"村人":{"note":"普通角色称呼"}}。没有术语需要补充时不必创建 glossary 文件。

可将自己的 collect(request) 接入相同输出格式：消息命令提取为 dialogue.lines；可选 title 命令和标题记录提取为独立 titles 资源；指定表字段的字符串提取为相应 path 的 text 资源。哪些命令和字段需要提取由游戏规则决定。
