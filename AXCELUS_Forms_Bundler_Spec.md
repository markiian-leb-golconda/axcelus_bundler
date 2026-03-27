# Axcelus PPVA Forms Bundler — Complete Specification

> **Source Documents:**
> - `Axcelus_Investor_Questionnaire_-_Decision_Tree.docx` → **Tag: `DECISION_TREE`** — Investor questionnaire selection logic based on PPM type, owner type, and entity structure
> - `AXCELUS_Annuity_Form_Matrix_09_26_2025.xlsx` → **Tag: `FORM_MATRIX`** — Master state-by-state form matrix mapping every form ID to every US jurisdiction

---

## 1. DOMAIN CONTEXT

Axcelus is a carrier/administrator for **Private Placement Variable Annuity (PPVA)** and **Private Placement Variable Life (PPVL)** insurance products. These are unregistered securities sold under SEC exemptions. Before a policy can be issued, the applicant must complete a **bundle of forms** — which forms are included depends on several parameters.

The two source documents together describe the **complete bundling logic**: the Decision Tree governs *investor qualification questionnaires* (the complex part), while the Form Matrix governs *operational/administrative forms* per state (the state-specific part).

---

## 2. DOCUMENT ANALYSIS

### 2.1 DECISION_TREE — Investor Questionnaire Selection

This document defines which **investor qualification questionnaires** to include. It operates under two SEC exemption regimes:

| PPM Type | SEC Exemption | Investor Requirements |
|---|---|---|
| **3(c)(7)** | Investment Company Act §3(c)(7) | Must be both **Accredited Investor (AI)** AND **Qualified Purchaser (QP)** |
| **3(c)(1)** | Investment Company Act §3(c)(1) | Must be **Accredited Investor (AI)** only |

The tree then branches on three questions:

1. **Is the Policy Owner an Individual or Entity?**
2. **Does the Entity rely on a Supporting Entity for AI/QP status?**
3. **Does the Entity or Supporting Entity rely on a Supporting Individual for AI/QP status?**

### 2.2 FORM_MATRIX — State-Specific Operational Forms

The spreadsheet is a 52-row (50 states + DC + PR) × 30-column matrix. Columns 2–17 are the same investor questionnaires from the Decision Tree (no state variation — just included for completeness). Columns 18–31 are **operational forms** that DO vary by state.

Key observations:
- **3 states are NOT approved for business**: New Hampshire (NH), New York (NY), Puerto Rico (PR) — all cells empty
- **Nevada (NV) and North Dakota (ND)** have no replacement form
- **California** is the only state with the **CA Senior Policy Free Look** form (for applicants 60+)
- **13 states** have state-specific Application form variants (suffix on APVA-197)
- **20 states** have state-specific Replacement form variants (instead of generic GNLRepl-0111)
- **Kansas** has a special fork: `KSReplDiff-0111` OR `KSReplSame-0111` depending on whether the replacing insurer is different or same

---

## 3. COMPLETE PARAMETER LIST

These are ALL parameters that affect form bundling. They can be used as fields in a form/UI.

### 3.1 Core Parameters

| # | Parameter | ID | Type | Options | Source |
|---|---|---|---|---|---|
| 1 | **State** | `state` | Select (dropdown) | All 50 states + DC (excluding NH, NY, PR) | FORM_MATRIX |
| 2 | **PPM Type** | `ppm_type` | Select | `3c7` = AI + QP required, `3c1` = AI only required | DECISION_TREE |
| 3 | **Owner Type** | `owner_type` | Select | `individual`, `entity` | DECISION_TREE |
| 4 | **Has Supporting Entity?** | `has_supporting_entity` | Boolean | Yes / No | DECISION_TREE (only if `owner_type=entity`) |
| 5 | **Has Supporting Individual?** | `has_supporting_individual` | Boolean | Yes / No | DECISION_TREE (only if `owner_type=entity`) |
| 6 | **Transaction Type** | `transaction_type` | Select | `new_business`, `replacement` | FORM_MATRIX |
| 7 | **Applicant Age 60+?** | `is_applicant_60_plus` | Boolean | Yes / No | FORM_MATRIX (only for CA) |
| 8 | **KS: Same or Different Insurer?** | `ks_insurer_type` | Select | `same`, `different` | FORM_MATRIX (only for KS replacements) |

### 3.2 Parameter Dependencies

