ADMIN_CONTENT_PUBLISHING_BACKEND_OBJECT_SPEC

scope (范围) = index.html#content
source_page (参考页面) = index.html
static_node_count (固定按钮节点数) = 17
dynamic_content_row_count (动态内容行数) = 0:k
dynamic_row_action_count (动态行操作数) = 0:2k
runtime_clickable_node_count (运行时可点击节点数) = 17:(17+2k)
field_format (字段格式) = field_key (中文字段名), cardinality, type|enum, ui
id_rule (ID规则) = integer, database_auto_generated
null_rule (空值规则) = 0:1
list_rule (列表规则) = 0:k | 1:k


[SHARED_OBJECT_RULE]

canonical_registry = docs/spec-handoff.md
shared_object_source = dashboard-spec.md; library-spec.md; Teacher App school-affairs-spec.md|comprehensive-coordination-spec.md|home-spec.md
shared_objects = db_admin, db_admin_school, db_school, db_file, db_party_study, db_party_activity, db_party_brand, db_coord_document, db_training, db_content_metric
new_business_object = NONE
admin_page_aggregate = db_admin_content_home
generic_content_table = FORBIDDEN; each category writes its existing canonical object


[CONTEXT_RULE]

admin_id = auth_session.admin_id
allowed_school_id = db_admin_school.school_id WHERE admin_id=current_admin_id AND is_active=1
current_school_id MUST IN allowed_school_id
permission = db_admin_school.role|permission_scope
required_permission = content.party.write|content.coord.write|content.training.write according to category
raw admin_id|school_id|publisher_id ui = context.hidden
client_editable = 0
backend_authorization_validation = REQUIRED


[DATA_INITIALIZATION_RULE]

prototype_content = HTML 中八条党建/通知/培训内容、标题、发布部门/管理员、日期、浏览/下载、状态、附件和表单日期均为 Mock
static_ui_content = 三个一级栏目、六个发布分类、表头、字段标签、操作说明和空状态文案
business_seed = NONE
production_initial_content_objects = db_party_study|db_party_activity|db_party_brand|db_coord_document|db_training EMPTY
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
| 10 | 导出数据 | Export Data | btn_admin_export | db_admin_content_home | current tab | file download |
| 11 | 党建学习 | Party Content Tab | btn_admin_content_party | db_party_study + db_party_activity + db_party_brand | NULL | local/API tab |
| 12 | 通知文件 | Coordination Content Tab | btn_admin_content_coord | db_coord_document | NULL | local/API tab |
| 13 | 教研培训 | Training Content Tab | btn_admin_content_training | db_training | NULL | local/API tab |
| 14 | 发布内容 | Open Publish | btn_admin_content_open_publish | selected canonical object | NULL | open modal |
| 15 | 上传附件 | Upload Attachment | btn_admin_content_upload_file | db_file | local file | upload and return file_id |
| 16 | 取消 | Cancel | btn_admin_content_cancel | selected canonical object | NULL | close modal |
| 17 | 发布 | Publish | btn_admin_content_publish | selected canonical object | validated form + file_id | create/publish |


[CATEGORY_OBJECT_MAP]

| UI category | canonical object | canonical category/type |
|---|---|---|
| 党建学习 | db_party_study | study_type=learning or explicit selected subtype |
| 党建活动 | db_party_activity | N/A |
| 品牌建设 | db_party_brand | N/A |
| 通知文件 | db_coord_document | coord_category=c2 notice |
| 政策法规 | db_coord_document | coord_category=c1 policy |
| 教研培训 | db_training | training event |

category_rule = 卫生保健/安全管理如从列表编辑，仍写 db_coord_document.coord_category=c5|c4；不得创建通知/卫生/安全独立表


[FORM_FIELD_INDEX]

| visible field | canonical target | rule |
|---|---|---|
| 标题 | selected object title field | required, server-side length validation |
| 栏目 | CATEGORY_OBJECT_MAP | required controlled enum |
| 正文/简介 | selected object summary/intro field | validate target-specific length |
| 附件 | selected object.file_id -> db_file.file_id | optional in prototype; target schema may require it |

production_publish_rule = current generic modal omits required fields such as activity_at, training start_at/end_at and some mandatory file/date fields; it may save a draft, but direct publish MUST return 422 until target-specific required fields are supplied


[DYNAMIC_CONTENT_NODE]

| node_name_cn | node_key | object | input | cardinality | jump/action |
|---|---|---|---|---|---|
| 内容行 | admin_content_row | mapped canonical content object | content_type, object_id FROM query_result | 0:k | NONE |
| 预览 | admin_content_preview | mapped object + db_file | object_id FROM query_result | 0:k | authorized preview |
| 下线 | admin_content_withdraw | mapped canonical object | object_id FROM query_result | 0:k | set canonical withdrawn status |

dynamic_rule = 标题、栏目、发布人、时间、浏览/下载、状态、附件和 object_id 来自 API；标题不得作为删除/下线键


[PAGE_OBJECT]

管理端内容发布首页 (Admin Content Home / db_admin_content_home)

