ADMIN_LIBRARY_BACKEND_OBJECT_SPEC

scope (范围) = index.html#library
source_page (参考页面) = index.html
static_node_count (固定操作节点数) = 25
dynamic_content_row_count (动态内容行数) = 0:k
dynamic_row_action_count (动态行操作数) = 0:3k
runtime_clickable_node_count (运行时可点击节点数) = 25:(25+3k)
field_format (字段格式) = field_key (中文字段名), cardinality, type|enum, ui
id_rule (ID规则) = integer, database_auto_generated
null_rule (空值规则) = 0:1
list_rule (列表规则) = 0:k | 1:k


[SHARED_OBJECT_RULE]

canonical_registry = docs/spec-handoff.md
shared_object_source = dashboard-spec.md; Teacher App home-spec.md|training-center-spec.md|school-affairs-spec.md|comprehensive-coordination-spec.md
shared_objects = db_admin, db_admin_school, db_school, db_school_term, db_teacher, db_file, db_resource, db_case, db_home_case, db_training_recommendation, db_party_study, db_party_activity, db_party_brand, db_coord_document, db_training
new_canonical_object_defined_here = db_content_metric (non-persistent aggregate view)
admin_page_aggregate = db_admin_library_home
resource_or_case_alias = FORBIDDEN


[CONTEXT_RULE]

admin_id = auth_session.admin_id
allowed_school_id = db_admin_school.school_id WHERE admin_id=current_admin_id AND is_active=1
current_school_id MUST IN allowed_school_id
permission = db_admin_school.role|permission_scope
required_permission = library.read; library.recommend; library.publish_state according to action
raw_identity_id_ui = context.hidden
client_editable = 0
backend_authorization_validation = REQUIRED


[DATA_INITIALIZATION_RULE]

prototype_content = HTML 中十条资源/案例、名称、教师、年级、提交时间、浏览量、推荐标记、状态、年级覆盖数、学期日期和示例 ID 均为 Mock
static_ui_content = 类型/分类/年级/状态筛选项、衣食住行艺固定分类、表头、按钮名、覆盖率标签和空状态文案
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
| 6 | 家园共育数据 | Home-School Data | nav_admin_home_school | nav_admin_home_school | NULL | index.html#home-school |
| 7 | 组织管理 | Organization Management | nav_admin_org | nav_admin_org | NULL | index.html#org |
| 8 | 内容发布 | Content Publishing | nav_admin_content | nav_admin_content | NULL | index.html#content |
| 9 | 成长册设置 | Growth Book Setting | nav_admin_growth_book | nav_admin_growth_book | NULL | index.html#growthbook |
| 10 | 导出数据 | Export Data | btn_admin_export | db_admin_library_home | current filters | file download |
| 11 | 类型筛选 | Type Filter | btn_admin_library_type | db_resource + db_case | all|resource|case | local/API filter |
| 12 | 分类筛选 | Category Filter | btn_admin_library_category | db_resource + db_case | all|resource_tag | local/API filter |
| 13 | 状态筛选 | Status Filter | btn_admin_library_status | db_resource + db_case | all|approved|pending|withdrawn | local/API filter |
| 14 | 名称搜索 | Name Search | btn_admin_library_search | db_admin_library_home | query_text | local/API filter |
| 15 | 年级筛选 | Grade Filter | btn_admin_library_grade | db_resource + db_case | all|k1|k2|k3|general | local/API filter |
| 16 | 名称排序 | Sort Name | btn_admin_library_sort_name | db_admin_library_home | asc|desc | local only |
| 17 | 类型排序 | Sort Type | btn_admin_library_sort_type | db_admin_library_home | asc|desc | local only |
| 18 | 分类排序 | Sort Category | btn_admin_library_sort_category | db_admin_library_home | asc|desc | local only |
| 19 | 年级排序 | Sort Grade | btn_admin_library_sort_grade | db_admin_library_home | asc|desc | local only |
| 20 | 上传人排序 | Sort Uploader | btn_admin_library_sort_uploader | db_admin_library_home | asc|desc | local only |
| 21 | 浏览量排序 | Sort Views | btn_admin_library_sort_views | db_content_metric | asc|desc | local only |
| 22 | 状态排序 | Sort Status | btn_admin_library_sort_status | db_admin_library_home | asc|desc | local only |
| 23 | 推荐状态排序 | Sort Recommendation | btn_admin_library_sort_recommendation | db_admin_library_home | asc|desc | local only |
| 24 | 学期时间筛选 | Term Window Filter | btn_admin_library_term | db_school_term + db_resource + db_case | term_id|all | API filter by created_at window |
| 25 | 提交时间排序 | Sort Submitted Time | btn_admin_library_sort_created_at | db_admin_library_home | asc|desc | local only |


[DYNAMIC_CONTENT_NODE]

| node_name_cn | node_key | object | input | cardinality | action/jump |
|---|---|---|---|---|---|
| 资源/案例行 | admin_library_content_row | db_resource|db_case | content_type, resource_id|case_id FROM query_result | 0:k | NONE |
| 推荐/取消推荐 | admin_library_recommend | db_home_case|db_training_recommendation | content_type, object_id FROM query_result | 0:k | upsert/toggle recommendation |
| 预览 | admin_library_preview | db_resource|db_case + db_file | object_id FROM query_result | 0:k | authorized preview |
| 上架/下架 | admin_library_publish_state | db_resource|db_case | object_id FROM query_result | 0:k | approved<->withdrawn |
| 年级推荐覆盖率 | admin_library_grade_coverage | db_home_case + db_case | current_school_id | 3 | NONE |

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
grade_case_coverage (年级推荐覆盖率), 3, composite(k1|k2|k3 each 0:3), ui=admin_library.coverage
term_id (筛选学期ID), 0:1, school_term, ui=admin_library.term_filter

