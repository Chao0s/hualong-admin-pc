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
shared_objects = db_admin, db_school, db_teacher, db_file, db_file_ref, db_party_study, db_party_activity, db_party_brand, db_coord_document, db_training, db_training_participation, db_notification, db_content_access_event, db_content_metric
new_business_object = NONE
admin_page_aggregate = db_admin_content_home
generic_content_table = FORBIDDEN; each category writes its existing canonical object


[CONTEXT_RULE]

admin_id = auth_session.admin_id
allowed_school_id = db_admin.school_id WHERE admin_id=current_admin_id AND admin_status=s1
current_school_id MUST IN allowed_school_id
permission = db_admin.role|permission_scope
required_permission = content.party.write|content.coord.write|content.training.write according to category
raw admin_id|school_id|publisher_id ui = context.hidden
client_editable = 0
backend_authorization_validation = REQUIRED


[DATA_INITIALIZATION_RULE]

prototype_content = HTML 中八条党建/通知/培训内容、标题、发布部门/管理员、日期、浏览/下载、状态、附件和表单日期均为 Mock
static_ui_content = 三个一级栏目、十一种发布内容类型、表头、字段标签、操作说明和空状态文案
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
| 政策法规 | db_coord_document | coord_category=c1 policy |
| 通知文件 | db_coord_document | coord_category=c2 notice |
| 组织架构 | db_coord_document | coord_category=c3 org |
| 安全管理 | db_coord_document | coord_category=c4 safety |
| 卫生保健 | db_coord_document | coord_category=c5 health |
| 师德师风 | db_coord_document | coord_category=c6 ethics |
| 跟岗交流 | db_coord_document | coord_category=c7 exchange |
| 教研培训 | db_training | training event |

category_rule = 综合协调固定使用 c1:c7 七类，不提供自定义分类，也不得创建分类独立表


[FORM_FIELD_INDEX]

| visible field | canonical target | rule |
|---|---|---|
| 标题 | selected object title field | required, server-side length validation |
| 栏目 | CATEGORY_OBJECT_MAP | required controlled enum; changing category switches the remaining fields |
| 正文 | selected object content field | party study/activity/brand required or optional per canonical rule; max 2000 |
| 学习类型 | db_party_study.study_type | party study only; required t1 policy / t2 learning / t3 system |
| 发布部门 | db_party_study.publisher_department | party study only; optional |
| 外部视频链接 | db_party_study.video_links | party study only; 0:k {title max 100, HTTPS url}; copy-out, never backend upload |
| 活动时间 | db_party_activity.activity_at | party activity only; required |
| 活动地点 | db_party_activity.activity_location | party activity only; optional |
| 品牌标签 | db_party_brand.brand_tag | party brand only; optional controlled array |
| 发布日期 | selected study/brand published_at | required; activity uses activity_at |
| 主文件 | db_file_ref usage_key=main_file | party study requires at least one; activity/brand optional |
| 正文媒体 | db_file_ref usage_key=inline_media | optional |
| 下载附件 | db_file_ref usage_key=download | optional |
| 综合协调正文 | db_coord_document.document_content | coordination categories only; required, max 2000 |
| 综合协调发布部门 | db_coord_document.publisher_department | coordination categories only; optional, max 50 |
| 综合协调发布日期 | db_coord_document.published_at | coordination categories only; required |
| 综合协调生效日期 | db_coord_document.effective_date | optional only for c1 policy / c4 safety / c5 health; all other categories must omit |
| 综合协调主文件 | db_file_ref usage_key=main_file | coordination categories only; exactly one required |
| 综合协调正文媒体／下载附件 | db_file_ref usage_key=inline_media\|download | coordination categories only; optional |
| 研修正文 | db_training.training_content | training only; required, max 2000; list excerpt is computed first 100 chars |
| 研修开始／结束 | db_training.start_at\|end_at | training only; start required, end optional; absent end means end of start date in school timezone |
| 研修地点／主讲人 | db_training.location\|speaker | training only; location individually optional, but publish requires location or complete meeting pair; speaker optional |
| 线上会议入口 | db_training.meeting_link_title\|meeting_url | training only; 0:1 pair, title max 100 and URL HTTPS; both present or both absent |
| 研修材料 | db_file_ref | training only; all optional, no required main_file |

production_publish_rule = one modal switches target-specific controls when category changes; direct publish requires every canonical required field and file rule. Server defaults for omitted business fields are FORBIDDEN. Party and coordination content do not require a second completion page; incomplete coordination content may remain s1 draft.


[DYNAMIC_CONTENT_NODE]

