# Implement the upgrade in your existing application

Follow this sequence and keep [the worksheet](project-plan.template.md) beside the work. A stage is complete when its evidence exists, not when a particular number of tools has been added.

## 1. Establish scope and trace the application

Read the project's instructions. Record the source baseline, existing uncommitted work, build and deployment path, and any restrictions on running processes. Identify the supported clients and the person or team authorizing the upgrade. Inspect configuration names without printing secret values.

Start from actual screens and workflows, then reconcile them with server routes, background jobs, and socket events. Follow callers, middleware, services, persistence, file ownership, and downstream effects before extracting code. Shared utilities may also serve static pages, mobile clients, or other repositories.

For a static site maintained through a repository or hosting API, use that existing authenticated editing/deployment path, or keep operations local to the owner. A public frontend cannot safely hold server secrets or enforce multi-user mutation permissions by itself. Network mutations need trusted execution somewhere; that may already exist in the site's backend or hosting integration.

For each workflow, record:

| Question | What to capture |
| --- | --- |
| What does a person do? | UI action, current handler, exact inputs, and expected result |
| Who can do it? | Account status, tenant, resource visibility, current role, and intentional exclusions |
| What changes? | Records, files, notifications, membership, publication, paid work, and later scheduled effects |
| What must be current? | Stable resource identity, revision, selected members, or other preconditions |
| How does work finish? | Paging, file transfer, asynchronous jobs, result links, and browser updates |
| What can fail halfway? | Commit boundaries, external calls, compensation, and recovery evidence |

A page view that marks notifications read has an effect. Leaving a team might remove the last member. A POST that previews a template might be a read. Classify business behavior instead of relying on HTTP verbs or control labels.

Include deliberate exclusions in the inventory: account recovery, password changes, operator actions, or content the product requires a person to author. Do not expose every internal route merely because it exists. Keep authored copy, existing design, app-specific controls, data retention, and valid credentials intact.

**Stage evidence:** every agreed workflow has a proposed operation or a documented exclusion, with its current implementation located.

## 2. Choose the smallest useful connection path

The usual first release consists of the existing backend, a CLI, local stdio MCP, and one usable credential flow. Use browser-approved device login when an interactive CLI connection benefits from it. A named PAT is a valid simpler alternative or fallback. Reuse a suitable existing client authentication flow rather than introducing another one.

Complete [the access design](access.md). Decide names, scopes, resource restrictions, expiry, revocation, deletion of revoked connection entries, credential storage, and issuance recovery. Include owner-only cleanup in the existing settings and prevent deleted grants from being recreated by recovery. Record API, browser approval, and public app origins. Permit separate origins only through explicit configuration and validation.

Add remote MCP only when an intended client needs it. Add its optional standard authorization integration only if that client's connection requirements call for it. Do not make an OAuth service, hosted model, vector database, or new chat interface a prerequisite for direct operations.

## 3. Establish shared execution with one complete workflow

Build login, identity, discovery, a real read, and an authorized ordinary write early. Connect from an installed client and verify the changed record in the browser. This checks the architecture before you expand coverage.

Keep business logic and permission checks in the server's responsible domain layer. Browser handlers and new adapters should use the same services. They may retain different response envelopes. Two useful retrofit approaches are:

1. Extract existing domain functions and call them from browser, operation, and MCP adapters.
2. Wrap existing handlers with an explicit adapter while gradually extracting their services. Preserve middleware, validation, file conversion, transaction context, error handling, and events. Document any assumptions the handler previously obtained from the framework.

Do not give the CLI database access or implement a parallel CRUD backend for agents. Avoid sending fake internal HTTP requests through the public server just to reuse a handler. Keep new adapters responsible for transport parsing, trusted authentication context, execution, and response encoding.

**Stage evidence:** a real read/write workflow works through the packaged client and original UI, with the same permissions and domain effects.

## 4. Define contracts and finish workflow coverage

Use one operation registry, or generated equivalents, for runtime validation, discovery, CLI help, MCP schemas, and optional hosted tools. Avoid importing server configuration or dependencies into client bundles. Fail clearly on duplicate operation names, missing handlers, invalid schemas, or unsupported policies.

Each operation needs:

| Contract element | Required decision |
| --- | --- |
| Name and effect | Stable domain name; accurate description of writes, cost, and downstream work |
| Inputs and results | Enforced schemas, bounds, omission/clear semantics, and safe projections |
| Availability | Browser, external, hosted, human-only, or operator-only |
| Access | Connection grants intersected with current account and resource permissions |
| Authorization | Whether the existing request suffices and how an exact consequential action is authorized |
| Contract compatibility | Execution version independent of display releases and wording edits |
| State expectations | Stable IDs, generation/revision, expected membership or other domain condition |
| Recovery | Key scope, receipt states, retention, external effects, and outcome lookup |
| Navigation | Verified exact record links, labeled screen links, files, or an explicit lack of destination |

