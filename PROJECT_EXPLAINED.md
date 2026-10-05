# Project Explained: What We Did, How We Did It, How to Run It

Use this document to prepare for interviews. Read Part 1 and Part 2 until you can say them without looking.

---

## Part 1. The project in one minute

### The 30-second pitch
> "I analysed an online store's sales to find where it makes and loses money. Using Python and SQL, I cleaned the data, then looked at revenue, profit margin, discounts, customer segments, retention and delivery. The big finding was that Electronics brings 73% of revenue but only an 8% margin, and discounts of 30% lose money on every sale. I recommended capping Electronics discounts and moving marketing spend to the channels whose customers come back."

### The 2-minute walkthrough (say it in this order)
1. **Problem:** a retail business wants to know where profit is leaking and which customers matter.
2. **Data:** four tables: customers, products, orders, order items (about 3,000 customers, 6,000 orders, 2023 to 2024). It is simulated data with realistic mess in it.
3. **Cleaning:** removed 59 duplicate orders, merged 19 spellings of city names into 10 cities plus "Unknown", excluded cancelled and returned orders from revenue.
4. **Analysis:** KPIs, monthly trend, category revenue vs margin, discount impact, channel quality, RFM customer segments, cohort retention, delivery delay vs rating.
5. **SQL:** I answered the same questions with 9 SQL queries (CTEs, joins, window functions) and checked that the numbers matched Python.
6. **Result:** three headline insights and four recommendations (see Part 3).
7. **Limits and next step:** the data is simulated and profit ignores shipping and returns, so next I would repeat it on a real dataset and build a Power BI dashboard.

---

## Part 2. What we did, step by step

For each step: **what** we did, **why** it matters, **how** it is done in the code.

### Step 1. Create the data (`data/generate_data.py`)
- **What:** a script builds four CSV files with a fixed random seed, so the data is reproducible.
- **Why:** it lets the project run anywhere with no downloads, and the data has business patterns to discover.
- **How:** products have a cost and a list price. Electronics costs 85% of its list price, so it has a thin margin. Each order line gets a discount of 0 to 30%, and electronics get bigger discounts. Late deliveries lower ratings. We then added mess on purpose: duplicate orders, inconsistent city spellings, missing cities, cancelled and returned orders.
- **Be honest about it:** the data is simulated. The patterns in it were designed in, so the project shows your *method*, not a discovery about a real company.

### Step 2. Clean the data (`analysis.py`, section 1)
| Problem | Fix | Why |
|---|---|---|
| 59 duplicate order rows | `drop_duplicates("order_id")` | duplicates would inflate revenue |
| 19 spellings of cities (bangalore, BENGALURU, New Delhi...) | strip spaces, title-case, map a few names | otherwise one city is counted as several |
| 40 missing cities | fill with "Unknown" | keep the customers, don't lose their revenue |
| Cancelled and returned orders | keep only `Delivered` for revenue | money was never kept |

### Step 3. Work out the money (`analysis.py`)
- **Revenue** of an order line = list price × quantity × (1 − discount%)
- **Cost** = unit cost × quantity
- **Profit** = revenue − cost, and **margin %** = profit ÷ revenue
- These are calculated per order line first, then summed. This is what lets us see profit by category and by discount level.

### Step 4. Headline KPIs
Revenue ₹3.92 Cr, profit ₹71 lakh, margin 18.2%, 5,446 delivered orders, 2,913 customers, average order value about ₹7,194, repeat-customer rate 54.5%.

### Step 5. Revenue trend
Monthly revenue bar chart. Revenue grew about 18% from 2023 to 2024. The months are lumpy because a few expensive electronics orders swing each month, so there is no clean festive peak. We said so openly instead of forcing a story.

### Step 6. Category performance
Electronics is 73% of revenue at an 8% margin. Beauty, Fashion and Sports are small but earn 43 to 58%. **Lesson: revenue is not profit.**

### Step 7. Discount impact
Margin by discount level: 25% at no discount, 15% at 10% off, 2% at 20% off, **−16% at 30% off**. On Electronics at 30% off, margin is −21%. We found 315 order lines sold below cost, losing about ₹4.4 lakh, and 99.8% of that was Electronics. **Why it happens:** Electronics cost is 85% of the list price, so any discount above about 15% eats the whole margin.

### Step 8. Acquisition channels
Repeat rate: Referral 75%, Email 72%, Organic 60%, Paid Search 46%, Social Media 35%. Social customers bring about 40% less revenue per customer than Email customers.

### Step 9. RFM segmentation
RFM scores each customer on three things:
- **R**ecency: days since the last order (scored 1 to 5, recent is better)
- **F**requency: number of orders (capped at 5)
- **M**onetary: total revenue (scored 1 to 5)

