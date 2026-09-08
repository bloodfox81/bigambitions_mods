# AI Service Agency Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Build a save-safe Big Ambitions AI Service Agency office business where AI Engineers earn an hourly service fee only when they have both a workstation and server capacity.

**Architecture:** Put allocation and economy rules in a game-independent .NET library. A thin mod adapter reads business state and posts income and expenses through verified game APIs. Use a BepInEx IL2CPP adapter only when the official API is verified to lack a necessary financial hook.

**Tech Stack:** C#, .NET SDK 8, `netstandard2.1`, xUnit, official Big Ambitions mod loader/API, BepInEx 6 IL2CPP only if needed.

---

## File Structure

| Path | Responsibility |
| --- | --- |
| `docs/runtime/big-ambitions-profile.md` | Game version, loader, assembly inventory, and confirmed API capabilities. |
| `src/Absolute.AiAgency.Core/` | Pure configuration, capacity, allocation, income, and maintenance rules. |
| `tests/Absolute.AiAgency.Core.Tests/` | Unit tests for every economy rule. |
| `src/Absolute.AiAgency.Mod/` | Loader entry point, game-facing content registration, and data adapter. |
| `tests/Absolute.AiAgency.Mod.Tests/` | Game adapter tests with fake facades. |
| `config/absolute.ai-service-agency.cfg.example` | Calibrated economy configuration. |
| `README.md` | Installation, compatibility, gameplay, configuration, and uninstall behavior. |

### Task 1: Verify The Installed-Game Integration Surface

**Files:**
- Create: `docs/runtime/big-ambitions-profile.md`
- Create: `docs/runtime/assembly-inventory.txt`

- [ ] **Step 1: Install Big Ambitions and create an unmodified baseline save**

Install from Steam, launch to the main menu once, create a normal save, then close the game.

- [ ] **Step 2: Record the actual executable and game version**

```powershell
$roots = @(
  "$env:ProgramFiles(x86)\Steam\steamapps\common\Big Ambitions",
  "$env:ProgramFiles\Steam\steamapps\common\Big Ambitions"
) | Where-Object { Test-Path -LiteralPath $_ }

$roots | ForEach-Object {
  Get-ChildItem -LiteralPath $_ -Filter '*.exe' -File |
    Select-Object FullName, @{ Name = 'ProductVersion'; Expression = { $_.VersionInfo.ProductVersion } }
}
```

Record resolved directory, executable, product version, checked date, and Steam build identifier if available.

- [ ] **Step 3: Inventory API assemblies without changing game files**

Run from the installed game directory:

```powershell
Get-ChildItem -LiteralPath . -Recurse -File -Include '*.dll','*.json' |
  Select-Object FullName, Length, LastWriteTime |
  Sort-Object FullName |
  Format-Table -AutoSize |
  Out-File -LiteralPath 'C:\Users\anxin\Desktop\Absolute\docs\runtime\assembly-inventory.txt' -Encoding utf8
```

Document Mono or IL2CPP, mod loader path, API assemblies, and the exact method signatures for: business registration, role registration, furniture registration, furniture lookup, hourly revenue, and daily expense.

- [ ] **Step 4: Select the supported integration path**

| Observed capability | Decision |
| --- | --- |
| Official API supports all six requirements | Use only the official API. |
| Official API lacks only revenue or expense posting | Use official content APIs plus a single isolated BepInEx adapter. |
| Official API lacks business, role, or furniture registration | Stop and redesign to an official-API-compatible extension. |

- [ ] **Step 5: Commit the compatibility profile**

```powershell
git add docs/runtime/big-ambitions-profile.md docs/runtime/assembly-inventory.txt
git commit -m "docs: record Big Ambitions compatibility profile"
```

### Task 2: Scaffold The Economy Core With Tests

**Files:**
- Create: `Absolute.AiAgency.sln`
- Create: `src/Absolute.AiAgency.Core/Absolute.AiAgency.Core.csproj`
- Create: `src/Absolute.AiAgency.Core/IsExternalInit.cs`
- Create: `tests/Absolute.AiAgency.Core.Tests/Absolute.AiAgency.Core.Tests.csproj`
- Create: `src/Absolute.AiAgency.Core/EconomyConfiguration.cs`
- Create: `tests/Absolute.AiAgency.Core.Tests/ConfigurationTests.cs`

- [ ] **Step 1: Create projects**

