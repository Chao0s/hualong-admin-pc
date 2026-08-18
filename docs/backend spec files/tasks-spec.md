ADMIN_TASK_MANAGEMENT_BACKEND_OBJECT_SPEC

scope (范围) = index.html#tasks
source_page (参考页面) = index.html
static_node_count (固定按钮节点数) = 16
dynamic_task_row_count (动态任务行数) = 0:k
dynamic_row_action_count (动态行操作数) = 0:2k
runtime_clickable_node_count (运行时可点击节点数) = 16:(16+2k)
field_format (字段格式) = field_key (中文字段名), cardinality, type|enum, ui
id_rule (ID规则) = integer, database_auto_generated
null_rule (空值规则) = 0:1
list_rule (列表规则) = 0:k | 1:k


[SHARED_OBJECT_RULE]

canonical_registry = docs/spec-handoff.md
shared_object_source = dashboard-spec.md; review-spec.md; Teacher App home-spec.md
shared_objects = db_admin, db_school, db_teacher, db_teacher_profile, db_task, db_task_assign, db_file
new_business_object = NONE
admin_page_aggregate = db_admin_task_home
task_alias = FORBIDDEN; admin-issued teacher work remains canonical db_task


[CONTEXT_RULE]

admin_id = auth_session.admin_id
allowed_school_id = db_admin.school_id WHERE admin_id=current_admin_id AND admin_status=s1
current_school_id MUST IN allowed_school_id
permission = db_admin.role|permission_scope
required_permission = task.read|task.publish|task.urge according to action
raw admin_id|school_id|teacher_id ui = context.hidden
client_editable = 0
backend_authorization_validation = REQUIRED


[DATA_INITIALIZATION_RULE]

prototype_content = HTML 中五条任务、标题、指派对象、日期、52/12/6 人、完成进度、状态和表单默认日期均为 Mock
static_ui_content = 状态筛选、固定指派类别、字段标签、说明和空状态文案
business_seed = NONE
production_initial_db_task = EMPTY
production_initial_db_task_assign = EMPTY
dynamic_list_without_data = []
dynamic_count_without_data = 0
unassigned_or_unstarted_status = not_started
hardcoded_business_id = FORBIDDEN
assignment_count_rule = 必须由发布时授权范围内的真实教师查询结果计算，不得使用 52|12|6 常量
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
| 10 | 导出数据 | Export Data | btn_admin_export | db_admin_task_home | current filter | file download |
| 11 | 全部 | All Tasks | btn_admin_tasks_all | db_task | all | local/API filter |
| 12 | 进行中 | Active Tasks | btn_admin_tasks_active | db_task | wait_accept|in_progress | local/API filter |
| 13 | 已完成 | Completed Tasks | btn_admin_tasks_complete | db_task | complete | local/API filter |
| 14 | 发布任务 | Publish Task | btn_admin_tasks_open_publish | db_task | NULL | open modal |
| 15 | 取消 | Cancel | btn_admin_tasks_cancel | db_task | NULL | close modal |
| 16 | 发布 | Publish | btn_admin_tasks_publish | db_task + db_task_assign | validated form | create task and assignments |


[FORM_FIELD_INDEX]

| field | object.field | required | source/rule |
|---|---|---:|---|
| 任务标题 | db_task.task_title | 1 | max_len=50 |
| 指派对象 | assignment selector | 1 | all_teachers|grade_group|department; selector is query input, not stored as identity IDs |
| 截止时间 | db_task.due_at | 1 | future datetime |
| 任务说明 | db_task.task_intro|task_division | 1 | split/validate per API contract; max_len follows canonical fields |

form_ui_note (表单控件标注位置) = 上表是散文，`extract-ui-binding.mjs` 读不到，所以 modal-task 长期零绑定。下面 [FORM_WRITE_OBJECTS] 是同一件事的机器可读形式，两者必须一起改


[FORM_WRITE_OBJECTS]

发布教师任务表单 (Publish Task Form / db_task)

task_title (任务标题), 1:1, max_len=50, ui=admin_task.title_input
task_intro (任务说明), 1:1, max_len=300, ui=admin_task.intro_input
due_at (截止时间), 1:1, datetime, ui=admin_task.due_input

canonical_source = REUSE Teacher App home-spec.md db_task；本块只登记 index.html#tasks 的 modal-task 实际写入的列
assignee_binding (指派对象控件) = 弹层的「指派对象」下拉不是任何一列：它的值是 all_teachers|grade_group|department 这类选择器语意，服务端解析成一批真实 teacher_id 后批量建 db_task_assign（见 [REUSED_OBJECT_USAGE] 的 publish_method）。因此登记为本页 persist=0 聚合 db_admin_task_home 上的 assignee_selector，不挂在 db_task_assign.teacher_id 上 —— 那一列存的是解析结果，不是使用者选的东西
task_division_binding (任务分工) = 原型只有一个「任务说明」textarea，写 task_intro；task_division 目前无控件，等 API 契约定下拆分方式后再补标注，不预先造一个假控件
created_by_admin_id (创建管理员) = derived，不出现在 data-ui


