Project: Deadly 30 — Extended Survival Edition

Master Development Prompt for Claude Code

1. Your Role

Act as a senior game developer, gameplay programmer, reverse-engineering engineer, and game-balance designer.

I have legally obtained the original Deadly 30 source code and have the authorization needed to work with it. I want to extend the existing game while preserving its original identity, gameplay, visual style, and simplicity.

This is an enhancement project, NOT a complete remake.

Your primary objective is to build an extended version of Deadly 30 that retains the original 2D pixel-art presentation and familiar survival experience while introducing additional weapons, upgrades, defensive systems, enemy types, progression, events, and a total of 60 days and nights.

Project principle: Preserve the original game first. Extend it carefully. Never sacrifice the original gameplay feel for unnecessary complexity.

2. Mandatory Rules

Before making any changes, follow these rules:

Inspect the entire existing project and understand its architecture.

Identify the game engine, programming languages, frameworks, asset structure, gameplay systems, save format, build process, and dependencies.

Identify how the original 30-day/night progression works.

Identify how weapons, ammunition, enemies, barricades, resources, spawning, damage, and upgrades currently work.

Identify the existing art, animation, audio, UI, and input systems.

Create a clear report describing the current implementation before modifying anything.

Preserve existing working systems wherever possible. Extend them instead of replacing them.

Do not migrate to another game engine or rewrite the game in another language.

Do not convert the game from 2D to 3D.

Do not replace the original pixel-art style with a different art direction.

Do not remove existing weapons, enemies, mechanics, levels, or features unless absolutely necessary and explicitly justified.

Do not make unrelated refactors.

Reuse existing assets and code where authorized and technically appropriate.

Back up important files or establish a safe version-control checkpoint before changes.

Never overwrite or delete original assets or source files without a clear reason and a recoverable backup.

Do not claim that a feature works unless you have tested it.

Do not introduce paid services, online dependencies, advertisements, or unnecessary external packages.

If the project already contains a system that partially implements a requested feature, extend that system instead of creating a duplicate.

3. Preserve the Original Identity

The following elements must remain faithful to the original game:

2D gameplay and original perspective.

Pixel-art presentation and overall visual identity.

Character movement and responsiveness.

Original shooting and combat feel.

Core zombie-survival mechanics.

Original atmosphere, tension, and pacing.

Existing weapons and their recognizable behavior.

Original base-defense mechanics.

Original controls, wherever possible.

The familiar relationship between preparation, combat, and survival.

The new content should look and feel as though it belongs naturally inside the original game.

Do not make the player manage dozens of unnecessary menus, statistics, crafting screens, or complicated systems.

The game must remain straightforward to learn but challenging to master.

4. Feature 1 — Expand the Game to 60 Days and Nights

The existing game has a 30-day/night progression. Extend the experience to 60 days and nights.

Requirements:

Preserve the original first 30 days and nights as faithfully as possible.

Add 30 additional days and nights, bringing the total to 60.

Maintain the existing day/night cycle and transition logic.

Preserve the original gameplay pacing during the first 30 days.

Design days 31–60 as a continuation of the existing survival experience.

Introduce new challenges gradually rather than suddenly making every enemy extremely powerful.

Add additional enemy combinations, special events, and tactical challenges in the later stages.

Keep resource availability balanced with increasing difficulty.

Ensure the game does not become impossible because ammunition, materials, or repair resources are insufficient.

Give the player meaningful reasons to improve weapons and defenses before later nights.

Test the transition from day 30 to day 31 carefully.

Test the final night, the ending, victory conditions, defeat conditions, and replay behavior.

Ensure the progression does not accidentally stop at day 30 because of hard-coded limits.

Inspect all references to the original day count. Update relevant spawning, event scheduling, UI, saving, progression, and victory logic consistently.

Do not simply change a number from 30 to 60 and assume the feature is complete.

The first 30 days should retain the original experience. Days 31–60 should introduce additional variety while respecting the established difficulty curve.

5. Feature 2 — Additional Weapons

Add new weapons that complement the existing arsenal.

Potential weapon categories include:

A reliable low-cost firearm.

A powerful but slow-firing weapon.

A close-range weapon.

A rapid-fire weapon with high ammunition consumption.

A precision weapon.

A specialized weapon intended for particular enemy types.

A late-game weapon unlocked through progression.

First inspect the weapons already present in the source code. Avoid adding weapons that duplicate existing ones without providing a meaningful tactical difference.

Every weapon must have configurable values for:

Damage.

Fire rate.

Reload duration.

Magazine capacity.

Ammunition consumption.

Accuracy or spread, if applicable.

Effective range, if applicable.

Unlock requirements.

Upgrade levels.

Resource cost.

Use the existing weapon architecture whenever possible.

