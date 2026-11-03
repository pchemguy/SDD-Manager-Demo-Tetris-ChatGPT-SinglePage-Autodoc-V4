# Single-page HTML Tetris

## Purpose and outcome

Provide a playable keyboard-controlled Tetris game delivered as one well-organized HTML file, with embedded application HTML, CSS and JavaScript. Players can open the same file directly or serve it over HTTP(S), including localhost. The project demonstrates development within SDD Man Demo; the repository currently contains guidance and skills, not a delivered game.

## Complete intended scope

- A classic playable baseline: a 10-column by 20-visible-row board, the seven tetrominoes, movement, rotation, gravity, soft drop, line clearing, score, level, game over, restart and pause/resume.
- Modern rules: hold, next-piece preview, ghost, 7-bag randomization, wall kicks and hard drop. The default general lock delay is one full gravity interval after becoming grounded; hard drop follows it unless immediate hard-drop locking is selected.
- Conditional ghost visibility, hiding projected positions above the board's midline and showing them again below it.
- Penetrator activation by hotkey, consuming one of at most three bonuses. A four-line clear awards one bonus. The activated piece passes through occupied cells without destroying them, must lock wholly inside the board without overlap, and is entirely white. Ordinary pieces use no white. Hard drop and the ghost share its lowest valid landing calculation. The information panel shows one symbol per available bonus.
- Six settings: ghost visibility (hide/dynamic/show), hard drop, next preview, hard-drop lock policy (general/immediate), general lock policy (failed downward attempt/full interval), and hold. Session-only configuration is a useful delivered increment before persistence.
- In HTTP(S), per-setting restoration uses valid explicit URL value, saved cookie, then default. Explicit valid URL values are saved even when equal to defaults. Changes update the setting's cookie and query parameter; omitted parameters retain saved values. In local-file mode, automatic configuration uses defaults and memory only, without cookie access or configuration-query access or modification.
- A session leaderboard of the ten best completed-game scores. Each result preserves a stable ID, non-negative integer score and UTC completion timestamp.
- Manual Save and Restore at the bottom of settings, hidden with that panel. Save downloads JSON or SQLite containing all six settings and the leaderboard; Restore accepts either and replaces both only after complete validation. JSON has a version, results, settings and a provided JSON Schema. SQLite has `high_scores(id, score, timestamp)`, uniquely keyed `settings(key, value)` rows and metadata format versioning. A failed restore shows an error and leaves results, settings, cookies and URL unchanged. Successful restore updates cookies/query in HTTP(S), and memory only in local-file mode.

## Interface and execution constraints

Panels appear left to right: leaderboard, game info and controls, board, settings, controls/help. Keep the board centered as optional panels are toggled, and keep the board and game info visible. Individual leaderboard/settings/help toggles sit above exactly two clickable gameplay buttons at the information panel's bottom: Start/Restart on the left and Pause/Start on the right. Other gameplay actions use the keyboard. Settings, visibility toggles and file actions are interface controls rather than additional gameplay buttons.

Detect execution mode with `location.protocol`. Verify both direct `file:` and served execution in Chromium and Firefox under the repository browser policy. No application backend, player-side package installation or build step is required. External dependencies are limited to widely used JavaScript libraries from established CDNs. CDN availability may be needed for SQLite operations; the core game and JSON operations should work without it.

## Proposed choices for approval

These are recommendations, not recorded human decisions.

| Concern | Proposal and reason |
| --- | --- |
| Application approach | Plain JavaScript, Canvas for the board and HTML panels. This keeps the runtime artifact simple and directly openable. A DOM-cell board is viable but offers little benefit for this small moving grid; a UI framework adds dependency and delivery overhead. |
| Ghost at the midline | Hide if any projected cell is in visible rows 0-9; show only when all projected cells are in rows 10-19. A straddling ghost is hidden. |
| Lock timing | Grounded lateral moves and rotations do not restart the interval. Becoming airborne cancels it; a later landing starts a new full interval. Pause freezes gameplay clocks. Capture the gravity interval when grounding begins. |
| Penetrator safety and hold | Spend the bonus immediately, once per piece; disable hold until that piece locks. Descent can cross blocked rows, but each completed movement ends at a non-overlapping pose; the final resting pose is the lowest legal pose for its column and rotation. Sideways and rotation destinations must remain legal. |
| Result lifecycle | Natural game over records one result. Restart abandons an unfinished run without recording it. Restart resets earned bonuses. Equal scores sort by earlier completion timestamp, then stable ID. |
| Settings and invalid inputs | Default to ghost always shown, hard drop/preview/hold enabled, full-interval general locking, and hard drop following general locking. Invalid automatic values fall back independently to the next source; invalid restore content rejects the entire file. Lock-policy changes take effect on the next spawned piece, preserving the current piece's timer. |
| Narrow screens | Preserve panel order and board centering when space permits; use horizontal scrolling below a readable minimum rather than removing mandatory panels. This means the full layout cannot fit every narrow viewport simultaneously. |

Exact scoring, gravity progression, keyboard bindings, wall-kick tables, file limits and error contracts belong in the detailed specification. Proposed defaults there remain subject to any material gameplay decision raised during preparation.

## Delivery and non-goals

Deliver the simplest sensible playable baseline first, followed by bounded useful increments covering the complete scope. Choose pre-1.0 versions for intermediate releases, targeting 1.0.0 for the verified complete scope. Every completed, verified and integrated implementation phase produces a tagged GitHub release with release notes and the downloadable HTML artifact; later file-format releases also provide the JSON Schema. Tests, schemas and development documentation may be separate repository files; application HTML/CSS/JavaScript remain in the single HTML deliverable.

The target is a single-player browser game. Accounts, online rankings, backend synchronization, mobile gameplay buttons, desktop wrappers and browser-extension packaging are outside the supplied scope. Full network-independent SQLite operation is not an assumed requirement.
