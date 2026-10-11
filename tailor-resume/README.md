# tailor-resume

Tailor a LaTeX resume to a specific job description, without fabricating anything.

## What's in here

| Path | Purpose |
| --- | --- |
| `SKILL.md` | The skill: workflow, truth filter, style rules, power verbs, compile/verify steps. |
| `PROFILE.template.md` | Blank form for the user's own data. Copy to `PROFILE.md` and fill it in. |
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

## Two things make this work

**1. Attestation labels.** `PROFILE.md` marks every claim `[attested]` or `[keyword-depth]`.
Those labels are binding, not advisory:

- `[attested]` → can become a bullet.
- `[keyword-depth]` → skills line only, **never** a bullet, even if the JD asks for it
  directly. The skill reports it as an unfilled gap instead.
- Unlabelled → treated as keyword-depth until confirmed.

**2. The truth filter.** The skill is forbidden from adding employers, metrics, awards,
ratings, or experience the profile doesn't attest, and must report every JD requirement it
declined to fake, plus every keyword-depth skill it refused to bullet.

## Privacy

`PROFILE.md`, `positioning.md`, and `tailored/` are gitignored by the repo's `.gitignore`.
Don't commit them — the template is committed precisely because it is empty, so the filled-in
form can stay local.

## Other design notes

- **Templates are locked.** Macros, margins, and section names are frozen because they fit
  the page budget and match what ATS parsers expect. Only content changes.
- **Compiling is verified.** 1-page must produce exactly 1 page with 0 overfull boxes; if it
  spills, trim content rather than shrinking fonts.