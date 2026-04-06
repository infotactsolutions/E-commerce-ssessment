# ShopFast — Junior Data Analyst Take-Home Assignment

**Estimated effort:** 4–6 hours  
**Tools:** Python *or* Excel (Part 1), SQL on the provided CSVs (Part 2), any charting tool (Part 3)

---

## 1. Business context

**ShopFast** is an e-commerce company. Over the **last six months**, **revenue from repeat customers** is down about **15%** compared with the prior period. Marketing leadership wants to understand **why customers are not returning** as strongly and **which product categories** may be underperforming.

You are the junior analyst on the project. Your job is to **clean the operational extracts**, **compute core metrics in SQL**, **visualize at least one key relationship**, and deliver **three concrete, data-backed recommendations** for the Head of Marketing.

---

## 2. What you will submit

| # | Deliverable | Format |
|---|-------------|--------|
| 1 | **Analysis notebook or script** | Jupyter Notebook (`.ipynb`) *or* Python scripts + comments *or* documented Excel workbook (clear sheets/tabs named) |
| 2 | **SQL** | `.sql` file(s) *or* SQL cells in the notebook — every analytical question in Part 2 must be answered with query text you would run |
| 3 | **Executive summary** | **One-page PDF** *or* **3 slides** (PDF or PPTX) aimed at a non-technical Head of Marketing |
| 4 | **Recommendations** | At least **three** specific actions; each must **reference a number or chart** from your work (e.g. “Category X is Y% of revenue but Z% of 1-star reviews”) |

**Optional but welcome:** a `README` in your fork/submission explaining how to run your code and which SQL engine you used.

---

## 3. Dataset bundle (`data/`)

All files are UTF-8 CSVs with headers.

| File | Description |
|------|-------------|
| `customers.csv` | One row per customer |
| `orders.csv` | One row per order line in the extract (**may contain duplicates**) |
| `order_items.csv` | Line items for **completed** orders (product/category economics) |
| `reviews.csv` | Reviews linked to orders/products (**some fields intentionally messy**) |

### 3.1 Data dictionary

**`customers.csv`**

| Column | Description |
|--------|-------------|
| `customer_id` | Unique customer key (e.g. `C0001`) |
| `signup_date` | When the account was created |
| `region` | Broad geography bucket |
| `acquisition_channel` | How the customer was acquired: `Social`, `Search`, or `Email` |

**`orders.csv`**

| Column | Description |
|--------|-------------|
| `order_id` | Order identifier (**should** be unique per real order; the extract may violate this) |
| `customer_id` | Customer who placed the order |
| `order_date` | Order placement date (**format may vary** — see Part 1) |
| `order_status` | `Completed`, `Cancelled`, `Refunded`, or `Pending` |
| `total_amount` | Order total in **USD** (as recorded in this extract) |
| `shipped_date` | Ship date (often empty if not completed/shipped) |
| `delivered_date` | Delivery date |
| `shipping_time_days` | **Pre-calculated** calendar days from `order_date` to `delivered_date` when both exist; you may recompute after cleaning dates |

**`order_items.csv`**

| Column | Description |
|--------|-------------|
| `order_id` | Links to `orders.order_id` |
| `product_id` | Product identifier |
| `category` | Merchandising category |
| `price` | Line price (USD) |
| `shipping_cost` | Allocated shipping portion for the line (USD) |

**`reviews.csv`**

| Column | Description |
|--------|-------------|
| `order_id` | Links to `orders.order_id` |
| `product_id` | Product reviewed |
| `review_score` | Integer **1–5** when present (**may be missing**) |
| `comment` | Free text (may be empty) |

---

## 4. Metric definitions (use these in your analysis)

These definitions are **binding** for this assignment so your answers are comparable.

### 4.1 “Completed” orders

Only rows where `order_status = 'Completed'` count toward **revenue**, **MAU** (below), and **CLV** (below), unless a task explicitly says otherwise.

### 4.2 Monthly Active Users (MAU)

For calendar month **M**: **distinct** `customer_id` who have **at least one** **Completed** order with `order_date` falling in month **M**.

Report MAU for the **last 12 full calendar months** present in the data (from the latest month in the file, backward).

### 4.3 Revenue by category

For **Part 2**, “revenue” at category level = sum of **`price` + `shipping_cost`** from `order_items.csv` for line items whose `order_id` belongs to a **Completed** order in `orders.csv`.

### 4.4 Customer Lifetime Value (CLV) — assignment definition

**Not** a predictive model. For each `customer_id`:

- **Customer lifetime value** = sum of `total_amount` over all **Completed** orders for that customer.

Then report **average CLV** **grouped by** `acquisition_channel` (from `customers.csv`).

