# Insta Hub — backend and deployment setup

## Current status
The site in this repository is a static frontend demo. The request form currently validates fields in the browser only; it does not save or transmit submissions. Sample creator data is hard-coded in `index.html`. No admin login, approval workflow, or Instagram data importer is implemented yet.

## Recommended architecture
- Static frontend: GitHub Pages or another static host.
- Database and authentication: Supabase.
- Server-side privileged operations: Supabase Edge Functions.
- Admin credentials: Supabase Auth (invite/create the admin account privately). Never place passwords, service-role keys, or private API tokens in HTML, JavaScript, or a public repository.

GitHub Pages alone cannot securely protect an admin password or run a private API.

## Suggested database
Create a `profiles` table with:
- `id` uuid primary key
- `display_name` text not null
- `username` text unique not null
- `instagram_url` text not null
- `avatar_url` text
- `posts_count` integer
- `following_count` integer
- `followers_count` integer
- `category` text
- `status` text not null default 'pending' (pending/approved/rejected)
- `created_at` timestamptz default now()

Enable Row Level Security. Allow public users to create pending requests only through a constrained insert policy or, preferably, an Edge Function with validation and rate limiting. Public users should only read approved profiles. Only authenticated admin users should approve, reject, edit, or delete profiles. Keep the Supabase service-role key exclusively in Edge Function secrets.

## Instagram data
Do not assume public profile scraping will work reliably. Instagram restricts automated access and profile fields may be unavailable. Prefer manual profile details or an official Meta/Instagram API integration where eligible and permitted. Always validate profile URLs server-side.

## Implementation checklist
1. Create a Supabase project and the profiles table.
2. Configure RLS policies for public approved-profile reads, pending submissions, and admin-only management.
3. Create an Edge Function for profile submissions with input validation, spam protection, and rate limiting.
4. Add Supabase Auth for a privately provisioned admin account; do not add public admin registration.
5. Build an admin-only dashboard that checks the authenticated user's authorization server-side.
6. Connect the frontend form to the Edge Function and load only approved profiles from the database.
7. Test unauthorized reads/writes and approval permissions before launch.

## Deployment
1. Push this repository to GitHub.
2. In GitHub, open **Settings → Pages**.
3. Choose **Deploy from a branch**, select `main` and `/(root)`, then save.
4. Configure Supabase separately and add only public project URL/anon key to the frontend when ready. The anon key is designed for browser use only when RLS is correctly configured; never expose service-role keys.
