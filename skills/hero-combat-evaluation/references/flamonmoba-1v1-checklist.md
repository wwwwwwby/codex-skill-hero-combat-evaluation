# FlamonMoba Hero Combat Evaluation Checklist

Use this bundled reference when reviewing or implementing hero combat evaluation code. It captures the current project rules; the author-confirmed hero Markdown remains the implementation source of truth.

## Project Flow Invariants

- Document trust boundary:
  - direct copy-pasted hero combat-evaluation documents are valid inputs; normalize pasted text into a Markdown working draft before review when needed
  - treat the skill/talent descriptions and numeric values supplied in the document as correct input
  - do not verify supplied descriptions or numbers against project configs, ability tables, hero UI descriptions, or current implementation unless the user explicitly asks for that validation
  - review only whether the author's combat-evaluation derivations from those supplied descriptions are reasonable, complete, and unambiguous
  - use project code lookup for implementation mechanics only: hero enum names, ability constants, talent unlock APIs, modifier property APIs, factory registration, and shared algorithm conventions
- Markdown normalization:
  - generated Markdown must be discussion-ready before it is presented to the author; raw DOCX extraction or pasted-text dumps are not acceptable review documents
  - store FlamonMoba hero combat-evaluation Markdown under `D:\work\Flamonmoba\CYTXDocuments\AIdocs` unless the user gives another destination
  - apply non-semantic formatting only: headings, stable section grouping, bullets, variable/code formatting, and formula readability cleanup
  - preserve supplied descriptions, numeric values, formulas, and design intent; do not silently rewrite combat logic while formatting
  - recommended section order: title/source note, base variables, short output, short defense expectation, long output, skill/talent contribution, ignored effects, open questions
  - place loose `null` placeholders or unexplained ignored effects under an explicit ignored/unclear section so they can be discussed
- Ambiguity handling:
  - if the document contains unclear combat-evaluation value derivations, unclear ignored effects, or mechanics not covered by current registered hero examples or this checklist, ask the author for design intent before coding
  - do not invent a 1v1 mapping for new mechanics, support effects, healing, immunity, cleanse, aura, terrain, conditional control, resource loops, summons, or ally-dependent behavior unless the author confirms the mapping in Markdown
  - if a hero has pre-cast damage reduction, immunity, shield, temporary HP, or shield-like defense that may be available or near-available, but the document omits `ShortExpectedDamageReduction` or `ShortExpectedShield`, ask the author to add the missing field before code generation
  - when in doubt, update the Markdown with the confirmed interpretation before implementation
- Review-question style:
  - when multiple review issues exist, present them together in one structured review by default; switch to one-by-one questioning only if the author asks for it
  - do not simply list variable names or checklist failures
  - for each issue, include the questionable mapping, why it may be a design defect or incomplete abstraction, the assistant's proposed interpretation, and the confirmation needed from the author
  - after the author answers, fold the confirmed decisions back into the Markdown before code generation
- Combat-attribute flow:
  - one evaluated hero maps to one class under `RobotCombatAttributes/RobotHero/`
  - each hero class inherits `RobotCombatAttributesBase`
  - current registered hero classes use the `*RobotCombatAttributes` suffix, not `*RobotCombatAttributesBase`; copy the current local naming convention before adding a class or factory case
  - each hero class calls `base.UpdateRobotCombatAttributes()` before assigning hero-specific values
  - each hero class assigns only that hero's combat-evaluation values: `CombatRange`, `MobilityDistance`, `AverageMobility`, `ControlDuration`, short damage, short expected defense, long DPS, true damage, and `EffectiveCombatReach`
  - before generating code, inspect the current local `RobotCombatAttributesBase`, `IRobotCombatAttributes`, factory, and at least two registered hero classes; match their exact class naming, method signature, and refresh flow
  - current local structure uses `UpdateRobotCombatAttributes()` for target-independent values and `UpdateRobotCombatAttributesWithEnemy(Player enemy)` for values that need a specific enemy
  - do not introduce a target-parameter overload such as `UpdateRobotCombatAttributes(Player enemy = null)`; use the separate `UpdateRobotCombatAttributesWithEnemy` interface instead
  - if a confirmed formula needs target-specific values such as enemy current HP, enemy shield, enemy resistance, target buff stacks, or target-dependent short expected defense values, implement or update `UpdateRobotCombatAttributesWithEnemy(Player enemy)` instead of dropping the term
  - `CalcEnemyWealth` must refresh target-specific attributes for both sides before comparing damage: self with enemy, and enemy with self
  - for a new hero, keep repository edits scoped to the hero class, the hero class `.meta`, the factory case, and optional confirmed Markdown under `CYTXDocuments/AIdocs`; do not edit existing hero classes for style or precedent cleanup unless the user requested a shared-base change
  - when adding any file or directory under `Assets`, include the corresponding Unity `.meta` file
  - `RobotCombatAttributesBaseFactory.CreateRobotCombatAttributes` registers heroes with a `switch (player.HeroName)` case
  - unsupported heroes return `null` from the factory and must fall back to the legacy enemy-wealth calculation
  - combat attributes may be refreshed outside `CalcEnemyWealth`; do not require `CalcEnemyWealth` to refresh self/enemy if the project already refreshes them elsewhere
  - `HasRobotCombatEvaluationData` indicates that a hero has registered evaluation data; it is not required to prove current-frame freshness
  - per-frame `LogMessage()` during combat-attribute refresh is accepted unless the user explicitly asks to remove it
  - `NumericalPet` does not need to maintain an `_enemyHp` cache; target HP terms should be read through `enemy` in `UpdateRobotCombatAttributesWithEnemy`
