# One Login to Rule Them All: Cross-App Access for MCP

| Field | Details |
| --- | --- |
| **Speaker** | Garrett Galow |
| **Affiliation** | WorkOS |
| **Type** | Track keynote |
| **Date** | 2026-04-28 |
| **Video** | [Watch on YouTube](https://www.youtube.com/watch?v=EmhRyw6xeT0) |
| **Tags** | `safety-and-governance` `coding-and-developer-tools` `apis-mcp-and-protocols` |

## Summary

Garrett Galow, who runs product at WorkOS, starts from a familiar annoyance: connecting an MCP client (a standard way for agents to call tools and data) to a service demands another consent screen, even when you already use the same company login for both. With a dozen servers per person and dozens of teammates, those approvals add up — but the real problem is worse than irritation.

Because each connection is authorized by the individual user rather than the company's identity provider (IdP, the system like Okta or Microsoft Entra that signs people in), IT loses a central view. An approved Cursor install and an unrecognized client can both become routes into sensitive Figma or Notion data. And when a laptop is compromised — Galow was affected by the recent axios npm supply-chain incident — revoking someone's SSO session does not necessarily kill the access and refresh tokens already issued outside it. Those can linger for days, weeks, or months.

His answer is **Cross-App Access (XAA)**: let the IdP act as a trust intermediary between applications that already trust it. The user logs into Okta once, and the client silently obtains credentials for the services the company permits, using a scoped token exchange. The demo shows Claude Code connecting to Figma with no extra consent click. Galow is clear about the boundary: XAA handles authentication and central revocation, while fine-grained authorization, discovery, and client registration still need work across an uneven ecosystem.

## Key Takeaways

- **Per-server consent breaks enterprise SSO.** Each MCP connection is authorized by the user, not the company IdP, so shared identity is never carried across apps.
- **IT loses visibility and control.** Administrators cannot easily see which MCP clients and services people connect, or govern which agents touch sensitive data.
- **Revoking SSO does not revoke everything.** Access and refresh tokens obtained outside the IdP can preserve standing access after an incident or a departure.
- **XAA makes the IdP the trust broker.** Since Cursor and Figma both already trust the same Okta instance, the IdP can issue credentials without fresh human consent.
- **The mechanism is a scoped token exchange.** Sign in once, request an ID-JAG (an IdP-issued assertion), and exchange it at the target service for an ordinary access token.
- **Silent renewal restores a revocation boundary.** Short-lived access tokens are renewed behind the scenes; if the IdP session is cut, renewal fails.
- **Admins authorize relationships, not just logins.** Okta's managed connections policy lets one app request access to another, while membership is still required.
- **Authentication is not authorization.** By default XAA grants the user's own service permissions; scoped, policy-imposed caveats are still being discussed.

## Detailed Notes

### Why every MCP server needs another login · 00:34
- Adding MCP servers in Cursor means a config file, then connecting each one: a browser window, a consent screen, and a redirect back.
- Sometimes a connection appears to be forgotten and the flow repeats, seemingly without reason.
- Per person this is a handful of logins; across a team it becomes dozens of people managing consent screens they do not really read.
- The root cause is that MCP uses OAuth (an authorization framework built around two systems not trusting each other), which assumes each app knows nothing about the others.
- Enterprise **single sign-on** already links apps through an IdP such as Okta or Microsoft Entra, but MCP's connection flow does not carry that shared identity into the delegation step.

### The credentials IT cannot see · 03:47
- By connecting arbitrary clients to arbitrary services outside the IdP, users remove the one place administrators can observe and govern those connections.
- An approved Cursor install and a client IT never sanctioned can both reach sensitive Figma or Notion data once a user grants access.
- Galow was caught in the axios npm supply-chain compromise about a week before the talk and did not know which dependency had pulled in the bad package.
- IT detected the machine, cut its network access, and invalidated his Okta sessions across applications — but local MCP connections and API keys needed a separate investigation.
- **Revoking an SSO session does not guarantee revoking credentials obtained outside it.** Access tokens and especially refresh tokens can preserve access, sometimes for months depending on service policy.
- **SCIM** (a standard for automating user provisioning and deprovisioning) can close the account-lifecycle gap, but many companies do not deploy it.
- Onboarding shows the same fragmentation: IT may configure the servers, yet each employee still authenticates and manages the connections.

### Use the trust apps already share · 06:37
- **Cross-App Access (XAA)** makes the IdP a trust intermediary between applications.
- Example: Cursor is the MCP client, Figma is the target service, and Okta is the company IdP.
- Cursor already trusts the company's `workos.okta.com` instance to sign employees in, and so does Figma — both know the user belongs to them.
- XAA uses those existing relationships to issue credentials for the client without a new human consent step.

### Demo: one Okta login, Figma connects · 07:36
- On one side, ordinary Claude Code with a Figma MCP server that still needs authentication — connecting would trigger the usual consent screen.
- On the other side, an explicitly beta, XAA-compatible Claude Code already configured against Galow's Okta environment.
- He logs into Okta once, starts the beta client, and opens the MCP server list: Figma is connected with no consent click.
- The visible difference is tiny on purpose — the goal is to *remove* an interaction, not add one.
- The same pattern extends from a single service like Figma to a whole list of MCP servers.

### The exchanges behind the magic · 09:36
- Four systems are involved, with the target's authorization server separated from its resource server (the API the client actually calls).

| Actor | Responsibility |
|---|---|
| Claude Code | Request credentials and call MCP tools |
| Okta | Authenticate the user and authorize cross-app requests |
| Figma authorization server | Validate the grant and issue an access token |
| Figma MCP resource server | Serve requests authenticated with that token |

1. **Sign into the IdP.** Okta returns an ID token and a refresh token, which Claude Code keeps.
2. **Request an ID-JAG.** **ID-JAG** (Identity Assertion JWT Authorization Grant) is an IdP-issued assertion for obtaining access elsewhere. Claude Code uses its refresh token to ask for one for Figma; Okta checks membership and the cross-app policy.
3. **Exchange the grant at Figma.** Claude Code sends the ID-JAG to Figma's authorization server, which validates it through its Okta trust and issues an access token.
- The token exchange is a form-encoded request using the `jwt-bearer` grant type with the ID-JAG as the assertion; the client still needs whatever authentication its registration requires.
- The key boundary: the ID-JAG goes to the **authorization server**, which validates it before issuing the resource credential.
- Step four is ordinary MCP: the same OAuth access token type as before. **XAA changes how the credential is obtained, not what the resource server receives.**

### Renewal and revocation boundaries · 11:54
- After the initial enterprise login, ID-JAG issuance and the Figma exchange are invisible to the user.
- How often the user must log in interactively is an IT policy choice — Galow cites daily and weekly as examples.
- Access tokens are often short-lived (he uses five minutes to illustrate); when one expires, the client silently repeats issuance and exchange.
- Five minutes is an illustration, not a stated lifetime for the demonstrated Figma token or a requirement of XAA.
- As long as the SSO session is active and policy allows it, the client keeps getting fresh grants.
- **Revocation boundary:** an already-issued access token may remain usable until it expires, but cutting access or locking the Okta session stops the next renewal. This brings continuing access back under the IdP without interrupting the user on every short token.

### What admins, clients, and servers must do · 13:06
- **Admins:** reuse the SSO applications you already have (Cursor or Claude Code, plus Figma). In Okta's managed connections portal, permit one app to request access to another — for example, Cursor to request Figma.
- When Cursor asks for a Figma ID-JAG, Okta checks that source-to-target policy before issuing it. The user must still belong to both applications; the connection policy does not replace membership.
- **MCP clients:** hold an XAA-compatible SSO connection with XAA enabled on the customer's IdP connection, request an ID-JAG, exchange it at the target authorization server, then use the returned token for normal MCP calls.
- IdP support is new and still developing; Galow says WorkOS implements the SSO, grant-request, and exchange stages, and that Cursor and Anthropic use that support through their WorkOS SSO connections.
- **MCP servers:** advertise support for the ID-JAG flow, accept the assertion, validate it against the trusted IdP (verifying the signed JWT, not merely decoding it), and only then issue an access token.
- Galow calls verification the familiar signed-JWT validation problem and points builders to the spec and a technical blog; he notes Claude is good at explaining the IETF terminology.

### Permissions, discovery, and remaining gaps · 17:00
- Asked whether XAA also solves **authorization** (for example, choosing Figma token scopes), Galow answers: by default, no. The user connects as themselves and gets their own Figma permissions.
- IdP-imposed permission caveats are under discussion but are not part of the spec he presents.
- How does a client know which app to request? Via an **audience**; he uses `mcp.figma.com` as a simplified example, with Okta mapping that target to its Figma application.
- Asked whether audience could be abused to escalate scopes, he says no: audience selects the recipient, it does not define a permission set. Okta must recognize the target and Figma must validate the assertion.
- **Microsoft Entra does not yet support XAA**; WorkOS is working with Microsoft on adoption. At the talk, availability was Okta-only, with OIDC connections supported and SAML planned.
- A refresh token, ID token, or SAML assertion can each prove the user's application session; that initial proof is tied to the SSO connection type.
- An attendee describes Entra integration friction: differing client behavior plus a resource/scope mismatch. Galow confirms requested scopes must fit what the server permits, without granting unauthorized scope, and leaves the details for after the talk.
- **Client registration** remains uneven. Entra lacked Dynamic Client Registration (DCR) in that setup, forcing a proxy. He describes DCR as broadly supported now, but an unsupported pairing requires preregistering the client.

| Registration approach | How the client is known |
|---|---|
| Preregistration | A registration exists before connection |
| Dynamic Client Registration (DCR) | The client registers programmatically |
| Client ID Metadata Documents (CIMD) | The server retrieves published client metadata |

- **CIMD** is newer and a better experience, but with less ecosystem support (a few months old) at the time. XAA removes repeated consent through shared enterprise trust, while discovery, scope handling, and client registration still need both ends to agree.

## Resources

- [Cross-App Access](https://oauth.net/cross-app-access/) — overview of the XAA approach.
- [Identity Assertion Authorization Grant draft](https://datatracker.ietf.org/doc/html/draft-ietf-oauth-identity-assertion-authz-grant-00) — the IETF draft behind the token exchange.
- [Cross App Access in Okta](https://developer.okta.com/docs/concepts/xaa/) — current Okta documentation for XAA roles and token exchanges.
- [MCP authorization specification](https://modelcontextprotocol.io/specification/2025-11-25/basic/authorization) — discovery, client registration, and authorization requirements.
- [RFC 9728: OAuth 2.0 Protected Resource Metadata](https://www.rfc-editor.org/rfc/rfc9728.html) — the protected-resource discovery standard.
- [Axios supply-chain compromise postmortem](https://github.com/axios/axios/issues/10636) — the March 2026 incident Galow describes.
- "Cross App Access: the enterprise way to govern AI app integrations" — WorkOS's walkthrough of the flow.
