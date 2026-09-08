# AI Service Agency Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Build an official-SDK Big Ambitions mod that adds an AI Service Agency office business, mod-owned server-rack furniture, and the confirmed AI-service economy without altering base-game office content.

**Architecture:** The mod is a Unity `2022.3.62f2` project based on Hovgaard Games' official SDK. A mod-owned Unity `BusinessType` asset reuses the base-game `OfficeBusinessSimulator`; mod-owned furniture is registered through the public item API. Role, skill, revenue, maintenance, and save integration are implemented only after the installed game's imported assemblies confirm public, supported APIs for each operation.

**Tech Stack:** Unity `2022.3.62f2`, official Hovgaard Games Big Ambitions Modding SDK, C#, Unity AssetBundles, SDK Mod Builder, Big Ambitions through Steam.

---

## Verified Constraints

- The official SDK repository is `https://github.com/hovgaardgames/bigambitions` at inspected revision `1e03ddd5071b77a32cfd8bba89f0c130ba6c7d73`.
- The SDK's supported content registration calls are `ModdingAPI.RegisterModBusinessType(BusinessType)` and `ItemsGetter.RegisterModItem(Item)`.
- New office content must refer to the SDK's `Assets/_BaDependencies/BusinessSimulators/OfficeBusinessSimulator.asset`.
- Unity scripts compile only after the SDK imports DLLs from an installed Big Ambitions copy. The game is not installed on this machine, so no mod code or Unity assets can yet be safely compiled or validated.
- Build and installation must use `Big Ambitions/Mod Builder > Build & Install`; the SDK places the result in the game's `ModsLocal` folder. Do not manually package a DLL, ship game DLLs, or use BepInEx.

## File Structure

| Path | Responsibility |
| --- | --- |
| `README.md` | Prerequisites, installation flow, compatibility version, gameplay rules, and removal behavior. |
| `Assets/Mods/absolute-ai-service-agency/` | Unity mod folder created in the official SDK project after its DLL import succeeds. |
| `Assets/Mods/absolute-ai-service-agency/Scripts/AiServiceAgencyMod.cs` | Mod lifecycle, asset loading, registration, clean unregistration, and only confirmed runtime integration. |
| `Assets/Mods/absolute-ai-service-agency/AiServiceAgency.asset` | Mod-owned `BusinessType` asset using original office creation requirements and `OfficeBusinessSimulator`. |
| `Assets/Mods/absolute-ai-service-agency/AiServerRack.asset` | Two-slot server-rack furniture `Item` asset. |
| `Assets/Mods/absolute-ai-service-agency/EnterpriseAiServerRack.asset` | Five-slot server-rack furniture `Item` asset. |
| `Assets/Mods/absolute-ai-service-agency/Locales/en.json` | Localized names, descriptions, and financial labels for all mod-owned content. |
| `Assets/Mods/absolute-ai-service-agency/absolute-ai-service-agency.asmdef` | SDK-compilable C# assembly definition with canonical imported game-DLL references. |
| `Assets/Mods/absolute-ai-service-agency/ModManifest.asset` | SDK packager manifest. |
| `docs/runtime/big-ambitions-capability-profile.md` | Game version, confirmed public API signatures, and explicit unsupported capabilities. |
| `docs/runtime/office-baseline.md` | Observed original office creation requirement and Programmer/service-fee financial baseline. |
| `docs/runtime/acceptance-results.md` | Fresh-save and existing-save acceptance evidence. |

### Task 1: Install And Bootstrap The Official SDK

**Files:**
- Create: `README.md`
- Create: `docs/runtime/big-ambitions-capability-profile.md`

- [ ] **Step 1: Install the exact prerequisites**

Install Big Ambitions through Steam. Install Unity Hub plus Unity `2022.3.62f2` with its macOS Build Support module, as required by the official SDK. Record the resolved game directory, Big Ambitions product version, SDK commit, Unity version, and checked date in `docs/runtime/big-ambitions-capability-profile.md`.

- [ ] **Step 2: Clone the official SDK and open it in the required Unity version**

```powershell
git clone https://github.com/hovgaardgames/bigambitions.git BigAmbitionsModSdk
```

