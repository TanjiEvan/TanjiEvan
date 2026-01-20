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

![AR Dashboard](screenshots/step1-01-dashboard.png)

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

![Going to Model View](screenshots/1__Going_to_Data_Model.png)

---

### 1.3 Verify Star Schema Structure

A **star schema** is the ideal structure for Fabric Data Agent. It consists of:
- **Fact tables** at the center containing transactional data
- **Dimension tables** surrounding them with descriptive attributes
- **Dimensions connect directly to fact tables** (no dimension-to-dimension chains)

![Star Schema](screenshots/2__Star_Schema.png)

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

![Manage Relationships](screenshots/3__Manage_Relations.png)

2. Verify all relationships show:
   - **Many-to-One** cardinality (`*:1`)
   - **Single** cross-filter direction

![Relationships Dialog](screenshots/4__The_Relationships.png)

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

![Prep data for AI Button](screenshots/5__Prep_Data_for_AI.png)

The panel opens with three configuration options:

| Option | Purpose |
|--------|---------|
| **Simplify the data schema** | Select which tables and fields the AI can access |
| **Verified answers** | Create predefined responses for common business questions |
| **Add AI instructions** | Provide context about industry terms, business rules, and data usage |

---

### 2.2 Simplify the Data Schema

1. Click **Simplify the data schema** from the left sidebar

![Simplify Data Schema Option](screenshots/6__SELECT_FIELD.png)

2. The field selection interface displays all tables in your model

![Field Selection Interface](screenshots/7__Simplify_the_data_option.png)

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

![Apply Changes](screenshots/8__Apply_Changes.png)

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

![Add AI Instructions](screenshots/9__Add_AI_Instructions.png)

---

### 3.3 Add Instructions and Apply

The editor accepts plain text or Markdown format (up to 10,000 characters). Enter your instructions and click **Apply**.

![AI Instructions Editor with Apply](screenshots/10__AI_Instructions_Apply.png)

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

### Overview

Verified answers allow you to save predefined responses for common business questions. When users ask questions that match these phrases, Copilot responds with the exact visual you configured — ensuring consistent, accurate answers for frequently asked questions.

---

### 4.1 Open Verified Answers

1. Click **Verified answers** from the left sidebar in the Prep data for AI panel

![Verified Answers Navigation](screenshots/11__Verified_Ans.png)

---

### 4.2 Understanding Verified Answers

The Verified answers panel displays setup instructions:

![Verified Answers Setup Instructions](screenshots/12__Verfied_Ans_In_depth_.png)

| Step | Action |
|------|--------|
| 1 | In a Power BI report using this model, right-click a visual and select "Set up verified answer" |
| 2 | Add common phrases or questions users might ask about the data |
| 3 | Copilot will show the saved verified answer when users ask related questions in chat |

Verified answers are created from the **Report view**, not from this panel directly.

---

### 4.3 Create a Verified Answer from a Visual

1. Switch to **Report view** in Power BI Desktop
2. Right-click on a visual you want to use as a verified answer
3. Select **Set up a verified answer** from the context menu

![Right-click to Set Up Verified Answer](screenshots/13__right_click_then_Verfied_ans.png)

In this example, we're creating a verified answer from the **Total Open Balance by Aging Period Name** bar chart.

---

### 4.4 Add Phrases with Copilot Suggestions

The configuration panel opens with two sections:

| Section | Purpose |
|---------|---------|
| **Phrases connected to verified answers** | Questions that will trigger this visual as a response |
| **Visual** | Preview of the chart that will be shown to users |

Copilot provides AI-generated suggestions based on the visual's content:

![Copilot Suggestions](screenshots/14__Verified_Ans_Suggestions.png)

**Suggested Phrases for This Visual:**
- Which aging buckets have the highest open balances?
- What is the total open balance by aging bucket?
- Which aging bucket contributes most to open balances?

Click **Refresh** to generate additional suggestions if needed.

---

### 4.5 Select Phrases and Apply

1. Click on each suggested phrase to add it to the verified answer
2. Added phrases display with a green checkmark
3. You can also type custom phrases in the "Enter a phrase" field
4. Click **Apply** to save the verified answer

![Add Phrases and Apply](screenshots/15__Click_on_them_to_add.png)

---

### 4.6 Best Practices for Verified Answers

| Practice | Reason |
|----------|--------|
| **Clear all slicer selections** before creating verified answers | The visual captures the current filter state — slicers would limit the answer's usefulness |
| **Use descriptive phrases** that match how users naturally ask questions | Improves matching accuracy |
| **Create verified answers for key metrics** | Ensures consistent responses for important business questions |
| **Test phrases in Copilot chat** before publishing | Validates that the verified answer triggers correctly |

---

### 4.7 Recommended Verified Answers for AR Model

Consider creating verified answers for these common AR questions:

| Visual | Sample Phrases |
|--------|----------------|
| Total Open Balance by Aging Period | "Show aging breakdown", "What's the aging distribution?" |
| Top 5 Overdue by Resellers | "Who owes the most?", "Which customers are most overdue?" |
| Key Metrics Cards | "What's the total AR balance?", "How much is overdue?" |

---

### ✓ Verification

After adding verified answers:

- Phrases appear with green checkmarks in the configuration panel
- The visual preview shows the chart that will be returned
- Changes are saved when you click Apply
- Verified answers will be available after publishing the semantic model

