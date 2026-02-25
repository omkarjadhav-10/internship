  ---
  Report Builder UI (DDP) — Complete Technical Roadmap

  ---
  Section 1: Project Architecture Overview

  1.1 Architecture Type

  This project follows a Feature-Layered Hybrid Architecture — a combination of:

  - Feature-based grouping at the component level (components/designer/, components/template/, components/modals/)
  - Layered architecture at the infrastructure level (api/, store/, hooks/, utils/)
  - Page-Container-Presentational pattern for routing and UI separation

  1.2 Application Purpose

  A PDF Template Designer for Baylor Genetics. Users create, version, edit, preview, and publish medical report templates. Templates are built visually using a drag-and-drop PDF design
  engine (pdfme) and stored via a REST API backend.

  1.3 High-Level Data Flow

  Browser
    └── React 18 SPA (Vite)
          ├── React Router 7 (Client-side Routing)
          ├── Redux Toolkit (Global State)
          │     └── Async Thunks → API Services → Axios → REST API
          ├── Mantine UI (Component Library)
          ├── Baylor pdfme (PDF Designer/Viewer/Generator)
          └── Playwright (E2E Testing, separate layer)

  1.4 Folder Responsibilities

  ┌──────────────────────────┬───────────────────────────────────────────────────────────────┐
  │          Folder          │                             Role                              │
  ├──────────────────────────┼───────────────────────────────────────────────────────────────┤
  │ src/pages/               │ Route-level containers (data fetching, orchestration)         │
  ├──────────────────────────┼───────────────────────────────────────────────────────────────┤
  │ src/components/designer/ │ PDF canvas, palettes, field groups, preview                   │
  ├──────────────────────────┼───────────────────────────────────────────────────────────────┤
  │ src/components/template/ │ Template creation, management wizards                         │
  ├──────────────────────────┼───────────────────────────────────────────────────────────────┤
  │ src/components/modals/   │ All modal dialogs (CRUD workflows)                            │
  ├──────────────────────────┼───────────────────────────────────────────────────────────────┤
  │ src/components/layout/   │ Header, sidebar chrome                                        │
  ├──────────────────────────┼───────────────────────────────────────────────────────────────┤
  │ src/store/               │ Redux slice + store config                                    │
  ├──────────────────────────┼───────────────────────────────────────────────────────────────┤
  │ src/api/                 │ Axios client, services, interceptors, error handling          │
  ├──────────────────────────┼───────────────────────────────────────────────────────────────┤
  │ src/hooks/               │ Reusable React hooks                                          │
  ├──────────────────────────┼───────────────────────────────────────────────────────────────┤
  │ src/utils/               │ Pure utility functions (transforms, validators, font loading) │
  ├──────────────────────────┼───────────────────────────────────────────────────────────────┤
  │ src/constants/           │ Application-wide constants                                    │
  ├──────────────────────────┼───────────────────────────────────────────────────────────────┤
  │ e2e/                     │ Playwright E2E tests (TypeScript, Page Object Model)          │
  └──────────────────────────┴───────────────────────────────────────────────────────────────┘

  1.5 Coding Patterns Identified

  ┌────────────────────────────────────┬──────────────────────────────────────────┐
  │              Pattern               │                Where Used                │
  ├────────────────────────────────────┼──────────────────────────────────────────┤
  │ Redux Async Thunks                 │ All API calls via templatesSlice.js      │
  ├────────────────────────────────────┼──────────────────────────────────────────┤
  │ Service Layer abstraction          │ api/services/*.js — decoupled API calls  │
  ├────────────────────────────────────┼──────────────────────────────────────────┤
  │ Axios Interceptor Middleware       │ api/interceptors/request.js, response.js │
  ├────────────────────────────────────┼──────────────────────────────────────────┤
  │ Custom Error Classes               │ APIError in errorHandler.js              │
  ├────────────────────────────────────┼──────────────────────────────────────────┤
  │ Custom React Hooks                 │ useFormValidation, useNotification       │
  ├────────────────────────────────────┼──────────────────────────────────────────┤
  │ forwardRef + useImperativeHandle   │ DesignerCanvas.jsx                       │
  ├────────────────────────────────────┼──────────────────────────────────────────┤
  │ Page Object Model                  │ All e2e/page-objects/*.ts                │
  ├────────────────────────────────────┼──────────────────────────────────────────┤
  │ Debouncing                         │ Field API sync in DesignerCanvas         │
  ├────────────────────────────────────┼──────────────────────────────────────────┤
  │ Ref-based stale closure prevention │ latestFieldGroupsRef, groupCallbacksRef  │
  ├────────────────────────────────────┼──────────────────────────────────────────┤
  │ Hash-based change detection        │ Template schema change tracking          │
  ├────────────────────────────────────┼──────────────────────────────────────────┤
  │ Feature flags via props            │ enableFieldSync prop                     │
  ├────────────────────────────────────┼──────────────────────────────────────────┤
  │ Controlled modal components        │ All modals/*.jsx                         │
  └────────────────────────────────────┴──────────────────────────────────────────┘

  1.6 Technical Debt & Complexity Areas

  ┌────────────────────────────────┬────────────────────────────────────────────────────────┬──────────┐
  │              Area              │                         Issue                          │ Severity │
  ├────────────────────────────────┼────────────────────────────────────────────────────────┼──────────┤
  │ DesignerCanvas.jsx (68.6 KB)   │ God component — too large, many responsibilities       │ High     │
  ├────────────────────────────────┼────────────────────────────────────────────────────────┼──────────┤
  │ fieldGroupManager.js (580 KB!) │ Enormous utility file, needs decomposition             │ High     │
  ├────────────────────────────────┼────────────────────────────────────────────────────────┼──────────┤
  │ fieldTypeMapping.js            │ Hardcoded UUIDs — TODO to load from API                │ Medium   │
  ├────────────────────────────────┼────────────────────────────────────────────────────────┼──────────┤
  │ No TypeScript in app code      │ All .jsx — no prop types, no TS interfaces             │ Medium   │
  ├────────────────────────────────┼────────────────────────────────────────────────────────┼──────────┤
  │ No unit tests                  │ Only E2E tests — no component/unit tests               │ Medium   │
  ├────────────────────────────────┼────────────────────────────────────────────────────────┼──────────┤
  │ Auth commented out             │ Token injection in interceptors is incomplete          │ Medium   │
  ├────────────────────────────────┼────────────────────────────────────────────────────────┼──────────┤
  │ templatesSlice.js (28 KB)      │ Single slice managing too many concerns                │ Medium   │
  ├────────────────────────────────┼────────────────────────────────────────────────────────┼──────────┤
  │ dynamicDatasourceMapping.js    │ Complex field-to-datasource wiring with no tests       │ Medium   │
  ├────────────────────────────────┼────────────────────────────────────────────────────────┼──────────┤
  │ Stale closure workarounds      │ Multiple useRef hacks to prevent pdfme stale callbacks │ Low-Med  │
  └────────────────────────────────┴────────────────────────────────────────────────────────┴──────────┘

  1.7 Project Prerequisites Summary

  Before contributing, a developer needs solid understanding of:
  1. React 18 (hooks, refs, composition)
  2. Redux Toolkit (slices, async thunks, selectors)
  3. Axios + interceptors
  4. React Router v6/v7
  5. Mantine component library
  6. Vite build tooling
  7. TypeScript basics (for E2E tests)
  8. Playwright (for tests)
  9. pdfme PDF design concepts

  ---
  Section 2: JavaScript Concepts Required

  2.1 Core JavaScript

  ┌───────────────────────────────────┬───────────────────────────────────────────────────────────────────────────────────┬────────────────┐
  │              Concept              │                                    Where Used                                     │   Difficulty   │
  ├───────────────────────────────────┼───────────────────────────────────────────────────────────────────────────────────┼────────────────┤
  │ ES Modules (import/export)        │ Every file                                                                        │ Beginner       │
  ├───────────────────────────────────┼───────────────────────────────────────────────────────────────────────────────────┼────────────────┤
  │ Destructuring (object + array)    │ const { values, errors } = useFormValidation(...) — everywhere                    │ Beginner       │
  ├───────────────────────────────────┼───────────────────────────────────────────────────────────────────────────────────┼────────────────┤
  │ Spread operator (...)             │ Redux state updates { ...state, templates: [...] }                                │ Beginner       │
  ├───────────────────────────────────┼───────────────────────────────────────────────────────────────────────────────────┼────────────────┤
  │ Template literals                 │ URL building \/v1/templates/${id}/versions``                                      │ Beginner       │
  ├───────────────────────────────────┼───────────────────────────────────────────────────────────────────────────────────┼────────────────┤
  │ Arrow functions                   │ Everywhere — event handlers, thunks, .map() callbacks                             │ Beginner       │
  ├───────────────────────────────────┼───────────────────────────────────────────────────────────────────────────────────┼────────────────┤
  │ Optional chaining (?.)            │ response?.data?.templates, error?.message                                         │ Beginner       │
  ├───────────────────────────────────┼───────────────────────────────────────────────────────────────────────────────────┼────────────────┤
  │ Nullish coalescing (??)           │ timeout ?? 30000, default props                                                   │ Beginner       │
  ├───────────────────────────────────┼───────────────────────────────────────────────────────────────────────────────────┼────────────────┤
  │ Short-circuit evaluation (&&, ||) │ isLoading && <Spinner />, conditional rendering                                   │ Beginner       │
  ├───────────────────────────────────┼───────────────────────────────────────────────────────────────────────────────────┼────────────────┤
  │ Array methods                     │ .map(), .filter(), .find(), .reduce(), .some(), .every(), .flatMap() — throughout │ Beginner-Inter │
  ├───────────────────────────────────┼───────────────────────────────────────────────────────────────────────────────────┼────────────────┤
  │ Object methods                    │ Object.keys(), Object.entries(), Object.fromEntries() — in transforms             │ Intermediate   │
  ├───────────────────────────────────┼───────────────────────────────────────────────────────────────────────────────────┼────────────────┤
  │ Ternary operators                 │ isLoading ? <Loader /> : <Content />                                              │ Beginner       │
  ├───────────────────────────────────┼───────────────────────────────────────────────────────────────────────────────────┼────────────────┤
  │ Default parameter values          │ function get(url, config = {})                                                    │ Beginner       │
  ├───────────────────────────────────┼───────────────────────────────────────────────────────────────────────────────────┼────────────────┤
  │ Rest parameters                   │ function createRule(validator, ...params)                                         │ Intermediate   │
  ├───────────────────────────────────┼───────────────────────────────────────────────────────────────────────────────────┼────────────────┤
  │ Computed property names           │ { [fieldId]: error } in form validation                                           │ Intermediate   │
  ├───────────────────────────────────┼───────────────────────────────────────────────────────────────────────────────────┼────────────────┤
  │ typeof / instanceof               │ error instanceof APIError, typeof config === 'object'                             │ Beginner       │
  └───────────────────────────────────┴───────────────────────────────────────────────────────────────────────────────────┴────────────────┘

  2.2 ES6+ Async Patterns

  ┌──────────────────────────────┬────────────────────────────────────────────────────────────────┬──────────────┐
  │           Concept            │                           Where Used                           │  Difficulty  │
  ├──────────────────────────────┼────────────────────────────────────────────────────────────────┼──────────────┤
  │ Promises                     │ Axios .then()/.catch(), baseline understanding                 │ Intermediate │
  ├──────────────────────────────┼────────────────────────────────────────────────────────────────┼──────────────┤
  │ async/await                  │ All async thunks, API service methods, test helpers            │ Intermediate │
  ├──────────────────────────────┼────────────────────────────────────────────────────────────────┼──────────────┤
  │ try/catch/finally            │ All API calls, error boundary logic                            │ Intermediate │
  ├──────────────────────────────┼────────────────────────────────────────────────────────────────┼──────────────┤
  │ Promise.all()                │ Batch page/field creation — createTemplatePages thunk          │ Intermediate │
  ├──────────────────────────────┼────────────────────────────────────────────────────────────────┼──────────────┤
  │ Promise.allSettled()         │ Error-tolerant batch operations                                │ Intermediate │
  ├──────────────────────────────┼────────────────────────────────────────────────────────────────┼──────────────┤
  │ Microtask vs Macrotask queue │ Needed to understand debounce and setTimeout behavior          │ Advanced     │
  ├──────────────────────────────┼────────────────────────────────────────────────────────────────┼──────────────┤
  │ Error propagation            │ throw error in service layer → caught by thunk rejectWithValue │ Intermediate │
  ├──────────────────────────────┼────────────────────────────────────────────────────────────────┼──────────────┤
  │ setTimeout/clearTimeout      │ Debounced field sync in DesignerCanvas                         │ Intermediate │
  └──────────────────────────────┴────────────────────────────────────────────────────────────────┴──────────────┘

  2.3 Functional Programming Concepts

  ┌────────────────────────┬──────────────────────────────────────────────────────────────┬──────────────┐
  │        Concept         │                          Where Used                          │  Difficulty  │
  ├────────────────────────┼──────────────────────────────────────────────────────────────┼──────────────┤
  │ Pure functions         │ All validators in validators.js return null or error string  │ Beginner     │
  ├────────────────────────┼──────────────────────────────────────────────────────────────┼──────────────┤
  │ Higher-order functions │ createRule(validator, ...params) — curried validator factory │ Intermediate │
  ├────────────────────────┼──────────────────────────────────────────────────────────────┼──────────────┤
  │ Function composition   │ Combining validators, chaining transformers                  │ Intermediate │
  ├────────────────────────┼──────────────────────────────────────────────────────────────┼──────────────┤
  │ Immutability           │ Redux state — never mutate, always return new objects        │ Intermediate │
  ├────────────────────────┼──────────────────────────────────────────────────────────────┼──────────────┤
  │ Currying               │ createRule(required) → returns validator function            │ Advanced     │
  ├────────────────────────┼──────────────────────────────────────────────────────────────┼──────────────┤
  │ Closures               │ Custom hooks capture state, interceptors capture config      │ Intermediate │
  └────────────────────────┴──────────────────────────────────────────────────────────────┴──────────────┘

  2.4 Advanced JavaScript

  ┌────────────────────────────────┬────────────────────────────────────────────────────────────────────┬──────────────┐
  │            Concept             │                             Where Used                             │  Difficulty  │
  ├────────────────────────────────┼────────────────────────────────────────────────────────────────────┼──────────────┤
  │ Class syntax                   │ class APIError extends Error in errorHandler.js                    │ Intermediate │
  ├────────────────────────────────┼────────────────────────────────────────────────────────────────────┼──────────────┤
  │ Custom Error classes           │ APIError with type, status, data, timestamp                        │ Intermediate │
  ├────────────────────────────────┼────────────────────────────────────────────────────────────────────┼──────────────┤
  │ Symbol                         │ Not directly used, but Redux Toolkit uses internally               │ Advanced     │
  ├────────────────────────────────┼────────────────────────────────────────────────────────────────────┼──────────────┤
  │ WeakRef / FinalizationRegistry │ Not used, but pdfme internals may use                              │ Advanced     │
  ├────────────────────────────────┼────────────────────────────────────────────────────────────────────┼──────────────┤
  │ Proxy / Reflect                │ Used by Redux Toolkit internally (Immer.js)                        │ Advanced     │
  ├────────────────────────────────┼────────────────────────────────────────────────────────────────────┼──────────────┤
  │ JSON serialization             │ JSON.parse(JSON.stringify(...)) for deep cloning                   │ Beginner     │
  ├────────────────────────────────┼────────────────────────────────────────────────────────────────────┼──────────────┤
  │ crypto.randomUUID()            │ In generateVersionId() utility                                     │ Intermediate │
  ├────────────────────────────────┼────────────────────────────────────────────────────────────────────┼──────────────┤
  │ Regex                          │ Version format validation /v\d{1,2}\.\d{1,2}/, field type matching │ Intermediate │
  ├────────────────────────────────┼────────────────────────────────────────────────────────────────────┼──────────────┤
  │ FormData API                   │ File upload in templateService.uploadFile()                        │ Intermediate │
  ├────────────────────────────────┼────────────────────────────────────────────────────────────────────┼──────────────┤
  │ Fetch / XHR                    │ Indirect — via Axios under the hood                                │ Intermediate │
  ├────────────────────────────────┼────────────────────────────────────────────────────────────────────┼──────────────┤
  │ import.meta.env                │ Vite environment variables throughout API config                   │ Intermediate │
  ├────────────────────────────────┼────────────────────────────────────────────────────────────────────┼──────────────┤
  │ Dynamic imports                │ Font loading, lazy loading patterns                                │ Intermediate │
  └────────────────────────────────┴────────────────────────────────────────────────────────────────────┴──────────────┘

  2.5 Module System

  ┌───────────────────────────┬──────────────────────────────────────────────────┬──────────────┐
  │          Concept          │                    Where Used                    │  Difficulty  │
  ├───────────────────────────┼──────────────────────────────────────────────────┼──────────────┤
  │ Named exports             │ All services, utilities, hooks                   │ Beginner     │
  ├───────────────────────────┼──────────────────────────────────────────────────┼──────────────┤
  │ Default exports           │ All components, pages                            │ Beginner     │
  ├───────────────────────────┼──────────────────────────────────────────────────┼──────────────┤
  │ Re-exports (barrel files) │ api/index.js, api/services/index.js              │ Intermediate │
  ├───────────────────────────┼──────────────────────────────────────────────────┼──────────────┤
  │ Tree shaking              │ Vite handles unused code elimination             │ Intermediate │
  ├───────────────────────────┼──────────────────────────────────────────────────┼──────────────┤
  │ Circular import awareness │ Service → client → interceptors dependency chain │ Advanced     │
  └───────────────────────────┴──────────────────────────────────────────────────┴──────────────┘

  ---
  Section 3: React Concepts Required

  3.1 Core React

  ┌──────────────────────────┬─────────────────────────────────────────────────┬──────────────┐
  │         Concept          │                   Where Used                    │  Difficulty  │
  ├──────────────────────────┼─────────────────────────────────────────────────┼──────────────┤
  │ JSX syntax               │ Every component file                            │ Beginner     │
  ├──────────────────────────┼─────────────────────────────────────────────────┼──────────────┤
  │ Functional components    │ All components (no class components)            │ Beginner     │
  ├──────────────────────────┼─────────────────────────────────────────────────┼──────────────┤
  │ Props                    │ All components receive typed props              │ Beginner     │
  ├──────────────────────────┼─────────────────────────────────────────────────┼──────────────┤
  │ Children prop            │ Layout.jsx wraps page content                   │ Beginner     │
  ├──────────────────────────┼─────────────────────────────────────────────────┼──────────────┤
  │ Event handling           │ onClick, onChange, onBlur, onKeyDown everywhere │ Beginner     │
  ├──────────────────────────┼─────────────────────────────────────────────────┼──────────────┤
  │ Conditional rendering    │ {isLoading && <Spinner />}, {error ? ... : ...} │ Beginner     │
  ├──────────────────────────┼─────────────────────────────────────────────────┼──────────────┤
  │ List rendering with keys │ Template cards, version lists, modal lists      │ Beginner     │
  ├──────────────────────────┼─────────────────────────────────────────────────┼──────────────┤
  │ Controlled inputs        │ All form fields (value + onChange)              │ Beginner     │
  ├──────────────────────────┼─────────────────────────────────────────────────┼──────────────┤
  │ Lifting state up         │ Modal state lifted to page-level components     │ Intermediate │
  ├──────────────────────────┼─────────────────────────────────────────────────┼──────────────┤
  │ Component composition    │ TemplateEditor composes 6+ child components     │ Intermediate │
  └──────────────────────────┴─────────────────────────────────────────────────┴──────────────┘

  3.2 React Hooks

  ┌─────────────────────┬──────────────────────────────────────────────────────────────┬──────────────┐
  │        Hook         │                          Where Used                          │  Difficulty  │
  ├─────────────────────┼──────────────────────────────────────────────────────────────┼──────────────┤
  │ useState            │ Local state in every component                               │ Beginner     │
  ├─────────────────────┼──────────────────────────────────────────────────────────────┼──────────────┤
  │ useEffect           │ Data fetching on mount, pdfme initialization, schema loading │ Intermediate │
  ├─────────────────────┼──────────────────────────────────────────────────────────────┼──────────────┤
  │ useRef              │ Designer refs, stale closure prevention, DOM access          │ Intermediate │
  ├─────────────────────┼──────────────────────────────────────────────────────────────┼──────────────┤
  │ useCallback         │ Memoizing callbacks in DesignerCanvas, TemplateEditor        │ Intermediate │
  ├─────────────────────┼──────────────────────────────────────────────────────────────┼──────────────┤
  │ useMemo             │ Derived data (field lists, filtered templates)               │ Intermediate │
  ├─────────────────────┼──────────────────────────────────────────────────────────────┼──────────────┤
  │ useContext          │ Indirect — Mantine uses context internally                   │ Intermediate │
  ├─────────────────────┼──────────────────────────────────────────────────────────────┼──────────────┤
  │ useReducer          │ Indirect — Redux uses it internally                          │ Intermediate │
  ├─────────────────────┼──────────────────────────────────────────────────────────────┼──────────────┤
  │ useImperativeHandle │ DesignerCanvas exposes methods via ref                       │ Advanced     │
  ├─────────────────────┼──────────────────────────────────────────────────────────────┼──────────────┤
  │ Custom hooks        │ useFormValidation, useNotification                           │ Intermediate │
  ├─────────────────────┼──────────────────────────────────────────────────────────────┼──────────────┤
  │ Hook rules          │ No hooks in conditions/loops (enforced by ESLint plugin)     │ Intermediate │
  └─────────────────────┴──────────────────────────────────────────────────────────────┴──────────────┘

  3.3 Advanced React Patterns

  ┌────────────────────────────┬──────────────────────────────────────────────────────────┬──────────────┐
  │          Pattern           │                        Where Used                        │  Difficulty  │
  ├────────────────────────────┼──────────────────────────────────────────────────────────┼──────────────┤
  │ forwardRef                 │ DesignerCanvas forwarding ref to parent                  │ Advanced     │
  ├────────────────────────────┼──────────────────────────────────────────────────────────┼──────────────┤
  │ useImperativeHandle        │ Exposing getTemplate(), updateFieldGroups() imperatively │ Advanced     │
  ├────────────────────────────┼──────────────────────────────────────────────────────────┼──────────────┤
  │ memo()                     │ PreviewCanvas wrapped in React.memo for perf             │ Intermediate │
  ├────────────────────────────┼──────────────────────────────────────────────────────────┼──────────────┤
  │ Render props               │ Indirect — Mantine modal uses render-based callbacks     │ Advanced     │
  ├────────────────────────────┼──────────────────────────────────────────────────────────┼──────────────┤
  │ Controlled vs Uncontrolled │ All modals are fully controlled (isOpen, onClose)        │ Intermediate │
  ├────────────────────────────┼──────────────────────────────────────────────────────────┼──────────────┤
  │ Lifting state up           │ Modal open/close state managed by parent page            │ Intermediate │
  ├────────────────────────────┼──────────────────────────────────────────────────────────┼──────────────┤
  │ Callback ref pattern       │ designerRef.current.getTemplate() imperative control     │ Advanced     │
  ├────────────────────────────┼──────────────────────────────────────────────────────────┼──────────────┤
  │ Stale closure prevention   │ useRef for callbacks inside pdfme event listeners        │ Advanced     │
  ├────────────────────────────┼──────────────────────────────────────────────────────────┼──────────────┤
  │ Derived state              │ Selectors compute derived data from Redux store          │ Intermediate │
  ├────────────────────────────┼──────────────────────────────────────────────────────────┼──────────────┤
  │ Optimistic UI              │ Loading states on all async operations                   │ Intermediate │
  └────────────────────────────┴──────────────────────────────────────────────────────────┴──────────────┘

  3.4 React Router v7 (Specific)

  ┌───────────────────┬───────────────────────────────────────────────────────────────┬──────────────┐
  │      Concept      │                          Where Used                           │  Difficulty  │
  ├───────────────────┼───────────────────────────────────────────────────────────────┼──────────────┤
  │ BrowserRouter     │ App.jsx wraps all routes                                      │ Beginner     │
  ├───────────────────┼───────────────────────────────────────────────────────────────┼──────────────┤
  │ Routes + Route    │ 6 route definitions in App.jsx                                │ Beginner     │
  ├───────────────────┼───────────────────────────────────────────────────────────────┼──────────────┤
  │ Link component    │ Navigation between template list and detail                   │ Beginner     │
  ├───────────────────┼───────────────────────────────────────────────────────────────┼──────────────┤
  │ useNavigate()     │ Programmatic navigation (save → redirect)                     │ Intermediate │
  ├───────────────────┼───────────────────────────────────────────────────────────────┼──────────────┤
  │ useParams()       │ Extracting :templateId, :versionId from URL                   │ Intermediate │
  ├───────────────────┼───────────────────────────────────────────────────────────────┼──────────────┤
  │ useLocation()     │ Reading navigation state (unsaved changes detection)          │ Intermediate │
  ├───────────────────┼───────────────────────────────────────────────────────────────┼──────────────┤
  │ Nested routes     │ Template detail → editor sub-navigation                       │ Intermediate │
  ├───────────────────┼───────────────────────────────────────────────────────────────┼──────────────┤
  │ Navigation guards │ useBlocker / browser history interception for unsaved changes │ Advanced     │
  ├───────────────────┼───────────────────────────────────────────────────────────────┼──────────────┤
  │ Route parameters  │ Multiple params: /editor/:templateId                          │ Intermediate │
  └───────────────────┴───────────────────────────────────────────────────────────────┴──────────────┘

  3.5 React + Redux Integration

  ┌───────────────────────┬──────────────────────────────────────────────┬──────────────┐
  │        Concept        │                  Where Used                  │  Difficulty  │
  ├───────────────────────┼──────────────────────────────────────────────┼──────────────┤
  │ Provider component    │ Wraps app in App.jsx                         │ Beginner     │
  ├───────────────────────┼──────────────────────────────────────────────┼──────────────┤
  │ useSelector()         │ Read state in all page components            │ Intermediate │
  ├───────────────────────┼──────────────────────────────────────────────┼──────────────┤
  │ useDispatch()         │ Dispatch actions in event handlers           │ Intermediate │
  ├───────────────────────┼──────────────────────────────────────────────┼──────────────┤
  │ Selectors             │ selectAllTemplates, selectTemplateById(id)   │ Intermediate │
  ├───────────────────────┼──────────────────────────────────────────────┼──────────────┤
  │ Selector memoization  │ createSelector from Redux Toolkit (inferred) │ Intermediate │
  ├───────────────────────┼──────────────────────────────────────────────┼──────────────┤
  │ Async thunk lifecycle │ pending, fulfilled, rejected states          │ Intermediate │
  ├───────────────────────┼──────────────────────────────────────────────┼──────────────┤
  │ rejectWithValue()     │ Error propagation in thunks                  │ Intermediate │
  ├───────────────────────┼──────────────────────────────────────────────┼──────────────┤
  │ extraReducers         │ Handling thunk actions in slice              │ Intermediate │
  └───────────────────────┴──────────────────────────────────────────────┴──────────────┘

  3.6 React Performance

  ┌───────────────────────┬──────────────────────────────────────────────────────────┬──────────────┐
  │       Technique       │                        Where Used                        │  Difficulty  │
  ├───────────────────────┼──────────────────────────────────────────────────────────┼──────────────┤
  │ React.memo()          │ PreviewCanvas prevents re-render on parent state changes │ Intermediate │
  ├───────────────────────┼──────────────────────────────────────────────────────────┼──────────────┤
  │ useCallback           │ Stable callback references for child props               │ Intermediate │
  ├───────────────────────┼──────────────────────────────────────────────────────────┼──────────────┤
  │ useMemo               │ Computed values that are expensive to recalculate        │ Intermediate │
  ├───────────────────────┼──────────────────────────────────────────────────────────┼──────────────┤
  │ Lazy loading          │ Font files loaded on demand                              │ Intermediate │
  ├───────────────────────┼──────────────────────────────────────────────────────────┼──────────────┤
  │ Debouncing            │ Field API sync throttled (DesignerCanvas)                │ Intermediate │
  ├───────────────────────┼──────────────────────────────────────────────────────────┼──────────────┤
  │ Batched state updates │ React 18 automatic batching                              │ Intermediate │
  ├───────────────────────┼──────────────────────────────────────────────────────────┼──────────────┤
  │ StrictMode            │ Double-renders in development for detecting side effects │ Intermediate │
  ├───────────────────────┼──────────────────────────────────────────────────────────┼──────────────┤
  │ Key prop optimization │ List rendering with stable keys                          │ Beginner     │
  ├───────────────────────┼──────────────────────────────────────────────────────────┼──────────────┤
  │ useRef vs useState    │ Refs for values that shouldn't trigger re-renders        │ Intermediate │
  └───────────────────────┴──────────────────────────────────────────────────────────┴──────────────┘

  3.7 React 18 Specifics

  ┌──────────────────────────┬────────────────────────────────────────────────────────────────────┬──────────────┐
  │         Feature          │                             Where Used                             │  Difficulty  │
  ├──────────────────────────┼────────────────────────────────────────────────────────────────────┼──────────────┤
  │ createRoot()             │ main.jsx entry point (new React 18 API)                            │ Beginner     │
  ├──────────────────────────┼────────────────────────────────────────────────────────────────────┼──────────────┤
  │ Automatic batching       │ All state updates auto-batched in event handlers                   │ Intermediate │
  ├──────────────────────────┼────────────────────────────────────────────────────────────────────┼──────────────┤
  │ Concurrent features      │ Available but not explicitly used (used via Mantine)               │ Advanced     │
  ├──────────────────────────┼────────────────────────────────────────────────────────────────────┼──────────────┤
  │ StrictMode double-invoke │ Can affect pdfme Designer initialization — important to understand │ Advanced     │
  └──────────────────────────┴────────────────────────────────────────────────────────────────────┴──────────────┘

  ---
  Section 4: Libraries & Ecosystem Knowledge

  4.1 Redux Toolkit

  ┌────────────────────┬───────────────────────────────────────────────────────────────┬──────────────┐
  │      Concept       │                          Where Used                           │  Difficulty  │
  ├────────────────────┼───────────────────────────────────────────────────────────────┼──────────────┤
  │ configureStore()   │ store.js                                                      │ Intermediate │
  ├────────────────────┼───────────────────────────────────────────────────────────────┼──────────────┤
  │ createSlice()      │ templatesSlice.js — defines reducers + actions                │ Intermediate │
  ├────────────────────┼───────────────────────────────────────────────────────────────┼──────────────┤
  │ createAsyncThunk() │ 15+ async operations in the slice                             │ Intermediate │
  ├────────────────────┼───────────────────────────────────────────────────────────────┼──────────────┤
  │ builder.addCase()  │ Handle thunk lifecycle in extraReducers                       │ Intermediate │
  ├────────────────────┼───────────────────────────────────────────────────────────────┼──────────────┤
  │ Immer integration  │ Direct state mutations in reducers (RTK handles immutability) │ Intermediate │
  ├────────────────────┼───────────────────────────────────────────────────────────────┼──────────────┤
  │ Action creators    │ Auto-generated from slice                                     │ Intermediate │
  ├────────────────────┼───────────────────────────────────────────────────────────────┼──────────────┤
  │ rejectWithValue()  │ Error handling in thunks                                      │ Intermediate │
  ├────────────────────┼───────────────────────────────────────────────────────────────┼──────────────┤
  │ RTK Query          │ Not used — manual thunks instead                              │ N/A          │
  ├────────────────────┼───────────────────────────────────────────────────────────────┼──────────────┤
  │ Redux DevTools     │ Works automatically with RTK                                  │ Beginner     │
  └────────────────────┴───────────────────────────────────────────────────────────────┴──────────────┘

  4.2 Axios

  ┌───────────────────────┬──────────────────────────────────────────────┬──────────────┐
  │        Concept        │                  Where Used                  │  Difficulty  │
  ├───────────────────────┼──────────────────────────────────────────────┼──────────────┤
  │ Instance creation     │ axios.create({baseURL, timeout, headers})    │ Intermediate │
  ├───────────────────────┼──────────────────────────────────────────────┼──────────────┤
  │ Request interceptors  │ Add metadata, logging, auth token            │ Intermediate │
  ├───────────────────────┼──────────────────────────────────────────────┼──────────────┤
  │ Response interceptors │ Transform data, handle errors                │ Intermediate │
  ├───────────────────────┼──────────────────────────────────────────────┼──────────────┤
  │ HTTP methods          │ get, post, put, patch, delete wrappers       │ Beginner     │
  ├───────────────────────┼──────────────────────────────────────────────┼──────────────┤
  │ FormData uploads      │ File upload with multipart/form-data         │ Intermediate │
  ├───────────────────────┼──────────────────────────────────────────────┼──────────────┤
  │ Error handling        │ error.response, error.request, error.message │ Intermediate │
  ├───────────────────────┼──────────────────────────────────────────────┼──────────────┤
  │ Timeout handling      │ 30s global timeout                           │ Intermediate │
  ├───────────────────────┼──────────────────────────────────────────────┼──────────────┤
  │ CancelToken           │ Not used — potential improvement             │ Advanced     │
  ├───────────────────────┼──────────────────────────────────────────────┼──────────────┤
  │ params vs data        │ GET params vs POST body                      │ Beginner     │
  └───────────────────────┴──────────────────────────────────────────────┴──────────────┘

  4.3 Mantine UI (v7)

  ┌──────────────────────┬────────────────────────────────────────────────┬──────────────┐
  │       Concept        │                   Where Used                   │  Difficulty  │
  ├──────────────────────┼────────────────────────────────────────────────┼──────────────┤
  │ MantineProvider      │ App-level theme provider                       │ Beginner     │
  ├──────────────────────┼────────────────────────────────────────────────┼──────────────┤
  │ Button, Input, Modal │ Used throughout all components                 │ Beginner     │
  ├──────────────────────┼────────────────────────────────────────────────┼──────────────┤
  │ Notifications system │ Toast notifications via useNotification hook   │ Intermediate │
  ├──────────────────────┼────────────────────────────────────────────────┼──────────────┤
  │ useDebouncedValue    │ Search input debouncing in TemplateList        │ Intermediate │
  ├──────────────────────┼────────────────────────────────────────────────┼──────────────┤
  │ Mantine hooks        │ @mantine/hooks package                         │ Intermediate │
  ├──────────────────────┼────────────────────────────────────────────────┼──────────────┤
  │ Mantine theming      │ Default theme (not heavily customized)         │ Intermediate │
  ├──────────────────────┼────────────────────────────────────────────────┼──────────────┤
  │ Accessibility        │ Built-in ARIA attributes on Mantine components │ Beginner     │
  └──────────────────────┴────────────────────────────────────────────────┴──────────────┘

  4.4 React Router v7

  See Section 3.4 above. Key additions:

  ┌──────────────────────────────────────┬──────────────┐
  │               Concept                │  Difficulty  │
  ├──────────────────────────────────────┼──────────────┤
  │ v7 breaking changes from v6          │ Intermediate │
  ├──────────────────────────────────────┼──────────────┤
  │ createBrowserRouter vs BrowserRouter │ Intermediate │
  ├──────────────────────────────────────┼──────────────┤
  │ Future flags                         │ Advanced     │
  └──────────────────────────────────────┴──────────────┘

  4.5 Baylor pdfme (PDF Engine)

  This is the most unique and complex library in the stack — a customized fork of the open-source pdfme library.

  ┌─────────────────────────────┬───────────────────────────────────────────────────┬──────────────┐
  │           Concept           │                    Where Used                     │  Difficulty  │
  ├─────────────────────────────┼───────────────────────────────────────────────────┼──────────────┤
  │ Designer component          │ Interactive drag-drop PDF builder                 │ Advanced     │
  ├─────────────────────────────┼───────────────────────────────────────────────────┼──────────────┤
  │ Viewer component            │ PDF preview with data binding                     │ Advanced     │
  ├─────────────────────────────┼───────────────────────────────────────────────────┼──────────────┤
  │ generate() function         │ Generate actual PDF binary                        │ Advanced     │
  ├─────────────────────────────┼───────────────────────────────────────────────────┼──────────────┤
  │ pdfme Schema format         │ JSON structure for PDF templates                  │ Advanced     │
  ├─────────────────────────────┼───────────────────────────────────────────────────┼──────────────┤
  │ Plugins (schemas)           │ text, image, table, svg, barcodes, checkbox, etc. │ Advanced     │
  ├─────────────────────────────┼───────────────────────────────────────────────────┼──────────────┤
  │ getDefaultFont()            │ Built-in font management                          │ Intermediate │
  ├─────────────────────────────┼───────────────────────────────────────────────────┼──────────────┤
  │ getInputFromTemplate()      │ Extract sample input for preview                  │ Intermediate │
  ├─────────────────────────────┼───────────────────────────────────────────────────┼──────────────┤
  │ BLANK_PDF                   │ Base64 blank PDF canvas                           │ Intermediate │
  ├─────────────────────────────┼───────────────────────────────────────────────────┼──────────────┤
  │ Designer onChangeTemplate   │ React to schema changes                           │ Advanced     │
  ├─────────────────────────────┼───────────────────────────────────────────────────┼──────────────┤
  │ Designer onSaveTemplate     │ Save trigger callback                             │ Advanced     │
  ├─────────────────────────────┼───────────────────────────────────────────────────┼──────────────┤
  │ Imperative Designer control │ via designerRef.current                           │ Advanced     │
  ├─────────────────────────────┼───────────────────────────────────────────────────┼──────────────┤
  │ Field plugins               │ Custom schema types for medical fields            │ Advanced     │
  └─────────────────────────────┴───────────────────────────────────────────────────┴──────────────┘

  4.6 Tailwind CSS

  ┌─────────────────────┬──────────────────────────────────────────────────────────┬──────────────┐
  │       Concept       │                        Where Used                        │  Difficulty  │
  ├─────────────────────┼──────────────────────────────────────────────────────────┼──────────────┤
  │ Utility classes     │ Inline styling in JSX (className="flex items-center...") │ Beginner     │
  ├─────────────────────┼──────────────────────────────────────────────────────────┼──────────────┤
  │ Responsive prefixes │ md:, lg: breakpoints                                     │ Intermediate │
  ├─────────────────────┼──────────────────────────────────────────────────────────┼──────────────┤
  │ @apply directive    │ Custom CSS classes in .css files                         │ Intermediate │
  ├─────────────────────┼──────────────────────────────────────────────────────────┼──────────────┤
  │ tailwind.config.js  │ Custom theme extension                                   │ Intermediate │
  ├─────────────────────┼──────────────────────────────────────────────────────────┼──────────────┤
  │ PurgeCSS            │ Tree-shaking unused Tailwind classes                     │ Intermediate │
  ├─────────────────────┼──────────────────────────────────────────────────────────┼──────────────┤
  │ PostCSS integration │ Tailwind as PostCSS plugin                               │ Intermediate │
  ├─────────────────────┼──────────────────────────────────────────────────────────┼──────────────┤
  │ Dark mode           │ Not configured                                           │ N/A          │
  └─────────────────────┴──────────────────────────────────────────────────────────┴──────────────┘

  4.7 Playwright (E2E Testing)

  ┌──────────────────────────┬──────────────────────────────────────────────┬──────────────┐
  │         Concept          │                  Where Used                  │  Difficulty  │
  ├──────────────────────────┼──────────────────────────────────────────────┼──────────────┤
  │ test() and expect()      │ Every test file                              │ Beginner     │
  ├──────────────────────────┼──────────────────────────────────────────────┼──────────────┤
  │ page.goto()              │ Navigate to URLs                             │ Beginner     │
  ├──────────────────────────┼──────────────────────────────────────────────┼──────────────┤
  │ page.click(), fill()     │ UI interactions                              │ Beginner     │
  ├──────────────────────────┼──────────────────────────────────────────────┼──────────────┤
  │ page.locator()           │ Element selection                            │ Intermediate │
  ├──────────────────────────┼──────────────────────────────────────────────┼──────────────┤
  │ Fixtures (test.extend()) │ base.fixture.ts — shared setup/teardown      │ Advanced     │
  ├──────────────────────────┼──────────────────────────────────────────────┼──────────────┤
  │ Page Object Model        │ page-objects/*.ts — abstracted selectors     │ Intermediate │
  ├──────────────────────────┼──────────────────────────────────────────────┼──────────────┤
  │ beforeEach/afterEach     │ Test lifecycle in fixtures                   │ Intermediate │
  ├──────────────────────────┼──────────────────────────────────────────────┼──────────────┤
  │ Network mocking          │ page.route() in network.helper.ts            │ Advanced     │
  ├──────────────────────────┼──────────────────────────────────────────────┼──────────────┤
  │ API testing              │ Direct HTTP calls inside Playwright tests    │ Advanced     │
  ├──────────────────────────┼──────────────────────────────────────────────┼──────────────┤
  │ Test tags                │ @ui, @api, @template-list for selective runs │ Intermediate │
  ├──────────────────────────┼──────────────────────────────────────────────┼──────────────┤
  │ WebServer config         │ Auto-start dev server before tests           │ Intermediate │
  ├──────────────────────────┼──────────────────────────────────────────────┼──────────────┤
  │ Global setup/teardown    │ Database reset via global-setup.ts           │ Advanced     │
  ├──────────────────────────┼──────────────────────────────────────────────┼──────────────┤
  │ Screenshot on failure    │ retainOnFirstFailure setting                 │ Intermediate │
  ├──────────────────────────┼──────────────────────────────────────────────┼──────────────┤
  │ TypeScript in tests      │ All test files are .ts or .ts                │ Intermediate │
  └──────────────────────────┴──────────────────────────────────────────────┴──────────────┘

  4.8 Lucide React & Tabler Icons

  ┌────────────────────────┬───────────────────────────────────────────────────┬────────────┐
  │        Concept         │                    Where Used                     │ Difficulty │
  ├────────────────────────┼───────────────────────────────────────────────────┼────────────┤
  │ Tree-shakeable imports │ import { Plus, ChevronRight } from 'lucide-react' │ Beginner   │
  ├────────────────────────┼───────────────────────────────────────────────────┼────────────┤
  │ Size and color props   │ <Plus size={16} color="white" />                  │ Beginner   │
  ├────────────────────────┼───────────────────────────────────────────────────┼────────────┤
  │ Icon as component      │ Used inline in JSX                                │ Beginner   │
  └────────────────────────┴───────────────────────────────────────────────────┴────────────┘

  ---
  Section 5: Tooling & DevOps Knowledge

  5.1 Vite

  ┌───────────────────────┬─────────────────────────────────────────────────────────┬──────────────┐
  │        Concept        │                       Where Used                        │  Difficulty  │
  ├───────────────────────┼─────────────────────────────────────────────────────────┼──────────────┤
  │ Vite dev server       │ HMR, instant startup, overlay errors                    │ Beginner     │
  ├───────────────────────┼─────────────────────────────────────────────────────────┼──────────────┤
  │ vite.config.js        │ Plugin config, resolve aliases, build options           │ Intermediate │
  ├───────────────────────┼─────────────────────────────────────────────────────────┼──────────────┤
  │ @vitejs/plugin-react  │ Fast Refresh (HMR for React components)                 │ Intermediate │
  ├───────────────────────┼─────────────────────────────────────────────────────────┼──────────────┤
  │ import.meta.env       │ Environment variables (VITE_API_BASE_URL, etc.)         │ Intermediate │
  ├───────────────────────┼─────────────────────────────────────────────────────────┼──────────────┤
  │ resolve.alias         │ Polyfill Node.js modules for browser (crypto, path, fs) │ Advanced     │
  ├───────────────────────┼─────────────────────────────────────────────────────────┼──────────────┤
  │ optimizeDeps          │ Pre-bundling exclusions for pdfme-ui                    │ Advanced     │
  ├───────────────────────┼─────────────────────────────────────────────────────────┼──────────────┤
  │ build.commonjsOptions │ Handle mixed ESM/CJS modules (pdfme)                    │ Advanced     │
  ├───────────────────────┼─────────────────────────────────────────────────────────┼──────────────┤
  │ Rollup internals      │ Vite uses Rollup for production build                   │ Advanced     │
  ├───────────────────────┼─────────────────────────────────────────────────────────┼──────────────┤
  │ ESM-only output       │ Modern browser targeting                                │ Intermediate │
  ├───────────────────────┼─────────────────────────────────────────────────────────┼──────────────┤
  │ Environment modes     │ development, production, test                           │ Intermediate │
  └───────────────────────┴─────────────────────────────────────────────────────────┴──────────────┘

  5.2 ESLint

  ┌──────────────────────┬───────────────────────────────────────────────────┬──────────────┐
  │       Concept        │                    Where Used                     │  Difficulty  │
  ├──────────────────────┼───────────────────────────────────────────────────┼──────────────┤
  │ Flat config format   │ eslint.config.js (new format, not .eslintrc)      │ Intermediate │
  ├──────────────────────┼───────────────────────────────────────────────────┼──────────────┤
  │ react-hooks plugin   │ Enforces hooks rules (deps arrays, order)         │ Intermediate │
  ├──────────────────────┼───────────────────────────────────────────────────┼──────────────┤
  │ react-refresh plugin │ Warns about non-component exports in fast-refresh │ Intermediate │
  ├──────────────────────┼───────────────────────────────────────────────────┼──────────────┤
  │ no-unused-vars       │ Allow uppercase patterns (constants)              │ Beginner     │
  ├──────────────────────┼───────────────────────────────────────────────────┼──────────────┤
  │ eslint-plugin-react  │ React-specific linting rules                      │ Intermediate │
  └──────────────────────┴───────────────────────────────────────────────────┴──────────────┘

  5.3 TypeScript (for tests)

  ┌────────────────────┬──────────────────────────────────────────────┬──────────────┐
  │      Concept       │                  Where Used                  │  Difficulty  │
  ├────────────────────┼──────────────────────────────────────────────┼──────────────┤
  │ Interfaces         │ Playwright fixture types, API helper types   │ Intermediate │
  ├────────────────────┼──────────────────────────────────────────────┼──────────────┤
  │ Type annotations   │ Function parameters and returns              │ Intermediate │
  ├────────────────────┼──────────────────────────────────────────────┼──────────────┤
  │ Generics           │ Page, BrowserContext types from Playwright   │ Intermediate │
  ├────────────────────┼──────────────────────────────────────────────┼──────────────┤
  │ tsconfig.json      │ Compiler options, paths, targets             │ Intermediate │
  ├────────────────────┼──────────────────────────────────────────────┼──────────────┤
  │ Type imports       │ import type { Page } from '@playwright/test' │ Intermediate │
  ├────────────────────┼──────────────────────────────────────────────┼──────────────┤
  │ as type assertions │ Used in test helpers                         │ Intermediate │
  └────────────────────┴──────────────────────────────────────────────┴──────────────┘

  5.4 Git & Version Control

  ┌────────────────────────────┬───────────────────────────────────────┬────────────┐
  │          Concept           │              Where Used               │ Difficulty │
  ├────────────────────────────┼───────────────────────────────────────┼────────────┤
  │ Branch strategy            │ master (work), main (PRs target)      │ Beginner   │
  ├────────────────────────────┼───────────────────────────────────────┼────────────┤
  │ Merge commits              │ PR-based merge workflow (see git log) │ Beginner   │
  ├────────────────────────────┼───────────────────────────────────────┼────────────┤
  │ .gitignore                 │ Excludes node_modules, dist, .env     │ Beginner   │
  ├────────────────────────────┼───────────────────────────────────────┼────────────┤
  │ Commit message conventions │ feat:, fix:, refactor: prefixes       │ Beginner   │
  └────────────────────────────┴───────────────────────────────────────┴────────────┘

  5.5 Azure Pipelines (CI/CD)

  ┌─────────────────────────────┬──────────────────────────────────────┬──────────────┐
  │           Concept           │              Where Used              │  Difficulty  │
  ├─────────────────────────────┼──────────────────────────────────────┼──────────────┤
  │ azure-pipelines.yml         │ Automated build and test pipeline    │ Intermediate │
  ├─────────────────────────────┼──────────────────────────────────────┼──────────────┤
  │ Playwright in CI            │ retries: 1 in CI, screenshot capture │ Intermediate │
  ├─────────────────────────────┼──────────────────────────────────────┼──────────────┤
  │ Environment variables in CI │ VITE_API_BASE_URL injection          │ Intermediate │
  ├─────────────────────────────┼──────────────────────────────────────┼──────────────┤
  │ npm registry                │ .npmrc for Baylor private packages   │ Intermediate │
  └─────────────────────────────┴──────────────────────────────────────┴──────────────┘

  5.6 PostCSS & Tailwind Build

  ┌──────────────────────────────────────┬──────────────┐
  │               Concept                │  Difficulty  │
  ├──────────────────────────────────────┼──────────────┤
  │ postcss.config.js — plugin chain     │ Beginner     │
  ├──────────────────────────────────────┼──────────────┤
  │ Autoprefixer — adds vendor prefixes  │ Beginner     │
  ├──────────────────────────────────────┼──────────────┤
  │ Tailwind content scanning — JIT mode │ Intermediate │
  └──────────────────────────────────────┴──────────────┘

  5.7 Node.js & npm

  ┌─────────────────────────┬──────────────────────────────────────┬──────────────┐
  │         Concept         │              Where Used              │  Difficulty  │
  ├─────────────────────────┼──────────────────────────────────────┼──────────────┤
  │ package.json scripts    │ dev, build, test:* commands          │ Beginner     │
  ├─────────────────────────┼──────────────────────────────────────┼──────────────┤
  │ package-lock.json       │ Exact dependency versioning          │ Beginner     │
  ├─────────────────────────┼──────────────────────────────────────┼──────────────┤
  │ .npmrc                  │ Private registry for Baylor packages │ Intermediate │
  ├─────────────────────────┼──────────────────────────────────────┼──────────────┤
  │ node_modules resolution │ Module resolution order              │ Intermediate │
  ├─────────────────────────┼──────────────────────────────────────┼──────────────┤
  │ Private packages        │ @baylor/pdfme-* — scoped packages    │ Intermediate │
  └─────────────────────────┴──────────────────────────────────────┴──────────────┘

  ---
  Section 6: Suggested Learning Roadmap (Step-by-Step)

  Phase 1: Foundation (Weeks 1–2)

  Goal: Understand the building blocks used by every file in this codebase.

  ┌──────┬────────────────────────────────────────────────────────────────────────────┬───────────────────┬────────────────┐
  │ Step │                                   Topic                                    │   Resource Type   │   Difficulty   │
  ├──────┼────────────────────────────────────────────────────────────────────────────┼───────────────────┼────────────────┤
  │ 1    │ JavaScript ES6+: destructuring, spread, arrow functions, optional chaining │ Hands-on practice │ Beginner       │
  ├──────┼────────────────────────────────────────────────────────────────────────────┼───────────────────┼────────────────┤
  │ 2    │ JavaScript async: promises, async/await, try/catch, Promise.all            │ Code exercises    │ Intermediate   │
  ├──────┼────────────────────────────────────────────────────────────────────────────┼───────────────────┼────────────────┤
  │ 3    │ JavaScript modules: import/export, named vs default, barrel files          │ Read + code       │ Beginner       │
  ├──────┼────────────────────────────────────────────────────────────────────────────┼───────────────────┼────────────────┤
  │ 4    │ JavaScript closures and the this keyword                                   │ Deep reading      │ Intermediate   │
  ├──────┼────────────────────────────────────────────────────────────────────────────┼───────────────────┼────────────────┤
  │ 5    │ Array methods: map, filter, reduce, find, flatMap                          │ Exercises         │ Beginner-Inter │
  ├──────┼────────────────────────────────────────────────────────────────────────────┼───────────────────┼────────────────┤
  │ 6    │ JavaScript classes and custom error types                                  │ Code examples     │ Intermediate   │
  └──────┴────────────────────────────────────────────────────────────────────────────┴───────────────────┴────────────────┘

  ---
  Phase 2: React Core (Weeks 3–4)

  Goal: Understand the React patterns used in every component.

  ┌──────┬───────────────────────────────────────────────────────────────┬──────────────┐
  │ Step │                             Topic                             │  Difficulty  │
  ├──────┼───────────────────────────────────────────────────────────────┼──────────────┤
  │ 7    │ React JSX, functional components, props, events               │ Beginner     │
  ├──────┼───────────────────────────────────────────────────────────────┼──────────────┤
  │ 8    │ useState, useEffect (dependencies, cleanup)                   │ Beginner     │
  ├──────┼───────────────────────────────────────────────────────────────┼──────────────┤
  │ 9    │ Controlled forms and validation patterns                      │ Intermediate │
  ├──────┼───────────────────────────────────────────────────────────────┼──────────────┤
  │ 10   │ Conditional rendering, list rendering with keys               │ Beginner     │
  ├──────┼───────────────────────────────────────────────────────────────┼──────────────┤
  │ 11   │ Lifting state up, component composition                       │ Intermediate │
  ├──────┼───────────────────────────────────────────────────────────────┼──────────────┤
  │ 12   │ useRef — DOM access AND value persistence across renders      │ Intermediate │
  ├──────┼───────────────────────────────────────────────────────────────┼──────────────┤
  │ 13   │ useCallback, useMemo — when and why to memoize                │ Intermediate │
  ├──────┼───────────────────────────────────────────────────────────────┼──────────────┤
  │ 14   │ Custom hooks — build useFormValidation from scratch           │ Intermediate │
  ├──────┼───────────────────────────────────────────────────────────────┼──────────────┤
  │ 15   │ React 18: createRoot, StrictMode behavior, automatic batching │ Intermediate │
  └──────┴───────────────────────────────────────────────────────────────┴──────────────┘

  ---
  Phase 3: State Management with Redux Toolkit (Week 5)

  Goal: Understand how global state flows through the entire app.

  ┌──────┬────────────────────────────────────────────────────────────┬──────────────┐
  │ Step │                           Topic                            │  Difficulty  │
  ├──────┼────────────────────────────────────────────────────────────┼──────────────┤
  │ 16   │ Redux fundamentals: store, actions, reducers               │ Intermediate │
  ├──────┼────────────────────────────────────────────────────────────┼──────────────┤
  │ 17   │ createSlice() — reducers + auto-generated actions          │ Intermediate │
  ├──────┼────────────────────────────────────────────────────────────┼──────────────┤
  │ 18   │ createAsyncThunk() — pending/fulfilled/rejected lifecycle  │ Intermediate │
  ├──────┼────────────────────────────────────────────────────────────┼──────────────┤
  │ 19   │ extraReducers — handling thunk actions                     │ Intermediate │
  ├──────┼────────────────────────────────────────────────────────────┼──────────────┤
  │ 20   │ useSelector / useDispatch in components                    │ Intermediate │
  ├──────┼────────────────────────────────────────────────────────────┼──────────────┤
  │ 21   │ Selectors: selectAllTemplates, selectTemplateById patterns │ Intermediate │
  ├──────┼────────────────────────────────────────────────────────────┼──────────────┤
  │ 22   │ Immer.js — why direct mutations are safe in RTK            │ Intermediate │
  └──────┴────────────────────────────────────────────────────────────┴──────────────┘

  ---
  Phase 4: React Router v7 (Week 6)

  Goal: Understand routing and navigation patterns.

  ┌──────┬──────────────────────────────────────────────────┬──────────────┐
  │ Step │                      Topic                       │  Difficulty  │
  ├──────┼──────────────────────────────────────────────────┼──────────────┤
  │ 23   │ BrowserRouter, Routes, Route setup               │ Beginner     │
  ├──────┼──────────────────────────────────────────────────┼──────────────┤
  │ 24   │ useNavigate(), useParams(), useLocation()        │ Intermediate │
  ├──────┼──────────────────────────────────────────────────┼──────────────┤
  │ 25   │ Navigation guards (unsaved changes interception) │ Advanced     │
  ├──────┼──────────────────────────────────────────────────┼──────────────┤
  │ 26   │ Route-level data passing via state in navigation │ Intermediate │
  └──────┴──────────────────────────────────────────────────┴──────────────┘

  ---
  Phase 5: API Layer (Week 7)

  Goal: Understand how HTTP requests are made and errors are handled.

  ┌──────┬─────────────────────────────────────────────────────────────────┬──────────────┐
  │ Step │                              Topic                              │  Difficulty  │
  ├──────┼─────────────────────────────────────────────────────────────────┼──────────────┤
  │ 27   │ Axios: instance, config, methods                                │ Intermediate │
  ├──────┼─────────────────────────────────────────────────────────────────┼──────────────┤
  │ 28   │ Axios interceptors: request (auth) + response (transform/error) │ Advanced     │
  ├──────┼─────────────────────────────────────────────────────────────────┼──────────────┤
  │ 29   │ Custom APIError class — error taxonomy                          │ Intermediate │
  ├──────┼─────────────────────────────────────────────────────────────────┼──────────────┤
  │ 30   │ Service layer pattern — why abstract API calls                  │ Intermediate │
  ├──────┼─────────────────────────────────────────────────────────────────┼──────────────┤
  │ 31   │ Environment variables in Vite (import.meta.env)                 │ Intermediate │
  └──────┴─────────────────────────────────────────────────────────────────┴──────────────┘

  ---
  Phase 6: UI Libraries (Week 8)

  Goal: Use and customize Mantine and icon libraries.

  ┌──────┬────────────────────────────────────────────────────┬──────────────┐
  │ Step │                       Topic                        │  Difficulty  │
  ├──────┼────────────────────────────────────────────────────┼──────────────┤
  │ 32   │ Mantine: MantineProvider, basic components         │ Beginner     │
  ├──────┼────────────────────────────────────────────────────┼──────────────┤
  │ 33   │ Mantine notifications system                       │ Intermediate │
  ├──────┼────────────────────────────────────────────────────┼──────────────┤
  │ 34   │ Mantine hooks (useDebouncedValue)                  │ Intermediate │
  ├──────┼────────────────────────────────────────────────────┼──────────────┤
  │ 35   │ Tailwind CSS utility classes and responsive design │ Beginner     │
  ├──────┼────────────────────────────────────────────────────┼──────────────┤
  │ 36   │ Lucide React and Tabler icons                      │ Beginner     │
  └──────┴────────────────────────────────────────────────────┴──────────────┘

  ---
  Phase 7: Advanced React Patterns (Week 9)

  Goal: Understand the most complex React patterns in this codebase.

  ┌──────┬──────────────────────────────────────────────────────────┬──────────────┐
  │ Step │                          Topic                           │  Difficulty  │
  ├──────┼──────────────────────────────────────────────────────────┼──────────────┤
  │ 37   │ forwardRef — exposing component refs to parents          │ Advanced     │
  ├──────┼──────────────────────────────────────────────────────────┼──────────────┤
  │ 38   │ useImperativeHandle — imperative API over a component    │ Advanced     │
  ├──────┼──────────────────────────────────────────────────────────┼──────────────┤
  │ 39   │ React.memo — preventing re-renders                       │ Intermediate │
  ├──────┼──────────────────────────────────────────────────────────┼──────────────┤
  │ 40   │ Stale closure problem — why useRef is used for callbacks │ Advanced     │
  ├──────┼──────────────────────────────────────────────────────────┼──────────────┤
  │ 41   │ Debouncing in React — debounced API sync pattern         │ Intermediate │
  ├──────┼──────────────────────────────────────────────────────────┼──────────────┤
  │ 42   │ Hash-based change detection pattern                      │ Advanced     │
  └──────┴──────────────────────────────────────────────────────────┴──────────────┘

  ---
  Phase 8: Tooling Deep Dive (Week 10)

  Goal: Understand the build and development environment.

  ┌──────┬─────────────────────────────────────────────┬──────────────┐
  │ Step │                    Topic                    │  Difficulty  │
  ├──────┼─────────────────────────────────────────────┼──────────────┤
  │ 43   │ Vite config: plugins, aliases, optimizeDeps │ Intermediate │
  ├──────┼─────────────────────────────────────────────┼──────────────┤
  │ 44   │ ESLint flat config + React hooks rules      │ Intermediate │
  ├──────┼─────────────────────────────────────────────┼──────────────┤
  │ 45   │ TypeScript basics (for reading test files)  │ Intermediate │
  ├──────┼─────────────────────────────────────────────┼──────────────┤
  │ 46   │ PostCSS + Tailwind build pipeline           │ Intermediate │
  └──────┴─────────────────────────────────────────────┴──────────────┘

  ---
  Phase 9: pdfme PDF Engine (Week 11–12)

  Goal: Understand the most unique and complex part of this codebase.

  ┌──────┬───────────────────────────────────────────────────────────────┬──────────────┐
  │ Step │                             Topic                             │  Difficulty  │
  ├──────┼───────────────────────────────────────────────────────────────┼──────────────┤
  │ 47   │ pdfme concepts: template schema, inputs, plugins              │ Advanced     │
  ├──────┼───────────────────────────────────────────────────────────────┼──────────────┤
  │ 48   │ Designer component: initialization, callbacks, schema changes │ Advanced     │
  ├──────┼───────────────────────────────────────────────────────────────┼──────────────┤
  │ 49   │ Viewer component: sample data binding                         │ Advanced     │
  ├──────┼───────────────────────────────────────────────────────────────┼──────────────┤
  │ 50   │ generate(): produce PDF binary from schema + inputs           │ Advanced     │
  ├──────┼───────────────────────────────────────────────────────────────┼──────────────┤
  │ 51   │ Field plugins: text, image, table, barcodes                   │ Advanced     │
  ├──────┼───────────────────────────────────────────────────────────────┼──────────────┤
  │ 52   │ Custom datasource binding to pdfme fields                     │ Advanced     │
  ├──────┼───────────────────────────────────────────────────────────────┼──────────────┤
  │ 53   │ Font loading and registration                                 │ Intermediate │
  └──────┴───────────────────────────────────────────────────────────────┴──────────────┘

  ---
  Phase 10: Testing with Playwright (Week 13)

  Goal: Understand and write E2E tests.

  ┌──────┬────────────────────────────────────────────────────┬──────────────┐
  │ Step │                       Topic                        │  Difficulty  │
  ├──────┼────────────────────────────────────────────────────┼──────────────┤
  │ 54   │ Playwright basics: test, expect, page.goto         │ Beginner     │
  ├──────┼────────────────────────────────────────────────────┼──────────────┤
  │ 55   │ Locators: page.locator(), getByText(), getByRole() │ Intermediate │
  ├──────┼────────────────────────────────────────────────────┼──────────────┤
  │ 56   │ Page Object Model pattern                          │ Intermediate │
  ├──────┼────────────────────────────────────────────────────┼──────────────┤
  │ 57   │ Test fixtures (test.extend())                      │ Advanced     │
  ├──────┼────────────────────────────────────────────────────┼──────────────┤
  │ 58   │ Network mocking (page.route())                     │ Advanced     │
  ├──────┼────────────────────────────────────────────────────┼──────────────┤
  │ 59   │ API helpers inside Playwright tests                │ Advanced     │
  └──────┴────────────────────────────────────────────────────┴──────────────┘

  ---
  Section 7: Estimated Time to Become Comfortable in This Codebase

  ┌──────────────────────────┬───────────────┬─────────────────────────────────────────────────────────────────────────────┐
  │          Level           │ Time Estimate │                               What You Can Do                               │
  ├──────────────────────────┼───────────────┼─────────────────────────────────────────────────────────────────────────────┤
  │ Read with understanding  │ 4–6 weeks     │ Trace data flow, understand component roles, read test output               │
  ├──────────────────────────┼───────────────┼─────────────────────────────────────────────────────────────────────────────┤
  │ Fix simple bugs          │ 6–8 weeks     │ Fix validation bugs, update UI text, add small features to modals           │
  ├──────────────────────────┼───────────────┼─────────────────────────────────────────────────────────────────────────────┤
  │ Add features confidently │ 3–4 months    │ Add new modals, new service methods, new Redux thunks, new Playwright tests │
  ├──────────────────────────┼───────────────┼─────────────────────────────────────────────────────────────────────────────┤
  │ Modify DesignerCanvas    │ 4–6 months    │ Requires deep pdfme understanding + stale closure mastery                   │
  ├──────────────────────────┼───────────────┼─────────────────────────────────────────────────────────────────────────────┤
  │ Own the architecture     │ 6–12 months   │ Refactor the God component, add TypeScript, improve test coverage           │
  └──────────────────────────┴───────────────┴─────────────────────────────────────────────────────────────────────────────┘

  Breakdown by Background

  ┌───────────────────────────────────┬───────────────────┐
  │          Your Background          │ Estimated Ramp-Up │
  ├───────────────────────────────────┼───────────────────┤
  │ Junior React Dev (1 year)         │ 4–6 months        │
  ├───────────────────────────────────┼───────────────────┤
  │ Mid-level React Dev (2–3 years)   │ 2–3 months        │
  ├───────────────────────────────────┼───────────────────┤
  │ Senior React Dev (3+ years)       │ 4–6 weeks         │
  ├───────────────────────────────────┼───────────────────┤
  │ Senior + Redux + pdfme experience │ 1–2 weeks         │
  └───────────────────────────────────┴───────────────────┘

  Hardest Parts to Learn (in order)

  1. DesignerCanvas.jsx — 68KB, forwardRef, stale closure refs, debounced API sync, pdfme integration (2–4 weeks alone)
  2. pdfme schema system — unique domain, not standard docs (1–2 weeks)
  3. fieldGroupManager.js — 580KB utility, complex group operations logic (1 week)
  4. Redux thunks + slice — 28KB of async operations and state (3–5 days)
  5. Playwright fixtures + Page Objects — advanced test patterns (3–5 days)

  Fastest Paths to Contribution

  If you want to contribute quickly, start with:
  - Modals (components/modals/*.jsx) — self-contained, well-structured, no pdfme dependency
  - API services (api/services/*.js) — simple CRUD, easy to extend
  - Custom hooks (hooks/) — isolated, testable units
  - Utility functions (utils/validators.js, utils/dateUtils.js) — pure functions

  ---
  This roadmap covers every concept used directly or indirectly in this codebase. Completing all phases makes you fully equipped to contribute to, maintain, and extend the Report Builder
  UI without surprises.
