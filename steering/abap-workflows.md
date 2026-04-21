# ABAP Development Workflows

Common workflows for ABAP development using the ADT MCP server. All development happens remotely on the SAP system — the MCP server is the bridge. To give the IDE full context (syntax highlighting, linting, navigation, diffs), we maintain a **local mirror** of ABAP objects in ABAPGit file format.

---

## ⚠️ Development and Test Systems Only

**This power and its MCP server should only be connected to development (DEV) and test/QA (TST/QAS) systems — never to production.**

The MCP server has full read and write access to ABAP objects, including creating, updating, deleting, and activating code. Running these operations against a production system risks:

- Unreviewed code changes going live immediately
- Accidental deletion or modification of critical objects
- Transport conflicts and data integrity issues
- Bypassing your organization's change management and approval processes

**Recommended landscape setup:**

| System | MCP Connected? | Purpose |
|--------|---------------|---------|
| **DEV** (Development) | ✅ Yes | Active development — create, update, test objects here |
| **TST / QAS** (Test / Quality) | ✅ Yes | Validation — run Clean Core checks, integration testing |
| **PRE / STG** (Pre-production) | ❌ No | Transport-based deployment only |
| **PRD** (Production) | ❌ No | Transport-based deployment only |

Code should reach production exclusively through the SAP transport system (or CI/CD pipelines triggering transports), not through direct MCP writes.

You can connect multiple DEV/TST systems simultaneously — each gets its own MCP server entry and its own local mirror folder (e.g., `DEV/`, `QAS/`).

---

## Local Mirror: Keeping the IDE in Context

### The Concept

When working with ABAP via an MCP server, the SAP system is the source of truth. But the IDE (Kiro) works best when it can see files on disk. The local mirror solves this:

- Every time you read or create an ABAP object via the MCP server, save the source code locally in ABAPGit file format
- The local folder is named after the SAP system (e.g., `BRE/`, `S4H/`, `A4H/`)
- Sub-folders mirror the SAP package hierarchy
- Files use ABAPGit naming conventions (see the **abapgit** steering file for details)
- ABAPGit repositories that have been cloned can be added alongside or merged into the mirror
- The mirror gives ABAPLint, syntax highlighting, and Kiro's context engine full visibility into your ABAP codebase

### Folder Structure

```
<workspace>/
├── BRE/                              ← SAP system "BRE" (local mirror)
│   └── src/
│       ├── zmy_package/
│       │   ├── package.devc.xml
│       │   ├── zcl_my_class.clas.abap
│       │   ├── zcl_my_class.clas.xml
│       │   ├── zcl_my_class.clas.testclasses.abap
│       │   ├── zi_my_entity.ddls.asddls
│       │   ├── zi_my_entity.ddls.xml
│       │   ├── ztmy_table.tabl.xml
│       │   └── subpackage/
│       │       ├── package.devc.xml
│       │       └── ...
│       └── zanother_package/
│           └── ...
├── S4H/                              ← SAP system "S4H" (another mirror)
│   └── src/
│       └── ...
├── my-abapgit-project/               ← Cloned ABAPGit repo (can coexist)
│   ├── .abapgit.xml
│   └── src/
│       └── ...
└── powers/
    └── sap-abap-base/
        └── ...
```

### File Naming Quick Reference

When saving locally, use these ABAPGit conventions:

| Object Type | Files to Save |
|-------------|---------------|
| Class | `zcl_example.clas.abap` (source), `zcl_example.clas.xml` (metadata), optionally `.clas.locals_def.abap`, `.clas.locals_imp.abap`, `.clas.testclasses.abap` |
| Program | `z_report.prog.abap` (source), `z_report.prog.xml` (metadata) |
| Interface | `zif_example.intf.abap` (source), `zif_example.intf.xml` (metadata) |
| CDS View | `zi_view.ddls.asddls` (DDL source), `zi_view.ddls.xml` (metadata) |
| Table | `ztmy_table.tabl.xml` (full definition in XML) |
| Domain | `zd_domain.doma.xml` |
| Data Element | `zde_element.dtel.xml` |
| Behavior Def | `zi_entity.bdef.asbdef` (source), `zi_entity.bdef.xml` (metadata) |
| Metadata Ext | `zc_entity.ddlx.asddlxs` (source), `zc_entity.ddlx.xml` (metadata) |
| Service Def | `zsd_service.srvd.srvdsrv` (source), `zsd_service.srvd.xml` (metadata) |
| Service Binding | `zui_binding.srvb.xml` |
| Function Group | `z_fg.fugr.xml` (metadata), `z_fg.fugr.z_fm_name.abap` (per FM) |

