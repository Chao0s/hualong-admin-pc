# Decision Log（前端修改记录）

> 用途：记录管理端原型（`index.html`）在评审后的修改决定，供后续调整 backend spec（`docs/backend spec files/`）时对照。
> 格式：每条记录 = 修改内容 + 涉及页面 + 对 backend 的影响。与 `hualong-teacher` / `hualong-parent` 两个仓库的 `decision.md` 同一约定。
>
> **本仓库自己的架构决策记在 [`docs/adr/`](docs/adr/)**：本文记「改了什么」，ADR 记「为什么这样划分、下次同类问题怎么裁」。
>
> **跨端的权威来源在后端仓库，本文不复制那些内容**：
> - 成长册的归属规则（谁能改什么）见 `hualong-backend/docs/ADR-0012-growth-book-content-ownership.md`
> - 决议与代价见 `hualong-backend/DECISIONS.md`（本仓库相关的是 **E9**）
> - 跨端对象注册表见 [`docs/spec-handoff.md`](docs/spec-handoff.md)
>
> **本仓库起步晚于另外两端**：`ab75687` 之前的三个提交早于 2026-08-01 的前端评审周期，因此本文从 2026-08-03 起记。此前的改动只有 commit message。

---

## 2026-08-03

本日是管理端的第一次实质设计工作，起因是后端 `db/GAPS.md` **G32**：W19 把成长册封面归给园所（`db_school.book_cover`，只有 admin 能写），但管理端 backend spec、`ui=` 标注、页面、`screens.tsv` **四层全部零落点**——一条已拍板的写入路径在三端都不存在。

本日决议已回冲后端，登记为 `DECISIONS.md` **E9**（五小节），归属规则抽成 **ADR-0012**。

### 1. 新增第九个视图「成长册设置」（`index.html#growthbook`）

**页面**：`index.html`（新增 `<section id="view-growthbook">` + 两个弹层）、`docs/backend spec files/growth-book-setting-spec.md`（新增，第 9 份）

**为什么不挂在已有模块**：
- **内容发布**（`#content`）的模型是「一条条发出去的资讯」——列表 + 发布弹层 + 浏览量 + 下线。而封面/园所介绍是**单例配置**，四项全对不上；且该 spec 明写 `generic_content_table = FORBIDDEN`，`db_school` 不是内容表。
- **组织管理**（`#org`）一度是候选（`db_school` 是园所主数据的根），但范围从「2 个字段」扩到「一园一套的册子配置 + 分发给教师端」之后，Tab 装不下。

**内容**：选用模板 → 园所素材（Logo / 封面照片 / 标题文字）→ 园所介绍 → 园长寄语（按年级）→ 园所栏目 → 预览整册 → 草稿/发布。

**对 backend 的影响**：`db_school` 需加六列——`school_intro`（B12 已定）、`book_cover`（W19 已定）、`logo_file_id`、`book_template_code`、`principal_message`、`book_setting_status(d1=draft|d2=published)`，后四者为本次新增。`can_generate_rule` 的班级级前置条件增加一条 `book_setting_status=d2`。**解掉 G32。**

### 2. 发布即分发；撤回只停止分发，不删数据

**页面**：`index.html#growthbook`

- `d1` 草稿态：教师端取不到模板，全园 `can_generate=0`。
- `d2` 已发布：模板下发，各班教师开始填充班级内容。
- 已发布时表单与模板选择整体锁死，须先撤回。

**撤回的语意与教师端相反**，这点写死防止混用：教师端 W16「撤回素材征集」的语意是**删除**（家长交的裁切成品无法保留）；管理端撤回**只停止分发，不删除任何班级已填内容**。

**对 backend 的影响**：后端不得为这两处复用同一套清理逻辑。

### 3. 美术归属：提出「园所自定义」后收窄为「选模板 + 填占位符」

**页面**：`index.html#growthbook`

评审中提出「边框、本子背景、美术设计应该来自园所」——这正是 `DECISIONS.md` W1b 注明的那条推翻路径（原文：「若要推翻，请明说，那会多一个 admin 端的素材管理入口」）。

追问「园所具体要做什么」后**自行收窄**：园所要的是**自家的内容**（logo、园所照片、园所介绍、园长寄语），不是自己排版面。最终定为——**我们提供 1-2 套封面+内页 template，园长选一套并填占位符**。

