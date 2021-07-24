Analysis 1: Overall descriptive results
================

## Overview

This code provides an overall view of children’s performance in the two
tasks.

## Load libraries

``` r
library("here")
library("tidyverse")
library("afex")
library("lme4")
library("janitor")

setwd(here())
```

## Load data

Compute subject level data for N and More separately, and combined

``` r
data_n_trial_level_raw = read.csv(here("Data/data_n_long.csv"), check.names = FALSE) 

data_n_trial_level = data_n_trial_level_raw %>%
  select(id, sex, ses, age_months, location, acc, completed_task, within_id, age_years_group) %>%
  mutate(task = "N")

data_more_trial_level_raw = read.csv(here("Data/data_more_long.csv"), check.names = FALSE) 

data_more_trial_level = data_more_trial_level_raw %>%
  select(id, sex, ses, age_months, location, acc, completed_task, within_id, age_years_group) %>%
  mutate(task = "More")

# Getting subj level data
data_n_subj_level = data_n_trial_level %>%
  group_by(id, age_months, sex, ses, location, task, completed_task, within_id, age_years_group) %>%
  summarise(acc_subj = mean(acc)) %>%
  mutate(sex = as.factor(sex),
         ses = as.factor(ses),
         id = as.factor(id),
         location = as.factor(location)) %>%
  data.frame()

data_more_subj_level = data_more_trial_level %>%
  group_by(id, age_months, sex, ses, location, task, completed_task, within_id, age_years_group) %>%
  summarise(acc_subj = mean(acc)) %>%
  mutate(sex = as.factor(sex),
         ses = as.factor(ses),
         id = as.factor(id),
         location = as.factor(location)) %>%
  data.frame()

# if doing combined data set
data_combined_subj_level = rbind(data_n_subj_level, data_more_subj_level)
```

## Demographic information

``` r
# Number of participants, complicated ---------------------
# total number of completed, not total participants because some children received both tasks
total_completed_tasks = nrow(data_combined_subj_level)
total_completed_tasks
```

    ## [1] 867

``` r
# total number of trials/observations
total_num_trials = nrow(data_n_trial_level) + nrow(data_more_trial_level)
total_num_trials
```

    ## [1] 13039

``` r
# number of children who received both tasks
data_within_data_subj_level = read.csv(here("Data/data_within_data.csv"), check.names = FALSE) %>%
  group_by(within_id, age_years_group, sex, age_months) %>%
  summarise(subj_acc = mean(acc)) %>%
  ungroup()

total_both_tasks = nrow(data_within_data_subj_level)
total_both_tasks
```

    ## [1] 323

``` r
# number of children who only received one task
total_one_task = total_completed_tasks - nrow(data_within_data_subj_level) * 2
total_one_task
```

    ## [1] 221

``` r
# total number of participants 
total_participants = total_both_tasks + total_one_task
total_participants
```

    ## [1] 544

``` r
# Participants' demographic info ------------------
# identify children who only received one task
subj_one_task = data_combined_subj_level %>%
  filter(completed_task == 1) %>%
  select(within_id, age_months, sex, location, age_years_group)

subj_two_tasks = data_combined_subj_level %>%
  filter(completed_task == 2) %>%
  group_by(within_id, age_months, sex, location, age_years_group) %>%
  summarise(n = n()) %>%
  select(-n)

subj_all = rbind(subj_one_task, subj_two_tasks)

subj_all %>%
  summarise(mean = mean(age_months)/12, median = median(age_months)/12, min = min(age_months)/12, max = max(age_months)/12)
```

    ##       mean   median      min      max
    ## 1 5.021142 4.994518 2.584978 7.042215

``` r
subj_all %>%
  tabyl(sex)
```

    ##   sex   n    percent valid_percent
    ##     M 255 0.46875000     0.5079681
    ##     F 247 0.45404412     0.4920319
    ##  <NA>  42 0.07720588            NA

``` r
subj_all %>%
  tabyl(age_years_group)
```

    ##  age_years_group   n     percent
    ##                2  12 0.022058824
    ##                3  69 0.126838235
    ##                4 191 0.351102941
    ##                5 155 0.284926471
    ##                6 113 0.207720588
    ##                7   4 0.007352941

``` r
subj_all %>%
  mutate(tested_in_school = ifelse(location == "Lab", "y", "n")) %>%
  tabyl(tested_in_school)
```

    ##  tested_in_school   n   percent
    ##                 n 226 0.4154412
    ##                 y 318 0.5845588

## model: acc\_subj \~ age + sex + location

