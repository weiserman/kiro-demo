---
inclusion: manual
name: abap-adt-a4h-install
description: "Step-by-step installation of the abap-adt-a4h MCP server on Windows and macOS — prerequisites, BTP vs on-prem credentials, Kiro config, verification, troubleshooting."
---

# Installing the ABAP ADT MCP Server

This guide walks you through installing `mcp-abap-adt` on **Windows** or **macOS**. The server supports two authentication modes:

- **BTP service keys** — OAuth2 client credentials for SAP BTP ABAP Environment (no interactive login after initial setup)
- **On-premise basic auth** — username/password via `.env` file for on-prem SAP systems

> Installation steps differ between Windows and macOS. Follow the section for your platform.

---

## Prerequisites

### SAP System Access

You need one of the following:

**For BTP ABAP Environment:**
- A BTP sub-account with an ABAP Environment instance
- A service key with `credential-type: binding-secret` containing a `uaa` block (`clientid`, `clientsecret`) and a top-level `url`

**For On-Premise SAP:**
- System URL (e.g. `https://vhcala4hci:50001`)
- Valid username and password
- SAP client number (e.g. `001`)
- System ID / SID (e.g. `A4H`)
- ADT services activated in transaction SICF (path: `/sap/bc/adt`) — your basis administrator can help

---

## Windows Installation

### Prerequisites (Windows)

**Install nvm-windows** (Node Version Manager):

```powershell
winget install CoreyButler.NVMforWindows
```

Restart PowerShell as Administrator, then:

```powershell
nvm install lts
nvm use lts

# Verify (you may need to reopen PowerShell)
node -v
npm -v
```

### Step 1: Install the MCP Server (Windows)

The server is installed globally via npm:

```powershell
npm install -g @mcp-abap-adt/connection
```

Verify:

```powershell
where.exe mcp-abap-adt
mcp-abap-adt --help
```

### Step 2: Set Up Credentials (Windows)

#### Option A: BTP Service Keys (Windows)

1. **Create the service-keys directory:**

   ```powershell
   New-Item -ItemType Directory -Force -Path "C:\sap\service-keys"
   ```

2. **Save the service key JSON** from your BTP cockpit into the directory. The filename (without `.json`) becomes the destination name.

   Example: save as `C:\sap\service-keys\AC1.json` → destination name is `AC1`.

   The service key must contain a `uaa` block with `clientid` / `clientsecret` and a top-level `url` pointing at the ABAP system. When these are present, the server uses the OAuth2 client-credentials grant.

3. **Test the connection:**

   ```powershell
   mcp-abap-adt --mcp=AC1 --auth-broker-path=C:\sap
   ```

   A browser window should open asking you to log into the SAP BTP ABAP system. After successful login you'll see a confirmation in the browser and the terminal.

4. **(Optional) Set AUTH_BROKER_PATH permanently** so you don't need `--auth-broker-path` every time:

   - `Win + R`, type `sysdm.cpl`, Enter
   - **Advanced** tab → **Environment Variables**
   - Under **User variables**, click **New**
   - Variable name: `AUTH_BROKER_PATH`
   - Variable value: `C:\sap`
   - OK all windows
   - **Close and reopen** your terminal

#### Option B: On-Premise Basic Auth (Windows)

1. **Create the service-keys directory:**

   ```powershell
   New-Item -ItemType Directory -Force -Path "C:\sap\service-keys"
   ```

