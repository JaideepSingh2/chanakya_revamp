<style>
pre, .mermaid, svg, img {
  page-break-inside: avoid;
  break-inside: avoid;
}
h1, h2, h3, h4 {
  page-break-after: avoid;
}
thead {
  display: table-header-group;
}
</style>

## 0. Existing Tables

These tables already exist in the environment and do not need to be created. They are included here for relationship reference only.

### 0.1 User (`systemuser`)

| Display Name | Data Type         | Possible Values / Logical Details |
| :----------- | :---------------- | :-------------------------------- |
| **User ID**  | Unique Identifier | (Primary Key) GUID                |

### 0.2 Project (`xr_project`)

| Display Name   | Data Type         | Possible Values / Logical Details |
| :------------- | :---------------- | :-------------------------------- |
| **Project ID** | Unique Identifier | (Primary Key) GUID                |

---

I would

## 1. Vertical Budget Head (`xr_verticalbudgethead`)

A unified hierarchical table that manages Verticals, Budget Heads, and Sub Budget Heads.

| Display Name                | Data Type           | Grouping          | Possible Values / Logical Details                                           |
| :-------------------------- | :------------------ | :---------------- | :-------------------------------------------------------------------------- |
| **xr_verticalbudgetheadid** | Unique Identifier   | Identification    | (Primary Key) GUID                                                          |
| **Vertical Budget Code**    | Auto-number         | Identification    | (Unique Text Identifier) `VBHC-{SEQNUM:5}`                                  |
| **Parent Category**         | Lookup              | Hierarchy/Context | Related to `Verticals & Budget Head` table (Self-referencing for hierarchy) |
| **Category Name**           | Single Line of Text | Core Data         | e.g., "Channel Partner", "Telecallers", "Hostess"                           |
| **Category Type**           | Choice              | Workflow/Logic    | `Vertical`, `Budget Head`, `Sub Budget Head`                                |

---

## 2. Project Budget (`xr_projectbudget`)

Stores the allocated budget for each Project + Category (Budget Head) combination, sourced from the existing Project_Wise_Budget list.

| Display Name                  | Data Type                        | Grouping          | Possible Values / Logical Details                                                            |
| :---------------------------- | :------------------------------- | :---------------- | :------------------------------------------------------------------------------------------- |
| **xr_projectbudgetid**        | Unique Identifier                | Identification    | (Primary Key) GUID                                                                           |
| **Project Budget Code**       | Auto-number                      | Identification    | (Unique Text Identifier) `PJBG-{SEQNUM:5}`                                                   |
| **Project**                   | Lookup (`xr_project`)            | Hierarchy/Context | References `xr_project.Project ID` (GUID); `ProjectName` may be shown in the UI as the label |
| **Vertical**                  | Lookup (`xr_verticalbudgethead`) | Hierarchy/Context | Filtered to `Category Type = Vertical`                                                       |
| **Budget Head**               | Lookup (`xr_verticalbudgethead`) | Hierarchy/Context | Filtered to children of selected Vertical                                                    |
| **Region**                    | Choice                           | Hierarchy/Context | e.g. `MMR`                                                                                   |
| **Budget (Allocated Amount)** | Currency                         | Core Data         | Base sanctioned amount                                                                       |
| **Approved Amount**           | Currency                         | Core Data         | Officially approved spend                                                                    |
| **Consumption**               | Currency                         | Core Data         | Amount utilized                                                                              |
| **Balance (In Rs)**           | Calculated (Currency)            | Core Data         | Native Calculation: `Budget` - `Consumption`                                                 |
| **Requestor**                 | Lookup (`systemuser`)            | Audit/Tracking    | Initiator of the request                                                                     |
| **Approver**                  | Lookup (`systemuser`)            | Audit/Tracking    | User who approved/rejected                                                                   |
| **Request Date & Time**       | Date and Time                    | Audit/Tracking    |                                                                                              |
| **Action Date & Time**        | Date and Time                    | Audit/Tracking    | Timestamp of approval/rejection                                                              |
| **Approver Remarks**          | Multiple Lines of Text           | Audit/Tracking    | Notes from the approver                                                                      |

---

## 3. Vendor Onboarding (`xr_vendoronboarding`)

