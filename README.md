# SDK Go SDK

A typed client library generated from the OpenAPI specification.

> Package `github.com/memos_unofficial_sdk/sdk` · Version `0.0.1` · 92 operations

## Quickstart

The example calls `IdentityProviderService_ListIdentityProviders` (GET `/api/v1/identity-providers`), a low-friction operation that requires no request arguments. Replace `https://api.example.com` with your server's URL.

```go
package main

import (
	"context"
	"fmt"

	sdk "github.com/memos_unofficial_sdk/sdk"
)

func main() {
	ctx := context.Background()
	config := sdk.ClientConfig{
		BaseURL: "https://api.example.com",
	}
	client := sdk.NewSdk(config)
	result, err := client.IdentityProviderService.ListIdentityProviders(ctx)
	if err != nil {
		panic(err)
	}
	fmt.Printf("%d identityProviders\n", len(result.IdentityProviders))
}
```

## Authentication

This build disables generated authentication. Eligible operations send no authentication credentials; use `ClientAuthConfig.Headers` only for explicit service-specific headers.

## Client behavior

- Base URL: set it in `ClientConfig` before the first request.
- Transport: net/http.
- Timeout: 30,000 ms per attempt.
- Retries: up to 3 attempts for status codes `408`, `425`, `429`, `500`, `502`, `503`, `504`, with 500–8,000 ms backoff.
- Idempotency: enabled for `POST`, `PATCH` using `Idempotency-Key`.
- Error telemetry: off. This build has no reporting endpoint and sends no error reports.

High-level operation methods return the typed response body directly. The low-level request layer returns an `SdkResponse<T>` envelope containing data, status, headers, request ID, latency, and attempt count.

## Errors and response metadata

All failure paths use a small, predictable hierarchy:

| Error | Meaning |
| --- | --- |
| `SdkValidationError` | A request argument failed an OpenAPI constraint before network I/O. |
| `SdkHttpError` | The server returned a non-2xx response. |
| `SdkNetworkError` | DNS, connection, TLS, or socket failure. |
| `SdkTimeoutError` | The configured per-attempt timeout elapsed. |

HTTP errors expose `StatusCode`, the response body and headers, plus `RequestID` when the server supplies one. Preserve the request ID in support logs; it is the fastest way to correlate a failed SDK call with server-side traces.

Match common statuses with `errors.Is(err, ErrNotFound)` against the sentinels `ErrBadRequest` (400), `ErrUnauthorized` (401), `ErrPermissionDenied` (403), `ErrNotFound` (404), `ErrConflict` (409), `ErrUnprocessableEntity` (422), `ErrRateLimit` (429), `ErrInternalServer` (any 5xx).

When the API declares a model for an error response, `DecodeErrorBody[ErrorModel](httpErr)` decodes the body into it, where `ErrorModel` is that model. The raw body stays available on the error.

## Project layout and API discovery

- Operation implementations are grouped under root-level namespace `.go` files.
- 119 component models are split by API domain under root-level `models_<domain>.go` or `models_<tag_path>_models.go` files in the same public `sdk` package.
- Component schemas can choose a nested model folder with `x-octri-sdk-tags: ["Billing/Invoices"]`; the first tag owns the model and `/` creates nesting.
- [`sdk-manifest.json`](sdk-manifest.json) is the language-neutral public API index: operations, request/response modes, model properties, enum values, and generation settings.
- Public barrel/module exports are the compatibility boundary. Import public model names from those exports; internal domain filenames may evolve without changing model names.

<!-- sdk-studio-mock-tests -->
## Local mock-server tests

Generated SDK includes schema-derived, zero-dependency mock server and network
contract suite. Node.js 20+ required. Contract probes use authored response
examples only; schema-synthesized routes remain available to the local server.

`./scripts/mock --port 4010` starts server. `./scripts/test` runs the mock contract suite, then native SDK tests. A zero-authored-example contract run succeeds with an explicit zero-test
summary; mismatches in authored examples still fail.
