# NESA Software Engineering Year 12 Quiz Web App — Project Overview

A handoff document for recreating and improving this project in Claude Code.

---

## 1. What this is

A single-page quiz web application for Year 12 students studying NSW NESA Software Engineering 11–12 (2022 syllabus). Students answer multiple-format questions, retry the ones they got wrong, and generate a printable report they submit to their teacher. The teacher can add, edit, and delete questions through an in-app interface.

**Live deployment target:** GitHub Pages (public hosting, free)
**Hosting model:** Static files only — no backend, no database, no server
**Audience:** One Year 12 Software Engineering teacher and ~15 students

---

## 2. Current state — what exists right now

Two production-ready files sit in `/mnt/user-data/outputs/`:

- **`index.html`** (~44 KB) — the entire app: HTML, CSS, JavaScript, all inline
- **`question_bank.json`** (~32 KB) — 80 questions across 4 topics, fetched at runtime

These are the files the teacher uploads to a public GitHub repo with Pages enabled.

### Question bank structure

The JSON is shaped as:

```json
{
  "_meta": { "title": "...", "version": "2.0", "totalQuestions": 80, "topics": {...} },
  "questions": [
    { "id": "ssa1", "t": "ssa", "type": "mcq", "q": "...", "opts": [...], "ans": 1, "exp": "..." },
    ...
  ]
}
```

Each question has a `type` field driving how it renders and how the answer is checked:

| Type     | Required fields              | Marking                                              |
| -------- | ---------------------------- | ---------------------------------------------------- |
| `mcq`    | `opts[]`, `ans` (index)      | Exact index match                                    |
| `multi`  | `opts[]`, `ans[]` (indices)  | Strict — must select every correct option and no incorrect ones. Show "Select all that apply" hint; do not reveal the number of correct answers |
| `tf`     | `ans` (boolean)              | Exact boolean match                                  |
| `match`  | `pairs[]` (term/def tuples)  | All pairs correct. **Each definition may only be selected once** — disable it in other dropdowns once chosen |
| `order`  | `items[]` in correct order   | Exact sequence match                                 |
| `fillin` | `accept[]` (alt answers)     | Case-insensitive includes match                      |
| `short`  | `keywords[]`, `sample`       | Keyword inclusion ≥50% = pass (no AI)                |

Topics are keyed: `ssa` (Secure Software Architecture), `web` (Programming for the Web), `auto` (Software Automation), `proj` (Software Engineering Project). Counts are roughly 19, 20, 19, 22 respectively.

### App features as built

**Student flow:**
1. Enter name (free text, no validation, no persistence)
2. Pick topic chip or "All topics"
3. Random selection from the topic pool — full pool shuffled, one pass
4. After each question: instant correct/incorrect feedback with explanation
5. End-of-session summary with correct/total/percentage
6. "Retry wrong answers" button starts a fresh quiz over only the wrong ones
7. "Generate report" produces a printable HTML overlay; native browser print → PDF
8. Report shows every question, student's answer, correct answer (if wrong), score

**Teacher flow:**
1. Open the site with `#teacher` appended to the URL (e.g. `yoursite.github.io/#teacher`) — there is no visible teacher button on the student home screen
2. The teacher dashboard appears directly; no PIN (a client-side PIN on a public static site is trivially bypassable, so it was removed in favour of an unlisted URL)
3. View filterable question list (by topic, by type)
4. Add/Edit/Delete questions via a modal with type-specific form fields
5. Export updated `question_bank.json` (download), import a `.json` (upload)
6. Re-upload the exported file to GitHub to push updates to students

**No persistence between sessions for students.** Everything lives in JS memory for one sitting. The report is the record.

### Tech specifics

- Pure vanilla JS — no framework
- Pure CSS — no Tailwind, no build step
- Fonts: Fraunces (display serif) + DM Sans (body) + DM Mono (labels), all from Google Fonts CDN
- Light theme — cream/off-white palette, dark ink for primary buttons, topic-coloured accents
- Drag-and-drop ordering uses native HTML5 drag events (works desktop, not great on touch — known limitation)
- No localStorage at all — teacher access is via an unlisted `#teacher` URL, not a stored PIN
- No external API calls — fully offline-capable after first load (fonts cache)

