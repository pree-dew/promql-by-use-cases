**Use Case**

How to find the Apdex score of my HTTP application?

**Explanation**

Apdex stands for Application Performance Index. It is a measure of application performance with respect to user satisfaction.

### Why Apdex score is a better measure of application performance?

The definition of application performance varies as per different personas.
For the engineer, it's response time; for SRE, it's uptime; for the product manager, it is user retention on the product.
These definitions vary as per the roles or priorities of each role. In such cases, we have to find a uniform way to measure the application
performance across multiple services, applications, and teams. How to do that?

**Apdex** solves that problem by quantifying **user experience** to a number, as at the end of any business
all roles are trying to find user experience by using different measurements.

To define Apdex in terms of user experiences, we take three different categories of users.
1. **Satisfied** - One who enjoys the application experience without hiccups or slowness.
2. **Tolerating** - One who faces a lag or slowness but keeps using the application without complaining.
3. **Frustrated** - One who abandoned the application due to lag or slowness.

Based on application response time, the Apdex score measures satisfied, tolerating, and frustrated users.

### How to find an Apdex score for an HTTP application?

- Define a satisfied user response time threshold, for example,
a happy customer would get a response within 1 second.

- 4 times the satisfied user threshold defines the threshold for tolerating users. So, using the above example, anything from 1 second to 4 seconds defines the tolerating user response time. Anything above that defines frustrated user response time.

- Find the Apdex score by:

  ```text
  Apdex score = (
                    No. of satisfied users +
                    (0.5) * No. of tolerating users
                     + 0 * No. of frustrated users
                ) / Total users
  ```

For any application, the number of users can be quantified by the number of requests, which is throughput.

Assumptions:
- Application received 1000 requests in total.
- The satisfied user threshold is 1 second.
- No. of requests finished within 1 second = 600
- No. of requests finished within 1 to 4 seconds = 200
- No. of requests finished greater than 4 seconds = 200

Apdex Score will be:

- Satisfied users : (600 * 1 = 600)
- Tolerating users : (200 * 0.5 = 100)
- Frustrating users : (200 * 0 = 0)

  ```
  Apdex score = (600 + 100 + 0 ) / 1000 = 0.7
  ```

**Promql**
- Threshold For Satisified Users : taking p50 **as** threshold
  
  **promql**
  ```
  histogram_quantile(0.50, sum by (le) (rate(http_requests_ms_bucket{}[4m])*60))
  ```
  
- Satisfied Users **as**  satisfied_users
  
  **promql**
  
  ```
   sum(
     topk(1, sum by (le) (rate(http_requests_ms_bucket{}[4m]) * 60)
     and
     label_value(sum by (le)(http_requests_ms_bucket{}), "le") <  threshold))
  ```

- Tolerating Users **as** tolerating_users
  
  **promql**
  
  ```
  sum(
     topk(1, sum by (le) (rate(http_requests_ms_bucket{}[4m]) * 60)
     and
     label_value(sum by (le)(http_requests_ms_bucket{}), "le") <  4 * threshold))

  -

  sum(
     topk(1, sum by (le) (rate(http_requests_ms_bucket{}[4m]) * 60)
     and
     label_value(sum by (le)(http_requests_ms_bucket{}), "le") <  threshold))
  
  ```

- Total Users **as** total_users
  
  **promql**
  
  ```
  sum (rate(http_requests_duration_milliseconds_count{}[4m]) * 60))
  ```  

- Final Promql

  ```
  Appdex score = ( satisfied_users + ( tolerating_users / 2 ) ) / total_users
  ```
  
**Function Explanations by Promqls**
```
label_value(query, label)

label_value(sum by (le)(http_requests_ms_bucket{}), "le") <  threshold
```

- `label_value` is used to get any label value from a query.
  Whatever labels are coming as part of output series from query, we can select which label values we are
  interested in by providing it's label as 2nd argument to **label_value** 
- Here threshold is latency (p50) so it must be either in seconds or milliseconds. On receiving label value
  **le** which gives time buckets, we are filtering all buckets which has value less than threshold.


```
query1
  and
query2

sum by (le) (rate(http_requests_ms_bucket{}[4m]) * 60)
     and
label_value(sum by (le)(http_requests_ms_bucket{}), "le")

```

- `and` operator is used to finding intersection between query1 and query2
- Here it will give all those timeseries where ouput timeseries of query1 is iƒprontersecting with output
  timeseries of query2

```
topk(k_no_of_series, query)

topk(1, sum by (le) (rate(http_requests_ms_bucket{}[4m]) * 60)
     and
     label_value(sum by (le)(http_requests_ms_bucket{}), "le") <  threshold)
```

- `topk` is used to get top k no. of series with highest value from query
- Here we are finding top 1 value with highest number of requests after performing **and** operation on
   query1 and query2. It is basically finding the bucket with highest number of requests with `le` less than 
   threshold.

