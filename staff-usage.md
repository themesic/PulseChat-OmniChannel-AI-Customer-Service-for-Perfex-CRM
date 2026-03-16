# 💬 Using PulseChat (Staff)

This page shows how staff members use PulseChat day‑to‑day: finding conversations, sending messages, managing groups and clients, and using the AI assistant.

---

## 1. Opening PulseChat

Once the module is enabled and you have the `view` capability:

- In the admin sidebar, click **Chat** (PulseChat).
- This opens the **full browser view** at `admin/pulsechat/chat` with a **three‑panel layout**:
  - **Left:** Contacts / Groups / Clients / Channels
  - **Center:** Conversation list + messages
  - **Right:** Details panel (participant info, CRM context, etc.)

---

## 2. Sidebar: Tabs & Search 🔎

At the top of the sidebar you’ll see:

- Your **avatar** and **status** (`online`, `away`, `busy`, `offline`).
  - Click the status dropdown to change your presence.

### Tabs

- **Staff**
  - Direct messages with other staff members.

- **Groups**
  - Group conversations (multi‑staff).

- **Clients** (if enabled by admin)
  - Conversations with clients (company contacts).

- **Channels** (if omnichannel is enabled)
  - External channels such as WhatsApp, Telegram, Email, etc.

Click a tab to switch the list below to that type.

### Sidebar Search

- Use the search box to **filter contacts** by name or company.
- Click the “×” button to clear the search.

---

## 3. Starting & Opening Conversations 🗣️

### 3.1 Direct staff chat

1. Click the **Staff** tab.
2. Find the staff member you want to chat with.
3. Click their name to open a **direct conversation**.

### 3.2 Group conversations

1. Click the **Groups** tab.
2. To open an existing group:
   - Click the group name.
3. To create a new group:
   - Use the **Create Group** button in the sidebar footer (only if your role has `create_groups` or you are admin).
   - Choose a **group name** and **members**.
   - Save: the group appears in the list and opens automatically.

### 3.3 Client conversations

If `pulsechat_clients_enabled` is on:

1. Click the **Clients** tab.
2. Select a client row (company or contact) to open / create a chat thread.
3. Messages you send are visible to that client in the **client portal** if they open chat there.

### 3.4 Channel conversations

See **[🌐 Omnichannel & Channels](omnichannel.md)** for full details, but at a glance:

1. Click the **Channels** tab.
2. Use filters (Status, Priority, Channel Type) to narrow conversations.
3. Click a conversation to open it in the center panel.

---

## 4. Sending Messages ✍️

In the center panel, at the bottom:

- **Emoji button** – opens the emoji picker.
- **Attach button** – upload files.
- **Audio button** – record and send voice messages.
- **Text area** – type your message.
- **Send button** – sends the current message.

Keyboard behavior:

- **Enter** – send the message.
- **Shift+Enter** – insert a newline.

---

## 5. Message Actions 🧰

Hover over a message (or use the options menu) to see available actions (subject to settings and permissions):

- **Reply** – quote a message and reply inline; a reply bar appears above the composer.
- **Forward** – send this message to another conversation.
- **Copy** – copy message text.
- **React** – add an emoji reaction.
- **Star** – mark message as important; appears in “Starred Messages”.
- **Pin** – pin message to the conversation (visible at the top).
- **Edit** – edit your own message (within the edit window if configured).
- **Delete** – remove your own message (if allowed).

The right‑click context menu and per‑message “three dots” menu both offer these actions.

---

## 6. Groups & Participants 👥

In a **group** conversation:

- Open the **Details** panel (right side).
- See:
  - Group name and description.
  - Member list.
  - Creation time and creator.

If you have permission:

- **Rename group** – via group options.
- **Add members** – select additional staff.
- **Remove members** – remove a member from the group.
- **Leave group** – if `pulsechat_allow_leave_groups` is enabled.

System messages (like “Alice renamed the group…”) are inserted to keep context.

---

## 7. Starred, Pinned & Shared Files 📌

The **sidebar footer** includes:

- **All Starred Messages** button
  - Shows a cross‑conversation list of messages you have starred.

- **All Pinned Messages** button
  - Lists pinned messages across conversations.

Inside a conversation:

- Pinned messages appear in a **pinned bar**.
- Shared files are accessible via the **paperclip** icon in the header, opening a shared files view.

---

## 8. Search 🔍

Use the **Search Messages** button in the conversation header to:

- Open the **Search modal**.
- Enter a search term to search within the current conversation.
- See results with context; click to jump to a specific message.

---

## 9. AI Assistant 🧠 (Staff View)

If AI is enabled (`pulsechat_ai_enabled = 1`) and you have the `use_ai` capability:

- An **AI toolbar** appears above the composer:
  - **Draft** – AI Draft Reply
  - **💡 Suggest** – Smart suggestions
  - **✏️ Rewrite** – Improve text
  - **🌐 Translate** – Translate text

### 9.1 Draft Reply

1. Ensure you’ve selected a conversation (staff, group, client, or channel).
2. Click **Draft**.
3. PulseChat sends recent conversation history to the AI.
4. The AI’s proposed reply is inserted into the composer for you to review and send.

### 9.2 Smart Suggestions

1. Click **Suggest**.
2. The AI returns 2–3 short replies as **chips** displayed just above the composer.
3. Click a chip to fill the composer with that suggestion.

### 9.3 Rewrite

1. Type a reply in the composer.
2. Click **Rewrite**.
3. AI rewrites your text (more professional / friendly, based on configuration) and replaces your draft.

### 9.4 Translate

1. Type a message in your language.
2. Click **Translate**.
3. AI translates the text (by default to English; the admin may adjust behavior) and replaces your draft.

See **[🧠 AI Assistant](ai-assistant.md)** for more technical detail.

---

## 10. Analytics & Channels 📊

If omnichannel analytics are enabled:

- A **bar chart** icon appears in the sidebar footer when **Channels** are active.
- Click it to open analytics dashboards (volume by channel, response times, AI usage, etc.).

This is primarily for supervisors and admins; normal agents may have read‑only or no access depending on their role.

---

## 11. Tips & Best Practices ✅

- **Use statuses** honestly so colleagues know if you’re available.
- **Star important messages** so you can find them quickly later.
- Use **groups** for internal team channels (e.g. “Support Team”, “Sales Europe”).
- For client conversations:
  - Always keep a professional tone (the default AI system prompt helps with this).
  - Use **Convert to Ticket** when a chat needs a formal support ticket.
- If something feels off (missing messages, slow updates), check the **transport bar** at the top to confirm if you’re on Pusher WebSockets or polling, and contact an admin if needed.

