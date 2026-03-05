# DB Schema Documenter

根据数据库表结构信息，自动生成规范的库表结构文档并保存为 Markdown 格式。

## 功能特性

- 从 `information_schema` 读取表结构元数据
- 生成带目录、字段表格、主外键标注的 Markdown 文档
- 表备注为空时自动推断补充
- 人员/用户类字段自动推断关联到 users、admins、persons 等表
- 支持 MySQL / MariaDB

## 安装（Cursor）

1. 打开 **Cursor Settings**（`Ctrl+Shift+J` / `Cmd+Shift+J`）
2. 进入 **Rules** → **Project Rules**
3. 点击 **Add Rule**
4. 选择 **Remote Rule (GitHub)**
5. 输入本仓库 URL，例如：
   ```
   https://github.com/你的用户名/你的仓库
   ```
   若技能在子目录，使用：
   ```
   https://github.com/你的用户名/你的仓库/tree/main/skills/db-schema-documenter
   ```

## 使用方式

在 Cursor Agent 对话中：

- 输入 `/db-schema-documenter` 显式调用
- 或说「生成数据库文档」「输出库表结构」「写数据库说明文档」

按提示提供：数据库连接信息、目标数据库名、可选表名，即可生成 `{DB_Name}_schema_docs.md`。

## 许可证

MIT
