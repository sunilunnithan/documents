# Python Developer Interview Guide
### CLI Tool for Postgres & Snowflake — Self-Service Platform

> Hiring for a regulated financial services environment.  
> Tool manages Postgres and Snowflake via CLI with auth, RBAC, and internal/external integrations.  
> Questions are designed for the GenAI era — testing reasoning, ownership, and system thinking over rote memorization.

---

## Interview Structure

| Phase | Questions | Focus |
|---|---|---|
| Warm-up coding challenge | Q0 | Algorithmic reasoning, complexity |
| Python core & debugging | Q1–Q5 | Language internals, AI-generated code pitfalls |
| CLI design & error handling | Q6–Q10 | Real-world CLI patterns |
| Postgres & Snowflake | Q11–Q15 | Data platform domain knowledge |
| GenAI & system design | Q16–Q20 | Agentic thinking, architecture |
| Intermediate Python | Q31–Q40 | Practical Python skills |

---

## The Coding Warm-Up: Filter Duplicates Without Collections

### The Challenge

**"Filter duplicate numbers from a list without using any collection classes (no set, dict, Counter, defaultdict, etc.)"**

---

### Solution Progression

A strong candidate works through this in stages rather than jumping straight to the optimal solution. That reasoning process is what you're evaluating.

#### Stage 1: Naive O(n²) — They Should Offer This First

```python
def filter_duplicates_naive(nums: list[int]) -> list[int]:
    result = []
    for i in range(len(nums)):
        is_duplicate = False
        for j in range(i):
            if nums[j] == nums[i]:
                is_duplicate = True
                break
        if not is_duplicate:
            result.append(nums[i])
    return result

# Test
print(filter_duplicates_naive([3, 1, 4, 1, 5, 9, 2, 6, 5, 3]))
# [3, 1, 4, 5, 9, 2, 6]
```

**Time complexity:** O(n²) — for each element, scan all previous elements  
**Space complexity:** O(n) — result list  
**When acceptable:** Small lists, memory-constrained environments

---

#### Stage 2: Sort-Based O(n log n)

```python
def filter_duplicates_sorted(nums: list[int]) -> list[int]:
    if not nums:
        return []

    sorted_nums = sorted(nums)  # O(n log n)
    result = [sorted_nums[0]]

    for i in range(1, len(sorted_nums)):
        if sorted_nums[i] != sorted_nums[i - 1]:
            result.append(sorted_nums[i])

    return result

# Test
print(filter_duplicates_sorted([3, 1, 4, 1, 5, 9, 2, 6, 5, 3]))
# [1, 2, 3, 4, 5, 6, 9]  — note: order not preserved
```

**Time complexity:** O(n log n)  
**Space complexity:** O(n)  
**Trade-off:** Order of first appearance is lost — candidate should call this out unprompted.

---

#### Stage 3: Order-Preserving Sort O(n log n)

```python
def filter_duplicates_order_preserving(nums: list[int]) -> list[int]:
    if not nums:
        return []

    indexed = sorted(range(len(nums)), key=lambda i: (nums[i], i))

    keep = [True] * len(nums)
    for j in range(1, len(indexed)):
        if nums[indexed[j]] == nums[indexed[j - 1]]:
            keep[indexed[j]] = False

    return [nums[i] for i in range(len(nums)) if keep[i]]

# Test
print(filter_duplicates_order_preserving([3, 1, 4, 1, 5, 9, 2, 6, 5, 3]))
# [3, 1, 4, 5, 9, 2, 6]  — first-appearance order preserved
```

---

#### Stage 4: Generator Version for Streaming

```python
from typing import Generator

def filter_duplicates_streaming(nums: list[int]) -> Generator[int, None, None]:
    seen = []  # plain list, no set/dict
    for num in nums:
        if num not in seen:
            seen.append(num)
            yield num

# Usage
for unique in filter_duplicates_streaming([3, 1, 4, 1, 5, 9, 2, 6, 5, 3]):
    print(unique, end=" ")
# 3 1 4 5 9 2 6
```

---

#### With `set` — The Optimal Solution

```python
def filter_duplicates_set(nums: list[int]) -> list[int]:
    seen = set()
    result = []
    for num in nums:
        if num not in seen:
            seen.add(num)
            result.append(num)
    return result

# Test
print(filter_duplicates_set([3, 1, 4, 1, 5, 9, 2, 6, 5, 3]))
# [3, 1, 4, 5, 9, 2, 6]
```

**Time complexity:** O(n) — single pass, O(1) set lookup  
**Space complexity:** O(n)

#### The One-Liner

```python
def filter_duplicates_oneliner(nums: list[int]) -> list[int]:
    return list(dict.fromkeys(nums))
```

If they offer this: acknowledge it's idiomatic, then ask them to explain what `dict.fromkeys` is doing internally and its time complexity.

---

#### Why Not a Dictionary?

`dict` is banned by the same "no collection classes" rule. But more importantly:

```python
# Works but semantically wrong
def filter_duplicates_dict(nums: list[int]) -> list[int]:
    seen = {}
    result = []
    for num in nums:
        if num not in seen:
            seen[num] = True   # value is throwaway
            result.append(num)
    return result
```

Using a `dict` with throwaway `True` values is a code smell — you're using a mapping when you only need membership. `set` is a `dict` with the value storage stripped out — same O(1) lookup, less memory, correct semantics.

---

#### Complexity Comparison

| Approach | Time | Space | Order Preserved | Notes |
|---|---|---|---|---|
| Naive nested loop | O(n²) | O(n) | Yes | No collections |
| Sort-based | O(n log n) | O(n) | No | No collections |
| Sort order-preserving | O(n log n) | O(n) | Yes | No collections |
| Generator + list scan | O(n²) | O(k) streaming | Yes | Lazy, no collections |
| `set` | O(n) | O(n) | Yes | Optimal, readable |
| `dict.fromkeys` | O(n) | O(n) | Yes | Idiomatic one-liner |

---

#### Red Flags
- Jumps straight to a solution without asking about order preservation
- Can't explain why `sorted()` is O(n log n)
- Doesn't realize `in` on a list is O(n), making their solution O(n²)
- Doesn't handle empty list or single element edge cases

#### Green Flags
- Immediately asks: *"Does order of first appearance need to be preserved?"*
- Offers multiple solutions with tradeoffs before being asked
- Naturally mentions generators when you mention large datasets
- Connects the constraint to real-world scenarios unprompted

---

## Questions 1–5: Python Core & Debugging

---

### Q1. Mutable Default Argument Bug

**"What's wrong with this code, and how would you fix it?"**

```python
def add_user_to_role(user, roles=[]):
    roles.append(user)
    return roles
```

**What a strong answer looks like:**

The default argument `roles=[]` is evaluated **once** at function definition time, not on each call. The same list is reused across all calls, silently accumulating state.

```python
# Broken behavior
add_user_to_role("alice")  # ['alice']
add_user_to_role("bob")    # ['alice', 'bob'] — expected ['bob']
add_user_to_role("carol")  # ['alice', 'bob', 'carol'] — keeps growing
```

**Realistic version with actual Postgres roles:**

```python
# Broken — same hidden list reused across every call
def assign_pg_roles(username: str, roles: list[str] = []) -> dict:
    roles.append(username)
    return {"username": username, "roles": roles}

assign_pg_roles("bob")    # looks fine first time
assign_pg_roles("carol")  # {"roles": ["bob", "carol"]} — bob bleeds into carol
```

**The fix — use `None` as sentinel:**

```python
from typing import Optional

PG_ROLES = {"db_admin", "db_writer", "db_reader"}

def assign_pg_roles(
    username: str,
    roles:    Optional[list[str]] = None
) -> dict:
    if roles is None:
        roles = ["db_reader"]   # safe default — least privilege

    invalid = set(roles) - PG_ROLES
    if invalid:
        raise ValueError(
            f"Unknown roles: {invalid}. Must be one of: {PG_ROLES}"
        )

    return {"username": username, "roles": roles}

# Now each call gets its own fresh list
assign_pg_roles("alice")                               # {"roles": ["db_reader"]}
assign_pg_roles("bob",   ["db_reader"])                # {"roles": ["db_reader"]}
assign_pg_roles("carol", ["db_reader", "db_writer"])   # {"roles": ["db_reader", "db_writer"]}
```

**The same bug in a class:**

```python
# Broken — roles list shared across ALL instances
class PgUser:
    def __init__(self, username: str, roles: list[str] = []):
        self.username = username
        self.roles    = roles

u1 = PgUser("alice")
u2 = PgUser("bob")
u1.roles.append("db_writer")
print(u2.roles)  # ["db_writer"] — bob got alice's role silently

# Fixed — dataclass with field factory
from dataclasses import dataclass, field

@dataclass
class PgUser:
    username: str
    roles:    list[str] = field(default_factory=lambda: ["db_reader"])
```

**Why it matters:** A provisioning function that silently accumulates roles could grant unintended privileges — `db_admin` leaking to a user who should only have `db_reader` is a compliance incident. No error, no warning, no stack trace — just wrong data silently written to Postgres.

**Follow-up:** *"Where else does this pattern bite you?"* — class-level mutable attributes, `lru_cache` on methods, module-level mutable config dicts mutated at import time.

---

### Q2. Generator Exhaustion

**"What does this print, and why?"**

```python
def get_schema_names(conn):
    return (row[0] for row in conn.fetchall())

schemas = get_schema_names(conn)
print(len(list(schemas)))   # first consumer
for s in schemas:           # second consumer
    print(s)
```

**What a strong answer looks like:**

The generator is exhausted after `list(schemas)`. The `for` loop iterates over an empty generator — it prints nothing, no error raised.

```python
# Fix option 1: materialize explicitly
schemas = list(get_schema_names(conn))

# Fix option 2: return a reusable iterable
def get_schema_names(conn):
    return [row[0] for row in conn.fetchall()]
```

