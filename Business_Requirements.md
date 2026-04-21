# Quota Share – Business Requirements Document

## Document Control

| Attribute | Detail |
|---|---|
| **Project** | IAT Aviation – Quota Share in Unity |
| **Version** | 1.0 DRAFT |
| **Status** | Draft – Pending Business Review |
| **Prepared By** | Derived from QS Working Sessions 1–3 |
| **Session Dates** | April 14, 15, 17, 2026 |
| **Stakeholders** | Aviation Underwriting (R. Lombardo), Product (C. Taylor, K. Hewes), IT (B. Marburger, B. Agarwal, B. Pati, C. Burnham), Operations (M. Jahn, M. Kalberg), Data (C. Smith) |
| **Last Updated** | 2026-04-21 |

---

## 1. Executive Summary

IAT Insurance Group writes Aviation business under a Quota Share (QS) arrangement, whereby IAT participates in a risk alongside one or more other carriers. IAT may act as the **Lead Carrier** (issuing the full policy and managing the account) or as a **Following (Participating) Carrier** (issuing only a two-page Following Form and billing its percentage of premium).

The current workflow is entirely manual (Frontier system + Excel billing summaries). The objective of this document is to define all business rules and functional requirements needed to automate the Quota Share workflow within the **Unity** policy administration system.

---

## 2. Background and Context

- IAT Aviation currently handles Quota Share policies manually outside of Unity.
- The business issues individual policies per coverage type (Aircraft, Platinum, GL). The move to a package policy in Unity requires explicit design decisions around how QS percentages are captured and applied per coverage part.
- Three working sessions (April 14–17, 2026) were held to gather and validate requirements from underwriting, operations, product, IT, and actuarial stakeholders.
- A Blueprint-phase design was completed approximately 4 months prior; this document reflects updates and clarifications from the working sessions.

---

## 3. Scope

### In Scope
- Lead Carrier scenario: new business, midterm, renewal
- Following Carrier scenario: new business, midterm, renewal
- Quota Share UI on Unity Pricing screen
- Policy structure screen updates (downstream data feed)
- Document generation rules (Following Form, NACHA, Rating Worksheet suppression)
- Billing integration (premium split by coverage bucket)
- Tax and fee handling for QS policies
- Downstream actuarial / data feed

### Out of Scope
- Excess Quota Share (IAT does ground-up QS only)
- Reinsurance treaty management
- Other carriers' billing or premium tracking
- NACHA completion by broker (broker responsibility)

---

## 4. Business Rules

### 4.1 Ground-Up Quota Share Definition

- IAT writes **ground-up quota share only**: IAT takes a vertical percentage of the full risk from the ground up.
- Excess quota share (where IAT takes a layer above a retention) is **not** supported.
- The QS percentage represents IAT's share of both **limits** and **premium**.
- The full 100% limits appear on the declaration page (Lead scenario). IAT's endorsement identifies IAT's specific percentage.

### 4.2 Lead Carrier Scenario

- IAT issues the **full policy** (100% limits, full coverage forms, all endorsements) as the lead.
- A **Quota Share endorsement** is attached to the policy identifying IAT's participation percentage and the remaining percentage required from other markets.
- The endorsement must state that coverage is **warranted** subject to maintaining other insurance in full force.
- IAT **bills only its percentage** of the total rated premium.
- IAT may charge a **Lead Fee**: a flat, fully-earned dollar amount for the administrative work of leading the account. The lead fee:
  - Is collected 100% by IAT (not shared with following markets)
  - Has no commission applied to it
  - Is not returned upon midterm cancellation (fully earned at inception)
  - Is entered as a discrete dollar field, not a percentage
- IAT does **not** track or validate other carriers' premiums, percentages, or financial status.
- The agent/broker is responsible for ensuring 100% of the risk is placed.