---

## Step 5: Add Descriptions to All Fields

### Overview

Descriptions help Copilot understand the meaning and purpose of each table, column, and measure. Well-written descriptions significantly improve AI response accuracy by providing context that column names alone cannot convey.

---

### 5.1 Access the Properties Pane

1. Click the **Model view** icon in the left sidebar
2. Select a table or column in the diagram
3. The **Properties** pane appears on the right side

![Model View with Properties Pane](screenshots/16__Add_Descriptions.png)

The Properties pane displays:

| Property | Purpose |
|----------|---------|
| **Name** | The table or column name |
| **Description** | Free-text explanation for AI context |
| **Synonyms** | Alternative names users might use in queries |
| **Row label** | Column to display as the row identifier |
| **Key column** | Column with unique values for the table |
| **Is hidden** | Whether the field is hidden from report view |
| **Is featured table** | Marks table for Excel data types integration |

---

### 5.2 Adding Descriptions

1. Select a table or column in the model diagram
2. In the Properties pane, locate the **Description** field
3. Enter a clear, concise description
4. The description saves automatically

You can also add descriptions via the **Data** pane on the right — select any table to navigate quickly.

---

### 5.3 Description Best Practices

| Practice | Example |
|----------|---------|
| **Explain business meaning** | "Customer's approved credit limit in USD" not just "Credit limit" |
| **Include units** | "Invoice amount in US Dollars (USD)" |
| **Clarify calculations** | "Calculated as Overdue Balance ÷ Total Open Balance × 100" |
| **Note data source** | "Sourced from ERP system, updated daily" |
| **Add synonyms** | Include alternative terms users might search for |

---

### 5.4 Table Descriptions

Add the following descriptions to each table:

| Table | Description |
|-------|-------------|
| **fact_AR Invoices** | Central fact table containing all accounts receivable invoice transactions. Each row represents a single invoice with amount, dates, customer reference, and aging status. Grain: one row per invoice. |
| **dim_Customers AR** | Customer dimension containing customer master data including contact information, credit limits, and risk classifications. Links to fact_AR Invoices via Customer Key. |
| **dim_Date** | Standard date dimension for time-based analysis. Contains calendar attributes for filtering and grouping by year, quarter, month, and day. |
| **dim_Aging Bucket** | Reference dimension defining aging period categories for AR analysis. Categories range from Current (not overdue) through 180+ days past due. |
| **ARMetrics** | Measures table containing all calculated DAX measures for AR analysis including balances, percentages, and performance metrics. |

---

### 5.5 Column Descriptions — dim_Customers AR

| Column | Description |
|--------|-------------|
| **Customer Display Name** | Full customer name for display in reports and user-facing outputs. Primary identifier for customer in all visualizations. |
| **Customer Number** | Unique alphanumeric customer identifier from source ERP system. Use for precise customer lookup and data joins. |
| **Customer Key** | Internal surrogate key for dimensional modeling. Do not use for business analysis — use Customer Display Name instead. |
| **Customer Credit Limit** | Maximum approved credit amount in USD that can be extended to this customer. Used to calculate available credit. |
| **Customer Credit Risk Group** | Risk classification category assigned to customer (e.g., Low, Medium, High). Determines credit terms and collection priority. |
| **Intercompany Status** | Indicates whether the customer is an intercompany entity (internal business unit) or external customer. |
| **Market Classification** | Market segment or classification assigned to the customer for reporting and analysis purposes. |
| **Reseller ID** | Unique identifier for the reseller entity. |
| **Reseller Name** | Business name of the reseller organization. |
| **ResellerName** | *Duplicate — deselect in AI schema. Use Reseller Name instead.* |
| **Reseller Company ID** | Unique identifier for the reseller's parent company. |
| **ResellerCompanyID** | *Duplicate — deselect in AI schema. Use Reseller Company ID instead.* |
| **Reseller Company Name** | Name of the reseller's parent company organization. |
| **Phone Number** | Primary business phone contact for the customer. |
| **PhoneNumber** | *Duplicate — deselect in AI schema. Use Phone Number instead.* |
| **Fax Number** | Business fax number for the customer. |
| **FaxNumber** | *Duplicate — deselect in AI schema. Use Fax Number instead.* |
| **Website URL** | Customer's business website address. |
| **WebsiteURL** | *Duplicate — deselect in AI schema. Use Website URL instead.* |
| **Postal Address Line 1** | First line of customer's mailing address (street number and name). |
| **PostalAddressLine1** | *Duplicate — deselect in AI schema. Use Postal Address Line 1 instead.* |
| **Postal Address Line 2** | Second line of mailing address (suite, floor, building). |
| **PostalAddressLine2** | *Duplicate — deselect in AI schema. Use Postal Address Line 2 instead.* |
| **Postal ZIP Code** | ZIP or postal code for customer's mailing address. |
| **PostalPostalCode** | *Duplicate — deselect in AI schema. Use Postal ZIP Code instead.* |
| **Postal City ID** | City identifier for customer's mailing address. |
| **PostalCityID** | *Duplicate — deselect in AI schema. Use Postal City ID instead.* |
| **Delivery Address Line 1** | First line of customer's shipping/delivery address. |
| **DeliveryAddressLine1** | *Duplicate — deselect in AI schema. Use Delivery Address Line 1 instead.* |
| **Delivery Address Line 2** | Second line of delivery address. |
| **DeliveryAddressLine2** | *Duplicate — deselect in AI schema. Use Delivery Address Line 2 instead.* |
| **Delivery Postal Code** | ZIP or postal code for delivery address. |
| **DeliveryPostalCode** | *Duplicate — deselect in AI schema. Use Delivery Postal Code instead.* |

