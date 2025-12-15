---
name: react-conventions
description: React/Next.js component structure and comment conventions
globs: ["**/*.tsx", "**/*.ts"]
---

# React/Next.js Conventions

## Component Structure (1-11 Order)

> **Next.js Special Files:** `page.tsx`, `layout.tsx`, `loading.tsx`, `error.tsx`, `not-found.tsx` follow this structure. Use section name like `Page Component` or `Layout Component` instead of just `Component`.

```typescript
'use client';

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

interface Props {
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
export function ComponentName({ value, onChange }: Props) {
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
  const [localValue, setLocalValue] = useState(value);

  /*
   * --------------------------------------------
   * 3. Query Hooks
   * --------------------------------------------
   */
  const { data } = useQuery({ ... });

  /*
   * --------------------------------------------
   * 4. Custom Hooks
   * --------------------------------------------
   */
  const debouncedValue = useDebounce(localValue, 300);

  /*
   * --------------------------------------------
   * 5. Computed Values
   * --------------------------------------------
   */
  /** Filtered items based on search */
  const filteredItems = useMemo(() => items.filter(...), [items]);

  /*
   * --------------------------------------------
   * 6. Derived Values
   * --------------------------------------------
   */
  const isEmpty = localValue.length === 0;

  /*
   * --------------------------------------------
   * 7. Callbacks
   * --------------------------------------------
   */
  /** Handle value change and notify parent */
  const handleChange = useCallback((newValue: string) => {
    setLocalValue(newValue);
    onChange?.(newValue);
  }, [onChange]);

  /*
   * --------------------------------------------
   * 8. Helper Functions
   * --------------------------------------------
   */
  /** Format value for display */
  const formatValue = (val: string) => val.trim();

  /*
   * --------------------------------------------
   * 9. Event Handlers
   * --------------------------------------------
   */
  /** Handle form submission */
  const handleSubmit = (e: FormEvent) => {
    e.preventDefault();
  };

  /*
   * --------------------------------------------
   * 10. Effects
   * --------------------------------------------
   */
  /** Sync local value with prop */
  useEffect(() => {
    setLocalValue(value);
  }, [value]);

  /*
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
/*
 * ============================================
 * Section Name
 * ============================================
 */

const FIRST_DECLARATION = 'value';
```

> **Important**: Use `/*` (not `/**`) for section headers. `/**` is JSDoc and IDE will show it as documentation for the next declaration. `/*` is a regular comment that IDE ignores.

### Component-level sections (inside component)
```typescript
/*
 * --------------------------------------------
 * 1. Section Name
 * --------------------------------------------
 */
```

### Function/variable descriptions (sections 2, 5-10)
```typescript
/** Filtered items based on search */
const filteredItems = useMemo(() => { ... }, []);
```

> Section 1 (External Hooks), 3-4 (Query Hooks, Custom Hooks) typically don't need comments (IDE hover shows their JSDoc), but add if extra context is helpful.
> Section 2 (States) and 5-10 use `/**` comments - useState/useRef don't show purpose in hover.
> Section 6 (Derived Values) - add comments only if variable name doesn't clearly explain purpose.

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
/*
 * ============================================
 * Context
 * ============================================
 */

/** My feature context */
const MyContext = createContext<Type | undefined>(undefined);

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
  /*
   * --------------------------------------------
   * 1. States
   * --------------------------------------------
   */
  /** Current context value */
  const [value, setValue] = useState('');

  /*
   * --------------------------------------------
   * 2. Callbacks - Actions
   * --------------------------------------------
   */

  /*
   * --------------------------------------------
   * 3. Context Value
   * --------------------------------------------
   */

  /*
   * --------------------------------------------
   * 4. Return
   * --------------------------------------------
   */
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
  /**
   * GET /features
   * Get feature list with filters
   *
   * @param params - Query parameters
   * @param params.page - Page number
   * @param params.limit - Items per page
   * @param params.search - Search keyword (optional)
   * @returns Feature list response
   */
  getFeatures: async (params: GetParams) => {},

  /**
   * POST /features
   * Create new feature
   *
   * @param params - Feature data
   * @param params.name - Feature name
   * @param params.description - Feature description (optional)
   * @returns Empty response on success
   */
  createFeature: async (data: CreateRequest) => {},
};
```

### Query Pattern
```typescript
/**
 * [Feature query hooks]
 * React Query hooks for feature service
 */

/*
 * ============================================
 * Query Keys
 * ============================================
 */

/** Feature query keys */
export const FEATURE_QUERY_KEY = {
  LIST: 'featureList',
  DETAIL: 'featureDetail',
} as const;

/*
 * ============================================
 * Hooks
 * ============================================
 */

/**
 * GET /features
 * Get feature list query
 *
 * @param params - Query parameters
 * @param params.page - Page number
 * @param params.limit - Items per page
 * @param params.search - Search keyword (optional)
 * @returns Feature list data
 */
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

### Hook Pattern
```typescript
/**
 * [Feature hooks]
 * Custom hooks for feature functionality
 */

/**
 * Hook to manage feature state with persistence
 *
 * @param initialValue - Initial feature value
 * @param options - Hook options (debounce, persist)
 * @returns Feature state and control functions
 *
 * @example
 * const { value, setValue, reset } = useFeature('default', { persist: true });
 */
export function useFeature(
  initialValue: string,
  options?: { debounce?: number; persist?: boolean }
) {
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
  return { value, setValue, reset, isLoading };
}
```