| node_name_cn | node_key | object | input | cardinality | jump/action |
|---|---|---|---|---|---|
| 内容行 | admin_content_row | mapped canonical content object | content_type, object_id FROM query_result | 0:k | NONE |
| 预览 | admin_content_preview | mapped object + db_file | object_id FROM query_result | 0:k | authorized preview |
| 查看研修 | admin_content_training_view | db_training + db_file_ref | training_id FROM current training row | 0:k | s1/s5 readonly detail；only s0 opens draft editor |
| 参与管理 | admin_content_training_participation | db_training_participation + db_teacher | training_id FROM current training row | 0:k | open training-specific drawer/detail layer |
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
training_participation_id (研修参与ID), 0:k, integer, ui=admin_content.training.participation.rows
participant_teacher_id (参与教师ID), 0:k, integer, ui=admin_content.training.participation.teacher
file_id (附件ID), 0:k, integer, ui=admin_content.attachment
content_metric_id (内容统计ID), 0:k, integer, ui=admin_content.metric
publish_category (发布栏目选择), 0:1, party_study|party_activity|party_brand|coord_document|training, ui=admin_content.publish.category_select
publish_title (发布标题输入), 0:1, max_len=100, ui=admin_content.publish.title_input
publish_body (发布正文输入), 0:1, max_len=2000, ui=admin_content.publish.body_input

publish_form_binding (发布弹层控件绑定) = modal-content 是一个按栏目分派的表单：同一个标题框按 publish_category 写 db_party_study.study_title、db_party_activity.activity_title、db_party_brand.brand_title、db_coord_document.document_title 或 db_training.training_title，正文框同理写各自的 *_content。因此三栏登记在本 persist=0 聚合上，而不是任选一张目标表 —— 挂在任一张上都只对五分之一的栏目成立，而 data-ui 是写在 markup 上的单一值，无法随 tab 改写。[FORM_FIELD_INDEX] 的 CATEGORY_OBJECT_MAP 仍是「哪个栏目落哪张表」的权威，本三栏只负责「哪个控件」
publish_form_unbuilt (原型尚未建的控件) = 学习类型、发布部门、外部视频链接、活动时间／地点、品牌标签、发布日期、研修开始／结束／地点／主讲人、会议入口与三类 db_file_ref 附件在 index.html 里都还没有控件（附件只有一个 toast 按钮）。它们在 [FORM_FIELD_INDEX] 已有列级归属，等原型建出控件时再逐个补 ui= 标注，不预先造假路径

rel_count (关系数量) = 11
rel_db (关联表) = db_admin, db_school, db_teacher, db_party_study, db_party_activity, db_party_brand, db_coord_document, db_training, db_training_participation, db_file, db_content_metric
rel_map (关系字段) = db_admin_content_home{admin_id}<->db_admin{admin_id}; db_admin_content_home{school_id}<->db_school{school_id}; db_admin_content_home{participant_teacher_id}<->db_teacher{teacher_id}; db_admin_content_home{study_id}<->db_party_study{study_id}; db_admin_content_home{activity_id}<->db_party_activity{activity_id}; db_admin_content_home{brand_id}<->db_party_brand{brand_id}; db_admin_content_home{document_id}<->db_coord_document{document_id}; db_admin_content_home{training_id}<->db_training{training_id}; db_admin_content_home{training_participation_id}<->db_training_participation{training_participation_id}; db_admin_content_home{file_id}<->db_file{file_id}; db_admin_content_home{content_metric_id}<->db_content_metric{content_metric_id}
persist = 0
object_type = aggregate


[REUSED_OBJECT_USAGE]

db_party_study|db_party_activity|db_party_brand = REUSE school-affairs-spec.md
db_coord_document = REUSE comprehensive-coordination-spec.md and its single coord_category enum
db_training = REUSE home-spec.md
db_file = REUSE shared file metadata
db_content_metric = REUSE library-spec.md; it remains persist=0. Party and coordination view/download values aggregate db_content_access_event e3/e4; absent metric returns 0 / 0


[CANONICAL_FIELD_EXTENSION]

admin_creator_rule = business objects created in Admin App require an authenticated admin creator without inventing a service teacher
db_party_study|db_party_activity|db_party_brand ADD created_by_admin_id(0:1, admin_id); the field is required when the record is created by Admin App
db_coord_document DROP creator_type, created_by; ADD created_by_admin_id(1:1, admin_id); Admin App is the only producer
db_training DROP creator_type, created_by; ADD created_by_admin_id(1:1, admin_id); Admin App is the only producer
db_party_study.study_status|db_party_activity.activity_status|db_party_brand.brand_status ADD s5=withdrawn
db_coord_document.document_status = s1=draft|s3=approved/published|s5=withdrawn only
db_training DROP training_type; training_status = s0=draft|s1=published|s5=withdrawn only; temporal phase is computed
extension_rule = extend original canonical objects only; app-prefixed content/training copies are FORBIDDEN

party_field_rule = db_party_study.study_summary -> study_content(max 2000); video_url -> video_links(JSON {title,url}); db_party_activity.activity_intro -> activity_content(max 2000); db_party_brand.brand_summary -> brand_content(max 2000)
party_type_rule = study_type fixed t1/t2/t3; work norm/archive requirement map to t3 system; no custom type
party_file_rule = db_file_ref usage_key main_file|inline_media|download; study requires >=1 main_file, activity/brand allow zero files
party_publish_result = validated direct publish creates s3 approved content; incomplete payload returns 422

