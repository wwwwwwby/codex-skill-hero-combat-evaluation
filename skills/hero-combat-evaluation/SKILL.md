---
name: hero-combat-evaluation
description: Use when the user provides a FlamonMoba hero combat-power evaluation document, pasted hero design text, .docx/.md 英雄战力评估, enemy 1v1 收益/强敌 design notes, or asks to audit skill/talent combat variables or generate RobotCombatAttributes/RobotHero code.
---

# Hero Combat Evaluation

## Overview

Turn a hero combat-power document into FlamonMoba enemy 1v1 收益/强敌 evaluation code.

**Core principle:** Confirm the design document before generating code. The confirmed Markdown is the source of truth.

**Review boundary:** Trust the skill/talent descriptions and numeric values supplied in the document. Do not verify them against project configs by default; review only whether the author's combat-evaluation derivations from those descriptions are reasonable, complete, and unambiguous.

**Markdown quality bar:** Any generated Markdown must be cleaned into a readable, discussion-ready working draft before presenting it to the author. Do not show raw DOCX extraction or pasted-text dumps as the review document.

## When To Use

Use when the user provides or references:
- A hero 战力评估逻辑 document
- A `.docx`, `.md`, or directly pasted document/design for enemy 1v1 收益/强敌 calculation
- Skill/talent descriptions that need conversion into combat-evaluation variables
- A request to generate `RobotCombatAttributes/RobotHero` code for a hero

Do not use for unrelated AI behavior-tree tuning, target-selection logic, or shared algorithm refactors unless the user explicitly ties them to hero combat evaluation.

## The Iron Law

```
NO HERO COMBAT-EVALUATION CODE BEFORE AUTHOR-CONFIRMED MARKDOWN
```

If the document is not confirmed, stop at document review. Do not generate code, do not "rough in" a class, and do not silently resolve design questions in code.

## Gate Function

Before writing or changing hero combat-evaluation code:

1. **Normalize:** If the source is not Markdown, create a separate Markdown working document and apply non-semantic formatting cleanup before presenting it.
2. **Review:** Check the author's variable derivations against the document's own skill/talent descriptions. Treat those descriptions and numbers as correct input.
3. **Clarify:** For every unreasonable, ambiguous, or not-covered-by-example derivation, ask the author to confirm intent and help revise the Markdown.
4. **Confirm:** Get explicit author confirmation that the final Markdown is correct.
5. **Generate:** Implement exactly what the confirmed Markdown says.

Skip any gate = stop. Ask the author.

## Workflow

### Step 1: Read And Normalize

- Read the full hero 战力评估逻辑 document.
- Understand that the document exists to support the new enemy 1v1 收益/强敌 calculation.
- Accept direct copy-pasted document text as a valid source; do not require a file upload.
- If the source is `.docx` or another non-Markdown format, extract it and create a Markdown copy for discussion and future edits.
- If the source is pasted text, create or maintain a Markdown working draft from that pasted content before review. Infer a reasonable file name from the hero name when obvious; ask only if the file destination is necessary and cannot be inferred.
- Store FlamonMoba hero combat-evaluation Markdown under `D:\work\Flamonmoba\CYTXDocuments\AIdocs` unless the user gives another destination.
- Before presenting the Markdown for discussion, perform non-semantic cleanup:
  - add a clear title and source note
  - group content into stable sections such as base variables, short output, short defense expectation, long output, skill/talent contribution, ignored effects, and open questions
  - use Markdown headings, bullets, and formula blocks/inline code so variables and formulas are easy to scan
  - preserve the supplied descriptions, numeric values, formulas, and design intent; do not change meaning during formatting
  - if raw text contains placeholders such as `null`, keep the evidence but place it under an explicit ignored/unclear section instead of leaving it as an unexplained loose line
- Keep the original file as evidence; use the Markdown copy as the editable working document.
- Do not check project configs or ability/talent tables to validate the document's skill descriptions or numeric values unless the user explicitly asks.
- Locate supporting code only for implementation mechanics: hero enum names, ability constants, talent unlock APIs, modifier property APIs, factory pattern, and existing hero-specific combat-attribute code if present.
- Load `references/flamonmoba-1v1-checklist.md`.

### Step 2: Review Before Coding

Treat the document's skill and talent descriptions as the design baseline. Assume the descriptions and their numeric values are correct.

Do not review whether the supplied description numbers match project config, skill UI, code, or tables. Review whether the authored combat-evaluation logic maps those supplied descriptions into the 1v1 model in a reasonable way.

