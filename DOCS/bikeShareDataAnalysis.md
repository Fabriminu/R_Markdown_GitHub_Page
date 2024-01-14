Bike Share-Data Analysis-Google Data Analytics-Capstone Project
================
Fabrizio Minute
2024-01-11

## Objective

Define how annual members and casual riders use *Cyclistic* bikes
differently

### Source data and software

Cyclistic’s historical trip data (From January to December 2023).

All analyzes were performed using R Software. The final report was done
via R Markdown.

### Import and cleaning data

Two packages were used:

- *tidyverse*: for data analysis
- *ggpubr*: to use the ggarrange function (allows you to merge several
  graphs created via ggplot)

``` r
library(tidyverse)
library(ggpubr)
```

After setting the working directory, the 12 csv files (one for each
month) with the data to be processed were imported via *read.csv*.

``` r
setwd("C:\\Users\\fabri\\OneDrive\\Desktop\\Analisi\\2023\\Coursera\\Google\\Data\\Original")
January<-read.csv("202301-divvy-tripdata.csv") 
February<-read.csv("202302-divvy-tripdata.csv") 
March<-read.csv("202303-divvy-tripdata.csv") 
April<-read.csv("202304-divvy-tripdata.csv") 
May<-read.csv("202305-divvy-tripdata.csv") 
June<-read.csv("202306-divvy-tripdata.csv") 
July<-read.csv("202307-divvy-tripdata.csv") 
August<-read.csv("202308-divvy-tripdata.csv") 
September<-read.csv("202309-divvy-tripdata.csv") 
October<-read.csv("202310-divvy-tripdata.csv") 
November<-read.csv("202311-divvy-tripdata.csv") 
December<-read.csv("202312-divvy-tripdata.csv") 
```

A single dataset was created using the *bind_rows* function.

``` r
df<-bind_rows(January, February, March, April, May, June, July, August, September, October, December)
```

*str()* function to compactly display the internal structure of df.

``` r
str(df)
```

    ## 'data.frame':    5357359 obs. of  13 variables:
    ##  $ ride_id           : chr  "F96D5A74A3E41399" "13CB7EB698CEDB88" "BD88A2E670661CE5" "C90792D034FED968" ...
    ##  $ rideable_type     : chr  "electric_bike" "classic_bike" "electric_bike" "classic_bike" ...
    ##  $ started_at        : chr  "2023-01-21 20:05:42" "2023-01-10 15:37:36" "2023-01-02 07:51:57" "2023-01-22 10:52:58" ...
    ##  $ ended_at          : chr  "2023-01-21 20:16:33" "2023-01-10 15:46:05" "2023-01-02 08:05:11" "2023-01-22 11:01:44" ...
    ##  $ start_station_name: chr  "Lincoln Ave & Fullerton Ave" "Kimbark Ave & 53rd St" "Western Ave & Lunt Ave" "Kimbark Ave & 53rd St" ...
    ##  $ start_station_id  : chr  "TA1309000058" "TA1309000037" "RP-005" "TA1309000037" ...
    ##  $ end_station_name  : chr  "Hampden Ct & Diversey Ave" "Greenwood Ave & 47th St" "Valli Produce - Evanston Plaza" "Greenwood Ave & 47th St" ...
    ##  $ end_station_id    : chr  "202480.0" "TA1308000002" "599" "TA1308000002" ...
    ##  $ start_lat         : num  41.9 41.8 42 41.8 41.8 ...
    ##  $ start_lng         : num  -87.6 -87.6 -87.7 -87.6 -87.6 ...
    ##  $ end_lat           : num  41.9 41.8 42 41.8 41.8 ...
    ##  $ end_lng           : num  -87.6 -87.6 -87.7 -87.6 -87.6 ...
    ##  $ member_casual     : chr  "member" "member" "casual" "member" ...

Selection of the variables of interest (in this project I did not
consider geographical coordinates although this information could
provide important information on the different behavior of users).

``` r
df_sel<-df%>%
  select(ride_id,started_at,ended_at,member_casual)
```

Check na values.

``` r
sum(is.na(df_sel))
```

    ## [1] 0

