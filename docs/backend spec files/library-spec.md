ADMIN_LIBRARY_BACKEND_OBJECT_SPEC

scope (范围) = index.html#view-library
source_page (参考页面) = index.html
static_node_count (固定操作节点数) = 13
dynamic_content_row_count (动态内容行数) = 0:k
dynamic_row_action_count (动态行操作数) = 0:3k
runtime_clickable_node_count (运行时可点击节点数) = 13:(13+3k)
field_format (字段格式) = field_key (中文字段名), cardinality, type|enum, ui
id_rule (ID规则) = integer, database_auto_generated
null_rule (空值规则) = 0:1
list_rule (列表规则) = 0:k | 1:k


[SHARED_OBJECT_RULE]

canonical_registry = docs/spec-handoff.md
shared_object_source = dashboard-spec.md; Teacher App home-spec.md|training-center-spec.md|school-affairs-spec.md|comprehensive-coordination-spec.md
shared_objects = db_admin, db_admin_school, db_school, db_teacher, db_file, db_resource, db_case, db_home_case, db_training_recommendation, db_party_study, db_party_activity, db_party_brand, db_coord_document, db_training
new_canonical_object_defined_here = db_content_metric (non-persistent aggregate view)
admin_page_aggregate = db_admin_library_home
resource_or_case_alias = FORBIDDEN


[CONTEXT_RULE]

admin_id = auth_session.admin_id
allowed_school_id = db_admin_school.school_id WHERE admin_id=current_admin_id AND active=1
current_school_id MUST IN allowed_school_id
permission = db_admin_school.role|permission_scope
required_permission = library.read; library.recommend; library.publish_state according to action
raw_identity_id_ui = context.hidden
client_editable = 0
backend_authorization_validation = REQUIRED


[DATA_INITIALIZATION_RULE]

prototype_content = HTML 中十条资源/案例、名称、教师、年级、浏览量、推荐标记、状态和示例 ID 均为 Mock
static_ui_content = 类型/分类/状态筛选项、衣食住行艺固定分类、表头、按钮名和空状态文案
business_seed = NONE
production_initial_db_resource = EMPTY
production_initial_db_case = EMPTY
production_initial_recommendation = EMPTY
dynamic_list_without_data = []
dynamic_count_without_data = 0
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
| 6 | 家园共育数据 | Home-School Data | nav_admin_home_school | nav_admin_home_school | NULL | index.html#homeschool |
| 7 | 组织管理 | Organization Management | nav_admin_org | nav_admin_org | NULL | index.html#org |
| 8 | 内容发布 | Content Publishing | nav_admin_content | nav_admin_content | NULL | index.html#content |
| 9 | 导出数据 | Export Data | btn_admin_export | db_admin_library_home | current filters | file download |
| 10 | 类型筛选 | Type Filter | btn_admin_library_type | db_resource + db_case | all|resource|case | local/API filter |
| 11 | 分类筛选 | Category Filter | btn_admin_library_category | db_resource + db_case | all|resource_tag | local/API filter |
| 12 | 状态筛选 | Status Filter | btn_admin_library_status | db_resource + db_case | all|approved|pending|withdrawn | local/API filter |
| 13 | 名称搜索 | Name Search | btn_admin_library_search | db_admin_library_home | query_text | local/API filter |


[DYNAMIC_CONTENT_NODE]

| node_name_cn | node_key | object | input | cardinality | action/jump |
|---|---|---|---|---|---|
| 资源/案例行 | admin_library_content_row | db_resource|db_case | content_type, resource_id|case_id FROM query_result | 0:k | NONE |
| 推荐/取消推荐 | admin_library_recommend | db_home_case|db_training_recommendation | content_type, object_id FROM query_result | 0:k | upsert/delete recommendation |
| 预览 | admin_library_preview | db_resource|db_case + db_file | object_id FROM query_result | 0:k | authorized preview |
| 上架/下架 | admin_library_publish_state | db_resource|db_case | object_id FROM query_result | 0:k | approved<->withdrawn |

dynamic_rule = 名称、类型、分类、年级、上传人、浏览量、状态、推荐标记和 object_id 必须来自 API；名称不得作为更新键


[PAGE_OBJECT]

管理端资源与案例首页 (Admin Library Home / db_admin_library_home)

admin_library_home_id (页面聚合ID), 1:1, integer, ui=admin_library.page
admin_id (管理员ID), 1:1, integer, ui=context.hidden
school_id (园所ID), 1:1, integer, ui=context.hidden
admin_school_id (授权ID), 1:1, integer, ui=context.hidden
resource_id (资源ID), 0:k, integer, ui=admin_library.rows
case_id (案例ID), 0:k, integer, ui=admin_library.rows
home_case_id (首页案例推荐ID), 0:k, integer, ui=admin_library.recommendation
training_recommendation_id (教研推荐ID), 0:k, integer, ui=admin_library.recommendation
content_metric_id (内容统计ID), 0:k, integer, ui=admin_library.views
file_id (预览文件ID), 0:k, integer, ui=admin_library.preview

