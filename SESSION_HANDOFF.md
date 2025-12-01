# PGE Rate Plan Analysis - Session Handoff Document

## PROMPT TO GIVE CLAUDE CODE NEXT SESSION:

```
I'm continuing work on my PGE utility bill analysis project. Please read the SESSION_HANDOFF.md file in the project root to understand the current state, then help me build the extraction and analysis pipeline to compare electricity costs across different PGE rate plans.
```

---

## PROJECT OVERVIEW

**Goal:** Compare actual electricity usage against different PGE residential rate plans to determine which would be cheapest.

**Approach:** ETL pipeline (Extract → Transform → Load/Analyze)

**Current Phase:** EXTRACTION (just starting)

**Tech Stack:** Python (mid-senior developer skill level), zero budget (free tools only)

---

## WHAT WE HAVE (Data Inventory)

### 1. Usage Data (VERIFIED GOOD)
- **Location:** `raw_data/electric/pge_electric_usage_interval_data_Service 1_1_2025-01-01_to_2025-11-29.csv`
- **Format:** PGE standard interval export
- **Granularity:** 15-minute intervals
- **Date Range:** Jan 1 - Nov 29, 2025
- **Structure:**
  - Metadata header (lines 1-5): Customer name, address, account number, service
  - Data columns (line 7+): TYPE, DATE, START TIME, END TIME, USAGE (kWh), COST, NOTES
  - Clean, well-formatted, ready to parse

### 2. Gas Data (Not prioritized yet)
- **Location:** `raw_data/gas/pge_natural_gas_usage_interval_data_Service 2_2_2025-01-01_to_2025-11-29.csv`
- **Status:** Exists but not examined yet

### 3. Rate Schedule Data (PARTIALLY VERIFIED)
- **Location:** `raw_data/res_rate_schedule.csv`
- **Format:** PGE residential rate comparison table
- **Contains:**
  - **E-1 (Tiered):** Tier 1 $0.40122/kWh, Tier 2 $0.50257/kWh, Daily min $0.39167/meter/day
  - **E-TOU-B (4-9pm peak):** Summer Peak $0.57985, Off-Peak $0.45679; Winter Peak $0.44321, Off-Peak $0.40441
  - **E-TOU-C (4-9pm every day):** Summer Peak $0.60729, Off-Peak $0.50429; Winter Peak $0.49312, Off-Peak $0.46312; Baseline credit -$0.10135/kWh
  - **E-TOU-D (5-8pm weekdays):** Summer Peak $0.56462, Off-Peak $0.42966; Winter Peak $0.47502, Off-Peak $0.43641
  - **Seasons:** Summer = June-September, Winter = October-May
- **Verification Status:** Structure looks correct, values in expected range, BUT not cross-checked against official PGE tariff PDFs (PDF fetch failed)

---

## KEY INFORMATION

### Current Rate Plan
- **User is on:** E-1 (Tiered pricing)

### Rate Plans to Compare
All residential PGE plans, specifically:
- E-1 (current plan - baseline)
- E-TOU-B
- E-TOU-C
- E-TOU-D
- E-ELEC (if eligible)
- EV-2-A (if eligible)

### Customer Details (from usage CSV)
- Name: Deniz Twelves
- Address: 1408 31ST AVE APT 301, SAN FRANCISCO CA 941223162
- Account: 2056577232
- Service: Service 1 (electric)

---

## CRITICAL OPEN QUESTIONS

### ⚠️ MUST RESOLVE BEFORE FULL ANALYSIS:

1. **Baseline Allowance (CRITICAL for E-1 calculations)**
   - User's tiered plan calculations depend on baseline kWh/month
   - Baseline varies by climate zone in SF
   - Likely 8-11 kWh/day for SF apartment
   - **Action:** Check user's PGE bill or ask them

2. **Rate Schedule Verification**
   - CSV rates not verified against official tariffs
   - **Recommendation:** Calculate one month (e.g., November) and compare to actual bill
   - If totals match within a few dollars, proceed with confidence

---

## PIPELINE ARCHITECTURE (Recommended)

### Phase 1: EXTRACTION
**Inputs:**
- `raw_data/electric/pge_electric_usage_interval_data_*.csv`
- `raw_data/res_rate_schedule.csv`
- User's baseline allowance (TBD)

**Outputs:**
- Clean usage DataFrame: datetime, kWh, hour, day_of_week, month, date
- Structured rate data by plan (easy to query)

**Key Tasks:**
- Parse PGE interval CSV (skip 6-line header, parse dates/times)
- Extract metadata
- Create datetime column combining date + start time
- Add derived columns: hour, day_of_week, month
- Parse rate schedule CSV into queryable structure

