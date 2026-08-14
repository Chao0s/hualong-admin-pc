ADMIN_ORGANIZATION_BACKEND_OBJECT_SPEC

scope (范围) = index.html#org
source_page (参考页面) = index.html
static_node_count (固定按钮节点数) = 17
dynamic_teacher_action_count (教师行动态操作数) = 0:2t
dynamic_class_action_count (班级行动态操作数) = 0:c
dynamic_child_action_count (幼儿行动态操作数) = 0:h
runtime_clickable_node_count (运行时可点击节点数) = 17:(17+2t+c+h)
field_format (字段格式) = field_key (中文字段名), cardinality, type|enum, ui
id_rule (ID规则) = integer, database_auto_generated
null_rule (空值规则) = 0:1
list_rule (列表规则) = 0:k | 1:k


[SHARED_OBJECT_RULE]

canonical_registry = docs/spec-handoff.md
shared_object_source = dashboard-spec.md; review-spec.md; Teacher App home-spec.md|home-school-spec.md
shared_objects = db_admin, db_admin_school, db_school, db_teacher, db_teacher_profile, db_class, db_teacher_class, db_child, db_upload, db_growth_record, db_assessment
reserved_identity_objects_reused = db_parent, db_parent_child (canonical = Parent App home-spec.md; 管理端仅 REUSE，不得重复定义)
admin_page_aggregate = db_admin_org_home
identity_alias = FORBIDDEN


[CONTEXT_RULE]

admin_id = auth_session.admin_id
allowed_school_id = db_admin_school.school_id WHERE admin_id=current_admin_id AND is_active=1
current_school_id MUST IN allowed_school_id
permission = db_admin_school.role|permission_scope
required_permission = org.read|org.teacher.write|org.class.write|org.child.write|org.account.reset according to action
all raw admin_id|school_id|teacher_id|class_id|child_id|parent_id ui = context.hidden
client_editable = 0; selectors submit opaque values that backend revalidates
backend_authorization_validation = REQUIRED
pii_rule = phone/contact fields require least-privilege permission and masked list output


[DATA_INITIALIZATION_RULE]

prototype_content = HTML 中六名教师、六个班级、六名幼儿、电话、家长、人数、累计上传、测评完成、档案状态及表单默认 30 人均为 Mock
static_ui_content = 三个组织分类、岗位/年级/性别选项、表头、操作说明和空状态文案
business_seed = NONE
base_identity_data = real db_school|db_admin|db_admin_school|db_teacher|db_class|db_teacher_class|db_child|db_parent|db_parent_child may be provisioned by deployment or authorized admin
dynamic_list_without_data = []
dynamic_count_without_data = 0
unassigned_or_unstarted_status = not_started
new_class_business_state = no roster -> child_count=0; no assessment -> assessment_status=not_started
new_teacher_upload_count = 0
new_child_growth_record_status = not_started
hardcoded_business_id = FORBIDDEN
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
| 10 | 导出数据 | Export Data | btn_admin_export | db_admin_org_home | current tab/filter | file download |
| 11 | 教师 | Teachers Tab | btn_admin_org_teacher | db_teacher | NULL | local/API tab |
| 12 | 班级 | Classes Tab | btn_admin_org_class | db_class | NULL | local/API tab |
| 13 | 幼儿 | Children Tab | btn_admin_org_child | db_child | NULL | local/API tab |
| 14 | 搜索 | Search | btn_admin_org_search | db_admin_org_home | query_text | local/API filter |
| 15 | 新增教师/班级/幼儿 | Add Entity | btn_admin_org_add | selected canonical object | selected tab | open modal |
| 16 | 取消 | Cancel | btn_admin_org_cancel | selected canonical object | NULL | close modal |
| 17 | 保存 | Save | btn_admin_org_save | selected canonical object | validated form | create entity/relation |


[FORM_FIELD_INDEX]

