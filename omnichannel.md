# 🌐 Omnichannel & Channels

PulseChat can act as an **omnichannel inbox** for external messaging platforms like WhatsApp, Telegram, Email, SMS, Slack, Viber, etc. This page explains how channels work, how to configure them, and how staff interact with them.

---

## 1. Concepts

- **Channel** – a configured connection to an external service (e.g. a WhatsApp Business number, a Telegram bot, an IMAP email inbox).
- **Channel conversation** – a thread with an external contact (e.g. a WhatsApp chat with a customer).
- **External contact** – unified record representing the outside party (phone, email, etc.), mapped to CRM clients/contacts/leads when possible.

Core tables (created in `122_version_122.php`):

- `pc_channels` – channel configurations (type, name, encrypted config).
- `pc_external_contacts` – high‑level external contact identities.
- `pc_contact_identities` – specific identifiers (phone, email, PSID, chat_id, etc.).
- `pc_automation_rules` – routing and automation rules.

---

## 2. Enabling Omnichannel

1. As admin, go to **PulseChat Settings**.
2. Enable:
   - **Enable Omnichannel Channels**
     - Option: `pulsechat_channels_enabled`
3. Save.
4. In the chat UI, a **Channels** tab will appear in the sidebar.

---

## 3. Configuring Channels ⚙️

In the chat UI:

1. Click the **Channels** tab.
2. Click the **cog icon** in the Channels filters bar (Channel Settings).
3. This opens the **Channel Settings modal**, where you can:
   - Create new channels.
   - Edit existing channels.
   - Test and save configuration.

Each channel type has its own configuration fields and requirements, implemented via adapter classes in `libraries/channels/`:

- `WhatsappAdapter.php`
- `TelegramAdapter.php`
- `EmailAdapter.php`
- (and others such as SMS, Slack, Viber, etc.)

The admin UI for channels is driven by:

- `controllers/Pulsechat_Channels.php`
- `assets/js/pulsechat-channels.js`

> **Note:** Many channel providers require webhooks (e.g. Telegram, WhatsApp Cloud API) and/or periodic polling (e.g. Email via IMAP). Make sure to configure your provider’s webhook URL or IMAP credentials as instructed in the Channel Settings UI.

---

## 4. Supported Channel Types

The base migration (`122_version_122.php`) declares these types:

- `whatsapp`
- `telegram`
- `messenger`
- `instagram`
- `email`
- `sms`
- `slack`
- `viber`

Different adapters may be present or added over time. Check `libraries/channels/` for available adapters in your version.

---

## 5. Inbound Flow (High‑Level)

1. **External message arrives**:
   - Via webhook (e.g. WhatsApp/Telegram → `modules/pulsechat/webhook.php` or `controllers/webhooks/ChannelWebhook.php`).
   - Via polling (e.g. Email IMAP via `pulsechat_cron_email_poll`).

2. **ChannelManager**:
   - `libraries/channels/ChannelManager.php` receives an inbound payload.
   - Validates rate limits, decodes message, and normalizes metadata.

3. **Contact matching**:
   - `libraries/ContactMatcher.php` tries to map the inbound identifier (phone, email, PSID, etc.) to an existing:
     - External contact (`pc_external_contacts`),
     - CRM contact / client (`tblcontacts` / `tblclients`),
     - Lead (`tblleads`),
     - or creates a new external contact record.

4. **Conversation routing**:
   - `pc_conversations` holds omnichannel conversations (type `channel`).
   - Automation rules (`pc_automation_rules`) can be applied to:
     - Auto‑assign to staff / team.
     - Set status, priority, tags.
     - Trigger actions (auto‑reply, ticket creation, AI response, etc.).

5. **Message creation**:
   - A message row is inserted into `pc_messages` with:
     - `channel_id`, `channel_type`, external `channel_conversation_id`.
     - Direction (`inbound`), status, delivery state.

6. **UI update**:
   - Channel conversations list is refreshed in `pulsechat-channels.js`.
   - The active agent sees the new conversation and can reply from PulseChat.

---

## 6. Outbound Flow (High‑Level)

When an agent replies in a channel conversation:

1. `pulsechat-channels.js` calls:
   - `config.channelSendMessageUrl` → `admin/pulsechat/channel_api/send_channel_message`

2. `Pulsechat_Channels::send_channel_message()`:
   - Validates permissions.
   - Persists the message in `pc_messages` as `outbound`.

3. **ChannelManager**:
   - Uses the appropriate adapter (`WhatsappAdapter`, `TelegramAdapter`, `EmailAdapter`, etc.) to send the message to the external platform.
   - Updates `delivery_status` (`queued`, `sent`, `delivered`, `read`, `failed`) and error messages if any.

4. **UI feedback**:
   - Message appears in the conversation with a status indicator.
   - Errors (e.g. invalid token, network issues) are surfaced in the UI, often with hints (e.g. “update your WhatsApp token in channel settings”).

---

## 7. Channel‑Specific Notes

### 7.1 WhatsApp

- Adapter: `libraries/channels/WhatsappAdapter.php`
- Requires:
  - **Permanent Access Token** from Meta (System User token with `whatsapp_business_messaging` permission).
  - Phone number ID and WhatsApp Business account details.
- UI:
  - Channel config form shows fields like access token, phone ID, webhook verify token, etc.
  - Errors from WhatsApp (e.g., “Session has expired”) are captured and shown to the agent with remediation hints.

### 7.2 Telegram

- Adapter: `libraries/channels/TelegramAdapter.php`
- Requires:
  - Bot token from BotFather.
- Uses webhooks:
  - You must configure Telegram with the webhook URL shown in Channel Settings.
  - Signature verification is handled in the adapter.

### 7.3 Email

- Adapter: `libraries/channels/EmailAdapter.php`
- Uses **IMAP polling** and **SMTP**:
  - IMAP: fetching inbound mail from an inbox.
  - SMTP: sending replies (or fallback to Perfex’s default SMTP if not set).
- Polling:
  - Implemented via `pulsechat_cron_email_poll()` in `pulsechat.php`.
  - Hooked to Perfex’s `after_cron_run`.
- TLS/OAuth:
  - Email adapter supports oauth or app‑password based logins depending on provider and configuration.

---

## 8. Automation Rules 🤖

In omnichannel mode, you can define **Automation Rules** for routing and actions.

- Stored in `pc_automation_rules`.
- Configurable via the omnichannel UI (Automation Rules section).
- Actions can include:
  - Assign to staff / team.
  - Set status / priority.
  - Add tags.
  - Send canned responses.
  - Create tickets.
  - **Trigger AI Response** (integrates with the AI assistant for full auto‑replies).

These rules allow you to build advanced workflows like:

- “If message comes from WhatsApp to the ‘Sales’ number, assign to Sales team and tag as `whatsapp-lead`.”
- “If channel is Email and subject contains ‘support’, auto‑create a ticket and mark conversation as `pending`.”

---

## 9. Channels Tab UI 🖥️

In the **Channels** tab, agents see:

- A **filters bar**:
  - Status filter (Open, Pending, Resolved, Closed).
  - Priority filter (Low, Normal, High, Urgent).
  - Channel type filter (WhatsApp, Telegram, Email, etc.).
  - Channel Settings button (cog).

- A **channel conversation list**:
  - Each row shows:
    - External contact name.
    - Channel icon / type.
    - Last message preview.
    - Status, priority, unread count.

- Selecting a conversation:
  - Opens it in the center chat panel.
  - The details panel shows CRM context for the contact (linked client/contact/lead if matched).

---

## 10. Best Practices ✅

- Always **test** a channel after configuring it:
  - Use a test phone/email.
  - Check that inbound and outbound messages work and statuses update.

- Use **Automation Rules** to:
  - Automatically assign new conversations to the right team.
  - Tag and prioritize by channel and content.

- Keep credentials up‑to‑date:
  - Tokens and app passwords often expire.
  - When you see repeated send failures, check Channel Settings and provider dashboards.

- For email channels:
  - Ensure your Perfex cron is configured and running regularly, or inbound emails will not be fetched.

If a specific channel misbehaves, see **[🧪 Troubleshooting & FAQ](troubleshooting.md)** for targeted checks.

