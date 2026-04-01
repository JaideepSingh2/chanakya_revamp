# Chanakya 2.0: Model-Driven App (MDA) Frontend Build Guide

This is a step-by-step guide to building the complete frontend of the Chanakya 2.0 application inside Power Apps. It covers the App structure, every Form, every View, Business Rules, and Command Bar actions you need to configure — in the exact order you should build them.

> [!NOTE]
> **What is a Model-Driven App?** Unlike Canvas Apps where you design every pixel, Model-Driven Apps (MDAs) auto-generate your UI from your Dataverse table structure. Your job is to configure *Forms* (create/edit screens), *Views* (list screens), and assemble them into an App via the *Sitemap* (navigation).

---

## Phase 0: Prerequisites

Before you touch the frontend, these must already be done:

- [x] All 7 custom tables created in Dataverse (see `02_Table_Schemas.md`)
- [x] All 3 Security Roles configured (see `04_Security_Role_Config.md`)
- [x] You have a Solution created in [make.powerapps.com](https://make.powerapps.com)

## Phase 0.5: Recommended Starting Point After Vendor Onboarding

If the **Vendor Onboarding** form is already built, do **not** jump straight into dashboards or polish. Build the rest of the frontend in the order below so each step unlocks the next one cleanly:

1. Build **Vendor Eval Detail** first.
   - This is the dependency for Vendor Evaluation.
   - Finalize which metric sections belong to which Vertical before laying out the form.
2. Build **Vendor Evaluation** next.
   - This completes the vendor selection journey end-to-end.
   - Get the 3-vendor comparison flow working before touching budget screens.
3. Create the core **Approved/Pending/My** views for:
   - Vendor Onboarding
   - Vendor Evaluation
4. Build **Budget Requisition** after Vendor Evaluation.
   - This depends on approved evaluation records being available for lookup.
5. Build **Project Budget** as the read-only financial reference screen.
   - Users need this view before they can confidently raise requisitions or operations.
6. Build **Budget Operation** after the Project Budget screen is available.
   - CPT and Strategy users need budget visibility before shifting or adding funds.
7. Add **Approve / Reject command bar actions** only after all transactional forms and views are stable.
8. Add **Dashboard, charts, and final polish** at the very end.

> [!TIP]
> For your current stage, the best immediate next deliverable is:
> **Vendor Eval Detail -> Vendor Evaluation -> related Views**

## Frontend Optimization Notes

The original guide is workable, but these adjustments will save rework:

1. **Do not rely on Business Rules for dependent lookup filtering.**
   - Business Rules can show, hide, lock, and set values.
   - For dynamic lookup filtering like `Vertical -> Budget Head -> Sub Budget Head` or filtering `Evaluated Vendor`, use **JavaScript form scripts** with `addPreSearch` / custom filters, or configure filtered lookup views where possible.
2. **Use separate forms for requesters and approvers on approval-heavy tables if needed.**
   - A single form is fine to start.
   - But if CPT/Strategy review becomes cluttered, create a compact **Approval Form** for `xr_vendoronboarding`, `xr_vendorevaluation`, `xr_budgetrequisition`, and `xr_budgetoperation`.
   - For **Vendor Evaluation**, this split is strongly recommended because the Vertical user prepares the request, but the Strategy user performs the actual evaluation decision.
3. **Treat `xr_vendorevaldetail` as a guided data-entry experience, not a flat long form.**
   - Group fields into vertical-specific sections.
   - Hide irrelevant sections aggressively so the form does not become overwhelming.
4. **Delay dashboards and charts until the transactional experience is working.**
   - Forms, views, lookup filters, and approval actions matter more than homepage polish.
5. **Lock down read-only financial screens early.**
   - `xr_projectbudget` should behave like a reference screen, not an editable form for normal users.

---

## Phase 1: Create the Model-Driven App

You will create a **single app** — **`Chanakya`** — that serves all three roles (Vertical POC, CPT, and Strategy SPOC). Security Roles will automatically restrict what each user can see and do within this one app.

1. Go to [make.powerapps.com](https://make.powerapps.com) → Select your **Solution**.
2. Click **New** → **App** → **Model-driven app**.
3. Name it: **`Chanakya`**.
4. Click **Create**. The App Designer will open.

> [!NOTE]
> You will build all Forms and Views first as standalone components inside the Solution, and then assemble them into the app via the Sitemap at the end.

---

## Phase 2: Build the Forms (Create/Edit Screens)

A **Form** in an MDA is what a user sees when they open or create a record. For each table, you will build at minimum a **Main Form** (the standard create/edit form).

Navigate to your table inside the Solution → **Forms** → **New** or click an existing **Main** form to edit.

---

### Form 1: Vendor Onboarding (`xr_vendoronboarding`)

**Purpose:** Used by the Vertical POC to register a new vendor.

#### Header Section (always visible at the top of the form)
| Field | Type | Notes |
| :--- | :--- | :--- |
| Company Name | Text | Make this the form **Title** field |
| Vendor Onboarding Code | Auto-number | Read-only. Set in form properties |
| Approval Status | Choice | Read-only on this form. Default = `Pending` |

#### Tab 1: "Vendor Details"
**Section 1.1 – Classification**
| Field | Type | Notes |
| :--- | :--- | :--- |
| Vertical | Lookup | Filtered to `Category Type = Vertical` |
| Budget Head | Lookup | Filtered to children of selected Vertical (Business Rule) |
| Sub Budget Head | Lookup | Filtered to children of selected Budget Head (Business Rule, Optional) |

**Section 1.2 – Contact Information**
| Field | Type | Notes |
| :--- | :--- | :--- |
| Contact Name | Text | Required |
| Email ID | Text | Required. Format: Email |
| Phone Number | Text | Required. 10-digit validation |
| Payment Credit Period | Text | No. of days |

**Section 1.3 – Tax & Legal**
| Field | Type | Notes |
| :--- | :--- | :--- |
| GSTIN | Text | Required. 15 characters exactly |
| PAN | Text | Auto-populated from GSTIN (chars 3-12). Read-only after GSTIN entry |

**Section 1.4 – Billing Address**
| Field | Type | Notes |
#### Step 1: Form Structure (Layout Settings)
For a premium look that balances screen space:
1.  Set the **Tab** (`Vendor Details`) to **1 column** layout.
2.  Set each **Section** (Classification, etc.) to **2 columns** internally under its Formatting properties.

#### Step 2: Header Configuration (Top Strip)
Drag these fields into the form header:
- **Company Name:** Set as **Title**. (Input will also be available on the form body).
- **Approval Status:** Locked/Read-only.
- **Vendor Onboarding Code:** Locked/Read-only.

---

#### Step 3: Tab 1 — "Vendor Details" (Field Arrangement)

Arrange your sections and fields exactly as follows:

**Section 1: "Classification" (2 Columns)**
- **Left:** Vertical (Required)
- **Right:** Budget Head (Required)
- **Left (Row 2):** Sub Budget Head

**Section 2: "Contact Information" (2 Columns)**
- **Left:** Company Name (Required)
- **Right:** Phone Number (Required)
- **Left (Row 2):** Contact Name (Required)
- **Right (Row 2):** Payment Credit Period
- **Left (Row 3):** Email ID (Required)

**Section 3: "PAN and GST Details" (2 Columns)**
- **Left:** GSTIN (Required)
- **Right:** PAN (Read-only)

**Section 4: "Billing Address" (2 Columns)**
- **Left:** Billing Address (Multi-line)
- **Right:** Billing City
- **Left (Row 2):** Billing Pin Code
- **Right (Row 2):** Billing State

---

#### Step 4: Tab 2 — "Approval Details" (Read-Only)
Create a new 1-column tab and add these 5 audit fields. **Crucially, set all of these to Read-only:**
- Requestor
- Request Date & Time
- Approver
- Action Date & Time
- Approver Remarks

---

#### Step 5: Save and Publish
1.  Click **Save**.
2.  Click **Publish** to make the form live for users.

---

#### Step 6: Configure Business Rules
After publishing the form, go to the table's **Business Rules** tab to add the following logic:

1.  **Auto-filter Budget Head:** Triggered by `Vertical`. Use **JavaScript lookup filtering**, not a Business Rule, to show only children of the selected Vertical.
2.  **Auto-filter Sub Budget Head:** Triggered by `Budget Head`. Use **JavaScript lookup filtering** to show only children of the selected Budget Head.
3.  **Lock Approval fields:** If `Approval Status = Pending`, lock all fields in the "Approval Details" tab.
4.  **PAN Visibility:** If `GSTIN` is blank, you can hide the `PAN` field. (Extraction logic usually requires a Plugin or Javascript).

---

### Form 2: Vendor Eval Detail (`xr_vendorevaldetail`)

**Purpose:** This is a child form used to capture one vendor's metrics. It is filled out 3 times per evaluation.

> [!IMPORTANT]
> This form is NOT directly accessed via the sitemap. It is opened from within the **Vendor Evaluation** form as an embedded Quick Create or sub-form. Build it first because the Vendor Evaluation form depends on it.

#### Form 3A: Vendor Evaluation - Request Form (Vertical POC)

#### Header Section
| Field | Notes |
| :--- | :--- |
| Vendor | Lookup to `xr_vendoronboarding`. This is the form Title. Required. |
| Vendor Eval Detail Code | Auto-number. Read-only. |

#### Tab 1: "Commercial Terms"
| Field | Type | Notes |
| :--- | :--- | :--- |
| Pricing | Text | String-based pricing entry |
| Discount | Text | |
| Management Fee | Text | |
| Payment Term | Choice | `15 days`, `30 days`, etc. |

#### Tab 2: "Performance Metrics"

**Section – Communication/Digital**
| Field | Type |
| :--- | :--- |
| SMS Delivery Ratio | Text |
| RCS Delivery Ratio | Text |
| Call for Connected Ratio | Text |
| Re-credits | Choice (Yes/No) |
| META Availability | Choice (Yes/No) |

**Section – Quality Ratings** *(All are 1–5 Choice fields)*
| Field |
| :--- |
| Rating (Overall) |
| Quality |
| Quality of Callers |
| Quality of Product |
| Quality of Work/Service |
| Quality of Previous Work |
| Finesse |

**Section – Experience & Track Record**
| Field | Type |
| :--- | :--- |
| Previous Clients | Text |
| Previous Clients Performance Rating | Choice (1–5) |
| Previous Clients Performance Quality | Choice (1–5) |
| Industry Expertise | Choice (1–5) |
| Industry Experience | Text |
| Technical Experience | Choice (1–5) |
| Category Expertise | Choice (1–5) |
| Scale of Budget Handled | Choice |

**Section – Delivery & Location**
| Field | Type |
| :--- | :--- |
| Timeline of Delivery | Choice (1–5) |
| On Time Delivery | Choice (1–5) |
| On Time Delivery Capability | Choice (1–5) |
| Product Delivery | Text |
| Product | Text |
| Product Name | Text |
| Location | Text |
| Reach Location | Text |
| Availability | Choice (Yes/No) |
| Site Availability | Text |
| Station Popularity | Text |
| Cancellation Policy | Choice (Yes/No) |
| Remarks | Multi-line Text |

#### Business Rules / Form Logic:
1. **Show/Hide vertical-specific sections:** The many fields in this form may not all apply to every vertical (e.g., "SMS Delivery Ratio" only applies to telecaller verticals).
   - **Trigger:** `Vertical` field value
   - **Action:** Show/Hide entire sections based on which Vertical is selected
   - *(You will configure this once you know exactly which fields map to which vertical)*

---

### Form 3: Vendor Evaluation (`xr_vendorevaluation`)

**Purpose:** Used by the Vertical POC to create a vendor evaluation request for a specific budget context by attaching up to 3 vendor comparison records. The Strategy user then reviews those details, performs the actual evaluation, selects the winning vendor, and approves or rejects the request.

> [!IMPORTANT]
> The role split for this module is:
> - **Vertical POC:** Creates the request and attaches the 3 vendor detail records
> - **Strategy:** Reviews the request, evaluates the vendors, sets the decision fields, and approves/rejects
>
> For this table, create **2 Main forms**:
> 1. **Vendor Evaluation - Request Form** for the Vertical POC
> 2. **Vendor Evaluation - Strategy Review Form** for the Strategy team

#### Header Section
| Field | Notes |
| :--- | :--- |
| Vendor Evaluation Code | Auto-number. Read-only. This is the Title field. |
| Approval Status | Read-only. Default = `Pending` |

#### Tab 1: "Evaluation Request"
**Section 1.1 – Project Context**
| Field | Type | Notes |
| :--- | :--- | :--- |
| Project | Lookup (`xr_project`) | Required |
| Vertical | Lookup | Filtered to `Category Type = Vertical` |
| Budget Head | Lookup | Filtered to children of selected Vertical |
| Sub Budget Head | Lookup | Optional |

**Section 1.2 – Vendor Comparison**
| Field | Type | Notes |
| :--- | :--- | :--- |
| Vendor 1 Details | Lookup (`xr_vendorevaldetail`) | Required. User clicks "New" to create a new detail record |
| Vendor 2 Details | Lookup (`xr_vendorevaldetail`) | Required |
| Vendor 3 Details | Lookup (`xr_vendorevaldetail`) | Required |

**Section 1.3 – Decision**
| Field | Type | Notes |
| :--- | :--- | :--- |
| Preferred Vendor | Choice | Keep on the table, but hide it on the Vertical POC Request Form |
| Reason for Preference | Multi-line Text | Keep on the table, but hide it on the Vertical POC Request Form |
| Final Selected Vendor | Choice | Keep on the table, but hide it on the Vertical POC Request Form |
| Evaluated Vendor | Lookup (`xr_vendoronboarding`) | Read-only. Keep hidden or read-only on the Request Form |

#### Tab 2: "Approval Details" (Read-Only)
*(Same pattern as Vendor Onboarding Approval Tab)*

#### Form 3B: Vendor Evaluation - Strategy Review Form

**Purpose:** Used by Strategy to review the submitted request, inspect the 3 vendor detail records, perform the evaluation, and take the approval decision.

#### Header Section
| Field | Notes |
| :--- | :--- |
| Vendor Evaluation Code | Auto-number. Read-only. This is the Title field. |
| Approval Status | Read-only until Strategy uses Approve/Reject |

#### Tab 1: "Evaluation Review"
Use the same three sections as the Request Form, but with a different editability model:

- **Project Context:** Keep `Project`, `Vertical`, `Budget Head`, and `Sub Budget Head` visible but **Read-only**
- **Vendor Comparison:** Keep `Vendor 1 Details`, `Vendor 2 Details`, and `Vendor 3 Details` visible but **Read-only** so Strategy can open each child record and inspect the metrics
- **Strategy Decision:** Show and allow editing for `Preferred Vendor`, `Reason for Preference`, and `Final Selected Vendor`. Keep `Evaluated Vendor` visible as **Read-only**

#### Tab 2: "Approval Details"
Add the standard audit fields and keep them read-only except for `Approver Remarks` where needed.

#### Business Rules:
1. **Auto-filter lookups** (Vertical → Budget Head → Sub Budget Head): Same as Vendor Onboarding form. Use **JavaScript lookup filtering** rather than Business Rules.
2. **Lock Strategy decision fields until all 3 vendors are filled:**
   - **Trigger:** `Vendor 1 Details`, `Vendor 2 Details`, or `Vendor 3 Details` is blank
   - **Action:** Lock `Preferred Vendor`, `Reason for Preference`, and `Final Selected Vendor`
3. **Hide decision fields from the Request Form**
   - On the Vertical POC Request Form, hide `Preferred Vendor`, `Reason for Preference`, `Final Selected Vendor`, and `Evaluated Vendor`
4. **Make context and vendor comparison fields read-only on the Strategy Review Form**
   - Strategy should evaluate the submitted request, not rewrite the request context

---

### Form 4: Budget Requisition (`xr_budgetrequisition`)

**Purpose:** Used by the Vertical POC to request budget spend (Monthly, Daily, or Direct).

#### Header Section
| Field | Notes |
| :--- | :--- |
| Budget Requisition Code | Auto-number. Title field. |
| Type | Choice: `Expense Requisition` or `Budget Adjustment`. Set on creation. |
| Approval Status | Read-only. Default = `Pending` |

#### Tab 1: "Request Details"
**Section 1.1 – Context**
| Field | Type | Notes |
| :--- | :--- | :--- |
| Project | Lookup | Required |
| Vertical | Lookup | Required |
| Budget Head | Lookup | Required. Filtered to Vertical's children |
| Sub Budget Head | Lookup | Optional |
| Evaluated Vendor | Lookup (`xr_vendorevaluation`) | The approved evaluation for this context |

**Section 1.2 – Requisition**
| Field | Type | Notes |
| :--- | :--- | :--- |
| Period | Choice | `Daily` / `Monthly`. **Hidden by default.** Show only when `Type = Expense Requisition` |
| Start Date & Time | DateTime | Required |
| End Date & Time | DateTime | Required for Monthly |
| Requested Parameter | Decimal | Amount in ₹ or Quantity |
| Description | Multi-line Text | Prompt: *"Kindly elaborate on the requirement"* |

#### Tab 2: "Approval Details" (Read-Only)

#### Business Rules:

1. **Show/Hide `Period` field:**
   - **Trigger:** `Type` changes
   - **Action:** Show `Period` only if `Type = Expense Requisition`; Hide if `Type = Budget Adjustment`

2. **Lock `End Date` for Daily requests:**
   - **Trigger:** `Period = Daily`
   - **Action:** Lock `End Date & Time` (daily requests have no end date)

3. **Auto-filter `Evaluated Vendor`:**
   - **Trigger:** `Sub Budget Head` (or `Budget Head`) is set
   - **Action:** Filter the `Evaluated Vendor` lookup to only show evaluations that match the same `Project`, `Vertical`, and `Budget Head` context
   - **Implementation Note:** Use **JavaScript lookup filtering** for this, not a Business Rule

---

### Form 5: Budget Operation (`xr_budgetoperation`)

**Purpose:** Used by CPT to request a Budget Addition, Deduction, or Shift between budget lines.

#### Header Section
| Field | Notes |
| :--- | :--- |
| Budget Operation Code | Auto-number. Title field. |
| Budget Operation Type | Choice: `Budget Shifting`, `Budget Addition`, `Budget Deduction` |
| Approval Status | Read-only. Default = `Pending` |

#### Tab 1: "Operation Details"
**Section 1.1 – "From" Context** *(Always visible)*
| Field | Type | Notes |
| :--- | :--- | :--- |
| Project | Lookup | Required |
| From Vertical | Lookup | Required |
| From Budget Head | Lookup | Required |
| From Sub Budget Head | Lookup | Optional |
| Amount | Currency | Required |

**Section 1.2 – "To" Context** *(Hidden by default)*
| Field | Type | Notes |
| :--- | :--- | :--- |
| To Vertical | Lookup | Shown only for Budget Shifting |
| To Budget Head | Lookup | Shown only for Budget Shifting |
| To Sub Budget Head | Lookup | Optional |

#### Tab 2: "Approval Details" (Read-Only)

#### Business Rules:

1. **Show/Hide "To" Section:**
   - **Trigger:** `Budget Operation Type` is set
   - **Action:** Show the entire "To Context" section only if `Budget Operation Type = Budget Shifting`
   - **Action:** Hide the "To Context" section for `Budget Addition` and `Budget Deduction`

2. **Lock fields after submission:**
   - **Trigger:** `Approval Status` is NOT `Pending`
   - **Action:** Lock all fields except `Approver Remarks`

---

### Form 6: Project Budget (`xr_projectbudget`)

**Purpose:** This is primarily a **read-only dashboard record** for displaying the financial state of a budget line. It is edited only by the Tech Admin (initial data load) or by Plugins (automated calculations).

> [!TIP]
> **Do not allow the Vertical POC or CPT to edit this form directly.** All changes come through the Requisition and Operation flows. Set all financial fields to **Read-Only** on this form.

#### Tab 1: "Budget Overview"
| Field | Type | Notes |
| :--- | :--- | :--- |
| Project | Lookup | |
| Vertical | Lookup | |
| Budget Head | Lookup | |
| Region | Choice | |
| Budget (Allocated Amount) | Currency | Set by Tech Admin / Plugin. Read-only for users. |
| Approved Amount | Currency | Read-only. Set by Plugin on Requisition Approval. |
| Consumption | Currency | Read-only. Set by Plugin on Daily spend. |
| Balance (In Rs) | Calculated Currency | Read-only. Formula: `Budget - Consumption` |

#### Tab 2: "Audit"
| Field |
| :--- |
| Requestor |
| Request Date & Time |
| Approver |
| Action Date & Time |
| Approver Remarks |

---

## Phase 3: Build the Views (List Screens)

A **View** is a list screen — like a table showing all records that match certain criteria. You will create specific views for each role so they see exactly what is relevant to them.

Navigate to your table inside the Solution → **Views** → **New**.

---

### Views for Vendor Onboarding (`xr_vendoronboarding`)

| View Name | Filter | Columns to Show | Used By |
| :--- | :--- | :--- | :--- |
| **Active Vendor Onboardings** | `Approval Status = Pending` | Code, Company Name, Vertical, Budget Head, Requestor, Request Date | CPT (Approval Queue) |
| **Approved Vendors** | `Approval Status = Approved` | Code, Company Name, Vertical, Budget Head, GSTIN, Approver, Action Date | All roles |
| **My Vendor Requests** | `Requestor = Current User` | Code, Company Name, Vertical, Status, Request Date | Vertical POC |
| **All Vendor Onboardings** | None (All Records) | Code, Company Name, Vertical, Budget Head, Status, Requestor | Strategy, Admin |

---

### Views for Vendor Evaluation (`xr_vendorevaluation`)

| View Name | Filter | Columns to Show | Used By |
| :--- | :--- | :--- | :--- |
| **Pending Evaluations** | `Approval Status = Pending` | Code, Project, Vertical, Budget Head, Requestor, Request Date | Strategy SPOC |
| **Approved Evaluations** | `Approval Status = Approved` | Code, Project, Vertical, Budget Head, Evaluated Vendor, Action Date | Vertical POC, CPT |
| **My Evaluation Requests** | `Requestor = Current User` | Code, Project, Vertical, Status, Request Date | Vertical POC |

---

### Views for Budget Requisition (`xr_budgetrequisition`)

| View Name | Filter | Columns to Show | Used By |
| :--- | :--- | :--- | :--- |
| **Pending Requisitions** | `Approval Status = Pending` | Code, Project, Vertical, Type, Period, Amount, Requestor, Request Date | Strategy SPOC |
| **Approved Requisitions** | `Approval Status = Approved` | Code, Project, Vertical, Type, Amount, Action Date | All |
| **My Requisitions** | `Requestor = Current User` | Code, Project, Vertical, Type, Status, Request Date | Vertical POC |
| **Auto-Approved Daily Spends** | `Type = Expense Requisition` AND `Period = Daily` AND `Status = Approved` | Code, Project, Amount, Requestor, Action Date | Strategy |

---

### Views for Budget Operation (`xr_budgetoperation`)

| View Name | Filter | Columns to Show | Used By |
| :--- | :--- | :--- | :--- |
| **Pending Budget Operations** | `Approval Status = Pending` | Code, Project, Operation Type, From Vertical, Amount, Requestor, Request Date | Strategy SPOC |
| **Approved Budget Operations** | `Approval Status = Approved` | Code, Project, Operation Type, Amount, Approver, Action Date | CPT, Strategy |
| **My Budget Operations** | `Requestor = Current User` | Code, Project, Operation Type, Status, Request Date | CPT |

---

### Views for Project Budget (`xr_projectbudget`)

| View Name | Filter | Columns to Show | Used By |
| :--- | :--- | :--- | :--- |
| **Project Budget Summary** | None | Project, Vertical, Budget Head, Allocated Budget, Approved Amount, Consumption, Balance | All |
| **Low Balance Alert** | `Balance (In Rs) < [Threshold]` | Project, Vertical, Budget Head, Balance | Strategy, Admin |

---

## Phase 4: Add Command Bar Buttons (Approve/Reject Actions)

The standard MDA save button is not enough. Your CPT and Strategy users need **Approve** and **Reject** buttons on the form. You can add these via **Power Fx commands** in the new Command Bar Designer.

### How to Add a Command:

1. Open the App Designer for the **Chanakya** app.
2. Click on the relevant Page (e.g., Vendor Onboarding).
3. Click **"..."** → **Edit command bar** → Select **Main form command bar**.
4. Click **"+ New"** → **Command**.

---

### Commands for Vendor Onboarding (CPT approves)

**Button: "✅ Approve Vendor"**

- **Label:** `Approve Vendor`
- **Action:** Run a Power Automate flow (Cloud Flow) OR use Power Fx:
  ```
  Patch(
    'Vendor Onboardings',
    Self.Selected.Item,
    {
      'Approval Status': 'Vendor Onboardings (Approval Status)'.Approved,
      Approver: LookUp(Users, 'Primary Email' = User().Email),
      'Action Date & Time': Now()
    }
  )
  ```
- **Visibility:** Only show when `Approval Status = Pending` AND current user has the CPT role.

**Button: "❌ Reject Vendor"**

- **Label:** `Reject Vendor`
- **Action:** Same as above but set status to `Rejected`. Also **prompt for remarks** before executing:
  ```
  If(IsBlank(Self.Selected.Item.'Approver Remarks'),
     Notify("Please enter Approver Remarks before rejecting.", NotificationType.Error),
     Patch(...)
  )
  ```
- **Visibility:** Same as Approve button.

---

### Commands for Vendor Evaluation (Strategy approves)

**Button: "✅ Approve Evaluation"**

- Same Power Fx pattern as above, patching `xr_vendorevaluation`.
- **Additional Action:** Trigger a Power Automate flow that populates the `Evaluated Vendor` lookup field based on the `Preferred Vendor` choice (Plugin handles this).

**Button: "❌ Reject Evaluation"**

- Prompt for remarks, then patch status to `Rejected`.

---

### Commands for Budget Requisition (Strategy approves)

**Button: "✅ Approve Requisition"**

- Patch status to `Approved`.
- **Important:** The actual balance deduction is handled by your **C# Plugin** on the status change. The button just updates the status.

**Button: "❌ Reject Requisition"**

- Prompt for remarks, patch to `Rejected`.

---

### Commands for Budget Operation (Strategy approves)

**Same Approve / Reject pattern.** The C# Plugin will handle the actual budget math on the `xr_projectbudget` table.

---

## Phase 5: Assemble the Sitemap (App Navigation)

The **Sitemap** is the left navigation panel of your app. You configure it in the App Designer.

---

### Sitemap: Chanakya App (All Roles)

All three roles (Vertical POC, CPT, Strategy SPOC) use the same single app. The sitemap is organized into **Areas** and **Groups** that logically separate day-to-day work from approvals.

```
📱 Chanakya App
│
├── 🗂️ Area: "My Work"
│   │
│   ├── Group: "Vendor Management"
│   │   ├── 📄 Vendor Onboarding    → View: "My Vendor Requests" (default)
│   │   └── 📄 Vendor Evaluation    → View: "My Evaluations" (default)
│   │
│   └── Group: "Budget & Spending"
│       ├── 📄 Project Budget       → View: "Project Budget Summary"
│       ├── 📄 Budget Requisition   → View: "My Requisitions" (default)
│       └── 📄 Budget Operation     → View: "My Budget Operations" (default)
│
└── 🗂️ Area: "Approvals"
    │
    ├── Group: "Vendor Approvals"
    │   └── 📄 Vendor Onboarding    → View: "Active Vendor Onboardings" (default)
    │
    └── Group: "Financial Approvals"
        ├── 📄 Vendor Evaluation    → View: "Pending Evaluations" (default)
        ├── 📄 Budget Requisition   → View: "Pending Requisitions" (default)
        └── 📄 Budget Operation     → View: "Pending Budget Operations" (default)
```

> [!TIP]
> You do **not** need to hide navigation items per role. Security Roles handle this automatically. A Vertical POC will see the "Approvals" area but the views will be empty (they have no data to approve). CPT will see pending vendor onboardings. Strategy will see pending evaluations, requisitions, and operations.

---

## Phase 6: Dashboards (Optional but Recommended)

A **Dashboard** is a home screen showing multiple views or charts at once. Add a single dashboard to the Chanakya app — it will show relevant tiles to each user based on what they have access to.

### Dashboard: Chanakya Home Dashboard

| Component | Type | Content | Relevant To |
| :--- | :--- | :--- | :--- |
| My Pending Requisitions | List | View: `My Requisitions` (Pending) | Vertical POC |
| Approved Vendor List | List | View: `Approved Vendors` | All |
| Budget Summary | List | View: `Project Budget Summary` | All |
| Pending Vendor Approvals | List | View: `Active Vendor Onboardings` | CPT |
| Pending Evaluations | List | View: `Pending Evaluations` | Strategy SPOC |
| Pending Requisitions | List | View: `Pending Requisitions` | Strategy SPOC |
| Budget Health | Chart | Bar chart: Balance vs Consumption by Vertical | Strategy SPOC |

---

## Phase 7: Final Configuration Checklist

Before publishing the app, verify the following:

### Forms
- [ ] All Lookup filters (Vertical → Budget Head → Sub Budget Head) are working via Business Rules
- [ ] All Approval Detail tabs are fully **Read-Only** (no editing by end-users)
- [ ] `Approval Status` defaults to `Pending` on all transactional forms
- [ ] `Requestor` field auto-sets to `Current User` on all forms
- [ ] `Budget Operation` form correctly shows/hides the "To" section via Business Rule

### Views
- [ ] All Views have the correct filter criteria set
- [ ] All Views are sorted correctly (most recent first using `Request Date & Time DESC`)
- [ ] `Pending` queues show the right columns for the approver

### Commands
- [ ] `Approve` and `Reject` buttons appear only when `Approval Status = Pending`
- [ ] `Reject` button validates that `Approver Remarks` is filled before saving
- [ ] Command visibility is role-appropriate (CPT sees Vendor Approve, Strategy sees Evaluation/Requisition Approve)

### Security & App Settings
- [ ] The `Chanakya` app is added to your Solution
- [ ] Security Roles (`Vertical POC`, `CPT Approval`, `Strategy Manager`) are assigned to the app via **App → Manage Roles** in the Solution
- [ ] Users can open the app without "Access Denied" errors (Basic User role is assigned)

### Publishing
1. In App Designer → Click **Publish** in the top-right.
2. After publishing, copy the **App URL** and distribute to your users.
3. Test each role by logging in as a test user with that specific security role.

---

## Summary: Build Order

Follow this sequence to avoid dependency errors:

1. ✅ Build **Form: Vendor Eval Detail** (first dependency after Vendor Onboarding)
2. ✅ Build **Form: Vendor Evaluation** (depends on Eval Detail form)
3. ✅ Build **Views** for Vendor Onboarding and Vendor Evaluation
4. ✅ Build **Form: Budget Requisition** (depends on approved Vendor Evaluation records)
5. ✅ Build **Form: Project Budget** (read-only reference screen)
6. ✅ Build **Form: Budget Operation** (works best after Project Budget is available)
7. ✅ Build remaining **Views** for Requisition, Operation, and Project Budget
8. ✅ Create the **Chanakya App** and assemble the Sitemap
9. ✅ Add **Command Bars** (Approve/Reject) to the app
10. ✅ Add the **Dashboard** only after the workflows are stable
11. ✅ **Assign Security Roles** to the app
12. ✅ **Publish** the app and test with each role
