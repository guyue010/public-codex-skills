---
name: lark-to-codex-bridge
description: Build, audit, harden, or productize a Lark/Feishu to Codex bridge that receives chat messages through event subscription or long connection, routes them to Codex, OpenAI, or a local CLI, and replies back to Lark. Use when implementing a Zara-style lark-channel-bridge adaptation for Codex, diagnosing duplicate replies/self-triggering/history replay, creating setup/doctor/start commands, designing App ID/Secret storage, or documenting Lark Open Platform bot permissions and event setup.
---

# Lark to Codex Bridge

## Acknowledgement

This skill and bridge pattern were inspired by Zara's open-source Feishu/Lark-to-Claude bridge work: `https://github.com/zarazhangrui/feishu-claude-code-bridge`. The connection approach, productization checklist, and setup logic here reference the framework and design ideas she shared. Thanks to Zara for the contribution and public sharing; this skill is meant to respect the original work while encouraging shared learning and collaborative innovation.

## Core Shape

Build the bridge as a long-running service, not as a static config file.

```text
Lark/Feishu bot
  -> event subscription, usually long connection for local-first use
  -> bridge service
  -> Codex or agent adapter, such as Codex CLI, OpenAI API, Claude CLI, or another tool
  -> Lark reply/card/file/document action
```

Keep secrets out of source control. Store local credentials in a gitignored config file with restrictive permissions, or store cloud credentials as platform secrets.

## Workflow

1. Confirm the runtime model.
   - Local-first: long connection, local config, can call local CLI and local files, stops when the machine stops.
   - Cloud-first: webhook or cloud-hosted persistent worker, cloud secrets, can run on phone while local computer is off, cannot touch local files unless a local agent is also online.
   - Hybrid: cloud bot for chat plus optional local agent for file/code tasks.

2. Check Feishu Open Platform setup.
   - Bot ability is enabled.
   - Event subscription mode is chosen: long connection for local persistent service, webhook for public HTTP callback.
   - Required events include `im.message.receive_v1`.
   - Optional events include card actions, reactions, bot-added events, and comment events.
   - Required scopes usually include `im:message` and `im:message:send_as_bot`; file/image flows also need resource scopes such as `im:resource`.
   - Some setup still requires manual confirmation in Feishu Open Platform unless an official app-management API/tool can perform it.

3. Implement the bridge in modules.
   - `config`: load app id, app secret, agent keys, domain, mode, allowlists.
   - `doctor`: validate credentials, token exchange, model/API key, required settings.
   - `runtime/process`: prevent multiple bridge processes for the same app.
   - `events`: receive and normalize Feishu events.
   - `guards`: ignore bot messages, startup history replays, duplicate events, unsupported message types, unauthorized users/chats.
   - `session`: maintain per-chat context and reset commands.
   - `agent`: isolate OpenAI/Codex/Claude adapters behind one interface.
   - `reply`: send status, final text, cards, or stream updates.
   - `media`: download files/images to safe local paths when supported.
   - `logs`: write structured logs without leaking secrets.

4. Add safety before adding features.
   - Resolve the bot's own open id at startup and skip messages from itself.
   - Drop messages older than service startup time to avoid history replay after reconnect.
   - Deduplicate by `message_id`, and also by chat plus normalized text within a short window.
   - Allowlist users or chats before public use.
   - Do not log secrets or full sensitive message bodies.
   - Add graceful stop and process registry before encouraging users to run multiple instances.

5. Verify end to end.
   - Run typecheck/build.
   - Run `doctor`.
   - Start the bridge and confirm long-connection readiness.
   - Send a private-message test.
   - Send a group `@bot` test.
   - Confirm one user message produces one status reply and one final reply, unless streaming/cards are intentionally enabled.
   - Restart the bridge and confirm old messages are not replayed into new answers.

## Productization Lessons From Zara's Bridge

Read `references/zara-design-notes.md` for the feature comparison checklist before deciding what to add next.

Key lessons to preserve:

- Provide a first-run setup path instead of making users hand-edit everything.
- Store config under a dedicated home directory, not scattered project files.
- Include `ps` and `stop` commands because multiple long connections for one app cause random event delivery.
- Make Open Platform requirements explicit; even Zara's wizard still tells users to manually confirm scopes and events.
- Treat sessions, workspaces, media, process registry, logs, in-chat commands, and card interactions as first-class runtime state.
- Preserve the user-facing message strategy: private chats respond directly; group chats should normally require `@bot`; ignore `@all`; document any override in `/config`.
- Consider streaming/status cards, quick-message batching, active-run interruption, and file/image handoff as product features rather than incidental implementation details.