| tab | visible field | canonical field/action | required |
|---|---|---|---:|
| teacher | 姓名 | db_teacher.teacher_name | 1 |
| teacher | 所在班级 | db_teacher.class_id | 1 |
| teacher | 职务 | db_teacher_profile.job_role | 1 |
| teacher | 联系电话 | db_teacher.phone | 1 |
| class | 班级名称 | db_class.class_name | 1 |
| class | 年级 | db_class.grade | 1 |
| class | 带班教师 | db_teacher.class_id + db_teacher.assignment_role=r1 | 0; “待分配” creates no relation |
| class | 计划幼儿数 | db_class.planned_child_count | 1 |
| child | 幼儿姓名 | db_child.child_name | 1 |
| child | 所在班级 | db_child.class_id | 1 |
| child | 性别 | db_child.gender | 1 |
| child | 家长联系人 | db_parent + db_child.caretakers | 1; production UI/API MUST submit structured parent_name and phone |

b5_b1_correction (2026-08-14 更正) = 上表原有四行引用已被决议拔掉的表：`db_teacher_class`（B5 —— 一位教师只属一个班，关系塌回 db_teacher.class_id + assignment_role 两个纯量列）与 `db_parent_child`（B1 —— 改 db_child.caretakers JSONB）。四行已按落地后的列改写。本节仍有三处未清的同类残留，属存量：[SHARED_OBJECT_RULE] 的 shared_objects 与 [DATA_INITIALIZATION_RULE] 的 base_identity_data 仍列 `db_teacher_class`／`db_parent_child`／`db_upload`（后者由 B10 拔除），[PAGE_OBJECT] 的 teacher_class_id／parent_child_id／upload_id 三栏同理 —— 它们是 persist=0 聚合上的栏位，改动牵涉 rel_map 与 admin_org.* 既有绑定，另批处理

form_ui_note (表单控件标注位置) = 上表说的是「哪一列」，下面 [FORM_WRITE_OBJECTS] 说的是「哪个控件」。两者必须并存：本表已经存在很久，但它是散文，`extract-ui-binding.mjs` 读不到，所以 index.html 的新增表单长期零绑定。标注必须写在管理端 spec 内而不是 Teacher canonical spec 内 —— 抽取器按 spec 所在仓库定 role，而 class_id 在 teacher 角色属 derived、在 admin 角色属 free（scope-rules.json）；写错仓库会让「管理员为幼儿选班级」被判成 §4.6 违规


[FORM_WRITE_OBJECTS]

新增教师表单 (Add Teacher Form / db_teacher)

teacher_name (教师姓名), 1:1, max_len=50, ui=admin_org.teacher.name_input
class_id (所在班级ID), 0:1, integer, ui=admin_org.teacher.class_select|admin_org.class.lead_teacher_select
assignment_role (班级角色), 0:1, r1=lead(主班)|r2=assistant(配班), ui=admin_org.class.lead_teacher_select
phone (联系电话), 1:1, phone, ui=admin_org.teacher.phone_input

