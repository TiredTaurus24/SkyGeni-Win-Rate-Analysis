
# SkyGeni Sales Intelligence: Velocity & Risk Analysis


##  Part 1: Problem Framing
**The Business Problem:**
The CRO sees a "Win Rate" problem, but the data proves it is a **Pipeline Velocity** problem. The sales team is hoarding opportunities that should have been disqualified weeks ago. By keeping "zombie deals" in the pipeline, we inflate the number of open opportunities, which mathematically suppresses the win rate and creates a false sense of security regarding future revenue.

**Key Questions Answered:**
1.  **Is the win rate drop global?** No. It is highly concentrated in the Europe region and specific Reps.
2.  **Are we losing to competitors?** No. We are losing to *indecision*. Cycle times are bloating by +12 days on average.
3.  **Is this a product issue or a people issue?** People. The variance between Top Reps (66% WR) and Bottom Reps (15% WR) in the *same region* confirms the product is sellable; the execution is inconsistent.

---

##  Part 2: Data Exploration & Insights

### Insight 1: Micro-Segment Failure (The Europe Bleed)
While the global win rate is stable (~45%), specific segments are hemorrhaging value.
*   **Finding:** The **Europe - Pro** segment has a Win Rate of **35.5%**.
*   **Impact:** This specific segment is dragging down the global average.
*   **Action:** Immediate review of sales enablement materials for the European market.

### Insight 2: Forensic Rep Analysis (Man vs. Market)
I investigated whether the Europe failure was structural (Market) or behavioral (People).
*   **Finding:** In the same region, Top Reps (e.g., Rep 21) close at **66%**, while Bottom Reps (e.g., Rep 3) close at **15%**.
*   **Verdict:** Since some reps are succeeding wildly, the market is viable. This is a **training and performance management issue**, not a product-market fit issue.

### Custom Metric: "Cycle Time Bloat"
I defined "Bloat" as `Current Cycle Duration - Historic Winning Benchmark`.
*   **Result:** Large deals are currently bloating by **+11.3 days**. This is the primary friction point in the funnel.

---

##  Part 3: Decision Engine (The Stall Detector)

I chose **Option D: Pipeline Anomaly Detection**, specifically focusing on "Stalled Deals."

**The Logic:**
Based on historical data, the average "Won" deal closes in **63 days**.
*   **Rule:** If `Sales Cycle > 75 Days` (Benchmark + 20% Buffer) AND `Stage != Won/Lost`, the deal is flagged as **"Rotting."**
*   **Prioritization:** Deals are ranked by `Risk Score = Deal Amount * Days Overdue`.

**Actionable Output:**
The system flagged **5 High-Priority Deals** currently rotting in the pipeline (120 days old).

| Deal ID | Rep | Region | Stage | Amount | Days Open | Action |
| :--- | :--- | :--- | :--- | :--- | :--- | :--- |
| **D00880** | Rep 11 | N. America | Proposal | $73,522 | 120 | **KILL/CLOSE** |
| **D04608** | Rep 4 | APAC | Demo | $18,456 | 120 | **DISQUALIFY** |
| **D02316** | Rep 22 | Europe | Closed | $6,510 | 120 | **UPDATE CRM** |
| ... | ... | ... | ... | ... | ... | ... |

---

##  Part 4: Mini System Design
**Product Name:** SkyGeni Velocity Guard

**Architecture:**
1.  **Ingestion:** Nightly ETL job pulls `deals` and `activities` objects from CRM (Salesforce/HubSpot).
2.  **Processing (The Brain):**
    *   Calculates rolling `Avg_Winning_Cycle` by Deal Type (Small/Medium/Large).
    *   Applies the "Stall Detector" logic to open pipeline.
3.  **Delivery (The Nudge):**
    *   **Slack Bot:** "Deal D00880 is 45 days overdue. Update status or it will be auto-closed in 48 hours."
    *   **Manager Dashboard:** A "Stalled Revenue" widget showing the total value of rotting deals per rep.

---

##  Part 5: Reflection & Critical Analysis

If we were to deploy this system into production tomorrow, here is where it would struggle and how I would improve it.

### 1. Weakest Assumptions
*   **The "One-Size-Fits-All" Benchmark:** My current logic assumes a **63-day** benchmark applies to all deals. In reality, an Enterprise deal naturally takes 120 days, while an SMB deal takes 14.
    *   *Risk:* The current model will generate **False Positives** for Enterprise deals (flagging them as "Stalled" when they are healthy) and **False Negatives** for SMB deals (ignoring them until they are already dead).
*   **Linearity of Time:** The model assumes that "Days Open" is the only proxy for risk. It ignores "Momentum." A deal that is 80 days old but had 3 meetings yesterday is healthy. A deal that is 40 days old with zero emails is dead. Time alone is a blunt instrument.

### 2. Production Failure Modes (Real-World Breakage)
*   **The "Shadow Pipeline" Effect:** If we aggressively flag stalled deals, Sales Reps will stop creating Opportunities in the CRM until the very last minute to avoid being "nagged" by the algorithm. This would destroy our visibility into early-stage pipeline.
*   **Seasonality Drifts:** The model does not account for Q4 urgency. In December, cycle times compress; in August (Europe), they expand. A static rule will throw errors during these seasonal shifts.

### 3. Future Roadmap (The "1-Month" Plan)
If I had one month to upgrade this system, I would move from **Descriptive Analytics** to **Predictive Risk Modeling**:

1.  **Implement Survival Analysis (Cox Proportional Hazards):**
    *   Instead of a static "Average Cycle," I would model the *probability of closing* over time. This allows us to say: "At Day 40, this deal has a 60% chance of survival. At Day 70, it drops to 12%." This is mathematically superior to simple averages.
2.  **Activity-Based Momentum Scoring:**
    *   Integrate email/calendar metadata. A deal score should boost if the prospect replies to an email, and decay if 14 days pass with no outbound activity.
3.  **Behavioral Nudges:**
    *   Instead of just flagging "Bad Deals," the system should recommend actions. e.g., *"Similar deals that won in this stage usually have a VP-level meeting by now. You don't. Suggest scheduling one."*

### 4. Least Confident Area
*   **Attribution of "Lost" Causes:** My current analysis groups all "Lost" deals together. I am least confident in *why* they were lost. Without structured reason codes (e.g., "Price," "Feature Gap," "Ghosted"), it is hard to tell if the "Stall" was caused by the Rep being lazy or the Product being too expensive.

---

