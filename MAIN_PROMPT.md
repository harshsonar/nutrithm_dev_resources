# Nutrithm --- Complete Android Beta Application

Act as a senior Flutter, NestJS and Supabase developer. Help me build
the beta version of a mobile application called **Nutrithm**.

I want a clean, maintainable and functional application. Keep the
implementation lean. Do not introduce unnecessary architecture,
abstractions or dependencies.

## 1. Technology stack

  Layer                    Technology
  ------------------------ ------------------------------------
  Frontend                 Flutter (Android only)
  State management         Riverpod
  Navigation               GoRouter
  Authentication           Supabase Auth
  Database                 Supabase PostgreSQL
  Backend                  NestJS
  AI food recognition      Google Gemini Flash-Lite
  Push notifications       Firebase Cloud Messaging
  Notification scheduler   NestJS Schedule
  Database access          Supabase Flutter SDK / Supabase JS

Supabase must remain the primary backend and the single source of truth
for application data. Use one modular NestJS application. Do not create
microservices.

------------------------------------------------------------------------

# 2. VERY IMPORTANT: Development workflow

Follow these steps strictly.

## STEP 1 --- Dependencies

First, provide a complete list of: 1. NPM runtime dependencies. 2. NPM
development dependencies. 3. Flutter packages.

Explain briefly what each package is used for.

**Do not install anything. I will install the dependencies myself.**

After listing dependencies, STOP and wait for my confirmation.

## STEP 2 --- Project structure and architecture review

After I confirm that dependencies are installed, provide: 1. Complete
Flutter project folder structure. 2. Complete NestJS project folder
structure. 3. Complete Supabase database schema. 4. Database
relationships and foreign keys. 5. Three initial NestJS API contracts.
6. Food database import architecture. 7. Supabase setup instructions. 8.
Firebase and Google OAuth configuration instructions. 9. Environment
variables for all components.

Show the ENTIRE frontend and backend structure before generating
implementation code. Do not generate code at this stage. Wait for my
approval.

## STEP 3 --- Implementation

Only after I approve the proposed structures: - Generate the Flutter
application. - Generate the NestJS backend. - Generate Supabase SQL
migrations. - Generate the INDB food import script. - Provide exact
setup instructions.

Do not run tests or production builds. Skip test implementation to save
tokens. Do not install dependencies yourself.

------------------------------------------------------------------------

# 3. Product requirements

## A. Authentication

Use Supabase Auth.

Support: - Email/password registration. - Email/password login. - Google
sign-in. - Logout. - Persistent authentication sessions.

Only Android is supported in beta. Do not implement custom
authentication.

## B. Onboarding

The onboarding process has three sections.

### 1. About you

Collect: - First name. - Age (18+ only). - Gender. - Height. - Weight. -
Activity level.

### 2. Body type

Allow users to select one of 7 body types:
- Underweight (`underweight`)
- Slim (`slim`)
- Moderately muscular (`moderately_muscular`)
- Muscular (`muscular`)
- Skinny fat (`skinny_fat`)
- Higher fat (`higher_fat`)
- Obese (`obese`)

Display body illustrations to help users understand the options.

Create separate illustrations for: - Men. - Women. - Generic human
silhouettes (total 21 SVG silhouettes: 7 body types × 3 gender variants).

If the user selects non-binary, another identity or prefers not to
disclose gender, display the generic illustration.

Use simple, consistent and non-judgmental SVG illustrations.

### 3. BMI calculation

Calculate BMI automatically:

BMI = weight in kilograms / (height in metres × height in metres).

Use Asian-Indian classification:

  BMI            Classification
  -------------- ----------------
  Below 18.5     Underweight
  18.5--22.9     Normal
  23--24.9       Overweight
  25 and above   Obesity

Do not automatically infer body type from BMI. Body type must be
independently selected by the user.

### 4. Food preferences

Support: - Vegetarian. - Eggetarian. - Non-vegetarian. - Vegan. - Jain.

Allow multiple allergy selections.

Allow optional health-condition information, including: - Prediabetes. -
Type 2 diabetes. - High BP. - PCOS. - Thyroid. - High cholesterol.

Store health-condition information privately. Do not use it in the
nutritional recommendation engine in beta.

