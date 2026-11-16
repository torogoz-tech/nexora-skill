# Nexora MCP functional catalog

The names in this guide are the public tool names, without any prefix added by the client connector. A tool appearing in the catalog does not grant permission: always start with `nexora_permissions_get`, discover the current schema, and respect actor, role, grants, project, revision, state and workflow.

The local implementation separates the code-query MCP (`crates/nexora-mcp/src/tool_catalog.rs`) from the platform MCP (`crates/nexora-api/src/*_mcp_*.rs` and `crates/nexora-api/schemas/*_tools.json`). Use published snapshots when discussing code; MCP queries do not read the live filesystem.

## Identity, projects, repositories and code

- Identity/projects: `nexora_permissions_get`, `nexora_projects_list`, `nexora_project_get`, `nexora_project_create`, `nexora_project_change`.
- Repositories/policies: `nexora_repositories_list`, `nexora_repository_get`, `nexora_repository_register`, `nexora_repository_change`, `nexora_repository_development_policy_get`, `nexora_repository_development_policy_set`.
- Synchronization/analysis: `nexora_sync_repository`, `nexora_repository_job_get`, `nexora_repository_job_cancel`, `nexora_repository_analysis_profile_get`, `nexora_repository_analysis_issues`, `nexora_index_repository`.
- Snapshot queries: `nexora_repository_overview`, `nexora_find_symbol`, `nexora_find_references`, `nexora_get_dependencies`, `nexora_traverse_graph`, `nexora_search_code`, `nexora_get_source_context`.
- Pinned context/evaluation: `nexora_context_manifest_create`, `nexora_context_manifest_get`, `nexora_context_manifest_status`, `nexora_context_package_create`, `nexora_context_package_get`, `nexora_context_package_export`, `nexora_evaluate_repository`.

Literal search is case-sensitive; line numbers start at 1; limits and truncation must be respected; an empty result does not prove absence. Evaluation does not run tests or prove deployment.

## Knowledge, documents, evidence and traceability

- RAG/contributions: `nexora_knowledge_context_search`, `nexora_knowledge_contributions_list`, `nexora_knowledge_contribution_get`, `nexora_knowledge_contribute`.
- Local/portable catalog: `nexora_list_pending`, `nexora_list_knowledge`, `nexora_get_knowledge`, `nexora_contribute_knowledge`, `nexora_revise_knowledge`, `nexora_review_knowledge`. These operate on local snapshot facts/contributions and do not replace platform knowledge or human reviews.
- Organization knowledge: `nexora_organization_documents_list`, `nexora_organization_document_get`, `nexora_organization_document_write`.
- Reviews: `nexora_knowledge_review_request`, `nexora_knowledge_review_list`, `nexora_knowledge_review_get`, `nexora_products_list`.
- Project documents: `nexora_documents_list`, `nexora_document_create`, `nexora_document_change`, `nexora_document_revise`, `nexora_document_get`, `nexora_document_version_get`, `nexora_document_versions_list`, `nexora_document_history`, `nexora_document_publish`, `nexora_document_publications_list`, `nexora_document_export`, `nexora_document_approval_get`, `nexora_document_transition`.
- Links/evidence: `nexora_document_link_create`, `nexora_document_link_remove`, `nexora_document_links_list`, `nexora_task_documents_list`, `nexora_evidence_upload`, `nexora_evidence_get`, `nexora_evidence_list`.

Documents do not use the task workflow. Creating, revising, reviewing and publishing are separate actions. A pending RAG summary is not published content. Attachments and evidence use references returned by Nexora, never local paths, base64 in Markdown or invented links.

## Requirements and features

- `nexora_features_list`: read existing features, links, proposals and `catalog_revision`.
- `nexora_feature_map_propose`: propose the mapping of an exact requirement version to existing/new features, optionally with proposed tasks. The proposal remains pending until an authorized human decision.
- `nexora_feature_complete`: complete an existing feature with required authority; it does not approve a map or complete linked tasks.
- `nexora_task_knowledge_link`: link a task to an exact feature or requirement; it does not create features or approve maps.

Look for reuse before creating a feature. A new requirement version or catalog revision requires rereading and a new proposal. Never duplicate tasks to bypass review.

## Help-desk service requests

