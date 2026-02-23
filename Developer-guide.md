# Developer Guide: Template and Version Lifecycle

> Scope: Frontend flow for templates, versions, editor operations, and related state/API files.

## 1. App Entry and Routing

### Entry Points

1. `src/main.jsx`
2. `src/App.jsx`

### Route Map

| Route                                                | Component                                           | Responsibility                               |
| ---------------------------------------------------- | --------------------------------------------------- | -------------------------------------------- |
| `/`                                                  | `src/pages/TemplateList.jsx`                        | Template listing, search, create entry point |
| `/template/:id`                                      | `src/pages/TemplateDetail.jsx`                      | Version list and actions                     |
| `/editor/:templateId`                                | `src/pages/TemplateEditor.jsx`                      | Main editor flow                             |
| `/editor/new-version/:baseTemplateId/:baseVersionId` | `src/pages/TemplateEditor.jsx`                      | Derivative/new-version editing flow          |
| `/test-configs`                                      | `src/components/viewer/TestConfigurationViewer.jsx` | Test configuration viewer                    |

---

## 2. Create Template

### UI Layer

- `src/pages/TemplateList.jsx`
- `src/components/template/NewTemplateWizard.jsx`

### Flow

1. User opens wizard from template list page.
2. Wizard loads options from `public/codes.json`.
3. Form validation runs.
4. `createTemplate` thunk is dispatched.
5. API call creates template record.

### Related Files

- `src/store/templatesSlice.js`
- `src/api/services/templateService.js`
- `src/utils/validation/validators.js`
- `src/hooks/useNotification.js`

---

## 3. Create Version

### UI Layer

- `src/pages/TemplateDetail.jsx`
- `src/components/modals/SaveVersionModal.jsx`

### Flow

1. User clicks **New Version**.
2. Modal validates version format and uniqueness.
3. `createTemplateVersion` thunk dispatches API create call.
4. Initial page is created for the new version.

### Related Files

- `src/store/templatesSlice.js`
- `src/api/services/templateService.js`
- `src/api/services/templatePageService.js`
- `src/constants/versionDefaults.js`

---

## 4. Duplicate Version

### UI Layer

- `src/components/modals/DuplicateVersionModal.jsx`

### Flow

1. User enters target version label.
2. `cloneTemplateVersion` thunk executes.
3. Backend clones schema/metadata into new version.

### Related Files

- `src/store/templatesSlice.js`
- `src/api/services/templateVersionService.js`

---

## 5. Edit Version (Designer)

### UI Layer

- `src/pages/TemplateEditor.jsx`
- `src/components/designer/DesignerCanvas.jsx`
- `src/components/designer/PreviewCanvas.jsx`

### Flow

1. Load version schema from backend or defaults.
2. Initialize pdfme designer with plugins/fonts/options.
3. User edits fields/pages/groups.
4. Save operation dispatches schema thunk.
5. Backend persists latest schema.

### Related Files

- `src/store/templatesSlice.js`
- `src/utils/schemaLoader.js`
- `src/utils/fieldTransformer.js`
- `src/utils/fieldTypeMapping.js`
- `src/api/services/templateVersionService.js`
- `src/api/services/templateFieldService.js`
- `src/api/services/templatePageService.js`
- `src/api/services/templateFieldGroupService.js`

---

## 6. Publish Version

### UI Layer

- `src/components/modals/AddCommentModal.jsx`

### Flow

1. User triggers publish action.
2. Publish comment is collected/validated.
3. Publish API marks target version active.

### Related Files

- `src/store/templatesSlice.js`
- `src/api/services/templateVersionService.js`

---

## 7. Rename and Delete Operations

### Rename Version

- UI: `src/components/modals/RenameModal.jsx`
- State: `src/store/templatesSlice.js`
- API: `src/api/services/templateService.js`

### Delete Version

- UI: `src/components/modals/DeleteModal.jsx`
- State: `src/store/templatesSlice.js`
- API: version/template delete endpoints via service layer

### Delete Template

- UI: `src/components/modals/DeleteTemplateModal.jsx`
- State: `src/store/templatesSlice.js`
- API: `src/api/services/templateService.js`

