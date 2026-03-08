# HKLS Planner — Laravel Web Application Plan

## 1) Product Goal
Build a Laravel-based web application to plan HKLS projects with:
- **Projects** as top-level entities.
- **Apartments** within projects.
- **Rooms** within apartments.
- **Floor-heating calculations** per room/apartment/project.
- **Automatic material lists** generated from calculated heating layouts.

---

## 2) Scope and Functional Requirements

### 2.1 Core Entities
1. **Project**
   - Name, customer, address, design parameters, status, revision number.
2. **Apartment**
   - Belongs to a project, has floor level, orientation, usage type.
3. **Room**
   - Belongs to an apartment, has geometry/area, thermal targets, floor build-up.
4. **Floor Heating Circuit / Zone**
   - Calculated heating loops with spacing, loop length, flow/temp assumptions.
5. **Material Item**
   - Pipe length, manifolds, valves, insulation, clips, controls, accessories.

### 2.2 User Workflows
1. Create project → add apartments → add rooms.
2. Enter room thermal/heating parameters.
3. Run calculation engine for floor heating.
4. Review warnings (e.g., too-long loop, high heat loss).
5. Generate material list (project + apartment + room level).
6. Export reports (PDF/CSV).

### 2.3 Non-Functional Requirements
- Multi-user with role-based access.
- Audit trail for engineering changes.
- Deterministic calculation versioning.
- Good performance for projects with many apartments/rooms.

---

## 3) Proposed Tech Stack
- **Backend**: Laravel 11, PHP 8.3, Eloquent ORM.
- **Frontend**: Blade + Livewire (or Inertia + Vue if richer interactivity is needed).
- **DB**: PostgreSQL (recommended) or MySQL.
- **Queue/Jobs**: Redis + Laravel Queue for heavy calculations/exports.
- **Auth**: Laravel Breeze/Fortify + policies for authorization.
- **Reporting**: DomPDF/Snappy for PDFs, native CSV export.

---

## 4) Domain Model (Initial)

### 4.1 Tables
- `users`
- `projects`
- `project_revisions`
- `apartments`
- `rooms`
- `room_surfaces` (optional: detailed envelope areas)
- `heating_inputs` (normalized room input values)
- `heating_calculations` (result snapshots)
- `heating_circuits`
- `material_catalog`
- `material_lists`
- `material_list_items`
- `exports`
- `audit_logs`

### 4.2 Relationships
- Project `hasMany` Apartments.
- Apartment `hasMany` Rooms.
- Room `hasOne` HeatingInput.
- Room `hasMany` HeatingCircuits.
- Calculation snapshots tied to Room/Apartment/Project + revision.
- Material lists generated from calculations and optionally merged by project.

---

## 5) Calculation Engine Design

### 5.1 Input Parameters (per room)
- Room area, perimeter, usable heated area.
- Design indoor temperature.
- Heat demand / specific W/m² target.
- Floor covering resistance.
- Pipe type and diameter.
- Max loop length and spacing constraints.
- Supply/return temperature assumptions.

### 5.2 Output Parameters
- Required heat output.
- Recommended spacing.
- Pipe length estimate.
- Number of circuits and loop lengths.
- Flow rate estimates.
- Constraint violations/warnings.

### 5.3 Engineering & Maintainability Approach
- Implement calculations in a dedicated domain service layer (`App\Domain\Heating\...`).
- Use immutable DTOs for inputs/outputs.
- Version formulas using a `calculation_version` field.
- Cover formulas with unit tests and reference fixtures.

> Note: exact national standards and manufacturer-specific formulas should be confirmed with domain engineers and encoded as explicit, testable rules.

---

## 6) Material List Generation

### 6.1 Mapping Logic
- Pipe length from loop totals + waste factor.
- Manifolds by total number of circuits per apartment.
- Accessories based on selected system profile (catalog-driven).
- Optional substitutions from preferred vendors.

### 6.2 Output Formats
- On-screen grouped BOM (by project/apartment/category).
- CSV export for procurement.
- PDF report with assumptions + calculation summary.

---

## 7) API / UI Modules

### 7.1 Modules
1. **Project Management**
2. **Apartment & Room Editor**
3. **Heating Parameters Form**
4. **Calculation Results & Validation Warnings**
5. **Material List Builder**
6. **Export Center**
7. **Admin: Material Catalog + Rule Config**

### 7.2 Suggested Routes (Web)
- `/projects`
- `/projects/{project}`
- `/projects/{project}/apartments/{apartment}`
- `/rooms/{room}/heating`
- `/rooms/{room}/calculate`
- `/projects/{project}/materials`
- `/exports/{export}`

---

## 8) Security, Permissions, and Audit
- Roles: `admin`, `engineer`, `viewer`.
- Policies for project-level access.
- Immutable calculation snapshots for traceability.
- Audit log entries for parameter changes and recalculation events.

---

## 9) Quality Strategy

### 9.1 Tests
- Unit tests for formulas and material-mapping rules.
- Feature tests for project/apartment/room CRUD.
- End-to-end tests for “input → calculate → materials → export”.

### 9.2 Validation
- Strong server-side input validation using Form Requests.
- Domain validation warnings distinct from hard errors.

---

## 10) Delivery Plan (Phased)

### Phase 1 — Foundation (1–2 weeks)
- Bootstrap Laravel app, auth, roles, base layout.
- Implement Projects/Apartments/Rooms CRUD.
- Initial migrations and factories.

### Phase 2 — Heating Engine MVP (2–3 weeks)
- Input forms and domain services.
- Room-level calculation with persisted snapshots.
- Basic warning system.

### Phase 3 — Material Lists (1–2 weeks)
- Material catalog and mapping engine.
- Aggregated BOM generation and CSV export.

### Phase 4 — Reporting & Hardening (1–2 weeks)
- PDF reports.
- Audit logs and revision flow.
- Test coverage expansion and performance tuning.

---

## 11) Risks & Mitigations
- **Formula correctness risk** → lock requirements with HKLS engineer + golden test cases.
- **Data model churn** → introduce revisions and migration strategy early.
- **Performance in large projects** → queue heavy calculations and cache aggregates.

---

## 12) Definition of Done (MVP)
- User can build full project hierarchy (project/apartment/room).
- User can run floor-heating calculation per room.
- System generates project-level material list automatically.
- Exports (CSV at minimum) are available.
- Core workflows covered by automated tests.