rel_count (关系数量) = 9
rel_db (关联表) = db_admin, db_school, db_admin_school, db_resource, db_case, db_home_case, db_training_recommendation, db_content_metric, db_file
rel_map (关系字段) = db_admin_library_home{admin_id}<->db_admin{admin_id}; db_admin_library_home{school_id}<->db_school{school_id}; db_admin_library_home{admin_school_id}<->db_admin_school{admin_school_id}; db_admin_library_home{resource_id}<->db_resource{resource_id}; db_admin_library_home{case_id}<->db_case{case_id}; db_admin_library_home{home_case_id}<->db_home_case{home_case_id}; db_admin_library_home{training_recommendation_id}<->db_training_recommendation{training_recommendation_id}; db_admin_library_home{content_metric_id}<->db_content_metric{content_metric_id}; db_admin_library_home{file_id}<->db_file{file_id}
persist = 0
object_type = aggregate


[CONTENT_OBJECTS]

内容统计视图 (Content Metric / db_content_metric)

content_metric_id (内容统计ID), 1:1, integer, computed, ui=content.metric.hidden
school_id (园所ID), 1:1, integer, ui=context.hidden
content_type (内容类型), 1:1, c1=resource|c2=case|c3=party_study|c4=party_activity|c5=party_brand|c6=coord_document|c7=training, ui=content.metric.hidden
resource_id (资源ID), 0:1, integer, ui=context.hidden
case_id (案例ID), 0:1, integer, ui=context.hidden
study_id (党建学习ID), 0:1, integer, ui=context.hidden
activity_id (党建活动ID), 0:1, integer, ui=context.hidden
brand_id (品牌内容ID), 0:1, integer, ui=context.hidden
document_id (综合协调文档ID), 0:1, integer, ui=context.hidden
training_id (研修ID), 0:1, integer, ui=context.hidden
view_count (浏览次数), 1:1, integer, derived, ui=content.metric.views
download_count (下载次数), 1:1, integer, derived, ui=content.metric.downloads

rel_count (关系数量) = 8
rel_db (关联表) = db_school, db_resource, db_case, db_party_study, db_party_activity, db_party_brand, db_coord_document, db_training
rel_map (关系字段) = db_content_metric{school_id}<->db_school{school_id}; IF content_type=c1, db_content_metric{resource_id}<->db_resource{resource_id}; IF content_type=c2, db_content_metric{case_id}<->db_case{case_id}; IF content_type=c3, db_content_metric{study_id}<->db_party_study{study_id}; IF content_type=c4, db_content_metric{activity_id}<->db_party_activity{activity_id}; IF content_type=c5, db_content_metric{brand_id}<->db_party_brand{brand_id}; IF content_type=c6, db_content_metric{document_id}<->db_coord_document{document_id}; IF content_type=c7, db_content_metric{training_id}<->db_training{training_id}
persist = 0
object_type = aggregate_view
check = content_type 对应一个目标ID非空，其余目标ID为 NULL
zero_rule = no usage event -> view_count=0, download_count=0


[REUSED_OBJECT_USAGE]

db_resource|db_case|db_file = REUSE home-spec.md
db_home_case = REUSE case recommendation for Teacher App home
db_training_recommendation = REUSE placement=p2 resource_list for Teacher App training-center resource recommendation
status_ui_map = 已发布 -> approved(s3); 待审核 -> pending(s2); 已下架 -> withdrawn(s5)


[CANONICAL_FIELD_EXTENSION]

db_resource.resource_status ADD s5=withdrawn(已下架)
db_case.case_status ADD s5=withdrawn(已下架)
extension_rule = 保留原对象名和 s1:s4 含义；不得创建管理端前缀的资源/案例副本
recommendation_rule = 案例推荐写 db_home_case；资源推荐写 db_training_recommendation placement=p2


[EMPTY_STATE]

IF no content, return [] AND empty_title=暂无资源或案例
IF filters/search have no match, return [] AND empty_title=没有符合筛选条件的内容
IF no usage data, view_count=0 AND download_count=0
IF no recommendation, recommended=0; do not synthesize a recommendation row


[NAV_OBJECTS]

shared_navigation_source = dashboard-spec.md [NAV_OBJECTS]
required_namespace = nav_admin_*


[JUMP_VALIDATION]

IF content_type NOT_IN(resource,case), return 400
IF resource_id|case_id NOT_FROM current query_result, return 404
IF object.school_id != current_school_id, return 403
IF preview file not related to object, return 403
IF recommend resource, REQUIRE status=approved AND write db_training_recommendation placement=p2
IF recommend case, REQUIRE status=approved AND write db_home_case
IF publish state action lacks library.publish_state, return 403
IF pending/rejected/deleted object is requested by a non-review action, return 403
