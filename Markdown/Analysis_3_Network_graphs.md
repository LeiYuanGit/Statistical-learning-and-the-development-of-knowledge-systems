Network graphs
================

## Overview

This code generates one network for each quartile.

-   The nodes represent the type (i.e., single, decade, hundred,
    thousand) of the two numbers in each trial.
-   The edges represent the average accuracy for that type of trial (I
    only plot if the average accuracy is above 70%).
-   The different colors of the edges further denote whether the trial
    involves zeros, transposition, or neither.

### Step 0:

Load all necessary packages

### Step 1:

Read in source file: data\_n\_long.csv

``` r
# set the task name
 task = "n"
# task = "more"

# load data
data = read.csv(here(paste0("Data/data_", task, "_long.csv")), header=TRUE, sep=",", check.names = "FALSE") 

# inspect it
str(data)
```

    ## 'data.frame':    6178 obs. of  33 variables:
    ##  $ excel_file_name          : chr  "Meap 3 Data only/sheet: combined data" "Meap 3 Data only/sheet: combined data" "Meap 3 Data only/sheet: combined data" "Meap 3 Data only/sheet: combined data" ...
    ##  $ id                       : int  31 32 33 34 35 36 37 38 39 40 ...
    ##  $ sex                      : chr  NA NA NA NA ...
    ##  $ dob                      : chr  "9/19/10" "5/6/10" "12/10/09" "11/8/09" ...
    ##  $ doe                      : chr  "5/22/13" "5/21/13" "5/21/13" "5/21/13" ...
    ##  $ age_months               : num  32.1 36.5 41.4 42.4 44.1 ...
    ##  $ age_years                : num  2.68 3.05 3.45 3.54 3.67 3.99 4.02 4.07 4.11 4.11 ...
    ##  $ age_years_group          : int  2 3 3 3 3 3 4 4 4 4 ...
    ##  $ location                 : chr  "KA" "KA" "KA" "KA" ...
    ##  $ percentage_correct       : num  0.562 0.375 0.438 0.625 0.75 ...
    ##  $ ses                      : int  1 1 1 1 1 1 1 1 1 1 ...
    ##  $ within_id                : chr  "9/19/10 5/22/13 NA KA" "5/6/10 5/21/13 NA KA" "12/10/09 5/21/13 NA KA" "11/8/09 5/21/13 NA KA" ...
    ##  $ twins                    : chr  "n" "n" "n" "n" ...
    ##  $ item                     : chr  "2 v 8" "2 v 8" "2 v 8" "2 v 8" ...
    ##  $ acc                      : int  1 1 1 1 1 1 1 1 1 1 ...
    ##  $ subj_acc                 : num  0.562 0.375 0.438 0.625 0.75 ...
    ##  $ quartile                 : int  1 1 1 2 2 2 1 1 1 1 ...
    ##  $ bigger_places            : int  1 1 1 1 1 1 1 1 1 1 ...
    ##  $ places                   : chr  "1v1" "1v1" "1v1" "1v1" ...
    ##  $ length_diff              : chr  "n" "n" "n" "n" ...
    ##  $ one_digit_diff_not_zero  : chr  "y" "y" "y" "y" ...
    ##  $ transposition            : chr  "n" "n" "n" "n" ...
    ##  $ inserting_zero           : chr  "n" "n" "n" "n" ...
    ##  $ e3_categories            : chr  "single" "single" "single" "single" ...
    ##  $ category                 : chr  "S" "S" "S" "S" ...
    ##  $ left_num                 : int  2 2 2 2 2 2 2 2 2 2 ...
    ##  $ right_num                : int  8 8 8 8 8 8 8 8 8 8 ...
    ##  $ left_number_range        : chr  "single" "single" "single" "single" ...
    ##  $ right_number_range       : chr  "single" "single" "single" "single" ...
    ##  $ item_comparison_unordered: chr  "single_single" "single_single" "single_single" "single_single" ...
    ##  $ item_comparison          : chr  "single_single" "single_single" "single_single" "single_single" ...
    ##  $ item_type                : chr  "Neither" "Neither" "Neither" "Neither" ...
    ##  $ completed_task           : int  2 2 2 2 2 2 2 2 1 1 ...

