---
name: openapi-generator-generate-client
description: Generate a client library (SDK) in a chosen language from an OpenAPI document using the hosted OpenAPI Generator Online API, then download the result.
api: openapi-generator:openapi-generator
generated: '2026-08-06'
method: generated
source: openapi/openapi-generator-online-swagger.json
operations:
  - clientOptions
  - getClientOptions
  - generateClient
  - downloadFile
---

# Generate a client library

Base URL: `https://api.openapi-generator.tech`. No authentication — the service
is anonymous (`authentication/openapi-generator-authentication.yml`).

## 1. Pick a generator

`GET /api/gen/clients` (`clientOptions`) returns a flat JSON array of every
supported client generator name — 84 at the last probe. There is no pagination;
the whole list comes back in one response.

```
GET https://api.openapi-generator.tech/api/gen/clients
```

Choose an exact string from that array. A name that is not in the array produces
a `404` in the next step.

## 2. Inspect that generator's options

`GET /api/gen/clients/{language}` (`getClientOptions`) returns a map keyed by
option name, each value a `CliOption` (`opt`, `description`, `type`, `default`,
`optValue`, `enum`). Read it before generating — option names differ per
generator.

```
GET https://api.openapi-generator.tech/api/gen/clients/go
```

## 3. Generate

`POST /api/gen/clients/{language}` (`generateClient`) with a `GeneratorInput`
body. Supply either `openAPIUrl` (a URL the service will fetch) or `spec` (the
document inline), plus `options` keyed by the names from step 2.

```
POST https://api.openapi-generator.tech/api/gen/clients/go
Content-Type: application/json

{
  "openAPIUrl": "https://raw.githubusercontent.com/OpenAPITools/openapi-generator/master/modules/openapi-generator/src/test/resources/2_0/petstore.yaml",
  "options": { "packageName": "petstore" }
}
```

The response is a `ResponseCode`: `{ "code": "<uuid>", "link": "<download url>" }`.

**This call is not idempotent.** Retrying generates a fresh artifact with a new
code — it does not replay the first result. Do not wrap it in an automatic retry
loop.

**Do not send `authorizationValue` casually.** It exists so the service can
fetch a credential-protected `openAPIUrl`, which means handing a third-party
secret to a public unauthenticated endpoint. For a protected spec, use the CLI
instead (`cli/openapi-generator-cli.yml`).

## 4. Download

`GET /api/gen/download/{fileId}` (`downloadFile`) with the `code` from step 3
returns a zip stream. Artifacts are transient — download immediately.

```
GET https://api.openapi-generator.tech/api/gen/download/<code>
```

## Errors

Errors return the Spring Boot default envelope, not RFC 9457:

```
{"timestamp":"2026-08-06T18:33:42.150Z","status":404,"error":"Not Found","path":"/api/gen/clients/notalanguage"}
```

- `404` — unknown route or unknown `{language}`. Re-read step 1.
- `500` on download — malformed or expired `fileId`. This is undeclared in the
  published spec and is returned instead of a `404`; treat it as "gone", not as a
  server fault to retry against.
- `401` / `403` are declared on every operation but are framework defaults; no
  auth layer exists to produce them.

See `errors/openapi-generator-problem-types.yml` and
`conventions/openapi-generator-conventions.yml`.

## When to use the CLI instead

The hosted service reports version 7.12.0 while the current release line is
7.24.0, publishes no rate limits, and has no status page or SLA. For anything
reproducible or in CI, use `openapi-generator-cli generate -i <spec> -g <name> -o <dir>`
(`cli/openapi-generator-cli.yml`).
