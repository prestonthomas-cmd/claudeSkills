# claude-skills

Personal Claude Code skills and global instructions, loaded into every cloud session by the environment setup script.

```
skills/<name>/SKILL.md   → linked to ~/.claude/skills/
CLAUDE.md                → copied to ~/.claude/CLAUDE.md (global instructions)
```

## Skills
- **repo-protocol**: every repo has `main` and `dev`. Claude works and pushes on `dev`, then asks before merging into `main`.

## Adding a skill
Create `skills/<name>/SKILL.md` with `name` and `description` frontmatter, then push to `main`. New sessions pick it up automatically.

**This repo is public. Don't put secrets or private details in it.**
