---
name: db-schema-documenter
description: 根据数据库表结构信息，自动生成规范的库表结构文档并保存为 Markdown 格式。当用户要求"生成数据库文档"、"输出库表结构"、"写数据库说明文档"或呼叫 DB_Schema_Documenter 时使用。
---

# DB Schema Documenter

根据数据库表结构元数据，生成格式规范的 Markdown 文档并保存为文件。

## ⚠️ 前置信息收集

触发本技能后，**必须先确认**以下信息（若用户初始指令中未提供）：

1. **数据库连接信息**：Host、Port、User、Password
2. **目标数据库名称 `DB_Name`**：【必填】
3. **目标数据表 `Table_Names`**：【选填，逗号分隔；不填则处理全部表】

**强制校验规则：**
- 若未提供 `DB_Name` → 拒绝执行，要求补充
- 若提供了多个数据库名 → 要求用户选择唯一一个
- 若未提供 `Table_Names` → 默认处理该库下全部表

**⚠️ 全量表时的提醒（必做）：**
当用户**未指定** `Table_Names` 时，**不得直接开始生成文档**。须先询问用户：**「确认生成全部，或请指定要排除/仅包含的表名」**，待用户确认后再执行后续步骤。

---

## 第一步：获取元数据

优先尝试直接连接数据库执行查询，若环境不支持则输出 SQL 让用户手动执行并回传结果。

**提取 SQL（适用于 MySQL / MariaDB）：**

```sql
SELECT
    t.TABLE_NAME,
    t.TABLE_COMMENT,
    c.COLUMN_NAME,
    c.COLUMN_TYPE,
    c.COLUMN_KEY,
    c.IS_NULLABLE,
    c.COLUMN_DEFAULT,
    c.COLUMN_COMMENT
FROM information_schema.TABLES t
JOIN information_schema.COLUMNS c
    ON t.TABLE_SCHEMA = c.TABLE_SCHEMA AND t.TABLE_NAME = c.TABLE_NAME
WHERE t.TABLE_SCHEMA = '{DB_Name}'
-- 若指定了表名，追加：AND t.TABLE_NAME IN ('table1','table2')
ORDER BY t.TABLE_NAME, c.ORDINAL_POSITION;
```

---

## 第二步：生成 Markdown

按以下结构严格排版，**一字不差**地遵循格式：

```markdown
# 📦 数据库结构文档: `{DB_Name}`
> 生成时间：{当前时间}

## 📑 目录
- [表名 (表注释)](#表名)
...

---

## {表名}
**表说明**: {TABLE_COMMENT，无则按「表备注自动补充」规则推断}

| 字段名 | 数据类型 | 主/外键 | 允许空 | 默认值 | 字段说明 |
| :--- | :--- | :---: | :---: | :--- | :--- |
| `字段名` | 数据类型 | 🔑 主键 / 🔗 外键/索引 / - | 是/否 | 默认值或 - | 字段说明 |

**💡 关联关系推断**:
（基于 xxx_id 推测关联；人员/用户类字段优先推断到 users、admins、persons、staff 等表；无明显关联则省略此块）

---
```

**字段转换规则：**
- `COLUMN_KEY = PRI` → `🔑 主键`
- `COLUMN_KEY = MUL` 或字段名含 `_id` 后缀 → `🔗 外键/索引`
- 其余 → `-`
- `IS_NULLABLE = YES` → `是`；`NO` → `否`
- `COLUMN_DEFAULT` 为 NULL → `-`

**表备注自动补充（当 TABLE_COMMENT 为空时）：**
根据表名推断并写入合适的表说明，例如：
- `user(s)`、`admin(s)` → 用户表 / 管理员表
- `order(s)`、`product(s)`、`goods` → 订单表 / 产品表 / 商品表
- `xxx_log`、`xxx_history` → xxx 日志表 / xxx 历史表
- `xxx_config`、`xxx_setting` → xxx 配置表
- `xxx_relation`、`xxx_map` → xxx 关联表
- 通用规则：将下划线转为空格、复数转单数语义，加「表」后缀；如 `order_items` → 订单明细表

**关联关系推断规则：**
- 人员/用户类字段（`user_id`、`admin_id`、`creator_id`、`create_by`、`update_by`、`operator_id`、`person_id`、`staff_id`、`owner_id` 等）→ 推断关联到 `users`、`admins`、`persons`、`staff` 等表中存在的对应表
- 优先匹配同库内表名：`users`、`user`、`admins`、`admin`、`persons`、`person`、`staff`、`employees`
- 输出格式示例：`user_id` → 关联 `users` 表；`create_by` → 关联 `users` 表（创建人）
- 其他 `xxx_id` 字段按常规推断到可能的 `xxx` 或 `xxxs` 表

---

## 第三步：保存文件

1. 文件命名：`{DB_Name}_schema_docs.md`
2. 使用写文件工具将完整 Markdown 内容保存到**当前工作目录**
3. 若无写文件权限 → 将全部内容输出在单个 `markdown` 代码块中，并提示用户手动复制保存

---

## 注意事项

- 每次仅处理一个数据库
- 目录锚点链接使用小写表名（GitHub Markdown 规范）
- 关联关系为推断，请用户自行核实
