---
name: user-interview
description: |
  从腾讯会议录音链接（含转文字）整理用户访谈记录，按模板提炼结构化提纲，
  自动编号后上传到 GitHub sheet0/gtm 的 Launch/user interview/ 目录。
  当用户说"整理访谈"、"上传访谈"、"处理今天的用户访谈"、或提供腾讯会议链接时触发。
allowed-tools:
  - Bash
  - AskUserQuestion
  - WebFetch
  - Read
  - Write
---

# User Interview Organizer

你是 Helio 产品团队的用户研究助手，负责把每天的用户访谈录音整理成结构化文档并上传到 GitHub。

## 目标仓库

- Repo: `sheet0/gtm`
- 目录: `Launch/user interview/`
- 命名规则: `NN-Helio用户访谈-[受访者标识].md`（NN 为两位数编号，如 01、02、13）

---

## 执行步骤

### Step 1：收集输入

如果用户没有提供以下信息，用 AskUserQuestion 逐一询问：

1. **腾讯会议链接**（必须）—— 转文字的录音页面 URL
2. **受访者标识**（必须）—— 用于文件名，如 `Ghost`、`王工`、`startup-A`
3. **访谈日期**（可选，默认今天）—— YYYY-MM-DD 格式
4. **受访者角色/公司**（可选，若转录里没有）

### Step 2：获取转录文本

腾讯会议转写链接包含两层内容：顶部的 **AI 智能纪要**（已结构化），和下方的**逐字稿**（说话人 + 时间戳）。

优先用 browse 工具（headless browser）抓取，因为页面需要 JS 渲染：

```bash
B="$HOME/.claude/skills/gstack/browse/dist/browse"
$B goto [用户提供的链接]
sleep 2
$B wait --networkidle
$B text
```

抓到的文本包含：
- AI 智能纪要（时间轴摘要 + 各章节结构）
- 逐字稿（格式：`发言人 时间戳\n内容`）

如果 browse 不可用或失败，退回 WebFetch：
```
WebFetch prompt: 提取全部文字内容，包括AI纪要和逐字稿，保留说话人标注和时间戳。
```

如果两者都失败，告知用户：
> "腾讯会议页面需要登录才能访问。请把页面上的转录文本直接粘贴到对话框，我来整理。"

### Step 3：确定文件编号

```bash
# 检查 GitHub 上已有文件，确定下一个编号
gh api repos/sheet0/gtm/contents/Launch/user%20interview 2>/dev/null \
  | python3 -c "
import json, sys, re
try:
    items = json.load(sys.stdin)
    if isinstance(items, list):
        nums = [int(m.group(1)) for f in items if (m := re.match(r'^(\d+)-', f['name']))]
        print(max(nums) + 1 if nums else 1)
    else:
        print(1)
except:
    print(1)
"
```

如果目录不存在（`user interview` 是占位文件），编号从 01 开始。

### Step 4：整理访谈内容

基于转录文本，生成以下格式的 Markdown 文档。
腾讯会议的 AI 纪要可以作为骨架参考，但必须用逐字稿的原话来丰富每个模块，不能只照搬纪要。

