# 最小可用三文件骨架

把 `<包名>` / `<子包>` / `前缀` / 代码名替换成自己的。UTF-8 无 BOM、2 空格缩进、文件尾留空行。

## 1. `packages/<包名>/init.lua`

```lua
-- 拓展入口：只有 init.lua 会被自动加载
local prefix = "packages.<包名>.pkg."

local <子包> = require(prefix .. "<子包>")

Fk:loadTranslationTable {
  ["<包名>"] = "扩展显示名",
}

return {
  <子包>,
}
```

## 2. `packages/<包名>/pkg/<子包>/init.lua`

```lua
local extension = Package:new("<子包>")
extension.extensionName = "<包名>" -- 必须是 packages/ 下的文件夹名

extension:loadSkillSkelsByPath("./packages/<包名>/pkg/<子包>/skills")

Fk:loadTranslationTable {
  ["<子包>"] = "武将一览里的分组名",
}

-- General:new(所属包, 代码名, 势力, 体力, [体力上限], [性别])
General:new(extension, "<前缀>__<真名>", "shu", 4):addSkills { "<前缀>__<技能名>" }

Fk:loadTranslationTable {
  ["<前缀>"] = "角标字",          -- 显示在将面右下角
  ["<前缀>__<真名>"] = "武将名",
  ["#<前缀>__<真名>"] = "称号",
  ["designer:<前缀>__<真名>"] = "设计者",
  ["cv:<前缀>__<真名>"] = "配音",
  ["illustrator:<前缀>__<真名>"] = "画师",
  ["~<前缀>__<真名>"] = "阵亡台词",
}

return extension
```

## 3. `packages/<包名>/pkg/<子包>/skills/<前缀>__<技能名>.lua`

```lua
local skill = fk.CreateSkill {
  name = "<前缀>__<技能名>",
  tags = { Skill.Compulsory }, -- 锁定技；普通触发技去掉这一行
}

skill:addEffect(fk.DrawNCards, { -- 摸牌阶段摸牌时
  anim_type = "drawcard",
  on_use = function(self, event, target, player, data)
    data.n = data.n + 1
  end,
})

Fk:loadTranslationTable {
  ["<前缀>__<技能名>"] = "技能名",        -- ≤2 字
  [":<前缀>__<技能名>"] = "锁定技，摸牌阶段，你多摸一张牌。",
  ["$<前缀>__<技能名>1"] = "技能语音文本",  -- 音频放 audio/skill/<前缀>__<技能名>1.mp3
}

return skill
```

## 换成主动技（签名与触发技不同）

```lua
skill:addEffect("active", {
  anim_type = "support",
  prompt = "#<前缀>__<技能名>-active", -- 必须补这条翻译，否则提示为空
  max_phase_use_time = 1,
  card_filter = Util.FalseFunc,        -- 无牌消耗时这样写
  target_filter = function(self, player, to_select, selected)
    return #selected == 0 and to_select ~= player
  end,
  target_num = 1,
  on_use = function(self, room, effect)
    local from = effect.from
    local target = effect.tos[1]
    from:drawCards(1, skill.name)      -- 效果写在这里
  end,
})

Fk:loadTranslationTable {
  ["#<前缀>__<技能名>-active"] = "你可以摸一张牌。",
}
```

要点：`card_filter(self, player, to_select, selected)`、`target_filter(self, player, to_select, selected)`、`on_use(self, room, effect)`，取 `effect.from` / `effect.tos` / `effect.cards`。触发技则是 `can_trigger(self, event, target, player, data)` + `on_use(self, event, target, player, data)`。真实样板：`packages/freekill-core/standard/pkg/skills/jieyin.lua`（选牌+选目标）、`kurou.lua`（无牌主动技）。

## 触发技加可选条件

```lua
skill:addEffect(fk.Damaged, {
  anim_type = "masochism",
  can_trigger = function(self, event, target, player, data)
    return target == player and player:hasSkill(skill.name)
  end,
  on_use = function(self, event, target, player, data)
    player.room:obtainCard(player, data.card, true, fk.ReasonJustMove, player, skill.name)
  end,
})
```

样板：`packages/freekill-core/standard/pkg/skills/jianxiong.lua`。

## 借用现成技能

```lua
General:new(extension, "<前缀>__<真名>", "god", 5):addSkills { "wushuang", "<前缀>__<技能名>" }
```
