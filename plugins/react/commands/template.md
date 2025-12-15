---
description: Show comment template for React/Next.js file types
---

# React Template

Display comment template for a specific file type (reference only, no file creation).

## Step 1: Select File Type

### Round 1 - Common Types

**Ask user (AskUserQuestion):**
"Which file type template do you need?

Available: Component, Context, Service, Query, DTO, Utils"

| Option | Description |
|--------|-------------|
| Component | React component with hooks structure |
| Context | Context + Provider + custom hook |
| Service | API service with methods |
| More types | Show Query, DTO, Utils templates |

### Round 2 - If "More types" selected

**Ask user (AskUserQuestion):**
"Select additional type:"

| Option | Description |
|--------|-------------|
| Query | React Query hooks |
| DTO | Request/Response type definitions |
| Utils | Utility helper functions |
| Hook | Custom React hooks |

## Step 2: Output Template

Based on selection, output the template as a code block.

> **Next.js Special Files:** `page.tsx`, `layout.tsx`, `loading.tsx`, `error.tsx`, `not-found.tsx` follow the Component template structure. Use section name like `Page Component` or `Layout Component` instead of just `Component`.

### Component Template

```typescript
'use client';

/*
 * ============================================
 * Constants
 * ============================================
 */

/** Constant description */
const MAX_VALUE = 100;

/*
 * ============================================
 * Type Definitions
 * ============================================
 */

/** ComponentName props */
interface ComponentProps {
  /** Prop description */
  value: string;
  /** Prop description */
  onChange?: (value: string) => void; // Optional callback
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
  const isValid = localValue.length <= MAX_VALUE;

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
  /** Helper methods */
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
    () => ({
      value,
      setValue,
      reset,
    }),
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

### Query Template

```typescript
/**
 * [Feature query hooks]
 * React Query hooks for feature service (CSR only)
 */

'use client';

import { useQuery, useMutation, useQueryClient } from '@tanstack/react-query';

import type { ApiResponse } from '@/types/api.types';

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
  return useQuery<ApiResponse<GetResponse>, Error, GetResponse>({
    queryKey: [FEATURE_QUERY_KEY.LIST, params],
    queryFn: () => featureService.getFeatures(params),
    select: (response) => response.data,
  });
};
```

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
 * GET /features
 * Response for feature list
 */
export interface GetResponse {
  /** List of features */
  items: Feature[];
  /** Total count */
  total: number;
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

/**
 * Calculate feature score
 * @param views - Number of views
 * @param likes - Number of likes
 * @returns Weighted score
 */
export function calculateScore(views: number, likes: number): number {
  return views * 0.3 + likes * 0.7;
}
```

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

## Output Format

```
📄 [Type] Template

[Code block with template]

💡 Tips:
- [Type-specific tips]
```
