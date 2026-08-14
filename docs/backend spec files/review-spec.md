ADMIN_REVIEW_CENTER_BACKEND_OBJECT_SPEC

scope (范围) = index.html#review
source_page (参考页面) = index.html
static_node_count (固定操作节点数) = 19
dynamic_review_row_count (动态审核行数) = 0:k
dynamic_row_action_count (动态行操作数) = 0:3k
dynamic_attachment_preview_count (动态附件预览数) = 0:a
runtime_clickable_node_count (运行时可点击节点数) = 19:(19+3k+a)
field_format (字段格式) = field_key (中文字段名), cardinality, type|enum, ui
id_rule (ID规则) = integer, database_auto_generated
null_rule (空值规则) = 0:1
list_rule (列表规则) = 0:k | 1:k


[SHARED_OBJECT_RULE]

canonical_registry (canonical 注册表) = docs/spec-handoff.md
shared_object_source (共享对象来源) = dashboard-spec.md; Teacher App home-spec.md|training-center-spec.md
shared_objects (共享对象) = db_admin, db_admin_school, db_school, db_class, db_child, db_parent, db_teacher, db_file, db_upload, db_resource, db_case, db_training, db_training_participation, db_child_profile_correction
new_canonical_objects_defined_here (本页真正新增对象) = db_teacher_profile, db_teacher_credential, db_teacher_profile_change, db_training_feedback, db_review_action
admin_page_aggregate (管理端页面聚合) = db_admin_review_home
rename_or_duplicate_shared_object (重命名或复制共享对象) = FORBIDDEN


[CONTEXT_RULE]

admin_id = auth_session.admin_id
allowed_school_id = db_admin_school.school_id WHERE admin_id=current_admin_id AND is_active=1
current_school_id MUST IN allowed_school_id
permission = db_admin_school.role|permission_scope
required_permission = review.resource|review.case|review.teacher_profile|review.training_feedback according to tab and target_type
child_profile_permission = 任何有效同园管理端登录身份均可审核，不新增细分 permission；school_id 仍由 session 派生并内联校验
raw_identity_id_ui = context.hidden
client_editable = 0
backend_authorization_validation = REQUIRED


[DATA_INITIALIZATION_RULE]

prototype_content (原型内容) = HTML 中九条资源/案例、三条档案修改、四条研修反馈、教师/班级名、附件、日期、简介、计数和审核状态均为 Mock
static_ui_content (可保留静态内容) = 四个审核分类、搜索提示、表头、审核说明、按钮名称和空状态文案
business_seed = NONE
production_initial_db_teacher_profile_change = EMPTY
production_initial_db_training_feedback = EMPTY
production_initial_db_review_action = EMPTY
dynamic_list_without_data = []
dynamic_count_without_data = 0
unassigned_or_unstarted_status = not_started
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
| 10 | 导出数据 | Export Data | btn_admin_export | db_admin_review_home | current filter | file download |
| 11 | 资源 | Resources Tab | btn_admin_review_resource | db_upload + db_resource | target_type=resource | local filter |
| 12 | 案例 | Cases Tab | btn_admin_review_case | db_upload + db_case | target_type=case | local filter |
| 13 | 教师资料 | Teacher Profile Tab | btn_admin_review_profile | db_teacher_profile_change | target_type=teacher_profile_change | local filter |
| 14 | 研修反馈 | Training Feedback Tab | btn_admin_review_feedback | db_training_feedback | target_type=training_feedback | local filter |
| 15 | 搜索 | Search | btn_admin_review_search | db_admin_review_home | query_text | local/API filter |
| 16 | 幼儿资料 | Child Profile Correction Tab | btn_admin_review_child_profile | db_child_profile_correction | target_type=child_profile_correction | local filter |
| 17 | 待审核／已处理 | Correction Status Tabs | btn_admin_review_child_status | db_child_profile_correction | correction_status | API filter |
| 18 | 班级筛选 | Correction Class Filter | btn_admin_review_child_class | db_child_profile_correction | class_id | API filter |
| 19 | 结果筛选 | Correction Result Filter | btn_admin_review_child_result | db_child_profile_correction | all|approved|rejected | API filter |


[DYNAMIC_CONTENT_NODE]

