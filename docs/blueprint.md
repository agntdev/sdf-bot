# SDF_Bot — Bot specification

**Archetype:** custom

**Voice:** warm and concise — write every user-facing message, button label, error, and empty state in this voice.

A friendly Telegram bot that engages in casual conversation, answers questions, and escalates important user messages (complaints/abuse) to the owner via private notifications while maintaining short-term conversation context.

> This is the complete contract for the bot. Implement EVERY entry point, flow, feature, integration, and edge case below. The completeness review checks the bot against this document after each build pass.

## Primary audience

- Any Telegram user initiating a chat

## Success criteria

- Owner receives timely notifications for complaints/abuse reports
- Users receive contextual responses in casual conversations
- Conversation history preserved for 72 hours

## Entry points

Every feature must be reachable from the bot's command/button surface (button-first; only /start and /help are slash commands).

- **/start** (command, actor: user, command: /start) — Open greeting with interaction hints
- **Report issue** (button, actor: user, callback: report:start) — Trigger complaint reporting flow
  - inputs: complaint text
  - outputs: notification confirmation

## Flows

### Conversation start
_Trigger:_ on_message

1. Detect new user session
2. Send greeting with capabilities

_Data touched:_ user_profile, conversation_session

### Casual chat
_Trigger:_ user_message

1. Analyze message context
2. Generate human-like response using recent history

_Data touched:_ conversation_session

### Complaint escalation
_Trigger:_ complaint keywords or /report command

1. Acknowledge request
2. Create notification item
3. Send private message to owner
4. Confirm to user

_Data touched:_ notification_item, user_profile

### Safety moderation
_Trigger:_ Abusive content detection

1. Send de-escalation response
2. Generate moderation notification
3. Notify owner privately

_Data touched:_ notification_item, user_profile

## Data entities

Durable data (must survive a restart) uses the toolkit's persistent store, never in-memory maps.

- **user_profile** _(retention: session)_ — Telegram user metadata
  - fields: telegram_id, display_name, language, conversation_context
- **conversation_session** _(retention: session)_ — Last 10 messages per user
  - fields: message_history, timestamp
- **notification_item** _(retention: persistent)_ — Escalation records
  - fields: user_id, excerpt, timestamp, notification_type
- **owner** _(retention: persistent)_ — Administrator account
  - fields: telegram_id

## Integrations

- **Telegram** (required) — Bot API messaging
Call external APIs against their real contract (correct endpoints, ids, params); credentials from env. Do not fake responses.

## Owner controls

- Telegram account for private notifications
- Keyword sensitivity settings for complaint detection

## Notifications

- Private Telegram messages to owner with user context when complaints/abuse detected

## Permissions & privacy

- Stores last 10 messages per user (72h retention)
- Only stores minimal metadata (telegram_id, display_name)
- No long-term personal data retention

## Edge cases

- Handling multiple simultaneous reports
- De-escalation of aggressive messages
- Language detection for complaint keywords

## Required tests

- End-to-end escalation flow with notification confirmation
- Context preservation across 10-message history
- Abuse content detection accuracy

## Assumptions

- Owner's Telegram ID must be pre-configured
- Russian is primary language with automatic fallback to user's language
- Session expiration after 72h inactivity