---

### 5.6 Column Descriptions — fact_AR Invoices

| Column | Description |
|--------|-------------|
| **Invoice ID** | Unique identifier for each invoice record. Primary key for the fact table. |
| **Invoice Number** | Business invoice number displayed on customer-facing documents. Use for invoice lookup and reference. |
| **Invoice Date** | Date the invoice was created in the system. |
| **Invoice Type** | Classification of invoice (e.g., Standard, Credit Memo, Debit Memo). |
| **Invoice Amount** | Original invoice amount before payments. May differ from Amount in USD if multi-currency. |
| **Amount in USD** | Invoice amount converted to US Dollars. Use for all financial reporting and aggregations. |
| **Reporting Currency Amount** | Invoice amount in the organization's reporting currency. |
| **Tax Amount** | Tax portion of the invoice amount in USD. |
| **Document Date** | Date the invoice document was issued to the customer. Used as the primary date for aging calculations. |
| **Payment Due Date** | Date by which payment is expected based on payment terms. Used to determine overdue status. |
| **Payment Date** | Actual date payment was received. NULL for unpaid invoices. |
| **Payment Amount** | Amount paid against this invoice. May be partial or full payment. |
| **Payment Status** | Current payment state of the invoice (e.g., Open, Paid, Partial, Written Off). |
| **Payment Terms Days** | Number of days allowed for payment from invoice date (e.g., Net 30, Net 60). |
| **Current Date** | Reference date used for aging calculations. Typically the report refresh date or analysis date. |
| **Days Past Due** | Number of days the invoice is overdue beyond payment terms. Zero or negative values indicate invoice is not yet due. |
| **Days to Pay** | Actual number of days taken to pay the invoice. NULL for unpaid invoices. Used to calculate Average Days to Pay (DSO). |
| **Aging Category** | Text label for the aging bucket (e.g., "Current", "1-30 Days", "31-60 Days"). Derived from Days Past Due. |
| **Aging Sort Order** | Numeric sort order for aging categories. Ensures proper sequence in visualizations (Current=1, 1-30=2, etc.). |
| **Customer Key** | Foreign key linking to dim_Customers AR. Internal use only — use Customer Display Name for reporting. |
| **Customer Number** | Customer identifier matching the customer master. Used for customer-level aggregations. |
| **Salesperson ID** | Identifier for the salesperson assigned to this invoice/customer. Internal key — deselect in AI schema. |

---

### 5.7 Column Descriptions — dim_Date

| Column | Description |
|--------|-------------|
| **Date** | Full calendar date. Primary key for the date dimension. Format: YYYY-MM-DD. |
| **Calendar Year** | Four-digit calendar year (e.g., 2024, 2025). Use for annual comparisons and YoY analysis. |
| **Calendar Quarter** | Calendar quarter designation (Q1, Q2, Q3, Q4). Q1 = Jan-Mar, Q2 = Apr-Jun, Q3 = Jul-Sep, Q4 = Oct-Dec. |
| **Month Name** | Full name of the month (e.g., January, February). Use for display in reports. |
| **Month Abbreviation** | Three-letter month abbreviation (e.g., Jan, Feb, Mar). Use for compact displays. |
| **Month Number** | Numeric month (1-12). January = 1, December = 12. Use for sorting and calculations. |
| **Month Start Date** | First day of the month. Useful for month-level aggregations and comparisons. |

---

### 5.8 Column Descriptions — dim_Aging Bucket

| Column | Description |
|--------|-------------|
| **Aging Period Name** | Display name for the aging category. Values: Current, 1-30 Days, 31-60 Days, 61-90 Days, 91-120 Days, 121-150 Days, 151-180 Days, 180+ Days. |
| **Aging Period Sort Order** | Numeric sequence for proper sorting in visualizations. Current = 1, 1-30 Days = 2, through 180+ Days = 8. |
| **Payment Status** | Indicates whether invoices in this bucket are Current (within terms) or Overdue (past due). |

---

### 5.9 Measure Descriptions — ARMetrics

