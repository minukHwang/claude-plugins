# React Plugin

React/Next.js development automation tools for Claude Code.

## Commands

| Command | Description |
|---------|-------------|
| `/react:comment` | Add/reformat comments following CLAUDE.md conventions |
| `/react:template` | Show comment template for specific file type |

## Skills

| Skill | Description |
|-------|-------------|
| `react-conventions` | Component structure & comment conventions (auto-referenced for `.tsx`/`.ts` files) |

## Supported File Types

| Pattern | Type |
|---------|------|
| `*.tsx` | Component |
| `*Context.tsx`, `*Provider.tsx` | Context |
| `*.service.ts` | Service |
| `*.query.ts` | Query |
| `*.dto.ts` | DTO |
| `*.util.ts`, `*.utils.ts` | Utility |
| `*.type.ts`, `*.types.ts` | Types |
| `*.constant.ts`, `*.constants.ts` | Constants |

## Comment Style Reference

| Location | Style | Example |
|----------|-------|---------|
| Top-level sections | `/*` + `====` | Type Definitions |
| Component sections | `/*` + `----` + number | 1. External Hooks |
| Constants/type properties | `/** */` | `/** Max items */` |
| Type annotations | `//` inline | `date: string; // YYYY-MM-DD` |
| Function/variable inside component | `/** */` | `/** Handle click */` |
| Logic explanation | `//` inline | `// Skip if empty` |

> **Important**: Use `/*` (not `/**`) for Top-level section and Component section headers. `/**` is JSDoc and IDE will show it as documentation for the next declaration. `/*` is a regular comment that IDE ignores.

**Which sections need `/** */` comments?**
- Section 2 (States), 5-10: Use `/** description */` - useState/useRef and custom logic need explanation
- Section 1 (External Hooks), 3-4 (Query/Custom Hooks): Typically no comments needed (IDE hover shows JSDoc), but add if extra context helpful
- Section 6 (Derived Values): Add comments only if variable name doesn't clearly explain purpose

---

## Template Examples

> **Next.js Special Files:** `page.tsx`, `layout.tsx`, `loading.tsx`, `error.tsx`, `not-found.tsx` follow the Component template structure. Use section name like `Page Component` or `Layout Component` instead of just `Component`.

### Component Template

```typescript
'use client';

/*
 * ============================================
 * Constants
 * ============================================
 */

/** Maximum number of items allowed */
const MAX_ITEMS = 100;

/*
 * ============================================
 * Type Definitions
 * ============================================
 */

/** ComponentName props */
interface ComponentProps {
  /** Prop description */
  value: string;
  /** Optional callback */
  onChange?: (value: string) => void;
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
export function ComponentName({ value, onChange }: ComponentProps) {
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
  /** Reference to input element */
  const inputRef = useRef<HTMLInputElement>(null);

  /*
   * --------------------------------------------
   * 3. Query Hooks
   * --------------------------------------------
   */
  const { data, isLoading } = useQuery({ ... });

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
  const filteredItems = useMemo(() => {
    return items.filter(...);
  }, [items]);

  /*
   * --------------------------------------------
   * 6. Derived Values
   * --------------------------------------------
   */
  const isEmpty = localValue.length === 0;
  const isValid = localValue.length <= MAX_ITEMS;

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
  const formatValue = (val: string) => {
    return val.trim().toLowerCase();
  };

  /*
   * --------------------------------------------
   * 9. Event Handlers
   * --------------------------------------------
   */
  /** Handle form submission */
  const handleSubmit = (e: FormEvent) => {
    e.preventDefault();
    // Submit logic
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
  return (
    <div>
      {/* JSX here */}
    </div>
  );
}
```

---

### Context Template

