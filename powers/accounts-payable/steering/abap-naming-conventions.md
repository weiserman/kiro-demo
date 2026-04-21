# ABAP Naming Conventions

This project uses the **`Z`** namespace for all custom ABAP
objects. Every new class, interface, CDS view entity, behavior definition,
service, table, message class, and ABAPGit repository you create must start
with this namespace.

Apply these names consistently — they are picked up by downstream tools
(ABAPLint rules, package validation hooks, Clean Core compliance checks)
that treat the namespace as authoritative.

## Namespace

- **Namespace prefix:** `Z`
- Registered at the tenant level in RAP Forge; change it centrally in
  tenant Settings rather than per-object.
- If the prefix is a reserved SAP namespace (e.g. `/DCB/`), all objects
  must be created inside that namespace's package hierarchy and follow
  its reservation rules.

## Development Package

All newly created ABAP objects in this project must be assigned to the
development package **`Z001`**.

- Classes, CDS views, behavior definitions, service definitions, tables,
  message classes — every new TADIR object goes into `Z001`
  unless the user explicitly picks a sub-package.
- When creating objects through the ABAP ADT MCP server, pass
  `Z001` as the `packageName` parameter.
- If a sub-package is needed (e.g. for test artefacts), create it *under*
  `Z001` rather than under `$TMP` or a foreign package.

## Object Naming Patterns

## RAP Object Naming Convention

### CDS Data Model (Root/Child Entities)
- Root Entity:        ZAP_R_[Object]
- Child Entity:       ZAP_R_[Object]_[Child]
- Draft Table:        ZAP_D_[Object]
- Database Table:     ZAP_T_[Object]

### CDS Projection (Consumption Layer)
- Projection:         ZAP_C_[Object]
- Child Projection:   ZAP_C_[Object]_[Child]
- Metadata Extension: ZAP_E_[Object]

### Behavior
- Behavior Def:       ZAP_R_[Object]  (same as root entity)
- Behavior Impl:      ZAPBP_R_[Object]

### Service
- Service Definition: ZAP_SD_[Object]
- Service Binding:    ZAP_SB_[Object]_[V4/V2]

### Value Helps & Abstract Entities
- Value Help:         ZAP_VH_[Domain]
- Abstract Entity:    ZAP_A_[Purpose]

## Rules

1. **No raw `Z*` / `Y*` objects** unless the namespace prefix is
   literally `Z` or `Y`. Use `Z` for every new object.
2. **Match the pattern above** — do not invent new role letters
   (`_X_`, `_Q_`, …). If a new kind of artifact appears, update the
   tenant's naming convention in Settings first, then regenerate.
3. **Keep `[Object]` short and semantic.** Avoid acronyms that only
   make sense to the current team. The object name reads alongside the
   namespace prefix, so something like `Z_R_Travel`
   is preferred over `Z_R_TRV01`.
4. **Draft tables (`_D_`) and database tables (`_T_`) share the root's
   object name.** They form a matched set — renaming one without the
   others breaks managed-RAP save sequencing.
5. **Service bindings carry a protocol suffix** (`_V4` for OData V4,
   `_V2` for V2). The binding name is what consumers see; the suffix
   signals the protocol contract.

## When Generating Code

When asked to create new RAP / ABAP artefacts, apply these conventions
automatically. Don't ask the user to confirm the namespace for each
object — it's fixed by tenant policy. If a generated name doesn't fit
a pattern above, pause and confirm with the user before proceeding.