---

## 8. Comments

- UI: `src/components/modals/CommentsModal.jsx`
- State: `src/store/templatesSlice.js`
- API: `src/api/services/templateVersionService.js`

Used to fetch and display per-version comment history.

---

## 9. API Layer Overview

| Domain       | Typical Actions                                  |
| ------------ | ------------------------------------------------ |
| Templates    | create, list, delete                             |
| Versions     | create, clone, update, publish, delete, comments |
| Pages        | create/bulk create, reorder, update, delete      |
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

## 10. Redux Responsibilities

- Store config: `src/store/store.js`
- Lifecycle thunks/selectors: `src/store/templatesSlice.js`

`templatesSlice` coordinates most template/version workflows between UI and API.

---

## 11. Cleanup/Tech-Debt Note

- Possible stale duplicate component: `src/components/DesignerCanvas.jsx`
- Active designer: `src/components/designer/DesignerCanvas.jsx`

Confirm imports before removing legacy component.

# Field‑Level Lifecycle (Pages, Fields, Field Groups)

## A. Pages

Where pages are created

- New version creation in src/pages/TemplateDetail.jsx
- Editor adds pages via pdfme (Designer)

Key files

- src/api/services/templatePageService.js
  API wrapper for page CRUD and bulk create.
- src/store/templatesSlice.js
  Thunks for creating/fetching pages.

Flow

1. New Version → Initial Page
   TemplateDetail calls createTemplatePages thunk.
   API: POST /v1/template-pages/bulk.
2. Editor Add/Delete Page
   DesignerCanvas listens to pdfme page changes and can call:
   - templatePageService.create / createBulk
   - templatePageService.delete
3. Reorder Pages
   templatePageService.reorder updates page ordering.

———

## B. Fields (pdfme elements)

Where fields are created/updated

- In src/components/designer/DesignerCanvas.jsx

Key files

- src/components/designer/DesignerCanvas.jsx
  Detects schema changes and syncs fields.
- src/utils/fieldTransformer.js
  Converts between pdfme fields and backend field model.
- src/utils/fieldTypeMapping.js
  Maps pdfme field type → backend UUID.
- src/api/services/templateFieldService.js
  Backend CRUD for fields.

Flow

1. Add Field in Designer
   pdfme updates schema; DesignerCanvas detects new fields.
2. Transform Field
   pdfmeToApiField converts to API format.
3. Sync to Backend
   templateFieldService.create (or createBatch).
4. Update Field
   Field changes are transformed and sent via templateFieldService.update.
5. Delete Field
   Removal in pdfme triggers templateFieldService.delete.

Important

- Field type IDs are controlled by src/utils/fieldTypeMapping.js.
- The fieldBackendId is preserved in schema to avoid re‑creating existing fields.

———

## C. Field Groups

What they are

- Reusable collections of fields (e.g., WGS report header).
- Can be global or local to a version.

Key files

- src/components/designer/FieldGroupPanel.jsx
  UI for managing groups and applying them to templates.
- src/utils/fieldGroupManager.js
  Local schema helper for inserting/removing group fields.
- src/api/services/templateFieldGroupService.js
  API calls for group CRUD and global group copy.

Flow

1. Load Groups
   - Global groups: templateFieldGroupService.getGlobalFieldGroups
   - Local groups: templateFieldGroupService.getAll
2. Apply Group to Template
   - Uses fieldGroupManager.applyToAllPages to insert fields.
3. Create Group
   - templateFieldGroupService.create
   - Fields are then associated with the group.
4. Copy Global Group
   - templateFieldGroupService.copyGlobalFieldGroup
   - Adds group fields to pages.
5. Delete Group
   - templateFieldGroupService.delete or cascadeDelete
   - Removes from backend and optionally from schema.

———

## D. Datasource Binding (Field Content)

Purpose

- Fields can bind to data placeholders like {{Patient.FirstName}}.

Key files

- src/utils/datasourceSchema.js
  Static/dynamic datasource options and sample data generation.
- src/utils/dynamicDatasourceOptions.js
  Builds dropdown options for the pdfme property panel.