#### Endorsement Forms (Lead)
- **AVA** – Aircraft coverage only
- **AVP** – Platinum coverage only
- **AVG** – General Liability coverage only
- **AV** – All coverage parts at the same percentage (single endorsement applicable across the package policy)
- State-specific variants exist for each (e.g., AVA-WV, AVP-WV, AVG-WV)

### 4.3 Following (Participating) Carrier Scenario

- IAT issues only a **two-page Following Form** (no declaration page, no coverage forms, no endorsements, no state forms).
- The Following Form states IAT's participation percentage, premium (our share), payment terms, "as agreed" for limits/coverage terms, and NACHA language.
- IAT also issues a **billing summary** for each transaction (inception, midterm, renewal).
- IAT does **not** issue endorsements for midterm changes when following; only a revised billing summary is issued.
- IAT's underwriters validate the lead carrier's premium by either:
  - **Full schedule rating**: entering all aircraft for smaller fleets (typically ≤ ~20 aircraft)
  - **Composite / spot-check rating**: entering 3–4 representative aircraft for large fleets (typically > 20 aircraft); underwriter then overrides to agreed premium
- IAT may accept the lead's premium, charge its own (delta), or decline to follow.

### 4.4 Policy Structure Rules

| Rule | Detail |
|---|---|
| **Single QS % across all coverage parts** | Use a single AV-level QS endorsement; all parts on one policy |
| **Different QS % by coverage part** | Must be separate policies; same submission number; underwriter duplicates quote and adjusts each copy |
| **Aircraft hull and liability** | Must carry the **same** QS percentage; cannot split hull from aircraft liability on a single policy |
| **GL sub-coverages** | All GL sub-coverages share the same QS % on the GL coverage part |
| **Mixed: one part QS, another not** | Allowed on the same package policy using coverage-part-level endorsements |
| **Duplicate quote workflow** | Unity duplicate-quote capability supports issuing both quotes as separate policies under the same submission number |
| **QS % lock** | The QS percentage cannot be changed midterm. Any change requires a full cancel/rewrite |
| **Lead/Following lock** | IAT's role is also locked at inception. Changing roles requires a cancel/rewrite |

### 4.5 Endorsement Rules

- When IAT is the **lead**, all endorsements are attached normally and billed at IAT's QS percentage.
- When IAT is the **following**, no endorsements are issued. The Following Form's "as agreed" language covers all lead terms.
- **Liability buy-ups** (temporary limit increases):
  - When Lead: issue endorsement with buy-up language at IAT's QS share
  - When Following: issue billing summary only for IAT's share of the flat buy-up premium
- **Effective date–limited endorsements** must use the Changes Endorsement form to capture the date window.
- QS percentage changes via endorsement are **not permitted**; cancel/rewrite required.

### 4.6 Premium Calculation and Override

#### Lead Scenario
1. Rating engine calculates 100% ground-up premium with pricing mods applied as normal.
2. A Quota Share endorsement is selected identifying IAT's %.
3. Billing is generated for IAT's % of the total rated premium.
4. The **Override Premium** field allows the underwriter to enter an exact dollar amount when the rating engine produces a rounded figure that does not match the desired amount (e.g., $49,992 instead of $50,000).

#### Following Scenario
1. Rating engine calculates an indicative 100% premium (full or spot-check schedule).
2. Underwriter compares IAT's indication to the lead's premium.
3. If agreed to lead's premium: enter override = lead's premium x IAT's %; system books that exact amount.
4. If pursuing a delta: enter IAT's desired premium directly via override.
5. Override must be allocated across hull and liability buckets for downstream reporting.

#### Hull / Liability Split
- Every booked premium must be allocated between Hull and Liability (and GL sub-coverages where applicable).
- Hull + Liability must equal the total booked premium.
- System shall display an amber warning if values do not reconcile.

### 4.7 Midterm Changes

