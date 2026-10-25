---
name: "freekill-general-authoring"
description: "新月杀(FreeKill)自制武将拓展:把自然语言或卡面图描述落成可加载、可实测的 Lua 拓展,覆盖拆解技能规则、查本机源码样板、编写技能与翻译、处理原画语音素材、登记拓展管理并验证游戏内效果。用户要自制武将、做个技能、改原画/语音、做扩展包时使用。不用于:修改 FreeKill 本体或官方包源码;无名杀等其他三国杀引擎的拓展;只问规则不写代码的咨询;扩展包运行报错或联机故障排查。"
metadata:
  version: "1.0.0"
---

# 新月杀自制武将

工作目标是把自然语言武将描述落成可加载的 FreeKill 拓展。全程区分三种状态:文件已写入、静态验收通过、游戏内实测通过;只在用户实际回报游戏内通过后才称“已生效”。

## 1. 确认版本和项目根目录

先定位安装目录,读取 `<root>\fk_ver`,确认 `<root>\packages` 存在。本机已知安装见 [local-environment](references/local-environment.md),内容可能过时,以 `fk_ver` 实测为准。

只修改用户指定或当前确认的版本,不要把新旧两份安装混用。若项目根目录不明确,先问用户;不要把文件写进另一份安装。

完成标准:记录实际版本号、根目录和目标 `packages` 目录。

## 2. 先拆规则,再写代码

用户给的是卡面图片时,先用图像工具放大裁切,逐字确认技能名、势力徽章、体力、称号和技能原文;形近字(如 悬/谏、沈/沉、奖励/樊励)和繁体极易误读,势力徽章也是重灾区,拿不准的当面问用户。

把用户描述拆成以下清单:

- 触发时机:`fk.TurnStart`、`fk.TargetConfirmed`、`fk.RoundEnd` 等
- 主动、触发、锁定、视为、禁止、手牌上限、目标选择或失效效果
- 参与者与目标的强制/可选关系
- 牌名限制、次数限制、回合/轮次持续时间
- 标记的进入、离开、清除时机
- 伤害、无效、技能失效、移动牌的先后顺序
- 空城、死亡、无人参与、无结果、多目标等边界情况

自然语言中的“其”“你”“其他角色”要先绑定到具体对象。若规则仍有歧义,向用户问一个最小澄清问题;不要在代码里默默选择关键规则。

完成标准:每个技能都有触发、成本、效果、持续时间和边界行为。

## 3. 查本机源码样板

先在 `<root>\packages` 中搜索中文翻译或技能内部名,再读取同名 Lua。优先复用本机 API,不凭旧文档猜接口;**API 与旧三国杀文档不同,任何方法调用前先在本机源码确认存在。**

按需读取 [core-api-samples](references/core-api-samples.md):可靠样板文件索引、常用 API 清单和实踩过的坑。

完成标准:每个复杂效果都有本机样板,或明确记录没有现成实现。

## 4. 创建拓展结构

直接套用 [package-template](references/package-template.md) 的最小三文件骨架(`init.lua` / 子包 `init.lua` / 技能文件);素材目录 `image/generals/`、`audio/skill/`、`audio/death/` 按 §9 放置。

规则:

- 入口只 `require` 子包;子包设置 `extension.extensionName`,并调用 `loadSkillSkelsByPath` 加载技能目录
- 代码名必须全局唯一;`__` 前是角标前缀,后半段是真名,用于识别同名武将;已有官方武将时必须加前缀
- 一个子包建议不超过 40 名武将,避免 Lua 单文件局部变量限制

完成标准:入口、子包、技能目录和 `General:new` 均存在,技能名与文件中的 `name` 一致。

## 5. 把拓展登记进 Pacman

Lua 加载和游戏内“拓展包管理”是两套机制。

- `packages/<包名>/init.lua` 存在就能被 `ModManager:loadPackages()` 加载,武将一览可见
- 拓展管理页只读 `<root>\packages\packages.db` 的 `packages` 表(列:`name,url,hash,enabled`)
- 禁用后的包名会进 `GetDisabledPacks()`;`loadPackages()` 遇到这些目录名就跳过

只写 Lua 不登记 `packages.db` 时:武将能用,但管理页认不到,也无法禁用后再加入对方服务器。

建包后必须:

1. 备份 `packages.db`
2. `INSERT INTO packages(name,url,hash,enabled) VALUES ('<包名>','local://packages/<包名>','<40位hash>',1)`
3. 在包目录初始化 git 并提交,把 `git rev-parse HEAD` 写回 `hash`
4. 用 `SELECT name FROM packages WHERE name='<包名>'` 确认行存在

不要用 `git commit --allow-empty-on-error`(不存在)。房间里禁单个武将走大厅“武将一览”;拓展设置页只列卡牌包。要进别人服务器时,在拓展管理里禁用整包并重启。本机没有 sqlite3 命令行,操作 `packages.db` 用 Python 的 `sqlite3` 模块。

完成标准:`packages.db` 有该行;禁用后 `GetDisabledPacks()` 含该目录名。

## 6. 写 Lua

直接在确认的 FreeKill 根目录里写文件。写之前确定:

- 目标版本和绝对路径
- 需要新增的精确文件列表
- 武将属性、技能原文、已解决的规则解释
- 本机参考文件
- 禁止修改官方包、禁止创建无关文件、不要未经用户要求删除或覆盖已有拓展
- UTF-8 无 BOM、2 空格缩进、LF 结尾