| Measure | Description |
|---------|-------------|
| **Total Open Balance** | Sum of all unpaid invoice amounts in USD. Represents total accounts receivable outstanding. Primary AR health metric. |
| **Open Balance Accounting Currency** | Total open balance expressed in the organization's accounting/reporting currency. |
| **Total Receivables Balance** | Total value of all receivables including both current and overdue amounts. Equivalent to Total Open Balance. |
| **Current Receivables Balance** | Sum of invoice amounts that are within payment terms (not yet overdue). Indicates healthy receivables. |
| **Overdue Receivables Balance** | Sum of invoice amounts that are past their due date. Key metric for collection focus. |
| **Overdue Balance Percentage** | Percentage of total AR that is overdue. Formula: Overdue Balance ÷ Total Open Balance × 100. Target: Below 30%. |
| **Current Balance Percentage** | Percentage of total AR that is current (not overdue). Formula: Current Balance ÷ Total Open Balance × 100. |
| **Average Days to Pay** | Average number of days customers take to pay invoices. Also known as DSO (Days Sales Outstanding). Lower is better. |
| **Total Customers with Open Invoices** | Count of distinct customers who have at least one unpaid invoice. |
| **Total Open Invoices** | Count of all unpaid invoices across all customers. |
| **Available Customer Credit** | Remaining credit available for a customer. Formula: Credit Limit − Current Open Balance. |
| **Customer Credit Limit** | Selected customer's maximum approved credit amount in USD. |
| **Monthly Amount Due** | Total invoice amount due within the selected month based on payment terms. |
| **Monthly Payment Amount** | Total payments received within the selected month. |
| **Monthly Payment Percentage** | Collection efficiency for the month. Formula: Monthly Payment ÷ Monthly Amount Due × 100. |
| **Payment Collection Percentage** | Overall collection rate across all receivables. Measures AR team effectiveness. |
| **Payment Variance** | Difference between expected and actual payment amounts. Negative indicates underpayment. |
| **Total Amount Due** | Total amount that should have been collected based on payment terms. |
| **Total Invoiced Amount** | Sum of all invoices issued (paid and unpaid). Represents total billing activity. |
| **Total Payment Received** | Sum of all payments collected from customers. |

---

### 5.10 Adding Synonyms

Synonyms help Copilot understand alternative terms users might use. Add synonyms in the **Synonyms** field of the Properties pane.

| Field | Recommended Synonyms |
|-------|---------------------|
| Total Open Balance | Outstanding Balance, AR Balance, Receivables Balance, Open AR |
| Overdue Receivables Balance | Past Due Balance, Delinquent Balance, Late Balance |
| Customer Display Name | Customer Name, Account Name, Client Name |
| Days Past Due | Days Overdue, Days Late, Overdue Days |
| Average Days to Pay | DSO, Days Sales Outstanding, Collection Period |

---

### ✓ Verification

After adding descriptions:

- Every table has a description explaining its purpose and grain
- Every column has a description with business meaning
- Every measure has a description including formula where applicable
- Key fields have synonyms for alternative search terms
- Descriptions are concise but informative

> **Key Principle:** Add descriptions to ALL fields — even those not exposed to AI. This builds good documentation habits and ensures the model is self-documenting for future maintainers.

---

## Step 6: Publish the Semantic Model

### Overview

After configuring the semantic model with AI optimizations (field selection, descriptions, AI instructions, and verified answers), publish it to the Power BI Service. The model must be published to a workspace with Fabric or Premium capacity to enable Data Agent functionality.

---

### 6.1 Publish from Power BI Desktop

1. Click the **Publish** button in the Home tab (Share section)

![Publish Button](screenshots/17__Publish_Button.png)

---

### 6.2 Select Destination Workspace

The **Publish to Power BI** dialog displays available workspaces:

![Select Workspace](screenshots/18__Select_Workspace.png)

1. Select a workspace with **Fabric capacity** or **Premium capacity**
2. Click **Select** to begin publishing

| Workspace Type | Data Agent Support |
|----------------|-------------------|
| Fabric capacity (F SKU) | ✓ Supported |
| Premium capacity (P SKU) | ✓ Supported |
| Premium Per User (PPU) | ✓ Supported |
| Pro workspace | ✗ Not supported |
| My workspace | ✗ Not supported for Data Agent |

---

### 6.3 Publish Confirmation

After successful publishing, a confirmation dialog appears:

![Publish Success](screenshots/19__Publish_Success.png)

The dialog provides:
- **Success** confirmation message
- Link to **open the report in Power BI Service**
- Option to **Get Quick Insights**

Click **Got it** to close the dialog, or click the link to open your report in Power BI Service.

---

### 6.4 What Gets Published

When you publish from Power BI Desktop, two artifacts are created in the workspace:

| Artifact | Description |
|----------|-------------|
| **Report** | The Power BI report with all visualizations |
| **Semantic Model** | The underlying data model with AI configurations |

All AI configurations (field selection, descriptions, AI instructions, verified answers) are included in the published semantic model.

---

### ✓ Verification

After publishing:

- The workspace shows both the report and semantic model
- The semantic model includes all AI configurations from Power BI Desktop
- The workspace has Fabric or Premium capacity assigned

---

## Step 7: Configure in Power BI Service

### Overview

After publishing, verify the AI configurations in Power BI Service and ensure the semantic model is properly configured for Copilot and Data Agent use. This step confirms that all settings from Power BI Desktop were successfully published.

---

### 7.1 Navigate to the Workspace

1. Open **Power BI Service** (app.powerbi.com)
2. Navigate to the workspace where you published the model

![Workspace View](screenshots/20__Workspace_View.png)

The workspace displays both artifacts created during publishing:

| Artifact | Type | Description |
|----------|------|-------------|
| Better Decisions AR Analytics | Report | The Power BI report with visualizations |
| Better Decisions AR Analytics | Semantic model | The underlying data model with AI configurations |

---

### 7.2 Open the Semantic Model

1. Click on the **Semantic model** row (not the Report)

![Select Semantic Model](screenshots/21__Select_Semantic_Model.png)

The semantic model details page opens, showing model information and available actions.

---

### 7.3 Access Prep Data for AI

