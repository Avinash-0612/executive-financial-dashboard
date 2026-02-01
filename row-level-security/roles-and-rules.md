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

**### 2. CFO (Chief Financial Officer)**
**Access Level:** All Financial Data
**DAX Filter:**
```dax
// CFO sees all financial data but restricted to Finance department costs
[Department] = "Finance" || [Department] = "Executive"

**### 3. Regional VP (Sales)**
**Access Level:** Regional Data Only
**DAX Filter:**
```dax
// Regional VP sees only their region
[Region] = USERNAME()
