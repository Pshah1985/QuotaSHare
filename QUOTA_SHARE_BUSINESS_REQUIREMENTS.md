# Quota Share – Business Requirements Artifact
**Product:** Unity Aviation Policy System
**Feature Area:** Pricing Screen – Quota Share
**Source:** QS Sessions 1, 2, 3 (04/14/26 – 04/17/26)
**Status:** Draft v1.0 – 04/21/26
**Prepared by:** Copilot / Pavan Shah

---

## 1. Overview

IAT Insurance writes aviation policies (Aircraft Hull & Liability, Platinum, Aviation General Liability) on both a **ground-up (100%)** and a **quota share** basis. A quota share policy means IAT takes a defined vertical percentage of the risk rather than the full exposure. IAT may act as either the **Lead Carrier** or a **Following (Participating) Carrier**.

This document captures the functional requirements for the Quota Share feature on the Unity **Pricing Screen**, derived from three business working sessions.

---

## 2. Scope

| In Scope | Out of Scope |
|---|---|
| Lead carrier quota share endorsement | Excess quota share (layered) |
| Following carrier following form | NACHA generation (separate deliverable) |
| Premium override at quote and bind | Commission system redesign |
| Hull / Liability premium split | Downstream reinsurance cession |
| Kentucky tax logic (aircraft location) | CAT / actuarial modeling |
| State-specific fee toggles | Inter-carrier billing reconciliation |

---

## 3. Business Rules

### BR-1: Quota Share Type
- MUST be ground-up quota share only. Excess (layered) quota share is NOT supported.
- IAT takes a consistent vertical percentage of all coverages within a coverage form.

### BR-2: Coverage Form Level
- Quota share participation percentage is set at the **coverage form level**, not the individual aircraft/item level.
- Different coverage forms on the same policy MAY have different QS percentages.
- If all forms share the same percentage, a single AV form may be used; otherwise separate per-form endorsements (AVA, AVP, AVG) are required.

### BR-3: Multiple Policies for Different Percentages
- **CONFIRMED DECISION (04/17/26):** If coverage forms have different QS percentages, **separate policy records** are required.
- The system MUST support issuing multiple policies from the same submission, sharing a submission number but with distinct policy numbers.
- The underwriter may duplicate a quote and modify each copy independently without re-keying.

### BR-4: Percentage Rules
- IAT Quota Share % must be between 0.01% and 99.99%.
- The system auto-computes: Other Carrier(s) % = 100 - IAT %.
- IAT does NOT track or validate the identities or percentages of other participating carriers.
- QS percentage is FIXED for the policy term. Mid-term percentage changes require cancellation and rewrite.

### BR-5: Lead Carrier Requirements
- Full policy issued at 100% rated value (all exposures, limits, aircraft schedules).
- A Quota Share endorsement is attached to indicate IAT's participation percentage.
- A **Lead Fee** (flat, fully earned, non-commissionable) may be charged 100% to IAT.
- Policy output: full deck page, coverage forms, state forms, endorsements, quote letter.
- NACHA is generated separately outside the standard policy packet.

### BR-6: Following Carrier Requirements
- IAT does NOT issue a full policy. Only a **two-page Following Form** is issued.
- The Following Form references the lead carrier's policy and states IAT's percentage and premium.
- NO endorsements, deck page, coverage forms, or state forms are attached.
- Documents produced: Following Form + billing summary. Rating worksheet is optional for internal use.
- IAT does NOT collect or remit the other carriers' premiums.

### BR-7: Premium Rating – Lead
- Full 100% rating must occur using the standard rating algorithm.
- The rated premium is reduced by the QS percentage for **billing only**.
- The policy document shows 100% limits and values; billing reflects IAT's share only.

### BR-8: Premium Rating – Following
- Underwriters validate the lead's premium via:
  1. **Full schedule entry** – all aircraft rated to compare at 100%.
  2. **Composite spot check** – 3-5 representative aircraft to confirm rates are in the ballpark.
- For large fleets (threshold TBD; see OI-1), composite spot check is used. Full entry is not required.
- Aircraft entered for spot check are NOT printed on the Following Form.

### BR-9: Premium Override
- An exact premium override input MUST be available on the Pricing Screen for quota share policies.
- The override must preserve the exact dollar amount entered (no rounding/recalculation drift).
- The override applies at initial quote, midterm endorsements, and renewals.
- When active, the system must clearly indicate the override is in effect.

