ADMIN_GROWTH_BOOK_SETTING_BACKEND_OBJECT_SPEC

scope (范围) = index.html#growthbook
source_page (参考页面) = index.html
static_node_count (固定按钮节点数) = 28
dynamic_template_card_count (动态模板卡片数) = 1:k
dynamic_section_layout_card_count (动态版式卡片数) = 1:k
dynamic_preset_photo_slot_count (预设两节动态图位数) = 0:k
dynamic_school_section_row_count (动态园所栏目行数) = 0:k
dynamic_preview_page_count (动态预览页数) = 0:k
runtime_clickable_node_count (运行时可点击节点数) = 28:(28+k)
field_format (字段格式) = field_key (中文字段名), cardinality, type|enum, ui
id_rule (ID规则) = integer, database_auto_generated
null_rule (空值规则) = 0:1
list_rule (列表规则) = 0:k | 1:k


[SHARED_OBJECT_RULE]

canonical_registry = docs/spec-handoff.md
shared_object_source = dashboard-spec.md; Teacher App home-spec.md|05 home-school-spec.md
shared_objects = db_admin, db_admin_school, db_school, db_file, db_class, db_growth_book_template, db_growth_book
new_business_object = db_school_book_section (园所级新增栏目；与班级级的 db_growth_book_section 是两个对象，见 school_section_scope_rule)
admin_page_aggregate = db_admin_growth_book_setting_home
design_asset_table = FORBIDDEN; 边框/底纹/配色不落业务表，见 design_ownership_rule
grade_scoped_template = FORBIDDEN; 管理端不下发年级级模版，见 no_grade_section_rule


[CONTEXT_RULE]

admin_id = auth_session.admin_id
allowed_school_id = db_admin_school.school_id WHERE admin_id=current_admin_id AND is_active=1
current_school_id MUST IN allowed_school_id
permission = db_admin_school.role|permission_scope
required_permission = growth_book.setting.read|growth_book.setting.write
raw admin_id|school_id|file_id ui = context.hidden
client_editable = 0
backend_authorization_validation = REQUIRED
write_role_rule = db_school 的全部本节字段只有 admin 能写，school_id derived（DECISIONS.md W19；对齐 05 home-school-spec.md 的 `IF 写入 db_school.book_cover AND role!=admin, return 403`）


[DATA_INITIALIZATION_RULE]

