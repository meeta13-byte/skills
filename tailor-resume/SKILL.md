---
name: tailor-resume
description: |
  Generate a tailored LaTeX resume for a specific job description. Reads a local profile reference file, picks the right base template (1-page / 2-page), then customizes summary, skills ordering, and bullet emphasis to mirror the JD's keywords — without fabricating experience. Compiles the result with pdflatex and verifies the page count.

  TRIGGER when: user pastes a job description, links to a job posting, asks to "tailor" / "customize" / "adapt" / "tailor my resume" for a specific role, or invokes /tailor-resume directly.
---

# Tailor Resume Skill

Customize **the user's** resume to a specific job description. Pick the right base, apply targeted edits, **never fabricate**.

## Data source of truth

Personal data lives in **`PROFILE.md`** (next to this `SKILL.md`, gitignored, never committed or pushed). Start from `PROFILE.template.md` — a blank form the user copies and fills in.

- If `PROFILE.md` is missing or looks stale, **ask the user for their resume/CV or point at the file** before editing anything. Do not invent identity, contact details, employers, dates, or metrics.
- `PROFILE.md` is authoritative. If a template and `PROFILE.md` disagree, `PROFILE.md` wins — re-port from it before compiling.
- Never write personal data into the templates, the skill, or any committed file.

### Honor the user's attestation labels

`PROFILE.md` marks claims **`[attested]`** or **`[keyword-depth]`**. These are binding:

- **`[attested]`** — may appear as an experience or project bullet, and on skills lines.
- **`[keyword-depth]`** — may appear on **skills lines only**. Never write it as an
  experience or project bullet, even when the JD asks for it directly. Instead, report the
  gap to the user (see Report format item 4).
- Anything **unlabelled** is ambiguous: treat it as keyword-depth until the user confirms.
- **Always tell the user in the report** which keyword-depth skills you deliberately left off
  the bullets, and why.

## Quick workflow

1. **Confirm the JD source** — pasted text, file path, or URL? If URL, fetch it.
2. **Load `PROFILE.md`** in full. Note the primary track (e.g. Full-Stack + DevOps/Cloud first, AI second) and the attestation/conflict notes if present.
3. **Pick the base template** — 1-page by default; 2-page only if the JD/recruiter asks or the user wants the long version.
4. **Read the chosen base `.tex` file in full.** It ships with placeholder content; if placeholders survived a previous run, re-port from `PROFILE.md` first.
5. **Plan the edits** — summary, skills ordering, bullet emphasis. Show a 5-line plan before editing if non-trivial.
6. **Create the role folder** `tailored/<company-slug>/<role-slug>/`. Save the JD verbatim to `jd.md`. Copy the base to `resume.tex` in the same folder.
7. **Apply edits** to `resume.tex`.
8. **Compile in place with pdflatex**, verify the page count holds.
9. **Write `notes.md`** with what changed and any gaps flagged.
10. **Report to the user.**

Never edit the base templates during tailoring — always work on a copy in `tailored/<company>/<role>/`.

## Base templates

Both live in `templates/` next to this file:

```
templates/resume-1page.tex   ← 1-page (default)
templates/resume-2page.tex   ← 2-page (only when explicitly requested)
```

The LaTeX macros, margins, and accent color (`#1F6B3A`) are proven to fit 1 page when full — keep them.

## Track emphasis (what to lead with per JD)

The default profile is **Full-Stack + DevOps/Cloud first, AI second**. One template, re-emphasized per JD:

- **DevOps / Cloud / Platform JD** (Docker, K8s, AWS, CI/CD, SRE): lead skills with the Cloud & DevOps line; lead projects with infra-heavy ones (AWS bullets first).
- **Backend / Full-Stack JD**: lead with Backend & APIs + Frontend lines; lead with the multi-tenant / real-time / webhook-integration project, then the multi-platform one.
- **Mobile JD** (React Native / Expo): lead with the app line; lead with the mobile + app-store project.
- **AI / LLM JD**: lead with the AI & Automation line; lead with agentic / RAG / embedding projects. Check the Truth filter first — AI depth is usually project-level, not research/training-level.
- **Startup vs Big Tech tone**: startup JDs → keep the role ask in the summary and shipping-speed metrics; Big Tech → drop the role ask, lead with scale/architecture framing.

If the JD is ambiguous between tracks, ask the user one question. Don't guess silently.

## What to customize per JD (in priority order)

### 1. Summary (highest leverage, ATS reads this first)

- 2–3 short sentences on 1-page. Mirror 3–5 keywords from the JD's requirements — but only ones the user actually has.
- Base claim shape: "Full-Stack & DevOps engineer building production web, mobile, and cloud-native systems: [JD-relevant capabilities]."

### 2. Technical Skills (second-highest ATS impact)