### 5. Daily routine

Collect: - Wake-up time. - Sleep time. - Breakfast time (nullable/optional). - Lunch time (nullable/optional). -
Dinner time (nullable/optional). - Workout time (optional).
(Note: meal times are nullable in `daily_routines` because some users skip meals).

Store the user's timezone.

------------------------------------------------------------------------

# 4. Nutritional recommendation engine

Use the combination of BMI category and selected body type to determine
relative nutritional emphasis.

Nutrients: - Protein. - Carbohydrates. - Fat. - Fibre.

Do not display exact calories, grams or nutrient quantities. Use
relative levels: Low, Moderate and High.

Display them using a circular nutrient visualisation, following the
Nutrithm prototype.

Create configurable nutritional rule templates in Supabase.

Use the following initial recommendation principles:

  -----------------------------------------------------------------------
  Body type                           Initial nutritional emphasis
  ----------------------------------- -----------------------------------
  Underweight                         Calorie/protein supportive, nutrient dense

  Slim                                Balanced nutrients, relatively
                                      higher protein

  Moderately muscular                 Protein-supportive, balanced
                                      carbohydrates and fats

  Muscular                            High protein support, balanced energy

  Skinny fat (low muscle / abd fat)   Protein-supportive, balanced portions

  Higher body fat                     Balanced portions, fibre emphasis

  Obese                               Fibre emphasis, moderate balanced portions
  -----------------------------------------------------------------------

Combinations: 28 configurable nutrition rule templates in Supabase (7 body types × 4 Asian-Indian BMI categories: Underweight, Normal, Overweight, Obese). BMI category must also be represented in the configurable rules. Treat
these as initial beta heuristics, not medical prescriptions. Do not
hardcode recommendation logic throughout Flutter widgets.

------------------------------------------------------------------------

# 5. Digestive-power engine

For beta, use seven predefined digestive-power curve templates, one for
each body type (underweight, slim, moderately_muscular, muscular, skinny_fat, higher_fat, obese).

Requirements: - Each curve represents digestive power throughout the
day. - Digestive power generally peaks around the afternoon. - Display
Peak, Moderate and Low periods. - Display a current-time indicator. -
Determine suggested meal windows. - Use the user's daily routine when
determining initial meal reminder timings.

Store the seven predefined templates in Supabase PostgreSQL (`014_seed_templates.sql`). Use JSONB
for the curve's time/value points and suggested meal windows.

Do not implement machine learning, wearable integration or individual
behavioural adaptation. Keep the initial curves simple and configurable.

------------------------------------------------------------------------

# 6. Flutter Home screen

Display: - User greeting. - Current digestive-power status. - Daily
digestive-power curve. - Current time indicator. - Nutritional circle
with relative protein, carbs, fat and fibre emphasis. - Current
meal-window recommendation. - Log Meal action.

Follow the supplied Nutrithm prototype's visual direction.

------------------------------------------------------------------------

# 7. Meal logging

For beta, support TEXT INPUT ONLY.

Example: "2 rotis, palak dal, curd at 1:30 PM"

Use NestJS to send the text to Gemini.

Gemini should extract: - Food names. - Quantities. - Household
measurements. - Meal time, when provided. - Meal category, when
identifiable.

NestJS must search the local Supabase food database for matching food
items.

Important rules: 1. Gemini must never invent nutritional information. 2.
Nutritional values must come exclusively from our stored food database.
3. Users must review and confirm recognised food items and portions
before saving. 4. Preserve the original text. 5. Save nutritional
snapshots with logged meals. 6. Handle unmatched food items without
inventing nutritional values. 7. Allow users to correct food matches and
quantities.

Do not implement photographs, videos or voice recognition.

------------------------------------------------------------------------

# 8. Food database --- IMPORTANT ARCHITECTURAL REQUIREMENT

Use the **Indian Nutrient Databank (INDB)** as our initial food database
source.

Website: https://www.anuvaad.org.in/indian-nutrient-databank/

## Local database import is mandatory

We are NOT going to query the INDB website or an external INDB API
whenever a user searches for food or logs a meal.

Instead:

1.  Obtain/download the INDB dataset.
2.  Import the relevant food data into our own Supabase PostgreSQL
    database.