```powershell
dotnet new sln --name Absolute.AiAgency
dotnet new classlib --framework netstandard2.1 --name Absolute.AiAgency.Core --output src/Absolute.AiAgency.Core
dotnet new xunit --framework net8.0 --name Absolute.AiAgency.Core.Tests --output tests/Absolute.AiAgency.Core.Tests
dotnet sln Absolute.AiAgency.sln add src/Absolute.AiAgency.Core/Absolute.AiAgency.Core.csproj tests/Absolute.AiAgency.Core.Tests/Absolute.AiAgency.Core.Tests.csproj
dotnet add tests/Absolute.AiAgency.Core.Tests/Absolute.AiAgency.Core.Tests.csproj reference src/Absolute.AiAgency.Core/Absolute.AiAgency.Core.csproj
```

- [ ] **Step 2: Write failing configuration tests**

```csharp
using Absolute.AiAgency.Core;
using Xunit;

namespace Absolute.AiAgency.Core.Tests;

public sealed class ConfigurationTests
{
    [Fact]
    public void DefaultConfiguration_UsesTwoAndFiveSlotRacks()
    {
        var configuration = EconomyConfiguration.Default;
        Assert.Equal(2, configuration.BasicRackSlots);
        Assert.Equal(5, configuration.EnterpriseRackSlots);
    }

    [Theory]
    [InlineData(0, 5)]
    [InlineData(2, 0)]
    public void Validate_RejectsNonPositiveRackCapacity(int basicSlots, int enterpriseSlots)
    {
        var configuration = EconomyConfiguration.Default with
        {
            BasicRackSlots = basicSlots,
            EnterpriseRackSlots = enterpriseSlots,
        };
        Assert.False(configuration.Validate().IsValid);
    }

    [Fact]
    public void Validate_RejectsAZeroSkillFeeMultiplier()
    {
        var configuration = EconomyConfiguration.Default with { FeePerSkillLevel = 0m };
        Assert.False(configuration.Validate().IsValid);
    }
}
```

- [ ] **Step 3: Verify tests fail before implementation**

Run: `dotnet test tests/Absolute.AiAgency.Core.Tests/Absolute.AiAgency.Core.Tests.csproj --no-restore`

Expected: compilation fails because `EconomyConfiguration` does not exist.

- [ ] **Step 4: Implement immutable configuration**

```csharp
// src/Absolute.AiAgency.Core/IsExternalInit.cs
namespace System.Runtime.CompilerServices;

internal static class IsExternalInit
{
}
```

```csharp
// src/Absolute.AiAgency.Core/EconomyConfiguration.cs
namespace Absolute.AiAgency.Core;

public sealed record EconomyConfiguration(
    int BasicRackSlots,
    int EnterpriseRackSlots,
    decimal BasicRackDailyMaintenance,
    decimal EnterpriseRackDailyMaintenance,
    decimal BaseHourlyServiceFee,
    decimal FeePerSkillLevel,
    decimal AiEngineerHourlyWage)
{
    public static EconomyConfiguration Default { get; } = new(2, 5, 0m, 0m, 0m, 0m, 0m);

    public ValidationResult Validate() => new(
        BasicRackSlots > 0 && EnterpriseRackSlots > 0 &&
        BasicRackDailyMaintenance >= 0m && EnterpriseRackDailyMaintenance >= 0m &&
        BaseHourlyServiceFee >= 0m && FeePerSkillLevel > 0m && AiEngineerHourlyWage >= 0m);
}

public sealed record ValidationResult(bool IsValid);
```

Leave money defaults at zero until Task 7 measures existing programmer values.

- [ ] **Step 5: Verify and commit**

```powershell
dotnet test tests/Absolute.AiAgency.Core.Tests/Absolute.AiAgency.Core.Tests.csproj
git add Absolute.AiAgency.sln src/Absolute.AiAgency.Core tests/Absolute.AiAgency.Core.Tests
git commit -m "feat: scaffold AI agency economy core"
```

### Task 3: Implement Server Capacity And Deterministic Allocation

**Files:**
- Create: `src/Absolute.AiAgency.Core/ServerCapacity.cs`
- Create: `src/Absolute.AiAgency.Core/SlotAllocator.cs`
- Create: `tests/Absolute.AiAgency.Core.Tests/SlotAllocatorTests.cs`

- [ ] **Step 1: Write failing allocation tests**

