# 🧪 Troubleshooting & FAQ

This page covers common issues and how to resolve them when working with PulseChat.

---

## 1. Chat Doesn’t Appear in Sidebar

**Symptoms:**

- No “Chat” menu item in the admin sidebar.

**Checks:**

1. **Module enabled?**
   - Go to **Setup → Modules**.
   - Ensure **PulseChat** is **Active**.

2. **Staff permissions?**
   - Your role must have the `view` capability for PulseChat.
   - Check **Setup → Staff → Roles** and confirm:
     - `view` under PulseChat is enabled for your role.

3. **PulseChat enabled option?**
   - In PulseChat settings, ensure:
     - `pulsechat_enabled` is set to `Yes`.

---

## 2. Transport Issues (Messages Slow or Not Real‑Time)

**Symptoms:**

- Messages appear only after page refresh or with delays.
- Typing indicators / read receipts are inconsistent.

**Checks:**

1. Look at the **transport bar** at the top of the chat:
   - “Pusher WebSockets — instant delivery” (blue) or
   - “Built‑in Polling — messages refresh every ~3s” (green).

2. If using **Pusher**:
   - Confirm Pusher credentials (App ID/Key/Secret/Cluster) in Perfex settings.
   - Ensure firewall is not blocking `pusher.com`.
   - Check Pusher dashboard for errors or connection limits.

3. If using **Polling**:
   - Some delay (a few seconds) is normal; this is expected behavior.

4. To switch transport:
   - Go to **PulseChat Settings → Real‑time Transport**.
   - Choose `pusher`, `polling`, or `auto`.
   - Save and reload the chat page.

---

## 3. Files Won’t Upload

**Symptoms:**

- Upload fails silently or with an error.

**Checks:**

1. **Allowed file types**
   - Ensure the extension is allowed in `pulsechat_allowed_file_types`.

2. **Max size**
   - File size must be ≤ `pulsechat_max_file_size_mb`.

3. **Permissions**
   - Verify `modules/pulsechat/uploads/` is writable.

4. **Error logs**
   - Check Perfex logs and server error logs for upload‑related messages.

---

## 4. Clients Can’t See or Use Chat

**Symptoms:**

- Staff can use PulseChat; clients cannot see any chat widget.

**Checks:**

1. **Client chat enabled?**
   - Ensure `pulsechat_clients_enabled` is set to **Yes** in settings.

2. **Client portal hooks**
   - PulseChat injects a widget into the client portal via `init_chat.php`.
   - Ensure there are no overrides or template customizations removing this.

3. **Permissions**
   - Only logged‑in contacts can use client chat; anonymous visitors won’t see it.

---

## 5. Omnichannel Messages Not Arriving

**Symptoms:**

- WhatsApp/Telegram/Email messages aren’t appearing in Channels.

**Checks:**

1. **Omnichannel enabled?**
   - `pulsechat_channels_enabled` = Yes.

2. **Channel config**
   - For each channel type:
     - WhatsApp: token, phone ID, webhook URL configured in Meta.
     - Telegram: bot token and webhook URL configured via BotFather.
     - Email: IMAP settings correct, SSL/ports correct, credentials valid.

3. **Webhooks**
   - Confirm provider webhooks are pointing to your `ChannelWebhook`/`webhook.php` endpoint.
   - Check logs for failed signature verifications.

4. **Email polling**
   - Ensure Perfex cron is running.
   - `pulsechat_cron_email_poll` will not run without `after_cron_run`.

---

## 6. AI Buttons Not Working

**Symptoms:**

- AI toolbar is visible but:
  - Clicking Draft/Suggest/Rewrite/Translate does nothing, or
  - Toasts show “AI not configured” or HTTP errors.

**Checks:**

1. **Settings**
   - `pulsechat_ai_enabled` = Yes.
   - Provider selected (`openai` or `anthropic`).
   - Correct API key entered for the selected provider.
   - AI feature toggles (Auto‑Reply, Suggestions, Rewrite, Translate, Sentiment) enabled as desired.

2. **Network**
   - Ensure your server can reach:
     - `https://api.openai.com`
     - `https://api.anthropic.com`
   - Check for firewall/proxy blocks or DNS issues.

3. **Error text**
   - Many errors return:
     - “OpenAI API key not configured.”
     - “Text is required.”
     - HTTP status with provider error message.

4. **AI logs**
   - Look at `pc_ai_logs` for more details:
     - `action_type`, `model`, `error` (if captured), token counts.

---

## 7. Database Errors (Undefined Constants)

**Symptoms:**

- Errors like “Undefined constant `TABLE_PC_MESSAGES`”.

**Checks:**

1. Ensure the latest PulseChat code is deployed:
   - `pulsechat.php` defines `TABLE_PC_*` constants.
2. Fallbacks in `Pulsechat_model`:
   - The model defines core table constants if they’re not already defined.
3. If you still see this:
   - Confirm there aren’t duplicate/old copies of the module in other paths.

---

## 8. Still Stuck?

If you’ve tried the above and still can’t resolve the issue:

1. Collect:
   - Exact error messages.
   - Relevant logs (PHP error logs, Perfex logs).
   - Screenshots of the relevant settings screens.
2. Contact support with:
   - Your Perfex version.
   - PulseChat version.
   - Any customization you’ve done (override views, modified code, etc.).

This information will make it much easier to diagnose and fix the problem quickly.

