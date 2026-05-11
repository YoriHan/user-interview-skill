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

**核心目标**：找场景——用户具体在用 Helio 做什么、怎么做、为什么用 Helio 而不用别的工具。

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

如果用户直接提供了逐字稿文本，跳到 Step 3。

### Step 2：获取转录文本

腾讯会议转写链接包含两层内容：顶部的 **AI 智能纪要**（已结构化），和下方的**逐字稿**（说话人 + 时间戳）。

**逐字稿是最重要的原材料**，AI 纪要只作为骨架参考。

用 browse 工具（headless browser）抓取，因为页面需要 JS 渲染：

```bash
B="$HOME/.claude/skills/gstack/browse/dist/browse"
$B goto [用户提供的链接]
sleep 2
$B wait --networkidle
$B text 2>&1 | head -500

# 继续滚动获取完整逐字稿
$B scroll --direction down --amount 3000
sleep 1
$B text 2>&1 | tail -400

# 如有必要继续滚动
$B scroll --direction down --amount 3000
sleep 1
$B text 2>&1 | tail -400
```

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

基于逐字稿生成以下格式的 Markdown 文档。

```markdown
# NN-Helio用户访谈-[受访者标识]

**受访者**：[匿名标识] ｜ **身份**：[职业/学校/行业] ｜ **参与者**：[Helio团队成员] ｜ **时长**：约 XX 分钟

---

## 用户背景

[3-4 段自然段，不用表格]

受访者身份、团队规模、核心工作场景（越具体越好，说清楚日常在做什么事情）。

AI 工具使用习惯：高频/低频，现在用什么，各自付不付费、多少钱。
迁移路径：[工具A]（用来做什么 / 为什么离开）→ [工具B]（为什么离开）→ Helio（原因）。

来 Helio 的核心动机：① ... ② ... ③ ...
内测时长和活跃频率，目前主要用到哪些功能。

---

## AI 工具全景

| 工具 | 使用场景 | 是否付费 | 备注/评价 |
|------|---------|---------|----------|
| [工具名] | [具体用来做什么] | [付费金额或免费] | [原话/关键评价] |
| ... | ... | ... | ... |
| 黑流（Helio） | [整合了哪些需求] | 内测 | [来 Helio 的核心价值] |

---

## 核心使用场景

> 本节是整份文档最重要的部分。P0 目标：找场景。每个场景单独一个 ### 小节。

### 场景一：[场景名称]

**配置方式**：[用了什么 Skill？自己写 prompt？从哪里引入的？]  
**触发时机**：[什么情况下会用这个场景？]  
**具体产出**：[做出来了什么？效果如何？]  
**与竞品对比**：[用户怎么评价 vs 竞品，尽量用原话]  

> 「[用户的关键原话]」

**卡点/问题**：[这个场景遇到的具体问题]

### 场景二：[场景名称]

[同上结构]

---

## Aha Moment 与卡点

**最爽的瞬间**：[具体是什么场景、什么时刻，引用原话]

**最大卡点**：[让用户最不爽的具体体验，引用原话]

**[重要功能] 未使用的真实原因**：[如果用户没用 Channel / Memory / 协作，找到真实原因（数据在哪？前提条件是什么？），不要简单写"未探索"]

> 「[用户原话]」

---

## 产品定位认知

[用户怎么理解和定位 Helio？它跟其他工具是替代还是互补？引用用户原话]

> 「[关键定位原话]」

---

## 问题反馈与需求

| 类别 | 详情 |
|------|------|
| Bug（高优先级） | 1. [具体 bug，说明现象和影响范围]<br>2. ...<br>3. ... |
| 认知卡点 | 1. [用户误解了什么功能？在哪里卡住了？]<br>2. ...<br>3. ... |
| 功能需求 | 1. [具体需求，说明使用场景]<br>2. ...<br>3. ... |

---

## 付费意愿

- **当前付费现状**：[现在为哪些工具付费，各多少钱]
- **Helio 付费价位**：[具体数字，说清楚人民币还是美金]
- **付费前提**：[用户提到了什么条件才愿意付费]
- **对定价结构的判断**：[用户对分层定价或其他定价方式的看法]

  > 「[付费相关的关键原话]」
```

