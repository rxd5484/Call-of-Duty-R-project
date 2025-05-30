# Call-of-Duty-R-project

![image](https://github.com/user-attachments/assets/000e46de-f9bf-464e-8179-dffde0c029ff)


🧠 Fuzzy Matching to Fix Spelling Errors
Voting data sometimes contains small typos or inconsistent spacing in map names (e.g., "miami strike" vs. "miami strik "). To ensure accurate analysis, we use fuzzy string matching to clean this up.

🔍 How It Works
We define a custom function correct_map_name() that:

Leaves blanks and valid names untouched

If the map name is NA or empty, it returns as-is.

If the name already exists in our reference list, it's left unchanged.

Corrects only incorrect names

Uses Levenshtein distance (via stringdist::stringdist) to find the closest match from the list of valid map names.

This is applied via sapply() to each of the map name columns: Map1, Map2, and Choice.


# Example usage:
votesDF <- votesDF %>%
  mutate(
    Map1 = sapply(Map1, correct_map_name, reference_maps),
    Map2 = sapply(Map2, correct_map_name, reference_maps),
    Choice = sapply(Choice, correct_map_name, reference_maps)
  )
🧾 Why It Matters
Without this step, small typos would fragment the data:

"drivein" and "drive-in" would be treated as different maps.




Charts and stats would be misleading or incomplete.

By standardizing map names upfront, we ensure accurate filtering, counting, and visualization later in the pipeline.
<img width="1073" alt="Screenshot 2025-05-30 at 5 27 17 PM" src="https://github.com/user-attachments/assets/5c0ee5a0-aa9a-4879-9e5a-1331aeb48ebf" />
<img width="1008" alt="Screenshot 2025-05-30 at 5 29 18 PM" src="https://github.com/user-attachments/assets/fbd89ab6-8c6c-42b3-9b05-4a8f4a021a65" />


To ensure a clean dataset, we remove any rows where one or more of the columns Map1, Map2, or Choice is missing or empty. This guarantees we're only analyzing complete vote records.


votesDFFiltered <- votesDF %>%
  filter(
    !is.na(Map1) & Map1 != "",
    !is.na(Map2) & Map2 != "",
    !is.na(Choice) & Choice != ""
  )
Why this matters:
Missing or malformed data can skew results. This step ensures that only complete and meaningful rows are passed into the analysis.




<img width="1081" alt="Screenshot 2025-05-30 at 5 29 04 PM" src="https://github.com/user-attachments/assets/16786ea7-dab7-4abc-ba91-ba03b08c9c81" />


🏆 Counting Wins for Each Map

This step determines how many times each map wins a vote.


winCounts <- votesDFFilteredWithTies %>%
  mutate(winningMap = case_when(
    isTie ~ Map1,    # Treat ties as wins for Map1
    TRUE ~ Choice    # Otherwise, take the actual chosen map
  )) %>%
  group_by(winningMap) %>%
  summarise(Wins = n()) %>%
  arrange(desc(Wins))
Additional Step:
Ties are tracked separately using a boolean isTie column derived from parsing vote counts.

Why this matters:
This helps rank maps based on how often they win, either by actual votes or due to ties.











We visualize the win probability of each map when it's shown as a voting option. This includes both wins by majority vote and ties (when Map1 is chosen by default).


ggplot(winProb, aes(x = reorder(Map, WinProbability), y = WinProbability)) +
  geom_bar(stat = "identity", fill = "purple") +
  coord_flip()
Interpretation:

Maps like Nuketown '84, Crossroads Strike, and Raid are most likely to win.

Maps like Miami or Echelon rarely get selected.











<img width="973" alt="Screenshot 2025-05-30 at 5 29 38 PM" src="https://github.com/user-attachments/assets/7bcb0839-0998-4cb4-9117-1abadd2d71a6" />



🌈 Heatmap-Style Bar Plot with Gradient

This plot shows the same win probability but uses a color gradient for better visual impact. Darker shades indicate higher popularity.


scale_fill_gradient(low = "lightblue", high = "darkblue")
Why use this version?
It’s visually appealing and draws quick attention to dominant maps.




<img width="1005" alt="Screenshot 2025-05-30 at 5 29 48 PM" src="https://github.com/user-attachments/assets/5f2a0328-ed7e-4978-ac71-04c6237cc9b4" />



🪄 Split by Win Type (Votes vs. Ties)

We break down how each map won:

Cyan: Won by actual vote.

Pink: Won by tie (Map1 automatically wins).


geom_bar(aes(fill = WinType), stat = "identity", position = "stack")
Insight:

Maps like Nuketown win mostly by actual votes.

Some lower-tier maps only win when there’s a tie


<img width="984" alt="Screenshot 2025-05-30 at 5 30 14 PM" src="https://github.com/user-attachments/assets/9cb5211c-3a04-4353-a05f-6eff15a8902d" />


📈 Score vs. XP by Game Type

This scatter plot visualizes the relationship between Score and Total XP across different game modes using colored regression lines.

🔍 What It Shows
Each point represents a match played, plotted by:

X-axis: Score earned in the match

Y-axis: Total XP gained

Colored trend lines indicate the linear relationship for each GameType:

🟥 Domination (red)

🟩 Hardpoint (green)

🟦 Kill Confirmed (blue)

🟪 TDM (purple)

💡 Key Insights
📈 Hardpoint has the steepest slope, meaning XP increases fastest with score—great for grinding.

🎯 TDM and Kill Confirmed show lower XP return per point scored.

🎯 Domination sits somewhere in between.

🧠 Interpretation
This visualization helps players identify which game mode offers the best XP-per-score payoff. Players aiming to level up quickly should favor Hardpoint based on this dataset.





<img width="885" alt="Screenshot 2025-05-30 at 5 30 21 PM" src="https://github.com/user-attachments/assets/3021bfcf-b836-4632-af91-3038d5efa699" />


📦 Total XP Distribution by Game Type

This boxplot illustrates how Total XP earned varies across different game modes. Each box represents the statistical distribution of XP outcomes for one game type.

🔍 What It Shows
X-axis: Game modes (Kill Confirmed, TDM, Hardpoint, Domination)

Y-axis: Total XP earned per match

Each box displays:

Median (center line)

Interquartile Range (IQR) (box)

Whiskers (range excluding outliers)

Outliers (individual dots beyond 1.5×IQR)

💡 Key Insights
🟩 Hardpoint shows the widest XP spread, with some of the highest outliers—great potential for XP farming.

🟥 Domination has the highest median, indicating strong XP consistency.

🟪 TDM and 🟦 Kill Confirmed tend to offer lower XP on average, with Kill Confirmed having the tightest (lowest) spread.

🧠 Interpretation
While all game modes can yield XP, players seeking both high consistency and ceiling might prefer Domination or Hardpoint. Kill Confirmed, while predictable, generally offers lower XP returns.











![image](https://github.com/user-attachments/assets/e6258b4f-549f-476f-a3af-7e497a6a4aaa)



📐 Predicted Total XP by Score and Game Type

This plot visualizes model-predicted Total XP as a function of score, broken down by game mode. It shows how XP is expected to increase with performance (score), after controlling for game type.

🔍 What It Shows
X-axis: Score earned during a match

Y-axis: Predicted Total XP (from a regression model)

Colored lines show predicted XP gain curves for each game type:

🟥 TDM

🟩 Kill Confirmed

🟦 Hardpoint

🟪 Domination

📊 Interpretation
🟪 Domination leads in XP gain at every score level — it's the most XP-efficient mode overall.

🟦 Hardpoint performs second-best, especially after mid-level scores.

🟩 Kill Confirmed and 🟥 TDM yield less XP for the same score, especially at higher performance levels.

The gap widens as score increases, emphasizing that mode choice matters more for high-performing players.

🧠 Why This Matters
This chart reveals not just correlations, but model-backed expected outcomes. It helps answer:

“If I score the same in any mode, which one gives me the most XP?”

The answer? Domination, hands down.


<img width="894" alt="Screenshot 2025-05-30 at 5 31 00 PM" src="https://github.com/user-attachments/assets/d0ee0b2d-2a78-4767-9f22-86892e2af106" />


Model Comparison – ROC AUC Scores

This boxplot compares the ROC AUC (Receiver Operating Characteristic Area Under Curve) scores of three machine learning models: XGBoost, Random Forest, and Logistic Regression.

Key Observations:
XGBoost shows the strongest overall performance. It has the highest median ROC score and relatively tight variability, indicating consistently strong classification ability across folds or runs.

Random Forest performs well, though not as consistently. Its ROC scores vary more, with a wider interquartile range, but its upper performance is competitive with XGBoost.

Logistic Regression has the lowest median ROC score, and a narrow range, meaning it performs consistently but less effectively than the other models in this setup.

Summary:
This comparison suggests that XGBoost is the best choice in terms of classification accuracy for this task, followed by Random Forest, while Logistic Regression underperforms in comparison.



<img width="983" alt="Screenshot 2025-05-30 at 5 31 10 PM" src="https://github.com/user-attachments/assets/c594958f-823a-4ded-85cf-51dd09a595ae" />

ROC Curve – Random Forest vs. Logistic Regression

This plot displays the ROC curve (Receiver Operating Characteristic) for the Random Forest classifier. It shows the tradeoff between the true positive rate (sensitivity) and the false positive rate at various classification thresholds.

Key Points:
The solid blue line is the Random Forest's ROC curve.

The dashed diagonal line represents random guessing (baseline performance).

The closer the curve is to the top-left corner, the better the model is at distinguishing between classes.

Performance Summary:
Random Forest demonstrates moderate performance, with the ROC curve staying above the diagonal for most of its range.

Logistic Regression (not shown in the plot but noted below) has an AUC of 0.4417, which is below random guessing (AUC < 0.5), suggesting poor classification performance on this dataset.

Conclusion:
Random Forest performs significantly better than Logistic Regression in this classification task, based on the shape of the ROC curve and the implied area under the curve.



<img width="1040" alt="Screenshot 2025-05-30 at 5 31 17 PM" src="https://github.com/user-attachments/assets/f3155f5d-50be-4592-96bd-a6b6d1a56dcf" />


ROC Curve – Logistic Regression with XGBoost AUC Comparison

This ROC curve shows the classification performance of the Logistic Regression model.

What the Plot Shows:
The blue curve represents the tradeoff between the true positive rate and the false positive rate at different threshold values.

The dashed diagonal line indicates the performance of a random classifier. A model performing above this line is making useful predictions.

In this case, the curve barely rises above the diagonal, reflecting relatively weak discriminatory ability.

AUC Scores:
Logistic Regression AUC is approximately 0.50, suggesting near-random performance.

XGBoost, noted just below the plot, achieved an AUC of 0.6478, which is a substantial improvement.

Interpretation:
Logistic Regression performs poorly on this dataset when compared to more advanced models like XGBoost. The XGBoost model demonstrates better separation between classes, as evidenced by its higher AUC score.


<img width="1027" alt="Screenshot 2025-05-30 at 5 31 25 PM" src="https://github.com/user-attachments/assets/e5a81b9d-0c36-4f5d-a4d9-d2d96ebd176a" />



ROC Curve and Performance Metrics – XGBoost

This plot shows the ROC curve for the XGBoost classifier, representing how well the model distinguishes between classes across all decision thresholds.

ROC Curve Insights
The curve lies well above the diagonal, indicating strong classification performance.

Compared to the baseline (dashed line), XGBoost maintains a high true positive rate with a relatively low false positive rate.

Classification Metrics Summary
Model	Accuracy	Precision	Recall	F1 Score
Logistic Regression	0.40	0.17	0.53	0.26
Random Forest	0.63	0.26	0.47	0.33
XGBoost	0.65	0.32	0.67	0.43

Key Observations
XGBoost outperforms both Random Forest and Logistic Regression across all major metrics: accuracy, precision, recall, and F1 score.

It achieves the best balance between precision and recall, as reflected in the highest F1 score.

The model is especially effective at minimizing false negatives while maintaining reasonable precision.

Conclusion
XGBoost is the most effective classifier in this analysis, offering both high predictive power and a strong tradeoff between sensitivity and precision.











