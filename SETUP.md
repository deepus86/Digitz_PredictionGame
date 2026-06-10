# World Cup 2026 Prediction Game — New Group Setup (Clone)

A clean copy of the family game for standing up a **new, independent group**.
It shares **no data** with any other instance — its own Supabase project, its own
deploy, its own GitHub repo.

**Ownership for this clone (as planned):**
- **Supabase project, Netlify site, GitHub repo** → your accounts
- **football-data token** → registered with the group's (friend's) email
- A **new, separate GitHub repository** (not the family one)

---

## 1. Database (Supabase)
1. https://supabase.com → **New project**.
2. **SQL Editor → New query** → paste the entire `supabase/schema.sql` → **Run**.
   (Creates tables, triggers, scoring, leaderboard, the `member_auth` PIN table +
   `verify_login()`, and the bonus lock/reveal columns.)
3. **Project Settings → API**: copy the **Project URL** + **publishable (anon) key**
   (for the app), and the **service key** (secret — for the sync only).

## 2. Members + PINs
In the SQL editor, add the members:
```sql
insert into members (name) values ('Name1'), ('Name2'), ('Name3');  -- all names, unique
```
Then load each member's PIN into `member_auth` (kids = 2-digit with a leading zero
e.g. `047`; adults = 3-digit):
```sql
insert into member_auth (member_id, pin)
select m.id, v.pin
from (values
  ('Name1','312'), ('Name2','047')   -- ('KidName','0XX')
) as v(name,pin)
join members m on m.name = v.name
on conflict (member_id) do update set pin = excluded.pin;
```
DM each person their PIN privately. **Tip:** ask Claude to generate the `members`
insert + a unique PIN list + ready-to-send DM messages (same as the family setup).

## 3. Auto-sync (football-data.org)
1. Register a **free token** at https://www.football-data.org/client/register
   using the **group's email**.
2. Push this project to a **new, separate GitHub repo** (your account).
3. Repo **Settings → Secrets and variables → Actions** → add three secrets with
   **this project's own values**:
   - `SUPABASE_URL`
   - `SUPABASE_SERVICE_KEY` (the secret key)
   - `FOOTBALL_DATA_TOKEN`
4. **Actions → "Sync World Cup results" → Run workflow** to load fixtures.
   Runs every 30 min after that.

*Prefer no GitHub?* Copy `.env.example` → `.env`, fill it in, run `node sync/sync.js`.

## 4. Configure the app
In `index.html` (top CONFIG block):
- `SUPABASE_URL` + `SUPABASE_ANON` → this project's URL + publishable key
- `REVEAL_MODE` → `'after_kickoff'` (or `'always'` / `'off'`)
- `CROWD_WINDOW_H` → `50`

## 5. Deploy
1. Make a clean bundle: copy `index.html` **and** `_headers` into a `deploy/` folder
   (`mkdir -p deploy && cp index.html _headers deploy/`).
2. Drag the **`deploy/` folder** onto https://app.netlify.com/drop.
   > ⚠️ Never deploy the whole project folder — it contains `.env`.
3. Share the public URL with the group.

## 6. Optional — bonus reveal timing
By default, bonus picks lock **and** reveal at tournament start. To hold the reveal
(e.g. to Round of 32) or reopen editing later, set the two dates:
```sql
-- Reveal everyone's bonus picks at Round of 32 kickoff
update settings set bonus_reveal_at = '2026-06-28T19:00:00+00' where id = 1;
```
Rule of thumb: keep `bonus_reveal_at` ≥ `bonus_lock_at` (no copying).

---

## Notes
- **Login = name + PIN.** Switching user also requires the target's PIN (blocks impersonation).
- This instance is **fully isolated** — use *this* project's keys everywhere; it can't affect any other group.
- For full architecture/feature reference, see the family project's `DOCS.md`.
- Local preview: `node server.js` → http://localhost:3456