| node_name_cn | node_key | object | input | cardinality | action |
|---|---|---|---|---|---|
| 待审核行 | admin_review_row | db_upload+db_resource|db_upload+db_case|db_teacher_profile_change|db_training_feedback|db_child_profile_correction | target_type, object_id FROM query_result | 0:k | NONE |
| 查看 | admin_review_view | same object as row | object_id FROM query_result | 0:k | open detail modal |
| 通过 | admin_review_approve | same object as row | object_id FROM query_result | 0:k | approve atomically + create db_review_action |
| 驳回 | admin_review_reject | same object as row | object_id FROM query_result | 0:k | reject atomically + create db_review_action |
| 审核附件预览 | admin_review_attachment_preview | db_file | file_id FROM query_result | 0:a | signed preview URL |

dynamic_rule (动态规则) = 行标题、教师、班级、变更内容、反馈、附件、日期、状态和 ID 全部来自接口；禁止使用 HTML 的 1..9 等示例 ID


[PAGE_OBJECT]

管理端审核中心 (Admin Review Home / db_admin_review_home)

admin_review_home_id (审核中心聚合ID), 1:1, integer, ui=admin_review.page
admin_id (管理员ID), 1:1, integer, ui=context.hidden
school_id (园所ID), 1:1, integer, ui=context.hidden
admin_school_id (管理员园所授权ID), 1:1, integer, ui=context.hidden
upload_id (上传流程ID), 0:k, integer, ui=admin_review.resource_case.rows
resource_id (资源ID), 0:k, integer, ui=admin_review.resource.rows
case_id (案例ID), 0:k, integer, ui=admin_review.case.rows
teacher_profile_change_id (教师档案修改ID), 0:k, integer, ui=admin_review.profile.rows
teacher_profile_id (教师专业档案ID), 0:k, integer, ui=admin_review.profile.current
credential_id (教师证书/奖项ID), 0:k, integer, ui=admin_review.profile.credential
feedback_id (研修反馈ID), 0:k, integer, ui=admin_review.feedback.rows
child_profile_correction_id (幼儿资料更正ID), 0:k, integer, ui=admin_review.child_profile.rows
review_action_id (审核动作ID), 0:k, integer, ui=admin_review.action.hidden
name_query (名称/教师搜索文字), 0:1, max_len=50, ui=admin_review.name_query
child_scope_filter (幼儿资料待审/已处理), 0:1, pending|processed, ui=admin_review.child_profile.scope_filter
child_class_filter (幼儿资料班级筛选), 0:1, all|class_id, ui=admin_review.child_profile.class_filter
child_result_filter (幼儿资料结果筛选), 0:1, all|approved|rejected, ui=admin_review.child_profile.result_filter

filter_binding (筛选控件绑定) = 以上五栏是查询参数不是列，因此落在本 persist=0 聚合上而不是各目标表：一个搜索框要同时搜五种 target_type 的名称与教师，写在任一目标表上都只对一种成立。班级筛选的 class_id 在管理端属 free（scope-rules.json），但服务端仍须把 derived school_id 内联进同一条 predicate。三个幼儿资料筛选只在该 tab 显示，其余 tab 隐藏而非删除

rel_count (关系数量) = 12
rel_db (关联表) = db_admin, db_school, db_admin_school, db_upload, db_resource, db_case, db_teacher_profile, db_teacher_credential, db_teacher_profile_change, db_training_feedback, db_child_profile_correction, db_review_action
rel_map (关系字段) = db_admin_review_home{admin_id}<->db_admin{admin_id}; db_admin_review_home{school_id}<->db_school{school_id}; db_admin_review_home{admin_school_id}<->db_admin_school{admin_school_id}; db_admin_review_home{upload_id}<->db_upload{upload_id}; db_admin_review_home{resource_id}<->db_resource{resource_id}; db_admin_review_home{case_id}<->db_case{case_id}; db_admin_review_home{teacher_profile_id}<->db_teacher_profile{teacher_profile_id}; db_admin_review_home{credential_id}<->db_teacher_credential{credential_id}; db_admin_review_home{teacher_profile_change_id}<->db_teacher_profile_change{teacher_profile_change_id}; db_admin_review_home{feedback_id}<->db_training_feedback{feedback_id}; db_admin_review_home{child_profile_correction_id}<->db_child_profile_correction{child_profile_correction_id}; db_admin_review_home{review_action_id}<->db_review_action{review_action_id}
persist (是否持久化) = 0
object_type (对象类型) = aggregate


