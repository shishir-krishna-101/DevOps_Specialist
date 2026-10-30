# HTTP and REST

HTTP is one of the most important boundaries in DevOps automation. Deployment tools, cloud APIs, monitoring systems, webhooks, internal services, and many platform interfaces communicate through HTTP.

> **Source note:** Protocol behavior in this guide is based on RFC 9110 and the official Python documentation. Examples and engineering patterns are teaching demonstrations created for this repository.

---

## 1. HTTP mental model

HTTP is a stateless application-level request/response protocol.

A simplified interaction is:

```text
client
  │
  │ HTTP request
  ▼
server
  │
  │ HTTP response
  ▼
client
```

An HTTP request communicates an intention toward a target resource. The server processes the request and returns a response containing status information and, when appropriate, a representation or other response content.

For DevOps automation, think of HTTP as a boundary:

```text
configuration
     ↓
validate
     ↓
build request
     ↓
send HTTP request
     ↓
inspect response
     ↓
validate application result
     ↓
report outcome
```

Do not treat a successful network exchange as automatically meaning that the requested business operation succeeded. The response still has to be interpreted.

---

## 2. HTTP request structure

An HTTP request contains protocol information such as:

- method
- target URI/request target
- request headers
- optional content/body

Conceptually:

```text
METHOD /resource HTTP-version
Header: value
Header: value

request body
```

For example, an API client might conceptually send:

```text
GET /health HTTP/1.1
Host: example.internal
Accept: application/json
```

The exact wire representation depends on the HTTP version and implementation. The important learning point is that the method, target, metadata, and content have distinct roles.

---

## 3. HTTP response structure

A response provides information such as:

- status code
- response headers
- optional response content/body

Conceptually:

```text
HTTP-version status-code reason
Header: value
Header: value

response body
```

For automation, the status code is an important first signal, but it is not necessarily the complete application-level result.

For example, an API may return a successful HTTP status together with a response body containing fields that your automation still needs to validate.

---

## 4. HTTP methods

HTTP defines request methods with different semantics.

Common methods encountered in DevOps APIs include:

| Method | Typical purpose |
|---|---|
| `GET` | Retrieve a representation of a resource. |
| `POST` | Submit data for processing or create/trigger an operation according to the target resource's semantics. |
| `PUT` | Replace a target resource's state/representation according to the target semantics. |
| `PATCH` | Apply a partial modification according to the target resource's semantics. |
| `DELETE` | Request removal of a target resource according to the target semantics. |

The method alone does not tell you the complete behavior of a particular API. Always consult the API's authoritative documentation for the endpoint's contract.

---

## 5. Status codes

HTTP response status codes communicate the result of the HTTP request at the protocol level.

The major classes are:

| Class | Meaning |
|---|---|
| `1xx` | Informational response. |
| `2xx` | Successful response. |
| `3xx` | Redirection. |
| `4xx` | Client error. |
| `5xx` | Server error. |

Common examples include:

- `200 OK`
- `201 Created`
- `202 Accepted`
- `204 No Content`
- `400 Bad Request`
- `401 Unauthorized`
- `403 Forbidden`
- `404 Not Found`
- `409 Conflict`
- `429 Too Many Requests`
- `500 Internal Server Error`
- `502 Bad Gateway`
- `503 Service Unavailable`
- `504 Gateway Timeout`

Do not build automation around an assumed status code without checking the API's documentation. For example, an asynchronous API may legitimately return `202` while work continues after the initial request.

---

## 6. Headers

Headers carry metadata and control information.

Examples include:

```text
Accept: application/json
Content-Type: application/json
Authorization: ...
User-Agent: ...
```

`Accept` communicates desired response media types.

`Content-Type` describes the media type of content being sent or represented.

For example:

```text
Content-Type: application/json
```

can indicate that the request body contains JSON.

The exact headers required depend on the API contract.

---

## 7. JSON and HTTP

A common DevOps API pattern is JSON over HTTP:

```text
Python object
     ↓
JSON serialization
     ↓
HTTP request body
     ↓
API
     ↓
JSON response
     ↓
JSON parsing
     ↓
Python object
```

The `json` module covered earlier in this track handles JSON serialization and parsing.

For example:

```python
import json

payload = {"environment": "production"}
body = json.dumps(payload).encode("utf-8")
```

The HTTP client then sends the bytes as request content with an appropriate `Content-Type` header when the API contract specifies JSON.

---

## 8. URLs and query parameters

URLs commonly identify an HTTP target and can contain query parameters.

For example:

```text
https://api.example.com/resources?environment=production&limit=10
```

