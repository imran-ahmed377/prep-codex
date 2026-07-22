# A/B Testing
## ⚡ One-line intuition

**A/B testing = “Show two versions to similar users and check if the difference is real or just luck.”**

## 🎯 Scenario

You run an e-commerce app and want to test:

**Does a new checkout button increase purchases?**

---

## 1. Define the hypothesis

Keep it very clear and measurable.

- **Null hypothesis (H₀):** New button does not change conversion rate  
- **Alternative (H₁):** New button increases conversion rate  

**Example:**  
Current conversion = 10%  
You expect → 12%

---

## 2. Choose your metric

Pick one primary metric (don’t overcomplicate).

**Example:**
- Primary: % of users who complete purchase  
- Secondary (optional): revenue per user  

---

## 3. Decide group split (A vs B)

Most common: **50/50 split**

- Group A → old button  
- Group B → new button  

**Why 50/50?**
- Fastest way to get results  
- Balanced comparison  

Sometimes companies use 90/10 for safer rollout, but 50/50 is standard for testing.

---

## 4. Determine sample size (how many users you need)

You need enough users to detect a real difference.

**Key inputs:**
- Baseline = 10%  
- Expected improvement = +2% (to 12%)  
- Confidence level = 95%  
- Power = 80%  

Using a calculator or stats tool:

👉 You’ll need roughly **~4,000–5,000 users per group**

_(You don’t calculate this manually in practice—tools do it.)_

---

## 5. Determine experiment duration

Duration depends on traffic.

**Example:**
- You get 1,000 users/day  
- Need 5,000 per group → 10,000 total  

👉 Duration ≈ **10 days**

**Add buffer:**
- Run at least 1–2 full weeks to capture weekday/weekend behavior  

👉 Final: **2 weeks**

---

## 6. Run the experiment properly

**Rules:**
- Randomly assign users  
- Keep conditions the same (no other changes)  
- Don’t stop early just because results “look good”  

---

## 7. Analyze results

Compare conversion rates using a statistical test like a t-test or proportion test.

**Example result:**
- A = 10%  
- B = 12%  
- (Chance of result is random, low p-value is good) p-value < 0.05 → significant  

👉 New button wins  

---

## 8. Decide rollout strategy

Don’t always go straight to 100%.

**Typical rollout:**
- Step 1: 10% users  
- Step 2: 50% users  
- Step 3: 100% users  

**Why gradual?**
- Catch bugs or unexpected issues  
- Reduce risk  

---

## 🧠 Simple summary (memory version)

- Hypothesis: what you expect to change  
- Metric: what you measure  
- Split: usually 50/50  
- Sample size: enough users to detect change  
- Duration: based on traffic (usually 1–2 weeks min)  
- Run test: random + controlled  
- Analyze: is difference real?  
- Rollout: gradual release  

---

# Product & Business 

1. Translate a business problem into a data science problem? 

- Clarify the goal
- define success metrics, 
- identify target variable, 
- gather relevant data, 
- frame it as a predictive or analytical problem.

2. Tell me about a time your model impacted business decisions.
- Built an A/B test model improving conversion by 5%, 
- leading the team to adopt the new design and increase revenue.

3. How do you handle ambiguity when you had no clear data or direction?
- Clarify goals, 
- Make reasonable assumptions, 
- Explore available data, 
- Prioritize quick experiments, 
- Iterate while aligning with stakeholders for direction.
---