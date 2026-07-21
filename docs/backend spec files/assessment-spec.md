ADMIN_ASSESSMENT_DATA_BACKEND_OBJECT_SPEC

scope (范围) = index.html#assessment
source_page (参考页面) = index.html
static_node_count (固定操作节点数) = 12
dynamic_class_row_count (动态班级行数) = 0:k
dynamic_row_action_count (动态行操作数) = 0:2k
runtime_clickable_node_count (运行时可点击节点数) = 12:(12+2k)
field_format (字段格式) = field_key (中文字段名), cardinality, type|enum, ui
id_rule (ID规则) = integer, database_auto_generated
null_rule (空值规则) = 0:1
list_rule (列表规则) = 0:k | 1:k


[SHARED_OBJECT_RULE]

canonical_registry = docs/spec-handoff.md
shared_object_source = dashboard-spec.md; Teacher App home-spec.md|home-school-spec.md
shared_objects = db_admin, db_admin_school, db_school, db_class, db_teacher, db_teacher_class, db_child, db_assessment, db_assessment_item
new_business_object = NONE
admin_page_aggregate = db_admin_assessment_home
assessment_alias = FORBIDDEN


[CONTEXT_RULE]

admin_id = auth_session.admin_id
allowed_school_id = db_admin_school.school_id WHERE admin_id=current_admin_id AND is_active=1
current_school_id MUST IN allowed_school_id
permission = db_admin_school.role|permission_scope
required_permission = assessment.read; assessment.export for export actions
class_id selector MUST reference db_class.school_id=current_school_id
raw identity IDs ui = context.hidden
client_editable = 0
backend_authorization_validation = REQUIRED


[DATA_INITIALIZATION_RULE]

prototype_content = HTML 中六个班级/教师、幼儿数、已测数、百分比、五域分数、雷达图、学期和报告均为 Mock
static_ui_content = 五大领域名称、5 分制说明、班级/学期筛选标签、表头、图例和空状态文案
business_seed = NONE
production_initial_db_assessment = EMPTY
production_initial_db_assessment_item = EMPTY
dynamic_list_without_data = []
dynamic_count_without_data = 0
unassigned_or_unstarted_status = not_started
roster_without_assessment = real class/child rows may appear with completed_count=0, completion_rate=0, domain_average=NULL, status=not_started
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
| 9 | 导出数据 | Export Data | btn_admin_export | db_admin_assessment_home | current filters | file download |
| 10 | 班级选择 | Class Selector | btn_admin_assessment_class | db_class | class_id FROM authorized query | refresh table/radar |
| 11 | 学期选择 | Term Selector | btn_admin_assessment_term | db_assessment | assessment_period | refresh table/radar |
| 12 | 导出班级报告 | Export Class Report | btn_admin_assessment_export | db_assessment + db_assessment_item | class_id, period | generated file download |


[DYNAMIC_CONTENT_NODE]

| node_name_cn | node_key | object | input | cardinality | jump/action |
|---|---|---|---|---|---|
| 班级测评进度行 | admin_assessment_class_row | db_class + db_teacher_class + db_child + db_assessment | class_id FROM query_result | 0:k | NONE |
| 雷达图 | admin_assessment_radar | db_assessment_item | class_id FROM query_result, period | 0:k | select row and redraw |
| 行内导出 | admin_assessment_row_export | db_assessment + db_assessment_item | class_id FROM query_result, period | 0:k | generated file download |

dynamic_rule = 班级、教师、人数、完成率、五域分数、状态和 class_id 来自 API；班级名称不得作为查询或导出授权键


[PAGE_OBJECT]

管理端测评数据首页 (Admin Assessment Home / db_admin_assessment_home)

