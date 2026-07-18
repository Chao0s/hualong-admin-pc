# Cross-App Spec Handoff

## 1. Purpose

This file transfers the specification rules from the Teacher App to the Parent App and Admin App.

All three apps belong to the same kindergarten, communicate with each other, and use the same database. UI pages and page aggregates may be app-specific, but the same business record must always reuse the same canonical `db_*` object.

Teacher App reference specs:

- `home-spec.md`
- `school-affairs-spec.md`
- `comprehensive-coordination-spec.md`
- `training-center-spec.md`
- `home-school-spec.md`


## 2. Non-Negotiable Rules

1. Read this handoff and all available Teacher App specs before creating Parent/Admin specs.
2. One real-world business entity has one canonical `db_*` name across all apps.
3. Never create an app-prefixed copy of a shared business table merely because another app displays it.
4. App-specific page aggregates and navigation nodes must use app-specific namespaces.
5. HTML sample content is Mock data unless explicitly identified as real configuration data.
6. Production business tables start empty. Do not seed sample documents, tasks, cases, children, percentages, statuses, IDs, dates, or counts.
7. Static navigation, page labels, category labels, instructions, and empty-state copy may remain in production.
8. Base identity/roster data may be initialized by deployment or an authorized administrator.
9. IDs are database-generated. Hard-coded IDs such as `case_id=1` are forbidden.
10. Raw identity IDs are not user-editable. The backend derives and validates them from the authenticated context.
11. Every `rel_count` must equal the number of objects in `rel_db`.
12. Every relationship must be listed in `rel_map` using `object{field}<->object{field}`.
13. Dynamic card titles, labels, counts, images, dates, statuses, and IDs must come from API results.
14. If a shared object already exists, reference it with `REUSE`; do not repeat or rename its schema in another file.


## 3. Canonical Existing Object Registry

### Identity and access context

| Canonical object | Meaning | Defined in |
|---|---|---|
| `db_school` | Kindergarten/school | `home-spec.md` |
| `db_teacher` | Teacher identity | `home-spec.md` |
| `db_class` | Class identity | `home-spec.md` |
| `db_teacher_class` | Authorized teacher-class relationship | `home-spec.md` |
| `db_child` | Child identity and class membership | `home-school-spec.md` |
| `db_file` | Shared uploaded file/media metadata | `home-spec.md` |

### Tasks and submissions

| Canonical object | Meaning | Defined in |
|---|---|---|
| `db_task` | Internal teacher work task | `home-spec.md` |
| `db_task_assign` | Teacher work-task assignment | `home-spec.md` |
| `db_parent_task` | Parent-child task issued by teacher | `home-school-spec.md` |
| `db_parent_task_submission` | Parent/child submission for a parent task | `home-school-spec.md` |

`db_task` and `db_parent_task` are different business entities and must not be merged.

### Teaching content and training

| Canonical object | Meaning | Defined in |
|---|---|---|
| `db_upload` | Upload workflow record | `home-spec.md` |
| `db_resource` | Course resource | `home-spec.md` |
| `db_case` | Course case | `home-spec.md` |
| `db_resource_case` | Resource-case many-to-many relation | `home-spec.md` |
| `db_training` | Teaching research/training event | `home-spec.md` |
| `db_training_recommendation` | Training-center recommendation placement | `training-center-spec.md` |

### Assessment, growth, and home-school collaboration

| Canonical object | Meaning | Defined in |
|---|---|---|
| `db_assessment` | Quality assessment | `home-spec.md` |
| `db_assessment_item` | Assessment item and score | `home-spec.md` |
| `db_moment` | Kindergarten moment/activity | `home-spec.md` |
| `db_moment_upload` | Per-child moment upload/progress | `home-spec.md` |
| `db_month_eval` | Teacher monthly child evaluation | `home-spec.md` |
| `db_growth_record` | Child growth-record aggregate | `home-school-spec.md` |
| `db_growth_book` | Child growth book | `home-school-spec.md` |
| `db_home_school_progress` | Non-persistent home-school progress view | `home-school-spec.md` |
| `db_community_submission` | Community coeducation submission | `home-school-spec.md` |

### Party affairs and coordination

| Canonical object | Meaning | Defined in |
|---|---|---|
| `db_party_study` | Party-study document/content | `school-affairs-spec.md` |
| `db_party_activity` | Party activity | `school-affairs-spec.md` |
| `db_party_brand` | Party brand-building content | `school-affairs-spec.md` |
| `db_party_feature` | Party-page featured/banner placement | `school-affairs-spec.md` |
| `db_coord_document` | Coordination document with category enum | `comprehensive-coordination-spec.md` |

Do not create seven coordination document tables. Reuse `db_coord_document.coord_category`.

### Teacher App page aggregates

| Canonical object | Page |
|---|---|
| `db_home` | Teacher home |
| `db_party_home` | Teacher party-affairs home |
| `db_coord_home` | Teacher coordination home |
| `db_training_home` | Teacher training-center home |
| `db_home_school` | Teacher home-school home |

