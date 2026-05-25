# Global Python Tenets [GLOBAL-PYTHON-TENETS]
This file defines global Python coding tenets (in [TENETS] section) for LLM coding agents. Project-local instructions may override it when they are more specific.

## Using This File [USING-THIS-FILE]
This [GLOBAL-PYTHON-TENETS] file is part of a hierarchy
  1. Project-specific Python guide (if it exists) 
  2. This [GLOBAL-PYTHON-TENETS] file
  3. Latest available [Google Python Style Guide](https://google.github.io/styleguide/pyguide.html)

Always try to retrieve the latest Google Python Style Guide from the internet first. If internet access is unavailable, use the local snapshot at [google-python-style-guide-2026-05-25.html](references/google-python-style-guide-2026-05-25.html).

If there is a conflict between the above three sources, the project-specific Python guide supersedes this [GLOBAL-PYTHON-TENETS] file, which then supersedes the latest available Google Python Style Guide or, when offline, the local snapshot.

### Finding tenets [FINDING-TENETS]
- When you see a tenet citation like "Tenet X [STABLE-ID]", give precedence to the semantic text of stable ID over the number when trying to find the relevant tenent. Numbering can get messed up.
- When explaining a change, cite the relevant tenet only when it clarifies the reasoning.
- Always cite a tenet in this form: "Tenet X [STABLE-ID]". You may cite the tenet in codebase as a comment, other markdown files, or even this file itself.

## Maintaining This File [WRITING-THIS-FILE]
- Give every referable section a unique, short, stable ID of the form [TENET-ID] placed at the end of the title. The ID should give enough context to the reader (LLM or human) on what the tenet is. There should be no punctuation or underscores in the stable ID but dashes are allowed. 
- The tenet may be referred to from other tenets within this file, some other markdown file, a comment in any of the codebases.
- Keep each tenet small, concise, concrete, and easy to cite.
- Add examples to every numbered tenet of the form "Good" and "Bad" typically with brief code snippets. Code snippets do not need to be syntactically valid; you can abbreviate using `...` or comments as needed.
- Before adding or editing a new tenet, check
  1. if the tenet already exists and edit the existing tenet instead of recreating. Never duplicate the same tenet.
  2. the latest available [Google Python Style Guide](https://google.github.io/styleguide/pyguide.html), falling back to the local snapshot at [google-python-style-guide-2026-05-25.html](references/google-python-style-guide-2026-05-25.html) when internet access is unavailable, or other relevant sources to see how the world handles a similar problem. Then educate the human briefly if needed and offer the human to edit/update a tenet.
- Never add a tenet without a good reason.
- Do not assume the human is always correct. Never add a tenet simply to placate the human when the human is wrong or "going against the grain".
- Anti-patterns are discouraged but allowed in this [GLOBAL-PYTHON-TENETS] file in the [EXCEPTIONS] section.

### Fixing numbering issues [FIX-NUMBERING-ISSUES]
- If you see that the tenet numbering is wrong (skipped numbers), let the human know and then offer to fix them. The human may delay fixing the numbering in some cases when the human expects a skipped number to be replaced later.
- When you find a "Tenet X [STABLE-ID]" citation but no Tenet X exists, let the human know, and then offer to fix them.


## Tenets [TENETS]
### Noted Exceptions [EXCEPTIONS]
None recorded yet.

### Tenet 1: Prefer Explicit Data Flow [PY-EXPLICIT-FLOW]
Pass dependencies and inputs directly instead of hiding them in globals,
environment lookups, or module-level mutable state.

Good:
```python
def build_report(rows: list[Row], formatter: Formatter) -> str:
    return formatter.render(rows)
```

Bad:
```python
def build_report() -> str:
    return GLOBAL_FORMATTER.render(load_rows_from_global_path())
```

### Tenet 2: Keep Functions Focused [PY-FOCUSED-FUNCTIONS]
A function should usually do one job at one level of abstraction. Split work
when naming the smaller operation makes the code easier to read.

Good:
```python
def active_users(users: Iterable[User]) -> list[User]:
    return [user for user in users if user.is_active]
```

Bad:
```python
def active_users_and_write_csv(users: Iterable[User], path: Path) -> None:
    ...
```

### Tenet 3: Use Types To Explain Boundaries [PY-TYPED-BOUNDARIES]
Add type hints at module and function boundaries. Use domain names, dataclasses,
TypedDict, Protocol, or simple value objects when primitives become ambiguous.

Good:
```python
@dataclass(frozen=True)
class RetryPolicy:
    attempts: int
    backoff_seconds: float


def fetch_invoice(invoice_id: str, retry: RetryPolicy) -> Invoice:
    ...
```

Bad:
```python
def fetch_invoice(invoice_id: str, attempts: int, backoff_seconds: float):
    ...
```

### Tenet 4: Do Not Mix Business Logic With IO [PY-IO-BOUNDARY]
Keep decisions separate from reading files, making network calls, printing,
logging, database access, or shell execution. IO wrappers should gather inputs
and persist outputs; pure functions should decide what should happen.

Good:
```python
def overdue_invoices(invoices: Iterable[Invoice], today: date) -> list[Invoice]:
    return [invoice for invoice in invoices if invoice.due_date < today]


def load_and_send_reminders(path: Path, today: date) -> None:
    invoices = read_invoices(path)
    for invoice in overdue_invoices(invoices, today):
        send_reminder(invoice)
```

Bad:
```python
def load_and_send_reminders(path: Path, today: date) -> None:
    for invoice in read_invoices(path):
        if invoice.due_date < today:
            send_reminder(invoice)
```

### Tenet 5: Handle Errors At The Right Level [PY-ERROR-LEVEL]
Catch exceptions where the code can add context, retry, recover, or choose a
user-facing response. Do not catch exceptions only to hide them.

Good:
```python
try:
    invoice = parse_invoice(raw_invoice)
except InvalidInvoice as exc:
    raise ImportError(f"Invalid invoice in {source_path}") from exc
```

Bad:
```python
try:
    invoice = parse_invoice(raw_invoice)
except Exception:
    return None
```

### Tenet 6: Make Tests Describe Behavior [PY-BEHAVIOR-TESTS]
Test observable behavior and meaningful edge cases. Avoid tests that only copy
the implementation structure.

Good:
```python
def test_overdue_invoices_excludes_invoice_due_today() -> None:
    today = date(2026, 5, 25)
    invoices = [Invoice("a", today), Invoice("b", today - timedelta(days=1))]

    assert overdue_invoices(invoices, today) == [invoices[1]]
```

Bad:
```python
def test_overdue_invoices_uses_less_than_operator() -> None:
    assert "<" in inspect.getsource(overdue_invoices)
```
