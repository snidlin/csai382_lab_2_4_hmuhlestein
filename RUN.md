# RUN.md

This repo contains a Databricks notebook ETL run with reproducible logging, input hashing, Pandas ETL, metrics output, and basic asserts.

## Repo layout
├─ data/
│ ├─ menu_items.csv
│ └─ order_details.csv
├─ etl_output/ # generated
├─ logs/ # generated
├─ data_hashes.json # generated
├─ README.md
└─ RUN.md


## Databricks setup

In Databricks:
1) Repos → Add Repo → select this GitHub repo.
2) Create a notebook in the repo
3) Attach to a cluster.

## Run

Run the notebook top-to-bottom.

Artifacts created in the repo working tree:
- `logs/run_YYYYMMDD_HHMM.log`
- `data_hashes.json`
- `etl_output/metrics_YYYYMMDD_HHMM.csv`

## ETL logic

Inputs:
- `data/menu_items.csv` columns: `menu_item_id, item_name, category, price`
- `data/order_details.csv` columns: `order_details_id, order_id, order_date, order_time, item_id`

Datetime parsing:
- `_order_dt` is created from `order_date + " " + order_time`
- Format: `%m/%d/%y %I:%M:%S %p` (example: `1/1/23 11:38:36 AM`)

Join:
- `order_details.item_id` → `menu_items.menu_item_id` (left join)

Derived:
- `revenue = price * quantity` (if `quantity` missing, defaults to 1)

Metrics:
- Top 5 items by total quantity
- Total revenue by category
- Busiest hour of day by row count (hour from `_order_dt`)

Output:
- `etl_output/metrics_YYYYMMDD_HHMM.csv` with columns: `metric, key, value`
  - `top_item_quantity` (key=item_name, value=quantity)
  - `revenue_by_category` (key=category, value=revenue)
  - `busiest_hour` (key=hour, value=row_count)

## Validate

Notebook asserts:
- `tidy` is non-empty
- `metrics_df` is non-empty
- required columns exist in `tidy`: `order_id, item_id, quantity, revenue`

## Push to GitHub

Files written by the notebook are in the repo folder in Databricks and are not on GitHub until committed + pushed.

From a notebook cell:

```bash
%sh
cd /Workspace/Users/hmm733@ensign.edu/csai382_lab_2_4_hmuhlestein
git status
git add data_hashes.json etl_output logs
git commit -m "Add ETL outputs and reproducibility artifacts"
git push