- Reorder groups so the JD's primary stack appears first; within a group, lead with terms the JD names.
- Don't add tech the user doesn't have (see Truth filter). Don't drop existing tech wholesale — surface priority, keep depth.

### 3. Experience bullets

- Lead with the entry whose work matches the JD most closely (strongest for backend/DevOps/full-stack; a different one for AI/LLM or mobile JDs). Keep community/leadership roles last.
- Re-emphasize bullets per JD, don't rewrite facts.

### 4. Projects (the core of most resumes — most room, most signal)

- Pick 2 projects on 1-page, 4–5 on 2-page, ordered by JD relevance (see Track emphasis).
- Reorder bullets within a project to surface the JD-relevant angle first. Don't change project URLs or fundamental tech stacks — those are real.
- Don't add projects that don't exist.

### 5. Achievements

- Lead with ambassador / community-program numbers for AI and ecosystem JDs; workshop-scale numbers for DevRel JDs; certifications last.

## Truth filter — DO NOT FABRICATE

Real interviewers probe everything on the page; one fabricated keyword sinks the screen.

**Hard nos — never add to a resume:**
- **Employment beyond the real entries.** Do not inflate duration, title, or scope, and never invent additional employers or clients. A campus community or student-club role must never be titled as employment at the organization's headline company.
- **Competitive programming claims.** Only cite what a public profile actually shows (solved counts, contests, rating, badges). Never write star ratings, contest ranks, or inflated solve counts.
- **AI/ML depth beyond what's attested.** RAG pipelines, agent frameworks, and model APIs are usually real at *project* level. Training, fine-tuning, or model-serving claims are usable **only** where `PROFILE.md` attests them, and fine-tuning only in the specific context it was done in. Still off-limits unless attested: published research, "ML engineer" as a primary title. Skills-line keywords with **no backing artifact** (vector DBs, Kafka, a self-hosted serving stack, data-science pipelines) may appear on the skills line, but **never as an experience or project bullet** — a recruiter clicking through will not find them. Always tell the user which claims are keyword-depth only.
- **Production orchestration at scale.** Practice-level K8s, tutorial Helm/ArgoCD, and "readiness" experiments are not production scale. Confirm with the user before writing any bullet that claims production use.
- **Made-up metrics.** Use the real numbers from `PROFILE.md` only. Approximate where the source is approximate; never invent new ones.
- **Timeline inflation.** Do not extend employment or graduation dates, and never convert "building projects since X" into "professional experience".

**Known data conflicts — verify with the user before relying on them:**
`PROFILE.md` has a **Truth filter notes** section with a "Conflicts to resolve with me" list.
Anything listed there is unresolved: **ask the user in this run**, use their answer, and offer
to write it back into `PROFILE.md` so the next run doesn't ask again. Typical entries:
- Any start date that postdates the present, or "Present" on an engagement that has ended.
- Role titles that differ between the resume, an older CV, and a portfolio.
- Awards or credentials that appear only on a portfolio with no other corroboration.
- Degree or program naming that differs between sources.
- Approved phrasing for sensitive claims (e.g. how to state years of experience).

If the JD requires something the user doesn't have, **state that gap explicitly in your report**. Don't silently add the missing keyword.

## Style and formatting rules (locked)

- **Template macros are locked** — do not modify `\resumeItem`, `\resumeSubheading`, `\resumeProjectHeading`, list-start/end macros. Only content changes.
- **Section names are canonical**: `Summary`, `Technical Skills`, `Experience`, `Projects`, `Education`, `Achievements`. ATS keyword-matchers look for these.
- **No em dashes** (`---`) in rendered text; no ` -- ` sentence breaks. Date ranges keep en dashes: `Oct 2025 -- Present`.
- **Project name → description** separated with `:`. Format: `\textbf{ProjectName}: Short Description $|$ \emph{stack list}`.
- **Accent color** `#1F6B3A`, `linkcolor` black. Don't change unless asked.
- **Achievements on 1-page needs `\item[]` after `\resumeSubHeadingListStart`** (latent template bug, keep it).
- **Tone**: professional, factual, keyword-rich, verb-led bullets — pick from the Power verbs table below, matched to the bullet's category. No startup-cringe voice.

## Power verbs (start every bullet with one, matched to what the bullet shows)

