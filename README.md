## CASE STUDY: U.S Gun Violence 
**Author: Domingo Guzman**

**Date: October 24, 2022**
#
_The case study follows the six step data analysis process:_

### ❓ [Ask](#1-ask)
### 💻 [Prepare](#2-prepare)
### 🛠 [Process](#3-process)
### 📊 [Analyze](#4-analyze)
### 📋 [Share](#5-share)
### 🧗‍♀️ [Act](#6-act)

#
![CSVGLogo](https://user-images.githubusercontent.com/77591203/197647981-c78f3dc8-f120-4499-ab57-d8b371a45c0d.png)


## Introduction

The Coalition to Stop Gun Violence (CSGV) is a non-profit gun control advocacy organization that is opposed to gun violence. The CSGV was founded in 1974, making it the nation’s oldest gun violence prevention organization. The CSGV believes that gun violence should be rare and abnormal. They pursue the goal of ending gun violence through policy development, advocacy, community engagement, and effective training. Through a combination of evidence-based policy development and aggressive lobbying, the CSGV is leading the way forward in fighting for a safer and stronger America. The organization has nine areas of focus regarding issues and campaigns. This case study will focus on two of them: 

1: Support for stricter mental health screening for firearm purchases. <br>
2: Ban the private sale of guns by instituting universal background checks.

Our analytics team has been asked to come up with strong, evidence-backed points that will assist the CSGV in gaining supporters.

## 1. Ask

**BUSINESS TASK: Analyze U.S gun violence data to create strong arguments that support the two areas of focus above for an upcoming campaign**

Primary stakeholders: Joshua Horwitz, executive director.

Secondary stakeholders: Local communities

## 2. Prepare
Data Sources: Gun Violence Incidents in the United States from Emmanuel F. Werr: https://www.kaggle.com/datasets/emmanuelfwerr/gun-violence-incidents-in-the-usa <br>
US Police Shootings from 2015- Sep 2022 from Ram Jas: https://www.kaggle.com/datasets/ramjasmaurya/us-police-shootings-from-20152022

The datasets have 3 CSV files, 24 columns, and 500,000 rows. The data also follows a ROCCC approach:

* **Reliability - MED:** The Gun Violence Incidents in the Unites States data is complete and accurate. It comes from the Gun Violence Archive (GVA). The GVA website maintains a database of known shootings in the United States, coming from law enforcement, media and government sources from all 50 states. The US Police shootings data was scraped from Wikipedia.
* **Original - MED:** GVA is an independent data collection and research group. Data on the US Police Shootings dataset gathered from The Counted, a website that tracks the number of people killed by police in the US.
* **Comprehensive - HIGH:** The data includes names, dates, manner of death, state where death occurred, city, address, number of people killed & injured, age, race, gender, whether victim was armed, and if there were signs of mental illness. 
* **Current - HIGH:** The data is current. It goes from 2013 to September 2022.
* **Cited - MED:** The data is cited from <a href="https://www.gunviolencearchive.org/">Gun Violence Archive</a>, and <a href="https://en.wikipedia.org/wiki/Lists_of_killings_by_law_enforcement_officers_in_the_United_States">Wikipedia</a>

### Loading packages

```
library(tidyverse)
library(lubridate) 
library(dplyr)
library(ggplot2)
library(tidyr)
library(janitor)
```

## 3. Process

### Importing the datasets

```
# Read the dataframes
all_incidents <- read_csv(all_incidents.csv)
mass_shootings <- read_csv("mass_shootings.csv")
police_shootings <- read_csv("police_shootings.csv")
```

Examining the data:

```
head(all_incidents)
colnames(all_incidents)
dim(all_incidents)
```

First, I will remove the "incident_id" column since it will not be used in the analysis.

```
all_incidents_clean <- all_incidents %>% select(-c(incident_id))
```

The all_incidents dataset covers data from 2013 - present day, while the police_shootings dataset covers data from 2015 - September 2022. This means that data from 2013-2014 will need to be factored out from all_incidents so that the same years can be compared.

```
# removing years 2013-2014
all_incidents_new <- all_incidents_clean %>% filter(date <= "2014-12-31")
```

Let's confirm that the years 2013-2014 were removed by checking the end of the dataframe.

```
tail(all_incidents_new)
```

```
# A tibble: 6 × 6
  date       state      city          address           n_kil…¹ n_inj…²
  <date>     <chr>      <chr>         <chr>               <dbl>   <dbl>
1 2015-01-01 Florida    Alachua       7108 NW 92nd Pla…       0       0
2 2015-01-01 New Jersey Jersey City   Virginia Avenue         0       3
3 2015-01-01 New York   Staten Island 1307 Arthur Kill…       0       2
4 2015-01-01 Michigan   Saint Joseph  396 Upton Drive         1       0
5 2015-01-01 New York   Rochester     402 West Ridge R…       0       1
6 2015-01-01 Ohio       Lorain        2217 East 28th St       0       3
```

Now that the correct years for both datasets have been verified, the next step is fixing the 'state' categories. The 'state' category in the police_shootings dataset is abbreviated, while the full name of the state is spelled out in the all_incidents_new and mass_shootings datasets. This must be fixed so the state formats are all the same.

```
head(all_incidents_new)

# A tibble: 6 × 6
  date       state          city        address         n_kil…¹ n_inj…²
  <date>     <chr>          <chr>       <chr>             <dbl>   <dbl>
1 2022-05-28 Arkansas       Little Rock W 9th St and B…       0       1
2 2022-05-28 Colorado       Denver      3300 block of …       0       1
3 2022-05-28 Missouri       Saint Louis Page Blvd and …       0       1
4 2022-05-28 South Carolina Florence    Old River Rd          0       2
5 2022-05-28 California     Carmichael  4400 block of …       1       0
6 2022-05-28 Kentucky       Louisville  400 block of M…       0       1
```

```
head(police_shootings)

# A tibble: 6 × 17
     id name    date       manne…¹ armed   age gender race  city  state
  <dbl> <chr>   <date>     <chr>   <chr> <dbl> <chr>  <chr> <chr> <chr>
1     1 Tim El… 2015-01-02 shot    gun      53 M      A     Shel… WA   
2     2 Lewis … 2015-01-02 shot    gun      47 M      W     Aloha OR   
3     3 John P… 2015-01-03 shot a… unar…    23 M      H     Wich… KS   
4     4 Matthe… 2015-01-04 shot    toy …    32 M      W     San … CA   
5     5 Michae… 2015-01-04 shot    nail…    39 M      H     Evans CO   
6     6 Kennet… 2015-01-04 shot    gun      18 M      W     Guth… OK 
```

```
head(mass_shootings)

# A tibble: 6 × 7
  `Incident ID` `Incident Date`   State City …¹ Address # Kil…² # Inj…³
          <dbl> <chr>             <chr> <chr>   <chr>     <dbl>   <dbl>
1        271363 December 29, 2014 Loui… New Or… Poydra…       0       4
2        269679 December 27, 2014 Cali… Los An… 8800 b…       1       3
3        270036 December 27, 2014 Cali… Sacram… 4000 b…       0       4
4        269167 December 26, 2014 Illi… East S… 2500 b…       1       3
5        268598 December 24, 2014 Miss… Saint … 18th a…       1       3
6        267792 December 23, 2014 Kent… Winche… 260 Ox…       1       3
```

I will use the built in state.abb and match functions to achieve this:

```
# change all states into their abbreviations 

all_incidents_new$state <- state.abb[match(all_incidents_new$state, state.name)]

mass_shootings$State <- state.abb[match(mass_shootings$State, state.name)]
```

** Next is removing null values, unnecessary columns **