```csharp
using Absolute.AiAgency.Core;
using Xunit;

namespace Absolute.AiAgency.Core.Tests;

public sealed class SlotAllocatorTests
{
    [Fact]
    public void Allocate_SelectsHighestSkillEngineersFirst()
    {
        var engineers = new[]
        {
            new WorkingEngineer("low", 1, true, true),
            new WorkingEngineer("high", 5, true, true),
            new WorkingEngineer("middle", 3, true, true),
        };

        var allocated = new SlotAllocator().Allocate(engineers, 2);

        Assert.Equal(new[] { "high", "middle" }, allocated.Select(x => x.EmployeeId));
    }

    [Fact]
    public void Allocate_UsesEmployeeIdAsStableTieBreaker()
    {
        var engineers = new[]
        {
            new WorkingEngineer("employee-b", 4, true, true),
            new WorkingEngineer("employee-a", 4, true, true),
        };

        Assert.Equal("employee-a", Assert.Single(new SlotAllocator().Allocate(engineers, 1)).EmployeeId);
    }

    [Fact]
    public void Capacity_CoversTenAndFiftySeatsWithoutRemainder()
    {
        var capacity = new ServerCapacity(2, 5);

        Assert.Equal(10, capacity.FromRacks(0, 2));
        Assert.Equal(50, capacity.FromRacks(0, 10));
    }
}
```

- [ ] **Step 2: Verify tests fail**

Run: `dotnet test tests/Absolute.AiAgency.Core.Tests/Absolute.AiAgency.Core.Tests.csproj --filter FullyQualifiedName~SlotAllocatorTests`

Expected: compilation fails because `WorkingEngineer`, `SlotAllocator`, and `ServerCapacity` do not exist.

- [ ] **Step 3: Implement capacity and allocation**

```csharp
namespace Absolute.AiAgency.Core;

public sealed record WorkingEngineer(string EmployeeId, int AiSkill, bool IsWorking, bool HasWorkstation);

public sealed class ServerCapacity
{
    private readonly int _basicRackSlots;
    private readonly int _enterpriseRackSlots;

    public ServerCapacity(int basicRackSlots, int enterpriseRackSlots)
    {
        _basicRackSlots = basicRackSlots;
        _enterpriseRackSlots = enterpriseRackSlots;
    }

    public int FromRacks(int basicRackCount, int enterpriseRackCount) =>
        checked((basicRackCount * _basicRackSlots) + (enterpriseRackCount * _enterpriseRackSlots));
}

public sealed class SlotAllocator
{
    public IReadOnlyList<WorkingEngineer> Allocate(IEnumerable<WorkingEngineer> engineers, int availableSlots) =>
        engineers
            .Where(engineer => engineer.IsWorking && engineer.HasWorkstation)
            .OrderByDescending(engineer => engineer.AiSkill)
            .ThenBy(engineer => engineer.EmployeeId, StringComparer.Ordinal)
            .Take(Math.Max(0, availableSlots))
            .ToArray();
}
```

- [ ] **Step 4: Verify and commit**

```powershell
dotnet test Absolute.AiAgency.sln
git add src/Absolute.AiAgency.Core/ServerCapacity.cs src/Absolute.AiAgency.Core/SlotAllocator.cs tests/Absolute.AiAgency.Core.Tests/SlotAllocatorTests.cs
git commit -m "feat: allocate AI server capacity by skill"
```

### Task 4: Implement Income Eligibility And Rack Maintenance

**Files:**
- Create: `src/Absolute.AiAgency.Core/AgencyEconomyCalculator.cs`
- Create: `tests/Absolute.AiAgency.Core.Tests/AgencyEconomyCalculatorTests.cs`

- [ ] **Step 1: Write failing revenue and maintenance tests**

```csharp
using Absolute.AiAgency.Core;
using Xunit;

namespace Absolute.AiAgency.Core.Tests;

public sealed class AgencyEconomyCalculatorTests
{
    [Fact]
    public void CalculatePeriod_OnlyAllocatedEngineersGenerateIncome()
    {
        var configuration = EconomyConfiguration.Default with { BaseHourlyServiceFee = 100m, FeePerSkillLevel = 10m };
        var engineers = new[]
        {
            new WorkingEngineer("high", 4, true, true),
            new WorkingEngineer("low", 1, true, true),
            new WorkingEngineer("no-desk", 5, true, false),
        };

        var result = new AgencyEconomyCalculator(configuration).CalculatePeriod(engineers, 1, 1m);

        Assert.Equal(140m, result.ServiceRevenue);
        Assert.Equal(new[] { "high" }, result.AllocatedEmployeeIds);
    }

    [Fact]
    public void CalculateDailyMaintenance_ChargesUnusedRacks()
    {
        var configuration = EconomyConfiguration.Default with { BasicRackDailyMaintenance = 20m, EnterpriseRackDailyMaintenance = 40m };

        Assert.Equal(80m, new AgencyEconomyCalculator(configuration).CalculateDailyMaintenance(2, 1));
    }
}
```

- [ ] **Step 2: Verify tests fail**

Run: `dotnet test tests/Absolute.AiAgency.Core.Tests/Absolute.AiAgency.Core.Tests.csproj --filter FullyQualifiedName~AgencyEconomyCalculatorTests`