Maintains the centralized database of all onboarded vendors based on the onboarding form.

| Display Name               | Data Type                        | Grouping          | Possible Values / Logical Details                       |
| :------------------------- | :------------------------------- | :---------------- | :------------------------------------------------------ |
| **xr_vendoronboardingid**  | Unique Identifier                | Identification    | (Primary Key) GUID                                      |
| **Vendor Onboarding Code** | Auto-number                      | Identification    | (Unique Text Identifier) `VDON-{SEQNUM:5}`              |
| **Company Name**           | Single Line of Text              | Identification    |                                                         |
| **Vertical**               | Lookup (`xr_verticalbudgethead`) | Hierarchy/Context | Filtered to `Category Type = Vertical`                  |
| **Budget Head**            | Lookup (`xr_verticalbudgethead`) | Hierarchy/Context | Filtered to children of selected Vertical               |
| **Sub Budget Head**        | Lookup (`xr_verticalbudgethead`) | Hierarchy/Context | Filtered to children of selected Budget Head (Optional) |
| **Email ID**               | Single Line of Text              | Core Data         | Format: Email (Regex validated in UI)                   |
| **Contact Name**           | Single Line of Text              | Core Data         |                                                         |
| **Phone Number**           | Single Line of Text              | Core Data         | 10 digits                                               |
| **Payment Credit Period**  | Single Line of Text              | Core Data         | No. of Days representation                              |
| **GSTIN**                  | Single Line of Text              | Core Data         | 15 characters                                           |
| **PAN**                    | Single Line of Text              | Core Data         | Extracted from GSTIN (chars 3 to 12) or 10 chars        |
| **Billing Address**        | Multiple Lines of Text           | Core Data         |                                                         |
| **Billing City**           | Single Line of Text              | Core Data         |                                                         |
| **Billing State**          | Single Line of Text              | Core Data         |                                                         |
| **Billing Pin Code**       | Single Line of Text              | Core Data         |                                                         |
| **Approval Status**        | Choice                           | Workflow/Logic    | `Pending`, `Approved`, `Rejected`                       |
| **Requestor**              | Lookup (`systemuser`)            | Audit/Tracking    | Initiator of the request                                |
| **Approver**               | Lookup (`systemuser`)            | Audit/Tracking    | User who approved/rejected                              |
| **Request Date & Time**    | Date and Time                    | Audit/Tracking    |                                                         |
| **Action Date & Time**     | Date and Time                    | Audit/Tracking    | Timestamp of approval/rejection                         |
| **Approver Remarks**       | Multiple Lines of Text           | Audit/Tracking    | Notes from the approver                                 |

---

## 4. Vendor Evaluation (`xr_vendorevaluation`)

The parent evaluation record. Holds the overall context (project, category, decision) and links to up to 3 vendor detail records.

> **Two-field approval pattern:** `Preferred Vendor` is set by the **Vertical POC** (their recommendation). `Final Selected Vendor` is set by the **Strategy Manager** during approval — it defaults to the POC's pick but can be overridden. `Evaluated vendor` is then resolved from `Final Selected Vendor`.

