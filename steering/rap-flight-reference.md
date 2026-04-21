# RAP Flight Reference Scenarios

The SAP ABAP Flight Reference Scenario is the official best-practice reference for RAP (RESTful Application Programming Model) development. It provides complete, production-quality implementations of every major RAP pattern — from read-only reporting to managed transactional apps with draft, extensibility, and tree views.

**Repository:** [SAP-samples/abap-platform-refscen-flight](https://github.com/SAP-samples/abap-platform-refscen-flight)

> **This power is pinned to branch `ABAP-platform-2025` (ABAP 8.16).**
> - **Browsable repo at this branch:** <https://github.com/SAP-samples/abap-platform-refscen-flight/tree/ABAP-platform-2025>
> - **Raw fetch base URL** (prepend to any `src/…` path): `https://raw.githubusercontent.com/SAP-samples/abap-platform-refscen-flight/ABAP-platform-2025/`
>
> Every fetch URL below already targets this branch. If the target system moves to a different SAP_BASIS release, regenerate the power to refresh this file.

This steering file is a **lookup index** — it does not contain the full source code. When you need a reference implementation, use the fetch URLs below to retrieve exactly the files you need. This keeps token consumption minimal while giving you access to the complete reference.

---

## Scenario Index

### Read-Only Scenarios

| Scenario | Package | Description | Use When |
|----------|---------|-------------|----------|
| **Read-Only List Report** | `/DMO/FLIGHT_READONLY` | Basic list reporting with CDS views and service binding | Building read-only Fiori list reports, analytical queries |
| **Analytical App** | `/DMO/FLIGHT_ANA` | Analytical CDS views with aggregations | Building analytical dashboards, KPI reports |

### Transactional Scenarios

| Scenario | Package | Description | Use When |
|----------|---------|-------------|----------|
| **Unmanaged** | `/DMO/FLIGHT_UNMANAGED` | Full custom CRUD logic, you control everything | Wrapping legacy code, complex business logic, non-standard persistence |
| **Managed** | `/DMO/FLIGHT_MANAGED` | Framework handles CRUD, you add validations/determinations | Standard CRUD apps, greenfield development, most common pattern |
| **Draft (Exclusive)** | `/DMO/FLIGHT_DRAFT` | Managed + exclusive draft for edit sessions | Apps needing save-as-draft, long-running edits, single-user editing |
| **Draft (Collaborative)** | `/DMO/FLIGHT_COLLDRAFT` | Managed + collaborative draft for multi-user editing | Multi-user editing scenarios, concurrent access |
| **Cross-BO** | `/DMO/FLIGHT_XBO` | Cross-business-object interactions | Scenarios involving multiple related business objects |

### Multi-Inline-Edit & Reuse

| Scenario | Package | Description | Use When |
|----------|---------|-------------|----------|
| **Multi-Inline-Edit** | `/DMO/FLIGHT_REUSE_SUPPLEMENT` | Inline editing of multiple rows in a table | Master data maintenance, bulk editing |
| **Carrier Maintenance** | `/DMO/FLIGHT_REUSE_CARRIER` | Multi-inline-edit for carrier data | Reference for reusable BO patterns |

### Tree Views

| Scenario | Package | Description | Use When |
|----------|---------|-------------|----------|
| **Read-Only Hierarchy** | `/DMO/FLIGHT_HIERARCHY_RO` | Hierarchical tree display (read-only) | Org charts, category trees, BOM displays |
| **Editable Hierarchy + Draft** | `/DMO/FLIGHT_HIERARCHY_DRAFT` | Editable tree with draft capabilities | Editable org structures, configurable hierarchies |

### RAP Extensibility

| Scenario | Package | Description | Use When |
|----------|---------|-------------|----------|
| **Data Model Extension** | `/DMO/FLIGHT_REUSE_AGENCY_SLOGN` | Extending a BO's data model with new fields | Adding custom fields to a released BO |
| **Behavior Extension** | `/DMO/FLIGHT_REUSE_AGENCY_CNTRY` | Extending a BO's behavior with new validations/actions | Adding custom logic to a released BO |
| **Node Extension** | `/DMO/FLIGHT_REUSE_AGENCY_REV` | Adding new child nodes to a BO | Extending a BO with new sub-entities |

---

## Key Objects per Scenario

When you need a reference implementation, these are the key files to fetch for each scenario. The file paths follow ABAPGit conventions under the `src/` directory.

### Managed Scenario (Most Common Starting Point)

This is the recommended starting pattern for most new RAP applications.

| Object | Type | Purpose |
|--------|------|---------|
| `/DMO/I_TRAVEL_M` | CDS View (DDLS) | Root interface view |
| `/DMO/I_BOOKING_M` | CDS View (DDLS) | Child interface view |
| `/DMO/I_BOOKINGSUPPLEMENT_M` | CDS View (DDLS) | Grandchild interface view |
| `/DMO/C_TRAVEL_M` | CDS View (DDLS) | Root consumption/projection view |
| `/DMO/C_BOOKING_M` | CDS View (DDLS) | Child consumption view |
| `/DMO/I_TRAVEL_M` | BDEF | Behavior definition (managed, with determinations/validations) |
| `/DMO/BP_TRAVEL_M` | Class (CLAS) | Behavior implementation (handler + saver) |
| `/DMO/UI_TRAVEL_M` | SRVD | Service definition |
| `/DMO/UI_TRAVEL_M_O4` | SRVB | Service binding (OData V4) |

### Draft Scenario

Extends the managed pattern with draft capabilities:

| Object | Type | Purpose |
|--------|------|---------|
| `/DMO/I_TRAVEL_D` | CDS View (DDLS) | Root interface view |
| `/DMO/C_TRAVEL_D` | CDS View (DDLS) | Root consumption view |
| `/DMO/I_TRAVEL_D` | BDEF | Behavior definition (managed + draft) |
| `/DMO/BP_TRAVEL_D` | Class (CLAS) | Behavior implementation |
| `/DMO/TRAVEL_D` | Table (TABL) | Active persistence table |
| `/DMO/TRAVELD_D` | Table (TABL) | Draft table |

### Unmanaged Scenario

Full custom implementation — useful as reference for wrapping legacy code:

| Object | Type | Purpose |
|--------|------|---------|
| `/DMO/I_TRAVEL_U` | CDS View (DDLS) | Root interface view |
| `/DMO/C_TRAVEL_U` | CDS View (DDLS) | Root consumption view |
| `/DMO/I_TRAVEL_U` | BDEF | Behavior definition (unmanaged) |
| `/DMO/BP_TRAVEL_U` | Class (CLAS) | Behavior implementation (full CRUD) |

---

## Fetching Reference Code

### Strategy: Fetch Only What You Need

When building a RAP application, don't fetch the entire repository. Instead:

1. **Identify the scenario** that matches your use case (see Scenario Index above)
2. **Determine the branch** from `system-info.md` (see Version Branch Mapping)
3. **Fetch only the specific files** you need as reference

### Fetch via Web (for reading reference code)

To fetch a specific file from the flight reference, construct the URL from the pinned base:

```
https://raw.githubusercontent.com/SAP-samples/abap-platform-refscen-flight/ABAP-platform-2025/src/{package_path}/{filename}
```

**Example — fetch the managed travel BDEF:**
```
https://raw.githubusercontent.com/SAP-samples/abap-platform-refscen-flight/ABAP-platform-2025/src/managed/%23dmo%23i_travel_m.bdef.asbdef
```

**Note:** The `/DMO/` namespace is encoded as `%23dmo%23` in URLs (the `#` becomes `%23`). In the file system, it appears as `#dmo#`.

### Fetch via MCP (if installed on the SAP system)

If the flight reference is already installed on your SAP system, you can read it directly:

```
ReadView: view_name = "/DMO/I_TRAVEL_M"
ReadBehaviorDefinition: behavior_definition_name = "/DMO/I_TRAVEL_M"
ReadClass: class_name = "/DMO/BP_TRAVEL_M"
```

This is the most efficient approach — no web fetch needed, and you get the exact version that matches your system.

### Check if Flight Reference is Installed

```
SearchObject: object_name = "/DMO/I_TRAVEL_M"
```

If it returns results, the flight reference is installed and you can read directly via MCP.

---

## Installing the Flight Reference on Your SAP System

### Prerequisites
- SAP_BASIS 7.54 or higher
- ABAPGit standalone report installed (`ZABAPGIT_STANDALONE`)
- `/DMO/` namespace configured (see namespace setup below)
- SSL certificates for github.com configured in STRUST

### Namespace Setup

The `/DMO/` namespace must be registered before importing:

1. Transaction `SE03` → Administration → Set System Change Option
2. Find `/DMO/` namespace → set to `Modifiable`
3. If namespace doesn't exist, register it via transaction `SE03`:
   - Namespace: `/DMO/`
   - Namespace Role: `C`
   - Repair License: `32869948212895959389`
   - Owner: `SAP`

### Import via ABAPGit

1. Create package `/DMO/FLIGHT` (software component: `HOME`)
2. Run `ZABAPGIT_STANDALONE` in SE38
3. Choose **New Online** → URL: `https://github.com/SAP/abap-platform-refscen-flight.git`
4. Package: `/DMO/FLIGHT`, Branch: select the branch matching your system version
5. **Pull** and activate all objects
6. Publish service bindings (open each → click **Publish**)
7. Generate sample data: run `/DMO/CL_FLIGHT_DATA_GENERATOR` as console app (F9)

---

## Scenario Selection Guide

Use this decision tree when starting a new RAP application:

```
Is it read-only (no create/update/delete)?
├── Yes → Read-Only List Report (/DMO/FLIGHT_READONLY)
│         Need analytics/aggregations? → Analytical (/DMO/FLIGHT_ANA)
│
└── No (transactional) →
    │
    Do you need to wrap existing legacy code?
    ├── Yes → Unmanaged (/DMO/FLIGHT_UNMANAGED)
    │
    └── No (greenfield) →
        │
        Do users need save-as-draft?
        ├── No → Managed (/DMO/FLIGHT_MANAGED)
        │
        └── Yes →
            │
            Do multiple users edit the same object simultaneously?
            ├── No → Draft Exclusive (/DMO/FLIGHT_DRAFT)
            └── Yes → Draft Collaborative (/DMO/FLIGHT_COLLDRAFT)

Additional patterns (combine with above):
- Need inline table editing? → Multi-Inline-Edit (/DMO/FLIGHT_REUSE_SUPPLEMENT)
- Need tree/hierarchy display? → Hierarchy RO or Draft
- Need extensibility? → Agency extensibility examples
```

---

## Data Model Overview

The flight reference uses a consistent domain model across all scenarios:

```
Travel (root)
├── Booking (child)
│   └── BookingSupplement (grandchild)
├── Agency (association)
├── Customer (association)
└── Carrier (association)
    └── Connection
        └── Flight
```

**Core tables:**
- `/DMO/TRAVEL` — Travel bookings (root entity)
- `/DMO/BOOKING` — Individual flight bookings
- `/DMO/BOOK_SUPPL` — Booking supplements (extras)
- `/DMO/AGENCY` — Travel agencies (master data)
- `/DMO/CUSTOMER` — Customers (master data)
- `/DMO/CARRIER` — Airlines (master data)
- `/DMO/CONNECTION` — Flight connections
- `/DMO/FLIGHT` — Scheduled flights

This model is intentionally simple but covers all RAP patterns: root/child/grandchild composition, associations, value helps, validations, determinations, actions, and draft.

---

## Source Attribution

Content derived from [SAP-samples/abap-platform-refscen-flight](https://github.com/SAP-samples/abap-platform-refscen-flight) (Apache 2.0 License). Content rephrased for compliance with licensing restrictions.