3.  Store the nutritional information in our `foods` table.
4.  Store food aliases and alternative names.
5.  Store standard household portion conversions.
6.  Use Supabase PostgreSQL as the source of truth for all runtime food
    searches and nutritional calculations.

The application must not depend on the INDB website being available
during normal operation.

## Food import script

Create a reusable import utility in the NestJS project:

``` text
backend/
├── scripts/
│   └── food-import/
│       ├── source/
│       ├── transform/
│       ├── import.ts
│       └── README.md
```

Responsibilities: - Read the downloaded INDB dataset. - Parse and
normalise the source data. - Map the relevant fields into our PostgreSQL
schema. - Preserve original INDB food identifiers wherever available. -
Preserve source and dataset-version information. - Import nutritional
values. - Import serving information where available. - Upsert records
without creating duplicates. - Support re-running the import when the
source dataset is updated.

The import should be idempotent. Do not invent food records or
nutritional values.

**Important:** If the actual INDB dataset file is not available in the
development workspace, do not fabricate an import or pretend it has been
completed. Instead: - Create the import utility structure. - Define the
expected source file format. - Explain where I should place the
downloaded dataset. - Document how I can execute the import after
providing the file. - Make the parser easy to adjust once the actual
dataset structure is inspected.

Do not make external INDB API calls from Flutter or NestJS during
runtime.

## Database structure for food data

Create the following tables:

### `food_data_sources`

Stores: - Source name. - Source URL. - Dataset version. - Attribution. -
Last imported timestamp.

### `foods`

Stores: - Food ID. - Source ID. - Original source food ID, where
available. - Food name. - Category. - Calories. - Protein. -
Carbohydrates. - Fat. - Fibre.

Store nutritional values on a consistent per-100g basis wherever the
source supports it.

### `food_aliases`

Stores: - Food ID. - Alternative food names. - Common spellings. -
Language, where available.

### `food_portions`

Stores: - Food ID. - Portion name. - Grams per unit. - Default portion
indicator. - Conversion verification status.

## Household measurements

Prioritise: - Katori. - Bowl. - Glass. - Roti. - Piece. - Tablespoon. -
Teaspoon.

Users should be able to enter quantities such as: - 2 katoris. - 1
glass. - 2 rotis. - 1.5 bowls.

Each food must have its OWN standard portion conversion. Do not assume
one katori or bowl always weighs the same across different foods.

Use available source serving information where possible. Do not
fabricate conversion factors. Flag missing or unverified conversions for
manual review.

The database must be independent of the AI provider.

Gemini identifies what the user ate. Our Supabase food database
determines its nutritional values.

------------------------------------------------------------------------

# 9. Meal planning

Users create their own meal plans. Do not automatically generate or
curate meal plans.

Support: - Weekly calendar. - Selecting dates. - Adding planned meals. -
Searching the local food database. - Adding custom meal names. -
Selecting portions for known foods. - Setting planned meal times. -
Editing and deleting planned meals. - Adding meal-preparation tasks. -
Setting preparation reminder times.

Example:

Tuesday: - Breakfast: Poha + curd, 8:00 AM. - Preparation task: Soak
peanuts, Monday at 9:30 PM.

Planned meals and preparation tasks must be stored in Supabase.

------------------------------------------------------------------------

# 10. Push notifications

Use Firebase Cloud Messaging.

Flutter packages: - `firebase_core` - `firebase_messaging` -
`flutter_local_notifications`

NestJS packages: - `firebase-admin` - `@nestjs/schedule`

Support two notification types.

### Meal reminders

-   Generate default reminder times based on the user's body type,
    digestive curve and daily routine.
-   Allow users to enable or disable individual meal reminders.
-   Allow users to modify reminder times.
-   Respect the user's local timezone.

### Meal-preparation reminders

-   Trigger based on preparation tasks in the meal plan.
-   Allow users to set preparation reminder times.

Store device tokens and notification jobs in Supabase.

Use a lightweight notification queue.

NestJS should run a scheduled worker to process due notifications and
send them through FCM. Prevent duplicate notifications.

The NestJS application must run as an always-on process so scheduled
notifications are not missed.

------------------------------------------------------------------------

# 11. Insights

Include the Insights tab, following the prototype's visual direction.

