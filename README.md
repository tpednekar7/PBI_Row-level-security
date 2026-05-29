# Power BI Row Level Security (RLS) — Implementation Guide

## Overview

This guide walks through implementing Row Level Security (RLS) in Power BI Desktop using a sample sales dataset. RLS restricts data access so each user only sees the rows they are permitted to view — for example, a salesperson sees only their own sales, while a manager sees everything.

---

## What's in the Excel File

The workbook contains two sheets:

### Sheet 1 — SalesData (Fact Table, 10 rows)

| Column | Description |
|---|---|
| OrderID | Unique order identifier |
| SalespersonName | Name of the salesperson who made the sale |
| Region | Geographic region (North / South / East / West) |
| Product | Product name |
| Category | Product category (Electronics / Furniture) |
| SaleAmount | Sale value in dollars |
| OrderDate | Date the order was placed |
| CustomerName | Name of the customer |

### Sheet 2 — UserRoles (RLS Mapping Table)

| Column | Description |
|---|---|
| UserEmail | Login email of the Power BI user |
| SalespersonName | Maps the user to their salesperson name in SalesData |
| Region | Region the user belongs to |
| Role | Either `Salesperson` or `Manager` |

> This is the key table. It maps each person's email to what data they are allowed to see.

---

## Step-by-Step: Implementing RLS in Power BI Desktop

### STEP 1 — Load the Data

1. Open Power BI Desktop
2. Click **Get Data → Excel Workbook**
3. Browse to and select the downloaded Excel file
4. In the Navigator, select **both sheets** — SalesData and UserRoles
5. Click **Load**
6. Both tables will now appear in the Fields pane on the right

---

### STEP 2 — Create a Relationship Between the Two Tables

1. Click the **Model view** icon on the left sidebar (three connected boxes)
2. Drag **SalespersonName** from the `UserRoles` table onto **SalespersonName** in `SalesData`
3. A line appears between the two tables — that is your relationship
4. Double-click the line to verify the settings:
   - Cardinality: **Many to one (\*:1)**
   - Cross filter direction: **Both**
   - Active: **checked**
   - Apply security filter in both directions: **checked**
5. Click **OK**

> **Why this matters:** Power BI uses this relationship to filter SalesData based on which user is logged in. Without it, RLS filters on UserRoles will not flow through to SalesData and all rows will still be visible.

---

### STEP 3 — Define the Roles

1. Go to the **Modeling tab** in the top menu
2. Click **Manage Roles**
3. Click **Create** and name the role: `Salesperson`
4. Select the **UserRoles** table in the left panel
5. In the DAX filter box, enter:

```dax
[UserEmail] = USERPRINCIPALNAME()
```

> **What this does:** When someone opens the report, Power BI captures their login email via `USERPRINCIPALNAME()`, finds their matching row in UserRoles, and because of the relationship created in Step 2, automatically filters SalesData to show only their permitted rows.

6. Click **Save**
7. Click **Create** again and name the second role: `Manager`
8. Leave the DAX filter box **empty** — managers see all data
9. Click **Save**

---

### STEP 4 — Test Before Publishing

1. Go to the **Modeling tab** → click **View as**
2. Select the `Salesperson` role
3. In the "Other user" box, type: `alice@company.com`
4. Click **OK**

A yellow banner appears at the top: *"Now viewing as alice@company.com"*

Your visuals should now show **only Alice Johnson's rows** — OrderIDs 1001, 1003, and 1009.

| Test User | Expected Rows Visible |
|---|---|
| alice@company.com | 1001, 1003, 1009 |
| bob@company.com | 1002, 1006, 1010 |
| carlos@company.com | 1004, 1007 |
| diana@company.com | 1005, 1008 |
| Manager role | All 10 rows |

5. Click **Stop viewing** to return to normal view

---

### STEP 5 — Publish and Assign Users (Power BI Service)

Once published to [app.powerbi.com](https://app.powerbi.com):

1. Go to your workspace
2. Find your **dataset** → click the three dots (**...**) → **Security**
3. You will see the roles you created: `Salesperson` and `Manager`
4. Add real user emails under each role:
   - Under `Salesperson` → add alice@company.com, bob@company.com, etc.
   - Under `Manager` → add the manager's email
5. Click **Save**

From this point on, each user will only see their permitted data when they open the report.

---

## How It All Connects

```
User logs in
    → Power BI captures their email via USERPRINCIPALNAME()
    → Matches email to a row in UserRoles table
    → Relationship flows the filter into SalesData
    → User sees only their permitted rows
```

---

## Common Issues & Fixes

| Problem | Likely Cause | Fix |
|---|---|---|
| All rows still visible after View as | No active relationship between tables | Create relationship with Both cross-filter direction and security filter enabled |
| Relationship showing cardinality error | Hidden spaces or type mismatch in SalespersonName | Use Trim + Clean in Power Query on both tables |
| Table visual showing no rows | Sort-by-column conflict or measure error | Temporarily remove the measure to isolate whether it's a column or measure issue |
