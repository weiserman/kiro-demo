---
inclusion: auto
name: abapgit
description: "ABAPGit overview, installation (standalone report and Eclipse ADT plugin), file naming conventions, and how to map local files to MCP server tools."
---

# ABAPGit: Version Control for ABAP

ABAPGit is the de facto open-source Git client for ABAP. It serializes ABAP development objects into files that can be stored in Git repositories, enabling version control, code review, and CI/CD workflows for ABAP development.

**Official docs:** [docs.abapgit.org](https://docs.abapgit.org)
**GitHub:** [github.com/abapGit/abapGit](https://github.com/abapGit/abapGit)

---

## Overview

ABAPGit converts ABAP repository objects (classes, programs, CDS views, tables, etc.) into a file-based representation that Git can track. Each ABAP object becomes one or more files on disk, organized by package into folders.

Key capabilities:
- Push ABAP objects from an SAP system to a Git repository
- Pull objects from a Git repository into an SAP system
- Track changes, diffs, and history for ABAP code
- Enable code review workflows via pull requests
- Support offline (ZIP) and online (Git URL) repository modes
- Works with GitHub, GitLab, Bitbucket, Azure DevOps, and any Git host

---

## Installation Options

ABAPGit can be used in two ways: as a standalone ABAP report inside SAP GUI, or as an Eclipse ADT plugin.

### Option 1: Standalone Report (SAP GUI)

The standalone version is a single ABAP report containing all ABAPGit code. It runs inside SAP GUI via transaction SE38.

**Installation steps:**

1. Download the latest ABAP source from [raw.githubusercontent.com/abapGit/build/main/zabapgit_standalone.prog.abap](https://raw.githubusercontent.com/abapGit/build/main/zabapgit_standalone.prog.abap) (right-click → Save As)

2. In your SAP system, create a new report via SE38, SE80, or ADT:
   - Report name: `ZABAPGIT_STANDALONE` (do **not** use `ZABAPGIT` — that name is reserved for the developer version)
   - Package: a local `$` package (e.g., `$ABAPGIT`)

3. In source code change mode, upload the downloaded file:
   - Utilities → More Utilities → Upload/Download → Upload

4. Activate the report

5. Run the report in SE38 to launch ABAPGit

**Prerequisites:**
- SAP BASIS 702 or higher
- SAP GUI for Windows (recommended) or SAP GUI for Java/HTML
- For online repositories: SSL must be configured (STRUST)

**When to use:** Quick setup, no Eclipse required, works on any SAP system with BASIS 702+.

### Option 2: Eclipse ADT Plugin

The ABAPGit Eclipse plugin integrates directly into ABAP Development Tools (ADT), providing a native Eclipse UI for ABAPGit operations.

**Installation steps:**

1. Open Eclipse with ADT installed

2. Go to **Help → Install New Software...**

3. Add the update site URL:
   ```
   https://eclipse.abapgit.org/updatesite/
   ```

4. Select **abapGit plugin for ABAP Development Tools (ADT)**

5. Click **Next**, accept the license, and finish the installation

6. Restart Eclipse

**After installation:**
- The ABAPGit Repositories view appears under **Window → Show View → Other → ABAP → abapGit Repositories**
- You can link ABAP packages to Git repositories directly from the Eclipse UI
- Stage, commit, push, and pull operations are available in the Eclipse interface

**Prerequisites:**
- Eclipse with ADT plugin installed (update site: `https://tools.hana.ondemand.com/latest`)
- Java 11 or 17 (bundled with latest Eclipse packages)
- SAP system connection configured in ADT

**When to use:** Preferred for daily development, integrates with ADT features (syntax highlighting, code navigation, transport management).

### Developer Version (for ABAPGit contributors)

If you want to contribute to ABAPGit itself:

1. Install the standalone version first
2. Run it and create a new online repository pointing to `https://github.com/abapGit/abapGit/`
3. Pull into package `$ABAPGIT`
4. Transaction `ZABAPGIT` becomes available

---

## File Naming Conventions

ABAPGit serializes each ABAP object into one or more files. Understanding the naming pattern is essential when working with ABAPGit repositories locally.

### General Pattern

```
<object_name>.<object_type>.<extension>
<object_name>.<object_type>.<extra>.<extension>
<object_name>.<object_type>.i18n.<language>.<extension>
```

- Filenames are **lowercase**
- Object metadata is stored in `.xml` (classic format) or `.json` (ABAP File Format / AFF)
- Source code is stored in `.abap` files
- Translations are stored in `.po` or `.properties` files

### File Encoding

All files use:
- **UTF-8** with leading BOM (`xEF BB BF`)
- **LF** (linefeed, `x0A`) as end-of-line character
- **2-space indentation** (no tabs)
- Final newline character

---

## Common File Types by Object

### Classes (CLAS)

A class produces up to 6 files:

| File | Content |
|------|---------|
| `zcl_example.clas.abap` | Main class definition and implementation |
| `zcl_example.clas.locals_def.abap` | Local class definitions (optional) |
| `zcl_example.clas.locals_imp.abap` | Local class implementations (optional) |
| `zcl_example.clas.testclasses.abap` | Local test classes (optional) |
| `zcl_example.clas.macros.abap` | Local macros (optional, legacy) |
| `zcl_example.clas.xml` | Class metadata (visibility, description, interfaces, etc.) |

Files are only created if they have content — empty includes are omitted.

### Programs / Reports (PROG)

| File | Content |
|------|---------|
| `z_my_report.prog.abap` | Program source code |
| `z_my_report.prog.xml` | Program metadata (type, description, application) |

### Interfaces (INTF)

| File | Content |
|------|---------|
| `zif_my_interface.intf.abap` | Interface definition |
| `zif_my_interface.intf.xml` | Interface metadata |

### Function Groups (FUGR)

Function groups produce multiple files:

| File | Content |
|------|---------|
| `z_my_fg.fugr.xml` | Function group metadata |
| `z_my_fg.fugr.z_my_fm.abap` | Function module source (one per FM) |
| `z_my_fg.fugr.lz_my_fgtop.abap` | TOP include (global data) |
| `z_my_fg.fugr.lz_my_fguxx.abap` | UXX include (auto-generated FM list) |
| `z_my_fg.fugr.lz_my_fgf01.abap` | Form routines include (optional) |

### CDS Views (DDLS)

| File | Content |
|------|---------|
| `zi_my_view.ddls.asddls` | CDS DDL source code |
| `zi_my_view.ddls.xml` | CDS view metadata |

### Tables (TABL)

| File | Content |
|------|---------|
| `ztmy_table.tabl.xml` | Table definition (fields, keys, technical settings) |

> Note: Tables are fully defined in the XML metadata — there is no separate `.abap` file.

### Structures (TABL — subtype structure)

| File | Content |
|------|---------|
| `zs_my_structure.tabl.xml` | Structure definition |

### Domains (DOMA)

| File | Content |
|------|---------|
| `zd_my_domain.doma.xml` | Domain definition (data type, length, fixed values) |

### Data Elements (DTEL)

| File | Content |
|------|---------|
| `zde_my_element.dtel.xml` | Data element definition (domain ref, labels) |

### Behavior Definitions (BDEF)

| File | Content |
|------|---------|
| `zi_my_entity.bdef.asbdef` | Behavior definition source |
| `zi_my_entity.bdef.xml` | Behavior definition metadata |

### Metadata Extensions (DDLX)

| File | Content |
|------|---------|
| `zc_my_entity.ddlx.asddlxs` | Metadata extension source (UI annotations) |
| `zc_my_entity.ddlx.xml` | Metadata extension metadata |

### Service Definitions (SRVD)

| File | Content |
|------|---------|
| `zsd_my_service.srvd.srvdsrv` | Service definition source |
| `zsd_my_service.srvd.xml` | Service definition metadata |

### Service Bindings (SRVB)

| File | Content |
|------|---------|
| `zui_my_binding.srvb.xml` | Service binding configuration |

### Access Control (DCLS)

| File | Content |
|------|---------|
| `zi_my_view.dcls.asdcls` | Access control source |
| `zi_my_view.dcls.xml` | Access control metadata |

### Packages (DEVC)

| File | Content |
|------|---------|
| `package.devc.xml` | Package metadata (super-package, transport layer) |

### Message Classes (MSAG)

| File | Content |
|------|---------|
| `z_my_messages.msag.xml` | Message class with all messages |

### Number Range Objects (NROB)

| File | Content |
|------|---------|
| `z_my_nrobj.nrob.xml` | Number range object definition |

---

## Classic Format vs ABAP File Format (AFF)

ABAPGit supports two serialization formats:

### Classic Format (XML-based)

- Used for most traditional object types
- Metadata stored in `.xml` files with `<abapGit>` root tag
- Source code in `.abap` files
- Well-established, works on all SAP systems

### ABAP File Format (AFF — JSON-based)

- Newer format developed by SAP
- Metadata stored in `.json` files
- Used for newer object types (CHKC, CHKO, CHKV, APLO, BGQC, etc.)
- Gradually being adopted for more object types
- Defined at [github.com/SAP/abap-file-formats](https://github.com/SAP/abap-file-formats)

Most common object types (CLAS, PROG, FUGR, TABL, DDLS, etc.) still use the classic XML format.

---

## Repository Structure Example

A typical ABAPGit repository for a RAP application might look like:

```
src/
├── package.devc.xml
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
├── zsd_my_service.srvd.srvdsrv
├── zsd_my_service.srvd.xml
├── zui_my_binding.srvb.xml
├── zc_my_entity.ddlx.asddlxs
├── zc_my_entity.ddlx.xml
└── subpackage/
    ├── package.devc.xml
    └── ...
```

The `.abapgit.xml` file in the repository root contains ABAPGit project settings (starting folder, folder logic, etc.).

---

## Working with ABAPGit and This Power

When using the ABAP ADT MCP server alongside ABAPGit:

1. **Reading objects** — Use the MCP server to read source code from the SAP system. The content corresponds to what ABAPGit would serialize into `.abap` files.

2. **Creating/updating objects** — Use the MCP server to create or update objects in the SAP system. After changes, use ABAPGit to push the updated objects to Git.

3. **Local file references** — When working with ABAP objects locally (e.g., in a cloned ABAPGit repo), the file extensions tell you the object type:
   - `.clas.abap` → class source → use `ReadClass` / `UpdateClass`
   - `.prog.abap` → program source → use `ReadProgram` / `UpdateProgram`
   - `.ddls.asddls` → CDS view DDL → use `ReadView` / `UpdateView`
   - `.tabl.xml` → table definition → use `ReadTable` / `UpdateTable`
   - `.bdef.asbdef` → behavior definition → use `ReadBehaviorDefinition` / `UpdateBehaviorDefinition`
   - `.ddlx.asddlxs` → metadata extension → use `ReadMetadataExtension` / `UpdateMetadataExtension`
   - `.intf.abap` → interface → use `ReadInterface` / `UpdateInterface`
   - `.fugr.*.abap` → function module → use `ReadFunctionModule` / `UpdateFunctionModule`

4. **Typical workflow:**
   - Clone the ABAPGit repo locally
   - Use the MCP server to read/modify objects in the SAP system
   - Use ABAPGit (SAP GUI or Eclipse) to stage and push changes to Git
   - Use Git for code review, branching, and history

---

## Key ABAPGit Concepts

| Concept | Description |
|---------|-------------|
| **Online repo** | Linked directly to a Git URL; push/pull over HTTPS |
| **Offline repo** | Uses ZIP file import/export; no direct Git connection |
| **Serializer** | Converts an ABAP object type to/from files |
| **Deserializer** | Converts files back into ABAP objects in the system |
| **Stage** | Select which objects to include in a commit |
| **Pull** | Import objects from Git into the SAP system |
| **Push** | Export objects from the SAP system to Git |
| **`.abapgit.xml`** | Repository settings file (folder logic, starting folder, ignore rules) |

---

## Supported Object Types

ABAPGit supports 100+ ABAP object types. The most commonly used ones include:

| Type | Description | Files |
|------|-------------|-------|
| CLAS | Class | `.clas.abap`, `.clas.xml`, `.clas.locals_*.abap`, `.clas.testclasses.abap` |
| PROG | Program / Report | `.prog.abap`, `.prog.xml` |
| INTF | Interface | `.intf.abap`, `.intf.xml` |
| FUGR | Function Group | `.fugr.xml`, `.fugr.*.abap` |
| TABL | Table / Structure | `.tabl.xml` |
| DDLS | CDS View (DDL Source) | `.ddls.asddls`, `.ddls.xml` |
| DDLX | Metadata Extension | `.ddlx.asddlxs`, `.ddlx.xml` |
| DOMA | Domain | `.doma.xml` |
| DTEL | Data Element | `.dtel.xml` |
| BDEF | Behavior Definition | `.bdef.asbdef`, `.bdef.xml` |
| SRVD | Service Definition | `.srvd.srvdsrv`, `.srvd.xml` |
| SRVB | Service Binding | `.srvb.xml` |
| DCLS | Access Control | `.dcls.asdcls`, `.dcls.xml` |
| DEVC | Package | `.devc.xml` |
| MSAG | Message Class | `.msag.xml` |
| TTYP | Table Type | `.ttyp.xml` |
| ENHO | Enhancement Implementation | `.enho.xml` |
| ENHS | Enhancement Spot | `.enhs.xml` |

For the full list, see [docs.abapgit.org/user-guide/reference/supported.html](https://docs.abapgit.org/user-guide/reference/supported.html).
