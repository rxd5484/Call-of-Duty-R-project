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

r
Copy
Edit
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

r
Copy
Edit
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

r
Copy
Edit
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

r
Copy
Edit
ggplot(winProb, aes(x = reorder(Map, WinProbability), y = WinProbability)) +
  geom_bar(stat = "identity", fill = "purple") +
  coord_flip()
Interpretation:

Maps like Nuketown '84, Crossroads Strike, and Raid are most likely to win.

Maps like Miami or Echelon rarely get selected.


