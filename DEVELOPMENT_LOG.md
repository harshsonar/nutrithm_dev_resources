# Nutrithm — Development Context Log

> **Purpose:** This file serves as context for any coding assistant. Updated on every development prompt.
> **Master Prompt Reference:** Always refer to and follow [`MAIN_PROMPT.md`](file:///c:/Users/hsonar/Personal/Projects/nutrithm/MAIN_PROMPT.md) in parallel. It is the primary project specification and architectural blueprint that guides all development requirements, boundaries, and design principles. Any amendments approved during prompts (such as 7 body types instead of 4) are logged below.

---

## What is Nutrithm?

A mobile health/nutrition app (Android beta) that helps users understand their nutritional needs through body type + BMI-based recommendations, digestive-power curves, meal logging with AI-powered food recognition, and meal planning — all grounded in an Indian food database (INDB).

---

## Tech Stack

| Layer | Technology |
|---|---|
| Frontend | Flutter (Android only) |
| State management | Riverpod (manual providers, no codegen) |
| Navigation | GoRouter |
| Auth | Supabase Auth (email/password + Google Sign-In) |
| Database | Supabase PostgreSQL (single source of truth) |
| Backend | NestJS (single modular app, NOT microservices) |
| AI food recognition | Google Gemini Flash-Lite via `@google/genai` |
| Push notifications | Firebase Cloud Messaging |
| Notification scheduler | NestJS `@nestjs/schedule` |
| Data classes | Freezed + json_serializable |

---

## Architecture Principles

- Supabase is the primary backend; Flutter talks to Supabase directly for CRUD via `supabase_flutter` SDK
- NestJS handles ONLY: AI meal parsing, FCM device tokens, push notification scheduling
- Only 3 NestJS API endpoints: `POST /api/v1/meals/parse-text`, `PUT /api/v1/devices/current`, `DELETE /api/v1/devices/current`
- All NestJS endpoints authenticate via Supabase access token (user ID derived from token, never from request body)
- Food database is locally imported into Supabase from INDB dataset — NO runtime external API calls
- AI (Gemini) identifies foods; Supabase food DB provides nutritional values — they are independent
- Nutritional recommendations use relative levels (Low/Moderate/High), never exact calories/grams
- Business rules (nutrition rules, digestive curves) are stored as configurable templates in Supabase, not hardcoded in Flutter
- RLS enforced on all user tables; service-role key used only on NestJS server

---

## Project Structure (Basic)

```
nutrithm/
├── nutrithm_app/          # Flutter app (created via `flutter create`)
│   ├── lib/
│   ├── android/
│   ├── pubspec.yaml
│   └── ...
├── nutrithm_backend/      # NestJS app (created via `nest new`)
│   ├── src/
│   ├── package.json
│   └── ...
└── DEVELOPMENT_LOG.md     # This file
```

---

## Important Points

- `riverpod_annotation`, `riverpod_generator`, `riverpod_lint`, `custom_lint` were EXCLUDED due to version conflicts with `freezed_annotation ^3.1.0` + `riverpod 3.5.0`. Providers are written manually.
- Flutter SDK: `^3.13.7`
- NestJS uses Vitest (not Jest), oxlint (not eslint alone), TypeScript `^6.0.3`
- No `dio` — using `http` package for 3 NestJS calls
- No local DB (hive/sqflite) — Supabase is the only data store
- No separate service locator (get_it) — Riverpod handles DI
- Beta excludes: wearables, explore tab, recipes, photo/voice meal logging, streaks, gamification, subscriptions, exact nutrient quantities, advanced digestive-power learning, auto meal-plan generation

---

## Development Progress

### Prompt 1 — Dependency listing (STEP 1)
- Listed all NPM runtime (14), NPM dev (12), Flutter (18), Flutter dev (6) dependencies
- User confirmed and installed them
- Backend project named: `nutrithm_backend`

### Prompt 2 — Dependency fix
- `riverpod_lint` + `custom_lint` conflict resolved by removing them
- Removed `riverpod_annotation` (codegen approach dropped)
- Added `http`, `freezed`, `json_serializable`, `build_runner` to pubspec
- Flutter `pub get` succeeded

### Prompt 3 — Development log + STEP 2
- Created this `DEVELOPMENT_LOG.md` context file
- Delivered Step 2: full project structure, DB schema (17 tables, 11 enums, 1 view, 1 trigger), 3 API contracts, food import architecture, RLS policy matrix, env vars

### Prompt 4 — Step 2 approval + changes + STEP 3 launch
- User approved Step 2 with 4 changes:
  - Body types expanded to 7: `underweight, slim, moderately_muscular, muscular, skinny_fat, higher_fat, obese`
  - Height precision: `NUMERIC(4,1)`, Weight: `NUMERIC(5,2)`
  - Meal times (breakfast, lunch, dinner) made nullable in `daily_routines`
  - `is_active` on `foods` kept as soft-delete flag
- Cascading impacts: 7 digestive curve templates, 28 nutrition rule combos (7 body types × 4 BMI), 21 SVGs (7 types × 3 genders)

### Prompt 5 — Full STEP 3 Implementation Delivered
- Switched to model Gemini 3.8 Flash (High) after previous background parallel task hit resource quota.
- Implemented and verified all deliverables across Database, Backend, Mobile Frontend, Assets, and Documentation:
  1. **Supabase Database (14 migration files in `nutrithm_backend/supabase/migrations/`)**:
     - `001_extensions.sql` (uuid-ossp, pg_trgm)
     - `002_enums.sql` (11 enums, 7 body types)
     - `003_profiles.sql` (profiles with NUMERIC(4,1) height, NUMERIC(5,2) weight, allergies, health conditions, auto-create trigger on auth.users)
     - `004_daily_routines.sql` (nullable meal times)
     - `005_digestive_curves.sql`, `006_nutrition_rules.sql`
     - `007_food_tables.sql` (food_data_sources, foods with is_active soft-delete & trigram index, food_aliases, food_portions)
     - `008_meals.sql`, `009_meal_planning.sql`, `010_notifications.sql`
     - `011_bmi_view.sql` (Asian-Indian classification calculation view)
     - `012_rls_policies.sql` (RLS enabled on all 17 tables; owner-only user tables, read-only food/template tables, backend-only notification_jobs)
     - `013_triggers.sql` (set_current_timestamp_updated_at triggers)
     - `014_seed_templates.sql` (7 predefined digestive curves across 24h & 28 body_type x BMI nutrition rule combinations)
  2. **NestJS Backend (`nutrithm_backend/src/`)**:
     - Supabase module + service (service-role client + user-token client)
     - SupabaseAuthGuard + CurrentUser decorator + GlobalHttpExceptionFilter
     - AI module + GeminiService using `@google/genai` (does not invent nutritional metrics)
     - Food module + FoodService (trigram fuzzy matching, aliases, portion nutritional calculation)
     - Meal Recognition (`POST /api/v1/meals/parse-text`)
     - Devices module (`PUT /api/v1/devices/current`, `DELETE /api/v1/devices/current`)
     - Notifications module (Firebase Admin SDK sender + `@Cron` scheduler worker)
     - Tested: `tsc --noEmit` and `npm run build` both succeeded with 0 errors!
  3. **Food Database Import Utility (`nutrithm_backend/scripts/food-import/`)**:
     - `source/.gitkeep`, `transform/parser.ts` (CSV & XLSX), `transform/normalizer.ts`, `import.ts`, `README.md`
     - Idempotent upserting into Supabase `foods`, `food_aliases`, `food_portions`
  4. **Flutter Mobile Application (`nutrithm_app/lib/`)**:
     - App theme (Material 3, custom nutrient colors, typography)
     - GoRouter declarative routing with ShellRoute bottom navigation & auth redirect
     - Core: `env.dart`, `supabase_provider.dart`, `api_client.dart` (calls NestJS endpoints), `notification_service.dart`, `bmi_calculator.dart`
     - Shared widgets: `loading_widget.dart`, `error_widget.dart`, `empty_state_widget.dart`, `nutrithm_button.dart`, `nutrithm_text_field.dart`, `time_picker_field.dart`, `section_header.dart`
     - 8 Feature Modules:
       - `auth`: Login, Register, Google Sign-In, Supabase Auth session stream
       - `onboarding`: 5-step PageView (About You, 7 Body Types with SVG cards, BMI Summary with Asian-Indian reference, Food Preferences with private health indicators, Daily Routine)
       - `home`: Greeting, current digestive status card, fl_chart 24h digestive curve with current-time indicator line, circular nutrient emphasis visualization (P/C/F/Fib relative levels), active/upcoming meal window card, Log Meal action
       - `meal_logging`: Free-text input, Gemini extraction, food DB matching, portion corrections, unmatched food handling (no invented values), snapshot saving
       - `food_catalog`: Real-time food DB search and portion preview
       - `meal_planning`: Weekly TableCalendar, day-grouped planned meals, custom/catalog meal additions, prep tasks with notification reminders
       - `insights`: Clean placeholder preview cards following prototype direction
       - `profile`: Profile overview, edit personal info, edit body type, edit diet/allergens, edit private health conditions, edit routine, edit FCM meal reminders, sign out
  5. **Assets & Documentation**:
     - 21 SVG body type silhouettes generated (7 body types × 3 gender variants: man, woman, generic) in `assets/body_types/`
     - `assets/.env` and `assets/.env.example` in `nutrithm_app`
     - `.env.example` in `nutrithm_backend`
     - `SUPABASE_SETUP.md` at root: 14 migration execution steps, OAuth & deep linking guide
     - `FOOD_DATABASE_IMPORT.md` at root: INDB download, parsing, portion conversions, verification queries


### Prompt 6 — Schema Review & RLS Deep Dive
- Clarified SQL syntax: `DO $$ BEGIN ... END $$`, dollar quoting, SQL trigger functions vs. NestJS services, and `on_auth_user_created` trigger mechanics.
- Updated `003_profiles.sql`: made `profile_allergies.allergy` and `profile_health_conditions.condition` nullable columns to avoid false assumptions.
- Verified and updated `009_meal_planning.sql`: removed redundant `idx_meal_plans_profile_date` index since PostgreSQL automatically provisions a unique B-tree index for `UNIQUE(profile_id, plan_date)`.
- Explained `012_rls_policies.sql`: Row-Level Security mechanics, query rewriting, `USING` vs `WITH CHECK`, and role scoping.


### Prompt 7 — Prompt Comparison & Alignment Audit
- Master prompt reference explicitly linked to `MAIN_PROMPT.md` at top of log.
- Compared `MAIN_PROMPT.md` with current codebase state: identified that initial implementation code for STEP 3 (Flutter, NestJS, Supabase SQL migrations, food import scripts, setup guides) has been delivered; current stage is SQL schema refinement, verification, and preparation for deployment/live testing.
- Flagged differences between original `MAIN_PROMPT.md` and approved evolutions (4 -> 7 body types, nullable routine meal times, removal of duplicate unique indexes, closing instruction pointer).

- Updated `MAIN_PROMPT.md`: synchronized body types (7 types), 21 SVGs, nullable routine meal times, 7 digestive curves, 28 nutrition rule templates, and updated closing project status line.