| Category | Power Verbs |
|---|---|
| Leadership & Management | Led, Supervised, Directed, Mentored, Coordinated, Managed, Oversaw, Spearheaded, Organized, Facilitated, Chaired, Delegated, Executed, Guided, Coached, Mobilized, Orchestrated, Piloted, Shaped |
| Strategy & Innovation | Developed, Devised, Conceptualized, Formulated, Designed, Initiated, Innovated, Pioneered, Crafted, Transformed, Restructured, Streamlined, Revamped, Reinvented, Improved, Reengineered |
| Results & Impact | Achieved, Exceeded, Accelerated, Delivered, Generated, Increased, Reduced, Enhanced, Boosted, Maximized, Expanded, Optimized, Strengthened, Amplified, Outperformed, Elevated, Drove |
| Collaboration & Communication | Collaborated, Partnered, Liaised, Communicated, Presented, Negotiated, Advocated, Facilitated, Influenced, Convinced, Advised, Consulted, Networked, Supported, Engaged, Coordinated |
| Analysis & Research | Analyzed, Assessed, Evaluated, Identified, Investigated, Researched, Measured, Quantified, Mapped, Examined, Diagnosed, Forecasted, Synthesized, Verified, Audited, Compared, Reviewed |
| Problem Solving & Decision Making | Resolved, Solved, Simplified, Prioritized, Streamlined, Eliminated, Determined, Diagnosed, Engineered, Rectified, Overhauled, Improved, Enhanced, Addressed, Overcame |
| Technical & Product Management | Built, Designed, Launched, Deployed, Implemented, Programmed, Configured, Tested, Automated, Integrated, Customized, Debugged, Upgraded, Modeled, Monitored, Documented |
| Customer & Stakeholder Focus | Advised, Educated, Trained, Onboarded, Supported, Counseled, Clarified, Demonstrated, Delivered, Guided, Resolved, Engaged, Retained, Strengthened relationships |
| Creativity & Marketing | Created, Conceptualized, Composed, Produced, Designed, Curated, Illustrated, Marketed, Promoted, Publicized, Storyboarded, Scripted, Developed campaigns, Innovated, Revitalized |

Don't repeat the same opening verb twice in a row on the page; a verb never overrides the Truth filter (a strong verb on a fabricated claim is still fabrication).

## Positioning & voice

If a `positioning.md` exists (local, gitignored), it is the approved voice for summaries, bios, and headlines — draw summary language from it rather than inventing framing. Typical shape:

- **Identity**: a systems thinker who owns delivery end-to-end and ships production-grade work, not demos.
- **One-liner**: "<Role> specializing in <domain>, built with <approach> and <delivery claim>."
- **Signature claims**: delivery speed vs standard timelines, automating repeatable work, real auth / monitoring / deployments.

On resumes, convert first-person voice to the factual third-person bullet style the Style rules require; first person is for LinkedIn About / cover letters only.

## Output paths and naming

```
tailored/<company-slug>/<role-slug>/
  ├── resume.tex     ← the tailored .tex
  ├── resume.pdf     ← compiled output
  ├── jd.md          ← job description, verbatim
  └── notes.md       ← changes summary + gaps flagged
```

Slugs are kebab-case and short: `Google DeepMind` → `google-deepmind`; `Software Engineer Intern (Cloud)` → `swe-intern-cloud`. If the target folder already exists, ask before overwriting (offer `resume.v1.tex` versioning).

## Compile + verify

Compile in place inside the role folder.

```sh
cd tailored/<company-slug>/<role-slug>
pdflatex -interaction=nonstopmode -halt-on-error resume.tex
grep -o 'Output written on resume.pdf ([0-9]* page' resume.log
grep -c '^Overfull' resume.log
```

- 1-page template must compile to **exactly 1 page**, **0 overfull boxes**. If it spills, trim a bullet or a summary line — don't shrink fonts or touch macros.
- 2-page must compile to exactly 2 pages, 0 overfull boxes.
- If pdflatex is missing, advise installing TeX Live or MiKTeX (`brew install --cask mactex` on macOS, `winget install MiKTeX.MiKTeX` on Windows).
- Clean up aux files before finishing: `rm -f resume.aux resume.log resume.out resume.synctex.gz`.

## Report format

After compiling, report:
1. **Base template** used and whether any content port was needed.
2. **Output path** (`tailored/...`).
3. **What changed** — 4–6 bullets max.
4. **What you did NOT add** because it would be fabrication — explicit list of JD requirements the user doesn't meet (CP ratings, published research, production K8s, years of experience, etc.).
5. **Which `[keyword-depth]` skills you kept on the skills line but refused to bullet**, with a one-line reason each.
6. **Page count + compile status** (and overfull-box count).
7. **Unverified items used** (awards, disputed dates, live links) and **conflicts you resolved** by asking — remind the user to verify before sending.

## When to ask before acting

- JD track is ambiguous (e.g. "platform engineer" — DevOps or backend emphasis?). Ask.
- JD requires more years of professional experience or a completed degree than the user has — flag it.
- JD demands something not in the inventory (Go, Rust, ML training, production K8s) — flag the gap and ask whether to apply anyway.
- `PROFILE.md` is missing or contradicts the resume on file — ask before editing.
- A JD demands a `[keyword-depth]` skill the user doesn't have attested — flag it, ask whether to apply it as a skills-line-only mention, and never bullet it without an explicit instruction plus an attestation.
- User pastes multiple JDs — ask which first, or do them sequentially.

Otherwise — pick the base, port if needed, edit, compile, report.