| Change Type | Lead Behavior | Following Behavior |
|---|---|---|
| Add aircraft (same type) | Rate normally; endorsement at IAT's QS % | Book lead's pro-rata premium at IAT's %; billing summary |
| Add aircraft (new/unique type) | Rate and validate; endorse | Spot-check; agree or pursue delta; override premium; billing summary only |
| Delete aircraft | Deletion endorsement at QS % | Remove from system; return premium; billing summary |
| Liability buy-up (temporary) | Endorsement with time window | Book flat override premium at IAT's %; billing summary; no endorsement issued |
| Change QS % | **Not permitted midterm – cancel/rewrite required** | |
| Switch Lead/Following | **Not permitted midterm – cancel/rewrite required** | |

- IAT as following carrier is not obligated to follow every midterm change and may decline.
- A **Total Aircraft on Risk** field must be maintained and updated at each midterm for downstream data.

### 4.8 Renewal

- QS % and IAT role may be renegotiated at renewal (new policy = new inception).
- Renewal workflow is the same as new business.

### 4.9 Billing

- IAT bills **only its QS percentage** of the premium.
- Lead Fee (lead scenario only) is billed at 100%; not shared; no commission.
- Payment terms for following policies are dictated by the lead carrier's terms; Unity must support flexible/custom billing schedules.
- Premium is allocated at the **coverage bucket level** (Hull, Liability, GL sub-coverages).
- The override field ensures exact dollar billing to match the lead's expected amounts (rounding differences of even $0.50 have caused reconciliation issues in the current manual process).

### 4.10 Taxes and Fees

#### Kentucky Tangible Property Tax
- Triggered by **aircraft location** (hangar/base airport), not insured's mailing address.
- Components: state tax, city/municipal tax, county tax.
- Educational institutions and certain religious organizations may be exempt.
- Unity must allow **suppression of Kentucky tax** at the aircraft level with separate state/city/county fields.
- FlightTech has automated this calculation – evaluate reuse.

#### Other State Fees

| Fee | Trigger | Notes |
|---|---|---|
| FIGA (Florida) | Insured address / issuing state | Tied to hurricane season assessments; expires 2026-09-30 but will recur |
| PLIGA (New Jersey) | Insured address / issuing state | Relatively stable |
| West Virginia fee | Issuing state | Relatively stable |
| SIGA (California) | Issuing state | Has come and gone historically |

- **Requirement**: Each fee must have an on/off switch with effective date range, configurable without code deployment.

### 4.11 NACHA / North American Claims Handling Agreement

- **IAT as Lead**: System pre-populates the NACHA with policy number, period, IAT QS %, lead fee, and pre-populated signature block. Generated as a **separate document** (not part of the policy packet). Broker circulates to following markets for signature.
- **IAT as Following**: IAT provides its information to the broker for consolidation.
- NACHA can only be completed after policy issuance (policy number required).
- If coverage parts carry different QS percentages (separate policies), separate NACHAs may be required per policy (open item OI-009).

### 4.12 Data and Downstream Reporting

- Policy structure flags must be extended to Aviation:
  - `policy_structure_type`: GROUND_UP_QS | EXCESS | FULL_RISK
  - `iat_role`: LEAD | FOLLOWING
  - `iat_qs_percentage`: numeric (0–100)
  - `other_carrier_qs_percentage`: 100 minus IAT %
- **Total Aircraft on Risk** field must be captured and passed downstream for following policies with spot-check rating.
- Hull and liability premium splits must be transmitted in the SDM feed.
- Rating Worksheet must **not** be included in the policy output packet for following policies.
- Medical expense and non-owned aircraft premium are allocated to the **liability** bucket.

---

## 5. Functional Requirements

### 5.1 Pricing Screen – Quota Share UI