All filenames are **lowercase**. See the **abapgit** steering file for the complete reference.

### Automatic Mirror Sync (Hook)

This power includes an **ABAP Mirror Sync** hook (`abap-mirror-sync`) that fires automatically after any MCP write operation — Create, Update, Delete, or Activate. The hook reminds the agent to:

- Re-read the affected object from SAP to get the latest activated source
- Save (or remove) the corresponding local files in ABAPGit format
- Handle class sub-includes (test classes, local definitions, local types)

This means you don't need to manually remember to sync — every time an object changes on the SAP system, the local mirror is updated automatically.

The hook matches any MCP tool whose name contains `Create`, `Update`, `Delete`, or `Activate` (e.g., `CreateClass`, `UpdateView`, `DeleteTable`, `ActivateObjects`).

> To disable the hook, remove or edit `.kiro/hooks/abap-mirror-sync.kiro.json` in your workspace.

### System Information File

When first connecting to a SAP system, create a `system-info.md` file in the mirror root. This file captures the backend release and is used by the agent to determine which ABAP language features are available (see the **abap-language** steering file for the version compatibility matrix).

**How to generate it:**

Query the `CVERS` table via the MCP server:

```
GetSqlQuery: sql_query = "SELECT COMPONENT, RELEASE, EXTRELEASE FROM CVERS WHERE COMPONENT IN ('SAP_BASIS', 'SAP_ABA', 'SAP_GWFND', 'SAP_UI', 'S4CORE')"
```

Then save the result as `<system>/system-info.md`:

```markdown
# System Information: BRE

| Component | Release | SP/Patch |
|-----------|---------|----------|
| SAP_BASIS | 750 | 0029 |
| SAP_ABA | 750 | 0029 |
| SAP_GWFND | 750 | 0029 |
| SAP_UI | 754 | 0015 |

## ABAP Version Summary

- **SAP_BASIS Release:** 750 SP29
- **ABAP Syntax Version:** v750
- **System Type:** on-premise / BTP Cloud (fill in as appropriate)

## Available ABAP Features

Based on SAP_BASIS 750, the following features are available:
- ✅ Inline declarations, constructor expressions, table expressions (7.40+)
- ✅ REDUCE, FILTER, GROUP BY (7.40 SP08+)
- ✅ FINAL(...), host expressions, UNION, IS INSTANCE OF (7.50+)
- ❌ Common Table Expressions / WITH (requires 7.51+)
- ❌ Enumerated types (requires 7.51+)
- ❌ Internal tables as data source FROM @itab (requires 7.52+)
- ❌ utclong type (requires 7.54+)
```

**Why this matters:**

- The agent reads `system-info.md` before writing ABAP code and avoids syntax that won't compile on the target system
- ABAPLint can be configured with the matching `syntax.version` in `abaplint.json` (e.g., `"version": "v750"`)
- When the **abap-language** steering file is loaded, the agent cross-references the version compatibility matrix against the system release to determine what's safe to use

**When to regenerate:** After a system upgrade or SP application — re-run the CVERS query and update the file.

---

## Workflow: Read and Mirror

The most common workflow — read an object from SAP and save it locally.

### Read a Class and Save Locally

1. **Read from SAP:**
   ```
   ReadClass: class_name = "ZCL_MY_CLASS"
   ```
   Returns full source code (definition + implementation), package, responsible user, and description.

2. **Save locally** as ABAPGit files:
   - `<system>/src/<package>/zcl_my_class.clas.abap` — the main source code
   - `<system>/src/<package>/zcl_my_class.clas.xml` — metadata (description, visibility, etc.)

3. **If the class has local types or test classes**, also read and save them:
   ```
   GetLocalTestClass: class_name = "ZCL_MY_CLASS"
   GetLocalDefinitions: class_name = "ZCL_MY_CLASS"
   GetLocalTypes: class_name = "ZCL_MY_CLASS"
   ```
   Save as:
   - `zcl_my_class.clas.testclasses.abap`
   - `zcl_my_class.clas.locals_def.abap`
   - `zcl_my_class.clas.locals_imp.abap`