**因此 W1b 不推翻，素材管理入口不开。** 连带保住三件事：① 后端红线 5 不动（版式库仍在 repo，不必发明一类「必须存在于生产的产品参考资料」）；② 分辨率 / 透明通道的品控门槛不转嫁给园方；③ W13「版面品质由我们负责」原样成立。

**曾经设计但已全部撤掉的三样**：`db_school.book_theme` 字段、模版快照列、素材上传的尺寸校验。前提没了，结论也就没了。

**对 backend 的影响**：无新增待办。原本要开的 admin 端素材管理入口不需要了。

### 4. 园所栏目：新对象 `db_school_book_section`，版式仍归我们

**页面**：`index.html#growthbook`（园所栏目卡片 + `#modal-gb-sec` 编辑弹层）

园所可新增全园统一的栏目（如「园所大事记」），内容由园所填定后**直接下发**，教师端**只读**——不可改、不可删、不可改插入位置。

**不提供自由网格编辑器**：从版式库的四种栏目版式（纯文字 / 上图下文 / 两图并排 / 整页大图）选一个，只填占位符。理由与第 3 条同源，且在园所级更强——园所栏目进全园每一本册子，排版事故的影响面是单个班级的 N 倍。照片槽位数由 `layout_code` 派生，不是可写字段。

**不复用班级级的 `db_growth_book_section`**：后者 `template_id` 为 `1:1 NOT NULL`、随班级 d1/d2 冻结、带 widget、可向家长征集；园所栏目四项全不成立。复用只能把 `template_id` 改可空再加 scope 判别列，等于制造一张多态表。

**锚点层级规则**：园所栏目的 `anchor_after` 取 `cover` | 预设 `section_key` | 另一个园所栏目 id，**不得锚到班级级栏目**（园所不知道各班有什么）；反向成立。规则是**下层可锚上层，上层不可锚下层**。成环挡在下拉选项层（排除自己与所有递归锚定到自己的），删除时锚在其后的栏目改锚到它原本的锚点，渲染端多轮扫描落位、解不开的落册尾。

**对 backend 的影响（新增一张表，46 → 47）**：`db_school_book_section`，字段见 `docs/backend spec files/growth-book-setting-spec.md`。`file_id` 的 0:k 走 `db_file_ref` 还是本表多列，**未定**。成环校验**服务端必须独立重做一次**——前端 UI 从来不是完整性边界。

### 5. 年级差异只在文字层分叉，不下发年级级栏目

**页面**：`index.html#growthbook`（园长寄语的四个年级页签）

评审提出「大班要『最后一天』、小班要『第一天』，管理端该不该按年级分开下发栏目」。**判定：不下发。**

- W19 原文举的例子就是「入学第一天」「毕业典礼」，本来就归班级。
- 年级差异恰恰是**个性化的证据，不是统一配置的证据**——一个栏目只要按年级分叉，「统一配置」就已经不统一了。
- `db_growth_book_template` 唯一键是 `class_id`，管理端再下发一层年级级模版会产生**两层模版的合并问题**（园所下发的栏目教师能不能删/能不能改/改了算谁的），正是 W13 花力气绕开的那条路。
- 年级不是唯一分叉维度，混龄班与转班会立刻要求「按班级配」，管理端就成了教师端的超集。

**替代路径**：园长寄语存 `{default, k1, k2, k3}`，教师端按 `db_class.grade` 取，该键为空回落 `default`，`default` 亦空则该占位符不渲染、不留空框。管理端界面只多三个页签，不引入第二层模版、不引入 override 表。先例：`db_resource.grade SET('k1','k2','k3')`。

**对 backend 的影响**：`principal_message` 落 json，取值域限 `default|k1|k2|k3`，其余键 return 400。

### 6. 不引入模版快照，E3 第 3 点维持不变

**页面**：无（是一处**决定不做**的记录）

曾考虑：园所在学期中改了内容，3 月生成与 6 月生成的册子会不一致，是否要在 `db_growth_book_template` 上加一列快照。**判定：不加。**

① 美术与版面仍在 repo，只随发版变，不存在运行时漂移（**这正是第 3 条收窄后才成立的**）；② 已生成的册子是 `db_growth_book.generated_file_id` 指向的成品文件，不受后续改动影响；③ 未生成的取最新内容是预期行为。

**若日后推翻第 3 条把美术交给园所，本条必须一并重审**——快照的需求是跟着「运行时可改的设计」来的。

### 7. 园所介绍与园长寄语：即时预览 + 确认保存

**页面**：`index.html#growthbook`

