# PCB3 evidence: composer coverage for PR 2862

Evidence only. No product change is included in this report.

Upstream pull request: https://github.com/backnotprop/plannotator/pull/2862
Fork head branch: `global-comment-composer` on `iurysza/plannotator`
Captured: 2026-11-08

## Frozen revisions

| Role | SHA | What it is |
| --- | --- | --- |
| Before | `72b00e0dd6b9a5a6f87139ff9b0161eeac1ad0e5` | PR base. Detached worktree at this commit. |
| After | `b73a7c306c5ed9a2a89848485ac7a6ff05014d8f` | PR head (`chore: merge upstream main`). This checkout. |

`git merge-base` of the two SHAs is the base SHA. The after tree already contains that base, so the pair is the PR's own diff (36 files, +1178/−122 at capture), not a comparison against an older release and not against later commits on upstream `main`.

GitHub metadata at freeze time: state OPEN, title "Give global comments the reply composer", author iury souza. `baseRefOid` and `headRefOid` matched the table above. The same fetch reported `mergeable: CONFLICTING` against the upstream `main` that has moved since this head. That conflict is outside this before/after story.

The Test and Security Scanning rollup fetched with that metadata showed the unit, DOM, OpenCode, and guides.show jobs succeeding, and Gitleaks, Semgrep CE, Trivy, and zizmor succeeding. CI was not re-run for this capture.

## What was compared

The reference is the normal Inbox reply composer: file attach, an image tile, a voice control, a saved draft, and Send. The goal under test is that same composer on every free-text reply or annotation in the Inbox, including Markdown, HTML and component documents, inline comments, global comments, guided review, code review, and attachment comments.

Voice in this product is dictation into the field. It is not an audio attachment. A microphone icon is reported only as a visible control. This capture did not record speech and did not deliver an audio file.

Fixed-choice controls (a pick-one question, thumbs, Skip, Mark up) are listed separately. They do not compose a free-text reply.

Read-only, guest, and linked-machine gates were not opened and were not relaxed.

## How the apps were run

Both SHAs were built with `bun run --cwd apps/inbox build` (after: 6370 modules; before: 6369). Each capture started `startInboxServer` with that build's `apps/inbox/dist/index.html`.

Scratch only:

- `HOME` and `PLANNOTATOR_DATA_DIR` under `/tmp/pcb3/run-{before,after}-*`
- `PLANNOTATOR_TEST_SANDBOX=1`, `PLANNOTATOR_BROWSER=/usr/bin/true`
- Updates, connect, phones, and notes off
- No `OPENAI_API_KEY`, no voice key file
- `inboxVoice` set to `{ on: true, engine: "api" }` so the mic control is allowed to render
- Light theme, code review on, tour and voice intro already dismissed
- Display name requested as `ledger-reviewer`; the page still showed a generated tater name. No personal name or mailbox appears.

Fixture, same on both sides: project `ledger`, a reply thread, a files thread with `notes/rollout.md`, `site/status-card.html` (a real `<article class="card">` with the button "Approve export"), `src/refresh.ts`, and `shot.png`, plus a guided review and a code review submitted through the local Inbox MCP, and a message with one choice question and one free-text question. `paste.png` (80×48) was the file chosen in the attach control.

Browser: Playwright Chromium, headless, `1440×900` and `390×844`, light color scheme, reduced motion, device scale 1. Launch flags included `--use-fake-device-for-media-stream` and `--use-fake-ui-for-media-stream`. Those flags never mattered: the mic stayed disabled and the server logged `voicePosts=0` on both runs.

Side-by-side page: [index.html](index.html).

## Coverage matrix

