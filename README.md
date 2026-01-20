# Session Logs

Claude Code 会话数据归档和反思系统。

## 目录结构

```
session-logs/
├── sessions/           # 原始会话数据
│   └── {year}/{month}/
│       ├── {day}-global.jsonl       # 全局历史
│       ├── {day}-{project}.jsonl    # 项目会话
│       └── {day}-metadata.json      # 当日元数据
│
├── summaries/          # 每日工作总结
│   └── {year}/{month}/
│       └── {day}-summary.md
│
├── reflections/        # 可检索的反思库
│   ├── _index.json     # 反思索引（用于快速检索）
│   ├── by-topic/       # 按主题分类
│   │   ├── testing.md
│   │   ├── debugging.md
│   │   ├── architecture.md
│   │   └── ...
│   ├── by-project/     # 按项目分类
│   │   └── {project-name}.md
│   └── by-date/        # 按日期分类
│       └── {year}-{month}-{day}.md
│
└── errors/             # 错误模式库
    ├── _index.json
    └── patterns/
        ├── typescript-errors.md
        ├── git-errors.md
        └── ...
```

## 相关 Skills

### 1. session-log
保存原始会话数据到 `sessions/` 目录。

**触发方式**:
- `/session-log`
- "保存会话日志"

**功能**:
- 从 `~/.claude/history.jsonl` 提取今日会话
- 数据脱敏处理
- 推送到 GitHub

### 2. daily-summary
分析会话数据，生成总结和反思。

**触发方式**:
- `/daily-summary`
- "总结今天的工作"

**前置条件**: 需先运行 `session-log`

**功能**:
- 生成 `summaries/` 每日总结
- 提取反思到 `reflections/`
- 更新 `errors/` 错误模式库

### 3. work-session (推荐)
智能编排器，自动调用底层 skills。

**触发方式**:
- `/work end` 或 "收工"

**功能**:
- 自动按顺序调用 `session-log` → `daily-summary`
- 汇总今日工作统计

## 使用流程

### 方式一：使用 work-session（推荐）

```bash
# 开始工作
/work start

# ... 工作中 ...

# 结束工作（自动保存会话 + 生成总结 + 生成反思）
/work end
```

### 方式二：手动调用

```bash
# 步骤 1: 保存会话数据
/session-log

# 步骤 2: 生成总结和反思
/daily-summary
```

## 数据格式

### 会话条目 (JSONL)
```json
{
  "type": "user|assistant|summary",
  "timestamp": "2026-01-19T09:13:08.031Z",
  "sessionId": "67f482fc-ba7e-48ef-afb6-f8aecb8d803a",
  "project": "~/project-name",
  "display": "消息内容"
}
```

### 反思索引 (_index.json)
```json
{
  "entries": [
    {
      "id": "ref-2026-01-19-001",
      "date": "2026-01-19",
      "title": "问题标题",
      "summary": "简短摘要",
      "topics": ["testing", "precision"],
      "projects": ["payment-kit"],
      "keywords": ["bn.js", "decimal"],
      "file": "by-topic/testing.md",
      "anchor": "2026-01-19---问题标题"
    }
  ]
}
```

## 隐私说明

- 所有会话数据在保存前会进行脱敏处理
- 过滤包含 `password`、`token`、`secret` 等敏感字段
- 路径中的用户名会被替换为 `~`

## 注意事项

1. **反思生成需要单独触发**: `session-log` 只保存原始数据，需要运行 `daily-summary` 才会生成反思
2. **建议使用 work-session**: 它会自动编排完整的收工流程
3. **前置条件**: `daily-summary` 依赖 `session-log` 已保存的数据