rel_count (关系数量) = 10
rel_db (关联表) = db_admin, db_school, db_school_term, db_admin_school, db_resource, db_case, db_home_case, db_training_recommendation, db_content_metric, db_file
rel_map (关系字段) = db_admin_library_home{admin_id}<->db_admin{admin_id}; db_admin_library_home{school_id}<->db_school{school_id}; db_admin_library_home{term_id}<->db_school_term{term_id}; db_admin_library_home{admin_school_id}<->db_admin_school{admin_school_id}; db_admin_library_home{resource_id}<->db_resource{resource_id}; db_admin_library_home{case_id}<->db_case{case_id}; db_admin_library_home{home_case_id}<->db_home_case{home_case_id}; db_admin_library_home{training_recommendation_id}<->db_training_recommendation{training_recommendation_id}; db_admin_library_home{content_metric_id}<->db_content_metric{content_metric_id}; db_admin_library_home{file_id}<->db_file{file_id}
persist = 0
object_type = aggregate


[GRADE_COVERAGE_RULE]

placement = index.html#library 推荐表格上方；不得放在 dashboard
grades = k1(小班)|k2(中班)|k3(大班)
query_rule = 对每个 grade 计算 LEAST(COUNT(*),3)，来源为 db_home_case JOIN db_case WHERE db_home_case.school_id=current_school_id AND db_case.school_id=current_school_id AND db_case.case_grade=grade AND db_case.case_status=s3 AND db_home_case.is_visible=1
display_rule = 固定显示三格“实际可命中数 / 3”；count<3 显示告警；count=0 显示重度告警
filter_independence = 覆盖率不受资源与案例表格当前类型、分类、状态筛选或名称搜索影响
empty_rule = 生产环境空表显示 k1=0/3|k2=0/3|k3=0/3；不得以 Mock 补足


[TABLE_VIEW_RULE]

sortable_columns = name|type|category|grade|uploader|created_at|views|status|recommended
non_sortable_columns = actions；操作不是数据列
sort_interaction = 点击列名按 asc -> desc 循环；切换到新列时从 asc 开始
sort_persistence = current page session only；刷新后使用 API 原始顺序；不得写 db_home_case.display_order 或任何业务表
grade_filter = all|k1(小班)|k2(中班)|k3(大班)|general(通用)
filter_composition = 年级可与类型、分类、状态和名称搜索同时生效；表格排序在过滤后执行
term_filter = 选 term_id 后由服务端读取同园 db_school_term，以 resource|case.created_at >= start_date 00:00:00 AND < end_date+1 day 过滤；all 不加时间条件
term_default = 后端返回 current_term_id（CURRENT_DATE BETWEEN start_date AND end_date）；寒暑假无命中时的默认值仍开放（USER-JOURNEY Q55-d1）


[TERM_OBJECT]

园所学期 (School Term / db_school_term)

school_term_id (园所学期记录ID), 1:1, integer, ui=context.hidden
school_id (园所ID), 1:1, integer, ui=context.hidden
term_id (学期ID), 1:1, max_len=20, ui=admin_library.term_filter
term_name (学期名称), 1:1, max_len=50, ui=admin_library.term_filter.label
start_date (开始日期), 1:1, date, ui=admin_library.term_filter.hidden
end_date (结束日期), 1:1, date, ui=admin_library.term_filter.hidden
created_at (创建时间), 1:1, datetime, ui=context.hidden
updated_at (更新时间), 1:1, datetime, ui=context.hidden

rel_count = 1
rel_db = db_school
rel_map = db_school_term{school_id}<->db_school{school_id}
unique = school_id, term_id
check = start_date <= end_date
overlap_rule = 同一 school_id 的 [start_date,end_date] 不得重叠；创建/更新时在同一事务内检查
current_rule = CURRENT_DATE BETWEEN start_date AND end_date；is_current 为接口派生值，不落列


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
db_training_recommendation = REUSE placement=p2 resource_list|p3 case_list for Teacher App training-center and partner recommendation；top featured is derived union
status_ui_map = 已发布 -> approved(s3); 待审核 -> pending(s2); 已下架 -> withdrawn(s5)


[CANONICAL_FIELD_EXTENSION]

db_resource.resource_status ADD s5=withdrawn(已下架)
db_case.case_status ADD s5=withdrawn(已下架)
extension_rule = 保留原对象名和 s1:s4 含义；不得创建管理端前缀的资源/案例副本
recommendation_rule = 资源推荐只 upsert db_training_recommendation placement=p2。案例推荐须同一交易 upsert db_home_case 与 db_training_recommendation placement=p3；取消时两列都转 is_visible=0，重推复用原列转 1 并更新 updated_at；任一写入失败整笔回滚
recommendation_ordering = db_home_case 按 updated_at DESC, home_case_id DESC；db_training_recommendation 三个展示区均按 updated_at DESC, training_recommendation_id DESC。管理表格的 local sort 不得写任何推荐业务列，且不提供上移/下移
recommendation_lifecycle = 两表均为简单二态，不提供 display_order／start_at／end_at 的 UI；db_training_recommendation 由 authenticated admin 写 created_by_admin_id，教师 creator FORBIDDEN


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
IF selected term_id NOT_IN db_school_term WHERE school_id=current_school_id, return 403
IF preview file not related to object, return 403
IF recommend resource, REQUIRE status=approved AND write db_training_recommendation placement=p2
IF recommend case, REQUIRE status=approved AND atomically write db_home_case + db_training_recommendation placement=p3
IF cancel case recommendation, atomically set both rows is_visible=0
IF publish state action lacks library.publish_state, return 403
IF pending/rejected/deleted object is requested by a non-review action, return 403
