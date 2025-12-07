---
description: Add or reformat comments in React/Next.js files following CLAUDE.md conventions
---

# React Comment

Apply CLAUDE.md comment conventions to React/Next.js files.

## Step 0: Select Target Files

**Ask user:**
"Which files should I add comments to?"

| Option | Description |
|--------|-------------|
| 1 | All files (all `.tsx`, `.ts` in project) |
| 2 | Changed files (`git diff` - all modified files) |
| 3 | Staged files (`git diff --staged` - staged files only) |
| 4 | Other (enter file path or describe context) |

### Get file list:

```bash
# Option 1: All files
find src -type f \( -name "*.tsx" -o -name "*.ts" \) | head -50

# Option 2: Changed files
git diff --name-only | grep -E '\.(tsx?|ts)$'

# Option 3: Staged files
git diff --staged --name-only | grep -E '\.(tsx?|ts)$'
```

### Option 4 handling:
- **File path**: `src/components/Button.tsx` → process that file
- **Context description**: "calendar related" → search and list matching files, confirm with user

### Multiple files:
Process each file through Steps 1-5 (batch processing)

## Step 1: Detect File Type

Determine file type from filename and path (support singular/plural):

| Pattern | Type |
|---------|------|
| `*.dto.ts` | DTO |
| `*.service.ts` | Service |
| `*.query.ts`, `*.queries.ts` | Query |
| `*.util.ts`, `*.utils.ts`, `*Util.ts`, `*Utils.ts` | Utility |
| `*.type.ts`, `*.types.ts` | Types |
| `*.constant.ts`, `*.constants.ts` | Constants |
| `lib/*.ts` | Library |
| `*Context.tsx`, `*Provider.tsx` | Context |
| `_hooks/*.ts`, `*.hook.ts`, `*.hooks.ts` | Hooks |
| `*.tsx` (component folder) | Component |
| Other `.ts` | Generic TS |
| Other `.tsx` | Generic TSX |

## Step 2: Analyze Current Comment State

Check existing comments:
- File-level JSDoc present?
- Section separators (`====` or `----`) present?
- Function/variable comments present?

## Step 3: Select Comment Mode

**Ask user:**
"How should I handle comments?"

| Option | Description |
|--------|-------------|
| 1 | Add missing (only add where missing) |
| 2 | Full reformat (reorganize everything) |
| 3 | Section only (only add section separators) |

## Step 4: Apply Comments by File Type

### 4.1 Component Files (*.tsx)

**Top-level sections (==== format, no numbering):**
```typescript
/**
 * ============================================
 * Constants
 * ============================================
 */

/**
 * ============================================
 * Type Definitions
 * ============================================
 */

/**
 * ============================================
 * Component
 * ============================================
 */
```

**Inside component (---- format, numbered):**
```typescript
function MyComponent() {
  /**
   * --------------------------------------------
   * 1. External Hooks
   * --------------------------------------------
   */
  // Navigation
  const router = useRouter();
  // Context
  const { user } = useAuthContext();

  /**
   * --------------------------------------------
   * 2. States
   * --------------------------------------------
   */
  const [value, setValue] = useState('');

  /**
   * --------------------------------------------
   * 3. Query Hooks
   * --------------------------------------------
   */

  /**
   * --------------------------------------------
   * 4. Custom Hooks
   * --------------------------------------------
   */

  /**
   * --------------------------------------------
   * 5. Computed Values
   * --------------------------------------------
   */

  /**
   * --------------------------------------------
   * 6. Derived Values
   * --------------------------------------------
   */

  /**
   * --------------------------------------------
   * 7. Callbacks
   * --------------------------------------------
   */

  /**
   * --------------------------------------------
   * 8. Helper Functions
   * --------------------------------------------
   */

  /**
   * --------------------------------------------
   * 9. Event Handlers
   * --------------------------------------------
   */

  /**
   * --------------------------------------------
   * 10. Effects
   * --------------------------------------------
   */

  /**
   * --------------------------------------------
   * 11. Return
   * --------------------------------------------
   */
  return (...);
}
```

