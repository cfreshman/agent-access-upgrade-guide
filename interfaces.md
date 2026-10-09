# CLI, MCP, files, and distribution

Adapters should make the same application capabilities usable in different environments. Their authentication and wire formats differ; their business rules should not.

## Discovery and bounded reads

Provide identity, capability enumeration or search, operation descriptions, and execution. A small application can expose individual domain tools. A larger one may use a compact search/describe/read/execute surface. Choose what helps clients find the right action; a generic execute tool must still enforce every domain contract.

Separate capability search from app-content search and public web search. “Export documents” describes a capability; it is not automatically a content filter. Start with deterministic enumeration and keyword matching unless the product needs more. Discovery should not unexpectedly launch paid processing.

Descriptions should state effects, cost, current-access requirements, required state, retry rules, and exclusions. Return stable resource IDs and verified links. Label a general screen link as such; do not present it as an exact record link. A returned URL is not evidence that a browser was opened.

Paginate content and large catalogs with enforced bounds. Limit work at the data query, not only by slicing an unbounded result afterward. Use deterministic ordering with a unique tie-breaker. Bind cursors to the intended filters and caller/tenant context, validate them, and reauthorize every page. State whether results are a snapshot or a traversal of changing data. Even keyset pagination needs documented semantics under concurrent changes.

Cache descriptions by server/origin, connection context, and contract revision. Treat caches as advisory. A cache must neither authorize a stale write nor mix one account's private discovery with another's. Invalidate appropriately when profiles or permissions change.

## Ship a usable CLI

Provide unauthenticated help/version plus commands for login, identity, discovery, describe, read, execute, receipt/status recovery, and logout. Add files, exports, jobs, and profiles when the app's workflows need them. Convenience commands should be thin wrappers over the shared operation API.

The CLI should:

- Return stable machine-readable success data on stdout and errors on stderr, with nonzero exit status for failures. Keep interactive progress separate. Publish stable error codes and recovery advice, including whether an effect may have occurred.
- Apply request and overall timeouts, cancellation, and bounded paging. An `--all` option needs item/page/byte limits and must not turn a read into a batch mutation.
- Save each mutation's key and exact request before dispatch, with private bounded storage. Reuse them on a documented retry. Do not invent a new action after a timeout.
- Accept structured input files for large text and precise JSON. Preserve encoding, newlines, and omitted-versus-cleared fields. Use argument arrays for subprocesses; do not interpolate content into shell code.
- Pin credentials to the configured origin. Reject redirects on credential-bearing requests unless a deliberately implemented policy can prove the destination is allowed. Never send a bearer to a returned arbitrary file or approval URL.
- Verify the returned account and actual approved access before saving a new connection. Accept and report the owner's changed scope/lifetime, including broader access or no expiry; do not reject it merely because it differs from the request. Identify the selected profile without exposing its secret. Multiple invocations must not corrupt the profile store.

An error envelope might include `code`, a safe message, a request key, `outcome` (`not_started`, `pending`, `complete`, or `unknown`), and a permitted recovery action. Those fields are your application contract, not MCP or HTTP standards. A transport exception alone usually cannot classify a write as `not_started`.

## Local MCP is the normal first adapter

Let a local host launch the installed CLI's MCP subcommand and reuse its private profile. Host configuration then contains the executable and nonsecret profile name. Follow the intended host's actual config format; `mcpServers` JSON is common but not universal.

Use a maintained SDK for the chosen language and protocol revision where practical. A local adapter can translate MCP calls into the operation API without exposing a remote MCP endpoint. It does not need an OAuth server.