``` r
head(data) 
```

    ##                         excel_file_name id  sex      dob     doe age_months
    ## 1 Meap 3 Data only/sheet: combined data 31 <NA>  9/19/10 5/22/13         32
    ## 2 Meap 3 Data only/sheet: combined data 32 <NA>   5/6/10 5/21/13         37
    ## 3 Meap 3 Data only/sheet: combined data 33 <NA> 12/10/09 5/21/13         41
    ## 4 Meap 3 Data only/sheet: combined data 34 <NA>  11/8/09 5/21/13         42
    ## 5 Meap 3 Data only/sheet: combined data 35 <NA>  9/20/09 5/22/13         44
    ## 6 Meap 3 Data only/sheet: combined data 36 <NA>  5/28/09 5/21/13         48
    ##   age_years age_years_group location percentage_correct ses
    ## 1       2.7               2       KA               0.56   1
    ## 2       3.0               3       KA               0.38   1
    ## 3       3.5               3       KA               0.44   1
    ## 4       3.5               3       KA               0.62   1
    ## 5       3.7               3       KA               0.75   1
    ## 6       4.0               3       KA               0.62   1
    ##                within_id twins  item acc subj_acc quartile bigger_places places
    ## 1  9/19/10 5/22/13 NA KA     n 2 v 8   1     0.56        1             1    1v1
    ## 2   5/6/10 5/21/13 NA KA     n 2 v 8   1     0.38        1             1    1v1
    ## 3 12/10/09 5/21/13 NA KA     n 2 v 8   1     0.44        1             1    1v1
    ## 4  11/8/09 5/21/13 NA KA     n 2 v 8   1     0.62        2             1    1v1
    ## 5  9/20/09 5/22/13 NA KA     n 2 v 8   1     0.75        2             1    1v1
    ## 6  5/28/09 5/21/13 NA KA     n 2 v 8   1     0.62        2             1    1v1
    ##   length_diff one_digit_diff_not_zero transposition inserting_zero
    ## 1           n                       y             n              n
    ## 2           n                       y             n              n
    ## 3           n                       y             n              n
    ## 4           n                       y             n              n
    ## 5           n                       y             n              n
    ## 6           n                       y             n              n
    ##   e3_categories category left_num right_num left_number_range
    ## 1        single        S        2         8            single
    ## 2        single        S        2         8            single
    ## 3        single        S        2         8            single
    ## 4        single        S        2         8            single
    ## 5        single        S        2         8            single
    ## 6        single        S        2         8            single
    ##   right_number_range item_comparison_unordered item_comparison item_type
    ## 1             single             single_single   single_single   Neither
    ## 2             single             single_single   single_single   Neither
    ## 3             single             single_single   single_single   Neither
    ## 4             single             single_single   single_single   Neither
    ## 5             single             single_single   single_single   Neither
    ## 6             single             single_single   single_single   Neither
    ##   completed_task
    ## 1              2
    ## 2              2
    ## 3              2
    ## 4              2
    ## 5              2
    ## 6              2

### Step 2:

Summarize accuracy based on node and edge types for each quartile

``` r
# It takes a corpus analysis approach, summarize over a "item_type, left_number_range, right_number_range"--instead of by person or item--the final results should not be interpreted as accuracy but the percentage of children who corrected answered that type of item.  

data_item_type_summary = data %>%
  group_by(quartile, item_type, item_comparison) %>%
  summarise(item_type_acc = round(mean(acc),2), number_responses = n()) %>% # round to 2 decimals and convert to a data frame to retain the digits format %>%
  filter(number_responses > 9) %>% # only include items that have more than 10 children responses 
  as.data.frame() %>%
  rowwise() %>%
  mutate(left_number_range = strsplit(item_comparison, "_")[[1]][1],
         right_number_range = strsplit(item_comparison, "_")[[1]][2]) %>%
  select(-item_comparison)
```

    ## `summarise()` has grouped output by 'quartile', 'item_type'. You can override using the `.groups` argument.

``` r
# inspect the results
data_item_type_summary
```

    ## # A tibble: 38 x 6
    ## # Rowwise: 
    ##    quartile item_type      item_type_acc number_responses left_number_range
    ##       <int> <chr>                  <dbl>            <int> <chr>            
    ##  1        1 Inserting_zero          0.41              616 decade           
    ##  2        1 Inserting_zero          0.37               59 hundred          
    ##  3        1 Neither                 0.5                88 hundred          
    ##  4        1 Neither                 0.73              101 single           
    ##  5        1 Neither                 0.58              101 single           
    ##  6        1 Neither                 0.44              176 teen             
    ##  7        1 Transposition           0.38              209 hundred          
    ##  8        1 Transposition           0.45               65 thousand         
    ##  9        2 Inserting_zero          0.64              534 decade           
    ## 10        2 Inserting_zero          0.45               71 hundred          
    ## # … with 28 more rows, and 1 more variable: right_number_range <chr>

