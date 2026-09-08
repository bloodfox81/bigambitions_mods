# Big Ambitions Business Expansion Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Build an official-SDK Big Ambitions mod that adds five retail businesses plus an AI Service Agency office business without altering base-game content.

**Architecture:** The mod is a Unity `2022.3.62f2` project based on Hovgaard Games' official SDK. Five mod-owned retail `BusinessType` assets and twenty-five mod-owned retail product `Item` assets reuse the confirmed base-game retail simulator, Customer Service skill, retail furniture, and product-source rules. The AI Service Agency reuses `OfficeBusinessSimulator`; its role, skill, revenue, maintenance, and save integration are implemented only after the installed game's imported assemblies confirm public APIs for each operation.

**Tech Stack:** Unity `2022.3.62f2`, official Hovgaard Games Big Ambitions Modding SDK, C#, Unity AssetBundles, SDK Mod Builder, Big Ambitions through Steam.

---

## Verified Constraints

- The official SDK repository is `https://github.com/hovgaardgames/bigambitions` at inspected revision `1e03ddd5071b77a32cfd8bba89f0c130ba6c7d73`.
- The SDK's supported content registration calls are `ModdingAPI.RegisterModBusinessType(BusinessType)` and `ItemsGetter.RegisterModItem(Item)`.
- New office content must refer to the SDK's `Assets/_BaDependencies/BusinessSimulators/OfficeBusinessSimulator.asset`.
- Retail content must use a verified base-game retail simulator, retail product-source IDs, and compatible unmodified base-game shelves/display cases; no retail furniture, retail role, retail skill, or retail settlement hook is added.
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
| `Assets/Mods/absolute-ai-service-agency/Retail/` | Five retail `BusinessType` assets and the twenty-five exclusive mod-owned product `Item` assets. |
| `Assets/Mods/absolute-ai-service-agency/Locales/en.json` | Localized office and retail business names, product names, and descriptions. |
| `docs/runtime/big-ambitions-capability-profile.md` | Game version, confirmed public API signatures, and explicit unsupported capabilities. |
| `docs/runtime/office-baseline.md` | Observed original office creation requirement and Programmer/service-fee financial baseline. |
| `docs/runtime/retail-baseline.md` | Observed original retail business simulator, requirements, product sources, compatible furniture, and Customer Service skill. |
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

Use an IDE metadata viewer or ILSpy against the DLLs imported into `Assets/_BaDependencies/GameDlls/`. Record fully qualified type names and public method/property signatures for business and item registration; retail simulator inputs; retail product-source and furniture compatibility; original retail creation requirements; `OfficeBusinessSimulator` inputs and hourly revenue flow; original Web Development Agency creation requirements; Computer Workstation assignment; employee-role registration; skill registration; scheduled/current work state; business-owned furniture lookup; revenue recording; daily business-expense recording; and supported mod save data.

- [ ] **Step 2: Complete this capability matrix**

Add this table to `docs/runtime/big-ambitions-capability-profile.md`; each completed row must include the exact observed type and member signature.

| Capability | Required for V1 | Public API found | Exact API | Decision |
| --- | --- | --- | --- | --- |
| Register business type | Yes | Unknown |  |  |
| Register furniture item | Yes | Unknown |  |  |
| Register retail product item | Yes | Unknown |  |  |
| Reuse retail simulator | Yes | Unknown |  |  |
| Reuse retail product sources | Yes | Unknown |  |  |
| Validate shelf/display compatibility | Yes | Unknown |  |  |
| Reuse Customer Service skill | Yes | Unknown |  |  |
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

### Task 4: Capture The Unmodified Retail Baseline

**Files:**
- Create: `docs/runtime/retail-baseline.md`

- [ ] **Step 1: Select and inspect the original retail reference business**

With no local mods enabled, inspect one original general retail business that supports mod-owned products and normal shelf/display sales. Record its `BusinessType` identity, simulator, suitable building type, tags, `businessRequirements`, product sources, Customer Service primary skill, creation flow, and customer-demand settings.

- [ ] **Step 2: Record compatible unmodified furniture**

For each selected original shelf, display case, and cash register, record its item identity, supported item type/placement capability, and the observed way it participates in normal retail sales. Identify at least one valid shelf and one valid display case before setting retail item metadata.

- [ ] **Step 3: Observe normal retail sales and sourcing**

On a mod-free save, source a reference product through every supported product-source route, stock it on the selected shelf and display case, hire and schedule a Customer Service employee, and record the observed sale and financial entry. The retail expansion must use this loop rather than custom revenue code.

- [ ] **Step 4: Commit the baseline evidence**

```powershell
git add docs/runtime/retail-baseline.md
git commit -m "docs: record original retail business baseline"
```

### Task 5: Create The Mod-Owned Unity Content

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
- Create: `Assets/Mods/absolute-ai-service-agency/Retail/CosmeticsStore.asset`
- Create: `Assets/Mods/absolute-ai-service-agency/Retail/PetSupplyStore.asset`
- Create: `Assets/Mods/absolute-ai-service-agency/Retail/OpticalStore.asset`
- Create: `Assets/Mods/absolute-ai-service-agency/Retail/BuildingBlockStore.asset`
- Create: `Assets/Mods/absolute-ai-service-agency/Retail/MusicStore.asset`
- Create: `Assets/Mods/absolute-ai-service-agency/Retail/Products/` containing 25 product `Item` assets

- [ ] **Step 1: Create the manifest and assembly definition through Unity**