```typescript
'use client';

import { createContext, useCallback, useContext, useMemo, useState } from 'react';

/*
 * ============================================
 * Type Definitions
 * ============================================
 */

/** Context value type */
interface ContextType {
  /** Current value */
  value: string;
  /** Update value */
  setValue: (value: string) => void;
  /** Reset to initial */
  reset: () => void;
}

/** Provider props */
interface ProviderProps {
  children: React.ReactNode;
}

/*
 * ============================================
 * Context
 * ============================================
 */

/** My feature context */
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

  /** Reset to initial state */
  const reset = useCallback(() => {
    setValue('');
  }, []);

  /*
   * --------------------------------------------
   * 3. Context Value
   * --------------------------------------------
   */
  const contextValue = useMemo(
    () => ({ value, setValue, reset }),
    [value, reset]
  );

  /*
   * --------------------------------------------
   * 4. Return
   * --------------------------------------------
   */
  return <MyContext.Provider value={contextValue}>{children}</MyContext.Provider>;
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
    throw new Error('useMyContext must be used within a MyProvider');
  }
  return context;
}
```

---

### Service Template

```typescript
/**
 * [Feature service]
 * API calls for feature management (SSR/CSR compatible)
 */

import axiosInstance from '@/lib/axios';
import type { ApiResponse } from '@/types/api.types';

import type { CreateRequest, GetParams, GetResponse } from './feature.dto';

/** Feature API service */
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
  getFeatures: async (params: GetParams): Promise<ApiResponse<GetResponse>> => {
    const { data } = await axiosInstance.get('/features', { params });
    return data;
  },

  /**
   * POST /features
   * Create new feature
   *
   * @param params - Feature data
   * @param params.name - Feature name
   * @param params.description - Feature description (optional)
   * @returns Empty response on success
   */
  createFeature: async (params: CreateRequest): Promise<ApiResponse<null>> => {
    const { data } = await axiosInstance.post('/features', params);
    return data;
  },
};
```

---

### Query Template

```typescript
/**
 * [Feature query hooks]
 * React Query hooks for feature service (CSR only)
 */

'use client';

import { useQuery } from '@tanstack/react-query';

import type { GetParams, GetResponse } from './feature.dto';
import { featureService } from './feature.service';

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
    select: (response) => response.data,
  });
};
```

---

### DTO Template

```typescript
/**
 * [Feature service type definitions]
 * Request and response types for feature endpoints
 */

/**
 * GET /features
 * Query parameters for feature list
 */
export interface GetParams {
  /** Page number (1-indexed) */
  page: number;
  /** Items per page */
  limit: number;
  /** Optional search keyword */
  search?: string; // Partial match
}

/**
 * POST /features
 * Request body for creating feature
 */
export interface CreateRequest {
  /** Feature name */
  name: string;
  /** Feature description */
  description?: string;
}
```

---

### Utils Template

```typescript
/**
 * [Feature utilities]
 * Helper functions for feature operations
 */

/**
 * Format feature name for display
 * @param name - Raw feature name
 * @returns Formatted name with proper casing
 */
export function formatFeatureName(name: string): string {
  return name.trim().charAt(0).toUpperCase() + name.slice(1);
}

/**
 * Check if feature is valid
 * @param feature - Feature to validate
 * @returns True if feature passes all validation rules
 */
export function isValidFeature(feature: Feature): boolean {
  return feature.name.length > 0 && feature.name.length <= 100;
}
```

---

### Hook Template

```typescript
/**
 * [Feature hooks]
 * Custom hooks for feature functionality
 */

'use client';

import { useCallback, useEffect, useState } from 'react';

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

  /** Reset to initial value */
  const reset = useCallback(() => {
    setValue(initialValue);
  }, [initialValue]);

  /** Update value with validation */
  const updateValue = useCallback((newValue: string) => {
    if (newValue.length <= 100) {
      setValue(newValue);
    }
  }, []);

  /*
   * --------------------------------------------
   * 3. Effects
   * --------------------------------------------
   */

  /** Persist to localStorage when value changes */
  useEffect(() => {
    if (options?.persist) {
      localStorage.setItem('feature-value', value);
    }
  }, [value, options?.persist]);

  /*
   * --------------------------------------------
   * 4. Return
   * --------------------------------------------
   */
  return {
    value,
    setValue: updateValue,
    reset,
    isLoading,
  };
}
```

## Component Section Order

```
1. External Hooks     6. Derived Values
2. States             7. Callbacks
3. Query Hooks        8. Helper Functions
4. Custom Hooks       9. Event Handlers
5. Computed Values   10. Effects
                     11. Return
```

## License

MIT
