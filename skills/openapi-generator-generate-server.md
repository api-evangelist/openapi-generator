---
name: openapi-generator-generate-server
description: Generate a server stub for a chosen framework from an OpenAPI document using the hosted OpenAPI Generator Online API, then download the result.
api: openapi-generator:openapi-generator
generated: '2026-08-06'
method: generated
source: openapi/openapi-generator-online-swagger.json
operations:
  - serverOptions
  - getServerOptions
  - generateServerForLanguage
  - downloadFile
---

# Generate a server stub

Base URL: `https://api.openapi-generator.tech`. No authentication.

## 1. List the frameworks

`GET /api/gen/servers` (`serverOptions`) returns a flat JSON array of every
supported server-stub framework — 65 at the last probe. Unpaginated.

```
GET https://api.openapi-generator.tech/api/gen/servers
```

Framework names are distinct from client generator names (for example
`aspnetcore`, `spring`, `go-server`). Use an exact string from this array.

## 2. Inspect the framework's options

`GET /api/gen/servers/{framework}` (`getServerOptions`) returns a map of
`CliOption` objects keyed by option name.

```
GET https://api.openapi-generator.tech/api/gen/servers/spring
```

## 3. Generate

`POST /api/gen/servers/{framework}` (`generateServerForLanguage`) with a
`GeneratorInput` body — `openAPIUrl` or inline `spec`, plus `options`.

```
POST https://api.openapi-generator.tech/api/gen/servers/spring
Content-Type: application/json

{
  "openAPIUrl": "https://example.com/openapi.yaml",
  "options": { "basePackage": "com.example.api" }
}
```

Returns `ResponseCode` — `{ "code": "<uuid>", "link": "<download url>" }`.

Not idempotent: a repeat POST produces a new artifact and a new code.

## 4. Download

`GET /api/gen/download/{fileId}` (`downloadFile`) with the `code`. Returns a zip
stream. Artifacts are transient; an unknown or expired code returns `500`.

## Errors

Spring Boot default envelope (`timestamp`, `status`, `error`, `path`). `404` for
an unknown `{framework}`; `500` for a bad download code. `401`/`403` are declared
on every operation but unreachable — the API has no auth layer. See
`errors/openapi-generator-problem-types.yml`.

## Build-time alternative

For a stub that regenerates as part of a build, prefer the Maven, Gradle or sbt
plugin over the hosted API — same options, pinned version, no network
dependency on a service running twelve minor versions behind the release line.
See `cli/openapi-generator-cli.yml` and https://openapi-generator.tech/docs/plugins.