- Ability data conventions:
  - inspect current registered hero implementations as precedent for skill level and cooldown access
  - current hero code reads skill level from `skill.Level`
  - current hero code evaluates cooldown gates with `skill.CurCdTime` and the author-confirmed threshold
  - if the document only says "short time" without a value, ask before coding
- Damage-addition conventions:
  - `WeaponDamageAddition` and `SkillDamageAddition` in authored hero documents are basic-attack-specific and skill-specific additions by project convention
  - do not raise a duplicate-multiplier concern against shared `ExtraDamageFactor` for these names unless the author explicitly describes them as global damage multipliers
- Basic-attack frequency conventions:
  - formulas that convert basic attacks through attack speed and time should use an authored fixed `BaseAttackFrequency` parameter
  - formulas that use a confirmed fixed hit count do not need `BaseAttackFrequency` for that term
  - runtime attack speed is a coefficient
  - conceptually, attacks per second are `runtime attack-speed coefficient / BaseAttackFrequency`
  - during code generation, inspect the current base first; in the current local base, protected `AttackSpeed` is already attacks per second (`attackSpeed / Self.RobotDPSConfig.HeroWeaponCD`), so do not divide by `BaseAttackFrequency` again inside a hero class
  - if a document includes basic attacks but omits `BaseAttackFrequency`, ask the author to provide it before code generation
- Base combat-attribute values:
  - `RobotCombatAttributesBase` reads `TotalWeaponDamage` from `MODIFIER_PROPERTY_TOTAL_WEAPON_DAMAGE`
  - inspect current local base before using `AttackSpeed`; in recent local code it may represent attacks per second (`attackSpeed / Self.RobotDPSConfig.HeroWeaponCD`) rather than attack interval
  - if `AttackSpeed` is attacks per second, normal-attack DPS is `TotalWeaponDamage * AttackSpeed`; if it is attack interval, DPS is `TotalWeaponDamage / AttackSpeed`
  - if raw `attackSpeed` is `0`, follow the base fallback value
  - current base exposes `WeaponDamageAddition` and `SkillDamageAddition`; when confirmed Markdown or project convention requires basic-attack-specific or skill-specific damage additions, use these protected fields instead of re-querying modifier properties inside each hero class
  - `ExtraDamageFactor` is read from `MODIFIER_PROPERTY_EXTRA_DAMAGE_FACTOR`
  - base combat attributes default `ShortExpectedDamageReduction` and `ShortExpectedShield` to `0`
  - `ShortExpectedDamageReduction` is a `0-1` pre-cast expected reduction for short physical/spell damage only; true damage is not reduced by this field
  - `ShortExpectedShield` is a flat pre-cast expected HP/shield value added to short combat survival and short-kill thresholds

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
  - `ShortExpectedDamageReduction` reduces only short physical/spell actual damage after resistance and damage-taken rates; true damage stays unreduced
  - post-fight health and short-kill thresholds use current HP plus non-castle shield plus non-negative `ShortExpectedShield`
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
- Confirmed hero combat-evaluation Markdown: `D:\work\Flamonmoba\CYTXDocuments\AIdocs`
- Current example hero implementation lives under `Assets/Scripts/Battle/Robot/Model_MOBA/RobotCombatAttributes/RobotHero/`; inspect the current registered class name before copying structure
- Shared 1v1 calculation: `Assets/Scripts/Battle/Robot/Model_MOBA/Modules/NumericalPet.cs`
- MOBA target wealth call site: `Assets/Scripts/Battle/Robot/Model_MOBA/Actions/CalcWealthMalice/CalcEnemyWealthMalice.cs`
- Extra damage factor is represented by `ModifierPropertyKeyName.MODIFIER_PROPERTY_EXTRA_DAMAGE_FACTOR`
- Player shield without castle shield is `CurShieldVar`; castle shield is separate as `CurCastleShield`
