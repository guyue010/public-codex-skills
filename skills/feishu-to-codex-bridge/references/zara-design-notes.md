# Zara lark-channel-bridge Design Notes

## What Zara Automates

- Provides an npm-distributed CLI named `lark-channel-bridge`.
- Runs a `start` command that detects missing config and launches a QR-based setup wizard.
- Stores credentials and runtime files under `~/.lark-channel/`.
- Registers running processes and exposes `ps` / `stop`.
- Uses long connection for Feishu/Lark events.
- Routes Feishu messages to a local CLI agent, originally Claude Code.

## What Zara Still Requires Manually

The README states that the first-run wizard creates or selects the app and writes credentials, but Open Platform still needs manual confirmation.

Required scopes:

- `im:message`
- `im:message:send_as_bot`
- `im:resource`

Required events:

- `im.message.receive_v1`
- `card.action.trigger`

Optional events:

- `im.message.reaction.created_v1`
- `im.message.reaction.deleted_v1`
- `im.chat.member.bot.added_v1`

So do not claim that the wizard fully configures Feishu Open Platform unless the current implementation has an API-backed proof.

## Runtime State Model

Zara keeps:

- `config.json`: app credentials, mode, preferences
- `sessions.json`: per-chat session and cwd
- `workspaces.json`: named workspace mappings
- `processes.json`: active process registry
- `media/<chatId>/`: downloaded media cache
- `logs/YYYY-MM-DD.log`: structured logs

This is a useful template for productizing a bridge beyond a prototype `.env`.

## Feature Areas To Compare

### Setup

- QR setup wizard
- config file creation
- file permissions
- Open Platform checklist
- doctor command

### Runtime

- long connection readiness
- reconnect handling
- process registry
- duplicate process detection
- graceful stop

### Message Handling

- private chat replies
- group `@bot` requirement
- ignore `@all`
- bot-self filtering
- duplicate event filtering
- history replay filtering
- quick consecutive message batching
- interrupt/stop active run

### Agent

- adapter interface
- local CLI process management
- streaming output parsing
- idle timeout
- workspace/cwd switching
- per-chat sessions

### Feishu UX

- status cards
- streaming cards
- tool-call rendering
- config cards
- help cards
- buttons for stop/workspace/config

### Media

- image/file download
- safe local cache
- cleanup policy
- pass file paths to the agent

### Security

- local config not committed
- secrets masked in logs
- allowlist users/chats
- rate limits
- OpenAI/API budget limits
- cloud deployment secret handling

## Recommended Roadmap For feishu-codex-bridge

1. Keep the current minimal bridge stable: doctor, start, self-filter, history-filter, dedupe, status reply.
2. Add process registry plus `ps` and `stop`.
3. Move from project `.env` to `~/.feishu-codex-bridge/config.json` with `0600` permissions.
4. Add `init` that creates config and prints the Open Platform checklist.
5. Add allowlist for user open ids and chat ids.
6. Add media download only after text handling is stable.
7. Add cards/streaming only after duplicate reply and self-trigger issues are solved.