prototype_content = HTML 中模板卡片缩略图、示例园所介绍、示例园长寄语、示例园所栏目「园所大事记」、logo 与封面照片均为 Mock
static_ui_content = 模块名、字段标签、年级页签名、栏目版式名、上传规格说明、按钮名和空状态文案
business_seed = NONE
production_initial_business_objects = db_school 本节内容字段 NULL；intro_layout_code|message_layout_code 取版式库默认值 photo；book_setting_status=d1；db_school_book_section EMPTY
layout_library_not_business_data = 页版式库（后端仓库 db/layout/*.json）随部署走、跟 git 走 code review，不是业务表，不受红线 5「生产业务表初始为空」约束；红线 5 的适用对象是 db_*
dynamic_list_without_data = []
dynamic_count_without_data = 0
hardcoded_business_id = FORBIDDEN
hardcoded_template_code = FORBIDDEN; 合法取值以版式库 manifest 为准
environment_isolation = demo|test 数据不得复制到 production


[STATIC_BUTTON_NODE_INDEX]

| n | button_name_cn | button_name_en | node_key | object | input | jump/action |
|---:|---|---|---|---|---|---|
| 1 | 数据看板 | Dashboard | nav_admin_dashboard | nav_admin_dashboard | NULL | index.html#dashboard |
| 2 | 审核中心 | Review Center | nav_admin_review | nav_admin_review | NULL | index.html#review |
| 3 | 资源与案例 | Resources and Cases | nav_admin_library | nav_admin_library | NULL | index.html#library |
| 4 | 任务管理 | Task Management | nav_admin_tasks | nav_admin_tasks | NULL | index.html#tasks |
| 5 | 测评数据 | Assessment Data | nav_admin_assessment | nav_admin_assessment | NULL | index.html#assessment |
| 6 | 家园共育数据 | Home-School Data | nav_admin_home_school | nav_admin_home_school | NULL | index.html#home-school |
| 7 | 组织管理 | Organization Management | nav_admin_org | nav_admin_org | NULL | index.html#org |
| 8 | 内容发布 | Content Publishing | nav_admin_content | nav_admin_content | NULL | index.html#content |
| 9 | 成长册设置 | Growth Book Setting | nav_admin_growth_book | nav_admin_growth_book | NULL | index.html#growthbook |
| 10 | 选用模板 | Select Template | btn_admin_gb_select_template | db_school | template_code FROM layout manifest | set book_template_code |
| 11 | 上传园所 Logo | Upload Logo | btn_admin_gb_upload_logo | db_file | local file | upload and return file_id |
| 12 | 上传封面照片 | Upload Cover Photo | btn_admin_gb_upload_cover | db_file | local file | upload and return file_id |
| 13 | 园长寄语 · 通用 | Principal Message Default Tab | btn_admin_gb_msg_default | db_school | NULL | local tab |
| 14 | 园长寄语 · 小班 | Principal Message K1 Tab | btn_admin_gb_msg_k1 | db_school | NULL | local tab |
| 15 | 园长寄语 · 中班 | Principal Message K2 Tab | btn_admin_gb_msg_k2 | db_school | NULL | local tab |
| 16 | 园长寄语 · 大班 | Principal Message K3 Tab | btn_admin_gb_msg_k3 | db_school | NULL | local tab |
| 17 | 预览整册 | Preview Book | btn_admin_gb_preview | db_admin_growth_book_setting_home | current draft values | open preview modal |
| 18 | 保存草稿 | Save Draft | btn_admin_gb_save_draft | db_school | validated form + file_id | 仅 d1 可 persist；d2 后隐藏 |
| 19 | 发布 | Publish Setting | btn_admin_gb_publish | db_school + db_school_book_section + db_file_ref | validated complete candidate | 唯一一次单事务发布为 d2；之后只读 |
| 20 | 园所介绍 · 确认 | Commit Intro | btn_admin_gb_commit_intro | db_school | intro textarea | 仅 d1 写草稿；d2 后只读 |
| 21 | 园长寄语 · 确认 | Commit Message | btn_admin_gb_commit_msg | db_school | current grade + textarea | 仅 d1 写草稿；d2 后只读 |
| 22 | ＋ 新增栏目 | Add School Section | btn_admin_gb_sec_add | db_school_book_section | NULL | open section modal |
| 23 | 栏目照片 · 上传 | Upload Section Photo | btn_admin_gb_sec_upload | db_file | local file, slot index | upload and return file_id |
| 24 | 栏目 · 保存 | Save Section | btn_admin_gb_sec_save | db_school_book_section | validated form | create/update |
| 25 | 栏目 · 删除 | Delete Section | btn_admin_gb_sec_delete | db_school_book_section | school_book_section_id | delete and re-anchor draft children |
| 26 | 栏目 · 取消 | Cancel Section | btn_admin_gb_sec_cancel | db_school_book_section | NULL | close modal |
| 27 | 园所介绍照片 · 上传 | Upload Intro Photo | btn_admin_gb_intro_upload | db_file | local file, slot index | upload and return file_id |
| 28 | 园长寄语照片 · 上传 | Upload Message Photo | btn_admin_gb_msg_upload | db_file | local file, slot index | upload and return file_id |


[FORM_FIELD_INDEX]

| visible field | canonical target | required |
|---|---|---:|
| 模板 | db_school.book_template_code | 1 (发布前置) |
| 园所名称（只读） | db_school.school_name | —— |
| 园所 Logo | db_school.logo_file_id -> db_file.file_id | 0 |
| 封面照片 | db_school.book_cover.image_file_id -> db_file.file_id | 0 |
| 封面标题文字 | db_school.book_cover.title_text | 0 |
| 园所介绍 · 版式 | db_school.intro_layout_code | 1 |
| 园所介绍 · 正文 | db_school.school_intro | 1 (发布前置) |
| 园所介绍 · 照片 | db_school.intro_file_id | 草稿 0:k；发布须传满 intro_layout_code 声明的槽位数 |
| 园长寄语 · 通用 | db_school.principal_message.default | 0 |
| 园长寄语 · 小/中/大班 | db_school.principal_message.k1/k2/k3 | 0 |
| 园长寄语 · 版式 | db_school.message_layout_code | 1 |
| 园长寄语 · 照片 | db_school.message_file_id | 草稿 0:k；整节有文字时发布须传满 message_layout_code 声明的槽位数 |
| 栏目名称 | db_school_book_section.name | 1 |
| 栏目版式 | db_school_book_section.layout_code | 1 |
| 插入位置 | db_school_book_section.anchor_after | 1 |
| 栏目标题文字 | db_school_book_section.heading_text | 0 |
| 栏目正文 | db_school_book_section.body_text | 草稿 0；版式声明 body=1 时发布必填。版式声明 body=0 时不出该控件 |
| 栏目照片 | db_school_book_section.file_id | 草稿 0:k；发布须传满 layout_code 声明的槽位数。槽位数不可写 |

commit_rule (确认的语意) = book_setting_status=d1 时，园所介绍与园长寄语的「确认」可写入服务器草稿，右侧预览随输入即时更新；d2 后页面全部只读，不再形成本机待发布候选
autosave_on_tab_switch = 仅 d1 可按草稿规则自动写入并 toast；d2 不调用任何内容写入 API
publish_commits_drafts = 唯一一次发布使用当前完整候选包做全量校验，并以单一事务从 d1 转 d2；发布成功后永久锁定

placeholder_fill_rule = 后端把以上取值嵌入所选 template 的同名占位符；管理端不产生版面，只产生内容
cover_title_fallback = title_text 为空时取 db_school.school_name + template 自带的默认后缀，不由客户端拼接
upload_constraint = logo、封面照片、预设两节照片、栏目照片一律走 db_file 通用上传；本 spec 不新增文件表，也不为美术素材开上传口
temporary_upload_rule (F16) = 仅 d1 接受上传并建立草稿引用；替换／取消只改本草稿引用，物理文件按全局零引用与 render lease 规则清理。d2 后拒绝上传或替换，不存在更新发布暂存分支
image_pipeline (Q62-j63) = 每档原始 bytes 最多 10MB；服务器实判格式、校正方向、移除 EXIF／定位等 metadata。普通照片按实际 layout 槽位比例裁切，长边最多 2000px，转 MozJPEG q82—85；Logo 可保留透明 PNG
resolution_rule = 成长册以 App 内／电子 150 DPI renderer 观看为目标，不以家长下载后打印 hardcopy 为设计目标。不要求 300 DPI、出血、CMYK 或整张 A4 的输入分辨率；最低像素只按 1240 × 1754 renderer 中实际槽位尺寸推导，小图不为纸本列印放大。固定 sRGB；文件体积仍是电子分享的实际约束


[DYNAMIC_CONTENT_NODE]

| node_name_cn | node_key | object | input | cardinality | jump/action |
|---|---|---|---|---|---|
| 模板卡片 | admin_gb_template_card | layout library manifest | NULL | 1:k | select template_code |
| 栏目版式卡片 | admin_gb_section_layout_card | layout library manifest | NULL | 1:k | select layout_code（园所栏目出四种，预设两节只出带正文的三种） |
| 预设节图位 | admin_gb_preset_photo_slot | db_file | slot index FROM layout_code | 0:k | upload |
| 园所栏目行 | admin_gb_section_row | db_school_book_section | school_book_section_id FROM query_result | 0:k | NONE |
| 栏目编辑 | admin_gb_section_edit | db_school_book_section | school_book_section_id FROM query_result | 0:k | open section modal |
| 预览页 | admin_gb_preview_page | db_school + db_school_book_section + layout library | current draft values | 0:k | render only |

dynamic_rule = 模板代码、栏目版式代码、名称与缩略图必须来自版式库 manifest 接口；栏目行的 school_book_section_id 必须来自 API，不得以栏目名称作为编辑/删除键


[PAGE_OBJECT]

管理端成长册设置首页 (Admin Growth Book Setting Home / db_admin_growth_book_setting_home)

admin_growth_book_setting_home_id (页面聚合ID), 1:1, integer, ui=admin_growth_book_setting.page
admin_id (管理员ID), 1:1, integer, ui=context.hidden
school_id (园所ID), 1:1, integer, ui=context.hidden
admin_school_id (授权ID), 1:1, integer, ui=context.hidden
logo_file_id (园所LogoID), 0:1, integer, ui=admin_growth_book_setting.logo
cover_file_id (封面照片ID), 0:1, integer, ui=admin_growth_book_setting.cover
school_book_section_id (园所栏目ID), 0:k, integer, ui=admin_growth_book_setting.sections

rel_count (关系数量) = 6
rel_db (关联表) = db_admin, db_school, db_admin_school, db_file, db_class, db_school_book_section
rel_map (关系字段) = db_admin_growth_book_setting_home{admin_id}<->db_admin{admin_id}; db_admin_growth_book_setting_home{school_id}<->db_school{school_id}; db_admin_growth_book_setting_home{admin_school_id}<->db_admin_school{admin_school_id}; db_admin_growth_book_setting_home{logo_file_id}<->db_file{file_id}; db_admin_growth_book_setting_home{cover_file_id}<->db_file{file_id}; db_admin_growth_book_setting_home{school_book_section_id}<->db_school_book_section{school_book_section_id}
class_ref_note = db_class 只用于预览时按 grade 取园长寄语变体，页面聚合不持有 class_id
persist = 0
object_type = aggregate


[CONTENT_OBJECTS]

园所 (School / db_school) [REUSE + EXTEND]

reuse_source (复用来源) = Teacher App home-spec.md (canonical definition)
引用字段 = school_id, school_name (ui=growth_book_setting.school_name_readonly), school_status
本 spec 落点字段 (DDL 见 CANONICAL_FIELD_EXTENSION):

school_intro (园所介绍), 0:1, max_len=1000, ui=growth_book_setting.intro_input
logo_file_id (园所LogoID), 0:1, integer, ui=growth_book_setting.logo_upload
book_template_code (成长册模板代码), 0:1, string, ui=growth_book_setting.template_select
book_cover (封面配置), 0:1, json {image_file_id, title_text}, ui=growth_book_setting.cover_upload|growth_book_setting.cover_title
intro_layout_code (园所介绍版式代码), 1:1, string, ui=growth_book_setting.intro_layout_select
intro_file_id (园所介绍照片ID), 0:k, integer, ui=growth_book_setting.intro_photo_upload
principal_message (园长寄语), 0:1, json {default, k1, k2, k3}, ui=growth_book_setting.principal_message
message_layout_code (园长寄语版式代码), 1:1, string, ui=growth_book_setting.message_layout_select
message_file_id (园长寄语照片ID), 0:k, integer, ui=growth_book_setting.message_photo_upload
book_setting_status (成长册设置状态), 1:1, d1=draft(草稿)|d2=published(已发布), ui=growth_book_setting.status
book_setting_revision (成长册草稿并发版本), 1:1, bigint, server-generated monotonic for d1 writes and first publish, ui=context.hidden
book_setting_published_at (发布时间), 0:1, datetime, server-generated on the only successful publish, ui=growth_book_setting.published.time

rel_count (关系数量) = 1
rel_db (关联表) = db_file
rel_map (关系字段) = db_school{logo_file_id}<->db_file{file_id}; db_school{book_cover.image_file_id}<->db_file{file_id}

design_ownership_rule (美术归属 / 2026-08-03 前端评审，收窄 W19 推翻范围):
封面与内页的美术（边框、底纹、配色）由我们提供的 1-2 套 template 承载，仍在后端仓库的页版式库里，园所不上传、不编排
园所填的是内容占位符：logo、整体照片、园所名称、园所介绍、园长寄语
DECISIONS.md W1b「若要推翻，请明说，那会多一个 admin 端的素材管理入口」不触发 —— 本 spec 不开素材管理入口
连带成立：红线 5 不动、分辨率/透明通道的品控门槛不转嫁给园所、W13「版面品质由我们负责」原样保留

template_source_rule (模板取值来源):
book_template_code 的合法取值来自后端仓库版式库的 manifest（db/layout/*.json），随部署走、跟 git 走 code review
管理端只存代码，不存版面与美术素材；理由见 layout-spec 7.3.5 与红线 5
版式库当前尚未建立（db/layout/ 目录不存在），manifest 接口与本页面同批落地

cover_layout_id_note (登记不改判):
W19 的 book_cover JSON 原含 layout_id。本 spec 把它上提为 book_template_code —— 封面与内页是同一套 template，不再单选封面版式
上提后 book_cover 只剩 image_file_id 与 title_text 两个标量，JSON 的价值已消失，建议 DDL 落地时拆成两列
本 spec 不自行改判，登记待后端确认

preset_section_layout_rule (预设两节可选版式 / 2026-08-03 前端评审，扩充本 spec 初版):
园所介绍与园长寄语初版只有纯文字，实际这两节都需要配图。改为与园所栏目**同一套版式**：从版式库选一个，只填占位符
**只出带正文的三种**（纯文字 / 上图下文 / 两图并排）。这两节以文字为主，「整页大图」没有正文位，给了也用不上
默认 photo（上图下文）—— 一张图配一段文字是这两节最常见的形态
照片槽位数由 layout_code 声明，**不是可写字段、不落列**，同 school_section_layout_rule
三处预览（填写区旁、整册预览、教师端渲染）走同一个渲染器，形状必定一致

preset_layout_not_grade_scoped_rule (版式与照片不按年级分叉):
园长寄语的**文字**按年级分叉（见下条），但**版式与照片整节一份**，四个年级共用
理由与 no_grade_section_rule 同源：分叉只发生在文字层。版式一旦也按年级分叉，就等于四套版面，
册子会因班级年级不同而长得不一样 —— 那正是「园所级 = 全园一样」这条定义要排除的情况
若某年级确实需要不同的图，那是**班级级内容**，走教师端的新增栏目，不在本节

principal_message_rule (园长寄语的年级变体 / 2026-08-03 前端评审):
教师端按 db_class.grade 取 k1|k2|k3；该键为空回落 default；default 亦为空则该占位符不渲染，不留空框
分叉只发生在文本层，不产生第二层模版、不产生 override 表、不产生班班版面分歧
先例：db_resource.grade SET('k1','k2','k3')「适用年级(多选)」，本仓库已有同类模式

no_grade_section_rule (按年级差异的栏目不由管理端下发 / 2026-08-03 前端评审):
「入学第一天」「最后一天」「毕业典礼」这类逐年级不同的栏目仍归班级，由教师自行新增（DECISIONS.md W19 原文举的就是这两个例子）
管理端不提供年级级新增栏目配置。理由：db_growth_book_template 唯一键是 class_id，admin 再下发一层年级级模版会产生两层模版的合并问题（园所下发的栏目老师能不能删/能不能改版面/改了算谁的），正是 W13 明确绕开的那条路
园所若要给毕业班统一致辞，走 principal_message_rule 的年级变体，不把整个栏目收上来
配套（教师端连带，不在本 spec 范围）：页版式库应预置若干「可选栏目版式」，教师新增栏目时从库中选而非从空白网格排，以免每个大班各排一遍且品质参差

no_snapshot_rule (不引入模版快照，F16):
本 spec 不为 db_growth_book_template 或 db_growth_book 增加任何皮肤/设置快照列
理由一：美术与版面仍在 repo，只随发版变，不存在运行时漂移
理由二：园所内容仅首次 d2 发布一次，之后永久唯读；既有 g2 与新册读取同一份 canonical
理由三：无需用逐册快照弥补运行时覆盖，也不引入设置版本表
published_only_addendum (F16) = 成长册只读取唯一合法 d2。d1 草稿不进入教师预览、生成、家长查看或重导；d2 后不得撤回、更新发布、删除栏目或替换附件

publish_gate_rule (发布即分发 / 2026-08-03 前端评审):
book_setting_status=d1 时教师端取不到模板，全园 can_generate=0；d2 才分发，且 d2 永久锁定
这是「管理端统一配置公共内容 → 分发给各班教师个性化填充」在数据层的落点
首次发布必须在单一事务内写入 db_school 成长册字段、全部 db_school_book_section 及其文件引用；任一校验或写入失败都保持 d1
d2 后永久禁止撤回或普通业务修改，不以是否已有 g2 作为截止点

can_generate_extension (对 05 home-school-spec.md can_generate_rule 的补充):
班级级前置条件增加一条：db_school.book_setting_status=d2
原有的「intro 取 db_school.school_intro 非空」由本 spec 的发布必填保证，不重复校验


园所栏目 (School Book Section / db_school_book_section)

school_book_section_id (园所栏目ID), 1:1, integer, ui=school_book_section.hidden
school_id (园所ID), 1:1, integer, ui=context.hidden
name (栏目名称), 1:1, max_len=10, ui=school_book_section.name_input
layout_code (栏目版式代码), 1:1, string, ui=school_book_section.layout_select
anchor_after (插入位置), 1:1, 'cover'|预设 section_key|另一个 school_book_section_id, ui=school_book_section.anchor_select
heading_text (标题文字), 0:1, max_len=20, ui=school_book_section.heading_input
body_text (正文), 0:1, max_len=300, ui=school_book_section.body_input
file_id (栏目照片ID), 0:k, integer, ui=school_book_section.photo_upload
created_by_admin_id (创建管理员ID), 1:1, integer, ui=context.hidden
created_at (创建时间), 1:1, datetime, ui=context.hidden

rel_count (关系数量) = 3
rel_db (关联表) = db_school, db_file, db_admin
rel_map (关系字段) = db_school_book_section{school_id}<->db_school{school_id}; db_school_book_section{file_id}<->db_file{file_id}; db_school_book_section{created_by_admin_id}<->db_admin{admin_id}

school_section_scope_rule (为什么不复用 db_growth_book_section / 2026-08-03 前端评审):
db_growth_book_section 的 row_scope 明写「只存新增栏目（班级级）」，template_id 是 1:1 NOT NULL、随班级模版 d1/d2 冻结、其 widget 走 db_book_widget、槽位可向家长征集
园所栏目四项都不成立：无 template_id、无班级冻结、无 widget 网格、**不向家长征集**。复用只能靠把 template_id 改成可空再加一个 scope 判别列，等于制造一张多态表
两表在渲染层合流即可 —— anchor_after 本来就要加 anchor_type 判别（见班级端 spec 的 anchor_after_extension），园所栏目只是多一个取值类别，不是新机制

school_section_layout_rule (版面归属 / 与 design_ownership_rule 同源):
园所栏目**不提供自由网格编辑器**。园所从版式库提供的若干栏目版式中选一个（纯文字 / 上图下文 / 两图并排 / 整页大图），只填占位符
理由一：与本 spec 的 design_ownership_rule 一致 —— 刚把美术与版面留在我们手上，不能在同一页上开一个自由排版入口
理由二：W13「版面品质由我们负责」在园所级更强 —— 园所栏目出现在全园每一本册子里，排版事故的影响面是单个班级的 N 倍
理由三：教师端那套编辑器（网格 / 重叠校验 / 字数推导 / run 阵列富文本）是移动端形态，管理端重做一套是纯重复
照片槽位数由 layout_code 声明，**不是可写字段**，不落列（同班级端 slot_count_derived 的处理）

school_section_readonly_rule (教师端只读 / W13 的一贯原则):
园所栏目下发后教师端**只读**：不可编辑、不可删除、不可改插入位置，只在成长册内容清单里显示为一行只读项
这正是不引入 override 表的前提 —— 一旦教师能改，就要回答「园所更新了栏目谁跟得到谁跟不到」，与 W13 绕开的那条路同构
园所栏目不进教师端的生成检查表：内容由园所填定，不存在「该幼儿是否齐备」的判定

school_section_anchor_rule (锚点层级):
园所栏目的 anchor_after 取值 = 'cover' | 预设 section_key | 另一个园所栏目的 school_book_section_id；**不得锚到班级级栏目**（园所不知道各班有什么）
反向成立：班级级栏目可以锚到园所栏目 —— 园所栏目对全园可见。规则是**下层可锚上层，上层不可锚下层**
成环挡在选项层（排除自己与所有递归锚定到自己的栏目），**服务端必须重做一次校验**，前端 UI 不是完整性边界
删除 d1 栏目时不留孤儿：只把同一 d1 草稿内锚在被删栏目之后的园所栏目改锚到最近仍存在的上游锚点
d1 尚未分发，班级栏目与 g2 不得引用它；d2 后栏目不可删除，故不递归修改已冻结班级模板
渲染端多轮扫描落位，仍解不开的（数据层的环或悬空锚点）一律落到册尾，不得让栏目消失

publish_concurrency_rule (F16) = 页面读取 d1 时返回 db_school.book_setting_revision；草稿写入与首次发布带该值做 CAS 并在成功后 revision+1。陈旧请求 409，保留本机候选供对照并要求重载，不自动合并、不 LWW；若另一管理员已先发布为 d2，候选作废并切换只读
publish_idempotency_rule (F16) = 唯一一次发布必须带幂等键；同键重放返回原结果，不重复写入、revision+1 或发布时间。不同键并发时只有第一个 d1→d2 成功，后来者 409
schoolwide_impact_rule (F16) = 只有 d1 草稿可删除栏目；影响只统计同一 d1 内直接／递归锚点并作最小修复。d2 后删除拒绝，不扫描班级模板、幼儿或既有 g2，不需要全园 impact fingerprint
content_safety_rule (F16) = 唯一一次发布前，管理员必须完整预览园所介绍、园长寄语、封面、Logo、园所栏目文字与全部图片，明确点击发布即完成当次人工把关。不调微信 API，不建另一 admin 复核、待审状态、queue 或 review action
full_preview_rule (Q62-j69) = 同一 candidate hash 下必须逐页预览 default／k1／k2／k3 全部适用版本；最终确认显示文字／图片／栏目总数。只校验本次明确确认与当前 candidate 一致，不建审核证据、操作或版本历史
low_pixel_rule (Q62-j70) = 依 1240 × 1754 renderer 的实际槽位计算建议像素；不足只显示清晰度提示，不阻挡发布、不自动插值放大。10MB、裁切、metadata 清理、长边上限与 JPEG／PNG 规则继续执行
publish_time_rule (F16) = 唯一一次发布成功事务写 db_school.book_setting_published_at=current business time，UI 固定称「发布时间」；校验失败、409、草稿上传与本机候选变化不改。不建发布历史，actor 只进既有 B2 系统日志且不得记录内容

preset_section_completeness_rule (预设两节的齐备判定):
与园所栏目同一套判定，作用于发布这一步，不挡保存草稿
园所介绍：正文本来就是发布必填；版式声明了照片槽位则须传满
园长寄语：**整节为空（通用与三个年级都没写）是合法的** —— 该占位符不渲染、不留空框，此时不检查图位；
只有整节有文字时才要求传满槽位
纯文字版式两节都不要求照片

school_section_completeness_rule (内容齐备才能发布 / 2026-08-03 前端评审):
园所栏目缺内容 = 版式声明了 N 个照片槽位却未传满，或 版式声明 body=1 而 body_text 为空
缺内容的栏目会在册子上印出空白框，因此**挡在发布这一步**，不挡保存草稿 —— 允许先建栏目占位、之后补齐
判定只看版式声明的槽位：纯文字版式不要求照片，整页大图版式不要求正文，不得一律要求全字段
提示强度：栏目行上一枚 badge（如「缺 1 张照片、正文」）+ 发布时一条 toast 点名第一个不齐备的栏目。不使用弹层或阻断式横幅 —— 管理者看一眼就知道少什么
heading_text 为空不算缺内容：回落取 name，不会印出空白

school_section_no_collection_rule (不向家长征集):
园所栏目没有 binding_key=collected 的等价物，不产生家长端待办，不写 db_book_material_submission
它是「填定即确定」的内容，与班级级新增栏目的征集流程完全不同 —— 后端不得复用同一套征集/齐备判定


[REUSED_OBJECT_USAGE]
db_file = REUSE shared file metadata；logo 与封面照片各占一条
db_class = REUSE identity object；仅按 grade 取园长寄语变体，管理端不写
db_growth_book_template|db_growth_book = REUSE Teacher App 05 home-school-spec.md；管理端只读、不写这两张表
layout library = 后端仓库资产，不是 db_* 对象，不进 canonical registry


[CANONICAL_FIELD_EXTENSION]

db_school ADD school_intro(0:1, max_len=1000) —— DECISIONS.md B12 已定，DDL 未落地，本 spec 提供第一个写入面
db_school ADD book_cover(0:1, json {image_file_id, title_text}) —— DECISIONS.md W19 已定，DDL 未落地，本 spec 解掉 GAPS.md G32
db_school ADD logo_file_id(0:1, integer) —— 本 spec 新增
db_school ADD book_template_code(0:1, string) —— 本 spec 新增
db_school ADD principal_message(0:1, json {default,k1,k2,k3}) —— 本 spec 新增
db_school ADD book_setting_status(1:1, d1=draft|d2=published, default d1) —— 本 spec 新增
db_school ADD book_setting_revision(1:1, bigint, default 0, server-generated monotonic) —— F16 d1 草稿／首次发布 CAS
db_school ADD book_setting_published_at(0:1, datetime, nullable, server-generated on successful publish) —— F16 唯一发布时间
db_school ADD intro_layout_code(1:1, string, default photo) —— 本 spec 新增
db_school ADD intro_file_id(0:k, integer) —— 本 spec 新增
db_school ADD message_layout_code(1:1, string, default photo) —— 本 spec 新增
db_school ADD message_file_id(0:k, integer) —— 本 spec 新增
db_school_column_growth_note (登记不改判) = 本 spec 累计为 db_school 增 11 列，其中 9 列是成长册专属。DDL 落地时可考虑收进一个 JSON 列或独立的 db_school_book_setting 表，避免身份表被单一功能撑大。**管理端不自行改判**，因为 W19 与 B12 已经把 book_cover 与 school_intro 定为 db_school 的直接列，改动会推翻两条已定决议
preset_section_file_note (0:k 的落列) = intro_file_id 与 message_file_id 的 0:k 走 db_file_ref（owner_object=db_school + usage_key）还是本表多列，未定；与 db_school_book_section.file_id 应取同一种做法
extension_rule = extend the canonical db_school only; app-prefixed school/setting copies are FORBIDDEN
preset_section_rule = 不新增预设栏目。园长寄语作为封面/扉页 template 的占位符，与 logo、园所名称、简介同层；预设 6 项的固定顺序（E3-1）不动
naming_rule = 园长寄语 principal_message 与预设第 6 项「教师寄语」db_teacher_message 是两个对象，不得合并或互相取值


[EMPTY_STATE]

IF book_template_code 未选, 预览返回 [] AND empty_title=请先选用模板
IF logo|封面照片 未上传, 对应占位符留白, 不阻挡保存草稿
IF principal_message 全部键为空, 该占位符不渲染
IF book_setting_status=d1, 教师端模板接口返回 [] AND empty_title=园所尚未发布成长册设置
IF layout manifest 为空, 返回 [] AND empty_title=版式库尚未部署
IF 无园所栏目, 返回 [] AND empty_title=还没有园所栏目
IF 园所栏目的照片槽位未上传, 该槽位在预览中留白, 不阻挡保存草稿
IF 园所栏目 body_text 为空 AND 版式声明 body=1, 预览显示占位提示, 不阻挡保存草稿


[NAV_OBJECTS]

shared_navigation_source = dashboard-spec.md [NAV_OBJECTS]
required_namespace = nav_admin_*
nav_addition = nav_admin_growth_book / db_admin_growth_book_setting_home / index.html#growthbook / rel_count=0
nav_count_change = 8 -> 9；dashboard-spec.md [NAV_OBJECTS] 与其余七份 spec 的 [STATIC_BUTTON_NODE_INDEX] 须同步加行，各自 static_node_count +1


[JUMP_VALIDATION]

IF book_template_code NOT_IN layout manifest, return 400
IF file_id not uploaded/authorized for current admin and school, return 403
IF 写入任一 db_school 本节字段 AND role!=admin, return 403
IF admin lacks growth_book.setting.write, return 403
IF 请求发布 AND book_template_code 为空 OR school_intro 为空, return 422
IF 请求发布 AND 任一 db_school_book_section 的声明照片槽位未传满或声明的 body_text 为空, return 422 并回传该 section_id 与缺失项
IF 撤回发布, return 409；d2 永久锁定
IF 请求保存服务端草稿 AND book_setting_status!=d1, return 409
IF 请求发布的 book_setting_revision != db_school.book_setting_revision, return 409 AND keep local candidate; no partial write
IF 请求发布幂等键已成功处理, return original result; MUST NOT write or increment revision again
IF revision 冲突 AND current status=d1, return current draft metadata AND preserve local candidate for manual comparison
IF revision 冲突 AND current status=d2, return 409 AND discard publish candidate; page becomes readonly
IF 请求发布前未完成本次全内容预览与明确确认, return 422
IF candidate hash 未逐页完成 default|k1|k2|k3 全部适用版本预览, return 422
IF 删除园所栏目 AND book_setting_status!=d1, return 409
IF principal_message 出现 default|k1|k2|k3 以外的键, return 400
IF layout_code NOT_IN layout manifest 的栏目版式清单, return 400
IF intro_layout_code|message_layout_code NOT_IN layout manifest 中声明 body=1 的版式, return 400
IF 请求发布 AND intro_layout_code 声明的照片槽位未传满, return 422
IF 请求发布 AND principal_message 任一键非空 AND message_layout_code 声明的照片槽位未传满, return 422
IF 上传照片数超过对应 layout_code 声明的槽位数, return 422
IF school_book_section_id NOT_FROM authorized current_school query, return 404
IF db_school_book_section.school_id != current_school_id, return 403
IF anchor_after 指向班级级 db_growth_book_section, return 400
IF anchor_after 成环（含递归）, return 409；服务端必须独立校验，不得依赖前端选项过滤
IF 删除园所栏目, 锚在其后的栏目 MUST 在同一事务内改锚到被删栏目原本的 anchor_after
IF 删除园所栏目, 直接或递归锚向它的同一 d1 草稿园所栏目 MUST 在同一事务改到最近存活上游锚点
IF 教师端请求写入 db_school_book_section, return 403
IF 上传照片数超过 layout_code 声明的槽位数, return 422
current_school_id MUST be backend-derived; client school_id is ignored
首次发布与全部文件引用 MUST be transactional；发布事务须重验每个 d1 file 的 uploader admin／current school 授权，不得留下未授权 file_id
