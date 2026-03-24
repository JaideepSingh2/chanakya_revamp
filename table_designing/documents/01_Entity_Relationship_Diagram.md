```mermaid
%%{init: {"theme":"base","themeVariables":{"primaryTextColor":"#0f172a","lineColor":"#64748b","fontFamily":"Segoe UI","background":"#ffffff"},"flowchart":{"curve":"basis","nodeSpacing":110,"rankSpacing":130,"htmlLabels":true}}}%%
flowchart TD

    classDef master fill:#f8fafc,stroke:#334155,stroke-width:1.6px,color:#0f172a;
    classDef entity fill:#ffffff,stroke:#94a3b8,stroke-width:1.4px,color:#0f172a;
    classDef ghost fill:transparent,stroke:transparent,color:transparent;

    %% Master Entities
    PRJ["xr_project<br/>--------------------<br/>Project ID"]:::master
    CAT["xr_verticalbudgethead<br/>--------------------<br/>xr_verticalbudgetheadid<br/>Parent Category"]:::master

    %% Invisible circular anchor to pull the loop into a round curve
    CAT_LOOP(( )):::ghost

    %% Vendor Entities
    VON["xr_vendoronboarding<br/>--------------------<br/>xr_vendoronboardingid<br/>Vertical<br/>Budget Head<br/>Sub Budget Head"]:::entity
    VED["xr_vendorevaldetail<br/>--------------------<br/>xr_vendorevaldetailid<br/>Vendor<br/>Vertical<br/>Budget Head"]:::entity

    %% Transactional / Operation Entities
    PB["xr_projectbudget<br/>--------------------<br/>xr_projectbudgetid<br/>Project<br/>Vertical<br/>Budget Head<br/>Evaluated vendor"]:::entity
    BRQ["xr_budgetrequisition<br/>--------------------<br/>xr_budgetrequisitionid<br/>Project<br/>Vertical<br/>Budget Head<br/>Sub Budget Head<br/>Vendor Name"]:::entity
    VEH["xr_vendorevaluation<br/>--------------------<br/>xr_vendorevaluationid<br/>Project<br/>Vertical<br/>Budget Head<br/>Sub Budget Head<br/>Vendor 1 Details<br/>Vendor 2 Details<br/>Vendor 3 Details"]:::entity
    BOP["xr_budgetoperation<br/>--------------------<br/>xr_budgetoperationid<br/>Project<br/>From Vertical<br/>From Budget Head<br/>From Sub Budget Head<br/>To Vertical<br/>To Budget Head<br/>To Sub Budget Head"]:::entity

    %% Relationships (Grouped by target for cleaner engine routing)

    %% Category Self Loop (Curved via invisible anchor)
    CAT -.->|A| CAT_LOOP
    CAT_LOOP -.-> CAT

    %% To Vendor Onboarding
    CAT -->|G| VON

    %% To Vendor Eval Detail
    CAT -->|H| VED
    VON -->|N| VED

    %% To Project Budget
    PRJ -->|B| PB
    CAT -->|F| PB
    VON -->|L| PB

    %% To Budget Requisition
    PRJ -->|D| BRQ
    CAT -->|I| BRQ
    VON -->|M| BRQ

    %% To Vendor Evaluation
    PRJ -->|C| VEH
    CAT -->|J| VEH
    VED -->|O| VEH

    %% To Budget Operation
    PRJ -->|E| BOP
    CAT -->|K| BOP
```

| Label | Relationship                                                                                                                                                                                                                                                                                                              |
| ----- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| A     | `xr_verticalbudgethead.Parent Category` looks up `xr_verticalbudgethead.xr_verticalbudgetheadid` (`1:n` self-relationship; each child category has `0..1` parent, and one parent category can have many children).                                                                                                        |
| B     | `xr_projectbudget.Project` looks up `xr_project.Project ID` (`1:n`).                                                                                                                                                                                                                                                      |
| C     | `xr_vendorevaluation.Project` looks up `xr_project.Project ID` (`1:n`).                                                                                                                                                                                                                                                   |
| D     | `xr_budgetrequisition.Project` looks up `xr_project.Project ID` (`1:n`).                                                                                                                                                                                                                                                  |
| E     | `xr_budgetoperation.Project` looks up `xr_project.Project ID` (`1:n`).                                                                                                                                                                                                                                                    |
| F     | `xr_projectbudget.Vertical` and `xr_projectbudget.Budget Head` each look up `xr_verticalbudgethead.xr_verticalbudgetheadid` (`1:n` per lookup).                                                                                                                                                                           |
| G     | `xr_vendoronboarding.Vertical`, `xr_vendoronboarding.Budget Head`, and `xr_vendoronboarding.Sub Budget Head` each look up `xr_verticalbudgethead.xr_verticalbudgetheadid` (`1:n` per lookup; `Sub Budget Head` is optional).                                                                                              |
| H     | `xr_vendorevaldetail.Vertical` and `xr_vendorevaldetail.Budget Head` each look up `xr_verticalbudgethead.xr_verticalbudgetheadid` (`1:n` per lookup).                                                                                                                                                                     |
| I     | `xr_budgetrequisition.Vertical`, `xr_budgetrequisition.Budget Head`, and `xr_budgetrequisition.Sub Budget Head` each look up `xr_verticalbudgethead.xr_verticalbudgetheadid` (`1:n` per lookup).                                                                                                                          |
| J     | `xr_vendorevaluation.Vertical`, `xr_vendorevaluation.Budget Head`, and `xr_vendorevaluation.Sub Budget Head` each look up `xr_verticalbudgethead.xr_verticalbudgetheadid` (`1:n` per lookup; `Sub Budget Head` is used if available).                                                                                     |
| K     | `xr_budgetoperation.From Vertical`, `From Budget Head`, `From Sub Budget Head`, `To Vertical`, `To Budget Head`, and `To Sub Budget Head` each look up `xr_verticalbudgethead.xr_verticalbudgetheadid` (`1:n` per lookup; `To*` fields apply to Budget Shifting, and sub-budget-head lookups are optional where defined). |
| L     | `xr_projectbudget.Evaluated vendor` looks up `xr_vendoronboarding.xr_vendoronboardingid` (`1:n`).                                                                                                                                                                                                                         |
| M     | `xr_budgetrequisition.Vendor Name` looks up `xr_vendoronboarding.xr_vendoronboardingid` (`1:n`).                                                                                                                                                                                                                          |
| N     | `xr_vendorevaldetail.Vendor` looks up `xr_vendoronboarding.xr_vendoronboardingid` (`1:n`).                                                                                                                                                                                                                                |
| O     | `xr_vendorevaluation.Vendor 1 Details`, `xr_vendorevaluation.Vendor 2 Details`, and `xr_vendorevaluation.Vendor 3 Details` each look up `xr_vendorevaldetail.xr_vendorevaldetailid` (`1:n` per lookup; logically one evaluation can hold up to 3 detail records).                                                         |