canonical_source = REUSE Teacher App home-spec.md db_teacher；本块只登记 index.html#org 新增/编辑教师弹层实际写入的列，不重新定义对象
phone_required (电话必填 / 2026-08-14 更正) = 本行此前写 0:1「选填」，与 DDL 不符：db_teacher.phone 是 NOT NULL + UNIQUE（A2 的跨端身份连结键、G20）。已改 1:1，[FORM_FIELD_INDEX] 的 required 同步改 1，原型 placeholder 从「选填」改为必填并加 maxlength=20。若产品确实要允许暂缺电话，要改的是 DDL 与 A2 身份方案，不能只在本文件写 0:1
lead_teacher_two_columns (带班教师写两列) = admin_org.class.lead_teacher_select 同时出现在 class_id 与 assignment_role 两行上，因为「谁带这个班」这一个动作要写两列：class_id 记「该教师属于本班」、assignment_role=r1 记「他是主班」。此前只绑了 class_id，主班身份没有任何落点，管理端因此没有一个控件能写 assignment_role。B5 把 db_teacher_class 塌进 db_teacher 之后这两列都在 db_teacher 上，所以一个控件写两列是正确的，服务端在同一交易内写
job_role_binding (职务控件) = 弹层的「职务」四个选项（带班教师／配班教师／保育员／教研组长）对应 db_teacher_profile.job_role 的 j1—j4，复用 review-spec.md 既有的 teacher_profile.job_role 标注，不另立第二个路径。它不是 db_teacher.assignment_role —— 后者只有 r1 主班／r2 配班两值（B5 砍掉 r3），装不下四个选项；[FORM_FIELD_INDEX] 那句「and/or db_teacher_class.assignment_role」写在 B5 删表之前，已不成立
school_id_binding (园所归属) = derived，不出现在 data-ui


新增班级表单 (Add Class Form / db_class)

class_name (班级名称), 1:1, max_len=50, ui=admin_org.class.name_input
grade (年级), 1:1, k1=small(小班)|k2=middle(中班)|k3=large(大班), ui=admin_org.class.grade_select
planned_child_count (计划幼儿数), 0:1, integer, ui=admin_org.class.planned_count_input

canonical_source = REUSE Teacher App home-spec.md db_class；planned_child_count 由本 spec 的 [CANONICAL_FIELD_EXTENSION] 引入
lead_teacher_binding (带班教师控件) = 弹层的「带班教师」下拉写的是 db_teacher.class_id，不是 db_class 的任何一列 —— B5 之后教师与班级的关系塌在 db_teacher 上，没有 db_teacher_class 可写。因此 admin_org.class.lead_teacher_select 与 admin_org.teacher.class_select 并列声明在上一块的 db_teacher.class_id 那一行：同一列的两个方向（在教师表单里选班、在班级表单里选教师），两条路径一个落点。选「待分配」不建立任何关系


新增幼儿表单 (Add Child Form / db_child)

child_name (幼儿姓名), 1:1, max_len=50, ui=admin_org.child.name_input
class_id (所在班级ID), 1:1, integer, ui=admin_org.child.class_select
gender (性别), 0:1, g1=female(女)|g2=male(男)|g3=unspecified(未指定), ui=admin_org.child.gender_select

canonical_source = REUSE Teacher App home-spec.md db_child；gender 沿 Parent App child-profile-spec.md 的 canonical 扩展，管理端不得改编码
parent_contact_binding (家长联系人控件) = 原型的单一「如：陈先生 138****0000」自由文本是 Mock，不可作为生产契约（[FORM_FIELD_INDEX] 已要求结构化 parent_name + phone）。它写的是 db_parent 而非 db_child，故标注 admin_org.child.parent_contact_input 挂在 db_parent.parent_name 上；结构化拆分与 relationship_type 必填仍是待办，见 [JUMP_VALIDATION] 的 422 规则


家长联系人 (Parent Contact / db_parent)

parent_name (家长姓名), 1:1, max_len=50, ui=admin_org.child.parent_contact_input

canonical_source = REUSE Parent App home-spec.md db_parent；本块只登记新增幼儿弹层写入的那一列，phone 与 relationship_type 待结构化后再登记


[DYNAMIC_CONTENT_NODE]