**Why it matters:** Query result generators piped through multiple transformations (filter → format → display) are common. Silent exhaustion produces empty output with no traceback.

**Follow-up:** *"When would you deliberately keep it a generator?"* — streaming large Snowflake result sets where materializing 2M rows would OOM.

---

### Q3. Closure Capturing Loop Variable

**"What does this print?"**

```python
commands = []
for cmd in ["connect", "query", "disconnect"]:
    commands.append(lambda: print(cmd))

for c in commands:
    c()
```

**What a strong answer looks like:**

Prints `disconnect` three times. All lambdas close over the **same** variable `cmd`, which holds the last value after the loop ends.

```python
# Fix: capture by value using default argument
commands = []
for cmd in ["connect", "query", "disconnect"]:
    commands.append(lambda c=cmd: print(c))

# Or use functools.partial
from functools import partial
commands = [partial(print, cmd) for cmd in ["connect", "query", "disconnect"]]
```

**Why it matters:** Dynamically registering CLI subcommand handlers in a loop will silently wire all commands to the last registered handler.

**Follow-up:** *"Does this also happen with list comprehensions?"* — No, comprehensions have their own scope in Python 3.

---

### Q4. Context Manager for Database Connection

**"Implement a context manager that wraps a database connection with automatic rollback on exception and guaranteed close."**

**What a strong answer looks like:**

```python
from contextlib import contextmanager
import psycopg2

@contextmanager
def managed_connection(dsn: str):
    conn = psycopg2.connect(dsn)
    try:
        yield conn
        conn.commit()
    except Exception as e:
        conn.rollback()
        raise  # re-raise so caller knows something failed
    finally:
        conn.close()

# Usage
with managed_connection(DSN) as conn:
    cur = conn.cursor()
    cur.execute("INSERT INTO audit_log VALUES (%s, %s)", (user, action))
```

**Class-based form — when you need state across `__enter__` and `__exit__`:**

```python
class ManagedConnection:
    def __init__(self, dsn):
        self.dsn  = dsn
        self.conn = None

    def __enter__(self):
        self.conn = psycopg2.connect(self.dsn)
        return self.conn

    def __exit__(self, exc_type, exc_val, exc_tb):
        if exc_type:
            self.conn.rollback()
        else:
            self.conn.commit()
        self.conn.close()
        return False  # don't suppress exceptions
```

**Why it matters:** Every Postgres/Snowflake command should go through this pattern. A candidate who reaches for try/finally without context managers will produce inconsistent, leak-prone connection handling.

**Follow-up:** *"What does `return False` in `__exit__` do?"* — does not suppress the exception. `return True` would swallow it silently.

---

### Q5. Rate-Limiting Decorator

**"Write a decorator that prevents a CLI command from being called more than N times per minute."**

**What a strong answer looks like:**

```python
import time
import functools
from collections import deque

def rate_limit(max_calls: int, period: int = 60):
    def decorator(func):
        calls = deque()

        @functools.wraps(func)
        def wrapper(*args, **kwargs):
            now = time.monotonic()
            while calls and now - calls[0] > period:
                calls.popleft()
            if len(calls) >= max_calls:
                wait = period - (now - calls[0])
                raise RuntimeError(
                    f"Rate limit exceeded. Try again in {wait:.1f}s."
                )
            calls.append(now)
            return func(*args, **kwargs)

        return wrapper
    return decorator

@rate_limit(max_calls=10, period=60)
def run_snowflake_query(sql: str):
    ...
```

**Why it matters:** Snowflake credits and Postgres connection limits are real costs. Without rate limiting a developer can accidentally fire 200 queries in a loop. `functools.wraps` is required — without it, introspection and help text break.

**Follow-up:** *"This works for a single process. How would you enforce this across multiple concurrent CLI sessions?"* — Redis with a sliding window counter, or a rate-limit sidecar service.

---

## Questions 6–10: CLI Design & Error Handling

---

### Q6. Designing Pluggable Auth

**"Your CLI needs to support three auth modes: static token, SSO/OAuth, and service account key. How do you design this in Python?"**

**What a strong answer looks like:**

```python
from abc import ABC, abstractmethod
from dataclasses import dataclass

@dataclass
class AuthCredential:
    token:    str
    metadata: dict

class AuthProvider(ABC):
    @abstractmethod
    def authenticate(self) -> AuthCredential: ...

    @abstractmethod
    def refresh(self, credential: AuthCredential) -> AuthCredential: ...

class StaticTokenProvider(AuthProvider):
    def __init__(self, token: str):
        self.token = token

    def authenticate(self) -> AuthCredential:
        return AuthCredential(token=self.token, metadata={"type": "static"})

    def refresh(self, credential):
        return credential

class OAuthProvider(AuthProvider):
    def __init__(self, client_id: str, client_secret: str, idp_url: str):
        self.client_id     = client_id
        self.client_secret = client_secret
        self.idp_url       = idp_url

    def authenticate(self) -> AuthCredential:
        token = self._exchange_code_for_token()
        return AuthCredential(token=token, metadata={"type": "oauth"})

    def refresh(self, credential): ...

class ServiceAccountProvider(AuthProvider):
    def __init__(self, key_path: str):
        self.key_path = key_path

    def authenticate(self) -> AuthCredential:
        # load private key, sign JWT
        ...

# Factory
def get_auth_provider(config: dict) -> AuthProvider:
    providers = {
        "static":          StaticTokenProvider,
        "oauth":           OAuthProvider,
        "service_account": ServiceAccountProvider,
    }
    auth_type = config.get("auth_type")
    if auth_type not in providers:
        raise ValueError(f"Unknown auth type: {auth_type}")
    return providers[auth_type](**config.get("auth_config", {}))
```

**Why it matters:** Adding a fourth auth method should be a new class, not a change to existing logic. Open/closed principle — critical for a tool that evolves across teams.

**Follow-up:** *"How do you store and cache the credential between CLI invocations?"* — OS keychain (`keyring` library), token file with `0600` permissions, or short-lived credential with refresh. Red flag: storing tokens in plain config files.

---

### Q7. Actionable Error Messages

**"A developer runs `db user-grant --env prod --role analyst` and gets a permission error. How does your error handling make this debuggable?"**

**What a strong answer looks like:**

```python
class RBACError(Exception):
    def __init__(self, message: str, user: str, role: str, env: str):
        self.user = user
        self.role = role
        self.env  = env
        super().__init__(message)

class InsufficientPrivilegeError(RBACError): ...
class RoleNotFoundError(RBACError): ...

def handle_rbac_error(e: RBACError, verbose: bool = False):
    print(f"[ERROR] Cannot grant role '{e.role}' to '{e.user}' in {e.env}.")
    print(f"        Reason: {e}")
    print(f"        Run with --verbose for diagnostic details.")
    print(f"        Contact: #platform-eng or run `db support-bundle`")

    if verbose:
        import traceback
        traceback.print_exc()
        print(f"        Auth context: {get_current_auth_context()}")

import sys
EXIT_CODES = {
    "permission_denied": 3,
    "not_found":         4,
    "timeout":           5,
    "unknown":           1,
}
sys.exit(EXIT_CODES["permission_denied"])
```

**Why it matters:** A raw psycopg2 or Snowflake exception dumped to the terminal is useless and potentially leaks schema/credential info. Good error design is what makes a self-service tool actually self-service.

**Follow-up:** *"How would you build a `db support-bundle` command?"* — collects sanitized logs, redacted config, last N commands, zips them for the platform team.

---

### Q8. CLI Versioning and Backward Compatibility

**"Your CLI is at v1.3. You need to rename `db grant-role` to `db role grant` in v2.0. How do you handle this without breaking existing scripts?"**

**What a strong answer looks like:**

```python
import typer

app     = typer.Typer()
role_app = typer.Typer()
app.add_typer(role_app, name="role")

# New canonical command
@role_app.command("grant")
def role_grant(
    user: str = typer.Option(...),
    role: str = typer.Option(...),
    env:  str = typer.Option(...)
):
    _do_grant(user, role, env)

# Deprecated alias — kept alive but noisy
@app.command("grant-role", hidden=True, deprecated=True)
def grant_role_deprecated(
    user: str = typer.Option(...),
    role: str = typer.Option(...),
    env:  str = typer.Option(...)
):
    import sys
    print(
        "[DEPRECATED] `db grant-role` will be removed in v3.0. "
        "Use `db role grant` instead.",
        file=sys.stderr
    )
    _do_grant(user, role, env)

def _do_grant(user, role, env):
    ...  # shared implementation — not duplicated
```

**Why it matters:** Developer tools that break silently destroy trust. A candidate who just says "update the docs" has never maintained a CLI used by more than five people.

**Follow-up:** *"How would you version the config file schema?"* — `version` field in config, migration function per version bump, never silently ignore unknown fields.

---

### Q9. `asyncio` vs Threading vs Multiprocessing

**"Your CLI runs the same query across 12 Snowflake environments simultaneously. Which concurrency model do you use and why?"**

**What a strong answer looks like:**

For I/O-bound work waiting on network responses — `asyncio` is correct:

```python
import asyncio

async def query_environment(env: str, sql: str) -> dict:
    loop   = asyncio.get_event_loop()
    result = await loop.run_in_executor(None, _sync_query, env, sql)
    return {"env": env, "result": result}

async def query_all_environments(envs: list[str], sql: str):
    tasks   = [query_environment(env, sql) for env in envs]
    results = await asyncio.gather(*tasks, return_exceptions=True)
    return results

def run_cross_env_query(sql: str, envs: list[str]):
    results = asyncio.run(query_all_environments(envs, sql))
    for r in results:
        if isinstance(r, Exception):
            print(f"[FAILED] {r}")
        else:
            print(f"[OK] {r['env']}: {r['result']}")
```

