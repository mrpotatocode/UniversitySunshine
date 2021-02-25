The Wage Gap Persists: How pay imbalances by gender unequally reward male professors at the University of Toronto
================
Thomas Rosenthal

March 1, 2021

## Introduction

Anintro…

## Data

brief..

### Dataset

Ontario makes available public sector salary disclosure from 1996 to
2019 (or most current year) avialable through the Open Data Ontario
portal, or for each year as .csv file from
[ontario.ca/page/public-sector-salary-disclosure](https://www.ontario.ca/page/public-sector-salary-disclosure).

Using

### Gender Assignment Process

### Ethical Considerations

Load Libraries

``` r
library(tidyverse)
library(data.table)
library(here)
library(rstanarm)
library(gender)
library(ggplot2)
library(bayesplot)
library(kableExtra)
```

Load Data

``` r
#function for fread (data.table)
read_plus <- function(flnm) {
    read_csv(flnm) %>% 
        mutate(filename = flnm) %>% #adds the filename
        mutate(`Salary Paid` = as.character(`Salary Paid`)) %>% #allows for $000,000 notation, some files were int
        mutate(`Taxable Benefits` = as.character(`Taxable Benefits`)) %>% 
        rename_with(str_to_title) #fixes title case vs lower case issue for colnames between filenames
  }

datafolder = paste0(here(),'/data')

raw_data = 
    list.files(path = datafolder,
      pattern = "*.csv",
      full.names = T) %>% 
    map_df(~read_plus(.))
```

``` r
#fix the rows that were skipped
#set the year as 2016, instead of NA
raw_data <- raw_data %>% mutate(`Calendar Year` = if_else(is.na(`Calendar Year`), 2016, `Calendar Year`))
```

``` r
raw_data %>% mutate(SalaryPaid = parse_number(`Salary Paid`), .after = "Salary Paid", .keep="unused")  %>%
    map_df(function(x) sum(is.na(x))) %>%
    gather(feature, num_nulls) %>%
    print(n = 100)
```

``` r
kable(head(raw_data))
```

<table>

<thead>

<tr>

<th style="text-align:left;">

Sector

</th>

<th style="text-align:left;">

Last Name

</th>

<th style="text-align:left;">

First Name

</th>

<th style="text-align:left;">

Salary Paid

</th>

<th style="text-align:left;">

Taxable Benefits

</th>

<th style="text-align:left;">

Employer

</th>

<th style="text-align:left;">

Job Title

</th>

<th style="text-align:right;">

Calendar Year

</th>

<th style="text-align:left;">

Filename

</th>

</tr>

</thead>

<tbody>

<tr>

<td style="text-align:left;">

Government of Ontario - Ministries

</td>

<td style="text-align:left;">

Almond

</td>

<td style="text-align:left;">

Margot

</td>

<td style="text-align:left;">

$123,153.63

</td>

<td style="text-align:left;">

$204.96

</td>

<td style="text-align:left;">

Aboriginal Affairs

</td>

<td style="text-align:left;">

Director, Corporate Management / Directrice, gestion ministérielle

</td>

<td style="text-align:right;">

2014

</td>

<td style="text-align:left;">

/Users/thomas/Documents/Second Year/Winter Term/Ethics in
DS/Sunshine\_List/data/2014-pssd-full-compendium-utf8-en.csv

</td>

</tr>

<tr>

<td style="text-align:left;">

Government of Ontario - Ministries

</td>

<td style="text-align:left;">

Aniol

</td>

<td style="text-align:left;">

Richard

</td>

<td style="text-align:left;">

$102,860.59

</td>

<td style="text-align:left;">

$173.98

</td>

<td style="text-align:left;">

Aboriginal Affairs

</td>

<td style="text-align:left;">

Senior Negotiator / Négociateur principal

</td>

<td style="text-align:right;">

2014

</td>

<td style="text-align:left;">

/Users/thomas/Documents/Second Year/Winter Term/Ethics in
DS/Sunshine\_List/data/2014-pssd-full-compendium-utf8-en.csv

</td>

</tr>

<tr>

<td style="text-align:left;">

Government of Ontario - Ministries

</td>

<td style="text-align:left;">

Bennett

</td>

<td style="text-align:left;">

Phyllis

</td>

<td style="text-align:left;">

$114,399.48

</td>

<td style="text-align:left;">

$189.54

</td>

<td style="text-align:left;">

Aboriginal Affairs

</td>

<td style="text-align:left;">

Manager, Issues Management and Media Relations / Chef, gestions des
questions d’intérêt et relations avec les m?dias

</td>

<td style="text-align:right;">

2014

</td>

<td style="text-align:left;">

/Users/thomas/Documents/Second Year/Winter Term/Ethics in
DS/Sunshine\_List/data/2014-pssd-full-compendium-utf8-en.csv

</td>

</tr>

<tr>

<td style="text-align:left;">

Government of Ontario - Ministries

</td>

<td style="text-align:left;">

Bird

</td>

<td style="text-align:left;">

Anne

</td>

<td style="text-align:left;">

$103,033.91

</td>

<td style="text-align:left;">

$176.37

</td>

<td style="text-align:left;">

Aboriginal Affairs

</td>

<td style="text-align:left;">

Team Lead / Chef d’équipe

</td>

<td style="text-align:right;">

2014

</td>

<td style="text-align:left;">

/Users/thomas/Documents/Second Year/Winter Term/Ethics in
DS/Sunshine\_List/data/2014-pssd-full-compendium-utf8-en.csv

</td>

</tr>

<tr>

<td style="text-align:left;">

Government of Ontario - Ministries

</td>

<td style="text-align:left;">

Bliss

</td>

<td style="text-align:left;">

Rose

</td>

<td style="text-align:left;">

$102,045.84

</td>

<td style="text-align:left;">

$175.66

</td>

<td style="text-align:left;">

Aboriginal Affairs

</td>

<td style="text-align:left;">

Manager, Performance Measures and Data / Chef, mesures et données de
rendement

</td>

<td style="text-align:right;">

2014

</td>

<td style="text-align:left;">

/Users/thomas/Documents/Second Year/Winter Term/Ethics in
DS/Sunshine\_List/data/2014-pssd-full-compendium-utf8-en.csv

</td>

</tr>

<tr>

<td style="text-align:left;">

Government of Ontario - Ministries

</td>

<td style="text-align:left;">

Carr

</td>

<td style="text-align:left;">

Douglas

</td>

<td style="text-align:left;">

$185,661.92

</td>

<td style="text-align:left;">

$298.74

</td>

<td style="text-align:left;">

Aboriginal Affairs

</td>

<td style="text-align:left;">

Assistant Deputy Minister, Negotiations and Reconciliation /
Sous-ministre adjoint, négociations et réconciliation

</td>

<td style="text-align:right;">

2014

</td>

<td style="text-align:left;">

/Users/thomas/Documents/Second Year/Winter Term/Ethics in
DS/Sunshine\_List/data/2014-pssd-full-compendium-utf8-en.csv

</td>

</tr>

</tbody>

</table>

``` r
UofT <- raw_data %>% filter(Employer %in% c('University of Toronto','University Of Toronto')) %>% 
                              rename(FirstName = `First Name`)
```

``` r
first_names <- UofT %>% distinct(FirstName)
nrow(first_names)

gendered_names <- gender(first_names$FirstName)
```

``` r
UofT_gen <- left_join(UofT, gendered_names, by = c("FirstName" = "name"))
UofT_gen <- UofT_gen %>% mutate(ID = row_number())
nrow(UofT_gen %>% filter(is.na(gender)) %>% distinct(FirstName))
```

``` r
adj <- UofT %>%  
  mutate(FirstNameadj = str_replace(FirstName, "\\s", "|")) %>% 
  separate(FirstNameadj, into = c("first","second"), sep = "\\|")

gendered_names <- gender(adj$first)
gendered_names_second <- gender(adj$second)
```

``` r
UofT_gen <- left_join(adj, gendered_names, by = c("first" = "name")) %>% distinct()
UofT_gen <- UofT_gen %>% mutate(ID = row_number())

UofT_gen_x <- left_join(adj, gendered_names_second, by = c("second" = "name")) %>% distinct()
UofT_gen <- UofT_gen %>% 
    mutate(gender = coalesce(gender,UofT_gen_x$gender))

nrow(UofT_gen %>% filter(is.na(gender)) %>% distinct(FirstName))
```

``` r
adj <-
  UofT %>% 
  mutate(FirstNameadj = str_replace(FirstName, "\\s", "|")) %>% 
  separate(FirstNameadj, into = c("first","second"), sep = "\\|") %>% 
  mutate(paren = str_extract(second, "\\(([^()]*)\\)")) %>% 
  mutate(paren = str_replace(paren,"\\(",'')) %>% 
  mutate(paren = str_replace(paren,"\\)",''))

gendered_names <- gender(adj$paren)
```

``` r
UofT_gen_y <- left_join(adj, gendered_names, by = c("paren" = "name")) %>% distinct()
UofT_gen <- UofT_gen %>% 
    mutate(gender = coalesce(gender,UofT_gen_y$gender))

nrow(UofT_gen %>% filter(is.na(gender)) %>% distinct(FirstName))
```

``` r
adj <- UofT %>%  
  mutate(FirstNameadj = str_replace(FirstName, "-", "|")) %>% 
  separate(FirstNameadj, into = c("first","second"), sep = "\\|")

gendered_names <- gender(adj$first)
```

``` r
UofT_gen_z <- left_join(adj, gendered_names, by = c("first" = "name")) %>% distinct()
UofT_gen <- UofT_gen %>% 
    mutate(gender = coalesce(gender,UofT_gen_z$gender))

nrow(UofT_gen %>% filter(is.na(gender)) %>% distinct(FirstName))
```

``` r
UofT_gen <- UofT_gen %>% mutate(SalaryPaid = parse_number(`Salary Paid`), .after = "Salary Paid", .keep="unused")

UofT_gen <- UofT_gen %>% mutate(CalendarYear =  as.factor(`Calendar Year`), .after = "Calendar Year", .keep="unused")
```

``` r
UofT_gen %>% group_by(gender,) %>% 
            summarize(gender_n = n(), 
                      mean_sal = mean(SalaryPaid),
                      max_sal = max(SalaryPaid))
```

``` r
`(ᵔᴥᵔ)` <- ggplot(UofT_gen, aes(x=CalendarYear, y=SalaryPaid, fill=gender)) + 
    geom_boxplot(outlier.shape = NA)

ylim1 = boxplot.stats(UofT_gen$SalaryPaid)$stats[c(1, 5)]

`ʕ•ᴥ•ʔ` = `(ᵔᴥᵔ)` + coord_cartesian(ylim = ylim1*1.05)
`ʕ•ᴥ•ʔ` 
```

``` r
UofT_prof <- UofT_gen %>% mutate(JobTitleAdj = str_replace(str_replace(`Job Title`, ", ", "|"), " of ","|")) %>% 
            separate(JobTitleAdj, into = c("outer","inner"), sep = "\\|") %>% filter(outer %like% 'rofessor') %>% 
            filter(CalendarYear ==2019) %>% group_by(outer) %>% 
            filter(n() >= 5)


UofT_prof$outer2 <- str_wrap(UofT_prof$outer, width = 15)
`|>.<|` <- ggplot(UofT_prof , aes(x=outer2, y=SalaryPaid, fill=gender)) + 
    geom_boxplot()

ylim1 = boxplot.stats(UofT_prof$SalaryPaid)$stats[c(1, 5)]

`|◉_◉|` = `|>.<|` + coord_cartesian(ylim = ylim1*1.05)
`|◉_◉|` + coord_flip()
```

``` r
UofT_dean <- UofT_gen %>% mutate(JobTitleAdj = str_replace(str_replace(`Job Title`, ", ", "|"), " of ","|")) %>% 
            separate(JobTitleAdj, into = c("outer","inner"), sep = "\\|") %>% filter(outer %like% 'Dean') %>% 
            filter(CalendarYear ==2019) %>% group_by(outer) %>% 
            filter(n() >= 5)

UofT_dean$outer2 <- str_wrap(UofT_dean$outer, width = 15)
`|-.-|` <- ggplot(UofT_dean, aes(x=outer2, y=SalaryPaid, fill=gender)) + 
    geom_boxplot()

ylim1 = boxplot.stats(UofT_dean$SalaryPaid)$stats[c(1, 5)]

`|⚆ _ ⚆|` = `|-.-|` + coord_cartesian(ylim = ylim1*1.05)
`|⚆ _ ⚆|` + coord_flip()
```

``` r
UofT_ischool <- UofT_gen %>% filter(`Job Title` %like% 'of Information') %>% 
            filter(CalendarYear ==2019) %>% filter(`Last Name` %in% c('Eskenazi','Fiege') == FALSE) %>% 
            mutate(JobTitleAdj = str_replace(str_replace(`Job Title`, ", ", "|"), " of ","|")) %>% 
            separate(JobTitleAdj, into = c("outer","inner"), sep = "\\|")

UofT_ischool$outer2 <- str_wrap(UofT_ischool$outer, width = 15)
`(☞ﾟヮﾟ)☞` <- ggplot(UofT_ischool ,aes(x=outer2, y=SalaryPaid, fill=gender)) + 
    geom_boxplot()
`(☞ﾟヮﾟ)☞`
```

``` r
`｡◕‿◕｡` <- ggplot(data = UofT_gen, aes(x=SalaryPaid)) + geom_density(aes(fill=gender), alpha = 0.4)
`｡◕‿◕｡` <- `｡◕‿◕｡` + facet_wrap( ~ gender)
`｡◕‿◕｡` + theme(axis.text.x = element_text(angle = 90, vjust = 0.5, hjust=1))
```

``` r
ggplot(UofT_gen, aes(x = SalaryPaid, y = ..density.., fill = gender)) +
  geom_density()
```

``` r
UofT_bi <- UofT_gen %>% fastDummies::dummy_cols(select_columns = "gender", ignore_na = TRUE) %>% 
  select(`Last Name`, FirstName, SalaryPaid, `Taxable Benefits`, `Job Title`, CalendarYear, gender_female) %>% 
  mutate(SalaryPaid100 = SalaryPaid/100) %>% select(-SalaryPaid) %>% 
  mutate(CalendarYear = as.integer(as.character(CalendarYear)))

#UofT_bi <- UofT_gen %>% mutate(gender = fastDummies::dummy_cols(select_columns = gender))
  
#UofT_bi_min <- UofT_bi %>% select(CalendarYear,gender_female, SalaryPaid100)



t_prior <- student_t(df = 7, location = 0, scale = 2.5)
fit1 <- stan_glm(gender_female ~ SalaryPaid100, data = UofT_bi,
                 family = binomial(link = "logit"),
                 prior = t_prior, prior_intercept = t_prior,
                 cores = 6, seed = 12345)

summary(fit1)

round(posterior_interval(fit1, prob = 0.5), 2)
```

``` r
theme_set(bayesplot::theme_default())
```

``` r
# Predicted probability as a function of x
pr_switch <- function(x, ests) plogis(ests[1] + ests[2] * x)
# A function to slightly jitter the binary data
jitt <- function(...) {
  geom_point(aes_string(...), position = position_jitter(height = 0.05, width = 0.1),
             size = 2, shape = 21, stroke = 0.2)
}
ggplot(UofT_bi, aes(x = SalaryPaid100, y = gender_female, color = gender_female)) +
  scale_y_continuous(breaks = c(0, 0.5, 1)) +
  jitt(x="SalaryPaid100") +
  stat_function(fun = pr_switch, args = list(ests = coef(fit1)),
                size = 2, color = "gray35")
```

``` r
fit2 <- update(fit1, formula = gender_female ~ SalaryPaid100 + CalendarYear)

summary(fit2)

(coef_fit2 <- round(coef(fit2), 3))
```

``` r
pr_switch2 <- function(x, y, ests) plogis(ests[1] + ests[2] * x + ests[3] * y)
grid <- expand.grid(SalaryPaid100 = seq(0, 10000, length.out = 101),
                    CalendarYear = seq(2012, 2019)) #seq(2016, 2019))
grid$prob <- with(grid, pr_switch2(SalaryPaid100, CalendarYear, coef(fit2)))
ggplot(grid, aes(x = CalendarYear, y = SalaryPaid100)) +
  geom_tile(aes(fill = prob)) +
  geom_jitter(data = UofT_bi, aes(color = factor(gender_female)), size = 2, alpha = 0.85) +
  scale_fill_gradient() +
  scale_color_manual("female", values = c("white", "black"), labels = c("No", "Yes"))
```

``` r
# Quantiles
q_ars <- quantile(UofT_bi$SalaryPaid100, seq(0, 1, 0.25))
q_dist <- quantile(UofT_bi$CalendarYear, seq(0, 1, 0.25))
base <- ggplot(UofT_bi) + xlim(c(0, NA)) +
  scale_y_continuous(breaks = c(0, 0.5, 1))
vary_arsenic <- base + jitt(x="CalendarYear", y="gender_female", color="gender_female") + xlim(2012,2019)#xlim(2016,2019)
vary_dist <- base + jitt(x="SalaryPaid100", y="gender_female", color="gender_female")
for (i in 1:5) {
  vary_dist <-
    vary_dist + stat_function(fun = pr_switch2, color = "gray35",
                              args = list(ests = coef(fit2), y = q_dist[i])
                              )
  vary_arsenic <-
    vary_arsenic + stat_function(fun = pr_switch2, color = "gray35",
                                 args = list(ests = coef(fit2), x = q_ars[i])
                                 )
}
bayesplot_grid(vary_dist, vary_arsenic,
               grid_args = list(ncol = 2))
```

``` r
UofT_prof_bi <- UofT_prof %>% rename("JobTitleSimple" = "outer") %>%
    fastDummies::dummy_cols(select_columns = "gender", ignore_na = TRUE) %>%
    select(`Last Name`, FirstName, SalaryPaid, `Taxable Benefits`, JobTitleSimple, CalendarYear, gender_female) %>% 
    mutate(SalaryPaid100 = SalaryPaid/100) %>% select(-SalaryPaid) %>% 
    mutate(CalendarYear = as.integer(as.character(CalendarYear))) 


t_prior <- student_t(df = 7, location = 0, scale = 2.5)
fit3 <- stan_glm(gender_female ~ SalaryPaid100, data = UofT_prof_bi,
                 family = binomial(link = "logit"),
                 prior = t_prior, prior_intercept = t_prior,
                 cores = 6, seed = 12345)

summary(fit3)

round(posterior_interval(fit3, prob = 0.5), 2)
```

``` r
# Predicted probability as a function of x
pr_switch <- function(x, ests) plogis(ests[1] + ests[2] * x)
# A function to slightly jitter the binary data
jitt <- function(...) {
  geom_point(aes_string(...), position = position_jitter(height = 0.05, width = 0.1),
             size = 2, shape = 21, stroke = 0.2)
}
ggplot(UofT_prof_bi, aes(x = SalaryPaid100, y = gender_female, color = gender_female)) +
  scale_y_continuous(breaks = c(0, 0.5, 1)) +
  jitt(x="SalaryPaid100") +
  stat_function(fun = pr_switch, args = list(ests = coef(fit3)),
                size = 2, color = "gray35")
```

``` r
fit4 <- update(fit3, formula = gender_female ~ SalaryPaid100 + JobTitleSimple)

summary(fit4)
(coef_fit4 <- round(coef(fit4), 3))
```

``` r
labelsp = c('Assistant Professor','Associate Professor','Associate Professor and Chair',
            'Professor','Professor and Associate Chair','Professor and Chair',
            'Professor and Dean','Professor and Director','Professor and Program Coordinator','University Professor')


pr_switch2 <- function(x, y, ests) plogis(ests[1] + ests[2] * x + ests[3] * y)
grid <- expand.grid(SalaryPaid100 = seq(0, 5000, length.out = 101),
                    JobTitleSimple = seq(0, 10, length.out = 10))
grid$prob <- with(grid, pr_switch2(SalaryPaid100, JobTitleSimple, coef(fit4)))
ggplot(grid, aes(y = SalaryPaid100, x = as.integer(as.factor(JobTitleSimple)))) +
  geom_tile(aes(fill = prob)) +
  scale_fill_gradient() +
  scale_x_continuous(breaks = c(1,2,3,4,5,6,7,8,9,10), labels = labelsp) +
  geom_jitter(data = UofT_prof_bi, aes(color = factor(gender_female)), size = 2, alpha = 0.85) +
  scale_color_manual("female", values = c("white", "black"), labels = c("No", "Yes")) 
```

``` r
UofT_cont <- UofT_gen %>%
    select(`Last Name`, FirstName, SalaryPaid, `Taxable Benefits`, `Job Title`, CalendarYear, gender) %>% 
    mutate(SalaryPaid100 = SalaryPaid/100) %>% select(-SalaryPaid) %>% 
    mutate(CalendarYear = as.integer(as.character(CalendarYear))) 


post1 <- stan_glm(SalaryPaid100 ~ gender, data = UofT_cont,
                  #family = gaussian(link = "identity"),
                  cores = 6, seed = 12345)

summary(post1)
```

``` r
base <- ggplot(UofT_cont, aes(x = gender, y = SalaryPaid100)) +
  geom_point(size = 1, position = position_jitter(height = 0.05, width = 0.1)) +ylim(1000,6000)

base + geom_abline(intercept = coef(post1)[1], slope = coef(post1)[2],
                   color = "skyblue4", size = 1)
```

``` r
draws <- as.data.frame(post1)
colnames(draws)[1:2] <- c("a", "b")

base +
  geom_abline(data = draws, aes(intercept = a, slope = b),
              color = "skyblue", size = 0.2, alpha = 0.25) +
  geom_abline(intercept = coef(post1)[1], slope = coef(post1)[2],
              color = "skyblue4", size = 1)
```

``` r
post2 <- update(post1, formula = . ~ gender + CalendarYear)
summary(post2)
```

``` r
reg0 <- function(x, ests) cbind(1, 0, x) %*% ests
reg1 <- function(x, ests) cbind(1, 1, x) %*% ests

args <- list(ests = coef(post2))
lgnd <- guide_legend(title = NULL)
base2 <- ggplot(UofT_cont, aes(x = CalendarYear, fill = gender)) + 
  geom_jitter(aes(y = SalaryPaid100), shape = 21, stroke = .2, size = 1) +
  guides(color = lgnd, fill = lgnd) +
  theme(legend.position = "right") +
  ylim(1000,2000)
base2 +
  stat_function(fun = reg0, args = args, aes(color = "female"), size = 1.5) +
  stat_function(fun = reg1, args = args, aes(color = "male"), size = 1.5) 
```

``` r
Salary_SQ <- seq(from = 1000, to = 6000, by = 100)
y_female <- posterior_predict(post2, newdata = data.frame(gender = "female", SalaryPaid100 = Salary_SQ,CalendarYear = 2020))
y_male <- posterior_predict(post2, newdata = data.frame(gender = "male", SalaryPaid100 = Salary_SQ,CalendarYear = 2020))
dim(y_female)
```

``` r
par(mfrow = c(1:2), mar = c(5,4,2,1))
boxplot(y_female, axes = FALSE, outline = FALSE, ylim = c(0,3500),
        xlab = "Salary", ylab = "Predicted 2020 Salary", main = "female")
axis(1, at = 1:ncol(y_female), labels = Salary_SQ, las = 3)
axis(2, las = 1)
boxplot(y_male, outline = FALSE, col = "red", axes = FALSE, ylim = c(0,3500),
        xlab = "Salary", ylab = NULL, main = "male")
axis(1, at = 1:ncol(y_female), labels = Salary_SQ, las = 3)
```

``` r
UofT_prof_cont <- UofT_prof %>% rename("JobTitleSimple" = "outer") %>%
    select(`Last Name`, FirstName, SalaryPaid, `Taxable Benefits`, JobTitleSimple, CalendarYear, gender) %>% 
    mutate(SalaryPaid100 = SalaryPaid/100) %>% select(-SalaryPaid) %>% 
    mutate(CalendarYear = as.integer(as.character(CalendarYear))) 

post3 <- stan_glm(SalaryPaid100 ~ gender, data = UofT_prof_cont,
                  #family = gaussian(link = "identity"),
                  cores = 6, seed = 12345)

summary(post3)
```

``` r
draws <- as.data.frame(post3)
colnames(draws)[1:2] <- c("a", "b")

base +
  geom_abline(data = draws, aes(intercept = a, slope = b),
              color = "skyblue", size = 0.2, alpha = 0.25) +
  geom_abline(intercept = coef(post3)[1], slope = coef(post3)[2],
              color = "skyblue4", size = 1)
```

``` r
post4 <- update(post3, formula = . ~ gender + JobTitleSimple)
summary(post4)
```

``` r
reg0 <- function(x, ests) cbind(1, 0, x) %*% ests
reg1 <- function(x, ests) cbind(1, 1, x) %*% ests

args <- list(ests = coef(post4))
lgnd <- guide_legend(title = NULL)
base2 <- ggplot(UofT_prof_cont, aes(x = JobTitleSimple, fill = gender)) + 
  geom_jitter(aes(y = SalaryPaid100), shape = 21, stroke = .2, size = 1) +
  guides(color = lgnd, fill = lgnd) +
  theme(legend.position = "right") +
  ylim(1000,5000)
base2 +
  stat_function(fun = reg0, args = args, aes(color = "female"), size = 1.5) +
  stat_function(fun = reg1, args = args, aes(color = "male"), size = 1.5) 
```

``` r
UofT_prof_sing <- UofT_prof %>% rename("JobTitleSimple" = "outer") %>%
    select(`Last Name`, FirstName, SalaryPaid, `Taxable Benefits`, JobTitleSimple, CalendarYear, gender) %>% 
    mutate(SalaryPaid100 = SalaryPaid/100) %>% select(-SalaryPaid) %>% 
    mutate(CalendarYear = as.integer(as.character(CalendarYear))) %>% 
    filter(JobTitleSimple == "Professor")

post5 <- stan_glm(SalaryPaid100 ~ gender, data = UofT_prof_sing,
                  #family = gaussian(link = "identity"),
                  cores = 6, seed = 12345)

summary(post5)
```

\#no work

``` r
Salary_SQ <- seq(from = 1000, to = 5900, by = 100)
# y_female <- tibble(gender = "female", SalaryPaid100 = Salary_SQ, JobTitleSimple = "Professor")
# y_female <- posterior_predict(post5, newdata = y_female)
# 
# y_male <- tibble(gender = "male", SalaryPaid100 = Salary_SQ, JobTitleSimple = "Professor")
# y_male <- posterior_predict(post5, newdata = y_male)

y_female <- tibble(gender_female = 0, SalaryPaid100 = Salary_SQ)
y_male <- tibble(gender_female = 1, SalaryPaid100 = Salary_SQ)
rbind(y_female,y_male)

y_<- posterior_predict(fit1, newdata = rbind(y_female,y_male))
```

``` r
par(mfrow = c(1:2), mar = c(5,4,2,1))
boxplot(y_, axes = FALSE, outline = FALSE, ylim = c(0,1),
        xlab = "Salary", ylab = "Predicted 2020 Salary", main = "female")
axis(1, at = 1:50, labels = Salary_SQ, las = 3)
axis(2, las = 1)
boxplot(y_, outline = FALSE, col = "red", axes = FALSE, ylim = c(0,1),
        xlab = "Salary", ylab = NULL, main = "male")
axis(1, at = 51:100, labels = Salary_SQ, las = 3)
```

## References

<div id="refs" class="references">

<div id="ref-rstanarm">

Brilleman, SL, MJ Crowther, M Moreno-Betancur, J Buros Novik, and R
Wolfe. n.d. “Joint Longitudinal and Time-to-Event Models via Stan.”
<https://github.com/stan-dev/stancon_talks/>.

</div>

<div id="ref-data.table">

Dowle, Matt, and Arun Srinivasan. 2019. *Data.table: Extension of
‘Data.frame‘*. <https://CRAN.R-project.org/package=data.table>.

</div>

<div id="ref-bayesplot">

Gabry, Jonah, Daniel Simpson, Aki Vehtari, Michael Betancourt, and
Andrew Gelman. 2019. “Visualization in Bayesian Workflow.” *J. R. Stat.
Soc. A* 182 (2): 389–402. <https://doi.org/10.1111/rssa.12378>.

</div>

<div id="ref-lubridate">

Grolemund, Garrett, and Hadley Wickham. 2011. “Dates and Times Made Easy
with lubridate.” *Journal of Statistical Software* 40 (3): 1–25.
<https://www.jstatsoft.org/v40/i03/>.

</div>

<div id="ref-fastDummies">

Kaplan, Jacob. 2020. *FastDummies: Fast Creation of Dummy (Binary)
Columns and Rows from Categorical Variables*.
<https://CRAN.R-project.org/package=fastDummies>.

</div>

<div id="ref-gender">

Mullen, Lincoln. 2020. *Gender: Predict Gender from Names Using
Historical Data*. <https://github.com/ropensci/gender>.

</div>

<div id="ref-here">

Müller, Kirill. 2017. *Here: A Simpler Way to Find Your Files*.
<https://CRAN.R-project.org/package=here>.

</div>

<div id="ref-R">

R Core Team. 2020. *R: A Language and Environment for Statistical
Computing*. Vienna, Austria: R Foundation for Statistical Computing.
<https://www.R-project.org/>.

</div>

<div id="ref-ggplot2">

Wickham, Hadley. 2016. *Ggplot2: Elegant Graphics for Data Analysis*.
Springer-Verlag New York. <https://ggplot2.tidyverse.org>.

</div>

<div id="ref-tidyverse">

Wickham, Hadley, Mara Averick, Jennifer Bryan, Winston Chang, Lucy
D’Agostino McGowan, Romain François, Garrett Grolemund, et al. 2019.
“Welcome to the tidyverse.” *Journal of Open Source Software* 4 (43):
1686. <https://doi.org/10.21105/joss.01686>.

</div>

<div id="ref-kableExtra">

Zhu, Hao. 2020. *KableExtra: Construct Complex Table with ’Kable’ and
Pipe Syntax*. <https://CRAN.R-project.org/package=kableExtra>.

</div>

</div>