---

## 3. How we got here — design decisions log

Worth understanding the reasoning behind the current shape, especially because several earlier directions were abandoned for valid reasons.

### Iteration history

**v1 — Inline question bank, localStorage-everything**
Built first as a single HTML with questions hardcoded inside the script and full student progress saved to `localStorage` keyed by student name. Teacher dashboard read from the same localStorage. Worked, but only on one device per student — the cross-device requirement broke it.

**v2 — Export/import progress JSON files**
Added per-student progress export/import to work around the device-locked storage. Acceptable but clunky for teenagers to manage. Still single-device by default.

**v3 — Split question bank into separate JSON file**
Moved questions out of the HTML into a fetched `question_bank.json`. Made question maintenance much cleaner. This is when "the teacher edits the JSON in GitHub" became a real workflow.

**v4 — Considered Google Sheets / Firebase / OAuth**
Spent a long discussion mapping requirements (real cross-device progress, teacher dashboard of all students, no student impersonation). The architecturally correct answer was Google Sheets + Google SSO, but it required IT approval the school may not have given. Held off until IT was consulted.

**v5 (current) — Generate-report model**
After IT asked "does the teacher actually need to track this themselves?", the answer was no — students submitting their own report works. This removed every hard problem at once:
- No database needed
- No authentication needed
- No cross-device sync needed
- No backend needed
- Just static files + browser memory + print to PDF

This is genuinely the right architecture for this user's actual requirements. Don't rebuild it as a database-backed app unless those requirements change.

### Things deliberately NOT included

- **AI marking of short answers.** Earlier versions used Claude API for short answer feedback. Removed because (a) requires API key embedded in public repo, (b) keyword scoring is "good enough" for self-assessment and the student sees the sample answer anyway, (c) no API key management hassle.
- **Student progress persistence.** Tried it. Adds complexity for marginal benefit when reports do the job. Don't reintroduce unless asked.
- **Teacher dashboard of student results.** Replaced by reports submitted via Google Classroom / email — whatever the school already uses.
- **Question retry across sessions.** Within a session yes, across sessions no — would require persistence.
- **Accounts / login / SSO.** Considered, dropped.
- **AI question generation.** Never built. Could be a future feature using Claude API in the teacher view.

---

## 4. Adjacent artifacts that exist (not part of the deployed app)

The teacher accumulated several supporting documents over the conversation. These are NOT part of the app but live alongside it:

- `NESA_Software_Engineering_Project_Lessons.docx` — 3 × 75-min lesson plans for the Software Engineering Project topic, with their own embedded quiz questions and formative tasks. Pre-dates the quiz app.
- `NESA_SE_Quiz_Questions_and_Answers.docx` — earlier 38-question doc.
- `NESA_SE_Quiz_80_Questions_and_Answers.docx` — current 80-question reference doc with answers and explanations.
- `SSA_Quiz_Questions.pptx` — Secure Software Architecture questions as a slide deck, with answers in speaker notes. Designed for in-class delivery rather than self-study.

Treat these as outputs of the same project but separate deliverables.

---

## 5. Recreating this in Claude Code

### Quick start for Claude Code

```
Project: NESA Software Engineering Year 12 Quiz Web App
Stack: Static HTML/CSS/JS only — no framework, no build step
Hosting: GitHub Pages (public repo)
Files needed: index.html, question_bank.json
Goal: A single-file quiz app students use to practise, with a report
      they submit to the teacher. Teacher can edit questions via an
      in-app form and export an updated JSON to push to GitHub.
```

### Recommended Claude Code workflow

1. Initialise a new repo with just two files
2. Build `question_bank.json` first — it's the data model, everything else follows
3. Build `index.html` in three passes:
   - Pass 1: home screen + load JSON + render one MCQ question end-to-end
   - Pass 2: all six question types + retry flow + report generation
   - Pass 3: teacher view + modal editor + export/import