```
state
├── IF state == NH, NY, PR → STOP: "Not approved for business"
├── IF state == CA AND is_applicant_60_plus → adds CA Senior Free Look
├── IF state == KS AND transaction_type == replacement → ask ks_insurer_type
├── IF state == NV or ND → no replacement form available
│
ppm_type
├── 3c7 → uses AI-QP questionnaires
├── 3c1 → uses AI-only questionnaires
│
owner_type
├── individual → individual questionnaires only (no supporting entity/individual questions)
├── entity → entity questionnaires + ask about supporting entity/individual
│   ├── has_supporting_entity → adds Supporting Questionnaire (Entity)
│   └── has_supporting_individual → adds Supporting Questionnaire (Individual)
│
transaction_type
├── new_business → includes all standard forms
├── replacement → includes replacement-specific forms (1035 Exchange, Lost Policy Letter, Replacement Form)
```

---

## 4. COMPLETE FORM CATALOG

### 4.1 Investor Qualification Questionnaires (16 forms)

These are NOT state-specific. They are selected purely by PPM type, owner type, and entity structure.

#### Group A: Individual Policy Owner (3 forms per PPM type)

| Form | Tag | When Included |
|---|---|---|
| Axcelus **AI-QP** Questionnaire - Individual Policy Owner - 02.2024 | `IND_AIQP` | `ppm_type=3c7` AND `owner_type=individual` |
| Axcelus **AI** Questionnaire - Individual Policy Owner - 02.2024 | `IND_AI` | `ppm_type=3c1` AND `owner_type=individual` |
| Axcelus Restricted Person Questionnaire - Individual Policy Owner - 02.2024 | `IND_RP` | `owner_type=individual` (always) |
| Axcelus Rule 5131 Questionnaire - Individual Policy Owner - 02.2024 | `IND_5131` | `owner_type=individual` (always) |

#### Group B: Entity Policy Owner (3 forms per PPM type)

| Form | Tag | When Included |
|---|---|---|
| Axcelus **AI-QP** Questionnaire - Entity Policy Owner - 02.2024 | `ENT_AIQP` | `ppm_type=3c7` AND `owner_type=entity` |
| Axcelus **AI** Questionnaire - Entity Policy Owner - 02.2024 | `ENT_AI` | `ppm_type=3c1` AND `owner_type=entity` |
| Axcelus Restricted Person Questionnaire - Entity Policy Owner - 02.2024 | `ENT_RP` | `owner_type=entity` (always) |
| Axcelus Rule 5131 Questionnaire - Entity Policy Owner - 02.2024 | `ENT_5131` | `owner_type=entity` (always) |

#### Group C: Supporting Entity Questionnaires (3 forms per PPM type)

| Form | Tag | When Included |
|---|---|---|
| Axcelus **AI-QP** Supporting Questionnaire (Delivered by Entity) - 02.2024 | `SUP_ENT_AIQP` | `owner_type=entity` AND `has_supporting_entity=true` AND `ppm_type=3c7` |
| Axcelus **AI** Supporting Questionnaire (Delivered by Entity) - 02.2024 | `SUP_ENT_AI` | `owner_type=entity` AND `has_supporting_entity=true` AND `ppm_type=3c1` |
| Axcelus Restricted Person Supporting Questionnaire (Delivered by Entity) - 02.2024 | `SUP_ENT_RP` | `owner_type=entity` AND `has_supporting_entity=true` (always) |
| Axcelus Rule 5131 Supporting Questionnaire (Delivered by Entity) - 02.2024 | `SUP_ENT_5131` | `owner_type=entity` AND `has_supporting_entity=true` (always) |

#### Group D: Supporting Individual Questionnaires (3 forms per PPM type)

| Form | Tag | When Included |
|---|---|---|
| Axcelus **AI-QP** Supporting Questionnaire (Delivered by Individual) - 02.2024 | `SUP_IND_AIQP` | `owner_type=entity` AND `has_supporting_individual=true` AND `ppm_type=3c7` |
| Axcelus **AI** Supporting Questionnaire (Delivered by Individual) - 02.2024 | `SUP_IND_AI` | `owner_type=entity` AND `has_supporting_individual=true` AND `ppm_type=3c1` |
| Axcelus Restricted Person Supporting Questionnaire (Delivered by Individual) - 02.2024 | `SUP_IND_RP` | `owner_type=entity` AND `has_supporting_individual=true` (always) |
| Axcelus Rule 5131 Supporting Questionnaire (Delivered by Individual) - 02.2024 | `SUP_IND_5131` | `owner_type=entity` AND `has_supporting_individual=true` (always) |

