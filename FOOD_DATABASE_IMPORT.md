# Indian Nutrient Databank (INDB) Import Guide

This document describes how to obtain, prepare, and import the Indian food dataset into Nutrithm's local Supabase PostgreSQL database.

---

## 1. Source Data Acquisition

1. Visit the Indian Nutrient Databank (INDB) website:
   https://www.anuvaad.org.in/indian-nutrient-databank/
2. Locate the downloadable food composition dataset (available in CSV or Excel `.xlsx` format).
3. If INDB requires registration or direct download, obtain the file and place it in:
   ```
   nutrithm_backend/scripts/food-import/source/
   ```
4. Supported file extensions: `.csv`, `.xlsx`, `.xls`.

---

## 2. Directory Layout

```
nutrithm_backend/
└── scripts/
    └── food-import/
        ├── source/              <-- Place your downloaded INDB file here
        │   └── indb_foods.csv   (or .xlsx)
        ├── transform/
        │   ├── parser.ts        <-- Reads CSV or Excel rows into JS objects
        │   └── normalizer.ts    <-- Maps column headers to Nutrithm schema
        ├── import.ts            <-- Main execution script that upserts into Supabase
        └── README.md
```

---

## 3. Expected Columns & Field Mapping

The parser in `transform/normalizer.ts` normalizes incoming fields to a consistent standard per 100 grams:

| INDB Expected Header (Case-Insensitive) | Nutrithm Target Column | Default / Fallback |
|---|---|---|
| `code`, `food_code`, `id`, `food_id` | `source_food_id` | Auto-generated hash if absent |
| `name`, `food_name`, `item_name`, `food_item` | `food_name` | Required |
| `group`, `food_group`, `category` | `category` | `'General'` |
| `energy_kcal`, `calories`, `energy`, `kcal` | `calories_per_100g` | `0.0` |
| `protein_g`, `protein`, `prot` | `protein_per_100g` | `0.0` |
| `carbohydrate_g`, `carbohydrate`, `carbs`, `cho` | `carbs_per_100g` | `0.0` |
| `fat_g`, `fat`, `total_fat`, `lipid` | `fat_per_100g` | `0.0` |
| `fibre_g`, `fiber`, `fibre`, `dietary_fibre` | `fibre_per_100g` | `0.0` |
| `aliases`, `alternative_names`, `local_names` | `food_aliases` | Split by comma/semicolon |

If your specific INDB download uses different column headers, you can easily adjust the mappings in `nutrithm_backend/scripts/food-import/transform/normalizer.ts`.

---

## 4. Standard Household Portions

During import, standard portion measurements with conversion factors are seeded for each food item:
- **Katori**: 150g (liquid/dal/curd) or 100g (cooked grains/vegetables)
- **Bowl**: 250g
- **Glass**: 200g (milk/beverages)
- **Roti / Piece**: 30g (per roti) or standard piece weight
- **Tablespoon (tbsp)**: 15g
- **Teaspoon (tsp)**: 5g

All imported portions start with status `'unverified'` until confirmed against empirical serving standards.

---

## 5. How to Run the Import

### Prerequisites
1. Ensure `nutrithm_backend/.env` is configured with valid Supabase credentials:
   ```env
   SUPABASE_URL=https://your-project.supabase.co
   SUPABASE_SERVICE_ROLE_KEY=your-service-role-key
   ```
2. Make sure dependencies are installed in `nutrithm_backend`.

### Execution
From the root of the project:

```bash
cd nutrithm_backend
npx tsx scripts/food-import/import.ts
```
*(or using `npx ts-node --esm scripts/food-import/import.ts`)*

The script will:
1. Scan `scripts/food-import/source/` for dataset files.
2. Register or update the INDB entry in `food_data_sources`.
3. Normalize records and upsert them into `foods` (idempotent, using `(source_id, source_food_id)` unique constraint).
4. Upsert aliases into `food_aliases`.
5. Seed default household portions into `food_portions`.
6. Output a summary of total inserted, updated, and skipped records.

---

## 6. How to Verify Imported Records

Run the following queries in the Supabase **SQL Editor**:

```sql
-- 1. Check data source record
SELECT * FROM food_data_sources;

-- 2. Count imported foods
SELECT count(*) FROM foods WHERE is_active = true;

-- 3. Sample 10 foods with their nutritional values per 100g
SELECT food_name, category, calories_per_100g, protein_per_100g, carbs_per_100g, fat_per_100g, fibre_per_100g
FROM foods
LIMIT 10;

-- 4. Check portions for a specific food (e.g. roti)
SELECT f.food_name, p.portion_name, p.grams_per_unit, p.is_default, p.verification
FROM foods f
JOIN food_portions p ON f.id = p.food_id
WHERE f.food_name ILIKE '%roti%'
LIMIT 10;

-- 5. Test fuzzy food search with pg_trgm
SELECT food_name, similarity(food_name, 'palak dhal') AS score
FROM foods
WHERE food_name % 'palak dhal'
ORDER BY score DESC
LIMIT 5;
```

---

## 7. Re-running and Updating

The import script is fully **idempotent**. When INDB releases an updated version of their dataset:
1. Replace the file in `scripts/food-import/source/`.
2. Update the `DATASET_VERSION` string in `import.ts` (or let it auto-detect).
3. Re-run `npx tsx scripts/food-import/import.ts`.
4. Existing foods will be updated, new foods inserted, and no duplicate records created.