2. **Create the `.env` file** (copy/paste and edit the values):

   ```powershell
   @'
   SAP_URL=https://vhcala4hci:50001
   SAP_CLIENT=001
   SAP_USERNAME=DEVELOPER
   SAP_PASSWORD=YourPasswordHere
   SAP_AUTH_TYPE=basic
   SAP_SYSTEM_TYPE=onprem
   SAP_LANGUAGE=EN
   TLS_REJECT_UNAUTHORIZED=0
   SAP_MASTER_SYSTEM=AC1
   '@ | Set-Content -Path "C:\sap\service-keys\AC1.env" -Encoding ascii
   ```

   **Alternative (manual):** open `C:\sap\service-keys` in Explorer, create a new file named `AC1.env` (make sure it's not `AC1.env.txt`), and paste the template from the reference section below.

3. **Quick smoke test (optional):**

   ```powershell
   mcp-abap-adt --transport=http --http-port=3000 --env=AC1
   ```

   You should see the server start and listen on port 3000. Press `Ctrl+C` to stop.

### Step 3: Configure Kiro (Windows)

Create or edit `.kiro/settings/mcp.json` (workspace level) or `~/.kiro/settings/mcp.json` (user level).

**BTP configuration:**
```json
{
  "mcpServers": {
    "abap-adt-btp": {
      "command": "mcp-abap-adt",
      "args": [
        "--mcp=AC1",
        "--auth-broker-path=C:\\sap"
      ],
      "env": {
        "AUTH_BROKER_PATH": "C:\\sap"
      }
    }
  }
}
```

**On-premise configuration:**
```json
{
  "mcpServers": {
    "abap-adt-a4h": {
      "command": "mcp-abap-adt",
      "args": [
        "--env-path=C:\\sap\\service-keys\\AC1.env"
      ]
    }
  }
}
```

---

## macOS Installation

### Prerequisites (macOS)

- **Homebrew** — install from [brew.sh](https://brew.sh) if you don't have it
- **Git** — `brew install git` (or Xcode command line tools)
- **Node.js 18+** — via Homebrew or nvm:

  ```bash
  # Option 1: Homebrew
  brew install node

  # Option 2: nvm (recommended)
  curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.40.1/install.sh | bash
  # Restart terminal, then:
  nvm install --lts
  nvm use --lts
  ```

  Verify:
  ```bash
  node -v
  npm -v
  ```

### Step 1: Install the MCP Server from Source (macOS)

On macOS, the server is built from source:

```bash
git clone --recurse-submodules https://github.com/fr0ster/mcp-abap-adt.git
cd mcp-abap-adt
npm install
npm run build
npm install -g .
```

Verify:

```bash
which mcp-abap-adt
mcp-abap-adt --help | head
```

### Step 2: Set Up Credentials (macOS)

#### Option A: BTP Service Keys (macOS)

1. **Create the service-keys directory:**

   ```bash
   mkdir -p ~/.config/mcp-abap-adt/service-keys
   ```

2. **Copy your downloaded service key JSON** into the directory. The filename (without `.json`, case-sensitive) becomes the destination name.

   ```bash
   # Example: AC1.json -> destination name "AC1"
   cp ~/Downloads/service-key.json ~/.config/mcp-abap-adt/service-keys/AC1.json
   chmod 600 ~/.config/mcp-abap-adt/service-keys/AC1.json
   ```

   The service key must contain a `uaa` block with `clientid` / `clientsecret` (`credential-type: binding-secret`) and a top-level `url` pointing at the ABAP system. When these are present, the server uses the OAuth2 client-credentials grant and connects without any interactive login.

3. **Smoke test (optional):**

   ```bash
   mcp-abap-adt --transport=http --http-port=3000 --auth-broker --mcp=AC1
   ```

   In another terminal:

   ```bash
   curl -s -X POST http://localhost:3000/mcp/stream/http \
     -H "Content-Type: application/json" \
     -H "Accept: application/json, text/event-stream" \
     -d '{"jsonrpc":"2.0","id":1,"method":"tools/call","params":{"name":"GetPackageContents","arguments":{"package_name":"ZLOCAL"}}}'
   ```

   A JSON response listing sub-packages means auth and connectivity are working. Stop the server with `Ctrl+C`.

#### Option B: On-Premise Basic Auth (macOS)

1. **Create the sessions directory:**

   ```bash
   mkdir -p ~/.config/mcp-abap-adt/sessions
   ```

   > On macOS, on-prem `.env` files go in `sessions/` (not `service-keys/`). The server resolves `--env=<NAME>` to `~/.config/mcp-abap-adt/sessions/<NAME>.env`.

2. **Create the `.env` file:**

   ```bash
   cat > ~/.config/mcp-abap-adt/sessions/AC1.env <<'EOF'
   SAP_URL=https://vhcala4hci:50001
   SAP_CLIENT=001
   SAP_USERNAME=DEVELOPER
   SAP_PASSWORD=YourPasswordHere
   SAP_AUTH_TYPE=basic
   SAP_SYSTEM_TYPE=onprem
   SAP_LANGUAGE=EN
   TLS_REJECT_UNAUTHORIZED=0
   SAP_MASTER_SYSTEM=AC1
   EOF
   chmod 600 ~/.config/mcp-abap-adt/sessions/AC1.env
   ```

3. **Smoke test (optional):**

   ```bash
   mcp-abap-adt --transport=http --http-port=3000 --env=AC1
   ```

   In another terminal:

   ```bash
   curl -s -X POST http://localhost:3000/mcp/stream/http \
     -H "Content-Type: application/json" \
     -H "Accept: application/json, text/event-stream" \
     -d '{"jsonrpc":"2.0","id":1,"method":"tools/call","params":{"name":"GetPackageContents","arguments":{"package_name":"$TMP"}}}'
   ```

   > Do **not** pass `--auth-broker` here — that flag forces service-key mode and ignores `.env`. For on-prem, let the server pick up the `.env`.

### Step 3: Configure Kiro (macOS)

Edit `~/.kiro/settings/mcp.json` (user level) or `.kiro/settings/mcp.json` in your workspace root.

**BTP configuration:**
```json
{
  "mcpServers": {
    "abap-adt-btp": {
      "command": "mcp-abap-adt",
      "args": ["--mcp=AC1"]
    }
  }
}
```

**On-premise configuration:**
```json
{
  "mcpServers": {
    "abap-adt-a4h": {
      "command": "mcp-abap-adt",
      "args": ["--env=AC1"]
    }
  }
}
```

### Multiple BTP Systems (macOS)

Drop additional service keys into `~/.config/mcp-abap-adt/service-keys/` (e.g. `DEV.json`, `PROD.json`) and add one entry per destination:

```json
{
  "mcpServers": {
    "abap-adt-dev":  { "command": "mcp-abap-adt", "args": ["--mcp=DEV"] },
    "abap-adt-prod": { "command": "mcp-abap-adt", "args": ["--mcp=PROD"] }
  }
}
```

You can have BTP and on-prem entries side-by-side — each spawns its own stdio process with its own destination.

---

## .env Template Reference

Replace placeholders with your actual values:

```env
SAP_URL=https://<host>:<port>
SAP_CLIENT=001
SAP_USERNAME=<your_sap_user>
SAP_PASSWORD=<your_password>
SAP_AUTH_TYPE=basic
SAP_SYSTEM_TYPE=onprem
SAP_LANGUAGE=EN
TLS_REJECT_UNAUTHORIZED=0
SAP_MASTER_SYSTEM=<SID>
```

**Important notes:**
- If your password contains `#`, enclose it in quotes: `SAP_PASSWORD="my#password"`
- `SAP_SYSTEM_TYPE=onprem` is **required** — some tools (e.g. Programs, RuntimeRunProgramWithProfiling) are hidden without it
- `SAP_MASTER_SYSTEM` must match the SAP system ID (SID) for transport-request binding on create/update operations
- `TLS_REJECT_UNAUTHORIZED=0` is only needed when the SAP server uses an untrusted / self-signed cert. Prefer importing the cert properly in production.
- Never commit `.env` files to Git

---

## Platform Comparison: On-Prem vs BTP at a Glance

| Aspect | BTP (service key) | On-prem (.env) |
|--------|-------------------|----------------|
| **Credential file (Windows)** | `C:\sap\service-keys\<NAME>.json` | `C:\sap\service-keys\<NAME>.env` |
| **Credential file (macOS)** | `~/.config/mcp-abap-adt/service-keys/<NAME>.json` | `~/.config/mcp-abap-adt/sessions/<NAME>.env` |
| **Auth method** | OAuth2 client credentials (from `uaa.clientsecret`) | HTTP basic (`SAP_USERNAME` / `SAP_PASSWORD`) |
| **Flag** | `--mcp=<NAME>` (+ `--auth-broker` when `.env` exists in cwd) | `--env=<NAME>` or `--env-path=<full-path>` |
| **SAP_SYSTEM_TYPE** | defaults to `cloud` | must be `onprem` |
| **Browser flow** | None (server-to-server) | None |

---

## Key Flags Reference

| Flag | Purpose |
|------|---------|
| `--mcp=<NAME>` | Destination name (loads service key `<NAME>.json` for BTP auth) |
| `--auth-broker-path=<PATH>` | Directory containing `service-keys/` folder (Windows) |
| `--auth-broker` | Force service key auth (ignore any `.env` in cwd) |
| `--env-path=<PATH>` | Full path to `.env` file (on-prem basic auth) |
| `--env=<NAME>` | Short form — resolves to `sessions/<NAME>.env` (macOS) or `service-keys/<NAME>.env` (Windows) |
| `--transport=http` | Use HTTP transport (for smoke testing only; Kiro uses stdio) |
| `--http-port=<PORT>` | Port for HTTP smoke test (default: 3000) |

---

## Verify the Connection

After configuring, Kiro should automatically detect and connect to the MCP server.

1. **Check the MCP Servers panel** in Kiro — the server should appear as connected.
2. **Try a read:** "Search for ABAP objects matching `SAPMV45A`".

If you get results, you're all set.

---

## Recommended Kiro / VS Code Extensions

For the best experience working with ABAP files locally (e.g. from an ABAPGit-cloned repository), install these extensions. They provide syntax highlighting, linting, and file-format support for `.abap`, `.asddls`, `.asbdef`, and other ABAPGit file types.

### Quick Install: Extension Pack (recommended)

The **Standalone ABAP Development Extension Pack** bundles all the essential extensions in one install:

- **Extension ID:** `larshp.standalone-abap-development`
- **Install:** open the Extensions panel in Kiro, search for `Standalone ABAP Development`, click Install.

This pack includes:
- ABAP syntax highlighting
- ABAPLint (static analysis / linting)
- ABAP CDS Language Support (syntax highlighting for `.asddls`, `.asdcls`)
- ABAP File Formats (JSON schema validation for AFF files)
- ABAP Artifacts Helper
- ABAP JSON Editor

### Individual Extensions

If you prefer to install extensions individually:

| Extension | ID | Purpose |
|-----------|-----|---------|
| **ABAP** | `larshp.vscode-abap` | Syntax highlighting for `.abap` files |
| **ABAPLint** | `larshp.vscode-abaplint` | Static analysis and linting for ABAP code |
| **ABAP CDS Language Support** | `hudakf.cds` | Syntax highlighting for CDS views (`.asddls`, `.asdcls`) |
| **ABAP File Formats** | `larshp.vscode-abap-file-formats` | JSON schema validation for AFF metadata files |

### ABAPLint Configuration

ABAPLint provides static analysis for ABAP code. It works locally without an SAP system connection — it reads `.abap` files directly.

**Setup:**

1. Install the ABAPLint extension (included in the extension pack above).

2. Create an `abaplint.json` file in your repository root. A minimal configuration:

   ```json
   {
     "global": {
       "files": "/src/**/*.*"
     },
     "syntax": {
       "version": "v816",
       "errorNamespace": "^(Z|Y|LCL_|LIF_)"
     },
     "rules": {
       "description_empty": true,
       "empty_statement": true,
       "empty_structure": true,
       "max_one_statement": true,
       "method_length": {
         "statements": 100,
         "errorWhenEmpty": true
       },
       "naming": {
         "patternKind": "required",
         "patternType": "regex",
         "patterns": [
           { "object": "CLAS", "regex": "^ZCL_" },
           { "object": "INTF", "regex": "^ZIF_" }
         ]
       },
       "obsolete_statement": { "compute": true, "move": true, "refresh": true },
       "preferred_compare_operator": {
         "badOperators": ["EQ", "NE", "GE", "GT", "LE", "LT"]
       }
     }
   }
   ```

3. ABAPLint will automatically lint `.abap` files when you open them.

**Key `syntax.version` values:**

| Value | Release |
|-------|---------|
| `v702` | SAP BASIS 7.02 |
| `v740sp08` | SAP BASIS 7.40 SP08 (ABAP 7.4) |
| `v750` | SAP BASIS 7.50 |
| `v758` | SAP BASIS 7.58 (S/4HANA 2023) |
| `Cloud` | ABAP Cloud (BTP / S/4HANA Cloud) |

Set this to match your target SAP system version so ABAPLint flags syntax that won't compile on your system.

**Full rules reference:** [rules.abaplint.org](https://rules.abaplint.org)

### File Associations

If Kiro doesn't automatically recognize ABAP file types, add these to your `settings.json`:

```json
{
  "files.associations": {
    "*.abap": "abap",
    "*.asddls": "cds",
    "*.asdcls": "cds",
    "*.asbdef": "cds",
    "*.asddlxs": "cds",
    "*.srvdsrv": "cds"
  }
}
```

---

## Troubleshooting

### `mcp-abap-adt` command not found
- **Windows:** ensure `npm install -g @mcp-abap-adt/connection` succeeded. Verify npm global bin is in PATH: `npm config get prefix`
- **macOS:** ensure `npm install -g .` succeeded from the cloned repo. Check `which mcp-abap-adt`
- Restart your terminal after installation

### `node -v` or `npm -v` gives an error
- **Windows with nvm:** restart PowerShell as Administrator, then `nvm install lts` and `nvm use lts`
- **macOS with nvm:** restart terminal, then `nvm install --lts` and `nvm use --lts`
- **macOS with Homebrew:** `brew install node`

### Missing SAP connection context
- The server was started without `--mcp=<NAME>` (or without `--env=<NAME>` for on-prem). Add the appropriate flag.

### AuthBroker not initialized
- A `.env` file in the current working directory is shadowing the service key. Remove it, or pass `--auth-broker` explicitly.

### Failed to get SAP URL from destination
- Service key filename doesn't match the destination name passed to `--mcp`, or the JSON is missing the top-level `url` field.

### BTP connection fails
- Verify the service key JSON is in the correct directory and contains a valid `uaa` block with `clientid` and `clientsecret`
- Check that the top-level `url` in the service key points to the correct ABAP system

### On-prem connection fails
- Verify the `.env` file path is correct and the file exists
- Check all required variables are set: `SAP_URL`, `SAP_CLIENT`, `SAP_USERNAME`, `SAP_PASSWORD`, `SAP_AUTH_TYPE`
- Ensure `SAP_SYSTEM_TYPE=onprem` is set — without it, some tools are hidden
- If using self-signed certificates, ensure `TLS_REJECT_UNAUTHORIZED=0` is in the `.env` file

### Empty results from $TMP
- Expected: `$TMP` is user-scoped and a technical client has none. Query a real package like `ZLOCAL` instead.

### SAP system rejects the connection
- Verify your credentials are correct
- Ensure your SAP user has authorization for ADT services
- Check that `/sap/bc/adt` is activated in SICF
- Ask your basis administrator to verify the ICF service configuration

### Kiro doesn't show the MCP server
- Double-check the JSON syntax in your `mcp.json` file
- On Windows, use escaped backslashes (`\\`) in JSON paths
- Restart Kiro (Command Palette → Reload Window) after editing `mcp.json`

### Secret rotation
- Treat the service key as a credential. If it leaks, rotate it in the BTP cockpit and replace the file.
