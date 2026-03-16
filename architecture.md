# 🧱 Architecture & Internals

This page is a high‑level technical overview of PulseChat’s architecture: how the module is structured, key classes, and important flows you may need to understand when debugging or extending the module.

---

## 1. Module Entry Point

**File:** `modules/pulsechat/pulsechat.php`

Responsibilities:

- Declares module metadata (name, version, author).
- Defines constants:
  - `PULSECHAT_MODULE_NAME`, `PULSECHAT_VERSION`
  - Path/URL constants: `PULSECHAT_MODULE_PATH`, `PULSECHAT_MODULE_URL`, upload paths.
  - DB table constants: `TABLE_PC_*` (conversations, messages, participants, etc.).
- Loads licensing core:
  - `core/Apiinit.php`
  - `third_party/node.php`
  - `vendor/autoload.php` (if present, for JWT/Requests).
- Registers hooks:
  - `register_activation_hook`
  - `register_language_files`
  - `hooks()->add_action('admin_init', 'pulsechat_register_permissions')`
  - `hooks()->add_action('app_init', 'pulsechat_actLib')` (licensing)
  - `hooks()->add_action('pre_activate_module', 'pulsechat_sidecheck')`
  - `hooks()->add_action('pre_deactivate_module', 'pulsechat_deregister')`
  - `hooks()->add_action('after_cron_run', 'pulsechat_cron_email_poll')` (email polling)
- Adds admin menu entry.
- Adds client‑side hooks for injecting the chat widget in the client portal when enabled.

---

## 2. Controllers

### 2.1 `controllers/Pulsechat.php`

Main admin controller for the chat UI:

- Extends `AdminController`.
- Constructor:
  - Ensures helper is loaded (`pulsechat/pulsechat`).
  - Redirects if module disabled.
  - Checks staff `view` capability.
  - Loads `Pulsechat_model`.
  - Determines transport (`pusher` vs. `polling`).
- Key methods:
  - `chat()` – renders the full chat view (`views/admin/chat_view.php`).
  - `complete_setup()` – handles initial transport wizard.
  - Numerous AJAX endpoints for:
    - Fetching contacts, groups, clients.
    - Sending/receiving messages (non‑channel).
    - Reactions, stars, pins, file uploads.
    - Search, starred/pinned/global views.
    - Export and analytics helpers (for in‑app analytics).

### 2.2 `controllers/Pulsechat_Channels.php`

Handles **omnichannel** and AI endpoints:

- Channels:
  - `conversations`, `conversation`, `mark_read`, `send_channel_message`, `contact`, `link_contact`, channel CRUD, etc.
- AI endpoints:
  - `ai_auto_reply()`
  - `ai_suggestions()`
  - `ai_rewrite()`
  - `ai_spelling()`
  - `ai_summary()`
  - `ai_sentiment()`
  - `ai_translate()`
  - These delegate to `PulsechatAI` and return normalized JSON.
- Analytics:
  - `analytics()`, `analytics_export()` for channel metrics (messages per day, per channel, AI usage).

### 2.3 `controllers/Env_ver.php`

Licensing activation controller:

- `activate()` and `upgrade_database()` call `modules\pulsechat\core\Apiinit::pre_validate()` and return JSON for the activation UI.

### 2.4 Webhook Controllers

- `controllers/webhooks/ChannelWebhook.php`
  - Receives webhooks from external providers (WhatsApp, Telegram, etc.).
  - Dispatches to the appropriate channel adapter.

- `Pulsechat_webhook.php` / `webhook.php`
  - Compatibility or convenience endpoints for some providers.

---

## 3. Models

### 3.1 `models/Pulsechat_model.php`

Central data access layer:

- Extends `App_Model`.
- Ensures UTF‑8mb4 connection (for emojis).
- Major responsibilities:
  - **Users & presence:**
    - `get_staff_users()`, `get_group_users()`, `get_my_groups()`, status management.
  - **Conversations:**
    - CRUD for `pc_conversations`, participant management in `pc_participants`.
    - Muting, pinning, archiving.
  - **Messages:**
    - Insert messages into `pc_messages`.
    - Apply reactions (`pc_reactions`), stars (`pc_starred_messages`), pins (`pc_pinned_messages`).
    - Load paginated messages for various conversation types.
  - **Files:**
    - Manage `pc_shared_files`, ensure messages and files stay in sync.
  - **Omnichannel:**
    - Handle `pc_channels`, `pc_external_contacts`, `pc_contact_identities`.
    - Automation rules (`pc_automation_rules`).
    - Analytics data in `pc_analytics_cache`, `pc_assignments_log`.
  - **AI Logs:**
    - `log_ai_usage()` writes to `pc_ai_logs`.
    - `get_ai_usage_stats()` aggregates AI usage for analytics.

All SQL uses `db_prefix()` to respect Perfex’s table prefix.

---

## 4. Libraries

### 4.1 `libraries/PulsechatAI.php`

AI engine abstraction:

- Handles all AI use‑cases:
  - Auto‑reply, suggestions, rewrite, spelling, summary, sentiment, translation, chatbot mode.
- Exposes methods like:
  - `generateAutoReply()`
  - `generateSuggestions()`
  - `rewriteMessage()`
  - `correctSpelling()`
  - `summarizeConversation()`
  - `analyzeSentiment()`
  - `translate()`
  - `chatbotReply()`
