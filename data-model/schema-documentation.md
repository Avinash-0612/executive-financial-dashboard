# Data Model Schema Documentation

## Model Architecture

**Star Schema with Snowflake Extensions**

## Entity Relationship Diagram

```mermaid
erDiagram
    FACT_SALES ||--o{ DIM_DATE : sold_on
    FACT_SALES ||--o{ DIM_CUSTOMER : purchased_by
    FACT_SALES ||--o{ DIM_PRODUCT : contains
    FACT_SALES ||--o{ DIM_EMPLOYEE : sold_by
    FACT_SALES ||--o{ DIM_REGION : in_region
    DIM_PRODUCT }o--|| DIM_CATEGORY : belongs_to
    DIM_CUSTOMER }o--|| DIM_GEOGRAPHY : located_in
Fact Tables

Table
Copy
Column	Data Type	Description	Relationship
SalesKey	BIGINT	Surrogate key	Primary Key
DateKey	INT	FK to DimDate	Many-to-One
CustomerKey	INT	FK to DimCustomer	Many-to-One
ProductKey	INT	FK to DimProduct	Many-to-One
EmployeeKey	INT	FK to DimEmployee	Many-to-One
RegionKey	INT	FK to DimRegion	Many-to-One
SalesOrderNumber	VARCHAR	Degenerate Dim	-
Quantity	INT	Units sold	-
UnitPrice	MONEY	Price per unit	-
UnitCost	MONEY	Cost per unit	-
DiscountAmount	MONEY	Discount given	-
SalesAmount	Calculated	Qty × Price	-
ProfitAmount	Calculated	(Price-Cost) × Qty	-
Storage: Import Mode (for performance)
Partitioning: Monthly partitions on DateKey
Incremental Refresh: Enabled (last 5 years)
Dimension Tables
DimDate (Type 0 - Fixed)
Table
Copy
Column	Data Type	Description
DateKey	INT	YYYYMMDD format
FullDate	DATE	Calendar date
DayOfWeek	TINYINT	1-7
DayName	VARCHAR	Monday, Tuesday...
MonthNumber	TINYINT	1-12
MonthName	VARCHAR	January, February...
Quarter	TINYINT	Q1, Q2, Q3, Q4
Year	SMALLINT	Calendar year
FiscalQuarter	TINYINT	Fiscal quarter
IsWeekend	BIT	1 if Sat/Sun
IsHoliday	BIT	1 if US Holiday
Special: Marked as Date Table in Power BI
DimCustomer (Type 2 - Slowly Changing Dimension)
Tracks historical changes to customer attributes.
Table
Copy
Column	Data Type	Description
CustomerKey	INT	Surrogate Key (PK)
CustomerID	INT	Natural Key (Business Key)
CustomerName	VARCHAR	Full name
Email	VARCHAR	Contact email
Segment	VARCHAR	Enterprise/SMB/Consumer
StartDate	DATE	SCD Effective Date
EndDate	DATE	SCD Expiration Date (NULL = Current)
IsCurrent	BIT	1 = Current record
SCD Logic: When customer segment changes, new row created with updated StartDate.
DimProduct (Type 1 - Overwrite)
Table
Copy
Column	Data Type	Description
ProductKey	INT	Surrogate Key
ProductID	VARCHAR	SKU/Natural Key
ProductName	VARCHAR	Description
CategoryKey	INT	FK to DimCategory
Subcategory	VARCHAR	Product sub-type
Brand	VARCHAR	Manufacturer
UnitCost	MONEY	Standard cost
UnitPrice	MONEY	List price
DimRegion (For RLS)
Table
Copy
Column	Data Type	Description
RegionKey	INT	Surrogate Key
RegionName	VARCHAR	Americas, EMEA, APAC
Country	VARCHAR	Country name
IsActive	BIT	Active in reporting
Relationships
FactSales → DimDate
Cardinality: Many-to-One
Cross-filter: Single
Active: Yes
FactSales → DimCustomer
Cardinality: Many-to-One
Cross-filter: Both (for customer insights)
FactSales → DimProduct
Cardinality: Many-to-One
Cross-filter: Single
Calculated Tables
Date Table (DAX Generated)
dax
Copy
DimDate = 
ADDCOLUMNS(
    CALENDAR(DATE(2020,1,1), DATE(2025,12,31)),
    "Year", YEAR([Date]),
    "Month", MONTH([Date]),
    "Quarter", QUARTER([Date])
)
Measure Table (Organizational)
dax
Copy
MeasureTable = {
    ("Total Revenue", NAMEOF('Measures'[Total Revenue]), 0),
    ("Profit Margin", NAMEOF('Measures'[Profit Margin]), 1)
}
Aggregations
For large datasets (>100M rows), set up aggregation tables:
AggSales_Monthly:
Grouped by: Year, Month, Category, Region
Pre-aggregated: Sum(Sales), Sum(Quantity)
Storage: Import Mode
Refresh: Daily
Performance Optimization
Indices: Columnstore indexes on high-cardinality columns
Sorting: DateKey sorted descending (recent data first)
Compression: Dictionary encoding on text columns
Relationships: 1-to-many only, no many-to-many without bridge
Data Sources
Table
Copy
Source	Connection Type	Refresh Mode
Azure Synapse	DirectQuery	Real-time
Excel Budgets	Import	Daily
REST API (Markets)	Import	Every 15 min
