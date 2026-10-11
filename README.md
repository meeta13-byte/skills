# Skills

Reusable OpenCode agent skills.

## Available skills

| Skill | Description |
| --- | --- |
| [`apk-security-audit`](./apk-security-audit) | 20-point security audit for an Android app, source project, or built APK. |
| [`webpage-security-audit`](./webpage-security-audit) | 20-point security audit for a website, web app, or backend API. |
| [`tailor-resume`](./tailor-resume) | Tailor a LaTeX resume to a job description without fabricating experience. |

The two audit skills are read-only by default: they report findings with `file:line` evidence
and propose fixes, but only edit code when asked.

`tailor-resume` ships with **no personal data** — your profile lives in a local `PROFILE.md`
that is gitignored. See its [README](./tailor-resume) for setup.

## Install

Copy a skill folder into your skills directory, for example:

```sh
cp -R apk-security-audit ~/.config/opencode/skills/
cp -R webpage-security-audit ~/.config/opencode/skills/
cp -R tailor-resume ~/.config/opencode/skills/
```

Then restart OpenCode (or start a new session) and invoke it by name, e.g.
"audit my Android app's security".