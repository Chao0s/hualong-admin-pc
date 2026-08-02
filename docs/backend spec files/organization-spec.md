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
| teacher | 所在班级 | create db_teacher_class.class_id | 1 |
| teacher | 职务 | db_teacher_profile.job_role and/or db_teacher_class.assignment_role | 1 |
| teacher | 联系电话 | db_teacher.phone | 0 |
| class | 班级名称 | db_class.class_name | 1 |
| class | 年级 | db_class.grade | 1 |
| class | 带班教师 | create db_teacher_class.teacher_id, assignment_role=lead | 0; “待分配” creates no relation |
| class | 计划幼儿数 | db_class.planned_child_count | 1 |
| child | 幼儿姓名 | db_child.child_name | 1 |
| child | 所在班级 | db_child.class_id | 1 |
| child | 性别 | db_child.gender | 1 |
| child | 家长联系人 | db_parent + db_parent_child | 1; production UI/API MUST submit structured parent_name and phone |


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
password reset MUST issue a one-time reset channel; hardcoded/default password response is FORBIDDEN
