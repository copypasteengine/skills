[README.md](https://github.com/user-attachments/files/25756426/README.md)
# Cursor Skills

个人 / 团队 Cursor Agent 技能集合。

## 技能列表

| 技能 | 说明 |
|------|------|
| [db-schema-documenter](db-schema-documenter/) | 根据数据库表结构自动生成规范的 Markdown 文档 |

## 安装方式

1. 打开 **Cursor Settings**（`Ctrl+Shift+J` / `Cmd+Shift+J`）
2. 进入 **Rules** → **Project Rules**
3. 点击 **Add Rule**
4. 选择 **Remote Rule (GitHub)**
5. 输入本仓库 URL：
   ```
   https://github.com/你的用户名/skills
   ```

安装后，仓库内所有技能将可供 Cursor Agent 使用。

## 使用技能

- 在 Agent 对话中输入 `/技能名` 显式调用
- 或直接描述需求，Agent 会根据上下文自动匹配相关技能

## License

MIT