These page aggregates are Teacher App read models and must not be reused as Parent/Admin page aggregates.


## 4. Reserved Names for New App Identity Objects

The following names are reserved for cross-app consistency. Define their fields once when first needed, then reuse them:

| Reserved object | Intended meaning |
|---|---|
| `db_parent` | Parent/guardian identity |
| `db_parent_child` | Authorized parent-child relationship |
| `db_admin` | Admin identity |
| `db_admin_school` | Authorized admin-school relationship and role scope |

Before creating another identity object, check whether one of these names represents the same entity.


## 5. App-Specific Naming

### Page aggregates

- Teacher App: existing names such as `db_home`, `db_training_home`.
- Parent App: use `db_parent_home`, `db_parent_<module>_home`.
- Admin App: use `db_admin_home`, `db_admin_<module>_home`.

Page aggregates should normally use:

```text
persist = 0
object_type = aggregate
```

### Navigation objects

- Teacher App: `nav_home`, `nav_party`, `nav_coord`, `nav_training`, `nav_home_school`.
- Parent App: `nav_parent_<page>`.
- Admin App: `nav_admin_<page>`.

Navigation objects are UI routes, not database tables.

### Node keys

- Static action: `btn_<app-or-module>_<action>`
- Dynamic card: `<app-or-module>_<entity>_card`
- Navigation: follow the app-specific `nav_*` namespace


## 6. Context and Security Rules

### Teacher App

```text
teacher_id = auth_session.teacher_id
school_id = db_teacher.school_id
allowed_class_id = db_teacher_class.class_id WHERE active=1
current_class_id MUST IN allowed_class_id
```

### Parent App

```text
parent_id = auth_session.parent_id
allowed_child_id = db_parent_child.child_id WHERE active=1
current_child_id MUST IN allowed_child_id
school_id = db_child.school_id
class_id = db_child.class_id
```

### Admin App

```text
admin_id = auth_session.admin_id
allowed_school_id = db_admin_school.school_id WHERE active=1
current_school_id MUST IN allowed_school_id
permission = db_admin_school.role|permission_scope
```

For all apps:

```text
raw identity ID ui = context.hidden
client_editable = 0
backend authorization validation = REQUIRED
```

`hidden` means the raw ID is not shown or directly editable. It is not a security control by itself.


## 7. Cross-App Reuse Examples

### Parent task flow

```text
Teacher App creates db_parent_task
Parent App reads db_parent_task
Parent App creates/updates db_parent_task_submission
Teacher App reads the same db_parent_task_submission
```

Do not create `db_teacher_parent_task` and `db_parent_app_task`.

### Kindergarten moments

```text
Teacher App creates db_moment and db_moment_upload
Parent App reads authorized records for its child
Admin App may aggregate the same records within its school scope
```

### Resources and cases

```text
Teacher/Admin workflows create or review db_resource and db_case
Any authorized app reads the same approved records
App-specific featured placement may use a separate recommendation object
```

The content object is shared; recommendation/page-placement objects may be app-specific.


## 8. Mock and Production Empty-State Rules

Treat all HTML examples as Mock by default, including:

- uploaded documents and attachments;
- resource/case cards and banners;
- task records and submissions;
- training records and signup statuses;
- child names and class names shown as examples;
- completion percentages, badges, counts, dates, and status labels;
- sample images, comments, evaluations, reports, and progress rows.

Production rules:

```text
business_seed = NONE
dynamic_list_without_data = []
dynamic_count_without_data = 0
unassigned_or_unstarted_status = not_started
hardcoded_business_id = FORBIDDEN
```

Base identity data is the exception. Real schools, accounts, class rosters, parent-child relationships, and admin permissions may be provisioned before first use.

If a real roster exists but no business record exists, show the real person with `not_started` only when the page genuinely requires a roster row. Never copy a Mock completion state.


## 9. Required Spec Structure

Use concise English filenames: `<page>-spec.md`.

Each core-page spec should contain, when applicable:

```text
<PAGE>_BACKEND_OBJECT_SPEC

scope
source_page
static_node_count
dynamic_*_count
runtime_clickable_node_count
field_format
id_rule
null_rule
list_rule

[SHARED_OBJECT_RULE]
[CONTEXT_RULE]
[DATA_INITIALIZATION_RULE]
[STATIC_BUTTON_NODE_INDEX]
[DYNAMIC_CONTENT_NODE]
[PAGE_OBJECT]
[CONTENT_OBJECTS] or [REUSED_OBJECT_USAGE]
[EMPTY_STATE]
[NAV_OBJECTS] or shared navigation reference
[JUMP_VALIDATION]
```

Cardinality meanings:

```text
0:1 = optional single value
1:1 = exactly one value
0:k = zero or more values
1:k = one or more values
```

Entity primary IDs are normally `1:1`. Aggregate lists may use `0:k`.


## 10. Relationship and Validation Format

Example:

```text
rel_count (关系数量) = 3
rel_db (关联表) = db_school, db_child, db_file
rel_map (关系字段) = db_example{school_id}<->db_school{school_id}; db_example{child_id}<->db_child{child_id}; db_example{file_id}<->db_file{file_id}
```

Required validation:

```text
IF context object NOT_AUTHORIZED, return 403
IF dynamic object_id NOT_FOUND, return 404
IF query enum invalid, return 400
IF object status is draft|pending|rejected|deleted and viewer lacks permission, return 403
```


## 11. Main Agent and Sub-Agent Governance

The main agent must remain responsible for:

- the canonical object registry;
- final `db_*`, field, enum, and status names;
- cross-app reuse decisions;
- relationship counts and maps;
- navigation namespace consistency;
- Mock-versus-production classification;
- final validation across every generated spec.

Sub-agents may:

- inspect assigned HTML pages;
- list visible nodes, fields, routes, and Mock content;
- draft page-local sections using names supplied by the main agent.

Sub-agents must not independently create or rename canonical shared objects. Any proposed new object must return to the main agent for collision checking and approval before use.


## 12. Completion Checks

Before reporting completion, verify:

- [ ] Only the user-specified core HTML pages received `*-spec.md` files.
- [ ] Every shared entity reuses an existing canonical `db_*` name.
- [ ] Parent/Admin page aggregates use app-specific names.
- [ ] Navigation objects use the correct app namespace.
- [ ] Every Mock record is excluded from production initialization.
- [ ] Static UI content is distinguished from dynamic business content.
- [ ] Every declared node count matches its table rows.
- [ ] Every `rel_count` matches `rel_db`.
- [ ] Every relationship field appears in `rel_map`.
- [ ] Every dynamic jump uses a runtime ID from a query result.
- [ ] Empty states exist for dynamic sections.
- [ ] Context IDs are backend-derived and authorization-checked.
- [ ] A final cross-app reuse report lists reused and newly introduced objects.


## 13. Copy-Paste Prompt for a New Conversation

```text
You are continuing a backend object-specification project for a kindergarten platform with three connected apps: Teacher, Parent, and Admin. The apps have different pages and user roles but belong to the same kindergarten and use the same database.

Your task is to inspect only the core HTML pages that I provide for the current app and create one concise English `<page>-spec.md` file for each core page. Do not create specs for every linked subpage unless I explicitly request that.

Before acting, completely read the attached `spec-handoff.md` and all available Teacher App reference specs. Treat `spec-handoff.md` as the governing cross-app naming and data rule. The existing canonical `db_*` names must be reused exactly. A shared real-world business record must never receive a separate Parent-App or Admin-App table merely because another UI displays it.

Keep one main-agent process responsible for the canonical object registry and all final naming decisions. You may use sub-agents to inspect pages in parallel, but they may only extract visible nodes, fields, routes, and Mock content or draft sections using names assigned by the main agent. Sub-agents must not independently create or rename canonical objects.

Critical production rule: all sample records visible in the HTML are Mock data. This includes documents, cases, resources, tasks, submissions, training records, child names, class names, dates, images, progress percentages, badges, counts, and completion statuses. Production business tables start empty. Keep only static navigation, labels, instructions, categories, and empty-state copy. Real identity/roster/permission data may be provisioned by an authorized administrator.

Use database-generated IDs; never hard-code sample IDs. Raw identity IDs must be derived from the authenticated context, hidden from direct editing, and validated by the backend. For Parent App pages, authorize `parent_id` through `db_parent_child` before using `child_id`. For Admin App pages, authorize `admin_id` and `school_id` through `db_admin_school` and permission scope.

First perform these steps:
1. Inventory the supplied core HTML pages and identify static navigation, dynamic business content, forms, filters, jumps, and Mock elements.
2. Build an internal reuse map from every page concept to the canonical object registry in `spec-handoff.md`.
3. Propose a new canonical object only when no existing object represents the same business entity. Use reserved identity names when applicable.
4. Generate the requested `*-spec.md` files in the same style as the Teacher App specs.
5. Validate node counts, navigation routes, object names, field names, enum reuse, `rel_count`, `rel_db`, `rel_map`, runtime IDs, context authorization, production-empty rules, and empty states.

Every spec should include the applicable sections from `spec-handoff.md`, especially `[SHARED_OBJECT_RULE]`, `[CONTEXT_RULE]`, `[DATA_INITIALIZATION_RULE]`, node indexes, `[PAGE_OBJECT]`, reused/new content objects, `[EMPTY_STATE]`, and `[JUMP_VALIDATION]`.

At completion, provide:
- links to all generated spec files;
- a list of canonical objects reused from the Teacher App;
- a list of genuinely new objects introduced for this app;
- any unresolved naming or permission questions;
- validation results for node counts, relationships, navigation, and Mock-data exclusion.

Current app: [PARENT or ADMIN]
Core HTML pages to inspect:
[PASTE OR ATTACH THE CORE HTML FILES HERE]
Expected spec filenames, if already decided:
[LIST FILENAMES HERE]
```
