# cyclistic data analysis process

## Data Processing

### Data Collection

We started by collecting the rides data of the last year. It was in 9 seprate csv files, so we needed to merge them into one csv_file.

```{r importing the data}
library(tidyverse)
library(lubridate)

q1 = read_csv('q1.csv')

april = read_csv('202004.csv')
may = read_csv('202005.csv')
june = read_csv('202006.csv')

july = read_csv('202007.csv')
august = read_csv('202008.csv')
september = read_csv('202009.csv')

october = read_csv('202010.csv')
november = read_csv('202011.csv')
december = read_csv('202012.csv')
```

after importing each month, we could merge them into quarters first and then merge all these quarters into one csv for the whole year. That way, if any month has faulty data-types or something else that might be a problem, we could troubleshoot it easily

```{r data merging, eval=FALSE, include=FALSE}
q2 <- bind_rows(april, may, june)
q3 <- bind_rows(july, august, september)
q4 <- bind_rows(october, november, december)
```

We can see that one of the months has the start_station_id stored as a string rather than a double, we can use the `str` function to see which month caused the error

```{r troubleshooting}
str(october)
str(november)
str(december)
```

after running, we find that `december` has the `start_station_id` and the `end_station_id` stored in `col_character()` rather than `col_double()`.

to fix this issue, we could type-cast the columns to render as a `double` using the `as.double()` function

```{r error fix}
december$start_station_id <- as.double(december$start_station_id)
december$end_station_id <- as.double(december$end_station_id)
```

after re-running the `data merging` chunck, we get the results we were waiting for.
Now we can merge everything into one dataframe for the whole year.

```{r data merging after fix}
q2 <- bind_rows(april, may, june)
q3 <- bind_rows(july, august, september)
q4 <- bind_rows(october, november, december)
```

```{r final data merging}
rides <- bind_rows(q1, q2, q3, q4)
```


### Removing Irrelevant Data

We can see that there are some columns that are totally irrelevant to the buiseness task at hand, so it's better to remove them.

```{r removing irrelevant data}
rides <- select(rides, -c(start_station_name, start_station_id, start_lng, start_lat, end_station_name, end_station_id, end_lng, end_lat))
```

Let's save the final csv file, so that we don't have to repeat all the steps if something goes wrong

```{r}
write_csv(rides, 'rides.csv')
```


## Data Analysis

As the business task was to find out the differences between the casual riders and the subscribed riders (members in the data set) in terms of bike usage, we will need to add some new columns that will help us further aggregate and analyze the data set. So after assuring that all columns we're going to use have their appropriate types, we start to create three new columns:

-   ride_date --\> representing the date of the ride
-   ride_day --\> representing the weekday of the ride
-   ride_duration --\> representing the time the ride took

```{r adding columns for further analysis}
rides$ride_date <- date(rides$started_at)
rides$ride_day <- weekdays(rides$ride_date)
rides$ride_duration <- as.numeric(difftime(rides$ended_at, rides$started_at))
```

### Data aggregation

After finally having all the data we need, it's time to start analyzing that data. For analyzing the data, we need to aggregate that data so that it gives us only only the important parts for knowing how the casuals and members differ.

We need the espresso of the data, if you will ;)

We will aggregate the data we have, to show us only the average ride_time for each membership on each weekday. so for example, in sundays, the average ride_time for casual riders is 12.3, and the average ride_time for members on sundays is 13.2.

```{r getting the important parts of the data.}
rides_summary <- aggregate(rides$ride_duration ~ rides$ride_day + rides$member_casual, FUN = mean) %>% rename(avg_ride_duration = 'rides$ride_duration', day = 'rides$ride_day', membership = 'rides$member_casual')
```

```{r sorting the aggregated data}
rides_summary$day <- factor(rides_summary$day, levels = c("Sunday", "Monday", 
    "Tuesday", "Wednesday", "Thursday", "Friday", "Saturday"))

rides_summary <- rides_summary[order(rides_summary$day), ]
```

### Saving the data
Now that our data is aggregated, we could generate a nice final report using google sheets.
let's save the aggregated data.

```{r}
write_csv(rides_summary, 'rides_aggregated.csv')
```

