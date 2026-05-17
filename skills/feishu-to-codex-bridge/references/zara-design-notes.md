# Zara lark-channel-bridge Design Notes

## What Zara Automates

- Provides an npm-distributed CLI named `lark-channel-bridge`.
- Runs a `start` command that detects missing config and launches a QR-based setup wizard.
- Stores credentials and runtime files under `~/.lark-channel/`.
- Registers running processes and exposes `ps` / `stop`.
- Uses long connection for Feishu/Lark events.
- Routes Feishu messages to a local CLI agent, originally Claude Code.
- Maintains one session per chat/topic so conversation can continue across turns.
- Supports streaming cards so text and tool calls appear in one live-updating card.
- Supports active-run interruption and quick consecutive message batching.
- Supports workspace switching and media/file download handoff.
- Uses in-chat slash commands and card buttons as the primary operator interface.

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

## In-Chat Slash Commands To Reuse

Zara's README documents a fuller command surface than the initial Feishu to Codex Bridge MVP. Useful commands to preserve or adapt:

| Command | Reusable behavior |
|---|---|
| `/new` / `/reset` | Clear the current chat session. |
| `/cd <path>` | Switch working directory and reset the session. |
| `/ws list` | List named workspaces, ideally with card buttons. |
| `/ws save <name>` | Save the current cwd as a named workspace. |
| `/ws use <name>` | Switch to a named workspace. |
| `/ws remove <name>` | Remove a named workspace. |
| `/status` | Show current cwd, session, agent, connection, and process status. |
| `/config` | Adjust preferences such as group mention behavior, reply style, tool-call display, and timeout defaults. |
| `/stop` | Stop the current active run, also available as a card button. |
| `/timeout [N|off|default]` | Configure idle timeout for the current session. |
| `/ps` | List active local bridge processes and identify the current one. |
| `/exit <id|#>` | Terminate a selected bridge process. |
| `/reconnect` | Force WebSocket or long-connection reconnect. |
| `/doctor [description]` | Feed recent logs plus the user's description to the agent for diagnosis. |
| `/help` | Return a help card. |
| unknown `/xxx` | Pass through to the agent only when not reserved by the bridge. |

## Feishu Message Strategy To Reuse

- Private chats reply to direct messages.
- Groups and topic groups should require `@bot` by default.
- Plain group messages without `@bot` should be ignored.
- `@all` should never trigger the bot.
- Cloud document comments should require `@bot` when supported.
- Mention policy should be configurable, but safe defaults matter.

## Card And Interaction Patterns

Zara treats cards as a control surface, not just decoration:

- `/help`: command reference card.
- `/status`: current cwd/session/agent/process card.
- `/ws list`: workspace list with selection buttons.
- `/config`: preference card with toggles/buttons.
- long-running run: streaming card with text, tool calls, and stop button.
- `/stop`: can be triggered from text command or card button.

For a Codex bridge, keep a simple text status reply as the first version, then add cards once duplicate reply and self-trigger guards are stable.

## Host CLI Details To Reuse

Zara's host CLI surface:

```text
lark-channel-bridge start [-c <config>]
lark-channel-bridge ps
lark-channel-bridge stop <id|#>
lark-channel-bridge --help
```

Additional placeholder areas in the README include `status`, `doctor`, `handover`, `workspace`, and `service`. For a Codex version, these are useful roadmap labels even if the first release only ships `start`, `doctor`, `ps`, `stop`, and `init`.

When multiple `start` processes use the same app, events may be randomly delivered to one long connection. The bridge should detect this and offer a safe operator choice before starting another process.

## FAQ Lessons To Reuse

- If the agent hangs, `/status` and `/new` should be easy recovery paths.
- Idle timeout can terminate a stuck child process after no output.
- Media bugs are common; downloaded file paths and cleanup behavior should be explicit.
- Migration from older config/cache locations should be a first-class command once public users exist.

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
- help card
- command passthrough rules
- group mention policy
- ignore `@all`

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
6. Add full in-chat command routing: `/new`, `/reset`, `/status`, `/stop`, `/help`, then `/ws`, `/cd`, `/config`, `/timeout`, `/ps`, `/exit`, `/reconnect`, `/doctor`.
7. Add message batching and active-run interruption policy.
8. Add media download only after text handling is stable.
9. Add cards/streaming only after duplicate reply and self-trigger issues are solved.
10. Add migration and service/cloud deployment docs once users depend on the bridge.
