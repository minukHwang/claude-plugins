---
name: react-conventions
description: React/Next.js component structure and comment conventions
globs: ["**/*.tsx", "**/*.ts"]
---

# React/Next.js Conventions

## Component Structure (1-11 Order)

```typescript
'use client';

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

export function ComponentName({ props }: Props) {
  /**
   * --------------------------------------------
   * 1. External Hooks
   * --------------------------------------------
   */
  // Navigation: useRouter, useSearchParams, usePathname
  // Context: useAuthContext, useThemeContext
  // Form: useForm
  // Other library hooks

  /**
   * --------------------------------------------
   * 2. States
   * --------------------------------------------
   */
  // useState, useRef

  /**
   * --------------------------------------------
   * 3. Query Hooks
   * --------------------------------------------
   */
  // useQuery, useMutation (after states - callbacks may use state setters)

  /**
   * --------------------------------------------
   * 4. Custom Hooks
   * --------------------------------------------
   */
  // useDebounce, useClickOutside, etc.

  /**
   * --------------------------------------------
   * 5. Computed Values
   * --------------------------------------------
   */
  // useMemo

  /**
   * --------------------------------------------
   * 6. Derived Values
   * --------------------------------------------
   */
  // Simple computations from state/props (no hooks)

  /**
   * --------------------------------------------
   * 7. Callbacks
   * --------------------------------------------
   */
  // useCallback

  /**
   * --------------------------------------------
   * 8. Helper Functions
   * --------------------------------------------
   */
  // Pure utility functions

  /**
   * --------------------------------------------
   * 9. Event Handlers
   * --------------------------------------------
   */
  // onClick, onSubmit, etc.

  /**
   * --------------------------------------------
   * 10. Effects
   * --------------------------------------------
   */
  // useEffect

  /**
   * --------------------------------------------
   * 11. Return
   * --------------------------------------------
   */
  return <div>{/* JSX */}</div>;
}
```

## Comment Conventions

### Top-level sections (outside component)
```typescript
/**
 * ============================================
 * Section Name
 * ============================================
 */
```

### Component-level sections (inside component)
```typescript
/**
 * --------------------------------------------
 * 1. Section Name
 * --------------------------------------------
 */
```

### Function/variable descriptions
```typescript
/** Description of function or variable */
const myFunction = () => {};
```

### Logic explanations
```typescript
// Inline comment explaining logic
```

### Interface properties
```typescript
interface Props {
  /** Property description */
  value: string;
  count: number; // Additional note
}
```

## File Types

### Context Pattern
```typescript
const MyContext = createContext<Type | undefined>(undefined);

export function MyProvider({ children }: ProviderProps) {
  // 1. States
  // 2. Callbacks - Actions
  // 3. Context Value (useMemo)
  // 4. Return
}

export function useMyContext() {
  const context = useContext(MyContext);
  if (context === undefined) {
    throw new Error('useMyContext must be used within MyProvider');
  }
  return context;
}
```

### Service Pattern
```typescript
/**
 * [Feature service]
 * API calls for feature management
 */

export const featureService = {
  /** GET /features */
  getFeatures: async (params: GetParams) => {},

  /** POST /features */
  createFeature: async (data: CreateRequest) => {},
};
```

### Query Pattern
```typescript
/**
 * [Feature query hooks]
 * React Query hooks for feature service
 */

export const FEATURE_QUERY_KEY = {
  LIST: 'featureList',
  DETAIL: 'featureDetail',
} as const;

export const useFeatureListQuery = (params: GetParams) => {
  return useQuery({
    queryKey: [FEATURE_QUERY_KEY.LIST, params],
    queryFn: () => featureService.getFeatures(params),
  });
};
```

### DTO Pattern
```typescript
/**
 * [Feature service type definitions]
 * Request and response types
 */

/** GET /features - Query params */
export interface GetParams {
  /** Page number */
  page: number;
}

/** POST /features - Request body */
export interface CreateRequest {
  /** Feature name */
  name: string;
}
```

### Utils Pattern
```typescript
/**
 * [Feature utilities]
 * Helper functions
 */

/**
 * Format feature name
 * @param name - Raw name
 * @returns Formatted name
 */
export function formatFeatureName(name: string): string {
  return name.trim();
}
```