### Phase 2: TRANSFORM
**For each rate plan:**
- **E-1:** Apply tiered logic based on monthly cumulative usage vs baseline
- **E-TOU-B/C/D:** Match each interval to peak/off-peak + season, apply rates
- **E-TOU-C:** Apply baseline credit
- **E-TOU-D:** Filter peak to weekdays only

**Calculations:**
- Cost per 15-min interval
- Daily totals
- Monthly totals
- Annual projection

### Phase 3: LOAD/ANALYSIS
**Outputs:**
- Cost comparison table (by month, by plan)
- Savings analysis vs current E-1
- Usage pattern insights (peak vs off-peak usage)
- Recommendation: best plan for user's usage patterns

---

## ARCHITECTURE PREFERENCE (from user)
User prefers: **Build full pipeline** so analysis can run in one go

**Suggested approach:**
- Python scripts/modules for extract + transform (reusable)
- Jupyter notebook for analysis + visualization (interactive exploration)
- Mix allows both automation and flexibility

---

## WHAT USER WANTS TO SEE (End Results)
- Which rate plan is cheapest overall?
- Monthly cost breakdown by plan
- How much could be saved by switching?
- Usage pattern insights (when power is used most)

---

## NEXT SESSION - IMMEDIATE ACTION ITEMS

1. **Ask user for baseline allowance** (check PGE bill or account)
2. **Decide output preferences:**
   - Just comparison table?
   - Detailed monthly breakdown?
   - Visualizations?
   - All of the above?
3. **Verify rate data:**
   - Option A: Quick validation (calc one month vs actual bill)
   - Option B: Proceed with caveat
4. **Build extraction module:**
   - Parse PGE interval CSV
   - Parse rate schedule CSV
   - Output clean data structures
5. **Build transform module:**
   - Cost calculation functions for each rate plan
   - Handle tiered vs TOU logic
6. **Create analysis notebook/script:**
   - Compare all plans
   - Generate recommendations

---

## TECHNICAL NOTES

### PGE Interval CSV Structure
- Skip first 6 rows (5 metadata + 1 blank)
- Header row 7: TYPE,DATE,START TIME,END TIME,USAGE (kWh),COST,NOTES
- Data starts row 8
- 15-minute intervals (96 per day)
- Times in HH:MM format
- Usage in kWh (decimal)
- Cost reflects current plan pricing

### Rate Schedule CSV Structure
- Complex header spanning multiple rows
- Key data starting around row 24 (E-1), row 40+ (TOU plans)
- Contains tier thresholds, TOU periods, seasonal rates
- Includes CARE/FERA discount info (may not apply)

---

## WEB RESOURCES (for reference)

- [PGE Electric Rates](https://www.pge.com/tariffs/en/rate-information/electric-rates.html)
- [E-TOU-C Tariff PDF](https://www.pge.com/tariffs/assets/pdf/tariffbook/ELEC_SCHEDS_E-TOU-C.pdf)
- [E-1 Tariff PDF](https://www.pge.com/tariffs/assets/pdf/tariffbook/ELEC_SCHEDS_E-1.pdf)
- [Residential Rate Plan Pricing](https://www.pge.com/assets/pge/docs/account/rate-plans/residential-electric-rate-plan-pricing.pdf)

Note: PDF fetches failed in last session, but links are valid for manual review.

---

## PROJECT STATUS

**Completed:**
- ✅ Identified data sources
- ✅ Confirmed current rate plan (E-1)
- ✅ Understood CSV data structure
- ✅ Found rate schedule data
- ✅ Defined pipeline architecture

**Blocked/Pending:**
- ⚠️ Baseline allowance (needed for E-1 calculations)
- ⚠️ Rate data verification
- ⏳ Code implementation (not started - user stopped me from writing code prematurely)

**Not Started:**
- Extract module
- Transform module
- Analysis notebook
- Visualization

---

## SESSION CONTEXT NOTES

**User's Working Style:**
- Wants advice/mentoring, not immediate code
- Mid-senior Python developer (can implement with guidance)
- Appreciates being asked before writing code
- Values understanding over quick solutions

**Mistakes Made This Session:**
- I jumped into writing code without being asked
- User correctly stopped me and asked for advice first
- Adjusted to provide mentoring/architecture guidance instead

**Good Interactions:**
- User has clean data ready
- Clear project goals
- Practical constraints (zero budget, Python)
- Willing to answer questions

---

## RESUMING THE SESSION

When Claude Code reads this file next time:
1. Acknowledge you've read the handoff document
2. Ask the user for baseline allowance (critical missing data)
3. Confirm output preferences
4. Ask if they want to verify rate data first OR proceed with caveat
5. Get permission before writing any code
6. Follow their lead on implementation vs. guidance
