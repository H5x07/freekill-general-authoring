# 本机环境

回链：[SKILL.md 开始前](../SKILL.md)。这张表记录作者本机上次确认的状态，**其他使用者请按自己的环境替换**；实际以各安装目录下的 `fk_ver` 实测为准。

## 已知安装

- `D:\FreeKill-v0.5.25\FreeKill-release`，版本 0.5.25，当前工作版本
- `E:\FreeKill-v0.5.20\FreeKill-release`，版本 0.5.20，旧版，不要混写

只改用户指定的版本。根目录不明确就先问，不要把文件写进另一份安装。

## 可用的工具

- 搜索：本机无 `rg`；`Get-ChildItem -Recurse | Select-String` 扫整个 `packages` 会长时间挂起，改用 `findstr`
  - 先 `Set-Location "<root>"` 再 `cmd /c "findstr /s /m /c:关键词 lua\*.lua"`；pwsh 里 `cd /d` 会报错
  - `findstr` 无匹配退出码为 1，`Command exited with code 1` 是“没搜到”不是命令失败
  - 限制到具体子目录，全库扫描也可能挂起
  - 按扩展名筛文件时 `-Include` 要求路径以 `\*` 结尾，否则**静默返回空**；空输出不能当“没搜到”下结论
- SQLite：本机没有 `sqlite3` 命令行，操作 `packages.db` 用 Python 的 `sqlite3` 模块
- Lua 解释器：本机没有独立的 `lua` / `luajit` / `luac`，语法检查用 Node 的 `luaparse`，见 [SKILL.md §10](../SKILL.md)
- Python：`$env:LOCALAPPDATA\Programs\Python\Python314\python.exe`（先 `Test-Path` 验证，找不到就停下问用户）。本 Skill 的 `scripts/check_package.py` 只需要 Python 3.10+ 标准库
- 游戏内测试：`& '.\FreeKill.exe' --testskills <skill_name>`，需要可用的游戏安装

## 写入与看图

- `apply_patch` 写不了 `D:\FreeKill...`（报 `Path escapes sandbox root`），改用 `edit` 或 `write`
- `edit` 的 `oldText` 必须全文唯一，多处重复要带上下文，唯一性失败时改用 `write` 整文件重写
- 看图工具只读工作区目录：要看包内原画，先复制到工作区再读

## 本机包的现状

- `ts` 与 `haoo_generals` 内容重叠（都注册 `ts__yuji`、`ts__jiekang` 与同名技能），`ts` 必须保持 `enabled=0`
- 新武将写进已在用的 `haoo_generals`，势力的内部 id 沿用该包已有的值，不要再造同义 id
- 不要靠改代码名或把重复包一起打开来绕开重名

## 协作约定

- 代码实现默认派给隔离执行者：任务里写死目标文件绝对路径、已拍板的规则口径（歧义先问清，最多三题）、必须自查的本机 API，以及“不要 git commit、不要动 packages.db”。子代理只交报告，回来后自己 `git diff` 逐行看，把结论落到源码 file:line
- “子代理说已通过”不算验收