Expected: compilation fails because `AgencyEconomyCalculator` does not exist.

- [ ] **Step 3: Implement the calculator**

```csharp
namespace Absolute.AiAgency.Core;

public sealed record IncomePeriodResult(decimal ServiceRevenue, IReadOnlyList<string> AllocatedEmployeeIds);

public sealed class AgencyEconomyCalculator
{
    private readonly EconomyConfiguration _configuration;
    private readonly SlotAllocator _slotAllocator = new();

    public AgencyEconomyCalculator(EconomyConfiguration configuration) => _configuration = configuration;

    public IncomePeriodResult CalculatePeriod(IEnumerable<WorkingEngineer> engineers, int availableSlots, decimal hours)
    {
        var allocated = _slotAllocator.Allocate(engineers, availableSlots);
        var revenue = allocated.Sum(engineer =>
            (_configuration.BaseHourlyServiceFee + (_configuration.FeePerSkillLevel * engineer.AiSkill)) * hours);
        return new IncomePeriodResult(revenue, allocated.Select(engineer => engineer.EmployeeId).ToArray());
    }

    public decimal CalculateDailyMaintenance(int basicRackCount, int enterpriseRackCount) =>
        (basicRackCount * _configuration.BasicRackDailyMaintenance) +
        (enterpriseRackCount * _configuration.EnterpriseRackDailyMaintenance);
}
```

- [ ] **Step 4: Verify and commit**

```powershell
dotnet test Absolute.AiAgency.sln --configuration Release
git add src/Absolute.AiAgency.Core/AgencyEconomyCalculator.cs tests/Absolute.AiAgency.Core.Tests/AgencyEconomyCalculatorTests.cs
git commit -m "feat: calculate AI agency revenue and maintenance"
```

### Task 5: Add A Game-Neutral Settlement Boundary

**Files:**
- Create: `src/Absolute.AiAgency.Core/AgencySettlementService.cs`
- Create: `tests/Absolute.AiAgency.Core.Tests/AgencySettlementServiceTests.cs`

- [ ] **Step 1: Write failing settlement tests with a fake game ledger**

```csharp
using Absolute.AiAgency.Core;
using Xunit;

namespace Absolute.AiAgency.Core.Tests;

public sealed class AgencySettlementServiceTests
{
    [Fact]
    public void SettleIncomePeriod_PostsOnlyAllocatedServiceRevenueToOwningAgency()
    {
        var configuration = EconomyConfiguration.Default with
        {
            BaseHourlyServiceFee = 100m,
            FeePerSkillLevel = 10m,
        };
        var gateway = new RecordingGateway();
        var snapshot = new AgencySnapshot(
            "agency-42", 1, 0,
            new[]
            {
                new WorkingEngineer("engineer-high", 5, true, true),
                new WorkingEngineer("engineer-low", 1, true, true),
                new WorkingEngineer("engineer-no-desk", 9, true, false),
            });

        var result = new AgencySettlementService(configuration).SettleIncomePeriod(snapshot, 1m, gateway);

        Assert.Equal(260m, result.ServiceRevenue);
        Assert.Equal(new[] { "engineer-high", "engineer-low" }, result.AllocatedEmployeeIds);
        Assert.Equal(new LedgerEntry("agency-42", "Hourly AI Service Fee", 260m), Assert.Single(gateway.Revenue));
        Assert.Empty(gateway.Expense);
    }

    [Fact]
    public void SettleDailyMaintenance_PostsCostsForEveryRackIncludingUnusedCapacity()
    {
        var configuration = EconomyConfiguration.Default with
        {
            BasicRackDailyMaintenance = 20m,
            EnterpriseRackDailyMaintenance = 40m,
        };
        var gateway = new RecordingGateway();
        var snapshot = new AgencySnapshot("agency-42", 2, 1, Array.Empty<WorkingEngineer>());

        var maintenance = new AgencySettlementService(configuration).SettleDailyMaintenance(snapshot, gateway);

        Assert.Equal(80m, maintenance);
        Assert.Equal(new LedgerEntry("agency-42", "AI Server Maintenance", 80m), Assert.Single(gateway.Expense));
    }

    private sealed class RecordingGateway : IGameAgencyGateway
    {
        public List<LedgerEntry> Revenue { get; } = new();
        public List<LedgerEntry> Expense { get; } = new();

        public void RecordServiceRevenue(string businessId, string lineItem, decimal amount) =>
            Revenue.Add(new LedgerEntry(businessId, lineItem, amount));

        public void RecordOperatingExpense(string businessId, string lineItem, decimal amount) =>
            Expense.Add(new LedgerEntry(businessId, lineItem, amount));
    }
}
```

- [ ] **Step 2: Verify tests fail**

