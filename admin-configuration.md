# ⚙️ Admin Configuration

This page explains how to configure PulseChat as an administrator: transport, permissions, limits, omnichannel channels, AI, and other options exposed in `views/admin/settings.php`.

All options below are found in **Setup → Modules → PulseChat → Settings** (which opens `admin/pulsechat/settings`).

---

## 1. General 🧩

These options control global module behavior.

- **Enable PulseChat**
  - Option: `pulsechat_enabled`
  - When **Yes**, PulseChat is active and the chat menu item appears for staff with permission.

- **Enable Client Chat**
  - Option: `pulsechat_clients_enabled`
  - When **Yes**, a chat widget is injected into the **client portal**, allowing staff ↔ client conversations.

- **Default User Status on Login**
  - Option: `pulsechat_default_status`
  - Values: `online`, `away`, `busy`, `offline`
  - Determines the initial presence status when staff open the admin area.

---

## 2. Notifications 🔔

- **Desktop Notifications**
  - Option: `pulsechat_desktop_notifications`
  - Enables browser desktop notifications for new messages (staff must allow notifications in the browser).

- **Sound Notifications**
  - Option: `pulsechat_sound_notifications`
  - Plays a sound (`assets/audio/notification.mp3`) when new messages arrive.

- **Toast Notifications**
  - Option: `pulsechat_toast_notifications`
  - Shows small pop‑up notifications inside the admin UI for key events.

---

## 3. Permissions & Access 🛡️

These options and staff capabilities control who can use what.

### 3.1 Staff Capabilities (role‑based)

Defined in `pulsechat.php` via `pulsechat_register_permissions()`:

- `view` – can see PulseChat UI.
- `send` – can send messages.
- `delete` – can delete messages (subject to additional settings).
- `create_groups` – can create group conversations.
- `manage_channels` – can configure omnichannel connections.
- `manage_automation` – can manage automation rules.
- `view_analytics` – can view analytics & reports.
- `use_ai` – can access AI tools (draft, suggestions, rewrite, translate, etc.).
- `assign` – can assign / reassign conversations.
- `view_all_channels` – can see all omnichannel conversations, not only assigned/own.

Assign these capabilities to roles in **Setup → Staff → Roles**.

### 3.2 Settings Tab Permissions

- **Allow Staff to Delete Messages**
  - Option: `pulsechat_staff_can_delete`

- **Allow Staff to Create Groups**
  - Option: `pulsechat_staff_can_create_groups`

- **Allow Leaving Groups**
  - Option: `pulsechat_allow_leave_groups`
  - If **No**, only admins can remove members or dissolve groups.

- **New Members See History**
  - Option: `pulsechat_new_members_see_history`
  - When **Yes**, newly added members see full group history.
  - When **No**, they see only messages sent after they joined.

- **Only Permitted Users**
  - Option: `pulsechat_only_permitted_users`
  - When **Yes**, only staff with the `view` capability (or admins) appear in staff lists and can use PulseChat.

---

## 4. Features 🎛️

Toggle optional functionality:

- **File Sharing**
  - `pulsechat_allow_file_sharing`
  - Enables upload of attachments via the paperclip button.

- **Audio Messages**
  - `pulsechat_allow_audio_messages`
  - Enables voice message recording and sending.

- **Reactions**
  - `pulsechat_allow_reactions`
  - Enables emoji reactions on messages.

- **Forwarding**
  - `pulsechat_allow_message_forwarding`

- **Pinning**
  - `pulsechat_allow_message_pinning`

- **Ticket Conversion**
  - `pulsechat_allow_ticket_conversion`
  - Enables “Convert to Ticket” button in chat header.

- **Typing Indicators**
  - `pulsechat_show_typing_indicators`

- **Read Receipts**
  - `pulsechat_show_read_receipts`

- **Online Status**
  - `pulsechat_show_online_status`

---

## 5. Limits & Data Management 📊

### 5.1 Limits

- **Max File Size**
  - `pulsechat_max_file_size_mb`
  - Maximum upload size for attachments (in MB).

- **Message Edit Window**
  - `pulsechat_edit_window_minutes`
  - How long after sending a message can be edited (`0` = never allowed).

- **Max Group Members**
  - `pulsechat_max_group_members`

