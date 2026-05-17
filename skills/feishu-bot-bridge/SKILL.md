---
name: feishu-bot-bridge
description: Build, audit, harden, or productize a Feishu/Lark bot bridge that receives chat messages through event subscription or long connection, routes them to an AI agent or local CLI, and replies back to Feishu. Use when implementing a Zara-style lark-channel-bridge clone, diagnosing duplicate replies/self-triggering/history replay, creating setup/doctor/start commands, designing App ID/Secret storage, or documenting Feishu Open Platform bot permissions and event setup.
---

# Feishu Bot Bridge

## Acknowledgement

This skill and bridge pattern were inspired by Zara's open-source Feishu/Lark-to-Claude bridge work: `https://github.com/zarazhangrui/feishu-claude-code-bridge`. The connection approach, productization checklist, and setup logic here reference the framework and design ideas she shared. Thanks to Zara for the contribution and public sharing; this skill is meant to respect the original work while encouraging shared learning and collaborative innovation.

## Core Shape

Build the bridge as a long-running service, not as a static config file.

```text
Feishu/Lark bot
  -> event subscription, usually long connection for local-first use
  -> bridge service
  -> agent adapter, such as OpenAI API, Codex CLI, Claude CLI, or another tool
  -> Feishu reply/card/file/document action
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
- Treat sessions, workspaces, media, process registry, and logs as first-class runtime state.

## Recommended Minimal CLI

For a reusable bridge, implement these host commands:

```text
bridge start [-c config]
bridge doctor
bridge ps
bridge stop <id|#>
bridge init
```

The first version can keep `init` simple: create a config template, set file permissions, and print the exact Feishu Open Platform checklist. Add QR/app creation only after confirming the Feishu API flow is reliable.

## Feishu In-Chat Commands

Start small:

```text
/help
/status
/new
/reset
/stop
```

Add later:

```text
/config
/ws list
/ws save <name>
/ws use <name>
/doctor
```

## Common Failure Modes

- **No response**: bot not added to chat, app not installed/published, event not subscribed, long connection not selected, wrong app secret.
- **Responds many times**: multiple processes running for same app, duplicate event delivery, missing `message_id`/content dedupe, bot processing its own replies.
- **Answers old messages**: bridge restarted and Feishu replayed events; drop events older than startup time.
- **Works locally but not on phone later**: local bridge stopped because computer slept/shut down; use cloud or hybrid mode.
- **Secret exposure**: rotate App Secret/API key, never paste secrets into chat, keep local config out of Git.

## When To Use References

- Use `references/zara-design-notes.md` when comparing against `lark-channel-bridge` or planning the next feature tranche.