- Catalog/products: `nexora_request_types_list`, `nexora_request_products_list`. Request types are independent from task types and can be archived, styled with an icon/color, and configured with criticality-based SLA policies.
- Read/list: `nexora_request_list`, `nexora_request_get`. Users see only their own requests; supervisors and authorized internal roles may see broader request scope. Never infer IT access from help-desk visibility.
- Create: `nexora_request_create` requires an idempotency `request_id`, title, real `type_id` and Markdown body; optional `product_id` and criticality (`low`, `medium`, `high`, `critical`) are validated against real catalogs.
- Conversation/attachments: `nexora_request_comment`, `nexora_request_attachment_upload`, `nexora_request_attachment_get`. Bodies are Markdown; attachments are image/PDF/Office evidence and are read only after authorization is checked again.
- IT change/linking: `nexora_request_change`, `nexora_request_link`. Internal IT may change status/type/product/criticality with `expected_revision`; links target only `task` or `feature` and do not mutate the target.
- Reporting: `nexora_request_report` supports date range, product, type, criticality, requester, status, pagination and CSV export. Reports exclude conversation bodies and technical data; CSV cells are protected against formula injection.

A service request starts from something existing: support, incident or service. Its only states are `new`, `in_progress` and `completed`; it does not enter the IT kanban or workflow. Changes use revision control and idempotent receipts; retries must reuse the same request key and payload. Internal attention records first-attention time, completion records resolution time, and the request receives an SLA snapshot at creation/type-criticality change. SLA clocks may be continuous or business-calendar based, including workdays, timezone and approved holidays. If a request reveals a new need, preserve it and create a separate requirement, linking them only when authorized.

The service-desk security boundary is deliberate: users and supervisors operate only on request tools and do not receive projects, repositories, tasks or knowledge. Shared request conversations do not include technical notes. Technical links are projected only when the current internal actor can read the target. A request attachment is not permission to expose the linked task or repository.

## Service-desk lifecycle example

1. Call `nexora_permissions_get` and identify whether the actor is `user`, `supervisor` or an internal IT role.
2. Call `nexora_request_types_list` and, when needed, `nexora_request_products_list`; never invent IDs.
3. Check visible requests to avoid duplicates.
4. Create with a stable idempotency key and a body containing symptom, expected outcome, impact, safe evidence and requester timezone.
5. Read the created request with `nexora_request_get` and preserve its number, revision, status, criticality and SLA projection.
6. Add conversation with `nexora_request_comment`; attach sanitized evidence with `nexora_request_attachment_upload`.
7. For internal IT handling, use current `expected_revision` with `nexora_request_change`; do not close merely because a task or feature was linked.
8. Link the request to an existing task/feature only after checking target authorization with `nexora_request_link`.
9. Use `nexora_request_report` for operational metrics and distinguish request completion from requirement delivery or production deployment.

## Help-desk authoring examples

Good request:

```markdown
## Symptom
The existing password-reset email is not arriving for one account.

## Expected outcome
Determine whether generation, delivery provider or recipient filtering failed.

## Evidence
Occurrence time with timezone, request number and a screenshot with tokens/emails redacted.
```

Bad request: “Build password recovery.” That describes a new requirement, not support for an existing service.

Good derived requirement: “Add one-time password recovery links with expiration, replay protection and an approved notification channel.” Preserve the original request and create the requirement separately.

## Tasks, backlog, collaboration and releases

- Catalog/backlog: `nexora_task_types_list`, `nexora_task_type_create`, `nexora_task_type_change`, `nexora_backlogs_list`, `nexora_backlog_create`, `nexora_backlog_get`, `nexora_backlog_change`, `nexora_backlog_reorder`.
- Tasks: `nexora_task_create`, `nexora_task_change`, `nexora_task_get`, `nexora_tasks_list`, `nexora_task_pool_list`, `nexora_task_pool_options`, `nexora_task_history`, `nexora_task_move`, `nexora_task_general_complete`.
- Lifecycle: `nexora_task_take`, `nexora_task_pause`, `nexora_task_resume`, `nexora_task_transition`, `nexora_task_transition_retry`, `nexora_task_transition_runs`, `nexora_task_stages_list`, `nexora_task_approval_get`, `nexora_task_workflow_migrate`.
- Git/work sessions: `nexora_task_work_prepare`, `nexora_task_work_get`, `nexora_task_work_report`, `nexora_task_commits_list`, `nexora_task_commit_bind`, `nexora_task_commit_remove`.
- Dependencies/subtasks: `nexora_task_relations_get`, `nexora_task_dependency_create`, `nexora_task_dependency_resolve`, `nexora_task_dependency_remove`, `nexora_task_dependencies_list`, `nexora_task_subtask_create`, `nexora_task_subtask_link`, `nexora_task_subtask_remove`, `nexora_task_project_change`.
- Collaboration/work logs: `nexora_task_comment_add`, `nexora_task_comments_list`, `nexora_task_work_logs_list`, `nexora_task_work_log_add`, `nexora_task_work_log_change`, `nexora_task_work_log_remove`, `nexora_task_knowledge_link`.
- Files: `nexora_task_image_upload`, `nexora_task_attachment_upload`, `nexora_task_attachments_list`, `nexora_task_documents_list`.
- Templates: `nexora_task_templates_list`, `nexora_task_template_create`, `nexora_task_template_change`, `nexora_task_template_publish`, `nexora_task_template_apply`, `nexora_task_template_get`, `nexora_task_template_version_get`, `nexora_task_template_versions_list`.
- Releases: `nexora_releases_list`, `nexora_release_get`, `nexora_release_create`, `nexora_release_tasks_add`, `nexora_release_compose`, `nexora_release_composition_get`.