1. In the semantic model toolbar, click **Prep data for AI**

![Prep Data for AI Button](screenshots/22__Prep_Data_for_AI_Service.png)

The toolbar also provides access to other useful features:
- **Explore this data** — Create ad-hoc reports
- **Share semantic model** — Grant access to other users
- **Analyze in Excel** — Connect Excel to the model
- **Open semantic model** — View in editing mode

---

### 7.4 Enable Large Model Storage (If Required)

The Prep data for AI panel may display a warning about large model storage:

![Large Model Storage Warning](screenshots/23__Large_Model_Storage.png)

If you see the message "Turn on the large semantic model storage format for this semantic model to prep your data for AI":

1. Click **Turn on large model storage**
2. Wait for the model to be converted (this may take a moment)

Large model storage is required for AI features to function properly.

---

### 7.5 Verify AI Configurations

After enabling large model storage (if needed), verify that all configurations from Power BI Desktop are present:

**Simplify the Data Schema:**

![Verify Schema](screenshots/24__Verify_Schema.png)

Confirm that:
- Core tables are selected (ARMetrics, dim_Aging Bucket, dim_Customers AR, dim_Date, fact_AR Invoices)
- Hidden tables are deselected (Reseller, ResellerCompany)
- Partial selections show for tables with some columns deselected

---

**Verified Answers:**

![Verify Answers](screenshots/25__Verify_Answers.png)

Confirm that your verified answers are present with their associated phrases:
- "Which customers have the highest open balances?" (3 phrases)
- "Which aging buckets have the highest open balances?" (3 phrases)
- "Who owes the most?", "Which customers are most overdue?" (4 phrases)

---

**AI Instructions:**

![Verify Instructions](screenshots/26__Verify_Instructions.png)

Confirm that your AI instructions are present, including:
- Time Intelligence guidance
- Data Model Structure documentation
- Key Metrics definitions
- Business terminology

---

### 7.6 Tenant Settings (Administrator)

For Copilot and Data Agent features to work, certain tenant settings must be enabled by a Power BI administrator. If you encounter issues, verify these settings are configured:

| Setting | Location | Required For |
|---------|----------|--------------|
| Users can use Copilot and other features powered by Azure OpenAI | Admin Portal → Tenant settings → Copilot and Azure OpenAI Service | All Copilot features |
| Users can use natural language to DAX in Quick measure suggestions | Admin Portal → Tenant settings → Q&A | Natural language queries |
| Allow shareable links for Copilot in Power BI | Admin Portal → Tenant settings → Copilot and Azure OpenAI Service | Sharing Copilot content |

> **Note:** Tenant settings are managed by Power BI administrators. If these features are not available, contact your admin to enable them.

---

### ✓ Verification

After configuring in Power BI Service:

- Semantic model is visible in the workspace
- Large model storage is enabled (if prompted)
- All AI configurations (schema, verified answers, instructions) are present
- Tenant settings are enabled (if you have admin access)

The semantic model is now ready for testing with Copilot.

---

## Step 8: Test with Copilot

### Overview

Before creating the Fabric Data Agent, test the semantic model with Copilot in Power BI Service. This validates that your AI configurations (field selection, descriptions, AI instructions, verified answers) are working correctly.

---

### 8.1 Open the Report in Power BI Service

1. Navigate to your workspace in Power BI Service
2. Click on the **Report** (not the semantic model)
3. The report opens in viewing mode

![Report in Service](screenshots/27__Report_in_Service.png)

---

### 8.2 Open the Copilot Pane

1. Click the **Copilot** button in the top-right toolbar

![Copilot Pane](screenshots/28__Copilot_Pane.png)

The Copilot pane opens with several sections:

| Section | Purpose |
|---------|---------|
| **Recommended** | AI-suggested questions based on your report |
| **Understand the data** | Questions to explore the data model |
| **Dig deeper** | Questions for detailed analysis |
| **Write a summary** | Generate narrative summaries |
| **Ask a question** | Free-form natural language input |

---

### 8.3 Test Questions

Test your semantic model with questions that validate both basic functionality and your AI configurations.

**Basic Validation Questions:**

| Question | What It Tests |
|----------|---------------|
| "What is the total open balance?" | Basic measure retrieval |
| "Show me the aging breakdown" | Should trigger verified answer |
| "Which customers have the highest overdue balance?" | Should trigger verified answer |

---

**Business Impact Question:**

Ask this question to test a real-world AR analysis scenario:

> **"Show me the top 10 customers by overdue balance with their credit risk group"**

This question tests:
- Ranking capability (top 10)
- Multiple field retrieval (customer, overdue balance, credit risk group)
- Business terminology understanding from AI instructions
- Proper joining of fact table with customer dimension

---

### 8.4 Review Copilot Response

Copilot returns both a narrative insight and a visualization:

![Copilot Test Response](screenshots/29__Copilot_Test_Response.png)

**Narrative Response:**
> "Based on the available data, the top 10 overdue accounts skew to Credit Risk Group C (7/10), all Wingtip Toys locations. Tailspin Toys appear twice in Group A (Clewiston, FL; South La Paloma, TX) and once in Group B (Hanoverton, OH)."

**Visual Response:**
- Bar chart showing Overdue Receivables Balance by Customer
- Color-coded by Customer Credit Risk Group (A, B, C)

---

### 8.5 Explore the Answer

