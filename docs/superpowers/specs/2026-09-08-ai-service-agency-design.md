# AI Service Agency Mod Design

## Goal

Add an AI-focused office business to Big Ambitions without changing existing office businesses. The new business must use the same core loop as the base game: qualified employees work at computer workstations and automatically generate an hourly service fee.

## Scope

The mod adds the following content:

- `AI Service Agency`, a new office business type.
- `AI Engineer`, a new employee role with an AI skill.
- `Hourly AI Service Fee`, the agency's only sellable service.
- `AI Server Rack`, a server furniture item with two AI Engineer slots.
- `Enterprise AI Server Rack`, a server furniture item with five AI Engineer slots.
- A daily fixed maintenance charge for every placed server rack.

The mod does not alter the behavior, employee roles, prices, or revenue of the base game's law firms, graphic design firms, web development agencies, programmers, or other office businesses.

## Player Loop

1. The player rents an office and creates an `AI Service Agency` using the same business-creation flow as base-game office businesses.
2. The player places enough base-game `Computer Workstation` furniture for the desired AI Engineer headcount.
3. The player purchases and places server racks inside the agency's office.
4. The player hires and schedules `AI Engineer` employees.
5. During each income period, eligible AI Engineers automatically generate `Hourly AI Service Fee` revenue.
6. Server maintenance is charged once per in-game day, including for unused slots.
7. The player grows the business by improving employee skill, adding workstations, and adding server capacity in two- or five-slot increments.

## Revenue Eligibility

An AI Engineer generates `Hourly AI Service Fee` only when all of these conditions are true:

1. The employee is scheduled and currently working.
2. The employee has a valid base-game `Computer Workstation` assigned by the normal office-business rules.
3. The employee has received an AI server slot for the current income period.

Employees who fail any condition remain normal employees: they receive their scheduled wage but generate no AI-service revenue for that period.

## Server Capacity And Allocation

Server capacity is shared by each individual `AI Service Agency`. Only racks placed inside that agency's office count toward its capacity.

| Furniture | AI Engineer slots | Purpose |
| --- | ---: | --- |
| `AI Server Rack` | 2 | Incremental capacity for small and medium offices. |
| `Enterprise AI Server Rack` | 5 | Space-efficient capacity for larger offices. |

The `2`- and `5`-slot sizes evenly fit the base-game office capacity progression of 10 and 50. This lets fully staffed offices reach exact server coverage without a forced leftover slot caused by rack sizing.

At each income period, the mod:

1. Collects working AI Engineers who have valid computer workstations.
2. Sorts them by AI skill from highest to lowest.
3. Assigns available server slots in that order.
4. Marks the first employees within total server capacity as revenue eligible.

Employees with equal AI skill use a stable deterministic tie-breaker based on their persistent employee identifier. This prevents allocation from changing unpredictably between income periods.

## Skill And Pricing Model

`AI Engineer` uses a dedicated AI skill. The skill has two effects:

- Higher-skill engineers receive server slots first when capacity is scarce.
- Higher-skill engineers earn a higher `Hourly AI Service Fee` when assigned a slot.

The default AI Engineer wage and the default AI service fee are both set above the base-game programmer equivalents. This preserves the intended high-investment, high-return position: an AI agency needs expensive staff and server equipment, but each fully provisioned engineer can outperform a standard programmer.

All tunable economic values are exposed through mod configuration, including employee wages, service-fee base rate, skill multiplier, rack purchase price, and daily maintenance. This permits balancing against the installed game version without changing source code.

## Daily Maintenance

Each placed rack incurs a fixed charge once per in-game day:

- The base rack's default maintenance is approximately equal to two hours of an AI Engineer's wage.
- The enterprise rack has a higher total daily maintenance cost but a lower maintenance cost per supported engineer.
- Maintenance is charged whether or not its slots were allocated that day.

The maintenance charge is recorded as an operating expense of the owning `AI Service Agency`, so it appears with other business costs rather than as a separate player-level penalty.

## UI And Feedback

The agency uses the base game's office-management and financial presentation wherever available. The mod adds only the information players need to understand the new rule:

- Agency summary: working AI Engineers, valid workstation count, server capacity, allocated server slots, and unallocated working engineers.
- Employee status: whether an AI Engineer is currently revenue eligible or waiting for server capacity.
- Financial entries: `Hourly AI Service Fee` revenue and daily server-maintenance expense.
- Furniture descriptions: supported slot count, purchase price, and daily maintenance cost.

No separate subscription, contract, customer queue, or manual server-assignment screen is included in the first release.

## Technical Boundary

The mod uses only Hovgaard Games' official Unity Modding SDK and the public APIs imported from the installed Big Ambitions game. The Unity project must use `2022.3.62f2`; content lives below `Assets/Mods/<mod-id>/`, is compiled through that SDK, and is installed with `Big Ambitions/Mod Builder` into `ModsLocal`.

The public SDK verifies registration paths for mod-owned `BusinessType` assets and furniture `Item` assets. The AI Service Agency must reuse the base-game `OfficeBusinessSimulator`, and its creation requirements must be copied from the selected original office-business asset after the game DLLs have been imported. It must not modify existing business, Programmer, or Computer Workstation assets.

The installed-game API must still be inspected before implementation to confirm supported registration or extension points for an employee role, a dedicated AI skill, an hourly office-service product, periodic settlement, daily business expenses, and optional business-management UI. The mod must not use BepInEx, runtime patches, reflection-based private-member access, or other unsupported loaders as a fallback. If a required public API is absent, the unavailable design element is removed or redesigned before release; no partially functional business is exposed to players.

The mod must not redistribute game assemblies, base-game assets, or save files. It uses a unique mod identifier for all assets and localization records, and stores only mod-owned state where a supported save-data API exists. Removing the mod must leave base-game save data untouched.

## Error Handling

- Missing required public API capability: do not register the AI Service Agency or its dependent content; document the unsupported game version and required SDK update.
- Missing server-furniture registration: disable the agency because it cannot satisfy its production rule.
- Missing optional UI hook: keep the business functional only when the confirmed financial and eligibility APIs remain available; document capacity through localized furniture descriptions and financial line items.
- Invalid mod-owned saved data: skip settlement for the affected agency, log its persistent identifier, and leave game-owned data unmodified.
- Invalid configuration value: use the documented default and report the rejected setting, provided the SDK's supported configuration API is available.

## Testing Strategy

Unit tests cover configuration validation, slot-capacity aggregation, deterministic high-skill allocation, tie-breaking, eligibility calculation, service-fee calculation, and daily-maintenance calculation.

Integration tests use game-facing adapters with fakes to verify these scenarios:

- A working AI Engineer with a workstation and available slot earns service revenue.
- A working AI Engineer without a workstation earns no service revenue.
- An eligible engineer becomes ineligible when all slots are consumed by higher-skill engineers.
- Server capacity from multiple two-slot and five-slot racks aggregates correctly.
- Ten and fifty eligible engineers can be covered exactly with the chosen rack sizes.
- All placed racks create a daily expense, including empty racks.
- Existing programmer and web-development-agency behavior is unchanged.

In-game verification uses a fresh save and an existing save: create an agency, place both rack types, hire and schedule engineers, fill and exceed capacity, advance an in-game day, and verify both revenue and maintenance entries in business finances.

## Non-Goals For First Release

- AI subscriptions, client contracts, projects, or customer queues.
- Manual employee-to-server assignment.
- Server failures, heat, electricity networks, or consumable inventory.
- Changes to existing base-game businesses or employee professions.
- New player-facing unlock requirements beyond the normal office-business creation requirements.