Assignment is not Take; Take is not completion; preparing/reporting a branch does not change state or prove push, merge or deployment. Subtasks must never be used to bypass blockers.

## Workflows, developers and approvals

- Workflows: `nexora_workflows_list`, `nexora_workflow_get`, `nexora_workflow_create`, `nexora_workflow_change`, `nexora_workflow_publish`, `nexora_workflow_version_get`, `nexora_workflow_versions_list`, `nexora_workflow_developers_list`.
- Assignments: `nexora_developer_workflows_list`, `nexora_developer_workflow_get`, `nexora_developer_workflow_assign`, `nexora_developer_tasks_list`.
- Approvals: `nexora_approval_request_create`, `nexora_approval_request_get`, `nexora_approval_requests_list`, `nexora_approval_request_cancel`.

Human approvals must not be simulated with an MCP key. A task transition uses transition approval; a document/knowledge change uses document review. Reuse `request_id` only with the same payload.

## Analysis profiles

`nexora_analysis_model_guide_get`, `nexora_analysis_profile_validate`, `nexora_analysis_profile_package`, `nexora_analysis_profiles_list`, `nexora_analysis_profile_get` and `nexora_analysis_profile_install` support declarative model guidance, validation, packaging and installation of immutable versions. Structural validation does not prove semantic coverage; test original cases and regressions first. The AI writes rules and Nexora validates/packages them.

## Git authorship and exposed administration tools

`nexora_git_authors_list` and `nexora_git_author_aliases_list` query verified authorship and aliases; never use the pusher or assignee as a substitute for commit authorship. `nexora_repository_change` and `nexora_repository_register` manage repository registration within current authority. `nexora_development_workflow_get` reads current development guidance. None of these tools grants Git authorization or permits identity forgery.

## Platform capabilities currently exposed through HTTP/API, not the MCP catalog

The code also implements product surfaces that must be taught without pretending they are MCP tools. The routes are registered in `crates/nexora-api/src/api_server.rs`; the handlers are the source of behavior and authorization.

- Labor calendar settings and calendar: `/api/v1/settings/labor-calendar` and `/api/v1/labor-calendar`; includes workday/calendar configuration, leave requests, holiday proposals and decisions, and approver assignment.
- Calendar activities: `/api/v1/calendar/activities`; list/create/read/change activities, record attendance and cancel activities.
- Personal work blocks: `/api/v1/calendar/work-blocks`; list/create/change/remove personal blocks used to represent planned work time.
- Overtime: `/api/v1/overtime`; list/create overtime requests, decide them, inspect planned evidence candidates, inspect request candidates and append evidence.
- Developer ratings: `/api/v1/members/{id}/ratings`, `/api/v1/me/weekly-rating` and `/api/v1/ratings`; read yearly/member ratings, create authorized ratings, read the actor's weekly view and browse the permitted directory. A background worker refreshes rating calculations.
- Rating feedback/evidence: `/api/v1/me/rating-feedback` and `/api/v1/members/{id}/rating-evidence`; retrieve/generate personal feedback and upload/read rating evidence subject to private access rules.

Do not call these routes through MCP unless the connected server explicitly exposes them. Do not infer that a manager, developer or supervisor can see all calendars, overtime, ratings or evidence: the handler and current authorization policy decide visibility. Ratings are not a substitute for task evidence, and overtime approval is not a substitute for recorded work logs.
