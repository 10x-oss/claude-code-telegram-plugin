# Claude Code Telegram Plugin (with Forum Topic Support)

Fork of the official Claude Code Telegram plugin with added support for **Telegram forum groups** — each topic in a group acts as a separate Claude Code conversation.

## What's different from the official plugin

- Passes `message_thread_id` from inbound messages to Claude
- Accepts `message_thread_id` in the reply tool for posting to the correct topic
- Typing indicators target the correct topic
- File attachments (photos, documents) are sent to the correct topic

## Prerequisites

- [Bun](https://bun.sh) — `curl -fsSL https://bun.sh/install | bash`
- A Telegram bot token from [@BotFather](https://t.me/BotFather)

## Setup

### 1. Create a bot

DM [@BotFather](https://t.me/BotFather) on Telegram and send `/newbot`. Copy the token it gives you (looks like `123456789:AAHfiqksKZ8...`).

### 2. Disable privacy mode

Still in BotFather, send `/setprivacy`, select your bot, choose **Disable**. This lets the bot see all messages in groups, not just @mentions.

### 3. Create a Telegram group with topics

Create a supergroup in Telegram and enable **Topics** in group settings. Each topic will become a separate Claude Code conversation.

### 4. Add the bot to the group

Add your bot to the group. After adding it, go to group info and confirm it says "has access to messages."

### 5. Install the plugin

Clone this repo and install it as a local plugin in Claude Code:

```sh
git clone https://github.com/10x-oss/claude-code-telegram-plugin.git
cd claude-code-telegram-plugin
bun install
```

Then in Claude Code:

```
claude plugin install /path/to/claude-code-telegram-plugin
```

Or copy the files into your plugins directory manually:

```sh
cp -r . ~/.claude/plugins/marketplaces/claude-plugins-official/external_plugins/telegram/
cd ~/.claude/plugins/marketplaces/claude-plugins-official/external_plugins/telegram/
bun install
```

### 6. Configure the token

```
/telegram:configure <your-bot-token>
```

Or write it manually:

```sh
mkdir -p ~/.claude/channels/telegram
echo "TELEGRAM_BOT_TOKEN=123456789:AAHfiqksKZ8..." > ~/.claude/channels/telegram/.env
```

### 7. Launch Claude Code with the channel

```sh
claude --channels plugin:telegram@claude-plugins-official
```

### 8. Pair yourself

DM your bot on Telegram — it replies with a 6-character pairing code. In your Claude Code session:

```
/telegram:access pair <code>
```

Then lock it down:

```
/telegram:access policy allowlist
```

### 9. Add the group

Get your group's supergroup ID. The easiest way: share a link to any message in the group — the URL will look like `https://t.me/c/XXXXXXXXXX/123`. Take the number after `/c/` and prepend `-100`:

```
/telegram:access group add -100XXXXXXXXXX --no-mention
```

### 10. Start chatting

Create a new topic in the group and send a message. The bot will respond in that topic. Each topic is an independent conversation.

## Running as a background service (Linux)

To keep the bot running permanently, create a systemd user service:

```sh
mkdir -p ~/.config/systemd/user

cat > ~/.config/systemd/user/claude-telegram.service << 'EOF'
[Unit]
Description=Claude Code Telegram bridge
After=network-online.target
Wants=network-online.target

[Service]
Type=simple
Environment=HOME=/root
Environment=IS_SANDBOX=1
ExecStart=/usr/bin/script -q -c "/root/.local/bin/claude --dangerously-skip-permissions --channels plugin:telegram@claude-plugins-official" /dev/null
Restart=always
RestartSec=5

[Install]
WantedBy=default.target
EOF

systemctl --user daemon-reload
systemctl --user enable --now claude-telegram.service
```

Check status: `systemctl --user status claude-telegram.service`

## Access control

See **[ACCESS.md](./ACCESS.md)** for DM policies, groups, mention detection, delivery config, and the full `access.json` schema.

## Tools exposed to the assistant

| Tool | Purpose |
| --- | --- |
| `reply` | Send to a chat. Takes `chat_id` + `text`, optionally `reply_to` (message ID) for threading, `message_thread_id` for forum topics, and `files` (absolute paths) for attachments. |
| `react` | Add an emoji reaction to a message by ID. Only Telegram's fixed whitelist is accepted. |
| `edit_message` | Edit a message the bot previously sent. |

## License

Same license as the original Claude Code Telegram plugin. See [LICENSE](./LICENSE).