Create `Assets/Mods/absolute-ai-service-agency/`. Use `Assets > Create > Big Ambitions > Mod Manifest`; set `ModId` to `absolute-ai-service-agency`, display name to `AI Service Agency`, version `0.1.0`, AssetBundle name to `absolute-ai-service-agency.unity3d`, Windows as target, the mod's own assembly definition, and its `Locales` folder. Copy the current SDK example's canonical precompiled-reference list unchanged into the assembly definition, changing only its assembly name; set `overrideReferences` to `true`, `autoReferenced` to `false`, and the only define constraint to `BA_GAME_DLLS_IMPORTED`.

- [ ] **Step 2: Create the business asset in the Unity Inspector**

Create a mod-owned `BusinessType` named `AiServiceAgency`. Use localization ID `absolute-ai-service-agency:businesstype_ai_service_agency`, assign `Assets/_BaDependencies/BusinessSimulators/OfficeBusinessSimulator.asset`, and copy `suitableBuildingType`, tags, office service product, employee primary skill, and `businessRequirements` from `office-baseline.md`. Never edit the original office asset.

- [ ] **Step 3: Create retail products and business assets in the Unity Inspector**

Create five mod-owned retail `BusinessType` assets with IDs `cosmetics_store`, `pet_supply_store`, `optical_store`, `building_block_store`, and `music_store`. For all five, copy the confirmed retail simulator, suitable building type, tags, product sources, Customer Service skill, customer-demand settings, and creation requirements from `retail-baseline.md`.

Create exactly five unique, mod-owned product `Item` assets per business: Cosmetics: lipstick, foundation, perfume, skincare set, makeup brushes. Pet Supply: pet food, treats, toys, leashes, cat litter. Optical: eyeglass frames, sunglasses, lenses, contact lenses, lens solution. Building Block: construction set, architecture set, mechanical set, character set, loose bricks. Music: guitar, keyboard, drum kit, headphones, instrument accessories.

Set every retail product's item type, price, product source, and shelf/display capability from the observed compatible base-game reference products. The business asset's `businessProducts` list must contain only its own five namespaced product IDs. Do not create new retail furniture, new retail staff roles, new retail skills, custom retail customers, or retail settlement code. Do not use `Lego` in any ID, localization value, icon, asset, or visual.

- [ ] **Step 4: Create both office furniture items and prefabs**

Create matching mod-owned `Item` assets and prefabs for two racks. Set `isFurniture` true, use unique names/localization IDs, and use only compatible observed placement metadata. Localized descriptions must state `2 AI Engineer server slots` for the basic rack and `5 AI Engineer server slots` for the enterprise rack. Set price and maintenance only after Task 3's Programmer baseline is known.

- [ ] **Step 5: Implement load and unload registration using confirmed APIs**

Follow the official `ExampleBusinessTypeMod` and `ExampleFurnitureMod` lifecycle: load the AssetBundle with `AssetService.GetBundle`, load the five retail `BusinessType` assets, twenty-five retail product `Item` assets, two office furniture `Item` assets, and office `BusinessType` by asset path. Register all items before their dependent business types; unregister business types before their items in `OnUnloadAsync`. Fail load clearly for a missing AssetBundle asset. The retail assets may register only after the retail rows in Task 2 and Task 4 baseline are complete. The office business may register only after Task 2's office requirements are complete.

- [ ] **Step 6: Add localization, compile, validate, and commit**

Add `Locales/en.json` entries for all six business types, all twenty-five retail products, AI Engineer, AI skill, service fee, both office furniture names/descriptions, server-capacity status, `Hourly AI Service Fee`, and `AI Server Maintenance`. Every key begins `absolute-ai-service-agency:`. Confirm the Unity Console has no compilation errors and run `Big Ambitions/Mod Builder` validation. Commit only source and Unity assets, never `Library/`, `Temp/`, `Logs/`, imported game DLLs, or `ModsLocal` output.

```powershell
git add Assets/Mods/absolute-ai-service-agency README.md
git commit -m "feat: add business expansion Unity content"
```

### Task 6: Implement Confirmed AI-Service Economy Hooks

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

### Task 7: Build, Install, And Verify Save Safety

**Files:**
- Create: `docs/runtime/acceptance-results.md`
- Modify: `README.md`

- [ ] **Step 1: Build through the official SDK**

Use `Big Ambitions/Mod Builder > Build & Install`. Confirm validation passes and the resulting mod is present under `ModsLocal`. Do not create a manual ZIP or copy game DLLs.

- [ ] **Step 2: Test a fresh save and record evidence**

Record save name, version, and financial evidence in `docs/runtime/acceptance-results.md` for each retail business: original retail creation prerequisites with no extra unlock; all five exclusive products available through the recorded product source; successful shelf/display stocking; scheduled Customer Service employee; normal retail sales; and no changes to the original retail business. Also record: original office creation prerequisites with no extra unlock; one workstation/rack/working AI Engineer producing service revenue; a working engineer without a workstation producing no service revenue; over-capacity high-skill and ID tie allocation; unused rack daily maintenance; two enterprise racks covering ten employees; and ten enterprise racks covering fifty employees.

- [ ] **Step 3: Test existing-save non-regression**

With the mod enabled, open the mod-free baseline saves from Tasks 3 and 4. Confirm original Web Development Agency, Programmer wage, service fee, Computer Workstation behavior, retail business behavior, retail furniture behavior, Customer Service employee behavior, and retail finance entries match their baseline documents. Disable the mod and reopen both saves; confirm their original businesses load without data changes.

- [ ] **Step 4: Final source verification and commit**

```powershell
git diff --check
git status --short
git add README.md docs/runtime/acceptance-results.md
git commit -m "docs: record AI agency acceptance testing"
```

Commit acceptance evidence only after every result is observed. The published Mod Builder output inside `ModsLocal` may contain only mod-owned assemblies, bundles, localization, manifest-derived metadata, and mod-owned assets.