### Read a CDS View and Save Locally

```
ReadView: view_name = "ZI_MY_VIEW"
```
Save as:
- `<system>/src/<package>/zi_my_view.ddls.asddls` — the DDL source
- `<system>/src/<package>/zi_my_view.ddls.xml` — metadata

### Read a Program and Save Locally

```
ReadProgram: program_name = "Z_MY_REPORT"
```
Save as:
- `<system>/src/<package>/z_my_report.prog.abap`
- `<system>/src/<package>/z_my_report.prog.xml`

For large programs with includes:
```
GetProgFullCode: name = "Z_MY_REPORT", type = "PROG/P"
```
Returns all includes in tree traversal order — save each include as a separate `.prog.abap` file.

### Mirror an Entire Package

To mirror a full package and its contents:

1. **Get the package tree:**
   ```
   GetPackageTree: package_name = "ZMY_PACKAGE"
   ```

2. **For each object in the tree**, read and save locally using the appropriate Read tool.

3. **Create the folder structure** matching the package hierarchy.

This gives the IDE complete visibility into the package for navigation, linting, and context.

---

## Workflow: Create Objects (Remote + Local Mirror)

When creating new objects, work remotely via MCP and save locally in parallel.

### Typical Creation Flow

1. **Create** the object on SAP (initial/empty state)
2. **Update** with source code or DDL
3. **Activate** the object
4. **Save** the source locally in ABAPGit format

### Create a Class

```
1. CreateClass: class_name = "ZCL_NEW_CLASS", package_name = "ZMY_PKG", description = "My new class"
2. UpdateClass: class_name = "ZCL_NEW_CLASS", source_code = "<full ABAP source>", activate = true
3. Save locally:
   - <system>/src/zmy_pkg/zcl_new_class.clas.abap
   - <system>/src/zmy_pkg/zcl_new_class.clas.xml
```

### Create a CDS View

```
1. CreateView: view_name = "ZI_NEW_VIEW", package_name = "ZMY_PKG"
2. UpdateView: view_name = "ZI_NEW_VIEW", ddl_source = "<full DDL source>", activate = true
3. Save locally:
   - <system>/src/zmy_pkg/zi_new_view.ddls.asddls
   - <system>/src/zmy_pkg/zi_new_view.ddls.xml
```

### Create a Table

```
1. CreateTable: table_name = "ZTMY_NEW_TABLE", package_name = "ZMY_PKG"
2. UpdateTable: table_name = "ZTMY_NEW_TABLE", ddl_code = "<full DDL>", activate = true
3. Save locally:
   - <system>/src/zmy_pkg/ztmy_new_table.tabl.xml
```

### Create a Domain → Data Element → Table (Chain)

When creating related DDIC objects, create and activate in dependency order:

```
1. CreateDomain: domain_name = "ZD_STATUS", datatype = "CHAR", length = 1,
   fixed_values = [{"low": "A", "text": "Active"}, {"low": "I", "text": "Inactive"}]

2. CreateDataElement: data_element_name = "ZDE_STATUS", type_kind = "domain",
   type_name = "ZD_STATUS", short_label = "Status", medium_label = "Status"

3. CreateTable + UpdateTable with fields referencing ZDE_STATUS

4. Save all locally in <system>/src/<package>/
```

### Batch Activation

When creating multiple related objects, use batch activation:
```
ActivateObjects: objects = [
  {"name": "ZD_STATUS", "type": "DOMA"},
  {"name": "ZDE_STATUS", "type": "DTEL"},
  {"name": "ZTMY_TABLE", "type": "TABL"}
]
```

---

## Workflow: Edit from Local Files

When you have a local mirror, you can edit files in the IDE and push changes to SAP.

### Edit a Class Locally, Then Deploy

1. **Edit** `<system>/src/zmy_pkg/zcl_my_class.clas.abap` in the IDE (with ABAPLint feedback)
2. **Read the updated file content**
3. **Push to SAP:**
   ```
   UpdateClass: class_name = "ZCL_MY_CLASS", source_code = "<content from local file>", activate = true
   ```
4. The local file is already up to date

### Edit a CDS View Locally, Then Deploy

