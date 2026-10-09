# Primary sources and review scope

Reviewed **October 9, 2026**. Follow the specification matching the protocol/SDK/host you actually ship. A dated specification link is more reproducible than an unqualified “latest,” but it does not establish compatibility by itself.

## Protocol references

| Source | Relevant use |
| --- | --- |
| [MCP 2026-07-28 specification](https://modelcontextprotocol.io/specification/2026-07-28) | Current published revision at review time |
| [Versioning and compatibility](https://modelcontextprotocol.io/specification/2026-07-28/basic/lifecycle) | Current lifecycle entry point redirects to versioning; per-request metadata and compatibility with initialization-based clients |
| [Current stdio binding](https://modelcontextprotocol.io/specification/2026-07-28/basic/transports/stdio) | Local subprocess transport and its compatibility behavior |
| [Current Streamable HTTP binding](https://modelcontextprotocol.io/specification/2026-07-28/basic/transports/streamable-http) | Requests, metadata/header checks, streams, and Origin validation |
| [Current tools specification](https://modelcontextprotocol.io/specification/2026-07-28/server/tools) | Discovery, tool results, schemas, and errors |
| [2025-11-25 lifecycle](https://modelcontextprotocol.io/specification/2025-11-25/basic/lifecycle) | A pinned reference for older initialization-based clients |
| [MCP authorization](https://modelcontextprotocol.io/specification/2026-07-28/basic/authorization) and [security considerations](https://modelcontextprotocol.io/specification/2026-07-28/basic/authorization/security-considerations) | Optional remote integration; discovery, client registration, token audience, issuer, and authorization-code protection |
| [RFC 8628: OAuth device authorization grant](https://www.rfc-editor.org/rfc/rfc8628.html) | Device-flow protocol and polling rules; custom app flows must not claim compliance from naming alone |
| [RFC 8252: OAuth for native apps](https://www.rfc-editor.org/rfc/rfc8252.html) | System-browser authorization and public native-client behavior |
| [RFC 9700: OAuth security best current practice](https://www.rfc-editor.org/rfc/rfc9700.html) | Security requirements when implementing that optional integration |
| [RFC 9110, conditional requests](https://www.rfc-editor.org/rfc/rfc9110.html#section-13.1.1) | Strong `If-Match` preconditions for HTTP resources |
| [RFC 8785: JSON Canonicalization Scheme](https://www.rfc-editor.org/rfc/rfc8785.html) | An Informational specification for deterministic JSON serialization |
| [JSON Schema Draft 2020-12 validation](https://json-schema.org/draft/2020-12/json-schema-validation) | Example schema dialect, validation keywords, and `format` behavior |

The recommendations about operation inventories, receipt authorization, effect recovery, credential lifecycle, UI preservation, and release verification are application engineering guidance. They are not additional MCP requirements. The guide's request keys, contract versions, policy fields, and receipt states are illustrative conventions rather than a newly standardized wire format.

## Hosting examples

[Deployment and scaling](deployment-and-scaling.md) uses [DigitalOcean Droplets](https://docs.digitalocean.com/products/droplets/), [GitHub Actions environments](https://docs.github.com/en/actions/reference/workflows-and-actions/deployments-and-environments), [AWS Lambda guidance](https://docs.aws.amazon.com/lambda/latest/dg/best-practices.html), and [Cloud Run's runtime contract](https://docs.cloud.google.com/run/docs/container-contract) as optional examples of different infrastructure concerns. These provider documents were checked at review time. They are not required products or substitutes for discovering the target setup and its current limits.

## Source evidence

[The reference comparison](reference-apps.md) links the publicly retrievable New Drugs files at one immutable commit. These substantiate only the described code patterns. The other named domain examples are labeled illustrations; no private repository links or unpublished installation commands are required to use this guide.

The review combined application-source inspection with retrieval of the cited public files and primary protocol references. It did not execute a reference application or connect to a user account. Live server behavior, package availability, provider guarantees, and host/platform conformance must be verified in the target project.

The supplied JSON Schemas and valid fixtures were checked with a Draft 2020-12 validator, including rejection of nine malformed-input/output cases. Relative links, section anchors, fenced JSON, and shell syntax were checked. The shell commands were checked against the pinned public CLI source, not run against a live account. These are documentation checks, not a substitute for the target application's acceptance tests.

When adapting or refreshing the guide, recheck the protocol references and public commit evidence. Keep recommended improvements distinct from features actually present in a reference. Record any untested guarantee in the project's [verification evidence](verification.md), rather than converting it into an assurance through stronger wording.