- Internally dispatches between:
  - OpenAI Chat Completions API.
  - Anthropic Messages API.
- Logs usage to `pc_ai_logs` via `Pulsechat_model`.

### 4.2 Channel Adapters

Namespace: `libraries/channels/`

- `ChannelAdapterInterface.php` – defines a common interface for channel adapters.
- Adapters for specific platforms:
  - `WhatsappAdapter.php`
  - `TelegramAdapter.php`
  - `EmailAdapter.php`
  - Potentially others (SMS, Slack, Viber, etc.).
- `ChannelManager.php`:
  - Orchestrates outbound send and inbound processing across adapters.
  - Handles rate limits, error logging, media downloads, and attachment handling.

### 4.3 Contact Matching

- `libraries/ContactMatcher.php`:
  - Converts channel identifiers (phone, email, chat IDs) into `pc_external_contacts`.
  - Attempts to match to Perfex clients/contacts/leads via core tables.

---

## 5. Views & Assets

### 5.1 Views

- `views/admin/chat_view.php`
  - Main admin chat page (full three‑panel UI).
  - Renders:
    - Sidebar (tabs, search, contact lists).
    - Content (messages and composer).
    - Details panel (CRM info, participants, analytics).
  - Injects configuration JS object `pulsechatConfig`.

- `views/admin/settings.php`
  - Settings panel rendered in `admin/pulsechat/settings`.
  - Organized sections:
    - General, Notifications, Permissions, Features, Limits, Data.
    - Omnichannel options.
    - AI Assistant configuration.

- `views/admin/settings_page.php`
  - Wrapper page for module settings (includes `settings.php` inside a Perfex panel with Save button).

- `views/activate.php`
  - Licensing activation screen used by `Apiinit::activate()`.

### 5.2 JavaScript

Located in `assets/js/`:

- `pulsechat-full.js`:
  - Main frontend logic for the chat UI.
  - Manages:
    - State (`state.activeChat`, contacts, groups, clients).
    - Message list rendering, infinite scroll.
    - Typing indicators, presence, read receipts.
    - Emoji picker and reactions.
    - Local display settings (theme, layout).
    - AI toolbar handlers (when channels are disabled).

- `pulsechat-channels.js`:
  - Omnichannel frontend.
  - Renders the Channels list, filters, and channel composer.
  - Handles AI actions in channel mode.
  - Implements canned responses and automation rule UI.

- `pulsechat-analytics.js`:
  - Renders charts and analytics for omnichannel usage.

### 5.3 CSS

- `assets/css/pulsechat-full.css`
  - Full styling for the admin chat UI.
  - Includes:
    - Layout (sidebar, content, details).
    - Emoji picker, context menus, modals.
    - Dark mode adjustments.

---

## 6. Migrations & Install

### 6.1 `install.php`

Executed on module activation via `pulsechat_activation_hook()`:

- Creates core tables (initial versions of `pc_conversations`, `pc_participants`, `pc_messages`, etc.).
- Adds default options such as:
  - `pulsechat_enabled`
  - `pulsechat_clients_enabled`
  - `pulsechat_transport`
  - Defaults for notifications, features, file limits, etc.

### 6.2 Migrations

Located in `migrations/`:

- `100_version_100.php` and subsequent – baseline and incremental changes.
- `122_version_122.php` – **Omnichannel Foundation**:
  - Adds `pc_channels`, `pc_external_contacts`, `pc_contact_identities`, `pc_automation_rules`, `pc_ai_logs`, etc.
  - Extends existing tables for channel support (`channel_type`, `subject`, `delivery_status`, `ai_suggested`, etc.).
  - Seeds initial PulseChat AI options.
- `123_version_123.php` – ensures `pc_realtime_events` exists.
- `124_version_124.php` – **Multi‑Provider AI**:
  - Adds options:
    - `pulsechat_ai_provider`
    - `pulsechat_ai_anthropic_api_key`

Perfex’s migration system ensures these run in order when you update the module.

---

## 7. Licensing Core

See **[🔐 Licensing](licensing.md)** for details, but from an architecture standpoint:

- `core/Apiinit.php` and `libraries/Pulsechat_aeiou.php` are independent of the rest of the module logic.
- They control:
  - Whether the module should be activated at all.
  - Whether it should stay active based on periodic remote verification.

If licensing fails, they call `app_modules->deactivate('pulsechat')`, which effectively disables the entire module until reactivated.

---

## 8. Extensibility Notes

When extending PulseChat:

- **New Channel Types:**
  - Implement `ChannelAdapterInterface` in a new adapter under `libraries/channels/`.
  - Register configuration fields in the channel settings UI.
  - Update `pc_channels` handling as needed.

- **New AI Actions:**
  - Add a new method to `PulsechatAI`.
  - Add a new controller endpoint in `Pulsechat_Channels`.
  - Wire a new button or UI control in `pulsechat-full.js` / `pulsechat-channels.js`.
  - Consider logging to `pc_ai_logs` for analytics.

- **Custom Integrations:**
  - Use `Pulsechat_model` for DB operations rather than writing direct SQL.
  - Reuse existing utilities (e.g. ContactMatcher) to avoid duplicate logic.

Keep in mind:

- PulseChat relies heavily on **Perfex hooks** and **App_Controller** behaviors; follow those patterns to remain compatible with core updates.
- Always add a migration for schema changes rather than modifying `install.php` only.