[CONTENT_OBJECTS]

教师专业档案 (Teacher Professional Profile / db_teacher_profile)

teacher_profile_id (教师专业档案ID), 1:1, integer, ui=teacher_profile.hidden
teacher_id (教师ID), 1:1, integer, ui=context.hidden
professional_title (职称), 0:1, max_len=50, ui=teacher_profile.professional_title
education_level (最高学历), 0:1, e1=secondary|e2=associate|e3=bachelor|e4=master|e5=doctor|e6=other, ui=teacher_profile.education
job_role (园内职务), 0:1, j1=lead_teacher|j2=assistant_teacher|j3=caregiver|j4=research_lead|j5=other, ui=teacher_profile.job_role
department_code (部门代码), 0:1, configured_key, ui=teacher_profile.department
career_summary (履历摘要), 0:1, max_len=1000, ui=teacher_profile.career_summary
updated_at (更新时间), 0:1, datetime, ui=teacher_profile.updated_at

rel_count (关系数量) = 1
rel_db (关联表) = db_teacher
rel_map (关系字段) = db_teacher_profile{teacher_id}<->db_teacher{teacher_id}
unique = teacher_id


教师证书与奖项 (Teacher Credential / db_teacher_credential)

credential_id (证书/奖项ID), 1:1, integer, ui=teacher_credential.hidden
teacher_id (教师ID), 1:1, integer, ui=context.hidden
credential_type (材料类型), 1:1, c1=education_certificate|c2=capability_certificate|c3=professional_award, ui=teacher_credential.type
credential_name (材料名称), 1:1, max_len=150, ui=teacher_credential.name
credential_level (级别), 0:1, l1=school|l2=district|l3=city|l4=province|l5=national|l6=other, ui=teacher_credential.level
issuer (颁发机构), 0:1, max_len=150, ui=teacher_credential.issuer
issued_date (颁发日期), 0:1, date, ui=teacher_credential.issued_at
file_id (证明文件ID), 1:1, integer, ui=teacher_credential.file
is_active (是否有效), 1:1, boolean, ui=context.hidden

rel_count (关系数量) = 2
rel_db (关联表) = db_teacher, db_file
rel_map (关系字段) = db_teacher_credential{teacher_id}<->db_teacher{teacher_id}; db_teacher_credential{file_id}<->db_file{file_id}


教师档案修改申请 (Teacher Profile Change / db_teacher_profile_change)

teacher_profile_change_id (档案修改ID), 1:1, integer, ui=admin_review.profile.hidden
school_id (园所ID), 1:1, integer, ui=context.hidden
teacher_id (教师ID), 1:1, integer, ui=admin_review.profile.teacher
teacher_profile_id (当前专业档案ID), 0:1, integer, ui=context.hidden
credential_id (被修改的既有证书/奖项ID), 0:k, integer, ui=context.hidden
change_payload (变更字段和值), 1:1, json, ui=admin_review.profile.change
file_id (证明附件ID), 0:k, integer, ui=admin_review.profile.attachment
change_status (修改状态), 1:1, s1=draft(草稿)|s2=pending(待审核)|s3=approved(已通过)|s4=rejected(已驳回)|s5=cancelled(已取消), ui=admin_review.profile.status
submitted_at (提交时间), 0:1, datetime, ui=admin_review.profile.submitted_at
applied_at (写入档案时间), 0:1, datetime, ui=context.hidden

rel_count (关系数量) = 5
rel_db (关联表) = db_school, db_teacher, db_teacher_profile, db_teacher_credential, db_file
rel_map (关系字段) = db_teacher_profile_change{school_id}<->db_school{school_id}; db_teacher_profile_change{teacher_id}<->db_teacher{teacher_id}; db_teacher_profile_change{teacher_profile_id}<->db_teacher_profile{teacher_profile_id}; db_teacher_profile_change{credential_id}<->db_teacher_credential{credential_id}; db_teacher_profile_change{file_id}<->db_file{file_id}