- src/components/datasource/SmartDatasourcePicker.jsx
  UI dropdown for selecting datasource fields.
- src/components/datasource/DynamicDatasourceField.jsx
  Entity/property picker.

Flow

1. Designer loads datasource options.
2. User selects placeholder fields.
3. Preview uses schemaLoader.generateInputsForSchema to inject sample values.

———

## E. Preview Rendering (Fields in PDF)

Key files

- src/components/designer/PreviewCanvas.jsx
- src/utils/schemaLoader.js

Flow

1. Preview gets schema + inputs.
2. pdfme viewer renders field content.
3. If conditions exist, condition input is requested before rendering.

———

## Quick Reference: Field‑Level APIs

      - POST /v1/template-pages
      - POST /v1/template-pages/bulk
      - PUT /v1/template-pages/reorder

- Fields:
  - POST /v1/template-fields
  - POST /v1/template-fields/batch
  - PATCH /v1/template-fields/:id
  - DELETE /v1/template-fields/:id
- Field Groups:
  - GET /v1/template-field-groups
  - POST /v1/template-field-groups
  - PATCH /v1/template-field-groups/:id
  - DELETE /v1/template-field-groups/:id
  - POST /v1/template-field-groups/:id/copy

———
———

## Happy Path: Template → Version → Edit → Save → Publish

1. User opens app
   src/main.jsx → src/App.jsx
   └─ routes user to "/"

2. User clicks "New Template"
   src/pages/TemplateList.jsx
   └─ opens src/components/template/NewTemplateWizard.jsx

3. User submits new template form
   NewTemplateWizard.jsx
   └─ validates with src/utils/validation/validators.js
   └─ dispatches createTemplate (src/store/templatesSlice.js)
   └─ API: src/api/services/templateService.js
   POST /v1/templates

4. User goes to template detail
   Route: /template/:id → src/pages/TemplateDetail.jsx
   └─ fetchTemplates + fetchTemplateVersions
   └─ API: templateService.getAll / getVersions

5. User clicks "New Version"
   TemplateDetail.jsx
   └─ opens SaveVersionModal.jsx
   └─ dispatches createTemplateVersion
   └─ API: templateService.createVersion
   POST /v1/templates/:id/versions
   └─ creates first page
   └─ API: templatePageService.createBulk
   POST /v1/template-pages/bulk

6. User clicks "Edit" version
   TemplateDetail.jsx
   └─ navigate to /editor/:versionId
   → src/pages/TemplateEditor.jsx

7. Editor loads schema
   TemplateEditor.jsx
   └─ dispatch loadVersionSchema
   └─ API: templateVersionService.getSchema
   GET /api/TemplateVersions/:versionId/schema
   └─ pdfme initialized
   └─ src/components/designer/DesignerCanvas.jsx
   └─ fonts loaded: src/utils/fonts.js

8. User adds a field (text/table/etc.)
   DesignerCanvas.jsx
   └─ detects schema change
   └─ transforms field: src/utils/fieldTransformer.js
   └─ maps type: src/utils/fieldTypeMapping.js
   └─ optional sync to backend:
   └─ templateFieldService.create
   POST /v1/template-fields

9. User clicks "Save"
   TemplateEditor.jsx
   └─ dispatch saveVersionSchema
   └─ API: templateVersionService.saveSchema
   PUT /api/TemplateVersions/:versionId/schema

10. User clicks "Publish"
    TemplateEditor.jsx
    └─ opens AddCommentModal.jsx
    └─ on confirm:
    └─ API: templateVersionService.publish
    POST /v1/templates/:templateId/versions/:versionId/publish

———

## Optional Happy Path: Duplicate Version

User clicks "Duplicate"
TemplateDetail.jsx
└─ DuplicateVersionModal.jsx
└─ dispatch cloneTemplateVersion
└─ API: templateVersionService.clone
POST /v1/templates/:templateId/versions/:versionId/clone

———

## Optional Happy Path: Delete Version / Template

Delete Version:
TemplateDetail.jsx → DeleteModal.jsx
└─ dispatch deleteTemplateVersion
└─ API: DELETE /v1/templates/:templateId/versions/:versionId