| Model | Use when | Avoid when |
|---|---|---|
| `asyncio` | Many I/O waits, single thread | CPU-heavy work, sync-only libraries |
| Threading | Sync I/O libraries, simple parallelism | GIL limits CPU work |
| Multiprocessing | CPU-bound (parsing, compression) | High memory overhead, simple I/O |

**Follow-up:** *"The Snowflake connector is synchronous. Does that change your answer?"* — `run_in_executor` wraps sync calls in a thread pool while asyncio orchestrates. Tests whether they understand the boundary between the two models.

---

### Q10. Bulk Insert: Right vs Wrong

**"A developer inserts 50,000 rows into Postgres via your CLI. Walk through the worst way, okay way, and right way."**

**What a strong answer looks like:**

```python
import psycopg2
from psycopg2.extras import execute_values

# WORST: one round-trip per row — 50,000 network calls
for row in data:
    cur.execute("INSERT INTO table VALUES (%s, %s)", row)

# OKAY: executemany — still row-by-row under the hood
cur.executemany("INSERT INTO table VALUES (%s, %s)", data)

# BETTER: execute_values — single multi-row INSERT
execute_values(
    cur,
    "INSERT INTO table (col1, col2) VALUES %s",
    data,
    page_size=1000
)

# BEST for large volumes: COPY protocol — bypasses SQL parser
import io
buffer = io.StringIO()
for row in data:
    buffer.write(f"{row[0]}\t{row[1]}\n")
buffer.seek(0)
cur.copy_from(buffer, "table", columns=("col1", "col2"))
```

| Method | Performance | Notes |
|---|---|---|
| Single `execute` | Catastrophic | 50,000 round trips |
| `executemany` | Marginal | Same network overhead |
| `execute_values` | 10-50x faster | Single statement per page |
| `COPY` | Fastest | Bypasses WAL for unlogged tables |

**Follow-up:** *"How does this change for Snowflake?"* — staged file uploads via `PUT` + `COPY INTO`, not row-level inserts.

---

## Questions 11–15: Postgres, Snowflake & Data Platform

---

### Q11. Connection Pooling in a CLI Context

**"How would you implement connection pooling for your CLI tool that talks to Postgres?"**

**What a strong answer looks like:**

This is a trap question. A strong candidate flags the problem immediately:

> CLI processes are short-lived. A connection pool lives in process memory. When the CLI exits, the pool is destroyed. You get zero reuse benefit across invocations.

```python
# For standard CLI invocations: just use a context manager
with managed_connection(DSN) as conn:
    run_query(conn, sql)

# For interactive/long-running mode only
from psycopg2 import pool

class ConnectionManager:
    _pool = None

    @classmethod
    def initialize(cls, dsn: str, min_conn: int = 2, max_conn: int = 10):
        cls._pool = pool.ThreadedConnectionPool(min_conn, max_conn, dsn)

    @classmethod
    def get_connection(cls):
        return cls._pool.getconn()

    @classmethod
    def release_connection(cls, conn):
        cls._pool.putconn(conn)
```

**The real production answer — pgBouncer:**

```
CLI → pgBouncer (pool of 20) → Postgres (max_connections = 100)

pgBouncer in transaction mode sits between CLI and Postgres,
multiplexing hundreds of short-lived CLI connections onto a small
number of actual server connections.
```

**Follow-up:** *"How does this change for Snowflake?"* — Snowflake charges per compute credit, not per connection. Concern shifts to warehouse auto-suspend tuning and query result caching.

---

### Q12. Postgres RBAC vs Snowflake RBAC

**"Explain the key differences between Postgres and Snowflake's role models, and how that changes how you'd implement a `db role grant` command."**

**What a strong answer looks like:**

```sql
-- Postgres: roles ARE users — no hard distinction
CREATE ROLE db_reader;
CREATE ROLE alice WITH LOGIN;
GRANT db_reader TO alice;
GRANT SELECT ON TABLE orders TO db_reader;

-- Snowflake: strict separation of USERS and ROLES
CREATE USER alice DEFAULT_ROLE=analyst;
CREATE ROLE analyst;
GRANT ROLE analyst TO USER alice;
-- Requires grants at every layer:
GRANT USAGE ON DATABASE mydb TO ROLE analyst;
GRANT USAGE ON SCHEMA mydb.public TO ROLE analyst;
GRANT SELECT ON TABLE mydb.public.orders TO ROLE analyst;
```

```python
class RoleGrantCommand:
    def _grant_postgres(self, user, role, env):
        # Single grant covers object privileges through role membership
        self._execute(f"GRANT {role} TO {user};", env)

    def _grant_snowflake(self, user, role, env):
        # Must grant at every layer of the object hierarchy
        statements = [
            f"GRANT ROLE {role} TO USER {user};",
            f"GRANT USAGE ON DATABASE {env}_db TO ROLE {role};",
            f"GRANT USAGE ON SCHEMA {env}_db.public TO ROLE {role};",
        ]
        for sql in statements:
            self._execute(sql, env)
```

**Follow-up:** *"A developer says their Snowflake query fails with 'insufficient privileges' but they were just granted the analyst role. What do you check first?"* — active role. Snowflake users must explicitly `USE ROLE analyst` or set it as default.

---

### Q13. Detecting and Preventing SQL Injection in a CLI

**"Your CLI accepts a `--filter` argument interpolated into a SQL query. How do you make this safe?"**

**What a strong answer looks like:**

```python
# DANGEROUS: string interpolation
sql = f"SELECT * FROM {table} WHERE status = '{filter_val}'"
# filter_val = "'; DROP TABLE users; --"  → catastrophic

# SAFE: parameterized queries for values
sql = "SELECT * FROM audit_log WHERE status = %s"
cur.execute(sql, (filter_val,))

# SAFE: psycopg2.sql for identifiers
from psycopg2 import sql as psql

ALLOWED_TABLES = {"audit_log", "user_grants", "role_assignments"}

def safe_identifier_query(table: str, filter_val: str, conn):
    if table not in ALLOWED_TABLES:
        raise ValueError(f"Table '{table}' is not queryable via CLI")

    query = psql.SQL("SELECT * FROM {} WHERE status = %s").format(
        psql.Identifier(table)
    )
    cur = conn.cursor()
    cur.execute(query, (filter_val,))
    return cur.fetchall()

# Whitelist of allowed operations
ALLOWED_OPERATIONS = {
    "select": {"audit_log", "user_grants", "role_assignments"},
    "insert": {"audit_log"},
    "update": set(),
    "delete": set(),
}
```

**Follow-up:** *"Does parameterization protect you against all injection vectors?"* — No. It protects values but not identifiers. Always whitelist table and column names.

---

### Q14. Audit Logging Every CLI Command

**"How would you design audit logging for every command run through your CLI, in a way that satisfies a compliance team?"**

**What a strong answer looks like:**

```python
import json, time, uuid, getpass, platform
from functools import wraps
from datetime import datetime, timezone

def audit_log(action: str):
    def decorator(func):
        @wraps(func)
        def wrapper(*args, **kwargs):
            event = {
                "event_id":  str(uuid.uuid4()),
                "timestamp": datetime.now(timezone.utc).isoformat(),
                "action":    action,
                "user":      getpass.getuser(),
                "hostname":  platform.node(),
                "args":      _sanitize_args(kwargs),
                "status":    "started",
            }
            _write_audit_event(event)
            try:
                result = func(*args, **kwargs)
                event["status"] = "success"
                return result
            except Exception as e:
                event["status"] = "failed"
                event["error"]  = str(e)
                raise
            finally:
                _write_audit_event(event)
        return wrapper
    return decorator

def _sanitize_args(kwargs: dict) -> dict:
    REDACTED = {"password", "token", "secret", "key", "credential"}
    return {
        k: "***REDACTED***" if k.lower() in REDACTED else v
        for k, v in kwargs.items()
    }

@app.command()
@audit_log(action="role.grant")
def role_grant(user: str, role: str, env: str):
    ...
```

**What compliance actually needs beyond code:**
- **Immutability:** Append-only log, write-once storage
- **Non-repudiation:** Log authenticated identity, not just OS user
- **Retention policy:** Typically 7 years in financial services
- **Tamper evidence:** Hash chaining or immutable store

**Follow-up:** *"A developer wants the audit log entry deleted after a mistake. How do you handle this?"* — You don't delete it. You append a correction event. Audit logs are append-only.

---

### Q15. Handling Snowflake Warehouse Auto-Suspend

**"A developer complains CLI queries are slow to start. You trace it to Snowflake warehouse cold start. How do you handle this?"**

**What a strong answer looks like:**

```python
import time
import snowflake.connector

class SnowflakeConnectionManager:
    WAREHOUSE_RESUME_TIMEOUT = 30
    WAREHOUSE_RESUME_POLL    = 2

    def ensure_warehouse_active(self, warehouse: str) -> None:
        cur = self.conn.cursor()
        cur.execute(f"SHOW WAREHOUSES LIKE '{warehouse}'")
        row   = cur.fetchone()
        state = row[3]  # 'STARTED', 'SUSPENDED', 'RESUMING'

        if state == "STARTED":
            return

        print(f"[INFO] Warehouse '{warehouse}' is {state}, resuming...")
        cur.execute(f"ALTER WAREHOUSE {warehouse} RESUME IF SUSPENDED")
        self._wait_for_warehouse(warehouse, cur)

    def _wait_for_warehouse(self, warehouse: str, cur) -> None:
        start = time.monotonic()
        while True:
            cur.execute(f"SHOW WAREHOUSES LIKE '{warehouse}'")
            if cur.fetchone()[3] == "STARTED":
                print(f"[INFO] Warehouse ready ({time.monotonic()-start:.1f}s)")
                return
            if time.monotonic() - start > self.WAREHOUSE_RESUME_TIMEOUT:
                raise TimeoutError(f"Warehouse '{warehouse}' did not resume in time")
            time.sleep(self.WAREHOUSE_RESUME_POLL)

# Keep-alive for interactive mode
def keepalive_ping(conn, interval: int = 240):
    import threading
    def ping():
        while True:
            time.sleep(interval)
            conn.cursor().execute("SELECT 1")
    threading.Thread(target=ping, daemon=True).start()
```

