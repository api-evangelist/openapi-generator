---
name: openapi-generator-discover-generators
description: Discover which client and server generators OpenAPI Generator supports and what each one can be configured with, before committing to a target language or framework.
api: openapi-generator:openapi-generator
generated: '2026-08-06'
method: generated
source: openapi/openapi-generator-online-swagger.json
operations:
  - clientOptions
  - serverOptions
  - getClientOptions
  - getServerOptions
---

# Discover generators and their options

The read-only half of the Online Generator API. No authentication, no
pagination, no rate-limit headers — four GETs and you have the full surface.

## 1. Both catalogs

```
GET https://api.openapi-generator.tech/api/gen/clients   # clientOptions   -> 84 names at last probe
GET https://api.openapi-generator.tech/api/gen/servers   # serverOptions   -> 65 names at last probe
```

Each returns a plain JSON array of strings. The two lists overlap in naming
convention but not in membership — `go` is a client generator, `go-server` is a
server generator. Never guess a name; always match one from the array.

## 2. Options for a specific generator

```
GET https://api.openapi-generator.tech/api/gen/clients/{language}    # getClientOptions
GET https://api.openapi-generator.tech/api/gen/servers/{framework}   # getServerOptions
```

Both return an object keyed by option name, where each value is a `CliOption`:

| field | meaning |
| --- | --- |
| `opt` | option name |
| `description` | human description |
| `type` | value type |
| `default` | default value |
| `optValue` | current/assigned value |
| `enum` | permitted values, when constrained |

Feed these keys directly into the `options` map of a `GeneratorInput` when
calling `generateClient` or `generateServerForLanguage`.

## 3. What the API does not tell you

The API returns names only. It does not expose a generator's **stability index**
— stable, beta, experimental or deprecated. That lives in the docs and in the
CLI:

```
openapi-generator-cli list --include all,beta,stable,experimental,deprecated
```

`list` excludes deprecated generators by default, so the API's arrays and the
CLI's default output can legitimately differ. Check
https://openapi-generator.tech/docs/generators before adopting a generator for
production.

## Errors

A `{language}` or `{framework}` not present in the corresponding array returns
`404` with the Spring Boot default envelope
(`{timestamp,status,error,path}`) — not RFC 9457. See
`errors/openapi-generator-problem-types.yml`.
