---
name: ios-android-diff-review
description: "iOS/Android 平台差异测试用例审查与补全。当用户需要对已有测试用例做 iOS/Android 双端差异 review、补全遗漏的平台专项场景、输出完整用例集时使用。支持直接粘贴用例文本，也支持从飞书文档读取用例并将结果写回文档。"
argument-hint: "[飞书文档 URL 或 token（可选，不填则从对话中读取用例）]"
metadata:
  requires:
    bins: ["lark-cli"]
---

# iOS/Android 差异测试用例审查与补全

> **前置条件：** 先阅读 [`../lark-shared/SKILL.md`](../lark-shared/SKILL.md)（认证与权限处理）。
> **核心分析逻辑：** 见本目录 [`prompts/diff-check.md`](prompts/diff-check.md)。

---

## 工作模式

本 Skill 支持两种输入方式，**优先使用飞书文档模式**：

| 模式 | 触发方式 | 说明 |
|------|---------|------|
| **文档模式** | 用户提供飞书文档 URL / token | 自动读取用例，结果可写回文档 |
| **文本模式** | 用户直接粘贴测试用例文本 | 直接分析，结果输出到对话 |

---

## 文档模式命令

### Step 1：读取飞书文档中的测试用例

```bash
# 读取文档全文（默认 XML 格式）
lark-cli docs +fetch --api-version v2 --doc "<文档URL或token>" --as user

# 若文档较大，仅读取正文内容（不含元数据）
lark-cli docs +fetch --api-version v2 --doc "<文档URL或token>" --as user --scope main-content
```

读取后，将文档内容交给 AI 按 `prompts/diff-check.md` 中的完整流程执行差异分析。

### Step 2：将补充用例追加写回飞书文档（可选，需用户确认）

```bash
# 将分析结果追加到原文档末尾（追加模式，不修改原有内容）
lark-cli docs +update --api-version v2 \
  --doc "<文档URL或token>" \
  --as user \
  --command append \
  --content "@<本地结果文件路径>.xml"
```

> **写回前必须确认：** 执行写回操作前，向用户展示补充用例摘要（共 X 条，覆盖规则 Y 条），获得用户明确确认后再执行。

### Step 3：（可选）在 Wiki 知识库中创建新节点存放完整结果

```bash
# 在指定 Wiki 节点下创建新文档
lark-cli wiki nodes create \
  --space-id "<space_id>" \
  --parent-node-token "<parent_token>" \
  --obj-type doc \
  --title "iOS/Android 差异补全用例 - <模块名> - $(date +%Y%m%d)"
```

---

## 完整编排流程

```
[输入] 飞书文档 URL / token
    ↓
Step 1: lark-cli docs +fetch → 获取用例原文
    ↓
[AI 分析] 按 prompts/diff-check.md 多轮差异检查
  → 第一轮：对照 R01–R82 规则库生成差异检查报告
  → 第一轮补充：生成符合 test-case-writing 标准的补充用例
  → 第二轮自检：检查遗漏 + 去重 + P0/P1 覆盖率
  → 迭代直到「新增遗漏：无」
    ↓
[确认] 向用户展示补充用例摘要，获得确认
    ↓
Step 2 (可选): lark-cli docs +update --command append → 写回飞书
    ↓
[输出] 差异检查报告 + 完整补充用例集
```

---

## 权限

| 操作 | 所需 scope | 身份 |
|------|-----------|------|
| 读取飞书文档 | `docx:document:readonly` | `--as user` |
| 写入/追加飞书文档 | `docx:document` | `--as user` |
| 读取 Wiki 节点 | `wiki:node:read` | `--as user` |
| 创建 Wiki 节点 | `wiki:node:create` | `--as user` |

---

## 输出格式

- 默认 **Markdown**（对话输出）
- 写回飞书时自动转换为 **XML**（飞书文档原生格式）
- 可额外请求：Excel 制表符 / CSV / JSON / TestRail 格式

---

## 快速使用示例

```
# 示例 1：文档模式
/ios-android-diff-review https://xxx.feishu.cn/docx/EXEpd8uqUoMs5FxyuWdcWzl1nNd

# 示例 2：文本模式（直接粘贴用例后触发）
/ios-android-diff-review
[粘贴你的测试用例内容]

# 示例 3：只分析不写回
/ios-android-diff-review <文档URL>
（分析完成后，告知 AI "不需要写回文档"）
```