**Follow-up:** *"How would you right-size the warehouse for CLI operations?"* — dedicated XS warehouse with aggressive auto-suspend (60s) and auto-resume, isolated from ETL workloads.

---

## Questions 16–20: GenAI, Agentic Thinking & System Design

---

### Q16. Defense-in-Depth for AI-Generated Commands

**"An AI agent integrated with your CLI auto-generates and executes `DROP TABLE prod.audit_log`. What's your defense-in-depth strategy?"**

**What a strong answer looks like:**

```python
from enum import Enum
import typer

class RiskLevel(Enum):
    LOW    = "low"
    MEDIUM = "medium"
    HIGH   = "high"
    DENIED = "denied"

COMMAND_RISK = {
    "select":   RiskLevel.LOW,
    "show":     RiskLevel.LOW,
    "insert":   RiskLevel.MEDIUM,
    "update":   RiskLevel.MEDIUM,
    "create":   RiskLevel.MEDIUM,
    "drop":     RiskLevel.HIGH,
    "delete":   RiskLevel.HIGH,
    "truncate": RiskLevel.HIGH,
}

def execute_command(sql: str, dry_run: bool = True, source: str = "human"):
    risk = classify_sql_risk(sql)

    if risk == RiskLevel.DENIED:
        raise PermissionError("Command class never permitted via CLI")

    if dry_run:
        print(f"[DRY RUN] Would execute: {sql}")
        return

    if risk == RiskLevel.HIGH and source == "human":
        typer.confirm("⚠️  HIGH RISK — are you sure?", default=False, abort=True)

    if risk == RiskLevel.HIGH and source == "agent":
        raise PermissionError(
            "Agents cannot execute HIGH risk commands. "
            "Requires human confirmation."
        )

    _execute_with_audit(sql, source=source)
```

**Full defense stack:**
1. Command classification — know the risk before touching the DB
2. Dry-run by default — agents never execute on first pass
3. Hard deny list — some operations never via CLI, period
4. Human-in-the-loop gate — HIGH risk requires interactive confirmation
5. RBAC enforcement — server-side, not just CLI-side
6. Immutable audit log — every attempt logged, success or not
7. Blast radius limits — row limits, table allowlists per role
8. Rollback capability — point-in-time recovery

**Follow-up:** *"The agent correctly identifies a table to drop — it's genuinely unused. How do you build a safe workflow?"* — soft delete: `RENAME TO _deprecated_table_20260101`, retain 30 days, then scheduled hard drop with second approval.

---

### Q17. Building a `--explain` Flag with LLM Integration

**"Design a `--explain` flag that uses an LLM to describe what a SQL query will do before execution."**

**What a strong answer looks like:**

```python
import anthropic
import typer

client = anthropic.Anthropic()

def explain_sql(sql: str, platform: str, schema_context: str = None) -> str:
    system_prompt = f"""You are a SQL explainer for a {platform} database
in a regulated financial services environment. Explain in 3-5 plain English
sentences: what data it reads/modifies, filters applied, potential impact,
and any risk flags. Be concise. Flag risks clearly."""

    user_message = f"Explain this SQL:\n\n{sql}"
    if schema_context:
        user_message += f"\n\nSchema context:\n{schema_context}"

    response = client.messages.create(
        model="claude-sonnet-4-20250514",
        max_tokens=400,
        system=system_prompt,
        messages=[{"role": "user", "content": user_message}]
    )
    return response.content[0].text

@app.command()
def query(
    sql:     str  = typer.Argument(...),
    explain: bool = typer.Option(False, "--explain"),
    execute: bool = typer.Option(False, "--execute"),
):
    if explain:
        try:
            print(explain_sql(sql, platform="snowflake"))
        except Exception as e:
            print(f"[WARN] Explain unavailable: {e}")  # graceful degradation

    if execute:
        run_query(sql)

# Cache repeated explanations
from functools import lru_cache

@lru_cache(maxsize=128)
def explain_sql_cached(sql: str, platform: str) -> str:
    return explain_sql(sql, platform)
```

**Follow-up:** *"The LLM says the query looks safe but it actually deletes 2M rows. Who is liable?"* — the LLM is an aid, not a guarantee. The CLI must still enforce its own risk classification independent of the explanation.

---

### Q18. Owning AI-Generated Code

**"You used Claude to generate 80% of the connection manager module. How do you take ownership of code you didn't write line by line?"**

**What a strong answer looks like:**

What ownership actually means:
1. I can explain every line if asked in a code review
2. I know the failure modes, not just the happy path
3. I've written tests that would catch regressions I care about
4. I've read the underlying library docs, not just the generated usage
5. I can modify it confidently without re-prompting the AI

```python
# Step 1: Read it adversarially — assume it's wrong
# What happens when conn is None?
# What happens on network timeout mid-query?
# What if credentials are rotated mid-session?

# Step 2: Write characterization tests
def test_connection_manager_closes_on_exception():
    with pytest.raises(QueryError):
        with managed_connection(TEST_DSN) as conn:
            raise QueryError("simulated failure")
    assert conn.closed == 1  # verify connection is closed

def test_rollback_on_exception():
    with managed_connection(TEST_DSN) as conn:
        cur = conn.cursor()
        cur.execute("INSERT INTO test_table VALUES (1)")
        raise Exception("simulated failure")
    # Verify row was not committed
    with managed_connection(TEST_DSN) as conn:
        cur.execute("SELECT COUNT(*) FROM test_table WHERE id = 1")
        assert cur.fetchone()[0] == 0
```

**Red flags:** "I trust it if the tests pass" — who wrote the tests? The same AI?

**Follow-up:** *"The AI-generated code has a subtle connection leak under a specific error condition. How do you find it?"* — chaos/fault injection testing, resource leak detection with `tracemalloc`, connection count monitoring via `pg_stat_activity`.

---

### Q19. Designing a `db support-bundle` Command

**"Build a `db support-bundle` command that produces everything the platform team needs to diagnose an issue."**

**What a strong answer looks like:**

```python
import os, json, zipfile, tempfile, platform
from datetime import datetime, timezone
from pathlib import Path
import typer

@app.command("support-bundle")
def support_bundle(
    output_dir:      str = typer.Option(".", "--output"),
    last_n_commands: int = typer.Option(50, "--history"),
):
    timestamp   = datetime.now(timezone.utc).strftime("%Y%m%d_%H%M%S")
    bundle_path = Path(output_dir) / f"db_support_bundle_{timestamp}.zip"

    with tempfile.TemporaryDirectory() as tmp:
        tmp = Path(tmp)

        # System info
        system_info = {
            "os":          platform.system(),
            "python":      platform.python_version(),
            "cli_version": get_cli_version(),
            "timestamp":   timestamp,
        }
        (tmp / "system_info.json").write_text(json.dumps(system_info, indent=2))

        # Sanitized config — redact all secrets
        sanitized = sanitize_config(load_config())
        (tmp / "config_sanitized.json").write_text(json.dumps(sanitized, indent=2))

        # Recent commands from audit log
        recent = get_recent_audit_events(n=last_n_commands)
        (tmp / "recent_commands.json").write_text(json.dumps(recent, indent=2))

        # Log tail
        log_path = get_log_path()
        if log_path.exists():
            lines = log_path.read_text().splitlines()[-500:]
            (tmp / "cli_log_tail.txt").write_text("\n".join(lines))

        # Connectivity checks
        (tmp / "connectivity.json").write_text(
            json.dumps(run_connectivity_checks(), indent=2)
        )

        with zipfile.ZipFile(bundle_path, "w", zipfile.ZIP_DEFLATED) as zf:
            for f in tmp.iterdir():
                zf.write(f, f.name)

    print(f"[OK] Bundle created: {bundle_path}")
    print(f"     Bundle contains NO passwords, tokens, or secrets")

def sanitize_config(config: dict) -> dict:
    SENSITIVE = {"password", "token", "secret", "key", "dsn"}
    return {
        k: "***REDACTED***" if any(s in k.lower() for s in SENSITIVE) else v
        for k, v in config.items()
    }
```

**Follow-up:** *"A developer is hesitant to send it — worried about what's in it."* — `--preview` flag that lists what would be included without creating the file, explicit confirmation listing categories, sanitization verifiable in open source code.

---

### Q20. The Full Arc: New Command From Scratch

**"Walk me through building `snowflake user-provision` from first conversation to production."**

**What a strong answer covers:**

**Phase 1 — Requirements before writing a line:**
- What's the minimum data needed? (name, email, role, warehouse?)
- Who can run this? All developers or admins only?
- Self-service or approval workflow?
- What's the rollback story if provisioning fails halfway?
- What are the audit requirements?

**Phase 2 — Interface design before implementation:**
```python
@snowflake_app.command("user-provision")
def user_provision(
    username:  str  = typer.Option(...),
    email:     str  = typer.Option(...),
    role:      str  = typer.Option(...),
    warehouse: str  = typer.Option("cli_wh"),
    env:       str  = typer.Option("dev"),
    dry_run:   bool = typer.Option(True, "--dry-run/--no-dry-run"),
    approver:  str  = typer.Option(None),
):
    ...
```

**Phase 3 — Error scenarios before happy path:**
- User already exists (idempotency needed)
- Role doesn't exist in target environment
- Provisioning succeeds but role grant fails (partial state)
- Prod requires approver but none provided

**Phase 4 — Implementation with rollback:**
```python
def provision_snowflake_user(username, email, role, warehouse, env, conn):
    steps = [
        ("create_user", lambda: _create_user(username, email, warehouse, conn)),
        ("grant_role",  lambda: _grant_role(username, role, conn)),
        ("set_default", lambda: _set_defaults(username, role, warehouse, conn)),
        ("notify_user", lambda: _send_welcome_email(username, email, env)),
    ]
    completed = []
    try:
        for step_name, step_fn in steps:
            step_fn()
            completed.append(step_name)
    except Exception as e:
        for step_name in reversed(completed):
            _rollback_step(step_name, username, conn)
        raise ProvisionError(f"Failed at '{step_name}', rolled back: {completed}") from e
```

**Phase 5 — Testing strategy:**
- Unit: each step in isolation with mocked connector
- Integration: against dev Snowflake with real provisioning
- Idempotency: run twice with same args — second run is no-op

**Follow-up:** *"The product owner asks you to skip dry-run for prod to save time. How do you respond?"* — push back with data: dry-run has caught X issues, prod errors require manual rollback averaging Y hours. If overruled, document the decision and risk explicitly.

---

## Questions 21–30: Advanced Python

---

### Q21. Dataclasses vs Pydantic vs TypedDict

**"You need to model a Snowflake connection config loaded from YAML, validated, and passed around your CLI. Which do you use and why?"**

```python
# TypedDict — type hints only, no runtime validation
# Use when: you own the data, trust the source
from typing import TypedDict
class SnowflakeConfigDict(TypedDict):
    account: str; username: str; warehouse: str

# dataclass — structured, no validation
from dataclasses import dataclass, field
@dataclass
class SnowflakeConfig:
    account:   str
    username:  str
    warehouse: str
    timeout:   int = 30
    tags:      list[str] = field(default_factory=list)

# Pydantic — validation, coercion, serialization
# Use when: loading from external sources (YAML, env vars, API)
from pydantic import BaseModel, field_validator, SecretStr, Field
class SnowflakeConfig(BaseModel):
    account:   str
    username:  str
    password:  SecretStr        # never logged or printed
    warehouse: str
    schema_:   str = Field("PUBLIC", alias="schema")
    timeout:   int = 30

    @field_validator("account")
    @classmethod
    def account_format(cls, v: str) -> str:
        if "." not in v and "-" not in v:
            raise ValueError(f"Account '{v}' looks invalid.")
        return v.lower()

    model_config = {"populate_by_name": True}
```

| | TypedDict | dataclass | Pydantic |
|---|---|---|---|
| Runtime validation | No | No | Yes |
| External input safe | No | No | Yes |
| Secret handling | No | No | SecretStr |
| Best for | Internal passing | Internal models | Config/API input |

**Follow-up:** *"What does Pydantic's `SecretStr` actually do?"* — masks value in `repr()`, `str()`, and logs. Without it, a debug log line leaks the password into audit logs — a compliance incident.

---

### Q22. Python Typing and Generics

**"Your CLI has a generic `execute` function that works for both Postgres and Snowflake and returns different result types. How do you type this properly?"**

```python
from typing import TypeVar, Protocol, overload, Literal
from dataclasses import dataclass

@dataclass
class QueryResult:
    rows: list[tuple]; columns: list[str]; rowcount: int

@dataclass
class CommandResult:
    affected_rows: int; message: str

T = TypeVar("T")

class DatabaseConnection(Protocol):
    def execute(self, sql: str, params: tuple = ()) -> QueryResult: ...
    def execute_command(self, sql: str) -> CommandResult: ...

def with_connection(conn: DatabaseConnection, fn) -> T:
    try:
        return fn(conn)
    finally:
        conn.close()

# Overloads for type-safe dispatch
@overload
def run_sql(sql: str, mode: Literal["query"])   -> QueryResult: ...
@overload
def run_sql(sql: str, mode: Literal["command"]) -> CommandResult: ...

def run_sql(sql: str, mode: str) -> QueryResult | CommandResult:
    if mode == "query":
        return _run_query(sql)
    return _run_command(sql)
```

**Follow-up:** *"What's the difference between `Protocol` and `ABC`?"* — `Protocol` is structural (duck typing), `ABC` requires explicit inheritance. `Protocol` is better here because you don't own the Snowflake or psycopg2 classes.

---

### Q23. Python Memory Management and `__slots__`

**"Your CLI processes query results with 500,000 rows mapped to Python objects. It's running OOM. How do you diagnose and fix this?"**

```python
import tracemalloc, sys
tracemalloc.start()

# Plain class — __dict__ per instance (~400 bytes each)
class QueryRow:
    def __init__(self, id, user, role, timestamp):
        self.id = id; self.user = user
        self.role = role; self.timestamp = timestamp

# 500k rows × 400 bytes = ~200MB just for dicts

# Fix 1: __slots__ — eliminates __dict__, ~40-50% less memory
class QueryRow:
    __slots__ = ("id", "user", "role", "timestamp")
    def __init__(self, id, user, role, timestamp):
        self.id = id; self.user = user
        self.role = role; self.timestamp = timestamp

# Fix 2: dataclass with slots (Python 3.10+)
from dataclasses import dataclass
@dataclass(slots=True)
class QueryRow:
    id: int; user: str; role: str; timestamp: str

# Fix 3: Generator — never materialize all rows
def stream_query_results(cur):
    for raw_row in cur:   # cursor is already an iterator
        yield QueryRow(*raw_row)

for row in stream_query_results(cur):
    process_row(row)   # peak memory is O(1), not O(n)
```

| Approach | Memory/object | Mutable |
|---|---|---|
| Plain class | ~400 bytes | Yes |
| `__slots__` | ~200 bytes | Yes |
| `namedtuple` | ~200 bytes | No |
| `dataclass(slots=True)` | ~200 bytes | Yes |
| Generator | O(1) total | N/A |

**Follow-up:** *"What's the thread safety implication of `cached_property`?"* — not thread-safe by default. Two threads can both execute the loader simultaneously. Use a lock or initialize eagerly if thread safety is required.

---

### Q24. Metaclasses and Class Decorators

**"You want every subclass of `BaseCommand` to auto-register so the CLI can discover commands dynamically. How do you implement this?"**

```python
# Option 1: __init_subclass__ — cleanest modern approach (PEP 487)
class BaseCommand:
    _registry: dict[str, type] = {}

    def __init_subclass__(cls, name: str = "", **kwargs):
        super().__init_subclass__(**kwargs)
        command_name = name or cls.__name__.lower()
        BaseCommand._registry[command_name] = cls

    def execute(self, **kwargs):
        raise NotImplementedError

class GrantRoleCommand(BaseCommand, name="role-grant"):
    def execute(self, **kwargs): ...

class RevokeRoleCommand(BaseCommand, name="role-revoke"):
    def execute(self, **kwargs): ...

# Dynamic dispatch — no if/else
def run_command(name: str, **kwargs):
    cmd_class = BaseCommand._registry.get(name)
    if not cmd_class:
        raise KeyError(f"Unknown command: {name}")
    cmd_class().execute(**kwargs)

# Option 2: Class decorator — explicit, good for opt-in
_REGISTRY: dict[str, type] = {}

def register_command(name: str):
    def decorator(cls):
        _REGISTRY[name] = cls
        return cls
    return decorator

@register_command("role-grant")
class GrantRoleCommand:
    def execute(self, **kwargs): ...
```

**Preference order:** `__init_subclass__` > class decorator > metaclass

**Follow-up:** *"What's the difference between a metaclass and `__init_subclass__`?"* — metaclass intercepts class creation machinery and can modify it; `__init_subclass__` is a hook called after creation, simpler and sufficient for registration.

---

### Q25. Descriptors and Property Internals

**"You want a `config` attribute that lazily loads from disk on first access and caches the result. Implement this properly."**

```python
# Option 1: property with manual caching
class SnowflakeClient:
    def __init__(self, config_path: str):
        self._config_path = config_path
        self._config      = None

    @property
    def config(self) -> SnowflakeConfig:
        if self._config is None:
            self._config = SnowflakeConfig.from_file(self._config_path)
        return self._config

# Option 2: cached_property — simplest for instance-level caching
from functools import cached_property

class SnowflakeClient:
    def __init__(self, config_path: str):
        self.config_path = config_path

    @cached_property
    def config(self) -> SnowflakeConfig:
        return SnowflakeConfig.from_file(self.config_path)
    # del client.config  → invalidates the cache

# Option 3: Reusable descriptor
class LazyFileLoader:
    def __set_name__(self, owner, name):
        self.private_attr = f"_lazy_{name}"

    def __init__(self, loader_fn):
        self.loader_fn = loader_fn

    def __get__(self, obj, objtype=None):
        if obj is None:
            return self
        cached = getattr(obj, self.private_attr, None)
        if cached is None:
            cached = self.loader_fn(obj)
            setattr(obj, self.private_attr, cached)
        return cached

class SnowflakeClient:
    config = LazyFileLoader(lambda self: SnowflakeConfig.from_file(self.config_path))
```

**Follow-up:** *"Why does `cached_property` work?"* — it's a non-data descriptor (only `__get__`, no `__set__`). First call executes `__get__`, stores result in instance `__dict__`. Subsequent access hits `__dict__` directly, bypassing the descriptor entirely.

---

### Q26. Exception Hierarchies and Exception Chaining

**"Design the exception hierarchy for your CLI and demonstrate proper exception chaining."**

