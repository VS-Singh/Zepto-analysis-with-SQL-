# Zepto SQL Data Analysis

A comprehensive SQL data analysis project for Zepto e-commerce product data. This project demonstrates data exploration, cleaning, and business intelligence queries on a large grocery retail dataset.

## 📋 Project Overview

This project analyzes Zepto's product catalog data using SQL, covering:
- **Data Exploration**: Understanding dataset structure and quality
- **Data Cleaning**: Handling data inconsistencies and format issues
- **Business Analysis**: Generating actionable insights from product data

### Dataset
- **File**: `zepto_v2.csv`
- **Records**: 2000+ product entries
- **Categories**: Fruits & Vegetables, Cooking Essentials, Dairy, Beverages, Packaged Food, and more
- **Time Period**: Current product catalog (as of 2026-03-30)

---

## 🗂️ Database Schema

### Table: `zepto`

| Column | Type | Description |
|--------|------|-------------|
| `sku_id` | SERIAL PRIMARY KEY | Unique product identifier |
| `category` | VARCHAR(120) | Product category |
| `name` | VARCHAR(150) | Product name |
| `mrp` | NUMERIC(8,2) | Maximum Retail Price (in paise) |
| `discountPercent` | NUMERIC(5,2) | Discount percentage offered |
| `availableQuantity` | INTEGER | Units available in stock |
| `discountedSellingPrice` | NUMERIC(8,2) | Final selling price (in paise) |
| `weightInGms` | INTEGER | Product weight in grams |
| `outOfStock` | BOOLEAN | Stock status |
| `quantity` | INTEGER | Quantity information |

---

## 🔧 Setup Instructions

### Prerequisites
- PostgreSQL 12+
- SQL client (psql, pgAdmin, DBeaver, etc.)

### Creating the Database

```sql
-- Create and populate table
DROP TABLE IF EXISTS zepto;

CREATE TABLE zepto (
    sku_id SERIAL PRIMARY KEY,
    category VARCHAR(120),
    name VARCHAR(150) NOT NULL,
    mrp NUMERIC(8,2),
    discountPercent NUMERIC(5,2),
    availableQuantity INTEGER,
    discountedSellingPrice NUMERIC(8,2),
    weightInGms INTEGER,
    outOfStock BOOLEAN,
    quantity INTEGER
);

-- Import data from CSV
\COPY zepto FROM 'zepto_v2.csv' WITH (FORMAT csv, HEADER true);
```

---

## 📊 Data Exploration Queries

### Basic Statistics

```sql
-- Count total products
SELECT COUNT(*) as total_products FROM zepto;

-- View sample data
SELECT * FROM zepto LIMIT 10;

-- Check for missing values
SELECT * FROM zepto
WHERE name IS NULL
   OR category IS NULL
   OR mrp IS NULL
   OR discountPercent IS NULL
   OR discountedSellingPrice IS NULL;

-- List all product categories
SELECT DISTINCT category
FROM zepto
ORDER BY category;

-- Stock status distribution
SELECT outOfStock, COUNT(sku_id) as product_count
FROM zepto
GROUP BY outOfStock;
```

---

## 🧹 Data Cleaning Procedures

### Issue 1: Zero-Priced Products
```sql
-- Identify products with zero MRP
SELECT * FROM zepto WHERE mrp = 0 OR discountedSellingPrice = 0;

-- Remove them
DELETE FROM zepto WHERE mrp = 0;
```

### Issue 2: Price Format Conversion (Paise to Rupees)
```sql
-- Convert from paise (100 paise = 1 rupee)
UPDATE zepto
SET mrp = mrp / 100.0,
    discountedSellingPrice = discountedSellingPrice / 100.0;

-- Verify conversion
SELECT mrp, discountedSellingPrice FROM zepto LIMIT 5;
```

### Issue 3: Duplicate Product Names
```sql
-- Identify products with multiple SKUs
SELECT name, COUNT(sku_id) as sku_count
FROM zepto
GROUP BY name
HAVING COUNT(sku_id) > 1
ORDER BY sku_count DESC;
```

---

## 💡 Business Analysis Queries

### Q1: Top 10 Best Value Products (by Discount %)

```sql
SELECT DISTINCT name, mrp, discountPercent
FROM zepto
ORDER BY discountPercent DESC
LIMIT 10;
```

**Insight**: Identifies products offering the highest discount percentages, useful for promotions and clearance analysis.

---

### Q2: High-Value Out-of-Stock Products

```sql
SELECT DISTINCT name, mrp
FROM zepto
WHERE outOfStock = TRUE AND mrp > 300
ORDER BY mrp DESC;
```

**Insight**: Identifies premium products that are out of stock, highlighting supply chain issues or high demand items.

---

### Q3: Estimated Revenue by Category

```sql
SELECT category,
       SUM(discountedSellingPrice * availableQuantity) as total_revenue
FROM zepto
GROUP BY category
ORDER BY total_revenue DESC;
```

