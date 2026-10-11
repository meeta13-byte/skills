# tailor-resume

Tailor a LaTeX resume to a specific job description, without fabricating anything.

## What's in here

| Path | Purpose |
| --- | --- |
| `SKILL.md` | The skill: workflow, truth filter, style rules, power verbs, compile/verify steps. |
| `PROFILE.template.md` | Template for your own data. Copy to `PROFILE.md` and fill it in. |
| `templates/resume-1page.tex` | 1-page LaTeX template with `%%FILL:` placeholders. |
| `templates/resume-2page.tex` | 2-page LaTeX template with `%%FILL:` placeholders. |

No personal data ships with this skill. Your identity, employers, dates, metrics, and
links live only in your local `PROFILE.md`.

## Setup

```sh
cp PROFILE.template.md PROFILE.md   # then edit it — this file is gitignored
```

Requires `pdflatex` (TeX Live or MiKTeX) to compile.

## Usage

Ask the agent to tailor your resume, paste a JD, or run `/tailor-resume`.

## Design notes

- **Truth filter is the point.** The skill is explicitly forbidden from adding employers,
  metrics, awards, ratings, or experience the profile doesn't attest, and must report the
  JD requirements it declined to fake.
- **Templates are locked.** Macros, margins, and section names are frozen because they fit
  the page budget and match what ATS parsers expect. Only content changes.
- **Compiling is verified.** 1-page must produce exactly 1 page with 0 overfull boxes; if it
  spills, trim content rather than shrinking fonts.

## Privacy

`PROFILE.md`, `positioning.md`, and `tailored/` are gitignored by the repo's `.gitignore`.
Don't commit them.