Rules, applied in this order: *Champions* (R ≥ 4, F ≥ 3, M ≥ 4), *Loyal* (F ≥ 3), *At risk* (R ≤ 2 and 2 orders), *New customers* (R ≥ 4 and 1 order), *Hibernating* (R ≤ 2), everything else *Needs attention*.
Result: Champions are 7% of customers but 19% of revenue. Champions plus Loyal are 23% of customers and 41% of revenue. These cut-offs are a judgement call, not a law.

### Step 10. Cohort retention
Group customers by the month of their first order, then see what percentage order again 1, 2, 3 months later. On average about 20% return in month 1 and about 13% in month 3. This shows a leaky bucket right after the first purchase.

### Step 11. Delivery vs ratings
On-time orders average 4.43 stars, 3 to 4 days late 3.71, 5 or more days late 2.74. Patna, Lucknow and Jaipur are the slowest cities (6 to 7 days against a 5-day promise).

### Step 12. SQL (`sql/`, `run_sql.py`)
- `00_views.sql` builds two views: `clean_orders` (removes duplicates) and `sales_lines` (delivered order lines with revenue and cost).
- `queries.sql` has 9 queries: KPIs, monthly growth with `LAG`, category margin, discount margin, top customers with `RANK`, repeat rate (CTE), repeat rate by channel, at-risk customers, rating by days late.
- **Why do both Python and SQL?** Analysts use SQL to pull data and Python or BI tools to explore and present it. Matching results also proves the numbers are right.

### Step 13. Recommendations
1. Cap Electronics discounts at 10 to 15%, and shift promotions to categories that stay profitable at 20 to 30% off.
2. Move spend from Social and Paid Search to Referral and Email.
3. Win-back email to the 1,234 repeat customers silent for 90+ days, plus a second-order offer in the first 30 days.
4. Fix last-mile delivery in Patna, Lucknow and Jaipur, or promise 7 days there.

---

## Part 3. Numbers cheat sheet

| Item | Value |
|---|---|
| Revenue / profit / margin | ₹3.92 Cr / ₹71 lakh / 18.2% |
| Orders / customers | 5,446 / 2,913 |
| Year-on-year revenue growth | about 18% |
| Electronics share of revenue / margin | 73% / 8.4% |
| Margin at 0% / 20% / 30% discount | 25% / 2% / −16% |
| Loss-making lines | 315 (3.3%), about ₹4.4 lakh lost |
| Repeat rate: Referral / Email / Paid / Social | 75% / 72% / 46% / 35% |
| Champions | 7% of customers, 19% of revenue |
| Month-1 / month-3 retention | about 20% / 13% |
| Rating: on time vs 5+ days late | 4.43 vs 2.74 |

If you re-run the code and a number differs slightly, trust your run's output (`outputs/findings.json`) and update the figures you quote.

---

## Part 4. Concepts in plain words

- **Revenue vs profit vs margin:** revenue is money in; profit is money in minus cost; margin is profit as a percentage of revenue.
- **Why exclude cancelled and returned orders:** the business didn't keep that money.
- **Duplicates:** the same order appearing twice. Left in, they overstate revenue.
- **CTE (`WITH`):** a named temporary result that makes a long query readable.
- **Window function (`LAG`, `RANK`):** calculates across related rows without collapsing them. `LAG` looks at the previous row (last month), `RANK` numbers rows in order.
- **RFM:** a simple way to score customers by how recently, how often and how much they buy.
- **Cohort:** a group of customers who started in the same month, followed over time.
- **Correlation vs causation:** late delivery goes with low ratings, but we did not prove it causes them. Channel differences are also not proven causal because customers were not randomly assigned to channels.

---

## Part 5. Interview questions and model answers

**1. Is this real data?**
No, it is a simulated dataset I generated with realistic problems built in. I did that so the project is fully reproducible. The method (cleaning, SQL, segmentation, cohorts) is what transfers, and I plan to repeat it on a real dataset like Olist.

**2. How did you build it? Did you use AI tools?**
Answer truthfully about the tools you used. Whatever you say, you must be able to explain and change every line. Practise by changing a discount rule or a segment threshold and re-running.

**3. What was the main finding?**
Revenue and profit tell different stories. Electronics is 73% of revenue at an 8% margin, and discounts above about 15% on it wipe out the profit.

**4. How did you clean the data?**
Removed duplicate orders by `order_id`, standardised city names, filled missing cities with "Unknown", and kept only delivered orders for revenue.

**5. Why did you count only delivered orders?**
Cancelled and returned orders don't bring in money. A limitation is that I didn't subtract the cost of returns.

**6. How did you calculate profit?**
Per order line: list price × quantity × (1 − discount) minus unit cost × quantity. It ignores shipping, payment fees and marketing cost.

