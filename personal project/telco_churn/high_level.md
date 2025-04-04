Below is a real-life-inspired scenario where a consultant leverages existing data to create a solution, delivering standardized datasets that enable analysts to uncover churn or risks. This example is grounded in practical data consulting contexts, aligns with the learning point "Deliver standardized datasets enabling analysts to uncover churn or risks," and reflects the themes from Ben and Dan’s conversation (e.g., data as a product, strategic value, and accessibility).

---

### Scenario: Reducing Customer Churn for a Telecom Provider

#### Background
- **Company**: TelcoX, a mid-sized telecom provider with 2 million subscribers across mobile and broadband services.
- **Problem**: TelcoX is losing ~5% of customers quarterly (high churn rate), costing millions in lost revenue. Leadership suspects poor service quality or pricing issues, but analysts lack clear, unified data to pinpoint causes.
- **Existing Data**: Scattered across silos:
  - Billing system (SQL Server): Payment history, plan details, late fees.
  - CRM (Salesforce): Customer demographics, support tickets, satisfaction scores.
  - Network logs (Hadoop): Call drops, data speeds, outages by region.
- **Current State**: Analysts spend weeks manually joining datasets in Excel, producing inconsistent churn reports—some flag pricing, others network issues, but no consensus or actionable insight.

#### Consultant’s Engagement
- **Role**: A data consultant (you) is hired to unify this mess into a solution that reveals churn drivers and risks.
- **Goal**: Deliver standardized datasets enabling analysts to uncover churn patterns and risks systematically.

---

### The Solution: Standardized Churn Analysis Datasets

#### Step 1: Assess and Explore Existing Data
- **Discovery**: You meet with stakeholders (marketing, ops, finance) and the data team, echoing Dan’s “first 90 days” approach (21:14-22:14).
  - Marketing: “Are customers leaving due to high prices?”
  - Ops: “Is it network outages?”
  - Analysts: “We can’t join data fast enough to test either.”
- **Data Inventory**: You catalog the silos:
  - **Billing**: 10 GB, 2M rows (customer ID, plan cost, payment status, churn flag).
  - **CRM**: 5 GB, 1.5M rows (customer ID, age, tickets, NPS scores).
  - **Network**: 50 GB, 100M rows (customer ID, call drops, speed metrics).
- **Possibility**: Combine these into a star schema to explore churn drivers (e.g., price sensitivity, service quality).

#### Step 2: Design and Build the Solution
- **Platform**: Use Snowflake (cloud data warehouse) for its scalability and ease (per your S3+Parquet pipeline and the SQL Server-to-Snowflake example).
- **Pipeline**:
  1. **Staging**: Load raw data into S3 using DLT (as in your earlier plan):
     - Billing → `s3://telcox/staging/billing_raw.parquet`.
     - CRM → `s3://telcox/staging/crm_raw.parquet`.
     - Network → `s3://telcox/staging/network_raw.parquet`.
  2. **Silver Layer (Star Schema)**: Transform with DuckDB and pandas:
     - **Fact Table**: `churn_fact`
       ```sql
       CREATE TABLE churn_fact AS
       SELECT b.customer_id, b.churn_flag, b.plan_cost, b.payment_late_days,
              c.nps_score, c.support_tickets, n.call_drop_count, n.avg_data_speed
       FROM 's3://telcox/staging/billing_raw.parquet' b
       LEFT JOIN 's3://telcox/staging/crm_raw.parquet' c ON b.customer_id = c.customer_id
       LEFT JOIN 's3://telcox/staging/network_raw.parquet' n ON b.customer_id = n.customer_id;
       ```
       - Size: ~20 GB post-join, stored as Parquet in `s3://telcox/silver/churn_fact/`.
     - **Dimension Tables**:
       - `customer_dim`: `customer_id`, `age`, `region` (~2 MB).
       - `time_dim`: `date_key`, `month`, `quarter` from billing dates (~1 MB).
       - Stored in `s3://telcox/silver/dims/`.
  3. **Gold Layer (Churn Mart)**: Pre-aggregate for analysts:
     ```sql
     CREATE TABLE churn_risk_mart AS
     SELECT t.month, c.region, AVG(f.plan_cost) AS avg_cost, AVG(f.nps_score) AS avg_nps,
            SUM(f.call_drop_count) AS total_drops, COUNT(CASE WHEN f.churn_flag = 1 THEN 1 END) AS churn_count,
            COUNT(*) AS total_customers, (churn_count::FLOAT / total_customers) AS churn_rate
     FROM 's3://telcox/silver/churn_fact/churn_fact.parquet' f
     JOIN 's3://telcox/silver/dims/customer_dim.parquet' c ON f.customer_id = c.customer_id
     JOIN 's3://telcox/silver/dims/time_dim.parquet' t ON f.date_key = t.date_key
     GROUP BY t.month, c.region;
     ```
     - Size: ~50 MB, stored in `s3://telcox/gold/churn_risk_mart/`.

#### Step 3: Deliver and Enable Analysts
- **Output**: Two standardized datasets:
  - **`churn_fact`**: Granular, row-level data for deep dives (e.g., individual customer churn drivers).
  - **`churn_risk_mart`**: Aggregated, ready for dashboards (e.g., churn rates by region/month).
- **Access**: Analysts query via Snowflake or pull Parquet files into Power BI from S3.
- **Documentation**: Simple schema notes (e.g., “`call_drop_count`: # of dropped calls per customer”)—self-service ready, per Dan’s “data as a product” (14:04-14:46).

---

### Outcome: New Possibilities and Problem Resolution
#### Before Consultant
- **Process**: Analysts manually exported CSV chunks, joined in Excel (hours per report), producing conflicting churn theories.
- **Insight**: No clear view—e.g., “Is it price or network?” remained guesswork.
- **Time**: Weeks per analysis, delaying action.

#### After Consultant
1. **Uncover Churn Drivers**:
   - Analysts query `churn_risk_mart` in Power BI:
     - Finding: Regions with >50 call drops/month have 8% churn vs. 3% elsewhere—network quality is a top driver.
     - Viz: Bar chart of churn rate vs. call drops, filtered by region.
2. **Spot Risks**:
   - From `churn_fact`: Customers with low NPS (<5) and high plan costs (>$80) churn at 12%—price sensitivity compounds service issues.
     - Viz: Scatter plot of NPS vs. cost, color-coded by churn flag.
3. **New Vision**: 
   - Marketing proposes targeted retention (e.g., discounts for high-drop areas), ops prioritizes network upgrades—data-driven decisions emerge.
   - Power BI dashboard deployed in days, not weeks, showcasing your analyst skills (per your goal).

#### Impact
- **Resolution**: Churn root causes (network > price) clarified, reducing guesswork—5% quarterly loss now actionable.
- **Efficiency**: Analysis drops from weeks to hours—standard datasets remove ETL grunt work.
- **ROI**: TelcoX estimates $2M annual savings if churn drops 1%, justifying your $50K consulting fee.

---


