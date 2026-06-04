---
name: hero-combat-evaluation
description: Use when the user provides a FlamonMoba hero combat-power evaluation document, pasted hero design text, .docx/.md 英雄战力评估, enemy 1v1 收益/强敌 design notes, or asks to audit skill/talent combat variables or generate RobotCombatAttributesBase code.
---

# Hero Combat Evaluation

## Overview

Turn a hero combat-power document into FlamonMoba enemy 1v1 收益/强敌 evaluation code.

**Core principle:** Confirm the design document before generating code. The confirmed Markdown is the source of truth.

**Review boundary:** Trust the skill/talent descriptions and numeric values supplied in the document. Do not verify them against project configs by default; review only whether the author's combat-evaluation derivations from those descriptions are reasonable, complete, and unambiguous.

## When To Use

Use when the user provides or references:
- A hero 战力评估逻辑 document
- A `.docx`, `.md`, or directly pasted document/design for enemy 1v1 收益/强敌 calculation
- Skill/talent descriptions that need conversion into combat-evaluation variables
- A request to generate `RobotCombatAttributesBase` code for a hero

Do not use for unrelated AI behavior-tree tuning, target-selection logic, or shared algorithm refactors unless the user explicitly ties them to hero combat evaluation.

## The Iron Law

```
NO HERO COMBAT-EVALUATION CODE BEFORE AUTHOR-CONFIRMED MARKDOWN
```

If the document is not confirmed, stop at document review. Do not generate code, do not "rough in" a class, and do not silently resolve design questions in code.

## Gate Function

Before writing or changing hero combat-evaluation code:

1. **Normalize:** If the source is not Markdown, create a separate Markdown working document.
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
- `LongPhysicalDamagePerSecond`
- `LongSpellDamagePerSecond`
- Talent modifiers
- Cooldown gates
- Explicitly ignored effects

If something looks wrong:
- List each issue separately.
- State why it is questionable using the document's own descriptions, the shared 1v1 model, or the current QiuZhang implementation pattern.
- Ask the author to confirm design intent.
- Help revise the Markdown.
- Do not code yet.

If the document contains anything not clearly covered by the current QiuZhang example document/code or this checklist, treat it as ambiguous. Ask the author how it should map into the 1v1 model before editing code.

Obvious typo or internal contradiction? Propose a concrete Markdown correction and still report it.

### Step 3: Confirm Markdown

After revisions:
- Present or reference the final Markdown document.
- Ask the author to confirm it is correct.
- Treat the confirmed Markdown as the implementation source of truth.

No confirmation means no code generation.

### Step 4: Generate Code

Follow the current QiuZhang implementation as the example.

Required code pattern:
- One evaluated hero maps to one class under `Assets/Scripts/Battle/Robot/Model_MOBA/RobotCombatAttributes/RobotHero/`.
- The class inherits `RobotCombatAttributesBase`.
- The class calls `base.UpdateRobotCombatAttributes()` first.
- The class assigns only that hero's combat-evaluation attributes.
- Register the class in `RobotCombatAttributesBaseFactory` using the current `switch (player.HeroName)` pattern:

```csharp
case EHeroName.SomeHero:
    return new SomeHeroRobotCombatAttributesBase(player);
```

Keep `Player.Robot.cs` as the shared property holder/delegator. Do not put hero formulas in `Player.Robot.cs`, `NumericalPet.cs`, or the factory. The factory maps enum to class only.

Preserve `NumericalPet.cs` unless the confirmed design requires an algorithm-wide change.

## Quick Reference

| Situation | Action |
|----------|--------|
| Source document is `.docx` | Extract and create Markdown working copy |
| Source is pasted document text | Normalize it into a Markdown working draft; do not reject it for lacking a file |
| Supplied skill/talent description or number differs from project config | Ignore by default; trust the supplied document unless the user explicitly asks for config validation |
| Variable derivation is unclear | Ask author; revise Markdown; do not code |
| Mechanic not present in QiuZhang example/checklist | Ask author for intended 1v1 mapping |
| Markdown not confirmed | Stop before code generation |
| Hero already has a class | Update that hero class only |
| New hero | Add one `RobotCombatAttributesBase` subclass and one factory `case` |
| Effect has no current 1v1 representation | Mark as not represented or ask author for intended mapping |

## Code Mapping Rules

- Required per-hero fields: `CombatRange`, `MobilityDistance`, `AverageMobility`, `ControlDuration`, `ShortPhysicalDamage`, `ShortSpellDamage`, `ShortTrueDamage`, `LongPhysicalDamagePerSecond`, `LongSpellDamagePerSecond`, `EffectiveCombatReach`.
- Use live battle properties for global multipliers and reductions, such as extra damage and damage-taken rate.
- Cooldown gates such as "usable or CD within 2 seconds" use the project's current ability cooldown API and the QiuZhang example as precedent.
- Formula placeholders such as `lv1`, `lv2`, and `lv3` usually mean the current level of that skill unless the confirmed Markdown says otherwise.
- Short true damage remains separate and does not receive physical/spell extra-damage multipliers unless the design explicitly changes that rule.

## Accepted Project Conventions

These are not review findings:
- Combat attributes may be refreshed outside `CalcEnemyWealth`.
- `HasRobotCombatEvaluationData` is an access/registration marker, not a per-frame freshness marker.
- Per-frame `LogMessage()` in `UpdateRobotCombatAttributes()` is acceptable unless the user asks to remove it.
- The new 1v1 path does not need to maintain `EnemyHP` unless a later requirement explicitly uses it.

## Red Flags - Stop

- "The document is close enough; I'll code it first."
- "I'll verify these skill numbers in project configs before reviewing the author's formula."
- "I'll infer this talent mapping in code."
- "The QiuZhang example does not cover this mechanic, but I can map it myself."
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