| ID | Requirement |
|---|---|
| FR-QS-001 | Pricing screen shall include a collapsible Quota Share section immediately above the Licensed Producer field |
| FR-QS-002 | Section shall include "Is IAT Lead Carrier?" toggle (Yes/No); default = No |
| FR-QS-003 | Numeric field (0–100, two decimal places) for IAT's QS percentage; required when section is active |
| FR-QS-004 | Read-only computed field displaying 100 minus IAT QS % |
| FR-QS-005 | When Lead = Yes: display Lead Fee ($) currency input (flat, fully earned, no commission) |
| FR-QS-006 | When Lead = No: display Lead Carrier Name, Lead Policy Number, Lead Underwriting Company text inputs |
| FR-QS-007 | "Override Premium" button reveals currency input with helper text: "Enter the exact premium to book. The system-rated premium will not be used for billing." |
| FR-QS-008 | Hull Premium and Liability Premium (IAT share) currency inputs; amber warning if they do not sum to booked premium |
| FR-QS-009 | Section header badge displays "IAT [n]%" once percentage is entered |
| FR-QS-010 | "Clear Quota Share" resets all fields and collapses the section |
| FR-QS-011 | Different QS % by coverage part is supported via duplicate-quote workflow (separate policies, same submission) |

### 5.2 Policy Structure Screen

| ID | Requirement |
|---|---|
| FR-PS-001 | Capture policy_structure_type, iat_role, iat_qs_percentage; feed to SDM downstream |
| FR-PS-002 | "Total Aircraft on Risk" numeric field for following policies with spot-check rating |

### 5.3 Document Generation

| ID | Requirement |
|---|---|
| FR-DG-001 | Lead: generate full policy packet (dec page, coverage forms, endorsements including QS endorsement, state forms) |
| FR-DG-002 | Following: generate only Following Form, Billing Summary, and CA Fraud Statement (CA accounts) |
| FR-DG-003 | Lead: generate pre-populated NACHA document as separate file at policy issuance |
| FR-DG-004 | Rating Worksheet generated for internal use; excluded from agent-facing packet when following |
| FR-DG-005 | Quote document includes Quota Share section with IAT % and role |

### 5.4 Billing Integration

| ID | Requirement |
|---|---|
| FR-BI-001 | Bill at IAT's QS % only |
| FR-BI-002 | Lead Fee billed as separate line item at 100%, flat, fully earned, no commission |
| FR-BI-003 | Premium allocated to billing system at Hull / Liability / GL sub-coverage level |
| FR-BI-004 | Support custom payment schedules to accommodate lead carrier terms |
| FR-BI-005 | Override field (FR-QS-007) enables exact dollar billing |

### 5.5 Data / Actuarial Feed

| ID | Requirement |
|---|---|
| FR-DA-001 | QS policy structure flags (type, role, %) included in SDM downstream feed |
| FR-DA-002 | total_aircraft_on_risk included in feed for following policies with spot-check rating |
| FR-DA-003 | Hull, Liability, GL sub-coverage premium amounts (at IAT share) transmitted in feed |

---

## 6. Non-Functional Requirements

| ID | Requirement |
|---|---|
| NFR-001 | Quota Share section responds to user input without page reload (client-side logic) |
| NFR-002 | All QS fields preserved when a quote is saved, duplicated, or converted to a policy |
| NFR-003 | QS percentage field validates input range 0.01–99.99 with inline error if out of range |
| NFR-004 | Override premium entry requires explicit user action (button click) to prevent accidental overrides |
| NFR-005 | Kentucky tax suppression override is auditable (user ID, timestamp, reason code) |
| NFR-006 | Tax fee on/off switches configurable by system administrators without code deployment |
| NFR-007 | NACHA document stored in document management system linked to the policy record |

---

## 7. Open Items / Decisions Required