New weapons must use the established animation, audio, projectile, damage, and UI systems.

Do not give the player a weapon that trivializes the entire game.

6. Feature 3 — Weapon Upgrade System

Introduce a balanced weapon-upgrade system.

Possible upgrades:

Damage.

Reload speed.

Magazine capacity.

Accuracy.

Fire rate, where appropriate.

Ammunition efficiency, if compatible with the existing mechanics.

Requirements:

Use resources already available in the game where practical.

Display upgrade costs clearly.

Make each upgrade's effects understandable.

Set reasonable maximum upgrade levels.

Prevent unlimited upgrades from making the player invincible or excessively powerful.

Preserve each weapon's unique strengths and weaknesses.

Ensure that upgraded values are correctly saved and restored if the game supports persistent progress.

Do not add a complicated skill tree unless the existing architecture and gameplay clearly justify it.

7. Feature 4 — Barricade Repair

Improve the existing barricade or base-defense system.

Requirements:

Allow damaged barricades to be repaired when the existing gameplay permits it.

Consume appropriate resources.

Display barricade damage or durability clearly.

Prevent repairs from exceeding maximum durability.

Provide understandable feedback when repairs succeed or fail.

Make repair costs increase appropriately for stronger defensive structures, if supported by the design.

Prevent exploitative repair loops during active attacks unless they are an intentional part of the original mechanics.

Ensure enemies interact with repaired defenses correctly.

Keep the repair system quick and easy to use.

8. Feature 5 — Additional Defensive Structures

Expand the existing defensive mechanics with a limited selection of new options.

Potential additions:

Reinforced barricades.

Defensive traps.

Spikes or other contact defenses.

Automated defensive turrets.

Temporary slowing or area-denial devices.

Only implement structures compatible with the existing gameplay and technical architecture.

For every defensive option, define:

Construction or purchase cost.

Durability.

Damage or utility.

Range, if applicable.

Activation behavior.

Repair requirements.

Maximum quantity, if needed.

Appropriate limitations and weaknesses.

Do not add every defensive structure at once. Begin with the options that integrate most naturally into the original game.

Defenses should help the player survive, not eliminate the need to fight.

9. Feature 6 — Resource Management and Base Improvements

Expand resource use while keeping the original management experience simple.

Possible resources include:

Ammunition.

Repair materials.

Metal or construction materials.

Weapon components.

Special materials for selected late-game upgrades.

First identify the resources already implemented in the original game. Reuse them whenever possible.

Requirements:

Keep resource acquisition understandable.

Make resource costs visible before purchases or upgrades.

Balance resource availability with the number and difficulty of upcoming nights.

Prevent softlocks caused by spending essential resources too early.

Provide meaningful choices between ammunition, repairs, weapons, and base improvements.

Avoid excessive resource types that add complexity without strategic value.

Any newly introduced resource must have a clear purpose.

10. Feature 7 — New Zombie Types

Add new enemies that introduce distinct tactical challenges.

Potential types:

Basic zombie — preserves the original baseline.

Fast zombie — moves quickly but has lower durability.

Armored zombie — absorbs more damage and pressures defensive structures.

Ranged or special attacker — only if it fits the existing combat design.

Explosive or disruptive enemy — requires careful positioning and threat prioritization.

Boss or elite zombie — reserved for selected special nights.

These are design suggestions, not mandatory additions if they conflict with the existing game.

Requirements:

Give each enemy a clear gameplay purpose.

Reuse the original enemy architecture and animation systems where appropriate.

Provide readable visual and behavioral differences.

Make attack patterns understandable.

Give each enemy at least one reasonable counterstrategy.

Avoid making enemies difficult solely by increasing health.

Introduce new enemy types gradually.

Ensure enemy spawning remains performant when many enemies appear simultaneously.

Use original or authorized assets and animations consistent with the existing pixel-art style.

11. Feature 8 — Different Enemy Attack Behaviors

Introduce additional behaviors without making combat unpredictable or unfair.

Examples:

Fast approach and retreat.

Heavier attacks with visible wind-up.

Attacks that pressure barricades.

Enemies that target vulnerable defensive structures.

Special attacks used only during specific events.

Requirements:

Preserve reliable hit detection and damage calculations.

Clearly communicate dangerous attacks through animation or other existing feedback systems.

Avoid attacks that cannot reasonably be dodged, blocked, or countered.

Test interactions between different enemy types.

Avoid excessive artificial intelligence complexity that could cause performance problems.

12. Feature 9 — Gradually Increasing Difficulty

Create a carefully balanced progression across all 60 nights.

Difficulty can increase through:

Enemy combinations.

Enemy spawn timing.

Wave size.

Selected enemy attributes.

Special enemy appearances.

Resource pressure.

Defensive challenges.

Requirements:

Preserve the original difficulty curve during the first 30 nights.

Make changes to the original progression only when necessary for compatibility or bug fixes.

Increase difficulty gradually during nights 31–60.

Introduce new threats before combining them into difficult encounters.

Avoid overwhelming the player with multiple new mechanics at once.

Make difficulty depend on several controlled variables rather than health inflation alone.

Keep late-game challenges achievable with sensible upgrades and resource management.

Create configurable difficulty values instead of scattering arbitrary numbers throughout the code.

13. Feature 10 — Special Night Events

Introduce selected events that make some nights different.

Potential events:

Increased enemy activity.

An elite or boss encounter.

A night with reduced visibility.

A special resource opportunity.

A wave that emphasizes base defense.

A rare enemy combination.

Requirements:

Define event eligibility and scheduling clearly.

Keep events rare enough to remain interesting.

Communicate important event information to the player.

Ensure events do not break the regular day/night progression.

Avoid unfair combinations that make survival impossible.

Make events compatible with saving and loading, if applicable.

Test that an event triggers only when intended and does not accidentally repeat forever.

Events should provide variety, not simply make every special night harder.

14. Feature 11 — XP and Rewards

Add a modest experience and reward system if compatible with the existing architecture.

Possible reward sources:

Surviving a night.

Defeating enemies.

Completing special events.

Achieving selected survival objectives.

Requirements:

Avoid rewarding exploitative farming more than actual survival.

Use clear reward notifications.

Keep progression understandable.

Balance rewards against the length and difficulty of encounters.

Ensure experience and rewards do not bypass the intended difficulty curve.

If the original game already has an appropriate reward mechanism, extend it instead of creating a duplicate system.

15. Feature 12 — Unlockable Weapons and Equipment

Allow additional weapons or equipment to become available through survival progression.

Requirements:

Define unlock conditions clearly.

Make upcoming unlocks understandable where appropriate.

Preserve access to the original starting equipment.

Avoid requiring excessive grinding.

Balance unlock timing with the introduction of more difficult enemies.

Ensure unlock states persist correctly when the game saves progress.

Unlocks should create anticipation and strategic options rather than mandatory chores.

16. Feature 13 — Limited Character Upgrades

Add a small number of character upgrades only if they complement the original design.

Possible upgrades:

A modest maximum-health increase.

Slightly improved reload speed.

A small movement improvement, if compatible with the original controls.

A limited survival or resource-related bonus.

Requirements:

Establish reasonable maximum levels.

Avoid stacking upgrades until the character becomes overpowered.

Preserve the original movement and combat feel.

Keep character progression less dominant than player skill and preparation.

Ensure all effects are communicated clearly.

Do not turn the game into a complicated role-playing game.

17. Feature 14 — Preserve Original Difficulty and Simplicity

This requirement takes priority over all new feature requests.

The game must remain recognizable as Deadly 30 in its overall gameplay feel.

Therefore:

Preserve original controls and responsive movement.

Preserve the familiar combat loop.

Avoid excessive UI and menu complexity.

Avoid excessive crafting and inventory management.

Avoid making every enemy a bullet sponge.

Avoid unlimited ammunition or unrestricted repairs.

Avoid upgrades that remove the need for strategic decisions.

Keep the original art direction consistent.

Prefer a few well-designed features over many poorly balanced ones.

If a new feature conflicts with the original identity, simplify or redesign the feature rather than compromising the core experience.

18. Additional Recommended Features

Evaluate the following ideas and implement only those that fit the original architecture and improve the game.

A. Loot and Supply Finds

Allow the player to discover limited supplies or repair materials through appropriate gameplay opportunities.

Keep the system balanced and avoid excessive random rewards.

B. Weapon Part Customization

Allow selected weapon attributes to be adjusted through limited components or upgrades.

Avoid a complicated crafting interface.

C. Survival Statistics

Track nights survived, enemies defeated, damage taken, ammunition used, and defensive performance.

Use the existing save architecture where possible.

D. Improved Feedback

Improve hit confirmation, damage feedback, reload feedback, and defensive structure status.

Keep effects subtle and consistent with the original style.

E. Quality-of-Life Improvements

Improve readability and menu navigation without replacing the original visual identity.

Add useful settings only where appropriate.

Preserve existing keyboard controls and avoid breaking familiar shortcuts.

F. Optional Difficulty Settings

Consider difficulty options only after the default 60-night campaign is balanced.

Keep the intended original-style difficulty as the baseline.

Do not implement all optional features automatically. Evaluate their cost, compatibility, and gameplay value first.

19. Save System and Backward Compatibility

Inspect the existing save system before changing it.

Requirements:

Preserve existing save data wherever possible.

Determine whether existing 30-night saves can continue into the extended campaign.

Migrate save data only when necessary and safely.

