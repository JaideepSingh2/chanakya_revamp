## 1. Master Data Tables

### 1.1 Project (`xr_project`)

_(Note: This table already exists in the environment and does not need to be created. It is included here for relationship reference only. Its true lookup target is its GUID, which standardizes here under the name Project ID.)_

| Display Name   | Data Type         | Possible Values / Logical Details |
| :------------- | :---------------- | :-------------------------------- |
| **Project ID** | Unique Identifier | (Primary Key) GUID                |

### 1.2 Verticals & Budget Head (`xr_verticalbudgethead`)

A unified hierarchical table that manages Verticals, Budget Heads, and Sub Budget Heads.

| Display Name                | Data Type           | Possible Values / Logical Details                                           |
| :-------------------------- | :------------------ | :-------------------------------------------------------------------------- |
| **xr_verticalbudgetheadid** | Unique Identifier   | (Primary Key) GUID                                                          |
| **Vertical Budget Number**  | Auto-number         | (Unique Text Identifier)                                                    |
| **Category Name**           | Single Line of Text | e.g., "Channel Partner", "Telecallers", "Hostess"                           |
| **Category Type**           | Choice              | `Vertical`, `Budget Head`, `Sub Budget Head`                                |
| **Parent Category**         | Lookup              | Related to `Verticals & Budget Head` table (Self-referencing for hierarchy) |

### 1.3 Project Budget Allocation (`xr_projectbudget`)

Stores the allocated budget for each Project + Category (Budget Head) combination, sourced from the existing Project_Wise_Budget SharePoint list.

| Display Name                  | Data Type                        | Possible Values / Logical Details                                                            |
| :---------------------------- | :------------------------------- | :------------------------------------------------------------------------------------------- |
| **xr_projectbudgetid**        | Unique Identifier                | (Primary Key) GUID                                                                           |
| **Allocation Number**         | Auto-number                      | (Unique Text Identifier) `PB-{SEQNUM}`                                                       |
| **Project**                   | Lookup (`xr_project`)            | References `xr_project.Project ID` (GUID); `ProjectName` may be shown in the UI as the label |
| **Vertical**                  | Lookup (`xr_verticalbudgethead`) | Filtered to `Category Type = Vertical`                                                       |
| **Budget Head**               | Lookup (`xr_verticalbudgethead`) | Filtered to children of selected Vertical                                                    |
| **Region**                    | Choice                           | e.g. `MMR`                                                                                   |
| **Evaluated vendor**          | Lookup (`xr_vendoronboarding`)   | Contracted vendor for this bucket limit                                                      |
| **Approver Role**             | Choice                           | e.g. `Strategy`                                                                              |
| **Budget (Allocated Amount)** | Currency                         | Base sanctioned amount                                                                       |
| **Pricing**                   | Currency                         | Base pricing                                                                                 |
| **Approved Amount**           | Currency                         | Officially approved spend                                                                    |
| **Tax**                       | Decimal Number                   | Tax percentage (e.g. 18%)                                                                    |
| **Consumption**               | Currency                         | Amount utilized                                                                              |
| **Balance (In Rs)**           | Calculated (Currency)            | Native Calculation: `Budget` - `Consumption`                                                 |
| **Consumption Percentage**    | Calculated (Decimal)             | Native Calculation: `(Consumption / Budget) * 100`                                           |

_(Note: Formatting columns like `BudgetFormatted`, as well as standard system fields like `Created`, `Modified`, `access` are natively handled by Dataverse and removed to avoid redundancy)._

---

## 2. Vendor Onboarding (`xr_vendoronboarding`)

Maintains the centralized database of all onboarded vendors based on the onboarding form.

| Display Name              | Data Type                        | Possible Values / Logical Details                       |
| :------------------------ | :------------------------------- | :------------------------------------------------------ |
| **xr_vendoronboardingid** | Unique Identifier                | (Primary Key) GUID                                      |
| **Onboarding Number**     | Auto-number                      | (Unique Text Identifier)                                |
| **Company Name**          | Single Line of Text              |                                                         |
| **Approval Status**       | Choice                           | `Pending`, `Approved`, `Rejected`                       |
| **Vertical**              | Lookup (`xr_verticalbudgethead`) | Filtered to `Category Type = Vertical`                  |
| **Budget Head**           | Lookup (`xr_verticalbudgethead`) | Filtered to children of selected Vertical               |
| **Sub Budget Head**       | Lookup (`xr_verticalbudgethead`) | Filtered to children of selected Budget Head (Optional) |
| **Email ID**              | Single Line of Text              | Format: Email (Regex validated in UI)                   |
| **Contact Name**          | Single Line of Text              |                                                         |
| **Phone Number**          | Single Line of Text              | 10 digits                                               |
| **Payment Credit Period** | Single Line of Text              | No. of Days representation                              |
| **GSTIN**                 | Single Line of Text              | 15 characters                                           |
| **PAN**                   | Single Line of Text              | Extracted from GSTIN (chars 3 to 12) or 10 chars        |
| **Billing Address**       | Multiple Lines of Text           |                                                         |
| **Billing Street 2**      | Single Line of Text              |                                                         |
| **Billing City**          | Single Line of Text              |                                                         |
| **Billing State**         | Single Line of Text              |                                                         |
| **Billing Code**          | Single Line of Text              |                                                         |