### BR-10: Hull / Liability Premium Split
- Booked premium must be allocated between Hull (Physical Damage) and Liability for billing routing and loss ratio calculations.
- Hull + Liability must equal the booked IAT premium. System warns if they do not reconcile.
- GL coverages and medical payments roll into the liability bucket.

### BR-11: Midterm Changes
- All midterm endorsements bill at IAT's QS percentage of the endorsement premium.
- Billing system receives only IAT's share.
- For large-fleet following policies, aircraft additions/removals are managed via billing summary update, not schedule entry.
- A **Total Number of Aircraft** field must be maintained and updated when aircraft are added or removed.

### BR-12: Aircraft Schedule – Following
- Small fleets: enter aircraft in the system for rating validation and claims reference.
- Large fleets: enter only the spot-check aircraft. Full schedule entry is NOT required.
- Aircraft schedule is NOT printed on the Following Form.
- A minimum aircraft identifier is needed for claims association. For unfiled schedules, "Schedule on File with Lead" is acceptable.
- A field for **Total Number of Aircraft on Risk** is required for downstream reporting.

### BR-13: Active IAT Aircraft Check
- The existing N-number duplicate check should return a **WARNING, not a hard stop**, for quota share following policies.

### BR-14: Documents Produced – Lead

| Document | Produced? |
|---|---|
| Deck Page | Yes (100% limits) |
| Coverage Forms | Yes |
| Quota Share Endorsement(s) | Yes (per coverage form) |
| State Forms | Yes |
| Quote Letter | Yes (includes QS section) |
| Binder | Yes |
| Rating Worksheet | Yes |
| NACHA | Separate document, stored separately |

### BR-15: Documents Produced – Following

| Document | Produced? |
|---|---|
| Following Form (2 pages) | Yes |
| Billing Summary | Yes |
| Rating Worksheet | Optional (internal use only) |
| Deck Page | No |
| Coverage Forms | No |
| Endorsements | No |
| State Forms | No |
| NACHA | Not generated by IAT when following |

### BR-16: Commission
- Lead role: commission per standard rules (can vary by broker/account).
- Following role: TBD – Rob Lombardo to validate (see OI-2).
- Lead Fee: no commission paid on lead fees.

### BR-17: Policy Structure Downstream
- A Policy Structure designation must be sent downstream to indicate: IAT is Lead or Following, and IAT's QS percentage by coverage form.
- Downstream systems use this to correctly calculate IAT's net exposure against 100% limits.

### BR-18: Kentucky Tax – Aircraft Location
- Kentucky tax is based on **location of the aircraft (tangible property)**, NOT the insured's mailing address.
- Override/checkbox must be available to suppress Kentucky tax on a per-aircraft basis.
- Fields needed: State tax, City tax, County tax (independently toggleable).
- Educational institutions are exempt (other exemption categories TBD – see OI-5).

### BR-19: State Surcharges / Fees
- Active fees: Kentucky (tangible property tax), PLIGA (NJ), West Virginia.
- Florida FIGA: sunsets September 30 – deactivate but retain infrastructure for reactivation.
- System must support staged fee schedules with effective/expiration dates.
- For quota share: taxes and fees apply to IAT's percentage of the applicable premium.

### BR-20: Endorsement Trigger / Placement
- QS feature is triggered by selection of the Following Form (following) or attaching the Quota Share Endorsement (lead).
- Placement within Unity (Pricing screen vs. Endorsements screen) is TBD (see OI-3).
- Once triggered, the Pricing screen must expose the premium override capability.

---

## 4. User Stories

| ID | As a… | I want to… | So that… |
|---|---|---|---|
| QS-US-01 | Underwriter | Designate a policy as Quota Share (Lead) and set my % per coverage form | The system bills only IAT's share while showing 100% limits to the insured |
| QS-US-02 | Underwriter | Designate a policy as a Following Form and enter lead policy details | The correct two-page form is produced and no full policy documents are generated |
| QS-US-03 | Underwriter | Override the final premium to an exact dollar amount | I can match or vary from the lead's pro-rata without rounding artifacts |
| QS-US-04 | Underwriter | Enter aircraft for spot-check rating without full fleet entry | Large fleets can be validated without duplicating the lead's schedule |
| QS-US-05 | Underwriter | See the computed IAT premium share and remaining carrier percentage | I can validate exposure allocation at a glance |
| QS-US-06 | Underwriter | Allocate booked premium between hull and liability | Billing routes the correct premium to each exposure bucket |
| QS-US-07 | Underwriter | Set a Lead Fee (flat, fully earned) on a Lead QS policy | IAT is compensated for lead carrier administration |
| QS-US-08 | AM/Underwriter | Enter total number of aircraft on risk for a large-fleet following policy | Downstream reporting has an aircraft count without requiring full schedule entry |
| QS-US-09 | System | Suppress Kentucky tax when the aircraft is not based in Kentucky | Tax calculation reflects tangible property location, not insured address |
| QS-US-10 | AM | Apply midterm endorsement premium at IAT's QS % automatically | Midterm billing always reflects the correct IAT share |