| Display Name               | Data Type                        | Grouping          | Possible Values / Logical Details                                                                                                                |
| :------------------------- | :------------------------------- | :---------------- | :----------------------------------------------------------------------------------------------------------------------------------------------- |
| **xr_vendorevaluationid**  | Unique Identifier                | Identification    | (Primary Key) GUID                                                                                                                               |
| **Vendor Evaluation Code** | Auto-number                      | Identification    | (Unique Text Identifier) `VNEV-{SEQNUM:5}`                                                                                                       |
| **Project**                | Lookup (`xr_project`)            | Hierarchy/Context |                                                                                                                                                  |
| **Vertical**               | Lookup (`xr_verticalbudgethead`) | Hierarchy/Context | Filtered to `Category Type = Vertical`                                                                                                           |
| **Budget Head**            | Lookup (`xr_verticalbudgethead`) | Hierarchy/Context | Filtered to children of selected Vertical                                                                                                        |
| **Sub Budget Head**        | Lookup (`xr_verticalbudgethead`) | Hierarchy/Context | Filtered to children of Budget Head (if available)                                                                                               |
| **Vendor 1 Details**       | Lookup (`xr_vendorevaldetail`)   | Core Data         | Links to the evaluation detail record for Vendor 1                                                                                               |
| **Vendor 2 Details**       | Lookup (`xr_vendorevaldetail`)   | Core Data         | Links to the evaluation detail record for Vendor 2                                                                                               |
| **Vendor 3 Details**       | Lookup (`xr_vendorevaldetail`)   | Core Data         | Links to the evaluation detail record for Vendor 3                                                                                               |
| **Preferred Vendor**       | Choice                           | Workflow/Logic    | `Vendor 1`, `Vendor 2`, `Vendor 3`. Set by **Vertical POC**. Read-only after submission.                                                         |
| **Reason for Preference**  | Multiple Lines of Text           | Workflow/Logic    | POC's justification for their recommendation. Read-only after submission.                                                                        |
| **Final Selected Vendor**  | Choice                           | Workflow/Logic    | `Vendor 1`, `Vendor 2`, `Vendor 3`. Set by **Strategy Manager** during approval. Defaults to `Preferred Vendor`. Required before Approve action. |
| **Approval Status**        | Choice                           | Workflow/Logic    | `Pending`, `Approved`, `Rejected`. Default = `Pending`. Drives button visibility and form locking.                                               |
| **Evaluated vendor**       | Lookup (`xr_vendoronboarding`)   | Workflow/Logic    | Auto-resolved from `Final Selected Vendor` on Approval (via Power Fx or Plugin). Read-only.                                                      |
| **Requestor**              | Lookup (`systemuser`)            | Audit/Tracking    | Initiator of the request                                                                                                                         |
| **Approver**               | Lookup (`systemuser`)            | Audit/Tracking    | User who approved/rejected                                                                                                                       |
| **Request Date & Time**    | Date and Time                    | Audit/Tracking    |                                                                                                                                                  |
| **Action Date & Time**     | Date and Time                    | Audit/Tracking    | Timestamp of approval/rejection                                                                                                                  |
| **Approver Remarks**       | Multiple Lines of Text           | Audit/Tracking    | Notes from the approver. **Required before Reject action.**                                                                                      |

---

## 5. Vendor Eval Detail (`xr_vendorevaldetail`)

A child record holding the complete metric profile for a single vendor within an evaluation. Each Evaluation Header links to up to 3 of these records.

