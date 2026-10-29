# Databases and Networking

Python becomes especially useful in DevOps when automation needs to exchange data with databases, services, APIs, and network endpoints. The goal of this topic is not to turn a DevOps engineer into a database administrator or network programmer, but to build enough understanding to write reliable automation around these boundaries.

> **Verified fact:** Python's standard library includes database access through the DB-API-oriented `sqlite3` module and networking facilities such as `socket`, `ssl`, and higher-level protocol clients. The Python documentation also provides dedicated modules for common Internet protocols and data formats.

---

## 1. Start With the Boundary

A useful mental model is:

```text
Python automation
      │
      ├── local files
      │
      ├── database
      │
      ├── network service
      │
      └── external API
```

Each boundary has different failure modes.

| Boundary | Typical concerns |
|---|---|
| Database | connections, transactions, queries, timeouts, locking, schema changes |
| Network | DNS, connection failures, latency, timeouts, TLS, retries |
| API | authentication, status codes, rate limits, payload validation |
| Local process | exit codes, signals, stdout/stderr, resource cleanup |

**Repository design decision:** Learn the Python boundary first, then learn the behavior and operational constraints of the external system being automated.

---

## 2. SQLite With `sqlite3`

Python's standard library includes `sqlite3`, which provides an interface to SQLite databases.

A minimal example:

```python
import sqlite3


connection = sqlite3.connect("inventory.db")

try:
    cursor = connection.cursor()
    cursor.execute("CREATE TABLE IF NOT EXISTS hosts (name TEXT, status TEXT)")
    connection.commit()
finally:
    connection.close()
```

For small local automation, SQLite can be useful because it stores database data in a file rather than requiring a separate database server.

> **Verified fact:** The `sqlite3` documentation describes SQLite as a lightweight disk-based database that does not require a separate server process.

---

## 3. Connections and Cursors

A database connection represents the database session. A cursor can be used to execute SQL statements and retrieve results.

```python
import sqlite3


with sqlite3.connect("inventory.db") as connection:
    cursor = connection.execute(
        "SELECT name, status FROM hosts"
    )

    for name, status in cursor:
        print(name, status)
```

The exact lifecycle and transaction behavior depend on the database driver you use. Do not assume that patterns from SQLite automatically apply to every database system.

---

## 4. Parameterized SQL

Do not construct SQL by concatenating untrusted values into query strings.

Prefer parameterized statements supported by the database interface:

```python
import sqlite3


host = "api-01"
status = "healthy"

with sqlite3.connect("inventory.db") as connection:
    connection.execute(
        "INSERT INTO hosts(name, status) VALUES (?, ?)",
        (host, status),
    )
    connection.commit()
```

This is both a correctness and security concern.

**Common mistake:**

```python
# Do not build SQL from untrusted input this way.
query = f"SELECT * FROM hosts WHERE name = '{host}'"
```

Use the parameter mechanism provided by the database driver instead.

---

## 5. Transactions

A transaction groups database operations so that they can be committed or rolled back according to the database's transaction semantics.

Conceptually:

```text
begin
  ↓
operation A
  ↓
operation B
  ↓
commit
```

If the workflow fails:

```text
begin
  ↓
operation A
  ↓
operation B fails
  ↓
rollback / failure handling
```

The exact transaction behavior differs between database systems and drivers, so verify the documentation for the database you are using.

> **Verified fact:** `sqlite3` provides transaction control through its connection interface, including `commit()` and `rollback()`.

---

## 6. Database Automation Patterns

Useful DevOps-oriented database tasks include:

- storing inventory metadata
- recording automation results
- reading deployment state
- tracking job history
- querying operational data
- generating reports from structured data

A good automation boundary is explicit:

```text
automation
    ↓
validate inputs
    ↓
open connection
    ↓
execute parameterized query
    ↓
validate result
    ↓
commit / rollback
    ↓
close resources
```

Avoid mixing unrelated business logic, SQL construction, logging, and connection management in one large function.

---

## 7. Database Safety Checklist

Before using a database from automation, identify:

- [ ] Which database engine is being used?
- [ ] Which Python driver is authoritative for that engine?
- [ ] How are connections created and closed?
- [ ] How are transactions controlled?
- [ ] Are SQL parameters bound safely?
- [ ] What happens when the database is unavailable?
- [ ] What happens if a statement partially succeeds?
- [ ] Are credentials stored outside source code?
- [ ] Are queries efficient enough for the expected data volume?
- [ ] Can the operation be retried safely?

The last question matters because retrying a read is different from retrying a write with side effects.

---

## 8. Networking Mental Model

At a high level, network communication can be viewed as:

```text
application
    ↓
protocol
    ↓
transport
    ↓
network
    ↓
remote endpoint
```

For application automation, you will commonly encounter:

- DNS name resolution
- TCP connections
- TLS-encrypted connections
- HTTP-based APIs
- sockets
- subprocesses that communicate with network tools

Python's standard library provides lower-level networking primitives as well as higher-level protocol modules.

---

