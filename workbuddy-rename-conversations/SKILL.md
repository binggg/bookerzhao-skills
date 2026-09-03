---
name: workbuddy-rename-conversations
description: Batch rename WorkBuddy conversation titles for a project via the local workbuddy.db SQLite. Triggers on "重命名会话/改会话标题/rename conversations in project". Only touches sessions.custom_title; never touches project name, content, or other metadata.
description_zh: "批量重命名 WorkBuddy 项目会话标题"
description_en: "Batch rename WorkBuddy conversation titles"
---

# workbuddy-rename-conversations

## When to use
用户要求批量重命名某个 WorkBuddy 项目下的会话标题（如统一成 `MMDD｜TYPE｜Topic` 格式）。

## Steps
1. 数据源：`~/.workbuddy/workbuddy.db`（SQLite，WAL 模式，应用运行中也可读写）。
2. 只读查询会话列表（按 cwd 过滤项目，排除当前进行中的会话）：
   ```sql
   SELECT id, is_background_automation, title, custom_title,
          datetime(created_at/1000,'unixepoch','+8 hours') AS created_cst
   FROM sessions
   WHERE cwd LIKE '%/<项目目录名>' AND deleted_at IS NULL
   ORDER BY created_at DESC;
   ```
3. **日期取 `created_at` 转 Asia/Shanghai（+8h），绝不用 `updated_at`**。注意 created_at 是毫秒时间戳。
4. 先输出 Before/After 两列表格给用户确认，确认后才写库。用户未指定语言时默认英文 TYPE 码（FEA/DES/FIX/OPT/REL/EXP/DOC/RES），指定中文则用 功能/设计/修复/优化/发布/探索/文档/研究；格式 `MMDD｜TYPE｜Topic`（竖线用全角 `｜`）。Topic 不重复项目名；看不出主题的保留原标题。
5. 写库：**只改 `custom_title` 字段**（侧栏优先显示它，原 `title` 是 AI 生成的摘要，保留不动）。用 heredoc 事务，逐条按 `WHERE id='...'` 精确更新，一次提交：
   ```bash
   sqlite3 ~/.workbuddy/workbuddy.db <<'SQL'
   BEGIN;
   UPDATE sessions SET custom_title='0903｜优化｜xxx' WHERE id='<uuid>';
   ...
   COMMIT;
   SQL
   ```
6. 复查：再跑一次只读 SELECT 校验所有 custom_title 已生效。

## Pitfalls
- 后台自动化会话 `title` 为空、`custom_title` 存的是自动化名（如 "ATO Scheduler Tick"），同样走 custom_title 更新。
- 排除当前正在进行的会话（它往往就是本次重命名请求本身）。
- 同一天多个同名自动化会话会得到相同标题（如两条 `0802｜优化｜Scheduler tick`），属正常，因日期本来就相同。
- 不要更新 `title`、`deleted_at`、`status` 或任何其他字段；绝不碰项目名。

## Verification
更新后用只读 SELECT 输出 `title => custom_title` 对照，确认全部数量匹配计划表格。
