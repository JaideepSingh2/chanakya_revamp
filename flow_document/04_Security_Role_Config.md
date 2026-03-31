# Security Role Configuration & Data Isolation Guide

This guide provides the exact implementation steps to configure Dataverse security so that **Vertical POCs** can seamlessly collaborate on records within their vertical, while remaining completely isolated from other verticals in the same Project Business Unit.

## Architecture Concept: The Owner Team Strategy
Since Dataverse Security Roles natively filter by **Ownership** (User/Team) and not by column values (like `Vertical = Marketing`), we will use the **Owner Team Strategy**. 

Instead of an individual user owning a Vendor record, a **Team** will own it. Any user added to that Team automatically inherits read/write access to the Team's records.

---

## Step 0: The Basic User Prerequisite
To ensure your users can open the Model-Driven App without crashing, they need Read access to dozens of hidden core system tables (User, Team, Business Unit, Saved Views, etc.). Do not build this from scratch!

**Rule:** When you onboard a new employee, you must assign them the standard Microsoft **"Basic User"** security role first (Note: This is the standard role for all Dataverse users, **NOT** the "Dynamics 365 App for Outlook User" role). Then, assign them the **`Chanakya - Vertical POC`** role second. The `Vertical POC` role will only govern the custom app tables.

## Step 1: Create the Security Role within a Solution

To ensure your security configuration is deployable across environments (Development -> UAT -> Production), you must create the security role inside a **Solution**.