Open `BigAmbitionsModSdk` from Unity Hub using Unity `2022.3.62f2`. Do not place mod files in the game installation directory.

- [ ] **Step 3: Import the installed game's DLLs through the SDK welcome flow**

In Unity, follow the SDK welcome dialog and choose the installed game's `Big Ambitions_Data\Managed` directory. Verify that `Assets/_BaDependencies/GameDlls/` contains the canonical assemblies and that the `BA_GAME_DLLS_IMPORTED` compilation define is active.

- [ ] **Step 4: Capture the imported DLL inventory without redistributing it**

Run from the SDK directory after import:

```powershell
Get-ChildItem -LiteralPath 'Assets\_BaDependencies\GameDlls' -File -Filter '*.dll' |
  Sort-Object Name |
  Select-Object Name, Length, LastWriteTime |
  Format-Table -AutoSize |
  Out-File -LiteralPath '..\Absolute\docs\runtime\big-ambitions-dll-inventory.txt' -Encoding utf8
```

Add only the inventory text file to Git. Never add the imported DLL files or their containing directory.

- [ ] **Step 5: Document the build route and commit**

Write `README.md` with these exact steps: open the SDK, import DLLs after game updates, select `Big Ambitions/Mod Builder`, run `Build & Install`, then confirm the mod under `ModsLocal` through the in-game `Mods > Mod Creator` menu.

```powershell
git add README.md docs/runtime/big-ambitions-capability-profile.md docs/runtime/big-ambitions-dll-inventory.txt
git commit -m "docs: document Big Ambitions SDK bootstrap"
```

### Task 2: Profile The Installed Public API Before Writing Game Code

**Files:**
- Modify: `docs/runtime/big-ambitions-capability-profile.md`

- [ ] **Step 1: Inspect public types and members from imported assemblies**

Use an IDE metadata viewer or ILSpy against the DLLs imported into `Assets/_BaDependencies/GameDlls/`. Record fully qualified type names and public method/property signatures for the following capabilities: business and furniture registration; `OfficeBusinessSimulator` inputs and hourly revenue flow; original Web Development Agency creation requirements; Computer Workstation identity and assignment rule; employee-role registration; skill registration; scheduled/current work state; business-owned furniture lookup; revenue recording; daily business-expense recording; and supported mod save data.

- [ ] **Step 2: Complete this capability matrix**

Add this table to `docs/runtime/big-ambitions-capability-profile.md`; each completed row must include the exact observed type and member signature.

| Capability | Required for V1 | Public API found | Exact API | Decision |
| --- | --- | --- | --- | --- |
| Register business type | Yes | Unknown |  |  |
| Register furniture item | Yes | Unknown |  |  |
| Reuse office simulator | Yes | Unknown |  |  |
| Register AI Engineer role | Yes | Unknown |  |  |
| Register AI skill | Yes | Unknown |  |  |
| Determine scheduled working state | Yes | Unknown |  |  |
| Verify Computer Workstation assignment | Yes | Unknown |  |  |
| Count agency-owned server racks | Yes | Unknown |  |  |
| Record hourly business revenue | Yes | Unknown |  |  |
| Record daily business maintenance | Yes | Unknown |  |  |
| Store mod-owned save data | Conditional | Unknown |  |  |

- [ ] **Step 3: Gate V1 without unsupported fallbacks**

Proceed only when every required capability has a public, supported API. If any required row has no public API, record `Unsupported`, do not register the AI Service Agency, and revise the design to omit that dependent mechanic. Do not introduce BepInEx, Harmony, private reflection, or patches.

- [ ] **Step 4: Commit the profile**

```powershell
git add docs/runtime/big-ambitions-capability-profile.md
git commit -m "docs: record AI agency API capabilities"
```

### Task 3: Capture The Unmodified Office Baseline

**Files:**
- Create: `docs/runtime/office-baseline.md`

- [ ] **Step 1: Record original office creation data**

With no local mods enabled, create or load a test save and inspect the selected original office business, expected to be the Web Development Agency. Record its `BusinessType` identity, building type, tags, `businessRequirements`, simulator, employee primary skill, service product identity, and business-creation flow.