Review these fields and derivations:
- `CombatRange`
- `MobilityDistance`
- `AverageMobility`
- `ControlDuration`
- `ShortPhysicalDamage`
- `ShortSpellDamage`
- `ShortTrueDamage`
- `ShortExpectedDamageReduction`
- `ShortExpectedShield`
- `LongPhysicalDamagePerSecond`
- `LongSpellDamagePerSecond`
- Talent modifiers
- Cooldown gates
- Explicitly ignored effects

If something looks wrong:
- List each issue separately.
- State why it is questionable using the document's own descriptions, the shared 1v1 model, or current registered hero implementation patterns.
- Ask the author to confirm design intent.
- Help revise the Markdown.
- Do not code yet.

For replacement skills, multi-stage casts, stance/form changes, or any mechanic where a skill slot can hold different runtime abilities, explicitly ask the author whether the combat-evaluation formula should read the current runtime ability from a skill slot. Confirm the slot before coding:
- Lower/bottom skill slot: `Self.GetAbilityBySlot(2)`
- Middle skill slot: `Self.GetAbilityBySlot(1)`
- Upper/top skill slot: `Self.GetAbilityBySlot(0)`

Before asking, inspect local evidence when available: existing hero combat-attribute code, behavior-tree assets, ability constants with paired names such as `_close` or `_1`/`_2`, and explicit behavior-tree checks such as `Check Slot With Skill Ability Type` / `Int32SlotIndex`. If that evidence confirms the runtime slot, record the confirmation in the Markdown review and implement from the slot ability. If it only suggests a replacement but does not prove the slot, ask the author to confirm the slot.

Once confirmed, use the slot ability for level and cooldown/readiness calculations, then pass that `Ability` through base helpers such as `GetAbilityLevel(...)` and `IsAbilityReadyWithin(...)`. Do not use a fixed `AbilityType` or a hand-written fallback list for that skill unless the author or current local implementation mechanics confirm that the skill is not slot-replaced.

When multiple issues are found, prefer one holistic review response instead of drip-feeding questions one by one, unless the author explicitly asks for step-by-step questioning. Do not output a bare checklist. Each issue must include the questionable mapping, your design concern or defect hypothesis, your proposed interpretation, and the exact confirmation needed from the author.

If the document contains anything not clearly covered by current registered hero examples or this checklist, treat it as ambiguous. Ask the author how it should map into the 1v1 model before editing code.

Obvious typo or internal contradiction? Propose a concrete Markdown correction and still report it.

### Step 3: Confirm Markdown

After revisions:
- Present or reference the final Markdown document.
- Ask the author to confirm it is correct.
- Treat the confirmed Markdown as the implementation source of truth.

No confirmation means no code generation.

### Step 4: Generate Code

Follow the current registered hero implementations as examples. Before editing, inspect the current local `RobotCombatAttributesBase`, `IRobotCombatAttributes`, factory, and at least two registered hero classes.

Required code pattern:
- One evaluated hero maps to one class under `Assets/Scripts/Battle/Robot/Model_MOBA/RobotCombatAttributes/RobotHero/`.
- The class inherits `RobotCombatAttributesBase`.
- Name the class `SomeHeroRobotCombatAttributes`, not `SomeHeroRobotCombatAttributesBase`, unless the current local registered heroes have changed to a different naming convention.
- The class calls `base.UpdateRobotCombatAttributes()` first.
- Use the current method signatures exactly. In the current local project, `UpdateRobotCombatAttributes()` refreshes attributes that do not need a specific enemy, and `UpdateRobotCombatAttributesWithEnemy(Player enemy)` refreshes attributes that depend on the target enemy.
- Do not add an overload such as `UpdateRobotCombatAttributes(Player enemy = null)`. Put target-specific formulas in `UpdateRobotCombatAttributesWithEnemy`.
- Do not compute target-specific formulas in `UpdateRobotCombatAttributes()` by passing `null`, `0`, or placeholder enemy values. Do not assign `0f` placeholders for target-specific damage fields in the no-argument refresh; let the target-specific refresh own those fields.
- Reuse protected helpers already available on `RobotCombatAttributesBase` before adding local helper methods. Current shared helpers include `HasTalent`, `IsSuperPowerReady`, `GetAbilityLevel`, `IsAbilityReadyWithin`, `IsChargeAbilityReadyWithin`, `GetSkillBullet`, and `GetCurrentSkillBullet`.
- Do not duplicate those helpers inside hero classes. If a new helper would be useful for more than one hero, add it to the base class instead of copying a private implementation into a hero class.
- Avoid private `static` utility methods in hero classes. Pure immutable lookup tables such as `private static readonly float[]` are acceptable, but formula helpers should normally be instance methods or base-class helpers.
- The class assigns only that hero's combat-evaluation attributes.
- Register the class in `RobotCombatAttributesBaseFactory` using the current `switch (player.HeroName)` pattern:

```csharp
case EHeroName.SomeHero:
    return new SomeHeroRobotCombatAttributes(player);
```

Keep `Player.Robot.cs` as the shared property holder/delegator. Do not put hero formulas in `Player.Robot.cs`, `NumericalPet.cs`, or the factory. The factory maps enum to class only.

Preserve `NumericalPet.cs` unless the confirmed design requires an algorithm-wide change. The current target-specific refresh scheme requires `CalcEnemyWealth` to refresh both sides before comparing damage: self with enemy, and enemy with self.

When adding files under `Assets`, include Unity `.meta` files for the new file and any new directory. For a new hero, keep repository edits scoped to the hero class and `.meta`, the factory case, and optional confirmed Markdown under `CYTXDocuments/AIdocs`; do not edit existing hero classes for style or precedent cleanup unless the user requested a shared-base change.

## Quick Reference

| Situation | Action |
|----------|--------|
| Source document is `.docx` | Extract and create discussion-ready Markdown working copy; do not present raw extraction |
| Source is pasted document text | Normalize it into a discussion-ready Markdown working draft; do not reject it for lacking a file |
| Raw document has loose paragraphs, repeated labels, or `null` placeholders | Reorganize non-semantically into sections before review |
| Supplied skill/talent description or number differs from project config | Ignore by default; trust the supplied document unless the user explicitly asks for config validation |
| Variable derivation is unclear | Ask author; revise Markdown; do not code |
| Mechanic not present in current examples/checklist | Ask author for intended 1v1 mapping |
| Markdown not confirmed | Stop before code generation |
| Hero already has a class | Update that hero class only |
| New hero | Add one `*RobotCombatAttributes` subclass and one factory `case` |
| New file under `Assets` | Include the corresponding Unity `.meta` file |
| Formula needs enemy HP, shield, resistance, or target buff stacks | Implement or update `UpdateRobotCombatAttributesWithEnemy(Player enemy)` |
| Target-specific field would be unknown in the no-argument refresh | Leave it to `UpdateRobotCombatAttributesWithEnemy`; do not assign a `0f` placeholder in `UpdateRobotCombatAttributes()` |
| Effect has no current 1v1 representation | Mark as not represented or ask author for intended mapping |
| Skill/talent description contains pre-cast reduction, immunity, shield, temporary HP, or shield-like survival value but the document omits `ShortExpectedDamageReduction`/`ShortExpectedShield` | Ask the author to add explicit values before code generation |
| Hero needs a helper that matches an existing base helper | Use the base helper; do not copy a private/static version into the hero class |
| Hero has replacement, multi-stage cast, stance/form change, or skill transformation mechanics | Inspect local slot evidence first; if not conclusive, ask the author whether the skill is runtime slot-replaced and confirm the slot: lower `2`, middle `1`, upper `0`; if confirmed, use `Self.GetAbilityBySlot(slot)` for level and readiness |

## Code Mapping Rules