[DYNAMIC_CONTENT_NODE]

| node_name_cn | node_key | object | input | cardinality | jump/action |
|---|---|---|---|---|---|
| 任务行 | admin_task_row | db_task + db_task_assign | task_id FROM query_result | 0:k | NONE |
| 明细 | admin_task_detail | db_task + db_task_assign | task_id FROM query_result | 0:k | detail data/modal; no separate spec requested |
| 催办 | admin_task_urge | db_task_assign | task_id FROM query_result | 0:k | notify assignees whose assign_status != complete |

dynamic_rule = 任务、人数、截止日期、进度和状态来自 API；动态跳转及 mutation 使用 task_id，不得使用任务标题


[PAGE_OBJECT]

管理端任务管理首页 (Admin Task Home / db_admin_task_home)

admin_task_home_id (页面聚合ID), 1:1, integer, ui=admin_task.page
admin_id (管理员ID), 1:1, integer, ui=context.hidden
school_id (园所ID), 1:1, integer, ui=context.hidden
admin_school_id (授权ID), 1:1, integer, ui=context.hidden
task_id (任务ID), 0:k, integer, ui=admin_task.rows
assign_id (任务分配ID), 0:k, integer, ui=admin_task.progress|admin_task.detail
teacher_id (执行教师ID), 0:k, integer, ui=admin_task.detail.assignee
teacher_profile_id (教师专业档案ID), 0:k, integer, ui=context.hidden
assignee_selector (指派对象选择), 0:1, all_teachers|grade_group|department, ui=admin_task.assignee_selector

rel_count (关系数量) = 6
rel_db (关联表) = db_admin, db_school, db_task, db_task_assign, db_teacher, db_teacher_profile
rel_map (关系字段) = db_admin_task_home{admin_id}<->db_admin{admin_id}; db_admin_task_home{school_id}<->db_school{school_id}; db_admin_task_home{task_id}<->db_task{task_id}; db_admin_task_home{assign_id}<->db_task_assign{assign_id}; db_admin_task_home{teacher_id}<->db_teacher{teacher_id}; db_admin_task_home{teacher_profile_id}<->db_teacher_profile{teacher_profile_id}
persist = 0
object_type = aggregate


[REUSED_OBJECT_USAGE]

db_task = REUSE home-spec.md; internal teacher work task
db_task_assign = REUSE home-spec.md; one row per resolved real teacher
publish_method = validate selector -> query authorized active teachers in current_school_id -> create db_task -> bulk create db_task_assign with assign_status=wait_accept
department_selector = resolve db_teacher_profile.department_code; grade_group selector resolves db_teacher_class -> db_class.grade
progress_method = accepted_count=COUNT(accepted_at IS NOT NULL); completed_count=COUNT(assign_status=complete); total_count=COUNT(assign_id); IF total_count=0, completion_rate=0
urge_method = recipients=assign rows WHERE assign_status IN(wait_accept,in_progress); recipient_count=0 returns no-op


[CANONICAL_FIELD_EXTENSION]

reason = existing db_task.created_by is teacher_id-only and cannot safely represent auth_session.admin_id
db_task ADD school_id, creator_type(c1=teacher|c2=admin), created_by_admin_id(0:1, integer, ref=db_admin.admin_id)
db_task CHANGE created_by cardinality from 1:1 to 0:1 integer(ref=db_teacher.teacher_id); exactly one of created_by and created_by_admin_id is required according to creator_type
db_task canonical rel_count = 5
db_task canonical rel_db = db_task_assign, db_file, db_school, db_teacher, db_admin
db_task canonical rel_map = db_task{task_id}<->db_task_assign{task_id}; db_task{file_id}<->db_file{file_id}; db_task{school_id}<->db_school{school_id}; IF creator_type=c1, db_task{created_by}<->db_teacher{teacher_id}; IF creator_type=c2, db_task{created_by_admin_id}<->db_admin{admin_id}
extension_rule = same db_task table; creating an app-prefixed task copy is FORBIDDEN


[EMPTY_STATE]

IF no task records, return [] AND empty_title=暂无任务
IF selected filter has no rows, return [] AND empty_title=该状态暂无任务
IF new task has real assignees but no acceptance/completion, accepted_count=0, completed_count=0, status=wait_accept


[NAV_OBJECTS]

shared_navigation_source = dashboard-spec.md [NAV_OBJECTS]
required_namespace = nav_admin_*


[JUMP_VALIDATION]

IF task_id NOT_FROM authorized current_school query, return 404
IF task.school_id != current_school_id, return 403
IF selector enum invalid, deadline invalid, or no eligible assignees, return 422
IF admin lacks task.publish|task.urge, return 403
IF urge requested for complete|cancelled task, return 409
task creation and all assignment creation MUST be one transaction
client-supplied teacher_id list MUST be intersected with authorized active teachers in current_school_id
