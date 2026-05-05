---
title: "Easy Peasy in Depth: State Management Without the Noise"
seoTitle: "Easy Peasy in Depth: State Management Without the Noise"
seoDescription: "A practical guide to Easy Peasy for React and React Native. Covers computed, listeners (actionOn, thunkOn, effectOn), pros and cons, and real-world integrations with TanStack Query and Axios —  all in TypeScript."
datePublished: 2026-04-18T17:20:49.913Z
cuid: cmo4lsfzp006s2cmi0ovzbsuj
slug: easy-peasy-in-depth
tags: react-native, reactjs, redux, state-management, easy-peasy

---

State management in React Native (and React) has always been a hot topic. Redux is powerful but verbose. Context API is built-in but can cause unnecessary re-renders at scale. [Easy Peasy](https://easy-peasy.dev/) sits in between: it's a Redux-based library that strips away most of the boilerplate while keeping the underlying power of Redux DevTools and the Flux pattern.

In this article we'll cover:

- What Easy Peasy is and how to set it up
- Its core concepts with TypeScript examples
- Computed: derived state done right
- Listeners: `actionOn`, `thunkOn`, and `effectOn`
- How to integrate it with [TanStack Query](https://tanstack.com/query/latest) (React Query) for server state
- How to integrate it with [Axios](https://axios-http.com/) for API calls
- Pros and cons from real-world usage

---

## What is Easy Peasy?

Easy Peasy is a thin abstraction layer on top of Redux. You define your entire state as a plain object (the model), and Easy Peasy generates the store, reducers, and actions for you — no boilerplate, no action type constants, no switch statements.

```bash
npm install easy-peasy
```

---

## Setting Up the Store

Let's build a small example: a task manager app.

```typescript
// store/tasksModel.ts
import { action, Action } from 'easy-peasy';

export interface Task {
  id: string;
  title: string;
  completed: boolean;
}

export interface TasksModel {
  items: Task[];
  addTask: Action<TasksModel, Task>;
  toggleTask: Action<TasksModel, string>;
  removeTask: Action<TasksModel, string>;
}

const tasksModel: TasksModel = {
  items: [],

  addTask: action((state, task) => {
    state.items.push(task);
  }),

  toggleTask: action((state, id) => {
    const task = state.items.find(t => t.id === id);
    if (task) task.completed = !task.completed;
  }),

  removeTask: action((state, id) => {
    state.items = state.items.filter(t => t.id !== id);
  }),
};

export default tasksModel;
```

```typescript
// store/index.ts
import { createStore, createTypedHooks } from 'easy-peasy';
import tasksModel, { TasksModel } from './tasksModel';

export interface StoreModel {
  tasks: TasksModel;
}

const store = createStore<StoreModel>({
  tasks: tasksModel,
});

export const { useStoreActions, useStoreState } = createTypedHooks<StoreModel>();

export default store;
```

```typescript
// App.tsx
import React from 'react';
import { StoreProvider } from 'easy-peasy';
import store from './store';
import TaskList from './TaskList';

export default function App() {
  return (
    <StoreProvider store={store}>
      <TaskList />
    </StoreProvider>
  );
}
```

And consuming it in a component:

```typescript
// TaskList.tsx
import React from 'react';
import { View, Text, Button } from 'react-native';
import { useStoreState, useStoreActions } from './store';

export default function TaskList() {
  const tasks = useStoreState(state => state.tasks.items);
  const toggleTask = useStoreActions(actions => actions.tasks.toggleTask);

  return (
    <View>
      {tasks.map(task => (
        <View key={task.id}>
          <Text style={{ textDecorationLine: task.completed ? 'line-through' : 'none' }}>
            {task.title}
          </Text>
          <Button title="Toggle" onPress={() => toggleTask(task.id)} />
        </View>
      ))}
    </View>
  );
}
```

No `mapStateToProps`, no `dispatch`, no action creators. Just typed hooks.

---

## Core Concepts

| Concept | Description |
|---|---|
| `action` | Synchronous state mutation. Uses Immer under the hood (direct mutations are safe). |
| `thunk` | Async side effects. Ideal for API calls. Has access to `actions`, `getState`, `injections`. |
| `computed` | Derived state. Cached and recalculated only when dependencies change. |
| `persist` | Built-in persistence with AsyncStorage or any storage engine. |

Here's a quick `thunk` example:

```typescript
import { thunk, Thunk } from 'easy-peasy';

export interface TasksModel {
  // ...previous fields
  fetchTasks: Thunk<TasksModel>;
}

const tasksModel: TasksModel = {
  // ...previous fields
  fetchTasks: thunk(async (actions) => {
    const response = await fetch('https://api.example.com/tasks');
    const tasks = await response.json();
    tasks.forEach((task: Task) => actions.addTask(task));
  }),
};
```

---

## Computed: Derived State Done Right

`computed` lets you derive values from your state without duplicating data. It's cached — Easy Peasy only recalculates when the selected dependencies actually change.

```typescript
import { computed, Computed } from 'easy-peasy';

export interface TasksModel {
  items: Task[];
  completedTasks: Computed<TasksModel, Task[]>;
  pendingCount: Computed<TasksModel, number>;
  hasCompleted: Computed<TasksModel, boolean>;
}

const tasksModel: TasksModel = {
  items: [],

  completedTasks: computed(state =>
    state.items.filter(t => t.completed)
  ),

  pendingCount: computed(state =>
    state.items.filter(t => !t.completed).length
  ),

  hasCompleted: computed(state =>
    state.items.some(t => t.completed)
  ),
};
```

Consuming computed values feels identical to consuming regular state:

```typescript
const pendingCount = useStoreState(state => state.tasks.pendingCount);
const completedTasks = useStoreState(state => state.tasks.completedTasks);
```

### Cross-model computed

What makes `computed` really powerful is that it can pull from other slices of the store using a second argument — a selector array:

```typescript
import { computed, Computed } from 'easy-peasy';
import { StoreModel } from './index';

export interface TasksModel {
  items: Task[];
  visibleTasks: Computed<TasksModel, Task[], StoreModel>;
}

const tasksModel: TasksModel = {
  items: [],

  visibleTasks: computed(
    [
      state => state.items,
      (state, storeState) => storeState.filters.activeFilter,
    ],
    (items, activeFilter) => {
      if (activeFilter === 'completed') return items.filter(t => t.completed);
      if (activeFilter === 'pending') return items.filter(t => !t.completed);
      return items;
    }
  ),
};
```

This is clean, typed, and only re-runs when `items` or `activeFilter` actually changes — not on every state update.

### A note on computed and re-renders

Because `computed` values are memoized inside the store (not inside the component), components using `useStoreState` with a computed property will only re-render when the computed output changes. This is a meaningful performance win over deriving values inline inside `useStoreState` selectors or `useMemo`.

---

## Listeners: Reacting to Actions Across the Store

This is one of Easy Peasy's most underrated features, and one of my favourites.

Listeners let you react to actions that happen anywhere in the store — without coupling the models together. There are three types: `actionOn` (synchronous), `thunkOn` (asynchronous), and `effectOn` (side effects without dispatching).

### actionOn

`actionOn` listens for one or more actions and runs a synchronous state update in response. Think of it as a cross-slice reducer.

A classic use case: clearing related state when the user logs out.

```typescript
// store/authModel.ts
import { action, Action } from 'easy-peasy';

export interface AuthModel {
  token: string | null;
  login: Action<AuthModel, string>;
  logout: Action<AuthModel>;
}

const authModel: AuthModel = {
  token: null,
  login: action((state, token) => { state.token = token; }),
  logout: action((state) => { state.token = null; }),
};

export default authModel;
```

```typescript
// store/tasksModel.ts
import { action, Action, actionOn, ActionOn } from 'easy-peasy';
import { StoreModel } from './index';

export interface TasksModel {
  items: Task[];
  addTask: Action<TasksModel, Task>;
  onLogout: ActionOn<TasksModel, StoreModel>;
}

const tasksModel: TasksModel = {
  items: [],

  addTask: action((state, task) => {
    state.items.push(task);
  }),

  onLogout: actionOn(
    (actions, storeActions) => storeActions.auth.logout,
    (state) => {
      state.items = [];
    }
  ),
};
```

When `auth.logout` fires, `tasks.onLogout` runs automatically and wipes the task list. No explicit call from the logout handler, no coupling between models — the tasks model simply declares its interest in that event.

You can also listen to **multiple actions at once**:

```typescript
onReset: actionOn(
  (actions, storeActions) => [
    storeActions.auth.logout,
    storeActions.auth.sessionExpired,
  ],
  (state) => {
    state.items = [];
  }
),
```

### thunkOn

`thunkOn` is the async equivalent. It listens for actions and can perform side effects in response — API calls, logging, analytics, triggering other thunks.

A common use case: logging an analytics event every time a task is completed.

```typescript
// store/analyticsModel.ts
import { thunkOn, ThunkOn } from 'easy-peasy';
import { StoreModel } from './index';

export interface AnalyticsModel {
  onTaskToggled: ThunkOn<AnalyticsModel, void, StoreModel>;
}

const analyticsModel: AnalyticsModel = {
  onTaskToggled: thunkOn(
    (actions, storeActions) => storeActions.tasks.toggleTask,
    async (actions, target) => {
      const taskId = target.payload;
      await analytics.track('task_toggled', { taskId });
    }
  ),
};

export default analyticsModel;
```

The `target` object gives you access to:
- `target.payload` — the payload the original action received
- `target.type` — the action type string (useful when listening to multiple actions)
- `target.result` — the return value of the action (for `thunkOn` listening to thunks)

### effectOn

`effectOn` is the lightest listener. It's designed for pure side effects — things like syncing to external storage, logging, or updating a non-React service — without needing to dispatch any actions back into the store.

Think of it as `useEffect` but scoped to store state changes instead of the React component lifecycle.

```typescript
import { effectOn, EffectOn } from 'easy-peasy';
import { StoreModel } from './index';

export interface TasksModel {
  items: Task[];
  syncEffect: EffectOn<TasksModel, StoreModel>;
}

const tasksModel: TasksModel = {
  items: [],

  syncEffect: effectOn(
    [state => state.items],
    async (dispatch, change) => {
      const { currentValue } = change;
      await AsyncStorage.setItem('tasks', JSON.stringify(currentValue));
    }
  ),
};
```

The `change` object gives you:
- `change.currentValue` — the new value of the selected state
- `change.previousValue` — what it was before
- `change.action` — the action that triggered the change

A subtle but important difference from `thunkOn`: `effectOn` reacts to **state changes**, not to specific actions. You pass a state selector, and it fires whenever that slice of state changes — regardless of which action caused it.

This makes it ideal for cases where you don't care *what* changed it, only *that* it changed:

```typescript
// Fires whenever items changes, regardless of whether it was
// addTask, removeTask, toggleTask, or fetchTasks that caused it
syncEffect: effectOn(
  [state => state.items],
  async (dispatch, change) => {
    await syncToServer(change.currentValue);
  }
),
```

Compare that to `thunkOn`, where you'd have to explicitly list every action that modifies `items`. With `effectOn`, the selector handles it automatically.

### Why listeners matter

Without listeners, cross-slice reactions force you to couple models explicitly — the auth model would need to know about tasks, analytics, notifications, and anything else that cares about logout. That's the wrong direction.

With listeners, the dependency is inverted: each model declares what it reacts to, and the auth model stays clean. This is especially valuable as your store grows.

It's a pattern similar to event-driven architecture, but fully typed and co-located with your state.

---

## Integration with TanStack Query (React Query)

The best pattern I've used in production is: **Easy Peasy for client state, TanStack Query for server state**. They don't compete — they complement.

The rule is simple:
- Data that lives on the server (fetched from an API, cached, invalidated) → TanStack Query
- Data that's purely client-side (current user session, UI preferences, navigation state, form draft) → Easy Peasy

```typescript
// store/uiModel.ts — client state only
import { action, Action } from 'easy-peasy';

export interface UIModel {
  selectedTaskId: string | null;
  sidebarOpen: boolean;
  setSelectedTask: Action<UIModel, string | null>;
  toggleSidebar: Action<UIModel>;
}

const uiModel: UIModel = {
  selectedTaskId: null,
  sidebarOpen: false,
  setSelectedTask: action((state, id) => { state.selectedTaskId = id; }),
  toggleSidebar: action((state) => { state.sidebarOpen = !state.sidebarOpen; }),
};

export default uiModel;
```

```typescript
// hooks/useTasks.ts — server state via TanStack Query
import { useQuery, useMutation, useQueryClient } from '@tanstack/react-query';
import { fetchTasks, createTask, Task } from '../api/tasks';

export function useTasks() {
  return useQuery({ queryKey: ['tasks'], queryFn: fetchTasks });
}

export function useCreateTask() {
  const queryClient = useQueryClient();
  return useMutation({
    mutationFn: createTask,
    onSuccess: () => queryClient.invalidateQueries({ queryKey: ['tasks'] }),
  });
}
```

```typescript
// TaskList.tsx — combining both
import React from 'react';
import { View, Text, ActivityIndicator } from 'react-native';
import { useTasks } from '../hooks/useTasks';
import { useStoreState, useStoreActions } from '../store';

export default function TaskList() {
  const { data: tasks, isLoading } = useTasks();
  const selectedTaskId = useStoreState(state => state.ui.selectedTaskId);
  const setSelectedTask = useStoreActions(actions => actions.ui.setSelectedTask);

  if (isLoading) return <ActivityIndicator />;

  return (
    <View>
      {tasks?.map(task => (
        <Text
          key={task.id}
          onPress={() => setSelectedTask(task.id)}
          style={{ fontWeight: task.id === selectedTaskId ? 'bold' : 'normal' }}
        >
          {task.title}
        </Text>
      ))}
    </View>
  );
}
```

This pattern keeps your Easy Peasy store lean and focused. No loading spinners or error flags cluttering your models.

---

## Integration with Axios

When using Easy Peasy thunks, injections are the cleanest way to keep your API layer decoupled from your store logic.

**Step 1: Create an Axios instance**

```typescript
// api/client.ts
import axios from 'axios';

const apiClient = axios.create({
  baseURL: 'https://api.example.com',
  timeout: 10000,
});

apiClient.interceptors.request.use(config => {
  const token = getAuthToken(); // your token retrieval logic
  if (token) config.headers.Authorization = `Bearer ${token}`;
  return config;
});

export default apiClient;
```

**Step 2: Inject the client into the store**

```typescript
// store/index.ts
import { createStore } from 'easy-peasy';
import apiClient from '../api/client';
import model, { StoreModel } from './model';

const store = createStore<StoreModel>(model, {
  injections: { apiClient },
});

export default store;
```

**Step 3: Use the injected client in thunks**

```typescript
// store/tasksModel.ts
import { thunk, Thunk, action, Action } from 'easy-peasy';
import { AxiosInstance } from 'axios';

interface Injections {
  apiClient: AxiosInstance;
}

export interface TasksModel {
  items: Task[];
  addTask: Action<TasksModel, Task>;
  fetchTasks: Thunk<TasksModel, void, Injections>;
}

const tasksModel: TasksModel = {
  items: [],

  addTask: action((state, task) => {
    state.items.push(task);
  }),

  fetchTasks: thunk(async (actions, _payload, { injections }) => {
    const { apiClient } = injections;
    const { data } = await apiClient.get<Task[]>('/tasks');
    data.forEach(task => actions.addTask(task));
  }),
};
```

The advantage of injections over direct imports: your thunks are now fully testable — you can mock `apiClient` in tests without touching the actual module.

```typescript
// tasksModel.test.ts
import { createStore } from 'easy-peasy';
import tasksModel from './tasksModel';

const mockApiClient = {
  get: jest.fn().mockResolvedValue({
    data: [{ id: '1', title: 'Test Task', completed: false }],
  }),
};

const store = createStore({ tasks: tasksModel }, {
  injections: { apiClient: mockApiClient },
});

test('fetchTasks populates store', async () => {
  await store.dispatch.tasks.fetchTasks();
  expect(store.getState().tasks.items).toHaveLength(1);
  expect(store.getState().tasks.items[0].title).toBe('Test Task');
});
```

---

## Pros

**1. Minimal boilerplate**
You define state and logic in one place. No action type constants, no switch reducers, no separate files for each concern.

**2. TypeScript-first**
`createTypedHooks` gives you full type inference across the entire store. Your IDE will autocomplete both state paths and actions.

**3. Immer under the hood**
You can mutate state directly inside `action` handlers — Easy Peasy wraps everything in Immer, so you always get immutable updates without the spread operator noise.

**4. Redux DevTools compatible**
Because it's Redux under the hood, the Redux DevTools extension works out of the box. Time travel debugging, action history, state snapshots — all available.

**5. Built-in persistence**
The `persist` helper integrates directly into the model. No extra middleware needed:

```typescript
import { persist } from 'easy-peasy';

const tasksModel = persist<TasksModel>({
  items: [],
  // ...actions
}, {
  storage: AsyncStorage,
});
```

**6. Thunks have access to the full store — typed**
Easy Peasy thunks receive `getState()` (local model), `getStoreState()` (full store), and `getStoreActions()` (typed dispatch for any model) as part of the third argument. All fully typed. This is a meaningful upgrade over Redux Thunk, which only provides an untyped `getState` for the full store with no equivalent of `getStoreActions`.

---

## Cons

**1. Not ideal for server state**
Easy Peasy is great for client state (UI state, user preferences, auth tokens). Using thunks to manage loading/error/data for every API call gets repetitive fast. Pairing it with TanStack Query, as covered above, is the right solution.

**2. Computed values can be tricky to debug**
`computed` properties are cached and rely on selector functions. If you're not careful about what you select as a dependency, you can end up with stale derived values that are hard to trace.

**3. Smaller ecosystem**
Redux has a massive ecosystem (Redux Toolkit, RTK Query, hundreds of middleware). Easy Peasy's ecosystem is much smaller. If you need advanced patterns (event sourcing, sagas, complex middleware), you may hit a wall.

**4. Thunks are not cancellable**
Unlike Redux-Saga or RTK Query, Easy Peasy thunks don't support cancellation. If a component unmounts before a thunk completes, you'll need to handle that manually with an `isMounted` flag or an `AbortController`.

**5. Increased bundle size**
Easy Peasy bundles `redux`, `redux-thunk`, `immer`, and `use-sync-external-store` as direct dependencies (v6.1.0). The combined runtime cost is roughly **13–15KB gzipped** — noticeably heavier than lighter alternatives like Zustand (~3KB) or Jotai (~3KB), and comparable to Redux Toolkit (~12KB). For React Native apps this is generally not a concern since the bundle isn't transferred over the wire on every load, but for web apps with performance budgets it's worth factoring in. The main contributor to the size is Immer — which is also responsible for the direct mutation syntax that makes Easy Peasy so ergonomic.

---

## Conclusion

Easy Peasy is an excellent choice for client state in React Native apps — especially for small-to-medium sized projects or teams that want Redux's reliability without its ceremony. TypeScript support is first-class, Immer mutations keep the code readable, and Redux DevTools work out of the box.

Its main weakness is server state management, but that's not a flaw — it's a scope decision. Pair it with TanStack Query and you cover both worlds cleanly.

The injection pattern with Axios keeps your store logic decoupled and testable, which matters as your project scales.

If you have any corrections or suggestions, they're more than welcome. Happy coding! 🚀

---

*Resources:*
- [Easy Peasy docs](https://easy-peasy.dev/)
- [TanStack Query docs](https://tanstack.com/query/latest)
- [Axios docs](https://axios-http.com/)
- [Immer docs](https://immerjs.github.io/immer/)
