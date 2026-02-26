# SearchInput — End-to-End Technical Breakdown

## Project Context

Two repositories work together to deliver the SearchInput functionality:

| Repository                        | Stack                                     |
| --------------------------------- | ----------------------------------------- |
| `reportbuilder-ui-ddp` (Frontend) | React + Vite + Redux Toolkit + Mantine UI |
| `reportbuilder-api-ddp` (Backend) | NestJS + TypeORM + PostgreSQL             |

---

## Table of Contents

1. [Frontend Flow](#1-frontend-flow)
2. [API Layer](#2-api-layer)
3. [Backend Flow](#3-backend-flow)
4. [Database & Search Engine](#4-database--search-engine)
5. [Caching & Performance](#5-caching--performance)
6. [Files Involved](#6-files-involved)
7. [End-to-End Flow Diagram](#7-end-to-end-flow-diagram)
8. [Edge Cases](#8-edge-cases)
9. [Improvements & Best Practices](#9-improvements--best-practices)

---

## 1. Frontend Flow

### The `SearchInput` Component

**File:** `src/components/common/SearchInput.jsx`

```jsx
export function SearchInput({
  value,
  onChange,
  placeholder = "Search",
  ...props
}) {
  const [focused, setFocused] = useState(false);
  const floating = value?.trim().length !== 0 || focused || undefined;

  return (
    <TextInput
      placeholder={placeholder}
      classNames={classes}
      value={value}
      onChange={onChange}
      onFocus={() => setFocused(true)}
      onBlur={() => setFocused(false)}
      autoComplete="off"
      data-floating={floating}
      leftSection={<IconSearch size={18} stroke={1.5} />}
      {...props}
    />
  );
}
```

This is a **fully controlled, presentational component** — it owns zero search logic. Its only piece of local state is `focused` (boolean), used exclusively to drive the `data-floating` attribute for the floating label CSS animation. The actual `value` and `onChange` handler are injected by the parent via props.

> `autoComplete="off"` prevents the browser from suggesting previous searches, avoiding UX confusion in a data-rich enterprise application.

---

### State Management

**File:** `src/pages/TemplateList.jsx`

All search state is **local to `TemplateList`** — none of it lives in Redux:

```jsx
const [searchValue, setSearchValue] = useState(""); // raw input value (instant)
const [debouncedSearch] = useDebouncedValue(searchValue, 500); // debounced copy (500ms delay)
const [currentPage, setCurrentPage] = useState(1);
const prevSearchRef = useRef(debouncedSearch); // tracks previous debounced value
```

- **`useDebouncedValue`** (from `@mantine/hooks`) creates a derived state value that only updates 500ms after the raw value stops changing. Every render during typing updates `searchValue` instantly (keeping the input responsive), while `debouncedSearch` updates only once the user pauses.
- **`prevSearchRef`** is a `useRef` — a mutable container that persists across renders without triggering re-renders. It detects whether the debounced search value changed between effect runs, which is necessary because the same `useEffect` also runs when `currentPage` changes.

---

### Event Handling

```jsx
const handleSearchChange = (event) => {
  setSearchValue(event.currentTarget.value);
};
```

This is the **only event handler**. It reads from `event.currentTarget.value` (`currentTarget` is slightly more reliable than `target` with React synthetic events). There is no `onSubmit` or `onKeyDown` — this is a **live search**, not a form submission.

---

### The Core `useEffect` — Triggering API Calls

```jsx
useEffect(() => {
  const searchChanged = prevSearchRef.current !== debouncedSearch;
  prevSearchRef.current = debouncedSearch;

  const page = searchChanged ? 1 : currentPage;
  if (searchChanged && currentPage !== 1) {
    setCurrentPage(1);
  }

  const params = { page, limit: itemsPerPage };
  if (debouncedSearch.trim()) {
    params.name = debouncedSearch.trim();
  }
  dispatch(fetchTemplates(params));
}, [dispatch, currentPage, debouncedSearch]);
```

This effect fires when either `currentPage` or `debouncedSearch` changes. The logic is:

1. Compare `debouncedSearch` against `prevSearchRef.current` to detect a real search change.
2. If the search changed → force `page = 1` regardless of current page state, and reset `currentPage` to 1. (This triggers another effect run, but on that run `searchChanged` will be `false`, so it's safe.)
3. Only append the `name` param if the trimmed search is non-empty — this prevents sending `name=` as an empty string to the backend.
4. Dispatch `fetchTemplates` to Redux.

> The `.trim()` on both the param assignment and the condition means whitespace-only searches (e.g., `"   "`) behave identically to an empty search — no `name` param is sent.

---

### Redux Thunk and API Call

**`src/store/templatesSlice.js`**

```js
export const fetchTemplates = createAsyncThunk(
  "templates/fetchAll",
  async (params = {}, { rejectWithValue }) => {
    try {
      const response = await templateService.getAll(params);
      return response;
    } catch (error) {
      return rejectWithValue(error.message);
    }
  },
);
```

**`src/api/services/templateService.js`**

```js
getAll: (params = {}) => {
  return get(endpoints.templates.list, { params })
},
```

**`src/api/client.js`**

```js
export const get = (url, config = {}) => {
  return apiClient.get(url, config);
};
```

Axios automatically serializes the `params` object into a query string. For example, `{ page: 1, limit: 10, name: 'genetic' }` becomes `?page=1&limit=10&name=genetic`.

---

### Loading, Error, and Empty States

```jsx
const isLoading = useSelector(selectTemplatesLoading);
const error = useSelector(selectTemplatesError);
```

| State       | Trigger                                                  | UI Behavior                 |
| ----------- | -------------------------------------------------------- | --------------------------- |
| **Loading** | `isLoading = true` during thunk `pending` phase          | Spinner / skeleton          |
| **Error**   | `rejectWithValue(error.message)` populates `state.error` | Error message               |
| **Empty**   | `templates.length === 0` after successful response       | "No Template Found" message |

The empty-state message uses `debouncedSearch` (not `searchValue`) — this is intentional, since displayed results correspond to the debounced value, not the instantaneous typing value:

```jsx
{
  debouncedSearch.trim()
    ? `No Template Found for "${debouncedSearch.trim()}"`
    : "No Template Found";
}
```

---

### Frontend Optimizations

| Optimization          | Implementation                        | Detail                                 |
| --------------------- | ------------------------------------- | -------------------------------------- |
| Debounce              | `useDebouncedValue(searchValue, 500)` | 500ms delay before API call            |
| Trim                  | `.trim()` before sending param        | Prevents spurious calls for whitespace |
| Page reset guard      | `prevSearchRef` comparison            | Avoids double-fetch on page reset      |
| No client-side filter | Server-side only                      | All filtering done in PostgreSQL       |
| `autoComplete="off"`  | Component prop                        | Prevents UX interference               |

---

## 2. API Layer

### Endpoint

```
GET /v1/templates
```

**Why GET?** Search is a read operation with no side effects. Query parameters in the URL are appropriate for filters/pagination — they are bookmarkable, cacheable by HTTP infrastructure, and semantically correct per REST conventions.

### Request Format

```
GET /v1/templates?page=1&limit=10&name=genetic
```

All parameters are optional. The backend applies defaults (`page=1`, `limit=10`) when absent.

### Response Structure

```json
{
  "data": [
    {
      "id": "uuid",
      "name": "Genetic Panel",
      "description": "...",
      "isActive": true,
      "createdBy": "user@example.com",
      "metadata": {},
      "createdAt": "2024-01-01T00:00:00Z",
      "updatedAt": "2024-01-01T00:00:00Z",
      "activeVersion": {
        "id": "uuid",
        "versionLabel": "v1.0",
        "status": "ACTIVE",
        "publishedAt": "2024-01-01T00:00:00Z"
      }
    }
  ],
  "total": 47,
  "page": 1,
  "limit": 10,
  "totalPages": 5
}
```

### Pagination

Offset-based (`page` + `limit`). The skip value is computed as `(page - 1) * limit`. TypeORM's `.skip().take()` translates to SQL `LIMIT` / `OFFSET`. Constraints enforced via `class-validator` decorators on the DTO:

- `page`: max 10,000
- `limit`: max 100

### Authentication / Authorization

No explicit auth guards were found on `GET /v1/templates`. For this internal enterprise tool, the auth layer may sit at the infrastructure level (reverse proxy or API gateway). No per-request auth headers were observed in the frontend API client for this call.

### Rate Limiting

No application-level rate limiting was found in the code. At 500ms debounce, a sustained search session generates at most 2 requests/second — low enough that rate limiting is not critical for single-user usage.

---

## 3. Backend Flow

**File:** `src/modules/templates/templates.controller.ts`

```ts
@Get()
@Version('1')
@Trace('GET /v1/templates')
async findAll(@Query() query: TemplateQueryDto): Promise<PaginatedTemplatesDto> {
  addSpanAttributes({ 'http.method': 'GET', 'http.route': '/v1/templates' });
  addSpanEvent('Received findAll request');
  const result = await this.templatesService.findAll(query);
  addSpanAttributes({ 'http.status_code': 200 });
  return result;
}
```

The `@Query()` decorator with `TemplateQueryDto` triggers NestJS's `ValidationPipe` automatically. Before `findAll` is called, the framework:

1. Parses the query string into a plain object.
2. Transforms it to a `TemplateQueryDto` instance (via `@Type(() => Number)` for numeric coercion).
3. Validates all constraints (min/max values, string length, boolean format).
4. Rejects the request with `400 Bad Request` if any constraint fails.

> `@Trace` decorators are OpenTelemetry instrumentation — each controller call creates a trace span, enabling distributed tracing across the service.

---

### Validation and Sanitization

**File:** `src/modules/templates/dto/template-query.dto.ts`

```ts
@IsOptional()
@IsString({ message: ValidationMessages.string('Search name') })
@MaxLength(255, { message: ValidationMessages.maxLength('Search name', 255) })
name?: string;
```

| Decorator         | Purpose                                               |
| ----------------- | ----------------------------------------------------- |
| `@IsOptional()`   | Field is not required                                 |
| `@IsString()`     | Rejects arrays, objects, and numbers passed as `name` |
| `@MaxLength(255)` | Prevents extremely long search strings                |

NestJS `ValidationPipe` strips unknown properties by default (`whitelist: true`). SQL injection is prevented structurally by TypeORM's **parameterized query builder** — the `name` value is never interpolated directly into SQL strings.

---

### Service Layer

**File:** `src/modules/templates/templates.service.ts`

```ts
async findAll(query: TemplateQueryDto): Promise<PaginatedTemplatesDto> {
  const { isActive, name, page = 1, limit = 10 } = query;
  const queryBuilder = this.templateRepository.createQueryBuilder('template');

  if (isActive !== undefined) {
    queryBuilder.andWhere('template.isActive = :isActive', { isActive });
  }
  if (name !== undefined) {
    queryBuilder.andWhere('template.name ILIKE :name', { name: `%${name}%` });
  }

  queryBuilder.orderBy('template.createdAt', 'DESC');
  queryBuilder.skip((page - 1) * limit).take(limit);

  const [templates, total] = await queryBuilder.getManyAndCount();
  // ... fetch active versions in a second query ...
  return { data, total, page, limit, totalPages };
}
```

- Filters are **conditionally chained** with `andWhere` — unused filters generate no SQL.
- `getManyAndCount()` issues a single SQL query returning both rows and the total count (using an internal subquery), avoiding a separate `COUNT(*)` round-trip.

---

## 4. Database & Search Engine

**Database:** PostgreSQL (TypeORM entity mapped to `report_templates` table)

### Search Query

```sql
SELECT template.*, ...
FROM report_templates template
WHERE template.name ILIKE '%searchterm%'
ORDER BY template.created_at DESC
LIMIT 10 OFFSET 0;
```

`ILIKE` is PostgreSQL's case-insensitive `LIKE`. The `%..%` wildcards on both sides implement **substring matching** — searching `"genetic"` matches `"Genetic Panel"`, `"GENETIC TEST"`, and `"Pre-Genetic Screen"`.

---

### Indexing

No custom index on `name` was found in the migration files. A standard B-tree index on `name` would **not** be used for `ILIKE '%..%'` queries (the leading wildcard disables index usage). For production performance at scale, a `pg_trgm` (trigram) index is appropriate:

```sql
CREATE INDEX idx_templates_name_trgm ON report_templates USING gin (name gin_trgm_ops);
```

This enables index-accelerated `ILIKE '%..%'` via trigram matching.

---

### Active Versions — Avoiding the N+1 Problem

After fetching paginated template results, the service fetches their active versions in a **single batched query** instead of N individual queries:

```ts
const activeVersions = await this.templateRepository.manager
  .createQueryBuilder(TemplateVersion, "version")
  .where("version.templateId IN (:...templateIds)", { templateIds })
  .andWhere("version.status = :status", { status: "ACTIVE" })
  .getMany();
```

This generates one `IN (...)` query regardless of how many templates are on the page.

---

## 5. Caching & Performance

| Layer    | Caching                 | Detail                                                               |
| -------- | ----------------------- | -------------------------------------------------------------------- |
| Frontend | None                    | No memoization, no SWR/React Query cache                             |
| Redux    | Short-lived in-memory   | Redux state holds last response; navigating away and back re-fetches |
| HTTP     | None explicit           | No `Cache-Control` headers; no CDN caching for this API              |
| Backend  | None                    | No Redis or in-memory cache                                          |
| Database | PostgreSQL buffer cache | OS-level page cache for hot data                                     |

The absence of caching is acceptable for an internal enterprise tool with low concurrent users. Each search fires a fresh database query.

> **Debounce effectiveness:** At 500ms, a user typing an 8-character search term typically generates 1–3 API calls instead of 8, reducing backend load by ~75%.

---

## 6. Files Involved

### Frontend (`reportbuilder-ui-ddp`)

| File                                           | Role                                                                                          |
| ---------------------------------------------- | --------------------------------------------------------------------------------------------- |
| `src/components/common/SearchInput.jsx`        | Presentational input component; renders Mantine `TextInput` with search icon and focus state  |
| `src/components/common/SearchInput.module.css` | CSS module for input styling, focus transitions, border radius                                |
| `src/pages/TemplateList.jsx`                   | Orchestrator: owns `searchValue`, `debouncedSearch`, page state; triggers dispatch            |
| `src/store/templatesSlice.js`                  | Redux slice: `fetchTemplates` async thunk, reducers for loading/fulfilled/rejected, selectors |
| `src/store/store.js`                           | Redux store configuration; registers `templatesReducer`                                       |
| `src/api/services/templateService.js`          | API service layer; `getAll(params)` method wrapping Axios                                     |
| `src/api/config/apiConfig.js`                  | Endpoint registry; `/v1/templates` as `endpoints.templates.list`                              |
| `src/api/client.js`                            | Axios instance factory; `get()` helper; response interceptor unwrapping `.data`               |

### Backend (`reportbuilder-api-ddp`)

| File                                              | Role                                                                                  |
| ------------------------------------------------- | ------------------------------------------------------------------------------------- |
| `src/modules/templates/templates.controller.ts`   | NestJS controller; `@Get()` with `@Query()` `TemplateQueryDto`; OpenTelemetry tracing |
| `src/modules/templates/templates.service.ts`      | Business logic; TypeORM QueryBuilder; `ILIKE` filter; batched version fetch           |
| `src/modules/templates/dto/template-query.dto.ts` | Validation DTO: `page`, `limit`, `isActive`, `name` with `class-validator` decorators |

### Tests

| File                                              | Role                                                                                               |
| ------------------------------------------------- | -------------------------------------------------------------------------------------------------- |
| `src/modules/templates/templates.service.spec.ts` | Unit tests: `ILIKE` behavior, combined filters, pagination math                                    |
| `test/e2e/templates.e2e-spec.ts`                  | Backend E2E: case-insensitive search, empty results, combined filters                              |
| `e2e/tests/ui/pagination-search.spec.ts`          | Playwright E2E: debounce timing, page reset, empty state, network interception                     |
| `e2e/tests/ui/template-list.spec.ts`              | Playwright E2E: rapid typing debounce, special characters, whitespace search                       |
| `e2e/tests/api/templates.api.spec.ts`             | API-level E2E: search by name _(note: uses `search` param — may mismatch backend)_                 |
| `e2e/page-objects/template-list.page.ts`          | Playwright page object: `search()` and `clearSearch()` helpers                                     |
| `e2e/helpers/wait.helper.ts`                      | `waitForSearchDebounce(page, ms=300)` — waits for debounce _(comment says 300ms; actual is 500ms)_ |
| `e2e/helpers/network.helper.ts`                   | `collectAPICallsByMethod()`, `assertDebouncedCall()`, `mockAPIError()`                             |
| `e2e/helpers/api.helper.ts`                       | `getTemplates(params)` — builds URL and fires direct API request                                   |

---

## 7. End-to-End Flow Diagram

```
User types "genetic" into <SearchInput />
    │
    ▼
handleSearchChange(event) → setSearchValue("genetic")
    │ (React re-render: input shows "g", "ge", "gen"... instantly)
    │
    ▼
useDebouncedValue fires after 500ms of no typing
    │ debouncedSearch = "genetic"
    │
    ▼
useEffect([dispatch, currentPage, debouncedSearch]) fires
    │ prevSearchRef.current ("") !== debouncedSearch ("genetic") → searchChanged = true
    │ prevSearchRef.current = "genetic"
    │ page = 1 (forced), setCurrentPage(1) called
    │ params = { page: 1, limit: 10, name: "genetic" }
    │
    ▼
dispatch(fetchTemplates({ page: 1, limit: 10, name: "genetic" }))
    │ Redux: state.isLoading = true → UI shows spinner
    │
    ▼
templateService.getAll({ page: 1, limit: 10, name: "genetic" })
    │
    ▼
axios.get("/v1/templates", { params: { page: 1, limit: 10, name: "genetic" } })
    │
    ▼
HTTP GET /v1/templates?page=1&limit=10&name=genetic
    │ (Transit over network to NestJS API)
    │
    ▼
NestJS ValidationPipe transforms query string:
    │ - "page"  → Number(1),     validated: int, min 1, max 10000 ✓
    │ - "limit" → Number(10),    validated: int, min 1, max 100   ✓
    │ - "name"  → "genetic",     validated: string, maxLength 255 ✓
    │ → TemplateQueryDto instance
    │
    ▼
TemplatesController.findAll(query) → @Trace span opened
    │
    ▼
TemplatesService.findAll({ page: 1, limit: 10, name: "genetic" })
    │
    ▼
TypeORM QueryBuilder:
    │ SELECT template.*
    │ FROM report_templates template
    │ WHERE template.name ILIKE '%genetic%'
    │ ORDER BY template.created_at DESC
    │ LIMIT 10 OFFSET 0
    │ → [templates[], total]
    │
    ▼
Batched version fetch:
    │ SELECT version.*
    │ FROM template_versions version
    │ WHERE version.templateId IN (...ids)
    │   AND version.status = 'ACTIVE'
    │   AND version.deletedAt IS NULL
    │
    ▼
Build PaginatedTemplatesDto:
    │ { data: [...], total: 3, page: 1, limit: 10, totalPages: 1 }
    │
    ▼
HTTP 200 JSON response
    │
    ▼
Axios responseInterceptor unwraps response.data
    │
    ▼
fetchTemplates.fulfilled reducer:
    │ state.isLoading = false
    │ state.templates = [mapped template objects]
    │ state.pagination = { total: 3, page: 1, limit: 10, totalPages: 1 }
    │
    ▼
React re-render:
    │ templates.length > 0 → render template cards
    │ OR templates.length === 0 → render `No Template Found for "genetic"`
    │
    ▼
User sees filtered results
```

---

## 8. Edge Cases

### Empty Search

When the user clears the input, `searchValue` becomes `""`. After 500ms, `debouncedSearch` becomes `""`. The `useEffect` detects a change; `debouncedSearch.trim()` is `""` (falsy), so the `name` param is not added. The backend receives `GET /v1/templates?page=1&limit=10` and returns all templates. ✅

### Whitespace-Only Search

A search of `"   "` after `.trim()` is `""` — identical behavior to an empty search. The backend never receives the whitespace string. Covered by Playwright test: `'should handle search with only whitespace'`. ✅

### Special Characters

The Playwright test `'should handle search with special characters without crash'` types `<script>alert(1)</script>`. This value:

- Is URL-encoded by Axios: `name=%3Cscript%3Ealert%281%29%3C%2Fscript%3E`
- Is validated as a plain string by `@IsString()` (length < 255) ✅
- Is passed to TypeORM as a parameterized value: `{ name: '%<script>alert(1)</script>%' }`
- Is received by PostgreSQL as a literal string — **no SQL injection possible**
- Is rendered by React as text, not HTML — **no XSS possible**

> **Known edge case:** `%` and `_` inside the search string are PostgreSQL `LIKE` wildcards and are currently **not escaped**. A search for `%` would match everything; `_` would match any single character. See [Improvements](#9-improvements--best-practices) for the fix.

### Timeout / Network Failure

The `fetchTemplates` thunk catches all errors via `rejectWithValue(error.message)`. The Redux slice's `.addCase(fetchTemplates.rejected, ...)` sets `state.error`, and the UI renders an error message.

### Large Result Sets

Constrained by `limit: max 100` and `totalPages: ceil(total / limit)`. The UI paginates via `currentPage` state. The backend never returns unbounded result sets.

### Concurrent Requests (Rapid Typing)

With 500ms debounce, fast typing typically results in a single request. However, if a user types slowly (one character every >500ms), multiple requests may be in-flight simultaneously. There is currently **no request cancellation** (e.g., Axios `CancelToken` or `AbortController`). A slow response from request N could arrive after request N+1's response, causing stale data to overwrite fresh results. This is a **known race condition** — see [Improvements](#9-improvements--best-practices).

---

## 9. Improvements & Best Practices

### 🔴 Critical: Fix the Race Condition

Add request cancellation using `AbortController` or RTK Query's built-in cancellation:

```ts
// In the async thunk
async (params, { signal }) => {
  const response = await apiClient.get("/v1/templates", { params, signal });
  return response;
};
```

Alternatively, migrate to **RTK Query**, which handles cancellation, caching, and deduplication automatically.

---

### 🔴 Fix: Escape LIKE Wildcards in Search

In `templates.service.ts`, escape `%` and `_` in the search term before wrapping in `%..%`:

```ts
const escaped = name.replace(/[%_\\]/g, "\\$&");
queryBuilder.andWhere("template.name ILIKE :name ESCAPE '\\'", {
  name: `%${escaped}%`,
});
```

---

### 🟡 Fix: Debounce Comment in Test Helper

`e2e/helpers/wait.helper.ts` references 300ms in a comment, but the actual debounce is 500ms. Update the comment and default parameter:

```ts
// waitForSearchDebounce: waits for the 500ms debounce in TemplateList.jsx
export async function waitForSearchDebounce(page: Page, ms = 500) {
  await page.waitForTimeout(ms + 100);
}
```

---

### 🟡 Fix: API Test Parameter Mismatch

`e2e/tests/api/templates.api.spec.ts:88` sends `{ search: templateName }`, but the backend expects `{ name: templateName }`. The test should be corrected, or a `search` alias should be added to the DTO.

---

### 🟢 Performance: Add Trigram Index for ILIKE

```sql
CREATE EXTENSION IF NOT EXISTS pg_trgm;
CREATE INDEX idx_templates_name_trgm ON report_templates USING GIN (name gin_trgm_ops);
```

This makes `ILIKE '%..%'` index-accelerated in PostgreSQL — critical once the table exceeds ~10,000 rows.

---

### 🟢 Performance: Migrate to RTK Query

Replace manual `createAsyncThunk` + reducers with RTK Query:

```ts
const templatesApi = createApi({
  endpoints: (builder) => ({
    getTemplates: builder.query({
      query: (params) => ({ url: "/v1/templates", params }),
    }),
  }),
});
```

**Benefits:** automatic caching (5-min default), deduplication of identical queries, background refetch, built-in loading/error states, and request cancellation.

---

### 🟢 Scalability: Add Redis Caching on Backend

For read-heavy deployments, cache common searches in Redis with a short TTL:

```ts
const cacheKey = `templates:${JSON.stringify(query)}`;
const cached = await redis.get(cacheKey);
if (cached) return JSON.parse(cached);

const result = await this.buildQuery(query);
await redis.setex(cacheKey, 30, JSON.stringify(result));
return result;
```

---

### 🔵 Security: Add Auth Guard to List Endpoint

Even for internal tools, `GET /v1/templates` should require authentication to prevent data enumeration by unauthenticated users:

```ts
@Get()
@UseGuards(JwtAuthGuard)
async findAll(@Query() query: TemplateQueryDto) { ... }
```

---

### 🔵 Code Organization: Extract Search State into a Custom Hook

The search + pagination logic in `TemplateList.jsx` is substantial enough to extract into its own hook:

```ts
function useTemplateSearch() {
  const [searchValue, setSearchValue] = useState("");
  const [debouncedSearch] = useDebouncedValue(searchValue, 500);
  const [currentPage, setCurrentPage] = useState(1);
  // ... effect and dispatch logic ...
  return { searchValue, setSearchValue, currentPage, setCurrentPage };
}
```

This makes `TemplateList` a pure view component and makes the search logic independently testable.
