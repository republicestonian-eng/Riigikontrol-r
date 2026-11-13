# Discord moderation and economy bot

A Node.js Discord bot with moderation and per-server economy commands. Economy balances, warnings, and temporary-ban expirations are stored in `data/bot-data.json`.

## Requirements

- Node.js 20 or newer
- A Discord application and bot token

## Setup

1. In the [Discord Developer Portal](https://discord.com/developers/applications), create an application and add a bot.
2. Invite it to your server with the `bot` and `applications.commands` scopes. Grant the moderation permissions for the commands you plan to use.
3. Copy `.env.example` to `.env` and set `DISCORD_TOKEN` and `DISCORD_CLIENT_ID`. Optionally set `DISCORD_GUILD_ID` to register commands to one server while testing.
   To use `t` text commands and resolve moderation targets by username, enable the **Message Content Intent** and **Server Members Intent** under your bot's Privileged Gateway Intents in the Discord Developer Portal.
4. Run `npm install`.
5. Run `npm run deploy` to register slash commands.
6. Run `npm start` to start the bot.

Without `DISCORD_GUILD_ID`, commands are registered globally and can take some time to appear. Keep `.env` private and never share your bot token.

## Commands

### Moderation

- `/timeout member minutes [reason]` — timeout a member (up to 28 days).
- `/kick member [reason]` and `/ban user [reason] [delete_message_seconds]`.
- `/purge amount` — delete up to 100 recent messages.
- `/warn member reason`, `/warnings member`, `/clearwarnings member`.

Moderation commands require the relevant Discord permission. The bot itself also needs the matching server permission, and its role must be above the target member’s role.

### Economy

- `/balance [user]`, `/daily`, and `/work`.
- `/pay user amount` and `/leaderboard`.

Daily claims award €100–€250 once every 24 hours; work awards €25–€100 once per hour. Balances are isolated per server and persisted locally, with no wallet cap.

The text prefix `t` starts the games:

- `t wheel <bet>` — spin for a random 0x–3x payout.
- `t plinko <bet>` — drop a chip for a random 0x–5x payout.
- `t blackjack <bet>` — play blackjack; reply `hit` or `stand` within 60 seconds. A timeout stands automatically.
- `t dice <bet> <number>` — pick 1–6; a correct pick pays 5x.
- `t lotto <bet> <number>` — pick 1–10; a correct pick pays 8x.

Individual game bets are limited to €10,000,000; wallet balances have no cap. Game payouts include the wager. The bot needs the **Message Content Intent** enabled in the Developer Portal to read these text commands.

Economy shortcuts: `t bal` and `t balance` show your balance. `t economyleaderboard`, `t elb`, `t eleaderboard`, and `t economyboard` show the same server leaderboard.

### Text moderation

Text moderation commands require the matching server permission from the person issuing the command. Commands that call Discord moderation APIs also require that permission for the bot. Use a username, user ID, or mention for member targets; username matching must be exact and unique.

- `t warn <member> <reason>`, `t warnings <member>`, and `t clearwarnings <member>`.
- `t ban <member> [duration] [reason]` — omit the duration for a permanent ban. Durations use `s`, `m`, `h`, `d`, or `w` (for example `30m` or `7d`). Temporary bans are saved and expire automatically, including after a bot restart.
- `t unban <user ID or mention> [reason]`.
- `t timeout <member> <duration> [reason]` and `t untimeout <member>`.
- `t kick <member> [reason]`.
- `t hardmute <member>` — removes all removable roles. Roles are **not backed up** and cannot be restored automatically; managed roles and roles above the bot may remain.
- `t purge <amount>` — delete 1–100 recent messages.
- `t setlog <server|user|message> <channel name, ID, or mention>` — configure the audit-log destination for that category. Use `t setlog <category> off` to disable it. Requires Manage Server permission.

The bot cannot moderate members whose highest role is equal to or above its own, and moderators cannot target members whose roles are equal to or above theirs (except the server owner).

Audit logs are configured independently for each server. Server logs include role and channel creation, edits, and deletion. User logs include server display-name changes, global username/display-name changes, and profile-picture changes. Message logs record edited/deleted messages, including bulk deletion summaries. For example, use `t setlog server #server-logs`, `t setlog user #user-logs`, and `t setlog message #message-logs`. The bot needs View Channel and Send Messages access in each selected channel.

## Tests

Run `npm test` to verify the economy store, games, and persistence behavior.
