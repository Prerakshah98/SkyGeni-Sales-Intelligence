# SkyGeni Sales Intelligence Challenge

**Role:** Data Science / Applied AI Engineer  
**Task:** Design a decision intelligence system to diagnose win rate drops and recommend actions.

---

## Part 1: Problem Framing

### 1. What do you think is the real business problem here?
The CRO stated that "pipeline volume is healthy" but "win rates have dropped." This suggests the problem is not lead generation (marketing is working), but **lead conversion** (sales execution is failing). We are likely dealing with:
* **Low-quality leads:** Deals that look promising initially but never close.
* **Stalled deals:** Sales reps spending too much time on customers who are not ready to buy.
* **Mismatched customers:** Pitching complex enterprise products to customers who only want basic tools.

### 2. What key questions should an AI system answer for the CRO?
The AI system should answer:
* At which specific stage are we losing the most deals?
* Which industries or regions are dragging down the average performance?
* Is the sales cycle getting longer over time?
* Do currently open deals share characteristics with deals lost last month?

### 3. What metrics matter most for diagnosing win rate issues?
The most critical metrics are:
* **Stage-by-Stage Conversion Rate:** To identify exactly where deals fall out of the funnel.
* **Late-Stage Loss Ratio:** The percentage of deals lost after the proposal stage (indicating wasted effort).

### 4. What assumptions are you making about the data or business?
* **Data Accuracy:** `deal_amount` is accurate and represents the Annual Contract Value (ACV).
* **Outcome Finality:** "Lost" deals are permanently dead, not just on hold for next year.
* **Process Consistency:** Sales stages have remained consistent and haven't changed definitions over the last couple of quarters.

---

## Part 2: Exploratory Data Analysis & Insights

I performed a deep dive into the provided dataset (`skygeni_sales_data.csv`) to identify the root causes of the win rate drop.

*(See `sassessment.ipynb` for code and visualizations)*

---

## Part 3: The Decision Engine (Deal Risk Scoring)

**Chosen Option:** Option A – Deal Risk Scoring
*(See `sassessment.ipynb` for code and visualizations)*


## Part 4: Mini System Design

### High Level Architecture
`Data Source` -> `Python Script (.pkl model)` -> `Database (Risk Scores)` -> `User Interface (Daily Email + Alerts)`

### Data Flow
1.  The system fetches all open deals and their history from the CRM.
2.  The Python script runs the pre-processing pipeline.
3.  The pre-trained model (`.pkl` file) generates a `risk_score` (0-100%) for every deal.
4.  Scores are saved to the Database for historical tracking.
5.  **Alert Logic:** If `risk_score` > Threshold (e.g., 60%), an alert is triggered.

### Example Alerts & Insights
**Email to Sales Manager (Daily):**
> **🚨 Risk Alert: Deal D01413 (EdTech)**
> * **Risk Score:** 62.1% (High)
> * **Reason:** Stuck in 'Qualified' for 119 days.
> * **Recommended Action:** Call client today or mark as 'Lost' to clean pipeline.

*Note: A weekly digest can also be sent containing total pipeline value vs. value at high risk.*

### Failure Cases & Limitations
* **Cold Start:** If there is a new sales rep, the model will lack historical performance data for them, potentially leading to inaccurate predictions.
* **Data Latency:** If a sales rep forgets to update the CRM (e.g., they had a demo but didn't log it), the model will use old data, flagging a healthy deal as "stagnant."

---

## Part 5: Reflection

### What assumptions in your solution are weakest?
* This data does not count deals that might resurface after 6 months or 1 year. Here, "Lost" is treated as a permanent state.
* In reality, sales data is dynamic. A sales rep might have had a call but just didn't log it in the data yet. This might lead the model to flag the deal incorrectly, affecting trust.

### What would break in real-world production?
* **Scalability:** If data grows to millions of records, the current script (loading all data into memory) will fail. We would need to process in chunks or use a data warehouse.
* **Gaming the System:** If sales reps know that "Qualified" deals hurt their score, they might fake-promote deals to the "Demo" stage just to avoid getting flagged, corrupting the data.

### What would you build next if given 1 month?
* **Feature Engineering:** I would add more behavioral data, such as "Time since last email/call" or "Number of stakeholders engaged," to make the risk score more robust.
* **Interactive Feedback:** I would add a system where a sales rep is notified immediately about the change in risk score whenever they take an action on a lead (e.g., "Great job! Moving this to Negotiation lowered risk by 15%").

### What part of your solution are you least confident about?
* **Accuracy:** The model accuracy is ~55%. While helpful for ranking, it is not perfect.
* **Lead Source Generalization:** I am unsure if the `lead_source` patterns (e.g., "Partner" leads) hold true across all regions or if they are specific to certain markets.