- 填写框右侧一张 A4 比例示意页，**随输入即时更新**；园长寄语那张跟随年级页签，切到「小班」而小班未写时直接显示通用那段并标注「回落通用」。
- 各加一枚「确认保存」按钮 + 「已保存 / ● 未保存」标记。**预览是即时的（看效果），落库要点按钮（安全感）**。
- 三处防丢失：切年级页签时若有未保存修改**自动提交并 toast 告知**；「保存草稿」与「发布设置」都先提交全部输入再执行，不会拿过期值做发布校验。

**对 backend 的影响**：无。纯前端交互，写入面不变。

### 8. 园所介绍与园长寄语改为可配图，与园所栏目共用同一套版式

**页面**：`index.html#growthbook`

这两节初版只有纯文字，实际都需要配图。改为与园所栏目**同一套版式**——从版式库选一个、只填占位符，**只出带正文的三种**（纯文字 / 上图下文 / 两图并排），默认「上图下文」。「整页大图」没有正文位，对这两节没有意义，不出。

**版式与照片整节一份，不按年级分叉。** 园长寄语的**文字**按年级分叉，但版式与照片四个年级共用——理由与第 5 条同源：分叉只发生在文字层。版式若也按年级分叉就是四套版面，册子会因班级年级不同而长得不一样，那正是「园所级 = 全园一样」要排除的情况。某年级真需要不同的图，那是班级级内容，走教师端新增栏目。

**三处预览统一到一个渲染器**：填写区旁的即时预览、整册预览、以及园所栏目，现在都走 `gbSecPage()`。预设两节被包装成与栏目同形的对象喂进去，形状必定一致，不会出现「填写时看到的和册子里不一样」。

**对 backend 的影响**：`db_school` 再加四列——`intro_layout_code`、`intro_file_id(0:k)`、`message_layout_code`、`message_file_id(0:k)`。累计本 spec 为 `db_school` 增 10 列（其中 8 列成长册专属），已登记 `db_school_column_growth_note`：DDL 落地时可考虑收进 JSON 或独立表，但**管理端不自行改判**——W19 与 B12 已把 `book_cover`、`school_intro` 定为直接列。

### 9. 空内容的园所栏目挡住发布（提示从轻）

**页面**：`index.html#growthbook`

版式声明了 N 个照片槽位却未传满、或声明 `body=1` 而正文为空 → 册子上会印出空白框，**挡在发布这一步，不挡保存草稿**（允许先建栏目占位）。

判定**只看该版式声明的槽位**：纯文字版式不要求照片，整页大图版式不要求正文。提示强度：栏目行一枚 badge（「缺 1 张照片、正文」）+ 发布时一条 toast 点名第一个不齐备的，不用弹层、不用阻断横幅。`heading_text` 为空不算缺——回落取栏目名，不会印出空白。

同一套判定也作用于第 8 条的预设两节：园所介绍正文必填 + 声明的图位须传满；园长寄语**整节为空是合法的**（占位符不渲染），只有整节有文字时才检查图位。

**对 backend 的影响**：`IF 请求发布 AND 任一 db_school_book_section 的声明槽位未传满或声明的 body_text 为空, return 422` 并回传 `section_id` 与缺失项；预设两节同理，见 spec 的 `preset_section_completeness_rule`。

### 10. 预览版心几何修正 —— 与后端 R26 是同一个错

**页面**：`index.html`（`.book-page` / `.book-frame` / `.book-body` 的 CSS）

预览页原用 `inset: 12% 13%` 切版心。**百分比 inset 的两轴基准不同**（横轴按宽、纵轴按高），用相近数字看起来对称，算出来是 176×255mm 而非 150×240mm，比例 0.691 而非 0.625——**预览里所有比例都是假的**。

按 W1b 改为按各自轴换算：上下 `28.5/297 = 9.596%`、左右 `30/210 = 14.286%`，实测 0.623（残差来自 1px 边框，缩略图层面可忽略）。

**这与后端 `MEMORY.md` R26 是同一个错，隔一天在另一个仓库又犯一次。** 静态检查与桩 DOM 都测不出来，是在真浏览器里量 `getBoundingClientRect` 才发现的。**教训：预览的几何要实测，不能读 CSS 推。**

**对 backend 的影响**：无。

### 11. `spec-handoff.md` 移出 `Archive/` 并补齐注册表

**页面**：`docs/spec-handoff.md`（原 `docs/Archive/spec-handoff.md`）

