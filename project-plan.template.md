# Agent access project worksheet

Copy and fill this from actual project source. Keep it as a short working plan and evidence record, not a chat transcript. Remove irrelevant rows. Use the repository's established documentation location and conventions.

## Scope and preservation

- Product and repository:
- Source baseline and existing unrelated changes:
- Applicable project instructions and process restrictions:
- Authorized workflows and intentional exclusions:
- Existing UI, authored copy, data, sessions, and behavior to preserve:
- Target users/clients and existing API or credential systems to reuse:
- Deployment stage and authorization; commit/push instructions:

## Existing system

- Browser authentication, account IDs, tenant/resource roles:
- API, approval, public, and development origins and allowed relationships:
- Persistence mode, transactions/conditional writes, indexes, initialization:
- Files, sockets, caches, notifications, schedules, external services:
- Runtime, packaging, build, deployment, and rollback path:
- Actual hosting/release mechanism and runtime limits, without assuming a provider:
- Intended workload, capacity budget, bottleneck, and instance/region model:
- Baseline verification and isolated fixtures available:

## Workflow inventory

Use one row per meaningful existing workflow. Locate browser and server code, including downstream effects. Each included row needs an operation and a way to verify it.

| Workflow | UI/handler/services | Operation or exclusion | Effects/cost | Grant + current resource permission | Authorization and state | Recovery strategy | Acceptance evidence |
| --- | --- | --- | --- | --- | --- | --- | --- |
| Identity/access | | | | | | | |
| List/read main resource | | | | | | | |
| Create/edit main resource | | | | | | | |
| Files/export if present | | | | | | | |
| Membership if present | | | | | | | |
| Consequential action if present | | | | | | | |
| Account/operator boundaries | | | | | | | |

## Decisions before implementation

Choose from the existing app and authorized scope. Record a tradeoff only when it affects a real unresolved choice; do not turn this list into repeated requests for approval.

| Decision | Selected approach and evidence |
| --- | --- |
| Shared operation/services location and legacy-route strategy | |
| Schema dialect, validator, output projection, text/clear semantics | |
| Contract revision and state-precondition enforcement on all adapters | |
| First login path: existing flow, device approval, or PAT | |
| Named grant, scope/resource restrictions, expiry, and management | |
| Issuance/delivery recovery and protected secret storage | |
| Local profiles, concurrent writes, platform protection | |
| Revocation cache bound and in-flight/dispatch guarantee | |
| Human authorization evidence and action/delegation binding | |
| Request key scope, durable client journal, normalization version | |
| Receipt recovery permissions, retention, and completed/uncertain behavior | |
| External effect IDs, deduplication, reconciliation, and fencing | |
| Paging, safe result links, files, exports, and preview visibility | |
| CLI runtime/package/allowlist and update/uninstall ownership | |
| Existing/requested hosted chat, history, cancellation, cost controls | |
| Existing settings components and generated setup guidance | |
| Shared state across workers, aggregate admission limits, and long-work handling | |

## Client compatibility

Local MCP is the normal starting point. Add rows only for clients you intend to support. Remote OAuth is optional and should have a named client requirement.

| Host/client and version | Local/remote transport | SDK and protocol revision(s) | Credential/login path | Features exercised | Evidence or gap |
| --- | --- | --- | --- | --- | --- |
| CLI | Operation API | | | | |
| Local MCP host | stdio | | Saved CLI profile | | |
| Optional remote host | | | | | |

## Execution stages

| Stage | Concrete deliverable | Done when | Evidence / remaining work |
| --- | --- | --- | --- |
| Inventory | Agreed workflows traced | Included operations/exclusions reconcile with the UI/routes | |
| First complete path | Login, identity, read, ordinary write | Installed client and browser show equivalent behavior | |
| Coverage | Remaining agreed workflows | Hardest file/membership/consequential workflow is included | |
| Recovery and access | Current permissions, guarded writes, receipts | Races, lost responses, and revoked access behave as promised | |
| Client release | Installable CLI and intended MCP host | Works outside checkout; no secret/config leakage | |
| Capacity/runtime fit | Representative bounded workload | Measured limits, worker recovery, and hosting constraints recorded | |
| Optional hosted work | Shared bounded execution | History, request recovery, cancellation, costs verified | |
| Authorized deployment | Routed artifacts and preserved state | Deployed checks pass, or explicitly remain outside scope | |

## Acceptance and finish

- Synthetic fixture IDs and narrowly scoped cleanup:
- [Verification gates](verification.md) selected, with results and untested cases:
- Exact built artifact/server revision and supported client platforms:
- Operation coverage and deliberate human/operator-only exclusions:
- Tested install/connect instructions and entry points:
- Current permission, approval-evidence, retry, and revocation guarantees:
- External effects and other meaningful limits:
- Deployment/rollback status and data/session preservation evidence:
- Task changes, unrelated work preserved, and Git publication status:
- Any unresolved decision or missing essential evidence:
