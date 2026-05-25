# Python Preferences

This file defines global Python preferences for LLM coding agents. Project-local
instructions may override it when they are more specific.

## Maintaining This File

- Keep each preference small, concrete, and easy to cite.
- Prefer stable principles over tool-specific habits that may age quickly.
- Add a short example to every numbered tenet.
- Preserve existing tenet numbers once they are referenced from code, docs, or
  review comments.
- If a tenet becomes too broad, split it and leave a short compatibility note
  under the old number.
- Give every referable section a stable ID. Keep IDs short, lowercase where
  practical, and searchable with command-f.
- Use the stable ID when referring to a rule from another file. The display
  number is only for reading order and may change.

## Reading This File

- Read this file before editing Python code unless a project gives a more
  specific Python preference file.
- Apply the numbered tenets as defaults, not as rigid laws.
- Prefer the nearest project instructions when they conflict with this file.
- When explaining a change, cite the relevant tenet only when it clarifies the
  reasoning.
- Prefer stable IDs over tenet numbers in comments, reviews, and docs.
- If a tenet does not apply cleanly, use judgment and explain the tradeoff.

## Tenets

### Tenet 1: Prefer Explicit Data Flow `[py-explicit-flow]`

Pass dependencies and inputs directly instead of hiding them in globals,
environment lookups, or module-level mutable state.

```python
# Prefer
def build_report(rows: list[Row], formatter: Formatter) -> str:
    return formatter.render(rows)

# Avoid
def build_report() -> str:
    return GLOBAL_FORMATTER.render(load_rows_from_global_path())
```

### Tenet 2: Keep Functions Focused `[py-focused-functions]`

A function should usually do one job at one level of abstraction. Split work
when naming the smaller operation makes the code easier to read.

```python
# Prefer
def active_users(users: Iterable[User]) -> list[User]:
    return [user for user in users if user.is_active]

# Avoid
def active_users_and_write_csv(users: Iterable[User], path: Path) -> None:
    ...
```

### Tenet 3: Use Types To Explain Boundaries `[py-typed-boundaries]`

Add type hints at module and function boundaries. Use domain names, dataclasses,
TypedDict, Protocol, or simple value objects when primitives become ambiguous.

```python
@dataclass(frozen=True)
class RetryPolicy:
    attempts: int
    backoff_seconds: float


def fetch_invoice(invoice_id: str, retry: RetryPolicy) -> Invoice:
    ...
```

### Tenet 4: Do Not Mix Business Logic With IO `[py-io-boundary]`

Keep decisions separate from reading files, making network calls, printing,
logging, database access, or shell execution. IO wrappers should gather inputs
and persist outputs; pure functions should decide what should happen.

```python
# Prefer
def overdue_invoices(invoices: Iterable[Invoice], today: date) -> list[Invoice]:
    return [invoice for invoice in invoices if invoice.due_date < today]


def load_and_send_reminders(path: Path, today: date) -> None:
    invoices = read_invoices(path)
    for invoice in overdue_invoices(invoices, today):
        send_reminder(invoice)
```

### Tenet 5: Handle Errors At The Right Level `[py-error-level]`

Catch exceptions where the code can add context, retry, recover, or choose a
user-facing response. Do not catch exceptions only to hide them.

```python
# Prefer
try:
    invoice = parse_invoice(raw_invoice)
except InvalidInvoice as exc:
    raise ImportError(f"Invalid invoice in {source_path}") from exc

# Avoid
try:
    invoice = parse_invoice(raw_invoice)
except Exception:
    return None
```

### Tenet 6: Make Tests Describe Behavior `[py-behavior-tests]`

Test observable behavior and meaningful edge cases. Avoid tests that only copy
the implementation structure.

```python
def test_overdue_invoices_excludes_invoice_due_today() -> None:
    today = date(2026, 5, 25)
    invoices = [Invoice("a", today), Invoice("b", today - timedelta(days=1))]

    assert overdue_invoices(invoices, today) == [invoices[1]]
```