4. Test by serving locally with `python -m http.server` or `npx serve`
5. Deploy via `gh-pages` branch or main + Pages settings

### Files structure

```
/
├── index.html         # The entire app
├── question_bank.json # All questions
├── README.md          # Optional: brief teacher-facing docs
└── .gitignore         # Optional
```

That's it. Nothing else needed.

---

## 6. Architectural improvements worth considering

The current app works and ships, but several improvements would meaningfully raise the quality if the next iteration has time. Listed in priority order.

### 6.1 Migrate to a real frontend toolchain (high value, low risk)

The current vanilla approach made sense when this was scoped to one HTML file uploaded to GitHub. As the teacher might want to maintain this over years and the codebase grows, vanilla becomes harder to work with.

**Recommended stack:**
- **Vite** for build tooling — instant dev server, fast HMR, single command to build for GitHub Pages
- **Vanilla JS + Web Components** OR **Preact** (3 KB) — keep the bundle tiny; Preact gives JSX without React's weight
- **TypeScript** — the question bank shape is well-defined; types catch the kinds of bugs that have happened in this project repeatedly (e.g. forgetting `accept[]` is required for fillin)
- Output is still a single `index.html` + `question_bank.json` deployable to GitHub Pages

**Why bother:**
- Module imports instead of one 1000-line file
- Real type safety on the question schema
- Component-based UI makes adding question types straightforward
- HMR makes development genuinely fast
- Build still produces static files — GitHub Pages workflow unchanged

### 6.2 Schema validation on the question bank (high value, low effort)

The teacher edits JSON via the in-app form, then exports and uploads to GitHub. If they hand-edit the JSON, one wrong shape silently breaks the app at runtime with no good error.

**Solution:** Use Zod (or similar) to validate `question_bank.json` on load. Show a meaningful error UI listing exactly which question is malformed and why. ~50 lines of code, massive UX improvement when things go wrong.

### 6.3 PWA install + offline support (medium value, low effort)

The app is already 100% client-side after initial load. Adding a service worker + manifest makes it installable on student devices and works offline indefinitely.

- Add `manifest.json`
- Cache `index.html` + `question_bank.json` + fonts with a service worker
- Students install it to their phone home screen, use without wifi

Vite has a first-party PWA plugin that does this in 10 lines.

### 6.4 Touch-friendly drag and drop (medium value, medium effort)

Native HTML5 drag events don't fire on iOS/iPadOS. The order question type is broken on iPad and iPhone, which is a real problem in classrooms.

**Solution:** Replace native drag/drop with `@dnd-kit/core` (works on both touch and mouse, ~20 KB, well-maintained). Alternatively, replace ordering with click-up/click-down buttons next to each item — uglier but works everywhere.

### 6.5 Report as proper PDF (medium value, medium effort)

Current "Generate report" uses browser print → save as PDF. Works but:
- Page breaks are unreliable
- Header/footer styling varies by browser
- No control over filename
- Some students/parents struggle with the print → PDF dialog

**Solution:** Use `jsPDF` or `pdfmake` (both around 50 KB) to build a real PDF in JS, named `firstname_lastname_quiz_report_DATE.pdf`. Predictable layout, one click to download. Better student/parent experience.

### 6.6 Question types worth adding (low priority, content-driven)

The current six cover most syllabus question shapes, but the syllabus also has:
- **Code reading** — show a code snippet, ask what it outputs / identify the bug
- **Diagram labelling** — annotate parts of a structure chart or DFD (would need image support in questions)
- **Multi-select** — "select all that apply" (genuinely different from MCQ)

These are nice-to-haves and only worth building if a teacher specifically asks.

### 6.7 If the teacher does want analytics later (high effort, high value)

The decision to drop the database was right for current needs but locks the teacher out of seeing trends. If they ever ask "which questions does my class get wrong most often?", the answer is: rebuild as a Firebase or Supabase app.