1.  Log in to [make.powerapps.com](https://make.powerapps.com).
2.  Select **Solutions** from the left navigation and open your **Chanakya 2.0** solution.
3.  Click **New** > **Security** > **Security role**.
4.  Enter the Name: **`Vertical POC`**.
5.  **Select the Business Unit:** Choose your **Root Business Unit** (the top-level BU of your environment).
    > [!IMPORTANT]
    > **Why Create in Root?** Security roles are inherited downwards. By creating the role in the Root BU, it becomes automatically available for every Project-specific Business Unit and Team you create later.
6.  Click **Save**. The security role editor will open in a new tab.

## Step 2: Configure Table Privileges (Security Roles Matrix)

In the security role editor, navigate to the **Custom Tables** tab. You need to configure three distinct roles based on the Chanakya workflow.

### 1. Vertical POC Role
This role is for the people executing the work (Marketing, Sales, CPT local reps) who onboard vendors and consume budgets. The key for data isolation here is using the 🟡 **User (Basic)** access level for transactional tables.

| Table Name | Read | Create | Write | Delete | Append | Append To | Assign | Share |
| :--- | :--- | :--- | :--- | :--- | :--- | :--- | :--- | :--- |
| **Vendor Onboarding** | 🟡 User | 🟡 User | 🟡 User | 🚫 None | 🟡 User | 🟡 User | 🚫 None | 🚫 None |
| **Vendor Evaluation** | 🟡 User | 🟡 User | 🟡 User | 🚫 None | 🟡 User | 🟡 User | 🚫 None | 🚫 None |
| **Vendor Eval Detail**| 🟡 User | 🟡 User | 🟡 User | 🚫 None | 🟡 User | 🟡 User | 🚫 None | 🚫 None |
| **Budget Requisition**| 🟡 User | 🟡 User | 🟡 User | 🚫 None | 🟡 User | 🟡 User | 🚫 None | 🚫 None |
| **Budget Operation**  | 🚫 None | 🚫 None | 🚫 None | 🚫 None | 🚫 None | 🚫 None | 🚫 None | 🚫 None |
| **Vertical Budget Head**| 🟢 Org | 🚫 None | 🚫 None | 🚫 None | 🟢 Org | 🟢 Org | 🚫 None | 🚫 None |
| **Project Budget**    | 🟥 PC | 🚫 None | 🚫 None | 🚫 None | 🟥 PC | 🟥 PC | 🚫 None | 🚫 None |
| **Project**           | 🟥 PC | 🚫 None | 🚫 None | 🚫 None | 🟥 PC | 🟥 PC | 🚫 None | 🚫 None |

> [!IMPORTANT]
> The **User** (Basic) level ensures that a Vertical POC can *only* interact with records they explicitly own, or records owned by a Team they are a member of. If you set this to Business Unit (BU), they will see everything in the Project.

### 2. Strategy SPOC Role
This role is for the primary decision-makers who bifurcate budgets and approve vendors and expenses across their assigned projects. **Parent: Child (PC) 🟥** access is used so they can see all records flowing up from the child Vertical Teams under their Project BU.

| Table Name | Read | Create | Write | Delete | Append | Append To | Assign | Share |
| :--- | :--- | :--- | :--- | :--- | :--- | :--- | :--- | :--- |
| **Vendor Onboarding** | 🟥 PC | 🚫 None | 🚫 None | 🚫 None | 🟥 PC | 🟥 PC | 🚫 None | 🚫 None |
| **Vendor Evaluation** | 🟥 PC | 🚫 None | 🟥 PC | 🚫 None | 🟥 PC | 🟥 PC | 🚫 None | 🚫 None |
| **Vendor Eval Detail**| 🟥 PC | 🚫 None | 🚫 None | 🚫 None | 🟥 PC | 🟥 PC | 🚫 None | 🚫 None |
| **Budget Requisition**| 🟥 PC | 🚫 None | 🟥 PC | 🚫 None | 🟥 PC | 🟥 PC | 🚫 None | 🚫 None |
| **Budget Operation**  | 🟥 PC | 🚫 None | 🟥 PC | 🚫 None | 🟥 PC | 🟥 PC | 🚫 None | 🚫 None |
| **Vertical Budget Head**| 🟢 Org | 🚫 None | 🚫 None | 🚫 None | 🟢 Org | 🟢 Org | 🚫 None | 🚫 None |
| **Project Budget**    | 🟥 PC | 🟥 PC | 🟥 PC | 🚫 None | 🟥 PC | 🟥 PC | 🚫 None | 🚫 None |
| **Project**           | 🟥 PC | 🚫 None | 🚫 None | 🚫 None | 🟥 PC | 🟥 PC | 🚫 None | 🚫 None |

### 3. CPT (Central Procurement POC) Role
This role is for the Procurement members responsible for approving vendors and handling budget shifts for their assigned projects. Since they are also project-specific, they use **Parent: Child (Deep) 🟥** access to oversee their assigned project's Vertical Teams.

| Table Name | Read | Create | Write | Delete | Append | Append To | Assign | Share |
| :--- | :--- | :--- | :--- | :--- | :--- | :--- | :--- | :--- |
| **Vendor Onboarding** | 🟥 PC | 🚫 None | 🟥 PC | 🚫 None | 🟥 PC | 🟥 PC | 🚫 None | 🚫 None |
| **Vendor Evaluation** | 🟥 PC | 🚫 None | 🚫 None | 🚫 None | 🟥 PC | 🟥 PC | 🚫 None | 🚫 None |
| **Vendor Eval Detail**| 🟥 PC | 🚫 None | 🚫 None | 🚫 None | 🟥 PC | 🟥 PC | 🚫 None | 🚫 None |
| **Budget Requisition**| 🚫 None | 🚫 None | 🚫 None | 🚫 None | 🚫 None | 🚫 None | 🚫 None | 🚫 None |
| **Budget Operation**  | 🟥 PC | 🟥 PC | 🟥 PC | 🚫 None | 🟥 PC | 🟥 PC | 🚫 None | 🚫 None |
| **Vertical Budget Head**| 🟢 Org | 🚫 None | 🚫 None | 🚫 None | 🟢 Org | 🟢 Org | 🚫 None | 🚫 None |
| **Project Budget**    | 🟥 PC | 🚫 None | 🟥 PC | 🚫 None | 🟥 PC | 🟥 PC | 🚫 None | 🚫 None |
| **Project**           | 🟥 PC | 🚫 None | 🚫 None | 🚫 None | 🟥 PC | 🟥 PC | 🚫 None | 🚫 None |

> [!IMPORTANT]
> **Strict Project Isolation:** By using **Parent: Child (Deep) 🟥** for both Strategy and CPT, you ensure they can only ever see data for projects where they are assigned to the Parent BU. They are now completely walled off from other projects in the environment.

---

## Step 3: Create Vertical Teams per Project

For each Project Business Unit, create Teams corresponding to your verticals using the naming convention **`[Project Name] [Vertical Name]`** (e.g., **`Project Alpha Marketing POC`** or **`Project Alpha CPT POC`**). You can do this manually, or automate it (Recommended).

**Manual Approach:**
1. Go to **Power Platform Admin Center** > Environments > Settings > Users + permissions > **Teams**.
2. Create a new Team:
   - **Team Name:** E.g., `Project Alpha Marketing POC`.
   - **Business Unit:** Select the relevant Project BU.
   - **Team Type:** `Owner`
   - **Member's privilege inheritance:** Select **`Direct User (Basic) access level and Team privileges`**.
     > [!IMPORTANT]
     > This setting is mandatory for the "Owner Team Strategy" to work with User-level 🟡 access. It allows users to "see" records owned by the Team as if they owned them directly.
   - **Administrator:** Set this to your **System Administrator** user.
3. Assign the **`Vertical POC`** security role to this Team.
4. Add the respective POC users as **Members** of this Team.
5. Repeat the process for Sales, IT, etc.

> [!TIP]
> **Automation Best Practice:** Do not create these teams manually! Build a C# Plugin (or Cloud Flow) that triggers on the creation of a new `xr_project` record. 
> The automation should: 
> 1) Create the new Business Unit for the Project.
> 2) Automatically generate the standard vertical "Owner" Teams (Marketing, Sales, etc.) under that BU.
> 3) Set the **`membershipsource`** and **`membersexcellence`** correctly in your C# code to reflect the inheritance setting above.
> 4) Use the `AssociateRequest` message to bind the **`Vertical POC`** Security Role to all newly generated Teams instantly.
> 
> *Result:* Your Tech Admin only ever has to drop User records into cleanly pre-built Teams!