```python
class CLIError(Exception):
    exit_code: int = 1
    def __init__(self, message: str, hint: str = ""):
        self.hint = hint
        super().__init__(message)

class ConfigError(CLIError):     exit_code = 2
class AuthError(CLIError):       exit_code = 3
class ConnectionError(CLIError): exit_code = 4
class QueryError(CLIError):      exit_code = 5
class RBACError(CLIError):       exit_code = 6

import psycopg2

def connect_postgres(dsn: str):
    try:
        return psycopg2.connect(dsn)
    except psycopg2.OperationalError as e:
        raise ConnectionError(
            message=f"Cannot connect to Postgres at {dsn}",
            hint="Check that the database is running and credentials are correct"
        ) from e   # preserves original error as __cause__

# Top-level handler
@app.command()
def query(sql: str, env: str = "dev"):
    try:
        conn    = connect_postgres(get_dsn(env))
        results = run_query(conn, sql)
        display_results(results)
    except CLIError as e:
        typer.echo(f"[ERROR] {e}", err=True)
        raise typer.Exit(code=e.exit_code)
```

**Follow-up:** *"What's the difference between `raise X from Y` and `raise X from None`?"* — `from Y` sets `__cause__`, shows full chain in traceback. `from None` suppresses the chain — useful when the original exception contains credentials that shouldn't appear in logs.

---

### Q27. Python `abc` and Interface Design

**"Design the abstraction layer for Postgres and Snowflake using ABCs."**

```python
from abc import ABC, abstractmethod
from contextlib import contextmanager

class DatabaseAdapter(ABC):

    @abstractmethod
    def connect(self) -> None: ...

    @abstractmethod
    def execute_query(self, sql: str, params: tuple = ()) -> QueryResult: ...

    @abstractmethod
    def execute_command(self, sql: str, params: tuple = ()) -> CommandResult: ...

    @abstractmethod
    def list_schemas(self) -> list[str]: ...

    @abstractmethod
    def commit(self) -> None: ...

    @abstractmethod
    def rollback(self) -> None: ...

    # Concrete shared behavior — no override needed
    @contextmanager
    def transaction(self):
        try:
            yield self
            self.commit()
        except Exception:
            self.rollback()
            raise

    # Template method — algorithm fixed, steps overridden per platform
    def provision_user(self, username: str, role: str, **kwargs) -> ProvisionResult:
        self._validate_username(username)
        self._create_user(username, **kwargs)
        self._grant_role(username, role)
        return ProvisionResult(username, role)

    def _validate_username(self, username: str) -> None:
        if not username.replace(".", "").replace("_", "").isalnum():
            raise ValueError(f"Invalid username: {username}")

    @abstractmethod
    def _create_user(self, username: str, **kwargs) -> None: ...

    @abstractmethod
    def _grant_role(self, username: str, role: str) -> None: ...

# Factory
def get_adapter(platform: str, config: dict) -> DatabaseAdapter:
    adapters = {"postgres": PostgresAdapter, "snowflake": SnowflakeAdapter}
    if platform not in adapters:
        raise ConfigError(f"Unknown platform: {platform}")
    return adapters[platform](**config)
```

**Follow-up:** *"ABCs enforce abstract methods — but when?"* — at instantiation time, not class definition. `TypeError` raised if any abstract method is unimplemented.

---

### Q28. Python `importlib` and Plugin Architecture

**"You want developers to drop a `.py` file into a `plugins/` folder and have the CLI auto-discover new commands. How do you implement this?"**

```python
import importlib.util
import inspect
from pathlib import Path

PLUGINS_DIR = Path.home() / ".db-cli" / "plugins"

def discover_plugins(plugins_dir: Path = PLUGINS_DIR) -> dict[str, type]:
    discovered = {}
    if not plugins_dir.exists():
        return discovered

    for path in plugins_dir.glob("*.py"):
        try:
            validate_plugin(path)
            module   = _load_module(f"db_cli.plugins.{path.stem}", path)
            commands = _extract_commands(module)
            discovered.update(commands)
        except Exception as e:
            print(f"[WARN] Failed to load plugin '{path.name}': {e}", file=sys.stderr)

    return discovered

def _load_module(name: str, path: Path):
    spec   = importlib.util.spec_from_file_location(name, path)
    module = importlib.util.module_from_spec(spec)
    spec.loader.exec_module(module)
    return module

def _extract_commands(module) -> dict[str, type]:
    return {
        obj.name: obj
        for _, obj in inspect.getmembers(module, inspect.isclass)
        if issubclass(obj, BaseCommand) and obj is not BaseCommand and obj.name
    }

def validate_plugin(path: Path) -> bool:
    stat = path.stat()
    if stat.st_mode & 0o002:
        raise SecurityError(f"Plugin {path} is world-writable — refusing to load")
    # Production: verify cryptographic signature
    return True
```

**Follow-up:** *"What's the security risk of auto-loading arbitrary Python files?"* — arbitrary code execution. Mitigations: cryptographic signing, allowlist of approved authors, sandboxed execution, security review gate before distribution.

---

### Q29. Testing Strategy: Mocks, Fixtures, and Fakes

**"How do you test a CLI command that talks to Snowflake without hitting the real account?"**

```python
import pytest
from unittest.mock import MagicMock
from typer.testing import CliRunner

# Strategy 1: Mock — verify specific calls
def test_role_grant_calls_correct_sql():
    mock_conn   = MagicMock()
    mock_cursor = MagicMock()
    mock_conn.cursor.return_value = mock_cursor

    grant_role(mock_conn, user="alice", role="analyst")

    mock_cursor.execute.assert_called_once_with(
        "GRANT ROLE analyst TO USER alice"
    )

# Strategy 2: Fake — real behavior, no network
class FakeSnowflakeAdapter(DatabaseAdapter):
    def __init__(self):
        self.users:   dict[str, dict]      = {}
        self.grants:  dict[str, list[str]] = {}
        self.queries: list[str]            = []

    def connect(self) -> None: pass

    def _create_user(self, username: str, **kwargs) -> None:
        if username in self.users:
            raise ValueError(f"User {username} already exists")
        self.users[username] = kwargs

    def _grant_role(self, username: str, role: str) -> None:
        self.grants.setdefault(username, []).append(role)

# Strategy 3: Pytest fixtures
@pytest.fixture
def fake_adapter():
    return FakeSnowflakeAdapter()

@pytest.fixture
def provisioned_user(fake_adapter):
    fake_adapter._create_user("alice", email="alice@co.com")
    fake_adapter._grant_role("alice", "analyst")
    return fake_adapter

def test_provision_duplicate_user_raises(provisioned_user):
    with pytest.raises(ProvisionError, match="already exists"):
        provision_user(provisioned_user, "alice", "analyst", "alice@co.com")

def test_provision_rollback_on_grant_failure(fake_adapter, monkeypatch):
    monkeypatch.setattr(fake_adapter, "_grant_role", lambda *_: (_ for _ in ()).throw(Exception()))
    with pytest.raises(ProvisionError):
        provision_user(fake_adapter, "charlie", "analyst", "c@co.com")
    assert "charlie" not in fake_adapter.users  # rollback worked
```

**Testing pyramid:**
```
       /  E2E  \       — real Snowflake dev account, few tests
      /----------\
     / Integration \   — FakeAdapter, full command flow
    /--------------\
   /   Unit Tests   \  — mocked deps, fast, many tests
```

**Follow-up:** *"When would you use a Mock over a Fake?"* — Mocks for verifying specific calls were made (interaction testing). Fakes for testing behavior and outcomes. Mock-heavy tests are brittle and test implementation rather than behavior.

---

### Q30. Python Performance Profiling

**"`db schema list` takes 8 seconds on a large Snowflake account with 2,000 schemas. How do you diagnose and fix it?"**

```python
# Step 1: Profile before optimizing
import cProfile, pstats, io

profiler = cProfile.Profile()
profiler.enable()
list_schemas(conn, account="large_account")
profiler.disable()

stream = io.StringIO()
pstats.Stats(profiler, stream=stream).sort_stats("cumulative").print_stats(20)
print(stream.getvalue())

# CULPRIT 1: N+1 query — 2000 round trips
def list_schemas_slow(conn):
    schemas = conn.execute("SHOW SCHEMAS").rows
    for schema in schemas:
        meta = conn.execute(f"DESCRIBE SCHEMA {schema.name}").rows  # ← fires per schema
    ...

# Fix: single batch query
def list_schemas_fast(conn):
    return conn.execute("""
        SELECT schema_name, created, last_altered, comment
        FROM information_schema.schemata
        ORDER BY schema_name
    """).rows

# CULPRIT 2: No pagination — blocks display until all 2000 loaded
def list_schemas_paginated(conn, page_size: int = 100):
    offset = 0
    while True:
        rows = conn.execute("""
            SELECT schema_name FROM information_schema.schemata
            ORDER BY schema_name LIMIT %s OFFSET %s
        """, (page_size, offset)).rows
        if not rows:
            break
        yield rows
        offset += page_size

# CULPRIT 3: No caching
from datetime import datetime, timedelta

class SchemaCache:
    def __init__(self, ttl_seconds: int = 300):
        self._cache:      dict[str, list]     = {}
        self._timestamps: dict[str, datetime] = {}
        self.ttl = timedelta(seconds=ttl_seconds)

    def get(self, account: str) -> list | None:
        if account not in self._cache:
            return None
        if datetime.now() - self._timestamps[account] > self.ttl:
            del self._cache[account]
            return None
        return self._cache[account]

    def set(self, account: str, schemas: list) -> None:
        self._cache[account]      = schemas
        self._timestamps[account] = datetime.now()
```

**Diagnosis framework:**
1. Measure first — cProfile, line_profiler, time segments
2. Identify pattern — N+1? No pagination? No cache?
3. Fix the right thing — don't optimize a function called once
4. Measure again — confirm the fix actually helped
5. Add a perf test — so it doesn't regress silently

**Follow-up:** *"The bottleneck is Snowflake's `SHOW SCHEMAS` itself — just slow on large accounts. What now?"* — cache aggressively with TTL, `--refresh` flag to bypass cache, background refresh so cache is always warm, document known limitation.