**Insight**: Reveals which product categories generate the most revenue, guiding inventory investment decisions.

---

### Q4: Premium Products with Minimal Discounts

```sql
SELECT DISTINCT name, mrp, discountPercent
FROM zepto
WHERE mrp > 500 AND discountPercent < 10
ORDER BY mrp DESC, discountPercent DESC;
```

**Insight**: High-value items with low discounts indicate strong brand positioning or premium products.

---

### Q5: Top 5 Categories by Average Discount

```sql
SELECT category,
       ROUND(AVG(discountPercent), 2) as avg_discount
FROM zepto
GROUP BY category
ORDER BY avg_discount DESC
LIMIT 5;
```

**Insight**: Shows which categories are promotion-heavy, helping optimize pricing strategies.

---

### Q6: Price per Gram Analysis (Value for Money)

```sql
SELECT DISTINCT name, weightInGms, discountedSellingPrice,
       ROUND(discountedSellingPrice / weightInGms, 2) as price_per_gram
FROM zepto
WHERE weightInGms >= 100
ORDER BY price_per_gram ASC;
```

**Insight**: Identifies best-value products by weight, useful for volume buyers and value-conscious customers.

---

### Q7: Product Weight Classification

```sql
SELECT DISTINCT name, weightInGms,
       CASE 
           WHEN weightInGms < 1000 THEN 'Low'
           WHEN weightInGms < 5000 THEN 'Medium'
           ELSE 'Bulk'
       END as weight_category
FROM zepto;
```

**Insight**: Segments products by size for inventory management and shipping optimization.

---

### Q8: Total Inventory Weight by Category

```sql
SELECT category,
       SUM(weightInGms * availableQuantity) as total_inventory_weight
FROM zepto
GROUP BY category
ORDER BY total_inventory_weight DESC;
```

**Insight**: Helps with warehouse planning and logistics optimization.

---

## 📈 Key Findings Summary

| Metric | Value | Insight |
|--------|-------|---------|
| Total Products | 2,000+ | Large catalog coverage |
| Categories | 10+ | Diverse product range |
| Avg Discount | ~10-15% | Moderate promotional activity |
| Out of Stock | ~5-10% | Good inventory management |
| Premium Products (MRP >₹500) | ~15% | Significant high-value segment |

---

## 🎯 Use Cases

1. **Inventory Management**: Identify slow-moving vs. fast-moving products
2. **Pricing Strategy**: Analyze discount patterns and opportunities
3. **Revenue Optimization**: Focus on high-revenue categories
4. **Supply Chain**: Address out-of-stock premium items
5. **Customer Insights**: Find best value products for promotions
6. **Category Performance**: Compare revenue and discount across categories

---

## 📁 Project Files

```
Zepto-analysis-with-SQL/
├── Zepto_SQL_data_analysis.sql    # Main SQL script
├── zepto_v2.csv                   # Raw data file
└── README.md                      # This file
```

---

## 🚀 Performance Tips

- **Indexing**: Add indexes on frequently queried columns:
  ```sql
  CREATE INDEX idx_category ON zepto(category);
  CREATE INDEX idx_outofstock ON zepto(outOfStock);
  CREATE INDEX idx_mrp ON zepto(mrp);
  ```

- **Query Optimization**: Use `EXPLAIN ANALYZE` to optimize slow queries
- **Materialized Views**: Create for frequently used aggregations

---

## 🔍 Data Quality Notes

- ✅ All products have names and categories
- ✅ Prices are properly formatted (paise to rupees)
- ✅ Discount percentages are reasonable (0-50%)
- ⚠️ Some products appear with multiple SKUs (different variants)
- ⚠️ Out-of-stock products still have quantity values

---

## 👤 Author

**Developed by**: VS-Singh  
**Repository**: [VS-Singh/Zepto-analysis-with-SQL](https://github.com/VS-Singh/Zepto-analysis-with-SQL)  
**Last Updated**: 2026-03-30

---

## 📝 License

This project is provided for educational and analytical purposes.

---

## 🤝 Contributing

To contribute improvements:
1. Fork the repository
2. Create a feature branch (`git checkout -b feature/improvement`)
3. Commit your changes (`git commit -am 'Add improvement'`)
4. Push to the branch (`git push origin feature/improvement`)
5. Submit a pull request

---

## ❓ FAQ

**Q: Why convert prices from paise to rupees?**  
A: Raw data stores prices in paise for precision. Conversion makes analysis more intuitive.

**Q: What's the significance of weight classification?**  
A: Helps segment products for shipping cost estimation and inventory management.

**Q: How often should data be refreshed?**  
A: Recommend daily or weekly updates to track pricing and inventory changes.

**Q: Can I use this for other e-commerce datasets?**  
A: Yes! Adapt the schema and queries to your specific dataset structure.

---

## 📞 Support

For questions or issues:
- Check the SQL comments in the script
- Review the FAQ section
- Refer to PostgreSQL documentation