## 9. Sockets

The `socket` module provides access to the operating system's socket interface.

A basic TCP client pattern is:

```python
import socket


with socket.create_connection(("example.com", 80), timeout=5) as sock:
    sock.sendall(b"GET / HTTP/1.0\\r\\nHost: example.com\\r\\n\\r\\n")
    data = sock.recv(4096)

print(data)
```

This example is intentionally low-level. In production automation, use an appropriate protocol-aware client when one is available instead of manually implementing a protocol.

> **Verified fact:** `socket.create_connection()` creates a connection to a TCP service and supports a timeout parameter.

---

## 10. DNS Is Not the Same as Connectivity

Resolving a hostname and connecting to a service are separate operations.

```text
hostname
   ↓
DNS resolution
   ↓
IP address
   ↓
TCP connection
   ↓
application protocol
```

A successful DNS lookup does not prove that:

- the service is listening
- the port is reachable
- TLS negotiation will succeed
- the application will return a healthy response

This distinction is important when designing health checks.

---

## 11. Timeouts

Network automation should avoid waiting indefinitely when an operation has a meaningful upper bound.

Possible timeout boundaries include:

- DNS resolution or connection establishment
- socket operations
- TLS operations
- HTTP requests
- database operations
- external command execution

A timeout should be treated as an operational outcome that needs handling, not simply as an exception to hide.

**Repository design decision:** Examples use short explicit timeouts to make failure boundaries visible. Production values must be selected for the actual workload.

---

## 12. TLS With `ssl`

Python's `ssl` module provides TLS-related functionality for network connections.

A common pattern is to create an SSL context and use it with a socket:

```python
import socket
import ssl


context = ssl.create_default_context()

with socket.create_connection(("example.com", 443), timeout=5) as sock:
    with context.wrap_socket(sock, server_hostname="example.com") as secure_sock:
        print(secure_sock.version())
```

The important security lesson is that certificate and hostname verification should not be casually disabled to make a connection succeed.

> **Verified fact:** `ssl.create_default_context()` creates an SSL context with secure default settings appropriate for many client-side uses.

---

## 13. HTTP APIs

DevOps automation frequently communicates through HTTP APIs.

A robust API interaction has several layers:

```text
request
  ↓
DNS / connection
  ↓
TLS
  ↓
HTTP
  ↓
status code
  ↓
response body
  ↓
validation
```

Do not treat a successful TCP connection as a successful API request. Similarly, do not treat every HTTP response as an application-level success without checking the API's documented status and payload semantics.

For production API automation, use a well-supported HTTP client appropriate to your environment and verify its current documentation rather than inventing client-specific APIs.

---

## 14. Retries and Backoff

Transient network failures can sometimes be retried, but retries are not automatically safe.

Before retrying, ask:

1. Was the operation read-only?
2. Could the server have processed the request before the client saw the failure?
3. Is the operation idempotent?
4. Does the service document retry behavior?
5. Is there a rate limit?
6. Should the retry delay increase between attempts?

A useful conceptual pattern is:

```text
attempt 1
   ↓ failure
wait
   ↓
attempt 2
   ↓ failure
wait longer
   ↓
attempt 3
```

**Operational lesson:** A retry can duplicate a side effect if the first request actually succeeded but the response was lost.

---

## 15. Connection Pooling

When automation repeatedly communicates with the same service, repeatedly establishing connections can be expensive.

Some higher-level clients provide connection pooling or persistent sessions. The exact API depends on the client library.

**Verified fact:** This repository does not prescribe a third-party HTTP client's API without verifying that client's current official documentation.

**Repository design decision:** Learn the underlying networking concepts first, then select a client library based on project requirements.

---

## 16. Concurrency and Networking

Network operations are frequently I/O-bound, which makes them a common candidate for concurrency.

For example:

```text
                ┌── service A
                │
input ──────────┼── service B
                │
                └── service C
                         ↓
                    aggregate results
```

Possible implementations include:

- threads for blocking clients
- asyncio for async-native clients
- process-based execution for workloads where process parallelism is actually needed

See **[Concurrency and Asyncio](./Concurrency%20and%20Asyncio.md)** for the concurrency model selection guide.

---

## 17. Resource Cleanup

Network and database resources should have explicit lifecycle management.

Prefer context managers when the API supports them:

```python
with sqlite3.connect("inventory.db") as connection:
    ...
```

and:

```python
with socket.create_connection(("example.com", 80), timeout=5) as sock:
    ...
```

This makes cleanup part of the program structure instead of relying on a later garbage-collection event or an accidental control-flow path.

---

## 18. Configuration and Secrets

Database and network automation commonly requires:

- hostnames
- ports
- database names
- usernames
- credentials
- API tokens
- certificate configuration
- timeouts

Do not hard-code secrets into source files.

Prefer configuration mechanisms appropriate to the environment, such as environment variables or a dedicated secret-management system.

Also avoid logging credentials, tokens, authorization headers, or sensitive connection details.

---

## 19. Observability

Network and database automation should make failures diagnosable.