- [ ] **Step 2: Observe the original workstation revenue loop**

Hire and schedule one base-game Programmer with one base-game Computer Workstation. Advance through one observed revenue interval and record the Programmer hourly wage, service-fee label and amount, interval duration, schedule state, and finance entries. Do not change that save's original assets.

- [ ] **Step 3: Document the compatibility baseline and commit**

In `docs/runtime/office-baseline.md`, state the exact original creation requirements to duplicate, the Computer Workstation identity needed for eligibility, and the baseline the AI Engineer wage and AI service fee must exceed.

```powershell
git add docs/runtime/office-baseline.md
git commit -m "docs: record original office business baseline"
```

### Task 4: Create The Mod-Owned Unity Content

**Files:**
- Create: `Assets/Mods/absolute-ai-service-agency/ModManifest.asset`
- Create: `Assets/Mods/absolute-ai-service-agency/absolute-ai-service-agency.asmdef`
- Create: `Assets/Mods/absolute-ai-service-agency/Scripts/AiServiceAgencyMod.cs`
- Create: `Assets/Mods/absolute-ai-service-agency/AiServiceAgency.asset`
- Create: `Assets/Mods/absolute-ai-service-agency/AiServerRack.asset`
- Create: `Assets/Mods/absolute-ai-service-agency/EnterpriseAiServerRack.asset`
- Create: `Assets/Mods/absolute-ai-service-agency/AiServerRack.prefab`
- Create: `Assets/Mods/absolute-ai-service-agency/EnterpriseAiServerRack.prefab`
- Create: `Assets/Mods/absolute-ai-service-agency/Locales/en.json`

- [ ] **Step 1: Create the manifest and assembly definition through Unity**

Create `Assets/Mods/absolute-ai-service-agency/`. Use `Assets > Create > Big Ambitions > Mod Manifest`; set `ModId` to `absolute-ai-service-agency`, display name to `AI Service Agency`, version `0.1.0`, AssetBundle name to `absolute-ai-service-agency.unity3d`, Windows as target, the mod's own assembly definition, and its `Locales` folder. Copy the current SDK example's canonical precompiled-reference list unchanged into the assembly definition, changing only its assembly name; set `overrideReferences` to `true`, `autoReferenced` to `false`, and the only define constraint to `BA_GAME_DLLS_IMPORTED`.

- [ ] **Step 2: Create the business asset in the Unity Inspector**

Create a mod-owned `BusinessType` named `AiServiceAgency`. Use localization ID `absolute-ai-service-agency:businesstype_ai_service_agency`, assign `Assets/_BaDependencies/BusinessSimulators/OfficeBusinessSimulator.asset`, and copy `suitableBuildingType`, tags, office service product, employee primary skill, and `businessRequirements` from `office-baseline.md`. Never edit the original office asset.

- [ ] **Step 3: Create both furniture items and prefabs**

Create matching mod-owned `Item` assets and prefabs for two racks. Set `isFurniture` true, use unique names/localization IDs, and use only compatible observed placement metadata. Localized descriptions must state `2 AI Engineer server slots` for the basic rack and `5 AI Engineer server slots` for the enterprise rack. Set price and maintenance only after Task 3's Programmer baseline is known.

- [ ] **Step 4: Implement load and unload registration using confirmed APIs**

Follow the official `ExampleBusinessTypeMod` and `ExampleFurnitureMod` lifecycle: load the AssetBundle with `AssetService.GetBundle`, load the three assets by asset path, register both furniture `Item` objects and the `BusinessType`, then unregister the same objects in `OnUnloadAsync`. Fail load clearly for a missing AssetBundle asset. Do not register content before Task 2's required matrix is complete.

- [ ] **Step 5: Add localization, compile, validate, and commit**

Add `Locales/en.json` entries for agency, AI Engineer, AI skill, service fee, both furniture names/descriptions, server-capacity status, `Hourly AI Service Fee`, and `AI Server Maintenance`. Every key begins `absolute-ai-service-agency:`. Confirm the Unity Console has no compilation errors and run `Big Ambitions/Mod Builder` validation. Commit only source and Unity assets, never `Library/`, `Temp/`, `Logs/`, imported game DLLs, or `ModsLocal` output.