写完先查磁盘再读文件,不接受只有口头报告的“完成”。有现成扩展时在已有包里追加武将;不要重复建空包。

完成标准:新增文件实际存在,路径与目标一致。

## 7. 客户端和服务端 API 分界

技能同时包含 `enabled_at_play`、`enabled_at_response`、`interaction` 等客户端回调,以及 `before_use`、`on_use` 等服务端结算回调。两侧不能混用 API。

客户端回调中:

- 不要使用 `player.room`
- 不要调用服务端专有的 `room:getOtherPlayers()`
- 使用 `Fk:currentRoom()`
- 当前角色使用 `Fk:currentRoom():getCurrent()`
- 存活角色使用 `Fk:currentRoom().alive_players`

例如客户端判断是否存在其他存活角色:

```lua
local room = Fk:currentRoom()
return table.find(room.alive_players, function(p)
  return p ~= player
end) ~= nil
```

服务端 `before_use` / `on_use` 中的 `player` 是 `ServerPlayer`,可以使用 `player.room`、`room:getOtherPlayers()`、`room:moveCardTo()` 等服务端接口。

完成标准:逐个检查客户端回调和服务端回调,确认没有把 `player.room` 或 `getOtherPlayers` 放进客户端路径。

## 8. 编写技能和翻译

触发技、主动技、可选条件、借用现成技能的代码骨架直接套用 [package-template](references/package-template.md),此处只列规则:

- 触发技的成本数据用 `event:setCostData` 保存,效果阶段用 `event:getCostData` 取回
- 主动技用 `addEffect("active", ...)`,区分 `card_filter`、`target_filter`、`on_use(room, effect)` 的参数
- 视为技要分别检查使用和响应入口
- “交给别人”与“把牌作为实体子卡使用”不同:交出后再视为牌时,优先参考 `addFakeSubcards` 与 `before_use`;不要让实体影牌被错误送进弃牌堆
- 修改伤害使用 `data:changeDamage(1)`,参考【裸衣】和【陷阵】
- 修改手牌上限使用 `addEffect("maxcards", { exclude_from = ... })`
- 非锁定技失效复用官方 `UncompulsoryInvalidity`,不要重写全局失效技能

翻译表必须覆盖武将名、称号、设计者/配音/画师、阵亡台词、技能名、技能描述和所有 prompt。技能名建议不超过 2 字(超过可能导致界面错位),称号建议不超过 7 字;锁定技提示不要手动加粗。

完成标准:所有技能文件 `return skill`,翻译键齐全,关键牌移动和伤害调用符合本机样板。

## 9. 原画和语音素材

原画路径:`packages/<包名>/image/generals/<代码名>.jpg`。当前官方将面常见尺寸为 250×292;收到其他比例的图片时,先检查构图,再裁切/缩放到该尺寸,保留脸部和关键道具。从整卡图裁原画时,把体力图标、势力徽章、技能文案区全部裁掉——这些是游戏运行时自己叠加的,烙进图里会重影。

语音路径:技能音放 `audio/skill/`,阵亡音放 `audio/death/`,文件名按技能和武将代码名匹配。不要修改官方素材;必要时使用 `resource_pak/<名字>/` 按原相对路径覆盖。

完成标准:素材路径、文件名、尺寸和编码已检查;没有把素材误放入另一份 FreeKill 安装。

## 10. 静态验收和错误修复

写完按顺序验收:

1. 递归列出拓展目录,确认所有预期文件存在,且没有无关新增文件
2. 检查 UTF-8 无 BOM、LF 结尾、2 空格缩进
3. 检查 `General:new`、`addSkills`、技能 `name` 和 `return skill` 的一致性
4. 搜索危险的客户端调用:`player.room`、`room:getOtherPlayers`、`room.current`
5. 复查关键状态的持续时间:`-turn`、`-phase`、`-round`、`-inhand`
6. 复查虚拟牌、实体子卡、`nullified`、`nullifiedTargets`、伤害修改和牌移动顺序
7. 确认 `packages.db` 已登记该目录名
8. 每个文件用真实 Lua 虚拟机做语法解析(`pip install lupa` 后对每个文件 `load()`),不要只靠肉眼读
9. 有报错时以最新堆栈为准,读取报错行前后代码和对应官方样板;不要只修第一处表面错误。修复后 grep 全包确认没有同款误用,重新 git 提交并把新 `git rev-parse HEAD` 写回 `packages.db` 的 `hash` 列(每轮改动都要,否则拓展管理页的 hash 与仓库脱节)

没有 Lua 解释器可用时(lupa 装不上),明确说未做语法解析;不能声称已经通过运行测试。

完成标准:静态检查无明显问题,或把具体剩余风险列给用户。

## 11. 游戏内验证

改完必须完整退出并重新启动正确版本的 `FreeKill.exe`;只关错误弹窗或热改不一定会重新加载 Lua。使用单机启动,开启自由选将,右键强制选择自制武将。开发测试最好使用独立客户端,避免调试拓展影响正常对局。

至少验证:

- 拓展管理能看到包名并能禁用/启用
- 武将一览能加载拓展和原画
- 每个技能能从正确时机触发
- 强制参与和可选参与角色符合技能文本
- 空城、死亡、无人参与、多目标、无结果和轮次边界不报错
- 视为使用/打出时牌移动位置正确
- 牌名次数限制、伤害增幅、无效和非锁定技失效的持续时间正确

完成标准:只在用户实际回报游戏内通过后,才称为“已生效”;否则称为“文件已写入/静态验收通过”。