Detailed metrics and calculations are TBD.

Do not invent metrics, fake progress or dummy statistics. Implement a
clean placeholder structure that can be extended later.

------------------------------------------------------------------------

# 12. Profile

Allow users to view and edit: - Personal information. - Body type. -
Dietary preference. - Allergies. - Health-condition information. - Daily
routine. - Reminder preferences. - Account settings.

------------------------------------------------------------------------

# 13. Excluded functionality

Do NOT implement: - Connect Wearables screen. - Wearable integrations or
associated data. - Explore tab. - Recipe discovery. - Swiggy/Zomato
integrations. - Photo/video meal recognition. - Voice recognition. -
Streaks. - Gamification and progression levels. - Subscription/payment
functionality. - Report generation or sharing. - Exact nutrient quantity
display. - Advanced digestive-power learning. - Automatic meal-plan
generation. - Medical recommendations.

Keep the implementation strictly within the beta scope.

------------------------------------------------------------------------

# 14. NestJS backend

Build one modular NestJS application.

Suggested modules:

``` text
backend/
├── src/
│   ├── main.ts
│   ├── app.module.ts
│   │
│   ├── common/
│   │   ├── guards/
│   │   ├── decorators/
│   │   └── filters/
│   │
│   ├── supabase/
│   ├── ai/
│   ├── food/
│   ├── meal-recognition/
│   ├── devices/
│   └── notifications/
│
├── scripts/
│   └── food-import/
│
├── supabase/
│
├── .env.example
└── package.json
```

Keep the AI provider replaceable through a simple interface.

Use Google's official `@google/genai` SDK. Configure the provider and
model through environment variables. Do not hardcode API keys or model
names in application business logic.

Do not send health-condition information, personal profile details or
other unnecessary personal data to Gemini. Only send the meal
description and the minimum context needed for food recognition.

Use synthetic meal data when developing against a free AI tier if its
data-use policy permits provider training or product improvement.

## Initial API endpoints --- ONLY THREE

Implement these three endpoints initially:

  ----------------------------------------------------------------------------
  Method                  Endpoint                     Purpose
  ----------------------- ---------------------------- -----------------------
  POST                    `/api/v1/meals/parse-text`   Extract meal
                                                       information and find
                                                       matching food records

  PUT                     `/api/v1/devices/current`    Register/update current
                                                       user's FCM device token

  DELETE                  `/api/v1/devices/current`    Remove current user's
                                                       device token
  ----------------------------------------------------------------------------

All endpoints must authenticate using a valid Supabase access token.

Never trust a user ID supplied in the request body. Derive the
authenticated user from the verified token.

Ordinary CRUD operations should use the Flutter Supabase SDK directly.
Do not create unnecessary CRUD APIs in NestJS.

------------------------------------------------------------------------

# 15. Supabase database schema

Create SQL migrations for the following tables:

### User and onboarding

-   `profiles`
-   `profile_allergies`
-   `profile_health_conditions`
-   `daily_routines`

### Nutritional and digestive configuration

-   `digestive_curve_templates`
-   `nutrition_rule_templates`

### Food database

-   `food_data_sources`
-   `foods`
-   `food_aliases`
-   `food_portions`

### Meal logging

-   `meals`
-   `meal_items`

### Meal planning

-   `meal_plans`
-   `meal_plan_items`
-   `meal_prep_tasks`

### Notifications

-   `meal_reminder_settings`
-   `user_devices`
-   `notification_jobs`

Implement appropriate: - Foreign keys. - Unique constraints. - Check
constraints. - Indexes. - Timestamps. - RLS policies.

Create a PostgreSQL view that calculates BMI and Asian-Indian BMI
classification from the user's profile.

Create appropriate access policies: - User-specific tables: owner-only
access using `auth.uid()`. - Food tables: authenticated users may read;
ordinary users cannot modify. - Digestive curves and nutritional rules:
authenticated users may read. - Health conditions: only the owner can
access. - Notification jobs: backend-only access. - Food source/import
metadata: backend-managed.

Create a profile trigger after Supabase user registration. Allow
onboarding information to remain incomplete until the user finishes
onboarding.

Use Supabase service-role credentials only on the NestJS server, never
in Flutter.

