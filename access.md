# Access and human authorization

The application needs to answer three different questions: which account is calling, what that connection may access now, and whether the proposed action is within the person's request. A write token answers only part of that.

## Separate identities and enforce current permissions

Construct an actor on the server from validated credentials. Include the account, source (browser, external connection, or hosted run), connection/run ID, granted scopes and restrictions, and current account status. Never accept a trusted owner, role, or run authority from ordinary tool arguments.

Effective access is the intersection of:

1. The connection's grant, including tenant/resource restrictions.
2. The account's current permissions for the requested resource.
3. The operation's source, role, and product policy.

Check ownership, membership, blocks, visibility, and account suspension as applicable. Hiding a tool from discovery is helpful but cannot enforce permission. A guessed operation name or resource ID must pass the same checks.

Trace old APIs, sockets, file URLs, export routes, and background work too. Either enforce the same connection policy on those paths or reject agent credentials there. Preserve existing browser authentication. Scope newly exposed credentials so they cannot fall through to a legacy unrestricted route. Check each socket room and each private subscription against its current audience.

State exactly when permission changes take effect. Per-request checks reject the next request; they do not undo admitted work. For strict protection against a membership change, revocation, or cancellation racing a write, coordinate the permission update and guarded effect using the same transactional lock, version guard, or equivalent serialization. Snapshot reads alone can permit conflicting decisions. Delayed dispatch needs its own policy and check.

## Use named, revocable grants

Store a distinct credential record with a nonsecret ID, owner, name, scopes/restrictions, creation time, expiry, revocation state, and optional last-use metadata. Generate opaque bearer secrets with a cryptographically secure generator. For high-entropy random secrets, store a cryptographic verification hash rather than the raw value. Passwords require their own password-storage scheme; do not apply this token rule to passwords.

An expiry such as 30 days can be a reasonable product default, but it is not a protocol requirement. Preserve existing valid credentials when adding this feature. Check expiry and revocation during authentication even if expired database rows have not been removed. Bound any authorization cache and document its revocation delay. Self-contained tokens require a revocation strategy if the product promises revocation before token expiry.

Expose credential management only through the intended account-management authority. An external connection should not silently mint a more powerful connection. A narrowly defined self-revoke operation may be available even to a read-only connection.

Use TLS outside explicitly configured local development. Keep secrets out of URLs, command arguments, chat, logs, telemetry, crash reports, and discovery responses. Authentication and issuance responses should not be cached by shared intermediaries. Rate-limit credential creation, code guessing, and failed authentication without accidentally locking legitimate owners out through one attacker-controlled counter.

## Choose one working login flow first

### Browser-approved device login

The CLI starts a request and privately retains a high-entropy polling secret. It displays a short code and approval link. In an authenticated browser, the person sees the actual account, matching code, connection name, permissions, and expiry, and approves or denies. The CLI obtains the credential over the private polling channel, verifies the account and granted access, then saves the connection.

The requesting name is a label supplied by a client, not verified client identity. Show the code and account clearly to reduce approval of someone else's request. Do not approve a code merely because it appears in a page, message, or retrieved document. Preserve the intended request across sign-in without automatically granting it.

Keep the browser code separate from the polling secret. The short code must never retrieve or derive a bearer token. Apply CSRF protection and the app's browser-origin checks to approval and denial; CORS alone is not authorization. Validate returned approval URLs against the configured relationship between API and browser origins. Reject unexpected schemes, embedded credentials, paths, and redirects that could expose secrets.

