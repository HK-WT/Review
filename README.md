# Review
Code Review Agent
# 🤖 AI Code Review Agent

基于 Claude 多 Agent 协作的自动化 PR 审查系统。

## 架构

```
PR 触发
  │
  ▼
┌─────────────────┐
│  解析 Agent      │  → 结构化拆解 Diff，识别模块与业务上下文
└────────┬────────┘
         │ parsed_context
         ▼
┌─────────────────────────────────────┐
│           审查 Agent（并行三路）       │
│  ┌──────────┐ ┌──────┐ ┌─────────┐ │
│  │ 规范检查 │ │ Bug  │ │ 安全扫描│ │
│  └──────────┘ └──────┘ └─────────┘ │
│         （复杂变更触发二次推理）        │
└────────────────┬────────────────────┘
                 │ review_results
                 ▼
         ┌──────────────┐
         │  总结 Agent  │  → 去重 + 分级 + 格式化
         └──────┬───────┘
                │
                ▼
      GitHub PR 评论（总体 + 行内）
```

## 快速开始

### 1. 配置环境变量

```bash
export ANTHROPIC_API_KEY="sk-ant-..."
export GITHUB_TOKEN="ghp_..."
export GITHUB_REPO="your-org/your-repo"
```

### 2. 安装依赖

```bash
pip install -r requirements.txt
```

### 3. 手动运行（Dry Run 预览）

```bash
# 只打印报告，不发布评论
python main.py --pr 42 --dry-run

# 正式运行，发布评论到 GitHub
python main.py --pr 42
```

### 4. 接入 GitHub Actions（自动触发）

将 `.github/workflows/ai_review.yml` 放入项目根目录。  
在仓库 Settings → Secrets 中添加 `ANTHROPIC_API_KEY`。  
之后每次 PR 创建或更新时自动触发审查。

## 问题分级说明

| 级别 | 标识 | 含义 |
|------|------|------|
| BLOCK | 🚫 | 必须修复才能合并，同时发布行内评论 |
| SUGGEST | ⚠️ | 强烈建议修复 |
| OPTIMIZE | 💡 | 可选优化，不影响合并 |

## CI 集成

`main.py` 在存在 BLOCK 问题时以 `exit(1)` 退出，可直接用于阻断 CI Pipeline。