九份 spec 都写着 `canonical_registry = docs/spec-handoff.md`，而文件在 `Archive/`——Archive 读起来像历史存档，但这份注册表是活的、管着每一份新 spec。

注册表本身是**交接当刻的快照，之后没跟**：登记 39 个对象，而 DDL 里有 12 个没登记（含管理端自己引入的 `db_review_action` 等 6 个）、E 组 8 张新表全部没登记、`db_community_submission` 已被 B11 拔掉却仍列为 canonical——**会让人复用一张正在被删的表**。已补到 61 个，新增两节（下游引入的对象 / 已决未落地），并把「注册表跨三节」写进第 15 节的复制粘贴 prompt。

移动与内容更新**拆成两个提交**：先还原成移动前的逐字节原样提交纯 rename（100% 相似度），再提交内容更新。混在一起会让 rename 相似度掉到阈值以下，git 显示成「删一个文件 + 加一个文件」，Archive 时期的历史就断了。

**对 backend 的影响**：无。但 `db/spec/screens.tsv` 已补登本仓库第 9 个视图 `index.html#growthbook`。

### 12. 补 `.gitignore`

**页面**：`.gitignore`（新增）

本仓库原先没有，Git Bash 崩溃转储与 Office 锁文件一直显示为未跟踪。合并了另外三个仓库已在用的模式：`.od-skills/`、`*.artifact.json`（Teacher/Parent）、`*.stackdump`（Parent）、`.DS_Store` `Thumbs.db` `desktop.ini` `.idea/` `.vscode/`（backend）、`~$*`（Teacher）。

---

## 本仓库仍未处理的

| # | 项目 | 说明 |
|---|---|---|
| 1 | 锚点命名漂移 | 九份 spec 写 `index.html#home-school`，代码是 `data-view="homeschool"`，后端 `screens.tsv` 也用 `homeschool`。**spec 是唯一不一致的那个**。新页面用了 `growthbook` 避免再造一处，旧的那处未动 |
| 2 | 新对象声明键名有四种写法 | `new_canonical_objects_defined_here`（review-spec，复数）、`new_canonical_object_defined_here`（library-spec，单数）、`new_identity_objects_defined_here`（dashboard-spec）、`new_business_object`（其余四份）。抽取工具要认得写四个分支。注册表 §5 已有统一落点，收敛时机比之前好 |
| 3 | 教师端连带未做 | `growth-book-render.js` 的 `SCHOOL_COVER` 常量与硬编码 6 色调色板要改成读园所设置；`05 home-school-spec.md` 的 `border_ownership` 要改写登记本次收窄 |
| 4 | `db/README.md` 的「17 份 backend spec」 | 本仓库从 8 份变 9 份，该数字已过期。但原值 17 与实际（Teacher 5 + Parent 5 + Admin 8 = 18）本来就对不上，口径不明，**未擅改**，待能跑工具时重新推导 |

---

## 2026-08-12：幼儿资料更正审核（后端 F13）

- 审核中心新增「幼儿资料」页签，内分待审核／已处理；两页都有班级与幼儿姓名筛选，已处理另有批准／驳回结果筛选。
- 待审详情并排显示提交时原值与家长建议值；若正式资料后来变化，再显示当前系统值和覆盖提示。批准仍完整采用家长建议快照，不逐栏合并。
- 任一有效同园管理端登录账号均可处理，不新增细分权限。批准、驳回都先一次确认；驳回理由须为 1—500 字。
- 批准同交易写入 `db_child`、终结申请并插入 t6/d1 审核动作，不通知家庭；驳回同交易写 t6/d2，并通知决定当下全部监护人。

## 2026-08-12：F17 成长册 App 内定稿覆盖

本节以 `hualong-backend/DECISIONS.md` F17 与 USER-JOURNEY Q63—Q86 为权威，覆盖本文早期所有 `can_generate`、g0／g1／g2、generated file、PDF、下载、分享、重导及 render lease 口径。

- 管理端只有 PC 后台，本期不做管理端小程序。
- 园所成长册设置仍是 d1 草稿、d2 一次发布并永久锁定；d1 时全园 `can_finalize=0`，d2 才分发给教师端。
- 幼儿成长册改为 b1 准备、b2 已定稿并只在 Teacher／Parent App 内开放；不产生可下载文件。
- 管理端家园共育统计统一称“成长册定稿率／已定稿开放比例”，不再称“生成率”。
- 办园质量评估的 120 题工具由 developer 以版本化代码资产维护，admin 不能编辑题文或工具结构。
