# Dense JSON-Style Comments (DJSC)

**A practical standard for giving AI coding agents clear, machine-readable instructions directly in your source code.**

> Replace scattered prose comments with structured JSON that both humans and AI agents can understand.

[![GitHub stars](https://img.shields.io/github/stars/losts1/dense-json-style-comments?style=social)](https://github.com/losts1/dense-json-style-comments/stargazers)
[![Last Commit](https://img.shields.io/github/last-commit/losts1/dense-json-style-comments)](https://github.com/losts1/dense-json-style-comments/commits/main)

---

## Quick Start

Copy and adapt this pattern in your code:

```python
# {"ai":{"goal":"add JWT auth + rate limit","must":["validate token","use redis","return 401 on fail"],"avoid":["log secrets","use global vars"],"note":"keep function <25 lines"}}
def authenticate_user(token: str) -> bool:
    ...
```

## What It Is

**Dense JSON-Style Comments (DJSC)** embed structured JSON objects as code comments to convey goal, constraints, anti-patterns, and context in a single dense line. The JSON structure replaces scattered prose with machine-parseable, unambiguous directives.

Instead of five lines of vague prose:

```python
# Add JWT authentication and rate limiting
# Must validate the token, use redis for rate limit tracking
# Return 401 on failure
# Don't log secrets or use global variables
# Keep the function under 25 lines
```

One structured line:

```python
# {"ai":{"goal":"add JWT auth + rate limit","must":["validate token","use redis","return 401 on fail"],"avoid":["log secrets","use global vars"],"note":"keep function <25 lines"}}
def authenticate(token: str):
    ...
```

---

## The Schema

The canonical structure uses four top-level keys inside a `"ai"` object:

| Key | Type | Purpose | Required |
|-----|------|---------|----------|
| `goal` | string | What the code should accomplish — the intent, not the implementation | ✅ Yes |
| `must` | string[] | Hard requirements — non-negotiable constraints that MUST be satisfied | ✅ Yes |
| `avoid` | string[] | Anti-patterns and forbidden behaviors — things NOT to do | Recommended |
| `note` | string | Additional context, soft constraints, style notes, or references | Optional |

### Minimal Form

```python
# {"ai":{"goal":"parse CSV into dict list","must":["handle quoted fields","skip empty rows"]}}
```

### Full Form

```python
# {"ai":{"goal":"parse CSV into dict list","must":["handle quoted fields","skip empty rows"],"avoid":["pandas dependency","loading full file into memory"],"note":"stream rows; see RFC 4180 for edge cases"}}
```

---

## When to Use DJSC

### ✅ Use When

| Situation | Why |
|-----------|-----|
| **AI-assisted coding** | The AI can parse structured constraints directly — no guessing from prose |
| **Multiple constraints** | 3+ requirements that prose would scatter across lines |
| **Hard vs soft distinction** | `must` vs `note` makes priority explicit |
| **Anti-patterns matter** | `avoid` catches mistakes prose comments often omit or bury |
| **Config/data-adjacent code** | Parsers, validators, serializers — where structure is the domain |
| **Code review context** | Reviewers (human or AI) scan one structured line faster than five prose lines |
| **Onboarding** | New developers or AI agents get precise intent + constraints instantly |

### ❌ Don't Use When

| Situation | Why |
|-----------|-----|
| **Single obvious constraint** | `# must return int` → just write prose, DJSC adds noise |
| **Narrative explanation** | "Why this algorithm works" needs prose, not JSON |
| **TODO/FIXME markers** | Use standard `# TODO:` — tooling expects that format |
| **Complex nested constraints** | If the JSON itself needs comments, you've gone too deep |
| **Non-code artifacts** | Markdown, docs, configs with native comment syntax don't need this |

### Rule of Thumb

> If the comment would be 2+ lines of scattered prose, DJSC probably wins.
> If it's one line of obvious context, prose is fine.

---

## How to Write DJSC

### 1. Start with `goal` — Intent, Not Implementation

❌ **Bad** — describes *how*:
```python
# {"ai":{"goal":"iterate list and filter nulls then map to uppercase"}}
```

✅ **Good** — describes *what*:
```python
# {"ai":{"goal":"return uppercase non-null values from list"}}
```

The `goal` is the spec. The AI figures out the implementation.

### 2. `must` — Only Hard Requirements

Things that, if violated, make the code **wrong** — not just suboptimal.

❌ **Bad** — preferences disguised as requirements:
```python
# {"ai":{"must":["use list comprehension","use snake_case"]}}
```

✅ **Good** — verifiable constraints:
```python
# {"ai":{"must":["return empty list on null input","preserve order","handle unicode"]}}
```

Test: *Can I write a test that fails if this is violated?* If no, it belongs in `note`.

### 3. `avoid` — The Anti-Patterns

Things that are **tempting but wrong** — the traps an AI (or human) might fall into.

```python
# {"ai":{"goal":"hash password","must":["use bcrypt","cost factor ≥ 12"],"avoid":["md5","sha256 direct","plaintext logging"]}}
```

`avoid` is the guardrail. Without it, the AI may choose the path of least resistance (e.g., `hashlib.sha256(pwd).hexdigest()`) which is technically "hashing" but cryptographically wrong.

### 4. `note` — Soft Context

Everything else: style preferences, soft limits, references, rationale.

```python
# {"ai":{"note":"keep <30 lines; see OWASP password storage cheat sheet"}}
```

---

## Extended Schema (Optional Keys)

For complex tasks, extend with domain-specific keys:

| Key | Type | Purpose |
|-----|------|---------|
| `perf` | string | Performance constraints (e.g., "O(n) max", "< 50ms p99") |
| `deps` | string[] | Allowed dependencies (e.g., `["redis", "jwt"]`) |
| `returns` | string | Expected return type/shape |
| `throws` | string[] | Expected error cases |
| `refs` | string[] | References (docs, RFCs, papers) |

Example:

```python
# {"ai":{"goal":"cache user session","must":["use redis","TTL 3600s","return None on miss"],"avoid":["in-memory dict","exposing session ID"],"perf":"< 2ms p99","deps":["redis","jwt"],"returns":"dict or None","throws":["ConnectionError"],"note":"see session design doc §3"}}
```

**Cave:** Only add keys when they carry information that isn't covered by the core four. More keys ≠ better. Dense ≠ bloated.

---

## Language Adaptation

DJSC works in any language with line comments:

| Language | Syntax |
|----------|--------|
| Python | `# {"ai":{...}}` |
| JavaScript/TS | `// {"ai":{...}}` |
| Go | `// {"ai":{...}}` |
| Rust | `// {"ai":{...}}` |
| Shell | `# {"ai":{...}}` |
| Java/C/C++ | `// {"ai":{...}}` |
| Ruby | `# {"ai":{...}}` |
| Lua | `-- {"ai":{...}}` |
| HTML | `<!-- {"ai":{...}} -->` |
| CSS | `/* {"ai":{...}} */` |

For block-comment languages, the JSON goes inside the block comment syntax. The `ai` wrapper key stays — it marks this as a machine-readable directive, not a human aside.

---

## Examples by Complexity

### Simple — One Function

```python
# {"ai":{"goal":"clamp value to range","must":["return min if below","return max if above"],"avoid":["mutating input"]}}
def clamp(value: float, lo: float, hi: float) -> float:
    ...
```

### Medium — Multiple Constraints

```typescript
// {"ai":{"goal":"retry HTTP request with exponential backoff","must":["max 3 retries","jitter ±25%","throw on final failure"],"avoid":["infinite retry","retrying 4xx except 429"],"note":"use fetch API; base delay 1s"}}
async function retryFetch(url: string, opts?: RequestInit): Promise<Response> {
  ...
}
```

### Complex — Full Specification

```go
// {"ai":{"goal":"connection pool with health checks","must":["max 10 conns","evict idle > 30s","reconnect on error","thread-safe"],"avoid":["busy-wait polling","global singleton"],"perf":"get < 1ms p99","returns":"*Pool","throws":["ErrPoolExhausted","ErrConnFailed"],"note":"see internal docs pool-v2 spec"}}
type Pool struct { ... }
```

### Module-Level — Architectural Intent

```python
# {"ai":{"goal":"event sourcing for order state","must":["append-only log","idempotent handlers","event version field"],"avoid":["mutable state","direct DB updates"],"note":"events in events/ dir; projectors in projectors/; start from order_created"}}
```

---

## Anti-Patterns

### 🚫 Over-nesting

```python
# BAD — JSON harder to read than the code
# {"ai":{"goal":"x","must":[{"type":"constraint","value":"y","reason":"because z"}]}}
```

DJSC is *dense*, not *deep*. Keep values flat — strings and string arrays, not nested objects.

### 🚫 Repeating the Code

```python
# BAD — this just restates the function signature
# {"ai":{"goal":"take string token, return bool","must":["param is string","return is bool"]}}
def is_valid(token: str) -> bool:
    ...
```

DJSC should add information the code doesn't already convey.

### 🚫 Vague `must`

```python
# BAD — not testable
# {"ai":{"must":["be fast","handle errors","good code quality"]}}
```

```python
# GOOD — verifiable
# {"ai":{"must":["< 10ms p99","return empty on invalid input","no global state"]}}
```

### 🚫 Missing `avoid`

```python
# BAD — AI will pick the obvious path, which may be wrong
# {"ai":{"goal":"hash password","must":["use bcrypt"]}}
```

```python
# GOOD — explicit guardrails
# {"ai":{"goal":"hash password","must":["use bcrypt","cost ≥ 12"],"avoid":["md5","sha256","plaintext in logs"]}}
```

---

## Comparison: DJSC vs Alternatives

| Aspect | DJSC | Prose Comments | Docstrings | Type Hints |
|--------|------|---------------|------------|------------|
| Machine-parseable | ✅ | ❌ | Partial | ✅ |
| Constraints explicit | ✅ must/avoid | ❌ mixed | Partial | Types only |
| Intent separate from impl | ✅ goal | ❌ often mixed | Partial | ❌ |
| Anti-patterns first-class | ✅ avoid | ❌ easy to miss | ❌ | ❌ |
| AI-native | ✅ | ❌ | Partial | ✅ |
| Human-readable | Good | ✅ best | ✅ | Good |
| Overhead per comment | Low (1 line) | Low (1-5 lines) | Medium (3-10 lines) | Low (inline) |

DJSC doesn't replace type hints or docstrings — it layers *intent and constraints* on top. Use all three:

```python
# {"ai":{"goal":"validate JWT with rate limiting","must":["check expiry","use redis","401 on fail"],"avoid":["log token payload","global redis client"]}}
def authenticate(token: str) -> AuthResult:
    """Validate JWT and enforce per-user rate limit via Redis.

    Args:
        token: JWT string from Authorization header.

    Returns:
        AuthResult with user_id and permissions.

    Raises:
        AuthError: On invalid, expired, or rate-limited tokens.
    """
    ...
```

- **DJSC** → what to build + hard constraints + anti-patterns (for AI)
- **Docstring** → how to call it + return types (for humans + tooling)
- **Type hints** → static types (for mypy/pyright)

---

## Workflow: When AI Reads DJSC

1. **Parse** the JSON object from the comment
2. **Extract** `goal` → this is the implementation target
3. **Check** `must` → every item must be satisfied; if any is unclear, ask
4. **Check** `avoid` → these are hard boundaries; never violate
5. **Consider** `note` → soft guidance; follow unless there's a strong reason not to
6. **Implement** toward `goal` within the constraint space defined by `must` ∩ ¬`avoid`
7. **Verify** — after implementation, confirm every `must` item is satisfied and no `avoid` item is violated

---

## Quick Reference

```
# {"ai":{"goal":"<WHAT>","must":["<HARD1>","<HARD2>"],"avoid":["<TRAP1>","<TRAP2>"],"note":"<CONTEXT>"}}
```

- `goal` — intent, not implementation
- `must` — testable, non-negotiable constraints
- `avoid` — tempting-but-wrong patterns
- `note` — soft context, references, style

**Dense > verbose. Structured > scattered. Intent > implementation.**