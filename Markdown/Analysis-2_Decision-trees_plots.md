Analysis 2: Decision trees plots
================

## Overview

This code generates

1.  a decision tree per quartile for the which N (or More) task.

2.  a series of graphs showing the individual trial performance for each
    of the terminal for each decision tree.

## Load libraries & data

``` r
library(party)
```

    ## Loading required package: grid

    ## Loading required package: mvtnorm

    ## Loading required package: modeltools

    ## Loading required package: stats4

    ## Loading required package: strucchange

    ## Loading required package: zoo

    ## 
    ## Attaching package: 'zoo'

    ## The following objects are masked from 'package:base':
    ## 
    ##     as.Date, as.Date.numeric

    ## Loading required package: sandwich

``` r
library(tidyverse)
```

    ## ── Attaching packages ─────────────────────────────────────── tidyverse 1.3.1 ──

    ## ✓ ggplot2 3.3.5     ✓ purrr   0.3.4
    ## ✓ tibble  3.1.2     ✓ dplyr   1.0.6
    ## ✓ tidyr   1.1.3     ✓ stringr 1.4.0
    ## ✓ readr   1.4.0     ✓ forcats 0.5.1

    ## ── Conflicts ────────────────────────────────────────── tidyverse_conflicts() ──
    ## x stringr::boundary() masks strucchange::boundary()
    ## x dplyr::filter()     masks stats::filter()
    ## x dplyr::lag()        masks stats::lag()

``` r
library(here)
```

    ## here() starts at /Users/leyu6965/Dropbox/GitHub/Statistical-learning-and-the-development-of-knowledge-systems

``` r
library(caret)
```

    ## Loading required package: lattice

    ## 
    ## Attaching package: 'caret'

    ## The following object is masked from 'package:purrr':
    ## 
    ##     lift

``` r
# CHAID package is not on CRAN, so special install 
#install.packages("CHAID", repos="http://R-Forge.R-project.org")
#library(CHAID)
# library(partykit) # partykit is only required by CHAID. DO NOT use it to generate the original trees
#detach("package:partykit", unload=TRUE) # detach is loaded 
```

## Load data (only need to change this for the More task analysis)

``` r
# indicate the task name, make it easier for adopting the same code for both the N and M tasks
task_name = "n"
#task_name = "more"

# input files
data_long = read.csv(here(paste0("Data/data_", task_name,"_long.csv")))

data_item_binomial = read.csv(here(paste0("Data/data_", task_name,"_item_binomial.csv")))

# within each quartile, some items don't have at least 10 children responses. exclude them from the item level accuracy graph, but not the overall analysis???
min_item_n = 10

data_item_binomial = data_item_binomial %>% 
  filter(item_total_n >= min_item_n) 
```

## Grow Ctrees

``` r
set.seed(240)

# make sure that all feature cols are factors
data_long = data_long %>%
  mutate(bigger_places = as.factor(bigger_places),
         places = as.factor(places),
         length_diff = as.factor(length_diff),
         one_digit_diff_not_zero = as.factor(one_digit_diff_not_zero),
         transposition = as.factor(transposition),
         inserting_zero = as.factor(inserting_zero))

# initialize empty variables 
prediction_acc_results  = NULL  # save confusion matrix results for each tree
data_long_terminal = NULL # save the terminal node number for each row (or trial), for plotting the item-level accuracy graphs for each quartile

for (i in 1:4) {
  # get quartile data
  temp_data = subset(data_long, quartile == i) 
  
  # grow ctree
  ct = ctree(factor(acc) ~ 
               bigger_places + places + length_diff + one_digit_diff_not_zero +
               transposition + inserting_zero, data = temp_data)
  
  # plot ctree and save it
  filename = here(paste0("Plots/trees/", task_name,"_ctree_q",i,".jpeg"))
  jpeg(filename, unit = "px", height = 4500, width = 4000, res = 300)
  plot(ct) # party can not change font size (but can change file size), partykit can and here is the syntax: gp = gpar(fontsize = 4))
  dev.off()
  
  # # save the tree
  # data_ct = here("Plots", "trees", paste0(task_name, "_ct_results_", i, ".txt"))
  # sink(data_ct)
  # print(ct)
  # sink()
  # 
  # # calculate prediction accuracy
  # # use the default 50% threshold
  # original_predict_value = predict(ct, temp_data, type = "response") 
  # # set new threshold
  # predict_prob_list = predict(ct, temp_data, type = "prob")
  # predict_prob = as.data.frame(do.call(rbind,predict_prob_list)) # converst from list to dataframe
  # new_predict_value = predict_prob %>%
  #   mutate(predict_value = ifelse(V2 > 0.5, 1, 0))
  # # create contingency table
  # table(new_predict_value$predict_value, temp_data$acc)
  # # print out prediction accuracy
  # prediction_acc = 1 - mean(new_predict_value$predict_value != temp_data$acc) 
  # prediction_acc_results = rbind(prediction_acc_results, prediction_acc)
  # 
  # # save terminal node number back to the long format data: the function "where()" show all trial data rows with terminal node number
  temp_data = cbind(temp_data, where(ct)) %>%
  rename(terminal_node = `where(ct)`)
  data_long_terminal = rbind(data_long_terminal, temp_data)
}

# save prediction accuracy in file
write.csv(prediction_acc_results, here(paste0("Data/", task_name, "ctree_model_accuracies.csv")), row.names = FALSE)

# current_node = nodes(ct, where = 1)
# current_weight = current_node[[1]]$weights
# test = cbind(temp_data, current_weight) %>%
#  select(places, current_weight)
```