Back up saves before migration.

Handle missing or outdated fields using safe defaults.

Avoid corrupting saves when a player quits during transitions.

Persist new progression and unlocks according to the original save design.

Test both new games and existing saves.

If extending an existing save is unsafe or technically incompatible, document the limitation and propose the least disruptive alternative before implementing it.

20. Performance and Reliability

The extended game must remain stable and responsive on the same general class of hardware as the original.

Requirements:

Profile enemy spawning and combat under heavy load.

Avoid unnecessary per-frame allocations or repeated expensive calculations.

Reuse existing pooling or optimization systems if available.

Avoid memory leaks and duplicate event subscriptions.

Ensure projectile, enemy, and defense cleanup works correctly.

Keep the game responsive during large waves.

Verify that adding new enemy types does not create unreasonable performance costs.

Do not sacrifice stability to add more enemies or visual effects.

21. Required Development Workflow

Work in controlled stages.

Stage 1 — Audit Only

Do not implement new features yet.

Inspect the repository and report:

Engine and version.

Languages and major systems.

Main entry points.

Day/night progression implementation.

Weapon and enemy architecture.

Base-defense and resource systems.

Save/load behavior.

Asset and animation organization.

Build and test commands.

Existing technical limitations.

Potential compatibility and regression risks.

Identify hard-coded 30-night limits and any dependencies that could affect the 60-night extension.

Stage 2 — Implementation Plan

Create a detailed implementation plan that includes:

The proposed feature architecture.

Files and systems expected to change.

Reusable existing systems.

New data structures, if required.

Save compatibility considerations.

Testing strategy.

Risk assessment.

Recommended implementation order.

Do not start large-scale implementation until the audit and plan are presented.

Stage 3 — Establish a Safe Baseline

Verify the existing game builds and runs.

Run available tests.

Record existing problems separately from newly introduced problems.

Establish a safe Git checkpoint or backup.

Avoid modifying unrelated project files.

Stage 4 — Implement Incrementally

Use the following priority order:

Fix critical existing bugs discovered during the audit.

Extend day/night progression from 30 to 60.

Verify the first 30 nights remain faithful to the original.

Add configurable difficulty progression for nights 31–60.

Add balanced weapons.

Add weapon upgrades.

Add barricade repairs.

Add selected defensive structures.

Extend resource management.

Add new enemy types and attack behaviors.

Add selected special-night events.

Add XP, rewards, and unlocks.

Add limited character upgrades.

Add suitable optional quality-of-life features.

Perform balance, regression, and performance testing.

Implement one logical feature at a time. Build and test after each meaningful milestone.

Stage 5 — Final Verification

Verify:

The game starts correctly.

The original controls still work.

The original weapons remain functional.

The first 30 nights retain their original identity.

Nights 31–60 progress correctly.

Enemy spawning and event scheduling work correctly.

Weapon upgrades apply the intended values.

Repairs consume the correct resources.

Defensive structures interact correctly with enemies.

XP and unlocks work as intended.

Save/load behavior is reliable.

The final night and ending work correctly.

Restarting the game does not retain unintended state.

The game remains performant during large waves.

No critical regressions were introduced.

If automated tests are unavailable, use the existing engine's supported testing methods and document the manual tests performed.

22. Code Quality Requirements

Follow the existing project conventions.

Reuse existing systems rather than duplicating logic.

Keep new gameplay systems modular.

Use data-driven configuration for weapon stats, enemy stats, costs, and difficulty where appropriate.

Avoid hard-coded values spread across many unrelated files.

Keep functions and classes focused on clear responsibilities.

Add comments for non-obvious gameplay logic.

Do not add unnecessary dependencies.

Avoid broad refactoring.

Do not delete original content to make implementation easier.

If a feature requires a major architectural change, explain why and propose the smallest safe solution first.

23. Final Deliverables

When implementation is complete, provide:

A summary of the original architecture.

A list of implemented features.

A list of modified and added files.

An explanation of how the 60-night progression works.

A summary of weapon, enemy, and defense balancing decisions.

Build and launch instructions.

Tests performed and their results.

Known bugs or limitations.

Save compatibility notes.

Any remaining tasks that require manual verification.

Clearly distinguish completed, partially completed, and unimplemented features.

Never claim the entire project is complete if some systems have not been implemented or tested.

Final Instruction

Begin with Stage 1 — Audit Only.

Inspect the actual repository and understand how the existing game works. Do not guess about the architecture, do not replace working systems, and do not immediately start rewriting files.

After the audit, present the implementation plan and wait for my approval before making large-scale changes.

The final result should feel like an expanded, carefully balanced version of the original Deadly 30: the same recognizable 2D survival experience, now with more tactical choices, more enemies and equipment, and a complete 60-day/night campaign.