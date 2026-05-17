# Feishu Open Platform Checklist

Use this as a user-facing checklist when setting up a Feishu to Codex bridge.

## App Basics

- Create or select a Feishu/Lark app.
- Enable bot ability.
- Record App ID.
- Generate App Secret and store it only in local config or cloud secrets.

## Permissions

Minimum text bot:

- `im:message`
- `im:message:send_as_bot`

Media/file bot:

- `im:resource`

Optional advanced interactions:

- card action permission as required by current Feishu docs
- reaction read/write scopes if using reaction controls

## Events

Required:

- `im.message.receive_v1`

Recommended for cards:

- `card.action.trigger`

Optional:

- `im.message.reaction.created_v1`
- `im.message.reaction.deleted_v1`
- `im.chat.member.bot.added_v1`

## Subscription Mode

- Local-first bridge: use long connection.
- Cloud HTTP bridge: use webhook callback.

## Install And Test

- Add the bot to a private chat or group.
- Private chat: send a plain text test.
- Group chat: mention the bot explicitly.
- Confirm exactly one status message and one final reply.
- Restart the bridge and confirm old messages are ignored.
