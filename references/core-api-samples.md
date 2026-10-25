# 常用 API 与本机样板索引

回链:[SKILL.md §3 查本机源码样板](../SKILL.md)。以下结论在 0.5.25 实测,每条都有本机样板。**API 与旧三国杀文档不同,任何方法调用前先在本机源码确认存在。**

卡牌/玩家的方法定义在 `packages\freekill-core\ltk\core\card.lua`、`player.lua` 和 `lua\fk_ex.lua`。例如卡牌点数是 `.number` 属性,没有 `getNumber()`(实踩过的坑)。

0.5.25 新核心的官方包位于 `packages/freekill-core/`;自己的拓展仍使用 `packages.<包名>`,不要把 `packages.freekill-core` 当作自己的前缀。

## 可靠样板

- `packages\freekill-core\standard\pkg\skills\yingzi.lua`:最小触发技
- `packages\freekill-core\standard\pkg\skills\jieyin.lua`:选牌、选目标、主动技
- `packages\freekill-core\standard\pkg\skills\jianxiong.lua`:带 `can_trigger` 的触发技
- `packages\shzl\pkg\mountain\skills\xiangle.lua`:`TargetConfirmed` 后令牌无效
- `packages\jsrg\pkg\beginning\skills\chaozheng.lua`、`shelun.lua`、`fayi.lua`:议事
- `packages\offline_new\pkg\zhencang\skills\queying.lua`:获得【影】并视为牌
- `packages\utility\utility.lua`:`U.Discussion`、`U.getShade`
- `packages\freekill-core\standard\aux_skills_dir\uncompulsory_invalidity.lua`:非锁定技失效

## 常用 API

- 蓄力技:`tags = { Skill.Charge }`,`local U = require "packages.utility.utility"` 后用 `U.skillCharged(player, 数值, 上限)` 改蓄力,变化事件 `U.SkillChargeChanged`,获得/失去技能用 `addAcquireEffect` / `addLoseEffect` 设初值。样板 `offline_new\pkg\piracy_e\skills\tuntian.lua`
- 无效化:单目标在 TargetConfirmed 里 `data.nullified = true`(`xiangle.lua`);整卡所有目标用 `data:setNullified()` 或往 `data.use.nullifiedTargets` 插目标(`gamemode\pkg\jiange_generals\skills\bashi.lua`);一张牌只询问一次可在 `data.use` 上挂自定义 Lua 字段做去重
- 强制用牌:`room:useVirtualCard(牌名, 子卡, from, to, skillName, 不计次数)`(第5参写入 card.skillName,不做距离校验,样板 `ol\pkg\menfa\skills\fenjian.lua`);`room:useCard{ from=, tos=, card= }`(`lijian.lua`)
- 无距离限制:`addEffect("targetmod", { bypass_distances = ... })`(`qicai.lua`);攻击范围判断 `player:inMyAttackRange(p)`
- 濒死与翻面:`fk.EnterDying`、`fk.AfterDying`(`wansha.lua`、`juejing.lua`);翻面事件 `fk.TurnedOver`
- 牌上标记:`room:setCardMark(Fk:getCardById(id), 标记名, 值)`,读 `card:getMark(标记名)`,跨区域持久(`brainhole_new\pkg\brainhole\skills\nuha.lua`);玩家表标记 `room:addTableMark` / `player:getTableMark`
- table 扩展:FreeKill 用 `table.simpleClone`(util.lua),**没有 `table.copy`**;`table.filter/map/contains/insertIfNeed` 都存在
- 失去体力不想立刻进濒死结算(如“同时失去体力”后再统一结算):在 `fk.BeforeHpChanged` 触发里把 `data.preventDying = true`(样板 `shzl\pkg\wind\skills\buqu.lua` 不屈),濒死判定在 `BeforeHpChanged` 之后才执行
- 询问 API:`askToSkillInvoke`、`askToChoice`(`fanjian.lua`)、`askToChoosePlayers`(`tuxi.lua`)、`askToChooseCardsAndPlayers`(`yiji.lua`)、`askToDiscard`
- 常用结算:`room:loseHp(p, n, 技能名)`(`kurou.lua`)、`room:recover{ who=, num=, recoverBy=, skillName= }`(`rende.lua`)、`room:throwCard(牌, 技能名, who)`、`room:moveCardTo(牌, Player.Hand, 目标, fk.ReasonGive, 技能名, nil, false, player)`
- 伤害来源:`data.skillName`,虚拟牌再查 `data.card.skillName`(`zhuque.lua`、`kuixiang.lua`);**改牌技能(朱雀羽扇 `data:changeCard` 等)会覆写 `card.skillName`,但会继承 `skillNames` 列表并追加新名**——判断“牌来自某技能”用 `table.contains(card.skillNames, 技能名)` 最稳;在 `fk.DamageCaused` 里 `data:changeDamage(n)`(`luoyi.lua`)
- 弃置统计:`fk.AfterCardsMove` 里 `move.from == player and move.moveReason == fk.ReasonDiscard`(`offline_new\pkg\bgmdiy\skills\hantong.lua`)
- 技能交互下拉:`interaction = function(self, player) return UI.CardNameBox{ choices = {...} } end`,on_use 里 `self.interaction.data` 取值(`baihu.lua`);另有 `UI.ComboBox`、`UI.OptionBox`
- 自定义势力:翻译表加 `["内部名"] = "显示名"` 即可(`brainhole_new\pkg\brainhole\init.lua` 的 `n_pigeon`/“鸽”)
