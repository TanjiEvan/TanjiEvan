# Building a Fabric Data Agent from Power BI Semantic Model
## Complete Internal Playbook v1.0

**Model:** Better Decisions Inc. - Accounts Receivable  
**Scope:** Semantic models only (no lakehouses/warehouses)

---

## Table of Contents

1. [Prerequisites](#prerequisites)
2. [Step 1: Verify Star Schema Model](#step-1-verify-star-schema-model)
3. [Step 2: Select Fields to Expose to AI](#step-2-select-fields-to-expose-to-ai)
4. [Step 3: Add AI Instructions](#step-3-add-ai-instructions)
5. [Step 4: Add Verified Answers](#step-4-add-verified-answers)
6. [Step 5: Add Descriptions to All Fields](#step-5-add-descriptions-to-all-fields)
7. [Step 6: Publish the Semantic Model](#step-6-publish-the-semantic-model)
8. [Step 7: Configure in Power BI Service](#step-7-configure-in-power-bi-service)
9. [Step 8: Test with Copilot](#step-8-test-with-copilot)
10. [Step 9: Create Fabric Data Agent](#step-9-create-fabric-data-agent)
11. [Step 10: Configure Permissions](#step-10-configure-permissions)
12. [Step 11: Publish Data Agent](#step-11-publish-data-agent)
13. [Step 12: Access from M365 Copilot](#step-12-access-from-m365-copilot)
14. [Step 13: Access from Excel](#step-13-access-from-excel)
15. [Step 14: Configure Copilot Studio](#step-14-configure-copilot-studio)
16. [Step 15: Access from Teams](#step-15-access-from-teams)

---

## Prerequisites

*[To be completed]*

---

## Step 1: Start with a Power BI-Optimized Semantic Model

### Overview

We begin with an existing Power BI report and semantic model that follows best practices. This is the **Better Decisions AR Analytics** — a working dashboard that we will optimize for Fabric Data Agent.

---

### 1.1 The Dashboard

Open your Power BI file. The report displays key AR metrics:

<img width="1442" height="806" alt="image" src="https://github.com/user-attachments/assets/052125de-7f48-4cad-8213-4f8fe99a163f" />


**Key Metrics Displayed:**

| Metric | Value | Description |
|--------|-------|-------------|
| Open Balance | $52.00M | Total unpaid invoices |
| Current Balance | $20.30M | Invoices not yet due |
| Overdue Balance | $33.73M | Invoices past due date |
| Overdue % | 64.87% | Percentage of balance that is overdue |

**Visuals Include:**
- Total Open Balance by Aging Period (bar chart)
- Top 5 Overdue by Resellers (bar chart)
- Customer details table with overdue breakdown

---

### 1.2 View the Semantic Model

1. Click the **Model view** icon in the left sidebar

<img width="1533" height="954" alt="1  Going to Data Model" src="https://github.com/user-attachments/assets/ee7f8a1f-aed3-4cfe-a3c1-b294483c168e" />


---

### 1.3 Verify Star Schema Structure

A **star schema** is the ideal structure for Fabric Data Agent. It consists of:
- **Fact tables** at the center containing transactional data
- **Dimension tables** surrounding them with descriptive attributes
- **Dimensions connect directly to fact tables** (no dimension-to-dimension chains)

<img width="1112" height="775" alt="2  Star Schema" src="https://github.com/user-attachments/assets/4fb01df0-1530-4a6c-980e-db7e9e767661" />


**Our Model:**

The **Better Decisions AR Analytics** semantic model follows star schema design with:

| Table | Type | Purpose |
|-------|------|---------|
| fact_AR Invoices | Fact | Invoice transactions with amounts and dates |
| dim_Customers AR | Dimension | Customer attributes and contact details |
| dim_Date | Dimension | Calendar dates for time-based analysis |
| dim_Aging Bucket | Dimension | Aging period categories |

---

### 1.4 Verify Relationships

1. Click **Manage relationships** in the Home tab

<img width="1397" height="935" alt="3  Manage Relations" src="https://github.com/user-attachments/assets/91ebd8e2-099f-48f7-a6fc-a913c46763fc" />


2. Verify all relationships show:
   - **Many-to-One** cardinality (`*:1`)
   - **Single** cross-filter direction

<img width="846" height="740" alt="4  The Relationships" src="https://github.com/user-attachments/assets/d58bae4c-4da3-45cd-86e2-b65fb66e326e" />


All relationships in our model are configured correctly as Many-to-One with Single direction filtering.

---

### ✓ Verification

For optimal Fabric Data Agent performance, we follow these star schema principles:

- Fact table at center with dimension tables connected directly
- All relationships are Many-to-One (*:1)
- All relationships use Single direction filtering
- No dimension-to-dimension relationships
- No bidirectional filtering

The model is ready for AI optimization.

---

## Step 2: Select Fields to Expose to AI

### Overview

The **Prep data for AI** feature allows you to configure which parts of your semantic model the AI can access. This includes selecting fields, adding descriptions, creating verified answers, and providing AI instructions.

---

### 2.1 Open Prep data for AI

1. Click **Prep data for AI** in the Home tab (Copilot section)

<img width="1530" height="961" alt="5  Prep Data for AI" src="https://github.com/user-attachments/assets/0c351c12-a46d-4b6b-9b09-4e1b6e2a96ea" />


The panel opens with three configuration options:

| Option | Purpose |
|--------|---------|
| **Simplify the data schema** | Select which tables and fields the AI can access |
| **Verified answers** | Create predefined responses for common business questions |
| **Add AI instructions** | Provide context about industry terms, business rules, and data usage |

---

### 2.2 Simplify the Data Schema

1. Click **Simplify the data schema** from the left sidebar

<img width="1437" height="793" alt="6  SELECT FIELD" src="https://github.com/user-attachments/assets/328b98aa-af24-4a6c-a395-f8324c398df2" />


2. The field selection interface displays all tables in your model

<img width="1432" height="790" alt="7  Simplify the data option" src="https://github.com/user-attachments/assets/28c5de1d-99fa-4729-a190-c58744f226fc" />


By default, all tables and columns are selected. The goal is to **deselect fields that could confuse the AI** or are not relevant for analysis.

---

### 2.3 Field Selection Guidelines

**Why Deselect Fields?**

- Reduces ambiguity for the AI when generating queries
- Improves response accuracy by limiting scope
- Removes duplicate or redundant columns
- Hides internal keys and technical fields

---

**Tables to Deselect:**

| Table | Reason |
|-------|--------|
| Reseller | Hidden support table — data already merged into dim_Customers AR |
| ResellerCompany | Hidden support table — data already merged into dim_Customers AR |

---

**Columns to Deselect in dim_Customers AR:**

The dim_Customers AR table contains duplicate columns from the merge operation. Deselect the redundant ones:

| Deselect | Keep | Reason |
|----------|------|--------|
| FaxNumber | Fax Number | Duplicate - different naming |
| PhoneNumber | Phone Number | Duplicate - different naming |
| WebsiteURL | Website URL | Duplicate - different naming |
| ResellerName | Reseller Name | Duplicate - different naming |
| ResellerCompanyID | Reseller Company ID | Duplicate - different naming |
| PostalCityID | Postal City ID | Duplicate - different naming |
| PostalPostalCode | Postal ZIP Code | Duplicate - different naming |
| PostalAddressLine1 | Postal Address Line 1 | Duplicate - different naming |
| PostalAddressLine2 | Postal Address Line 2 | Duplicate - different naming |
| DeliveryAddressLine1 | Delivery Address Line 1 | Duplicate - different naming |
| DeliveryAddressLine2 | Delivery Address Line 2 | Duplicate - different naming |
| DeliveryPostalCode | Delivery Postal Code | Duplicate - different naming |

---

**Columns to Deselect in fact_AR Invoices:**

| Column | Reason |
|--------|--------|
| Customer Key | Internal key — use Customer Display Name instead |
| Salesperson ID | Internal key — not relevant for AR analysis |

---

### 2.4 Apply Changes

After deselecting the tables and columns listed above, click **Apply** to save the schema configuration.

<img width="1421" height="785" alt="8  Apply Changes ( Field Selection )" src="https://github.com/user-attachments/assets/5f6b6c4f-d1ad-4120-a886-34e9fb025441" />


The AI will now only have access to the selected fields, improving response accuracy.

---

## Step 3: Add AI Instructions

### Overview

AI Instructions provide context to help Copilot understand your data, business terminology, and how to respond to queries. The first set of instructions belongs in the semantic model — not just in the agent.

---

### 3.1 Two Places for Instructions

| Location | Purpose |
|----------|---------|
| **Model-level instructions** | General guidance in the main AI instructions panel |
| **Field-level instructions** | Specific context within each selected field (via descriptions) |

---

### 3.2 Open AI Instructions

1. Click **Add AI instructions** from the left sidebar

<img width="1432" height="791" alt="9  Add AI Instructions" src="https://github.com/user-attachments/assets/aab625bb-eb7e-4381-882e-47bc4b9bbd1c" />


---

### 3.3 Add Instructions and Apply

The editor accepts plain text or Markdown format (up to 10,000 characters). Enter your instructions and click **Apply**.

<img width="1430" height="790" alt="10  Added Instructions" src="https://github.com/user-attachments/assets/9d6c9121-5300-4984-9320-5e3254ac9791" />


---

### 3.4 Instruction Focus Areas

**Time Intelligence**
- How to interpret "current," "recent," "latest" queries
- Handling fiscal vs calendar periods
- Date-based filtering logic

**Variance Calculations**
- Year-over-Year (YoY) comparisons
- Month-over-Month (MoM) analysis
- Budget vs Actual handling

**Business Definitions**
- Company-specific terms the AI wouldn't know
- Custom metric definitions
- Business rules and thresholds

---

### 3.5 Sample AI Instructions

Below is a sample AI instruction set for the AR model:

```
## Time Intelligence

When users ask about "current" data:
- First determine the latest available date in the semantic model
- Base analysis on actual data timeframe, not current real-world date
- Always specify the "as of" date in responses

Query translations:
- "Current receivables" = Latest available receivables position
- "This month's collections" = Most recent complete month in data
- "Year-to-date" = January through latest available month

## Data Model Structure

Primary Tables:
- fact_AR Invoices: Central fact table with invoice amounts, dates, payment status
- dim_Customers AR: Customer master with credit limits and contact details
- dim_Aging Bucket: Aging periods (Current, 1-30, 31-60, 61-90, 91-120, 121-150, 151-180, 180+ days)
- dim_Date: Time dimension for date-based analysis

## Key Metrics

Balance Metrics:
- Total Open Balance: All unpaid invoice amounts in USD
- Current Receivables Balance: Invoices within payment terms (not overdue)
- Overdue Receivables Balance: Invoices past due date

Performance Metrics:
- Average Days to Pay (DSO): Average collection period in days
- Payment Collection Percentage: Collection efficiency rate
- Overdue Balance Percentage: Percentage of balance that is overdue

## Business Terminology

| Term | Meaning |
|------|---------|
| Outstanding | Open Balance, Receivables Balance |
| Past Due | Overdue, Delinquent, Beyond Terms |
| Current | Within payment terms, Not overdue |
| DSO | Days Sales Outstanding, Average Days to Pay |
| Collection Rate | Payment Collection Percentage |

## Business Definition: Overdue Percentage

"Overdue Percentage" is a key health metric for Better Decisions Inc.
- Formula: Overdue Balance ÷ Total Open Balance × 100
- Target: Below 30%
- Current Status: ~65% (requires attention)
- Used for: AR health assessment, collection prioritization

## Response Guidelines

- Always include currency (USD) in balance responses
- Format large numbers: $52M, $174K
- Default to Top 5 for rankings unless specified
- Include percentage context for balance metrics
- State data "as of" date for time-sensitive queries
```

---

### 3.6 Key Principle

> The first set of instructions belongs in the semantic model — not just in the agent. This ensures consistency across all AI experiences (Copilot, Data Agent, M365).

---

## Step 4: Add Verified Answers

*[Waiting for screenshots]*

---

## Step 5: Add Descriptions to All Fields

*[Waiting for screenshots]*

---

## Step 6: Publish the Semantic Model

*[Waiting for screenshots]*

---

## Step 7: Configure in Power BI Service

*[Waiting for screenshots]*

---

## Step 8: Test with Copilot

*[Waiting for screenshots]*

---

## Step 9: Create Fabric Data Agent

*[Waiting for screenshots]*

---

## Step 10: Configure Permissions

*[Waiting for screenshots]*

---

## Step 11: Publish Data Agent

*[Waiting for screenshots]*

---

## Step 12: Access from M365 Copilot

*[Waiting for screenshots]*

---

## Step 13: Access from Excel

*[Waiting for screenshots]*

---

## Step 14: Configure Copilot Studio

*[Waiting for screenshots]*

---

## Step 15: Access from Teams

*[Waiting for screenshots]*

---
