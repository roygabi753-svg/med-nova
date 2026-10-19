# MED NOVA — Allied Health Sciences Learning Library

A Next.js + TypeScript + Tailwind + Supabase starter for an Indian allied-health learning platform. This repository contains a polished responsive public UI, department and starter subject directory, Supabase email/password registration/login, protected account/admin routes, an initial PostgreSQL schema, and baseline RLS policies.

## Important status disclosure

This is a **starter implementation**, not a fully production-ready or independently security-audited platform. The public taxonomy and sample subject maps are illustrative; they are not a complete, regulator-approved syllabus. Real analytics beyond live profile/material counts, content authoring CRUD, password-reset UI, MFA enrollment, upload malware scanning, rate limiting, consent UI, full audit event coverage, and automated end-to-end security tests remain to be implemented and verified before launch. No external Supabase project or deployment has been configured by this project generation.

## Requirements

- Node.js 20.9+ and npm
- Supabase project (Auth + PostgreSQL)
- Git

## Local setup

1. Copy `.env.example` to `.env.local`.
2. Fill `NEXT_PUBLIC_SUPABASE_URL`, `NEXT_PUBLIC_SUPABASE_ANON_KEY`, and `NEXT_PUBLIC_SITE_URL` from your Supabase project. Never put a service-role key in browser code or commit secrets.
3. Install dependencies: `npm install`
4. Apply `supabase/migrations/0001_med_nova.sql` using the Supabase CLI (`supabase link`, then `supabase db push`) or the SQL editor in a disposable development project after reviewing it.
5. In Supabase Auth settings, enable email confirmations and configure the Site URL and redirect URL `http://localhost:3000/auth/callback`.
6. Start the app: `npm run dev`
7. Open `http://localhost:3000`.

If you use the Supabase CLI, install it separately and authenticate through its supported secure login flow. Do not paste secrets into chat or source files.

## First Super Admin — secure bootstrap

New accounts are always created with `student` role. Never allow role assignment from browser metadata. After registering and verifying the first account, identify that account's UUID in Supabase Auth. With project-owner access, run the following **once** in the Supabase SQL editor, replacing the UUID:
```sql
update public.profiles set role='super_admin' where id='YOUR-VERIFIED-USER-UUID';
insert into public.security_audit_events(actor_id,action,target_type,target_id,details)
values ('YOUR-VERIFIED-USER-UUID','initial_super_admin_bootstrap','profile','YOUR-VERIFIED-USER-UUID','{"method":"owner-reviewed SQL bootstrap"}'::jsonb);
```
Run only in the trusted Supabase dashboard, verify the target UUID and immediately remove any temporary credentials or access used for bootstrap. Do not expose a public endpoint for initial role assignment. For stronger auditability, document the owner, time, approval and change ticket in your operational records.

## Content hierarchy and editorial workflow

Target hierarchy: Department → Degree/Diploma → Year → Semester → Subject → Unit → Topic → Learning Resources. The SQL schema provides courses, subjects, units, topics and learning materials. Add content editor assignment UI/API and review workflow before using it as a CMS. Publish only original, open-license, or explicitly permitted content. Track the license/permission evidence and clinical reviewer for materials and diagrams.

## Deployment outline

1. Push this repository to a private/public Git repository as appropriate.
2. Import to Vercel (or another maintained Next.js host).
3. Add production environment variables in the host's encrypted project settings; do not commit `.env.local`.
4. Set `NEXT_PUBLIC_SITE_URL` to the HTTPS domain; add the domain and `/auth/callback` to Supabase Auth redirect allowlist.
5. Configure a custom domain and verify HTTPS before switching DNS traffic.
6. Apply migrations to production only after review and a tested staging run.
7. Configure database backups/PITR as available on your Supabase plan, test restore, add uptime/error monitoring, and set retention policies.
8. Enable dependency alerts and run `npm audit`, build checks and a separate security review before launch.

## Security notes / remaining work

- RLS is enabled on application tables and most policies are defined in the migration; test policies with student/editor/admin/super-admin/anonymous identities in a non-production project before launch.
- Role checks use database profiles; user-supplied metadata does not grant privileges. The `assign_role` RPC is Super Admin-only. Initial Super Admin setup is an explicit owner action.
- A production release still needs tested rate limiting (including login, search and forms), CSRF/origin strategy for mutations, password-reset route and email templates, optional administrator MFA enforcement, session revocation policy, safe upload flow with MIME sniffing/size caps and malware scanning, CSP tuning with nonces/hashes, analytics consent capture, retention/deletion jobs, complete audit logging, accessibility checks, penetration testing and recovery drills.
- Review the RLS migration carefully before production. In particular, build a proper content-review workflow, validate that question answer keys are never exposed to students before submission, and make role/permission changes through a server-verified administrative workflow.
- Avoid collecting patient information. Educational content must not be used in place of medical advice or supervised clinical training.

## Commands

- `npm run dev` — local development
- `npm run build` — production build
- `npm run test` — Vitest tests (add tests under `src/**/*.test.ts`)

## License

No license is granted for third-party educational materials by this scaffold. Review and document rights for every imported resource.
