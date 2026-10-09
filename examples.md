# Examples you can adapt

The JSON files here define a **fictional document application**, not a new protocol or a functioning client. They contact no service and contain no credentials. Implement equivalent fields in your existing API; do not add document features to an unrelated product.

## One ordinary authorized edit

The person asked to replace one document's content. The client has already read that document as revision `r17`, and the account currently has editor access. No additional confirmation dialog is required for this ordinary edit.

- [Operation definition](examples/documents.update.json): input/output schemas and application policy.
- [Request](examples/update.request.json): exact input, accepted contract, and a synthetic request key.
- [Result](examples/update.result.json): committed new revision, receipt state, and an example link.

The schemas use JSON Schema Draft 2020-12. The metadata around them is illustrative application design. `policy` text documents rules that the server must implement; it does not enforce them. A schema cannot check current membership, commit atomically, or prove authorization.

This fictional client flow is expressed as steps rather than a runnable installation command:

```text
1. Connect to the intended site/account and verify the returned grant.
2. Discover documents.get and documents.update; read their actual contracts.
3. Read the authorized document and retain its stable ID and current revision.
4. Prepare the exact requested content and a new logical request key.
5. Save the request privately, then send it through the operation adapter.
6. If the response is lost, recover that key; do not create a second action.
7. Read the current document and verify the requested change in the browser.
```

The sample key is a fixture. Generate a fresh key for a real new action. Do not submit the synthetic document ID or example URL to a real app. The example URL uses the reserved `.example` domain.

The request expects `r17`; the result reports `r18`. An identical retry returns its authorized receipt even though `r17` is no longer current. A different content value with the same key is a conflict. A different action against stale `r17` is a state conflict. The resulting revision can be different again by the time the client reads, because another authorized actor may have edited it.

For publication, add the real audience/destination and the exact draft or bounded delegation being authorized. For deletion, bind the stable record generation and relevant related state. A generic `confirmed: true` field would be a host assertion, not independent human proof. See [authorization evidence](access.md#authorize-the-action-without-repeated-ceremonies) and [executor ordering](reliability.md#executor-ordering).

## Public CLI syntax example

The following read-only command syntax is present in the pinned [New Drugs CLI source](https://github.com/cfreshman/newdrugs/blob/e7720bb805285fdd6942f7ce072ddd9f0ab0d7b4/cli/index.ts). Use it only if that client is already installed and the `default` profile is intentionally connected to the account you want to inspect. These commands are not installation instructions, and current help/contracts remain authoritative.

```sh
newdrugs --profile default read identity.get
newdrugs --profile default search --keyword --limit 10
newdrugs --profile default describe posts.list
newdrugs --profile default read posts.list '{"scope":"own","limit":10}'
```

No account was connected and no live commands were executed to prepare this guide. The public source supports the syntax; it does not prove which package or server is currently deployed. Private account results should remain out of shared logs.

For hosts that accept an `mcpServers` object, this configuration reuses that client's private profile:

```json
{
  "mcpServers": {
    "newdrugs": {
      "command": "newdrugs",
      "args": ["--profile", "default", "mcp"]
    }
  }
}
```

Verify the actual host's configuration, executable search path, protocol support, and environment before using it. This is a local subprocess setup. It does not claim remote OAuth support, and it includes no secret.

## Nonsecret connection prompt

Generate your app's version from its real package instructions, command syntax, origins, and identity operation. Replace every bracketed field before displaying it to users.

```text
Connect my agent to [site] as [intended account] with [access and expiry].
Use the app's documented installation instructions: [instructions location].
Start [actual browser-approved login command] and keep the private polling
process running. Show me its approval link and short code.
Use my browser to approve only if I have authorized browser interaction for
this exact account and grant; otherwise leave the approval for me.
If the account or grant differs, let me resolve it before proceeding.
Do not collect my password or extract browser cookies.
Verify the returned identity and actual access with [identity command].
Keep secrets out of chat, logs, command arguments, and host configuration.
Connect and inspect capabilities only; do not change app content during setup.
Register local MCP using [host-specific instructions] if that is in my request.
```

For a PAT flow, replace browser/polling instructions with the host's protected local secret-entry path. Never generate a prompt containing the token. Connection setup is separate from authorization to act on content afterward.
