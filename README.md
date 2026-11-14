# freekill-general-authoring

新月杀([FreeKill](https://github.com/Qsgs-Fans/FreeKill))自制武将的 Agent Skill:让 Agent 把自然语言武将描述(或卡面图)落成**可加载、可实测**的 Lua 拓展,而不是只交出一堆"看起来能用"的文件。

## 它做什么

- **拆解技能规则**:把描述拆成触发时机、成本、效果、持续时间、边界情况,歧义处先问清再写码;“获得/失去/喝掉技能”这类状态机会先确认承载方式与显示位置
- **查本机源码样板**:在 `packages/` 中定位真实样板和 API 用法,不凭旧文档猜接口
- **编写技能与翻译**:套用三文件骨架(init.lua / 子包 / 技能文件),翻译键齐全;覆盖无实体牌视为技、酒限次、“未获得的技能”(相关技能 + 客户端钩子动态显隐)这类进阶形态
- **登记拓展管理**:写入 `packages.db` 并回填 git hash,让拓展能被禁用/启用(含管理页还原冲掉工作区的修复流程)
- **静态验收 + 游戏内验证**:lupa 语法解析、客户端/服务端 API 分界检查,并严格区分"文件已写入 / 静态验收通过 / 游戏内实测通过"三种完成状态

## 安装

把本目录复制到宿主的 skills 目录,例如:

```text
~/.zcode/skills/freekill-general-authoring/
```

或使用 skills CLI:

```bash
npx skills add Haoo-7/freekill-general-authoring
```

## 使用

对 Agent 说:

- "帮我做个武将,技能是……"
- "把这张武将卡图做成可玩的武将"
- "给现有拓展加一个新武将 / 改原画语音"

**不会**触发:修改 FreeKill 本体或官方包源码、无名杀等其他引擎的拓展、纯规则咨询、联机故障排查。

## 目录结构

```text
SKILL.md                          # 主流程:11 个步骤,每步有完成标准
references/package-template.md    # 最小三文件代码骨架(触发技/主动技/借用技能)
references/core-api-samples.md    # 0.5.25 实测的常用 API 与本机样板索引
references/local-environment.md   # 作者本机的安装记录(其他使用者请按自己环境修改)
```

## 兼容与边界

- 需要本机有可用的 FreeKill 安装;主流程版本无关,API 清单与骨架以 **0.5.25** 实测为准
- `local-environment.md` 记录的是作者机器的安装路径,拿到别处使用请改成自己的环境
- 修改 `packages.db` 前会先备份;静态校验通过不代表游戏内生效,以实测回报为准

## 致谢

结构按 [oil-skill-creator](https://github.com/oil-oil/oil-skill-creator) 的方法论整理与校验。
