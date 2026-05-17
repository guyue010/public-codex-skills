# Feishu to Codex Bridge Skill

Build, audit, harden, or productize a Feishu/Lark to Codex bridge that receives chat messages, routes them to Codex, OpenAI, or a local CLI, and replies back to Feishu.

## Acknowledgement

This connection approach was inspired by Zara's open-source Feishu/Lark-to-Claude bridge:

https://github.com/zarazhangrui/feishu-claude-code-bridge

The bridge structure, setup flow, and productization checklist in this skill reference the framework and logic Zara shared. Thank you to Zara for the contribution and public sharing. This project aims to respect the original work while supporting shared learning, adaptation, and collaborative innovation.

## What This Skill Covers

- Feishu/Lark to Codex bridge architecture.
- Long-connection versus webhook runtime choices.
- Open Platform permission and event checklist.
- Guardrails for duplicate replies, self-triggering, and history replay.
- Local and cloud deployment considerations.
- Configuration and secret-handling guidance.

## Secret Handling

Do not commit real `.env` files, Feishu App Secrets, OpenAI API keys, local logs, chat history, or private user data. Use `.env.example` for public examples and keep real credentials in local config or cloud secret storage.

## Entry Point

Use `SKILL.md` as the operational instruction file. Use files under `references/` for setup and comparison notes.