The two variables started_at and ended_at are of type “character”. You
need to convert them to “datetime”.

``` r
df_sel$started_at<-as_datetime(df_sel$started_at)
df_sel$ended_at<-as_datetime(df_sel$ended_at)
```

Three new variables were created using the *mutate* function:

- *day*
- *month*
- *time_track* (difference between started_at and ended_at), which
  indicates the duration of the bike rental

``` r
df_final<-df_sel %>% 
  mutate(month=month(started_at,label=TRUE),day=wday(started_at,label=TRUE),time_track=difftime(ended_at,started_at,units = "mins"))
```

*time_track* turns out to be negative in some cases. There were probably
data entry errors in the database. For this reason I decided to only
consider trips \> 1 minute (minimum reasonable time for a bicycle trip)
and \< 1 day (1440 minutes).

In a real case study, this aspect should be explored in more detail.

``` r
df_final_cleaned<-df_final %>% 
  filter(time_track>1 & time_track<1440)
```

### Data analysis and visualisation

In order to evaluate the different behavior of Cyclistic members, I
created a summary table with the sum of the *time_track*, the number of
runs (*counts*) and the average time per run (*mean*).

``` r
df_summary<-df_final_cleaned %>% 
  group_by(member_casual) %>% 
  summarize(sum=sum(time_track), mean=mean(time_track),count=n()) %>% 
  arrange(-sum)
```

Then i plotted the number of bike rides, the total time and the average
time per ride into three graphs, subsequently joined using the
*ggarrange* function. The most interesting data that emerges is the
average time spent cycling, which is longer for casual customers. This
data suggests a different use of bikes by the two categories of
customers.

``` r
total_count_plot<-ggplot(df_summary,aes(member_casual,count,fill=member_casual))+
  geom_bar(stat="identity")+
  labs(title = "N° BICYCLE RIDES",y="COUNT")+
  theme(legend.title = element_blank(),
        legend.text = element_text(size=14),
        axis.text.x = element_blank(),
        axis.title.x = element_blank(),
        plot.title = element_text(size=10))

time_track_plot<-ggplot(df_summary,aes(member_casual,sum,fill=member_casual))+
  geom_bar(stat="identity")+
  labs(title = "TOTAL TIME TRACK",y="TIME(minutes)")+
  theme(legend.title = element_blank(),
        legend.text = element_text(size=14),
        axis.text.x = element_blank(),
        axis.title.x = element_blank(),
        plot.title = element_text(size=10))

mean_time_track_plot<-ggplot(df_summary,aes(member_casual,mean,fill=member_casual))+
  geom_bar(stat="identity")+
  labs(title = "MEAN TIME TRACK",y="TIME(minutes)")+
  theme(legend.title = element_blank(),
        legend.text = element_text(size=14),
        axis.text.x = element_blank(),
        axis.title.x = element_blank(),
        plot.title = element_text(size=10))

ggarrange(total_count_plot,time_track_plot,mean_time_track_plot,ncol=3,common.legend = TRUE,legend = "bottom")
```

<img src="bikeShareDataAnalysis_files/figure-gfm/unnamed-chunk-7-1.png" style="display: block; margin: auto;" />

For this reason, two summary tables were created to try to understand if
there are different weekly and monthly trends for the two categories of
customers.

From the analysis of the number of bike trips and the time spent during
the different trips, two distinct trends emerge for the two types of
customers:

- *member*: bicycles used mainly on working days;
- *casual*: bicycles used mostly on weekends.

These results suggest that member customers use bikes mainly for work
reasons, while casual customers probably use them for leisure reasons.

``` r
#summary table (days)

df_summary_day<-df_final_cleaned %>% 
  group_by(member_casual,day) %>% 
  summarize(sum=sum(time_track), mean=mean(time_track),count=n()) %>% 
  arrange(-sum)
```