研修反馈 (Training Feedback / db_training_feedback)

feedback_id (研修反馈ID), 1:1, integer, ui=admin_review.feedback.hidden
school_id (园所ID), 1:1, integer, ui=context.hidden
training_id (研修活动ID), 1:1, integer, ui=admin_review.feedback.training
teacher_id (反馈教师ID), 1:1, integer, ui=admin_review.feedback.teacher
feedback_text (反馈内容), 1:1, max_len=1000, ui=admin_review.feedback.text|training_detail.feedback_input
feedback_status (反馈状态), 1:1, s2=pending(待审核)|s3=published(已公开)|s4=rejected(已驳回)|s5=withdrawn(已撤回), ui=admin_review.feedback.status
submitted_at (提交时间), 0:1, datetime, ui=admin_review.feedback.submitted_at
published_at (公开时间), 0:1, datetime, ui=context.hidden

rel_count (关系数量) = 3
rel_db (关联表) = db_school, db_training, db_teacher
rel_map (关系字段) = db_training_feedback{school_id}<->db_school{school_id}; db_training_feedback{training_id}<->db_training{training_id}; db_training_feedback{teacher_id}<->db_teacher{teacher_id}
visibility_rule = s2 pending|s4 rejected|s5 withdrawn only author and authorized review/admin flow may read; only s3 published is visible to all active formal teachers
publication_rule = teacher submission never appears immediately in the public feedback list; approval decision + feedback_status=s3 + db_review_action insert are one transaction
unique = training_id + teacher_id
participation_rule = only matching db_training_participation.status=s3 completed and NOW>training.effective_end_at may submit; one optional feedback per teacher/training; feedback never affects completion
content_rule = feedback_text required max 1000; attachments FORBIDDEN
identity_rule = published list reads current db_teacher.teacher_name through teacher_id; no duplicated name snapshot
draft_rule = server draft NONE；submit directly INSERTS s2 pending；unsubmitted text is page-local and discarded when leaving
ui_surface_rule = feedback_text 有两个界面：教师在 Teacher/screens/training-detail.html 写（training_detail.feedback_input），admin 在审核中心读（admin_review.feedback.text）。两个路径并列在同一行 ui= 上，因为对象 canonical 只此一份、Teacher/04 training-center-spec.md 只 REUSE 不重新宣告字段；把标注写到教师端 spec 会让同一列抽出两份互不知情的绑定
submission_lock = first successful submission permanently freezes feedback_text；s2|s3|s4|s5 cannot edit or submit again；UNIQUE(training_id,teacher_id) forbids a replacement row
rejection_rule = target_type=training_feedback AND decision=d2 rejected REQUIRES decision_reason max 500；reason is returned to the author；other target types retain their own optionality
terminal_rule = admin rejection s4 and author withdrawal s5 are terminal；row and immutable review actions remain；physical delete and resubmission are FORBIDDEN
withdrawal_rule = author may change own s2 pending or s3 published feedback to s5；s2 exits review queue, s3 disappears publicly immediately；s4|s5 show no withdrawal action
public_stream_rule = feedback is the only public comment stream；count only feedback_status=s3 while db_training.training_status=s1；ORDER BY published_at DESC, feedback_id DESC with stable cursor and load more
training_withdrawn_rule = when training_status=s5, public count=0 and public stream=[] even though feedback rows and review history remain
comment_entity = NONE；独立“评论”标签、计数、对象或回复审核流均禁止


[CHILD_PROFILE_CORRECTION_REUSE]

object = db_child_profile_correction，canonical 定义来自 backend DECISIONS.md F13 与 Parent child-profile-spec.md
list_tabs = pending 只查 r1，submitted_at ASC + id ASC；processed 查 r2|r3，reviewed_at DESC + id DESC；两页都服务端游标分页
filters = 两页可选 class_id 与 child_name；processed 另有 all|approved|rejected。服务端先套 derived school_id 与全部筛选，再分页
detail = r1 并排 original_* 与 proposed_*；若 live db_child 已不同于 original，再显示 current canonical 与提示，但不阻止批准
approval = 一次确认后同交易把 proposed 三栏完整写 db_child、r1→r2、写 reviewed_at 与 t6/d1 review_action；不发送家长通知
rejection = 一次确认内要求 trim 后 1—500 字理由，同交易 r1→r3、写时间与 t6/d2 action，并通知决定当下全部 caretakers
concurrency = 任一有效同园管理端账号可处理；第一笔终局交易成功，后到请求 409，不得再写 action／通知


