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

- `rate` will give us change in metric per sec calculated over 1m. Multiplying by 60 will
   give this change in metric per min.
-  Aggregated the metric by label1 and label2 for each min. using sum by over 1min data receivied from rate.
-  `max_over_time` find the series with maximum value over a duration of time specified. Here, it will take
    **xh** of data and find maximum from it.


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
