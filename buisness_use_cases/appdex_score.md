**Use Case**

How to find Appdex score of my HTTP application?

**Explanation**

Appdex stands for Application Performance Index. It's a measure of application performance with respect
to user satisfaction. 

**Why Appdex score is a better measure**

Definition of application performance varies as per different personas. Foe engineer it's response time, 
for SRE it's uptime, for product it's user retention on product. These definitions are varying as per the 
roles or priorities of these roles. In such cases if we have to find a uniform way to measure application
performance across multiple services, applications, teams then how to do that.
**Appdex** solves that problem by quantifying **user experience** to a number, as at the end for any buisness
all roles are trying to find user experience by using different measurements.

To define appdex in terms of user experiences, we take three different category of users.
1. **Satisfied** - One who enjoys the application experience without any hicups or slowness.
2. **Tolerating** - One who face some lag or slowness but still keep using the application without complaining
3. **Frustrated** - One who is abandones the application due to lag or slowness.

Appdex measures satisfied, tolerating and frustated users on the basis of application response time.

**How to find Appdex score for my application**

- Define a satisified user response time threshold, for example, a happy customer whould get response with
        in 1 sec.
        
- 4 times the satisified user threshold defines the threshold for tolerating users. So by using above example
        anything time from 1 sec to 4 sec defines the tolerating user response time. Anything above that defines
        frustated user response time.
        
- Find appdex user by:

  ```        
  Appdex score = (
                    No. of satisfied user +
                    (0.5) * No. of tolerating user
                     + 0 * No. of frustated user
                ) / Total users
  ``` 

**Example for Appdex maths**

For any application, no of users can be quantifies by no. of requests. 

Assumptions:
- Application received a throughput of 1000 requests.
- Satisfied user thrreshold is 1 sec
- No. of requests finished within 1 sec = 600
- No. of requests finished within 1 to 4 sec = 200
- No. of request finished greater than 4 sec = 200

Appdex Score will be:

- Satisified users : (600 * 1 = 600)
- Tolerating users : (200 * 0.5 = 100)
- Frustating users : (200 * 0 = 0)
  
  ```
  score = (600 + 100 + 0 ) / 1000 = 0.7
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

