# Guitar Class — Project Briefing

## What this is

A single-page private learning platform for guitar students. Students sign in to view assigned lesson resources, upload practice videos, get teacher/facilitator feedback, and use built-in practice tools (fretboard scale viewer, chord library, strumming/plucking patterns, chord progression trainer). Admin manages everything from one panel.

Live: https://pdchm.github.io/guitar-class/
Repo: https://github.com/PDCHM/guitar-class

## Tech stack

- **Frontend:** one `index.html` file. Vanilla JS as an ES module, inline SVG icons, inline `<style>`. No framework, no build step.
- **Auth:** Firebase Auth (email/password + Google `signInWithPopup`). Admin uses a separate local password gate, **not** Firebase Auth.
- **Database:** Firestore. Collections: `students`, `uploads`, `songs`, `categories`, `lessons`, `seasons`, `facilitators`, `announcements`, `challenges`, `songRequests`, `practiceLogs`, `lessonPlans`, `attendance`, `chordProgressions`, `practicePatterns`, `toolSettings`, `settings`.
- **Media uploads:** Cloudinary (videos, images, audio, PDFs). Single upload preset, auto-detects resource type.
- **Hosting:** GitHub Pages from `main` branch — every push to `main` deploys.

## Key constants

| Thing | Value |
|---|---|
| Firebase project ID | `guitar-class-app-ccaf3` |
| Cloudinary cloud name | `dyg9hxapf` |
| Cloudinary upload preset | `guitar-class` |
| Admin password | `pd555` (hard-coded client-side — see Known Issues) |
| Live URL | https://pdchm.github.io/guitar-class/ |

Admin can also be unlocked by tapping the nav logo 5× or typing `admin` on the keyboard.

## What's built

**Pages:** library (public landing), upload (legacy anonymous submit), admin, facilitators, student-auth, learner (signed-in main).

**Learner tabs:** My Resources, Submit, Progress, Practice (with Scales / Strumming / Plucking / Progressions sub-tabs), Chords, Class Feed, Community, Practice Log.

**Admin sections:** Add Resource, Resource List, Categories, Lessons, Students, Facilitators, Announcements, Practice Tools, Lesson Planner, Attendance, Bulk Message, Weekly Challenge, Song Requests, App Settings, Recordings, Seasons (graduate/archive flow).

**Practice tools:** canvas-rendered fretboard with 9 scale groups (pentatonic, modes, blues, major/minor, triads, chromatic, diatonic, diminished, CAGED — standard positions). Chord library with finger-diagram canvases. Strumming/plucking pattern cards (admin-editable with arrow notation). Chord progression trainer with Web Audio metronome + simple strum synthesis.

**Lesson labels are category-aware:** Basic 1–10, Int. 1–10, Adv. 1–7, or generic Lesson 1–10 when no category. The same string is used for `students.completedLessons` checkboxes, the Progress tab, the Submit dropdown, and the welcome-header quick stats.

**Publish toggles** on every practice tool — scale tools, strumming/plucking patterns, chord progressions all have `published` flags; unpublished items are hidden from students.

**Pending-student notifications:** count badge on the Students quick-nav button + dismissible orange banner below the admin stats row.

## Known issues / in progress

- **`firestore.rules` is in the repo but NOT safe to deploy as-is.** The rules restrict `students/{uid}` writes to the owning user, but admin actions (approve, set category, mark lessons complete) run without Firebase Auth — they'd be rejected. Deploy only after moving admin to Firebase Auth + custom claims, or to a Cloud Function backend.
- Admin password (`pd555`) is in client JS — anyone viewing source can read it. Defensible only because the site's user base is trusted; treat as low-security gate.
- **GitHub PAT lives in Firestore `settings/app.githubToken`** so the PDF uploader can commit to `/pdfs/` via the GitHub Contents API. Any authenticated reader of `settings/app` can fetch it. Use a **fine-grained PAT** scoped to *this repo only*, with **Contents: Read & Write** and nothing else. Rotate if anyone with admin-gate access leaves the trust circle.
- `signInWithPopup` can be flaky on mobile Safari / iOS Chrome. No `signInWithRedirect` fallback is implemented.
- Migrating existing students whose `completedLessons` use old `"Lesson 1"`-style strings to the new category-prefixed format hasn't been done; their old completions won't display until re-ticked.
- BADGE_DEFS icons, mood selector emoji, leaderboard medals (🥇🥈🥉), and toast strings still use emoji intentionally — per the icon-replacement spec, only UI/button icons were converted to inline SVG.
- The bare `console.log` in `drawFretboard` was removed in `e4f5908`; if you re-add temporary debug logs, remove them before pushing.

## Coding rules

1. **Single HTML file.** Everything lives in `index.html`. No separate JS/CSS files. No build step. No bundler.

2. **Run `node --check` after every JS edit.** Extract the module script and parse it:
   ```bash
   awk '/<script type="module">/{flag=1;next} /<\/script>/{flag=0} flag' index.html > /tmp/script.mjs && node --check /tmp/script.mjs
   ```
   Don't push without `SYNTAX OK`.

3. **All HTML-invokable functions must be on `window`.** Inline `onclick`/`onchange` handlers execute in global scope. ES module module-scoped bindings are NOT visible there. Always write `window.foo = (...)` for anything called from HTML, and reference handlers as `onclick="window.foo()"` for clarity. (Bare `foo()` calls between functions inside the module work via global lookup once assigned, but be explicit at the HTML boundary.)

4. **No composite Firestore queries.** Use single-field `where()` only — avoids needing composite indexes (which require Firebase Console setup the user hasn't enabled). For multi-field filtering, fetch broader and filter client-side.

5. **Inline SVG only for icons.** No icon fonts, no PNG/JPG icons, no external SVG files. Use stroke-based Heroicons-style with `currentColor` and consistent sizing (nav 18px, button 14–16px, section 20px, hero 24–28px). Emoji is allowed only for: user-generated content, mood selectors, BADGE_DEFS, leaderboard medals, and toast messages.

6. **Don't break the working tree.** Verify with `git status` before assuming a commit is needed; if there's nothing to commit, say so rather than amending the prior commit just to satisfy a queued command.

7. **Auto-yes on destructive/shared-state actions.** User has pre-authorised commit/push/amend/force-push for their own recent commits — execute queued git commands without re-asking. (Saved in user memory.)
