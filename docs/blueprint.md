# GroupGuard — Bot specification

**Archetype:** community

**Voice:** professional and concise — write every user-facing message, button label, error, and empty state in this voice.

GroupGuard is a Telegram moderation bot that enforces human verification for new members, detects and removes spam/bots, and provides admin controls for configuration and reporting. It maintains persistent records of members, trusted users, and moderation actions, with configurable thresholds and automated enforcement rules.

> This is the complete contract for the bot. Implement EVERY entry point, flow, feature, integration, and edge case below. The completeness review checks the bot against this document after each build pass.

## Primary audience

- Telegram group admin
- Group members

## Success criteria

- Reduces spam messages by 90% through automated detection
- Ensures 100% of new members complete human verification before posting
- Maintains accurate audit logs of all moderation actions
- Delivers daily reports with 100% uptime for admin review

## Entry points

Every feature must be reachable from the bot's command/button surface (button-first; only /start and /help are slash commands).

- **/start** (command, actor: user, command: /start) — Open main admin configuration menu
- **I'm human** (button, actor: user, callback: verify:member) — Verify new member's humanity to lift restrictions
- **/warn** (command, actor: admin, command: /warn) — Issue warning to member with optional reason
- **/mute** (command, actor: admin, command: /mute) — Mute member with duration and reason
- **/kick** (command, actor: admin, command: /kick) — Kick member with reason
- **/ban** (command, actor: admin, command: /ban) — Ban member with reason
- **/trust** (command, actor: admin, command: /trust) — Mark member as trusted (exempt from checks)
- **/untrust** (command, actor: admin, command: /untrust) — Remove trusted status from member
- **/set_welcome** (command, actor: admin, command: /set_welcome) — Configure welcome message and rules text
- **/set_thresholds** (command, actor: admin, command: /set_thresholds) — Adjust spam detection thresholds and timeouts
- **/set_automations** (command, actor: admin, command: /set_automations) — Configure which automated actions are enabled
- **/report** (command, actor: admin, command: /report) — Request summary report of moderation activity
- **/log** (command, actor: admin, command: /log) — Fetch recent audit log entries

## Flows

### New member verification
_Trigger:_ new_member_joined

1. Send welcome message with 'I'm human' button
2. Restrict member until verification
3. Verify on button click or remove on timeout

_Data touched:_ Member record, Configuration, Audit log

### Spam detection
_Trigger:_ message_received

1. Check message against spam rules
2. Apply configured automated action if threshold met
3. Post explanation message in group
4. Log action in audit log

_Data touched:_ Member record, Moderation action, Audit log

### Admin command execution
_Trigger:_ /command

1. Validate admin privileges
2. Execute command (warn/mute/kick/etc)
3. Send confirmation response
4. Update audit log

_Data touched:_ Member record, Moderation action, Audit log

### Periodic reporting
_Trigger:_ daily_schedule or /report

1. Aggregate moderation data
2. Format report with statistics
3. Send to admin private chat

_Data touched:_ Audit log, Member record

## Data entities

Durable data (must survive a restart) uses the toolkit's persistent store, never in-memory maps.

- **Group** _(retention: persistent)_ — Telegram chat the bot is moderating
  - fields: chat_id, title, creation_date
- **Member record** _(retention: persistent)_ — Tracking of user participation and trust status
  - fields: user_id, display_name, join_timestamp, trust_flag, verification_status, message_history
- **Admin** _(retention: persistent)_ — Sole administrator account
  - fields: user_id, username
- **Moderation action** _(retention: persistent)_ — Record of automated or manual moderation decisions
  - fields: action_type, target_user_id, timestamp, reason, evidence
- **Audit log** _(retention: persistent)_ — History of all moderation actions
  - fields: action_id, action_type, user_id, timestamp, admin_user_id, reason
- **Configuration** _(retention: persistent)_ — Bot settings and thresholds
  - fields: welcome_text, rules_text, verification_timeout, spam_thresholds, automated_actions, trusted_users

## Integrations

- **Telegram** (required) — Bot API messaging and group management
Call external APIs against their real contract (correct endpoints, ids, params); credentials from env. Do not fake responses.

## Owner controls

- Configure welcome message and rules
- Adjust spam detection thresholds
- Set verification timeout duration
- Toggle automated action types
- Review audit logs
- Generate moderation reports
- Manage trusted user list

## Notifications

- Welcome message with verification button
- Spam action explanation posts
- Verification timeout notifications
- Daily summary reports
- On-demand audit logs

## Permissions & privacy

- Only admin can access configuration and reports
- New members are restricted until verification
- Trusted users are exempt from spam checks
- Pinned messages are never moderated
- Admin account is never targeted for moderation

## Edge cases

- Member joins and leaves before verification
- Admin tries to moderate another admin
- Verification timeout occurs during bot restart
- Spam threshold changes mid-verification period
- Audit log retention period expires

## Required tests

- Verify new member verification flow with timeout
- Test spam detection against known patterns
- Validate admin command permissions
- Confirm audit log persistence across restarts
- Test report generation with different time periods

## Assumptions

- Only one admin account exists
- Default verification timeout is 3 minutes
- Default spam thresholds are 48h account age, 3 identical messages in 30s, 5 messages in 10s
- Automated actions follow default escalation path
