# Report Builder UI — Codebase Knowledge Transfer

> A comprehensive technical guide for developers onboarding to the Report Builder UI codebase. This document covers architecture, data flow, API patterns, Redux state management, UI components, and known issues — organized for quick reference and deep understanding.

---

## Table of Contents

1. [Project Overview](#project-overview)
2. [Phase 1 — Foundation](#phase-1--foundation)
   - [src/main.jsx — Entry Point](#srcmainjsx--entry-point)
   - [src/App.jsx — Root Component](#srcappjsx--root-component)
   - [src/store/store.js — Redux Store](#srcstorestorej--redux-store)
3. [Phase 2 — API Layer](#phase-2--api-layer)
   - [apiConfig.js — Configuration](#apiconfigjs--configuration)
   - [client.js — Axios Instance](#clientjs--axios-instance)
   - [request.js — Request Interceptor](#requestjs--request-interceptor)
   - [templateService.js](#templateservicejs)
   - [templatePageService.js](#templatepageservicejs)
   - [templateVersionService.js](#templateversionservicejs)
4. [Phase 3 — Store Layer (Redux Slice)](#phase-3--store-layer-redux-slice)
   - [State Shape](#state-shape)
   - [Synchronous Reducers](#synchronous-reducers)
   - [Async Thunks](#async-thunks)
   - [Selectors](#selectors)
5. [Phase 4 — UI Layer](#phase-4--ui-layer)
   - [TemplateList.jsx](#templatelistjsx)
   - [NewTemplateWizard.jsx](#newtemplatewizardjsx)
   - [TemplateDetail.jsx](#templatedetailjsx)
   - [TemplateEditor.jsx](#templateeditorjsx)
   - [DesignerCanvas.jsx](#designercanvasjsx)
6. [Happy Path Walkthrough](#happy-path-walkthrough)
7. [Debugging Guide](#debugging-guide)
8. [Architectural Risks & Known Issues](#architectural-risks--known-issues)
9. [Summary](#summary)

---

## Project Overview

The **Report Builder UI** is a React-based application that allows users to create, version, design, and publish PDF report templates. It uses:

- **React + Vite** as the frontend framework and build tool
- **Redux Toolkit** for global state management (a single slice: `templates`)
- **Mantine UI** as the component and design system library
- **React Router v6** for client-side navigation
- **Axios** for HTTP communication with the backend REST API
- **`@baylor/pdfme-ui`** — a third-party imperative PDF designer component embedded inside `DesignerCanvas`

The application follows a layered architecture:

```
Entry Point (main.jsx)
    └── App.jsx (Providers + Router)
         ├── TemplateList      → browse templates
         ├── TemplateDetail    → view/manage versions
         └── TemplateEditor    → design PDF layouts
              └── DesignerCanvas  → pdfme designer bridge
```

---

## Phase 1 — Foundation

### `src/main.jsx` — Entry Point

**Purpose:** Bootstraps the React application by mounting the root component tree into the DOM.

```jsx
// src/main.jsx
createRoot(document.getElementById("root")).render(
  <StrictMode>
    <App />
  </StrictMode>,
);
```

**Key Notes:**

- Has no inputs, no props, no Redux access — purely bootstraps the app.
- Depends on `index.html` having a `<div id="root">` element.
- Wraps everything in `<React.StrictMode>`, which **intentionally double-invokes** component effects in development mode to catch side-effect bugs.

> ⚠️ **StrictMode double-invocation:** If you're debugging a `useEffect` that appears to fire twice (e.g., creating two templates), this is the reason. `TemplateEditor` uses a `workflowInitiatedRef` guard to prevent this — see [TemplateEditor.jsx](#templateeditorjsx).

---

### `src/App.jsx` — Root Component

**Purpose:** Wires together the three global providers and defines the complete application route map.

**Providers applied (outermost → innermost):**

1. `<Provider store={store}>` — Redux
2. `<MantineProvider>` — Mantine design system
3. `<Notifications position="top-right" zIndex={2077}>` — Toast container
4. `<BrowserRouter>` — React Router

**Route Map:**

| Path                                                 | Component        | Workflow                                   |
| ---------------------------------------------------- | ---------------- | ------------------------------------------ |
| `/`                                                  | `TemplateList`   | Browse and search templates                |
| `/template/:id`                                      | `TemplateDetail` | View template versions                     |
| `/designer`                                          | `TemplateEditor` | Create a brand-new template                |
| `/editor/:templateId`                                | `TemplateEditor` | Edit an existing version                   |
| `/editor/new-version/:baseTemplateId/:baseVersionId` | `TemplateEditor` | Create a new version from an existing base |

> ⚠️ **Hidden gotcha:** `/designer` and `/editor/:templateId` both render `TemplateEditor` but trigger completely different internal workflows. The branching happens at runtime inside `TemplateEditor` by checking `location.pathname === '/designer'`. A new developer reading the route map alone won't see this divergence.

**Startup Side Effect — `SchemaInitializer`:**

```js
// App.jsx — SchemaInitializer component
useEffect(() => {
  initializeDefaultSchemas(store.dispatch); // Seeds WGS schema into Redux before any route renders
}, []);
```

- This fires **before any route renders** and passes the `dispatch` function directly to a utility (bypassing `useDispatch`).
- If `initializeDefaultSchemas` throws, there is **no error boundary** catching it here.

---

### `src/store/store.js` — Redux Store

**Purpose:** Configures and exports the single Redux store.

```js
// store.js
export const store = configureStore({
  reducer: {
    templates: templatesReducer,
  },
  middleware: (getDefaultMiddleware) =>
    getDefaultMiddleware({
      serializableCheck: {
        ignoredActions: ["persist/PERSIST", "persist/REHYDRATE"],
      },
    }),
});
```

**Redux State Shape:**

```js
{
  templates: {
    templates: [],           // Array of template objects (with nested versions[])
    isLoading: false,        // Global loading flag
    isVersionsLoading: false,// Separate loading flag for version fetches
    error: null,             // Last error message string
    pagination: {
      total: 0,
      page: 1,
      limit: 10,
      totalPages: 0
    }
  }
}
```

> ⚠️ **`persist/PERSIST` and `persist/REHYDRATE` are ignored** in `serializableCheck`, but `redux-persist` is not actually integrated. This looks like residual boilerplate and can be removed.

> ⚠️ **Tight coupling — `store.getState()` used directly:** `TemplateEditor.jsx` (line 589) imports the raw store object and calls `store.getState()` imperatively inside a `useEffect`. This bypasses React's reactivity — if the selector's output changes, **the component won't re-render**.

---

## Phase 2 — API Layer

### `apiConfig.js` — Configuration

**Purpose:** Centralized configuration object for all API URLs, headers, and environment flags.

```js
// Environment-driven base URL
const BASE_URL = import.meta.env.VITE_API_BASE_URL || "http://localhost:3000";
const TIMEOUT = import.meta.env.VITE_API_TIMEOUT || 30000;
```

**Exported shape:**

```js
apiConfig = {
  baseURL,
  timeout,
  headers: { 'Content-Type': 'application/json' },
  endpoints: {
    templates: { getAll, getById, create, update, delete, ... },
    templateVersions: { getPdfSchema, ... },
  },
  isDevelopment,
  isProduction,
}
```

> ⚠️ **Maintenance trap:** `templateVersionService.js` hard-codes URL strings directly (e.g., `` `/v1/templates/${templateId}/versions` ``), while `templateService.js` uses `apiConfig.endpoints.*`. If the API base path changes, only `templateService` gets the fix automatically — `templateVersionService` stays broken.

---

### `client.js` — Axios Instance

**Purpose:** Creates a configured Axios instance and exports convenience helper functions.

```js
// Named exports used by all service files
export const get = (url, config) => apiClient.get(url, config);
export const post = (url, data, config) => apiClient.post(url, data, config);
export const put = (url, data, config) => apiClient.put(url, data, config);
export const patch = (url, data, config) => apiClient.patch(url, data, config);
export const del = (url, config) => apiClient.delete(url, config);
```

The helper-function pattern is good design — service files consume a stable interface without knowing Axios internals.

---

### `request.js` — Request Interceptor

**Purpose:** Attaches a request timestamp and gates verbose logging behind `isDevelopment`.

```js
// Attaches timing metadata to every outgoing request
config.metadata = { startTime: new Date() };
```

> ⚠️ **Auth is disabled:** Bearer token injection (lines 16–19) is commented out. **No authentication headers are sent with any API request.** This is either intentional (localhost-only API) or an incomplete feature. Do not assume this codebase is production-auth-ready.

---

### `templateService.js`

**Purpose:** Service layer for Template-level CRUD and version operations against `v1/templates`.

| Method                                       | HTTP   | Description              |
| -------------------------------------------- | ------ | ------------------------ |
| `getAll(params)`                             | GET    | Fetch paginated list     |
| `getById(id)`                                | GET    | Fetch single template    |
| `create(data)`                               | POST   | Create template          |
| `update(id, data)`                           | PATCH  | Partial update (not PUT) |
| `delete(id)`                                 | DELETE | Remove template          |
| `getVersions(templateId)`                    | GET    | List versions            |
| `createVersion(templateId, data)`            | POST   | Add a version            |
| `updateVersion(templateId, versionId, data)` | PATCH  | Update version           |
| `deleteVersion(templateId, versionId)`       | DELETE | Remove version           |
| `publish(templateId, versionId)`             | POST   | Publish a version        |
| `uploadFile(templateId, file)`               | POST   | File upload              |
| `search(params)`                             | GET    | Search templates         |
| `getPdfSchema(templateId, versionId)`        | GET    | Load PDF schema          |

> ⚠️ **Categorization mismatch:** `getPdfSchema` lives in `templateService` but actually hits a `templateVersions` endpoint (`endpoints.templateVersions.getPdfSchema`). Anyone searching for schema-loading code in the version service won't find it there.

---

### `templatePageService.js`

**Purpose:** All HTTP operations for `TemplatePage` entities.

| Method                             | HTTP   | Notes                                |
| ---------------------------------- | ------ | ------------------------------------ |
| `create(versionId, pageData)`      | POST   | Single page                          |
| `createBulk(versionId, pagesData)` | POST   | Wraps in `{ pages: [...] }` envelope |
| `getAll(versionId)`                | GET    | —                                    |
| `getById(pageId)`                  | GET    | —                                    |
| `update(pageId, data)`             | PUT    | Full replace (not PATCH)             |
| `delete(pageId)`                   | DELETE | —                                    |
| `reorder(versionId, pageOrder)`    | PUT    | Full replace                         |

> ⚠️ **Inconsistent update semantics:** `templatePageService.update` uses `PUT` (full replace), while `templateService.update` uses `PATCH` (partial). Verify with the API team whether this is intentional.

---

### `templateVersionService.js`

**Purpose:** HTTP operations for the `TemplateVersion` sub-resource — schema save/load, publish, clone, comments.

> ⚠️ **Dead import:** Line 6 contains `import { version } from 'react'`. This is unused dead code that also shadows any local `version` variable name. It should be removed immediately.

> ⚠️ **URL inconsistency:** `addComment` posts to `.../comment` (singular) while `getComments` fetches from `.../comments` (plural). Confirm with the API team whether this is intentional REST semantics.

> ⚠️ **Base path mismatch:** `saveSchema` uses `/api/TemplateVersions/${versionId}/schema` while all other endpoints use `/v1/...`. This suggests schema storage may be a separate legacy service.

---

## Phase 3 — Store Layer (Redux Slice)

### State Shape

The Redux state is **denormalized** — versions live nested inside each template object rather than in a separate normalized map.

```js
// Template object shape
{
  id, name, description, totalFields, lastModified,
  author, isActive, metadata, createdAt, updatedAt,
  versions: [
    {
      id, version, lastModified, author, status, description,
      published, publishedAt, createdAt, updatedAt,
      rendererEngine, designSettings, schema, comments?
    }
  ]
}
```

**Trade-off:** Fast reads for "give me template X and its versions," but any version update requires scanning the entire `templates` array to find the parent. Fine at low volume; consider ID-map normalization at scale.

---

### Synchronous Reducers

| Reducer                                                        | What It Does                                                                                                                |
| -------------------------------------------------------------- | --------------------------------------------------------------------------------------------------------------------------- |
| `setLoading(bool)`                                             | Manually override the loading flag                                                                                          |
| `setError(message)`                                            | Set error string, clear loading                                                                                             |
| `addNewVersion({ templateId, baseVersionId, author, schema })` | Creates a new version in-memory (legacy local-only flow). Promotes old `active` → `stable`.                                 |
| `updateVersion({ versionId, updates })`                        | Patches a version in-place. If it's the active version, propagates `totalFields` and `lastModified` to the parent template. |
| `updateVersionSchema({ versionId, schema })`                   | Updates schema and recalculates field count by summing keys across page schemas.                                            |
| `initializeVersionSchema({ versionId, schema })`               | Identical logic to `updateVersionSchema` but named separately for WGS initialization.                                       |
| `deleteVersion({ versionId })`                                 | Removes from parent's `versions[]`. If deleted version was active, promotes `versions[0]`.                                  |

> ⚠️ `updateVersionSchema` and `initializeVersionSchema` contain **identical logic** — unnecessary duplication. A future refactor should merge them into a single reducer.

---

### Async Thunks

| Thunk                                                                          | API Call                                 | Key Behavior                                                        |
| ------------------------------------------------------------------------------ | ---------------------------------------- | ------------------------------------------------------------------- |
| `fetchTemplates(params)`                                                       | `GET /v1/templates`                      | Transforms API → UI shape. Updates pagination.                      |
| `createTemplate(data)`                                                         | `POST /v1/templates`                     | Prepends new template via `unshift`.                                |
| `updateTemplate({ id, data })`                                                 | `PATCH /v1/templates/:id`                | Finds by index, merges selected fields.                             |
| `deleteTemplate(id)`                                                           | `DELETE /v1/templates/:id`               | Filters out by ID.                                                  |
| `fetchTemplateVersions({ templateId, params })`                                | `GET /v1/templates/:id/versions`         | Transforms and stores versions inside the matching template.        |
| `createTemplateVersion({ templateId, versionData })`                           | `POST /v1/templates/:id/versions`        | `unshift`s new version.                                             |
| `updateTemplateVersion({ templateId, versionId, data })`                       | `PATCH /v1/templates/:id/versions/:vid`  | Updates version in-place.                                           |
| `deleteTemplateVersion({ templateId, versionId })`                             | `DELETE /v1/templates/:id/versions/:vid` | Filters version out. Backend manages status promotion.              |
| `cloneTemplateVersion({ ..., newVersionLabel, copyPages, copyExportOptions })` | `POST .../clone`                         | Adds clone at top of versions, hard-codes `status='draft'`.         |
| `fetchVersionComments({ templateId, versionId })`                              | `GET .../comments`                       | Attaches comments to the matching version object.                   |
| `saveVersionSchema({ versionId, schema, updatedBy })`                          | `PUT /api/TemplateVersions/:id/schema`   | ⚠️ **COMMENTED OUT in TemplateEditor** — save only posts a comment. |
| `loadVersionSchema(versionId)`                                                 | `GET /api/TemplateVersions/:id/schema`   | Fallback when `getPdfSchema` API fails.                             |
| `createTemplatePages({ versionId, pagesData })`                                | `POST /v1/template-pages/bulk`           | Bulk page creation, injects `versionId` into each page.             |
| `fetchTemplatePages(versionId)`                                                | `GET /v1/template-pages?versionId=:id`   | Fetches pages for a version.                                        |

> 🚨 **Critical Bug:** In `TemplateEditor.handleSave` (line 980–984), the `saveVersionSchema` dispatch is commented out. **Clicking "Save" only posts a comment — the schema is never persisted to the backend.** This is the biggest functional gap in the codebase. Start debugging here by uncommenting line 980 and verifying the schema endpoint.

---

### Selectors

```js
// Simple selectors
selectAllTemplates(state)       → state.templates.templates[]
selectTemplatesLoading(state)   → state.templates.isLoading
selectVersionsLoading(state)    → state.templates.isVersionsLoading
selectTemplatesError(state)     → state.templates.error
selectTemplatesPagination(state)→ state.templates.pagination

// Factory selectors (return a new function each call — not memoized by default)
selectTemplateById(templateId)(state)              → template object | undefined
selectVersionById(versionId)(state)                → { template, version } | null
selectActiveVersionByTemplate(templateId)(state)   → active version | versions[0]
selectVersionSchema(versionId)(state)              → version.schema | null
```

> ⚠️ **Performance note:** `selectVersionById`, `selectActiveVersionByTemplate`, and `selectVersionSchema` all perform **full linear scans** of `state.templates.templates`. As factory selectors, they don't memoize with standard `useSelector`. Wrap with `createSelector` from Reselect if performance becomes an issue.

---

## Phase 4 — UI Layer

### `TemplateList.jsx`

**Route:** `/`

**Purpose:** Application home page — paginated, searchable template list with wizard entry point.

**Redux consumed:**

| Selector                    | Used For                       |
| --------------------------- | ------------------------------ |
| `selectAllTemplates`        | Render template cards          |
| `selectTemplatesLoading`    | Opacity overlay while loading  |
| `selectTemplatesError`      | Error banner with retry button |
| `selectTemplatesPagination` | Pagination controls            |

**Thunks dispatched:**

- `fetchTemplates({ page, limit, name? })` — on mount and whenever `currentPage` or `debouncedSearch` (500ms debounce via `@mantine/hooks`) changes.
- Uses a **smart page-reset pattern:** when the search term changes, `currentPage` resets to `1` before dispatching — avoids showing page 3 of a brand-new search.

```js
// TemplateList.jsx — smart search + page reset pattern
useEffect(() => {
  const searchChanged = debouncedSearch !== prevSearchRef.current;
  const page = searchChanged ? 1 : currentPage;
  if (searchChanged) prevSearchRef.current = debouncedSearch;
  dispatch(
    fetchTemplates({ page, limit: 10, name: debouncedSearch || undefined }),
  );
}, [currentPage, debouncedSearch]);
```

> ⚠️ `fetchTemplateVersions` is imported but never called from `TemplateList` — it's dead code.

---

### `NewTemplateWizard.jsx`

**Purpose:** Modal form that handles both **creating a new template** and **editing existing template metadata**.

**Props:**

| Prop           | Type     | Default | Description                      |
| -------------- | -------- | ------- | -------------------------------- |
| `isOpen`       | boolean  | —       | Controls modal visibility        |
| `onClose`      | function | —       | Close callback                   |
| `editMode`     | boolean  | `false` | Switches between create and edit |
| `templateData` | object   | `null`  | Pre-fills form in edit mode      |

**Create flow (no API call here):**

```js
navigate('/designer', {
  state: { templateMetadata, hospitalCode, testCode, ... }
});
// The actual template creation happens in TemplateEditor.createTemplateWorkflow
```

> ⚠️ **Misleading UX:** A "Template created successfully!" toast fires immediately on `navigate()` — **before the template exists in the backend.** If `createTemplateWorkflow` in `TemplateEditor` subsequently fails, the user already saw a success message.

**Edit flow:**

```js
dispatch(updateTemplate({ id, data })).unwrap();
```

> ⚠️ **Dead code:** Lines 67–135 contain an entire commented-out `fetchTestConfiguration` workflow. This should either be implemented or removed.

---

### `TemplateDetail.jsx`

**Route:** `/template/:id`

**Purpose:** Version management page — CRUD for versions (create, rename, delete, duplicate) plus comments.

**Thunks dispatched:**

| Thunk                                                                     | When                                                     |
| ------------------------------------------------------------------------- | -------------------------------------------------------- |
| `fetchTemplates()`                                                        | On mount if template not in Redux (handles page refresh) |
| `fetchTemplateVersions({ templateId, params })`                           | On mount + after every mutation                          |
| `createTemplateVersion({ templateId, versionData })`                      | "Add New Version" submit                                 |
| `createTemplatePages({ versionId, pagesData })`                           | Immediately after version creation (blank page init)     |
| `updateTemplateVersion({ templateId, versionId, data })`                  | Rename confirm                                           |
| `deleteTemplateVersion({ templateId, versionId })`                        | Delete confirm                                           |
| `cloneTemplateVersion({ ..., copyPages: true, copyExportOptions: true })` | Duplicate confirm                                        |
| `fetchVersionComments({ templateId, versionId })`                         | Comment icon click                                       |
| `deleteTemplate(templateId)`                                              | Delete template confirm                                  |

**Navigation on version click:**

```js
navigate(`/editor/${versionId}?templateId=${templateId}`);
```

> ✅ **`versionsLoadedRef` pattern:** A `useRef` tracks whether versions have been loaded to prevent duplicate API calls on re-renders. After any mutation, the ref is reset and `fetchTemplateVersions` is re-dispatched — a deliberate choice to always refresh from the source of truth rather than trust optimistic Redux updates.

---

### `TemplateEditor.jsx`

**Routes:** `/designer`, `/editor/:templateId`, `/editor/new-version/:baseTemplateId/:baseVersionId`

**Purpose:** Full-screen PDF template editor — orchestrates the three-step creation workflow, loads/saves schemas, manages design vs. preview mode, and bridges Redux to `DesignerCanvas`.

**Three-step creation workflow (sequential, not parallelizable):**

```
Step 1: dispatch(createTemplate(...))          → GET template ID
Step 2: dispatch(createTemplateVersion(...))   → GET version ID  (rollback: deleteTemplate if fails)
Step 3: dispatch(createTemplatePages(...))     → GET page IDs    (rollback: deleteTemplate if fails)
```

**StrictMode guard — prevents double-creation:**

```js
// workflowInitiatedRef prevents StrictMode from double-firing the creation effect
const workflowInitiatedRef = useRef(false);

useEffect(() => {
  if (workflowInitiatedRef.current) return;
  workflowInitiatedRef.current = true;
  createTemplateWorkflow(metadata);
}, []);
```

> 🚨 **Broken save path:** `handleSave` (line 956) has `dispatch(saveVersionSchema(...))` commented out (lines 980–984). Only a comment is saved. **Schema changes are not persisted.** To debug, uncomment line 980 and verify the schema API endpoint.

> ⚠️ **Stale Redux after publish:** There is no Redux state update after publishing. The version's `published` and `status` fields remain stale in Redux until `fetchTemplateVersions` is called again on the next `TemplateDetail` mount.

> ⚠️ **Imperative Redux reads:** All three selectors (`selectVersionById`, `selectActiveVersionByTemplate`, `selectVersionSchema`) are read via `store.getState()` inside `useEffect`, not `useSelector`. The component will **not re-render** if these values change after mount.

---

### `DesignerCanvas.jsx`

**Purpose:** A `forwardRef` wrapper around the third-party `@baylor/pdfme-ui` Designer component. Adds field-API sync, page-API sync, field group management, and an imperative handle for parent control.

**Props:**

| Prop                     | Type     | Description                         |
| ------------------------ | -------- | ----------------------------------- |
| `template`               | object   | pdfme schema object                 |
| `onTemplateChange`       | function | Called on every canvas change       |
| `onSelectedFieldsChange` | function | Fires when field selection changes  |
| `versionId`              | string   | Current version ID                  |
| `pageIds[]`              | array    | Page backend IDs                    |
| `enableFieldSync`        | boolean  | Feature flag for backend field sync |
| `onPageAdded`            | function | Notifies parent of new page         |
| `onPageRemoved`          | function | Notifies parent of deleted page     |
| `onGroupUpdated`         | function | Notifies parent of group change     |

**Imperative handle (exposed to parent via `useImperativeHandle`):**

```js
{
  getTemplate(),              // Pull current schema from pdfme Designer
  updateTemplate(template),   // Push schema into Designer
  updateTemplateInternal(t),  // Internal-only update (no onTemplateChange callback)
  addElement(elementType),    // Programmatically add a field
  registerExternalFields(f),  // Register fields from external sources
  generateHTMLReport(),       // Generate HTML preview
}
```

**Direct API calls (bypassing Redux):**

- `templatePageService.create()` — new pages
- `templatePageService.delete()` — removed pages
- `templateFieldService.*` — field sync when `enableFieldSync=true`
- `templateFieldGroupService.*` — group management

> ✅ The `forwardRef + useImperativeHandle` pattern is the correct choice — the pdfme Designer is an imperative widget, not a React-native component, so a ref bridge is necessary.

> ⚠️ **`groupOperationInProgressRef` (line 99):** A synchronization primitive that prevents `detectAndSyncFieldChanges` from running during group operations (which rename fields, triggering false deletes/creates). If you see spurious backend field deletions during group operations, investigate here first.

> ⚠️ **`latestFieldGroupsRef` (line 94):** pdfme's Designer does not preserve `fieldGroups` (a custom extension) when pages are added or removed. This ref acts as a manual backup to restore them. This is a known incompatibility with the third-party library.

---

## Happy Path Walkthrough

> "A user opens the app, browses the template list, creates a new template using the wizard, opens the editor, designs a page, saves, and publishes."

### Leg 1 — App Boots

`main.jsx` mounts `<App>`. Redux `<Provider>`, `<MantineProvider>`, and `<Router>` initialize. `SchemaInitializer` fires `initializeDefaultSchemas(store.dispatch)` in the background, seeding WGS schema data. The Router resolves `/` and renders `<TemplateList>`.

### Leg 2 — Template List Loads

```
useEffect → dispatch(fetchTemplates({ page: 1, limit: 10 }))
         → GET /v1/templates?page=1&limit=10
         → fetchTemplates.fulfilled
         → state.templates.templates populated, isLoading → false
         → Template cards appear
```

### Leg 3 — User Opens the Wizard

1. User clicks **"New Template"** → `setShowNewTemplateWizard(true)`.
2. `fetch('/codes.json')` populates hospital/test code dropdowns.
3. User fills in name, hospital code, test code.
4. `validateForm()` passes — duplicate name check runs against Redux templates.
5. `navigate('/designer', { state: { templateMetadata, hospitalCode, testCode } })`.

> ⚠️ Toast "Template created successfully!" fires **here** — before the backend has created anything.

### Leg 4 — Template Creation Workflow

Three sequential API calls in `TemplateEditor.createTemplateWorkflow`:

```
POST /v1/templates                      → createdTemplateId
POST /v1/templates/:id/versions         → createdVersionId
POST /v1/template-pages/bulk            → createdPages[0].id
```

If Step 2 or 3 fails → `dispatch(deleteTemplate(createdTemplateId))` rolls back.

### Leg 5 — Canvas Initializes

`isLoading` becomes `false` → `<DesignerCanvas>` renders. pdfme `Designer` mounts into `containerRef.current`, rendering a blank A4 canvas. `getDatasourceOptionsForCanvas` loads asynchronously to populate field dropdowns.

### Leg 6 — User Designs on Canvas

1. User drags a field onto the canvas.
2. pdfme fires `onChangeTemplate(newTemplate)`.
3. `DesignerCanvas` calls `onTemplateChange(newTemplate)` → `TemplateEditor.handleTemplateChange` updates local `template` state.
4. If `enableFieldSync=true`: `detectAndSyncFieldChanges` runs → `templateFieldService.create(...)` persists the field to the backend.

### Leg 7 — User Saves

```
handleSave("Initial design", 'save')
  → designerRef.current.getTemplate()          ✅ Gets schema from canvas
  → dispatch(saveVersionSchema(...))            ❌ COMMENTED OUT — schema NOT saved
  → templateVersionService.addComment(...)      ✅ Comment saved
  → showSuccess("Template saved successfully!") ← misleading — schema was not persisted
```

### Leg 8 — User Publishes

```
handleSave(comment, 'publish')
  → (same as save above)
  → templateVersionService.publish(templateId, versionId)
  → POST /v1/templates/:templateId/versions/:versionId/publish
  → showSuccess("Template published successfully!")
  → Redux state NOT updated (stale DRAFT status until next TemplateDetail mount)
```

---

## Debugging Guide

Use this table to place `console.log` statements when investigating specific problems:

| Problem                                      | File                    | Location                           | What to Log                                                   |
| -------------------------------------------- | ----------------------- | ---------------------------------- | ------------------------------------------------------------- |
| Templates not loading                        | `TemplateList.jsx`      | `useEffect` line 45                | `params` object                                               |
| Wizard navigation state lost                 | `NewTemplateWizard.jsx` | Line 327                           | `metadata` before `navigate()`                                |
| Template/version/page creation failure       | `TemplateEditor.jsx`    | After each `.unwrap()` in workflow | `createdTemplateId`, `createdVersionId`, `createdPages`       |
| Schema not saving                            | `TemplateEditor.jsx`    | Line 980                           | Uncomment `saveVersionSchema` dispatch; log `currentTemplate` |
| Field sync issues (spurious creates/deletes) | `DesignerCanvas.jsx`    | `detectAndSyncFieldChanges`        | Field name + operation type                                   |
| Published version shows wrong status         | `TemplateEditor.jsx`    | After `publishTemplate()` returns  | Full response body                                            |

---

## Architectural Risks & Known Issues

| #   | Risk                                                                                       | Location                         | Severity    |
| --- | ------------------------------------------------------------------------------------------ | -------------------------------- | ----------- |
| 1   | **Schema save is commented out** — canvas changes are never persisted to the backend       | `TemplateEditor.jsx:980`         | 🔴 Critical |
| 2   | **"Success" toast fires before backend template creation** — misleading UX                 | `NewTemplateWizard.jsx:342`      | 🟠 High     |
| 3   | **Published version status not updated in Redux** — stale UI until page re-load            | `TemplateEditor.jsx:1096`        | 🟠 High     |
| 4   | **Auth interceptor disabled** — no authentication headers are sent with any request        | `interceptors/request.js:16`     | 🟠 High     |
| 5   | **`store.getState()` used imperatively** instead of `useSelector` — no reactivity          | `TemplateEditor.jsx:589,631,632` | 🟡 Medium   |
| 6   | **Dead React import in `templateVersionService.js`** (`import { version } from 'react'`)   | `templateVersionService.js:6`    | 🟡 Medium   |
| 7   | **Factory selectors do linear scans without memoization** — performance risk at scale      | `templatesSlice.js:807`          | 🟡 Medium   |
| 8   | **`fieldGroups` lost by pdfme on page operations** — mitigated by `latestFieldGroupsRef`   | `DesignerCanvas.jsx:94`          | 🟡 Medium   |
| 9   | **`updateVersionSchema` and `initializeVersionSchema` have identical logic** — duplication | `templatesSlice.js`              | 🟢 Low      |
| 10  | **`redux-persist` boilerplate with no actual redux-persist integration** — dead config     | `store.js`                       | 🟢 Low      |

---

## Summary

The Report Builder UI is a well-structured React + Redux application with clear architectural boundaries, but carries several important gaps that a new developer must understand before making changes:

**What works well:**

- Clean API layer with Axios helpers and a stable service interface
- Thoughtful Redux Toolkit usage with clear slice structure
- Defensive patterns like `workflowInitiatedRef` (StrictMode guard) and `versionsLoadedRef` (deduplication)
- `forwardRef + useImperativeHandle` correctly used for the imperative pdfme canvas bridge
- Debounced server-side search with smart page reset in `TemplateList`

**What needs immediate attention:**

1. **Uncomment `saveVersionSchema` dispatch** in `TemplateEditor.handleSave` (line 980) — this is the top-priority fix to restore schema persistence.
2. **Move the success toast in `NewTemplateWizard`** to after `createTemplateWorkflow` completes successfully.
3. **Dispatch a Redux update after publish** so the version status reflects `published` without requiring a full page reload.
4. **Enable authentication** in `interceptors/request.js` before any production deployment.
5. **Remove dead code:** `import { version } from 'react'` in `templateVersionService.js`, commented-out wizard workflow (lines 67–135), and unused `fetchTemplateVersions` import in `TemplateList`.