1. **Edit** `<system>/src/zmy_pkg/zi_my_view.ddls.asddls`
2. **Push to SAP:**
   ```
   UpdateView: view_name = "ZI_MY_VIEW", ddl_source = "<content from local file>", activate = true
   ```

### Deploy via ABAPGit (Alternative)

If the SAP system has ABAPGit installed (standalone report or Eclipse plugin), you can also deploy by:

1. **Edit files locally** in the mirror or ABAPGit repo
2. **Commit and push** to a Git remote (GitHub, GitLab, etc.)
3. **In SAP**, open ABAPGit and **pull** from the Git repository
4. ABAPGit deserializes the files back into ABAP objects and activates them

This is useful for:
- Batch deployments of many objects at once
- Code review workflows (pull requests before deploying)
- CI/CD pipelines that trigger ABAPGit pulls
- Deploying to systems where you don't have direct MCP access

---

## Integrating Cloned ABAPGit Repositories

If you've already cloned an ABAPGit repository (e.g., an open-source ABAP project), it can coexist with your system mirrors:

```
<workspace>/
├── BRE/                          ← System mirror
│   └── src/
├── my-abapgit-project/           ← Cloned ABAPGit repo
│   ├── .abapgit.xml
│   ├── abaplint.json
│   └── src/
│       └── zmy_pkg/
│           ├── zcl_example.clas.abap
│           └── ...
```

- The cloned repo has its own `.abapgit.xml` and potentially its own `abaplint.json`
- You can reference files from the cloned repo when working with the MCP server
- To deploy a cloned repo to your SAP system, either:
  - Use ABAPGit in SAP to pull from the Git remote
  - Use the MCP server to create/update objects one by one from the local files

---

## Searching and Navigating

### Search for Objects
```
SearchObject: object_name = "ZMM_*"
```
Supports wildcards. Optionally filter by type: `object_type = "CLAS/OC"` for classes only.

### Browse a Package
```
GetPackageTree: package_name = "ZMY_PACKAGE"
```
Returns the full hierarchical tree including subpackages and all objects with descriptions.

### Where-Used Analysis
```
GetWhereUsed: object_name = "ZCL_MY_CLASS", object_type = "class"
```
Finds all objects that reference or depend on the given object.

### Query Table Data
```
GetSqlQuery: sql_query = "SELECT * FROM ztmy_table WHERE status = 'A'", row_number = 50
```
Executes read-only SQL against tables and CDS views.

---

## RAP Development

### Create a Basic RAP Business Object

A typical RAP stack consists of:
1. Database table
2. Interface CDS view (I_ view)
3. Consumption CDS view (C_ view)
4. Behavior definition
5. Behavior implementation
6. Service definition
7. Service binding
8. Metadata extension (for Fiori UI)

### Step-by-Step RAP Creation

```
1. Create table:
   CreateTable + UpdateTable for ZTMY_ENTITY

2. Create interface view:
   CreateView + UpdateView for ZI_MY_ENTITY
   (SELECT from ZTMY_ENTITY with proper annotations)

3. Create consumption view:
   CreateView + UpdateView for ZC_MY_ENTITY
   (SELECT from ZI_MY_ENTITY with UI annotations)

4. Create behavior definition:
   CreateBehaviorDefinition: name = "ZI_MY_ENTITY",
   root_entity = "ZI_MY_ENTITY", implementation_type = "Managed"

5. Update behavior definition source:
   UpdateBehaviorDefinition with CRUD operations, validations, etc.

6. Create behavior implementation:
   CreateBehaviorImplementation: class_name = "ZBP_MY_ENTITY",
   behavior_definition = "ZI_MY_ENTITY"

7. Create service definition:
   CreateServiceDefinition: service_definition_name = "ZSD_MY_ENTITY",
   source_code = "define service ZSD_MY_ENTITY { expose ZC_MY_ENTITY; }"

8. Create service binding:
   CreateServiceBinding: service_binding_name = "ZUI_MY_ENTITY_O4",
   service_definition_name = "ZSD_MY_ENTITY", binding_type = "ODataV4"

9. Publish service binding:
   UpdateServiceBinding: desired_publication_state = "published"

10. Create metadata extension:
    CreateMetadataExtension + UpdateMetadataExtension for ZC_MY_ENTITY
    (Fiori UI annotations: list report, object page layout)
```

**After creating the full RAP stack, save all objects locally:**

