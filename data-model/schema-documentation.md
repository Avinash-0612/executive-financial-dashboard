# Data Model Schema Documentation


## Model Architecture
**Star Schema with Snowflake Extensions**


---


## Entity Relationship Diagram



erDiagram
    FACT_SALES ||--o{ DIM_DATE : sold_on
    FACT_SALES ||--o{ DIM_CUSTOMER : purchased_by
    FACT_SALES ||--o{ DIM_PRODUCT : contains
    FACT_SALES ||--o{ DIM_EMPLOYEE : sold_by
    FACT_SALES ||--o{ DIM_REGION : in_region
    DIM_PRODUCT }o--|| DIM_CATEGORY : belongs_to
    DIM_CUSTOMER }o--|| DIM_GEOGRAPHY : located_in
Fact Table
FactSales

Granularity: Individual Transaction

Column	Data Type	Description	Relationship
SalesKey	BIGINT	Surrogate key	Primary Key
DateKey	INT	Foreign key to DimDate	Many-to-One
CustomerKey	INT	Foreign key to DimCustomer	Many-to-One
ProductKey	INT	Foreign key to DimProduct	Many-to-One
EmployeeKey	INT	Foreign key to DimEmployee	Many-to-One
RegionKey	INT	Foreign key to DimRegion	Many-to-One
SalesOrderNumber	VARCHAR	Degenerate dimension	—
Quantity	INT	Units sold	—
UnitPrice	MONEY	Price per unit	—
UnitCost	MONEY	Cost per unit	—
DiscountAmount	MONEY	Discount applied	—
SalesAmount	Calculated	Quantity × UnitPrice	—
ProfitAmount	Calculated	(UnitPrice − UnitCost) × Quantity	—
Dimension Tables
DimDate

Type: Type 0 (Fixed / Static)

Column	Data Type	Description
DateKey	INT	YYYYMMDD format
FullDate	DATE	Calendar date
DayOfWeek	TINYINT	1–7
DayName	VARCHAR	Monday, Tuesday, etc.
MonthNumber	TINYINT	1–12
MonthName	VARCHAR	January, February, etc.
Quarter	TINYINT	Q1–Q4
Year	SMALLINT	Calendar year
FiscalQuarter	TINYINT	Fiscal quarter
IsWeekend	BIT	1 if Saturday or Sunday
IsHoliday	BIT	1 if U.S. holiday
DimCustomer

Type: Type 2 (Slowly Changing Dimension)
Tracks historical changes to customer attributes.

Column	Data Type	Description
CustomerKey	INT	Surrogate key (Primary Key)
CustomerID	INT	Natural/business key
CustomerName	VARCHAR	Full customer name
Email	VARCHAR	Contact email
Segment	VARCHAR	Enterprise / SMB / Consumer
StartDate	DATE	Record effective date
EndDate	DATE	Record expiration date (NULL = current)
IsCurrent	BIT	1 = current active record
DimProduct

Type: Type 1 (Overwrite)

Column	Data Type	Description
ProductKey	INT	Surrogate key
ProductID	VARCHAR	SKU / natural key
ProductName	VARCHAR	Product description
CategoryKey	INT	Foreign key to category
Subcategory	VARCHAR	Product sub-type
Brand	VARCHAR	Manufacturer
UnitCost	MONEY	Standard cost
UnitPrice	MONEY	List price
DimRegion

Purpose: Row-Level Security (RLS)

Column	Data Type	Description
RegionKey	INT	Surrogate key
RegionName	VARCHAR	Americas, EMEA, APAC
Country	VARCHAR	Country name
IsActive	BIT	Active for reporting
Relationships

FactSales → DimDate

Cardinality: Many-to-One

Cross-filter direction: Single

Active: Yes

FactSales → DimCustomer

Cardinality: Many-to-One

Cross-filter direction: Both (customer analytics)

FactSales → DimProduct

Cardinality: Many-to-One

Cross-filter direction: Single

Calculated Tables
Date Table (DAX Generated)
DimDate =
ADDCOLUMNS(
    CALENDAR ( DATE(2020,1,1), DATE(2025,12,31) ),
    "Year", YEAR([Date]),
    "Month", MONTH([Date]),
    "Quarter", QUARTER([Date])
)
Measure Table (Organizational)
MeasureTable = {
    ("Total Revenue", NAMEOF('Measures'[Total Revenue]), 0),
    ("Profit Margin", NAMEOF('Measures'[Profit Margin]), 1)
}
Aggregations

For large datasets (>100M rows), pre-aggregation is recommended.

AggSales_Monthly

Grouped by: Year, Month, Category, Region

Pre-aggregated metrics:

SUM(SalesAmount)

SUM(Quantity)

Storage mode: Import

Refresh frequency: Daily

Performance Optimization

Columnstore indexes on high-cardinality fact columns

Sort FactSales by DateKey (descending)

Dictionary encoding on text-based dimensions

Enforce 1-to-many relationships

Avoid many-to-many relationships unless using bridge tables

Data Sources
Source	Connection Type	Refresh Mode
Azure Synapse	DirectQuery	Real-time
Excel Budgets	Import	Daily
REST API (Markets)	Import	Every 15 min
