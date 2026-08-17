# kbg-platform-core

The small, **domain-free** primitives that enforce the [KBG Platform Surfaces canon](../../docs/canon/README.md)'s
doctrines. Artifact 2 of the canon.

## What this is — and deliberately is not

The canon's thesis is that **code is cheap to port but expensive to trust, so the global asset is
the CONTRACT, not the library.** That constrains what belongs here.

**In scope** — the handful of mechanisms every project would otherwise re-implement identically,
where re-implementing it *differently* is the actual bug:

| Module | Doctrine it makes hard to violate |
|---|---|
| `result` | **D4** — missing data is a finding. A caller cannot read a value without handling its absence. |
| `authority` | **D1** deny-by-default, and *one* function drives both the disabled button and the 403. |
| `versioned` | **D2** — append-only. Changing a value supersedes; it never updates in place. |
| `outbox` | **D1 + Integration §10** — the environment gate. Non-production **holds**, it does not pretend. |
| `idem` | **D3** — idempotency by client-minted source id. |
| `testkit` | **D5 + D7** — prove the control controls; refuse to run against a non-fixture database. |

**Out of scope, on purpose:** ORM models, migrations, GL adapters, HTTP framework glue, anything
domain-specific. Those belong to each project, built against the contract in `docs/canon/` and
proven by that surface's gates. A package that shipped them would be exactly the "expensive to
trust" code the thesis warns about.

**No third-party runtime dependencies.** Pure standard library, so it drops into any Python project
regardless of ORM, web framework, or database driver. Adapters are *callables you pass in*.

## Install

```bash
pip install -e packages/kbg-platform-core
```

## Use

```python
from kbg_platform_core import Resolved, Gap, Authority, Decision, VersionedWriter, Outbox, Mode

# D4 — absence is not None, it is a Gap you must handle
price = resolve_price(product)              # -> Resolved | Gap
if price.is_gap:
    return {"blocked": True, "reason": price.reason, "code": price.code}
amount = price.value                        # .value on a Gap raises, by design
```

```python
# D1 — one function answers "may I", and the UI and the API both call it
class MyAuthority(Authority):
    def _evaluate(self, actor, action, obj):
        if action not in self.granted(actor):
            return Decision.deny("not granted to this role",
                                 who_can="Plant Manager")
        return Decision.allow()

auth = MyAuthority()
d = auth.check(actor, "order.correct", order)
# UI:  {"enabled": d.allowed, "reason": d.reason, "whoCan": d.who_can}
# API: if not d.allowed: raise HTTPException(403, d.reason)
```

```python
# Integration §10 — the gate, provable three ways
box = Outbox(mode=Mode.UAT, store=my_store)
row = box.enqueue(topic="erp.invoice.push.v1", source_key=invoice_id, payload=payload)
assert row.state is State.HELD                 # not sent, not faked, inspectable
assert row.held_reason.startswith("integration_mode=uat")
```

## Testing your adoption

`testkit` carries the two assertions the canon insists on:

```python
from kbg_platform_core.testkit import require_fixture_db, assert_control_controls

require_fixture_db(current_db_name(), allowed={"myapp_dev", "myapp_test"})

assert_control_controls(
    describe="off-location threshold",
    run=lambda: derive_flags(visit),           # returns the observed output
    change=lambda: set_threshold(2000),        # config-only change
    restore=lambda: set_threshold(200),
)
# fails if the output did NOT change -> the threshold is hardcoded somewhere
```

## Versioning

`0.x` while the canon's first three consumers (Vijay, Aurafab, Solomon) shake it out. Breaking
changes are expected and will be listed in each surface's divergence register, not hidden behind a
compatibility shim.
