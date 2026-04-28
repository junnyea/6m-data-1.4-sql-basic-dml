# 🎓 Instructor Guide — Lesson 1.4: SQL Basic — DML

> **Branch:** `feature/instructor-guide`
> **Audience:** Instructors and teaching assistants
> **Companion to:** `lesson.md`, `pre-class.md`, `assignment.md`

---

## 1. Lesson Overview & Instructor Objectives

| | |
|---|---|
| **Duration** | 3 hours |
| **Format** | Flipped Classroom + Hands-On SQL in DbGate |
| **Dataset** | Singapore HDB Resale Flat Prices 2017 |
| **Learner entry point** | Completed 1.3 (DDL); can create tables and import data; no SELECT experience |

By the end of this lesson learners should be able to:

1. Write `SELECT` queries using `WHERE`, `ORDER BY`, `DISTINCT`, `LIKE`, and arithmetic operators.
2. Summarise datasets using aggregate functions (`COUNT`, `SUM`, `AVG`, `MIN`, `MAX`).
3. Group results with `GROUP BY` and filter aggregated results with `HAVING`.
4. Transform data using `CASE` statements and `CAST` for type conversion.

**Instructor's primary job:** DML is where data *comes alive* for learners — they're no longer defining structure, they're answering questions. Frame every query as answering a business question, not as a syntax exercise. Use the HDB dataset as a real-world context that Singapore-based learners will find immediately relatable.

---

## 2. Concept Analogies

### The Conversation Analogy (SELECT basics)

> "Querying a database is having a conversation with your data. You ask in a structured way, and the database answers."

| Clause | What You're Saying |
|--------|--------------------|
| `SELECT` | "Give me these columns..." |
| `FROM` | "...from this table..." |
| `WHERE` | "...but only rows that match this rule..." |
| `ORDER BY` | "...and arrange the answer this way." |

**The execution order revelation:** Learners often write queries top-to-bottom (SELECT then FROM) but databases *execute* them in a different order: `FROM` → `WHERE` → `GROUP BY` → `HAVING` → `SELECT` → `ORDER BY`. This explains why you can't use a column alias from SELECT in a WHERE clause — WHERE runs *before* SELECT.

---

### WHERE vs. HAVING — "Before the Kitchen vs. After the Kitchen"

This is the single most important distinction in the lesson.

> "Think of a restaurant. WHERE is the rule at the *ingredient level* — 'only use organic tomatoes.' HAVING is the rule at the *dish level* — 'only serve meals that weigh more than 300g.' You can't apply the dish rule before the dish is made."

| Clause | When it runs | What it filters |
|--------|-------------|----------------|
| `WHERE` | Before aggregation | Individual rows |
| `HAVING` | After aggregation | Group summaries |

**The "This will fail — why?" query** in the lesson is the best teaching moment for this. Run it together, let learners see the error, then ask them to explain what went wrong before giving the answer.

---

### GROUP BY — "Sorting Mail into Piles"

> "GROUP BY is like sorting your mail into piles by sender. Once you've sorted them, you can count how many letters each sender sent, or find the heaviest envelope from each pile. But you can no longer talk about individual letters — you're working with piles now."

This directly explains why you can't SELECT a non-aggregated column that isn't in the GROUP BY — once rows are grouped into piles, there's no single value for that column anymore.

---

### CASE — "The Label Maker"

> "CASE is the SQL label maker. You feed it raw values and it sticks a human-readable category on them. Instead of seeing '420000' in your results, you see 'Entry-Level'."

Connect this to business reporting: finance teams don't want to see raw numbers in reports — they want categories, bands, and buckets. CASE is how you build those.

---

### CAST — "The Translator"

> "Dates stored as text are like a calendar written in a language the computer doesn't understand. CAST is the translator — it converts '2017-01' (a string) into an actual date object the database can do math on."

**The "can you add 7 days to text?" test:** Write `'2017-01' + 7 days` on the board. Ask: can the database compute this? (No.) Then write `CAST('2017-01-01' AS DATE) + INTERVAL 7 DAYS`. Now it can. That's why CAST matters.

---

## 3. Real-World Use Cases

### The HDB Dataset — Singapore Context

This dataset is highly relevant for Singapore-based learners. Before starting:
- "This is real transaction data — these are actual prices people paid for flats."
- "This is the kind of data PropertyGuru and 99.co query thousands of times a day when users use their search filters."

### SELECT * in Production

Real-world rule: `SELECT *` is almost never used in production code. When a table has 50 columns and millions of rows, selecting all columns for every query wastes memory and slows the database. Senior engineers write specific column names. Frame `SELECT *` as the "learning wheels" — useful for exploring a new table, never in production queries.

### GROUP BY for Government Planning

The reflection question about government planners is a genuine use case. The Singapore government uses HDB transaction data for exactly this:
- Which towns have the highest transaction volumes? → Where to focus MRT expansion.
- Which flat types have rising average prices? → Where to increase supply.

### CASE in Financial Services

Banks use CASE statements extensively to create customer segments:
```sql
CASE
  WHEN annual_income > 250000 THEN 'Private Banking'
  WHEN annual_income > 80000 THEN 'Priority Banking'
  ELSE 'Retail Banking'
END AS customer_tier
```
This segmentation drives which products are shown, which interest rates apply, and which relationship managers are assigned.

---

## 4. Activity Facilitation Notes

### Part 1: Basic Retrieval & Sorting

**Opening move:** Before writing any SQL, show learners the raw table with `SELECT * FROM resale_flat_prices_2017 LIMIT 10`. Ask:
- "What kind of questions could you answer with this data?"
- Let them brainstorm: "Which town has the most expensive flats? Which flat type is most common? Has Punggol gotten more expensive over time?"

Then frame the lesson: "Everything we do today is learning the vocabulary to answer those questions."

**The `SELECT *` memory question:** Pause here for 1 minute. Ask: "If this table had 100 columns and a million rows, what happens to your computer's memory?" → Out-of-memory errors, slow queries. This motivates always selecting specific columns.

**Common mistake at this stage:** Learners forget to include `FROM` or misspell the table name. DbGate's autocomplete helps — encourage using it.

---

### Part 2: GROUP BY & HAVING (the critical 55 min)

**Run the "broken" query first:**
```sql
-- This fails — run it intentionally
SELECT town, AVG(resale_price) AS avg_price
FROM resale_flat_prices_2017
WHERE avg_price > 600000
GROUP BY town;
```

Read the error message aloud. Ask: "Why did it fail?" Accept wrong answers before guiding toward the correct one. The goal is for learners to *understand the error*, not just memorize the fix.

**Then fix it with HAVING** and ask: "What changed? What does HAVING do that WHERE can't?" Reinforce: WHERE = row-level filter (before grouping), HAVING = group-level filter (after grouping).

**The "multiple GROUP BY" query** is a natural progression: group by `(town, lease_commence_date)` to show that you can group by multiple dimensions. Ask: "If you're a buyer looking for new flats (lease year > 2010) in central areas, which query answers your question?"

---

### Part 3: CASE and CAST

**For CASE — make it personal:**
Ask: "In your industry, what categories do you use to segment data?" Let learners suggest their own categories, then help them write a CASE statement for it. Personalised examples stick better than generic ones.

**For CAST and dates:**
The `month` column in the HDB dataset is stored as text `'2017-01'`. Before teaching CAST, ask: "Can you ORDER BY this column and get January before February?" (Yes, because alphabetical order matches date order in this format — but it's coincidental and fragile.) Then ask: "Can you calculate how many months ago each transaction was?" (No — you can't do date math on text.) This motivation makes CAST feel necessary, not arbitrary.

---

## 5. Timing & Pacing Notes

| Part | Planned | Common Overrun | Mitigation |
|------|---------|---------------|-----------|
| Part 1: SELECT/WHERE | 55 min | Learners spend too long on personal exercises | Give a hard 5-min timer for free exercises; bring group back for debrief |
| Part 2: GROUP BY/HAVING | 55 min | WHERE vs HAVING discussion can run 20+ min | After the "broken query" moment, spend max 5 min on WHY before showing the fix — discussion continues in reflection |
| Part 3: CASE/CAST | 55 min | CAST date math takes longer if learners haven't seen date types before | The optional ALTER TABLE exercise can be cut if time is short |

---

## 6. Common Learner Questions

**Q: "Why do SQL keywords have to be uppercase?"**
A: They don't — SQL is case-insensitive for keywords. But uppercase convention makes queries easier to read: you instantly see what's a SQL keyword vs. a column name vs. a table name. Professional practice is to use uppercase keywords.

**Q: "Can I use a column alias in WHERE?"**
A: Not in most SQL engines (including DuckDB). Aliases are assigned in the SELECT step, which runs *after* WHERE. You'd need to repeat the expression or use a subquery / CTE (covered in 1.5).

**Q: "What's the difference between `=` and `LIKE`?"**
A: `=` is exact match — `WHERE town = 'PUNGGOL'` finds only exact matches. `LIKE 'P%'` uses pattern matching — `%` means "any characters after P". Use `=` when you know the exact value; use `LIKE` when searching by partial match.

**Q: "How does PropertyGuru use these filters in real time?"**
A: When a user drags the price slider on PropertyGuru, the frontend sends a request that builds a SQL query dynamically — something like `WHERE resale_price BETWEEN [min] AND [max] AND town = '[selected_town]'`. The response comes back in milliseconds because the table is indexed. This is exactly what you built in this lesson.
