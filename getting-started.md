# 🚀 Getting Started

This page walks you through installing, activating, and opening PulseChat for the first time.

---

## 1. Requirements

- **Perfex CRM** ≥ `3.0.0`
- PHP version compatible with your Perfex installation (PulseChat itself does not require special extensions beyond what Perfex already uses)

---

## 2. Installation

1. **Upload the module**
   - Copy the `pulsechat` module folder into your Perfex installation under:
     - `modules/pulsechat`
   - Ensure all subfolders are present:
     - `controllers`, `models`, `views`, `assets`, `migrations`, `language`, `libraries`, `core`, `third_party`, `vendor`, etc.

2. **Verify file permissions**
   - The webserver/PHP user must be able to read the module files.
   - The upload directory must be writable:
     - `modules/pulsechat/uploads/`

3. **Log in as admin**
   - Log into your Perfex CRM as a staff member with **admin** privileges.

---

## 3. First‑time Setup Wizard ⚙️

On first use as an admin:

1. Go to **PulseChat** from the left sidebar:
   - You’ll see a **chat icon** (`fa-comments`) added by the module.
   - Click it to open the full PulseChat page (`admin/pulsechat/chat`).

2. If initial setup isn’t completed, you’ll see a **wizard**:
   - This guides you through selecting the **transport mode**:
     - **Pusher WebSockets** – instant real‑time messaging (requires Pusher credentials).
     - **Built‑in Polling** – no external service, messages refresh every ~3 seconds.
   - For Pusher, you’ll be asked for:
     - App ID, Key, Secret, Cluster

3. After saving, the wizard marks setup as complete and redirects you into the main chat UI.

You can always change transport later from **PulseChat Settings**, so the wizard is just a convenience for first‑time installs.

---

## 4. Where to Find PulseChat in Perfex

- **Admin area**
  - Left sidebar: a **“Chat”** entry (PulseChat) under the main navigation.
  - Opens the full **three‑panel chat UI** (`chat_view.php`).

- **Client area (optional)**
  - If **client chat** is enabled in settings, a floating chat widget appears in the client portal, allowing staff ↔ client conversations.

- **Modules page**
  - **Setup → Modules → PulseChat**
    - Links:
      - **Settings** – opens `admin/pulsechat/settings` (full PulseChat configuration).
      - **Open Chat** – shortcut to `admin/pulsechat/chat`.

---

## 5. Basic Data Model (High‑Level)

PulseChat stores its data in dedicated tables (all prefixed via `db_prefix()`):

- `pc_conversations` – all conversations (staff DM, groups, clients, omnichannel)
- `pc_participants` – participants in each conversation
- `pc_messages` – individual messages (inbound/outbound, media, system messages)
- `pc_channels` – omnichannel connections (WhatsApp, Telegram, Email, etc.)
- `pc_external_contacts`, `pc_contact_identities` – external contact mapping for channels
- `pc_ai_logs` – AI usage and cost tracking
- `pc_analytics_cache`, `pc_assignments_log` – analytics and assignment history

You don’t need to manually manage these tables—Perfex migrations handle creation and updates—but understanding them helps when troubleshooting or building integrations.

---

## 6. Next Steps

Once PulseChat is installed and activated:

- Configure it from **[⚙️ Admin Configuration](admin-configuration.md)**:
  - Transport, permissions, limits, omnichannel, AI providers, etc.
- Learn how staff use it day‑to‑day in **[💬 Using PulseChat (Staff)](staff-usage.md)**.
- If you plan to connect WhatsApp, Telegram, Email, or others, see **[🌐 Omnichannel & Channels](omnichannel.md)**.

