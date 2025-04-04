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