---

## 5. UI Requirements – Pricing Screen

### QS-UI-01: Quota Share Panel
- An interactive **collapsible Quota Share panel** appears on the Pricing Screen **above the Licensed Producer field**.
- Panel is collapsed by default if no QS data has been entered; expanded if data exists.

### QS-UI-02: Lead / Following Toggle
- Binary toggle: "No (Following)" | "Yes (Lead)"
- Toggle drives conditional field display.

### QS-UI-03: IAT QS % Input
- Numeric input, 2 decimal places, range 0.01–99.99.
- Inline validation with error message.
- On change: auto-compute Other Carrier(s) % = 100 – IAT %.
- Sidebar panel updates to show IAT dollar share of base premium.

### QS-UI-04: Lead Carrier Fields (when Lead = Yes)
- Lead Fee ($) – currency input
- Helper text: "Flat, fully earned – no commission"

### QS-UI-05: Following Carrier Fields (when Lead = No)
- Lead Carrier Name (required)
- Lead Policy Number (required)
- Lead Underwriting Company

### QS-UI-06: Premium Override
- "Override Premium" button reveals override input.
- Override input accepts exact dollar amount.
- When active, a visible indicator shows override is in effect.
- "Clear Override" resets to algorithm-computed amount.

### QS-UI-07: Hull / Liability Premium Allocation
- Two inputs: Hull Premium – IAT Share ($), Liability Premium – IAT Share ($).
- Inline validation: sum must equal booked premium.
- Green check shown when balanced; amber warning when not.

### QS-UI-08: Sidebar Summary Panel
- Shows: IAT Role (Lead / Following badge), IAT %, Other Carrier %, IAT dollar share.
- Appears only when QS data is present.

### QS-UI-09: Clear Quota Share
- "Clear Quota Share" link resets all QS fields and collapses the panel.
- Confirmation prompt required.

---

## 6. Open Items / Decisions Required

| # | Topic | Owner | Status |
|---|---|---|---|
| OI-1 | Exact threshold for "large fleet" (spot check vs full entry) | Rob Lombardo | Open |
| OI-2 | Commission rate on following QS policies | Rob Lombardo | Open |
| OI-3 | Placement of QS UI trigger (Pricing vs Endorsements screen) | Bikash/Coomer/Cheryl | Open |
| OI-4 | NACHA system generation – scope and form layout | Madison / Brian M | Open |
| OI-5 | Kentucky tax exemption categories beyond educational institutions | Connor Taylor / Legal | Open |
| OI-6 | Rating worksheet suppression for following form | Cheryl / Brian M | Open |
| OI-7 | Two NACHA documents when aircraft and GL have different percentages | Rob Lombardo | Open |
| OI-8 | Downstream reporting field: total aircraft count on following | Christine (BIT) | Open |

---

## 7. Glossary

| Term | Definition |
|---|---|
| Ground-up Quota Share | IAT takes a percentage of risk from the first dollar (not excess) |
| Lead Carrier | IAT issues the full policy and manages the account |
| Following Carrier | IAT participates behind the lead; issues only the two-page Following Form |
| NACHA | North American Claims Handling Agreement – signed by all participating carriers |
| Following Form | Two-page IAT document issued when acting as a following carrier |
| Premium Override | Underwriter-entered exact premium that bypasses the rating algorithm for billing |
| Hull | Physical damage coverage on aircraft |
| Composite Rating | Spot-check rating of a few representative aircraft to validate the lead's premium |
| FIGA | Florida Insurance Guaranty Association surcharge |
| PLIGA | NJ-based insurance guaranty fund fee |
| Delta | A premium difference between IAT's view of correct premium and the lead's premium
