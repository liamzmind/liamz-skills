# claude-skills

Six practical skills for Claude: decision coaching, working discipline, code audits, mobile release closing, a butler-style chat voice, and YouTube channel synthesis. Each skill is a folder with a `SKILL.md` that Claude loads when your request matches its description.

> Status: early releases. `release-close` and `yt-synthesis` are v0.1 and have had limited real-world testing. Feedback and issues are welcome.

## The skills

| Skill | What it does | Use it when you say |
| --- | --- | --- |
| [`decision-making`](skills/decision-making) | A six-phase method for big decisions (triage, frame, gather, stress-test, choose, run and review), with a review that separates a bad decision from bad luck. | "Help me decide...", "was that a mistake?", "I'm second-guessing this" |
| [`dev-rules`](skills/dev-rules) | Working rules for AI-assisted projects: clarify first, stay in scope, spec before building, verify instead of assuming, report honestly, define done. | "Follow my dev rules", "plan this before coding", "tell me what you verified" |
| [`code-audit`](skills/code-audit) | A read-only, evidence-based audit of a codebase, ranked by severity and leverage, with `path:line` evidence and confidence levels. Never edits files. | "Audit this repo", "what is wrong with this code?" |
| [`release-close`](skills/release-close) | Gated audit-then-close for Android and iOS app updates before Google Play and App Store Connect. Stops after the audit, and never uploads, tags, pushes or submits without a separate request. | "Close the Android update", "prepare the iOS release" |
| [`ai-butler`](skills/ai-butler) | A composed, candid chat voice: formal in manner, honest in substance, dry when it helps. Feels like talking to Alfred. Conversation only. | "Use the AI butler voice" |
| [`yt-synthesis`](skills/yt-synthesis) | Verifies a YouTube channel, picks the 10 most viewed and 10 most personally relevant videos from the last 365 days, reads their transcripts, and writes one source-linked synthesis report. | "Run a YT synthesis on @channel" |

## Install

Each skill is self-contained in its folder. For Claude Code, copy a folder from `skills/` into your personal skills directory and start a new session:

```
macOS / Linux:  ~/.claude/skills/<skill-name>/
Windows:        %USERPROFILE%\.claude\skills\<skill-name>\
```

Example (macOS / Linux):

```bash
git clone https://github.com/liamzmind/claude-skills.git
cp -r claude-skills/skills/decision-making ~/.claude/skills/
```

Example (Windows PowerShell):

```powershell
git clone https://github.com/liamzmind/claude-skills.git
Copy-Item claude-skills\skills\decision-making "$env:USERPROFILE\.claude\skills\" -Recurse
```

Prefer a single file? The `single-file/` folder has one self-contained Markdown file per skill, with its reference material appended. These are convenient for reading, sharing or pasting, but the folder version in `skills/` is the one to install.

If you use Claude somewhere other than Claude Code, check the current Anthropic documentation for how that surface loads skills.

## Repository layout

```
claude-skills/
├── README.md
├── skills/                      # install these (one folder per skill)
│   ├── decision-making/
│   │   ├── SKILL.md
│   │   └── references/templates.md
│   ├── dev-rules/
│   │   ├── SKILL.md
│   │   └── references/full-rules.md
│   ├── code-audit/
│   │   └── SKILL.md
│   ├── release-close/
│   │   ├── SKILL.md
│   │   └── references/checklists.md
│   ├── ai-butler/
│   │   └── SKILL.md
│   └── yt-synthesis/
│       ├── SKILL.md
│       └── references/
│           ├── report-template.md
│           └── profile.md       # edit this with your own context
└── single-file/                 # one bundled .md per skill, for reading and sharing
```

## Skill notes

**decision-making.** Walks you through the phases one at a time, asks for written answers, and ends with a decision record. It never decides for you and does not replace professional advice on medical, legal or financial specifics. Includes templates for a decision brief, commitment note and checkpoint review.

**dev-rules.** `SKILL.md` is a short operating discipline (about 70 lines). `references/full-rules.md` is the complete long-form text of every rule; Claude reads it when a rule needs more detail. Install both, or delete the link to the reference if you want the short version only.

**code-audit.** Read-only by design: it does not edit, format, install, commit or delete anything, and it asks before any command that writes. It treats secrets and personal data as sensitive and reports where they are, never their values.

**release-close.** Looks for your release configuration in `CLAUDE.md`, `AGENTS.md` or a `.release-close.md` (a template is in `references/checklists.md`) and asks once for anything missing. The store limits and Google Play and App Store requirements in its checklists are written from general knowledge; confirm them against current Play Console and App Store Connect documentation before relying on them.

**ai-butler.** A response register only. It does not change tool access, authority or safety rules, and it does not apply to Claude Code or terminal work.

**yt-synthesis.** Works best with [`yt-dlp`](https://github.com/yt-dlp/yt-dlp) installed; otherwise it falls back to slower web and browser methods. Edit `references/profile.md` with your own interests, projects and goals so relevance picks match you. It never fabricates transcripts, marks any video it cannot read as unavailable, and keeps quotes short. Respect each creator's work: the report is a personal study aid.

## Contributing

Issues and pull requests are welcome: bug reports, clearer wording, new checklists, and results from using a skill on a real project. When you report a problem, include the skill name, your request, and what you expected.

## License

MIT. See [LICENSE](LICENSE).

## Disclaimer

These skills are independent community work. They are not affiliated with or endorsed by Anthropic, DC, Marvel or Disney. Character names appear only as comparisons describing a style.