If implementing the OAuth device grant, implement [RFC 8628](https://www.rfc-editor.org/rfc/rfc8628.html), not just its field names. Its polling rules include waiting the supplied interval (five seconds when omitted), adding five seconds after `slow_down`, reducing frequency after connection timeouts, and stopping on terminal errors. Use bounded request timeouts and an overall deadline. An app-specific device flow may borrow these rules, but should be labeled app-specific.

Device authorization is useful for terminal and cross-device setup. An existing native-client browser login may be equally suitable. Neither choice requires an embedded browser or collecting the person's password.

### Make approval and issuance recoverable

Choose an implementation supported by the existing storage system:

| Storage capability | Approach |
| --- | --- |
| Suitable transactions | Atomically persist the winning account/grant and its credential record |
| Atomic conditional writes | Persist an immutable approval decision and issuance ID, then finish issuance idempotently |

Use a unique code constraint with collision handling. Exactly one approval/denial wins the pending request. Persist the full approved grant, including its fixed expiry, before recovery can issue anything. Recovery uses a unique issuance ID and create-if-absent semantics; it must not overwrite an existing credential's revocation, owner, or scope. Enforce connection-count limits consistently under concurrent approvals if the app has a cap.

Handle crashes after approval, after credential creation, and after delivery but before local save. A database transaction does not make token delivery over the network atomic. Define a short, protected retry window for returning the same issued secret, or fail safely with a visible, revocable stranded grant. Never create another active grant on every poll. Recheck account and credential status before delivery; deny expired or revoked credentials.

Prefer the existing auth library's issuance/delivery mechanism. One custom option uses a random bearer with bounded encrypted delivery storage, separate from the durable verification hash. Another derives the bearer from the high-entropy polling secret, as the public reference does. **In the derived design, the polling secret remains credential-equivalent for the bearer lifetime, even after the device request expires.** Treat it accordingly. Hash-only credential records do not mean every recovery design can avoid all reversible delivery material. Do not improvise cryptography from a short code or account name.

### PAT alternative

A PAT is a named token the person creates in the app with chosen access and expiry. Reveal it once through the protected creation flow; list responses contain metadata only. The CLI can receive it through a non-echoing local prompt, stdin, or a supported secret manager, then verify identity before saving.

Stdin avoids command-history and process-argument exposure but is not automatically private: an agent tool or pipeline may record its input. Do not ask the person to paste a token into the conversation, or write a shell command containing it. Use the host's actual secret-entry mechanism. Keep PAT support only if it is useful; the guide does not require implementing every login alternative.

## Save and disconnect safely

Use the OS credential store where appropriate, or private configuration with platform-specific protection. POSIX files normally use owner-only permissions and directories; Windows needs appropriate ACLs. Check ownership and file type, avoid symlinks and unsafe parent paths, and use secure file opens rather than assuming a prior path check eliminates races. Serialize concurrent changes and replace files atomically. Keep profile origin and account identity pinned; verify before replacing a working profile.

| Action | Promised effect |
| --- | --- |
| Local logout | Remove the selected local credential; other profiles and browser sessions remain intact |
| Server revocation | Reject subsequent authentication according to the documented cache/in-flight policy |
| Revoke and logout | Revoke first, then remove local state; retain recovery information if the response is lost |
| Uninstall | Remove only the package and explicitly owned configuration/registrations |

A retry of self-revocation may receive an authentication failure because revocation succeeded. That proves the credential is unusable if confirmed by the server, not the exact historical reason it became unusable. Report what is known. Losing local config does not revoke a grant; retain browser management for stranded or lost connections.

## Authorize the action, without repeated ceremonies

An ordinary edit already requested by the person normally needs no second approval. A broad write grant, however, is not a request to publish, send, purchase, or delete arbitrary content. Determine the product's consequential actions from their effects and the user's task, not from an assumption that every agent write needs a dialog.

For a consequential action, record what the person actually authorized: account, operation, stable target IDs, content or revision, destination/audience, relevant cost limits, and validity window where applicable. Bind execution to those details. A request to publish a particular draft must not publish a replacement draft. The [state/retry design](reliability.md) explains the commit boundary.

Some requests explicitly delegate creation and execution, such as writing and publishing within stated constraints. Represent that bounded delegation honestly; do not claim the person reviewed bytes that did not exist yet. Standing instructions need defined targets, effects, duration, and cancellation. They cannot expand themselves because retrieved content suggests it. Honor product rules requiring human-authored text.

| Evidence | What the application can establish |
| --- | --- |
| `confirmed: true`, `--yes`, or similar | The token holder or host asserted authorization. The app cannot independently establish human approval from this field |
| App-held action review | An authenticated approval channel accepted the stored action binding, independently of a tool argument |
| Verified signed host assertion | The identified host asserted the bound action, subject to its own approval guarantees and trust agreement |

An app review should show the consequence and target, bind an immutable action digest to the account/connection and logical request, expire appropriately, and be reserved/consumed atomically with that request. Retries of the same action use that association; different actions cannot reuse it. A signature needs audience, issuer, expiry, nonce/replay checks, and action binding. It is not inherently proof that a human approved.

Even browser approval establishes activity through an authenticated account channel, not proof of a physical person's attention. A browser can be automated. If the product requires stronger user verification, select and verify that mechanism explicitly. Do not describe a client boolean, ordinary session, or model-generated quotation as independent human proof.

Hosted agents need trusted request/run context outside model arguments. Keyword matching for “publish” is insufficient: negation, quoted instructions, examples, historical requests, and changed targets must not authorize effects. Stored pages, files, tool descriptions, and tool output cannot grant access or override the current human instruction.
