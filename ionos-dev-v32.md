# ionos-dev-v32 Implementation Notes

## Goal

Create `ionos-dev-v32` branch based on the **v9.0.3** upstream tag, carrying forward
only the still-relevant IONOS customisations from `ionos-dev` (which was based on v8.7.1 / stable31).

## Commands to create ionos-dev-v32

```bash
# 1. Start from the v9.0.3 tag
git checkout -b ionos-dev-v32 v9.0.3

# 2. Apply the one remaining IONOS frontend change
#    The cherry-pick will have a minor conflict in the Document_Loaded block
#    because v9.0.3 added Insert_Button logic after documentReady().
#    Resolve by keeping Hide_Menu_Item BEFORE documentReady():
git cherry-pick 24347e673991683fbc8d4a9dd272fb042e9aaffa

# If there is a conflict, the resolved block should look like:
#   } else if (args.Status === 'Document_Loaded') {
#       this.sendPostMessage('Hide_Menu_Item', { id: 'help' })   ← IONOS
#       this.documentReady()
#       if (loadState('richdocuments', 'open_local_editor', true) && !this.isEmbedded) {
#           this.sendPostMessage('Insert_Button', { ... })        ← upstream v9.0.3
#       }
#   }
git add src/view/Office.vue
git cherry-pick --continue
```

## What is already in v9.0.3 (no cherry-pick needed)

| IONOS change | Upstream location in v9.0.3 |
|---|---|
| `lib/TokenManager.php` — `userCanEdit($editoruid)` check | Merged via PR #4698 (`39610ba99`) |
| `lib/Listener/RegisterTemplateFileCreatorListener.php` — `userCanEdit()` guard | Merged via PR #4807 (`deab95d3e`), then further refactored with `loggedInUser()` pattern |
| `tests/lib/Listener/RegisterTemplateFileCreatorListenerTest.php` — IONOS unit tests | Merged via PR #4773 (`7fabdedb2`) |

## What is obsolete (do NOT backport)

| IONOS change | Reason |
|---|---|
| `src/public.js` — edit_groups guard for public shares (`f1db58f79`) | `src/public.js` was completely rewritten in v9.0.3. The backend `RegisterTemplateFileCreatorListener.php` already enforces the same restriction server-side via `loggedInUser()` + `userCanEdit()`. |

## Database migrations (run `occ upgrade` on deploy)

Two new migrations ship with v9.0.3 that alter the `richdocuments_wopi` table:

| File | Change |
|---|---|
| `lib/Migration/Version9000Date20250128212050.php` | `guest_displayname` column → VARCHAR(255) |
| `lib/Migration/Version10000Date20251217143558.php` | `version` column: INT/bigint → VARCHAR(1024), default `'0'` |

These are upstream changes; no IONOS-specific database changes are needed.

## Nextcloud compatibility

`appinfo/info.xml` in v9.0.3 requires Nextcloud **32** (`min-version="32" max-version="32"`).
The old `ionos-dev` targeted Nextcloud 31.