## Getting more terminal overall performance and item graphs

``` r
# Summarize terminal nodes overall accuracy
data_terminal_summary = data_long_terminal %>%
  group_by(quartile, terminal_node) %>%
  summarise(acc_terminal = mean(acc), total_n = n(), success_n = sum(acc)) %>%
  rowwise() %>%
  mutate(binomial_p = round(binom.test(success_n, total_n, 0.5)$p.value, 4))
```

    ## `summarise()` has grouped output by 'quartile'. You can override using the `.groups` argument.

``` r
# select items in different terminals and link it to binomial test results file to generate the item accuracy graph for each terminal
data_long_terminal_shortened = data_long_terminal %>% # extract only useful info: quartile_item, and terminal node number
  group_by(quartile, item, terminal_node) %>%
  summarise(count = n()) %>%
  filter(count >= min_item_n) %>% # within each quartile, some items have lower n and maybe excluded, using the min_item_n set at the beginning as the threshold. 
  mutate(quartile_item = paste(quartile, item)) %>% # create a unique key to be linked to the binomial data
  ungroup() %>%
  select(quartile_item, terminal_node)
```

    ## `summarise()` has grouped output by 'quartile', 'item'. You can override using the `.groups` argument.

``` r
# read in item by quartile binomial result data and combine with terminal node number
data_item_binomial_terminal = data_item_binomial %>%
  mutate(quartile_item = paste(quartile, item)) %>%
  left_join(data_long_terminal_shortened, by = "quartile_item")

# # total item number for each quartile
# total_item_num_quartile = data_item_binomial_terminal %>%
#   group_by(quartile, terminal_node) %>%
#   summarise(item_count = n())
# 
# max_item = max(total_item_num_quartile$item_count)

# plot the item level accuracy graph for each terminal
for (i in 1:4) {
  fontsize = 9
  # get current quartile data
  temp_quartile = subset(data_item_binomial_terminal, quartile == i) %>%
    droplevels() # so don't interfere with the unique function later
  
  # calculate total node number for the current terminal
  nodes = unique(temp_quartile$terminal_node)
  total_node_num = length(nodes)
  
  # generate the item level plot
  for (j in 1:total_node_num) {
    # set fontsize
    fontsize = 9
    
    # # figure out the resized height (relative to the max number of items for all quartile all terminal)
    # total_item_num = subset(total_item_num_quartile,quartile == i & terminal_node == nodes[j])$item_count
    # resized_height = total_item_num/max_item 
    
    # select current data
    data_item_temp = subset(temp_quartile, terminal_node == nodes[j])
    
    # plot
    ggplot(data = data_item_temp, aes (x = reorder(item, item_acc), y = item_acc, label = star)) +
      geom_segment(aes(x=reorder(item, item_acc), xend=reorder(item, item_acc), y=0, yend=item_acc), color="grey", size = 0.5) +
      geom_point( color="orange", size=2) +
      theme_bw() + theme(panel.border = element_blank(), panel.grid.major = element_blank(),
                         panel.grid.minor = element_blank(), axis.line = element_line(colour = "black")) +
      #ggtitle("The Which-N task") +
      #ylab("Proportion of correct trials") + 
      #xlab("Items") +
      geom_hline(yintercept=0.5, linetype="dashed", color = "orange", size = 0.5) +
      theme(plot.title = element_text(size = fontsize, hjust = 0.5), text=element_text(size=fontsize),
            axis.title.x=element_blank(), axis.title.y = element_blank()) +
      coord_flip() +
      geom_text(data = data_item_temp, aes(label = star), nudge_y = -0.05, size = 4) +
      ylim(c(0,1.05))
    
    loc_file = here("Plots", "trees", paste0(task_name,"_tree_items","_quartile", i,"_node", nodes[j],".jpeg"))
    ggsave(loc_file, height = 3.5, width = 2, dpi = 300, limitsize = FALSE)
  }
}
```
