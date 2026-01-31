# Web API (LaunchData) & Build Instructions

This document covers the **LaunchAccount** web API `LaunchData` parameter and how to **compile** the project locally or via **GitHub Actions**.

---

## LaunchAccount Web API — LaunchData Parameter

The Web API supports an optional **LaunchData** (or **launchData**) query parameter when launching an account into a game. This is the same data you can enter in the RAM UI’s “Launch Data” field (e.g. Roblox `launchData` JSON).

### Endpoint

- **Method:** `LaunchAccount`
- **Query parameters:**
  - **PlaceId** (required) — Roblox place ID.
  - **JobId** (optional) — Server job ID, or leave empty for a random server.
  - **FollowUser** (optional) — `"true"` to join by user (not used with PlaceId launch).
  - **JoinVIP** (optional) — `"true"` for VIP/private server.
  - **LaunchData** or **launchData** (optional) — JSON string passed to Roblox as `launchData` (e.g. `{"psCode":"lanrp"}`).

### Example requests

**Launch into place with launchData (JSON):**

```
GET http://localhost:PORT/Method?Method=LaunchAccount&Account=USERNAME&PlaceId=4924922222&LaunchData={"psCode":"lanrp"}
```

**URL-encoded (recommended for non-trivial JSON):**

- Parameter name: `LaunchData` or `launchData`
- Value: valid JSON, optionally URL-encoded (e.g. `%7B%22psCode%22%3A%22lanrp%22%7D` for `{"psCode":"lanrp"}`).

**cURL example:**

```bash
curl -G "http://localhost:PORT/Method" \
  --data-urlencode "Method=LaunchAccount" \
  --data-urlencode "Account=YourAccountName" \
  --data-urlencode "PlaceId=4924922222" \
  --data-urlencode 'LaunchData={"psCode":"lanrp"}'
```

**PowerShell example:**

```powershell
$base = "http://localhost:PORT/Method"
$query = @{
  Method   = "LaunchAccount"
  Account  = "YourAccountName"
  PlaceId  = "4924922222"
  LaunchData = '{"psCode":"lanrp"}'
}
Invoke-RestMethod -Uri $base -Method Get -Body $query
```

Replace `PORT` with the Web Server port configured in RAM (Settings → Web Server), and include password parameters if the Web Server is password-protected.

### Behavior

- If **LaunchData** / **launchData** is omitted or empty, the launch behaves as before (no `launchData`).
- If provided, the value is passed through to `Account.JoinServer(...)` and then to Roblox (e.g. `roblox://experiences/start?placeId=...&launchData=...`). Invalid JSON may result in an error from RAM.

---

## Compiling the Project

The solution is **.NET Framework 4.7.2**, **Windows-only** (WinForms, CefSharp). Build on Windows with Visual Studio or MSBuild.

### Prerequisites (local build)

- **Windows** (required).
- **Visual Studio 2022** (or 2019) with workload **“.NET desktop development”** (includes .NET Framework 4.7.2 and MSBuild).
- **NuGet** (VS usually includes it; for command line you can use `nuget.exe` or the VS Developer Command Prompt).

### Option 1: Visual Studio (GUI)

1. Clone or open the repo and open **`RBX Alt Manager.sln`** in Visual Studio.
2. Let NuGet restore packages (automatic on open, or right-click solution → **Restore NuGet Packages**).
3. Set configuration to **Release** and platform **Any CPU**.
4. **Build → Build Solution** (or F6).
5. Output is under **`build\Release\`** (e.g. `Roblox Account Manager.exe`).

### Option 2: Command line (MSBuild)

From a **Developer Command Prompt for VS** (or a shell where `msbuild` and `nuget` are on PATH):

```bat
cd path\to\Roblox-Account-Manager-3.7.3

nuget restore "RBX Alt Manager.sln"
msbuild "RBX Alt Manager.sln" /p:Configuration=Release /p:Platform="Any CPU"
```

If `nuget` is not in PATH, use the full path to `nuget.exe` or install it from [nuget.org](https://www.nuget.org/downloads). Output is in **`build\Release\`**.

### Option 3: GitHub Actions (CI)

The repo includes a workflow that builds the solution on every push and pull request (and optionally on a schedule or release).

- **Workflow file:** `.github/workflows/build.yml`
- **Trigger:** Push and pull_request to the default branch (and optionally `workflow_dispatch`).
- **Runner:** `windows-latest` (Windows 2022).
- **Steps:**
  1. Checkout repo.
  2. Setup MSBuild (Visual Studio build tools).
  3. Restore NuGet packages.
  4. Build the solution in **Release | Any CPU**.
  5. Upload the contents of **`build\Release\`** as a workflow artifact named `Roblox-Account-Manager-Release`.

**To use the GitHub Actions build:**

1. Push your changes (or open a PR) so the workflow runs.
2. Open the **Actions** tab → select the **Build** workflow run.
3. Download the **`Roblox-Account-Manager-Release`** artifact from the run summary; it contains the built output (e.g. `Roblox Account Manager.exe` and dependencies).

**To run the workflow manually:** If `workflow_dispatch` is enabled in `build.yml`, go to Actions → Build → “Run workflow”.

---

## Summary

| Topic | Details |
|--------|--------|
| **LaunchData in Web API** | Optional query parameter `LaunchData` or `launchData` on `LaunchAccount`; value is JSON (e.g. `{"psCode":"lanrp"}`), optionally URL-encoded. |
| **Local build** | Open `RBX Alt Manager.sln` in Visual Studio, restore NuGet, build Release; or use `nuget restore` + `msbuild` from command line. Output: `build\Release\`. |
| **GitHub Actions build** | Use `.github/workflows/build.yml`; artifact `Roblox-Account-Manager-Release` contains the compiled app. |