``` r
sum_day_plot<-ggplot(df_summary_day,aes(member_casual,sum,fill=member_casual))+
  geom_bar(stat="identity")+
  facet_grid(~day)+
  labs(title = "TOTAL TIME TRACK",y="TIME(minutes)")+
  theme(legend.title = element_blank(),
        legend.text = element_text(size=14),
        axis.text.x = element_blank(),
        axis.title.x = element_blank(),
        plot.title = element_text(size=10))

count_day_plot<-ggplot(df_summary_day,aes(member_casual,count,fill=member_casual))+
  geom_bar(stat="identity")+
  facet_grid(~day)+
  labs(title = "N° BICYCLE RIDES",y="COUNT")+
  theme(legend.title = element_blank(),
        legend.text = element_text(size=14),
        axis.text.x = element_blank(),
        axis.title.x = element_blank(),
        plot.title = element_text(size=10))

mean_day_plot<-ggplot(df_summary_day,aes(member_casual,mean,fill=member_casual))+
  geom_bar(stat="identity")+
  facet_grid(~day)+
  labs(title = "MEAN TIME TRACK",y="TIME(minutes)")+
  theme(legend.title = element_blank(),
        legend.text = element_text(size=14),
        axis.text.x = element_blank(),
        axis.title.x = element_blank(),
        plot.title = element_text(size=10))

plot_days<-ggarrange(sum_day_plot,mean_day_plot,count_day_plot,nrow=1,common.legend = TRUE,legend = "bottom")

annotate_figure(plot_days, top = text_grob("WEEKLY TREND", color = "red", face = "bold", size = 14))
```

<img src="bikeShareDataAnalysis_files/figure-gfm/unnamed-chunk-9-1.png" style="display: block; margin: auto;" />

The analysis of the trends in the different months suggests a seasonal
trend for both customers, with a greater number of bike rides in the
warmer months compared to the colder ones. This graph also clearly shows
the differences in the average time of using the bicycle among the
different customers (greater among casual customers).

``` r
df_summary_month<-df_final_cleaned %>% 
  group_by(member_casual,month) %>% 
  summarize(sum=sum(time_track), mean=mean(time_track),count=n()) %>% 
  arrange(-sum)
```

``` r
sum_month_plot<-ggplot(df_summary_month,aes(member_casual,sum,fill=member_casual))+
  geom_bar(stat="identity")+
  facet_grid(~month)+
  labs(title = "TOTAL TIME TRACK",y="TIME(minutes)")+
  theme(legend.title = element_blank(),
        legend.text = element_text(size=14),
        axis.text.x = element_blank(),
        axis.title.x = element_blank(),
        plot.title = element_text(size=10))

count_month_plot<-ggplot(df_summary_month,aes(member_casual,count,fill=member_casual))+
  geom_bar(stat="identity")+
  facet_grid(~month)+
  labs(title = "N° BICYCLE RIDES",y="COUNT")+
  theme(legend.title = element_blank(),
        legend.text = element_text(size=14),
        axis.text.x = element_blank(),
        axis.title.x = element_blank(),
        plot.title = element_text(size=10))

average_month_plot<-ggplot(df_summary_month,aes(member_casual,mean,fill=member_casual))+
  geom_bar(stat="identity")+
  facet_grid(~month)+
  labs(title = "MEAN TIME TRACK",y="TIME(minutes)")+
  theme(legend.title = element_blank(),
        legend.text = element_text(size=14),
        axis.text.x = element_blank(),
        axis.title.x = element_blank(),
        plot.title = element_text(size=10))

plot_months<-ggarrange(sum_month_plot,average_month_plot,count_month_plot,nrow=1,common.legend = TRUE,legend = "bottom")

annotate_figure(plot_months, top = text_grob("SEASONAL TREND", color = "red", face = "bold", size = 14))
```

<img src="bikeShareDataAnalysis_files/figure-gfm/unnamed-chunk-11-1.png" style="display: block; margin: auto;" />

## CONCLUSIONS

The analysis of the data for the year 2023 highlighted how there are
differences in the behavior of *member* customers compared to *casual*
customers.

In particular, *member* customers seem to use the bicycle more
constantly during the different days of the week, probably because they
use the bicycle to go to work.

*Casual* customers, on the other hand, use the bike more frequently
during the weekend, with longer usage times than *member* customers. To
verify through analysis of the departure and arrival locations the
presence of particular “attractions” (libraries, cinemas, etc.) which
would support the hypothesis of use of bicycles by casual customers for
recreational purposes.
