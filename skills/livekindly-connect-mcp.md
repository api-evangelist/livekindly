---
name: Connect to the LIVEKINDLY MCP server
description: >-
  Discover and authorize against the OAuth-protected WordPress MCP Adapter endpoint on
  thelivekindlyco.com, using only the RFC 8414 / RFC 9728 discovery documents the site actually
  serves — and know what remains unknowable without a credential.
api: mcp/livekindly-mcp.yml
operations: [tools/list, initialize]
generated: '2026-08-04'
method: generated
source: >-
  Derived from mcp/livekindly-mcp.yml, scopes/livekindly-scopes.yml and the two live
  /.well-known/ documents saved in well-known/. Probe results observed 2026-08-04.
---

# Connect to the LIVEKINDLY MCP server

`thelivekindlyco.com` runs the WordPress MCP Adapter and — unusually for a corporate marketing
site — fronts it with a complete OAuth discovery surface. This skill is the honest procedure for
connecting. It cannot hand you a tool list, because there is no public one.

## What is actually there

| Surface | URL | Anonymous result |
|---|---|---|
| MCP namespace index | `https://thelivekindlyco.com/wp-json/mcp` | 200 |
| OAuth-protected MCP endpoint | `https://thelivekindlyco.com/wp-json/mcp/mcp-oauth-server` | 401 `mcp_unauthorized` |
| Default MCP endpoint | `https://thelivekindlyco.com/wp-json/mcp/mcp-adapter-default-server` | 401 `rest_forbidden` |
| Abilities registry | `https://thelivekindlyco.com/wp-json/wp-abilities/v1/abilities` | 401 `rest_forbidden` |

## Steps

1. **Attempt the call and read the challenge.** Post JSON-RPC to the OAuth endpoint with
   `Accept: application/json, text/event-stream`:

   ```
   POST https://thelivekindlyco.com/wp-json/mcp/mcp-oauth-server
   {"jsonrpc":"2.0","id":1,"method":"tools/list"}
   ```

   You get HTTP 401 and — correctly — a bearer challenge that tells you where to go next:

   ```
   WWW-Authenticate: Bearer realm="https://thelivekindlyco.com",
     resource_metadata="https://thelivekindlyco.com/.well-known/oauth-protected-resource"
   ```

   Do not hardcode endpoints. Follow `resource_metadata`. This is the RFC 9728 flow working as
   designed.

2. **Fetch the protected-resource metadata.** `GET /.well-known/oauth-protected-resource` returns
   the resource identifier (`.../wp-json/mcp/mcp-oauth-server`), its authorization server
   (`https://thelivekindlyco.com`), `bearer_methods_supported: ["header"]` and
   `scopes_supported: ["mcp"]`.

3. **Fetch the authorization-server metadata.** `GET /.well-known/oauth-authorization-server` on the
   named issuer returns:
   - `authorization_endpoint`: `https://thelivekindlyco.com/oauth/authorize`
   - `token_endpoint`: `https://thelivekindlyco.com/oauth/token`
   - `revocation_endpoint`: `https://thelivekindlyco.com/oauth/revoke`
   - `grant_types_supported`: `authorization_code`, `refresh_token`
   - `code_challenge_methods_supported`: `S256` — **PKCE is mandatory**
   - `token_endpoint_auth_methods_supported`: `none` — public clients only, send no client secret
   - `client_id_metadata_document_supported`: `true`

4. **Identify your client.** There is **no** `registration_endpoint`, so RFC 7591 dynamic client
   registration is not available. Use the client-ID-metadata-document pattern instead: your
   `client_id` is an HTTPS URL that resolves to your client metadata document.

5. **Run authorization code + PKCE.** Generate a verifier, send the `S256` challenge to
   `/oauth/authorize`, exchange the code at `/oauth/token`, request scope `mcp`. A LIVEKINDLY
   WordPress account has to approve the grant — there is no self-service path.

6. **Retry with the token.** Send `Authorization: Bearer <token>`, then `initialize` followed by
   `tools/list`. Only at this point does the tool set become knowable.

7. **Revoke when done.** `POST /oauth/revoke`.

## Rules

- **Do not assume a tool set.** `tools/list` is gated on both endpoints and the backing
  `wp-abilities` registry is gated too. No llms.txt or documentation names a single tool. Any tool
  list attributed to this server without an authenticated introspection is invented — including
  "the usual WordPress MCP Adapter defaults."
- **`mcp` is one coarse scope.** There is no per-resource or per-ability decomposition, and the
  scope's semantics are not published. A token granted here carries whatever the approving
  WordPress user can do. Treat it as high-privilege and short-lived.
- **This is not a LIVEKINDLY product.** The company advertises this endpoint nowhere and offers no
  developer support. It was found by enumerating `https://thelivekindlyco.com/wp-json/`. Anyone
  planning to depend on it should ask LIVEKINDLY first — it can disappear with a plugin update.
- **Do not attempt credentials you have not been given.** Every observation in this repo was made
  anonymously; that is the only acceptable posture.

## If you only need data

You almost certainly do not need MCP here. Everything public on this host is already readable
without a credential through the content API — see `livekindly-read-newsroom.md` and
`livekindly-brands-and-partners.md`.
