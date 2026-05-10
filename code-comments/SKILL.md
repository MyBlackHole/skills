---
name: code-comments
description: Use when adding Chinese annotation comments to code, or when asked to translate/add supplementary comments in Chinese to existing code
---

# Adding Chinese Code Comments (代码备注)

## Overview

Adding Chinese comments **directly into the original source files** as **supplementary** annotations — editing the file in-place, not creating a separate annotation file or copy. The original English comments are the authoritative documentation — Chinese comments are auxiliary translations/explanations that must **never** replace them.

## Core Rules

1. **Add comment blocks BEFORE important functions/classes/types** — a Chinese summary block above the declaration
2. **Add comment lines BEFORE key code segments** — a Chinese explanation line before the code block it describes
3. **PRESERVE all original English comments** — do NOT remove, modify, or overwrite them. They stay exactly as-is.
4. **Chinese comments are SUPPLEMENTARY only** — added alongside, never replacing, the original English
5. **EDIT THE ORIGINAL FILE IN-PLACE** — do not create a separate annotation file, a copy, or a new file. Modify the original source file directly.
6. **Do NOT change any code** — comments-only edits
7. **Use `// 备注:` as unified Chinese prefix** — keeps format consistent and searchable across the codebase

## Comment Formats

**Prefix convention**: All Chinese annotations use `// 备注:` as a unified prefix.
Exception: File headers use `// =====` separator style.

### File header
For complex files, add at the very top of the file:

```
// ============================================
// 文件说明：用户认证相关中间件
// 包含：JWT 验证、角色权限检查、登录状态维护
// ============================================
// original English header comment — DO NOT TOUCH
```

### Function-level comment block
Place BEFORE the function declaration:

```
// 备注：处理用户数据，验证并规范化邮箱，计算活跃度得分
// 备注：@param user - 用户对象，需包含 email 字段
// 备注：@returns 标准化用户数据对象
// original English comment — DO NOT TOUCH
function existingFunction(param) {
```

### Class / Interface / Type comment block
Place BEFORE the class/interface/type declaration:

```
// 备注：用户数据仓储层
// 备注：负责用户数据的 CRUD 操作及缓存管理
// 备注：依赖 RedisService, DatabaseService
// original English comment — DO NOT TOUCH
class UserRepository {
```

### Code segment comment line
Place BEFORE the code line/key segment — Chinese on its own line, adjacent to the original English:

```
  // 备注：验证用户输入，确保 user 对象及其 email 属性存在
  // original English comment — DO NOT TOUCH
  const result = someOperation();
```

**Key rule**: Chinese and English must be on separate lines. The order of adjacent lines (Chinese above English or English above Chinese) is flexible — both are acceptable.

## When NOT to Add Chinese Comments

Avoid adding `// 备注:` in these cases — the English is already sufficient:

| Situation | Example | Why Skip |
|-----------|---------|----------|
| Self-explanatory one-liners | `count++`, `flag = !flag` | Code already reads itself |
| Simple getter/setter | `get name() { return this._name; }` | Obvious from signature |
| Trivial type conversion | `String(value)`, `Number(str)` | Type name says it all |
| Existing English already clear | `// Check if user is admin` | English explains it sufficiently |
| Boilerplate / framework hooks | `useEffect`, `componentDidMount` | Framework conventions are standard |

**Rule of thumb**: If you have to think about whether it needs a Chinese note — it probably does. If it's immediately obvious — skip it.

## Common Mistakes

| Mistake | Why It's Wrong | Correct |
|---------|---------------|---------|
| Deleting `// Validate input` and replacing with `// 验证输入` | Original English comment lost | Keep `// Validate input`, add `// 验证输入` as a separate line |
| Merging `// Process user data` into a larger Chinese block | Original wording lost | Keep the original comment line, add Chinese block separately |
| Claiming "retained original comments" but they're gone | Self-deception — verify with explicit check | After editing, re-read the file and confirm every original comment still exists |
| Adding Chinese comment replacing an English one on the same line | Original comment overwritten | Keep original on its own line, add Chinese on a separate adjacent line |
| Creating a separate annotation file instead of editing the original | Comments detached from source code | Edit the original file directly — use Edit tool to insert Chinese comments inline |
| Over-annotating trivial code (`i++` → `// 备注：自增i`) | Noise that hurts readability | Skip obvious one-liners — English is sufficient |
| Using inconsistent Chinese prefix (mix of `// 备注:`, `// 说明:`, `// 中文:`) | Hard to search/filter later | Always use `// 备注:` — consistent and searchable |

## Red Flags

- **"I retained all original English comments"** — Did you actually verify? Re-read the file to confirm.
- **"The Chinese makes the English redundant"** — Doesn't matter. Keep ALL original comments.
- **"I merged them into a combined comment"** — Don't merge. Keep separate.
- **"The English comment is obvious, the Chinese replaces it"** — Preserve means preserve. Keep both.
- **"I'll create a separate annotation file to keep the originals clean"** — No. Edit the original file in-place. The whole point is having Chinese annotations in the same file as the code.
- **"Every line needs a Chinese note"** — No. Skip trivial one-liners, obvious getter/setters, and boilerplate.
- **"I'll use different prefixes for variety"** — No. Always `// 备注:` for consistency.

## Verification Checklist (MANDATORY)

After adding Chinese comments, ALWAYS verify:

- [ ] Re-read the file and confirm every original English comment is still present word-for-word
- [ ] Chinese comments are on separate lines, not replacing English ones
- [ ] No code was modified
- [ ] Function-level blocks are before the function declaration, not inside it
- [ ] Code segment comments are immediately before the relevant code
