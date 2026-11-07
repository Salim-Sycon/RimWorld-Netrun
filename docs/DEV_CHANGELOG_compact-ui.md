# Dev changelog: Compact interface (issue #7)

Branch `feat/compact-ai-room`, based on `dev` at `64cc221`. Developer-facing notes for review. The player-facing line is in `CHANGELOG.md` under Unreleased.

## What it is

A new setting, **Compact interface** (default **off**). When on, busy screens show less: labelled buttons are text-only everywhere, and the AI room also folds rare actions and drops repeated information. With the setting off, behaviour is unchanged.

The setting is in the terminal under MENU, Customization, Graphics (next to "CRT lines"), and from the main menu under Mod settings, Customization. It is saved as `compactUI` and loads as off from older settings files.

## Behaviour with the setting on

| Where | Change |
|---|---|
| Every screen that draws a labelled button through `NetrunUI.Button` | The icon is not drawn. Buttons with an empty label (icon-only) keep their icon. |
| `NetrunUI.ButtonWidth(label, icon: true)` | No longer adds the 30 px icon gap, so buttons are narrower. |
| AI room, buttons | Common actions stay (service, second service, open quest, Feed, Fulfil). Auto-feed, Upload, Move to coprocessor, Download, Move between coprocessors, Isolate/Connect and Erase move into one **More...** button that opens a `FloatMenu`. A disabled entry shows its reason in brackets, and nothing is added when the reason is empty. |
| AI room, chips | The request chip is not drawn (the request has its own row right below). |
| AI room, info rows | The food row stays, without an icon. The "no request" row is not drawn. The request row appears only when there is a request, without an icon. The request history moves into the food row's tooltip. |
| Board header | The "Active n/m" chip only shows while a contract is active (the m is the number of fixers, which the chip never explained). |

## Files changed (9 files, +147 / -26)

| File | Change |
|---|---|
| `Source/NetrunSettings.cs` | New `CompactUI` field, toggle in `DrawGraphics`, reset in `ResetCustomization`, save in `ExposeData` (default false). |
| `Source/Terminal/NetrunUI.cs` | `Button` drops the icon in compact mode when the label is non-empty. `ButtonWidth` skips the icon gap in compact mode. |
| `Source/Terminal/AITab.cs` | `Compact` property, `Fold` and `OpenMore` for the "More..." button, `ButtonSpec.Secondary`, compact paths in `InfoRowsHeight`, `DrawInfoRows` and `InfoRowLine`, `Chip` wrapper, per-frame element counters and `CountReport`. |
| `Source/Terminal/ContractsTab.cs` | Board header chip condition. |
| `Source/NetrunDebugActions.cs` | New dev action "UI: AI room element count" (category "Netrun: tools and achievements"). |
| `Languages/English/Keyed/AI.xml`, `Terminal.xml` | New keys. |
| `Languages/Russian (Русский)/Keyed/AI.xml`, `Terminal.xml` | New keys. |

New keys (English and Russian): `NR_SettingsCompact`, `NR_SettingsCompactTip`, `NR_AIMore`, `NR_AIMoreTip`.

## Dev action

"UI: AI room element count" prints chips, buttons, info rows and icons from the last drawn frame of the AI room, and whether that frame was compact or full. Open the AI room, then run it with the setting on and off to compare. Counters are reset at the start of each `AITab.Draw` and copied only on Repaint, so Layout events do not double-count.

## Checks done

- `dotnet build Source/Netrun.csproj -c Release -warnaserror`: 0 warnings, 0 errors (Windows .NET SDK 8.0.133, CI-style references plus the local Hacking Expansion DLL).
- `python tools/check_keys.py`: no missing or unused keys, English and Russian.
- A separate C# code review of the diff found no blocking problems. Two small points were fixed (empty brackets on disabled "More..." entries, stale counts in the dev report). Two were left on purpose: compact mode allocates two lists and a closure per card per frame (about three `Buttons()` calls per card, an existing pattern), and the compact request history in the tooltip is plain text.
- Tested in game by the author with the setting on: the interface looked decluttered. The AI room was the main screen compared.

## Not verified

- Each other screen's buttons in compact mode one by one (Core, Board, Fixers, Club, Studio, Archive, Heat, Help, Gateway panels, fixer shop and scrub windows). The change is one shared function, but a button that relied on its icon for meaning would show only its label.
- The Board header chip change in game.
- Russian label lengths in the narrower buttons.
- `Button` with an empty label and an icon (kept as is), for example the emergency exit on the Gateway map.

## Known limits and follow-ups

- Only the AI room has the full treatment (fewer rows and folded actions). Other tabs (Fixers, Studio, Club, Archive) draw meaningful icons (portraits, item icons, faction emblems), so they were left alone until someone decides what is decoration.
- Several items from `docs/AUDIT_UI_2026-10-23.md` were already implemented on `dev` when this was written (sort deduplicated, zero-reputation bar skipped, reason under DIVE, one-word risk), so they were not repeated here.
- The Gateway map and side panel are the maintainer's current UI plan (`docs/PLAN_UI_2026-10-23.md`); this change touches none of that code except the shared button function.
- This branch does not include the duplicate-icon fix from PR #5. That fix is independent (scanlines and icon replay), and the two merge without conflicts.
- `docs/STATUS.md` and `docs/TASKS.md` are not updated (they are written in Russian and owned by the maintainer).

## Rollback

Turn the setting off, or revert the single commit. The saved key `compactUI` is harmless if left in settings.

## How to test

1. Back up your save and settings.
2. Terminal, MENU, Customization, Graphics: tick **Compact interface**.
3. AI room (from the AI server and from the terminal's AI tab): check text-only buttons, one **More...** button, no duplicate request chip, no "no request" row, request history on hover of the food row. Use **Erase** from More... and confirm it still asks for confirmation. Run the dev action with the setting on and off.
4. Visit the other terminal screens and confirm every button is still readable and clickable.
5. Board: with no contract running, the Active chip is hidden.
6. Untick the setting and confirm everything looks as before.
