ADMIN_DASHBOARD_BACKEND_OBJECT_SPEC

scope (范围) = index.html#dashboard
source_page (参考页面) = index.html
static_node_count (固定操作节点数) = 14
dynamic_review_action_count (动态快捷审核节点数) = 0:6
runtime_clickable_node_count (运行时可点击节点数) = 14:20
field_format (字段格式) = field_key (中文字段名), cardinality, type|enum, ui
id_rule (ID规则) = integer, database_auto_generated
null_rule (空值规则) = 0:1
list_rule (列表规则) = 0:k | 1:k


[SHARED_OBJECT_RULE]

canonical_registry (canonical 注册表) = docs/spec-handoff.md
shared_object_source (共享对象来源) = Teacher App home-spec.md|home-school-spec.md; review-spec.md
shared_objects (共享对象) = db_school, db_teacher, db_class, db_child, db_resource, db_case, db_assessment, db_teacher_profile_change, db_training_feedback
new_identity_objects_defined_here (本页首次定义的身份对象) = db_admin
admin_page_aggregate (管理端页面聚合) = db_admin_home
rename_or_duplicate_shared_object (重命名或复制共享对象) = FORBIDDEN


[CONTEXT_RULE]

admin_id_source (管理员ID来源) = auth_session.admin_id
allowed_school_id_source (授权园所来源) = db_admin.school_id WHERE admin_id=current_admin_id AND admin_status=s1
current_school_rule (当前园所规则) = current_school_id MUST IN allowed_school_id
permission_source (权限来源) = db_admin.role|permission_scope
required_permission (所需权限) = dashboard.read; dashboard.export for btn_admin_export
raw_identity_ui (原始身份ID界面) = context.hidden
client_editable (前端可编辑) = 0
backend_authorization_validation (后台授权校验) = REQUIRED


[DATA_INITIALIZATION_RULE]

prototype_content (原型内容) = HTML 中园所名称、管理员姓名、班级/教师/幼儿、KPI、趋势、审核队列、案例领域数量、完成率、日期和状态均为 demo|test Mock
static_ui_content (可保留静态内容) = 八项导航、页面/栏目名称、五大领域标签、图例、操作说明和空状态文案
business_seed (生产业务种子) = NONE
dynamic_list_without_data (无业务数据动态列表) = []
dynamic_count_without_data (无业务数据动态数量) = 0
unassigned_or_unstarted_status (未分配或未开始状态) = not_started
base_identity_data (基础数据) = db_school|db_admin|db_teacher|db_class|db_child 可由部署或有权管理员初始化
hardcoded_business_id (固定业务ID) = FORBIDDEN
environment_isolation (环境隔离) = demo|test 数据不得复制到 production


[STATIC_BUTTON_NODE_INDEX]

| n | button_name_cn | button_name_en | node_key | object | input | jump |
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
| 10 | 导出数据 | Export Data | btn_admin_export | db_admin_home | current_school_id, filter_context | file download |
| 11 | 查看/收起数据表 | Toggle Trend Table | btn_admin_dashboard_trend_table | db_admin_home | NULL | local expand/collapse |
| 12 | 全部审核 | Review All | btn_admin_dashboard_review_all | db_admin_review_home | current_school_id | index.html#review |
| 13 | 去审核 | Go to Review | btn_admin_dashboard_review | db_admin_review_home | current_school_id | index.html#review |
| 14 | 测评数据 | Assessment Data | btn_admin_dashboard_assessment | db_admin_assessment_home | current_school_id | index.html#assessment |


[DYNAMIC_CONTENT_NODE]

| node_name_cn | node_key | object | source/filter | cardinality | jump/action |
|---|---|---|---|---|---|
| KPI 指标 | admin_dashboard_kpi | db_admin_home | current_school_id | 4 | NONE |
| 资源/案例周趋势 | admin_dashboard_trend_point | db_resource + db_case | created_at within selected 12-week window | 0:k | NONE |
| 待审核队列行 | admin_dashboard_review_row | db_resource|db_case|db_teacher_profile_change|db_training_feedback | pending, ORDER BY submitted_at DESC LIMIT 6 | 0:6 | NONE |
| 快捷通过 | admin_dashboard_review_approve | same object as row | object_id FROM query_result | 0:6 | approve with permission and write db_review_action |
| 案例领域分布行 | admin_dashboard_case_domain_row | db_case | case_status=s3, GROUP BY case_field | 5 | NONE |
| 班级测评进度行 | admin_dashboard_assessment_row | db_assessment + db_class | current_school_id, selected term | 0:k | NONE |

dynamic_rule (动态规则) = 标题、名称、数量、日期、百分比、状态和所有 object_id 必须来自授权后的接口结果；不得使用 HTML 数组或名称作为业务键


[PAGE_OBJECT]

管理端数据看板 (Admin Dashboard / db_admin_home)