admin_assessment_home_id (页面聚合ID), 1:1, integer, ui=admin_assessment.page
admin_id (管理员ID), 1:1, integer, ui=context.hidden
school_id (园所ID), 1:1, integer, ui=context.hidden
admin_school_id (授权ID), 1:1, integer, ui=context.hidden
class_id (班级ID), 0:k, integer, ui=admin_assessment.rows|selector
teacher_id (带班教师ID), 0:k, integer, ui=admin_assessment.teacher
teacher_class_id (教师班级关系ID), 0:k, integer, ui=context.hidden
child_id (在园幼儿ID), 0:k, integer, ui=admin_assessment.total
assessment_id (测评ID), 0:k, integer, ui=admin_assessment.done|status
item_id (测评项ID), 0:k, integer, ui=admin_assessment.domain_average|radar

rel_count (关系数量) = 9
rel_db (关联表) = db_admin, db_school, db_admin_school, db_class, db_teacher, db_teacher_class, db_child, db_assessment, db_assessment_item
rel_map (关系字段) = db_admin_assessment_home{admin_id}<->db_admin{admin_id}; db_admin_assessment_home{school_id}<->db_school{school_id}; db_admin_assessment_home{admin_school_id}<->db_admin_school{admin_school_id}; db_admin_assessment_home{class_id}<->db_class{class_id}; db_admin_assessment_home{teacher_id}<->db_teacher{teacher_id}; db_admin_assessment_home{teacher_class_id}<->db_teacher_class{teacher_class_id}; db_admin_assessment_home{child_id}<->db_child{child_id}; db_admin_assessment_home{assessment_id}<->db_assessment{assessment_id}; db_admin_assessment_home{item_id}<->db_assessment_item{item_id}
persist = 0
object_type = aggregate


[REUSED_OBJECT_USAGE]

db_class|db_teacher|db_teacher_class|db_child = REUSE canonical roster and assignment objects
db_assessment|db_assessment_item = REUSE Teacher App assessment records; no admin assessment table
total_count = COUNT(db_child WHERE class_id=row.class_id AND enrollment_status=active)
completed_count = COUNT(DISTINCT db_assessment.child_id WHERE class_id=row.class_id AND assessment_period=selected_period AND assessment_status=complete)
completion_rate = IF total_count=0 THEN 0 ELSE completed_count/total_count*100
domain_average = AVG(db_assessment_item.score GROUP BY assessment_domain) only for submitted/complete assessments; no scores -> NULL
school_average = weighted aggregation of real child assessment items, not average of Mock class percentages


[CANONICAL_FIELD_EXTENSION]

reason = current Admin UI requires per-child five-domain completion while canonical db_assessment currently lacks child and domain dimensions
db_assessment ADD school_id(1:1), child_id(0:1)
db_assessment.assessment_scope ADD a4=child(幼儿)
db_assessment.assessment_period ACCEPT YYYY-MM|school_term
db_assessment canonical rel_count = 5
db_assessment canonical rel_db = db_teacher, db_class, db_child, db_school, db_assessment_item
db_assessment canonical rel_map = db_assessment{teacher_id}<->db_teacher{teacher_id}; db_assessment{class_id}<->db_class{class_id}; db_assessment{child_id}<->db_child{child_id}; db_assessment{school_id}<->db_school{school_id}; db_assessment{assessment_id}<->db_assessment_item{assessment_id}
db_assessment_item ADD assessment_domain, 1:1, f1=health|f2=language|f3=social|f4=science|f5=art
extension_rule = extend the same canonical objects; app-prefixed assessment-record copies are FORBIDDEN


[EMPTY_STATE]

IF no real classes, return [] AND empty_title=暂无班级数据
IF class exists but no active child, total_count=0, completed_count=0, completion_rate=0, status=not_started
IF active roster exists but no assessment, keep real class row with completed_count=0, completion_rate=0, domain_average=NULL, status=not_started
IF selected class has no scores, radar_series=[] AND show radar_empty_title=暂无测评分数


[NAV_OBJECTS]

shared_navigation_source = dashboard-spec.md [NAV_OBJECTS]
required_namespace = nav_admin_*


[JUMP_VALIDATION]

IF period format invalid, return 400
IF class_id NOT_FROM authorized current_school class query, return 403
IF dynamic class_id missing or NOT_FOUND, return 404
IF assessment belongs to another school/class/period, return 403
IF export lacks assessment.export, return 403
report generation MUST use server-side current_school_id and class_id from authorized query result