### Step 3:

Plot the network graphs for all 4 quartiles

``` r
# Construct the network (using all 4 quartiles data, but select each quartile in the loop below for plotting individual quartile graph)
# each link needs to have the left and right number range, the type of comparison, and quartile group number

links = data_item_type_summary %>%
  select(left_number_range, right_number_range, item_type, quartile, item_type_acc) %>%
  mutate(item_type = as.factor(item_type)) # make sure that the comparison type is a factor for plotting edge colors

nodes = unique(data_item_type_summary$left_number_range)

# Loop through each quartile to generate 4 plots
set.seed(99) # set a random seed, so that every time you run the code, it gives you the same network graph

network_stat = NULL # empty variable to save network measures for later analysis

for (i in 1:4) {
  
  # select current quartile data
  temp_links = subset(links, quartile == i) %>%
    filter(item_type_acc > .7) # only plot edges that have at least 70% children who answered correctly
  
  # construct the network
  net = graph_from_data_frame(d = temp_links, vertices = nodes, directed = F)  
  
  # calculate and save network statistics
  cl = cluster_leading_eigen(net)
  
  network_stat_temp = data.frame("quartile" = i, "density" = edge_density(net, loops = TRUE),
                                 "diameter" = diameter(net, directed = F), "degree_mean" = mean(degree(net)), 
                                 "degree" = degree(net), "closeness" = closeness(net), 
                                 "betweenness" = betweenness(net), "modularity" = modularity(cl)) %>%
    rownames_to_column("node_name") # turn the row names into another column called "node_name" (e.g., hundred, single)
  
  network_stat = rbind(network_stat, network_stat_temp)
  
  # set edge colors
  colrs = adjustcolor(c("red", "blue", "green"), alpha = .6)
  
  # set layout
  graph_attr(net, "layout") = layout_with_kk
  
  # open a file
  jpeg(here("Plots",paste0("/networks/network_", task, "_q", i,".jpg")), units="in", width=10, height=9, res=300)
  
  # plot the graph
  plot(net, edge.width = E(net)$item_type_acc * 10 , edge.label = E(net)$item_type_acc, 
       edge.color =  colrs[temp_links$item_type], edge.label.color = "black", 
        edge.label.dist = 2.5, edge.label.cex = 1.5,
        vertex.color = "gray",
        vertex.label.cex = 2, vertex.label.color = "black", vertex.label.dist = 2.5)
  legend("topleft", inset = c(-0.05, 0), legend = levels(temp_links$item_type), fill =colrs)
  
  # close the file
  dev.off()
}
```

    ## Warning in closeness(net): At centrality.c:2784 :closeness centrality is not
    ## well-defined for disconnected graphs

    ## Warning in closeness(net): At centrality.c:2784 :closeness centrality is not
    ## well-defined for disconnected graphs

    ## Warning in closeness(net): At centrality.c:2784 :closeness centrality is not
    ## well-defined for disconnected graphs

``` r
# save network measures for future plotting
write.csv(network_stat, here(paste0("Data/data_", task, "_network_measures.csv")))
```

##### Now, show the N plots

<figure>
<img src="../Plots/networks/network_n_q1.jpg" style="width:70.0%" alt="Quartile 1 network graph for the N task" /><figcaption aria-hidden="true">Quartile 1 network graph for the N task</figcaption>
</figure>

<figure>
<img src="../Plots/networks/network_n_q2.jpg" style="width:70.0%" alt="Quartile 2 network graph for the N task" /><figcaption aria-hidden="true">Quartile 2 network graph for the N task</figcaption>
</figure>

<figure>
<img src="../Plots/networks/network_n_q3.jpg" style="width:70.0%" alt="Quartile 3 network graph for the N task" /><figcaption aria-hidden="true">Quartile 3 network graph for the N task</figcaption>
</figure>

<figure>
<img src="../Plots/networks/network_n_q4.jpg" style="width:70.0%" alt="Quartile 4 network graph for the N task" /><figcaption aria-hidden="true">Quartile 4 network graph for the N task</figcaption>
</figure>

##### Now, show the More plots