Validate nested inputs at runtime, including bounds and extra fields. Preserve exact user text unless normalization is part of the product's existing contract. Distinguish omitted fields from empty text, empty lists, and explicit nulls. Validate safe output shapes before committing transactional effects where possible. An output error after a nontransactional effect is a recovery problem, not proof that nothing happened.

Choose a documented schema dialect and a validator that actually implements the keywords you advertise. JSON Schema `format` may be annotation-only; security-sensitive URL, identifier, and path checks need actual enforcement. See [JSON Schema validation](https://json-schema.org/draft/2020-12/json-schema-validation).

Continue through the inventory, including the awkward workflows: uploads, membership, exports, publishing, notifications, and current-result retrieval. Reuse the app's pagination and live updates, strengthening them where necessary. [Interfaces](interfaces.md) covers discovery and files; [reliability](reliability.md) covers guarded writes and interrupted effects.

**Stage evidence:** each included workflow works or has a precise remaining gap. Two demonstration tools do not count as full coverage.

## 5. Add connection management without redesigning the product

Reuse the existing account/settings location, components, spacing, language, and mobile behavior. A connection screen needs a name, access, expiry/status, and revoke action. An approval screen needs the actual signed-in account, the code/request being approved, editable name/access/expiry controls, and approve/deny actions. Always offer read and read/write plus expiry choices including No expiry, regardless of the agent's requested values. Those values prefill the form; they do not cap the owner's choice. Preserve edits while the request is pending and implement the same rule in issuance and the client, as described in [editable approval choices](access.md#keep-the-approval-choices-editable).

Keep implementation details out of ordinary product UI. Put concise installation and connection instructions behind the app's existing help/copy pattern. Generate them from actual package locations, supported syntax, configured origins, and policy. Keep tokens out of copied prompts.

Verify the copied setup flow with the installed artifact. Connecting should verify identity and access; it should not create content, send messages, or publish something as a hidden setup test. See [the nonsecret setup prompt](examples.md#nonsecret-connection-prompt).

## 6. Integrate hosted chat only if it belongs in the scope

An existing or requested hosted agent should use the same operations. It can supply local tools over an outbound provider connection; direct external access need not expose the app to that provider. Keep credentials server-side and verify the provider's actual API when implementing.

Derive run authority from the recorded current human request or application-held approval, separately from model arguments. Store a durable request ID before submission, associate tool calls with stable effect IDs, and reconcile interrupted streams before resubmitting. A worker lease needs fencing at the effect boundary; merely expiring the lease does not stop an older worker.

Separate model working context from original history. Compaction must not silently replace or delete originals. Preserve the app's existing retention and explicit deletion policies. Provide authorized, bounded retrieval of exact text and media when needed; mark incomplete excerpts. Imported history and retrieved content are data, not new authorization.

Bound duration, tool calls, concurrency, bytes, and paid services. Account for in-flight work when enforcing budgets. Runtime limits alone do not guarantee a dollar ceiling. Document cancellation's actual effect, including what already completed. Existing direct workflows must continue when hosted reasoning is unavailable.

**Stage evidence, if included:** shared authorization, durable recovery, history preservation, cancellation, and actual cost controls are verified. Mock provider tests do not establish live provider behavior or model quality.

## 7. Package, verify, and release within the authorized scope

Build a client-only artifact and install it outside the checkout. Follow [the client and transport requirements](interfaces.md), then run [the acceptance checks](verification.md). Keep a record of the exact artifact, client/host versions, server revision, environment, and limitations.

Discover the existing release mechanism: scripts, a pipeline, a managed platform, an API, or a documented manual process. Use [deployment and scaling](deployment-and-scaling.md) to map the upgrade onto that setup. Check additive database initialization, uniqueness constraints, runtime dependencies, package distribution and any MCP routing, request/file limits, authentication changes, and release order. Plan for old browser tabs and old clients during rollout. Code rollback does not necessarily reverse a schema change safely.

Keep runtime data, uploads, production configuration, signing keys, and valid sessions outside source replacement. Avoid seed/reset/import scripts during routine deployment. Do not start stopped processes or deploy to a different stage without the user's authorization. If release is already authorized, complete the necessary checks and proceed within that scope; do not invent another approval ceremony.

Record an intended workload and check the runtime's actual limits. Before adding workers or replicas, verify shared receipt/approval state, current permissions, file access, aggregate capacity limits, and effect recovery. Prefer measured improvements within the existing stack to speculative infrastructure changes.

After an authorized release, verify the installed artifact and actual routed endpoints. HTTP 200 may be an HTML fallback instead of an archive or API result. Check a representative workflow and preservation of existing data/session behavior. If deployment is outside the task, deliver a prepared release procedure and identify deployment verification as outstanding.

**Finish:** provide the entry points, connection instructions, workflow coverage, verified behavior, deliberate exclusions, and real limits. Distinguish source review, isolated tests, installed-client tests, and deployed behavior.