Click **Explore answer** to open the interactive exploration view:

![Explore Answer Button](screenshots/30__Explore_Answer_Button.png)

The exploration view provides full interactivity:

![Explore Answer View](screenshots/31__Explore_Answer_View.png)

| Feature | Description |
|---------|-------------|
| **Filter applied** | "Customer Display Name top 10 by Overdue Receivables Balance" |
| **Visual options** | Change chart type, sort order, drill options |
| **Data pane** | Access all available measures and dimensions |
| **Rearrange data** | Modify axes and legend assignments |

This demonstrates that Copilot correctly understood the request and generated an actionable visualization for collection prioritization.

---

### 8.6 Additional Test Questions

| Business Scenario | Test Question |
|-------------------|---------------|
| **Collection Priority** | "What percentage of our AR is over 90 days past due?" |
| **Customer Risk** | "Show me customers with high credit risk and overdue balances" |
| **Aging Trend** | "Which aging bucket has the largest balance?" |
| **Top Exposure** | "Who are our top 5 customers by total open balance?" |
| **Payment Performance** | "What is our average days to pay?" |

---

### 8.7 Verify AI Configurations

As you test, confirm that:

| Configuration | How to Verify |
|---------------|---------------|
| **Field Selection** | Copilot doesn't reference deselected fields (Reseller, ResellerCompany) |
| **Descriptions** | Copilot correctly interprets field meanings |
| **AI Instructions** | Time intelligence and business terms are understood |
| **Verified Answers** | Matching phrases return the configured visual |

---

### 8.8 Troubleshooting

| Issue | Possible Cause | Solution |
|-------|----------------|----------|
| Copilot not available | Tenant settings not enabled | Contact Power BI admin |
| Incorrect results | Missing or unclear descriptions | Update field descriptions |
| Verified answer not triggering | Phrase doesn't match | Add more phrase variations |
| Copilot references wrong fields | Duplicate column names | Deselect redundant columns in schema |

---

### ✓ Verification

After testing with Copilot:

- Basic questions return accurate results
- Verified answers trigger for matching phrases
- Business terminology from AI instructions is understood
- Copilot uses only the fields exposed in the simplified schema

The semantic model is validated and ready for Fabric Data Agent creation.

---

## Step 9: Create Fabric Data Agent

### Overview

A Fabric Data Agent is a generative AI agent that connects to your semantic model and can answer complex questions across multiple conversational interfaces including M365 Copilot, Excel, and Teams. Unlike Copilot in Power BI reports, Data Agents can be shared broadly and accessed outside of Power BI.

---

### 9.1 Create New Item

1. Navigate to your workspace in Power BI Service
2. Click **+ New item** in the toolbar

![New Item Button](screenshots/32__New_Item.png)

---

### 9.2 Select Data Agent

1. In the New item panel, search for "data agent"
2. Select **Data agent (preview)** under "Analyze and train data"

![Select Data Agent](screenshots/33__Select_Data_Agent.png)

The Data Agent is described as: "Build a generative AI agent that understands your data and can answer complex questions in a variety of conversational interfaces."

---

### 9.3 Name the Data Agent

1. Enter a name for your Data Agent (e.g., "AR Agent")
2. Click **Create**

![Name Data Agent](screenshots/34__Name_Data_Agent.png)

---

### 9.4 Data Agent Editor

The Data Agent editor opens with setup instructions:

![Data Agent Editor](screenshots/35__Data_Agent_Editor.png)

The setup process involves three steps:
1. Add a data source and select tables in the Explorer pane
2. Ask questions in the chat to test the agent's responses
3. Improve accuracy and quality by adding instructions and examples

Click **+ Data source** or **Add a data source** to continue.

---

### 9.5 Select Semantic Model

The OneLake catalog displays available data sources:

![Select Semantic Model](screenshots/36__Select_Semantic_Model.png)

1. Locate your semantic model (**Better Decisions AR Analytics**)
2. Select it and click **Connect**

You can filter by:
- **My data** — Items you own
- **Endorsed in your org** — Certified or promoted items
- **Favorites** — Items you've starred

---

### 9.6 Verify Tables

After connecting, the Explorer pane shows the semantic model with all available tables:

![Verify Tables](screenshots/37__Verify_Tables.png)

Confirm that all required tables are selected:
- ✓ ARMetrics
- ✓ dim_Aging Bucket
- ✓ dim_Customers AR
- ✓ dim_Date
- ✓ fact_AR Invoices

The main panel displays the agent name and sample question prompts.

---

### 9.7 Add Agent Instructions

1. Click the **Setup** tab in the Explorer pane

![Setup Tab](screenshots/38__Setup_Tab.png)

2. Click **Agent instructions** in the left panel to open the instructions editor

![Agent Instructions Editor](screenshots/39__Agent_Instructions_Editor.png)

The instructions editor renders Markdown formatting, displaying your instructions with proper headings, bullet points, and sections. Agent instructions define the agent's persona, response guidelines, and behavioral rules.

---

### 9.8 Sample Agent Instructions

Add the following instructions to your Data Agent:

```markdown
## Agent Identity

You are the AR Agent for Better Decisions Inc. Your role is to help finance teams, credit managers, and executives analyze accounts receivable data and make informed collection decisions.

## Your Capabilities

You can answer questions about:
- Customer balances and overdue amounts
- Aging analysis and trends
- Credit risk assessments
- Collection prioritization
- Payment performance metrics (DSO, collection rates)

## Response Guidelines

### Formatting
- Always display currency values in USD with appropriate formatting ($52M, $174K)
- Round percentages to two decimal places
- When showing rankings, default to Top 5 unless the user specifies otherwise
- Include the data "as of" date when discussing balances or time-sensitive metrics

### Context
- The data represents accounts receivable for Better Decisions Inc.
- Aging buckets: Current, 1-30 Days, 31-60 Days, 61-90 Days, 91-120 Days, 121-150 Days, 151-180 Days, 180+ Days
- "Overdue" means any invoice past its payment due date (not in "Current" bucket)
- Credit Risk Groups: A (lowest risk), B (medium risk), C (highest risk)

### Business Rules
- Overdue Percentage target is below 30% — current status (~65%) requires attention
- High-priority collections: Customers with overdue balance > $100K AND Credit Risk Group C
- DSO (Days Sales Outstanding) benchmark: Industry average is 45 days

## Suggested Analysis Approaches

When asked about collection priorities:
1. First identify customers with highest overdue balances
2. Cross-reference with credit risk classification
3. Consider aging bucket distribution (older = higher priority)

When asked about AR health:
1. Report Total Open Balance and Overdue Percentage
2. Compare current metrics to targets
3. Highlight concerning trends (e.g., increasing 90+ day balances)

## Limitations

- You cannot modify data or update records
- You cannot send emails or initiate collection actions
- You cannot access data outside the AR semantic model
- For forecast or prediction questions, clarify that you analyze historical data only
```

---

### 9.9 Instructions: Model vs Agent

| Location | Purpose | Example Content |
|----------|---------|-----------------|
| **Semantic Model** | Data context, field meanings, business terminology | "Overdue = invoices past due date", "DSO = Days Sales Outstanding" |
| **Data Agent** | Agent persona, response formatting, behavioral rules | "You are the AR Agent", "Default to Top 5 for rankings" |

Both work together — the semantic model instructions help the agent understand the data, while agent instructions shape how it responds.

---

### ✓ Verification

After creating the Data Agent:

- Agent appears in the workspace with type "Data agent"
- Semantic model is connected with all tables visible
- Agent instructions are configured
- Test chat interface is available

The Data Agent is ready for testing and permission configuration.

---

## Step 10: Configure Permissions

### Overview

Sharing a Data Agent requires careful attention to permissions. Access to the Data Agent is **not the same** as access to the underlying data — users need appropriate permissions on both the agent AND the semantic model to use it successfully.

---

### 10.1 Share the Data Agent

1. In the Data Agent editor, click the **Share** button in the top-right corner

![Share Button](screenshots/40__Click_on_share.png)

---

### 10.2 Create and Send Link

The sharing dialog opens with options to share the agent:

![Share Dialog](screenshots/41__Permission.png)

| Option | Description |
|--------|-------------|
| **Copy link** | Generate a shareable URL |
| **by Email** | Send invitation via email |
| **by Teams** | Share directly in Microsoft Teams |
| **by PowerPoint** | Embed in a PowerPoint presentation |

Click **People in your organization can view** to configure permission scope.

---

### 10.3 Select Permission Scope

Configure who can access the Data Agent:

![Select Permissions](screenshots/42__Select_Permissions.png)

**People who can view this Data agent:**

| Option | Use Case |
|--------|----------|
| **People in your organization** | Broad access for entire organization |
| **People with existing access** | Only those who already have workspace access |
| **Specific people** | Named individuals or security groups |

**Additional permissions:**

| Permission | Grants |
|------------|--------|
| **Share** | Ability to reshare the agent with others |
| **Edit and view details** | Modify agent configuration |
| **View details** | See agent settings without editing |

---

### 10.4 Critical: Data Source Permissions

> ⚠️ **Important:** Access to the data agent isn't the same as access to connected data sources. People you share the data agent with will only get responses based on the data they have permission to view.

This means:
- Sharing the Data Agent does **NOT** automatically grant data access
- Users must have **Build permission** on the underlying semantic model
- Without proper semantic model permissions, users will receive errors or empty results

---

### 10.5 Grant Semantic Model Permissions

To ensure users can query data through the agent, grant Build permission on the semantic model:

**Option 1: Direct Permission**
1. Navigate to the workspace
2. Find the semantic model (**Better Decisions AR Analytics**)
3. Click **...** → **Manage permissions**
4. Add users and grant **Build** permission

**Option 2: Workspace Role**
Users with these workspace roles automatically have Build permission:

| Workspace Role | Build Permission |
|----------------|------------------|
| Admin | ✓ Yes |
| Member | ✓ Yes |
| Contributor | ✓ Yes |
| Viewer | ✗ No — must grant Build separately |

---

### 10.6 Permission Summary

| Layer | What It Controls | Required For |
|-------|------------------|--------------|
| **Data Agent** | Access to the agent interface | Using the conversational UI |
| **Semantic Model (Build)** | Access to query the data | Getting actual results from queries |

Both permissions are required for a complete user experience.

---

### ✓ Verification

After configuring permissions:

- Data Agent is shared with target users/groups
- Users have Build permission on the semantic model
- Test with a user who has both permissions to confirm access

---