| Display Name                             | Data Type                        | Grouping          | Possible Values / Logical Details                                                                                                         |
| :--------------------------------------- | :------------------------------- | :---------------- | :---------------------------------------------------------------------------------------------------------------------------------------- |
| **xr_vendorevaldetailid**                | Unique Identifier                | Identification    | (Primary Key) GUID                                                                                                                        |
| **Vendor Eval Detail Code**              | Auto-number                      | Identification    | (Unique Text Identifier) `VEDT-{SEQNUM:5}`                                                                                                |
| **Vendor**                               | Lookup (`xr_vendoronboarding`)   | Hierarchy/Context | The vendor being assessed in this record                                                                                                  |
| **Vertical**                             | Lookup (`xr_verticalbudgethead`) | Hierarchy/Context | _(Hidden)_ Used to drive UI Business Rule logic                                                                                           |
| **Budget Head**                          | Lookup (`xr_verticalbudgethead`) | Hierarchy/Context | _(Hidden)_ Used to drive UI Business Rule logic                                                                                           |
| **Rating**                               | Choice                           | Core Data         | `1`, `2`, `3`, `4`, `5`                                                                                                                   |
| **Pricing**                              | Single Line of Text              | Core Data         | String-based pricing format                                                                                                               |
| **Discount**                             | Single Line of Text              | Core Data         |                                                                                                                                           |
| **Management Fee**                       | Single Line of Text              | Core Data         |                                                                                                                                           |
| **Payment Term**                         | Choice                           | Workflow/Logic    | `15 days post submission`, `30 days post submission`, `45 days post submission`, `50% in advance rest post submission`, `100% in advance` |
| **SMS Delivery Ratio**                   | Single Line of Text              | Core Data         | text percentages                                                                                                                          |
| **RCS Delivery Ratio**                   | Single Line of Text              | Core Data         | text percentages                                                                                                                          |
| **Call for connected Ratio**             | Single Line of Text              | Core Data         |                                                                                                                                           |
| **Re-credits**                           | Choice                           | Core Data         | `Yes`, `No`                                                                                                                               |
| **META Availability**                    | Choice                           | Core Data         | `Yes`, `No`                                                                                                                               |
| **Quality**                              | Choice                           | Core Data         | `1`, `2`, `3`, `4`, `5`                                                                                                                   |
| **Quality of Callers**                   | Choice                           | Core Data         | `1`, `2`, `3`, `4`, `5`                                                                                                                   |
| **Quality of Product**                   | Choice                           | Core Data         | `1`, `2`, `3`, `4`, `5`                                                                                                                   |
| **Quality of Work/Service**              | Single Line of Text              | Core Data         |                                                                                                                                           |
| **Quality of previous Work**             | Choice                           | Core Data         | `1`, `2`, `3`, `4`, `5`                                                                                                                   |
| **Finesse**                              | Choice                           | Core Data         | `1`, `2`, `3`, `4`, `5`                                                                                                                   |
| **Previous Clients**                     | Single Line of Text              | Core Data         |                                                                                                                                           |
| **Previous Clients Performance Rating**  | Choice                           | Core Data         | `1`, `2`, `3`, `4`, `5`                                                                                                                   |
| **Previous Clients Performance quality** | Choice                           | Core Data         | `1`, `2`, `3`, `4`, `5`                                                                                                                   |
| **Industry Expertise**                   | Choice                           | Core Data         | `1`, `2`, `3`, `4`, `5`                                                                                                                   |
| **Industry Experience**                  | Single Line of Text              | Core Data         |                                                                                                                                           |
| **Technical Experience**                 | Choice                           | Core Data         | `1`, `2`, `3`, `4`, `5`                                                                                                                   |
| **Category Expertise**                   | Choice                           | Core Data         | `1`, `2`, `3`, `4`, `5`                                                                                                                   |
| **Scale of Budget Handled**              | Choice                           | Core Data         | `upto 30 lakhs`, `30 lakhs to 70 lakhs`, `70 lakhs to 1 crore`, `1 crore & above`                                                         |
| **Timeline of delivery**                 | Choice                           | Core Data         | `1`, `2`, `3`, `4`, `5`                                                                                                                   |
| **On Time Delivery**                     | Choice                           | Core Data         | `1`, `2`, `3`, `4`, `5`                                                                                                                   |
| **On Time Delivery capability**          | Choice                           | Core Data         | `1`, `2`, `3`, `4`, `5`                                                                                                                   |
| **Product Delivery**                     | Single Line of Text              | Core Data         |                                                                                                                                           |
| **Product**                              | Single Line of Text              | Core Data         |                                                                                                                                           |
| **Product Name**                         | Single Line of Text              | Core Data         |                                                                                                                                           |
| **Location**                             | Single Line of Text              | Core Data         |                                                                                                                                           |
| **Reach Location**                       | Single Line of Text              | Core Data         | Specific location target                                                                                                                  |
| **Availability**                         | Choice                           | Core Data         | `Yes`, `No`                                                                                                                               |
| **Site Availability**                    | Single Line of Text              | Core Data         |                                                                                                                                           |
| **Station Popularity**                   | Single Line of Text              | Core Data         |                                                                                                                                           |
| **Cancellation Policy**                  | Choice                           | Workflow/Logic    | `Yes`, `No`                                                                                                                               |

---

## 6. Budget Requisition (`xr_budgetrequisition`)

Captures standard budget requisitions for approved spend requests.