```
<system>/src/zmy_pkg/
├── ztmy_entity.tabl.xml
├── zi_my_entity.ddls.asddls
├── zi_my_entity.ddls.xml
├── zc_my_entity.ddls.asddls
├── zc_my_entity.ddls.xml
├── zi_my_entity.bdef.asbdef
├── zi_my_entity.bdef.xml
├── zbp_my_entity.clas.abap
├── zbp_my_entity.clas.locals_imp.abap
├── zbp_my_entity.clas.xml
├── zsd_my_entity.srvd.srvdsrv
├── zsd_my_entity.srvd.xml
├── zui_my_entity_o4.srvb.xml
├── zc_my_entity.ddlx.asddlxs
└── zc_my_entity.ddlx.xml
```

---

## Transport Management

### List Open Transports
```
ListTransports (defaults to current user, modifiable only)
```

### Create a New Transport
```
CreateTransport: description = "ZMY_PKG: New feature implementation"
```

### Inspect a Transport
```
GetTransport: transport_number = "E19K905635"
```
Returns metadata, included objects, tasks, and status.

---

## Unit Testing

### Run ABAP Unit Tests
```
1. RunUnitTest: tests = [{"container_class": "ZCL_MY_CLASS", "test_class": "LTCL_MY_TEST"}]
2. GetUnitTestStatus: run_id = "<returned run_id>" (with long polling)
3. GetUnitTestResult: run_id = "<returned run_id>"
```

### Run CDS Unit Tests
```
1. CreateCdsUnitTest: class_name = "ZCL_CDS_TEST", package_name = "ZMY_PKG",
   cds_view_name = "ZI_MY_VIEW"
2. UpdateCdsUnitTest: class_name = "ZCL_CDS_TEST", test_class_source = "<test code>"
3. Run and check results using GetCdsUnitTest
```

### Read Local Test Classes
```
GetLocalTestClass: class_name = "ZCL_MY_CLASS"
```

### Update Local Test Classes
```
UpdateLocalTestClass: class_name = "ZCL_MY_CLASS",
test_class_code = "<updated test source>"
```

Save test classes locally as `zcl_my_class.clas.testclasses.abap` after reading or updating.

---

## Profiling and Runtime Analysis

### Profile a Class
```
1. RuntimeRunClassWithProfiling: class_name = "ZCL_MY_CLASS",
   description = "Performance test"
2. RuntimeAnalyzeProfilerTrace: trace_id_or_uri = "<returned trace_id>",
   view = "hitlist", top = 20
```

### Profile a Program
```
1. RuntimeRunProgramWithProfiling: program_name = "Z_MY_REPORT"
2. RuntimeAnalyzeProfilerTrace with the returned trace_id
```

### Analyze Database Accesses
```
RuntimeGetProfilerTraceData: trace_id_or_uri = "<trace_id>",
view = "db_accesses"
```

### Check Runtime Dumps
```
1. RuntimeListDumps: user = "MYUSER", top = 10
2. RuntimeGetDumpById: dump_id = "<dump_id>", response_mode = "both"
```

### Check Gateway Errors
```
RuntimeGetGatewayErrorLog: max_results = 20
```

---

## Syntax Checking

Run syntax checks before activating to catch errors early:

```
CheckClass: class_name = "ZCL_MY_CLASS"
CheckView: view_name = "ZI_MY_VIEW"
CheckTable: table_name = "ZTMY_TABLE"
CheckProgram: program_name = "Z_MY_REPORT"
CheckBehaviorDefinition: name = "ZI_MY_ENTITY"
CheckMetadataExtension: name = "ZC_MY_ENTITY"
```

You can also validate hypothetical code without saving:
```
CheckClass: class_name = "ZCL_MY_CLASS", source_code = "<code to validate>"
CheckView: view_name = "ZI_MY_VIEW", ddl_source = "<DDL to validate>"
```

---

## Enhancement Discovery

### Find Enhancements for an Object
```
GetEnhancements: object_name = "SAPMV45A", object_type = "PROG"
```

### Inspect an Enhancement Spot
```
GetEnhancementSpot: enhancement_spot = "ES_SAPMV45A"
```

### Read Enhancement Implementation
```
GetEnhancementImpl: enhancement_spot = "ES_SAPMV45A",
enhancement_name = "ZE_MY_ENHANCEMENT"
```
