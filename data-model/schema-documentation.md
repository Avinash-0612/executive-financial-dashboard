# Data Model Schema Documentation

## Model Architecture
**Star Schema with Snowflake Extensions**

---

## Entity Relationship Diagram

```markdown
```mermaid
erDiagram
FACT_SALES ||--o{ DIM_DATE : sold_on
FACT_SALES ||--o{ DIM_CUSTOMER : purchased_by
FACT_SALES ||--o{ DIM_PRODUCT : contains
FACT_SALES ||--o{ DIM_EMPLOYEE : sold_by
FACT_SALES ||--o{ DIM_REGION : in_region
DIM_PRODUCT }o--|| DIM_CATEGORY : belongs_to
DIM_CUSTOMER }o--|| DIM_GEOGRAPHY : located_in

## Fact Table

### FactSales

Granularity: Individual Transaction  
Storage Mode: Import  
Partitioning: Monthly partitions on DateKey  
Incremental Refresh: Enabled (last 5 years)

| Column | Data Type | Description |
|-------|-----------|-------------|
...
