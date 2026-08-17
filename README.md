# kbg-platform-core — MOVED

> ## → **https://github.com/MohanKbgrid/kbg-platform**
>
> This content now lives at [`bases/core`](https://github.com/MohanKbgrid/kbg-platform/tree/main/bases/core)
> in the **kbg-platform** repo. **This repository is archived and read-only.**

## Why it moved

A product inheriting the platform needs three things: the **contracts** that say what to build, the
**gate tests** that prove it, and a **base** to start from. Split across separate repos, the specs
and the code they describe sat in different places — and the specs are the half that actually
travels between products.

They are now one repo, so there is one name to remember and one URL to hand an agent:

```
canon/           the specs — 10 surfaces, 12 doctrines, adoption guides, gate tests
bases/core/      domain-free Python primitives
bases/ui-kit/    the visual and behavioural language
```

## Get the Python primitives

```bash
git clone https://github.com/MohanKbgrid/kbg-platform.git /tmp/kbg
cp -r /tmp/kbg/bases/core/src/*  <your-project>/app/vendor/kbg_platform_core/
git -C /tmp/kbg rev-parse HEAD    # ← record this as your base commit
```

**Vendor it, do not depend on it.** Copy it in, own it, change it freely. There is deliberately no
`pip install` / `npm install` — a shared dependency spanning unrelated products becomes either a
lowest-common-denominator that fits none of them, or a "shared" package with per-project branches,
which is worse than two honest copies.

Divergence is expected and is not propagated back. What propagates is the **gate**: when a bug
reveals a missing rule, the rule goes into that surface's spec in `canon/`, so every product
inherits the *test* while keeping its own code.