| node_name_cn | node_key | object | input | cardinality | jump/action |
|---|---|---|---|---|---|
| 教师行 | admin_org_teacher_row | db_teacher + db_teacher_profile + db_teacher_class | teacher_id FROM query_result | 0:t | NONE |
| 编辑教师 | admin_org_teacher_edit | db_teacher + db_teacher_profile + db_teacher_class | teacher_id FROM query_result | 0:t | update authorized fields |
| 重置密码 | admin_org_teacher_reset_password | identity provider | teacher_id FROM query_result | 0:t | issue reset flow; never return password |
| 班级行 | admin_org_class_row | db_class + db_teacher_class | class_id FROM query_result | 0:c | NONE |
| 幼儿名册 | admin_org_class_roster | db_child | class_id FROM query_result | 0:c | roster detail/modal; no separate spec requested |
| 幼儿行 | admin_org_child_row | db_child + db_parent_child | child_id FROM query_result | 0:h | NONE |
| 成长档案 | admin_org_child_archive | db_growth_record | child_id FROM query_result | 0:h | archive detail/modal; no separate spec requested |

dynamic_rule = 所有名称、电话、计数、状态与 ID 来自授权 API；更新、重置、名册和档案操作必须使用 runtime ID，不得使用姓名/班名


[PAGE_OBJECT]

管理端组织管理首页 (Admin Organization Home / db_admin_org_home)

admin_org_home_id (页面聚合ID), 1:1, integer, ui=admin_org.page
admin_id (管理员ID), 1:1, integer, ui=context.hidden
school_id (园所ID), 1:1, integer, ui=context.hidden
admin_school_id (授权ID), 1:1, integer, ui=context.hidden
teacher_id (教师ID), 0:k, integer, ui=admin_org.teacher.rows
teacher_profile_id (教师专业档案ID), 0:k, integer, ui=admin_org.teacher.role
class_id (班级ID), 0:k, integer, ui=admin_org.class.rows
teacher_class_id (教师班级关系ID), 0:k, integer, ui=admin_org.assignment
child_id (幼儿ID), 0:k, integer, ui=admin_org.child.rows
parent_id (家长ID), 0:k, integer, ui=admin_org.child.contact
parent_child_id (家长幼儿关系ID), 0:k, integer, ui=admin_org.child.contact
upload_id (上传记录ID), 0:k, integer, ui=admin_org.teacher.upload_count
growth_record_id (成长档案ID), 0:k, integer, ui=admin_org.child.archive
assessment_id (测评ID), 0:k, integer, ui=admin_org.class.assessment
name_query (姓名搜索文字), 0:1, max_len=50, ui=admin_org.name_query

filter_binding (搜索控件绑定) = name_query 是查询参数不是列，落在本 persist=0 聚合上：同一个搜索框按当前 tab 分别搜 db_teacher.teacher_name、db_class.class_name、db_child.child_name，写在任一张表上都只对一个 tab 成立。tab 本身是三个按钮不是写入控件，不需要 ui= 标注

rel_count (关系数量) = 13
rel_db (关联表) = db_admin, db_school, db_admin_school, db_teacher, db_teacher_profile, db_class, db_teacher_class, db_child, db_parent, db_parent_child, db_upload, db_growth_record, db_assessment
rel_map (关系字段) = db_admin_org_home{admin_id}<->db_admin{admin_id}; db_admin_org_home{school_id}<->db_school{school_id}; db_admin_org_home{admin_school_id}<->db_admin_school{admin_school_id}; db_admin_org_home{teacher_id}<->db_teacher{teacher_id}; db_admin_org_home{teacher_profile_id}<->db_teacher_profile{teacher_profile_id}; db_admin_org_home{class_id}<->db_class{class_id}; db_admin_org_home{teacher_class_id}<->db_teacher_class{teacher_class_id}; db_admin_org_home{child_id}<->db_child{child_id}; db_admin_org_home{parent_id}<->db_parent{parent_id}; db_admin_org_home{parent_child_id}<->db_parent_child{parent_child_id}; db_admin_org_home{upload_id}<->db_upload{upload_id}; db_admin_org_home{growth_record_id}<->db_growth_record{growth_record_id}; db_admin_org_home{assessment_id}<->db_assessment{assessment_id}
persist = 0
object_type = aggregate


[CONTENT_OBJECTS]

家长 (Parent / db_parent) [REUSE]

