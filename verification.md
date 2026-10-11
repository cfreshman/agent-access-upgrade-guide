# Verify the behavior you are shipping

Use the project's existing checks or a focused isolated harness. Do not introduce a broad test framework just to follow this guide. Respect restrictions on tests, server starts, and deployment, and record the resulting verification limits.

Prepare synthetic accounts, separately scoped credentials, and resources with recorded IDs. For a team app, include different roles and a removed member. For an owner-only app, two owners may suffice. Use an isolated environment or designated test resources. Cleanup must affect only those recorded resources and credentials, never historical user data.

Keep evidence compact and private where necessary. Use synthetic secrets for logging checks; never print real tokens to prove they exist.

## Gate 1: operation coverage and browser parity

| Check | Passing evidence |
| --- | --- |
| Workflow inventory | Every agreed workflow maps to an operation or justified exclusion |
| Shared behavior | Browser, CLI, MCP, and optional hosted calls reach the same domain implementation |
| Actual effects | Hidden writes, paid imports, live updates, notifications, and delayed work are classified accurately |
| Validation | Invalid nested input, excess size, unknown fields, and missing preconditions fail before effects |
| Safe results | Output omits secrets and unauthorized records; exact IDs/links resolve correctly |
| Ordinary edit | Authorized change appears through existing UI/live behavior with copy and controls preserved |
| Direct access | Previously model-free operations still work without a hosted model key |

Do not count a registry asserting its own contents as proof of behavior. Inspect the stored effect and the observable result. Include the domain's difficult workflow, such as file attachment, team role change, or publication.

## Gate 2: identity, grants, and current access

| Case | Expected result |
| --- | --- |
| Wrong account's ID, file, cursor, export, or receipt | No unauthorized data or effect |
| Read-only credential attempts a write | Rejected through every supported adapter and guessed operation name |
| Same token after membership/role/visibility change | Current policy enforced, including protected receipt payloads |
| Expired row still present | Credential and device expiry enforced independently of cleanup |
| Revoked credential, already-running MCP client | Next protected request rejected within the promised revocation bound |
| Owner deletes a revoked connection, then reloads settings | Entry absent; display metadata removed; browser and sibling connections preserved |
| Active or another owner's connection submitted for deletion | Rejected without changing its grant or disclosing another owner's record |
| Deleted connection used or deleted again | Credential remains rejected; repeated owner deletion safely succeeds |
| Delete races approval, polling, or interrupted issuance recovery | No revived credential, delivered bearer, or reappearing connection entry |
| Legacy API/socket path | Agent token cannot bypass its grant; browser sessions remain valid |
| Agent attempts credential escalation | Browser/account-management policy enforced |
| Local logout and uninstall | Only intended owned local state removed |
| Revocation races a commit/dispatch | Observed result matches the documented in-flight policy |

Test approval evidence at the claimed trust level. For a host-attested flag, verify exact action/state binding and document that the token holder can assert the flag. For app-held approval, test wrong account/connection, altered digest, expiry, concurrent consumption, and recovery by the same logical request. Check that negation, quoted content, old requests, and tool output cannot supply trusted hosted-run authorization.

## Gate 3: connection setup and recovery

For a device flow, exercise pending, slowdown, timeouts, denial, and expiry. Verify that only the short code/link are announced. Check wrong-origin approval, code guessing limits, and unexpected approval URLs. Two accounts racing to decide a request must not create two grants.

Verify the owner's editable choices through the actual form, server, and installed client:

| Requested → owner-approved | Expected result |
| --- | --- |
| Read for 1 day → read/write with no expiry | All choices available; final grant accepted and displayed by the CLI |
| Read/write for 30 days → read for 7 days | Narrower/shorter final grant honored |
| Read for 1 day → read for 90 days | Longer finite lifetime honored without clamping |
| No expiry → a finite lifetime | Owner's deadline persisted and enforced |
| Owner edits while request status refreshes | Controls stay editable while pending; polling/rerenders do not reset selections |
| No-expiry credential beyond the original requested/default lifetime | Still authenticates if otherwise authorized; no fallback expiry or cleanup deletion |
| No-expiry credential revoked or owner loses resource access | Revocation/current permissions still enforced |
| Expired device request with No expiry selected | Approval/delivery rejected; unlimited credential lifetime cannot extend the login deadline |

Interrupt after the decision, after credential creation, after delivery, and before local save. Recovery must preserve the same owner/grant and absolute expiry or explicit no-expiry value, avoid duplicate credentials, and never reactivate a revoked grant. A conflicting resubmitted choice cannot change a decision already committed. Verify any encrypted delivery cleanup or secret-derivation assumptions.

For a PAT, verify protected creation, one-time reveal, metadata-only listing, safe local input, expiry, and revocation. For all flows, a failed identity check must preserve the old working profile. Exercise concurrent profile updates and reject unsafe credential paths. Verify each supported platform's permissions rather than extrapolating POSIX behavior to Windows.