**7. Why does a 30% discount lose money?**
Margin is price minus cost. With cost at 85% of list price for Electronics, selling at 70% of list is below cost. In my data the cost ratios are assumptions, so with real data I'd check actual costs first.

**8. Explain RFM and how you chose the thresholds.**
Recency, frequency and monetary value, each scored 1 to 5. I capped frequency at 5 orders and used quintiles for recency and money. The segment rules are a judgement call. I'd test them against real campaign results.

**9. What does the cohort chart tell you?**
Only about 20% of customers order again within a month of their first purchase, so the first 30 days after purchase is the best place to improve retention.

**10. Which channel is best and how sure are you?**
Referral and Email customers repeat most. But I didn't run a significance test and customers weren't randomly assigned, so I'd call it a strong signal, not proof.

**11. What would you do next with real data?**
Add shipping and return costs, test channel differences statistically, build a dashboard, and check if the discount finding holds with actual cost data.

**12. What was the hardest part?**
Pick something true for you. Good options: deciding what counts as revenue, making Python and SQL totals match, or choosing RFM cut-offs.

**13. Why both SQL and Python?**
SQL is for getting and aggregating data from a database. Python is for deeper analysis, charts and customer segmentation. Matching results is a built-in check.

**14. Write a SQL query for month-over-month growth.**
Practise `queries.sql` number 02: aggregate revenue by month in a CTE, then use `LAG(revenue) OVER (ORDER BY month)` to compute `(revenue − previous) ÷ previous`.

**15. What are the limitations?**
Simulated data, no shipping or return costs, no statistical tests, arbitrary segment thresholds, and monthly revenue is noisy because of a few large orders.

**16. What would you tell the business to do on Monday morning?**
Cap Electronics discounts at 15%, email the 1,234 silent repeat customers, and shift some ad spend toward referral and email.

---

## Part 6. How to run the project

### What you need
- **Python 3.10 or newer** (tested on Python 3.12 with pandas 3.0). Check with `python --version` (on Windows try `py --version`).
- A terminal: Command Prompt or PowerShell on Windows, Terminal on Mac or Linux.
- Optional: **VS Code** with the Python extension, and **DB Browser for SQLite** to look inside the database.

### Steps
1. Unzip `retail-analytics.zip` and open a terminal inside the `retail-analytics` folder.
2. Create and activate a virtual environment:
   - Windows: `python -m venv .venv` then `.venv\Scripts\activate`
   - Mac/Linux: `python3 -m venv .venv` then `source .venv/bin/activate`
3. Install the libraries: `pip install -r requirements.txt`
4. (Optional) regenerate the data: `python data/generate_data.py`. The CSVs are already included, so you can skip this.
5. Run the analysis: `python analysis.py`
   - Prints the KPIs and findings.
   - Saves 7 charts to `outputs/charts/`, tables to `outputs/tables/`, and `outputs/findings.json`.
6. Run the SQL: `python run_sql.py`
   - Builds `retail.db` and prints all 9 query results, also saved to `outputs/sql_results.txt`.
7. Open `outputs/charts/` to see the charts.

### Using it like a notebook
Open `analysis.py` in VS Code. Each `# %%` line marks a cell, and you will see a "Run Cell" button above it. For a real Jupyter notebook, run `pip install jupyter`, then copy the cells in. If you run it in a notebook, delete the `matplotlib.use("Agg")` line so charts appear inline.

### Troubleshooting
| Problem | Fix |
|---|---|
| `python` not found | use `python3` (Mac/Linux) or `py` (Windows) |
| `ModuleNotFoundError: pandas` | activate the virtual environment, then run `pip install -r requirements.txt` again |
| `FileNotFoundError` for the CSVs | run commands from the `retail-analytics` folder, not from inside `data/` or `sql/` |
| Charts don't pop up | they are saved as PNG files in `outputs/charts/`, not displayed in a window |
| `retail.db` locked or can't be deleted | close DB Browser, then run `run_sql.py` again |

### Push it to GitHub
Create a new repository, then in the project folder:
```
git init
git add .
git commit -m "Retail sales, profit and retention analysis"
git branch -M main
git remote add origin https://github.com/<your-username>/retail-sales-profit-analysis.git
git push -u origin main
```
Embed a chart in your README with `![](outputs/charts/03_discount_margin_heatmap.png)` so visitors see results straight away.

---

## Part 7. Before the interview checklist

- [ ] I can give the 30-second pitch and the 2-minute walkthrough from memory.
- [ ] I ran every script and I can find each chart and table.
- [ ] I can write queries 02, 04 and 06 from `queries.sql` without looking.
- [ ] I can explain RFM, cohort retention and margin in plain words.
- [ ] I changed one thing (for example a discount cap or segment rule) and re-ran it.
- [ ] I can say clearly that the data is simulated and what I would do with real data.
- [ ] The dashboard and GitHub links are on my resume and LinkedIn.
