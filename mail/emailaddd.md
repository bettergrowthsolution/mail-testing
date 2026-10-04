Here's exactly where to update for each new email (using `ram@` and `lakhan@` as examples):

---

## 🎯 The 6 places to update

### 1️⃣ **Supabase → Authentication → Users**
**Location:** Supabase Dashboard → **Authentication** → **Users** → **Add User**

Add each new user:
- Email: `ram@bettergrowthsolutions.com`
- Password: (set one)
- ✅ Auto Confirm

**Do this for:** `ram@`, `lakhan@` — plus `accounting@` and `rajvardhan@` if not already added.

---

### 2️⃣ **Supabase → SQL Editor (RLS policy)**
**Location:** Supabase → **SQL Editor** — run this if you want the new user to see **their own** received emails:

```sql
drop policy if exists "users read own inbound" on received_emails;
create policy "users read own inbound"
  on received_emails for select
  to authenticated
  using (
    recipient_email = (auth.jwt() ->> 'email')
    or (auth.jwt() ->> 'email') = 'admin@bettergrowthsolutions.com'
  );
```

*(This already works for any user whose email matches `recipient_email` — so **no change is needed** unless you've hardcoded addresses. Verify it says `auth.jwt() ->> 'email'`, not a specific address.)*

---

### 3️⃣ **Supabase Edge Function → `send-email` → Code**
**Location:** Supabase → **Edge Functions** → **send-email** → **Code**

Find the `ALLOWED_SENDERS` array near the top:

```ts
const ALLOWED_SENDERS = [
  "hr@bettergrowthsolutions.com",
  "accounting@bettergrowthsolutions.com",
  "rajvardhan@bettergrowthsolutions.com",
];
```

Add the new ones:

```ts
const ALLOWED_SENDERS = [
  "hr@bettergrowthsolutions.com",
  "accounting@bettergrowthsolutions.com",
  "rajvardhan@bettergrowthsolutions.com",
  "ram@bettergrowthsolutions.com",
  "lakhan@bettergrowthsolutions.com",
];
```

Click **Deploy**.

---

### 4️⃣ **Cloudflare → Email Routing → Routes**
**Location:** Cloudflare Dashboard → **Email** → **Email Routing** → **Routes**

For each new address, create a rule:
- Custom address: `ram` (and `lakhan`)
- Action 1: **Send to a Worker** → `email-processor`
- Action 2: **Send to an email** → `bettergrowtthsolutions@gmail.com`
- Save

**Skip `info@`** (leave it forwarding to `bettergrowthsolution@gmail.com` only).

---

### 5️⃣ **`admin.html` → SENDERS array**
**Location:** your `admin.html` file → search for:

```js
const SENDERS = [
  "hr@bettergrowthsolutions.com",
  "accounting@bettergrowthsolutions.com",
  "rajvardhan@bettergrowthsolutions.com"
];
```

Add:

```js
const SENDERS = [
  "hr@bettergrowthsolutions.com",
  "accounting@bettergrowthsolutions.com",
  "rajvardhan@bettergrowthsolutions.com",
  "ram@bettergrowthsolutions.com",
  "lakhan@bettergrowthsolutions.com"
];
```

This is what makes the "Send as" dropdown in the admin composer show the new senders.

---

### 6️⃣ **`admin.html` → user-chip order (optional)**
**Location:** `admin.html` → search for:

```js
const order = ["all", "hr@bettergrowthsolutions.com", "accounting@bettergrowthsolutions.com",
               "rajvardhan@bettergrowthsolutions.com", "admin@bettergrowthsolutions.com"];
```

Add the new addresses to the end:

```js
const order = ["all", "hr@bettergrowthsolutions.com", "accounting@bettergrowthsolutions.com",
               "rajvardhan@bettergrowthsolutions.com", "ram@bettergrowthsolutions.com",
               "lakhan@bettergrowthsolutions.com", "admin@bettergrowthsolutions.com"];
```

*(This is optional — the chips will still show up, just in a different order.)*

---

## 📋 Quick reference table

| # | Where | What to update |
|---|---|---|
| 1 | Supabase → Auth → Users | Add `ram@`, `lakhan@` |
| 2 | Supabase → SQL Editor | Nothing (RLS is dynamic) |
| 3 | Supabase → Edge Functions → send-email → Code | `ALLOWED_SENDERS` array |
| 4 | Cloudflare → Email Routing → Routes | Create rules for `ram@`, `lakhan@` |
| 5 | `admin.html` | `SENDERS` array |
| 6 | `admin.html` | `order` array (optional) |

---

## ✅ Do this for each new email

For `ram@bettergrowthsolutions.com`:
1. Add user in Supabase Auth
2. Add to `ALLOWED_SENDERS`
3. Create Email Routing rule (Worker + Gmail backup)
4. Add to `SENDERS` in admin.html
5. Redeploy the Edge Function

Then repeat for `lakhan@` — same 5 steps.

---

## 🚫 Don't touch

- `info@bettergrowthsolutions.com` — never modify its rule
- `received_emails` table — no changes needed
- `activity_logs` — no changes needed
- `index.html` — no changes needed (login is dynamic)
- `dashboard.html` — no changes needed (shows own data)

---

That's all. Once you've made those edits, the new addresses work everywhere: login, sending, receiving, admin panel, and logs.