## Recommended Minimal CLI

For a reusable bridge, implement these host commands:

```text
bridge start [-c config]
bridge ps
bridge stop <id|#>
bridge init
bridge doctor
bridge --help
```

The first version can keep `init` simple: create a config template, set file permissions, and print the exact Feishu Open Platform checklist. Add QR/app creation only after confirming the Feishu API flow is reliable. If multiple local bridge processes use the same app, detect that before `start` and offer a safe choice such as continue, kill old, or abort.

## Feishu In-Chat Commands

Implement commands in two layers. The MVP layer should be stable before adding cards or workspace controls:

```text
/help
/status
/new
/reset
/stop
```

The fuller Zara-inspired command set is:

```text
/new
/reset
/cd <path>
/ws list
/ws save <name>
/ws use <name>
/ws remove <name>
/status
/config
/stop
/timeout [N|off|default]
/ps
/exit <id|#>
/reconnect
/doctor
/help
```

Command behavior to preserve:

- `/new` and `/reset`: clear the current chat session.
- `/cd <path>`: change the current working directory and reset the session.
- `/ws list`: list named workspaces, preferably as a card with buttons.
- `/ws save <name>`: save the current working directory as a named workspace.
- `/ws use <name>`: switch to a named workspace and reset the session.
- `/ws remove <name>`: delete a named workspace.
- `/status`: show cwd, session, agent mode, process id, and connection state, preferably as a card.
- `/config`: adjust preferences such as group mention requirement, status style, streaming/card detail, and timeout defaults.
- `/stop`: stop the current active run; also expose a stop button on long-running cards.
- `/timeout [N|off|default]`: set or clear idle timeout for the current session.
- `/ps`: list bridge processes registered on this machine and identify the process handling the reply.
- `/exit <id|#>`: terminate a selected bridge process; self-exit should be graceful.
- `/reconnect`: force long-connection reconnect when the bot is silent after network issues.
- `/doctor [description]`: summarize recent logs and the user's description for self-diagnosis.
- `/help`: return a compact help card.
- Unknown `/xxx`: pass through to the agent only when it is not a reserved bridge command.

## Feishu UX And Message Strategy

- Private chat: respond to plain text by default.
- Group or topic chat: require `@bot` by default.
- Never respond to `@all`.
- Cloud document comments should require `@bot` if supported.
- Quick consecutive messages can be batched into one agent request.
- A new user message during an active run may interrupt the old run or queue behind it; make this policy explicit.
- Use cards for `/help`, `/status`, `/ws list`, `/config`, long-running status, and stop actions when card events are enabled.
- Streaming cards can show model text, tool calls, and status in one message instead of sending many separate replies.

## Recommended Runtime Data Directory

For a productized local-first bridge, prefer a dedicated home directory over project-local files:

```text
~/.lark-codex-bridge/config.json
~/.lark-codex-bridge/sessions.json
~/.lark-codex-bridge/workspaces.json
~/.lark-codex-bridge/processes.json
~/.lark-codex-bridge/media/<chatId>/
~/.lark-codex-bridge/logs/YYYY-MM-DD.log
```

Use restrictive permissions for credentials. Rotate logs, clean media caches, and mask secrets before using logs in `/doctor`.

## Common Failure Modes

- **No response**: bot not added to chat, app not installed/published, event not subscribed, long connection not selected, wrong app secret.
- **Responds many times**: multiple processes running for same app, duplicate event delivery, missing `message_id`/content dedupe, bot processing its own replies.
- **Answers old messages**: bridge restarted and Feishu replayed events; drop events older than startup time.
- **Works locally but not on phone later**: local bridge stopped because computer slept/shut down; use cloud or hybrid mode.
- **Secret exposure**: rotate App Secret/API key, never paste secrets into chat, keep local config out of Git.
- **Group noise**: group messages trigger without `@bot`; enforce mention policy and ignore `@all`.
- **Agent appears frozen**: local CLI or API call hangs; add idle timeout, `/stop`, and `/doctor`.
- **Wrong workspace**: session points to a deleted or unintended cwd; show cwd in `/status` and reset after `/cd` or `/ws use`.
- **Media not visible to agent**: download files/images to a safe local path and pass that path explicitly.

## When To Use References

- Use `references/zara-design-notes.md` when comparing against `lark-channel-bridge` or planning the next feature tranche.