Run: `dotnet test tests/Absolute.AiAgency.Core.Tests/Absolute.AiAgency.Core.Tests.csproj --filter FullyQualifiedName~AgencySettlementServiceTests`

Expected: compilation fails because `AgencySnapshot`, `IGameAgencyGateway`, `LedgerEntry`, and `AgencySettlementService` do not exist.

- [ ] **Step 3: Implement the settlement contract and service**

```csharp
namespace Absolute.AiAgency.Core;

public sealed record AgencySnapshot(
    string BusinessId,
    int BasicRackCount,
    int EnterpriseRackCount,
    IReadOnlyList<WorkingEngineer> Engineers);

public sealed record LedgerEntry(string BusinessId, string LineItem, decimal Amount);

public interface IGameAgencyGateway
{
    void RecordServiceRevenue(string businessId, string lineItem, decimal amount);
    void RecordOperatingExpense(string businessId, string lineItem, decimal amount);
}

public sealed class AgencySettlementService
{
    private const string ServiceFeeLineItem = "Hourly AI Service Fee";
    private const string MaintenanceLineItem = "AI Server Maintenance";
    private readonly AgencyEconomyCalculator _calculator;
    private readonly ServerCapacity _serverCapacity;

    public AgencySettlementService(EconomyConfiguration configuration)
    {
        _calculator = new AgencyEconomyCalculator(configuration);
        _serverCapacity = new ServerCapacity(configuration.BasicRackSlots, configuration.EnterpriseRackSlots);
    }

    public IncomePeriodResult SettleIncomePeriod(AgencySnapshot snapshot, decimal hours, IGameAgencyGateway gateway)
    {
        var slots = _serverCapacity.FromRacks(snapshot.BasicRackCount, snapshot.EnterpriseRackCount);
        var result = _calculator.CalculatePeriod(snapshot.Engineers, slots, hours);
        if (result.ServiceRevenue > 0m)
        {
            gateway.RecordServiceRevenue(snapshot.BusinessId, ServiceFeeLineItem, result.ServiceRevenue);
        }

        return result;
    }

    public decimal SettleDailyMaintenance(AgencySnapshot snapshot, IGameAgencyGateway gateway)
    {
        var maintenance = _calculator.CalculateDailyMaintenance(snapshot.BasicRackCount, snapshot.EnterpriseRackCount);
        if (maintenance > 0m)
        {
            gateway.RecordOperatingExpense(snapshot.BusinessId, MaintenanceLineItem, maintenance);
        }

        return maintenance;
    }
}
```

- [ ] **Step 4: Verify and commit**

```powershell
dotnet test Absolute.AiAgency.sln --configuration Release
git add src/Absolute.AiAgency.Core/AgencySettlementService.cs tests/Absolute.AiAgency.Core.Tests/AgencySettlementServiceTests.cs
git commit -m "feat: add AI agency settlement boundary"
```

### Task 6: Bind The Core To Verified Big Ambitions APIs

**Files:**
- Create: `src/Absolute.AiAgency.Mod/Absolute.AiAgency.Mod.csproj`
- Create: `src/Absolute.AiAgency.Mod/AiServiceAgencyPlugin.cs`
- Create: `src/Absolute.AiAgency.Mod/AiAgencyContentRegistrar.cs`
- Create: `src/Absolute.AiAgency.Mod/BigAmbitionsAgencyGateway.cs`
- Create: `src/Absolute.AiAgency.Mod/AiAgencySaveData.cs`
- Create: `tests/Absolute.AiAgency.Mod.Tests/Absolute.AiAgency.Mod.Tests.csproj`
- Create: `tests/Absolute.AiAgency.Mod.Tests/BigAmbitionsAgencyGatewayTests.cs`
- Modify: `Absolute.AiAgency.sln`
- Modify: `docs/runtime/big-ambitions-profile.md`

- [ ] **Step 1: Stop unless the compatibility profile has all required bindings**

Read `docs/runtime/big-ambitions-profile.md` and require an exact public type, assembly path, and method signature for each binding in this table before creating `Absolute.AiAgency.Mod.csproj`:

| Required binding | Required recorded behavior |
| --- | --- |
| Mod entry point | Loads a mod-owned assembly and receives the game version. |
| Business registration | Registers a new business without replacing a base-game definition and records the same creation prerequisites as an original office business. |
| Employee role registration | Registers `AI Engineer` and its AI skill. |
| Furniture registration | Registers both racks and exposes their placed owning business. |
| Workstation lookup | Reports whether a working engineer has a normal base-game Computer Workstation. |
| Periodic callback | Runs once for the verified office-income period. |
| Financial ledger | Posts revenue and operating expense to a specific business. |
| Mod save data | Reads and writes a namespaced record without serializing base-game objects. |

