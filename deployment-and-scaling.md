# Fit the existing deployment and scale from evidence

Treat hosting and release mechanics as facts to discover in the target project. This guide does not assume SSH, a particular cloud, a container runtime, a process manager, a writable server filesystem, or a CI provider. The operation contracts and access rules should survive changes in those choices.

## Map the upgrade onto what already exists

| Existing setup | Where the upgrade can fit | What to investigate |
| --- | --- | --- |
| Static site maintained in a repository | Local CLI/MCP over the existing source and publishing workflow | File ownership, build output, publication authority, and whether a remote mutation API is actually needed |
| CMS, hosted application, or backend platform | Existing APIs, supported extensions, or server functions | Available permissions, rate limits, extension constraints, and supported deployment mechanism |
| Managed application platform | The existing application release and configuration system | Routing, secrets, durable storage, lifecycle, and rollback |
| Functions or edge runtime | Bounded operation handlers plus existing durable storage/work facilities | Runtime limits, request duration, concurrency, storage semantics, and transport support |
| Containers or an orchestrated service | Existing app/service image and rollout | Health/readiness, instance coordination, draining, shared state, and mixed-version behavior |
| VM, VPS, on-premises, or shared host | The current application and its supported release path | Permissions, dependency/runtime installation, durable data, capacity, and recovery |

These are adaptation options, not a migration sequence. A static site may need only local repository access. A managed backend may already provide authentication, transactions, storage, and jobs. A working VM deployment may need no new infrastructure at all.

Write a short release map: where code is built, how a specific revision becomes live, where configuration/secrets and durable data live, which origins/routes clients use, and how rollback works. Read the applicable scripts, pipeline, provider configuration, or documented manual procedure. Do not invent a universal deploy command.

Apply the same release checks through that mechanism: validate the artifact, initialize required constraints safely, preserve data/configuration, coordinate compatible client/server changes, check actual routed responses, and retain a recovery path. Deployment authority remains whatever the project/user has established. Adding agent operations does not give an agent permission to control the hosting account.

## A VM script is one example

For example, a project on a VM such as a [DigitalOcean Droplet](https://docs.digitalocean.com/products/droplets/) might use an authorized operator or coding agent to run a reviewed pull/build/activate script. That is one possible workflow, not a guide requirement or a statement about the reference apps' production setup.

A manual process becomes harder to manage as releases, environments, services, or collaborators multiply. The next improvement might be a reproducible build, an identified release artifact, serialized deployments, automated checks, or an existing CI/CD pipeline invoking the same release procedure. A managed platform's release API may already provide those pieces. For example, [GitHub Actions environments](https://docs.github.com/en/actions/reference/workflows-and-actions/deployments-and-environments) offer one platform-specific way to associate deployment controls and secrets with environments; other systems have their own mechanisms.

Release automation and serving capacity are separate concerns. Automating deployment does not fix a slow query, and adding servers does not fix an unreliable release script. A VM is not inherently limited to manual deployment. Improve the actual bottleneck rather than changing hosting providers just because an agent interface is being added.

## Account for agent workloads

Agents may issue many requests quickly, keep paginating, retry failures together, or run several jobs per account. Measure the workload in operations, concurrency, bytes, query work, and external cost, not just user count.

Start with a representative workload covering small reads, large reads/exports, ordinary writes, and any expensive file or external-service operation. Observe latency, errors, database work/connections, CPU/memory, queue age, and storage/network use. Set explicit request and concurrency limits based on those measurements. Keep tokens and private content out of metrics and logs.

Enforce bounds where work happens: indexed queries, page sizes, export size/time limits, upload bytes, decoded media dimensions, and external-call concurrency. Reject or queue excess work with useful retry guidance. Apply per-account/connection fairness as well as aggregate capacity limits, so one agent cannot starve ordinary browser use. Bound and stagger retries to avoid synchronized retry storms.

Do not introduce a queue, cache, broker, or separate service merely to tick a scalability box. Use the existing stack until a concrete capacity or durability requirement calls for more.

## When requests can reach different instances

Credential grants, approvals, receipts, and resource revisions must use authoritative state visible to every instance that can handle the action. An in-memory lock or receipt map does not coordinate another process or survive replacement. Enforce uniqueness and guarded writes in the appropriate shared store. Durable single-instance storage can still be entirely appropriate while the app has that operating model.

Review these boundaries before increasing concurrency:

- **Permissions:** revocation and role changes must obey the promised propagation bound across authorization caches and subscriptions.
- **Work claims:** concurrent callers and retried jobs must share logical keys; expired leases need fencing and effect recovery.
- **Capacity:** connection pools, admission limits, quotas, and external-service budgets need an aggregate bound. More app instances can overload a smaller database or provider.
- **Files and events:** another instance must be able to retrieve committed attachments and deliver required updates. Local files are not shared merely because instances run the same image.
- **Rollout:** old and new workers may coexist; preserve accepted contracts and receipt recovery across that interval.

For multiple regions, decide where writes, grants, and receipts are authoritative and what consistency the storage actually provides. Do not assume asynchronously replicated state can enforce an immediate global revocation or unique effect. Keep strict invariants at an appropriate coordination boundary.

## Respect the runtime's execution model

Request deadlines, process shutdown, idle suspension, memory, body limits, and streaming support vary by platform and configuration. Do not hardcode limits copied from another app. Verify the target's current documentation and settings. Local stdio MCP can avoid a remote MCP endpoint, but its upstream application requests still face those limits.

If work cannot reliably finish within a request, use the app's existing durable job/workflow facilities and return an authorized status handle. Do not assume an unawaited task keeps running after the response or that a temporary filesystem preserves receipts. As concrete platform examples, [AWS Lambda guidance](https://docs.aws.amazon.com/lambda/latest/dg/best-practices.html) discusses execution reuse and duplicate-event handling, while [Cloud Run's runtime contract](https://docs.cloud.google.com/run/docs/container-contract) defines request and instance lifecycle constraints. Read the equivalent source for the actual target.

Verify the intended workload with its real deployment shape. Where multiple workers are supported, send identical retries to different workers, replace a worker mid-action, and exercise the slow/large workflow near configured limits. Report the tested workload and bottleneck. A successful single-user request is not a capacity result.