### 4.2 Operational / Administrative Forms (14 form types)

These vary by state and transaction type.

| # | Form | Tag | Transaction | Condition | State Variation |
|---|---|---|---|---|---|
| 1 | Private Placement Memorandum Receipt | `PPM_RECEIPT` | New Business | Always | No (same for all states) |
| 2 | Application for Variable Annuity Contract | `APPLICATION` | New Business | Always | **Yes** — state-specific form IDs (see §5) |
| 3 | W-9 | `W9` | New Business | Always | No |
| 4 | Acknowledgement Form | `ACK` | New Business | Always | No |
| 5 | Document Delivery Instructions | `DOC_DELIVERY` | New Business | Always | No |
| 6 | Interested Party Request | `INTERESTED_PARTY` | New Business | Always | No |
| 7 | Notice of Insurance Information Practices | `NOTICE_INS` | New Business | Always | No |
| 8 | CA Senior Policy Free Look | `CA_FREE_LOOK` | New Business | **Only CA** AND applicant 60+ | California only |
| 9 | 1035 Absolute Exchange Form | `1035_EXCHANGE` | **Replacement only** | If replacement | No |
| 10 | 1035 Lost Policy Letter | `1035_LOST` | **Replacement only** | If replacement | No |
| 11 | Replacement Form | `REPLACEMENT` | **Replacement only** | If replacement | **Yes** — state-specific form IDs (see §5) |
| 12 | Privacy Policy | `PRIVACY` | New Business | Always | No |
| 13 | Privacy Policy - CA Residents | `PRIVACY_CA` | New Business | Always (included for all states) | No |
| 14 | Wire Instructions | `WIRE` | New Business | Always | No |

---

## 5. STATE-SPECIFIC FORM MAPPINGS

### 5.1 Application Form Variants by State

| State | Form ID | Notes |
|---|---|---|
| **Default** (most states) | `APVA-197` | Standard application |
| AZ | `APVA-197AZ` | Arizona-specific |
| AR, ME, OH | `APVA-197F` | Shared "F" variant |
| CA | `APVA-197CA` | California-specific |
| DC | `APVA-197DC` | DC-specific |
| FL | `APVA-197FL` | Florida-specific |
| HI | `APVA-2012` | Hawaii — completely different form number |
| MA | `APVA-197MA` | Massachusetts-specific |
| MD | `APVA-197MD` | Maryland-specific |
| NC | `APVA-197NC` | North Carolina-specific |
| NJ | `APVA-197NJ` | New Jersey-specific |
| OK | `APVA-197OK` | Oklahoma-specific |
| TX | `APVA-197TX` | Texas-specific |

### 5.2 Replacement Form Variants by State

| State | Form ID(s) | Notes |
|---|---|---|
| **Default** (most states) | `GNLRepl-0111` | General replacement form |
| AR | `ArkCompForm-0111` | Arkansas comparison form |
| CA | `CARepl-0111` | California replacement |
| DE | `DERepl-0111` | Delaware replacement |
| FL | `FLReplA-0111` **AND** `FLDFS-H1-1981` | Florida requires **TWO** replacement forms |
| GA | `GARepl-1116` | Georgia replacement |
| ID | `IDRepl-0111` | Idaho replacement |
| IL | `ILReplA-1112` **AND** `ILReplB-1112` | Illinois requires **TWO** replacement forms |
| IN | `INRepl-0111` | Indiana replacement |
| KS | `KSReplDiff-0111` **OR** `KSReplSame-0111` | Kansas — depends on same/different insurer |
| MA | `MARepl-0111` | Massachusetts replacement |
| MI | `MIRepl-0111` | Michigan replacement |
| MN | `MNRepl-0111` | Minnesota replacement |
| MO | `MORepl-0111` | Missouri replacement |
| OK | `OKRepl-0111` | Oklahoma replacement |
| PA | `PARepl-0111` | Pennsylvania replacement |
| SD | `SDRepl1035Ex-1116` | South Dakota replacement |
| TN | `TNRepl-1116` | Tennessee replacement |
| WA | `WARepl-1116` | Washington replacement |
| WY | `WYRepl-0313` | Wyoming replacement |
| NV, ND | **NONE** | No replacement form exists |
| NH, NY, PR | **N/A** | Not approved for business |

---

## 6. BUNDLING ALGORITHM (PSEUDOCODE)