- **Allowed File Types**
  - `pulsechat_allowed_file_types`
  - Comma‑separated list of extensions (e.g. `.jpg,.png,.pdf,.docx`).

### 5.2 Data Retention

Defined in the same Settings page:

- **Auto‑Purge After (months)**
  - `pulsechat_auto_purge_months`
  - `0` = never purge.
  - Otherwise, messages older than the specified number of months can be purged by the cleanup logic.

---

## 6. Appearance 🎨

PulseChat inherits Perfex styling and adds its own.

Key options:

- **Brand Color**
  - `pulsechat_brand_color`
  - Used for primary accents in the chat UI.

- **Chat Background**
  - `pulsechat_chat_bg`
  - Select from predefined backgrounds/themes.

There are also user‑level preferences (dark/light theme) stored per staff and applied via `data-theme` attributes in `chat_view.php`.

---

## 7. Transport (Real‑time) ⚡

PulseChat supports:

- **Built‑in Polling** – Periodic AJAX calls to fetch new messages (no external services).
- **Pusher WebSockets** – Real‑time delivery via Pusher Channels.
- **Auto** – Use Pusher when properly configured; else fall back to polling.

Relevant settings in the “Real‑time Transport” section:

- `pulsechat_transport` – `auto`, `pusher`, or `polling`
- Additional Pusher settings pulled from Perfex core settings (App ID, Key, Secret, Cluster).

You can change these at any time in the PulseChat settings; the UI indicates the active mode via the **transport bar** at the top of the chat.

---

## 8. Omnichannel Channels 🌐

To enable the omnichannel inbox:

- Toggle **Enable Omnichannel Channels**
  - Option: `pulsechat_channels_enabled`
  - When **Yes**, a **Channels** tab appears in the left sidebar and the omnichannel backend is active.

Each channel (WhatsApp, Telegram, Email, etc.) is configured in the **Channels** UI (gear icon in the Channels tab) and stored in `pc_channels` as encrypted JSON config.

See **[🌐 Omnichannel & Channels](omnichannel.md)** for per‑channel details.

---

## 9. AI Assistant 🧠

PulseChat can use OpenAI (ChatGPT) or Anthropic (Claude) for:

- Draft replies
- Smart suggestions (chips)
- Message rewriting
- Spelling/grammar
- Summaries
- Translation
- Sentiment analysis

In the **AI Assistant — Multi‑Provider** section you will find:

- **Enable AI Features**
  - `pulsechat_ai_enabled`

- **AI Provider**
  - `pulsechat_ai_provider` – `openai` or `anthropic`

- **OpenAI API Key**
  - `pulsechat_ai_api_key`

- **Anthropic API Key**
  - `pulsechat_ai_anthropic_api_key`

- **AI Model**
  - `pulsechat_ai_model` – options include:
    - OpenAI: `gpt-4o`, `gpt-4o-mini`, `gpt-3.5-turbo`
    - Anthropic: `claude-3-5-sonnet-20241022`, `claude-3-5-haiku-20241022`, `claude-3-opus-20240229`

- **System Prompt**
  - `pulsechat_ai_system_prompt`
  - Global instruction used by all AI calls (e.g. “You are a helpful customer support assistant…”).

- **Temperature**
  - `pulsechat_ai_temperature`

- **Max Tokens**
  - `pulsechat_ai_max_tokens`

- **Feature Toggles**
  - `pulsechat_ai_auto_reply` – show **Draft** button (AI reply draft).
  - `pulsechat_ai_suggestions` – show suggestion chips.
  - `pulsechat_ai_rewrite` – enable rewrite button.
  - `pulsechat_ai_translate` – enable translation button.
  - `pulsechat_ai_sentiment` – sentiment analysis helpers.

See **[🧠 AI Assistant](ai-assistant.md)** for how these appear in the UI and how they behave in conversations.

---

## 10. Licensing 🔐

PulseChat’s licensing is configured via:

- `pulsechat_verification_id`
- `pulsechat_product_token`
- `pulsechat_last_verification`
- `pulsechat_heartbeat`

These are **managed automatically** by the licensing flow and should not be edited manually. To change license:

- Deactivate PulseChat in **Setup → Modules**.
- Activate it again; the activation screen will re‑appear, letting you enter a new purchase key.

See **[🔐 Licensing](licensing.md)** for more detail.

