# Project Notes: Template and Version Lifecycle

> Scope: Frontend flow for templates, versions, editor, and related API/state files.

## Quick Overview

This app is a **Report Template Builder**.

- A `Template` contains multiple `Versions`
- A `Version` contains `Pages`
- A `Page` contains `Fields`
- `Field Groups` are reusable field sets

Core stack:

- React + Vite
- Redux (`src/store/templatesSlice.js`)
- Axios service layer (`src/api/services/*`)
- pdfme designer/preview
- Mantine UI + notifications

---

## App Entry and Routes

### Entry

1. `src/main.jsx`
2. `src/App.jsx`

### Main Routes

| Route                                                | Page                                                | Purpose                                 |
| ---------------------------------------------------- | --------------------------------------------------- | --------------------------------------- |
| `/`                                                  | `src/pages/TemplateList.jsx`                        | List templates, create template         |
| `/template/:id`                                      | `src/pages/TemplateDetail.jsx`                      | Version list and version actions        |
| `/editor/:templateId`                                | `src/pages/TemplateEditor.jsx`                      | Edit/version designer                   |
| `/editor/new-version/:baseTemplateId/:baseVersionId` | `src/pages/TemplateEditor.jsx`                      | Create/edit from base version           |
| `/test-configs`                                      | `src/components/viewer/TestConfigurationViewer.jsx` | Test config viewer (currently disabled) |

---

## Lifecycle Notes

## 1. Create Template

**UI files**

- `src/pages/TemplateList.jsx`
- `src/components/template/NewTemplateWizard.jsx`

**Flow**

1. User clicks **New Template**.
2. Wizard loads code options from `public/codes.json`.
3. Validation runs.
4. `createTemplate` thunk is dispatched.
5. API creates template, then user is moved toward editor flow.

**Related files**

- `src/store/templatesSlice.js`
- `src/api/services/templateService.js`
- `src/utils/validation/validators.js`
- `src/hooks/useNotification.js`

---

## 2. Create Version

**UI files**

- `src/pages/TemplateDetail.jsx`
- `src/components/modals/SaveVersionModal.jsx`

**Flow**

1. User clicks **New Version**.
2. Version label is validated.
3. `createTemplateVersion` thunk runs.
4. Initial page is created for the new version.

**Related files**

- `src/store/templatesSlice.js`
- `src/api/services/templateService.js`
- `src/api/services/templatePageService.js`
- `src/constants/versionDefaults.js`

---

## 3. Duplicate Version

**UI file**

- `src/components/modals/DuplicateVersionModal.jsx`

**Flow**

1. User provides a new version label.
2. `cloneTemplateVersion` thunk is dispatched.
3. Backend clones source version.

**Related files**

- `src/store/templatesSlice.js`
- `src/api/services/templateVersionService.js`

---

## 4. Edit Version (Designer)

**UI files**

- `src/pages/TemplateEditor.jsx`
- `src/components/designer/DesignerCanvas.jsx`
- `src/components/designer/PreviewCanvas.jsx`

**Flow**

1. Load schema from backend (or blank/default).
2. Render pdfme designer.
3. User edits fields/pages/groups.
4. Save schema through Redux thunk to backend.

**Related files**

- `src/store/templatesSlice.js`
- `src/utils/schemaLoader.js`
- `src/utils/fieldTransformer.js`
- `src/utils/fieldTypeMapping.js`
- `src/api/services/templateVersionService.js`
- `src/api/services/templateFieldService.js`
- `src/api/services/templatePageService.js`
- `src/api/services/templateFieldGroupService.js`

---

## 5. Publish Version

**UI file**

- `src/components/modals/AddCommentModal.jsx`

**Flow**

1. User clicks publish.
2. Publish comment is collected.
3. Publish API marks version active.

**Related files**

- `src/store/templatesSlice.js`
- `src/api/services/templateVersionService.js`

---

## 6. Rename / Delete

### Rename Version

- UI: `src/components/modals/RenameModal.jsx`
- State: `src/store/templatesSlice.js`
- API: `src/api/services/templateService.js`

### Delete Version

- UI: `src/components/modals/DeleteModal.jsx`
- State: `src/store/templatesSlice.js`
- API: `src/api/services/templateService.js` and/or version service methods

### Delete Template

- UI: `src/components/modals/DeleteTemplateModal.jsx`
- State: `src/store/templatesSlice.js`
- API: `src/api/services/templateService.js`

---

## 7. Comments

- UI: `src/components/modals/CommentsModal.jsx`
- State: `src/store/templatesSlice.js`
- API: `src/api/services/templateVersionService.js`

Used to fetch and display per-version comment history.

---

## API Surface (High Level)

| Domain       | Typical Actions                                  |
| ------------ | ------------------------------------------------ |
| Templates    | create, list, delete                             |
| Versions     | create, clone, update, publish, delete, comments |
| Pages        | create (bulk), update, reorder, delete           |
| Fields       | create, update, batch create                     |
| Field Groups | create, copy, list, delete                       |

Primary files:

- `src/api/client.js`
- `src/api/config/apiConfig.js`
- `src/api/services/templateService.js`
- `src/api/services/templateVersionService.js`
- `src/api/services/templatePageService.js`
- `src/api/services/templateFieldService.js`
- `src/api/services/templateFieldGroupService.js`

---

## Redux Notes

- Store setup: `src/store/store.js`
- Main lifecycle thunks/selectors: `src/store/templatesSlice.js`

`templatesSlice` is the central orchestrator between UI and API for template/version lifecycle actions.

---

## Important Cleanup Notes

- Legacy duplicate component exists: `src/components/DesignerCanvas.jsx`
- Active designer component: `src/components/designer/DesignerCanvas.jsx`

If no imports reference the legacy file, it can be considered stale.