---

## Questions 31–40: Intermediate Python

---

### Q31. List Comprehensions vs Loops

**"Rewrite this loop as a list comprehension, and tell me when you'd avoid comprehensions."**

```python
# Original
result = []
for user in users:
    if user["active"]:
        result.append(user["email"].lower())

# List comprehension
result = [user["email"].lower() for user in users if user["active"]]

# Dict comprehension
email_to_role = {user["email"]: user["role"] for user in users if user["active"]}

# Generator — for large Snowflake result sets
active_emails = (user["email"].lower() for user in users if user["active"])
```

**When to avoid comprehensions:**

```python
# 1. Complex logic — use a helper function
def is_eligible(user: dict) -> bool:
    return user["active"] and user["role"] in ALLOWED_ROLES and not user["suspended"]

result = [transform(u["email"]) for u in users if is_eligible(u)]

# 2. Side effects — use a loop
for u in users:
    audit_log(u)   # not [audit_log(u) for u in users]

# 3. Huge datasets — use a generator
all_rows = (process(row) for row in cursor)  # lazy, constant memory
```

**Follow-up:** *"What's the difference between `[]` and `()` in a comprehension?"* — `[]` materializes a list immediately, `()` creates a lazy generator that can only be iterated once.

---

### Q32. `*args` and `**kwargs`

**"When and how do you use `*args` and `**kwargs`? Give a practical CLI example."**

```python
def run_with_retry(fn, *args, max_retries=3, **kwargs):
    """Run any database function with retry logic."""
    for attempt in range(1, max_retries + 1):
        try:
            return fn(*args, **kwargs)
        except ConnectionError as e:
            if attempt == max_retries:
                raise
            wait = 2 ** attempt
            print(f"[RETRY] Attempt {attempt} failed, retrying in {wait}s...")
            time.sleep(wait)

# Usage — any function signature works
result = run_with_retry(execute_query, conn, "SELECT 1", timeout=30)

# ** unpacking — merging configs
defaults = {"port": 5432, "timeout": 30}
overrides = {"host": "prod-db", "timeout": 60}
final = {**defaults, **overrides}
# {"port": 5432, "timeout": 60, "host": "prod-db"} — overrides wins

# ** unpacking into function call
def connect(host, port, database, user, password): ...
conn = connect(**config)
```

**Follow-up:** *"What's the full parameter order in a function signature?"* — `def fn(pos, /, normal, *args, keyword_only, **kwargs)`.

---

### Q33. File Handling and `pathlib`

**"Write a function that reads `.sql` migration files from a directory and executes them against Postgres, respecting admin/writer/reader roles."**

```python
from pathlib import Path
from enum import Enum

class PgRole(str, Enum):
    ADMIN  = "db_admin"    # DDL: CREATE, DROP, ALTER
    WRITER = "db_writer"   # DML: INSERT, UPDATE, DELETE
    READER = "db_reader"   # SELECT only

def classify_migration(sql: str) -> PgRole:
    first_token = sql.strip().split()[0].upper()
    DDL = {"CREATE", "DROP", "ALTER", "TRUNCATE", "GRANT", "REVOKE"}
    DML = {"INSERT", "UPDATE", "DELETE", "MERGE"}
    if first_token in DDL: return PgRole.ADMIN
    if first_token in DML: return PgRole.WRITER
    return PgRole.READER

def run_sql_migrations(
    migrations_dir: str | Path,
    conn,
    dry_run: bool = False
) -> list[str]:
    migrations_path = Path(migrations_dir)

    if not migrations_path.exists():
        raise FileNotFoundError(f"Not found: {migrations_path}")

    sql_files    = sorted(migrations_path.glob("*.sql"))
    current_role = get_current_pg_role(conn)
    executed     = []
    cur          = conn.cursor()

    for sql_file in sql_files:
        sql           = sql_file.read_text(encoding="utf-8").strip()
        required_role = classify_migration(sql)

        assert_role_sufficient(current_role, required_role)
        print(f"[{'DRY RUN' if dry_run else 'RUN'}] {sql_file.name} (requires: {required_role.value})")

        if not dry_run:
            try:
                cur.execute(sql)
                conn.commit()
                executed.append(sql_file.name)
            except Exception as e:
                conn.rollback()
                raise RuntimeError(f"Migration failed: {sql_file.name}") from e
        else:
            executed.append(sql_file.name)

    return executed
```

**Migration folder structure:**

```
migrations/
├── schema/   ← DDL    — needs db_admin  (CREATE TABLE, ALTER, DROP)
├── seed/     ← DML    — needs db_writer (INSERT reference data)
├── views/    ← SELECT — needs db_reader (CREATE VIEW)
└── grants/   ← GRANT  — needs db_admin
```

**Follow-up:** *"A `.sql` file has both `CREATE TABLE` and `INSERT`. How does your classifier handle it?"* — classify whole file as `ADMIN` since the highest-risk statement wins. Better: enforce single-statement-type per file as a linting rule.

---

### Q34. Environment Variables and Config Hierarchy

**"Your CLI config can come from a config file, environment variables, or CLI flags. How do you implement precedence?"**

```python
import os, yaml
from pathlib import Path
from dataclasses import dataclass
from typing import Optional

@dataclass
class CLIConfig:
    snowflake_account:  str
    snowflake_user:     str
    snowflake_password: str
    warehouse:          str = "CLI_WH"
    environment:        str = "dev"

def load_config(config_file: Optional[Path] = None, cli_overrides: dict = None) -> CLIConfig:
    """
    Precedence (highest to lowest):
    1. CLI flags        --env prod
    2. Environment vars DB_CLI_ENV=prod
    3. Config file      ~/.db-cli/config.yaml
    4. Defaults         hardcoded in CLIConfig
    """
    config = {"warehouse": "CLI_WH", "environment": "dev"}

    # Layer 2: Config file
    config_path = config_file or Path.home() / ".db-cli" / "config.yaml"
    if config_path.exists():
        config.update(yaml.safe_load(config_path.read_text()) or {})

    # Layer 3: Environment variables
    ENV_MAPPING = {
        "DB_CLI_SNOWFLAKE_ACCOUNT":  "snowflake_account",
        "DB_CLI_SNOWFLAKE_USER":     "snowflake_user",
        "DB_CLI_SNOWFLAKE_PASSWORD": "snowflake_password",
        "DB_CLI_WAREHOUSE":          "warehouse",
        "DB_CLI_ENV":                "environment",
    }
    for env_var, key in ENV_MAPPING.items():
        if value := os.environ.get(env_var):
            config[key] = value

    # Layer 4: CLI flags — highest priority
    if cli_overrides:
        config.update({k: v for k, v in cli_overrides.items() if v is not None})

    missing = [f for f in ["snowflake_account", "snowflake_user", "snowflake_password"]
               if not config.get(f)]
    if missing:
        raise ConfigError(f"Missing required config: {', '.join(missing)}")

    return CLIConfig(**config)
```

**Follow-up:** *"How do you handle secrets specifically?"* — secrets come only from environment variables or a secrets manager (Vault, AWS Secrets Manager, `keyring`). Never written to disk.

---

### Q35. String Formatting and f-strings

**"What are the different ways to format strings in Python and when would you use each?"**

```python
user = "alice"; role = "analyst"; latency = 1.2345

# f-strings — default choice, readable, fast
message = f"Granted role '{role}' to '{user}' in {latency:.2f}s"

# .format() — when template is defined separately from values
TEMPLATES = {
    "grant_success": "Granted role '{role}' to '{user}'",
    "grant_failure": "Failed to grant '{role}' to '{user}': {error}",
}
def format_message(key: str, **kwargs) -> str:
    return TEMPLATES[key].format(**kwargs)

# Template strings — for user-supplied templates (safer than f-strings)
from string import Template
msg = Template("Hello $user, your role is $role").safe_substitute(user=user, role=role)

# NEVER use f-strings or .format() for SQL — injection risk
sql    = "SELECT * FROM users WHERE username = %s"  # correct
params = (user,)
cur.execute(sql, params)

# f-string formatting features
print(f"{1234567.89:,.2f}")  # 1,234,567.89
print(f"{0.856:.1%}")        # 85.6%
print(f"{user!r}")           # 'alice'
print(f"{'left':<20}")       # left-padded
print(f"{42:05d}")           # 00042
```

**Follow-up:** *"What's the risk of f-strings in log messages at DEBUG level?"* — f-strings are evaluated eagerly even if the log level means the message is never written. Use lazy `%` formatting: `logger.debug("Result: %s", expensive_fn)` — only evaluated if DEBUG is active.

---

### Q36. Working with JSON and YAML

**"Your CLI reads a YAML config and writes a JSON audit log entry. Show me how to handle both, including edge cases."**

```python
import json, yaml
from pathlib import Path
from datetime import datetime, timezone
from typing import Any

def load_yaml_config(path: Path) -> dict:
    try:
        config = yaml.safe_load(path.read_text(encoding="utf-8"))  # safe_load always
    except FileNotFoundError:
        raise ConfigError(f"Config file not found: {path}")
    except yaml.YAMLError as e:
        raise ConfigError(f"Invalid YAML in {path}:\n{e}")

    if config is None:
        raise ConfigError(f"Config file is empty: {path}")
    return config

class AuditLogEncoder(json.JSONEncoder):
    def default(self, obj: Any) -> Any:
        if isinstance(obj, datetime): return obj.isoformat()
        if isinstance(obj, set):      return sorted(list(obj))
        if isinstance(obj, Path):     return str(obj)
        return super().default(obj)

def write_audit_event(event: dict, log_path: Path) -> None:
    event["timestamp"] = datetime.now(timezone.utc)
    log_line = json.dumps(event, cls=AuditLogEncoder, ensure_ascii=False)
    with log_path.open("a", encoding="utf-8") as f:   # append-only
        f.write(log_line + "\n")

def read_audit_log(log_path: Path, last_n: int = 50) -> list[dict]:
    if not log_path.exists():
        return []
    lines  = log_path.read_text(encoding="utf-8").splitlines()[-last_n:]
    events = []
    for i, line in enumerate(lines, 1):
        try:
            events.append(json.loads(line.strip()))
        except json.JSONDecodeError as e:
            print(f"[WARN] Skipping malformed log line {i}: {e}")
    return events

# DANGEROUS — yaml.load can execute arbitrary Python
# data = yaml.load("!!python/object/apply:os.system ['rm -rf /']", Loader=yaml.Loader)
# SAFE — always use safe_load
```

