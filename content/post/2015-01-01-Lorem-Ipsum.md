---
title: "Extinct Plants"
output: pdf_document
---

```{r setup, include=FALSE}
knitr::opts_chunk$set(echo = TRUE)
```

```{r}
library(ggplot2)
library(dplyr)
library(stringr)
library(tidyverse)
library(here)
library(patchwork)
```

```{r}
ep <- read.csv(here("tidytuesday-master", "data", "2020", "2020-08-18", "plants.csv"))

ap <- read.csv(here("tidytuesday-master", "data", "2020", "2020-08-18", "actions.csv"))

ep
ap
```

```{r}
country_clean <- ep %>%
  filter(!is.na(country)) %>%
  mutate(abbr = substr(country, 1,10)) %>%
  group_by(country, red_list_category, abbr) %>%
  summarize(Count = n()) %>%
  filter(Count >= 3)

c1 <- ggplot(country_clean, aes(x= abbr, y=Count, color = red_list_category)) +
  geom_point() +
  theme(axis.text.x = element_text(angle = 90))

c2 <- ggplot(ep, aes(x=continent, fill = group)) +
  geom_bar() 
  
c2 /
  c1
```

```{r}

threat <- ep %>%
  group_by(year_last_seen) %>%
  mutate(threat_level = (threat_AA + threat_BRU + threat_RCD + threat_ISGD + threat_EPM + threat_CC + threat_HID + threat_P + threat_TS + threat_NSM + threat_GE + threat_NA)) %>%
  mutate(action_level = action_LWP + action_SM + action_LP + action_RM + action_EA + action_NA) %>%
  filter(!is.na(year_last_seen)) %>%
  ungroup(year_last_seen) %>%
  mutate(year_last_seen = factor(year_last_seen, levels=c("Before 1900", "1900-1919", "1920-1939", "1940-1959", "1960-1979", "1980-1999", "2000-2020")))

line_graph <- threat %>%
  group_by(year_last_seen) %>%
  summarize(Count = n())  

ggplot(data = threat, aes(x = year_last_seen, y = threat_level)) +
  geom_point(position = "jitter", aes(size = action_level, color = red_list_category)) +
  scale_y_continuous(name = "Threat Level", sec.axis = sec_axis(trans=~.*20, name = "Number of Extinctions")) +
  geom_boxplot(data = line_graph, aes(x = year_last_seen, y = Count/15))

```

```{r}
actions <- ap %>%
  left_join(ep) %>%
  filter(action_taken == 1) %>%
  group_by(country) %>%
  mutate(con_count = n()) %>%
  filter(con_count >= 12)

ggplot(actions, aes(x= action_type, fill = country))+
  geom_bar() +
  theme(axis.text.x = element_text(angle = 90))

ggplot(actions, aes(x= action_type, fill = continent))+
  geom_bar() +
  theme(axis.text.x = element_text(angle = 90))
````