Project Context
Two repositories work together:

Frontend: reportbuilder-ui-ddp — React + Vite + Redux Toolkit + Mantine UI
Backend: reportbuilder-api-ddp — NestJS + TypeORM + PostgreSQL

1. Frontend Flow
   The SearchInput Component
   File: src/components/common/SearchInput.jsx

export function SearchInput({ value, onChange, placeholder = 'Search', ...props }) {
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
This is a fully controlled, presentational component — it owns zero search logic. It holds only one piece of local state: focused (boolean), used exclusively to drive the data-floating attribute for the floating label CSS animation. The actual search value and onChange handler are injected by the parent via props.

autoComplete="off" prevents the browser from suggesting previous searches, avoiding UX confusion in a data-rich enterprise app.

State Management
File: src/pages/TemplateList.jsx

All search state is local to TemplateList — none of it lives in Redux:

const [searchValue, setSearchValue] = useState('') // raw input value (instant)
const [debouncedSearch] = useDebouncedValue(searchValue, 500) // debounced copy (500ms delay)
const [currentPage, setCurrentPage] = useState(1)
const prevSearchRef = useRef(debouncedSearch) // tracks previous debounced value
useDebouncedValue is from @mantine/hooks. It creates a derived state value that only updates 500ms after the raw value stops changing. This means every React render during typing updates searchValue instantly (keeping the input responsive), but debouncedSearch only updates once the user pauses.

prevSearchRef is a useRef — a mutable container that persists across renders without causing re-renders. It is used to detect whether the debounced search value actually changed between effect runs, which is necessary because the same useEffect also runs when currentPage changes.

Event Handling

const handleSearchChange = (event) => {
setSearchValue(event.currentTarget.value)
}
This is the only event handler. It reads from event.currentTarget.value (not event.target.value — Mantine wraps events, but both work; currentTarget is slightly more reliable with React synthetic events). There is no onSubmit or onKeyDown — this is a live search, not a form submission.

The Core useEffect — Triggering API Calls

useEffect(() => {
const searchChanged = prevSearchRef.current !== debouncedSearch
prevSearchRef.current = debouncedSearch

const page = searchChanged ? 1 : currentPage
if (searchChanged && currentPage !== 1) {
setCurrentPage(1)
}

const params = { page, limit: itemsPerPage }
if (debouncedSearch.trim()) {
params.name = debouncedSearch.trim()
}
dispatch(fetchTemplates(params))
}, [dispatch, currentPage, debouncedSearch])
This effect fires when either currentPage or debouncedSearch changes. The logic:

Compare debouncedSearch against prevSearchRef.current to detect a real search change
If search changed → force page = 1 regardless of current page state, and reset currentPage to 1 (this triggers another effect run, but on that run searchChanged will be false, so it's safe)
Only append name param if the trimmed search is non-empty (prevents sending name= as an empty string to the backend)
Dispatch fetchTemplates to Redux
The .trim() on both the param assignment and the condition means whitespace-only searches (" ") behave as no-search — no name param is sent.

Redux Thunk and API Call
File: src/store/templatesSlice.js

export const fetchTemplates = createAsyncThunk(
'templates/fetchAll',
async (params = {}, { rejectWithValue }) => {
try {
const response = await templateService.getAll(params)
return response
} catch (error) {
return rejectWithValue(error.message)
}
}
)
File: src/api/services/templateService.js

getAll: (params = {}) => {
return get(endpoints.templates.list, { params })
},
File: src/api/client.js

export const get = (url, config = {}) => {
return apiClient.get(url, config)
}
Axios serializes the params object into a query string automatically. So { page: 1, limit: 10, name: 'genetic' } becomes ?page=1&limit=10&name=genetic.

Loading, Error, and Empty States
From the Redux selectors:

const isLoading = useSelector(selectTemplatesLoading)
const error = useSelector(selectTemplatesError)
Loading: isLoading is true during the pending phase of the thunk — the UI renders a spinner/skeleton
Error: rejectWithValue(error.message) populates state.error, the UI renders an error message
Empty: When templates.length === 0 after a successful response:

{debouncedSearch.trim()
? `No Template Found for "${debouncedSearch.trim()}"`
: "No Template Found"}
Note it uses debouncedSearch (not searchValue) for the empty-state message — this is correct, since the displayed results correspond to the debounced value, not the instantaneous typing value.

Optimizations
Optimization Implementation Detail
Debounce useDebouncedValue(searchValue, 500) 500ms delay before API call
Trim .trim() before sending param Prevents spurious calls for whitespace
Page reset guard prevSearchRef comparison Avoids double-fetch on page reset
No client-side filter Server-side only All filtering done in PostgreSQL
autoComplete="off" Component prop Prevents UX interference 2. API Layer
Endpoint

GET /v1/templates
Why GET? Search is a read operation with no side effects. Query parameters in the URL are appropriate for filters/pagination — they are bookmarkable, cacheable by HTTP infrastructure, and semantically correct per REST conventions.

Request Format

GET /v1/templates?page=1&limit=10&name=genetic
All parameters are optional. The backend applies defaults (page=1, limit=10) when absent.

Response Structure

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
Pagination
Offset-based (page + limit). The skip value is computed as (page - 1) \* limit. TypeORM's .skip().take() translates to SQL LIMIT / OFFSET. Constraints: page max 10,000, limit max 100, enforced via class-validator decorators on the DTO.

Authentication/Authorization
The exploration found no auth guards on GET /v1/templates specifically — this is an internal enterprise tool and the auth layer may be at the infrastructure level (reverse proxy, API gateway). No per-request auth headers were observed in the frontend API client for this call.

Rate Limiting
No explicit rate limiting was found in the code. At 500ms debounce, a sustained search session generates at most 2 requests/second — low enough that application-level rate limiting is not critical for single-user usage.

3. Backend Flow
   File: src/modules/templates/templates.controller.ts

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
The @Query() decorator with TemplateQueryDto triggers NestJS's ValidationPipe automatically. Before findAll is called, the framework:

Parses the query string into a plain object
Transforms it to a TemplateQueryDto instance (via @Type(() => Number) for numeric coercion)
Validates all constraints (min/max values, string length, boolean format)
Rejects the request with 400 Bad Request if any constraint fails
@Trace decorators are OpenTelemetry instrumentation — each controller call creates a trace span, enabling distributed tracing across the service.

Validation and Sanitization
File: src/modules/templates/dto/template-query.dto.ts

@IsOptional()
@IsString({ message: ValidationMessages.string('Search name') })
@MaxLength(255, { message: ValidationMessages.maxLength('Search name', 255) })
name?: string;
@IsOptional() — field is not required
@IsString() — rejects arrays, objects, numbers passed as name
@MaxLength(255) — prevents extremely long search strings
NestJS ValidationPipe strips unknown properties by default (whitelist: true)
SQL injection is prevented structurally by TypeORM's parameterized query builder — the name value is never interpolated into SQL strings directly.

Service Layer
File: src/modules/templates/templates.service.ts

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
queryBuilder.skip((page - 1) \* limit).take(limit);

const [templates, total] = await queryBuilder.getManyAndCount();
// ... fetch active versions in a second query ...
return { data, total, page, limit, totalPages };
}
The service uses the repository pattern via TypeORM's QueryBuilder. Filters are conditionally chained with andWhere — unused filters generate no SQL. getManyAndCount() issues a single SQL query that returns both the rows and the total count (using a subquery internally), avoiding a separate COUNT(\*) round-trip.

4. Database / Search Engine
   Database: PostgreSQL (TypeORM entity mapped to report_templates table)

Search Query

SELECT template.\*, ...
FROM report_templates template
WHERE template.name ILIKE '%searchterm%'
ORDER BY template.created_at DESC
LIMIT 10 OFFSET 0;
ILIKE is PostgreSQL's case-insensitive LIKE. The %..% wildcards on both sides implement substring matching — searching "genetic" will match "Genetic Panel", "GENETIC TEST", "Pre-Genetic Screen".

Indexing
No custom index on name was found in the migration files. A standard B-tree index on name would not be used for ILIKE '%..%' queries (leading wildcard disables index). For production performance at scale, a pg_trgm (trigram) index would be appropriate:

CREATE INDEX idx_templates_name_trgm ON report_templates USING gin (name gin_trgm_ops);
This enables index-accelerated ILIKE '%..%' via trigram matching.

Active Versions Optimization
After fetching the paginated template results, the service fetches their active versions in a single batched query instead of N individual queries (the N+1 problem):

const activeVersions = await this.templateRepository.manager
.createQueryBuilder(TemplateVersion, 'version')
.where('version.templateId IN (:...templateIds)', { templateIds })
.andWhere('version.status = :status', { status: 'ACTIVE' })
.getMany();
This generates one IN (...) query regardless of how many templates are on the page.

5. Caching & Performance
   Layer Caching Detail
   Frontend None No memoization, no SWR/React Query cache
   Redux Short-lived in-memory Redux state holds last response; navigating away and back re-fetches
   HTTP None explicit No Cache-Control headers observed; no CDN caching for this API
   Backend None No Redis or in-memory cache
   Database PostgreSQL buffer cache OS-level page cache for hot data
   The absence of caching is acceptable for an internal enterprise tool with low concurrent users. Each search fires a fresh DB query.

Debounce effectiveness: At 500ms, a user typing an 8-character search term typically generates 1-3 API calls instead of 8, reducing backend load by ~75%.

6. Files Involved
   File Role
   src/components/common/SearchInput.jsx Presentational input component; renders Mantine TextInput with search icon and focus state
   src/components/common/SearchInput.module.css CSS module for input styling, focus transitions, border radius
   src/pages/TemplateList.jsx Orchestrator: owns searchValue, debouncedSearch, page state; triggers dispatch
   src/store/templatesSlice.js Redux slice: fetchTemplates async thunk, reducers for loading/fulfilled/rejected, selectors
   src/store/store.js Redux store configuration; registers templatesReducer
   src/api/services/templateService.js API service layer; getAll(params) and search(query) methods wrapping Axios
   src/api/config/apiConfig.js Endpoint registry; /v1/templates as endpoints.templates.list
   src/api/client.js Axios instance factory; get() helper; response interceptor unwrapping .data
   src/modules/templates/templates.controller.ts NestJS controller; @Get() with @Query() TemplateQueryDto; OpenTelemetry tracing
   src/modules/templates/templates.service.ts Business logic; TypeORM QueryBuilder; ILIKE filter; batched version fetch
   src/modules/templates/dto/template-query.dto.ts Validation DTO: page, limit, isActive, name with class-validator decorators
   src/modules/templates/templates.service.spec.ts Unit tests: ILIKE behavior, combined filters, pagination math
   test/e2e/templates.e2e-spec.ts Backend E2E: case-insensitive search, empty results, combined filters
   e2e/tests/ui/pagination-search.spec.ts Playwright E2E: debounce timing, page reset, empty state, network interception
   e2e/tests/ui/template-list.spec.ts Playwright E2E: rapid typing debounce, special characters, whitespace search
   e2e/tests/api/templates.api.spec.ts API-level E2E: search by name (note: uses search param, may mismatch backend)
   e2e/page-objects/template-list.page.ts Playwright page object: search() and clearSearch() helpers
   e2e/helpers/wait.helper.ts waitForSearchDebounce(page, ms=300) — waits for debounce (comment says 300ms, actual is 500ms)
   e2e/helpers/network.helper.ts collectAPICallsByMethod(), assertDebouncedCall(), mockAPIError()
   e2e/helpers/api.helper.ts getTemplates(params) — builds URL and fires direct API request
7. End-to-End Flow Diagram

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
│ - "page" → Number(1), validated: int, min 1, max 10000 ✓
│ - "limit" → Number(10), validated: int, min 1, max 100 ✓
│ - "name" → "genetic", validated: string, maxLength 255 ✓
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
│ SELECT template._
│ FROM report_templates template
│ WHERE template.name ILIKE '%genetic%'
│ ORDER BY template.created_at DESC
│ LIMIT 10 OFFSET 0
│ → [templates[], total]
│
▼
Batched version fetch:
│ SELECT version._
│ FROM template_versions version
│ WHERE version.templateId IN (...ids)
│ AND version.status = 'ACTIVE'
│ AND version.deletedAt IS NULL
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
User sees filtered results 8. Edge Cases
Empty Search
When the user clears the input, searchValue becomes "". After 500ms, debouncedSearch becomes "". The useEffect detects a change, calls debouncedSearch.trim() which is "" — falsy — so name param is not added to the request. The backend receives GET /v1/templates?page=1&limit=10 and returns all templates. ✓

Whitespace-Only Search
A search of " " after .trim() is "" — identical behavior to empty search. The backend never receives the whitespace. Confirmed by Playwright test: 'should handle search with only whitespace'.

Special Characters
The Playwright test 'should handle search with special characters without crash' types <script>alert(1)</script>. This value:

Is sent URL-encoded by Axios: name=%3Cscript%3Ealert%281%29%3C%2Fscript%3E
Is validated as a plain string by @IsString() — length < 255 ✓
Is passed to TypeORM as a parameterized value: { name: '%<script>alert(1)</script>%' }
PostgreSQL receives it as a literal string, never as SQL — no SQL injection possible
The response is JSON — React renders it as text, not HTML — no XSS possible
Characters like !@#$%^&\*() are tested similarly. % and _ inside the search string are PostgreSQL LIKE wildcards — currently NOT escaped, meaning a search for % would match everything and _ would match any single character. This is a known edge case (see improvements below).

Timeout / Network Failure
The fetchTemplates thunk catches all errors and calls rejectWithValue(error.message). The Redux slice's .addCase(fetchTemplates.rejected, ...) sets state.error. The UI checks error from useSelector and renders an error message.

Large Result Sets
Constrained by limit: max 100 and totalPages: ceil(total/limit). The UI paginates via currentPage state. The backend never returns unbounded result sets.

Concurrent Requests (Rapid Typing)
With 500ms debounce, if a user types quickly and then pauses, only one request fires. However, if the user types slowly (one character every >500ms), multiple requests can be in-flight simultaneously. There is no request cancellation (e.g., Axios CancelToken or AbortController). A slow response from request N could arrive after request N+1's response, causing stale results to overwrite fresh ones. This is a race condition — see improvements below.

9. Improvements & Best Practices
   Critical: Fix the Race Condition
   Add request cancellation using AbortController or RTK Query's built-in cancellation:

// In the thunk
async (params, { signal }) => {
const response = await apiClient.get('/v1/templates', { params, signal })
return response
}
Or migrate to RTK Query which handles cancellation, caching, and deduplication automatically.

Fix: Escape LIKE Wildcards in Search
In templates.service.ts, escape % and \_ in the search term before wrapping in %..%:

const escaped = name.replace(/[%_\\]/g, '\\$&')
queryBuilder.andWhere('template.name ILIKE :name ESCAPE \'\\\'', { name: `%${escaped}%` })
Fix: Debounce Comment in Test Helper
e2e/helpers/wait.helper.ts says "300ms in TemplateList.jsx" but the actual debounce is 500ms. Update the comment and default parameter:

export async function waitForSearchDebounce(page: Page, ms = 500) {
await page.waitForTimeout(ms + 100);
}
Fix: API Test Parameter Mismatch
e2e/tests/api/templates.api.spec.ts:88 sends { search: templateName } but the backend expects { name: templateName }. The test should be corrected or a search alias added to the DTO.

Performance: Add Trigram Index for ILIKE

CREATE EXTENSION IF NOT EXISTS pg_trgm;
CREATE INDEX idx_templates_name_trgm ON report_templates USING GIN (name gin_trgm_ops);
This makes ILIKE '%..%' index-accelerated in PostgreSQL, critical once the table exceeds ~10k rows.

Performance: Migrate to RTK Query
Replace manual createAsyncThunk + reducers with RTK Query:

const templatesApi = createApi({
endpoints: (builder) => ({
getTemplates: builder.query({ query: (params) => ({ url: '/v1/templates', params }) })
})
})
Benefits: automatic caching (5-min default), deduplication of identical queries, background refetch, built-in loading/error states, request cancellation.

Scalability: Add Redis Caching on Backend
For read-heavy deployments, cache common searches in Redis with a short TTL (e.g., 30 seconds):

const cacheKey = `templates:${JSON.stringify(query)}`
const cached = await redis.get(cacheKey)
if (cached) return JSON.parse(cached)
const result = await this.buildQuery(query)
await redis.setex(cacheKey, 30, JSON.stringify(result))
return result
Security: Add Auth Guard to List Endpoint
Even for internal tools, GET /v1/templates should require authentication to prevent data enumeration by unauthenticated users. A NestJS JWT guard:

@Get()
@UseGuards(JwtAuthGuard)
async findAll(@Query() query: TemplateQueryDto) { ... }
Code Organization: Extract Search State into Custom Hook
The search + pagination logic in TemplateList.jsx is substantial enough to extract:

function useTemplateSearch() {
const [searchValue, setSearchValue] = useState('')
const [debouncedSearch] = useDebouncedValue(searchValue, 500)
const [currentPage, setCurrentPage] = useState(1)
// ... effect and dispatch logic ...
return { searchValue, setSearchValue, currentPage, setCurrentPage }
}
This makes TemplateList a pure view component and makes the search logic independently testable.
