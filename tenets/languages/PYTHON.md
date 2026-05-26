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
with Tenet 5 [PY-PURE-DOMAIN-FUNCTIONS]. A small alternative is to pass the
parser config into `parse_invoice(...)`.
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

### Tenet 3: Do Not Duplicate Code [PY-DO-NOT-DUPLICATE-CODE]
Avoid copying the same logic into multiple places. Before writing similar code,
look for an existing function, method, fixture, constant, type, or module that
already owns the behavior. Reuse or extend that owner when the callers need the
same rule, and extract a small helper when duplication would make future
changes easy to miss.

Do not hide different behavior behind a shared abstraction only because two
snippets look similar. Keep code separate when the concepts or reasons for
change are different, even if some lines currently match.

Good:
```python
def total_price(items: Iterable[LineItem]) -> Money:
    return sum((item.price for item in items), start=Money.zero())


def invoice_total(invoice: Invoice) -> Money:
    return total_price(invoice.items)


def cart_total(cart: Cart) -> Money:
    return total_price(cart.items)
```
Reason: One pricing rule has one owner, so a future rule change is made in one
place.

Bad:
```python
def invoice_total(invoice: Invoice) -> Money:
    return sum((item.price for item in invoice.items), start=Money.zero())


def cart_total(cart: Cart) -> Money:
    return sum((item.price for item in cart.items), start=Money.zero())
```
Reason: The same pricing rule is copied into two functions, so the next pricing
change can update one caller and silently leave the other behind.

### Tenet 4: Keep Business Logic Separate [PY-BUSINESS-LOGIC-BOUNDARY]
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

### Tenet 5: Keep Domain Functions Pure [PY-PURE-DOMAIN-FUNCTIONS]
Prefer functions whose results are determined by their arguments. Keep
environment lookups, clocks, randomness, filesystem access, network calls,
logging, and other process or external state at the program boundary. Read and
validate deploy configuration such as environment variables once near startup,
then pass the resulting values into focused functions.

Some functions are expected to be impure: entrypoints, CLI handlers, web route
handlers, task runners, adapters, repositories, clients, file readers/writers,
and logging or metrics setup. Keep that impurity near the edge and pass plain
values into the functions that express domain rules.

This tenet does not forbid environment variables. Environment variables are
often the right place for deploy-specific configuration, but repeated
`os.getenv(...)` calls inside business logic make behavior depend on hidden
process state and make tests harder to reason about.

Good:
```python
@dataclass(frozen=True)
class ReportConfig:
    timezone: ZoneInfo
    include_drafts: bool


def build_report(rows: Iterable[Row], config: ReportConfig) -> Report:
    ...


def main() -> None:
    config = ReportConfig(
        timezone=ZoneInfo(require_env("REPORT_TIMEZONE")),
        include_drafts=parse_bool(require_env("REPORT_INCLUDE_DRAFTS")),
    )
    rows = load_rows()
    write_report(build_report(rows, config))
```
Reason: Environment-dependent configuration is read at the edge, while the
domain function receives explicit inputs and is easy to test.

Bad:
```python
def build_report(rows: Iterable[Row]) -> Report:
    timezone = ZoneInfo(os.getenv("REPORT_TIMEZONE", "UTC"))
    include_drafts = os.getenv("REPORT_INCLUDE_DRAFTS") == "1"
    ...
```
Reason: The function's behavior depends on hidden process state, so callers and
tests cannot understand the report rule from the function signature.

### Tenet 6: Use Types To Explain Boundaries [PY-TYPED-BOUNDARIES]
Add type hints at module and function boundaries. When a group of values
travels together as one domain concept, or when primitives make a boundary
ambiguous, name the concept with a dataclass, TypedDict, Protocol, type alias,
or small value object.

Define named types near the package or module that owns the concept. If the
type is used by multiple files in that package, move it to a predictable shared
owner such as `models.py`, `schemas.py`, `contracts.py`, or a domain-specific
module. Avoid placing shared types only where they were first needed, and avoid
broad junk drawer modules detached from ownership.

Good:
```python
# billing/models.py
@dataclass(frozen=True)
class RetryPolicy:
    attempts: int
    backoff_seconds: float


# billing/invoices.py
def fetch_invoice(invoice_id: str, retry: RetryPolicy) -> Invoice:
    ...
```
Reason: The retry settings travel together as one boundary concept, and the
shared type lives in a predictable owner within the billing package.

Bad:
```python
# billing/invoices.py
def fetch_invoice(invoice_id: str, attempts: int, backoff_seconds: float):
    ...


# billing/reminders.py
def send_invoice_reminder(invoice_id: str, attempts: int, backoff_seconds: float):
    ...
```
Reason: The same unnamed retry concept is repeated as loose primitives, so
callers must infer which values belong together and where the shared contract
lives.

### Tenet 7: Do Not Swallow Exceptions [PY-DO-NOT-SWALLOW-EXCEPTIONS]
Catch only the exception types the code knows how to handle. Do not catch
exceptions only to hide bugs, discard failures, or let the program continue as
if nothing happened. Let unexpected exceptions propagate.

Swallowing an exception is acceptable only when the code intentionally handles a
specific expected failure. The common cases are: returning a documented local
fallback such as `None` or a default, retrying or using a fallback source,
choosing a user-facing response at an application boundary, or creating an
isolation point where the failure is recorded before execution continues. When
translating an exception instead of swallowing it, raise a meaningful built-in
or domain exception and preserve the cause with `raise ... from exc` unless
there is a deliberate reason to suppress it.

Good:
```python
def parse_optional_int(text: str) -> int | None:
    try:
        return int(text)
    except ValueError:
        return None
```
Reason: Invalid numeric input is an expected local failure, and the function's
contract makes the fallback explicit.

Good:
```python
try:
    invoice = parse_invoice(raw_invoice)
except InvalidInvoice as exc:
    raise ImportError(f"Invalid invoice in {source_path}") from exc
```
Reason: The lower-level parse failure is translated with context while
preserving the original cause.

Bad:
```python
try:
    invoice = parse_invoice(raw_invoice)
except Exception:
    return None
```
Reason: A broad catch hides unexpected failures and gives the caller no signal
about what went wrong.

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

### Tenet 9: Do Not Mix CLI With Package Code [PY-SEPARATE-CLI-FROM-PACKAGE]
When Python code is a package, keep reusable functionality importable without
going through a command-line interface. Put argument parsing, terminal IO,
process exits, and console formatting in a small CLI entrypoint that calls the
package API. A caller in another codebase should be able to import the package
and use the same core behavior without shelling out to the CLI.

CLI entrypoints may live in the package when that matches the project layout,
but they should behave as adapters over importable functions rather than as the
only place where the behavior exists.

Good:
```python
# report_builder/reports.py
def build_report(rows: Iterable[Row], include_drafts: bool) -> Report:
    ...


# report_builder/cli.py
def main(argv: Sequence[str] | None = None) -> int:
    args = parse_args(argv)
    report = build_report(load_rows(args.input), args.include_drafts)
    print(render_report(report))
    return 0
```
Reason: The report behavior is available as an importable package API, and the
CLI only adapts command-line inputs and terminal output.

Bad:
```python
# report_builder/cli.py
def main() -> None:
    args = parse_args()
    rows = load_rows(args.input)
    # all report-building rules live here
    print(render_report(...))
```
Reason: Another Python caller has to invoke the CLI or duplicate its internals
to use the same report-building behavior.

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