---

## 3. Vendor Evaluation - Header (`xr_vendorevaluation`)

The parent evaluation record. Holds the overall context (project, category, decision) and links to up to 3 vendor detail records.

| Display Name              | Data Type                        | Possible Values / Logical Details                  |
| :------------------------ | :------------------------------- | :------------------------------------------------- |
| **xr_vendorevaluationid** | Unique Identifier                | (Primary Key) GUID                                 |
| **Evaluation Number**     | Auto-number                      | (Unique Text Identifier) e.g., `EVAL-{SEQNUM}`     |
| **Project**               | Lookup (`xr_project`)            |                                                    |
| **Vertical**              | Lookup (`xr_verticalbudgethead`) | Filtered to `Category Type = Vertical`             |
| **Budget Head**           | Lookup (`xr_verticalbudgethead`) | Filtered to children of selected Vertical          |
| **Sub Budget Head**       | Lookup (`xr_verticalbudgethead`) | Filtered to children of Budget Head (if available) |
| **Vendor 1 Details**      | Lookup (`xr_vendorevaldetail`)   | Links to the evaluation detail record for Vendor 1 |
| **Vendor 2 Details**      | Lookup (`xr_vendorevaldetail`)   | Links to the evaluation detail record for Vendor 2 |
| **Vendor 3 Details**      | Lookup (`xr_vendorevaldetail`)   | Links to the evaluation detail record for Vendor 3 |
| **Preferred Vendor**      | Choice                           | `Vendor 1`, `Vendor 2`, `Vendor 3`                 |
| **Reason for Preference** | Multiple Lines of Text           |                                                    |

---

## 4. Vendor Evaluation - Detail (`xr_vendorevaldetail`)

A child record holding the complete metric profile for a single vendor within an evaluation. Each Evaluation Header links to up to 3 of these records.

| Display Name                             | Data Type                        | Possible Values / Logical Details                                                                                                         |
| :--------------------------------------- | :------------------------------- | :---------------------------------------------------------------------------------------------------------------------------------------- |
| **xr_vendorevaldetailid**                | Unique Identifier                | (Primary Key) GUID                                                                                                                        |
| **Evaluation Detail Number**             | Auto-number                      | (Unique Text Identifier) e.g., `EVALD-{SEQNUM}`                                                                                           |
| **Vendor**                               | Lookup (`xr_vendoronboarding`)   | The vendor being assessed in this record                                                                                                  |
| **Vertical**                             | Lookup (`xr_verticalbudgethead`) | _(Hidden)_ Used to drive UI Business Rule logic                                                                                           |
| **Budget Head**                          | Lookup (`xr_verticalbudgethead`) | _(Hidden)_ Used to drive UI Business Rule logic                                                                                           |
| **Remarks**                              | Multiple Lines of Text           |                                                                                                                                           |
| **Rating**                               | Choice                           | `1`, `2`, `3`, `4`, `5`                                                                                                                   |
| **Pricing**                              | Single Line of Text              | String-based pricing format                                                                                                               |
| **Payment Term**                         | Choice                           | `15 days post submission`, `30 days post submission`, `45 days post submission`, `50% in advance rest post submission`, `100% in advance` |
| **Discount**                             | Single Line of Text              |                                                                                                                                           |
| **Management Fee**                       | Single Line of Text              |                                                                                                                                           |
| **SMS Delivery Ratio**                   | Single Line of Text              | text percentages                                                                                                                          |
| **RCS Delivery Ratio**                   | Single Line of Text              | text percentages                                                                                                                          |
| **Call for connected Ratio**             | Single Line of Text              |                                                                                                                                           |
| **Re-credits**                           | Choice                           | `Yes`, `No`                                                                                                                               |
| **META Availability**                    | Choice                           | `Yes`, `No`                                                                                                                               |
| **Quality**                              | Choice                           | `1`, `2`, `3`, `4`, `5`                                                                                                                   |
| **Quality of Callers**                   | Choice                           | `1`, `2`, `3`, `4`, `5`                                                                                                                   |
| **Quality of Product**                   | Choice                           | `1`, `2`, `3`, `4`, `5`                                                                                                                   |
| **Quality of Work/Service**              | Single Line of Text              |                                                                                                                                           |
| **Quality of previous Work**             | Choice                           | `1`, `2`, `3`, `4`, `5`                                                                                                                   |
| **Finesse**                              | Choice                           | `1`, `2`, `3`, `4`, `5`                                                                                                                   |
| **Previous Clients**                     | Single Line of Text              |                                                                                                                                           |
| **Previous Clients Performance Rating**  | Choice                           | `1`, `2`, `3`, `4`, `5`                                                                                                                   |
| **Previous Clients Performance quality** | Choice                           | `1`, `2`, `3`, `4`, `5`                                                                                                                   |
| **Industry Expertise**                   | Choice                           | `1`, `2`, `3`, `4`, `5`                                                                                                                   |
| **Industry Experience**                  | Single Line of Text              |                                                                                                                                           |
| **Technical Experience**                 | Choice                           | `1`, `2`, `3`, `4`, `5`                                                                                                                   |
| **Category Expertise**                   | Choice                           | `1`, `2`, `3`, `4`, `5`                                                                                                                   |
| **Scale of Budget Handled**              | Choice                           | `upto 30 lakhs`, `30 lakhs to 70 lakhs`, `70 lakhs to 1 crore`, `1 crore & above`                                                         |
| **Timeline of delivery**                 | Choice                           | `1`, `2`, `3`, `4`, `5`                                                                                                                   |
| **On Time Delivery**                     | Choice                           | `1`, `2`, `3`, `4`, `5`                                                                                                                   |
| **On Time Delivery capability**          | Choice                           | `1`, `2`, `3`, `4`, `5`                                                                                                                   |
| **Product Delivery**                     | Single Line of Text              |                                                                                                                                           |
| **Product**                              | Single Line of Text              |                                                                                                                                           |
| **Product Name**                         | Single Line of Text              |                                                                                                                                           |
| **Location**                             | Single Line of Text              |                                                                                                                                           |
| **Reach Location**                       | Single Line of Text              | Specific location target                                                                                                                  |
| **Availability**                         | Choice                           | `Yes`, `No`                                                                                                                               |
| **Site Availability**                    | Single Line of Text              |                                                                                                                                           |
| **Station Popularity**                   | Single Line of Text              |                                                                                                                                           |
| **Cancellation Policy**                  | Choice                           | `Yes`, `No`                                                                                                                               |