Delete Template:
TemplateDetail.jsx → DeleteTemplateModal.jsx
└─ dispatch deleteTemplate
└─ API: DELETE /v1/templates/:templateId

# Canvas Architecture Overview

The “canvas” is the pdfme Designer wrapped by your React component. The flow looks like this:

1. TemplateEditor.jsx owns the editing state and passes a template to the canvas.
2. DesignerCanvas.jsx (the one under src/components/designer/) initializes pdfme Designer with plugins.
3. pdfme Designer renders the canvas, manages selection, resizing, dragging, and emits schema changes.
4. Changes are pushed back to React state and optionally synced to backend fields/pages.

———

# Main Files Involved

## 1) Canvas Wrapper (Core)

src/components/designer/DesignerCanvas.jsx
This is the real “canvas engine wrapper” for the editor.

- Creates the pdfme Designer instance.
- Supplies plugins for element types.
- Supplies datasource options for binding fields.
- Handles onChangeTemplate (schema updates).
- Synchronizes changes with API (fields/pages/groups).

Important: There is also an older duplicate file
src/components/DesignerCanvas.jsx
It is not used by the editor route. The active one is the one inside src/components/designer/.

———

## 2) Palette UI (Buttons to Add Elements)

src/components/designer/ElementPalette.jsx
This renders buttons like Text, Image, Line, Rectangle, Ellipse, Table.
When clicked, it calls onAddElement(type, defaultProps).

———

## 3) Editor Orchestrator

src/pages/TemplateEditor.jsx
This connects Palette → Canvas:

- Holds the current template state
- Passes onAddElement to the palette
- Passes template + onTemplateChange to DesignerCanvas

———

## 4) pdfme Plugins (Actual element types)

Provided by:
@baylor/pdfme-ui and @baylor/pdfme-schemas

DesignerCanvas imports:

import { Designer } from '@baylor/pdfme-ui'
import {
text, image, svg, table, barcodes,
line, rectangle, ellipse,
dateTime, date, time,
select, radioGroup, checkbox
} from '@baylor/pdfme-schemas'

These plugins define how each element is rendered and edited.

———

# How Adding Elements Works (Step‑by‑Step)

### Step 1: User clicks a palette button

File: src/components/designer/ElementPalette.jsx

Example element definition:

{
type: 'text',
name: 'Text Field',
icon: <Type />,
defaultProps: { width: 120, height: 20, fontSize: 12, text: 'Click to edit text' }
}

On click:

onAddElement(element.type, element.defaultProps)

———

### Step 2: Editor inserts element into the template schema

File: src/pages/TemplateEditor.jsx

It receives onAddElement and tells the DesignerCanvas (or directly updates template state) to insert a new field into the
current page schema.

The schema is a structure like:

template.schemas[pageIndex][fieldName] = {
type: 'text',
position: { x: 20, y: 30 },
width: 120,
height: 20,
fontSize: 12,
content: 'Click to edit text'
}

———

### Step 3: DesignerCanvas updates pdfme Designer

File: src/components/designer/DesignerCanvas.jsx

When template changes, it calls pdfme Designer’s update and re-renders the canvas.

———

### Step 4: pdfme Designer renders element

Library: @baylor/pdfme-ui + @baylor/pdfme-schemas

Each plugin (text, image, line, ellipse, etc.) knows how to draw and resize on canvas.

———

### Step 5: User edits the element on canvas

(pdfme Designer internally handles selection, dragging, resizing)

It emits a new template via onChangeTemplate.

———

### Step 6: React state updates

DesignerCanvas.jsx receives the new template and calls onTemplateChange → TemplateEditor.jsx.

———

# How Each Element Type is Supported

Element types come from pdfme schema plugins, used in DesignerCanvas.jsx:

- text → text fields
- image → images (incl. signature)
- line → line
- rectangle → rectangle
- ellipse → circle/ellipse
- table → tables
- svg, barcodes, date, time, etc. (supported but not all exposed in palette)

Where their default props are set