For normal authenticated user operations, respect RLS. Do not
unnecessarily bypass RLS with the service-role key.

Use `pg_trgm` for efficient fuzzy food-name matching where appropriate.

------------------------------------------------------------------------

# 16. Flutter application structure

Use feature-first architecture.

Suggested structure:

``` text
mobile/
├── android/
├── assets/
│   ├── body_types/
│   │   ├── man/
│   │   ├── woman/
│   │   └── generic/
│   └── fonts/
│
└── lib/
    ├── main.dart
    │
    ├── app/
    │   ├── app.dart
    │   ├── router.dart
    │   └── theme/
    │
    ├── core/
    │   ├── config/
    │   ├── supabase/
    │   ├── api/
    │   ├── notifications/
    │   ├── errors/
    │   └── utils/
    │
    ├── shared/
    │   └── widgets/
    │
    └── features/
        ├── auth/
        ├── onboarding/
        ├── home/
        ├── meal_logging/
        ├── food_catalog/
        ├── meal_planning/
        ├── insights/
        └── profile/
```

Use Riverpod for state management and GoRouter for navigation.

Main bottom navigation: - Home. - Plan. - Insights. - Profile.

Meal logging should be accessible from the Home screen.

Maintain consistent theme, spacing, typography and reusable components.
Follow the supplied Nutrithm prototype's design direction.

------------------------------------------------------------------------

# 17. Supabase setup documentation

Create a file named `SUPABASE_SETUP.md`.

Provide exact, sequential dashboard instructions that I can follow
myself.

Include: 1. Creating a Supabase project. 2. Enabling Email/Password
authentication. 3. Configuring Google OAuth through Google Cloud and
Supabase. 4. Configuring Android redirect/deep-link URLs. 5. Creating
PostgreSQL enums, tables and relationships. 6. Running SQL migration
files. 7. Enabling RLS. 8. Setting up RLS policies. 9. Enabling
`pg_trgm`. 10. Importing the INDB dataset into Supabase. 11. Configuring
environment variables. 12. Generating/configuring API credentials.

I should be able to follow these instructions without needing to
understand the underlying implementation.

Also create a separate `FOOD_DATABASE_IMPORT.md` explaining: - Where to
obtain the INDB data. - Where to place downloaded source files. - What
fields are expected. - How the import script works. - How to run it. -
How to verify the imported records. - How to rerun the import after a
future dataset update.

Do not claim the food database has been imported until the actual
dataset file has been supplied and processed.

------------------------------------------------------------------------

# 18. Environment variables

Flutter:

``` env
SUPABASE_URL=
SUPABASE_ANON_KEY=
NEST_API_BASE_URL=
```

NestJS:

``` env
PORT=

SUPABASE_URL=
SUPABASE_ANON_KEY=
SUPABASE_SERVICE_ROLE_KEY=

AI_PROVIDER=gemini
GEMINI_API_KEY=
GEMINI_MODEL=

FIREBASE_PROJECT_ID=
FIREBASE_CLIENT_EMAIL=
FIREBASE_PRIVATE_KEY=
```

Firebase must also be configured in Android using the appropriate
`google-services.json` file.

Never expose any private backend credentials in Flutter.

------------------------------------------------------------------------

# 19. Final instructions

-   Keep the application lean.
-   Do not overengineer.
-   Do not introduce microservices.
-   Do not introduce Redis, Kafka, RabbitMQ or an unnecessary ORM.
-   Keep Supabase as the primary database.
-   Keep NestJS limited to the responsibilities described above.
-   Keep the food database locally imported into Supabase.
-   Never depend on external INDB queries during runtime.
-   Keep AI and food data independent.
-   Keep business rules configurable instead of hardcoding them in UI
    components.
-   Use proper loading, empty and error states.
-   Maintain clean naming conventions.
-   Avoid generating unnecessary documentation or code.
-   Do not run tests or production builds.
-   Do not install packages.

**Project Status:** STEP 1 (Dependencies), STEP 2 (Architecture Review), and STEP 3 (Implementation Generation) are completed. The project is currently in the **schema verification, refinement, and environment configuration stage**. Refer to `DEVELOPMENT_LOG.md` for prompt-by-prompt evolution and current tasks.
