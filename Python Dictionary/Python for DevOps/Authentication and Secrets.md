# Authentication and Secrets

Authentication is where DevOps automation crosses from ordinary configuration into security-sensitive behavior. A script may need an API token, password, certificate, signing key, or another credential to communicate with a service.

> **Source note:** Python-specific behavior in this guide is based on the official Python documentation. Authentication schemes remain service-specific and must be verified against the target service's authoritative documentation. Examples in this guide are teaching demonstrations unless explicitly identified otherwise.

---

## 1. Authentication mental model

Separate these concerns:

```text
identity
   ↓
authentication
   ↓
authorization
   ↓
request
   ↓
service response
```

- **Authentication** establishes or proves an identity.
- **Authorization** determines what that identity is permitted to do.
- **Credential** is the material used by an authentication mechanism.
- **Secret** is sensitive information that should be protected from unauthorized disclosure.

The exact authentication protocol is defined by the service you are calling. Python does not provide one universal API-token format.

---

## 2. Common credential forms

DevOps automation can encounter credentials such as:

- API tokens
- bearer tokens
- usernames and passwords
- client certificates and private keys
- short-lived credentials
- signed requests
- service-account credentials

The correct handling depends on the service's authentication contract.

Do not infer an authentication scheme merely from the fact that an API uses HTTP.

For example, a service may document:

```text
Authorization: Bearer <token>
```

That is a service contract, not a universal requirement for every API.

---

## 3. Keep secrets outside source code

Avoid embedding credentials directly in Python source:

```python
# Bad
TOKEN = "real-secret-value"
```

Source-controlled credentials can be exposed through repository history, logs, copied code, or other distribution paths.

Prefer a runtime configuration mechanism appropriate to the environment, such as an environment variable or a dedicated secret-management system.

For example:

```python
import os

TOKEN = os.environ["API_TOKEN"]
```

The environment variable name above is a repository design example, not a standard Python variable.

---

## 4. Required versus optional credentials

A script should distinguish required authentication material from optional configuration.

For required configuration:

```python
import os

try:
    token = os.environ["API_TOKEN"]
except KeyError as exc:
    raise RuntimeError("API_TOKEN is required") from exc
```

For optional configuration:

```python
import os

token = os.getenv("API_TOKEN")
```

Do not silently continue with an empty or missing credential when authentication is required.

---

## 5. Environment variables are configuration, not a secret manager

Environment variables are useful for passing runtime configuration to a process.

Python exposes them through `os.environ` and `os.getenv()`.

However, the security properties of environment variables depend on the operating environment and process-management system.

Therefore:

```text
environment variable
        ≠
complete secret-management strategy
```

For production systems, use the secret-management facilities appropriate to the platform and deployment environment.

---

## 6. Do not log credentials

Never deliberately log values such as:

- access tokens;
- passwords;
- private keys;
- authorization headers;
- session credentials;
- secret-bearing request bodies.

Avoid patterns such as:

```python
print(token)
```

or:

```python
logger.debug("request headers: %s", headers)
```

when `headers` contains authentication material.

Prefer operational information that does not expose the credential:

```python
logger.info("calling deployment API")
```

If debugging requires inspecting a sensitive value, use a controlled mechanism that does not place the secret into normal logs.

---

## 7. Authentication headers

An HTTP API may place credentials in a request header.

For example, a documented bearer-token API might require:

```python
headers = {
    "Authorization": f"Bearer {token}",
    "Accept": "application/json",
}
```

This example demonstrates constructing a header. The authentication scheme must come from the target API's documentation.

Keep the credential-bearing value inside the narrowest possible scope.

---

## 8. `urllib.request` authentication boundary

Python's `urllib.request` supports authentication-related handlers and request headers.

For a simple token-based HTTP API, a request can be constructed with headers:

```python
from urllib.request import Request

request = Request(
    "https://example.com/api/status",
    headers={
        "Authorization": f"Bearer {token}",
        "Accept": "application/json",
    },
)
```