Use the Task 1 decision table: use only the official API when all bindings exist; add one isolated BepInEx IL2CPP layer only when a financial callback or ledger binding is the single missing capability; stop implementation when business, role, or furniture registration is unsupported.

- [ ] **Step 2: Write failing ledger-adapter tests**

```csharp
using Absolute.AiAgency.Core;
using Absolute.AiAgency.Mod;
using Xunit;

namespace Absolute.AiAgency.Mod.Tests;

public sealed class BigAmbitionsAgencyGatewayTests
{
    [Fact]
    public void RecordServiceRevenue_ForwardsBusinessIdDescriptionAndAmount()
    {
        var ledger = new RecordingBusinessLedger();
        var gateway = new BigAmbitionsAgencyGateway(ledger);

        gateway.RecordServiceRevenue("agency-42", "Hourly AI Service Fee", 250m);

        Assert.Equal(new LedgerEntry("agency-42", "Hourly AI Service Fee", 250m), Assert.Single(ledger.Revenue));
    }

    [Fact]
    public void RecordOperatingExpense_ForwardsBusinessIdDescriptionAndAmount()
    {
        var ledger = new RecordingBusinessLedger();
        var gateway = new BigAmbitionsAgencyGateway(ledger);

        gateway.RecordOperatingExpense("agency-42", "AI Server Maintenance", 40m);

        Assert.Equal(new LedgerEntry("agency-42", "AI Server Maintenance", 40m), Assert.Single(ledger.Expense));
    }

    private sealed class RecordingBusinessLedger : IBusinessLedger
    {
        public List<LedgerEntry> Revenue { get; } = new();
        public List<LedgerEntry> Expense { get; } = new();

        public void AddRevenue(string businessId, string description, decimal amount) =>
            Revenue.Add(new LedgerEntry(businessId, description, amount));

        public void AddExpense(string businessId, string description, decimal amount) =>
            Expense.Add(new LedgerEntry(businessId, description, amount));
    }
}
```

- [ ] **Step 3: Verify adapter tests fail**

Run: `dotnet test tests/Absolute.AiAgency.Mod.Tests/Absolute.AiAgency.Mod.Tests.csproj --filter FullyQualifiedName~BigAmbitionsAgencyGatewayTests`

Expected: compilation fails because the mod project, `IBusinessLedger`, and `BigAmbitionsAgencyGateway` do not exist.

- [ ] **Step 4: Implement and verify the testable ledger boundary**

```csharp
using Absolute.AiAgency.Core;

namespace Absolute.AiAgency.Mod;

public interface IBusinessLedger
{
    void AddRevenue(string businessId, string description, decimal amount);
    void AddExpense(string businessId, string description, decimal amount);
}

public sealed class BigAmbitionsAgencyGateway : IGameAgencyGateway
{
    private readonly IBusinessLedger _ledger;

    public BigAmbitionsAgencyGateway(IBusinessLedger ledger) => _ledger = ledger;

    public void RecordServiceRevenue(string businessId, string lineItem, decimal amount) =>
        _ledger.AddRevenue(businessId, lineItem, amount);

    public void RecordOperatingExpense(string businessId, string lineItem, decimal amount) =>
        _ledger.AddExpense(businessId, lineItem, amount);
}
```

Run: `dotnet test tests/Absolute.AiAgency.Mod.Tests/Absolute.AiAgency.Mod.Tests.csproj --filter FullyQualifiedName~BigAmbitionsAgencyGatewayTests`

Expected: two passing tests.

- [ ] **Step 5: Implement only the profile-recorded game bindings**

Implement the entry point and registrar from the exact types and signatures in the completed profile. Register precisely these mod-owned identifiers:

```text
absolute.ai-service-agency.business
absolute.ai-engineer.role
absolute.hourly-ai-service-fee.service
absolute.ai-server-rack.furniture
absolute.enterprise-ai-server-rack.furniture
```

On each verified income callback, construct one `AgencySnapshot` per AI Service Agency from the current game state and call `AgencySettlementService.SettleIncomePeriod`. On the verified day rollover callback, call `AgencySettlementService.SettleDailyMaintenance` once per agency. Resolve rack ownership and workstations from current game objects on each callback.

At load, log the detected game version and each resolved binding. Register the AI Service Agency with the identical creation condition recorded for the selected original office business in Task 1; do not add an independent unlock gate. If any required binding is absent, show the compatibility error, do not expose the AI Service Agency creation option, and do not change any base-game content. Do not reference or patch Programmer, Web Development Agency, Computer Workstation, or any existing business identifier.