Do not construct query strings by manually concatenating arbitrary user input when proper URL encoding is required.

Python's `urllib.parse` module provides URL parsing and encoding utilities.

For example, a query mapping can be encoded with:

```python
from urllib.parse import urlencode

query = urlencode({"environment": "production", "limit": 10})
url = f"https://api.example.com/resources?{query}"
```

The exact endpoint and parameter names remain API-specific.

---

## 9. Python's standard-library HTTP client options

Python provides HTTP functionality in the standard library.

Two important modules are:

```text
urllib.request
http.client
```

`urllib.request` provides a higher-level interface for opening URLs and supports HTTP/HTTPS requests, request objects, headers, data, authentication-related handlers, redirection handling, and other opener functionality.

`http.client` provides lower-level HTTP and HTTPS client classes.

For ordinary URL-based automation, `urllib.request` is generally easier to approach first.

The Python documentation also points to the third-party Requests package as a higher-level HTTP client interface. This repository will not assume that package is installed unless a later topic explicitly introduces dependency management for it.

---

## 10. `urllib.request.urlopen()`

A basic request can be made with `urlopen()`:

```python
from urllib.request import urlopen

with urlopen("https://example.com", timeout=10) as response:
    body = response.read()
```

The response can provide information such as:

```python
response.status
response.headers
response.url
```

Use a context manager so the response resource is closed after use.

The timeout is important in automation because network operations can block.

---

## 11. Build a request with `Request`

`urllib.request.Request` represents a URL request and can specify request data, headers, and a method.

For example:

```python
import json
from urllib.request import Request, urlopen

payload = {"environment": "production"}
body = json.dumps(payload).encode("utf-8")

request = Request(
    "https://example.com/api/deployments",
    data=body,
    headers={
        "Content-Type": "application/json",
        "Accept": "application/json",
    },
    method="POST",
)

with urlopen(request, timeout=10) as response:
    result = response.read()
```

The URL and endpoint contract in this example are illustrative only. They are not claims about a real service.

---

## 12. Read and decode a response

HTTP response bodies are bytes at the lower-level interface.

If the response is known to contain UTF-8 JSON, a client can decode and parse it deliberately:

```python
import json

with urlopen(request, timeout=10) as response:
    raw_body = response.read()
    text = raw_body.decode("utf-8")
    data = json.loads(text)
```

Do not assume every response is JSON. Check the API contract and, when appropriate, inspect response metadata such as `Content-Type` before parsing.

---

## 13. HTTP errors are not the same as network errors

An HTTP server can return a response indicating an error, while a network operation can fail before a usable response is received.

Examples of different failure categories include:

```text
HTTP response received
    ↓
4xx / 5xx status

versus

No usable HTTP response
    ↓
connection / timeout / protocol failure
```

Python's `urllib.request` can raise `HTTPError` for HTTP errors and `URLError` for protocol-level URL-opening errors.

A robust automation script should distinguish these cases when its recovery behavior differs.

---

## 14. Timeouts

Network operations can block, so automation should establish explicit timeout behavior appropriate to the operation.

For example:

```python
with urlopen(url, timeout=10) as response:
    body = response.read()
```

A timeout is not the same thing as an application-level failure response.

If an operation is retryable, the retry policy should be designed separately from the timeout itself.

Do not blindly retry every timeout or every HTTP error.

---

## 15. Authentication boundary

Many APIs require authentication.

Common HTTP mechanisms include credentials or tokens conveyed through request metadata, but the exact authentication scheme is API-specific.

For example, an API may document a header such as:

```text
Authorization: Bearer <token>
```

Do not assume this scheme for an arbitrary API. Read the service's authoritative authentication documentation.

Never hard-code credentials into source code.

Keep authentication material outside source-controlled code and avoid logging secrets.

---

## 16. REST is an architectural style, not a Python library

The term REST refers to an architectural style for distributed systems. HTTP provides the protocol semantics commonly used by RESTful APIs.

A REST-style API commonly exposes resources through URLs and uses HTTP methods and representations to interact with them.

However:

```text
HTTP API ≠ automatically REST
```

An HTTP service can use HTTP without satisfying every constraint associated with REST.

For this repository, use **REST API** when discussing an HTTP API that presents a resource-oriented interface, but do not claim that every HTTP endpoint is REST merely because it uses HTTP verbs.

---

## 17. Resource-oriented thinking

When working with an API, ask:

```text
What resource am I interacting with?
What representation does the API use?
What operation does the HTTP method express?
What does the response status mean?
What does the response body mean?
```