| Surface | Before capability | After capability | Media / paste | Voice recording / input | Draft behavior | Evidence |
| --- | --- | --- | --- | --- | --- | --- |
| Thread reply, wide | Inbox reply composer. Paperclip, image tile, mic, Send. | Unchanged. This is the reference, not a new surface. | File input exercised both sides (`paste.png`, state done). Clipboard paste was not driven. | Mic visible, `aria-disabled`, label "Dictate: not available here", note "Voice needs an OpenAI API key on this computer." No take, no transcript, no audio attachment. | Cancel, reopen: text and the attached file come back. Server reply draft. | `thread-reply-empty-*.png`, `thread-reply-media-*.png`, `thread-reply-voice-*.png`, `thread-reply-draft-*.png` |
| Thread reply, 390px | Same composer. | Unchanged. | Same file control is in the narrow reply. | Same blocked mic. | Same draft restore, including the file. | `thread-reply-empty-narrow-*.png` |
| Markdown global comment, wide | Plain popover. Placeholder "Add a global comment…", button Add. No paperclip, no mic. | Reply composer inside the popover: paperclip (`data-global-attach`), image tile, mic, Add. | After: `paste.png` attached from the file input, still listed after Add, and still present after Send (filename, upload path, and image preview in the stored reply). Before: no attach control, so Add and Send carry text only. Clipboard paste not driven. | After: same blocked dictation mic and API-key note. Before: no mic. Neither side recorded. | After: closing and reopening restores text and the file (server comment draft). Before: the popover restores text only, in memory, and has no file slot. | `markdown-global-empty-*-crop.png`, `markdown-global-media-*.png`, `markdown-global-voice-*.png`, `markdown-global-draft-*.png`, `markdown-global-staged-*.png`, `markdown-global-sent-*.png` |
| Markdown global, 390px | Plain sheet: textarea and Add. | Same rich composer as wide: paperclip, blocked mic, Add. | Not re-attached on the narrow sheet. The control is the same `data-global-attach` input. | After shows the blocked mic. Before has none. | Not re-opened on the narrow sheet. Wide draft behavior above is the one that was checked. | `markdown-global-empty-narrow-*-crop.png` |
| Guided-review global comment | Plain Add popover. No attach, no mic. | Same rich global composer as Markdown. | After: `paste.png` sits in the composer. Add-then-Send was not repeated here; the submit path is the one proven on Markdown. Before: no attach. | After: blocked mic. Before: none. | Not closed and reopened on this surface. The draft store key differs by scope; the Markdown reopen is the exercised case. | `guide-global-empty-*-crop.png`, `guide-global-media-*.png`, `guide-global-voice-*.png` |
| Code-review global comment | Plain Add popover. | Same rich global composer. | Empty composer shown. Attach was exercised on the guide global, which uses this same component. | After: blocked mic. Before: none. | Not reopened here. | `review-global-empty-*-crop.png` |
| HTML / component global comment | No Global comment control. Side-by-side and the 390px tool bar (back, pen, comments, pager, more) have no opener. | Still no opener. The provider is mounted in code, and the viewer button is not rendered because the pane passes `hideControls`. | No composer, so nothing to attach. | No control. | No composer. | `html-no-global-button-*.png`, `html-no-global-button-narrow-*.png` |
| HTML text selection | Popover "Add a comment…", Save, thumbs, small image button. Not the reply composer. | Unchanged. `data-global-composer` false, `data-global-attach` false, no mic. | The small image button is the old popover control. It was not used, and it is not the inbox upload path. | None. | Not checked. This popover is not the server comment draft. | `html-selection-empty-*.png` |
| Markdown selection comment | Popover "Add a comment…", Save. No paperclip, no mic. | Unchanged. | None. Viewer `allowImages` is false. | None. | Not checked. | `markdown-inline-empty-*-crop.png` |
| Message selection comment | Same plain popover. | Unchanged. | None. | None. | Not checked. | `message-inline-empty-*-crop.png` |
| Guided-review line comment | "Leave feedback…", "+ Add suggested code", Add Comment. | Unchanged. | None. | None. | Not checked. | `guide-line-empty-*.png` |
| Code-review line comment | Same line toolbar. | Unchanged. | None. | None. | Not checked. | `review-line-empty-*.png` |
| Code file line comment | Popover "Add a comment…", Save. | Unchanged. | None. | None. | Not checked. | `code-comment-empty-*-crop.png` |
| Image annotation | "Comment on this image…", Mark up, Comment. | Unchanged, including 390px. | None on the comment row. The picture itself is the attachment. | None. | Not checked. | `image-comment-empty-*.png`, `image-comment-empty-narrow-*.png` |
| Choice question | CSV / JSON / Other, Add note, Skip. Not a reply composer. | Unchanged. | Not a composer. | None. | Not a reply draft. | `question-controls-*.png` |
| Free-text question | "Type your answer…", Add note, Skip. Plain textarea. | Unchanged. | None. | None. | Not checked. | `question-controls-*.png` |

## What changed, in one paragraph

