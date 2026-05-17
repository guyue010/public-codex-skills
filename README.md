# public-codex-skills

Reusable Codex skills that are safe to share publicly.

This repository contains public, sanitized skill templates and workflows. It is separate from private skill backups so personal configuration, local paths, API keys, app secrets, logs, and project-specific memory stay out of public source control.

## Skills

- `skills/xhs-publish-note` - publish image-based Xiaohongshu notes through Chrome and Creator Studio.
- `skills/feishu-to-codex-bridge` - design, audit, and productize a Feishu/Lark to Codex bridge that routes messages to Codex, OpenAI, or a local CLI.

## Safety Rules

- Do not commit `.env`, local config, API keys, Feishu App Secret, OpenAI API keys, cookies, logs, screenshots with private data, or exported chat history.
- Use `.env.example` for configuration examples.
- Keep public skills generic: describe the workflow, required permissions, setup checks, and failure modes without embedding a personal app or account.
- Review references before publishing a new skill, especially when the skill touches browser automation, messaging platforms, account login, or external APIs.

## Suggested Install Pattern

Copy a skill folder into your Codex skills directory:

```bash
mkdir -p ~/.codex/skills
cp -R skills/xhs-publish-note ~/.codex/skills/
```

Then restart or reload Codex so the new skill is discovered.

## Repository Layout

```text
skills/
  <skill-name>/
    SKILL.md
    agents/
    references/
```

Each skill should be self-contained enough for another user to understand when to use it, what setup is required, and where the safety boundaries are.
