# Test Inventory - CareerAdapt AI v0.9.x

## Test Configuration Files

| Config | Purpose |
|--------|---------|
| vitest.config.ts | Unit test configuration |
| vitest.ai-real.config.ts | AI real integration tests |
| vitest.c1-eval.config.ts | C1 evaluation tests |
| vitest.c2-eval.config.ts | C2 evaluation tests |
| playwright.config.ts | E2E test configuration |

## Test Commands

```bash
# Unit tests
pnpm test

# AI real integration tests
pnpm test:ai:real

# C1 evaluation
pnpm test:c1:eval

# C2 evaluation
pnpm test:c2:eval

# E2E tests (requires dev server)
pnpm test:e2e

# Full verification
pnpm verify
```

## RC Status (v0.9.3-rc.1)

| Gate | Status |
|------|--------|
| Deterministic gates | PASS (112 files / 780 tests) |
| Lint | PASS |
| Typecheck | PASS |
| Build | PASS |
| C1/C2 | PASS |
| Real Provider smoke | FAIL (HTTP 401) |
| Canonical PDF | PASS |
| Final Job Resume PDF | FAIL |

## Journey Results

| Journey | Result |
|---------|--------|
| J1 File → Profile → General Resume → PDF | FAIL |
| J2 Conversation → Profile → Resume | FAIL |
| J3 Re-import / Reconcile | FAIL |
| J4 Branch Isolation | FAIL |
| J5 Persistence / Reload | FAIL |
| J6 Failure Recovery | FAIL |

Last updated: 2026-07-29