<img src="../Plots/networks/network_more_q1.jpg" style="width:70.0%" alt="Quartile 1 network graph for the More task" />
<img src="../Plots/networks/network_more_q2.jpg" style="width:70.0%" alt="Quartile 2 network graph for the More task" />
<img src="../Plots/networks/network_more_q3.jpg" style="width:70.0%" alt="Quartile 3 network graph for the More task" />
<img src="../Plots/networks/network_more_q4.jpg" style="width:70.0%" alt="Quartile 4 network graph for the More task" />


    ## plot network measures individually, don't want this, but keep for now in case co-author is interested

    ```r
    # network_stat_long = network_stat %>%
    #   group_by(quartile) %>%
    #   summarise(density = mean(density), diameter = mean(diameter), degree = mean(degree), modularity = mean(modularity)) %>%
    #   gather(key = "measure", value = "value", c("density":"modularity")) %>%
    #   mutate(quartile = as.factor(quartile),
    #          value = round(value, 3)) %>%
    #   as.data.frame()
    # 
    # ggplot(data = network_stat_long, aes(x = quartile, y = value, group = 1)) +
    #   geom_line() +
    #   facet_wrap(~measure, scales = "free") 
    # 
    # ggsave(here(paste0("Plots/", task, "_network_stats.jpeg")), width = 6, height = 4, dpi = 300)
    # 
    # # individual node analysis
    # # closeness
    # temp_closeness = network_stat %>%
    #   select(node_name, quartile, closeness) %>%
    #   mutate(node_name = factor(node_name, levels = c("single", "teen", "decade", "hundred", "thousand")))
    #   
    # 
    # ggplot(temp_closeness, aes(x = quartile, y = closeness, fill = node_name)) +
    #   geom_bar(stat = "identity", position = position_dodge())
    # 
    # ggsave(here(paste0("Plots/", task, "_network_stats_closeness.jpeg")), width = 6, height = 4, dpi = 300)
    # 
    # # degree
    # temp_degree = network_stat %>%
    #   select(node_name, quartile, degree) %>%
    #   mutate(node_name = factor(node_name, levels = c("single", "teen", "decade", "hundred", "thousand")))
    #   
    # 
    # ggplot(temp_degree, aes(x = quartile, y = degree, fill = node_name)) +
    #   geom_bar(stat = "identity", position = position_dodge())
    # 
    # ggsave(here(paste0("Plots/", task, "_network_stats_degree.jpeg")), width = 6, height = 4, dpi = 300)

## Network stats: this approach is not appliable here, but keep for now in case co-author wants to see

``` r
# data_wide_raw = read.csv(here("Data/data_n_raw_wide.csv"),header=TRUE,sep=",", check.names = "FALSE") %>%
#   mutate(quartile = ntile(acc, 4)) %>%
#   select(id, quartile, acc, c("2 v 8": "189 v 198"))
# 
# data_wide = data_wide_raw %>%
#   gather(key = "item", value = "acc", c("2 v 8": "189 v 198"))
# 
# data_item_category = read.csv(here("Data/data_n_item.csv"),header=TRUE,sep=",", check.names = "FALSE")
# 
# data_wide_item_type = data_wide %>%
#   left_join(data_item_category, by = "item") %>%
#   select(id, quartile, acc, item_comparison) %>%
#   group_by(id, quartile, item_comparison) %>%
#   summarise(accuracy = mean(acc, na.rm = TRUE)) %>%
#   spread(item_comparison, accuracy) %>%
#   ungroup()
# 
# network_data_temp = data_wide_item_type %>%
#   filter(quartile == 2) %>%
#   select(-id, -quartile)
# 
# network = estimateNetwork(network_data_temp, default = "EBICglasso")
# plot(network, layout = "spring")
# 
# 
# network_data_temp = data_wide_raw %>%
#   filter(quartile == 1) %>%
#   select(-id, -quartile, -acc)
# 
# network = estimateNetwork(network_data_temp, default = "pcor", missing = "pairwise")
# plot(network, layout = "spring")
# 
# 
# library("qgraph")
# i = 2
# # select current quartile data
#   temp_links = subset(links, quartile == i) %>%
#     filter(item_type_acc > .7) # only plot edges that have at least 70% children who answered correctly
#   net = graph_from_data_frame(d = temp_links, vertices = nodes, directed = F)
# 
#   # convert to qgraph
#   a_matrix = as.matrix(as_adjacency_matrix(net, attr = "item_type_acc", sparse = F)) # don't forget to convert to matrix to be used with qgraph
#   # plot using qgraph instead
#   qgraph(a_matrix)
#   centralityPlot(a_matrix, include =c("Betweenness","Closeness","Strength"))
# 
# library("bootnet")
#   # stability of centrality indices
#   central_stability = bootnet(a_matrix, nBoots = 1000, type = "case", statistics = c("edge", "strength", "closeness", "betweenness"))
# 
#   
#   q1 = estimateNetwork(item_category_count, default = "EBICglasso")
# plot(q1, layout = "spring", labels = colnames(q1))
```