```
function getBundledForms(params):
    forms = []

    // ── STEP 0: Validate state ──
    if params.state in [NH, NY, PR]:
        return ERROR("State not approved for business")

    // ── STEP 1: Investor Qualification Questionnaires ──
    if params.owner_type == "individual":
        if params.ppm_type == "3c7":
            forms.push(IND_AIQP)
        else:
            forms.push(IND_AI)
        forms.push(IND_RP)
        forms.push(IND_5131)

    else if params.owner_type == "entity":
        // Base entity forms
        if params.ppm_type == "3c7":
            forms.push(ENT_AIQP)
        else:
            forms.push(ENT_AI)
        forms.push(ENT_RP)
        forms.push(ENT_5131)

        // Supporting Entity forms
        if params.has_supporting_entity:
            if params.ppm_type == "3c7":
                forms.push(SUP_ENT_AIQP)
            else:
                forms.push(SUP_ENT_AI)
            forms.push(SUP_ENT_RP)
            forms.push(SUP_ENT_5131)

        // Supporting Individual forms
        if params.has_supporting_individual:
            if params.ppm_type == "3c7":
                forms.push(SUP_IND_AIQP)
            else:
                forms.push(SUP_IND_AI)
            forms.push(SUP_IND_RP)
            forms.push(SUP_IND_5131)

    // ── STEP 2: Universal Operational Forms ──
    forms.push(PPM_RECEIPT)
    forms.push(APPLICATION[params.state])   // state-specific form ID
    forms.push(W9)
    forms.push(ACK)
    forms.push(DOC_DELIVERY)
    forms.push(INTERESTED_PARTY)
    forms.push(NOTICE_INS)
    forms.push(PRIVACY)
    forms.push(PRIVACY_CA)
    forms.push(WIRE)

    // ── STEP 3: CA-specific ──
    if params.state == "CA" AND params.is_applicant_60_plus:
        forms.push(CA_FREE_LOOK)

    // ── STEP 4: Replacement-specific ──
    if params.transaction_type == "replacement":
        forms.push(1035_EXCHANGE)
        forms.push(1035_LOST)

        replacement = REPLACEMENT_MAP[params.state]
        if replacement is not None:
            if params.state == "KS":
                if params.ks_insurer_type == "same":
                    forms.push(KSReplSame-0111)
                else:
                    forms.push(KSReplDiff-0111)
            else:
                // FL and IL have multiple forms (split on "&")
                for each form in replacement.split("&"):
                    forms.push(form.trim())
        else:
            // NV, ND — no replacement form available
            forms.push(NOTE: "No state replacement form available")

    return forms
```

---

## 7. EDGE CASES & WARNINGS

1. **Florida & Illinois**: Replacement requires TWO separate forms each (joined with "&" in matrix)
2. **Kansas**: Replacement form depends on whether the replacing insurer is the same or different
3. **Nevada & North Dakota**: Have operational forms for new business, but NO replacement form — replacement transactions may not be supported
4. **NH, NY, PR**: Completely excluded — no forms at all, not approved for business
5. **CA Senior Free Look**: Only triggers for California AND only when applicant is 60+ years old
6. **Hawaii Application**: Uses a completely different form number (`APVA-2012` vs `APVA-197*`)
7. **Privacy Policy - CA Residents**: Included for ALL states (not just CA) — it's a universal disclosure
8. **Investor questionnaires (cols 2–17)**: Have NO state variation — same forms regardless of state
9. **Application form (col 19)**: The only "New Business" form that varies by state — the rest are universal
10. **Supporting Entity/Individual questions**: Only relevant for Entity owners — should be hidden for Individual owners

---

## 8. WEB APPLICATION SPEC (for Claude Code)

### 8.1 Overview

Build a **static single-page application** (React, deployed to GitHub Pages) with 3 views:
1. **Login** — hardcoded password gate
2. **Bundler** — parameter form → generates form list
3. **Logic Reference** — developer-readable explanation of all rules

### 8.2 Tech Stack

- React (single `.jsx` file using CDN or bundled)
- Tailwind CSS (CDN)
- No backend — all logic is client-side
- Deployed as static site to GitHub Pages
- Use `HashRouter` for GitHub Pages compatibility (no server-side routing)

### 8.3 Page 1: Login

**Route:** `#/login` (default)

**Behavior:**
- Show a simple centered login form with password input
- Hardcoded password: `axcelus2025`
- On correct password → redirect to `#/bundler`
- On incorrect → show inline error "Incorrect password"
- Store auth state in `sessionStorage` (cleared on tab close)
- All other routes redirect to login if not authenticated

**UI:** Minimal — Axcelus branding, single input, single button.