## Step 11: Publish Data Agent

### Overview

Publishing makes the Data Agent available for use. There are two publishing options: publish to Fabric only (for internal testing) or publish to the Microsoft 365 Copilot Agent Store (for broad organizational access).

---

### 11.1 Open Publish Dialog

1. In the Data Agent editor, click **Publish** in the toolbar
2. The Publish data agent dialog opens

---

### 11.2 Option A: Publish to Fabric Only

For internal testing or Fabric-only access, leave the M365 toggle **Off**:

![General Publish](screenshots/43__General_Publish.png)

| Field | Description |
|-------|-------------|
| **Name** | The agent name (AR Agent) |
| **Description of purpose and capabilities** | Explain what questions the agent can answer and what data it accesses |
| **Also publish to the Agent Store in Microsoft 365 Copilot** | Toggle **Off** |

Click **Publish** to make the agent available within Fabric.

---

### 11.3 Option B: Publish to M365 Copilot

For access through Microsoft 365 Copilot, Excel, and other M365 applications, toggle the option **On**:

![M365 Publish](screenshots/44__M365_Publish.png)

| Setting | Status |
|---------|--------|
| **Also publish to the Agent Store in Microsoft 365 Copilot** | Toggle **On** |
| **Availability** | Available immediately for personal use and sharing |

Click **Publish** to deploy to both Fabric and M365.

---

### 11.4 Publishing Options Comparison

| Option | Access Points | Use Case |
|--------|---------------|----------|
| **Fabric Only** (Toggle Off) | Fabric portal, shared links | Internal testing, limited rollout |
| **M365 Copilot** (Toggle On) | M365 Copilot, Excel, Teams, Fabric | Broad organizational deployment |

---

### 11.5 Description Best Practices

The description helps users understand what the agent does. Include:

**Example Description:**
> "The AR Agent answers questions about Better Decisions Inc. accounts receivable data. Ask about customer balances, overdue amounts, aging analysis, credit risk assessments, and collection priorities. Data includes invoices, customers, and payment history."

---

### ✓ Verification

After publishing:

- Agent status changes from "Draft" to published
- Agent appears in the workspace item list
- If M365 toggle was On, agent appears in M365 Copilot Agent Store

---

## Step 12: Access from M365 Copilot

### Overview

After publishing to the Microsoft 365 Copilot Agent Store, users with M365 Copilot licenses can access the Data Agent directly from the M365 Copilot interface. This provides a familiar chat experience integrated with other M365 productivity tools.

---

### 12.1 Open M365 Copilot

1. Navigate to **microsoft365.com/copilot** or open the M365 Copilot app
2. In the left sidebar under **Agents**, locate your published agent

![M365 Copilot Agents](screenshots/45__M365_Copilot_Agents.png)

The AR Agent appears in the Agents list alongside other available agents in your organization.

---

### 12.2 Start a Conversation

1. Click on the **AR Agent** to open a chat session
2. Ask a business question to test the agent

**Recommended Test Question:**

> "What is our AR health status and which customers should be prioritized for collections?"

---

### 12.3 Review Agent Response

The AR Agent responds with a comprehensive, actionable analysis:

![M365 Copilot Response](screenshots/46__M365_Copilot_Response.png)

**Sample Response:**

**🔎 Accounts Receivable (AR) Health Status**

**Overall Status: Needs Immediate Attention**
- Total Open AR: $52M
- Total Overdue: 64.87% (target is 30%) → Overdue is more than double the target
- 90+ Day Overdue: $29.7M
  - This represents 57.1% of the entire AR portfolio
  - Indicates significant aging and collection risk

**Aging Breakdown:**
- Current: $18.3M
- 1-30 Days: $2.0M
- 31-60 Days: $1.0M

The agent also suggests follow-up actions: "Show me the customer-level priority list for overdue balances over $100K"

---

### 12.4 Key Observations

The response demonstrates that the agent is following configured instructions:

| Instruction | Evidence in Response |
|-------------|---------------------|
| Compare to 30% target | "target is 30%" explicitly mentioned |
| Format currency properly | $52M, $29.7M, $18.3M formatting |
| Provide actionable insights | "Needs Immediate Attention" status |
| Include percentage context | 64.87%, 57.1% calculations |

---

### 12.5 Additional Test Questions

| Question | Business Value |
|----------|----------------|
| "Which Credit Risk Group C customers have overdue balances over $100K?" | Identifies highest-risk accounts for immediate action |
| "How does our current overdue percentage compare to the 30% target?" | Performance against KPI |
| "What's the total exposure in the 90+ days aging buckets?" | Quantifies severely delinquent receivables |
| "Show me the trend of overdue balance by aging bucket" | Identifies if AR health is improving or deteriorating |
| "Which customers have the highest DSO?" | Identifies slow-paying customers |

---

### 12.6 License Requirements

| Requirement | Details |
|-------------|---------|
| **M365 Copilot License** | Required for each user accessing the agent through M365 Copilot |
| **Semantic Model Build Permission** | Required for data access (configured in Step 10) |
| **Data Agent Access** | Granted through sharing (configured in Step 10) |

---

### ✓ Verification

After accessing from M365 Copilot:

- Agent appears in the Agents list
- Chat interface opens when agent is selected
- Questions return accurate, formatted responses
- Agent follows configured instructions (formatting, business rules)

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