| Display Name                                        | Data Type                        | Grouping          | Possible Values / Logical Details                                      |
| :-------------------------------------------------- | :------------------------------- | :---------------- | :--------------------------------------------------------------------- |
| **xr_budgetrequisitionid**                          | Unique Identifier                | Identification    | (Primary Key) GUID                                                     |
| **Budget Requisition Code**                         | Auto-number                      | Identification    | (Unique Text Identifier) `BGRQ-{SEQNUM:5}`                             |
| **Project**                                         | Lookup (`xr_project`)            | Hierarchy/Context |                                                                        |
| **Vertical**                                        | Lookup (`xr_verticalbudgethead`) | Hierarchy/Context |                                                                        |
| **Budget Head**                                     | Lookup (`xr_verticalbudgethead`) | Hierarchy/Context | _(Also used for Event Type: `CP Meet`, `JBP Meet`, `CP Success Meet`)_ |
| **Sub Budget Head**                                 | Lookup (`xr_verticalbudgethead`) | Hierarchy/Context | _(Also used for Sub Events)_                                           |
| **Evaluated Vendor**                                | Lookup (`xr_vendoronboarding`)   | Hierarchy/Context | Linked to the Vendor Onboarding table                                  |
| **Start Date & Time**                               | Date and Time                    | Core Data         |                                                                        |
| **End Date & Time**                                 | Date and Time                    | Core Data         |                                                                        |
| **Amount**                                          | Currency                         | Core Data         | Monetary value for the requisition                                     |
| **Quantity**                                        | Decimal Number                   | Core Data         | Numeric count for the requisition                                      |
| **Description**                                     | Multiple Lines of Text           | Core Data         | "Kindly elaborate on the requirement"                                  |
| **Approval Status**                                 | Choice                           | Workflow/Logic    | `Pending`, `Approved`, `Rejected`                                      |
| **Period** _(Hidden Field - Conditionally Visible)_ | Choice                           | Workflow/Logic    | `Daily`, `Monthly`                                                     |
| **Requestor**                                       | Lookup (`systemuser`)            | Audit/Tracking    | Initiator of the request                                               |
| **Approver**                                        | Lookup (`systemuser`)            | Audit/Tracking    | User who approved/rejected                                             |
| **Request Date & Time**                             | Date and Time                    | Audit/Tracking    |                                                                        |
| **Action Date & Time**                              | Date and Time                    | Audit/Tracking    | Timestamp of approval/rejection                                        |
| **Approver Remarks**                                | Multiple Lines of Text           | Audit/Tracking    | Notes from the approver                                                |

---

## 7. Budget Operation (`xr_budgetoperation`)

Handles structural shifts, additions, or deductions of budget across hierarchical verticals.

| Display Name              | Data Type                        | Grouping          | Possible Values / Logical Details                        |
| :------------------------ | :------------------------------- | :---------------- | :------------------------------------------------------- |
| **xr_budgetoperationid**  | Unique Identifier                | Identification    | (Primary Key) GUID                                       |
| **Budget Operation Code** | Auto-number                      | Identification    | (Unique Text Identifier) `BGOP-{SEQNUM:5}`               |
| **Project**               | Lookup (`xr_project`)            | Hierarchy/Context |                                                          |
| **From Vertical**         | Lookup (`xr_verticalbudgethead`) | Hierarchy/Context | Applicable for all operation types                       |
| **From Budget Head**      | Lookup (`xr_verticalbudgethead`) | Hierarchy/Context | Applicable for all operation types                       |
| **From Sub Budget Head**  | Lookup (`xr_verticalbudgethead`) | Hierarchy/Context | Optional selection                                       |
| **To Vertical**           | Lookup (`xr_verticalbudgethead`) | Hierarchy/Context | Applicable only for Budget Shifting                      |
| **To Budget Head**        | Lookup (`xr_verticalbudgethead`) | Hierarchy/Context | Applicable only for Budget Shifting                      |
| **To Sub Budget Head**    | Lookup (`xr_verticalbudgethead`) | Hierarchy/Context | Optional selection                                       |
| **Amount**                | Currency                         | Core Data         | Float/Decimal value                                      |
| **Budget Operation Type** | Choice                           | Workflow/Logic    | `Budget Shifting`, `Budget Addition`, `Budget Deduction` |
| **Approval Status**       | Choice                           | Workflow/Logic    | `Pending`, `Approved`, `Rejected`                        |
| **Requestor**             | Lookup (`systemuser`)            | Audit/Tracking    | Initiator of the request                                 |
| **Approver**              | Lookup (`systemuser`)            | Audit/Tracking    | User who approved/rejected                               |
| **Request Date & Time**   | Date and Time                    | Audit/Tracking    |                                                          |
| **Action Date & Time**    | Date and Time                    | Audit/Tracking    | Timestamp of approval/rejection                          |
| **Approver Remarks**      | Multiple Lines of Text           | Audit/Tracking    | Notes from the approver                                  |
