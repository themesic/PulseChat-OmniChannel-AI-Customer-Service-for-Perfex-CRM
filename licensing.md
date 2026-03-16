# 🔐 Licensing

PulseChat uses an Envato‑style licensing system consistent with other Themesic modules (e.g. the GraphQL API module). This page explains how licensing works, how to activate PulseChat, and what happens during periodic checks.

---

## 1. Overview

Licensing is implemented via:

- `core/Apiinit.php` – core activation and verification logic.
- `libraries/Pulsechat_aeiou.php` – Envato & remote API communication.
- `third_party/node.php` – shared endpoint constants.
- `controllers/Env_ver.php` – handles activation requests from the UI.
- `views/activate.php` – activation form shown on first module activation.

Configuration is stored in Perfex options:

- `pulsechat_verification_id`
- `pulsechat_product_token`
- `pulsechat_last_verification`
- `pulsechat_heartbeat`

You normally don’t touch these directly; they are managed by the module.

---

## 2. Activation Flow 🧾

### 2.1 First activation

1. Go to **Setup → Modules** in Perfex.
2. Click **Activate** on **PulseChat**.
3. Before the module actually activates, Perfex fires a `pre_activate_module` hook.
   - In `pulsechat.php`:

     ```php
     hooks()->add_action('pre_activate_module', PULSECHAT_MODULE_NAME . '_sidecheck');
     function pulsechat_sidecheck($module)
     {
         if (PULSECHAT_MODULE_NAME === $module['system_name']) {
             modules\pulsechat\core\Apiinit::activate($module);
         }
     }
     ```

4. `Apiinit::activate($module)` checks whether `pulsechat_verification_id` exists:
   - If not present:
     - It renders the **activation view** (`modules/pulsechat/views/activate.php`).
     - That view contains a form for the **purchase key**, posting to `Env_ver::activate`.
     - Script exits right after rendering; normal module activation is paused until activation succeeds.

### 2.2 Activation screen

The activation page (`views/activate.php`):

- Explains what key is needed.
- Has a form:
  - `purchase_key` (required)
  - Hidden `module_name` and `original_url`
  - `submit_url` points to `admin/pulsechat/env_ver/activate`.
- Uses AJAX (`appValidateForm`) to submit.

On success:

- The controller returns a JSON object with `status = true` and `original_url`.
- The script redirects the browser to `original_url`, which is the Perfex activation URL (`modules/activate/pulsechat`).
- Perfex then activates the module normally.

On failure:

- The error message from the remote validation is shown as a toast (`alert_float("danger", ...)`).

---

## 3. Env_ver Controller

`controllers/Env_ver.php` has two main methods:

- `activate()`
- `upgrade_database()`

Both call:

```php
$res = modules\pulsechat\core\Apiinit::pre_validate(
    $this->input->post('module_name'),
    $this->input->post('purchase_key')
);
```

If `$res['status']` is true, they also set:

```php
$res['original_url'] = $this->input->post('original_url');
```

and return the JSON response.

---

## 4. `Apiinit::pre_validate` 🔎

In `core/Apiinit.php`, `pre_validate($module_name, $code)`:

1. Makes sure a purchase key was provided.
2. Ensures the same purchase code is not already used by another active module:
   - Iterates all activated modules.
   - Checks their stored `_verification_id` values.
3. Uses `Pulsechat_aeiou::getPurchaseData($code)` to query Envato’s API.
4. Validates:
   - The purchase data exists and is valid.
   - The Envato item ID matches the module’s item ID (from the module header URI).
5. Compiles metadata (user agent, domain, IP, OS, purchase code, etc.) into a JSON payload.
6. Sends a registration request to your license server:
   - `REG_PROD_POINT` (from `third_party/node.php`).
7. If registration succeeds:
   - Stores `verification_id` (base64‑encoded).
   - Stores `product_token`.
   - Updates `last_verification`.
   - Clears any `heartbeat` entry.

If the license server is temporarily unreachable (5xx/404), the code:

- Stores a heartbeat entry (status, end point, code).
- Treats it as a **soft success** (`status: true`) so the admin can continue.

---

## 5. Periodic Verification ⏱️

Two parts periodically verify the license:

### 5.1 `Apiinit::the_da_vinci_code`

Called in `pulsechat.php` on module load:

```php
modules\pulsechat\core\Apiinit::the_da_vinci_code(PULSECHAT_MODULE_NAME);
```

It:

- Reads `pulsechat_verification_id` and `pulsechat_product_token`.
- Decodes the JWT.
- Ensures:
  - The item ID matches the module’s URI item ID.
  - Buyer and purchase code match.