---

## Step 4: Implement Ownership Routing (C# Plugin)

We must ensure that when a Marketing POC clicks "New Vendor" and saves the record, the resulting record is owned by the *Marketing Team*, not the individual user who clicked the button.

While this can be built with Power Automate, **a Synchronous Pre-Operation C# Plugin is the architectural best practice**. It sets the Owner *before* the record is written to the database, achieving atomicity, preventing UI flicker, and saving secondary API calls.

1. **Register the Plugin:** Create a C# Plugin and register it on the `Create` message, synchronous, **Pre-Operation** stage for the foundational tables: `xr_vendoronboarding`, `xr_vendorevaluation`, and `xr_budgetrequisition`.
2. **Extract the Context:** Inside the `IPlugin` execution context, extract the `Vertical` lookup (`EntityReference`) from the `Target` input entity.
3. **Resolve the Team:** Query Dataverse to determine the corresponding Owner Team GUID for that project/vertical combination.
4. **Mutate the Target:** Update the `ownerid` attribute directly on the `Target` input parameter. Because this is Pre-Operation, Dataverse will save this updated owner to the database natively.
   ```csharp
   target["ownerid"] = new EntityReference("team", targetTeamId);
   ```

> [!TIP]
> **Dynamic Routing (Best Practice):** Instead of keeping a hardcoded mapping in your C# code, add a new lookup column called `Default Owner Team` directly on the **Vertical Budget Head** (`xr_verticalbudgethead`) table. 
> Your plugin simply queries the selected Vertical record, grabs the `Default Owner Team` GUID, and sets the new record's `ownerid` dynamically based on configuration!

---

## Step 4: UI Configuration (App Views)

In your Chanakya Model-Driven App, **you do not need to create custom filtered views** (like "Marketing Vendors View") to enforce this isolation.

Use the standard **"Active Vendor Onboardings"** view without modifying it.
- **For a Marketing POC:** Because Dataverse applies security at the database level *before* showing data on the screen, opening this view will automatically filter out Sales vendors. They will precisely only see the exact list of records owned by the Marketing Team.
- **For a Strategy Manager:** Assuming they possess an `Organization-Level` read role, opening the exact same view will show them all vendors across Marketing, Sales, etc.
