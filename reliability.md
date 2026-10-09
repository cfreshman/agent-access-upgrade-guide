# Revisions, retries, and effects

Two requests can look identical and mean different things. Conversely, a timeout can hide a successful change. Design recovery around a logical action and its durable evidence, rather than around one HTTP request.

## Keep four identifiers separate

| Identifier | Answers |
| --- | --- |
| Operation contract version | Does this call still mean what the client understood? |
| Resource identity and revision | Is this the same record and state that were selected? |
| Logical request / idempotency key | Is this another attempt at the same action? |
| Authorization binding | Does this action match the person's permission or approved delegation? |

The app release version is separate again. A prose edit should not invalidate a compatible contract. A semantic change can require a new contract even if the schema is unchanged. A new contract cannot silently reinterpret an old approval.

For new writes, require an accepted contract version or another explicit compatibility mechanism across CLI, HTTP, MCP, and hosted adapters. Checking an optional CLI header leaves other adapters unprotected. Keep reads appropriately compatible; not every read needs a version pin.

Compare resource expectations in the actual write, not only in an earlier read. Use a conditional update, a transaction with appropriate serialization, or the storage system's equivalent. Check the affected-row count. If using HTTP entity tags, `If-Match` uses strong comparison and normally reports a failed precondition with 412; a custom operation API can instead define a structured state-conflict result. See [HTTP conditional requests](https://www.rfc-editor.org/rfc/rfc9110.html#section-13.1.1).

Stable IDs must distinguish replacement records. A revision that restarts at 1 after deletion is insufficient if the identifier can be reused. Publication may need draft and destination revisions; deletion may need a generation ID and the exact attachment/checkpoint set; membership changes may need current role and selected member state. Choose the domain's actual invariant.

## Preserve a logical action before sending it

Generate a high-entropy request key for each new mutation. Persist it with the operation version and exact normalized request before sending, so a client crash does not lose the only recovery handle. Keep that journal private and bounded; it may contain sensitive content. The CLI should expose a nonsecret status/recovery command. MCP and hosted callers need the same continuity.

Reuse the key only for the identical logical action. A JSON-RPC request ID, HTTP connection ID, or newly generated key on each retry is not this mechanism. For a genuinely idempotent domain action, document which effects repeat: storing the same title twice might still send two notifications.

Use a unique storage constraint for the selected receipt scope, commonly tenant plus connection plus key, or tenant plus account plus key. State whether another connection can recover that action. Do not deduplicate across unrelated users.

