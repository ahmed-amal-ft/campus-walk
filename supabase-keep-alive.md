# Supabase keep-alive setup (AAST Campus Walks)

Keeps a free-plan Supabase project from pausing for inactivity by sending one small request to the database every day. Your survey schema stays unchanged, and no fake survey responses are written.

Setup has two parts:

1. A tiny **ping function** in your database. You paste it into the Supabase SQL Editor once.
2. A **scheduler** that calls the function daily. Pick **Option A (GitHub Actions)** or **Option B (cron-job.org)**.

---

## Part 1: Add the ping function

1. Open your project in the Supabase Dashboard.
2. Go to **SQL Editor → New query**.
3. Paste the SQL below and click **Run**.

```sql
-- Keep-alive ping: returns the current time. Reads no survey data, writes nothing.
create or replace function public.keep_alive()
returns timestamptz
language sql
stable
as $$ select now() $$;

revoke all on function public.keep_alive() from public;
grant execute on function public.keep_alive() to anon, authenticated;
```

It's safe to run again: `create or replace` just overwrites the function.

### Find your project URL and public key

In the Dashboard, go to **Project Settings → API** (or **API Keys**) and copy:

- **Project URL**, for example `https://abcdefghijkl.supabase.co`
- **Public key**, which is either:
  - the **publishable key** (starts with `sb_publishable_...`), or
  - the legacy **anon key** (a long string starting with `eyJ...`).

Either key works. **Never use the `service_role` or secret key** here.

### Test it once by hand (optional)

On macOS, Linux or Windows PowerShell 7+:

```bash
curl -X POST "https://YOUR-PROJECT.supabase.co/rest/v1/rpc/keep_alive" \
  -H "apikey: YOUR_PUBLIC_KEY" \
  -H "Content-Type: application/json" \
  -d '{}'
```

A timestamp in the reply, such as `"2026-11-07T06:00:00.123+00:00"`, means it works.

---

## Part 2, Option A: GitHub Actions (free, runs from GitHub)

### Steps

1. Sign in to [github.com](https://github.com) and create a **new repository**. Make it **Private**; see the note below.
2. In the repository, go to **Settings → Secrets and variables → Actions → New repository secret** and add two secrets:
   - `SUPABASE_URL`: your project URL, e.g. `https://abcdefghijkl.supabase.co` (no trailing slash)
   - `SUPABASE_KEY`: your publishable or anon key
3. Create a file at exactly this path in the repository: `.github/workflows/keep-alive.yml`
   (On GitHub: **Add file → Create new file**, then type the full path including the folders.)
4. Paste the content below and **Commit**.
5. Test it: go to the **Actions** tab, select **Supabase keep-alive**, click **Run workflow**. A green tick means it worked.

### `.github/workflows/keep-alive.yml`

```yaml
name: Supabase keep-alive

on:
  schedule:
    - cron: "17 6 * * *"   # every day at 06:17 UTC (08:17 Cairo)
  workflow_dispatch:        # allows running it manually from the Actions tab

jobs:
  ping:
    runs-on: ubuntu-latest
    timeout-minutes: 5
    steps:
      - name: Ping Supabase
        run: |
          response=$(curl -sS --fail-with-body -X POST \
            "${{ secrets.SUPABASE_URL }}/rest/v1/rpc/keep_alive" \
            -H "apikey: ${{ secrets.SUPABASE_KEY }}" \
            -H "Content-Type: application/json" \
            -d '{}')
          echo "Supabase replied: $response"
```

### Notes

- **Failures:** If the ping fails (for example, the project is already paused or the key is wrong), the run turns red, and GitHub emails you by default.
- **Public repositories:** GitHub disables scheduled workflows in public repositories after about 60 days without commits. A private repository avoids this. If you get an email saying the workflow was disabled, re-enable it from the **Actions** tab.
- **Timing:** GitHub may run scheduled jobs some minutes late. That doesn't matter here.

---

## Part 2, Option B: cron-job.org (free, no code, web form)

1. Create a free account at [cron-job.org](https://cron-job.org).
2. Click **Create cronjob** and fill in:
   - **Title:** `Supabase keep-alive`
   - **URL:** `https://YOUR-PROJECT.supabase.co/rest/v1/rpc/keep_alive`
   - **Execution schedule:** every day, at any time you like
3. Open the **Advanced** tab and set:
   - **Request method:** `POST`
   - **Headers:** add two headers:
     - `apikey` = your publishable or anon key
     - `Content-Type` = `application/json`
   - **Request body:** `{}`
4. Turn on **Notify me on failure** (in the notifications settings).
5. Save, then use **Test run**. A response status of **200** with a timestamp in the body means it works.

---

## Good to know

- **It's a workaround, not a guarantee.** Supabase doesn't publish an exact activity threshold. Its docs say a few database requests a day over the past week is typically enough. Keep watching for Supabase's warning email, which arrives before any pause.
- **If the project pauses anyway,** restore it from the Dashboard (**Resume project**) within 90 days. Your data and settings are kept.
- **Security:** The function only returns the current time. The public key used here is the same one your questionnaire page already exposes, so nothing new is revealed. Your survey table still accepts inserts only.
- **To remove it later:** delete the workflow or cron job, then run:

  ```sql
  drop function if exists public.keep_alive();
  ```