- Checks whether the next verification time has passed (`check_interval`).
- If due, sends a validation request to:
  - `VAL_PROD_POINT` (from `third_party/node.php`).
- Logs heartbeat errors as needed.

If verification fails and a token exists, it **deactivates PulseChat** via:

```php
get_instance()->app_modules->deactivate($module_name);
```

Importantly, if **no** verification data exists yet (fresh install, no key entered), it returns early to avoid auto‑deactivation.

### 5.2 `Pulsechat_aeiou::validatePurchase`

Called on every `app_init` via `pulsechat_actLib`:

```php
hooks()->add_action('app_init', PULSECHAT_MODULE_NAME . '_actLib');
function pulsechat_actLib()
{
    $CI = &get_instance();
    $CI->load->library(PULSECHAT_MODULE_NAME . '/Pulsechat_aeiou');
    $envato_res = $CI->pulsechat_aeiou->validatePurchase(PULSECHAT_MODULE_NAME);
    if (!$envato_res) {
        set_alert('danger', 'One of your modules failed its verification and got deactivated. Please reactivate or contact support.');
    }
}
```

`validatePurchase()`:

- Loads and decodes the stored JWT (`_product_token`).
- Checks:
  - Item ID.
  - Buyer.
  - Purchase code.
- If the check interval has elapsed, triggers a remote validation call to `VAL_PROD_POINT`.
- On repeated failure:
  - Deactivates the module and returns `false`.

If **no** verification data exists yet (never activated), it simply returns `true` and does not deactivate or show alerts.

---

## 6. Deactivation 🔄

When you deactivate PulseChat in **Setup → Modules**:

- Before actual deactivation, Perfex calls `pre_deactivate_module`.
  - `pulsechat_deregister` is hooked there:

    ```php
    hooks()->add_action('pre_deactivate_module', PULSECHAT_MODULE_NAME . '_deregister');
    function pulsechat_deregister($module)
    {
        if (PULSECHAT_MODULE_NAME === $module['system_name']) {
            delete_option(PULSECHAT_MODULE_NAME . '_verification_id');
            delete_option(PULSECHAT_MODULE_NAME . '_last_verification');
            delete_option(PULSECHAT_MODULE_NAME . '_product_token');
            delete_option(PULSECHAT_MODULE_NAME . '_heartbeat');
        }
    }
    ```

- This **clears the license state** locally.
- Next time you click **Activate**, the activation screen appears again and you can enter a new key.

---

## 7. Common Licensing Issues & Fixes

### 7.1 “One of your modules failed its verification and got deactivated…”

This alert appears when `validatePurchase()` returns `false`. Possible causes:

- Token or verification ID is invalid or tampered with.
- Remote validation endpoint (`VAL_PROD_POINT`) reports the license as invalid or revoked.
- There was a network error, and the grace/heartbeat window has expired.

**What to do:**

1. Deactivate PulseChat in **Setup → Modules**.
2. Activate it again:
   - Enter your purchase key in the activation form.
3. If the error persists, contact support with:
   - Your license key.
   - The exact error message (from the toast/alert, or server logs).

### 7.2 Activation form doesn’t appear

If you click **Activate** and the activation form doesn’t show:

- Ensure the `pre_activate_module` hook is present in `pulsechat.php` (it should be if you’re on this code).
- Ensure there are **no PHP fatal errors** before the hook runs.
- Check the Perfex **error logs** for hints.

### 7.3 Moving PulseChat to Another Domain

Licensing data includes:

- Activated domain.
- Purchase code.
- Token.

If you move Perfex to a new domain:

1. Deactivate PulseChat on the old environment if possible.
2. Ensure the new environment has:
   - A **fresh copy** of the module.
   - No leftover `pulsechat_verification_id` / `pulsechat_product_token` values.
3. Activate PulseChat on the new domain and enter your purchase key again.
4. If the license server reports the key already in use, contact support to reset the activation or free the key from the old domain.

---

## 8. Security Notes

- License validation is done via **HTTPS** requests to:
  - Envato’s API (`api.envato.com`) and
  - Your own license endpoints (`REG_PROD_POINT`, `VAL_PROD_POINT`).
- JWT tokens are signed and validated using `Firebase\JWT` with `HS512`.
- Verification IDs are stored **base64‑encoded**, not encrypted; assume they are opaque internal references, not secrets.

For security, avoid:

- Manually editing license options in the database.
- Sharing your purchase key, token, or verification ID with untrusted parties.

