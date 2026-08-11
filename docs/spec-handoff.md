# Cross-App Spec Handoff

## 1. Purpose

This file transfers the specification rules from the Teacher App to the Parent App and Admin App.

All three apps belong to the same kindergarten, communicate with each other, and use the same database. UI pages and page aggregates may be app-specific, but the same business record must always reuse the same canonical `db_*` object.

Teacher App reference specs (filenames carry a numeric prefix in the Teacher repo):

- `01 home-spec.md`
- `02 school-affairs-spec.md`
- `03 comprehensive-coordination-spec.md`
- `04 training-center-spec.md`
- `05 home-school-spec.md`

Parent App specs: `home-spec.md`, `child-profile-spec.md`, `growth-book-spec.md`, `kindergarten-moments-spec.md`, `parent-tasks-spec.md`.

Admin App specs: the nine files under `docs/backend spec files/`.

The backend repository is the authority for landed DDL (`db/01_schema.sql`, machine-readable in `db/spec/tables.tsv`) and for decisions (`DECISIONS.md`, `db/GAPS.md`). **When this registry and the backend disagree, the backend wins and this file must be corrected.**


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

> **The registry does not end here.** This section covers only what the Teacher App handed over. Objects introduced later by the Parent/Admin apps are in section 5, and objects that are decided but whose DDL has not landed are in section 6. Search all three before proposing anything new.

### Identity and access context

| Canonical object | Meaning | Defined in |
|---|---|---|
| `db_school` | Kindergarten/school | `01 home-spec.md` |
| `db_teacher` | Teacher identity | `01 home-spec.md` |
| `db_class` | Class identity | `01 home-spec.md` |
| `db_teacher_class` | Authorized teacher-class relationship | `01 home-spec.md` |
| `db_child` | Child identity and class membership | `05 home-school-spec.md` |
| `db_file` | Shared uploaded file/media metadata | `01 home-spec.md` |
| `db_file_ref` | File-to-owner reference row | `01 home-spec.md` |
| `db_notification` | Cross-app notification | backend `db/01_schema.sql` |

### Tasks and submissions

| Canonical object | Meaning | Defined in |
|---|---|---|
| `db_task` | Internal teacher work task | `01 home-spec.md` |
| `db_task_assign` | Teacher work-task assignment | `01 home-spec.md` |
| `db_parent_task` | Parent-child task issued by teacher | `05 home-school-spec.md` |
| `db_parent_task_submission` | Parent/child submission for a parent task | `05 home-school-spec.md` |

`db_task` and `db_parent_task` are different business entities and must not be merged.

### Teaching content and training

| Canonical object | Meaning | Defined in |
|---|---|---|
| `db_upload` | Upload workflow record | `01 home-spec.md` |
| `db_resource` | Course resource | `01 home-spec.md` |
| `db_case` | Course case | `01 home-spec.md` |
| `db_resource_case` | Resource-case many-to-many relation | `01 home-spec.md` |
| `db_training` | Teaching research/training event | `01 home-spec.md` |
| `db_training_recommendation` | Training-center recommendation placement | `04 training-center-spec.md` |
| `db_home_case` | Teacher-home featured case placement | `01 home-spec.md` |

### Assessment, growth, and home-school collaboration

| Canonical object | Meaning | Defined in |
|---|---|---|
| `db_assessment` | Quality assessment | `01 home-spec.md` |
| `db_assessment_item` | Assessment item and score | `01 home-spec.md` |
| `db_moment` | Kindergarten moment/activity | `01 home-spec.md` |
| `db_moment_upload` | Per-child moment upload/progress | `01 home-spec.md` |
| `db_month_eval` | Teacher monthly child evaluation | `01 home-spec.md` |
| `db_month_eval_moment` | Monthly-evaluation ↔ moment relation | `01 home-spec.md` |
| `db_term_eval` | Teacher end-of-term child evaluation | `01 home-spec.md` |
| `db_child_assessment` | Child comprehensive assessment (five domains) | `05 home-school-spec.md` |
| `db_parent_evaluation` | Parent-side evaluation | `05 home-school-spec.md` |
| `db_growth_record` | Child growth-record aggregate | `05 home-school-spec.md` |
| `db_growth_book` | Child growth book | `05 home-school-spec.md` |
| `db_home_school_progress` | Non-persistent home-school progress view | `05 home-school-spec.md` |
| ~~`db_community_submission`~~ | **DEPRECATED** — see below | `05 home-school-spec.md` |

