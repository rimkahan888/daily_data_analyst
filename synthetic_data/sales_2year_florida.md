To simulate inventory optimization for your sports apparel retailer case study, the **fact_sales table** should have enough rows to reflect realistic trends without being unwieldy. Here’s a structured approach:

---

### **Recommended Fact Table Size**  
| **Component**       | **Value**              | **Calculation**                  | **Total Rows** |  
|----------------------|------------------------|-----------------------------------|----------------|  
| **Time**             | 2 years (730 days)     | Daily sales data for 2 years      | 730 days       |  
| **Stores**           | 10 stores in Florida   | 10 physical locations             | 10 stores      |  
| **Products**         | 500 SKUs               | 50 categories × 10 brands         | 500 products   |  
| **Customers**        | 5,000 unique customers | Repeat buyers + one-time shoppers | 5,000 customers|  
| **Transactions**     | 30 transactions/store/day | 30 × 10 stores × 730 days      | **219,000 rows** |  

**Total Rows in `fact_sales`**: ~220,000 rows (enough to model seasonality, regional demand, and inventory turnover).

---

### **Why This Works**  
1. **Realistic Seasonality**:  
   - 2 years of data captures summer/winter cycles (e.g., swimwear spikes in June).  
   - Example: Miami Beach store sells 2x more swimwear in summer than Orlando.  

2. **Regional Variance**:  
   - Miami stores: 60% swimwear, 20% athleisure.  
   - Orlando stores: 40% golf apparel, 30% running shoes.  

3. **Deadstock Identification**:  
   - Include 50 low-turnover items (e.g., winter jackets) with 1-2 sales/month.  

---

### **Sample Data Generation Code (Python)**  
Use `pandas` and `faker` to create synthetic data:  
```python
import pandas as pd
from faker import Faker
import numpy as np

# Initialize
fake = Faker()
np.random.seed(42)

# Generate 220K rows
dates = pd.date_range(start='2022-01-01', end='2023-12-31', freq='D')
stores = [f'Store_{i}' for i in range(1, 11)]
products = [f'Product_{i}' for i in range(1, 501)]
customers = [fake.uuid4() for _ in range(5000)]

data = []
for date in dates:
    for store in stores:
        for _ in range(30):  # 30 transactions/store/day
            product = np.random.choice(products)
            customer = np.random.choice(customers)
            units_sold = np.random.poisson(lam=3)  # Avg 3 units/transaction
            revenue = units_sold * np.random.uniform(20, 100)  # $20-$100/item
            data.append([date, store, product, customer, units_sold, revenue])

fact_sales = pd.DataFrame(data, columns=['date', 'store', 'product', 'customer', 'units_sold', 'revenue'])
```

---

### **Key Customizations for Realism**  
1. **Seasonal Adjustments**:  
   ```python
   # Boost swimwear sales in June-August for Miami stores
   mask = (fact_sales['date'].dt.month.isin([6, 7, 8])) & (fact_sales['store'].str.contains('Miami'))
   fact_sales.loc[mask, 'units_sold'] = fact_sales.loc[mask, 'units_sold'] * 2
   ```

2. **Regional Preferences**:  
   ```python
   # Orlando stores sell more golf apparel
   golf_products = [f'Product_{i}' for i in range(300, 350)]
   mask = (fact_sales['store'].str.contains('Orlando')) & (fact_sales['product'].isin(golf_products))
   fact_sales.loc[mask, 'units_sold'] = fact_sales.loc[mask, 'units_sold'] * 1.5
   ```

3. **Low-Turnover Items**:  
   ```python
   # Add winter jackets with minimal sales
   winter_products = [f'Product_{i}' for i in range(450, 500)]
   mask = fact_sales['product'].isin(winter_products)
   fact_sales.loc[mask, 'units_sold'] = np.random.choice([0, 1], size=sum(mask), p=[0.95, 0.05])
   ```

---

### **Analysis-Ready Data**  
- **Inventory Turnover**:  
  ```sql
  -- Products with lowest turnover
  SELECT product, SUM(units_sold) / COUNT(DISTINCT date) AS monthly_turnover
  FROM fact_sales
  GROUP BY product
  ORDER BY monthly_turnover ASC
  LIMIT 10;
  ```
  **Output**: Winter jackets (`monthly_turnover = 0.1`) → Liquidate stock.  

- **Regional Demand**:  
  ```sql
  -- Top categories in Miami vs. Orlando
  SELECT 
    store,
    CASE WHEN product BETWEEN 'Product_1' AND 'Product_100' THEN 'Swimwear'
         WHEN product BETWEEN 'Product_101' AND 'Product_200' THEN 'Athleisure'
         ELSE 'Other' END AS category,
    SUM(units_sold) AS total_units
  FROM fact_sales
  GROUP BY store, category;
  ```
  **Insight**: Miami sells 3x more swimwear than Orlando.

---

### **Tools for Simulation**  
- **SQL**: Use DuckDB or PostgreSQL for querying.  
- **Python**: `pandas` for data generation, `prophet` for forecasting.  
- **Visualization**: Plotly or Tableau for dashboards.  

This setup balances realism with practicality, ensuring you can showcase **inventory optimization skills** in job interviews! 🏃♂️🛒
