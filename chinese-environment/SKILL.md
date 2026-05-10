---
name: chinese-environment
description: Use when setting up a project for Chinese-speaking developers, or when all output (docs, comments, commits) should be in Chinese
---

# Chinese Environment (中文环境)

## Overview

Define the conventions for a Chinese-first development environment. All project-facing output uses Simplified Chinese, while the code itself stays in English. This skill declares **what** to do — code annotation details are delegated to the `code-comments` skill.

## Core Rules

1. **All AI → user communication in Simplified Chinese** — explanations, suggestions, code reviews, Q&A
2. **Project documentation in Chinese** — README, API docs, changelog, spec documents, design docs
3. **Code stays in English** — variable names, function names, class names, type names, API signatures remain English
4. **Code comments in Chinese** — use `// 备注:` prefix; **REQUIRED SUB-SKILL:** also load `code-comments` for detailed annotation format rules; preserve original English comments
5. **Commit messages in Chinese** — follow conventional commits format: `类型: 中文描述`
6. **Project locale: China standard** — `zh_CN.UTF-8`, timezone `Asia/Shanghai`, encoding `UTF-8`

## Dimension Examples

### Documentation
```
<!-- 好：文档用中文 -->
# 用户积分管理系统
用户积分管理系统是一个用于追踪和管理用户积分的 Web 应用。

<!-- 不好：文档用英文（除非项目明确要求英文文档） -->
# User Points Management System
A web application for tracking user points.
```

### Code Comments
```
<!-- code-comments skill 负责具体格式，这里只声明原则 -->

// 备注：计算用户积分
// 备注：根据发帖数、评论数、认证状态计算，上限 1000 分
function calculateScore(user) { ... }
```

### Commit Messages
```
fix: 修复用户积分计算溢出问题
feat: 添加积分排行榜功能
docs: 更新 API 使用文档
refactor: 重构积分计算逻辑
```

## Common Mistakes

| Mistake | Why It's Wrong | Correct |
|---------|---------------|---------|
| 文档用英文写 | 中文环境项目文档应面向中文读者 | 所有项目文档用中文 |
| Commit message 用英文 | 与中文环境不一致 | `fix: 修复XX问题` |
| 把代码也翻译成中文（变量名、函数名） | 混入中文命名降低代码可移植性 | 代码保持英文，仅注释和文档用中文 |
| 注释和文档用了中文但忘记引用 code-comments | 缺少具体注释格式指导 | 引用 `code-comments` 处理注释细节 |

## Red Flags

- **"I'll write docs in English since code is in English"** — No. Documentation audience is separate from code audience.
- **"Chinese variable names are fine for this project"** — No. Keep code language-agnostic. Only comments and docs in Chinese.
- **"I already replied in Chinese, that's enough"** — Not enough. Documentation, commit messages, and code comments must also follow the rules.
- **"I'll handle comments however I want"** — No. Use `code-comments` skill for consistent annotation format.

## Verification Checklist

- [ ] All documentation is in Simplified Chinese
- [ ] All code identifiers (variables, functions, classes, types) are in English
- [ ] Commit messages follow `类型: 中文描述` format
- [ ] Code comments reference `code-comments` skill or use `// 备注:` prefix
- [ ] Original English comments in code are preserved (per code-comments rules)
- [ ] locale and timezone are set to China standard