`db_community_submission` is **deprecated by `DECISIONS.md` B11 (2026-07-30)**: community coeducation is a feed view over `db_parent_task` + `db_parent_task_submission`, not its own entity. The row still exists in the backend DDL because B11 is 已定/待實作. **Do not reference it in new specs, and do not resurrect it under another name.** The entry is kept here rather than deleted so the decision stays discoverable.

### Party affairs and coordination

| Canonical object | Meaning | Defined in |
|---|---|---|
| `db_party_study` | Party-study document/content | `02 school-affairs-spec.md` |
| `db_party_activity` | Party activity | `02 school-affairs-spec.md` |
| `db_party_brand` | Party brand-building content | `02 school-affairs-spec.md` |
| `db_party_feature` | Party-page featured/banner placement | `02 school-affairs-spec.md` |
| `db_coord_document` | Coordination document with category enum | `03 comprehensive-coordination-spec.md` |

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

| Reserved object | Intended meaning | Defined in |
|---|---|---|
| `db_parent` | Parent/guardian identity | Parent App `home-spec.md` |
| `db_parent_child` | Authorized parent-child relationship | Parent App `home-spec.md` |
| `db_admin` | Admin identity | Admin App `dashboard-spec.md` |
| `db_admin_school` | Authorized admin-school relationship and role scope | Admin App `dashboard-spec.md` |

All four are now defined; treat them as REUSE, not as names still awaiting a first definition.

Before creating another identity object, check whether one of these names represents the same entity.


## 5. Objects Introduced by the Parent and Admin Apps

Section 3 lists what the Teacher App handed over. These were introduced afterwards by a downstream app and are equally canonical — **check this section too before proposing a new object.**

| Canonical object | Meaning | Defined in |
|---|---|---|
| `db_teacher_profile` | Teacher professional profile (job role, department) | Admin `review-spec.md` |
| `db_teacher_credential` | Teacher certificate/credential attachment | Admin `review-spec.md` |
| `db_teacher_profile_change` | Pending change request against a teacher profile | Admin `review-spec.md` |
| `db_training_feedback` | Teacher feedback on a training event, public after review | Admin `review-spec.md` |
| `db_review_action` | Review decision audit row — insert-only, same transaction as the decision | Admin `review-spec.md` |
| `db_content_metric` | Non-persistent view/download metric aggregate | Admin `library-spec.md` |
| `db_school_book_section` | School-level growth-book section, filled by admin and pushed read-only to teachers | Admin `growth-book-setting-spec.md` |

`db_review_action` carries a security rule, not just a shape: the review decision, the target-table status update and the `db_review_action` insert must happen **in one transaction**, and the row is never updated or deleted (backend `CLAUDE.md` red line 2).

`db_school_book_section` is **not** the same object as the class-level `db_growth_book_section`; the two differ in owner, lifecycle, collection behaviour and layout source. See `school_section_scope_rule` in `growth-book-setting-spec.md` before attempting to merge them.


## 6. Decided but Not Yet Landed

These objects are settled in the backend `DECISIONS.md` but their DDL has not been written. They are **already canonical for naming purposes** — do not invent a synonym because `tables.tsv` does not list them yet.

| Object | Meaning | Decision | Spec |
|---|---|---|---|
| `db_teacher_message` | Teacher message to a child, 300 chars, no attachments | E1 | `05 home-school-spec.md` |
| `db_scale_item` | Assessment scale item bank, versioned | E2 | `05 home-school-spec.md` |
| `db_child_assessment_item` | Per-item score, 124 rows per assessment | E2 | `05 home-school-spec.md` |
| `db_growth_book_template` | Growth-book template, class-level only | E3 / W13 / W19 | `05 home-school-spec.md` |
| `db_growth_book_section` | Teacher-added growth-book section (class-level) | E3 / W13 | `05 home-school-spec.md` |
| `db_book_widget` | Widget on a class-level section's grid | E3 / W5 | `05 home-school-spec.md` |
| `db_book_material_submission` | Parent (or teacher-proxied) submission for one widget slot | E3 / W15 | `05 home-school-spec.md` |
| `db_growth_material` | Class-level channel from moments/community into the growth book | E3 / W11 | `05 home-school-spec.md` |
| `db_school_book_section` | School-level growth-book section | Admin-side, pending backend review | `growth-book-setting-spec.md` |
| `db_school_term` | School-level term calendar and authoritative date window | F3 | `library-spec.md` |
| `db_content_check` | Ephemeral in-flight WeChat content checks; terminal batches are deleted | F12 | backend `DECISIONS.md` |
| `db_child_profile_correction` | Immutable family request to correct child name, birth date and gender | F13 | `review-spec.md` |