The endpoint and header scheme in this example are illustrative only.

The official `urllib.request` documentation also describes authentication handlers for authentication mechanisms supported by the module.

---

## 9. Basic authentication

Some services use HTTP Basic Authentication.

Do not assume that a service accepts Basic Authentication just because it uses HTTP.

When Basic Authentication is required, follow the service's documentation and Python's documented authentication facilities rather than manually inventing a protocol implementation.

Credentials used for Basic Authentication remain sensitive even though the mechanism is standardized.

Use HTTPS when credentials are transmitted over a network unless the service and security architecture explicitly establish another protected transport.

---

## 10. HTTPS and TLS

HTTPS uses TLS to protect HTTP communication.

Python's `ssl` module provides TLS and peer-authentication facilities.

The official documentation provides `ssl.create_default_context()` for creating an SSL context with secure default settings for common purposes.

For example:

```python
import ssl

context = ssl.create_default_context()
```

A custom TLS configuration should be based on the requirements of the target environment.

Do not disable certificate verification merely because a connection fails.

A certificate or hostname problem should be investigated rather than bypassed blindly.

---

## 11. Passing an SSL context to HTTP requests

`urllib.request.urlopen()` accepts an `ssl.SSLContext` through its `context` parameter.

For example:

```python
import ssl
from urllib.request import urlopen

context = ssl.create_default_context()

with urlopen("https://example.com", timeout=10, context=context) as response:
    body = response.read()
```

The URL is a teaching example.

The exact trust configuration required by an enterprise environment depends on its certificate authority and network architecture.

---

## 12. Private keys and certificates

Some authentication systems use client certificates and private keys.

Treat private keys as highly sensitive credentials.

Do not:

- commit private keys to source control;
- print private keys in logs;
- place private keys in exception messages;
- copy private keys into ordinary configuration files without an appropriate security design.

Python's `ssl` documentation provides certificate and key-loading APIs for TLS contexts. The exact certificate configuration belongs to the deployment's security architecture.

---

## 13. Secret generation with `secrets`

Python provides the `secrets` module for generating cryptographically strong random values suitable for managing secrets such as authentication tokens.

For example:

```python
import secrets

token = secrets.token_urlsafe(32)
```

The purpose of this API is important:

```text
random module
    → modelling and simulation

secrets module
    → security-sensitive random values
```

Do not use `random` when the requirement is cryptographically strong randomness.

The `secrets` documentation also provides token-generation functions such as `token_bytes()`, `token_hex()`, and `token_urlsafe()`.

---

## 14. Generated tokens versus credentials issued by a service

Generating a random token with Python does not automatically authenticate that token with an external service.

For example:

```python
new_token = secrets.token_urlsafe(32)
```

only creates a random value.

A service must explicitly support the creation and use of such a credential.

Keep these concepts separate:

```text
secure random value
        ≠
valid credential for an external API
```

---

## 15. Passwords are different from API tokens

Do not treat passwords as ordinary configuration strings.

If your application is responsible for storing passwords, use the password-storage mechanism appropriate to the application and security architecture.

The Python `secrets` documentation explicitly warns against storing passwords in recoverable form and discusses salted, one-way password hashing.

For DevOps automation, prefer short-lived or narrowly scoped credentials when the target platform supports them, according to that platform's documented security model.

---

## 16. Credential scope

A useful security principle for automation is to minimize what a credential can do.

Prefer credentials that are:

- limited to the required service;
- limited to the required operations;
- short-lived when supported;
- separately managed from source code;
- revocable;
- auditable.

These properties are security and platform design considerations rather than Python language features.

Always verify the available controls in the platform that issues the credential.

---

## 17. Authentication failures

Authentication failures should be handled differently from ordinary application failures when the recovery action differs.

For example:

```text
401 / authentication failure
        ↓
credential or authentication configuration problem

403 / authorization failure
        ↓
identity authenticated but operation not permitted
```

The exact meaning of a status code must be interpreted according to the HTTP semantics and the target API's documentation.

Do not automatically retry an authentication failure with the same credential.

A retry does not repair an invalid or expired credential unless the authentication system explicitly supports refreshing or replacing it.

---

## 18. Refreshing short-lived credentials

Some authentication systems use short-lived credentials and a separate mechanism to obtain fresh credentials.

The correct workflow is service-specific:

```text
obtain credential
      ↓
use credential
      ↓
credential expires
      ↓
refresh or obtain replacement
      ↓
continue operation
```

Do not invent refresh endpoints, token fields, or expiration semantics.

Those details must come from the authentication provider's authoritative documentation.

---

## 19. Secret-safe error handling

Exceptions can accidentally expose credentials.

Avoid constructing errors from complete requests or secret-bearing objects:

```python
raise RuntimeError(f"request failed: {request}")
```

A request object may contain sensitive headers or data depending on how it was constructed.

Prefer a safe summary:

```python
raise RuntimeError("deployment API request failed")
```

When adding diagnostic context, include identifiers that are useful for investigation without including credentials.

---

## 20. Secret-safe configuration objects

If configuration is represented as a Python dictionary, avoid treating all fields as safe to display:

```python
config = {
    "endpoint": "https://example.com",
    "token": token,
}
```

A diagnostic representation should deliberately redact sensitive fields:

```python
def redact_config(config):
    safe = dict(config)
    if "token" in safe:
        safe["token"] = "<redacted>"
    return safe
```

This is a repository design pattern, not a built-in Python security feature.

A real system may need a more comprehensive redaction policy covering passwords, keys, cookies, authorization headers, and other sensitive fields.

---

## 21. Do not confuse encoding with encryption

Operations such as:

```python
value.encode("utf-8")
```

convert text to bytes.

Base64-style representations encode data for transport or representation; they do not by themselves make the data secret.

Encryption is a separate security mechanism.

Keep these concepts distinct:

```text
encoding
   ≠
encryption
   ≠
authentication
   ≠
authorization
```

---

## 22. Authentication and configuration precedence

A practical DevOps client may combine configuration sources:

```text
built-in defaults
      ↓
configuration file
      ↓
environment
      ↓
CLI arguments
      ↓
resolved configuration
```

The exact precedence is a repository design decision unless the target platform defines one.

Authentication material should have a deliberately defined source rather than being silently selected from multiple locations.

Make the source of a credential observable without exposing the credential itself.

For example:

```python
logger.info("using API token from environment")
```

is safer than logging the token value.

---

## 23. Testing authentication logic

Authentication code should be tested without placing real production credentials into the test suite.

Useful tests include:

- missing credential;
- malformed configuration;
- request header construction;
- authentication failure;
- authorization failure;
- expired credential behavior;
- token refresh logic when supported;
- secret redaction;
- TLS configuration failures.

Use fake values for unit tests:

```python
test_token = "test-token"
```

A test credential should not be a real production credential.

The `unittest.mock` tools covered earlier can help isolate network and credential boundaries.

---

## 24. A small authenticated API boundary

Keep credential handling separate from application logic where practical:

```python
class APIClient:
    def __init__(self, base_url, token):
        self.base_url = base_url
        self.token = token

    def headers(self):
        return {
            "Authorization": f"Bearer {self.token}",
            "Accept": "application/json",
        }
```

This is a teaching design, not a universal architecture.

A stronger boundary is:

```text
credential source
      ↓
authentication configuration
      ↓
HTTP client
      ↓
API response
      ↓
application logic
```

This makes it easier to test authentication behavior without coupling every function to the environment.

---

## 25. Common mistakes

### Mistake 1: hard-coding secrets

Keep credentials out of source-controlled code.

### Mistake 2: logging request headers

Authentication headers may contain credentials.

