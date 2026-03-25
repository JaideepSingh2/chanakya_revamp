```mermaid
%%{init: {"theme":"base","themeVariables":{"primaryTextColor":"#0f172a","lineColor":"#64748b","fontFamily":"Segoe UI","background":"#ffffff"},"flowchart":{"curve":"basis","nodeSpacing":110,"rankSpacing":130,"htmlLabels":true}}}%%
flowchart TD

    classDef master fill:#f8fafc,stroke:#334155,stroke-width:1.6px,color:#0f172a;
    classDef entity fill:#ffffff,stroke:#94a3b8,stroke-width:1.4px,color:#0f172a;
    classDef ghost fill:transparent,stroke:transparent,color:transparent;

    %% Master Entities
    PRJ["<b>xr_project</b><br/>--------------------<br/>Project ID"]:::master
    CAT["<b>xr_verticalbudgethead</b><br/>--------------------<br/>xr_verticalbudgetheadid<br/>Parent Category"]:::master

    %% Invisible circular anchor to pull the loop into a round curve
    CAT_LOOP(( )):::ghost

    %% Vendor Entities
    VON["<b>xr_vendoronboarding</b><br/>--------------------<br/>xr_vendoronboardingid<br/>Vertical<br/>Budget Head<br/>Sub Budget Head"]:::entity
    VED["<b>xr_vendorevaldetail</b><br/>--------------------<br/>xr_vendorevaldetailid<br/>Vendor<br/>Vertical<br/>Budget Head"]:::entity

    %% Transactional / Operation Entities
    PB["<b>xr_projectbudget</b><br/>--------------------<br/>xr_projectbudgetid<br/>Project<br/>Vertical<br/>Budget Head<br/>Evaluated vendor"]:::entity
    BRQ["<b>xr_budgetrequisition</b><br/>--------------------<br/>xr_budgetrequisitionid<br/>Project<br/>Vertical<br/>Budget Head<br/>Sub Budget Head<br/>Vendor Name"]:::entity
    VEH["<b>xr_vendorevaluation</b><br/>--------------------<br/>xr_vendorevaluationid<br/>Project<br/>Vertical<br/>Budget Head<br/>Sub Budget Head<br/>Vendor 1 Details<br/>Vendor 2 Details<br/>Vendor 3 Details<br/>Evaluated vendor"]:::entity
    BOP["<b>xr_budgetoperation</b><br/>--------------------<br/>xr_budgetoperationid<br/>Project<br/>From Vertical<br/>From Budget Head<br/>From Sub Budget Head<br/>To Vertical<br/>To Budget Head<br/>To Sub Budget Head"]:::entity

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
    VON -->|P| VEH

    %% To Budget Operation
    PRJ -->|E| BOP
    CAT -->|K| BOP
```

| Label | Relationship                                                                                                                                                                                                                                                                                                              |
| ----- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| A     | <b>xr_verticalbudgethead</b>.Parent Category looks up <b>xr_verticalbudgethead</b>.xr_verticalbudgetheadid (`1:n` self-relationship; each child category has `0..1` parent, and one parent category can have many children).                                                                                           |
| B     | <b>xr_projectbudget</b>.Project looks up <b>xr_project</b>.Project ID (`1:n`).                                                                                                                                                                                                                                           |
| C     | <b>xr_vendorevaluation</b>.Project looks up <b>xr_project</b>.Project ID (`1:n`).                                                                                                                                                                                                                                        |
| D     | <b>xr_budgetrequisition</b>.Project looks up <b>xr_project</b>.Project ID (`1:n`).                                                                                                                                                                                                                                       |
| E     | <b>xr_budgetoperation</b>.Project looks up <b>xr_project</b>.Project ID (`1:n`).                                                                                                                                                                                                                                         |
| F     | <b>xr_projectbudget</b>.Vertical and <b>xr_projectbudget</b>.Budget Head each look up <b>xr_verticalbudgethead</b>.xr_verticalbudgetheadid (`1:n` per lookup).                                                                                                                                                         |
| G     | <b>xr_vendoronboarding</b>.Vertical, <b>xr_vendoronboarding</b>.Budget Head, and <b>xr_vendoronboarding</b>.Sub Budget Head each look up <b>xr_verticalbudgethead</b>.xr_verticalbudgetheadid (`1:n` per lookup; `Sub Budget Head` is optional).                                                                     |
| H     | <b>xr_vendorevaldetail</b>.Vertical and <b>xr_vendorevaldetail</b>.Budget Head each look up <b>xr_verticalbudgethead</b>.xr_verticalbudgetheadid (`1:n` per lookup).                                                                                                                                                   |
| I     | <b>xr_budgetrequisition</b>.Vertical, <b>xr_budgetrequisition</b>.Budget Head, and <b>xr_budgetrequisition</b>.Sub Budget Head each look up <b>xr_verticalbudgethead</b>.xr_verticalbudgetheadid (`1:n` per lookup).                                                                                                  |
| J     | <b>xr_vendorevaluation</b>.Vertical, <b>xr_vendorevaluation</b>.Budget Head, and <b>xr_vendorevaluation</b>.Sub Budget Head each look up <b>xr_verticalbudgethead</b>.xr_verticalbudgetheadid (`1:n` per lookup; `Sub Budget Head` is used if available).                                                            |
| K     | <b>xr_budgetoperation</b>.From Vertical, From Budget Head, From Sub Budget Head, To Vertical, To Budget Head, and To Sub Budget Head each look up <b>xr_verticalbudgethead</b>.xr_verticalbudgetheadid (`1:n` per lookup; `To*` fields apply to Budget Shifting, and sub-budget-head lookups are optional where defined). |
| L     | <b>xr_projectbudget</b>.Evaluated vendor looks up <b>xr_vendoronboarding</b>.xr_vendoronboardingid (`1:n`).                                                                                                                                                                                                            |
| M     | <b>xr_budgetrequisition</b>.Vendor Name looks up <b>xr_vendoronboarding</b>.xr_vendoronboardingid (`1:n`).                                                                                                                                                                                                             |
| N     | <b>xr_vendorevaldetail</b>.Vendor looks up <b>xr_vendoronboarding</b>.xr_vendoronboardingid (`1:n`).                                                                                                                                                                                                                   |
| O     | <b>xr_vendorevaluation</b>.Vendor 1 Details, <b>xr_vendorevaluation</b>.Vendor 2 Details, and <b>xr_vendorevaluation</b>.Vendor 3 Details each look up <b>xr_vendorevaldetail</b>.xr_vendorevaldetailid (`1:n` per lookup; logically one evaluation can hold up to 3 detail records).                                |
| P     | <b>xr_vendorevaluation</b>.Evaluated vendor looks up <b>xr_vendoronboarding</b>.xr_vendoronboardingid (`1:n`).                                                                                                                                                                                                         |