On Markdown, on a guided review, and on a code review, the global-comment popover changed from a plain textarea and Add into the reply composer: paperclip, file tile, dictation mic, and a server draft. Adding a global comment on Markdown with `paste.png` attached kept the file in the comments list, and Send wrote it into the stored reply (`Files for global comment … paste.png: /tmp/pcb3/…/inbox/uploads/…` plus the image preview). The same Add and Send on the base SHA wrote the sentence only. Thread reply already had the composer on both SHAs.

## Gaps

The all-surfaces goal is not done.

1. HTML and component documents still have no global-comment opener, wide or narrow. `AttachmentPane` renders `HtmlViewer` with `hideControls`, so the viewer's own "Comment" button (title "Add global comment") never mounts. The new provider wraps that pane and has nothing to open it. Proposed fix, not built: show that existing control in side-by-side and on the narrow tool bar, and point it at the composer the provider already supplies. Leave read-only, guest, and linked-machine behavior as it is. A linked machine already falls back to the plain popover in `GlobalCommentComposer`.

2. Every selection and line composer is still the old plain field, on both SHAs. That includes Markdown selection, message selection, HTML selection, code-file lines, guided-review lines, and code-review lines. HTML selection also shows the old popover image button, which is not the inbox file uploader and was not shown surviving a Send. Proposed fix, not built: host the reply composer from those entry points if every free-text annotation is in scope.

3. Image comments and the free-text question stay on their own plain textareas. No paperclip, no mic.

4. The microphone does not prove recording. With voice switched on and no API key, the product disables the button and explains why. This browser never started a take. Transcription posts audio to the local Inbox, which then calls an external model; that call was not made. Dictation remains separate from attaching an audio file, which this composer does not do.

## Exercised, simulated, untested

Exercised in the running app, both SHAs, same fixture and viewport:

- Open the thread reply, attach `paste.png` through the file input, see the tile, cancel, reopen, and see the text and file restored.
- Open the Markdown global comment. Before: type, Add, Send, and read a text-only reply. After: attach `paste.png`, Add, see the filename in the comments panel, Send, and read the file line plus the image preview in the stored reply.
- Reopen the Markdown global comment and see the draft (text only before; text and file after).
- Open guide and review global comments. Attach `paste.png` on the guide composer after the change.
- Drag-select `return early();` on the guide and on the review and open "Leave feedback…".
- Open a code-file line comment, a Markdown selection comment, and a message selection comment.
- Open the image comment row, wide and at 390px.
- Show the choice question and the free-text question. Answers were not submitted.
- Open the HTML card, confirm there is no Global comment button, then drag-select the sentence in the document and open the selection popover. Same popover on both SHAs.
- Open the 390px HTML tool bar and confirm it has no Global comment control.

Simulated:

- "Pasted image" means the paperclip's file input received `paste.png` via `setInputFiles`. The clipboard paste handler and an OS file drop were not driven.
- The voice control was made visible by the scratch setting `inboxVoice.on`. The disabled state is the real no-key state, not a styled mock. No audio was supplied to the app.

Not tested:

- A listening recording, a waveform, a transcript, or dictated text landing in the field.
- An audio attachment. The product path under test does not create one.
- Mod+Enter as its own shot. Add and Send were mouse clicks.
- Guest, read-only, and a thread owned by a linked machine.
- Shape-board comments, diagram part comments, video comments, the new-thread dialog, the code-review decision note, permission deny reasons, action-item notes, annotation edit fields, and the pulls quick reply. Those were not in the fixture. From the source, shape global comments still use the plain popover and are outside `GlobalCommentProvider`.
- Add-then-Send on the guide and review global composers. The file was shown staged in the guide composer. The survival proof is the Markdown global comment, which uses the same Add and the same feedback formatter.
- Clicking the HTML "Approve export" button as a pin. The composer that opened came from dragging the paragraph.

## Capture limits

- Screenshots are the running Inbox at the frozen SHAs. They are not a mock.
- Generated tater names differ between the two browsers. The document text, file names, and controls are the matched fixture.
- Scratch paths such as `/tmp/pcb3/…` appear in the sent feedback because that is where the upload was stored. They are not a personal mailbox.
- No video. A recording would repeat these stills and could not show a microphone take this browser session did not start.
- No product code was edited. Capture helpers stayed in the workspace and were not part of the product.