### Mistake 3: assuming every API uses Bearer tokens

Authentication is API-specific.

### Mistake 4: disabling TLS verification

Investigate certificate and trust problems instead of blindly bypassing verification.

### Mistake 5: using `random` for security-sensitive tokens

Use the `secrets` module for cryptographically strong random values.

### Mistake 6: retrying authentication failures indefinitely

A retry does not fix an invalid credential.

### Mistake 7: storing real credentials in tests

Use test credentials or mocks.

### Mistake 8: confusing encoding with encryption

Encoding does not provide secrecy.

### Mistake 9: assuming environment variables are a complete secret-management system

Their security depends on the runtime environment.

### Mistake 10: inventing undocumented authentication behavior

Use the target service's authoritative authentication documentation.

---

## 26. Practical exercise

Build a small API-client configuration layer that:

1. reads an API endpoint from an environment variable;
2. reads a required token from an environment variable;
3. fails clearly when the token is missing;
4. constructs an authentication header without printing the token;
5. logs the endpoint but not the credential;
6. uses HTTPS for the example endpoint;
7. tests the missing-token and header-construction cases with fake credentials.

Do not use a real production credential for the exercise.

---

## 27. Knowledge checks

You should be able to answer:

1. What is the difference between authentication and authorization?
2. Why should credentials not be hard-coded in source code?
3. Why are environment variables useful but not automatically a complete secret-management solution?
4. What is the purpose of `secrets`?
5. Why should `random` not be used for security-sensitive token generation?
6. Why should authentication headers not normally be logged?
7. Why is disabling TLS certificate verification a poor general solution to certificate errors?
8. Why is a generated random token not automatically a valid API credential?
9. Why should authentication failures not always be retried?
10. Why must authentication schemes be verified against the target service's documentation?

---

## 28. Completion criteria

Consider this topic complete when you can:

- explain authentication versus authorization;
- identify common credential types;
- load required credentials without hard-coding them;
- construct documented authentication headers;
- keep credentials out of logs and exceptions;
- explain the role of HTTPS and TLS;
- use `ssl.create_default_context()` appropriately;
- explain when `secrets` is appropriate;
- distinguish encoding from encryption;
- test authentication boundaries without real production credentials;
- recognize when authentication behavior must be verified from service documentation.

---

## 29. Verified facts vs repository design

### Verified facts

- Python exposes environment variables through `os.environ` and related APIs.
- Python's `secrets` module is intended for cryptographically strong random values suitable for managing secrets.
- `urllib.request` provides URL-opening functionality and authentication-related handlers.
- `urllib.request.urlopen()` accepts an SSL context through its `context` parameter.
- Python's `ssl` module provides TLS and peer-authentication facilities.
- `ssl.create_default_context()` creates an SSL context with secure default settings for common purposes.

### Repository design decisions

- Keep credentials outside source-controlled Python code.
- Avoid logging credentials and secret-bearing request data.
- Separate credential acquisition from application logic.
- Use fake credentials for authentication unit tests.
- Prefer explicit authentication configuration and failure handling.

### Demonstrations

Examples using `API_TOKEN`, `Bearer`, `example.com`, and `APIClient` are teaching demonstrations unless the surrounding text explicitly identifies them as verified service behavior.

---

## 30. Authoritative references

- Python `secrets`: https://docs.python.org/3/library/secrets.html
- Python `os`: https://docs.python.org/3/library/os.html
- Python `urllib.request`: https://docs.python.org/3/library/urllib.request.html
- Python `ssl`: https://docs.python.org/3/library/ssl.html
- Python `urllib.parse`: https://docs.python.org/3/library/urllib.parse.html
- Python `logging`: https://docs.python.org/3/library/logging.html
- Python `unittest.mock`: https://docs.python.org/3/library/unittest.mock.html
- HTTP Semantics (RFC 9110): https://www.rfc-editor.org/rfc/rfc9110