reuse_source (复用来源) = Parent App home-spec.md (canonical definition)
引用字段 = parent_id, parent_name (ui=admin_org.child.parent_name), phone (ui=admin_org.child.parent_phone), parent_status
parent_status (家长账号状态) = s1=active(启用)|s2=suspended(暂停)|s3=closed(注销)
school_scope_rule (园所归属规则) = db_parent 不存 school_id；管理端园所过滤经 db_parent_child->db_child.school_id 派生


家长幼儿关系 (Parent-Child Relation / db_parent_child) [REUSE]

reuse_source (复用来源) = Parent App home-spec.md (canonical definition)
引用字段 = parent_child_id, parent_id, child_id, relationship_type (ui=admin_org.child.relationship), is_primary_contact (ui=admin_org.child.primary_contact), is_active
relationship_type (监护关系) = r1=mother(母亲)|r2=father(父亲)|r3=grandparent(祖辈)|r4=guardian(监护人)|r5=other(其他)
unique = parent_id + child_id


[REUSED_OBJECT_USAGE]

db_teacher|db_class|db_teacher_class|db_child = REUSE Teacher App identity objects
db_teacher_profile = REUSE review-spec.md for job_role and department_code
db_upload = REUSE upload workflow; teacher upload count=COUNT(upload_id WHERE teacher_id=row.teacher_id)
db_growth_record = REUSE child growth record; absent record maps to not_started
db_assessment = REUSE assessment; class progress is derived
password_reset = external identity-provider command, not a new business table and not a direct password mutation


[CANONICAL_FIELD_EXTENSION]

db_teacher ADD phone(0:1, phone)
db_class ADD planned_child_count(0:1, nonnegative integer)
db_child.gender = REUSE canonical extension (Parent App child-profile-spec.md): g1=female(女)|g2=male(男)|g3=unspecified(未指定); 管理端不得重复 ADD 或改变编码
extension_rule = extend the same canonical identity objects; app-prefixed teacher/class/child copies are FORBIDDEN


[EMPTY_STATE]

IF selected tab has no real records, return [] AND empty_title=暂无教师|暂无班级|暂无幼儿
IF search has no match, return [] AND empty_title=没有匹配的记录
IF teacher has no upload, upload_count=0
IF class has no assessment, assessment_status=not_started and completed_count=0
IF child has no growth record, growth_record_status=not_started


[NAV_OBJECTS]

shared_navigation_source = dashboard-spec.md [NAV_OBJECTS]
required_namespace = nav_admin_*


[JUMP_VALIDATION]

IF selected tab NOT_IN(teacher,class,child), return 400
IF runtime teacher_id|class_id|child_id NOT_FROM authorized current_school query, return 403
IF runtime object NOT_FOUND, return 404
IF write/reset permission missing, return 403
IF class selector belongs to another school, return 403
IF parent contact is unstructured/invalid or relationship_type missing, return 422
create/update entity and related db_teacher_class|db_parent_child rows MUST be transactional
child_transfer_override (B9/F15/Q62-j18—j46) = 学期中修改 db_child.class_id 不是普通 update。new_class_id=current class 在影响预览前 422。预览只回待删除草稿／终局／active batch、改写 g1／保留 g2 数量；二次确认后正式交易重算，任一数量漂移 409 并要求再确认。真实成功 transition 同交易清理未受旧 g2 保护素材、改班、写 audit，并向当下 caretakers 各发一笔合并 n5；相同 idempotency key 重放返回原成功且不重复，往返转班每次真实 transition 各通知
child_transfer_error_ui = 所有失败明示「未发生任何变更」：数量漂移 409 原框更新数量；g0 409 保留目标班并提示生成结束后重试；同班 422 刷新／禁选；scope/object 404 返回名册；5xx 保留表单。失败不得建 n5／audit 或部分清理
password reset MUST issue a one-time reset channel; hardcoded/default password response is FORBIDDEN