For example, an imaginary deployment API might expose:

```text
GET    /deployments/123
POST   /deployments
DELETE /deployments/123
```

These paths are teaching examples, not verified endpoints.

The actual resource paths, methods, fields, and response contracts must come from the API documentation.

---

## 18. Validate the response

A reliable automation client should not stop at:

```python
response = urlopen(request)
```

Instead, think in stages:

```text
request sent
    ↓
HTTP response received?
    ↓
status acceptable?
    ↓
content type expected?
    ↓
body parses?
    ↓
required fields present?
    ↓
application state acceptable?
```

For example:

```python
if response.status not in {200, 201, 202}:
    raise RuntimeError(f"unexpected HTTP status: {response.status}")
```

The acceptable statuses in this example are a repository demonstration. A real API client must use the statuses documented by the target service.

---

## 19. Idempotency and retries

Retries are an important DevOps topic, but HTTP method semantics alone are not enough to determine whether an entire application operation can safely be retried.

Before retrying, consider:

- whether the request reached the server;
- whether the server performed the operation;
- whether the operation can safely be repeated;
- whether the API documents idempotency behavior;
- whether the API provides an idempotency mechanism;
- which failures are transient;
- how many attempts are appropriate;
- what backoff policy should be used.

For example, blindly repeating a `POST` after a timeout can be unsafe if the original request may already have been processed.

Retry behavior should therefore be part of the API-specific design rather than a generic wrapper around every HTTP call.

---

## 20. Pagination

APIs often limit the number of resources returned by one response.

An API may expose pagination through query parameters, response links, cursors, or another documented mechanism.

Do not assume a universal parameter such as `page` or `limit`.

The correct client logic is:

```text
request first page
      ↓
process response
      ↓
inspect documented pagination information
      ↓
request next page
      ↓
repeat until complete
```

The exact pagination contract belongs to the API being consumed.

---

## 21. Rate limits

An API may restrict request frequency.

Some APIs communicate rate limiting through response status codes and/or headers. One commonly encountered status is `429 Too Many Requests`, but the exact retry guidance is service-specific.

If an API documents a retry-after mechanism, follow that contract rather than inventing a fixed delay.

Rate limiting belongs in the design of clients that make repeated API calls.

---

## 22. TLS and HTTPS

HTTPS is HTTP carried over a TLS-protected connection.

For Python's `http.client`, `HTTPSConnection` provides HTTPS connections using SSL/TLS support from Python's `ssl` module.

Do not disable certificate or hostname verification merely to make an automation script work around a certificate problem.

Instead, investigate the certificate, trust configuration, hostname, or environment causing the failure.

The exact trust configuration depends on the environment and API deployment.

---

## 23. Proxies and environment configuration

HTTP clients may operate through proxies depending on environment configuration.

Python's `urllib.request` documents proxy handling and the use of proxy-related environment variables.

This matters in enterprise DevOps environments where outbound traffic may be routed through controlled network infrastructure.

Do not assume direct Internet connectivity from a script. Treat proxy configuration as part of the runtime environment.

---

## 24. Logging HTTP operations

HTTP automation should integrate with the logging topic covered earlier.

Useful operational events include:

```python
logger.info("requesting deployment status")
logger.info("received HTTP status %s", response.status)
logger.error("deployment API request failed")
```

Avoid logging:

- access tokens;
- passwords;
- authorization headers;
- private keys;
- complete secret-bearing request bodies;
- sensitive response bodies.

Log enough information to diagnose the operation without turning the log into a credential store.

---

## 25. A small API-client boundary

For a reusable DevOps tool, keep HTTP mechanics separate from business logic.

For example:

```python
class DeploymentClient:
    def __init__(self, base_url, token):
        self.base_url = base_url
        self.token = token

    def get_status(self, deployment_id):
        # Build request
        # Send request
        # Parse response
        # Validate API contract
        # Return application data
        pass
```

The class above is a design demonstration, not a claim that every API client should use a class.

The important boundary is:

```text
HTTP transport
      ↓
API contract parsing
      ↓
application/domain logic
```

This makes testing and error handling easier to reason about.

---

## 26. Testing HTTP automation

HTTP code should be tested without requiring a real production service for every unit test.

Useful test boundaries include:

- request construction;
- URL/query encoding;
- headers;
- JSON serialization;
- status handling;
- response parsing;
- malformed responses;
- timeout handling;
- authentication failures;
- retry decisions.

The exact mocking approach can be introduced with the `unittest.mock` topic already covered in Complete Python.

Integration tests can then verify behavior against a controlled test service when one is available.