- ElementPalette.jsx sets base dimensions, font size, and content.
- pdfme propPanel allows further property editing.

———

# How Backend Sync Works (Optional)

File: src/components/designer/DesignerCanvas.jsx

If enableFieldSync is true:

1. Every field is transformed via:
   - src/utils/fieldTransformer.js
   - src/utils/fieldTypeMapping.js
2. Sent to backend through:
   - src/api/services/templateFieldService.js
3. Page changes use:
   - src/api/services/templatePageService.js

———

# Why Canvas Works Smoothly

Integration summary

1. TemplateEditor.jsx owns the state and passes it down.
2. DesignerCanvas.jsx wraps pdfme and subscribes to changes.
3. ElementPalette.jsx provides new fields with sensible defaults.
4. pdfme plugins handle rendering and editing.
5. The template schema stays the single source of truth.

# Quick File Map (Canvas Specific)

- src/pages/TemplateEditor.jsx
  Orchestrates editor state and passes callbacks.
- src/components/designer/DesignerCanvas.jsx
  Core canvas logic: initialization + update + sync.
- src/components/designer/ElementPalette.jsx
  Adds new elements with defaults.
- src/utils/fieldTransformer.js
  pdfme ↔ backend conversion.
- src/utils/fonts.js
  ———

Lifecycle: Add Text Field → Edit → Save → Preview → Publish

1. Add text field (User action)
   - UI path A (built‑in pdfme sidebar):
     The pdfme Designer renders its own left sidebar and handles adding elements.
     File: src/components/designer/DesignerCanvas.jsx
     It injects CSS for .pdfme-designer sidebar, so the built‑in UI is used.
   - UI path B (programmatic):
     DesignerCanvas exposes addElement through its ref.
     File: src/components/designer/DesignerCanvas.jsx
     Function: addElement(elementType, properties)
     It inserts a new field into template.schemas[0] and calls designerRef.current.updateTemplate(...).
2. Canvas receives the new element
   - File: src/components/designer/DesignerCanvas.jsx
   - Function: designerRef.current.onChangeTemplate((newTemplate) => { ... })
   - What happens:
     - Preserves fieldGroups so pdfme doesn’t drop them.
     - Calls detectAndSyncFieldChanges(newTemplate) to sync with backend.
     - Calls onTemplateChange(newTemplate) to update editor state.
3. New field is detected and created in backend
   - File: src/components/designer/DesignerCanvas.jsx
   - Function: detectAndSyncFieldChanges(...) → handleNewField(...)
   - Transforms field format:
     pdfmeToApiField(...) in src/utils/fieldTransformer.js
     Uses getFieldTypeId(...) from src/utils/fieldTypeMapping.js
   - API call:
     templateFieldService.create(fieldData, pageId)
     File: src/api/services/templateFieldService.js
     Endpoint: POST /v1/template-fields (pageId passed as query param)
   - Post‑create:
     Backend ID is written into the pdfme schema field (fieldObj.id = backendFieldId) so later updates map correctly.
4. Edit the text field
   - User edits text/position/size inside pdfme Designer UI.
   - File: src/components/designer/DesignerCanvas.jsx
   - Function: handleFieldUpdate(fieldName, pdfmeField, pageIndex)
   - Debounced update: 500ms
   - API call:
     templateFieldService.update(fieldId, fieldData)
     File: src/api/services/templateFieldService.js
     Endpoint: PATCH /v1/template-fields/:fieldId
5. Save template (schema)
   - File: src/pages/TemplateEditor.jsx
   - Function: handleSave(comment, operation)
   - Important current behavior:
     The code that saves schema to backend is commented out:

     // dispatch(saveVersionSchema({ versionId, schema: currentTemplate }))
     So “Save” currently only posts a comment and shows toast, but does not store schema unless field‑level sync is
     enabled (it is).

   - Comment API call:
     templateVersionService.addComment(templateId, versionId, commentData)
     File: src/api/services/templateVersionService.js
     Endpoint: POST /v1/templates/:templateId/versions/:versionId/comment