审核动作 (Review Action / db_review_action)

review_action_id (审核动作ID), 1:1, integer, ui=admin_review.action.hidden
school_id (园所ID), 1:1, integer, ui=context.hidden
admin_id (审核管理员ID), 1:1, integer, ui=context.hidden
target_type (审核目标类型), 1:1, t1=resource|t2=case|t3=teacher_profile_change|t4=training_feedback|t6=child_profile_correction, ui=admin_review.action.hidden
resource_id (资源ID), 0:1, integer, ui=context.hidden
case_id (案例ID), 0:1, integer, ui=context.hidden
teacher_profile_change_id (档案修改ID), 0:1, integer, ui=context.hidden
feedback_id (研修反馈ID), 0:1, integer, ui=context.hidden
child_profile_correction_id (幼儿资料更正ID), 0:1, integer, ui=context.hidden
decision (审核决定), 1:1, d1=approved(通过)|d2=rejected(驳回), ui=admin_review.action
decision_reason (审核意见), 0:1, max_len=500, ui=admin_review.action.reason
created_at (审核时间), 1:1, datetime, ui=admin_review.action.created_at

rel_count (关系数量) = 7
rel_db (关联表) = db_school, db_admin, db_resource, db_case, db_teacher_profile_change, db_training_feedback, db_child_profile_correction
rel_map (关系字段) = db_review_action{school_id}<->db_school{school_id}; db_review_action{admin_id}<->db_admin{admin_id}; IF target_type=t1, db_review_action{resource_id}<->db_resource{resource_id}; IF target_type=t2, db_review_action{case_id}<->db_case{case_id}; IF target_type=t3, db_review_action{teacher_profile_change_id}<->db_teacher_profile_change{teacher_profile_change_id}; IF target_type=t4, db_review_action{feedback_id}<->db_training_feedback{feedback_id}; IF target_type=t6, db_review_action{child_profile_correction_id}<->db_child_profile_correction{child_profile_correction_id}
check (校验) = target_type 对应的一个目标ID必须非空，其余目标ID必须为 NULL


[REUSED_OBJECT_USAGE]

db_upload|db_resource|db_case = REUSE home-spec.md; resource/case approval updates both upload_status and target status in one transaction
db_training|db_teacher|db_file = REUSE existing canonical objects; no admin-prefixed copies
profile_approval = validate change_payload allowlist -> update db_teacher_profile and/or create/update db_teacher_credential -> mark change approved
approval_flow = pending -> approved/published; rejection_flow = pending -> rejected; db_review_action is immutable audit history


[EMPTY_STATE]

IF selected tab has no pending rows, return [] AND count=0
empty_title = 该分类没有待审核的内容
IF search has no match, return [] AND empty_title=没有匹配的待审核内容
attachment_without_data = []


[NAV_OBJECTS]

shared_navigation_source (共享管理端导航) = dashboard-spec.md [NAV_OBJECTS]
required_namespace (命名空间) = nav_admin_*


[JUMP_VALIDATION]

IF target_type NOT_IN(resource,case,teacher_profile_change,training_feedback,child_profile_correction), return 400
IF object_id not returned by current authorized query, return 404
IF object.school_id != current_school_id, return 403
IF admin lacks target-specific review permission, return 403
IF current status != pending, return 409
IF target_type=training_feedback AND decision=rejected AND decision_reason IS EMPTY, return 422
IF target_type=child_profile_correction AND decision=rejected AND trim(decision_reason) length NOT_IN 1..500, return 422
IF target_type=child_profile_correction, any valid admin login for current_school_id is authorized; do not apply target-specific permission
IF decision=rejected AND product requires a reason, REQUIRE decision_reason ELSE return 422
approval/rejection/status update/db_review_action creation MUST be one transaction
raw admin_id|school_id|teacher_id from client MUST be ignored and re-derived
