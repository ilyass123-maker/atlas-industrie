# Decision note architecture of the Atlas Industrie campus

## 1. Context and requirements
Atlas Industrie, a manufacturing company operating in Morocco, has acquired an adjacent second building, expanding its footprint across two sites:
- **Building A:** Corporate headquarters and IT facility (200 staff, IT servers including ERP and messaging, main technical room).
- **Building B:** Production plant (100 staff, new line with industrial sensors, PLCs/automates, and production supervision server).
- **Interconnection:** ~150 meters apart, with fiber-optic links feasible between technical rooms.
- **Management Directive:** "The network must never go down, above all for production," with a strict requirement to justify the cost within a capped budget. Building B must maintain autonomous manufacturing capability even if Building A is completely isolated or offline.

## 2. Technical targets retained
- **Availability Targets:** 99.9% for office IT (approx. 8.8 hours of downtime/year) and 99.99% for critical industrial production (approx. 53 minutes of downtime/year). *Note: True "five nines" (99.999%) is economically unviable and technically unreachable without automated maintenance scripts.*
- **Recovery Objectives:** 
  - **RTO (Recovery Time Objective):** 15 minutes for production, requiring automated failover and rapid configuration reload rather than lengthy manual troubleshooting.
  - **RPO (Recovery Point Objective):** 24 hours for configurations, ensured via daily automated device configuration backups.

## 3. The two architectures compared
Three structural scenarios were evaluated:
1. **S1 (Three-Tier Legacy):** 2 Cores + 4 Distribution switches + 10 Access switches + Firewalls. Provides strong scalability but introduces unnecessary hops and excessive CAPEX/OPEX for a medium campus.
2. **S2 (Distributed Collapsed Core — Recommended):** Merges Core and Distribution layers into a pair of fixed L3 switches split across Building A (CC-A) and Building B (CC-B), interconnected by dual fiber routes. Carries local OT gateways and supervision on CC-B to ensure site autonomy.
3. **S3 (Single-Site Collapsed Core):** Both core switches centralized in Building A. While cheaper upfront, it violates the business mandate because the total loss of Building A cuts off Building B's default gateways and routing, halting production.

## 4. Five-year costing
Calculations are based on a 5-year horizon, vendor support at 10% of CAPEX/year, and operations costing 1.5 days per device per year at 5,400 MAD/day (8,100 MAD/device/year). Power and cooling are excluded for clarity.

| Scenario | CAPEX (MAD) | Support (5 yrs, MAD) | Operations (5 yrs, MAD) | Total 5-Yr TCO (MAD) | Device Count |
| :--- | :--- | :--- | :--- | :--- | :--- |
| **S1 (Three-Tier)** | 1,155,600 | 577,800 | 729,000 | **2,462,400** | 18 |
| **S2 (Distributed Core)** | 766,800 | 383,400 | 567,000 | **1,717,200** | 14 |
| **S3 (Single-Site Core)** | 745,200 | 372,600 | 567,000 | **1,684,800** | 14 |

- **S2 vs S1 Savings:** S2 reduces the 5-year TCO by **745,200 MAD (~30%)**, freeing up budget for security monitoring and training.
- **S2 vs S3 Delta:** S2 costs **32,400 MAD more (~2% increase)** over 5 years compared to S3.

## 5. Decision and justification
- **Selected Architecture:** Scenario 2 (Distributed Collapsed Core).
- **Justification:** S3 is cheaper but fails to satisfy the core business requirement of surviving the loss of Building A. The **32,400 MAD** price premium of S2 over S3 is evaluated against an assumed production stoppage cost of **108,000 MAD per hour**. This means the distributed core investment pays for itself entirely within **18 minutes** of prevented production downtime. Furthermore, adopting NetDevOps automation can further reduce S2's operational costs down to a 5-year TCO of 1,528,200 MAD.

## 6. Risks and points of failure
- **Common-Cause Failures:** Theoretical parallel availability calculations assume independent failures. If redundant fiber paths share the same cable chamber or physical trench under the courtyard, a single excavation accident cuts both links simultaneously. Mitigation: Enforce physically distinct trenching ("north gallery" vs. "south facade").
- **Access Switch Bottleneck:** Even with dual-attached cores, non-redundant access switches set an availability ceiling. Critical zones mitigate this via careful port pinning and rapid hardware replacement SLAs.
- **IT-to-OT Pivot:** Risk of ransomware spreading from office workstations to industrial controllers is controlled via strict default-deny firewalls separating the `10.42.16.0/20` IT zone from the `10.42.64.0/20` OT zone.

## 7. Assumptions and limits
- **Pricing:** Fictional conversion rates (1 EUR = 10.8 MAD) and fixed unit prices; actual vendor quotations will vary.
- **Exclusions:** Building electrical grid capacity, HVAC/cooling infrastructure, civil engineering trenching costs, and physical security badges are excluded from this network-layer TCO model.
- **OT Availability Trade-off:** PLCs do not support 802.1X dynamic authentication; factory availability supersedes strict port quarantine, so static MAC filtering and change logging are accepted as compensating controls.