6. Preview the template
   - File: src/pages/TemplateEditor.jsx
   - Function: handleViewModeChange()
   - Uses:
     getInputFromTemplate(currentTemplate) from @baylor/pdfme-common
   - Renders preview:
     src/components/designer/PreviewCanvas.jsx
     Uses pdfme Viewer to render the final PDF view.
7. Publish the version
   - User action: click Publish → AddCommentModal
   - File: src/components/modals/AddCommentModal.jsx
   - Publish call:
     publishTemplate(templateId, versionId) in TemplateEditor.jsx
     Calls templateVersionService.publish(...)
   - API endpoint:
     POST /v1/templates/:templateId/versions/:versionId/publish

———

Key Files Summary (for this lifecycle)

1. src/pages/TemplateEditor.jsx
   Orchestrates editor state, save, preview, publish.
2. src/components/designer/DesignerCanvas.jsx
   Canvas wrapper, adds fields, syncs to API.
3. src/utils/fieldTransformer.js
   Converts pdfme field → backend field format.
4. src/utils/fieldTypeMapping.js
   Maps pdfme type to backend UUID.
5. src/api/services/templateFieldService.js
   create, update, delete field APIs.
6. src/api/services/templateVersionService.js
   addComment, publish.
7. src/components/designer/PreviewCanvas.jsx
   Renders preview via pdfme Viewer.

———
Great — here is the same full lifecycle walkthrough for all element types (text, image, line, rectangle, ellipse, table,
datasource, date/time, checkbox/select/radio). I’ll show:

- How they are added
- How they’re stored in schema
- How they sync to backend
- How preview/publish treats them
- Which files + functions are involved

———

# Common Lifecycle (applies to all elements)

Add → Edit → Save → Preview → Publish

Shared files & functions

- Add: src/components/designer/DesignerCanvas.jsx → addElement(type, props)
- Change detection: DesignerCanvas.jsx → onChangeTemplate
- Sync create/update/delete: DesignerCanvas.jsx → detectAndSyncFieldChanges
- Transform to backend: src/utils/fieldTransformer.js
- Type mapping: src/utils/fieldTypeMapping.js
- API: src/api/services/templateFieldService.js
- Preview: src/components/designer/PreviewCanvas.jsx
- Publish: src/pages/TemplateEditor.jsx → templateVersionService.publish(...)

———

# 1) TEXT

Add

- DesignerCanvas.addElement('text', props)
- Default props:

  { type:'text', width:120, height:20, fontSize:12, fontColor:'#000', text:'Sample Text' }

Schema

schemas[page][fieldName] = {
type: 'text',
position: {x,y},
width, height,
fontSize, fontColor,
content or text
}

Backend

- Transformed via pdfmeToApiField
- fieldTypeId = text UUID in fieldTypeMapping.js

Preview

- Rendered via pdfme text plugin.

———

# 2) IMAGE / SIGNATURE

Add

- addElement('image', props)
- Signature is a custom image subtype:
  - type: 'signature'

Schema

{
type: 'image',
content: 'data:image/..' or empty,
width, height,
objectFit: 'contain'
}

Backend

- Stored with content.text for image data.

Preview

- Uses pdfme image plugin.

———

# 3) LINE

Add

- addElement('line', props)

Schema

{
type:'line',
width, height,
borderWidth, borderColor, color
}

Backend

- Same flow, stored in style.

Preview

- pdfme line plugin.

———

# 4) RECTANGLE

Add

- addElement('rectangle', props)

Schema

{
type:'rectangle',
width, height,
color, borderWidth, borderColor
}

Preview

- pdfme rectangle plugin.

———

# 5) ELLIPSE (CIRCLE)

Add

- addElement('ellipse', props)

Schema

{
type:'ellipse',
width, height,
color, borderWidth, borderColor
}

Preview

- pdfme ellipse plugin.

———

# 6) TABLE

Add

- Tables are special in editor:
  - TemplateEditor.jsx → handleAddTable(tableSchema)
  - It directly injects the table schema into template.schemas[0].

Schema

{
type: 'table',
content: JSON.stringify([...]),
head: [...],
showHead: true,
tableStyles, headStyles, bodyStyles
}

Backend

