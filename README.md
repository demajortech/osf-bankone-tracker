# OSF BankOne Migration Tracker — setup guide

## What this starter includes
- Mobile-friendly login page and shared migration dashboard
- Checklist tasks from 16 October through 30 November 2026
- Search and phase/status filters, progress totals and task evidence links
- Sign-in with any provisioned OSF company email, including `@osfng.org`
- Database-level rule: signed-in staff can view tasks; only the assigned owner or project admin can update a task's status/evidence/note
- Task change history trigger

## Important security notes
- This is a starter project, not a certified banking system or replacement for BankOne.
- Do not store customer names, account numbers, BVN/NIN, KYC files, passwords, balances or transaction-level records in this tracker. Store task evidence in OSF-approved restricted storage and link to it only for authorized reviewers.
- GitHub Pages is static hosting. Supabase provides authentication, database and authorization. A website URL alone does not grant access.
- Never put a Supabase `service_role` or secret key in `app.js`. Use only the project's publishable/anon key with the SQL security policies enabled.
- Each task has one `owner_email` used for permission enforcement. If several departments contribute, nominate one accountable person and list other teams in `owner_label`.
- Seed tasks initially use `unassigned@osfng.org`; replace every one with the real accountable owner's email before use.
- Ask OSF management to approve use of third-party hosting/database services and confirm data protection, retention, backups and account ownership before production use.

## A. Create the backend
1. Visit https://supabase.com and create a project under an account controlled by an authorized OSF administrator.
2. Open the project's SQL Editor.
3. Open `supabase_setup.sql` in a text editor. Review the task list and dates with the Coordinator and Finance.
4. Run the SQL in Supabase. Confirm the `profiles`, `tasks`, and `task_activity` tables exist.
5. In Authentication settings, disable public sign-ups if available. Staff should be provisioned by the authorized admin, not allowed to self-register.
6. In project API/Connect settings, copy the Project URL and publishable key (some projects label it `anon`). Never copy the `service_role`/secret key.

## B. Assign the tasks to actual staff
1. In Supabase Table Editor, open `tasks`.
2. For each task, change `owner_email` from `unassigned@osfng.org` to the exact company email of the one accountable owner, e.g. `e.prince@osfng.org`.
3. Adjust `owner_label` to show the owner's name/role and any contributing teams.
4. Verify all task owners with the Coordinator before staff access the website. Until assigned, the task cannot be updated by ordinary staff.

## C. Create user accounts and profiles
1. In Supabase Authentication > Users, add/invite each staff member using their real company email.
2. Use the approved password/invite process. For reliable invitation/reset emails, configure an authorized SMTP sender. Do not assume the company's email can receive Supabase messages until tested.
3. Copy each user's UUID from Authentication > Users.
4. In SQL Editor, insert one profile row per staff member. Example (replace the UUID and details):

```sql
insert into public.profiles (user_id,email,display_name,role)
values ('PASTE-USER-UUID-HERE','e.prince@osfng.org','ICT Tracker Admin','admin');
```

For all other staff use `role='staff'`. Only one or a small number of approved administrators should have the `admin` role. Do not let users update their own profile role.

## D. Connect the front end
1. Open `app.js` in a text editor.
2. Replace `PASTE_SUPABASE_PROJECT_URL_HERE` with the Project URL.
3. Replace `PASTE_SUPABASE_PUBLISHABLE_OR_ANON_KEY_HERE` with the publishable/anon key.
4. Save the file. These are the only configuration values needed in the frontend.

## E. Publish the website on GitHub Pages
1. Create a GitHub repository approved by OSF. Public repositories expose their HTML/CSS/JS source, so do not commit secrets or customer data.
2. Create a repository named `osf-bankone-tracker`.
3. Upload `index.html`, `styles.css`, and `app.js`. Keep `supabase_setup.sql` and staff assignment details in an access-controlled location; only authorized admins should have them.
4. In the repository, open Settings > Pages.
5. Under Build and deployment, choose deploy from a branch, select `main` and `/(root)`, then Save.
6. Wait for GitHub Pages to display the published site URL and open it.
7. In Supabase Authentication URL settings, set the Site URL to the published URL and add it to allowed redirect URLs if the project requires it.
8. Test login and task updates with at least two separate staff accounts.

## F. Test permissions before launch
- Signed-out visitors cannot read the task list.
- Staff can view all tasks after login.
- A staff member can update only tasks assigned to their exact email.
- A different staff member cannot update that task.
- An admin can update all tasks.
- Status changes show in the dashboard after reload.
- Public sign-up is disabled and only approved accounts exist.
- No customer-level confidential information is stored in the tracker.
- Invite/password-reset email delivery, data export, backup and account recovery have been tested.
- Ask a technically competent independent reviewer to check the Supabase row-level security policies before relying on the app for project governance.

## How to use it with WhatsApp
After publishing, copy the website URL, post it in the dedicated OSF migration WhatsApp group and pin the message. Staff open the URL and sign in using their provisioned company email and password. The link itself does not grant access.

## Current deliberate limitations
- Task creation and reassignment are done by the authorized database administrator; the web UI only updates status, evidence link and note.
- The accountable owner email is the access-control key. Keep it accurate.
- The target 30 November go-live is conditional on successful migration, UAT, BankOne confirmation and formal OSF approval.
