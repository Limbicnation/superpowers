---
name: refactoring-impact-predictor
description: Use when renaming, moving, or restructuring code—before executing changes—to predict ripple effects and create safe migration plans
---

# Refactoring Impact Predictor

## Overview

Refactoring without impact analysis causes broken builds, missed updates, and documentation drift. This skill predicts all ripple effects before you change anything.

**Core principle:** Map ALL impacts before ANY changes. No surprises.

## When to Use

- Renaming functions, classes, variables, or files
- Moving code between modules/packages
- Changing function signatures
- Extracting or inlining code
- Restructuring directories
- Modifying public APIs
- Deprecating functionality

**Especially use when:**

- Change affects code you didn't write
- Multiple teams depend on this code
- External consumers exist (libraries, APIs)
- Documentation references the code

## The Impact Analysis Process

### Phase 1: Build Reference Graph

**Find all references to the target:**

```bash
# Direct references (function/class usage)
grep -rn "functionName" --include="*.ts" --include="*.js"

# Import statements
grep -rn "from.*module" --include="*.ts"

# Type references
grep -rn ": TypeName" --include="*.ts"

# String references (dynamic calls, configs)
grep -rn '"functionName"' .
grep -rn "'functionName'" .
```

**Categorize references:**

| Category | Risk Level | Examples |
|----------|------------|----------|
| **Direct Imports** | 🟢 Low | IDE can auto-fix |
| **Type References** | 🟢 Low | IDE can auto-fix |
| **Dynamic Calls** | 🟠 Medium | `obj[methodName]()` |
| **String References** | 🔴 High | Config files, reflection |
| **External Consumers** | 🔴 High | Published packages, APIs |
| **Documentation** | 🟡 Medium | README, docstrings |

### Phase 2: Impact Classification

**Output format:**

```
REFACTORING: Rename formatDate → formatDateTime

IMPACT ANALYSIS:
────────────────────────────────────────────
✅ SAFE (Automated - 47 sites)
   ├── src/utils/date.ts:12     (definition)
   ├── src/components/Card.tsx:8    (import)
   ├── src/pages/Dashboard.tsx:23   (import)
   └── ... 44 more

⚠️ NEEDS REVIEW (8 sites)
   ├── tests/date.test.ts:15    (test mocks)
   ├── docs/api.md:127          (documentation)
   └── config/mappings.json:45  (string reference)

❌ EXTERNAL (3 consumers)
   ├── @company/shared-utils (imports formatDate)
   ├── API endpoint /v1/format (query param)
   └── Mobile app (v2.3.1 uses this)

CONFIDENCE SCORE: 78%
   Automated: 47/58 (81%)
   Manual review: 8/58 (14%)
   External blockers: 3 (require coordination)
```

### Phase 3: Migration Plan

**For safe refactoring (>90% confidence):**

```markdown
## Migration Plan: formatDate → formatDateTime

### Step 1: Update Definition
- [ ] Rename function in src/utils/date.ts

### Step 2: Automated Updates (IDE)
- [ ] Run "Rename Symbol" refactoring
- [ ] Verify 47 reference updates

### Step 3: Manual Updates
- [ ] Update docs/api.md:127
- [ ] Update config/mappings.json:45
- [ ] Update test mocks in tests/date.test.ts

### Step 4: Verification
- [ ] Run full test suite
- [ ] Check build succeeds
- [ ] Grep for old name (should be zero)
```

**For risky refactoring (<70% confidence):**

```markdown
## Migration Plan: Breaking API Change

### Option A: Deprecation Path (Recommended)
1. Add new function alongside old
2. Mark old as @deprecated
3. Update internal callers
4. Notify external consumers
5. Remove after 2 release cycles

### Option B: Hard Cutover
1. Coordinate with all consumers
2. Schedule maintenance window
3. Update all at once
4. Monitor for breakage
```

## Confidence Scoring

| Factor | Points |
|--------|--------|
| All references found by static analysis | +30 |
| 100% test coverage on affected code | +20 |
| No dynamic/string references | +20 |
| No external consumers | +15 |
| Documentation is current | +10 |
| Single codebase (monorepo) | +5 |

| Score | Meaning |
|-------|---------|
| 90-100% | Safe for automated refactoring |
| 70-89% | Manual review recommended |
| 50-69% | Deprecation path advised |
| <50% | High risk, need coordination |

## Common Refactoring Patterns

### Rename (Function/Class/Variable)

```bash
# 1. Find all references
grep -rn "oldName" .

# 2. Use IDE rename (captures types)
# 3. Grep again to verify zero remaining
grep -rn "oldName" . | wc -l  # Should be 0
```

### Move (File/Module)

```bash
# 1. Find all imports
grep -rn "from.*oldPath" .

# 2. Move file
git mv src/old/file.ts src/new/file.ts

# 3. Update imports (IDE or sed)
sed -i 's|from.*oldPath|from "new/path"|g' **/*.ts
```

### Change Signature

```markdown
# Before
function calculate(a: number, b: number): number

# After
function calculate(options: CalculateOptions): number

# Migration
1. Create new signature
2. Add adapter: calculate(a, b) → calculate({a, b})
3. Update callers incrementally
4. Remove adapter after all updated
```

## Quick Reference

```
Before ANY Refactoring:

□ Grep for all references (code, tests, docs, configs)
□ Categorize by risk (automated vs manual)
□ Identify external consumers
□ Calculate confidence score
□ Choose migration strategy
□ Create step-by-step plan
□ Verify with tests after each step
```

## Integration with Existing Skills

- **REQUIRED:** Use `superpowers:writing-plans` for complex refactoring plans
- **REQUIRED:** Use `superpowers:test-driven-development` to verify no regressions
- **RECOMMENDED:** Use `superpowers:verification-before-completion` after migration