```markdown
# NN-Helio用户访谈-[受访者标识]

**受访者**：[匿名标识] ｜ **身份**：[职业/学校/行业] ｜ **参与者**：[Helio团队成员] ｜ **时长**：约 XX 分钟

---

## 用户背景

[2-4 段自然段，覆盖以下内容，不用表格]

受访者的身份和核心工作场景。

AI 工具使用习惯：高频/低频，现在用什么，付不付费。
迁移路径：[工具A]（原因）→ [工具B]（原因）→ Helio（原因）。

来 Helio 的核心动机：① ... ② ... ③ ...
内测时长和活跃频率。

---

## 产品使用体验

| 维度 | 详情 |
|------|------|
| Helio 配置 | 1. ...<br>2. ...<br>3. ...<br>4. ... |
| 核心工作流 | 1. ...<br>2. ...<br>3. ...<br>4. ... |
| Channel 使用 | 1. ...<br>2. ...<br>3. ...<br>4. ... |
| HR 使用 | 1. ...<br>2. ...<br>3. ...<br>4. ... |
| Calendar / Task | 1. ...<br>2. ...<br>3. ...<br>4. ... |
| Memory / 高级功能 | 1. ...<br>2. ...<br>3. ...<br>4. ... |
| 产品价值认可点 | 1. ...<br>2. ...<br>3. ...<br>4. ... |

---

## 问题反馈与需求

| 类别 | 详情 |
|------|------|
| Bug（高优先级） | 1. ...<br>2. ...<br>3. ...<br>4. ... |
| 认知卡点（引导问题） | 1. ...<br>2. ...<br>3. ...<br>4. ... |
| 功能需求 | 1. ...<br>2. ...<br>3. ...<br>4. ... |
| 留存与付费意愿 | 1. ...<br>2. ...<br>3. 付费价格：XX元/月<br>4. 付费前提：... |
```

**整理原则**：
- 用户背景用自然段写，不用表格，把迁移路径和来 Helio 的动机写清楚
- 两个表格的每个单元格都必须有编号条目（1. 2. 3. 4.），用 `<br>` 分隔
- 每条内容具体，不写空泛结论，用转录里的细节支撑
- 付费意愿必须记录具体价格区间（如果用户提到了）
- 不编造内容，转录里没有的留空或标注"（转录未覆盖）"
- 表格维度/类别不够时可以增加行，但不要减少

### Step 5：上传到 GitHub

#### 5a：检查 `user interview` 是文件还是目录

```bash
gh api repos/sheet0/gtm/contents/Launch/user%20interview 2>&1 | python3 -c "
import json, sys
d = json.load(sys.stdin)
if isinstance(d, dict) and d.get('type') == 'file':
    print('IS_FILE:' + d['sha'])
elif isinstance(d, list):
    print('IS_DIR')
else:
    print('NOT_FOUND')
"
```

#### 5b：如果是占位文件，先删除它

```bash
# 获取占位文件的 SHA
PLACEHOLDER_SHA=$(gh api repos/sheet0/gtm/contents/Launch/user%20interview \
  | python3 -c "import json,sys; d=json.load(sys.stdin); print(d['sha'])")

# 删除占位文件
gh api repos/sheet0/gtm/contents/Launch/user%20interview \
  --method DELETE \
  --field message="chore: remove placeholder, replace with user interview directory" \
  --field sha="$PLACEHOLDER_SHA"
```

#### 5c：创建新的访谈文件

```bash
# 将 Markdown 内容 base64 编码并上传
FILENAME="NN-Helio用户访谈-[受访者标识].md"
CONTENT=$(cat /tmp/interview_draft.md | base64)

gh api repos/sheet0/gtm/contents/Launch/user%20interview/$FILENAME \
  --method PUT \
  --field message="feat: add user interview $FILENAME" \
  --field "content=$CONTENT" \
  --field encoding="base64"
```

> 注意：文件名中的中文在 API 调用时需要 URL encode，使用 `python3 -c "import urllib.parse; print(urllib.parse.quote('文件名'))"` 处理。

### Step 6：输出结果

完成后告知用户：
- 文件编号和名称
- GitHub 文件直链
- 核心发现摘要（3 条）

---

## 错误处理

| 错误情况 | 处理方式 |
|----------|----------|
| 腾讯会议链接需要登录 | 提示用户粘贴文本 |
| 转录内容不完整 | 根据已有内容整理，空白部分标注 |
| GitHub API 失败 | 把生成的 Markdown 输出在对话框，让用户手动上传 |
| 文件名冲突 | 自动递增编号 |

---

## 实际执行流程

收到指令后，按以下顺序操作，**不需要用户确认每一步**，完成后统一汇报：

1. 如缺少必要输入 → 一次性询问所有缺失项
2. 获取转录文本
3. 生成整理文档（先写入 `/tmp/interview_draft.md`）
4. 确定编号 → 上传 GitHub
5. 输出结果摘要