| # | Item | Owner | Priority |
|---|---|---|---|
| OI-001 | Confirm: Can different QS % by coverage part exist on a single policy, or is separate-policy always required? | R. Lombardo / C. Taylor | High |
| OI-002 | Commission treatment for following carrier: is IAT bound by lead's commission rate, or can IAT set its own? | R. Lombardo | High |
| OI-003 | Define threshold (number of aircraft) above which spot-check composite rating is acceptable vs full schedule entry | R. Lombardo | Medium |
| OI-004 | Confirm whether NACHA should be system-generated in Unity or remain manually completed | R. Lombardo / M. Jahn | Medium |
| OI-005 | Determine final placement of QS trigger in Unity: endorsement screen vs pricing screen vs policy structure screen | B. Marburger / C. Burnham / C. Smith | High |
| OI-006 | Kentucky municipal tax automation: evaluate FlightTech boundary data reuse vs manual override approach | B. Marburger / J. Adamo | Medium |
| OI-007 | Rating worksheet: confirm exact suppression logic for following policy output packet | C. Smith / B. Agarwal | Low |
| OI-008 | Time-bound endorsement workflow (e.g., liability buy-up for film production): does Unity support effective date windows on endorsements? | C. Taylor / K. Hewes | Medium |
| OI-009 | Multi-NACHA scenario: when coverage parts have separate policies, does each require its own NACHA? | R. Lombardo / C. Taylor | Medium |
| OI-010 | Following policy – midterm aircraft add: exact workflow for large fleet spot-check policies | M. Jahn / R. Lombardo | High |

---

## 8. Assumptions and Constraints

1. IAT writes ground-up quota share only. Excess quota share is out of scope.
2. The QS percentage is between 0.01% and 99.99%.
3. Unity's existing duplicate-quote capability is the solution for multiple percentages across coverage parts.
4. IAT does not track or validate other carriers' participation, premiums, or solvency.
5. The agent/broker is solely responsible for ensuring 100% of the risk is placed and the NACHA is returned.
6. The lead carrier's policy number is required before the following form can be fully issued.
7. FIGA (Florida) is expected to sunset September 30, 2026 but may be reinstated.
8. Kentucky tangible property tax rules apply to all lines of business with mobile property.
9. Medical expense and non-owned aircraft premium are allocated to the liability bucket.

---

## 9. Glossary

| Term | Definition |
|---|---|
| **Ground-Up Quota Share** | A co-insurance arrangement where each participant takes a vertical percentage of the risk from the first dollar of loss |
| **Lead Carrier** | The carrier that issues the full policy, manages the account, negotiates terms, and handles claims on behalf of all participants |
| **Following / Participating Carrier** | A carrier that takes a percentage of the risk but issues only a Following Form |
| **Following Form** | A two-page document issued by the following carrier acknowledging its participation percentage, premium share, and deferral to the lead's policy terms |
| **NACHA** | North American Claims Handling Agreement – document signed by all QS participants defining claims handling responsibilities and fees |
| **QS Percentage** | Each carrier's share of the insured risk and premium (all percentages must total 100%) |
| **Lead Fee** | An optional flat, fully-earned fee charged by the lead carrier; collected 100% by the lead; no commission |
| **Composite Rating / Spot Check** | Rating 3–4 representative aircraft to validate lead carrier's premium without entering the full fleet schedule |
| **Premium Override** | A user-entered exact dollar premium that replaces the system-calculated rated premium for booking purposes |
| **Hull Premium** | Premium allocated to the physical damage (hull) coverage component |
| **Liability Premium** | Premium allocated to the aviation liability coverage component |
| **FIGA** | Florida Insurance Guaranty Association assessment fee |
| **PLIGA** | New Jersey insolvency guaranty fund fee |
| **KY Tangible Property Tax** | Kentucky tax assessed on tangible property (aircraft) based on physical location, not insured's mailing address |
| **Delta** | A pricing difference between the lead carrier's premium and the following carrier's desired premium |
| **AVA / AVP / AVG / AV** | IAT endorsement form prefixes for Aircraft, Platinum, GL, and all-coverage quota share endorsements |
| **SDM** | System Data Management – IAT's downstream data feed to actuarial and reporting systems |