ADMIN_HOME_SCHOOL_DATA_BACKEND_OBJECT_SPEC

scope (范围) = index.html#home-school
source_page (参考页面) = index.html
static_node_count (固定操作节点数) = 10
dynamic_class_row_count (动态班级行数) = 0:k
dynamic_detail_action_count (动态明细节点数) = 0:k
runtime_clickable_node_count (运行时可点击节点数) = 10:(10+k)
field_format (字段格式) = field_key (中文字段名), cardinality, type|enum, ui
id_rule (ID规则) = integer, database_auto_generated
null_rule (空值规则) = 0:1
list_rule (列表规则) = 0:k | 1:k


[SHARED_OBJECT_RULE]

canonical_registry = docs/spec-handoff.md
shared_object_source = dashboard-spec.md; Teacher App home-spec.md|home-school-spec.md
shared_objects = db_admin, db_school, db_class, db_teacher, db_teacher_class, db_child, db_home_school_progress, db_moment, db_moment_upload, db_parent_task, db_parent_task_submission, db_growth_record, db_growth_book
new_business_object = NONE
admin_page_aggregate = db_admin_home_school_home
shared_progress_alias = FORBIDDEN


[CONTEXT_RULE]

admin_id = auth_session.admin_id
allowed_school_id = db_admin.school_id WHERE admin_id=current_admin_id AND admin_status=s1
current_school_id MUST IN allowed_school_id
permission = db_admin.role|permission_scope
required_permission = home_school_data.read
class_id for detail MUST belong to current_school_id
raw identity IDs ui = context.hidden
client_editable = 0
backend_authorization_validation = REQUIRED


[DATA_INITIALIZATION_RULE]

prototype_content = HTML 中六个班级/教师/幼儿数、四项 KPI、全部百分比、进度条和明细提示均为 Mock
static_ui_content = 四项指标名称、说明、表头、按钮名和空状态文案
business_seed = NONE
production_initial_business_objects = db_moment|db_moment_upload|db_parent_task|db_parent_task_submission|db_growth_record|db_growth_book EMPTY
dynamic_list_without_data = []
dynamic_count_without_data = 0
unassigned_or_unstarted_status = not_started
roster_without_business = real class rows may display with all rates=0 and statuses=not_started
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
| 10 | 导出数据 | Export Data | btn_admin_export | db_admin_home_school_home | current period | file download |


[DYNAMIC_CONTENT_NODE]

| node_name_cn | node_key | object | input | cardinality | jump/action |
|---|---|---|---|---|---|
| 全园 KPI | admin_home_school_kpi | db_home_school_progress | current_school_id, period | 4 | NONE |
| 班级进度行 | admin_home_school_class_row | db_class + db_home_school_progress | class_id FROM query_result | 0:k | NONE |
| 班级明细 | admin_home_school_detail | db_home_school_progress | class_id FROM query_result, period | 0:k | detail modal/data; no separate spec requested |

dynamic_rule = 班级、教师、幼儿数、百分比、状态和 class_id 必须来自 API；不得使用 HTML 的班名或百分比作为业务参数


[PAGE_OBJECT]

管理端家园共育数据首页 (Admin Home-School Data Home / db_admin_home_school_home)

admin_home_school_home_id (页面聚合ID), 1:1, integer, ui=admin_home_school.page
admin_id (管理员ID), 1:1, integer, ui=context.hidden
school_id (园所ID), 1:1, integer, ui=context.hidden
admin_school_id (授权ID), 1:1, integer, ui=context.hidden
class_id (班级ID), 0:k, integer, ui=admin_home_school.rows
teacher_id (带班教师ID), 0:k, integer, ui=admin_home_school.teacher
teacher_class_id (教师班级关系ID), 0:k, integer, ui=context.hidden
child_id (幼儿ID), 0:k, integer, ui=admin_home_school.child_count
home_school_progress_id (家园进度ID), 0:k, integer, ui=admin_home_school.metrics
moment_id (在园时光ID), 0:k, integer, ui=admin_home_school.moment
moment_upload_id (在园时光上传ID), 0:k, integer, ui=admin_home_school.moment
parent_task_id (亲子任务ID), 0:k, integer, ui=admin_home_school.parent_task
parent_task_submission_id (亲子任务提交ID), 0:k, integer, ui=admin_home_school.parent_task
growth_record_id (成长档案ID), 0:k, integer, ui=admin_home_school.growth_record
growth_book_id (成长册ID), 0:k, integer, ui=admin_home_school.growth_book

rel_count (关系数量) = 13
rel_db (关联表) = db_admin, db_school, db_class, db_teacher, db_teacher_class, db_child, db_home_school_progress, db_moment, db_moment_upload, db_parent_task, db_parent_task_submission, db_growth_record, db_growth_book
rel_map (关系字段) = db_admin_home_school_home{admin_id}<->db_admin{admin_id}; db_admin_home_school_home{school_id}<->db_school{school_id}; db_admin_home_school_home{class_id}<->db_class{class_id}; db_admin_home_school_home{teacher_id}<->db_teacher{teacher_id}; db_admin_home_school_home{teacher_class_id}<->db_teacher_class{teacher_class_id}; db_admin_home_school_home{child_id}<->db_child{child_id}; db_admin_home_school_home{home_school_progress_id}<->db_home_school_progress{home_school_progress_id}; db_admin_home_school_home{moment_id}<->db_moment{moment_id}; db_admin_home_school_home{moment_upload_id}<->db_moment_upload{moment_upload_id}; db_admin_home_school_home{parent_task_id}<->db_parent_task{parent_task_id}; db_admin_home_school_home{parent_task_submission_id}<->db_parent_task_submission{parent_task_submission_id}; db_admin_home_school_home{growth_record_id}<->db_growth_record{growth_record_id}; db_admin_home_school_home{growth_book_id}<->db_growth_book{growth_book_id}
persist = 0
object_type = aggregate


[REUSED_OBJECT_USAGE]

db_home_school_progress = REUSE non-persistent per-child aggregate view from home-school-spec.md
db_moment|db_moment_upload|db_parent_task|db_parent_task_submission|db_growth_record|db_growth_book = REUSE canonical business records
class_moment_rate = completed required moment coverage / assigned required moment coverage; denominator=0 -> 0
class_parent_task_rate = complete submissions / required submissions; denominator=0 -> 0
class_growth_record_rate = complete growth records / active roster count; roster count=0 -> 0
class_growth_book_rate = generated growth books / active roster count; roster count=0 -> 0
school_kpi = aggregate numerators and denominators across authorized classes; do not average class percentages


[EMPTY_STATE]

IF no real classes, return [] AND empty_title=暂无班级数据 AND all KPI=0
IF class exists but no roster, return class row with child_count=0, all rates=0, status=not_started
IF roster exists but no business records, return real class row with all rates=0 and all statuses=not_started
IF detail has no assigned content, return [] AND empty_title=暂无家园共育记录


[NAV_OBJECTS]

shared_navigation_source = dashboard-spec.md [NAV_OBJECTS]
required_namespace = nav_admin_*


[JUMP_VALIDATION]

IF class_id missing from dynamic detail request, return 400
IF class_id NOT_FROM authorized current_school query, return 403
IF class_id NOT_FOUND, return 404
IF child_id appears in detail and child.class_id != requested class_id, return 403
IF admin lacks home_school_data.read, return 403
current_school_id MUST be backend-derived; client school_id is ignored