- Required per-hero fields: `CombatRange`, `MobilityDistance`, `AverageMobility`, `ControlDuration`, `ShortPhysicalDamage`, `ShortSpellDamage`, `ShortTrueDamage`, `ShortExpectedDamageReduction`, `ShortExpectedShield`, `LongPhysicalDamagePerSecond`, `LongSpellDamagePerSecond`, `EffectiveCombatReach`.
- Use live battle properties for global multipliers and reductions, such as extra damage and damage-taken rate.
- `ShortExpectedDamageReduction` is a pre-cast short-window defensive expectation, expressed as a `0-1` non-true-damage reduction factor. Set it only while the relevant skill is available or within the author-confirmed near-ready threshold. After the skill is cast, the value should naturally return to `0` because runtime reduction, resistance, or immunity is expected to be captured by live battle properties.
- `ShortExpectedShield` is a pre-cast short-window expected shield or shield-like temporary survival value, expressed as flat HP. Set it only while the relevant skill is available or within the author-confirmed near-ready threshold. After the skill is cast, the value should naturally return to `0` because runtime `CurShieldVar` is expected to capture the real shield.
- If a confirmed defensive effect applies only to specific damage types, decide during document review how it should be approximated by `ShortExpectedDamageReduction`; if there is no confirmed approximation, do not invent one in code.
- `WeaponDamageAddition` and `SkillDamageAddition` in authored hero documents are project-convention names for basic-attack-specific and skill-specific damage additions. Do not flag them as duplicate with shared `ExtraDamageFactor` unless the author explicitly says they are global damage multipliers.
- For formulas that convert basic attacks through attack speed and time, require a fixed authored `BaseAttackFrequency` parameter and the runtime attack-speed coefficient during document review. If the confirmed formula uses a fixed hit count, do not require `BaseAttackFrequency` for that term. During code generation, inspect the current base field semantics first. In the current local project, protected `AttackSpeed` is already attacks per second (`attackSpeed / Self.RobotDPSConfig.HeroWeaponCD`), so normal-attack DPS uses `TotalWeaponDamage * AttackSpeed` and short windows multiply by the window duration.
- If the current base exposes `WeaponDamageAddition` and `SkillDamageAddition`, use those protected fields when applying weapon-specific or skill-specific damage additions; do not re-query the modifier properties in each hero class.
- Target-specific formula inputs, such as enemy current HP or target buff stack count, belong in `UpdateRobotCombatAttributesWithEnemy(Player enemy)`. Keep the no-argument update as the target-independent baseline.
- Target-specific damage fields should be assigned only in `UpdateRobotCombatAttributesWithEnemy`. Avoid wrapper methods such as `UpdateDamageAttributes(enemy)` when the split between target-independent and target-specific formulas becomes less obvious.
- Never call a target-specific formula from the no-argument update with `null`, `0`, or a fake enemy value. The no-argument update runs outside the specific enemy comparison loop and must not overwrite target-specific results.
- Before adding local helper methods, inspect `RobotCombatAttributesBase` and use the existing protected helpers. Current base helpers cover talent checks, superpower readiness, ability level fallback, cooldown windows, charge readiness, and bullet count.
- Keep hero-local helpers narrow and formula-specific. Do not add private copies of base helpers, and do not add private `static` helper methods for formula decomposition; pure immutable lookup arrays are the allowed static exception.
- For heroes with runtime skill replacement, multi-stage casts, stance/form changes, or skill transformations, confirm the slot with the author before coding and then use `Self.GetAbilityBySlot(slot)` for the current runtime ability. Slot mapping is lower/bottom `2`, middle `1`, upper/top `0`. Use that slot ability for `GetAbilityLevel`, cooldown readiness, bullet checks, and any formula that depends on the currently replaced skill.
- Local evidence can confirm the slot when behavior-tree assets or existing code explicitly expose it, for example `Self.GetAbilityBySlot(slot)` or `Check Slot With Skill Ability Type` with `Int32SlotIndex`. Paired ability names such as `_close`, `_1`, or `_2` are only replacement hints unless paired with an explicit slot.
- Do not use fixed `AbilityType` lookup or a hand-written fallback ability-type list for a confirmed slot-replaced skill. If the document hints at multi-stage or replacement behavior but the slot is not confirmed, stop at Markdown review and ask.
- Cooldown gates use the project's current ability cooldown API; use the author-confirmed threshold from the Markdown, with current registered heroes only as precedent for API access.
- Formula placeholders such as `lv1`, `lv2`, and `lv3` usually mean the current level of that skill unless the confirmed Markdown says otherwise.
- Short true damage remains separate and does not receive physical/spell extra-damage multipliers unless the design explicitly changes that rule.
- Short expected damage reduction applies only to short physical/spell actual damage in the shared algorithm; short true damage is not reduced by it unless the shared algorithm is explicitly changed later.

## Accepted Project Conventions

These are not review findings:
- Combat attributes may be refreshed outside `CalcEnemyWealth`.
- `HasRobotCombatEvaluationData` is an access/registration marker, not a per-frame freshness marker.
- Per-frame `LogMessage()` in `UpdateRobotCombatAttributes()` is acceptable unless the user asks to remove it.
- `NumericalPet` does not need to maintain an `_enemyHp` cache; target HP terms should be read through `enemy` in `UpdateRobotCombatAttributesWithEnemy`.

## Red Flags - Stop

- "The document is close enough; I'll code it first."
- "I'll show the raw DOCX extraction first and organize it later."
- "I'll verify these skill numbers in project configs before reviewing the author's formula."
- "I'll infer this talent mapping in code."
- "The current examples do not cover this mechanic, but I can map it myself."
- "This is probably what the author meant."
- "The source is docx, but I can discuss edits without a Markdown copy."
- "The author has not confirmed, but the implementation is obvious."
- "I'll add a formula directly to `NumericalPet.cs` for this hero."

All of these mean: stop, clarify, update Markdown, and get confirmation.

## Completion Requirements

When reporting back, separate:
- Markdown document created or updated
- Design issues raised and author decisions
- Code files generated or updated
- Verification commands run and their result