- [ ] **Step 6: Add a namespaced, minimal save-data record**

```csharp
namespace Absolute.AiAgency.Mod;

public sealed record AiAgencySaveData(int SchemaVersion);

public static class AiAgencySaveDataSchema
{
    public const string ModDataKey = "absolute.ai-service-agency";
    public const int CurrentSchemaVersion = 1;

    public static AiAgencySaveData CreateNew() => new(CurrentSchemaVersion);
}
```

Store only `AiAgencySaveData` below `absolute.ai-service-agency`. Do not serialize copied game business, employee, or furniture objects. When the schema is unreadable or newer than `CurrentSchemaVersion`, skip settlement for the affected agency, log its business ID, and leave all game-owned data unmodified.

- [ ] **Step 7: Build, test, and commit after the profile-backed implementation**

```powershell
dotnet test Absolute.AiAgency.sln --configuration Release
dotnet build Absolute.AiAgency.sln --configuration Release
git add Absolute.AiAgency.sln src/Absolute.AiAgency.Mod tests/Absolute.AiAgency.Mod.Tests docs/runtime/big-ambitions-profile.md
git commit -m "feat: add Big Ambitions AI agency adapter"
```

### Task 7: Calibrate The Economy And Write User Configuration

**Files:**
- Create: `docs/runtime/programmer-baseline.md`
- Create: `config/absolute.ai-service-agency.cfg.example`
- Create: `README.md`
- Modify: `src/Absolute.AiAgency.Core/EconomyConfiguration.cs`
- Modify: `tests/Absolute.AiAgency.Core.Tests/ConfigurationTests.cs`

- [ ] **Step 1: Record a mod-free Programmer baseline**

Create a Web Development Agency in a separate, unmodified save. Hire and schedule one Programmer with one normal Computer Workstation. Advance through one office-income period and record the game version, Programmer hourly wage, `Hourly Programmer Fee`, actual interval duration, and financial-entry labels in `docs/runtime/programmer-baseline.md`. Do not run the mod in this save.

- [ ] **Step 2: Write failing calibration tests**

```csharp
[Fact]
public void ValidateAgainstProgrammerBaseline_RejectsEqualWageOrServiceFee()
{
    var configuration = EconomyConfiguration.Default with
    {
        AiEngineerHourlyWage = 100m,
        BaseHourlyServiceFee = 100m,
        FeePerSkillLevel = 1m,
    };

    Assert.False(configuration.ValidateAgainstProgrammerBaseline(100m, 100m).IsValid);
}

[Fact]
public void ValidateAgainstProgrammerBaseline_AcceptsHigherWageAndServiceFee()
{
    var configuration = EconomyConfiguration.Default with
    {
        AiEngineerHourlyWage = 101m,
        BaseHourlyServiceFee = 101m,
        FeePerSkillLevel = 1m,
    };

    Assert.True(configuration.ValidateAgainstProgrammerBaseline(100m, 100m).IsValid);
}
```

- [ ] **Step 3: Verify the tests fail**

Run: `dotnet test tests/Absolute.AiAgency.Core.Tests/Absolute.AiAgency.Core.Tests.csproj --filter FullyQualifiedName~ConfigurationTests`

Expected: compilation fails because `ValidateAgainstProgrammerBaseline` does not exist.

- [ ] **Step 4: Implement strict baseline comparison**

Add this method to `EconomyConfiguration`:

```csharp
public ValidationResult ValidateAgainstProgrammerBaseline(decimal programmerHourlyWage, decimal programmerHourlyFee) =>
    new(
        Validate().IsValid &&
        programmerHourlyWage >= 0m &&
        programmerHourlyFee >= 0m &&
        AiEngineerHourlyWage > programmerHourlyWage &&
        BaseHourlyServiceFee > programmerHourlyFee);
```

- [ ] **Step 5: Enter measured configuration values and prove the arithmetic**

Populate `config/absolute.ai-service-agency.cfg.example` with decimals measured in Step 1. The configuration must use these exact keys and satisfy every stated inequality:

```ini
[economy]
ai_engineer_hourly_wage = 0
base_hourly_service_fee = 0
fee_per_ai_skill_level = 0
ai_server_rack_slots = 2
enterprise_ai_server_rack_slots = 5
ai_server_rack_daily_maintenance = 0
enterprise_ai_server_rack_daily_maintenance = 0
```

Before committing, replace the five zero-valued economic entries with measured values and check:

```text
AI Engineer hourly wage > Programmer hourly wage
Base hourly AI Service Fee > Hourly Programmer Fee
Fee per AI skill level > 0
AI Server Rack daily maintenance = 2 * AI Engineer hourly wage
Enterprise AI Server Rack daily maintenance > AI Server Rack daily maintenance
Enterprise AI Server Rack daily maintenance / 5 < AI Server Rack daily maintenance / 2
```

