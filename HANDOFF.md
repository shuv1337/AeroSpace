# HANDOFF

## Objective
- Fix AeroSpace’s Ghostty tab-transient window bug where opening/closing tabs can damage tiling layout by briefly registering a fake AX window.

## Current status
- Structural fix implemented and pushed to fork `origin/main`.
- Local-only Xcode signing/project overrides committed separately.
- A concise handoff file is being committed separately for future pickup.

## Key context
- Root cause was a transient Ghostty AX window appearing during tab operations.
- Attribute-only filtering in `AxUiElementWindowType` was too race-prone because the transient window can partially disappear before classification.
- Winning direction was to debounce registration of new Ghostty windows instead of relying only on window heuristics.
- `origin` now points to `git@github.com:shuv1337/AeroSpace.git`.
- `upstream` points to `git@github.com:nikitabobko/AeroSpace.git`.

## Important files
- `Sources/AppBundle/tree/MacApp.swift` — defers registration of newly seen Ghostty windows; also avoids redundant set-frame calls and keeps focused windows alive in AX refresh.
- `Sources/AppBundle/layout/refresh.swift` — skips registering deferred new windows during refresh.
- `Sources/AppBundle/model/AxUiElementWindowType.swift` — Ghostty-specific transient/quick-terminal heuristics.
- `Sources/AppBundle/util/accessibility.swift` — added `AXDocument` accessor.
- `axDumps/ghostty_new_tab_transient.json5` — captured transient-window fixture.
- `AeroSpace.xcodeproj/project.pbxproj` — local signing/debug app naming overrides for this machine.

## Validation
- Ran: `swift test --filter AxWindowKindTest` ✅
- Rebased local `main` onto `origin/main` and pushed successfully.

## Changed artifacts / commits
- `5463b9ab` — `fix: debounce transient Ghostty AX windows`
- `9a0800d2` — `chore: add local Xcode signing overrides`
- Pending after this file write: separate commit for `HANDOFF.md`

## Next steps
1. Manually verify Ghostty tab open/close behavior against the pushed build.
2. If behavior is good, open a PR from `shuv1337/AeroSpace` to upstream.
3. Decide whether the local Xcode signing override commit should stay only on the fork or be excluded from upstream PR.

## Risks / open questions
- Manual validation of Ghostty behavior still matters; only targeted tests were run.
- `AeroSpace.xcodeproj/project.pbxproj` contains local-machine signing changes that may not belong in an upstream PR.
- `HANDOFF.md` is ephemeral context and likely should not be included in any upstream PR.