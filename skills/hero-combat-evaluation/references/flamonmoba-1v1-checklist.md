# FlamonMoba Hero Combat Evaluation Checklist

Use this bundled reference when reviewing or implementing hero combat evaluation code. It captures the current project rules; the author-confirmed hero Markdown remains the implementation source of truth.

## Project Flow Invariants

- Ambiguity handling:
  - if the document contains unclear value derivations, unclear ignored effects, or mechanics not covered by the current QiuZhang example document/code, ask the author for design intent before coding
  - do not invent a 1v1 mapping for new mechanics, support effects, healing, immunity, cleanse, aura, terrain, conditional control, resource loops, summons, or ally-dependent behavior unless the author confirms the mapping in Markdown
  - when in doubt, update the Markdown with the confirmed interpretation before implementation
- Combat-attribute flow:
  - one evaluated hero maps to one class under `RobotCombatAttributes/RobotHero/`
  - each hero class inherits `RobotCombatAttributesBase`
  - each hero class calls `base.UpdateRobotCombatAttributes()` before assigning hero-specific values
  - each hero class assigns only that hero's combat-evaluation values: `CombatRange`, `MobilityDistance`, `AverageMobility`, `ControlDuration`, short damage, long DPS, true damage, and `EffectiveCombatReach`
  - `RobotCombatAttributesBaseFactory.CreateRobotCombatAttributes` registers heroes with a `switch (player.HeroName)` case
  - unsupported heroes return `null` from the factory and must fall back to the legacy enemy-wealth calculation
  - combat attributes may be refreshed outside `CalcEnemyWealth`; do not require `CalcEnemyWealth` to refresh self/enemy if the project already refreshes them elsewhere
  - `HasRobotCombatEvaluationData` indicates that a hero has registered evaluation data; it is not required to prove current-frame freshness
  - per-frame `LogMessage()` during combat-attribute refresh is accepted unless the user explicitly asks to remove it
  - the new 1v1 path does not need to maintain `EnemyHP` unless a later consumer explicitly requires it
- Ability data conventions:
  - follow the current QiuZhang implementation as precedent for skill level and cooldown access
  - current QiuZhang code reads skill level from `skill.Level`
  - current QiuZhang code treats a skill as usable for combat evaluation when `skill.CurCdTime < 2`
- Base combat-attribute values:
  - `RobotCombatAttributesBase` reads `TotalWeaponDamage` from `MODIFIER_PROPERTY_TOTAL_WEAPON_DAMAGE`
  - its protected `AttackSpeed` field is the attack interval, computed as `1 / attackSpeed`
  - if raw `attackSpeed` is `0`, the base uses `1` before reciprocal conversion
  - `ExtraDamageFactor` is read from `MODIFIER_PROPERTY_EXTRA_DAMAGE_FACTOR`

## Shared Algorithm Invariants

- Damage-taken rates:
  - `PhysicalResistanceTakenRate = 1 - PhysicalResistance / (PhysicalResistance + 100)`
  - `SpellResistanceTakenRate = 1 - SpellResistance / (SpellResistance + 100)`
  - final physical/spell taken rates multiply `DamageTakenRate`
  - true damage taken rate is `1`
- Free attack time:
  - `EffectiveCombatReach = CombatRange + MobilityDistance`
  - distance loss uses `max(0, distance - reach) / max(AverageMobility, epsilon)`
  - self/enemy free attack time must be clamped to `[0, MaxFreeAttackTime]`
- Damage conversion:
  - physical and spell short/long damage multiply the attacker's `ExtraDamageFactor`
  - true damage does not multiply `ExtraDamageFactor`
  - post-fight health uses current HP plus non-castle shield only
- Kill branch:
  - if only self can kill within short damage, enemy is not strong and utility is `KillEnemyUtility`
  - if only enemy can kill within short damage, enemy is strong and utility is `0`
  - reachability checks run only when both sides can kill within short damage
  - if neither side can short-kill, continue to normal tri-state logic
- Strong fallback:
  - if no kill branch produced utility and self post-fight health is `<= 0`, enemy is strong and utility is `0`
- Normal tri-state:
  - compare post-fight survival time delta
  - neutral targets produce `NeutralEnemyUtility` only if the neutral distance trigger is satisfied
  - neutral targets outside trigger range return utility `0`, not saturated utility
  - non-neutral utility starts at `NeutralEnemyUtility` and adds saturated extra utility

## FlamonMoba Code Anchors

- Shared combat-evaluation properties and delegate entry: `Assets/Scripts/Battle/View/Model_MOBA/ViewModel/Actor/Player.Robot.cs`
- Common combat-attribute base and factory: `Assets/Scripts/Battle/Robot/Model_MOBA/RobotCombatAttributes/`
- Hero-specific combat attributes: `Assets/Scripts/Battle/Robot/Model_MOBA/RobotCombatAttributes/RobotHero/`
- Current example hero implementation: `Assets/Scripts/Battle/Robot/Model_MOBA/RobotCombatAttributes/RobotHero/QiuZhangRobotCombatAttributesBase.cs`
- Shared 1v1 calculation: `Assets/Scripts/Battle/Robot/Model_MOBA/Modules/NumericalPet.cs`
- MOBA target wealth call site: `Assets/Scripts/Battle/Robot/Model_MOBA/Actions/CalcWealthMalice/CalcEnemyWealthMalice.cs`
- Extra damage factor is represented by `ModifierPropertyKeyName.MODIFIER_PROPERTY_EXTRA_DAMAGE_FACTOR`
- Player shield without castle shield is `CurShieldVar`; castle shield is separate as `CurCastleShield`