**Function/variable descriptions:** Use `/** description */` format

### 4.2 Context Files (*Context.tsx, *Provider.tsx)

**Top-level sections:**
```typescript
/**
 * ============================================
 * Type Definitions
 * ============================================
 */

/**
 * ============================================
 * Context
 * ============================================
 *
 * Context description here
 */

/**
 * ============================================
 * Provider
 * ============================================
 *
 * Provider description
 *
 * @param children - Child components to wrap
 */

/**
 * ============================================
 * Custom Hook
 * ============================================
 */
```

**Inside Provider (numbered, subcategories allowed):**
```typescript
function MyProvider({ children }) {
  /**
   * --------------------------------------------
   * 1. States
   * --------------------------------------------
   */

  /**
   * --------------------------------------------
   * 2. Callbacks - Navigation
   * --------------------------------------------
   */

  /**
   * --------------------------------------------
   * 3. Callbacks - Helpers
   * --------------------------------------------
   */

  /**
   * --------------------------------------------
   * 4. Context Value
   * --------------------------------------------
   */

  /**
   * --------------------------------------------
   * 5. Return
   * --------------------------------------------
   */
}
```

### 4.3 Service Files (*.service.ts)

**Structure (NO section separators!):**
```typescript
/**
 * [Service name]
 * Description (SSR/CSR compatible, etc.)
 */

export const xxxService = {
  /**
   * HTTP_METHOD /endpoint
   * Description
   */
  methodName: async (...) => { ... },
};
```

### 4.4 Query Files (*.query.ts)

**Structure:**
```typescript
/**
 * [Query hooks name]
 * Description (CSR only, etc.)
 */

'use client';

/**
 * ============================================
 * Query Keys
 * ============================================
 */
export const XXX_QUERY_KEY = { ... };

/**
 * ============================================
 * Hooks
 * ============================================
 */

/**
 * HTTP_METHOD /endpoint
 * Description
 */
export const useXxxQuery = (...) => { ... };
```

### 4.5 DTO Files (*.dto.ts)

**Structure:**
```typescript
/**
 * [DTO name]
 * Description
 */

/**
 * HTTP_METHOD /endpoint
 * Description
 */
export interface XxxParams {
  /** Field description */
  fieldName: string;
  /** Field description */
  anotherField: number; // Additional note (inline)
}
```

### 4.6 Utils Files (*.util.ts, *.utils.ts)

**Structure:**
```typescript
/**
 * [Utilities name]
 * Description
 */

/**
 * Function description
 * @param paramName - Parameter description
 * @returns Return value description
 */
export function functionName(...): ReturnType { ... }
```

### 4.7 Types/Constants Files

**Structure:**
```typescript
/**
 * [Type/Constants name]
 * Description
 */

/** Type description */
export type XxxType = ...;

/** Constant description */
export const XXX_CONSTANT = ...;
```

## Step 5: Show Changes Summary

```
📝 Comment changes:
  - File-level JSDoc: Added
  - Section headers: 3 added
  - Function descriptions: 5 added

Apply changes? (y/n)
```

## Comment Style Rules Reference

| Location | Style | Example |
|----------|-------|---------|
| Top-level sections | JSDoc + `====` | `Type Definitions` |
| Component sections | JSDoc + `----` + number | `1. External Hooks` |
| Constants/type properties | `/** */` | `/** Max items */` |
| Type annotations | `//` inline | `date: string; // YYYY-MM-DD` |
| Function/variable inside component | `/** */` | `/** Handle click */` |
| Logic explanation | `//` inline | `// Skip if empty` |

## Constraints

1. **Preserve existing content**: Only modify comment style, keep the meaning
2. **English comments**: All comments should be in English
3. **Skip empty sections**: Don't add section headers for empty sections
4. **Match file conventions**: Use patterns from existing code in the project