``` r
model = lm(acc_subj ~ age_months + task + sex + location, data = data_combined_subj_level)
summary(model)
```

    ## 
    ## Call:
    ## lm(formula = acc_subj ~ age_months + task + sex + location, data = data_combined_subj_level)
    ## 
    ## Residuals:
    ##      Min       1Q   Median       3Q      Max 
    ## -0.63098 -0.10392  0.01705  0.10521  0.38006 
    ## 
    ## Coefficients:
    ##                        Estimate Std. Error t value Pr(>|t|)    
    ## (Intercept)            0.117493   0.045015   2.610  0.00923 ** 
    ## age_months             0.009580   0.000559  17.137  < 2e-16 ***
    ## taskN                  0.019781   0.012570   1.574  0.11597    
    ## sexF                   0.004189   0.012013   0.349  0.72741    
    ## locationCV             0.064575   0.047803   1.351  0.17713    
    ## locationHarmony       -0.089068   0.043485  -2.048  0.04087 *  
    ## locationKA            -0.022630   0.047148  -0.480  0.63137    
    ## locationKC             0.029331   0.041516   0.707  0.48008    
    ## locationLab            0.015960   0.030593   0.522  0.60203    
    ## locationPLE            0.007941   0.045534   0.174  0.86160    
    ## locationPrep School    0.025792   0.038358   0.672  0.50153    
    ## locationUM            -0.040638   0.046285  -0.878  0.38022    
    ## locationJack & Jill   -0.125873   0.053794  -2.340  0.01954 *  
    ## locationJenny's Place -0.182763   0.074896  -2.440  0.01490 *  
    ## locationBDLC           0.171787   0.060859   2.823  0.00488 ** 
    ## locationTA             0.028855   0.052261   0.552  0.58102    
    ## ---
    ## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
    ## 
    ## Residual standard error: 0.167 on 775 degrees of freedom
    ##   (76 observations deleted due to missingness)
    ## Multiple R-squared:  0.3491, Adjusted R-squared:  0.3365 
    ## F-statistic: 27.71 on 15 and 775 DF,  p-value: < 2.2e-16

``` r
anova(model)
```

    ## Analysis of Variance Table
    ## 
    ## Response: acc_subj
    ##             Df  Sum Sq Mean Sq  F value    Pr(>F)    
    ## age_months   1 10.2176 10.2176 366.5795 < 2.2e-16 ***
    ## task         1  0.1639  0.1639   5.8803   0.01554 *  
    ## sex          1  0.0220  0.0220   0.7890   0.37467    
    ## location    12  1.1823  0.0985   3.5348 3.945e-05 ***
    ## Residuals  775 21.6013  0.0279                       
    ## ---
    ## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1

## Plots

``` r
fontsize = 13

# scatter plot: Accuracy is linearly related to age, but no sex difference
ggplot(data = data_combined_subj_level, aes(x = age_months, y = acc_subj, color = sex)) +
  geom_point(cex = 0.8) +
  geom_smooth(data = subset(data_combined_subj_level, sex == "F"), method="lm", na.rm = F, size = 1, color = "lightcoral", fill = "lightcoral") +
  geom_smooth(data = subset(data_combined_subj_level, sex == "M"), method="lm", na.rm = F, size = 1, color = "cyan3", fill = "cyan3") +
  theme_bw() + theme(panel.border = element_blank(), panel.grid.major = element_blank(),
                     panel.grid.minor = element_blank(), axis.line = element_line(colour = "black")) +
  #ggtitle("The Which-N task") +
  ylab("Proportion of correct trials") + 
  xlab("Age (months)") +
  labs(color = "Sex") +
  theme(plot.title = element_text(size = fontsize, hjust = 0.5), text=element_text(size=fontsize)) +
  scale_color_manual(values = c("lightcoral", "cyan3","grey72"))
```

![](Analysis-1_Overall-descriptive-results_files/figure-gfm/unnamed-chunk-5-1.png)<!-- -->

``` r
fig = here("Plots", "descriptive", "scatterplot_acc_by_age_gender.jpeg")
ggsave(fig, height = 4, width = 6, dpi = 300)

# school plot: box plot
ggplot(data_combined_subj_level, aes(x = reorder(location, acc_subj), y = acc_subj)) +
  geom_boxplot() +
  #geom_jitter() +
  #ylim(c(0,1.05)) +
  coord_flip() +
  ylab("Proportion of correct trials") +
  xlab("Schools") +
  geom_hline(yintercept=0.5, linetype="dashed", color = "lightcoral", size = 0.8) + 
  theme_bw() + theme(panel.border = element_blank(), panel.grid.major = element_blank(),
                     panel.grid.minor = element_blank(), axis.line = element_line(colour = "black")) + 
  theme(plot.title = element_text(size = fontsize, hjust = 0.5), text=element_text(size=fontsize), legend.position = "none", axis.text.y = element_text(size = 0))
```

![](Analysis-1_Overall-descriptive-results_files/figure-gfm/unnamed-chunk-5-2.png)<!-- -->

``` r
fig = here("Plots", "descriptive", "acc_by_school.jpeg")
ggsave(fig, height = 4, width = 4.5, dpi = 300)

# task difference plot, N develops earlier than more: box plot with jitters?
ggplot(data_combined_subj_level, aes(x = acc_subj, fill = task)) +
  geom_density(aes(y = ..scaled..), adjust = 1.5, alpha = 0.4) +
  xlab("Accuracy") +
  ylab("Density") +
  scale_fill_discrete(name = "Task", labels = c("More", "N")) +
  theme_classic(base_size = 13)
```

![](Analysis-1_Overall-descriptive-results_files/figure-gfm/unnamed-chunk-5-3.png)<!-- -->

``` r
fig = here("Plots", "descriptive", "acc_by_task.jpeg")
ggsave(fig, height = 4, width = 6, dpi = 300)
```

## The effect of some features (i.e., length\_diff, transposition)

``` r
data_combined_trial_level = rbind(data_n_trial_level_raw, data_more_trial_level_raw)
```
