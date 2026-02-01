# Row-Level Security (RLS) Configuration

## Overview
This document defines the security roles and DAX filter rules implemented in the Executive Financial Dashboard to ensure data governance and compliance (GDPR/SOC2).

## Security Roles

### 1. CEO (Chief Executive Officer)
**Access Level:** Full Organization
**DAX Filter:** None (Unrestricted access to all data)
```dax
// CEO sees everything - no filters applied
TRUE()
2. CFO (Chief Financial Officer)
Access Level: All Financial Data
DAX Filter:
dax
Copy
// CFO sees all financial data but restricted to Finance department costs
[Department] = "Finance" || [Department] = "Executive"
3. Regional VP (Sales)
Access Level: Regional Data Only
DAX Filter:
dax
Copy
// Regional VP sees only their region
[Region] = USERNAME()
Note: Assumes username matches region name (e.g., "North America")
4. Department Head
Access Level: Department Specific
DAX Filter:
dax
Copy
// Department heads see only their department
[Department] = LOOKUPVALUE(
    DimUser[Department],
    DimUser[Email],
    USERNAME()
)
5. External Auditor
Access Level: Read-Only, Aggregated Only
DAX Filter:
dax
Copy
// Auditors see aggregated data only, no individual transactions
[HierarchLevel] = "Summary"
Note: This role has additional Object-Level Security (OLS) preventing access to sensitive columns
Implementation Steps
Power BI Desktop:
Go to Modeling → Manage Roles
Create roles listed above
Add DAX filter expressions to appropriate tables
Power BI Service:
Publish report to workspace
Go to Dataset → Security
Add email addresses to respective roles
Enable "Test as role" before production
Dynamic Row-Level Security (Advanced):
For complex organizations, use bridge table approach:
dax
Copy
// Dynamic RLS using UserAttributes table
DimRegion[RegionID] IN 
SELECTCOLUMNS(
    FILTER(
        UserAccess,
        UserAccess[Email] = USERNAME()
    ),
    "RegionID", UserAccess[RegionID]
)
Security Matrix
Table
Copy
Role	Revenue Data	Cost Data	Customer PII	Forecasts	Real-time
CEO	✅ Full	✅ Full	✅ Full	✅ Full	✅ Full
CFO	✅ Full	✅ Full	❌ Masked	✅ Full	✅ Full
Regional VP	✅ Regional	✅ Regional	❌ No Access	✅ Regional	✅ Regional
Dept Head	✅ Dept	✅ Dept	❌ No Access	✅ Dept	❌ No
Auditor	✅ Summary	✅ Summary	❌ No Access	❌ No	❌ No
Testing RLS
Always test RLS before publishing:
Power BI Desktop: View as → Select Role
Power BI Service: Dataset → Security → Test as role
DAX Check: Verify row counts match expected results
Compliance Notes
GDPR: European users only see EU data (filtered by [Region] = "Europe")
SOX Compliance: Financial data access logged via Power BI Activity Log
Data Masking: SSN, Account Numbers masked via OLS (Object-Level Security)
Audit Trail: All access tracked in Azure Log Analytics
