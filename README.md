# KAWEISKILL

DeepSeek Harness / Claude Agent Skills 集合。

## 内容

| Skill | 来源 | 说明 |
|---|---|---|
| [`learn-skill/`](learn-skill/SKILL.md) | 自研 | **学习架构 D v4**：把一门系统性学问压成「一个顶层整体 ＋ 一个贯穿地基元素」，并用「连结关系图」把二者钉在一起。含建库脚本、一体知识点单元模板、5 阶段学习路径（第 0 → 0.5 → 1 → 2 遍 ＋ 应用穿针）、1/3/7/30 天间隔复习、缺口账本、以及图片资料的多模态摄入规范。 |
| [`lexiang/`](lexiang/) | **腾讯乐享官方**（`@lexiang/skills` v1.1.2，MIT） | **乐享 AI 知识库 MCP 技能包**（6 个 skill）：配置向导、搜索阅读、文档写入、Block 编辑、文件上传下载、外部数据源导入。详见下方「[lexiang — 乐享 AI 知识库](#lexiang--乐享-ai-知识库)」。 |

---

# lexiang — 乐享 AI 知识库

> ⚠️ **来源声明**：`lexiang/` 下的 6 个 skill 是 **腾讯乐享官方发布的 `@lexiang/skills` 包（v1.1.2，MIT 许可）**，此处为原样收录，**非本仓库原创**。
> 官方安装方式：`npx @lexiang/skills install`

## 这是什么

腾讯乐享（lexiangla.com）知识库的 **MCP 技能包**。它让 AI Agent 能**读写**企业的乐享知识库 —— 不只是"知道有哪些文件"，而是**能读到文件正文、能创建和编辑文档**。

## 六个 skill

| Skill | 职责 |
|---|---|
| `lexiang-setup` | **MCP 配置向导** —— 首次配置、Token 管理、连接验证、401 排障、多租户切换 |
| `lexiang-search` | **搜索与阅读** —— 关键词/语义搜索、知识库浏览、目录导航、文档读取 |
| `lexiang-writer` | **文档写入** —— 创建页面/文件夹、导入 Markdown/HTML、公众号收藏 |
| `lexiang-blocks` | **已有页面编辑** —— Block 级创建/更新/删除/移动、Markdown 转 Block |
| `lexiang-files` | **文件上传下载** —— 三步上传（申请→PUT→确认）、文件详情、下载 |
| `lexiang-connectors` | **外部数据源** —— 腾讯会议录制导入、iWiki 文档迁移 |

## 安装

```bash
# 官方方式（推荐，自动检测已安装的 Agent 并复制到对应目录）
npx @lexiang/skills install

# 或从本仓库手动复制
# 把 lexiang/ 下的 6 个目录复制到你的 skills 目录，例如：
#   ~/.agents/skills/      （通用标准）
#   ~/.dsh/skills/         （DeepSeek Harness）
#   ~/.claude/skills/      （Claude Code）
```

## 配置（必做）

乐享 MCP 用 **Bearer Token 静态鉴权**（不涉及 OAuth）。需要两个参数：

| 参数 | 说明 | 从哪拿 |
|---|---|---|
| `COMPANY_FROM` | 企业标识（32 位十六进制） | https://lexiangla.com/mcp |
| `LEXIANG_TOKEN` | 访问令牌（格式 `lxmcp_xxx`） | 同上，登录后生成 |

写入 MCP 配置文件（`~/.mcporter/mcporter.json` 等）：

```json
{
  "mcpServers": {
    "lexiang": {
      "enabled": true,
      "url": "https://mcp.lexiang-app.com/mcp?preset=meta&company_from=你的COMPANY_FROM",
      "transportType": "streamable-http",
      "headers": {
        "Authorization": "Bearer 你的LEXIANG_TOKEN"
      }
    }
  }
}
```

> 🔒 **安全提醒**：`LEXIANG_TOKEN` 等同账号访问权，**不要提交到任何公开仓库**。本仓库中的 `lexiang-setup/mcp.json` 只含 `${COMPANY_FROM}` / `${LEXIANG_TOKEN}` 占位符。

**Token 过期怎么办**：报 401 时**不需要重新获取 token**，点「续期」即可恢复：

```
https://lexiangla.com/mcp?company_from=你的COMPANY_FROM
```

## 能做什么（实测）

连接成功后，乐享 MCP 提供 **78 个工具**，分 20 个类别。关键能力：

| 能力 | 代表工具 | 实测结果 |
|---|---|---|
| **读正文**（不只是文件名） | `search_kb_search` | ✅ 返回正文片段，如 `"《公路桥涵设计通用规范》(JTG D60-..."` |
| **真向量语义检索** | `search_kb_embedding_search` | ✅ 带相似度 score，可设 `threshold`（0-1） |
| **列知识库 / 目录树** | `space_list_spaces`、`entry_list_children` | ✅ |
| **创建文档** | `entry_create_entry` | ✅ 实测创建成功 |
| **Block 级编辑** | `block_update_block`、`block_create_block_descendant` | 21 个工具 |
| **智能表格 CRUD** | `smartsheet_*` | 14 个工具 |
| **文件上传** | `file_apply_upload` → PUT → `file_commit_upload` | 三步上传 |
| **文件版本回滚** | `file_list_revisions`、`file_revert_file` | |
| **文件翻译** | `translation_apply_file_translation` | |
| **评论读取** | `comment_list_comments` | |

### 调用要点（踩坑记录）

1. **meta 工具直接调**（`list_tool_categories` / `search_tools` / `get_tool_schema`），**不能走 `call_tool`**
2. **业务工具必须走 `call_tool`**：`call_tool(tool_name="xxx", arguments={...})`
3. `search_kb_embedding_search` 的参数是 **`filters.keyword`**，不是顶层 `query`
4. `entry_set_entry_validity` 用 **`validity_type`**（`force_expire` = 强制失效）
5. 响应用 **SSE 格式**（`data:` 行），不是纯 JSON

## 对比：乐享 vs IMA

| 能力 | IMA OpenAPI | 乐享 MCP |
|---|---|---|
| 读正文 | ❌ 只能匹配文件名 | ✅ 返回正文片段 |
| 向量语义检索 | ❌ 假语义 | ✅ 真向量 + score + threshold |
| 创建文档 | ❌ | ✅ |
| 编辑已有页面 | ❌ | ✅ Block 级 |
| 智能表格 | ❌ | ✅ 14 个工具 |
| 文件版本回滚 | ❌ | ✅ |
| 文件翻译 | ❌ | ✅ |
| 接口数量 | 10 个 | **78 个** |

---

# learn-skill — 顶层整体 ＋ 贯穿地基的深度学习

> 学习不是"读百科"，而是：**先立顶层整体 → 找到贯穿它的地基元素 → 用地基"钉穿"顶层每个角落 → 再用现实问题穿针验证。**

## 一、这是什么

一个**可执行的学习方法论 skill**。它不产出"知识总结"，产出一套**结构化、可自测、可复习、可增量维护**的学习 Wiki。

核心模型（一页骨架，两个尺度）：

```
骨架（顶层整体）   核心问题 + 领域公式 领域 ≝ ⟨要素…⟩ + 协同图
        ▲ 连结关系图（支撑线：作用于 / 相对它成立 / 由它组成）
地基（贯穿元素）   一个（或极少数）基本原子 ── 顶层一切共同作用的对象 / 前提
```

**关键铁律**：顶层与地基**不是两张图**，而是同一件事的不同尺度。Wiki 里只维护**一张**「连结关系图」，地基在下、顶层在上。

### 地基元素的三种形态

| 形态 | 含义 | 例 |
|---|---|---|
| **对象型** | 顶层所有运算都**作用在它身上** | 微积分 → **函数** |
| **背景型** | 顶层所有定律都**相对它成立** | 牛顿力学 → **惯性系** |
| **原料型** | 顶层所有构造都**由它组成** | Python → **对象/值** |

**判定标准**：删掉它，顶层每个要素都失去意义；并且必须通过**贯穿度自检表**（逐要素打勾，有一个不命中就说明地基没找对）。

## 二、安装

### 方式 A：DeepSeek Harness（用户技能目录）

```powershell
# 1) 取到 skill
git clone https://github.com/dawalixi1980/KAWEISKILL.git
# 2) 复制到用户技能目录（DSH 会自动发现，无需重启）
Copy-Item .\KAWEISKILL\learn-skill "$env:USERPROFILE\.dsh\skills\learn-skill" -Recurse
```

验证：让 agent 说一句"用 learn-skill 教我 XXX"，或看技能目录里出现 `learn-skill\SKILL.md`。

### 方式 B：Claude / 其他 Agent Skills 运行时

把 `learn-skill/` 整个目录放进你的 skills 目录即可（结构符合 Agent Skills 约定：`SKILL.md` + `references/` + `scripts/`）。

### 依赖

**本 skill 自身零第三方依赖**：`scripts/init-wiki.py` 只用 Python 标准库（`argparse` / `io` / `os` / `sys` / `datetime`）。

它**可选**协作三个兄弟 skill，缺了也能用（只是少对应能力）：

| 兄弟 skill | 作用 | 缺失时 |
|---|---|---|
| `math-skill`（math-render） | 公式 / 函数图 → PNG | 公式只能留 LaTeX 文本 |
| `llm-wiki-skill` | Wiki 持久化底座（目录与链接规则、IMA 模式） | 直接用本 skill 自带的 `wiki-layout.md` |
| `ima-skill` | IMA 向量库检索（做母知识源） | 改用本地文件 / 上传资料 |

## 三、快速开始（3 步）

```powershell
# 第 1 步：建骨架（生成完整目录树，纯脚手架）
python learn-skill\scripts\init-wiki.py --path .\我的微积分 --domain "微积分"

# 第 2 步：填 wiki\00-骨架\index.md 的「顶层三件套 + 地基元素 + 连结关系图」
#   （让 agent 用 learn-skill 帮你填，或自己照 references/learning-arch-d.md 写）

# 第 3 步：逐要素教学 → 每个要素一个文件夹 + ≥1 个「一体知识点单元」
```

生成的结构：

```
<项目>/
├── raw/源文件清单.md            # 母知识登记（单一事实源）
├── raw/images/                  # 输入原图（截图/拍照）— 只读存档
├── media/<slug>.md              # 图片的图文卡（OCR/描述/洞察）— 让图可检索
├── index.md  log.md  WIKI-SCHEMA.md
└── wiki/
    ├── 00-骨架/                 ★ 知识点＝文件夹
    │   ├── index.md             # 顶层三件套 + 地基元素 + 连结关系图（双向标注）
    │   └── img/
    ├── 01-问题索引.md            # 从根到叶的问题表 + 1/3/7/30 掌握状态
    ├── 02-缺口账本.md            # 第 2 遍自测答不出的真缺口
    ├── 索引-<分类>.md
    ├── <分类>/<要素>/            # 每个顶层要素一个文件夹
    │   ├── index.md             # 要素页 = 若干「一体知识点单元」
    │   └── img/                 # 该知识点的公式图/函数图
    └── 应用/钉子表.md             # 现实问题穿针（顶坐标 / 底坐标）
```

## 四、学习路径（5 阶段）

| 阶段 | 动作 | 产出 |
|---|---|---|
| **第 0 遍 · 整体** | 立核心问题 + 领域公式（要素 3±1）+ 协同图 | `wiki/00-骨架/index.md` |
| **第 0.5 遍 · 找地基** | 锁贯穿元素 + **贯穿度自检表** | 骨架页「地基元素」节 |
| **第 1 遍 · 爬升** | 逐要素教学（每要素 = 1 文件夹 + ≥1 单元） | `<分类>/<要素>/index.md` |
| **第 2 遍 · 下山自测** | **合上图**，只看顶层公式，自推每要素如何作用在地基上 | `02-缺口账本.md` |
| **应用穿针** | 每要素 2–3 个现实问题 | `应用/钉子表.md` |

复习：`01-问题索引.md` 按 **1 / 3 / 7 / 30 天** 复查，**只看问题不看答案**。

> **第 2 遍是生死线**：AI 时代最容易产生"流畅性错觉"——读着解释觉得全懂，其实没懂。唯一解药是合上它自己推。

## 五、一体知识点单元（两档）

- **最小档（默认）**：`问题 → 符号 → 语义 → 机制(作用在地基元素上) → 定位`
- **完整档（复杂概念）**：再加 `协同演示`、`双向标注`、`应用钉子`

**「机制」字段必填**——它必须说清"这个知识点如何作用在地基元素上"。没写 = **悬空砖块** = 不合格。这正是本方法论区别于"脑图/知识卡片"的地方：**信息从 O(N) 压缩到 O(1) + 索引线**。

## 六、教学铁律（11 条速记）

顶层整体先行 · 锁定贯穿地基 · 必出连结关系图 · **在地基上教学**（机制 + 语义合并讲）· 一体知识点单元 · 协同链演示 · 单一理论核心 · 符号化优先 · 应用穿针 · 鼓励重绘连结图 · 图文并茂。

完整 11 条见 [`references/learning-arch-d.md`](learn-skill/references/learning-arch-d.md) 第七节。

## 七、九大失败模式（自查）

| 误区 | 表现 | 修正 |
|---|---|---|
| 把底层当另一套架构 | 底层列一堆平行机制 | 压缩为"贯穿顶层的基本元素" |
| 找不到贯穿元素 | 说不出"顶层每件事作用在谁身上" | 问"删掉它顶层是否全失效" |
| 元素没有贯穿 | 只服务某一要素 | 换能覆盖全部的（用自检表） |
| 只有顶层没有底层 | 能复述概念，说不出对象/前提 | 先锁地基 |
| **砖块悬空** | 知识点没挂到地基上 | 每个单元都标"对地基的什么运算" |
| 把课后习题当钉子 | 只考背/算 | 换"答案有意义"的现实问题 |
| 平行多个根 | 多个互不相干的顶层根 | 合并为一个根 + 一个地基 |
| 顶层要素爆炸 | 领域公式列了 8 个要素 | 收敛到 3±1 |
| 顶层底层做成两页 | 骨架拆成两张平行图 | 合成一页 + 一张连结关系图 |

## 八、文件结构

```
learn-skill/
├── SKILL.md                        # 入口：核心模型 / 结构 / 路径 / 铁律
├── _meta.json                      # slug + version
├── references/
│   ├── learning-arch-d.md          # ★ 理论层：D v4 完整定义（254 行）
│   ├── unit-template.md            # 一体知识点单元模板 + 微积分/牛顿力学示例
│   ├── workflows.md                # 操作层：建库/摄入/教学/自测/复习/健康检查
│   ├── wiki-layout.md              # 目录与链接规则
│   ├── media.md                    # 图片资料多模态摄入（图文卡）
│   └── question-design.md          # 问题设计规范
└── scripts/
    └── init-wiki.py                # 建骨架（零依赖，纯标准库）
```

## 九、上面那个理论的一个已知缺口（欢迎补）

`learning-arch-d.md` 假设**学问都能被结构化**（都有一个唯一核心问题 + 一个贯穿地基）。但现实中存在：

- **程序性知识**（游泳、骑车、写代码的手感）——没有"领域公式"，只有身体记忆；
- **默会知识 / 品味**（什么叫好设计）——难以符号化，而第 8 条要求"符号化优先"；
- **无中心知识**（历史、临床医学）——硬找"唯一地基"会**过度简化成误导**。

**建议的补丁**：给"贯穿度自检"加一个**地基缺席分支**——当换 2–3 个候选仍覆盖不全时，判定为**"无中心型"**，放弃单一地基，改为**"多锚点 + 场景索引"**，并把学习重心前移到第 2 遍自测与穿针层。

## 许可

MIT
