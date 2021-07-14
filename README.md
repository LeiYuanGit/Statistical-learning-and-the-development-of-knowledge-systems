# Statistical-learning-and-the-development-of-knowledge-systems

## Abstract
Human knowledge of the world is often organized into systems of inter-related, multi-level, relational structures (e.g., language, symbolic numbers, physical systems). A perennial question has been how these knowledge systems are learned during development, or if they are indeed learnable at all, as opposed to constrained by prior, innately specified biases. Going beyond individual patterns (e.g., learning novel countable nouns or transitive verbs) and toy examples (e.g., learning ABA, AAB patterns) examined in the laboratory, in the current study, we trace the development of one knowledge system—how multi-digit symbolic numbers are named and how they are used to represent discrete quantities—during the preschool and kindergarten period (3- to 6-year-olds). Using a corpus data analysis approach, we combined several datasets from 678 children who contributed 13039 individual responses in two different tasks (i.e., identifying a named multi-digit number between two, or choosing a numerically larger number between two). Combining traditional statistical analyses methods, recursive partitioning machine learning algorithms and network analysis, we show how learning starts out with piecemeal knowledge and good enough solutions (as opposed to be guided by clean rules or biases), but gradually make connections among the various individual regularities and integrate them into a coherent knowledge system. Results have implications on the development of knowledge systems and school teaching of symbol systems.

## Data source files
- [data_n_raw_wide.csv](Data/data_n_raw_wide.csv) contains 404 children's responses to the which-N task
- [data_more_raw_wide.csv](Data/data_more_raw_wide.csv) contains 465 children's responses to the which-More task
- xx children have recieved both tasks. 

## Code/analysis files
- [Data cleaning.md](Markdown/Data cleaning.md)
This code cleans the raw N and More participants data:

1. convert data from wide to long format
2. delete NA items
3. create a separate subj level accuracy column and use it to categorize children into 4 quartiles
4. adding item feature information
5. output the results into new csv files for further analysis 

It also identifies children who had completed both tasks, by concatenating their "DOB, DOE, and location" and find common values both the n and more data.

- [Analysis 1_Overall descriptive results.Rmd](Markdown/Analysis 1_Overall descriptive results.Rmd) 
This code provides an overall view of children's performance in the two tasks. 