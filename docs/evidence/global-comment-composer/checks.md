# Verification notes

The focused checks cover draft persistence and isolation, pasted and dropped files, failed uploads, file retention, native Undo, focus, and delivered file bytes. Dictation uses simulated recordings, not a live transcription service.

The demo uses scratch data. It shows attaching a screenshot and log, reloading the page, restoring the draft, staging the comment with Add, and delivering both files with Send.

Lint, builds and the external package smoke check completed successfully. The full repository suite is not green:

- Root typecheck reports the pre-existing nullable-text error at `packages/inbox/updates.test.tsx:275`, reproduced on the untouched starting revision.
- Baseline reruns reproduce several CLI, launch, certificate, installer and browser-timeout failures. Some other full-lane failures remain unclassified.
- Local Bun is 1.4.2. CI pins 1.4.4.

No unrelated fixes are included. The review base holds the starting upstream snapshot and review evidence. The feature branch contains no demo assets.