Keep stdout exclusively for protocol messages, including startup, update, error, and shutdown paths. Logs go to stderr. Bound input and upstream work, propagate cancellation appropriately, and close transports, listeners, and child processes. See [the stdio binding](https://modelcontextprotocol.io/specification/2026-07-28/basic/transports/stdio).

Tool annotations describe behavior; they do not enforce authorization. Do not label every wrapper read-only or idempotent regardless of what it invokes. For a generic execute tool, use conservative annotations and accurate operation descriptions. A retry key required in tool arguments can support identical-call recovery; a hidden new key generated on every call cannot. Follow the selected revision's tool error and structured-result rules, including `isError` for application failures. See [MCP tools](https://modelcontextprotocol.io/specification/2026-07-28/server/tools).

Text, image content, resources, and resource links serve different purposes. A URL alone does not establish that the model saw an image or read a file. Return actual bounded content when the workflow requires inspection, and retain authorization checks on later resource reads.

## Choose and test MCP compatibility explicitly

As checked on October 9, 2026, the current published revision is **2026-07-28**. That is a reference point, not a claim that every host or installed SDK supports it. Record host version, transport, SDK version, supported protocol revisions, authentication method, and exercised features in the worksheet. Pin documentation to those revisions.

| Revision family | Relevant lifecycle |
| --- | --- |
| 2025-11-25 and earlier | Initialization-based negotiation; use that revision's transport/session rules |
| 2026-07-28 | Per-request protocol/client metadata, `server/discover`, and no initialization handshake |

Modern and legacy behavior require deliberate compatibility handling. The stdio and HTTP bindings specify different fallback mechanics. Do not infer support from a package name or a successful `tools/list`, and do not downgrade automatically on every application error. See [current versioning](https://modelcontextprotocol.io/specification/2026-07-28/basic/lifecycle) and [the 2025-11-25 lifecycle](https://modelcontextprotocol.io/specification/2025-11-25/basic/lifecycle).

For modern HTTP, preserve and validate the required metadata headers against the request body, including method/name and any schema-designated parameters. The 2026 revision removes protocol sessions and the old GET stream, but still allows request-scoped SSE responses. A JSON-only server can be sufficient; a conforming client must handle the response forms required by its binding. Use SDK transport behavior rather than a guessed line-to-POST relay. Validate supplied Origins, authenticate protected requests, and configure proxy paths, timeouts, buffering, body limits, and connection cleanup. See [Streamable HTTP](https://modelcontextprotocol.io/specification/2026-07-28/basic/transports/streamable-http).

Protocol cancellation and connection loss do not undo an application commit. Preserve request keys across transport retries, reconnects, and any multi-round-trip interaction. Recovery and application completion remain governed by [the effect lifecycle](reliability.md).

## Optional remote OAuth integration

Build this only when an intended remote host needs its standard “Connect” flow. A remote host that supports safely provisioned bearer credentials may use that supported path instead. A custom device page or PAT endpoint does not establish standard connector compatibility.

For the standard flow, use an established implementation and the selected [MCP authorization specification](https://modelcontextprotocol.io/specification/2026-07-28/basic/authorization). Verify protected-resource and authorization-server discovery, a mutually supported client registration method, authorization-code/PKCE behavior, issuer and redirect validation, and resource/audience-bound access tokens. The 2026 specification prefers client metadata documents where supported; dynamic registration is not universally required. Test actual renewal and revocation behavior rather than assuming refresh tokens will be issued.

Public native clients cannot keep an embedded client secret confidential. Use the system browser and an appropriate callback flow when implementing native authorization-code login, following [RFC 8252](https://www.rfc-editor.org/rfc/rfc8252.html) and [OAuth security best practice, RFC 9700](https://www.rfc-editor.org/rfc/rfc9700.html). Keep provider keys, app access tokens, browser credentials, and operator credentials separate. Do not pass unrelated upstream tokens through as app authorization.

This option can use the app's existing identity infrastructure. It does not inherently require a new provider or service. Keep its added compatibility work separate from the basic local client release.

## Handle real files and exports

Reuse existing ownership, storage limits, attachment rules, and cleanup. File metadata alone is not a complete file workflow.

| Area | Required behavior |
| --- | --- |
| Upload | Enforce byte and decoded-size limits, accepted media, ownership, readiness, and attachment permissions |
| Staging | Separate staged bytes from committed references; reconcile retries and clean only proven unreferenced temporary data |
| Download | Authorize every private access; preserve exact bytes; bound transfer and verify integrity where available |
| Export | Preserve the documented representation, encoding, and complete paging/snapshot semantics |
| Local output | Refuse unexpected overwrite; reject path traversal and unsafe file targets; avoid deriving arbitrary paths from untrusted filenames |
| Deletion | Follow the app's attachment lifecycle and current ownership; do not delete a shared file because one reference disappeared |

Decide whether a file URL requires authentication or is an intentionally shareable bearer link. Do not describe an unguessable URL as private authentication. Public previews and capability URLs can disclose content before publication; publishing approval cannot retroactively protect an already exposed preview. Avoid original filenames and private metadata in public URLs and projections unless explicitly intended.

For server-side URL imports or image inspection, defend against SSRF at the network boundary: restrict schemes and destinations, prevent access to private/link-local/metadata services, account for DNS rebinding, validate every redirect, and bound time, redirects, bytes, decompression, and decoded dimensions. Prefer an established safe fetcher or egress control. A string hostname check alone is inadequate. Never forward app credentials to fetched URLs. Preserve intended private-network integrations only under their separate configured policy.

For active HTML/SVG previews, preserve isolation from app credentials and APIs and follow the product's content policy. For CSV intended for spreadsheet opening, decide how to handle formula-like untrusted cells; label any escaping mode so an exact-data export does not silently change content.

## Package and install independently

Use the app's supported runtime or native packaging ecosystem. Build a small client artifact with an explicit file allowlist or constructed staging directory. Include only runtime client code, required dependencies, and necessary notices. Exclude server configuration, env files, credentials, runtime data, logs, caches, and private fixtures.

Inspect the final archive and install it into an isolated user-owned location outside the repository. Run without source imports or development dependencies. Verify documented runtime requirements, executable names, setup commands, and the actual download/registry location. Do not solve installation with an unrequested global privileged command.

Start with a reliable explicit update path. If implementing self-update, verify install ownership, trusted release origin, integrity, rollback, timeouts, and preservation of profiles. A checksum fetched from the same origin detects corruption; it does not independently protect against that origin being compromised. A signing mechanism needs a separately trusted key to offer that protection.

Keep update banners out of MCP stdout and avoid self-overwriting a source checkout. Uninstall must affect only known owned artifacts. Installation alone should not silently alter an agent host's configuration; register it as part of the user's intended setup.
