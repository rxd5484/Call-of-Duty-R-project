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