```powershell
git add Assets/Mods/absolute-ai-service-agency README.md
git commit -m "feat: add AI service agency Unity content"
```

### Task 5: Implement Confirmed AI-Service Economy Hooks

**Files:**
- Modify: `Assets/Mods/absolute-ai-service-agency/Scripts/AiServiceAgencyMod.cs`
- Create: `Assets/Mods/absolute-ai-service-agency/Scripts/AiServiceAllocation.cs`
- Create: `Assets/Mods/absolute-ai-service-agency/Scripts/AiServiceSettlement.cs`
- Create: `Assets/Mods/absolute-ai-service-agency/Tests/EditMode/AiServiceAllocationTests.cs`

- [ ] **Step 1: Write a failing EditMode allocation test**

Test a known immutable list of employee snapshots holding persistent ID, AI skill, scheduled-and-working status, and workstation validity. Assert capacity is `2 * basicRacks + 5 * enterpriseRacks`; employees without workstations are excluded; highest AI skill wins; equal skill sorts by ordinal employee ID; two enterprise racks cover ten employees; and ten enterprise racks cover fifty employees.

- [ ] **Step 2: Run it through Unity Test Runner**

Confirm the test fails before `AiServiceAllocation` exists.

- [ ] **Step 3: Implement the deterministic allocation model**

Implement a small Unity-independent model that filters `IsScheduledAndWorking && HasComputerWorkstation`, sorts descending by `AiSkill` then ordinal `EmployeeId`, and takes calculated capacity. It must retain no game objects, mutate no original employee object, and use no non-deterministic collection order.

- [ ] **Step 4: Verify the test passes and bind only confirmed APIs**

Run the EditMode test and verify it passes. Then use only the exact public APIs recorded in Task 2 to identify each agency, count its placed racks, select working AI Engineers, verify Computer Workstation assignment, allocate capacity at each confirmed income period, post `Hourly AI Service Fee`, and charge every placed rack once per confirmed in-game day as `AI Server Maintenance`.

The AI Engineer wage and all generated fees must be strictly greater than the measured Programmer values in `office-baseline.md`. Charge maintenance for unused capacity. Do not change original-role wages, original office revenue, original workstation behavior, or base-game finance records.

- [ ] **Step 5: Commit the verified economy**

```powershell
git add Assets/Mods/absolute-ai-service-agency/Scripts Assets/Mods/absolute-ai-service-agency/Tests
git commit -m "feat: add AI service capacity and settlement"
```

### Task 6: Build, Install, And Verify Save Safety

**Files:**
- Create: `docs/runtime/acceptance-results.md`
- Modify: `README.md`

- [ ] **Step 1: Build through the official SDK**

Use `Big Ambitions/Mod Builder > Build & Install`. Confirm validation passes and the resulting mod is present under `ModsLocal`. Do not create a manual ZIP or copy game DLLs.

- [ ] **Step 2: Test a fresh save and record evidence**

Record save name, version, and financial evidence in `docs/runtime/acceptance-results.md` for: original office creation prerequisites with no extra unlock; one workstation/rack/working AI Engineer producing service revenue; a working engineer without a workstation producing no service revenue; over-capacity high-skill and ID tie allocation; unused rack daily maintenance; two enterprise racks covering ten employees; and ten enterprise racks covering fifty employees.

- [ ] **Step 3: Test existing-save non-regression**

With the mod enabled, open the mod-free baseline save from Task 3 and confirm original Web Development Agency, Programmer wage, service fee, and Computer Workstation behavior match `office-baseline.md`. Disable the mod and reopen the same save; confirm the original business loads without data changes.

- [ ] **Step 4: Final source verification and commit**

```powershell
git diff --check
git status --short
git add README.md docs/runtime/acceptance-results.md
git commit -m "docs: record AI agency acceptance testing"
```

Commit acceptance evidence only after every result is observed. The published Mod Builder output inside `ModsLocal` may contain only mod-owned assemblies, bundles, localization, manifest-derived metadata, and mod-owned assets.