### 8.4 Page 2: Bundler (main page)

**Route:** `#/bundler`

**Layout:** Two-panel — left side has the parameter form, right side (or below on mobile) shows the generated form list.

**Parameter Form Fields:**

```
1. State (dropdown)
   - 50 states + DC
   - Exclude NH, NY, PR (or show them greyed out with "Not approved" tooltip)
   - Default: none selected

2. PPM Type (radio buttons)
   - "3(c)(7) — AI + QP Required"
   - "3(c)(1) — AI Only"

3. Owner Type (radio buttons)
   - "Individual"
   - "Entity"

4. [CONDITIONAL — only if owner_type=entity]
   Has Supporting Entity? (toggle/checkbox)

5. [CONDITIONAL — only if owner_type=entity]
   Has Supporting Individual? (toggle/checkbox)

6. Transaction Type (radio buttons)
   - "New Business"
   - "Replacement"

7. [CONDITIONAL — only if state=CA]
   Applicant Age 60+? (toggle/checkbox)

8. [CONDITIONAL — only if state=KS AND transaction_type=replacement]
   Replacing Insurer? (radio buttons)
   - "Different insurer"
   - "Same insurer"
```

**Generate Button:** "Generate Form Bundle"

**Results Table:** Simple table with columns:
| # | Form Tag | Form Name | Form ID | Category | Condition |
|---|---|---|---|---|---|
| 1 | IND_AIQP | Axcelus AI-QP Questionnaire... | — | Investor Qualification | PPM 3(c)(7) + Individual |
| 2 | PPM_RECEIPT | Private Placement Memorandum Receipt | PPM Receipt | Operational | Always |
| ... | ... | ... | ... | ... | ... |

**Categories for grouping:**
- `Investor Qualification` — the questionnaire forms
- `Operational — Universal` — PPM Receipt, W-9, Ack, etc.
- `Operational — State-Specific` — Application form, Replacement form
- `Operational — Conditional` — CA Free Look, 1035 forms

### 8.5 Page 3: Logic Reference

**Route:** `#/logic`

**Content:** A clean, readable page that explains ALL logic rules. Structured as:

1. **Parameter definitions** — what each parameter means, what values it accepts
2. **Decision tree diagram** (text-based or simple visual) — showing the branching logic for investor questionnaires
3. **State-specific form tables** — application variants and replacement variants
4. **Edge cases** — the warnings from §7 above
5. **Complete form catalog** — all 30 forms with their tags and conditions

This page is a **reference for developers** — it should be concise, use code-style formatting for form tags and conditions, and be scannable.

### 8.6 Data Model (embed in app)

All data should be hardcoded as JavaScript objects/arrays in the app:

```javascript
// State -> Application Form ID
const APPLICATION_FORMS = {
  AL: "APVA-197", AK: "APVA-197", AZ: "APVA-197AZ",
  AR: "APVA-197F", CA: "APVA-197CA", CO: "APVA-197",
  CT: "APVA-197", DE: "APVA-197", DC: "APVA-197DC",
  FL: "APVA-197FL", GA: "APVA-197", HI: "APVA-2012",
  ID: "APVA-197", IL: "APVA-197", IN: "APVA-197",
  IA: "APVA-197", KS: "APVA-197", KY: "APVA-197",
  LA: "APVA-197", ME: "APVA-197F", MD: "APVA-197MD",
  MA: "APVA-197MA", MI: "APVA-197", MN: "APVA-197",
  MS: "APVA-197", MO: "APVA-197", MT: "APVA-197",
  NE: "APVA-197", NV: "APVA-197", NJ: "APVA-197NJ",
  NM: "APVA-197", NC: "APVA-197NC", ND: "APVA-197",
  OH: "APVA-197F", OK: "APVA-197OK", OR: "APVA-197",
  PA: "APVA-197", RI: "APVA-197", SC: "APVA-197",
  SD: "APVA-197", TN: "APVA-197", TX: "APVA-197TX",
  UT: "APVA-197", VT: "APVA-197", VA: "APVA-197",
  WA: "APVA-197", WV: "APVA-197", WI: "APVA-197",
  WY: "APVA-197"
};

// State -> Replacement Form ID(s)
const REPLACEMENT_FORMS = {
  AL: ["GNLRepl-0111"], AK: ["GNLRepl-0111"], AZ: ["GNLRepl-0111"],
  AR: ["ArkCompForm-0111"], CA: ["CARepl-0111"], CO: ["GNLRepl-0111"],
  CT: ["GNLRepl-0111"], DE: ["DERepl-0111"], DC: ["GNLRepl-0111"],
  FL: ["FLReplA-0111", "FLDFS-H1-1981"],
  GA: ["GARepl-1116"], HI: ["GNLRepl-0111"],
  ID: ["IDRepl-0111"],
  IL: ["ILReplA-1112", "ILReplB-1112"],
  IN: ["INRepl-0111"], IA: ["GNLRepl-0111"],
  KS: null, // special handling — KSReplDiff-0111 or KSReplSame-0111
  KY: ["GNLRepl-0111"], LA: ["GNLRepl-0111"],
  ME: ["GNLRepl-0111"], MD: ["GNLRepl-0111"],
  MA: ["MARepl-0111"], MI: ["MIRepl-0111"],
  MN: ["MNRepl-0111"], MS: ["GNLRepl-0111"],
  MO: ["MORepl-0111"], MT: ["GNLRepl-0111"],
  NE: ["GNLRepl-0111"], NV: null, // no replacement form
  NJ: ["GNLRepl-0111"], NM: ["GNLRepl-0111"],
  NC: ["GNLRepl-0111"], ND: null, // no replacement form
  OH: ["GNLRepl-0111"], OK: ["OKRepl-0111"],
  OR: ["GNLRepl-0111"], PA: ["PARepl-0111"],
  RI: ["GNLRepl-0111"], SC: ["GNLRepl-0111"],
  SD: ["SDRepl1035Ex-1116"], TN: ["TNRepl-1116"],
  TX: ["GNLRepl-0111"], UT: ["GNLRepl-0111"],
  VT: ["GNLRepl-0111"], VA: ["GNLRepl-0111"],
  WA: ["WARepl-1116"], WV: ["GNLRepl-0111"],
  WI: ["GNLRepl-0111"], WY: ["WYRepl-0313"]
};

// States not approved
const BLOCKED_STATES = ["NH", "NY", "PR"];
```

### 8.7 GitHub Pages Deployment

The repo should have:
```
/
├── index.html          ← single-file React app (CDN imports)
├── README.md           ← this spec (or a short version)
└── .nojekyll           ← prevents Jekyll processing
```

Enable GitHub Pages from repo Settings → Pages → Source: main branch, root folder.

### 8.8 Non-Functional Requirements

- Mobile-responsive (the parameter form stacks vertically on small screens)
- No external API calls — everything is client-side
- Print-friendly results table (add a print stylesheet or print button)
- Clean, professional UI suitable for insurance/financial industry (no playful aesthetics)
- Conditional fields should animate in/out smoothly (not just appear/disappear)

---

## 9. FORM SEQUENCE ORDER

Per the Form Matrix, operational forms should appear in this order in the bundle:

1. Private Placement Memorandum Receipt
2. Application for Variable Annuity Contract
3. W-9
4. Acknowledgement Form
5. Document Delivery Instructions
6. Interested Party Request
7. Notice of Insurance Information Practices
8. CA Senior Policy Free Look *(if applicable)*
9. 1035 Absolute Exchange Form *(if replacement)*
10. 1035 Lost Policy Letter *(if replacement)*
11. Replacement Form *(if replacement)*
12. Privacy Policy
13. Privacy Policy - CA Residents
14. Wire Instructions

Investor Qualification Questionnaires should appear BEFORE operational forms in the bundle (they are the first thing the investor fills out).

---

## 10. SIGNATURE REQUIREMENTS (for reference)

| Form | Signer |
|---|---|
| Individual Questionnaires | Proposed Policy Owner |
| Entity Questionnaires | Authorized Signer(s) per formation docs |
| Supporting Entity Questionnaires | Authorized Signer(s) for Supporting Entity |
| Supporting Individual Questionnaires | Supporting Individual |
| PPM Receipt | Policy Owner |
| Application | Proposed Annuitant, Contingent Annuitant, Applicant/Owner, Agent |
| W-9 | Policy Owner |
| Acknowledgement | Policy Owner & Witness |
| Document Delivery | Policy Owner |
| Interested Party Request | Policy Owner |
| Notice of Insurance | n/a (informational) |
| CA Senior Free Look | Applicant |
| 1035 Exchange | Policy Owner and Witness |
| 1035 Lost Policy Letter | Policy Owner and Witness |
| Replacement Form | n/a (varies by state) |
| Privacy Policy | n/a (informational) |
| Privacy Policy - CA | n/a (informational) |
| Wire Instructions | n/a (informational) |
