# State Management

**Version:** 1.0.0
**Last Updated:** 2026-01-09

---

## Table of Contents

1. [Overview](#overview)
2. [State Management Strategy](#state-management-strategy)
3. [React Context](#react-context)
4. [Component State Patterns](#component-state-patterns)
5. [Form State Management](#form-state-management)
6. [Data Fetching Patterns](#data-fetching-patterns)
7. [Caching Strategies](#caching-strategies)
8. [Local Storage](#local-storage)

---

## Overview

The NPDI application uses a combination of React Context, component-level state, and local storage for state management. This approach keeps the architecture simple while meeting the application's needs.

### State Types

| Type | Technology | Use Case |
|------|------------|----------|
| Global app state | React Context | User authentication, app-wide settings |
| Page/component state | useState/useReducer | Form data, UI state, loading states |
| Form state | react-hook-form | Complex form management |
| Persistent state | localStorage | User preferences, session data |
| Server state | API calls | Database data (tickets, users) |

---

## State Management Strategy

### Architecture Diagram

```
┌─────────────────────────────────────────────────────────────┐
│                        App Component                         │
│  ┌───────────────────────────────────────────────────────┐  │
│  │                    AuthProvider                        │  │
│  │  ┌─────────────────────────────────────────────────┐  │  │
│  │  │              Router / Pages                      │  │  │
│  │  │                                                  │  │  │
│  │  │  ┌────────────────┐  ┌────────────────────────┐ │  │  │
│  │  │  │  Page State    │  │   Component State      │ │  │  │
│  │  │  │  (useState)    │  │   (useState)           │ │  │  │
│  │  │  └────────────────┘  └────────────────────────┘ │  │  │
│  │  │                                                  │  │  │
│  │  │  ┌────────────────────────────────────────────┐ │  │  │
│  │  │  │           Form State (react-hook-form)     │ │  │  │
│  │  │  └────────────────────────────────────────────┘ │  │  │
│  │  └─────────────────────────────────────────────────┘  │  │
│  └───────────────────────────────────────────────────────┘  │
└─────────────────────────────────────────────────────────────┘
                              │
                              ▼
                    ┌─────────────────┐
                    │  localStorage   │
                    └─────────────────┘
```

### When to Use Each Approach

| Scenario | Approach |
|----------|----------|
| User authentication | AuthContext |
| Form data | react-hook-form |
| List filtering/sorting | Component useState |
| Loading states | Component useState |
| Modal open/close | Component useState |
| User preferences | localStorage + useState |
| Session persistence | localStorage |

---

## React Context

### AuthContext

**Location:** `client/src/context/AuthContext.jsx`

**Purpose:** Manages user authentication state and profile selection.

**State:**
```javascript
{
  user: {
    id: string,
    employeeId: string,
    email: string,
    firstName: string,
    lastName: string,
    role: 'PRODUCT_MANAGER' | 'PM_OPS' | 'ADMIN',
    sbu: string
  } | null,
  token: string | null,
  profiles: Profile[],
  loading: boolean
}
```

**Usage:**
```jsx
import { useAuth } from '../context/AuthContext';

const MyComponent = () => {
  const { user, loading, selectProfile, logout } = useAuth();

  if (loading) return <Loading />;
  if (!user) return <Navigate to="/select-profile" />;

  return (
    <div>
      <p>Welcome, {user.firstName}</p>
      <p>Role: {user.role}</p>
      <button onClick={logout}>Logout</button>
    </div>
  );
};
```

**Provided Values:**

| Value | Type | Description |
|-------|------|-------------|
| `user` | Object\|null | Current user profile |
| `token` | String\|null | Auth token (dev mode) |
| `profiles` | Array | Available profiles |
| `loading` | Boolean | Loading state |
| `selectProfile` | Function | Select a profile by ID |
| `logout` | Function | Clear auth state |
| `updateUserProfile` | Function | Update user data |

### Creating New Context

```jsx
// context/MyContext.jsx
import React, { createContext, useContext, useState, useEffect } from 'react';

const MyContext = createContext();

export const useMyContext = () => {
  const context = useContext(MyContext);
  if (!context) {
    throw new Error('useMyContext must be used within MyProvider');
  }
  return context;
};

export const MyProvider = ({ children }) => {
  const [state, setState] = useState(initialState);

  const value = {
    state,
    updateState: (newState) => setState(prev => ({ ...prev, ...newState }))
  };

  return (
    <MyContext.Provider value={value}>
      {children}
    </MyContext.Provider>
  );
};
```

---

## Component State Patterns

### Loading States

```jsx
const TicketList = () => {
  const [tickets, setTickets] = useState([]);
  const [loading, setLoading] = useState(true);
  const [error, setError] = useState(null);

  useEffect(() => {
    const fetchTickets = async () => {
      try {
        setLoading(true);
        setError(null);
        const response = await productAPI.getTickets();
        setTickets(response.data);
      } catch (err) {
        setError(err.message);
      } finally {
        setLoading(false);
      }
    };

    fetchTickets();
  }, []);

  if (loading) return <LoadingSpinner />;
  if (error) return <ErrorMessage message={error} />;

  return <TicketTable tickets={tickets} />;
};
```

### Filter/Search State

```jsx
const Dashboard = () => {
  const [filters, setFilters] = useState({
    status: '',
    sbu: '',
    search: ''
  });

  const handleFilterChange = (key, value) => {
    setFilters(prev => ({ ...prev, [key]: value }));
  };

  const filteredTickets = useMemo(() => {
    return tickets.filter(ticket => {
      if (filters.status && ticket.status !== filters.status) return false;
      if (filters.sbu && ticket.sbu !== filters.sbu) return false;
      if (filters.search && !ticket.productName.includes(filters.search)) return false;
      return true;
    });
  }, [tickets, filters]);

  return (
    <div>
      <FilterBar filters={filters} onChange={handleFilterChange} />
      <TicketList tickets={filteredTickets} />
    </div>
  );
};
```

### Modal State

```jsx
const TicketActions = ({ ticket }) => {
  const [showDeleteModal, setShowDeleteModal] = useState(false);
  const [showEditModal, setShowEditModal] = useState(false);

  return (
    <>
      <button onClick={() => setShowEditModal(true)}>Edit</button>
      <button onClick={() => setShowDeleteModal(true)}>Delete</button>

      {showEditModal && (
        <EditModal
          ticket={ticket}
          onClose={() => setShowEditModal(false)}
        />
      )}

      {showDeleteModal && (
        <DeleteConfirmModal
          onConfirm={handleDelete}
          onCancel={() => setShowDeleteModal(false)}
        />
      )}
    </>
  );
};
```

---

## Form State Management

### react-hook-form Integration

**Location:** Used in `ProductTicketForm.jsx` and other form components.

**Setup:**
```jsx
import { useForm, useFieldArray } from 'react-hook-form';

const ProductTicketForm = ({ ticket, onSubmit }) => {
  const {
    register,
    handleSubmit,
    control,
    watch,
    setValue,
    formState: { errors, isDirty }
  } = useForm({
    defaultValues: ticket || getDefaultValues()
  });

  // For array fields (like SKU variants)
  const { fields, append, remove } = useFieldArray({
    control,
    name: 'skuVariants'
  });

  return (
    <form onSubmit={handleSubmit(onSubmit)}>
      <input {...register('productName', { required: true })} />
      {errors.productName && <span>Required</span>}

      {fields.map((field, index) => (
        <div key={field.id}>
          <input {...register(`skuVariants.${index}.sku`)} />
          <button type="button" onClick={() => remove(index)}>Remove</button>
        </div>
      ))}

      <button type="button" onClick={() => append({})}>Add SKU</button>
      <button type="submit" disabled={!isDirty}>Save</button>
    </form>
  );
};
```

### Watching Field Values

```jsx
const FormWithConditional = () => {
  const { register, watch, control } = useForm();

  // Watch a single field
  const productionType = watch('productionType');

  // Watch multiple fields
  const [listPrice, standardCost] = watch(['listPrice', 'standardCost']);

  // Calculate derived value
  const margin = useMemo(() => {
    if (!listPrice || !standardCost) return 0;
    return ((listPrice - standardCost) / listPrice) * 100;
  }, [listPrice, standardCost]);

  return (
    <form>
      <select {...register('productionType')}>
        <option value="Produced">Produced</option>
        <option value="Procured">Procured</option>
      </select>

      {productionType === 'Procured' && (
        <input {...register('vendorName')} placeholder="Vendor Name" />
      )}

      <p>Calculated Margin: {margin.toFixed(2)}%</p>
    </form>
  );
};
```

### Programmatic Form Updates

```jsx
const FormWithLookup = () => {
  const { setValue, watch } = useForm();
  const casNumber = watch('chemicalProperties.casNumber');

  const handleCASLookup = async () => {
    try {
      const data = await productAPI.lookupCAS(casNumber);
      if (data) {
        // Batch update multiple fields
        setValue('chemicalProperties.molecularFormula', data.formula);
        setValue('chemicalProperties.molecularWeight', data.weight);
        setValue('productName', data.name);
      }
    } catch (error) {
      toast.error('CAS lookup failed');
    }
  };

  return (
    <form>
      <input {...register('chemicalProperties.casNumber')} />
      <button type="button" onClick={handleCASLookup}>
        Lookup CAS
      </button>
    </form>
  );
};
```

---

## Data Fetching Patterns

### Basic Fetch Pattern

```jsx
const useTickets = (filters) => {
  const [tickets, setTickets] = useState([]);
  const [loading, setLoading] = useState(true);
  const [error, setError] = useState(null);

  const fetchTickets = useCallback(async () => {
    try {
      setLoading(true);
      const response = await productAPI.getTickets(filters);
      setTickets(response.data);
      setError(null);
    } catch (err) {
      setError(err.message);
    } finally {
      setLoading(false);
    }
  }, [filters]);

  useEffect(() => {
    fetchTickets();
  }, [fetchTickets]);

  return { tickets, loading, error, refetch: fetchTickets };
};
```

### Optimistic Updates

```jsx
const handleStatusChange = async (ticketId, newStatus) => {
  // Optimistically update UI
  setTickets(prev => prev.map(t =>
    t._id === ticketId ? { ...t, status: newStatus } : t
  ));

  try {
    await productAPI.updateStatus(ticketId, { status: newStatus });
    toast.success('Status updated');
  } catch (error) {
    // Revert on error
    setTickets(prev => prev.map(t =>
      t._id === ticketId ? { ...t, status: originalStatus } : t
    ));
    toast.error('Failed to update status');
  }
};
```

---

## Caching Strategies

### In-Memory Caching

```jsx
// Simple cache for lookup data
const casCache = new Map();

const lookupCAS = async (casNumber) => {
  // Check cache first
  if (casCache.has(casNumber)) {
    return casCache.get(casNumber);
  }

  // Fetch from API
  const response = await productAPI.lookupCAS(casNumber);

  // Cache the result
  casCache.set(casNumber, response.data);

  return response.data;
};
```

### useMemo for Computed Values

```jsx
const Dashboard = ({ tickets }) => {
  // Memoize expensive computations
  const stats = useMemo(() => ({
    total: tickets.length,
    byStatus: tickets.reduce((acc, t) => {
      acc[t.status] = (acc[t.status] || 0) + 1;
      return acc;
    }, {}),
    avgProcessingTime: calculateAvgTime(tickets)
  }), [tickets]);

  return <StatsDisplay stats={stats} />;
};
```

---

## Local Storage

### Persisted State Hook

```jsx
// hooks/useLocalStorage.js
export const useLocalStorage = (key, initialValue) => {
  const [storedValue, setStoredValue] = useState(() => {
    try {
      const item = localStorage.getItem(key);
      return item ? JSON.parse(item) : initialValue;
    } catch (error) {
      console.error(`Error reading ${key} from localStorage:`, error);
      return initialValue;
    }
  });

  const setValue = (value) => {
    try {
      const valueToStore = value instanceof Function ? value(storedValue) : value;
      setStoredValue(valueToStore);
      localStorage.setItem(key, JSON.stringify(valueToStore));
    } catch (error) {
      console.error(`Error writing ${key} to localStorage:`, error);
    }
  };

  return [storedValue, setValue];
};
```

### Usage Examples

**User Preferences:**
```jsx
const Dashboard = () => {
  const [preferences, setPreferences] = useLocalStorage('dashboardPrefs', {
    itemsPerPage: 25,
    defaultView: 'list',
    showCompleted: false
  });

  return (
    <Settings
      preferences={preferences}
      onUpdate={(key, value) => setPreferences(prev => ({
        ...prev,
        [key]: value
      }))}
    />
  );
};
```

**Session Persistence:**
```jsx
// In AuthContext
useEffect(() => {
  const savedProfileId = localStorage.getItem('selectedProfile');
  const savedProfileData = localStorage.getItem('currentProfileData');

  if (savedProfileId && savedProfileData) {
    setUser(JSON.parse(savedProfileData));
  }
}, []);

const selectProfile = (profile) => {
  setUser(profile);
  localStorage.setItem('selectedProfile', profile.id);
  localStorage.setItem('currentProfileData', JSON.stringify(profile));
};
```

### localStorage Keys Used

| Key | Type | Description |
|-----|------|-------------|
| `selectedProfile` | string | Selected profile ID |
| `currentProfileData` | JSON | Full profile object |
| `dashboardFilters` | JSON | Saved filter preferences |
| `ticketFormDraft` | JSON | Unsaved form data |

---

**Last Updated:** 2026-01-09
**Maintained by:** NPDI Development Team
