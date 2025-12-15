---
description: Add or reformat comments in React/Next.js files following CLAUDE.md conventions
---

# React Comment

Apply CLAUDE.md comment conventions to React/Next.js files.

## Step 0: Select Target Files

**Ask user (AskUserQuestion):**
"Which files should I add comments to?"

| Option | Description |
|--------|-------------|
| All files | All `.tsx`, `.ts` files in project |
| Changed files | Modified files from `git diff` |
| Staged files | Staged files from `git diff --staged` |

### Get file list:

```bash
# All files
find src -type f \( -name "*.tsx" -o -name "*.ts" \) | head -50

# Changed files
git diff --name-only | grep -E '\.(tsx?|ts)$'

# Staged files
git diff --staged --name-only | grep -E '\.(tsx?|ts)$'
```

User can also enter custom input via "Other" option:
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

**Ask user (AskUserQuestion):**
"How should I handle comments?"

| Option | Description |
|--------|-------------|
| Add missing | Only add comments where missing |
| Full reformat | Reorganize all comments |
| Section only | Only add section separators |

## Step 4: Apply Comments by File Type

### 4.1 Component Files (*.tsx)

**Top-level sections (==== format, no numbering):**
```typescript
/*
 * ============================================
 * Constants
 * ============================================
 */

const MAX_ITEMS = 100;

/*
 * ============================================
 * Type Definitions
 * ============================================
 */

interface MyComponentProps {
  // ...
}

/*
 * ============================================
 * Component
 * ============================================
 */

/**
 * Component description
 *
 * @param value - Current value
 * @param onChange - Callback when value changes
 */
```

**Inside component (---- format, numbered):**
```typescript
function MyComponent() {
  /*
   * --------------------------------------------
   * 1. External Hooks
   * --------------------------------------------
   */
  const router = useRouter();
  const { user } = useAuthContext();

  /*
   * --------------------------------------------
   * 2. States
   * --------------------------------------------
   */
  /** Current input value */
  const [value, setValue] = useState('');

  /*
   * --------------------------------------------
   * 3. Query Hooks
   * --------------------------------------------
   */

  /*
   * --------------------------------------------
   * 4. Custom Hooks
   * --------------------------------------------
   */

  /*
   * --------------------------------------------
   * 5. Computed Values
   * --------------------------------------------
   */

  /*
   * --------------------------------------------
   * 6. Derived Values
   * --------------------------------------------
   */

  /*
   * --------------------------------------------
   * 7. Callbacks
   * --------------------------------------------
   */

  /*
   * --------------------------------------------
   * 8. Helper Functions
   * --------------------------------------------
   */

  /*
   * --------------------------------------------
   * 9. Event Handlers
   * --------------------------------------------
   */

  /*
   * --------------------------------------------
   * 10. Effects
   * --------------------------------------------
   */

  /*
   * --------------------------------------------
   * 11. Return
   * --------------------------------------------
   */
  return (...);
}
```

**Function/variable descriptions (sections 2, 5-10):** Use `/** description */` for States, Computed Values, Callbacks, Helper Functions, Event Handlers, Effects (shown in IDE hover). Sections 1, 3-4 typically don't need comments (IDE hover shows their JSDoc), but add if extra context is helpful. Section 6 (Derived Values) - add comments only if variable name doesn't clearly explain purpose.

### 4.2 Context Files (*Context.tsx, *Provider.tsx)

**Top-level sections:**
```typescript
/*
 * ============================================
 * Type Definitions
 * ============================================
 */

interface ContextType {
  // ...
}

interface ProviderProps {
  children: React.ReactNode;
}

/*
 * ============================================
 * Context
 * ============================================
 */

const MyContext = createContext<ContextType | undefined>(undefined);

/*
 * ============================================
 * Provider
 * ============================================
 */

/**
 * My feature provider
 *
 * @param children - Child components to wrap
 */
export function MyProvider({ children }: ProviderProps) {
  // ...
}

/*
 * ============================================
 * Custom Hook
 * ============================================
 */

/**
 * Hook to access MyContext
 *
 * @returns Context value with state and actions
 * @throws Error if used outside of MyProvider
 */
export function useMyContext() {
  // ...
}
```

**Inside Provider (numbered, subcategories allowed):**
```typescript
function MyProvider({ children }) {
  /*
   * --------------------------------------------
   * 1. States
   * --------------------------------------------
   */
  /** Current context value */
  const [value, setValue] = useState('');

  /*
   * --------------------------------------------
   * 2. Callbacks - Navigation
   * --------------------------------------------
   */

  /*
   * --------------------------------------------
   * 3. Callbacks - Helpers
   * --------------------------------------------
   */

  /*
   * --------------------------------------------
   * 4. Context Value
   * --------------------------------------------
   */

  /*
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
   *
   * @param params - Parameter description
   * @param params.field1 - Field 1 description
   * @param params.field2 - Field 2 description (optional)
   * @returns Response description
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

/*
 * ============================================
 * Query Keys
 * ============================================
 */

export const XXX_QUERY_KEY = { ... };

/*
 * ============================================
 * Hooks
 * ============================================
 */

/**
 * HTTP_METHOD /endpoint
 * Description
 *
 * @param params - Query parameters
 * @param params.field1 - Field 1 description
 * @param params.field2 - Field 2 description (optional)
 * @returns Query data
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

### 4.8 Hook Files (*.hook.ts, *.hooks.ts)

**Structure:**
```typescript
/**
 * [Hook name]
 * Description
 */

'use client';

/**
 * Hook description
 *
 * @param initialValue - Initial value
 * @param options - Hook options
 * @returns State and control functions
 *
 * @example
 * const { value, setValue } = useFeature('default');
 */
export function useFeature(initialValue: string, options?: Options) {
  /*
   * --------------------------------------------
   * 1. States
   * --------------------------------------------
   */
  /** Current feature value */
  const [value, setValue] = useState(initialValue);
  /** Loading state for async operations */
  const [isLoading, setIsLoading] = useState(false);

  /*
   * --------------------------------------------
   * 2. Callbacks
   * --------------------------------------------
   */

  /*
   * --------------------------------------------
   * 3. Effects
   * --------------------------------------------
   */

  /*
   * --------------------------------------------
   * 4. Return
   * --------------------------------------------
   */
  return { value, setValue, reset };
}
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
| Top-level sections | `/*` + `====` | `Type Definitions` |
| Component sections | `/*` + `----` + number | `1. External Hooks` |
| Constants/type properties | `/** */` | `/** Max items */` |
| Type annotations | `//` inline | `date: string; // YYYY-MM-DD` |
| Function/variable inside component | `/** */` (shown in IDE hover) | `/** Handle click */` |
| Logic explanation | `//` inline | `// Skip if empty` |

> **Important**: Use `/*` (not `/**`) for Top-level section and Component section headers. `/**` is JSDoc and IDE will show it as documentation for the next declaration. `/*` is a regular comment that IDE ignores.

## Constraints

1. **Preserve existing content**: Only modify comment style, keep the meaning
2. **English comments**: All comments should be in English
3. **Skip empty sections**: Don't add section headers for empty sections
4. **Match file conventions**: Use patterns from existing code in the project
5. **Use `/*` for section headers**: Prevents IDE from showing section header as documentation