Customers with **no** completed orders may be excluded from the average **or** included as **0** — **pick one approach**, state it in your write-up, and apply it consistently.

### 4.5 Duplicate orders

Treat rows as **duplicates** if they share the same **`order_id`** *and* the same **`customer_id`**, **`order_date`**, **`order_status`**, and **`total_amount`** (i.e. exact repeated extract rows). **Keep one** row per logical duplicate group and document your rule (e.g. keep first occurrence).

---

## 5. Assignment tasks

### Part 1 — Data cleaning (Python or Excel)

1. **Dates:** Normalize all date columns you use (`signup_date`, `order_date`, and any ship/delivery fields) to a **single** parseable format in your cleaned dataset or staging tables. **Document** how you handled ambiguous or invalid values.
2. **Reviews:** Address **missing** `review_score` values (impute, drop, or flag — **justify** your choice for downstream charts/SQL).
3. **Duplicates:** Identify and **remove** duplicate **order** records per the rule in §4.5.
4. **QA:** State **one** sanity check you ran (e.g. row counts before/after, non-negative amounts, date ordering).

### Part 2 — SQL / aggregation

Using **SQL** against the cleaned data (import CSVs into SQLite, PostgreSQL, DuckDB, BigQuery, etc. — your choice):

1. **MAU trend:** Result set with **month** and **MAU** for the last **12** months per §4.2.
2. **Top categories:** **Top 5** product **`category`** values by **total revenue** per §4.3 (show category name and revenue).
3. **CLV by channel:** **Average CLV** per **`acquisition_channel`** per §4.4 (show channel and average CLV).

> **Tip:** If your engine needs it, describe or script the **import** step; graders care most about **correct SQL logic** and **clear joins**.

### Part 3 — Visualization & retention narrative

1. **Chart:** One visualization exploring the relationship between **shipping time** (use `shipping_time_days` from **Completed** orders with non-null values, after cleaning) and **`review_score`** (join or merge as needed). Label axes and state what you conclude **in plain language**.
2. **Retention health:** One **small dashboard** (e.g. 2–4 charts in a notebook, Looker/Tableau/Power BI, or the same 3-slide deck) that summarizes **retention-related** signals — e.g. MAU trend, repeat vs one-time buyers over time, category mix, or review/shipping patterns. You do **not** need a formal “retention model”; you **do** need a **coherent story** for Marketing.

### Part 4 — Recommendations

Provide **at least three** recommendations that are:

- **Specific** (who/what/when at a high level),
- **Grounded** in **your** tables/charts,
- **Distinct** (not three versions of the same idea).

---

## 6. How to submit

Follow the instructions from your recruiter. Typical patterns:

- **GitHub:** Fork or clone this repo, push to **your** repository, and share the link; **do not** commit secrets.
- **Zip:** `code/` + `sql/` + `report.pdf` in one archive named `YourName_ShopFast_Analyst.zip`.

**Deadline:** [set by recruiter]

---

## 7. Evaluation rubric (what we look for)

| Area | Weight | We reward |
|------|--------|-----------|
| **Data handling** | ~30% | Correct duplicate logic, defensible missing-value and date handling, reproducible steps |
| **SQL** | ~30% | Right grain (order vs line vs customer), correct filters for Completed orders, readable queries |
| **Visualization & narrative** | ~25% | Clear charts, honest labels, insight tied to the business problem |
| **Executive communication** | ~15% | Non-technical summary, prioritized recommendations with evidence |

**Common pitfalls:** counting cancelled orders as revenue; double-counting revenue after failing to dedupe orders; MAU that is “distinct orders” instead of **distinct customers**; CLV averages that omit the join to `customers` for channel.

---

## 8. Integrity & support

- **Allowed:** Documentation, Stack Overflow-style references, generative AI **if** your employer allows it — **you must** understand and be able to explain every query and figure.
- **Not allowed:** Submitting another candidate’s work or a shared solution from a forum.

Questions about **ambiguous business rules**? State your assumption **in the report** and proceed.

---

## 9. Repository layout (for candidates)

```text
shopfast-analyst-assignment/
├── LICENSE            ← MIT (see file for terms)
├── README.md          ← this file (instructions)
├── data/
│   ├── customers.csv
│   ├── orders.csv
│   ├── order_items.csv
│   └── reviews.csv
└── scripts/
    └── generate_datasets.py   ← for maintainers only; candidates use data/ as-is
```

**Candidates:** work only from the **`data/`** CSVs unless you are explicitly told otherwise. The `scripts/` folder is for **maintainers** who may regenerate synthetic data; your analysis should not depend on running it.

---

*Synthetic data for assessment purposes — not real customer or business information.*
