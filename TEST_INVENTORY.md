# Test Inventory - CareerAdapt AI v0.9.x

## Test Configuration Files

| Config | Purpose |
|--------|---------|
| vitest.config.ts | Unit test configuration |
| vitest.ai-real.config.ts | AI real integration tests |
| vitest.c1-eval.config.ts | C1 evaluation tests |
| vitest.c2-eval.config.ts | C2 evaluation tests |
| playwright.config.ts | E2E test configuration |

## Unit & Integration Tests (tests/)

### Domain Tests
- `tests/unit/` - Domain logic unit tests
- `tests/integration/` - Integration tests

### E2E Tests
- `tests/e2e/` - Playwright E2E tests

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

## Coverage Areas

| Area | Test Type | Status |
|------|-----------|--------|
| Resume Import | Unit + E2E | ✅ |
| Profile Intake | Unit | ✅ |
| Job Optimization | Unit + E2E | ✅ |
| Tailoring Engine | Unit | ✅ |
| Agent Runtime | Unit | ✅ |
| PDF Export | Integration | ✅ |
| Schema Validation | Unit | ✅ |

Last updated: 2026-07-29
