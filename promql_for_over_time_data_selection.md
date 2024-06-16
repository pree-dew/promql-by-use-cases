**Use Case**

Find max value of metric received in past x hours where metric is a counter.

**Promql**

```
max_over_time (
     sum by (label1, label2) (
         rate (metric{}[ scrape_interval * 4 ]) * 60
     ) [xh] )
```

**Explanation**

- `rate` will give us change in metric per sec calculated over 4 times of scrape_interval. Multiplying by 60 will
   give this change in metric per min. Further explanation: 
   https://github.com/pree-dew/promql-by-use-cases/blob/main/promql_for_counters.md#using-rate-on-counters
-  Aggregate the metric by label1 and label2 for each min (assuming step = 1m), using `sum by`.
-  `max_over_time` find the series with maximum value over a duration of time specified. Here, it will take
    **xh** of data and find maximum from it. Depending on your step interval, it will first find
     ( (x * 60) / step ) data points, then find max from those data points per timeseries.


**Example**

Suppose if we want to find maximum number of requests received over a duration of 1 hour

```
max_over_time (
     sum by (service) (
         rate (http_requests_total{}[4m]) * 60
     ) [1h] )
```

<img width="1331" alt="Screenshot 2024-01-16 at 11 44 08 AM" src="https://github.com/pree-dew/promql-by-use-cases/assets/132843509/ec60c77b-bd26-492b-b0f4-61b382576d0d">


Green -> actual signal

Pink -> max calculated 1 hour

1. It will find number of requests received per min
2. Aggregate it on service label for each min
3. Take 60 data points to calculate max_over_time, assuming step = 1m and return it's max



**Use Case**

Find average value of metric received in past x hours where metric is a counter.

**Promql**

```
avg_over_time (
     sum by (label1, label2) (
         rate (metric{}[ scrape_interval * 4 ]) * 60
     ) [xh] )
```

**Explanation**

- `rate` will give us change in metric per sec calculated over 4 times of scrape_interval. Multiplying by 60 will
   give this change in metric per min. Further explanation: 
   https://github.com/pree-dew/promql-by-use-cases/blob/main/promql_for_counters.md#using-rate-on-counters
-  Aggregate the metric by label1 and label2 for each min (assuming step = 1m), using `sum by`.
-  `avg_over_time` find the series with average value over a duration of time specified. Here, it will take
    **xh** of data and find maximum from it. Depending on your step interval, it will first find
     ( (x * 60) / step ) data points, then find average of all those points per timeseries.


**Example**

Suppose if we want to find avg number of requests received over a duration of 1 hour

```
avg_over_time (
     sum by (service) (
         rate (http_requests_total{}[4m]) * 60
     ) [1h] )
```

<img width="1358" alt="Screenshot 2024-06-16 at 4 33 29 PM" src="https://github.com/pree-dew/promql-by-use-cases/assets/132843509/c397715c-23ca-46c3-80ff-0373d95aef57">

Yellow -> Original metric
Green -> Average of all point received in past 1hour

Mathematically, if we want to find avg over 5mins from 10:00 am to 10:05 am

| time   | 10:01am | 10:02am | 10:03am | 10:04am | 10:05am | 
| -------| ------- | ------  | ------  | ------- | ------- |
| Value  |   20    |   30    |   40    |   50    |   60    |

If `query{}[5m]` output is represent in above table then `avg_over_time(query{}[5m])` will give
 ```
  avg_over_time(query{}[5m]) = (20 + 30 + 40 + 50 + 60 ) / 5
                             = 40
 ```


**Use Case**

Find first sample of metric received in past x hours where metric is a counter.

**Promql**

```
first_over_time (
     sum by (label1, label2) (
         rate (metric{}[ scrape_interval * 4 ]) * 60
     ) [xh] )
```

**Explanation**

- `rate` will give us change in metric per sec calculated over 4 times of scrape_interval. Multiplying by 60 will
   give this change in metric per min. Further explanation: 
   https://github.com/pree-dew/promql-by-use-cases/blob/main/promql_for_counters.md#using-rate-on-counters
-  Aggregate the metric by label1 and label2 for each min (assuming step = 1m), using `sum by`.
-  `first_over_time` will return first sample received in specified window. Here, it will take
    **xh** of data and return first sample from it. Depending on your step interval, it will first find
     ( (x * 60) / step ) data points, then find first sample of those points per timeseries.


**Example**

Suppose if we want to find first sample received for number of requests received over a duration of 1 hour

```
first_over_time (
     sum by (service) (
         rate (http_requests_total{}[4m]) * 60
     ) [1h] )
```
<img width="1358" alt="Screenshot 2024-06-16 at 4 50 36 PM" src="https://github.com/pree-dew/promql-by-use-cases/assets/132843509/fd0b1711-6b47-445a-a32f-6221e2b7ea50">



Yellow -> Original metric
Green -> First point received in past 1 hour.

Image shows the point at `15:51` aligns with green at the end showing same value

Mathematically, if we want to find first sample over 5mins from 10:00 am to 10:05 am

| time   | 10:01am | 10:02am | 10:03am | 10:04am | 10:05am | 
| -------| ------- | ------  | ------  | ------- | ------- |
| Value  |   20    |   30    |   40    |   50    |   60    |

If `query{}[5m]` output is represent in above table then `first_over_time(query{}[5m])` will give
 ```
  first_over_time(query{}[5m]) = 20
 ```


**Use Case**

Find last sample of metric received in past x hours where metric is a counter.

**Promql**

```
last_over_time (
     sum by (label1, label2) (
         rate (metric{}[ scrape_interval * 4 ]) * 60
     ) [xh] )
```

**Explanation**

- `rate` will give us change in metric per sec calculated over 4 times of scrape_interval. Multiplying by 60 will
   give this change in metric per min. Further explanation: 
   https://github.com/pree-dew/promql-by-use-cases/blob/main/promql_for_counters.md#using-rate-on-counters
-  Aggregate the metric by label1 and label2 for each min (assuming step = 1m), using `sum by`.
-  `last_over_time` will return last sample received in specified window. Here, it will take
    **xh** of data and return first sample from it. Depending on your step interval, it will first find
     ( (x * 60) / step ) data points, then find last sample of those points per timeseries.


**Example**

Suppose if we want to find last sample received for number of requests received over a duration of 1 hour

```
last_over_time (
     sum by (service) (
         rate (http_requests_total{}[4m]) * 60
     ) [1h] )
```
<img width="1355" alt="Screenshot 2024-06-16 at 4 59 47 PM" src="https://github.com/pree-dew/promql-by-use-cases/assets/132843509/1b907992-7124-4a5e-b127-08899ea37abd">

<img width="1342" alt="Screenshot 2024-06-16 at 5 05 26 PM" src="https://github.com/pree-dew/promql-by-use-cases/assets/132843509/d8c1b565-2c37-47f7-94eb-09c81c7ab288">



Yellow -> Original metric
Green -> Last point received in past 1 hour.

Image shows the last value is going to be the actual metric value received.
Mathematically, if we want to find last sample over 5mins from 10:00 am to 10:05 am

| time   | 10:01am | 10:02am | 10:03am | 10:04am | 10:05am | 
| -------| ------- | ------  | ------  | ------- | ------- |
| Value  |   20    |   30    |   40    |   50    |   60    |

If `query{}[5m]` output is represent in above table then `last_over_time(query{}[5m])` will give
 ```
  last_over_time(query{}[5m]) = 60
 ```