**整理铁律**：
1. **逐字稿原话 > AI 纪要**：每个重要结论都要有逐字稿里的原话支撑，用 `> 「...」` 格式引用
2. **场景要具体**：写清楚配置方式（用了什么 Skill？自己写 prompt？）、触发时机、产出、竞品对比
3. **未使用功能写真实原因**：Channel / Memory 未使用不能只写"未探索"，要找到真实前置条件
4. **竞品必须点名**：用户提到的竞品名称、优劣评价都要记录，不要模糊化
5. **付费意愿要有数字**：价格区间、货币单位、付费前提缺一不可
6. **不编造内容**：转录里没有的标注"（访谈未覆盖）"
7. **删除无关模块**：HR / Calendar 等如果用户完全未涉及，直接不写，不要留空行占位

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
PLACEHOLDER_SHA=$(gh api repos/sheet0/gtm/contents/Launch/user%20interview \
  | python3 -c "import json,sys; d=json.load(sys.stdin); print(d['sha'])")

gh api repos/sheet0/gtm/contents/Launch/user%20interview \
  --method DELETE \
  --field message="chore: remove placeholder, replace with user interview directory" \
  --field sha="$PLACEHOLDER_SHA"
```

#### 5c：创建或更新访谈文件

```bash
FILENAME="NN-Helio用户访谈-[受访者标识].md"
ENCODED_FILENAME=$(python3 -c "import urllib.parse; print(urllib.parse.quote('$FILENAME'))")
CONTENT=$(base64 < /tmp/interview_draft.md)

# 检查文件是否已存在（更新时需要传 SHA）
EXISTING_SHA=$(gh api repos/sheet0/gtm/contents/Launch/user%20interview/$ENCODED_FILENAME 2>/dev/null \
  | python3 -c "import json,sys; d=json.load(sys.stdin); print(d.get('sha',''))" 2>/dev/null || echo "")

if [ -n "$EXISTING_SHA" ]; then
  # 更新已有文件
  gh api repos/sheet0/gtm/contents/Launch/user%20interview/$ENCODED_FILENAME \
    --method PUT \
    --field message="feat: update user interview $FILENAME" \
    --field "content=$CONTENT" \
    --field encoding="base64" \
    --field sha="$EXISTING_SHA"
else
  # 新建文件
  gh api repos/sheet0/gtm/contents/Launch/user%20interview/$ENCODED_FILENAME \
    --method PUT \
    --field message="feat: add user interview $FILENAME" \
    --field "content=$CONTENT" \
    --field encoding="base64"
fi
```

### Step 6：输出结果

完成后告知用户：
- 文件编号和名称
- GitHub 文件直链
- 核心发现摘要（3 条，聚焦场景和产品 insight）

---

## 错误处理

| 错误情况 | 处理方式 |
|----------|----------|
| 腾讯会议链接需要登录 | 提示用户粘贴逐字稿文本 |
| 转录内容不完整 | 根据已有内容整理，空白部分标注 |
| GitHub API 失败 | 把生成的 Markdown 输出在对话框，让用户手动上传 |
| 文件名冲突 | 自动递增编号 |

---

## 实际执行流程

收到指令后，按以下顺序操作，**不需要用户确认每一步**，完成后统一汇报：

1. 如缺少必要输入 → 一次性询问所有缺失项
2. 获取转录文本（browse 优先，滚动获取完整逐字稿）
3. 生成整理文档（先写入 `/tmp/interview_draft.md`）
4. 确定编号 → 上传 GitHub（支持新建和更新）
5. 输出结果摘要