---

## 27. Common mistakes

### Mistake 1: treating every HTTP response as success

Always interpret the response status and API-specific response contract.

### Mistake 2: assuming every API is REST

HTTP and REST are related but not synonymous.

### Mistake 3: hard-coding API endpoints from examples

Endpoint paths and fields are API-specific. Verify them from authoritative documentation.

### Mistake 4: retrying every failure

Some failures are permanent, and some operations may not be safe to repeat.

### Mistake 5: omitting timeouts

A network operation that can block indefinitely is dangerous in automation.

### Mistake 6: logging authorization headers

Treat credentials and tokens as sensitive data.

### Mistake 7: assuming every response is JSON

Check the documented response contract and content metadata when appropriate.

### Mistake 8: disabling TLS verification to bypass errors

Investigate trust and certificate configuration instead.

### Mistake 9: inventing pagination parameters

Use the API's documented pagination mechanism.

### Mistake 10: mixing HTTP transport with business logic

Keep request/response mechanics separated from higher-level automation decisions where practical.

---

## 28. DevOps practice exercise

Build a small Python API client for a service whose official documentation you can inspect.

Your client should:

1. load its base URL from configuration;
2. load authentication material without hard-coding it;
3. construct a request with the documented method and endpoint;
4. send the request with an explicit timeout;
5. inspect the HTTP status;
6. inspect the documented response content type;
7. parse JSON when the API returns JSON;
8. validate required response fields;
9. log useful operational events without secrets;
10. distinguish HTTP errors from network failures;
11. implement retries only where the API contract makes them appropriate;
12. expose a small application-level result to its caller.

Do not use a guessed API contract. Choose an API with authoritative documentation and implement only behavior you can verify.

---

## 29. Knowledge checks

Before moving on, you should be able to answer:

1. What is the HTTP request/response model?
2. What information does an HTTP request contain?
3. What information does an HTTP response contain?
4. What are the major HTTP status-code classes?
5. What are `GET`, `POST`, `PUT`, `PATCH`, and `DELETE` commonly used for?
6. What is the difference between `Accept` and `Content-Type`?
7. Why is JSON commonly used with HTTP APIs?
8. What does `urllib.request.urlopen()` provide?
9. Why should automation clients use timeouts?
10. What is the difference between an HTTP error response and a network failure?
11. Why is REST not synonymous with HTTP?
12. Why should retry behavior be API-specific?
13. What is the purpose of pagination?
14. Why should API clients validate response content instead of checking only the status code?
15. Why should TLS verification not simply be disabled to bypass certificate errors?
16. Why should HTTP authentication material never be logged?

---

## 30. Completion criteria

You are ready to move on when you can:

- explain the HTTP request/response model;
- distinguish methods, headers, status codes, and bodies;
- construct a basic HTTP request in Python;
- use `urllib.request` for a simple HTTP/HTTPS operation;
- parse JSON responses safely;
- handle HTTP and network failures separately;
- configure explicit timeouts;
- explain the relationship between HTTP APIs and REST;
- reason about retries and idempotency;
- understand pagination and rate-limit boundaries;
- keep authentication material out of logs and source code;
- separate transport code from application logic.

---

## 31. Verified facts vs repository design

### Verified facts

RFC 9110 defines HTTP as a stateless application-level protocol and describes request/response semantics, methods, status codes, representations, and related HTTP concepts.

The official Python documentation defines `urllib.request` for opening URLs and HTTP/HTTPS requests, including `urlopen()`, `Request`, timeout handling, response objects, and related exceptions. The official `http.client` documentation defines lower-level HTTP and HTTPS client classes.

### Repository design decisions

The following are teaching choices for this repository:

- using a request → response → validate pipeline for DevOps automation;
- teaching `urllib.request` before introducing third-party HTTP clients;
- separating transport, API-contract parsing, and application logic;
- emphasizing explicit timeouts and secret-safe logging;
- treating retries and pagination as API-specific behaviors.

These choices are not requirements imposed by HTTP or Python.

---

## 32. Authoritative references

- HTTP Semantics — RFC 9110: https://www.rfc-editor.org/rfc/rfc9110
- Python `urllib.request`: https://docs.python.org/3/library/urllib.request.html
- Python `http.client`: https://docs.python.org/3/library/http.client.html
- Python `urllib.parse`: https://docs.python.org/3/library/urllib.parse.html
- Python `ssl`: https://docs.python.org/3/library/ssl.html
- Python `json`: https://docs.python.org/3/library/json.html
- Python `logging`: https://docs.python.org/3/library/logging.html
