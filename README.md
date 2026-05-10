# my-claude-skills

Personal user-level skills for [Claude Code](https://docs.claude.com/en/docs/claude-code), version-controlled via dotfiles.

These are tailored to my own role (business team lead at an AI solutions company, with an engineering background) and writing style. They're public for reference, not a general-purpose template.

## Layout

```
.claude/
└── skills/
    ├── work-style/         # baseline: tone, format, prohibitions (always-on)
    ├── proposal-writing/   # Korean B2B/government proposals (개조식)
    └── code-review/        # teammate code/report review
```

`~/.claude/skills/` is symlinked to `./.claude/skills/`, so edits in this repo take effect immediately.

## Skills

| Skill | When it triggers | What it does |
|---|---|---|
| `work-style` | Always | Korean-English mixed prose, condensed verb endings (~함/~임), conclusion-first replies, scope discipline, no auto-comments/READMEs. |
| `proposal-writing` | "제안서/RFP/과제 신청서/사업계획서" or proposal docs shared | Drafts/restructures Korean proposals in strict 개조식. Pre-submission scan for empty commitments, unsourced numbers, RFP gaps, terminology drift. |
| `code-review` | "리뷰해줘 / 검토 / 봐줘" + code or report | Theme-grouped review (leakage, reproducibility, viz, stats, etc.). Points out problems and *intent* of fixes — never writes the fix code. Ends with merge / fix-then-merge / redesign verdict. |

## Setup (for me, or anyone curious)

```sh
git clone https://github.com/mybloodykeyboard/my-claude-skills ~/dotfiles
ln -s ~/dotfiles/.claude/skills ~/.claude/skills
```

Skills auto-load when their `description` matches the conversation. Confirm with `/plugins` or by listing skills in Claude Code.

## Notes

- `settings.local.json` is gitignored — local-only by Claude Code convention.
- Memories (`~/.claude/projects/.../memory/`) are not in this repo; they're per-host.
- The Top 3 here were created first because they apply broadly. Engineering-specific skills (`data-analysis`, `ml-workflow`, `mvp-prototyping`, etc.) are deliberately deferred until real project work shapes them.