Extensions to existing tables that are decided but unlanded — notably `db_school.school_intro` (B12) and `db_school.book_cover` (W19) — are tracked in the owning spec's `[CANONICAL_FIELD_EXTENSION]`, not here.


## 7. App-Specific Naming

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


## 8. Context and Security Rules

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


## 9. Cross-App Reuse Examples

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


## 10. Mock and Production Empty-State Rules

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


## 11. Required Spec Structure

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


## 12. Relationship and Validation Format

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


## 13. Main Agent and Sub-Agent Governance

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


## 14. Completion Checks

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


## 15. Copy-Paste Prompt for a New Conversation

```text
You are continuing a backend object-specification project for a kindergarten platform with three connected apps: Teacher, Parent, and Admin. The apps have different pages and user roles but belong to the same kindergarten and use the same database.

Your task is to inspect only the core HTML pages that I provide for the current app and create one concise English `<page>-spec.md` file for each core page. Do not create specs for every linked subpage unless I explicitly request that.

Before acting, completely read the attached `spec-handoff.md` and all available Teacher App reference specs. Treat `spec-handoff.md` as the governing cross-app naming and data rule. The existing canonical `db_*` names must be reused exactly. A shared real-world business record must never receive a separate Parent-App or Admin-App table merely because another UI displays it.

Keep one main-agent process responsible for the canonical object registry and all final naming decisions. You may use sub-agents to inspect pages in parallel, but they may only extract visible nodes, fields, routes, and Mock content or draft sections using names assigned by the main agent. Sub-agents must not independently create or rename canonical objects.

Critical production rule: all sample records visible in the HTML are Mock data. This includes documents, cases, resources, tasks, submissions, training records, child names, class names, dates, images, progress percentages, badges, counts, and completion statuses. Production business tables start empty. Keep only static navigation, labels, instructions, categories, and empty-state copy. Real identity/roster/permission data may be provisioned by an authorized administrator.

Use database-generated IDs; never hard-code sample IDs. Raw identity IDs must be derived from the authenticated context, hidden from direct editing, and validated by the backend. For Parent App pages, authorize `parent_id` through `db_parent_child` before using `child_id`. For Admin App pages, authorize `admin_id` and `school_id` through `db_admin_school` and permission scope.

First perform these steps:
1. Inventory the supplied core HTML pages and identify static navigation, dynamic business content, forms, filters, jumps, and Mock elements.
2. Build an internal reuse map from every page concept to the canonical object registry in `spec-handoff.md`. The registry spans **three sections, not one**: section 3 (handed over by the Teacher App), section 5 (introduced later by the Parent/Admin apps), and section 6 (decided but DDL not yet landed). An object absent from the backend `tables.tsv` may still be canonical — check section 6 before concluding it does not exist.
3. Propose a new canonical object only when no existing object in any of those three sections represents the same business entity. Use reserved identity names when applicable. Never reference an object marked DEPRECATED.
4. Generate the requested `*-spec.md` files in the same style as the Teacher App specs.
5. Validate node counts, navigation routes, object names, field names, enum reuse, `rel_count`, `rel_db`, `rel_map`, runtime IDs, context authorization, production-empty rules, and empty states.

Every spec should include the applicable sections from `spec-handoff.md`, especially `[SHARED_OBJECT_RULE]`, `[CONTEXT_RULE]`, `[DATA_INITIALIZATION_RULE]`, node indexes, `[PAGE_OBJECT]`, reused/new content objects, `[EMPTY_STATE]`, and `[JUMP_VALIDATION]`.

At completion, provide:
- links to all generated spec files;
- a list of canonical objects reused, split by which registry section they came from (3, 5, or 6);
- a list of genuinely new objects introduced for this app, which must then be added to section 5 of `spec-handoff.md`;
- any unresolved naming or permission questions;
- validation results for node counts, relationships, navigation, and Mock-data exclusion.

Current app: [PARENT or ADMIN]
Core HTML pages to inspect:
[PASTE OR ATTACH THE CORE HTML FILES HERE]
Expected spec filenames, if already decided:
[LIST FILENAMES HERE]
```