coord_field_rule = document_summary -> document_content(max 2000); card excerpt is computed first 100 chars; effective_date only c1|c4|c5 and display-only; allow_preview|allow_download removed
coord_file_rule = db_file_ref usage_key main_file|inline_media|download; exactly one main_file required, other references optional
coord_publish_result = complete validated payload may publish directly as s3; partial payload may save only as s1 draft; teacher submission and s2|s4 review states are FORBIDDEN
training_field_rule = training_intro -> training_content(max 2000); card excerpt is computed first 100 chars; no credit_hours field; all db_file_ref materials optional; at most one meeting_link_title(max 100)+meeting_url(HTTPS) pair; publish requires location OR complete meeting pair, and both means hybrid
training_publish_result = authorized admin may save s0 draft and directly publish a complete payload as s1; teacher creation is FORBIDDEN; training_type is removed and temporal phase is derived
training_withdrawal_before_end = in one transaction set training_status=s5, convert every current s1 registered participation to s2 cancelled with cancelled_at, and create one n5 system notification for each affected teacher; hide activity, files and meeting immediately
training_withdrawal_after_end = set training_status=s5 but preserve completed participation; My Training may return withdrawn title/time shell, while files, meeting and public feedback are forbidden
training_withdrawn_terminal = s5 cannot return to s1 and cannot be edited；a new event is required to hold another session
training_admin_participation = start_at<=NOW<effective_end_at only, authorized admin may add a new s1 row, restore an existing s2 row to s1, or cancel s1 to s2; every action writes B2 generic operation log and creates one n5 notification for the affected teacher; unchanged creates neither notification nor duplicate row; NOW>=effective_end_at is frozen
training_participation_ui_surface = current Content Publishing > Training row action “参与管理” opens a training-specific drawer/detail layer；no new top-level page and no placement in teacher directory
training_participation_permission = OPEN next Q58 frontier；不得在未拍板前静默复用 content.training.write 或自行新增权限键
training_reschedule = F16 FORBIDS after s0→s1；start_at/end_at are draft-only. Schedule change requires withdrawing the old activity and creating a new s0; participation rows stay on the old activity and are not migrated
training_published_edit = F16 FORBIDS all s1 content/file writes regardless of NOW；title, content, location, meeting pair, speaker and materials are immutable. Only the existing withdrawal flow remains


[PARTY_METRIC_RULE]

teacher_detail_success = append db_content_access_event e3 viewed
teacher_file_success = append db_content_access_event e4 downloaded with file_id
repeat_rule = repeated successful actions are repeated events
exclude = failed request|admin preview|external link copy
counter_rule = SELECT COUNT(*) from event source; stored mutable counters are FORBIDDEN

[COORD_METRIC_RULE]

teacher_sheet_success = append db_content_access_event e3 viewed with content_type=c6 and document_id
teacher_file_success = append db_content_access_event e4 downloaded with content_type=c6, document_id and file_id
repeat_rule = repeated successful actions are repeated events
exclude = failed request|admin preview
receipt_rule = coordination notice files do not create read/completion receipts
counter_rule = SELECT COUNT(*) from event source; stored mutable counters are FORBIDDEN

| canonical object after extension | rel_count | rel_db | rel_map |
|---|---:|---|---|
| db_party_study | 3 | db_school, db_file, db_admin | db_party_study{school_id}<->db_school{school_id}; db_party_study{file_id}<->db_file{file_id}; db_party_study{created_by_admin_id}<->db_admin{admin_id} |
| db_party_activity | 3 | db_school, db_file, db_admin | db_party_activity{school_id}<->db_school{school_id}; db_party_activity{file_id}<->db_file{file_id}; db_party_activity{created_by_admin_id}<->db_admin{admin_id} |
| db_party_brand | 3 | db_school, db_file, db_admin | db_party_brand{school_id}<->db_school{school_id}; db_party_brand{file_id}<->db_file{file_id}; db_party_brand{created_by_admin_id}<->db_admin{admin_id} |
| db_coord_document | 3 | db_school, db_file_ref|db_file, db_admin | db_coord_document{school_id}<->db_school{school_id}; db_coord_document{document_id}<->db_file_ref{owner_id WHERE owner_object=db_coord_document}<->db_file{file_id}; db_coord_document{created_by_admin_id}<->db_admin{admin_id} |
| db_training | 3 | db_school, db_file_ref|db_file, db_admin | db_training{school_id}<->db_school{school_id}; db_training{training_id}<->db_file_ref{owner_id WHERE owner_object=db_training}<->db_file{file_id}; db_training{created_by_admin_id}<->db_admin{admin_id} |


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