- pdfmeToApiField encodes content as JSON string.

Preview

- pdfme table plugin.

———

# 7) DATASOURCE FIELD

Add

- addElement('datasource', props)

Schema

{
type:'datasource',
entity:'Patient',
property:'Name',
datasourceField:'Patient.Name'
}

Preview

- PreviewCanvas has a custom datasource plugin:
  - Converts datasource into text before rendering.

———

# 8) DATE / TIME / DATETIME

Add

- addElement('date'), addElement('time'), addElement('dateTime')

Schema

{
type:'date',
format:'YYYY-MM-DD'
}

Backend

- Stored in rendererExtras.format.

Preview

- pdfme date, time, dateTime plugins.

———

# 9) CHECKBOX / SELECT / RADIO

Add

- addElement('checkbox'), addElement('select'), addElement('radioGroup')

Schema

{ type:'checkbox', ... }
{ type:'select', options:[...] }

Backend

- Mapped by field type UUID.

Preview

- pdfme plugin renders selection elements.

———

# Element Lifecycle: End‑to‑End Example (Text Field)

User clicks "Text Field"
→ DesignerCanvas.addElement('text', props)
→ schema updated (schemas[0][text_123] = {...})
→ Designer emits onChangeTemplate(newTemplate)
→ detectAndSyncFieldChanges sees new field
→ pdfmeToApiField + getFieldTypeId
→ POST /v1/template-fields
→ backend field id saved into schema (fieldObj.id)
→ user edits text content
→ detectAndSyncFieldChanges → handleFieldUpdate (debounced)
→ PATCH /v1/template-fields/:fieldId
→ user clicks Preview
→ PreviewCanvas uses pdfme Viewer to render
→ user clicks Publish
→ POST /v1/templates/:templateId/versions/:versionId/publish

Common wrapper (what templateFieldService.create sends)

{
"fieldTypeId": "uuid-from-fieldTypeMapping",
"name": "field_name",
"pageId": "page-uuid-1",
"zIndex": 1,
"position": {
"x": 50,
"y": 50,
"width": 120,
"height": 20
},
"style": {},
"content": {
"text": "",
"format": null
},
"required": false,
"readOnly": false,
"hide": false,
"conditions": null,
"variables": {},
"rendererExtras": {}
}

———

# 1) Text Field

{
"fieldTypeId": "a1b2c3d4-e5f6-4a7b-8c9d-0e1f2a3b4c5d",
"name": "text_1700000000000",
"pageId": "page-uuid-1",
"zIndex": 1,
"position": { "x": 50, "y": 50, "width": 120, "height": 20 },
"style": {
"fontSize": 12,
"fontColor": "#000000",
"fontName": "DIN Next LT Pro",
"alignment": "left",
"verticalAlignment": "top",
"lineHeight": 1,
"characterSpacing": 0
},
"content": {
"text": "Sample Text",
"format": null
},
"required": false,
"readOnly": false,
"hide": false,
"conditions": null,
"variables": {},
"rendererExtras": {
"rotate": 0
}
}

———

# 2) Image Field

{
"fieldTypeId": "b2c3d4e5-f6a7-4b8c-9d0e-1f2a3b4c5d6e",
"name": "image_1700000000000",
"pageId": "page-uuid-1",
"zIndex": 1,
"position": { "x": 50, "y": 50, "width": 100, "height": 100 },
"style": {},
"content": {
"text": "data:image/png;base64,iVBORw0KGgoAAA...",
"format": null
},
"required": false,
"readOnly": false,
"hide": false,
"conditions": null,
"variables": {},
"rendererExtras": {
"objectFit": "contain"
}
}

———

# 3) Line

{
"fieldTypeId": "d4e5f6a7-b8c9-4d0e-9f2a-3b4c5d6e7f8a",
"name": "line_1700000000000",
"pageId": "page-uuid-1",
"zIndex": 1,
"position": { "x": 50, "y": 50, "width": 100, "height": 2 },
"style": {
"borderWidth": 1,
"borderColor": "#000000",
"color": "#000000"
},
"content": {
"text": "",
"format": null
},
"required": false,
"readOnly": false,
"hide": false,
"conditions": null,
"variables": {},
"rendererExtras": {}
}