If remote OAuth was selected, separately test the intended host's real discovery, connection, wrong-audience/issuer rejection, redirect checks, and renewal behavior. A local relay test does not cover that path.

## Gate 4: concurrent and interrupted mutations

| Injected condition | Expected result |
| --- | --- |
| Concurrent identical requests | One logical effect; consistent recovery |
| Same key, reordered equivalent object fields | Same action under the documented normalization |
| Same key, changed content/target/version | Conflict; no second effect |
| Lost response after commit | Recover original completion, including after revision advance or deletion |
| Crash after claim, before effect | Documented recovery; no permanent unexplained lock |
| Crash after effect, before receipt | Reconciliation or explicit uncertainty; no blind rerun |
| External dispatch repeated or worker lease lost | Deduplication/fencing demonstrated, or its limit recorded |
| Handler error after partial work | Truthful effect status, not a guaranteed no-effect failure |
| Resource changes after read/review | Guarded write rejects stale state |
| Contract changes before new execution | Every write adapter rejects unsupported semantics |
| Deployment after successful execution | Authorized completed receipt remains recoverable without executing the new contract |
| Approval already consumed by this request | Completion recovery works; a different request cannot reuse approval |
| Receipt payload expires or cleanup runs | Unresolved work retained; absence never silently treated as proof of no effect |

Inspect effect counts and stored records, not just status codes. Simulated provider behavior verifies local recovery logic; it cannot prove an actual provider's deduplication guarantee. Do not trigger paid or public effects merely to test the pattern unless that verification is authorized.

## Gate 5: installed CLI, actual MCP host, and files

Install the produced archive into an isolated user-owned location. Run outside the checkout with independent private config and without access to source imports. Inspect its file list for unwanted server code, configuration, logs, and data.

Check help/version, identity, full paginated discovery, schemas, representative reads/writes, status recovery, permission errors, revocation, and logout. Capture stdout, stderr, and exit codes independently. Kill the client after dispatch to confirm its recovery key survives. Verify request/response size limits, timeout behavior, exact text/binary transfer, and refusal to overwrite an existing output file unexpectedly.

Use the intended MCP host and selected protocol revision. Test discovery, a read, an authorized write, a denied write, application errors, and lifecycle cleanup. For 2025 revisions, test initialization and negotiated behavior. For 2026-07-28, test per-request metadata and applicable header/body checks. If supporting both, exercise both paths and unsupported-version behavior.

MCP stdout must remain clean on startup, update checks, failures, and shutdown. Verify timeouts and interrupted calls do not leave orphan transports or processes. Test private resource access, real image/file content, and invalid Origins where applicable. A short cleanup check does not prove long-running memory stability.

Exercise upload staging, attachment failure, repeated upload completion, deletion of one shared reference, and unauthorized downloads where relevant. Test remote-import destination and size controls with isolated fixtures. An export check should compare actual content, not merely the existence of a file.

## Gate 6: the complete path and release

For each principal client path, connect from the packaged artifact, verify identity, discover capabilities, perform an ordinary authorized workflow, inspect the result in the browser, recover one repeated mutation, and revoke the test connection. Include the hardest applicable file, membership, or consequential workflow.

Check connection settings and approval UI at relevant desktop/mobile sizes against existing controls. Test wrong-account, expired, denied, broader/narrower access, and shorter/longer/no-expiry states. Source inspection does not establish that the copied setup instructions or visible flow work.

Before an authorized deployment, verify additive migrations/indexes, release order, download/API/MCP routing, upload limits, dependency/runtime requirements, and rollback. Afterward, inspect actual response types and artifact contents, existing sessions, persisted test records, uploads, and relevant live updates. Preserve production configuration and unrelated services. If deployment is outside scope, mark these deployed checks outstanding.

Follow [the actual deployment and scaling model](deployment-and-scaling.md), whether releases use a provider API, pipeline, script, or another established path. Record the representative workload, latency/errors, query work, resource use, and queue delay where applicable. Check request/runtime limits and aggregate budgets. If supporting multiple instances, test duplicate requests on different workers, worker replacement during an action, shared file access, and revocation propagation. Do not infer capacity or cross-instance correctness from a single-user check.

For included hosted chat, additionally verify durable request/tool IDs, shared execution policy, exact history/media retrieval after compaction, cancellation/fencing, and enforced runtime/cost bounds. Check that external access survives hosted-provider failure.

## Report evidence, not a blanket guarantee

Provide the actual entry points and install/connect instructions; inventory coverage and exclusions; artifact/server/host versions tested; representative successes and injected failures; and any remaining platform, provider, concurrency, deployment, or endurance limits. Record whether each claim comes from source review, isolated execution, installed-client testing, or deployed observation.

An incomplete essential gate remains incomplete. Continue the work that is authorized, and describe the specific missing evidence without claiming the whole upgrade is verified.