Fingerprint the operation, execution semantics, normalized arguments, target expectations, and effect binding. Keep transport IDs and fresh proof signatures outside the action identity. Specify defaults, null/omitted distinctions, number handling, Unicode handling, and deterministic key ordering. Plain `JSON.stringify` insertion order is not a portable canonical format. A maintained implementation of [JCS, RFC 8785](https://www.rfc-editor.org/rfc/rfc8785.html) is one option for its supported JSON domain; it is an Informational RFC, not a requirement to replace your existing serializer.

Reject ambiguous encodings such as duplicate object keys when they could be interpreted differently by components. Preserve exact authored text; canonical serialization is not permission to trim, translate, or rewrite it. Store the fingerprint algorithm/version and normalized action so recovery survives later changes to defaults or serialization.

## A receipt is a state machine

Use the existing job or transaction machinery where it fits. The names below are illustrative:

| Receipt state | Meaning and permitted next step |
| --- | --- |
| `accepted` / `running` | The action has a unique durable claim; return status and bounded retry advice |
| `complete` | Return the recorded result after checking current access to it |
| `failed_no_effect` | A terminal failure known to precede all effects; replay the failure, or follow an explicitly documented resumable policy |
| `uncertain` | An effect may have occurred; reconcile against authoritative state before executing again |

Authenticate before looking up protected receipts. On an existing key, compare against its original action identity, using the recorded normalization rules. Reject changed content or targets. Authorize access to its status and result under current policy, including current visibility of any embedded private data.

**Recover a completed action before applying new-execution state checks.** A successful update has already advanced its revision; a successful deletion has removed its target. Requiring the old precondition again would make their lost responses unrecoverable. Similarly, a consumed approval should remain bound to the same request rather than requiring another approval to return success.

This is not permission to return private data after access was removed. Design separate authorization for receipt status and for its payload. After deletion, an owner may be entitled to a minimal completion record without reconstructing the deleted content. After leaving a team, the caller may lose access to the old result entirely. A revoked credential still fails authentication. Never rerun an operation as a substitute for a result you may no longer disclose.

Keep receipt recovery available across compatible deployments and, where needed, after an operation is retired. Resolve an existing receipt using its recorded contract; apply current-contract validation to **new execution**. Do not silently execute an unsupported old contract because a client supplied an old key. If full replay cannot be supported, provide authorized minimal status and a recovery path.

## Match the mechanism to the effects

| Mechanism | What it can establish | Remaining work |
| --- | --- | --- |
| Mutation and receipt in one transaction | Core database change and result commit together | External calls, files, event delivery, and transaction callback retries |
| Unique claim plus guarded legacy effect | Competing callers cannot freely start the same action | Crash recovery and proof of whether a nontransactional effect happened |
| Durable intent/outbox plus worker | Committed work is discoverable for later dispatch | Duplicate dispatch, stale workers, provider reconciliation, and final completion |

Never place an uncontrolled external call inside a transaction callback that the database may rerun. For messages, paid processing, webhooks, or other remote effects, persist a stable effect ID with the intent and use the downstream service's deduplication or status lookup. An outbox alone gives recoverable dispatch, not exactly-once delivery.

A lease timeout does not prove the previous worker stopped. Use fencing or an effect-specific conditional claim that rejects stale workers, and downstream deduplication where needed. If the provider offers neither deduplication nor authoritative lookup, document the remaining uncertain-outcome case instead of promising automatic safe replay.

If the database change succeeded but file cleanup or notification dispatch failed, record those separately. Return a truthful pending/partial status when appropriate. Do not report “nothing changed” because later work failed. Do not classify all 4xx errors as effect-free; a legacy handler may have already committed work before returning one.

## Executor ordering

This is pseudocode, not a transaction implementation. The atomic boundaries must match the selected storage system and the domain's authority rules.

```text
execute(envelope, credential):
  authenticate credential and current account
  validate bounded envelope and request-key syntax

  if a receipt exists in this caller's allowed recovery scope:
    match the original action using its recorded contract/normalization
    authorize current access to the receipt status and result
    return complete, running, known failure, or uncertain; do not execute again

  resolve accepted contract; validate and normalize domain input
  check source, scopes, current resource permissions, and action authorization

  at the domain's atomic claim/commit boundary:
    uniquely claim the request (a racing existing claim goes to recovery)
    coordinate current authority with concurrent revocation/role changes
    compare resource identity/revision inside the guarded mutation
    reserve any app-held approval for this exact logical request
    perform the existing domain mutation
    validate result; persist receipt and any durable external intent

  dispatch/reconcile external effects under their own stable effect IDs
  return committed result or truthful pending status and a recovery handle
```

For transactional database-only changes, the claim, mutation, approval reservation, and completed receipt can commit together. For a nontransactional handler, persist the claim before the effect and recover the intermediate states explicitly. Do not implement the pseudocode as “insert claim, call anything, mark complete” and assume the crash gap disappeared.

## Bound retries and retention

| Situation | Client behavior |
| --- | --- |
| Retryable read failure | Bounded backoff, respecting server retry advice and cancellation |
| Write response lost | Recover status or retry the same action/key under the server's documented policy |
| Key already used for different input | Stop; never replace the key to bypass the conflict |
| Current state changed | Read and resolve the conflict; a changed action needs a new key and matching authorization |
| New-execution contract rejected | Describe again and reassess; do not automatically endorse new consequences |
| External outcome uncertain | Reconcile; do not blindly resubmit with a fresh key |
| Permission revoked | Stop; neither execution nor protected replay may bypass it |

Declare the retry window and receipt retention separately from product history. Keep enough durable evidence through the longest supported client/job retry horizon. Unresolved work must not disappear when a short cleanup timer expires. If payloads expire, a smaller tombstone or durable effect ID can preserve deduplication without retaining all content.

After deduplication evidence expires, absence of a receipt does **not** prove the action never happened. Inspect authoritative state, or require deliberate recovery if the effect cannot be determined. Idempotency is a bounded application guarantee, not an unlimited property of a header name.