**Suggested stack if this becomes a requirement:**
- Frontend: same Vite + Preact as recommended above
- Backend: Supabase (Postgres + auth + row-level security — generous free tier, no server to maintain)
- Auth: Magic link email or school SSO via Supabase Auth
- Data model: `questions`, `students`, `quiz_sessions`, `answers` tables
- The teacher view becomes a real analytics dashboard with per-student and per-question stats

This is a much bigger build. Don't do it pre-emptively.

### 6.8 Other small polishes

- Keyboard navigation for quiz answers (number keys to select MCQ options)
- Dark mode (CSS variables make this nearly free)
- A teacher-facing question bank diff tool — "show me what I'm about to change" before they re-upload
- Session resume — if the student accidentally closes the tab, prompt to resume on reload (would need localStorage)
- Better feedback on short answers — keyword matching is rough; a simple sentence-level "did your answer mention X" string check works better than substring matching for common cases like "test cases" vs "testing"

---

## 7. Constraints worth remembering

**The school IT environment is restricted.** The teacher cannot necessarily use:
- Google Cloud Console / Sheets API
- Firebase / Supabase
- Anthropic API (requires key, gets visible in public repo)
- Anything requiring student account creation
- Anything that stores PII outside the school's existing systems

**GitHub Pages must be public.** The teacher's account does not have GitHub Enterprise with private Pages. This is why no API keys can be embedded in the deployed code, and why all student names are kept out of the repo entirely.

**Students may use a different device each session.** No assumption that the same browser is used twice. This is why progress is intentionally NOT persisted.

**The teacher is not a developer.** The workflow has to be: "edit questions in the app, click Export, drag the file to GitHub, click Commit". Nothing requiring command-line tools, build steps, or understanding of git beyond the GitHub web UI.

---

## 8. Acceptance test — does the rebuild work?

A successful rebuild passes this manual checklist:

- [ ] Open `index.html` directly from disk → app loads, can take a quiz, all six question types work
- [ ] Matching question: once a definition is selected in one dropdown, it disappears (or is greyed out) from every other dropdown — students cannot pick the same answer four times
- [ ] Open via GitHub Pages → identical behaviour
- [ ] Take a quiz with deliberate wrong answers → end-screen shows correct count, lists wrong questions, retry button restarts with only those
- [ ] Generate report → printable view with student name, date, each question, each answer, correctness, score
- [ ] Print dialog → can save as PDF
- [ ] Student home screen shows NO teacher button anywhere
- [ ] Visiting the site with `#teacher` in the URL opens the teacher dashboard; without it, only the student view is reachable
- [ ] Add a new question of each type → appears in the list
- [ ] Edit an existing question → changes persist
- [ ] Delete a question → it's gone
- [ ] Export `question_bank.json` → file downloads with all current questions
- [ ] Import that file in a fresh browser session → questions restored
- [ ] Mobile portrait viewport (375px wide) → all screens usable, no horizontal scroll
- [ ] Light keyboard test → can tab through buttons, no keyboard traps

---

## 9. One-paragraph summary for fast onboarding

A single-file vanilla-JS quiz web app for Year 12 NESA Software Engineering. Students enter their name, choose a topic, answer 80 questions across six types (multiple choice, true/false, matching, drag-to-order, fill-in-the-blank, short answer), retry the ones they got wrong, then generate a printable PDF report they submit to their teacher. The teacher reaches a hidden dashboard via an unlisted `#teacher` URL (no PIN — a client-side PIN on a public static site is bypassable, so an unlisted URL is used instead) to add, edit, and delete questions through forms, then exports a `question_bank.json` file they upload to a public GitHub Pages repo to push updates. No database, no accounts, no backend — deliberately. The architecture is right for the user's actual needs (a single teacher with one class) and the constraints of their school IT environment (no Google Cloud Console access, no API keys allowed in public repos, no student data outside the school's existing systems). For improvements, prioritise: migrate to Vite + TypeScript + Preact, add Zod validation, make it a PWA, replace HTML5 drag/drop with `@dnd-kit` for touch support, and generate real PDFs with `jsPDF`. Only rebuild it as a database-backed app if the teacher requests cross-session analytics.