**Follow-up:** *"What's JSONL and why is it better than a JSON array for logs?"* — one JSON object per line. Append without parsing the whole file, survives partial writes, readable line-by-line. A JSON array requires reading and rewriting entirely on every append.

---

### Q37. Virtual Environments and Dependency Management

**"A developer clones your CLI repo. Walk me through how you'd make sure their dependencies are clean and isolated."**

```toml
# pyproject.toml
[project]
name            = "db-cli"
version         = "1.0.0"
requires-python = ">=3.11"
dependencies    = [
    "typer>=0.9",
    "psycopg2-binary>=2.9",
    "snowflake-connector-python>=3.0",
    "pydantic>=2.0",
    "pyyaml>=6.0",
    "rich>=13.0",
]

[project.scripts]
db = "db_cli.main:app"

[project.optional-dependencies]
dev = ["pytest>=7.0", "pytest-cov", "mypy", "ruff"]
```

```bash
# Setup
pyenv install 3.11.8
pyenv local 3.11.8               # writes .python-version

python -m venv .venv
source .venv/bin/activate

pip install -r requirements.txt  # pinned versions
# or
pip install -e ".[dev]"          # editable install with dev deps

# Generate pinned lockfile from pyproject.toml
pip-compile pyproject.toml -o requirements.txt
```

```python
# Runtime dependency check
import importlib.util, sys

def check_dependencies() -> None:
    required = {
        "psycopg2":  "psycopg2-binary",
        "snowflake": "snowflake-connector-python",
        "pydantic":  "pydantic",
    }
    missing = [pkg for mod, pkg in required.items()
               if importlib.util.find_spec(mod) is None]
    if missing:
        print(f"[ERROR] Missing: {', '.join(missing)}")
        print(f"        Run: pip install {' '.join(missing)}")
        sys.exit(1)
```

**Follow-up:** *"What's the difference between `requirements.txt` and `pyproject.toml` dependencies?"* — `pyproject.toml` declares abstract dependencies with version ranges. `requirements.txt` is a pinned lockfile for reproducibility. You need both.

---

### Q38. Logging vs Print

**"Your CLI uses `print()` everywhere. Why is that a problem and how do you fix it?"**

```python
import logging, sys
from pathlib import Path

# Why print() is a problem:
# - Can't be silenced without redirecting stdout
# - No severity levels — everything looks the same
# - No timestamps or source info
# - Mixes user-facing output with diagnostics
# - Can't route to file without losing terminal output

def setup_logging(log_level: str = "INFO", log_file: Path = None, verbose: bool = False):
    logger = logging.getLogger("db_cli")
    logger.setLevel(logging.DEBUG)

    # Console — user-facing, clean
    console = logging.StreamHandler(sys.stderr)
    console.setLevel(logging.DEBUG if verbose else logging.WARNING)
    console.setFormatter(logging.Formatter("%(levelname)s: %(message)s"))
    logger.addHandler(console)

    # File — diagnostic, full format
    if log_file:
        fh = logging.FileHandler(log_file, encoding="utf-8")
        fh.setLevel(logging.DEBUG)
        fh.setFormatter(logging.Formatter("%(asctime)s %(name)s %(levelname)s %(message)s"))
        logger.addHandler(fh)

    return logger

logger = logging.getLogger("db_cli.commands.role")

def grant_role(user: str, role: str, env: str):
    logger.debug("Starting grant_role: user=%s role=%s env=%s", user, role, env)
    try:
        _execute_grant(user, role, env)
        logger.info("Role '%s' granted to '%s' in %s", role, user, env)
    except PermissionError as e:
        logger.error("Permission denied: %s", e)
        raise

# Key separation:
# print()  → user-facing results     → stdout
# logging  → operational/diagnostic  → stderr + file
```

**Follow-up:** *"A developer pipes your CLI output through `grep`. Which messages should appear?"* — only query results on stdout. All diagnostic messages to stderr. `grep` only sees stdout, so logging doesn't interfere.

---

### Q39. Working with `subprocess`

**"Your CLI needs to call `pg_dump` to create a Postgres backup. Show me how to do this safely."**

```python
import subprocess, shutil, os
from pathlib import Path

def run_pg_dump(database: str, output_path: Path, host: str = "localhost",
                port: int = 5432, username: str = "postgres", password: str = None):

    if not shutil.which("pg_dump"):
        raise RuntimeError("pg_dump not found. Install PostgreSQL client tools.")

    # Command as list — never as a string (injection risk)
    cmd = [
        "pg_dump",
        "--host",     host,
        "--port",     str(port),
        "--username", username,
        "--file",     str(output_path),
        "--format",   "custom",
        "--no-password",
        database
    ]

    # Password via environment — never as CLI argument (visible in `ps aux`)
    env = os.environ.copy()
    if password:
        env["PGPASSWORD"] = password

    try:
        result = subprocess.run(
            cmd,
            env=env,
            capture_output=True,
            text=True,
            timeout=300,
            check=False
        )
    except subprocess.TimeoutExpired:
        raise RuntimeError(f"pg_dump timed out after 300s")

    if result.returncode != 0:
        raise RuntimeError(f"pg_dump failed (exit {result.returncode}):\n{result.stderr}")

    return result

# NEVER do this — shell injection risk
# os.system(f"pg_dump {database} > {output}")
# subprocess.run(f"pg_dump {database}", shell=True)
```

**Key rules:**
1. Always use a list, never a string with `shell=True`
2. Never build commands with string formatting
3. Pass secrets via environment variables, not CLI arguments
4. Always set a timeout
5. Always check returncode

**Follow-up:** *"What's the difference between `subprocess.run()`, `subprocess.call()`, and `subprocess.Popen()`?"* — `run()` is the modern high-level API, blocks until complete. `call()` is older equivalent. `Popen()` is low-level, non-blocking, for streaming output. New code should always use `run()`.

---

### Q40. Type Hints and `Optional`

**"Your colleague says type hints are just comments and don't do anything at runtime. How do you respond?"**

```python
# The colleague is mostly right about runtime — but misses the point.
# Type hints are enforced at development time, not runtime.
# Value: mypy catches bugs, IDE autocomplete works, code is self-documenting,
# refactoring is safer, AI tools generate better code.

from typing import Optional
from psycopg2.extensions import connection as PgConnection

# Before — what does this return? Nobody knows.
def get_user_roles(conn, username, include_inactive=None): ...

# After — self-documenting
def get_user_roles(
    conn:             PgConnection,
    username:         str,
    include_inactive: bool = False,
) -> list[str]:
    ...

# Optional — nullable return
def find_user(conn: PgConnection, username: str) -> dict | None: ...

# Union — multiple accepted types
def connect(config: dict | Path | str) -> PgConnection:
    if isinstance(config, dict):  return psycopg2.connect(**config)
    if isinstance(config, Path):  return psycopg2.connect(config.read_text().strip())
    return psycopg2.connect(config)

# Type narrowing — isinstance informs the type checker
def process_result(result: QueryResult | CommandResult) -> str:
    if isinstance(result, QueryResult):
        return f"{result.rowcount} rows returned"    # checker knows: QueryResult
    return f"{result.affected_rows} rows affected"   # checker knows: CommandResult

# Sequence vs list in signatures
def display_rows(rows: Sequence[tuple], columns: Sequence[str]) -> None: ...
# Sequence accepts list, tuple, range — be permissive about inputs
# list is specific — use for return types
```

**Follow-up:** *"What's the difference between `list[str]` and `Sequence[str]` as a parameter type?"* — `list[str]` only accepts a list. `Sequence[str]` accepts any ordered sequence. Prefer `Sequence` for inputs (Postel's Law), `list` for outputs.

---

## Scoring Rubric

| Score | Description |
|---|---|
| 5 — Exceptional | Answers unprompted, explains tradeoffs, connects to real-world consequences, raises edge cases |
| 4 — Strong | Correct answer with good explanation, handles follow-up well |
| 3 — Adequate | Correct answer, some gaps in depth or rationale |
| 2 — Weak | Partial answer, misses key aspects, struggles with follow-up |
| 1 — Poor | Incorrect or no answer |

### Suggested Interview Flow (90 minutes)

| Time | Activity |
|---|---|
| 0–15 min | Warm-up coding challenge (filter duplicates) |
| 15–30 min | Q1–Q5: Python core debugging |
| 30–50 min | Q6–Q10: CLI design |
| 50–65 min | Q11–Q13: Postgres/Snowflake domain knowledge |
| 65–80 min | Q31–Q35: Intermediate Python (pick 3) |
| 80–90 min | Q20: Full arc question + candidate questions |

### Non-Negotiable Green Flags
- Asks clarifying questions before writing code
- Volunteers tradeoffs without being prompted
- Connects technical decisions to the regulated environment context
- Demonstrates ownership mindset for AI-generated code

### Non-Negotiable Red Flags
- Can't explain complexity of their own solution
- Uses `shell=True` in subprocess without hesitation
- Says "I trust it if the tests pass" about AI-generated code
- No awareness of RBAC or least-privilege principles

---

*Generated for: Python Developer Interview — CLI Tool for Postgres & Snowflake Self-Service Platform*