---

## 5. Budget / Expense Requisition (`xr_budgetrequisition`)

Consolidates standard Expense Requisitions and specific Vendor Adjustments.

| Display Name                                            | Data Type                        | Possible Values / Logical Details          |
| :------------------------------------------------------ | :------------------------------- | :----------------------------------------- |
| **xr_budgetrequisitionid**                              | Unique Identifier                | (Primary Key) GUID                         |
| **Requisition Number**                                  | Auto-number                      | (Unique Text Identifier) `REQ-{SEQNUM}`    |
| **Type**                                                | Choice                           | `Expense Requisition`, `Budget Adjustment` |
| **Project**                                             | Lookup (`xr_project`)            |                                            |
| **Vertical**                                            | Lookup (`xr_verticalbudgethead`) |                                            |
| **Budget Head**                                         | Lookup (`xr_verticalbudgethead`) |                                            |
| **Sub Budget Head**                                     | Lookup (`xr_verticalbudgethead`) |                                            |
| **Vendor Name**                                         | Lookup (`xr_vendoronboarding`)   |                                            |
| **Start Date**                                          | Date Only                        |                                            |
| **End Date**                                            | Date Only                        |                                            |
| **Requested Parameter**                                 | Decimal Number                   | Amount or Quantity                         |
| **Description**                                         | Multiple Lines of Text           | "Kindly elaborate on the requirement"      |
| **Period** _(Hidden Field - Conditionally Visible)_     | Choice                           | `Daily`, `Monthly`                         |
| **Event Type** _(Hidden Field - Conditionally Visible)_ | Choice                           | `CP Meet`, `JBP Meet`, `CP Success Meet`   |

---

## 6. Budget Operation Request (`xr_budgetoperation`)

Handles structural shifts, additions, or deductions of budget across hierarchical verticals.

| Display Name             | Data Type             | Possible Values / Logical Details                        |
| :----------------------- | :-------------------- | :------------------------------------------------------- |
| **xr_budgetoperationid** | Unique Identifier     | (Primary Key) GUID                                       |
| **Operation Number**     | Auto-number           | (Unique Text Identifier) `OP-{SEQNUM}`                   |
| **Project**              | Lookup (`xr_project`) |                                                          |
| **Budget Operation**     | Choice                | `Budget Shifting`, `Budget Addition`, `Budget Deduction` |
| **Amount**               | Currency              | Float/Decimal value                                      |

### 6.1 "From" Context _(Visible on Budget Shifting, Budget Addition, and Budget Deduction)_

| Display Name             | Data Type                        | Possible Values / Logical Details |
| :----------------------- | :------------------------------- | :-------------------------------- |
| **From Vertical**        | Lookup (`xr_verticalbudgethead`) |                                   |
| **From Budget Head**     | Lookup (`xr_verticalbudgethead`) |                                   |
| **From Sub Budget Head** | Lookup (`xr_verticalbudgethead`) | Optional selection                |

### 6.2 "To" Context _(Visible ONLY on Budget Shifting)_

| Display Name           | Data Type                        | Possible Values / Logical Details |
| :--------------------- | :------------------------------- | :-------------------------------- |
| **To Vertical**        | Lookup (`xr_verticalbudgethead`) |                                   |
| **To Budget Head**     | Lookup (`xr_verticalbudgethead`) |                                   |
| **To Sub Budget Head** | Lookup (`xr_verticalbudgethead`) | Optional selection                |
