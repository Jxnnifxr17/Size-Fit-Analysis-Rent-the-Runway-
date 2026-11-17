# Size & Fit Analysis: Where Do Garments Miss?

Analysis of 192,544 Rent the Runway reviews to find which product categories and sizes have the highest fit-mismatch rates, who is affected, and what sizing changes would reduce return risk.  

Interactive dashboard: Open the dashboard (fit_dashboard.html is the same page as a standalone file)  

## Business question
 
Which categories and sizes carry the most fit risk, which customers are affected, and what should a merchandising or product team change?  

Fit complaints drive returns, which cost money and hurt customer experience. This project turns review data into specific, auditable sizing recommendations.  

## Key findings

| Finding | Evidence |
| :--- | :--- |
| About 1 in 4 reviews reports a fit problem, split evenly between small and large. | 26.2% mismatch (13.4% run small, 12.8% run large) |
| Coats, pants, and shifts have the highest mismatch rates. | Coat: 40.4%, Pants: 37.7%, Shift: 31.6% |
| Coats skew large; skirts and jackets skew small. | Coat: 36.7% large vs. 3.7% small |
| Risk concentrates in specific category-size cells. | Coat size 4–6: 47% mismatch; Mini dress size 16–20: 45% mismatch |
| Size rented relative to the body is the strongest predictor. | Renting a bigger size than typical for your body: 24.7% say "runs small" vs. 6.5% for the smallest group (odds ratio 1.69 per SD) |
| Tall customers report length problems in some categories. | "Too short" mentioned (tall vs. short): 12.0% vs. 2.8% in jumpsuits, 15.2% vs. 6.2% in rompers, 15.5% vs. 7.3% in shifts |


Recommendations generated: 63 (45 "runs large", 15 "runs small", 3 length), for example add a "runs large, consider sizing down" note for coats in sizes 4-6 or offer a tall option for jumpsuits. Full list in outputs/sizing_recommendations.csv.  

## Approach  
- Clean: parse height and weight, remove implausible values, bucket sizes (0-2, 4-6, 8-10, 12-14, 16-20, 22+), compute BMI, lump categories with fewer than 300 reviews into "other".
- Mine review text: negation-aware regex rules extract fit phrases ("runs small", "too tight") plus problem areas and length complaints. Text flags the item's sizing tendency; the label records the customer's outcome. Both are reported (see Notes).
- Measure risk: mismatch rates by category and category × size with Wilson 95% confidence intervals. A cell is flagged only when its lower bound exceeds the overall rate and the gap is at least 3 percentage points, so large categories don't flag on trivial differences.
- Engineer "size vs. body": the gap between the size rented and the size typical for that height and BMI. Raw size and BMI correlate at 0.75, which muddies model coefficients.
- Model: logistic regression (odds ratios, for explanation) and gradient boosting (for prediction), compared against an "always fit" baseline. rating is excluded to avoid leakage.
- Recommend: transparent rules turn flagged cells and tall-vs-short length gaps into actions a merchandiser can audit.

## Model results

| Model | Macro-F1 | AUC (one-vs-rest) |
| :--- | :---: | :---: |
| Baseline (always "fit") | 0.283 | 0.500 |
| Logistic regression | 0.341 | 0.634 |
| Gradient boosting | 0.369 | 0.651 |


Fit is hard to predict from measurements alone, so the models mainly support the category, size and height findings. Permutation importance ranks size vs. body first, ahead of category and body type.

## Repository contents

| File / Directory | Description |
| :--- | :--- |
| `fit_analysis.ipynb` | Notebook with saved outputs (run order matches sections above) |
| `fit_analysis.py` | Same pipeline as a script |
| `fit_dashboard.html` | Standalone interactive dashboard |
| `outputs/fit_by_category.csv` | Mismatch, small and large rates by category (with CIs) |
| `outputs/fit_by_category_size.csv` | Rates by category x size bucket (n >= 40) |
| `outputs/fit_by_size_vs_body.csv` | Rates by size relative to body |
| `outputs/height_effects_by_category.csv` | Tall vs. short comparisons |
| `outputs/sizing_recommendations.csv` | Flagged cells and actions |
| `outputs/model_metrics.csv`, `outputs/summary.txt` | Model evaluation metrics and text summary |
| `outputs/figures/` | Charts used in the write-up |


The CSVs are formatted for Tableau Public if you want to rebuild the dashboard there.

## Reproduce it
- Download the Rent the Runway dataset (renttherunway_final_data.json) from the McAuley Lab public datasets page at UCSD. The data is not included here.  
- Place it next to the notebook.  
- Install dependencies: pip install pandas numpy scikit-learn statsmodels matplotlib  
- Run fit_analysis.ipynb, or python fit_analysis.py.  

Outputs are written to outputs/. The script runs in a few minutes on a laptop.  

## Notes on interpretation
- Text vs. label: when a review says an item "runs small", the reviewer's own fit label is still "fit" about half the time, because renters often sized up on purpose. Text recall against the labels is under 40%, so the text rules are a complement to the labels, not a replacement.
- Size vs. body is an association in self-reported data. One plausible reading is that renters adjust their size using item notes but under-correct. This is a hypothesis, not a causal claim.
- Small samples: the coat (980 reviews) and pants (422) findings rest on modest counts. The confidence intervals account for this, but word conclusions carefully.

## Limitations
- No return flag. "Ran small/large" is a proxy for return risk, not a measured return rate.
- Occasion-wear renters. Findings may not generalize to everyday apparel or purchased items.
- Self-reported labels and measurements, with missing height or weight for roughly 15% of reviews.
- Rule-based text extraction misses many phrasings.
- Cost estimates in the notebook (section 8) use illustrative assumptions (30% return rate, $18 per return), not values from the data. Replace them with sourced figures before quoting dollar impact.

## Possible extensions
- Replace regex extraction with an embedding or fine-tuned classifier and measure the recall gain.
- Add item-level analysis (item_id) to find individual products with chronic sizing problems.
- Validate on a dataset with actual returns, such as retailer data.
- Test whether the recommended fit notes change mismatch rates, using an A/B design.

## Data citation
Misra, R., Wan, M., McAuley, J. Decomposing fit semantics for product size recommendation in metric spaces. RecSys 2018.