———

# 4) Rectangle

{
"fieldTypeId": "f6a7b8c9-d0e1-4f2a-bb4c-5d6e7f8a9b0c",
"name": "rectangle_1700000000000",
"pageId": "page-uuid-1",
"zIndex": 1,
"position": { "x": 50, "y": 50, "width": 100, "height": 60 },
"style": {
"color": "#ffffff",
"borderWidth": 1,
"borderColor": "#000000"
},
"content": {
"text": "",
"format": null
},
"required": false,
"readOnly": false,
"hide": false,
"conditions": null,
"variables": {},
"rendererExtras": {}
}

———

# 5) Ellipse (Circle)

{
"fieldTypeId": "e5f6a7b8-c9d0-4e1f-aa3b-4c5d6e7f8a9b",
"name": "ellipse_1700000000000",
"pageId": "page-uuid-1",
"zIndex": 1,
"position": { "x": 50, "y": 50, "width": 100, "height": 100 },
"style": {
"color": "#ffffff",
"borderWidth": 1,
"borderColor": "#000000"
},
"content": {
"text": "",
"format": null
},
"required": false,
"readOnly": false,
"hide": false,
"conditions": null,
"variables": {},
"rendererExtras": {}
}

———

# 6) Table

{
"fieldTypeId": "c3d4e5f6-a7b8-4c9d-8e1f-2a3b4c5d6e7f",
"name": "table_auto_1700000000000",
"pageId": "page-uuid-1",
"zIndex": 1,
"position": { "x": 15, "y": 100, "width": 180, "height": 80 },
"style": {},
"content": {
"text": "{\"content\":[[\"${{Table.[INDEX].Col1}}\",\"${{Table.[INDEX].Col2}}\"]],\"head\":[\"Col1\",\"Col2\"],
\"showHead\":true}",
"format": "table"
},
"required": false,
"readOnly": false,
"hide": false,
"conditions": null,
"variables": {},
"rendererExtras": {
"tableStyles": { "borderWidth": 0.2, "borderColor": "#CCCCCC" },
"headStyles": { "fontSize": 8, "backgroundColor": "#E8F4FD" },
"bodyStyles": { "fontSize": 8 },
"headWidthPercentages": [50, 50]
}
}

———

# 7) Datasource Field

{
"fieldTypeId": "f2a3b4c5-d6e7-4f8a-9b0c-1d2e3f4a5b6c",
"name": "datasource_1700000000000",
"pageId": "page-uuid-1",
"zIndex": 1,
"position": { "x": 50, "y": 50, "width": 100, "height": 16 },
"style": {
"fontSize": 10,
"fontColor": "#000000"
},
"content": {
"text": "",
"format": null
},
"required": false,
"readOnly": false,
"hide": false,
"conditions": null,
"variables": {},
"rendererExtras": {}
}

———

# 8) Date / Time / DateTime

{
"fieldTypeId": "e7f8a9b0-c1d2-4e3f-af5b-6c7d8e9f0a1b",
"name": "date_1700000000000",
"pageId": "page-uuid-1",
"zIndex": 1,
"position": { "x": 50, "y": 50, "width": 100, "height": 16 },
"style": {
"fontSize": 10,
"fontColor": "#000000"
},
"content": {
"text": "",
"format": null
},
"required": false,
"readOnly": false,
"hide": false,
"conditions": null,
"variables": {},
"rendererExtras": {
"format": "YYYY-MM-DD",
"locale": "en"
}
}

———

# 9) Checkbox / Select / RadioGroup

{
"fieldTypeId": "b4c5d6e7-f8a9-4b0c-9d2e-3f4a5b6c7d8e",
"name": "checkbox_1700000000000",
"pageId": "page-uuid-1",
"zIndex": 1,
"position": { "x": 50, "y": 50, "width": 16, "height": 16 },
"style": {},
"content": { "text": "", "format": null },
"required": false,
"readOnly": false,
"hide": false,
"conditions": null,
"variables": {},
"rendererExtras": {}
}

———