admin_content_home_id (页面聚合ID), 1:1, integer, ui=admin_content.page
admin_id (管理员ID), 1:1, integer, ui=context.hidden
school_id (园所ID), 1:1, integer, ui=context.hidden
admin_school_id (授权ID), 1:1, integer, ui=context.hidden
study_id (党建学习ID), 0:k, integer, ui=admin_content.party.rows
activity_id (党建活动ID), 0:k, integer, ui=admin_content.party.rows
brand_id (品牌建设ID), 0:k, integer, ui=admin_content.party.rows
document_id (协调文档ID), 0:k, integer, ui=admin_content.coord.rows
training_id (教研培训ID), 0:k, integer, ui=admin_content.training.rows
file_id (附件ID), 0:k, integer, ui=admin_content.attachment
content_metric_id (内容统计ID), 0:k, integer, ui=admin_content.metric

rel_count (关系数量) = 10
rel_db (关联表) = db_admin, db_school, db_admin_school, db_party_study, db_party_activity, db_party_brand, db_coord_document, db_training, db_file, db_content_metric
rel_map (关系字段) = db_admin_content_home{admin_id}<->db_admin{admin_id}; db_admin_content_home{school_id}<->db_school{school_id}; db_admin_content_home{admin_school_id}<->db_admin_school{admin_school_id}; db_admin_content_home{study_id}<->db_party_study{study_id}; db_admin_content_home{activity_id}<->db_party_activity{activity_id}; db_admin_content_home{brand_id}<->db_party_brand{brand_id}; db_admin_content_home{document_id}<->db_coord_document{document_id}; db_admin_content_home{training_id}<->db_training{training_id}; db_admin_content_home{file_id}<->db_file{file_id}; db_admin_content_home{content_metric_id}<->db_content_metric{content_metric_id}
persist = 0
object_type = aggregate


[REUSED_OBJECT_USAGE]

db_party_study|db_party_activity|db_party_brand = REUSE school-affairs-spec.md
db_coord_document = REUSE comprehensive-coordination-spec.md and its single coord_category enum
db_training = REUSE home-spec.md
db_file = REUSE shared file metadata
db_content_metric = REUSE library-spec.md; absent metric returns 0 / 0


[CANONICAL_FIELD_EXTENSION]

admin_creator_rule = business objects created in Admin App require an authenticated admin creator without inventing a service teacher
db_party_study|db_party_activity|db_party_brand ADD created_by_admin_id(0:1, admin_id); the field is required when the record is created by Admin App
db_coord_document ADD creator_type(c1=teacher|c2=admin), created_by_admin_id(0:1); CHANGE created_by to 0:1; exactly one creator ID required
db_training ADD creator_type(c1=teacher|c2=admin), created_by_admin_id(0:1); CHANGE created_by to 0:1; exactly one creator ID required
db_party_study.study_status|db_party_activity.activity_status|db_party_brand.brand_status|db_coord_document.document_status ADD s5=withdrawn
db_training.training_status ADD s0=draft and s5=withdrawn
extension_rule = extend original canonical objects only; app-prefixed content/training copies are FORBIDDEN

| canonical object after extension | rel_count | rel_db | rel_map |
|---|---:|---|---|
| db_party_study | 3 | db_school, db_file, db_admin | db_party_study{school_id}<->db_school{school_id}; db_party_study{file_id}<->db_file{file_id}; db_party_study{created_by_admin_id}<->db_admin{admin_id} |
| db_party_activity | 3 | db_school, db_file, db_admin | db_party_activity{school_id}<->db_school{school_id}; db_party_activity{file_id}<->db_file{file_id}; db_party_activity{created_by_admin_id}<->db_admin{admin_id} |
| db_party_brand | 3 | db_school, db_file, db_admin | db_party_brand{school_id}<->db_school{school_id}; db_party_brand{file_id}<->db_file{file_id}; db_party_brand{created_by_admin_id}<->db_admin{admin_id} |
| db_coord_document | 4 | db_school, db_teacher, db_file, db_admin | db_coord_document{school_id}<->db_school{school_id}; IF creator_type=c1, db_coord_document{created_by}<->db_teacher{teacher_id}; db_coord_document{file_id}<->db_file{file_id}; IF creator_type=c2, db_coord_document{created_by_admin_id}<->db_admin{admin_id} |
| db_training | 4 | db_school, db_teacher, db_file, db_admin | db_training{school_id}<->db_school{school_id}; IF creator_type=c1, db_training{created_by}<->db_teacher{teacher_id}; db_training{file_id}<->db_file{file_id}; IF creator_type=c2, db_training{created_by_admin_id}<->db_admin{admin_id} |


[EMPTY_STATE]

IF selected tab has no records, return [] AND empty_title=该栏目暂无内容
IF no metric, view_count=0 AND download_count=0
IF attachment list empty, return []
draft without required target fields remains draft and is not visible in Teacher App


[NAV_OBJECTS]

shared_navigation_source = dashboard-spec.md [NAV_OBJECTS]
required_namespace = nav_admin_*


[JUMP_VALIDATION]

IF category not in CATEGORY_OBJECT_MAP, return 400
IF object_id NOT_FROM authorized current_school query, return 404
IF object.school_id != current_school_id, return 403
IF file_id not uploaded/authorized for current admin and school, return 403
IF target-specific required fields missing, return 422 and do not mark published/open
IF permission missing for mapped category, return 403
IF withdraw requested for already withdrawn/deleted content, return 409
content creation/status update MUST use admin_id from auth context; title/date/by values from HTML are forbidden
