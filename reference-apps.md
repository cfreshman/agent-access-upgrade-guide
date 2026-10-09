# Reference apps and transferable patterns

These are architecture examples, not prerequisites or deployment certifications. The public source case was checked on **October 9, 2026**. Only verified public committed source is linked. The two shorter domain examples are explicitly illustrative; the guide does not depend on access to their implementation.

## New Drugs: a public operation layer across a broad application

The [New Drugs repository](https://github.com/cfreshman/newdrugs/tree/e7720bb805285fdd6942f7ce072ddd9f0ab0d7b4) is public. The links below use commit `e7720bb805285fdd6942f7ce072ddd9f0ab0d7b4`, whose files were retrieved for this review. Later commits, local development changes, and live deployments are outside those source claims.

| Inspected implementation | What it demonstrates | What to carry further in a new upgrade |
| --- | --- | --- |
| [Catalog](https://github.com/cfreshman/newdrugs/blob/e7720bb805285fdd6942f7ce072ddd9f0ab0d7b4/shared/catalog.ts) and [executor](https://github.com/cfreshman/newdrugs/blob/e7720bb805285fdd6942f7ce072ddd9f0ab0d7b4/server/operations.ts) | Validated named operations dispatch to domain services; write scope and product-specific restrictions are enforced | Reconcile your own workflows and enforce current resource permission on execution and result recovery |
| [Contract revisions](https://github.com/cfreshman/newdrugs/blob/e7720bb805285fdd6942f7ce072ddd9f0ab0d7b4/shared/operationContract.ts) | Execution semantics have a revision separate from descriptive wording | Require an accepted contract consistently across each write adapter and preserve old receipt recovery |
| [Website operations](https://github.com/cfreshman/newdrugs/blob/e7720bb805285fdd6942f7ce072ddd9f0ab0d7b4/server/websites.ts) | Draft edits/publication check revision; whole-site deletion also binds site identity, published revision, and checkpoint IDs | Identify equivalent domain state in your app, including replacement-record identity |
| [Device login](https://github.com/cfreshman/newdrugs/blob/e7720bb805285fdd6942f7ce072ddd9f0ab0d7b4/server/deviceLogin.ts) | Approval and credential creation use a transaction; polling checks expiry and credential status | Test response loss and local-save failure too; the derived polling secret remains credential-equivalent |
| [CLI profiles](https://github.com/cfreshman/newdrugs/blob/e7720bb805285fdd6942f7ce072ddd9f0ab0d7b4/cli/config.ts) and [device client](https://github.com/cfreshman/newdrugs/blob/e7720bb805285fdd6942f7ce072ddd9f0ab0d7b4/cli/deviceLogin.ts) | Origin validation, named profiles, permission checks, serialized atomic writes, and identity verification before save | Verify your platform's credential protection and account expectations; do not copy app-specific origin exceptions |
| [MCP adapter](https://github.com/cfreshman/newdrugs/blob/e7720bb805285fdd6942f7ce072ddd9f0ab0d7b4/server/mcp.ts) | Compact discovery/read/execute tools reuse the executor and expose file resources | Select and test the actual host, revision, annotations, schemas, and current-access behavior |
| [Client packaging](https://github.com/cfreshman/newdrugs/blob/e7720bb805285fdd6942f7ce072ddd9f0ab0d7b4/scripts/package-cli.mjs) | A constructed client package separates distribution from the source checkout | Inspect and independently install your artifact; native installers and self-update are optional |

Three limits matter when reusing the pattern:

1. The [HTTP route](https://github.com/cfreshman/newdrugs/blob/e7720bb805285fdd6942f7ce072ddd9f0ab0d7b4/server/app.ts) checks expected kind/version when supplied. The inspected MCP call schema does not carry that same version requirement. This is not universal contract enforcement.
2. The executor commits core database results with receipts, but staging, cleanup, and other effects need separate analysis. Its main completed-receipt branch returns before the domain handler runs again; account-level checks alone do not establish current permission for every embedded result. Apply the explicit recovery authorization in [reliability](reliability.md).
3. The MCP `confirmed` field is described as a host attestation. It is not independent proof of a human decision. The app's own product policies, such as human-authored profiles, are specific to that product.

The useful lesson is separation of responsibilities: shared domain operations, state-aware effects, private connections, and installable adapters. The language, database, runtime, social features, and hosting configuration are not part of the prescription.

## Minnow: a small website builder, illustrated

For a personal site builder, a small set of direct operations can be clearer than a large discovery system: read a draft, edit exact content, inspect assets, restore a checkpoint, publish, or unpublish. Existing and external agents can use those same operations while optional hosted chat supplies its own reasoning.

This is an **illustrative adaptation**, not a claim that a public Minnow source release or deployed client implements the guide. Keep the domain lesson: ordinary edits belong to the draft; publication has its own authorization and state binding. Preserve original text, images, and history according to product policy. A public preview may expose content before publication and needs its own disclosure rules.

For a new implementation, include expected draft revision on publication, recoverable issuance, and durable request identity. A small tool list does not eliminate those requirements. A model's quotation of a human sentence should not be treated as a general authorization mechanism.

## Sheet Social: an existing team application, illustrated

For an established team application, wrapping proven handlers can avoid a large rewrite. Inventory notes, reference documents, outputs, membership, exports, and account controls; then adapt only the agreed workflows into operations. Keep existing browser response shapes and live updates while centralizing current role checks.

This is also an **illustrative adaptation**, not a public source or deployment claim for an agent-access release. Its useful distinction is that broad workflow coverage can coexist with intentional exclusions: account controls, privileged reruns, or actions that a product deliberately does not offer need not become general agent capabilities.

A standalone database can support recoverable approval through an immutable decision and unique issuance ID; a new transaction service is not automatically necessary. When adapting legacy handlers, inspect their failure boundaries, query limits, and post-write effects. Do not assume an error means no effect, or that slicing a completed result bounds the query's work.

## Using references responsibly

Trace the affected implementation rather than transplanting whole applications. Public source demonstrates code at one revision, not a supported package, tested host matrix, or live security guarantee. Recheck availability and licensing before redistributing third-party implementation code. The synthetic examples in this guide are original illustrations of the recommended contract, and do not copy private application source.