admin_home_id (管理端首页ID), 1:1, integer, ui=admin_dashboard.page
admin_id (当前管理员ID), 1:1, integer, ui=context.hidden
school_id (当前园所ID), 1:1, integer, ui=context.hidden
child_id (在园幼儿ID), 0:k, integer, ui=admin_dashboard.kpi.child_count
teacher_id (教职工ID), 0:k, integer, ui=admin_dashboard.kpi.teacher_count
resource_id (资源ID), 0:k, integer, ui=admin_dashboard.resource.trend|admin_dashboard.kpi.resource_count
case_id (案例ID), 0:k, integer, ui=admin_dashboard.case.trend|admin_dashboard.case.domain
assessment_id (测评ID), 0:k, integer, ui=admin_dashboard.assessment.rows
class_id (班级ID), 0:k, integer, ui=admin_dashboard.assessment.rows
teacher_profile_change_id (教师资料变更ID), 0:k, integer, ui=admin_dashboard.review.queue
feedback_id (研修反馈ID), 0:k, integer, ui=admin_dashboard.review.queue

rel_count (关系数量) = 10
rel_db (关联表) = db_admin, db_school, db_child, db_teacher, db_resource, db_case, db_assessment, db_class, db_teacher_profile_change, db_training_feedback
rel_map (关系字段) = db_admin_home{admin_id}<->db_admin{admin_id}; db_admin_home{school_id}<->db_school{school_id}; db_admin_home{child_id}<->db_child{child_id}; db_admin_home{teacher_id}<->db_teacher{teacher_id}; db_admin_home{resource_id}<->db_resource{resource_id}; db_admin_home{case_id}<->db_case{case_id}; db_admin_home{assessment_id}<->db_assessment{assessment_id}; db_admin_home{class_id}<->db_class{class_id}; db_admin_home{teacher_profile_change_id}<->db_teacher_profile_change{teacher_profile_change_id}; db_admin_home{feedback_id}<->db_training_feedback{feedback_id}
persist (是否持久化) = 0
object_type (对象类型) = aggregate


[CONTENT_OBJECTS]

管理员 (Admin / db_admin)

admin_id (管理员ID), 1:1, integer, ui=context.hidden
admin_name (管理员姓名), 1:1, max_len=50, ui=topbar.admin_name
school_id (园所ID), 1:1, integer, ui=context.hidden
role (管理员角色), 1:1, r1=super_admin(超级管理员)|r2=school_admin(园所管理员)|r3=department_admin(部门管理员)|r4=auditor(审核员)|r5=analyst(数据查看员), ui=topbar.admin_role
permission_scope (权限范围), 1:k, permission_key, ui=context.hidden
admin_status (管理员状态), 1:1, s1=active(启用)|s2=inactive(停用)|s3=suspended(暂停), ui=context.hidden

rel_count (关系数量) = 1
rel_db (关联表) = db_school
rel_map (关系字段) = db_admin{school_id}<->db_school{school_id}


管理员园所授权 (Admin-School Authorization / 无独立表)

[db_admin_school 已删除 —— DECISIONS.md B3：单园系统下它实际是 1:1，
 整张併回 db_admin（school_id + role + permission_scope，见上方）。
 permission_scope 本来就已是 JSON，併表不增加新的存储形态。
 is_active 比照 B1 用「从数组/关系中移除」表达，不留软旗标。
 代价：管理员的授权边界改为读 db_admin 上的 JSON，服务层需涵盖 GAPS.md G22 的三项失去。]


[REUSED_OBJECT_USAGE]

db_school|db_teacher|db_class|db_child = REUSE identity/roster objects; only real provisioned records may contribute to KPI
db_resource|db_case = REUSE Teacher App objects; trend and totals are derived from real created_at/status values
db_assessment = REUSE Teacher App object; class progress is derived, not seeded
db_teacher_profile_change|db_training_feedback = REUSE definitions in review-spec.md after registry approval


[EMPTY_STATE]

IF no roster, child_count=0 AND teacher_count=0
IF no resource/case records, trend_series=[] AND resource_count=0 AND case_domain_rows=[]
IF no pending review records, review_rows=[] AND pending_count=0 AND show review_empty_title=审核队列已清空
IF real classes exist but no assessment records, return class rows with status=not_started, completed_count=0, percentage=0
IF no real classes, assessment_rows=[] AND show assessment_empty_title=暂无班级数据


[NAV_OBJECTS]

| navigation_object | object_ref | route | rel_count |
|---|---|---|---:|
| nav_admin_dashboard | db_admin_home | index.html#dashboard | 0 |
| nav_admin_review | db_admin_review_home | index.html#review | 0 |
| nav_admin_library | db_admin_library_home | index.html#library | 0 |
| nav_admin_tasks | db_admin_task_home | index.html#tasks | 0 |
| nav_admin_assessment | db_admin_assessment_home | index.html#assessment | 0 |
| nav_admin_home_school | db_admin_home_school_home | index.html#home-school | 0 |
| nav_admin_org | db_admin_org_home | index.html#org | 0 |
| nav_admin_content | db_admin_content_home | index.html#content | 0 |
| nav_admin_growth_book | db_admin_growth_book_setting_home | index.html#growthbook | 0 |


[JUMP_VALIDATION]

IF view NOT_IN(dashboard,review,library,tasks,assessment,home-school,org,content,growthbook), return 400
IF admin_id NOT_AUTHORIZED_FOR current_school_id, return 403
IF required permission missing, return 403
IF node_key=admin_dashboard_review_approve, REQUIRE target_type AND object_id FROM query_result; re-read pending status atomically; NOT_FOUND=404; non-pending=409
IF export requested, derive school_id from authorized context and ignore client-supplied raw identity IDs
hardcoded_business_id = FORBIDDEN
