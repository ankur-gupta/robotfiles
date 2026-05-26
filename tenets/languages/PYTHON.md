# Global Python Tenets [PY-GLOBAL-PYTHON-TENETS]
This file defines global Python coding tenets (in [PY-TENETS] section) for LLM coding agents. Project-local instructions may override it when they are more specific.

## Using This File [PY-USING-THIS-FILE]
This [PY-GLOBAL-PYTHON-TENETS] file is part of a hierarchy
  1. Project-specific Python guide (if it exists) 
  2. This [PY-GLOBAL-PYTHON-TENETS] file
  3. Latest available [Google Python Style Guide](https://google.github.io/styleguide/pyguide.html)

Always try to retrieve the latest Google Python Style Guide from the internet first. If internet access is unavailable, use the local snapshot at [google-python-style-guide-2026-05-25.html](references/google-python-style-guide-2026-05-25.html).

If there is a conflict between the above three sources, the project-specific Python guide supersedes this [PY-GLOBAL-PYTHON-TENETS] file, which then supersedes the latest available Google Python Style Guide or, when offline, the local snapshot.

### Finding tenets [PY-FINDING-TENETS]
- When you see a tenet citation like "Tenet X [PY-TENET-ID]", give precedence to the semantic text of stable ID over the number when trying to find the relevant tenet. Numbering can get messed up.
- When explaining a change, cite the relevant tenet only when it clarifies the reasoning.
- Always cite a tenet in this form: "Tenet X [PY-TENET-ID]". You may cite the tenet in codebase as a comment, other markdown files, or even this file itself.

## Maintaining This File [PY-WRITING-THIS-FILE]
- Give every referable section a unique, short, stable tenet ID of the form [PY-TENET-ID] placed at the end of the title. The tenet ID should start with `PY-` to reflect that this is a Python tenet. Codebases can be polyglot, and the tenet ID should be unique across all tenets in the `$REPO_ROOT/tenets` folder. The tenet ID should give enough context to the reader (LLM or human) on what the tenet is. There should be no punctuation or underscores in the stable ID but dashes are allowed.
- The tenet may be referred to from other tenets within this file, some other markdown file, a comment in any of the codebases.
- Keep each tenet small, concise, concrete, and easy to cite.
- Add examples to every numbered tenet of the form "Good" and "Bad" typically with brief code snippets. Code snippets do not need to be syntactically valid; you can abbreviate using `...` or comments as needed. Add brief reasoning to each example so readers can see why the example satisfies or violates the tenet.
- Before adding or editing a new tenet, check
  1. if the tenet already exists and edit the existing tenet instead of recreating. Never duplicate the same tenet.
  2. the latest available [Google Python Style Guide](https://google.github.io/styleguide/pyguide.html), falling back to the local snapshot at [google-python-style-guide-2026-05-25.html](references/google-python-style-guide-2026-05-25.html) when internet access is unavailable, or other relevant sources to see how the world handles a similar problem. Then educate the human briefly if needed and offer the human to edit/update a tenet.
- Never add a tenet without a good reason.
- Do not assume the human is always correct. Never add a tenet simply to appease the human when the human is wrong or "going against the grain".
- Anti-patterns are discouraged but allowed in this [PY-GLOBAL-PYTHON-TENETS] file in the [PY-EXCEPTIONS] section.

### Fixing numbering issues [PY-FIX-NUMBERING-ISSUES]
- If you see that the tenet numbering is wrong (skipped numbers), let the human know and then offer to fix them. The human may delay fixing the numbering in some cases when the human expects a skipped number to be replaced later.
- When you find a "Tenet X [PY-TENET-ID]" citation but no Tenet X exists, let the human know, and then offer to fix them.


## Tenets [PY-TENETS]
### Noted Exceptions [PY-EXCEPTIONS]
None recorded yet.

### Tenet 1: Do not appease the user unless explicitly asked [PY-DO-NOT-APPEASE-USER]
Assume the user is smart but not always correct. Do not explain routine Python
choices unless the user demonstrates a misunderstanding. When the user requests
an anti-pattern or shows confusion about a Python tradeoff, briefly say so,
give a small example, and avoid changing code merely to appease the user unless
the user explicitly asks to override this guidance. Record intentional
anti-pattern overrides in [PY-EXCEPTIONS].

Good:
```text
That would make the parser depend on a mutable module global, which conflicts
with Tenet 4 [PY-EXPLICIT-FLOW]. A small alternative is to pass the parser
config into `parse_invoice(...)`.
```

Bad:
```text
Sure, I moved the parser config into a module global because you asked.
```

### Tenet 2: Maintain Code Locality [PY-MAINTAIN-CODE-LOCALITY]
Before adding code, check whether it belongs in the current function, module,
package, or layer. A good rule of thumb is to check whether the surrounding
code serves the same concern. Keep modules focused on their stated
responsibility, and put shared helpers in the existing module or package that
owns that concern instead of placing unrelated code near the caller.

Good:
```python
# storage/files.py
def safe_write(path: Path, data: bytes) -> None:
    ...

# serializers/widgets.py
def write_widget_json(widget: Widget, path: Path) -> None:
    # do some custom logic here
    storage.files.safe_write(path, widget_to_json(widget))
    ...
```
Reason: The generic disk I/O helper lives with other I/O code, while the
serializer module only handles widget-to-JSON behavior.

Bad:
```python
# serializers/widgets.py
def safe_write(path: Path, data: bytes) -> None:
    ...

def write_widget_json(widget: Widget, path: Path) -> None:
    # do some custom logic here
    safe_write(path, widget_to_json(widget))
    ...
```
Reason: Both functions are in the same file, so generic disk I/O code is mixed
into a module whose responsibility is widget JSON serialization.

### Tenet 3: Keep Business Logic Separate [PY-BUSINESS-LOGIC-BOUNDARY]
Keep domain decisions separate from infrastructure and other surrounding
concerns. Business rules should not be hidden inside file or network IO,
database access, shell execution, UI or CLI handling, framework request
objects, wire or file serialization, logging, metrics, tracing, environment
lookups, clocks, randomness, or generic helper utilities. Put those concerns
at the edges and pass plain inputs into focused domain functions.

This is the boundary that Clean Architecture and Hexagonal Architecture
protect, but do not add architectural layers merely to name the pattern. When
implementing a feature, first identify which parts are business rules and which
parts are generic or infrastructural. Reuse existing generic functionality when
the owning module already provides it. Implement business rules separately from
generic helpers and infrastructure, then put each piece in the module or
package that owns its concern as described by Tenet 2
[PY-MAINTAIN-CODE-LOCALITY].

Good:
```python
def overdue_invoices(invoices: Iterable[Invoice], today: date) -> list[Invoice]:
    return [invoice for invoice in invoices if invoice.due_date < today]


def load_and_send_reminders(path: Path, today: date) -> None:
    invoices = read_invoices(path)
    for invoice in overdue_invoices(invoices, today):
        send_reminder(invoice)
```
Reason: The domain rule can be read and tested without file access, network
calls, logging, or framework objects.

Bad:
```python
def load_and_send_reminders(path: Path, logger: Logger) -> None:
    for invoice in read_invoices(path):
        if invoice.due_date < date.today():
            logger.info("Sending reminder for %s", invoice.id)
            send_reminder(invoice)
```
Reason: The overdue rule is tangled with file IO, the system clock, logging,
and sending side effects, so changing or testing the rule requires unrelated
concerns.

### Tenet 4: Prefer Explicit Data Flow [PY-EXPLICIT-FLOW]
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

### Tenet 5: Keep Functions Focused [PY-FOCUSED-FUNCTIONS]
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

### Tenet 6: Use Types To Explain Boundaries [PY-TYPED-BOUNDARIES]
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

### Tenet 7: Handle Errors At The Right Level [PY-ERROR-LEVEL]
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

### Tenet 8: Make Tests Describe Behavior [PY-BEHAVIOR-TESTS]
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

## Additional Python References [PY-ADDITIONAL-REFERENCES]
When the Google Python Style Guide is not enough to resolve a question, use
these sources as additional reference points:

- [Python Enhancement Proposals](https://peps.python.org/), especially accepted
  informational PEPs such as [PEP 8](https://peps.python.org/pep-0008/).
- [Official Python Tutorial](https://docs.python.org/3/tutorial/), especially
  the sections on modules, packages, errors, classes, and the standard library.
- [Official Python Language Reference](https://docs.python.org/3/reference/)
  when syntax, import behavior, execution model, or data model details matter.
- [The Hitchhiker's Guide to Python](https://docs.python-guide.org/) for
  community guidance on project structure and maintainable Python practice.