- [ ] **Step 6: Write user-facing installation and compatibility documentation**

`README.md` must document the exact supported game version, mod-loader version, DLL installation path, configuration path, namespaced content IDs, agency creation flow, 2/5 server-slot rule, workstation requirement, skill-priority allocation, daily maintenance, financial line items, and mod-removal behavior. State clearly that original office businesses, Programmers, Computer Workstations, and their financial calculations are not changed.

- [ ] **Step 7: Verify and commit**

```powershell
dotnet test Absolute.AiAgency.sln --configuration Release
git add docs/runtime/programmer-baseline.md config/absolute.ai-service-agency.cfg.example README.md src/Absolute.AiAgency.Core/EconomyConfiguration.cs tests/Absolute.AiAgency.Core.Tests/ConfigurationTests.cs
git commit -m "docs: calibrate AI agency economy"
```

### Task 8: Run Save-Safety And Gameplay Acceptance Tests

**Files:**
- Create: `docs/runtime/acceptance-results.md`
- Create: `dist/absolute-ai-service-agency.zip`

- [ ] **Step 1: Verify a clean source tree, tests, and release build**

```powershell
git status --short
dotnet test Absolute.AiAgency.sln --configuration Release
dotnet build src/Absolute.AiAgency.Mod/Absolute.AiAgency.Mod.csproj --configuration Release
```

Expected: `git status --short` prints no tracked or untracked changes before the build, tests report no failures, and build output identifies the mod DLL path.

- [ ] **Step 2: Record fresh-save evidence for all gameplay rules**

Create `docs/runtime/acceptance-results.md` with this complete matrix. Fill the Result and Evidence columns with observed game behavior, save names, and finance-log references while testing.

| Scenario | Expected result | Result | Evidence |
| --- | --- | --- | --- |
| Create AI Service Agency in an office | Same recorded office creation prerequisite is used; no extra unlock gate |  |  |
| Place one Computer Workstation and one base rack | Two slots are available to that agency only |  |  |
| Schedule one AI Engineer with a workstation | One `Hourly AI Service Fee` entry posts for each income period |  |  |
| Remove the workstation during work | Service revenue is absent for that employee |  |  |
| Exceed server capacity | Highest AI skill receives slots; employee ID breaks equal-skill ties |  |  |
| Place unused base and enterprise racks | Daily maintenance is charged for every rack |  |  |
| Use two enterprise racks | Ten engineers can be allocated with no remainder |  |  |
| Use ten enterprise racks | Fifty engineers can be allocated with no remainder |  |  |

- [ ] **Step 3: Test the existing-save non-regression boundary**

Open the unmodified baseline save from Task 1 with the mod enabled. Confirm that Web Development Agency revenue, Programmer wages, and normal Computer Workstation behavior have the same values as documented in `docs/runtime/programmer-baseline.md`. Close the game, disable the mod, reopen the same save, and confirm its existing base-game businesses load without data changes.

- [ ] **Step 4: Package only mod-owned files**

Create the archive from the only release DLL produced by Task 8 Step 1. The `$modDll` assertion prevents packaging an old or ambiguous build artifact.

```powershell
$packageRoot = Join-Path (Get-Location) 'dist\package-root'
$modDll = @(Get-ChildItem -LiteralPath 'src\Absolute.AiAgency.Mod\bin\Release' -Recurse -File -Filter 'Absolute.AiAgency.Mod.dll')
if ($modDll.Count -ne 1) { throw "Expected one release mod DLL, found $($modDll.Count)." }
New-Item -ItemType Directory -Force -Path $packageRoot | Out-Null
Copy-Item -LiteralPath 'README.md' -Destination $packageRoot
Copy-Item -LiteralPath 'config\absolute.ai-service-agency.cfg.example' -Destination $packageRoot
Copy-Item -LiteralPath $modDll[0].FullName -Destination $packageRoot
Compress-Archive -LiteralPath "$packageRoot\*" -DestinationPath 'dist\absolute-ai-service-agency.zip' -Force
```

Inspect the ZIP before publication. It may contain the mod DLL, required mod-owned dependencies, the example configuration, and README. It must not contain game assemblies, game assets, executable files, or save data.

- [ ] **Step 5: Commit acceptance documentation, not compiled output**

```powershell
git add docs/runtime/acceptance-results.md README.md config/absolute.ai-service-agency.cfg.example
git commit -m "docs: record AI agency acceptance testing"
```

Do not commit `dist/` when it contains compiled output. Publish the inspected ZIP through the selected mod distribution channel.
