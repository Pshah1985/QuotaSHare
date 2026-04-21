# QuotaShare – IAT Aviation Quota Share Project

This repository contains artifacts developed from the IAT Aviation Quota Share working sessions (Sessions 1–3, April 14–17, 2026).

## Files

| File | Description |
|---|---|
| `pricing.html` | Interactive Pricing screen replica with Quota Share UI section |
| `Business_Requirements.md` | Full Business Requirements Document (BRD) for Quota Share in Unity |

## Background

IAT Insurance Group writes Aviation Quota Share (QS) business where IAT acts as either the **Lead Carrier** (issues full policy) or a **Following/Participating Carrier** (issues two-page Following Form only). This project defines the requirements to automate the QS workflow within the Unity policy administration system.

## Key Concepts

- **Ground-up QS only** – IAT takes a vertical percentage from dollar one (no excess QS)
- **Lead scenario** – Full policy issued; QS endorsement (AVA/AVP/AVG/AV) attached; only IAT's % billed
- **Following scenario** – Two-page Following Form + billing summary only; no dec page, no endorsements
- **Premium override** – Required because rating engine may not produce exact dollar match
- **Separate policies** – Required when different QS % apply to different coverage parts on the same account

## Sessions

| Session | Date | Topics Covered |
|---|---|---|
| Session 1 | April 14, 2026 | Ground-up QS definition, Lead carrier scenario, endorsement forms, billing, claims |
| Session 2 | April 15, 2026 | Following carrier scenario, Following Form, premium override, midterm, taxes/fees |
| Session 3 | April 17, 2026 | Separate policy decision, premium override mechanics, large fleet spot-check, Kentucky tax, NACHA, document generation |