Useful information can include:

- operation name
- target service or database identifier
- elapsed time
- retry attempt
- exception type
- high-level result
- correlation or request identifier when available

Avoid logging sensitive payloads merely because they are available.

A useful distinction is:

```text
DEBUG → diagnostic detail
INFO  → normal operation
WARNING → unexpected but recoverable condition
ERROR → operation failed
```

Use the logging design from the project's Python logging topic rather than scattering `print()` calls through production automation.

---

## 20. Testing Database and Network Code

External systems make tests slower and less deterministic.

Separate pure logic from external boundaries:

```text
business logic
     │
     ├── pure functions → easy unit tests
     │
     └── database/network adapter → boundary tests
```

Useful strategies include:

- unit testing parsing and transformation logic
- mocking external boundaries where appropriate
- integration testing against controlled environments
- testing timeout and failure paths
- testing malformed responses
- testing retry behavior

Do not claim that a mock proves the real service behaves the same way. Mocks test your assumptions about the boundary; integration tests verify behavior against an actual implementation.

---

## 21. Common Mistakes

### Mistake 1: Building SQL with string concatenation

Use parameterized queries.

### Mistake 2: Treating DNS success as service health

A hostname resolving does not prove application availability.

### Mistake 3: No network timeout

An automation job that can wait forever is difficult to operate safely.

### Mistake 4: Blind retries

Retries can duplicate side effects and can overload an already unhealthy service.

### Mistake 5: Disabling TLS verification

Do not weaken certificate or hostname verification simply to bypass a configuration problem.

### Mistake 6: Logging secrets

Credentials and tokens should not appear in ordinary logs.

### Mistake 7: Mixing SQL, networking, and business logic

Keep external-system adapters separate from the core decision-making logic.

### Mistake 8: Assuming one database driver works like another

Driver APIs, transaction semantics, parameter styles, and supported features can differ.

---

## 22. Practical Exercises

### Exercise 1 — SQLite inventory

Create a small SQLite database containing service names and statuses.

Requirements:

- create the table if needed
- insert records with parameterized SQL
- query records
- update a status
- handle a failed operation safely

### Exercise 2 — TCP connectivity check

Write a script that accepts a hostname and port and reports whether a TCP connection can be established within a configured timeout.

Do not call a successful TCP connection proof of application health.

### Exercise 3 — TLS inspection

Use Python's `ssl` module to establish a TLS connection to a test endpoint and inspect the negotiated connection information.

### Exercise 4 — Retry design

Create a small function that retries a simulated transient failure. Document why the operation is safe to retry and what happens after the retry limit is reached.

### Exercise 5 — Boundary separation

Design a program with three layers:

```text
CLI/configuration
       ↓
application logic
       ↓
external-system adapter
```

Keep database or network calls out of the pure application logic where practical.

---

## 23. Knowledge Check

1. What is the difference between a database connection and a cursor?
2. Why should SQL parameters be bound instead of concatenated into query strings?
3. What is a transaction?
4. Why does DNS success not prove service health?
5. What problem does a network timeout solve?
6. What is the purpose of TLS?
7. Why can blindly retrying a write be dangerous?
8. When might threads be useful for network automation?
9. Why should secrets stay out of logs and source code?
10. Why should external-system code be separated from pure business logic?

---

## 24. Completion Checklist

You are ready to move on when you can:

- [ ] connect to SQLite using `sqlite3`
- [ ] execute parameterized SQL
- [ ] explain commit and rollback at a basic level
- [ ] distinguish DNS, TCP, TLS, and application-level health
- [ ] use socket timeouts deliberately
- [ ] explain the role of TLS verification
- [ ] design retry behavior around idempotency and service limits
- [ ] separate network/database adapters from application logic
- [ ] test failure paths without requiring production systems
- [ ] explain when concurrency can help network automation

---

## 25. Verified Facts vs Repository Design

### Verified facts

The following are grounded in the official Python documentation:

- Python provides the `sqlite3` module for SQLite database access.
- Python provides networking primitives through modules including `socket` and `ssl`.
- `socket.create_connection()` supports a timeout argument.
- `ssl.create_default_context()` provides a client-oriented SSL context with secure defaults.
- Python provides standard-library support for concurrent execution and asynchronous I/O.

### Repository design decisions

- Examples favor small, deterministic automation scenarios.
- The guide emphasizes explicit timeouts and resource cleanup.
- External-system adapters are treated as separate boundaries.
- Third-party client APIs are not documented unless their authoritative documentation has been verified.

---

## References

- Python Standard Library: https://docs.python.org/3/library/
- `sqlite3`: https://docs.python.org/3/library/sqlite3.html
- `socket`: https://docs.python.org/3/library/socket.html
- `ssl`: https://docs.python.org/3/library/ssl.html
- Internet protocols and support: https://docs.python.org/3/library/internet.html
- Concurrency and Asyncio: https://docs.python.org/3/library/concurrency.html
- Testing and Mocking: https://docs.python.org/3/library/unittest.html
