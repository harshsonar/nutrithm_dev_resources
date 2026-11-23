# Supabase Setup Guide for Nutrithm

This guide walks you through setting up Supabase PostgreSQL and Supabase Auth step-by-step for the **Nutrithm** application.

---

## 1. Create a Supabase Project

1. Log into your [Supabase Dashboard](https://supabase.com/dashboard).
2. Click **New project**.
3. Select your organization.
4. Set **Project Name**: `nutrithm` (or your preferred name).
5. Set a secure **Database Password** (store this in your password manager).
6. Choose the **Region** closest to your target audience (e.g. `ap-south-1` Mumbai for Indian users).
7. Choose the **Free** plan (or Pro) and click **Create new project**.
8. Wait 1–2 minutes for the database to finish provisioning.

---

## 2. Enable Extensions and Run Migrations

1. Go to **SQL Editor** in the left sidebar.
2. Click **New query**.
3. Run each migration file in sequence from `nutrithm_backend/supabase/migrations/`:

| Order | File | Description |
|---|---|---|
| 1 | `001_extensions.sql` | Enables `uuid-ossp` and `pg_trgm` |
| 2 | `002_enums.sql` | Creates 11 custom ENUM types |
| 3 | `003_profiles.sql` | Creates `profiles`, `profile_allergies`, `profile_health_conditions`, and auth trigger |
| 4 | `004_daily_routines.sql` | Creates `daily_routines` table |
| 5 | `005_digestive_curves.sql` | Creates `digestive_curve_templates` |
| 6 | `006_nutrition_rules.sql` | Creates `nutrition_rule_templates` |
| 7 | `007_food_tables.sql` | Creates `food_data_sources`, `foods`, `food_aliases`, `food_portions` |
| 8 | `008_meals.sql` | Creates `meals` and `meal_items` |
| 9 | `009_meal_planning.sql` | Creates `meal_plans`, `meal_plan_items`, `meal_prep_tasks` |
| 10 | `010_notifications.sql` | Creates `meal_reminder_settings`, `user_devices`, `notification_jobs` |
| 11 | `011_bmi_view.sql` | Creates `user_bmi` calculation view |
| 12 | `012_rls_policies.sql` | Enables RLS on all tables and creates policies |
| 13 | `013_triggers.sql` | Adds automatic `updated_at` triggers |
| 14 | `014_seed_templates.sql` | Seeds 7 digestive curves and 28 nutrition rule combinations |

Alternatively, you can copy all 14 files in numerical order and run them in one transaction.

---

## 3. Configure Authentication Providers

### A. Email / Password
1. Navigate to **Authentication** → **Providers** in the left sidebar.
2. Under **Email**, ensure **Enable Email provider** is toggled ON.
3. Toggle **Confirm email** OFF for development/beta if you want immediate signup without requiring email verification, or ON for production.

### B. Google OAuth
1. In the Supabase Dashboard, go to **Authentication** → **Providers** → **Google**.
2. Toggle Google ON.
3. You will see a **Callback URL (for OAuth)** in the format:
   ```
   https://<your-project-ref>.supabase.co/auth/v1/callback
   ```
4. Copy this Callback URL.
5. Go to [Google Cloud Console](https://console.cloud.google.com/):
   - Select your project.
   - Go to **APIs & Services** → **Credentials**.
   - Click **Create Credentials** → **OAuth client ID**.
   - Application type: **Web application**.
   - Name: `Nutrithm Supabase Auth`.
   - Under **Authorized redirect URIs**, paste the Supabase Callback URL copied in step 4.
   - Click **Create**.
   - Copy the **Client ID** and **Client Secret**.
6. Back in Supabase Dashboard → **Google Provider**, paste the **Client ID** and **Client Secret**.
7. Click **Save**.

---

## 4. Configure Android Deep Linking & Redirect URLs

1. In Supabase Dashboard, go to **Authentication** → **URL Configuration**.
2. Set **Site URL** to:
   ```
   com.nutrithm.app://login-callback
   ```
3. Under **Redirect URLs**, click **Add URL** and add:
   ```
   com.nutrithm.app://login-callback
   com.nutrithm.app/**
   ```
4. Click **Save**.

---

## 5. Retrieve API Credentials

1. Go to **Project Settings** (gear icon) in the bottom-left sidebar.
2. Click **API**.
3. Under **Project URL**, copy the URL:
   - Example: `https://xyzcompany.supabase.co`
4. Under **Project API keys**:
   - Copy the `anon` `public` key (for Flutter and NestJS public verification).
   - Copy the `service_role` `secret` key (for NestJS backend only — **NEVER** expose in Flutter).

---

## 6. Verification Checklist

Run these queries in the Supabase **SQL Editor** to verify:

```sql
-- 1. Verify extensions
SELECT * FROM pg_extension WHERE extname IN ('uuid-ossp', 'pg_trgm');

-- 2. Verify tables created (should return 17 tables)
SELECT table_name FROM information_schema.tables 
WHERE table_schema = 'public' AND table_type = 'BASE TABLE'
ORDER BY table_name;

-- 3. Verify seed data
SELECT count(*) FROM digestive_curve_templates; -- Expected: 7
SELECT count(*) FROM nutrition_rule_templates;   -- Expected: 28

-- 4. Verify RLS is enabled on all public tables
SELECT tablename, rowsecurity FROM pg_tables WHERE schemaname = 'public';
```
