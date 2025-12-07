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
| Top-level sections | JSDoc + `====` | Type Definitions |
| Component sections | JSDoc + `----` + number | 1. External Hooks |
| Function/variable | `/** */` | Build URL with date |
| Inline annotation | `//` | `// YYYY-MM-DD format` |

---

## Template Examples

### Component Template

```typescript
'use client';

/**
 * ============================================
 * Constants
 * ============================================
 */

/** Maximum number of items allowed */
const MAX_ITEMS = 100;

/**
 * ============================================
 * Type Definitions
 * ============================================
 */

interface ComponentProps {
  /** Prop description */
  value: string;
  /** Optional callback */
  onChange?: (value: string) => void;
}

/**
 * ============================================
 * Component
 * ============================================
 */

export function ComponentName({ value, onChange }: ComponentProps) {
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
  const [localValue, setLocalValue] = useState(value);
  const inputRef = useRef<HTMLInputElement>(null);

  /**
   * --------------------------------------------
   * 3. Query Hooks
   * --------------------------------------------
   */
  const { data, isLoading } = useQuery({ ... });

  /**
   * --------------------------------------------
   * 4. Custom Hooks
   * --------------------------------------------
   */
  /** Debounced value for API calls */
  const debouncedValue = useDebounce(localValue, 300);

  /**
   * --------------------------------------------
   * 5. Computed Values
   * --------------------------------------------
   */
  /** Filtered items based on search */
  const filteredItems = useMemo(() => {
    return items.filter(...);
  }, [items]);

  /**
   * --------------------------------------------
   * 6. Derived Values
   * --------------------------------------------
   */
  const isEmpty = localValue.length === 0;
  const isValid = localValue.length <= MAX_ITEMS;

  /**
   * --------------------------------------------
   * 7. Callbacks
   * --------------------------------------------
   */
  /** Handle value change and notify parent */
  const handleChange = useCallback((newValue: string) => {
    setLocalValue(newValue);
    onChange?.(newValue);
  }, [onChange]);

  /**
   * --------------------------------------------
   * 8. Helper Functions
   * --------------------------------------------
   */
  /** Format value for display */
  const formatValue = (val: string) => {
    return val.trim().toLowerCase();
  };

  /**
   * --------------------------------------------
   * 9. Event Handlers
   * --------------------------------------------
   */
  const handleSubmit = (e: FormEvent) => {
    e.preventDefault();
    // Submit logic
  };

  /**
   * --------------------------------------------
   * 10. Effects
   * --------------------------------------------
   */
  /** Sync local value with prop */
  useEffect(() => {
    setLocalValue(value);
  }, [value]);

  /**
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

/**
 * ============================================
 * Type Definitions
 * ============================================
 */

interface ContextType {
  /** Current value */
  value: string;
  /** Update value */
  setValue: (value: string) => void;
  /** Reset to initial */
  reset: () => void;
}

interface ProviderProps {
  children: React.ReactNode;
}

/**
 * ============================================
 * Context
 * ============================================
 *
 * Context description here
 */

const MyContext = createContext<ContextType | undefined>(undefined);

/**
 * ============================================
 * Provider
 * ============================================
 *
 * Provider description
 *
 * @param children - Child components to wrap
 */
export function MyProvider({ children }: ProviderProps) {
  /**
   * --------------------------------------------
   * 1. States
   * --------------------------------------------
   */
  const [value, setValue] = useState('');

  /**
   * --------------------------------------------
   * 2. Callbacks - Actions
   * --------------------------------------------
   */

  /** Reset to initial state */
  const reset = useCallback(() => {
    setValue('');
  }, []);

  /**
   * --------------------------------------------
   * 3. Context Value
   * --------------------------------------------
   */
  const contextValue = useMemo(
    () => ({ value, setValue, reset }),
    [value, reset]
  );

  /**
   * --------------------------------------------
   * 4. Return
   * --------------------------------------------
   */
  return <MyContext.Provider value={contextValue}>{children}</MyContext.Provider>;
}

/**
 * ============================================
 * Custom Hook
 * ============================================
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

export const featureService = {
  /**
   * GET /features
   * Get feature list with filters
   */
  getFeatures: async (params: GetParams): Promise<ApiResponse<GetResponse>> => {
    const { data } = await axiosInstance.get('/features', { params });
    return data;
  },

  /**
   * POST /features
   * Create new feature
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

/**
 * ============================================
 * Query Keys
 * ============================================
 */
export const FEATURE_QUERY_KEY = {
  LIST: 'featureList',
  DETAIL: 'featureDetail',
} as const;

/**
 * ============================================
 * Hooks
 * ============================================
 */

/**
 * GET /features
 * Get feature list query
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
